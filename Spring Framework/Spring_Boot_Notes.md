# Spring Boot - Interview Questions & Answers

---

## 1. What is Spring Boot?

**Spring Boot** is an **opinionated, convention-over-configuration extension of the Spring Framework** that simplifies the bootstrapping and development of production-ready Spring applications. It eliminates boilerplate configuration by providing **auto-configuration**, **embedded servers**, and **starter dependencies**, allowing developers to build and run a Spring application with minimal setup — often with a single `java -jar` command.

**Real World Analogy:** Building a Spring app from scratch is like assembling a PC from individual parts. Spring Boot is like buying a pre-built laptop — everything is already configured and ready to use out of the box.

---

## 2. Goals of Spring Boot?

| Goal | Description |
|---|---|
| **Rapid Development** | Get a production-ready app running in minutes, not days |
| **Zero XML Configuration** | No need for `applicationContext.xml` or `web.xml` |
| **Auto Configuration** | Automatically configures beans based on classpath and properties |
| **Standalone Applications** | Apps run as a simple `java -jar` — no external server needed |
| **Opinionated Defaults** | Sensible defaults out of the box; override only when needed |
| **Production Ready** | Built-in health checks, metrics, and monitoring via Actuator |
| **No Code Generation** | Achieves magic through configuration, not code generation |

---

## 3. Features of Spring Boot?

| Feature | Description |
|---|---|
| **Auto Configuration** | Automatically sets up beans based on classpath dependencies |
| **Starter Projects** | Pre-packaged dependency sets (e.g., `spring-boot-starter-web`) |
| **Embedded Server** | Comes with embedded Tomcat/Jetty/Undertow — no WAR deployment needed |
| **Spring Initializr** | Web tool to bootstrap a Spring Boot project in seconds |
| **Actuator** | Production monitoring endpoints (health, metrics, env, beans) |
| **Externalized Configuration** | `application.properties` / `application.yml` for environment-specific config |
| **Profiles** | Different configurations for `dev`, `test`, `prod` environments |
| **Spring Boot DevTools** | Auto-restart and live reload during development |
| **Dependency Management (BOM)** | Manages compatible dependency versions via `spring-boot-starter-parent` |
| **Logging** | Auto-configured logging with Logback by default |

---

## 4. Spring Boot vs Spring?

| Aspect | Spring Framework | Spring Boot |
|---|---|---|
| **Configuration** | Manual XML or Java config required | Auto-configured — zero boilerplate |
| **Server** | Deploy WAR to external Tomcat/JBoss | Embedded Tomcat — run as JAR |
| **Dependency Management** | Manually manage all dependencies and versions | Managed via starter parent BOM |
| **Setup Time** | Days (for large enterprise apps) | Minutes |
| **Boilerplate** | High — need `DispatcherServlet`, `ViewResolver`, etc. | None — all auto-configured |
| **Use Case** | Full control over every configuration | Rapid application development |
| **Opinionated?** | No — you decide everything | Yes — sensible defaults provided |

> **Spring Boot does NOT replace Spring.** It is built **on top of Spring** and uses Spring internally. Spring Boot = Spring + Auto Configuration + Embedded Server + Starter Projects.

---

## 5. Spring Boot vs Spring MVC?

| Aspect | Spring MVC | Spring Boot |
|---|---|---|
| **What it is** | Web framework for building MVC/REST apps | Tool to quickly set up Spring applications |
| **Purpose** | Handles HTTP requests, routing, views | Simplifies configuration and deployment |
| **Configuration** | Requires manual `DispatcherServlet`, `ViewResolver` setup | Auto-configures Spring MVC when `spring-boot-starter-web` is added |
| **Server** | Deployed as WAR to external server | Embedded server — run as JAR |
| **Can exist without other?** | Spring MVC without Spring Boot — yes (but verbose) | Spring Boot without Spring MVC — yes (e.g., batch apps) |

> **They complement each other.** In a typical Spring Boot web app, **Spring Boot sets up** the infrastructure and **Spring MVC handles** the web layer.

---

## 6. Importance of @SpringBootApplication?

`@SpringBootApplication` is the **single most important annotation** in Spring Boot. It is a **composed annotation** that combines three critical annotations:

