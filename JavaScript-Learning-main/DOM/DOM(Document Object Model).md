# Document Object Model
The **DOM** stands for **Document Object Model**.

- It is a programming interface for web documents (HTML, XML, SVG).
- It represents the page so that programs can change the document structure, style, and content.
- The DOM represents the document as a tree of objects, where each object corresponds to a part of the document, such as an element, an attribute, or a piece of text.

Essentially, the DOM is a **platform- and language-neutral interface** that allows programs and scripts (like JavaScript) to dynamically access and update the content, structure, and style of a document.

### Key Concepts of the DOM:

1.  **Tree Structure**:
    *   The DOM models an HTML document as a logical tree. Each node in the tree represents a part of the document.
    *   The top-level node is the `Document` object.
    *   The `<html>` element is typically the root element node.
    *   Elements like `<body>`, `<head>`, `<div>`, `<p>`, `<a>`, etc., are represented as **element nodes**.
    *   Text inside elements is represented as **text nodes**.
    *   Attributes of elements (e.g., `id="myId"`, `class="myClass"`) are represented as **attribute nodes**.
    *   Comments are **comment nodes**.

    **Example HTML:**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>My Page</title>
    </head>
    <body>
        <h1 id="main-title">Welcome</h1>
        <p>This is a <span>paragraph</span>.</p>
    </body>
    </html>
    ```

    **Simplified DOM Tree Representation:**
    ```
    Document
    └── html
        ├── head
        │   └── title
        │       └── "My Page" (text node)
        └── body
            ├── h1 (id="main-title")
            │   └── "Welcome" (text node)
            └── p
                ├── "This is a " (text node)
                ├── span
                │   └── "paragraph" (text node)
                └── "." (text node)
    ```

2.  **Objects and Properties**:
    *   Every part of the document (elements, attributes, text) is represented as a JavaScript object.
    *   These objects have properties (e.g., `id`, `className`, `innerHTML`, `textContent`, `style`) and methods (e.g., `addEventListener`, `appendChild`, `removeChild`, `querySelector`).

3.  **API (Application Programming Interface)**:
    *   The DOM provides a set of APIs (methods and properties) that JavaScript can use to:
        *   **Find/Select elements**: 
        * `document.getElementById()` [[Select Element#document.getElementById(id)]], 
        
        * `document.querySelector()` [[Select Element#document.querySelector(selector)]],
        
        * `document.querySelectorAll()` [[Select Element#document.querySelectorAll(selector)]],
        
        * `document.getElementsByClassName()` [[Select Element#document.getElementByClassName(className)]],
        
        * `document.getElementsByTagName()` [[Select Element#document.getElementsByTagName(tagName)]].
        
        *   **Change HTML content**: 
        * `element.innerHTML` [[Changing & Retrieving the HTML Content#innerHTML]],
        
        * `element.innerText` [[Changing & Retrieving the HTML Content#innerText]],
        
        * `element.textContent` [[Changing & Retrieving the HTML Content#textContent]].
        
        *   **Change CSS styles**: `element.style.property`.
        *   **Add/Remove elements**: `document.createElement()`, `parentNode.appendChild()`, `parentNode.removeChild()`.
        *   **Add/Remove attributes**: `element.setAttribute()`, `element.removeAttribute()`, `element.getAttribute()`.
        *   **Handle events**: `element.addEventListener()`, `element.removeEventListener()`.

### The `document` Object:

The `document` object is the entry point to the DOM. It represents the entire web page and is the root of the DOM tree. It's a property of the `window` object (`window.document`).

**Common `document` object properties and methods:** [[Find Elements by Object Collections]]
 
*   `document.documentElement`: Refers to the `<html>` element.
*   `document.head`: Refers to the `<head>` element.
*   `document.body`: Refers to the `<body>` element.
*   `document.title`: Gets or sets the title of the document.
*   `document.URL`: Gets the full URL of the document.
*   `document.forms`: A collection of all `<form>` elements.
*   `document.links`: A collection of all `<a>` elements with an `href` attribute.
*   `document.images`: A collection of all `<img>` elements.

### How JavaScript Interacts with the DOM:

JavaScript uses the DOM API to manipulate the web page.

**Example of DOM Manipulation:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>DOM Example</title>
    <style>
        .highlight {
            color: blue;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1 id="myHeading">Hello, DOM!</h1>
    <button id="changeTextBtn">Change Text</button>
    <button id="addParagraphBtn">Add Paragraph</button>
    <div id="container"></div>

    <script>
        // 1. Get an element by its ID
        const heading = document.getElementById('myHeading');
        console.log(heading.textContent); // Output: "Hello, DOM!"

        // 2. Change its content
        heading.textContent = "DOM Manipulation in Action!";

        // 3. Change its style
        heading.style.color = "red";

        // 4. Add an event listener to a button
        const changeTextButton = document.getElementById('changeTextBtn');
        changeTextButton.addEventListener('click', () => {
            heading.textContent = "Text changed by button!";
            heading.classList.add('highlight'); // Add a CSS class
        });

        // 5. Create and append a new element
        const addParagraphButton = document.getElementById('addParagraphBtn');
        const container = document.getElementById('container');

        addParagraphButton.addEventListener('click', () => {
            const newParagraph = document.createElement('p'); // Create a new <p> element
            newParagraph.textContent = "This is a dynamically added paragraph.";
            container.appendChild(newParagraph); // Add it to the container div
        });
    </script>
</body>
</html>
```

### DOM vs. HTML Source Code:

*   **HTML Source Code**: The initial text file that defines the structure of the web page.
*   **DOM**: A live, in-memory representation of the document. It can be modified by JavaScript, and these modifications are immediately reflected in what the user sees in the browser. The DOM can also differ from the initial HTML if the browser corrects invalid HTML or if scripts have already modified it.

### DOM vs. BOM:

*   **DOM (Document Object Model)**: Focuses on the **content** of the web page (the HTML structure, elements, text, attributes). It's accessed via the `document` object.
*   **BOM (Browser Object Model)** [[BOM(Browse Object Model)]]: Focuses on the **browser window** itself and its environment (e.g., `window`, `navigator`, `screen`, `location`, `history`). It's accessed via the `window` object.

The DOM is a fundamental concept for web development, enabling dynamic and interactive web pages by allowing JavaScript to read and modify the structure and content of a document.