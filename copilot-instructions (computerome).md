# Computerome 2.0 HPC System - Copilot Instructions

This document provides comprehensive guidance for AI agents and developers working on the Computerome 2.0 HPC system at DTU (Technical University of Denmark).

---

## ⚠️ CRITICAL: Before Executing ANY Task

**ALWAYS perform these checks first when starting work:**

### 1. Verify Current Environment
```bash
# Check where you are running (login node vs compute node)
hostname
# If hostname is login01.computerome.local - you're on login node (limited resources)
# If hostname is like g-XX-cXXXX - you're on a compute node (job environment)

# Check current working directory
pwd

# Check available disk space in working directory
df -h .
```

### 2. Check Module Availability
```bash
# Verify tools module is loaded (required for most software)
module list

# If 'tools' not loaded:
module load tools
```

### 3. Check Resource Availability (if needed)
```bash
# For GPU tasks - verify GPU access
nvidia-smi 2>/dev/null || echo "No GPU available on this node"

# For conda environments
conda --version 2>/dev/null || echo "Conda not available - load anaconda module"
```

### 4. Identify Project Context
```bash
# Check which project directory you're working in
echo $PWD | grep -o 'reg_[0-9]*\|cu_[0-9]*\|ku_[0-9]*' || echo "Not in a project directory"

# Check group memberships (for permission issues)
id
```

---

## 🚫 What NOT to Do

1. **NEVER write large files to `/home/people/<user>/`** - Home quota is ~10GB
2. **NEVER run compute-intensive tasks on login nodes** - Use PBS jobs
3. **NEVER hardcode absolute paths with usernames** - Use `$USER`, `$PBS_O_WORKDIR`
4. **NEVER assume modules are loaded** - Always check `module list` first
5. **NEVER install conda environments in home directory** - Use project directory
6. **NEVER leave large temporary files** - Clean up after jobs
7. **NEVER submit jobs without `-A <project_id>`** - Jobs will fail
8. **NEVER use `qsub --help`** - It hangs; refer to this documentation instead
9. **NEVER assume GPU availability** - Must request GPU nodes explicitly

---

## 🔑 Key Conventions for This System

### Path Conventions
```bash
# Project working directory pattern
/home/projects/<project_id>/people/<username>/

# Shared project data
/home/projects/<project_id>/data/

# Project applications/tools
/home/projects/<project_id>/apps/

# Reference databases (system-wide)
/home/databases/
```

### Environment Variables to Set
```bash
# CRITICAL: Set these to avoid home quota issues
export PROJECT_ID="reg_000xx"  # Change to your project
export WORK_DIR="/home/projects/$PROJECT_ID/people/$USER"
export TMPDIR="$WORK_DIR/tmp"
export XDG_CACHE_HOME="$WORK_DIR/.cache"
export SINGULARITY_CACHEDIR="$WORK_DIR/.singularity"
```

### File Organization Pattern
```
/home/projects/<project_id>/
├── apps/           # Shared project applications
├── data/           # Shared project data
├── people/
│   └── <user>/
│       ├── bin/            # User binaries (e.g., VS Code CLI)
│       ├── conda_envs/     # Conda environments
│       ├── .cache/         # Cache directory
│       ├── .singularity/   # Singularity cache
│       ├── tmp/            # Temporary files
│       └── <project_name>/ # Analysis projects
└── scratch/        # Project scratch space
```

---

## System Overview

- **System Name**: Computerome 2.0
- **Operating System**: Rocky Linux 8 (kernel 4.18.0-553.x)
- **Job Scheduler**: PBS/Torque 7.0.1 with Moab
- **Default Queue**: `batch`
- **Login Node**: `login01.computerome.local`
- **Contact**: computerome@dtu.dk

## Cluster Resources

### Node Types

