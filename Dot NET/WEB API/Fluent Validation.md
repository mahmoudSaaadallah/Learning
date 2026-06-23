### What is FluentValidation?

**FluentValidation** is a popular .NET library for building strongly-typed validation rules. Unlike `System.ComponentModel.DataAnnotations` attributes, which are declarative and often limited to simple, property-level checks, FluentValidation uses a **fluent interface** (we'll explain this shortly!) to define validation rules in a more expressive and programmatic way.

It allows you to:

*   Define validation rules for a specific type (e.g., a DTO, a command, a domain model).
*   Group related validation rules together in a single validator class.
*   Perform complex, conditional, and cross-property validation with ease.
*   Integrate seamlessly with ASP.NET Core's model validation pipeline.
*   Support asynchronous validation.
*   Inject dependencies into your validators (e.g., a service to check for unique usernames).

### Why FluentValidation over `System.ComponentModel.DataAnnotations`?

While `DataAnnotations` attributes are simple and built-in, they have limitations that FluentValidation addresses:

1.  **Separation of Concerns:** `DataAnnotations` attributes mix validation logic directly into your DTOs/models. FluentValidation separates validation rules into dedicated validator classes, keeping your models clean and focused on data representation.
2.  **Complex Logic:** `DataAnnotations` struggle with:
    *   **Cross-property validation:** (e.g., `EndDate` must be after `StartDate`). While possible with custom class-level attributes, it can become cumbersome.
    *   **Conditional validation:** (e.g., `CreditCardNumber` is required *only if* `PaymentMethod` is 'CreditCard').
    *   **Asynchronous validation:** (e.g., checking if a username is unique in the database).
    *   **Dependency Injection:** `DataAnnotations` attributes cannot easily inject services.
3.  **Readability and Maintainability:** FluentValidation's fluent API often makes complex validation rules more readable and easier to maintain than a collection of custom `ValidationAttribute` classes.
4.  **Testability:** Validator classes are plain C# classes, making them very easy to unit test in isolation.
5.  **Error Messages:** FluentValidation provides powerful ways to customize error messages, including placeholders and conditional messages.

For any non-trivial validation, FluentValidation is almost always the superior choice in a production environment.

### Getting Started with FluentValidation

#### 1. Installation

You'll typically need two NuGet packages for an ASP.NET Core application:

```bash
dotnet add package FluentValidation
dotnet add package FluentValidation.AspNetCore
```

#### 2. Define Your DTO/Model

Let's reuse our `CreateEventDto` and `CreateBookingDto` from the previous discussion, but this time, we'll remove the `DataAnnotations` attributes from them to keep them clean.

```csharp
// DTOs/CreateEventDto.cs
public class CreateEventDto
{
    public string Name { get; set; }
    public DateTime EventDate { get; set; }
    public string Description { get; set; }
    public int MaxAttendees { get; set; }
}

// DTOs/CreateBookingDto.cs
public class CreateBookingDto
{
    public Guid CustomerId { get; set; }
    public DateTime StartDate { get; set; }
    public DateTime EndDate { get; set; }
    public int NumberOfGuests { get; set; }
    public string SpecialRequests { get; set; }
}
```

#### 3. Create a Validator Class

For each DTO/model you want to validate, you create a corresponding validator class that inherits from `AbstractValidator<T>`.

```csharp
// Validators/CreateEventDtoValidator.cs
using FluentValidation;

public class CreateEventDtoValidator : AbstractValidator<CreateEventDto>
{
    public CreateEventDtoValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Event name is required.")
            .Length(3, 100).WithMessage("Event name must be between 3 and 100 characters.");

        RuleFor(x => x.EventDate)
            .NotEmpty().WithMessage("Event date is required.")
            .Must(BeAFutureDate).WithMessage("The event date must be in the future.");

        RuleFor(x => x.Description)
            .MaximumLength(500).WithMessage("Description cannot exceed 500 characters.");

        RuleFor(x => x.MaxAttendees)
            .GreaterThan(0).WithMessage("Maximum attendees must be greater than 0.");
    }

    private bool BeAFutureDate(DateTime date)
    {
        return date > DateTime.UtcNow;
    }
}
```

**Explanation:**

*   `RuleFor(x => x.Name)`: Specifies the property to which the following rules apply.
*   `.NotEmpty()`: A built-in validator ensuring the string is not null or empty.
*   `.Length(3, 100)`: Ensures the string length is within the specified range.
*   `.WithMessage(...)`: Customizes the error message for the preceding rule.
*   `.Must(BeAFutureDate)`: Allows you to define custom validation logic using a method or a lambda expression.

#### 4. Integrate with ASP.NET Core

To make ASP.NET Core automatically use your FluentValidation validators during model binding, you need to register them in `Program.cs`.

```csharp
// Program.cs
using FluentValidation;
using FluentValidation.AspNetCore; // Required for AddFluentValidationAutoValidation

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();


// Register FluentValidation
builder.Services.AddFluentValidationAutoValidation(); // Enables automatic validation
builder.Services.AddValidatorsFromAssemblyContaining<CreateEventDtoValidator>(); // Scans assembly for validators


>>>>>>>>>>>>>>>>>>>>>>>>>>>>
// Another way to Inject the FluentValidation 
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
<<<<<<<<<<<<<<<<<<<<<<<<<<<<


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
app.Run();
```

Now, your controllers will automatically trigger FluentValidation when a `CreateEventDto` is received:

```csharp
// Controllers/EventsController.cs
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class EventsController : ControllerBase
{
    [HttpPost]
    public IActionResult CreateEvent([FromBody] CreateEventDto createEventDto)
    {
        // With AddFluentValidationAutoValidation, ModelState.IsValid will be populated
        // by FluentValidation. You don't need to manually call a validator.
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState); // Returns 400 with validation errors
        }

        return Ok("Event created successfully (mock).");
    }
}
```

### Advanced FluentValidation Features

#### 1. Cross-Property Validation

Let's validate that `EndDate` is after `StartDate` for `CreateBookingDto`.

```csharp
// Validators/CreateBookingDtoValidator.cs
using FluentValidation;

public class CreateBookingDtoValidator : AbstractValidator<CreateBookingDto>
{
    public CreateBookingDtoValidator()
    {
        RuleFor(x => x.CustomerId)
            .NotEmpty().WithMessage("Customer ID is required.");

        RuleFor(x => x.StartDate)
            .NotEmpty().WithMessage("Start date is required.")
            .Must(BeAFutureDate).WithMessage("Start date must be in the future.");

        RuleFor(x => x.EndDate)
            .NotEmpty().WithMessage("End date is required.")
            .Must(BeAFutureDate).WithMessage("End date must be in the future.")
            .GreaterThan(x => x.StartDate).WithMessage("End date must be after the start date."); // Cross-property validation

        RuleFor(x => x.NumberOfGuests)
            .InclusiveBetween(1, 10).WithMessage("Number of guests must be between 1 and 10.");

        RuleFor(x => x.SpecialRequests)
            .MaximumLength(200).WithMessage("Special requests cannot exceed 200 characters.");
    }

    private bool BeAFutureDate(DateTime date)
    {
        return date > DateTime.UtcNow;
    }
}
```
Notice `GreaterThan(x => x.StartDate)` which directly compares `EndDate` to `StartDate` of the same object.

#### 2. Conditional Validation (`When`, `Unless`)

Imagine `SpecialRequests` is only allowed if `NumberOfGuests` is greater than 5.

```csharp
// Inside CreateBookingDtoValidator constructor
RuleFor(x => x.SpecialRequests)
    .MaximumLength(200).WithMessage("Special requests cannot exceed 200 characters.")
    .When(x => x.NumberOfGuests > 5); // Only validate if NumberOfGuests > 5

// Or, if a property is required conditionally:
RuleFor(x => x.SpecialRequests)
    .NotEmpty().WithMessage("Special requests are required for large bookings.")
    .When(x => x.NumberOfGuests > 5);
```

#### 3. Asynchronous Validation (Database Checks)

Let's say we want to ensure an event name is unique. This requires a database lookup, which is an asynchronous operation.

First, we need a service to check uniqueness:

```csharp
// Services/IEventService.cs
public interface IEventService
{
    Task<bool> IsEventNameUniqueAsync(string name, CancellationToken cancellationToken = default);
}

// Services/EventService.cs (Mock implementation)
public class EventService : IEventService
{
    private readonly List<string> _existingEventNames = new List<string> { "Tech Conference 2026", "Dev Summit" };

    public Task<bool> IsEventNameUniqueAsync(string name, CancellationToken cancellationToken = default)
    {
        // Simulate database call delay
        return Task.FromResult(!_existingEventNames.Contains(name, StringComparer.OrdinalIgnoreCase));
    }
}
```

Now, inject `IEventService` into your validator:

```csharp
// Validators/CreateEventDtoValidator.cs
using FluentValidation;

public class CreateEventDtoValidator : AbstractValidator<CreateEventDto>
{
    public CreateEventDtoValidator(IEventService eventService) // Inject service
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Event name is required.")
            .Length(3, 100).WithMessage("Event name must be between 3 and 100 characters.")
            .MustAsync(async (name, cancellation) => await eventService.IsEventNameUniqueAsync(name, cancellation))
            .WithMessage("An event with this name already exists.");

        // ... other rules
    }

    private bool BeAFutureDate(DateTime date)
    {
        return date > DateTime.UtcNow;
    }
}
```

**Important:** When using `AddValidatorsFromAssemblyContaining`, FluentValidation's DI integration will automatically resolve `IEventService` for your validator. You just need to ensure `IEventService` and `EventService` are registered in your DI container (e.g., `builder.Services.AddScoped<IEventService, EventService>();`).

### FluentValidation vs. Fluent API: The Distinction

This is a common point of confusion, so let's clarify.
[[Fluent API]]
1.  **Fluent API (Design Pattern):**
    *   A **Fluent API** (or Fluent Interface) is a **design pattern** that aims to make code more readable and expressive by chaining method calls. Each method in the chain typically returns the current object instance, allowing you to call another method on it immediately.
    *   The goal is to create a domain-specific language (DSL) that reads almost like natural language.
    *   **Examples of Fluent APIs in .NET:**
        *   **LINQ:** `myCollection.Where(x => x.Age > 18).OrderBy(x => x.Name).Select(x => x.Name);`
        *   **Entity Framework Core:** `dbContext.Users.Where(u => u.IsActive).Include(u => u.Orders).ToList();`
        *   **ASP.NET Core Builder:** `app.UseRouting().UseAuthentication().UseAuthorization().MapControllers();`
        *   **Mapster Configuration:** `TypeAdapterConfig<Product, ProductDto>.NewConfig().Map(...).Ignore(...);`

2.  **FluentValidation (Library that uses a Fluent API):**
    *   **FluentValidation** is a **specific .NET library** designed for validation.
    *   It *uses* the Fluent API design pattern to define its validation rules. The `RuleFor(x => x.Name).NotEmpty().Length(3, 100).WithMessage(...)` syntax is a prime example of a Fluent API in action.

**In summary:** FluentValidation is a library, and its configuration syntax is an example of a Fluent API. They are not competing concepts; rather, FluentValidation leverages the benefits of the Fluent API pattern to provide a powerful and readable way to define validation rules.

### Production-Level Considerations

1.  **Layered Validation:**
    *   **API Input (DTOs) with FluentValidation:** This is your primary validation layer. Catch all structural and basic business rule violations here.
    *   **Service Layer (Business Logic):** For complex business rules that require database lookups, external service calls, or intricate domain logic that's too complex for a validator, handle it in your service layer. FluentValidation can handle *some* async/dependency-based validation, but if it becomes too heavy, the service layer is better.
    *   **Domain Models (Invariants):** If you're using DDD, your domain models should enforce their own invariants (rules that ensure the object is always in a valid state).
2.  **Error Handling and Response:** Ensure your API consistently returns `400 Bad Request` with a clear, structured error payload when FluentValidation fails. ASP.NET Core's default `BadRequest(ModelState)` is a good start, but you might want to customize it for a more consistent API experience.
3.  **Performance:** FluentValidation is generally very performant. For extremely high-throughput scenarios, you can pre-compile validators, but for most applications, the default setup is sufficient.
4.  **Testing:** Write unit tests for your validator classes. They are plain C# classes, making them easy to instantiate and test with various inputs.
5.  **Localization:** FluentValidation supports localization of error messages, which is important for international applications.
6.  **Reusability:** You can create reusable validation rules and even compose validators for complex object graphs.

### Senior Insight

As a senior developer, I consider FluentValidation an essential tool for almost any modern .NET backend project.

1.  **Embrace Separation:** The biggest win with FluentValidation is the clean separation of validation logic from your DTOs. This makes your models easier to read, your validation logic easier to find, and both easier to test independently.
2.  **Don't Over-Validate in Attributes:** Once you adopt FluentValidation, resist the urge to use `DataAnnotations` attributes for anything beyond the most basic `[Required]` or `[JsonIgnore]` (which aren't validation anyway). Keep your DTOs clean.
3.  **Contextual Validation:** FluentValidation excels at contextual validation. You might have different validation rules for `CreateProductDto` versus `UpdateProductDto`. You can easily define separate validators for each, or use `RuleSet`s within a single validator.
4.  **Dependency Injection is Powerful:** The ability to inject services into validators (e.g., `IUserRepository` to check for unique emails) is a game-changer. This allows you to push more complex, state-dependent validation closer to the input, failing fast before hitting deeper business logic. However, be mindful of making validators too "heavy" by injecting too many services; sometimes, a complex check is still better in the service layer.
5.  **Readability is Key:** The fluent syntax, when used well, makes validation rules incredibly readable. Treat your validators as living documentation of your input constraints.
6.  **Consistency:** Standardize your validation approach across the entire application. If you choose FluentValidation, use it consistently. This reduces cognitive load for other developers.

