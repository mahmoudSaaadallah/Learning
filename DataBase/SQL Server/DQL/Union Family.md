### The Union Family: Combining Result Sets

The `UNION` family of operators is used to combine the result sets of two or more `SELECT` statements into a single result set. Think of it as stacking one query's output on top of another. While seemingly simple, the nuances between `UNION` and `UNION ALL` are critical for performance and data accuracy.

Before we dive into the specifics, there are fundamental rules that apply to all `UNION` operations:

1.  **Number of Columns:** Each `SELECT` statement within the `UNION` query must have the same number of columns.
2.  **Order of Columns:** The columns in each `SELECT` statement must be in the same logical order.
3.  **Compatible Data Types:** The data types of corresponding columns in each `SELECT` statement must be compatible. SQL Server will attempt implicit conversions if possible, but it's best practice to ensure they are the same or explicitly `CAST`/`CONVERT` [[T-SQL Casting]] them.
4.  **Column Names:** The column names in the final result set are typically derived from the first `SELECT` statement.

Let's set up some hypothetical tables to illustrate:

**1. `Employees` Table:**

| EmployeeID | FirstName | LastName | JobTitle      |
|------------|-----------|----------|---------------|
| 101        | Alice     | Smith    | Developer     |
| 102        | Bob       | Johnson  | Manager       |
| 103        | Carol     | Davis    | Developer     |
| 104        | David     | Brown    | QA Engineer   |

**2. `Contractors` Table:**

| ContractorID | FirstName | LastName | JobTitle      |
|--------------|-----------|----------|---------------|
| 201          | Carol     | Davis    | Developer     | -- This person also works as a contractor
| 202          | Eve       | White    | UI Designer   |
| 203          | Frank     | Green    | Project Lead  |
| 204          | Alice     | Smith    | Developer     | -- This person also works as a contractor

Notice that Carol Davis and Alice Smith appear in both tables, representing individuals who might be both full-time employees and also work on specific projects as contractors. This will be key for our examples.

---

#### 1. `UNION` Operator

The `UNION` operator combines the result sets of two or more `SELECT` statements and, crucially, **removes duplicate rows** from the final result. It implicitly performs a `DISTINCT` operation on the combined data.

**Purpose:** To get a consolidated list of unique rows from multiple queries.

**Syntax:**
```sql
SELECT column1, column2, ...
FROM Table1
[WHERE condition]
UNION
SELECT column1, column2, ...
FROM Table2
[WHERE condition];
```

**Example:** Retrieve a unique list of all people (employees and contractors) and their job titles.

```sql
SELECT FirstName, LastName, JobTitle
FROM Employees
UNION
SELECT FirstName, LastName, JobTitle
FROM Contractors;
```

**Result:**

| FirstName | LastName | JobTitle      |
|-----------|----------|---------------|
| Alice     | Smith    | Developer     |
| Bob       | Johnson  | Manager       |
| Carol     | Davis    | Developer     |
| David     | Brown    | QA Engineer   |
| Eve       | White    | UI Designer   |
| Frank     | Green    | Project Lead  |

**Explanation:**
*   The query combines the rows from both `Employees` and `Contractors`.
*   `Alice Smith, Developer` appears in both tables, but `UNION` returns it only once.
*   `Carol Davis, Developer` also appears in both tables, but `UNION` returns it only once.
*   The `EmployeeID` and `ContractorID` columns were not selected, as they are distinct identifiers for different roles, but the `FirstName`, `LastName`, `JobTitle` combination is considered a duplicate across the two sets.

**Performance Consideration:** Because `UNION` must sort and compare all rows to identify and remove duplicates, it can be more resource-intensive and slower than `UNION ALL`, especially with large datasets.

---

#### 2. `UNION ALL` Operator

The `UNION ALL` operator combines the result sets of two or more `SELECT` statements and **includes all rows**, even if they are duplicates. It does not perform any implicit `DISTINCT` operation.

**Purpose:** To get a complete, unfiltered concatenation of all rows from multiple queries, including any duplicates.

