---
title: Quick Reference
---

## Quick Reference Card: SLURM Commands on Sagehen HPC

This is a handy cheat sheet for common SLURM commands. Print it out or bookmark it!

## Connection and Setup

```bash
# Connect to Sagehen
ssh <myusername>@sagehen.hpc.pomona.edu

# Check you're on head node
hostname  # Should show "sagehen"

# View available modules
module avail

# Load a module
module load miniconda3

# Load multiple modules
module load miniconda3 cuda r/4.5.1
```

## Cluster Information

```bash
# View available partitions and nodes
sinfo

# View cluster status summary
sinfo -s

# View specific partition
sinfo -p gpu

# Check account/group quota
quota_check.sh

# Check group resource usage
squeue | grep group_name
```

## Submitting Jobs

```bash
# Interactive job (bash shell on compute node)
srun -c 4 --mem=16G -t 00:30:00 -p amd --pty bash -l

# Interactive job with GPU
srun --gres=gpu:a100:1 -c 8 --mem=64G -t 02:00:00 -p gpu --pty bash -l

# Submit batch script
sbatch myscript.sh

# Submit with inline parameters
sbatch --cpus-per-task=8 --mem=32G myscript.sh
```

## Monitoring Jobs

```bash
# View your running/pending jobs
squeue -u $USER

# View specific job
squeue -j 12345

# View jobs in amd partition
squeue -p amd

# Watch queue updates every 2 seconds
watch -n 2 squeue -u $USER

# Show more detail (CPU, memory, time limit)
squeue -u $USER --format=jobid,name,cpus,mem,timelimit,state
```

## Job Management

```bash
# Cancel a job
scancel 12345

# Cancel all your jobs (use with caution!)
scancel -u $USER

# Cancel jobs by name
scancel --name=test_job

# Cancel pending jobs only
scancel -u $USER --state=PENDING
```

## Job History and Analysis

```bash
# View completed jobs (last several)
sacct -u $USER

# View specific job details
sacct -j 12345

# View job efficiency (resource usage)
seff 12345

# Show memory usage
seff 12345 | grep Memory

# View all jobs since yesterday
sacct -u $USER --starttime=2026-03-04

# Export to CSV
sacct -u $USER --format=jobid,jobname,state,elapsed --delimiter=,
```

## Common SLURM Directives (for batch scripts)

```bash
#!/bin/bash -l
#SBATCH --job-name=myjob              # Job name
#SBATCH --partition=amd               # amd, gpu, or short
#SBATCH --cpus-per-task=4             # CPU cores
#SBATCH --mem=16G                     # RAM (per node)
#SBATCH --time=01:30:00               # Max time (HH:MM:SS)
#SBATCH --nodes=1                     # Number of nodes
#SBATCH --output=job_%j.out           # Output file (%j=job ID)
#SBATCH --error=job_%j.err            # Error file
#SBATCH --gres=gpu:a100:1             # GPUs (type:count)
#SBATCH --mail-user=you@pomona.edu    # Email address
#SBATCH --mail-type=END               # Email on END, FAIL, BEGIN, ALL
#SBATCH --qos=normal                  # Quality of service
#SBATCH --array=1-10                  # Job array (10 jobs)
```

## Resource Request Examples

```bash
# Small quick job (1 core, 1 hour)
-c 1 --mem=4G -t 01:00:00 -p short

# Medium CPU job (8 cores, 4 hours)
-c 8 --mem=32G -t 04:00:00 -p amd

# Large CPU job (32 cores, 24 hours)
-c 32 --mem=256G -t 24:00:00 -p amd

# Single GPU job
--gres=gpu:a100:1 -c 8 --mem=64G -t 12:00:00 -p gpu

# Multi-GPU job
--gres=gpu:2 -c 16 --mem=128G -t 12:00:00 -p gpu
```

## Environment Variables in Jobs

Inside a batch script or interactive session, these are automatically set:

```bash
$SLURM_JOB_ID          # Job ID number
$SLURM_JOB_NAME        # Job name
$SLURM_ARRAY_TASK_ID   # Task ID (for job arrays)
$SLURM_CPUS_PER_TASK   # Number of CPUs allocated
$SLURM_MEM             # RAM allocated (MB)
$SLURM_JOB_USER        # Your username
$SLURM_TMPDIR          # Scratch directory path
```

