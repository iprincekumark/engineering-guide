# Spring Data — Interview Questions & Answers

---

## 1. What is Spring Data?

**Spring Data** is a **Spring ecosystem project that provides a unified, consistent, and simplified data access layer** for various data stores — relational databases, NoSQL databases, in-memory stores, search engines, and more. It eliminates boilerplate repository code by providing ready-made implementations through interfaces.

| Aspect | Description |
|---|---|
| **What it is** | An umbrella project with multiple sub-modules for different data stores |
| **What it is NOT** | A replacement for JPA/JDBC — it sits **on top of** them |
| **Core Idea** | Define an interface → Spring generates the implementation automatically |
| **Maintained by** | Spring (Pivotal / VMware) |
| **Key Feature** | Derived Query Methods — write zero SQL, just name your method correctly |

### Spring Data Sub-Modules:

| Module | Data Store |
|---|---|
| **Spring Data JPA** | Relational DB via JPA/Hibernate |
| **Spring Data JDBC** | Relational DB via plain JDBC |
| **Spring Data MongoDB** | MongoDB (NoSQL) |
| **Spring Data Redis** | Redis (Cache/Key-Value Store) |
| **Spring Data Elasticsearch** | Elasticsearch (Search Engine) |
| **Spring Data Cassandra** | Apache Cassandra (Wide-Column Store) |

**Real World Analogy:** Spring Data is like a **universal TV remote**. Instead of having a different remote (code) for every TV brand (database), one universal remote (Spring Data) works across all of them with the same buttons (API).

---

## 2. Need for Spring Data?

**Without Spring Data**, developers had to write repetitive, low-level boilerplate code for every entity — same CRUD operations copy-pasted across every repository class. Spring Data eliminates this entirely.

### Without Spring Data (Manual EntityManager — Boilerplate):
```java
@Repository
public class UserDaoImpl {

    @PersistenceContext
    private EntityManager entityManager;

    // Every single repo needs these methods repeated!
    public void save(User user) {
        entityManager.persist(user);
    }

    public User findById(int id) {
        return entityManager.find(User.class, id);
    }

    public List<User> findAll() {
        return entityManager
            .createQuery("SELECT u FROM User u", User.class)
            .getResultList();
    }

    public void update(User user) {
        entityManager.merge(user);
    }

    public void delete(int id) {
        User user = entityManager.find(User.class, id);
        entityManager.remove(user);
    }

    // Same 30+ lines repeated for Product, Order, Department...
}
```

### With Spring Data (Zero Boilerplate):
```java
// That's it. Spring generates ALL CRUD operations automatically at runtime.
public interface UserRepository extends JpaRepository<User, Integer> {
    // Nothing to write — save(), findById(), findAll(), delete() all available!
}
```

### Problem → Solution Table:

| Problem Without Spring Data | How Spring Data Solves It |
|---|---|
| Repeated `persist()`, `merge()`, `remove()` code per entity | Single `JpaRepository` interface provides all CRUD |
| Manual JPQL for every custom query | Derived query methods from method names |
| Manual pagination logic every time | Built-in `findAll(Pageable)` support |
| Different API for different DB types | Unified `Repository` abstraction across all stores |
| Sorting requires verbose `CriteriaQuery` | Built-in `findAll(Sort)` support |
| Writing `EntityManager` boilerplate everywhere | Spring injects a fully working implementation |

**Real World Analogy:** Without Spring Data, it's like writing a **withdrawal slip by hand** every time at a bank. With Spring Data, you use an **ATM** — the same buttons work for every operation, no paperwork needed.

---

## 3. What is Spring Data JPA?

**Spring Data JPA** is a **Spring Data sub-module** that builds on top of JPA and Hibernate to provide **automatic repository implementations, derived query methods, and pagination support** for relational databases — without writing a single line of SQL or JPQL for standard operations.

| Aspect | Description |
|---|---|
| **Built on top of** | JPA (spec) + Hibernate (implementation) |
| **Key Interface** | `JpaRepository<Entity, ID>` |
| **Core Feature** | Auto-generates SQL/JPQL from method names at runtime |
| **Dependency** | `spring-boot-starter-data-jpa` |

### Dependency:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <!-- Includes: Spring Data JPA + Hibernate + Spring JDBC + Transaction Management -->
</dependency>
```

### Setup — Entity + Repository:
```java
// 1. Entity
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;
    private String email;
    private String city;

    // constructors, getters, setters
}

// 2. Repository — extend JpaRepository, get everything for free
public interface UserRepository extends JpaRepository<User, Integer> {

    // Derived query — Spring generates: SELECT * FROM users WHERE name = ?
    List<User> findByName(String name);

    // Derived query — Spring generates: SELECT * FROM users WHERE city = ? AND email = ?
    List<User> findByCityAndEmail(String city, String email);

