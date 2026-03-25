# Database (JDBC, JPA, Hibernate) - Interview Questions & Answers

---

## 1. What is Spring JDBC?

**Spring JDBC** is a Spring module that provides a **thin abstraction layer over plain JDBC (Java Database Connectivity)** to eliminate boilerplate code like opening/closing connections, handling `SQLException`, creating statements, and iterating `ResultSet`. It keeps the developer in full control of SQL while removing repetitive infrastructure code.

| Plain JDBC (Without Spring) | Spring JDBC (With Spring) |
|---|---|
| Manually open connection | Spring manages connection |
| Manually create `PreparedStatement` | Spring creates statement |
| Manually iterate `ResultSet` | Spring maps results via `RowMapper` |
| Manually handle `SQLException` (checked) | Spring converts to unchecked `DataAccessException` |
| Manually close connection in `finally` | Spring auto-closes resources |
| ~30 lines for a simple query | ~3 lines with `JdbcTemplate` |

**Real World Analogy:** Plain JDBC is like cooking from scratch — buy ingredients, chop, boil, season. Spring JDBC is like a meal-prep kit — ingredients are prepped, you just follow simple steps.

---

## 2. What is JdbcTemplate?

**`JdbcTemplate`** is the **central class of Spring JDBC** that simplifies all JDBC operations. It handles connection management, statement creation, exception translation, and resource cleanup automatically. You provide the SQL and a `RowMapper` — Spring does everything else.

### Common JdbcTemplate Methods:

| Method | Purpose |
|---|---|
| `queryForObject()` | Fetch a single row / scalar value |
| `query()` | Fetch a list of rows |
| `update()` | Execute INSERT / UPDATE / DELETE |
| `execute()` | Execute DDL (CREATE TABLE, etc.) |
| `batchUpdate()` | Execute batch INSERT / UPDATE |

### Setup:
```java
// application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
```

```java
@Repository
public class UserRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    // INSERT
    public int save(User user) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        return jdbcTemplate.update(sql, user.getName(), user.getEmail());
    }

    // SELECT single row
    public User findById(int id) {
        String sql = "SELECT * FROM users WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, new UserRowMapper(), id);
    }

    // SELECT all rows
    public List<User> findAll() {
        String sql = "SELECT * FROM users";
        return jdbcTemplate.query(sql, new UserRowMapper());
    }

    // SELECT scalar value
    public int countUsers() {
        String sql = "SELECT COUNT(*) FROM users";
        return jdbcTemplate.queryForObject(sql, Integer.class);
    }

    // DELETE
    public int deleteById(int id) {
        String sql = "DELETE FROM users WHERE id = ?";
        return jdbcTemplate.update(sql, id);
    }
}
```

---

## 3. What is RowMapper?

**`RowMapper`** is a functional interface in Spring JDBC used to **map each row of a `ResultSet` to a Java object**. It is called once per row and converts the raw JDBC result into a domain object. You implement it to define how columns map to object fields.

```java
public interface RowMapper<T> {
    T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
```

### Custom RowMapper:
```java
public class UserRowMapper implements RowMapper<User> {

    @Override
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setName(rs.getString("name"));
        user.setEmail(rs.getString("email"));
        user.setAge(rs.getInt("age"));
        return user;
    }
}
```

### Using Lambda (Java 8+):
```java
public List<User> findAll() {
    String sql = "SELECT * FROM users";
    return jdbcTemplate.query(sql, (rs, rowNum) -> {
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setName(rs.getString("name"));
        user.setEmail(rs.getString("email"));
        return user;
    });
}
```

### BeanPropertyRowMapper (Auto-mapping):
```java
// Auto-maps column names to bean property names (by convention)
public List<User> findAll() {
    return jdbcTemplate.query("SELECT * FROM users",
        new BeanPropertyRowMapper<>(User.class));
    // Column "user_name" → field "userName" (camelCase conversion)
}
```

---

## 4. What is JPA?

**JPA (Java Persistence API)** is a **Java EE / Jakarta EE specification** (not an implementation) that defines a **standard way to map Java objects (entities) to relational database tables** and perform database operations using objects instead of SQL. JPA is part of the `jakarta.persistence` package.

