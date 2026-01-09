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
---

