### HTTP Status Codes: The Language of the Web

HTTP status codes are three-digit numbers returned by a server in response to an HTTP request. They are part of the HTTP response header and indicate the outcome of the request. Think of them as a standardized way for your API to tell the client: "Here's what happened with your request."

They are grouped into five classes, determined by the first digit:

*   **`1xx` Informational:** The request was received, continuing process. (Rarely used in typical API responses).
*   **`2xx` Success:** The request was successfully received, understood, and accepted.
*   **`3xx` Redirection:** Further action needs to be taken by the user agent to fulfill the request.
*   **`4xx` Client Error:** The request contains bad syntax or cannot be fulfilled.
*   **`5xx` Server Error:** The server failed to fulfill an apparently valid request.

Let's break down the most common and important ones you'll use in your ASP.NET Core Web APIs.

---

### `2xx` Success Codes

These codes indicate that the client's request was successfully processed.

#### `200 OK`
*   **Meaning:** The request has succeeded. This is the most common success code.
*   **When to Use:**
    *   For successful `GET` requests where a resource is retrieved.
    *   For successful `PUT` or `POST` requests where the resource is updated or created, and you want to return the updated/created resource in the response body.
    *   For successful `DELETE` requests where you might return a confirmation message.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _productRepository.GetById(id);
        if (product == null)
        {
            return NotFound(); // Returns 404
        }
        return Ok(product); // Returns 200 with product in body
    }
    ```

#### `201 Created`
*   **Meaning:** The request has been fulfilled and resulted in a new resource being created. The response typically includes a `Location` header pointing to the URI of the newly created resource, and often the resource itself in the response body.
*   **When to Use:**
    *   Exclusively for successful `POST` requests that result in the creation of a new resource.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpPost]
    public ActionResult<Product> CreateProduct(Product newProduct)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState); // Returns 400
        }
        _productRepository.Add(newProduct);
        // The Location header should point to the URI of the newly created resource
        return CreatedAtAction(nameof(GetProduct), new { id = newProduct.Id }, newProduct);
        // This returns 201, sets Location header, and includes newProduct in body
    }
    ```

#### `204 No Content`
*   **Meaning:** The server successfully processed the request, but is not returning any content. This is useful for operations where the client doesn't need to know anything specific about the outcome beyond success.
*   **When to Use:**
    *   For successful `DELETE` requests where you don't need to return the deleted item or a confirmation message.
    *   For successful `PUT` or `POST` requests where the resource is updated/created, but the client doesn't need the resource back in the response body.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpDelete("{id}")]
    public IActionResult DeleteProduct(int id)
    {
        var product = _productRepository.GetById(id);
        if (product == null)
        {
            return NotFound(); // Returns 404
        }
        _productRepository.Delete(product);
        return NoContent(); // Returns 204
    }
    ```

#### `202 Accepted`
*   **Meaning:** The request has been accepted for processing, but the processing has not been completed. The request might or might not be acted upon, and may be disallowed when processing occurs. This is often used for asynchronous operations.
*   **When to Use:**
    *   When a request initiates a long-running process (e.g., generating a report, processing a large file) that will complete later. The client can then poll another endpoint to check the status.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpPost("process-large-file")]
    public IActionResult ProcessLargeFile([FromForm] IFormFile file)
    {
        // Start a background task to process the file
        _backgroundProcessingService.EnqueueFileProcessing(file);
        // Return a status URL where the client can check progress
        return Accepted($"/api/status/{Guid.NewGuid()}"); // Returns 202
    }
    ```

---

### `3xx` Redirection Codes

These codes indicate that the client needs to take further action to complete the request, usually by redirecting to a different URL. Less common in typical REST APIs, but important for web applications.

