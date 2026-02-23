# 02 — HTTP, REST & Backend Basics

> **Goal:** Understand the foundational protocols and patterns that EVERY Java backend communicates through — HTTP, REST, JSON, and API design best practices.

---

## Table of Contents

1. [Client–Server Architecture](#1-clientserver-architecture)
2. [HTTP Protocol Deep Dive](#2-http-protocol-deep-dive)
3. [REST API Principles](#3-rest-api-principles)
4. [JSON Serialization](#4-json-serialization)
5. [API Design Best Practices](#5-api-design-best-practices)
6. [API Versioning](#6-api-versioning)
7. [Idempotency](#7-idempotency)
8. [Pagination](#8-pagination)
9. [Error Handling](#9-error-handling)
10. [Validation](#10-validation)
11. [Interview Notes](#11-interview-notes)
12. [Summary](#12-summary)
13. [References](#13-references)

---

## 1. Client–Server Architecture

**Definition:**
Client–server architecture is a computing model where a **client** (browser, mobile app, another service) sends requests to a **server** (your Java backend), which processes them and sends back responses. The client initiates; the server responds.

**Why it exists:**
Separating concerns between presentation (client) and data processing/storage (server) allows each side to evolve independently. Backend logic is centralized, secured, and scalable.

**Real-life analogy:**
A restaurant. The customer (client) places an order (request) with the waiter, who communicates it to the kitchen (server). The kitchen prepares the food (processes the request) and the waiter delivers it (response). The customer doesn't need to know how to cook.

**Backend example:**
```
Client (React App)                     Server (Spring Boot)
     │                                       │
     │── GET /api/users/42 ──────────────────→│
     │                                       │──→ UserController
     │                                       │──→ UserService
     │                                       │──→ UserRepository (DB)
     │←── 200 OK { "id": 42, "name": ... } ──│
     │                                       │
```

**Key characteristics:**
- **Stateless communication** — each request contains all info needed (no server-side session required)
- **Centralized logic** — business rules, validation, and security live on the server
- **Scalable** — add more server instances behind a load balancer
- **Protocol** — HTTP/HTTPS is the standard transport

---

## 2. HTTP Protocol Deep Dive

### 2.1 What is HTTP?

**Definition:**
HTTP (HyperText Transfer Protocol) is an application-layer protocol for transmitting data over the web. It follows a request–response model where the client sends an HTTP request and the server returns an HTTP response.

**Why it exists:**
Every web API, browser, and mobile app communicates via HTTP. Your Spring Boot API endpoints are HTTP endpoints. Understanding HTTP deeply means understanding how your API actually works on the wire.

**Real-life analogy:**
HTTP is like the postal system. A letter (request) has a destination address (URL), instructions (method — "read this" or "deliver this"), and content (body). The response is the reply letter with a status (delivered, undeliverable, wrong address).

### 2.2 HTTP Methods

| Method | Purpose | Safe? | Idempotent? | Request Body? |
|--------|---------|-------|-------------|---------------|
| `GET` | Retrieve a resource | ✅ | ✅ | ❌ |
| `POST` | Create a new resource | ❌ | ❌ | ✅ |
| `PUT` | Replace a resource entirely | ❌ | ✅ | ✅ |
| `PATCH` | Partially update a resource | ❌ | ❌* | ✅ |
| `DELETE` | Remove a resource | ❌ | ✅ | ❌ |
| `HEAD` | Same as GET but no body | ✅ | ✅ | ❌ |
| `OPTIONS` | Describe communication options | ✅ | ✅ | ❌ |

*PATCH can be designed to be idempotent, but is not guaranteed by the spec.

**Backend example:**
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @GetMapping("/{id}")        // GET — retrieve
    public UserDto getUser(@PathVariable Long id) { ... }

    @PostMapping                // POST — create
    public UserDto createUser(@RequestBody @Valid CreateUserRequest req) { ... }

    @PutMapping("/{id}")        // PUT — full replace
    public UserDto updateUser(@PathVariable Long id, @RequestBody @Valid UpdateUserRequest req) { ... }

    @PatchMapping("/{id}")      // PATCH — partial update
    public UserDto patchUser(@PathVariable Long id, @RequestBody Map<String, Object> updates) { ... }

    @DeleteMapping("/{id}")     // DELETE — remove
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) { ... }
}
```

### 2.3 HTTP Status Codes

| Range | Category | Common Codes |
|-------|----------|-------------|
| 1xx | Informational | `100 Continue` |
| 2xx | Success | `200 OK`, `201 Created`, `204 No Content` |
| 3xx | Redirection | `301 Moved Permanently`, `304 Not Modified` |
| 4xx | Client Error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests` |
| 5xx | Server Error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout` |

**Backend usage guide:**
```java
@PostMapping
public ResponseEntity<UserDto> createUser(@RequestBody @Valid CreateUserRequest request) {
    UserDto created = userService.create(request);
    URI location = URI.create("/api/v1/users/" + created.getId());
    return ResponseEntity.created(location).body(created); // 201 Created
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.deleteById(id);
    return ResponseEntity.noContent().build(); // 204 No Content
}
```

### 2.4 HTTP Headers

**Key headers for backend development:**

| Header | Purpose | Example |
|--------|---------|---------|
| `Content-Type` | Type of request/response body | `application/json` |
| `Accept` | What the client expects | `application/json` |
| `Authorization` | Auth credentials | `Bearer eyJhbGciOi...` |
| `Cache-Control` | Caching directives | `max-age=3600, public` |
| `X-Request-Id` | Request tracing | `550e8400-e29b-41d4-a716-446655440000` |
| `X-Forwarded-For` | Client IP behind proxy | `203.0.113.50` |
| `ETag` | Resource version for caching | `"33a64df551425fcc55e"` |
| `Retry-After` | When to retry after 429/503 | `60` (seconds) |

---

## 3. REST API Principles

### 3.1 What is REST?

**Definition:**
REST (Representational State Transfer) is an architectural style for designing networked APIs. A RESTful API uses HTTP methods, URIs to identify resources, and standard status codes to communicate results. It is resource-oriented, stateless, and cacheable.

**Why it exists:**
REST provides a uniform, predictable way for clients to interact with backend services. Instead of inventing custom protocols, REST leverages HTTP's built-in semantics.

**Real-life analogy:**
REST is like a well-organized library. Each book (resource) has a unique shelf location (URI). You can read (GET), add (POST), replace (PUT), or remove (DELETE) books using standard operations. The librarian (server) doesn't remember your previous visits (stateless).

**Key constraints:**
1. **Client–Server** — separation of concerns
2. **Stateless** — each request contains all info needed
3. **Cacheable** — responses can declare cacheability
4. **Uniform Interface** — standard methods, URIs, representations
5. **Layered System** — client doesn't know if it talks directly to server or a proxy
6. **Code on Demand** (optional) — server can send executable code

### 3.2 Resource Naming Conventions

```
# ✅ GOOD — nouns, plural, consistent
GET    /api/v1/users           → List all users
GET    /api/v1/users/42        → Get user 42
POST   /api/v1/users           → Create user
PUT    /api/v1/users/42        → Update user 42
DELETE /api/v1/users/42        → Delete user 42
GET    /api/v1/users/42/orders → Get orders of user 42

# ❌ BAD — verbs, inconsistent
GET    /api/v1/getUsers
POST   /api/v1/createUser
GET    /api/v1/user/42/getOrders
DELETE /api/v1/deleteUser?id=42
```

**Key rules:**
- Use **nouns** (not verbs) for resources
- Use **plural** names (`/users`, not `/user`)
- Use **path params** for identity (`/users/42`)
- Use **query params** for filtering (`/users?status=active`)
- Use **kebab-case** for multi-word paths (`/order-items`)
- Nest sub-resources logically (`/users/42/orders`)

---

## 4. JSON Serialization

**Definition:**
JSON (JavaScript Object Notation) serialization is the process of converting Java objects into JSON format for API responses, and **deserialization** is the reverse — converting incoming JSON into Java objects. Spring Boot uses **Jackson** as the default JSON library.

**Why it exists:**
JSON is the universal data exchange format for APIs. Your backend must convert database entities to JSON responses and parse JSON request bodies into Java objects.

**Backend example:**
```java
// Java object
@Getter @Setter
public class UserDto {
    private Long id;
    private String name;
    private String email;

    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    private LocalDateTime createdAt;

    @JsonIgnore // Won't appear in JSON
    private String internalCode;

    @JsonProperty("user_type") // Custom JSON field name
    private String userType;
}

// Serialized JSON output
{
    "id": 42,
    "name": "Prince Kumar",
    "email": "prince@example.com",
    "createdAt": "2026-02-23T16:48:46",
    "user_type": "PREMIUM"
}
```

**Key Jackson annotations:**

| Annotation | Purpose |
|-----------|---------|
| `@JsonProperty` | Custom JSON field name |
| `@JsonIgnore` | Exclude field from JSON |
| `@JsonFormat` | Date/time formatting |
| `@JsonInclude(NON_NULL)` | Skip null fields |
| `@JsonCreator` | Custom deserialization constructor |
| `@JsonSerialize` / `@JsonDeserialize` | Custom serializer/deserializer |

---

## 5. API Design Best Practices

### 5.1 Consistent Response Structure

**Definition:**
A consistent response structure means every API endpoint returns data in the same envelope format, making it predictable for clients to parse and handle.

**Backend example:**
```java
// Standard response wrapper
@Getter @Builder
public class ApiResponse<T> {
    private final boolean success;
    private final int statusCode;
    private final String message;
    private final T data;
    private final LocalDateTime timestamp;
    private final String path;
}

// Success response
{
    "success": true,
    "statusCode": 200,
    "message": "User retrieved successfully",
    "data": {
        "id": 42,
        "name": "Prince Kumar"
    },
    "timestamp": "2026-02-23T16:48:46",
    "path": "/api/v1/users/42"
}

// Error response
{
    "success": false,
    "statusCode": 404,
    "message": "User not found with id: 99",
    "data": null,
    "timestamp": "2026-02-23T16:48:46",
    "path": "/api/v1/users/99"
}
```

### 5.2 HATEOAS (Hypermedia)

**Definition:**
HATEOAS (Hypermedia As The Engine Of Application State) is a REST constraint where responses include links to related actions, enabling clients to discover available operations dynamically.

**Backend example:**
```java
// Spring HATEOAS
@GetMapping("/{id}")
public EntityModel<UserDto> getUser(@PathVariable Long id) {
    UserDto user = userService.findById(id);
    return EntityModel.of(user,
        linkTo(methodOn(UserController.class).getUser(id)).withSelfRel(),
        linkTo(methodOn(UserController.class).getAllUsers()).withRel("users"),
        linkTo(methodOn(OrderController.class).getOrdersByUser(id)).withRel("orders")
    );
}

// Response with links
{
    "id": 42,
    "name": "Prince Kumar",
    "_links": {
        "self": { "href": "/api/v1/users/42" },
        "users": { "href": "/api/v1/users" },
        "orders": { "href": "/api/v1/users/42/orders" }
    }
}
```

---

## 6. API Versioning

**Definition:**
API versioning is the practice of maintaining multiple versions of an API so that existing clients aren't broken when the API evolves. It allows you to introduce breaking changes in new versions while keeping old versions operational.

**Why it exists:**
APIs evolve — fields get renamed, endpoints change, response formats update. Without versioning, every change could break every client. Versioning provides backward compatibility.

**Real-life analogy:**
Software update channels — users on v1 continue to work while v2 adds new features. They can migrate at their own pace.

**Backend example — strategies:**

```java
// 1. URI Versioning (most common, recommended)
@RequestMapping("/api/v1/users")
public class UserControllerV1 { ... }

@RequestMapping("/api/v2/users")
public class UserControllerV2 { ... }

// 2. Header Versioning
@GetMapping(value = "/api/users", headers = "X-API-Version=1")
public List<UserDtoV1> getUsersV1() { ... }

@GetMapping(value = "/api/users", headers = "X-API-Version=2")
public List<UserDtoV2> getUsersV2() { ... }

// 3. Query Parameter Versioning
@GetMapping(value = "/api/users", params = "version=1")
public List<UserDtoV1> getUsersV1() { ... }

// 4. Content Negotiation (Accept header)
@GetMapping(value = "/api/users", produces = "application/vnd.myapi.v1+json")
public List<UserDtoV1> getUsersV1() { ... }
```

**When to use:**
- URI versioning for public APIs (simple, explicit)
- Header versioning for internal APIs (cleaner URIs)
- Always version from the start — retrofitting is painful

---

## 7. Idempotency

**Definition:**
An operation is idempotent if performing it multiple times has the same effect as performing it once. The result is identical regardless of how many times the request is repeated.

**Why it exists:**
Network failures cause retries. If a client sends a payment request but doesn't receive a response (network timeout), it will retry. Without idempotency, the user could be charged twice. Idempotent APIs are safe to retry.

**Real-life analogy:**
Pressing an elevator button multiple times doesn't call multiple elevators — the first press registers the request, and subsequent presses have no additional effect.

**Backend example:**
```java
// Idempotency key pattern for POST requests
@PostMapping("/payments")
public ResponseEntity<PaymentDto> processPayment(
        @RequestHeader("Idempotency-Key") String idempotencyKey,
        @RequestBody @Valid PaymentRequest request) {

    // Check if this request was already processed
    Optional<Payment> existing = paymentService.findByIdempotencyKey(idempotencyKey);
    if (existing.isPresent()) {
        return ResponseEntity.ok(toDto(existing.get())); // Return same result
    }

    // Process new payment
    Payment payment = paymentService.process(request, idempotencyKey);
    return ResponseEntity.status(HttpStatus.CREATED).body(toDto(payment));
}
```

**HTTP method idempotency:**
| Method | Idempotent? | Why |
|--------|-------------|-----|
| GET | ✅ | Reading doesn't change state |
| PUT | ✅ | Replaces entire resource — same input = same result |
| DELETE | ✅ | Deleting twice = resource is still deleted |
| POST | ❌ | Creates a new resource each time |
| PATCH | ❌* | Depends on implementation |

---

## 8. Pagination

**Definition:**
Pagination is the practice of dividing large result sets into smaller, manageable pages. Instead of returning 100,000 records at once, you return 20 at a time with metadata about total count and navigation.

**Why it exists:**
Returning large datasets in a single response causes memory pressure on the server, network bandwidth waste, and UI rendering lag. Pagination controls resource usage and improves UX.

**Real-life analogy:**
A book has chapters and pages. You read one page at a time, not the whole book in one glance. You can skip to page 50 or go to the next page.

**Backend example:**
```java
// Spring Data JPA Pagination
@GetMapping("/users")
public ResponseEntity<Page<UserDto>> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "createdAt,desc") String[] sort) {

    Pageable pageable = PageRequest.of(page, size, Sort.by(parseSortOrders(sort)));
    Page<UserDto> result = userService.findAll(pageable);
    return ResponseEntity.ok(result);
}

// Response
{
    "content": [ { "id": 1, "name": "..." }, ... ],
    "pageable": {
        "pageNumber": 0,
        "pageSize": 20,
        "sort": { "sorted": true, "direction": "DESC" }
    },
    "totalElements": 1523,
    "totalPages": 77,
    "first": true,
    "last": false
}
```

**Pagination strategies:**

| Strategy | Pros | Cons | Use When |
|----------|------|------|----------|
| Offset-based (page/size) | Simple, well-supported by JPA | Slow on large offsets, data shifts | Admin panels, small datasets |
| Cursor-based (keyset) | Consistent, fast on large datasets | Can't jump to arbitrary page | Infinite scrolling, feeds |
| Seek-based | Very efficient with indexes | More complex queries | High-performance APIs |

**Cursor-based example:**
```java
@GetMapping("/feed")
public ResponseEntity<CursorPage<PostDto>> getFeed(
        @RequestParam(required = false) String cursor,
        @RequestParam(defaultValue = "20") int limit) {

    // cursor is the last item's ID or timestamp
    List<Post> posts = postService.findAfterCursor(cursor, limit + 1);
    boolean hasNext = posts.size() > limit;
    if (hasNext) posts.remove(posts.size() - 1); // remove extra

    String nextCursor = hasNext ? posts.get(posts.size() - 1).getId().toString() : null;
    return ResponseEntity.ok(new CursorPage<>(toDtoList(posts), nextCursor, hasNext));
}
```

---

## 9. Error Handling

**Definition:**
API error handling is the strategy for communicating failures to clients in a consistent, machine-readable format. It involves mapping exceptions to appropriate HTTP status codes and structured error response bodies.

**Why it exists:**
Clients need to understand why a request failed and what to do about it. Generic "500 Internal Server Error" responses are useless for debugging. Well-structured errors improve developer experience and reduce support burden.

**Backend example:**
```java
// Standard error response format (RFC 7807 — Problem Details)
@Getter @Builder
public class ProblemDetail {
    private String type;        // URI reference identifying the error type
    private String title;       // Short summary
    private int status;         // HTTP status code
    private String detail;      // Human-readable explanation
    private String instance;    // URI of the specific occurrence
    private Map<String, Object> extensions; // Additional context
}

// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {
        ProblemDetail problem = ProblemDetail.builder()
            .type("https://api.example.com/errors/not-found")
            .title("Resource Not Found")
            .status(404)
            .detail(ex.getMessage())
            .instance(request.getRequestURI())
            .build();
        return ResponseEntity.status(404).body(problem);
    }

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ProblemDetail> handleValidation(ConstraintViolationException ex) {
        Map<String, Object> violations = new HashMap<>();
        ex.getConstraintViolations().forEach(v ->
            violations.put(v.getPropertyPath().toString(), v.getMessage())
        );
        ProblemDetail problem = ProblemDetail.builder()
            .type("https://api.example.com/errors/validation")
            .title("Validation Failed")
            .status(422)
            .detail("One or more fields failed validation")
            .extensions(violations)
            .build();
        return ResponseEntity.status(422).body(problem);
    }
}
```

**Error response (RFC 7807 format):**
```json
{
    "type": "https://api.example.com/errors/not-found",
    "title": "Resource Not Found",
    "status": 404,
    "detail": "User not found with id: 99",
    "instance": "/api/v1/users/99"
}
```

---

## 10. Validation

**Definition:**
Validation is the process of checking that incoming data conforms to expected rules before processing it. In Spring Boot, validation uses Bean Validation (JSR 380 / Jakarta Validation) annotations and can be applied at the controller, service, or entity level.

**Why it exists:**
Never trust client input. Invalid data can cause database errors, security vulnerabilities, or application crashes. Validation catches bad data early and returns clear error messages.

**Real-life analogy:**
A bouncer at a club entrance checks IDs, dress code, and capacity. Invalid entries are rejected with a clear reason before they ever get inside.

**Backend example:**
```java
// Request DTO with validation annotations
public record CreateUserRequest(
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    String name,

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    String email,

    @NotNull(message = "Age is required")
    @Min(value = 18, message = "Must be at least 18")
    @Max(value = 120, message = "Invalid age")
    Integer age,

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone number format")
    String phone
) {}

// Controller with validation
@PostMapping("/users")
public ResponseEntity<UserDto> createUser(
        @RequestBody @Valid CreateUserRequest request) { // @Valid triggers validation
    UserDto user = userService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(user);
}
```

**Key validation annotations:**

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@NotNull` | Field must not be null | `@NotNull Long id` |
| `@NotBlank` | String must not be null/empty/whitespace | `@NotBlank String name` |
| `@Size` | String/collection length constraints | `@Size(min=2, max=50)` |
| `@Min` / `@Max` | Numeric range | `@Min(0) @Max(100)` |
| `@Email` | Valid email format | `@Email String email` |
| `@Pattern` | Regex match | `@Pattern(regexp="...")` |
| `@Past` / `@Future` | Date in past/future | `@Past LocalDate dob` |
| `@Valid` | Cascade validation to nested objects | `@Valid Address address` |

**Custom validator:**
```java
// Custom annotation
@Target({FIELD})
@Retention(RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already registered";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator implementation
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        return email != null && !userRepository.existsByEmail(email);
    }
}
```

---

## 11. Interview Notes

### HTTP & REST — Top Interview Questions

1. **What is the difference between PUT and PATCH?**
   - PUT replaces the entire resource (idempotent). PATCH modifies specific fields (can be non-idempotent). Example: PUT sends all fields; PATCH sends only `{ "name": "new name" }`.

2. **Explain REST constraints.**
   - Stateless, client–server, cacheable, uniform interface, layered system, code on demand (optional).

3. **How do you handle API versioning?**
   - URI versioning (`/v1/users`), header versioning, query param, content negotiation. URI versioning is most common for public APIs.

4. **What is idempotency and why does it matter?**
   - Same request repeated = same result. Critical for payment APIs and retries. Use idempotency keys for POST requests.

5. **offset vs cursor pagination — when to use which?**
   - Offset: simple, supports page jumping, slow on large offsets. Cursor: consistent, fast, no page jumping. Use cursor for feeds/infinite scroll.

6. **How do you structure error responses?**
   - RFC 7807 Problem Details format. Consistent structure with type, title, status, detail, instance.

7. **How does @Valid work in Spring Boot?**
   - Triggers Jakarta Bean Validation on the annotated parameter. Violations throw `MethodArgumentNotValidException`, caught by `@RestControllerAdvice`.

---

## 12. Summary

| Topic | Key Takeaway |
|-------|-------------|
| Client–Server | Client requests, server responds, stateless communication |
| HTTP Methods | GET (read), POST (create), PUT (replace), PATCH (update), DELETE (remove) |
| Status Codes | 2xx success, 4xx client error, 5xx server error |
| REST | Resource-oriented, stateless, cacheable, uniform interface |
| JSON | Jackson is Spring Boot's default; use annotations to control serialization |
| Versioning | Start with URI versioning (`/api/v1/`); version from day one |
| Idempotency | Safe to retry; use idempotency keys for POST |
| Pagination | Offset for admin panels, cursor for feeds |
| Error Handling | Use RFC 7807 Problem Details; `@RestControllerAdvice` for global handling |
| Validation | `@Valid` + Bean Validation annotations; never trust client input |

---

## 13. References

- [RFC 7231 — HTTP/1.1 Semantics and Content](https://datatracker.ietf.org/doc/html/rfc7231)
- [RFC 7807 — Problem Details for HTTP APIs](https://datatracker.ietf.org/doc/html/rfc7807)
- [Roy Fielding's REST Dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [Spring MVC Documentation](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [Baeldung — REST API Best Practices](https://www.baeldung.com/rest-api-best-practices)
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)
- [Google API Design Guide](https://cloud.google.com/apis/design)
- [JSON:API Specification](https://jsonapi.org/)

---

> **Previous:** [← 01 — Java Backend Fundamentals](./01_Java_Backend_Fundamentals.md)  
> **Next:** [03 — Spring Core →](./03_Spring_Core.md)
