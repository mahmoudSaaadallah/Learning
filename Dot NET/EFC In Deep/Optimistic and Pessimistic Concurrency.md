## Optimistic and Pessimistic Concurrency

When multiple users or processes try to modify the same data simultaneously, concurrency issues can arise. To prevent data corruption and ensure data integrity, we employ concurrency control mechanisms. The two primary strategies are Optimistic Concurrency and Pessimistic Concurrency.

### 1. Optimistic Concurrency

**Concept:**
Optimistic concurrency assumes that conflicts are rare. It allows multiple users to read and modify data concurrently without explicit locks. Conflicts are detected only when a user attempts to save changes. If a conflict is detected (meaning another user modified the data since it was last read), the application typically informs the user, and they must decide how to resolve it (e.g., refresh their data and reapply changes, or overwrite the other user's changes).

**How it Works:**
The core idea is to verify that the data being updated o*r deleted has not changed since it was originally retrieved. This is typically achieved using a **version column** (also known as a timestamp, row version, or concurrency token) in the database table.

*   **Read:** When an entity is read, its current version value is also read and stored.
*   **Update/Delete:** When the entity is saved, the `UPDATE` or `DELETE` statement includes a `WHERE` clause that checks if the version column in the database still matches the version that was originally read.
    *   If they match, the update/delete proceeds, and the version column is typically incremented or updated (e.g., a new timestamp is generated).
    *   If they *don't* match, it means another user modified the data in between, and a concurrency conflict is detected. The database operation will affect 0 rows, and EF Core will throw a `DbUpdateConcurrencyException`.

**Implementation in EF Core (.NET 6+):**

You can configure a property as a concurrency token using:

1.  **`[Timestamp]` Attribute:** For `byte[]` properties. EF Core automatically configures this as a row version.

```csharp
public class Product
{
	public int Id { get; set; }
	public string Name { get; set; }
	public decimal Price { get; set; }

	[Timestamp] // This property will be used for optimistic concurrency
	public byte[] RowVersion { get; set; }
}
```

2.  **`IsRowVersion()` Fluent API:** For `byte[]` properties in `OnModelCreating`.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	modelBuilder.Entity<Product>()
		.Property(p => p.RowVersion)
		.IsRowVersion(); // Configures as a row version
}
```

3.  **`IsConcurrencyToken()` Fluent API:** For any property type (e.g., `int`, `Guid`, `DateTime`). You'd typically manage the incrementing/updating of this value yourself, or let the database handle it (e.g., a `DateTime` column with a `DEFAULT GETDATE()` and `ON UPDATE GETDATE()` trigger).

```csharp
public class Product
{
	public int Id { get; set; }
	public string Name { get; set; }
	public decimal Price { get; set; }
	public int Version { get; set; } // Custom version number
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	modelBuilder.Entity<Product>()
		.Property(p => p.Version)
		.IsConcurrencyToken(); // Configures as a concurrency token
}
```
When using a custom concurrency token like `Version`, you would typically increment it in your application logic before calling `SaveChanges()`:
```csharp
var product = await _context.Products.FindAsync(productId);
if (product != null)
{
	product.Name = "New Name";
	product.Version++; // Manually increment the version
	try
	{
		await _context.SaveChangesAsync();
	}
	catch (DbUpdateConcurrencyException ex)
	{
		// Handle conflict: e.g., reload entity, merge changes, or inform user
	}
}
```

**Pros:**
*   **High Concurrency:** No locks are held, allowing many users to access and modify data simultaneously.
*   **Scalability:** Better for highly concurrent systems as it avoids the overhead of managing locks.
*   **Simpler Read Operations:** Reads are never blocked by locks.

**Cons:**
*   **Conflict Resolution:** Requires application-level logic to handle conflicts (e.g., refresh data, merge changes, retry).
*   **Rollbacks/Retries:** If a conflict occurs, the transaction might need to be rolled back, and the operation retried.
*   **Lost Updates:** If not implemented correctly, it can lead to lost updates if conflicts are not detected or handled.

### 2. Pessimistic Concurrency

**Concept:**
Pessimistic concurrency assumes that conflicts are frequent. It prevents conflicts by locking the data resource (e.g., a row, a table) as soon as it's accessed for modification. This lock prevents other users from modifying the data until the lock is released.

**How it Works:**
Database-level locks are used.

*   **Read for Update:** When a user intends to modify data, they acquire a lock on that data. This might be an exclusive lock on a row or a table.
*   **Modification:** While the lock is held, no other user can modify the locked data. Other users attempting to access the locked data will either wait for the lock to be released or receive an error, depending on the database and **transaction isolation level**.
*   **Release Lock:** The lock is released when the transaction is committed or rolled back.

**Implementation in EF Core (.NET 6+):**
EF Core itself does not directly provide built-in mechanisms for pessimistic locking. Pessimistic concurrency is typically implemented at the database level using specific SQL commands or transaction isolation levels.

You would usually achieve this by:
*   **Database-Specific Locking Hints:** Using raw SQL queries with locking hints (e.g., `SELECT ... WITH (UPDLOCK)` in SQL Server, `SELECT ... FOR UPDATE` in PostgreSQL/MySQL).
*   **Transaction Isolation Levels:** Setting a higher transaction isolation level (e.g., `Serializable`) can implicitly apply locks, but this often comes with significant performance overhead and can lead to deadlocks.

**Example (using raw SQL for illustration, not direct EF Core API):**

```csharp
// This is conceptual, demonstrating the SQL. EF Core doesn't have a direct API for this.
// You'd use FromSqlRaw or similar.
await using var transaction = await context.Database.BeginTransactionAsync(IsolationLevel.Serializable);

