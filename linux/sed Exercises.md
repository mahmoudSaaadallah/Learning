**Display the lines that contain the word “lp” in /etc/passwd file.**

```Bash
sed -n '/lp/p' /etc/passwd
```

---
**Display /etc/passwd file except the third line.**

```Bash
sed '3d' /etc/passwd
```

---
**Display /etc/passwd file except the last line.**

```Bash
sed '$d' /etc/passwd
```

---
**Display /etc/passwd file except the lines that contain the word “lp”.**

```Bash
sed '/lp/d' /etc/passwd
```

---
**Substitute all the words that contain “lp” with “mylp” in /etc/passwd file.**

```Bash
sed 's/lp/mylp/g' /etc/passwd
```

---
replace **whole words that contain "lp"** (like "help", "alpha", "pulp") entirely with `"mylp"`?

```Bash
sed -E 's/\w*lp\w*/mylp/g' etc/passwd
```
### 🧠 Explanation:

- `-E` → enables **extended regex** (lets you use `\w`, `+`, etc. without escaping parentheses)
- `\w*lp\w*` → matches any word that contains `"lp"`:
    - `\w*` = 0 or more letters/numbers before or after
    - So it matches `lp`, `help`, `pulp`, `alpha`, `lpuser`, etc.

