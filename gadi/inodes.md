# Where Did My Inodes Go?

You have 12 TB free on `/g/data` and your jobs won't start. `lquota` says you're at 2.1 million of 2 million. Somewhere in a project space that six people have been writing to for three years, something has quietly produced a million files, and now nobody can run anything.

Inode exhaustion is the storage failure that catches people out, because it's invisible right up until it isn't. Disk space you can feel filling up. Inodes just run out.

## What you're actually being charged for

An inode is a filesystem object: **a file, a directory, or a symlink**. Not bytes — *things*. A 4 KB `README` and a 40 GB ERA5 file each cost you exactly one.

Gadi applies an **iQuota** alongside the size quota, on both `/scratch` and `/g/data`. `lquota` shows both:

``` bash
------------------------------------------------------------------------
       fs       Usage    Quota    Limit   iUsage    iQuota    iLimit
------------------------------------------------------------------------
su28   scratch  438.76GB  1.0TB    2.0TB    204000    202000    404000   Over inode limit
su28   gdata     18.36TB  20.0TB  21.5TB   4453467   6000000   6300000
------------------------------------------------------------------------
```

Read the columns carefully, because there are two thresholds and they do different things:

* **`iQuota`** is your allocation. Cross it and a **one-week grace clock** starts.
* **`iLimit`** is typically 2× the quota — a hard ceiling you cannot exceed.

Stay over `iQuota` for a week and **jobs from that project go to `H` (held)** until you get back under. Same as exceeding your block quota. Nothing fails loudly; jobs simply stop starting, and if you weren't the one who blew the quota, the cause is not obvious from where you're sitting.

## Why NCI rations them at all

It isn't arbitrary bookkeeping. Every inode is an entry in Lustre's **metadata servers** — the shared namespace layer I wrote about in the `/scratch` vs `/g/data` post. A handful of MDS targets serve every user on the system, and each `open`, `stat`, or `readdir` is a round trip to them.

Millions of small files don't just consume your allocation; they degrade the filesystem **for everyone**. This is why requests for more inodes get more scrutiny than requests for more terabytes, and why "just ask NCI to raise it" is usually the wrong first move. Clean up first — you'll be asked whether you did.

## Finding the offender

The tools disagree with each other, and knowing why saves you an afternoon.

**`lquota`** — live, per-project totals for `/scratch` and `/g/data`. Start here.

**`nci_account -P ng72`** — shows `iGrant`, `iUsage`, `iAvail`. If `iAvail` goes negative, batch access is on hold.

**`nci-files-report`** — the good one. Breaks usage down *by user*, which is how you find who to talk to:

``` bash
nci-files-report --project ng72 --filesystem gdata
nci-files-report --group   ng72 --filesystem scratch

```

The `--project` / `--group` distinction matters more than it looks — see the next section.

**Important caveat:** `nci_account` and `nci-files-report` query an **accounting database populated by a periodic filesystem crawl**. They are as fresh as the last crawl. Delete a million files and the report may not notice for hours. `lquota` is live. If the two disagree, that's usually why — not a bug.

**Don't trust `du --inodes`.** It's convenient and it **under-reports**. A crude `find` is more honest:

``` bash
# file+dir count per top-level subdirectory
for d in */; do
  printf "%10d  %s\n" "$(find "$d" | wc -l)" "$d"
done | sort -rn

```

On a Lustre directory with millions of entries, that `find` is itself a metadata hammering — run it in a job, not on a login node, and expect it to take a while.

