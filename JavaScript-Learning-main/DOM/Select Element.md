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
```JavaScript
// Select the Heading in our webpage with id = mainTitle
let mainHeading = document.getElementById("mainTitle");
```

- After getting this element we could apply different operation on this element like:
####  Changing the HTML Content 
`innerHTML` property in JavaScript is used to get or set the HTML content **inside** an HTML element.

######  HTML:
```html
<div id="example">Hello <b>World</b></div>
```

###### JavaScript:
```JavaScript
let content = document.getElementById("example").innerHTML; 
console.log(content); // Output: Hello <b>World</b>  

// setting the content of the html element to new content
document.getElementById("example").innerHTML = "Goodbye <i>World</i>";
```

###### Resulting HTML:
```html
<div id="example">Goodbye <i>World</i></div>
```

- So `innerHTML` property used to override the content of the html element.
##### 🛠️ Use cases:
- Updating parts of a web page without refreshing.
- Inserting new HTML dynamically (e.g., adding a list of items).
- Basic DOM manipulation when you don't need fine control.

