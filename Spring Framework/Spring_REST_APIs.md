# REST APIs - Interview Questions & Answers

---

## 1. What is REST?

**REST (Representational State Transfer)** is an **architectural style** for designing distributed web services. It defines a set of constraints and principles for building scalable, stateless, and loosely coupled web APIs over HTTP. A web service that follows REST principles is called a **RESTful Web Service**.

| REST Constraint | Description |
|---|---|
| **Client-Server** | Client and server are independent — separated by a uniform interface |
| **Stateless** | Each request must contain all information needed — server stores no client session state |
| **Cacheable** | Responses must define whether they are cacheable to improve performance |
| **Uniform Interface** | Consistent resource identification via URIs, standard HTTP methods |
| **Layered System** | Client doesn't know if it's talking directly to the server or through intermediaries (proxy, gateway) |
| **Code on Demand** *(Optional)* | Server can send executable code to client (e.g., JavaScript) |

**Real World Analogy:** REST is like ordering from a restaurant menu. The menu (API) defines what's available. You (client) place an order (request) with all details. The kitchen (server) prepares it without remembering your past orders (stateless). You get your food (response).

---

## 2. Key Concepts in REST API Design?

| Concept | Description | Example |
|---|---|---|
| **Resource** | Any entity exposed via the API | User, Product, Order |
| **URI** | Unique address to identify a resource | `/api/users/42` |
| **HTTP Methods** | Define the action to perform on a resource | `GET`, `POST`, `PUT`, `DELETE`, `PATCH` |
| **HTTP Status Codes** | Indicate the result of the request | `200 OK`, `201 Created`, `404 Not Found` |
| **Representation** | Format of the resource data | JSON, XML |
| **Statelessness** | Server does not store client session between requests | Each request is self-contained |
| **HATEOAS** | Responses include links to related actions | `{ "links": [{"rel": "self", "href": "/users/1"}] }` |

### HTTP Methods and Their Purpose:

| Method | Operation | Idempotent? | Safe? |
|---|---|---|---|
| `GET` | Read / Fetch a resource | ✅ Yes | ✅ Yes |
| `POST` | Create a new resource | ❌ No | ❌ No |
| `PUT` | Replace a resource entirely | ✅ Yes | ❌ No |
| `PATCH` | Partially update a resource | ✅ Yes | ❌ No |
| `DELETE` | Remove a resource | ✅ Yes | ❌ No |

---

## 3. Best Practices of REST?

| Best Practice | Right Way | Wrong Way |
|---|---|---|
| **Use nouns in URIs** | `/api/users` | `/api/getUsers` |
| **Use plural nouns** | `/api/products` | `/api/product` |
| **Use HTTP methods correctly** | `DELETE /api/users/1` | `GET /api/deleteUser?id=1` |
| **Use proper HTTP status codes** | `201 Created` for POST | `200 OK` for everything |
| **Version your API** | `/api/v1/users` | `/api/users` (no version) |
| **Use lowercase URIs** | `/api/user-orders` | `/api/UserOrders` |
| **Use hyphens not underscores** | `/api/user-profiles` | `/api/user_profiles` |
| **Consistent error responses** | Standard error JSON body | Plain text errors |
| **Paginate large collections** | `/api/users?page=1&size=20` | Return all records at once |
| **Secure APIs** | HTTPS + JWT/OAuth2 | HTTP with no auth |
| **Use HATEOAS** | Include navigation links | Client hardcodes URLs |
| **Filter/Sort via query params** | `/api/users?role=admin&sort=name` | In request body |

---

## 4. Example GET Method?

```java
@RestController                         // @Controller + @ResponseBody
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    // GET all users
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.findAll();
        return ResponseEntity.ok(users);    // 200 OK + body
    }

    // GET user by ID
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable int id) {
        User user = userService.findById(id);
        if (user == null) {
            return ResponseEntity.notFound().build(); // 404 Not Found
        }
        return ResponseEntity.ok(user);              // 200 OK + user JSON
    }

    // GET users with filter
    @GetMapping("/search")
    public ResponseEntity<List<User>> searchUsers(
            @RequestParam String name,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return ResponseEntity.ok(userService.search(name, page, size));
    }
}
```

