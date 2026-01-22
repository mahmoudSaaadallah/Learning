`GROUPING SETS` is a powerful and flexible extension to the `GROUP BY` clause in SQL Server (and standard SQL) that allows you to specify multiple grouping criteria within a single `SELECT` statement. Instead of writing separate queries and combining them with `UNION ALL` [[Union Family]], `GROUPING SETS` enables you to generate a single result set containing aggregates for various combinations of columns, including subtotals and grand totals, all in one go.

It's the most general form of the `GROUP BY` extensions, with `CUBE` [[T-SQL CUBE]] and `ROLLUP` [[T-SQL Rollup]] being specialized, shorthand versions of `GROUPING SETS`.

### 1. Purpose of GROUPING SETS

The primary purpose of `GROUPING SETS` is to:

*   **Consolidate multiple `GROUP BY` queries**: Avoid writing multiple `SELECT` statements with `GROUP BY` and then combining them using `UNION ALL`. This improves query performance and readability.
*   **Generate custom aggregate reports**: Produce subtotals and grand totals for specific, non-hierarchical, or arbitrary combinations of columns that `CUBE` or `ROLLUP` might not directly support.
*   **Provide flexibility**: You have complete control over which grouping combinations are included in the final result set.

### 2. Syntax

The `GROUPING SETS` operator is used within the `GROUP BY` clause:

```sql
SELECT
    column1,
    column2,
    ...,
    aggregate_function(expression)
FROM
    your_table
GROUP BY
    GROUPING SETS (
        (column_set_1),
        (column_set_2),
        ...,
        (column_set_N),
        () -- Optional: for grand total
    );
```

Each `(column_set_X)` represents a distinct `GROUP BY` clause.
*   An empty parenthesis `()` specifies a grand total (aggregation over all rows).
*   A single column `(column_name)` aggregates by that column.
*   Multiple columns `(column1, column2)` aggregate by the combination of those columns.

### 3. How GROUPING SETS Works

Conceptually, `GROUPING SETS` works by performing each specified `GROUP BY` operation independently and then combining their results into a single output set. SQL Server optimizes this process, often executing it more efficiently than separate `UNION ALL` queries.

For rows that belong to a particular grouping set, the columns *not* included in that specific grouping set will appear as `NULL` in the result set. These `NULL`s indicate that the corresponding aggregate value is a subtotal or grand total, not a specific data point for that column.

### 4. The `GROUPING()` Function (Revisited)

As with `CUBE` and `ROLLUP`, the `GROUPING(column_name)` function is essential for distinguishing between `NULL` values that represent an aggregate level (a subtotal or grand total) and actual `NULL` values present in the original data.

