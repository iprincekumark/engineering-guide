# 13 — Backend Best Practices

> **Goal:** Establish coding standards and patterns that make Java backends maintainable, secure, testable, and production-ready.

---

## Table of Contents

1. [DTO vs Entity Separation](#1-dto-vs-entity-separation)
2. [Exception Strategy](#2-exception-strategy)
3. [Validation Strategy](#3-validation-strategy)
4. [Logging Standards](#4-logging-standards)
5. [Secure Coding](#5-secure-coding)
6. [API Versioning](#6-api-versioning)
7. [Testing Practices](#7-testing-practices)
8. [Code Review Practices](#8-code-review-practices)
9. [12-Factor App Principles](#9-12-factor-app-principles)
10. [Project Structure](#10-project-structure)
11. [Interview Notes](#11-interview-notes)
12. [Summary](#12-summary)
13. [References](#13-references)

---

## 1. DTO vs Entity Separation

**Definition:**
DTOs (Data Transfer Objects) are objects used to transfer data between layers (controller ↔ service ↔ client). Entities are JPA-managed objects mapped to database tables. They MUST be separate.

**Why it matters:**
Exposing entities directly causes: circular references in JSON, exposing sensitive fields (passwords), coupling API contract to database schema, and unintended lazy-loading.

```java
// ✅ Separate DTO
public record UserDto(Long id, String name, String email, LocalDateTime createdAt) {}
public record CreateUserRequest(
    @NotBlank String name,
    @Email String email,
    @NotBlank @Size(min=8) String password
) {}

// Entity — internal to the persistence layer
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private String email;
    private String passwordHash; // NEVER expose in DTO
    @OneToMany(mappedBy = "user")
    private List<Order> orders; // Could cause circular JSON
}

// Mapper (manual or MapStruct)
@Component
public class UserMapper {
    public UserDto toDto(User user) {
        return new UserDto(user.getId(), user.getName(), user.getEmail(), user.getCreatedAt());
    }
}
```

---

## 2. Exception Strategy

**Pattern: Custom exceptions → Global handler → Consistent error responses**

```java
// 1. Custom exception hierarchy
public abstract class AppException extends RuntimeException {
    private final HttpStatus status;
    protected AppException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }
}

public class ResourceNotFoundException extends AppException {
    public ResourceNotFoundException(String resource, String field, Object value) {
        super(String.format("%s not found with %s: %s", resource, field, value), HttpStatus.NOT_FOUND);
    }
}

public class BusinessException extends AppException {
    public BusinessException(String message) {
        super(message, HttpStatus.UNPROCESSABLE_ENTITY);
    }
}

// 2. Global handler
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(AppException.class)
    public ResponseEntity<ErrorResponse> handleApp(AppException ex, HttpServletRequest req) {
        return buildResponse(ex.getStatus(), ex.getMessage(), req);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex, HttpServletRequest req) {
        Map<String, String> errors = ex.getFieldErrors().stream()
            .collect(Collectors.toMap(FieldError::getField, FieldError::getDefaultMessage, (a, b) -> a));
        return buildResponse(HttpStatus.BAD_REQUEST, "Validation failed", req, errors);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex, HttpServletRequest req) {
        log.error("Unhandled exception", ex);
        return buildResponse(HttpStatus.INTERNAL_SERVER_ERROR, "An unexpected error occurred", req);
    }
}
```

**Rules:**
- Use unchecked exceptions (RuntimeException)
- Never expose stack traces in production
- Map exceptions to specific HTTP status codes
- Log unexpected exceptions at ERROR level
- Return consistent error response structure (RFC 7807)

---

## 3. Validation Strategy

**Layered validation:**

| Layer | What to Validate | How |
|-------|-----------------|-----|
| Controller | Input format, required fields | `@Valid` + Bean Validation |
| Service | Business rules, state transitions | Manual checks, throw `BusinessException` |
| Repository/DB | Data integrity | Constraints, unique indexes |

```java
// Controller — format validation
@PostMapping
public UserDto create(@RequestBody @Valid CreateUserRequest req) { ... }

// Service — business validation
public UserDto create(CreateUserRequest req) {
    if (userRepo.existsByEmail(req.email()))
        throw new BusinessException("Email already registered");
    // ...
}

// Database — integrity constraints
@Column(unique = true, nullable = false)
private String email;
```

---

## 4. Logging Standards

```java
// ✅ DO
log.info("Order created: orderId={}, userId={}, total={}", orderId, userId, total);
log.error("Payment failed for order {}: {}", orderId, ex.getMessage(), ex);
log.debug("Fetching user from cache: key={}", cacheKey);

// ❌ DON'T
log.info("Order created: " + order.toString()); // String concatenation
log.info("Password: " + user.getPassword());     // Sensitive data
log.error(ex.getMessage());                       // No context, no stack trace
System.out.println("Debug: " + value);            // Never use sysout
```

**Rules:**
- Use SLF4J with parameterized messages (`{}`)
- Never log sensitive data (passwords, tokens, SSN)
- Always log context (IDs, user, action)
- Use MDC for request correlation
- ERROR: unrecoverable, needs attention. WARN: unexpected but handled. INFO: business events. DEBUG: diagnostic.

---

## 5. Secure Coding

| Rule | Implementation |
|------|---------------|
| Never trust input | Validate ALL inputs (path params, headers, body) |
| Parameterized queries | JPA/JPQL (default). Never concatenate SQL |
| Hash passwords | BCrypt/Argon2 — never store plaintext |
| Protect secrets | Environment variables or vaults, never hardcode |
| Minimal error info | Don't expose stack traces or internal details |
| HTTPS only | TLS termination at load balancer |
| Principle of least privilege | Services run with minimum permissions |
| Dependency scanning | Snyk / OWASP Dependency-Check |
| Rate limiting | Protect against abuse |
| CORS whitelist | Explicit allowed origins, never `*` in production |

---

## 6. API Versioning

**Recommended: URI versioning**

```
/api/v1/users    — original
/api/v2/users    — breaking changes (new response format)
```

**Rules:**
- Version from day one
- Only increment for breaking changes
- Support at least N-1 version
- Document deprecation timeline
- Use different controller classes per version

---

## 7. Testing Practices

**Test pyramid:**
```
        /  E2E  \        ← Few: Full system tests
       / Integration\     ← Some: Service + DB, API tests
      /   Unit Tests  \   ← Many: Fast, isolated, mock dependencies
```

```java
// Unit test — fast, no Spring context
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock private UserRepository userRepo;
    @InjectMocks private UserService userService;

    @Test
    void createUser_shouldHashPassword() {
        // given, when, then pattern
    }
}

// Integration test — with Spring context + real DB
@SpringBootTest
@AutoConfigureMockMvc
@Testcontainers
class UserControllerIT {
    @Container
    static PostgreSQLContainer<?> db = new PostgreSQLContainer<>("postgres:16");

    @Autowired MockMvc mockMvc;

    @Test
    void createUser_shouldReturn201() throws Exception {
        mockMvc.perform(post("/api/v1/users")
            .contentType(APPLICATION_JSON)
            .content("""
                {"name": "Prince", "email": "prince@example.com", "password": "secret123"}
            """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("Prince"));
    }
}
```

---

## 8. Code Review Practices

| Check | Why |
|-------|-----|
| Logic correctness | Does it do what it should? |
| Edge cases | Nulls, empty lists, boundary values |
| Error handling | Proper exceptions, no swallowed errors |
| SQL performance | EXPLAIN on new queries, N+1 check |
| Security | Input validation, no SQL injection, no secrets in code |
| Naming | Clear, descriptive names |
| Tests | Are new paths covered? |
| Thread safety | Shared mutable state? Concurrent access? |

---

## 9. 12-Factor App Principles

| Factor | Spring Boot Implementation |
|--------|---------------------------|
| Codebase | One repo per service (Git) |
| Dependencies | Maven/Gradle (explicit) |
| Config | `application.yml` + environment variables |
| Backing services | DataSource, Redis, Kafka as attached resources |
| Build/release/run | CI/CD pipeline (build → release → deploy) |
| Processes | Stateless services (no in-memory sessions) |
| Port binding | Embedded Tomcat on configurable port |
| Concurrency | Horizontal scaling via container orchestration |
| Disposability | Fast startup, graceful shutdown |
| Dev/prod parity | Same Docker image, different config |
| Logs | Structured logs to stdout (collected by ELK/Datadog) |
| Admin processes | Flyway migrations, one-off scripts |

---

## 10. Project Structure

```
src/main/java/com/example/app/
├── config/              ← @Configuration classes
├── controller/          ← REST Controllers
├── dto/                 ← Request/Response DTOs
│   ├── request/
│   └── response/
├── entity/              ← JPA Entities
├── exception/           ← Custom exceptions + GlobalExceptionHandler
├── mapper/              ← Entity ↔ DTO mappers
├── repository/          ← JPA Repositories
├── service/             ← Business logic
│   └── impl/
├── security/            ← JWT, filters, SecurityConfig
├── util/                ← Utilities, constants
└── Application.java

src/main/resources/
├── application.yml
├── application-dev.yml
├── application-prod.yml
├── db/migration/        ← Flyway migration scripts
└── logback-spring.xml

src/test/java/com/example/app/
├── controller/          ← Integration tests
├── service/             ← Unit tests
└── repository/          ← Repository tests
```

---

## 11. Interview Notes

1. **Why separate DTOs from entities?** — Security (don't expose passwords), decoupling (API ≠ DB schema), prevent lazy-loading issues, circular references.
2. **How do you handle exceptions in Spring Boot?** — Custom exception hierarchy + `@RestControllerAdvice` + consistent error response (RFC 7807).
3. **What testing strategy do you follow?** — Test pyramid. Many unit tests (Mockito), some integration tests (SpringBootTest + Testcontainers), few E2E tests.
4. **What are 12-factor app principles?** — Best practices for cloud-native apps: stateless processes, externalized config, disposability, dev/prod parity.

---

## 12. Summary

| Practice | Key Rule |
|----------|---------|
| DTO vs Entity | Never expose entities via API |
| Exceptions | Global handler, custom hierarchy, consistent responses |
| Validation | Controller (format), Service (business), DB (integrity) |
| Logging | SLF4J, parameterized, MDC, never log secrets |
| Security | Validate inputs, hash passwords, protect secrets |
| Testing | Unit > Integration > E2E |

---

## 13. References

- [Spring Boot Best Practices](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [12-Factor App](https://12factor.net/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Martin Fowler — Testing](https://martinfowler.com/testing/)
- [Baeldung — Best Practices](https://www.baeldung.com/spring-boot-best-practices)

---

> **Previous:** [← 12 — Real-World Backend Issues](./12_Real_World_Backend_Issues_Fixes.md)
> **Next:** [14 — Interview Prep →](./14_Java_Backend_Interview_Prep.md)
