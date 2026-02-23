# 14 — Java Backend Interview Prep

> **Goal:** Comprehensive interview preparation covering Core Java, Spring Boot, databases, system design, scenario-based questions, and how to explain your projects.

---

## Table of Contents

1. [Core Java Questions](#1-core-java-questions)
2. [Spring Boot Questions](#2-spring-boot-questions)
3. [Database Questions](#3-database-questions)
4. [System Design Questions](#4-system-design-questions)
5. [Scenario & Debugging Questions](#5-scenario--debugging-questions)
6. [Project Explanation Guide](#6-project-explanation-guide)
7. [Common Mistakes](#7-common-mistakes)
8. [Expected Depth by Level](#8-expected-depth-by-level)
9. [Mock Interview Practice](#9-mock-interview-practice)
10. [References](#10-references)

---

## 1. Core Java Questions

### Fundamentals

**Q: What is the difference between `==` and `.equals()`?**
`==` compares references (memory addresses). `.equals()` compares logical content. For Strings, `==` may work due to the String Pool but should never be relied upon — always use `.equals()`.

**Q: Why must you override `hashCode()` when you override `equals()`?**
If two objects are `.equals()`, they MUST have the same `hashCode()`. Otherwise, HashMap/HashSet behavior breaks — equal objects end up in different buckets.

**Q: What is the difference between `final`, `finally`, and `finalize()`?**
- `final`: Keyword — constant variable, uninheritable class, unoverridable method
- `finally`: Block — always executes after try/catch (cleanup)
- `finalize()`: Deprecated method called by GC before object is collected (never rely on this)

### Collections

**Q: How does HashMap work internally?**
`hashCode()` → bucket index via `(n-1) & hash`. Collision → linked list (JDK 7) or linked list → red-black tree when ≥ 8 entries (JDK 8+). Load factor 0.75 → rehash. Not thread-safe.

**Q: HashMap vs ConcurrentHashMap?**
HashMap is not thread-safe. ConcurrentHashMap uses CAS + synchronized on individual bins (JDK 8+). No null keys/values in CHM. Use CHM for shared state across threads.

**Q: ArrayList vs LinkedList?**
ArrayList: Array-backed, O(1) random access, cache-friendly — use this 95% of the time. LinkedList: Node-based, O(1) add/remove at ends — only for Deque use cases.

### Concurrency

**Q: What is the difference between `synchronized` and `ReentrantLock`?**
`synchronized`: implicit, block-scoped, no timeout. `ReentrantLock`: explicit lock/unlock, supports tryLock with timeout, fairness policy, interruptible. Use ReentrantLock for advanced scenarios.

**Q: What are virtual threads?**
JDK 21+ lightweight threads managed by JVM (~KB stack vs ~MB for platform threads). Enable millions of concurrent I/O-bound tasks with simple blocking code. Spring Boot 3.2+ supports natively.

**Q: What is the volatile keyword?**
Guarantees visibility across threads. A volatile write happens-before subsequent volatile reads. It does NOT provide atomicity — use `AtomicInteger` or `synchronized` for compound operations.

### JVM

**Q: Explain JVM memory areas.**
Heap (objects — Young Gen + Old Gen), Stack (per-thread — local vars, method calls), Metaspace (class metadata), Code Cache (JIT-compiled code).

**Q: What garbage collectors do you know? Which would you choose for a REST API?**
Serial (dev), Parallel (batch), G1 (default, balanced), ZGC (sub-ms pauses), Shenandoah. For REST APIs: G1 (general) or ZGC (latency-critical).

---

## 2. Spring Boot Questions

**Q: What is the difference between Spring and Spring Boot?**
Spring is the framework (IoC, DI, AOP, MVC). Spring Boot is an opinionated layer providing auto-configuration, embedded server, starter deps, and production features (Actuator).

**Q: How does auto-configuration work?**
`@EnableAutoConfiguration` loads auto-configuration classes from `META-INF/spring/AutoConfiguration.imports`. Each uses `@Conditional` annotations to determine if beans should be created.

**Q: Why is constructor injection preferred over field injection?**
Immutability (`final` fields), explicit dependencies (easy to see what a class needs), testability (pass mocks via constructor), fails fast at startup if a dependency is missing.

**Q: Explain the Bean lifecycle.**
Instantiation → Dependency Injection → Aware callbacks → `@PostConstruct` → BeanPostProcessor → **Ready** → `@PreDestroy` → Destroy.

**Q: What is the proxy self-invocation problem?**
Calling a `@Transactional` method from within the same class bypasses the AOP proxy — the annotation has no effect. Fix: inject self via `@Lazy @Autowired`, or extract to a separate bean.

**Q: What are Spring profiles?**
Mechanism to activate different configurations per environment. `@Profile("dev")` beans are only loaded when `spring.profiles.active=dev`. Use for dev vs prod DataSource, logging levels, etc.

**Q: How does Spring MVC process a request?**
Request → Filter chain → DispatcherServlet → HandlerMapping (finds controller) → HandlerAdapter → Interceptors → @Controller method → MessageConverter (JSON) → Response.

**Q: What is `@Transactional` and how does it work?**
Declarative transaction management via AOP proxy. Starts transaction before method, commits on success, rolls back on RuntimeException. Supports propagation (REQUIRED, REQUIRES_NEW), isolation levels, rollback rules.

---

## 3. Database Questions

**Q: What is an index? When would you NOT create one?**
B-Tree data structure for fast lookups. Don't create on: rarely queried columns, small tables, columns with very low cardinality, or tables with heavy writes and few reads.

**Q: Explain ACID.**
Atomicity (all or nothing), Consistency (valid state transitions), Isolation (concurrent transactions don't interfere), Durability (committed data survives crashes).

**Q: What are isolation levels?**
READ_UNCOMMITTED (dirty reads possible), READ_COMMITTED (no dirty reads — PG default), REPEATABLE_READ (no non-repeatable reads — MySQL default), SERIALIZABLE (strictest, slowest).

**Q: Optimistic vs pessimistic locking?**
Optimistic: version column checked at update time — no locks, retries on conflict. Use for low-contention (most web apps). Pessimistic: `SELECT FOR UPDATE` — locks rows during transaction. Use for high-contention (finance).

**Q: What is the N+1 problem and how do you fix it?**
1 query for N parents → N queries for their children = N+1 total. Fix: JOIN FETCH, `@EntityGraph`, `@BatchSize`, or DTO projections.

**Q: What is connection pooling?**
Reusing pre-created database connections instead of creating new ones per query. HikariCP is Spring Boot's default. Pool size ≈ (cores × 2) + spindles.

---

## 4. System Design Questions

### How to Approach (5-Step Framework)

1. **Clarify** (3-5 min): Functional requirements, non-functional requirements, scale
2. **Estimate** (2 min): Users, QPS, data volume, storage
3. **High-Level Design** (10 min): Components, data flow, API design
4. **Deep Dive** (15 min): Database schema, caching, scaling, trade-offs
5. **Edge Cases** (5 min): Failures, monitoring, security

### Practice Scenarios

**Design a URL Shortener:**
- API: POST /shorten → returns short URL. GET /abc → redirects
- Short code: Base62 encoding or random 7 chars
- Storage: Relational DB + Redis cache for hot URLs
- Scale: Read-heavy → cache aggressively, read replicas

**Design a Rate Limiter:**
- Algorithms: Token bucket (most common), sliding window
- Storage: Redis (INCR + EXPIRE)
- Where: API Gateway or per-service middleware
- Rules: Per user, per IP, per API key

**Design a Notification System:**
- Components: API → Priority Queue (Kafka) → Workers → Providers (SMS/Email/Push)
- Challenges: Retry with backoff, deduplication, user preferences
- Scale: Partition by user ID for ordering

**Design an E-commerce Order System:**
- Services: User, Product, Order, Payment, Inventory, Notification
- Patterns: Saga for distributed transactions, Event-driven communication
- Database: PostgreSQL per service, Redis for caching
- Consistency: Eventual consistency between services

---

## 5. Scenario & Debugging Questions

**Q: Your API response time increased from 200ms to 5 seconds. How do you debug?**
1. Check monitoring dashboards (Grafana) for anomalies
2. Distributed tracing (Zipkin) to identify the slow span
3. If DB: check slow query log, run EXPLAIN ANALYZE
4. If external service: check circuit breaker state, timeout configs
5. If GC: check GC logs for long pauses
6. If thread pool: check thread dump for blocked threads

**Q: Users report intermittent 500 errors, but logs show no errors. What happened?**
- Check if error logging is configured correctly (missing `@RestControllerAdvice`?)
- Check log level (ERROR only? Switch to WARN/INFO)
- Check downstream service logs (the error may be in another service)
- Check for connection pool exhaustion (no error logged, just timeout)
- Check for OOM in a sidecar or background thread

**Q: A feature works in staging but fails in production. Why?**
- Different database (schema, data volume, indexes)
- Different env variables / config (profiles not matched)
- Different scale (race conditions under load)
- Time-dependent logic (timezone differences)
- Dependency version differences

**Q: How do you handle a database migration with zero downtime?**
1. Backward-compatible schema changes only (add columns, not rename/delete)
2. Run Flyway migration on startup (add new column with default)
3. Deploy new code that writes to both old and new columns
4. Backfill data for existing rows
5. Deploy code that reads from new column
6. Remove old column in a future migration

---

## 6. Project Explanation Guide

### STAR Framework for Projects

**S — Situation:** What was the project? What problem did it solve?
**T — Task:** What was your specific responsibility?
**A — Action:** What technical decisions did you make and why?
**R — Result:** What was the outcome? Metrics?

### Example

> "I built an **order management microservice** for an e-commerce platform.
>
> **Challenge:** The monolithic order system couldn't scale — order processing took 5 seconds under peak load.
>
> **My role:** I designed and implemented the order service, payment integration, and event-driven notifications.
>
> **Technical decisions:**
> - Spring Boot with PostgreSQL and Redis caching
> - Saga pattern for distributed transactions (order → payment → inventory)
> - Kafka for async notification delivery
> - Circuit breaker (Resilience4j) for payment gateway calls
>
> **Result:** Order processing time reduced from 5s to 200ms. System handles 10x peak traffic with autoscaling."

### What Interviewers Look For

| What They Ask | What They Actually Evaluate |
|--------------|---------------------------|
| "Tell me about your project" | Can you communicate clearly? |
| "Why did you choose X?" | Decision-making and trade-off analysis |
| "What was the hardest challenge?" | Problem-solving and resilience |
| "What would you do differently?" | Self-awareness and growth mindset |
| "How does X work internally?" | Depth of understanding |

---

## 7. Common Mistakes

| Mistake | Why It's Bad | Fix |
|---------|-------------|-----|
| Vague answers | Shows no depth | Give specific examples with context |
| No trade-offs | Real engineering is about trade-offs | Always mention pros/cons |
| Memorized definitions | Sounds robotic, breaks on follow-ups | Understand deeply, explain in your words |
| Not asking questions | Shows passivity | Clarify requirements before jumping in |
| Rushing system design | Miss critical requirements | Spend 5 min clarifying before designing |
| Ignoring non-functionals | Shows inexperience | Always mention: scale, reliability, security |

---

## 8. Expected Depth by Level

| Topic | Junior (0-2 yr) | Mid (2-5 yr) | Senior (5+ yr) |
|-------|-----------------|--------------|-----------------|
| Core Java | Basics, Collections | Internals, Concurrency | JVM, Memory Model |
| Spring Boot | CRUD, annotations | Auto-config, lifecycle | Custom starters, internals |
| Database | Basic SQL, CRUD | Indexing, optimization | Sharding, distributed DB |
| System Design | — | Simple designs | Complex, trade-offs |
| Architecture | Layered | Clean/Hexagonal | DDD, Event Sourcing |
| Production | — | Monitoring basics | On-call, debugging, tuning |

---

## 9. Mock Interview Practice

### 30-Minute Mock Interview Plan

1. **Intro (3 min):** Tell me about yourself and your project
2. **Core Java (7 min):** 2-3 questions on collections, concurrency, JVM
3. **Spring Boot (7 min):** DI, auto-config, transactions, exception handling
4. **Database (5 min):** Indexing, N+1, transactions
5. **Scenario (5 min):** Production debugging or architecture decision
6. **System Design (3 min):** Quick design of a simple system

### Self-Assessment Checklist

- [ ] Can I explain HashMap internals with a whiteboard?
- [ ] Can I explain Spring Bean lifecycle from memory?
- [ ] Can I write a `@RestController` from scratch?
- [ ] Can I explain @Transactional propagation types?
- [ ] Can I design a URL shortener in 15 minutes?
- [ ] Can I debug a production latency spike step-by-step?
- [ ] Can I explain my project using STAR framework?

---

## 10. References

- [JavaGuide (GitHub)](https://github.com/Snailclimb/JavaGuide)
- [System Design Primer (GitHub)](https://github.com/donnemartin/system-design-primer)
- [Baeldung — Interview Questions](https://www.baeldung.com/java-interview-questions)
- [Spring Boot Interview Questions](https://www.baeldung.com/spring-boot-interview-questions)
- [LeetCode — System Design](https://leetcode.com/discuss/interview-question/system-design)
- [Grokking System Design](https://www.designgurus.io/course/grokking-the-system-design-interview)

---

> **Previous:** [← 13 — Backend Best Practices](./13_Backend_Best_Practices.md)
> **Next:** [15 — Backend Learning Roadmap →](./15_Backend_Learning_Roadmap.md)
