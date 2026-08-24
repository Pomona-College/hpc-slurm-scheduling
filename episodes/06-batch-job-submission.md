---
title: "Batch Job Submission"
teaching: 20
exercises: 15
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I submit a batch job?
- How do I check job status and progress?
- Where are my job results, and how do I follow them in real time?
- How do I get email notifications without spamming myself?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Submit jobs with `sbatch` and verify submission
- Read SLURM status codes and find the cause of pending jobs
- Locate and view output files during and after a run
- Configure email notifications appropriately for the job size
- Recognize and avoid the most common submission mistakes

:::::::::::::::::::::::::::::::::::::::::::::::

## Why Batch Submission Is the Right Default

The shell prompt on the Sagehen login node is convenient but not where your real work belongs. Login nodes are shared and intentionally constrained: a long calculation on the login node steals resources from every other user trying to edit, transfer, or schedule. Batch submission moves your work to the compute partition where SLURM can give it dedicated CPU, memory, and time, and where your job survives if you close your laptop.

The pattern is consistent: you write a short shell script that includes both SLURM directives (lines starting with `#SBATCH`) and the commands to run, then submit it with `sbatch`. SLURM puts the job in queue, runs it when resources are available, and writes its stdout and stderr to files in your current directory.

## Submitting with sbatch

```bash
sbatch my_script.sh
```

SLURM responds immediately:
```
Submitted batch job 12345
```

Your job ID is **12345**. Use this number to track and manage the job. The submitting shell can close, your laptop can sleep, you can disconnect from the VPN, and the job continues to run.

## Checking Job Status

```bash
# All your jobs
squeue -u $USER

# A specific job
squeue -j 12345

# Estimated start time for pending jobs
squeue -u $USER --start
```

Output:
```
     JOBID PARTITION     NAME     USER ST       TIME  NODES CPUS
     12345       amd analysis   alice  R       0:23      1    4
```

Status codes:

- `R` Running: actively executing
- `PD` Pending: waiting for resources, see the REASON field for why
- `CG` Completing: cleanup phase, almost done
- `CD` Completed: finished successfully
- `F` Failed: ended with non-zero exit code
- `CA` Cancelled: stopped via scancel
- `TO` Timeout: exceeded `--time` limit and was killed

::::::::::::::::::::::::::::::::::::::: callout

## Decoding "why is my job pending?"

`squeue -u $USER` shows a `(REASON)` column for pending jobs:

`(Resources)`: waiting for matching nodes to become available. Normal for busy partitions; just wait.

`(Priority)`: jobs with higher priority are ahead. This is fair-share doing its job.

`(QOSMaxJobsPerUserLimit)`: you have hit a per-user concurrency cap. Wait for one of yours to finish.

`(ReqNodeNotAvail)`: the constraint you set (e.g., a specific GPU type) currently has no available node. May take a long time.

`(Dependency)`: job is waiting for another job to complete (`--dependency=afterok:JOBID`).

::::::::::::::::::::::::::::::::::::::::::::::::

## Output Files

By default, SLURM writes a single combined log to `slurm-JOBID.out` in the directory where you submitted. For most workflows, separating stdout and stderr is more useful:

```bash
#SBATCH --output=analysis_%j.out   # stdout
#SBATCH --error=analysis_%j.err    # stderr
```

The `%j` placeholder is replaced by the job ID. Other useful placeholders: `%x` for job name, `%a` for array task ID, `%N` for node name. A common pattern that puts logs in their own folder:

```bash
#SBATCH --output=logs/%x_%j.out
#SBATCH --error=logs/%x_%j.err
```

Make sure `logs/` exists before submitting. SLURM will fail to start the job if the output path is invalid.

### Viewing Output

```bash
# While running (real-time)
tail -f analysis_12345.out

# After completion
cat analysis_12345.out

# Last 50 lines (useful for long jobs)
tail -50 analysis_12345.out

# Search for a specific string
grep -i error analysis_12345.out
```

You can also specify a full path so logs land somewhere predictable:

```bash
#SBATCH --output=/rhome/alice/logs/job_%j.out
```

## Email Notifications

```bash
#SBATCH --mail-user=alice@pomona.edu
#SBATCH --mail-type=END        # When finished
#SBATCH --mail-type=FAIL       # On failure
#SBATCH --mail-type=ALL        # Any state change (verbose)
```

::::::::::::::::::::::::::::::::::::::: callout

## Email Best Practices

For testing and short jobs: no email at all. Watch with `squeue` instead.

