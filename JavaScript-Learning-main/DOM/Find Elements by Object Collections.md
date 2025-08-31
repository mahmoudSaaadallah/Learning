# Common `document` object properties and methods

- There are some properties in the `document` object that we could use to access or get some important elements or values.

## document.title
- `document.title` property that used to get the title of our document(web page)which is typically displayed in the browser's title bar or tab.
- - When you read `document.title`, it returns a string containing the current title of the HTML document.
-  This is the same text that appears between the `<title>` and `</title>` tags in your HTML.
-  When you assign a new string value to `document.title`, the browser immediately updates the document's title.
- This change is reflected in the browser's title bar or tab.
- The change is dynamic and does not require a page reload.
- The string you assign is treated as plain text. Any HTML tags within the string will be displayed literally, not rendered as HTML.

### Syntax:

```javascript
// Get the current title
const currentTitle = document.title;

// Set a new title
document.title = "New Page Title";
```



---
# Document.images
`document.images` is a legacy property that returns an `HTMLCollection` of all `<img>` elements within the current document.
### What is `document.images`?

- **Type**: It's an `HTMLCollection`, which is an array-like object (but not a true array) containing all `<img>` elements found in the document.
- **Live Collection**: It's a "live" collection, meaning that if images are added to or removed from the DOM, the `document.images` collection automatically updates to reflect these changes.
- **Read-Only**: You cannot directly add or remove elements from this collection. It merely reflects the current state of the DOM.
- **Access**: You can access individual image elements by their index (e.g., `document.images[0]`) or by their `name` or `id` attribute (e.g., `document.images.myImageId`).

### Syntax:

```javascript
// Get the current title
const allImages = document.images;

```

- we could apply different properties to this collection of images like:

```JavaScript
let allImages = document.image;

// return number of image in my webpage.
console.log(allImages.length);

// Get the src for specific image in the webpage.
console.log(allImages[0].src);

// Change the length or the width of the image
allImages[1].width = 480;
allImages[1].length = 320;

// Change the whole style for the image
allImages[2].style = "display:block; width: 100px;";

```

**3. Accessing Images by `id` or `name` attribute:**

If an image has an `id` or `name` attribute, you can access it directly as a property of `document.images`.

HTML:

```html
<img src="image1.jpg" id="myImageId" alt="My first image">
<img src="image2.png" name="productImage" alt="Product shot">
```

JavaScript:

```javascript
const imageById = document.images.myImageId;
if (imageById) {
    console.log("Image by ID source:", imageById.src);
}

const imageByName = document.images.productImage;
if (imageByName) {
    console.log("Image by Name alt text:", imageByName.alt);
}
```

_Note: If multiple images share the same `name`, `document.images.name` will return an `HTMLCollection` of those images._


---
# document.forms
n JavaScript, `document.forms` is a legacy property that returns an `HTMLCollection` of all `<form>` elements within the current document.

Here's a detailed explanation:

### What is `document.forms`?

- **Type**: It's an `HTMLCollection`, which is an array-like object (but not a true array) containing all `<form>` elements found in the document.
- **Live Collection**: Similar to `document.images`, it's a "live" collection. This means that if forms are dynamically added to or removed from the DOM, the `document.forms` collection automatically updates to reflect these changes.
- **Read-Only**: You cannot directly add or remove elements from this collection. It merely reflects the current state of the DOM.
- **Access**: You can access individual form elements by their index (e.g., `document.forms[0]`) or by their `name` or `id` attribute (e.g., `document.forms.myFormId`).


### Syntax:

```javascript
// Get the current title
const allforms = document.forms;

```

- It's the same as the `document.images` we could also access the specific form using its id or name attribute.
- Also we could modify some data about the forms like the style, action,... and more.
- Also we could access the elements inside the selected form using the element id or name and retrieve or change its data.

JavaScript:

```javascript
const loginForm = document.forms.loginForm; // Access by ID
if (loginForm) {
    console.log("Login form action:", loginForm.action);
}

const searchForm = document.forms.searchForm; // Access by Name
if (searchForm) {
    console.log("Search form method:", searchForm.method);
}
```

_Note: If multiple forms share the same `name`, `document.forms.name` will return an `HTMLCollection` of those forms._


---
There is also `document.head`, `document.body`, `document.links`, and `document.anchors`.
- The difference between `document.anchors` and `document.links` is :
	- `document.anchors` returns all the `<a></a>` tags only.
	- `document.links` returns all the `<a></a>` tags and all the other links like the `href` for the elements.
