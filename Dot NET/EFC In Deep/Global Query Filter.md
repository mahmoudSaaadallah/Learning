### Global Query Filters in Entity Framework Core

A **Global Query Filter** allows you to define a LINQ query predicate (a `WHERE` clause) that is automatically applied to *all* queries for a specific entity type in your `DbContext`. This means you don't have to remember to add the same `Where` clause repeatedly throughout your application code; EF Core handles it for you.

This feature is  configured in your `DbContext`'s `OnModelCreating` method.

#### How it Works

When you define a global query filter for an entity, EF Core intercepts every query targeting that entity type and injects the specified `WHERE` clause into the generated SQL. This happens transparently, ensuring that the filter is always applied unless explicitly bypassed.

**Key Characteristics:**
*   **Automatic Application:** Applied to all queries from `DbSet<T>` and navigation properties.
*   **Centralized Logic:** Defines filtering logic in one place (`OnModelCreating`).
*   **Composition:** Filters are combined with any `Where` clauses you explicitly add to your LINQ queries using `AND`.

#### Benefits

1.  **Consistency:** Ensures that critical filtering logic (like soft deletion or multi-tenancy) is *always* applied, preventing accidental data exposure or manipulation.
2.  **Reduced Boilerplate:** Eliminates the need to repeat the same `Where` clause across numerous queries, making your application code cleaner and less error-prone.
3.  **Improved Maintainability:** If the filtering logic needs to change, you only update it in one place (`OnModelCreating`), and the change propagates throughout your application.
4.  **Security:** Helps enforce data access rules at the database level, reducing the risk of unauthorized data access.

#### Common Use Cases

**1. Soft Deletion**

This is perhaps the most common and impactful use case. Instead of physically deleting records, you mark them as "deleted" (e.g., with an `IsDeleted` flag). A global query filter then ensures that these soft-deleted records are never returned by default.

**Example:**

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public bool IsDeleted { get; set; } // Can be a regular property or a shadow property
}

public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configure global query filter for soft deletion
        modelBuilder.Entity<Product>()
            .HasQueryFilter(p => !p.IsDeleted); // Only return products where IsDeleted is false
    }

    // Example of soft deleting a product
    public void SoftDeleteProduct(int productId)
    {
        var product = Products.Find(productId); // Find bypasses query filters
        if (product != null)
        {
            product.IsDeleted = true;
            SaveChanges();
        }
    }
}

// Usage:
using (var context = new AppDbContext())
{
    // This query will automatically include 'WHERE IsDeleted = 0'
    var activeProducts = await context.Products.ToListAsync();

    // Even if you add another WHERE clause, the IsDeleted filter is still applied:
    var cheapActiveProducts = await context.Products
        .Where(p => p.Price < 50)
        .ToListAsync();
    // Generated SQL will be something like: SELECT ... FROM Products WHERE IsDeleted = 0 AND Price < 50
}
```

**2. Multi-Tenancy**

In multi-tenant applications, each tenant typically only sees their own data. A global query filter can automatically filter entities by a `TenantId` property.

**Example:**

```csharp
public class Order
{
    public int Id { get; set; }
    public string OrderNumber { get; set; }
    public int TenantId { get; set; } // Identifies which tenant owns this order
    // ... other properties
}

public class AppDbContext : DbContext6
{
    private readonly int _currentTenantId; // Injected or set based on current user

    public AppDbContext(DbContextOptions<AppDbContext> options, int currentTenantId) : base(options)
    {
        _currentTenantId = currentTenantId;
    }

    public DbSet<Order> Orders { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configure global query filter for multi-tenancy
        modelBuilder.Entity<Order>()
            .HasQueryFilter(o => o.TenantId == _currentTenantId);
    }
}

