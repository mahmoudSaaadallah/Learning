### What is a T-SQL Indexed View?

An **Indexed View** (often referred to as a **Materialized View** in other database systems) is a view that has a unique clustered index created on it. When you create a unique clustered index on a view, SQL Server actually *materializes* the result set of the view's underlying `SELECT` statement and stores it physically on disk, just like a regular table. Subsequent non-clustered indexes can then be built on top of this materialized data.

Unlike a standard view [[T-SQL Standard View]], which executes its `SELECT` statement every time it's queried, an indexed view's data is pre-computed and stored. This means that queries against the indexed view (or even queries against the base tables that the optimizer can rewrite to use the indexed view) can retrieve data much faster because the complex joins, aggregations, or computations have already been performed.

### Why Use Indexed Views? The Performance Imperative

The primary motivation for using indexed views is **query performance optimization**, especially for:

1.  **Complex Aggregations:** Queries involving `SUM`, `COUNT`, `AVG`, `MIN`, `MAX` over large datasets can be very slow. An indexed view can pre-compute these aggregates, making subsequent queries almost instantaneous.
2.  **Frequent Joins:** Views that involve joining multiple large tables can be expensive. An indexed view can store the pre-joined result, eliminating the need for repeated join operations.
3.  **Data Warehousing and Reporting:** In OLAP (Online Analytical Processing) environments, where read-heavy, complex analytical queries are common, indexed views can dramatically speed up report generation and data analysis.
4.  **Reducing CPU and I/O:** By storing pre-computed results, indexed views reduce the CPU cycles needed for calculations and the I/O required to read and process raw data.

### How Indexed Views Work

1.  **Materialization:** When the unique clustered index is created on the view, the view's `SELECT` statement is executed, and its result set is stored in a special structure on disk.
2.  **Automatic Maintenance:** This is a critical feature. SQL Server automatically maintains the data in the indexed view whenever changes (inserts, updates, deletes) occur in the underlying base tables. This ensures that the indexed view always reflects the current state of the base data. This automatic maintenance comes with a cost, as DML operations on base tables will take longer.
3.  **Query Optimizer Integration:** The SQL Server query optimizer is intelligent enough to recognize when an indexed view can satisfy a query, even if the query doesn't explicitly reference the view. If a query against the base tables can be answered more efficiently by using the indexed view, the optimizer will automatically rewrite the query plan to use the indexed view. This is known as **view matching** or **automatic query rewrite**.

### Key Requirements for Creating an Indexed View

Creating an indexed view is more restrictive than creating a standard view due to the need for physical materialization and automatic maintenance. Here are the most important requirements:

1.  **`WITH SCHEMABINDING`:** The view must be created with the `WITH SCHEMABINDING` option. This binds the view to the schema of the underlying tables, preventing them from being dropped or altered in a way that would affect the view definition without first dropping or altering the view. This ensures the view's integrity.
2.  **Unique Clustered Index:** The *first* index created on the view *must* be a `UNIQUE CLUSTERED INDEX`. This is what materializes the view.
3.  **Deterministic Functions:** All functions referenced in the view (e.g., `SUM`, `COUNT`, `AVG`, `GETDATE()`, `RAND()`) must be deterministic. A deterministic function always returns the same result for the same set of input values. `GETDATE()` and `RAND()` are non-deterministic, for example.
4.  **`COUNT_BIG` for `COUNT(*)`:** If you use `COUNT(*)`, you must use `COUNT_BIG(*)` instead.
5.  **No `DISTINCT`, `TOP`, `ROWSET`, `TEXTPTR`, `COMPUTE`, `COMPUTE BY`:** These clauses are generally not allowed.
6.  **No `OUTER JOIN` (with some exceptions):** `LEFT OUTER JOIN` and `RIGHT OUTER JOIN` are allowed if the `WHERE` clause filters out `NULL` values from the outer side, effectively making it an `INNER JOIN`. `FULL OUTER JOIN` is not allowed.
7.  **No `UNION`, `INTERSECT`, `EXCEPT`:** These set operators are not allowed.
8.  **Base Tables Must Be in the Same Database:** All tables referenced in the view must be in the same database.
9.  **`NULL`ability:** Columns in the view that can be `NULL` must be explicitly defined as `NULL`able in the view definition if they are derived from expressions.
10. **`QUOTED_IDENTIFIER` and `ANSI_NULLS`:** These `SET` options must be `ON` when the view is created and when any DML operations are performed on the base tables.

### Example: Indexed View for Sales Order Aggregations

Let's use our `Sales` database example again, focusing on aggregating sales data.

