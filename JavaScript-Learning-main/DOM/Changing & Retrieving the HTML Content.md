## Changing & Retrieving the HTML Content `innerHTML`, `innerText`, `textContent`.

- Before discussing how to retrieve an change the html content we have to learn how to select the html element to apply the next properties.[[Select Element]].

#innerHTMl
### innerHTML
Changing the HTML Content 
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
- We have to that with `innerHTML` we could insert an html tag or more and this tag will be act as it's been written in the html file.
- As we saw in the previous example when we changed the `innerHTML` with the html element which has `example` id, we added `<i></i>` tag which will be execute as it has been written in the original html file.
- Why we discuss this point because there is another prop which also change in html, but can't insert html tag and make it work which we will discuss later `innerText`.
##### 🛠️ Use cases:
- Updating parts of a web page without refreshing.
- Inserting new HTML dynamically (e.g., adding a list of items).
- Basic DOM manipulation when you don't need fine control.


---
#innerText
### innerText
#### Changing the HTML Text
`innerText` property in DOM JavaScript is used to get or set the text content of an element, as it is rendered on the screen. It's a powerful property, but it's important to understand its nuances, especially when compared to similar properties like `textContent` and `innerHTML`.
- The `innerText` property represents the text content of a node. This means it takes into account how the text is displayed visually in the browser, including styling, visibility, and layout.

#####  🔍 Example:

###### HTML:

```html
<div id="example">Hello <b style="display:none">bold</b> World</div>
```

###### JavaScript:

```JavaScript
let text = document.getElementById("example").innerText;
console.log(text); // Output: "Hello  World"

text.innerText = "<p>Welcom</p>";
console.log(text); // Output: <p>Welcom</p>"
```

- The word **"bold"** is not shown because it's hidden with `display: none`, and `innerText` ignores hidden elements.
- Also when changing the `innerText` and inject html tag `<p></p>`, it displayed as its without executing.
---

### ⚠️ **Note of Caution:**

##### Key Characteristics and Behavior:

1. **Aware of Styling and Visibility:**
    - `innerText` respects CSS styling. If an element or its content is hidden via CSS (e.g., `display: none`, `visibility: hidden`), `innerText` will **not** return that text.
    - It also considers line breaks and other formatting that would be rendered by the browser. For example, if there are multiple spaces, it will collapse them as the browser would.
    - It will not return text from elements that are not rendered, such as `<script>` or `<style>` tags.

2. **Performance Implications:**
    - Because `innerText` needs to consider the visual rendering of the element (CSS, layout, visibility), accessing it can be computationally more expensive than `textContent`.
    - When you read `innerText`, the browser might need to perform a reflow (recalculate the layout of the document) to determine the rendered text, which can impact performance, especially in large or complex documents.

3. **Setting `innerText`:**
    - When you set `innerText`, any existing child nodes of the element are removed, and the specified string is inserted as a single text node.
    - The string is treated as plain text; any HTML tags within the string will be escaped and displayed literally, not rendered as HTML.

4. **Security:**
    - Setting `innerText` is generally safe from cross-site scripting (XSS) attacks because it automatically escapes HTML characters. If you set `element.innerText = "<script>alert('XSS')</script>";`, the browser will display the literal string `<script>alert('XSS')</script>` rather than executing the script.


##### When to Use `innerText`:
- **When you need the text exactly as the user sees it:** This is the primary use case. If you want to extract text that is visible and formatted according to the page's CSS, `innerText` is the correct choice.
- **When displaying user-provided text safely:** If you're taking input from a user and want to display it as plain text within an element, `innerText` prevents HTML injection.


---
#textContent
### textContent
The `textContent` property is used to get or set the text content of a node and all its descendants.
- Unlike `innerText`, `textContent` retrieves the raw, unformatted text, regardless of styling or visibility.

##### 🔍 Example:

###### HTML:

```html
<div id="demo">Hello <span style="display:none">invisible</span> World</div>
```

###### JavaScript:

```javascript
console.log(document.getElementById("demo").textContent);
// Output: "Hello invisible World"

console.log(document.getElementById("demo").innerText);
// Output: "Hello  World"

console.log(document.getElementById("demo").innerHTML);
// Output: 'Hello <span style="display:none">invisible</span> World'
```

---

### 📌 When to Use `textContent`:

- You need **all textual content**, even if it’s not visible.
- You’re updating text and want to **strip any HTML**.
- You want the **fastest, safest** way to read or write text.

---
### Comparison with `textContent` and `innerHTML`:

It's crucial to understand how `innerText` differs from its counterparts:

| Feature            | `innerText`                                                   | `textContent`                                        | `innerHTML`                  |
| :----------------- | :------------------------------------------------------------ | :--------------------------------------------------- | :--------------------------- |
| **Content**        | Rendered text (respects CSS, visibility)                      | All text content (including hidden, script, style)   | HTML content (tags and text) |
| **Performance**    | Slower (may trigger reflow)                                   | Faster                                               | Slower (parsing HTML)        |
| **Visibility**     | Ignores hidden elements                                       | Includes hidden elements                             | Includes hidden elements     |
| **Formatting**     | Respects CSS formatting (e.g., line breaks, space collapsing) | Returns raw text (preserves all spaces, line breaks) | Returns raw HTML string      |
| **Security (Set)** | Safe (escapes HTML)                                           | Safe (escapes HTML)                                  | Unsafe (can inject scripts)  |
| **Use Case**       | Displaying text as seen by user                               | Extracting all raw text                              | Manipulating HTML structure  |

#### ✅ Key Differences Between `textContent`, `innerText`, and `innerHTML`:

|Feature|`textContent`|`innerText`|`innerHTML`|
|---|---|---|---|
|Returns|All text, even if hidden|Only **visible** text (as rendered)|Returns HTML including tags|
|Affects layout?|No|Yes (reads computed styles like `display: none`)|Yes|
|Ignores|CSS styles (like hidden text)|No, considers styles|No, includes all markup|
|Safer?|Yes (plain text)|Yes (plain text)|No (can expose XSS risk if misused)|
|Includes tags?|❌ No|❌ No|✅ Yes|