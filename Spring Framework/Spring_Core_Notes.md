# Spring Core - Interview Questions & Answers

---

## 1. What is Loose Coupling?

**Loose Coupling** is a design principle where classes have **minimal direct dependencies** on each other's concrete implementations. Instead of instantiating dependencies internally, classes depend on **abstractions (interfaces)**, making them independent of specific implementations. This is achieved in Spring through **Dependency Injection (DI)** and **Inversion of Control (IoC)**, where the **Spring IoC Container** is responsible for instantiating, managing, and injecting the dependent objects (beans) at runtime, rather than the class managing its own dependencies.

**Code Example:**
```java
// Interface (abstraction)
public interface Engine {
    void start();
}

// Implementations
public class PetrolEngine implements Engine {
    public void start() { System.out.println("Petrol Engine started!"); }
}

public class ElectricEngine implements Engine {
    public void start() { System.out.println("Electric Engine started!"); }
}

// Car depends on abstraction, not concrete class
public class Car {
    private Engine engine;
    public Car(Engine engine) { this.engine = engine; }
    public void drive() { engine.start(); }
}
```

**Real World Analogy:** A USB-C charger works with any USB-C device — phone, laptop, tablet. You can swap the device without changing the charger. That's loose coupling.

---

## 2. What is a Dependency?

A **dependency** is when one class **needs another class to perform its work**. If **Class A cannot function without Class B**, then **Class B is a dependency of Class A**. In Spring, these dependencies are managed by the **IoC Container**, which creates and injects them automatically using **Dependency Injection (DI)**, so the dependent class doesn't need to instantiate them using the `new` keyword.

**Code Example:**
```java
public class Car {
    // Engine is a DEPENDENCY of Car
    private Engine engine;
}
```

**Real World Analogy:** A lamp depends on electricity to work. Electricity is the dependency of the lamp.

---

## 3. What is IOC (Inversion of Control)?

**IoC (Inversion of Control)** is a design principle where the **control of creating and managing objects is transferred from the class itself to an external container (Spring IoC Container)**. Instead of a class creating its own dependencies using the `new` keyword, the **Spring Container takes that responsibility** and injects the required dependencies automatically.

| Without IoC (Normal Flow) | With IoC (Inverted Flow) |
|---|---|
| Your class creates its own objects | Spring Container creates objects for you |
| You are in control | Spring is in control |
| Tight Coupling | Loose Coupling |

**Code Example:**
```java
// Without IoC
public class Car {
    Engine engine = new Engine(); // Car controls creation
}

// With IoC
@Component
public class Car {
    @Autowired
    Engine engine; // Spring Container controls creation
}
```

**Real World Analogy:** Think of a restaurant. Without IoC, you go into the kitchen and cook your own food. With IoC, you sit at the table and the waiter brings the food to you. The kitchen (Spring Container) is in control.

> **IoC is a principle where the control of object creation, configuration, and lifecycle management is inverted from the class to the Spring IoC Container**, reducing tight coupling and improving modularity and testability of the application.

---

## 4. What is Dependency Injection?

**Dependency Injection (DI)** is the **implementation/technique of IoC** where the Spring IoC Container **injects the required dependencies into a class at runtime**, rather than the class creating them itself. It decouples the construction of objects from their usage, making the code more modular, testable, and maintainable. Spring supports three types of DI: **Constructor Injection**, **Setter Injection**, and **Field Injection**.

**Real World Analogy:** A hotel provides towels, soap, and toiletries to your room. You don't bring your own. The hotel (Spring) injects the required items (dependencies) into your room (class).

---

## 5. Can you give few examples of Dependency Injection?

### Constructor Injection
```java
@Component
public class Car {
    private final Engine engine;

    @Autowired
    public Car(Engine engine) { // Injected via constructor
        this.engine = engine;
    }
}
```

### Setter Injection
```java
@Component
public class Car {
    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) { // Injected via setter
        this.engine = engine;
    }
}
```

### Field Injection
```java
@Component
public class Car {
    @Autowired
    private Engine engine; // Injected directly into the field
}
```

---

## 6. What is Auto Wiring?

**Auto Wiring** is a Spring feature where the **IoC Container automatically resolves and injects the dependencies of a bean without explicit configuration**. Using the `@Autowired` annotation, Spring scans the application context, finds the matching bean by **type**, and injects it automatically. It eliminates the need for manual bean wiring in XML or Java configuration files.

**Code Example:**
```java
@Component
public class Car {
    @Autowired // Spring automatically finds and injects Engine bean
    private Engine engine;
}
```

