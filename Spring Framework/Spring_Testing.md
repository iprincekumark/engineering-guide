# Testing (Unit & Integration) — Interview Questions & Answers

---

## 1. Unit Testing in Spring?

**Unit Testing** is the practice of **testing individual units of code (methods/classes) in isolation** — without starting a Spring context, hitting a real database, or calling real external services. Dependencies are replaced with **mocks** so only the logic of the class under test is verified.

| Aspect | Description |
|---|---|
| **Goal** | Test one class/method in complete isolation |
| **Speed** | Very fast — no Spring context, no DB, no network |
| **Dependencies** | Replaced with mocks (fake objects) |
| **Annotation** | `@ExtendWith(MockitoExtension.class)` — JUnit 5 |
| **Framework** | JUnit 5 + Mockito |
| **What to test** | Service layer logic, utility methods, business rules |

### Dependencies:
```xml
<!-- Spring Boot Test Starter — includes JUnit 5 + Mockito + AssertJ + MockMvc -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### Example — Unit Test for a Service:
```java
// Service under test
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public User getUserById(int id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found: " + id));
    }

    public User createUser(User user) {
        if (user.getName() == null || user.getName().isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        return userRepository.save(user);
    }
}

// Unit Test — NO Spring context loaded, UserRepository is mocked
@ExtendWith(MockitoExtension.class)         // Activates Mockito annotations
class UserServiceTest {

    @Mock                                   // Creates a mock of UserRepository
    private UserRepository userRepository;

    @InjectMocks                            // Injects mocks into UserService
    private UserService userService;

    @Test
    void getUserById_ShouldReturnUser_WhenUserExists() {
        // ARRANGE — define mock behavior
        User mockUser = new User(1, "Alice", "alice@example.com");
        Mockito.when(userRepository.findById(1))
               .thenReturn(Optional.of(mockUser));

        // ACT — call the method under test
        User result = userService.getUserById(1);

        // ASSERT — verify the result
        assertEquals("Alice", result.getName());
        assertEquals("alice@example.com", result.getEmail());
    }

    @Test
    void getUserById_ShouldThrowException_WhenUserNotFound() {
        // ARRANGE
        Mockito.when(userRepository.findById(99))
               .thenReturn(Optional.empty());

        // ACT + ASSERT
        assertThrows(RuntimeException.class,
            () -> userService.getUserById(99));
    }

    @Test
    void createUser_ShouldThrowException_WhenNameIsBlank() {
        User blankUser = new User(0, "", "test@example.com");

        assertThrows(IllegalArgumentException.class,
            () -> userService.createUser(blankUser));

        // Verify repository was NEVER called (short-circuited by validation)
        Mockito.verify(userRepository, Mockito.never()).save(any());
    }
}
```

### Unit Test vs Integration Test:

| Aspect | Unit Test | Integration Test |
|---|---|---|
| **Scope** | Single class/method | Multiple layers together |
| **Spring Context** | ❌ Not loaded | ✅ Loaded (full or partial) |
| **Database** | ❌ Mocked | ✅ Real or in-memory (H2) |
| **Speed** | ⚡ Very fast (ms) | 🐢 Slower (seconds) |
| **Annotation** | `@ExtendWith(MockitoExtension.class)` | `@SpringBootTest` |
| **Purpose** | Test logic in isolation | Test components working together |

**Real World Analogy:** Unit testing is like testing **each car part individually** — engine on a test bench, brakes on a rig — before assembling the car. You isolate each part to verify it works on its own.

---

## 2. What is Mockito?

**Mockito** is the **most popular Java mocking framework** used in unit testing. It allows you to create **fake (mock) objects** that simulate the behavior of real dependencies, so you can test a class in complete isolation without needing a real database, file system, or external service.

| Aspect | Description |
|---|---|
| **What it does** | Creates mock objects that mimic real dependencies |
| **Core feature** | Stub method return values + verify interactions |
| **Bundled with** | `spring-boot-starter-test` (no extra dependency needed) |
| **Works with** | JUnit 5 via `@ExtendWith(MockitoExtension.class)` |
| **Key package** | `org.mockito.Mockito` |

### Core Mockito Operations:

```java
// 1. CREATE a mock
UserRepository mockRepo = Mockito.mock(UserRepository.class);

// 2. STUB — define what the mock returns when called
Mockito.when(mockRepo.findById(1))
       .thenReturn(Optional.of(new User(1, "Alice", "alice@test.com")));

// 3. STUB — simulate throwing an exception
Mockito.when(mockRepo.findById(99))
       .thenThrow(new RuntimeException("User not found"));

// 4. STUB void methods — doNothing / doThrow
Mockito.doNothing().when(mockRepo).deleteById(1);
Mockito.doThrow(new RuntimeException()).when(mockRepo).deleteById(99);

