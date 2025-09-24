## 1.Array

- Arrays in JavaScript are ordered, zero-indexed collections of data. They are a special type of object used to store multiple values in a single variable.
- **Can Hold Mixed Data Types**: A single array can contain elements of different data types (numbers, strings, booleans, objects, other arrays, etc.).
- **Arrays are Objects**: In JavaScript, arrays are objects, meaning they have properties and methods. They inherit from `Array.prototype`.
- **Mutable**: You can change the elements of an array after it's created.


--- 

## 2. Creating Arrays

There are two primary ways to create arrays:

**a) Array Literal (Most Common)**  
This is the simplest and most preferred way.
```JavaScript
// An Empty Array.
let array = [];

// An array with elements
let fruits = ["Apple", "Banana", "Cherry"];

// An array with mixed data types
let mixedArray = [1, "hello", true, { key: "value" }, null];
```

**b) `Array` Constructor**  
Less common, especially for creating arrays with initial elements, but useful for creating an array of a specific length.

```javascript
// An empty array
let arr = new Array(); 

let arr2 = new Array(5); // Creates an array of length 5 with undefined values.

let arr3 = new Array("Red", "Green", "Blue");
let arr4 = new Array(1, 2, 3);

```


--- 

## 3. Accessing Elements
```JavaScript
let colors = ["Red", "Green", "Blue"];
console.log(colors[0]); // Red.
console.log(colors[1]); // Green
```

----

## 4. Modifying Elements
You can change an element's value by assigning a new value to its index.

```javascript
let numbers = [10, 20, 30];
numbers[1] = 25; // Change the second element
console.log(numbers); // Output: [10, 25, 30]

// You can also add elements by assigning to a new index
numbers[3] = 40;
console.log(numbers); // Output: [10, 25, 30, 40]

// If you skip indices, "sparse arrays" are created with `empty` slots
numbers[6] = 70;
console.log(numbers); // Output: [10, 25, 30, 40, empty × 2, 70]
console.log(numbers[4]); // Output: undefined
```


---

## 5. Array `length` Property

The `length` property returns the number of elements in the array.

```javascript
let items = ["A", "B", "C"];
console.log(items.length); // Output: 3

// You can also use `length` to truncate or extend an array
items.length = 2;
console.log(items); // Output: ["A", "B"]

items.length = 5;
console.log(items); // Output: ["A", "B", empty × 3]
```




---

## 6. Common Array Methods
- JavaScript provides a rich set of built-in methods for manipulating arrays.
### Push():
- `push()`: used to add one or more elements to the end of the array and returns the new length. it's like the append in python.
```JavaScript
let arr = [1, 2]; 
arr.push(3, 4); // arr is now [1, 2, 3, 4]

arr.push("Ahmed");
console.log(arr); // [1, 2, 3, 4, "Ahmed"]
```


---

### pop():
- `pop()`: Removes the last element from an array and returns that element.

```JavaScript
let arr = [1, 2, 3];
let last = arr.pop(); // last is 3, arr is now [1, 2]

console.log(arr.pop()); // 2
```


---

### unshift():
`unshift()`: used to add one or more elements in the beginning of the array and returns the new length.

```JavaScript
let arr = [3, 4];
arr.unshift(1, 2); // arr is now [1, 2, 3, 4]
arr.unshift("Essam");
console.log(arr); // ["Essam", 1, 2, 3, 4]
```


---


### shift():
`shift()`: Removes the first element from an array and returns that element.

```JavaScript
let arr = [1, 2, 3];
let first = arr.shift(); // first is 1, arr is now [2, 3]
```


---


### sort():
`sort()`: used to sort the array alphabetically, so it doesn't sort numbers.

```JavaScript
let arr = ["omar", "Osman", "Saad", "Ibrahim", "Ali", "Mahmoud"];
console.log(arr.sort()); // [ 'Ali', 'Ibrahim', 'Mahmoud', 'Osman', 'Saad', 'omar' ]
```

**How to use `sort()` to sort numeric array?**
- there is a specific way to user `sort()` function to sort numeric array by using a compare function.

