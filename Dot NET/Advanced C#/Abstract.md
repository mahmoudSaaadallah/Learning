### Understanding the `abstract` Keyword in C Sharp

In C#, the `abstract` keyword is a powerful tool that allows us to define classes and class members (methods, properties, indexers, and events) that are incomplete and must be implemented by a derived class. Think of it as laying down a blueprint or a contract that concrete implementations must adhere to.

#### 1. Abstract Classes

An `abstract` class is a class that cannot be instantiated directly. Its primary purpose is to serve as a base class for other classes. It often contains `abstract` members, but it can also contain concrete (non-abstract) members.

**Key Characteristics of Abstract Classes:**
*   **Cannot be instantiated:** You cannot create an object of an `abstract` class using the `new` keyword.
*   **Must be inherited:** To use an `abstract` class, you must derive a concrete class from it.
*   **Can contain `abstract` members:** These are members (methods, properties, etc.) that have no implementation in the `abstract` class itself.
*   **Can contain non-abstract (concrete) members:** `abstract` classes can have fields, constructors, and methods with full implementations, just like a regular class.
*   **Cannot be `sealed` [[Sealed]]:** An `abstract` class cannot be `sealed` because `sealed` prevents inheritance, which contradicts the purpose of an `abstract` class.

#### 2. Abstract Methods

An `abstract` method is a method declared within an `abstract` class that has no implementation (no method body). It only has a signature.

**Key Characteristics of Abstract Methods:**
*   **No implementation:** An `abstract` method declaration ends with a semicolon, not a method body.
*   **Must be overridden:** Any non-abstract class that inherits from an `abstract` class *must* provide an implementation for all inherited `abstract` methods using the `override` keyword.
*   **Implicitly `virtual`:** `abstract` methods are implicitly `virtual`, meaning they can be overridden in derived classes. You cannot explicitly use `virtual` with `abstract`.
*   **Only in `abstract` classes:** An `abstract` method can only be declared within an `abstract` class.

### Why Use `abstract` Classes and Methods?

The primary motivations for using `abstract` types are:

1.  **Enforcing a Contract:** They define a common interface [[Interface]] or a set of behaviors that all derived classes *must* implement. This ensures consistency across related types.
2.  **Code Reusability:** You can provide common, concrete implementations for certain behaviors in the `abstract` base class, preventing code duplication in derived classes.
3.  **Polymorphism:** `abstract` classes are excellent for achieving polymorphism, allowing you to treat objects of different derived types uniformly through a common base type.
4.  **Preventing Incomplete Objects:** By making a class `abstract`, you prevent developers from instantiating an object that is inherently incomplete or doesn't make sense on its own.

### Detailed Examples using C# (C# 12 Compatible)

Let's illustrate these concepts with some practical C# examples.

#### Example 1: Basic Abstract Class and Method

Imagine we're building a system for various geometric shapes. All shapes have an area, but the calculation differs for each.

```csharp
// Define an abstract base class for all shapes
public abstract class Shape
{
    // An abstract method for calculating area.
    // All derived concrete shapes MUST implement this.
    public abstract double CalculateArea();

    // A concrete method that all shapes can use.
    public void DisplayShapeInfo()
    {
        Console.WriteLine($"This is a {this.GetType().Name}.");
    }
}

// A concrete class representing a Circle
public class Circle : Shape
{
    public double Radius { get; set; }

    public Circle(double radius)
    {
        Radius = radius;
    }

    // Override the abstract method to provide specific implementation for Circle
    public override double CalculateArea()
    {
        return Math.PI * Radius * Radius;
    }
}

// A concrete class representing a Rectangle
public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }

    // Override the abstract method to provide specific implementation for Rectangle
    public override double CalculateArea()
    {
        return Width * Height;
    }
}

public class Program
{
    public static void Main(string[] args)
    {
        // You cannot instantiate an abstract class directly:
        // Shape myShape = new Shape(); // This would cause a compile-time error!

        Circle circle = new Circle(5);
        Rectangle rectangle = new Rectangle(4, 6);

        Console.WriteLine($"Circle Area: {circle.CalculateArea():F2}");
        circle.DisplayShapeInfo(); // Using the concrete method from the abstract base class

        Console.WriteLine($"Rectangle Area: {rectangle.CalculateArea():F2}");
        rectangle.DisplayShapeInfo(); // Using the concrete method from the abstract base class

        Console.WriteLine("\n--- Demonstrating Polymorphism ---");
        // We can treat derived classes as their abstract base type
        List<Shape> shapes = new List<Shape>
        {
            new Circle(3),
            new Rectangle(2, 5),
            new Circle(7.5)
        };

        foreach (Shape s in shapes)
        {
            Console.WriteLine($"{s.GetType().Name} Area: {s.CalculateArea():F2}");
        }
    }
}
```

