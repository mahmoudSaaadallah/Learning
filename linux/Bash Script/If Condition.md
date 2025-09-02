# If Condition
The `if` condition is a fundamental control flow statement in Bash scripting, allowing your script to make decisions and execute different blocks of code based on whether a certain condition is true or false.

Let's break it down in detail.

There is another file that contains lot of examples about using `if` condition in bash script, you could check it here [[If Condition Examples]].

---

### The Basic Structure of an `if` Statement

The most common and basic structure of an `if` statement in Bash is:

```bash
if condition; then
    # Commands to execute if the condition is true (exit status 0)
fi
```

- **`if`**: The keyword that starts the conditional block.
- **`condition`**: This is the core of the `if` statement. In Bash, a "condition" is actually a **command** that is executed. The `if` statement then checks the **exit status** of that command.
    - An exit status of `0` (zero) means "success" or "true".
    - An exit status of any non-zero value (e.g., `1`, `2`, `127`) means "failure" or "false".
- **`; then`**: The semicolon is used to separate the `condition` from the `then` keyword if they are on the same line. If `then` is on the next line, the semicolon is optional. `then` marks the beginning of the commands to be executed if the condition is true.
- **`fi`**: The keyword that closes the `if` block (it's `if` spelled backward).

**Example:**

```bash
#!/bin/bash

my_variable="hello"

if [ "$my_variable" == "hello" ]; then
    echo "The variable contains 'hello'."
fi

echo "Script finished."
```

- `[ "$my_variable" == "hello" ]` the condition in bash script when surrounding with square [ ] we have to put space between the condition and the brackets or will not work.
- Bash interprets `[ ... ]` as a command, so it **requires spaces** around it like any other command:
    - `[ condition ]` is OK
    - `[$var -eq 200]` is NOT OK



----

### Common ways to write conditions in bash:

#### 1. Using `[ ]` (test command)

This is the traditional and most common way:

```Bash
if [ "$var" -eq 200 ]; then
  echo "Var is 200"
fi
```

- Requires spaces after `[` and before `]`.
- `[ ]` is actually a command (alias for `test`).

---

#### 2. Using `[[ ]]` (bash’s extended test)

More modern and safer, especially with string operations:

```Bash
if [[ $var -eq 200 ]]; then
  echo "Var is 200"
fi
```

- Supports more operators.
- Allows pattern matching and regex.
- Less need for quoting variables.

---

#### 3. Using `(( ))` for arithmetic evaluation

If you’re doing numeric comparisons:

```Bash
if (( var == 200 )); then
  echo "Var is 200"
fi
```

- This treats `var` as a number.
- No need for `$` before variables inside `(( ))`.
- Easier for math operations.

---

#### 4. Direct command execution (checks exit status)

If you want to test the success of a command:

```Bash
if grep -q "pattern" file.txt; then
  echo "Pattern found"
fi
```

---

### So to answer simply:

- For numeric and string tests, **you usually surround the condition with `[ ]` or `[[ ]]`**.
- For arithmetic, `(( ))` is cleaner.
- Just plain parentheses `( )` **do not work** as condition brackets — they create subshells.

---

### `if-else` Statement
To execute a different set of commands when the condition is false, you use the `else` block:

```bash
if condition; then
    # Commands if condition is true
else
    # Commands if condition is false
fi
```

**Example:**

```bash
#!/bin/bash

read -p "Enter a number: " num

if (( num % 2 == 0 )); then
    echo "$num is an even number."
else
    echo "$num is an odd number."
fi
```

#readCommand
`read` is a **built-in command** that allows you to **take input from the user or from a file/stream and store it in a variable**.
Read with prompt (using `-p` option) This shows the prompt on the same line as the input.

- Don't forget that it will be better to use `(( ))` to deal with numerical numbers, as using `[]` will make `>` and `<` not work, and also more other comparison operators.

---

### `if-elif-else` Statement (Multiple Conditions)

For handling multiple, mutually exclusive conditions, you use `elif` (short for "else if"):

```bash
if condition1; then
    # Commands if condition1 is true
elif condition2; then
    # Commands if condition1 is false, but condition2 is true
elif condition3; then
    # Commands if condition1 and condition2 are false, but condition3 is true
else
    # Commands if all preceding conditions are false
fi
```

The `elif` blocks are evaluated in order. As soon as one condition is true, its corresponding commands are executed, and the rest of the `elif` and `else` blocks are skipped.

**Example:**

```bash
#!/bin/bash

read -p "Enter your age: " age

if (( age < 18 )); then
    echo "You are a minor."
elif (( age >= 18 && age < 65 )); then
    echo "You are an adult."
else
    echo "You are a senior citizen."
fi
```

---

**Check types of conditions in Bash [[Types Of Conditions In Bash]]**

----

**Don't forget to check the [[If Condition Examples]] file**.