#### `301 Moved Permanently`
*   **Meaning:** The requested resource has been permanently moved to a new URI. Clients should update their links.
*   **When to Use:**
    *   When a resource's URI has changed permanently. Search engines will update their indexes.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpGet("old-product-path/{id}")]
    public IActionResult OldProductPath(int id)
    {
        // Assuming the new path is /api/products/{id}
        return RedirectPermanent($"/api/products/{id}"); // Returns 301
    }
    ```

#### `302 Found` (or `303 See Other`, `307 Temporary Redirect`)
*   **Meaning:** The requested resource resides temporarily under a different URI. Clients should continue to use the original URI for future requests.
*   **When to Use:**
    *   Temporary redirects, often after a `POST` request to prevent resubmission on refresh (Post/Redirect/Get pattern).
*   **ASP.NET Core Example:**
    ```csharp
    [HttpPost("submit-form")]
    public IActionResult SubmitForm(MyFormModel model)
    {
        // Process form data
        // ...
        return RedirectToAction(nameof(ConfirmationPage)); // Returns 302
    }
    ```

#### `304 Not Modified`
*   **Meaning:** The resource has not been modified since the version specified by the request headers (`If-Modified-Since` or `If-None-Match`). The client can use its cached copy.
*   **When to Use:**
    *   For `GET` requests when implementing client-side caching. The server checks if the resource has changed and, if not, tells the client to use its cached version.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpGet("{id}")]
    public IActionResult GetProductWithCaching(int id)
    {
        var product = _productRepository.GetById(id);
        if (product == null) return NotFound();

        // Example: Check If-Modified-Since header
        if (Request.Headers.ContainsKey("If-Modified-Since"))
        {
            // In a real scenario, you'd compare this with the product's last modified date
            // For simplicity, let's assume it hasn't changed
            return StatusCode(304); // Returns 304
        }
        return Ok(product);
    }
    ```

---

### `4xx` Client Error Codes

These codes indicate that the client made a mistake or sent an invalid request.

#### `400 Bad Request`
*   **Meaning:** The server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax, invalid request message framing, or deceptive request routing).
*   **When to Use:**
    *   When request body data fails validation (e.g., a required field is missing, a number is out of range).
    *   When query parameters are invalid or missing.
    *   When the request payload is malformed JSON/XML.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpPost]
    public IActionResult CreateProduct(Product newProduct)
    {
        if (!ModelState.IsValid)
        {
            // ASP.NET Core automatically returns 400 with validation errors
            // if you use [ApiController] attribute.
            return BadRequest(ModelState); // Returns 400 with validation details
        }
        // ...
        return CreatedAtAction(nameof(GetProduct), new { id = newProduct.Id }, newProduct);
    }
    ```

#### `401 Unauthorized`
*   **Meaning:** The request has not been applied because it lacks valid authentication credentials for the target resource.
*   **When to Use:**
    *   When the client tries to access a protected resource without providing any authentication token (e.g., JWT) or with an invalid/expired token.
*   **ASP.NET Core Example:**
    ```csharp
    [Authorize] // Requires authentication
    [HttpGet("protected-data")]
    public IActionResult GetProtectedData()
    {
        // If authentication fails, ASP.NET Core's middleware will typically
        // return 401 before reaching this action.
        return Ok("This is protected data.");
    }
    // You can explicitly return it too:
    // return Unauthorized();
    ```

#### `403 Forbidden`
*   **Meaning:** The server understood the request but refuses to authorize it. Unlike `401`, the client's authentication credentials *are* valid, but they don't have the necessary permissions to access the resource.
*   **When to Use:**
    *   When an authenticated user tries to access a resource they are not authorized for (e.g., a regular user trying to access an admin-only endpoint).
*   **ASP.NET Core Example:**
    ```csharp
    [Authorize(Roles = "Admin")] // Requires 'Admin' role
    [HttpGet("admin-data")]
    public IActionResult GetAdminData()
    {
        // If authentication succeeds but authorization (role check) fails,
        // ASP.NET Core's middleware will typically return 403.
        return Ok("This is admin data.");
    }
    // You can explicitly return it too:
    // return Forbid();
    ```

#### `404 Not Found`
*   **Meaning:** The server cannot find the requested resource.
*   **When to Use:**
    *   When a `GET` request is made for a resource that does not exist (e.g., `/api/products/999` where product 999 doesn't exist).
    *   When a `PUT` or `DELETE` request targets a resource that doesn't exist.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _productRepository.GetById(id);
        if (product == null)
        {
            return NotFound(); // Returns 404
        }
        return Ok(product);
    }
    ```