// 5. VERIFY — confirm a method was called with specific arguments
Mockito.verify(mockRepo).findById(1);                        // Called exactly once
Mockito.verify(mockRepo, Mockito.times(2)).findById(1);      // Called exactly twice
Mockito.verify(mockRepo, Mockito.never()).deleteById(anyInt()); // Never called
Mockito.verify(mockRepo, Mockito.atLeastOnce()).findAll();   // Called at least once

// 6. ARGUMENT MATCHERS — flexible matching
Mockito.when(mockRepo.findById(Mockito.anyInt()))
       .thenReturn(Optional.of(new User()));

Mockito.when(mockRepo.save(Mockito.any(User.class)))
       .thenReturn(new User(1, "Bob", "bob@test.com"));
```

### Mockito Answer — Dynamic Return Values:
```java
// Return different values based on input
Mockito.when(mockRepo.findById(Mockito.anyInt()))
       .thenAnswer(invocation -> {
           int id = invocation.getArgument(0);
           return id > 0
               ? Optional.of(new User(id, "User" + id, "user" + id + "@test.com"))
               : Optional.empty();
       });
```

**Real World Analogy:** Mockito is like a **stunt double on a movie set**. Instead of the real actor (database), the stunt double (mock) performs scripted actions on cue — you define exactly what it does and verify it behaved as expected.

---

## 3. Favorite Mocking Framework?

**Mockito** is the industry-standard and most widely used mocking framework for Java — and the preferred choice for Spring Boot projects.

| Framework | Description | Verdict |
|---|---|---|
| **Mockito** | Most popular, clean API, deep Spring Boot integration | ✅ Industry standard — go-to choice |
| **EasyMock** | Older framework, record-replay model, more verbose | ⚠️ Less popular today |
| **PowerMock** | Extends Mockito/EasyMock to mock static/final/private | ⚠️ Use only when necessary — indicates design smell |
| **JMockit** | Powerful but complex API, less community support | ⚠️ Niche use only |
| **WireMock** | Mocks HTTP services (REST APIs, external services) | ✅ Best for HTTP-level integration testing |

### Why Mockito is the Favorite:

| Reason | Detail |
|---|---|
| **Bundled with Spring Boot** | Included in `spring-boot-starter-test` — zero extra setup |
| **Clean, readable API** | `when(...).thenReturn(...)` reads like plain English |
| **Annotation support** | `@Mock`, `@InjectMocks`, `@Spy`, `@Captor` — minimal boilerplate |
| **Spring integration** | `@MockBean` replaces beans in Spring context seamlessly |
| **Active community** | Most StackOverflow answers, tutorials, and docs use Mockito |
| **BDD support** | `BDDMockito.given(...).willReturn(...)` for BDD-style tests |

### BDD Style with Mockito (Behavior-Driven Development):
```java
import static org.mockito.BDDMockito.*;

@Test
void shouldReturnUserWhenFound() {
    // GIVEN
    given(userRepository.findById(1))
        .willReturn(Optional.of(new User(1, "Alice", "alice@test.com")));

    // WHEN
    User result = userService.getUserById(1);

    // THEN
    then(userRepository).should().findById(1);
    assertEquals("Alice", result.getName());
}
```

---

## 4. Mock Data with Mockito?

**Mocking data** means **defining what a mock object should return** when specific methods are called — so your test runs against predictable, controlled fake data instead of a real database or service.

### Stubbing Return Values:
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private OrderService orderService;

    // --- Stub single object ---
    @Test
    void shouldReturnOrder() {
        Order mockOrder = new Order(1, "PENDING", 250.0);
        when(orderRepository.findById(1))
            .thenReturn(Optional.of(mockOrder));

        Order result = orderService.getOrder(1);
        assertEquals("PENDING", result.getStatus());
    }

    // --- Stub a List ---
    @Test
    void shouldReturnAllOrders() {
        List<Order> mockOrders = List.of(
            new Order(1, "PENDING", 100.0),
            new Order(2, "SHIPPED", 200.0),
            new Order(3, "DELIVERED", 300.0)
        );
        when(orderRepository.findAll()).thenReturn(mockOrders);

        List<Order> result = orderService.getAllOrders();
        assertEquals(3, result.size());
        assertEquals("SHIPPED", result.get(1).getStatus());
    }

    // --- Stub consecutive calls (different result each time) ---
    @Test
    void shouldReturnDifferentResultsOnConsecutiveCalls() {
        when(orderRepository.findById(1))
            .thenReturn(Optional.of(new Order(1, "PENDING", 100.0)))   // 1st call
            .thenReturn(Optional.of(new Order(1, "SHIPPED", 100.0)));  // 2nd call

        assertEquals("PENDING", orderService.getOrder(1).getStatus());
        assertEquals("SHIPPED", orderService.getOrder(1).getStatus());
    }

    // --- Stub void method (e.g., send email) ---
    @Test
    void shouldCallNotificationOnSave() {
        Order order = new Order(1, "PENDING", 100.0);
        when(orderRepository.save(any(Order.class))).thenReturn(order);

        orderService.placeOrder(order);

        // Verify the repo's save() was called with exactly this order
        verify(orderRepository).save(order);
    }

    // --- Stub to throw exception ---
    @Test
    void shouldHandleRepositoryException() {
        when(orderRepository.findById(anyInt()))
            .thenThrow(new RuntimeException("DB connection failed"));

        assertThrows(RuntimeException.class,
            () -> orderService.getOrder(1));
    }
}
```

