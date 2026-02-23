# 03 — Spring Core

> **Goal:** Understand the foundation of the entire Spring ecosystem — the IoC container, Dependency Injection, Bean lifecycle, and Aspect-Oriented Programming.

---

## Table of Contents

1. [What is Spring Framework?](#1-what-is-spring-framework)
2. [Inversion of Control (IoC)](#2-inversion-of-control-ioc)
3. [Dependency Injection (DI)](#3-dependency-injection-di)
4. [Spring IoC Container](#4-spring-ioc-container)
5. [Bean Definition & Registration](#5-bean-definition--registration)
6. [Bean Scopes](#6-bean-scopes)
7. [Bean Lifecycle](#7-bean-lifecycle)
8. [BeanPostProcessor & BeanFactoryPostProcessor](#8-beanpostprocessor--beanfactorypostprocessor)
9. [Aspect-Oriented Programming (AOP)](#9-aspect-oriented-programming-aop)
10. [Spring Proxies](#10-spring-proxies)
11. [Qualifiers & Primary](#11-qualifiers--primary)
12. [Profiles](#12-profiles)
13. [Interview Notes](#13-interview-notes)
14. [Summary](#14-summary)
15. [References](#15-references)

---

## 1. What is Spring Framework?

**Definition:**
Spring Framework is a comprehensive Java application framework that provides infrastructure support for developing Java applications. At its core, it manages the creation, configuration, and lifecycle of application components (called "beans") through an Inversion of Control (IoC) container.

**Why it exists:**
Before Spring, Java backend code was tightly coupled — classes created their own dependencies, making testing, swapping implementations, and maintenance difficult. Spring decouples components, makes testing easy, and provides built-in support for transactions, security, and web development.

**Real-life analogy:**
Spring is like a building management company. You (the developer) design apartments (classes), but the management company (Spring container) handles plumbing (database connections), electrical wiring (configuration), security systems (Spring Security), and maintenance schedules (lifecycle management). You just define what goes where.

**Key characteristics:**
- Lightweight — no heavy EJB containers needed
- Non-invasive — your POJOs don't need to extend Spring classes
- Modular — use only what you need
- Testable — DI makes unit testing trivial
- Ecosystem — Spring Boot, Data, Security, Cloud are all built on Spring Core

---

## 2. Inversion of Control (IoC)

**Definition:**
Inversion of Control is a design principle where the control of object creation and dependency management is transferred from the application code to a framework or container. Instead of your code creating dependencies, the container creates them and provides them to you.

**Why it exists:**
Without IoC, every class creates and manages its own dependencies — leading to tight coupling, untestable code, and painful changes. IoC inverts this: the container manages everything.

**Real-life analogy:**
Without IoC: You cook your own meals at home (you control everything — shopping, cooking, cleaning).
With IoC: You go to a restaurant. You just order what you want (declare dependencies), and the restaurant (IoC container) handles procurement, cooking, and serving.

**Backend example:**
```java
// ❌ WITHOUT IoC — tight coupling
public class OrderService {
    private final OrderRepository repository = new MySqlOrderRepository(); // hard-coded
    private final EmailService emailService = new SmtpEmailService();       // hard-coded

    public void placeOrder(Order order) {
        repository.save(order);
        emailService.send(order.getUserEmail(), "Order placed");
    }
}
// Problems: Can't test without database, can't swap email provider

// ✅ WITH IoC (Spring) — loose coupling
@Service
public class OrderService {
    private final OrderRepository repository;   // interface
    private final EmailService emailService;     // interface

    public OrderService(OrderRepository repository, EmailService emailService) {
        this.repository = repository;            // Spring provides the implementation
        this.emailService = emailService;        // Spring provides the implementation
    }

    public void placeOrder(Order order) {
        repository.save(order);
        emailService.send(order.getUserEmail(), "Order placed");
    }
}
// Benefits: Testable (mock dependencies), swappable, clean
```

---

## 3. Dependency Injection (DI)

**Definition:**
Dependency Injection is the concrete mechanism through which IoC is achieved. It means that an object's dependencies are "injected" (provided) by an external source (the Spring container) rather than created by the object itself.

**Why it exists:**
DI enables loose coupling, testability, and flexibility. When dependencies are injected, you can easily swap implementations (e.g., mock for testing, different provider for production).

### 3.1 Types of Injection

#### Constructor Injection (Recommended ✅)

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    // Spring automatically injects — @Autowired is optional for single constructor
    public UserService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
}
```

**Why preferred:**
- Dependencies are `final` — immutable, thread-safe
- Clearly declares required dependencies
- Fails fast if a dependency is missing (at startup, not runtime)
- Easy to test (just pass mocks via constructor)

#### Setter Injection

```java
@Service
public class NotificationService {
    private EmailClient emailClient;

    @Autowired
    public void setEmailClient(EmailClient emailClient) {
        this.emailClient = emailClient;
    }
}
```

**When to use:** Optional dependencies that have defaults.

#### Field Injection (Avoid ❌)

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository; // ❌ Not recommended

    // Problems:
    // - Can't make it final
    // - Hidden dependencies
    // - Hard to test without Spring context
    // - Circumvents encapsulation
}
```

---

## 4. Spring IoC Container

**Definition:**
The Spring IoC Container is the core component that manages the complete lifecycle of application objects (beans). It reads configuration (annotations, Java config, or XML), creates beans, wires dependencies, and manages their scope and destruction.

**Two main containers:**

| Container | Interface | Description |
|-----------|-----------|-------------|
| BeanFactory | `BeanFactory` | Basic container, lazy initialization |
| ApplicationContext | `ApplicationContext` | Full-featured, extends BeanFactory. Adds event publishing, AOP, i18n, environment abstraction |

**Backend usage:** Always use `ApplicationContext` (Spring Boot uses `AnnotationConfigServletWebServerApplicationContext`).

**How it works:**
```
┌────────────────────────────────────────────────────────┐
│                   Spring IoC Container                  │
│                                                        │
│  1. Read @Configuration, @Component, @Service, etc.    │
│  2. Create BeanDefinitions (metadata)                   │
│  3. Instantiate beans in dependency order               │
│  4. Inject dependencies (constructor/setter/field)      │
│  5. Call lifecycle callbacks (@PostConstruct, etc.)      │
│  6. Beans are ready to use                              │
│  7. On shutdown: @PreDestroy, then destroy beans        │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │ UserService  │──│UserRepository│──│  DataSource  │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└────────────────────────────────────────────────────────┘
```

---

## 5. Bean Definition & Registration

**Definition:**
A Bean is any object that is managed by the Spring IoC container. Beans are defined via annotations or Java configuration and registered in the container at startup.

### 5.1 Stereotype Annotations

| Annotation | Purpose | Layer |
|-----------|---------|-------|
| `@Component` | Generic Spring-managed bean | Any |
| `@Service` | Business logic | Service layer |
| `@Repository` | Data access, exception translation | DAO layer |
| `@Controller` | Web MVC controller | Web layer |
| `@RestController` | REST controller (`@Controller` + `@ResponseBody`) | Web layer |
| `@Configuration` | Defines `@Bean` methods | Configuration |

```java
// Component scanning
@SpringBootApplication // includes @ComponentScan
public class Application { ... }

// Beans discovered automatically via classpath scanning
@Repository
public class JpaUserRepository implements UserRepository { ... }

@Service
public class UserService { ... }

@RestController
public class UserController { ... }
```

### 5.2 Java Configuration (`@Bean`)

```java
@Configuration
public class AppConfig {

    @Bean // Registers a bean manually
    public RestTemplate restTemplate() {
        return new RestTemplateBuilder()
            .connectTimeout(Duration.ofSeconds(5))
            .readTimeout(Duration.ofSeconds(10))
            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    @Bean
    @ConditionalOnProperty(name = "app.cache.enabled", havingValue = "true")
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "orders");
    }
}
```

---

## 6. Bean Scopes

**Definition:**
A bean scope defines the lifecycle and visibility of a bean instance within the Spring container. It determines how many instances of a bean are created and how long they live.

| Scope | Description | Instances | Use Case |
|-------|-------------|-----------|----------|
| `singleton` (default) | One instance per container | 1 | Services, Repositories, Config |
| `prototype` | New instance per request for bean | N | Stateful objects, builders |
| `request` | One per HTTP request | 1/request | Request-scoped data |
| `session` | One per HTTP session | 1/session | Session data (rare in REST) |
| `application` | One per ServletContext | 1 | Application-wide shared state |

**Backend example:**
```java
// Singleton (default) — one instance shared by all
@Service
@Scope("singleton") // this is the default, no need to specify
public class UserService { ... }

// Prototype — new instance every time it's requested
@Component
@Scope("prototype")
public class ReportGenerator {
    private final List<String> lines = new ArrayList<>();
    // Each injection gets a fresh instance
}

// Request scope — one per HTTP request
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestContext {
    private String correlationId;
    // Fresh instance for each HTTP request
}
```

**When to use:**
- **Singleton:** 95% of beans — services, repos, configs (stateless)
- **Prototype:** When you need a fresh instance each time (stateful processors)
- **Request:** Request-specific context (correlation IDs, user context)

---

## 7. Bean Lifecycle

**Definition:**
The bean lifecycle is the sequence of steps Spring follows from bean creation to destruction. Understanding it is essential for initializing resources (DB connections, caches) and cleaning up (closing connections, flushing buffers).

```
Container Startup
       │
       ▼
  Instantiation (new)
       │
       ▼
  Populate Properties (DI)
       │
       ▼
  BeanNameAware.setBeanName()
       │
       ▼
  BeanFactoryAware.setBeanFactory()
       │
       ▼
  ApplicationContextAware.setApplicationContext()
       │
       ▼
  BeanPostProcessor.postProcessBeforeInitialization()
       │
       ▼
  @PostConstruct
       │
       ▼
  InitializingBean.afterPropertiesSet()
       │
       ▼
  Custom init-method
       │
       ▼
  BeanPostProcessor.postProcessAfterInitialization()
       │
       ▼
  ═══ BEAN READY ═══
       │
       ▼ (container shutdown)
  @PreDestroy
       │
       ▼
  DisposableBean.destroy()
       │
       ▼
  Custom destroy-method
```

**Backend example:**
```java
@Service
public class CacheWarmupService {

    private final ProductRepository productRepository;
    private final CacheManager cacheManager;

    @PostConstruct // Runs after DI is complete
    public void warmUpCache() {
        log.info("Warming up product cache...");
        List<Product> topProducts = productRepository.findTop100ByOrderBySalesDesc();
        Cache cache = cacheManager.getCache("products");
        topProducts.forEach(p -> cache.put(p.getId(), p));
        log.info("Cache warmed up with {} products", topProducts.size());
    }

    @PreDestroy // Runs before bean is destroyed
    public void cleanUp() {
        log.info("Clearing product cache...");
        cacheManager.getCache("products").clear();
    }
}
```

---

## 8. BeanPostProcessor & BeanFactoryPostProcessor

### 8.1 BeanPostProcessor

**Definition:**
A BeanPostProcessor is a hook that allows you to modify or wrap bean instances before and after their initialization. Spring's own features (AOP proxies, `@Autowired` resolution, `@Transactional`) are implemented using BeanPostProcessors.

**Backend example:**
```java
@Component
public class LoggingBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        if (bean.getClass().isAnnotationPresent(Service.class)) {
            log.debug("Initializing service: {}", beanName);
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        // Could wrap bean in a proxy here
        return bean;
    }
}
```

### 8.2 BeanFactoryPostProcessor

**Definition:**
A BeanFactoryPostProcessor modifies bean **definitions** (metadata) before any beans are created. It operates at the configuration level, not on instances.

**Backend example:**
```java
// PropertySourcesPlaceholderConfigurer is the most common BFPP
// It resolves @Value("${property}") placeholders
@Component
public class CustomBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory factory) {
        BeanDefinition bd = factory.getBeanDefinition("dataSource");
        bd.getPropertyValues().addPropertyValue("maxPoolSize", 20);
    }
}
```

---

## 9. Aspect-Oriented Programming (AOP)

**Definition:**
AOP is a programming paradigm that enables modularizing cross-cutting concerns — functionalities that span multiple classes and layers (logging, security, transactions, metrics). Instead of duplicating this code, AOP lets you define it once and apply it declaratively.

**Why it exists:**
Without AOP, every service method would contain boilerplate for logging, security checks, transaction management, and metrics. AOP separates these concerns, keeping business logic clean.

**Real-life analogy:**
Security cameras in a building. Every room (method) is monitored, but the cameras (aspects) are installed and managed separately. You don't embed a camera inside each room's furniture — you install them at entry points (join points) and they work transparently.

### 9.1 AOP Concepts

| Term | Definition | Example |
|------|-----------|---------|
| **Aspect** | A module that encapsulates a cross-cutting concern | `@Aspect` class for logging |
| **Join Point** | A point in execution where an aspect can be applied | Method execution |
| **Advice** | The action taken at a join point | `@Before`, `@After`, `@Around` |
| **Pointcut** | An expression that selects join points | `execution(* com.app.service.*.*(..))` |
| **Weaving** | The process of applying aspects to target objects | At runtime via proxies (Spring default) |

### 9.2 Advice Types

```java
@Aspect
@Component
@Slf4j
public class PerformanceAspect {

    // @Before — runs before the method
    @Before("execution(* com.app.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        log.info("Calling: {}", joinPoint.getSignature().getName());
    }

    // @AfterReturning — runs after successful return
    @AfterReturning(pointcut = "execution(* com.app.service.*.*(..))", returning = "result")
    public void logAfterReturn(JoinPoint joinPoint, Object result) {
        log.info("Method {} returned: {}", joinPoint.getSignature().getName(), result);
    }

    // @AfterThrowing — runs if method throws an exception
    @AfterThrowing(pointcut = "execution(* com.app.service.*.*(..))", throwing = "ex")
    public void logException(JoinPoint joinPoint, Exception ex) {
        log.error("Method {} threw: {}", joinPoint.getSignature().getName(), ex.getMessage());
    }

    // @Around — wraps the method (most powerful)
    @Around("@annotation(Timed)") // custom annotation
    public Object measureExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.nanoTime();
        try {
            Object result = joinPoint.proceed(); // execute the target method
            return result;
        } finally {
            long elapsed = (System.nanoTime() - start) / 1_000_000;
            log.info("Method {} took {} ms", joinPoint.getSignature().getName(), elapsed);
        }
    }
}
```

### 9.3 Common AOP Use Cases in Backend

| Use Case | Advice Type | Example |
|----------|------------|---------|
| Logging | `@Around` | Log method entry, exit, duration |
| Transaction management | `@Around` | `@Transactional` is AOP-powered |
| Security | `@Before` | `@PreAuthorize` checks |
| Caching | `@Around` | `@Cacheable` is AOP-powered |
| Rate limiting | `@Before` | Check rate limit before proceeding |
| Retry | `@Around` | `@Retryable` wraps with retry logic |
| Metrics | `@Around` | Track method call counts and latency |

---

## 10. Spring Proxies

**Definition:**
A Spring proxy is a wrapper object that Spring creates around your bean to intercept method calls and add behavioral aspects (transactions, security, caching). When you inject a bean annotated with `@Transactional`, you're actually injecting a proxy, not the original object.

**Why it exists:**
Spring AOP works by creating proxies at runtime. `@Transactional`, `@Cacheable`, `@Async`, and `@PreAuthorize` all require proxying to intercept calls and add behavior before/after the actual method execution.

### 10.1 JDK Dynamic Proxy vs CGLIB

| Feature | JDK Dynamic Proxy | CGLIB Proxy |
|---------|-------------------|-------------|
| Mechanism | Implements the interfaces | Subclasses the class |
| Requirement | Bean must implement an interface | Any class (non-final) |
| Speed | Slightly faster creation | Slightly faster invocation |
| Default | Used when interface exists | Used when no interface (Spring Boot default) |

**The self-invocation trap:**
```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(Order order) {
        // ... save order
        this.notifyUser(order); // ❌ Self-invocation — SKIPS the proxy!
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void notifyUser(Order order) {
        // This @Transactional WON'T work because there's no proxy intercept
    }
}

// ✅ FIX: Inject self or extract to another bean
@Service
public class OrderService {
    @Lazy @Autowired private OrderService self; // proxy-aware self-reference

    @Transactional
    public void placeOrder(Order order) {
        // ...
        self.notifyUser(order); // ✅ Goes through the proxy
    }
}
```

---

## 11. Qualifiers & Primary

**Definition:**
When multiple beans implement the same interface, Spring needs to know which one to inject. `@Qualifier` specifies the exact bean by name, while `@Primary` marks a bean as the default choice.

**Backend example:**
```java
public interface StorageService {
    void store(byte[] data, String path);
}

@Service("localStorage")
public class LocalStorageService implements StorageService { ... }

@Service("s3Storage")
@Primary // Default when no qualifier is specified
public class S3StorageService implements StorageService { ... }

// Usage
@Service
public class FileUploadService {
    // Injects S3StorageService (marked @Primary)
    public FileUploadService(StorageService storageService) { ... }

    // Or explicitly choose
    public FileUploadService(@Qualifier("localStorage") StorageService storageService) { ... }
}
```

---

## 12. Profiles

**Definition:**
Spring Profiles provide a way to segregate parts of your application configuration and make them available only in specific environments. A bean or configuration class annotated with `@Profile` is only loaded when that profile is active.

**Backend example:**
```java
// Dev profile — use in-memory database
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}

// Prod profile — use production database
@Configuration
@Profile("prod")
public class ProdConfig {
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://prod-db:5432/myapp");
        return new HikariDataSource(config);
    }
}

// Activate via application.yml
spring:
  profiles:
    active: dev
```

---

## 13. Interview Notes

### Spring Core — Top Interview Questions

1. **What is IoC and DI? How are they different?**
   - IoC is the principle (framework controls object creation). DI is the mechanism (injecting dependencies via constructor/setter). IoC is the "what," DI is the "how."

2. **Why is constructor injection preferred?**
   - Immutable (`final` fields), fails fast, clear dependencies, easy to test, no reflection hacks.

3. **Explain the Bean lifecycle.**
   - Instantiation → DI → Aware callbacks → `@PostConstruct` → BeanPostProcessor → Ready → `@PreDestroy` → Destroy.

4. **What is the difference between `@Component`, `@Service`, `@Repository`?**
   - All register beans. `@Repository` adds exception translation. `@Service` is semantic (business logic). `@Controller`/`@RestController` is for web layer. All are `@Component` specializations.

5. **Explain AOP with a real use case.**
   - Cross-cutting concerns. Example: `@Transactional` uses `@Around` advice to start/commit/rollback transactions. Logging aspect tracks method execution time.

6. **What is the proxy self-invocation problem?**
   - Calling a `@Transactional` method from within the same class bypasses the proxy. Fix: inject self, or extract to another bean.

7. **What are Bean scopes? When would you use prototype?**
   - Singleton (default, 95%), prototype (new instance each time — for stateful objects), request, session.

8. **What is `@Primary` vs `@Qualifier`?**
   - `@Primary` sets default implementation. `@Qualifier` explicitly names which bean to inject. `@Qualifier` overrides `@Primary`.

---

## 14. Summary

| Topic | Key Takeaway |
|-------|-------------|
| IoC | Framework manages objects, not your code |
| DI | Constructor injection is king — immutable, testable |
| Container | ApplicationContext manages all beans |
| Stereotypes | `@Service`, `@Repository`, `@Controller` for semantic clarity |
| Scopes | Singleton default; prototype for stateful beans |
| Lifecycle | `@PostConstruct` for init, `@PreDestroy` for cleanup |
| BeanPostProcessor | How Spring implements `@Autowired`, `@Transactional` internally |
| AOP | Cross-cutting concerns — logging, transactions, security |
| Proxies | Self-invocation bypasses proxy — be careful with `@Transactional` |

---

## 15. References

- [Spring Framework Documentation](https://docs.spring.io/spring-framework/reference/)
- [Spring IoC Container](https://docs.spring.io/spring-framework/reference/core/beans.html)
- [Baeldung — Spring Core](https://www.baeldung.com/spring-core-annotations)
- [Spring AOP Documentation](https://docs.spring.io/spring-framework/reference/core/aop.html)
- [Baeldung — AOP](https://www.baeldung.com/spring-aop)
- [Martin Fowler — Inversion of Control](https://martinfowler.com/articles/injection.html)
- [Baeldung — Bean Lifecycle](https://www.baeldung.com/spring-bean-lifecycle)

---

> **Previous:** [← 02 — HTTP, REST & Backend Basics](./02_HTTP_REST_Backend_Basics.md)  
> **Next:** [04 — Spring Boot →](./04_Spring_Boot.md)
