### Types of Conditions in Bash

The "condition" part of an `if` statement can be formed in several ways, each suited for different types of tests.

#### 1. `test` command or `[` (Single Bracket)

The `test` command (or its alias `[`) is a standard utility for evaluating expressions. It sets its exit status to `0` for true and `1` for false.

**Syntax:** `[ expression ]`  
**Important:** _There must be spaces around the brackets and around the operators_

**a) File Tests:**  
Used to check properties of files and directories.

| Operator | Description                                           | Example                         |
| :------- | :---------------------------------------------------- | :------------------------------ |
| `-f`     | True if file exists and is a regular file.            | `[ -f "myfile.txt" ]`           |
| `-d`     | True if file exists and is a directory.               | `[ -d "my_directory" ]`         |
| `-e`     | True if file exists (regardless of type).             | `[ -e "any_file_or_dir" ]`      |
| `-s`     | True if file exists and has a size greater than zero. | `[ -s "non_empty.txt" ]`        |
| `-r`     | True if file exists and is readable.                  | `[ -r "readable.txt" ]`         |
| `-w`     | True if file exists and is writable.                  | `[ -w "writable.txt" ]`         |
| `-x`     | True if file exists and is executable.                | `[ -x "executable_script.sh" ]` |

**b) String Tests:**  
Used to compare strings.

|Operator|Description|Example|
|:--|:--|:--|
|`=`|True if strings are equal.|`[ "$str1" = "$str2" ]`|
|`==`|Same as `=`, but specific to Bash (and some other shells).|`[ "$str1" == "$str2" ]`|
|`!=`|True if strings are not equal.|`[ "$str1" != "$str2" ]`|
|`-z`|True if string is empty (zero length).|`[ -z "$empty_str" ]`|
|`-n`|True if string is not empty (non-zero length).|`[ -n "$non_empty_str" ]`|

**c) Integer Tests:**  
Used to compare integers. **Note:** These are for integers only. For string comparisons, use `=` or `==`.

|Operator|Description|Example|
|:--|:--|:--|
|`-eq`|Equal to|`[ "$num1" -eq "$num2" ]`|
|`-ne`|Not equal to|`[ "$num1" -ne "$num2" ]`|
|`-gt`|Greater than|`[ "$num1" -gt "$num2" ]`|
|`-ge`|Greater than or equal to|`[ "$num1" -ge "$num2" ]`|
|`-lt`|Less than|`[ "$num1" -lt "$num2" ]`|
|`-le`|Less than or equal to|`[ "$num1" -le "$num2" ]`|

**Example using `[`:**

```bash
#!/bin/bash

filename="test.txt"

if [ -f "$filename" ]; then
    echo "$filename exists and is a regular file."
else
    echo "$filename does not exist or is not a regular file."
fi
```

#### 2. `[[` (Double Bracket) - Bash-specific

The `[[ ... ]]` construct is a Bash keyword (not an external command like `[`). It's generally preferred for its enhanced capabilities and safer behavior.

**Advantages of `[[ ... ]]`:**

- **No word splitting or pathname expansion:** Variables inside `[[ ... ]]` do not need to be quoted (though it's still good practice for clarity).
- **Logical operators:** Supports `&&` (AND), `||` (OR), and `!` (NOT) directly within the brackets.
- **Regular expression matching:** Uses the `=~` operator.

**Syntax:** `[[ expression ]]`  
**Important:** Still requires spaces around the brackets and operators.

**a) File, String, Integer Tests:**  
The same operators (`-f`, `-d`, `=`, `!=`, `-eq`, etc.) work as with `[`, but with the advantages mentioned above.

**b) Regular Expression Matching:**

|Operator|Description|Example|
|:--|:--|:--|
|`=~`|True if string matches the regular expression.|`[[ "$str" =~ ^[0-9]+$ ]]`|

**c) Logical Operators:**

| Operator | Description | Example                          |
| :------- | :---------- | :------------------------------- |
| `&&`     | Logical AND | `[[ -f "file1" && -f "file2" ]]` |
| `        |             | `                                |
| `!`      | Logical NOT | `[[ ! -e "non_existent.txt" ]]`  |