**Sample Response:**
```json
// GET /api/users/1
{
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "role": "ADMIN"
}
```

---

## 5. What Happens When Returning a Bean?

When a Spring `@RestController` method returns a **Java bean (POJO)**, Spring automatically **serializes it to JSON** using **Jackson** (`jackson-databind`) — which is auto-configured when `spring-boot-starter-web` is on the classpath.

```java
public class User {
    private int id;
    private String name;
    private String email;
    // getters and setters (required for Jackson serialization)
}

@RestController
public class UserController {

    @GetMapping("/user")
    public User getUser() {
        return new User(1, "Alice", "alice@example.com");
        // Spring/Jackson converts this → JSON automatically
    }
}
```

**What Spring does internally:**
```
Java Bean (User object)
    ↓ Jackson ObjectMapper (auto-configured)
JSON string: { "id": 1, "name": "Alice", "email": "alice@example.com" }
    ↓ HTTP Response body
Client receives JSON with Content-Type: application/json
```

### Jackson Customization Annotations:

| Annotation | Purpose |
|---|---|
| `@JsonProperty("full_name")` | Custom JSON field name |
| `@JsonIgnore` | Exclude field from JSON |
| `@JsonFormat(pattern="dd/MM/yyyy")` | Date format in JSON |
| `@JsonInclude(NON_NULL)` | Exclude null fields from JSON |

---

## 6. What is GetMapping?

`@GetMapping` is a **composed annotation** that is a shortcut for `@RequestMapping(method = RequestMethod.GET)`. It maps HTTP **GET requests** to a specific controller method. It is part of a family of HTTP method-specific mapping annotations introduced in Spring 4.3.

```java
// These two are equivalent:
@RequestMapping(value = "/users", method = RequestMethod.GET)
@GetMapping("/users")   // Shorthand — preferred in modern Spring

// Full family of HTTP method annotations:
@GetMapping("/users")           // GET  — Read
@PostMapping("/users")          // POST — Create
@PutMapping("/users/{id}")      // PUT  — Replace
@PatchMapping("/users/{id}")    // PATCH — Partial Update
@DeleteMapping("/users/{id}")   // DELETE — Delete
```

---

## 7. Example POST Method?

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping                                   // Maps HTTP POST /api/users
    public ResponseEntity<User> createUser(
            @RequestBody @Valid User user) {        // @RequestBody: reads JSON from request body

        User savedUser = userService.save(user);

        // Build Location URI: /api/users/{newId}
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(savedUser.getId())
            .toUri();

        // 201 Created + Location header + saved user in body
        return ResponseEntity.created(location).body(savedUser);
    }
}
```

**Request:**
```json
// POST /api/users
// Content-Type: application/json
{
    "name": "Bob",
    "email": "bob@example.com"
}
```

**Response:**
```
HTTP/1.1 201 Created
Location: http://localhost:8080/api/users/5
Content-Type: application/json

{
    "id": 5,
    "name": "Bob",
    "email": "bob@example.com"
}
```

---

## 8. HTTP Status for Resource Creation?

The correct HTTP status code for a **successful resource creation** (POST) is **`201 Created`**.

| Status Code | When to Use |
|---|---|
| `200 OK` | Successful GET, PUT, PATCH, DELETE |
| `201 Created` | Successful POST — new resource created |
| `204 No Content` | Successful DELETE or PUT with no response body |
| `400 Bad Request` | Invalid request data / validation failure |
| `401 Unauthorized` | Authentication required |
| `403 Forbidden` | Authenticated but not authorized |
| `404 Not Found` | Resource does not exist |
| `409 Conflict` | Resource already exists / state conflict |
| `422 Unprocessable Entity` | Validation errors |
| `500 Internal Server Error` | Unexpected server-side error |

> **Best Practice:** Along with `201 Created`, also return a **`Location` header** containing the URI of the newly created resource (e.g., `Location: /api/users/5`), so the client knows where to find it.

---

## 9. Why ResponseEntity?

**`ResponseEntity<T>`** is a Spring class that represents the **entire HTTP response** — including the **status code**, **headers**, and **body**. It gives you full control over the HTTP response compared to returning a plain object.

| Returning Plain Object | Returning `ResponseEntity` |
|---|---|
| Status always `200 OK` | Full control over status code |
| No custom headers | Can add custom headers |
| Less expressive | Explicit and intention-revealing |
| Cannot return `404`, `201`, etc. | Can return any HTTP status |

```java
// Plain object — always 200, no control
@GetMapping("/{id}")
public User getUser(@PathVariable int id) {
    return userService.findById(id); // Can't return 404 if not found
}