**Syntax:**
```sql
SELECT column1, column2, ...
FROM Table1
[WHERE condition]
UNION ALL
SELECT column1, column2, ...
FROM Table2
[WHERE condition];
```

**Example:** Retrieve a complete list of all entries for people (employees and contractors) and their job titles, including duplicates.

```sql
SELECT FirstName, LastName, JobTitle
FROM Employees
UNION ALL
SELECT FirstName, LastName, JobTitle
FROM Contractors;
```

**Result:**

| FirstName | LastName | JobTitle     |                                     |
| --------- | -------- | ------------ | ----------------------------------- |
| Alice     | Smith    | Developer    |                                     |
| Bob       | Johnson  | Manager      |                                     |
| Carol     | Davis    | Developer    |                                     |
| David     | Brown    | QA Engineer  |                                     |
| Carol     | Davis    | Developer    | -- Duplicate from Contractors table |
| Eve       | White    | UI Designer  |                                     |
| Frank     | Green    | Project Lead |                                     |
| Alice     | Smith    | Developer    | -- Duplicate from Contractors table |

**Explanation:**
*   All rows from both `Employees` and `Contractors` are returned.
*   `Alice Smith, Developer` appears twice because it exists in both original tables.
*   `Carol Davis, Developer` also appears twice.

**Performance Consideration:** Since `UNION ALL` does not need to sort and remove duplicates, it is generally **faster and less resource-intensive** than `UNION`. If you know your combined result sets won't have duplicates, or if you explicitly want to keep all duplicates, `UNION ALL` is the preferred choice.

---

### Key Differences Summary

| Feature        | `UNION`                                     | `UNION ALL`                                 |
|----------------|---------------------------------------------|---------------------------------------------|
| **Duplicates** | Removes duplicate rows (performs `DISTINCT`) | Includes all rows, even duplicates          |
| **Performance**| Generally slower (requires sorting/hashing) | Generally faster (no sorting/hashing)       |
| **Use Case**   | When a unique list of combined data is needed | When all combined data is needed, regardless of uniqueness |

---

### Important Considerations and Best Practices

1.  **Column Compatibility:** Always ensure that the corresponding columns in your `SELECT` statements have compatible data types. If they are not identical, use `CAST()` or `CONVERT()` to explicitly make them compatible to avoid errors or unexpected implicit conversions.
```sql
-- Example of explicit conversion for compatibility
SELECT EmployeeID, FirstName, LastName, CAST(Salary AS VARCHAR(20)) AS Value
FROM Employees
UNION ALL
SELECT ContractorID, FirstName, LastName, CAST(HourlyRate * HoursWorked AS VARCHAR(20)) AS Value
FROM Contractors;
```

2.  **`ORDER BY` Clause:** The `ORDER BY` clause can only be specified once, at the very end of the entire `UNION` query, and it applies to the final combined result set.
```sql
SELECT FirstName, LastName, JobTitle
FROM Employees
UNION ALL
SELECT FirstName, LastName, JobTitle
FROM Contractors
ORDER BY LastName, FirstName; -- Applies to the entire combined result
```

3.  **Column Aliases:** Column aliases defined in the first `SELECT` statement will be used for the final result set. Aliases in subsequent `SELECT` statements are ignored.

4.  **When to Choose `UNION` vs. `UNION ALL`:**
    *   **Prefer `UNION ALL`** unless you specifically need to remove duplicates. It's more efficient.
    *   If you need to remove duplicates, but only based on a subset of columns, you might use `UNION ALL` and then apply `DISTINCT` to the specific columns in an outer query or use a `GROUP BY` clause. However, if the entire row is a duplicate, `UNION` is simpler.

5.  **Multiple `UNION` Operations:** You can chain multiple `UNION` or `UNION ALL` operators together. The rules of precedence are from left to right, but it's often clearer to use parentheses if you have a mix of `UNION` and `UNION ALL` and specific duplicate handling is required at intermediate steps.

Mastering the `UNION` family is crucial for effectively integrating and presenting data from various sources within your SQL Server environment. Always consider the impact on duplicates and performance when making your choice between `UNION` and `UNION ALL`.