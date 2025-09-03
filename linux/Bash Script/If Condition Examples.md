Okay, let's dive into a lot of examples and problems using `if` conditions in Bash scripts. We'll cover various types of conditions and scenarios.

---

### General Structure Reminder

```bash
if condition; then
    # Code if condition is true
fi

if condition; then
    # Code if condition is true
else
    # Code if condition is false
fi

if condition1; then
    # Code if condition1 is true
elif condition2; then
    # Code if condition1 is false, but condition2 is true
else
    # Code if all preceding conditions are false
fi
```

---

### Examples with `[ ... ]` (Single Bracket - `test` command)

**1. Check if a file exists:**

```bash
#!/bin/bash

FILE="my_document.txt"

if [ -f "$FILE" ]; then
    echo "$FILE exists and is a regular file."
else
    echo "$FILE does not exist or is not a regular file."
fi

# To test:
# touch my_document.txt
# ./script.sh
# rm my_document.txt
# ./script.sh
```

**2. Check if a directory exists:**

```bash
#!/bin/bash

DIR="my_project_folder"

if [ -d "$DIR" ]; then
    echo "$DIR exists and is a directory."
else
    echo "$DIR does not exist or is not a directory."
fi

# To test:
# mkdir my_project_folder
# ./script.sh
# rmdir my_project_folder
# ./script.sh
```

**3. Check if a string is empty:**

```bash
#!/bin/bash

read -p "Enter something: " user_input

if [ -z "$user_input" ]; then
    echo "You didn't enter anything!"
else
    echo "You entered: '$user_input'"
fi
```

**4. Compare two strings:**

```bash
#!/bin/bash

PASSWORD="secret"
read -s -p "Enter password: " entered_password
echo # For a newline after password input

if [ "$entered_password" = "$PASSWORD" ]; then
    echo "Access granted!"
else
    echo "Access denied. Incorrect password."
fi
```
 
**5. Compare two numbers (integers):**

```bash
#!/bin/bash

num1=10
num2=20

if [ "$num1" -lt "$num2" ]; then
    echo "$num1 is less than $num2."
elif [ "$num1" -eq "$num2" ]; then
    echo "$num1 is equal to $num2."
else
    echo "$num1 is greater than $num2."
fi
```

---

### Examples with `[[ ... ]]` (Double Bracket - Bash-specific)

**1. String comparison with `==` (Bash specific, often preferred):**

```bash
#!/bin/bash

name="Alice"

if [[ "$name" == "Alice" ]]; then
    echo "Hello, Alice!"
else
    echo "You are not Alice."
fi
```

**2. Logical AND (`&&`) for multiple conditions:**

```bash
#!/bin/bash

read -p "Enter username: " username
read -s -p "Enter password: " password
echo

if [[ "$username" == "admin" && "$password" == "secure123" ]]; then
    echo "Login successful!"
else
    echo "Invalid credentials."
fi
```

**3. Logical OR (`||`) for multiple conditions:**

```bash
#!/bin/bash

read -p "Are you a student or a teacher? (s/t): " role

if [[ "$role" == "s" || "$role" == "S" ]]; then
    echo "Welcome, student!"
elif [[ "$role" == "t" || "$role" == "T" ]]; then
    echo "Welcome, teacher!"
else
    echo "Invalid role entered."
fi
```

**4. Regular expression matching (`=~`):**

```bash
#!/bin/bash

read -p "Enter a filename (e.g., document.txt): " filename

# Check if the filename ends with .txt or .pdf
if [[ "$filename" =~ \.(txt|pdf)$ ]]; then
    echo "This looks like a text or PDF file."
else
    echo "This is not a .txt or .pdf file."
fi
```

**5. Combining file tests and logical operators:**

```bash
#!/bin/bash

FILE="report.log"

if [[ -f "$FILE" && -r "$FILE" && -s "$FILE" ]]; then
    echo "$FILE exists, is readable, and is not empty."
else
    echo "$FILE is missing, not readable, or empty."
fi
```

---

### Examples with `(( ... ))` (Double Parentheses - Arithmetic Evaluation)

**1. Simple arithmetic comparison:**

```bash
#!/bin/bash

read -p "Enter a number: " num

if (( num > 0 )); then
    echo "$num is positive."
elif (( num < 0 )); then
    echo "$num is negative."
else
    echo "$num is zero."
fi
```

**2. Checking divisibility (modulo operator `%`):**

```bash
#!/bin/bash

read -p "Enter a number: " number

if (( number % 2 == 0 )); then
    echo "$number is an even number."
else
    echo "$number is an odd number."
fi
```

**3. Complex arithmetic conditions with logical operators:**

```bash
#!/bin/bash

read -p "Enter a score (0-100): " score

if (( score >= 90 && score <= 100 )); then
    echo "Grade: A"
elif (( score >= 80 && score < 90 )); then
    echo "Grade: B"
elif (( score >= 70 && score < 80 )); then
    echo "Grade: C"
elif (( score >= 60 && score < 70 )); then
    echo "Grade: D"
elif (( score >= 0 && score < 60 )); then
    echo "Grade: F"
else
    echo "Invalid score entered."
fi
```