| Annotation | Purpose |
|---|---|
| `@SpringBootConfiguration` | Marks the class as a Spring configuration class (specialization of `@Configuration`) |
| `@EnableAutoConfiguration` | Enables Spring Boot's auto-configuration mechanism |
| `@ComponentScan` | Scans the current package and sub-packages for Spring components |

```java
@SpringBootApplication  // Replaces all three annotations below
// @SpringBootConfiguration
// @EnableAutoConfiguration
// @ComponentScan
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);  // Starts the Spring application
    }
}
```

**What `SpringApplication.run()` does:**
1. Creates the `ApplicationContext`
2. Triggers auto-configuration
3. Starts the embedded server
4. Runs `CommandLineRunner` / `ApplicationRunner` beans

---

## 7. What is Auto Configuration?

**Auto Configuration** is Spring Boot's mechanism of **automatically configuring Spring beans based on the dependencies present on the classpath** and the properties defined in `application.properties`. It uses `@Conditional` annotations internally to decide whether to create a bean or not.

```
If spring-boot-starter-web is on classpath
  → Auto-configure DispatcherServlet, ViewResolver, Jackson, Tomcat

If spring-boot-starter-data-jpa is on classpath
  → Auto-configure EntityManagerFactory, DataSource, TransactionManager

If H2 dependency is on classpath
  → Auto-configure in-memory DataSource
```

**How it works internally:**
```java
// Spring Boot reads this file at startup:
// META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
// It contains a list of all auto-configuration classes like:
// org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
// org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration

// Example of what an auto-config class looks like internally:
@Configuration
@ConditionalOnClass(DataSource.class)       // Only if DataSource class is on classpath
@ConditionalOnMissingBean(DataSource.class) // Only if no DataSource bean is defined by user
public class DataSourceAutoConfiguration {
    @Bean
    public DataSource dataSource() { ... }  // Auto-creates DataSource
}
```

> **Key Point:** Auto Configuration is **non-invasive** — if you define your own bean, Spring Boot backs off and uses yours instead.

---

## 8. How to Find Auto Configuration Details?

There are multiple ways to inspect what Spring Boot has auto-configured:

### Option 1: Enable Debug Logging
```properties
# application.properties
logging.level.org.springframework=DEBUG
# or
debug=true
```
This prints a **Conditions Evaluation Report** at startup showing:
- ✅ **Positive matches** — auto-configurations that were applied
- ❌ **Negative matches** — auto-configurations that were skipped and why
- 🔷 **Exclusions** — auto-configurations explicitly excluded

### Option 2: Spring Boot Actuator `/actuator/conditions`
```properties
management.endpoints.web.exposure.include=conditions
```
Hit `http://localhost:8080/actuator/conditions` to see all condition evaluations as JSON.

### Option 3: Spring Boot Actuator `/actuator/beans`
```
http://localhost:8080/actuator/beans
```
Lists all beans currently registered in the ApplicationContext.

### Option 4: IDE Support
- IntelliJ IDEA Ultimate shows auto-configuration hints in the Spring Boot run dashboard.

---

## 9. What is an Embedded Server?

An **Embedded Server** is a web server (like Tomcat, Jetty, or Undertow) that is **packaged inside the application JAR itself**, so the application can run standalone without needing an externally installed server. Spring Boot embeds the server as a dependency, and it starts automatically when the app starts.

| Traditional Approach | Spring Boot Embedded Server |
|---|---|
| Build WAR file | Build JAR file |
| Deploy WAR to external Tomcat | Run `java -jar myapp.jar` |
| Tomcat installed separately | Tomcat is inside the JAR |
| Requires server admin setup | No server setup needed |
| Complex deployment | Simple, portable deployment |

**Real World Analogy:** Traditional deployment is like shipping food ingredients and expecting the restaurant to have a kitchen. Embedded server is like delivering a fully cooked meal in its own container — just open and eat.

---

## 10. Default Embedded Server?

The **default embedded server in Spring Boot is Apache Tomcat**.

When you add `spring-boot-starter-web`, Tomcat is automatically included:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- Includes spring-boot-starter-tomcat by default -->
</dependency>
```

Default port: **8080**

```properties
# Change default port
server.port=9090

