  
![alt text](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/progit.jpeg)

## The Pro Git book is available free to read online

**[Pro Git](https://www.git-scm.com/book/en/v2)**

# 1. Getting Started

> - Local Version Control (LVC)
> - Central Version Control (CVC)
> - Distributed Version Control (DVC)

## Local Version Control Systems

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git1.png)

## Centeral Version Control Systems

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git2.jpg)

## Distributed Version Control Systems

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git3.jpg)

## A Short History of Git

### A Short History of Git

> 2002 Linux Kerner in BitKeepers  
> 2005 From BitKeepers to Git
> 
> - Speed
> - Simple design
> - Support for non-linear development
> - Distributed
> - Scalable

## Git Basics

> - Differnt than others (eg Subversion or Perforce)
> - Snapshots Not Differences

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git4.jpg)

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git5.jpg)

## Git is different

> - Nearly Eveything is Local
> - Git Has Integrity
> - Git Generally Only Adds Data
> - The Three States

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git6.jpg)

### The Command Line (Cli vs GUI)

## Installing Git

`$ apt-get install git`

[http://git-scm.com/download/linux](http://git-scm.com/download/linux)

!sudo apt install git

!git --version #Check the current version of Git

## First-Time Git Setup

### Identity

> `$ git config --global user.name "John Doe"`  
> `$ git config --global user.email johndoe@example.com`

### Configurations

> - `$ git config --global` # User (global) -> ~/.gitconfig
> - `$ git config --system` # System -> /etc/gitconfig
> - `$ git config`  # Project -> /.git/conifg

### Text Editor

> `$ git config --global core.editor vi [or code 🙂]`

### Color Output

> `$ git config --global color.ui true`

!git config --global user.name "John Doe"

!git config --global user.email johndoe@example.com

!git config --global core.editor vi # or vim,vi,code, etc.

# Check your settings
!git config --list # or git config user.name , etc.

---

# 2. Git Basics

> - Configure and initialize a repository
> - Begin and stop tracking files and stage and commit changes
> - Set up Git to ignore certain files and file patterns
> - Undo mistakes quickly and easily
> - Browse the history of your project
> - View changes between commits
> - Push and pull from remote repositories

## Getting a Git Repository

### Initializing a Repository in an Existing Directory

# Make sure you are at the right top level directory
!git init

### The code base goes like this

> `$ cd /path/to/my/codebase`  
> `$ git init`  
> `$ git add .`  
> `$ git commit`

So far nothing is tracked. To start tracking you must specifiy what to track in the directory using `git add`.  
Examples:  
`$ git add *.py` # only track files with .py extensions  
or  
`$ git add .` # track everything  
`$ git README` # only track the file called README

### Cloning an Existing Repository

Cloning downloads everything with history!  
It also initializes the local repo by copying the .git folder from remote:  
`$ git clone https://github.com/libgit2/libgit2`

To create a new directory for the cloned repo:  
`$ git clone https://github.com/libgit2/libgit2 myLibgit`

!git clone https://github.com/libgit2/libgit2

## Recording Changes to the Repository

**Remember!**  
Each file is:

- _Untracked_: (Git does not care about it but will keep alerting you)
- _Tracked_:
    - Unmodified (Either never changed since tracked, or commited and never changed afterwards)
    - Modified (Either doesn't have a snapshot or been changed since last commit)
    - Staged (Had some changes and ready to be committed)

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git7.jpg)

## Checking the Status of Your Files

**Check the status of the file by:**  
`$ git status`  
`$ git status -s`

!git status

## Tracking New Files

`$ git add README`

!git add README

## Staging Modified Files

**What happens when you modify a staged (not yet committed) file?**  
**What options do we have?**

## Ignoring Files

- **Use .gitgnore file**
    
- **Use RegEx or glob patterns**
    
- Examples:
    
    - # a comment - this is ignored
    - *.a # no .a files
    - !lib.a # but do track lib.a, even though you're ignoring .a files above
    - /TODO # only ignore the root TODO file, not subdir/TODO
    - build/ # ignore all files in the build/ directory
    - doc/*.txt # ignore doc/notes.txt, but not doc/server/arch.txt
- More templates for different languages are in [This GitHub Repo](https://github.com/github/gitignore)
    

## What has changed?

To see the changes between WD, Staged and Committed use `git diff`

> `$ git diff # To show differences between WD and staged area`  
> `$ git diff --cached # To show differencce between staged and Commiited`

### Show changes between Commits (Compare commits)

> `$ git diff <begincommitrefs>..<endcommitrefs> --color-words` # Refs is the SHA of the commit

!git diff 

## Committing Your Changes

`$ git commit` # this will launch the editor you chose earlier  
`$ git commit -m "<msg>"` # this will commit using the provided message

**Remember** that git commit will record changes in the staging area only.  
Any modified but not staged files will remain the same

## Skipping the Staging Area

`$ git commit -a -m "<msg>"`

## Removing Files

`$ git rm <filename> # removes file from tracking and from filesystem`  
`$ git rm --cached <file> # removes file from staging (i.e. unstaging file)`

**Remember** that even operations on the file level need to be committed!

## Moving Files

Git doesn’t explicitly track file movement. If you rename a file in Git,  
no metadata is stored in Git that tells it you renamed the file ☹️​

`$ git mv file_from file_to`

## Viewing the Commit History

**Very simple!**

> `$ git log`

Some formatting could be useful when you have a long complex history.  
Try cloning the following repo and check its commit history.

> `$ git clone https://github.com/schacon/simplegit-progit`

**Try:**

> `$ git log -p -2`  
> `$ git log --stat`  
> `$ git log --pretty=oneline`  
> `$ git log --pretty=format:"%h - %an, %ar : %s"`  
> `$ git log --pretty=format:"%h %s" --graph`

**Limiting Log Output**

> `$ git log --since=2.weeks`

**Show Commit**

> `$ git show <commitrefs>` # Refs is the SHA of the commit

## Undoing Things

### Change the most recent Commit

**If you forgot to add files or messed up the commit message**

> `$ git commit --amend`

**The following results in a single commit**

> `$ git commit -m 'initial commit'`  
> `$ git add forgotten_file`  
> `$ git commit --amend`

### Untracking tracked file

> `$ git rm --cached <filename>`

### Unstaging staged file

**NOT RESTORING A PREVIOUS VERIOSN!**

> `$ git restore --staged <filenmame>` # recent command  
> `$ git reset HEAD <filenmame>` # old command but still works so far

### Unmodifying a Modified File

**REVERTING IT FROM THE LAST COMMIT**

> `$ git restore <filename>`

### Restore (revert) a committed change

**This will revert everything committed in a specific commit. That's why it's better to have atomic commits**

> `$ git revert <commitrefs>`

### Deleted Files!

**Deleted a file but not committed**

> `$ git checkout HEAD <filename>`

**Deleted a file and committed**

> `$ git reset --hard HEAD~1`

## Working with Remotes

### Showing Your Remotes

> `$ git clone https://github.com/schacon/ticgit`  
> `$ git remote`  
> `$ git remote -v`

### Add a remote to an existing repo

> `$ git remote add [shortname] [url]`

### Fetching from Remote

fetches any new work that has been pushed to  
that server since you cloned (or last fetched from) it**

> `$ git fetch [remote-name]`

**IMPORTANT: Fetch doesn’t automatically merge it with any  
of your work or modify what you’re currently working on.  
You have to merge it manually into your work when you’re ready.**

### Pushing to Your Remotes

**Means uploading any changes you have made to the remote that**

> `$ git push [remote-name] [branch-name]`

### Inspecting a Remote

**See more information about a particular remote**

> `$ git remote show [remote-name]`  
> `$ git remote show origin`

### Removing and Renaming Remotes

> `$ git remote rename [current-remote-name] [new-remote-name]`

**If you want to remove a remote for some reason—you’ve moved  
the server or are no longer using a particular mirror,  
or perhaps a contributor isn’t contributing anymore**

> `$ git remote rm [remote-name]`

### Inspecting a Remote

**see more information about a particular remote**

> `$ git remote show [remote-name]`  
> `$ git remote show origin`

## Tagging

**Tagging is like marking a specific commit at a specific  
point in history. Typically used to mark release points**  
(e.g. v1.0, v1.5, etc.)

### Listing your tags

> `$ git tag`

### Creating Tags

**Two types of tags:**

- Lightweight: very much like a branch that doesn’t change—it’s just a pointer to a specific commit.
- Annotated: stored as full objects in the Git database.

### Annotated Tags

**can be created when committing**

> `$ git tag -a <version> -m "<msg>"`  
> `$ git show <version>`

### Lightweight Tags

**Just a commit checksum added to a file, no more info stored.**

> `$ git tag <version>`  
> `$ git show <version>`

### Tagging Later

**If you have some previous commits and you forgot to tag  
you can later on tag a previous commit by referring  
to its checksum**

> `$ git tag -a <version> [commit-checksum]`

**Remember that you can retrieve the commits checkshum by using the following**

> `$ git log --pretty=oneline`

### Sharing Tags

**By default git push doesn't push tags. You mush specify**

> `$ git push [remote-name] [tagname]`

---

## 3. Git Branching

### Revise how Git stores data

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git8.jpg)

**A branch in Git is simply a lightweight movable pointer to one of these commits.**

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git9.jpg)

### Creating a New Branch

### To create a new branch without switching to it

`$ git branch testing`

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git10.jpg)

**After creating the branch without switching to it the HEAD pointer will still be pointing at the current branch**  
![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git11.jpg)

# Typical code
!cd /.git/refs/heads # A file for each branch, currently only a file named 'master'
!git branch testing  # A new branch called 'testing' is created
!git branch          # list all existing branches
!git show HEAD       # will show that the current HEAD points to both branches master and testing
!ls .git/refs/heads # now the directory has two files: master and testing
!git log --oneline --decorate # to see where the HEAD is currently pointing at, for now it should point at both branches

### Switching Branches

`$ git switch <branch-name>` # The new command starting 2.23  
`$ git checkout <branch-name>` # The old command

**Switching to another branch simply moves the HEAD poiter to that branch**  
![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git12.jpg)

**While by default the new branch will start at the current revision you can still create a new branch at a previous revision by adding the revision's sha at the end of the command** `git branch <branch-name> <SHA>`

!git switch testing # to switch to testing branch
# do some change to any file in the working tree
!git commit -am "A first commit to the testing branch" # A commit is made and the HEAD pointer will point at testing only
!git log --oneline # make sure that now HEAD points at testing only
!git switch master # switch back to master
!git log --online # now HEAD points at master only
!git log --all --online #  '--all' easier to show the log for all branchers regardless of the current branch

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git13.jpg)

**IMPORTANT: switching to a different branch not only moves the HEAD pointer to that branch but also reverts the contents of the working tree to that point in time. Moving the HEAD is like rewinding**

# To watch the HEAD movement in action try this
!git switch testing # Switch to testing branch
# Change something in README.txt
!git commit -am "Changes README.txt" # commit the changes
!git switch master # switch back to master branch
# Check the README file and make sure it was reverted back

### Divergent History

> 1. Created and switched to a branch
> 2. Did some work on it
> 3. Switched back to your main branch
> 4. Did other work

**Both of those changes are isolated in separate branches**![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git14.jpg)

#print out the history of your commits, showing where your branch pointers are and how your history has diverged.
!git log --oneline --decorate --graph --all 

## Merging

# do some changes to a file then commit it (c0 is the initial commit, c1 and c2 are two subsequent commits)
# now the current branch is master and HEAD points to it at C2

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git15.jpg)

!git checkout -b iss53 #creates a branch called iss53 and switches to it
# now the both branches are at C2 and the HEAD is now pointing to iss53

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git16.jpg)

# do some change to the file and commit 
# now the iss53 branch is at C3 and the master branch is left at C2 and the HEAD is still pointing to iss53

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git17.jpg)

!git checkout master # switches back to master branch
!git checkout -b hotfix # creates a new branch called hotfix and switches to it
# now the current branch is hotfix and it's at C4, master is still left at C2 and the HEAD points to hotfix

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git18.jpg)

!git checkout master # switches back to master
!git merge hotfix 
# merges hotfix branch with master, in other words fast-forwards master to C4 commit and the HEAD now points at master

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git19.jpg)

!git branch -d hotfix # -d option deletes the branch hotfix as it's no longer needed
!git checkout iss53 # switches back to branch iss53 to continue working on the issue
# do some changes
!git commit -am "some changes committed at iss53" # commit them, now you have C5 where iss53 branch is currently at

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git20.jpg)

**Now what happens if you want to merge iss53 branch into master?**  
**It is not the same as what we did with hotfix as now the current commits (C5 in iss53 and C4 in master) don't have the same ancestor**

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git21.jpg)

**Instead of just moving the branch pointer forward, Git creates a new snapshot that results from this three-way merge and automatically creates a new commit that points to it. This is referred to as a merge commit, and is special in that it has more than one parent.**

!git checkout master
!git merge iss53

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git22.jpg)

!git branch -d iss53 # now you can delete the iss53 branch as it's no longer needed

### Merge Conflicts

**If you try to do merge commit with conflicting files in both branches git will notify you and abort the process** **To check the conflicting portions of the file use `git status` then resolve the conflict then commit then merge again**

!git branch -v # to list all branches along with the last commit per each branch. * character indicates the current HEAD pointer

!git branch --merged # to view branches already merged with the current branch
!git branch --no-merged # list all branches that aren't merged with the current
!git branch -d testing # will attempt to delete the testing branch but will fail if it has unmerged work
!git branch -D testing # will forcibly delete the testing branch even if it has unmerged work
!git branch --move bad-branch-name corrected-branch-name # rename a branch. CAREFUL not to rename a branch currently shared
!git branch --move master main # you can also rename the master branch as it is exactly similar to any other branch

### Remote Branches

# Walkthrough demo for remote branch workflow and (fethc, pull, push)

# 1. Create remote project with 3 commits
!git mkdir ~/remotegit 
!cd ~/remotegit

## Add line and commit
!echo "This is the first line in the file create at the remote master branch" >> file.txt
!git init
!git add .
!git commit -m "Initial Commit"

## Add line and commit
!echo "Second line in file at remote master branch" >> file.txt
!git add .
!git commit -m "Second commit after adding line 2"

## Add line and commit
!echo "Third line in file at remote master branch" >> file.txt
!git add .
!git commit -m "Third commit after adding line 2"

## List commits. You should see 3 commits at master branch
!git log --oneline

# 2. Clone the "remote" branch and rename the local clone to localgit
!cd ~
!git clone ~/remotegit localgit
!cd ~/localgit

## List the current commits after cloning, you should see 3 commits with the exact SHA as the remote
!git log --oneline

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git23.jpg)

# 3. Make changes to the master branches in both repos at the same time

# Change the remote master branch
!cd ~/remotegit
!echo "First line at the remote master branch after cloning" >> file.txt
!git add .
!git commit -m "First commit at remote branch after cloning"

# Repeat one more time
!echo "Second line at the remote master branch after cloning" >> file.txt
!git add .
!git commit -m "Second commit at remote branch after cloning"

# List the current commits: you should be able to see 5 entries
!git log --oneline

# Change the local master branch
!cd ~/localgit
!echo "First line at the local master branch after cloning" >> file.txt
!git add .
!git commit -m "First commit at local branch after cloning"

# Repeat one more time
!echo "Second line at the local master branch after cloning" >> file.txt
!git add .
!git commit -m "Second commit at local branch after cloning"

# List the current commits: you should be able to see 5 entries
# Notice the origin/master and origin/HEAD still points at the inital location after cloning
!git log --oneline

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git24.jpg)

# 4. Synchronize work in local with changes make at the remote repo (origin) by git fetch

!git fetch origin

# verify that the local log now has new references as in remote
# now you should see 7 refs: 3 original + 2 from remote + 2 from local
# can you understand the graph?

!git log --oneline --all --graph

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git25.jpg)

# Check the file in the local repo and make sure it hasn't changed despite the git fetch
!cat ~/localrepo/file.txt

# To harden the changes made remotely to the work tree you will need to merge the remote master into the local master
# If you want to fetch and merge in the same time you should use git pull

# 5. Create a local branch and push it to remote (synchronize upstream)

!git checkout -b serverfix 
!echo "First line in file at local serverfix branch" >> file.txt
!git commit -am "First commit at local serverfix branch"

# Synchronize (push) local branch to remote
!git push origin serverfix
!cd ~/remotegit
!git log --oneline # this won't show anything new as you must use --all to be able to see other branches
!git log --oneline --all 
# now you should be able to see the new serverfix branch and it's line of refs connecting to 
# the common ancestor at the remote master branch

# You can also get a summary by using git branch -v
!git branch -v

# you can also use git push to delete a remote branch
!git push --delete serverfix

### Rebasing

## Merging

`$ git checkout experiment`  
`$ git merge master`  
![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git26.jpg)  
![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git27.jpg)

## Rebasing

`$ git checkout experiment`  
`$ git rebase master`  
![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git28.jpg)![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git29.jpg)

## Git and VSCode

**Built-in Git support**  
**Gitlens extension**

# GitHub

- **What is GitHub?**
- **Create Account**
- **Clone Repo**
- **Create Repo**
    - Git operations on GitHub
    - GitHub operations
- **Fork Repo**