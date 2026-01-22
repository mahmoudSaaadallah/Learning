### Understanding `ROLLUP` in SQL Server

The `ROLLUP` operator is an extension to the `GROUP BY` clause in SQL Server (and standard SQL). Its primary purpose is to generate **subtotals for hierarchies of columns** specified in the `GROUP BY` list, along with a **grand total**. It essentially produces a result set that contains the aggregates that would be returned by a simple `GROUP BY` clause, plus additional rows for subtotals and a grand total.

Think of it as generating a summary report where you want to see totals at various levels of detail, from the most granular grouping all the way up to an overall total.

#### How `ROLLUP` Works

When you use `ROLLUP(column1, column2, ..., columnN)`, it generates a result set that includes:

1.  **Standard `GROUP BY` rows:** Aggregates for all specified columns (`column1, column2, ..., columnN`).
2.  **Subtotal rows:** Aggregates for `(column1, column2, ..., columnN-1)`, then `(column1, column2, ..., columnN-2)`, and so on, until `(column1)`.
3.  **Grand total row:** An aggregate for all rows, effectively `()`.

The order of columns within `ROLLUP` matters, as it defines the hierarchy for which subtotals are generated. `ROLLUP(A, B)` is different from `ROLLUP(B, A)`. `ROLLUP(A, B)` will produce subtotals for `(A, B)`, `(A)`, and `()`.

#### Syntax

The `ROLLUP` operator is used within the `GROUP BY` clause:

```sql
SELECT
    column1,
    column2,
    -- ...
    aggregate_function(columnX) AS Total
FROM
    YourTable
GROUP BY
    ROLLUP(column1, column2, ..., columnN);
```

#### Identifying `ROLLUP` Rows with `GROUPING` and `GROUPING_ID`

When `ROLLUP` generates subtotal or grand total rows, the columns that are being "rolled up" will have `NULL` values in those rows. To distinguish these `NULL`s (which represent an aggregate level) from actual `NULL` data values in your table, SQL Server provides two useful functions:

1.  **`GROUPING(column_name)`:** Returns `1` if the column is being aggregated (i.e., it's a subtotal or grand total row for that column), and `0` if the column is part of the grouping.
2.  **`GROUPING_ID(column1, column2, ...)`:** Returns an integer representing the level of aggregation. It's a bitmap where each bit corresponds to a column in the `GROUP BY` list. A `1` in a bit position means that column is aggregated, and `0` means it's part of the grouping. This is particularly useful for ordering or filtering specific aggregation levels.

---

### Detailed Examples

Let's use a hypothetical `Sales` table to illustrate.

**`Sales` Table Structure:**

```sql
CREATE TABLE Sales (
    Region VARCHAR(50),
    ProductCategory VARCHAR(50),
    ProductSubCategory VARCHAR(50),
    SalesAmount DECIMAL(10, 2)
);

INSERT INTO Sales (Region, ProductCategory, ProductSubCategory, SalesAmount) VALUES
('East', 'Electronics', 'Laptops', 1500.00),
('East', 'Electronics', 'Laptops', 1200.00),
('East', 'Electronics', 'Smartphones', 800.00),
('East', 'Clothing', 'Shirts', 300.00),
('West', 'Electronics', 'Laptops', 1800.00),
('West', 'Electronics', 'Smartphones', 950.00),
('West', 'Clothing', 'Pants', 450.00),
('Central', 'Clothing', 'Shirts', 200.00),
('Central', 'Electronics', 'Laptops', 1000.00);
```

#### Example 1: Basic `ROLLUP`

Let's get sales totals by `Region`, then by `ProductCategory`, and finally a grand total.

```sql
SELECT
    Region,
    ProductCategory,
    SUM(SalesAmount) AS TotalSales
FROM
    Sales
GROUP BY
    ROLLUP(Region, ProductCategory);
```

**Expected Output (conceptual):**

| Region  | ProductCategory | TotalSales | --                             |
| :------ | :-------------- | :--------- | ------------------------------ |
| Central | Clothing        | 200.00     |                                |
| Central | Electronics     | 1000.00    |                                |
| Central | NULL            | 1200.00    | -- Subtotal for Central Region |
| East    | Clothing        | 300.00     |                                |
| East    | Electronics     | 3500.00    |                                |
| East    | NULL            | 3800.00    | -- Subtotal for East Region    |
| West    | Clothing        | 450.00     |                                |
| West    | Electronics     | 2750.00    |                                |
| West    | NULL            | 3200.00    | -- Subtotal for West Region    |
| NULL    | NULL            | 8200.00    | -- Grand Total                 |

Notice how `NULL` appears in `ProductCategory` for region subtotals, and in both `Region` and `ProductCategory` for the grand total.

#### Example 2: Using `GROUPING` to Identify Aggregation Levels

To make the output more readable and distinguish actual `NULL` data from `ROLLUP` generated `NULL`s, we can use `GROUPING`.

```sql
SELECT
    CASE WHEN GROUPING(Region) = 1 THEN 'All Regions' ELSE Region END AS Region,
    CASE WHEN GROUPING(ProductCategory) = 1 THEN 'All Categories' ELSE ProductCategory END AS ProductCategory,
    SUM(SalesAmount) AS TotalSales,
    GROUPING(Region) AS IsRegionTotal,
    GROUPING(ProductCategory) AS IsCategoryTotal
FROM
    Sales
GROUP BY
    ROLLUP(Region, ProductCategory)
ORDER BY
    GROUPING(Region), Region, GROUPING(ProductCategory), ProductCategory;
```

**Output:**

| Region      | ProductCategory | TotalSales | IsRegionTotal | IsCategoryTotal |
| :---------- | :-------------- | :--------- | :------------ | :-------------- |
| Central     | Clothing        | 200.00     | 0             | 0               |
| Central     | Electronics     | 1000.00    | 0             | 0               |
| Central     | All Categories  | 1200.00    | 0             | 1               |
| East        | Clothing        | 300.00     | 0             | 0               |
| East        | Electronics     | 3500.00    | 0             | 0               |
| East        | All Categories  | 3800.00    | 0             | 1               |
| West        | Clothing        | 450.00     | 0             | 0               |
| West        | Electronics     | 2750.00    | 0             | 0               |
| West        | All Categories  | 3200.00    | 0             | 1               |
| All Regions | All Categories  | 8200.00    | 1             | 1               |

This output is much clearer for reporting purposes. The `ORDER BY` clause using `GROUPING` helps to sort the subtotals and grand total logically at the end of their respective groups.

#### Example 3: Using `GROUPING_ID`

`GROUPING_ID` provides a single integer value that can be very useful for complex ordering or filtering.

```sql
SELECT
    Region,
    ProductCategory,
    SUM(SalesAmount) AS TotalSales,
    GROUPING_ID(Region, ProductCategory) AS GroupingLevel
FROM
    Sales
GROUP BY
    ROLLUP(Region, ProductCategory)
ORDER BY
    GroupingLevel, Region, ProductCategory;
```

**`GROUPING_ID` Interpretation:**
*   `GROUPING_ID(Region, ProductCategory)`:
    *   `0` (binary `00`): Both `Region` and `ProductCategory` are grouped (detail rows).
    *   `1` (binary `01`): `ProductCategory` is aggregated, `Region` is grouped (subtotal by `Region`).
    *   `3` (binary `11`): Both `Region` and `ProductCategory` are aggregated (grand total).

**Output (with `GroupingLevel`):**

| Region  | ProductCategory | TotalSales | GroupingLevel |
| :------ | :-------------- | :--------- | :------------ |
| Central | Clothing        | 200.00     | 0             |
| Central | Electronics     | 1000.00    | 0             |
| East    | Clothing        | 300.00     | 0             |
| East    | Electronics     | 3500.00    | 0             |
| West    | Clothing        | 450.00     | 0             |
| West    | Electronics     | 2750.00    | 0             |
| Central | NULL            | 1200.00    | 1             |
| East    | NULL            | 3800.00    | 1             |
| West    | NULL            | 3200.00    | 1             |
| NULL    | NULL            | 8200.00    | 3             |

---

### Use Cases for `ROLLUP`

*   **Hierarchical Reporting:** Generating reports that require subtotals at various levels of a hierarchy (e.g., Sales by Year, then Quarter, then Month).
*   **Summary Dashboards:** Providing aggregated data for business intelligence dashboards where users need to see totals at different granularities.
*   **Financial Analysis:** Summarizing financial data by department, cost center, and then overall.
*   **Data Warehousing:** Pre-calculating aggregates for OLAP cubes or summary tables.

---
[[T-SQL CUBE]]
[[T-SQL Grouping sets]]
### `ROLLUP` vs. `CUBE` vs. `GROUPING SETS`

It's important to understand how `ROLLUP` fits into the broader family of `GROUP BY` extensions:

*   **`ROLLUP(A, B, C)`:** Generates subtotals for the hierarchy `(A, B, C)`, `(A, B)`, `(A)`, and `()`. The order matters.
    *   Equivalent to `GROUPING SETS((A, B, C), (A, B), (A), ())`

*   **`CUBE(A, B, C)`:** Generates subtotals for *all possible combinations* of the specified columns, plus a grand total. This is useful when there's no inherent hierarchy, and you want to analyze data from every possible angle.
    *   Equivalent to `GROUPING SETS((A, B, C), (A, B), (A, C), (B, C), (A), (B), (C), ())`

*   **`GROUPING SETS((A, B), (C), (A, C))`:** Allows you to explicitly define *exactly which* grouping combinations you want. This is the most flexible option and can be used to achieve the results of both `ROLLUP` and `CUBE`, as well as any custom combination.

**When to choose `ROLLUP`:**
When you have a clear hierarchical relationship between your grouping columns and you need subtotals that follow that hierarchy, along with a grand total. It's more concise than `GROUPING SETS` for this specific pattern and more efficient than `CUBE` if you don't need all possible combinations.

---

In essence, `ROLLUP` is a powerful and elegant tool for generating multi-level aggregations in SQL Server. It simplifies complex reporting queries, making them more readable and often more performant than achieving the same results with multiple `UNION ALL` statements. Mastering `ROLLUP` (and its cousins `CUBE` and `GROUPING SETS`) is a key skill for anyone involved in data analysis and reporting with SQL Server.