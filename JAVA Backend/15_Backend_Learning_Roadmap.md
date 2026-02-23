# 15 — Backend Learning Roadmap

> **Goal:** A step-by-step path from beginner to advanced Java Backend developer, with timelines, projects to build, and resources at each stage.

---

## Table of Contents

1. [Phase 1: Foundation (Weeks 1–4)](#phase-1-foundation-weeks-14)
2. [Phase 2: Spring Boot & APIs (Weeks 5–8)](#phase-2-spring-boot--apis-weeks-58)
3. [Phase 3: Database Mastery (Weeks 9–11)](#phase-3-database-mastery-weeks-911)
4. [Phase 4: Security & Testing (Weeks 12–14)](#phase-4-security--testing-weeks-1214)
5. [Phase 5: Architecture & System Design (Weeks 15–18)](#phase-5-architecture--system-design-weeks-1518)
6. [Phase 6: Advanced & Production (Weeks 19–22)](#phase-6-advanced--production-weeks-1922)
7. [Phase 7: Interview Prep (Weeks 23–26)](#phase-7-interview-prep-weeks-2326)
8. [Projects to Build](#projects-to-build)
9. [Daily Practice Routine](#daily-practice-routine)
10. [Resources](#resources)

---

## Phase 1: Foundation (Weeks 1–4)

### What to Learn
- [ ] Java syntax, data types, control flow
- [ ] OOP deep: Encapsulation, Inheritance, Polymorphism, Abstraction
- [ ] Collections: List, Set, Map, Queue (internals of HashMap, ArrayList)
- [ ] Exception handling (checked vs unchecked, custom exceptions)
- [ ] Streams API (filter, map, reduce, collect, groupingBy)
- [ ] Generics basics
- [ ] Basic multithreading (Thread, Runnable, synchronized)
- [ ] Build tool: Maven or Gradle basics

### Project to Build
**Console-based Library Management System**
- CRUD operations on books/users
- Use Collections (HashMap for storage)
- Exception handling for invalid operations
- Streams for searching/filtering

### Milestone Check
- [ ] Can you explain HashMap internals?
- [ ] Can you write Streams pipelines from memory?
- [ ] Can you create custom exceptions?

---

## Phase 2: Spring Boot & APIs (Weeks 5–8)

### What to Learn
- [ ] Spring Core: IoC, DI, Bean lifecycle, `@Component`, `@Service`, `@Repository`
- [ ] Spring Boot: Auto-configuration, `application.yml`, profiles
- [ ] REST Controllers: `@RestController`, `@GetMapping`, `@PostMapping`, etc.
- [ ] Request/response handling: `@PathVariable`, `@RequestParam`, `@RequestBody`
- [ ] DTOs and mappers (entity ↔ DTO separation)
- [ ] Validation: `@Valid`, Bean Validation annotations
- [ ] Exception handling: `@RestControllerAdvice`, global exception handler
- [ ] HTTP basics: Methods, status codes, headers

### Project to Build
**REST API: Task Management System**
- CRUD REST API with Spring Boot
- Input validation with Bean Validation
- Global exception handler with consistent error responses
- Proper HTTP status codes
- Swagger/OpenAPI documentation
- Pagination and sorting

### Milestone Check
- [ ] Can you build a REST API from scratch?
- [ ] Can you explain the Spring Bean lifecycle?
- [ ] Can you handle exceptions globally?

---

## Phase 3: Database Mastery (Weeks 9–11)

### What to Learn
- [ ] SQL deep: JOINs, subqueries, aggregations, window functions
- [ ] Spring Data JPA: Entities, repositories, derived queries, JPQL
- [ ] Hibernate: Persistence context, dirty checking, entity states
- [ ] Relationships: `@OneToMany`, `@ManyToOne`, `@ManyToMany`
- [ ] Lazy vs eager loading, N+1 problem and fixes
- [ ] `@Transactional`: Propagation, isolation, rollback rules
- [ ] Database migrations: Flyway or Liquibase
- [ ] Indexing basics and EXPLAIN plans

### Project to Build
**E-Commerce API**
- Products, Categories, Orders, Users (entity relationships)
- JPA with PostgreSQL
- Complex queries (find top-selling products, revenue reports)
- Database migrations with Flyway
- Pagination for product listings

### Milestone Check
- [ ] Can you explain the N+1 problem and fix it?
- [ ] Can you write a JOIN FETCH query?
- [ ] Can you read an EXPLAIN plan?

---

## Phase 4: Security & Testing (Weeks 12–14)

### What to Learn
- [ ] Spring Security: Authentication, Authorization, SecurityFilterChain
- [ ] JWT: Generate, validate, refresh tokens
- [ ] Role-based access control (RBAC)
- [ ] Password encoding (BCrypt)
- [ ] CORS configuration
- [ ] Unit testing: JUnit 5, Mockito
- [ ] Integration testing: `@SpringBootTest`, MockMvc
- [ ] Test containers for database tests

### Project to Build
**Secure Blog API**
- User registration + login with JWT
- Role-based endpoints (admin, user)
- Blog posts CRUD (only author can edit/delete)
- Unit tests for services
- Integration tests for controllers

### Milestone Check
- [ ] Can you implement JWT auth from scratch?
- [ ] Can you write unit tests with Mockito?
- [ ] Can you explain the Spring Security filter chain?

---

## Phase 5: Architecture & System Design (Weeks 15–18)

### What to Learn
- [ ] Layered architecture (Controller → Service → Repository)
- [ ] Clean Architecture basics
- [ ] Design patterns: Strategy, Observer, Factory, Builder
- [ ] Caching: Redis, `@Cacheable`
- [ ] Messaging basics: Kafka or RabbitMQ
- [ ] System design methodology (5-step framework)
- [ ] API design best practices, versioning
- [ ] Docker basics: Dockerfile, docker-compose

### Project to Build
**URL Shortener (System Design Implementation)**
- REST API: Shorten URL, redirect, analytics
- Redis caching for hot URLs
- Docker Compose (app + PostgreSQL + Redis)
- Rate limiting
- Click analytics (async via Kafka or Spring Events)

### Milestone Check
- [ ] Can you design a URL shortener on a whiteboard?
- [ ] Can you explain cache-aside pattern?
- [ ] Can you dockerize a Spring Boot app?

---

## Phase 6: Advanced & Production (Weeks 19–22)

### What to Learn
- [ ] Microservices: API Gateway, Service Discovery, Circuit Breaker
- [ ] Distributed systems: CAP theorem, Saga pattern
- [ ] Observability: Structured logging, Micrometer metrics, distributed tracing
- [ ] JVM internals: Memory model, GC algorithms, tuning flags
- [ ] Performance: Connection pooling, thread pools, async processing
- [ ] Virtual threads (JDK 21+)
- [ ] CI/CD: GitHub Actions or Jenkins
- [ ] Kubernetes basics

### Project to Build
**Microservices: Order Management System**
- 3-4 services: User, Product, Order, Payment
- API Gateway (Spring Cloud Gateway)
- Service Discovery (Eureka or Consul)
- Inter-service communication (REST + Kafka)
- Circuit breaker (Resilience4j)
- Docker Compose for local development
- Centralized logging

### Milestone Check
- [ ] Can you explain the Saga pattern?
- [ ] Can you configure Resilience4j circuit breaker?
- [ ] Can you debug a request across microservices?

---

## Phase 7: Interview Prep (Weeks 23–26)

### Daily Practice
- [ ] 2-3 Java backend interview questions daily
- [ ] 1 system design problem per week
- [ ] Mock interviews (peer or online)
- [ ] Review real-world issues from [File 12](./12_Real_World_Backend_Issues_Fixes.md)
- [ ] Practice explaining your projects using STAR framework

### Focus Areas
- Core Java (Collections, Concurrency, JVM)
- Spring Boot (DI, lifecycle, transactions)
- Database (Indexing, N+1, isolation levels)
- System Design (URL shortener, rate limiter, notification system)
- Scenario questions (production debugging)

---

## Projects to Build

| Stage | Project | Key Skills |
|-------|---------|-----------|
| Beginner | Library Management (console) | Java, Collections, Streams |
| Early Backend | Task Manager API | Spring Boot, REST, Validation |
| Database | E-Commerce API | JPA, SQL, Relationships |
| Security | Secure Blog API | JWT, RBAC, Testing |
| Architecture | URL Shortener | Redis, Docker, System Design |
| Advanced | Microservices Order System | Microservices, Kafka, Resilience4j |

---

## Daily Practice Routine

```
Morning (1 hour):
  - Read 1 concept from this guide
  - Write code example from memory

Afternoon (2 hours):
  - Work on current phase project
  - Practice builds and debugging

Evening (1 hour):
  - Review interview questions
  - Write notes/flashcards
  - 1 system design sketch (15 min)
```

---

## Resources

### Books
- *Head First Java* — Beginner
- *Effective Java* by Joshua Bloch — Intermediate
- *Spring in Action* by Craig Walls — Spring
- *Designing Data-Intensive Applications* by Martin Kleppmann — System Design

### Online
- [Baeldung](https://www.baeldung.com/) — Spring Boot tutorials
- [Vlad Mihalcea](https://vladmihalcea.com/) — JPA/Hibernate
- [Spring Official Guides](https://spring.io/guides)
- [System Design Primer (GitHub)](https://github.com/donnemartin/system-design-primer)
- [JavaGuide (GitHub)](https://github.com/Snailclimb/JavaGuide)

### Practice
- [LeetCode](https://leetcode.com/) — System Design section
- [HackerRank — Java](https://www.hackerrank.com/domains/java)
- [Spring Initializr](https://start.spring.io/) — Project scaffolding

---

> **Previous:** [← 14 — Interview Prep](./14_Java_Backend_Interview_Prep.md)
> **Next:** [16 — Testing Strategy Deep Dive →](./16_Testing_Strategy_Deep_Dive.md)
