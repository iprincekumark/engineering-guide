# AOP (Aspect-Oriented Programming) — Interview Questions & Answers

---

## 1. What are Cross-Cutting Concerns?

**Cross-cutting concerns** are **functionalities that span across multiple layers and classes of an application** — they are not part of the core business logic but are needed everywhere. Scattering this code across every class leads to code duplication, tight coupling, and poor maintainability.

| Concern | Where It Appears |
|---|---|
| **Logging** | Every service method, every controller, every repository |
| **Security / Authorization** | Every endpoint, every service call |
| **Transaction Management** | Every DB operation |
| **Performance Monitoring** | Every method execution |
| **Exception Handling** | Every layer of the application |
| **Caching** | Every expensive method call |
| **Auditing** | Every create/update/delete operation |
| **Retry Logic** | Every external API or DB call |

### The Problem — Code Tangling and Scattering:
```java
// WITHOUT AOP — logging, security, and transaction mixed INTO business logic
@Service
public class OrderService {

    public Order placeOrder(Order order) {

        // Cross-cutting concern #1 — Logging (not business logic)
        logger.info("Entering placeOrder() with order: {}", order);

        // Cross-cutting concern #2 — Security (not business logic)
        if (!securityContext.hasRole("USER")) {
            throw new UnauthorizedException("Access denied");
        }

        // Cross-cutting concern #3 — Performance tracking (not business logic)
        long start = System.currentTimeMillis();

        // ✅ Actual business logic — just 3 lines
        validateOrder(order);
        Order saved = orderRepository.save(order);
        notificationService.sendConfirmation(saved);

        // Cross-cutting concern #3 continued — Performance tracking
        long end = System.currentTimeMillis();
        logger.info("placeOrder() executed in {} ms", (end - start));

        // Cross-cutting concern #1 continued — Logging
        logger.info("Exiting placeOrder() with result: {}", saved);

        return saved;
    }
}
```

**Problems with the above:**
- Business logic is buried under infrastructure code
- Same logging/security/timing code copy-pasted in 50 other service methods
- Changing the logging format requires editing every single class
- Unit testing business logic is harder — must deal with cross-cutting noise

**Real World Analogy:** Cross-cutting concerns are like **building security** in an office. Every floor (class) needs badge scanners, CCTV, and fire alarms — but it would be insane to wire each employee's desk individually. You install it centrally and it applies everywhere automatically.

---

## 2. How to Implement Cross-Cutting Concerns?

**AOP (Aspect-Oriented Programming)** is the Spring solution for implementing cross-cutting concerns **centrally in one place (Aspect)** and applying them automatically across the entire application — without touching the business logic classes at all.

| Approach | Description | Verdict |
|---|---|---|
| **Copy-Paste** | Repeat concern code in every class | ❌ Terrible — duplicated, unmaintainable |
| **Inheritance / Base Class** | Put concern in a parent class | ⚠️ Limited — Java is single-inheritance |
| **Decorator Pattern** | Wrap each class manually | ⚠️ Verbose — needs a wrapper per class |
| **Spring AOP (Aspects)** | Define concern once, apply via pointcut expressions | ✅ Best — clean, centralized, non-invasive |

### With AOP — Business Logic is Clean:
```java
// ✅ Service has ONLY business logic — no logging, no timing, no security
@Service
public class OrderService {

    public Order placeOrder(Order order) {
        validateOrder(order);
        Order saved = orderRepository.save(order);
        notificationService.sendConfirmation(saved);
        return saved;
    }
}

// ✅ All cross-cutting concerns live here — applied automatically by Spring AOP
@Aspect
@Component
public class LoggingAspect {

    // Applied to EVERY method in the service layer — zero changes to OrderService
    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecution(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("Entering: {}", joinPoint.getSignature().getName());
        Object result = joinPoint.proceed();               // Execute real method
        logger.info("Exiting:  {}", joinPoint.getSignature().getName());
        return result;
    }
}
```

### AOP Core Concepts at a Glance:

