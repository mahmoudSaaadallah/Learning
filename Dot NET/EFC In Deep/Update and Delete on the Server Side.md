## Update and Delete on the Server Side using `ExecuteDelete` and `ExecuteUpdate` in EF Core

### The "Why" - When to use these methods

Traditionally, to update or delete entities with EF Core, you would:
1.  Query the entities from the database.
2.  Modify their properties (for updates) or mark them for deletion.
3.  Call `_dbContext.SaveChanges()`.

This process involves:
*   **Loading data into memory**: Potentially many entities.
*   **Change tracking**: EF Core tracks every change.
*   **Round trips**: One to fetch, one to save.

For scenarios involving a large number of entities, or when you simply want to update/delete based on a condition without needing the entities' current state in your application, this traditional approach can be inefficient.

`ExecuteUpdate` and `ExecuteDelete` bypass these steps, generating a single `UPDATE` or `DELETE` SQL statement that is executed directly on the database server.

### 1. `ExecuteUpdate`

`ExecuteUpdate` allows you to update multiple entities in the database based on a query, without loading them into memory.

#### How it Works

You define a query that selects the entities to be updated, and then you specify the changes using an expression. EF Core translates this into a single `UPDATE` statement.

#### Syntax and Usage (Latest .NET/EF Core)

Let's imagine we have a `Product` entity:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public DateTime LastUpdated { get; set; }
    public bool IsActive { get; set; }
}

// In your DbContext
public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    // ... other DbSets
}
```

**Example 1: Increase price for all active products**

```csharp
using var context = new AppDbContext();

// Increase the price of all active products by 10% and update their LastUpdated timestamp
var affectedRows = await context.Products
    .Where(p => p.IsActive)
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(p => p.Price, p => p.Price * 1.10m) // Increase price by 10%
        .SetProperty(p => p.LastUpdated, p => DateTime.UtcNow)); // Update timestamp

Console.WriteLine($"Updated {affectedRows} products.");
```

**Explanation:**
*   `.Where(p => p.IsActive)`: This is your filtering condition, just like any LINQ query.
*   `.ExecuteUpdateAsync(...)`: This is the method that triggers the server-side update.
*   `setters => setters.SetProperty(...)`: Inside the `ExecuteUpdateAsync` method, you use the `SetProperty` method to specify which properties to update and how.
    *   The first lambda (`p => p.Price`) identifies the property to update.
    *   The second lambda (`p => p.Price * 1.10m`) defines the new value for that property. This can be a constant, or it can be an expression based on the entity's *current* values in the database.

**Example 2: Deactivate out-of-stock products and set their price to zero**

```csharp
using var context = new AppDbContext();

var affectedRows = await context.Products
    .Where(p => p.StockQuantity == 0 && p.IsActive)
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(p => p.IsActive, false)
        .SetProperty(p => p.Price, 0m)
        .SetProperty(p => p.LastUpdated, DateTime.UtcNow));

Console.WriteLine($"Deactivated and zeroed price for {affectedRows} products.");
```

#### Senior-Level Considerations for `ExecuteUpdate`:
[[Tracking vs. No-Tracking]]
*   **No Change Tracking**: This is the most crucial point. Entities are *not* loaded into memory, so EF Core's change tracker is completely bypassed. This means:
    *   No `DbContext.Entry(entity).State` changes.
    *   No `SaveChanges()` needed (or involved).
    *   No `SavingChanges` or `SavedChanges` events are fired.
    *   Auditing properties (like `CreatedBy`, `ModifiedBy`, `ModifiedDate`) that rely on change tracking will *not* be automatically updated unless you explicitly set them in the `SetProperty` calls.
*   **Performance**: Excellent for bulk updates. It translates to a single `UPDATE` statement, minimizing database round trips and memory consumption.
*   **Concurrency**: By default, `ExecuteUpdate` does not handle optimistic concurrency tokens (like `RowVersion` or `Timestamp` columns) unless you explicitly include them in your `Where` clause or `SetProperty` calls. If you need robust concurrency control, traditional updates might be safer, or you'll need to implement custom logic.
*   **Related Data**: `ExecuteUpdate` only operates on the entity type you query. It does not automatically update related entities or trigger cascade updates defined in your model. You would need separate `ExecuteUpdate` calls for each related entity type.
*   **Limitations**: Complex updates involving multiple joins or subqueries might be difficult or impossible to express directly with `ExecuteUpdate`. For such cases, raw SQL or traditional EF Core might be necessary.2
*   **Return Value**: Returns the number of rows affected.

### 2. `ExecuteDelete`

`ExecuteDelete` allows you to delete multiple entities from the database based on a query, without loading them into memory.

#### How it Works

Similar to `ExecuteUpdate`, you define a query that selects the entities to be deleted. EF Core then translates this into a single `DELETE` statement.

#### Syntax and Usage (Latest .NET/EF Core)

**Example 1: Delete all inactive products**

```csharp
using var context = new AppDbContext();

// Delete all products that are currently inactive
var affectedRows = await context.Products
    .Where(p => !p.IsActive)
    .ExecuteDeleteAsync();

