---
title: SLURM Job Scheduling on Sagehen
---

## Welcome to Workshop 9: SLURM Job Scheduling on Sagehen

This workshop is designed to teach you how to effectively schedule and manage your computational jobs on **Sagehen**, Pomona College's research computing cluster. Whether you're running machine learning models, statistical analyses, molecular simulations, or other compute-intensive work, understanding SLURM job scheduling is essential to making the most of Sagehen's powerful computing resources.

### What You'll Learn

By the end of this workshop, you will be able to:

- Understand the role of job schedulers and why SLURM is used on Sagehen
- Navigate the Sagehen cluster architecture and available resources
- Submit interactive jobs for exploratory work and debugging
- Create and submit batch scripts for long-running computations
- Monitor and manage your jobs effectively
- Request and utilize GPU resources appropriately
- Optimize your job submissions using advanced SLURM features

### About Sagehen

**Sagehen** (sagehen.hpc.pomona.edu) is Pomona College's research computing cluster managed by Information Technology Services (ITS). The cluster features:

- **Computing nodes**: AMD-based compute nodes with 128 CPUs and 500GB RAM each
- **GPU nodes**: Specialized nodes with NVIDIA GPUs (A100, L40S, RTX PRO 6000)
- **Storage**: Home directories, shared lab storage, and high-speed scratch space
- **Job scheduler**: SLURM (Simple Linux Utility for Resource Management)
- **Multi-user environment**: Safe resource sharing among hundreds of researchers

### Workshop Structure

This 7-episode workshop is organized as follows:

1. **Introduction to SLURM** - Learn what job schedulers do and basic SLURM concepts
2. **Partitions and Resources** - Explore Sagehen's available compute partitions and how to choose the right one
3. **Interactive Jobs** - Run interactive sessions with `srun` for immediate feedback
4. **Batch Jobs** - Create reproducible batch scripts with `sbatch` for production runs
5. **Monitoring and Managing** - Track and control your submitted jobs
6. **GPU Jobs** - Harness the power of Sagehen's GPU resources
7. **Advanced Tips** - Optimize performance with scratch storage, job arrays, and troubleshooting

::::::::::::::::::::::::::::::::::::: prereq

## Prerequisites

Before starting this workshop, you should have:
- **Access to Sagehen**: An active HPC account (contact its-hpc@pomona.edu if you need one)
- **SSH client**: Ability to connect to a remote system via SSH
- **Basic Linux knowledge**: Comfort with command line operations (files, directories, basic commands)
- **Text editor**: Familiarity with editing text files (nano, vim, or your preferred editor)

If you're new to Linux or command line work, we recommend completing a Linux basics workshop first.

::::::::::::::::::::::::::::::::::::::::::::::::

### How to Use This Workshop

- **For learners**: Each episode includes hands-on challenges. We recommend working through them on Sagehen as you learn.
- **For instructors**: See the [Instructor Notes](instructors/instructor-notes.md) for timing, common issues, and teaching tips.
- **For reference**: Check the [Quick Reference Card](learners/reference.md) for common SLURM commands.

### Getting Help

Questions or issues? Reach out to:

- **Email**: its-hpc@pomona.edu
- **Director of Research Computing**: Andrew Wilson
- **In-person**: Visit ITS in person during office hours

### About Carpentries Workbench

This workshop is built using The Carpentries Workbench, a modern lesson development framework that emphasizes hands-on learning and community collaboration.

[Learn more about The Carpentries](https://carpentries.org/)

---

**Ready to get started?** Begin with [Episode 1: Introduction to SLURM](episodes/01-intro-slurm.md).