```javascript
let arr = [3, 2, 5, 5, 1, 6, 7];

// Sort in ascending order
arr.sort(function(a, b){
	return a - b;
});
console.log(arr); // [ 1, 2, 3, 5, 5, 6, 7 ]

// Sort in Descending order
arr.sort(function(a, b){
	return b - a;
})
console.log(arr); // [ 7, 6, 5, 5, 3, 2, 1 ]
```

💡 Using arrow functions (shorter syntax):

```javascript
let arr = [3, 2, 5, 5, 1, 6, 7];

// Sort in ascending order
arr.sort((a, b) =>  a - b)
console.log(arr); // [ 1, 2, 3, 5, 5, 6, 7 ]

// Sort in Descending order
arr.sort((a, b) =>  b - a)
console.log(arr); // [ 7, 6, 5, 5, 3, 2, 1 ]
```

##### ⚠️ Why `numbers.sort()` without a compare function is wrong:

```JavaScript
let numbers = [5, 12, 1, 42, 7]; 
numbers.sort(); 
console.log(numbers);  // [1, 12, 42, 5, 7] ❌ Wrong
```

This happens because `sort()` converts numbers to strings and compares them character by character:
- `"1"` < `"12"` < `"42"` < `"5"` < `"7"


--- 

### splice():

*   `splice(start, deleteCount, item1, ..., itemN)`: A versatile method that can add, remove, or replace elements at any position.
 #### 🔹 Parameters

- `start`:  
    The index at which to start changing the array.
    
- `deleteCount`:  
    The number of elements to remove starting from `start`.
    
- `item1, item2, ...` _(optional)_:  
    Elements to **add** to the array starting from `start`.


```JavaScript
let arr = ["a", "b", "c", "d", "e"];
arr.splice(2, 1); // Remove 1 element starting at index 2 (removes "c") -> ["a", "b", "d", "e"]
arr.splice(1, 0, "x", "y"); // Add "x", "y" at index 1 (no elements removed) -> ["a", "x", "y", "b", "d", "e"]
arr.splice(2, 2, "p", "q"); // Remove 2 elements from index 2, then add "p", "q" -> [ 'a', 'x', 'p', 'q', 'd', 'e' ]
```



---


### Slice()
- The `slice(start, end)` method in JavaScript is used to create a **<span style="color:rgb(0, 176, 80)">shallow</span> <span style="color:rgb(0, 176, 80)">copy</span>** of a portion of an array into a **<span style="color:rgb(0, 176, 80)">new</span> <span style="color:rgb(0, 176, 80)">array</span>**, without modifying the original array.
#### Parameters
- `start` _(optional)_:
    
    - The index at which to begin extraction.
    - If omitted, defaults to `0`.
    - If negative, counts from the end of the array (`-1` is the last element).
    
- `end` _(optional)_:
    
    - The index **before which** to end extraction (i.e., up to but not including this index).
    - If omitted, extracts through the end of the array.
    - If negative, counts from the end of the array.

### ✅ Examples

```JavaScript
let fruits = ["apple", "banana", "cherry", "date", "fig"];

// Basic usage
let sliced = fruits.slice(1, 4);
console.log(sliced); // ["banana", "cherry", "date"]

// Omitting end
console.log(fruits.slice(2)); // ["cherry", "date", "fig"]

// Using negative indexes
console.log(fruits.slice(-3, -1)); // ["cherry", "date"]

