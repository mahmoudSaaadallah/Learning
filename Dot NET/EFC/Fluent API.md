### Entity Framework Core: Fluent API
The Fluent API provides a code-first way to configure your EF Core model using a set of methods that can be chained together, offering a highly expressive and powerful syntax. All Fluent API configurations are typically performed by overriding the `OnModelCreating` method in your `DbContext` class.

**Why is Fluent API crucial for senior developers?**

*   **Power & Flexibility:** It allows for configurations that are impossible or very difficult with [[Data Annotations]] (e.g., composite keys, complex indexes, many-to-many relationships without a join entity, inheritance mapping).
*   **Separation of Concerns:** It keeps your entity classes clean, free from database-specific attributes. Your domain entities can remain pure Plain Old CLR Objects (POCOs), while mapping concerns are centralized in your `DbContext`. This is a cornerstone of clean architecture and domain-driven design.
*   **Centralized Configuration:** All your model configurations are in one place (`OnModelCreating`), making it easier to review, maintain, and understand the overall database schema.
*   **Refactoring Safety:** Using lambda expressions (`p => p.Name`) for property references is refactoring-safe, unlike string literals in some Data Annotations.

**Key Object:** The `ModelBuilder` object, passed into `OnModelCreating`, is your entry point for all Fluent API configurations.

Let's revisit our `Blog` and `Post` entities and configure them using the Fluent API.

---

#### **Project Setup (Conceptual)**

We'll use the same `Blog` and `Post` entities, but this time, we'll remove most of the Data Annotations to demonstrate how Fluent API takes over.

```csharp
// Entities/Blog.cs
// No Data Annotations here for demonstration purposes
public class Blog
{
    public int BlogId { get; set; } // EF convention will still pick this up as PK
    public string Name { get; set; }
    public string Url { get; set; }
    public string Description { get; set; }
    public byte[] RowVersion { get; set; } // For concurrency

    public ICollection<Post> Posts { get; set; } = new List<Post>();
}

// Entities/Post.cs
// No Data Annotations here for demonstration purposes
public class Post
{
    public int PostId { get; set; } // EF convention will still pick this up as PK
    public string Title { get; set; }
    public string Content { get; set; }

    public int BlogId { get; set; } // Foreign Key property
    public Blog Blog { get; set; } // Navigation property
}

// Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // All Fluent API configurations go here
        // We'll fill this in with examples below
    }
}
```

---

#### **Core Fluent API Configurations with Examples**

Let's configure our `Blog` and `Post` entities using the Fluent API within `OnModelCreating`.

