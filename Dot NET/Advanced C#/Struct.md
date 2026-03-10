### What is a `struct`? The Core Distinction

At its heart, a `struct` (short for structure) in C# is a **value type**. This is the single most crucial distinction you must internalize. Unlike `classes`, which are **reference types**, structs behave differently in terms of memory allocation, assignment, and how they are passed around.

Think of it this way:
*   **Reference Types (Classes)**: When you create an instance of a class, you're creating an object on the **heap**, and the variable you use to refer to it actually holds a *reference* (an address) to that object. If you assign one class variable to another, both variables will point to the *same* object in memory.
*   **Value Types (Structs)**: When you create an instance of a struct, the actual data of the struct is stored directly where the variable is declared. If you assign one struct variable to another, a *copy* of the entire struct's data is made. They are completely independent entities thereafter.

### Key Characteristics and Differences from Classes

Let's break down the implications of being a value type:

1.  **Memory Allocation**:
    *   **Structs**: Typically allocated on the **stack** when declared as local variables or method parameters. When a struct is a field within a class or another struct, it's embedded directly within that containing type's memory, whether that's on the heap or stack. This can lead to performance benefits by reducing garbage collection pressure.
    *   **Classes**: Always allocated on the **heap**. The garbage collector manages their lifetime.

2.  **Assignment and Passing**:
    *   **Structs**: Assignment creates a **copy** of the data. Passing a struct to a method also passes a copy. This means changes made to the struct within the method do not affect the original struct outside the method, unless you explicitly use `ref` or `out` keywords.
    *   **Classes**: Assignment copies the **reference**. Passing a class instance to a method passes a copy of the reference, meaning both the original variable and the method parameter point to the *same* object. Changes made within the method *will* affect the original object.

3.  **Inheritance**:
    *   **Structs**: Cannot inherit from other structs or classes, and cannot be inherited from. They implicitly inherit from `System.ValueType`, which itself inherits from `System.Object`. This means they can override methods like `ToString()`, `Equals()`, and `GetHashCode()`.
    *   **Classes**: Support single inheritance and can implement multiple interfaces.

