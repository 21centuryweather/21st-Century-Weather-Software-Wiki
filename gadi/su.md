# Where Your SUs Actually Go: Right-Sizing PBS Jobs on Gadi

Most people learn the Gadi cost model the same way: someone asks why the project burned a quarter's allocation in three weeks, and you go digging. The answer is almost never "we did too much science." It's usually a handful of jobs requesting resources they never used, and one specific mistake that can cost you **12× more SU than you expect** without a single warning.

Here's how charging actually works, and how to stop leaking allocation.

## The formula

Every job on Gadi is charged by the same equation:

``` bash
Job Cost (SU) = Queue Charge Rate × Resource Requested × Walltime Used (hours)
```

where the middle term is the part that surprises people:

``` bash
Resource Requested = Max(NCPUs Requested,
                         (Memory Requested / Max Memory Per Node) × Max NCPUs Per Node)
```

Three things follow immediately, and they're worth internalising before anything else:

1. **You're charged on what you *request*, not what you *use*.** Ask for 48 cores and use 4, you pay for 48. PBS reserves the resources; the scheduler doesn't care that your code was single-threaded.
2. **Walltime is the exception** — you're charged for walltime *used*, not requested. A job that asks for 10 hours and finishes in 2 is charged for 2. (The full 10 is *reserved* against your grant while it runs, so it still affects what else you can submit.)
3. **CPUs and memory don't add up — they compete.** You pay for whichever is larger. This is the whole game.

## The memory trap

That `Max()` is where allocations quietly die.

On the `normal` queue, a node has 48 cores and 192 GiB. So memory is charged as *the fraction of the node's RAM you took, expressed in core-equivalents*. Request a lot of memory with few cores, and you've made the rest of the node unusable for everyone else — so you're billed as if you'd taken it all.

Watch the same 8-core job get more expensive purely through the `mem` flag, over 5 hours on `normal`:

| `ncpus` | `mem` | Charged on | Cost |
|---|---|---|---|
| 8 | 16 GB | CPUs — `Max(8, 4)` | **80 SU** |
| 8 | 128 GB | Memory — `Max(8, 32)` | **320 SU** |
| 8 | 190 GB | Memory — `Max(8, 47.5)` | **475 SU** |

Same cores. Same walltime. **Nearly 6× the cost**, entirely from memory you may not have needed. The 190 GB case is the pathological one: you asked for 8 cores and got billed for 47.5, because 190 of 192 GiB is effectively the whole node.

**The breakeven is your node's memory-per-core ratio.** On `normal` that's 4 GB/core (192 ÷ 48). Stay at or under `4 × ncpus` GB and CPUs dominate your bill; go above it and memory takes over. On `normalsr` (Sapphire Rapids) the ratio is ~4.8 GB/core (500 ÷ 104); on `normalbw` it's ~4.6 (128 ÷ 28).

So: **`mem=190GB` is not a safe default.** It's the single most expensive thing you can casually type into a PBS script. If your job needs 30 GB, ask for 30 GB.

## Know your queue's charge rate

Rates vary by more than 4× across CPU queues. Current rates (per resource-hour):

| Queue | Hardware | Charge rate | Cores/node | Mem/node | Mem/core |
|---|---|---|---|---|---|
| `normalbw` | Broadwell | **1.25 SU** | 28 | 128 / 256 GB | ~4.6 GB |
| `normalsl` | Skylake | **1.5 SU** | 32 | 192 GB | 6 GB |
| `normal` | Cascade Lake | **2 SU** | 48 | 190 GB | ~4 GB |
| `normalsr` | Sapphire Rapids | **2 SU** | 104 | 500 GB | ~4.8 GB |
| `hugemem` | Cascade Lake | **3 SU** | 48 | 1470 GB | ~30 GB |
| `hugemembw` | Broadwell | **1.25 SU** | 28 | 1020 GB | ~36 GB |
| `megamem` | Cascade Lake | **5 SU** | 48 | 2990 GB | ~62 GB |
| `express` | Cascade Lake | **6 SU** | 48 | 190 GB | ~4 GB |
| `gpuvolta` | V100 | **3 SU** | 12/GPU | 382 GB | ~8 GB |
| `dgxa100` | A100 | **4.5 SU** | 16/GPU | 2000 GB | ~15 GB |
| `gpuhopper` | H200 | **7.5 SU** | 12/GPU | 1024 GB | ~21 GB |

A few consequences worth acting on:

**`express` costs 3× `normal`.** It buys queue priority, not speed — identical hardware. If you're not genuinely blocked waiting, you're paying triple for nothing. Debug on `express` with tiny jobs; run production on `normal`.

**`normalbw` is 1.25 SU vs `normal` at 2 SU.** Broadwell is older and slower per core, but at 62% of the price. If your job isn't wall-clock-critical and doesn't scale beautifully, benchmark it — for a lot of embarrassingly parallel analysis work, the older nodes are the cheaper answer.

**`hugemembw` (1.25 SU) vs `hugemem` (3 SU)** is the sleeper. Need 400 GB for a modest core count? `hugemembw` gives you *more* memory per core (~36 vs ~30 GB) at **less than half the charge rate**. Slower silicon, but if you're memory-bound rather than compute-bound — which describes a lot of large-array climate analysis — that's the trade you want. The catch: cores come in multiples of 7 on `hugemembw`.