FluentValidation is a significant upgrade over `DataAnnotations` for most real-world applications. It provides the flexibility, power, and maintainability needed to handle complex validation requirements effectively, while its use of a Fluent API makes the validation rules clear and expressive.

------------------------------
## Dynamic Error Messages


FluentValidation allows you to embed various placeholders into your `WithMessage()` calls, which are then replaced with actual values at runtime when a validation error occurs. This makes your error messages much more descriptive without hardcoding values.

The two most common and useful placeholders are indeed `{PropertyName}` and `{PropertyValue}`.

### Understanding `{PropertyName}` and `{PropertyValue}`

*   **`{PropertyName}`**: This placeholder will be replaced by the **name of the property** that the validation rule is applied to.
    *   Example: If you have `RuleFor(x => x.Name)` and the validation fails, `{PropertyName}` will become "Name".
*   **`{PropertyValue}`**: This placeholder will be replaced by the **actual value of the property** that failed validation.
    *   Example: If `RuleFor(x => x.Age).GreaterThan(18)` fails for an `Age` of `16`, `{PropertyValue}` will become "16".

### How to Use Them in `WithMessage()`

You simply include them within the string you pass to `WithMessage()`. FluentValidation will automatically parse and replace them.

Let's revisit our `CreateEventDtoValidator` and enhance its error messages using these placeholders.

