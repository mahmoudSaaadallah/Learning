### The Essence of Joins

In a relational database, data is often distributed across multiple tables to ensure data integrity, reduce redundancy, and optimize storage – a concept known as normalization [[Normalization]]. However, to answer meaningful business questions, we frequently need to combine related data from these separate tables. This is precisely where **Joins** come into play.

A `JOIN` clause is used to combine rows from two or more tables based on a related column between them. It allows us to reconstruct a unified view of data that is logically connected but physically separated.

To illustrate, let's consider two simple tables:

**1. `Departments` Table:**

| DepartmentID | DepartmentName |
|--------------|----------------|
| 1            | Human Resources|
| 2            | Information Technology|
| 3            | Sales          |
| 4            | Marketing      |
| 5            | Research       |

**2. `Employees` Table:**

| EmployeeID | FirstName | LastName | DepartmentID | Salary |
|------------|-----------|----------|--------------|--------|
| 101        | Alice     | Smith    | 1            | 70000  |
| 102        | Bob       | Johnson  | 2            | 85000  |
| 103        | Carol     | Davis    | 1            | 72000  |
| 104        | David     | Brown    | 6            | 60000  | -- Department 6 does not exist
| 105        | Eve       | White    | NULL         | 65000  | -- No department assigned
| 106        | Frank     | Green    | 3            | 90000  |

Notice the `DepartmentID` column in both tables. This is our common link, often referred to as a foreign key in the `Employees` table referencing the primary key in the `Departments` table.

### Types of Joins in SQL Server

SQL Server supports several types of joins, each designed for specific data retrieval requirements. They can be broadly categorized into `INNER JOIN`, `OUTER JOIN` (Left, Right, Full), `CROSS JOIN`, and `SELF JOIN`.

#### 1. INNER JOIN

The `INNER JOIN` is the most common type of join. It returns only the rows that have matching values in **both** tables. If a row in one table does not have a corresponding match in the other table based on the join condition, it is excluded from the result set.

**Purpose:** To retrieve records where a match exists in both joined tables.

**Syntax:**
```sql
SELECT column_list
FROM TableA
INNER JOIN TableB
ON TableA.matching_column = TableB.matching_column;
```

**Example:** Retrieve employees along with their department names.

```sql
SELECT
    E.FirstName,
    E.LastName,
    D.DepartmentName
FROM
    Employees AS E
INNER JOIN
    Departments AS D ON E.DepartmentID = D.DepartmentID;
```
The previous query could be written in different way to make it as more close to other databases providers like `mysql`, and others.
```sql
SELECT
    E.FirstName,
    E.LastName,
    D.DepartmentName
FROM
    Employees AS E, Departments AS D 
where E.DepartmentID = D.DepartmentID;
```
- So here in those two different ways to write the same query we could notice that when using the word `inner join` then we have to use `on` for the condition, and if we sperate the tables with `,` comma, then we have to use `where` for the condition.

**Result:**

| FirstName | LastName | DepartmentName    |
|-----------|----------|-------------------|
| Alice     | Smith    | Human Resources   |
| Bob       | Johnson  | Information Technology|
| Carol     | Davis    | Human Resources   |
| Frank     | Green    | Sales             |

**Explanation:**
*   Alice, Bob, Carol, and Frank all have valid `DepartmentID`s (1, 2, 1, 3 respectively) that exist in the `Departments` table.
*   David (EmployeeID 104) is excluded because `DepartmentID` 6 does not exist in the `Departments` table.
*   Eve (EmployeeID 105) is excluded because her `DepartmentID` is `NULL`, which doesn't match any `DepartmentID` in the `Departments` table.
*   The 'Marketing' (ID 4) and 'Research' (ID 5) departments are also excluded because no employees are currently assigned to them.

#### 2. OUTER JOINS

Outer joins are used when you want to include rows from one or both tables even if they don't have a match in the other table. There are three types: `LEFT OUTER JOIN`, `RIGHT OUTER JOIN`, and `FULL OUTER JOIN`. The `OUTER` keyword is optional but often used for clarity.

##### a. LEFT OUTER JOIN (or LEFT JOIN)

A `LEFT JOIN` returns all rows from the **left table** (the first table in the `FROM` clause) and the matching rows from the right table. If there's no match for a row in the left table, the columns from the right table will have `NULL` values.

**Purpose:** To see all records from the "primary" table and any related records from the "secondary" table.

**Syntax:**
```sql
SELECT column_list
FROM TableA
LEFT JOIN TableB
ON TableA.matching_column = TableB.matching_column;
```