| Aspect | Description |
|---|---|
| **What it is** | A specification / standard (set of interfaces and annotations) |
| **What it is NOT** | An implementation — you need a JPA provider |
| **Core Concept** | ORM (Object-Relational Mapping) — Java class ↔ DB table |
| **Key Annotations** | `@Entity`, `@Table`, `@Id`, `@Column`, `@OneToMany`, etc. |
| **Query Language** | JPQL (Java Persistence Query Language) — object-oriented SQL |
| **Popular Providers** | Hibernate (most popular), EclipseLink, OpenJPA |

**Real World Analogy:** JPA is like a **job description** (what needs to be done). Hibernate is the **employee** who actually does the work.

---

## 5. What is Hibernate?

**Hibernate** is the **most popular implementation of the JPA specification**. It is an ORM (Object-Relational Mapping) framework that automatically handles the conversion between Java objects and database tables, generates SQL queries, manages caching, and handles database-specific dialect differences.

| Aspect | JPA | Hibernate |
|---|---|---|
| **Type** | Specification (interfaces) | Implementation (concrete classes) |
| **Package** | `jakarta.persistence` | `org.hibernate` |
| **SQL Generation** | Defines standard | Generates actual SQL |
| **Extra Features** | Standard only | Caching (L1/L2), HQL, Criteria API, Batch processing |
| **Portability** | Switch provider without changing JPA code | Hibernate-specific features tie you to Hibernate |

```xml
<!-- Spring Boot auto-configures Hibernate when this is added -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <!-- Includes: Hibernate + Spring Data JPA + JDBC + Transaction Management -->
</dependency>
```

---

## 6. Define Entity in JPA?

An **Entity** is a **plain Java class (POJO) annotated with `@Entity`** that represents a table in a relational database. Each instance of the entity corresponds to one row in the table, and each field corresponds to a column.

### Rules for an Entity:
- Must be annotated with `@Entity`
- Must have a **no-argument constructor** (public or protected)
- Must have a **primary key** field annotated with `@Id`
- Must not be `final`

```java
@Entity                          // Marks this class as a JPA entity (maps to a DB table)
@Table(name = "users")           // Optional: specify table name (default = class name)
public class User {

    @Id                          // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment
    private int id;

    @Column(name = "full_name", nullable = false, length = 100)
    private String name;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(name = "date_of_birth")
    @Temporal(TemporalType.DATE)
    private Date birthDate;

    @Transient                   // Not mapped to any column — ignored by JPA
    private String tempField;

    // No-arg constructor (required by JPA)
    public User() {}

    // Parameterized constructor, getters, setters
}
```

### Key JPA Column Annotations:

| Annotation | Purpose |
|---|---|
| `@Entity` | Marks class as JPA entity |
| `@Table(name="...")` | Specifies table name |
| `@Id` | Marks primary key field |
| `@GeneratedValue` | Auto-generates primary key value |
| `@Column` | Customizes column mapping (name, nullable, length) |
| `@Transient` | Excludes field from persistence |
| `@Temporal` | Specifies date/time type for `java.util.Date` |
| `@Lob` | Maps large objects (CLOB/BLOB) |
| `@Enumerated` | Maps Java enum to DB column |

---

## 7. What is Entity Manager?

**`EntityManager`** is the **primary JPA interface for performing all database operations** on entities. It is the bridge between the Java application and the database, responsible for persisting, finding, merging, removing entities, and executing JPQL queries.

| Method | Operation | Description |
|---|---|---|
| `persist(entity)` | INSERT | Save a new entity to DB |
| `find(Class, id)` | SELECT by PK | Find entity by primary key |
| `merge(entity)` | UPDATE | Update a detached entity |
| `remove(entity)` | DELETE | Delete an entity |
| `createQuery(jpql)` | SELECT (JPQL) | Execute JPQL query |
| `flush()` | SYNC | Sync persistence context to DB |
| `detach(entity)` | — | Remove entity from persistence context |
| `contains(entity)` | — | Check if entity is managed |

```java
@Repository
public class UserDao {

    @PersistenceContext     // Injects the EntityManager managed by Spring
    private EntityManager entityManager;

    // INSERT
    public void save(User user) {
        entityManager.persist(user);
    }

    // SELECT by ID
    public User findById(int id) {
        return entityManager.find(User.class, id);
    }

    // UPDATE
    public User update(User user) {
        return entityManager.merge(user);
    }

    // DELETE
    public void delete(int id) {
        User user = entityManager.find(User.class, id);
        entityManager.remove(user);
    }

    // JPQL Query
    public List<User> findByName(String name) {
        return entityManager
            .createQuery("SELECT u FROM User u WHERE u.name = :name", User.class)
            .setParameter("name", name)
            .getResultList();
    }
}
```

