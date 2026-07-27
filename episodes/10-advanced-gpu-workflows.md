---
title: "Advanced GPU Workflows"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How do I monitor GPU utilization?
- How do I write PyTorch and TensorFlow GPU code?
- How do I run multi-GPU training?
- How do I troubleshoot GPU errors?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Monitor GPU utilization with `nvidia-smi`
- Write GPU-aware code in PyTorch and TensorFlow
- Submit multi-GPU training jobs
- Troubleshoot common GPU errors (CUDA OOM, GPU not detected)

:::::::::::::::::::::::::::::::::::::::::::::::

## Monitoring GPU Usage

### nvidia-smi

From a terminal on the GPU node:

```bash
nvidia-smi                          # One-time snapshot
watch -n 5 nvidia-smi               # Update every 5 seconds
nvidia-smi dmon                     # Continuous monitoring
```

Key metrics:
- **GPU-Util**: Should be above 80% for well-utilized GPU jobs
- **Memory-Usage**: How much of GPU memory is consumed
- **Power**: Higher power = busier GPU

## GPU Code Examples

### PyTorch

```python
import torch

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# Move model and data to GPU
model = torch.nn.Sequential(
    torch.nn.Linear(1000, 500),
    torch.nn.ReLU(),
    torch.nn.Linear(500, 10)
).to(device)

x = torch.randn(1000, 1000).to(device)
output = model(x)
```

### TensorFlow

```python
import tensorflow as tf

# TensorFlow auto-places on GPU if available
model = tf.keras.Sequential([
    tf.keras.layers.Dense(500, activation='relu', input_shape=(1000,)),
    tf.keras.layers.Dense(10)
])
model.compile(optimizer='adam', loss='mse')
model.fit(x_train, y_train, epochs=10)
```

## Multi-GPU Training

```bash
#!/bin/bash -l
#SBATCH --job-name=dist_train
#SBATCH --partition=gpu
#SBATCH --gres=gpu:2
#SBATCH --cpus-per-task=16
#SBATCH --mem=128G
#SBATCH --time=12:00:00

module load cuda python/3.11

python -m torch.distributed.launch \
    --nproc_per_node=2 \
    train_distributed.py
```

Request only multiple GPUs if your code actually supports distributed training.

## Troubleshooting GPU Errors

### "CUDA out of memory"

Your model or data exceeds GPU memory.

```bash
# Reduce batch size
python train.py --batch_size 64    # Was 256

# Or request a larger GPU
#SBATCH --gres=gpu:a100:1          # 80 GB instead of V100's 16 GB
```

### "GPU not detected" / `torch.cuda.is_available()` returns False

```bash
# Verify you requested a GPU
grep gres your_script.sh

# Verify CUDA module is loaded
module list | grep cuda

# Check allocation
squeue -j $SLURM_JOB_ID --format=gres
```

### Low GPU Utilization

Causes: data loading bottleneck, code not on GPU, small batch size.

Fixes:
1. Add parallel data loading (`num_workers` in PyTorch DataLoader)
2. Verify `.to(device)` or `.cuda()` in your code
3. Increase batch size if GPU memory allows
4. If utilization stays low, use CPU instead

### Job Killed for Exceeding Memory

GPU jobs have both CPU RAM and GPU memory limits:
```bash
# Check which ran out
cat slurm-JOBID.err
seff JOBID | grep Memory
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: GPU Benchmark

Create a script that benchmarks GPU vs CPU matrix multiplication:

```bash
#!/bin/bash -l
#SBATCH --job-name=gpu_bench
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=2
#SBATCH --mem=16G
#SBATCH --time=00:10:00
#SBATCH --output=bench_%j.out

module load cuda python/3.11

python << 'EOF'
import torch, time

size = 10000
x_gpu = torch.randn(size, size).cuda()
y_gpu = torch.randn(size, size).cuda()
torch.cuda.synchronize()
start = time.time()
z = torch.mm(x_gpu, y_gpu)
torch.cuda.synchronize()
gpu_time = time.time() - start

x_cpu = x_gpu.cpu(); y_cpu = y_gpu.cpu()
start = time.time()
z_cpu = torch.mm(x_cpu, y_cpu)
cpu_time = time.time() - start

print(f"GPU: {gpu_time:.4f}s, CPU: {cpu_time:.4f}s, Speedup: {cpu_time/gpu_time:.1f}x")
EOF
```

Submit and compare the GPU vs CPU times.

:::::::::::::::::::::::: solution

## Solution

The GPU should be significantly faster for large matrix multiplication (often 10-100x). The exact speedup depends on the GPU type and matrix size. This demonstrates why GPU acceleration matters for linear algebra and deep learning workloads.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Monitor GPU utilization with `nvidia-smi`; aim for over 80% GPU-Util
- PyTorch uses `.to(device)` or `.cuda()`; TensorFlow auto-places on GPU
- Multi-GPU training requires both `--gres=gpu:2` and distributed training code
- CUDA OOM: reduce batch size or request a larger GPU (A100 has 80 GB)
- GPU not detected: check `--gres`, `module load cuda`, and partition selection
- Low utilization often means a CPU data-loading bottleneck or undersized batches

::::::::::::::::::::::::::::::::::::::::::::::
