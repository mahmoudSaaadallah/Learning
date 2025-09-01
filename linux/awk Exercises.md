## Using `awk` answer the following:

**All the following examples will be applied to the passwd file**.

- <span style="color:rgb(255, 0, 0)">Print full name (comment) of all users in the system.</span> 
```bash
awk -F: 'print $5' /etc/passwd
```


---

- <span style="color:rgb(255, 0, 0)">Print login, full name (comment) and home directory of all users.( Print each line preceded by a line number)</span>
```bash
awk -F: '{print "Login: " $1 " FullName: " $5 " Home: " $6 "\n"}' /etc/passwd
```
- `"\n"` to make new line.


---
- <span style="color:rgb(255, 0, 0)">Print login, uid and full name (comment) of those uid is greater than 500</span> 
```bash
awk -F: '{if ($3 > 500){print "Login: " $1 " UID: " $3 " FullName: " $5 "\n"}}' /etc/passwd
```
- here we used `if` condation.


---
- <span style="color:rgb(255, 0, 0)">Print login, uid and full name (comment) of those uid is exactly 500</span> 
```bash
awk -F: '{if ($3 == 500){print "Login: " $1 " UID: " $3 " FullName: " $5 "\n"}}' /etc/passwd
```


---
- <span style="color:rgb(255, 0, 0)">Print line from 5 to 15 from /etc/passwd</span> 
```Bash
awk '{for (i = 5; i <= 15; i++){if (NR == i) {print $0})}}' /etc/passwd
```
 Or 
 ```Bash
 awk '{if(NR >= 5 && NR <= 15){print $0}}'
 ```


---
- <span style="color:rgb(255, 0, 0)">Change lp to mylp</span> 
```Bash
awk '{gsub(/lp/, "mylp"); print}' /etc/passwd
```
Or
```Bash
awk '{gsub("lp", "mylp"); print}' /etc/passwd
```


---
- <span style="color:rgb(255, 0, 0)">Print all information about greatest uid.</span> 
```Bash
awk 'BEGIN{max=0}{if($3 > max){max=$3; maxLine=$0}} END{print maxLine}' /etc/passwd
```


---
- <span style="color:rgb(255, 0, 0)">Get the sum of all accounts id’s.</span> 
```Bash
awk 'BEGIN{sum=0}{sum = sum + $3} END{print sum}' /etc/passwd
```
