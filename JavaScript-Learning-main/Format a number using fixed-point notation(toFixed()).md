## Format Numbers
- To format a number depend on the fixed-point we have several ways to do that:

### Using toFixed()
- toFixed(): is a method that we use to format a number using **<mark style="background: #D2B3FFA6;">fixed-point notation</mark>**.
- This method return the result as a string not a number representing the number with a specified number of digits after the decimal point. So we have to parse it to number if we want to work with it as a number.
  check the parsing(Cast to number)[[Parsing(Casting to number)]]

-  The toFixed() method accepts an int that represents the number of digits that we want to get after the decimal point.
- Also it round the number.

```JavaScript
var num = 123.457;
var fixed = num.toFixed(2);
console.log(fixed, typeof fixed); // 123.46 string
```

- As we can see in the example above the result has been rounded because the third number after the decimal point is greater than 5, and result also is string.

```JavaScript
let num = 3.14159;
console.log(num.toFixed());     // "3"
console.log(num.toFixed(2));    // "3.14"
console.log(num.toFixed(4));    // "3.1416"  (rounded)
console.log(num.toFixed(6));    // "3.141590" (adds trailing zero)
```

##### Edge Cases
```JavaScript
(2.5).toFixed(0);     // "3"
(2.49).toFixed(0);    // "2"
(12345.6789).toFixed(2); // "12345.68"
(0.1 + 0.2).toFixed(2);  // "0.30" (floating point quirk handled)
```

---

### Using toPrecision()
- toPrecision(): used for formatting numbers with a **specific number of _significant digits_** — not just decimal places.
#### ✅ What it does?
- Formats a number to a specified **<mark style="background: #D2B3FFA6;">number of significant digits</mark>**, regardless of whether those digits are before or after the decimal point.
- So the toPrescision() method accepts an int number which represents the number of numbers that we want to return as whole with and without decimal point.
- It also round the number if it need to be rounded.
- It return string exactly like the toFixed() method.
- Check casting to number [[Parsing(Casting to number)]] 

```JavaScript
let num = 123.456;

// 3 significant digits
console.log(num.toPrecision(3)); // "123"

// 5 significant digits
console.log(num.toPrecision(5)); // "123.46" (rounded)

// 7 significant digits
console.log(num.toPrecision(7)); // "123.4560"
```

- If we try to use this method with number less than the number of digits before the decimal point it will return a weird result like:

```JavaScript
let num = 12345.678;
console.log(num.toPrecision(1)); // "1e+4"
console.log(num.toPrecision(2)); // "1.2e+4"
console.log(num.toPrecision(3)); // "1.23e+4"
console.log(num.toPrecision(4)); // "1.235e+4"
console.log(num.toPrecision(5)); // "12346"
```

#### 🔍 Why does `num.toPrecision(1)` return `"1e+4"`?

Because:
- `toPrecision(1)` means: "Show this number with **only 1 significant digit**."
    
- The number `12345.678` has **five digits before the decimal point**, so to keep **only one significant digit**, JavaScript **rounds it to 10000**.
    
- But instead of displaying `10000`, it uses **scientific notation**:  
    👉 `1 × 100000` → `"1e+4"`