// ResponseEntity — full control
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable int id) {
    User user = userService.findById(id);
    if (user == null) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).build();     // 404
    }
    return ResponseEntity.ok(user);                                      // 200 + body
}

// Custom headers
@GetMapping("/{id}")
public ResponseEntity<User> getUserWithHeaders(@PathVariable int id) {
    User user = userService.findById(id);
    return ResponseEntity.ok()
        .header("X-Custom-Header", "value")
        .header("Cache-Control", "max-age=3600")
        .body(user);
}
```

---

## 10. What is HATEOAS?

**HATEOAS (Hypermedia as the Engine of Application State)** is a REST constraint where **API responses include hyperlinks to related actions and resources**. The client navigates the API dynamically by following these links — it doesn't need to hardcode any URLs. This makes the API self-discoverable and loosely coupled.

**Without HATEOAS:**
```json
{
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
}
// Client must KNOW the URL structure to take any action
```

**With HATEOAS:**
```json
{
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "_links": {
        "self":    { "href": "http://localhost:8080/api/users/1" },
        "update":  { "href": "http://localhost:8080/api/users/1", "method": "PUT" },
        "delete":  { "href": "http://localhost:8080/api/users/1", "method": "DELETE" },
        "orders":  { "href": "http://localhost:8080/api/users/1/orders" }
    }
}
// Client follows links — no need to hardcode URLs
```

**Real World Analogy:** HATEOAS is like a **website with clickable links**. You land on the homepage and navigate by clicking. You don't need to know all URLs in advance — the page tells you where you can go next.

---

## 11. Example HATEOAS Response?

```json
// GET /api/users/1
{
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "role": "ADMIN",
    "_links": {
        "self": {
            "href": "http://localhost:8080/api/users/1"
        },
        "all-users": {
            "href": "http://localhost:8080/api/users"
        },
        "user-orders": {
            "href": "http://localhost:8080/api/users/1/orders"
        }
    }
}

// GET /api/users (collection with embedded links)
{
    "_embedded": {
        "users": [
            {
                "id": 1,
                "name": "Alice",
                "_links": {
                    "self": { "href": "http://localhost:8080/api/users/1" }
                }
            },
            {
                "id": 2,
                "name": "Bob",
                "_links": {
                    "self": { "href": "http://localhost:8080/api/users/2" }
                }
            }
        ]
    },
    "_links": {
        "self": { "href": "http://localhost:8080/api/users" }
    }
}
```

---

## 12. How to Implement HATEOAS?

```xml
<!-- Add Spring HATEOAS dependency -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

```java
// Step 1: Extend EntityModel to wrap your resource with links
import org.springframework.hateoas.EntityModel;
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public EntityModel<User> getUserById(@PathVariable int id) {
        User user = userService.findById(id);

        // Wrap entity in EntityModel and add links
        return EntityModel.of(user,
            linkTo(methodOn(UserController.class).getUserById(id))
                .withSelfRel(),                                          // self link
            linkTo(methodOn(UserController.class).getAllUsers())
                .withRel("all-users"),                                   // all-users link
            linkTo(methodOn(OrderController.class).getOrdersByUser(id))
                .withRel("user-orders")                                  // orders link
        );
    }

    @GetMapping
    public CollectionModel<EntityModel<User>> getAllUsers() {
        List<EntityModel<User>> users = userService.findAll().stream()
            .map(user -> EntityModel.of(user,
                linkTo(methodOn(UserController.class)
                    .getUserById(user.getId())).withSelfRel()))
            .collect(Collectors.toList());

        return CollectionModel.of(users,
            linkTo(methodOn(UserController.class).getAllUsers()).withSelfRel());
    }
}
```

---

## 13. REST Documentation?

**REST API documentation** describes all API endpoints, request/response formats, parameters, status codes, and authentication requirements so developers can consume the API correctly.

