## Type:
- NaN: refers to Not a Number.
- Even with this name its type is number type.

---
## 🔎How to check NaN?
- To check the NaN value we use isNaN() method which return true if the value is Not a Number and false if it's a number.

 ```JavaScript
 console.log(isNaN("abc")); // true
 ```

- There are some cases where the isNaN() method return false even with NaN values like:
```JavaScript
console.log(isNaN(null)); // false, because null is coerced to 0 (a number)
console.log(isNaN('123')); // false, because '123' can be converted to a number
```

- As we can the isNaN() method treat null as a number.
- The reason `console.log(isNaN(null));` outputs `false` is due to how the `isNaN()` function works internally, specifically its **type coercion** rules.

Here's the breakdown:
1. **`isNaN()`'s Coercion Rule:** The `isNaN()` function first attempts to convert its argument to a number. If the argument cannot be converted to a number, or if it's already `NaN`, then `isNaN()` returns `true`. Otherwise, it returns `false`.
    
2. **`null` to Number Coercion:** When `null` is coerced to a number in JavaScript, it becomes `0`.
    
    - You can see this by trying `Number(null)` in your console, which will output `0`.
    - Similarly, `null + 0` results in `0`.
- This is the reason of why `console.log(isNaN('123');` outputs `false` because the `'123'` could be converted to number.
- Check parsing(Casting to number) [[Parsing(Casting to number)]].

##### 🤔What if try to check `NaN == NaN`?
- t's **one of the strangest and most infamous quirks** in JavaScript (and many other programming languages based on IEEE 754 floating-point math):
- This will return **<mark style="background: #D2B3FFA6;">false</mark>** this is because `NaN` (Not-a-Number) is **defined** to be **not equal to anything**, **including itself**.
- Even `NaN === NaN` will be `false`.

- ✅ Summary


| Check                 | Result                      |
| --------------------- | --------------------------- |
| `NaN == NaN`          | `false` ❌                   |
| `NaN === NaN`         | `false` ❌                   |
| `Number.isNaN(NaN)`   | `true` ✅                    |
| `Number.isNaN('abc')` | `false` ✅(doesn't coerce)   |
| `isNaN('abc')`        | `true` ❌ (coerces to NaN)\| |

- So to check about the `NaN` itself not about its meaning we could use `Number.isNaN()`