### Get the files in the staging area 
- As we know that there is a staging area that hold or index the tracked file and make them ready to be committed.
- But how could we see the files in the staging area?
```bash
git ls-files
```
- The `git ls-files` used to return all the files in the staging area.

- Also we could use this command with option `-s` to see the SHA for the staged files
```bash
git ls-files -s
100644 b7aec520dec0a7516c18eb4c68b64ae1eb9b5a5e 0     file.txt
```
- `b7aec520dec0a7516c18eb4c68b64ae1eb9b5a5e` This is the SHA for the file.txt.
---

### Remove file from staging area
- What if we by mistake added file the staging area how could we un-stage this file?
- We could use one of the following commands:
```bash
git rm --cached <fileName>
git restore --staged <fileName>
```
- The `git rm --cached` will remove file from the staging area, as the `git restore --staged` do.
- **What is the difference between those commands? and when to use which?**
- To answer this question we will imagen the following scenario, we have created a new file in our working directory and this file is still untracked like the following
```bash
git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        file.txt

nothing added to commit but untracked files present (use "git add" to track)
```
- So as the file is still untracked then there is no copies of the file in the repo, then after adding it to the staging area we can't use the normal `git restore --staged <filename>` command to untrack or remove it from the staging area, as this command depend on the copy of that file in the repo and we don't have any copies for this file so it will return the following error if we try.
```bash
git restore --staged file.txt
fatal: could not resolve HEAD
```
- This output means there are no copies in for this file in the repo, so the git can't restore the old copy from the repo.
- On the other hand we could use `git rm --cached <filename>` for this scenario, because this command doesn't need any copy from the repo it just delete the cached copy in the staging area.
```bash
git rm --cached file.txt
rm 'file.txt'
```

- The Other scenario is if we already have a copy of this file in the repo, which mean this file already committed before, but we modified it and we added it to the staging area, so the correct command here to un-stage the file is to use `git restore --staged <filename>`, because this command will rely on the last copy of this file in the repo to restore it to the stage area which mean delete the modified version of the file, so the file will return modified, but not added to the staging area.
```bash
git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   file.txt

git restore --staged file.txt

git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   file.txt

no changes added to commit (use "git add" and/or "git commit -a")
```


**Now the third Scenario**
- What if the changes that we made to this file is wrong and we need to get the file before these changes which is the last version in the stage area and repo then we could use `git restore <filename>`
```bash
git restore file.txt
git status
On branch master
nothing to commit, working tree clean
```
- As we can see in the this command after we restored the previous version and check the status we find `nothing to commit`, which means we get the last version from the staging are which is the version before modification.
---

### SHA for the committed files 
- When we commit any changes in a single file this commit will generate three files in the `.git/objects`, those three files will have names as a SHA.
	- One file for the **tree** 'directory'
	- One for the **Blob** 'file'
	- One for **commit** it self.