4.  **Default Constructors**:
    *   **Structs (C# 9 and earlier)**: You could not declare an explicit parameterless constructor. The compiler always provided a default one that initialized all fields to their default values (0 for numeric types, `null` for reference types, etc.).
    *   **Structs (C# 10 and later)**: You *can* declare an explicit parameterless constructor. This is a significant improvement, allowing for more control over initialization.

5.  **Immutability**:
    *   While not strictly enforced, structs are often designed to be **immutable**. This means once a struct is created, its internal state cannot be changed. Immutability simplifies reasoning about code, especially in concurrent scenarios, and aligns well with their copy-by-value semantics. C# provides features like `readonly struct` and `record struct` to encourage and enforce this.

6.  **Boxing and Unboxing**:
    *   When a struct is treated as an `object` or an interface type it implements, it undergoes **boxing**. This means a new object is allocated on the heap, and the struct's value is copied into it.
    *   When a boxed struct is cast back to its original struct type, it undergoes **unboxing**, where the value is copied from the heap object back to a stack location.
    *   Both boxing and unboxing incur performance overhead and heap allocations, which can negate the performance benefits of structs if done excessively.

### When to Use Structs (Guidelines)

The general guidance for using structs is often summarized by these points:

*   **Small Size**: Structs are best for small, lightweight data types that represent a single value or a small group of related values (e.g., a point in 2D space, a color, a monetary amount).
*   **Immutability**: They should ideally be immutable or have very limited mutable state.
*   **Value Semantics**: When you want copy-by-value behavior, where each variable holds its own independent copy of the data.
*   **Performance Critical Scenarios**: When you need to minimize heap allocations and garbage collection overhead, especially in tight loops or high-performance code, and the struct is small enough that copying it is cheaper than managing references.

Avoid structs for large data structures, mutable types, or when you need reference semantics (e.g., polymorphism, shared state).

### C# 10+ Enhancements for Structs

The C# language has evolved to make structs even more powerful and easier to work with, particularly in C# 10 and beyond.

#### 1. Parameterless Constructors

As mentioned, C# 10 allows you to define your own parameterless constructors for structs. This gives you explicit control over the default state of a struct instance.

**Example (C# 10+):**

```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }

    // Explicit parameterless constructor
    public Point()
    {
        X = 0; // Initialize X to 0
        Y = 0; // Initialize Y to 0
        Console.WriteLine("Point() constructor called.");
    }

    public Point(int x, int y)
    {
        X = x;
        Y = y;
        Console.WriteLine($"Point({x}, {y}) constructor called.");
    }

    public override string ToString() => $"({X}, {Y})";
}

// Usage:
Point p1 = new Point(); // Calls the parameterless constructor
Console.WriteLine($"p1: {p1}"); // Output: p1: (0, 0)

Point p2 = new Point(10, 20); // Calls the parameterized constructor
Console.WriteLine($"p2: {p2}"); // Output: p2: (10, 20)
```

#### 2. `record struct`

Introduced in C# 10, `record struct` combines the benefits of structs (value types) with the features of records (concise syntax for data-centric types, built-in value equality, and non-destructive mutation via `with` expressions).

`record struct` can be declared as `record struct` (mutable by default) or `readonly record struct` (immutable by default, highly recommended).

**Key Features of `record struct`:**

*   **Positional Parameters**: Allows for concise declaration of properties in the constructor.
*   **Value Equality**: The `Equals` method is automatically generated to compare all public properties/fields by value.
*   **`ToString()` Override**: Automatically generates a `ToString()` method that prints the type name and all public properties/fields.
*   **`with` Expressions**: Enables non-destructive mutation, creating a new instance with specified properties changed, leaving the original untouched.

**Example (`readonly record struct` - C# 10+):**

Let's define a `Money` type. It's a perfect candidate for a `readonly record struct` because money amounts should be immutable and compared by value.

```csharp
public readonly record struct Money(decimal Amount, string Currency)
{
    // You can add custom methods or properties
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
        {
            throw new InvalidOperationException("Cannot add money of different currencies.");
        }
        return this with { Amount = Amount + other.Amount }; // Using 'with' expression
    }

    // You can also define an explicit constructor if needed,
    // but the primary constructor handles most cases.
    public Money() : this(0m, "USD") { } // Parameterless constructor for record struct
}

// Usage:
Money cash1 = new Money(100.50m, "USD");
Money cash2 = new Money(100.50m, "USD");
Money cash3 = new Money(200.00m, "EUR");

Console.WriteLine($"cash1: {cash1}"); // Output: Money { Amount = 100.50, Currency = USD }
Console.WriteLine($"cash2: {cash2}"); // Output: Money { Amount = 100.50, Currency = USD }
Console.WriteLine($"cash3: {cash3}"); // Output: Money { Amount = 200.00, Currency = EUR }

// Value equality
Console.WriteLine($"cash1 == cash2: {cash1 == cash2}"); // Output: True (due to value equality)
Console.WriteLine($"cash1 == cash3: {cash1 == cash3}"); // Output: False

// Non-destructive mutation using 'with' expression
Money cash4 = cash1 with { Amount = 150.00m };
Console.WriteLine($"cash4: {cash4}"); // Output: Money { Amount = 150.00, Currency = USD }
Console.WriteLine($"cash1 (original): {cash1}"); // Output: Money { Amount = 100.50, Currency = USD } (unchanged)

// Using the custom Add method
Money totalCash = cash1.Add(new Money(25.00m, "USD"));
Console.WriteLine($"totalCash: {totalCash}"); // Output: Money { Amount = 125.50, Currency = USD }

// Using the parameterless constructor
Money defaultCash = new Money();
Console.WriteLine($"defaultCash: {defaultCash}"); // Output: Money { Amount = 0.00, Currency = USD }
```

#### 3. `with` Expressions for Non-Record Structs (C# 10+)

Even for regular `struct` types (not `record struct`), C# 10 allows you to use `with` expressions, provided the struct has public fields or properties. This is particularly useful for creating new instances with modified values from an existing instance, promoting a more functional, immutable style even with traditional structs.

**Example (C# 10+):**

Let's revisit our `Point` struct, making it `readonly` to encourage immutability.

```csharp
public readonly struct Point
{
    public int X { get; init; } // 'init' accessor for immutable properties
    public int Y { get; init; }

    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }

    public override string ToString() => $"({X}, {Y})";
}

// Usage:
Point origin = new Point(0, 0);
Console.WriteLine($"Origin: {origin}"); // Output: Origin: (0, 0)

// Create a new point based on 'origin' but with a different X value
Point movedPoint = origin with { X = 10 };
Console.WriteLine($"Moved Point: {movedPoint}"); // Output: Moved Point: (10, 0)
Console.WriteLine($"Origin (original): {origin}"); // Output: Origin (original): (0, 0) (unchanged)
```
Notice the use of `init` accessors. These allow properties to be set only during object initialization (either in the constructor or via an object initializer), making the struct effectively immutable after construction.

### Conclusion

Structs are a powerful feature in C#, offering distinct advantages in specific scenarios, primarily when dealing with small, immutable, value-based data types where performance and memory allocation are critical. The enhancements in C# 10, particularly `record struct` and the ability to define parameterless constructors and use `with` expressions, have significantly improved their usability and made it easier to write robust, immutable value types.

As a discerning engineer, your choice between `class` and `struct` should be a deliberate one, guided by the semantics you require (reference vs. value), the size and mutability of your data, and the performance characteristics of your application. Don't shy away from structs; understand them, and wield them effectively.

Any questions?