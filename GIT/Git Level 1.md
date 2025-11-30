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
   