---

## 8. What is Persistence Context?

**Persistence Context** is a **first-level cache / managed environment** maintained by the `EntityManager` that tracks all entity instances during a transaction. Any entity retrieved or persisted through the `EntityManager` becomes **"managed"** — the persistence context automatically detects changes and syncs them to the database at the end of the transaction (without explicit `update()` calls).

### Entity Lifecycle States:

| State | Description | How to reach it |
|---|---|---|
| **New / Transient** | Object created but not associated with any persistence context | `new User()` |
| **Managed / Persistent** | Associated with persistence context — changes auto-tracked | `persist()`, `find()`, `merge()` |
| **Detached** | Was managed but persistence context was closed | After transaction ends or `detach()` |
| **Removed** | Marked for deletion — deleted at next flush/commit | `remove()` |

```java
@Transactional
public void updateUserName(int id, String newName) {
    User user = entityManager.find(User.class, id); // State: MANAGED
    user.setName(newName); // Just set the value — NO explicit update() needed!
    // Persistence Context detects the change and issues UPDATE SQL at commit
}
```

**Real World Analogy:** Persistence Context is like a **shopping cart**. You add/modify items (entities) in the cart (context). When you checkout (commit transaction), all changes are applied to the database (store inventory) at once.

---

## 9. Mapping Relationships in JPA?

JPA supports mapping **associations between entities** that reflect the relationships between tables in the database. Relationships are mapped using annotations and can be **unidirectional** (one side knows about the other) or **bidirectional** (both sides know about each other).

| Concept | Description |
|---|---|
| **Owning Side** | The entity that holds the foreign key column in the DB — responsible for persisting the relationship |
| **Inverse Side** | The non-owning side — uses `mappedBy` to reference the owning side's field |
| **`mappedBy`** | Tells JPA "I am NOT the owner; the owner is the field named X in the other entity" |
| **`@JoinColumn`** | Specifies the foreign key column name |
| **`CascadeType`** | Defines which operations (PERSIST, MERGE, REMOVE, ALL) cascade from parent to child |
| **`FetchType`** | `LAZY` (load on demand) or `EAGER` (load immediately with parent) |

---

## 10. Types of Relationships?

| Relationship | Annotation | DB Representation | Example |
|---|---|---|---|
| **One-to-One** | `@OneToOne` | Foreign key in one table | User ↔ Passport |
| **One-to-Many** | `@OneToMany` | Foreign key in "many" table | Department → Employees |
| **Many-to-One** | `@ManyToOne` | Foreign key in "many" table | Employee → Department |
| **Many-to-Many** | `@ManyToMany` | Join/junction table | Student ↔ Course |

---

## 11. One-to-One Mapping?

A **One-to-One** relationship means one entity is associated with **exactly one instance** of another entity.

**Example:** One `User` has one `Address`.

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    @OneToOne(cascade = CascadeType.ALL)    // Operations on User cascade to Address
    @JoinColumn(name = "address_id")        // Foreign key column in "users" table
    private Address address;               // Owning side

    // getters and setters
}

@Entity
@Table(name = "addresses")
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String street;
    private String city;
    private String zipCode;

    @OneToOne(mappedBy = "address")         // Inverse side — "address" is field in User
    private User user;

    // getters and setters
}
```

**DB Structure:**
```
users table:        id | name  | address_id (FK)
addresses table:    id | street | city | zip_code
```

---

## 12. One-to-Many Mapping?

A **One-to-Many** relationship means one entity is associated with **multiple instances** of another entity.

**Example:** One `Department` has many `Employees`.

```java
@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    @OneToMany(mappedBy = "department",         // Inverse side
               cascade = CascadeType.ALL,
               fetch = FetchType.LAZY)          // Load employees only when accessed
    private List<Employee> employees = new ArrayList<>();

    // getters and setters
}