**Example using `[[`:**

```bash
#!/bin/bash

name="Alice"
age=30

if [[ "$name" == "Alice" && "$age" -gt 25 ]]; then
    echo "Hello, Alice! You are over 25."
fi

if [[ "hello world" =~ "world" ]]; then
    echo "String contains 'world'."
fi
```

#### 3. `((` (Double Parentheses) - Arithmetic Evaluation

The `(( ... ))` construct is used for evaluating arithmetic expressions. It's a Bash-specific feature and is excellent for integer comparisons. It returns an exit status of `0` if the expression evaluates to non-zero, and `1` if it evaluates to zero.

**Syntax:** `(( expression ))`  
**Advantages:** C-style syntax for arithmetic, no need for `-eq`, `-gt`, etc.

|Operator|Description|Example|
|:--|:--|:--|
|`==`|Equal to|`(( num1 == num2 ))`|
|`!=`|Not equal to|`(( num1 != num2 ))`|
|`>`|Greater than|`(( num1 > num2 ))`|
|`>=`|Greater than or equal to|`(( num1 >= num2 ))`|
|`<`|Less than|`(( num1 < num2 ))`|
|`<=`|Less than or equal to|`(( num1 <= num2 ))`|
|`&&`|Logical AND|`(( num1 > 0 && num2 < 10 ))`|
|`||`|

**Example using `((`:**

```bash
#!/bin/bash

score=85

if (( score >= 90 )); then
    echo "Grade: A"
elif (( score >= 80 )); then
    echo "Grade: B"
else
    echo "Grade: C or lower"
fi
```

#### 4. Exit Status of Any Command

Remember, the `if` statement simply checks the exit status of the command that follows it. This means you can use _any_ command as a condition.

**Example:**

```bash
#!/bin/bash

if grep -q "error" /var/log/syslog; then
    echo "Errors found in syslog!"
    # Further actions like sending an alert
else
    echo "No errors found in syslog."
fi

# Another example: checking if a command succeeded
if cp /path/to/source /path/to/destination; then
    echo "File copied successfully."
else
    echo "Failed to copy file."
fi
```

- `grep -q`: The `-q` (quiet) option for `grep` suppresses output and simply returns an exit status of `0` if a match is found, and `1` if not. This is perfect for `if` conditions.

---

### Best Practices and Common Pitfalls

1. **Always Quote Variables:** When using `[ ... ]` for string comparisons, always quote your variables (e.g., `"$my_var"`). This prevents issues if the variable is empty or contains spaces, which could lead to syntax errors or unexpected behavior due to word splitting. While `[[ ... ]]` is more forgiving, quoting is still good practice.
    
    - **Pitfall:** `[ $my_var = "value" ]` will fail if `my_var` is empty (expands to `[ = "value" ]`, which is a syntax error).
    - **Correct:** `[ "$my_var" = "value" ]`
2. **Spaces are Crucial:** Remember the spaces around the brackets and operators:
    
    - `[ -f "file" ]` (correct)
    - `[-f "file"]` (incorrect)
    - `[ "$var"=="value" ]` (incorrect)
3. **Prefer `[[ ... ]]` for String and File Tests:** It's generally safer, more powerful (regex, logical operators), and less prone to word splitting issues than `[ ... ]`.
    
4. **Prefer `(( ... ))` for Integer Arithmetic:** It offers a more natural C-like syntax for numerical comparisons and calculations.
    
5. **Use `grep -q` for Checking File Content:** When you just need to know if a pattern exists in a file, `grep -q` is the most efficient way to get an exit status for an `if` condition.
    
6. **Indentation:** Use consistent indentation to make your `if` statements readable and easy to follow.
    
7. **Shebang:** Always start your script with `#!/bin/bash` to ensure it's executed with Bash, which supports all the features discussed (like `[[` and `((`).


----

**Don't forget to check the [[If Condition Examples]] file**.