try
{
    // Acquire an exclusive lock on the product row
    var product = await context.Products
        .FromSqlRaw("SELECT * FROM Products WHERE Id = {0} WITH (UPDLOCK)", productId) // SQL Server specific
        .FirstOrDefaultAsync();

    if (product != null)
    {
        // Perform modifications
        product.Price += 5m;
        await context.SaveChangesAsync(); // This will commit the changes and release the lock

        await transaction.CommitAsync();
    }
}
catch (Exception ex)
{
    await transaction.RollbackAsync();
    // Handle exceptions, including potential deadlocks
}
```

**Pros:**
*   **Prevents Conflicts:** Guarantees that no conflicts will occur during the modification period.
*   **Simpler Application Logic:** Once a lock is acquired, the application doesn't need to worry about conflict resolution.

**Cons:**
*   **Reduced Concurrency:** Locks reduce the number of users who can access the data simultaneously.
*   **Performance Overhead:** Managing locks adds overhead to the database.
*   **Deadlocks:** Can lead to deadlocks if transactions acquire locks in different orders.
*   **Scalability Issues:** Can become a bottleneck in high-traffic applications.
*   **Long-Running Transactions:** Holding locks for extended periods can severely impact performance.

### Comparison Table

| Feature             | Optimistic Concurrency                               | Pessimistic Concurrency                               |
| :------------------ | :--------------------------------------------------- | :---------------------------------------------------- |
| **Assumption**      | Conflicts are rare.                                  | Conflicts are frequent.                               |
| **Mechanism**       | Version column (timestamp, row version, custom int). | Database locks (row-level, table-level).              |
| **Conflict Mgmt.**  | Detects conflicts at save time; requires app logic.  | Prevents conflicts by locking; no app conflict logic. |
| **Concurrency**     | High.                                                | Low.                                                  |
| **Performance**     | Generally better for high-traffic reads.             | Can be slower due to lock overhead.                   |
| **Scalability**     | More scalable.                                       | Less scalable.                                        |
| **EF Core Support** | Built-in via `[Timestamp]` or `IsRowVersion()`.      | Not directly built-in; relies on raw SQL/DB features. |
| **Use Cases**       | Most web applications, collaborative editing.        | Critical financial transactions, inventory control.   |

### Senior-Level Considerations with `ExecuteUpdate` and `ExecuteDelete`
[[Update and Delete on the Server Side]]
This is where your understanding of concurrency tokens becomes crucial when using the bulk operations we discussed earlier.

**`ExecuteUpdate` and `ExecuteDelete` DO NOT automatically respect or update optimistic concurrency tokens.**

When you use `ExecuteUpdate` or `ExecuteDelete`, EF Core generates a single `UPDATE` or `DELETE` statement directly. It does not load entities into memory, and therefore, it does not read the `RowVersion` or `ConcurrencyToken` property, nor does it automatically include it in the `WHERE` clause or update it in the `SET` clause.

**If you need optimistic concurrency with `ExecuteUpdate` or `ExecuteDelete`, you must explicitly include the concurrency token in your `WHERE` clause and/or `SET` clause.**

**Example: `ExecuteUpdate` with Optimistic Concurrency (Manual)**

Let's say you want to update a product's price, but only if its `RowVersion` matches a specific value you previously read.

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    [Timestamp]
    public byte[] RowVersion { get; set; } // Concurrency token
}

// ... later in your code
using var context = new AppDbContext();

// Simulate reading an entity and its RowVersion
var productToModify = await context.Products.AsNoTracking().FirstOrDefaultAsync(p => p.Id == 1);
if (productToModify == null)
{
    Console.WriteLine("Product not found.");
    return;
}

var originalRowVersion = productToModify.RowVersion;
var newPrice = productToModify.Price * 1.05m; // Increase by 5%

// Attempt to update, but only if the RowVersion matches the original
var affectedRows = await context.Products
    .Where(p => p.Id == productToModify.Id && p.RowVersion == originalRowVersion) // Crucial: Check RowVersion
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(p => p.Price, newPrice)
        // Note: For byte[] RowVersion, the database typically updates it automatically on UPDATE.
        // For custom int/Guid concurrency tokens, you might need to explicitly increment/update it here:
        // .SetProperty(p => p.Version, p => p.Version + 1)
    );

if (affectedRows == 0)
{
    Console.WriteLine("Optimistic concurrency conflict detected! Product was modified by another user.");
    // Handle the conflict: e.g., reload data, inform user, retry.
}
else
{
    Console.WriteLine($"Product {productToModify.Id} updated successfully.");
}
```

**Key Takeaway for Senior Developers:**
*   Understand the trade-offs between optimistic and pessimistic concurrency. Optimistic is generally preferred for most modern web applications due to its scalability.
*   Be aware that `ExecuteUpdate` and `ExecuteDelete` bypass EF Core's change tracking and, consequently, its automatic optimistic concurrency handling.
*   If you need optimistic concurrency with bulk operations, you *must* manually incorporate the concurrency token into your `Where` clause to ensure data integrity.
*   For complex scenarios requiring pessimistic locking, you'll likely need to drop down to raw SQL or leverage database-specific features.

