## Infinity
- Infinity is a numeric value in Js, so we have different ways to check if the value is Infinity or not:

## ✅ 1. Using Built-in Methods

### a) Using `isFinite()` (global function)

- Returns `false` for `Infinity`, `-Infinity`, and `NaN`
    
- Returns `true` for finite numbers <mark style="background: Yellow;">(including numeric strings coerced to numbers)</mark>
    

```JavaScript
isFinite(Infinity);     // false 
isFinite(-Infinity);    // false
isFinite(123);          // true
isFinite("123");        // true (because of coercion)```
```


### b) Using `Number.isFinite()` (ES6+)

- Returns `false` for `Infinity`, `-Infinity`, `NaN`, **and** numeric strings (no coercion)
    
- Returns `true` only for finite numbers
    

```JavaScript
Number.isFinite(Infinity);     // false
Number.isFinite(-Infinity);    // false 
Number.isFinite(123);          // true 
Number.isFinite("123");        // false (no coercion)
```

---

## ✅ 2. Without Using Built-in Methods

### Approach:

- Check if type is `'number'`
    
- Check for `NaN` (using `value === value`, since only `NaN !== NaN`)
    
- Explicitly exclude `Infinity` and `-Infinity`
    

```JavaScript
function isFiniteNumber(value) {
  return (
    typeof value === 'number' &&
    value === value &&           // exclude NaN
    value !== Infinity &&        // exclude +Infinity
    value !== -Infinity          // exclude -Infinity
  );
}
```

### Why does this work?

- `typeof value === 'number'` ensures it's a number type (not string, null, etc.)
    
- `value === value` filters out `NaN` because only `NaN !== NaN`
    
- Explicit checks exclude the infinite values which are still considered numbers in JS

**🔎 <span style="color:rgb(0, 176, 80)">Summary Table</span><span style="color:rgb(0, 176, 80)">:</span>**

| Check Method                              | Returns `false` for `Infinity`? | Returns `false` for `-Infinity`? | Coerces strings? |
| ----------------------------------------- | ------------------------------- | -------------------------------- | ---------------- |
| `isFinite()`                              | Yes                             | Yes                              | Yes              |
| `Number.isFinite()`                       | Yes                             | Yes                              | No               |
| Custom check (`typeof + explicit checks`) | Yes                             | Yes                              | No               |


Check NaN [[NaN(Not a Number)]].