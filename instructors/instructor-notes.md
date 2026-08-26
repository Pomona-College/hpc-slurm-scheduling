---
title: Instructor Notes
---

## Teaching This Workshop

Welcome, instructor! This guide will help you teach the SLURM Job Scheduling workshop effectively. It includes timing, common issues, and pedagogical approaches.

## Overview

This is a 2-hour hands-on workshop designed for researchers at all levels who need to use Sagehen, Pomona's HPC cluster. Learners come away with practical skills to run interactive and batch jobs effectively.

**Total Contact Time**: ~2 hours
- Teaching: ~85 minutes
- Exercises: ~35 minutes
- Setup/Troubleshooting: ~10 minutes
- Buffer: ~10 minutes

## Learner Backgrounds

Expect a mix of:
- **New to HPC**: First-time users, may not know what a scheduler does
- **Some Linux experience**: Comfortable with command line but never used SLURM
- **Impatient to run code**: Want to know how to get their job running ASAP

The challenges should accommodate all levels. Beginners work through the basics; advanced learners can experiment with extensions.

## Episode-by-Episode Guidance

### Episode 1: Introduction to SLURM (15 min teaching)

**Learning Objectives**: Understand what a job scheduler is and why Sagehen has this structure.

**Key Points to Emphasize**:
- The head node (sagehen) is **not** for running jobs; it only has 2 CPUs and will crash if someone runs a big job there
- SLURM is just resource coordination; it's not magic
- Analogies help: SLURM is like an airport scheduler assigning gates, jobs are like planes, compute nodes are like gates

**Common Misconceptions**:
- "Can't I just SSH to a node and run my job?" → No, then you bypass accounting and can mess up the cluster
- "The head node looks powerful" → No, those resources are for the OS and login management
- "SLURM is hard to learn" → Actually very logical once you understand the metaphor

**Challenge Advice**:
- Challenge: Exploring Sagehen
  - This requires SSH access. Have 1-2 learners try while others listen
  - If account issues arise, proceed with screenshare demo
  - Key output to show: nodes in different states (idle, alloc), three partitions visible
  - Don't worry about understanding the output format; we explain it in later episodes

**Timing**: 15 min teaching + 5 min challenge = 20 min total

---

### Episode 2: Partitions and Resources (15 min teaching)

**Learning Objectives**: Choose appropriate resources and partitions for different jobs.

**Key Points to Emphasize**:
- Three partitions exist for different purposes: `amd` (general compute, default for most jobs), `gpu` (GPU-accelerated work), and `short` (quick test/debug jobs with shorter walltime)
- Resource requests are educated guesses; you adjust based on experience
- GPU requests require actually needing a GPU
- Group limits exist for fairness, not to limit individuals

**Live Interaction Idea**:
- Show `sinfo` output and have learners identify nodes, partitions, status
- Predict: "If I want a 1-hour Python analysis, what should I request?" → Discuss

**Challenge Advice**:
- Challenge 1: Matching jobs to partitions
  - These are scenarios; discuss aloud
  - Main insight: GPU work → `gpu`; quick test/debug jobs → `short`; everything else → `amd`
- Challenge 2: Resource estimation
  - These are estimates; discuss why each is reasonable
  - Emphasize: "You'll refine these over time by looking at `seff`"

**Common Issues**:
- "Why can't I request 2TB of RAM?" → Node only has 500GB
- "Is -c the same as --cpus-per-task?" → Yes, -c is shorthand (use either)

**Timing**: 15 min teaching + 10 min challenges = 25 min total

---

### Episode 3: Interactive Jobs (15 min teaching)

**Learning Objectives**: Hands-on experience running code immediately on a compute node.

**Key Points to Emphasize**:
- Interactive jobs are for development/debugging, not production
- `srun --pty bash -l` is your magic formula for interactive work
- You're on a different hostname now (a001, not sagehen)
- Time limits still apply; if you exceed them, you're kicked off
- Use the `short` partition for short interactive sessions and quick debugging; use `amd` for longer interactive work

**Live Demo**:
Do this while explaining:
```bash
srun -c 2 --mem=8G -t 00:10:00 -p short --pty bash -l
# Wait for allocation
hostname   # Show you're on a compute node now
module load miniconda3
python -c "import sys; print(sys.version)"
exit       # Back to head node
```