// Usage:
// When creating DbContext, pass the current tenant's ID
var options = new DbContextOptionsBuilder<AppDbContext>().UseSqlServer("...").Options;
using (var context = new AppDbContext(options, 123)) // Tenant ID 123
{
    // This query will automatically include 'WHERE TenantId = 123'
    var tenantOrders = await context.Orders.ToListAsync();
}
```
**Note:** For multi-tenancy, the `_currentTenantId` needs to be dynamically provided to the `DbContext`. This is typically done via dependency injection, where the `TenantId` is resolved from the current user's context (e.g., from a JWT claim or session).

#### Bypassing Global Query Filters: `IgnoreQueryFilters()`

Sometimes, you need to temporarily disable the global query filter for a specific query. For instance, an administrator might need to view all soft-deleted items, or a super-admin might need to see data across all tenants.

You can achieve this using the `.IgnoreQueryFilters()` extension method on your `IQueryable`.

**Example:**

```csharp
using (var context = new AppDbContext())
{
    // Get all products, including soft-deleted ones
    var allProducts = await context.Products
        .IgnoreQueryFilters() // This bypasses the !p.IsDeleted filter
        .ToListAsync();

    // Get only soft-deleted products
    var deletedProducts = await context.Products
        .IgnoreQueryFilters() // Bypass the default filter
        .Where(p => p.IsDeleted) // Apply a specific filter for deleted items
        .ToListAsync();
}
```

**Important:** Use `IgnoreQueryFilters()` judiciously. Bypassing filters can expose sensitive data if not handled carefully, especially in multi-tenant scenarios. Ensure proper authorization checks are in place when allowing users to bypass filters.

#### Senior Considerations

1.  **Performance Impact:**
    *   **Simple Filters:** For simple filters (like `!IsDeleted` or `TenantId == X`), the performance impact is usually negligible. The database can optimize these `WHERE` clauses efficiently.
    *   **Complex Filters:** If your filter involves complex logic, subqueries, or functions that prevent index usage, it *can* impact query performance. Always profile your queries.
    *   **Index Usage:** Ensure that the columns used in your global query filters (e.g., `IsDeleted`, `TenantId`) are indexed in your database. This is crucial for performance.

2.  **Debugging and Discoverability:**
    *   **Hidden Filters:** Global query filters can make debugging tricky because the `WHERE` clause isn't explicitly visible in your LINQ query. If you're getting unexpected results (e.g., missing data), remember to check `OnModelCreating` for filters.
    *   **Logging:** Enable EF Core logging to see the actual SQL generated, which will include the applied filters. This is invaluable for debugging.

3.  **Composition with Other `Where` Clauses:**
    *   Global filters are always applied first and combined with `AND` with any subsequent `Where` clauses.
    *   Example: `context.Products.Where(p => p.Price > 10)` with `HasQueryFilter(p => !p.IsDeleted)` will result in `WHERE !p.IsDeleted AND p.Price > 10`.

4.  **Inheritance Hierarchies:**
    *   Filters defined on a base entity type will also apply to all derived entity types.
    *   Filters defined on a derived entity type will only apply to that specific type.

5.  **Security Implications:**
    *   While global query filters enhance security by enforcing data access rules, they are not a substitute for robust authorization. Always ensure that users are authorized to perform actions on the data they are trying to access, even if the filter prevents them from seeing it.
    *   Be extremely careful with `IgnoreQueryFilters()` in production code, especially in public-facing APIs.

6.  **Testing:**
    *   When writing unit or integration tests, be aware of global query filters. You might need to use `IgnoreQueryFilters()` in some tests to verify the state of soft-deleted or tenant-specific data.
    *   Consider having separate `DbContext` configurations for testing if your filters depend on runtime values (like `_currentTenantId`).

7.  **Limitations:**
    *   **`Find()` Method:** The `Find()` method (which looks up an entity by its primary key) *does not* apply global query filters. It directly queries the database by primary key. This is important for soft deletion: `_context.Products.Find(deletedProductId)` *will* return a soft-deleted product if it exists in the database and is tracked.
    *   **`Load()` Method (Explicit Loading):** Similarly, explicit loading using `Entry().Reference().Load()` or `Entry().Collection().Load()` also *does not* apply global query filters. If you explicitly load a collection, it will load all related entities, regardless of filters.
    *   **`Attach()`/`Update()`:** Filters are for *queries*. When you attach or update an entity, the filter is not involved.

By understanding these nuances, you can leverage global query filters effectively to build more robust and secure EF Core applications.

Do you have any specific scenarios in mind where you're considering using `HasQueryFilter`, or would you like to explore how it interacts with other EF Core features?