# Change context path
server.servlet.context-path=/myapp
```

---

## 11. Other Embedded Servers?

Spring Boot supports **three embedded servers**. You can swap Tomcat for Jetty or Undertow:

| Server | Best For | How to Use |
|---|---|---|
| **Tomcat** (default) | General-purpose, most widely used | Included by default with `spring-boot-starter-web` |
| **Jetty** | Lightweight, long-lived connections, WebSocket | Exclude Tomcat, add Jetty starter |
| **Undertow** | High-performance, non-blocking, low memory | Exclude Tomcat, add Undertow starter |

### Switching to Jetty:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

---

## 12. What are Starter Projects?

**Starter Projects** are **pre-packaged sets of dependencies** (Maven/Gradle) that group together all the libraries needed for a specific feature or technology. Instead of manually adding 10+ dependencies with compatible versions, you add **one starter** and Spring Boot handles the rest.

**Naming Convention:** `spring-boot-starter-{feature}`

```xml
<!-- Instead of adding Jackson, Hibernate Validator, Tomcat, Spring MVC separately... -->
<!-- Just add ONE starter: -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- This brings in: Spring MVC + Jackson + Tomcat + Validation + Logging -->
```

---

## 13. Examples of Starter Projects?

| Starter | What it Includes | Use Case |
|---|---|---|
| `spring-boot-starter-web` | Spring MVC, Tomcat, Jackson, Validation | Build REST APIs and web apps |
| `spring-boot-starter-data-jpa` | Spring Data JPA, Hibernate, JDBC | Database access with JPA/ORM |
| `spring-boot-starter-security` | Spring Security | Authentication and Authorization |
| `spring-boot-starter-test` | JUnit 5, Mockito, AssertJ, MockMvc | Unit and integration testing |
| `spring-boot-starter-thymeleaf` | Thymeleaf template engine | Server-side rendered HTML pages |
| `spring-boot-starter-actuator` | Health, Metrics, Info endpoints | Production monitoring |
| `spring-boot-starter-mail` | JavaMail, Spring Mail | Sending emails |
| `spring-boot-starter-cache` | Spring Cache abstraction | Application-level caching |
| `spring-boot-starter-amqp` | Spring AMQP, RabbitMQ | Message queue integration |
| `spring-boot-starter-batch` | Spring Batch | Large-scale batch processing |
| `spring-boot-starter-webflux` | Spring WebFlux, Reactor | Reactive, non-blocking web apps |
| `spring-boot-starter-validation` | Hibernate Validator, JSR-380 | Bean validation |

---

## 14. What is Starter Parent?

**`spring-boot-starter-parent`** is a special Maven parent POM that provides:
- **Dependency Management (BOM)** — pre-defined, compatible versions for hundreds of libraries
- **Default Plugin Configuration** — Maven Surefire, Spring Boot Maven Plugin, etc.
- **Java version defaults**
- **Resource filtering** for `application.properties` / `application.yml`

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>
```

Once declared, you can add any Spring Boot-managed dependency **without specifying a version** — the parent manages it for you.

---

## 15. What is Defined in Starter Parent?

| What's Defined | Details |
|---|---|
| **Java Version** | Default Java source/target version (e.g., Java 17 for Spring Boot 3.x) |
| **Dependency Versions** | Hundreds of pre-tested library versions (Hibernate, Jackson, Tomcat, etc.) |
| **Plugin Management** | Maven Compiler Plugin, Surefire Plugin, Spring Boot Maven Plugin |
| **Resource Filtering** | `*.properties` and `*.yml` files auto-filtered for `@project.version@` placeholders |
| **Encoding** | Default UTF-8 encoding for source and resources |
| **Inherited from `spring-boot-dependencies`** | The actual BOM (Bill of Materials) containing all version declarations |

```xml
<!-- You can override any default version: -->
<properties>
    <java.version>21</java.version>
    <jackson.version>2.16.0</jackson.version>
</properties>
```

---

## 16. Dependency Management in Spring Boot?

Spring Boot manages dependency versions through the **`spring-boot-dependencies` BOM (Bill of Materials)**. This ensures all libraries work together without version conflicts.

| Approach | How |
|---|---|
| **Via Parent POM** | Declare `spring-boot-starter-parent` as Maven parent — simplest approach |
| **Via BOM import** | Import `spring-boot-dependencies` in `<dependencyManagement>` — for projects with their own parent |