```sql
-- 1. Ensure SET options are correctly configured for the session
SET NUMERIC_ROUNDABORT OFF;
SET ANSI_PADDING ON;
SET ANSI_WARNINGS ON;
SET ARITHABORT ON;
SET CONCAT_NULL_YIELDS_NULL ON;
SET QUOTED_IDENTIFIER ON;
SET ANSI_NULLS ON;
GO

-- 2. Create sample base tables (if not already existing from previous examples)
-- Customers (CustomerID, FirstName, LastName)
-- Orders (OrderID, CustomerID, OrderDate, TotalAmount)
-- OrderItems (OrderItemID, OrderID, ProductID, Quantity, UnitPrice)
-- Products (ProductID, ProductName, Price)

-- For simplicity, let's assume we have these tables and some data.
-- Let's create a simplified Orders and OrderItems for this example.
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY IDENTITY(1,1),
    OrderDate DATE NOT NULL,
    CustomerID INT NOT NULL,
    TotalAmount DECIMAL(18, 2) NOT NULL
);

CREATE TABLE OrderItems (
    OrderItemID INT PRIMARY KEY IDENTITY(1,1),
    OrderID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    UnitPrice DECIMAL(10, 2) NOT NULL,
    CONSTRAINT FK_OrderItems_Orders FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);

INSERT INTO Orders (OrderDate, CustomerID, TotalAmount) VALUES
('2023-01-10', 1, 150.00),
('2023-01-10', 2, 200.00),
('2023-02-15', 1, 100.00),
('2023-03-20', 3, 300.00),
('2024-01-05', 2, 250.00);

INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice) VALUES
(1, 101, 2, 75.00),
(1, 102, 1, 75.00),
(2, 101, 4, 50.00),
(3, 103, 1, 100.00),
(4, 101, 3, 100.00),
(5, 102, 5, 50.00);
GO

-- 3. Create the view with SCHEMABINDING
CREATE VIEW [[Sales.DailyOrderSummary]]
WITH SCHEMABINDING
AS
SELECT
    OrderDate,
    COUNT_BIG(o.OrderID) AS TotalOrders,
    SUM(oi.Quantity * oi.UnitPrice) AS TotalRevenue,
    COUNT_BIG(DISTINCT o.CustomerID) AS UniqueCustomers -- Note: COUNT(DISTINCT) is allowed in indexed views
FROM
    dbo.Orders AS o -- Must use two-part name (schema.table)
JOIN
    dbo.OrderItems AS oi ON o.OrderID = oi.OrderID
GROUP BY
    OrderDate;
GO

-- 4. Create a UNIQUE CLUSTERED INDEX on the view
-- This is the step that materializes the view.
CREATE UNIQUE CLUSTERED INDEX IX_DailyOrderSummary_OrderDate
ON [[Sales.DailyOrderSummary]] (OrderDate);
GO

-- Now, let's query the view
SELECT * FROM [[Sales.DailyOrderSummary]];

-- Or, query the base tables, and the optimizer might use the indexed view
-- (Check the execution plan to confirm view matching)
SELECT
    OrderDate,
    SUM(oi.Quantity * oi.UnitPrice) AS DailySales
FROM
    dbo.Orders AS o
JOIN
    dbo.OrderItems AS oi ON o.OrderID = oi.OrderID
WHERE
    o.OrderDate >= '2023-01-01'
GROUP BY
    OrderDate;
```

In the second `SELECT` statement, even though we're querying `Orders` and `OrderItems` directly, the SQL Server optimizer is likely to use the pre-computed data from `[[Sales.DailyOrderSummary]]` because it can satisfy the query more efficiently. This is the "magic" of indexed views.

### Considerations and Best Practices

1.  **Maintenance Overhead:** While indexed views boost read performance, they add overhead to DML operations (INSERT, UPDATE, DELETE) on the underlying base tables. Every change to a base table requires a corresponding update to the indexed view, which consumes CPU and I/O resources. This makes them ideal for read-heavy workloads.
2.  **Storage Space:** Indexed views consume additional disk space because their data is physically stored.
3.  **Complexity:** They are more complex to design and manage due to the strict creation requirements and the impact on DML.
4.  **When to Use:**
    *   Queries involve complex joins and aggregations that run frequently.
    *   The underlying data changes infrequently (or the performance gain outweighs the DML overhead).
    *   You need to accelerate reporting or OLAP queries.
5.  **When NOT to Use:**
    *   The underlying data changes very frequently.
    *   The view definition is simple (e.g., just selecting a few columns from a single table) where a standard index on the base table would suffice.
    *   Disk space is a major constraint.
6.  **Monitoring:** Regularly monitor the performance of DML operations on base tables and the usage of indexed views to ensure they are providing a net benefit.
7.  **Enterprise Edition Feature:** In some older versions of SQL Server, automatic query rewrite (where the optimizer uses the indexed view without explicit reference) was an Enterprise Edition feature. Standard Edition could create and query indexed views directly, but wouldn't get the automatic rewrite benefit. Always check the specific SQL Server version and edition documentation.

### Conclusion

Indexed Views are a powerful tool for achieving significant performance gains in specific scenarios, particularly for complex, read-intensive queries. They transform a logical view into a physical, pre-computed dataset, allowing the SQL Server optimizer to deliver results with remarkable speed. However, this power comes with trade-offs in terms of DML overhead and storage, necessitating careful design and a clear understanding of your workload characteristics. They are a prime example of how database professionals can leverage advanced features to optimize data access in demanding environments.