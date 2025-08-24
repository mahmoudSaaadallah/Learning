- In JavaScript, `==` (loose equality) and `===` (strict equality) are used to compare two values. The fundamental difference lies in how they handle **type coercion**.

🧠 Overview

| Operator | Name            | Performs Type Conversion? | Checks Value Equality | Checks Type Equality |
| -------- | --------------- | ------------------------- | --------------------- | -------------------- |
| `==`     | Loose Equality  | ✅ Yes                     | ✅ Yes                 | ❌ No                 |
| `===`    | Strict Equality | ❌ No                      | ✅ Yes                 | ✅                    |

--- 

## ✅ `===` (Strict Equality)
The strict equality operator compares two values for equality **without performing any type conversion (coercion)**.

**Key Characteristics:**

1. **Compares Type First:** It first checks if the two values have the same data type.
2. **No Type Coercion:** If the types are different, it immediately returns `false` without attempting to convert either value.
3. **Compares Value Second:** If the types are the same, it then compares the values.

**Algorithm (Simplified):**

- If `Type(x)` is different from `Type(y)`, return `false`.
- If `Type(x)` is the same as `Type(y)`, compare their values:
    - **Numbers:** `x` and `y` are equal if they have the same numeric value. Special cases: `NaN` is not equal to anything, including itself. `+0` and `-0` are considered equal.
    - **Strings:** `x` and `y` are equal if they have the same sequence of characters.
    - **Booleans:** `x` and `y` are equal if they are both `true` or both `false`.
    - **`null` and `undefined`:** `null` is only strictly equal to `null`. `undefined` is only strictly equal to `undefined`.
    - **Objects (including arrays, functions):** `x` and `y` are equal if they refer to the _exact same object in memory_. They are compared by reference, not by value.
    - **Symbols:** `x` and `y` are equal if they refer to the _exact same Symbol value_.
    - **BigInts:** `x` and `y` are equal if they have the same numeric value.

**Examples and Edge Cases for `===`:**

| Expression                         | Result  | Explanation                                                                                                                              |
| :--------------------------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `1 === 1`                          | `true`  | Same type (Number), same value.                                                                                                          |
| `1 === '1'`                        | `false` | Different types (Number vs. String).                                                                                                     |
| `true === 1`                       | `false` | Different types (Boolean vs. Number).                                                                                                    |
| `false === 0`                      | `false` | Different types (Boolean vs. Number).                                                                                                    |
| `null === undefined`               | `false` | Different types (`null` vs. `undefined`).                                                                                                |
| `null === null`                    | `true`  | Same type, same value.                                                                                                                   |
| `undefined === undefined`          | `true`  | Same type, same value.                                                                                                                   |
| `NaN === NaN`                      | `false` | **Crucial Edge Case:** `NaN` is the only value in JS not strictly equal to itself. Use `Number.isNaN()` or `x !== x` to check for `NaN`. |
| `0 === -0`                         | `true`  | Same type, same numeric value.                                                                                                           |
| `Infinity === Infinity`            | `true`  | Same type, same numeric value.                                                                                                           |
| `{a: 1} === {a: 1}`                | `false` | Different objects in memory (compared by reference).                                                                                     |
| `const obj = {a: 1}; obj === obj;` | `true`  | Same object reference.                                                                                                                   |
| `[] === []`                        | `false` | Different array objects in memory.                                                                                                       |
| `Symbol('a') === Symbol('a')`      | `false` | Different Symbol values (each `Symbol()` call creates a unique symbol).                                                                  |
| `1n === 1`                         | `false` | Different types (BigInt vs. Number).                                                                                                     |

--- 

### `==` (Loose Equality Operator)

The loose equality operator compares two values for equality **after performing type conversion (coercion)** if their types are different.

**Key Characteristics:**

1. **Performs Type Coercion:** If the types of the two values are different, JavaScript attempts to convert one or both values to a common type before making the comparison.
2. **Complex Coercion Rules:** The rules for type coercion can be complex and sometimes counter-intuitive.
3. **Compares Value:** After (potential) coercion, it compares the values.

**Algorithm (Simplified, but the actual algorithm is more detailed):**

