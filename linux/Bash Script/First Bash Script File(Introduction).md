- To make a bash script file we need to make a normal file, as we know everything in Linux is a file, so there is no meaning for the extension of the file.

- As it's most common to use the `.sh` as extension for the bash script files, so we will do the same.

**First Let's make a new file with `.sh` extension**
-  we could do that using any editor that we prefer like nano:
```Bash
nano scirpt.sh
```

- or using Vi or Vim:
```Bash
vi script.sh
```

**Changing file Permission**
- After creating and edit the file we need to change its permission to add execute permission, because all the files in Linux not executable by default.
```bash
chmod +x script.sh
```
- `chmod` to change the permissions for files.
- `+x` to add execute permission.

- Now we could add whatever script we want to run in this file

**Run the file**
- To run the file we have different ways like using the path of the file:
```Bash
/home/ubuntu/playground/script.sh
```

or

- Adding the path of our script file to the PATH variable as all the running files must be in specific paths.
- To get all the executable paths we could get them using
```Bash
echo $PATH
```

```OUTPUT
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
- So all these paths are the paths that the bash shell will search about your file in, if you enter only the name of the file without the whole path.
- This why we need to put all the path for the file to run it.
- On the other hand we could add our file path to the PATH variable like:
```Bash
 PATH=$PATH:/home/ubuntu/playground
```
- This command will add `/hom/ubuntu/playground` path to the PATH variable.

- Now after adding the file path we could run it directly using the file name.
```Bash
script.sh
```


---
## Some Important Points
- There are some important points that we have to know while working with Bash

- We have to know that not all the bash script files are named with `.sh` extension, so how we could know that the bash script files.
- There is a specific line that we have to add at the beginning of all our Bash script files.

**Adding `#!/bin/bash`**
`#!/bin/bash` this line is called a shebang also called a hash bang, and it's the very first line in our script file.
- we can't add anything before this line or even space or blank line before this command
- It is responsible for **telling the operating system which interpreter should be used to execute the script.**

#### Here's a breakdown:

- `#!`: This is the magic number (or "shebang") that the operating system's program loader looks for at the very beginning of an executable script file. 
- It indicates that the file is a script and specifies the interpreter.
- `/bin/bash`: This is the absolute path to the Bash interpreter program on most Unix-like systems (Linux, macOS, WSL).