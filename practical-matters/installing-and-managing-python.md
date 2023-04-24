<h1 align='center'>Installing and Managing Python</h1>

<h6 align='center'>How to avoid creating problems for your future self</h6>

This document covers the process of *correctly* installing Python, and how best to manage multiple Python versions. This may seem overly simple for an "intermediate" Python guide, but an incorrect Python installation is bound to cause headaches in the future.

**Table of Contents:**
- [Download and installation](#download-and-installation)
  - [Reinstalling a preexisting version](#reinstalling-a-preexisting-version)
  - [Installing a new version](#installing-a-new-version)
- [Managing multiple versions](#managing-multiple-versions)
- [Summary](#summary)



## Download and installation

### Reinstalling a preexisting version

If you already have Python installed, it's probably best that you reinstall it following these instructions. All that you will need to do is export your current list of installed packages (~30 seconds), download and reinstall Python (~2 minutes), point VS Code (or any other IDE's you use) to the new Python locations (~30 seconds), and finally reinstall your packages (~30 seconds plus however long they take to download). All in all, a few minutes of work now can easily save you many times that debugging things in the future.

Let's assume that the version you want to reinstall is `3.X` (so if you were reinstalling Python 3.11, use `3.11`).

1. Open a terminal (cmd or PowerShell) and run `py -0p`. This lists all the versions you have installed, and their paths. Make a note of the directory that the Python executable is installed into (`<install path>`). 
   - If your version isn't listed here, that means you screwed up when you initially installed it, and Python has no idea that it even exists. In that case, just manually delete any files related to that version that you can find (you may find them in `C:\Users\%USERNAME%\AppData\Local\Programs\Python\Python`).
2. Export your list of currently installed packages by running `py -3.X -m pip list > requirements3X.txt`, which saves them to the file `requirements3X.txt` in your current directory.
3. In the Windows Settings under `Apps > Installed Apps`, find the appropriate Python version and uninstall it. This should open the Python installer window, where you can then select `Uninstall`.
   - If this doesn't work for any reason, you can also try manually deleting everything. Also ensure that you remove the path to this installation from your system `PATH`. Search for `Edit the system environment variables`, or run `SystemPropertiesAdvanced.exe` from the Windows Run command dialog box or through a terminal. Click `Environment variables...`. Double-click on the `Path` variables for both your user account *and* the system. Look for `<install path>\` and `<install path>\Scripts\` and remove them. Make sure you don't remove entries for *other* Python versions, because then you won't be able to repeat these steps for them afterwards.
4. Proceed with the steps below in [installing a new version](#installing-a-new-version).
5. After completing all these steps, navigate to the directory with your `requirements3X.txt` file and run `py -3.X -m pip install -r requirements3X.txt`.
6. Open VS Code (or any other IDE's) and point them towards your new Python installation (in VS Code, `> Python: Select Interpreter`, then either choose it from the list or `Enter interpreter path... > Find...` and locate the Python executable at `C:\Program Files\Python3X\python.exe`).

That's it! Repeat this process for each version of Python you have installed. Of course, if you only want to uninstall old versions you can stop after step 3.

### Installing a new version

Installing a new Python is fairly simple and straightforward, and very similar to what you might have already done in the past (just with a few key changes).

Let's assume that the version you want to install is `3.X` (so if you were installing Python 3.11, use `3.11`).

1. Find the download for the `Windows installer (64-bit)` for the latest micro version (i.e., the largest `Y` for `3.X.Y` of the available versions). For your convenvience, here are the latest (as of April 2023) download links for all currently supported Python versions:
   - [Python 3.11.3](https://www.python.org/ftp/python/3.11.3/python-3.11.3-amd64.exe)
   - [Python 3.10.11](https://www.python.org/ftp/python/3.10.11/python-3.10.11-amd64.exe)
   - [Python 3.9.13](https://www.python.org/ftp/python/3.9.13/python-3.9.13-amd64.exe)
   - [Python 3.8.10](https://www.python.org/ftp/python/3.8.10/python-3.8.10-amd64.exe)
   - [Python 3.7.9](https://www.python.org/ftp/python/3.7.9/python-3.7.9-amd64.exe)
2. Run the installer. **Do not click `Install Now`**. Tick `Install launcher for all users (recommended)` if it isn't already selected. **Make sure to tick `Add Python 3.X to PATH`. It is very important that you do this.** Click `Customize installation`.
3. Tick *all* optional features (including adding the `py launcher` for all users). Click `Next`.
4. Tick all options, except `Download debugging symbols` and `Download debug binaries (requires VS 2017 or later)`. After you tick `Install for all users`, the install location should now be `C:\Program Files\Python3X`. If it isn't, change it so that it is. Click `Install`. (Note: the main reason for installing to `Program Files` instead of the default user installation location, `AppData`, is that Windows can be a little problematic about write access to `AppData`. Usually it works, but occasionally it doesn't. Whatever makes it suddenly decide not to function seemingly at random is beyond me.)
5. After the installation finishes, open a terminal and run `py -0p`. You should now see your new installation and the correct path! If not, you haven't added it to your `PATH` correctly. In the Windows Settings under `Apps > Installed Apps`, click the three dots next to the specifc Python version, and click `Modify`. A similar dialog to the installer will appear, where you can change various options (i.e., tick and untick the various installation options). Select `Add Python 3.X to PATH` and make a note to read instructions more carefully next time.

Congratulations! You have installed Python, but you're not done yet. Make sure you keep reading below to see how to properly manage your installation.

## Managing multiple versions

You may have noticed that in all the instructions above, we've used `py` instead of `python` when running terminal commands - why is that? First, we need to look at how the terminal figures out what to do when we run a command like `python` or `pip`. When you run `COMMAND`, the process is *roughly* as follows:

1. If `COMMAND` is a valid terminal command (e.g., `dir`, `cd`, `echo`, et cetera), that command is run.
2. Otherwise, if `COMMAND` is an explicit path to an executable or batch file (e.g., `"C:\Program Files\Python 311\python.exe"`), that file is run.
3. Otherwise, Windows begins searching entries of the system `PATH` to find an executable or batch file named `COMMAND` to execute. For each directory entered in the `PATH`, in the order that they are listed, the terminal searches for `COMMAND.exe` or `COMMAND.bat`. If a file is found, the search stops and the terminal executes that file.
4. Otherwise, if step 3 exhausts the search of the `PATH` and no matching files are found, an error is raised.

This presents a problem if there are multiple versions of the same executable (in this case, `python.exe`) on the system. Because only the first executable is run, whichever Python version happens to be first in the `PATH` entries will be run. *Usually*, this is the most recently installed version, which is *usually* the latest Python version. However, depending on which order you installed different Python version in, you will end up running different Python versions - hardly desirable behaviour.

To deal with this, Python has a specific program, the *Python launcher for Windows* which is installed as a separate app. It's purpose is to use various heuristics to determine which installed Python version should be used to execute a specific Python command.

Running `py -0p` lists all the Python versions the launcher can find in order of preference, along with their paths; `py -0` will do the same but without the paths. For example, if you have Python 3.8 through to 3.11 installed, you should see something like this:

```
> py -0p
 -V:3.11 *        C:\Program Files\Python311\python.exe
 -V:3.10          C:\Program Files\Python310\python.exe
 -V:3.9           C:\Program Files\Python39\python.exe
 -V:3.8           C:\Program Files\Python38\python.exe
```

Note that the `*` denotes the default Python version, and so running `py` without any additional arguments will take you straight to the usual Python REPL:

```
> py
Python 3.11.0 (main, Oct 24 2022, 18:26:48) [MSC v.1933 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

To choose another version instead, we simply add the `-3.X` argument to run Python version `3.X`:

```
> py -3.8
Python 3.8.9 (tags/v3.8.9:a743f81, Apr  6 2021, 14:02:34) [MSC v.1928 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

By setting the environment variable `PYLAUNCHER_DRYRUN` to any value, we can see what command the launcher is executing under the hood, which is explicitly specifying the full path to the desired Python executable:

```
> set PYLAUNCHER_DRYRUN=1

> py
"C:\Program Files\Python311\python.exe"

> py -3.8
"C:\Program Files\Python38\python.exe"
```

This is useful, but the most useful feature is specifying which version to install packages to. You may be used to running `pip` directly as a command, which has the same problem that Windows uses the first entry in the `PATH`. However, `pip` is actually written as a Python module, and can be invoked with the `-m` flag (see [command line options](./command-line-usage-and-pip.md) for more information); the `pip.exe` bundled with a Python installation is mostly for convenience. To install a package `<package>`, instead of the usual `pip install <package>`, we would run `py -3.X -m pip install <package>`, which installs it to Python `3.X`. Again, we can see that under the hood, the launcher just explicitly specifies *which* Python executable is used to run the command - `py -3.X <command>` does not change the command in any way compared to running it as just `python <command>`:

```
> py -3.9 -m pip list
"C:\Program Files\Python39\python.exe" -m pip list
```

## Summary

- Follow the instructions above to [correctly download and install Python](#download-and-installation).
- Use the Python launcher `py` to specify which Python version should be used to run a command:
  - `py -0p` lists the installed Python versions and their paths.
  - `py` runs the default version's `python.exe` and enter the usual REPL.
  - `py -3.X` enters the REPL for version `3.X`.
  - `py -3.X -m pip install <package>` installs `<package>` for version `3.X`.
