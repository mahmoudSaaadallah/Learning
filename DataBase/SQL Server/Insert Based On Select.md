### The `INSERT INTO SELECT` Statement in SQL Server: Populating Existing Tables

The `INSERT INTO SELECT` statement is used to copy data from one or more source tables into an **existing** destination table. Unlike `SELECT INTO` [[Select Into]], which creates a *new* table, `INSERT INTO SELECT` is designed to add rows to a table that already exists in your database.

**Purpose:**
*   To populate a new, empty table with initial data from another source.
*   To archive old data from an active table into an archive table.
*   To move data between tables, often after some transformation or filtering.
*   To combine data from multiple sources into a single reporting or staging table.

**Basic Syntax:**
```sql
INSERT INTO TargetTableName (Column1, Column2, ...)
SELECT SourceColumn1, SourceColumn2, ...
FROM SourceTableName
[WHERE condition]
[GROUP BY column_list]
[HAVING condition];
```
Or, to insert all columns (if the column order and count match exactly):
```sql
INSERT INTO TargetTableName
SELECT *
FROM SourceTableName
[WHERE condition];
```

Let's use our familiar `Employees` and `Departments` tables, and imagine we have a new table called `NewHires` or an archive table `OldEmployees`.

**`Employees` Table:**

| EmployeeID | FirstName | LastName | DepartmentID | Salary |
|------------|-----------|----------|--------------|--------|
| 101        | Alice     | Smith    | 1            | 70000  |
| 102        | Bob       | Johnson  | 2            | 85000  |
| 103        | Carol     | Davis    | 1            | 72000  |
| 104        | David     | Brown    | 6            | 60000  |
| 105        | Eve       | White    | NULL         | 65000  |
| 106        | Frank     | Green    | 3            | 90000  |

**`Departments` Table:**
	
| DepartmentID | DepartmentName         |
|--------------|------------------------|
| 1            | Human Resources        |
| 2            | Information Technology |
| 3            | Sales                  |
| 4            | Marketing              |
| 5            | Research               |

### Key Differences from `SELECT INTO`

This is a critical distinction:

*   **`SELECT INTO`**: **Creates a new table** and inserts data. The target table must *not* exist.
*   **`INSERT INTO SELECT`**: **Inserts data into an existing table**. The target table must *already* exist.

### Examples of `INSERT INTO SELECT`

#### 1. Basic Insertion from a Single Table

Let's assume we have an existing table `NewHires` with the same schema as `Employees`, and we want to add some employees to it.

```sql
-- First, ensure the target table exists (this would be done once)
CREATE TABLE NewHires (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    DepartmentID INT,
    Salary DECIMAL(10, 2)
);

-- Now, insert data into NewHires from Employees
INSERT INTO NewHires (EmployeeID, FirstName, LastName, DepartmentID, Salary)
SELECT EmployeeID, FirstName, LastName, DepartmentID, Salary
FROM Employees
WHERE EmployeeID IN (101, 103); -- Just a subset for demonstration
```
**Explanation:** This copies the specified columns for employees 101 and 103 from the `Employees` table into the `NewHires` table.

#### 2. Inserting Specific Columns

You can choose to insert only a subset of columns, provided the data types are compatible.

```sql
-- Assume a table for basic employee info exists
CREATE TABLE EmployeeBasicInfo (
    EmployeeID INT PRIMARY KEY,
    FullName VARCHAR(100)
);

-- Insert only ID and a concatenated full name
INSERT INTO EmployeeBasicInfo (EmployeeID, FullName)
SELECT
    EmployeeID,
    CONCAT(FirstName, ' ', LastName)
FROM
    Employees;
```
**Explanation:** This populates `EmployeeBasicInfo` with the employee ID and a combined full name, demonstrating data transformation during insertion.

#### 3. Inserting Filtered Data

You can use a `WHERE` clause to filter which rows from the source table(s) are inserted.

```sql
-- Assume an archive table exists
CREATE TABLE OldEmployees (
    EmployeeID INT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Salary DECIMAL(10, 2)
);

-- Archive employees with salary less than 70000
INSERT INTO OldEmployees (EmployeeID, FirstName, LastName, Salary)
SELECT EmployeeID, FirstName, LastName, Salary
FROM Employees
WHERE Salary < 70000;
```
**Explanation:** Only employees with a salary less than $70,000 are copied to the `OldEmployees` archive table.

