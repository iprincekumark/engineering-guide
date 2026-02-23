# 08 — Backend Architecture & System Design

> **Goal:** Master architectural patterns and system design principles used in real Java backend systems.

---

## Table of Contents

1. [Layered Architecture](#1-layered-architecture)
2. [Clean Architecture](#2-clean-architecture)
3. [Hexagonal Architecture](#3-hexagonal-architecture)
4. [Domain-Driven Design (DDD) Basics](#4-domain-driven-design-ddd-basics)
5. [CQRS](#5-cqrs)
6. [Event Sourcing](#6-event-sourcing)
7. [System Design Methodology](#7-system-design-methodology)
8. [Common System Design Scenarios](#8-common-system-design-scenarios)
9. [Interview Notes](#9-interview-notes)
10. [Summary](#10-summary)
11. [References](#11-references)

---

## 1. Layered Architecture

**Definition:**
Layered architecture organizes code into horizontal layers where each layer has a specific responsibility and can only depend on the layer directly below it. It is the most common architecture in Spring Boot applications.

**Why it exists:**
Without layers, business logic, database access, and HTTP handling are mixed together — making code untestable, hard to maintain, and tightly coupled.

**Real-life analogy:**
A corporate hierarchy: CEO (Controller) issues directives, managers (Service) process and decide, and workers (Repository) execute the actual work. Each level communicates only with adjacent levels.

**Backend example:**
```
┌─────────────────────────────┐
│   Controller Layer          │  ← Handles HTTP, validation, routing
│   (REST Controllers)        │
├─────────────────────────────┤
│   Service Layer             │  ← Business logic, orchestration
│   (Business Services)       │
├─────────────────────────────┤
│   Repository Layer          │  ← Data access, queries
│   (JPA Repositories, DAOs)  │
├─────────────────────────────┤
│   Database                  │  ← Persistence
└─────────────────────────────┘
```

```java
// Controller — handles HTTP, delegates to service
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final OrderService orderService;
    @PostMapping
    public ResponseEntity<OrderDto> create(@RequestBody @Valid CreateOrderRequest req) {
        return ResponseEntity.status(201).body(orderService.createOrder(req));
    }
}

// Service — business logic
@Service
@Transactional
public class OrderService {
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    public OrderDto createOrder(CreateOrderRequest req) {
        inventoryService.reserveItems(req.getItems());
        Order order = orderRepository.save(toEntity(req));
        return toDto(order);
    }
}

// Repository — data access
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByUserIdAndStatus(Long userId, OrderStatus status);
}
```

**When to use:** Most CRUD applications, small-to-medium services.
**When NOT to use:** Complex domains with intricate business rules → consider Clean/Hexagonal.

---

## 2. Clean Architecture

**Definition:**
Clean Architecture (by Robert C. Martin) organizes code in concentric circles where inner layers represent business rules and outer layers represent infrastructure. Dependencies always point inward — business logic never depends on frameworks, databases, or web layers.

**Why it exists:**
In layered architecture, business logic often depends on JPA entities, Spring annotations, and framework classes. Clean Architecture isolates business rules so they're testable, portable, and framework-independent.

```
┌──────────────────────────────────────────────┐
│         Frameworks & Drivers (outer)         │
│    Spring MVC, JPA, Kafka, HTTP clients      │
├──────────────────────────────────────────────┤
│         Interface Adapters                    │
│    Controllers, Repositories, Presenters      │
├──────────────────────────────────────────────┤
│         Application Business Rules            │
│    Use Cases (Application Services)           │
├──────────────────────────────────────────────┤
│         Enterprise Business Rules (core)     │
│    Entities, Value Objects, Domain Services   │
└──────────────────────────────────────────────┘

Dependency Rule: Dependencies point INWARD only
Inner layers NEVER know about outer layers
```

**Key characteristics:**
- Domain entities have NO framework annotations
- Use cases define application behavior
- Ports (interfaces) define what the application needs
- Adapters implement ports using specific technologies

---

## 3. Hexagonal Architecture

**Definition:**
Hexagonal Architecture (Ports & Adapters, by Alistair Cockburn) structures applications around the core domain, with ports (interfaces) defining how the domain interacts with the outside world, and adapters implementing those ports for specific technologies.

**Why it exists:**
The core business logic should not care whether data comes from REST, gRPC, or CLI, or whether it's stored in PostgreSQL, MongoDB, or a file. Hexagonal architecture makes these interchangeable.

```
              ┌─────────────┐
    REST ────→│             │────→ PostgreSQL
   Adapter    │   DOMAIN    │     Adapter (out)
   (in)       │   (Core)    │
              │             │
   gRPC ────→│  Ports &    │────→ Kafka
   Adapter    │  Use Cases  │     Adapter (out)
   (in)       │             │
              └─────────────┘
```

**Backend example:**
```java
// PORT (interface — defined in domain layer)
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(OrderId id);
}

// ADAPTER (implementation — in infrastructure layer)
@Repository
public class JpaOrderRepositoryAdapter implements OrderRepository {
    private final SpringDataOrderRepo springRepo;
    public Order save(Order order) { return toDomain(springRepo.save(toJpa(order))); }
    public Optional<Order> findById(OrderId id) { return springRepo.findById(id.value()).map(this::toDomain); }
}

// USE CASE (application layer — framework-free)
public class PlaceOrderUseCase {
    private final OrderRepository orderRepo; // port
    private final PaymentGateway paymentGateway; // port

    public OrderId execute(PlaceOrderCommand cmd) {
        Order order = Order.create(cmd.userId(), cmd.items());
        paymentGateway.charge(order.total());
        return orderRepo.save(order).id();
    }
}
```

---

## 4. Domain-Driven Design (DDD) Basics

**Definition:**
DDD is an approach to software development that centers the design around the business domain. It emphasizes modeling the problem space using a ubiquitous language shared between developers and domain experts.

### Key DDD Concepts

| Concept | Definition | Example |
|---------|-----------|---------|
| Entity | Object with unique identity | `Order`, `User` (identified by ID) |
| Value Object | Immutable, no identity, defined by attributes | `Money`, `Address`, `EmailAddress` |
| Aggregate | Cluster of entities with a root entity | `Order` (root) + `OrderItem` (child) |
| Aggregate Root | Single entry point to the aggregate | `Order` — all access goes through it |
| Repository | Persistence abstraction for aggregates | `OrderRepository` |
| Domain Service | Business logic that doesn't belong to an entity | `PricingService`, `TransferService` |
| Domain Event | Something meaningful that happened | `OrderPlacedEvent` |
| Bounded Context | Clear boundary around a domain model | Order context vs Inventory context |

```java
// Value Object
public record Money(BigDecimal amount, Currency currency) {
    public Money add(Money other) {
        if (!this.currency.equals(other.currency))
            throw new CurrencyMismatchException();
        return new Money(this.amount.add(other.amount), this.currency);
    }
}

// Aggregate Root
public class Order {
    private OrderId id;
    private UserId userId;
    private List<OrderItem> items = new ArrayList<>(); // aggregate boundary
    private OrderStatus status;

    public void addItem(Product product, int quantity) {
        if (status != OrderStatus.DRAFT)
            throw new OrderAlreadyPlacedException();
        items.add(new OrderItem(product.id(), quantity, product.price()));
    }

    public Money total() {
        return items.stream().map(OrderItem::subtotal).reduce(Money.ZERO, Money::add);
    }
}
```

---

## 5. CQRS

**Definition:**
CQRS (Command Query Responsibility Segregation) separates the write model (commands — create, update, delete) from the read model (queries — retrieval). Each side can have its own data store, schema, and optimization strategy.

**Why it exists:**
Read and write workloads have different characteristics. Writes need normalization, validation, and consistency. Reads need denormalization, speed, and flexibility. CQRS lets you optimize each independently.

```
    Command Side                    Query Side
┌───────────────┐              ┌───────────────┐
│ CreateOrder   │              │ GetOrderList  │
│ UpdateOrder   │              │ GetOrderDetail│
│ CancelOrder   │              │ SearchOrders  │
├───────────────┤              ├───────────────┤
│ Write Model   │──(events)──→│ Read Model    │
│ (Normalized)  │              │ (Denormalized)│
│ PostgreSQL    │              │ Elasticsearch │
└───────────────┘              └───────────────┘
```

**When to use:** Complex domains with different read/write patterns, high-read systems, event sourcing.
**When NOT to use:** Simple CRUD apps — adds unnecessary complexity.

---

## 6. Event Sourcing

**Definition:**
Instead of storing the current state, event sourcing stores every state change as an immutable event. The current state is reconstructed by replaying all events.

```
Traditional:  Account { balance: 850 }
Event Sourced: [
    AccountCreated(balance=1000),
    MoneyWithdrawn(amount=200),
    MoneyDeposited(amount=50)
]  → replay → balance = 1000 - 200 + 50 = 850
```

**When to use:** Audit trails (finance, banking), temporal queries, CQRS systems.
**When NOT to use:** Simple CRUD, small applications.

---

## 7. System Design Methodology

### Step-by-Step Approach (for Interviews)

1. **Clarify Requirements** (~5 min)
   - Functional: What features? What API endpoints?
   - Non-functional: Scale (users, QPS), latency, availability, consistency

2. **Estimate Scale** (~2 min)
   - DAU, QPS, storage, bandwidth
   - Example: 10M users, 1000 QPS reads, 100 QPS writes

3. **High-Level Design** (~10 min)
   - API design, major components, data flow
   - Draw: Client → LB → API Gateway → Services → DB/Cache

4. **Deep Dive** (~15 min)
   - Database schema, caching strategy, scaling plan
   - Trade-offs, bottleneck identification

5. **Address Edge Cases** (~5 min)
   - Failure handling, rate limiting, monitoring

---

## 8. Common System Design Scenarios

### URL Shortener
- **Write:** Generate short code → store mapping in DB → return short URL
- **Read:** Lookup short code → redirect to original URL
- **Key decisions:** Base62 encoding, cache hot URLs (Redis), handle collisions

### Rate Limiter
- **Algorithms:** Token bucket, sliding window, fixed window
- **Store:** Redis (atomic increment + TTL)
- **Apply at:** API Gateway level or per-service

### Notification System
- **Components:** API → Queue (Kafka) → Notification workers → SMS/Email/Push providers
- **Key decisions:** Priority queues, retry with backoff, delivery guarantees

---

## 9. Interview Notes

1. **Explain layered architecture.** — Controller → Service → Repository. Separation of concerns. Standard in Spring Boot.
2. **What is Hexagonal Architecture?** — Ports & Adapters. Domain core is isolated. External concerns connect via interfaces.
3. **What is DDD? Key concepts?** — Design centered on business domain. Entities, Value Objects, Aggregates, Bounded Contexts.
4. **When would you use CQRS?** — When read and write models differ significantly. High-read systems. Event sourcing.
5. **How do you approach a system design question?** — Requirements → Scale → High-level design → Deep dive → Edge cases.

---

## 10. Summary

| Pattern | When to Use | Complexity |
|---------|------------|------------|
| Layered | Most CRUD apps | Low |
| Clean | Complex domains, framework independence | Medium |
| Hexagonal | Swappable infrastructure, multiple interfaces | Medium |
| DDD | Complex business domains | High |
| CQRS | Different read/write patterns | High |
| Event Sourcing | Audit trails, temporal queries | Very High |

---

## 11. References

- [Robert C. Martin — Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Alistair Cockburn — Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Martin Fowler — CQRS](https://martinfowler.com/bliki/CQRS.html)
- [Martin Fowler — Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Eric Evans — Domain-Driven Design](https://www.domainlanguage.com/)
- [System Design Primer (GitHub)](https://github.com/donnemartin/system-design-primer)

---

> **Previous:** [← 07 — Database Design & Persistence](./07_Database_Design_Persistence.md)
> **Next:** [09 — Microservices & Distributed Backend →](./09_Microservices_Distributed_Backend.md)
