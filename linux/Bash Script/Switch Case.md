The `case` statement in Bash scripting is a powerful conditional construct that allows you to execute different blocks of code based on pattern matching against a given expression. It's often used as a more readable and efficient alternative to a long series of `if/elif/else` statements when you're checking a single variable against multiple possible values or patterns.

### Syntax

The basic syntax of a `case` statement is as follows:

```bash
case EXPRESSION in
  PATTERN1)
    COMMANDS_FOR_PATTERN1
    ;;
  PATTERN2 | PATTERN3)
    COMMANDS_FOR_PATTERN2_OR_PATTERN3
    ;;
  PATTERN4)
    COMMANDS_FOR_PATTERN4
    ;;
  *)
    DEFAULT_COMMANDS
    ;;
esac
```

Let's break down each part:

1.  **`case EXPRESSION in`**:
    *   `case`: The keyword that initiates the statement.
    *   `EXPRESSION`: This is the value (often a variable) that will be compared against each `PATTERN`.
    *   `in`: Another keyword, indicating the start of the pattern list.

2.  **`PATTERN)`**:
    *   `PATTERN`: This is a pattern (which can include wildcards) that the `EXPRESSION` is compared against.
    *   `)`: A closing parenthesis marks the end of the pattern definition for a specific case.

3.  **`COMMANDS_FOR_PATTERN`**:
    *   These are the commands that will be executed if the `EXPRESSION` matches the `PATTERN`. You can have one or more commands here.

4.  **`;;`**:
    *   This is the **case terminator**. It signifies the end of the commands for a particular pattern. Once a match is found and its commands are executed, `;;` ensures that the script exits the `case` statement and continues execution after `esac`. Without it, the script would "fall through" and attempt to match subsequent patterns, which is usually not desired.

5.  **`PATTERN1 | PATTERN2)`**:
    *   You can specify multiple patterns for a single block of commands by separating them with a pipe (`|`). If the `EXPRESSION` matches *any* of these patterns, the associated commands will be executed.

6.  **`*)`**:
    *   This is the **default case**. The asterisk (`*`) acts as a wildcard that matches any string. If the `EXPRESSION` does not match any of the preceding patterns, it will match this `*` pattern, and the `DEFAULT_COMMANDS` will be executed. It's good practice to include a default case to handle unexpected inputs.

7.  **`esac`**:
    *   This keyword (`case` spelled backward) marks the end of the entire `case` statement.

### Pattern Matching in `case`

The `case` statement uses shell globbing (wildcard) patterns, not regular expressions.

*   **`*`**: Matches any sequence of zero or more characters.
*   **`?`**: Matches any single character.
*   **`[]`**: Matches any one of the enclosed characters.
    *   `[abc]` matches 'a', 'b', or 'c'.
    *   `[0-9]` matches any digit.
    *   `[a-zA-Z]` matches any letter.
*   **`|`**: Used to separate multiple patterns for a single block of commands.

### Examples

#### 1. Simple Menu Selection

```bash
#!/bin/bash

echo "Choose an option:"
echo "1. Display current date"
echo "2. List files in current directory"
echo "3. Print working directory"
echo "q. Quit"

read -p "Enter your choice: " choice

case $choice in
  1)
    echo "Current date: $(date)"
    ;;
  2)
    echo "Files: $(ls -l)"
    ;;
  3)
    echo "Current directory: $(pwd)"
    ;;
  q|Q) # Matches 'q' or 'Q'
    echo "Exiting..."
    exit 0
    ;;
  *) # Default case for any other input
    echo "Invalid option: $choice. Please choose 1, 2, 3, or q."
    ;;
esac

echo "Script finished."
```

#### 2. Using Wildcards

```bash
#!/bin/bash

read -p "Enter a filename: " filename

case $filename in
  *.txt)
    echo "This is a text file."
    ;;
  *.sh)
    echo "This is a Bash script."
    ;;
  image_*.{jpg,png,gif}) # Matches image_something.jpg, image_something.png, etc.
    echo "This is an image file."
    ;;
  [0-9]*) # Starts with a digit
    echo "Filename starts with a number."
    ;;
  *)
    echo "Unknown file type or pattern."
    ;;
esac
```

#### 3. Checking User Input (Yes/No)

```bash
#!/bin/bash

read -p "Do you want to proceed? (yes/no): " answer

case $answer in
  yes|y|Y|Yes|YES)
    echo "Proceeding..."
    # Your commands to proceed
    ;;
  no|n|N|No|NO)
    echo "Aborting."
    # Your commands to abort
    exit 1
    ;;
  *)
    echo "Invalid input. Please answer 'yes' or 'no'."
    ;;
esac
```

### Key Advantages of `case`

*   **Readability**: For multiple conditional checks against a single variable, `case` is often much cleaner and easier to read than nested `if/elif/else` statements.
*   **Efficiency**: In some scenarios, `case` can be slightly more efficient as the expression is evaluated only once.
*   **Pattern Matching**: It excels at pattern matching using shell globs, which can be more concise for certain string comparisons than complex `if` conditions.

### When to Use `case` vs. `if/elif/else`

*   **Use `case` when**:
    *   You are comparing a single variable or expression against multiple possible values or patterns.
    *   You need to use shell globbing (wildcard) patterns for matching.
    *   You want a clear, structured way to handle menu selections or different input types.
*   **Use `if/elif/else` when**:
    *   You need to evaluate complex logical conditions (e.g., `if [[ $x -gt 10 && $y -lt 20 ]]`).
    *   You are comparing multiple different variables or expressions in a single condition.
    *   You need to perform numerical comparisons (`-eq`, `-ne`, `-gt`, etc.) or file tests (`-f`, `-d`, etc.).

In summary, the `case` statement is a fundamental and highly useful construct in Bash scripting for handling conditional logic based on pattern matching, leading to more robust and readable scripts.