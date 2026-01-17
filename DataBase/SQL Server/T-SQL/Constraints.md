### What are Constraints?

**Constraints** are rules enforced on data columns in a table. They are used to limit the type of data that can go into a table, ensuring the accuracy and reliability of the data in the database. If there is any violation between the constraint and the data action, the action is aborted.

**Why are Constraints Important?**

*   **Data Integrity:** They prevent invalid data from being entered into the database.
*   **Data Consistency:** They maintain the relationships and rules between tables.
*   **Reliability:** They ensure that your data adheres to predefined business rules.
*   **Performance:** Indexes are often automatically created for `PRIMARY KEY` and `UNIQUE` constraints, which can significantly improve query performance.

### Types of Constraints in SQL Server

We'll cover the most common and essential types:

1.  **`NOT NULL`:** Ensures that a column cannot have `NULL` values. [[Create Table with Constrains]]
2.  **`UNIQUE`:** Ensures that all values in a column (or a group of columns) are different.
3.  **`PRIMARY KEY` (PK):** Uniquely identifies each record in a table. It must contain `UNIQUE` and `NOT NULL` values. A table can have only one Primary Key.
4.  **`FOREIGN KEY` (FK):** Links two tables together by referencing the Primary Key of another table. It enforces referential integrity. [[Entity and Relationship Integrity]]
5.  **`CHECK`:** Ensures that all values in a column satisfy a specific condition.
6.  **`DEFAULT`:** Provides a default value for a column when no value is specified during an `INSERT` operation. (While technically a default value, it's often grouped with constraints as it enforces a rule about data entry).

---

### Adding Constraints

You can add constraints in two main ways:

1.  **During Table Creation (`CREATE TABLE` statement):** This is generally the preferred method as it defines the integrity rules from the outset.
2.  **After Table Creation (`ALTER TABLE` statement):** This is useful when you need to add new rules to an existing table or modify existing ones.

#### 1. Adding Constraints During Table Creation

Here's how you define various constraints when you first create a table:

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY IDENTITY(1,1), -- PRIMARY KEY (Column-level)
    FirstName VARCHAR(50) NOT NULL,           -- NOT NULL
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE,                -- UNIQUE (Column-level)
    PhoneNumber VARCHAR(20) UNIQUE,           -- UNIQUE (Column-level)
    HireDate DATE DEFAULT GETDATE(),          -- DEFAULT constraint
    Salary DECIMAL(10, 2) CHECK (Salary >= 30000.00), -- CHECK constraint (Column-level)
    DepartmentID INT,
    -- Table-level constraints
    CONSTRAINT UQ_Employee_Email UNIQUE (Email), -- UNIQUE (Table-level, named)
    CONSTRAINT CK_Employee_HireDate CHECK (HireDate <= GETDATE()), -- CHECK (Table-level, named)
    CONSTRAINT FK_Employee_Department FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID) -- FOREIGN KEY (Table-level, named)
);

CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY IDENTITY(1,1),
    DepartmentName VARCHAR(100) UNIQUE NOT NULL
);
```

**Explanation of the above `CREATE TABLE` statement:**

*   **`PRIMARY KEY`:** `EmployeeID` is defined as the primary key. `IDENTITY(1,1)` makes it an auto-incrementing integer.
*   **`NOT NULL`:** `FirstName` and `LastName` cannot be `NULL`.
*   **`UNIQUE`:** `Email` and `PhoneNumber` must contain unique values. Notice `Email` has both a column-level `UNIQUE` and a table-level `UNIQUE` constraint named `UQ_Employee_Email`. While redundant here, it demonstrates both syntaxes. Table-level constraints allow you to name them explicitly and apply them to multiple columns if needed (e.g., `UNIQUE (Col1, Col2)`).
*   **`DEFAULT`:** `HireDate` will automatically be set to the current date if no value is provided during an `INSERT`.
*   **`CHECK`:** `Salary` must be greater than or equal to 30000.00. `HireDate` must be less than or equal to the current date.
*   **`FOREIGN KEY`:** `DepartmentID` in `Employees` is a foreign key referencing `DepartmentID` in the `Departments` table. This ensures that an `Employee` can only be assigned to an existing `Department`.

**Naming Constraints:**
It's a best practice to explicitly name your constraints using the `CONSTRAINT constraint_name` syntax, especially for `UNIQUE`, `CHECK`, and `FOREIGN KEY` constraints. This makes it much easier to identify, modify, or drop them later. SQL Server will generate a default, often cryptic, name if you don't provide one.

#### 2. Adding Constraints After Table Creation

You can add constraints to an existing table using the `ALTER TABLE ADD CONSTRAINT` statement. This is particularly useful if you're modifying an existing schema or if the table already contains data that needs to be validated against the new constraint.

**Syntax:**

```sql
ALTER TABLE table_name
ADD CONSTRAINT constraint_name constraint_type (column_name(s)) [constraint_options];
```

Let's assume we created the `Employees` table without some of the constraints initially:

```sql
-- Initial table creation (without some constraints)
CREATE TABLE Employees_NoConstraints (
    EmployeeID INT IDENTITY(1,1),
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100),
    PhoneNumber VARCHAR(20),
    HireDate DATE,
    Salary DECIMAL(10, 2),
    DepartmentID INT
);
```

Now, let's add them:

```sql
-- Add PRIMARY KEY
ALTER TABLE Employees_NoConstraints
ADD CONSTRAINT PK_Employees_EmployeeID PRIMARY KEY (EmployeeID);

