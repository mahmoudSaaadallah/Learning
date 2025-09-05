[[Function]]

### 🔧 Exercise 1: Greet Function (Review)

**Define a function `greet_user` that:**

- Takes **1 argument** (a name)
- Prints: `Hello, <name>!`
```Bash
#!/bin/bash

greet_user() {
    cho "Hello, $1"
}
greet_user Alice
```

-----------------------------
### ⚙️ Exercise 2: Sum Two Numbers

Write a function called `add_numbers` that:

- Takes **two numbers**
- Adds them
- Returns the result via `echo`
```Bash
#!/bin/bash

sum() {
	echo $(( $1 + $2 ))
}
sum 11 15
```

-----------------------------
### 🧪 Exercise 3: Is Even

Write a function `is_even` that:
- Takes one number
- Prints `"Even"` or `"Odd"`
```Bash
#!/bin/bash

even_odd() {
    if (( $# == 0 )); then
		read -p "Please enter the number: " number
		if(( $number % 2 == 0 )); then
			echo "$number is an even number"
		else
			echo "$number is an odd number"
		fi
	else
		if(( $1 % 2 == 0 )); then
			echo "$1 is an even number"
		else
			echo "$1 is an odd number"
		fi
	fi
}
even_odd $1

```

---------------------------------
### 💡 Exercise 4: File Check

Write a function `file_exists` that:
- Takes a **filename**
- Prints `"File exists"` if it exists, otherwise `"File not found"`

```Bash
#!/bin/bash

check_file() {
	if (( $# == 0 )); then
		echo "You didn't provide any arguments"
		return 1
	fi
	if [ -f $1 ]; then
		echo "$1 file exist"
	else
		echo "$1 file not exist"
	fi
}
check_file /etc/passwd
```

---

### ✅ Exercise 5: sum array

Can you write a function called `sum_array` that:
- Accepts an array of numbers
- Sums them up
- Echoes the result

```Bash
#!/bin/bash

sum_arr() {
	local arr=("$@")
	local result=0
	for num in "${arr[@]}"; do
		result=$(( $result + num ))
	done
	echo "$result"
}
a=( 1 2 3 4 )
sum_arr ${a[@]}
```

