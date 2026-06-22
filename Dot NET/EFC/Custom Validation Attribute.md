### What are Validation Attributes?

In .NET, **Validation Attributes** (part of the `System.ComponentModel.DataAnnotations` namespace) are a declarative way to add validation rules to properties of your classes. When these classes are used as input models in ASP.NET Core (e.g., in an `[ApiController]` method with `[FromBody]`), the framework automatically performs validation based on these attributes during model binding.

Examples of built-in validation attributes include:
[[Data Annotations]]
*   `[Required]`
*   `[StringLength(maxLength, minLength)]`
*   `[Range(minimum, maximum)]`
*   `[EmailAddress]`
*   `[RegularExpression(pattern)]`
*   `[MinLength]`, `[MaxLength]` (from .NET 6+)
*   `[Length(min, max)]` (from .NET 8+ )

When validation fails, ASP.NET Core populates `ModelState` with errors, which can then be returned to the client, typically as a `400 Bad Request` response.

### Why Do We Need Custom Validation Attributes?

While the built-in attributes cover many common scenarios, you'll frequently encounter business rules that are unique to your application and cannot be expressed with standard attributes.

For example:

*   **Cross-property validation:** "The `EndDate` must be after the `StartDate`."
*   **Conditional validation:** "If `PaymentType` is 'CreditCard', then `CardNumber` is required."
*   **Complex business rules:** "A product's `DiscountPercentage` cannot exceed 50% if its `Category` is 'Electronics'."
*   **Database-dependent validation:** "The `Username` must be unique in the database." (Though this is often better handled in the service layer).
*   **Specific format validation:** "A custom product code must follow a specific internal format."

In these cases, a custom validation attribute allows you to encapsulate that specific business rule directly on the property or class it applies to.

### How to Create a Custom Validation Attribute

To create a custom validation attribute, you typically follow these steps:

1.  **Inherit from `ValidationAttribute`**: This is the base class for all validation attributes.
2.  **Override the `IsValid` method**: This is where your custom validation logic resides.
    *   `IsValid(object value, ValidationContext validationContext)`: This overload is preferred for more complex scenarios, especially when you need access to other properties of the object being validated or services from the DI container.
    *   `IsValid(object value)`: A simpler overload if you only need to validate the property's value itself.
3.  **Provide an error message**: You can set a default `ErrorMessage` property or use `FormatErrorMessage` for dynamic messages.

Let's create a practical example: a `FutureDateAttribute` that ensures a `DateTime` property is in the future.

#### Example 1: Simple Property-Level Validation (`FutureDateAttribute`)

This attribute will check if a `DateTime` property's value is in the future.

```csharp
// Attributes/FutureDateAttribute.cs
using System.ComponentModel.DataAnnotations;

public class FutureDateAttribute : ValidationAttribute
{
    public FutureDateAttribute()
    {
        // Set a default error message
        ErrorMessage = "The {0} must be a future date.";
    }

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (value == null)
        {
            // If the property is not required, null is valid.
            // Use [Required] alongside this if the date must be present.
            return ValidationResult.Success;
        }

        if (value is DateTime dateTime)
        {
            if (dateTime > DateTime.UtcNow)
            {
                return ValidationResult.Success;
            }
            else
            {
                // Use FormatErrorMessage to include the property name ({0})
                return new ValidationResult(FormatErrorMessage(validationContext.DisplayName));
            }
        }
        else if (value is DateTimeOffset dateTimeOffset)
        {
            if (dateTimeOffset > DateTimeOffset.UtcNow)
            {
                return ValidationResult.Success;
            }
            else
            {
                return new ValidationResult(FormatErrorMessage(validationContext.DisplayName));
            }
        }

        // If the type is not DateTime or DateTimeOffset, it's an invalid usage of the attribute
        return new ValidationResult($"The {validationContext.DisplayName} field must be a DateTime or DateTimeOffset.");
    }
}
```

#### How to Use It