| Term | Meaning | Analogy |
|---|---|---|
| **Aspect** | The class containing the cross-cutting concern | Security department |
| **Advice** | The actual code to run (the action) | Badge scan procedure |
| **Pointcut** | Expression defining WHERE advice applies | "All doors on floors 1–10" |
| **Join Point** | A specific method execution being intercepted | Employee walking through door |
| **Weaving** | Process of applying advice to target classes | Installing badge scanners |

**Real World Analogy:** AOP is like setting up a **toll booth on a highway**. Instead of stopping each car individually and asking for payment, you install one booth — every car that passes through that road automatically gets checked. The car (business logic) doesn't change; only the road setup (aspect) does.

---

## 3. Logging Every Request?

**Logging every method execution** in the service layer using a single `@Aspect` class — without adding a single log statement to any service class.

### Dependency:
```xml
<!-- AOP support in Spring Boot -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### Complete Logging Aspect:
```java
@Aspect
@Component
public class LoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);

    // ==================== LOG METHOD ENTRY + ARGUMENTS ====================
    @Before("execution(* com.example.service.*.*(..))")
    public void logMethodEntry(JoinPoint joinPoint) {
        String methodName  = joinPoint.getSignature().getName();
        String className   = joinPoint.getTarget().getClass().getSimpleName();
        Object[] args      = joinPoint.getArgs();

        logger.info("[ENTRY] {}.{}() | Args: {}", className, methodName,
                    Arrays.toString(args));
    }

    // ==================== LOG METHOD EXIT + RETURN VALUE ====================
    @AfterReturning(
        pointcut = "execution(* com.example.service.*.*(..))",
        returning = "result"
    )
    public void logMethodExit(JoinPoint joinPoint, Object result) {
        String methodName = joinPoint.getSignature().getName();
        String className  = joinPoint.getTarget().getClass().getSimpleName();

        logger.info("[EXIT]  {}.{}() | Returned: {}", className, methodName, result);
    }

    // ==================== LOG EXCEPTIONS ====================
    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing  = "ex"
    )
    public void logException(JoinPoint joinPoint, Exception ex) {
        String methodName = joinPoint.getSignature().getName();
        String className  = joinPoint.getTarget().getClass().getSimpleName();

        logger.error("[ERROR] {}.{}() | Exception: {} | Message: {}",
                     className, methodName,
                     ex.getClass().getSimpleName(), ex.getMessage());
    }

    // ==================== LOG FULL LIFECYCLE WITH @Around ====================
    @Around("execution(* com.example.service.*.*(..))")
    public Object logFullLifecycle(ProceedingJoinPoint joinPoint) throws Throwable {
        String method = joinPoint.getSignature().toShortString();
        logger.info(">> START: {}", method);

        try {
            Object result = joinPoint.proceed();             // Execute actual method
            logger.info("<< END:   {} | Result: {}", method, result);
            return result;
        } catch (Throwable ex) {
            logger.error("!! ERROR: {} | {}", method, ex.getMessage());
            throw ex;                                        // Re-throw — don't swallow
        }
    }
}
```

### Console Output (Automatic — No Changes to Services):
```
[ENTRY] OrderService.placeOrder() | Args: [Order{id=0, amount=250.0}]
>> START: OrderService.placeOrder(..)
<< END:   OrderService.placeOrder(..) | Result: Order{id=5, amount=250.0}
[EXIT]  OrderService.placeOrder() | Returned: Order{id=5, amount=250.0}
```

### Logging Controller Requests (HTTP Layer):
```java
@Aspect
@Component
public class ControllerLoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(ControllerLoggingAspect.class);

    // Intercept ALL controller methods
    @Around("within(@org.springframework.web.bind.annotation.RestController *)")
    public Object logControllerRequest(ProceedingJoinPoint joinPoint) throws Throwable {
        String method    = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();

        logger.info("[HTTP REQUEST]  {}.{}()", className, method);
        Object result = joinPoint.proceed();
        logger.info("[HTTP RESPONSE] {}.{}() completed", className, method);

        return result;
    }
}
```

**Real World Analogy:** A logging aspect is like **CCTV cameras at every door**. You don't ask each employee to write in a diary when they enter/exit a room. The cameras (aspect) silently record everything — every person (method) walking through every door (execution point) is automatically captured.

---

## 4. Performance Tracking?

**Performance tracking** measures **how long each method takes to execute** and logs a warning if it exceeds a threshold — applied across all service methods from a single Aspect without touching any business class.

```java
@Aspect
@Component
public class PerformanceTrackingAspect {

