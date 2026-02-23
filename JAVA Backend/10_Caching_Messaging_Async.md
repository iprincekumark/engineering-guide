# 10 — Caching, Messaging & Async

> **Goal:** Master caching strategies (Redis, Caffeine), message brokers (Kafka, RabbitMQ), event-driven architecture, and async processing in Java backends.

---

## Table of Contents

1. [Caching Fundamentals](#1-caching-fundamentals)
2. [Cache Patterns](#2-cache-patterns)
3. [Spring Cache Abstraction](#3-spring-cache-abstraction)
4. [Redis](#4-redis)
5. [Caffeine (In-Memory Cache)](#5-caffeine-in-memory-cache)
6. [Messaging Fundamentals](#6-messaging-fundamentals)
7. [Apache Kafka](#7-apache-kafka)
8. [RabbitMQ](#8-rabbitmq)
9. [Event-Driven Architecture](#9-event-driven-architecture)
10. [Async Processing Patterns](#10-async-processing-patterns)
11. [Interview Notes](#11-interview-notes)
12. [Summary](#12-summary)
13. [References](#13-references)

---

## 1. Caching Fundamentals

**Definition:**
Caching is the practice of storing frequently accessed data in a fast-access layer (memory) to reduce expensive operations (database queries, API calls). Cached data can be served in microseconds vs milliseconds for a DB hit.

**Why it exists:**
Databases are the biggest bottleneck in most backends. If the same product page is viewed 10,000 times/minute, querying the DB each time is wasteful. Cache the result → serve from memory → 100x faster.

**Real-life analogy:**
Your phone's recent contacts. Instead of scrolling through your entire phonebook (database), frequently called numbers appear at the top (cache).

**Key metrics:**
- **Cache hit ratio:** Percentage of requests served from cache (target: >90%)
- **Cache miss:** Data not in cache → fallback to DB → store in cache
- **TTL (Time-To-Live):** How long data stays in cache before expiring

---

## 2. Cache Patterns

| Pattern | How It Works | When to Use |
|---------|-------------|-------------|
| **Cache-Aside** | App reads from cache first; on miss, reads from DB and populates cache | Most common, manual control |
| **Read-Through** | Cache handles DB reads automatically | When cache library supports it |
| **Write-Through** | Writes go to cache AND DB simultaneously | Strong consistency needed |
| **Write-Behind** | Writes go to cache, DB updated asynchronously | High write throughput |
| **Write-Around** | Writes go to DB only, cache filled on read | Write-heavy, read-infrequent data |

**Cache-Aside example:**
```java
public UserDto getUser(Long id) {
    // 1. Check cache first
    String cacheKey = "user:" + id;
    UserDto cached = redisTemplate.opsForValue().get(cacheKey);
    if (cached != null) return cached; // Cache HIT

    // 2. Cache MISS → query DB
    UserDto user = userRepository.findById(id)
        .map(this::toDto)
        .orElseThrow(() -> new ResourceNotFoundException("User", "id", id));

    // 3. Populate cache
    redisTemplate.opsForValue().set(cacheKey, user, Duration.ofMinutes(30));
    return user;
}
```

---

## 3. Spring Cache Abstraction

**Definition:**
Spring Cache provides a declarative caching layer via annotations (`@Cacheable`, `@CachePut`, `@CacheEvict`), abstracting the cache provider (Redis, Caffeine, EhCache).

```java
@Configuration
@EnableCaching
public class CacheConfig { }

@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public ProductDto findById(Long id) {
        // Only called on cache miss — result is cached automatically
        return productRepository.findById(id).map(this::toDto).orElseThrow();
    }

    @CachePut(value = "products", key = "#result.id")
    public ProductDto update(Long id, UpdateProductRequest req) {
        // Always executes — updates the cache with the return value
        Product product = productRepository.findById(id).orElseThrow();
        product.setName(req.name());
        return toDto(productRepository.save(product));
    }

    @CacheEvict(value = "products", key = "#id")
    public void delete(Long id) {
        // Removes from cache after execution
        productRepository.deleteById(id);
    }

    @CacheEvict(value = "products", allEntries = true)
    @Scheduled(fixedRate = 3600000) // Every hour
    public void clearCache() {
        log.info("Product cache cleared");
    }
}
```

---

## 4. Redis

**Definition:**
Redis is an in-memory data structure store used as a cache, message broker, and data store. It supports strings, hashes, lists, sets, sorted sets, and more — with sub-millisecond latency.

**Why it exists:**
Redis provides shared, persistent caching across multiple application instances. Unlike in-memory caches (Caffeine), all instances see the same cached data.

**Backend use cases:**
- Caching (most common)
- Session storage
- Rate limiting
- Distributed locks
- Leaderboards (sorted sets)
- Pub/Sub messaging

**Spring Boot + Redis:**
```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      timeout: 2000ms
  cache:
    type: redis
    redis:
      time-to-live: 30m
      cache-null-values: false
```

```java
// Using RedisTemplate for complex operations
@Service
public class RateLimiterService {
    private final StringRedisTemplate redisTemplate;

    public boolean isAllowed(String clientIp, int maxRequests, Duration window) {
        String key = "rate:" + clientIp;
        Long count = redisTemplate.opsForValue().increment(key);
        if (count == 1) {
            redisTemplate.expire(key, window);
        }
        return count <= maxRequests;
    }
}

// Distributed Lock
@Service
public class DistributedLockService {
    private final StringRedisTemplate redisTemplate;

    public boolean acquireLock(String lockKey, Duration timeout) {
        return Boolean.TRUE.equals(
            redisTemplate.opsForValue().setIfAbsent(lockKey, "locked", timeout)
        );
    }

    public void releaseLock(String lockKey) {
        redisTemplate.delete(lockKey);
    }
}
```

---

## 5. Caffeine (In-Memory Cache)

**Definition:**
Caffeine is a high-performance, near-optimal in-memory caching library for Java. It's the default in-memory cache provider for Spring Boot.

**When to use Caffeine vs Redis:**
- **Caffeine:** Single-instance cache, no network overhead, ultra-fast, but data is NOT shared across instances.
- **Redis:** Distributed cache, shared across instances, network overhead, but consistent across all servers.

```java
@Configuration
@EnableCaching
public class CaffeineCacheConfig {
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager();
        manager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(10_000)
            .expireAfterWrite(Duration.ofMinutes(10))
            .recordStats());
        return manager;
    }
}
```

---

## 6. Messaging Fundamentals

**Definition:**
Messaging is asynchronous communication between services through a message broker. The producer sends messages to a broker (Kafka, RabbitMQ), and consumers read them independently — decoupling sender and receiver.

**Why it exists:**
Synchronous calls between services create tight coupling and cascading failures. Messaging decouples services, enables event-driven processing, and handles traffic spikes through buffering.

**Real-life analogy:**
Email vs phone call. A phone call (sync) requires both parties to be available simultaneously. Email (async) lets you send a message and the receiver processes it when ready.

---

## 7. Apache Kafka

**Definition:**
Kafka is a distributed event streaming platform for high-throughput, fault-tolerant message processing. It stores messages in ordered, immutable logs (topics) that can be consumed by multiple consumers independently.

**Key concepts:**
- **Topic:** Named log of messages (like a database table)
- **Partition:** Topics are split for parallelism
- **Producer:** Publishes messages to topics
- **Consumer:** Reads messages from topics
- **Consumer Group:** Multiple consumers sharing work on a topic
- **Offset:** Position of a consumer in the log

```java
// Kafka Producer
@Service
@RequiredArgsConstructor
public class OrderEventProducer {
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public void publishOrderCreated(Order order) {
        OrderEvent event = new OrderEvent("ORDER_CREATED", order.getId(), order.getTotal());
        kafkaTemplate.send("order-events", order.getId().toString(), event);
    }
}

// Kafka Consumer
@Component
@Slf4j
public class OrderEventConsumer {

    @KafkaListener(topics = "order-events", groupId = "notification-service")
    public void handleOrderEvent(OrderEvent event) {
        log.info("Received order event: {}", event);
        if ("ORDER_CREATED".equals(event.type())) {
            notificationService.sendOrderConfirmation(event.orderId());
        }
    }
}
```

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      group-id: notification-service
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"
```

---

## 8. RabbitMQ

**Definition:**
RabbitMQ is a message broker that implements AMQP (Advanced Message Queuing Protocol). Unlike Kafka's log-based approach, RabbitMQ uses exchanges and queues with sophisticated routing capabilities.

**Kafka vs RabbitMQ:**

| Feature | Kafka | RabbitMQ |
|---------|-------|---------|
| Model | Distributed log (stream) | Message queue |
| Throughput | Very high (millions/sec) | High (tens of thousands/sec) |
| Message retention | Configured retention period | Until consumed |
| Replay | Yes (consumers can re-read) | No (message deleted after ACK) |
| Use case | Event streaming, analytics | Task queues, routing |
| Ordering | Per partition | Per queue |

```java
// RabbitMQ Producer
@Service
public class EmailTaskProducer {
    private final RabbitTemplate rabbitTemplate;

    public void sendEmailTask(EmailTask task) {
        rabbitTemplate.convertAndSend("email-exchange", "email.send", task);
    }
}

// RabbitMQ Consumer
@Component
public class EmailTaskConsumer {
    @RabbitListener(queues = "email-queue")
    public void handleEmailTask(EmailTask task) {
        emailService.send(task.to(), task.subject(), task.body());
    }
}
```

---

## 9. Event-Driven Architecture

**Definition:**
An architectural pattern where services communicate through events — a producer publishes an event when something happens, and interested consumers react to it. Services are decoupled; they don't call each other directly.

```
Order Service ──(OrderPlaced)──→ Kafka ──→ Notification Service
                                      ──→ Inventory Service
                                      ──→ Analytics Service
```

**Key principles:**
- **Loose coupling:** Services don't know about each other
- **Eventual consistency:** State converges over time
- **Event replay:** Can rebuild state from events
- **Scalability:** Add new consumers without modifying producers

---

## 10. Async Processing Patterns

### 10.1 Fire-and-Forget
```java
@Async
public void sendWelcomeEmail(User user) {
    emailClient.send(user.getEmail(), "Welcome!", body);
}
```

### 10.2 Request-Reply (Async)
```java
@Async
public CompletableFuture<Report> generateReport(ReportRequest req) {
    Report report = reportEngine.generate(req);
    return CompletableFuture.completedFuture(report);
}
```

### 10.3 Task Queue Pattern
```
Producer → Queue (RabbitMQ) → Worker 1
                             → Worker 2
                             → Worker 3
```

Multiple workers process tasks from a shared queue — scales horizontally.

---

## 11. Interview Notes

1. **When to use Redis vs Caffeine?** — Redis: distributed (multiple instances). Caffeine: single JVM (faster, no network).
2. **Explain cache-aside pattern.** — Check cache → miss → query DB → populate cache. Most common pattern.
3. **What is cache stampede and how to prevent it?** — Many concurrent requests miss cache simultaneously → all hit DB. Prevent with locking or early refresh.
4. **Kafka vs RabbitMQ?** — Kafka: event streaming, replay, high throughput. RabbitMQ: task queues, routing, message acknowledgment.
5. **What is eventual consistency?** — Data will be consistent eventually, but may be stale temporarily. Used in event-driven systems.
6. **How do you handle message ordering in Kafka?** — Ordering is guaranteed within a partition. Use a consistent partition key (e.g., orderId).

---

## 12. Summary

| Topic | Key Takeaway |
|-------|-------------|
| Caching | Reduces DB load by 90%+; cache-aside is most common |
| Redis | Distributed cache, rate limiting, locks, sessions |
| Caffeine | Ultra-fast in-memory cache for single JVM |
| Kafka | Event streaming, high throughput, message replay |
| RabbitMQ | Task queues, routing, message acknowledgment |
| Event-Driven | Decoupled services communicating via events |

---

## 13. References

- [Redis Documentation](https://redis.io/docs/)
- [Spring Data Redis](https://docs.spring.io/spring-data/redis/reference/)
- [Spring Cache Documentation](https://docs.spring.io/spring-framework/reference/integration/cache.html)
- [Caffeine GitHub](https://github.com/ben-manes/caffeine)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Spring for Apache Kafka](https://docs.spring.io/spring-kafka/reference/)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials)
- [Martin Fowler — Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)

---

> **Previous:** [← 09 — Microservices & Distributed Backend](./09_Microservices_Distributed_Backend.md)
> **Next:** [11 — Observability & Performance Tuning →](./11_Observability_Performance_Tuning.md)
