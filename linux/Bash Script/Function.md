# Function
Functions in Bash are a way to group a set of commands together under a single name. This allows you to reuse code, make your scripts more organized, readable, and maintainable.

Here's a detailed breakdown of Bash functions:

### 1. Why Use Functions?

*   **Modularity**: Break down complex scripts into smaller, manageable units.
*   **Reusability**: Write a piece of code once and call it multiple times.
*   **Readability**: Improve the clarity of your script by giving descriptive names to blocks of code.
*   **Maintainability**: Easier to debug and modify specific parts of your script.

### 2. Function Syntax

There are two common ways to define a function in Bash:

**Method 1 (Preferred and more common):**

```bash
function_name () {
  # Commands to be executed
  # ...
}
```

**Method 2 (POSIX standard, works in most shells):**

```bash
function function_name {
  # Commands to be executed
  # ...
}
```

**Key points:**
*   The `()` after the function name is crucial in Method 1.
*   The `function` keyword is optional in Method 1 but required in Method 2.
*   The commands within the function must be enclosed in curly braces `{}`.
*   The opening brace `{` must be on the same line as the function name (or `function_name ()`) or separated by a semicolon.
*   The closing brace `}` must be on its own line or followed by a semicolon.

**Example:**

```bash
# Method 1
my_first_function () {
  echo "Hello from my first function!"
}

# Method 2
function my_second_function {
  echo "This is my second function."
}
```

### 3. Calling Functions

To execute a function, simply type its name:

```bash
my_first_function
my_second_function
```

Functions must be defined before they are called in the script.

### 4. Passing Arguments to Functions

Arguments are passed to a function just like they are passed to a script: by listing them after the function name when calling it. Inside the function, these arguments are accessed using special variables:

*   `$1`, `$2`, `$3`, ...: Positional parameters representing the first, second, third argument, and so on.
*   `$@`: All arguments as separate strings. Useful when iterating over arguments.
*   `$*`: All arguments as a single string.
*   `$#`: The number of arguments passed to the function.
*   `$0`: Inside a function, `$0` still refers to the name of the script itself, not the function name.

**Example:**

```bash
greet_user () {
  echo "Hello, $1!"
  echo "You provided $# arguments."
  echo "All arguments: $@"
}

greet_user "Alice"
greet_user "Bob" "Smith" "Developer"
```

**Output:**

```
Hello, Alice!
You provided 1 arguments.
All arguments: Alice
Hello, Bob!
You provided 3 arguments.
All arguments: Bob Smith Developer
```

### 5. Return Values

Functions in Bash can "return" values in two primary ways:

#### a. Exit Status (using `return`)

The `return` command is used to exit a function and set its exit status. The exit status is an integer between 0 and 255.
*   `0` typically indicates success.
*   Any non-zero value indicates an error.

The exit status can be checked using the special variable `$?` immediately after the function call.

```bash
check_number () {
  local num=$1
  if [[ $num -gt 10 ]]; then
    echo "$num is greater than 10."
    return 0 # Success
  else
    echo "$num is not greater than 10."
    return 1 # Failure
  fi
}

check_number 15
if [[ $? -eq 0 ]]; then
  echo "Function succeeded."
else
  echo "Function failed."
fi

check_number 5
if [[ $? -eq 0 ]]; then
  echo "Function succeeded."
else
  echo "Function failed."
fi
```

**Output:**

```
15 is greater than 10.
Function succeeded.
5 is not greater than 10.
Function failed.
```

#### b. Output (using `echo` or `printf`)

If you want a function to return a string or a value that can be used in a variable, you typically `echo` or `printf` the value, and then capture it using command substitution (`$()` or backticks `` ` ``).

```bash
get_full_name () {
  local first_name=$1
  local last_name=$2
  echo "$first_name $last_name"
}

name=$(get_full_name "John" "Doe")
echo "Full name: $name"

# You can also pipe the output
get_full_name "Jane" "Smith" | tr 'a-z' 'A-Z'
```

**Output:**

```
Full name: John Doe
JANE SMITH
```

### 6. Variable Scope: `local` vs. Global

By default, any variable defined inside a Bash function is a **_global variable_**. This means it can be accessed and modified from anywhere in the script, including outside the function. This can lead to unexpected side effects and bugs.

To prevent this, use the `local` keyword to declare variables within a function. **Local variables** are only accessible within the function where they are defined.

```bash
global_var="I am global"

my_function () {
  local local_var="I am local to my_function"
  global_var="I was changed inside my_function" # Modifies the global variable
  echo "Inside function: local_var = $local_var"
  echo "Inside function: global_var = $global_var"
}

echo "Before function call: global_var = $global_var"
# echo "Before function call: local_var = $local_var" # This would cause an error

my_function

echo "After function call: global_var = $global_var"
echo "After function call: local_var = $local_var" # This will be empty or error, as local_var is out of scope
```

**Output:**

```
Before function call: global_var = I am global
Inside function: local_var = I am local to my_function
Inside function: global_var = I was changed inside my_function
After function call: global_var = I was changed inside my_function
After function call: local_var =
```

**Best Practice**: Always use `local` for variables declared within functions unless you specifically intend for them to be global.

### 7. Example: A Simple Utility Function

```bash
# Function to log messages with a timestamp
log_message () {
  local message=$1
  echo "$(date '+%Y-%m-%d %H:%M:%S') - $message"
}

# Function to check if a file exists
check_file_exists () {
  local file_path=$1
  if [[ -f "$file_path" ]]; then
    log_message "File '$file_path' exists."
    return 0
  else
    log_message "Error: File '$file_path' does not exist."
    return 1
  fi
}

# Main script logic
log_message "Starting script..."

check_file_exists "/etc/passwd"
if [[ $? -eq 0 ]]; then
  echo "Proceeding with operations on /etc/passwd..."
else
  echo "Aborting due to missing file."
fi

check_file_exists "/non/existent/file.txt"
if [[ $? -eq 0 ]]; then
  echo "Proceeding with operations on /non/existent/file.txt..."
else
  echo "Aborting due to missing file."
fi

log_message "Script finished."
```

This example demonstrates using `local` variables, passing arguments, returning exit statuses, and using `echo` for output, all within a structured script.