# 07 — Database Design & Persistence

> **Goal:** Master SQL, indexing, query optimization, transactions, connection pooling, and database scaling — the data layer foundation of every backend.

---

## Table of Contents

1. [SQL Deep Dive](#1-sql-deep-dive)
2. [Normalization](#2-normalization)
3. [Indexing](#3-indexing)
4. [Query Optimization](#4-query-optimization)
5. [Transactions & Isolation Levels](#5-transactions--isolation-levels)
6. [Connection Pooling (HikariCP)](#6-connection-pooling-hikaricp)
7. [Database Performance Tuning](#7-database-performance-tuning)
8. [Scaling Strategies](#8-scaling-strategies)
9. [NoSQL Overview](#9-nosql-overview)
10. [Interview Notes](#10-interview-notes)
11. [Summary](#11-summary)
12. [References](#12-references)

---

## 1. SQL Deep Dive

**Definition:**
SQL (Structured Query Language) is the standard language for querying and manipulating relational databases. Every Java backend developer must understand SQL deeply — ORM generates SQL, and bad SQL kills performance.

### 1.1 Essential Queries for Backend Developers

```sql
-- Joins (most asked in interviews)
SELECT u.name, COUNT(o.id) AS order_count, SUM(o.total) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.status = 'ACTIVE'
GROUP BY u.id, u.name
HAVING SUM(o.total) > 1000
ORDER BY total_spent DESC
LIMIT 20;

-- Window Functions (ranking, running totals)
SELECT name, department, salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank,
    SUM(salary) OVER (PARTITION BY department) AS dept_total
FROM employees;

-- Common Table Expressions (CTE)
WITH monthly_revenue AS (
    SELECT DATE_TRUNC('month', created_at) AS month,
        SUM(total) AS revenue
    FROM orders WHERE status = 'COMPLETED'
    GROUP BY DATE_TRUNC('month', created_at)
)
SELECT month, revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month,
    round((revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0
        / LAG(revenue) OVER (ORDER BY month), 2) AS growth_pct
FROM monthly_revenue;

-- Upsert (INSERT ... ON CONFLICT)
INSERT INTO user_preferences (user_id, theme, language)
VALUES (42, 'dark', 'en')
ON CONFLICT (user_id)
DO UPDATE SET theme = EXCLUDED.theme, language = EXCLUDED.language;
```

---

## 2. Normalization

**Definition:**
Database normalization is the process of organizing tables to minimize data redundancy and dependency. Each normal form (1NF → 5NF) adds stricter rules.

| Normal Form | Rule | Example Violation |
|-------------|------|-------------------|
| 1NF | Atomic values, no repeating groups | `phone: "123,456"` (comma-separated) |
| 2NF | 1NF + no partial dependencies | Non-key column depends on part of composite key |
| 3NF | 2NF + no transitive dependencies | `zip_code → city` (city depends on zip, not PK) |
| BCNF | Every determinant is a candidate key | Rare — used for edge cases |

**Backend perspective:**
- Normalize to 3NF for transactional data (orders, users)
- Denormalize for read-heavy data (dashboards, reports, search)
- In practice: 3NF for writes + materialized views or caches for reads

---

## 3. Indexing

**Definition:**
An index is a data structure (B-Tree, Hash, GIN, etc.) that speeds up data retrieval at the cost of extra storage and slower writes. It's like a book's table of contents — instead of reading every page, you jump to the right one.

**Why it exists:**
Without indexes, the database performs a full table scan (reading every row) for each query. On a table with millions of rows, this can take seconds. Indexes reduce this to milliseconds.

### 3.1 Index Types

| Type | Description | Use Case |
|------|-------------|----------|
| B-Tree (default) | Balanced tree, supports `=`, `<`, `>`, `BETWEEN`, `LIKE 'prefix%'` | Most queries |
| Hash | Hash table, supports only `=` | Exact match lookups |
| GIN (Generalized Inverted) | Inverted index for multi-value columns | Full-text search, JSON, arrays |
| GiST | Generalized search tree | Geospatial queries |
| Composite | Index on multiple columns | Multi-column WHERE/ORDER BY |
| Covering (INCLUDE) | Stores extra columns in index | Avoids table lookup |
| Partial | Index on a subset of rows | `WHERE status = 'ACTIVE'` |

### 3.2 Indexing Best Practices

```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (column order matters!)
-- Leftmost prefix rule: can use (status), (status, created_at), but NOT (created_at) alone
CREATE INDEX idx_orders_status_date ON orders(status, created_at);

-- Partial index (only index active users)
CREATE INDEX idx_users_active_email ON users(email) WHERE status = 'ACTIVE';

-- Covering index (includes extra columns to avoid table lookup)
CREATE INDEX idx_orders_covering ON orders(user_id) INCLUDE (total, status);
```

**Key rules:**
- Index columns in WHERE, JOIN, ORDER BY, GROUP BY clauses
- Column order matters in composite indexes (leftmost prefix rule)
- Don't over-index — each index slows writes
- Monitor unused indexes and remove them

---

## 4. Query Optimization

**Definition:**
Query optimization is the process of rewriting SQL queries and tuning database configuration to minimize execution time and resource usage. The EXPLAIN plan is your primary tool.

### 4.1 Reading EXPLAIN Plans

```sql
EXPLAIN ANALYZE SELECT u.name, COUNT(o.id)
FROM users u JOIN orders o ON u.id = o.user_id
WHERE u.status = 'ACTIVE'
GROUP BY u.name;

-- Output shows:
-- Seq Scan (full table scan — bad for large tables)
-- Index Scan (uses index — good)
-- Bitmap Index Scan (combination approach)
-- Nested Loop / Hash Join / Merge Join (join strategies)
-- actual time, rows, loops
```

### 4.2 Common Optimization Techniques

| Problem | Fix |
|---------|-----|
| Sequential scan on large table | Add appropriate index |
| `SELECT *` | Select only needed columns |
| N+1 queries (from ORM) | Use JOIN FETCH or batch queries |
| Missing composite index | Create composite index matching query pattern |
| Function on indexed column (`WHERE UPPER(email) = ...`) | Use functional index or store normalized |
| Correlated subquery | Rewrite as JOIN |
| Large OFFSET pagination | Switch to cursor/keyset pagination |
| Sorting without index | Create index matching ORDER BY |

---

## 5. Transactions & Isolation Levels

**Definition:**
A transaction is a sequence of operations that are executed as a single atomic unit — either all succeed (commit) or all fail (rollback). ACID properties guarantee reliability.

**ACID:**
- **Atomicity:** All or nothing
- **Consistency:** Database moves from one valid state to another
- **Isolation:** Concurrent transactions don't interfere
- **Durability:** Committed data survives crashes

### 5.1 Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|-----------|-------------------|-------------|-------------|
| READ UNCOMMITTED | ✅ | ✅ | ✅ | Fastest |
| READ COMMITTED | ❌ | ✅ | ✅ | Default (PostgreSQL) |
| REPEATABLE READ | ❌ | ❌ | ✅ | Default (MySQL) |
| SERIALIZABLE | ❌ | ❌ | ❌ | Slowest |

### 5.2 Locking

```sql
-- Pessimistic locking (SELECT FOR UPDATE)
SELECT * FROM accounts WHERE id = 42 FOR UPDATE;
-- Other transactions wait until this one commits

-- Optimistic locking (version column — handled by JPA @Version)
UPDATE products SET price = 99.99, version = version + 1
WHERE id = 42 AND version = 5;
-- If version changed, update affects 0 rows → retry
```

**When to use:**
- **Optimistic:** Low contention (most web apps) — check version at update time
- **Pessimistic:** High contention (inventory, financial) — lock rows during transaction

---

## 6. Connection Pooling (HikariCP)

**Definition:**
Connection pooling maintains a pool of reusable database connections. Instead of creating a new connection for each query (expensive — ~50ms), you borrow one from the pool and return it after use.

**Why it exists:**
Creating a DB connection involves TCP handshake, authentication, and resource allocation. Doing this per query under high load would cripple performance. Pooling amortizes the cost.

**Real-life analogy:**
A car rental agency. Instead of buying a new car for each customer, the agency maintains a fleet (pool). Customers rent (borrow) and return cars (connections).

**Configuration:**
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20           # Max connections in pool
      minimum-idle: 5                 # Min idle connections
      idle-timeout: 300000            # 5 min — close idle connections
      max-lifetime: 1800000           # 30 min — recycle connections
      connection-timeout: 30000       # 30 sec — wait for connection
      leak-detection-threshold: 60000 # 1 min — detect connection leaks
```

**Pool sizing formula (from HikariCP wiki):**
```
pool_size = (core_count * 2) + effective_spindle_count
Example: 4-core server with SSD = (4 * 2) + 1 = 9-10 connections
```

---

## 7. Database Performance Tuning

| Technique | Impact |
|-----------|--------|
| Add missing indexes | 10x-1000x query speedup |
| Use connection pooling | Eliminate connection overhead |
| Optimize slow queries (EXPLAIN) | Reduce response times |
| Use read replicas for reads | Reduce load on primary |
| Batch inserts/updates | Reduce round trips |
| Use database-level pagination (LIMIT/OFFSET or keyset) | Prevent memory issues |
| Avoid `SELECT *` | Reduce data transfer |
| Cache frequently-read data (Redis) | Reduce DB load |
| Use prepared statements | Prevent SQL injection + plan reuse |
| Monitor with pg_stat_statements / slow query log | Find bottlenecks |

---

## 8. Scaling Strategies

### 8.1 Vertical Scaling
More CPU, RAM, faster disk for the existing server. Simple but has a ceiling.

### 8.2 Read Replicas
Route read queries to replicas, writes to primary. Spring Boot example:
```java
@Transactional(readOnly = true) // Routes to read replica
public List<UserDto> findAll() { ... }

@Transactional // Routes to primary
public UserDto create(CreateUserRequest req) { ... }
```

### 8.3 Sharding
Partition data across multiple databases based on a shard key (e.g., `user_id % shard_count`).

### 8.4 Partitioning
Split a large table into smaller partitions within the same database:
```sql
CREATE TABLE orders (
    id BIGSERIAL, user_id BIGINT, created_at TIMESTAMP, total DECIMAL
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2025 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
CREATE TABLE orders_2026 PARTITION OF orders
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
```

---

## 9. NoSQL Overview

| Type | Database | Use Case |
|------|----------|----------|
| Document | MongoDB | Flexible schemas, content management |
| Key-Value | Redis | Caching, sessions, rate limiting |
| Column-Family | Cassandra | Time-series, high write throughput |
| Graph | Neo4j | Social networks, recommendations |
| Search Engine | Elasticsearch | Full-text search, log analysis |

**When to consider NoSQL:**
- Flexible/evolving schema requirements
- Extreme scale (millions of writes/sec)
- Specific access patterns (graph traversal, full-text search)
- Caching layer alongside relational DB

---

## 10. Interview Notes

1. **What is an index and when should you create one?** — B-Tree data structure for fast lookups. Create on WHERE, JOIN, ORDER BY columns. Don't over-index (slows writes).
2. **Explain ACID properties.** — Atomicity, Consistency, Isolation, Durability. Foundation of relational databases.
3. **What are isolation levels? What's the default in PostgreSQL?** — READ_COMMITTED. Prevents dirty reads but allows non-repeatable reads and phantoms.
4. **Optimistic vs Pessimistic locking?** — Optimistic: version check at update (low contention). Pessimistic: locks rows during transaction (high contention).
5. **How do you optimize a slow query?** — EXPLAIN ANALYZE → check for seq scans → add indexes → rewrite query → check N+1.
6. **What is connection pooling? Why is it important?** — Reuses DB connections. Without it, creating connections per query is ~50ms overhead.
7. **How would you scale a database?** — Read replicas → partitioning → sharding. Cache frequently-read data.

---

## 11. Summary

| Topic | Key Takeaway |
|-------|-------------|
| SQL | Window functions, CTEs, JOINs — essential for backend |
| Normalization | 3NF for transactions, denormalize for reads |
| Indexing | B-Tree default; composite index order matters |
| Query Optimization | EXPLAIN ANALYZE is your best friend |
| Transactions | ACID; choose isolation level based on data sensitivity |
| Connection Pooling | HikariCP default; pool size ≈ (cores × 2) + spindles |
| Scaling | Read replicas → partitioning → sharding |

---

## 12. References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Use The Index, Luke](https://use-the-index-luke.com/)
- [HikariCP Wiki — Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)
- [Baeldung — Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)
- [Vlad Mihalcea — Database Tutorials](https://vladmihalcea.com/)
- [MySQL Documentation](https://dev.mysql.com/doc/)

---

> **Previous:** [← 06 — Spring Security](./06_Spring_Security.md)
> **Next:** [08 — Backend Architecture & System Design →](./08_Backend_Architecture_System_Design.md)
