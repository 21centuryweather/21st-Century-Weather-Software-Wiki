# `/scratch` vs `/g/data` vs `$PBS_JOBFS` on Gadi: An I/O Performance View

On Gadi, `/scratch` and `/g/data` are both Lustre filesystems and both project-scoped, but they sit on different hardware, tuned for different jobs. `$PBS_JOBFS`, however, is a node-local NVMe/SSD. If you care about how fast your reads and writes go (and on climate-scale data, you should), choosing the right one for a given access pattern can swing your I/O time by an order of magnitude. Here's the performance-oriented breakdown.

## The hardware underneath

The reason these two behave differently starts below the mount point:

- **`/scratch`** runs on NetApp enterprise arrays fronted by a **DDN Lustre** parallel filesystem, engineered for **up to \~900 GB/s aggregate throughput**. This is the tier NCI built to absorb the concurrent write load of thousands of running jobs.

- **`/g/data`** is also Lustre, but provisioned as the **large-capacity, shared persistent tier**. It's sized for holding petabytes across many projects, not for soaking up burst write traffic from compute nodes.

- **`$PBS_JOBFS`** (node-local NVMe/SSD) is the fastest storage you can touch for the right access pattern, it's local, so it sidesteps the InfiniBand fabric and the shared metadata servers entirely.

All of the shared filesystems hang off a 200 Gb/s HDR InfiniBand fabric in a Dragonfly+ topology, so network isn't usually your bottleneck, the storage backend and your access pattern are.

## Performance side-by-side

| | `$PBS_JOBFS` | `/scratch` | `/g/data`|
|--- | --- | --- | ---|
|**Backend** | Node-local NVMe/SSD | DDN Lustre (NetApp arrays) | Lustre (capacity tier) |
|**Peak aggregate** | Per-node, very fast | \~900 GB/s system-wide | Lower; not burst-tuned |
|**Best access pattern** | Small/random, metadata-heavy, temp files | Large sequential parallel I/O | Reading curated datasets |
|**Shared contention** | None (yours alone for the job) | High, but engineered for it | High, *\*not\** engineered for burst writes |
|**Visible after job?** | No — wiped at job end | Yes | Yes |
| **Write model output here?** | Stage here if metadata-heavy | **Yes — primary target** | Discouraged from compute nodes |
| |  |  | |

## Why `/scratch` is the write target

`/scratch` is where you should be writing model output and doing heavy parallel I/O during a run. It's not just policy, it's the only shared tier actually provisioned for that throughput. Writing large simulation output straight to `/g/data` from compute nodes fights for a filesystem that wasn't built for burst write concurrency, so you get more contention, more variance, and slower jobs, and you make `/g/data` slower for everyone else reading reference data at the same time.

The intended flow is: **write hot output to `/scratch` during the run → stage the keepers over to `/g/data` afterward** from an analysis or data-mover job. That keeps the high-bandwidth writes on the high-bandwidth filesystem.

## The real performance killer: small files and metadata

Gadi is excellent at large-scale parallel I/O and **significantly worse at frequent small operations**, because every `open`/`stat`/`create` is a round trip to a shared metadata server (MDS) that all users share. On climate workloads this is usually where time quietly disappears.

- **Consolidate** --- Millions of tiny files (per-timestep NetCDF, unpacked archives) hammer the MDS and burn your **inode quota**, which applies to both `/scratch` and `/g/data`. Bundle into `tar`, or use consolidated Zarr / well-chunked NetCDF instead of thousands of fragments.

- **`conda` is a classic offender** --- it creates enormous small-file trees. Prefer `pip` and the pre-tuned Gadi modules, or keep environments off the shared Lustre metadata path where you can.

- **Push metadata-heavy work to `$PBS_JOBFS`** --- If a job does lots of small/temp/random I/O, do it on node-local NVMe (request enough with `-l jobfs=…`, default is tiny) and copy only the final results back to `/scratch`. It's faster **and** it spares the shared MDS.

## A performance-first hierarchy

1. **`$PBS_JOBFS`** --- node-local NVMe. Fastest for small/random/metadata-heavy and scratch-during-job work. Gone at job end.

2. **`/scratch`** --- high-bandwidth DDN Lustre. Primary target for large parallel reads/writes and model output.

3\. **`/g/data/`** --- capacity Lustre. Read curated datasets from here; stage keepers here after a run. Don't use it as a burst-write scratchpad.

## tl:dr

- Writing output? → `/scratch`, not `/g/data`.

- Large sequential reads/writes? → keep them on `/scratch`, not `/g/data`.

- Lots of small/temp I/O? → do it on `$PBS_JOBFS`, request enough `jobfs`.

- Producing many small files? → consolidate (`tar/Zarr/chunked NetCDF`); watch inodes.

- Check quota and inode usage with `lquota` / `nci_account`.

## Questions & Answers

### Can you elaborate on what you mean by ‘metadata-heavy’ work? Typically I think of ‘meta data’ as the attributes in the netCDF file. Are there other ways this can be viewed?

Lustre splits every file operation across two server types:

* **MDS (Metadata Server)** — owns the *namespace*: filenames, directory tree, inodes, permissions, timestamps, file sizes, and which OSTs hold a file's bytes.
* **OSTs (Object Storage Targets)** — own the actual *bytes*.

So `open()`, `stat()`, `create()`, `unlink()`, `readdir()`, `rename()`, `chmod()` are MDS operations. They move essentially no data, but each is a network round trip to a server shared by every user on Gadi. There are hundreds of OSTs and only a handful of MDS targets, that asymmetry is why namespace ops are the contention point.

**"Metadata-heavy" = a high rate of namespace operations relative to bytes moved.** Useful mental metric: *bytes per file operation*.

* Reading one 33 GB ERA5 file is \~33 GB/open, trivially metadata-light. 
* Reading 200,000 tiny files is a few KB/open, metadata-bound, and no amount of backend bandwidth saves you.

Classic offenders, none of which look like I/O:

* `xr.open_mfdataset()` across thousands of files — every file gets opened and stat'd before a single byte of science data moves
* `conda` environments (100k+ small files); Python imports walking `sys.path`
* untarring, `git status` on a large repo, `ls` on a huge directory
* checkpoint schemes writing one small file per rank per timestep

---

### Do I have to do anything other than requesting enough jobfs to be making use of $PBS_JOBFS? and are there any downsides to just always requesting a large amount?

To access `$PBS_JOBFS` within python scripts and save a netcdf file there:

```python
import os
import xarray as xr

jobfs = os.environ.get("PBS_JOBFS", "/tmp")
out = os.path.join(jobfs, "intermediate.nc")

ds.to_netcdf(out)
```

All you need to do to get the `jobfs` is request in the pbs script or ARE settings.

No there are no downsides to requesting the full `jobfs` allocation. Each node has a set amount attached to it (each queue limits tell you the max amount), only when you have access to the node can you use it. So if you don't request any jobfs it just sits there doing nothing.