---

### Examples with Command Exit Status

**1. Check if a command succeeded:**

```bash
#!/bin/bash

# Try to copy a file that might not exist
cp non_existent_file.txt /tmp/copied_file.txt

if [ $? -eq 0 ]; then # $? holds the exit status of the last command
    echo "File copied successfully."
else
    echo "File copy failed."
fi

# Or more idiomatically:
if cp existing_file.txt /tmp/another_copy.txt; then
    echo "File copied successfully (idiomatic)."
else
    echo "File copy failed (idiomatic)."
fi
```

**2. Check if a user exists using `id` command:**

```bash
#!/bin/bash

read -p "Enter a username to check: " username

if id "$username" &>/dev/null; then # Redirect stdout/stderr to /dev/null
    echo "User '$username' exists."
else
    echo "User '$username' does not exist."
fi
```

**3. Check for a pattern in a file using `grep -q`:**

```bash
#!/bin/bash

LOG_FILE="/var/log/syslog" # Or any log file you have
SEARCH_TERM="error"

if grep -q "$SEARCH_TERM" "$LOG_FILE"; then
    echo "The term '$SEARCH_TERM' was found in $LOG_FILE."
else
    echo "The term '$SEARCH_TERM' was NOT found in $LOG_FILE."
fi
```

---

### Problems and Solutions

**Problem 1: Disk Space Alert**
Write a script that checks if the root filesystem (`/`) has less than 10% free space. If it does, print an alert.

**Solution 1:**

```bash
#!/bin/bash

# Get the percentage of used space on the root filesystem
# df -h / | awk 'NR==2 {print $5}' extracts the 5th column (Use%) from the 2nd line
# sed 's/%//' removes the '%' sign
USED_PERCENT=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')
FREE_PERCENT=$((100 - USED_PERCENT))

if (( FREE_PERCENT < 10 )); then
    echo "ALERT: Low disk space on /! Only $FREE_PERCENT% free."
else
    echo "Disk space on / is healthy ($FREE_PERCENT% free)."
fi
```

**Problem 2: Validate IP Address Format**
Write a script that takes an input string and checks if it *looks like* a simple IPv4 address (e.g., `192.168.1.1`). It doesn't need to validate the number ranges, just the `X.X.X.X` format.

**Solution 2:**

```bash
#!/bin/bash

read -p "Enter an IP address: " ip_address

# Regex to match four numbers separated by dots
# \b for word boundary, [0-9]+ for one or more digits, \. for literal dot
if [[ "$ip_address" =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
    echo "'$ip_address' appears to be a valid IPv4 format."
else
    echo "'$ip_address' does not match IPv4 format."
fi

# Test cases:
# 192.168.1.1 -> Valid
# 10.0.0.255 -> Valid
# 1.2.3 -> Invalid
# 1.2.3.4.5 -> Invalid
# abc.def.ghi.jkl -> Invalid
```

**Problem 3: Check User Permissions for a File**
Write a script that takes a filename as an argument and checks if the current user has read, write, and execute permissions for it.

**Solution 3:**

```bash
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

FILE="$1"

if [ ! -e "$FILE" ]; then
    echo "Error: File '$FILE' does not exist."
    exit 1
fi

echo "Checking permissions for '$FILE':"

if [ -r "$FILE" ]; then
    echo "  - Read permission: YES"
else
    echo "  - Read permission: NO"
fi

if [ -w "$FILE" ]; then
    echo "  - Write permission: YES"
else
    echo "  - Write permission: NO"
fi

if [ -x "$FILE" ]; then
    echo "  - Execute permission: YES"
else
    echo "  - Execute permission: NO"
fi

# To test:
# touch test_file.txt
# chmod 644 test_file.txt
# ./script.sh test_file.txt
# chmod 700 test_script.sh
# ./script.sh test_script.sh
```

**Problem 4: Check if a process is running**
Write a script that checks if a specific process (e.g., `nginx`, `apache2`, `sshd`) is running.

**Solution 4:**

```bash
#!/bin/bash

read -p "Enter process name to check (e.g., nginx, sshd): " process_name

# pgrep -x checks for exact process name
if pgrep -x "$process_name" &>/dev/null; then
    echo "Process '$process_name' is running."
else
    echo "Process '$process_name' is NOT running."
fi

# To test:
# Try 'sshd' or 'nginx' if you have them running.
# Try a non-existent process like 'foobar'.
```

---

These examples cover a wide range of `if` condition uses in Bash, from simple checks to more complex logical evaluations and command exit status handling. Remember to always use the shebang, quote your variables, and use appropriate test constructs (`[`, `[[`, `((`) for the task at hand.


---
### Example for script that could be used to manage packages in our system.
- The following script could be used to manage the packages in our system like(install, update, and remove).

```Bash
#!/bin/bash

if [ $# -lt 2 ]; then
	echo "You have to specify the operation and the package name."
	echo "For example install htop      OR      remove Vlc"
	exit 1
else
	sudo apt "$1" "$2"
fi
```
- To use this script we have to pass two arguments to it when executing like

```Bash
./pack.sh install vlc
```
- This call will make this script install vlc player in your machine.