    private static final Logger logger =
        LoggerFactory.getLogger(PerformanceTrackingAspect.class);

    private static final long SLOW_METHOD_THRESHOLD_MS = 500; // Warn if > 500ms

    // ==================== TRACK ALL SERVICE METHODS ====================
    @Around("execution(* com.example.service.*.*(..))")
    public Object trackPerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        String method    = joinPoint.getSignature().toShortString();
        long   startTime = System.currentTimeMillis();

        try {
            Object result = joinPoint.proceed();                    // Execute method
            return result;
        } finally {
            long executionTime = System.currentTimeMillis() - startTime;

            if (executionTime > SLOW_METHOD_THRESHOLD_MS) {
                logger.warn("[SLOW METHOD] {} | Took: {} ms ⚠️", method, executionTime);
            } else {
                logger.info("[PERF] {} | Took: {} ms", method, executionTime);
            }
        }
    }

    // ==================== CUSTOM ANNOTATION — @TrackTime ====================
    // Apply performance tracking only to methods annotated with @TrackTime
    @Around("@annotation(com.example.annotation.TrackTime)")
    public Object trackAnnotatedMethods(ProceedingJoinPoint joinPoint) throws Throwable {
        long   start  = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long   end    = System.currentTimeMillis();

        logger.info("[TRACK] {} executed in {} ms",
                    joinPoint.getSignature().getName(), (end - start));
        return result;
    }
}
```

### Custom `@TrackTime` Annotation:
```java
// Define the annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TrackTime {}

// Use it selectively on any method
@Service
public class ReportService {

    @TrackTime                          // AOP intercepts this and tracks time
    public Report generateMonthlyReport(int month, int year) {
        // Heavy computation...
        return report;
    }

    public List<String> getReportNames() {  // NOT tracked — no @TrackTime
        return reportRepository.findAllNames();
    }
}
```

### Sample Output:
```
[PERF]        UserService.getUserById(..)       | Took: 12 ms
[PERF]        OrderService.placeOrder(..)       | Took: 85 ms
[SLOW METHOD] ReportService.generateReport(..) | Took: 1240 ms ⚠️
[TRACK]       generateMonthlyReport executed in 1240 ms
```

**Real World Analogy:** Performance tracking via AOP is like **speed sensors on a highway**. You don't ask each driver to time themselves — sensors are installed at checkpoints (pointcuts), automatically flag speeders (slow methods), and log every vehicle's speed (execution time) without interfering with traffic (business logic).

---

## 5. What is Aspect and Pointcut?

### What is an Aspect?

**An Aspect** is a **plain Java class annotated with `@Aspect`** that **encapsulates a cross-cutting concern**. It contains one or more **Advices** (the code to run) and **Pointcuts** (the conditions defining where to run them).

```java
@Aspect          // Marks this class as an AOP Aspect
@Component       // Registers it as a Spring bean
public class SecurityAspect {
    // Contains advices + pointcuts for security checks
}
```

### What is a Pointcut?

**A Pointcut** is an **expression that defines WHICH methods should be intercepted** by the advice. Spring AOP uses **AspectJ pointcut expression language** to match methods by package, class, method name, parameters, annotations, and more.

```java
// Pointcut Expression Syntax:
// execution( [modifier] return-type declaring-type.method-name(params) [throws] )

// Examples:

// All methods in UserService
execution(* com.example.service.UserService.*(..))

// All methods in any class under the service package
execution(* com.example.service.*.*(..))

// All methods in any class under service package AND sub-packages
execution(* com.example.service..*.*(..))

// Only public methods returning String in any class
execution(public String com.example..*.*(..))

// Methods with exactly one String parameter
execution(* com.example.service.*.*(String))

// Methods with any parameters starting with String
execution(* com.example.service.*.*(String, ..))

// Any method annotated with @Transactional
@annotation(org.springframework.transaction.annotation.Transactional)