For medium jobs (hours): `FAIL` only. You will be notified about problems but not flooded with completion notices.

For long important jobs (days): `END,FAIL`. You want to know either way, but only once.

For job arrays: `FAIL` only. Otherwise a 100-task array sends 100 completion emails.

The `BEGIN` mail-type exists but is almost never useful: by the time the email arrives, your job has already started.

:::::::::::::::::::::::::::::::::::::::::::::::

## Changing Directory

By default, scripts run in the directory where you submitted. If your submitted script is in `/rhome/alice/scripts/` but your data is in `/rhome/alice/project/`, the script's relative paths will break.

Two solutions, in order of preference:

```bash
# Option A: cd at the top of the script (clearest)
cd /rhome/alice/project
python analysis.py

# Option B: use absolute paths everywhere (works but verbose)
python /rhome/alice/scripts/analysis.py /rhome/alice/project/data.csv
```

For jobs that read or write a lot, `cd` to a directory under `/scratch` instead of `/rhome` (covered in Workshop 12).

## Common Submission Mistakes

A few mistakes show up over and over in real submissions:

Forgetting `--time`. The default is short. A long job submitted without an explicit time limit will be killed mid-run.

Setting `--mem` to the maximum because you do not know what the job needs. This makes the job harder to schedule. Run a short test first with `seff JOBID` to right-size memory.

Mixing tabs and spaces in the SLURM directives. Some directives parse silently incorrectly. Stick to spaces.

Forgetting to load required modules inside the script. The login-node environment does not carry over. Always include `module load miniconda3` (or whatever you need) at the top of the script.

Submitting from `/scratch`. Output files go to the submission directory. Submit from `/rhome` so logs persist after the job's scratch is cleaned.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Submit and Monitor a Batch Job

1. Create `analysis.py`:
   ```python
   import time
   print("Starting analysis...")
   start = time.time()
   result = sum(range(10000000))
   print(f"Result: {result}")
   print(f"Completed in {time.time() - start:.2f}s")
   ```

2. Create `run_analysis.sh`:
   ```bash
   #!/bin/bash -l
   #SBATCH --job-name=python_test
   #SBATCH --cpus-per-task=1
   #SBATCH --mem=2G
   #SBATCH --time=00:05:00
   #SBATCH --partition=short
   #SBATCH --output=analysis_%j.out

   module load miniconda3
   python analysis.py
   ```

3. Submit: `sbatch run_analysis.sh`
4. Check status: `squeue -u $USER`
5. View output: `cat analysis_JOBID.out`
6. After completion, run `seff JOBID` to see actual CPU and memory usage.

:::::::::::::::::::::::: solution

## Solution

After submission, `squeue` shows the job briefly. The output file contains the computation result and timing. The `seff JOBID` output reveals you over-requested memory: a sum of 10 million integers needs about 50 MB, not 2 GB. On the next run, drop `--mem` to `512M`. This iteration is exactly how you tune SLURM scripts: submit, watch, measure with `seff`, adjust.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Read a Pending Job

A friend submitted a job 30 minutes ago and it is still pending. They run `squeue -u $USER` and see:

```
JOBID PARTITION     NAME      USER ST       TIME  NODES NODELIST(REASON)
54321       gpu  big_train   bob   PD       0:00      1 (Resources)
```

Explain in plain English what the REASON column means and one thing they could do.

:::::::::::::::::::::::: solution

## Solution

`(Resources)` means the GPU partition has no node currently available that matches the job's requirements. SLURM is just waiting for a GPU node to free up.

Two reasonable responses: wait it out (queues clear naturally), or run `squeue -p gpu --start` to see estimated start times for all pending GPU jobs and decide if it is worth waiting. If the job needs an A100 specifically, switching to `--gres=gpu:l40s:1` or `--gres=gpu:l40s:1` will likely start sooner because those GPUs are less oversubscribed.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Submit batch jobs with `sbatch script.sh`; the returned ID is how you track everything later
- Status codes: R (running), PD (pending), CG (completing), CD (done), F (failed), TO (timeout)
- The `(REASON)` column for pending jobs tells you why they are not running yet
- Output files use `--output` and `--error` directives; `%j` and `%x` substitute job ID and name
- Use `tail -f` to watch output in real-time while a job runs
- Email: `FAIL` for medium jobs, `END,FAIL` for long ones, nothing for tests, `FAIL` only for arrays
- Always run `seff JOBID` after completion to right-size future submissions
- You can log out after submission; jobs run unattended

::::::::::::::::::::::::::::::::::::::::::::::