    // Custom JPQL query
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain%")
    List<User> findByEmailDomain(@Param("domain") String domain);

    // Custom Native SQL query
    @Query(value = "SELECT * FROM users WHERE city = :city", nativeQuery = true)
    List<User> findByCityNative(@Param("city") String city);
}
```

### Using the Repository in Service:
```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public User createUser(User user) {
        return userRepository.save(user);           // INSERT or UPDATE
    }

    public Optional<User> getUserById(int id) {
        return userRepository.findById(id);         // SELECT by PK
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();            // SELECT all
    }

    public void deleteUser(int id) {
        userRepository.deleteById(id);             // DELETE by PK
    }

    public List<User> getUsersByCity(String city) {
        return userRepository.findByCity(city);    // Derived query
    }
}
```

### Derived Query Method Keywords:

| Keyword | Example Method | Generated SQL Condition |
|---|---|---|
| `findBy` | `findByName(String name)` | `WHERE name = ?` |
| `And` | `findByNameAndCity(...)` | `WHERE name = ? AND city = ?` |
| `Or` | `findByNameOrEmail(...)` | `WHERE name = ? OR email = ?` |
| `Like` | `findByNameLike(String pattern)` | `WHERE name LIKE ?` |
| `Containing` | `findByNameContaining(String s)` | `WHERE name LIKE %?%` |
| `StartingWith` | `findByNameStartingWith(String s)` | `WHERE name LIKE ?%` |
| `OrderBy` | `findByNameOrderByIdAsc(...)` | `ORDER BY id ASC` |
| `GreaterThan` | `findByAgeGreaterThan(int age)` | `WHERE age > ?` |
| `Between` | `findByAgeBetween(int a, int b)` | `WHERE age BETWEEN ? AND ?` |
| `IsNull` | `findByEmailIsNull()` | `WHERE email IS NULL` |

**Real World Analogy:** Spring Data JPA is like a **smart filing assistant**. You just say *"get me all users from Bangalore"* — you don't tell it how to open the cabinet, search the folders, or return results. It figures all that out from the method name alone.

---

## 4. What is CrudRepository?

**`CrudRepository`** is a **Spring Data core interface** that provides **basic CRUD (Create, Read, Update, Delete) operations** for any entity. It is the root-level repository interface from which all others extend — defining the fundamental data access contract.

```java
// Spring Data's CrudRepository interface (simplified):
public interface CrudRepository<T, ID> extends Repository<T, ID> {

    <S extends T> S save(S entity);              // INSERT or UPDATE (by ID presence)
    <S extends T> Iterable<S> saveAll(Iterable<S> entities); // Batch INSERT/UPDATE

    Optional<T> findById(ID id);                 // SELECT WHERE id = ?
    boolean existsById(ID id);                   // SELECT COUNT WHERE id = ?
    Iterable<T> findAll();                       // SELECT all rows
    Iterable<T> findAllById(Iterable<ID> ids);   // SELECT WHERE id IN (...)

    long count();                                // SELECT COUNT(*)

    void deleteById(ID id);                      // DELETE WHERE id = ?
    void delete(T entity);                       // DELETE given entity
    void deleteAllById(Iterable<ID> ids);        // DELETE WHERE id IN (...)
    void deleteAll();                            // DELETE all rows
}
```

### Usage Example:
```java
// 1. Define Repository
public interface ProductRepository extends CrudRepository<Product, Integer> {

    // CrudRepository gives: save, findById, findAll, delete, count — all free!

    // Add custom derived query on top:
    List<Product> findByCategory(String category);
}

// 2. Entity
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;
    private String category;
    private double price;

    // constructors, getters, setters
}