// Any class annotated with @RestController
within(@org.springframework.web.bind.annotation.RestController *)

// All beans whose name matches pattern
bean(userService) or bean(*Service)
```

### Named Pointcuts — Reuse Without Repetition:
```java
@Aspect
@Component
public class AppAspects {

    // ==================== REUSABLE NAMED POINTCUTS ====================

    @Pointcut("execution(* com.example.service.*.*(..))")
    public void allServiceMethods() {}          // Reusable pointcut — name it

    @Pointcut("execution(* com.example.repository.*.*(..))")
    public void allRepositoryMethods() {}

    @Pointcut("within(@org.springframework.web.bind.annotation.RestController *)")
    public void allControllerMethods() {}

    @Pointcut("allServiceMethods() || allRepositoryMethods()")
    public void allDataAccessMethods() {}       // Combine pointcuts with ||

    // ==================== USE THE NAMED POINTCUTS IN ADVICE ====================

    @Before("allServiceMethods()")              // Reuse — clean and DRY
    public void logServiceEntry(JoinPoint jp) {
        logger.info("Service called: {}", jp.getSignature().getName());
    }

    @Around("allDataAccessMethods()")           // Applies to service + repo both
    public Object trackDataAccess(ProceedingJoinPoint jp) throws Throwable {
        logger.info("Data access: {}", jp.getSignature().getName());
        return jp.proceed();
    }
}
```

### Pointcut Expression Quick Reference:

| Expression | Matches |
|---|---|
| `execution(* com.example.service.*.*(..))` | All methods in all classes in service package |
| `execution(* com.example..*.*(..))` | All methods in service package and sub-packages |
| `execution(public * *(..))` | All public methods anywhere |
| `execution(* get*(..))` | All methods starting with `get` |
| `execution(* *(String, ..))` | Methods whose first param is String |
| `within(com.example.service.*)` | All methods within service package classes |
| `@annotation(com.example.TrackTime)` | Methods annotated with `@TrackTime` |
| `@within(org.springframework.stereotype.Service)` | Methods in classes annotated with `@Service` |
| `bean(*Service)` | All beans whose name ends with `Service` |
| `args(String, ..)` | Methods called with a String as first argument |

**Real World Analogy:** An **Aspect** is like the **security department** of a company. A **Pointcut** is the **rulebook** that says: *"Check every person entering through the main gate on floors 1–5 between 9am and 6pm."* The Aspect holds the policy; the Pointcut defines exactly where and when it applies.

---

## 6. Types of AOP Advices?

**Advice** is the **actual code that runs** when a matched join point (method) is intercepted. Spring AOP provides 5 types of advice — each running at a different point in the method lifecycle.

| Advice Type | Annotation | When It Runs |
|---|---|---|
| **Before** | `@Before` | Before the method executes |
| **After Returning** | `@AfterReturning` | After method returns successfully |
| **After Throwing** | `@AfterThrowing` | After method throws an exception |
| **After (Finally)** | `@After` | After method completes — success OR exception |
| **Around** | `@Around` | Wraps the entire method — before AND after |

### Method Lifecycle with All Advices:
```
Method Called
     │
     ▼
[@Before]          ← Runs first — before method body
     │
     ▼
  Method executes
     │
     ├──── Success ──── [@AfterReturning]   ← Only on success
     │
     ├──── Exception ── [@AfterThrowing]    ← Only on exception
     │
     ▼
