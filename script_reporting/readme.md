# Script reporting

Todd Knutson

## Introduction

The Slurm job scheduler is used by MSI systems to coordinate job submissions. These job scripts are simply shell scripts that include special header information, called slurm declaratives, that specify the resources needed for the job. In addition, we can include a couple `trap` (see `man bash` for details) commands that run certain commands if any errors occur during our script or when the script ends.

## Example shell/slurm script header

Note a few things:

- This job is requesting 1 core, 4 threads, 32 GB of memory, for 1 hour, on the `agsmall` partition (`agate` cluster).
- This job will not inherit any bash variables present when the job was submitted (i.e. `#SBATCH --export=NONE`) and will not load your .bashrc (unless you have specifically configured this action).
- This job will receive all email communication at the default address (submitter's email, e.g. $USER@umn.edu) and it's not required to specify it. Not specifying an email address makes the script more portable for others to copy/use.
- A `THREADS` variable is created that holds the number of threads the slurm job requested (i.e. `SLURM_CPUS_PER_TASK`), or if it's not currently running a slurm job, `THREADS` is set to `1`. This allows for easy and uniform access (whether running a job or not) to the number of threads available for the script.
- If an error occurs, a `trap` function will run, reporting the last bash command run and it's exit code.
- When the script ends (for any reason, error or success), a `trap` function will run, reporting the date/time, print the current env variables, and print slurm job details (if running a slurm job)
- **Best feature**: the `#SBATCH --error=%x.e%j` and `#SBATCH --output=%x.o%j` will automatically name the stderr and stdout files, based on the slurm script name. For example, if the slurm job file is named `my_analysis.slurm`, then the stderr file will be named `my_analysis.slurm.e12345678`, with the digits representing the SLURM_JOB_ID. A similar pattern occurs for the stdout file. This is ideal because it makes the log files obvious and associated with the script. `%x` is a special slurm value that equals the slurm script name and `%j` equals the job id number.

```
#!/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --time=4:00:00
#SBATCH --mem=32gb
#SBATCH --tmp=4gb
#SBATCH --error=%x.e%j
#SBATCH --output=%x.o%j
#SBATCH --export=NONE
#SBATCH --mail-type=ALL
#SBATCH --partition=agsmall

#######################################################################
# Script preliminaries
#######################################################################

# Exit script immediately upon error
set -o errexit -o errtrace -o pipefail -o functrace

function trap_my_error {
    >&2 echo "ERROR: \"${BASH_COMMAND}\" command failed with exit code $? [$(date)]"
}

function trap_my_exit {
    echo "[$(date)] Script exit."
    # Print env variables
    declare -p
    # Print slurm job details
    if [ -n "${SLURM_JOB_ID+x}" ]; then
        scontrol show job "${SLURM_JOB_ID}"
        sstat -j "${SLURM_JOB_ID}" --format=JobID,MaxRSS,MaxVMSize,NTasks,MaxDiskWrite,MaxDiskRead
    fi
}
# Execute these functions after any error (i.e. nonzero exit code) or
# when exiting the script (i.e. with zero or nonzero exit code).
trap trap_my_error ERR
trap trap_my_exit EXIT

# If not a slurm job, set THREADS to 1
THREADS=$([ -n "${SLURM_CPUS_PER_TASK+x}" ] && echo "${SLURM_CPUS_PER_TASK}" || echo 1)
export THREADS

echo "[$(date)] Script start."

#######################################################################
# Script
#######################################################################



# write your specific tasks here...

```

## Example R script header

Note a few things:

- A `THREADS` variable is created that holds the number of threads the slurm job requested (i.e. `SLURM_CPUS_PER_TASK`), or if it's not currently running a slurm job, `THREADS` is set to `1`. This allows for easy and uniform access (whether running a job or not) to the number of threads available for the script.
- If an error occurs,

```
#!/usr/bin/env Rscript
rm(list = ls(all.names = TRUE))
set.seed(20)

options(
    rlang_backtrace_on_error = "full",
    error = rlang::entrace,
    menu.graphics = FALSE,
    repos = c("CRAN" = "https://mirror.las.iastate.edu/CRAN"),
    mc.cores = as.integer(system("[ ! -z ${THREADS+x} ] && echo ${THREADS} || echo 1", intern = TRUE))
)

curr_threads <- getOption("mc.cores")
if (curr_threads < 1 | is.null(curr_threads)) {
    stop("Error: The bash THREADS variable is less than 1 or null.")
}
print(paste0("number of threads: ", curr_threads))




#######################################################################
# Load R packages
#######################################################################


library(tidyverse)
library(glue)
library(openxlsx)


#######################################################################
# Script parameters
#######################################################################


out_dir <- glue("/path/to/out")


if (!dir.exists(glue("{out_dir}"))) {
    dir.create(glue("{out_dir}"), recursive = TRUE)
}
setwd(glue("{out_dir}"))

```