-- Add NOT NULL (this is done directly on the column, not with ADD CONSTRAINT)
ALTER TABLE Employees_NoConstraints
ALTER COLUMN FirstName VARCHAR(50) NOT NULL;

ALTER TABLE Employees_NoConstraints
ALTER COLUMN LastName VARCHAR(50) NOT NULL;

-- Add UNIQUE constraint
ALTER TABLE Employees_NoConstraints
ADD CONSTRAINT UQ_Employees_Email UNIQUE (Email);

ALTER TABLE Employees_NoConstraints
ADD CONSTRAINT UQ_Employees_PhoneNumber UNIQUE (PhoneNumber);

-- Add CHECK constraint
ALTER TABLE Employees_NoConstraints
ADD CONSTRAINT CK_Employees_Salary_Min CHECK (Salary >= 30000.00);

ALTER TABLE Employees_NoConstraints
ADD CONSTRAINT CK_Employees_HireDate_Valid CHECK (HireDate <= GETDATE());

-- Add DEFAULT constraint
ALTER TABLE Employees_NoConstraints
ADD CONSTRAINT DF_Employees_HireDate DEFAULT GETDATE() FOR HireDate;

-- Add FOREIGN KEY constraint (assuming Departments table exists)
-- First, ensure the referenced table (Departments) has a PK
CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY IDENTITY(1,1),
    DepartmentName VARCHAR(100) UNIQUE NOT NULL
);

ALTER TABLE Employees_NoConstraints
ADD CONSTRAINT FK_Employees_DepartmentID FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
    ON DELETE NO ACTION -- Example: Prevent deletion of department if employees exist
    ON UPDATE CASCADE;   -- Example: Update DepartmentID in Employees if it changes in Departments
```

**Important Note for `ALTER TABLE ADD CONSTRAINT`:**
When adding constraints to an existing table, SQL Server will check all existing data in the table to ensure it complies with the new constraint. If any existing data violates the constraint, the `ALTER TABLE` statement will fail. For `NOT NULL`, if the column contains `NULL`s, you must update them before adding the constraint.

---

### Updating Constraints

Directly "updating" most constraints in SQL Server is not a common operation. Instead, you typically follow a two-step process:

1.  **Drop** the existing constraint.
2.  **Add** a new constraint with the desired modifications.

For example, if you wanted to change the minimum salary in our `CK_Employees_Salary_Min` constraint from 30000.00 to 35000.00:

```sql
-- 1. Drop the existing CHECK constraint
ALTER TABLE Employees_NoConstraints
DROP CONSTRAINT CK_Employees_Salary_Min;

-- 2. Add the new CHECK constraint with the updated condition
ALTER TABLE Employees_NoConstraints
ADD CONSTRAINT CK_Employees_Salary_Min CHECK (Salary >= 35000.00);
```

For `NOT NULL` constraints, you would use `ALTER COLUMN` again:

```sql
-- To change a column from NOT NULL to NULL
ALTER TABLE Employees_NoConstraints
ALTER COLUMN Email VARCHAR(100) NULL;

-- To change a column from NULL to NOT NULL (requires all existing values to be non-NULL)
-- First, update any NULL values if they exist
UPDATE Employees_NoConstraints SET Email = 'unknown@example.com' WHERE Email IS NULL;
ALTER TABLE Employees_NoConstraints
ALTER COLUMN Email VARCHAR(100) NOT NULL;
```

---

### Deleting Constraints

To remove a constraint from a table, you use the `ALTER TABLE DROP CONSTRAINT` statement. You need to know the name of the constraint you want to drop.

**Syntax:**

```sql
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;
```

**Examples:**

```sql
-- Delete a PRIMARY KEY constraint
ALTER TABLE Employees_NoConstraints
DROP CONSTRAINT PK_Employees_EmployeeID;

-- Delete a UNIQUE constraint
ALTER TABLE Employees_NoConstraints
DROP CONSTRAINT UQ_Employees_Email;

-- Delete a CHECK constraint
ALTER TABLE Employees_NoConstraints
DROP CONSTRAINT CK_Employees_Salary_Min;

-- Delete a FOREIGN KEY constraint
ALTER TABLE Employees_NoConstraints
DROP CONSTRAINT FK_Employees_DepartmentID;

-- Deleting a DEFAULT constraint
ALTER TABLE Employees_NoConstraints
DROP CONSTRAINT DF_Employees_HireDate;

-- Deleting a NOT NULL constraint (this is done directly on the column)
ALTER TABLE Employees_NoConstraints
ALTER COLUMN FirstName VARCHAR(50) NULL;
```

**Finding Constraint Names:**
If you didn't explicitly name your constraints, or if you forget a name, you can find them by querying the system catalog views:

```sql
-- For all constraints on a specific table
SELECT
    OBJECT_NAME(parent_object_id) AS TableName,
    name AS ConstraintName,
    type_desc AS ConstraintType
FROM
    sys.objects
WHERE
    parent_object_id = OBJECT_ID('Employees_NoConstraints')
    AND type IN ('PK', 'UQ', 'F', 'C', 'D'); -- PK, Unique, Foreign Key, Check, Default
```

---

Understanding and effectively using constraints is a hallmark of a skilled database developer. They are your first line of defense against bad data and are crucial for building robust, scalable, and maintainable database systems. Always strive to define your integrity rules as constraints in the database rather than relying solely on application-level validation, as this ensures data quality regardless of how the data is accessed.