**Example:** Retrieve all employees and their department names, including employees without an assigned department or with a non-existent department.

```sql
SELECT
    E.FirstName,
    E.LastName,
    D.DepartmentName
FROM
    Employees AS E
LEFT JOIN
    Departments AS D ON E.DepartmentID = D.DepartmentID;
```

**Result:**

| FirstName | LastName | DepartmentName         |
| --------- | -------- | ---------------------- |
| Alice     | Smith    | Human Resources        |
| Bob       | Johnson  | Information Technology |
| Carol     | Davis    | Human Resources        |
| David     | Brown    | NULL                   |
| Eve       | White    | NULL                   |
| Frank     | Green    | Sales                  |

**Explanation:**
*   All employees from the `Employees` table (the left table) are included.
*   David and Eve, who had no matching department, now appear in the result set, with `NULL` in the `DepartmentName` column.
*   Departments like 'Marketing' and 'Research' still do not appear because they are not linked to any employee, and `Departments` is the right table.

##### b. RIGHT OUTER JOIN (or RIGHT JOIN)

A `RIGHT JOIN` is the mirror image of a `LEFT JOIN`. It returns all rows from the **right table** and the matching rows from the left table. If there's no match for a row in the right table, the columns from the left table will have `NULL` values.

**Purpose:** To see all records from the "secondary" table and any related records from the "primary" table.

**Syntax:**
```sql
SELECT column_list
FROM TableA
RIGHT JOIN TableB
ON TableA.matching_column = TableB.matching_column;
```

**Example:** Retrieve all departments and the employees assigned to them, including departments with no employees.

```sql
SELECT
    D.DepartmentName,
    E.FirstName,
    E.LastName
FROM
    Employees AS E
RIGHT JOIN
    Departments AS D ON E.DepartmentID = D.DepartmentID;
```

**Result:**

| DepartmentName    | FirstName | LastName |
|-------------------|-----------|----------|
| Human Resources   | Alice     | Smith    |
| Human Resources   | Carol     | Davis    |
| Information Technology| Bob       | Johnson  |
| Sales             | Frank     | Green    |
| Marketing         | NULL      | NULL     |
| Research          | NULL      | NULL     |

**Explanation:**
*   All departments from the `Departments` table (the right table) are included.
*   'Marketing' and 'Research' departments, which had no employees, now appear with `NULL` values for `FirstName` and `LastName`.
*   Employees David and Eve, who had no matching department, are excluded because they don't have a corresponding `DepartmentID` in the `Departments` table (the right table).

##### c. FULL OUTER JOIN (or FULL JOIN)

A `FULL JOIN` returns all rows when there is a match in either the left or the right table. It combines the results of both `LEFT JOIN` and `RIGHT JOIN`. If a row in the left table has no match in the right table, the right-side columns are `NULL`. Conversely, if a row in the right table has no match in the left table, the left-side columns are `NULL`.

**Purpose:** To see all records from both tables, showing where matches exist and where they don't.

**Syntax:**
```sql
SELECT column_list
FROM TableA
FULL JOIN TableB
ON TableA.matching_column = TableB.matching_column;
```

**Example:** Retrieve all employees and all departments, showing where they match and where they don't.

```sql
SELECT
    E.FirstName,
    E.LastName,
    D.DepartmentName
FROM
    Employees AS E
FULL JOIN
    Departments AS D ON E.DepartmentID = D.DepartmentID;
```

**Result:**

| FirstName | LastName | DepartmentName    |
|-----------|----------|-------------------|
| Alice     | Smith    | Human Resources   |
| Bob       | Johnson  | Information Technology|
| Carol     | Davis    | Human Resources   |
| Frank     | Green    | Sales             |
| David     | Brown    | NULL              |
| Eve       | White    | NULL              |
| NULL      | NULL     | Marketing         |
| NULL      | NULL     | Research          |

**Explanation:**
*   This result set includes all employees (even those without a valid department) and all departments (even those without employees).
*   David and Eve appear with `NULL` for `DepartmentName`.
*   'Marketing' and 'Research' departments appear with `NULL` for `FirstName` and `LastName`.

#### 3. CROSS JOIN

A `CROSS JOIN` produces a Cartesian product of the two tables involved. This means every row from the first table is combined with every row from the second table. It does not require a join condition (`ON` clause).

**Purpose:** To generate all possible combinations of rows between two tables. This is rarely used in typical data retrieval but can be useful for generating test data or specific statistical analyses.

**Syntax:**
```sql
SELECT column_list
FROM TableA
CROSS JOIN TableB;
```

**Example:** Combine every employee with every department.