**Real World Analogy:** Like a power strip with auto-detect — you plug in any device and it automatically provides the right voltage. You don't configure anything manually.

---

## 7. What are the Important Roles of an IOC Container?

The **Spring IoC Container** plays the following critical roles:

1. **Object Creation** — Instantiates beans defined in configuration (XML, Java, or annotations).
2. **Dependency Injection** — Wires dependencies between beans automatically.
3. **Lifecycle Management** — Manages the complete lifecycle of beans from creation to destruction (`@PostConstruct`, `@PreDestroy`).
4. **Configuration Management** — Reads and processes configuration metadata (XML, `@Configuration` classes, annotations).
5. **Bean Scoping** — Manages bean scopes like `singleton`, `prototype`, `request`, `session`, etc.
6. **AOP Support** — Provides integration with Aspect-Oriented Programming for cross-cutting concerns.

---

## 8. What are Bean Factory and Application Context?

Both are **implementations of the Spring IoC Container**, but differ in capability:

- **BeanFactory** (`org.springframework.beans.factory.BeanFactory`) — The **basic/root IoC container**. It provides fundamental DI support. It uses **lazy initialization** (beans are created only when requested). Lightweight and suitable for resource-constrained environments.

- **ApplicationContext** (`org.springframework.context.ApplicationContext`) — A **superset of BeanFactory**. It adds enterprise-level features like event propagation, internationalization (i18n), AOP integration, and **eager initialization** (all singleton beans are created at startup). It is the **recommended container** for most Spring applications.

---

## 9. Can you Compare Bean Factory with Application Context?

| Feature | BeanFactory | ApplicationContext |
|---|---|---|
| Bean Initialization | Lazy (on demand) | Eager (at startup) |
| Annotation Support | Limited | Full (`@Autowired`, `@Component`, etc.) |
| Internationalization (i18n) | ❌ Not supported | ✅ Supported |
| Event Publishing | ❌ Not supported | ✅ Supported |
| AOP Integration | ❌ Manual | ✅ Built-in |
| Recommended for | Resource-constrained environments | All enterprise applications |
| Common Implementation | `XmlBeanFactory` (deprecated) | `AnnotationConfigApplicationContext` |

> **Use ApplicationContext** in almost all cases. BeanFactory is rarely used in modern Spring applications.

---

## 10. How do you Create an Application Context with Spring?

There are three common ways:

### Using XML Configuration
```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

### Using Java Configuration
```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

### Using Spring Boot (Auto-configured)
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(MyApp.class, args);
    }
}
```

---

## 11. How does Spring Know Where to Search for Components or Beans?

Spring uses **Component Scanning** to search for beans. When `@ComponentScan` is specified (or implicitly via `@SpringBootApplication`), Spring scans the **specified base package and all its sub-packages** for classes annotated with stereotype annotations like `@Component`, `@Service`, `@Repository`, and `@Controller`, and registers them as beans in the IoC Container.

---

## 12. What is a Component Scan?

**Component Scan** is the mechanism by which the **Spring IoC Container automatically detects and registers beans** by scanning specified packages for classes annotated with Spring stereotype annotations (`@Component`, `@Service`, `@Repository`, `@Controller`). It eliminates the need to manually define every bean in XML or Java configuration.

**Real World Analogy:** Like a recruiter scanning LinkedIn profiles in a particular city for candidates with specific skills — Spring scans packages for classes with specific annotations.

---

## 13. How do you Define a Component Scan in XML and Java Configurations?

### XML Configuration
```xml
<context:component-scan base-package="com.example.myapp" />
```

### Java Configuration
```java
@Configuration
@ComponentScan(basePackages = "com.example.myapp")
public class AppConfig {
}
```

---

## 14. How is it Done with Spring Boot?

In Spring Boot, component scanning is **automatically enabled** via `@SpringBootApplication`, which internally includes `@ComponentScan`. It scans the **package of the main class and all its sub-packages** by default.

```java
@SpringBootApplication // Includes @ComponentScan automatically
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

To customize the scan:
```java
@SpringBootApplication(scanBasePackages = "com.example.myapp")
```

---

## 15. What does @Component Signify?

`@Component` is a **generic Spring stereotype annotation** that marks a Java class as a **Spring-managed bean**. When Spring's component scan detects a class annotated with `@Component`, it automatically **instantiates it and registers it in the IoC Container** as a bean, making it available for dependency injection throughout the application.

