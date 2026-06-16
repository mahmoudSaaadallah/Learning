## REST Constraints: Building Robust Web Services

### 1. Introduction: What is REST and Why Constraints Matter

**REST (Representational State Transfer)** is an architectural style for designing networked applications. It's not a protocol or a standard, but rather a set of guidelines and principles that, when followed, lead to systems with desirable properties like scalability, simplicity, and modifiability.

The "constraints" are the core rules that define a RESTful system. Adhering to these constraints is what makes an API "RESTful." They were first introduced by Roy Fielding in his doctoral dissertation in 2000. Think of them as the blueprint for building a highly effective and interoperable web service.

**Why do these constraints matter?**
They provide a common language and understanding for how clients and servers interact, enabling independent evolution of components, better caching, and improved scalability. Violating these constraints often leads to "REST-ish" APIs that miss out on the full benefits of the architectural style.

### 2. Prerequisites: Understanding APIs and Client-Server Interaction

Before we dissect the constraints, let's quickly align on a couple of basic concepts:

*   **API (Application Programming Interface):** Essentially, a set of definitions and protocols for building and integrating application software. In the context of web services, it's how different software systems communicate with each other.
*   **Client-Server Interaction:** This is the fundamental model where a "client" (e.g., a web browser, a mobile app, another backend service) requests resources or services from a "server" (your .NET backend application), and the server responds.

With that in mind, let's explore the six core REST constraints.

### 3. The Six REST Constraints Explained

#### 3.1. Client-Server

**Basic Idea:** This constraint mandates a clear separation of concerns between the client and the server. The client is responsible for the user interface and user experience, while the server is responsible for data storage, business logic, and providing resources.

**Deeper Details:**
This separation allows client and server components to evolve independently. The client doesn't need to know about the server's internal data storage mechanisms, and the server doesn't need to worry about how the client renders the data. This promotes portability of the client code across multiple platforms and improves scalability by allowing server components to be deployed and managed separately.

**Practical Example (ASP.NET Core):**
Consider a typical ASP.NET Core Web API backend serving data to a React frontend.

```csharp
// Server-side (ASP.NET Core Web API Controller)
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        if (product == null)
        {
            return NotFound(); // Server handles data retrieval and business logic
        }
        return Ok(product); // Server returns data representation
    }
}

// Client-side (Conceptual JavaScript/TypeScript for a React app)
// async function fetchProduct(productId) {
//     const response = await fetch(`/api/products/${productId}`); // Client makes request
//     if (!response.ok) {
//         throw new Error(`HTTP error! status: ${response.status}`);
//     }
//     const product = await response.json(); // Client receives data
//     // Client then renders 'product' data in the UI
//     console.log(product);
// }
```

**Line-by-Line Explanation:**
*   `ProductsController`: This is a server-side component. It's responsible for handling HTTP requests related to products.
*   `IProductService`: An abstraction for business logic and data access. The controller delegates these responsibilities.
*   `GetProduct(int id)`: This method retrieves product data from the service.
*   `return NotFound()` / `return Ok(product)`: The server decides the outcome of the request and returns an appropriate HTTP status code and a data representation (e.g., JSON).
*   Client-side `fetchProduct`: The client simply makes an HTTP request to a known URL, receives the data, and then uses it to update its UI. It doesn't care how the server got the product or stored it.

**Common Mistakes:**
*   **Tight Coupling:** A client making assumptions about the server's internal database schema or file system.
*   **Shared UI Logic:** Trying to render UI components directly from the server (e.g., server-side rendering frameworks that blur the lines too much for a pure REST API).

**Senior Insight:**
"The client-server constraint is the bedrock of distributed systems. It forces a clean contract between the two parties. As a senior engineer, I look for clear boundaries. If a client needs to know too much about the server's internals, or vice-versa, it's a red flag for maintainability and scalability. This separation is key to microservices architectures, allowing teams to work on different parts of the system independently."

**Senior Considerations:**
*   **Scalability:** Decoupling allows independent scaling of client and server. You can add more server instances without affecting clients, and vice-versa.
*   **Maintainability:** Changes in the server's internal implementation (e.g., switching databases) don't necessarily require client changes, as long as the API contract remains stable.
*   **Security:** The server can enforce security policies without the client needing to understand the underlying mechanisms.

#### 3.2. Stateless

