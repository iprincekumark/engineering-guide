# 01 — Java Backend Fundamentals

> **Goal:** Master the Core Java concepts that directly impact backend development — from OOP and Collections to JVM internals and Design Patterns.

---

## Table of Contents

1. [Object-Oriented Programming (Deep)](#1-object-oriented-programming-deep)
2. [Collections Internals](#2-collections-internals)
3. [Streams API](#3-streams-api)
4. [Exception Handling](#4-exception-handling)
5. [Multithreading & Concurrency](#5-multithreading--concurrency)
6. [JVM Internals](#6-jvm-internals)
7. [Java Memory Model](#7-java-memory-model)
8. [Garbage Collection](#8-garbage-collection)
9. [Design Principles (SOLID & Beyond)](#9-design-principles-solid--beyond)
10. [Design Patterns (Backend-Relevant)](#10-design-patterns-backend-relevant)
11. [Interview Notes](#11-interview-notes)
12. [Summary](#12-summary)
13. [References](#13-references)

---

## 1. Object-Oriented Programming (Deep)

### 1.1 Encapsulation

**Definition:**
Encapsulation is the practice of bundling data (fields) and the methods that operate on that data into a single unit (a class), while restricting direct access to the internal state from outside code.

**Why it exists:**
In backend systems, encapsulation protects the integrity of an object's state. Without it, any piece of code could modify internal fields directly, leading to invalid data, broken invariants, and unpredictable behavior across services.

**Real-life analogy:**
A bank ATM lets you withdraw money via a defined interface (enter PIN → select amount → get cash). You cannot reach inside the machine and grab bills. The ATM encapsulates its internal mechanism.

**Backend example:**
```java
public class BankAccount {
    private BigDecimal balance; // hidden from outside

    public BankAccount(BigDecimal initialBalance) {
        if (initialBalance.compareTo(BigDecimal.ZERO) < 0)
            throw new IllegalArgumentException("Balance cannot be negative");
        this.balance = initialBalance;
    }

    public void withdraw(BigDecimal amount) {
        if (amount.compareTo(balance) > 0)
            throw new InsufficientFundsException("Not enough balance");
        this.balance = this.balance.subtract(amount);
    }

    public BigDecimal getBalance() {
        return balance;
    }
}
```

**Key characteristics:**
- Fields are `private`; access is through getters/setters or business methods
- Validates state transitions inside the class
- Reduces coupling — external code depends on the interface, not on internal structure
- Enables changing implementation without breaking clients

**When to use:**
Always — every entity, service, and DTO in a backend should expose only what's necessary.

**When NOT to use:**
Data classes like DTOs may use `public` fields or Java Records for simplicity when no invariants need protection.

---

### 1.2 Inheritance

**Definition:**
Inheritance is a mechanism where a new class (child/subclass) acquires the properties and behaviors of an existing class (parent/superclass), enabling code reuse and hierarchical classification.

**Why it exists:**
Backend systems share common logic across related types. Inheritance lets you define that logic once and specialize it where needed, reducing duplication.

**Real-life analogy:**
A "Vehicle" defines common properties (speed, fuel type). A "Truck" inherits from Vehicle but adds payload capacity. A "Car" adds passenger count.

**Backend example:**
```java
public abstract class BaseEntity {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}

@Entity
public class User extends BaseEntity {
    private String username;
    private String email;
}

@Entity
public class Order extends BaseEntity {
    private String orderNumber;
    private BigDecimal totalAmount;
}
```

**Key characteristics:**
- Java supports single inheritance (one parent class)
- Use `extends` for classes, `implements` for interfaces
- Subclass inherits non-private members
- `super()` calls parent constructor
- `@Override` indicates method redefinition

**When to use:**
- Shared base entities (auditing fields)
- Template Method pattern in services
- Framework extension points (e.g., extending `OncePerRequestFilter`)

**When NOT to use:**
- When you only need shared behavior (use composition instead)
- Deep inheritance hierarchies (> 2 levels) — they become brittle
- "Is-a" relationship doesn't genuinely apply

---

### 1.3 Polymorphism

**Definition:**
Polymorphism is the ability of a single interface or method to behave differently depending on the actual object type at runtime (runtime polymorphism) or the method signature at compile time (compile-time polymorphism/overloading).

**Why it exists:**
Backend systems must handle varying behavior through a uniform API. Polymorphism lets you write generic code that works with any implementation — essential for strategy selection, plugin architectures, and service abstraction.

**Real-life analogy:**
A "pay" button on a checkout page works the same way for the user, but behind the scenes it may process via credit card, UPI, or PayPal. Same interface, different behavior.

**Backend example:**
```java
// Runtime polymorphism via interface
public interface NotificationService {
    void send(String to, String message);
}

@Service
public class EmailNotificationService implements NotificationService {
    @Override
    public void send(String to, String message) {
        // Send via SMTP
    }
}

@Service
public class SmsNotificationService implements NotificationService {
    @Override
    public void send(String to, String message) {
        // Send via SMS gateway
    }
}

// Usage — caller doesn't care about the implementation
@Service
public class OrderService {
    private final NotificationService notificationService;

    public OrderService(@Qualifier("emailNotificationService") NotificationService ns) {
        this.notificationService = ns;
    }

    public void placeOrder(Order order) {
        // ... process order
        notificationService.send(order.getUserEmail(), "Order placed!");
    }
}
```

**Key characteristics:**
- **Compile-time (overloading):** Same method name, different parameters
- **Runtime (overriding):** Subclass provides a specific implementation of a parent method
- JVM uses **virtual method dispatch** to resolve the correct method at runtime
- Interfaces enable polymorphism without inheritance coupling

**When to use:**
- Strategy pattern for interchangeable algorithms
- Multiple notification channels, payment gateways, storage backends
- Any scenario requiring "same API, different behavior"

---

### 1.4 Abstraction

**Definition:**
Abstraction is the process of hiding complex implementation details and exposing only the essential features of an object or system through well-defined interfaces.

**Why it exists:**
Backend services interact with databases, caches, message queues, and third-party APIs. Abstraction shields consumers from underlying complexity — a repository hides SQL, a service hides business rules.

**Real-life analogy:**
A car driver uses a steering wheel, pedals, and gear shift. They don't interact with the engine's combustion process, fuel injection, or transmission internals.

**Backend example:**
```java
// Abstraction via interface — hides whether we use JPA, JDBC, or MongoDB
public interface UserRepository {
    User findById(Long id);
    User save(User user);
    void deleteById(Long id);
}

// Implementation detail — hidden from the service layer
@Repository
public class JpaUserRepository implements UserRepository {
    @PersistenceContext
    private EntityManager em;

    @Override
    public User findById(Long id) {
        return em.find(User.class, id);
    }
    // ...
}
```

**Key characteristics:**
- Achieved through `abstract` classes and `interfaces`
- Hides "how" and exposes "what"
- Enables loose coupling between layers
- Foundation of clean architecture

**When to use:**
- Service ↔ Repository boundaries
- External API clients (wrap third-party SDKs behind an interface)
- Any boundary where implementations may change

---

## 2. Collections Internals

### 2.1 HashMap

**Definition:**
HashMap is a hash-table-based implementation of the `Map` interface that stores key-value pairs. It uses hashing to compute the bucket index for each key, providing O(1) average-case lookup.

**Why it exists:**
Backends constantly need fast key-based lookups — caching user sessions, deduplicating data, counting frequencies, building indexes.

**Real-life analogy:**
A library card catalog where each book is filed under a unique code. You compute the filing location from the code (hash), go directly there, and find the book.

**Backend example:**
```java
// Caching user data in-memory
Map<Long, UserDto> userCache = new HashMap<>();
userCache.put(user.getId(), toDto(user));
UserDto cached = userCache.get(userId); // O(1)
```

**Key characteristics:**
- **Hashing:** `key.hashCode()` → bucket index via `(n - 1) & hash`
- **Collision handling:** JDK 8+ uses linked list → red-black tree (when bucket ≥ 8 nodes)
- **Capacity & Load factor:** Default capacity 16, load factor 0.75 → rehashes at 12 entries
- **NOT thread-safe** — use `ConcurrentHashMap` for concurrent access
- **Allows one `null` key** and multiple `null` values
- **No ordering guarantee**

**When to use:**
- Fast lookups by key
- In-memory caches (small scale)
- Frequency counting, grouping

**When NOT to use:**
- When you need sorted keys → use `TreeMap`
- When you need insertion order → use `LinkedHashMap`
- When multiple threads write → use `ConcurrentHashMap`

---

### 2.2 ConcurrentHashMap

**Definition:**
ConcurrentHashMap is a thread-safe variant of HashMap that allows concurrent reads and writes without locking the entire map. It uses fine-grained locking (or CAS operations in JDK 8+) at the bucket level.

**Why it exists:**
Backend servers handle hundreds of concurrent requests. A plain HashMap with external synchronization creates a bottleneck. ConcurrentHashMap enables high-throughput concurrent access.

**Real-life analogy:**
A large warehouse with multiple loading docks. Different workers can load/unload at different docks simultaneously without blocking each other.

**Backend example:**
```java
// Thread-safe in-memory rate limiter
private final ConcurrentHashMap<String, AtomicInteger> requestCounts = new ConcurrentHashMap<>();

public boolean allowRequest(String clientIp) {
    AtomicInteger count = requestCounts.computeIfAbsent(clientIp, k -> new AtomicInteger(0));
    return count.incrementAndGet() <= MAX_REQUESTS_PER_MINUTE;
}
```

**Key characteristics:**
- No `null` keys or values (unlike HashMap)
- JDK 8+ uses CAS + `synchronized` on individual bins
- `compute()`, `merge()`, `computeIfAbsent()` are atomic operations
- Read operations generally don't block
- Higher memory overhead than HashMap

**When to use:**
- Shared mutable state across threads
- In-memory caches accessed by multiple request threads
- Rate limiters, counters, session stores

---

### 2.3 ArrayList vs LinkedList

**Definition:**
- **ArrayList:** A resizable array-backed list. Elements are stored in contiguous memory.
- **LinkedList:** A doubly-linked list. Each element (node) holds a reference to the previous and next node.

**Why it exists:**
Different access patterns require different data structures. Random access favors ArrayList; frequent insertions/removals at arbitrary positions may favor LinkedList (though in practice, ArrayList is almost always faster due to CPU cache locality).

**Backend example:**
```java
// ArrayList — most common, excellent for ordered collections
List<OrderItem> items = new ArrayList<>();
items.add(new OrderItem("Laptop", 1));
OrderItem first = items.get(0); // O(1) random access

// LinkedList — rare in backend, used for queue-like structures
Deque<Task> taskQueue = new LinkedList<>();
taskQueue.addLast(new Task("Process payment"));
Task next = taskQueue.pollFirst(); // O(1) from head
```

**Key characteristics:**

| Feature | ArrayList | LinkedList |
|---|---|---|
| Random access | O(1) | O(n) |
| Add at end | O(1) amortized | O(1) |
| Add at index | O(n) | O(1) if at iterator position |
| Memory | Compact, cache-friendly | Node overhead, scattered |
| Backend usage | ~95% of cases | Queues/deques only |

**When to use ArrayList:** Almost always — the default choice.
**When to use LinkedList:** Only when you need O(1) add/remove at both ends (Deque interface) and never random access.

---

### 2.4 HashSet, TreeSet, LinkedHashSet

**Definition:**
- **HashSet:** Unordered set backed by HashMap. O(1) add/contains/remove.
- **TreeSet:** Sorted set backed by red-black tree. O(log n) operations.
- **LinkedHashSet:** Insertion-ordered set backed by LinkedHashMap.

**Backend example:**
```java
// Deduplication
Set<String> processedOrderIds = new HashSet<>();
if (!processedOrderIds.add(orderId)) {
    log.warn("Duplicate order: {}", orderId);
    return;
}

// Sorted leaderboard
TreeSet<Score> leaderboard = new TreeSet<>(Comparator.comparingInt(Score::getPoints).reversed());
```

---

### 2.5 Queue and Deque

**Definition:**
- **Queue:** A FIFO (First-In-First-Out) collection. Elements are added at the tail and removed from the head.
- **Deque:** A double-ended queue. Elements can be added/removed from both ends.

**Backend example:**
```java
// Task queue for async processing
Queue<Runnable> taskQueue = new LinkedList<>();
taskQueue.offer(() -> sendEmail(user));

// Priority queue for scheduling
PriorityQueue<Job> jobQueue = new PriorityQueue<>(
    Comparator.comparingInt(Job::getPriority)
);
```

---

## 3. Streams API

### 3.1 What are Streams?

**Definition:**
The Streams API (introduced in Java 8) is a declarative pipeline for processing collections of data. A stream represents a sequence of elements that supports sequential and parallel aggregate operations (filter, map, reduce, collect).

**Why it exists:**
Backend code frequently transforms, filters, and aggregates data from databases and APIs. Streams replace verbose `for` loops with concise, readable, and composable pipelines.

**Real-life analogy:**
A factory assembly line: raw materials enter → each station (filter, transform, assemble) processes items → finished products come out at the end. You declare what each station does, not how items move between stations.

**Backend example:**
```java
// Transform entity list to DTOs, filtering active users
List<UserDto> activeUsers = users.stream()
    .filter(User::isActive)
    .map(user -> new UserDto(user.getId(), user.getName(), user.getEmail()))
    .sorted(Comparator.comparing(UserDto::getName))
    .collect(Collectors.toList());

// Group orders by status
Map<OrderStatus, List<Order>> ordersByStatus = orders.stream()
    .collect(Collectors.groupingBy(Order::getStatus));

// Calculate total revenue
BigDecimal totalRevenue = orders.stream()
    .map(Order::getAmount)
    .reduce(BigDecimal.ZERO, BigDecimal::add);
```

**Key characteristics:**
- **Lazy evaluation:** Intermediate operations (filter, map) don't execute until a terminal operation (collect, forEach) is invoked
- **Not reusable:** A stream can be consumed only once
- **Intermediate ops:** `filter()`, `map()`, `flatMap()`, `sorted()`, `distinct()`, `peek()`
- **Terminal ops:** `collect()`, `forEach()`, `reduce()`, `count()`, `findFirst()`, `anyMatch()`
- **Parallel streams:** `parallelStream()` — use cautiously; beneficial only for CPU-heavy operations on large datasets

**When to use:**
- Transforming collections (entity → DTO)
- Filtering, grouping, aggregating data
- Functional-style data pipelines

**When NOT to use:**
- Side-effect-heavy operations (use a `for` loop)
- Small collections where readability doesn't improve
- When you need index access during iteration

---

## 4. Exception Handling

### 4.1 Checked vs Unchecked Exceptions

**Definition:**
- **Checked exceptions** (subclass of `Exception` but not `RuntimeException`) must be declared in the method signature or caught. The compiler enforces handling.
- **Unchecked exceptions** (subclass of `RuntimeException`) do not require explicit handling. They typically represent programming errors.

**Why it exists:**
Backend systems interact with I/O, networks, and databases — all of which can fail. Exceptions provide a structured way to signal and handle failures without cluttering return values.

**Backend example:**
```java
// Checked — forces the caller to handle
public User findUserOrThrow(Long id) throws UserNotFoundException {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
}

// Unchecked — for programming errors
public void processOrder(Order order) {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
}
```

**Key characteristics:**

| Feature | Checked | Unchecked |
|---|---|---|
| Superclass | `Exception` | `RuntimeException` |
| Compiler enforced | Yes | No |
| Use for | Recoverable failures | Programming bugs |
| Backend preference | Less common now | More common (Spring uses unchecked) |

---

### 4.2 Backend Exception Strategy

**Definition:**
A backend exception strategy is a structured approach to creating, throwing, catching, and translating exceptions across application layers. It defines what types of exceptions exist, where they're thrown, how they're caught, and what HTTP responses they produce.

**Backend example:**
```java
// Custom business exception
public class ResourceNotFoundException extends RuntimeException {
    private final String resourceName;
    private final String fieldName;
    private final Object fieldValue;

    public ResourceNotFoundException(String resourceName, String fieldName, Object fieldValue) {
        super(String.format("%s not found with %s: '%s'", resourceName, fieldName, fieldValue));
        this.resourceName = resourceName;
        this.fieldName = fieldName;
        this.fieldValue = fieldValue;
    }
}

// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));

        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            LocalDateTime.now(),
            errors
        );
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

---

## 5. Multithreading & Concurrency

### 5.1 Thread Basics

**Definition:**
A thread is the smallest unit of execution within a process. In Java, every program has at least one thread (the `main` thread). Multithreading enables concurrent execution of tasks within the same JVM process.

**Why it exists:**
Backend servers handle thousands of simultaneous requests. Each request is typically handled by a thread (or virtual thread). Understanding threading is essential for writing safe, performant backend code.

**Real-life analogy:**
A restaurant kitchen with multiple chefs. Each chef (thread) works on a different order (request) concurrently. They share the same kitchen equipment (shared resources), so coordination is needed.

**Backend example:**
```java
// Spring Boot handles this automatically via thread pool
// Each HTTP request gets a thread from the pool
@GetMapping("/users/{id}")
public ResponseEntity<UserDto> getUser(@PathVariable Long id) {
    // This method runs on a thread from Tomcat's pool
    UserDto user = userService.findById(id);
    return ResponseEntity.ok(user);
}
```

---

### 5.2 Synchronization & Locks

**Definition:**
Synchronization is a mechanism that controls access to shared resources by multiple threads, ensuring that only one thread can execute a critical section at a time. Locks are explicit synchronization primitives (like `ReentrantLock`) that provide more flexibility than the `synchronized` keyword.

**Backend example:**
```java
// Using synchronized
public class SequenceGenerator {
    private long counter = 0;

    public synchronized long nextValue() {
        return ++counter;
    }
}

// Using ReentrantLock (more flexible)
public class TicketBookingService {
    private final ReentrantLock lock = new ReentrantLock();

    public boolean bookSeat(int seatNumber) {
        lock.lock();
        try {
            if (isSeatAvailable(seatNumber)) {
                markSeatBooked(seatNumber);
                return true;
            }
            return false;
        } finally {
            lock.unlock(); // Always unlock in finally
        }
    }
}
```

---

### 5.3 Executor Framework & Thread Pools

**Definition:**
The Executor framework (`java.util.concurrent`) provides a higher-level API for managing threads. Instead of creating threads manually, you submit tasks to an executor that manages a pool of reusable threads.

**Why it exists:**
Creating a new thread per request is expensive (each thread ~1 MB stack). Thread pools reuse threads, control concurrency, and prevent resource exhaustion.

**Backend example:**
```java
// Custom thread pool for async operations
@Configuration
public class AsyncConfig {

    @Bean(name = "emailExecutor")
    public Executor emailExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("email-");
        executor.setRejectedExecutionHandler(new CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {

    @Async("emailExecutor")
    public CompletableFuture<Void> sendWelcomeEmail(User user) {
        // Runs on the emailExecutor thread pool
        emailClient.send(user.getEmail(), "Welcome!", buildBody(user));
        return CompletableFuture.completedFuture(null);
    }
}
```

**Key characteristics:**

| Pool Type | Description | Backend Use |
|---|---|---|
| `FixedThreadPool` | Fixed number of threads | Known concurrency limit |
| `CachedThreadPool` | Creates threads as needed, reuses idle | Short-lived tasks |
| `ScheduledThreadPool` | Scheduled/periodic tasks | Cron jobs, polling |
| `ForkJoinPool` | Work-stealing for recursive tasks | Parallel streams |
| Virtual Threads (JDK 21+) | Lightweight threads | High-concurrency I/O |

---

### 5.4 Virtual Threads (Project Loom — JDK 21+)

**Definition:**
Virtual threads are lightweight threads managed by the JVM (not the OS). They enable writing blocking I/O code in a simple, synchronous style while achieving the scalability of asynchronous/reactive code.

**Why it exists:**
Traditional OS threads are heavy (~1 MB each). A server with 10,000 concurrent requests needs 10,000 threads — that's ~10 GB just for stacks. Virtual threads use only a few KB each, enabling millions of concurrent tasks.

**Backend example:**
```java
// Spring Boot 3.2+ with virtual threads
# application.yml
spring:
  threads:
    virtual:
      enabled: true

// Each request now runs on a virtual thread automatically!
// No code changes needed in controllers/services
@GetMapping("/orders")
public List<OrderDto> getOrders() {
    // This runs on a virtual thread, not a platform thread
    return orderService.findAll();
}

// Manual usage
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<String>> futures = urls.stream()
        .map(url -> executor.submit(() -> fetchFromApi(url)))
        .toList();
}
```

**Key characteristics:**
- Managed by JVM, not OS
- Extremely cheap to create (KB vs MB)
- Perfect for I/O-bound backends
- Don't pool virtual threads — create new ones per task
- Avoid `synchronized` blocks with I/O (use `ReentrantLock` instead)
- Spring Boot 3.2+ supports virtual threads natively

---

### 5.5 CompletableFuture

**Definition:**
CompletableFuture is a class that represents a future result of an asynchronous computation. It supports chaining, combining, and composing async operations with callbacks.

**Backend example:**
```java
public OrderSummary getOrderSummary(Long orderId) {
    CompletableFuture<Order> orderFuture = CompletableFuture
        .supplyAsync(() -> orderService.findById(orderId));

    CompletableFuture<List<Payment>> paymentsFuture = CompletableFuture
        .supplyAsync(() -> paymentService.findByOrderId(orderId));

    CompletableFuture<ShippingInfo> shippingFuture = CompletableFuture
        .supplyAsync(() -> shippingService.getInfo(orderId));

    // Combine all three — runs concurrently
    return CompletableFuture.allOf(orderFuture, paymentsFuture, shippingFuture)
        .thenApply(v -> new OrderSummary(
            orderFuture.join(),
            paymentsFuture.join(),
            shippingFuture.join()
        ))
        .join();
}
```

---

## 6. JVM Internals

### 6.1 JVM Architecture

**Definition:**
The Java Virtual Machine (JVM) is an abstract computing machine that executes Java bytecode. It provides platform independence ("write once, run anywhere") and manages memory, threads, and class loading.

**Why it exists:**
The JVM is the runtime that powers every Java backend. Understanding its architecture directly impacts your ability to diagnose performance problems, tune memory, and prevent production incidents.

**Real-life analogy:**
The JVM is like an operating system within an operating system. Your Java application is a tenant; the JVM provides it with memory management, security, and execution services.

**Key components:**

```
┌─────────────────────────────────────────────────────┐
│                    JVM Architecture                   │
├─────────────────────────────────────────────────────┤
│  Class Loader Subsystem                               │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐         │
│  │Bootstrap │→│Extension │→│Application   │         │
│  │Loader    │ │Loader    │ │Loader        │         │
│  └──────────┘ └──────────┘ └──────────────┘         │
├─────────────────────────────────────────────────────┤
│  Runtime Data Areas                                   │
│  ┌─────┐ ┌───────┐ ┌─────────┐ ┌────────┐ ┌──────┐│
│  │Heap │ │Method │ │  Stack  │ │   PC   │ │Native││
│  │     │ │ Area  │ │(per     │ │Register│ │Method││
│  │     │ │       │ │ thread) │ │        │ │Stack ││
│  └─────┘ └───────┘ └─────────┘ └────────┘ └──────┘│
├─────────────────────────────────────────────────────┤
│  Execution Engine                                     │
│  ┌──────────┐ ┌─────┐ ┌──────────────────┐          │
│  │Interpreter│ │ JIT │ │Garbage Collector │          │
│  └──────────┘ └─────┘ └──────────────────┘          │
└─────────────────────────────────────────────────────┘
```

**Key areas for backend developers:**
- **Heap:** Where all objects live. Divided into Young Gen (Eden + Survivor) and Old Gen.
- **Stack:** Per-thread memory for method calls and local variables.
- **Method Area (Metaspace):** Stores class metadata, static variables, constant pool.
- **JIT Compiler:** Compiles hot bytecode to native code for performance.

---

## 7. Java Memory Model

**Definition:**
The Java Memory Model (JMM) defines how threads interact through memory — specifically, the rules governing when a write by one thread becomes visible to a read by another thread.

**Why it exists:**
Modern CPUs use registers, caches, and instruction reordering for performance. Without the JMM, one thread could write a value and another thread could never see it. The JMM defines "happens-before" relationships that guarantee visibility.

**Real-life analogy:**
Two colleagues sharing a Google Doc. Without rules, if colleague A types a sentence, colleague B might not see it for minutes (or ever if they have a cached offline copy). The "happens-before" relationship is like Google Doc's real-time sync — it guarantees that after A saves, B sees the update.

**Key characteristics:**
- **`volatile`:** Guarantees visibility. Reads/writes go directly to main memory.
- **`synchronized`:** Provides both atomicity and visibility.
- **Happens-before:** `synchronized` unlock → subsequent lock; `volatile` write → subsequent read; `Thread.start()` → run's first statement.

**Backend example:**
```java
// Without volatile — thread may never see the update
public class ServerShutdownHook {
    private volatile boolean running = true; // volatile ensures visibility

    public void start() {
        new Thread(() -> {
            while (running) { // Reads 'running' from main memory each time
                processRequest();
            }
        }).start();
    }

    public void stop() {
        running = false; // Write is immediately visible to the reader thread
    }
}
```

---

## 8. Garbage Collection

### 8.1 GC Basics

**Definition:**
Garbage Collection (GC) is the JVM's automatic memory management process that identifies and reclaims memory occupied by objects that are no longer reachable from any live reference.

**Why it exists:**
Manual memory management (like C/C++) is error-prone (memory leaks, dangling pointers). GC automates this, letting developers focus on business logic. However, GC pauses can impact backend latency.

**Real-life analogy:**
A hotel housekeeping service. Once a guest checks out (the object has no live references), housekeeping cleans the room (reclaims memory). You never worry about cleaning rooms yourself, but if housekeeping runs during peak hours, it slows down check-in.

**Key characteristics:**

| GC Algorithm | Description | Use Case |
|---|---|---|
| Serial GC | Single-threaded, stops the world | Development, small apps |
| Parallel GC | Multi-threaded, stops the world | Throughput-oriented batches |
| G1 GC (default JDK 9+) | Region-based, balanced latency/throughput | General-purpose backend |
| ZGC (JDK 15+) | Ultra-low pause (<1 ms) | Latency-sensitive APIs |
| Shenandoah | Concurrent, low-pause | Alternative to ZGC |

**Backend example — tuning for a Spring Boot API:**
```bash
# JVM flags for a production REST API
java -jar app.jar \
  -XX:+UseZGC \
  -Xms2g -Xmx2g \
  -XX:+UseStringDeduplication \
  -XX:MaxGCPauseMillis=10
```

**When to use specific GCs:**
- **G1 GC:** Default choice for most Spring Boot applications
- **ZGC:** When you need consistent sub-millisecond GC pauses (trading platforms, real-time APIs)
- **Parallel GC:** Batch processing, data pipelines where throughput matters more than latency

---

## 9. Design Principles (SOLID & Beyond)

### 9.1 Single Responsibility Principle (SRP)

**Definition:**
A class should have only one reason to change — it should encapsulate exactly one responsibility or concern.

**Backend example:**
```java
// ❌ BAD — multiple responsibilities
public class UserService {
    public User createUser(UserDto dto) { /* ... */ }
    public void sendWelcomeEmail(User user) { /* ... */ }    // Email concern
    public String generateReport(List<User> users) { /* ... */ } // Report concern
}

// ✅ GOOD — each class has one responsibility
public class UserService {
    public User createUser(UserDto dto) { /* ... */ }
}

public class EmailService {
    public void sendWelcomeEmail(User user) { /* ... */ }
}

public class UserReportService {
    public String generateReport(List<User> users) { /* ... */ }
}
```

---

### 9.2 Open/Closed Principle (OCP)

**Definition:**
Software entities should be open for extension but closed for modification. You should be able to add new behavior without changing existing code.

**Backend example:**
```java
// ✅ Open for extension via new implementations
public interface PaymentProcessor {
    PaymentResult process(Payment payment);
}

@Component
public class CreditCardProcessor implements PaymentProcessor { /* ... */ }

@Component
public class UpiProcessor implements PaymentProcessor { /* ... */ }

// Adding PayPal requires no change to existing code
@Component
public class PayPalProcessor implements PaymentProcessor { /* ... */ }

// Registry pattern for dynamic dispatch
@Component
public class PaymentProcessorRegistry {
    private final Map<PaymentType, PaymentProcessor> processors;

    public PaymentProcessorRegistry(List<PaymentProcessor> allProcessors) {
        this.processors = allProcessors.stream()
            .collect(Collectors.toMap(PaymentProcessor::getType, Function.identity()));
    }

    public PaymentProcessor getProcessor(PaymentType type) {
        return Optional.ofNullable(processors.get(type))
            .orElseThrow(() -> new UnsupportedPaymentTypeException(type));
    }
}
```

---

### 9.3 Liskov Substitution, Interface Segregation, Dependency Inversion

**Liskov Substitution (LSP):**
Subtypes must be substitutable for their base types without breaking the program.

**Interface Segregation (ISP):**
Clients should not be forced to depend on interfaces they don't use. Prefer small, focused interfaces.

```java
// ❌ BAD — fat interface
public interface UserOperations {
    User create(UserDto dto);
    void delete(Long id);
    void sendNotification(User user);
    Report generateReport();
}

// ✅ GOOD — segregated interfaces
public interface UserCrudService { User create(UserDto dto); void delete(Long id); }
public interface NotificationService { void sendNotification(User user); }
public interface ReportService { Report generateReport(); }
```

**Dependency Inversion (DIP):**
High-level modules should not depend on low-level modules. Both should depend on abstractions.

```java
// ❌ BAD — service depends on concrete class
public class OrderService {
    private MySqlOrderRepository repository = new MySqlOrderRepository();
}

// ✅ GOOD — depends on abstraction, injected
public class OrderService {
    private final OrderRepository repository; // interface

    public OrderService(OrderRepository repository) {
        this.repository = repository; // injected by Spring
    }
}
```

---

## 10. Design Patterns (Backend-Relevant)

### 10.1 Singleton Pattern

**Definition:**
Ensures a class has exactly one instance throughout the application and provides a global access point.

**Backend example:**
Spring beans are singletons by default:
```java
@Service // Singleton scope by default
public class CacheService {
    private final Map<String, Object> cache = new ConcurrentHashMap<>();
    // One instance shared across all requests
}
```

---

### 10.2 Factory Pattern

**Definition:**
Defines an interface for creating objects but lets subclasses or concrete factories decide which class to instantiate.

**Backend example:**
```java
@Component
public class NotificationFactory {

    public Notification create(NotificationType type, String message) {
        return switch (type) {
            case EMAIL -> new EmailNotification(message);
            case SMS   -> new SmsNotification(message);
            case PUSH  -> new PushNotification(message);
        };
    }
}
```

---

### 10.3 Builder Pattern

**Definition:**
Separates the construction of a complex object from its representation, enabling step-by-step construction.

**Backend example:**
```java
// Using Lombok @Builder
@Builder
@Getter
public class ApiResponse<T> {
    private final int status;
    private final String message;
    private final T data;
    private final LocalDateTime timestamp;
}

// Usage
ApiResponse<UserDto> response = ApiResponse.<UserDto>builder()
    .status(200)
    .message("User retrieved successfully")
    .data(userDto)
    .timestamp(LocalDateTime.now())
    .build();
```

---

### 10.4 Strategy Pattern

**Definition:**
Defines a family of interchangeable algorithms and makes them selectable at runtime. Each algorithm is encapsulated in its own class.

**Backend example:**
```java
public interface PricingStrategy {
    BigDecimal calculatePrice(Order order);
}

@Component
public class RegularPricing implements PricingStrategy {
    public BigDecimal calculatePrice(Order order) {
        return order.getBasePrice();
    }
}

@Component
public class PremiumPricing implements PricingStrategy {
    public BigDecimal calculatePrice(Order order) {
        return order.getBasePrice().multiply(new BigDecimal("0.85")); // 15% discount
    }
}

@Service
public class PricingService {
    private final Map<CustomerTier, PricingStrategy> strategies;

    public BigDecimal getPrice(Order order, CustomerTier tier) {
        return strategies.get(tier).calculatePrice(order);
    }
}
```

---

### 10.5 Observer Pattern

**Definition:**
Defines a one-to-many dependency so that when one object changes state, all its dependents are notified and updated automatically.

**Backend example (Spring Events):**
```java
// Event
public record OrderPlacedEvent(Long orderId, String userEmail, BigDecimal amount) {}

// Publisher
@Service
public class OrderService {
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public Order placeOrder(OrderRequest request) {
        Order order = orderRepository.save(toEntity(request));
        eventPublisher.publishEvent(new OrderPlacedEvent(
            order.getId(), request.getUserEmail(), order.getTotal()
        ));
        return order;
    }
}

// Listeners (observers)
@Component
public class OrderNotificationListener {
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        emailService.sendOrderConfirmation(event.userEmail(), event.orderId());
    }
}

@Component
public class InventoryListener {
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        inventoryService.reserveItems(event.orderId());
    }
}
```

---

### 10.6 Proxy Pattern

**Definition:**
Provides a surrogate or placeholder for another object to control access to it.

**Backend example:**
Spring AOP uses proxies extensively:
```java
@Transactional // Spring creates a proxy that wraps this method
public void transferFunds(Long fromId, Long toId, BigDecimal amount) {
    // The proxy starts a transaction BEFORE this code runs
    Account from = accountRepo.findById(fromId).orElseThrow();
    Account to = accountRepo.findById(toId).orElseThrow();
    from.debit(amount);
    to.credit(amount);
    // The proxy commits the transaction AFTER this code succeeds
    // Or rolls back if an exception is thrown
}
```

---

## 11. Interview Notes

### Core Java — Top Interview Questions

1. **What is the difference between `==` and `.equals()`?**
   - `==` compares references (memory address). `.equals()` compares logical equality (content). Always override `.equals()` and `hashCode()` together.

2. **Explain HashMap internals.**
   - Hashing → bucket index → linked list or red-black tree (≥8 nodes). Rehashing at load factor 0.75. Not thread-safe.

3. **What is the difference between `synchronized` and `ReentrantLock`?**
   - `synchronized` is implicit, block-scoped. `ReentrantLock` supports tryLock, timed lock, fairness, interruptible lock.

4. **Explain the Java Memory Model and `volatile`.**
   - JMM defines happens-before relationships. `volatile` ensures visibility across threads.

5. **What are virtual threads?**
   - JDK 21+ lightweight threads managed by JVM. Enable millions of concurrent tasks with simple blocking code.

6. **Stream vs Collection?**
   - Collection stores data. Stream processes data lazily. Streams are not reusable.

7. **How does GC work? Which GC would you use for a REST API?**
   - GC reclaims unreachable objects. G1 for general APIs, ZGC for latency-critical services.

8. **Explain SOLID principles with backend examples.**
   - SRP (one responsibility per class), OCP (extend without modifying), LSP (substitutable subtypes), ISP (small interfaces), DIP (depend on abstractions).

---

## 12. Summary

| Topic | Key Takeaway |
|---|---|
| OOP | Encapsulation protects state; Polymorphism enables flexible APIs |
| Collections | HashMap for lookups, ConcurrentHashMap for thread-safety, ArrayList for almost everything |
| Streams | Declarative data processing; lazy evaluation; don't overuse |
| Exceptions | Use unchecked for business errors; `@RestControllerAdvice` for global handling |
| Concurrency | Thread pools > raw threads; Virtual threads are the future |
| JVM | Heap (objects), Stack (per-thread), Metaspace (classes), JIT (hot code) |
| GC | G1 default; ZGC for low latency; tune heap, not code |
| SOLID | Foundation of maintainable backend code |
| Patterns | Strategy + Observer + Builder + Factory = most-used in Spring backends |

---

## 13. References

- [Oracle Java Tutorials](https://docs.oracle.com/javase/tutorial/)
- [Java Language Specification](https://docs.oracle.com/javase/specs/jls/se21/html/index.html)
- [Baeldung — Java Concurrency](https://www.baeldung.com/java-concurrency)
- [JEP 444 — Virtual Threads](https://openjdk.org/jeps/444)
- [Effective Java by Joshua Bloch (3rd Edition)](https://www.oreilly.com/library/view/effective-java/9780134686097/)
- [Java Memory Model — JSR 133 FAQ](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)
- [Baeldung — Design Patterns in Java](https://www.baeldung.com/design-patterns-series)
- [Spring Framework Documentation](https://docs.spring.io/spring-framework/reference/)

---

> **Next:** [02 — HTTP, REST & Backend Basics →](./02_HTTP_REST_Backend_Basics.md)
