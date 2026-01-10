### String Concatenation in SQL Server

String concatenation is the process of joining two or more character strings end-to-end to form a new, longer string. This is frequently used for formatting output, creating dynamic labels, or combining data from different columns.

#### 1. Using the `+` Operator (Concatenation Operator)

The `+` operator is the traditional and most commonly used method for string concatenation in SQL Server. It's straightforward for combining non-`NULL` strings.

**Syntax:**
```sql
expression1 + expression2 + ... + expressionN
```
Where `expression` can be a column, a literal string, or another expression that evaluates to a character or binary string.

**Example Scenario:** Let's use our `Employees` table to combine `FirstName` and `LastName` into a full name.

**`Employees` Table (revisiting for this example):**

| EmployeeID | FirstName | LastName | DepartmentID | Salary |
|------------|-----------|----------|--------------|--------|
| 101        | Alice     | Smith    | 1            | 70000  |
| 102        | Bob       | Johnson  | 2            | 85000  |
| 103        | Carol     | Davis    | 1            | 72000  |
| 105        | Eve       | NULL     | NULL         | 65000  | -- LastName is NULL
| 106        | Frank     | Green    | 3            | 90000  |

**Example Query:**
```sql
SELECT
    EmployeeID,
    FirstName + ' ' + LastName AS FullName
FROM
    Employees;
```

**Result:**

| EmployeeID | FullName        |
|------------|-----------------|
| 101        | Alice Smith     |
| 102        | Bob Johnson     |
| 103        | Carol Davis     |
| 105        | NULL            | -- **Important: NULL propagation**
| 106        | Frank Green     |

**Explanation and `NULL` Handling with `+`:**
The most critical aspect of the `+` operator for concatenation is its behavior with `NULL` values. If *any* of the expressions being concatenated with `+` is `NULL`, the entire result of the concatenation will be `NULL`. This is known as **`NULL` propagation**.

In the example above, for `EmployeeID` 105 (Eve), `LastName` is `NULL`. When `FirstName + ' ' + LastName` is evaluated, `NULL` is introduced, causing the entire `FullName` for Eve to become `NULL`.

To handle `NULL` values with the `+` operator, you typically need to use functions like `ISNULL()` or `COALESCE()` on each potentially `NULL` expression.

**Example with `ISNULL()` to handle `NULL`s:**
```sql
SELECT
    EmployeeID,
    FirstName + ' ' + ISNULL(LastName, '') AS FullName
FROM
    Employees;
```

**Result:**

| EmployeeID | FullName    |                                       |
| ---------- | ----------- | ------------------------------------- |
| 101        | Alice Smith |                                       |
| 102        | Bob Johnson |                                       |
| 103        | Carol Davis |                                       |
| 105        | Eve         | -- `LastName` (NULL) replaced with '' |
| 106        | Frank Green |                                       |

Here, `ISNULL(LastName, '')` replaces `NULL` with an empty string, preventing `NULL` propagation.


_**In other Scenario**_
**Example Scenario:** Let's use our `Employees` table to combine `EmpoyeeID` and `LastName` into a  new column.
- As the `EmployeeID` is `int`, and the `LastName` is `varchar()` then contacting them will `+` operator will case an Error, so we cant use the concatenation operator `+` with different data types. 
- But we could work around to solve this Scenario by using the `Convert()` function which could convert one data type to another.

```SQL
select convert(varchar(5), EmployeeId) + '-' + isnull(LastName, ' ') As 'New Column'
from employee;
```


| New Column    |
| ------------- |
| 101 - Smith   |
| 102 - Johnson |
| 103 - Davis   |
| 105 -         |
| 106 - Green   |
- AS we can see the `convert()` function converted the `EmployeeID` from `int` to `varchar(5)` to make it suitable to be concatied with the `LastName` which is `varchar()`.  

#### 2. Using the `CONCAT()` Function

The `CONCAT()` function was introduced in SQL Server 2012 and provides a more convenient way to concatenate strings, especially when dealing with `NULL` values.

