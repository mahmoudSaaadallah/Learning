### The `SELECT` Statement: The Art of Data Retrieval

The `SELECT` statement is used to retrieve data from one or more tables in a database. It allows you to specify which columns to retrieve, which rows to filter, how to sort the results, and even how to aggregate data.

#### Basic Syntax

The most basic form is:

```sql
SELECT column1, column2, ...
FROM table_name;
```
Or, to retrieve all columns:

```sql
SELECT *
FROM table_name;
```

#### Our Sample Database Schema

To make our examples concrete, let's imagine we have the following tables:

**1. `Employees` Table:**
-   `EmployeeID` (INT, PK)
-   `FirstName` (NVARCHAR(50))
-   `LastName` (NVARCHAR(50))
-   `Email` (NVARCHAR(100))
-   `PhoneNumber` (NVARCHAR(20))
-   `HireDate` (DATE)
-   `JobTitle` (NVARCHAR(50))
-   `Salary` (DECIMAL(10, 2))
-   `DepartmentID` (INT, FK to `Departments`)
-   `ManagerID` (INT, FK to `Employees` itself)

**2. `Departments` Table:**
-   `DepartmentID` (INT, PK)
-   `DepartmentName` (NVARCHAR(50))
-   `Location` (NVARCHAR(50))

**3. `Projects` Table:**
-   `ProjectID` (INT, PK)
-   `ProjectName` (NVARCHAR(100))
-   `StartDate` (DATE)
-   `EndDate` (DATE)
-   `Budget` (DECIMAL(12, 2))
-   `DepartmentID` (INT, FK to `Departments`)

Now, let's dive into the examples!

---

### `SELECT` Examples with Different Scenarios

#### 1. Basic Selection

**Retrieve all columns for all employees:**

```sql
SELECT *
FROM Employees;
```

**Retrieve specific columns for all employees:**

```sql
SELECT FirstName, LastName, JobTitle, Salary
FROM Employees;
```

#### 2. `DISTINCT` Clause

Used to return only unique values in the specified column(s).

**Retrieve all unique job titles:**

```sql
SELECT DISTINCT JobTitle
FROM Employees;
```

**Retrieve unique combinations of department and job title:**

```sql
SELECT DISTINCT DepartmentID, JobTitle
FROM Employees;
```

#### 3. `WHERE` Clause (Filtering Rows)

The `WHERE` clause is used to filter records based on a specified condition.

**a. Basic Condition:**

**Employees with a salary greater than $70,000:**

```sql
SELECT FirstName, LastName, Salary
FROM Employees
WHERE Salary > 70000.00;
```

**Employees in 'IT' department (assuming DepartmentID 10 is IT):**

```sql
SELECT FirstName, LastName, JobTitle
FROM Employees
WHERE DepartmentID = 10;
```

**b. `AND`, `OR`, `NOT` Operators:**

**Employees in 'IT' department with a salary greater than $80,000:**

```sql
SELECT FirstName, LastName, JobTitle, Salary
FROM Employees
WHERE DepartmentID = 10 AND Salary > 80000.00;
```

**Employees in 'IT' or 'HR' department:**

```sql
SELECT FirstName, LastName, DepartmentID
FROM Employees
WHERE DepartmentID = 10 OR DepartmentID = 20; -- Assuming 20 is HR
```

**Employees NOT in the 'Sales' department (assuming DepartmentID 30 is Sales):**

```sql
SELECT FirstName, LastName, DepartmentID
FROM Employees
WHERE NOT DepartmentID = 30;
```

**c. `BETWEEN` Operator:**

Used to select values within a given range (inclusive).

**Employees hired between January 1, 2020, and December 31, 2022:**

```sql
SELECT FirstName, LastName, HireDate
FROM Employees
WHERE HireDate BETWEEN '2020-01-01' AND '2022-12-31';
```

**Employees with salary between $60,000 and $90,000:**

```sql
SELECT FirstName, LastName, Salary
FROM Employees
WHERE Salary BETWEEN 60000.00 AND 90000.00;
```

**d. `IN` Operator:**

Used to specify multiple possible values for a column.

**Employees whose job title is 'Software Engineer' or 'Data Analyst':**

