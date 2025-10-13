# Select Elements
- In Java script there are many ways to select single element or multiple elements from the html structure.
- Before we start this methods we have to know that all these methods belongs to the Document Object.
- Let's create our html code that we will work with:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>DOM Practice Page</title>
  <style>
    .highlight {
      background-color: yellow;
    }
  </style>
</head>
<body>

  <h1 id="mainTitle">Welcome to DOM Practice</h1>

  <p class="description" name="descPara">This is a paragraph with a class and name.</p>
  <p class="description">This is another paragraph with the same class.</p>

  <input type="text" id="username" name="username" placeholder="Enter username">

  <input type="password" id="password" name="password" placeholder="Enter password">

  <button id="submitBtn" class="action" name="submitBtn">Submit</button>

  <button id="resetBtn" class="action" name="resetBtn">Reset</button>

  <div id="output" class="output-container" name="resultArea">
    <p id="statusMessage">Status will appear here...</p>
  </div>

  <ul id="itemsList" class="list" name="itemList">
    <li class="item" id="item1" name="listItem">Item 1</li>
    <li class="item" id="item2" name="listItem">Item 2</li>
    <li class="item" id="item3" name="listItem">Item 3</li>
  </ul>

  <a href="#bottom" id="scrollLink" name="scrollAnchor">Go to bottom</a>

  <br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
  <div id="bottom">Bottom of the page</div>
</body>
</html>
```

---
#getElementById
## document.getElementById(id)
`document.getElementById(id)` used to **retrieve a reference to an HTML element by its unique `id` attribute**.
- As the id in `html` is unique, then `document.getElementById(id)` used to select only one element.
### Parameters

- **`id`**:
    - **Required**. A <span style="color:rgb(0, 176, 80)">string</span> representing the `id` attribute of the element you want to find.
    - The `id` attribute in HTML **must be unique** within the entire document. If multiple elements have the same `id`, `getElementById()` will only return the first one it encounters in the document's source order.
### Example
#### HTML
```html
<div id="demo">Hello <span style="display:none">invisible</span> World</div>
```

#### JavaScript
```JavaScript
// Select the Heading in our webpage with id = mainTitle
let demoDiv = document.getElementById("demo");
```

- After getting this element we could apply different operation on this element like:

```JavaScript
console.log(demoDiv.textContent);
// Output: "Hello invisible World"

console.log(demoDiv.innerText);
// Output: "Hello  World"

console.log(demoDiv.innerHTML);
// Output: 'Hello <span style="display:none">invisible</span> World'
```

- Al the previous operations are getter and setter `textContent`, `innerText`, `innerHtml`.
- This mean we could change the data inside the tag using them.


---
#getElementsByTagName
## document.getElementsByTagName(tagName)
`document.getElementsByTagName(tagName)`  is a way to select elements from a document based on their tag name.
- The `getElementsByTagName()` method of the `Document` interface returns a live `HTMLCollection` of all of the elements in the document with the given tag name.

### Key Characteristics and Behavior:

1. **Selection by Tag Name:**
    - It takes a string argument, which is the tag name (e.g., `'div'`, `'p'`, `'a'`, `'img'`).
    - It returns all elements that match that tag name, regardless of their position in the DOM tree.

2. **`HTMLCollection` Return Type:**    
    - The method returns an `HTMLCollection`. This is an _array-like object_ (you can access elements by index and it has a `length` property), but it's **not a true JavaScript array**.
    - You can iterate over it using a `for` loop or convert it to an array using `Array.from()` or the spread operator (`...`).
    - **Live Collection:** This is a crucial point. An `HTMLCollection` returned by `getElementsByTagName` is **live**. This means that if elements are added to or removed from the document that match the specified tag name _after_ the collection has been created, the collection will automatically update to reflect these changes.

3. **Case-Insensitivity (HTML documents):**
    - In HTML documents, tag names are case-insensitive. So, `document.getElementsByTagName('p')` will find `<p>`, `<P>`, etc.
    - In XML (including XHTML served as XML), tag names are case-sensitive.

4. **Context:**
    - You can call `getElementsByTagName` on the `document` object to search the entire document.
    - You can also call it on any specific element to search only within that element's descendants.

### Syntax:

```javascript
// Search the entire document
document.getElementsByTagName(tagName);

// Search within a specific element's descendants
element.getElementsByTagName(tagName);
```

- `tagName`: A string representing the tag name to match. The special value `'*'` can be used to return all elements in the document (or within the specified element).

### Example:
```JavaScript
// get all list items from the previous html code in the beging of this document.
let lstitems = document.getElementsByTagName("li");

console.log(lstitems.lenght); // 3

// Convert to a real array and then iterate (modern approach)
Array.from(lstitems).forEach(item => {
console.log(item.textContent);
});
// Output:
// Item 1
// Item 2
// Item 3