```sql
SELECT
    E.FirstName,
    D.DepartmentName
FROM
    Employees AS E
CROSS JOIN
    Departments AS D;
```

**Result (partial, as it would be 6 employees * 5 departments = 30 rows):**

| FirstName | DepartmentName    |
|-----------|-------------------|
| Alice     | Human Resources   |
| Alice     | Information Technology|
| Alice     | Sales             |
| Alice     | Marketing         |
| Alice     | Research          |
| Bob       | Human Resources   |
| ...       | ...               |
| Frank     | Research          |

**Explanation:**
*   Each of the 6 employees is paired with each of the 5 departments, resulting in 30 rows.
*   There's no logical connection implied; it's purely a combinatorial output.

#### 4. SELF JOIN

A `SELF JOIN` is not a distinct type of join keyword like `INNER` or `LEFT`. Instead, it's a conceptual join where a table is joined with itself. This is useful when you need to compare rows within the same table, treating it as if it were two separate tables. To perform a self-join, you must use table aliases to distinguish between the two instances of the table.

**Purpose:** To compare rows within the same table, find hierarchical relationships, or identify duplicates.

**Syntax:**
```sql
SELECT column_list
FROM TableA AS A1
INNER JOIN TableA AS A2
ON A1.column = A2.column;
```

**Example:** Find employees who earn more than their direct manager (assuming `Employees` table has a `ManagerID` column that references `EmployeeID`). Let's augment our `Employees` table for this example:

**Augmented `Employees` Table:**

| EmployeeID | FirstName | LastName | DepartmentID | Salary | ManagerID |                               |
| ---------- | --------- | -------- | ------------ | ------ | --------- | ----------------------------- |
| 101        | Alice     | Smith    | 1            | 70000  | 102       |                               |
| 102        | Bob       | Johnson  | 2            | 85000  | NULL      | -- Bob is a top-level manager |
| 103        | Carol     | Davis    | 1            | 72000  | 101       |                               |
| 104        | David     | Brown    | 6            | 60000  | 102       |                               |
| 105        | Eve       | White    | NULL         | 65000  | 101       |                               |
| 106        | Frank     | Green    | 3            | 90000  | 102       |                               |

```sql
SELECT
    E.FirstName AS EmployeeFirstName,
    E.LastName AS EmployeeLastName,
    E.Salary AS EmployeeSalary,
    M.FirstName AS ManagerFirstName,
    M.LastName AS ManagerLastName,
    M.Salary AS ManagerSalary
FROM
    Employees AS E -- This is the employee instance
INNER JOIN
    Employees AS M -- This is the manager instance
ON
    E.ManagerID = M.EmployeeID
WHERE
    E.Salary > M.Salary;
```

**Result:**

| EmployeeFirstName | EmployeeLastName | EmployeeSalary | ManagerFirstName | ManagerLastName | ManagerSalary |
|-------------------|------------------|----------------|------------------|-----------------|---------------|
| Frank             | Green            | 90000          | Bob              | Johnson         | 85000         |

**Explanation:**
*   We join the `Employees` table to itself, aliasing one instance as `E` (for Employee) and the other as `M` (for Manager).
*   The join condition `E.ManagerID = M.EmployeeID` links each employee to their respective manager.
*   The `WHERE` clause then filters for employees whose salary is greater than their manager's salary.


---
### Scenario: Employees, Departments, and Projects

Imagine we have the following three tables:

**1. `Departments` Table:**

| DepartmentID | DepartmentName         |
| :----------- | :--------------------- |
| 1            | Human Resources        |
| 2            | Information Technology |
| 3            | Sales                  |
| 4            | Marketing              |
| 5            | Research               |

**2. `Employees` Table:**

| EmployeeID | FirstName | LastName | DepartmentID | Salary |
| :--------- | :-------- | :------- | :----------- | :----- |
| 101        | Alice     | Smith    | 1            | 70000  |
| 102        | Bob       | Johnson  | 2            | 85000  |
| 103        | Carol     | Davis    | 1            | 72000  |
| 104        | David     | Brown    | 6            | 60000  | -- Department 6 does not exist
| 105        | Eve       | White    | NULL         | 65000  | -- No department assigned
| 106        | Frank     | Green    | 3            | 90000  |

**3. `Projects` Table:**
This table stores information about projects, and each project is associated with a `DepartmentID`.

| ProjectID | ProjectName     | DepartmentID |
| :-------- | :-------------- | :----------- |
| 1         | HR Portal       | 1            |
| 2         | New CRM System  | 3            |
| 3         | Data Migration  | 2            |
| 4         | Mobile App Dev  | 2            |
| 5         | Internal Tool   | NULL         | -- Not assigned to a specific department yet