**Syntax:**
```sql
CONCAT (expression1, expression2, ..., expressionN)
```
The `CONCAT()` function takes two or more string arguments and joins them.

**Example Query:**
```sql
SELECT
    EmployeeID,
    CONCAT(FirstName, ' ', LastName) AS FullName
FROM
    Employees;
```

**Result:**

| EmployeeID | FullName        |
|------------|-----------------|
| 101        | Alice Smith     |
| 102        | Bob Johnson     |
| 103        | Carol Davis     |
| 105        | Eve             | -- **Important: NULLs treated as empty strings**
| 106        | Frank Green     |

**Explanation and `NULL` Handling with `CONCAT()`:**
The key advantage of `CONCAT()` is its built-in handling of `NULL` values. Unlike the `+` operator, `CONCAT()` implicitly converts any `NULL` argument to an empty string (`''`) before concatenation. This means `NULL` values do not propagate and will not turn the entire result into `NULL`, in addition to that it also convert all the values to `varchar()` to make them suitable to be concatied together.

For `EmployeeID` 105 (Eve), `CONCAT(FirstName, ' ', LastName)` effectively becomes `CONCAT('Eve', ' ', '')`, resulting in 'Eve '.

**Advantages of `CONCAT()` over `+`:**
*   **`NULL` Handling:** Automatically converts `NULL` inputs to empty strings, simplifying code and preventing unexpected `NULL` results.
*   **Readability:** Can be more readable, especially when concatenating many expressions, as you don't need to wrap each potentially `NULL` expression with `ISNULL()` or `COALESCE()`.
*   **Data Type Conversion:** `CONCAT()` implicitly converts all arguments to string types before concatenation, which can be convenient. The `+` operator might require explicit `CAST` or `CONVERT` if you're mixing string and numeric types, for instance.

#### 3. Using the `CONCAT_WS()` Function (Concatenate With Separator)

While not explicitly asked for, it's worth mentioning `CONCAT_WS()` as an even more advanced and convenient concatenation function, also introduced in SQL Server 2012.

**Syntax:**
```sql
CONCAT_WS (separator, expression1, expression2, ..., expressionN)
```
This function concatenates a list of string arguments with the specified separator. It also skips `NULL` values, meaning the separator is only inserted between non-`NULL` expressions.

**Example Query:**
```sql
SELECT
    EmployeeID,
    CONCAT_WS(' ', FirstName, LastName) AS FullName
FROM
    Employees;
```

**Result:**

| EmployeeID | FullName        |
|------------|-----------------|
| 101        | Alice Smith     |
| 102        | Bob Johnson     |
| 103        | Carol Davis     |
| 105        | Eve             | -- `LastName` (NULL) is skipped, no extra space
| 106        | Frank Green     |

**Explanation:** For Eve, `CONCAT_WS(' ', 'Eve', NULL)` results in just 'Eve' because the `NULL` `LastName` is ignored, and no separator is added for it. This is often the most elegant solution for names or addresses where parts might be missing.

### Choosing the Right Concatenation Method

*   **`+` Operator:** Use when you are certain all parts will be non-`NULL`, or when you explicitly want `NULL` propagation (e.g., if any part is missing, the whole result should be `NULL`). If `NULL`s are possible, you *must* use `ISNULL()` or `COALESCE()` with each nullable expression.
*   **`CONCAT()` Function:** This is generally the preferred method for most string concatenation tasks, especially when `NULL` values might be present, as it handles them gracefully by treating them as empty strings. It's more readable and less error-prone than `+` with multiple `ISNULL()` calls.
*   **`CONCAT_WS()` Function:** Ideal when you need to concatenate multiple strings with a consistent separator, and you want to automatically skip any `NULL` values without adding extra separators. This is often the cleanest solution for constructing formatted strings like full names, addresses, or lists.

As a senior developer, I strongly advocate for using `CONCAT()` or `CONCAT_WS()` in modern SQL Server development due to their superior `NULL` handling and improved readability, which leads to more robust and maintainable code.