```csharp
// Inside AppDbContext.cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // --- Blog Entity Configurations ---
    modelBuilder.Entity<Blog>(entity =>
    {
        // 1. Primary Key (explicitly define, even if by convention)
        entity.HasKey(b => b.BlogId);

        // 2. Table Name and Schema
        entity.ToTable("WebBlogs", "blogging"); // Maps to 'WebBlogs' table in 'blogging' schema

        // 3. Property Configurations
        entity.Property(b => b.Name)
              .IsRequired() // NOT NULL
              .HasMaxLength(200) // NVARCHAR(200)
              .HasColumnName("BlogName"); // Custom column name

        entity.Property(b => b.Url)
              .HasColumnType("varchar(500)") // Custom database type
              .HasDefaultValue("https://default.com"); // Default value for new rows

        entity.Property(b => b.Description)
              .HasMaxLength(1000); // NVARCHAR(1000), nullable by default

        // 4. Concurrency Token (Row Version)
        entity.Property(b => b.RowVersion)
              .IsRowVersion(); // Maps to 'rowversion' or 'timestamp' in SQL Server

        // 5. Index
        entity.HasIndex(b => b.Name) // Creates a non-unique index on Name
              .IsUnique(); // Makes the index unique

        // 6. Ignore Property (similar to [NotMapped])
        // If Blog had a 'TemporaryProperty'
        // entity.Ignore(b => b.TemporaryProperty);

        // 7. Data Seeding (Fluent API equivalent of HasData)
        entity.HasData(
            new Blog { BlogId = 1, Name = "My Fluent Blog", Url = "https://fluent.com/first", Description = "A blog configured with Fluent API." }
        );
    });

    // --- Post Entity Configurations ---
    modelBuilder.Entity<Post>(entity =>
    {
        // 1. Primary Key
        entity.HasKey(p => p.PostId);

        // 2. Property Configurations
        entity.Property(p => p.Title)
              .IsRequired()
              .HasMaxLength(500);

        entity.Property(p => p.Content)
              .HasColumnType("ntext"); // Example of a different column type

        // 3. Relationship Configuration (One-to-Many: Blog has many Posts)
        entity.HasOne(p => p.Blog) // A Post has one Blog
              .WithMany(b => b.Posts) // A Blog has many Posts
              .HasForeignKey(p => p.BlogId) // The foreign key property in Post
              .OnDelete(DeleteBehavior.Cascade); // Cascade delete behavior

        // 4. Composite Key Example (if Post had a composite key)
        // entity.HasKey(p => new { p.PostId, p.BlogId });

        // 5. Unique Constraint Example (if Title should be unique per Blog)
        // entity.HasIndex(p => new { p.BlogId, p.Title }).IsUnique();
    });

    // --- Example of a Many-to-Many Relationship (without a join entity in C#) ---
    // If you had a 'Tag' entity and a Blog could have many Tags, and a Tag could be on many Blogs
    // modelBuilder.Entity<Blog>()
    //     .HasMany(b => b.Tags)
    //     .WithMany(t => t.Blogs)
    //     .UsingEntity(j => j.ToTable("BlogTags")); // Creates a join table named BlogTags
}
```

Let's break down the key Fluent API methods:

1.  **`modelBuilder.Entity<TEntity>(...)`**:
    *   **Purpose:** Starts the configuration for a specific entity type.
    *   **Example:** `modelBuilder.Entity<Blog>(entity => { ... });`
    *   **Senior Insight:** This is your entry point. The `entity` parameter (often named `b` for Blog, `p` for Post) is an `EntityTypeBuilder<TEntity>` object, which provides all the methods for configuring that entity.
