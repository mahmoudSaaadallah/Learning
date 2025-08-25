# Location
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