```sql
SELECT FirstName, LastName, JobTitle
FROM Employees
WHERE JobTitle IN ('Software Engineer', 'Data Analyst');
```

**Employees from departments 10, 20, or 40:**

```sql
SELECT FirstName, LastName, DepartmentID
FROM Employees
WHERE DepartmentID IN (10, 20, 40);
```

**e. `LIKE` Operator:**

Used for pattern matching with wildcards (`%` for any sequence of characters, `_` for any single character).

**Employees whose first name starts with 'J':**

```sql
SELECT FirstName, LastName
FROM Employees
WHERE FirstName LIKE 'J%';
```

**Employees whose last name contains 'son':**

```sql
SELECT FirstName, LastName
FROM Employees
WHERE LastName LIKE '%son%';
```

**Employees whose email address has 'mit.edu' domain:**

```sql
SELECT FirstName, LastName, Email
FROM Employees
WHERE Email LIKE '%@mit.edu';
```

**f. `IS NULL` / `IS NOT NULL`:**

Used to check for `NULL` values.

**Employees who do not have a manager assigned:**

```sql
SELECT FirstName, LastName
FROM Employees
WHERE ManagerID IS NULL;
```

**Employees who have a phone number recorded:**

```sql
SELECT FirstName, LastName, PhoneNumber
FROM Employees
WHERE PhoneNumber IS NOT NULL;
```

#### 4. `ORDER BY` Clause (Sorting Results)

Used to sort the result-set in ascending (`ASC`, default) or descending (`DESC`) order.

**a. Single Column Sort (Ascending):**

**All employees, ordered by last name alphabetically:**

```sql
SELECT FirstName, LastName, JobTitle
FROM Employees
ORDER BY LastName ASC;
```

**b. Single Column Sort (Descending):**

**Employees ordered by salary from highest to lowest:**

```sql
SELECT FirstName, LastName, Salary
FROM Employees
ORDER BY Salary DESC;
```

**c. Multiple Column Sort:**

**Employees ordered by department ID (ascending), then by salary (descending) within each department:**

```sql
SELECT FirstName, LastName, DepartmentID, Salary
FROM Employees
ORDER BY DepartmentID ASC, Salary DESC;
```

#### 5. `TOP` Clause (SQL Server Specific) / `OFFSET-FETCH` (Standard SQL)

Used to limit the number of rows returned.

**a. `TOP` (Absolute Number):**

**The top 5 highest-paid employees:**

```sql
SELECT TOP 5 FirstName, LastName, Salary
FROM Employees
ORDER BY Salary DESC;
```

**b. `TOP` (Percentage):**

**The top 10% highest-paid employees:**

```sql
SELECT TOP 10 PERCENT FirstName, LastName, Salary
FROM Employees
ORDER BY Salary DESC;
```

**c. `OFFSET-FETCH` (For Pagination - SQL Server 2012+):**

**Retrieve employees for the second page, with 10 employees per page (skipping the first 10, taking the next 10):**

```sql
SELECT FirstName, LastName, HireDate
FROM Employees
ORDER BY HireDate ASC
OFFSET 10 ROWS
FETCH NEXT 10 ROWS ONLY;
```

#### 6. Aggregate Functions

Used to perform calculations on a set of rows and return a single summary value. Common functions include `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`.

**Total number of employees:**

```sql
SELECT COUNT(EmployeeID) AS TotalEmployees
FROM Employees;
```

**Average salary of all employees:**

```sql
SELECT AVG(Salary) AS AverageSalary
FROM Employees;
```

**Highest and lowest salary:**

```sql
SELECT MAX(Salary) AS MaxSalary, MIN(Salary) AS MinSalary
FROM Employees;
```

**Total budget across all projects:**

```sql
SELECT SUM(Budget) AS TotalProjectBudget
FROM Projects;
```

#### 7. `GROUP BY` Clause

Used with aggregate functions to group the result-set by one or more columns.

**Count of employees in each department:**

```sql
SELECT DepartmentID, COUNT(EmployeeID) AS NumberOfEmployees
FROM Employees
GROUP BY DepartmentID;
```

**Average salary for each job title:**

```sql
SELECT JobTitle, AVG(Salary) AS AverageSalary
FROM Employees
GROUP BY JobTitle;
```

**Total budget for projects in each department:**