**Output of Example 1:**
```
Circle Area: 78.54
This is a Circle.
Rectangle Area: 24.00
This is a Rectangle.

--- Demonstrating Polymorphism ---
Circle Area: 28.27
Rectangle Area: 10.00
Circle Area: 176.71
```

In this example, `Shape` is an `abstract` class. It defines a contract: any `Shape` *must* know how to `CalculateArea()`. However, it doesn't provide the implementation because the calculation varies. `Circle` and `Rectangle` are concrete implementations that fulfill this contract. Notice how `DisplayShapeInfo()` is a concrete method in `Shape`, demonstrating code reuse.

#### Example 2: Abstract Properties

`abstract` properties work similarly to `abstract` methods. They declare a property without providing an implementation for its `get` or `set` accessors.

```csharp
public abstract class Employee
{
    // Abstract property: derived classes must implement Name
    public abstract string Name { get; set; }

    // Concrete property with default implementation
    public string Department { get; set; } = "Unassigned";

    // Abstract method: derived classes must implement CalculateSalary
    public abstract decimal CalculateSalary();

    public void PrintDetails()
    {
        Console.WriteLine($"Employee: {Name}, Department: {Department}");
    }
}

public class FullTimeEmployee : Employee
{
    private string _name; // Backing field for the abstract property
    public override string Name
    {
        get { return _name; }
        set { _name = value; }
    }

    public decimal MonthlySalary { get; set; }

    public FullTimeEmployee(string name, decimal monthlySalary)
    {
        Name = name;
        MonthlySalary = monthlySalary;
    }

    public override decimal CalculateSalary()
    {
        return MonthlySalary;
    }
}

public class Contractor : Employee
{
    private string _name;
    public override string Name
    {
        get { return _name; }
        set { _name = value; }
    }

    public decimal HourlyRate { get; set; }
    public int HoursWorked { get; set; }

    public Contractor(string name, decimal hourlyRate, int hoursWorked)
    {
        Name = name;
        HourlyRate = hourlyRate;
        HoursWorked = hoursWorked;
    }

    public override decimal CalculateSalary()
    {
        return HourlyRate * HoursWorked;
    }
}

public class Payroll
{
    public static void Main(string[] args)
    {
        FullTimeEmployee ftEmployee = new FullTimeEmployee("Alice Smith", 5000m);
        ftEmployee.Department = "Engineering";

        Contractor contractor = new Contractor("Bob Johnson", 75m, 160);
        contractor.Department = "IT Support";

        List<Employee> employees = new List<Employee>
        {
            ftEmployee,
            contractor
        };

        foreach (Employee emp in employees)
        {
            emp.PrintDetails();
            Console.WriteLine($"Calculated Salary: {emp.CalculateSalary():C}");
            Console.WriteLine("--------------------");
        }
    }
}
```

**Output of Example 2:**
```
Employee: Alice Smith, Department: Engineering
Calculated Salary: $5,000.00
--------------------
Employee: Bob Johnson, Department: IT Support
Calculated Salary: $12,000.00
--------------------
```

Here, `Employee` is an `abstract` class that mandates a `Name` property and a `CalculateSalary` method. `FullTimeEmployee` and `Contractor` provide their specific implementations for these `abstract` members, while sharing the `Department` property and `PrintDetails` method from the base class.

### Abstract Classes vs. Interfaces (A Brief Distinction)

While both `abstract` classes and `interfaces` define contracts, they serve slightly different purposes:

*   **Abstract Classes:** Can have both `abstract` (unimplemented) and concrete (implemented) members. They can also have fields and constructors. A class can inherit from only one `abstract` class. Best for "is-a" relationships where there's a strong common base implementation or state.
*   **Interfaces:** Can only declare members (methods, properties, events, indexers) without implementation (prior to C# 8 default interface methods). They cannot have fields or constructors. A class can implement multiple interfaces. Best for "can-do" relationships, defining a capability or behavior.