- What is important about these file is the `commit` one because through this file the git could know which Blob changes and which tree this blog belong to.
- All this happen through the `commit` file here.
- To see the data in this commit file we need first to know this commit file.
- So we use `git log` to return all the logs with SHA for each log, and this SHA is the name of the `commit file`
```bash
git log
commit 5fedaba1cda0f9253c6695f7c72fdcdddb723603 (HEAD -> master)
Author: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Thu Jan 8 22:45:25 2026 +0200

    initial commit
```
- Her the SHA is `5fedaba1cda0f9253c6695f7c72fdcdddb723603`. 
- Through this SHA, we could use read it to know the SHA for the `tree`, which will lead us the SHA for the `blob` that was changed.
- To read this `commit` file we use `git cat-file -p <SHA For Commit>`.
```bash
git cat-file -p 5fedaba1cda0f9253c6695f7c72fdcdddb723603
tree 15853f9f8c707ec3f3aee8702de10b8583516ab3
author Mahmoud <mahmoud.saadallah73@gmail.com> 1767905125 +0200
committer Mahmoud <mahmoud.saadallah73@gmail.com> 1767905125 +0200

initial commit
```
- Here as we can see in the first line of the output we get the SHA for the `Tree`, which is `15853f9f8c707ec3f3aee8702de10b8583516ab3`, and some other information like who made this commit, what is his email, when did he make it, and the message for the commit.
- Now after we know the SHA for the `tree` from the SHA of the `commit`, we could use this SHA with the same command to get the `blob`.
```bash
git cat-file -p 15853f9f8c707ec3f3aee8702de10b8583516ab3
100644 blob b7aec520dec0a7516c18eb4c68b64ae1eb9b5a5e    file.txt
```
- Here the output for this command is the SHA for the `blob` and its name.
- Now if we applied the same command for the SHA of the `blob`, will get the content of this file.
```bash
git cat-file -p b7aec520dec0a7516c18eb4c68b64ae1eb9b5a5e
Hello, Git
```
- As we can see we get `Hell, Git` sentence which is the content for that `blob`.
- So Through the SHA for the `commit`, we get all the information about this commit(`tree`, `blob`, `blob content`).


> Now if we add a new commit and check the `log` again we would find it like this .

```bash
git log
commit aca1723f6db3f4d195c75b39cd3464f6afe587bd (HEAD -> master)
Author: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Sun Jan 11 01:11:57 2026 +0200

    Adding new line

commit 5fedaba1cda0f9253c6695f7c72fdcdddb723603
Author: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Thu Jan 8 22:45:25 2026 +0200

    initial commit
```
- As we can see from the result of the log we have two commits and those commits order by the data from the newest to the oldest.

- If we try now to get the objects inside the `.git/objects` using `find` which is a `linux` command not a `git` command then the result will be like this.
```bash
find .git/objects -type f
.git/objects/15/853f9f8c707ec3f3aee8702de10b8583516ab3
.git/objects/59/3d87abfc8b770dd95f0b85d0d29480c4939e7c
.git/objects/5f/edaba1cda0f9253c6695f7c72fdcdddb723603
.git/objects/6a/bb415e0ae3450be0f323113bcca2af195ebbff
.git/objects/ac/a1723f6db3f4d195c75b39cd3464f6afe587bd
.git/objects/b7/aec520dec0a7516c18eb4c68b64ae1eb9b5a5e
.git/objects/d7/5076329671631201e3268738a9c98e33eb9aca
```
- New if we look to this result, we will find that there are six objects now in the `.git/objects` directory, as we said each `commit` creates three objects `SHA`s, so there are six objects for two commits.
- Also if we try to read the content of the fifth `SHA` (I know it from the `git log`), which is the content for the second `commit`, then we will get the result like this.
- But before we read this content we have to know that `git` take the first two characters from the `SHA` and make a directory with them and the rest of the `SHA` would be the file inside this directory, so the first `SHA` is `aca1723f6db3f4d195c75b39cd3464f6afe587bd`
```bash
git cat-file -p aca1723f6db3f4d195c75b39cd3464f6afe587bd
tree d75076329671631201e3268738a9c98e33eb9aca
parent 5fedaba1cda0f9253c6695f7c72fdcdddb723603
author Mahmoud <mahmoud.saadallah73@gmail.com> 1768086717 +0200
committer Mahmoud <mahmoud.saadallah73@gmail.com> 1768086717 +0200

Adding new line
```
- In the result, we have new line shows up(the second line) `parent 5fedaba1cda0f9253c6695f7c72fdcdddb723603`, we didn't see this line before with first commit, so what is this line?
- This line is the `parent` `SHA` which refers to the previous `commit` directly before this, so `git` treat the files like `Nodes` (linked list) connected to each other, each `commit` has the `SHA` for his `parent` object, and this philosophy makes the `git` trace all the changes and the history for the changes.


---