// 3. Service using CrudRepository
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Product save(Product product) {
        return productRepository.save(product);     // INSERT (id=0) or UPDATE (id set)
    }

    public Optional<Product> findById(int id) {
        return productRepository.findById(id);      // Returns Optional — null safe!
    }

    public Iterable<Product> findAll() {
        return productRepository.findAll();
    }

    public long count() {
        return productRepository.count();
    }

    public void deleteById(int id) {
        productRepository.deleteById(id);
    }

    public boolean exists(int id) {
        return productRepository.existsById(id);
    }
}
```

### CrudRepository vs JpaRepository:

| Feature | `CrudRepository` | `JpaRepository` |
|---|---|---|
| **Basic CRUD** | ✅ Yes | ✅ Yes (inherited) |
| **Returns** | `Iterable<T>` | `List<T>` (more useful) |
| **Pagination** | ❌ No | ✅ Yes |
| **Sorting** | ❌ No | ✅ Yes |
| **Batch operations** | ❌ No | ✅ `saveAll()`, `deleteInBatch()` |
| **Flush / EntityManager** | ❌ No | ✅ `flush()`, `saveAndFlush()` |
| **Use when** | Simple apps, non-relational DBs | Spring Data JPA / relational DBs |

> **Best Practice:** For Spring Data JPA with relational databases, prefer `JpaRepository` over `CrudRepository` — it extends `PagingAndSortingRepository` and `CrudRepository` both, giving you everything.

**Real World Analogy:** `CrudRepository` is like a **basic Swiss Army knife** — it has the essential tools (save, find, delete) that cover 80% of everyday needs. `JpaRepository` is the **premium version** with extra attachments.

---

## 5. What is PagingAndSortingRepository?

**`PagingAndSortingRepository`** is a **Spring Data interface that extends `CrudRepository`** and adds two powerful capabilities — **pagination** (splitting large result sets into pages) and **sorting** (ordering results by one or more fields). It prevents loading thousands of rows into memory at once.

```java
// Spring Data's PagingAndSortingRepository (simplified):
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

    Iterable<T> findAll(Sort sort);          // Fetch all rows — sorted
    Page<T> findAll(Pageable pageable);      // Fetch one page of results
}
```

### Key Classes:

| Class / Interface | Purpose |
|---|---|
| `Pageable` | Interface — holds page number, page size, and sort info |
| `PageRequest` | Concrete implementation of `Pageable` — use to create page requests |
| `Page<T>` | Result container — holds the data + total count + metadata |
| `Sort` | Defines sort direction (ASC/DESC) and field name(s) |

### Usage Example:
```java
// 1. Repository
public interface EmployeeRepository
        extends PagingAndSortingRepository<Employee, Integer> {

    // Custom paginated query — Spring Data handles the rest
    Page<Employee> findByDepartment(String department, Pageable pageable);
}

// 2. Entity
@Entity
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;
    private String department;
    private double salary;

    // constructors, getters, setters
}

// 3. Service demonstrating Pagination + Sorting
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;

    // --- SORTING only ---
    public Iterable<Employee> getAllSorted() {
        Sort sort = Sort.by(Sort.Direction.ASC, "name");     // ORDER BY name ASC
        return employeeRepository.findAll(sort);
    }

    // --- PAGINATION only ---
    public Page<Employee> getPage(int pageNo, int pageSize) {
        Pageable pageable = PageRequest.of(pageNo, pageSize);
        // pageNo is 0-indexed: page 0 = first page, page 1 = second page
        return employeeRepository.findAll(pageable);
    }

    // --- PAGINATION + SORTING together ---
    public Page<Employee> getPageSorted(int pageNo, int pageSize) {
        Pageable pageable = PageRequest.of(
            pageNo,
            pageSize,
            Sort.by(Sort.Direction.DESC, "salary")    // ORDER BY salary DESC
        );
        return employeeRepository.findAll(pageable);
    }

    // --- USING Page<T> metadata ---
    public void printPageInfo(int pageNo, int pageSize) {
        Page<Employee> page = employeeRepository.findAll(
            PageRequest.of(pageNo, pageSize)
        );

        System.out.println("Data:           " + page.getContent());      // List of employees
        System.out.println("Current Page:   " + page.getNumber());       // 0, 1, 2 ...
        System.out.println("Page Size:      " + page.getSize());         // 10
        System.out.println("Total Elements: " + page.getTotalElements());// e.g. 250
        System.out.println("Total Pages:    " + page.getTotalPages());   // e.g. 25
        System.out.println("Is First Page:  " + page.isFirst());         // true/false
        System.out.println("Is Last Page:   " + page.isLast());          // true/false
        System.out.println("Has Next:       " + page.hasNext());         // true/false
    }
}
```

### How `Page<T>` Works Internally:

```
Total employees = 100, pageSize = 10

PageRequest.of(0, 10) → Page 1: employees 1–10    → SQL: LIMIT 10 OFFSET 0
PageRequest.of(1, 10) → Page 2: employees 11–20   → SQL: LIMIT 10 OFFSET 10
PageRequest.of(2, 10) → Page 3: employees 21–30   → SQL: LIMIT 10 OFFSET 20
...
PageRequest.of(9, 10) → Page 10: employees 91–100 → SQL: LIMIT 10 OFFSET 90
```

### Repository Hierarchy (Full Picture):

```
Repository<T, ID>                                  ← Marker interface (no methods)
    └── CrudRepository<T, ID>                      ← save, findById, findAll, delete, count
            └── PagingAndSortingRepository<T, ID>  ← + findAll(Sort), findAll(Pageable)
                    └── JpaRepository<T, ID>       ← + flush, saveAndFlush, deleteInBatch
```

**Real World Analogy:** `PagingAndSortingRepository` is like a **library catalog system**. Instead of dumping all 10,000 books on the table (loading all rows), you ask: *"Show me page 3 of Science books, sorted by title A→Z"* — you get exactly 20 books, sorted, with info on how many total pages remain.

---

*Happy Learning Spring Data! 🗄️*
