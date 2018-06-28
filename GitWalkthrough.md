# Git Walkthrough

Author:  Peter Gartland
Date:    6/21/2018
Version: 1.0

This guide is intended to be a quick intro and useful reference for people new to using Git and those who just can't seem to remember the git clone command.

## 1. Brief overview of Git and version control


## 2. Cloning a repo
When starting work on a project, the first step is usually to clone the project's Git repo to your workspace with the following command:  `$ git clone -c http.sslVerify=false -b <branchName> https://code.in.dynetics.com/scm/git/ProjectName <location>`

This will download the entire git repo to the location you specify.  If no location is given, it will place the repo in a directory named `ProjectName` in your current directory.

  - The `-c http.sslVerify=false` option disables SSL verification for this command.
  - The `-b <branchName>` option will automatically check out the branch specified when the download finishes.  If you leave off this option, the default branch is the `master` branch.

From Levi's sacred git setup guide:

  "Because we use a git server with a self-signed SSL certificate, we need to disable certificate checking when cloning the repository. After checkout, we need to disable certificate checking for the repository. Also, set the push default to “simple” so only the current branch is pushed."

After cloning, `cd` into the repo, and set the following config options:

    $ git config --global http.sslVerify false
    $ git config --global push.default simple


## 3. Checking out a branch
You can see which branch you currently have checked out with the `git status` command.  If you haven't made any changes or added any new files to the repo, the output should look similar to the following.

    $ git status
    On branch master
    nothing to commit, working tree clean

Typically, you will want to work in a branch other than `master`.  To see what branches currently exist, use the command

    $ git branch -a

With the `-a` option, the list will include your local branches as well as all the ones in the remote repo.  Your current branch will be highlighted.

As for switching to an existing branch, use the following command assuming the branch `devel` exists:

    $ git checkout devel


To create your very own local branch called `newFeature`, use the following command.  You'll also need to switch to it using git checkout.

    $ git branch newFeature
    $ git checkout newFeature


To create a new branch and switch to it in a single command, pass the `-b` option to git checkout:

    $ git checkout -b newFeature


Note that when you create a new branch, all of the previous branch's history comes with it.  You're now ready to start working in your new branch!


## 4. Committing your changes
After you've made changes to files in the repo or added new ones, the `git status` command will show a list of all the modified files separated into two groups:

    $ git status
    On branch newFeature
    Changes not staged for commit:
      <list of modified files>

    Untracked files:
      <list of new files>

The "Changes not staged for commit" list shows modified files that are part of the repo already, and the "Untracked files" list shows any new files that have not been committed to the repo yet.  Before actually committing changes, you have to choose which of the files from the git status list you want to commit.  In Git lingo, this is called "staging files for commit" and is done with the `git add` command:

    $ git add <file1> <file2>...

There are some useful options that make adding multiple files easier.  For example, `git add -u` will stage all of the modified files but none of the untracked files.  If you accidentally stage the wrong file, use `git reset HEAD <file>` to unstage it.  After choosing which files you want to commit, the `git status` output will move them to a third group labeled "Changes to be committed".

    $ git status
    On branch newFeature
    Changes to be committed:
      <list of files staged for commit>

    Changes not staged for commit:
      <list of other modified files>

    Untracked files:
      <list of other new files>

To perform the commit, simply use

    $ git commit

This will automatically open a command line text editor and prompt you to write a commit message, which should be a short description of the changes you've made.

If you made a very small change, or you just don't like vim, use the `-m` option to pass your commit message as an additional argument:

    $ git commit -m "Message describing the changes."

A good rule of thumb is to only commit usable code, i.e. code that compiles and runs as bug-free as possible.  If necessary, make a note of any caveats in the commit message.


## 5. Interacting with remote repos
The git repos for every project are stored and backed up on the Dynetics network.  Besides cloning, there are two other commands for interacting with a remote repo:  `git pull` and `git push`.  They are straight forward commands and usually give good hints for what you need to do if you make a mistake.

Before pulling from or pushing to your new branch, you need to make sure it is being tracked by the remote repo.  To tell the remote repo to start tracking one of your local branches, use

    $ git push -u origin <branchName>

Use `git push` to upload your own local commits to the remote repo, and use `git pull` to update your branch with commits that other people have pushed to the remote repo.  In general, pushing and pulling often is good practice if multiple people are working in the same branch.  This allows everyone keep up with each other's progress and can help resolve conflicts earlier rather than later.  However, this also requires all parties to **avoid pushing code that does not compile or contains untested changes.**

Git pull is really a combination of two commands:  git fetch followed by git merge.


## 6. Merging branches
Usually, the goal is to merge your new branch back into the original one.  When you're happy and have made your final commit to your working branch, check out the parent branch and do a `git pull` to make sure you're up to date.  Then do

    $ git merge <workingBranchName>