### How to see changes in the files 
- After we change the data inside a file and for some reasons, we need to see those changes, so we need to compare between the data inside the file in the working directory and the staging area ore between the changes in the staging area and the git repository then, we use `git diff <fileName>` command.
```bash
git diff file.txt
warning: in the working copy of 'file.txt', LF will be replaced by CRLF the next time Git touches it
diff --git a/file.txt b/file.txt
index 593d87a..573004b 100644
--- a/file.txt
+++ b/file.txt
@@ -1,2 +1,4 @@
 Hello, Git
+test line
 new line
+Third line in file.txt
```
- Here in the result to know the changes we have to look at the `+` sign after `@@ -1,2 +1,4 @@` line, because any added line will has a `+` sign at the begging, which means this line has been added to the file.
- Also if some line deleted it would start with `-` sign.

now let's add this file to the staging area, and use `git diff` again.
```bash
git add file.txt
git diff

```
- As we can see we will get nothing now because the file already in the staging area, which means the snapshot that inside the working director is the already inside the staging area, so there are no differences.
- What if we want to see the changes between the staging area and the repository before committing then we have to use `--staged` option with the `git diff` command.
```bash
git diff --staged 
diff --git a/file.txt b/file.txt
index 593d87a..573004b 100644
--- a/file.txt
+++ b/file.txt
@@ -1,2 +1,4 @@
 Hello, Git
+test line
 new line
+Third line in file.txt
```
- Here is the result which exactly as before.


---
- After committing the file we could see the changes that happened in specific commit without using `git cat-file`, because this is a boring way to go form the `commit` `SHA` to the `tree` `SHA` to the `Blob` `SHA` till we get the content of the file, and even when we go through all this process, we can't know the changes, cause it won't be marked like the `git diff`. 
- So we have to use `git show <Commit SHA>`, to display the changes that happened to this commit.
```bash
git show f12c6c5b17fa2805494110d476495bcb2d034b67
commit f12c6c5b17fa2805494110d476495bcb2d034b67 (HEAD -> master)
Author: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Sun Jan 11 02:07:26 2026 +0200

    Adding new changes to use git diff command

diff --git a/file.txt b/file.txt
index 593d87a..573004b 100644
--- a/file.txt
+++ b/file.txt
@@ -1,2 +1,4 @@
 Hello, Git
+test line
 new line
+Third line in file.txt
```
- As we can see this command give us all the data that we need about this commit
	- `Commit SHA`.
	- `Author`.
	- `Date`.
	- `Commit Message`.
	- `Changes`.
---

### Playing with logs
- We could add some options to `git log` command to do specific action like filtering, or summarizing, and so on.
- `git log -n <number>`: will return the last `<number>` of logs from the log file.
```bash
git log -n 2
commit f12c6c5b17fa2805494110d476495bcb2d034b67 (HEAD -> master)
Author: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Sun Jan 11 02:07:26 2026 +0200

    Adding new changes to use git diff command

commit aca1723f6db3f4d195c75b39cd3464f6afe587bd
Author: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Sun Jan 11 01:11:57 2026 +0200

git log -n 1
commit f12c6c5b17fa2805494110d476495bcb2d034b67 (HEAD -> master)
Author: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Sun Jan 11 02:07:26 2026 +0200

    Adding new changes to use git diff command
```

- `git log --oneline`: will summarize the each commit in single line.
```bash
git log --oneline
f12c6c5 (HEAD -> master) Adding new changes to use git diff command
aca1723 Adding new line
5fedaba initial commit
```

- `git log <fileName>`: git all the commit that related to specific file.
```bash
git log --oneline file.txt
f12c6c5 (HEAD -> master) Adding new changes to use git diff command
aca1723 Adding new line
5fedaba initial commit
```

