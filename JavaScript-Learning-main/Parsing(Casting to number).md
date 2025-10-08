## Casting to integer
* To cast a string in Js to integer we could use parseint() method that accept any data type and convert it to integer if it possible.

```JavaScript
var x = parseInt("258.90");
console.log(x, typeof x); //258 'number'
```

* We have to know that even if the string that we want to parse to integer has a characters the parseInt() method will work.

```JavaScript
var str = parseInt("23.3abc");
console.log(str, typeof str); // 23 'number'
```

* As we saw in the previous example we it also work even with string with character.
* But we have to know how the parseInt() method work when the string contains characters?

- _The pareseInt() will start by checking the first character in the parameter, if it a number it goes to the next number and so on, until it find a decimal point or a character then it will stop and return the result._

- <span style="color:rgb(255, 255, 0)">What if the first char in the string was a letter or a decimal point?</span>
- Then it will stop directly and return NaN(Not a Number);

```JavaScript
var str = parseInt("a23.3abc");
console.log(str, typeof str); // NaN 'number'
```

- As we saw in this example as the string started with letter it returns NaN, because the parseInt() method didn't find number.
- We also have to know that NaN is a type of number, even if it Not a Number, but it's under the Number class. [[NaN(Not a Number)]].


------- 

## Casting to float
* To cast a string in Js to float we could use parseFloat() method that accept any data type and convert it to float if it possible.

```JavaScript
var x = parseFloat("258.90");
console.log(x, typeof x); //258.90 'number'
```

* We have to know that even if the string that we want to parse to float has a characters the parseFloat() method will work.

```JavaScript
var str = parseFloat("23.3abc");
console.log(str, typeof str); // 23.3 'number'
```

* As we saw in the previous example we it also work even with string with character.
* But we have to know how the parseFloat() method work when the string contains characters?

- <mark style="background: #FFB86CA6;">The parseFloat() work exactly as the parseInt() work.</mark>

```JavaScript
var str = parseFloat("a23.3abc");
console.log(str, typeof str); // NaN 'number'
```

- As we saw in this example as the string started with letter it returns NaN, because the parseFloat() method didn't find number.
- We also have to know that NaN is a type of number, even if it Not a Number, but it's under the Number class.


---

## Casting With Number()
- In addition to cast using parsing we could cast using the Number class.
- We have to know that casting using Number will cast to integer and float depend on the value that we want to cast.
- If it contains decimal point the Number will cast it to float, else it will cast it to int.
- <span style="color:rgb(255, 255, 0)">What is the different between casting using parsing and Number class?</span>
- _Casting with Number() is more restricted than parsing, as if the string that we want to cast contains any character or any thing except numbers it will return NaN._

```JavaScript
var str = Number("23.3");
console.log(str, typeof str); // 23.3 'number'
```

```JavaScript
var str = Number("23.3abc");
console.log(str, typeof str); // NaN 'number'
```

- Here it return NaN, because the string contains characters not only numbers.

- learn How to check NaN? [[NaN(Not a Number)]].


--- 
## The Unary `+` Operator for Numeric Conversion

The `+` operator, when used as a unary operator (i.e., preceding a single operand, not between two operands for addition), attempts to convert its operand into a number. It's essentially a _shorthand_ for `Number()`.

**Syntax:** `+value`

**How it works:**
When the unary `+` operator is applied to a value, JavaScript's internal `ToNumber` abstract operation is invoked. This operation follows specific rules to convert various data types into their numeric representation.

### Examples and Behavior

Let's look at how it behaves with different types, and then compare it to the methods you've already discussed in your note.

```javascript
// 1. Strings representing valid numbers (integers and floats)
console.log(+"123");        // 123 (number)
console.log(+"3.14");       // 3.14 (number)
console.log(+"-50");        // -50 (number)
console.log(+"0xFF");       // 255 (number, hexadecimal)
console.log(+"  100  ");    // 100 (number, trims whitespace)

// 2. Empty string
console.log(+"");           // 0 (number)

// 3. Booleans
console.log(+true);         // 1 (number)
console.log(+false);        // 0 (number)

// 4. null and undefined
console.log(+null);         // 0 (number)
console.log(+undefined);    // NaN (number)

// 5. Strings with non-numeric characters
console.log(+"123abc");     // NaN (number) - Crucial difference from parseInt/parseFloat
console.log(+"abc123");     // NaN (number)
console.log(+"3.14.15");    // NaN (number) - Only one decimal point allowed
```

