### 1. Any
`Any` checks whether a sequence contains any elements or whether any element satisfies a specific condition.

- **Senior Note (Short-Circuiting):** `Any` is highly efficient because it returns `true` as soon as the first matching element is found. It does not iterate the entire collection unless necessary.
- **Performance Tip:** Always prefer `if (items.Any())` over `if (items.Count() > 0)`. While modern .NET optimizes `Count()` for `ICollection`, `Any()` is semantically clearer and safer for `IEnumerable` sources (like database queries or infinite streams).

```csharp
record Product(string Name, decimal Price, bool IsInStock);

Product[] inventory = [
    new("Laptop", 1200, true),
    new("Mouse", 25, true),
    new("Monitor", 300, false)
];

// 1. Check if collection is not empty
bool hasItems = inventory.Any(); 

// 2. Check with predicate
bool hasExpensiveItems = inventory.Any(p => p.Price > 1000);
```

---

### 2. All
`All` determines whether every element in a sequence satisfies a condition.

- **Senior Note (The Vacuous Truth):** A critical edge case is that **`All` returns `true` for empty collections**, regardless of the predicate. This is a mathematical concept known as "vacuous truth." Always check `Any() && All(...)` if you need to ensure the list isn't empty.
- **Short-Circuiting:** It returns `false` immediately upon finding the first element that does *not* satisfy the condition.

```csharp
// Returns true if ALL are in stock
bool allInStock = inventory.All(p => p.IsInStock);

// Senior Edge Case: Empty Collections
List<Product> emptyList = [];
bool result = emptyList.All(p => p.Price > 100); // Returns TRUE
```

---

### 3. Contains
`Contains` checks if a specific element exists in the collection.

- **Senior Note (Equality):** For reference types, `Contains` uses `EqualityComparer<T>.Default`. If you are using `class`, it checks reference equality. If you use `record`, it checks value equality.
- **Performance:** On a `List<T>`, `Contains` is $O(n)$. On a `HashSet<T>` or `Dictionary<K,V>`, it is $O(1)$. If you find yourself calling `.Contains()` inside a loop, consider converting your source to a `HashSet`.

```csharp
// Value equality with records
var mouse = new Product("Mouse", 25, true);
bool hasMouse = inventory.Contains(mouse); 

// Using IEqualityComparer (Modern C#)
bool hasLaptopCaseInsensitive = inventory
    .Select(p => p.Name)
    .Contains("laptop", StringComparer.OrdinalIgnoreCase);
```

---

### Comparison Summary

| Method | Logic | Short-circuits? | Empty Collection Result |
| :--- | :--- | :--- | :--- |
| **Any()** | Is there at least one element? | Yes (at 1st element) | `false` |
| **Any(pred)** | Does at least one match? | Yes (at 1st match) | `false` |
| **All(pred)** | Do all elements match? | Yes (at 1st mismatch) | `true` (Vacuous Truth) |
| **Contains(val)** | Does this specific value exist? | Yes (at 1st match) | `false` |

---

### Senior-Level Best Practices

- **Avoid Negation Confusion:** Instead of `!items.Any(p => !p.IsValid)`, use `items.All(p => p.IsValid)`. It is significantly more readable and maps better to natural language.
- **.NET 8/9 Optimizations:** In recent .NET versions, the LINQ implementation for `Any()` has been further optimized to check for `ICollection.Count` properties internally before attempting to get an enumerator, making it $O(1)$ for most concrete collections.
- **Database Context (EF Core):** When using these in EF Core, `Any` translates to `EXISTS` in SQL, and `All` translates to `NOT EXISTS(SELECT ... WHERE NOT condition)`. `Any` is generally more performant in SQL than counting rows.
- **Readability with Collection Expressions:** When checking against a fixed set of values, use the new C# 12 syntax:
  ```csharp
  string[] validStatuses = ["Active", "Pending", "Approved"];
  if (validStatuses.Contains(currentStatus)) { ... }
  ```