- `git log --graph`: gill all the commits like a graph, and this is great when we have different branches
```bash
git log --graph
* commit f12c6c5b17fa2805494110d476495bcb2d034b67 (HEAD -> master)
| Author: Mahmoud <mahmoud.saadallah73@gmail.com>
| Date:   Sun Jan 11 02:07:26 2026 +0200
|
|     Adding new changes to use git diff command
|
* commit aca1723f6db3f4d195c75b39cd3464f6afe587bd
| Author: Mahmoud <mahmoud.saadallah73@gmail.com>
| Date:   Sun Jan 11 01:11:57 2026 +0200
|
|     Adding new line
|
* commit 5fedaba1cda0f9253c6695f7c72fdcdddb723603
  Author: Mahmoud <mahmoud.saadallah73@gmail.com>
  Date:   Thu Jan 8 22:45:25 2026 +0200

      initial commit
```
- or we could use `git log --oneline --decorate --graph --all`
----

### Change the commit message
- If make a commit and we want to modify the message for this commit then we could use `git commit --amend`, this command will give us chance to modify the last commit.
```bash
git commit --amend
```
- This command will open `vim` with the last commit message that we could modify.
---

### Rolling back and forward in the commits
- first let's see the logs
```bash
git log --oneline
67c561e (HEAD -> master) Ading the fourth line
e4b74a7 third line added
942d9a3 Addin the second line
e1feee0 inital commit
```
- As we can see from the previous command we have four commits in our repo and the `HEAD` if referring to the last commit with `SHA` starts with `67c561e`.
- What if I want to back to previous commit and git this commit to my working directory?
- Before we answer this question we have to know what is the `HEAD`? 
- The `HEAD` is a file that has a pointer for the `SHA` for last commit.
- So to roll back in the commits we could use `git reset HEAD~<n>` as `n` is the number of version that you want to move backward.
- So `git reset HEAD~1` will back to the `SHA` `e4b74a7` and `git rest HEAD~2` will back to `942d9a3` and so on.
```bash
git reset HEAD~1
Unstaged changes after reset:
M       file.txt
```
- Now what had happened?
- What happened here is the the `HEAD` file now is pointing on the third commit `e4b74a7` and this version of the file stored to the staging area.
- The `git` restore this commit to the staging area not the working directory directly to give us change to check changes, and to prevent loosing the un-staged changes, because may be have some changes that not saved yet to the stage or the repo so this will save it from lost.
- Now after getting this old version to the staging area we could check the difference between it and the working directory using `git diff`.
- By the way, when using the `git reset HEAD~<n>` command, we could make the older version back to the working directory direct using `--hard` option with this command.
```bash
git reset --hard HEAD~1
```
- This command will discard the changes in the working directory and automatically override the file in it, and also in the staging area.

- New after rolling back, let's check the logs
```bash
git log --oneline
e4b74a7 (HEAD -> master) third line added
942d9a3 Addin the second line
e1feee0 inital commit
```
- As we can see from the result, there are only three commits, and the fourth commit has gone, _**Does this mean we lost that commit?**_
- The Answer is **No**, but we can't see its `SHA` in the logs, but we could get all the actions that applied on the repo using other commend `git reflog`, and this command will has the `SHA` for the commit we rolled back from.
```bash
git reflog
e4b74a7 (HEAD -> master) HEAD@{0}: reset: moving to HEAD~1
67c561e HEAD@{1}: commit: Ading the fourth line
e4b74a7 (HEAD -> master) HEAD@{2}: commit: third line added
942d9a3 HEAD@{3}: commit: Addin the second line
e1feee0 HEAD@{4}: commit (initial): inital commit
```
- AS we can see in the second line of the output we have `67c561e` which is the `SHA` for the commit that removed, but in the real work it didn't remove it just moved from the commits that appear with the `git log` command.

