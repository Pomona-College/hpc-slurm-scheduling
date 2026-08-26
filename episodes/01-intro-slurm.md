---
title: "Introduction to SLURM"
teaching: 15
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: questions

- What is a job scheduler and why do we need one?
- What is SLURM?
- Why can't I just run jobs on the head node?

::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand the purpose of job schedulers in shared computing environments
- Learn the basic architecture of Sagehen HPC
- Distinguish between head nodes and compute nodes
- Understand the SLURM job lifecycle

:::::::::::::::::::::::::::::::::::::::::::::::

## What is a Job Scheduler?

When multiple researchers share a computing cluster, a **job scheduler** coordinates access:

1. **Manages resources** -- tracks CPU, memory, and GPU availability
2. **Queues jobs** -- accepts requests and holds them until resources are free
3. **Allocates resources** -- assigns specific compute nodes to jobs
4. **Ensures fairness** -- prevents any one user from monopolizing the cluster
5. **Enforces policies** -- implements time limits, quotas, and priority rules

Without a scheduler, researchers would need to manually coordinate who runs what and when.

## SLURM: Simple Linux Utility for Resource Management

**SLURM** is the job scheduler used on Sagehen. It is one of the most widely deployed schedulers in HPC, running on thousands of clusters worldwide.

### Key SLURM Concepts

- **Jobs**: A unit of work -- either **interactive** (real-time shell) or **batch** (scripted, unattended)
- **Partitions**: Logical groupings of nodes with different characteristics
- **Resources**: What you request for your job (CPU cores, memory, time, GPUs)

## The Sagehen HPC Cluster Architecture

### Head Node

- **Address**: sagehen.hpc.pomona.edu
- **CPU**: 2 threads; **RAM**: 8 GB
- **Purpose**: Login, submit jobs, manage files

::::::::::::::::::::::::::::::::::::::: callout

## The Head Node is Sacred

The head node has only 2 CPU threads and 8 GB of RAM. Running compute jobs there will crash the login server, kill other users' jobs, and result in account restrictions. Always submit jobs to compute nodes via SLURM.

:::::::::::::::::::::::::::::::::::::::::::::::

### Compute Nodes

| Partition | Specs | Max Time | Best For |
|-----------|-------|----------|----------|
| amd | See `sinfo -p amd` for current configuration | 30 days | General compute |
| gpu | 10 GPUs across multiple nodes (4× A100, 4× L40S, 2× RTX PRO 6000; see Workshop 16) | 30 days | ML, GPU-accelerated work |
| short | See `sinfo -p short` for current configuration | Shorter max walltime than amd/gpu | Quick test jobs, debugging, rapid prototyping |

## The SLURM Job Lifecycle

1. **Submission**: You run `sbatch` (batch) or `srun` (interactive) on the head node
2. **Queue**: SLURM accepts your job and places it in a queue
3. **Waiting**: SLURM waits for available resources and priority scheduling
4. **Allocation**: SLURM assigns specific compute nodes
5. **Execution**: Your job runs on those nodes
6. **Completion**: Output is written to files; nodes are freed

## SLURM Command Overview

| Command | Purpose |
|---------|---------|
| `srun` | Start an interactive job |
| `sbatch` | Submit a batch script |
| `squeue` | View jobs in the queue |
| `scancel` | Cancel a job |
| `sacct` | View job history |
| `seff` | Analyze job efficiency |
| `sinfo` | View cluster status |

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Exploring Sagehen HPC

Connect to Sagehen and explore its structure.

1. Connect: `ssh <myusername>@sagehen.hpc.pomona.edu`
2. Complete DUO MFA authentication
3. Check the head node:
   ```bash
   hostname
   nproc
   free -h
   ```
4. View partitions:
   ```bash
   sinfo
   sinfo -s
   ```

Look for the `amd`, `gpu`, and `short` partitions in the output.

:::::::::::::::::::::::: solution

## Solution

You should see `sagehen` from `hostname`, `2` from `nproc`, and ~8 GB from `free -h`. The `sinfo` output shows the `amd`, `gpu`, and `short` partitions with their node counts and time limits.

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- A job scheduler coordinates resource sharing on a cluster
- SLURM is the job scheduler on Sagehen
- The head node (2 CPUs, 8 GB RAM) is for login and job submission only
- Jobs run on compute nodes in three partitions: `amd` (general compute), `gpu` (GPU-accelerated), and `short` (quick test/debug jobs)
- SLURM enforces fair access through queuing, resource limits, and priorities

::::::::::::::::::::::::::::::::::::::::::::::

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
