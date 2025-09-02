# Exit code
In a Bash script, an **exit code** (also known as an **exit status** or **return code**) is a numerical value returned by a command or a script when it finishes execution. 
It's a crucial mechanism for indicating whether the command or script succeeded or failed, and if it failed, what kind of error occurred.

1. **Purpose**:
    
    - **Success/Failure Indication**: By convention, an exit code of `0` indicates successful execution. Any non-zero exit code (typically `1` to `255`) indicates an error or abnormal termination. Different non-zero codes can signify different types of errors.
    - **Conditional Execution**: Exit codes are frequently used in `if` statements, `while` loops, or with logical operators (`&&` for "AND", `||` for "OR") to control the flow of a script based on the success or failure of previous commands.
    - **Script Chaining**: When one script calls another, the calling script can check the exit code of the called script to react accordingly.
    
2. **How to Get the Exit Code**:
    
    - The special variable `$?` holds the exit code of the _last executed foreground command_.
    
```bash
    ls /nonexistent_directory
    echo "Exit code of ls: $?" # Will likely be 1 or 2 (non-zero)
    
    ls /
    echo "Exit code of ls: $?" # Will be 0
    ```
    
3. **How to Set the Exit Code in a Script**:
    
    - **Implicitly**: If your script doesn't explicitly use the `exit` command, its exit code will be the exit code of the _last command_ executed in the script.
    - **Explicitly**: You can use the `exit` command followed by a number to set the script's exit code directly.
    
```bash
    #!/bin/bash
    
    # Example 1: Script exits with 0 (success)
    echo "This script will succeed."
    exit 0
    
    # Example 2: Script exits with 1 (general error)
    # if [ ! -f "myfile.txt" ]; then
    #   echo "Error: myfile.txt not found!" >&2 # Output error to stderr
    #   exit 1
    # fi
    
    # Example 3: Script exits with a specific error code
    # if [ -z "$1" ]; then
    #   echo "Usage: $0 <argument>" >&2
    #   exit 2 # Error code 2 for "missing argument"
    # fi
    ```
    
4. **Common Exit Code Conventions**:
    
    - `0`: Success.
    - `1`: General error, catchall for miscellaneous errors.
    - `2`: Misuse of shell builtins or missing keyword, or sometimes for missing arguments.
    - `126`: Command invoked cannot execute (e.g., permissions problem).
    - `127`: Command not found.
    - `128 + N`: Fatal error signal `N` (e.g., `130` for `Ctrl+C` which sends `SIGINT`, signal `2`).

Understanding and utilizing exit codes is fundamental for writing robust and reliable Bash scripts.

----
### Examples for using exit code `$?`
- Using exit code with script file which is responsible for install `htop` package to our system.

```Bash
#!/bin/bash

package=htop
sudo apt install $package

echo "The exit code for package installation is: $?"
```
- In the previous example we used `$?` exit code to check what happened in the last command, as the exit code store the status of the last command and we could retrieve it. 
```Bash
ubuntu@ubuntu2004:~/playground$ ./test.sh
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  htop
0 upgraded, 1 newly installed, 0 to remove and 25 not upgraded.
Need to get 80.5 kB of archives.
After this operation, 225 kB of additional disk space will be used.
Err:1 http://us.archive.ubuntu.com/ubuntu focal/main amd64 htop amd64 2.2.0-2build1
  Temporary failure resolving 'us.archive.ubuntu.com'
E: Failed to fetch http://us.archive.ubuntu.com/ubuntu/pool/main/h/htop/htop_2.2.0-2build1_amd64.deb  Temporary failure resolving 'us.archive.ubuntu.com'
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
The status of installing htop package is: 100
```
- here when we execute our script it gives us the that output which as we can see in the last line the status is: 100. This mean there was an error with installing and the installing didn't work correctly.
- The installing didn't work because my vim doesn't apply to install any package.

- _We could modify our script to make it automatically send mail to the administrator to fix the problem if the installing doesn't happen. _

```Bash
#!/bin/bash

package=htop
sudo apt install $package

if(( $? != 0 )); then
	mail -s "Installing $package package faild try to install it again." ubuntu
fi
```

#### Using `if` Statements to Handle Command Outcomes

This is a more explicit way to react to command success or failure.

```bash
#!/bin/bash

FILE="important_data.txt"

echo "--- Checking for file existence ---"
if [ -f "$FILE" ]; then
    echo "$FILE exists. Proceeding..."
    # Further actions if file exists
    cat "$FILE"
else
    echo "$FILE does not exist. Creating it..."
    echo "This is some important data." > "$FILE"
    if [ $? -eq 0 ]; then # Check if the 'echo > file' command succeeded
        echo "$FILE created successfully."
    else
        echo "Error creating $FILE."
        exit 1 # Exit script with an error code
    fi
fi

echo ""
echo "--- Running a command and checking its exit code ---"
grep "data" "$FILE"
if [ $? -eq 0 ]; then
    echo "Found 'data' in $FILE."
else
    echo "Did not find 'data' in $FILE."
    exit 1
fi

# Clean up
rm "$FILE"
rmdir my_new_directory 2>/dev/null # Remove directory, suppress error if it doesn't exist
```

**Explanation**:

- `[ -f "$FILE" ]` is a test command. Its exit code is `0` if the file exists, `1` otherwise. The `if` statement evaluates this exit code.
- After `echo "This is some important data." > "$FILE"`, we check `$?` to confirm the file creation was successful.
- After `grep`, we check `$?`. `grep` returns `0` if it finds a match, and `1` if it doesn't find a match (or `2` for other errors).


#### Setting Custom Exit Codes in Your Script

You can explicitly control the exit code of your own script using the `exit` command.

```bash
#!/bin/bash

# Function to perform a task
perform_task() {
    local input_arg="$1"
    if [ -z "$input_arg" ]; then
        echo "Error: No argument provided to perform_task." >&2
        return 1 # Return 1 to indicate an error within the function
    fi
    echo "Task performed with argument: $input_arg"
    return 0 # Return 0 to indicate success
}

echo "--- Script Start ---"

# Scenario 1: Call function with missing argument
perform_task
if [ $? -ne 0 ]; then
    echo "Script exiting due to missing argument error from function."
    exit 101 # Custom error code for "function argument missing"
fi

echo ""

# Scenario 2: Call function with valid argument
perform_task "some_value"
if [ $? -ne 0 ]; then
    echo "Script exiting due to an error from function."
    exit 102 # Custom error code for "general function error"
fi

echo ""

# Scenario 3: Check for a required environment variable
if [ -z "$MY_API_KEY" ]; then
    echo "Error: MY_API_KEY environment variable is not set." >&2
    exit 103 # Custom error code for "missing environment variable"
fi

echo "MY_API_KEY is set. Script finished successfully."
exit 0 # Explicitly exit with 0 for success
```

**Explanation**:

- `return N` is used within functions to set the function's exit code.
- `exit N` is used in the main script body to set the script's overall exit code and terminate the script.
- We use different non-zero exit codes (`101`, `102`, `103`) to differentiate between various types of errors, which can be helpful for debugging or for other scripts that call this one.