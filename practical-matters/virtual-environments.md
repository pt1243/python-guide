<h1 align='center'>Virtual Environments</h1>

<!-- <h6 align='center'>How to avoid creating problems for your future self</h6> TODO: better subtitle -->

This document covers how to use virtual environments to manage your Python installations across multiple projects and eliminate the possibility for dependency conflicts.

**Table of Contents:**
- [Introduction and motivation](#introduction-and-motivation)
  - [An overview of virtual environments](#an-overview-of-virtual-environments)
- [Creating and using virtual environments](#creating-and-using-virtual-environments)
  - [Using virtualenv](#using-virtualenv)
  - [Using the venv module](#using-the-venv-module)
  - [Using VS Code](#using-vs-code)
- [Technical overview](#technical-overview)
- [Summary](#summary)


## Introduction and motivation

The design of Python and its packaging systems means that installed packages (i.e., libraries) are kept in the file system alongside the Python interpreter executable; this means that only one version of a given package may be installed with a given Python installation. This causes issues when different projects sharing the installation have conflicting package requirements.

### An overview of virtual environments

The answer to this is the use of *virtual environments*. The [Python tutorial on virtual environments](https://docs.python.org/3/tutorial/venv.html) has a good overview of what a virtual environment is:

> A self-contained directory tree that contains a Python installation for a particular version of Python, plus a number of additional packages.

Essentially, a virtual environment allows you to create a separate, "clean" Python installation that contains its own, independent collection of packages for a specific Python version. They are useful for:

- Avoiding dependency conflicts between different projects. If project A requires `some-library` version 1 and is incompatible with version 2, but project B requires a minimum of version 2 of `some-library`, it is not possible to maintain a consistent environment state that can meet the requirements of both projects.
- Maintaining the correct versions of packages for different projects. It is possible that multiple projects can coexist and share a set of packages. However, when you want to later update the packages (perhaps for bugfixes, performance improvements, or other useful changes), it becomes almost impossible to know what projects will be impacted. At worst, some code may silently exhibit unintended behaviour and result in incorrect results that appear to be correct.
- Explicitly specificing project dependencies. When working from the system-wide Python installations, it is very easy to import some package and forget to include it in the dependencies. Your local copy of the project will work fine, but any other users will encounter issues even if they install the listed dependencies correctly.

It should be clear that independent virtual environments are a necessity as you begin to work with a range of complex and varied Python projects.

Of course, there is no need to go to the complete opposite extreme: projects that use only the standard library modules, simple scripts thrown together for a single use application, or other basic utilities with extremely simple dependency requirements don't necessitate creating a virtual environment; using the system environment is perfectly fine. A good rule of thumb is to *create an environment for any project that you expect to reuse and maintain, that you are collaborating on with others, that has complex dependencies, or is complex enough to require multiple files of Python code*.

I would recommend therefore keeping a small collection of packages installed in your system environment for when you need to run quick command line/IDLE experimentation, and only create virtual environments for more complex projects. I personally have `numpy`, `scipy`, and `pandas` if I want to check some small code snippet or verify some behaviour; `black`, `flake8`, and `pytest` for running these tools without installing them for small projects; and `notebook` and `ipython` for running Jupyter notebooks, or for using IPython magic functions. Note that for a specific project using Jupyter notebooks should have its own virtual environment - the system-wide version is just for opening files for an initial inspection.

## Creating and using virtual environments

A virtual environment exists in a directory, typically either `.venv` or `venv`, at the root of a project. They are excluded from version control systems such as Git, and are considered disposable - i.e., it should be simply to delete it and recreate it from scratch (such as through a `requirements.txt` file), and project code is not placed in the environment. They are also not copyable or movable once created, as some of the resulting configuration files include hardcoded paths to the interpreter in the virtual environment.

A further discussion of how virtual environments work is present below in the [technical overview](#technical-overview), but in short, the virtual environment maintains its own copy of a Python interpreter (the actual executable that runs Python) and related installed packages, independent from the system-wide installation packages and other virtual environments.

Once created, a virtual environment should be "activated", which prepends the virtual environment's interpreter and related scripts to the system `PATH`; this means that running commands such as `python` or `pip` will match against and run through the virtual environment's interpreter instead of the system-wide installations. The virtual environment includes several platform-specific activation scripts in the binary directory (`bin` on POSIX, and `Scripts` on Windows). When run, the script will:

1. Prepend the virtual environment's binary directory to the system `PATH`.
2. Add a prompt, typically `(.venv)` or similar, to the beginning of the terminal prompt as an indication that the virtual environment is activated.
3. Add a `deactivate` command which can be run to deactivate the environment and return to the default behaviour.

Activating an environment can be done with the following commands, depending on the platform. The virtual environment directory is assumed to be named `.venv`; if this is not the case, replace it with the actual name.

<table>
    <thead>
        <tr>
            <th>Platform</th>
            <th>Shell</th>
            <th>Activation command</th>
            <th>Deactivation command</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Windows</td>
            <td>cmd or PowerShell</td>
            <td><code>.venv\scripts\activate</code></td>
            <td rowspan=3><code>deactivate</code></td>
        </tr>
        <tr>
            <td rowspan=2>POSIX (macOS or Linux)</td>
            <td>bash or zsh</td>
            <td><code>. .venv/bin/activate</code>, or <code>source .venv/bin/activate</code></td>
        </tr>
        <tr>
            <td>Other shells</td>
            <td>See <a href="https://docs.python.org/3/library/venv.html#how-venvs-work">this table</a> in the Python docs</td>
        </tr>
    </tbody>
</table>

Do note that the activation script for cmd is `activate.bat`, and for PowerShell it is `activate.ps1`; the shells will match against files with these extensions. On the other hand, the script for bash and zsh is actually called `activate` with no extension.

Once the environment is activated, it will also take precedence in the Python launcher for Windows. This is indicated as a standalone `*` in the list of interpreters:

```
D:\project-folder>py -0p
 -V:3.12 *        C:\Program Files\Python312\python.exe
 -V:3.11          C:\Program Files\Python311\python.exe
 -V:3.10          C:\Program Files\Python310\python.exe
 -V:3.9           C:\Program Files\Python39\python.exe
 -V:3.8           C:\Program Files\Python38\python.exe

D:\project-folder>.venv\scripts\activate

(.venv) D:\project-folder>py -0p
  *               D:\project-folder\.venv\Scripts\python.exe
 -V:3.12          C:\Program Files\Python312\python.exe
 -V:3.11          C:\Program Files\Python311\python.exe
 -V:3.10          C:\Program Files\Python310\python.exe
 -V:3.9           C:\Program Files\Python39\python.exe
 -V:3.8           C:\Program Files\Python38\python.exe
```

It should be noted that activating a virtual environment is not required to use it; it is always possible to specify the paths to the virtual environment's executables directly. You should get into the habit of immediately activating the virtual environment for a project as soon as you open it to use it or work on it. Some editors such as VS Code may also automatically activate it for you when creating or reopening an integrated terminal window; however, such behaviour is not always reliable or consistent, so it is good practice to remember the entries in the table above.

Virtual environments can be created through two main tools: `virtualenv`, a third party library, and `venv`, a standard library module. `virtualenv` preceeds `venv` and offers more features, while `venv` provides only the core functionality (but does not require external dependencies). Some of the most important differences are, roughly in descending order of importance:

- `virtualenv` can automatically discover arbitrarily installed Python versions and create environments from them, while `venv` can only create an environment from the interpreter it is run from; i.e., running `py -3.10 -m venv` can only create a Python 3.10 environment.
- `venv` is relatively slow, while `virtualenv` can be extremely fast; see the [technical overview](#technical-overview) below.
- `virtualenv` will automatically add a `.gitignore` file to the environment directory; `venv` will eventually do so, but only for Python 3.13 onwards (see [python/cpython #83417](https://github.com/python/cpython/issues/83417)).
- `virtualenv` can be upgraded with `pip`, while `venv` is locked to the release date of the major Python version.
- `virtualenv` has a much richer API for interacting with it from within Python code, and is much more extendable.

For these reasons, I strongly suggest using `virtualenv`. The first two points are particularly important; this is how tools like `tox` can run tests against all the Python versions you have installed. This is why I recommend having a range of different Python versions installedThe best way to install it is to run `py -3.12 -m pip install virtualenv`, replacing 3.12 with the latest system-wide Python version you have.

### Using virtualenv

`virtualenv` is easy to use. The basic command syntax is:

<pre>virtualenv <i>destination</i> [-p <i>interpreter</i>] [<i>options</i>]</pre>

- <code><i>destination</i></code> is required, and is the folder in which the environment will be installed. As mentioned above, this is typically `.venv` or `.venv`.
- `-p` can be used to specify a specific <code><i>interpreter</i></code> to use as the base to create the environment from. This can either be the path to a Python interpreter (relative or absolute), or a specifier that identifies the interpreter's implementation, version, and architecture. Further discussion of this is detailed in the [technical overview](#technical-overview) below, but the simplest way is simply to specify something like `3.10`, replacing 3.10 with your desired version. If it is not present, `virtualenv` will use the version of Python it is installed into; this is why I suggest installing it into the latest system-wide version, so that a plain `virtualenv .venv` will use the latest version of Python.
- Various other <code><i>options</i></code> can also be passed, but it's unlikely that you will need them. See the [virtualenv CLI interface documentation](https://virtualenv.pypa.io/en/latest/cli_interface.html) for a full list of possible arguments.

This means that creating a Python 3.11 environment is as simple as `virtualenv .venv -p 3.11`, and creating a quick environment for testing is just `virtualenv .venv`. Combined with the significant speed increases over `venv`, it becomes extremely trivial to use virtual environments with `virtualenv`.

### Using the venv module

The use of `venv` is similar, but requires launching it as a module from a Python interpreter with the target version.

<pre>py -3.X -m venv <i>destination</i></pre>

As with `virtualenv`, <code><i>destination</i></code> is required and is typically `.venv` or `venv`. `3.X` should be replaced with the desired Python version. For this reason, it's useful to have multiple versions of Python 

Note that as with the rest of this guide, `python` should be substituted for the appropriate `py -3.X` on Windows to use the appropriate Python version. For this reason, it may be useful to multiple versions installed on your system.

The `-m` flag invokes the Python module `venv` (more explanation of this is available in the document about [command line options](/practical-matters/command-line-usage.md)).

### Using VS Code

Virtual environments can also be created through VS Code, although I don't recommend it as it is slower and more error prone than `virtualenv` or `venv`. Plus, learning how to create and activate virtual environments "manually" will help resolve probably 90+% of the issues in dealing with virtual environments.

Nevertheless, the process of creating a virtual environment in VS Code can be accomplished by running the command `Python: Create Environment... > Venv` and then selecting the appropriate system interpreter. If a `requirements.txt` file is present in the workspace, you also have the option to automatically install these dependencies.

Afterwards, VS Code should automatically select the environment interpreter, and activate the environment when you open a terminal. If it does not, you can always select the correct interpreter by manually locating the Python executable in `.venv\Scripts\python.exe`, or activate it manually.

If you encounter errors while trying to activate the environment related to PowerShell permission issues, see the [VS Code setting](/practical-matters/vs-code-settings) document for more information about the appropriate settings change to resolve this.

## Technical overview

Work in progress.

## Summary

- Virtual environments are a necessity for managing independent Python installations to avoid dependency conflicts.
- Virtual environments are created by running `py -3.X -m venv .venv` to create a Python `3.X` environment in the current directory.
- After creation, an environment must be activated to point the `py`, `python`, and `pip` commands to the correct executable. This is done by running `.venv\Scripts\activate`.
- Packages can now be installed in the new environment without affecting other environments or the system-wide installations through `pip install <package>`.
- Running the command `Python: Create Environment...` in VS Code will automatically create and activate an environment.
