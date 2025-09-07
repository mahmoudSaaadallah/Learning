**Create a script that asks for user name then send a greeting to him.**

```Bash
#!/bin/bash
read -p "Please enter your name: " name
echo "Hello, $name"
```

---
**Create a script called s1 that calls another script s2 where:**
- a. In s1 there is a variable called x, it's value 5 
- b. Try to print the value of x in s2 by two different ways.

**s1.sh**

```Bash
#!/bin/bash

x=5
export x
./s2.sh "$x"
```

**s2.sh**

```Bash
#!/bin/bash

# The frist way using the passed argument
echo "First way: $1" 

# The second way using the export variable which will  be a global variable in this session.
echo "Second way: $x"
```

---
**3. Create a script called mycp where:**
- a. It copies a file to another
- b. It copies multiple files to a directory.

**Copy single file**

```Bash
#!/bin/bash
if (( $# == 2 )); then
	file=$1
	dest=$2
	cp "$1" "$2"
elif (( $# == 0 )); then
	read -p "Please enter the path of the file to be copied: " file
	read -p "Please enter the path of the destination file: " dest
	cp "$file" "$dest"
elif (( $# == 1 )); then
	read -p "You provided the file, now enter the path for destination: " dest
	file=$1
	cp "$1" "$dest"
else
   echo "You have to execute the script again and provide the file and the destination."
fi
```

**Coping multiple files**
```Bash
#!/bin/bash

if (( $# == 2 )); then
        file=$1
        dest=$2
        cp "$1" "$2"
elif (( $# == 0 )); then
        read -p "Please enter the path of the file to be copied: " file
        read -p "Please enter the path of the destination file: " dest
        cp "$file" "$dest"
elif (( $# == 1 )); then
        read -p "You provided the file, now enter the path for destination: " dest
        file=$1
        cp "$1" "$dest"
elif (( $# > 2 )); then
        dest=${@: -1}
        cp "${@:1:$#-1}" "$dest"
else
   echo "You have to execute the script again and provide the file and the destination."
fi
```