[@After]           ← Always runs — like finally block
```

### 1. `@Before` — Runs Before the Method:
```java
@Before("execution(* com.example.service.*.*(..))")
public void beforeAdvice(JoinPoint joinPoint) {
    // Runs BEFORE the target method — cannot stop execution (unless throws exception)
    String method = joinPoint.getSignature().getName();
    Object[] args = joinPoint.getArgs();

    logger.info("[BEFORE] Calling: {} with args: {}", method, Arrays.toString(args));

    // Common uses: logging entry, security check, input validation
}
```

### 2. `@AfterReturning` — Runs After Successful Return:
```java
@AfterReturning(
    pointcut  = "execution(* com.example.service.UserService.createUser(..))",
    returning = "result"    // Binds the return value to this parameter
)
public void afterReturningAdvice(JoinPoint joinPoint, Object result) {
    // Runs ONLY when method completes without throwing an exception
    // 'result' holds what the method returned
    logger.info("[AFTER RETURNING] {} returned: {}",
                joinPoint.getSignature().getName(), result);

    // Common uses: audit logging, send notification after successful save
}
```

### 3. `@AfterThrowing` — Runs When Exception is Thrown:
```java
@AfterThrowing(
    pointcut = "execution(* com.example.service.*.*(..))",
    throwing = "ex"         // Binds the thrown exception to this parameter
)
public void afterThrowingAdvice(JoinPoint joinPoint, Exception ex) {
    // Runs ONLY when the method throws an exception
    // Does NOT suppress the exception — it still propagates to the caller
    logger.error("[EXCEPTION] {}.{}() threw: {} — {}",
                 joinPoint.getTarget().getClass().getSimpleName(),
                 joinPoint.getSignature().getName(),
                 ex.getClass().getSimpleName(),
                 ex.getMessage());

    // Common uses: error logging, alerting, sending to monitoring system
}
```

### 4. `@After` — Runs Always (Like `finally`):
```java
@After("execution(* com.example.service.*.*(..))")
public void afterAdvice(JoinPoint joinPoint) {
    // Runs ALWAYS — whether method succeeded or threw an exception
    // Like a finally block
    logger.info("[AFTER] Completed: {}", joinPoint.getSignature().getName());

    // Common uses: resource cleanup, releasing locks, audit trail
}
```

### 5. `@Around` — Most Powerful — Wraps Entire Execution:
```java
@Around("execution(* com.example.service.*.*(..))")
public Object aroundAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
    // Runs BEFORE AND AFTER the method
    // Must call joinPoint.proceed() to execute the actual method
    // Can modify arguments, modify return value, or prevent execution entirely

    String method = joinPoint.getSignature().toShortString();
    logger.info("[AROUND - BEFORE] {}", method);

    Object[] args = joinPoint.getArgs();
    // Can modify args: args[0] = modifiedArg;

    Object result;
    try {
        result = joinPoint.proceed(args);    // Execute the real method
        // Can modify result: result = transformedResult;
        logger.info("[AROUND - AFTER] {} returned: {}", method, result);
    } catch (Throwable ex) {
        logger.error("[AROUND - ERROR] {} threw: {}", method, ex.getMessage());
        throw ex;
    }

    return result;
}
```

### Advice Execution Order:
```
@Around (before proceed)
    → @Before
    → Target Method Executes
    → @AfterReturning (if no exception) OR @AfterThrowing (if exception)
    → @After (always)
@Around (after proceed)
```

### When to Use Which Advice:

| Use Case | Best Advice |
|---|---|
| Log method entry + arguments | `@Before` |
| Log successful result | `@AfterReturning` |
| Log / alert on exceptions | `@AfterThrowing` |
| Cleanup (always needed) | `@After` |
| Performance tracking | `@Around` |
| Retry logic | `@Around` |
| Caching | `@Around` |
| Modify input/output | `@Around` |
| Security check (block call) | `@Around` or `@Before` |

**Real World Analogy:** Think of advices as different **event triggers on a bank ATM**:
- `@Before` — card inserted, PIN prompt appears
- `@AfterReturning` — cash dispensed, print receipt
- `@AfterThrowing` — card declined, show error on screen
- `@After` — card always ejected at the end no matter what
- `@Around` — the entire ATM session controller — runs everything, can modify anything

---

## 7. What is Weaving?

**Weaving** is the **process of linking Aspects (cross-cutting concerns) with the target application code (classes/methods)**. It is how Spring AOP actually applies the advice to the target objects — inserting the aspect logic at the matched join points.

| Aspect | Description |
|---|---|
| **What it does** | Connects the Aspect to the target class/method at runtime |
| **Result** | A **proxy object** that wraps the real bean and intercepts method calls |
| **When it happens** | At application startup (for Spring AOP) |
| **Mechanism** | Spring creates a dynamic proxy (JDK or CGLIB) of the target bean |

### Types of Weaving:

| Type | When It Happens | Who Does It | Description |
|---|---|---|---|
| **Compile-time Weaving** | During `javac` compilation | AspectJ compiler (`ajc`) | Aspect code is woven directly into `.class` files at build time |
| **Load-time Weaving (LTW)** | When class is loaded by JVM | AspectJ + Java Agent | Aspect is woven as the classloader loads the `.class` file |
| **Runtime Weaving** | At application runtime | **Spring AOP** | Spring creates a dynamic **proxy** around the target bean at startup |

### How Spring AOP Runtime Weaving Works:
```
Application Startup
        │
        ▼