#### 4. Using Joins with `INSERT INTO SELECT`

This is very common for combining data from multiple tables before inserting it into a single destination.

```sql
-- Assume a reporting table exists
CREATE TABLE DepartmentEmployeeReport (
    DepartmentName VARCHAR(100),
    EmployeeFirstName VARCHAR(50),
    EmployeeLastName VARCHAR(50),
    EmployeeSalary DECIMAL(10, 2)
);

-- Populate the report table with joined data
INSERT INTO DepartmentEmployeeReport (DepartmentName, EmployeeFirstName, EmployeeLastName, EmployeeSalary)
SELECT
    D.DepartmentName,
    E.FirstName,
    E.LastName,
    E.Salary
FROM
    Employees AS E
INNER JOIN
    Departments AS D ON E.DepartmentID = D.DepartmentID
WHERE
    D.DepartmentName = 'Human Resources';
```
**Explanation:** This query joins `Employees` and `Departments` to get the department name alongside employee details, then inserts only the Human Resources employees into the `DepartmentEmployeeReport` table.

#### 5. Using Aggregate Functions with `INSERT INTO SELECT`

You can insert summary data into a table by using `GROUP BY` and aggregate functions in your `SELECT` statement.

```sql
-- Assume a summary table exists
CREATE TABLE DepartmentSalaryStats (
    DepartmentName VARCHAR(100) PRIMARY KEY,
    TotalSalary DECIMAL(18, 2),
    AverageSalary DECIMAL(18, 2),
    NumberOfEmployees INT
);

-- Populate the summary table
INSERT INTO DepartmentSalaryStats (DepartmentName, TotalSalary, AverageSalary, NumberOfEmployees)
SELECT
    D.DepartmentName,
    SUM(E.Salary),
    AVG(E.Salary),
    COUNT(E.EmployeeID)
FROM
    Employees AS E
INNER JOIN
    Departments AS D ON E.DepartmentID = D.DepartmentID
GROUP BY
    D.DepartmentName;
```
**Explanation:** This creates a summary of total, average, and count of employees per department and inserts it into `DepartmentSalaryStats`.

### Important Considerations and Best Practices

1.  **Target Table Must Exist:** This is the most crucial rule. If `TargetTableName` does not exist, the statement will fail. Use `CREATE TABLE` first.
2.  **Column Count and Order:**
    *   If you specify column names in the `INSERT INTO` clause (e.g., `INSERT INTO TargetTable (Col1, Col2)`), the number and order of columns in the `SELECT` list must match these specified columns.
    *   If you omit the column list in `INSERT INTO` (e.g., `INSERT INTO TargetTable SELECT *`), then the number and order of columns in the `SELECT` list *must exactly match* the number and order of columns in the `TargetTableName`.
3.  **Data Type Compatibility:** The data types of the source columns in the `SELECT` statement must be compatible with the data types of the corresponding columns in the `TargetTableName`. Implicit conversions will occur if possible, but explicit `CAST()` or `CONVERT()` is recommended for clarity and to prevent errors.
4.  **Constraints:** `INSERT INTO SELECT` respects all constraints (Primary Key, Foreign Key, NOT NULL, CHECK, UNIQUE) on the `TargetTableName`. If the inserted data violates any constraint, the entire `INSERT` operation will fail.
5.  **Identity Columns:** If the target table has an `IDENTITY` column, you typically omit it from the `INSERT INTO` column list, and SQL Server will automatically generate values. If you need to explicitly insert values into an `IDENTITY` column, you must use `SET IDENTITY_INSERT TargetTableName ON;` before the `INSERT` statement and `SET IDENTITY_INSERT TargetTableName OFF;` afterwards.
6.  **Performance:** For large datasets, `INSERT INTO SELECT` is generally efficient. Ensure that the `SELECT` query itself is optimized (e.g., using appropriate indexes on joined and filtered columns).
7.  **Transaction Control:** Always wrap `INSERT` statements, especially those affecting many rows, in a transaction (`BEGIN TRAN`, `COMMIT TRAN`, `ROLLBACK TRAN`) during development and testing. This allows you to verify the results and undo changes if necessary.

The `INSERT INTO SELECT` statement is a powerful and flexible tool for managing data flow within your SQL Server environment. Understanding its proper usage and considerations is vital for maintaining data integrity and building robust database solutions.