**Now What if we want to roll forward or as git call Fast Forward?**
- To do that we could use `git reset HEAD@{<n>}` as `n` is the number for the reference to this commit, which we could find when we use `git reflog` as `67c561e HEAD@{1}: commit: Ading the fourth line`, so the reference is `1`
```bash
git reset HEAD@{1}

git log --oneline
67c561e (HEAD -> master) Ading the fourth line
e4b74a7 third line added
942d9a3 Addin the second line
e1feee0 inital commit
```
- Now we could see that the commit return back again and the `HEAD` in pointing to it now.

---
### Tags 
- In Git when we are working on a project and this project has a core changes then we call these changes new version, and using git we could put a tag for a specific commit to mark at as a specific version, and this is the power of `git tag -a <version> -m <message>` command.
```bash
git log --online
9ca8c9f (HEAD -> master) Adding the fifth line to the txt file
67c561e Ading the fourth line
e4b74a7 third line added
942d9a3 Addin the second line
e1feee0 inital commit
```
- As we can see from the output of the `git log` command, we have five commits.
- Let's consider that after the fifth commit, we want to consider that all the previous changes were a new version, which will be `v2.0`, and we want to put tag with that, so we will use.
```bash
git tag -a v2.0 -m 'New Version v2.0'
```
- There is no output for `git tag` command, but the power of this command appears when we use `git show <version>`, we this command we don't have to look for all the commits messages to get the commit for the new version, instead we could use this command to get it directly.
```bash
git show v2.0
tag v2.0
Tagger: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Thu Jan 15 22:51:05 2026 +0200

New version v2.0

commit 9ca8c9fc982eff06f24229e95052183d4d9064b1 (HEAD -> master, tag: v2.0)
Author: Mahmoud <mahmoud.saadallah73@gmail.com>
Date:   Thu Jan 15 22:47:37 2026 +0200

    Adding the fifth line to the txt file

diff --git a/file.txt b/file.txt
index f62a38b..7a0e831 100644
--- a/file.txt
+++ b/file.txt
@@ -2,3 +2,4 @@ Hello, Git
 second line added
 third line added
 Fourth line added
+Fifth line added
```
- As we can see from the output for the `git show <version>`, we get all the information that we want know about the commit for that specific version.

----
---
## Branching 
- I've started a new empty repo for this section, so we will not find the previous commits.
- Now I've made three commits and here are the commits for them
```bash
git log --oneline --decorate --graph --all
* a674d87 (HEAD -> master) Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```

![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git9.jpg)

- So here in this image, we could see that we have three commits and the `HEAD` points to the `Master` branch which points to the last `Commit`.
- So here we have only one branch, which is the `Master` branch.

To create new branch, we use `git brance <branchName>` command.
```bash
git branch testing
```
- This command has created a new branch we name `testing`

```bash
git branch my_new_branch
```

This creates a new branch called `my_new_branch`. The thing is, I rarely use this command because usually I want to create a branch and switch to it immediately. So I use this command instead:

```bash
git switch -c my_new_branch
```

The `switch` command allows you to switch branches, and the `-c` flag tells Git to create a new branch if it doesn't already exist.


To see all the branches that we have we use `git branch` command
```bash
git branch
* master
  testing
```
- As we can see in the result we have two branches `master` and `testing`, and the `*` satiric sign here refers to the `HEAD`, and in git bash we will find the `master` branch in green color which mean its the current branch, which mean if we created any new commit this commit will be in the master branch.

To change the name of the branch we could use `git branch -m <oldName> <newName>`
```bash
git branch -m testing test

git branch
* master
  test
  

git branch -m test testing

git branch
* master
  testing
  
```

let's see logs.
```bash
git log --oneline --decorate --graph --all
* a674d87 (HEAD -> master, testing) Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```
- The output of `git log` here means as the following image mean
  ![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git11.jpg)
- This mean we have three `commits`, and two `branchs`, which both of them point to the last `commit`, and the `HEAD` points to the `master` branch.