| Type | Count | CPUs | Memory | GPU | Properties |
|------|-------|------|--------|-----|------------|
| Thin nodes | ~599 | 40 cores (2x Intel Xeon Gold 6230 @ 2.10GHz) | ~188 GB | None | `thinnode,compute` |
| Fat nodes | ~50 | 40 cores | ~1.5 TB | None | `fatnode,compute` |
| GPU V100 nodes | ~37 | 40 cores | ~188 GB | 1x Tesla V100 16GB | `gpuv1,compute,gmem16g` |
| GPU A100 nodes | ~32 | 20 cores | ~384 GB | 1x GRID A100D 20GB | `gpua1,compute,gmem20g` |

**Total nodes**: ~721

### Storage

| Mount Point | Type | Size | Purpose |
|-------------|------|------|---------|
| `/home/people/<user>` | NFS | Shared (limited quota ~10GB) | Personal home directory |
| `/home/projects/<project>` | NFS | 25 PB total | Project storage (main working area) |
| `/home/databases` | NFS | Shared | Shared reference databases |
| `/scratch` | Local XFS | ~390 GB | Node-local scratch (temporary) |
| `/services/tools` | NFS | Shared | Software installations |

### Important Storage Notes

- **Home directory quota is limited (~10 GB)** - Store large data in project directories
- Project directories have automatic snapshots in `.snapshots/` folder
- Snapshots are retained hourly (recent) and weekly (older)
- Use `/scratch` for temporary files during job execution (cleaned after job)

## Job Submission (PBS/Torque)

### PBS Script Template

```bash
#!/bin/bash
#PBS -N job_name              # Job name
#PBS -l nodes=1:ppn=20        # 1 node, 20 cores
#PBS -l mem=80g               # Memory request
#PBS -l walltime=02:00:00     # Wall time (HH:MM:SS)
#PBS -A <project_id>          # Account/project (e.g., reg_00041)
#PBS -o /path/to/output.log   # Standard output file
#PBS -e /path/to/error.log    # Standard error file
#PBS -j oe                    # Join stdout and stderr
#PBS -q batch                 # Queue name

# Change to working directory
cd $PBS_O_WORKDIR

# Load required modules
module load tools
module load <software>/<version>

# Your commands here
```

### Common PBS Directives

| Directive | Description | Example |
|-----------|-------------|---------|
| `-N` | Job name | `-N my_analysis` |
| `-l nodes=X:ppn=Y` | Request X nodes with Y processors per node | `-l nodes=1:ppn=20` |
| `-l mem=Xg` | Memory request | `-l mem=80g` |
| `-l walltime=HH:MM:SS` | Maximum runtime | `-l walltime=24:00:00` |
| `-A` | Account/project | `-A reg_00041` |
| `-o` | Output file path | `-o /path/output.log` |
| `-e` | Error file path | `-e /path/error.log` |
| `-j oe` | Join output and error | `-j oe` |
| `-q` | Queue name | `-q batch` |

### Requesting Specific Node Types

```bash
# Request a fat node (high memory)
#PBS -l nodes=1:ppn=40:fatnode,mem=1400g,walltime=24:00:00

# Request a GPU V100 node
#PBS -l nodes=1:ppn=40:gpuv1:gpus=1,mem=180g,walltime=12:00:00

# Request a GPU A100 node
#PBS -l nodes=1:ppn=20:gpua1:gpus=1,mem=380g,walltime=12:00:00
```

### Interactive Sessions

```bash
# Basic interactive session
qsub -I -l nodes=1:ppn=4,mem=16g,walltime=02:00:00 -A <project_id>

# Interactive with GPU
qsub -I -l nodes=1:ppn=20:gpuv1:gpus=1,mem=80g,walltime=02:00:00 -A <project_id>
```

### Job Management Commands

| Command | Description |
|---------|-------------|
| `qsub script.sh` | Submit a job |
| `qstat` | Show all jobs |
| `qstat -u $USER` | Show your jobs |
| `qstat -f <job_id>` | Show job details |
| `qdel <job_id>` | Delete/cancel a job |
| `qstat -Q` | Show queue status |
| `pbsnodes -a` | Show all nodes |

