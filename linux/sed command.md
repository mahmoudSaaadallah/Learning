# sed
`sed` refers to stream editor, which could be used to filter and modify text files.

- By default `sed` command prints the modified stream to standard output, without changing the original file.

### What is `sed`?

- `sed` is a non-interactive text editor. Unlike interactive editors like `vi` or `nano`, `sed` processes text automatically based on a script of commands. 
- It reads input line by line, applies the specified operations, and then outputs the result.

### Primary Use Cases

- **Text Substitution:** Replacing specific patterns with other strings. This is arguably its most common use.
- **Deletion:** Removing lines or parts of lines that match a pattern.
- **Insertion/Appending:** Adding new lines before or after lines that match a pattern.
- **Filtering:** Extracting specific lines from a file.
- **Text Transformation:** Reordering, reformatting, or manipulating text in various ways.

### Basic Syntax

The general syntax for `sed` is:  
```bash
sed [options] 'script' [file...]
```

- **`options`**: Control `sed`'s behavior (e.g., `-i` for in-place editing(to change the original file itself, `-n` to suppress default output(stop the default action that happen after execute which is printing the result)).
- **`script`**: A single `sed` command or a series of commands. This is often enclosed in single quotes to prevent shell expansion.
- **`file...`**: One or more input files. If no files are specified, `sed` reads from standard input(keyboard).

A `sed` command typically follows the format: `[address]command[arguments]`

- **`address`**: Specifies which lines the command should apply to. This can be a line number, a range of line numbers, or a regular expression. If no address is given, the command applies to all lines.
- **`command`**: The action to perform (e.g., `s` for substitute(change), `d` for delete, `p` for print).
- **`arguments`**: Additional parameters for the command (e.g., the pattern and replacement string for `s`).



### Common `sed` Commands and Examples

#### 1.Substitution (`s`)  
- This is the most frequently used command. It replaces occurrences of a pattern with a replacement string.  
	`s/pattern/replacement/flags`

- **Replace the first occurrence of "foo" with "bar" on each line:**  
```bash
sed 's/foo/bar/' file.txt
```



- **Replace all occurrences of "foo" with "bar" on each line (global flag `g`):**  
```bash
sed 's/foo/bar/g' file.txt
```
Use `g` for global replacement (all matches on a line)




- **Replace "foo" with "bar" only on lines containing "baz":**  
```bash
sed '/baz/s/foo/bar/g' file.txt
```



- **Case-insensitive replacement (ignore case flag `i`):**  
```bash
sed 's/foo/bar/gi' file.txt
```




- **Using a different delimiter (e.g., `#` instead of `/`) for paths:**  
```bash
sed 's#/old/path#/new/path#g' file.txt
```




- **Addressing**
- `sed` commands can be applied to specific lines or ranges.
    - Single line:
        `sed '3s/old/new/' file`
        Substitute only on line 3.
    
    - Range of lines:
        `sed '2,5d' file`
        Delete lines 2 to 5.



#### Deletion (`d`)
Deletes lines that match a pattern or are within a specified range.
    
- **Delete lines containing "error":**  
```bash
sed '/error/d' log.txt
```



- **Delete lines 1 to 5:**  
```bash
sed '1,5d' file.txt
```



- **Delete the last line:**  
```bash 
sed '$d' file.txt
```



#### Printing (`p`)
Prints lines that match a pattern. Often used with the `-n` option to suppress default output and only print explicitly selected lines.

- **Print only lines containing "important":**  
``` bash
sed -n '/important/p' file.txt
```



- **Print lines 10 to 20:**  
```bash
sed -n '10,20p' file.txt
```


- By default, `sed` prints all lines.
    
- Use `-n` to suppress automatic printing.
    
- Use `p` to print specific lines.
    

Example: Print only lines containing "error":
```bash
sed -n '/error/p' file
```




#### Insertion (`i`) and Appending (`a`)
   Adds new lines of text. `i` inserts _before_ the addressed line, `a` appends _after_.
  
- **Insert "--- HEADER ---" before the first line:**  
```bash
sed '1i\--- HEADER ---' file.txt
```


- **Append "--- FOOTER ---" after the last line:**  
```bash
sed '$a\--- FOOTER ---' file.txt
```



#### In-place Editing (`-i`)  
Modifies the file directly instead of printing to standard output. It's crucial to use this carefully, as it overwrites the original file. You can also create a backup.

- **Replace "old" with "new" directly in `file.txt`:**  
```bash
sed -i 's/old/new/g' file.txt
```




- **Replace "old" with "new" and create a backup `file.txt.bak`:**  
```bash
sed -i.bak 's/old/new/g' file.txt
```



####  Using Multiple Commands
- Multiple commands can be separated by `-e` or `;` inside the script.
```bash
sed -e 's/foo/bar/' -e '/baz/d' file
```

or

```bash
sed 's/foo/bar/; /baz/d' file
```



---
#### Examples of Practical Uses

- Replace text globally:
```bash
sed 's/apple/orange/g' file.txt
```
   
- Delete blank lines:
```bash
sed '/^$/d' file.txt
```

- Insert a line before a matching pattern:
```bash
sed '/pattern/i\This is a new line' file.txt
```

- Append a line after a matching pattern:
 ```bash
 sed '/pattern/a\This is appended' file.txt
 ```

- Print specific lines (e.g., lines 10 to 20):
```bash
sed -n '10,20p' file.txt
```

 ---> If you know the exact line number you want to exclude:


- Exclude line 5(print all the line except specific line):
```bash
sed '5d' your_file.txt
```
---> This command tells `sed` to delete the 5th line and print all other lines.

- print all the line except the last line:
```bash
sed '$d' /etc/passwd
```


- **Exclude all lines containing the word "error" and print the rest:**
```bash
sed '/error/d' your_file.txt
```
 ---> This command deletes any line that contains the string "error" and print the other 
       lines.

- Display /etc/passwd file except the lines that contain the word “lp”.
```bash
sed '/lp/d' /etc/passwd
```

- Substitute all the words that contain “lp” with “mylp” in /etc/passwd file.
```bash
sed 's/lp/mylp/g' /etc/passwd
```