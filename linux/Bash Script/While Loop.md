The `while` loop in Bash is a fundamental control flow statement that allows you to repeatedly execute a block of commands as long as a specified condition remains true. It's incredibly versatile and used for tasks ranging from reading files line by line to polling for external events.

Let's break it down in detail.
### 1. Basic Syntax

The general syntax of a `while` loop is:

```bash
while condition; do
    # Commands to be executed repeatedly
    # ...
done
```

- **`while`**: The keyword that initiates the loop.
- **`condition`**: This is the most crucial part. It's a command or a list of commands. The `while` loop evaluates the **exit status** of this `condition`.
    - If the `condition` command(s) exit with a status of `0` (success), the loop body (`commands`) is executed.
    - If the `condition` command(s) exit with a non-zero status (failure), the loop terminates.
- **`do`**: Marks the beginning of the loop's body.
- **`commands`**: One or more Bash commands that will be executed in each iteration as long as the `condition` is true.
- **`done`**: Marks the end of the loop's body.

---

**Check types of the conditions in Bash [[Types Of Conditions In Bash]]**

---
### 2. Loop Control Commands

Inside the loop body, you can use special commands to alter the loop's flow:

- **`break`**: Immediately exits the `while` loop (and any outer loops if given an argument, e.g., `break 2`). Execution continues with the command after `done`.
- **`continue`**: Skips the rest of the current iteration of the loop and proceeds to the next iteration (i.e., it goes back to evaluate the `condition` again).
- **`exit`**: Terminates the _entire script_, not just the loop.

---
### Practical Examples

#### Example 1: Simple Counter

```bash
#!/bin/bash

COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1)) # Increment COUNT
    sleep 0.5 # Wait for half a second
done
echo "Loop finished. Final count: $COUNT"
```

**Explanation**:

- The loop continues as long as `COUNT` is less than or equal to 5.
- `COUNT=$((COUNT + 1))` performs arithmetic increment.

----
#### Example 2: Printing all the even numbers from 1 to 100
```Bash
#!/bin/bash
counter=1
while [ $counter -le 100 ]; do
	if (( counter % 2 == 0 )); then
		echo $counter
	fi
	counter=$((counter + 1))
done
```

---
#### Example 3: Getting the summation of entered nubmer:
```Bash
#!/bin/bash

sum=0
read -p "Enter number to get summation:" number
while (( $number > 0 )); do
        sum=$(($number + $sum))
        number=$(($number - 1))
done
echo $sum 
```

---
#### example 4: check the password
```Bash
#!/bin/bash

counter=3
pass="open123"
while  (( 1 > 0 )); do
        read -p "Enter password: " password
        if [ "$pass" == "$password" ]; then
                echo "access accepted."
                exit
        else
                counter=$(($counter - 1))
        fi
        if (($counter <= 0)); then
                echo "access denied"
                exit
        fi
done
```

---

#### Example 5: Reading a File Line by Line (Most Common Use Case)

This is a very common and powerful pattern.

```bash
#!/bin/bash

# Create a dummy file for demonstration
echo "Apple" > fruits.txt
echo "Banana" >> fruits.txt
echo "Cherry" >> fruits.txt
echo "Date" >> fruits.txt

echo "--- Reading fruits.txt ---"
while IFS= read -r line; do
    echo "Processing: $line"
done < fruits.txt

rm fruits.txt # Clean up
```

**Explanation**:

- **`IFS=`**: Sets the Internal Field Separator to empty. This prevents `read` from stripping leading/trailing whitespace from lines.
- **`read -r line`**:
    - `read`: Reads a single line from standard input.
    - `-r`: Prevents backslash escapes from being interpreted (e.g., `\n` would be read as `\n` not a newline).
    - `line`: The variable where the read line will be stored.
- **`< fruits.txt`**: Redirects the content of `fruits.txt` to the standard input of the `while` loop. The `read` command inside the loop then reads from this input.
- The `read` command returns an exit status of `0` as long as it successfully reads a line. When it reaches the end of the file (EOF), it returns a non-zero status, terminating the loop.

#### Example 6: Processing Command Output

```bash
#!/bin/bash

echo "--- Listing files and directories ---"
ls -l | while IFS= read -r entry; do
    if [[ "$entry" == d* ]]; then # Check if it's a directory (starts with 'd')
        echo "Directory: $entry"
    elif [[ "$entry" == -* ]]; then # Check if it's a regular file (starts with '-')
        echo "File: $entry"
    fi
done
```