```csharp
// DTOs/EventDto.cs
using System.ComponentModel.DataAnnotations;

public class CreateEventDto
{
    [Required(ErrorMessage = "Event name is required.")]
    [StringLength(100, MinimumLength = 3, ErrorMessage = "Event name must be between 3 and 100 characters.")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Event date is required.")]
    [FutureDate(ErrorMessage = "The event date must be in the future.")] // Using our custom attribute
    public DateTime EventDate { get; set; }

    [StringLength(500, ErrorMessage = "Description cannot exceed 500 characters.")]
    public string Description { get; set; }
}
```

#### Example 2: Class-Level Validation (Cross-Property Validation)

This attribute will check if `EndDate` is after `StartDate`. This requires the attribute to be applied at the class level, and the `IsValid` method will receive the entire object.

```csharp
// Attributes/DateRangeAttribute.cs
using System.ComponentModel.DataAnnotations;

[AttributeUsage(AttributeTargets.Class, AllowMultiple = false)] // Apply to class, only once
public class DateRangeAttribute : ValidationAttribute
{
    private readonly string _startDatePropertyName;
    private readonly string _endDatePropertyName;

    public DateRangeAttribute(string startDatePropertyName, string endDatePropertyName)
    {
        _startDatePropertyName = startDatePropertyName;
        _endDatePropertyName = endDatePropertyName;
        ErrorMessage = "The end date must be after the start date.";
    }

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (value == null)
        {
            return ValidationResult.Success; // Let [Required] handle null objects
        }

        // Get the properties using reflection
        var startDateProperty = validationContext.ObjectType.GetProperty(_startDatePropertyName);
        var endDateProperty = validationContext.ObjectType.GetProperty(_endDatePropertyName);

        if (startDateProperty == null || endDateProperty == null)
        {
            throw new ArgumentException($"Properties '{_startDatePropertyName}' or '{_endDatePropertyName}' not found on type '{validationContext.ObjectType.Name}'.");
        }

        var startDate = startDateProperty.GetValue(value) as DateTime?;
        var endDate = endDateProperty.GetValue(value) as DateTime?;

        // If either date is null, let [Required] handle it or assume valid for this check
        if (!startDate.HasValue || !endDate.HasValue)
        {
            return ValidationResult.Success;
        }

        if (endDate.Value > startDate.Value)
        {
            return ValidationResult.Success;
        }
        else
        {
            return new ValidationResult(ErrorMessage, new[] { _startDatePropertyName, _endDatePropertyName });
        }
    }
}
```

#### How to Use It

```csharp
// DTOs/BookingDto.cs
using System.ComponentModel.DataAnnotations;

[DateRange("StartDate", "EndDate", ErrorMessage = "Booking end date must be after the start date.")]
public class CreateBookingDto
{
    [Required]
    public Guid CustomerId { get; set; }

    [Required]
    [FutureDate(ErrorMessage = "Start date must be in the future.")]
    public DateTime StartDate { get; set; }

    [Required]
    [FutureDate(ErrorMessage = "End date must be in the future.")] // Still good to have property-level validation
    public DateTime EndDate { get; set; }

    [Range(1, 10, ErrorMessage = "Number of guests must be between 1 and 10.")]
    public int NumberOfGuests { get; set; }
}
```

### Integration with ASP.NET Core

When you use these DTOs as parameters in your ASP.NET Core controller actions, the framework automatically handles the validation:

```csharp
// Controllers/EventsController.cs
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class EventsController : ControllerBase
{
    // private readonly IEventService _eventService;

    // public EventsController(IEventService eventService)
    // {
    //     _eventService = eventService;
    // }

    [HttpPost]
    public IActionResult CreateEvent([FromBody] CreateEventDto createEventDto)
    {
        if (!ModelState.IsValid) // This check is often implicit with [ApiController]
        {
            return BadRequest(ModelState); // Returns 400 with validation errors
        }

        // If ModelState is valid, proceed with business logic
        // var eventId = await _eventService.CreateEventAsync(createEventDto);
        // return CreatedAtAction(nameof(GetEvent), new { id = eventId }, createEventDto);
        return Ok("Event created successfully (mock).");
    }

    [HttpPost("booking")]
    public IActionResult CreateBooking([FromBody] CreateBookingDto createBookingDto)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        return Ok("Booking created successfully (mock).");
    }
}
```

