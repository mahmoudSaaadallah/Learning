# Window.Location
- The **`location` object** in JavaScript is a crucial part of the Browser Object Model (BOM)[[BOM(Browse Object Model)]]. 
- It provides detailed information about the current URL of the document and offers methods to navigate to new URLs.

- It is a property of the `window` object[[Window Object]], meaning you can access it as `window.location` or simply `location` (since `window` is the global object).


## Understanding the URL Structure

To understand the `location` object, it's helpful to recall the typical structure of a URL:
`protocol://hostname:port/pathname?search#hash`

Example: `https://www.example.com:8080/path/to/page.html?name=Alice&id=123#section2`


---
## `location` Object Properties

- Each property of the `location` object corresponds to a specific part of the current URL.
- Most of these properties are **read/write**, meaning you can change them to navigate the browser to a new URL.

### location.href
- `location.href` Returns or sets the entire URL as a string.
-  Assuming current URL is https://www.example.com/path?query=test#hash

```JavaScript
console.log(location.href); // Output: "https://www.example.com/path?query=test#hash"

// To navigate to a new URL:
location.href = "https://www.google.com"; // Navigates to Google
```

---

### location.protocol
`location.protocol` Returns or sets the protocol of the URL, including the colon(:).
#### Example
```JavaScript
// Assuming current URL is https://www.example.com
console.log(location.protocol); // Output: "https:"

location.protocol = "http:"; // Changes protocol and reloads page (if allowed by server)
```

---

### location.host
 **`location.host`**: Returns or sets the hostname and port number of the URL.
#### Example
```JavaScript
// Assuming current URL is https://www.example.com:8080/
console.log(location.host); // Output: "www.example.com:8080"

// Assuming current URL is https://www.example.com/ (default port 443 for HTTPS)
console.log(location.host); // Output: "www.example.com"

location.host = "newhost.com:3000"; // Navigates to new host/port
```

---

### location.hostname
`location.hostname` Returns or sets the hostname(domain name) of the URL, without the port number.
#### Example
```JavaScript
// Assuming current URL is https://www.example.com:8080/
console.log(location.hostname); // Output: "www.example.com"

location.hostname = "anotherdomain.org"; // Navigates to new domain
```

---

### location.port
	`location.port` Returns or sets the port number of the URL, Returns an empty string if the port is the default(80 for HTTP, 443 for HTTPS).
#### Example
```JavaScript
// Assuming current URL is https://www.example.com:8080/
console.log(location.port); // Output: "8080"

// Assuming current URL is https://www.example.com/
console.log(location.port); // Output: "" (empty string)

 location.port = "3000"; // Navigates to same host, new port
```

---

### location.pathname
 `location.pathname` Returns or sets the path part of the URL, starting with a slash(/).
#### Example
```JavaScript
// Assuming current URL is https://www.example.com/users/profile.html?id=1
console.log(location.pathname); // Output: "/users/profile.html"
location.pathname = "/admin/dashboard.html"; // Navigates to new path
```

---

### location.search
`location.search` Returns or sets the query string part of the URL, starting with a question mark(?).
#### Example
```JavaScript
// Assuming current URL is https://www.example.com/page?name=John&age=30
console.log(location.search); // Output: "?name=John&age=30"

location.search = "?category=books"; // Navigates to same page with new query string
```

- **Note**: To parse individual query parameters, you'll often use `URLSearchParams` (e.g., `new URLSearchParams(location.search)`).

- That's very important note as when we retrieve the query string we usually use the (&) symbol to parse the query and get each element alone then parsing the result again using(=) symbol to get the key and the value.
- But using `URLSearchParms` class we could automatically parsing the result directly.

```JavaScript
// suppose the URL is: https://example.com/?product=shirt&color=blue&size=m
const params = new URLSearchParms(location.search);

console.log(params.get("product"));  // "shirt"
console.log(params.get("color"));    // "blue"
console.log(params.get("size"));     // "m"

// Check if a parameter exists
if (params.has("discount")) {
  console.log("Discount applied!");
} else {
  console.log("No discount");
}

// Loop through all params
for (const [key, value] of params.entries()) {
  console.log(`${key} = ${value}`);
}

```