#### Example: Enhanced `CreateEventDtoValidator`

```csharp
// DTOs/CreateEventDto.cs (No DataAnnotations)
public class CreateEventDto
{
    public string Name { get; set; }
    public DateTime EventDate { get; set; }
    public string Description { get; set; }
    public int MaxAttendees { get; set; }
    public decimal TicketPrice { get; set; }
}

// Validators/CreateEventDtoValidator.cs
using FluentValidation;

public class CreateEventDtoValidator : AbstractValidator<CreateEventDto>
{
    private readonly IEventService _eventService; // Assuming this is injected

    public CreateEventDtoValidator(IEventService eventService)
    {
        _eventService = eventService;

        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("The event {PropertyName} cannot be empty.")
            .Length(3, 100).WithMessage("The {PropertyName} must be between {MinLength} and {MaxLength} characters. You entered {TotalLength} characters.")
            .MustAsync(async (name, cancellation) => await _eventService.IsEventNameUniqueAsync(name, cancellation))
            .WithMessage("An event with the name '{PropertyValue}' already exists. Please choose a different name.");

        RuleFor(x => x.EventDate)
            .NotEmpty().WithMessage("The {PropertyName} is required.")
            .Must(BeAFutureDate).WithMessage("The {PropertyName} must be a future date. You provided '{PropertyValue:yyyy-MM-dd}'."); // Custom format for DateTime

        RuleFor(x => x.Description)
            .MaximumLength(500).WithMessage("The {PropertyName} cannot exceed {MaxLength} characters.");

        RuleFor(x => x.MaxAttendees)
            .GreaterThan(0).WithMessage("The {PropertyName} must be greater than {ComparisonValue}. Current value: {PropertyValue}.");

        RuleFor(x => x.TicketPrice)
            .GreaterThanOrEqualTo(0).WithMessage("The {PropertyName} cannot be negative. Value: {PropertyValue}.");
    }

    private bool BeAFutureDate(DateTime date)
    {
        return date > DateTime.UtcNow;
    }
}
```