#### `405 Method Not Allowed`
*   **Meaning:** The HTTP method used in the request is not supported for the resource identified by the URI.
*   **When to Use:**
    *   If a client tries to `POST` to an endpoint that only supports `GET` (e.g., trying to create a resource at `/api/products/1` instead of `/api/products`). ASP.NET Core handles this automatically.
*   **ASP.NET Core Example:**
    ```csharp
    // If you only have a [HttpGet] for /api/products/{id}
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id) { /* ... */ return Ok(); }

    // And a client tries to POST to /api/products/1, ASP.NET Core will automatically
    // return 405 Method Not Allowed.
    ```

#### `409 Conflict`
*   **Meaning:** The request could not be completed due to a conflict with the current state of the target resource.
*   **When to Use:**
    *   When trying to create a resource that already exists and uniqueness is enforced (e.g., creating a user with an email that's already registered).
    *   Optimistic concurrency control: when trying to update a resource that has been modified by another client since it was last retrieved.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpPost]
    public IActionResult RegisterUser(User newUser)
    {
        if (_userRepository.UserExists(newUser.Email))
        {
            return Conflict("A user with this email already exists."); // Returns 409
        }
        _userRepository.Add(newUser);
        return CreatedAtAction(nameof(GetUser), new { id = newUser.Id }, newUser);
    }
    ```

#### `422 Unprocessable Entity`
*   **Meaning:** The server understands the content type of the request entity, and the syntax of the request entity is correct, but it was unable to process the contained instructions. This is often used for semantic validation errors.
*   **When to Use:**
    *   When the request body is syntactically correct (e.g., valid JSON) but contains logical errors that prevent processing (e.g., a password that doesn't meet complexity requirements, or a date range where the end date is before the start date). It's a more specific `400 Bad Request`.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpPost("order")]
    public IActionResult PlaceOrder(OrderDto order)
    {
        if (order.Quantity <= 0)
        {
            // Using ValidationProblem for consistency with ASP.NET Core's default 400
            // but explicitly returning 422 for semantic errors.
            ModelState.AddModelError(nameof(order.Quantity), "Quantity must be greater than zero.");
            return UnprocessableEntity(new ValidationProblemDetails(ModelState)); // Returns 422
        }
        // ... process order
        return Ok("Order placed.");
    }
    ```

---

### `5xx` Server Error Codes

These codes indicate that the server encountered an unexpected condition that prevented it from fulfilling the request.

#### `500 Internal Server Error`
*   **Meaning:** A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.
*   **When to Use:**
    *   This is the default fallback for unhandled exceptions in your API. You generally don't explicitly return `500` unless you catch an exception and decide to return a generic error.
    *   It signifies that something went wrong on the server's side that wasn't the client's fault.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpGet("trigger-error")]
    public IActionResult TriggerError()
    {
        // This will cause an unhandled exception, which ASP.NET Core's
        // exception handling middleware will typically catch and return a 500.
        throw new InvalidOperationException("Something went terribly wrong!");
        // If you wanted to explicitly return it (less common):
        // return StatusCode(500, "An unexpected error occurred.");
    }
    ```

#### `501 Not Implemented`
*   **Meaning:** The server does not support the functionality required to fulfill the request.
*   **When to Use:**
    *   When an endpoint or a specific feature is defined in your API contract (e.g., OpenAPI spec) but hasn't been implemented yet.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpPut("{id}")]
    public IActionResult UpdateProduct(int id, Product updatedProduct)
    {
        // Feature not yet implemented
        return StatusCode(501, "Product update functionality is not yet available."); // Returns 501
    }
    ```

#### `503 Service Unavailable`
*   **Meaning:** The server is currently unable to handle the request due to a temporary overload or scheduled maintenance, which will likely be alleviated after some delay.
*   **When to Use:**
    *   During planned maintenance windows.
    *   When your service is under extreme load and needs to shed requests gracefully.
    *   Often accompanied by a `Retry-After` header.
