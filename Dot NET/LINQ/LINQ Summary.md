## Introduction to LINQ

LINQ, which stands for **Language Integrated Query**, is a powerful feature in C# that allows you to write queries and perform operations on data directly within your code. It enables you to work with collections, databases, XML, and other data sources using a consistent and expressive syntax.

*   **LINQ:** Language Independent Query.
*   **Purpose:** Use 40+ functions (Query Operators) against data, regardless of the data store.

## Core C# Features Supporting LINQ

### Implicitly Typed Local Variables (`var`)

When you use `var` to declare any variable in C#, it's called an **Implicitly Typed Local Variable**. This means the compiler infers and specifies the data type based on the default value assigned to the variable at compile time.

**Example:**

```csharp
var d = 124.213; // d is inferred as type double
Console.WriteLine(d.GetType()); // Output: System.Double
// As C# is a strong data type language, you cannot change the data type at runtime.
// d = "new data"; // Compile-time error: Cannot implicitly convert type 'string' to 'double'.
```

### Extension Methods
[[Extension Functions]]
Extension methods allow you to add new methods to existing types without modifying the original type's source code. This is crucial for LINQ, as most LINQ operators are implemented as extension methods on `IEnumerable<T>` or `IQueryable<T>`.

**Conditions for creating an Extension Method:**

1.  The method must be defined in a `static` class.
2.  The first parameter of the extension method must be preceded by the `this` keyword, indicating the type it extends.

**Example:**

Consider an `int` and a `mirror` function to reverse its digits:

```csharp
// Original static method
class Int32Extension
{
    public static int Mirror(int x)
    {
        string s = x.ToString();
        char[] charArray = s.ToCharArray();
        Array.Reverse(charArray);
        return int.Parse(new string(charArray));
    }
}

// Usage:
int x = 12345;
WriteLine(Int32Extension.Mirror(x)); // Output: 54321

// To call it as a member method (e.g., x.Mirror()), we make it an extension method:
static class Int32ExtensionV2
{
    public static int Mirror(this int x) // 'this' keyword makes it an extension method
    {
        string s = x.ToString();
        char[] charArray = s.ToCharArray();
        Array.Reverse(charArray);
        return int.Parse(new string(charArray));
    }
}

// Usage as an extension method:
WriteLine(x.Mirror()); // Output: 54321
```

---
---

### Anonymous Types
[[Anonyms Object]]

Anonymous types in C# allow you to create objects with properties without explicitly defining a named class type. They are particularly useful when you need to project data from a data source (like a LINQ query) into a new type without the overhead of creating a formal class.

**Key Characteristics:**

*   **Properties:** Anonymous types consist of a set of read-only properties. Each property is associated with a value, and you can access these properties using dot notation.
*   **Implicitly Typed:** The type of an anonymous type is inferred by the compiler. You don't specify a type explicitly; instead, the compiler generates a type behind the scenes.
*   **Read-Only (Immutable):** The properties of an anonymous type are read-only. Once set during initialization, their values cannot be changed.
*   **Creation:** To create an anonymous type, you must use `var` as its type.

**Syntax:**

```csharp
var anonymousObject = new { Property1 = value1, Property2 = value2, ... };
```

**Examples:**

```csharp
var Employee1 = new { Id = 1, Name = "Mahmoud", Salary = 12000 };
WriteLine(Employee1); // Output: { Id = 1, Name = Mahmoud, Salary = 12000 }

// Employee1.Id = 2; // Compile-time error: Property or indexer 'AnonymousType#1.Id' cannot be assigned to -- it is read only.

// Anonymous Type Equality:
// If you create another object with the same property names (case-sensitive), same property types,
// and in the same sequence, it will be considered an object of the same anonymous type.
var Employee2 = new { Id = 2, Name = "Ibrahim", Salary = 5000 }; // Different values, same type structure
var Employee3 = new { Id = 2, Salary = 5000, Name = "Ibrahim" }; // Different property sequence, thus a new anonymous type
var Employee4 = new { Id = 1, Name = "Mahmoud", Salary = 12000 }; // Same values as Employee1

// Anonymous types override Equals() and GetHashCode() based on value equality, not reference equality.
if (Employee1.Equals(Employee4))
    WriteLine("Equals"); // Output: Equals (because values are the same)

WriteLine(Employee1.GetHashCode()); // Will be the same as Employee4.GetHashCode()
WriteLine(Employee4.GetHashCode());
```

**Modern C#/.NET Notes:**

