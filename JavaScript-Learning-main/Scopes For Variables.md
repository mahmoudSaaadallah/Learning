### What is Scope?

In JavaScript, **scope** determines the accessibility of variables, functions, and objects in some particular part of your code during runtime. It dictates where a variable can be referenced and used. Understanding scope is crucial for preventing naming collisions, managing memory, and writing modular code.

JavaScript primarily has three types of scope:

1.  **Global Scope**
2.  **Function Scope** (for `var`)
3.  **Block Scope** (for `let` and `const`)

There's also the concept of **Lexical Scope**, which describes how nested functions resolve variables from their outer environments.

---

### 1. Global Scope

Variables declared in the global scope are accessible from anywhere in your JavaScript code, including within functions and blocks.

*   **Declaration**: A variable declared outside of any function or block.
*   **Accessibility**: Globally accessible.
*   **Caveat**: In browsers, _global variables become properties of the `window` object_. In Node.js, they become properties of the `global` object. This is often referred to as "polluting the global namespace" and is generally considered bad practice as it can lead to naming conflicts and make code harder to reason about.
* we could create a global scope variable using `var` outside function or without it, but _when we declare a variable without `var` we have to initialize it in the same line. _
* When we create variable without `var` inside a function we have to call the function before using this variable outside it to make sure that the engine has created the variable.

**Example:**

```javascript
var globalVar = "I'm global (var)";
let globalLet = "I'm global (let)";
const globalConst = "I'm global (const)";

function accessGlobals() {
  console.log(globalVar);   // "I'm global (var)"
  console.log(globalLet);   // "I'm global (let)"
  console.log(globalConst); // "I'm global (const)"
}

variableWithoutVar = 20;
console.log(variableWithoutVar);
accessGlobals();
console.log(globalVar);     // "I'm global (var)"

function accessGlobals2() {
	variableWithoutVar2 = 12;
}
console.log(variableWithoutVar2) // return ReferenceError as the variable hasn't been created yet.
accessGlobals2()
console.log(variableWithoutVar2) // 12
```
**Best Practice**: Minimize the use of global variables. Encapsulate your code within modules or functions to keep variables in a more restricted scope.

---

### 2. Function Scope (`var`)

Variables declared with `var` are **function-scoped**. This means they are accessible anywhere within the function in which they are declared, regardless of any block statements (like `if` or `for` loops) inside that function.

*   **Declaration**: Using the `var` keyword inside a function.
*   **Accessibility**: Within the entire function.
*   **Hoisting**: `var` declarations are "hoisted" to the top of their function scope. This means the declaration is processed before any code is executed, but the assignment remains in place. The variable is initialized with `undefined` until its actual assignment.

**Example:**

```javascript
function demonstrateVarScope() {
  if (true) {
    var functionScopedVar = "I'm inside an if block";
    console.log(functionScopedVar); // "I'm inside an if block"
  }
  console.log(functionScopedVar); // "I'm inside an if block" - Still accessible outside the if block!

  for (var i = 0; i < 3; i++) {
    // ...
  }
  console.log(i); // 3 - 'i' is still accessible outside the loop!
}

demonstrateVarScope();
// console.log(functionScopedVar); // Error: functionScopedVar is not defined (correctly not global)

// Hoisting example:
function hoistingExample() {
  console.log(hoistedVar); // undefined (declaration is hoisted, but not assignment)
  var hoistedVar = "I was hoisted";
  console.log(hoistedVar); // "I was hoisted"
}
hoistingExample();
```

**Issues with `var` (Why we avoid it):**

*   **Lack of Block Scope**: This is the biggest problem. Variables "leak" out of `if` statements, `for` loops, and `while` loops, leading to unexpected behavior and bugs (especially in closures within loops).
*   **Redeclaration**: `var` allows you to redeclare the same variable multiple times in the same scope without an error, which can lead to accidental overwrites.
    ```javascript
    var x = 10;
    var x = 20; // No error, x is now 20
    console.log(x);
    ```

---

### 3. Block Scope (`let` and `const`) - Variables without `var`

Introduced in ES6 (ECMAScript 2015), `let` and `const` provide **block scope**. This is a significant improvement over `var` and is the standard for modern JavaScript and TypeScript development.

*   **Declaration**: Using `let` or `const` inside any block of code (defined by `{}`). This includes `if` blocks, `for` loops, `while` loops, `try...catch` blocks, and functions.
*   **Accessibility**: Only within the block where they are declared.
*   **Temporal Dead Zone (TDZ)**: Unlike `var`, `let` and `const` declarations are *not* initialized to `undefined` during hoisting. They are "hoisted" to the top of their block, but they remain in a "Temporal Dead Zone" until their declaration line is executed. Accessing them before declaration results in a `ReferenceError`.

