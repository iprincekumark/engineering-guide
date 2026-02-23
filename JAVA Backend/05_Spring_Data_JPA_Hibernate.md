# 05 — Spring Data JPA & Hibernate

> **Goal:** Master Java persistence — from JPA and Hibernate internals to transactions, performance pitfalls (N+1), and advanced querying.

---

## Table of Contents

1. [What is JPA?](#1-what-is-jpa)
2. [What is Hibernate?](#2-what-is-hibernate)
3. [Spring Data JPA](#3-spring-data-jpa)
4. [Entity Mapping](#4-entity-mapping)
5. [Relationship Mapping](#5-relationship-mapping)
6. [Lazy vs Eager Loading](#6-lazy-vs-eager-loading)
7. [N+1 Problem](#7-n1-problem)
8. [Querying Strategies](#8-querying-strategies)
9. [Transactions Deep Dive](#9-transactions-deep-dive)
10. [Hibernate Internals](#10-hibernate-internals)
11. [Auditing](#11-auditing)
12. [Performance Best Practices](#12-performance-best-practices)
13. [Interview Notes](#13-interview-notes)
14. [Summary](#14-summary)
15. [References](#15-references)

---

## 1. What is JPA?

**Definition:**
JPA (Jakarta Persistence API, formerly Java Persistence API) is a **specification** that defines a standard way to map Java objects to relational database tables and manage their lifecycle. It is NOT an implementation — it's a set of interfaces and annotations that ORM frameworks implement.

**Why it exists:**
Without JPA, you'd write raw JDBC code — manual SQL, ResultSet parsing, connection management. JPA abstracts this, letting you work with Java objects instead of SQL strings, while remaining vendor-agnostic.

**Real-life analogy:**
JPA is like an electrical outlet standard. The specification defines the plug shape and voltage. Different manufacturers (Hibernate, EclipseLink) build the actual outlets — but your device (application) works with any of them.

**Key characteristics:**
- Defines annotations: `@Entity`, `@Table`, `@Id`, `@Column`, `@OneToMany`, etc.
- Defines `EntityManager` API for CRUD operations
- Defines JPQL (Java Persistence Query Language) for object-oriented queries
- Defines transaction management integration

---

## 2. What is Hibernate?

**Definition:**
Hibernate is the most popular JPA implementation (ORM framework). It implements the JPA specification and adds additional features like caching (L1/L2), dirty checking, batch processing, and HQL.

**Why it exists:**
Hibernate was created before JPA existed. It pioneered ORM in Java. When JPA was standardized, Hibernate became the reference implementation.

**Hibernate-specific features beyond JPA:**
- **L2 Cache:** Application-level cache shared across sessions
- **Dirty checking:** Automatically detects changed fields and generates UPDATE SQL with only modified columns
- **Batch operations:** `hibernate.jdbc.batch_size` for bulk inserts/updates
- **Custom types:** `@Type` for custom column mappings
- **Filters:** `@Filter` for dynamic WHERE clauses

---

## 3. Spring Data JPA

**Definition:**
Spring Data JPA is a Spring module that sits on top of JPA/Hibernate and eliminates even more boilerplate by generating repository implementations automatically from interface method names.

**Backend example:**
```java
// Spring Data JPA — you define the interface, Spring generates the implementation
public interface UserRepository extends JpaRepository<User, Long> {

    // Derived query — Spring generates SQL from method name
    Optional<User> findByEmail(String email);

    List<User> findByStatusAndCreatedAtAfter(UserStatus status, LocalDateTime date);

    boolean existsByEmail(String email);

    @Query("SELECT u FROM User u WHERE u.department.name = :deptName")
    List<User> findByDepartmentName(@Param("deptName") String departmentName);

    @Query(value = "SELECT * FROM users WHERE email LIKE %:domain", nativeQuery = true)
    List<User> findByEmailDomain(@Param("domain") String domain);

    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.lastLoginAt < :date")
    int deactivateInactiveUsers(@Param("status") UserStatus status, @Param("date") LocalDateTime date);
}
```

---

## 4. Entity Mapping

**Definition:**
Entity mapping is the process of defining how a Java class (entity) maps to a database table. Each field maps to a column, and the entity lifecycle is managed by JPA.

**Backend example:**
```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_user_email", columnList = "email", unique = true),
    @Index(name = "idx_user_status", columnList = "status")
})
@Getter @Setter
@NoArgsConstructor
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true, length = 255)
    private String email;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private UserStatus status = UserStatus.ACTIVE;

    @Column(name = "password_hash", nullable = false)
    private String passwordHash;

    @Embedded
    private Address address;

    @CreationTimestamp
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;

    @Version // Optimistic locking
    private Long version;
}

// Embeddable value object
@Embeddable
@Getter @Setter
public class Address {
    private String street;
    private String city;
    private String state;

    @Column(name = "zip_code")
    private String zipCode;
}
```

**ID generation strategies:**

| Strategy | Description | Use Case |
|----------|-------------|----------|
| `IDENTITY` | Database auto-increment | MySQL, PostgreSQL (simple) |
| `SEQUENCE` | Database sequence | PostgreSQL (high-performance) |
| `TABLE` | Uses a separate table | Portable, but slow |
| `UUID` | UUID generator | Distributed systems, APIs |

---

## 5. Relationship Mapping

### 5.1 @OneToMany / @ManyToOne

**Definition:**
Represents a parent–child relationship where one entity has many related entities (e.g., one User has many Orders).

```java
@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // One user has many orders
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();

    // Helper methods for bidirectional consistency
    public void addOrder(Order order) {
        orders.add(order);
        order.setUser(this);
    }

    public void removeOrder(Order order) {
        orders.remove(order);
        order.setUser(null);
    }
}

@Entity
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Many orders belong to one user
    @ManyToOne(fetch = FetchType.LAZY) // ALWAYS use LAZY for @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
}
```

### 5.2 @ManyToMany

```java
@Entity
public class Student {
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}
```

### 5.3 @OneToOne

```java
@Entity
public class User {
    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private UserProfile profile;
}

@Entity
public class UserProfile {
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    @MapsId // Shares the same PK as User
    private User user;
}
```

---

## 6. Lazy vs Eager Loading

**Definition:**
- **Lazy loading:** Related entities are NOT loaded from the database until you explicitly access them. A proxy is created instead.
- **Eager loading:** Related entities are loaded immediately with the parent entity in a single query (or N+1 queries).

**Why it matters:**
Eager loading fetches everything — even data you don't need. This causes unnecessary queries, memory bloat, and performance degradation. Lazy loading defers queries until actually needed.

**Default fetch types:**

| Mapping | Default Fetch |
|---------|--------------|
| `@ManyToOne` | EAGER ⚠️ (change to LAZY!) |
| `@OneToOne` | EAGER ⚠️ (change to LAZY!) |
| `@OneToMany` | LAZY ✅ |
| `@ManyToMany` | LAZY ✅ |

**Best practice:** Always use `FetchType.LAZY` and load eagerly only when needed via JOIN FETCH.

```java
// ❌ BAD — eager loading by default
@ManyToOne // defaults to EAGER
private User user;

// ✅ GOOD — explicit lazy
@ManyToOne(fetch = FetchType.LAZY)
private User user;
```

---

## 7. N+1 Problem

**Definition:**
The N+1 problem occurs when Hibernate executes 1 query to fetch N parent entities, and then N additional queries (one per parent) to fetch each parent's related entities. For 100 users with orders, this means 1 + 100 = 101 queries instead of 1-2.

**Why it exists:**
Lazy loading defers loading until access. When you iterate over parents and access their children, each access triggers a separate SQL query. It's a performance killer.

**Real-life analogy:**
Ordering 100 packages from Amazon, but each package is shipped in a separate delivery truck (query) instead of all in one truck.

**Backend example — the problem:**
```java
// Triggers N+1!
List<User> users = userRepository.findAll(); // 1 query: SELECT * FROM users
for (User user : users) {
    user.getOrders().size(); // N queries: SELECT * FROM orders WHERE user_id = ?
}
```

**Solutions:**

```java
// Solution 1: JOIN FETCH (JPQL)
@Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.status = :status")
List<User> findByStatusWithOrders(@Param("status") UserStatus status);

// Solution 2: @EntityGraph
@EntityGraph(attributePaths = {"orders", "orders.items"})
List<User> findByStatus(UserStatus status);

// Solution 3: @BatchSize (Hibernate-specific)
@OneToMany(mappedBy = "user")
@BatchSize(size = 25) // Loads 25 users' orders in one query
private List<Order> orders;

// Solution 4: DTO Projection (best for read-only)
@Query("""
    SELECT new com.app.dto.UserOrderSummary(u.id, u.name, COUNT(o))
    FROM User u LEFT JOIN u.orders o
    GROUP BY u.id, u.name
    """)
List<UserOrderSummary> findUserOrderSummaries();
```

---

## 8. Querying Strategies

### 8.1 Derived Queries

```java
// Spring Data derives SQL from method name
List<User> findByNameContainingIgnoreCase(String name);
List<User> findTop10ByStatusOrderByCreatedAtDesc(UserStatus status);
long countByStatus(UserStatus status);
void deleteByStatusAndCreatedAtBefore(UserStatus status, LocalDateTime date);
```

### 8.2 JPQL Queries

```java
@Query("SELECT u FROM User u WHERE u.email LIKE %:domain AND u.status = :status")
List<User> findActiveByEmailDomain(@Param("domain") String domain, @Param("status") UserStatus status);
```

### 8.3 Native Queries

```java
@Query(value = """
    SELECT u.*, COUNT(o.id) as order_count
    FROM users u LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.id
    HAVING COUNT(o.id) > :minOrders
    """, nativeQuery = true)
List<User> findUsersWithMinOrders(@Param("minOrders") int minOrders);
```

### 8.4 Specifications (Dynamic Queries)

```java
// Specification for dynamic filtering
public class UserSpecifications {

    public static Specification<User> hasStatus(UserStatus status) {
        return (root, query, cb) -> cb.equal(root.get("status"), status);
    }

    public static Specification<User> nameLike(String name) {
        return (root, query, cb) -> cb.like(cb.lower(root.get("name")), "%" + name.toLowerCase() + "%");
    }

    public static Specification<User> createdAfter(LocalDateTime date) {
        return (root, query, cb) -> cb.greaterThan(root.get("createdAt"), date);
    }
}

// Usage — combine dynamically
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {}

// Service
public Page<UserDto> searchUsers(UserSearchCriteria criteria, Pageable pageable) {
    Specification<User> spec = Specification.where(null);

    if (criteria.getStatus() != null)
        spec = spec.and(UserSpecifications.hasStatus(criteria.getStatus()));
    if (criteria.getName() != null)
        spec = spec.and(UserSpecifications.nameLike(criteria.getName()));
    if (criteria.getCreatedAfter() != null)
        spec = spec.and(UserSpecifications.createdAfter(criteria.getCreatedAfter()));

    return userRepository.findAll(spec, pageable).map(this::toDto);
}
```

### 8.5 Projections

```java
// Interface projection (read-only, lightweight)
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}

List<UserSummary> findByStatus(UserStatus status);

// Class projection (DTO)
public record UserNameEmail(Long id, String name, String email) {}

@Query("SELECT new com.app.dto.UserNameEmail(u.id, u.name, u.email) FROM User u WHERE u.status = 'ACTIVE'")
List<UserNameEmail> findActiveUserSummaries();
```

---

## 9. Transactions Deep Dive

### 9.1 @Transactional

**Definition:**
`@Transactional` is a Spring annotation that declaratively manages database transactions. When applied to a method, Spring starts a transaction before the method executes, commits it if the method succeeds, and rolls back if an unchecked exception is thrown.

**Why it exists:**
Manual transaction management (begin → execute → commit/rollback) is verbose and error-prone. `@Transactional` automates this through AOP proxies.

**Backend example:**
```java
@Service
@RequiredArgsConstructor
public class TransferService {

    private final AccountRepository accountRepository;

    @Transactional // All or nothing — both updates succeed or both roll back
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepository.findById(fromId)
            .orElseThrow(() -> new ResourceNotFoundException("Account", "id", fromId));
        Account to = accountRepository.findById(toId)
            .orElseThrow(() -> new ResourceNotFoundException("Account", "id", toId));

        from.debit(amount);   // If this succeeds...
        to.credit(amount);    // ...but this fails → both roll back

        accountRepository.save(from);
        accountRepository.save(to);
    }
}
```

### 9.2 Propagation

| Propagation | Behavior |
|-------------|----------|
| `REQUIRED` (default) | Join existing transaction or create new one |
| `REQUIRES_NEW` | Always create a new transaction (suspends current) |
| `MANDATORY` | Must run within existing transaction (throws if none) |
| `SUPPORTS` | Use transaction if exists, otherwise non-transactional |
| `NOT_SUPPORTED` | Run non-transactionally (suspend if exists) |
| `NEVER` | Must NOT run within a transaction (throws if exists) |
| `NESTED` | Nested transaction with savepoints |

```java
@Transactional
public void placeOrder(OrderRequest request) {
    Order order = orderRepository.save(toEntity(request));

    // Audit log should persist even if order processing fails later
    auditService.log(order); // Uses REQUIRES_NEW
}

@Service
public class AuditService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(Order order) {
        auditRepository.save(new AuditLog("ORDER_PLACED", order.getId()));
    }
}
```

### 9.3 Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|-------------------|-------------|
| `READ_UNCOMMITTED` | ✅ Possible | ✅ Possible | ✅ Possible |
| `READ_COMMITTED` | ❌ Prevented | ✅ Possible | ✅ Possible |
| `REPEATABLE_READ` | ❌ Prevented | ❌ Prevented | ✅ Possible |
| `SERIALIZABLE` | ❌ Prevented | ❌ Prevented | ❌ Prevented |

**Default:** PostgreSQL = READ_COMMITTED, MySQL = REPEATABLE_READ

### 9.4 Rollback Rules

```java
// Rolls back only on unchecked exceptions (RuntimeException) by default
@Transactional

// Roll back on checked exceptions too
@Transactional(rollbackFor = Exception.class)

// Don't roll back on specific exception
@Transactional(noRollbackFor = EmailSendException.class)
```

---

## 10. Hibernate Internals

### 10.1 Persistence Context (L1 Cache)

**Definition:**
The persistence context (also called L1 cache or first-level cache) is a session-scoped cache that Hibernate maintains. Every entity loaded within a transaction is cached, and changes are tracked automatically (dirty checking).

```java
@Transactional
public void updateUser(Long id, String newName) {
    User user = userRepository.findById(id).orElseThrow(); // Loaded into L1 cache
    user.setName(newName); // Dirty checking detects this change
    // NO explicit save() needed — Hibernate auto-flushes at transaction commit
}
```

### 10.2 Entity States

```
┌───────────┐    persist()    ┌───────────┐
│  Transient │──────────────→│  Managed  │
│  (new)     │               │ (in PC)   │
└───────────┘               └─────┬─────┘
                                  │
                    detach() /    │  merge()
                    clear()       │
                                  ▼
                            ┌───────────┐
                            │ Detached  │
                            │(outside PC)│
                            └───────────┘
                                  │
                    remove()      │
                                  ▼
                            ┌───────────┐
                            │  Removed  │
                            └───────────┘
```

- **Transient:** New object, not yet persisted
- **Managed:** In persistence context, changes are tracked
- **Detached:** Was managed, now outside the persistence context
- **Removed:** Marked for deletion

### 10.3 Flush Modes

**Flush** is when Hibernate synchronizes in-memory changes to the database (executes SQL).

| Mode | When |
|------|------|
| `AUTO` (default) | Before queries, at commit |
| `COMMIT` | Only at transaction commit |
| `MANUAL` | Only when `flush()` is called explicitly |

---

## 11. Auditing

**Definition:**
JPA auditing automatically tracks who created/modified an entity and when, without manual code in every service method.

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext()
            .getAuthentication())
            .map(Authentication::getName)
            .or(() -> Optional.of("system"));
    }
}

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter @Setter
public abstract class BaseEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String updatedBy;
}

@Entity
public class Product extends BaseEntity {
    private String name;
    private BigDecimal price;
    // createdAt, updatedAt, createdBy, updatedBy are automatic
}
```

---

## 12. Performance Best Practices

| Practice | Why |
|----------|-----|
| Always use `FetchType.LAZY` | Prevent unnecessary data loading |
| Use JOIN FETCH for read queries | Solve N+1 in one query |
| Use DTO projections for read-only | Less memory, faster queries |
| Enable batch operations | `hibernate.jdbc.batch_size=25` |
| Use `@Version` for optimistic locking | Prevent lost updates |
| Avoid `CascadeType.ALL` blindly | Can cause unintended deletes |
| Use `@QueryHints` for read-only | `@QueryHints(@QueryHint(name = HINT_READONLY, value = "true"))` |
| Monitor SQL with `show-sql` / p6spy | Catch N+1 in development |
| Use connection pooling (HikariCP) | Spring Boot default, configure pool size |

---

## 13. Interview Notes

1. **What is the N+1 problem? How do you fix it?**
   - 1 query for parents + N queries for children. Fix: JOIN FETCH, @EntityGraph, @BatchSize, or DTO projections.

2. **Explain @Transactional propagation types.**
   - REQUIRED (default, join or create), REQUIRES_NEW (always new), MANDATORY (must exist), etc.

3. **What is dirty checking in Hibernate?**
   - Hibernate compares current entity state with snapshot at load time. Changed fields generate UPDATE SQL automatically at flush.

4. **Lazy vs Eager loading — which is default and which should you use?**
   - `@ManyToOne`/`@OneToOne` default to EAGER (change to LAZY!). `@OneToMany`/`@ManyToMany` default to LAZY. Always prefer LAZY.

5. **What is the persistence context?**
   - L1 cache scoped to a session/transaction. Entities are cached, tracked, and auto-flushed.

6. **How does @Version (optimistic locking) work?**
   - Adds a version column. On update, Hibernate checks `WHERE id = ? AND version = ?`. If version changed, throws `OptimisticLockingFailureException`.

7. **When would you use a native query vs JPQL?**
   - JPQL for entity queries (portable). Native SQL for database-specific features, complex aggregations, or performance-critical queries.

---

## 14. Summary

| Topic | Key Takeaway |
|-------|-------------|
| JPA | Specification (interfaces). Hibernate is the implementation |
| Spring Data JPA | Auto-generates repos from method names |
| Entity Mapping | `@Entity`, `@Table`, `@Column`, `@Id` + generation strategies |
| Relationships | Always LAZY. Use helper methods for bidirectional consistency |
| N+1 | Detect early, fix with JOIN FETCH or projections |
| Transactions | `@Transactional` = AOP proxy. Watch for self-invocation trap |
| Persistence Context | L1 cache, dirty checking, auto-flush |
| Auditing | `@CreatedDate`, `@LastModifiedBy` — automatic with `@EnableJpaAuditing` |

---

## 15. References

- [Spring Data JPA Documentation](https://docs.spring.io/spring-data/jpa/reference/jpa.html)
- [Hibernate ORM Documentation](https://hibernate.org/orm/documentation/)
- [Baeldung — Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)
- [Vlad Mihalcea — JPA/Hibernate Blog](https://vladmihalcea.com/)
- [Baeldung — N+1 Problem](https://www.baeldung.com/hibernate-n-plus-1-problem)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)

---

> **Previous:** [← 04 — Spring Boot](./04_Spring_Boot.md)  
> **Next:** [06 — Spring Security →](./06_Spring_Security.md)