### ArgumentCaptor — Capture and Inspect Passed Data:
```java
@Test
void shouldSaveOrderWithCorrectStatus() {
    ArgumentCaptor<Order> orderCaptor = ArgumentCaptor.forClass(Order.class);

    orderService.placeOrder(new Order(0, null, 150.0));

    verify(orderRepository).save(orderCaptor.capture());

    Order savedOrder = orderCaptor.getValue();
    assertEquals("PENDING", savedOrder.getStatus());  // Verify default status was set
    assertEquals(150.0, savedOrder.getAmount());
}
```

**Real World Analogy:** Mocking data is like a **flight simulator** for pilots. Instead of flying a real plane (hitting a real DB), the simulator (mock) responds with scripted scenarios — bad weather, engine failure — so you can test every condition safely.

---

## 5. Mocking Annotations?

Mockito provides **annotations** to reduce boilerplate when creating and injecting mocks, making test classes clean and readable.

| Annotation | Purpose |
|---|---|
| `@Mock` | Creates a full mock — all methods return default values (null/0/false) unless stubbed |
| `@InjectMocks` | Creates the class under test and **injects all `@Mock` fields** into it automatically |
| `@Spy` | Creates a partial mock — **real methods are called** unless explicitly stubbed |
| `@Captor` | Creates an `ArgumentCaptor` to capture method arguments for assertion |
| `@MockBean` | Spring Boot annotation — replaces a bean in the Spring context with a mock |

### `@Mock` — Full Mock:
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;  // Every method returns null/empty by default

    @InjectMocks
    private UserService userService;        // UserService gets userRepository injected

    @Test
    void testFindById() {
        when(userRepository.findById(1))
            .thenReturn(Optional.of(new User(1, "Alice", "alice@test.com")));

        User user = userService.getUserById(1);
        assertEquals("Alice", user.getName());
    }
}
```

### `@Spy` — Partial Mock (Real + Stubbed):
```java
@ExtendWith(MockitoExtension.class)
class EmailServiceTest {

    @Spy
    private EmailService emailService = new EmailService(); // Real object, partially mocked

    @Test
    void testSendEmail() {
        // Stub only the external call — real logic runs for everything else
        doNothing().when(emailService).sendEmail(anyString());

        emailService.notifyUser("alice@test.com");

        verify(emailService).sendEmail("alice@test.com"); // Verify it was called
    }
}
```

### `@Captor` — Capture Arguments:
```java
@ExtendWith(MockitoExtension.class)
class NotificationServiceTest {

    @Mock
    private EmailSender emailSender;

    @Captor
    private ArgumentCaptor<String> emailCaptor;  // Captures String argument passed to sendEmail

    @InjectMocks
    private NotificationService notificationService;

    @Test
    void shouldSendEmailToCorrectAddress() {
        notificationService.sendWelcomeEmail(new User(1, "Alice", "alice@test.com"));

        verify(emailSender).send(emailCaptor.capture());
        assertEquals("alice@test.com", emailCaptor.getValue());
    }
}
```

### `@Mock` vs `@Spy`:

| Aspect | `@Mock` | `@Spy` |
|---|---|---|
| **Real method called?** | ❌ No — returns null/0/false | ✅ Yes — calls real implementation |
| **When to use** | When dependency is external (DB, API) | When class has logic you want to keep real |
| **Stubbing** | Must stub all methods you need | Only stub the methods you want to override |
| **Default behavior** | Returns empty/null | Delegates to real method |

---

## 6. What is MockMvc?

**`MockMvc`** is a **Spring Test framework class that simulates HTTP requests to your controller layer** without starting a real web server. It lets you test your REST endpoints — request mapping, request/response format, status codes, headers — in a fast, in-memory way.

| Aspect | Description |
|---|---|
| **What it tests** | Controller layer (REST endpoints) |
| **Web server** | ❌ Not started — simulated in-memory |
| **Spring context** | ✅ Partial (only web layer) with `@WebMvcTest` |
| **Speed** | ⚡ Fast — no real HTTP, no port binding |
| **Key class** | `MockMvc` from `org.springframework.test.web.servlet` |

### MockMvc Core Methods:

| Method | Purpose |
|---|---|
| `mockMvc.perform(get("/url"))` | Simulate a GET request |
| `mockMvc.perform(post("/url").content(...))` | Simulate a POST with body |
| `.andExpect(status().isOk())` | Assert HTTP status = 200 |
| `.andExpect(jsonPath("$.name").value("Alice"))` | Assert JSON field value |
| `.andExpect(content().contentType(MediaType.APPLICATION_JSON))` | Assert content type |
| `.andReturn()` | Get the result for further assertions |

### Example — Testing a REST Controller:
```java
// Controller under test
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable int id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User saved = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable int id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}