*   `GROUPING(column_name)` returns `1` if the column is being aggregated (i.e., it's a placeholder `NULL` for a subtotal/grand total).
*   `GROUPING(column_name)` returns `0` if the column is part of the current grouping level (i.e., its value is an actual data value, which could be `NULL` if the original data contained `NULL`).

### 5. The `GROUPING_ID()` Function

`GROUPING_ID()` is another useful function that returns an integer bitmap representing the grouping level for a row. It takes a list of columns as arguments, similar to `GROUPING SETS` itself.

`GROUPING_ID(column1, column2, ...)`:
*   For each column in the argument list, it assigns a bit (0 or 1).
*   If a column is part of the current grouping level, its bit is `0`.
*   If a column is being aggregated (i.e., its value is `NULL` due to aggregation), its bit is `1`.
*   The bits are then combined into an integer, read from right to left.

For example, `GROUPING_ID(A, B)`:
*   If grouping by `(A, B)`: `00` (binary) = `0`
*   If grouping by `(A)`: `01` (binary) = `1` (B is aggregated)
*   If grouping by `(B)`: `10` (binary) = `2` (A is aggregated)
*   If grouping by `()`: `11` (binary) = `3` (A and B are aggregated)

`GROUPING_ID()` can be very helpful for filtering or ordering results based on specific aggregation levels.

### 6. Detailed Examples

Let's use the same `Sales` table from the `CUBE` explanation:

```sql
CREATE TABLE Sales (
    Region VARCHAR(50),
    Product VARCHAR(50),
    SalesAmount DECIMAL(10, 2)
);

INSERT INTO Sales (Region, Product, SalesAmount) VALUES
('East', 'Laptop', 1000.00),
('East', 'Mouse', 50.00),
('West', 'Laptop', 1200.00),
('West', 'Keyboard', 75.00),
('Central', 'Laptop', 900.00),
('Central', 'Mouse', 60.00),
('East', 'Laptop', 1100.00);
```

#### Scenario 1: Basic Custom Grouping Sets

Suppose you want to see total sales by `Region`, total sales by `Product`, and a grand total, but *not* the combination of `Region` and `Product`.

```sql
SELECT
    Region,
    Product,
    SUM(SalesAmount) AS TotalSales,
    GROUPING(Region) AS IsRegionSubtotal,
    GROUPING(Product) AS IsProductSubtotal,
    GROUPING_ID(Region, Product) AS GroupingLevel
FROM
    Sales
GROUP BY
    GROUPING SETS (
        (Region),      -- Group by Region
        (Product),     -- Group by Product
        ()             -- Grand total
    )
ORDER BY
    GroupingLevel, Region, Product;
```

**Expected Output:**

| Region  | Product  | TotalSales | IsRegionSubtotal | IsProductSubtotal | GroupingLevel |
| :------ | :------- | :--------- | :--------------- | :---------------- | :------------ |
| Central | NULL     | 960.00     | 0                | 1                 | 1             |
| East    | NULL     | 2150.00    | 0                | 1                 | 1             |
| West    | NULL     | 1275.00    | 0                | 1                 | 1             |
| NULL    | Keyboard | 75.00      | 1                | 0                 | 2             |
| NULL    | Laptop   | 4200.00    | 1                | 0                 | 2             |
| NULL    | Mouse    | 110.00     | 1                | 0                 | 2             |
| NULL    | NULL     | 4385.00    | 1                | 1                 | 3             |

**Explanation:**
*   `GroupingLevel = 1`: Corresponds to `(Region)` grouping. `Product` is aggregated, so `IsProductSubtotal` is `1`.
*   `GroupingLevel = 2`: Corresponds to `(Product)` grouping. `Region` is aggregated, so `IsRegionSubtotal` is `1`.
*   `GroupingLevel = 3`: Corresponds to `()` (grand total). Both `Region` and `Product` are aggregated.

#### Scenario 2: Emulating `CUBE` with `GROUPING SETS`

`CUBE(Region, Product)` is equivalent to:

```sql
GROUP BY GROUPING SETS (
    (Region, Product),
    (Region),
    (Product),
    ()
);
```

Let's run this:

```sql
SELECT
    Region,
    Product,
    SUM(SalesAmount) AS TotalSales,
    GROUPING(Region) AS IsRegionSubtotal,
    GROUPING(Product) AS IsProductSubtotal,
    GROUPING_ID(Region, Product) AS GroupingLevel
FROM
    Sales
GROUP BY
    GROUPING SETS (
        (Region, Product),
        (Region),
        (Product),
        ()
    )
ORDER BY
    GroupingLevel, Region, Product;
```

**Expected Output (similar to the `CUBE` example, but with `GroupingLevel`):**

| Region  | Product  | TotalSales | IsRegionSubtotal | IsProductSubtotal | GroupingLevel |
| :------ | :------- | :--------- | :--------------- | :---------------- | :------------ |
| Central | Laptop   | 900.00     | 0                | 0                 | 0             |
| Central | Mouse    | 60.00      | 0                | 0                 | 0             |
| East    | Laptop   | 2100.00    | 0                | 0                 | 0             |
| East    | Mouse    | 50.00      | 0                | 0                 | 0             |
| West    | Keyboard | 75.00      | 0                | 0                 | 0             |
| West    | Laptop   | 1200.00    | 0                | 0                 | 0             |
| Central | NULL     | 960.00     | 0                | 1                 | 1             |
| East    | NULL     | 2150.00    | 0                | 1                 | 1             |
| West    | NULL     | 1275.00    | 0                | 1                 | 1             |
| NULL    | Keyboard | 75.00      | 1                | 0                 | 2             |
| NULL    | Laptop   | 4200.00    | 1                | 0                 | 2             |
| NULL    | Mouse    | 110.00     | 1                | 0                 | 2             |
| NULL    | NULL     | 4385.00    | 1                | 1                 | 3             |

**Explanation:**
*   `GroupingLevel = 0`: Corresponds to `(Region, Product)` (base grouping).
*   `GroupingLevel = 1`: Corresponds to `(Region)` (Product aggregated).
*   `GroupingLevel = 2`: Corresponds to `(Product)` (Region aggregated).
*   `GroupingLevel = 3`: Corresponds to `()` (grand total).

#### Scenario 3: Emulating `ROLLUP` with `GROUPING SETS`

`ROLLUP(Region, Product)` is equivalent to:

```sql
GROUP BY GROUPING SETS (
    (Region, Product),
    (Region),
    ()
);
```

Let's run this:

```sql
SELECT
    Region,
    Product,
    SUM(SalesAmount) AS TotalSales,
    GROUPING(Region) AS IsRegionSubtotal,
    GROUPING(Product) AS IsProductSubtotal,
    GROUPING_ID(Region, Product) AS GroupingLevel
FROM
    Sales
GROUP BY
    GROUPING SETS (
        (Region, Product),
        (Region),
        ()
    )
ORDER BY
    GroupingLevel, Region, Product;
```

**Expected Output:**

| Region    | Product  | TotalSales | IsRegionSubtotal | IsProductSubtotal | GroupingLevel |
| :-------- | :------- | :--------- | :--------------- | :---------------- | :------------ |
| Central   | Laptop   | 900.00     | 0                | 0                 | 0             |
| Central   | Mouse    | 60.00      | 0                | 0                 | 0             |
| East      | Laptop   | 2100.00    | 0                | 0                 | 0             |
| East      | Mouse    | 50.00      | 0                | 0                 | 0             |
| West      | Keyboard | 75.00      | 0                | 0                 | 0             |
| West      | Laptop   | 1200.00    | 0                | 0                 | 0             |
| Central   | NULL     | 960.00     | 0                | 1                 | 1             |
| East      | NULL     | 2150.00    | 0                | 1                 | 1             |
| West      | NULL     | 1275.00    | 0                | 1                 | 1             |
| NULL      | NULL     | 4385.00    | 1                | 1                 | 3             |

**Explanation:**
*   Notice that the `(Product)` only grouping from `CUBE` is missing, as `ROLLUP` follows a hierarchy.
*   `GroupingLevel = 0`: Corresponds to `(Region, Product)`.
*   `GroupingLevel = 1`: Corresponds to `(Region)`.
*   `GroupingLevel = 3`: Corresponds to `()` (grand total).

#### Scenario 4: Complex, Non-Hierarchical Grouping

Imagine you have `Region`, `Product`, and `SalesPerson` columns. You want to see:
*   Sales by `Region` and `Product`.
*   Sales by `SalesPerson` only.
*   A grand total.

Neither `CUBE` nor `ROLLUP` can achieve this specific combination directly without generating many unwanted groupings. `GROUPING SETS` is perfect here.

Let's add `SalesPerson` to our table:

```sql
ALTER TABLE Sales ADD SalesPerson VARCHAR(50);

UPDATE Sales SET SalesPerson = 'Alice' WHERE Region = 'East' AND Product = 'Laptop';
UPDATE Sales SET SalesPerson = 'Bob' WHERE Region = 'East' AND Product = 'Mouse';
UPDATE Sales SET SalesPerson = 'Alice' WHERE Region = 'West' AND Product = 'Laptop';
UPDATE Sales SET SalesPerson = 'Charlie' WHERE Region = 'West' AND Product = 'Keyboard';
UPDATE Sales SET SalesPerson = 'Bob' WHERE Region = 'Central' AND Product = 'Laptop';
UPDATE Sales SET SalesPerson = 'Charlie' WHERE Region = 'Central' AND Product = 'Mouse';
UPDATE Sales SET SalesPerson = 'Alice' WHERE Region = 'East' AND Product = 'Laptop' AND SalesAmount = 1100.00;
```

Now, the query:

```sql
SELECT
    Region,
    Product,
    SalesPerson,
    SUM(SalesAmount) AS TotalSales,
    GROUPING(Region) AS IsRegionSubtotal,
    GROUPING(Product) AS IsProductSubtotal,
    GROUPING(SalesPerson) AS IsSalesPersonSubtotal,
    GROUPING_ID(Region, Product, SalesPerson) AS GroupingLevel
FROM
    Sales
GROUP BY
    GROUPING SETS (
        (Region, Product),   -- Group by Region and Product
        (SalesPerson),       -- Group by SalesPerson
        ()                   -- Grand total
    )
ORDER BY
    GroupingLevel, Region, Product, SalesPerson;
```

**Expected Output (partial, as it can be long):**

| Region    | Product  | SalesPerson | TotalSales | IsRegionSubtotal | IsProductSubtotal | IsSalesPersonSubtotal | GroupingLevel |
| :-------- | :------- | :---------- | :--------- | :--------------- | :---------------- | :-------------------- | :------------ |
| Central   | Laptop   | NULL        | 900.00     | 0                | 0                 | 1                     | 1             |
| Central   | Mouse    | NULL        | 60.00      | 0                | 0                 | 1                     | 1             |
| East      | Laptop   | NULL        | 2100.00    | 0                | 0                 | 1                     | 1             |
| East      | Mouse    | NULL        | 50.00      | 0                | 0                 | 1                     | 1             |
| West      | Keyboard | NULL        | 75.00      | 0                | 0                 | 1                     | 1             |
| West      | Laptop   | NULL        | 1200.00    | 0                | 0                 | 1                     | 1             |
| NULL      | NULL     | Alice       | 2300.00    | 1                | 1                 | 0                     | 6             |
| NULL      | NULL     | Bob         | 950.00     | 1                | 1                 | 0                     | 6             |
| NULL      | NULL     | Charlie     | 135.00     | 1                | 1                 | 0                     | 6             |
| NULL      | NULL     | NULL        | 4385.00    | 1                | 1                 | 1                     | 7             |

**Explanation:**
*   The first set of rows (`GroupingLevel = 1`) shows sales by `Region` and `Product`. `SalesPerson` is `NULL` and `IsSalesPersonSubtotal` is `1` because it's aggregated out.
*   The next set of rows (`GroupingLevel = 6`) shows sales by `SalesPerson`. `Region` and `Product` are `NULL` and their `GROUPING` flags are `1`.
*   The last row (`GroupingLevel = 7`) is the grand total.

### 7. Benefits and Use Cases

*   **Single Query Efficiency**: Often more efficient than `UNION ALL` because the database can scan the base table once and perform multiple aggregations in memory.
*   **Readability**: A single, concise query is easier to read and maintain than multiple `UNION ALL` statements.
*   **Flexibility**: Allows for highly customized aggregation reports, combining specific dimensions as needed.
*   **Analytical Reporting**: Ideal for business intelligence and data warehousing scenarios where users need to view data at various levels of granularity without writing complex, repetitive queries.
*   **Ad-hoc Analysis**: Quickly generate different summary views of data for exploration.

In conclusion, `GROUPING SETS` is the most versatile of the `GROUP BY` extensions. While `CUBE` and `ROLLUP` offer convenient shortcuts for common aggregation patterns, `GROUPING SETS` provides the ultimate control, allowing you to define precisely which combinations of columns you want to aggregate, making it an indispensable tool for advanced SQL reporting and analysis. Always remember to leverage `GROUPING()` or `GROUPING_ID()` to correctly interpret the `NULL` values in your result set.