@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")         // FK column in employees table — OWNING SIDE
    private Department department;

    // getters and setters
}
```

**DB Structure:**
```
departments table:  id | name
employees table:    id | name | department_id (FK → departments.id)
```

**Usage:**
```java
@Transactional
public void createDepartmentWithEmployees() {
    Department dept = new Department();
    dept.setName("Engineering");

    Employee emp1 = new Employee();
    emp1.setName("Alice");
    emp1.setDepartment(dept);    // Set the owning side

    Employee emp2 = new Employee();
    emp2.setName("Bob");
    emp2.setDepartment(dept);

    dept.getEmployees().add(emp1);
    dept.getEmployees().add(emp2);

    entityManager.persist(dept); // Cascades to employees
}
```

---

## 13. Many-to-Many Mapping?

A **Many-to-Many** relationship means multiple instances of one entity are associated with multiple instances of another entity. This requires a **join table** in the database.

**Example:** Many `Students` can enroll in many `Courses`.

```java
@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",                    // Join table name
        joinColumns = @JoinColumn(name = "student_id"),      // FK to this entity
        inverseJoinColumns = @JoinColumn(name = "course_id") // FK to other entity
    )
    private List<Course> courses = new ArrayList<>();

    // getters and setters
}

@Entity
@Table(name = "courses")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String title;

    @ManyToMany(mappedBy = "courses")   // Inverse side
    private List<Student> students = new ArrayList<>();

    // getters and setters
}
```

**DB Structure:**
```
students table:        id | name
courses table:         id | title
student_course table:  student_id (FK) | course_id (FK)   ← Join table
```

---

## 14. Define DataSource?

**`DataSource`** is a standard Java interface (`javax.sql.DataSource` / `jakarta.sql.DataSource`) that represents a **factory for database connections**. It manages a pool of database connections so they can be reused efficiently instead of creating a new connection for every request.

```properties
# Spring Boot auto-configures DataSource from these properties:
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Connection Pool settings (HikariCP — default in Spring Boot)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
```

| DataSource Provider | Description |
|---|---|
| **HikariCP** | Default in Spring Boot — fastest, lightweight connection pool |
| **Apache DBCP2** | Apache's connection pool implementation |
| **c3p0** | Older, widely used connection pool |
| **Tomcat Pool** | Tomcat's built-in connection pool |

**Manual DataSource Bean (Java Config):**
```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        ds.setUsername("root");
        ds.setPassword("secret");
        ds.setMaximumPoolSize(10);
        return ds;
    }
}
```

**Real World Analogy:** DataSource is like a **taxi dispatch center**. Instead of every passenger buying their own car (new DB connection), they call the dispatch (connection pool) and get an available taxi (existing connection). Much more efficient.

---

## 15. Use of persistence.xml?

**`persistence.xml`** is the **standard JPA configuration file** located at `META-INF/persistence.xml`. It defines a **Persistence Unit** — which includes the JPA provider, database connection properties, entity classes, and other JPA settings. It was the traditional way to configure JPA before Spring Boot automated this.

```xml
<!-- src/main/resources/META-INF/persistence.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.1"
    xmlns="http://xmlns.jcp.org/xml/ns/persistence">

    <persistence-unit name="myPU" transaction-type="RESOURCE_LOCAL">

        <!-- JPA Provider -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <!-- Entity classes -->
        <class>com.example.entity.User</class>
        <class>com.example.entity.Department</class>

        <properties>
            <!-- DataSource -->
            <property name="javax.persistence.jdbc.url"
                      value="jdbc:mysql://localhost:3306/mydb"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="secret"/>
            <property name="javax.persistence.jdbc.driver"
                      value="com.mysql.cj.jdbc.Driver"/>

            <!-- Hibernate properties -->
            <property name="hibernate.dialect"
                      value="org.hibernate.dialect.MySQL8Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            <property name="hibernate.show_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

> **In Spring Boot**, `persistence.xml` is **NOT needed**. Spring Boot auto-configures the `EntityManagerFactory` and `DataSource` from `application.properties`. `persistence.xml` is only used in **plain JPA / Java EE** applications without Spring Boot.

---

## 16. Configure Entity Manager Factory?

**`EntityManagerFactory`** is the **factory for `EntityManager` instances**. It is expensive to create (reads all entity mappings at startup), so it is created **once per application** and reused throughout the lifecycle.

