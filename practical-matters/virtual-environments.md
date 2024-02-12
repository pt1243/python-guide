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
  - [Adjusting the module search path](#adjusting-the-module-search-path)
  - [How `virtualenv` works](#how-virtualenv-works)
- [Summary](#summary)
- [Further reading](#further-reading)


## Introduction and motivation

The design of Python and its packaging systems means that installed site packages are kept in the file system alongside the Python interpreter; this means that only one version of a given package may be installed with a given Python installation. This causes issues when different projects sharing the installation have conflicting package requirements.

### An overview of virtual environments

The answer to this is the use of *virtual environments*. The [Python tutorial on virtual environments](https://docs.python.org/3/tutorial/venv.html) has a good overview of what a virtual environment is:

> A self-contained directory tree that contains a Python installation for a particular version of Python, plus a number of additional packages.

Before proceeding, it may be helpful to define and clarify some terms:

- The Python *interpreter* or *executable* is the `python.exe` program that executes Python code.
- The *standard library* consists of the modules included in Python, such as `os` and `math`, while *site packages* are user-installed packages such as `numpy`.
- A Python *environment* refers to the interpreter and its related packages that are used for a specific piece of code or project.
- The *base* or *system* Python is the global, system-wide installation of Python. There may be multiple system Python installations of different versions.
- A *virtual environment* refers abstractly to a separate environment with its own packages, or concretely, to the folder on disk (such as `.venv`) where it is located; this will be explained further on.
- The *major* and *minor* version numbers are the first two numbers in a Python version; for example, the major and minor versions for Python 3.12.1 are 3 and 12 respectively. These are commonly referred to in the Python documentation (and this guide) as an italicized *`X`* and *`Y`*, indicating that they should be replaced with the actual values. For instance, a directory called <code>python<i>X</i>.<i>Y</i></code> would be `python3.12` for 3.12, `python3.11` for 3.11, and so on.

Essentially, a virtual environment allows you to create a separate, "clean" Python installation that contains its own, independent collection of site packages for a specific Python version. They are useful for:

- Avoiding dependency conflicts between different projects. If project A requires `some-library` version 1 and is incompatible with version 2, but project B requires a minimum of version 2 of `some-library`, it is not possible to maintain a consistent environment state that can meet the requirements of both projects.
- Maintaining the correct versions of packages for different projects. It is possible that multiple projects can coexist and share a set of packages. However, when you want to later update the packages (perhaps for bugfixes, performance improvements, or other useful changes), it becomes almost impossible to know what projects will be impacted. At worst, some code may silently exhibit unintended behaviour and result in incorrect results that appear to be correct.
- Explicitly specifying project dependencies. When working from the system-wide Python installations, it is very easy to import some package and forget to include it in the dependencies. Your local copy of the project will work fine, but any other users will encounter issues even if they install the listed dependencies correctly.
- Not interfering with packages for the OS Python installation. This is not an issue for Windows, but parts of some Linux operating systems are written in Python and require specific Python and library versions to work correctly.

It should be clear that independent virtual environments are a necessity as you begin to work with a range of complex and varied Python projects.

Of course, there is no need to go to the complete opposite extreme: projects that use only the standard library modules, simple scripts thrown together for a single use application, or other basic utilities with extremely simple dependency requirements don't necessitate creating a virtual environment; using the system environment is perfectly fine. A good rule of thumb is to *create a virtual environment for any project that you expect to reuse and maintain, that you are collaborating on with others, that has complex dependencies, or that is complex enough to require multiple files of Python code*.

I would recommend therefore keeping a small collection of packages installed in your system environment for when you need to run quick command line/IDLE experimentation, and only create virtual environments for more complex projects. I personally have `numpy`, `scipy`, and `pandas` if I want to check some small code snippet or verify some behaviour; `black`, `flake8`, and `pytest` for running these tools without installing them for small projects; and `notebook` and `ipython` for running Jupyter notebooks, or for using IPython magic functions. Note that for a specific project using Jupyter notebooks should have its own virtual environment - the system-wide version is just for opening files for an initial inspection.

## Creating and using virtual environments

A virtual environment exists in a directory, typically either `.venv` or `venv`, at the root of a project. They are excluded from version control systems such as Git, and are considered disposable - i.e., it should be simple to delete it and recreate it from scratch (such as through a `requirements.txt` file), and project code is not placed in the environment. They are also not copyable or movable once created, as some of the resulting configuration files include hardcoded paths to the interpreter in the virtual environment.

A further discussion of how virtual environments work is present below in the [technical overview](#technical-overview), but in short, the virtual environment maintains its own copy of a Python interpreter (the actual executable that runs Python) and related installed packages, independent from the system-wide installed packages and other virtual environments. The standard library modules are shared with the base (system-wide) Python installation.

Once created, a virtual environment should be "activated", which prepends the virtual environment's interpreter and related scripts to the system `PATH`; this means that running commands such as `python` or `pip` will match against and run through the virtual environment's interpreter instead of the system-wide installations. The virtual environment includes several platform-specific activation scripts in the binary directory (`bin` on POSIX, and `Scripts` on Windows). When run, the script will:

1. Prepend the virtual environment's binary directory to the system `PATH`.
2. Set the `VIRTUAL_ENV` environment variable to the virtual environment's directory.
3. Add a prompt, typically `(.venv)` or similar, to the beginning of the terminal prompt as an indication that the virtual environment is activated.
4. Add a `deactivate` command which can be run undo steps 1-3 and return to the default behaviour.

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

You can then run <code>pip install <i>package</i></code> to install packages into the environment.

It should be noted that activating a virtual environment is not required to use it; it is always possible to specify the paths to the virtual environment's executables directly. You should get into the habit of immediately activating the virtual environment for a project as soon as you open it to use it or work on it. Some editors such as VS Code may also automatically activate it for you when creating or reopening an integrated terminal window; however, such behaviour is not always reliable or consistent, so it is good practice to remember the entries in the table above.

Virtual environments can be created through two main tools: `virtualenv`, a third party library, and `venv`, a standard library module. `virtualenv` precedes `venv` and offers more features, while `venv` provides only the core functionality (but does not require external dependencies). Some of the most important differences are, roughly in descending order of importance:

- `virtualenv` can automatically discover arbitrarily installed Python versions and create environments from them, while `venv` can only create an environment from the interpreter it is run from; i.e., running `py -3.10 -m venv` can only create a Python 3.10 environment.
- `venv` is relatively slow, while `virtualenv` can be extremely fast; see the [technical overview](#how-virtualenv-works) below.
- `virtualenv` will automatically add a `.gitignore` file to the environment directory; `venv` will eventually do so, but only for Python 3.13 onwards (see [python/cpython #83417](https://github.com/python/cpython/issues/83417)).
- `virtualenv` can be upgraded with `pip`, while `venv` is locked to the release date of the minor Python version.
- `virtualenv` has a much richer API for interacting with it from within Python code, and is much more extendable.

For these reasons, I strongly suggest using `virtualenv`. The first two points are particularly important; this is how tools like `tox` can discover, create environments for, and run tests against all the Python versions you have installed. This is why I recommend having a range of different Python versions installed. The best way to install `virtualenv` is to run `py -3.12 -m pip install virtualenv`, replacing 3.12 with the latest system-wide Python version you have.

### Using virtualenv

`virtualenv` is easy to use. The basic command syntax is:

<pre>virtualenv <i>destination</i> [-p <i>interpreter</i>] [<i>options</i>]</pre>

- <code><i>destination</i></code> is required and is the folder in which the environment will be installed. As mentioned above, this is typically `.venv` or `.venv`.
- `-p` can be used to specify a specific <code><i>interpreter</i></code> to use as the base to create the environment from. This can either be the path to a Python interpreter (relative or absolute), or a specifier that identifies the interpreter's implementation, version, and architecture. Further discussion of this is detailed in the [technical overview](#how-virtualenv-works) below, but the simplest way is simply to specify <code><i>X</i>.<i>Y</i></code>, replacing the major and minor numbers with the desired version. If `-p` is not passed, `virtualenv` will use the version of Python it is installed into; this is why I suggest installing it into the latest system-wide version, so that a plain `virtualenv .venv` will use the latest version of Python.
- Various other <code><i>options</i></code> can also be passed, but it's unlikely that you will need them. See the [virtualenv CLI documentation](https://virtualenv.pypa.io/en/latest/cli_interface.html) for a full list of possible arguments.

This means that creating a Python 3.11 environment is as simple as `virtualenv .venv -p 3.11`, and creating a quick environment for testing is just `virtualenv .venv`. Combined with the significant speed increases over `venv`, it becomes extremely trivial to use virtual environments with `virtualenv`.

### Using the venv module

The use of `venv` is similar but requires launching it as a module from a Python interpreter with the target version.

<pre>py -3.<i>Y</i> -m venv <i>destination</i></pre>

As with `virtualenv`, <code><i>destination</i></code> is required and is typically `.venv` or `venv`. *`Y`* should be replaced with the desired minor Python version. For this reason, it's useful to have multiple versions of Python installed on your system.

The `-m` flag invokes the Python module `venv` (more explanation of this is available in the document about [command line options](/practical-matters/command-line-usage.md)).

### Using VS Code

Virtual environments can also be created through VS Code, although I don't recommend it as it is slower and more error prone than `virtualenv` or `venv` (both in terms of environment creation, but also in the sense that typing out the commands is probably faster than using VS Code's interface). Plus, learning how to create and activate virtual environments "manually" will help resolve nearly all of the issues that arise when dealing with virtual environments.

Nevertheless, the process of creating a virtual environment in VS Code can be accomplished by running the command `Python: Create Environment... > Venv` and then selecting the appropriate system interpreter. If a `requirements.txt` file is present in the workspace, you also have the option to automatically install these dependencies.

Afterwards, VS Code should automatically select the environment interpreter, and activate the environment when you open a terminal. If it does not, you can always select the correct interpreter by manually locating the Python executable in `.venv\Scripts\python.exe`, and you can activate the environment manually.

If you encounter errors while trying to activate the environment related to PowerShell permission issues, see the [VS Code setting](/practical-matters/vs-code-settings) document for more information about the appropriate settings changes to resolve this.

## Technical overview

This section intends to provide a general overview of the mechanisms behind the functionality of virtual environments. This is more information than necessary for day to day use of virtual environments, but still (in my opinion) interesting to know about. It also provides a somewhat gentler introduction to the Python import machinery, without having to go into full depth about finders, loaders, module specs, and so on. Still, feel free to skip to the [summary](#summary) below.

This section is broken down into two parts. The first is focused on how a virtual environment impacts Python's module search path so that the virtual environment's packages are used instead of the system installations packages. The second discusses the process of creating such an environment, specifically through `virtualenv`. These sections will assume that the root folder directory is `D:\project-folder`, the virtual environment is installed in `.venv`, and the base Python installation is 3.12, installed at `C:\Program Files\Python312`.

### Adjusting the module search path

Python's import protocol is not overly complex, but there are a lot of interacting parts that make it difficult to fully grasp. For now, it is sufficient to say that when you try to import a module, Python will use various *finders* listed in `sys.meta_path` to try and locate it, and one of these finders searches an *import path*, `sys.path`, to try and locate the module in the various directories in this path. The virtual environment's functionality is achieved by changing some of the entries on this path to point toward the site packages held by the virtual environment instead of the base Python installation.

The next part will get complex, even though it simplifies many parts of the process. Just the determination of the executable location and setting the prefixes, before we even reach the `site` processing steps, takes about 1000 lines of C and 1400 lines of Python code ([getpath.c](https://github.com/python/cpython/blob/main/Modules/getpath.c), and [getpath.py](https://github.com/python/cpython/blob/main/Modules/getpath.py) and [the sysconfig module](https://github.com/python/cpython/blob/main/Lib/sysconfig/__init__.py) respectively); consider yourself warned. Even so, this is a simplification that tries to synthesize the information from multiple places in the Python documentation, not  all of which are fully consistent.

We will start by seeing what happens without a virtual environment to explain the basics of the process, and then investigate the differences when a virtual environment is in use.

The first step Python performs when it starts is to determine the location of the actual executable that it is running from. On Windows, this is relatively trivial, but on Unix it involves things such as resolving symlinks and several other checks to try and obtain the executable location. This is made available at `sys.executable`, and the directory containing the executable is called *`home`*.

The next step is for Python to determine where the standard library modules are located. The directory containing platform-independent Python modules (i.e. those written in pure Python code) is called the *`prefix`*, and similarly, the directory containing extension modules (those with components written in C or C++) is called the *`exec_prefix`*. If the `PYTHONHOME` environment variable is set, this is the preferred source for setting *`prefix`* and *`exec_prefix`*. If `PYTHONHOME` contains a single directory, this is used for both *`prefix`* and *`exec_prefix`*. Different directories can be specifed as <code><i>prefix</i>:<i>exec_prefix</i></code>, replacing the colon with a semicolon on Windows.

If `PYTHONHOME` is not set (which is usually the case), the directories are instead found by starting from the Python executable and looking for certain "landmark" files and directories in order to confirm that *`home`* is indeed the correct location of the Python installation. The first is the ZIP archive <code>python<i>XY</i>.zip</code>; on Windows, this is searched for in *`home`*, and on Unix it is searched for in `lib`. If the archive doesn't exist, the next step is to look for <code><i>home</i>\Lib\os.py</code>, and on Unix, in <code>lib/python<i>X</i>.<i>Y</i>/os.py</code>. This is usually where the landmark search succeeds. On Windows, *`prefix`* and *`exec_prefix`* are the same, but on other platforms, <code>lib/python<i>X</i>.<i>Y</i>/lib-dynload</code> is searched for and used as a landmark for *`exec_prefix`*.

Once found, the directories are available at `sys.prefix` and `sys.exec_prefix`. Now, the actual construction of `sys.path` can begin. Several entries are added, in the following order:

1. The first entry is the directory containing the input script if there is one (which is not necessarily the current directory). Otherwise, such as when running the interactive interpreter or using the `-c` or `-m`, this is the current directory.
2. The second entry is the ZIP archive mentioned above. This is notable in that it is the only entry on `sys.path` that is added regardless of if it exists or not.
3. Next, if the `PYTHONPATH` environment variable is set, it is split on the platform-specific delimiter (semicolon on Windows, otherwise a colon), and each entry is appended to `sys.path`.
4. Next, on Windows only, any additional "application paths" found in the registry that are subkeys of <code>\SOFTWARE\Python\PythonCore\\<i>X</i>.<i>Y</i>\PythonPath</code> under both the `HKEY_CURRENT_USER` and `HKEY_LOCAL_MACHINE` hives are processed. (The current Python installers only use HKLM, so HKCU is generally empty and not used). Subkeys that have semicolon-delimited path strings as their default values will have each path appended to `sys.path`. The subkey normally looks something like `C:\Program Files\Python312\Lib\;C:\Program Files\Python312\DLLs\`.
5. If the landmark search succeeded at verifying *`home`*, the relevant standard library directories relative to that folder are added, constructed from the *`prefix`* and *`exec_prefix`*, and depend on the platform, but on Windows will be something. These are platform dependent (but will usually be something like `C:\Program Files\Python312\Lib` and `C:\Program Files\Python312\DLLs` on Windows). On Windows, the executable directory is also added.

Now that the standard library locations have been added to `sys.path`, Python proceeds to add the `site-packages` directories, containing the site packages. This is done through the `site` module, which is automatically imported and has its `main()` method called during interpreter startup unless the `-S` flag is passed when launching Python.

`site` will first construct up to four directories from combinations of two head parts and two tail parts. The head parts are the *`prefix`* and *`exec_prefix`*, and the tail parts are the empty string and either `Lib\site-packages` (Windows) or <code>lib/python<i>X</i>.<i>Y</i>/site-package</code> (Unix). For each combination, if the constructed site packages directory exists, it is appended to `sys.path` and also inspected for *path configuration files*.

A path configuration file has a filename ending in `.pth`. For each line in such a file:

- If the line is only whitespace or begins with a `#``, it is skipped.
- If the line starts with `import` followed by a space or tab character, that line is executed.
- Otherwise, the contents of that line are appended to `sys.path`. Non-existent items are never added to `sys.path`. No check is made that the item refers to a directory and not a file. No item is added to `sys.path` more than once.

Next, an attempt is to made to import a module called `sitecustomize`, which can perform arbitrary site-specific customizations. If this fails with an `ImportError` or a subclass thereof, and the exception's `name` attribute is `sitecustomize`, it is silently ignored. Then, an attempt is made to import a module called `usercustomize` if `site.ENABLE_USER_SITE` is `True` (which is usually the case, unless Python is started with `-s` or it is disabled through the environment variable `PYTHONNOUSERSITE`, in which case it is `False`); similarly, if this raises an `ImportError` or a subclass thereof with the `name` attribute equal to `usercustomize`, it is silently ignored.

`site` also handles adding the *user site packages* directory, which is specific to the Python version and user. To understand why this is, we first need to explain the concept of *installation schemes*. Installation schemes are dependent on the platform and the options you select when installing Python, and are used to determine where certain files should go. Python currently supports nine schemes (although only six are meaningfully distinct). Each scheme defines the names and locations of eight distinct file path identifiers for different purposes, although there is usually some overlap in the actual paths between them. The full list of schemes and their path definitions is available at the [sysconfig documentation](https://docs.python.org/3/library/sysconfig.html#installation-paths). You can run `python -m sysconfig` to see the current list of definitions on your system.

The [installation instructions](./installing-and-managing-python.md) to install Python system-wide on Windows results in the [`nt` scheme](https://docs.python.org/3/library/sysconfig.html#nt), which is relative to *`prefix`* (which is `C:\Program Files\Python312` or similar). However, package managers such as `pip` don't usually have permission to write to `C:\Program Files`, and so must fall back to installing packages into the [`nt_user` scheme](https://docs.python.org/3/library/sysconfig.html#nt-user), which is relative to a directory called *`userbase`*. This is accessible at `site.USER_BASE`, and the subdirectory for site packages specifically is `site.USER_SITE`. The latter is typically:

- <code>~/.local/lib/python<i>X</i>.<i>Y</i>/site-packages</code> for Unix and non-framework macOS builds
- <code>~/Library/Python/<i>X</i>.<i>Y</i>/lib/python/site-packages</code> for macOS framework builds
- <code><i>%APPDATA%</i>\Python\Python<i>XY</i>\site-packages</code> for Windows

If `site.ENABLE_USER_SITE` is `True`, the user site packages directory is added to `sys.path` and processed for path configuration files. With this, the `sys.path` initialization is complete. Before moving on to the differences when a virtual environment is present, let's review what a typical `sys.path` would look like without a virtual environment. The comments indicate the origin of each line.

```python
>>> import sys
>>> from pprint import pprint
>>> pprint(sys.path)
['',                                                                        # current directory (step 1)
 'C:\\Program Files\\Python312\\python312.zip',                             # ZIP archive (step 2)
 'C:\\Program Files\\Python312\\DLLs',                                      # registry key (step 4) & standard library directory (step 5)
 'C:\\Program Files\\Python312\\Lib',                                       # registry key (step 4) & standard library directory (step 5)
 'C:\\Program Files\\Python312',                                            # executable directory (step 5)
 'C:\\Users\\Jeremy\\AppData\\Roaming\\Python\\Python312\\site-packages',   # user site packages
 'C:\\Program Files\\Python312\\Lib\\site-packages']                        # global site packages
```

Note that the `DLLs` and `Lib` folders are found in both step 4 (the registry) and step 5 (the standard library directories constructed from the *`prefix`* and *`exec_prefix`*), but `site`'s processing includes a step that removes duplicate entries. However, even when `-S` is used, the directories are deduplicated; it is unclear where this step occurs.

Now, with that complete, we can look at how the process differs when a virtual environment is in use. Compared to the process above, it is relatively simple.

First, if a file named `pyvenv.cfg` is found either adjacent to the Python executable or one directory above it, the file is scanned for lines in the form <code><i>key</i> = <i>value</i></code>. If one of the keys is `home`, and the value is an absolute path, and `PYTHONHOME` is not set, the prefix finding process continues using that value, which is the base Python installation. Afterwards, `sys.prefix` and `sys.exec_prefix` are set to the location of the virtual environment (i.e. `D:\project-folder\.venv`), while `sys.base_prefix` and `sys.base_exec_prefix` are set to the values found when searching the base installation. (When not running in a virtual environment, `sys.base_prefix` is equal to `sys.prefix`, and `sys.base_exec_prefix` is equal to `sys.exec_prefix`. It is sufficient to determine if a script is being run from a virtual environment by checking if `sys.prefix != sys.base_prefix`.)

When `site` searches for site packages, if `pyvenv.cfg` contains the key `include-system-site-packages` with a value set to `true`, the global site packages will be added to `sys.path`; for any other value, only the virtual environment's site packages are added.

A typical `pyvenv.cfg` file might look like the following:

```
home = C:\Program Files\Python312
implementation = CPython
version_info = 3.12.1.final.0
virtualenv = 20.24.7
include-system-site-packages = false
base-prefix = C:\Program Files\Python312
base-exec-prefix = C:\Program Files\Python312
base-executable = C:\Program Files\Python312\python.exe
```

This results in the following `sys.path`:

```python
>>> import sys
>>> from pprint import pprint
>>> pprint(sys.path)
['',                                                # current directory (step 1)
 'C:\\Program Files\\Python312\\python312.zip',     # ZIP archive (step 2)
 'C:\\Program Files\\Python312\\DLLs',              # registry key (step 4) & base Python standard library directory (step 5)
 'C:\\Program Files\\Python312\\Lib',               # registry key (step 4) & base Python standard library directory (step 5)
 'C:\\Program Files\\Python312',                    # base python executable directory (step 5)
 'D:\\project-folder\\.venv',                       # virtual environment executable directory
 'D:\\project-folder\\.venv\\Lib\\site-packages']   # virtual environment site packages
```

That's pretty much all there is to it. There is one more caveat though: if a file with a `._pth` (not `.pth`) extension with the same name as the shared library or executable (i.e. `python._pth` or `python312._pth`) is found, this completely overrides the `sys.path` initialization. All environment variables are ignored, registry entries are not added, and `site` is not imported unless one line specifies `import site`. Blank lines and lines starting with a `#` are ignored. All other lines must be either an absolute or relative path. Imports other to `site` are not permitted, and no other code can be specified. This is primarily used only in applications that bundle Python and need to completely override the normal process.

### How `virtualenv` works

**Work in progress.**

Now that we understand how the `pyvenv.cfg` file allows us to create virtual environments, we can look at how `virtualenv` works to set one up. This occurs in two phases: phase 1 discovers a Python interpreter to create a virtual environment from, while phase 2 creates the virtual environment in the specified location.

In phase 1, `virtualenv` will try to find a Python interpreter that matches a given *specifier*. The specifier is the *`interpreter`* argument passed to the `-p` option, and it allows you to specify what type of virtual environment you would like to create. `virtualenv` will always know about one such specifier: the Python interpreter it is currently running from. If `-p` is not passed, this is what it will use when creating the environment.

Otherwise, `virtualenv` will try to find a Python interpreter to satisfy the specifier. The specifier can be a path to a Python interpreter (either absolute or relative), but most of the time, we don't want to bother typing out the full path to the executable each time. Instead, we can also specify the Python implementation, version, and architecture in that order.

- The Python implementation is all alphabetic characters, and may be something like `cpython` or `pypy`. This is optional, and if not provided, defaults to `python`, meaning any implementation.
- The version is a dot separated version number. You may specify as little as the major version (`3`), but this will usually include the minor version as well, such as `3.9` or `3.12`. The micro version can also be included, such as in `3.8.4`, but as there are normally not any significant changes between micro versions, this is usually overly restrictive.
- The architecture is either `-32` or `-64` (bit), and is optional; if missing, it defaults to `any`. Again, this is usually overly restrictive, as you're unlikely to run into a 32 bit version of Python nowadays.

So, for example:

- `python3.8.1` would match any implementation, with a version of exactly `3.8.1`, and any architecture.
- `3` would match any implementation with a major version of `3`, and any architecture.
- `cpython3` would match a `CPython` implementation with a major version of `3`, and any architecture.
- `pypy2` would match a `PyPy` implementation with a major version of `2`, and any architecture.

This is normally of the form <code>3.<i>Y</i></code>.

Given the specifier, `virtualenv` will apply one of two strategies depending on the platform. On Windows, it will look into the registry to see if any registered versions match the specification, as specified in [PEP 514](https://peps.python.org/pep-0514/). On other platforms, it will iterate through the folders on the system `PATH` and try to find an interpreter with a name mamtching the specification.

Given the base interpreter to create an environment, phase 2 proceeds with actually creating the environment. This can be broken down further into three sub-steps, which are accomplished using *creators*, *seeders*, and *activators*.

Creators are responsible for either creating the actual directories and files for the virtual environments. There are two creation methods available; the first is the `venv` method. As you might imagine, this delegates creation to the `venv` module. However, this has the downside that `virtualenv` must create a new process to invoke the module, which is especially slow on Windows. The alternative is the `builtin` method - this means that `virtualenv` knows exactly what files to create and can do so itself. The `builtin` creator is an alias to a creator specific to the platform and version, and will be something like `cpython3-win`. If the `builtin` creator is available, `virtualenv` will prefer to use that for performance reasons.

The seeder is responsible for installing 

## Summary

- Virtual environments are a necessity for managing independent Python installations to avoid dependency conflicts.
- Virtual environments are created by running <code>virtualenv .venv -p 3.<i>Y</i></code> to create a Python <code>3.<i>Y</i></code> environment in the current directory.
- After creation, an environment must be activated to point the `py`, `python`, and `pip` commands to the correct executable. This is done by running `.venv\Scripts\activate`.
- Packages can now be installed in the new environment without affecting other environments or the system-wide installations through <code>pip install <i>package</i></code>.
- Environments can be deactivated by running `deactivate`.

## Further reading

Work in progress.