#### Summary:
- `new URLSearchParams(location.search)` creates an object to **read, add, delete, or modify** query parameters.
- It saves you from messy string manipulations.
- Works in all modern browsers.

---

### location.hash
`location.hash` Returns or sets the fragment identifier(anchor) of the URL, starting with a hash symbol(#).
#### Example:
```JavaScript
// Assuming current URL is https://www.example.com/page#section-2
console.log(location.hash); // Output: "#section-2"

location.hash = "#top"; // Scrolls to the element with id="top" on the current page
```

- **Note**: Changing `location.hash` does _not_ cause a full page reload; it just scrolls to the corresponding element on the page and adds an entry to the browser's history.
#### Common uses of `location.hash`:
1. **Navigating to a section on the page**
    - Browsers automatically scroll to the element with the matching `id`.
2. **Single Page Applications (SPAs) routing**
    - Hash-based routing uses the hash part of the URL to simulate navigation without reloading the page.
3. **Reading or modifying the hash with JavaScript**

---

### location.origin
`location.origin` Read-only used to returns the canonical form of the origin of the current document.
- The origin includes the protocol, hostname, and port.
- It's useful for security checks (Same-Origin Policy).
#### Example
```JavaScript
// Assuming current URL is https://www.example.com:8080/path
console.log(location.origin); // Output: "https://www.example.com:8080"

// Assuming current URL is http://localhost/
console.log(location.origin); // Output: "http://localhost"
```

---

## location Object Methods
### location.assign(url)
`location.assign(url)` loads a new document  at the specified URL.
- This method adds the new URL to the browser's history, so the user can use the "back" button to return to the previous page.

#### Example
```javascript
location.assign("https://www.mozilla.org"); // Navigates to Mozilla, adds to history
```

-  **Equivalent to**: Setting `location.href = url;`
- So both `assing(URL)` method and changing the `href = URL`  value will forward you to the new URL.    

### location.replace(URL)
`location.replace(URL)` Replaces the current document with a new one at the specified URL.
-  **Crucially, this method does NOT add the new URL to the browser's history.** This means the user cannot use the "back" button to return to the page that called `replace()`.

#### Example
```JavaScript
 location.replace("https://www.wikipedia.org"); // Navigates to Wikipedia, but current page is removed from history
```

- **Use Case**: Useful for preventing users from going back to a page they shouldn't (e.g., after a <span style="color:rgb(255, 200, 0)">successful form submission</span> or a <span style="color:rgb(255, 200, 0)">login page</span>).


----

### location.reload(forceGet)
`location.reload(forceGet)` Reloads the current document.
- **`forceGet` (optional)**: A boolean value.
    - If `true`, the browser bypasses its cache and reloads the page from the server.
    - If `false` (default), the browser might reload the page from its cache.

#### Example
```JavaScript
location.reload();         // Reloads, potentially from cache
location.reload(true);     // Forces a reload from the server
```

-----------------

## Important Considerations

- **Same-Origin Policy**: For security reasons, JavaScript running on one origin (protocol, hostname, port) cannot directly access the `location` object's properties of a document from a different origin loaded in an `iframe` or new window, except for `href`. Attempting to do so will result in a security error. However, you can _set_ `location.href` to navigate to a different origin.
- **Changing Properties vs. Methods**:
    - Setting `location.href`, `location.protocol`, `location.host`, `location.hostname`, `location.port`, `location.pathname`, or `location.search` will cause the browser to navigate to the new URL, similar to `location.assign()`.
    - Setting `location.hash` will _not_ cause a full page reload, only a scroll to the element and a history entry.
- **`URL` Interface**: For more robust URL parsing and manipulation, especially when you don't want to navigate the current page, the global `URL` interface is often preferred. You can create a `URL` object from a string and access its properties without affecting `window.location`.
    
    ```javascript
    const urlString = "https://example.com/path?param=value#hash";
    const url = new URL(urlString);
    console.log(url.hostname); // "example.com"
    console.log(url.searchParams.get('param')); // "value"
    ```
    

The `location` object is a powerful tool for controlling browser navigation and understanding the current page's address.