---
title: "Job Arrays and Dependencies"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How can I run many similar jobs efficiently?
- How do job arrays work?
- How do I use scratch and tmpfs storage?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Use job arrays for parameter sweeps and batch file processing
- Construct array indices and map them to parameters
- Use scratch and tmpfs storage for fast I/O during jobs

:::::::::::::::::::::::::::::::::::::::::::::::

## Job Arrays: Parameter Sweeps

A **job array** submits many similar jobs at once. Each gets a unique task ID via `$SLURM_ARRAY_TASK_ID`.

### Basic Syntax

```bash
#SBATCH --array=1-10
```

This submits 10 jobs, numbered 1 through 10.

### Example: Learning Rate Sweep

```bash
#!/bin/bash -l
#SBATCH --job-name=lr_sweep
#SBATCH --array=1-4
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=02:00:00
#SBATCH --partition=amd
#SBATCH --output=sweep_%a.out

module load miniconda3

LRS=(0.001 0.01 0.1 1.0)
LR=${LRS[$((SLURM_ARRAY_TASK_ID - 1))]}

echo "Task $SLURM_ARRAY_TASK_ID: Learning rate = $LR"
python train.py --learning_rate $LR --output results_${LR}.csv
```

Submit once with `sbatch sweep.sh` and SLURM creates 4 independent jobs.

### File Processing Array

```bash
#!/bin/bash -l
#SBATCH --array=1-100

FILES=$(ls /bigdata/lab/images/*.jpg | sort)
FILE=$(echo "$FILES" | sed -n "${SLURM_ARRAY_TASK_ID}p")

python process_image.py $FILE
```

### Parameter Combinations (Grid Search)

```bash
#!/bin/bash -l
#SBATCH --array=1-12

# 3 learning rates x 4 batch sizes = 12 combinations
LRS=(0.001 0.01 0.1)
BATCH_SIZES=(32 64 128 256)

LR_IDX=$(( (SLURM_ARRAY_TASK_ID - 1) / 4 ))
BATCH_IDX=$(( (SLURM_ARRAY_TASK_ID - 1) % 4 ))

LR=${LRS[$LR_IDX]}
BATCH=${BATCH_SIZES[$BATCH_IDX]}

python train.py --lr $LR --batch $BATCH
```

### Monitoring Arrays

```bash
squeue -u $USER              # Shows all array tasks
squeue -j 12345_1            # Check specific task
scancel 12345                # Cancel entire array
scancel 12345_3              # Cancel single task
```

## Fast Storage: Scratch and Tmpfs

### Scratch Storage Pattern

Use `/scratch` for I/O-intensive work (SSD-backed, much faster than network storage):

```bash
#!/bin/bash -l
#SBATCH --time=02:00:00

WORK_DIR=/scratch/$SLURM_JOB_USER/$SLURM_JOB_ID
mkdir -p $WORK_DIR

# Copy input to scratch
cp /rhome/$USER/large_input.dat $WORK_DIR/

# Work on scratch (fast I/O)
cd $WORK_DIR
python process.py

# Copy results back (scratch is deleted when job completes)
cp output.csv /rhome/$USER/results/
```

### Tmpfs: RAM-Based Storage

For extremely hot data (accessed millions of times):

```bash
TMPFS_DIR=/tmpfs/$SLURM_JOB_USER/$SLURM_JOB_ID
mkdir -p $TMPFS_DIR
cp hot_data.bin $TMPFS_DIR/
python algorithm.py --input $TMPFS_DIR/hot_data.bin
```

::::::::::::::::::::::::::::::::::::::: callout

## Tmpfs Caution

Tmpfs uses RAM. Using too much reduces memory available for your program and can cause the OS to kill processes. Use tmpfs only for small, frequently-accessed data.

:::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Parameter Sweep Array

Create and submit a 5-task job array:

```bash
#!/bin/bash -l
#SBATCH --job-name=param_sweep
#SBATCH --array=1-5
#SBATCH --cpus-per-task=1
#SBATCH --mem=2G
#SBATCH --time=00:05:00
#SBATCH --partition=short
#SBATCH --output=array_%a.out

PARAMS=(10 50 100 500 1000)
PARAM=${PARAMS[$((SLURM_ARRAY_TASK_ID - 1))]}

echo "Task $SLURM_ARRAY_TASK_ID: Parameter = $PARAM"
python -c "print(f'Result: {sum(range($PARAM))}')"
```

Submit with `sbatch`, then check each output file: `cat array_1.out` through `cat array_5.out`.

:::::::::::::::::::::::: solution

## Solution

Five jobs run independently, each with a different parameter value. Results show sum(range(N)) for N = 10, 50, 100, 500, 1000. This demonstrates how arrays parallelize parameter exploration efficiently.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Job arrays (`--array=1-N`) submit many similar jobs with one command
- Each task gets a unique `$SLURM_ARRAY_TASK_ID` for parameter mapping
- Use arrays for parameter sweeps, file processing, and grid searches
- `/scratch` provides fast SSD storage deleted when the job completes
- `/tmpfs` provides RAM-backed storage deleted when the job ends
- Always copy important results from scratch back to `/rhome` or `/bigdata`

::::::::::::::::::::::::::::::::::::::::::::::