Spring scans for @Aspect classes
        │
        ▼
Spring scans all beans — checks if any pointcut matches
        │
        ▼
For matching beans → Spring creates a PROXY (not the real object)
        │
        ▼
Your code gets the PROXY injected (not the original bean)
        │
        ▼
Method call → Proxy intercepts → Runs advice → Calls real method
```

### Spring AOP Proxy Types:

| Proxy Type | Used When | How |
|---|---|---|
| **JDK Dynamic Proxy** | Target class implements an interface | Proxy implements the same interface |
| **CGLIB Proxy** | Target class does NOT implement an interface | Proxy extends (subclasses) the target class |

```java
// Spring auto-selects proxy type:

// Case 1: Interface present → JDK Dynamic Proxy
public interface UserService { User getUserById(int id); }

@Service
public class UserServiceImpl implements UserService {  // Spring uses JDK proxy
    public User getUserById(int id) { ... }
}

// Case 2: No interface → CGLIB proxy
@Service
public class OrderService {    // No interface — Spring uses CGLIB subclass proxy
    public Order placeOrder(Order order) { ... }
}
```

### Weaving Internals — What Actually Happens:
```java
// What you write (Target):
@Service
public class UserService {
    public User getUserById(int id) {
        return userRepository.findById(id).orElseThrow();
    }
}

// What Spring generates at runtime (CGLIB Proxy — simplified concept):
public class UserService$$SpringCGLIB extends UserService {

    @Override
    public User getUserById(int id) {
        // 1. Run @Before advice
        loggingAspect.logMethodEntry(...);

        // 2. Start @Around timing
        long start = System.currentTimeMillis();

        try {
            // 3. Call REAL method on the original object
            User result = super.getUserById(id);

            // 4. Run @AfterReturning advice
            loggingAspect.logMethodExit(result);
            return result;

        } catch (Exception ex) {
            // 5. Run @AfterThrowing advice
            loggingAspect.logException(ex);
            throw ex;
        } finally {
            // 6. Run @After advice
            loggingAspect.logCompletion();
            logger.info("Took: {} ms", System.currentTimeMillis() - start);
        }
    }
}
```

### Important Weaving Limitation — Self-Invocation:
```java
@Service
public class PaymentService {

    public void processPayment(Order order) {
        validateOrder(order);   // ⚠️ INTERNAL call — AOP does NOT intercept this!
        chargeCard(order);
    }