To switch form one branch to another we use 
 `git switch <branchName>` or `git checkout <branchName>`.
 ```bash
 git switch testing
 Switched to branch 'testing'
 
 $ git branch
  master
* testing
 ```
 - Here we switched to the `testing` branch and using `git branch` we could know that the current branch is `testing` and the `HEAD` is now point to this branch.
 ![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git12.jpg)

Now let's try to make changes in the file and see the new commit 
```bash
$ git log --oneline --decorate --graph --all
* 49d84bf (HEAD -> testing) First Commit in testing branch
* a674d87 (master) Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```
- After making new commit while we are in the the `master` branch now we could see from the `git log --graph` that the `HEAD` is pointing to `testing` branch which has a new commit, and the `master` branch still with the third commit.


I want to read the content of the `file.txt` that we are using here in this example, while we are in the `testing` branch.
```bash
cat file.txt
Hello, Git
Second line in file
Third line in file
Fourth line in file
```
- As we can see we have four lines in our file, and the fourth line in the line that we added while we are using the `testing` branch, and we made the last commits according to this line.

Now What will happen if we switch again to the `master` branch.
```bash
git switch master
Switched to branch 'master'

cat file.txt
Hello, Git
Second line in file
Third line in file
```
- As we can see the fourth line disappeared, Why this happen?
- This happened because this branch doesn't have any idea about the changes that happened in the other branches, so this branch still point to the third `commit` which has this version of content not the version is the `testing` branch.

Let's see logs to see where is the head pointing?
```bash
git log --oneline --decorat --graph --all
* 49d84bf (testing) First Commit in testing branch
* a674d87 (HEAD -> master) Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```
- The `HEAD` is pointing to the third commit, which the master branch also pointing.
![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git13.jpg)


**Merging**
- Now if we ok with changes that happened on any branch and we want to `merge` the changes that happened on this branch to the `master` branch we have first to switch to the `master` branch then use `git merge <branchName>`

```bash
git switch master
Switched to branch 'master'

git merge testing
Updating a674d87..49d84bf
Fast-forward
 file.txt | 1 +
 1 file changed, 1 insertion(+)
```
- Here in the output of the `git merge` command we have `fat-forward`, because the `master` branch and the `HEAD` have been moved forward to the fourth commit, which was the commit that created on the `testing` branch, but now it's not just belong to the `testing` branch, now `master` branch also could see the changes that happened on it.

```bash
cat file.txt
Hello, Git
Second line in file
Third line in file
Fourth line in file
```
- Now we have the fourth line even we are in the `master` branch, because we already merged the changes.

After merging the changes, may be we need to delete the branch, do to so we use
 `git branch -d <branchName>`
```bash
git branch -d testing
Deleted branch testing (was 49d84bf).
```

Let's check the branches and logs 
```bash
git branch 
* master

git log --oneline --decorate --graph --all
* 49d84bf (HEAD -> master) First Commit in testing branch
* a674d87 Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```
- As we can see the testing branch has been deleted, and the `HEAD` point to the `master` branch which point to the `48d84bf` commit.

**The previous scenario is not the common scenario, as we saw we after creating the testing branch, we didn't change anything in the master branch and we kept it until we finish working on the testing branch and then we merged our changes.**

**In real world we will have more than two branches and each will have some changes and most of the time these changes make conflicts and we have to learn how to solve these conflicts.**

#### Divergent history 
**![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git14.jpg)

Let's create the `testing` branch again
```bash
git branch testing

git switch testing
Switched to branch 'testing'
```
- Now we are in the `testing` branch.

Let's add new line to the `file.txt` while we are in the `testing` branch.
```bash
echo 'Fifth Line in file useing testing branch' >> file.txt

git commit -am 'Fifth line added in testing branch'
warning: in the working copy of 'file.txt', LF will be replaced by CRLF the next time Git touches it
[testing 91333b1] Fifth line added in testing branch
 1 file changed, 1 insertion(+)


git log --oneline --decorate --graph --all
* 91333b1 (HEAD -> testing) Fifth line added in testing branch
* 49d84bf (master) First Commit in testing branch
* a674d87 Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```
- Now as previous we could see from the output of the previous commands we have `HEAD` point to the `testing` branch which is point to the fifth commit and the `master` branch still point to the fourth commit.


