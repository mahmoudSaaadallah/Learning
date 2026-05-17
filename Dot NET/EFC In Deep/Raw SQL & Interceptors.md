### When EF Core's LINQ Generation Isn't Enough

EF Core's LINQ provider is excellent at translating common LINQ patterns into efficient SQL. However, it has limitations:

1.  **Complex or Highly Optimized Queries:**
    *   **Database-Specific Features:** Some advanced database features (e.g., specific window functions, recursive CTEs, full-text search syntax, advanced spatial queries) might not have direct LINQ equivalents or might be translated inefficiently.
    *   **Performance Tuning:** For extremely performance-critical sections, a hand-tuned SQL query written by a DBA or an experienced SQL developer might outperform EF Core's generated SQL.
    *   **Complex Joins/Subqueries:** While LINQ can handle many joins, very intricate join patterns or deeply nested subqueries can sometimes be clearer and more performant when written directly in SQL.

2.  **Stored Procedures:**
    *   When you need to execute a stored procedure for business logic, data manipulation, or complex reporting that resides on the database server.

3.  **Legacy Database Interactions:**
    *   Working with existing databases where specific views, functions, or stored procedures are already defined and must be used.
    *   When you need to query tables that don't have a corresponding entity in your `DbContext` (though this is less common with modern EF Core).