```xml
<!-- Option 1: Parent POM (simplest) -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>

<!-- Option 2: BOM import (when you have your own parent) -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.2.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

## 17. What is Spring Initializr?

**Spring Initializr** is a **web-based project generator** at [https://start.spring.io](https://start.spring.io) that lets you **bootstrap a Spring Boot project in seconds** by selecting your project settings and downloading a ready-to-import ZIP.

| What You Configure | Options |
|---|---|
| **Build Tool** | Maven / Gradle |
| **Language** | Java / Kotlin / Groovy |
| **Spring Boot Version** | Latest stable / milestone |
| **Project Metadata** | Group, Artifact, Name, Package, Description |
| **Java Version** | 17, 21, etc. |
| **Dependencies** | Select starters (Web, JPA, Security, etc.) |

**Also available in IDEs:**
- IntelliJ IDEA: `File → New → Project → Spring Initializr`
- Spring Tool Suite (STS): Built-in Initializr wizard
- VS Code: Spring Boot Extension Pack

---

## 18. What is application.properties?

**`application.properties`** is the **primary configuration file** for a Spring Boot application, located at `src/main/resources/`. It allows you to configure the application behavior, override auto-configuration defaults, and define custom properties — all without changing Java code.

```properties
# Server Configuration
server.port=9090
server.servlet.context-path=/api

# DataSource
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret

# JPA / Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# Logging
logging.level.com.example=DEBUG

# App name
spring.application.name=my-spring-app
```

> Spring Boot also supports **`application.yml`** (YAML format) as an alternative to `application.properties`.

---

## 19. What can be Customized in application.properties?

| Category | Example Properties |
|---|---|
| **Server** | `server.port`, `server.servlet.context-path`, `server.ssl.*` |
| **DataSource** | `spring.datasource.url`, `spring.datasource.username`, `spring.datasource.driver-class-name` |
| **JPA/Hibernate** | `spring.jpa.show-sql`, `spring.jpa.hibernate.ddl-auto`, `spring.jpa.properties.*` |
| **Logging** | `logging.level.*`, `logging.file.name`, `logging.pattern.console` |
| **Spring MVC** | `spring.mvc.view.prefix`, `spring.mvc.view.suffix`, `spring.mvc.format.date` |
| **Security** | `spring.security.user.name`, `spring.security.user.password` |
| **Actuator** | `management.endpoints.web.exposure.include`, `management.server.port` |
| **Profiles** | `spring.profiles.active=dev` |
| **Custom Properties** | Any key-value pair you define: `app.name=MyApp`, `app.max-retries=3` |
| **Cache** | `spring.cache.type`, `spring.cache.redis.*` |
| **Mail** | `spring.mail.host`, `spring.mail.port`, `spring.mail.username` |

---

## 20. How to Externalize Configuration?

**Externalizing configuration** means keeping environment-specific values (DB passwords, URLs, ports) **outside the application code**, so you can change them per environment without rebuilding.

Spring Boot supports multiple externalization mechanisms (in order of priority — higher overrides lower):

| Priority | Source | Example |
|---|---|---|
| **1 (Highest)** | Command-line arguments | `java -jar app.jar --server.port=9090` |
| **2** | OS Environment Variables | `export SERVER_PORT=9090` |
| **3** | `application-{profile}.properties` | `application-prod.properties` |
| **4** | `application.properties` (inside JAR) | `src/main/resources/application.properties` |
| **5 (Lowest)** | `@PropertySource` annotations | `@PropertySource("classpath:custom.properties")` |

```bash
# Override at runtime without changing code:
java -jar myapp.jar --spring.datasource.url=jdbc:mysql://prod-server/db --server.port=8081
```

---

## 21. How to Add Custom Properties?

You can define your own custom properties in `application.properties` and inject them into Spring beans:

### Using @Value
```properties
# application.properties
app.name=MySpringApp
app.max-retries=3
app.timeout=5000
```

```java
@Component
public class AppConfig {

    @Value("${app.name}")
    private String appName;

    @Value("${app.max-retries:5}") // Default value = 5 if property not found
    private int maxRetries;

