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