// Copying the whole array
let copy = fruits.slice();
console.log(copy); // ["apple", "banana", "cherry", "date", "fig"]
```





---

**b) Iteration Methods**

*   `forEach(callback)`: Executes a provided function once for each array element.
    ```javascript
    let numbers = [1, 2, 3];
    numbers.forEach(num => console.log(num * 2)); // Output: 2, 4, 6
    ```
*   `map(callback)`: Creates a new array populated with the results of calling a provided function on every element in the calling array.
    ```javascript
    let numbers = [1, 2, 3];
    let doubled = numbers.map(num => num * 2); // doubled is [2, 4, 6]
    ```
*   `filter(callback)`: Creates a new array with all elements that pass the test implemented by the provided function.
    ```javascript
    let numbers = [1, 2, 3, 4, 5];
    let evens = numbers.filter(num => num % 2 === 0); // evens is [2, 4]
    ```
*   `reduce(callback, initialValue)`: Executes a reducer function on each element of the array, resulting in a single output value.
    ```javascript
    let numbers = [1, 2, 3, 4];
    let sum = numbers.reduce((acc, current) => acc + current, 0); // sum is 10
    ```
*   `find(callback)`: Returns the value of the first element in the array that satisfies the provided testing function. Otherwise, `undefined` is returned.
    ```javascript
    let users = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }];
    let bob = users.find(user => user.name === "Bob"); // bob is { id: 2, name: "Bob" }
    ```
*   `findIndex(callback)`: Returns the index of the first element in the array that satisfies the provided testing function. Otherwise, -1 is returned.
    ```javascript
    let users = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }];
    let bobIndex = users.findIndex(user => user.name === "Bob"); // bobIndex is 1
    ```

**c) Searching and Testing**

*   `indexOf(element, fromIndex)`: Returns the first index at which a given element can be found in the array, or -1 if it is not present.
    ```javascript
    let arr = ["a", "b", "c", "a"];
    console.log(arr.indexOf("a")); // 0
    console.log(arr.indexOf("d")); // -1
    ```
*   `includes(element, fromIndex)`: Determines whether an array includes a certain value among its entries, returning `true` or `false` as appropriate.
    ```javascript
    let arr = [1, 2, 3];
    console.log(arr.includes(2)); // true
    console.log(arr.includes(4)); // false
    ```
*   `some(callback)`: Tests whether at least one element in the array passes the test implemented by the provided function.
*   `every(callback)`: Tests whether all elements in the array pass the test implemented by the provided function.

**d) Transforming and Combining**

*   `slice(start, end)`: Returns a shallow copy of a portion of an array into a new array object selected from `start` to `end` (end not included).
    ```javascript
    let arr = [1, 2, 3, 4, 5];
    let subArr = arr.slice(1, 4); // subArr is [2, 3, 4]
    ```
*   `concat(array1, array2, ...)`: Used to merge two or more arrays. This method does not change the existing arrays, but instead returns a new array.
    ```javascript
    let arr1 = [1, 2];
    let arr2 = [3, 4];
    let combined = arr1.concat(arr2, [5, 6]); // combined is [1, 2, 3, 4, 5, 6]
    ```
*   `join(separator)`: Creates and returns a new string by concatenating all of the elements in an array (or an array-like object), separated by commas or a specified separator string.
    ```javascript
    let fruits = ["Apple", "Banana", "Cherry"];
    let str = fruits.join(", "); // str is "Apple, Banana, Cherry"
    ```
*   `reverse()`: Reverses an array in place. The first array element becomes the last, and the last array element becomes the first.
    ```javascript
    let arr = [1, 2, 3];
    arr.reverse(); // arr is now [3, 2, 1]
    ```
*   `sort(compareFunction)`: Sorts the elements of an array in place and returns the sorted array. The default sort order is ascending, built upon converting the elements into strings, then comparing their sequences of UTF-16 code units values. For numbers, you usually need a `compareFunction`.
    ```javascript
    let numbers = [3, 1, 4, 1, 5, 9];
    numbers.sort(); // numbers is now [1, 1, 3, 4, 5, 9] (works for single digits)

    let moreNumbers = [10, 2, 100, 5];
    moreNumbers.sort(); // [10, 100, 2, 5] (incorrect string sort)
    moreNumbers.sort((a, b) => a - b); // [2, 5, 10, 100] (correct numeric sort)
    ```

**e) Other**

*   `Array.isArray(value)`: Determines whether the passed value is an `Array`.
    ```javascript
    console.log(Array.isArray([1, 2])); // true
    console.log(Array.isArray("hello")); // false
    ```

Understanding these concepts and methods will give you a strong foundation for working with arrays in JavaScript.