```java
@Component
public class EmailService {
    public void sendEmail(String message) {
        System.out.println("Sending email: " + message);
    }
}
```

---

## 16. What does @Autowired Signify?

`@Autowired` is a Spring annotation that **instructs the IoC Container to automatically inject the matching dependency (bean) into the annotated field, constructor, or setter method**. Spring resolves the dependency by **type** by default. If multiple beans of the same type exist, it uses `@Qualifier` or `@Primary` to resolve the conflict.

```java
@Component
public class Car {
    @Autowired // Spring injects the Engine bean automatically
    private Engine engine;
}
```

---

## 17. What's the Difference Between @Controller, @Component, @Repository, and @Service?

All four are **Spring stereotype annotations** and are specializations of `@Component`. They all register the class as a Spring bean, but carry **semantic meaning** and have **layer-specific behavior**:

| Annotation | Layer | Purpose |
|---|---|---|
| `@Component` | Generic | General-purpose Spring bean; no specific layer |
| `@Service` | Business/Service Layer | Marks a class containing business logic |
| `@Repository` | Persistence/DAO Layer | Marks a class interacting with the database; enables **exception translation** (converts DB exceptions to Spring's `DataAccessException`) |
| `@Controller` | Presentation/Web Layer | Marks a class as a Spring MVC controller that handles HTTP requests |

```java
@Controller   // Handles web requests
public class UserController { }

@Service      // Contains business logic
public class UserService { }

@Repository   // Interacts with database
public class UserRepository { }

@Component    // Generic bean
public class UtilityHelper { }
```

---

## 18. What are Some Important Spring Annotations?

| Annotation | Purpose |
|---|---|
| `@SpringBootApplication` | Entry point; combines `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan` |
| `@Component` | Generic Spring bean |
| `@Service` | Business layer bean |
| `@Repository` | DAO/persistence layer bean |
| `@Controller` | Web/MVC layer bean |
| `@RestController` | `@Controller` + `@ResponseBody`; for REST APIs |
| `@Autowired` | Auto wire dependencies |
| `@Qualifier` | Resolve ambiguity when multiple beans of same type exist |
| `@Primary` | Mark a bean as the default when multiple candidates exist |
| `@Bean` | Declare a bean inside a `@Configuration` class |
| `@Configuration` | Marks a class as source of bean definitions |
| `@ComponentScan` | Configure component scanning packages |
| `@Scope` | Define bean scope (singleton, prototype, etc.) |
| `@Value` | Inject values from properties files |
| `@PostConstruct` | Method to run after bean initialization |
| `@PreDestroy` | Method to run before bean destruction |

---

## 19. What is the Default Scope of a Bean?

The **default scope of a Spring bean is `singleton`**. This means the **IoC Container creates only one instance of the bean per application context**, and the same instance is shared and reused across the entire application wherever that bean is injected.

```java
@Component
// @Scope("singleton") — this is the default, no need to explicitly mention
public class UserService { }
```

---

## 20. Are Spring Beans Thread Safe?

**No, Spring beans are NOT thread safe by default.** Since the default scope is `singleton`, the **same bean instance is shared across multiple threads**. If the bean holds any **mutable state (instance variables)**, concurrent access can cause race conditions and data inconsistency. To handle this:
- Make beans **stateless** (no instance variables).
- Use **`prototype` scope** for stateful beans.
- Use **synchronization** or **thread-local variables** where necessary.

---

## 21. What are the Other Scopes Available?

| Scope | Description | Use Case |
|---|---|---|
| `singleton` | One instance per IoC container (default) | Stateless shared services |
| `prototype` | New instance every time bean is requested | Stateful beans |
| `request` | One instance per HTTP request (Web only) | Web applications |
| `session` | One instance per HTTP session (Web only) | User session data |
| `application` | One instance per ServletContext (Web only) | App-wide shared state |
| `websocket` | One instance per WebSocket session | WebSocket apps |

```java
@Component
@Scope("prototype")
public class ShoppingCart { } // New cart for every user request
```

---

## 22. How is Spring's Singleton Bean Different from GoF Singleton Pattern?

| Aspect | GoF Singleton Pattern | Spring Singleton Bean |
|---|---|---|
| Scope | One instance per **JVM/ClassLoader** | One instance per **Spring IoC Container** |
| Implementation | Private constructor + static instance | Managed entirely by Spring Container |
| Testability | Hard to mock/test | Easy to mock and test |
| Multiple Instances | Impossible (one per JVM) | Possible (one per container; multiple containers = multiple instances) |
| Control | Developer controls instantiation | Spring Container controls instantiation |

---

## 23. What are the Different Types of Dependency Injections?

Spring supports **three types** of Dependency Injection:

1. **Constructor Injection** — Dependencies injected via the class constructor.
2. **Setter Injection** — Dependencies injected via setter methods.
3. **Field Injection** — Dependencies injected directly into fields using `@Autowired`.

---

## 24. What is Setter Injection?

**Setter Injection** is a type of DI where the **Spring IoC Container injects the dependency by calling the setter method** of the dependent class after instantiating it via the default constructor. It is used for **optional dependencies**.

```java
@Component
public class Car {
    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

---

## 25. What is Constructor Injection?

**Constructor Injection** is a type of DI where the **Spring IoC Container injects the dependency through the constructor** of the dependent class at the time of bean creation. It is the **recommended approach** for **mandatory dependencies** as it ensures the bean is always in a fully initialized state and supports **immutability** (using `final` fields).

```java
@Component
public class Car {
    private final Engine engine;

    @Autowired
    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

---

## 26. How do you Choose Between Setter and Constructor Injections?

| Criteria | Constructor Injection | Setter Injection |
|---|---|---|
| Dependency Type | Mandatory dependencies | Optional dependencies |
| Immutability | ✅ Supports `final` fields | ❌ Cannot use `final` |
| Circular Dependency | ❌ Harder to resolve | ✅ Easier to resolve |
| Recommended by Spring | ✅ Yes (preferred) | For optional/legacy cases |
| Testability | Easier to test (no partial state) | Slightly harder |

> **Spring Team recommends Constructor Injection** for mandatory dependencies as it ensures complete initialization and supports immutability.

---

## 27. What are the Different Options to Create Application Contexts?

| Class | Configuration Type |
|---|---|
| `ClassPathXmlApplicationContext` | XML config from classpath |
| `FileSystemXmlApplicationContext` | XML config from file system |
| `AnnotationConfigApplicationContext` | Java `@Configuration` classes |
| `AnnotationConfigWebApplicationContext` | Java config for Web applications |
| `SpringApplication.run()` | Spring Boot (auto-configured) |

---

## 28. What is the Difference Between XML and Java Configurations?

| Aspect | XML Configuration | Java Configuration |
|---|---|---|
| Format | XML file | Java class with `@Configuration` |
| Type Safety | ❌ No compile-time checks | ✅ Compile-time type safety |
| Refactoring | ❌ IDE support limited | ✅ Full IDE support |
| Readability | Verbose for large apps | Cleaner and concise |
| Introduced in | Spring 1.x | Spring 3.x+ |

---

## 29. How do you Choose Between XML and Java Configurations?

- **Prefer Java Configuration** (`@Configuration`) for all modern Spring and Spring Boot applications — it is type-safe, easier to refactor, and IDE-friendly.
- **Use XML Configuration** only when working with **legacy Spring applications** that already have XML-based setup, or when externalizing configuration that non-developers need to modify without recompiling.

> **Spring Boot applications exclusively use Java/annotation-based configuration** by convention.

---

## 30. How does Spring do Autowiring?

Spring performs autowiring by **scanning the IoC container for beans that match the required dependency** and injecting them. The matching process follows this order:
1. **By Type** — Finds a bean matching the declared type.
2. **By Qualifier** — If multiple beans of the same type exist, uses `@Qualifier` to narrow down.
3. **By Primary** — If multiple beans exist and one is marked `@Primary`, that bean is selected.
4. **By Name** — Falls back to matching the field/parameter name with the bean name.

---

## 31. What are the Different Kinds of Matching Used by Spring for Autowiring?

| Matching Strategy | Description |
|---|---|
| **By Type** (`byType`) | Default; matches bean by its class/interface type |
| **By Name** (`byName`) | Matches bean by the field or setter method name |
| **By Constructor** | Matches constructor arguments by type |
| **`@Qualifier`** | Explicitly specifies the bean name to inject when multiple candidates exist |
| **`@Primary`** | Marks one bean as the default candidate when multiple beans of same type exist |

---

## 32. How do you Debug Problems with Spring Framework?

1. **Enable Debug Logging** — Add `logging.level.org.springframework=DEBUG` in `application.properties`.
2. **Check Bean Registration** — Use `context.getBeanDefinitionNames()` to list all registered beans.
3. **Read Exception Stack Trace carefully** — Spring exceptions like `NoSuchBeanDefinitionException` and `NoUniqueBeanDefinitionException` clearly indicate the problem.
4. **Verify Component Scan package** — Ensure the package being scanned contains your annotated classes.
5. **Use Spring Boot Actuator** — `/actuator/beans` endpoint lists all beans at runtime.
6. **Check Circular Dependencies** — Look for `BeanCurrentlyInCreationException`.

---

## 33. How do you Solve NoUniqueBeanDefinitionException?

`NoUniqueBeanDefinitionException` occurs when **Spring finds more than one bean of the required type** and cannot decide which one to inject.

**Solutions:**

### Using @Primary
```java
@Component
@Primary // This bean will be injected by default
public class PetrolEngine implements Engine { }

@Component
public class ElectricEngine implements Engine { }
```

### Using @Qualifier
```java
@Component
public class Car {
    @Autowired
    @Qualifier("electricEngine") // Explicitly specify which bean to inject
    private Engine engine;
}
```

---

## 34. How do you Solve NoSuchBeanDefinitionException?

`NoSuchBeanDefinitionException` occurs when **Spring cannot find a bean of the required type or name** in the IoC Container.

**Common Causes & Solutions:**

| Cause | Solution |
|---|---|
| Class not annotated with `@Component` etc. | Add appropriate stereotype annotation |
| Class not in scanned package | Fix `@ComponentScan` base package |
| Typo in bean name with `@Qualifier` | Verify bean name matches exactly |
| Missing `@Configuration` or `@Bean` | Add proper configuration |

---

## 35. What is @Primary?

`@Primary` is a Spring annotation used to **mark one bean as the default candidate** for autowiring when **multiple beans of the same type** exist in the IoC Container. When Spring finds multiple matching beans and `@Qualifier` is not specified, it selects the `@Primary` bean.

```java
@Component
@Primary // Default choice when Engine type is requested
public class PetrolEngine implements Engine { }

@Component
public class ElectricEngine implements Engine { }
```

---

## 36. What is @Qualifier?

`@Qualifier` is a Spring annotation used to **explicitly specify which bean should be injected** when multiple beans of the same type exist in the IoC Container. It overrides the default `@Primary` selection and resolves `NoUniqueBeanDefinitionException` by targeting a specific bean by its name.

```java
@Component
public class Car {
    @Autowired
    @Qualifier("electricEngine") // Explicitly inject ElectricEngine
    private Engine engine;
}
```

---

## 37. What is CDI (Contexts and Dependency Injection)?

**CDI (Contexts and Dependency Injection)** is a **Java EE / Jakarta EE standard specification** (`JSR-299`, later `JSR-365`) for dependency injection in Java applications. It defines a standard set of annotations like `@Inject`, `@Named`, `@Singleton`, `@Qualifier` for DI, making the code portable across any CDI-compliant container (like WildFly, Payara, etc.) without being tied to a specific framework like Spring.

| CDI Annotation | Spring Equivalent |
|---|---|
| `@Inject` | `@Autowired` |
| `@Named` | `@Component` / `@Qualifier` |
| `@Singleton` | `@Scope("singleton")` |
| `@Qualifier` | `@Qualifier` |

---

## 38. Does Spring Support CDI?

**Yes, Spring supports CDI annotations.** Spring Framework provides compatibility with CDI annotations like `@Inject` and `@Named` through the `javax.inject` / `jakarta.inject` package. You can use `@Inject` instead of `@Autowired` and `@Named` instead of `@Component` in a Spring application, and Spring will resolve them correctly.

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named // Works like @Component in Spring
public class Car {
    @Inject // Works like @Autowired in Spring
    private Engine engine;
}
```

---

## 39. Would you Recommend CDI or Spring Annotations?

| Aspect | CDI Annotations | Spring Annotations |
|---|---|---|
| Portability | ✅ Portable across Java EE containers | ❌ Spring-specific |
| Features | Basic DI only | Rich ecosystem (AOP, Boot, Cloud, etc.) |
| Spring Compatibility | ✅ Supported in Spring | N/A |
| Community & Ecosystem | Smaller | Massive |
| Recommended for | Java EE / Jakarta EE apps | Spring / Spring Boot apps |

> **Recommendation:** Use **Spring annotations** (`@Autowired`, `@Component`) when building Spring or Spring Boot applications — they provide richer features, better IDE support, and tighter Spring ecosystem integration. Use **CDI annotations** only if you need **portability across Java EE containers** or are working in a pure Jakarta EE environment.

---

*Happy Learning Spring! 🌱*
