### 1. Null Functions

Null values represent the absence of data. They are not equivalent to zero or an empty string. Handling NULLs correctly is crucial to prevent unexpected behavior and ensure data integrity. SQL Server provides several functions specifically designed to manage these elusive values.

#### a. `ISNULL (check_expression, replacement_value)`

- **Description:** Replaces NULL with the specified replacement value. If `check_expression` is not NULL, it returns `check_expression`. If `check_expression` is NULL, it returns `replacement_value`. The data type of `replacement_value` must be implicitly convertible to the data type of `check_expression`.
- **Usage:** Ideal for ensuring that a column always returns a non-NULL value, especially in display or calculation contexts where NULLs might cause issues. It's often used in `SELECT` statements to present more user-friendly output or in `WHERE` clauses to handle NULLs in comparisons.
- **Example:**

```sql
-- Assume a table 'Products' with a 'Description' column that can be NULL
CREATE TABLE Products (
  ProductID INT PRIMARY KEY,
  ProductName NVARCHAR(100),
  Description NVARCHAR(MAX)
);

INSERT INTO Products (ProductID, ProductName, Description) VALUES
(1, 'Laptop', 'High-performance laptop'),
(2, 'Mouse', NULL),
(3, 'Keyboard', 'Ergonomic keyboard');

SELECT
  ProductName,
  ISNULL(Description, 'No description available') AS ProductDescription
FROM Products;
```

  **Output:**

| ProductName | ProductDescription       |
| :---------- | :----------------------- |
| Laptop      | High-performance laptop  |
| Mouse       | No description available |
| Keyboard    | Ergonomic keyboard       |

#### b. `COALESCE (expression1, expression2, ..., expressionN)`

- **Description:** Evaluates the arguments in order and returns the first non-NULL expression. All expressions must be of a compatible data type. Unlike `ISNULL`, `COALESCE` can take multiple arguments, offering greater flexibility. It adheres to the SQL standard.
- **Usage:** Extremely useful when you have several potential sources for a value and you want to pick the first available (non-NULL) one. This is common in data integration scenarios or when consolidating information from multiple columns.
- **Example:**

```sql
-- Assume a table 'Customers' with multiple contact numbers
CREATE TABLE Customers (
  CustomerID INT PRIMARY KEY,
  CustomerName NVARCHAR(100),
  Phone1 NVARCHAR(20),
  Phone2 NVARCHAR(20),
  Email NVARCHAR(100)
);

INSERT INTO Customers (CustomerID, CustomerName, Phone1, Phone2, Email) VALUES
(101, 'Alice', '555-1234', NULL, 'alice@example.com'),
(102, 'Bob', NULL, '555-5678', 'bob@example.com'),
(103, 'Charlie', NULL, NULL, 'charlie@example.com');

SELECT
  CustomerName,
  COALESCE(Phone1, Phone2, 'No phone available') AS PrimaryContactPhone
FROM Customers;
```

  **Output:**

| CustomerName | PrimaryContactPhone |
| :----------- | :------------------ |
| Alice        | 555-1234            |
| Bob          | 555-5678            |
| Charlie      | No phone available  |

#### c. `NULLIF (expression1, expression2)`

- **Description:** Returns NULL if `expression1` is equal to `expression2`. Otherwise, it returns `expression1`. This function is often used to "nullify" specific values that should be treated as missing data.
- **Usage:** Primarily used to prevent division by zero errors (by making the divisor NULL if it's zero) or to treat specific placeholder values (like an empty string or a default '0') as NULL for logical operations or aggregations, which might then be handled by `ISNULL` or `COALESCE`.
- **Example:**

```sql
-- Assume a table 'SalesData' where 'UnitsSold' might be 0, but we want to treat it as NULL for average calculations
CREATE TABLE SalesData (
  SaleID INT PRIMARY KEY,
  ProductName NVARCHAR(100),
  UnitsSold INT,
  Revenue DECIMAL(10, 2)
);

INSERT INTO SalesData (SaleID, ProductName, UnitsSold, Revenue) VALUES
(1, 'Widget A', 100, 1000.00),
(2, 'Widget B', 0, 0.00), -- Treat 0 units sold as NULL for some analyses
(3, 'Widget C', 50, 750.00),
(4, 'Widget D', 0, 0.00);

SELECT
  ProductName,
  UnitsSold,
  NULLIF(UnitsSold, 0) AS UnitsSold_AsNullIfZero,
  Revenue / NULLIF(UnitsSold, 0) AS PricePerUnit -- Avoids division by zero
FROM SalesData;
```

  **Output:**

| ProductName | UnitsSold | UnitsSold_AsNullIfZero | PricePerUnit |
| :---------- | :-------- | :--------------------- | :----------- |
| Widget A    | 100       | 100                    | 10.000000    |
| Widget B    | 0         | NULL                   | NULL         |
| Widget C    | 50        | 50                     | 15.000000    |
| Widget D    | 0         | NULL                   | NULL         |



