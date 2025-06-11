# How to create Python environments on MSI

Todd Knutson
2021-09-01

## Introduction

There are many ways to install and use python installations and packages on MSI systems. The strategy below is how I'm (currently) using python, but I'm open to learning alternative methods.

I like to use various different python/package versions or combinations for different projects or tasks. Often, a certain software tool depends on a specific python/package version that must be available in the environment. Thus, having the flexibility to install and activate precise environments when needed is important. This strategy will employ the standard python (ver 3+) package `venv` to create and manage different environments.

### Basic strategy:

- Use a base python installation (i.e. not from Anaconda) via `module load`
- Create a new python virtual environment for each project (or even subproject)
- Activate and deactivate specific virtual environments when necessary

## How to create a new `venv`

For each project or subproject, create a directory at the top level to store the python virtual environment.

```
MY_PROJ_DIR="$HOME/my_example_project"

mkdir -p $MY_PROJ_DIR
cd $MY_PROJ_DIR
```

Load whatever base python version needed for this venv:

```
module purge
module load /home/lmnp/knut0297/software/modulesfiles/python3/3.7.10
```

Create the virtual environment by providing a simple uniform name (e.g. `pyvenv`). The virtual environment will contain a symlink to the python executable.

```
python -m venv pyvenv
```

## How to use the python virtual environment?

When the python environment is needed:

1. Load the appropriate python version (making sure any shared libraries are added to the bash env `PATH`, `LD_LIBRARY_PATH`, or `PYTHONPATH` variables).
2. Source the `activate` script

```
module purge
module load /home/lmnp/knut0297/software/modulesfiles/python3/3.7.10

cd $MY_PROJ_DIR
source pyvenv/bin/activate
```

3. Run python scripts or interactive interpreter
4. Install packages via `pip` on the bash command line

```

pip install pandas==1.3.2
pip install session-info==1.0.0

# Check installations
python -c "import pandas; import session_info; session_info.show()"

```

5. Deactivate the python virtual environment when done. A `deactivate` bash function is created when a virtual environment is activated. This function can be executed from any path in your bash shell.

```
deactivate
```

## Notes

For general (non-project-based) usage, consider creating a virtual environment in your `$HOME` directory that has many useful packages installed.
