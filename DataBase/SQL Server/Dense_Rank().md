This function is particularly interesting because of how it handles ties, offering a different perspective on ranking compared to `ROW_NUMBER()` [[Row_Number()]].

### The `DENSE_RANK()` Function in SQL Server

The `DENSE_RANK()` function is a window function that assigns a rank to each row within a specified partition of a result set. Unlike `ROW_NUMBER()`, `DENSE_RANK()` assigns **consecutive ranks** to rows, and if there are ties (rows with the same value in the `ORDER BY` clause), they receive the **same rank**. The next rank assigned after a tie is still consecutive, without any gaps.

**Purpose:**
*   To assign ranks to rows where ties should receive the same rank, and subsequent ranks should be consecutive.
*   To find the "top N distinct values" or "top N groups" within a dataset.
*   To analyze data where the absolute position matters less than the relative rank of distinct values.

**Syntax:**
```sql
DENSE_RANK() OVER (
    [PARTITION BY column1, column2, ...]
    ORDER BY column3 [ASC|DESC], column4 [ASC|DESC], ...
)
```

As with `ROW_NUMBER()`, the `OVER` clause defines the window, `PARTITION BY` groups the rows, and `ORDER BY` specifies the sorting within each group.

Let's use our augmented `Employees` table again to clearly demonstrate `DENSE_RANK()`'s behavior, especially with ties:

**`Employees` Table:**

| EmployeeID | FirstName | LastName | DepartmentID | Salary | HireDate   |                                                                               |
| ---------- | --------- | -------- | ------------ | ------ | ---------- | ----------------------------------------------------------------------------- |
| 101        | Alice     | Smith    | 1            | 70000  | 2020-01-15 |                                                                               |
| 102        | Bob       | Johnson  | 2            | 85000  | 2019-03-10 |                                                                               |
| 103        | Carol     | Davis    | 1            | 72000  | 2020-02-20 |                                                                               |
| 104        | David     | Brown    | 6            | 60000  | 2021-07-01 |                                                                               |
| 105        | Eve       | White    | NULL         | 65000  | 2020-05-12 |                                                                               |
| 106        | Frank     | Green    | 3            | 90000  | 2018-11-01 |                                                                               |
| 107        | Grace     | Hopper   | 5            | 95000  | 2017-09-20 |                                                                               |
| 108        | Charlie   | Chaplin  | 2            | 78000  | 2019-05-01 |                                                                               |
| 109        | Anna      | Anderson | 1            | 71000  | 2020-01-15 |                                                                               |
| 110        | John      | Doe      | 2            | 85000  | 2019-03-10 | -- Bob and John have same salary and hire date (tie for salary and hire date) |

### Examples of `DENSE_RANK()` Usage

#### 1. Simple `DENSE_RANK()` over the Entire Result Set

Let's assign a dense rank to all employees based on their salary in descending order.

```sql
SELECT
    EmployeeID,
    FirstName,
    LastName,
    Salary,
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS DenseRank
FROM
    Employees
ORDER BY
    Salary DESC, EmployeeID ASC; -- Added EmployeeID for consistent tie-breaking in display
```

**Result (ordered by Salary DESC, EmployeeID ASC):**

| EmployeeID | FirstName | LastName | Salary | DenseRank |
|------------|-----------|----------|--------|-----------|
| 107        | Grace     | Hopper   | 95000  | 1         |
| 106        | Frank     | Green    | 90000  | 2         |
| 102        | Bob       | Johnson  | 85000  | 3         |
| 110        | John      | Doe      | 85000  | 3         |
| 103        | Carol     | Davis    | 72000  | 4         |
| 109        | Anna      | Anderson | 71000  | 5         |
| 101        | Alice     | Smith    | 70000  | 6         |
| 108        | Charlie   | Chaplin  | 78000  | 7         |
| 105        | Eve       | White    | 65000  | 8         |
| 104        | David     | Brown    | 60000  | 9         |

**Explanation:**
*   Grace Hopper (95000) gets `DenseRank` 1.
*   Frank Green (90000) gets `DenseRank` 2.
*   Bob Johnson (85000) and John Doe (85000) have the same salary, so they both get `DenseRank` 3.
*   Crucially, the next distinct salary (72000 for Carol Davis) receives `DenseRank` 4. There is no gap in the ranking sequence, unlike what you might see with `RANK()` [[Rank()]].

#### 2. `DENSE_RANK()` with `PARTITION BY`

Let's find the dense rank of employees within each department based on their salary.

**Example:** Assign a dense rank to employees within each department based on salary (highest first).

```sql
SELECT
    EmployeeID,
    FirstName,
    LastName,
    DepartmentID,
    Salary,
    DENSE_RANK() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) AS DepartmentDenseRank
FROM
    Employees
ORDER BY
    DepartmentID, Salary DESC, EmployeeID ASC;
```

**Result (partial, showing relevant columns and ordering):**

