# 16 — Testing Strategy Deep Dive

> **Goal:** Master the testing practices essential for Java backend development — from unit tests to integration tests, TDD, mocking, and contract testing.

---

## Table of Contents

1. [Testing Pyramid](#1-testing-pyramid)
2. [Unit Testing (JUnit 5 + Mockito)](#2-unit-testing-junit-5--mockito)
3. [Integration Testing](#3-integration-testing)
4. [TestContainers](#4-testcontainers)
5. [MockMvc (API Testing)](#5-mockmvc-api-testing)
6. [TDD / BDD](#6-tdd--bdd)
7. [Mocking Strategies](#7-mocking-strategies)
8. [Test Coverage](#8-test-coverage)
9. [Contract Testing](#9-contract-testing)
10. [Interview Notes](#10-interview-notes)
11. [Summary](#11-summary)
12. [References](#12-references)

---

## 1. Testing Pyramid

```
         /  E2E Tests  \         ← Few: Full system, slow, expensive
        / Integration   \        ← Some: Service + real DB, medium speed
       /   Unit Tests    \       ← Many: Fast, isolated, mock deps
```

| Layer | Speed | Scope | Dependencies | Count |
|-------|-------|-------|-------------|-------|
| Unit | ~ms | Single class | Mocked | 70-80% |
| Integration | ~sec | Multiple layers | Real DB (TestContainers) | 15-20% |
| E2E | ~min | Full system | All real | 5-10% |

---

## 2. Unit Testing (JUnit 5 + Mockito)

**Definition:**
Unit tests verify individual units of code (methods, classes) in isolation. All external dependencies are mocked.

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock private UserRepository userRepository;
    @Mock private PasswordEncoder passwordEncoder;
    @InjectMocks private UserService userService;

    @Test
    @DisplayName("Should create user with hashed password")
    void createUser_shouldHashPasswordAndSave() {
        // Given
        CreateUserRequest request = new CreateUserRequest("Prince", "prince@test.com", "pass123");
        when(userRepository.existsByEmail("prince@test.com")).thenReturn(false);
        when(passwordEncoder.encode("pass123")).thenReturn("$2a$12$hashedvalue");
        when(userRepository.save(any(User.class))).thenAnswer(inv -> {
            User u = inv.getArgument(0);
            u.setId(1L);
            return u;
        });

        // When
        UserDto result = userService.create(request);

        // Then
        assertThat(result.name()).isEqualTo("Prince");
        verify(passwordEncoder).encode("pass123");
        verify(userRepository).save(argThat(user ->
            user.getPasswordHash().equals("$2a$12$hashedvalue")
        ));
    }

    @Test
    @DisplayName("Should throw exception when email already exists")
    void createUser_duplicateEmail_shouldThrow() {
        CreateUserRequest request = new CreateUserRequest("Prince", "exists@test.com", "pass");
        when(userRepository.existsByEmail("exists@test.com")).thenReturn(true);

        assertThatThrownBy(() -> userService.create(request))
            .isInstanceOf(BusinessException.class)
            .hasMessageContaining("already registered");

        verify(userRepository, never()).save(any());
    }
}
```

**Best practices:**
- One assertion focus per test (test one behavior)
- Use Given-When-Then structure
- Test happy path AND edge cases
- Don't test framework behavior (Spring, JPA)
- Use meaningful test names (`@DisplayName`)
- Use AssertJ for fluent assertions

---

## 3. Integration Testing

**Definition:**
Integration tests verify that multiple components work together correctly — controllers + services + real databases + real Spring context.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@Testcontainers
@ActiveProfiles("test")
class UserControllerIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired private MockMvc mockMvc;
    @Autowired private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void createUser_shouldReturn201AndPersist() throws Exception {
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name": "Prince", "email": "prince@test.com", "password": "secret123"}
                """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("Prince"))
            .andExpect(jsonPath("$.email").value("prince@test.com"))
            .andExpect(jsonPath("$.id").isNumber());

        assertThat(userRepository.count()).isEqualTo(1);
        assertThat(userRepository.findByEmail("prince@test.com")).isPresent();
    }

    @Test
    void createUser_invalidEmail_shouldReturn400() throws Exception {
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name": "Prince", "email": "not-an-email", "password": "secret123"}
                """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.message").value("Validation failed"));
    }
}
```

---

## 4. TestContainers

**Definition:**
Testcontainers is a Java library that provides lightweight, disposable Docker containers for integration tests. Instead of H2 (which behaves differently from PostgreSQL), you test against the real database.

**Why it exists:**
H2 doesn't support PostgreSQL-specific features (JSONB, window functions, ON CONFLICT). TestContainers guarantees your tests use the exact same database engine as production.

**Setup:**
```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

**Available containers:** PostgreSQL, MySQL, MongoDB, Redis, Kafka, RabbitMQ, Elasticsearch, and more.

---

## 5. MockMvc (API Testing)

**Definition:**
MockMvc simulates HTTP requests against your Spring MVC controllers without starting a real HTTP server. It tests the full Spring MVC stack (filters, interceptors, message converters).

```java
// GET request
mockMvc.perform(get("/api/v1/users/42")
        .accept(MediaType.APPLICATION_JSON))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.id").value(42))
    .andExpect(jsonPath("$.name").value("Prince"));

// POST with auth
mockMvc.perform(post("/api/v1/orders")
        .contentType(MediaType.APPLICATION_JSON)
        .header("Authorization", "Bearer " + jwtToken)
        .content(objectMapper.writeValueAsString(orderRequest)))
    .andExpect(status().isCreated())
    .andExpect(header().exists("Location"));

// DELETE
mockMvc.perform(delete("/api/v1/users/42")
        .header("Authorization", "Bearer " + adminToken))
    .andExpect(status().isNoContent());
```

---

## 6. TDD / BDD

**TDD (Test-Driven Development):**
1. **Red:** Write a failing test first
2. **Green:** Write minimum code to make it pass
3. **Refactor:** Improve code while keeping tests green

**BDD (Behavior-Driven Development):**
```java
@Test
@DisplayName("Given a valid order, When payment is processed, Then order status should be CONFIRMED")
void processPayment_validOrder_shouldConfirm() {
    // Given
    Order order = Order.create(userId, items);

    // When
    order.confirmPayment(paymentId);

    // Then
    assertThat(order.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
    assertThat(order.getPaymentId()).isEqualTo(paymentId);
}
```

---

## 7. Mocking Strategies

| Strategy | Use Case | Tool |
|----------|----------|------|
| Mock | Replace dependency with fake | `@Mock` (Mockito) |
| Spy | Partial mock — real methods + override some | `@Spy` (Mockito) |
| Stub | Return predefined values | `when().thenReturn()` |
| Verify | Assert a method was called | `verify(mock).method()` |
| ArgumentCaptor | Capture arguments passed to mocks | `ArgumentCaptor.forClass()` |

```java
// Capture and inspect arguments
@Test
void placeOrder_shouldPublishEvent() {
    ArgumentCaptor<OrderEvent> captor = ArgumentCaptor.forClass(OrderEvent.class);

    orderService.placeOrder(request);

    verify(eventPublisher).publishEvent(captor.capture());
    OrderEvent event = captor.getValue();
    assertThat(event.orderId()).isNotNull();
    assertThat(event.amount()).isEqualTo(new BigDecimal("99.99"));
}
```

---

## 8. Test Coverage

**What to measure:**
- Line coverage (how many lines are executed)
- Branch coverage (how many if/else branches are tested)
- **Target:** 70-80% overall. 90%+ for critical business logic.

**What NOT to do:**
- Don't chase 100% — diminishing returns
- Don't test getters/setters, POJOs, DTOs
- Don't write tests just to increase the number

**Tools:** JaCoCo (most common for Java), IntelliJ built-in coverage.

---

## 9. Contract Testing

**Definition:**
Contract testing verifies that the API contract (request/response format) between a consumer (frontend, other service) and producer (your API) is maintained. Changes that break the contract are caught before deployment.

**Tool:** Spring Cloud Contract, Pact

```java
// Producer-side contract test
@SpringBootTest(webEnvironment = MOCK)
@AutoConfigureMockMvc
@AutoConfigureStubRunner
class UserApiContractTest {
    @Test
    void getUserById_shouldMatchContract() {
        mockMvc.perform(get("/api/v1/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").isNumber())
            .andExpect(jsonPath("$.name").isString())
            .andExpect(jsonPath("$.email").isString());
    }
}
```

---

## 10. Interview Notes

1. **What is the test pyramid?** — Many unit tests, some integration, few E2E. Unit tests are fastest, most numerous.
2. **How do you test a Spring Boot controller?** — MockMvc for web layer, `@SpringBootTest` for full integration, TestContainers for real DB.
3. **Why use TestContainers over H2?** — H2 behaves differently from PostgreSQL. TestContainers test against the real engine.
4. **What do you mock and what do you not mock?** — Mock external dependencies (DB, APIs). Don't mock the class under test or simple utilities.
5. **What's your target test coverage?** — 70-80% overall, 90%+ for business-critical code. Focus on meaningful tests, not coverage numbers.

---

## 11. Summary

| Topic | Key Takeaway |
|-------|-------------|
| Test Pyramid | Unit (70%) > Integration (20%) > E2E (10%) |
| Unit Tests | JUnit 5 + Mockito, test in isolation |
| Integration | SpringBootTest + TestContainers for real DB |
| MockMvc | Test controllers without real HTTP server |
| TDD | Red → Green → Refactor |
| Coverage | 70-80% target, 90%+ for critical logic |

---

## 12. References

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://site.mockito.org/)
- [TestContainers](https://testcontainers.com/)
- [Baeldung — Testing](https://www.baeldung.com/spring-boot-testing)
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [Spring Testing Documentation](https://docs.spring.io/spring-framework/reference/testing.html)

---

> **Previous:** [← 15 — Backend Learning Roadmap](./15_Backend_Learning_Roadmap.md)
> **Next:** [17 — DevOps, CI/CD & Containerization →](./17_DevOps_CI_CD_Containerization.md)