With `[ApiController]` attribute on your controller, ASP.NET Core automatically performs `ModelState.IsValid` check and returns a `BadRequest` response with validation errors if validation fails. You often don't need to write `if (!ModelState.IsValid)` explicitly.

### Production-Level Considerations

1.  **Where to Validate**:
    *   **API Input (DTOs):** This is the primary place for custom validation attributes. They ensure that incoming data conforms to basic structural and business rules *before* it even hits your service layer. This provides immediate feedback to clients.
    *   **Service Layer:** For more complex business rules that might involve database lookups, interactions with other services, or complex state changes, the service layer is the appropriate place. While you *could* inject services into validation attributes, it often leads to tightly coupled and harder-to-test attributes.
    *   **Domain Models:** If you have rich domain models, they can contain their own internal validation logic (e.g., in constructors or methods) to ensure they are always in a valid state.

2.  **Error Messages**: Make error messages clear, concise, and helpful for API consumers. Consider localization if your API serves multiple languages.

3.  **Performance**: Validation attributes use reflection, which has a minor performance overhead. For most applications, this is negligible. For extremely high-performance scenarios with thousands of validations per second, you might consider pre-compiled validators (like those generated by FluentValidation).

4.  **Testability**: Custom validation attributes are easy to unit test in isolation.

5.  **Alternative: FluentValidation**: For very complex validation scenarios, especially those involving conditional logic, complex object graphs, or dependency injection, a library like **FluentValidation** is often preferred over custom `ValidationAttribute`s.
    *   **Pros of FluentValidation**: More expressive, easier to test, supports DI out-of-the-box, better for complex rules, can be integrated with ASP.NET Core's model validation.
    *   **Cons of FluentValidation**: Adds another dependency and a slightly different syntax.
    *   **When to use Custom Attributes**: Simple, declarative, property-level or simple cross-property rules.
    *   **When to use FluentValidation**: Complex, conditional, or service-dependent validation.

6.  **Security (Over-posting)**: Validation attributes primarily focus on data *validity*, not *authorization*. Ensure your DTOs don't expose sensitive properties that a malicious user could try to set (e.g., `IsAdmin = true`). This is where mapping (as discussed with Mapster) and careful DTO design come into play.

### Senior Insight

As a senior developer, I approach validation with a layered strategy:

1.  **"Fail Fast" at the Edge:** The first line of defense is always at the API input (DTOs). Use `ValidationAttribute`s (both built-in and custom) to catch obvious errors as early as possible. This saves processing power and provides immediate feedback to the client. This is where `[Required]`, `[StringLength]`, `[FutureDate]` etc., shine.
2.  **Business Logic Validation in Services:** Any validation that requires querying the database, interacting with other services, or involves complex, multi-step business rules belongs in the service layer. For example, "Is this email already registered?" or "Can this user afford this purchase?" These are not suitable for `ValidationAttribute`s because attributes should ideally be stateless and self-contained.
3.  **Domain Invariants in Domain Models:** If you're using Domain-Driven Design, your domain entities should enforce their own invariants. For example, an `Order` entity's `AddItem` method might throw an exception if the product is out of stock. This ensures that a domain object is always in a valid state once created or modified.
4.  **Choose the Right Tool:** For simple, declarative rules, custom `ValidationAttribute`s are perfectly fine and integrate seamlessly with ASP.NET Core. For anything more complex, especially cross-cutting concerns or rules that depend on external state, **FluentValidation is almost always the superior choice**. It offers a much more flexible and testable API for complex validation logic. I often use a combination: simple `[Required]` and `[StringLength]` on DTOs, and then FluentValidation for more intricate DTO-level rules.
5.  **Clear Error Responses:** Ensure your API returns consistent and informative error responses when validation fails. ASP.NET Core's default `BadRequest(ModelState)` is a good start, but you might want to standardize the error payload for your API.

Custom validation attributes are a powerful tool in your .NET backend toolkit. They allow you to extend the framework's built-in validation capabilities to enforce your unique business rules declaratively, leading to cleaner, more robust, and more maintainable code.

What's next on our journey?