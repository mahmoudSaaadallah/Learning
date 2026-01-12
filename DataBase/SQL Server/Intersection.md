### The `INTERSECT` Operator: Finding Common Ground

The `INTERSECT` operator is used to combine the result sets of two or more `SELECT` statements, but unlike `UNION` [[Union Family]] which combines all unique rows, `INTERSECT` returns only the rows that are **common to all** the `SELECT` statements. In essence, it finds the intersection of the data sets.

**Purpose:** To identify and retrieve records that exist in *all* participating queries. This is particularly useful for finding entities that share multiple characteristics or belong to multiple groups.

Just like with `UNION`, there are fundamental rules that apply to all `INTERSECT` operations:

1.  **Number of Columns:** Each `SELECT` statement within the `INTERSECT` query must have the same number of columns.
2.  **Order of Columns:** The columns in each `SELECT` statement must be in the same logical order.
3.  **Compatible Data Types:** The data types of corresponding columns in each `SELECT` statement must be compatible. SQL Server will attempt implicit conversions if possible, but it's best practice to ensure they are the same or explicitly `CAST`/`CONVERT` them.
4.  **Column Names:** The column names in the final result set are typically derived from the first `SELECT` statement.

Let's reuse our `Employees` and `Contractors` tables for a clear demonstration:

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
| 201          | Carol     | Davis    | Developer     |
| 202          | Eve       | White    | UI Designer   |
| 203          | Frank     | Green    | Project Lead  |
| 204          | Alice     | Smith    | Developer     |

Recall that Alice Smith and Carol Davis appear in both tables with the same `FirstName`, `LastName`, and `JobTitle`. These are the "common ground" we're looking for.

---

#### Using the `INTERSECT` Operator

The `INTERSECT` operator returns only the distinct rows that are present in *all* the `SELECT` statements.

**Syntax:**
```sql
SELECT column1, column2, ...
FROM Table1
[WHERE condition]
INTERSECT
SELECT column1, column2, ...
FROM Table2
[WHERE condition];
```

**Example:** Retrieve a list of people (and their job titles) who are *both* full-time employees *and* contractors.

```sql
SELECT FirstName, LastName, JobTitle
FROM Employees
INTERSECT
SELECT FirstName, LastName, JobTitle
FROM Contractors;
```

**Result:**

| FirstName | LastName | JobTitle    |
|-----------|----------|-------------|
| Alice     | Smith    | Developer   |
| Carol     | Davis    | Developer   |

**Explanation:**
*   The query evaluates the result set of the first `SELECT` statement (from `Employees`) and the second `SELECT` statement (from `Contractors`).
*   It then identifies the rows that are *identical* in both result sets.
*   `Alice Smith, Developer` exists in both.
*   `Carol Davis, Developer` also exists in both.
*   Other rows (Bob Johnson, David Brown from `Employees`; Eve White, Frank Green from `Contractors`) are unique to their respective tables and thus are excluded from the `INTERSECT` result.
*   Importantly, `INTERSECT` implicitly performs a `DISTINCT` operation, meaning even if a common row appeared multiple times in *both* original queries, it would only appear once in the final `INTERSECT` result.

---

### Key Characteristics and Differences

*   **Duplicate Handling:** `INTERSECT` automatically removes duplicate rows from its final result. If a row appears in both sets, it's included only once. This is similar to `UNION` but different from `UNION ALL`.
*   **Order of Operations:** The order of the `SELECT` statements around `INTERSECT` does not affect the final result, as intersection is commutative (A ∩ B = B ∩ A).
*   **Performance:** Like `UNION`, `INTERSECT` requires sorting and comparing rows to find common elements and remove duplicates. This can make it less performant than `UNION ALL` for very large datasets, but often more efficient than achieving the same result with complex `JOIN` and `WHERE` clauses (e.g., `INNER JOIN` with additional `AND` conditions for all columns).

### Important Considerations and Best Practices

1.  **Column Compatibility:** As with `UNION`, strict adherence to the same number of columns and compatible data types is crucial. Mismatches will lead to errors. Use `CAST()` or `CONVERT()` for explicit type handling if necessary.
2.  **`ORDER BY` Clause:** The `ORDER BY` clause can only be specified once, at the very end of the entire `INTERSECT` query, and it applies to the final combined result set.
```sql
SELECT FirstName, LastName, JobTitle
FROM Employees
INTERSECT
SELECT FirstName, LastName, JobTitle
FROM Contractors
ORDER BY LastName, FirstName; -- Applies to the entire common result
```
1.  **Alternative Implementations:** While `INTERSECT` is concise, you can often achieve the same result using `INNER JOIN` combined with a `WHERE` clause that checks for equality across all selected columns. However, `INTERSECT` is generally more readable and often optimized well by the query optimizer for this specific task.
```sql
-- Equivalent using INNER JOIN (more verbose)
SELECT E.FirstName, E.LastName, E.JobTitle
FROM Employees AS E
INNER JOIN Contractors AS C
	ON E.FirstName = C.FirstName
	AND E.LastName = C.LastName
	AND E.JobTitle = C.JobTitle;
```
Note that the `INNER JOIN` approach might return duplicates if, for example, 'Alice Smith, Developer' appeared twice in the `Employees` table and once in `Contractors`. `INTERSECT` would still return it only once. To match `INTERSECT`'s behavior exactly with `INNER JOIN`, you'd often need `SELECT DISTINCT`.

2.  **Multiple `INTERSECT` Operations:** You can chain multiple `INTERSECT` operators together to find rows common to three or more result sets.
```sql
SELECT Col1, Col2 FROM TableA
INTERSECT
SELECT Col1, Col2 FROM TableB
INTERSECT
SELECT Col1, Col2 FROM TableC;
```

`INTERSECT` is a powerful and elegant operator for finding commonalities between datasets. Understanding when and how to use it effectively will significantly enhance your ability to write clear, efficient, and accurate SQL queries for complex analytical requirements.