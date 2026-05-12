### What is a T-SQL Standard View?

At its core, a **view** in T-SQL is a virtual table whose content is defined by a query. It doesn't store data itself; rather, it's a stored query that, when referenced, dynamically retrieves data from one or more underlying base tables. Think of it as a pre-defined window into your data. When you query a view, the database engine essentially executes the view's underlying `SELECT` statement and presents the results as if they were coming from a table.

A "Standard View" typically refers to a non-indexed view, which is the most common type. It's a logical construct that simplifies data access and enhances security without the physical storage overhead of an indexed view [[T-SQL Indexed View]].

### Why Use Standard Views? The Pillars of Good Database Design

Views aren't just a neat trick; they are fundamental to building scalable, secure, and maintainable database applications. Here are the primary reasons we employ them:

1.  **Security (Data Access Control):**
    *   Views allow you to restrict access to specific rows and columns of a table. Instead of granting users direct access to sensitive base tables, you can grant them access only to a view that exposes a subset of the data.
    *   **Example:** A `HumanResources` table might contain salary information. You can create a view that excludes the `Salary` column and grant general employees access to this view, while only HR personnel get access to the full table or a view that includes salary.

2.  **Simplification of Complex Queries:**
    *   Many business requirements involve joining multiple tables, applying aggregations, or complex filtering. Views encapsulate these intricate queries, presenting a simplified interface to end-users or applications.
    *   Instead of rewriting complex joins every time, developers can simply query the view. This promotes code reusability and reduces the likelihood of errors.

3.  **Data Abstraction and Consistency:**
    *   Views provide a layer of abstraction between the application and the underlying database schema. If the schema of the base tables changes (e.g., a column is renamed, or tables are normalized/denormalized), you can often modify the view definition to accommodate these changes without impacting the applications that query the view.
    *   They can also enforce data consistency by presenting a unified, consistent view of data that might be spread across several tables or even different schemas.

4.  **Customization of Data Presentation:**
    *   You can rename columns, reorder them, or derive new columns (e.g., `FullName = FirstName + ' ' + LastName`) within a view to present data in a more user-friendly or application-specific format.

5.  **Backward Compatibility:**
    *   When refactoring a database, views can be used to maintain the old schema's interface, allowing older applications to continue functioning while new applications use the updated schema.

### Creating a Standard View: Syntax and Examples

The basic syntax for creating a view is straightforward:

```sql
CREATE VIEW ViewName
AS
SELECT column1, column2, ...
FROM Table1
JOIN Table2 ON Table1.ID = Table2.ID
WHERE condition;
```

Let's illustrate with some practical examples using a hypothetical `Sales` database.

**Scenario:** We have `Customers`, `Orders`, and `Products` tables.

#### Example 1: Simple View for Customer Order Information

Let's say we frequently need to see customer names along with their order details.

```sql
-- Assume these tables exist:
-- Customers (CustomerID, FirstName, LastName, Email)
-- Orders (OrderID, CustomerID, OrderDate, TotalAmount)
-- Products (ProductID, ProductName, Price)
-- OrderItems (OrderItemID, OrderID, ProductID, Quantity, UnitPrice)

CREATE VIEW [[CustomerOrderSummary]]
AS
SELECT
    c.CustomerID,
    c.FirstName + ' ' + c.LastName AS CustomerName,
    o.OrderID,
    o.OrderDate,
    o.TotalAmount
FROM
    Customers AS c
JOIN
    Orders AS o ON c.CustomerID = o.CustomerID
WHERE
    o.OrderDate >= '2023-01-01'; -- Only show orders from 2023 onwards
GO

-- To use the view:
SELECT *
FROM [[CustomerOrderSummary]]
WHERE CustomerName LIKE 'John%';
```

#### Example 2: Security View (Hiding Sensitive Data)

Suppose the `Customers` table also contains a `CreditCardNumber` column, which should not be accessible to all users.

