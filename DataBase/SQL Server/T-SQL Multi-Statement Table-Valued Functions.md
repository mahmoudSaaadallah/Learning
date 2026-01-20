### Multi-Statement Table-Valued Functions (MSTVFs): Flexibility with Caution

A **Multi-Statement Table-Valued Function (MSTVF)** is another type of User-Defined Function (UDF) in SQL Server that returns a `TABLE` data type. However, unlike Inline Table-Valued Functions [[T-SQL Inline Table-Valued Functions]] (ITVFs), MSTVFs allow for a more complex function body, containing multiple T-SQL statements, variables, conditional logic, loops, and even temporary tables, before populating and returning a table variable.

#### What is a Multi-Statement Table-Valued Function?

An MSTVF is a UDF that:
1.  Accepts zero or more input parameters.
2.  Declares a table variable as its return type.
3.  Executes a block of T-SQL code (enclosed in `BEGIN...END`) to populate this table variable.
4.  Returns the populated table variable.

#### Purpose and Advantages

MSTVFs offer greater flexibility than ITVFs, making them suitable for scenarios where the logic cannot be expressed in a single `SELECT` statement:
*   **Complex Logic**: When you need to perform multiple steps, use conditional logic (`IF/ELSE`), loops (`WHILE`), or declare intermediate variables to derive your final result set.
*   **Temporary Storage**: You can declare and use table variables within the function body to store intermediate results, which can be useful for complex aggregations or data transformations.
*   **Encapsulation of Procedural Logic**: They allow you to encapsulate procedural T-SQL logic that ultimately produces a tabular result.
*   **Code Reusability**: Like other UDFs, they promote code reuse and modularity.

#### Syntax for Creating a Multi-Statement Table-Valued Function

The basic syntax for creating an MSTVF is:

```sql
CREATE FUNCTION [schema_name.]function_name
(
    [@parameter_name] [data_type] [= default_value] [,...n]
)
RETURNS @return_table_variable TABLE
(
    column1_name data_type [NULL|NOT NULL],
    column2_name data_type [NULL|NOT NULL],
    -- ... more columns
    PRIMARY KEY (column_name) -- Optional, but good practice for performance
)
AS
BEGIN
    -- Multiple T-SQL statements to populate @return_table_variable
    -- e.g., INSERT INTO @return_table_variable SELECT ...
    --       UPDATE @return_table_variable SET ...
    --       IF/ELSE, WHILE loops, etc.

    RETURN; -- Returns the populated table variable
END;
GO
```

**Key Components**:
*   `schema_name.function_name`: The name of your function.
*   `@parameter_name data_type`: Input parameters.
*   `RETURNS @return_table_variable TABLE (...)`: This is the defining characteristic. You *must* explicitly define the schema (column names and data types) of the table variable that the function will return.
*   `BEGIN...END`: The function body, containing one or more T-SQL statements.
*   `RETURN;`: This statement simply signifies the end of the function execution and returns the populated table variable.

#### Detailed Examples

Let's illustrate with some practical examples where an MSTVF might be considered.

**Example 1: Calculating Employee Performance Scores with Conditional Logic**

Imagine a scenario where an employee's performance score is calculated based on multiple factors and conditional rules.

```sql
-- Drop the function if it already exists
IF OBJECT_ID('dbo.CalculateEmployeePerformance') IS NOT NULL
    DROP FUNCTION dbo.CalculateEmployeePerformance;
GO

CREATE FUNCTION dbo.CalculateEmployeePerformance
(
    @MinSalesTarget DECIMAL(18, 2) = 50000.00,
    @MinProjectsCompleted INT = 3
)
RETURNS @EmployeePerformance TABLE
(
    EmployeeID INT,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    SalesYTD DECIMAL(18, 2),
    ProjectsCompleted INT,
    PerformanceScore INT,
    PerformanceCategory NVARCHAR(50)
)
AS
BEGIN
    -- Step 1: Get base employee data
    INSERT INTO @EmployeePerformance (EmployeeID, FirstName, LastName, SalesYTD, ProjectsCompleted, PerformanceScore, PerformanceCategory)
    SELECT
        e.EmployeeID,
        e.FirstName,
        e.LastName,
        e.SalesYTD,
        COUNT(ep.ProjectID) AS ProjectsCompleted,
        0 AS PerformanceScore, -- Initialize score
        'Uncategorized' AS PerformanceCategory -- Initialize category
    FROM
        Employees AS e
    LEFT JOIN
        EmployeeProjects AS ep ON e.EmployeeID = ep.EmployeeID
    GROUP BY
        e.EmployeeID, e.FirstName, e.LastName, e.SalesYTD;

    -- Step 2: Apply conditional logic to calculate score and category
    UPDATE @EmployeePerformance
    SET
        PerformanceScore =
            CASE
                WHEN SalesYTD >= @MinSalesTarget AND ProjectsCompleted >= @MinProjectsCompleted THEN 100
                WHEN SalesYTD >= @MinSalesTarget THEN 75
                WHEN ProjectsCompleted >= @MinProjectsCompleted THEN 60
                ELSE 30
            END,
        PerformanceCategory =
            CASE
                WHEN SalesYTD >= @MinSalesTarget AND ProjectsCompleted >= @MinProjectsCompleted THEN 'Excellent'
                WHEN SalesYTD >= @MinSalesTarget THEN 'Good Sales'
                WHEN ProjectsCompleted >= @MinProjectsCompleted THEN 'Good Projects'
                ELSE 'Needs Improvement'
            END;

    RETURN;
END;
GO

-- How to use it:
SELECT *
FROM dbo.CalculateEmployeePerformance(60000.00, 4); -- Custom targets
```
*(Assuming `Employees` and `EmployeeProjects` tables exist with `SalesYTD` column)*

