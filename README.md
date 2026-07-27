# SLURM Job Scheduling

Pomona College HPC Workshop Series

## Overview

This workshop provides comprehensive training on SLURM, the job scheduling system used on Sagehen. Learners will master interactive and batch job submission, resource allocation across different partitions, job monitoring, GPU computing, and performance optimization techniques essential for efficient cluster computing.

## Episodes

1. **Introduction to SLURM**: Understand job schedulers, cluster architecture, and why resource coordination matters
2. **Partitions and Resources**: Learn available hardware options and how to request appropriate resources
3. **Interactive Jobs**: Run exploratory work with immediate feedback using interactive compute sessions
4. **Batch Jobs**: Write SLURM scripts and submit production jobs that run unattended
5. **Monitoring and Managing**: Track job progress, analyze resource efficiency, and troubleshoot failures
6. **GPU Jobs**: Request and effectively use GPU accelerators for machine learning and scientific computing
7. **Advanced Tips**: Optimize I/O with scratch storage, use job arrays for multiple runs, and apply performance strategies

## Prerequisites

- Completion of Workshop 0: Introduction to HPC Systems (or equivalent SSH and cluster experience)
- Basic Linux command-line proficiency
- Text editor knowledge (nano, vim, or preference)
- Sagehen HPC cluster account with SSH access

## Learning Objectives

After completing this workshop, learners will be able to:
- Understand how SLURM coordinates resource sharing and fair allocation
- Distinguish between interactive and batch jobs and select appropriate types
- Write SLURM batch scripts with correct headers and resource requests
- Submit jobs to appropriate partitions (`amd`, `gpu`, `short`) based on requirements
- Monitor running jobs and analyze resource utilization
- Request and use GPUs responsibly and effectively
- Optimize I/O performance using scratch storage and caching strategies
- Debug common SLURM errors and failures independently

## Target Audience

Researchers running computationally intensive work including statistical analyses, simulations, machine learning training, molecular dynamics, and data processing on shared computing resources.

## Duration

Approximately 2 to 4 hours depending on depth: core material (Episodes 1-5) takes 2 hours, full workshop with GPU and advanced topics takes 3-4 hours.

## Technical Requirements

- Sagehen HPC cluster account
- SSH access to sagehen.hpc.pomona.edu
- Text editor for script writing
- Basic understanding of Linux command line
- For GPU exercises: knowledge of GPU-accelerated libraries (optional)

## Contact

- **Email**: its-hpc@pomona.edu
- **Workshop Author**: Andrew Wilson, Director of Research Computing

## License

This workshop is licensed under [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).

## Citation

Wilson, A. (2026). *SLURM Job Scheduling*. Pomona College ITS Research Computing.