4.  **Bulk Operations (Less Common with `FromSql`):**
    *   While `FromSql` is primarily for *reading* data, sometimes you might need to execute raw `INSERT`, `UPDATE`, or `DELETE` statements for bulk operations that are more efficient than loading entities into memory and then saving changes. (For this, `ExecuteSqlRaw` or `ExecuteSqlInterpolated` are used, which we can cover if you'd like).

### Using `FromSqlRaw` and `FromSqlInterpolated`

These methods allow you to execute raw SQL queries that return entity types. The key is that the SQL query *must* return columns that map to the properties of the entity type you're querying, and the column names *must* match the property names (or be aliased to match).

#### Common Constraints for `FromSqlRaw`/`FromSqlInterpolated`:

-   The SQL query must return columns that match the properties of the entity type being queried.
-   The column names in the result set must match the property names (case-insensitive for most databases, but best to match exactly).
-   The entity type must have a primary key defined.
-   The query must return all properties that constitute the primary key.

#### 1. `FromSqlRaw(string sql, params object[] parameters)`

-   **Purpose:** Executes a raw SQL query using a format string and an array of parameters.
-   **Security Concern:** If you concatenate parameters directly into the `sql` string, you are vulnerable to **SQL Injection**. You *must* pass parameters as separate `object[]` arguments. EF Core will then correctly parameterize them.
-   **Use Case:** When you have a fixed SQL string and need to pass dynamic values safely.

**Example with `FromSqlRaw`:**

Let's continue with our `Student` entity.

```csharp
using System;
using System.Linq;
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;

public class RawSqlExample
{
    public static void Run()
    {
        using (var context = new AppDbContext())
        {
            context.Database.EnsureCreated();
            if (!context.Students.Any())
            {
                context.Students.AddRange(
                    new Student { FirstName = "Alice", LastName = "Smith", Age = 20, GPA = 3.8M },
                    new Student { FirstName = "Bob", "LastName = "Johnson", Age = 22, GPA = 3.5M },
                    new Student { FirstName = "Charlie", LastName = "Brown", Age = 21, GPA = 3.9M },
                    new Student { FirstName = "Diana", LastName = "Prince", Age = 20, GPA = 3.7M },
                    new Student { FirstName = "Eve", LastName = "Adams", Age = 23, GPA = 3.2M }
                );
                context.SaveChanges();
            }

            Console.WriteLine("--- Using FromSqlRaw ---");

            // Scenario 1: Simple query with a parameter
            string firstNameParam = "Alice";
            var studentsRaw = context.Students
                                     .FromSqlRaw("SELECT * FROM Students WHERE FirstName = {0}", firstNameParam)
                                     .ToList();

            Console.WriteLine($"Students found using FromSqlRaw for '{firstNameParam}':");
            foreach (var student in studentsRaw)
            {
                Console.WriteLine($"  - {student.FirstName} {student.LastName}, Age: {student.Age}");
            }

            // Scenario 2: Chaining LINQ after FromSqlRaw
            // EF Core will compose the LINQ operators onto the raw SQL query.
            // The raw SQL becomes a subquery.
            var youngStudentsRaw = context.Students
                                          .FromSqlRaw("SELECT * FROM Students") // Get all students first
                                          .Where(s => s.Age < 22) // Then filter in LINQ
                                          .OrderBy(s => s.LastName)
                                          .ToList();

            Console.WriteLine("\nYoung students (Age < 22) using FromSqlRaw + LINQ:");
            foreach (var student in youngStudentsRaw)
            {
                Console.WriteLine($"  - {student.FirstName} {student.LastName}, Age: {student.Age}");
            }

            // What SQL was generated for Scenario 2? Something like:
            // SELECT [s].[Id], [s].[Age], [s].[FirstName], [s].[GPA], [s].[LastName]
            // FROM (
            //     SELECT * FROM Students
            // ) AS [s]
            // WHERE [s].[Age] < 22
            // ORDER BY [s].[LastName]
        }
    }
}
```

#### 2. `FromSqlInterpolated(FormattableString sql)`

-   **Purpose:** Executes a raw SQL query using C# 6 interpolated strings.
-   **Security Benefit:** This is the **recommended** method for raw SQL queries with parameters because it automatically converts interpolated string parameters into `DbParameter` objects, making it inherently safe against **SQL Injection**.
-   **Readability:** Often more readable than `FromSqlRaw` as parameters are directly embedded in the string.

**Example with `FromSqlInterpolated`:**

```csharp
using System;
using System.Linq;
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;

public class InterpolatedSqlExample
{
    public static void Run()
    {
        using (var context = new AppDbContext())
        {
            context.Database.EnsureCreated();
            // Assume students are already seeded from previous example

            Console.WriteLine("\n--- Using FromSqlInterpolated ---");

            // Scenario 1: Simple query with interpolated parameters (SAFE!)
            string lastNameParam = "Brown";
            int minAgeParam = 20;
            var studentsInterpolated = context.Students
                                              .FromSqlInterpolated($"SELECT * FROM Students WHERE LastName = {lastNameParam} AND Age >= {minAgeParam}")
                                              .ToList();

            Console.WriteLine($"Students found using FromSqlInterpolated for '{lastNameParam}' and Age >= {minAgeParam}:");
            foreach (var student in studentsInterpolated)
            {
                Console.WriteLine($"  - {student.FirstName} {student.LastName}, Age: {student.Age}");
            }

            // Scenario 2: Calling a Stored Procedure
            // Note: Stored procedures must return columns that map to the entity properties.
            // If the SP returns a different shape, you might need to map it to a keyless entity type or a DTO.
            string spName = "GetStudentsByAge"; // Assume you have a stored procedure like:
                                                // CREATE PROCEDURE GetStudentsByAge @MinAge INT
                                                // AS
                                                // SELECT * FROM Students WHERE Age >= @MinAge
            int ageThreshold = 22;
            var studentsFromSp = context.Students
                                        .FromSqlInterpolated($"EXEC {spName} @MinAge = {ageThreshold}")
                                        .ToList();

            Console.WriteLine($"\nStudents found using Stored Procedure '{spName}' with Age >= {ageThreshold}:");
            foreach (var student in studentsFromSp)
            {
                Console.WriteLine($"  - {student.FirstName} {student.LastName}, Age: {student.Age}");
            }
        }
    }
}
```

### Interceptors: Extending EF Core's Behavior

Interceptors provide hooks into EF Core's operations, allowing you to observe, modify, or suppress behavior. They are incredibly powerful for advanced scenarios like:

-   **Logging:** Capturing all SQL commands executed by EF Core, including raw SQL.
-   **Auditing:** Adding audit information (e.g., `CreatedBy`, `ModifiedBy`) automatically.
-   **Performance Monitoring:** Measuring query execution times.
-   **Query Rewriting:** Modifying SQL commands before they are sent to the database (e.g., adding `WHERE` clauses for multi-tenancy).
-   **Error Handling:** Customizing how EF Core reacts to database errors.

There are various types of interceptors, such as `IDbCommandInterceptor`, `IDbConnectionInterceptor`, `ISaveChangesInterceptor`, etc.

**Example: Simple Command Interceptor for Logging**

```csharp
using Microsoft.EntityFrameworkCore.Diagnostics;
using System.Data.Common;
using System.Threading;
using System.Threading.Tasks;
using System;

public class CommandInterceptor : IDbCommandInterceptor
{
    public InterceptionResult<DbDataReader> ReaderExecuting(
        DbCommand command,
        CommandEventData eventData,
        InterceptionResult<DbDataReader> result)
    {
        Console.WriteLine($"\n--- Interceptor: Executing SQL Command ---");
        Console.WriteLine($"  Command Text: {command.CommandText}");
        Console.WriteLine($"  Parameters: {string.Join(", ", command.Parameters.Cast<DbParameter>().Select(p => $"{p.ParameterName} = {p.Value}"))}");
        Console.WriteLine($"  Connection: {command.Connection.ConnectionString}");
        Console.WriteLine($"------------------------------------------");
        return result;
    }

    public ValueTask<InterceptionResult<DbDataReader>> ReaderExecutingAsync(
        DbCommand command,
        CommandEventData eventData,
        InterceptionResult<DbDataReader> result,
        CancellationToken cancellationToken = default)
    {
        Console.WriteLine($"\n--- Interceptor (Async): Executing SQL Command ---");
        Console.WriteLine($"  Command Text: {command.CommandText}");
        Console.WriteLine($"  Parameters: {string.Join(", ", command.Parameters.Cast<DbParameter>().Select(p => $"{p.ParameterName} = {p.Value}"))}");
        Console.WriteLine($"  Connection: {command.Connection.ConnectionString}");
        Console.WriteLine($"---------------------------------------------------");
        return ValueTask.FromResult(result);
    }

    // Implement other methods as needed (e.g., NonQueryExecuting, ScalarExecuting)
    // For brevity, I'm only showing ReaderExecuting here.
}

// To register the interceptor in your DbContext:
public class AppDbContextWithInterceptor : DbContext
{
    public DbSet<Student> Students { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=SchoolDB;Trusted_Connection=True;")
                      .AddInterceptors(new CommandInterceptor()); // Register your interceptor here
    }
}
```

Now, if you use `AppDbContextWithInterceptor`, every time a database command is executed (including those from `FromSqlRaw` or `FromSqlInterpolated`), your `CommandInterceptor` will log the SQL.

### Senior Developer Considerations:

1.  **Prioritize LINQ:** Always try to use LINQ-to-Entities first. It's more maintainable, type-safe, and generally database-agnostic.
2.  **Use Raw SQL Judiciously:** Reserve `FromSqlRaw`/`FromSqlInterpolated` for scenarios where LINQ is genuinely insufficient or leads to significantly suboptimal performance.
3.  **SQL Injection Prevention:** **Always** use parameterized queries. `FromSqlInterpolated` is the safest and most readable way to do this. Never concatenate user input directly into a SQL string.
4.  **Portability:** Be aware that raw SQL ties you to a specific database syntax. If you ever need to switch database providers, these raw SQL queries will likely need to be rewritten.
5.  **Testing:** Raw SQL queries are not compile-time checked by EF Core. Thorough testing is crucial to ensure they are correct and perform as expected.
6.  **Interceptors for Cross-Cutting Concerns:** Use interceptors for concerns that cut across your data access layer, like logging, auditing, or multi-tenancy filtering, rather than scattering that logic throughout your application.

Mastering when and how to use raw SQL, along with understanding the power of interceptors, demonstrates a comprehensive grasp of EF Core and the ability to make informed architectural and performance decisions.
