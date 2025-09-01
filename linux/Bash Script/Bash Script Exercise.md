# Bash Script Exercise

- <span style="color:rgb(255, 0, 0)">Create a script that asks for user name then send a greeting to him.</span>

```Bash
#!/bin/bash

read -p "Enter your Name: " name
echo "Hello, $name"
```


---
- <span style="color:rgb(255, 0, 0)">Create a script called s1 that calls another script s2 where:<br>a. In s1 there is a variable called x, it's value 5<br>b. Try to print the value of x in s2 by two different ways.<br></span>
**S1 file**
```Bash
#!/bin/bash
x=5
export x
./s2.sh "$x"
```
- In the s1.sh file we created a variable x with value 5 then we export it to make it global to our current session.
- then we called the s2.sh file and pass `$x` the value of the variable x to it.

**S2 file**
```Bash
#!/bin/bash

echo "first way to print x: $x"

echo "Scond way to print x: $1"
```
- In the s2.sh file we used the argument x which we passed in s1.sh file while calling s2.sh and we print it twice once as a global variable `$x` another one as `$1` the first argument.



---
s
