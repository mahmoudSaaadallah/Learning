### The Motivation: The Boilerplate Problem

Before `record` types, creating a simple, immutable(Read Only) data class in C# often involved a lot of boilerplate code. Imagine you wanted a `Person` class with `FirstName` and `LastName`. To make it truly immutable and behave like a "value" (where two people are considered equal if their names are the same, regardless of memory location), you'd typically write something like this:

```csharp
public class Person
{
    public string FirstName { get; }
    public string LastName { get; }

    public Person(string firstName, string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }

    // Override Equals for value-based equality
    public override bool Equals(object obj)
    {
        return obj is Person person &&
               FirstName == person.FirstName &&
               LastName == person.LastName;
    }

    // Override GetHashCode to match Equals behavior
    public override int GetHashCode()
    {
        return HashCode.Combine(FirstName, LastName);
    }

    // Override ToString for meaningful output
    public override string ToString()
    {
        return $"Person {{ FirstName = {FirstName}, LastName = {LastName} }}";
    }

    // And if you wanted non-destructive mutation (a 'with' expression),
    // you'd have to write a method for that too.
}
```

That's a lot of code for a simple data holder! The `record` keyword was introduced to drastically reduce this boilerplate, providing built-in functionality for value-based equality, concise property declaration, and non-destructive mutation.

---

### What is a `record`?

At its core, a `record` is a special kind of reference type (like a `class`) that provides *value-based equality* and other features out-of-the-box. While it's a reference type, it's designed to behave more like a value type in terms of comparison.

The primary goal of `record` types is to simplify the creation of immutable data models, often referred to as Data Transfer Objects (DTOs) or Value Objects.

---

### Key Features and Examples

Let's break down the powerful features that `record` types provide.

#### 1. Positional Records: Concise Declaration

The most striking feature is the ability to declare properties and a primary constructor in a single line.

```csharp
// C# 9+
public record Person(string FirstName, string LastName);

// This single line generates:
// - A public class named Person
// - Public, read-only (init-only) properties: FirstName and LastName
// - A public primary constructor that takes FirstName and LastName
// - Overrides for Equals, GetHashCode, and ToString
// - Support for 'with' expressions
// - Support for deconstruction
```

**Usage:**

```csharp
public class Program
{
    public static void Main()
    {
        // Instantiation
        Person person1 = new Person("Alice", "Smith");
        Person person2 = new Person("Bob", "Johnson");

        Console.WriteLine($"Person 1: {person1}"); // Uses auto-generated ToString
        Console.WriteLine($"Person 2: {person2}");
    }
}
```

#### 2. Value-Based Equality

This is a cornerstone of `record` types. Unlike `class` types, where `==` and `Equals()` by default check for *reference equality* (do they point to the same object in memory?), `record` types automatically implement `Equals()` and `GetHashCode()` to provide *value-based equality*. This means two records are considered equal if all their public property values are equal.

```csharp
public class Program
{
    public static void Main()
    {
        Person person1 = new Person("Alice", "Smith");
        Person person2 = new Person("Bob", "Johnson");
        Person person3 = new Person("Alice", "Smith"); // Same values as person1

        Console.WriteLine($"person1 == person2: {person1 == person2}"); // False
        Console.WriteLine($"person1 == person3: {person1 == person3}"); // True (value-based equality)

        Console.WriteLine($"person1.Equals(person3): {person1.Equals(person3)}"); // True
    }
}
```

#### 3. Automatic `ToString()` Implementation

`record` types automatically generate a `ToString()` method that outputs the type name and the names and values of all public properties. This is incredibly useful for debugging and logging.

```csharp
public class Program
{
    public static void Main()
    {
        Person person1 = new Person("Alice", "Smith");
        Console.WriteLine(person1); // Output: Person { FirstName = Alice, LastName = Smith }
    }
}
```

#### 4. Non-Destructive Mutation (`with` expressions)

Often, you need to create a *new* instance of a data type that is identical to an existing one, but with one or more properties changed. This is where `with` expressions shine. They allow you to create a copy of a record instance, applying modifications to specified properties, without altering the original object.

```csharp
public class Program
{
    public static void Main()
    {
        Person originalPerson = new Person("Alice", "Smith");
        Console.WriteLine($"Original: {originalPerson}");

        // Create a new record with a modified LastName
        Person updatedPerson = originalPerson with { LastName = "Johnson" };
        Console.WriteLine($"Updated: {updatedPerson}");

        Console.WriteLine($"Original (unchanged): {originalPerson}"); // Original is immutable
    }
}
```

This is a powerful pattern for immutable data, as it avoids side effects and makes your code easier to reason about.

#### 5. Deconstruction

Records also automatically support deconstruction, allowing you to extract property values into separate variables.

```csharp
public class Program
{
    public static void Main()
    {
        Person person = new Person("Alice", "Smith");

        // Deconstruct the record
        var (firstName, lastName) = person;
        Console.WriteLine($"Deconstructed: {firstName} {lastName}");
    }
}
```

#### 6. Inheritance

Records can inherit from other records. However, a record cannot inherit from a class, and a class cannot inherit from a record.

```csharp
public record Vehicle(string Manufacturer, string Model);
public record Car(string Manufacturer, string Model, int NumberOfDoors) : Vehicle(Manufacturer, Model);

public class Program
{
    public static void Main()
    {
        Car myCar = new Car("Toyota", "Camry", 4);
        Console.WriteLine(myCar); // Output: Car { Manufacturer = Toyota, Model = Camry, NumberOfDoors = 4 }
    }
}
```

#### 7. Record Structs (C# 10+)

While `record` by itself creates a reference type (`record class`), C# 10 introduced `record struct`. This allows you to define value-based equality and other record features for value types.

```csharp
public record struct Point(int X, int Y);

public class Program
{
    public static void Main()
    {
        Point p1 = new Point(10, 20);
        Point p2 = new Point(10, 20);
        Point p3 = new Point(30, 40);

        Console.WriteLine($"p1 == p2: {p1 == p2}"); // True (value-based equality for structs)
        Console.WriteLine($"p1 == p3: {p1 == p3}"); // False
    }
}
```

**`record class` vs. `record struct`:**
*   **`record class` (default `record`):** Reference type. Stored on the heap. Value-based equality. Best for larger data models where copying might be expensive, or when you need polymorphism.
*   **`record struct`:** Value type. Stored on the stack (or inline in other types). Value-based equality. Best for small, immutable data where copying is cheap and you want value semantics (like `Point` or `Money`).

---

### When to Use `record`

*   **Data Transfer Objects (DTOs):** When you need simple objects to carry data between layers of an application.
*   **Immutable Data Models:** When you want to ensure that once an object is created, its state cannot be changed. This simplifies concurrency and reasoning about your code.
*   **Value Objects:** When the identity of an object is determined by its values, not its memory location (e.g., a `Color` object, a `Money` amount, a `Coordinate`).
*   **Functional Programming Paradigms:** Records fit well into functional styles where immutability is preferred.

---

### When NOT to Use `record`

*   **When Reference Equality is Important:** If you need to distinguish between two objects that have the same property values but are distinct entities (e.g., two different `Customer` objects with the same name but different IDs, where the ID is the true identifier). In such cases, a traditional `class` is more appropriate.
*   **When Complex Behavior is Primary:** While records can have methods, if the primary purpose of your type is to encapsulate complex behavior and state changes, a `class` is generally a better fit. Records are optimized for data.
*   **When Mutable State is Desired:** If your object is designed to have its state change frequently after creation, a `class` with mutable properties is more suitable.
