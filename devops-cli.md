# Cheatsheet of CLI Commands for DevOps Tools/Applications

**Contents**:

- [Cheatsheet of CLI Commands for DevOps Tools/Applications](#cheatsheet-of-cli-commands-for-devops-toolsapplications)
  - [Vagrant (Local VMs)](#vagrant-local-vms)
    - [Create a `Vagrantfile`](#create-a-vagrantfile)
    - [Manage Boxes](#manage-boxes)
    - [Vagrant Commands](#vagrant-commands)
  - [Git (Versioning)](#git-versioning)
    - [Set Up SSH Config for Git Accounts: Personal and Corporate](#set-up-ssh-config-for-git-accounts-personal-and-corporate)
    - [Configure Global Username and User emal for an Account](#configure-global-username-and-user-emal-for-an-account)
    - [Sign your Commits](#sign-your-commits)
    - [Initiate Git](#initiate-git)
    - [Update Changes from the Repo](#update-changes-from-the-repo)
    - [Make Changes to a Repo](#make-changes-to-a-repo)
    - [Review Changes](#review-changes)
    - [Rollback Changes](#rollback-changes)
      - [Unstaged](#unstaged)
      - [Staged](#staged)
      - [Committed](#committed)
    - [Branching \& Merging](#branching--merging)
  - [GitFlow](#gitflow)

---
## Vagrant (Local VMs)

### Create a `Vagrantfile`

* `vagrant -v`: Check Vagrant version

* `vagrant init`: Initialise Vagrant with a non-specific Vagran­tfile and 
./.vagrant directory

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

### Set Up SSH Config for Git Accounts: Personal and Corporate

* Go to Git Bash

* Generate an SSH key for each account:
  
  ```
  > ssh-keygen -t ed25519 -C "your.personal.email@email.com"
  > ssh-keygen -t ed25519 -C "your.corporate.email@company.com"
  ```

  - Make sure to provide a unique name for each key when prompted (e.g., 
  `id_ed25519_personal` and `id_ed25519_company`)

* Add keys to SSH agent:
  
  - Run SSH agent:
    
    ```
    eval "$(ssh-agent -s)"
    ```

  - Add keys:
  
    ```
    > ssh-add ~/.ssh/id_ed25519_personal
    > ssh-add ~/.ssh/id_ed25519_company
    ```

* Add the public keys (`.pub`) to each of the GitHub account settings:
  
  - Show the public key. For example:
    
    ```
    cat ~/.ssh/id_ed25519_company.pub
    ```

  - Copy the public key

  - Go to Settings > SSH and GPG keys > New SSH key
  
  - Give a name and paste the public key

* Configure SSH Config file:
  
  - Create a config file:
  
    ```
    vim ~/.ssh/config
    ```
  
  - Add configurations for each account:
    
    ```
    # Personal account
    Host your_personal_git_server # Or your personal Git server's hostname
      HostName github.com
      User git
      IdentityFile ~/.ssh/id_ed25519_personal

    # Corporate account
    Host your_corporate_git_server # Replace with your corporate Git server's hostname
      HostName github.com
      User git
      IdentityFile ~/.ssh/id_ed25519_corporate
    ```

* Test the SSH connection for each account:
  
  For example:
    
  ```
  ssh -T git@your_personal_git_server
  ```
  ```
  Hi [github_account]! You've successfully authenticated, but GitHub does not provide shell access.
  ```

* Clone a repository: 
  
  ```
  git clone git@your_personal_git_server:git_account/repo.git
  ```

  or 

  ```
  git clone git@your_corporate_git_server:git_account/repo.git
  ```

### Configure Global Username and User emal for an Account

* Set global username:
  
  ```
  git config --global user.name "[first_name last_name]"
  ```

  - Use the `--local` flag to set **corporate account** configurations only for
  the specific corporate repository directory (You need to `cd` to the repository
  directory)

* Set global email address:
  
  ```
  git config --global user.email "[valid-email]"
  ```

  - Use the `--local` flag to set **corporate account** configurations only for
  the specific corporate repository directory (You need to `cd` to the repository
  directory)

* Check the contents of the configuration file:
  
  ```
  cat .git/config
  ```

### Sign your Commits

Source: https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification

* Download GPG for CLI: https://www.gnupg.org/download/

In Git Bash:

* Generate a GPG key pair:
  
  ```
  gpg --full-generate-key
  ```

  - At the prompts, specify the kind, key size, expiration time, add user ID 
  (Enter the verified email address for your GitHub account), set a passphrase
  **save this passphrase as you will use it when signing your commits**

* List the long form of the GPG keys:
  
  ```
  gpg --list-secret-keys --keyid-form=long
  ```
  ```
  /Users/hubot/.gnupg/secring.gpg
  ------------------------------------
  sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
  uid                          Hubot <hubot@example.com>
  ssb   4096R/4BB6D45482678BE3 2016-03-10
  ```

* Copy the long form of the GPG key ID to use. For example: `3AA5C34371567BD2`

* Print the GPG key ID in ASCII armor format for the GPG key ID copied:
`3AA5C34371567BD2`

  ```
  gpg --armor --export 3AA5C34371567BD2
  ```

* Copy your GPG key, beginning with `-----BEGIN PGP PUBLIC KEY BLOCK-----` and
ending with `-----END PGP PUBLIC KEY BLOCK-----`

* Go to your GitHub account > Settings > SSH and GPG keys > New GPG key and 
add the complete key

* Set global `user.signingkey`  and `commit.gpgsign`:
  
  ```
  > git config --global user.signingkey 3AA5C34371567BD2
  > git config --global commit.gpgsign true
  ```

* Sign your commits:
  
  ```
  git commit -S -m "YOUR COMMIT MESSAGE"
  ```

  - You need to provide the passphrase you set up when you generated your GPG key

### Initiate Git

* `git init` or `git init [directory_path]`: Initialise a directory as a Git 
repository

* `git clone [url_to_repo].git` or `git clone -b [branch_name] [url_to_repo].git`:
retrieve a repository from a location via URL. `-b [branch_name]` to specify the 
branch

* `git remote show origin`: Check the current remote URL of your configuration file

* `git remote set-url origin [new_url_to_repo].git`: Add or change the remote 
origin

* `git branch -M main`: Set master branch as `main`

### Update Changes from the Repo

* `git pull`: Fetch and merge any commits from the tracking remote branch

* `git fetch origin [branch_name]`: Fetch a specific `[branch_name]`, from the repo

### Make Changes to a Repo

* `git status`: Show modified files in the working directory

* `git add .` or `git add *` or `git add [file_name]`: Add current directory files 
(`.`) or all the files in the directory (`*`) or a specific file (`[file_name]`) 
to your next commit (stage)

* `git reset [file_name]`: Unstage a file while retaining the changes in the 
working directory

* `git diff`: Show the differences of what is changed but not staged

* `git diff --staged`: Show the differences of what is staged but not yet committed

* `git diff HEAD`: Show difference between working directory and last commit

* `git commit -m "[message]"`: Commit your staged content as a new commit snapshot 
with a descriptive `[message]` of your changes.

* `git push -u origin [branch_name]`: Push local branch commits to the remote 
repository branch

### Review Changes

* `git log -[num_limit]`: Limit the number of commits by `[num_limit]`. For 
example, `git log -5` for 5 last commits.

* `git log --oneline`: Condense each commit to a single line

* `git log -p`: Display the full diff of each commit

* `git log --stat`: Include which files were altered and the relative number 
of lines that were added or deleted from each of them

* `git log --author= "[pattern]"`: Search for commits by a particular author

* `git log --grep="[pattern]"`: Search for commits with a commit message that 
matches `"[pattern]"`

* `git log [since]..[until]`: Show commits that occur between `[since]` and 
`[until]` commits

* `git show [commit_id]`: Show a specific commit using its identifier `[commit_id]`

* `git rm [file_name]`: Delete `[file_name]` from project and stage the removal 
for commit

* `git mv [existing-path] [new-path]`: Change an existing file path and stage 
the move

### Rollback Changes 

#### Unstaged

* `git checkout [file_name]`: Undo changes to a specific `[file_name]` before 
staging compared to the remote repository

#### Staged

* `git restore --staged [file_name]`: Restores `[file_name]` from the staging area

* `git restore [file_name]`: Discard changes in `[file_name]`

* `git reset [file_name]`: Remove `[file_name]` from the staging area, but leave
the working directory unchanged. This unstages a file without overwriting any 
changes

#### Committed

* `git revert  [commit_id/name]`: Create a new commit that undoes all of the 
changes made in `[commit_id/name]`, then apply it to the current branch 
(records history)

* `git revert HEAD`: Revert to the previous commit (records history)

* `git reset --hard [commit_id/name]`: Update the branch by adding or removing 
commits based on the `[commit_id/name]`

* `git reset HEAD~[num]`: Reset a number `[num]` of commits backwards

### Branching & Merging

* `git branch`: List your branches. Argument `-a` lists all your branches

* `git branch [branch_name]` or `git checkout -b [branch_name]`: Create and check 
out a new branch named `[branch_name]`

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