**Basic Idea:** Each request from a client to the server must contain all the information needed to understand and process the request. The server must not store any client context or session state between requests.

**Deeper Details:**
This is one of the most critical constraints for scalability. If a server doesn't need to remember anything about previous requests from a client, then any server instance can handle any request. This simplifies load balancing, improves reliability (if one server fails, another can pick up without losing client state), and makes the system easier to scale horizontally. Any state that needs to be maintained for a client must be sent by the client with each request (e.g., authentication tokens, pagination parameters).

**Practical Example (ASP.NET Core with JWT Authentication):**
Instead of server-side sessions, we use JWT (JSON Web Tokens).

```csharp
// Server-side (ASP.NET Core Web API)
// 1. User logs in, server issues a JWT
[HttpPost("login")]
public ActionResult Login([FromBody] LoginModel model)
{
    // ... validate credentials ...
    var token = _jwtService.GenerateToken(model.Username);
    return Ok(new { Token = token }); // Server sends token to client
}

// 2. Client includes JWT in subsequent requests
[Authorize] // This attribute ensures the request has a valid JWT
[HttpGet("profile")]
public ActionResult GetUserProfile()
{
    // The server extracts user identity from the JWT in the request header
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    // ... retrieve user profile based on userId ...
    return Ok(new { UserId = userId, Message = "Welcome to your profile!" });
}

// Client-side (Conceptual JavaScript/TypeScript)
// async function getProfile(token) {
//     const response = await fetch('/api/profile', {
//         headers: {
//             'Authorization': `Bearer ${token}` // Client sends token with each request
//         }
//     });
//     // ... handle response ...
// }
```

**Line-by-Line Explanation:**
*   `Login` endpoint: After successful authentication, the server generates a JWT and sends it to the client. The server *does not* store any session information about this login.
*   `[Authorize]` attribute: This middleware in ASP.NET Core intercepts requests. It expects an `Authorization` header with a `Bearer` token.
*   `User.FindFirst(ClaimTypes.NameIdentifier)?.Value`: The server extracts the user's identity directly from the JWT provided in the *current request*. It doesn't look up a session ID in its own memory or database.
*   Client-side `Authorization` header: The client is responsible for storing the JWT (e.g., in local storage) and sending it with every protected request.

**Common Mistakes:**
*   **Server-Side Sessions:** Using `HttpContext.Session` in ASP.NET Core for storing user-specific data between requests. While convenient for traditional web apps, it violates statelessness for a REST API.
*   **Sticky Sessions:** Relying on load balancers to always route a client's requests to the same server instance because that instance holds the client's state. This severely limits scalability.

**Senior Insight:**
"Statelessness is paramount for horizontal scalability. If your server needs to remember anything about a client between requests, you're creating a bottleneck. JWTs are a fantastic example of how to achieve stateless authentication. When I see session IDs being passed around and stored on the server in a REST API, my immediate thought is 'how will this scale under heavy load?'"

**Senior Considerations:**
*   **Scalability:** Enables easy horizontal scaling. Any server can handle any request, allowing you to add more instances behind a load balancer.
*   **Reliability:** No single point of failure due to session loss. If a server goes down, other servers can continue processing requests without interruption.
*   **Performance:** Reduces server memory footprint as no session data needs to be stored.
*   **Security:** Can simplify security by avoiding session fixation attacks if tokens are handled correctly.

#### 3.3. Cacheable

**Basic Idea:** Responses from the server must explicitly or implicitly define themselves as cacheable or non-cacheable. This allows clients and intermediaries (like proxies) to cache responses, reducing server load and improving perceived performance.

**Deeper Details:**
The server indicates whether a response can be cached and for how long using HTTP headers. If a client or proxy has a cached response, it can serve that response directly without contacting the origin server, provided the cached response is still valid. This is a powerful mechanism for improving efficiency.

**Practical Example (ASP.NET Core with Cache-Control and ETag):**