### PBS Environment Variables

Available inside a running job:

| Variable | Description |
|----------|-------------|
| `$PBS_JOBID` | Job ID |
| `$PBS_JOBNAME` | Job name |
| `$PBS_NODEFILE` | File containing allocated nodes |
| `$PBS_O_WORKDIR` | Directory where job was submitted |
| `$PBS_NP` | Number of processors allocated |
| `$PBS_NUM_PPN` | Processors per node |
| `$PBS_WALLTIME` | Requested walltime in seconds |

## Module System

The system uses Environment Modules for software management.

### Basic Module Commands

```bash
module load tools              # Required to access most software
module avail                   # List all available modules
module avail <name>            # Search for specific module
module load <name>/<version>   # Load a module
module unload <name>           # Unload a module
module list                    # List loaded modules
module purge                   # Unload all modules
```

### Commonly Used Modules

#### Programming Languages
```bash
module load python/3.12.0
module load R/4.4.0
module load anaconda3/2024.06-1
```

#### Bioinformatics Tools
```bash
# Alignment
module load bwa/0.7.17
module load bowtie2/2.5.4
module load star/2.7.10b
module load hisat2/2.2.1

# SAM/BAM Processing
module load samtools/1.22.1
module load bcftools/1.22
module load bedtools/2.31.1

# RNA-seq
module load salmon/1.10.2
module load kallisto/0.46.0

# Single-cell/Spatial
module load cellranger/6.1.2
# Note: spaceranger may need to be installed in project directory

# Variant Calling
module load gatk/4.1.9.0

# Workflow Managers
module load nextflow/24.10.3
module load snakemake/8.20.5
```

#### Containers
```bash
module load singularity/4.3.0
module load apptainer/1.4.0
```

#### Utilities
```bash
module load git/2.40.0
module load cmake/3.27.6
```

## VS Code Remote Development

### Setting Up VS Code Tunnel

Since direct SSH may have limitations, use VS Code tunnels for remote development:

1. Download VS Code CLI to your project directory:
```bash
mkdir -p /home/projects/<project>/people/<user>/bin
cd /home/projects/<project>/people/<user>/bin
curl -L "https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64" | tar -xz
```

2. Create a tunnel job script:
```bash
#!/bin/bash
#PBS -N vscode_tunnel
#PBS -l nodes=1:ppn=20,mem=80g,walltime=04:00:00
#PBS -A <project_id>
#PBS -o /home/projects/<project>/people/<user>/vscode_tunnel.out
#PBS -e /home/projects/<project>/people/<user>/vscode_tunnel.err
#PBS -j oe

cd $PBS_O_WORKDIR

CODE_BIN="/home/projects/<project>/people/<user>/bin/code"

$CODE_BIN tunnel user login --provider github
$CODE_BIN tunnel --name <tunnel-name> --accept-server-license-terms \
    --cli-data-dir /home/projects/<project>/people/<user>/.vscode-cli
```

3. Add to `.bashrc` to avoid home directory quota issues:
```bash
export VSCODE_SERVER_DIR="/home/projects/<project>/people/<user>/.vscode-server"
export VSCODE_CLI_DATA_DIR="/home/projects/<project>/people/<user>/.vscode-cli"
export TMPDIR=/home/projects/<project>/people/<user>/tmp
export XDG_CACHE_HOME=/home/projects/<project>/people/<user>/.cache
```

## Best Practices

### Storage Management

1. **Use project directories** for all data and analysis work
2. **Avoid home directory** for large files (limited quota)
3. **Use `/scratch`** for temporary files during jobs
4. **Clean up** intermediate files after analysis

### Job Submission

1. **Request appropriate resources** - don't over-request
2. **Use walltime estimates** - jobs exceeding walltime are killed
3. **Test with small datasets** before full runs
4. **Use job arrays** for parallel processing of multiple files

