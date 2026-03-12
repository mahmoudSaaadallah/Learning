As we discussed with interfaces[[Interface]], polymorphism is a powerful concept. Virtual functions are the mechanism through which **runtime polymorphism** (also known as dynamic dispatch) is achieved in C# when working with inheritance.

### What is a Virtual Function?

In C#, a **virtual function** (or more accurately, a virtual method or property) is a member of a base class that can be **overridden** by a derived class. When you call a virtual method on an object through a reference to its base class, the runtime determines which version of the method (the base class's or a derived class's override) to execute based on the *actual type* of the object, not the type of the reference.

**Key Concepts:**

1.  **`virtual` keyword:** This keyword is used in the base class to declare a method, property, event, or indexer as virtual. It signals that derived classes are permitted to provide their own implementation of this member.
2.  **`override` keyword:** This keyword is used in a derived class to provide a new implementation for a virtual member inherited from a base class. It explicitly states that this method is replacing the base class's virtual method.
3.  **`new` keyword (for hiding):** It's important to distinguish `override` from `new`. If a derived class defines a method with the same signature as a base class method *without* using `override` (and the base method isn't virtual), or if it uses `new`, it **hides** the base class method. This is *not* polymorphism; the method called depends on the *type of the reference*, not the actual object type. We want `override` for polymorphism.
4.  **`sealed` keyword:** A virtual override can be declared `sealed` in a derived class. This prevents any further derived classes from overriding that specific method.

### Why do we use Virtual Functions?

*   **Runtime Polymorphism:** [[Polymorphism]] This is the primary reason. It allows you to write generic code that operates on base class references, but the specific behavior executed is determined by the actual derived type at runtime.
*   **Extensibility:** Base classes can define a common interface for a set of operations, allowing derived classes to customize or specialize those operations without altering the base class's structure.
*   **Template Method Pattern:** Virtual methods are fundamental to implementing design patterns like the Template Method, where a base class defines the skeleton of an algorithm, and derived classes implement specific steps.

### Example: Animals and Their Sounds

Let's illustrate this with a classic example: animals making sounds.