```sql
CREATE VIEW [[PublicCustomerInfo]]
AS
SELECT
    CustomerID,
    FirstName,
    LastName,
    Email,
    PhoneNumber -- Exclude CreditCardNumber
FROM
    Customers;
GO

-- Grant SELECT permission on this view, not on the base Customers table
GRANT SELECT ON [[PublicCustomerInfo]] TO PublicRole;
```

#### Example 3: View for Aggregated Data (Simplifying Reporting)

A common requirement is to see the total sales per product.

```sql
CREATE VIEW [[ProductSalesSummary]]
AS
SELECT
    p.ProductName,
    SUM(oi.Quantity * oi.UnitPrice) AS TotalRevenue,
    COUNT(DISTINCT o.OrderID) AS NumberOfOrders
FROM
    Products AS p
JOIN
    OrderItems AS oi ON p.ProductID = oi.ProductID
JOIN
    Orders AS o ON oi.OrderID = o.OrderID
GROUP BY
    p.ProductName;
GO

-- Querying the aggregated view is much simpler for reporting
SELECT ProductName, TotalRevenue
FROM [[ProductSalesSummary]]
ORDER BY TotalRevenue DESC;
```


---
### Conditions for an Updatable View

A view is generally updatable if it meets these criteria:

1.  **Single Base Table:** The view must be based on a single underlying table. If it involves joins between multiple tables, it's typically not directly updatable (though there are advanced scenarios with `INSTEAD OF` triggers that can make multi-table views "updatable" programmatically, but that's beyond a standard view's inherent capability).
2.  **No Aggregate Functions:** The view's `SELECT` statement must not contain aggregate functions (`SUM`, `COUNT`, `AVG`, `MIN`, `MAX`).
3.  **No `DISTINCT` Clause:** The view cannot use the `DISTINCT` keyword.
4.  **No `GROUP BY` or `HAVING` Clauses:** These clauses imply aggregation, making the view non-updatable.
5.  **No `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT` Operators:** These combine result sets, making direct modification ambiguous.
6.  **No Derived Columns:** Columns that are computed (e.g., `FirstName + ' ' + LastName`) cannot be updated directly. You can update other columns, but not the derived ones.
7.  **All `NOT NULL` Columns from the Base Table are Included:** If the base table has `NOT NULL` columns without a default value, they must be included in the view for `INSERT` operations to succeed.

Let's set up a simple scenario and demonstrate these operations.

---

### Scenario: Employee Information

We'll create a simple `Employees` table and then a view based on it.

```sql
-- 1. Create a sample base table
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY IDENTITY(1,1),
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT    NULL,
    Email VARCHAR(100) UNIQUE,
    HireDate DATE,
    Salary DECIMAL(10, 2)
);
GO

-- Insert some initial data
INSERT INTO Employees (FirstName, LastName, Email, HireDate, Salary) VALUES
('Alice', 'Smith', 'alice.s@example.com', '2022-01-15', 60000.00),
('Bob', 'Johnson', 'bob.j@example.com', '2021-03-20', 75000.00),
('Charlie', 'Brown', 'charlie.b@example.com', '2023-07-01', 55000.00);
GO

-- 2. Create an updatable view
-- This view selects a subset of columns and filters some rows,
-- but it's based on a single table and meets the updatability criteria.
CREATE VIEW [[ActiveEmployeesView]]
AS
SELECT
    EmployeeID,
    FirstName,
    LastName,
    Email,
    HireDate
FROM
    Employees
WHERE
    HireDate IS NOT NULL; -- Just a simple filter
GO

-- Let's see the initial data through the view
SELECT * FROM [[ActiveEmployeesView]];
```

**Initial View Data:**

| EmployeeID | FirstName | LastName | Email               | HireDate   |
|------------|-----------|----------|---------------------|------------|
| 1          | Alice     | Smith    | alice.s@example.com | 2022-01-15 |
| 2          | Bob       | Johnson  | bob.j@example.com   | 2021-03-20 |
| 3          | Charlie   | Brown    | charlie.b@example.com | 2023-07-01 |

---

### 1. `INSERT` through the View

