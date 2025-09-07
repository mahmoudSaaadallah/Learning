# awk Command
- The `awk` command is a powerful text processing tool in Linux/Unix, often referred to as a "pattern scanning and processing language."
- It's designed for extracting and manipulating data from text files or standard input, especially when the data is structured into records (lines) and fields (words).

- `awk` reads input line by line, and for each line, it attempts to match a specified pattern. If a line matches the pattern, `awk` performs a corresponding action. If no pattern is specified, the action is performed on every line.

### Basic Structure of `awk`

The general syntax for `awk` is:  
```bash
awk 'pattern { action }' [filename...]
```

- **`pattern`**: A regular expression or a conditional expression that determines which lines the action should apply to. If omitted, the action applies to all lines.
- **`action`**: A series of commands enclosed in curly braces `{}` that `awk` executes when a line matches the pattern. If omitted, `awk` prints the entire line.
- **`filename...`**: One or more input files. If no files are specified, `awk` reads from standard input.


### Key Concepts

1. **Records and Fields:**
    - `awk` processes input one **record** at a time. By default, a record is a single line.
    - Each record is divided into **fields**. By default, fields are separated by whitespace (spaces, tabs).
    - `$0` refers to the entire current record (the whole line).
    - `$1`, `$2`, `$3`, etc., refer to the first, second, third, and subsequent fields of the current record.

2. **Field Separator (`FS`)**:  
    You can change the field separator using the `-F` option or by setting the `FS` built-in variable.

3. **Built-in Variables**:  
    `awk` provides several useful built-in variables:
    
    - `NR`: Number of the current record (line number).
    - `NF`: Number of fields in the current record.
    - `RS`: Record Separator (default is newline).
    - `FS`: Field Separator (default is whitespace).
    - `OFS`: Output Field Separator (default is space).
    - `ORS`: Output Record Separator (default is newline).
    - `FILENAME`: Name of the current input file.

4. **Special Patterns (`BEGIN` and `END`)**:
    
    - **`BEGIN { action }`**: The action specified here is executed _once_ before `awk` starts processing any input lines. This is useful for setting variables, printing headers, etc.
    - **`END { action }`**: The action specified here is executed _once_ after `awk` has finished processing all input lines. Useful for printing summaries, totals, etc.


----
### Examples (Using Shell Multi-line Input and Script Files)

- **Getting all the content of the passwd file:**
```bash
awk '{print $0}' /etc/passwd
```

---> The output will be exactly like the `cat` command.
```bash
cat /etc/passwd
```

- As we know that the `/etc/passwd` file contains the users for our system and consist of seven fields (loginName, password, UID, GID, Comment, Homer Dir, Shell).
- Now if we want to get the userName we could use `awk` with the `$1` to get the first field.
```bash
# If we use awk '{print $1}' /etc/passwd it will return wrond answer why??
# Because the default seperator is blank(space)(tab), but in the /etc/passwd file the seperator is clone(:).
# So we have to change the default seperator using option -F
awk -F: '{print $1}' /etc/passwd
```


---> option `-F:` means change the separator to be clone.

----

Let's create a sample data file named `data.txt` for our examples:

```bash
Name,Age,City,Occupation,Salary
Alice,30,New York,Engineer,75000
Bob,24,London,Designer,60000
Charlie,35,Paris,Doctor,90000
David,29,New York,Engineer,80000
Eve,42,London,Manager,110000
Frank,28,Berlin,Developer,70000
```

---

#### Example 1: Print Specific Fields (Default Field Separator)

Let's say we have a file `names.txt` with names and ages separated by spaces:

```
John 30
Jane 25
Mike 40
```

**Goal:** Print only the names.
**Shell Command:**

```bash
awk '{ print $1 }' names.txt
```

**Output:**

```
John
Jane
Mike
```

---

#### Example 2: Print Specific Fields with a Custom Field Separator

Using our `data.txt` file, where fields are comma-separated.

**Goal:** Print the Name and Salary for each person.

**Shell Command:**

```bash
awk -F',' '{ print "Name: " $1 ", Salary: " $5 }' data.txt
```

- `-F','` sets the field separator to a comma.
- `$1` is the Name, `$5` is the Salary.

**Output:**