```csharp
// Server-side (ASP.NET Core Web API)
[HttpGet("{id}")]
public async Task<ActionResult<ProductDto>> GetProduct(int id)
{
    var product = await _productService.GetProductByIdAsync(id);
    if (product == null)
    {
        return NotFound();
    }

    // Generate an ETag based on the product's content or version
    var etag = $"\"{product.GetHashCode()}\""; // Simple example, use a proper hash/version in production

    // Check if the client sent an If-None-Match header with the same ETag
    if (Request.Headers.TryGetValue("If-None-Match", out var ifNoneMatch) && ifNoneMatch.Contains(etag))
    {
        return StatusCode(304); // Not Modified
    }

    Response.Headers.Add("Cache-Control", "public, max-age=300"); // Cache for 5 minutes
    Response.Headers.Add("ETag", etag); // Add ETag for conditional requests

    return Ok(product);
}
```

**Line-by-Line Explanation:**
*   `var etag = $"\"{product.GetHashCode()}\"";`: A simple ETag generation. In production, this would be a more robust hash of the resource's content or a version identifier.
*   `Request.Headers.TryGetValue("If-None-Match", ...)`: The server checks if the client sent an `If-None-Match` header. If the client has a cached version, it sends the ETag of that version.
*   `return StatusCode(304);`: If the client's ETag matches the current ETag on the server, it means the resource hasn't changed. The server responds with `304 Not Modified`, telling the client to use its cached version. This saves bandwidth and server processing.
*   `Response.Headers.Add("Cache-Control", "public, max-age=300");`: This header tells clients and proxies that the response can be cached for 300 seconds (5 minutes) and is "public" (can be cached by shared caches).
*   `Response.Headers.Add("ETag", etag);`: The server includes the ETag in the response, so clients can use it for future conditional requests.

**Common Mistakes:**
*   **Not Using Caching Headers:** Failing to include `Cache-Control`, `Expires`, `ETag`, or `Last-Modified` headers, leading to clients always fetching fresh data unnecessarily.
*   **Caching Sensitive Data:** Accidentally caching user-specific or highly sensitive information, which could lead to data leakage.
*   **Incorrect Cache Invalidation:** Not having a strategy to invalidate cached data when the underlying resource changes, leading to stale data being served.

**Senior Insight:**
"Caching is a performance superpower, but it's also a source of some of the hardest bugs ('cache invalidation is one of the two hard problems in computer science'). As a senior, I always think about the cacheability of every endpoint. For read-heavy operations, proper caching can drastically reduce database load and improve response times. For write operations, it's usually 'no-cache' or very short expiry. Always consider the freshness requirements of the data."

**Senior Considerations:**
*   **Performance:** Significantly reduces latency and server load for frequently accessed, static, or semi-static resources.
*   **Scalability:** Allows the system to handle more requests with the same server resources.
*   **Data Freshness:** Requires careful consideration of `max-age`, `ETag`, and `Last-Modified` to ensure clients don't receive stale data.
*   **Security:** Be extremely cautious about caching authenticated or sensitive data. Use `private` or `no-store` for such responses.

#### 3.4. Uniform Interface

**Basic Idea:** This is the most fundamental and critical constraint. It simplifies the overall system architecture by having a single, consistent way of interacting with resources, regardless of their type or the underlying server implementation.

**Deeper Details:**
A uniform interface decouples the client from the server, allowing each to evolve independently. It's achieved through four sub-constraints:

##### 3.4.1. Identification of Resources

**Basic Idea:** Individual resources are identified in requests using URIs (Uniform Resource Identifiers).

**Practical Example:**
*   `GET /api/products/123` - Identifies a specific product.
*   `GET /api/orders` - Identifies the collection of all orders.
*   `POST /api/users` - Identifies the collection where new users can be added.

**Senior Insight:** "Clear, predictable URIs are essential. They should represent nouns (resources), not verbs (actions). Avoid `GET /api/getProduct?id=123` in favor of `GET /api/products/123`. This makes the API intuitive and self-documenting."

##### 3.4.2. Manipulation of Resources Through Representations

**Basic Idea:** Clients interact with resources by exchanging representations of those resources. When a client requests a resource, the server sends a representation (e.g., JSON, XML, HTML). To modify a resource, the client sends a representation of the desired state to the server.

**Practical Example (ASP.NET Core):**