**Explanation**:

- The output of `ls -l` is piped (`|`) as standard input to the `while` loop.
- Each line of `ls -l` output is read into the `entry` variable.
- The `[[ "$entry" == d* ]]` and `[[ "$entry" == -* ]]` use glob patterns to check the first character of the line, which indicates file type in `ls -l` output.

**Important Note on Subshells with Pipes**: When you pipe output to a `while` loop (e.g., `command | while ...`), the `while` loop runs in a **subshell**. This means any variables modified _inside_ the loop will _not_ be accessible or retain their changes _outside_ the loop.

```bash
#!/bin/bash
COUNTER=0
echo "1" | while read num; do
    COUNTER=$((COUNTER + num))
    echo "Inside loop, COUNTER is: $COUNTER" # Will be 1
done
echo "Outside loop, COUNTER is: $COUNTER" # Will still be 0 (original value)
```

**Workaround for Subshell Issue**:

1. **Process Substitution (Bash 4+)**:
    
```bash
    COUNTER=0
    while read num; do
        COUNTER=$((COUNTER + num))
        echo "Inside loop, COUNTER is: $COUNTER"
    done < <(echo "1"; echo "2"; echo "3") # Output of commands is treated as a file
    echo "Outside loop, COUNTER is: $COUNTER" # Will be 6
   ```
    
2. **Redirecting a file directly (if applicable)**: As shown in Example 5.

#### Example 4: Infinite Loop with `break` (Menu-Driven Script)
- This example contains `case` you could check it from here [[Switch Case]].

```bash
#!/bin/bash

echo "--- Simple Menu ---"
while true; do
    echo "1. Say Hello"
    echo "2. Show Date"
    echo "3. Exit"
    read -p "Enter your choice: " CHOICE

    case "$CHOICE" in
        1) echo "Hello there!" ;;
        2) date ;;
        3) echo "Exiting menu. Goodbye!" ; break ;; # Exit the loop
        *) echo "Invalid choice. Please try again." ;;
    esac
    echo "" # Newline for readability
done
echo "Script finished."
```

**Explanation**:

- `while true` creates an infinite loop.
- The `case` statement handles user input.
- When `CHOICE` is `3`, `break` is executed, terminating the `while` loop.

#### Example 5: Waiting for a Resource (Polling)

```bash
#!/bin/bash

FILE_TO_WAIT_FOR="my_data_file.txt"
TIMEOUT_SECONDS=10
ELAPSED_TIME=0

echo "--- Waiting for $FILE_TO_WAIT_FOR to appear ---"
while [ ! -f "$FILE_TO_WAIT_FOR" ] && [ "$ELAPSED_TIME" -lt "$TIMEOUT_SECONDS" ]; do
    echo "Waiting... ($ELAPSED_TIME/$TIMEOUT_SECONDS seconds)"
    sleep 1
    ELAPSED_TIME=$((ELAPSED_TIME + 1))
done

if [ -f "$FILE_TO_WAIT_FOR" ]; then
    echo "$FILE_TO_WAIT_FOR found!"
    # Further processing
else
    echo "Timeout: $FILE_TO_WAIT_FOR did not appear within $TIMEOUT_SECONDS seconds."
    exit 1
fi

# Simulate file creation after a delay
( sleep 3; echo "Some data" > "$FILE_TO_WAIT_FOR" ) & # Run in background
```

**Explanation**:

- The loop continues as long as the file _does not_ exist (`! -f "$FILE_TO_WAIT_FOR"`) AND the `ELAPSED_TIME` is less than `TIMEOUT_SECONDS`.
- `sleep 1` pauses execution for 1 second in each iteration.
- The `&` at the end of the `( sleep 3; ... )` command runs it in the background, simulating an external process creating the file.

---

### 6. Best Practices and Considerations

- **Always have an exit condition**: Ensure your `while` loop has a condition that will eventually become false, or a `break` statement, to prevent infinite loops.
- **Use `IFS=` and `read -r` for file processing**: This is crucial for robustly handling file content, especially if lines might contain spaces or backslashes.
- **Be aware of subshells with pipes**: If you need to modify variables that persist outside a `while` loop fed by a pipe, use process substitution (`< <(...)`) or redirect a file directly.
- **Error Handling**: Combine `while` loops with `if` statements and `exit` codes for robust error handling.
- **Readability**: Use clear variable names and comments. Indent the loop body for better readability.