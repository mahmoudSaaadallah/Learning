# Bash Variables
- To create a variable in bash we could add it like:
```Bash
var="Mahmoud"
```
- Here as we can see there is no space between the name of the variable and the equal sign `=` also there is no space between the equal sign and the value `"Mahmoud"`.
- This mandatory as we can't add space with variables in the shell.

- To call this variable and retrieve its data we could use `ehco` command.
```Bash
echo $var
```
- `$` dollar sign must be sued before the variable name to get its value.
- We have to know that without using the dollar sign `echo` command will print the word after it not its value as it will deal with it as a string not a variable.

==We have to know that the variables that we create in our shell is used for this session only, so if we close this session using `exit` command then all our created variables will be automatically deleted.==



---
## Difference between using double quote and single quote.
- When using double quote with string that includes a variable name with `$` dollar sign before it we will be able to get the value of that variable.

```Bash
name="Mahmoud Saadallah"
echo "hello, $name"
```

```output
hell, Mahmoud Saadallah
```

- but when using single quote this will print the string as it's without  the value.
```Bash
name="Mahmoud Saadallah"
echo 'hell, $name'
```

```output
hell, $name
```
- As we can see even the editor didn't color the variable name as it's between double quote, so when ever we want to concat a variable inside a string we need to use double quote.


---

## Making variable with value from a command
- We could make a variable that stores an output of one of Linux command. For example to store the output of the `ls` command into a variable we could do that by:
```Bash
var=$(ls)
```
- `$(ls)` this is called a subshell.
- Subshell allows you to execute a command in the background and send it's result to anywhere like storing in variable or echo or whatever you want. 

---
## Environment Variables
- In Linux there are some variables that already defined in the OS by default like `USER`, `PATH`, `HOME`, and more and more
- To display all the environment variables we could use `env` command which will return all the environment variables.