| Documentation Tool | Description |
|---|---|
| **Swagger / OpenAPI** | Most popular — auto-generates interactive docs from code annotations |
| **Spring REST Docs** | Test-driven docs — generates documentation from integration tests |
| **Postman** | API testing tool that can also export/generate documentation |
| **Redoc** | Beautiful OpenAPI documentation renderer |
| **RAML** | YAML-based API description language |

> **OpenAPI Specification (OAS)** is the industry standard format (formerly called Swagger). Swagger is the tooling ecosystem built around OpenAPI.

---

## 14. Swagger Basics?

**Swagger** is an **open-source toolset** built around the **OpenAPI Specification** for designing, building, documenting, and consuming REST APIs.

| Component | Purpose |
|---|---|
| **OpenAPI Spec** | Standard JSON/YAML format describing the API |
| **Swagger UI** | Interactive browser-based UI to explore and test APIs |
| **Swagger Editor** | Web editor to write OpenAPI specs |
| **Springdoc OpenAPI** | Library to auto-generate OpenAPI docs for Spring Boot apps |

```xml
<!-- Add Springdoc OpenAPI (modern Swagger for Spring Boot 3.x) -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

After adding the dependency:
- **Swagger UI:** `http://localhost:8080/swagger-ui.html`
- **OpenAPI JSON:** `http://localhost:8080/v3/api-docs`
- **OpenAPI YAML:** `http://localhost:8080/v3/api-docs.yaml`

---

## 15. Auto-Generate Swagger Docs?

Spring Boot + Springdoc OpenAPI **automatically generates documentation** from your controller code — zero additional configuration needed:

```java
// Just a normal @RestController — Swagger auto-documents it!
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable int id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody @Valid User user) {
        return ResponseEntity.status(201).body(userService.save(user));
    }
}
```

Springdoc scans all controllers and generates:
- Endpoint list with HTTP methods and paths
- Request body schema (from `@RequestBody`)
- Response schema (from return type)
- Path parameters (from `@PathVariable`)
- Query parameters (from `@RequestParam`)

---

## 16. Customize Swagger Docs?

```java
// API-level info
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("User Management API")
                .description("REST API for managing users")
                .version("v1.0")
                .contact(new Contact()
                    .name("Dev Team")
                    .email("dev@example.com"))
                .license(new License()
                    .name("Apache 2.0")));
    }
}

// Controller/method-level annotations
@RestController
@RequestMapping("/api/users")
@Tag(name = "User API", description = "Operations related to users")  // Group tag
public class UserController {

    @GetMapping("/{id}")
    @Operation(
        summary = "Get user by ID",
        description = "Returns a single user by their unique ID"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    public ResponseEntity<User> getUser(
            @Parameter(description = "User ID", required = true)
            @PathVariable int id) {
        return ResponseEntity.ok(userService.findById(id));
    }
}

// Model-level annotations
public class User {
    @Schema(description = "Unique user ID", example = "1")
    private int id;

    @Schema(description = "Full name of user", example = "Alice Smith")
    private String name;

    @Schema(description = "Email address", example = "alice@example.com")
    private String email;
}
```

```properties
# Customize Swagger UI path
springdoc.swagger-ui.path=/api-docs-ui
springdoc.api-docs.path=/api-docs

# Disable Swagger in production
springdoc.swagger-ui.enabled=false   # Set via profile
```

---

## 17. What is Swagger UI?

**Swagger UI** is an **interactive, browser-based web interface** that automatically renders an OpenAPI specification as a visual, human-friendly documentation page. It allows developers to:
- Browse all available API endpoints
- View request/response schemas
- **Execute API calls directly from the browser** (built-in API client)
- View authentication requirements

| URL | What you see |
|---|---|
| `http://localhost:8080/swagger-ui.html` | Full interactive Swagger UI |
| `http://localhost:8080/v3/api-docs` | Raw OpenAPI JSON spec |

---

## 18. What is Representation of Resource?

**Representation** is the **format/structure in which a resource's state is returned** in an HTTP response. The same resource can be represented in multiple formats depending on what the client requests.

```
Resource: User (id=1, name="Alice")

JSON Representation:          XML Representation:
{                             <User>
  "id": 1,                      <id>1</id>
  "name": "Alice"               <name>Alice</name>
}                             </User>
```