// MockMvc Test
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;  // For serializing objects to JSON

    // --- Test GET ---
    @Test
    void getUser_ShouldReturn200_WhenUserExists() throws Exception {
        User mockUser = new User(1, "Alice", "alice@test.com");
        when(userService.getUserById(1)).thenReturn(mockUser);

        mockMvc.perform(get("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())                            // HTTP 200
            .andExpect(jsonPath("$.id").value(1))                  // id = 1
            .andExpect(jsonPath("$.name").value("Alice"))          // name = "Alice"
            .andExpect(jsonPath("$.email").value("alice@test.com"))// email matches
            .andDo(print());                                       // Print request/response to console
    }

    // --- Test POST ---
    @Test
    void createUser_ShouldReturn201_WhenUserCreated() throws Exception {
        User inputUser  = new User(0, "Bob", "bob@test.com");
        User savedUser  = new User(2, "Bob", "bob@test.com");
        when(userService.createUser(any(User.class))).thenReturn(savedUser);

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(inputUser)))
            .andExpect(status().isCreated())                       // HTTP 201
            .andExpect(jsonPath("$.id").value(2))
            .andExpect(jsonPath("$.name").value("Bob"));
    }

    // --- Test DELETE ---
    @Test
    void deleteUser_ShouldReturn204_WhenDeleted() throws Exception {
        doNothing().when(userService).deleteUser(1);

        mockMvc.perform(delete("/api/users/1"))
            .andExpect(status().isNoContent());                    // HTTP 204
    }
}
```

**Real World Analogy:** MockMvc is like a **Postman simulator built into your test suite**. Instead of manually firing HTTP requests, MockMvc sends them in code — automatically, repeatably, and without needing a running server.

---

## 7. What is @WebMvcTest?

**`@WebMvcTest`** is a **Spring Boot test slice annotation** that loads **only the web layer** (controllers, filters, `@ControllerAdvice`, `Jackson` converters) into the Spring context — without loading repositories, services, or the full application context. It is used with `MockMvc` to test controllers in isolation.

| Aspect | Description |
|---|---|
| **What it loads** | Controllers, `@ControllerAdvice`, `WebMvcConfigurer`, filters |
| **What it does NOT load** | `@Service`, `@Repository`, `@Component`, JPA/DB config |
| **Service layer** | Must be provided as `@MockBean` |
| **Speed** | ⚡ Fast — partial context, no DB |
| **Auto-configures** | `MockMvc` — inject with `@Autowired` |

```java
// Load only UserController's web layer — nothing else
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;           // Auto-configured by @WebMvcTest

    @MockBean                          // UserService is NOT in web layer — must mock it
    private UserService userService;

    @Test
    void shouldReturn404_WhenUserNotFound() throws Exception {
        when(userService.getUserById(99))
            .thenThrow(new RuntimeException("User not found"));

        mockMvc.perform(get("/api/users/99"))
            .andExpect(status().isInternalServerError()); // Or 404 if mapped in @ControllerAdvice
    }
}
```

### @WebMvcTest vs @SpringBootTest:

| Aspect | `@WebMvcTest` | `@SpringBootTest` |
|---|---|---|
| **Context loaded** | Web layer only | Full application context |
| **Database** | ❌ Not loaded | ✅ Loaded (real or H2) |
| **Speed** | ⚡ Fast | 🐢 Slower |
| **Use case** | Unit test controllers | Integration test full app |
| **MockMvc** | Auto-configured | Must add `@AutoConfigureMockMvc` |

> **Best Practice:** Use `@WebMvcTest` for controller unit tests. Use `@SpringBootTest` for full integration tests that need the complete stack.

**Real World Analogy:** `@WebMvcTest` is like testing only the **front door and reception desk** of a hotel — you verify the entry point works correctly, without setting up the kitchen, rooms, or housekeeping (service/repository layers).

---

## 8. What is @MockBean?

**`@MockBean`** is a **Spring Boot test annotation** that creates a Mockito mock and **registers it as a bean in the Spring application context**, replacing the real bean. It is used when a test loads a Spring context (via `@WebMvcTest` or `@SpringBootTest`) and you need to mock a dependency within that context.

| Aspect | Description |
|---|---|
| **What it does** | Creates a Mockito mock AND registers it in the Spring context |
| **Replaces** | The real Spring bean with a mock for the duration of the test |
| **Used with** | `@WebMvcTest`, `@SpringBootTest` — any test that loads a Spring context |
| **Difference from `@Mock`** | `@Mock` is pure Mockito (no Spring context); `@MockBean` is Spring-aware |

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean                        // Replaces the real UserService bean in context
    private UserService userService;

    @MockBean                        // Replaces the real EmailService bean in context
    private EmailService emailService;

    @Test
    void shouldReturnUser() throws Exception {
        when(userService.getUserById(1))
            .thenReturn(new User(1, "Alice", "alice@test.com"));

        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

### `@Mock` vs `@MockBean`:

| Aspect | `@Mock` | `@MockBean` |
|---|---|---|
| **Framework** | Pure Mockito | Spring Boot Test |
| **Spring context** | ❌ No context needed | ✅ Registers in Spring context |
| **Used with** | `@ExtendWith(MockitoExtension.class)` | `@WebMvcTest` / `@SpringBootTest` |
| **Replaces Spring bean** | ❌ No | ✅ Yes — replaces real bean |
| **Use case** | Service/unit tests without Spring | Controller tests with Spring context |

> **Rule of Thumb:** If your test uses `@WebMvcTest` or `@SpringBootTest` — use `@MockBean`. If your test uses only `@ExtendWith(MockitoExtension.class)` — use `@Mock`.

---

## 9. Unit Test with MockMvc?

A **complete MockMvc unit test** combines `@WebMvcTest` + `@MockBean` + `MockMvc` to test the full HTTP lifecycle of a controller — request parsing, response serialization, status codes, and JSON output — all without a running server.

### Complete Example — CRUD Controller Test:
```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProductService productService;

    @Autowired
    private ObjectMapper objectMapper;

    // ==================== GET ALL ====================
    @Test
    void getAllProducts_ShouldReturn200WithList() throws Exception {
        List<Product> products = List.of(
            new Product(1, "Laptop", 999.99, "Electronics"),
            new Product(2, "Phone",  499.99, "Electronics")
        );
        when(productService.getAllProducts()).thenReturn(products);

        mockMvc.perform(get("/api/products")
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.length()").value(2))
            .andExpect(jsonPath("$[0].name").value("Laptop"))
            .andExpect(jsonPath("$[1].price").value(499.99))
            .andDo(print());
    }

    // ==================== GET BY ID ====================
    @Test
    void getProductById_ShouldReturn200_WhenFound() throws Exception {
        Product product = new Product(1, "Laptop", 999.99, "Electronics");
        when(productService.getProductById(1)).thenReturn(product);

        mockMvc.perform(get("/api/products/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("Laptop"))
            .andExpect(jsonPath("$.category").value("Electronics"));
    }

    @Test
    void getProductById_ShouldReturn404_WhenNotFound() throws Exception {
        when(productService.getProductById(99))
            .thenThrow(new ProductNotFoundException("Product not found: 99"));

        mockMvc.perform(get("/api/products/99"))
            .andExpect(status().isNotFound());
    }

    // ==================== POST ====================
    @Test
    void createProduct_ShouldReturn201_WithSavedProduct() throws Exception {
        Product input  = new Product(0,  "Tablet", 299.99, "Electronics");
        Product saved  = new Product(3,  "Tablet", 299.99, "Electronics");
        when(productService.createProduct(any(Product.class))).thenReturn(saved);

        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(input)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(3))
            .andExpect(jsonPath("$.name").value("Tablet"));
    }

    // ==================== PUT ====================
    @Test
    void updateProduct_ShouldReturn200_WithUpdatedProduct() throws Exception {
        Product updated = new Product(1, "Laptop Pro", 1199.99, "Electronics");
        when(productService.updateProduct(eq(1), any(Product.class))).thenReturn(updated);

        mockMvc.perform(put("/api/products/1")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updated)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Laptop Pro"))
            .andExpect(jsonPath("$.price").value(1199.99));
    }

    // ==================== DELETE ====================
    @Test
    void deleteProduct_ShouldReturn204() throws Exception {
        doNothing().when(productService).deleteProduct(1);

        mockMvc.perform(delete("/api/products/1"))
            .andExpect(status().isNoContent());

        verify(productService, times(1)).deleteProduct(1);
    }
}
```

### Common MockMvc Result Matchers:

| Matcher | Assertion |
|---|---|
| `status().isOk()` | HTTP 200 |
| `status().isCreated()` | HTTP 201 |
| `status().isNoContent()` | HTTP 204 |
| `status().isNotFound()` | HTTP 404 |
| `status().isBadRequest()` | HTTP 400 |
| `status().isInternalServerError()` | HTTP 500 |
| `jsonPath("$.name").value("X")` | JSON field equals value |
| `jsonPath("$.length()").value(3)` | Array has 3 elements |
| `content().json("{...}")` | Full JSON body match |
| `header().string("Location", "/api/...")` | Response header check |

---

## 10. What is JSONAssert?

**`JSONAssert`** is a **Java library for asserting JSON equality in tests**. Instead of checking field-by-field with `jsonPath`, it lets you compare an entire JSON string against expected JSON — with control over whether the comparison is strict or lenient.

| Aspect | Description |
|---|---|
| **What it does** | Compares two JSON strings for equality |
| **Strict mode** | Field order and exact key set must match |
| **Lenient mode** | Only checks that expected fields exist — extra fields allowed |
| **Bundled with** | `spring-boot-starter-test` — no extra dependency |
| **Key class** | `org.skyscreamer.jsonassert.JSONAssert` |

### JSONAssert Modes:

| Mode | Constant | Behavior |
|---|---|---|
| **Strict** | `JSONCompareMode.STRICT` | Exact match — same keys, same values, same array order |
| **Lenient** | `JSONCompareMode.NON_EXTENSIBLE` | Expected keys must exist, extra keys in actual are allowed |
| **Lenient (arrays)** | `JSONCompareMode.LENIENT` | Extra keys AND extra array elements allowed |

### Example:
```java
import org.skyscreamer.jsonassert.JSONAssert;
import org.skyscreamer.jsonassert.JSONCompareMode;

class JSONAssertTest {

    // ==================== STRICT mode ====================
    @Test
    void strictMatch_ShouldPass_WhenExactMatch() throws JSONException {
        String actual   = "{\"id\":1, \"name\":\"Alice\", \"email\":\"alice@test.com\"}";
        String expected = "{\"id\":1, \"name\":\"Alice\", \"email\":\"alice@test.com\"}";

        JSONAssert.assertEquals(expected, actual, JSONCompareMode.STRICT); // PASS
    }

    @Test
    void strictMatch_ShouldFail_WhenExtraField() throws JSONException {
        String actual   = "{\"id\":1, \"name\":\"Alice\", \"extra\":\"field\"}";
        String expected = "{\"id\":1, \"name\":\"Alice\"}";

        JSONAssert.assertEquals(expected, actual, JSONCompareMode.STRICT); // FAIL — extra field not allowed
    }

    // ==================== LENIENT mode ====================
    @Test
    void lenientMatch_ShouldPass_WhenExtraFieldPresent() throws JSONException {
        String actual   = "{\"id\":1, \"name\":\"Alice\", \"extra\":\"field\"}";
        String expected = "{\"id\":1, \"name\":\"Alice\"}";

        JSONAssert.assertEquals(expected, actual, JSONCompareMode.LENIENT); // PASS — extra fields allowed
    }

    // ==================== Used with MockMvc ====================
    @Test
    void shouldMatchFullResponseBody() throws Exception {
        User mockUser = new User(1, "Alice", "alice@test.com");
        when(userService.getUserById(1)).thenReturn(mockUser);

        MvcResult result = mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andReturn();

        String actualJson   = result.getResponse().getContentAsString();
        String expectedJson = "{\"id\":1,\"name\":\"Alice\",\"email\":\"alice@test.com\"}";

        JSONAssert.assertEquals(expectedJson, actualJson, JSONCompareMode.LENIENT);
    }
}
```

**Real World Analogy:** `JSONAssert` is like a **document diff tool**. Strict mode is like saying *"these two contracts must be word-for-word identical"*. Lenient mode is like saying *"the contract must contain these key clauses — extra clauses are fine"*.

---

## 11. Integration Test in Spring Boot?

**Integration Testing** verifies that **multiple layers of the application work correctly together** — Controller → Service → Repository → Database. Unlike unit tests, integration tests load the full (or partial) Spring context and may use a real or in-memory database.

| Aspect | Description |
|---|---|
| **Goal** | Test the full request-response cycle end-to-end |
| **Spring context** | ✅ Fully loaded |
| **Database** | Real DB or in-memory H2 |
| **Speed** | 🐢 Slower than unit tests |
| **Annotation** | `@SpringBootTest` |
| **HTTP calls** | Via `TestRestTemplate` or `MockMvc` (with `@AutoConfigureMockMvc`) |

### H2 In-Memory Database for Tests:
```xml
<!-- pom.xml — H2 only on test classpath -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

```properties
# src/test/resources/application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=create-drop     # Fresh schema for each test run
spring.jpa.show-sql=true
```

### Integration Test Example:
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")                        // Use application-test.properties
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class UserIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @LocalServerPort
    private int port;

    private String baseUrl() {
        return "http://localhost:" + port + "/api/users";
    }

    @Test
    @Order(1)
    void createUser_ShouldReturn201() {
        User newUser = new User(0, "Alice", "alice@test.com");

        ResponseEntity<User> response = restTemplate.postForEntity(
            baseUrl(), newUser, User.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertTrue(response.getBody().getId() > 0);
        assertEquals("Alice", response.getBody().getName());
    }

    @Test
    @Order(2)
    void getAllUsers_ShouldReturn200WithUsers() {
        ResponseEntity<List> response = restTemplate.getForEntity(
            baseUrl(), List.class);

        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertFalse(response.getBody().isEmpty());
    }
}
```

### Unit Test vs Integration Test — When to Use What:

| Scenario | Recommended Test Type |
|---|---|
| Testing service business logic | Unit Test (`@ExtendWith(MockitoExtension.class)`) |
| Testing controller request/response | Unit Test (`@WebMvcTest`) |
| Testing repository queries against DB | Integration Test (`@DataJpaTest`) |
| Testing full API end-to-end | Integration Test (`@SpringBootTest`) |
| Testing external service calls | Unit Test + WireMock |

**Real World Analogy:** Integration testing is like a **full dress rehearsal** before opening night — all actors (layers), props (database), and stage (server) are present and working together, not just individual actors practicing their lines alone.

---

## 12. What is @SpringBootTest?

**`@SpringBootTest`** is a **Spring Boot annotation that loads the complete application context** for integration tests — scanning all beans, auto-configurations, and properties just like the real application startup.

| Aspect | Description |
|---|---|
| **What it loads** | Full application context (all beans, configs, DB) |
| **Web environment** | Configurable — mock, random port, defined port |
| **Use case** | Integration tests, end-to-end tests |
| **Slow?** | Yes — full context startup takes time |

### WebEnvironment Options:

| Option | Behavior | Use Case |
|---|---|---|
| `MOCK` (default) | Mock web environment — no real server | Use with `MockMvc` + `@AutoConfigureMockMvc` |
| `RANDOM_PORT` | Starts real server on a random available port | Use with `TestRestTemplate` |
| `DEFINED_PORT` | Starts real server on `server.port` (default 8080) | Rarely used in tests |
| `NONE` | No web environment — no server at all | Service/repository integration tests |

```java
// Option 1: MOCK web environment + MockMvc
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldReturnUserList() throws Exception {
        mockMvc.perform(get("/api/users"))
            .andExpect(status().isOk());
    }
}

