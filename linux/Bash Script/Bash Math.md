# Bash Math
- In Bash if we want to add two numbers we can't go an write `5+4` directly like this, as it's not like python.
- On the other hand we have to use `expr` command before the mathematical operation to make it work
```Bash
expr 5 + 5
# 10
expr 5+5
# 5+5
```
- `expr` refers to expression and it tell the bash shell that we want to make an expression.
- As we saw after `expr` we have to add spaces in the operation to get the result or it will not work, and will return it as it is.

```Bash
expr 2 - 1
# 1

expr 100 / 4
# 25
```

- The subtraction and the division also work the same.

```Bash
expr 2 * 4
# Error
```
- With multiplication we can't use `*` because it's a wildcard in Linux.
- So to solve this problem we have to use scape character

```Bash
expr 2 \* 4
# 8
```
- using scape character will make `*` work fine.

**We could do the same things with variables:**
```Bash
var=22
var2=11
expr $var / $var2
# 2
```






----
## Using echo to do Mathematical operation:
You can add two numbers using `echo` by leveraging Bash's **arithmetic expansion**.
The syntax for arithmetic expansion in Bash is `$((expression))`.
Here's how you do it:

```bash
echo $((5 + 3))
```

**Explanation:**

- `echo`: This command is used to display lines of text.
- `$(( ... ))`: This is Bash's arithmetic expansion. Whatever mathematical expression you put inside the double parentheses will be evaluated, and its result will be substituted into the command line.

**Example with variables:**

You can also use variables:

```bash
num1=10
num2=25
echo $((num1 + num2))
```

This will output `35`.

You can perform other arithmetic operations as well:

- Addition: `+`
- Subtraction: `-`
- Multiplication: `*`
- Division: `/`
- Modulo (remainder): `%`

For example:

```bash
echo $((10 * 5))   # Output: 50
echo $((20 / 4))   # Output: 5
echo $((17 % 5))   # Output: 2 (remainder of
```