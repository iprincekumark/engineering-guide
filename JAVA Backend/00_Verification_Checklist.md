# 00 — Verification Checklist

> A quick-reference checklist to verify that you've covered all essential Java Backend concepts. Use this before interviews or when reviewing your knowledge.

---

## Core Java

- [ ] Can you explain OOP principles with backend examples?
- [ ] How does HashMap work internally? (hash, buckets, collisions, tree)
- [ ] What is the difference between `==` and `.equals()`?
- [ ] Can you write Streams pipelines (filter, map, collect, groupingBy)?
- [ ] Checked vs unchecked exceptions — when to use each?
- [ ] synchronized vs ReentrantLock vs volatile?
- [ ] What are virtual threads (JDK 21+)?
- [ ] Explain JVM memory areas (Heap, Stack, Metaspace)
- [ ] Name 3 GC algorithms and when to use each
- [ ] Explain SOLID principles with code examples

---

## Spring Framework

- [ ] What is Inversion of Control (IoC) and Dependency Injection (DI)?
- [ ] Constructor vs Field vs Setter injection — which is best and why?
- [ ] Explain the Bean lifecycle (creation → init → use → destroy)
- [ ] What are Bean scopes? (Singleton, Prototype, Request, Session)
- [ ] What is AOP? How do Spring proxies work?
- [ ] How does auto-configuration work in Spring Boot?
- [ ] Explain @Transactional (propagation, isolation, rollback rules)
- [ ] What is the proxy self-invocation problem?
- [ ] How does Spring MVC process an HTTP request?

---

## Database & JPA

- [ ] Can you write complex SQL (JOINs, window functions, CTEs)?
- [ ] What is database normalization? (1NF, 2NF, 3NF)
- [ ] How does indexing work? (B-Tree, composite, partial)
- [ ] Can you read an EXPLAIN ANALYZE plan?
- [ ] Lazy vs Eager loading — defaults and best practices?
- [ ] What is the N+1 problem? 4 ways to fix it?
- [ ] Explain entity states (Transient, Managed, Detached, Removed)
- [ ] Optimistic vs Pessimistic locking?
- [ ] What is connection pooling (HikariCP)?
- [ ] How do you scale a database? (replicas, partitioning, sharding)

---

## Security

- [ ] How does the Spring Security filter chain work?
- [ ] JWT: generate, validate, structure (Header.Payload.Signature)
- [ ] OAuth2 flows (Authorization Code, PKCE, Client Credentials)
- [ ] Password encoding (BCrypt, Argon2 — never plaintext)
- [ ] CORS — what and why? How to configure?
- [ ] CSRF — when to disable for REST APIs?
- [ ] @PreAuthorize vs @PostAuthorize
- [ ] RBAC: Users → Roles → Permissions

---

## Architecture & System Design

- [ ] Layered architecture (Controller → Service → Repository)
- [ ] Clean Architecture / Hexagonal Architecture concepts
- [ ] DDD basics (Entity, Value Object, Aggregate, Bounded Context)
- [ ] CQRS and Event Sourcing — when to use?
- [ ] Microservices vs Monolith — trade-offs?
- [ ] API Gateway, Service Discovery, Load Balancing
- [ ] Circuit Breaker pattern (Resilience4j)
- [ ] Saga pattern for distributed transactions
- [ ] CAP theorem — CP vs AP?
- [ ] Design a URL shortener / rate limiter / notification system

---

## Caching & Messaging

- [ ] Cache patterns (Cache-Aside, Write-Through, Write-Behind)
- [ ] Redis vs Caffeine — when to use each?
- [ ] Spring @Cacheable, @CachePut, @CacheEvict
- [ ] Kafka vs RabbitMQ — key differences?
- [ ] Event-driven architecture principles
- [ ] What is a cache stampede and how to prevent it?

---

## Observability & Performance

- [ ] Three pillars: Logs, Metrics, Traces
- [ ] Structured logging (JSON, MDC, SLF4J)
- [ ] Metrics (Micrometer → Prometheus → Grafana)
- [ ] Distributed tracing (OpenTelemetry, Zipkin)
- [ ] JVM tuning flags (-Xms, -Xmx, GC selection)
- [ ] Thread pool sizing (Tomcat, HikariCP, @Async)

---

## Production & Best Practices

- [ ] DTO vs Entity separation — why mandatory?
- [ ] Exception hierarchy + @RestControllerAdvice
- [ ] Layered validation (Controller → Service → DB)
- [ ] 12-Factor App principles
- [ ] Test pyramid (Unit > Integration > E2E)
- [ ] Docker multi-stage builds for Spring Boot
- [ ] CI/CD pipeline stages
- [ ] Kubernetes basics (Pod, Deployment, Service, Probes)

---

## Real-World Debugging

- [ ] How to diagnose a memory leak?
- [ ] How to fix a slow query?
- [ ] How to debug a deadlock?
- [ ] How to handle thread/connection pool exhaustion?
- [ ] How to reduce GC pauses?
- [ ] How to prevent cascade failures in microservices?

---

## Files Index

| # | File | Topic |
|---|------|-------|
| 01 | [Java Backend Fundamentals](./01_Java_Backend_Fundamentals.md) | Core Java, OOP, Collections, Streams, Concurrency, JVM |
| 02 | [HTTP, REST & Backend Basics](./02_HTTP_REST_Backend_Basics.md) | HTTP, REST, JSON, API Design, Validation |
| 03 | [Spring Core](./03_Spring_Core.md) | IoC, DI, Beans, AOP, Proxies, Profiles |
| 04 | [Spring Boot](./04_Spring_Boot.md) | Auto-config, MVC, REST, Actuator, Async |
| 05 | [Spring Data JPA & Hibernate](./05_Spring_Data_JPA_Hibernate.md) | Entity Mapping, N+1, Transactions, Queries |
| 06 | [Spring Security](./06_Spring_Security.md) | JWT, OAuth2, Filter Chain, RBAC |
| 07 | [Database Design & Persistence](./07_Database_Design_Persistence.md) | SQL, Indexing, Optimization, Scaling |
| 08 | [Backend Architecture & System Design](./08_Backend_Architecture_System_Design.md) | Clean, Hexagonal, DDD, CQRS |
| 09 | [Microservices & Distributed Backend](./09_Microservices_Distributed_Backend.md) | Gateway, Circuit Breaker, Saga, CAP |
| 10 | [Caching, Messaging & Async](./10_Caching_Messaging_Async.md) | Redis, Kafka, RabbitMQ, Event-Driven |
| 11 | [Observability & Performance](./11_Observability_Performance_Tuning.md) | Logging, Metrics, Tracing, JVM Tuning |
| 12 | [Real-World Issues & Fixes](./12_Real_World_Backend_Issues_Fixes.md) | Memory Leaks, Slow Queries, Deadlocks |
| 13 | [Backend Best Practices](./13_Backend_Best_Practices.md) | DTOs, Exceptions, Validation, 12-Factor |
| 14 | [Interview Prep](./14_Java_Backend_Interview_Prep.md) | Q&A, System Design, Project Explanation |
| 15 | [Learning Roadmap](./15_Backend_Learning_Roadmap.md) | Phased Study Plan with Projects |
| 16 | [Testing Strategy](./16_Testing_Strategy_Deep_Dive.md) | JUnit, Mockito, TestContainers, TDD |
| 17 | [DevOps & CI/CD](./17_DevOps_CI_CD_Containerization.md) | Docker, Kubernetes, GitHub Actions |

---

> **Good luck with your Java Backend journey! 🚀**