// Get All elements in the Dcoument file.
let allElements = document.getElementsByTagName("*");
console.log(allElements.length); // 39
```


- We can't change the html content using one of the changing properties[[Changing & Retrieving the HTML Content]] with `getElementsByTagName` directly, as it retrieve a collection of html elements not just single element this why we use `Elements` keyword.
- But we could change any retrieved element using its index number like:

```javaScript
let lsitems = document.getElementsByTagName("li");

lsitems[0].innerText = "This is an Changed list item using its index number";
```

- Now the text of the first element will be changed and the elements will be like:

```html
<ul id="itemsList" class="list" name="itemList">
    <li class="item" id="item1" name="listItem">This is an Changed list item using its index number</li>
    <li class="item" id="item2" name="listItem">Item 2</li>
    <li class="item" id="item3" name="listItem">Item 3</li>
  </ul>
```

- And that's exactly what we have to do to deal with each selected elements to apply any change on it, we have to use its index number. 
### When to Use `getElementsByTagName`:

- When you need to select all elements of a specific type (e.g., all paragraphs, all images, all links).
- When you need a **live collection** that automatically updates as the DOM changes. This can be useful in scenarios where you're constantly adding/removing elements and need an up-to-date list without re-querying.



---
#getElementByClassName
## document.getElementByClassName(className)
`document.getElementByClassName(className)` method of the `Document` interface returns a live `HTMLCollection` of all of the elements in the document that have all of the class names specified in the argument.

### Key Characteristics and Behavior:

1. **Selection by Class Name(s):**
    - It takes a string argument, which can be a single class name or a space-separated list of class names.
    - If multiple class names are provided (e.g., `'highlight important'`), an element must possess _all_ of those classes to be included in the returned collection. The order of the classes in the element's `class` attribute <span style="color:rgb(192, 0, 0)">does not matter</span>.

2. **`HTMLCollection` Return Type:**   
    - Similar to `getElementsByTagName`, this method returns an `HTMLCollection`. This is an array-like object (you can access elements by index and it has a `length` property), but it's **not a true JavaScript array**.
    - You can iterate over it using a `for` loop or convert it to an array using `Array.from()` or the spread operator (`...`).
    - **Live Collection:** This is a critical feature. An `HTMLCollection` returned by `getElementsByClassName` is **live**. This means that if elements are added to or removed from the document, or if an element's `class` attribute is modified to match or no longer match the specified class names _after_ the collection has been created, the collection will automatically update to reflect these changes.

3. **Case-Sensitivity:**   
    - Class names are **case-sensitive**. `'myClass'` is different from `'myclass'`.

4. **Context:**   
    - You can call `getElementsByClassName` on the `document` object to search the entire document.
    - You can also call it on any specific element to search only within that element's descendants.

### Syntax:

```javascript
// Search the entire document
document.getElementsByClassName(classNames);

// Search within a specific element's descendants
element.getElementsByClassName(classNames);
```

- `classNames`: A string representing one or more class names to match, separated by spaces.

### Example:
```JavaScript
let actionClass = document.getElementsByClassName('action');
// This will get all the elements that have the class action.

console.log(actionClass.lenght); // 2 , because there are two buttons with this class name.

// To change anything about the retreved elements we need to acess them using index number.
actionClass[1].textContent = "Button with actionClass";
// This will change the text inside the second button.
```

### When to Use `getElementsByClassName`:

- When you need to select all elements that share a specific styling or behavioral characteristic defined by a class.
- When you need a **live collection** that automatically updates as elements are added, removed, or have their class attributes changed in the DOM.



---
#querySelectorAll
## document.querySelectorAll(selector)
`document.querySelectorAll()` is one of the most powerful and commonly used methods in modern DOM JavaScript for selecting elements. It allows you to select elements using <span style="color:rgb(255, 0, 0)">CSS selectors</span>, providing a highly flexible and familiar way to target specific parts of your document.

### What is `document.querySelectorAll()`?

The `document.querySelectorAll()` method returns a **static (non-live) `NodeList`** representing a list of the document's elements that match the specified group of CSS selectors.

### Key Characteristics and Behavior:

#CssSelector
1. **Selection by CSS Selectors:**
    - This is its primary strength. You can use almost any valid CSS selector string to target elements. This includes:
        - **Tag names:** `'p'`, `'div'`, `'a'`
        - **Class names:** `'.my-class'`, `'.another-class'`
        - **IDs:** `'#my-id'`
        - **Attributes:** `'[data-attribute]'`, `'[type="text"]'`
        - **Combinators:**
            - Descendant: `'div p'` (paragraphs inside divs)
            - Child: `'ul > li'` (direct list item children of a ul)
            - Adjacent Sibling: `'h1 + p'` (a paragraph immediately following an h1)
            - General Sibling: `'h1 ~ p'` (any paragraph following an h1)
        - **Pseudo-classes:** `':hover'`, `':nth-child(odd)'`, `':first-child'`
        - **Multiple selectors (comma-separated):** `'p, .my-class, #my-id'` (selects all paragraphs, OR elements with `my-class`, OR the element with `my-id`)

2. **`NodeList` Return Type:**
    - The method returns a `NodeList`. Like `HTMLCollection`, it's an array-like object (has `length` and allows access by index).
    - **Static Collection:** This is a crucial difference from `getElementsByTagName` and `getElementsByClassName`. A `NodeList` returned by `querySelectorAll` is **static**. This means it's a snapshot of the DOM at the moment the method was called. If elements are added, removed, or their attributes change _after_ the `NodeList` is created, the `NodeList` will **not** automatically update. You would need to call `querySelectorAll()` again to get an updated list.

3. **Iteration:**
    - `NodeList` objects can be iterated using a `for` loop, `for...of` loop, or the `forEach()` method (which is available on `NodeList` in modern browsers). You can also convert it to a true array using `Array.from()` or the spread operator (`...`).

4. **Context:**
    - You can call `querySelectorAll` on the `document` object to search the entire document.
    - You can also call it on any specific element to search only within that element's descendants. This is very useful for scoping your searches.

5. **Error Handling:**
    - If the provided selector string is invalid, `querySelectorAll()` will throw a `SyntaxError` exception.

### Syntax:

```javascript
// Search the entire document
document.querySelectorAll(selectors);