Let's now switch back to the `master` branch
```bash
git switch master
Switched to branch 'master'

cat file.txt
Hello, Git
Second line in file
Third line in file
Fourth line in file

```
- As we know the line that we added using the `testing` branch will disappear as we have merged them yet.


Let's now add new file using the `master` branch
```bash
echo 'First line Using master branch' >>  file2.txt 

$ git add .
warning: in the working copy of 'file2.txt', LF will be replaced by CRLF the next time Git touches it


$ git commit -am 'New file added in master branch'
[master 9732058] New file added in master branch
 1 file changed, 1 insertion(+)
 create mode 100644 file2.txt

```

Now let's see the logs
```bash
git log --oneline --decorate --graph --all
* 9732058 (HEAD -> master) New file added in master branch
| * 91333b1 (testing) Fifth line added in testing branch
|/
* 49d84bf First Commit in testing branch
* a674d87 Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```
- Now the magic happen, previously everything was liner, which mean we could see the commits and the changes in the liner way, but now each branch has its own changes that the other branch can't see yet.
**![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git14.jpg)
- This is how it looks like now, and this will change the way of merging as the merging will not be direct like previous in the liner changes.

Before merging let's add another change in the testing branch.
```bash
git switch testing
Switched to branch 'testing'

echo 'sixth line using testing branch' >> file.txt

git commit -am 'Six line into the file using testing branch'
warning: in the working copy of 'file.txt', LF will be replaced by CRLF the next time Git touches it
[testing 2ce9be4] Six line into the file using testing branch
 1 file changed, 1 insertion(+)

```

Let's check logs
```bash
git log --oneline --decorate --graph --all
* 2ce9be4 (HEAD -> testing) Six line into the file using testing branch
* 91333b1 Fifth line added in testing branch
| * 9732058 (master) New file added in master branch
|/
* 49d84bf First Commit in testing branch
* a674d87 Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```
![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git20.jpg)
- This is how the log looks like 

Let's merge and see how the merge will be applied.
```bash
git switch master
Switched to branch 'master'

git merge testing
Merge made by the 'ort' strategy.
 file.txt | 2 ++
 1 file changed, 2 insertions(+)
```
- We have to know that when we try to merge the git will open `vim` to ask us to enter a message for the merging, **But Why we need a message for the merging?**
- This happen because in this situation the merging is not happen directly like before, here git will create new commit to the new merged branches, this way git ask us for the message, it's for the new commit.

