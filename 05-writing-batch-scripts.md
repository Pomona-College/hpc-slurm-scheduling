---
title: "Writing Batch Scripts"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- What is a batch script and how is it structured?
- What are the essential SBATCH directives?
- How do I load modules and run code in batch scripts?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Write complete SLURM batch scripts
- Use SBATCH directives to request resources
- Load modules and configure environments in scripts
- Understand SLURM environment variables available in jobs

:::::::::::::::::::::::::::::::::::::::::::::::

## Why Batch Jobs?

Batch jobs run unattended while you do other things. They are the backbone of research computing:

- Run long jobs (hours, days, weeks)
- Submit many jobs at once
- Reproducible: exact commands saved in the script
- Run overnight or over weekends
- Get email notifications when done

## Anatomy of a Batch Script

```bash
#!/bin/bash -l
#SBATCH --job-name=my_job
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=00:30:00
#SBATCH --partition=amd
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err

module load miniconda3

python my_analysis.py
```

### The Shebang Line

```bash
#!/bin/bash -l
```

Always use `-l` so your `.bashrc` is read and modules are available.

### SBATCH Directives

| Directive | Meaning | Example |
|-----------|---------|---------|
| `--job-name` | Name shown in squeue | `--job-name=analysis_v1` |
| `--cpus-per-task` | CPU cores | `--cpus-per-task=8` |
| `--mem` | Total RAM | `--mem=32G` |
| `--time` | Max job time | `--time=04:00:00` |
| `--partition` | Partition | `--partition=amd` |
| `--output` | stdout file (`%j` = job ID) | `--output=job_%j.out` |
| `--error` | stderr file | `--error=job_%j.err` |
| `--gres` | GPUs | `--gres=gpu:2` |
| `--mail-user` | Email address | `--mail-user=me@pomona.edu` |
| `--mail-type` | When to email | `--mail-type=END,FAIL` |

### SLURM Environment Variables

SLURM sets useful variables inside your job:

| Variable | Meaning |
|----------|---------|
| `$SLURM_JOB_ID` | Job ID |
| `$SLURM_JOB_NAME` | Job name |
| `$SLURM_CPUS_PER_TASK` | Cores requested |
| `$SLURM_ARRAY_TASK_ID` | Array task ID |

## Example Batch Scripts

### Python Data Analysis

```bash
#!/bin/bash -l
#SBATCH --job-name=data_analysis
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=01:00:00
#SBATCH --partition=amd
#SBATCH --output=analysis_%j.out

module load miniconda3
python analysis.py
```

### R Statistical Analysis (short test run)

```bash
#!/bin/bash -l
#SBATCH --job-name=stats
#SBATCH --cpus-per-task=2
#SBATCH --mem=8G
#SBATCH --time=00:30:00
#SBATCH --partition=short
#SBATCH --output=r_analysis_%j.out

module load r/4.5.1
Rscript analysis.R
```

Quick R analyses with a short walltime are a good fit for the `short` partition. For longer R jobs, change `--partition=short` to `--partition=amd`.

### GPU Deep Learning

```bash
#!/bin/bash -l
#SBATCH --job-name=train_model
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --time=12:00:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:a100:1
#SBATCH --output=train_%j.out

module load cuda miniconda3
source ~/ml_env/bin/activate
python train.py --epochs 100
```

## Best Practices



### Stop on First Error

```bash
#!/bin/bash -l
set -e    # Exit on first error

python step1.py  # Must succeed
python step2.py  # Only runs if step1 succeeds
```

### Include Logging

```bash
echo "Job started at $(date)"
echo "Running on $(hostname)"
echo "Job ID: $SLURM_JOB_ID"

python my_analysis.py

echo "Job completed at $(date)"
```

### Use Scratch for I/O

```bash
cp /rhome/$USER/large_input.dat /scratch/$SLURM_JOB_USER/$SLURM_JOB_ID/
cd /scratch/$SLURM_JOB_USER/$SLURM_JOB_ID/
python process.py
cp output.csv /rhome/$USER/results/
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Write a Batch Script

Create `simple_job.sh` that:

1. Uses the `short` partition with 1 core and 1 GB RAM for 5 minutes
2. Prints the hostname, job ID, and current date
3. Sleeps for 10 seconds
4. Prints "Job complete!"

```bash
#!/bin/bash -l
#SBATCH --job-name=hello_world
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=00:05:00
#SBATCH --partition=short
#SBATCH --output=hello_%j.out

echo "Hello from Sagehen!"
echo "Job ID: $SLURM_JOB_ID"
echo "Running on: $(hostname)"
sleep 10
echo "Job complete!"
```

Save this file and make it executable with `chmod +x simple_job.sh`.

:::::::::::::::::::::::: solution

## Solution

The script includes all required SBATCH directives and uses SLURM environment variables. The `%j` in the output filename ensures each run creates a unique file. The `sleep 10` simulates actual computation.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Batch scripts are bash scripts with `#SBATCH` directives for resource requests
- First line must be `#!/bin/bash -l` (the `-l` flag loads your environment)
- Key directives: `--job-name`, `--cpus-per-task`, `--mem`, `--time`, `--partition`, `--output`
- Always load modules inside the script, not just interactively
- Use `set -e` to stop on first error
- Include logging (date, hostname, job ID) for debugging
- Use scratch storage for I/O-intensive intermediate files

::::::::::::::::::::::::::::::::::::::::::::::
