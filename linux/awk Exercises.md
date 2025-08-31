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