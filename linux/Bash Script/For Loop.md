# For Loop
Let's explore the `for` loop in detail.

---

### 1. Basic Syntax and Types

Bash `for` loops come in a few main forms:

#### a) List-based `for` Loop (most common)

This form iterates over a list of words or items.

```bash
for ITEM in LIST; do
    # Commands to be executed for each ITEM
    # ...
done
```

*   **`for`**: The keyword that initiates the loop.
*   **`ITEM`**: A variable name that will hold the current item from the `LIST` in each iteration. You can choose any valid variable name.
*   **`in LIST`**: Specifies the list of items to iterate over. The `LIST` can be:
    *   A space-separated string of words.
    *   An array.
    *   The result of a command substitution (`$(command)`).
    *   File globbing patterns (`*.txt`).
*   **`do`**: Marks the beginning of the loop's body.
*   **`commands`**: One or more Bash commands that will be executed for each `ITEM`.
*   **`done`**: Marks the end of the loop's body.

**Shorthand**: If `in LIST` is omitted, the loop iterates over the positional parameters (`$@`, which expands to `$1`, `$2`, `$3`, etc.).

```bash
for ITEM; do # Equivalent to: for ITEM in "$@"; do
    # ...
done
```

#### b) C-style `for` Loop (arithmetic)

This form is similar to `for` loops found in C, Java, or JavaScript, typically used for numerical iteration.

```bash
for (( INITIALIZATION; CONDITION; INCREMENT )); do
    # Commands to be executed repeatedly
    # ...
done
```

*   **`(( ... ))`**: This is Bash's arithmetic expansion construct.
*   **`INITIALIZATION`**: An arithmetic expression executed once at the beginning of the loop (e.g., `i=0`).
*   **`CONDITION`**: An arithmetic expression evaluated before each iteration. If it evaluates to non-zero (true), the loop body executes. If it evaluates to zero (false), the loop terminates.
*   **`INCREMENT`**: An arithmetic expression executed after each iteration of the loop body (e.g., `i++`).


---

### 2. Loop Control Commands

Just like `while` loops, `for` loops also support:

*   **`break`**: Immediately exits the `for` loop (and any outer loops if given an argument, e.g., `break 2`). Execution continues with the command after `done`.
*   **`continue`**: Skips the rest of the current iteration of the loop and proceeds to the next iteration (i.e., it goes back to process the next item in the list or re-evaluate the C-style condition).
*   **`exit`**: Terminates the *entire script*, not just the loop.

---
### 3. Simple examples
#### Getting numbers from 1 to 10
```Bash
#!/bin/bash
for var in {1..10}; do
	echo $var
done
```

#### Getting odd numbers until 100
```Bash
#!/bin/bash

for var in {1..100}; do
	if (( $var % 2 != 0 )); then
		echo $var
	fi
done
```

#### Getting all the `.mp4` files in the current directory
```Bash
for file in *.mp4; do
	ls -l "$file";
done
```

#### Getting all the divisible by 5 in specific range.
```Bash
#!/bin/bash
read -p "What is the start point: " s
read -p "What is the end point: " e

for (( var=$s; var<=$e; var++ )); do
	if (( var % 5 == 0)); then
		echo $var 
	fi
done
```


### 4. Practical Examples

#### Example 1: Iterating Over a Simple List of Strings

```bash
#!/bin/bash

echo "--- Iterating over fruits ---"
for fruit in Apple Banana Cherry Date; do
    echo "I like $fruit."
done

echo ""
echo "--- Iterating over numbers ---"
for num in 10 20 30 40 50; do
    echo "Number: $((num / 2))"
done
```

**Explanation**:
*   The `for` loop processes each word (Apple, Banana, etc.) in the provided list.
*   The `fruit` variable takes on the value of each word in turn.

#### Example 2: Iterating Over Files (Globbing)

```bash
#!/bin/bash

# Create some dummy files
touch file1.txt file2.log another.txt report.pdf

echo "--- Processing .txt files ---"
for file in *.txt; do
    echo "Found text file: $file"
    # Example: You could process the file here, e.g., cat "$file"
done

echo ""
echo "--- Processing all files and directories ---"
for item in *; do
    if [ -f "$item" ]; then
        echo "File: $item"
    elif [ -d "$item" ]; then
        echo "Directory: $item"
    fi
done

# Clean up
rm file1.txt file2.log another.txt report.pdf
```

