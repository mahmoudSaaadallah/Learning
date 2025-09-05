# Data Streams
In Linux, processes interact with the system and other processes through **data streams**. These streams are essentially channels for input and output. Every program, when it starts, typically has three standard data streams automatically opened for it:

### Standard Data Streams

1.  **Standard Input (stdin)** | (_File Descriptor 0_)
    *   This is the channel from which a program receives its input.
    *   By default, `stdin` is connected to your keyboard, meaning programs expect you to type input directly.
    *   Examples: `read` command, `cat` without arguments waiting for input.

2.  **Standard Output (stdout)** | (_File Descriptor 1_)
    *   This is the channel where a program sends its normal output.
    *   By default, `stdout` is connected to your terminal screen, so you see the results of commands printed there.
    *   Examples: `ls`, `echo`, `cat file.txt`.

3.  **Standard Error (stderr)** | (_File Descriptor 2_)
    *   This is the channel where a program sends its error messages or diagnostic output.
    *   Like `stdout`, `stderr` is also connected to your terminal screen by default. This separation allows you to distinguish between normal output and error messages, and to handle them differently.
    *   Examples: `ls non_existent_file`, `grep -r non_existent_pattern`.

### Redirection

Redirection is the process of changing the default source or destination of these standard data streams. Instead of the keyboard for input or the screen for output/errors, you can redirect them to files or even to other commands.

#### 1. Output Redirection (stdout and stderr)

This allows you to send the output of a command to a file instead of the screen.

*   **`>` (Redirect stdout, overwrite)**
    *   Sends the `stdout` of a command to a file. If the file exists, it will be overwritten.
    *   Example: `ls -l > file_list.txt` (The output of `ls -l` is saved to `file_list.txt`, overwriting it if it exists.)

*   **`>>` (Redirect stdout, append)**
    *   Sends the `stdout` of a command to a file. If the file exists, the output will be appended to the end of the file. If the file doesn't exist, it will be created.
    *   Example: `echo "Another line" >> file_list.txt` (Adds "Another line" to the end of `file_list.txt`.)

*   **`2>` (Redirect stderr, overwrite)**
    *   Sends the `stderr` of a command to a file. The `2` refers to the file descriptor for `stderr`.
    *   Example: `ls non_existent_file 2> errors.log` (Any error messages from `ls` are saved to `errors.log`.)

*   **`2>>` (Redirect stderr, append)**
    *   Sends the `stderr` of a command to a file, appending to it if it exists.
    *   Example: `grep -r non_existent_pattern /etc 2>> errors.log` (Appends error messages to `errors.log`.)

*   **`&>` or `>&` (Redirect both stdout and stderr, overwrite)**
    *   Sends both `stdout` and `stderr` to the same file.
    *   Example: `my_command &> all_output.log` (Both normal output and errors go to `all_output.log`.)

*   **`&>>` (Redirect both stdout and stderr, append)**
    *   Sends both `stdout` and `stderr` to the same file, appending to it.
    *   Example: `my_command &>> all_output.log`

*   **Redirecting stderr to stdout (and then both to a file)**
    *   A common pattern is to merge `stderr` into `stdout` and then redirect the combined stream.
    *   `command > file 2>&1`
        *   `>` file: Redirects `stdout` to `file`.
        *   `2>&1`: Redirects `stderr` (file descriptor 2) to wherever `stdout` (file descriptor 1) is currently pointing (which is `file`). The order matters here; `2>&1` must come after `> file`.
    *   Example: `find / -name "*.conf" > results.txt 2>&1` (Both found files and permission errors are saved to `results.txt`.)

#### 2. Input Redirection

This allows you to provide input to a command from a file instead of the keyboard.

*   **`<` (Redirect stdin)**
    *   Takes the content of a file and feeds it as `stdin` to a command.
    *   Example: `wc -l < file_list.txt` (Counts the lines in `file_list.txt` by reading its content as input to `wc -l`.)

*   **`<<<` (Here String)**
    *   Provides a single string as `stdin` to a command.
    *   Example: `grep "hello" <<< "hello world"` (The string "hello world" is passed as input to `grep`.)

*   **`<<EOF` (Here Document)**
    *   Allows you to provide multi-line input directly within the script or command line, terminated by a specified delimiter (e.g., `EOF`).
    *   Example:
        ```bash
        cat << END_OF_INPUT
        This is the first line.
        This is the second line.
        END_OF_INPUT
        ```
        (The `cat` command will receive the two lines between `<<END_OF_INPUT` and `END_OF_INPUT` as its standard input.)

#### 3. Pipes

*   **`|` (Pipe)**
    *   The pipe operator connects the `stdout` of one command to the `stdin` of another command. This allows you to chain commands together, where the output of one becomes the input of the next.
    *   Example: `ls -l | grep ".txt"` (The output of `ls -l` is piped as input to `grep`, which then filters for lines containing ".txt".)
    *   Example: `cat file.txt | sort | uniq` (The content of `file.txt` is sorted, and then unique lines are displayed.)

Understanding data streams and redirection is fundamental to effective command-line usage and scripting in Linux, enabling powerful automation and data manipulation.