*   **ASP.NET Core Example:**
    ```csharp
    [HttpGet("health")]
    public IActionResult GetHealth()
    {
        // Imagine a scenario where a critical dependency is down
        if (!_dependencyService.IsHealthy())
        {
            Response.Headers.Add("Retry-After", "60"); // Retry in 60 seconds
            return StatusCode(503, "Service is temporarily unavailable due to dependency issues."); // Returns 503
        }
        return Ok("Service is healthy.");
    }
    ```

---

### Senior Insight

1.  **Consistency is King:** The most important rule is to be consistent across your API. If `POST` to `/users` returns `201 Created` with the new user, ensure all other `POST` operations for resource creation follow this pattern. Inconsistency leads to confusion for API consumers.

2.  **Use `ProblemDetails` for Errors:** For `4xx` and `5xx` errors, especially `400`, `401`, `403`, `404`, `409`, and `422`, ASP.NET Core provides `ProblemDetails` (and `ValidationProblemDetails` for validation errors). This is a standardized way (RFC 7807) to convey error details in a machine-readable format.
    *   **Example:**
        ```json
        {
          "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
          "title": "Bad Request",
          "status": 400,
          "detail": "One or more validation errors occurred.",
          "instance": "/api/products",
          "errors": {
            "Name": [
              "The Name field is required."
            ],
            "Price": [
              "The Price field must be greater than 0."
            ]
          }
        }
        ```
    *   ASP.NET Core's `[ApiController]` attribute automatically handles `ModelState` validation errors and returns a `400 Bad Request` with `ValidationProblemDetails`. For custom `4xx` errors, you can return `BadRequest(new ProblemDetails { ... })` or `StatusCode(409, new ProblemDetails { ... })`.

3.  **Don't Leak Sensitive Information in `5xx` Errors:** When a `500 Internal Server Error` occurs, ensure your production environment doesn't expose stack traces or other sensitive internal details to the client. Use a generic error message and log the full details internally for debugging. ASP.NET Core's default error handling middleware is configurable for this.

4.  **Idempotency and Status Codes:**
    *   An operation is **idempotent** if it can be applied multiple times without changing the result beyond the initial application.
    *   `GET`, `PUT`, `DELETE` are generally considered idempotent. `POST` is generally not.
    *   When a `PUT` request (which is idempotent) tries to create a resource that already exists, a `200 OK` or `204 No Content` is often appropriate if the resource state matches the request. If the `PUT` is meant to *replace* a resource and it doesn't exist, `201 Created` is also valid. If it's meant to *update* and the resource doesn't exist, `404 Not Found` is correct.
    *   For `POST` requests, if you try to create a resource that already exists and your business logic dictates it's a conflict, `409 Conflict` is the right choice.

5.  **Document Your Status Codes:** Use tools like Swagger/OpenAPI (via Swashbuckle or NSwag in ASP.NET Core) to explicitly document the expected status codes for each endpoint. This helps API consumers understand how to interact with your API and handle different outcomes.
    *   **Example using Swashbuckle annotations:**
        ```csharp
        /// <summary>
        /// Gets a product by its ID.
        /// </summary>
        /// <param name="id">The ID of the product.</param>
        /// <returns>A product object.</returns>
        /// <response code="200">Returns the product.</response>
        /// <response code="404">If the product is not found.</response>
        [HttpGet("{id}")]
        [ProducesResponseType(typeof(Product), StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status404NotFound)]
        public ActionResult<Product> GetProduct(int id) { /* ... */ }
        ```

6.  **Middleware for Global Error Handling:** Implement global exception handling middleware (e.g., `UseExceptionHandler` or custom middleware) to catch unhandled exceptions and return appropriate `500 Internal Server Error` responses consistently, often with `ProblemDetails`. This prevents your API from crashing and provides a uniform error experience.

By mastering these status codes and applying these senior insights, you'll build APIs that are not only functional but also highly usable, maintainable, and robust. Keep practicing, and always think about the client's perspective when designing your API responses!