### Environment Setup

Recommended `.bashrc` additions:
```bash
# Load basic tools
module load tools git/2.40.0

# Set project-specific paths (adjust project ID)
export PROJECT_DIR="/home/projects/<project_id>"
export WORK_DIR="$PROJECT_DIR/people/$USER"

# Avoid home directory quota issues
export TMPDIR="$WORK_DIR/tmp"
export XDG_CACHE_HOME="$WORK_DIR/.cache"
export SINGULARITY_CACHEDIR="$WORK_DIR/.singularity"
export NXF_SINGULARITY_CACHEDIR="$WORK_DIR/.singularity"

# Custom binaries
export PATH="$WORK_DIR/bin:$PATH"
```

### Conda/Mamba Environments

```bash
# Load anaconda
module load anaconda3/2024.06-1

# Create environment in project directory (not home!)
conda create --prefix /home/projects/<project>/people/<user>/conda_envs/myenv python=3.10

# Activate
conda activate /home/projects/<project>/people/<user>/conda_envs/myenv
```

### Singularity/Apptainer Containers

```bash
module load singularity/4.3.0

# Set cache directory to avoid home quota issues
export SINGULARITY_CACHEDIR="/home/projects/<project>/people/<user>/.singularity"

# Pull and run container
singularity pull docker://ubuntu:latest
singularity exec ubuntu_latest.sif <command>

# Run with binding project directory
singularity exec --bind /home/projects:/home/projects container.sif <command>
```

## Reference Databases

Common databases available in `/home/databases/`:

- `/home/databases/alphafold` - AlphaFold databases
- `/home/databases/blastdb` - BLAST databases
- `/home/databases/ctat_resource_lib` - CTAT resources for fusion detection
- `/home/databases/eggnog-mapper` - eggNOG-mapper databases

## Troubleshooting

### Common Issues

1. **Disk quota exceeded**: Move data to project directory, clean cache
2. **Job stuck in queue**: Check `qstat -Q` for queue status, verify resource requests
3. **Module not found**: Run `module load tools` first
4. **Permission denied**: Check group membership with `id` command

### Checking Job Status

```bash
# Check your jobs
qstat -u $USER

# Check job details
qstat -f <job_id>

# Check why job is waiting
checkjob <job_id>
```

### Getting Help

- Email: computerome@dtu.dk
- Wiki: https://www.computerome.dk/wiki/user-guide

## Quick Reference Card

```bash
# Submit job
qsub script.sh

# Check jobs
qstat -u $USER

# Cancel job
qdel <job_id>

# Interactive session
qsub -I -l nodes=1:ppn=4,mem=16g,walltime=02:00:00 -A <project>

# Load modules
module load tools
module load <software>/<version>

# Check available modules
module avail <search_term>
```

---

## 🤖 Agent-Specific Guidelines

### When Creating Scripts

1. **Always use absolute paths or proper variables**
   ```bash
   # Good
   cd $PBS_O_WORKDIR
   OUTDIR="/home/projects/${PROJECT_ID}/people/${USER}/results"
   
   # Bad
   cd ~/results
   OUTDIR="results"
   ```

2. **Always include error handling**
   ```bash
   set -e  # Exit on first error
   set -u  # Error on undefined variables
   ```

3. **Always specify module versions explicitly**
   ```bash
   # Good
   module load samtools/1.22.1
   
   # Bad - version may change
   module load samtools
   ```

### When Running Commands

1. **Check command availability before running**
   ```bash
   command -v samtools &>/dev/null || { echo "samtools not found"; exit 1; }
   ```

2. **Use `module avail <name>` to find available software** - not `ml search`

3. **For long-running commands, consider PBS jobs** instead of running directly

### When Debugging Issues

1. **Check job output files first**
   ```bash
   cat /path/to/job.out
   cat /path/to/job.err
   ```

