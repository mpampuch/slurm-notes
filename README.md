# slurm-notes

## What is Slurm?

Slurm (Simple Linux Utility for Resource Management) is an open-source workload manager and job scheduler commonly used in high-performance computing (HPC) clusters. It is widely used in research labs, universities, and supercomputers. It helps allocate compute resources (CPUs, GPUs, memory, nodes), schedule and queue jobs, manage multiple users fairly and monitor running jobs.

## Core Slurm Commands

### 1. `sbatch` — Submit batch jobs

Used to submit a job script to the scheduler.

```Bash
sbatch job.sh
```

#### What it does:

* Sends your script to Slurm
* Runs it later when resources are available
* Returns a **Job ID**

#### Example script:

```Bash
#!/usr/bin/env bash
#SBATCH --time=00:30:00                     # Set your job walltime (D-HH:MM:SS)
#SBATCH --nodes=1                           # Set how many compute nodes / machines to request
#SBATCH --cpus-per-task=1                   # Set how many CPU's / threads you need
#SBATCH --ntasks=1                          # Set how many tasks / processes you need (generally 1 for serial tasks)
#SBATCH --mem=6G                            # Set how much memory should be allocated per node
#SBATCH --partition=batch                   # Set which partition / queue your job will run in.
#SBATCH --output=SLURM_LOGS/.slurm_%j.out   # Set where SLURM stdout will go
#SBATCH --error=SLURM_LOGS/.slurm_%j.err    # Set where SLURM stderr will go
#SBATCH --mail-type=ALL                     # Set emailing rules for this job (options: BEGIN, END, FAIL, REQUEUE, ALL, NONE)
#SBATCH --job-name=JOB-NAME                 # Set your job name

python script.py
```


### 2. `srun` — Run interactive or parallel jobs

Used to run jobs directly on allocated resources.

> [!NOTE] 
> You need to specifiy the `--time=` parameter for this (at least on the KAUST IBEX)

```Bash
srun --time=08:00:00 --mem=128Gb --cpus-per-task=16 --pty bash
```

#### What it does:

* Launches tasks immediately
* Can be used inside `sbatch` scripts
* Common for MPI / parallel execution

###  Attach interactive shell to existing allocation

```Bash
srun --jobid=46876067 --overlap --pty bash
```

#### What it does:

* `--jobid=46876067`: attaches to an already running Slurm job allocation
* `--overlap`: allows multiple `srun` steps to share the same allocated resources
* `--pty bash`: opens an interactive Bash shell on the compute node

> [!NOTE] 
> This is effectively the same as ssh-ing into a compute node.


### 3. `sinfo` — View cluster status

Shows available nodes and partitions.

```Bash
sinfo
```

#### What it shows:

* Which partitions exist (GPU, CPU, debug, etc.)
* Node availability
* Free vs busy resources

#### Cluster dashboard

The following command 

```bash
sinfo -o "%16n %9m %8z %4R %6a %10A %10O %14l %14L %30f %10G %50E"
``` 

gives you this output:

```
nodes
HOSTNAMES        MEMORY    S:C:T    PART AVAIL  NODES(A/I) CPU_LOAD   TIMELIMIT      DEFAULTTIME    AVAIL_FEATURES                 GRES       REASON                                            
cn604-10         730112    2:96:1   batc up     1/0        29.16      14-00:00:00    n/a            amd,cpu_amd_epyc_9655,el9,ibex (null)     none                                              
cn604-17         730112    2:96:1   batc up     1/0        158.10     14-00:00:00    n/a            amd,cpu_amd_epyc_9655,el9,ibex (null)     none                                              
cn511-16         730112    2:96:1   batc up     0/0        0.00       14-00:00:00    n/a            amd,cpu_amd_epyc_9655,el9,ibex (null)     NHC: Watchdog timer unable to terminate hung NHC p
cn504-16         730112    2:96:1   batc up     1/0        40.09      14-00:00:00    n/a            amd,cpu_amd_epyc_9655,el9,ibex (null)     NHC: [ICINGA] CRITICAL: ping4                     
cn605-20-l       336896    2:20:1   batc up     1/0        33.32      14-00:00:00    n/a            cpu_intel_gold_6148,el9,ibex20 (null)     none                                              
cn506-02-r       467968    2:64:1   batc up     0/1        0.32       14-00:00:00    n/a            amd,cpu_amd_epyc_7702,el9,ibex (null)     none                                              
cn603-29-l       336896    2:20:1   batc up     0/1        35.78      14-00:00:00    n/a            cpu_intel_gold_6148,el9,ibex20 (null)     none                                              
cn113-35-l       467968    2:64:1   c203 up     1/0        99.77      140-00:00:00   n/a            amd,cpu_amd_epyc_7702,el9,hwpe (null)     none                                              
bcl603-03-r      336896    2:20:1   c203 up     0/1        0.36       140-00:00:00   n/a            cpu_intel_gold_6148,el9,ibex20 (null)     none                                              
gpu510-32        336896    2:20:1   debu up     1/0        2.39       2:00:00        n/a            cpu_intel_gold_6248,el9,gpu,gp gpu:v100:2 none                                              
dgpu609-14       205824    2:18:1   debu up     0/1        2.21       2:00:00        n/a            cpu_intel_e5_2699_v3,el9,gpu,i gpu:p6000: none                                              
dgpu501-10       205824    2:18:1   gpu  up     1/0        6.18       14-00:00:00    n/a            cpu_intel_e5_2699_v3,el9,gpu,g gpu:gtx_10 none                                              
gpu502-06        336896    2:16:1   gpu  up     1/0        10.84      14-00:00:00    n/a            cpu_intel_gold_6142,el9,gpu,gp gpu:gtx_10 none                                              
gpu101-02-l      467968    1:64:1   gpu2 up     1/0        4.34       1-00:00:00     n/a            4gpus,a100,amd,cpu_amd_epyc_77 gpu:a100:4 none                                              
gpu214-18        730112    2:24:1   gpu2 up     1/0        37.39      1-00:00:00     n/a            cpu_intel_platinum_8260,el9,gp gpu:v100:8 none                                              
dgpu501-18       205824    2:18:1   gpu4 up     1/0        6.36       4:00:00        n/a            cpu_intel_e5_2699_v3,el9,gpu,g gpu:gtx_10 none                                              
gpu502-06        336896    2:16:1   gpu4 up     1/0        10.84      4:00:00        n/a            cpu_intel_gold_6142,el9,gpu,gp gpu:gtx_10 none                                              
gpu109-02-l      467968    1:64:1   gpu7 up     1/0        6.22       3-00:00:00     n/a            4gpus,a100,amd,cpu_amd_epyc_77 gpu:a100:4 none                                              
gpu109-02-r      467968    1:64:1   gpu7 up     1/0        3.68       3-00:00:00     n/a            4gpus,a100,amd,cpu_amd_epyc_77 gpu:a100:4 none                                              
gpu212-06        730112    2:24:1   gpu_ up     1/0        24.94      14-00:00:00    n/a            cpu_intel_platinum_8260,el9,gp gpu:v100:8 none                                              
gpu102-02        992256    2:64:1   gpu_ up     1/0        4.73       1-00:00:00     n/a            8gpus,a100,amd,cpu_amd_epyc_77 gpu:a100:8 none                                              
gpu212-06        730112    2:24:1   gpu_ up     1/0        24.94      1-00:00:00     n/a            cpu_intel_platinum_8260,el9,gp gpu:v100:8 none                                              
gpu102-02        992256    2:64:1   gpu_ up     1/0        4.73       3-00:00:00     n/a            8gpus,a100,amd,cpu_amd_epyc_77 gpu:a100:8 none                                              
gpu102-09        992256    2:64:1   gpu_ up     1/0        8.94       3-00:00:00     n/a            8gpus,a100,amd,cpu_amd_epyc_77 gpu:a100:8 none                                              
lm602-04         3069952   4:8:1    larg up     1/0        16.29      14-00:00:00    n/a            cpu_intel_xeon_gold_6134m,el9, (null)     none                                              
lm611-18         16438272  4:32:1   larg up     1/0        112.34     14-00:00:00    n/a            cpu_intel_xeon_gold_6448h,el9, (null)     none                                              
lm508-18         3018752   4:12:1   larg up     1/0        48.22      14-00:00:00    n/a            cascadelake,cpu_intel_xeon_gol (null)     none                                              
lm508-02         3018752   4:12:1   larg up     0/1        0.20       14-00:00:00    n/a            cascadelake,cpu_intel_xeon_gol (null)     none                                              
lm508-04         3018752   4:12:1   larg up     0/1        0.32       14-00:00:00    n/a            cascadelake,cpu_intel_xeon_gol (null)     none                                              
lm508-06         3018752   4:12:1   larg up     0/1        0.25       14-00:00:00    n/a            cascadelake,cpu_intel_xeon_gol (null)     none                                             
```

This is a **SLURM cluster node inventory**.

It shows:

* all compute machines in the cluster
* their hardware (CPU, RAM, GPU)
* their current load
* whether they are usable
* and any hardware/health problems

#### What each part means

##### 1. Nodes (HOSTNAMES)

These are machine names like:

* `cn604-10` → CPU compute node
* `gpu102-02` → GPU compute node
* `lm611-18` → large-memory node

Prefixes usually mean:

* `cn` → CPU nodes
* `gpu` / `dgpu` → GPU nodes
* `lm` → large memory nodes

##### 2. Memory (MEMORY)

Total RAM available on that node.

Examples:

* `205824` ≈ 200 GB
* `730112` ≈ 730 GB
* `992256` ≈ ~1 TB
* `16438272` ≈ ~16 TB (huge node)

##### 3. CPU layout (S:C:T)

Example: `2:64:1`

Meaning:

* **S** = sockets (CPU chips on the server)
* **C** = cores per socket
* **T** = threads per core (hyperthreading)

So `2:64:1` means:

* 2 CPUs
* 64 cores each
* no hyperthreading

##### 4. Partition (PART)

This is the **job queue type**.

Examples:

* `batc` → batch CPU jobs (long running)
* `gpu`, `gpu2`, `gpu7` → GPU queues
* `debu` → short debug jobs
* `larg` → large-memory jobs

So this controls what kind of jobs are allowed on the node.

##### 5. Status (AVAIL)

* `up` = node is working
* (not shown here but common: `down`, `drain`) = unusable

So all shown nodes are technically online.

##### 6. Jobs (NODES A/I)

Format: `active/idle`

Examples:

* `1/0` → one job running
* `0/1` → idle, free
* `0/0` → no jobs or special state

This tells you whether the node is busy or available

##### 7. CPU_LOAD

This is current usage pressure.

Interpretation:

* `0–5` → idle
* `5–30` → moderate load
* `30–100+` → heavy load
* `100+` → extremely busy / oversubscribed

Example:

* `cn604-17 → 158.10` = very overloaded

##### 8. TIMELIMIT

Maximum job runtime allowed.

Examples:

* `14-00:00:00` → 14 days
* `1-00:00:00` → 1 day
* `2:00:00` → 2 hours (debug nodes)

##### 9. Features (AVAIL_FEATURES)

Tags describing hardware/software:

Examples:

* `amd`, `intel` → CPU vendor
* `cpu_amd_epyc_9655` → exact CPU model
* `el9` → OS version (RHEL/Rocky Linux 9)
* `ibex` → cluster network/config label

This is used for scheduling like “run only on AMD nodes” or “needs EL9”

##### 10. GRES (GPU resources)

This is the most important for GPU nodes.

Examples:

* `gpu:a100:8` → 8 NVIDIA A100 GPUs
* `gpu:v100:2` → 2 V100 GPUs
* `gpu:gtx_10` → GTX 10-series GPUs
* `(null)` → no GPU

This tells you what hardware accelerators exist on the node

##### 11. REASON (problems)

If not `none`, something is wrong.

Examples in your output:

###### Health check failure

* `NHC: Watchdog timer...`
* Node health check process is stuck

###### Network failure

* `ICINGA: CRITICAL: ping4`
* Node failed ping → may be unreachable

So these nodes may be unreliable even if marked "up"


### 4. `squeue` — View running & pending jobs

(Not in your list, but essential)

```Bash
squeue
```

#### What it shows:

* Jobs currently running
* Jobs waiting in queue
* Job IDs, users, status


### 5. `scontrol` — Inspect or modify jobs (low-level)

Used for detailed job and cluster control.

```Bash
scontrol show job 47018064
```

#### What it does:

* Shows full job configuration
* Node allocation details
* Timers, dependencies, constraints

#### Advanced use:

* Update job parameters (admins mostly)


### 6. `sacct` — Job accounting history

Shows completed job statistics.

```Bash
sacct -j 47018064
```

Example output:

```
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
47018064     iqtree3_b+      batch pi-lauersk         24  COMPLETED      0:0 
47018064.ba+      batch            pi-lauersk         24  COMPLETED      0:0 
47018064.ex+     extern            pi-lauersk         24  COMPLETED      0:0 
```

#### What it shows:

* Job status (COMPLETED, FAILED, etc.)
* CPU usage
* Exit codes
* Job steps (`.batch`, `.extern`)

#### Important:

* Works after job finishes
* Uses Slurm accounting database


### 7. `seff` — Efficiency report (user-friendly)

(Not core Slurm, but very common on clusters)

```Bash
seff 47018064
```

Example output:

```
Job ID: 47018064
Cluster: dragon
User/Group: pampum/g-pampum
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 24
CPU Utilized: 02:58:30
CPU Efficiency: 96.17% of 03:05:36 core-walltime
Job Wall-clock time: 00:07:44
Memory Utilized: 305.98 MB
Memory Efficiency: 0.47% of 64.00 GB (64.00 GB/node)
```

#### What it shows:

* CPU efficiency
* Memory usage
* Wall time vs CPU time

#### Example insight:

* Helps detect over-requested memory or idle CPUs

## Understanding Pending Jobs & Fairshare

One of the most common situations is seeing jobs like:

```text
JOBID      PARTITION NAME      USER    ST  TIME  NODES  NODELIST(REASON)
47924278   batch     my-job    user    PD  0:00      1  (Priority)
```

### Pending (`PD`) does **not** mean stuck

A job with

```text
ST = PD
REASON = (Priority)
```

means:

- Slurm has successfully accepted your job.
- Your job is waiting in the scheduler queue.
- There is **no error** with your job.

It **does not** mean:

- the cluster is broken
- your job script failed
- the scheduler is hung

Usually it simply means other queued jobs currently have a higher scheduling priority.

---

## Why a job stays in `(Priority)`

Slurm schedules jobs using multiple priority components, including:

- **Fairshare** – how much of the cluster you (or your project) have used recently.
- **Age** – jobs gain priority the longer they wait.
- **Job size** – some clusters reward or penalize very large jobs.
- **Partition/QoS** – some partitions or QoS levels receive higher priority.

You can inspect the priority calculation with:

```bash
sprio -j <jobid>
```

Example:

```text
JOBID       PRIORITY

AGE         0
FAIRSHARE   3746
JOBSIZE     1
PARTITION   0
QOS         0
SITE        0
```

Interpretation:

- `AGE = 0` → newly submitted job
- `FAIRSHARE = 3746` → all current priority comes from fairshare
- `JOBSIZE = 1` → little/no effect
- `PARTITION/QOS = 0` → no additional bonuses

---

## What is Fairshare?

Fairshare prevents one user or project from monopolizing the cluster.

Users who have consumed lots of resources recently receive lower scheduling priority, while users who have used fewer resources receive higher priority.

Fairshare gradually recovers over time as older usage expires.

View your fairshare:

```bash
sshare -u $USER
```

or

```bash
sshare -l -u $USER
```

Example:

```text
Account         User     RawUsage  EffectvUsage  FairShare
pi-lauersk      pampum   1836601      1.000000    0.374585
```

Meaning:

| Column | Meaning |
|---------|---------|
| `RawUsage` | Historical resource consumption |
| `EffectvUsage` | Relative usage within your account |
| `FairShare` | Fairshare factor (closer to **1.0** is better) |

A fairshare around **0.3–0.5** is fairly normal and is **not** by itself enough to explain very long queue times.

---

## Why does `sprio` show 3746?

Many Slurm installations convert the fairshare factor into an integer priority score.

For example:

```text
FairShare = 0.374585
```

becomes approximately

```text
FAIRSHARE = 3746
```

in `sprio`.

The absolute number is not important—it only matters relative to other queued jobs.

---

## "Idle" nodes may not actually be available

A node showing a low CPU load does **not** necessarily mean your job can start.

Slurm schedules based on allocated resources, not instantaneous CPU usage.

A node may still be unavailable because:

- CPUs are already allocated
- insufficient memory is free
- GPUs are allocated
- another job has a reservation
- backfill scheduling cannot fit your requested walltime

---

## Useful commands when jobs are pending

### Check priority

```bash
sprio -j <jobid>
```

Shows how your scheduling priority is computed.

---

### Inspect the job

```bash
scontrol show job <jobid>
```

Useful fields:

- `Reason`
- `ReqTRES`
- `NumCPUs`
- `MinMemory`
- `TimeLimit`

These reveal what resources Slurm is trying to satisfy.

---

### Estimate start time

```bash
squeue --start -j <jobid>
```

If Slurm can predict a start time, it will display one.

If it shows `N/A`, the scheduler cannot yet estimate when enough resources will become available.

---

## Interpreting cluster status

A healthy cluster can still have many pending jobs.

For example:

- Nodes with low CPU load are **not necessarily free**.
- Some nodes may be reserved.
- Other users may have higher scheduling priority.
- Your job may simply be waiting its turn.

Seeing

```text
Reason=(Priority)
```

is generally a sign that the scheduler is functioning normally rather than that something is wrong.
