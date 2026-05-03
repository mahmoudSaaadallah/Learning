#### **1. Creating a New .NET Project**

You can start with either a Console Application for simplicity or a Web API project, which is more representative of real-world scenarios where EF Core shines.

**Option A: Console Application (Recommended for initial learning)**

This is great for quickly testing EF Core features without the overhead of a web framework.

1.  **Open your terminal or command prompt.**
2.  **Navigate to your desired directory.**
3.  **Create the project:**
```bash
dotnet new console -n EfCoreGettingStarted
cd EfCoreGettingStarted
```
This creates a new console application named `EfCoreGettingStarted` and navigates into its directory.

**Option B: Web API Project (For a more realistic context)**

If you're eager to see EF Core in a web context, you can start with a Web API.

1.  **Open your terminal or command prompt.**
2.  **Navigate to your desired directory.**
3.  **Create the project:**
```bash
dotnet new webapi -n EfCoreWebApi
cd EfCoreWebApi
```
This creates a new ASP.NET Core Web API project named `EfCoreWebApi`.

For this guide, I'll primarily use the **Console Application** for clarity, but I'll show the Web API configuration steps where relevant.

---

#### **2. Installing Necessary NuGet Packages**

Once your project is created, you need to add the EF Core packages.

1.  **`Microsoft.EntityFrameworkCore.SqlServer`**: This is the database provider for SQL Server. If you were using PostgreSQL, you'd install `Npgsql.EntityFrameworkCore.PostgreSQL`; for SQLite, `Microsoft.EntityFrameworkCore.Sqlite`, etc.
2.  **`Microsoft.EntityFrameworkCore.Tools`**: This package provides the EF Core command-line interface (CLI) tools, which are essential for managing migrations (creating and updating your database schema).

**From your project directory (e.g., `EfCoreGettingStarted` or `EfCoreWebApi`), run the following commands:**

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

**Explanation:**
*   `dotnet add package`: The command to add a NuGet package to your project.
*   `Microsoft.EntityFrameworkCore.SqlServer`: Enables EF Core to communicate with SQL Server databases.
*   `Microsoft.EntityFrameworkCore.Tools`: Provides commands like `dotnet ef migrations add` and `dotnet ef database update`. These tools are crucial for the "Code-First" approach we're using.

---

#### **3. Defining Your Entity (Model)**

Before we configure the `DbContext`, let's define a simple C# class that will represent a table in our database. We'll reuse our `Product` class.

Create a new file named `Product.cs` in your project and add the following:

```csharp
// Product.cs
using System;

public class Product
{
    public int Id { get; set; } // EF Core convention: 'Id' or 'ProductId' will be the primary key
    public string Name { get; set; }
    public decimal Price { get; set; }
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow; // Default value for convenience
}
```
**Note:** EF Core follows conventions. By default, a property named `Id` or `<ClassName>Id` will be recognized as the primary key.

---

#### **4. Configuring the DbContext and Connection String**

The `DbContext` is the central class in EF Core. It's responsible for:
*   Managing database connections.
*   Querying data from the database.
*   Tracking changes to your entities.
*   Saving changes back to the database.

You'll create a class that inherits from `Microsoft.EntityFrameworkCore.DbContext`.

##### **Option A: Simple Configuration using `OnConfiguring` (Good for Console Apps)**

This method is straightforward for simple applications like console apps where you might not be using a full Dependency Injection container.

Create a new file named `ApplicationDbContext.cs`:

```csharp
// ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    // DbSet represents a collection of all entities in the context,
    // or that can be queried from the database.
    public DbSet<Product> Products { get; set; }

    // This method is called by EF Core to configure the database provider and connection string.
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // Configure to use SQL Server with a local database.
        // (localdb)\mssqllocaldb is a common development instance of SQL Server Express.
        // Ensure MultipleActiveResultSets=true if you might have multiple active data readers on a single connection.
        optionsBuilder.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=EfCoreGettingStartedDb;Trusted_Connection=True;MultipleActiveResultSets=true");
    }

    // You can also use OnModelCreating for more advanced model configuration (Fluent API)
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Example: Ensure Name is required and has a max length
        modelBuilder.Entity<Product>()
            .Property(p => p.Name)
            .IsRequired()
            .HasMaxLength(255);
    }
}
```

**Using it in `Program.cs` (Console App):**

```csharp
// Program.cs
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore; // Needed for MigrateAsync

public class Program
{
    public static async Task Main(string[] args)
    {
        // Instantiate the DbContext directly
        using (var dbContext = new ApplicationDbContext())
        {
            // Ensure the database is created and migrations are applied.
            // In a real application, you'd typically run migrations via CLI or during deployment.
            await dbContext.Database.MigrateAsync();
            Console.WriteLine("Database ensured and migrations applied.");

            // Add a new product
            var newProduct = new Product { Name = "Gaming Keyboard", Price = 79.99m };
            dbContext.Products.Add(newProduct);
            await dbContext.SaveChangesAsync();
            Console.WriteLine($"Added product: {newProduct.Name} with ID: {newProduct.Id}");

            // Retrieve all products
            var products = await dbContext.Products.ToListAsync();
            Console.WriteLine("\nAll Products:");
            foreach (var p in products)
            {
                Console.WriteLine($"- {p.Id}: {p.Name} (${p.Price:F2})");
            }
        }
    }
}
```

##### **Option B: Configuration using `appsettings.json` and Dependency Injection (Preferred for Web Apps)**

