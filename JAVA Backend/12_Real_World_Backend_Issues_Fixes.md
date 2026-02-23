# 12 — Real-World Backend Issues & Fixes

> **Goal:** Learn to identify, debug, and fix the most common production issues in Java backend systems. Each scenario follows: Problem → Root Cause → Debugging → Fix → Prevention → Team Lesson.

---

## Table of Contents

1. [Memory Leaks](#1-memory-leaks)
2. [Slow Queries](#2-slow-queries)
3. [Deadlocks](#3-deadlocks)
4. [Thread Exhaustion](#4-thread-exhaustion)
5. [High CPU Usage](#5-high-cpu-usage)
6. [GC Pauses](#6-gc-pauses)
7. [API Latency Spikes](#7-api-latency-spikes)
8. [Transaction Failures](#8-transaction-failures)
9. [Microservice Cascade Failures](#9-microservice-cascade-failures)
10. [Cache Stampede](#10-cache-stampede)
11. [Security Misconfigurations](#11-security-misconfigurations)
12. [Connection Pool Exhaustion](#12-connection-pool-exhaustion)
13. [Summary](#13-summary)
14. [References](#14-references)

---

## 1. Memory Leaks

**Problem:**
Application memory usage grows steadily over days. Eventually throws `OutOfMemoryError: Java heap space`. On restart, it works fine initially but degrades again.

**Root Cause:**
Objects are created but never garbage-collected because they're still referenced — typically via static collections, event listeners not deregistered, or objects stored in a `ThreadLocal` without cleanup.

**Debugging Steps:**
1. Enable heap dumps: `-XX:+HeapDumpOnOutOfMemoryError`
2. Trigger heap dump manually: `jmap -dump:format=b,file=heap.hprof <pid>`
3. Analyze with Eclipse MAT or VisualVM
4. Look for dominator tree — largest retained objects
5. Check for `HashMap`, `ArrayList`, or custom caches growing unbounded

**Fix:**
```java
// ❌ Problem — unbounded cache
private static final Map<String, Object> cache = new HashMap<>();
public void process(String key, Object value) {
    cache.put(key, value); // Never evicts — grows forever
}

// ✅ Fix — bounded cache with eviction
private final Cache<String, Object> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(Duration.ofMinutes(30))
    .build();
```

**Prevention:**
- Use bounded caches (Caffeine, Guava) — never unbounded collections
- Always clear `ThreadLocal` in finally blocks
- Use `-XX:+HeapDumpOnOutOfMemoryError` in all environments
- Monitor heap usage with alerts at 80% threshold

**Team Lesson:**
"Static collections without eviction policies are memory leaks waiting to happen. Every cache must have a size limit and TTL."

---

## 2. Slow Queries

**Problem:**
API response times increase from 100ms to 5-10 seconds. Users complain about slowness. Database CPU is at 90%.

**Root Cause:**
A report query was added without an index. As data grew to millions of rows, the query performs a full table scan — O(n) instead of O(log n).

**Debugging Steps:**
1. Enable slow query log: `log_min_duration_statement = 500` (PostgreSQL)
2. Identify the slow query
3. Run `EXPLAIN ANALYZE` on the query
4. Look for "Seq Scan" on large tables — indicates missing index
5. Check for sequential scans, nested loops with large row estimates

**Fix:**
```sql
-- Before: Full table scan (5 seconds on 10M rows)
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42 AND status = 'ACTIVE';
-- Seq Scan on orders, rows=10000000

-- Fix: Add composite index
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- After: Index scan (2ms)
-- Index Scan using idx_orders_user_status, rows=15
```

**Prevention:**
- Review `EXPLAIN ANALYZE` for all new queries
- Add indexes during development, not after complaints
- Monitor slow query log in production
- Set up alerts for queries >500ms

**Team Lesson:**
"Every new query on a table with >10K rows should be reviewed with EXPLAIN ANALYZE before merging."

---

## 3. Deadlocks

**Problem:**
Intermittent `org.hibernate.exception.LockAcquisitionException`. Some transactions hang and eventually timeout. Occurs under concurrent load.

**Root Cause:**
Two transactions lock resources in opposite order. Transaction A locks row 1 then waits for row 2. Transaction B locks row 2 then waits for row 1.

**Debugging Steps:**
1. Check database deadlock logs: `SHOW ENGINE INNODB STATUS` (MySQL) or `pg_stat_activity` (PostgreSQL)
2. Identify the conflicting transactions and the tables/rows involved
3. Check the order of UPDATE/INSERT statements in the code

**Fix:**
```java
// ❌ Problem — inconsistent lock ordering
// Thread 1: lock account A, then lock account B
// Thread 2: lock account B, then lock account A

// ✅ Fix — consistent lock ordering
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {
    Long firstId = Math.min(fromId, toId);
    Long secondId = Math.max(fromId, toId);

    Account first = accountRepo.findByIdForUpdate(firstId);  // Always lock lower ID first
    Account second = accountRepo.findByIdForUpdate(secondId);
    // ... transfer logic
}
```

**Prevention:**
- Always acquire locks in a consistent global order
- Use `SELECT ... FOR UPDATE` with consistent ordering
- Use optimistic locking (`@Version`) when possible — avoids deadlocks entirely
- Set reasonable lock timeout: `innodb_lock_wait_timeout = 10`

**Team Lesson:**
"Deadlocks are caused by inconsistent lock ordering. Define a global ordering convention (e.g., by ID) and apply it everywhere."

---

## 4. Thread Exhaustion

**Problem:**
Application stops accepting new requests. All 200 Tomcat threads are occupied. Health checks fail. No errors in application logs.

**Root Cause:**
A downstream API (payment service) is slow (30-second timeout). Each request holds a Tomcat thread for 30 seconds. Under moderate load (10 req/sec), all 200 threads are consumed in 20 seconds.

**Debugging Steps:**
1. Take a thread dump: `jstack <pid>` or Actuator `/threaddump`
2. Look for threads in `WAITING` or `TIMED_WAITING` state
3. Identify the common stack trace — all stuck waiting on the same HTTP call
4. Check HikariCP pending requests: `hikaricp_connections_pending`

**Fix:**
```java
// ❌ Without timeout — thread waits indefinitely
restTemplate.getForObject("http://payment-service/charge", PaymentResult.class);

// ✅ With aggressive timeouts
@Bean
public RestTemplate restTemplate() {
    return new RestTemplateBuilder()
        .connectTimeout(Duration.ofSeconds(2))
        .readTimeout(Duration.ofSeconds(5))
        .build();
}

// ✅ Circuit breaker to fail fast
@CircuitBreaker(name = "payment", fallbackMethod = "paymentFallback")
public PaymentResult charge(PaymentRequest req) {
    return restTemplate.postForObject("/charge", req, PaymentResult.class);
}
```

**Prevention:**
- Set aggressive timeouts on ALL external calls (2-5 seconds)
- Use circuit breakers for downstream services
- Monitor thread pool utilization with alerts at 80%
- Consider virtual threads (JDK 21+) for I/O-bound services

---

## 5. High CPU Usage

**Problem:**
CPU usage is consistently at 95%+. Application is slow. No recent deployments.

**Root Cause:**
An infinite loop in a background thread, or inefficient algorithm (O(n²) on a growing dataset), or excessive GC (minor pauses every 50ms).

**Debugging Steps:**
1. Take CPU profile: `async-profiler -d 30 -f output.jfr <pid>`
2. Generate flame graph to visualize CPU hotspots
3. If top stack is GC → check heap size and allocation rate
4. If top stack is application code → identify the hot method

**Fix:**
```java
// ❌ O(n²) — checking for duplicates
for (int i = 0; i < list.size(); i++) {
    for (int j = i + 1; j < list.size(); j++) {
        if (list.get(i).equals(list.get(j))) { /* duplicate */ }
    }
}

// ✅ O(n) — using HashSet
Set<String> seen = new HashSet<>();
for (String item : list) {
    if (!seen.add(item)) { /* duplicate */ }
}
```

**Prevention:**
- Profile before optimizing — measure, don't guess
- Review algorithmic complexity during code review
- Monitor CPU with alerts at 80%
- Use flame graphs regularly to find hotspots

---

## 6. GC Pauses

**Problem:**
P99 latency spikes every 30-60 seconds. During spikes, all requests take 500ms-2s. CPU shows periodic bursts.

**Root Cause:**
Full GC pauses. The old generation fills up → JVM stops all threads to collect garbage → multi-second pause.

**Debugging Steps:**
1. Enable GC logging: `-Xlog:gc*:file=gc.log:time,level`
2. Analyze with GCViewer or GCEasy.io
3. Check for full GC frequency and duration
4. Check if heap is too small or allocation rate is too high

**Fix:**
```bash
# Switch to ZGC for sub-millisecond pauses
java -XX:+UseZGC -Xms4g -Xmx4g -jar app.jar

# Or tune G1GC
java -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -Xms4g -Xmx4g -jar app.jar
```

**Prevention:**
- Set `-Xms` = `-Xmx` (avoids heap resizing)
- Use ZGC for latency-sensitive APIs
- Monitor GC pause metrics in Grafana
- Alert on full GC events or pauses >500ms

---

## 7. API Latency Spikes

**Problem:**
P95 latency increases from 200ms to 2-5 seconds at specific times of day.

**Root Cause:**
A scheduled job runs every hour and loads 100K records into memory, causing GC pressure and saturating the DB connection pool.

**Fix:**
- Move heavy jobs to a separate service/instance
- Use streaming queries instead of loading all records into memory
- Stagger scheduled jobs to avoid peak hours
- Use read replicas for heavy queries

---

## 8. Transaction Failures

**Problem:**
`@Transactional` method silently doesn't roll back on exception. Data is partially committed.

**Root Cause:**
The method catches the exception internally, or the exception is checked (only unchecked exceptions trigger rollback by default), or self-invocation bypasses the proxy.

**Fix:**
```java
// ❌ Swallowing exception — no rollback
@Transactional
public void processOrder(Order order) {
    try {
        orderRepo.save(order);
        paymentService.charge(order); // throws
    } catch (Exception e) {
        log.error("Failed", e); // Transaction commits anyway!
    }
}

// ✅ Let it propagate, or rethrow
@Transactional(rollbackFor = Exception.class) // roll back on ALL exceptions
public void processOrder(Order order) {
    orderRepo.save(order);
    paymentService.charge(order); // RuntimeException → auto rollback
}
```

---

## 9. Microservice Cascade Failures

**Problem:**
One microservice (Payment) slows down. Within minutes, Order service, Cart service, and API Gateway all become unresponsive.

**Root Cause:**
Each service synchronously calls Payment with a 30s timeout. All threads get blocked waiting. Caller services fail, then their callers fail — a cascade.

**Fix:**
- Circuit breakers on all downstream calls
- Aggressive timeouts (2-5 seconds)
- Bulkhead pattern (separate thread pools per downstream)
- Async processing for non-critical operations
- Fallbacks for graceful degradation

---

## 10. Cache Stampede

**Problem:**
Application experiences sudden spike in DB queries. Redis shows cache miss rate at 100% for specific keys. DB CPU/connections spike.

**Root Cause:**
A popular cached item expires, and thousands of concurrent requests all miss the cache simultaneously → all query the DB → DB overwhelmed.

**Fix:**
```java
// Solution 1: Distributed lock — only one thread refreshes
public ProductDto getProduct(Long id) {
    String key = "product:" + id;
    ProductDto cached = redisTemplate.opsForValue().get(key);
    if (cached != null) return cached;

    String lockKey = "lock:product:" + id;
    boolean locked = redisTemplate.opsForValue().setIfAbsent(lockKey, "1", Duration.ofSeconds(10));
    if (locked) {
        try {
            ProductDto product = productRepo.findById(id).map(this::toDto).orElseThrow();
            redisTemplate.opsForValue().set(key, product, Duration.ofMinutes(30));
            return product;
        } finally {
            redisTemplate.delete(lockKey);
        }
    }
    // Other threads wait and retry
    Thread.sleep(100);
    return getProduct(id); // retry
}

// Solution 2: Random TTL jitter — stagger expiration
long ttlSeconds = 1800 + ThreadLocalRandom.current().nextInt(300); // 30-35 min
redisTemplate.opsForValue().set(key, value, Duration.ofSeconds(ttlSeconds));
```

---

## 11. Security Misconfigurations

**Problem:**
Security audit reveals: actuator endpoints exposed publicly, debug logging enabled in production, CORS set to `*`, sensitive data in error responses.

**Fix:**
```yaml
# Lock down actuator
management:
  endpoints:
    web:
      exposure:
        include: health, info

# Disable debug in production
logging:
  level:
    root: INFO

# Restrict CORS
spring:
  web:
    cors:
      allowed-origins: https://myapp.com
```

**Prevention:**
- Security review checklist before every release
- Automated security scanning (OWASP ZAP, Snyk)
- Separate config for dev/staging/production profiles
- Never expose stack traces in production error responses

---

## 12. Connection Pool Exhaustion

**Problem:**
`HikariPool-1 - Connection is not available, request timed out after 30000ms`

**Root Cause:**
Long-running transactions hold connections. Or connection leak — code acquires connection but never returns it.

**Fix:**
- Optimize slow queries (most common cause)
- Enable leak detection: `leak-detection-threshold: 60000`
- Ensure `@Transactional` covers only necessary scope
- Increase pool size if genuinely under-provisioned

---

## 13. Summary

| Issue | Quick Diagnosis | Quick Fix |
|-------|----------------|-----------|
| Memory Leak | Heap dump + MAT | Bounded caches, clear ThreadLocal |
| Slow Queries | EXPLAIN ANALYZE | Add indexes |
| Deadlocks | DB deadlock log | Consistent lock ordering |
| Thread Exhaustion | Thread dump | Timeouts + circuit breakers |
| High CPU | Flame graph | Optimize algorithm |
| GC Pauses | GC log analysis | ZGC or tune heap |
| Cache Stampede | Redis miss rate | Lock + TTL jitter |
| Connection Pool | HikariCP metrics | Fix slow queries, detect leaks |

---

## 14. References

- [Baeldung — OutOfMemoryError](https://www.baeldung.com/java-memory-leaks)
- [Eclipse MAT](https://eclipse.dev/mat/)
- [async-profiler](https://github.com/async-profiler/async-profiler)
- [GCEasy — GC Log Analyzer](https://gceasy.io/)
- [Resilience4j — Circuit Breaker](https://resilience4j.readme.io/)
- [HikariCP — Connection Pool](https://github.com/brettwooldridge/HikariCP)

---

> **Previous:** [← 11 — Observability & Performance Tuning](./11_Observability_Performance_Tuning.md)
> **Next:** [13 — Backend Best Practices →](./13_Backend_Best_Practices.md)
