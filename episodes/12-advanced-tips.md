---
title: "Advanced Tips and Troubleshooting"
teaching: 10
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: questions

- How can I improve job reliability?
- How do I check my group's resource quota?
- What are common SLURM errors and how do I fix them?
- How can I optimize job performance?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Implement checkpoint/resume patterns for long jobs
- Check group quotas and resource usage
- Troubleshoot common SLURM errors systematically
- Apply performance optimization techniques

:::::::::::::::::::::::::::::::::::::::::::::::

## Improving Job Reliability

### Checkpoint/Resume Pattern

For long jobs, save progress so you can restart if interrupted:

```bash
#!/bin/bash -l
CHECKPOINT=/scratch/$SLURM_JOB_USER/$SLURM_JOB_ID/checkpoint.pkl

if [ -f "$CHECKPOINT" ]; then
    echo "Resuming from checkpoint..."
    python continue_job.py --checkpoint $CHECKPOINT
else
    echo "Starting new job..."
    python start_job.py --save_checkpoint $CHECKPOINT
fi
```

### Output Validation

Verify results before declaring success:

```bash
python train.py --output model.pkl

if [ ! -f "model.pkl" ]; then
    echo "ERROR: model.pkl not created!"
    exit 1
fi

echo "Validation passed!"
```

### Use set -e

Stop the script on the first error instead of continuing:

```bash
#!/bin/bash -l
set -e

python step1.py   # If this fails, step2 won't run
python step2.py
```

## Checking Quota and Resource Usage

```bash
# Check your group's storage and compute quota
quota_check.sh

# Count your running cores
squeue -u $USER --states=RUNNING --format=cpus --noheader | awk '{s+=$1} END {print s}'

# Historical usage (last 7 days)
sacct -u $USER --starttime=$(date -d '7 days ago' +%Y-%m-%d) \
  --format=jobid,jobname,state,elapsed,cpus
```

## Common Errors and Solutions

### "Job remains queued indefinitely"

```bash
squeue -j JOBID -O jobid,reason
```

- **Resources**: No node available -- reduce resource requests or wait
- **QOSMaxPerUser**: Hit quota -- wait for other jobs to finish
- **ReqNodeNotAvail**: Specific node is down -- remove `-w` flag

### "Permission denied" on Files

```bash
chmod +x script.sh              # Make script executable
ls -la /bigdata/lab/             # Check permissions
sbatch /rhome/$USER/script.sh   # Use absolute paths
```

### "Module not found"

Ensure shebang is `#!/bin/bash -l` (the `-l` flag initializes the module environment).

## Performance Optimization

### Email Notification Strategy

- **Testing**: No email
- **Medium jobs**: `--mail-type=FAIL`
- **Long important jobs**: `--mail-type=END,FAIL`
- **Job arrays**: `--mail-type=FAIL` only (avoid floods)

### NUMA Awareness (Large Memory Jobs)

```bash
#SBATCH --mem-bind=local
```

### CPU Binding (Multi-threaded Code)

```bash
#SBATCH --cpus-per-task=8
export OMP_PLACES=cores
export OMP_PROC_BIND=close
```

### Monitor Queue Competition

```bash
squeue --states=PENDING | wc -l
# If > 100, cluster is busy; consider off-peak submission
```

## Complete Production Job Template

```bash
#!/bin/bash -l
#SBATCH --job-name=production_pipeline
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --time=12:00:00
#SBATCH --partition=amd
#SBATCH --output=/rhome/$USER/logs/job_%j.out
#SBATCH --mail-user=user@pomona.edu
#SBATCH --mail-type=FAIL,END

set -e

echo "Job started at $(date) on $(hostname), ID: $SLURM_JOB_ID"

module load miniconda3
WORK_DIR=/scratch/$SLURM_JOB_USER/$SLURM_JOB_ID
mkdir -p $WORK_DIR && cd $WORK_DIR

cp /bigdata/lab/input.csv $WORK_DIR/
python /rhome/$USER/src/analyze.py --input input.csv --output results.pkl

cp results.pkl /rhome/$USER/results/results_${SLURM_JOB_ID}.pkl
echo "Job completed at $(date)"
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Scratch Storage Benchmark

Compare I/O speed between home and scratch storage:

```bash
#!/bin/bash -l
#SBATCH --job-name=io_bench
#SBATCH --cpus-per-task=1
#SBATCH --mem=4G
#SBATCH --time=00:10:00
#SBATCH --partition=short

WORK_DIR=/scratch/$SLURM_JOB_USER/$SLURM_JOB_ID
mkdir -p $WORK_DIR

dd if=/dev/urandom of=$WORK_DIR/test.bin bs=1M count=100 2>/dev/null

echo "Home storage:"
time dd if=$WORK_DIR/test.bin of=/rhome/$USER/test.tmp bs=1M count=100 2>/dev/null
rm /rhome/$USER/test.tmp

echo "Scratch storage:"
time dd if=$WORK_DIR/test.bin of=$WORK_DIR/test2.bin bs=1M count=100 2>/dev/null
```

Submit and compare the times.

:::::::::::::::::::::::: solution

## Solution

Scratch should be noticeably faster (often 2-10x) than home storage because it uses local SSDs instead of the network filesystem. This is why scratch is valuable for I/O-heavy jobs.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Use checkpoint/resume patterns for long jobs that might be interrupted
- Validate output files before declaring success
- Use `set -e` to stop scripts on the first error
- Check quota with `quota_check.sh` before large submissions
- Common fixes: timeout -> more time, OOM -> more RAM, pending -> smaller resources
- Use scratch for fast I/O; always copy results back to persistent storage
- Follow the production template: logging, error handling, scratch, validation

::::::::::::::::::::::::::::::::::::::::::::::
