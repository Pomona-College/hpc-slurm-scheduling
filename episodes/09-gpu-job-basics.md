---
title: "GPU Job Basics"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- When should I use a GPU?
- How do I request GPUs in SLURM?
- Which GPU type should I choose?
- How do I write a GPU batch script?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand when GPUs accelerate your work and when they do not
- Master `--gres=gpu` syntax for GPU requests
- Match GPU requirements to available hardware
- Write and submit GPU batch scripts

:::::::::::::::::::::::::::::::::::::::::::::::

## When Should You Use a GPU?

### Workloads That Benefit

- **Deep Learning**: Training neural networks (PyTorch, TensorFlow)
- **Matrix Math**: Large matrix multiplications, dense linear algebra
- **GPU-Accelerated Libraries**: RAPIDS, CuPy, Dask-CUDA
- **Molecular Dynamics**: GPU-accelerated GROMACS, NAMD
- **Custom CUDA Code**: Parallel kernels

### Workloads That Do NOT Benefit

- Single-threaded serial code
- I/O-bound jobs (waiting for disk/network)
- Light computation (simple statistics, data cleaning)
- Memory-bound sequential code

::::::::::::::::::::::::::::::::::::::: callout

## Don't Waste GPU Resources

Before submitting a GPU job:
1. Test on CPU first
2. Run with `nvidia-smi` and check utilization
3. If GPU utilization is below 50%, run on CPU instead

GPU nodes are expensive and limited. Wasting them slows everyone else down.

:::::::::::::::::::::::::::::::::::::::::::::::

## Requesting GPUs

### Syntax

```bash
#SBATCH --gres=gpu:N              # N GPUs, any type
#SBATCH --gres=gpu:TYPE:N         # N GPUs of specific type
```

### Examples

```bash
#SBATCH --gres=gpu:1              # 1 GPU, any type (fastest to allocate)
#SBATCH --gres=gpu:a100:1         # 1 A100 (80 GB, production)
#SBATCH --gres=gpu:v100:1         # 1 V100 (16 GB, prototyping)
#SBATCH --gres=gpu:l40s:1         # 1 L40S (48 GB, inference)
#SBATCH --gres=gpu:2              # 2 GPUs, any type
```

### Which GPU to Choose

- **Starting a project**: `gpu:1` (let SLURM choose)
- **Need lots of GPU memory**: `gpu:a100:1` (80 GB)
- **Production training**: `gpu:a100:1` (fastest)
- **Prototyping**: `gpu:v100:1` (most available)

## GPU Batch Script

```bash
#!/bin/bash -l
#SBATCH --job-name=train_model
#SBATCH --partition=gpu
#SBATCH --gres=gpu:a100:1
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --time=12:00:00
#SBATCH --output=train_%j.out

module load cuda python/3.11
source ~/ml_env/bin/activate

python train.py --epochs 100 --batch_size 128
```

Key points:
- `--partition=gpu`: Must use the gpu partition
- `--gres=gpu:a100:1`: Request the GPU
- `--cpus-per-task=8`: GPUs still need CPU support threads
- `module load cuda`: Required for CUDA libraries

## Interactive GPU Testing

Test GPU code before submitting batch jobs:

```bash
srun --partition=gpu --gres=gpu:v100:1 \
     --cpus-per-task=4 --mem=16G --time=00:30:00 \
     --pty bash -l
```

Inside the session:
```bash
module load cuda python/3.11
python -c "import torch; print(torch.cuda.is_available())"
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: GPU Test Job

Create and submit a GPU test script:

```bash
#!/bin/bash -l
#SBATCH --job-name=gpu_test
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=2
#SBATCH --mem=8G
#SBATCH --time=00:05:00
#SBATCH --output=gpu_%j.out

module load cuda python/3.11

python << 'EOF'
import torch
print(f"CUDA available: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    x = torch.randn(1000, 1000).cuda()
    y = torch.mm(x, x)
    print(f"GPU computation result shape: {y.shape}")
EOF
```

Submit with `sbatch` and check the output.

:::::::::::::::::::::::: solution

## Solution

Output should show `CUDA available: True`, the GPU model name, and a `torch.Size([1000, 1000])` result. This confirms your GPU job is configured correctly and the GPU is being used.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- GPUs accelerate parallel computation: deep learning, matrix math, CUDA code
- Do not use GPUs for serial, I/O-bound, or light computational tasks
- Request GPUs with `--gres=gpu:TYPE:N` (e.g., `--gres=gpu:a100:1`)
- Always use `--partition=gpu` and `module load cuda` for GPU jobs
- A100 for production, V100 for prototyping, L40S for inference
- Test GPU code interactively before submitting long batch jobs

::::::::::::::::::::::::::::::::::::::::::::::