**Don't reach for `hugemem` reflexively.** A memory-hungry job on `normal` charged at `Max(ncpus, mem_proportion)` may still beat a `hugemem` job at 3 SU. Do the arithmetic both ways.

## Whole nodes: the rounding tax

Any job larger than one node **must request CPUs in multiples of the full node**. On `normal` that's multiples of 48; on `normalsr`, 104; on `normalbw`, 28.

So a job asking for 20,000 cores on `normal` actually requests **20,016** — you round up to 417 nodes and pay for all of it. If your decomposition lands at 50 cores, you're paying for 96 and wasting 46. Choose core counts that land on node boundaries, or shrink to fit one node.

This is also why `ncpus=1, mem=190GB` is such a trap: you didn't request a node, but you priced one.

## Walltime: request generously, but not absurdly

Since you're billed on walltime *used*, over-requesting walltime doesn't directly cost SU. But it isn't free either:

- The **full requested walltime is reserved** against your grant while the job runs, cutting what else you can submit. `nci_account` shows this under `Reserved`.
- **Shorter walltime requests schedule sooner.** PBS backfills — a 2-hour job slots into gaps a 48-hour job can't.
- Walltime limits **tighten as core count rises**: on `normal`, 48 hours up to 672 cores, but only 5 hours at 3024+ cores. Scaling up shortens your ceiling.
- If the job **doesn't have enough SU available** for the requested walltime at submission, it goes straight to `H` (held). That's a `nci_account` problem, not a code problem.

Rule of thumb: request ~1.3× your benchmarked runtime. Enough headroom to survive a slow node, not so much that you never get scheduled.

## Measure what you actually used

The `.o` file at job end tells you everything. Read it every time:

``` bash
   Service Units: 96.00
   NCPUs Requested: 48          NCPUs Used: 48
   CPU Time Used: 09:12:33
   Memory Requested: 190.0GB    Memory Used: 31.4GB      <-- !!
   Walltime requested: 02:00:00 Walltime Used: 01:00:12
   JobFS requested: 100.0MB     JobFS used: 0B
```

That job paid 190 GB rates and touched 31 GB. Requesting `mem=40GB` would have dropped it from ~96 SU to ~48 SU — half, for a one-line change.

The diagnostic to compute is **CPU efficiency**:

``` bash
efficiency = CPU Time Used / (NCPUs Used × Walltime Used)
```

Here: `9:12:33 / (48 × 1:00:12)` ≈ **19%**. You're paying for 48 cores and using the equivalent of about 9. Something is serial, I/O-blocked, or badly threaded. NCI's guidance is to target the sweet spot where you're **using at least 80%** of what you reserved.

Check your balance before submitting a campaign:

```bash
nci_account -P gb02          # grant, used, reserved, available
nqstat -P gb02               # what's running and queued
```

## The five most expensive habits

1. **`mem=190GB` by default.** Costs up to 6× on identical work. Set memory to your measured high-water mark plus ~20%.
2. **`express` for production runs.** 3× the rate for priority you don't need.
3. **Requesting cores a serial script can't use.** 48 cores at 19% efficiency is 39 cores of pure waste. Profile first.
4. **Ignoring `normalbw` / `hugemembw`.** Often 40–60% cheaper for work that isn't clock-critical.
5. **Never reading the `.o` file.** Every job hands you a free audit and most people delete it unread.

## A worked right-sizing

Before — a Dask analysis job someone wrote in a hurry:

```bash
#PBS -q express
#PBS -l ncpus=48
#PBS -l mem=190GB
#PBS -l walltime=10:00:00
#PBS -l storage=gdata/gb02+scratch/gb02
```

Ran in 3 hours, peaked at 45 GB, and the `.o` file showed 12 of 48 cores meaningfully busy (Lustre metadata was the bottleneck, not CPU).

Cost: `6 SU × Max(48, 47.5) × 3h` = **864 SU**.

After:

```bash
#PBS -q normal
#PBS -l ncpus=12
#PBS -l mem=48GB
#PBS -l jobfs=100GB
#PBS -l walltime=4:00:00
#PBS -l storage=gdata/gb02+scratch/gb02
```

Cost: `2 SU × Max(12, 12) × 3h` = **72 SU**.

**12× cheaper, same science, same wall-clock.** Nothing exotic — just matching the request to the measurement. (The `jobfs` line is separate from cost: it's not charged, but the default is only **100 MB**, so any real temp I/O needs it declared. Keeping spill on node-local NVMe is what let the core count drop honestly.)

## Checklist before you submit a campaign

- [ ] Ran one job first and read the `.o` file — cores, memory, walltime *used*.
- [ ] `mem` ≤ `4 × ncpus` GB on `normal`? If not, is the memory genuinely needed?
- [ ] CPU efficiency ≥ 80%? If not, cut `ncpus` rather than hoping.
- [ ] Checked whether `normalbw` / `hugemembw` does the job at 1.25 SU.
- [ ] On `express` only for genuine debugging.
- [ ] Multi-node core count lands on a node boundary.
- [ ] Walltime ≈ 1.3× benchmark, not a round 48.
- [ ] `jobfs` requested if you use more than 100 MB of temp space.
- [ ] `nci_account -P <proj>` has room before you queue 500 jobs.

---