// Option 2: Real server on RANDOM_PORT + TestRestTemplate
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserApiIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @LocalServerPort
    private int port;

    @Test
    void shouldReturnUserList() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "http://localhost:" + port + "/api/users", String.class);

        assertEquals(HttpStatus.OK, response.getStatusCode());
    }
}

// Option 3: No web — test services/repos directly
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)
class UserServiceIntegrationTest {

    @Autowired
    private UserService userService;  // Real beans, real DB

    @Test
    void shouldSaveAndRetrieveUser() {
        User saved = userService.createUser(new User(0, "Alice", "alice@test.com"));
        assertNotNull(saved.getId());

        User found = userService.getUserById(saved.getId());
        assertEquals("Alice", found.getName());
    }
}
```

> **Note:** `@SpringBootTest` loads the **entire** application context. For faster tests, prefer `@WebMvcTest` (web layer only) or `@DataJpaTest` (JPA layer only) when you don't need the full stack.

**Real World Analogy:** `@SpringBootTest` is like doing a **complete systems check** on a spacecraft — every component, every integration, every subsystem is powered on and verified together before launch.

---

## 13. What is @LocalServerPort?

**`@LocalServerPort`** is a **Spring Boot test annotation** that **injects the actual port number** on which the embedded test server started. It is used when `@SpringBootTest(webEnvironment = RANDOM_PORT)` starts a server on a random port — you need to know that port to build the correct base URL for your HTTP test calls.

| Aspect | Description |
|---|---|
| **What it injects** | The actual runtime port the embedded server is running on |
| **Used with** | `@SpringBootTest(webEnvironment = RANDOM_PORT)` |
| **Why random port?** | Avoids port conflicts in CI/CD environments where port 8080 may be busy |
| **Equivalent to** | `@Value("${local.server.port}")` |

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ProductApiTest {

    @LocalServerPort                     // Injects the actual port — e.g. 54231
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    // Build base URL dynamically using the injected port
    private String url(String path) {
        return "http://localhost:" + port + path;
    }

    @Test
    void getProduct_ShouldReturn200() {
        ResponseEntity<Product> response = restTemplate.getForEntity(
            url("/api/products/1"), Product.class);

        assertEquals(HttpStatus.OK, response.getStatusCode());
    }

    @Test
    void createProduct_ShouldReturn201() {
        Product input = new Product(0, "Keyboard", 79.99, "Accessories");

        ResponseEntity<Product> response = restTemplate.postForEntity(
            url("/api/products"), input, Product.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertTrue(response.getBody().getId() > 0);
    }
}
```