// Search within a specific element's descendants
element.querySelectorAll(selectors);
```

- `selectors`: A string containing one or more CSS selectors separated by commas.

### Example:

```JavaScript
let selectedId = document.querySelectorAll("#output"); // Select the element that has id output.
console.log(selectedId[0].innerText);

let selectedClass = document.querySelectorAll(".item"); // Select all the elemnts that has class item.
console.log(selectedClass[1].innerHTML);

let selectedTag = document.querySelectorAll("p"); // Select all paragraphs tag. 
console.log(selectedTag[2].textContent);
```

- As we can in the previous example we used `#` before the id name and `.` before the class name exactly the same as selecting in CSS.
- Also to access any selected value we have to use its index number like an array, actually it's a real array.


---
#querySelector 
## document.querySelector(selector)
`document.querySelector()` is the single-element counterpart to `document.querySelectorAll()`. It's an incredibly useful and widely adopted method for selecting a specific element in the DOM using CSS selectors.

### What is `document.querySelector()`?

The `document.querySelector()` method returns the **first `Element`** within the document that matches the specified group of CSS selectors. If no elements match, it returns `null`.

### Key Characteristics and Behavior:

1. **Selection by CSS Selectors:**
    - Like `querySelectorAll`, it accepts any valid CSS selector string. This makes it extremely flexible for targeting elements by tag name, class, ID, attributes, combinators, pseudo-classes, etc.
    - Examples: `'p'`, `'.my-class'`, `'#my-id'`, `'div > p.highlight'`, `'[data-action="delete"]'`.

2. **Returns a Single Element (or `null`):**
    - This is the key difference from `querySelectorAll`. Even if multiple elements in the document match the selector, `querySelector()` will **only return the very first one** it encounters in a depth-first traversal of the DOM.
    - If no element matches the selector, it returns `null`.

3. **Context:**    
    - You can call `querySelector` on the `document` object to search the entire document.
    - You can also call it on any specific element to search only within that element's descendants. This is excellent for scoping your searches and finding elements relative to a parent.

4. **Error Handling:**
    - If the provided selector string is invalid, `querySelector()` will throw a `SyntaxError` exception.

### Syntax:
```javascript
// Search the entire document for the first match
document.querySelector(selectors);

// Search within a specific element's descendants for the first match
element.querySelector(selectors);
```

- `selectors`: A string containing one or more CSS selectors separated by commas. (Though typically, for `querySelector`, you'd use a single, specific selector to target one element)

- We could use `querySelector` to get element with specific attribute like getting the input with attribute `type="password"`.

```HTML
<input type="text" id="username" name="username" placeholder="Enter username">

<input type="password" id="password" name="password" placeholder="Enter password">

<button id="submitBtn" class="action" name="submitBtn">Submit</button>

<button id="resetBtn" class="action" name="resetBtn">Reset</button>
```

```JavaScript
const passwordInput = document.querySelector("input[type=password]");
console.log(passwordInput.value);
```
### When to Use `document.querySelector()`:

- **When you need to select a single, specific element:** This is its primary use case.
- **When you need the flexibility of CSS selectors:** It's a powerful alternative to `getElementById` when you want to use classes, attributes, or more complex relationships to target a unique element.
- **When you want to find the _first_ occurrence of an element matching a selector:** Even if multiple elements match, if you only care about the first one, `querySelector` is efficient.
- **For scoping searches:** Calling it on an `element` object (e.g., `myDiv.querySelector('p')`) is excellent for finding descendants within a particular parent.