```sql
SELECT DepartmentID, SUM(Budget) AS TotalBudget
FROM Projects
GROUP BY DepartmentID;
```

#### 8. `HAVING` Clause

Used to filter groups created by the `GROUP BY` clause, similar to `WHERE` but for aggregated results.

**Departments with more than 5 employees:**

```sql
SELECT DepartmentID, COUNT(EmployeeID) AS NumberOfEmployees
FROM Employees
GROUP BY DepartmentID
HAVING COUNT(EmployeeID) > 5;
```

**Job titles where the average salary is above $75,000:**

```sql
SELECT JobTitle, AVG(Salary) AS AverageSalary
FROM Employees
GROUP BY JobTitle
HAVING AVG(Salary) > 75000.00;
```

#### 9. `JOIN` Operations

Used to combine rows from two or more tables based on a related column between them.

**a. `INNER JOIN` (Most Common):** Returns rows when there is a match in both tables.

**Employees and their corresponding department names:**

```sql
SELECT E.FirstName, E.LastName, E.JobTitle, D.DepartmentName
FROM Employees AS E
INNER JOIN Departments AS D ON E.DepartmentID = D.DepartmentID;
```

**Projects and the department responsible for them:**

```sql
SELECT P.ProjectName, P.Budget, D.DepartmentName, D.Location
FROM Projects AS P
INNER JOIN Departments AS D ON P.DepartmentID = D.DepartmentID;
```

**b. `LEFT JOIN` (or `LEFT OUTER JOIN`):** Returns all rows from the left table, and the matching rows from the right table. If there's no match, `NULL` values are returned for the right table's columns.

**All employees, and their department name if they have one (employees without a department will still appear):**

```sql
SELECT E.FirstName, E.LastName, D.DepartmentName
FROM Employees AS E
LEFT JOIN Departments AS D ON E.DepartmentID = D.DepartmentID;
```

**c. `RIGHT JOIN` (or `RIGHT OUTER JOIN`):** Returns all rows from the right table, and the matching rows from the left table. If there's no match, `NULL` values are returned for the left table's columns.

**All departments, and any employees assigned to them (departments with no employees will still appear):**

```sql
SELECT D.DepartmentName, E.FirstName, E.LastName
FROM Employees AS E
RIGHT JOIN Departments AS D ON E.DepartmentID = D.DepartmentID;
```

**d. `FULL JOIN` (or `FULL OUTER JOIN`):** Returns all rows when there is a match in one of the tables. If there is no match, the rows without a match will also be listed.

**All employees and all departments, showing matches where they exist, and `NULL`s where they don't:**

```sql
SELECT E.FirstName, E.LastName, D.DepartmentName
FROM Employees AS E
FULL JOIN Departments AS D ON E.DepartmentID = D.DepartmentID;
```

**e. `SELF JOIN`:** Joining a table to itself. Useful for hierarchical data like managers and employees.

**Employees and their managers:**

```sql
SELECT E.FirstName AS EmployeeFirstName, E.LastName AS EmployeeLastName,
       M.FirstName AS ManagerFirstName, M.LastName AS ManagerLastName
FROM Employees AS E
LEFT JOIN Employees AS M ON E.ManagerID = M.EmployeeID;
```

#### 10. Subqueries (Nested Queries)

A query nested inside another SQL query.

**Employees who earn more than the average salary:**

```sql
SELECT FirstName, LastName, Salary
FROM Employees
WHERE Salary > (SELECT AVG(Salary) FROM Employees);
```

**Departments that have at least one project with a budget over $1,000,000:**

```sql
SELECT DepartmentName
FROM Departments
WHERE DepartmentID IN (SELECT DepartmentID FROM Projects WHERE Budget > 1000000.00);
```

#### 11. `UNION` and `UNION ALL`

Used to combine the result-set of two or more `SELECT` statements.
-   `UNION` removes duplicate rows.
-   `UNION ALL` includes duplicate rows.
(Both queries must have the same number of columns, and the columns must have similar data types).

**Combine employee names and project names into a single list:**

```sql
SELECT FirstName AS Name, 'Employee' AS Type
FROM Employees
UNION ALL
SELECT ProjectName AS Name, 'Project' AS Type
FROM Projects;
```

---