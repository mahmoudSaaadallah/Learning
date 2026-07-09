### Topic: Owned Entity Types in Entity Framework Core

#### 1. What are Owned Entity Types?

In Entity Framework Core, an **Owned Entity Type** allows you to model a concept that is conceptually part of another entity (the "owner") and does not have its own independent identity or lifecycle. Think of them as **value objects** or complex types that are intrinsically tied to their owner.

Key characteristics:
*   They are defined as part of an owner entity.
*   By default, they are mapped to the **same database table** as their owner. This is often referred to as "table splitting" or "complex types" in other ORMs.
*   They do **not** have their own primary key defined in the model (EF Core will shadow-property one if needed for internal tracking, but you don't define it).
*   They do **not** have their own `DbSet<T>` in the `DbContext`. You access them only through their owner.
*   Their lifecycle is entirely dependent on their owner. If the owner is deleted, the owned entity is deleted.

#### 2. The Problem Owned Entity Types Solve

Consider a `Customer` entity. A customer might have a `ShippingAddress` and a `BillingAddress`. Both addresses have properties like `Street`, `City`, `State`, `ZipCode`.

Without owned types, you might:
1.  **Create a separate `Address` entity:** This would require `Address` to have its own primary key, and you'd have two one-to-one relationships from `Customer` to `Address` (e.g., `ShippingAddressId`, `BillingAddressId`). This creates separate tables and requires joins, even though an `Address` might not make sense existing independently of a `Customer`.
2.  **Embed all address properties directly in `Customer`:** `ShippingStreet`, `ShippingCity`, `BillingStreet`, `BillingCity`, etc. This leads to code duplication and a less organized model.

Owned entity types provide a clean solution by allowing you to encapsulate the `Address` properties into a separate class while still mapping them to the `Customer` table.

#### 3. How to Mark a Class as an Owned Entity Type

You can mark a class as an owned entity type using either:

1.  **The `[Owned]` attribute:** Placed directly on the class definition.
2.  **Fluent API configuration:** Using `OwnsOne()` or `OwnsMany()` in `OnModelCreating`. This is generally preferred for more complex scenarios or when you want to keep your domain models clean of EF Core specific attributes.

Let's use an example:

**Scenario:** A `Order` entity that has a `ShippingAddress` and a `BillingAddress`. An `Address` is a value object that doesn't exist independently.

**1. Define the Owned Type:**

```csharp
// Models/Address.cs
// This class represents a value object.
// It doesn't need an Id property because it's owned by another entity.
// We can mark it with [Owned] attribute.
using Microsoft.EntityFrameworkCore; // Required for [Owned]

[Owned] // This attribute marks Address as an owned entity type
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string ZipCode { get; set; }
    public string Country { get; set; }
}
```

**2. Define the Owner Entity:**

```csharp
// Models/Order.cs
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal TotalAmount { get; set; }

    // ShippingAddress is an instance of the owned type Address
    public Address ShippingAddress { get; set; }

    // BillingAddress is another instance of the owned type Address
    public Address BillingAddress { get; set; }

    // Other properties...
}
```

**3. DbContext Configuration (Fluent API - Recommended):**

Even if you use the `[Owned]` attribute, it's good practice to explicitly configure owned types in `OnModelCreating` for clarity, especially when dealing with multiple instances of the same owned type on an owner (like `ShippingAddress` and `BillingAddress`). This allows you to specify column names to avoid conflicts.

```csharp
// Data/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }

    public DbSet<Order> Orders { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>(entity =>
        {
            // Configure ShippingAddress as an owned type
            entity.OwnsOne(o => o.ShippingAddress, address =>
            {
                // Map properties of ShippingAddress to columns in the Orders table
                address.Property(a => a.Street).HasColumnName("ShippingStreet");
                address.Property(a => a.City).HasColumnName("ShippingCity");
                address.Property(a => a.State).HasColumnName("ShippingState");
                address.Property(a => a.ZipCode).HasColumnName("ShippingZipCode");
                address.Property(a => a.Country).HasColumnName("ShippingCountry");
            });

            // Configure BillingAddress as an owned type
            entity.OwnsOne(o => o.BillingAddress, address =>
            {
                // Map properties of BillingAddress to columns in the Orders table
                address.Property(a => a.Street).HasColumnName("BillingStreet");
                address.Property(a => a.City).HasColumnName("BillingCity");
                address.Property(a => a.State).HasColumnName("BillingState");
                address.Property(a => a.ZipCode).HasColumnName("BillingZipCode");
                address.Property(a => a.Country).HasColumnName("BillingCountry");
            });
        });
    }
}
```

**Resulting Database Schema (after migration):**

The `Orders` table would look something like this:

| Id | OrderDate | TotalAmount | ShippingStreet | ShippingCity | ShippingState | ShippingZipCode | ShippingCountry | BillingStreet | BillingCity | BillingState | BillingZipCode | BillingCountry |
|----|-----------|-------------|----------------|--------------|---------------|-----------------|-----------------|---------------|-------------|--------------|----------------|----------------|
| 1  | ...       | ...         | 123 Main St    | Anytown      | CA            | 90210           | USA             | 456 Oak Ave   | Otherville  | NY           | 10001          | USA            |

Notice how all `Address` properties are flattened into the `Orders` table.

#### 4. Key Characteristics and Usage

*   **No Independent Identity:** An `Address` instance in this context cannot exist without an `Order`. If you delete an `Order`, its `ShippingAddress` and `BillingAddress` data are also deleted.
*   **No `DbSet`:** You cannot write `_context.Addresses.Add(new Address())`. You always interact with them through their owner: `_context.Orders.Add(new Order { ShippingAddress = new Address { ... } })`.
*   **Optional Owned Types:** If the owned type property is nullable (e.g., `public Address? ShippingAddress { get; set; }`), EF Core will map all its columns as nullable. If the property is `null`, all corresponding columns in the database will be `NULL`.
*   **Required Owned Types:** If the owned type property is non-nullable (e.g., `public Address ShippingAddress { get; set; }`), EF Core will map its columns as non-nullable. You must provide a value for it.
*   **Nested Owned Types:** An owned type can itself own other types. For example, `Address` could own a `GeoLocation` type.
*   **`OwnsMany()`:** If an owner has a collection of owned types (e.g., `Order` has `List<OrderItem>`), you use `OwnsMany()`. In this case, EF Core will create a **separate table** for the owned collection, but it will still be implicitly owned by the parent and won't have its own `DbSet`. The owned collection table will have a foreign key back to the owner and a shadow primary key.

#### 5. Senior Insights

1.  **When to Use Owned Types (Value Objects):**
    *   **Conceptual Grouping:** When a set of properties logically belongs together and describes a single concept (e.g., `Address`, `Money`, `Dimensions`, `AuditInfo`).
    *   **No Independent Existence:** The object doesn't make sense on its own; its lifecycle is tied to its owner.
    *   **No Shared Identity:** An `Address` owned by `Order A` is distinct from an `Address` owned by `Order B`, even if their property values are identical. They are not shared instances.
    *   **No `DbSet` Required:** You don't need to query or manage these objects independently.

2.  **When NOT to Use Owned Types (Regular Entities):**
    *   **Independent Identity:** If the object needs its own primary key and can exist independently (e.g., `Product`, `Category`).
    *   **Shared Instances:** If multiple entities can reference the *same instance* of an object (e.g., multiple `Order`s referencing the *same* `Customer`). This requires a foreign key relationship and a separate table.
    *   **Independent Queries:** If you need to query or manage these objects directly via their own `DbSet`.
    *   **Complex Relationships:** Owned types cannot have relationships to other *non-owner* entities. If your `Address` needs to reference a `Country` entity, it should probably be a regular entity with a foreign key.

3.  **Table Splitting vs. Owned Types:**
    *   **Table Splitting:** A single entity is mapped to *multiple tables*. This is less common and used when you want to logically separate parts of a single entity into different tables for performance or organizational reasons.
    *   **Owned Types:** Multiple *conceptual* entities (owner + owned) are mapped to a *single table* (by default for `OwnsOne`). This is the primary use case for value objects.
    *   **`OwnsMany` Exception:** When using `OwnsMany`, EF Core *does* create a separate table for the collection of owned types, but it's still managed as part of the owner's aggregate.

4.  **Performance:**
    *   For `OwnsOne`, mapping to the same table avoids database joins, which can be a performance benefit for read operations.
    *   For `OwnsMany`, it creates a separate table, similar to a one-to-many relationship, but without the need for an explicit `DbSet` for the owned type.

5.  **Querying Owned Types:**
    *   You query them directly through their owner.
    *   Example: `_context.Orders.Where(o => o.ShippingAddress.City == "Anytown").ToList();`
    *   EF Core handles the mapping to the flattened columns automatically.

6.  **Immutability (Domain-Driven Design Context):**
    *   In DDD, value objects are often immutable. While C# doesn't enforce this directly, you can design your `Address` class to be immutable (e.g., using `record` types in C# 9+ or private setters with constructor initialization). This aligns well with the concept of owned types.