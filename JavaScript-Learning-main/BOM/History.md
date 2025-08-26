# Window.history
- `window.history` object is a part of BOM [[BOM(Browse Object Model)]].
- It provides an interface to interact with the browser's session history.
- It allows you to navigate backward and forward through the user's history and, more advanced, manipulate the history stack for single-page applications (SPAs) without full page reloads.

## Basic Navigation Methods
- These methods allow you to move through the history stack similar to using the browser's back and forward buttons.
### history.back()
- **Description**: Navigates to the previous URL in the browser's history.
- **Equivalent to**: Clicking the browser's "Back" button.
#### Example
```JavaScript
// Go back one page
history.back();
```

---
### history.forward()
- **Description**: Navigates to the next URL in the browser's history.
- **Equivalent to**: Clicking the browser's "forward" button.
#### Example
```JavaScript
// Go back one page
history.forward();
```

--- 
### history.go(delta)
- **Description**: Navigates to a specific entry in the history list, relative to the current page.
- **`delta`**:
    - A positive integer moves forward (e.g., `1` for forward one page, `2` for forward two pages).
    - A negative integer moves backward (e.g., `-1` for back one page, `-2` for back two pages).
    - `0` reloads the current page (similar to `location.reload()`).
#### Example
```JavaScript
// Go back two pages
history.go(-2);

// Go forward one page
history.go(1);

// Reload the current page
history.go(0);
```


---
## Properties
### history.length
`history.lenght` is a read-only property that returns the number of URLs is the history list for the current window.

#### Example
```JavaScript
console.log(`History length: ${history.length}`); // e.g., 5
```

- **Note**: This count includes the current page.

### history.state
`history.state` is a read-only property that returns the state object associated with the current history entry.
- This is particularly useful with `pushState()` and `replaceState()` for SPAs.
#### Example
```JavaScript
// If a state was pushed with { page: 'home' }
console.log(history.state); // Output: { page: 'home' }
```

