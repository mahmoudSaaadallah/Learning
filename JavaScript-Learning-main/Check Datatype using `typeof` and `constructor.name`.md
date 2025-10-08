
- When dealing with datatypes in JavaScript we  sometimes need to check the type of the variables to take some action.
- To get the type of the variable we could use `typeof` prop.

### 1. `typeof` Operator

The `typeof` operator returns a string indicating the _type_ of its unevaluated operand. It's primarily designed for checking primitive types.

**Syntax:** `typeof operand`

**What it returns:**

- `"undefined"`: For `undefined`
- `"boolean"`: For `true` or `false`
- `"number"`: For numbers (integers, floats, `NaN`, `Infinity`)
- `"string"`: For strings
- `"symbol"`: For Symbols (ES6+)
- `"bigint"`: For BigInts (ES11+)
- `"function"`: For functions (this is a special case; functions are objects but `typeof` gives 'function')
- `"object"`: For `null`, arrays, plain objects, `Date` objects, `RegExp` objects, and instances of custom classes.

**Examples:**

```javascript
console.log(typeof 42);             // "number"
console.log(typeof "hello");        // "string"
console.log(typeof true);           // "boolean"
console.log(typeof undefined);      // "undefined"
console.log(typeof Symbol('id'));   // "symbol"
console.log(typeof 10n);            // "bigint"
console.log(typeof function() {});  // "function"
console.log(typeof {});             // "object"
console.log(typeof []);             // "object"
console.log(typeof new Date());     // "object"
console.log(typeof null);           // "object" - The infamous "billion-dollar mistake"
```

**Strengths of `typeof`:**

- **Reliable for Primitives**: It's the most straightforward and reliable way to check for primitive types (`string`, `number`, `boolean`, `symbol`, `bigint`, `undefined`).
- **Identifies Functions**: It correctly distinguishes functions from other objects.
- **Fast**: It's a built-in operator, so it's generally very fast.

**Limitations of `typeof`:**

- **`null` is "object"**: This is the biggest gotcha. `typeof null` returns `"object"`, which is misleading and requires an additional `!== null` check when trying to identify actual objects.
- **Doesn't Distinguish Object Types**: For all non-function objects (arrays, dates, custom class instances, plain objects), `typeof` simply returns `"object"`. It cannot tell you if something is an array, a date, or a specific class instance.
- Also when we create variable using `new` key word the `typeof` always return `"object"` without specifying which type of objects.
```JavaScript
var num = new Number(10);
console.log(typeof num); // 'object'

if(typeof num == 'number'){
	console.log("num variable is Number"); // This will never execute
}
```

- In this example it returns `"object"` not `"Number"` which is actually the type of the variable, but `typeof` dealing with any instance or variable initialized with `new` keyword as an `object`.

- To solve this situation we don't use `typeof` to check the type, instead we use `constructor.name`.

---
### 2. `constructor.name` Property

Every JavaScript object (except for `null` and `undefined`) has a `constructor` property, which points to the constructor function that created the object. The `name` property of a function object typically holds the name of that function.

**Syntax:** `value.constructor.name`

**What it returns:** A string representing the name of the constructor function.

**Examples:**

```javascript
console.log((42).constructor.name);             // "Number"
console.log(("hello").constructor.name);        // "String"
console.log((true).constructor.name);           // "Boolean"
// console.log(undefined.constructor.name);     // TypeError: Cannot read properties of undefined (reading 'constructor')
// console.log(null.constructor.name);          // TypeError: Cannot read properties of null (reading 'constructor')
console.log(new Number(10).construcor.name)     // "Number"
console.log(Symbol('id').constructor.name);     // "Symbol"
console.log((10n).constructor.name);            // "BigInt"
console.log((function() {}).constructor.name);  // "Function"
console.log({}.constructor.name);               // "Object"
console.log([].constructor.name);               // "Array"
console.log(new Date().constructor.name);       // "Date"

class MyClass {}
const myInstance = new MyClass();
console.log(myInstance.constructor.name);       // "MyClass"
```

**Strengths of `constructor.name`:**

- **Distinguishes Object Types**: It can differentiate between various built-in object types (`Array`, `Date`, `RegExp`, `Object`) and custom class instances.
- **More Specific**: Provides a more granular type identification for objects than `typeof`.

**Limitations of `constructor.name`:**

- **Fails for `null` and `undefined`**: Accessing `.constructor` on `null` or `undefined` will throw a `TypeError`. You must always check for these first.
- **Doesn't Work for Primitives Directly**: While `(42).constructor.name` works, it's because JavaScript temporarily "boxes" the primitive into its object wrapper (`Number` object) to access the property. This is less direct and can be confusing.
- **Can Be Minified/Obfuscated**: In production builds, especially with minification, constructor function names can be changed (e.g., `MyClass` might become `a` or `t`). This makes `constructor.name` unreliable for cross-module or long-term type checking.
- **Can Be Spoofed**: The `constructor` property can be reassigned, or an object can be created without a standard constructor chain (e.g., `Object.create(null)`), making `constructor.name` unreliable in untrusted environments or when dealing with maliciously crafted objects.
- **Inheritance Issues**: If you have a class `Child` extending `Parent`, `new Child().constructor.name` will be `"Child"`, not `"Parent"`. If you need to check if an object is an instance of a particular class _or any of its ancestors_, `instanceof` is better.

