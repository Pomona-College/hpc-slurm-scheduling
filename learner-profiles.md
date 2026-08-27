---
title: Learner Profiles
---

## Who Takes This Workshop?

This workshop serves researchers across Pomona College with diverse backgrounds, goals, and computing needs. Here are three typical learner profiles to guide workshop design:

---

## Profile 1: Sam - Chemistry Graduate Student

**Background**: Sam is in their 3rd year of a chemistry PhD, working on computational drug discovery. They have basic Linux experience from a undergraduate "computing in chemistry" course but have never used HPC.

**Motivation**: "My molecular dynamics simulations take weeks on my laptop. My advisor says Sagehen HPC can run them in days. I need to understand how to submit jobs there."

**What Sam needs**:
- To get basic MD simulations running without deep understanding of SLURM
- Confidence that their code will run correctly
- A way to check if jobs are progressing
- Guidance on appropriate resource requests for MD (CPUs vs GPUs vs time)

**What Sam will learn**:
- ✓ How to write batch scripts with correct SLURM directives
- ✓ How to request appropriate resources for MD simulations
- ✓ How to monitor job progress and check efficiency
- ✓ Where to find fast storage for I/O-heavy simulations
- Sam might skip: Deep GPU optimization (MD runs on CPU), job arrays (runs one large job at a time)

**Sam's challenges**:
- Remembering SLURM syntax (solved by bookmarking the reference card)
- Estimating resource needs (solved by iterating with `seff`)
- Debugging when code fails on a compute node (solved by understanding output files)

**Sam's success criteria**:
- "I can submit a batch job that runs my GROMACS simulation"
- "I understand what `seff` tells me about efficiency"
- "My simulations run in 2-3 days instead of weeks"

---

## Profile 2: Alex - Neuroscience Undergraduate Researcher

**Background**: Alex is a senior doing an honors thesis on machine learning applied to neural imaging. They've taken programming courses (Python) and used Jupyter notebooks for data analysis, but never used a terminal much.

**Motivation**: "I need to train a neural network on 10,000 brain scans. My laptop takes forever. A mentor told me to try Sagehen. I want to learn enough to not slow down my research."

**What Alex needs**:
- Step-by-step guidance (might not be comfortable with Linux yet)
- Clear examples they can copy and modify
- Understanding of GPU usage (their models are deep learning)
- Patience with the learning curve

**What Alex will learn**:
- ✓ How to connect to Sagehen and navigate the file system
- ✓ How to write Python batch scripts with module loading
- ✓ How to request GPUs and verify they're being used
- ✓ How to diagnose common errors (out of memory, timeout)
- ✓ How to iteratively improve resource estimates

