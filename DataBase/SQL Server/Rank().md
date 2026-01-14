### The `RANK()` Function in SQL Server

The `RANK()` function is a window function that assigns a rank to each row within a specified partition of a result set. Similar to `DENSE_RANK()` [[Dense_Rank()]], `RANK()` assigns the **same rank** to rows that have identical values in the `ORDER BY` clause (i.e., ties). However, unlike `DENSE_RANK()`, `RANK()` then **skips subsequent ranks** after a tie. This means there can be gaps in the ranking sequence.

**Purpose:**
*   To assign ranks to rows where ties should receive the same rank, and subsequent ranks should reflect the actual count of rows preceding them (thus creating gaps).
*   To find the "top N" items where you want to acknowledge ties but also reflect the true position of the next non-tied item.
*   To analyze data where the absolute position *after* ties is important.

**Syntax:**
```sql
RANK() OVER (
    [PARTITION BY column1, column2, ...]
    ORDER BY column3 [ASC|DESC], column4 [ASC|DESC], ...
)
```

As with other ranking functions, the `OVER` clause defines the window, `PARTITION BY` groups the rows, and `ORDER BY` specifies the sorting within each group.

Let's use our augmented `Employees` table again to clearly demonstrate `RANK()`'s behavior, especially with ties:

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

### Examples of `RANK()` Usage

#### 1. Simple `RANK()` over the Entire Result Set

Let's assign a rank to all employees based on their salary in descending order.

```sql
SELECT
    EmployeeID,
    FirstName,
    LastName,
    Salary,
    RANK() OVER (ORDER BY Salary DESC) AS GlobalRank
FROM
    Employees
ORDER BY
    Salary DESC, EmployeeID ASC; -- Added EmployeeID for consistent tie-breaking in display
```

**Result (ordered by Salary DESC, EmployeeID ASC):**

| EmployeeID | FirstName | LastName | Salary | GlobalRank |                       |
| ---------- | --------- | -------- | ------ | ---------- | --------------------- |
| 107        | Grace     | Hopper   | 95000  | 1          |                       |
| 106        | Frank     | Green    | 90000  | 2          |                       |
| 102        | Bob       | Johnson  | 85000  | 3          |                       |
| 110        | John      | Doe      | 85000  | 3          |                       |
| 103        | Carol     | Davis    | 72000  | 5          | -- Rank 4 is skipped! |
| 109        | Anna      | Anderson | 71000  | 6          |                       |
| 101        | Alice     | Smith    | 70000  | 7          |                       |
| 108        | Charlie   | Chaplin  | 78000  | 8          |                       |
| 105        | Eve       | White    | 65000  | 9          |                       |
| 104        | David     | Brown    | 60000  | 10         |                       |

**Explanation:**
*   Grace Hopper (95000) gets `GlobalRank` 1.
*   Frank Green (90000) gets `GlobalRank` 2.
*   Bob Johnson (85000) and John Doe (85000) have the same salary, so they both get `GlobalRank` 3.
*   **Crucially, because two rows received rank 3, the next distinct salary (72000 for Carol Davis) receives `GlobalRank` 5.** Rank 4 is skipped. This is the defining characteristic of `RANK()`.

#### 2. `RANK()` with `PARTITION BY`

Let's find the rank of employees within each department based on their salary.

**Example:** Assign a rank to employees within each department based on salary (highest first).

```sql
SELECT
    EmployeeID,
    FirstName,
    LastName,
    DepartmentID,
    Salary,
    RANK() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) AS DepartmentRank
FROM
    Employees
ORDER BY
    DepartmentID, Salary DESC, EmployeeID ASC;
```

**Result (partial, showing relevant columns and ordering):**

| EmployeeID | FirstName | LastName | DepartmentID | Salary | DepartmentRank |                       |
| ---------- | --------- | -------- | ------------ | ------ | -------------- | --------------------- |
| 103        | Carol     | Davis    | 1            | 72000  | 1              |                       |
| 109        | Anna      | Anderson | 1            | 71000  | 2              |                       |
| 101        | Alice     | Smith    | 1            | 70000  | 3              |                       |
| 102        | Bob       | Johnson  | 2            | 85000  | 1              |                       |
| 110        | John      | Doe      | 2            | 85000  | 1              |                       |
| 108        | Charlie   | Chaplin  | 2            | 78000  | 3              | -- Rank 2 is skipped! |
| 106        | Frank     | Green    | 3            | 90000  | 1              |                       |
| 107        | Grace     | Hopper   | 5            | 95000  | 1              |                       |
| 104        | David     | Brown    | 6            | 60000  | 1              |                       |
| 105        | Eve       | White    | NULL         | 65000  | 1              |                       |

