
## Window object
- Before discussing the BOM let's first learn what is the object window.

- The **`window` object** is the most fundamental and top-level object in the Browser Object Model (BOM) in client-side JavaScript. It represents the browser's window or a frame within a browser.

- In a web browser environment, the `window` object is the **global object**. This means that all global JavaScript variables, functions, and objects are actually properties or methods of the `window` object.

- For example, when you declare `let myVar = 10;` or `function sayHello() {}`, `myVar` becomes `window.myVar` and `sayHello` becomes `window.sayHello`.
- You can often omit `window.` when accessing its properties or methods (e.g., `alert()` is actually `window.alert()`).

- All the prop and the methods that we call directly without name of object belong to the `window` object.

### Some methods in the Window object
#### alert():
- `window.alert("message")`: Displays an alert box with a message and an OK button.
```JavaScript
// Calling without the window object.
alert("This is an alert");

// Calling with the window object.
window.alert("This is also an alert");
```

---

#### confirm()
- `window.confirm("message")`: Displays a dialog box with a message and an OK and a Cancel button.
-  It pauses the execution until the user clicks one of the buttons.
- It returns a **boolean**:
	- `true` if the user clicks **OK**.
	- `false` if the user clicks **Cancel**.

```JavaScript
if (confirm("Are you sure you want to delete this item?")) {
  console.log("User clicked OK");
  // Perform delete action
} else {
  console.log("User clicked Cancel");
  // Do nothing or cancel the action
}
```

##### Important Notes:

- `confirm()` is **blocking**, which means it stops the code execution until user responds.
- Because it’s modal, it can disrupt user experience if overused.
- Styling or customizing the dialog is **not possible** — it depends on the browser.

---

#### prompt()
- `prompt()` is a method of the `window` object.
- It displays a **modal dialog box** that asks the user to input some text.
- It has a message to show and an optional default input value.
- When the user submits, it returns the **string** entered by the user.
- If the user clicks **Cancel**, it returns `null`.

##### Syntax:
`let userInput = prompt(message, defaultValue);`

```JavaScript
let name = prompt("Please enter your name:", "John Doe");

if (name !== null) {
  console.log("Hello, " + name + "!");
} else {
  console.log("User cancelled the prompt.");
}
```

##### Important Notes:

- Like `confirm()`, `prompt()` is **blocking**.
- You can only get **text input**; no other types of input are supported
- You cannot style or customize the prompt dialog (browser-dependent).


--- 

#### print()
`window.print()` used to print the current webpage.

#### How does `window.print()` work?
- It opens the browser's **Print Dialog**.
- The user can then choose a printer, page settings, and print the page.
- It doesn’t print something directly from code; instead, it shows the print preview and options.

```JavaScript
// print the current webpage
print();
window.print();

// Also we could use it with the DOM to make a print button which will print the webpage when it will be clicked.
// When the user clicks a button, open the print dialog
document.getElementById("printBtn").onclick = function() {
  window.print();
};
```


--- 

#### scroll by && scroll to
##### 🔹 `scrollTo()`

##### 📌 What it does:

Scrolls the window **to an exact position** on the page.

##### 📄 Syntax:

`window.scrollTo(x, y);`

- `x`: Horizontal position in **pixels** (from the left edge).
- `y`: Vertical position in **pixels** (from the top edge).

##### ✅ Example:

```JavaScript
// Scroll to the top-left corner (0,0) 
window.scrollTo(0, 0);  

// Scroll to 500px from the top 
window.scrollTo(0, 500);
```

---

#### 🔹 `scrollBy()`

##### 📌 What it does:

Scrolls the window **by a relative amount** from its current position.

##### 📄 Syntax:

`window.scrollBy(x, y);`

- `x`: Number of pixels to scroll horizontally.
- `y`: Number of pixels to scroll vertically.

##### ✅ Example:

```JavaScript
// Scroll down by 300 pixels from current position 
window.scrollBy(0, 300);
  
// Scroll left by 100 pixels 
window.scrollBy(-100, 0);
```


##### 🔁 Key Difference:

|Feature|`scrollTo()`|`scrollBy()`|
|---|---|---|
|Scroll Type|**Absolute** (to a position)|**Relative** (from current position)|
|Use Case|Jump to a known location|Move by a certain amount|

--- 

#### Inner(outer)height && Inner(outer)width


![[Pasted image 20250825183307.png]]

### 🔹 1. `window.innerHeight` & `window.innerWidth`

##### 📌 Definition:
- The **viewport size** (the visible area of the web page **excluding** browser UI like toolbars, tabs, etc.).
- Includes **scrollbars**, but not the browser chrome.
##### 📄 Example:

```JavaScript
console.log(window.innerWidth);  // e.g., 1280 
console.log(window.innerHeight); // e.g., 720
```


---
### 🔹 2. `window.outerHeight` & `window.outerWidth`

##### 📌 Definition:
- The **entire browser window size**, including:

    - Viewport
    - Scrollbars
    - Developer tools (if open)
    - Browser UI (tabs, address bar, bookmarks bar, etc.)

##### 📄 Example:

```JavaScript
console.log(window.outerWidth);  // e.g., 1440
console.log(window.outerHeight); // e.g., 900
```
---
## 🆚 Comparison Table

|Property|Measures|Includes UI?|Includes Scrollbars?|
|---|---|---|---|
|`innerWidth`|Viewport width|❌ No|✅ Yes|
|`innerHeight`|Viewport height|❌ No|✅ Yes|
|`outerWidth`|Full browser window width|✅ Yes|✅ Yes|
|`outerHeight`|Full browser window height|✅ Yes|✅ Yes|

---

##### 🧪 Example Use Case

```JavaScript
if (window.innerWidth < 600) {   
	alert("You're using a small screen!"); 
}
```

Useful for:
- Responsive designs
- Custom UI adjustments
- Handling popups or centering elements
- 
---


Check window.open [[Window.open]].