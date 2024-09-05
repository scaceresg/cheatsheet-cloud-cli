# Cheatsheet of CLI Commands for DevOps Tools/Applications

## Contents

* [Vagrant](#vagrant-local-vms)

* [Git](#git-versioning)

* [GitFlow](#gitflow)

* [Docker](#docker)

---
## Vagrant (Local VMs)

### Create a `Vagrantfile`

* `vagrant -v`: Check Vagrant version

* `vagrant init`: Initialise Vagrant with a non-specific Vagran­tfile and ./.vagrant
directory

* `vagrant init [url-to-vagrant-box]`: Initialise Vagrant with a specific box

### Manage Boxes

* `vagrant box list`: List the installed vagrant boxes

* `vagrant box remove [box_na­me/id] virtualbox`: Remove the `[box_name/id]` box

### Vagrant Commands

You need to be in the same directory as the Vagran­tfile:

* `vagrant up`: Start a Vagrant machine using a `Vagrantfile`

* `vagrant up --prov­ision | tee provis­ion.log`: Start a Vagrant machine, forcing
provisioning and saving all the output logs to `provision.log`

* `vagrant status`: Show the status of the Vagrant machine

* `vagrant halt`: Stop a Vagrant machine

* `vagrant reload`: Load new `Vagrantfile` configuration

* `vagrant ssh`: Connect to the machine via SSH

* `vagrant destroy`: Stop and delete the Vagrant machine

* `vagrant global­-status`: Show the status of all Vagrant machines

* `vagrant global­-status --prune`: Show the global status, removing the invalid
entries

---
## Git (Versioning)

### Set Up Git Config

* `git config --global user.name "[first_name last_name]"`: Set global username

* `git config --global user.email "[valid-email]"`: Set global email of username

* `cat .git/config`: Check the contents of the configuration file

**REMEMBER**: Use SSH Keys (`ssh-keygen -t rsa` in Linux) to connect with GitHub repos

### Initiate Git

* `git init` or `git init [directory_path]`: Initialise a directory as a Git repository

* `git clone [url_to_repo].git` or `git clone -b [branch_name] [url_to_repo].git`:
retrieve a repository from a location via URL. `-b [branch_name]` to specify the branch

* `git remote show origin`: Check the current remote URL of your configuration file

* `git remote set-url origin [new_url_to_repo].git`: Add or change the remote origin

* `git branch -M main`: Set master branch as `main`

### Update Changes from the Repo

* `git pull`: Fetch and merge any commits from the tracking remote branch

* `git fetch origin [branch_name]`: Fetch a specific `[branch_name]`, from the repo

### Make Changes to a Repo

* `git status`: Show modified files in the working directory

* `git add .` or `git add *` or `git add [file_name]`: Add current directory files (`.`)
or all the files in the directory (`*`) or a specific file (`[file_name]`) to your next
commit (stage)

* `git reset [file_name]`: Unstage a file while retaining the changes in the working
directory

* `git diff`: Show the differences of what is changed but not staged

* `git diff --staged`: Show the differences of what is staged but not yet committed

* `git diff HEAD`: Show difference between working directory and last commit

* `git commit -m "[message]"`: Commit your staged content as a new commit snapshot with
a descriptive `[message]` of your changes.

* `git push -u origin [branch_name]`: Push local branch commits to the remote repository
branch

### Review Changes

* `git log -[num_limit]`: Limit the number of commits by `[num_limit]`. For example,
`git log -5` for 5 last commits.

* `git log --oneline`: Condense each commit to a single line

* `git log -p`: Display the full diff of each commit

* `git log --stat`: Include which files were altered and the relative number of lines that
were added or deleted from each of them

* `git log --author= "[pattern]"`: Search for commits by a particular author

* `git log --grep="[pattern]"`: Search for commits with a commit message that matches
`"[pattern]"`

* `git log [since]..[until]`: Show commits that occur between `[since]` and `[until]`

* `git show [commit_id]`: Show a specific commit using its identifier `[commit_id]` 

### Rollback Changes 

#### Unstaged

* `git checkout [file_name]`: Undo changes to a specific `[file_name]` before staging
compared to the remote repository

#### Staged

* `git restore --staged [file_name]`: Restores `[file_name]` from the staging area

* `git restore [file_name]`: Discard changes in `[file_name]`

* `git reset [file_name]`: Remove `[file_name]` from the staging area, but leave the working
directory unchanged. This unstages a file without overwriting any changes

#### Committed

* `git revert  [commit_id/name]`: Create a new commit that undoes all of the changes made
in `[commit_id/name]`, then apply it to the current branch (records history)

* `git revert HEAD`: Revert to the previous commit (records history)

* `git reset --hard [commit_id/name]`: Update the branch by adding or removing commits
based on the `[commit_id/name]`

* `git reset HEAD~[num]`: Reset a number `[num]` of commits backwards

### Branching & Merging

* `git branch`: List your branches. Argument `-a` lists all your branches

* `git branch [branch_name]` or `git checkout -b [branch_name]`: Create and check out a
new branch named `[branch_name]`

* `git checkout [branch_name]`: Switch to an existing branch

* `git merge [branch_name]`: Merge `[branch_name]` into the current branch

---
## GitFlow

![](https://wac-cdn.atlassian.com/dam/jcr:cc0b526e-adb7-4d45-874e-9bcea9898b4a/04%20Hotfix%20branches.svg?cdnVersion=2212)

1. Install `git-flow`: Linux: `apt-get install git-flow`

2. Create and push the `develop` branch from `main`:

  * `git checkout develop`
  
  * `git push -u origin develop`: 

3. Initialise: `git flow init`

4. Create/Finish a `[feature_name]` branch from `develop`:

  * Create: `git flow feature start [feature_name]`

  * Finish: `git flow feature finish [feature_name]`

5. Create/Finish a release with a `"[version]"` based on the develop branch:

  * Create: `git flow release start "[version]"`

  * Finish:
  ```
  git checkout master
  git checkout merge release/[version]
  git flow release finish "[version]"
  ```

6. Create/Finish a `[hotfix_name]` branch:

  * Create: `git flow hotfix start [hotfix_name]`

  * Finish: `git flow hotfix finish [hotfix_name]`

---
## Docker





