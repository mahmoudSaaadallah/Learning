
### Setup and Config
These commands are used to configure your Git environment and access help documentation.

| Command      | Usage/Description                                                                                           | Example Syntax                             |
| :----------- | :---------------------------------------------------------------------------------------------------------- | :----------------------------------------- |
| `git config` | Sets configuration variables that control how Git looks and operates at the system, global, or local level. | `git config --global user.name "John Doe"` |
| `git help`   | Displays comprehensive manual page documentation for any Git command.                                       | `git help config`                          |

### Getting and Creating Projects
These commands are the primary ways to obtain a Git repository to start working.

| Command | Usage/Description | Example Syntax |
| :--- | :--- | :--- |
| `git init` | Initializes a new Git repository in an existing directory, creating a `.git` subdirectory. | `git init` |
| `git clone` | Creates a copy of an existing Git repository, including its full history and all data. | `git clone https://github.com/libgit2/libgit2` |

### Basic Snapshotting
These commands form the core workflow for staging changes and recording snapshots of your project.

| Command        | Usage/Description                                                                                            | Example Syntax                    |
| :------------- | :----------------------------------------------------------------------------------------------------------- | :-------------------------------- |
| `git add`      | Adds file content from the working directory to the staging area (index) for the next commit.                | `git add README`                  |
| `git status`   | Displays the state of the working directory and staging area, showing modified, staged, and untracked files. | `git status`                      |
| `git diff`     | Shows differences between the working directory, staging area, and various commits.                          | `git diff --staged`               |
| `git difftool` | Launches an external graphical tool to view differences between trees.                                       | `git difftool`                    |
| `git commit`   | Records the current contents of the staging area as a new permanent snapshot in the Git directory.           | `git commit -m "Initial version"` |
| `git reset`    | Undoes changes by moving the HEAD pointer and optionally updating the index or working directory.            | `git reset --hard HEAD~`          |
| `git rm`       | Removes files from the staging area and working directory.                                                   | `git rm PROJECTS.md`              |
| `git mv`       | Renames or moves a file, directory, or symlink.                                                              | `git mv file_from file_to`        |
| `git clean`    | Removes untracked files and directories from the working directory.                                          | `git clean -f -d`                 |

### Branching and Merging
These commands manage development lines and integrate changes between different branches.

| Command | Usage/Description | Example Syntax |
| :--- | :--- | :--- |
| `git branch` | Lists, creates, renames, or deletes branches. | `git branch testing` |
| `git checkout` | Switches branches or restores working tree files. | `git checkout testing` |
| `git switch` | Specifically used to switch between branches or create new ones. | `git switch -c new-branch` |
| `git merge` | Integrates changes from one or more branches into the current branch. | `git merge hotfix` |
| `git mergetool` | Launches an external merge helper to resolve merge conflicts. | `git mergetool` |
| `git log` | Displays the history of commits reachable from the current branch. | `git log --oneline --graph` |
| `git stash` | Temporarily shelves changes in the working directory to clean it without committing. | `git stash` |
| `git tag` | Creates, lists, or deletes tags to mark specific points in history as important. | `git tag -a v1.4 -m "msg"` |

### Sharing and Updating Projects
These commands interact with remote repositories to synchronize and share work.

| Command | Usage/Description | Example Syntax |
| :--- | :--- | :--- |
| `git fetch` | Downloads objects and refs from another repository without merging them into your local work. | `git fetch origin` |
| `git pull` | Fetches changes from a remote repository and immediately tries to merge them into the current branch. | `git pull origin master` |
| `git push` | Updates remote refs and associated objects with local changes. | `git push origin master` |
| `git remote` | Manages the set of tracked remote repositories. | `git remote add pb <url>` |
| `git archive` | Creates a compressed archive file (like zip or tar) of a specific project snapshot. | `git archive master --format=zip` |
| `git submodule` | Manages external Git repositories embedded within your project directory. | `git submodule add <url>` |

### Inspection and Comparison
These tools provide ways to examine Git objects and project summaries.

| Command | Usage/Description | Example Syntax |
| :--- | :--- | :--- |
| `git show` | Displays detailed information about Git objects like commits or tags. | `git show v1.4` |
| `git shortlog` | Summarizes log output, often grouping commits by author for use in changelogs. | `git shortlog --no-merges` |
| `git describe` | Finds a human-readable name for a commit based on the nearest tag. | `git describe master` |

### Debugging
These commands assist in finding bugs or tracking authorship of code changes.

| Command | Usage/Description | Example Syntax |
| :--- | :--- | :--- |
| `git bisect` | Uses a binary search to find the specific commit that introduced a bug. | `git bisect start` |
| `git blame` | Annotates each line of a file with the commit and author that last modified it. | `git blame -L 69,82 Makefile` |
| `git grep` | Searches through committed trees, the index, or the working directory for strings or patterns. | `git grep -n "pattern"` |

### Patching and History Rewriting
These commands allow for the modification of the commit history or application of individual changes.

| Command | Usage/Description | Example Syntax |
| :--- | :--- | :--- |
| `git cherry-pick` | Applies the changes introduced by an existing commit as a new commit on the current branch. | `git cherry-pick e43a6` |
| `git rebase` | Reapplies commits from one branch on top of another base commit. | `git rebase master` |
| `git revert` | Creates a new commit that applies the exact inverse changes of a targeted commit to undo it. | `git revert -m 1 HEAD` |
| `git filter-branch` | Rewrites large numbers of commits according to custom filters. | `git filter-branch --subdirectory-filter trunk HEAD` |

### Email Workflows
Git provides tools for contributing to projects that use mailing lists to manage patches.

| Command | Usage/Description | Example Syntax |
| :--- | :--- | :--- |
| `git apply` | Applies a patch to files in the working directory or index without creating a commit. | `git apply /tmp/patch.patch` |
| `git am` | Applies a series of patches from an email mailbox (mbox) and automatically creates commits. | `git am 0001-limit.patch` |
| `git format-patch` | Prepares patches for email submission by turning commits into mbox-formatted files. | `git format-patch -M origin/master` |
| `git imap-send` | Uploads a patch series to an IMAP drafts folder. | `cat *.patch \| git imap-send` |
| `git send-email` | Sends patches generated by `format-patch` directly via email. | `git send-email *.patch` |
| `git request-pull` | Generates a summary of changes and a URL for a maintainer to pull changes from. | `git request-pull origin/master myfork` |

### Administration and Internals
Low-level commands used for repository maintenance, data recovery, and internal manipulation.

| Command | Usage/Description | Example Syntax |
| :--- | :--- | :--- |
| `git gc` | Optimizes the repository by packing objects and removing unreachable data. | `git gc --auto` |
| `git fsck` | Verifies the connectivity and validity of the objects in the database. | `git fsck --full` |
| `git reflog` | Displays a log of where the HEAD and branch references have been locally. | `git reflog` |
| `git cat-file` | Inspects the content, type, or size of Git objects at a low level. | `git cat-file -p <sha>` |
| `git hash-object` | Computes the SHA-1 of data and optionally writes it to the object database. | `git hash-object -w file` |
| `git replace` | Allows you to specify that one object in Git should be treated as another without rewriting history. | `git replace <old> <new>` |
| `git credential` | Interacts with credential helpers to store and retrieve passwords. | `git credential fill` |