| Concept | Description |
|---|---|
| **Resource** | The actual entity (User, Product) — exists on the server |
| **Representation** | The formatted version of the resource sent to/from client |
| **Media Type** | Describes the format — `application/json`, `application/xml` |
| **Content-Type** | Header telling the server the format of the request body |
| **Accept** | Header telling the server what format the client wants back |

---

## 19. What is Content Negotiation?

**Content Negotiation** is the mechanism by which the **client and server agree on the format (representation) of the data** exchanged in a request/response. The client specifies what format it can accept, and the server responds in that format if supported.

| Direction | Header | Example |
|---|---|---|
| Client → Server (request body format) | `Content-Type` | `Content-Type: application/json` |
| Client → Server (desired response format) | `Accept` | `Accept: application/xml` |
| Server → Client (response body format) | `Content-Type` | `Content-Type: application/json` |

**Flow:**
```
Client:  GET /api/users/1
         Accept: application/xml          ← "I want XML"

Server:  HTTP/1.1 200 OK
         Content-Type: application/xml   ← "Here's your XML"
         <User><id>1</id><name>Alice</name></User>
```

---

## 20. Header for Content Negotiation?

| Header | Sender | Purpose | Example Value |
|---|---|---|---|
| `Accept` | Client | What format client wants in response | `application/json`, `application/xml`, `*/*` |
| `Content-Type` | Client / Server | Format of the body being sent | `application/json`, `application/xml` |
| `Accept-Language` | Client | Preferred response language | `en-US`, `fr-FR` |
| `Accept-Encoding` | Client | Preferred compression | `gzip`, `deflate` |

```http
# Request asking for XML response:
GET /api/users/1 HTTP/1.1
Host: localhost:8080
Accept: application/xml

# Request sending JSON body:
POST /api/users HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Accept: application/json

{"name": "Alice", "email": "alice@example.com"}
```

---

## 21. Implement Content Negotiation in Spring Boot?

Spring Boot handles JSON automatically. To support **multiple formats**:

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping(value = "/{id}",
        produces = {
            MediaType.APPLICATION_JSON_VALUE,   // application/json
            MediaType.APPLICATION_XML_VALUE     // application/xml
        })
    public ResponseEntity<User> getUser(@PathVariable int id) {
        // Spring auto-serializes to JSON or XML based on Accept header
        return ResponseEntity.ok(userService.findById(id));
    }
}
```

```properties
# application.properties — enable content negotiation via URL extension (optional)
spring.mvc.contentnegotiation.favor-parameter=true
# Now: /api/users/1?format=xml  OR  /api/users/1?format=json
```

---

## 22. Add XML Support?

By default Spring Boot only supports JSON. To add **XML support**, add the Jackson XML dependency:

```xml
<!-- Maven -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

```java
// Annotate your model for XML serialization
import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlRootElement;

@JacksonXmlRootElement(localName = "User")   // Root XML element name
public class User {
    private int id;
    private String name;
    private String email;
    // getters and setters
}
```

**Result for `Accept: application/xml`:**
```xml
<User>
    <id>1</id>
    <name>Alice</name>
    <email>alice@example.com</email>
</User>
```

---

## 23. Exception Handling in REST?

In REST APIs, exceptions must be **converted into meaningful HTTP error responses** with the correct status code and a structured JSON error body — never expose raw stack traces to clients.

```java
// Global exception handler for REST APIs
@RestControllerAdvice                           // @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)       // 404
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(404, ex.getMessage(), LocalDateTime.now());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)     // 400
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return new ErrorResponse(400, message, LocalDateTime.now());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)  // 500
    public ErrorResponse handleAll(Exception ex) {
        return new ErrorResponse(500, "Internal server error", LocalDateTime.now());
    }
}
```

**Standard Error Response Structure:**
```java
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;
    // Optional: String path, List<String> details
}
```

```json
// Error response body:
{
    "status": 404,
    "message": "User not found with id: 42",
    "timestamp": "2024-03-15T10:30:00"
}
```

---

## 24. Best Practices for REST Exception Handling?

