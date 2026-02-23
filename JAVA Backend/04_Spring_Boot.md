# 04 — Spring Boot

> **Goal:** Master how Spring Boot removes boilerplate, auto-configures your application, and provides production-ready features — from REST controllers to Actuator endpoints.

---

## Table of Contents

1. [What is Spring Boot?](#1-what-is-spring-boot)
2. [Auto-Configuration Internals](#2-auto-configuration-internals)
3. [Spring MVC & REST Controllers](#3-spring-mvc--rest-controllers)
4. [Request–Response Lifecycle](#4-requestresponse-lifecycle)
5. [Externalized Configuration](#5-externalized-configuration)
6. [Spring Boot Actuator](#6-spring-boot-actuator)
7. [Spring Events](#7-spring-events)
8. [Scheduling](#8-scheduling)
9. [Async Processing](#9-async-processing)
10. [WebFlux Basics](#10-webflux-basics)
11. [Interview Notes](#11-interview-notes)
12. [Summary](#12-summary)
13. [References](#13-references)

---

## 1. What is Spring Boot?

**Definition:**
Spring Boot is an opinionated framework built on top of Spring Framework that simplifies the creation and deployment of production-ready Spring applications. It eliminates boilerplate configuration through auto-configuration, starter dependencies, and an embedded server.

**Why it exists:**
Configuring a Spring application manually requires dozens of XML/Java config files, dependency management, server setup, and tuning. Spring Boot automates all of this — you write business code, Spring Boot handles the rest.

**Real-life analogy:**
Spring Framework is like getting all the car parts and assembly instructions. Spring Boot is like buying a fully assembled car — ready to drive. You can still customize the engine or seats, but it works out of the box.

**Backend example:**
```java
@SpringBootApplication // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args); // Starts embedded Tomcat
    }
}
```

**Key characteristics:**
- Embedded server (Tomcat, Jetty, or Undertow) — no WAR deployment needed
- Starter dependencies (`spring-boot-starter-web`, `spring-boot-starter-data-jpa`)
- Auto-configuration — sensible defaults, overridable
- Production-ready features (Actuator, health checks, metrics)
- DevTools for fast development cycle

---

## 2. Auto-Configuration Internals

**Definition:**
Auto-configuration is Spring Boot's mechanism for automatically configuring beans based on the classpath, properties, and existing beans. If `spring-boot-starter-data-jpa` is on the classpath and a DataSource is configured, Spring Boot automatically creates an EntityManagerFactory, TransactionManager, and JPA repositories.

**Why it exists:**
Manual configuration of DataSource, JPA, Jackson, Security, etc. is repetitive and error-prone. Auto-configuration applies sensible defaults, and you override only what you need.

**How it works:**

```
┌──────────────────────────────────────────────────────────┐
│                Auto-Configuration Flow                    │
│                                                          │
│  @SpringBootApplication                                  │
│       │                                                  │
│       ▼                                                  │
│  @EnableAutoConfiguration                                │
│       │                                                  │
│       ▼                                                  │
│  Reads META-INF/spring/                                  │
│  org.springframework.boot.autoconfigure.AutoConfiguration│
│  .imports                                                │
│       │                                                  │
│       ▼                                                  │
│  Evaluates @Conditional annotations:                     │
│  ┌────────────────────────────────────────┐              │
│  │ @ConditionalOnClass(DataSource.class)  │ → Yes?       │
│  │ @ConditionalOnMissingBean(DataSource)  │ → No custom? │
│  │ @ConditionalOnProperty("spring.ds...")  │ → Configured?│
│  └────────────────────────────────────────┘              │
│       │                                                  │
│       ▼                                                  │
│  Creates and registers auto-configured beans             │
└──────────────────────────────────────────────────────────┘
```

**Key `@Conditional` annotations:**

| Annotation | Condition |
|-----------|-----------|
| `@ConditionalOnClass` | Class is on classpath |
| `@ConditionalOnMissingBean` | No user-defined bean exists |
| `@ConditionalOnProperty` | Specific property is set |
| `@ConditionalOnBean` | Another bean exists |
| `@ConditionalOnWebApplication` | Running as web app |

**Debugging auto-configuration:**
```bash
# See what was auto-configured (and why)
java -jar app.jar --debug

# Or in application.yml
debug: true

# Produces a CONDITIONS EVALUATION REPORT showing:
# Positive matches (what was configured)
# Negative matches (what was skipped and why)
```

**Excluding auto-configuration:**
```java
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    SecurityAutoConfiguration.class
})
public class Application { ... }
```

---

## 3. Spring MVC & REST Controllers

### 3.1 @RestController

**Definition:**
`@RestController` is a specialized Spring annotation that combines `@Controller` and `@ResponseBody`. Every method in a `@RestController` returns data directly serialized to JSON/XML instead of rendering a view template.

**Backend example:**
```java
@RestController
@RequestMapping("/api/v1/products")
@RequiredArgsConstructor
@Slf4j
public class ProductController {

    private final ProductService productService;

    @GetMapping
    public ResponseEntity<Page<ProductDto>> getAllProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        return ResponseEntity.ok(productService.findAll(PageRequest.of(page, size)));
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductDto> getProductById(@PathVariable Long id) {
        return ResponseEntity.ok(productService.findById(id));
    }

    @PostMapping
    public ResponseEntity<ProductDto> createProduct(
            @RequestBody @Valid CreateProductRequest request) {
        ProductDto created = productService.create(request);
        URI location = URI.create("/api/v1/products/" + created.getId());
        return ResponseEntity.created(location).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<ProductDto> updateProduct(
            @PathVariable Long id,
            @RequestBody @Valid UpdateProductRequest request) {
        return ResponseEntity.ok(productService.update(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 3.2 Key Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@RequestMapping` | Base path for controller | `@RequestMapping("/api/v1/users")` |
| `@GetMapping` | HTTP GET | `@GetMapping("/{id}")` |
| `@PostMapping` | HTTP POST | `@PostMapping` |
| `@PutMapping` | HTTP PUT | `@PutMapping("/{id}")` |
| `@DeleteMapping` | HTTP DELETE | `@DeleteMapping("/{id}")` |
| `@PathVariable` | Extracts URI path variable | `@PathVariable Long id` |
| `@RequestParam` | Extracts query parameter | `@RequestParam String status` |
| `@RequestBody` | Deserializes JSON body | `@RequestBody CreateRequest req` |
| `@RequestHeader` | Extracts HTTP header | `@RequestHeader("Authorization") String token` |
| `@Valid` | Triggers Bean Validation | `@Valid CreateRequest req` |
| `@ResponseStatus` | Sets response status code | `@ResponseStatus(HttpStatus.CREATED)` |

---

## 4. Request–Response Lifecycle

**Definition:**
The request–response lifecycle in Spring Boot describes the complete journey of an HTTP request from the moment it hits the server to when the response is sent back to the client.

```
Client Request (HTTP)
       │
       ▼
┌──────────────┐
│  Embedded    │ (Tomcat/Jetty)
│  Servlet     │
│  Container   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Filter      │ (Security filters, CORS, logging)
│  Chain       │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Dispatcher   │ (Front controller — routes to handler)
│ Servlet      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Handler      │ (Finds the right @Controller method)
│ Mapping      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Handler      │ (Resolves @PathVariable, @RequestBody, etc.)
│ Adapter      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Interceptors │ (Pre-handle: auth checks, logging)
│ (preHandle)  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ @Controller  │ (Your business logic runs here)
│   Method     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Message      │ (Jackson serializes to JSON)
│ Converter    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Interceptors │ (Post-handle: response modification)
│ (postHandle) │
└──────┬───────┘
       │
       ▼
HTTP Response → Client
```

---

## 5. Externalized Configuration

**Definition:**
Externalized configuration is Spring Boot's mechanism for externalizing application settings (database URLs, credentials, feature flags) outside the code. Configuration can come from `application.yml`, environment variables, command-line arguments, or config servers.

**Why it exists:**
Hard-coding configuration makes deployments rigid. The same artifact should be deployable to dev, staging, and production by changing only configuration, not code.

### 5.1 Configuration Sources (Priority Order)

```
1. Command-line arguments (highest priority)
   java -jar app.jar --server.port=9090

2. OS environment variables
   export SERVER_PORT=9090

3. application-{profile}.yml
   (application-prod.yml, application-dev.yml)

4. application.yml (default)

5. @PropertySource

6. Default values (lowest priority)
```

### 5.2 @ConfigurationProperties (Type-Safe Config)

```java
// application.yml
app:
  mail:
    host: smtp.gmail.com
    port: 587
    sender: noreply@example.com
    templates:
      welcome: "welcome-email"
      reset: "password-reset"

// Type-safe config class
@Configuration
@ConfigurationProperties(prefix = "app.mail")
@Validated
@Getter @Setter
public class MailProperties {

    @NotBlank
    private String host;

    @Min(1) @Max(65535)
    private int port;

    @Email
    private String sender;

    private Map<String, String> templates;
}

// Usage — inject like any Spring bean
@Service
public class EmailService {
    private final MailProperties mailProperties;

    public void sendWelcome(User user) {
        String template = mailProperties.getTemplates().get("welcome");
        // ...
    }
}
```

---

## 6. Spring Boot Actuator

**Definition:**
Spring Boot Actuator provides production-ready monitoring and management endpoints for your application. It exposes health checks, metrics, environment info, thread dumps, and more — essential for operations teams and monitoring systems.

**Why it exists:**
In production, you need visibility into your application's health, performance, and configuration without reading logs or attaching debuggers. Actuator provides this out of the box.

**Setup:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, env, loggers, threaddump
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized
  health:
    db:
      enabled: true
    diskspace:
      enabled: true
```

**Key endpoints:**

| Endpoint | Purpose | URL |
|----------|---------|-----|
| `/actuator/health` | Application health status | Readiness/liveness probes |
| `/actuator/info` | Application info | Version, git commit |
| `/actuator/metrics` | Application metrics | JVM, HTTP, custom |
| `/actuator/env` | Configuration properties | Env vars, config |
| `/actuator/loggers` | View/change log levels at runtime | Debug in production |
| `/actuator/threaddump` | Current thread dump | Diagnose deadlocks |
| `/actuator/heapdump` | Heap dump (binary) | Memory analysis |

**Custom health indicator:**
```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    private final DataSource dataSource;

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(2)) {
                return Health.up()
                    .withDetail("database", "reachable")
                    .withDetail("responseTime", "< 2s")
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withDetail("database", "unreachable")
                .withException(e)
                .build();
        }
        return Health.down().build();
    }
}
```

---

## 7. Spring Events

**Definition:**
Spring Events is an implementation of the Observer pattern within the Spring container. Components can publish events, and other components can listen for and react to those events — enabling loose coupling between modules.

**Why it exists:**
Direct method calls between services create tight coupling. Events decouple publishers from subscribers — the order service doesn't need to know about notifications, inventory, or analytics. It just publishes an event.

**Real-life analogy:**
A newspaper publisher doesn't know who reads the paper. It publishes articles, and subscribers receive them. Adding a new subscriber doesn't require changing the publisher.

**Backend example:**
```java
// 1. Define the event
public record OrderPlacedEvent(
    Long orderId,
    Long userId,
    BigDecimal totalAmount,
    LocalDateTime placedAt
) {}

// 2. Publish the event
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public OrderDto placeOrder(CreateOrderRequest request) {
        Order order = orderRepository.save(toEntity(request));

        eventPublisher.publishEvent(new OrderPlacedEvent(
            order.getId(), request.getUserId(),
            order.getTotal(), LocalDateTime.now()
        ));

        return toDto(order);
    }
}

// 3. Listen to the event (multiple independent listeners)
@Component
@Slf4j
public class NotificationEventListener {

    @EventListener
    public void handleOrderPlaced(OrderPlacedEvent event) {
        log.info("Sending notification for order {}", event.orderId());
        // Send email/SMS notification
    }
}

@Component
public class InventoryEventListener {

    @EventListener
    public void handleOrderPlaced(OrderPlacedEvent event) {
        // Deduct inventory
    }
}

// 4. Async listener (non-blocking)
@Component
public class AnalyticsEventListener {

    @Async
    @EventListener
    public void handleOrderPlaced(OrderPlacedEvent event) {
        // Track analytics — runs in a separate thread
    }
}

// 5. Transactional event listener (runs after transaction commits)
@Component
public class PostCommitListener {

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleAfterCommit(OrderPlacedEvent event) {
        // Only runs if the transaction committed successfully
        // Safe for sending notifications, webhooks, etc.
    }
}
```

---

## 8. Scheduling

**Definition:**
Spring Scheduling allows you to execute methods at fixed intervals or cron expressions without external schedulers. Annotate a method with `@Scheduled`, and Spring invokes it automatically.

**Why it exists:**
Backends frequently need periodic tasks: cleanup stale sessions, send reminder emails, generate reports, sync data from external systems. Spring Scheduling handles this in-process.

**Backend example:**
```java
@Configuration
@EnableScheduling
public class SchedulingConfig { }

@Service
@Slf4j
public class ScheduledTasks {

    // Fixed rate — every 60 seconds (irrespective of execution time)
    @Scheduled(fixedRate = 60_000)
    public void cleanExpiredSessions() {
        int cleaned = sessionService.removeExpired();
        log.info("Cleaned {} expired sessions", cleaned);
    }

    // Fixed delay — 30 seconds AFTER previous execution finishes
    @Scheduled(fixedDelay = 30_000, initialDelay = 10_000)
    public void syncExternalData() {
        externalService.sync();
    }

    // Cron expression — every day at 2 AM
    @Scheduled(cron = "0 0 2 * * *")
    public void generateDailyReport() {
        reportService.generateDaily();
    }

    // Cron with timezone
    @Scheduled(cron = "0 0 9 * * MON-FRI", zone = "Asia/Kolkata")
    public void sendWeeklyDigest() {
        notificationService.sendDigest();
    }
}
```

**Cron expression format:**
```
┌──────────── second (0-59)
│ ┌────────── minute (0-59)
│ │ ┌──────── hour (0-23)
│ │ │ ┌────── day of month (1-31)
│ │ │ │ ┌──── month (1-12)
│ │ │ │ │ ┌── day of week (0-7, 0/7=Sun, or MON-SUN)
│ │ │ │ │ │
* * * * * *
```

---

## 9. Async Processing

**Definition:**
Async processing in Spring Boot allows methods to execute in a separate thread, freeing the caller to continue without waiting. Annotate a method with `@Async`, and Spring executes it on a different thread from a configured thread pool.

**Why it exists:**
Some tasks (sending emails, processing images, calling external APIs) don't need to block the main request thread. Async processing improves response times by offloading non-critical work.

**Backend example:**
```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "asyncExecutor")
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}

@Service
public class UserService {

    @Async("asyncExecutor")
    public CompletableFuture<UserAnalytics> computeAnalytics(Long userId) {
        // Runs in asyncExecutor thread pool
        UserAnalytics analytics = analyzeUser(userId);
        return CompletableFuture.completedFuture(analytics);
    }

    @Async("asyncExecutor")
    public void sendWelcomeEmail(User user) {
        // Fire-and-forget — caller doesn't wait
        emailService.send(user.getEmail(), "Welcome!", "...");
    }
}

// Controller usage
@PostMapping("/users")
public ResponseEntity<UserDto> createUser(@RequestBody @Valid CreateUserRequest req) {
    UserDto user = userService.create(req);
    userService.sendWelcomeEmail(user); // Non-blocking — returns immediately
    return ResponseEntity.status(HttpStatus.CREATED).body(user);
}
```

**Key rules:**
- `@Async` method MUST be in a different class than the caller (proxy limitation)
- Return `void` for fire-and-forget, `CompletableFuture<T>` for retrievable results
- Always configure a custom thread pool — default uses `SimpleAsyncTaskExecutor` (no pool!)
- Handle exceptions with `AsyncUncaughtExceptionHandler`

---

## 10. WebFlux Basics

**Definition:**
Spring WebFlux is Spring's reactive web framework, built on Project Reactor. It provides non-blocking, backpressure-aware request handling using `Mono` (0-1 elements) and `Flux` (0-N elements) reactive types.

**Why it exists:**
Traditional Spring MVC uses one thread per request. Under high concurrency (10K+ concurrent connections), this model exhausts thread pools. WebFlux uses event loops and non-blocking I/O, enabling many more concurrent connections with fewer threads.

**Real-life analogy:**
Spring MVC is like a restaurant where each waiter serves one table exclusively until the meal is over. Spring WebFlux is like a single waiter efficiently serving multiple tables — taking an order at table 1, delivering food to table 3, checking on table 2 — because no single task "blocks" the waiter.

**Backend example:**
```java
// WebFlux controller
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    private final ProductService productService;

    @GetMapping("/{id}")
    public Mono<ProductDto> getProduct(@PathVariable Long id) {
        return productService.findById(id); // Returns immediately, emits later
    }

    @GetMapping
    public Flux<ProductDto> getAllProducts() {
        return productService.findAll(); // Streams results as they become available
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<ProductDto> createProduct(@RequestBody Mono<CreateProductRequest> request) {
        return request.flatMap(productService::create);
    }
}
```

**When to use WebFlux vs MVC:**

| Aspect | Spring MVC | Spring WebFlux |
|--------|-----------|----------------|
| Threading | Thread-per-request | Event loop (Netty) |
| Blocking | Supports blocking I/O | Non-blocking only |
| Scalability | Thousands of concurrent requests | Tens of thousands |
| Complexity | Simple, familiar | Steeper learning curve |
| DB support | JDBC, JPA | R2DBC (reactive) |
| Use when | Most APIs, CRUD apps | High-concurrency, streaming |

**When NOT to use WebFlux:**
- When your application uses blocking libraries (JDBC, JPA)
- Under moderate concurrency (<1000 concurrent requests)
- When your team is not familiar with reactive programming
- Most CRUD applications don't benefit from WebFlux

---

## 11. Interview Notes

### Spring Boot — Top Interview Questions

1. **What is the difference between Spring and Spring Boot?**
   - Spring is the framework (IoC, DI, AOP). Spring Boot is the opinionated layer that auto-configures Spring + provides embedded server + starter dependencies.

2. **How does auto-configuration work?**
   - `@EnableAutoConfiguration` loads classes from `META-INF/spring/AutoConfiguration.imports`. Each auto-config class uses `@Conditional` annotations to decide whether to configure beans.

3. **Explain the request lifecycle in Spring MVC.**
   - Request → Filters → DispatcherServlet → HandlerMapping → HandlerAdapter → Interceptors (pre) → Controller → MessageConverter → Interceptors (post) → Response.

4. **How do you handle external configuration?**
   - `application.yml`, environment variables, command-line args. Use `@ConfigurationProperties` for type-safe binding.

5. **What is Actuator? How do you secure it?**
   - Production monitoring endpoints. Secure via Spring Security — expose only `/health` and `/info` publicly; protect others with authentication.

6. **What is the difference between `@Async` and event listeners?**
   - `@Async` runs a specific method asynchronously. Event listeners decouple producers from consumers. `@Async @EventListener` combines both.

7. **When would you use WebFlux over MVC?**
   - High-concurrency scenarios (chat, streaming, SSE). Not for CRUD with blocking databases.

---

## 12. Summary

| Topic | Key Takeaway |
|-------|-------------|
| Spring Boot | Convention over configuration — auto-configures based on classpath |
| Auto-config | `@Conditional` annotations decide whether to create beans |
| Controllers | `@RestController` + `@RequestMapping` for REST APIs |
| Lifecycle | Filters → DispatcherServlet → Controller → MessageConverter |
| Config | `@ConfigurationProperties` for type-safe, validated configuration |
| Actuator | `/health`, `/metrics`, `/loggers` — essential for production |
| Events | Decouple modules; use `@TransactionalEventListener` for post-commit logic |
| Scheduling | `@Scheduled` for periodic tasks; cron expressions for complex schedules |
| Async | `@Async` for non-blocking operations; always configure a thread pool |
| WebFlux | Reactive, non-blocking — use only for high-concurrency scenarios |

---

## 13. References

- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Boot Auto-configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.auto-configuration)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Baeldung — Spring Boot](https://www.baeldung.com/spring-boot)
- [Baeldung — Spring Events](https://www.baeldung.com/spring-events)
- [Spring WebFlux Documentation](https://docs.spring.io/spring-framework/reference/web/webflux.html)
- [Project Reactor](https://projectreactor.io/)

---

> **Previous:** [← 03 — Spring Core](./03_Spring_Core.md)  
> **Next:** [05 — Spring Data JPA & Hibernate →](./05_Spring_Data_JPA_Hibernate.md)
