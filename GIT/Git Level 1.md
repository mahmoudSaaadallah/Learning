### To check the status of the files in our repo 
```bash
git status
```

- The file could be in one  of three status:
	-  `untracked`: Not being tracked by Git.
	- `staged`: Marked for inclusion in the next commit.
	- `committed`: Saved to the repository's history.

---
### To Add the file to the stage (add it to the "index")
```bash
git add <fileName | filePaht | pattern>
```

---
### Commit
- A commit is a snapshot of the repository at a given point in time. It's a way to save the state of the repository, and it's how Git keeps track of changes to the project. A commit comes with a message that describes the changes made in the commit.

```bash
git commit -m "your message here"
```


### Track logs
- A Git repo is a (potentially very long) list of commits, where each commit represents the _full state of the repository_ at a given point in time.

- The [git log](https://git-scm.com/docs/git-log) command shows a history of the commits in a repository. This is what makes Git a version control system. You can see:
	- Who made a commit
	- When the commit was made
	- What was changed
##### Commit Hash
- Each commit has a unique identifier called a "*commit hash*". This is a long string of characters that uniquely identifies the commit.
> 5ba786fcc93e8092831c01e71444b9baa2228a4f

- we could refer to any commit or change within Git by using the first 7 characters of its hash `5ba786f`.
- To display all the logs we use
```bash
git log
```
- `git log` will return all the logs, but we could specify the number of logs using
   `-n <number_of_logs>` option.

---
### .git/objects
**It's Just Files All the Way Down**

All the data in a Git repository is stored directly in the (hidden) `.git` directory. That includes all the commits, branches, tags, and other objects we'll learn about later.

Git is made up of [objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) that are stored in the `.git/objects` directory. A commit is just a type of object.

-  Git has a built-in plumbing command, [cat-file](https://git-scm.com/docs/git-cat-file), that allows us to see the contents of a commit without needing to futz around with the object files directly.
```bash
git cat-file -p <commitHash>
```

example
```bash
$ git log -n 10
> commit d57fc01b6099fd73b9f361b6a2a45de885fe0609 (HEAD -> master)
	 Author: mahmoudsaaadallah <mahmoud.saadallah73@gmail.com>
	 Date:   Sun Nov 30 09:31:44 2025 +0200
	A: add contents.md
	
$ git cat-file -p d57fc01b6099fd73b9f361b6a2a45de885fe0609
> tree 5b21d4f16a4b07a6cde5a3242187f6a5a68b060f
	author mahmoudsaaadallah <mahmoud.saadallah73@gmail.com> 1764487904 +0200
	committer mahmoudsaaadallah <mahmoud.saadallah73@gmail.com> 1764487904 +0200
	A: add contents.md
```

---
### Trees and Blobs

Now that we understand some of our plumbing equipment, let's get into the pipes. Here are some terms to know:

- `tree`: git's way of storing a directory
- `blob`: git's way of storing a file

Here's what I got when I inspected my last commit:

```bash
$ git cat-file -p d57fc01b6099fd73b9f361b6a2a45de885fe0609

tree 5b21d4f16a4b07a6cde5a3242187f6a5a68b060f
author mahmoudsaaadallah <mahmoud.saadallah73@gmail.com> 1764487904 +0200
committer mahmoudsaaadallah <mahmoud.saadallah73@gmail.com> 1764487904 +0200
A: add contents.md
```

Notice that we can see:

- The `tree` object
- The `author`
- The `committer`
- The commit message

However, we _cannot_ see the contents of the `contents.md` file itself! That's because the `blob` object stores it.

### cat-file 
Use `git cat-file -p` again, but this time with the hash of the `tree` object instead of the commit hash. You should see a `blob` object with _its_ own hash

```bash
# Using commit hash
$ git cat-file -p d57fc01b6099fd73b9f361b6a2a45de885fe0609
tree 5b21d4f16a4b07a6cde5a3242187f6a5a68b060f
author mahmoudsaaadallah <mahmoud.saadallah73@gmail.com> 1764487904 +0200
committer mahmoudsaaadallah <mahmoud.saadallah73@gmail.com> 1764487904 +0200
A: add contents.md

# Using tree hash
$ git cat-file -p 5b21d4f16a4b07a6cde5a3242187f6a5a68b060f
100644 blob ef7e93fc61a91deecaa551c4707e4c3049af42c9    contents.md # blob hash

# Use `cat-file` _again_, using the `blob` object's hash to view its contents.
$ git cat-file -p ef7e93fc61a91deecaa551c4707e4c3049af42c9
This is the first line in the txt file.
```

**To Illustrate**
we have three hashes now:
	1. _Commit Hash_.
	2. _Tree Hash_.
	3. _Blob Hash_.


- You have to know that the previous example has the (tree, Author, Committer, and Commit) only because this is the first commit in this repo.
- But if we added another commit to the same repo we will find new field appear which is (parent).

```bash
$ git log
commit 9f4d366ae5d495a55b2bed2280fcd5e2b56b763c (HEAD -> master)
Author: mahmoudsaaadallah <mahmoud.saadallah73@gmail.com>
Date:   Wed Dec 3 19:37:47 2025 +0200

    B: Adding titles.md file

commit d57fc01b6099fd73b9f361b6a2a45de885fe0609
Author: mahmoudsaaadallah <mahmoud.saadallah73@gmail.com>
Date:   Sun Nov 30 09:31:44 2025 +0200

    A: add contents.md
    
$ git cat-file -p 9f4d366ae5d495a55b2bed2280fcd5e2b56b763c
tree 3843e9a820792877796d0ad3dfa5d2860155cab7
parent d57fc01b6099fd73b9f361b6a2a45de885fe0609  # this is the new field because this commit has a parent commit.
author mahmoudsaaadallah <mahmoud.saadallah73@gmail.com> 1764783467 +0200
committer mahmoudsaaadallah <mahmoud.saadallah73@gmail.com> 1764783467 +0200

B: Adding titles.md file
```

### Storing Data
- When commit changes, Git stores an entire _snapshot_ of files on a per_commit level, not only the changes.
- While it's true that Git stores entire snapshots, it _does_ have some performance optimizations so that your `.git` directory doesn't get too unbearably large.
	- Git [compresses and packs](https://git-scm.com/book/en/v2/Git-Internals-Packfiles) files to store them more efficiently.
	- Git deduplicates files that are the same across different commits. If a file doesn't change between commits, Git will only store it once
	
