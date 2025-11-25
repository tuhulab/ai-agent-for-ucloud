# Computerome 2.0 HPC System (DTU)
## AI Copilot Instructions & Architecture Reference

**Platform**: Computerome 2.0 at Technical University of Denmark (DTU)  
**Document Version**: 1.0  
**Last Updated**: November 2025  
**Author**: [Tu Hu](https://github.com/tuhulab)

---

> **⚠️ Disclaimer**
>
> This document is provided solely by Tu Hu in a personal capacity. It is not affiliated with, endorsed by, or representative of DTU or Computerome. The content is provided "AS IS" without warranties or guarantees. Use at your own risk — you are solely responsible for any decisions or outcomes resulting from its use.

---

## Table of Contents

1. [Critical Pre-Execution Checks](#critical-pre-execution-checks)
2. [What NOT to Do](#what-not-to-do)
3. [Key Conventions](#key-conventions)
4. [System Overview](#system-overview)
5. [Cluster Resources](#cluster-resources)
6. [Job Submission (PBS/Torque)](#job-submission-pbstorque)
7. [Module System](#module-system)
8. [VS Code Remote Development](#vs-code-remote-development)
9. [Best Practices](#best-practices)
10. [Agent-Specific Guidelines](#agent-specific-guidelines)
11. [Troubleshooting](#troubleshooting)
12. [Quick Reference](#quick-reference)

---

## Critical Pre-Execution Checks

**⚠️ ALWAYS perform these checks first when starting work:**

### 1. Verify Current Environment
```bash
# Check where you are running (login node vs compute node)
hostname
# login01.computerome.local = login node (limited resources)
# g-XX-cXXXX = compute node (job environment)

# Check current working directory
pwd

# Check available disk space
df -h .
```

### 2. Check Module Availability
```bash
# Verify tools module is loaded
module list

# If 'tools' not loaded:
module load tools
```

### 3. Check Resource Availability
```bash
# For GPU tasks
nvidia-smi 2>/dev/null || echo "No GPU available on this node"

# For conda environments
conda --version 2>/dev/null || echo "Conda not available - load anaconda module"
```

### 4. Identify Project Context
```bash
# Check which project directory you're working in
echo $PWD | grep -o 'reg_[0-9]*\|cu_[0-9]*\|ku_[0-9]*' || echo "Not in a project directory"

# Check group memberships
id
```

---

## What NOT to Do

| ❌ Don't | ✅ Do Instead |
|----------|---------------|
| Write large files to `/home/people/<user>/` | Use project directory `/home/projects/<project>/` |
| Run compute-intensive tasks on login nodes | Submit PBS jobs |
| Hardcode absolute paths with usernames | Use `$USER`, `$PBS_O_WORKDIR` |
| Assume modules are loaded | Check `module list` first |
| Install conda environments in home directory | Use project directory |
| Leave large temporary files | Clean up after jobs |
| Submit jobs without `-A <project_id>` | Always specify account |
| Use `qsub --help` | Refer to documentation (it hangs) |
| Assume GPU availability | Request GPU nodes explicitly |

---

## Key Conventions

### Path Conventions
```bash
# Project working directory
/home/projects/<project_id>/people/<username>/

# Shared project data
/home/projects/<project_id>/data/

# Reference databases (system-wide)
/home/databases/
```

### Environment Variables to Set
```bash
export PROJECT_ID="reg_000xx"  # Your project
export WORK_DIR="/home/projects/$PROJECT_ID/people/$USER"
export TMPDIR="$WORK_DIR/tmp"
export XDG_CACHE_HOME="$WORK_DIR/.cache"
export SINGULARITY_CACHEDIR="$WORK_DIR/.singularity"
```

### Recommended Directory Structure
```
/home/projects/<project_id>/
├── apps/               # Shared project applications
├── data/               # Shared project data
├── people/
│   └── <user>/
│       ├── bin/        # User binaries
│       ├── conda_envs/ # Conda environments
│       ├── .cache/     # Cache directory
│       ├── tmp/        # Temporary files
│       └── <project>/  # Analysis projects
└── scratch/            # Project scratch space
```

---

## System Overview

| Property | Value |
|----------|-------|
| System Name | Computerome 2.0 |
| Operating System | Rocky Linux 8 |
| Job Scheduler | PBS/Torque 7.0.1 with Moab |
| Default Queue | `batch` |
| Login Node | `login01.computerome.local` |
| Support | computerome@dtu.dk |

---

## Cluster Resources

### Node Types

| Type | Count | CPUs | Memory | GPU |
|------|-------|------|--------|-----|
| Thin nodes | ~599 | 40 cores | ~188 GB | None |
| Fat nodes | ~50 | 40 cores | ~1.5 TB | None |
| GPU V100 | ~37 | 40 cores | ~188 GB | 1x Tesla V100 16GB |
| GPU A100 | ~32 | 20 cores | ~384 GB | 1x GRID A100D 20GB |

**Total**: ~721 nodes

### Storage

| Mount Point | Size | Purpose |
|-------------|------|---------|
| `/home/people/<user>` | ~10GB quota | Personal home (limited!) |
| `/home/projects/<project>` | 25 PB total | Project storage (main work area) |
| `/home/databases` | Shared | Reference databases |
| `/scratch` | ~390 GB | Node-local temporary |

---

## Job Submission (PBS/Torque)

### PBS Script Template

```bash
#!/bin/bash
#PBS -N job_name              # Job name
#PBS -l nodes=1:ppn=20        # 1 node, 20 cores
#PBS -l mem=80g               # Memory request
#PBS -l walltime=02:00:00     # Wall time
#PBS -A <project_id>          # Account (REQUIRED)
#PBS -o /path/to/output.log   # Stdout file
#PBS -e /path/to/error.log    # Stderr file
#PBS -j oe                    # Join stdout/stderr
#PBS -q batch                 # Queue name

cd $PBS_O_WORKDIR

module load tools
module load <software>/<version>

# Your commands here
```

### Common PBS Directives

| Directive | Description | Example |
|-----------|-------------|---------|
| `-N` | Job name | `-N my_analysis` |
| `-l nodes=X:ppn=Y` | Nodes and cores | `-l nodes=1:ppn=20` |
| `-l mem=Xg` | Memory | `-l mem=80g` |
| `-l walltime=HH:MM:SS` | Max runtime | `-l walltime=24:00:00` |
| `-A` | Project account | `-A reg_00041` |
| `-q` | Queue | `-q batch` |

### Requesting Special Nodes

```bash
# Fat node (high memory)
#PBS -l nodes=1:ppn=40:fatnode,mem=1400g,walltime=24:00:00

# GPU V100 node
#PBS -l nodes=1:ppn=40:gpuv1:gpus=1,mem=180g,walltime=12:00:00

# GPU A100 node
#PBS -l nodes=1:ppn=20:gpua1:gpus=1,mem=380g,walltime=12:00:00
```

### Interactive Sessions

```bash
# Basic interactive session
qsub -I -l nodes=1:ppn=4,mem=16g,walltime=02:00:00 -A <project_id>

# Interactive with GPU
qsub -I -l nodes=1:ppn=20:gpuv1:gpus=1,mem=80g,walltime=02:00:00 -A <project_id>
```

### PBS Environment Variables

| Variable | Description |
|----------|-------------|
| `$PBS_JOBID` | Job ID |
| `$PBS_JOBNAME` | Job name |
| `$PBS_O_WORKDIR` | Submission directory |
| `$PBS_NP` | Number of processors |

---

## Module System

### Basic Commands

```bash
module load tools              # Required for most software
module avail                   # List all modules
module avail <name>            # Search modules
module load <name>/<version>   # Load module
module list                    # List loaded
module purge                   # Unload all
```

### Common Modules

```bash
# Languages
module load python/3.12.0
module load R/4.4.0
module load anaconda3/2024.06-1

# Bioinformatics
module load samtools/1.22.1
module load bwa/0.7.17
module load star/2.7.10b
module load nextflow/24.10.3

# Containers
module load singularity/4.3.0
module load apptainer/1.4.0
```

---

## VS Code Remote Development

### Setting Up VS Code Tunnel

1. **Download VS Code CLI**:
```bash
mkdir -p /home/projects/<project>/people/$USER/bin
cd /home/projects/<project>/people/$USER/bin
curl -L "https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64" | tar -xz
```

2. **Create tunnel job script**:
```bash
#!/bin/bash
#PBS -N vscode_tunnel
#PBS -l nodes=1:ppn=20,mem=80g,walltime=04:00:00
#PBS -A <project_id>
#PBS -j oe

cd $PBS_O_WORKDIR
CODE_BIN="/home/projects/<project>/people/$USER/bin/code"

$CODE_BIN tunnel user login --provider github
$CODE_BIN tunnel --name <tunnel-name> --accept-server-license-terms \
    --cli-data-dir /home/projects/<project>/people/$USER/.vscode-cli
```

3. **Add to `.bashrc`**:
```bash
export VSCODE_SERVER_DIR="/home/projects/<project>/people/$USER/.vscode-server"
export VSCODE_CLI_DATA_DIR="/home/projects/<project>/people/$USER/.vscode-cli"
```

---

## Best Practices

### Storage Management

```bash
# Use project directories for all work
export WORK_DIR="/home/projects/$PROJECT_ID/people/$USER"

# Avoid home directory quota issues
export TMPDIR="$WORK_DIR/tmp"
export XDG_CACHE_HOME="$WORK_DIR/.cache"
```

### Conda Environments

```bash
module load anaconda3/2024.06-1

# Create in project directory (NOT home!)
conda create --prefix /home/projects/<project>/people/$USER/conda_envs/myenv python=3.10

# Activate
conda activate /home/projects/<project>/people/$USER/conda_envs/myenv
```

### Singularity Containers

```bash
module load singularity/4.3.0

export SINGULARITY_CACHEDIR="/home/projects/<project>/people/$USER/.singularity"

singularity exec --bind /home/projects:/home/projects container.sif <command>
```

---

## Agent-Specific Guidelines

### When Creating Scripts

```bash
# ✅ Good - use variables
cd $PBS_O_WORKDIR
OUTDIR="/home/projects/${PROJECT_ID}/people/${USER}/results"

# ❌ Bad - hardcoded paths
cd ~/results
```

### Error Handling

```bash
set -e  # Exit on first error
set -u  # Error on undefined variables
```

### Module Loading

```bash
# ✅ Good - explicit version
module load samtools/1.22.1

# ❌ Bad - version may change
module load samtools
```

### Command Availability

```bash
command -v samtools &>/dev/null || { echo "samtools not found"; exit 1; }
```

---

## Troubleshooting

### Decision Tree

```
Command not found?
├── Load 'tools' module: module load tools
├── Search for module: module avail <command>
└── Check PATH: which <command>

Permission denied?
├── Check file permissions: ls -la <file>
├── Check group membership: id
└── Verify project access: groups | grep <project_id>

Disk quota exceeded?
├── Check home quota: df -h /home/people/$USER
├── Move to project dir: mv <files> /home/projects/<project>/
└── Clean cache: rm -rf ~/.cache/*

Job won't start?
├── Check queue: qstat -Q
├── Check resources: Are you over-requesting?
└── Check account: Is -A <project_id> correct?
```

### Job Monitoring

```bash
qstat -u $USER          # Your jobs
qstat -f <job_id>       # Job details
checkjob <job_id>       # Why job is waiting
```

---

## Quick Reference

### Job Commands
```bash
qsub script.sh          # Submit job
qstat -u $USER          # Check your jobs
qdel <job_id>           # Cancel job
```

### Interactive Session
```bash
qsub -I -l nodes=1:ppn=4,mem=16g,walltime=02:00:00 -A <project>
```

### Module Commands
```bash
module load tools
module avail <search>
module load <name>/<version>
```

### Storage Checks
```bash
df -h /home/people/$USER           # Home quota
df -h /home/projects/<project>     # Project space
du -sh <directory>                 # Directory size
```

---

## Additional Resources

- **Computerome Wiki**: https://www.computerome.dk/wiki/user-guide
- **Support Email**: computerome@dtu.dk
- **PBS/Torque Documentation**: https://adaptivecomputing.com/cherry-services/torque-resource-manager/

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | November 2025 | Initial release |
