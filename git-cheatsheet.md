<h1 align='center'>Git Cheatsheet</h1>

This document is intended to be a quick reference for common Git operations and commands.

**Table of Contents:**
- [Introduction and terminology](#introduction-and-terminology)
- [Initial setup](#initial-setup)
- [Clone a repository](#clone-a-repository)
- [Working with branches](#working-with-branches)
- [Undoing things](#undoing-things)
  - [Editing the last commit](#editing-the-last-commit)


## Introduction and terminology

Note: values in angle brackets and italics, *`<like this>`*, should be replaced with the actual parameters when running the commands.

## Initial setup

First set up an SSH key if you have not done so already. I recommend using your GitHub anonymous email, visible [here](https://github.com/settings/emails). All the commands in the following list of steps should be run in Git Bash on Windows.

1. Create a new key (if you do not have one already).
   1. GitHub: <code>ssh-keygen -t ed25519 -C "<i>&lt;your email address&gt;</i>"</code>
   2. GitLab: <code>ssh-keygen -t rsa -b 4096 -C "<i>&lt;your email address&gt;</i>"</code>
2. Hit `Enter` a few times until the command finishes to skip generating a passphrase.
3. Start the SSH agent: `eval "$(ssh-agent -s)"`
4. Add the key to the SSH agent:
   1. GitHub: `ssh-add ~/.ssh/id_ed25519`
   2. GitLab: `ssh-add ~/.ssh/id_rsa`
5. Copy the public key (the entire output of the following command) and add it to your SSH keys in either service.
   1. GitHub: `cat ~/.ssh/id_ed25519.pub`
   2. GitHub: `cat ~/.ssh/id_rsa.pub`
6. The first time you use that key to clone a repository, you will get a warning about the authenticity of the host. Enter `yes` to continue.

Next, set your name and email address in your Git config:

<pre>
git config --global user.name "<i>&lt;your full name&gt;</i>"
git config --global user.email "<i>&lt;your email address&gt;</i>"
</pre>

## Clone a repository

| Task | Command |
| --- | --- |
| Clone a repository from GitHub | <code>git clone git@<span>github.com:<i>username&gt;</i>/<i>repository&gt;</i></code> |
| Clone a repository from GitLab | Use `git clone` with the specific `git@...` command provided by GitLab. |

## Working with branches

| Task | Command |
| --- | --- |
| Switch to a local branch | <code>git switch <i>&lt;branch name&gt;</i></code> |
| Switch to a remote branch | <pre>git fetch<br>git switch <i>&lt;branch name&gt;</i></pre>
| Publish a branch | <code>git push -u origin <i>&lt;branch name&gt;</i></code> | 
| Create a new branch from `HEAD` | <code>git switch -c <i>&lt;branch name&gt;</i></code> |
| Rename a branch | <code>git branch -m <i>&lt;new branch name&gt;</i></code> |
| Delete a branch | <code>git branch -d <i>&lt;branch name&gt;</i></code> |
| Forcefully delete an unmerged branch<br>**Warning: this will destroy unmerged work** | <code>git branch -D <i>&lt;branch name&gt;</i></code> |

## Undoing things

### Editing the last commit

| Task | Command |
| --- | --- |
| Change the message of the last commit without adding files | <code>git commit --amend -m "<i>&lt;new message&gt;</i>"</code> |
| Add a file to the last commit without changing the message | <pre>git add <i>&lt;file&gt;</i> [<i>&lt;file 2&gt; ...</i>]<br>git commit --amend --no-edit</pre> |
| Add a file and change the message of the last commit | <pre>git add <i>&lt;file&gt;</i> [<i>&lt;file 2&gt; ...</i>]<br>git commit --amend -m "<i>&lt;new message&gt;</i>"</pre> |