Example usage:
```bash
cd /scratch/$SLURM_JOB_USER/$SLURM_JOB_ID
python script.py --output results_${SLURM_JOB_ID}.csv
```

## Partition Quick Reference

```
amd:      see `sinfo -p amd` for current configuration; 30-day max job time
gpu:      10 GPUs across multiple nodes (4× A100, 4× L40S, 2× RTX PRO 6000), 30-day limit (see Workshop 16 for full hardware breakdown)
short:    see `sinfo -p short` for current configuration; shorter max walltime (good for quick test/debug jobs and rapid prototyping)
```

## Storage Paths

```bash
/rhome/$USER              # Home directory; /rhome and /bigdata share a single 1 TB lab quota on BeeGFS (check usage with quota_check.sh)
/bigdata/lab/<labname>          # Lab shared storage; shared 1 TB quota with /rhome
/scratch/$USER/$JOBID     # Fast job scratch (auto-deleted when job completes)
/tmpfs/$USER/$JOBID       # RAM-based temp (auto-deleted when job completes)
```

## Module Commands

```bash
module avail              # List all available modules
module avail python       # Find Python versions
module load miniconda3   # Load specific module
module list               # Show loaded modules
module unload python      # Unload a module
module purge              # Unload all modules
module swap old new       # Switch modules
```

## Monitoring While Running

```bash
# From head node, check job resources
squeue -j 12345 --format=jobid,cpus,mem,gres,state

# From compute node (if connected), monitor
free -h                   # Memory usage
top                       # CPU usage
nvidia-smi                # GPU usage (if on GPU node)
```

## Common Time Formats

```bash
-t 00:30:00              # 30 minutes
-t 02:00:00              # 2 hours
-t 1-00:00:00            # 1 day
-t 7-00:00:00            # 1 week
-t 30-00:00:00           # 30 days (max for amd/gpu)
```

## Quick Diagnostics

```bash
# Why is my job pending?
squeue -j 12345 -O jobid,reason

# How much time did my job use?
seff 12345 | grep "Wall-clock"

# Did my job run out of memory?
seff 12345 | grep Memory

# When did my job finish?
sacct -j 12345 --format=end

# How many cores is my group using?
squeue -t RUNNING | awk '{cores+=$7} END {print cores}'
```

## Common Errors and Quick Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| "Job submit/cancel time limit exceeded" | Group quota exceeded | Wait for jobs to finish |
| "sbatch: error: job_script: No such file or directory" | Script doesn't exist | Check file path and name |
| "Killed" in output | Out of memory | Increase `--mem=`, reduce batch size |
| ExitCode 138 (TIMEOUT) | Time limit exceeded | Increase `-t` flag |
| "CUDA error" | GPU not requested | Add `--gres=gpu:1` |
| Pending for hours | No resources available | Reduce resource request or try later |

## Escape Hatch: Check Everything

```bash
# Your jobs and their status
squeue -u $USER

# Why is job X not starting?
squeue -j 12345 -O jobid,reason,priority

# How efficient was job X?
seff 12345

# What's the cluster status?
sinfo

# Your group's usage?
quota_check.sh
```

## Getting Help

```bash
# SLURM help
man srun       # srun manual
man sbatch     # sbatch manual
man squeue     # squeue manual

# Sagehen-specific help
its-hpc@pomona.edu     # Email support

# Check our workshop
Visit episodes/ folder for full documentation
```

## Batch Script Template

```bash
#!/bin/bash -l
#SBATCH --job-name=myanalysis
#SBATCH --partition=amd
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=02:00:00
#SBATCH --output=job_%j.out
#SBATCH --mail-user=you@pomona.edu
#SBATCH --mail-type=END,FAIL

# Load environment
module load miniconda3

# Your code here
python analysis.py
```

Save as `job.sh` and submit with: `sbatch job.sh`

---

**Last Updated**: March 2026
**For more details**: See the full workshop episodes

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