**Example 2: Generating a Date Range Table**

This function generates a table of dates within a specified range, which can be useful for reporting or calendar-related tasks.

```sql
IF OBJECT_ID('dbo.GenerateDateRange') IS NOT NULL
    DROP FUNCTION dbo.GenerateDateRange;
GO

CREATE FUNCTION dbo.GenerateDateRange
(
    @StartDate DATE,
    @EndDate DATE
)
RETURNS @DateRange TABLE
(
    DateValue DATE PRIMARY KEY CLUSTERED,
    DayOfWeekName NVARCHAR(10),
    IsWeekend BIT
)
AS
BEGIN
    DECLARE @CurrentDate DATE = @StartDate;

    WHILE @CurrentDate <= @EndDate
    BEGIN
        INSERT INTO @DateRange (DateValue, DayOfWeekName, IsWeekend)
        VALUES
        (
            @CurrentDate,
            DATENAME(dw, @CurrentDate),
            CASE WHEN DATENAME(dw, @CurrentDate) IN ('Saturday', 'Sunday') THEN 1 ELSE 0 END
        );

        SET @CurrentDate = DATEADD(day, 1, @CurrentDate);
    END;

    RETURN;
END;
GO

-- How to use it:
SELECT *
FROM dbo.GenerateDateRange('2026-01-01', '2026-01-10');
```

#### Critical Performance Considerations and Drawbacks

This is where the "caution" part of MSTVFs comes into play. While flexible, MSTVFs have significant performance limitations that make them generally less desirable than ITVFs for performance-critical operations:

1.  **No Inlining**: Unlike ITVFs, the SQL Server query optimizer *cannot* inline the logic of an MSTVF into the calling query. It treats the MSTVF as a "black box."
2.  **Fixed Cardinality Estimates**: The optimizer has very poor cardinality estimates for MSTVFs. Historically, it would often assume a fixed number of rows (e.g., 1 or 100 rows) are returned, regardless of the actual data. This leads to highly inefficient query plans, especially when the MSTVF is joined with other large tables.
3.  **No Predicate Pushdown**: Filters applied to the output of an MSTVF in the `WHERE` clause of the calling query cannot be "pushed down" into the function's internal logic. This means the function will often generate its *entire* result set, and *then* the outer query will filter it, leading to much more work than necessary.
4.  **No Parallelism**: Queries involving MSTVFs are typically forced into a serial execution plan, preventing the benefits of parallel processing on multi-core systems.
5.  **Context Switching Overhead**: Each call to an MSTVF involves context switching, similar to scalar UDFs, adding overhead.
6.  **No Index Usage on Table Variable**: While you can define a `PRIMARY KEY` or `UNIQUE` constraint on the table variable within the MSTVF, these are primarily for data integrity and do not translate into usable indexes for the *outer* query optimizer. The optimizer cannot use indexes on the table variable to optimize joins or filters from the calling query.

**Impact of SQL Server 2017/2019 and Beyond**:
SQL Server 2017 introduced **interleaved execution** for MSTVFs, which helps with cardinality estimates by running the MSTVF once at compile time to get an actual row count. SQL Server 2019 further enhanced this. While these improvements are beneficial, they do not fundamentally change the "no inlining" and "no predicate pushdown" issues. MSTVFs still generally perform worse than ITVFs or other set-based alternatives.

#### Alternatives to MSTVFs

Given the performance drawbacks, it's often recommended to explore alternatives before resorting to an MSTVF:

*   **Inline Table-Valued Functions (ITVFs)**: If your logic can be expressed in a single `SELECT` statement (even a complex one with CTEs, subqueries, and joins), an ITVF is almost always the superior choice due to inlining.
*   **Views**: For non-parameterized tabular results, a view is a good option.
*   **Common Table Expressions (CTEs)**: For complex, multi-step logic within a single query, CTEs are excellent for breaking down the problem into readable, optimized steps.
*   **Stored Procedures**: If you need to perform data modifications, complex procedural logic, or return multiple result sets, a stored procedure is the appropriate choice. Stored procedures also benefit from full optimization.
*   **Temporary Tables or Table Variables (within a stored procedure or batch)**: For very complex intermediate steps, explicitly creating and populating temporary tables or table variables within a stored procedure can offer better control and performance than an MSTVF.

#### When to (Carefully) Use an MSTVF

Despite the warnings, there are niche scenarios where an MSTVF might be considered:
*   **Small Datasets**: If the function will always operate on and return a very small number of rows, the performance overhead might be negligible.
*   **Complex Procedural Logic**: When the logic is genuinely procedural (e.g., involving loops, multiple conditional updates to intermediate data) and cannot be refactored into a single `SELECT` statement or a stored procedure that returns a result set.
*   **Legacy Code**: Maintaining existing MSTVFs where refactoring is not feasible.
*   **Readability/Encapsulation over Absolute Performance**: In some non-critical reporting scenarios, the benefit of encapsulating complex logic for readability might outweigh the performance cost.

### Conclusion

Multi-Statement Table-Valued Functions offer unparalleled flexibility for encapsulating complex, procedural T-SQL logic that returns a tabular result. However, this flexibility comes at a significant performance cost due to the optimizer's inability to inline their logic and accurately estimate cardinality. As a senior database developer, it's crucial to understand these limitations and always prioritize Inline Table-Valued Functions or other set-based alternatives whenever possible. Use MSTVFs sparingly, and only after careful consideration and performance testing, especially in high-volume or performance-critical environments.