---

### Example: Multi-Level Join Query

Let's say you want to retrieve a list of all employees, their department names, and any projects that are associated with their department. We'll use `LEFT JOIN`s to ensure that all employees and departments are included, even if there's no matching project.

```sql
SELECT
    E.FirstName,
    E.LastName,
    D.DepartmentName,
    P.ProjectName
FROM
    Employees AS E LEFT JOIN Departments AS D
    ON E.DepartmentID = D.DepartmentID
	LEFT JOIN Projects AS P 
	ON D.DepartmentID = P.DepartmentID;
```

**Explanation of the Query:**

1.  **`FROM Employees AS E`**: We start with the `Employees` table, aliased as `E`, as our primary table.
2.  **`LEFT JOIN Departments AS D ON E.DepartmentID = D.DepartmentID`**:
    *   This is the first join, connecting `Employees` to `Departments`.
    *   It links employees to their respective departments using the `DepartmentID` column.
    *   Since it's a `LEFT JOIN`, all employees will be included in the result, even if their `DepartmentID` doesn't exist in the `Departments` table (like David) or is `NULL` (like Eve). For these employees, `DepartmentName` will be `NULL`.
3.  **`LEFT JOIN Projects AS P ON D.DepartmentID = P.DepartmentID`**:
    *   This is the second join, connecting the *result of the first join* (which now includes employee and department info) to the `Projects` table, aliased as `P`.
    *   It links departments to their associated projects using the `DepartmentID` column.
    *   Again, a `LEFT JOIN` ensures that all department information (from the previous join's result) is retained, even if a department has no projects. If a department has multiple projects, the employee and department information will be duplicated for each project. If a department has no projects, `ProjectName` will be `NULL`.

### Expected Result:

| FirstName | LastName | DepartmentName         | ProjectName        |
| :-------- | :------- | :--------------------- | :----------------- |
| Alice     | Smith    | Human Resources        | HR Portal          |
| Bob       | Johnson  | Information Technology | Data Migration     |
| Bob       | Johnson  | Information Technology | Mobile App Dev     |
| Carol     | Davis    | Human Resources        | HR Portal          |
| David     | Brown    | NULL                   | NULL               |
| Eve       | White    | NULL                   | NULL               |
| Frank     | Green    | Sales                  | New CRM System     |

**Analysis of the Result:**

*   **Alice (EmployeeID 101)**: Works in Human Resources (ID 1), which has the 'HR Portal' project.
*   **Bob (EmployeeID 102)**: Works in Information Technology (ID 2), which has two projects: 'Data Migration' and 'Mobile App Dev'. Bob's information is repeated for each project.
*   **Carol (EmployeeID 103)**: Works in Human Resources (ID 1), which has the 'HR Portal' project.
*   **David (EmployeeID 104)**: Has `DepartmentID` 6, which doesn't exist. So, `DepartmentName` and `ProjectName` are `NULL`.
*   **Eve (EmployeeID 105)**: Has `DepartmentID` `NULL`. So, `DepartmentName` and `ProjectName` are `NULL`.
*   **Frank (EmployeeID 106)**: Works in Sales (ID 3), which has the 'New CRM System' project.

This example demonstrates how you can progressively join tables to gather related data from different parts of your database schema. The choice of `INNER` vs. `OUTER` joins at each step is critical for determining which rows are included or excluded based on matching conditions.

---
### Choosing the Right Join

The choice of join type is crucial and depends entirely on the specific data you need to retrieve:

*   **`INNER JOIN`**: When you only want records that have a match in *both* tables. This is the most restrictive and often the most performant.
*   **`LEFT JOIN`**: When you want *all* records from the left table, regardless of whether they have a match in the right table. Useful for finding "unmatched" records in the right table (e.g., `WHERE RightTable.ID IS NULL`).
*   **`RIGHT JOIN`**: When you want *all* records from the right table, regardless of whether they have a match in the left table. Similar to `LEFT JOIN` but with the table roles reversed.
*   **`FULL JOIN`**: When you want *all* records from *both* tables, showing where matches exist and where they don't. Useful for comprehensive analysis of relationships.
*   **`CROSS JOIN`**: Rarely used for typical queries; primarily for generating all possible combinations.
*   **`SELF JOIN`**: When you need to compare or relate rows within the same table.

Understanding these distinctions is fundamental for any database developer. Mastering joins allows you to efficiently navigate complex data models and extract precisely the information required for business intelligence, reporting, and application logic. Always consider the cardinality of your tables and the specific question you're trying to answer when selecting your join type.