to merge your working branch into the original branch.  If there are no conflicts, then you're done!  If there are conflicts, the merge will remain in progress until you resolve the conflicts or abort.  Grab the person whose changes conflict with yours, and use

    $ git mergetool

to aid in resolving these conflicts.

--------------------------------------------------------------------------------

## Appendix A. List of useful git commands
Use `git help <command>` to see the man page for a specific command.

  - git status
  - git branch
  - git checkout
  - git add
  - git commit
  - git log
  - git diff
  - git difftool
  - git pull
  - git push
  - git merge
  - git mergetool
  - git stash

--------------------------------------------------------------------------------

## Appendix B. Git Submodules
By default, when you add a submodule to your repo, it comes in the "detached HEAD" state.  This means it is treated as a single commit from the submodule's home repo and does not receive updates unless it is explicitly fast-forwarded to another commit (see #3 in Section B.1).  Future users who pull from your repo will get the submodule in the same detached HEAD state that was last committed to your repo.  Because of this feature, submodules should be treated more like libraries than actual repos.

### B.1 How to pull an existing submodule into a newly cloned repo:
1. Go to the root directory of your git repo and run

        $ git submodule init

2. Then run the following command to pull to the submodule:

        $ git submodule update

3. (Optional) To update the submodule to the latest commit from the remote repo, run the following command:

        $ git submodule update --remote

This command should be used with care, as updating the submodule to another commit may break code in your repo that depends on the previous version.

### B.2 How to add a submodule that does not already exist in your repo:
1. Go to the root directory of your repo.

2. Run the following command,

        $ git submodule add https://code.in.dynetics.com/scm/git/RepoName <path/for/submodule>

and `RepoName` will be placed in the `<path/for/submodule>` folder.

--------------------------------------------------------------------------------

## Appendix C. Gitignore Guide
Gitignore files keep your repo tidy by preventing specified files or directories from being tracked.

Some general features:

  - Files matched in a gitignore file do not appear in the output of `git status`.

  - Paths listed in a gitignore file are relative to that gitignore file's location.

  - A file or directory name by itself automatically matches at any depth beneath the gitignore file.

  - Gitignore files lower in the directory structure take precedence over higher level gitignore files; however, a lower level gitignore file will not be parsed if it is matched in a gitignore above it.

  - Within a gitignore file, the last line determines the outcome for a particular match.

The following is a list of directories in a git repo named ProjectName and the suggested contents of the gitignore files residing there.  Section C.1 uses many simple gitignore files spread throughout the repo to handle directories individually.  Section C.2 achieves the same result as Section C.1 but uses a single gitignore file in the root directory of the repo.

For more details, check out https://git-scm.com/docs/gitignore.

### C.1 Using Multiple .gitignore Files

#### C.1.1 ProjectName/.gitignore

    # Random log files generated by Vivado anywhere in the repo
    .Xil
    vivado*.jou
    vivado*.log
    vivado*.str
    vivado*.debug
    webtalk*.jou
    webtalk*.log

    # Compiled Python files
    *.pyc

#### C.1.2 ProjectName/firmware/.gitignore

    # Automatically generated Vivado project support files anywhere in the firmware directory
    *.cache
    *.hw
    *.ip_user_files
    *.runs
    *.sim
    *.srcs

#### C.1.3 ProjectName/firmware/projectBoard/ip/.gitignore

    # Everything in the IP directory except the .xci and .vho files
    */**
    !*/*.xci
    !*/*.vho

#### C.1.4 ProjectName/firmware/unitTests/.gitignore

    # Files generated by Modelsim in the unitTests directory except the .mpf file
    work
    *.cr.mti
    transcript
    vsim.wlf


### C.2. Using A Single .gitignore File

#### C.2.1 ProjectName/.gitignore

    # Random log files generated by Vivado anywhere in the repo
    .Xil
    vivado*.jou
    vivado*.log
    vivado*.str
    vivado*.debug
    webtalk*.jou
    webtalk*.log

    # Compiled Python files
    *.pyc

    # Automatically generated Vivado project files anywhere in the firmware directory
    firmware/**/*.cache
    firmware/**/*.hw
    firmware/**/*.ip_user_files
    firmware/**/*.runs
    firmware/**/*.sim
    firmware/**/*.srcs

    # All automatically generated IP files except the .xci and .vho files
    firmware/**/ip/*/**
    !firmware/**/ip/*/*.xci
    !firmware/**/ip/*/*.vho

    # Files generated by Modelsim in the unitTests directory except the .mpf file
    firmware/unitTests/**/work
    firmware/unitTests/**/*.cr.mti
    firmware/unitTests/**/transcript
    firmware/unitTests/**/vsim.wlf
