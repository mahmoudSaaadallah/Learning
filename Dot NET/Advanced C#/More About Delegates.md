### Generic Delegates: Type Safety and Reusability

Just like you can have generic classes and methods (e.g., `List<T>`, `MyMethod<T>`), you can also define **generic delegate types**. This means the delegate's signature can include type parameters, allowing it to work with any data type without having to define a new delegate for each specific type.

The primary benefits are:
1.  **Type Safety:** The compiler ensures that the methods assigned to the delegate match the specified generic types, preventing runtime errors.
2.  **Reusability:** You define the delegate once, and then you can use it with `int`, `string`, `MyCustomObject`, or any other type.
3.  **Reduced Boilerplate:** No need to declare `IntProcessor`, `StringProcessor`, `CustomerProcessor` delegates separately.

#### 1. Custom Generic Delegate Declaration

Let's start by defining our own generic delegate. Suppose we want a delegate that can process any type of data.

```csharp
// This delegate can point to any method that:
// - Takes one parameter of type T
// - Returns void
public delegate void DataProcessor<T>(T data);

// This delegate can point to any method that:
// - Takes one parameter of type TInput
// - Returns a value of type TResult
public delegate TResult Transformer<TInput, TResult>(TInput input);
```

Here, `DataProcessor<T>` and `Transformer<TInput, TResult>` are generic delegate types. `T`, `TInput`, and `TResult` are type parameters that will be specified when we use the delegate.

#### 2. Using Custom Generic Delegates

Let's see them in action:

```csharp
public class GenericDelegateExamples
{
    // Methods to be assigned to DataProcessor<T>
    public static void PrintString(string s)
    {
        Console.WriteLine($"String Data: {s.ToUpper()}");
    }

    public static void PrintInt(int i)
    {
        Console.WriteLine($"Integer Data: {i * 10}");
    }

    public static void PrintDateTime(DateTime dt)
    {
        Console.WriteLine($"DateTime Data: {dt.ToShortDateString()}");
    }

    // Methods to be assigned to Transformer<TInput, TResult>
    public static int StringLength(string s)
    {
        return s.Length;
    }

    public static bool IsEven(int i)
    {
        return i % 2 == 0;
    }

    public static string FormatDate(DateTime dt)
    {
        return dt.ToString("yyyy-MM-dd HH:mm:ss");
    }

    public static void Run()
    {
        Console.WriteLine("--- Using DataProcessor<T> ---");

        // DataProcessor for strings
        DataProcessor<string> stringProcessor = PrintString;
        stringProcessor("hello world");

        // DataProcessor for integers
        DataProcessor<int> intProcessor = PrintInt;
        intProcessor(42);

        // DataProcessor for DateTime
        DataProcessor<DateTime> dateTimeProcessor = PrintDateTime;
        dateTimeProcessor(DateTime.Now);

        Console.WriteLine("\n--- Using Transformer<TInput, TResult> ---");

        // Transformer from string to int
        Transformer<string, int> stringToIntTransformer = StringLength;
        Console.WriteLine($"Length of 'MIT': {stringToIntTransformer("MIT")}");

        // Transformer from int to bool
        Transformer<int, bool> intToBoolTransformer = IsEven;
        Console.WriteLine($"Is 7 even? {intToBoolTransformer(7)}");
        Console.WriteLine($"Is 10 even? {intToBoolTransformer(10)}");

        // Transformer from DateTime to string
        Transformer<DateTime, string> dateTimeToStringTransformer = FormatDate;
        Console.WriteLine($"Formatted current date: {dateTimeToStringTransformer(DateTime.Now)}");
    }
}
```

As you can see, we defined `DataProcessor<T>` and `Transformer<TInput, TResult>` once, and then we could use them with `string`, `int`, `DateTime`, etc., without creating new delegate types for each.

#### 3. The Power of `Action<T>` and `Func<T, TResult>` (Revisited)

In our previous discussion [[Delegates]], we touched upon `Action` and `Func`. It's crucial to understand that these are, in fact, **generic delegates** [[Generic]] provided by the .NET framework! They are the most common way to work with generic delegates because they cover almost all scenarios.

*   `Action<T1, ..., Tn>`: A generic delegate for methods that take `n` parameters and return `void`.
*   `Func<T1, ..., Tn, TResult>`: A generic delegate for methods that take `n` parameters and return a value of type `TResult`.
*   `Predicate<T1, ...,Tn>`: A generic delegate for method that take `n` parameters and return `bool`.

Let's rewrite some of the previous examples using `Action` and `Func` to emphasize their generic nature:

```csharp
public class BuiltInGenericDelegateExamples
{
    public static void Run()
    {
        Console.WriteLine("--- Using Action<T> ---");

        // Action<string>
        Action<string> printStringAction = (s) => Console.WriteLine($"Action String: {s.ToLower()}");
        printStringAction("HELLO GENERIC ACTION");

        // Action<int>
        Action<int> printIntAction = (i) => Console.WriteLine($"Action Int: {i + 100}");
        printIntAction(50);

        // Action<DateTime>
        Action<DateTime> printDateTimeAction = (dt) => Console.WriteLine($"Action DateTime: {dt.ToLongDateString()}");
        printDateTimeAction(DateTime.UtcNow);

        Console.WriteLine("\n--- Using Func<TInput, TResult> ---");

        // Func<string, int>
        Func<string, int> getStringLengthFunc = (s) => s.Length;
        Console.WriteLine($"Length of 'Advanced Delegates': {getStringLengthFunc("Advanced Delegates")}");

        // Func<double, double>
        Func<double, double> calculateSquareRootFunc = (d) => Math.Sqrt(d);
        Console.WriteLine($"Square root of 81: {calculateSquareRootFunc(81.0)}");

        // Func<int, int, string>
        Func<int, int, string> compareNumbersFunc = (a, b) => a > b ? $"{a} is greater" : $"{b} is greater or equal";
        Console.WriteLine(compareNumbersFunc(15, 10));
        Console.WriteLine(compareNumbersFunc(5, 5));
    }
}
```

Notice how `Action<T>` and `Func<T, TResult>` allow us to define the *behavior* (the delegate) once, and then apply it to different types, maintaining full type safety. This is incredibly powerful for creating flexible APIs and reusable components.

### Advanced Topic: Covariance and Contravariance with Generic Delegates

This is where things get a bit more academic, but it's a hallmark of truly understanding generic delegates. C# supports **covariance** and **contravariance** for generic delegates, which allows for more flexible type matching than strict equality.

*   **Covariance (`out` keyword):** Allows a delegate to be assigned a method that returns a *more derived* type than specified by the delegate's generic parameter. Applies to return types.
*   **Contravariance (`in` keyword):** Allows a delegate to be assigned a method that takes a *less derived* type (or a base type) as a parameter than specified by the delegate's generic parameter. Applies to input parameters.

Let's illustrate with an example. Consider a hierarchy: `Animal` -> `Dog`.

```csharp
public class Animal { public virtual void MakeSound() => Console.WriteLine("Animal sound"); }
public class Dog : Animal { public override void MakeSound() => Console.WriteLine("Woof!"); }

public class CovarianceContravarianceExample
{
    // Covariant delegate (return type)
    // 'out T' means T can be more derived than specified
    public delegate T CovariantProducer<out T>();

    // Contravariant delegate (input parameter)
    // 'in T' means T can be less derived (a base type) than specified
    public delegate void ContravariantConsumer<in T>(T item);

    // Methods for CovariantProducer
    public static Dog GetDog() => new Dog();
    public static Animal GetAnimal() => new Animal();

    // Methods for ContravariantConsumer
    public static void FeedAnimal(Animal animal) => Console.WriteLine($"Feeding an {animal.GetType().Name}");
    public static void TrainDog(Dog dog) => Console.WriteLine("Training a dog: " + dog.GetType().Name);

    public static void Run()
    {
        Console.WriteLine("\n--- Covariance Example (out T) ---");
        CovariantProducer<Animal> animalProducer;

        // We can assign a method that returns a Dog (more derived) to a delegate expecting an Animal
        animalProducer = GetDog; // Covariance in action!
        Animal producedAnimal = animalProducer();
        Console.WriteLine($"Produced: {producedAnimal.GetType().Name}"); // Output: Dog

        // We can also assign a method that returns an Animal
        animalProducer = GetAnimal;
        producedAnimal = animalProducer();
        Console.WriteLine($"Produced: {producedAnimal.GetType().Name}"); // Output: Animal

        Console.WriteLine("\n--- Contravariance Example (in T) ---");
        ContravariantConsumer<Dog> dogConsumer;

        // We can assign a method that takes an Animal (less derived/base type) to a delegate expecting a Dog
        dogConsumer = FeedAnimal; // Contravariance in action!
        dogConsumer(new Dog()); // We can pass a Dog to FeedAnimal, which expects an Animal

        // We can also assign a method that takes a Dog
        dogConsumer = TrainDog;
        dogConsumer(new Dog());
    }
}
```

**Key takeaway for `in` and `out`:**
*   `out T` (covariance): If a delegate returns `T`, you can assign a method that returns `T` or any type *derived from* `T`. Think "output" can be more specific.
*   `in T` (contravariance): If a delegate takes `T` as an argument, you can assign a method that takes `T` or any type *base to* `T`. Think "input" can be more general.

The built-in `Func<TInput, TResult>` is covariant on `TResult` and contravariant on `TInput`. The `Action<T>` delegates are contravariant on their input parameters. This is why you can often pass a `Func<object, string>` to a method expecting a `Func<string, string>` (contravariance on input) or assign a `Func<string>` to a `Func<object>` (covariance on output, if `string` was derived from `object`).