When you `INSERT` into an updatable view, the data is inserted into the underlying base table. You must provide values for all `NOT NULL` columns of the base table that are *not* defined with a default value, even if they are not part of the view's `SELECT` list.

In our `[[ActiveEmployeesView]]`, `Salary` is a `NULL`able column, so we don't *have* to provide it. `EmployeeID` is an `IDENTITY` column, so it's auto-generated.

```sql
-- Insert a new employee through the view
INSERT INTO [[ActiveEmployeesView]] (FirstName, LastName, Email, HireDate)
VALUES ('Diana', 'Prince', 'diana.p@example.com', '2024-01-10');
GO

-- Verify the insert by querying the view
SELECT * FROM [[ActiveEmployeesView]];

-- And also by querying the base table to see the Salary column (which will be NULL)
SELECT * FROM Employees;
```

**View Data After INSERT:**

| EmployeeID | FirstName | LastName | Email               | HireDate   |
|------------|-----------|----------|---------------------|------------|
| 1          | Alice     | Smith    | alice.s@example.com | 2022-01-15 |
| 2          | Bob       | Johnson  | bob.j@example.com   | 2021-03-20 |
| 3          | Charlie   | Brown    | charlie.b@example.com | 2023-07-01 |
| 4          | Diana     | Prince   | diana.p@example.com | 2024-01-10 |

**Base Table Data After INSERT (showing `Salary` as `NULL` for Diana):**

| EmployeeID | FirstName | LastName | Email               | HireDate   | Salary   |
|------------|-----------|----------|---------------------|------------|----------|
| 1          | Alice     | Smith    | alice.s@example.com | 2022-01-15 | 60000.00 |
| 2          | Bob       | Johnson  | bob.j@example.com   | 2021-03-20 | 75000.00 |
| 3          | Charlie   | Brown    | charlie.b@example.com | 2023-07-01 | 55000.00 |
| 4          | Diana     | Prince   | diana.p@example.com | 2024-01-10 | NULL     |

---

### 2. `UPDATE` through the View

When you `UPDATE` an updatable view, the changes are applied to the corresponding rows in the underlying base table. You can only update columns that are part of the view's `SELECT` list and are not derived.

```sql
-- Update Diana Prince's email through the view
UPDATE [[ActiveEmployeesView]]
SET Email = 'diana.prince@example.com'
WHERE FirstName = 'Diana' AND LastName = 'Prince';
GO

-- Verify the update by querying the view
SELECT * FROM [[ActiveEmployeesView]] WHERE FirstName = 'Diana';

-- And also by querying the base table
SELECT * FROM Employees WHERE FirstName = 'Diana';
```

**View Data After UPDATE:**

| EmployeeID | FirstName | LastName | Email                  | HireDate   |
|------------|-----------|----------|------------------------|------------|
| 4          | Diana     | Prince   | diana.prince@example.com | 2024-01-10 |

---

### 3. `DELETE` through the View

When you `DELETE` rows from an updatable view, the corresponding rows are deleted from the underlying base table.

```sql
-- Delete Charlie Brown through the view
DELETE FROM [[ActiveEmployeesView]]
WHERE FirstName = 'Charlie' AND LastName = 'Brown';
GO

-- Verify the delete by querying the view
SELECT * FROM [[ActiveEmployeesView]];

-- And also by querying the base table
SELECT * FROM Employees;
```

**View Data After DELETE:**

| EmployeeID | FirstName | LastName | Email                  | HireDate   |
|------------|-----------|----------|------------------------|------------|
| 1          | Alice     | Smith    | alice.s@example.com    | 2022-01-15 |
| 2          | Bob       | Johnson  | bob.j@example.com      | 2021-03-20 |
| 4          | Diana     | Prince   | diana.prince@example.com | 2024-01-10 |

Charlie Brown is now gone from both the view and the base table.

---

### Example of a Non-Updatable View

Let's create a view that *cannot* be updated and see what happens if we try.