**Explanation**:
*   `*.txt` is a glob pattern that expands to all files ending with `.txt` in the current directory.
*   `*` expands to all files and directories in the current directory.
*   It's crucial to **quote** the `$file` variable (e.g., `"$file"`) to handle filenames with spaces correctly.

#### Example 3: Iterating Over Command-Line Arguments

```bash
#!/bin/bash

echo "--- Processing command-line arguments ---"
if [ "$#" -eq 0 ]; then
    echo "Usage: $0 <arg1> <arg2> ..."
    exit 1
fi

for arg in "$@"; do # "$@" expands to "$1" "$2" "$3" ...
    echo "Argument received: $arg"
done

# Example usage: ./myscript.sh hello world "with spaces"
```

**Explanation**:
*   `"$@"` is a special parameter that expands to all positional parameters (`$1`, `$2`, etc.), each as a separate word, and correctly handles arguments with spaces when quoted.
*   `$#` gives the number of arguments.

#### Example 4: C-style `for` Loop for Numerical Iteration

```bash
#!/bin/bash

echo "--- Counting from 1 to 5 ---"
for (( i=1; i<=5; i++ )); do
    echo "Current number: $i"
done

echo ""
echo "--- Counting down from 10 by 2s ---"
for (( j=10; j>=0; j-=2 )); do
    echo "Countdown: $j"
done
```

**Explanation**:
*   The `(( ... ))` syntax allows standard C-like arithmetic operations.
*   `i++` increments `i` by 1.
*   `j-=2` decrements `j` by 2.

#### Example 5: Iterating Over an Array

```bash
#!/bin/bash

declare -a COLORS=("Red" "Green" "Blue" "Yellow" "Purple")

echo "--- Iterating over an array ---"
for color in "${COLORS[@]}"; do # "${COLORS[@]}" expands to all elements of the array
    echo "Color: $color"
done

echo ""
echo "--- Iterating over array indices ---"
for index in "${!COLORS[@]}"; do # "${!COLORS[@]}" expands to all indices of the array
    echo "Index $index: ${COLORS[$index]}"
done
```

**Explanation**:
*   `declare -a COLORS=(...)` declares an array.
*   `"${COLORS[@]}"` is the correct way to expand all elements of an array, ensuring each element is treated as a separate word, even if it contains spaces.
*   `"${!COLORS[@]}"` expands to the indices (keys) of the array.

#### Example 6: Using `break` and `continue`

```bash
#!/bin/bash

echo "--- Demonstrating break and continue ---"
for i in 1 2 3 4 5 6 7 8 9 10; do
    if (( i % 2 != 0 )); then # If i is odd
        echo "Skipping odd number: $i"
        continue # Skip to the next iteration
    fi

    echo "Processing even number: $i"

    if (( i >= 6 )); then
        echo "Reached 6 or greater, breaking loop."
        break # Exit the loop entirely
    fi
done
echo "Loop finished."
```

**Explanation**:
*   `continue` causes the loop to immediately jump to the next item (`i=3`, `i=5`).
*   `break` causes the loop to terminate entirely when `i` becomes `6`.

---

### 5. Best Practices and Considerations

*   **Quoting Variables**: Always quote variables that expand to file paths or strings, especially in list-based `for` loops (e.g., `for file in *; do echo "$file"; done`). This prevents issues with filenames containing spaces or special characters.
*   **`IFS` (Internal Field Separator)**: For list-based loops, Bash splits the `LIST` based on the characters in the `IFS` variable (default is space, tab, newline). If you're iterating over items with spaces that you want to treat as a single item, you might need to temporarily change `IFS` or use arrays.
    ```bash
    # Example: Iterating over a string with custom delimiter
    OLD_IFS=$IFS
    IFS=',' # Set comma as delimiter
    for item in "apple,banana,cherry"; do
        echo "Item: $item"
    done
    IFS=$OLD_IFS # Restore original IFS
    ```
*   **Readability**: Use clear variable names and indent the loop body.
*   **Performance**: For very large lists or files, consider if other tools (like `xargs` or `find -exec`) might be more efficient, especially if you're calling external commands repeatedly.

The `for` loop is a powerful and frequently used construct in Bash scripting, essential for automating tasks that involve processing collections of data.