Let's see logs to see the new commit
```bash
git log --oneline --decorate --graph --all
*   2ca06c1 (HEAD -> master) Merge branch 'testing'
|\
| * 2ce9be4 (testing) Six line into the file using testing branch
| * 91333b1 Fifth line added in testing branch
* | 9732058 New file added in master branch
|/
* 49d84bf First Commit in testing branch
* a674d87 Third line added to file
* 2768669 Second line added to file
* 534c4cc Inital Commit
```
![drawing](https://raw.githubusercontent.com/ahmedsami76/AraBigData/a4b6d07d50e36eded3e80966eedb903579f2e34d/Git/images/git22.jpg)
- Now as we can see from the graph that the changes that happened on the `testing` branch has been merged to the `master` class.
- But we have to notice that this merge happened using new commit `2ca06c1` and the `master` branch and `HEAD` are pointing now to that commit.
- Also we have to know that the changes that happened on the `testing` branch will be visible to the `master` branch, but the changes that happened on the `master` will not be visible to the `testing` branch 

To make sure let's see if the testing branch could see the `file2.txt` that we have created using master branch
```bash
git switch master
Switched to branch 'master'

ls -l
total 2
-rw-r--r-- 1 HP 197121 149 Jan 16 23:11 file.txt
-rw-r--r-- 1 HP 197121  32 Jan 16 23:20 file2.txt
# Now we have two files on the master branch

git switch testing
Switched to branch 'testing'

ls -l
total 1
-rw-r--r-- 1 HP 197121 149 Jan 16 23:11 file.txt
# Here we have only one file on the testing branch 
# This mean the changes that happend on the master branch would be visiable for testing branch
```



#### Merge Conflicts

A merge conflict occurs when two commits _modify the same line_ and Git can't automatically decide which change to keep and which change to discard.

Conflicting changes on two different branches is not a problem. The problem only arises when you try to _merge_ those branches. When you do, Git will detect the conflict and ask you to resolve it.

New let's do the following 
1. Get the content of the `file3.txt`.
2. Create new branch `testing`.
3. Switching to the `testing` branch, and modify the `file3.txt` by adding a new line, then commit the changes.
4. Modify the `file3.txt` using the `master` branch by adding new line, then commit the changes.
```bash
# 1
cat file3.txt
Hello, Git

# 2
git switch -c testing
Switched to a new branch 'testing'

# 3
git branch
  master
* testing
# 3
echo "Second Line using testing branch" >> file3.txt
# 3
cat file3.txt
 Hello, Git
Second Line using testing branch

# 3 
git commit -am "Second line added to file3 using testing branch"
warning: in the working copy of 'file3.txt', LF will be replaced by CRLF the next time Git touches it
[testing f26eaf2] Second line added to file3 using testing branch
 1 file changed, 1 insertion(+
 
 
#4
git switch master
Switched to branch 'master'

# 4
cat file3.txt
Hello, Git

# 4
echo "Second Line using master branch" >> file3.txt

# 4
cat file3.txt
 Hello, Git
Second Line using master branch

# 4
git commit -am "Second line added to file3 using master branch"
warning: in the working copy of 'file3.txt', LF will be replaced by CRLF the next time Git touches it
[master 38f31dd] Second line added to file3 using master branch
 1 file changed, 1 insertion(+)


```

- Now as we can see from the previous commands now we have two different versions for `file3.txt` in each branch.

If we tried to merge them now we will get a conflict.
```bash
git merge testing
Auto-merging file3.txt
CONFLICT (content): Merge conflict in file3.txt
Automatic merge failed; fix conflicts and then commit the result.

```
- This happened because in two different branches, we have changed the content of the same file with different values, and git can't merge them directly as git can't decide which version will be used and which one will be removed, so it asked to fix conflicts first then make new commit to save changes to make a correct merge.

To fix conflict we have to use any editor to open the conflicted file and fix it manually.
We will use `vim`
```bash
vim file3.txt
Hello, Git
<<<<<<< HEAD
Second Line using master branch
=======
Second Line using testing branch
>>>>>>> testing
```
- As we can see `git` already marked the lines that make the conflict to help us to fix them.
- By the way we could keep the changes from `master` branch, or keep the changes from `tesing` branch, or both of them by editing the file manually.

So here we will keep the changes from both of the branches, but we will delete the marked lines that git made.
```bash
cat file3.txt
Hello, Git
Second Line using master branch
Second Line using testing branch
```

After that we have to make new commit with the new changes to solve the conflict and make the merge happen.
```bash
git commit -am 'Fix the conflict between master and testing branch'
[master 8a4dc94] Fix the conflict between master and testing branch

```

Now let's check the logs.
```bash
git log --oneline --decorate --graph --all
*   8a4dc94 (HEAD -> master) Fix the conflict between master and testing branch
|\
| * f26eaf2 (testing) Second line added to file3 using testing branch
* | 38f31dd Second line added to file3 using master branch
|/
* 60ddfa2 first commit to file3.txt

```
- As we can see the conflict has been solved using new commit.