| EmployeeID | FirstName | LastName | DepartmentID | Salary | DepartmentDenseRank |
|------------|-----------|----------|--------------|--------|---------------------|
| 103        | Carol     | Davis    | 1            | 72000  | 1                   |
| 109        | Anna      | Anderson | 1            | 71000  | 2                   |
| 101        | Alice     | Smith    | 1            | 70000  | 3                   |
| 102        | Bob       | Johnson  | 2            | 85000  | 1                   |
| 110        | John      | Doe      | 2            | 85000  | 1                   |
| 108        | Charlie   | Chaplin  | 2            | 78000  | 2                   |
| 106        | Frank     | Green    | 3            | 90000  | 1                   |
| 107        | Grace     | Hopper   | 5            | 95000  | 1                   |
| 104        | David     | Brown    | 6            | 60000  | 1                   |
| 105        | Eve       | White    | NULL         | 65000  | 1                   |

**Explanation:**
*   Within `DepartmentID` 1: Carol (72000) is rank 1, Anna (71000) is rank 2, Alice (70000) is rank 3.
*   Within `DepartmentID` 2: Bob (85000) and John (85000) both get `DepartmentDenseRank` 1. Charlie (78000) then gets `DepartmentDenseRank` 2, maintaining consecutive ranks.

#### 3. Using `DENSE_RANK()` for "Top N Distinct Values per Group"

Similar to `ROW_NUMBER()`, `DENSE_RANK()` is often used in a CTE or subquery to filter results.

**Example:** Retrieve the top 2 highest *distinct salary levels* from each department.

```sql
WITH RankedSalaries AS (
    SELECT
        EmployeeID,
        FirstName,
        LastName,
        DepartmentID,
        Salary,
        DENSE_RANK() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) AS SalaryDenseRank
    FROM
        Employees
)
SELECT
    EmployeeID,
    FirstName,
    LastName,
    DepartmentID,
    Salary
FROM
    RankedSalaries
WHERE
    SalaryDenseRank <= 2
ORDER BY
    DepartmentID, SalaryDenseRank, EmployeeID ASC;
```

**Result:**

| EmployeeID | FirstName | LastName | DepartmentID | Salary |
|------------|-----------|----------|--------------|--------|
| 103        | Carol     | Davis    | 1            | 72000  |
| 109        | Anna      | Anderson | 1            | 71000  |
| 102        | Bob       | Johnson  | 2            | 85000  |
| 110        | John      | Doe      | 2            | 85000  |
| 108        | Charlie   | Chaplin  | 2            | 78000  |
| 106        | Frank     | Green    | 3            | 90000  |
| 107        | Grace     | Hopper   | 5            | 95000  |
| 104        | David     | Brown    | 6            | 60000  |
| 105        | Eve       | White    | NULL         | 65000  |

**Explanation:**
*   For `DepartmentID` 1, we get Carol (rank 1) and Anna (rank 2).
*   For `DepartmentID` 2, both Bob and John (salary 85000) are rank 1. Charlie (salary 78000) is rank 2. All three are included because they represent the top two *distinct salary levels*.

### Key Differences from `ROW_NUMBER()` and `RANK()`

This is the critical part for understanding `DENSE_RANK()`:

*   **`ROW_NUMBER()`**: Assigns unique, consecutive integers. Ties get different numbers (e.g., 1, 2, 3, 4 for four tied rows).
*   **`RANK()` (to be discussed next)**: Assigns the same rank to tied rows, but then skips ranks. For four tied rows, it would be 1, 1, 1, 1, and the next rank would be 5.
*   **`DENSE_RANK()`**: Assigns the same rank to tied rows, and the next rank is *consecutive* (no gaps). For four tied rows, it would be 1, 1, 1, 1, and the next rank would be 2.

| Value | ROW_NUMBER() | RANK() | DENSE_RANK() |
|-------|--------------|--------|--------------|
| 100   | 1            | 1      | 1            |
| 90    | 2            | 2      | 2            |
| 80    | 3            | 3      | 3            |
| 80    | 4            | 3      | 3            |
| 80    | 5            | 3      | 3            |
| 70    | 6            | 6      | 4            |
| 60    | 7            | 7      | 5            |

### Important Considerations and Best Practices

1.  **`ORDER BY` is Mandatory:** Just like `ROW_NUMBER()`, `DENSE_RANK()` requires an `ORDER BY` clause within its `OVER()` specification to define the ranking order.
2.  **`PARTITION BY` for Grouped Ranks:** Use `PARTITION BY` when you need to restart the ranking for each distinct group (e.g., ranking employees within each department).
3.  **Performance:** As a window function, `DENSE_RANK()` can be resource-intensive on large datasets, especially if the `ORDER BY` or `PARTITION BY` columns are not indexed.
4.  **Logical Processing Order:** Remember that window functions are evaluated *after* `WHERE`, `GROUP BY`, and `HAVING` clauses, but *before* the final `SELECT` and `ORDER BY` of the outer query. This means you typically need a CTE or subquery to filter based on the generated rank.

`DENSE_RANK()` is an invaluable function for scenarios where you need to understand the relative standing of items, particularly when ties are common and you want a continuous sequence of ranks without gaps. It's a powerful tool for analytical queries and data segmentation.

Next, we'll explore `RANK()` to complete our understanding of how SQL Server handles ties in ranking!