This is the standard and recommended approach for ASP.NET Core applications. It separates configuration from code, makes your `DbContext` easily testable, and integrates seamlessly with the framework's DI container.

**Step 1: Add `appsettings.json`**

In your project root, create a file named `appsettings.json` (if it doesn't exist, which it will for a Web API project).

```json
// appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=EfCoreWebApiDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```
**Note:** For a console app, you'd also need to install `Microsoft.Extensions.Configuration.Json` and `Microsoft.Extensions.Configuration.FileExtensions` to load `appsettings.json`. For a Web API, this is handled automatically.

**Step 2: Modify `ApplicationDbContext.cs`**

Remove the `OnConfiguring` method. The `DbContextOptions` will now be passed in via the constructor by the DI container.

```csharp
// ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    // Constructor now accepts DbContextOptions
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>()
            .Property(p => p.Name)
            .IsRequired()
            .HasMaxLength(255);
    }
}
```

**Step 3: Configure DI in `Program.cs` (for both Console and Web API)**

This is where you tell the .NET host how to create instances of your `DbContext` and provide it with the connection string from `appsettings.json`.

**For a Console Application (requires additional NuGet packages):**

You'll need to install `Microsoft.Extensions.Hosting` and `Microsoft.Extensions.Configuration.Json` and `Microsoft.Extensions.Configuration.FileExtensions` to mimic the DI setup of a web app.

```bash
dotnet add package Microsoft.Extensions.Hosting
dotnet add package Microsoft.Extensions.Configuration.Json
dotnet add package Microsoft.Extensions.Configuration.FileExtensions
```

Then, modify your `Program.cs`:

```csharp
// Program.cs (Console App with DI)
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration; // For IConfiguration
using Microsoft.Extensions.DependencyInjection; // For IServiceCollection
using Microsoft.Extensions.Hosting; // For Host.CreateDefaultBuilder

public class Program
{
    public static async Task Main(string[] args)
    {
        // Create a host builder to configure services and load appsettings.json
        var host = Host.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((context, config) =>
            {
                config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
            })
            .ConfigureServices((context, services) =>
            {
                // Get connection string from configuration
                var connectionString = context.Configuration.GetConnectionString("DefaultConnection");

                // Register DbContext with DI
                services.AddDbContext<ApplicationDbContext>(options =>
                    options.UseSqlServer(connectionString));

                // Register a service that uses the DbContext (optional, for demonstration)
                services.AddTransient<ProductService>();
            })
            .Build();

        // Get the service scope to resolve services
        using (var scope = host.Services.CreateScope())
        {
            var serviceProvider = scope.ServiceProvider;
            var dbContext = serviceProvider.GetRequiredService<ApplicationDbContext>();
            var productService = serviceProvider.GetRequiredService<ProductService>();

            // Ensure database is created and migrations applied
            await dbContext.Database.MigrateAsync();
            Console.WriteLine("Database ensured and migrations applied.");

            // Use the service to perform operations
            await productService.AddAndListProductsAsync();
        }

        await host.RunAsync(); // Keep the console app running if needed, though not strictly necessary for this example
    }
}

// A simple service to demonstrate using DbContext via DI
public class ProductService
{
    private readonly ApplicationDbContext _dbContext;

    public ProductService(ApplicationDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task AddAndListProductsAsync()
    {
        var newProduct = new Product { Name = "Mechanical Keyboard", Price = 120.50m };
        _dbContext.Products.Add(newProduct);
        await _dbContext.SaveChangesAsync();
        Console.WriteLine($"Added product: {newProduct.Name} with ID: {newProduct.Id}");

        var products = await _dbContext.Products.ToListAsync();
        Console.WriteLine("\nAll Products:");
        foreach (var p in products)
        {
            Console.WriteLine($"- {p.Id}: {p.Name} (${p.Price:F2})");
        }
    }
}
```

**For a Web API Project (standard `Program.cs`):**

```csharp
// Program.cs (Web API)
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration; // Not explicitly needed, but good to know it's there

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Configure DbContext with SQL Server and connection string from appsettings.json
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

// Optional: Apply migrations on startup (useful for dev/test, but consider alternatives for production)
using (var scope = app.Services.CreateScope())
{
    var dbContext = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    await dbContext.Database.MigrateAsync();
    Console.WriteLine("Database ensured and migrations applied on startup.");
}

app.Run();
```

---

**Summary of Configuration Approaches:**

| Feature                 | `OnConfiguring` Method                               | `appsettings.json` + Dependency Injection (DI)                               |
| :---------------------- | :--------------------------------------------------- | :--------------------------------------------------------------------------- |
| **Use Case**            | Simple console apps, quick tests.                    | ASP.NET Core web apps, larger applications, microservices.                   |
| **Connection String**   | Hardcoded in `DbContext` class.                      | Stored in `appsettings.json` (or environment variables, Azure Key Vault).    |
| **Flexibility**         | Less flexible, requires code change for connection.  | Highly flexible, easy to change connection string per environment.           |
| **Testability**         | Harder to unit test `DbContext` directly.            | Easy to mock `DbContextOptions` or use in-memory databases for testing.      |
| **Best Practice**       | Generally discouraged for production applications.   | **Recommended** for modern .NET applications.                                |
| **Dependencies**        | Minimal.                                             | Requires `Microsoft.Extensions.Hosting`, `Microsoft.Extensions.Configuration` packages (often included in web templates). |

---

You now have a project set up with EF Core, an entity, and a `DbContext` configured to connect to a SQL Server database. The next step in our plan will be to actually create the database schema using **Database Migrations**!