```csharp
// Server-side (ASP.NET Core Web API)
// GET request returns a JSON representation
[HttpGet("{id}")]
public ActionResult<ProductDto> GetProduct(int id)
{
    // ... returns ProductDto as JSON ...
}

// PUT request sends a JSON representation to update
[HttpPut("{id}")]
public async Task<ActionResult> UpdateProduct(int id, [FromBody] UpdateProductDto productDto)
{
    // ... server receives JSON, maps to internal model, updates ...
    return NoContent(); // 204 No Content
}

// Client-side (Conceptual JavaScript/TypeScript)
// async function updateProduct(productId, newProductData) {
//     const response = await fetch(`/api/products/${productId}`, {
//         method: 'PUT',
//         headers: { 'Content-Type': 'application/json' },
//         body: JSON.stringify(newProductData) // Client sends JSON representation
//     });
//     // ...
// }
```

**Senior Insight:** "The choice of representation format (JSON, XML) is a contract. JSON is almost always preferred in modern APIs due to its lightweight nature and widespread support. Ensure your API consistently uses the agreed-upon format for both requests and responses."

##### 3.4.3. Self-Descriptive Messages

**Basic Idea:** Each message (request or response) contains enough information to describe how to process the message. This means using standard HTTP methods, status codes, and media types.

**Practical Example (ASP.NET Core):**

```csharp
// Server-side (ASP.NET Core Web API)
[HttpPost] // HTTP POST method indicates creation
public ActionResult<ProductDto> CreateProduct([FromBody] CreateProductDto productDto)
{
    // ... create product ...
    var newProduct = _productService.AddProduct(productDto);
    return CreatedAtAction(nameof(GetProduct), new { id = newProduct.Id }, newProduct);
    // 201 Created status code, Location header, and JSON body
}

// If a resource is not found
[HttpGet("{id}")]
public ActionResult<ProductDto> GetProduct(int id)
{
    // ...
    if (product == null)
    {
        return NotFound(); // 404 Not Found status code
    }
    // ...
}
```

**Line-by-Line Explanation:**
*   `[HttpPost]`: The HTTP method itself (`POST`) tells the client that this endpoint is for creating a new resource.
*   `CreatedAtAction(...)`: This helper method in ASP.NET Core automatically sets the HTTP status code to `201 Created` and adds a `Location` header pointing to the URI of the newly created resource. This is self-descriptive.
*   `NotFound()`: Returns `404 Not Found`, clearly indicating that the requested resource does not exist.
*   `Content-Type: application/json`: (Implicitly handled by ASP.NET Core) The `Content-Type` header in the response tells the client how to interpret the body.

**Common Mistakes:**
*   **Misusing HTTP Methods:** Using `GET` to change server-side state, or `POST` for idempotent updates.
*   **Generic Status Codes:** Always returning `200 OK` even for errors, and putting error details in the response body. This breaks standard HTTP semantics.
*   **Missing Content-Type:** Not specifying the media type of the request/response body.

**Senior Insight:** "HTTP methods and status codes are your API's vocabulary. Use them correctly and consistently. A `GET` should *never* have side effects. A `201 Created` should always include a `Location` header. This adherence to standards makes your API predictable and easy for any client to consume, even without specific documentation."

##### 3.4.4. HATEOAS (Hypermedia As The Engine Of Application State)

**Basic Idea:** Resources should contain links to related resources, guiding the client on available actions and transitions. The client discovers available actions by following these links, rather than hardcoding URIs.

**Deeper Details:**
This is often considered the most challenging and least implemented REST constraint. It means that a client, upon receiving a resource representation, should be able to discover what other actions it can perform or what other resources it can access by examining the links embedded within that representation. This makes the API truly self-discoverable and allows the server to evolve its URI structure without breaking clients.

**Practical Example (ASP.NET Core - simplified):**

```csharp
// Server-side (ASP.NET Core Web API)
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public List<LinkDto> Links { get; set; } = new List<LinkDto>();
}

public class LinkDto
{
    public string Href { get; set; }
    public string Rel { get; set; } // Relationship (e.g., "self", "update", "delete")
    public string Method { get; set; } // HTTP method (e.g., "GET", "PUT", "DELETE")
}

[HttpGet("{id}")]
public ActionResult<ProductDto> GetProduct(int id)
{
    var product = _productService.GetProductById(id);
    if (product == null) return NotFound();

    var productDto = new ProductDto
    {
        Id = product.Id,
        Name = product.Name,
        Price = product.Price
    };

    // Add HATEOAS links
    productDto.Links.Add(new LinkDto { Href = Url.Link(nameof(GetProduct), new { id = product.Id }), Rel = "self", Method = "GET" });
    productDto.Links.Add(new LinkDto { Href = Url.Link(nameof(UpdateProduct), new { id = product.Id }), Rel = "update", Method = "PUT" });
    productDto.Links.Add(new LinkDto { Href = Url.Link(nameof(DeleteProduct), new { id = product.Id }), Rel = "delete", Method = "DELETE" });

    return Ok(productDto);
}
```