*   **`record` types (C# 9 / .NET 5+):** For scenarios where you need a named type with value equality and immutability, `record` types offer a more formal and powerful alternative to anonymous types. They automatically generate `Equals`, `GetHashCode`, and `ToString` methods based on property values.

```csharp
public record Employee(int Id, string Name, decimal Salary);
var employeeRecord1 = new Employee(1, "Mahmoud", 12000);
var employeeRecord4 = new Employee(1, "Mahmoud", 12000);
WriteLine(employeeRecord1.Equals(employeeRecord4)); // Output: True
```

## LINQ Query Fundamentals

### LINQ Queries Against Sequences

*   **Sequence:** Any class implementing the `IEnumerable<T>` interface.
*   **Local Sequence:** Data sources like `L2Object` (in-memory collections), `L2ADO` (ADO.NET DataSets), `L2XML` (XML documents).
*   **Remote Sequence:** Data sources like `L2Sql` (SQL databases), `L2EF` (Entity Framework).
*   A sequence contains elements.

## LINQ Query Operators

LINQ operators can generally be used in two syntaxes:

1.  **Fluent Syntax (Method Syntax):** Uses extension methods directly on the collection.
2.  **Query Expression (Query Syntax):** Resembles SQL query syntax, starting with `from` and ending with `select` or `group by`.

### Where Operator (Filtering)
[[LINQ Where]]
The `Where` operator is used to filter elements from a collection or data source based on a specified condition (a predicate). It retrieves only the elements that meet the criteria.

**Syntax (Fluent):**

```csharp
IEnumerable<T> filteredCollection = sourceCollection.Where(element => condition);
```

*   `sourceCollection`: The original collection or data source.
*   `element`: A placeholder variable representing each element in the collection.
*   `condition`: A Boolean expression that defines the filtering criteria.

**Examples:**

```csharp
List<int> intLst = new List<int>() { 1, 6, 7, 8, 9, 12, 17, 5};
List<string> NameLst = new List<string> { "Mahmoud", "Ahmed", "Emad", "Omer", "Saadallah" };

// Fluent Syntax (using Enumerable static method)
var Result = Enumerable.Where(intLst, i => i % 2 == 0); // Static method call
var Result1 = Enumerable.Where(NameLst, i => i.Length > 5);
IEnumerable<int> Result2 = Enumerable.Where(intLst, i => i % 3 == 0);

// Using Where as an Extension Method (more common)
var Result3 = NameLst.Where(i => i.Length > 5);

// Query Expression (Query Syntax)
var Result4 = from i in NameLst
              where i.Length > 5
              select i;
// Query expressions are SQL-like and are easy to use with Join, Group, Let, Into.
// They start with 'from' (introducing a range variable) and end with 'select' or 'group by'.

// Type of Result4 is WhereListIterator (an internal iterator type).
WriteLine(Result4.GetType()); // Output: WhereListIterator

// To change the type to a specific collection type, use casting operators:
var Result5 = NameLst.Where(i => i.Length > 5).ToList();
WriteLine(Result5.GetType()); // Output: System.Collections.Generic.List`1[System.String]

foreach (string i in Result5)
    WriteLine(i); // Output: Mahmoud, Saadallah

// Multiple Conditions:
// 1. Using && operator
var Result6 = ProductList.Where(i => (i.UnitsInStock == 0) && (i.Category == "Meat/Poultry"));
// 2. Chaining Where clauses
Result6 = ProductList.Where(i => i.UnitsInStock == 0).Where(i => i.Category == "Meat/Poultry");
```

#### Deferred Execution

Most LINQ operators, including `Where`, use **deferred execution**. This means the query is not executed immediately when it's defined, but rather when its results are actually enumerated (e.g., in a `foreach` loop, or when `ToList()`, `ToArray()` are called). This allows the query to always operate on the latest version of the data.

```csharp
NameLst.AddRange(new string[] { "Ibrahim", "Mostafa", "said" });
foreach(var i in Result4) // Result4 was defined before AddRange, but will include new items
    Write(i + ", "); // Output: "Mahmoud", "Ahmed", "Emad", "Omer", "Saadallah", "Ibrahim", "Mostafa", "said"
// If Result5 (which was ToList()ed) were enumerated here, it would NOT include the new items.
```

#### Indexed Where

There's an overload of `Where` that accepts two parameters: the element and its zero-based index, returning a `bool`. This is only valid with Fluent Syntax.

```csharp
// Select products with 0 units in stock that are among the first 10 products in the list.
Result6 = ProductList.Where((p, i) => p.UnitsInStock == 0 && i < 10);
```

### Select Operator (Projection)
[[LINQ Select]]
The `Select` operator is used to project or transform elements from a collection into a new form. It allows you to create a new sequence of values based on some computation or transformation applied to each element.

**Syntax (Fluent):**

```csharp
IEnumerable<TResult> projection = sourceCollection.Select(element => transformation);
```

*   `sourceCollection`: The original collection or data source.
*   `element`: A placeholder variable representing each element in the source collection.
*   `transformation`: An expression that defines how each element should be transformed.

**Examples:**

```csharp
// Projecting to a single property (string)
var Result7 = ProductList.Select(i => i.ProductName);
foreach (var i in Result7)
    Write (i + ",  ");

// Query Expression equivalent
Result7 = from p in ProductList
          select p.ProductName;

// Projecting to an Anonymous Type
var Result8 = ProductList.Select(p => new { Name = p.ProductName });
// Query Expression equivalent
Result8 = from p in ProductList
          select new { Name = p.ProductName };

// Real-world example: Applying a discount and selecting specific properties
var DiscountedLst = ProductList.Select(p => new
{
    ProductName = p.ProductName,
    p.ProductID, // Property name inferred from source
    p.Category,
    UnitPrice = p.UnitPrice * 0.9M // Transformed property
});
```

#### Indexed Select

Similar to `Indexed Where`, `Select` also has an overload that provides the element's index. This is only valid with Fluent Syntax.

```csharp
var Result9 = ProductList.Select((p, i) => new
{
    Index = i,
    ProductName = p.ProductName
});
```

### Ordering Elements (`OrderBy`, `ThenBy`)
[[LINQ OrderBy]]
The `OrderBy` operator sorts elements from a collection based on one or more specified criteria, in ascending or descending order.

**Syntax (Fluent):**

```csharp
IEnumerable<TSource> orderedCollection = sourceCollection.OrderBy(element => keySelector);
```

*   `keySelector`: An expression that specifies the property or value by which to sort.

**Examples:**

```csharp
// Default is Ascending
var Result10 = ProductList.OrderBy(p => p.UnitsInStock);

// Query Expression equivalent
Result10 = from p in ProductList
           orderby p.UnitsInStock // Default is ascending
           select p;

// Chaining LINQ operators: The output of one operator becomes the input of the next.
var Result11 = ProductList.OrderBy(p => p.UnitsInStock)
    .Select(i => (i.ProductName, i.ProductID))
    .Where(p => p.ProductID > 7);

// Descending Order
Result11 = ProductList.OrderByDescending(p => p.UnitsInStock)
    .Select(i => (i.ProductName, i.ProductID))
    .Where(p => p.ProductID > 7);

// Multiple Order Criteria (Secondary Sorting)
// Fluent Syntax: Use ThenBy for subsequent sorting criteria.
Result10 = ProductList.OrderBy(p => p.UnitsInStock).ThenBy(p => p.UnitPrice);
Result10 = ProductList.OrderBy(p => p.UnitsInStock).ThenByDescending(p => p.UnitPrice);

// Query Expression: List multiple criteria separated by commas.
Result10 = from p in ProductList
           orderby p.UnitsInStock ascending, p.UnitPrice descending
           select p;
```

**Modern C#/.NET Notes:**

*   **`OrderBy` with `record` types:** When sorting `record` types, the default comparison will use the defined properties. If you need custom comparison logic, you can provide an `IComparer<T>`.
*   **`MinBy` and `MaxBy` (C# 10 / .NET 6+):** These operators allow you to find the element that has the minimum or maximum value for a specific key, without sorting the entire collection.

```csharp
var productWithMinPrice = ProductList.MinBy(p => p.UnitPrice);
var productWithMaxStock = ProductList.MaxBy(p => p.UnitsInStock);
```

### Element Operators (Single Element Retrieval)

Element operators retrieve a single element from a collection based on specific criteria. They are useful for finding a unique element or the first/last element. These operators are generally only valid with Fluent Syntax.
#Important_Note
1.  **`First()`**: Returns the first element in a sequence. `First(predicate)` returns the first element that satisfies a condition. Throws an exception if no matching element is found.
2.  **`FirstOrDefault()`**: Returns the first element in a sequence, or `default(T)` if the sequence is empty. `FirstOrDefault(predicate)` returns the first element that satisfies a condition, or `default(T)` if no such element is found. Useful for avoiding exceptions.
3.  **`Single()`**: Returns the *only* element in a sequence. `Single(predicate)` returns the *only* element that satisfies a condition. Throws an exception if there is more than one matching element or no matching element.
4.  **`SingleOrDefault()`**: Returns the *only* element in a sequence, or `default(T)` if the sequence is empty. `SingleOrDefault(predicate)` returns the *only* element that satisfies a condition, or `default(T)` if no such element is found. Throws an exception if there are multiple matching elements.
5.  **`Last()`**: Returns the last element in a sequence. `Last(predicate)` returns the last element that satisfies a condition. Throws an exception if no matching element is found.
6.  **`LastOrDefault()`**: Returns the last element in a sequence, or `default(T)` if the sequence is empty. `LastOrDefault(predicate)` returns the last element that satisfies a condition, or `default(T)` if no such element is found. Useful for avoiding exceptions.
7.  **`ElementAt(index)`**: Returns the element at a specified zero-based index. Throws an `ArgumentOutOfRangeException` if the index is out of range.
8.  **`ElementAtOrDefault(index)`**: Returns the element at a specified index, or `default(T)` if the index is out of range.

**Examples:**

```csharp
var single = ProductList.First(); // First product
WriteLine(single);

single = ProductList.FirstOrDefault(p => p.UnitPrice % 5 == 0); // First product with unit price divisible by 5
WriteLine(single);

// single = ProductList.Single(p => p.UnitPrice % 2 == 0); // Throws exception if multiple products have even unit price
single = ProductList.Single(p => p.ProductID == 7); // No exception if only one product has ProductID 7
WriteLine(single);

// It's generally better to use `OrDefault` variants to avoid exceptions.

single = ProductList.Last(p => p.UnitPrice > 50);
WriteLine(single);

single = ProductList.ElementAt(7);
WriteLine(single);
```

#### Hybrid Syntax

You can combine Query Expression and Fluent Syntax. When doing so, the Query Expression part must be enclosed in parentheses before applying Fluent Syntax methods.

```csharp
var single2 = (from p in ProductList
               where p.UnitsInStock == 0
               select new {p.ProductName, p.UnitPrice}).First();
```

### Aggregate Operators (Summary Operations)

Aggregate operators perform cumulative or iterative operations on elements in a collection, combining them into a single result.
[[LINQ Popular Methods distinct, count, sum, min, max, avg ,take, skip]]
1.  **`Count()`**: Returns the total number of elements in a collection. `Count(predicate)` returns the number of elements that satisfy a condition.
2.  **`Sum()`**: Computes the sum of all numeric values in a collection. `Sum(selector)` computes the sum of a projected numeric property.
3.  **`Min()`**: Finds the minimum value in a collection. `Min(selector)` finds the minimum value of a projected property. (Requires `IComparable` if no selector is used on complex types).
4.  **`Max()`**: Finds the maximum value in a collection. `Max(selector)` finds the maximum value of a projected property. (Requires `IComparable` if no selector is used on complex types).
5.  **`Average()`**: Computes the average (mean) of all numeric values in a collection. `Average(selector)` computes the average of a projected numeric property.
6.  **`Aggregate()`**: Applies a custom aggregation function to a collection. This is the most flexible aggregate operator, allowing complex custom reductions.
    *   `Aggregate((acc, next) => ...)`: Applies an accumulator function.
    *   `Aggregate(seed, (acc, next) => ...)`: Provides an initial seed value.
    *   `Aggregate(seed, (acc, next) => ..., resultSelector)`: Provides a seed and a final result transformation.

**Examples:**

```csharp
var agg = ProductList.Count(); // Total count
agg = ProductList.Count(p => p.UnitPrice > 50); // Count with condition
WriteLine(agg);

agg = (int)ProductList.Sum(p => p.UnitPrice);
WriteLine(agg);

agg = (int)ProductList.Min(p => p.UnitPrice);
WriteLine(agg);

agg = (int)ProductList.Max(p => p.UnitPrice);
WriteLine(agg);

agg = (int)ProductList.Average(p => p.UnitPrice);
WriteLine(agg);

// Example of Aggregate: Calculate the product of all UnitPrices
// decimal productOfPrices = ProductList.Aggregate(1M, (currentProduct, p) => currentProduct * p.UnitPrice);
```

**Modern C#/.NET Notes:**
[[LINQ GroupBy]]
*   **`CountBy` (C# 10 / .NET 6+):** This operator allows you to count occurrences of elements based on a key selector, returning a dictionary-like structure (`IReadOnlyDictionary<TKey, int>`).

```csharp
var categoryCounts = ProductList.CountBy(p => p.Category);
foreach (var entry in categoryCounts)
{
	WriteLine($"Category: {entry.Key}, Count: {entry.Value}");
}
```

### Generators Operators (Sequence Creation)

Generator operators create sequences of values based on specified patterns or rules, rather than operating on existing collections. They are called as static members of the `Enumerable` class.

1.  **`Range(start, count)`**: Generates a sequence of consecutive integers within a specified range.
```csharp
var numbers = Enumerable.Range(1, 5); // Generates 1, 2, 3, 4, 5
```

2.  **`Repeat(element, count)`**: Generates a sequence that repeats a specified value a specified number of times.
```csharp
var repeatedValues = Enumerable.Repeat("Hello", 3); // Generates "Hello", "Hello", "Hello"
```
*   **Important for Reference Types:** If you `Repeat` a reference type, it does not create multiple objects. Instead, it creates a sequence of references to the *same* object. Modifying the original object will affect all instances in the repeated sequence.
```csharp
var Gen4 = Enumerable.Repeat(ProductList[3], 5);
// ... (enumerate Gen4)
ProductList[3].ProductName = "Temp"; // Modifies the original object
// ... (enumerate Gen4 again)
// Now all products in Gen4 will have the name "Temp".
```

3.  **`Empty<TResult>()`**: Generates an empty sequence of a specified type.
```csharp
var emptySequence = Enumerable.Empty<int>(); // Generates an empty sequence
```

4.  **`DefaultIfEmpty<TSource>(defaultValue)`**: Generates a sequence with a single default value if the source sequence is empty. If the source sequence is not empty, it returns the source sequence unchanged.
```csharp
var numbers = new List<int>();
var result = numbers.DefaultIfEmpty(0); // Generates a sequence containing only 0 if 'numbers' is empty
var Gen5 = Enumerable.Range(5, 3); // Generates 5, 6, 7
var Gen6 = Gen5.DefaultIfEmpty(0); // Returns 5, 6, 7 (Gen5 is not empty)
```

5.  **`Generate` (Not a standard LINQ operator, but a common pattern for sequence generation):** The example provided for `Generate` is a common pattern for generating sequences based on a seed, condition, and iteration function, often implemented manually or using libraries like `MoreLINQ`. The standard `Enumerable` class does not have a `Generate` method.
```csharp
// Example of a custom Fibonacci sequence generator (similar to the concept of 'Generate')
// This would typically be a custom extension method or a manual loop.
// var fibonacci = Enumerable.Generate(
//      seed: (1, 1),
//      condition: pair => pair.Item1 <= 100,
//      iterate: pair => (pair.Item2, pair.Item1 + pair.Item2),
//      resultSelector: pair => pair.Item1
// ); // Generates the Fibonacci sequence up to 100
```

6.  **`ToLookup<TSource, TKey>(keySelector)`**: Generates a lookup table from a sequence by grouping elements based on a key selector. It's similar to `GroupBy` but creates an `ILookup<TKey, TElement>` data structure, which is optimized for efficient retrieval of grouped elements by key.
```csharp
var Gen5 = Enumerable.Range(0, 10); // Example sequence
var lookup = Gen5.ToLookup(n => n % 2 == 0); // Groups even and odd numbers
// lookup[true] would contain all even numbers, lookup[false] all odd numbers.
```

7.  **`ToDictionary<TSource, TKey>(keySelector)`**: Generates a dictionary from a sequence using key and value selectors.
    *   **Overloads:**
        *   `ToDictionary(keySelector)`: Uses the element itself as the value.
        *   `ToDictionary(keySelector, comparer)`: Specifies a custom comparer for the keys.
        *   `ToDictionary(keySelector, elementSelector)`: Specifies both key and value selectors.
        *   `ToDictionary(keySelector, elementSelector, comparer)`: Specifies key, value, and a key comparer.
```csharp
// var Gen7 = Gen.ToDictionary(n => n % 2 == 2); // This key selector will always be false, resulting in an empty dictionary or error if key is not unique.
// A more practical example:
var productDictionary = ProductList.ToDictionary(p => p.ProductID); // Key: ProductID, Value: Product object
var productNamesById = ProductList.ToDictionary(p => p.ProductID, p => p.ProductName); // Key: ProductID, Value: ProductName
```

---
### SelectMany Operator (Flattening)
[[LINQ SelectMany]]
The `SelectMany` operator in LINQ is used to project and flatten elements from nested collections or sequences into a single, flattened sequence. It's particularly useful when you have a collection of collections (or a nested structure) and you want to work with the individual elements inside those nested collections as if they were all part of a single flat collection.

**Return Type:** `IEnumerable<TResult>`: This is the resulting flattened sequence containing the elements from the nested collections.

**Examples:**

```csharp
List<string> strlst = new List<string> { "Ahmed Mohammed", "Mahmoud Saadallah", "Ibrahim Osman" };

// Fluent Syntax
var Selector = strlst.SelectMany(s => s.Split(" "));
foreach(var word in Selector)
    Write(word + " "); // Output: Ahmed Mohammed Mahmoud Saadallah Ibrahim Osman

// Query Expression equivalent
var Selector2 = from name in strlst
                from s in name.Split(" ") // 'from' clause for flattening
                select s;
```

### Set Operators

Set operators in LINQ are used to perform set-related operations on collections or sequences. These operators allow you to manipulate collections as sets, taking into account concepts like union, intersection, difference, and distinct values.
[[LINQ Popular Methods Union, Intersect, Except, Concat]]

1.  **`Union(secondSequence)`**: Returns a new collection that contains all *distinct* elements from both source collections. It effectively removes duplicates.
```csharp
var unionResult = collection1.Union(collection2);
```

2.  **`Intersect(secondSequence)`**: Returns a new collection that contains the elements that exist in *both* source collections.
```csharp
var intersectResult = collection1.Intersect(collection2);
```

3.  **`Except(secondSequence)`**: Returns a new collection that contains the elements that exist in the *first* collection but *not* in the second collection.
```csharp
var exceptResult = collection1.Except(collection2);
```

4.  **`Distinct()`**: Returns a new collection with distinct elements from the source collection.
```csharp
var distinctResult = collection.Distinct();
```

5.  **`Concat(secondSequence)`**: Returns a new collection that contains all elements from both source collections, preserving the order and *allowing duplicates*.
```csharp
var concatResult = collection1.Concat(collection2);
```

6.  **`SequenceEqual(secondSequence)`**: Determines if two collections have the same elements in the same order and quantity. Returns `true` or `false`.
```csharp
bool areEqual = collection1.SequenceEqual(collection2);
```

**Examples:**

```csharp
var seq1 = Enumerable.Range(0, 100); // Creates a range from 0 to 99
var seq2 = Enumerable.Range(50, 100); // Creates a range from 50 to 149

var output = seq1.Union(seq2); // Result: 0-149 (distinct elements, duplicates removed)
// Union has overloads that accept an IEqualityComparer<T> for custom comparison logic.

output = seq1.Concat(seq2); // Result: 0-99, then 50-149 (duplicates allowed)

output = output.Distinct(); // Removes duplicate values from the concatenated sequence.

output = seq1.Except(seq2); // Result: 0-49 (elements in seq1 but not in seq2)

output = seq1.Intersect(seq2); // Result: 50-99 (elements common to both seq1 and seq2)
```

**Modern C#/.NET Notes:**
#Important_Note
*   **Collection Expressions (C# 12 / .NET 8+):** While not directly LINQ operators, collection expressions provide a concise syntax for creating collections, which can then be used with LINQ set operations.
```csharp
List<int> list1 = [1, 2, 3];
List<int> list2 = [3, 4, 5];
var union = list1.Union(list2); // [1, 2, 3, 4, 5]
```
*   **Range and Index Operators (C# 8 / .NET Core 3.0+):** These operators (`..` and `^`) provide a concise way to slice arrays and lists, which can be useful when preparing sequences for set operations.
```csharp
var lst3 = Enumerable.Range(0, 100).ToList();
WriteLine(lst3[^1]); // Returns the last element (99)
WriteLine(lst3[^5]); // Returns the fifth element from the end (95)
List<int> lst4 = lst3[1..10]; // Returns a list of elements from index 1 up to (but not including) index 10.
```

### Casting Operators

Casting in LINQ allows you to convert the type of a sequence or its elements.

1.  **`ToList()`**: Converts the sequence to a `List<T>`.
2.  **`ToArray()`**: Converts the sequence to an array (`T[]`).
3.  **`ToDictionary()`**: Converts the sequence to a `Dictionary<TKey, TValue>`.
    *   **Overloads:**
        *   `ToDictionary(keySelector)`: Uses the element itself as the value.
        *   `ToDictionary(keySelector, comparer)`: Specifies a custom comparer for the keys.
        *   `ToDictionary(keySelector, elementSelector)`: Specifies both key and value selectors.
        *   `ToDictionary(keySelector, elementSelector, comparer)`: Specifies key, value, and a key comparer.
4.  **`ToHashSet()`**: Converts the sequence to a `HashSet<T>`. `HashSet<T>` is a collection that stores unique elements and provides very fast lookup operations.
    *   **Overloads:**
        *   `ToHashSet()`: Creates a `HashSet<T>` with the default equality comparer.
        *   `ToHashSet(comparer)`: Specifies a custom equality comparer.
5.  **`ToLookup()`**: Converts the sequence to an `ILookup<TKey, TElement>`. Similar to `ToDictionary`, but allows multiple values per key.
    *   **Overloads:** Similar to `ToDictionary` with key and element selectors and comparers.

**Examples:**

```csharp
Product[] casting1 = ProductList.Where(p => p.UnitsInStock == 0).ToArray();
List<Product> casting2 = ProductList.Where(p => p.UnitsInStock == 0).ToList();
var casting3 = ProductList.Where(p => p.UnitsInStock == 0).ToHashSet();
var casting4 = ProductList.Where(p => p.UnitsInStock == 0).ToDictionary(p => p.ProductID);
var casting5 = ProductList.Where(p => p.UnitsInStock == 0).ToLookup(p => p.ProductID);
```

### Quantifiers Operators
[[LINQ Popular Methods Any, All, Contains]]
Quantifiers are operators that determine whether certain elements in a collection satisfy a specific condition. Quantifiers return a Boolean value (`true` or `false`) based on whether the condition is met by any or all elements in the collection.

1.  **`Any()`**: Checks whether there are any elements in a collection. `Any(predicate)` checks if any elements satisfy a specified condition.
    *   Returns `true` if at least one element meets the condition; otherwise, `false`.
    *   If used without a predicate, it checks if the collection contains any elements at all.
2.  **`All(predicate)`**: Checks whether 
	*all* elements in a collection satisfy a specified condition.
    *   Returns `true` if every element in the collection meets the condition; otherwise, `false`.

**Examples:**

```csharp
List<int> intlst = new List<int> { 1, 2, 3, 4, 5, 6, 7 };

WriteLine(intlst.Any(n => n % 2 == 0)); // Output: True (because 2, 4, 6 are even)
WriteLine(intlst.Any(n => n < 0));      // Output: False (no negative numbers)
WriteLine(intlst.Any());                // Output: True (list is not empty)

WriteLine(intlst.All(n => n > 0));      // Output: True (all numbers are positive)
WriteLine(intlst.All(n => n % 2 == 0)); // Output: False (not all numbers are even)
```

### Zip Operator

The `Zip` operator in LINQ is used to combine elements from two (or more) sequences (collections) into a single sequence, element by element, based on their position. It creates pairs (or tuples) of elements from the input sequences where the elements at the same index are combined.

The `Zip` operator continues processing until it reaches the end of the *shortest* input sequence. The length of the resulting sequence is determined by the shortest of the input sequences.

**Overloads:**
1.  **`Zip(secondSequence)`**: Combines two sequences into a sequence of `ValueTuple<TFirst, TSecond>`.
2.  **`Zip(secondSequence, resultSelector)`**: Combines two sequences and applies a `resultSelector` function to each pair of elements to produce a custom output type.
3.  **`Zip(secondSequence, thirdSequence)` (C# 11 / .NET 7+)**: Combines three sequences into a sequence of `ValueTuple<TFirst, TSecond, TThird>`.

**Examples:**

```csharp
List<int> number = new List<int> { 1, 2, 3, 4 };
List<string> words = new List<string> { "One", "Two", "Three" };

// Zip with two sequences (default tuple output)
var zipped = number.Zip(words);
foreach(var i in zipped)
    Write(i); // Output: (1, One)(2, Two)(3, Three)
// The '4' from 'number' is ignored because 'words' is shorter.

// Zip with a result selector to create an anonymous type
var zipped2 = number.Zip(words, (n, name) => new {n, Names = name.ToUpper()});
foreach(var i in zipped2)
    Write(i); // Output: { n = 1, Names = ONE }{ n = 2, Names = TWO }{ n = 3, Names = THREE }

// Zip with three sequences (C# 11 / .NET 7+)
List<char> ch = new List<char>() { '!', '@', '#', '$', '%' };
var zipped3 = number.Zip(words, ch);
foreach (var i in zipped3)
    Write(i); // Output: (1, One, !)(2, Two, @)(3, Three, #)
// The '4' from 'number' and '$', '%' from 'ch' are ignored because 'words' is the shortest.
```

### Grouping
[[LINQ GroupBy]]
The `GroupBy` operator in LINQ is used to group elements from a collection based on one or more key criteria. This operator allows you to organize data into groups, where each group contains elements that share a common key or set of keys. The result is typically an enumerable of groups, where each group has a key and contains a collection of elements that match that key.

**Key points to remember about GroupBy:**

*   The result of `GroupBy` is an `IEnumerable<IGrouping<TKey, TElement>>`. `IGrouping<TKey, TElement>` itself is an `IEnumerable<TElement>` and has a `Key` property.
*   You can iterate through these groups to access the elements within each group.
*   You can use multiple key criteria by providing a more complex keySelector lambda expression (e.g., an anonymous type for the key).
*   You can perform various operations on the grouped data, such as aggregations, filtering, and projections, using other LINQ operators after the `GroupBy` operation.
*   `GroupBy` is a powerful operator for organizing and aggregating data based on specific criteria, making it a valuable tool in LINQ queries for tasks like data summarization and reporting.

**Examples:**

```csharp
// Query Expression for Grouping
var items = from p in ProductList
            where p.UnitsInStock > 0
            group p by p.Category; // Group products by their Category

foreach (var proGroup in items)
{
    WriteLine(proGroup.Key); // Output: Beverages, Condiments, Produce, etc.
    // You can iterate through proGroup to access products within that category:
    // foreach (var product in proGroup) { WriteLine($"  - {product.ProductName}"); }
}

// Grouping with additional filtering and ordering on the groups themselves (using 'into')
items = from p in ProductList
        where p.UnitsInStock > 0
        group p by p.Category
        into Groups // 'into' allows you to continue querying the groups
        where Groups.Count() > 10 // Filter groups with more than 10 products
        orderby Groups.Count() descending // Order groups by their count
        select Groups;

foreach (var proGroup in items)
    WriteLine(proGroup.Key); // Output: Confections, Beverages, Seafood, Condiments

// Fluent Syntax for Grouping
items = ProductList.GroupBy(p => p.Category)
    .Where(g => g.Count() > 10)
    .OrderByDescending(g => g.Count());

foreach (var proGroup in items)
    WriteLine(proGroup.Key); // Output: Confections, Beverages, Seafood, Condiments
```

**Modern C#/.NET Notes:**

*   **`GroupBy` with `record` types:** `record` types can be used effectively as keys in `GroupBy` operations, leveraging their built-in value equality.
*   **`GroupJoin`:** For joining two sequences and then grouping the results of the join, `GroupJoin` is a specialized operator.

### Let and Into Keywords (Query Syntax Only)

The `Let` and `Into` keywords are specific to LINQ Query Syntax and provide powerful ways to structure complex queries.

1.  **`Into` Keyword:**
    *   The `into` keyword allows you to continue a query after a `group by` or `select` clause. It effectively starts a *new* query from the results of the previous clause.
    *   When `into` is used, the previous range variable (e.g., `p` in `from p in ProductList`) is no longer accessible. You must use the new range variable introduced by `into`.
    *   It's useful for performing further filtering, ordering, or projection on the *groups* themselves after a `group by`, or on the *projected elements* after a `select`.

    **Example (from `GroupBy` section):**

```csharp
var items = from p in ProductList
			where p.UnitsInStock > 0
			group p by p.Category
			into Groups // 'Groups' is the new range variable, 'p' is no longer accessible
			where Groups.Count() > 10
			orderby Groups.Count() descending
			select Groups;
```

**Example with `select` and `into`:**

```csharp
List<string> names = new List<string>(){ "Mahmoud", "Aly", "Osman", "Sally", "Shrouk", "Omar" };
var Novowels = from n in names
			   select Regex.Replace(n, "[aeiouAEIOU]", string.Empty) // Project to strings without vowels
			   into deletedVowels // 'deletedVowels' is the new range variable, 'n' is no longer accessible
			   where deletedVowels.Length >= 4 // Filter the projected strings
			   select deletedVowels;
foreach (var n in Novowels)
	Write(n + "  "); // Output: Mhmd  Slly  Shrk
```

2.  **`Let` Keyword:**
    *   The `let` keyword allows you to introduce a new range variable that stores the result of an expression. This new variable can then be used in subsequent clauses of the query.
    *   Unlike `into`, when `let` is used, the *original* range variable (e.g., `n` in `from n in names`) remains accessible alongside the new `let` variable.
    *   It's useful for storing intermediate results or complex calculations that you want to reuse within the same query, improving readability and potentially performance by avoiding redundant calculations.

    **Example:**

```csharp
List<string> names = new List<string>(){ "Mahmoud", "Aly", "Osman", "Sally", "Shrouk", "Omar" };
var Novowels = from n in names
			   let deletedVowels = Regex.Replace(n, "[aeiouAEIOU]", string.Empty) // 'deletedVowels' is a new variable
			   where deletedVowels.Length >= 4
			   orderby n.Length // 'n' (the original range variable) is still accessible here
			   select deletedVowels;
foreach (var n in Novowels)
	Write(n + "  "); // Output: Mhmd  Slly  Shrk
```

**Summary of `Let` vs. `Into`:**

*   **`Let`**: Introduces a new variable for an intermediate calculation, keeping previous range variables in scope. Useful for readability and reusing complex expressions.
*   **`Into`**: Continues a query from the *result* of a `select` or `group by` clause, effectively starting a new query scope and making previous range variables inaccessible. Useful for chaining operations on the transformed or grouped data.