- If `Type(x)` is the same as `Type(y)`, compare their values (same as `===`).
- If `x` is `null` and `y` is `undefined`, return `true`.
- If `x` is `undefined` and `y` is `null`, return `true`.
- If `Type(x)` is Number and `Type(y)` is String, convert `y` to Number and compare.
- If `Type(x)` is String and `Type(y)` is Number, convert `x` to Number and compare.
- If `Type(x)` is Boolean, convert `x` to Number (`true` becomes `1`, `false` becomes `0`) and compare.
- If `Type(y)` is Boolean, convert `y` to Number (`true` becomes `1`, `false` becomes `0`) and compare.
- If `Type(x)` is Object and `Type(y)` is String or Number or Symbol, convert `x` to a primitive value (using `valueOf()` or `toString()`) and compare.
- If `Type(x)` is BigInt and `Type(y)` is Number, convert both to Number (if possible without loss of precision) or BigInt and compare.
- ...and many more specific rules.

**Examples and Edge Cases for `==`:**

|Expression|Result|Explanation|
|:--|:--|:--|
|`1 == 1`|`true`|Same type, same value.|
|`1 == '1'`|`true`|String `'1'` is coerced to Number `1`. `1 == 1`.|
|`true == 1`|`true`|Boolean `true` is coerced to Number `1`. `1 == 1`.|
|`false == 0`|`true`|Boolean `false` is coerced to Number `0`. `0 == 0`.|
|`null == undefined`|`true`|**Special Rule:** `null` and `undefined` are loosely equal to each other, but not to any other value.|
|`null == 0`|`false`|`null` is only loosely equal to `null` or `undefined`.|
|`undefined == 0`|`false`|`undefined` is only loosely equal to `null` or `undefined`.|
|`NaN == NaN`|`false`|**Crucial Edge Case:** Same as `===`, `NaN` is not equal to itself.|
|`0 == -0`|`true`|Same type, same numeric value.|
|`Infinity == Infinity`|`true`|Same type, same numeric value.|
|`'' == 0`|`true`|Empty string `''` is coerced to Number `0`. `0 == 0`.|
|`' ' == 0`|`true`|String with whitespace `' '` is coerced to Number `0`. `0 == 0`.|
|`[] == 0`|`true`|Array `[]` is coerced to empty string `''`, then `''` is coerced to `0`. `0 == 0`.|
|`[] == ''`|`true`|Array `[]` is coerced to empty string `''`. `'' == ''`.|
|`[1] == '1'`|`true`|Array `[1]` is coerced to string `'1'`. `'1' == '1'`.|
|`[1] == 1`|`true`|Array `[1]` is coerced to string `'1'`, then `'1'` is coerced to `1`. `1 == 1`.|
|`{} == {}`|`false`|**Objects are compared by reference, even with `==`.** No coercion for the objects themselves, only if one side is a primitive.|
|`Symbol('a') == Symbol('a')`|`false`|Symbols are unique, no coercion.|
|`1n == 1`|`true`|BigInt `1n` is coerced to Number `1`. `1 == 1`.|

---

### When to Use Which (Best Practices)

- **Prefer `===` (Strict Equality):**
    
    - **General Recommendation:** In almost all cases, it is best practice to use `===`. It makes your code more predictable, easier to read, and less prone to subtle bugs caused by unexpected type coercion.
    - **Clarity:** It clearly states that you intend to compare both the value and the type.

- **When `==` (Loose Equality) might be acceptable (with caution):**
    
    - **Checking for `null` or `undefined`:** A common idiom is `value == null`. This expression will return `true` if `value` is either `null` or `undefined`, and `false` otherwise. This can be a concise way to check for both, but some prefer the more explicit `value === null || value === undefined`.
    - **Legacy Code:** You might encounter it in older codebases.
    - **Specific Scenarios:** Very occasionally, you might have a specific scenario where you _intend_ for type coercion to happen and you fully understand its implications. However, these cases are rare and often better handled with explicit type conversions.

- It's better to not use == or === to check the NaN and Infinity value instead there are other different ways to compare NaN and Infinity
  check [[NaN(Not a Number)]].
  check [[Check The Infinity Values]].