**Line-by-Line Explanation:**
*   `ProductDto` now includes a `Links` property.
*   `LinkDto`: A simple class to represent a hypermedia link, including the URI (`Href`), the relationship (`Rel`), and the HTTP method (`Method`).
*   `Url.Link(...)`: ASP.NET Core's `UrlHelper` can generate URIs based on route names, making it easier to create links dynamically.
*   The response for a product now includes links like:
    ```json
    {
      "id": 1,
      "name": "Laptop",
      "price": 1200.00,
      "links": [
        { "href": "/api/products/1", "rel": "self", "method": "GET" },
        { "href": "/api/products/1", "rel": "update", "method": "PUT" },
        { "href": "/api/products/1", "rel": "delete", "method": "DELETE" }
      ]
    }
    ```
    A client would receive this and know it can `PUT` to `/api/products/1` to update it, or `DELETE` to `/api/products/1` to delete it, without needing to hardcode these URLs.

**Common Mistakes:**
*   **Ignoring HATEOAS:** Most APIs are "Level 2" REST (using HTTP methods and status codes correctly) but don't implement HATEOAS, making them less flexible.
*   **Hardcoding Links:** Generating links manually instead of using URL helpers, which can lead to broken links if routes change.

**Senior Insight:**
"HATEOAS is the holy grail of REST. It's what truly decouples the client from the server's URI structure. While often overlooked due to its complexity, a truly HATEOAS-driven API is incredibly resilient to change. The client becomes a 'state machine' navigating the API based on the links provided, rather than a 'hardcoded client' that breaks when a URL changes. For internal APIs, it might be overkill, but for public-facing APIs, it's a mark of maturity."

**Senior Considerations (Uniform Interface as a whole):**
*   **Maintainability:** A consistent interface reduces the learning curve for new developers and makes the API easier to maintain.
*   **Interoperability:** Standardized interactions allow a wide range of clients (browsers, mobile apps, other services) to consume the API.
*   **Evolvability:** HATEOAS, in particular, allows the server to change its URI structure without breaking existing clients, as clients discover links dynamically.

#### 3.5. Layered System

**Basic Idea:** A client cannot ordinarily tell whether it is connected directly to the end server, or to an intermediary along the way. This allows for the introduction of intermediate layers (proxies, load balancers, API gateways) without affecting the client or the server.

**Deeper Details:**
This constraint promotes architectural flexibility. You can add layers like caching servers, security proxies, or load balancers between the client and the actual backend server. Each layer can provide additional functionality (e.g., shared caching, security policy enforcement) without the client or the server needing to be aware of its presence.

**Practical Example (ASP.NET Core with an API Gateway):**
Imagine your ASP.NET Core Web API is deployed behind an API Gateway (like Ocelot, Azure API Management, or AWS API Gateway).

```
Client (Mobile App)
      |
      V
API Gateway (e.g., Ocelot)
      | (Routes /api/products to Product Service)
      V
Product Service (ASP.NET Core Web API)
      | (Communicates with database)
      V
Database (SQL Server)
```

**Senior Insight:**
"Layered systems are fundamental to modern distributed architectures. They allow us to introduce cross-cutting concerns like authentication, rate limiting, logging, and caching at different levels without polluting our core business logic. An API Gateway is a prime example, acting as a single entry point for clients while routing requests to various backend microservices. This improves security, scalability, and simplifies client interaction."

**Senior Considerations:**
*   **Scalability:** Layers like load balancers distribute traffic, enabling horizontal scaling.
*   **Security:** Firewalls and API gateways can enforce security policies, perform authentication/authorization, and protect backend services.
*   **Maintainability:** Each layer can be developed, deployed, and managed independently.
*   **Performance:** Caching layers can significantly improve response times.

#### 3.6. Code-On-Demand (Optional)

**Basic Idea:** Servers can temporarily extend or customize the functionality of a client by transferring executable code (e.g., JavaScript applets, compiled components).