**Challenge Advice**:
- Challenge 1: Your First Interactive Session
  - Most important is that they experience the allocation → connection flow
  - Some might have account issues; have backups ready
  - Success criteria: They run a command on a different node and see the hostname change

- Challenge 2: Interactive with Code
  - This is more realistic; they create a script and run it
  - If nano/vi are unfamiliar, show basics
  - Point out: "See how you could edit and rerun instantly? That's why interactive is great for development"

- Challenge 3: GPU Interactive (if available)
  - Most learners won't reach this in workshop time; it's for after
  - But if GPU nodes are available, let one or two try during break

**Timing**: 15 min teaching + 15 min challenges = 30 min total

---

### Episode 4: Batch Jobs (20 min teaching)

**Learning Objectives**: Write production-ready batch scripts that run unattended.

**Key Points to Emphasize**:
- Batch scripts are the "production" path
- The shebang and `#SBATCH` lines must be perfect or they'll fail silently
- Always use `-l` flag (login shell)
- Always load modules in the script (not just in your shell)
- Output files are your debugging tool when things go wrong

**Live Demo**:
Create a minimal example:
```bash
cat > simple.sh << 'EOF'
#!/bin/bash -l
#SBATCH --job-name=test
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=00:02:00
#SBATCH --output=test_%j.out

echo "Hello from $(hostname)"
EOF

sbatch simple.sh
squeue -u $USER
```

Then when complete, `cat test_12345.out` to show output file.

**Challenge Advice**:
- Challenge 1: Simple Batch Script
  - Learners should be able to write this themselves now
  - First success: seeing "Hello from a00X" in output file
  - If script fails, walk through: check output file, look for error messages

- Challenge 2: Python in Batch
  - Show the numpy import and computation are realistic
  - Emphasize: "This would fail if you forgot `module load miniconda3`"
  - Ask: "Would this have worked on your laptop?" → Probably slower

- Challenge 3: GPU Batch (only if GPU partition available)
  - This is advanced; save for advanced learners or after workshop

**Common Errors to Preempt**:
- Missing `#!/bin/bash -l` → Script fails silently
- Typo in `#SBATCH --cpus-per_task` → SLURM ignores the line, uses default
- Output file not found → Check spelling, check working directory

**Timing**: 20 min teaching + 15 min challenges = 35 min total

---

### Episode 5: Monitoring and Managing (15 min teaching)

**Learning Objectives**: Track jobs and understand efficiency.

**Key Points to Emphasize**:
- `squeue` is your current queue lookup, `sacct` is historical
- `seff` tells you if you're wasting resources
- `scancel` is your "kill switch" but use carefully
- Exit code 0 = success, non-zero = error
- Exit code 138 = timeout (common mistake)

**Live Demo**:
```bash
# Submit a job
sbatch --time=10:00 simple.sh

# Immediately check
squeue -u $USER

# After it finishes, check history
sacct -u $USER | tail -3

# Analyze efficiency
seff JOBID
```

**Challenge Advice**:
- Challenge 1: Job Monitoring
  - They submit a real job and watch it progress
  - Use this to show efficiency; point out low values are expected for `sleep` job

- Challenge 2: Cancellation
  - They practice cancelling (safely)
  - Emphasize: "This is irreversible; double-check job ID"
  - Point out: exit code shows CANCELLED, not an error

**Timing**: 15 min teaching + 10 min challenges = 25 min total

---

### Episode 6: GPU Jobs (10 min teaching)

**Learning Objectives**: Understand when/how to use GPUs; avoid wasting them.

**Key Points to Emphasize**:
- GPUs are expensive; only use if your code actually benefits
- `--gres=gpu:1` is how you request GPUs
- `nvidia-smi` is your GPU diagnostic tool
- The three GPU types on Sagehen HPC have different characteristics
- Not all jobs have access to GPU partition (verify first)

**Live Demo (if GPU access available)**:
```bash
# Request interactive GPU session
srun --gres=gpu:l40s:1 -c 2 --mem=4G -t 00:05:00 -p gpu --pty bash -l

# Check GPU is available
module load cuda
nvidia-smi
```