#### Let's trace some potential error messages:

1.  **`Name` is empty:**
    *   Input: `{ "Name": "", "EventDate": "2027-01-01", ... }`
    *   Error: "The event Name cannot be empty."
    *   Here, `{PropertyName}` became "Name".

2.  **`Name` is too short (e.g., "ab"):**
    *   Input: `{ "Name": "ab", "EventDate": "2027-01-01", ... }`
    *   Error: "The Name must be between 3 and 100 characters. You entered 2 characters."
    *   Here, `{PropertyName}` became "Name", `{MinLength}` became "3", `{MaxLength}` became "100", and `{TotalLength}` became "2".

3.  **`Name` is not unique (e.g., "Tech Conference 2026"):**
    *   Input: `{ "Name": "Tech Conference 2026", "EventDate": "2027-01-01", ... }`
    *   Error: "An event with the name 'Tech Conference 2026' already exists. Please choose a different name."
    *   Here, `{PropertyValue}` became "Tech Conference 2026".

4.  **`EventDate` is in the past (e.g., "2025-01-01"):**
    *   Input: `{ "Name": "New Event", "EventDate": "2025-01-01", ... }`
    *   Error: "The EventDate must be a future date. You provided '2025-01-01'."
    *   Here, `{PropertyName}` became "EventDate", and `{PropertyValue:yyyy-MM-dd}` formatted the `DateTime` value.

