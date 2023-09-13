<h1 align='center'>Virtual Environments</h1>

<!-- <h6 align='center'>How to avoid creating problems for your future self</h6> TODO: better subtitle -->

This document covers how to use virtual environments to manage your Python installations across multiple projects and eliminate the possibility for dependency conflicts.

**Table of Contents:**
- [Introduction and motivation](#introduction-and-motivation)
  - [The solution](#the-solution)
- [Creating and using virtual environments](#creating-and-using-virtual-environments)
  - [Using the venv module](#using-the-venv-module)
  - [Using VS Code](#using-vs-code)
- [Summary](#summary)


## Introduction and motivation

The design of Python and its packaging systems means that installed packages are kept in the file system alongside the Python interpreter executable; this means that only one version of a given package may be installed with a given Python installation. This causes issues when different projects sharing the installation have conflicting package requirements.

### The solution

The answer to this is the use of *virtual environments*. The [Python documentation](https://docs.python.org/3/tutorial/venv.html) has a good overview of what a virtual environment is:

> A self-contained directory tree that contains a Python installation for a particular version of Python, plus a number of additional packages.

Essentially, a virtual environment allows you to create a separate, "clean" Python installation that contains its own, independent collection of packages for a specific Python version. Using virtual environments will help resolve some of the following issues, which you may have encountered:

- Project A requires an earlier version of a library, but a new project that you have quickly thrown together, B, requires a new version due to incompatible library changes. Either you go through the process of redownloading the Python installer and creating an entire new Python installation for what was supposed to be a quick and easy new project, or you break project A's functionality by upgrading the library.
- You upgrade a library to a new version, but you have no idea which existing projects sharing that interpreter depend on functionality from the previous library version. Other than testing each one individually, you will most likely not discover any problems until some time down the road when you try and run one of them and it unexpectedly breaks.
- Without going through the process of checking each installed library (potentially hundreds) for updates, and validating new versions of each against every existing project, you are stuck with whatever versions you installed at the time. This means you may be missing out on bugfixes, performance improvements, or other useful changes.
- You want to clean up your installed packages and remove those which are no longer in use, but without reviewing every project's dependencies, it is impossible to know which are in use and which are safe to remove.
- You are sharing code with other users and they have different versions of the project dependencies installed. At best, this may outright throw an error and inform you of the problem; at worst, the code will run without errors, but may produce subtly different wrong results.

It should be clear that independent virtual environments are a necessity as you begin to work with more complex and varied Python projects.

Of course, there is no need to go to the complete opposite extreme: projects that use only the standard library modules, simple scripts thrown together for a single use application, or other basic utilities with extremely simple dependency requirements don't necessitate creating a virtual environment for; using the system environment is perfectly fine. A good rule of thumb is to *create an environment for any project that you expect to reuse and maintain, that you are collaborating on with others, that has complex dependencies, or is complex enough to require multiple files of Python code*.

I would recommend therefore keeping a small collection of packages installed in your system environment for when you need to run quick command line/IDLE experimentation, and only create virtual environments for more complex projects.

<details>

<summary>As an example, here are the packages that I keep installed in each of my system environments:</summary>

- `numpy` for efficient array operations and mathematical data processing.
- `scipy` for other general-purpose scientific computing.
- `pandas` for data storage and manipulation.
- `matplotlib` for graphing and visualising data.
- `flake8` for linting.
- `mypy` for type checking.
- `black` for code formatting.
- `pytest` to run tests.
- `attrs` for an alternative to the standard library `dataclass`.
- `notebook` for Jupyter Notebooks.
- `ipython` for the interactive Python kernel (what powers the "cells" in Jupyter notebooks).
- `wheel` for building packages.

While the dependencies for all of these do add up to a large collection of installed packages (over 100, most from `notebook`), the important thing is that there are few complex dependency relationships, and any small scripts I run using these libraries is not dependent on their version.

Do note that these packages are specific to my use case, and the Python code I write most often involves these libraries. For someone else who often writes quick scripts for HTTP requests, the `requests` library would be an obvious candidate for a system-wide package, while mathematically oriented libraries like NumPy and SciPy would probably not be justified.

</details>

## Creating and using virtual environments

### Using the venv module

The creation of virtual environments is handled by the `venv` standard library module. This can be done as follows:

```
python -m venv <environment name>
```
<!-- TODO: check for consistent command line syntax (eg. angle brackets, square brackets, caps, etc) -->

Note that as with the rest of this guide, `python` should be substituted for the appropriate `py -3.X` on Windows to use the appropriate Python version. This also allows the creation of virtual environments for any system environments you have installed; for this reason, it may be useful to multiple versions installed on your system. I personally have Python 3.7 to 3.11 installed, as they are the currently supported Python versions.

The `-m` flag invokes the Python module `venv` (more explanation of this is available in the document about [command line options](/practical-matters/command-line-usage-and-pip.md)). This creates a new environment within the `environment name` folder in the current directory. While this can be any name you want, by convention this is usually called `.venv` to indicate to others that it is a virtual environment folder and should not be manually modified. This also has the advantage that most `.gitignore` templates should ignore this, and the leading period treats the folder as "hidden" in certain circumstances (e.g., when running certain commands on Linux).

Therefore, for practical use, assume we have some Python project located in the folder `some-python-project\`. We would create a Python 3.11 virtual environment with the following command:

```
D:\some-python-project> py -3.11 -m venv .venv
```

This creates the folder `some-python-project\.venv\` and installs a new copy of the Python 3.11 interpreter there.

The next step after creating an environment is to *activate* it. Activating an environment sets the appropriate environment variables to ensure that subsequent Python commands point to the newly created installation. This is done by running the following script:

```
D:\some-python-project> .venv\Scripts\activate 

(.venv) D:\some-python-project> 
```

Note how after activating the environment, the terminal prompt is preceded by `(.venv)`. This indicates that the virtual environment present in `.venv` has been successfully activated. Running `py -0p` shows that the Python launcher is now aware of the environment and has selected it as the default:

```
(.venv) D:\some-python-project> py -0p
  *               D:\some-python-project\.venv\Scripts\python.exe
 -V:3.11          C:\Program Files\Python311\python.exe
 -V:3.10          C:\Program Files\Python310\python.exe
 -V:3.9           C:\Program Files\Python39\python.exe
 -V:3.8           C:\Program Files\Python38\python.exe
 -V:3.7           C:\Program Files\Python37\python.exe
```

The activation script also sets the appropriate environment variables that running `python` or `pip` commands will point to the correct executables in the environment. Running `pip list` to list the installed packages now correctly shows that only the basic `pip` and `setuptools` packages are installed to bootstrap the environment:

```
(.venv) D:\some-python-project> pip list
Package    Version
---------- -------
pip        23.1.2
setuptools 65.5.0
```

We can now install packages easily using either `py -m pip` or `pip`:

```
(.venv) D:\some-python-project> pip install numpy
Collecting numpy
  Using cached numpy-1.24.3-cp310-cp310-win_amd64.whl (14.8 MB)
Installing collected packages: numpy
Successfully installed numpy-1.24.3

(.venv) D:\some-python-project> pip list
Package    Version
---------- -------
numpy      1.24.3
pip        23.1.2
setuptools 65.5.0
```

Virtual environments can be deactivated by running the command `deactivate`, which returns the default Python versions to the system-wide ones. Note that closing a terminal will also require the environment to be reactivated; it is therefore to get into the habit of activating the environment every time you interact with the project.

### Using VS Code

The process of creating a virtual environment in VS Code can be accomplished by running the command `Python: Create Environment... > Venv` and then selecting the appropriate system interpreter. If a `requirements.txt` file is present in the workspace, you also have the option to automatically install these dependencies.

Afterwards, VS Code should automatically select the environment interpreter, and activate the environment when you open a terminal. If it does not, you can always select the correct interpreter by manually locating the Python executable in `.venv\Scripts\python.exe`, and running `.venv\Scripts\activate.ps1` in the terminal (as the default terminal is PowerShell).

If you encounter errors while trying to activate the environment related to PowerShell permission issues, see the [VS Code setting](/practical-matters/vs-code-settings) document for more information about the appropriate settings to resolve this.

## Summary

- Virtual environments are a necessity for managing independent Python installations to avoid dependency conflicts.
- Virtual environments are created by running `py -3.X -m venv .venv` to create a Python `3.X` environment in the current directory.
- After creation, an environment must be activated to point the `py`, `python`, and `pip` commands to the correct executable. This is done by running `.venv\Scripts\activate`.
- Packages can now be installed in the new environment without affecting other environments or the system-wide installations through `pip install <package>`.
- Running the command `Python: Create Environment...` in VS Code will automatically create and activate an environment.