    @Value("${app.timeout}")
    private long timeout;
}
```

### Using @ConfigurationProperties (Recommended for grouped properties)
```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private int maxRetries;
    private long timeout;
    // getters and setters
}
```

---

## 22. What is @ConfigurationProperties?

`@ConfigurationProperties` is a Spring Boot annotation that **binds a group of related properties from `application.properties`** to a Java class by matching the prefix. It is the **recommended approach** for working with multiple related properties as it is type-safe, IDE-friendly, and supports validation.

```properties
# application.properties
mail.host=smtp.gmail.com
mail.port=587
mail.username=test@gmail.com
mail.password=secret
mail.ssl-enabled=true
```

```java
@Component
@ConfigurationProperties(prefix = "mail")  // Binds all "mail.*" properties
@Validated                                  // Optional: enable validation
public class MailProperties {

    @NotNull
    private String host;

    private int port;

    private String username;
    private String password;
    private boolean sslEnabled;

    // getters and setters (required for binding)
}
```

```java
@Service
public class EmailService {

    @Autowired
    private MailProperties mailProperties; // Inject the bound properties

    public void sendEmail() {
        System.out.println("Connecting to: " + mailProperties.getHost());
    }
}
```

| `@Value` | `@ConfigurationProperties` |
|---|---|
| One property at a time | Groups related properties together |
| Not type-safe for complex types | Fully type-safe |
| No IDE autocompletion | IDE autocompletion + metadata |
| Cannot validate easily | Supports `@Validated` |
| Good for 1-2 properties | Recommended for 3+ related properties |

---

## 23. What is a Profile?

A **Profile** in Spring Boot is a way to define **environment-specific configurations and beans** that are only activated in certain environments (e.g., `dev`, `test`, `prod`). It allows the same codebase to behave differently across environments without any code changes.

```properties
# Activate profile
spring.profiles.active=dev
```

**Real World Analogy:** A profile is like a wardrobe for different seasons — same person, different clothes depending on the environment (summer/winter = dev/prod).

---

## 24. Beans for Specific Profile?

Use `@Profile` annotation to create beans that are **only registered when a specific profile is active**:

```java
// Only created when "dev" profile is active
@Component
@Profile("dev")
public class DevDataSource implements DataSource {
    // Uses H2 in-memory DB for development
}

// Only created when "prod" profile is active
@Component
@Profile("prod")
public class ProdDataSource implements DataSource {
    // Uses MySQL for production
}

// Active for both dev and test profiles
@Component
@Profile({"dev", "test"})
public class MockEmailService implements EmailService {
    public void sendEmail(String msg) {
        System.out.println("Mock email: " + msg); // Don't actually send emails in dev/test
    }
}

// Active for all profiles EXCEPT prod
@Component
@Profile("!prod")
public class DebugLogger { }
```

---

## 25. Profile-Based Configuration?

Spring Boot automatically loads **`application-{profile}.properties`** when a profile is active, **overriding** the values in the base `application.properties`:

```
src/main/resources/
├── application.properties          ← Base config (loaded always)
├── application-dev.properties      ← Loaded when profile = dev
├── application-test.properties     ← Loaded when profile = test
└── application-prod.properties     ← Loaded when profile = prod
```

```properties
# application.properties (base — always loaded)
spring.application.name=MyApp
logging.level.root=INFO

# application-dev.properties (overrides base for dev)
server.port=8080
spring.datasource.url=jdbc:h2:mem:devdb
logging.level.com.example=DEBUG

# application-prod.properties (overrides base for prod)
server.port=80
spring.datasource.url=jdbc:mysql://prod-server:3306/mydb
logging.level.com.example=WARN
```

```properties
# Activate profile:
spring.profiles.active=dev

# Or via command line (recommended for production):
# java -jar myapp.jar --spring.profiles.active=prod
```

---

## 26. Different Configurations for Environments?

| Aspect | dev | test | prod |
|---|---|---|---|
| **Database** | H2 in-memory | H2 / Test DB | MySQL / PostgreSQL |
| **Logging Level** | DEBUG | INFO | WARN / ERROR |
| **Email** | Mock/Console | Mock | Real SMTP |
| **Server Port** | 8080 | 8081 | 80 / 443 |
| **Security** | Relaxed (no auth) | Mocked | Full auth + HTTPS |
| **Cache** | Disabled | Disabled | Redis / Ehcache |

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:devdb
  jpa:
    show-sql: true
logging:
  level:
    com.example: DEBUG

---
# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/myapp
    username: ${DB_USER}        # Read from env variables
    password: ${DB_PASSWORD}
  jpa:
    show-sql: false
logging:
  level:
    com.example: WARN
```

---