| Best Practice | Description |
|---|---|
| **Never expose stack traces** | Log internally; return user-friendly messages only |
| **Use correct HTTP status codes** | `404` for not found, `400` for bad input, `500` for server errors |
| **Consistent error response structure** | Always return same JSON structure: `status`, `message`, `timestamp` |
| **Use `@RestControllerAdvice`** | Centralize all exception handling — don't scatter across controllers |
| **Custom exception classes** | `UserNotFoundException extends RuntimeException` — specific and readable |
| **Handle validation separately** | `MethodArgumentNotValidException` → field-level error messages |
| **Log all exceptions** | Use SLF4J Logger — include request path, user, and exception details |
| **Distinguish client vs server errors** | 4xx for client mistakes, 5xx for server issues |
| **Return `ProblemDetail` (RFC 7807)** | Spring 6 / Boot 3 standard error format — use `ProblemDetail` class |

### RFC 7807 ProblemDetail (Spring 6+):
```java
@ExceptionHandler(ResourceNotFoundException.class)
public ProblemDetail handleNotFound(ResourceNotFoundException ex) {
    ProblemDetail problem = ProblemDetail
        .forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    problem.setTitle("Resource Not Found");
    problem.setProperty("timestamp", LocalDateTime.now());
    return problem;
}
```

---

## 25. HTTP Error Statuses?

### 4xx — Client Errors

| Status | Name | When to Use |
|---|---|---|
| `400` | Bad Request | Malformed request, invalid syntax |
| `401` | Unauthorized | Authentication required / token missing |
| `403` | Forbidden | Authenticated but lacks permission |
| `404` | Not Found | Resource does not exist |
| `405` | Method Not Allowed | HTTP method not supported for this endpoint |
| `409` | Conflict | Resource already exists / state conflict |
| `415` | Unsupported Media Type | Wrong `Content-Type` header |
| `422` | Unprocessable Entity | Validation errors on well-formed request |
| `429` | Too Many Requests | Rate limit exceeded |

### 5xx — Server Errors

| Status | Name | When to Use |
|---|---|---|
| `500` | Internal Server Error | Unexpected server-side failure |
| `502` | Bad Gateway | Upstream server returned invalid response |
| `503` | Service Unavailable | Server temporarily down / overloaded |
| `504` | Gateway Timeout | Upstream server timed out |

---

## 26. Implement Error Handling in Spring Boot?

### Complete Implementation:

```java
// Step 1: Custom exception
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(int id) {
        super("User not found with id: " + id);
    }
}

// Step 2: Standard error response model
@Getter
@AllArgsConstructor
public class ApiError {
    private int status;
    private String message;
    private String path;
    private LocalDateTime timestamp;
}

// Step 3: Global handler
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ApiError> handleNotFound(
            UserNotFoundException ex, HttpServletRequest request) {
        log.error("User not found: {}", ex.getMessage());
        ApiError error = new ApiError(
            404, ex.getMessage(), request.getRequestURI(), LocalDateTime.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiError> handleValidation(
            MethodArgumentNotValidException ex, HttpServletRequest request) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining("; "));
        ApiError error = new ApiError(
            400, message, request.getRequestURI(), LocalDateTime.now());
        return ResponseEntity.badRequest().body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiError> handleAll(
            Exception ex, HttpServletRequest request) {
        log.error("Unexpected error: ", ex);
        ApiError error = new ApiError(
            500, "An unexpected error occurred",
            request.getRequestURI(), LocalDateTime.now());
        return ResponseEntity.internalServerError().body(error);
    }
}

// Step 4: Throw in controller
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable int id) {
    User user = userService.findById(id);
    if (user == null) throw new UserNotFoundException(id);
    return ResponseEntity.ok(user);
}
```

---

## 27. HTTP Status for Validation Errors?

| Scenario | Recommended Status |
|---|---|
| Missing required field / malformed JSON | `400 Bad Request` |
| Field-level validation failure (`@Valid`) | `400 Bad Request` or `422 Unprocessable Entity` |
| Business rule violation (duplicate email) | `409 Conflict` |

> **Spring Boot default:** `MethodArgumentNotValidException` returns `400 Bad Request` by default.
> **RFC 7807 recommendation:** Use `422 Unprocessable Entity` for semantic validation failures.

---

## 28. Handle Validation Errors?

