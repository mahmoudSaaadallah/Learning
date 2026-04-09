A lambda expression is an **unnamed method** written in place of a delegate instance. The C# compiler converts a lambda expression into one of two forms:

-   A delegate instance.
-   An expression tree, of type `Expression<TDelegate>`, which represents the code inside the lambda expression as a traversable object model. This allows the lambda expression to be interpreted later at runtime (e.g., in LINQ to SQL/Entities).

## Basic Syntax and Examples

In its simplest form, a lambda expression looks like this:

```csharp
 delegate int Transformer (int i); // Assuming this delegate is defined elsewhere
 Transformer sqr = x => x * x;
 Console.WriteLine (sqr(3)); // Output: 9
```

The `x => x * x` part is the lambda expression.

### Internal Compiler Behavior
Internally, the compiler resolves lambda expressions by writing a private method and then moving the expression’s code into that method.

### General Form
A lambda expression has the following general form:

```
(parameters) => expression-or-statement-block
```

For convenience, you can omit the parentheses around the parameters if and only if there is exactly one parameter of an inferable type.

In our example, `x => x * x`, there is a single parameter `x`, and the expression is `x * x`.

### Parameter and Return Type Correspondence
Each parameter of the lambda expression corresponds to a delegate parameter, and the type of the expression (which may be `void`) corresponds to the return type of the delegate.

For instance, if we have the delegate:
```csharp
delegate int Transformer (int i);
```
The `x` in `x => x * x` corresponds to the `i` parameter, and the expression `x * x` (which evaluates to an `int`) corresponds to the `int` return type, making it compatible with the `Transformer` delegate.

### Statement Blocks
A lambda expression's code can be a statement block instead of a single expression. We can rewrite our example using a statement block:

```csharp
 Transformer sqr = x => { return x * x; };
 Console.WriteLine (sqr(3)); // Output: 9
```

## Common Usage with `Func` and `Action`

Lambda expressions are most commonly used with the built-in `Func` and `Action` delegates. You will often see the square example written as:

```csharp
Func<int,int> sqr = x => x * x;
Console.WriteLine(sqr(3)); // Output: 9
```

### Multiple Parameters
Here’s an example of an expression that accepts two parameters:

```csharp
Func<string,string,int> totalLength = (s1, s2) => s1.Length + s2.Length;
int total = totalLength ("hello", "world"); // total is 10
```

### Discard Parameters (C# 9)
If you do not need to use certain parameters, you can discard them with an underscore (`_`):

```csharp
Func<string,string,int> totalLength = (_,_) => 0; // Example: always returns 0, ignoring inputs
```

### Zero Arguments
Here’s an example of an expression that takes zero arguments:

```csharp
Func<string> greeter = () => "Hello, world";
Console.WriteLine(greeter()); // Output: Hello, world
```

### Implicit Typing with Lambdas (C# 10)
From C# 10, the compiler permits implicit typing with lambda expressions that can be resolved via the `Func` and `Action` delegates. This allows for a shorter syntax:

```csharp
var greeter = () => "Hello, world"; // Compiler infers Func<string>
Console.WriteLine(greeter());
```

## Explicitly Specifying Lambda Parameter and Return Types

The compiler can usually infer the type of lambda parameters contextually. However, when this is not the case, you must specify the type of each parameter explicitly.

Consider the following methods:
```csharp
void Foo<T> (T x) { /* ... */ }
void Bar<T> (Action<T> a) { /* ... */ }
```

The following code will fail to compile because the compiler cannot infer the type of `x`:
```csharp
// Bar (x => Foo (x)); // Error: What type is x?
```

We can fix this by explicitly specifying `x`'s type:
```csharp
Bar ((int x) => Foo (x)); // Fix 1: Explicit parameter type
```

This particular example can also be fixed in two other ways:
```csharp
Bar<int> (x => Foo (x)); // Fix 2: Specify type parameter for Bar
Bar<int> (Foo);          // Fix 3: Use method group conversion with type parameter
```

### Explicit Parameter Types for `var` (C# 10)
Another use for explicit parameter types (from C# 10) is when using `var` for the delegate:

```csharp
var sqr = (int x) => x * x; // Compiler infers sqr to be of type Func<int,int>
```
Without specifying `int`, implicit typing would fail because the compiler would know `sqr` should be `Func<T,T>` but wouldn’t know what `T` should be.

### Explicit Return Types (C# 10)
From C# 10, you can also specify the lambda return type:

```csharp
var sqr = int (int x) => x * x; // Explicitly specifies int as the return type
```
Specifying a return type can improve compiler performance, especially with complex nested lambdas.

## Default Lambda Parameters (C# 12)

Just as ordinary methods can have optional parameters:

```csharp
void Print (string message = "") => Console.WriteLine (message);
```

So too can lambda expressions (from C# 12):

```csharp
var print = (string message = "") => Console.WriteLine (message);
print ("Hello"); // Output: Hello
print ();        // Output: (empty line)
```
This feature is particularly useful when working with libraries such as ASP.NET Minimal API.