### Key Differences and When to Use Which

|Feature|`typeof`|`constructor.name`|
|:--|:--|:--|
|**Primary Use**|Checking primitive types|Distinguishing specific object/class instances|
|**Return Value**|String (e.g., "number", "object")|String (e.g., "Number", "Array", "MyClass")|
|**`null` behavior**|Returns `"object"` (bug)|Throws `TypeError`|
|**`undefined` behavior**|Returns `"undefined"`|Throws `TypeError`|
|**Primitives**|Direct and reliable|Requires boxing, less direct|
|**Functions**|Returns `"function"`|Returns `"Function"`|
|**Objects (general)**|Returns `"object"` (generic)|Returns specific constructor name (e.g., "Array")|
|**Reliability**|High for primitives, low for objects|Moderate (can be spoofed, minified, fails on null/undefined)|
|**Performance**|Very fast|Generally fast, but involves property lookup|

**When to use `typeof`:**

- To check if a value is a primitive (`string`, `number`, `boolean`, `symbol`, `bigint`, `undefined`).
- To check if a value is a function.
- As a first-pass check to ensure a value is not `null` or `undefined` before attempting to access properties (though `if (value)` or `value !== null && value !== undefined` is often clearer).

**When to use `constructor.name`:**

- When you need to differentiate between specific built-in object types (like `Array`, `Date`, `RegExp`) _and you are certain the object is not `null` or `undefined`_.
- When you need to identify instances of custom classes _in a development environment where minification hasn't obfuscated names_, and you're not concerned about spoofing.

### Better Practices for Type Checking (Senior Dev Perspective)

Given the limitations, senior developers often rely on more robust and explicit methods for type checking:

1. **Strict Equality for `null` and `undefined`**:
    
    ```javascript
    if (value === null) { /* ... */ }
    if (value === undefined) { /* ... */ }
    if (value == null) { /* checks for both null and undefined */ }
    ```
    
2. **`Array.isArray()` for Arrays**:  
    This is the most reliable way to check for arrays.
    
    ```javascript
    if (Array.isArray(value)) { /* ... */ }
    ```
    
3. **`instanceof` for Class Instances**:  
    This checks if an object is an instance of a specific class or any class in its prototype chain. It's reliable for custom classes and built-in types that are constructors.
    
    ```javascript
    class MyCustomError extends Error {}
    const err = new MyCustomError("Something happened");
    
    if (err instanceof MyCustomError) { /* ... */ } // true
    if (err instanceof Error) { /* ... */ }         // true
    if (new Date() instanceof Date) { /* ... */ }   // true
    ```
    
    **Limitation**: `instanceof` doesn't work across different JavaScript realms (e.g., iframes, web workers) because each realm has its own global objects and constructors.
    
4. **`Object.prototype.toString.call()` for Built-in Types**:  
    This is the most robust way to get the internal `[[Class]]` property of an object, which reliably identifies built-in types.
    
    ```javascript
    function getType(value) {
      return Object.prototype.toString.call(value).slice(8, -1);
    }
    
    console.log(getType({}));           // "Object"
    console.log(getType([]));           // "Array"
    console.log(getType(new Date()));   // "Date"
    console.log(getType(null));         // "Null"
    console.log(getType(undefined));    // "Undefined"
    console.log(getType(42));           // "Number"
    console.log(getType("hello"));      // "String"
    console.log(getType(function(){})); // "Function"
    ```
    
    This method is generally considered the most reliable for identifying the "true" type of built-in objects.
    
5. **TypeScript Type Guards**:  
    In TypeScript, the compiler helps you with type narrowing, often making explicit runtime checks less necessary or more focused.
    
    ```typescript
    function processInput(input: string | number | MyClass) {
      if (typeof input === 'string') {
        // input is string
      } else if (typeof input === 'number') {
        // input is number
      } else if (input instanceof MyClass) {
        // input is MyClass
      }
    }
    ```
    

### Conclusion

`typeof` is excellent for primitives and functions. `constructor.name` can be useful for distinguishing specific object types in controlled environments but comes with significant caveats regarding `null`/`undefined`, minification, and spoofing. For robust, production-grade type checking in JavaScript, especially for objects, prefer `Array.isArray()`, `instanceof`, or `Object.prototype.toString.call()`. In TypeScript, leverage the type system and type guards to achieve compile-time safety, reducing the need for many runtime checks.