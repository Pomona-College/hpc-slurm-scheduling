---
title: "Managing and Canceling Jobs"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I cancel a running or queued job?
- What do I do when a job is stuck pending?
- How do I troubleshoot common job failures?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Use `scancel` to cancel jobs safely
- Diagnose why jobs are stuck in PENDING state
- Troubleshoot timeout, out-of-memory, and other common failures

:::::::::::::::::::::::::::::::::::::::::::::::

## Canceling Jobs with scancel

### Cancel a Single Job

```bash
scancel 12345
```

The job stops immediately. Output files written so far are preserved.

### Cancel Multiple Jobs

```bash
# Cancel all your jobs
scancel -u $USER

# Cancel by name
scancel --name=test

# Cancel only pending jobs
scancel --state=PENDING -u $USER

# Cancel by partition
scancel -u $USER --partition=amd
```

::::::::::::::::::::::::::::::::::::::: callout

## Be Careful with scancel

Cancellation is immediate and irreversible. Before canceling:
1. Verify the job ID: `squeue -j 12345`
2. Consider if partial results should be saved
3. Never use `scancel -u $USER` unless you intend to cancel everything

A safer approach:
```bash
# Preview what will be cancelled
squeue -u $USER --name=test_job
# Then cancel
scancel --name=test_job
```

:::::::::::::::::::::::::::::::::::::::::::::::

## Troubleshooting: Job Stuck in PENDING

Check the reason:
```bash
squeue -j JOBID -O jobid,reason
```

| Reason | Meaning | Solution |
|--------|---------|----------|
| Resources | No suitable node available | Reduce resource requests or wait |
| QOSMaxPerUser | Hit quota | Wait for other jobs to finish |
| ReqNodeNotAvail | Requested node is down | Remove `-w` flag; let SLURM choose |

## Troubleshooting: Job Failures

### TIMEOUT (exit code 138)

Job exceeded the `--time` limit.

```bash
seff JOBID | grep "Job Wall-clock time"
# Resubmit with more time
sbatch --time=08:00:00 script.sh
```

### OUT OF MEMORY

Output contains "Killed" or "Memory limit exceeded."

```bash
seff JOBID | grep "Memory"
# Resubmit with more RAM
sbatch --mem=64G script.sh
```

### FAILED (non-zero exit code)

Your code had an error, not a SLURM problem.

```bash
cat slurm-JOBID.out
cat slurm-JOBID.err
# Fix the error in your code, then resubmit
```

### "Module not found"

Ensure your shebang has the `-l` flag:
```bash
#!/bin/bash -l    # MUST have -l
```

Or explicitly source the module environment:
```bash
source /etc/profile
module load python/3.11
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Cancel Practice

1. Submit a long-running job:
   ```bash
   sbatch --time=01:00:00 -p amd --wrap="sleep 3600"
   ```
2. Verify it is running: `squeue -u $USER`
3. Cancel it: `scancel JOBID`
4. Verify cancellation: `squeue -u $USER`
5. Check history: `sacct -j JOBID --format=jobid,state,exitcode`

You should see STATE=CANCELLED.

:::::::::::::::::::::::: solution

## Solution

After canceling, `squeue` no longer shows the job. `sacct` shows STATE=CANCELLED with exit code 0:0, because cancellation is not a failure -- the job simply did not run to completion.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Use `scancel JOBID` to cancel a specific job; `scancel -u $USER` cancels all your jobs
- Always verify the job ID before canceling
- Pending jobs: check reason with `squeue -O reason`
- Timeout: increase `--time`; OOM: increase `--mem`
- Script errors show as FAILED with non-zero exit code; check output files
- Ensure `#!/bin/bash -l` so modules load correctly in batch scripts

::::::::::::::::::::::::::::::::::::::::::::::