**Deeper Details:**
This is the only optional constraint. While common in traditional web applications (e.g., a browser downloading JavaScript to enhance its UI), it's less frequently applied to pure backend REST APIs. It allows for dynamic client-side logic updates without requiring a full client application redeployment.

**Practical Example:**
*   A web browser downloading and executing JavaScript from a server to render a dynamic form or perform client-side validation.
*   A mobile app downloading a small script to update a specific UI component's behavior without a full app store update.

**Senior Insight:**
"For most backend APIs, Code-On-Demand isn't a primary concern. Our focus is usually on providing data and services, not executable code. However, it's important to recognize its role in the broader web ecosystem, especially for rich client applications. If you're building a backend that serves a dynamic frontend, the frontend's JavaScript is essentially 'code-on-demand' from the perspective of the browser."

**Senior Considerations:**
*   **Security:** Transferring executable code always introduces security risks. Code must be trusted and properly sandboxed.
*   **Performance:** Downloading and executing code adds overhead.
*   **Complexity:** Managing and versioning client-side code delivered this way can be complex.

### 4. When to Use REST (and When Not To)

**When to Use REST:**
*   **Public APIs:** When you need a widely adopted, interoperable, and flexible API for external consumption.
*   **Resource-Oriented Systems:** When your system naturally models as a collection of resources that can be created, read, updated, and deleted (CRUD operations).
*   **Scalable Web Services:** When you need to build highly scalable services that can be easily distributed and cached.
*   **Decoupled Systems:** When you want to ensure a clear separation between client and server, allowing independent evolution.

**When Not to Use REST (or consider alternatives):**
*   **High-Performance, Low-Latency Communication:** For scenarios requiring extremely low latency or real-time communication (e.g., gaming, financial trading), protocols like gRPC or WebSockets might be more suitable due to their binary serialization and persistent connections.
*   **Complex Graph-Like Data Queries:** If clients frequently need to fetch highly interconnected data with varying structures in a single request, GraphQL might offer more flexibility by allowing clients to specify exactly what data they need.
*   **Event-Driven Architectures:** For internal service-to-service communication where services react to events, message queues and event buses are often preferred over synchronous REST calls.
*   **RPC-Style Operations:** If your operations are more command-oriented (e.g., "processOrder," "generateReport") rather than resource-oriented, a pure REST approach might feel forced. However, even then, you can often model these as resources (e.g., a "report" resource that you `POST` to initiate generation).

### 5. Summary

REST constraints are a powerful set of architectural principles that guide the design of robust, scalable, and maintainable web services. By adhering to **Client-Server separation**, ensuring **Statelessness**, enabling **Cacheability**, maintaining a **Uniform Interface** (with its sub-constraints of resource identification, manipulation through representations, self-descriptive messages, and HATEOAS), and building a **Layered System**, you create APIs that are resilient, performant, and easy to consume. While Code-On-Demand is optional, the other five constraints are fundamental to achieving true RESTfulness. Understanding and applying these principles is a hallmark of a strong backend developer.

### 6. Practical Exercise

**Scenario:** You are building a simple e-commerce API using ASP.NET Core. You need to manage customer orders.

**Task:** Design the API endpoints for managing orders, keeping the REST constraints in mind. Specifically:

1.  **Create a new order:**
    *   Which HTTP method would you use?
    *   What URI would represent this action?
    *   What HTTP status code should be returned on success, and what header should accompany it?
    *   How would you ensure the response is self-descriptive?
2.  **Retrieve a specific order by its ID:**
    *   Which HTTP method?
    *   What URI?
    *   How would you make this endpoint cacheable for 1 minute, and how would you support conditional requests (e.g., using `If-None-Match`)?
    *   What status code for "not found"?
3.  **Update an existing order:**
    *   Which HTTP method?
    *   What URI?
    *   What status code for success (no content returned)?
4.  **Consider Statelessness:** If a client needs to know if an order is "pending" or "shipped," where would this state information reside in a RESTful interaction? (Hint: not on the server's session).
5.  **HATEOAS (Bonus):** For the `GET /api/orders/{id}` response, what links might you include to guide the client on further actions (e.g., "cancel order", "view customer details")?

Think about the structure of your `OrderDto` and `CreateOrderDto` classes, and how your ASP.NET Core controller methods would implement these. You don't need to write full service logic, just the controller actions and the relevant HTTP responses/headers.