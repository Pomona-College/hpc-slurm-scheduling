---
title: "Requesting Resources"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I specify CPU, memory, time, and GPU resources in SLURM?
- How do I estimate what resources my job needs?
- What storage options are available during a job?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Master SLURM resource request syntax for CPU, memory, time, and GPUs
- Learn how to estimate resource needs for different workloads
- Understand storage options available during jobs

:::::::::::::::::::::::::::::::::::::::::::::::

## SLURM Resource Request Keywords

### CPU and Memory

| Parameter | Meaning | Example |
|-----------|---------|---------|
| `-c` / `--cpus-per-task` | CPU cores per task | `-c 4` |
| `-N` / `--nodes` | Number of nodes | `-N 1` |
| `--mem` | RAM per node | `--mem=50G` |
| `--mem-per-cpu` | RAM per CPU core | `--mem-per-cpu=4G` |

### Time

| Parameter | Example | Meaning |
|-----------|---------|---------|
| `-t` / `--time` | `-t 00:30:00` | 30 minutes |
| | `-t 1-12:00:00` | 1 day 12 hours |
| | `-t 1440` | 1440 minutes (1 day) |

### GPUs

| Parameter | Example | Meaning |
|-----------|---------|---------|
| `--gres=gpu:N` | `--gres=gpu:2` | 2 GPUs (any type) |
| `--gres=gpu:type:N` | `--gres=gpu:a100:1` | 1 A100 specifically |

### Partition

| Parameter | Example |
|-----------|---------|
| `-p` / `--partition` | `-p amd` |

## Estimating Resource Needs

### CPU Cores

1. Is your code multi-threaded? Check if it uses OpenMP, multiprocessing, etc.
2. How many threads does it spawn?
3. **Rule of thumb**: Start with 4 cores. Double if too slow.

### Memory

1. How big are your input datasets?
2. Does the algorithm create temporary copies?
3. **Rule of thumb**: Request 2-4x the size of your largest input file.

### Time

1. Have you run this job before? How long did it take?
2. **Rule of thumb**: Request 2x your best guess. SLURM runs your job as soon as there's space.

### GPUs

1. Does your code use CUDA, PyTorch, or TensorFlow?
2. **Rule of thumb**: Start with 1 GPU. Check utilization with `nvidia-smi`.

## Common Resource Request Patterns

### Small CPU job (quick test)
```bash
srun -c 2 --mem=8G -t 00:30:00 -p short my_script.py
```

### Medium CPU job
```bash
srun -c 16 --mem=128G -t 04:00:00 -p amd my_analysis.R
```

### Large CPU job
```bash
srun -c 64 --mem=450G -t 24:00:00 -p amd heavy_simulation.sh
```

### Single GPU job
```bash
srun --gres=gpu:a100:1 -c 8 --mem=64G -t 12:00:00 -p gpu train_model.py
```

## Storage During Jobs

| Storage | Speed | Persistence | Use For |
|---------|-------|-------------|---------|
| `/rhome` | Medium | Persistent | Source code, results |
| `/bigdata` | Medium | Persistent | Shared datasets |
| `/scratch/$SLURM_JOB_USER/$SLURM_JOB_ID` | Fast (SSD) | Deleted when job completes | Working files |
| `/tmpfs` | Very fast (RAM) | Deleted when job ends | Hot data |

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Resource Estimation

Estimate the resources needed for these jobs:

1. Processing a 10 GB CSV file with pandas (under 1 hour)
2. Training a ResNet-50 model on ImageNet (24 hours on a single GPU)
3. Compiling a large C++ codebase with 16 parallel jobs

:::::::::::::::::::::::: solution

## Solution

1. **10 GB CSV**: `-c 4 --mem=50G -t 01:00:00 -p amd` (4 cores, 5x file size for memory)
2. **ResNet-50**: `--gres=gpu:a100:1 -c 8 --mem=64G -t 24:00:00 -p gpu`
3. **C++ compilation**: `-c 16 --mem=32G -t 00:30:00 -p short` (parallel make with `-j16`; short walltime fits the `short` partition)

Start conservative and adjust based on actual usage from `seff`.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Resource requests use SLURM flags: `-c` (cores), `--mem` (memory), `-t` (time), `--gres=gpu` (GPUs)
- Start conservative with estimates; monitor actual usage with `seff` and adjust
- Time format: HH:MM:SS or D-HH:MM:SS for longer jobs
- Use `/scratch` for fast temporary I/O during jobs; use `/rhome` or `/bigdata` for persistent results
- Over-requesting wastes resources and makes other users wait longer

::::::::::::::::::::::::::::::::::::::::::::::