    @Transactional              // @Transactional on this method
    public void chargeCard(Order order) {
        // This @Transactional WON'T work when called from processPayment() above
        // because it's a direct Java call — bypasses the proxy entirely
    }
}
// FIX: Inject PaymentService into itself, or move chargeCard() to a separate bean
```

> **Key Rule:** Spring AOP only intercepts method calls that go **through the proxy**. Calls from within the same class bypass the proxy and are NOT intercepted.

**Real World Analogy:** Weaving is like **installing a smart switchboard** in an office. Every phone call (method call) goes through the switchboard (proxy). The switchboard can record calls (logging aspect), check authorization (security aspect), or time the call (performance aspect) — all before connecting to the actual recipient (real method). Direct intercom calls within the same desk (self-invocation) bypass the switchboard entirely.

---

## 8. Spring AOP vs AspectJ?

**Spring AOP** and **AspectJ** are both AOP frameworks for Java, but they differ fundamentally in how and when they apply aspects, what join points they support, and the complexity of setup.

| Aspect | Spring AOP | AspectJ |
|---|---|---|
| **Type** | Runtime proxy-based AOP | Full AOP framework (compile/load/runtime) |
| **Weaving** | Runtime (proxy) | Compile-time, Load-time, or Runtime |
| **Join Points** | Method execution only | Method, constructor, field access, static initializer, and more |
| **Proxy requirement** | Must go through Spring proxy | No proxy — directly modifies bytecode |
| **Self-invocation** | ❌ Does NOT work | ✅ Works (no proxy limitation) |
| **Performance** | Slight overhead (proxy dispatch) | Faster — woven into bytecode directly |
| **Setup complexity** | ✅ Simple — just `spring-boot-starter-aop` | ⚠️ Complex — needs AspectJ compiler or agent |
| **Spring integration** | ✅ Native, seamless | ⚠️ Needs `@EnableLoadTimeWeaving` or compile setup |
| **Annotation syntax** | Same AspectJ annotations (`@Aspect`, `@Before`, etc.) | Same annotations — just different weaving |
| **Use case** | Most enterprise Spring applications | Complex AOP needs — fields, constructors, self-calls |

### Spring AOP — Simple Setup:
```java
// Just add dependency + annotations — works out of the box
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void log(JoinPoint jp) {
        logger.info("Method called: {}", jp.getSignature().getName());
    }
}
// application.properties: spring.aop.auto=true (default — no config needed)
```

### AspectJ — Compile-Time Weaving Setup:
```xml
<!-- pom.xml — AspectJ compiler plugin needed -->
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <configuration>
        <complianceLevel>17</complianceLevel>
        <source>17</source>
        <target>17</target>
    </configuration>
    <executions>
        <execution>
            <goals><goal>compile</goal></goals>
        </execution>
    </executions>
</plugin>
```

```java
// AspectJ can intercept field access, constructors, static blocks — Spring AOP cannot
@Aspect
public class FieldAccessAspect {

    // ✅ Intercept field READ — NOT possible in Spring AOP
    @Before("get(private String com.example.User.email)")
    public void beforeFieldRead(JoinPoint jp) {
        logger.info("Reading field: email");
    }

    // ✅ Intercept constructor — NOT possible in Spring AOP
    @Before("execution(com.example.User.new(..))")
    public void beforeConstructor(JoinPoint jp) {
        logger.info("User object being created");
    }
}
```

### Spring AOP using AspectJ Annotations:
```java
// Spring AOP BORROWS AspectJ's annotation syntax (@Aspect, @Before, @Around, etc.)
// but uses Spring's proxy-based runtime weaving — NOT AspectJ's compiler/agent
// This is the most common setup in Spring Boot projects

@Aspect       // AspectJ annotation — but woven by Spring's proxy mechanism
@Component
public class TransactionLoggingAspect {

    @Around("@annotation(org.springframework.transaction.annotation.Transactional)")
    public Object logTransactions(ProceedingJoinPoint jp) throws Throwable {
        logger.info("Transaction started for: {}", jp.getSignature().getName());
        Object result = jp.proceed();
        logger.info("Transaction committed for: {}", jp.getSignature().getName());
        return result;
    }
}
```

### When to Use Which:

| Scenario | Recommendation |
|---|---|
| Standard Spring Boot enterprise application | **Spring AOP** — simple, sufficient for 95% of use cases |
| Need to intercept field access or constructors | **AspectJ** |
| Need AOP to work on self-invocation inside same class | **AspectJ** |
| Non-Spring Java application needs AOP | **AspectJ** |
| Maximum performance with complex AOP rules | **AspectJ (compile-time)** |
| Quick setup, team familiar with Spring | **Spring AOP** |

> **Best Practice:** Start with **Spring AOP**. It covers method-level interception for logging, security, caching, and transactions — the vast majority of real-world AOP needs. Switch to **AspectJ** only when you hit Spring AOP's limitations (self-invocation, field interception, constructor interception).

**Real World Analogy:** Spring AOP is like **CCTV at the building entrance** — it monitors everyone who enters and exits through the main door (proxy). AspectJ is like **body cameras on every employee** — it tracks every single action (field access, object creation, method call) regardless of location, but requires much more setup and resources to deploy.

---

*Happy Learning Spring AOP! 🔀*
