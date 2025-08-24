## Getting specific substring
- To get a substring we could use two different ways:

### 1. Using substring()
- substring(): is Method that belongs to String class that used to get specific substring.
- **Parameters:**
    - `startIndex`: The index of the first character to include.
    - `endIndex` (optional): The index of the first character to _exclude_. The `endIndex` will not be included.
    - If omitted, `substring()` extracts to the end of the string.

- **Behavior with Arguments:**
    1. **Negative Arguments:** Any negative argument is treated as `0`.
    2. **`startIndex` > `endIndex`:** If `startIndex` is greater than `endIndex`, `substring()` swaps the two arguments internally. It effectively treats the smaller number as the `startIndex` and the larger number as the `endIndex`.
    3. **`NaN` or non-numeric arguments:** Treated as `0`.

```JavaScript
const str = "Hello World";

console.log(str.substring(0, 5));   // "Hello" (from index 0 up to 5)
console.log(str.substring(6));      // "World" (from index 6 to the end)
console.log(str.substring(5, 0));   // "Hello" (arguments 5 and 0 are swapped internally to 0, 5)
console.log(str.substring(-3, 5));  // "Hello" (negative -3 is treated as 0, so 0, 5)
console.log(str.substring(3, 1));   // "ll" (arguments 3 and 1 are swapped internally to 1, 3)
console.log(str.substring(1, 3));   // "ll"
```

### 2. Using slice()
- slice(): is Method that belongs to String class too.
- **Purpose:** Extracts characters from string.
- **Parameters:**
    - `startIndex`: The index of the first character to include.
    - `endIndex` (optional): The index of the first character to _exclude_. If omitted, `slice()` extracts to the end of the string.
- **Behavior with Arguments:**
    1. **Negative Arguments:** Negative arguments are treated as an offset from the end of the string.
        - `slice(-1)` extracts the last character.
        - `slice(-3)` extracts the last three characters.
        - `slice(0, -1)` extracts from the beginning up to the last character (excluding the last).
    2. **`startIndex` > `endIndex`:** If `startIndex` is greater than `endIndex`, `slice()` returns an empty string (`""`). It does _not_ swap the arguments.
    3. **`NaN` or non-numeric arguments:** Treated as `0`.

```JavaScript
const str = "Hello World";

console.log(str.slice(0, 5));    // "Hello" (from index 0 up to 5)
console.log(str.slice(6));       // "World" (from index 6 to the end)
console.log(str.slice(-5));      // "World" (last 5 characters)
console.log(str.slice(0, -1));   // "Hello Worl" (from beginning up to the second to last character)
console.log(str.slice(-5, -1));  // "Worl" (from 5th from end up to 1st from end)
console.log(str.slice(5, 0));    // "" (startIndex 5 is greater than endIndex 0)
console.log(str.slice(3, 1));    // "" (startIndex 3 is greater than endIndex 1)
```

### When to Use Which?

- **Use `slice()`** when you want to use negative indices to count from the end of the string, or when you want a strict behavior where `startIndex` greater than `endIndex` results in an empty string. It's generally considered more predictable and flexible for common use cases.
- **Use `substring()`** if you prefer its argument-swapping behavior when `startIndex` is greater than `endIndex`, or if you want negative indices to be treated as `0`. This can sometimes be more forgiving if you're not sure about the order of your indices.

In most modern JavaScript development, `slice()` is more commonly used due to its more intuitive handling of negative indices and its stricter behavior regarding argument order.