This is exactly the itch that led me to write **[`rudu`](https://zenodo.org/records/15347728)**, a parallel Rust replacement for `du` — it uses rayon to fan traversal across threads and reports true allocated blocks rather than apparent sizes, which makes walking a deep `/g/data` tree survivable rather than an overnight job.

## The gotcha nobody warns you about: group vs project quotas

Gadi runs **two different quota models depending on which filesystem your project sits on**:

| Model | Where | Charged to |
|----|----|----|
| **Project quota** | `/scratch`, `gdata5`, `gdata6` | The project owning the **directory** the file sits in |
| **Group quota** | `gdata1a`, `gdata1b`, `gdata4` | The **Unix group owner** of the file, wherever it lives |

On a group-quota filesystem, a file's location is irrelevant — its **group ownership** decides whose quota it hits. So someone can `chgrp` files, or write with the wrong primary group, and charge inodes to a project whose directories don't contain a single one of those files.

This is the classic "our numbers don't add up" case: `du` on your directories shows nothing like what `lquota` reports, because the files being charged to you are *somewhere else entirely*. `nci-files-report --group` finds them; `--project` won't.

Check which model applies to you with `nci_account` before you start hunting in the wrong place.

## The usual suspects

In rough order of how often they turn out to be the culprit:

**1. `conda`.** NCI names it explicitly, and it deserves the callout. A single environment can be 100,000+ files, mostly tiny. Half a dozen environments across a project and you've spent your entire allocation on Python packaging. Options: use `pip` and Gadi's pre-tuned modules; use the shared `hh5` analysis environments rather than rolling your own; or put the environment **inside a Singularity/Apptainer image**, which collapses the whole tree into one inode. That last one is the trick worth knowing — the container is a single file to Lustre.

**2. Untarred reference datasets.** Someone downloads an archive, extracts it "temporarily" to look at one file, and 400,000 inodes stay there for two years.

**3. Per-timestep model output.** One NetCDF per timestep per variable is the default of a lot of workflows and it scales exactly the wrong way. Concatenate along time.

**4. `.git` object stores.** A repo with a long history on `/g/data` is tens of thousands of small objects. `git gc --aggressive` helps; keeping code in `$HOME` (backed up, separate quota) helps more.

**5. Pip/Torch caches.** `~/.cache` grows without limit and nobody looks. Point `TORCH_HOME`, `PIP_CACHE_DIR` somewhere deliberate, or clear them.

**6. Dead job scratch.** Failed runs leave temp trees behind. The `/scratch` purge eventually gets them but **quarantined files still count against your inode quota** while they sit there, so purging doesn't relieve pressure as fast as you'd hope.

## Zarr is not automatically the answer

This one bites climate people specifically, and it's worth being blunt about because Zarr gets recommended as the fix for everything.

**Zarr solves file-internal metadata. It can make your inode problem dramatically worse.** Every chunk is a separate file on disk. Chunk an ERA5 array too finely and a single "file" becomes a store with hundreds of thousands of inodes.

The arithmetic is worth doing before you convert anything:

``` bash
inodes ≈ (array elements / elements per chunk) × number of variables

```

An hourly ERA5 field at 0.25° chunked as `(1, 721, 1440)` — one timestep per chunk — is **8,760 inodes per variable per year**. Forty years, ten variables: **3.5 million inodes** for one dataset. That's an entire project allocation, gone, in a conversion you ran because someone said Zarr was faster.

Mitigations, roughly in order of preference:

* **Chunk coarsely along time.** `(168, 721, 1440)` — a week per chunk — cuts inode count by 168× versus per-timestep. Aim for chunks in the tens-to-hundreds of MB, not single-digit MB.
* **Use consolidated metadata** so you're not paying an inode and an MDS round trip per `.zarray`.
* **Prefer Kerchunk/virtual Zarr over conversion.** Leave the NetCDF where it is and generate references — you get Zarr-style access with *zero* new data inodes.
* **If you do use Kerchunk, write Parquet references, not JSON.** A per-file JSON reference tree is itself an inode swarm — the exact problem you were solving. Parquet collapses it.
* **Consider a sharded layout** (Zarr v3) if you need fine chunks for read performance but can't afford the file count.

The general principle: **file count is a design parameter, not an implementation detail.** Decide it before you write 40 years of data, because fixing it afterwards means rewriting the whole store.

## Fixing it

**`tar` the cold stuff.** The blunt instrument, and it works:

``` bash
tar -cf archive.tar directory/ && rm -rf directory/

```

400,000 inodes → 1. Do it in a `copyq` job, not on a login node.

**Archive to `massdata`.** Anything you're keeping but not reading belongs on tape. But **tar first** — massdata is explicitly not designed for small files, and pushing thousands of them there gives terrible performance for everyone.

**Use `$PBS_JOBFS` for anything transient.** Files created on node-local NVMe **never touch your inode quota** — they don't exist on Lustre at all. If a job creates 50,000 temp files and keeps 10, do the work on jobfs and copy back only the keepers. This is the single highest-leverage habit change, and it makes your jobs faster for the same reason.

**Containerise environments.** One SIF file instead of 100,000 conda files (This is waht the conda envs in analysis3 do).

## Quick triage

``` bash
lquota                                        # who's over, live
nci_account -P <proj>                         # iGrant / iUsage / iAvail
nci-files-report --project <proj> --filesystem gdata   # by user, by location
nci-files-report --group   <proj> --filesystem gdata   # by group ownership

```

## tl;dr

* Over `iQuota`? The one-week clock is already running.
* Numbers don't reconcile? Check group-vs-project quota model, and crawl staleness.
* Hunt with `find | wc -l` or `rudu` — not `du --inodes`.
* Suspect conda first. Then untarred archives. Then `.git`.
* `tar` cold directories; push to `massdata` *after* tarring.
* Move transient file creation to `$PBS_JOBFS`.
* Before converting anything to Zarr, compute the chunk count.