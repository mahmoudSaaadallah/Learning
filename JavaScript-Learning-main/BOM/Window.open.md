# Window.open()
- The `window.open()` method in JavaScript is used to open a new browser window or a new tab. It's a powerful function, but its behavior can be influenced by browser settings, <span style="color:rgb(255, 200, 0)">pop-up blockers</span>, and <span style="color:rgb(255, 200, 0)">user interaction</span>.

---

## Syntax
The `window.open()` method has the following syntax:

```javascript
window.open(url, windowName, windowFeatures, replace);
```

All parameters are optional, but typically you'll provide at least the `url`.

### Parameters

- **`url` (optional)**:
    - A string specifying the URL of the document to load into the new window/tab.
    - If omitted or an empty string (`""`), a new blank window/tab will be opened (e.g., `about:blank`).
- **`windowName` (optional)**:
    - A string specifying the name of the browsing context (window or tab).
    - This name can be used as the `target` attribute of an `<a>` tag or a `<form>` tag.
    - If a window/tab with this `windowName` already exists, `window.open()` will load the `url` into that existing window/tab instead of opening a new one.
    - Special values:
        - `_self`: Loads the URL into the current browsing context.
        - `_blank`: Loads the URL into a new, unnamed browsing context (a new tab/window). This is the default if `windowName` is omitted.
        - `_parent`: Loads the URL into the parent browsing context of the current one.
        - `_top`: Loads the URL into the top-most browsing context (the full window).
- **`windowFeatures` (optional)**:
    - A string containing a comma-separated list of window features. These features control the appearance and behavior of the new window.
    - **Important**: Most modern browsers largely ignore these features for new _tabs_ and often for new _windows_ as well, especially if they are opened without direct user interaction. They are more reliably applied to true pop-up windows.
    - Common features include:
        - `width` (or `innerWidth`): Width of the window in pixels.
        - `height` (or `innerHeight`): Height of the window in pixels.
        - `left` (or `screenX`): Distance from the left edge of the screen.
        - `top` (or `screenY`): Distance from the top edge of the screen.
        - `menubar`: `yes` or `no` (or `1` or `0`) to show/hide the browser's menu bar.
        - `toolbar`: `yes` or `no` to show/hide the browser's toolbar.
        - `location`: `yes` or `no` to show/hide the address bar.
        - `status`: `yes` or `no` to show/hide the status bar.
        - `resizable`: `yes` or `no` to allow/prevent resizing.
        - `scrollbars`: `yes` or `no` to show/hide scrollbars.
        - `popup`: `yes` or `no` (often used to hint to the browser that this should be a pop-up window, not a tab).
        - `noopener`: Prevents the new window from having a reference to the opening window (`window.opener` will be null). This is a security best practice, especially when opening untrusted third-party links.
        - `noreferrer`: Similar to `noopener`, prevents the `Referer` header from being sent to the new window.
    - **Example `windowFeatures` string**:

```Css
width=600,height=400,left=100,top=100,resizable=yes,scrollbars=yes,toolbar=no,menubar=no
```

- **`replace` (optional)**:
    - A boolean value.
    - If `true`, the `url` replaces the current entry in the new window's history list.
    - If `false` (default), a new entry is created in the new window's history list.
    - This parameter is rarely used and often ignored by browsers.

###  Return Value: `windowObjectReference`

`window.open()` returns a reference to the newly opened window (a `Window` object).

- If the call fails (e.g., due to a pop-up blocker), it returns `null`.
- You can use this reference to interact with the new window, such as:
    - `newWindow.document.write("Hello from opener!");`
    - `newWindow.close();`
    - `newWindow.focus();`
    - `newWindow.location.href = "another-url.html";`


```JavaScript
function openNewWindow() {
    const newWindow = window.open("https://www.example.com", "_blank", "width=800,height=600");
    if (newWindow) {
        newWindow.focus(); // Try to bring the new window to the front
    } else {
        alert("Pop-up blocked! Please allow pop-ups for this site.");
    }
}
```

### Examples

**a) Opening a blank new tab:**

```javascript
window.open(); // Opens a new blank tab (or window)
```

**b) Opening a URL in a new tab:**

```javascript
window.open("https://www.google.com", "_blank");
```

**c) Opening a URL in a named window (or reusing it):**

```javascript
// First click opens a new tab/window named 'myWindow'
window.open("https://www.example.com", "myWindow");

// Subsequent clicks will load the URL into the *same* 'myWindow'
window.open("https://www.another-example.com", "myWindow");
```

**d) Opening a pop-up window with specific dimensions (often ignored by modern browsers for tabs):**

```javascript
const popup = window.open(
    "popup.html",
    "myPopup",
    "width=400,height=300,left=200,top=200,resizable=no,scrollbars=no"
);

if (popup) {
    popup.focus();
    // You can interact with the popup if it's from the same origin
    // popup.document.body.innerHTML = "<h1>Welcome!</h1>";
} else {
    alert("Pop-up blocked!");
}
```


#### close()
#### `newWindow.close();`
- used to close the opening window using the name of the window that we specify while opining the window.

```JavaScript
var myWidow = window.open(
	"https://www.google.com", "_blank", "width=600, height=650");
setTimeout(function() {
	myWindow.close();
}, 10000);
```

#### focus()
#### `newWindow.focus();`
- used to focus on the opening window using the name of the window that we specify while opining the window.
- Focusing means make the widow on the screen.

```JavaScript
var myWidow = window.open(
	"https://www.google.com", "_blank", "width=600, height=650");
setTimeout(function() {
	myWindow.focus();
}, 10000);
```