## 27. What is Spring Boot Actuator?

**Spring Boot Actuator** is a sub-project of Spring Boot that provides **production-ready monitoring and management endpoints** for your application. It exposes various HTTP endpoints (or JMX) that give insights into the running application's health, metrics, environment, beans, and more — without writing any code.

```xml
<!-- Add Actuator dependency -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# Expose all actuator endpoints
management.endpoints.web.exposure.include=*

# Or expose specific ones
management.endpoints.web.exposure.include=health,metrics,info,beans
```

---

## 28. Monitoring Using Actuator?

| Endpoint | URL | What it Shows |
|---|---|---|
| **health** | `/actuator/health` | App health status (UP/DOWN), DB, disk space |
| **metrics** | `/actuator/metrics` | JVM memory, CPU, HTTP request stats |
| **info** | `/actuator/info` | Custom app info (version, description) |
| **beans** | `/actuator/beans` | All Spring beans in ApplicationContext |
| **mappings** | `/actuator/mappings` | All `@RequestMapping` URL mappings |
| **conditions** | `/actuator/conditions` | Auto-configuration condition evaluation report |
| **env** | `/actuator/env` | All environment properties and values |
| **loggers** | `/actuator/loggers` | View and change log levels at runtime |
| **threaddump** | `/actuator/threaddump` | Current JVM thread dump |
| **heapdump** | `/actuator/heapdump` | Download JVM heap dump |
| **shutdown** | `/actuator/shutdown` | Gracefully shut down the app (disabled by default) |

```json
// GET /actuator/health
{
  "status": "UP",
  "components": {
    "db": { "status": "UP", "details": { "database": "MySQL" } },
    "diskSpace": { "status": "UP" }
  }
}
```

---

## 29. Application Environment Details?

The `/actuator/env` endpoint exposes **all configuration properties** available to the application from all sources:

```properties
# Enable env endpoint
management.endpoints.web.exposure.include=env
```

```json
// GET /actuator/env
{
  "propertySources": [
    {
      "name": "systemEnvironment",
      "properties": { "JAVA_HOME": { "value": "/usr/lib/jvm/java-17" } }
    },
    {
      "name": "applicationConfig:[classpath:/application.properties]",
      "properties": {
        "server.port": { "value": "8080" },
        "spring.application.name": { "value": "MyApp" }
      }
    }
  ]
}
```

You can also programmatically access environment details:
```java
@Autowired
private Environment environment;

public void printProfile() {
    System.out.println("Active Profiles: " +
        Arrays.toString(environment.getActiveProfiles()));
    System.out.println("Port: " + environment.getProperty("server.port"));
}
```

---

## 30. What is CommandLineRunner?

**`CommandLineRunner`** is a Spring Boot functional interface that allows you to **execute custom code immediately after the Spring ApplicationContext is fully initialized** and before the application starts accepting requests. It is commonly used for **data seeding, startup validation, or initialization tasks**.

```java
@FunctionalInterface
public interface CommandLineRunner {
    void run(String... args) throws Exception;
}
```

### Usage Example:
```java
@Component
public class DataSeeder implements CommandLineRunner {

    @Autowired
    private UserRepository userRepository;

    @Override
    public void run(String... args) throws Exception {
        // Runs once at startup — after Spring context is ready
        if (userRepository.count() == 0) {
            userRepository.save(new User("admin", "admin@example.com"));
            System.out.println("Admin user seeded successfully!");
        }
    }
}
```

### Multiple CommandLineRunners with ordering:
```java
@Component
@Order(1)  // Runs first
public class DatabaseInitializer implements CommandLineRunner {
    public void run(String... args) { System.out.println("DB initialized"); }
}

@Component
@Order(2)  // Runs second
public class CacheWarmer implements CommandLineRunner {
    public void run(String... args) { System.out.println("Cache warmed up"); }
}
```

| `CommandLineRunner` | `ApplicationRunner` |
|---|---|
| `run(String... args)` — raw string args | `run(ApplicationArguments args)` — parsed args with named options |
| Simpler | More powerful for complex CLI args |

**Real World Analogy:** `CommandLineRunner` is like a **pre-flight checklist** a pilot runs before takeoff — everything is set up and ready, and the runner does a final check/initialization before the plane (app) takes off.

---

*Happy Learning Spring Boot! 🚀*