> **Why not hardcode port 8080?** In CI/CD pipelines, port 8080 might already be in use by another process, causing tests to fail. `RANDOM_PORT` + `@LocalServerPort` guarantees a free port is always used.

---

## 14. What is TestRestTemplate?

**`TestRestTemplate`** is a **Spring Boot test utility class** that wraps `RestTemplate` and is **specifically designed for integration tests**. It sends real HTTP requests to the embedded test server started by `@SpringBootTest(webEnvironment = RANDOM_PORT)` and handles error responses gracefully (without throwing exceptions on 4xx/5xx).

| Aspect | Description |
|---|---|
| **What it does** | Sends real HTTP requests to the test server |
| **Auto-configured by** | `@SpringBootTest(webEnvironment = RANDOM_PORT or DEFINED_PORT)` |
| **Error handling** | Returns `ResponseEntity` with error status — does NOT throw exceptions |
| **vs RestTemplate** | RestTemplate throws on 4xx/5xx; TestRestTemplate returns the response |
| **Inject with** | `@Autowired` |

### Core Methods:

| Method | HTTP | Purpose |
|---|---|---|
| `getForEntity(url, ResponseType.class)` | GET | Fetch response with status |
| `postForEntity(url, body, ResponseType.class)` | POST | Create resource, get response |
| `put(url, body)` | PUT | Update resource (returns void) |
| `delete(url)` | DELETE | Delete resource (returns void) |
| `exchange(url, method, entity, ResponseType.class)` | Any | Full control over request |

