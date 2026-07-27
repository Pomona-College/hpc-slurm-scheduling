---
title: "Interactive Jobs"
teaching: 15
exercises: 15
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I get an interactive shell on a compute node?
- How do I request resources for an interactive session?
- When should I use interactive vs batch jobs?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Master `srun --pty bash` for interactive sessions
- Request CPU, memory, and GPU resources for interactive jobs
- Use interactive sessions for development and debugging
- Know when to switch to batch jobs

:::::::::::::::::::::::::::::::::::::::::::::::

## The `srun` Command

`srun` starts interactive jobs. The most common form for a short interactive debugging session:

```bash
srun -c 4 --mem=16G -t 00:30:00 -p short --pty bash -l
```

- `-c 4`: 4 CPU cores
- `--mem=16G`: 16 GB RAM
- `-t 00:30:00`: 30 minutes
- `-p short`: `short` partition (good for short interactive work; for sessions longer than the `short` walltime limit, use `-p amd`)
- `--pty bash -l`: interactive login shell

After SLURM allocates resources, your prompt changes to show the compute node:
```
[user@compute-node ~]$
```

Type `exit` or Ctrl+D to end the session and free resources.

## Common Interactive Patterns

### Quick Test (2 minutes, 1 core)
```bash
srun -c 1 --mem=4G -t 00:02:00 -p short --pty bash -l
```

### Development Session (30 minutes, 4 cores)
```bash
srun -c 4 --mem=16G -t 00:30:00 -p short --pty bash -l
```

### Large Interactive (2 hours, 16 cores)
```bash
srun -c 16 --mem=100G -t 02:00:00 -p amd --pty bash -l
```

(For interactive work longer than the `short` partition's max walltime, use `-p amd` instead.)

### Interactive GPU Session
```bash
srun --gres=gpu:a100:1 -c 8 --mem=64G -t 04:00:00 -p gpu --pty bash -l
```

::::::::::::::::::::::::::::::::::::::: callout

## Request Time Generously

If you finish early, just `exit` to free resources. If time runs out, your session is killed abruptly. Better to release early than get a timeout surprise.

:::::::::::::::::::::::::::::::::::::::::::::::

## Working in Interactive Sessions

Load modules and run code just like on your laptop:

```bash
[user@compute-node ~]$ module load python/3.11
[user@compute-node ~]$ python my_analysis.py
```

Edit and rerun for iterative development:

```bash
[user@compute-node ~]$ nano my_script.R
[user@compute-node ~]$ Rscript my_script.R
```

### Using tmux for Persistent Sessions

If your connection drops, a regular interactive session dies. Use `tmux` inside your session:

```bash
tmux new-session -s work
# Detach: Ctrl+B, then D
# Reconnect: tmux attach-session -t work
```

## When to Use Interactive vs Batch

**Interactive**: developing code, debugging, exploring data, quick tests

**Batch**: stable code, jobs over 2 hours, many parallel jobs, unattended runs

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Your First Interactive Session

1. Start a session: `srun -c 2 --mem=8G -t 00:10:00 -p short --pty bash -l`
2. Verify you are on a compute node:
   ```bash
   hostname
   echo $SLURM_JOB_ID
   free -h
   nproc
   ```
3. Load Python and run a quick test:
   ```bash
   module load python/3.11
   python -c "import numpy as np; print(np.random.randn(5))"
   ```
4. Exit: `exit`
5. Confirm you are back on the head node: `hostname`

:::::::::::::::::::::::: solution

## Solution

Inside the session, `hostname` shows a compute node (e.g., `a005`), `free -h` shows 500 GB total RAM, and `nproc` shows 128 cores. After `exit`, `hostname` returns `sagehen`. The Python test confirms modules work on compute nodes.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Use `srun --pty bash -l` for interactive sessions on compute nodes
- Request resources: `-c` (cores), `--mem` (RAM), `-t` (time), `-p` (partition)
- Interactive jobs are best for development, testing, and debugging
- Use the `short` partition for short interactive sessions and quick debugging; use `amd` for longer interactive work
- Use tmux inside sessions for connection resilience
- Switch to batch jobs when code is stable and needs long or unattended runs

::::::::::::::::::::::::::::::::::::::::::::::
