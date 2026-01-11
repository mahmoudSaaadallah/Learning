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
- The `git rm --cached` will remove file from the staging area.
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