**Alex might skip**:
- Job arrays (they'll run one training job at a time, not parameter sweeps)
- Scratch storage optimization (they'll learn this when hitting I/O bottlenecks)

**Alex's challenges**:
- Unfamiliar with command line; needs extra patience with Episode 1
- Will make mistakes in batch scripts; needs encouragement to read error messages
- Might request too many resources initially; needs to see `seff` results

**Alex's success criteria**:
- "I can train my model on a GPU without my laptop overheating"
- "My job finishes in under an hour instead of 4 hours on my laptop"
- "I understand what the error messages mean and can fix them"

---

## Profile 3: Morgan - Economics Faculty Researcher

**Background**: Morgan is an associate professor of economics using statistical computing for large-scale data analysis. They have experience with R and STATA on their desktop but have never used any HPC system.

**Motivation**: "I'm analyzing 50 years of economic data with complex models. My department just got Sagehen access. I want to scale up my analysis without leaving my workflow."

**What Morgan needs**:
- Integration with existing R scripts (minimal rewriting)
- Understanding of parallel computing on CPUs
- Large memory requirements (data in RAM)
- Batch job submission without interactive development

**What Morgan will learn**:
- ✓ How to convert R scripts to batch jobs with minimal changes
- ✓ How to request large amounts of CPU and memory
- ✓ How to process the results back on their desktop
- ✓ How to manage multiple related jobs (perhaps related parameter sweeps)
- ✓ When to use scratch storage for large intermediate files

**Morgan might skip**:
- GPU training (their work is CPU-based statistics)
- Interactive jobs (they prefer writing complete scripts)
- Module loading details (they've already installed R locally)

**Morgan's challenges**:
- Expects SLURM to work like their desktop (different paradigm)
- May write overly-complex resource requests initially
- Might forget about data cleanup and hit storage quotas

**Morgan's success criteria**:
- "My analysis that took 2 weeks now runs in 2 hours using 64 cores"
- "I understand resource requirements well enough to not waste compute time"
- "I can manage 20+ jobs efficiently without overwhelming the cluster"

---

## Differentiated Instruction Based on Profiles

### For Sam (Chemistry Student)

**Emphasize**:
- Episode 4: Batch jobs (their primary workflow)
- Episode 5: Monitoring and efficiency (key for long simulations)
- Episode 7: Scratch storage (important for I/O)

**Example challenge**: "Write a batch script for GROMACS with --time=24:00:00"

**Extension**: "Run a job array with different temperatures or parameters"

### For Alex (Neuroscience Student)

**Emphasize**:
- Episode 3: Interactive jobs (for development)
- Episode 6: GPU jobs (critical for their ML work)
- Episode 5: Troubleshooting common errors

**Example challenge**: "Write a PyTorch training script, convert to batch job, monitor GPU with nvidia-smi"

**Extension**: "Implement checkpoint saving so you can resume long training runs"

### For Morgan (Economics Professor)

**Emphasize**:
- Episode 2: Resource planning (crucial for large data)
- Episode 4: Batch jobs with email notifications
- Episode 7: Advanced tips (job arrays, efficiency tuning)

**Example challenge**: "Estimate resources for analyzing your 50GB dataset, create batch script with multiple analysis steps"

**Extension**: "Design a job array to test 10 different model specifications"

---

## Common Struggles and How We Address Them

### Struggle: "SLURM Syntax is Hard"

**Solution for all profiles**:
- Provide batch script templates
- Emphasize the reference card
- Show "common patterns" section
- Encourage copying and modifying examples

### Struggle: "I Don't Know How Much to Request"

**Solution**:
- Teach estimation heuristics (2-4x input file size for memory, etc.)
- Show `seff` output
- Normalize iterative refinement
- Provide examples for different research types

### Struggle: "My Job Failed and I Don't Know Why"

**Solution**:
- Teach where to find error messages (output files)
- Provide a troubleshooting table
- Show exit codes and what they mean
- Encourage reading error messages carefully

### Struggle: "Everything is on the Command Line"

**Solution**:
- For GUI-oriented learners, show that this is normal in HPC
- Provide examples in their primary language (Python, R, MATLAB)
- Show results can be visualized on their laptop
- Emphasize: "You write code, submit it, get results back"

---

## Adjusting Pacing by Learner Type

| Learner Type | Time Spent | Episodes to Emphasize | Episodes to Skim |
|---|---|---|---|
| Sam (Domain expert, HPC novice) | 2 hours | 1-5, 7 | 6 (briefly) |
| Alex (Novice everywhere) | 3-4 hours | 1-6 with extra help | Extend with extra challenges |
| Morgan (Experienced researcher) | 1.5 hours | 2, 4-5, 7 | 1-3 (quickly) |

---

## Sample Workshop Dialogues

### If You Spot Sam Looking Confused

**Instructor**: "I see you're looking at the time format. It's HH:MM:SS, so 01:30:00 is one hour and thirty minutes. Think of it like a clock."

**Sam**: "Oh, so 24:00:00 is a full day?"

**Instructor**: "Exactly! And on the amd partition, you can go up to 30-00:00:00, which is 30 days. Try it in your script."

### If You Spot Alex Frustrated with an Error

**Instructor**: "Great! Your job failed, which is normal! Let's look at the error. What does the output file say?"

**Alex**: "[reads] 'Killed... out of memory'"

**Instructor**: "Perfect diagnosis. That means your program tried to use more RAM than you requested. Look at `seff` to see how much you actually used, then request a bit more. This is how we learn!"

### If You Spot Morgan Impatient to Optimize

**Instructor**: "You're thinking about job arrays and parallel processing. Great! For now, let's get one job running correctly. Then we'll optimize. In HPC, correctness comes before speed."

**Morgan**: "Fair point. I'm used to tweaking code locally; this feels different."

**Instructor**: "Exactly. Here, we plan upfront, then let the cluster do the work. You'll see the benefits when your 50GB dataset finishes in hours instead of days."

---

## Learning Outcomes by Profile

### Sam Learns...
- "I can estimate resources for molecular dynamics and avoid overrequesting"
- "When a job fails, I know how to read the error message and fix it"
- "I can achieve 10x speedup by properly using Sagehen"

### Alex Learns...
- "GPU acceleration is powerful but requires proper code setup"
- "The command line is less scary when you have a template to follow"
- "I can iterate on training/testing much faster on Sagehen"

### Morgan Learns...
- "Managing many jobs is straightforward with job arrays"
- "Large-scale analysis is feasible without special hardware at home"
- "Efficiency matters when using shared resources"

---

## Customizing This Workshop for Your Audience

When you teach this workshop:

1. **Ask learners to introduce themselves** at the start
2. **Identify which profile(s) match** your audience
3. **Emphasize the relevant episodes** and examples
4. **Adjust challenge complexity** (easy for beginners, hard for advanced)
5. **Provide post-workshop office hours** for one-on-one help

Remember: All three profiles (and others!) are valuable and welcome. The goal is to help them succeed with their research on Sagehen.

---

## For More Details

- See [Instructor Notes](../instructors/instructor-notes.md) for teaching strategy
- See individual episodes for specific learning objectives
- See [Setup Guide](setup.md) for account and access issues
