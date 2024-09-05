# Cheatsheet of CLI Commands for DevOps Tools/Applications

## Contents

* [Vagrant](#vagrant-local-vms)

* [Git](#git-versioning)

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

* `git branch -M main`: Set master branch 

### Make Changes to a Repo

* `git status`: Show modified files in the working directory

* `git add .` or `git add *` or `git add [file_name]`: Add current directory files (`.`)
or all the files in the directory (`*`) or a specific file (`[file_name]`) to your next
commit (stage)

* `git reset [file_name]`: Unstage a file while retaining the changes in the working
directory

* `git diff`: Show the differences of what is changed but not staged

* `git diff --staged`: Show the differences of what is staged but not yet committed

* `git commit -m "[message]"`: Commit your staged content as a new commit snapshot with
a descriptive `[message]` of your changes.

* 

* 