```java
// Model with validation annotations
public class UserRequest {
    @NotNull(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be 2-50 characters")
    private String name;

    @NotNull(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @Min(value = 18, message = "Age must be at least 18")
    private int age;
    // getters and setters
}

// Controller — trigger validation with @Valid
@PostMapping
public ResponseEntity<User> createUser(@RequestBody @Valid UserRequest request) {
    // If validation fails → MethodArgumentNotValidException is thrown automatically
    return ResponseEntity.status(201).body(userService.save(request));
}

// Handle validation exception globally
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<Map<String, String>> handleValidationErrors(
        MethodArgumentNotValidException ex) {

    Map<String, String> errors = new LinkedHashMap<>();
    ex.getBindingResult().getFieldErrors()
        .forEach(error -> errors.put(error.getField(), error.getDefaultMessage()));

    return ResponseEntity.badRequest().body(errors);
}
```

**Error Response:**
```json
// POST /api/users (with invalid data)
// HTTP 400 Bad Request
{
    "name": "Name must be 2-50 characters",
    "email": "Invalid email format",
    "age": "Age must be at least 18"
}
```

---

## 29. Why API Versioning?

**API versioning** is the practice of **managing changes to an API** so that existing clients are not broken when the API evolves. Without versioning, any breaking change (removing a field, changing a response structure) would break all existing clients.

| Reason | Description |
|---|---|
| **Backward compatibility** | Old clients keep working while new clients use the new version |
| **Breaking changes** | Removing/renaming fields, changing data types, restructuring responses |
| **Parallel development** | Different teams/clients can use different versions simultaneously |
| **Controlled deprecation** | Old versions can be deprecated and removed gradually |
| **Consumer-driven evolution** | APIs can evolve without forcing all consumers to update at once |

---

## 30. Versioning Options?

| Strategy | How | Example URL / Header | Pros | Cons |
|---|---|---|---|---|
| **URI Versioning** | Version in URL path | `/api/v1/users` | Simple, visible, cacheable | URL pollution |
| **Request Parameter** | Version as query param | `/api/users?version=1` | Easy to default | Less clean URLs |
| **Header Versioning** | Custom request header | `X-API-Version: 1` | Clean URLs | Not browser-friendly |
| **Media Type (Accept)** | Version in Accept header | `Accept: application/vnd.app.v1+json` | Most RESTful | Complex for clients |

> **Most widely used in practice:** URI versioning (`/api/v1/`) — simple, visible, and cacheable.

---

## 31. Implement Versioning?

### URI Versioning (Most Common)
```java
// Version 1
@RestController
@RequestMapping("/api/v1/users")
public class UserV1Controller {

    @GetMapping("/{id}")
    public UserV1 getUser(@PathVariable int id) {
        return new UserV1(id, "Alice");   // V1 response: {id, name}
    }
}

// Version 2 — adds email field
@RestController
@RequestMapping("/api/v2/users")
public class UserV2Controller {

    @GetMapping("/{id}")
    public UserV2 getUser(@PathVariable int id) {
        return new UserV2(id, "Alice", "alice@example.com"); // V2: {id, name, email}
    }
}
```

### Request Parameter Versioning
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping(value = "/{id}", params = "version=1")
    public UserV1 getUserV1(@PathVariable int id) { ... }

    @GetMapping(value = "/{id}", params = "version=2")
    public UserV2 getUserV2(@PathVariable int id) { ... }
}
// GET /api/users/1?version=1  OR  /api/users/1?version=2
```

### Header Versioning
```java
@GetMapping(value = "/{id}", headers = "X-API-Version=1")
public UserV1 getUserV1(@PathVariable int id) { ... }

@GetMapping(value = "/{id}", headers = "X-API-Version=2")
public UserV2 getUserV2(@PathVariable int id) { ... }
```

### Media Type Versioning
```java
@GetMapping(value = "/{id}",
    produces = "application/vnd.company.app-v1+json")
public UserV1 getUserV1(@PathVariable int id) { ... }

@GetMapping(value = "/{id}",
    produces = "application/vnd.company.app-v2+json")
public UserV2 getUserV2(@PathVariable int id) { ... }
// Client sends: Accept: application/vnd.company.app-v2+json
```

---

*Happy Learning REST APIs! 🌐*
