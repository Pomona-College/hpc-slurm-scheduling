---
title: Setup
---

## Getting Started with Sagehen

This page will guide you through setting up access to Sagehen, Pomona College's research computing cluster. By the end, you'll be able to connect and start running jobs.

## Prerequisites

Before we begin, you need:

1. **Pomona College credentials**: Your @pomona.edu email and password
2. **DUO authentication**: Access to your Pomona DUO account (for two-factor authentication)
3. **A computer with**: Terminal or SSH client (macOS/Linux have this built-in; Windows users can use PowerShell, WSL, or PuTTY)
4. **An HPC account**: Contact its-hpc@pomona.edu to request one if you don't have it

## Step 1: Request an HPC Account

If you don't already have access to Sagehen:

1. Email its-hpc@pomona.edu with:
   - Your full name
   - Your @pomona.edu email
   - Your department and advisor/PI name
   - Brief description of your research

2. The HPC team will set up your account (usually 1-2 business days)

3. You'll receive confirmation when your account is ready

## Step 2: Set Up SSH Access

SSH (Secure Shell) is how you connect to Sagehen. The connection address is:

```
sagehen.hpc.pomona.edu
```

### On macOS or Linux

1. Open Terminal
2. Connect to Sagehen:
   ```bash
   ssh <myusername>@sagehen.hpc.pomona.edu
   ```
   Replace `username` with your Pomona username (part before @pomona.edu)

3. First time connecting, you'll see:
   ```
   The authenticity of host 'sagehen.hpc.pomona.edu' can't be established.
   ECDSA key fingerprint is SHA256:...
   Are you sure you want to continue connecting (yes/no/[fingerprint])?
   ```
   Type `yes` and press Enter

### On Windows

**Option A: Use PowerShell (Windows 10 and newer)**

1. Open PowerShell (search for "PowerShell" in Start menu)
2. Connect to Sagehen:
   ```powershell
   ssh <myusername>@sagehen.hpc.pomona.edu
   ```

**Option B: Use WSL (Windows Subsystem for Linux)**

1. Install WSL: Follow Microsoft's WSL installation guide
2. Open WSL terminal
3. Connect to Sagehen:
   ```bash
   ssh <myusername>@sagehen.hpc.pomona.edu
   ```

**Option C: Use PuTTY**

1. Download PuTTY from https://www.putty.org/
2. Install and open PuTTY
3. In the "Host Name" field, enter: `sagehen.hpc.pomona.edu`
4. Leave Port as 22
5. Click "Open"
6. When prompted, type your username

## Step 3: Complete DUO Authentication

When you connect, you'll see:

```
Pomona DUO for Sagehen HPC
Duo Authentication

1. Push 'Approve' on your Pomona Duo app
2. Enter an authentication code

Passcode or option (1-3):
```

**Option 1: Use Duo Mobile App (Recommended)**

1. Open the Duo Mobile app on your phone/device
2. You'll see a push notification for "Pomona College"
3. Tap "Approve"
4. Wait for the app to confirm

**Option 2: Enter an Authentication Code**

If you don't have the app or can't use push:

1. Type `2` and press Enter to get an SMS code
2. Check your Pomona-registered phone for a text
3. Enter the 6-digit code when prompted

**Option 3: Backup Code**

If you've lost access to Duo, you can use a backup code (if you saved one earlier):

1. Type your backup code
2. Note: This uses up one of your limited backup codes

## Step 4: First Login Test

After authenticating with Duo, you should see:

```
[<myusername>@sagehen ~]$
```

Great! You're connected to the head node.

### Verify Your Environment

Run these commands to confirm everything is set up:

```bash
# Check your username
whoami

# Check which cluster
hostname

# List modules available
module avail

# View available partitions
sinfo
```

## Step 5: Configure Your Environment (Optional)

You can customize your shell environment by editing `~/.bashrc`:

```bash
nano ~/.bashrc
```

Useful additions:

```bash
# Set default editor
export EDITOR=nano

# Add helpful aliases
alias sq='squeue -u $USER'
alias sa='sacct -u $USER'
alias ll='ls -lah'

# Set up your favorite modules to auto-load
module load miniconda3  # Load by default
```

Save with Ctrl+O, Enter, Ctrl+X.

## Troubleshooting

### "Permission denied (publickey)"

**Problem**: SSH key issue or wrong credentials

**Solution**:
- Verify username is correct (not email, just the part before @)
- Example: if email is alice@pomona.edu, username is `alice`
- Check you're connecting to correct host: `sagehen.hpc.pomona.edu`

```bash
ssh alice@sagehen.hpc.pomona.edu
```

### "DUO authentication failed"

**Problem**: Two-factor authentication didn't work

**Solutions**:
- Confirm Duo app is installed and registered with Pomona
- Check phone/device for push notification (might be silent)
- Try SMS option (type `2` for SMS code)
- Verify phone number registered with Pomona is correct
- Contact its-hpc@pomona.edu if still having issues

### "Connection timeout" or "No route to host"

**Problem**: Can't reach sagehen.hpc.pomona.edu

**Solutions**:
- Check internet connection
- Verify you can reach other Pomona services (email, Canvas)
- If on off-campus network, you may need to use Pomona VPN
- Contact its-hpc@pomona.edu to verify your account is active

### "Command not found" after login

**Problem**: Software isn't installed globally

**Solution**: Use the module system. See Episode 4 for details.

```bash
module load miniconda3
python --version
```

## Next Steps

Once connected, you're ready to begin the workshop:

1. Start with [Episode 1: Introduction to SLURM](../episodes/01-intro-slurm.md)
2. Work through each episode in order
3. Complete the challenges to build your skills
4. Refer to the [Quick Reference Card](reference.md) for command syntax

## Getting Help

If you encounter issues:

1. **Technical problems**: Email its-hpc@pomona.edu
2. **Questions about this workshop**: Ask your instructor
3. **Job failures**: Check [Episode 5: Monitoring and Managing](../episodes/05-monitoring-managing.md) for troubleshooting

## Important Reminders

::::::::::::::::::::::::::::::::::::::: callout
## Sagehen is a Shared Resource

Remember:
- **Don't run jobs on the head node** (sagehen.hpc.pomona.edu) - use compute nodes
- **Close your connection when done**: Type `exit` or `logout`
- **Respect quota limits**: Check with `quota_check.sh`
- **Be a good neighbor**: Don't waste resources on tests, clean up temp files

:::::::::::::::::::::::::::::::::::::::::::::::

## Your First Session Checklist

- [ ] SSH connection works
- [ ] DUO authentication completes
- [ ] You can run `sinfo` to see cluster status
- [ ] You see the bash prompt: `[<myusername>@sagehen ~]$`
- [ ] You can load a module: `module load miniconda3`

Once all these work, proceed to the first episode!

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