### Complete Integration Test Example:
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class OrderApiIntegrationTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    private String url(String path) {
        return "http://localhost:" + port + "/api/orders" + path;
    }

    // --- POST — Create Order ---
    @Test
    void createOrder_ShouldReturn201_WithOrderId() {
        Order newOrder = new Order(0, "PENDING", 500.0);

        ResponseEntity<Order> response = restTemplate.postForEntity(
            url(""), newOrder, Order.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertTrue(response.getBody().getId() > 0);
        assertEquals("PENDING", response.getBody().getStatus());
    }

    // --- GET — Fetch Order ---
    @Test
    void getOrder_ShouldReturn200_WhenOrderExists() {
        ResponseEntity<Order> response = restTemplate.getForEntity(
            url("/1"), Order.class);

        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertNotNull(response.getBody());
    }

    // --- GET — 404 Handling (no exception thrown!) ---
    @Test
    void getOrder_ShouldReturn404_WhenOrderNotFound() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            url("/9999"), String.class);

        assertEquals(HttpStatus.NOT_FOUND, response.getStatusCode());
        // TestRestTemplate does NOT throw an exception — safely returns the 404
    }

    // --- PUT — Update Order ---
    @Test
    void updateOrder_ShouldReturn200_WithUpdatedStatus() {
        Order updatedOrder = new Order(1, "SHIPPED", 500.0);

        restTemplate.put(url("/1"), updatedOrder);

        // Verify update by fetching again
        ResponseEntity<Order> response = restTemplate.getForEntity(
            url("/1"), Order.class);
        assertEquals("SHIPPED", response.getBody().getStatus());
    }

    // --- DELETE ---
    @Test
    void deleteOrder_ShouldReturn204() {
        restTemplate.delete(url("/1"));

        // Verify it's gone
        ResponseEntity<String> response = restTemplate.getForEntity(
            url("/1"), String.class);
        assertEquals(HttpStatus.NOT_FOUND, response.getStatusCode());
    }

    // --- exchange() for full control ---
    @Test
    void updateWithExchange_ShouldReturn200() {
        Order updated = new Order(1, "DELIVERED", 500.0);
        HttpEntity<Order> requestEntity = new HttpEntity<>(updated);

        ResponseEntity<Order> response = restTemplate.exchange(
            url("/1"),
            HttpMethod.PUT,
            requestEntity,
            Order.class
        );

        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals("DELIVERED", response.getBody().getStatus());
    }
}
```

### TestRestTemplate vs MockMvc:

| Aspect | `TestRestTemplate` | `MockMvc` |
|---|---|---|
| **HTTP layer** | Real HTTP requests over TCP | Simulated in-memory HTTP |
| **Server required** | ✅ Real embedded server | ❌ No server needed |
| **Context** | `@SpringBootTest(RANDOM_PORT)` | `@WebMvcTest` or `@SpringBootTest` + `@AutoConfigureMockMvc` |
| **Speed** | 🐢 Slower | ⚡ Faster |
| **Use case** | Full end-to-end integration tests | Controller unit/slice tests |
| **Filters / interceptors** | ✅ Included (real HTTP pipeline) | ✅ Included (simulated pipeline) |

**Real World Analogy:** `TestRestTemplate` is like a **real customer placing an order on your website** — the request travels through the full stack (load balancer → controller → service → DB) just like production. `MockMvc` is like a **tester pressing buttons on a simulator** — fast and controlled, but not the real deal.

---

*Happy Learning Spring Testing! 🧪*