```
Name: Name, Salary: Salary
Name: Alice, Salary: 75000
Name: Bob, Salary: 60000
Name: Charlie, Salary: 90000
Name: David, Salary: 80000
Name: Eve, Salary: 110000
Name: Frank, Salary: 70000
```

Notice the header line is also processed. We'll address that next.

---

#### Example 3: Skip Header and Print Formatted Output

**Goal:** Print Name, City, and Occupation for all records _except_ the header, with a custom output format.

**Shell Command (Multi-line):**

```bash
awk -F',' '
NR == 1 { next } # Skip the first line (header)
{
    print "Person: " $1 " lives in " $3 " and works as a " $4 "."
}' data.txt
```

- `NR` refers to the Number Record(line number).
- `NR == 1 { next }`: If the current record number (`NR`) is 1, skip to the next record.
- `print ...`: Prints the formatted string using fields `$1`, `$3`, and `$4`.

**Output:**

```
Person: Alice lives in New York and works as a Engineer.
Person: Bob lives in London and works as a Designer.
Person: Charlie lives in Paris and works as a Doctor.
Person: David lives in New York and works as a Engineer.
Person: Eve lives in London and works as a Manager.
Person: Frank lives in Berlin and works as a Developer.
```

---
#### Skip the lines that the UserName is root:
```bash
awk -F: '$1=="root" {next} {print $0}' /etc/passwd
```

- `$1=="root" {next}` If the first filed is the root skip and go to the next.
- `print $0` print the current line.

---
#### Example 4: Conditional Printing (Using `if` statements)

**Goal:** Find all engineers and print their name and city.

**Shell Command (Multi-line):**

```bash
awk -F',' '
NR == 1 { next }
$4 == "Engineer" {
    print $1 " is an Engineer in " $3 "."
}' data.txt
```

- `$4 == "Engineer"`: This is the pattern. The action will only execute if the fourth field is exactly "Engineer".

**Output:**

```
Alice is an Engineer in New York.
David is an Engineer in New York.
```

---

#### Example 5: Calculate Sum and Average (Using `BEGIN` and `END`)

**Goal:** Calculate the total and average salary from `data.txt`.

**Shell Command (Multi-line):**

```bash
awk -F',' '
BEGIN {
    total_salary = 0;
    count = 0;
    print "--- Salary Report ---"
}
NR == 1 { next } # Skip header
{
    total_salary += $5; # Add salary to total
    count++;            # Increment count
}
END {
    print "Total Salary: " total_salary
    print "Average Salary: " total_salary / count
    print "--- End Report ---"
}' data.txt
```

- `BEGIN`: Initializes `total_salary` and `count`, and prints a header.
- `total_salary += $5`: Adds the value of the 5th field (salary) to `total_salary`. `awk` automatically treats fields as numbers when used in arithmetic operations.
- `END`: Prints the calculated total and average.

**Output:**

```
--- Salary Report ---
Total Salary: 485000
Average Salary: 80833.3
--- End Report ---
```

---

#### Example 6: Using `awk` with a Script File

For more complex `awk` programs, it's better to put the script in a separate file.

Let's create a file named `process_data.awk`:

```bash
# process_data.awk
# This script processes data.txt to categorize people by age and city.

BEGIN {
    FS = ","; # Set field separator
    print "--- Detailed Employee Report ---";
    print "Name        | Age | City      | Occupation | Salary";
    print "------------|-----|-----------|------------|-------";
}

NR == 1 { next } # Skip header line

# Process each data line
{
    printf "%-12s| %-3s | %-9s | %-10s | %-6s\n", $1, $2, $3, $4, $5;

    # Conditional logic based on age
    if ($2 < 30) {
        print "  (Young professional)";
    } else if ($2 >= 30 && $2 < 40) {
        print "  (Mid-career professional)";
    } else {
        print "  (Experienced professional)";
    }

    # Conditional logic based on city
    if ($3 == "New York") {
        print "  (Based in NYC)";
    } else if ($3 == "London") {
        print "  (Based in London)";
    }
    print ""; # Add an empty line for readability
}

END {
    print "--- Report End ---";
}
```

**Shell Command to run the script:**

```bash
awk -f process_data.awk data.txt
```

- `-f process_data.awk`: Tells `awk` to read its script from the file `process_data.awk`.

**Output:**