**Challenge Advice**:
- Challenge 1: Simple GPU Computation
  - This works only if learners have GPU access
  - Output shows GPU name and computation time
  - Main insight: GPU is allocated but you need code to use it

- Challenge 2: PyTorch Training
  - Advanced; likely only 1-2 learners will do this
  - Success is when PyTorch runs and uses GPU
  - Error: CUDA out of memory → reduce batch size

**Note**: If GPU partition not available to learners, skip practical challenges but still explain concepts.

**Timing**: 10 min teaching + 10 min challenges (if GPU available) = 20 min

---

### Episode 7: Advanced Tips (10 min teaching)

**Learning Objectives**: Understand optimization techniques, job arrays, and troubleshooting.

**Key Points to Emphasize**:
- Scratch storage is fast; use it for I/O-heavy work
- Job arrays submit many jobs efficiently
- Email carefully (don't spam yourself)
- Common errors and their fixes are in Episode 5
- Always validate your output exists and is reasonable

**Don't overload learners with all topics**:
- Focus on 1-2 topics: scratch storage and job arrays are most useful
- Advanced tips can be reference material

**Challenge Advice**:
- Challenge 1: Scratch Benchmark
  - Shows real performance difference
  - Takes a few seconds to run
  - Insight: Scratch is noticeably faster

- Challenge 2: Job Array
  - Demonstrates submitting multiple jobs at once
  - Teaches use of `$SLURM_ARRAY_TASK_ID`
  - Output shows 5 separate jobs with different parameters

**Timing**: 10 min teaching + 5 min challenges = 15 min

---

## Scheduling and Pacing

**2-hour workshop outline**:

| Time | Content | Duration |
|------|---------|----------|
| 0:00-0:20 | Episode 1: Intro to SLURM | 20 min |
| 0:20-0:45 | Episode 2: Partitions | 25 min |
| 0:45-1:15 | Episode 3: Interactive Jobs | 30 min |
| 1:15-1:35 | Episode 4: Batch Jobs (first 15 min) | 20 min |
| 1:35-1:50 | Episode 5: Monitoring (abbreviated) | 15 min |
| 1:50-2:00 | Wrap-up, Q&A, next steps | 10 min |

**Full day workshop** (3-4 hours):
- Teach all 12 episodes in order
- Do 1-2 challenges per episode
- Allow more hands-on time
- Time for one-on-one troubleshooting

## Technology Setup

### What You Need

1. **Sagehen access**: You must have an HPC account (contact ITS in advance)
2. **Screen sharing**: Have a terminal open showing live interactions
3. **Backup demo**: Prepare screenshots if live demos fail
4. **Learner setup**: Ensure all learners have accounts before workshop (or use shared demo account)

### Testing Before Workshop

Day before, verify:
- [ ] Can SSH to sagehen successfully
- [ ] Can allocate interactive job
- [ ] Can submit batch job
- [ ] Can view job with squeue/sacct
- [ ] Modules load correctly
- [ ] GPU nodes available (if teaching Episode 6)

### Troubleshooting Common Setup Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "Permission denied" on SSH | Account not active | Email its-hpc@pomona.edu to activate |
| DUO fails | Not enrolled | User must register for Pomona Duo first |
| Module load fails | Wrong environment | Use `source ~/.bashrc` or logout/login |
| Jobs won't allocate | Quota exceeded | Wait for other jobs to finish, or use fewer resources |
| GPU not detected | Not requesting GPU | Add `--gres=gpu:1` to command |

## Pedagogical Tips

### Building Understanding

**For complete beginners**:
- Start with the airport/queue analogy
- Show sinfo output early and explain what each column means
- Have them run commands themselves; watching isn't enough

**For experienced Linux users**:
- They'll grasp concepts quickly; challenge them with resource estimation
- Point them to Episode 7 for optimization
- After workshop, suggest they try job arrays and parameter sweeps

**For researchers with prior HPC experience** (SGE, PBS, etc.):
- They'll notice similarities to their old scheduler
- Highlight SLURM-specific syntax: `#SBATCH` vs `#PBS`
- They can skip Episodes 1-2 and jump to Episode 3

### Engagement Strategies

1. **Make it relevant**: Ask about their research and what they'll compute
2. **Live demos**: Not just slides; actually run commands
3. **Mistakes are good**: When something fails, use it as teaching moment
4. **Peer learning**: Have participants help each other with challenges
5. **Celebrate success**: "Great! You successfully submitted your first job!"

### Handling Errors

When a command fails during live demo:

1. **Don't panic**: Normal; teaches debugging
2. **Read the error**: Show learners how to interpret error messages
3. **Think aloud**: "The error says 'resource not available'; that means... let's check what's running"
4. **Fix it**: Try a different approach; show flexibility
5. **Learn from it**: "This is why we always check `squeue` first"

## Assessment

### Informal Assessment (Throughout)

- Watch learners do challenges
- Ask questions: "Why did you request 16GB instead of 8GB?"
- Observe: Can they navigate file systems? Write scripts?

### Formal Assessment (Optional)

At end of workshop, ask learners to:
1. Submit an interactive job with specific resources
2. Create a batch script that runs their own code
3. Monitor and interpret job efficiency with `seff`

Success criteria:
- [ ] Job submitted successfully
- [ ] Job allocated on compute node (not head node)
- [ ] Output file created with results
- [ ] Learner can explain resource choices

## Extended Workshops

If you have more time (4+ hours):

**Add these activities**:
- **Your own research**: Have each learner plan how they'd run their research on Sagehen
- **Optimization exercise**: Submit jobs with wrong resources, then use `seff` to optimize
- **Troubleshooting lab**: Provide broken scripts; learners fix them
- **Parameter sweep**: Design a real parameter sweep for their research

**Assign homework**:
- Submit one real batch job
- Check efficiency with `seff`
- Report back on what they'd optimize next time

## Common Questions Learners Ask

| Q | A |
|---|---|
| "Can I run my job directly on sagehen.hpc.pomona.edu?" | No, use `srun` or `sbatch` to run on compute nodes |
| "How long until my job starts?" | Depends on queue; `amd` minutes to hours, `gpu` variable, `short` typically fastest because the scheduler can back-fill quickly |
| "How do I cancel a job that's already running?" | `scancel JOBID`; output files written so far are kept |
| "Why did my job timeout?" | You requested too little time. Check `seff` to see how long it actually needed |
| "How do I share my results with lab-mates?" | Use `/bigdata/lab/<labname>` shared storage |
| "Can I use a GPU for my Python code?" | Only if code uses PyTorch, TensorFlow, CuPy, or similar. Check with `nvidia-smi` |
| "Can I run multiple jobs on one GPU?" | Not currently; future versions will support sharing |
| "What if I hit my group's core limit?" | Your new jobs wait in queue until others finish |

## Feedback and Iteration

After each workshop:

1. **Collect feedback**: Ask learners what was most useful, what confused them
2. **Note issues**: What problems arose? Update troubleshooting section
3. **Iterate**: Adjust teaching order, add examples, cut content that didn't land
4. **Contact support**: Email its-hpc@pomona.edu with suggestions for cluster improvements

## Instructor Resources

### Key Contacts

- **HPC Support**: its-hpc@pomona.edu
- **Cluster Admin**: Andrew Wilson, Director of Research Computing, ITS
- **Emergency**: If cluster is down, email its-hpc@pomona.edu

### References

- SLURM official documentation: https://slurm.schedmd.com/
- Sagehen documentation (internal): Check with ITS
- The Carpentries: https://carpentries.org/

### Teaching Tips from Experience

- **Start with a win**: In Episode 3, everyone successfully allocates an interactive job. Celebrate this!
- **The "amd" partition is the workhorse**: Use it for production demos. Use the `short` partition for any quick test/debug demo so it gets scheduled fastest.
- **Show real output**: Learners connect better with real job output than synthetic examples
- **Don't teach job arrays in a 2-hour workshop**: Save for advanced session. Stick to core episodes 1-5
- **Have a rubber duck**: When learners get stuck, talking through it with someone (or something) helps debug
- **Normalize failures**: "I forget the syntax for time too; let me check the reference card"

---

**Good luck teaching! Your learners will leave ready to harness Sagehen's power.**

For updates or corrections to these notes, contact its-hpc@pomona.edu.

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