```csharp
using System;
using System.Collections.Generic; // For List<Animal>

// Base Class
public class Animal
{
    public string Name { get; set; }

    public Animal(string name)
    {
        Name = name;
    }

    // This is a virtual method. Derived classes can override it.
    public virtual void MakeSound()
    {
        Console.WriteLine($"{Name} makes a generic animal sound.");
    }

    // A non-virtual method - cannot be overridden, only hidden with 'new'
    public void Eat()
    {
        Console.WriteLine($"{Name} is eating.");
    }
}

// Derived Class 1
public class Dog : Animal
{
    public Dog(string name) : base(name) { }

    // Override the virtual MakeSound method
    public override void MakeSound()
    {
        Console.WriteLine($"{Name} barks: Woof! Woof!");
    }

    // Example of a method specific to Dog
    public void Fetch()
    {
        Console.WriteLine($"{Name} fetches the ball.");
    }
}

// Derived Class 2
public class Cat : Animal
{
    public Cat(string name) : base(name) { }

    // Override the virtual MakeSound method
    public override void MakeSound()
    {
        Console.WriteLine($"{Name} meows: Meow!");
    }
}

// Derived Class 3 - Demonstrating 'sealed'
public class Puppy : Dog
{
    public Puppy(string name) : base(name) { }

    // We can override MakeSound again, but let's say we want to seal it here
    public sealed override void MakeSound()
    {
        Console.WriteLine($"{Name} whimpers: Yip! Yip!");
    }
}

// If we tried to create a class like this, it would cause a compile-time error:
// public class Chihuahua : Puppy
// {
//     public override void MakeSound() // Error: Cannot override sealed member 'Puppy.MakeSound()'
//     {
//         Console.WriteLine("Chihuahua yaps!");
//     }
// }

public class Program
{
    public static void Main(string[] args)
    {
        Console.WriteLine("--- Demonstrating Virtual Methods ---");

        Animal genericAnimal = new Animal("Generic");
        Dog myDog = new Dog("Buddy");
        Cat myCat = new Cat("Whiskers");
        Puppy myPuppy = new Puppy("Sparky");

        genericAnimal.MakeSound(); // Output: Generic makes a generic animal sound.
        myDog.MakeSound();        // Output: Buddy barks: Woof! Woof!
        myCat.MakeSound();        // Output: Whiskers meows: Meow!
        myPuppy.MakeSound();      // Output: Sparky whimpers: Yip! Yip!

        Console.WriteLine("\n--- Demonstrating Polymorphism with Base Class References ---");

        // We can store all these different animal types in a list of the base type 'Animal'
        List<Animal> zoo = new List<Animal>
        {
            new Dog("Max"),
            new Cat("Luna"),
            new Animal("Unknown Creature"),
            new Puppy("Daisy")
        };

        foreach (Animal animal in zoo)
        {
            Console.Write($"{animal.Name}: ");
            animal.MakeSound(); // The correct MakeSound() for each actual type is called at runtime.
            animal.Eat();       // Non-virtual method always calls the base implementation (or hidden if 'new' was used)
        }

        Console.WriteLine("\n--- Difference between 'override' and 'new' (Hiding) ---");

        // Let's imagine a scenario where a derived class 'hides' a base method
        // (This is generally discouraged for polymorphic behavior)
        class Bird : Animal
        {
            public Bird(string name) : base(name) { }

            // This 'hides' the base MakeSound method, it does NOT override it.
            // The compiler will warn you about this, suggesting 'new' keyword.
            public new void MakeSound()
            {
                Console.WriteLine($"{Name} chirps: Tweet! Tweet!");
            }
        }

        Bird myBird = new Bird("Tweety");
        myBird.MakeSound(); // Output: Tweety chirps: Tweet! Tweet! (Called directly on Bird type)

        Animal hiddenAnimal = new Bird("Robin");
        hiddenAnimal.MakeSound(); // Output: Robin makes a generic animal sound.
                                  // Because the reference type is Animal, and Bird's MakeSound
                                  // is not an override, the base Animal.MakeSound is called.
                                  // This is why 'override' is crucial for polymorphism.
    }
}
```

**Output of the `Main` method:**

```
--- Demonstrating Virtual Methods ---
Generic makes a generic animal sound.
Buddy barks: Woof! Woof!
Whiskers meows: Meow!
Sparky whimpers: Yip! Yip!

--- Demonstrating Polymorphism with Base Class References ---
Max: Max barks: Woof! Woof!
Max is eating.
Luna: Luna meows: Meow!
Luna is eating.
Unknown Creature: Unknown Creature makes a generic animal sound.
Unknown Creature is eating.
Daisy: Daisy whimpers: Yip! Yip!
Daisy is eating.

--- Difference between 'override' and 'new' (Hiding) ---
Tweety chirps: Tweet! Tweet!
Robin makes a generic animal sound.
```

### Virtual Functions vs. Interfaces

This is a common point of confusion, and it's important to understand the distinction:

*   **Virtual Functions:**
    *   Operate within an **inheritance hierarchy**.
    *   A base class provides a **default implementation** (or an abstract one if the method is `abstract virtual`).
    *   Derived classes *can* choose to override this implementation.
    *   You can only inherit from one base class, so virtual methods are limited to a single inheritance chain.
    *   Best for "is-a" relationships where derived classes are specialized versions of a base class.

*   **Interfaces:**
    *   Define a **contract** of behavior, not an implementation (though C# 8.0+ allows default implementations).
    *   Classes *must* implement all non-default members of an interface.
    *   A class can implement **multiple interfaces**, allowing for "multiple inheritance of type."
    *   Best for "can-do" relationships, where different, unrelated classes might share a common capability.

**When to choose which?**

*   Use **virtual functions** when you have a strong "is-a" relationship, and you want to provide a default behavior that derived classes can specialize.
*   Use **interfaces** when you want to define a contract that multiple, potentially unrelated classes can adhere to, promoting loose coupling and allowing for flexible architectural designs.