```
--- Detailed Employee Report ---
Name        | Age | City      | Occupation | Salary
------------|-----|-----------|------------|-------
Alice       | 30  | New York  | Engineer   | 75000 
  (Mid-career professional)
  (Based in NYC)

Bob         | 24  | London    | Designer   | 60000 
  (Young professional)
  (Based in London)

Charlie     | 35  | Paris     | Doctor     | 90000 
  (Mid-career professional)

David       | 29  | New York  | Engineer   | 80000 
  (Young professional)
  (Based in NYC)

Eve         | 42  | London    | Manager    | 110000
  (Experienced professional)
  (Based in London)

Frank       | 28  | Berlin    | Developer  | 70000 
  (Young professional)

--- Report End ---
```

---

#### Example 7: Using Loops (`for` loop)

**Goal:** Print all fields of each line, prefixed with their field number.

**Shell Command (Multi-line):**

```bash
awk -F',' '
NR == 1 { next }
{
    print "Line " NR ":"
    for (i = 1; i <= NF; i++) {
        print "  Field " i ": " $i
    }
    print "" # Empty line for separation
}' data.txt
```

- `for (i = 1; i <= NF; i++)`: Loops from the first field (`i=1`) up to the total number of fields (`NF`).

**Output:**

```
Line 2:
  Field 1: Alice
  Field 2: 30
  Field 3: New York
  Field 4: Engineer
  Field 5: 75000

Line 3:
  Field 1: Bob
  Field 2: 24
  Field 3: London
  Field 4: Designer
  Field 5: 60000

... (and so on for other lines)
```


#### Using Loops (`for` loop)
- What about getting all the userName from /etc/passwd file numbered with the line number.
```bash
awk -F: 'BEGIN{counter = 0}
{for(i = 1; i <= NF; i++)
	{if(i==1)
		{print counter " : " $i; counter++} 
	else {break}}}' /etc/passwd
```

##### 🔍 Explanation:
- `-F:` → sets colon as the field separator
- `counter` → counts the lines/users
- `for (i = 1; i <= NF; i++)` → loops over all fields
- `if (i == 1)` → only acts on the **first field**
- `print counter " - " $i` → prints counter and first field
- `break` → exits the loop after processing `$1`
---

#### Example 8: String Manipulation (e.g., `substr`, `length`)

**Goal:** Print the first three characters of each name and the length of their occupation.

**Shell Command (Multi-line):**

```bash
awk -F',' '
NR == 1 { next }
{
    name_prefix = substr($1, 1, 3); # Get first 3 chars of name
    occupation_len = length($4);    # Get length of occupation string
    print "Name Prefix: " name_prefix ", Occupation Length: " occupation_len
}' data.txt
```

- `substr(string, start, length)`: Extracts a substring.
- `length(string)`: Returns the length of a string.

**Output:**

```
Name Prefix: Ali, Occupation Length: 8
Name Prefix: Bob, Occupation Length: 8
Name Prefix: Cha, Occupation Length: 6
Name Prefix: Dav, Occupation Length: 8
Name Prefix: Eve, Occupation Length: 7
Name Prefix: Fra, Occupation Length: 9
```

---

#### Example 9: Replacing Text (`sub`, `gsub`)

**Goal:** Replace "New York" with "NYC" and "London" with "LDN" in the output.

**Shell Command (Multi-line):**

```bash
awk -F',' '
NR == 1 { next }
{
    line = $0; # Work on a copy of the line
    sub(/New York/, "NYC", line); # Replace first occurrence in 'line'
    gsub(/London/, "LDN", line);  # Replace all occurrences in 'line'
    print line;
}' data.txt
```

- `sub(regex, replacement, target_string)`: Replaces the _first_ occurrence of `regex` in `target_string`. If `target_string` is omitted, it defaults to `$0`.
- `gsub(regex, replacement, target_string)`: Replaces _all_ occurrences of `regex` in `target_string`. If `target_string` is omitted, it defaults to `$0`.

**Output:**

```
Alice,30,NYC,Engineer,75000
Bob,24,LDN,Designer,60000
Charlie,35,Paris,Doctor,90000
David,29,NYC,Engineer,80000
Eve,42,LDN,Manager,110000
Frank,28,Berlin,Developer,70000
```

---

**You could see more exercises about awk here** [[awk Exercises]].