### Comparison with `parseInt()`, `parseFloat()`, and `Number()`

Your note already covers `parseInt()`, `parseFloat()`, and `Number()`. Let's see how the unary `+` operator stacks up against them.

#### 1. vs. `parseInt()` and `parseFloat()`

The most significant difference here is how they handle strings that contain non-numeric characters.

*   **`parseInt()` / `parseFloat()`**: These methods are more "forgiving." They parse the string from left to right and stop at the first non-numeric character (or decimal point for `parseInt` if not explicitly handled by radix). They return the number parsed up to that point.
```javascript
console.log(parseInt("23.3abc"));   // 23
console.log(parseFloat("23.3abc")); // 23.3
console.log(parseInt("a23.3abc"));  // NaN
```

*   **Unary `+` operator**: This operator is much **stricter**. If the string contains *any* non-numeric characters (other than leading/trailing whitespace or a single decimal point), it will result in `NaN`. It expects the entire string to represent a valid number.
```javascript
console.log(+"23.3abc");   // NaN
console.log(+"a23.3abc");  // NaN
```

   This strictness can be a benefit or a drawback depending on your use case. If you expect a string to be *purely* numeric, `+` or `Number()` are better choices for validation. If you need to extract a number from a string that might have units or other text, `parseInt()`/`parseFloat()` are more suitable.

#### 2. vs. `Number()` (Constructor/Function)

The unary `+` operator behaves **identically** to the `Number()` function (when called as a function, not as a constructor with `new`).

```javascript
console.log(+"123");        // 123
console.log(Number("123")); // 123

console.log(+"23.3abc");    // NaN
console.log(Number("23.3abc")); // NaN

console.log(+null);         // 0
console.log(Number(null));  // 0

console.log(+undefined);    // NaN
console.log(Number(undefined)); // NaN
```
They both perform the same `ToNumber` conversion. The unary `+` is simply a more concise syntax for the same operation.

### Advantages of the Unary `+` Operator

1.  **Conciseness**: It's the shortest way to convert a value to a number.
2.  **Performance**: In many JavaScript engines, the unary `+` operator can be slightly faster than `Number()` because it's a primitive operation, though the difference is often negligible for most applications.
3.  **Strictness**: For scenarios where you expect a string to be a perfectly valid number, its strictness (returning `NaN` for non-numeric strings) acts as a form of validation.
4.  **TypeScript Compatibility**: TypeScript understands this operator for type conversion. If you have a `string` or `string | number` type, applying `+` will correctly narrow the type to `number`.

### Disadvantages

1.  **Readability for Beginners**: For developers new to JavaScript, `+value` might be less immediately obvious than `Number(value)`.
2.  **Strictness (as a drawback)**: If your input strings might contain non-numeric suffixes (like `"10px"`) and you still want to extract the numeric part, `parseInt()` or `parseFloat()` are necessary.

### TypeScript Implications

TypeScript understands the unary `+` operator as a type conversion mechanism.

```typescript
let priceString: string = "99.99";
let priceNumber: number = +priceString; // TypeScript correctly infers priceNumber as 'number'

let maybeNumber: string | null = "123";
let converted: number = +maybeNumber; // TypeScript infers 'number', but at runtime, if maybeNumber is null, converted will be 0.

let invalidString: string = "abc";
let result: number = +invalidString; // TypeScript allows this, result will be NaN at runtime.
```
TypeScript will correctly infer the type as `number` after the unary `+` operation. This is useful for type safety, as it ensures that subsequent operations on `converted` are treated as numeric.

### Conclusion

The unary `+` operator is a powerful, concise, and strict way to convert values to numbers in JavaScript. It behaves identically to `Number()` and is stricter than `parseInt()` or `parseFloat()`. As a senior developer, you should understand its behavior, especially its strictness with non-numeric strings, and choose it when conciseness and strict numeric validation are desired. For extracting numbers from mixed strings, `parseInt()` or `parseFloat()` remain the tools of choice.