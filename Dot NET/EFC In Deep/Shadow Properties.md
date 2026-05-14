### Shadow Properties in Entity Framework Core

Shadow properties are properties that are part of the Entity Framework Core model and exist in the database, but they are *not* defined in your .NET entity class. EF Core tracks their state and can persist them to the database, even though they don't have a corresponding C# property.

They are particularly useful for metadata, auditing, or framework-level concerns that you don't want to expose directly on your domain entities, keeping your domain model cleaner and focused on business logic.

**Why use them?**

*   **Clean Domain Models:** Keep your C# entity classes focused purely on domain logic, free from persistence-specific concerns like `CreatedDate`, `LastModifiedDate`, `IsDeleted`, or `TenantId`.
*   **Auditing:** Automatically track creation/modification timestamps or the user who performed an action without cluttering every entity.
*   **Soft Deletion:** Implement soft deletion (`IsDeleted` flag) without adding `IsDeleted` to every entity class.
*   **Multi-tenancy:** Store a `TenantId` on entities without exposing it in the domain model, enforcing tenant isolation at the data access layer.
*   **Concurrency Tokens:** While EF Core has `[ConcurrencyCheck]` and `IsRowVersion()`, a shadow property could be used for a custom concurrency token.

---

#### Defining Shadow Properties (Configuration)

You define shadow properties in your `DbContext`'s `OnModelCreating` method.

**Example Scenario:** Let's say we want to add `CreatedDate` and `LastUpdated` timestamps to our `Order` entity, and also implement soft deletion using an `IsDeleted` flag.
#Important_Note 
```csharp
// Entity Class (no CreatedDate, LastUpdated, IsDeleted properties)
public class Order
{
    public int Id { get; set; }
    public string OrderNumber { get; set; }
    public decimal TotalAmount { get; set; }
    public ICollection<OrderItem> Items { get; set; } = new List<OrderItem>();
    public Customer Customer { get; set; }
    public int CustomerId { get; set; }
}

// DbContext Configuration
public class AppDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    public DbSet<Customer> Customers { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Define Shadow Properties for the Order entity
        modelBuilder.Entity<Order>()
            .Property<DateTime>("CreatedDate") // Shadow property named "CreatedDate" of type DateTime
            .HasDefaultValueSql("GETDATE()"); // Set a default value in the database

        modelBuilder.Entity<Order>()
            .Property<DateTime>("LastUpdated") // Shadow property named "LastUpdated" of type DateTime
            .HasDefaultValueSql("GETDATE()"); // Set a default value in the database

        modelBuilder.Entity<Order>()
            .Property<bool>("IsDeleted") // Shadow property named "IsDeleted" of type bool
            .HasDefaultValue(false); // Default to false

        // Configure a global query filter for soft deletion
        modelBuilder.Entity<Order>()
            .HasQueryFilter(o => EF.Property<bool>(o, "IsDeleted") == false);
    }

    public override int SaveChanges()
    {
        UpdateShadowProperties();
        return base.SaveChanges();
    }

    public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        UpdateShadowProperties();
        return base.SaveChangesAsync(cancellationToken);
    }

    private void UpdateShadowProperties()
    {
        var entries = ChangeTracker.Entries()
            .Where(e => e.State == EntityState.Added || e.State == EntityState.Modified);

        foreach (var entry in entries)
        {
            if (entry.Metadata.FindProperty("CreatedDate") != null && entry.State == EntityState.Added)
            {
                entry.Property("CreatedDate").CurrentValue = DateTime.UtcNow;
            }

            if (entry.Metadata.FindProperty("LastUpdated") != null && entry.State == EntityState.Modified)
            {
                entry.Property("LastUpdated").CurrentValue = DateTime.UtcNow;
            }
        }
    }
}
```

**Explanation:**
*   `modelBuilder.Entity<Order>().Property<DateTime>("CreatedDate")`: This tells EF Core that the `Order` entity has a property named "CreatedDate" of type `DateTime`, even though it's not in the `Order` C# class.
*   `.HasDefaultValueSql("GETDATE()")`: This configures the database column to have a default value, which is useful for `CreatedDate`.
*   `HasQueryFilter`: This is a senior-level technique. By adding a global query filter, *every* query for `Order` entities will automatically append `WHERE IsDeleted = 0` (or `false`), ensuring that soft-deleted entities are not returned by default. This is crucial for consistent soft-delete behavior.

---

#### Using Shadow Properties (Reading & Writing)

Since shadow properties are not part of your C# class, you interact with them via the `EntityEntry` API.

**1. Setting Values (Writing):**