2. **Check disk quota if seeing write errors**
   ```bash
   df -h /home/people/$USER
   df -h /home/projects/<project_id>
   ```

3. **Check module conflicts**
   ```bash
   module list
   module purge
   module load tools
   ```

### When Working with Data

1. **Always check file sizes before operations**
   ```bash
   ls -lh <file>
   du -sh <directory>
   ```

2. **Use `/scratch` for temporary files in jobs**
   ```bash
   SCRATCH_DIR="/scratch/$USER/$PBS_JOBID"
   mkdir -p $SCRATCH_DIR
   # ... do work ...
   rm -rf $SCRATCH_DIR
   ```

3. **Compress large output files**
   ```bash
   gzip large_output.txt
   ```

---

## 📋 Common Task Patterns

### Pattern: Submit and Monitor a Job
```bash
# 1. Create job script
cat > my_job.sh << 'EOF'
#!/bin/bash
#PBS -N my_analysis
#PBS -l nodes=1:ppn=20,mem=80g,walltime=04:00:00
#PBS -A reg_00041
#PBS -j oe

cd $PBS_O_WORKDIR
module load tools
module load samtools/1.22.1

# Your commands here
samtools view -b input.sam > output.bam
EOF

# 2. Submit
JOB_ID=$(qsub my_job.sh)
echo "Submitted job: $JOB_ID"

# 3. Monitor
qstat -u $USER
```

### Pattern: Set Up Conda Environment
```bash
# Load anaconda
module load tools
module load anaconda3/2024.06-1

# Create environment in project directory
ENV_PATH="/home/projects/reg_00041/people/$USER/conda_envs/myenv"
conda create --prefix $ENV_PATH python=3.10 -y

# Activate
conda activate $ENV_PATH

# Install packages
conda install numpy pandas scikit-learn -y
```

### Pattern: Run Singularity Container
```bash
# Set cache directory
export SINGULARITY_CACHEDIR="/home/projects/reg_00041/people/$USER/.singularity"
mkdir -p $SINGULARITY_CACHEDIR

# Load singularity
module load tools
module load singularity/4.3.0

# Pull image
singularity pull docker://biocontainers/samtools:v1.9-4-deb_cv1

# Run with proper bindings
singularity exec \
  --bind /home/projects:/home/projects \
  --bind /home/databases:/home/databases \
  samtools_v1.9-4-deb_cv1.sif \
  samtools --version
```

### Pattern: VS Code Tunnel Session
```bash
# Submit tunnel job
qsub /home/projects/reg_00041/people/$USER/start_vscode_tunnel.sh

# Check output for authentication link
tail -f /home/projects/reg_00041/people/$USER/vscode_tunnel.out

# Connect via VS Code: Remote Tunnels extension
```

---

## 🔧 Troubleshooting Decision Tree

```
Problem: Command not found
├── Did you load 'tools' module? → module load tools
├── Did you load specific module? → module avail <command>
└── Is it in PATH? → which <command> or command -v <command>

Problem: Permission denied
├── Check file permissions → ls -la <file>
├── Check group membership → id
└── Check if in correct project → groups | grep <project_id>

Problem: Disk quota exceeded
├── Check home quota → df -h /home/people/$USER
├── Move data to project dir → mv <files> /home/projects/<project>/
└── Clean cache → rm -rf ~/.cache/* (if safe)

Problem: Job won't start
├── Check queue status → qstat -Q
├── Check resource request → Are you requesting too much?
└── Check project account → Is -A <project_id> correct?

Problem: Job failed
├── Check error file → cat <job>.err
├── Check output file → cat <job>.out  
└── Check exit code → Was there OOM or timeout?
```

---

## 📚 Additional Resources

- **Computerome Wiki**: https://www.computerome.dk/wiki/user-guide
- **Support Email**: computerome@dtu.dk
- **PBS/Torque Documentation**: https://adaptivecomputing.com/cherry-services/torque-resource-manager/