**Explanation:**
*   Within `DepartmentID` 1: Carol (72000) is rank 1, Anna (71000) is rank 2, Alice (70000) is rank 3. No ties, so no gaps.
*   Within `DepartmentID` 2: Bob (85000) and John (85000) both get `DepartmentRank` 1. Because two rows received rank 1, the next distinct salary (78000 for Charlie) receives `DepartmentRank` 3. Rank 2 is skipped.

#### 3. Using `RANK()` for "Top N per Group" (with tie consideration)

You can use `RANK()` in a CTE or subquery to filter results, especially when you want to include all tied rows at a certain rank, and the subsequent rank should reflect the skipped positions.

**Example:** Retrieve the top 2 highest-paid employees from each department, including all ties at the 2nd position.

```sql
WITH RankedEmployees AS (
    SELECT
        EmployeeID,
        FirstName,
        LastName,
        DepartmentID,
        Salary,
        RANK() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) AS DepartmentRank
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
    RankedEmployees
WHERE
    DepartmentRank <= 2
ORDER BY
    DepartmentID, DepartmentRank, EmployeeID ASC;
```

**Result:**

| EmployeeID | FirstName | LastName | DepartmentID | Salary |
|------------|-----------|----------|--------------|--------|
| 103        | Carol     | Davis    | 1            | 72000  |
| 109        | Anna      | Anderson | 1            | 71000  |
| 102        | Bob       | Johnson  | 2            | 85000  |
| 110        | John      | Doe      | 2            | 85000  |
| 106        | Frank     | Green    | 3            | 90000  |
| 107        | Grace     | Hopper   | 5            | 95000  |
| 104        | David     | Brown    | 6            | 60000  |
| 105        | Eve       | White    | NULL         | 65000  |

**Explanation:**
*   For `DepartmentID` 1, Carol (rank 1) and Anna (rank 2) are included.
*   For `DepartmentID` 2, both Bob and John (salary 85000) are rank 1. Charlie (salary 78000) is rank 3. Since we filtered for `DepartmentRank <= 2`, only Bob and John are included. If Charlie's salary was tied with Anna's in Department 1, he would also be included if his rank was 2.

### Key Differences from `ROW_NUMBER()` and `DENSE_RANK()`

This table summarizes the behavior of the three primary ranking functions:

| Value | ROW_NUMBER() | RANK() | DENSE_RANK() |
|-------|--------------|--------|--------------|
| 100   | 1            | 1      | 1            |
| 90    | 2            | 2      | 2            |
| 80    | 3            | 3      | 3            |
| 80    | 4            | 3      | 3            |
| 80    | 5            | 3      | 3            |
| 70    | 6            | 6      | 4            |
| 60    | 7            | 7      | 5            |

*   **`ROW_NUMBER()`**: Unique, consecutive integers. No ties, no gaps.
*   **`RANK()`**: Same rank for ties, but skips subsequent ranks (creates gaps).
*   **`DENSE_RANK()`**: Same rank for ties, and next rank is consecutive (no gaps).

### Important Considerations and Best Practices

1.  **`ORDER BY` is Mandatory:** Like all ranking functions, `RANK()` requires an `ORDER BY` clause within its `OVER()` specification to define the ranking order.
2.  **`PARTITION BY` for Grouped Ranks:** Use `PARTITION BY` when you need to restart the ranking for each distinct group (e.g., ranking employees within each department).
3.  **Performance:** As a window function, `RANK()` can be resource-intensive on large datasets, especially if the `ORDER BY` or `PARTITION BY` columns are not indexed.
4.  **Logical Processing Order:** Remember that window functions are evaluated *after* `WHERE`, `GROUP BY`, and `HAVING` clauses, but *before* the final `SELECT` and `ORDER BY` of the outer query. This means you typically need a CTE or subquery to filter based on the generated rank.
5.  **Choosing the Right Function:**
    *   Use `ROW_NUMBER()` when you need a unique sequential number for every row, regardless of ties (e.g., for pagination or strict deduplication).
    *   Use `DENSE_RANK()` when you want tied rows to have the same rank, and you want the subsequent ranks to be consecutive without gaps (e.g., for "top N distinct scores").
    *   Use `RANK()` when you want tied rows to have the same rank, but you want the next rank to reflect the count of items that precede it, thus creating gaps (e.g., for competitive rankings where ties share a position, and the next person is truly "Nth place").

`RANK()` is a powerful and flexible function for scenarios where the presence of ties and the subsequent numbering of non-tied items are important for your analysis. Understanding its behavior in contrast to `ROW_NUMBER()` and `DENSE_RANK()` is key to selecting the most appropriate ranking strategy for your data.