You typically set shadow property values in `SaveChanges()` or `SaveChangesAsync()` overrides in your `DbContext` to ensure they are consistently applied.

```csharp
// (See UpdateShadowProperties method in the DbContext example above)

// When you add a new order:
var newOrder = new Order
{
    OrderNumber = "ORD-001",
    TotalAmount = 100.00m,
    CustomerId = 1 // Assuming customer exists
};
_context.Orders.Add(newOrder);
await _context.SaveChangesAsync();
// CreatedDate will be set by the DB default or by UpdateShadowProperties
// LastUpdated will be set by UpdateShadowProperties
// IsDeleted will be false by default
```

**2. Getting Values (Reading):**

You can retrieve the value of a shadow property from a tracked entity using `Entry().Property().CurrentValue`.

```csharp
var order = await _context.Orders.FirstOrDefaultAsync(o => o.Id == 1);

if (order != null)
{
    // Accessing the shadow property value
    var createdDate = _context.Entry(order).Property<DateTime>("CreatedDate").CurrentValue;
    var lastUpdated = _context.Entry(order).Property<DateTime>("LastUpdated").CurrentValue;
    var isDeleted = _context.Entry(order).Property<bool>("IsDeleted").CurrentValue;

    Console.WriteLine($"Order ID: {order.Id}");
    Console.WriteLine($"Created Date: {createdDate}");
    Console.WriteLine($"Last Updated: {lastUpdated}");
    Console.WriteLine($"Is Deleted: {isDeleted}");
}
```

**3. Querying with Shadow Properties:**

You can use shadow properties in LINQ queries using `EF.Property<TEntity>(entity, propertyName)`. This is essential for filtering, ordering, or projecting based on these properties.

```csharp
// Find all orders updated in the last 24 hours
var recentOrders = await _context.Orders
    .Where(o => EF.Property<DateTime>(o, "LastUpdated") > DateTime.UtcNow.AddDays(-1))
    .ToListAsync();

// Find all soft-deleted orders (bypassing the global query filter)
// Use IgnoreQueryFilters() to see entities that would normally be filtered out
var deletedOrders = await _context.Orders
    .IgnoreQueryFilters() // Temporarily disable global query filters
    .Where(o => EF.Property<bool>(o, "IsDeleted") == true)
    .ToListAsync();

// Order by CreatedDate
var oldestOrders = await _context.Orders
    .OrderBy(o => EF.Property<DateTime>(o, "CreatedDate"))
    .ToListAsync();

// Projecting a shadow property into a DTO
var orderAuditDto = await _context.Orders
    .Where(o => o.Id == 1)
    .Select(o => new OrderAuditDto
    {
        OrderId = o.Id,
        OrderNumber = o.OrderNumber,
        CreatedDate = EF.Property<DateTime>(o, "CreatedDate"),
        LastUpdated = EF.Property<DateTime>(o, "LastUpdated"),
        IsDeleted = EF.Property<bool>(o, "IsDeleted")
    })
    .FirstOrDefaultAsync();
```

---

#### Senior Considerations:

*   **Global Query Filters for Soft Delete:** As demonstrated, `HasQueryFilter` is the canonical way to implement soft deletion with shadow properties. It ensures that `IsDeleted = true` entities are excluded from *almost* all queries by default. Remember to use `IgnoreQueryFilters()` when you *do* need to access soft-deleted entities (e.g., for an admin recovery feature).
*   **`SaveChanges` Override for Auditing:** Overriding `SaveChanges()` and `SaveChangesAsync()` in your `DbContext` is the standard pattern for automatically managing shadow properties like `CreatedDate` and `LastUpdated`. This centralizes the logic and ensures consistency.
*   **Discoverability:** The main drawback of shadow properties is their lack of discoverability. They don't appear in IntelliSense on your entity objects. Developers new to the codebase might not realize these properties exist or how to interact with them. Good documentation and consistent patterns are key.
*   **When to use a regular property vs. shadow property:**
    *   **Regular Property:** If a property is genuinely part of the domain model's public contract, is frequently accessed by business logic, or needs to be directly manipulated by application code, make it a regular C# property.
    *   **Shadow Property:** If a property is purely for persistence concerns, auditing, or framework-level behavior, and you want to keep your domain model clean, a shadow property is a strong candidate.
*   **Performance:** There's no significant performance difference between shadow properties and regular properties once mapped to the database. The overhead is primarily in the `DbContext`'s `ChangeTracker` and `OnModelCreating` configuration.

By strategically using shadow properties, you can build more robust, maintainable, and domain-centric applications with EF Core, separating concerns effectively.