###### Primary Key 🗝 
1.  **`entity.HasKey(propertyExpression)`**:
    *   **Purpose:** Configures the primary key(s) for the entity.
    *   **Example:** `entity.HasKey(b => b.BlogId);`
    *   **Composite Key Example:** `entity.HasKey(p => new { p.PostId, p.BlogId });` (This is a major advantage over Data Annotations) [[Data Annotations#Primary Key 🗝]].

###### Table description
1.  **`entity.ToTable("TableName", "SchemaName")`**:
    *   **Purpose:** Specifies the database table name and optional schema.
    *   **Example:** `entity.ToTable("WebBlogs", "blogging");` [[Data Annotations#Table description]].

2.  **`entity.Property(propertyExpression)`**:
    *   **Purpose:** Starts configuration for a specific property of the entity.
    *   **Example:** `entity.Property(b => b.Name)`
    *   **Senior Insight:** This returns a `PropertyBuilder<TProperty>` object, allowing you to chain further property-specific configurations.

###### Required Prosperity 
1.  **`IsRequired()`**:
    *   **Purpose:** Configures the property as non-nullable (`NOT NULL` in the database).
    *   **Example:** `entity.Property(b => b.Name).IsRequired();` [[Data Annotations#Required Prosperity]].
 
###### String Length 
1.  **`HasMaxLength(length)`**:
    *   **Purpose:** Sets the maximum length for string or byte array properties.
    *   **Example:** `entity.Property(b => b.Name).HasMaxLength(200);` [[Data Annotations#String Length]].

###### Column description.
1.  **`HasColumnName("ColumnName")`**:
    *   **Purpose:** Specifies the database column name.
    *   **Example:** `entity.Property(b => b.Name).HasColumnName("BlogName");`

2.  **`HasColumnType("dbType")`**:
    *   **Purpose:** Specifies the exact database column type.
    *   **Example:** `entity.Property(b => b.Url).HasColumnType("varchar(500)");`
    *   **Senior Insight:** Use this when you need precise control over the database type, especially for performance or compatibility reasons (e.g., `decimal(18,2)`, `date`, `text`). [[Data Annotations#Column description]].

3.  **`HasDefaultValue(value)` / `HasDefaultValueSql("SQL_EXPRESSION")`**:
    *   **Purpose:** Sets a default value for the column when a row is inserted without providing a value for that column.
    *   **Example:** `entity.Property(b => b.Url).HasDefaultValue("https://default.com");`
    *   **Example SQL:** `entity.Property(b => b.CreatedDate).HasDefaultValueSql("GETDATE()");`
    *   **Senior Insight:** `HasDefaultValueSql` is incredibly powerful for database-generated defaults (timestamps, GUIDs, etc.) and is a common requirement in production systems.

###### Row Value
1. **`IsRowVersion()`**:
    *   **Purpose:** Configures a `byte[]` property as a concurrency token (row version).
    *   **Example:** `entity.Property(b => b.RowVersion).IsRowVersion();` [[Data Annotations#Row Value]].

2. **`HasIndex(propertyExpression)` / `HasIndex(anonymousObject)`**:
    *   **Purpose:** Creates one or more indexes on the specified property(ies).
    *   **Example (Single Column):** `entity.HasIndex(b => b.Name);`
    *   **Example (Composite Index):** `entity.HasIndex(p => new { p.BlogId, p.Title });`
    *   **Senior Insight:** Indexes are critical for query performance. Always consider adding indexes to foreign keys and columns frequently used in `WHERE` clauses, `ORDER BY`, or `JOIN` operations.

3. **`IsUnique()`**:
    *   **Purpose:** Makes an index a unique constraint.
    *   **Example:** `entity.HasIndex(b => b.Name).IsUnique();`
    *   **Senior Insight:** Essential for enforcing data integrity where certain values must be unique across a table (e.g., email addresses, usernames).

###### Not Mapped Propriety
1. **`Ignore(propertyExpression)`**:
    *   **Purpose:** Excludes a property from being mapped to the database.
    *   **Example:** `entity.Ignore(b => b.TemporaryProperty);` [[Data Annotations#Not Mapped Propriety]].

###### Foreign Key 🗝 
1. **Relationship Configuration (`HasOne`, `WithMany`, `HasForeignKey`, `OnDelete`)**:
    *   **Purpose:** Defines how entities relate to each other (one-to-many, one-to-one, many-to-many). [[Data Annotations#Foreign Key 🗝]].
    *   **One-to-Many Example:**
```csharp
entity.HasOne(p => p.Blog) // A Post has one Blog
	  .WithMany(b => b.Posts) // A Blog has many Posts
	  .HasForeignKey(p => p.BlogId) // The foreign key property in Post
	  .OnDelete(DeleteBehavior.Cascade); // What happens on principal deletion
```
*   **`OnDelete` Behaviors:**
        *   `Cascade`: Deletes dependent entities when the principal is deleted. **Use with caution!**
        *   `Restrict` (or `ClientSetNull` for optional relationships): Prevents deletion of the principal if there are dependent entities.
        *   `SetNull`: Sets foreign key values to NULL in dependent entities when the principal is deleted (requires FK column to be nullable).
        *   `NoAction`: No action is taken (database default, often `Restrict` or `NoAction` depending on DB).
    *   **Senior Insight:** Relationship configuration is one of the most powerful aspects of Fluent API. Pay close attention to `OnDelete` behavior, as incorrect configuration can lead to unintended data loss or referential integrity errors. Always choose the most appropriate behavior for your domain.

15. **`UsingEntity(...)` (for Many-to-Many without explicit join entity)**:
    *   **Purpose:** Configures the join table for a many-to-many relationship where you don't have a dedicated C# entity for the join table.
    *   **Example:** `modelBuilder.Entity<Blog>().HasMany(b => b.Tags).WithMany(t => t.Blogs).UsingEntity(j => j.ToTable("BlogTags"));`
    *   **Senior Insight:** This simplifies many-to-many relationships, but if you need to add extra properties to the join table (e.g., `DateAdded` to `BlogTag`), you'll need to create an explicit join entity and configure it as two one-to-many relationships.
      
16. `HasComputedColumnSql("")` method is used to specify that the property should map to a computed column. The method takes a string indicating the expression used to generate the default value for a database column.
---

#### **Advanced Fluent API & Senior Considerations**
#Important_Note
#Clean_Architecture
1.  **Organizing `OnModelCreating` (Clean Architecture)**
    For larger applications, `OnModelCreating` can become very long and hard to manage.
    *   **Separate Configuration Classes:** Create a separate class for each entity's configuration that implements `IEntityTypeConfiguration<TEntity>`.

```csharp
// Configurations/BlogConfiguration.cs
public class BlogConfiguration : IEntityTypeConfiguration<Blog>
{
	public void Configure(EntityTypeBuilder<Blog> builder)
	{
		builder.HasKey(b => b.BlogId);
		builder.ToTable("WebBlogs", "blogging");
		// ... other Blog configurations ...
	}
}

// In AppDbContext.cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	modelBuilder.ApplyConfiguration(new BlogConfiguration());
	modelBuilder.ApplyConfiguration(new PostConfiguration());
	// Or to apply all configurations from an assembly:
	// modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
}
```
>*Senior Insight:* This is the recommended approach for maintainability, testability, and adhering to the Single Responsibility Principle in larger projects.

2.  **Inheritance Mapping:**
    Fluent API is used to configure Table-Per-Hierarchy (TPH), Table-Per-Type (TPT), and Table-Per-Concrete-Type (TPC) inheritance strategies.
    *   **TPH (Default):** `modelBuilder.Entity<BaseEntity>().HasDiscriminator<string>("DiscriminatorColumn").HasValue<Derived1>("Type1").HasValue<Derived2>("Type2");`
    *   **TPT:** `modelBuilder.Entity<DerivedEntity>().ToTable("DerivedTable");`
    *   **TPC:** Requires more advanced configuration.
    *   *Senior Insight:* Understanding inheritance mapping is crucial for modeling complex domain hierarchies. TPH is often the simplest and most performant, but TPT/TPC might be necessary for specific database design requirements.

3.  **Value Conversions:**
    Map properties of one type in your C# model to a different type in the database.
    *   **Example:** Storing an `enum` as a `string` in the database.
```csharp
entity.Property(b => b.Status)
	  .HasConversion<string>(); // Converts enum to string for DB storage
```
>  *Senior Insight:* Extremely useful for handling complex types, value objects, or when you need to store data in a specific format in the database that doesn't directly map to a C# primitive.

4.  **Shadow Properties:**
    Properties that are part of the EF Core model but do not exist in the .NET entity class. Their value and state are maintained purely by EF Core.
    *   **Example:** `modelBuilder.Entity<Blog>().Property<DateTime>("LastUpdated");`
    *   *Senior Insight:* Useful for audit fields (e.g., `CreatedBy`, `LastModifiedDate`) that you want EF Core to manage without cluttering your domain entities. You access them via `dbContext.Entry(entity).Property("PropertyName").CurrentValue`.

5.  **Precedence:**
    Remember, Fluent API configurations always override Data Annotations and EF Core conventions. This gives you ultimate control.

6.  **Migrations Impact:**
    Any changes made in `OnModelCreating` using Fluent API will be detected by `dotnet ef migrations add` and translated into schema changes in your migration files. Always generate a new migration after making Fluent API changes.