```sql
-- Create a non-updatable view (due to JOIN and aggregate function)
CREATE VIEW [[EmployeeDepartmentSummary]]
AS
SELECT
    e.FirstName + ' ' + e.LastName AS FullName,
    d.DepartmentName,
    COUNT(e.EmployeeID) OVER (PARTITION BY d.DepartmentName) AS EmployeesInDept
FROM
    Employees AS e
JOIN
    Departments AS d ON e.DepartmentID = d.DepartmentID; -- Assuming a Departments table exists
GO

-- Attempting to INSERT into this view would fail:
-- INSERT INTO [[EmployeeDepartmentSummary]] (FullName, DepartmentName) VALUES ('New Guy', 'IT');
-- Error: View or function 'EmployeeDepartmentSummary' is not updatable because the modification affects multiple base tables.

-- Attempting to UPDATE a derived column would fail:
-- UPDATE [[EmployeeDepartmentSummary]] SET FullName = 'New Name' WHERE FullName = 'Alice Smith';
-- Error: The column 'FullName' cannot be modified because it is either a computed column or is the result of a UNION operator.
```

In the `[[EmployeeDepartmentSummary]]` example, the view is non-updatable for several reasons:
*   It involves a `JOIN` between `Employees` and `Departments`.
*   It uses an aggregate function (`COUNT(...) OVER (...)`).
*   It has a derived column (`FullName`).

---

### Conclusion on Updatability

While the ability to modify data through views can be convenient for simplifying application logic or enforcing specific business rules (e.g., only allowing updates to certain columns), it's crucial to design your views carefully. For complex scenarios, especially those involving multiple tables or intricate business logic, using `INSTEAD OF` triggers on views is the more robust and flexible approach to control DML operations, as it allows you to define exactly how `INSERT`, `UPDATE`, and `DELETE` statements on the view should translate into operations on the underlying base tables.

For standard, single-table views that meet the updatability criteria, direct DML operations are a powerful feature for data manipulation.
### Advanced View Options (Briefly)

While the above covers "Standard Views," it's worth mentioning a couple of important `CREATE VIEW` options:

*   **`WITH ENCRYPTION`**: Encrypts the definition of the view in the `sys.syscomments` table. This prevents others from easily viewing the underlying `SELECT` statement.
*   **`WITH SCHEMABINDING`**: This is a crucial option. It binds the view to the schema of the underlying base tables. This means that the base tables cannot be dropped or altered in a way that would affect the view definition unless the view is dropped or altered first. `SCHEMABINDING` is a prerequisite for creating **indexed views** (also known as materialized views in other systems), which physically store the result set and can significantly improve performance for complex queries, but they come with their own set of considerations regarding storage and update overhead.

### Considerations and Best Practices

1.  **Updatable Views:** Not all views are updatable. A view is generally updatable (meaning you can `INSERT`, `UPDATE`, or `DELETE` rows through it) if it's based on a single table and doesn't contain `DISTINCT`, `GROUP BY`, `HAVING`, `UNION`, or aggregate functions. Views involving multiple tables or complex logic are typically read-only.
2.  **Performance:** While views simplify queries, they don't inherently improve performance (unless they are indexed views). The query optimizer still has to execute the underlying `SELECT` statement. Overly complex views, or views built on other views, can sometimes lead to performance degradation if not carefully designed.
3.  **Naming Conventions:** Use clear, descriptive names for your views, often prefixed with `vw` or `v` (e.g., `vwCustomerOrders`).
4.  **Dependencies:** Be mindful of dependencies. If you alter a base table that a view relies on, the view might become invalid. `SCHEMABINDING` helps prevent accidental breaking changes.
5.  **`ALTER VIEW` and `DROP VIEW`:**
    *   To modify an existing view, use `ALTER VIEW ViewName AS SELECT ...`.
    *   To remove a view, use `DROP VIEW ViewName;`.

### Conclusion

Standard Views are an indispensable tool in the T-SQL developer's arsenal. They empower us to build more secure, maintainable, and user-friendly database systems by abstracting complexity, enforcing security, and providing flexible data presentation. While they are virtual constructs, their impact on the architecture and usability of a database is profoundly real. Mastering their creation and application is a hallmark of a proficient database professional.