Console.WriteLine($"Deleted {affectedRows} inactive products.");
```

**Explanation:**
*   `.Where(p => !p.IsActive)`: Your filtering condition for deletion.
*   `.ExecuteDeleteAsync()`: The method that triggers the server-side delete.

**Example 2: Delete products older than a certain date and with zero stock**

```csharp
using var context = new AppDbContext();

var cutoffDate = DateTime.UtcNow.AddYears(-2); // Products older than 2 years

var affectedRows = await context.Products
    .Where(p => p.LastUpdated < cutoffDate && p.StockQuantity == 0)
    .ExecuteDeleteAsync();

Console.WriteLine($"Deleted {affectedRows} old, out-of-stock products.");
```

#### Senior-Level Considerations for `ExecuteDelete`:

*   **No Change Tracking**: Just like `ExecuteUpdate`, `ExecuteDelete` bypasses the change tracker.
    *   No `DbContext.Entry(entity).State` changes.
    *   No `SaveChanges()` needed.
    *   No `SavingChanges` or `SavedChanges` events.
*   **Performance**: Extremely efficient for bulk deletions. A single `DELETE` statement is executed.
#Important_Note 
*   **Cascade Deletes**: This is a critical point. `ExecuteDelete` does *not* trigger EF Core's in-memory cascade delete behavior. If you have relationships configured with `DeleteBehavior.Cascade` in your EF Core model, these will only be honored if the *database itself* has cascade delete rules defined for the foreign keys. If not, you might end up with orphaned child records or a foreign key constraint violation.
    *   **Recommendation**: For complex graphs with cascade deletes, either ensure your database handles cascades, or use traditional EF Core deletion (load and remove) if you rely on EF Core's in-memory cascade logic.
*   **Concurrency**: Similar to `ExecuteUpdate`, `ExecuteDelete` doesn't inherently handle optimistic concurrency tokens.
*   **Return Value**: Returns the number of rows affected.
*   **Caution**: Use with extreme care! There's no undo. Always double-check your `Where` clause.

### General Senior-Level Best Practices and Considerations

1.  **Transactions**: When combining `ExecuteUpdate`/`ExecuteDelete` with other EF Core operations (e.g., traditional `SaveChanges()`), always wrap them in a database transaction to ensure atomicity.

    ```csharp
    using var context = new AppDbContext();
    await using var transaction = await context.Database.BeginTransactionAsync();

    try
    {
        // Perform a traditional update first
        var productToUpdate = await context.Products.FirstOrDefaultAsync(p => p.Id == 1);
        if (productToUpdate != null)
        {
            productToUpdate.Name = "Updated Product Name";
            await context.SaveChangesAsync(); // This uses change tracking
        }

        // Then perform a bulk update
        var affectedBulkRows = await context.Products
            .Where(p => p.StockQuantity < 10)
            .ExecuteUpdateAsync(setters => setters
                .SetProperty(p => p.StockQuantity, p => p.StockQuantity + 5));

        // Then perform a bulk delete
        var affectedDeleteRows = await context.Products
            .Where(p => p.IsActive == false && p.LastUpdated < DateTime.UtcNow.AddMonths(-6))
            .ExecuteDeleteAsync();

        await transaction.CommitAsync();
        Console.WriteLine("Transaction committed successfully.");
    }
    catch (Exception ex)
    {
        await transaction.RollbackAsync();
        Console.WriteLine($"Transaction rolled back: {ex.Message}");
    }
    ```

2.  **Auditing**: Since these methods bypass change tracking, any automatic auditing mechanisms (e.g., interceptors that populate `CreatedDate`, `ModifiedDate`) will not be triggered. If auditing is required for these operations, you must implement it manually (e.g., by inserting audit records in the same transaction).

3.  **Caching**: If you have an application-level cache, remember that `ExecuteUpdate` and `ExecuteDelete` will modify the database directly, potentially invalidating cached data. You'll need a strategy to refresh or invalidate your cache after these operations.

4.  **Read-Your-Own-Writes Consistency**: If you perform an `ExecuteUpdate` or `ExecuteDelete` and then immediately query the affected data using traditional EF Core, you might encounter stale data if your `DbContext` instance is still tracking old versions of those entities. It's often best to use a fresh `DbContext` instance or clear the change tracker (`context.ChangeTracker.Clear()`) if you need to immediately re-query the affected data after a bulk operation.

5.  **Error Handling**: Database errors (e.g., constraint violations) will be thrown as `DbUpdateException` or similar, just like with `SaveChanges()`.

6.  **When to Prefer Traditional EF Core**:
    *   When you need to load entities into memory for complex business logic before deciding to update/delete.
    *   When you rely heavily on EF Core's change tracking for auditing, concurrency, or automatic property population.
    *   When you need EF Core's in-memory cascade delete behavior for complex object graphs.
    *   When dealing with a small number of entities where the performance gain of bulk operations is negligible, and the clarity of traditional operations is preferred.

By mastering `ExecuteUpdate` and `ExecuteDelete`, you gain powerful tools for optimizing database interactions in your .NET applications, a hallmark of a senior software engineer. Always consider the trade-offs and choose the right tool for the job!