5.  **`MaxAttendees` is 0:**
    *   Input: `{ "Name": "New Event", "EventDate": "2027-01-01", "MaxAttendees": 0, ... }`
    *   Error: "The MaxAttendees must be greater than 0. Current value: 0."
    *   Here, `{PropertyName}` became "MaxAttendees", `{ComparisonValue}` became "0" (from `GreaterThan(0)`), and `{PropertyValue}` became "0".

### Other Useful Placeholders

FluentValidation provides many other context-specific placeholders depending on the validator you're using:

*   **`{MinLength}` / `{MaxLength}` / `{TotalLength}`**: For `Length` and `MinimumLength`/`MaximumLength` validators.
*   **`{Min}` / `{Max}`**: For `Range` validators.
*   **`{ComparisonValue}`**: For comparison validators like `GreaterThan`, `LessThan`, `Equal`, etc. This is the value you are comparing against.
*   **`{ExpectedPrecision}` / `{ExpectedScale}`**: For `ScalePrecision` validators.
*   **`{CollectionIndex}`**: When validating items within a collection (e.g., `RuleForEach`).
*   **`{ValidatorType}`**: The type of the validator that failed.

You can also use standard C# string formatting within `{PropertyValue}` for `DateTime` or numeric types, as shown with `{PropertyValue:yyyy-MM-dd}`.

### Production-Level Considerations

1.  **User Experience:** Dynamic error messages significantly improve the user experience for API consumers (whether they are frontend developers or other services). They get precise information about *what* went wrong and *why*.
2.  **Debugging:** Clear error messages also aid in debugging, making it easier for developers to understand validation failures.
3.  **Localization:** When dealing with multiple languages, these placeholders are invaluable. You can provide localized error message templates, and the placeholders will still be correctly substituted.
4.  **Consistency:** Using placeholders helps maintain consistency in your error messages across different properties and validators.
5.  **Avoid Hardcoding:** Never hardcode values like "3" or "100" in your error messages if they are derived from the validation rule itself. Always use the appropriate placeholders (`{MinLength}`, `{MaxLength}`). This prevents discrepancies if you change the rule but forget to update the message.
