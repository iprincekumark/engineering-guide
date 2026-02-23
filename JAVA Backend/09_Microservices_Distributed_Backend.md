# 09 — Microservices & Distributed Backend

> **Goal:** Understand microservices architecture, service communication, resilience patterns, and distributed systems principles.

---

## Table of Contents

1. [Monolith vs Microservices](#1-monolith-vs-microservices)
2. [API Gateway](#2-api-gateway)
3. [Service Discovery](#3-service-discovery)
4. [Load Balancing](#4-load-balancing)
5. [Circuit Breaker Pattern](#5-circuit-breaker-pattern)
6. [Inter-Service Communication](#6-inter-service-communication)
7. [Distributed Transactions (Saga Pattern)](#7-distributed-transactions-saga-pattern)
8. [CAP Theorem](#8-cap-theorem)
9. [Consistency Models](#9-consistency-models)
10. [Distributed Tracing](#10-distributed-tracing)
11. [Interview Notes](#11-interview-notes)
12. [Summary](#12-summary)
13. [References](#13-references)

---

## 1. Monolith vs Microservices

**Definition:**
- **Monolith:** Single deployable unit containing all application logic (user service, order service, payment — all in one JAR).
- **Microservices:** Application split into small, independently deployable services, each owning a specific business domain.

**Comparison:**

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| Deployment | Single unit | Independent per service |
| Scaling | Scale everything together | Scale specific services |
| Data | Shared database | Database per service |
| Team | One team, one codebase | Independent teams |
| Complexity | Low initially, grows fast | High initially, manageable at scale |
| Testing | Easier (all in one) | Harder (distributed) |
| Debugging | Stack traces | Distributed tracing needed |

**When to start with a monolith:**
- New project, small team, unclear domain boundaries
- Premature microservices is a top architecture mistake

**When to migrate to microservices:**
- Team is large (>15 developers)
- Independent deployment is needed
- Different scaling requirements per component
- Domain boundaries are well understood

---

## 2. API Gateway

**Definition:**
An API Gateway is the single entry point for all client requests in a microservices architecture. It routes requests to the appropriate service, handles cross-cutting concerns (auth, rate limiting, logging), and can aggregate responses.

**Why it exists:**
Without a gateway, clients must know about every microservice's address. The gateway abstracts this, providing a unified API to clients.

**Real-life analogy:**
A hotel front desk. Guests (clients) don't go directly to housekeeping, kitchen, or maintenance. They talk to the front desk (gateway), which routes their requests to the right department.

```
                            ┌──────────────┐
Client ──→ API Gateway ──→  │ User Service  │
           (Spring Cloud    ├──────────────┤
            Gateway)  ──→   │ Order Service │
                      ──→   ├──────────────┤
                            │Payment Service│
                            └──────────────┘
```

**Spring Cloud Gateway example:**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/v1/users/**
          filters:
            - StripPrefix=0
            - name: CircuitBreaker
              args:
                name: userServiceCB
                fallbackUri: forward:/fallback/users

        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/v1/orders/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

**Gateway responsibilities:**
- Request routing to appropriate services
- Authentication / authorization
- Rate limiting
- Load balancing
- Request/response transformation
- Circuit breaking
- Logging and monitoring

---

## 3. Service Discovery

**Definition:**
Service discovery is the mechanism by which microservices find each other's network locations (IP, port) dynamically, without hardcoding addresses. Services register themselves and discover others through a registry.

**Why it exists:**
In dynamic environments (Kubernetes, cloud), service instances spin up/down constantly with different IPs. Hardcoding addresses is impossible.

**Real-life analogy:**
A company phone directory. When you need to reach HR, you look up the directory (registry) for the current phone number, rather than memorizing it.

**Spring Cloud + Eureka:**
```yaml
# Eureka Server (registry)
server:
  port: 8761
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false

# Eureka Client (in each microservice)
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

```java
// Using service name instead of hardcoded URL
@FeignClient(name = "user-service") // Resolved via Eureka
public interface UserClient {
    @GetMapping("/api/v1/users/{id}")
    UserDto getUserById(@PathVariable Long id);
}
```

**Alternatives to Eureka:** Consul, Kubernetes DNS (built-in), Zookeeper.

---

## 4. Load Balancing

**Definition:**
Load balancing distributes incoming requests across multiple instances of a service to prevent overload, improve throughput, and provide fault tolerance.

| Type | Where | How |
|------|-------|-----|
| Server-side (L7) | At load balancer (Nginx, AWS ALB) | Routes via rule (round-robin, least connections) |
| Client-side | In the calling service | Spring Cloud LoadBalancer picks an instance |

**Spring Cloud LoadBalancer:**
```java
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate(); // Automatically resolves service names
}

// Usage
restTemplate.getForObject("http://user-service/api/v1/users/42", UserDto.class);
// "user-service" is resolved to an actual IP via service discovery + load balancing
```

---

## 5. Circuit Breaker Pattern

**Definition:**
A circuit breaker prevents cascading failures by monitoring calls to a downstream service. When failures exceed a threshold, it "opens" the circuit — failing immediately instead of waiting for timeouts, giving the downstream service time to recover.

**Why it exists:**
If Service A calls Service B, and B is down, each request waits for a timeout (30s). Under load, A's threads are exhausted → A goes down → cascade failure.

**States:**
```
CLOSED ──(failures > threshold)──→ OPEN ──(wait duration)──→ HALF-OPEN
  ↑                                                              │
  └──────────(success in half-open)──────────────────────────────┘
                                 (failures in half-open → OPEN)
```

**Resilience4j (Spring Boot's default):**
```java
@Service
public class OrderService {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<PaymentResult> processPayment(PaymentRequest req) {
        return CompletableFuture.supplyAsync(() -> paymentClient.charge(req));
    }

    public CompletableFuture<PaymentResult> paymentFallback(PaymentRequest req, Throwable t) {
        log.error("Payment service unavailable: {}", t.getMessage());
        return CompletableFuture.completedFuture(PaymentResult.pending("Service temporarily unavailable"));
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 30s
        permitted-number-of-calls-in-half-open-state: 3
  retry:
    instances:
      paymentService:
        max-attempts: 3
        wait-duration: 1s
  timelimiter:
    instances:
      paymentService:
        timeout-duration: 5s
```

---

## 6. Inter-Service Communication

### 6.1 Synchronous (Request-Response)

| Technology | Use Case |
|-----------|----------|
| REST (HTTP) | Simple API calls, CRUD |
| gRPC | High-performance, binary, streaming |
| OpenFeign | Declarative REST client (Spring) |

```java
// Feign Client
@FeignClient(name = "inventory-service", fallback = InventoryFallback.class)
public interface InventoryClient {
    @GetMapping("/api/v1/inventory/{productId}")
    InventoryDto checkStock(@PathVariable Long productId);
}
```

### 6.2 Asynchronous (Event-Driven)

| Technology | Use Case |
|-----------|----------|
| Kafka | High-throughput event streaming |
| RabbitMQ | Message queuing, routing |
| Spring Events | In-process events |

**Choose sync vs async:**
- **Sync:** When you need the response immediately (check inventory before order)
- **Async:** When you don't need immediate response (send notification after order)

---

## 7. Distributed Transactions (Saga Pattern)

**Definition:**
The Saga pattern manages distributed transactions across multiple microservices by breaking them into a sequence of local transactions, each with a compensating action (undo) in case of failure.

**Why it exists:**
Traditional ACID transactions don't work across microservices (each has its own database). Sagas provide eventual consistency through coordinated local transactions.

**Types:**

| Type | How | When |
|------|-----|------|
| Choreography | Each service publishes events, next service reacts | Simple flows, few services |
| Orchestration | Central orchestrator coordinates the steps | Complex flows, many services |

**Choreography example (Order → Payment → Inventory):**
```
OrderService                     PaymentService                InventoryService
     │                                │                              │
     │── OrderCreated event ─────────→│                              │
     │                                │── PaymentCharged event ─────→│
     │                                │                              │── InventoryReserved event
     │                                │                              │
     │  (if payment fails)            │                              │
     │←── PaymentFailed event ────────│                              │
     │── CancelOrder (compensate)     │                              │
```

---

## 8. CAP Theorem

**Definition:**
In a distributed system, you can only guarantee **two out of three** properties simultaneously:
- **C**onsistency — Every read receives the most recent write
- **A**vailability — Every request receives a response
- **P**artition tolerance — System continues despite network partitions

**Why it matters:**
Network partitions WILL happen. So the real choice is: **CP** (consistent but may be unavailable) or **AP** (available but may serve stale data).

| Choice | Behavior | Example |
|--------|----------|---------|
| CP | Rejects requests during partitions | Banking systems, ZooKeeper |
| AP | Returns potentially stale data | DNS, Cassandra, shopping carts |

---

## 9. Consistency Models

| Model | Description | Example |
|-------|-------------|---------|
| Strong | Read always returns latest write | Single DB, synchronous replication |
| Eventual | Will converge eventually | DNS propagation, async replication |
| Causal | Maintains cause-effect order | Chat messages in correct order |
| Read-your-writes | You see your own writes immediately | User profile updates |

---

## 10. Distributed Tracing

**Definition:**
Distributed tracing tracks a request as it flows across multiple microservices, using trace IDs and span IDs. It's essential for debugging in microservices.

```
Request → API Gateway → User Service → Order Service → Payment Service
 trace-id: abc-123
   span-1: gateway (5ms)
   span-2: user-service (15ms)
   span-3: order-service (25ms)
   span-4: payment-service (100ms)
```

**Tools:** Zipkin, Jaeger, OpenTelemetry (via Micrometer Tracing in Spring Boot 3+).

---

## 11. Interview Notes

1. **When to use microservices vs monolith?** — Start monolith. Migrate when team grows, independent deployment needed, domains are clear.
2. **What is a circuit breaker?** — Prevents cascading failures. CLOSED → OPEN → HALF-OPEN states. Resilience4j in Spring Boot.
3. **Explain the Saga pattern.** — Distributed transactions via local transactions + compensating actions. Choreography (events) or Orchestration (coordinator).
4. **What is CAP theorem?** — Distributed systems can guarantee only 2 of 3 (Consistency, Availability, Partition tolerance). Partitions are inevitable → choose CP or AP.
5. **Sync vs Async communication?** — Sync (REST/gRPC) for immediate responses. Async (Kafka/RabbitMQ) for decoupled, non-blocking operations.
6. **How do you debug a request across microservices?** — Distributed tracing with trace IDs (OpenTelemetry, Zipkin).

---

## 12. Summary

| Topic | Key Takeaway |
|-------|-------------|
| Monolith vs Micro | Start monolith, migrate when needed |
| API Gateway | Single entry point, cross-cutting concerns |
| Service Discovery | Dynamic service registration (Eureka, K8s DNS) |
| Circuit Breaker | Prevent cascading failures (Resilience4j) |
| Communication | Sync for queries, Async for events |
| Saga | Distributed transactions via local txns + compensating actions |
| CAP | Choose CP or AP — partitions are inevitable |

---

## 13. References

- [Microservices.io — Patterns](https://microservices.io/patterns/)
- [Martin Fowler — Microservices](https://martinfowler.com/articles/microservices.html)
- [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)
- [Resilience4j Documentation](https://resilience4j.readme.io/)
- [Chris Richardson — Saga Pattern](https://microservices.io/patterns/data/saga.html)
- [CAP Theorem Explained](https://www.ibm.com/topics/cap-theorem)

---

> **Previous:** [← 08 — Backend Architecture & System Design](./08_Backend_Architecture_System_Design.md)
> **Next:** [10 — Caching, Messaging & Async →](./10_Caching_Messaging_Async.md)