### Spring Boot (Auto-configured — Zero Config):
```properties
# application.properties — Spring Boot auto-creates EntityManagerFactory
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

### Manual Java Configuration (Non-Boot):
```java
@Configuration
@EnableTransactionManagement
public class JpaConfig {

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(
            DataSource dataSource) {

        LocalContainerEntityManagerFactoryBean factory =
            new LocalContainerEntityManagerFactoryBean();

        factory.setDataSource(dataSource);
        factory.setPackagesToScan("com.example.entity"); // Scan for @Entity classes
        factory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());

        Properties jpaProperties = new Properties();
        jpaProperties.put("hibernate.hbm2ddl.auto", "update");
        jpaProperties.put("hibernate.show_sql", "true");
        jpaProperties.put("hibernate.dialect", "org.hibernate.dialect.MySQL8Dialect");
        factory.setJpaProperties(jpaProperties);

        return factory;
    }

    @Bean
    public PlatformTransactionManager transactionManager(
            EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}
```

### `spring.jpa.hibernate.ddl-auto` Options:

| Value | Behavior |
|---|---|
| `none` | No schema action |
| `validate` | Validates schema matches entities (no changes) |
| `update` | Updates schema to match entities (adds columns) |
| `create` | Drops and creates schema on startup |
| `create-drop` | Creates on startup, drops on shutdown |

---

## 17. Transaction Management?

**Transaction Management** ensures that a group of database operations either **all succeed (commit)** or **all fail together (rollback)**, maintaining data integrity. Spring provides **declarative transaction management** via `@Transactional`, eliminating the need for manual `begin/commit/rollback` code.

### Without Spring (Manual — verbose):
```java
EntityTransaction tx = entityManager.getTransaction();
try {
    tx.begin();
    entityManager.persist(user);
    entityManager.persist(account);
    tx.commit();    // Both succeed — commit
} catch (Exception e) {
    tx.rollback();  // Either fails — rollback both
}
```

### With Spring @Transactional (Declarative):
```java
@Service
public class BankService {

    @Autowired
    private AccountRepository accountRepo;

    @Transactional  // Spring handles begin, commit, rollback automatically
    public void transferMoney(int fromId, int toId, double amount) {
        Account from = accountRepo.findById(fromId);
        Account to = accountRepo.findById(toId);

        from.setBalance(from.getBalance() - amount); // Debit
        to.setBalance(to.getBalance() + amount);     // Credit

        accountRepo.save(from);
        accountRepo.save(to);
        // If any exception occurs above → Spring auto-rollbacks BOTH operations
    }
}
```

### @Transactional Key Attributes:

| Attribute | Options | Description |
|---|---|---|
| `propagation` | `REQUIRED` (default), `REQUIRES_NEW`, `SUPPORTS`, `NOT_SUPPORTED`, `NEVER`, `MANDATORY`, `NESTED` | How the transaction interacts with existing transactions |
| `isolation` | `DEFAULT`, `READ_UNCOMMITTED`, `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE` | Controls visibility of concurrent transactions |
| `rollbackFor` | Exception class | Rollback for specific exception types |
| `noRollbackFor` | Exception class | Don't rollback for specific exception types |
| `readOnly` | `true` / `false` | Optimizes read-only transactions (no dirty checking) |
| `timeout` | seconds | Max time for transaction before timeout |

### Propagation Types Explained:

| Propagation | Behavior |
|---|---|
| `REQUIRED` (default) | Join existing transaction; create new one if none exists |
| `REQUIRES_NEW` | Always create a new transaction; suspend existing one |
| `SUPPORTS` | Join existing transaction if present; run non-transactionally if not |
| `NOT_SUPPORTED` | Always run non-transactionally; suspend existing transaction |
| `NEVER` | Must run non-transactionally; throw exception if transaction exists |
| `MANDATORY` | Must run within existing transaction; throw exception if none |

### Enable Transaction Management:
```java
@Configuration
@EnableTransactionManagement  // Enable Spring's @Transactional support
public class AppConfig { }
```

> **Spring Boot auto-enables** `@EnableTransactionManagement` when `spring-boot-starter-data-jpa` is on the classpath — no explicit config needed.

**Real World Analogy:** A transaction is like a **bank transfer**. If you send $1000 from Account A to Account B — both the debit and credit must succeed together. If the credit fails, the debit must be rolled back. You can't have money disappear halfway.

---

*Happy Learning Spring Database (JDBC, JPA, Hibernate)! 🗄️*