#### `let`

*   **Mutable**: Variables declared with `let` can be reassigned to a new value.
*   **No Redeclaration**: You cannot redeclare a `let` variable in the same scope.

**Example:**

```javascript
function demonstrateLetScope() {
  let blockScopedLet = "I'm outside any specific block";
  console.log(blockScopedLet); // "I'm outside any specific block"

  if (true) {
    let innerLet = "I'm inside an if block";
    console.log(innerLet); // "I'm inside an if block"
    // let blockScopedLet = "New value"; // Error: Cannot redeclare block-scoped variable 'blockScopedLet'.
    blockScopedLet = "I've been reassigned"; // Valid: Reassignment is allowed
  }
  // console.log(innerLet); // Error: innerLet is not defined (correctly block-scoped!)

  for (let j = 0; j < 3; j++) {
    // 'j' is only accessible within this loop block
    console.log(`Loop j: ${j}`);
  }
  // console.log(j); // Error: j is not defined (correctly block-scoped!)
}

demonstrateLetScope();

// TDZ example:
function tdzExample() {
  // console.log(tdzLet); // ReferenceError: Cannot access 'tdzLet' before initialization
  let tdzLet = "I'm out of TDZ";
  console.log(tdzLet); // "I'm out of TDZ"
}
tdzExample();
```

#### `const`

*   **Immutable Reference**: Variables declared with `const` must be initialized at the time of declaration and **cannot be reassigned** to a new value.
*   **No Redeclaration**: Like `let`, you cannot redeclare a `const` variable in the same scope.
*   **Important Nuance**: While the *reference* itself is immutable, the *contents* of objects or arrays declared with `const` *can* be modified.

**Example:**

```javascript
function demonstrateConstScope() {
  const PI = 3.14159;
  console.log(PI); // 3.14159

  // PI = 3.14; // Error: Assignment to constant variable.

  const user = {
    name: "Alice",
    age: 30
  };
  console.log(user); // { name: "Alice", age: 30 }

  user.age = 31; // Valid: The *contents* of the object can be modified
  console.log(user); // { name: "Alice", age: 31 }

  // user = { name: "Bob", age: 25 }; // Error: Assignment to constant variable. (Cannot reassign the 'user' reference)

  if (true) {
    const blockConst = "I'm inside a block";
    console.log(blockConst);
  }
  // console.log(blockConst); // Error: blockConst is not defined (correctly block-scoped)
}

demonstrateConstScope();

// Must be initialized:
// const uninitializedConst; // Error: 'const' declarations must be initialized.
```

---

### 4. Lexical Scope (Closures)

Lexical scope (also known as static scope) means that the scope of a variable is determined by its position in the source code at the time of writing, not at runtime. This is how JavaScript's **closures** work.

A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In simpler terms, an inner function remembers and can access variables from its outer (enclosing) function's scope, even after the outer function has finished executing.

**Example:**

```javascript
function outerFunction(outerVar) {
  let innerVar = "I'm from inner function";

  function innerFunction(param) {
    console.log(outerVar); // Accesses outerFunction's 'outerVar'
    console.log(innerVar); // Accesses outerFunction's 'innerVar'
    console.log(param);    // Accesses innerFunction's 'param'
  }

  return innerFunction;
}

const myClosure = outerFunction("I'm from outer function");
myClosure("I'm a parameter to inner function");
// Output:
// "I'm from outer function"
// "I'm from inner function"
// "I'm a parameter to inner function"

// Even after outerFunction has finished, myClosure still "remembers" outerVar and innerVar.
```

---

### Best Practices for Variable Declaration (Modern JavaScript/TypeScript)

As a senior developer, your default approach should be:

1.  **Always use `const` by default.** This promotes immutability, makes your code more predictable, and helps prevent accidental reassignments.
2.  **Only use `let` if you explicitly need to reassign the variable.** If a variable's value needs to change over its lifetime (e.g., a counter in a loop, a mutable state variable), then `let` is appropriate.
3.  **Avoid `var` entirely.** There are virtually no modern use cases where `var` is superior to `let` or `const`, and its quirks (especially lack of block scope and redeclaration) lead to more bugs.

### TypeScript's Role

TypeScript enforces these JavaScript scoping rules at compile time. It will give you errors for:

*   Accessing `let` or `const` variables before their declaration (TDZ errors).
*   Redeclaring `let` or `const` variables in the same scope.
*   Assigning to a `const` variable after its initial declaration.
*   Using `const` without initialization.

This compile-time feedback is invaluable, catching potential runtime errors related to scope and variable usage before your code even runs.

By consistently applying `let` and `const` and understanding their block-scoping behavior, you'll write cleaner, safer, and more robust JavaScript and TypeScript applications.