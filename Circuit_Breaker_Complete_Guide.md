
# Circuit Breaker in Spring Boot — Complete Production Guide From Beginner to Advanced | Interview Ready | Production Oriented
```
tags: springboot, java, microservices, systemdesign, backend 
```


> **Follow me on:** [Linkedin](https://www.linkedin.com/in/iprincekumark/)

---

## SECTION 1: BASIC UNDERSTANDING

### 1.1 What Is a Circuit Breaker?

A **Circuit Breaker** is a design pattern that prevents an application from repeatedly trying to call a service that is likely to fail. It acts as a protective wrapper around external calls (REST APIs, databases, message brokers) that monitors failures and "trips" when failures exceed a threshold.

**Simple Definition:**

It is a safety switch between your service and an external dependency. When the dependency is unhealthy, the switch opens and stops sending requests — protecting your system from wasting resources on calls that will fail anyway.

---

### 1.2 Why Is It Needed in Microservices?

In a monolith, if a function fails, you get an exception and handle it. In microservices, a failure in **Service B** can cascade to **Service A**, **C**, **D**, and bring down the entire system.

**Without Circuit Breaker:**

```
User Request → Service A → Service B (DOWN) → Service A waits...
→ Thread blocked → More requests come → All threads blocked
→ Service A is now DOWN too → Service C calls A → C is DOWN
→ Entire system collapses
```

**With Circuit Breaker:**

```
User Request → Service A → Circuit Breaker → Service B (DOWN)
→ Circuit Breaker detects failure → Opens the circuit
→ Returns fallback immediately → No thread blocking
→ Service A stays healthy → System survives
```

---

### 1.3 Real-World Analogy

Think of the electrical circuit breaker in your house:

- **Normal state:** Electricity flows freely from the grid to your appliances.
- **Short circuit happens:** Wires overheat, risk of fire.
- **Circuit breaker TRIPS:** Cuts off electricity immediately.
- **You fix the problem, then RESET the breaker.**
- **Electricity flows again.**

Software Circuit Breaker works exactly the same way:

- **Normal state:** Requests flow from your service to the dependency.
- **Dependency starts failing:** Timeouts, 500 errors, exceptions.
- **Circuit Breaker TRIPS:** Stops sending requests. Returns fallback.
- **After a wait period, it sends a TEST request.**
- **If the test succeeds, circuit CLOSES.** Normal flow resumes.

---

### 1.4 What Problems It Solves

**Problem 1: CASCADING FAILURES**

One failing service takes down the entire chain of services. Circuit Breaker **isolates the failure**.

**Problem 2: THREAD EXHAUSTION**

Tomcat has 200 threads. If all are waiting on a dead service, no threads are left to serve other requests. Circuit Breaker **fails fast** — threads are freed immediately.

**Problem 3: RESOURCE WASTE**

Sending 1000 requests/second to a dead service wastes CPU, memory, and network bandwidth. Circuit Breaker **stops the waste**.

**Problem 4: SLOW RECOVERY**

When a failing service comes back, hammering it with full traffic can bring it down again. Circuit Breaker sends a **few test requests first** (half-open).

**Problem 5: POOR USER EXPERIENCE**

Users wait 30 seconds for a timeout error. Circuit Breaker returns a **fallback in milliseconds**.

---

## SECTION 2: CORE CONCEPTS

### 2.1 The Three States

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ┌─────────┐    failure rate    ┌─────────┐    wait duration    │
│   │ CLOSED  │ ──── exceeds ────→ │  OPEN   │ ──── expires ────→ │
│   │         │    threshold       │         │                     │
│   └─────────┘                    └─────────┘                     │
│       ↑                                           │              │
│       │                                           ↓              │
│       │  test request succeeds    ┌───────────┐                  │
│       └────────────────────────── │ HALF-OPEN │                  │
│                                   └───────────┘                  │
│       │                                           │              │
│       └────── test request fails ─────────────────┘              │
│                (back to OPEN)                                    │
└──────────────────────────────────────────────────────────────────┘
```

**STATE 1: CLOSED (Normal Operation)**

- All requests pass through to the target service.
- Failures are recorded in a sliding window.
- If failure rate exceeds the configured threshold → transition to OPEN.
- This is the **"healthy"** state.

**STATE 2: OPEN (Circuit Tripped)**

- **NO requests** pass through. All are rejected immediately.
- A fallback method is invoked instead.
- The circuit stays open for a configured "wait duration."
- After the wait duration expires → transition to HALF-OPEN.
- This is the **"protection"** state.

**STATE 3: HALF-OPEN (Testing Recovery)**

- A limited number of test requests are allowed through.
- If these test requests SUCCEED → transition to CLOSED (recovered).
- If these test requests FAIL → transition back to OPEN (still broken).
- This is the **"probing"** state.

---

### 2.2 Failure Rate Threshold

- The **percentage of failed calls** that triggers the circuit to open.
- Default in Resilience4j: **50%**
- **Example:** `failureRateThreshold = 50` → If 50% of calls in the sliding window fail, the circuit opens.

**Production Guidance:**

| Service Type | Recommended Threshold |
|--------------|----------------------|
| Critical services (payments) | 30-40% |
| Non-critical services (analytics) | 60-70% |

---

### 2.3 Slow Call Rate Threshold

- Independently tracks calls that complete but **take too long**.
- A call is considered "slow" if it exceeds `slowCallDurationThreshold`.
- If the percentage of slow calls exceeds `slowCallRateThreshold`, the circuit opens.
- **Example:** `slowCallDurationThreshold = 2s`, `slowCallRateThreshold = 80%` → If 80% of calls take longer than 2 seconds, the circuit opens.

**Why This Matters:**

A service can return **200 OK** but take 10 seconds. It's not "failing" by HTTP status, but it's killing your threads. Slow call threshold catches this.

---

### 2.4 Sliding Window

The sliding window is **how the Circuit Breaker counts failures**.

**TYPE 1: COUNT-BASED SLIDING WINDOW**

- Tracks the **last N calls**.
- **Example:** `slidingWindowSize = 10` → Looks at the last 10 calls. If 5 out of 10 failed (50%), and `failureRateThreshold = 50%`, the circuit opens.
- **Pro:** Simple, predictable.
- **Con:** One burst of failures can trip the circuit.

**TYPE 2: TIME-BASED SLIDING WINDOW**

- Tracks all calls in the **last N seconds**.
- **Example:** `slidingWindowSize = 60` → Looks at all calls in the last 60 seconds.
- **Pro:** Better for services with variable traffic.
- **Con:** Low traffic periods may not collect enough data points.

**MINIMUM NUMBER OF CALLS:**

- `minimumNumberOfCalls = 5` (default)
- The circuit won't evaluate the failure rate until at least 5 calls have been recorded. Prevents tripping on 1 failure out of 1 call.

---

### 2.5 Wait Duration in Open State

- How long the circuit stays OPEN before transitioning to HALF-OPEN.
- **Default:** 60 seconds.
- **Example:** `waitDurationInOpenState = 30s` → After 30 seconds, the circuit moves to HALF-OPEN and allows test requests.

**Production Guidance:**

| Duration | Impact |
|----------|--------|
| Too short (5s) | You'll hammer a recovering service |
| Too long (300s) | Users wait too long for recovery |
| Sweet spot | 15-60 seconds for most services |

---

### 2.6 Permitted Number of Calls in Half-Open

- How many test requests are allowed in the HALF-OPEN state.
- **Default:** 10.
- If these test calls meet the failure threshold → back to OPEN.
- If they pass → CLOSED.
- **Example:** `permittedNumberOfCallsInHalfOpenState = 5` → 5 test requests. If 3+ succeed (assuming 50% threshold), CLOSE.

---

### 2.7 Automatic Transition Behavior

By default, Resilience4j does **NOT** automatically transition from OPEN to HALF-OPEN. It transitions only when a new call is made after the wait duration.

**To enable automatic transition:**

```yaml
automaticTransitionFromOpenToHalfOpenEnabled: true
```

This starts a timer. When the wait duration expires, the state changes to HALF-OPEN even if no new calls are made.

---

## SECTION 3: WHY CIRCUIT BREAKER IS IMPORTANT IN PRODUCTION

### 3.1 Cascading Failure

**Real scenario:**

```
API Gateway → Order Service → Payment Service → Bank API
```

Bank API goes down.

- Payment Service calls Bank API: timeout (30s).
- Order Service calls Payment Service: timeout (30s).
- API Gateway calls Order Service: timeout (30s).
- User waits 90 seconds. Gets 504 Gateway Timeout.

Meanwhile, all threads are blocked across all services. 1000 users make requests. 1000 threads blocked. Every service in the chain is now unresponsive.

**This is a cascading failure. One service took down four.**

**With Circuit Breaker:**

- Bank API goes down.
- Payment Service's circuit breaker detects failures.
- After 5 failures, circuit OPENS.
- Payment Service returns fallback: "Payment processing delayed."
- Order Service receives fallback instantly (5ms, not 30s).
- API Gateway responds to user instantly.
- Threads are free. System stays healthy.

---

### 3.2 Thread Exhaustion

**Spring Boot (Tomcat) default:** 200 threads.

**Without Circuit Breaker:**

- Each request to a dead service blocks a thread for 30s (timeout).
- After ~7 seconds at 30 RPS, all 200 threads are blocked.
- New requests are queued → queue fills → requests rejected.
- Your service is effectively **DOWN**.

**With Circuit Breaker:**

- Circuit opens after first few failures.
- Subsequent requests return fallback in < 1ms.
- Threads are immediately freed.
- Your 200 threads continue serving other endpoints.

---

### 3.3 Resource Starvation

Beyond threads, blocked calls consume:

- Database connections (if the call involves DB)
- Network sockets (TCP connections to the dead service)
- Memory (request/response buffers held in memory)
- CPU (retry loops, exception handling)

**Circuit Breaker eliminates all of this waste when the circuit is open.**

---

### 3.4 How It Improves Resilience

**Resilience** = the ability to handle failures gracefully.

Circuit Breaker provides:

| Benefit | Description |
|---------|-------------|
| **FAIL-FAST** | Don't wait for timeout. Return immediately. |
| **FALLBACK** | Serve a degraded but useful response. |
| **RECOVERY** | Automatically probe and recover when service heals. |
| **ISOLATION** | Failures in one dependency don't affect others. |
| **BACK-PRESSURE** | Don't overwhelm a recovering service. |

---

## SECTION 4: IMPLEMENTATION IN SPRING BOOT

### 4.1 Dependencies

**Maven (pom.xml):**

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**Gradle (build.gradle):**

```gradle
implementation 'io.github.resilience4j:resilience4j-spring-boot3:2.2.0'
implementation 'org.springframework.boot:spring-boot-starter-aop'
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

**NOTE:** `spring-boot-starter-aop` is **REQUIRED**. Resilience4j uses AOP proxies for the `@CircuitBreaker` annotation. Without it, the annotation does nothing silently.

---

### 4.2 Application.yml Configuration

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        registerHealthIndicator: true
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        slowCallRateThreshold: 80
        slowCallDurationThreshold: 2s
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 5
        automaticTransitionFromOpenToHalfOpenEnabled: true
        recordExceptions:
          - java.io.IOException
          - java.net.SocketTimeoutException
          - org.springframework.web.client.HttpServerErrorException
        ignoreExceptions:
          - com.example.BusinessException
```

**EXPLANATION OF EACH PROPERTY:**

| Property | Value | Meaning |
|----------|-------|---------|
| `slidingWindowType` | COUNT_BASED | Track last N calls |
| `slidingWindowSize` | 10 | Track last 10 calls |
| `minimumNumberOfCalls` | 5 | Need 5 calls before evaluating |
| `failureRateThreshold` | 50 | Open if 50% fail |
| `slowCallRateThreshold` | 80 | Open if 80% are slow |
| `slowCallDurationThreshold` | 2s | "Slow" = > 2 seconds |
| `waitDurationInOpenState` | 30s | Stay open for 30 seconds |
| `permittedNumberOfCallsInHalfOpenState` | 5 | Allow 5 test calls |
| `automaticTransitionFromOpenToHalfOpenEnabled` | true | Auto-transition via timer |
| `recordExceptions` | [list] | These count as failures |
| `ignoreExceptions` | [list] | These do NOT count as failures |
| `registerHealthIndicator` | true | Expose in /actuator/health |

---

### 4.3 Service Implementation with @CircuitBreaker

```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Service
@RequiredArgsConstructor
@Slf4j
public class PaymentService {

    private final RestTemplate restTemplate;

    private static final String PAYMENT_SERVICE_URL =
        "http://payment-service/api/v1/payments";

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    public PaymentResponse processPayment(PaymentRequest request) {
        log.info("Calling Payment Service for order: {}", request.getOrderId());

        return restTemplate.postForObject(
            PAYMENT_SERVICE_URL,
            request,
            PaymentResponse.class
        );
    }

    // ── FALLBACK METHOD ──
    // Must have the SAME return type as the original method.
    // Must accept the same parameters PLUS Throwable as the last parameter.

    private PaymentResponse paymentFallback(PaymentRequest request, Throwable ex) {
        log.error("Circuit Breaker activated for order: {}. Error: {}",
                  request.getOrderId(), ex.getMessage());

        return PaymentResponse.builder()
            .orderId(request.getOrderId())
            .status("PAYMENT_DEFERRED")
            .message("Payment service is temporarily unavailable. " +
                     "Your order has been saved and payment will be " +
                     "processed shortly.")
            .build();
    }
}
```

---

### 4.4 Controller

```java
import org.springframework.web.bind.annotation.*;
import lombok.RequiredArgsConstructor;

@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
public class OrderController {

    private final PaymentService paymentService;

    @PostMapping("/{orderId}/pay")
    public ResponseEntity<PaymentResponse> pay(
            @PathVariable String orderId,
            @RequestBody PaymentRequest request) {

        request.setOrderId(orderId);
        PaymentResponse response = paymentService.processPayment(request);

        if ("PAYMENT_DEFERRED".equals(response.getStatus())) {
            return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                .body(response);
        }

        return ResponseEntity.ok(response);
    }
}
```

---

### 4.5 DTOs

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class PaymentRequest {
    private String orderId;
    private BigDecimal amount;
    private String currency;
    private String paymentMethod;
}

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class PaymentResponse {
    private String orderId;
    private String transactionId;
    private String status;
    private String message;
    private LocalDateTime timestamp;
}
```

---

### 4.6 RestTemplate Configuration with Timeouts

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        HttpComponentsClientHttpRequestFactory factory =
            new HttpComponentsClientHttpRequestFactory();
        factory.setConnectTimeout(3000);    // 3 seconds to connect
        factory.setReadTimeout(5000);       // 5 seconds to read response
        return new RestTemplate(factory);
    }
}
```

**IMPORTANT:** Always set timeouts on RestTemplate/WebClient. Without timeouts, threads block indefinitely. The Circuit Breaker can only count a failure AFTER the call completes (or times out).

---

## SECTION 5: ADVANCED CONFIGURATION

### 5.1 Programmatic Configuration

```java
import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import io.github.resilience4j.circuitbreaker.CircuitBreakerRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;

@Configuration
public class CircuitBreakerConfiguration {

    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry() {

        // ── Default Config (applied to all circuit breakers) ──
        CircuitBreakerConfig defaultConfig = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .slowCallRateThreshold(80)
            .slowCallDurationThreshold(Duration.ofSeconds(2))
            .waitDurationInOpenState(Duration.ofSeconds(30))
            .slidingWindowType(CircuitBreakerConfig.SlidingWindowType.COUNT_BASED)
            .slidingWindowSize(10)
            .minimumNumberOfCalls(5)
            .permittedNumberOfCallsInHalfOpenState(5)
            .automaticTransitionFromOpenToHalfOpenEnabled(true)
            .recordExceptions(
                IOException.class,
                SocketTimeoutException.class,
                HttpServerErrorException.class
            )
            .ignoreExceptions(
                BusinessException.class
            )
            .build();

        // ── Custom Config for Payment Service ──
        CircuitBreakerConfig paymentConfig = CircuitBreakerConfig.custom()
            .failureRateThreshold(30)           // More sensitive
            .waitDurationInOpenState(Duration.ofSeconds(60))  // Longer wait
            .slidingWindowSize(20)              // Larger window
            .minimumNumberOfCalls(10)
            .build();

        return CircuitBreakerRegistry.of(defaultConfig,
            Map.of("paymentService", paymentConfig));
    }
}
```

---

### 5.2 Using CircuitBreakerRegistry Programmatically

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class InventoryService {

    private final CircuitBreakerRegistry registry;
    private final RestTemplate restTemplate;

    public InventoryResponse checkInventory(String productId) {

        CircuitBreaker circuitBreaker = registry.circuitBreaker("inventoryService");

        // Decorate the supplier with circuit breaker
        Supplier<InventoryResponse> decoratedSupplier =
            CircuitBreaker.decorateSupplier(circuitBreaker, () -> {
                return restTemplate.getForObject(
                    "http://inventory-service/api/v1/inventory/" + productId,
                    InventoryResponse.class
                );
            });

        // Execute with fallback
        return Try.ofSupplier(decoratedSupplier)
            .recover(throwable -> {
                log.error("Inventory check failed: {}", throwable.getMessage());
                return InventoryResponse.builder()
                    .productId(productId)
                    .available(true)    // Optimistic fallback
                    .message("Inventory check unavailable, allowing order")
                    .build();
            })
            .get();
    }
}
```

---

### 5.3 Event Listeners (Monitor State Transitions)

```java
@Component
@Slf4j
public class CircuitBreakerEventListener {

    @Autowired
    private CircuitBreakerRegistry registry;

    @PostConstruct
    public void registerEventListeners() {

        registry.getAllCircuitBreakers().forEach(cb -> {

            cb.getEventPublisher()
                .onStateTransition(event -> {
                    log.warn("Circuit Breaker '{}': {} → {}",
                        event.getCircuitBreakerName(),
                        event.getStateTransition().getFromState(),
                        event.getStateTransition().getToState());

                    // Send alert to Slack/PagerDuty
                    if (event.getStateTransition().getToState()
                            == CircuitBreaker.State.OPEN) {
                        alertService.sendCriticalAlert(
                            "Circuit OPEN: " + event.getCircuitBreakerName()
                        );
                    }
                })
                .onError(event -> {
                    log.error("Circuit Breaker '{}': Error recorded - {}",
                        event.getCircuitBreakerName(),
                        event.getThrowable().getMessage());
                })
                .onSuccess(event -> {
                    log.debug("Circuit Breaker '{}': Success. Duration: {}ms",
                        event.getCircuitBreakerName(),
                        event.getElapsedDuration().toMillis());
                });
        });
    }
}
```

---

### 5.4 Combining with Retry, Rate Limiter, Bulkhead

**EXECUTION ORDER (outermost to innermost):**

```
Retry → CircuitBreaker → RateLimiter → Bulkhead → TimeLimiter → Function
```

**This means:**

- Retry wraps Circuit Breaker (retries AFTER circuit breaker decision)
- Bulkhead limits concurrent calls INSIDE the circuit breaker

**application.yml:**

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
        slidingWindowSize: 10

  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 1s
        retryExceptions:
          - java.io.IOException

  bulkhead:
    instances:
      paymentService:
        maxConcurrentCalls: 20
        maxWaitDuration: 500ms

  ratelimiter:
    instances:
      paymentService:
        limitForPeriod: 100
        limitRefreshPeriod: 1s
        timeoutDuration: 0

  timelimiter:
    instances:
      paymentService:
        timeoutDuration: 3s
```

**Usage with Multiple Annotations:**

```java
@Service
public class PaymentService {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
    @Retry(name = "paymentService")
    @Bulkhead(name = "paymentService")
    @RateLimiter(name = "paymentService")
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest req) {
        return CompletableFuture.supplyAsync(() ->
            restTemplate.postForObject(PAYMENT_URL, req, PaymentResponse.class)
        );
    }

    private CompletableFuture<PaymentResponse> fallback(
            PaymentRequest req, Throwable ex) {
        return CompletableFuture.completedFuture(
            PaymentResponse.builder()
                .status("DEFERRED")
                .message(ex.getMessage())
                .build()
        );
    }
}
```

---

### 5.5 Timeout vs Circuit Breaker — The Difference

| Aspect | Timeout | Circuit Breaker |
|--------|---------|-----------------|
| **Scope** | Applied to a SINGLE call | Applied across MANY calls over time |
| **Purpose** | "If this call doesn't respond in 5 seconds, abort it." | "If 50% of recent calls failed, stop sending more." |
| **Memory** | Does not remember history | Remembers failure history via sliding window |
| **Prevention** | Does not prevent future calls | Prevents future calls when the circuit is open |

**THEY COMPLEMENT EACH OTHER:**

- **Timeout** ensures a single slow call doesn't block forever.
- **Circuit Breaker** ensures repeated failures don't consume resources.

**Best Practice: ALWAYS use both together.**

**Call Flow:**

```
Request → Circuit Breaker (is it open?) → Timeout (max 5s)
        → External Service → Response
```

If timeout fires → counted as failure by Circuit Breaker. If enough timeouts → Circuit Breaker opens.

---

## SECTION 6: MONITORING & OBSERVABILITY

### 6.1 Actuator Integration

**application.yml:**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, circuitbreakers, circuitbreakerevents, prometheus
  endpoint:
    health:
      show-details: always
  health:
    circuitbreakers:
      enabled: true
```

**ENDPOINTS:**

**GET /actuator/health**

Shows circuit breaker health status:

```json
{
  "circuitBreakers": {
    "paymentService": {
      "status": "UP",     // or "CIRCUIT_OPEN", "CIRCUIT_HALF_OPEN"
      "details": {
        "state": "CLOSED",
        "failureRate": "0.0%",
        "slowCallRate": "0.0%",
        "bufferedCalls": 7,
        "failedCalls": 0,
        "slowCalls": 0,
        "notPermittedCalls": 0
      }
    }
  }
}
```

**Other Endpoints:**

- **GET /actuator/circuitbreakers** - Lists all registered circuit breakers and their current states
- **GET /actuator/circuitbreakerevents** - Shows recent events: state transitions, errors, successes
- **GET /actuator/circuitbreakerevents/{name}** - Events for a specific circuit breaker

---

### 6.2 Prometheus Integration

**Add dependency:**

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

**Resilience4j automatically exports these metrics to Prometheus:**

| Metric Name | Description |
|-------------|-------------|
| `resilience4j_circuitbreaker_state` | Current state (0=CLOSED, 1=OPEN, 2=HALF_OPEN) |
| `resilience4j_circuitbreaker_calls_seconds_count` | Total calls count |
| `resilience4j_circuitbreaker_calls_seconds_sum` | Total call duration |
| `resilience4j_circuitbreaker_calls_seconds_bucket` | Call duration histogram |
| `resilience4j_circuitbreaker_failure_rate` | Current failure rate % |
| `resilience4j_circuitbreaker_slow_call_rate` | Current slow call rate % |
| `resilience4j_circuitbreaker_buffered_calls` | Buffered calls in window |
| `resilience4j_circuitbreaker_not_permitted_calls_total` | Rejected calls (circuit open) |

Available at: **GET /actuator/prometheus**

---

### 6.3 Grafana Dashboard

**Key Panels to Create:**

**Panel 1: Circuit Breaker State (Time Series)**
- Query: `resilience4j_circuitbreaker_state{name="paymentService"}`
- Visualization: State timeline showing CLOSED/OPEN/HALF_OPEN over time

**Panel 2: Failure Rate (Gauge)**
- Query: `resilience4j_circuitbreaker_failure_rate{name="paymentService"}`
- Thresholds: Green < 25%, Yellow < 50%, Red >= 50%

**Panel 3: Calls per Second (Graph)**
- Successful: `rate(resilience4j_circuitbreaker_calls_seconds_count{kind="successful"}[5m])`
- Failed: `rate(resilience4j_circuitbreaker_calls_seconds_count{kind="failed"}[5m])`

**Panel 4: Not Permitted Calls (Counter)**
- Query: `rate(resilience4j_circuitbreaker_not_permitted_calls_total[5m])`
- Alert: If > 0 for more than 1 minute → circuit is open and rejecting traffic

**Panel 5: Call Duration P99 (Histogram)**
- Query: `histogram_quantile(0.99, resilience4j_circuitbreaker_calls_seconds_bucket)`

---

### 6.4 What Metrics to Monitor in Production

**CRITICAL ALERTS (page immediately):**

- Circuit state changed to OPEN
- `not_permitted_calls > 0` sustained for > 2 minutes
- Failure rate > threshold for > 1 minute

**WARNING ALERTS (create ticket):**

- Failure rate increasing trend (approaching threshold)
- Slow call rate > 50%
- Frequent OPEN → HALF_OPEN → OPEN cycling (service flapping)

**DASHBOARD MONITORING (daily review):**

- Failure rate trend over 24 hours
- P99 call duration trend
- Number of state transitions per day

---

## SECTION 7: COMMON MISTAKES

### 7.1 Wrong Fallback Implementation

**❌ BAD — fallback calls another service that could also be down:**

```java
private PaymentResponse paymentFallback(PaymentRequest req, Throwable ex) {
    return backupPaymentService.process(req);  // What if this fails too?
}
```

**✅ GOOD — fallback returns cached/default data:**

```java
private PaymentResponse paymentFallback(PaymentRequest req, Throwable ex) {
    return PaymentResponse.builder()
        .status("DEFERRED")
        .message("Payment will be retried")
        .build();
}
```

**RULES FOR FALLBACKS:**

1. Should NOT call another external service (unless it has its own CB)
2. Should return cached data, default values, or queued responses
3. Should be fast — the whole point is to fail fast
4. Should log the error for debugging
5. Must have the SAME return type as the original method
6. Must accept the SAME parameters + Throwable as last parameter

---

### 7.2 Not Configuring Sliding Window Properly

**❌ BAD:** `slidingWindowSize = 2`, `failureRateThreshold = 50`

One failure out of 2 calls = 50% → circuit opens. Perfectly healthy service has one bad request → circuit trips.

**✅ GOOD:** `slidingWindowSize >= 10`, `minimumNumberOfCalls >= 5`

This absorbs noise and prevents premature tripping.

---

### 7.3 Using Default Config Blindly

**Default Resilience4j config:**

- `failureRateThreshold = 50`
- `slidingWindowSize = 100`
- `waitDurationInOpenState = 60s`

100 calls in the sliding window is **too large** for low-traffic services. If your service handles 10 requests/minute, it takes 10 minutes to fill the window. The circuit won't react to failures quickly.

**ALWAYS tune configuration** per service based on:

- Request volume (RPS)
- Acceptable failure rate
- Expected recovery time of the dependency
- Business criticality

---

### 7.4 When NOT to Use Circuit Breaker

**DO NOT use Circuit Breaker for:**

**1. DATABASE CALLS within the same service**

- Use connection pool timeouts and query timeouts instead.
- A circuit breaker on DB calls would prevent ALL database access even for different queries/tables.

**2. INTERNAL METHOD CALLS**

- Circuit Breaker works via AOP proxy. Internal method calls (`this.method()`) bypass the proxy.

**3. IDEMPOTENT OPERATIONS where retry is better**

- If the operation is idempotent and the failure is transient, a simple retry with backoff may be more appropriate.

**4. BATCH PROCESSING**

- Circuit Breaker on batch items would trip on legitimate data errors. Use error handling per item instead.

**5. FIRE-AND-FORGET async operations**

- If you don't need the response, just send and log failures. Don't add circuit breaker complexity.

---

## SECTION 8: INTERVIEW QUESTIONS & ANSWERS

### Q1: What is a Circuit Breaker pattern and why is it used?

**Answer:** Circuit Breaker is a resilience pattern that prevents an application from making calls to a service that is likely to fail. It monitors failure rates and, when failures exceed a threshold, it "opens" the circuit — immediately rejecting subsequent calls and returning a fallback response. This prevents cascading failures, thread exhaustion, and resource waste in a microservices architecture. It is similar to an electrical circuit breaker that trips to prevent damage.

---

### Q2: Explain the three states of a Circuit Breaker.

**Answer:**

**CLOSED** — Normal state. All requests pass through. Failures are recorded in a sliding window. If the failure rate exceeds the threshold, the circuit transitions to OPEN.

**OPEN** — Protective state. All requests are immediately rejected with a fallback response. No calls reach the target service. After a configured wait duration, the circuit transitions to HALF-OPEN.

**HALF-OPEN** — Recovery testing state. A limited number of test requests are allowed through. If they succeed, the circuit transitions back to CLOSED. If they fail, it transitions back to OPEN.

---

### Q3: How does Resilience4j differ from Netflix Hystrix?

**Answer:** Netflix Hystrix is in maintenance mode (no active development). Resilience4j is the recommended replacement.

**Key differences:**

| Aspect | Hystrix | Resilience4j |
|--------|---------|--------------|
| **Architecture** | Thread Pool model (separate threads) | Semaphore model (lighter, no extra threads) |
| **Modularity** | Monolithic | Modular — pick only what you need |
| **Java Support** | Older Java versions | Designed for Java 8+ with functional programming |
| **Spring Boot** | Basic integration | Better integration with auto-configuration |
| **Reactive** | Limited | Supports both synchronous and reactive (Project Reactor) |

---

### Q4: What is the difference between count-based and time-based sliding window?

**Answer:**

**COUNT-BASED:** Tracks the last N calls. Example: last 10 calls. If 5 out of 10 failed (50%), the circuit opens. Predictable but can be skewed by bursts.

**TIME-BASED:** Tracks all calls in the last N seconds. Example: all calls in the last 60 seconds. Better for services with variable traffic patterns. But in low-traffic periods, few data points may lead to inaccurate rates.

---

### Q5: Can you explain how fallback methods work? What are the rules?

**Answer:** A fallback method is called when the circuit breaker rejects a call (circuit open) or when the call throws a recorded exception.

**Rules:**

1. Must have the SAME return type as the original method.
2. Must accept the SAME parameters as the original method.
3. Must have one additional parameter of type Throwable (at the end).
4. Can be private, but must be in the same class.
5. Should not call another external service without its own circuit breaker.
6. Should return cached data, defaults, or queue the operation for later.

---

### Q6: What happens if the Circuit Breaker annotation is on a private method?

**Answer:** It will NOT work. Resilience4j uses Spring AOP proxies. AOP proxies only intercept calls made through the proxy (i.e., from outside the class). Private methods are called internally and bypass the proxy.

Similarly, calling a `@CircuitBreaker` method from within the same class (self-invocation) bypasses the proxy. The circuit breaker is never triggered.

**Fix:** Extract the circuit breaker method into a separate `@Service` bean.

---

### Q7: How do you combine Circuit Breaker with Retry?

**Answer:** The execution order matters. In Resilience4j with Spring Boot:

```
Retry → Circuit Breaker → Function
```

This means: Retry wraps the Circuit Breaker. If the circuit is open, the retry gets a `CallNotPermittedException` and does NOT retry (because the circuit is open, not a transient failure).

If the circuit is closed and the call fails, the failure is recorded by the circuit breaker. The retry then retries the call (which goes through the circuit breaker again).

**Configuration example:**

- Retry: `maxAttempts=3`, `waitDuration=1s`
- CircuitBreaker: `failureRateThreshold=50`

**Result:** Each call is retried up to 3 times. Each attempt's outcome (success/failure) is recorded by the circuit breaker.

---

### Q8: What is the difference between Circuit Breaker and Bulkhead?

**Answer:**

**CIRCUIT BREAKER:** Stops ALL calls when the failure rate is high. Protects your system from a failing dependency.

**BULKHEAD:** Limits the CONCURRENT NUMBER of calls. Even if calls are succeeding, only N threads can call the dependency simultaneously. Protects your system from a slow dependency consuming all threads.

**They solve different problems and should be used together:**

- Bulkhead prevents thread exhaustion from slow responses.
- Circuit Breaker prevents repeated calls to a failing service.

---

### Q9: How would you handle Circuit Breaker in an event-driven system?

**Answer:** In event-driven systems (Kafka, RabbitMQ), the Circuit Breaker pattern applies differently:

**1. PRODUCER SIDE:** If the message broker is down, the circuit breaker can prevent producing more messages. Fallback: write to a local outbox table and retry later.

**2. CONSUMER SIDE:** If the consumer calls an external service during message processing, wrap that call in a circuit breaker. When the circuit opens, the consumer can NACK the message (requeue for later) or send it to a Dead Letter Queue.

**3. BACKPRESSURE:** If the circuit is open, stop consuming messages from the queue. Resume when the circuit closes.

---

### Q10: Describe a production scenario where Circuit Breaker saved the system.

**Answer:** E-commerce platform during Black Friday. The Recommendation Service became overloaded and started responding in 15 seconds instead of 100ms.

**Without Circuit Breaker:** All 200 Tomcat threads on the Product Service were blocked waiting for recommendations. Product pages stopped loading. Users couldn't browse products. Revenue dropped to zero.

**With Circuit Breaker:** After 5 slow calls (`slowCallRateThreshold=80%`), the circuit opened. Product Service returned products WITHOUT recommendations (fallback: empty recommendation list). Pages loaded in 200ms instead of 15 seconds. Users could browse and purchase. Revenue continued. When the Recommendation Service recovered, the half-open state detected success, and recommendations returned.

**Impact:** Saved approximately **$2M in revenue** during the 45-minute outage.

---

## SECTION 9: PERFORMANCE & TRADE-OFFS

### 9.1 Overhead Cost

Resilience4j's overhead is minimal:

| Component | Complexity | Cost |
|-----------|-----------|------|
| Sliding window | O(1) per call | Ring buffer internally |
| State check | O(1) | AtomicReference comparison |
| Metric recording | O(1) | Atomic counters |
| **Total overhead per call** | | **~1-5 microseconds** |

Compared to a network call (1-500ms), the overhead is **negligible**.

---

### 9.2 When It Can Reduce Performance

**1. OVER-SENSITIVE CONFIGURATION**

Small sliding window + low threshold = frequent circuit opening. Service flaps between OPEN and CLOSED. Users see intermittent failures even when the dependency is mostly healthy.

**2. FALLBACK IS EXPENSIVE**

If your fallback queries a database or another service, you're replacing one latency problem with another.

**3. METRICS OVERHEAD IN HIGH-THROUGHPUT**

At 100,000+ RPS, the metric recording and event publishing can add measurable overhead. Tune event consumer buffer sizes.

---

### 9.3 Best Practices for High-Traffic Systems

1. Use **COUNT-BASED** sliding window for consistent traffic. Use **TIME-BASED** sliding window for bursty traffic.

2. Set `minimumNumberOfCalls` high enough to absorb noise. Low traffic: 5-10. High traffic: 20-50.

3. Tune `waitDurationInOpenState` based on dependency recovery time. If the dependency takes 2 minutes to recover, set wait to 30s. Circuit tries recovery 4 times during the outage.

4. Different circuit breakers for different endpoints. `/payments` and `/refunds` on the same service can behave differently. Use separate circuit breaker instances per endpoint.

5. Keep fallbacks in-memory. No network calls. Return cached data, default values, or accept-and-queue patterns.

6. Monitor circuit breaker state transitions. Alert on OPEN state. Track flapping (rapid state changes).

7. Test circuit breakers regularly. Chaos engineering: deliberately fail dependencies in staging. Verify that the circuit opens, fallbacks work, and recovery happens.

8. Combine with timeout + retry + bulkhead for defense in depth.

---

## SECTION 10: REAL-WORLD ARCHITECTURE EXAMPLE

### 10.1 E-Commerce System Architecture

```
┌──────────┐     ┌────────────────┐     ┌──────────────────┐
│  React   │────→│  API Gateway   │────→│  Order Service   │
│ Frontend │     │ (Spring Cloud) │     │  (Spring Boot)   │
└──────────┘     └────────────────┘     └────────┬─────────┘
                                                  │
                        ┌────────────────────────┬┴───────────────┐
                        │                        │                │
                 ┌──────┴───────┐   ┌────────────┴──┐   ┌───────┴────────┐
                 │   Payment    │   │  Inventory    │   │ Notification   │
                 │   Service    │   │  Service      │   │ Service        │
                 │ [CB: 30%/30s]│   │ [CB: 50%/15s] │   │ [CB: 70%/10s]  │
                 └──────┬───────┘   └───────────────┘   └────────────────┘
                        │
                 ┌──────┴───────┐
                 │  Bank API    │
                 │  (External)  │
                 └──────────────┘
```

Each downstream call has its own Circuit Breaker with configuration tuned to the criticality and behavior of that service.

---

### 10.2 Payment Service Failure Scenario

**TIMELINE:**

| Time | Event |
|------|-------|
| **T=0s** | Bank API becomes unresponsive (network issue). |
| **T=0-5s** | Payment Service makes 10 calls to Bank API. Each times out after 5 seconds. Circuit Breaker records 10 failures. Failure rate = 100% > threshold (30%). Circuit Breaker → OPEN. |
| **T=5-35s** | Circuit is OPEN. All payment requests immediately return fallback: "Payment processing delayed. Order saved." Order Service receives the fallback. Order is saved with status "PAYMENT_PENDING". User sees: "Order placed! Payment being processed." Meanwhile: Order Service is fully functional (threads free). Inventory Service is fully functional. Users can continue browsing and placing orders. Background job will retry pending payments later. |
| **T=35s** | Wait duration expires. Circuit → HALF-OPEN. 5 test requests are sent to Bank API. Bank API is still down. 5/5 fail → Circuit → OPEN again. |
| **T=65s** | Wait duration expires again. Circuit → HALF-OPEN. 5 test requests sent. Bank API has recovered. 5/5 succeed → Circuit → CLOSED. |
| **T=65s+** | Normal operation resumes. Background job retries all PAYMENT_PENDING orders. Payments are processed. Users are notified: "Payment confirmed." |

**RESULT:**

- Zero downtime for the e-commerce platform.
- Users could continue shopping throughout the outage.
- All orders were eventually processed.
- No cascading failure to other services.

---

### 10.3 Complete Production Order Service Code

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;
    private final OrderRepository orderRepository;

    @Transactional
    public OrderResponse createOrder(OrderRequest request) {

        // 1. Check inventory (circuit breaker protected)
        InventoryResponse inventory =
            inventoryService.checkAvailability(request.getProductId());

        if (!inventory.isAvailable()) {
            throw new OutOfStockException(request.getProductId());
        }

        // 2. Save order with PENDING status
        Order order = Order.builder()
            .productId(request.getProductId())
            .userId(request.getUserId())
            .amount(request.getAmount())
            .status(OrderStatus.CREATED)
            .build();
        order = orderRepository.save(order);

        // 3. Process payment (circuit breaker protected)
        PaymentResponse payment =
            paymentService.processPayment(
                PaymentRequest.builder()
                    .orderId(order.getId())
                    .amount(order.getAmount())
                    .build()
            );

        // 4. Update order based on payment result
        if ("SUCCESS".equals(payment.getStatus())) {
            order.setStatus(OrderStatus.CONFIRMED);
            order.setTransactionId(payment.getTransactionId());
        } else if ("PAYMENT_DEFERRED".equals(payment.getStatus())) {
            order.setStatus(OrderStatus.PAYMENT_PENDING);
            // Background job will retry later
        }
        orderRepository.save(order);

        // 5. Send notification (circuit breaker protected, fire-and-forget)
        notificationService.sendOrderConfirmation(order);

        return OrderResponse.from(order);
    }
}
```

**PaymentService.java (with full resilience stack):**

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class PaymentService {

    private final RestTemplate restTemplate;

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    @Bulkhead(name = "paymentService")
    public PaymentResponse processPayment(PaymentRequest request) {
        log.info("Processing payment for order: {}", request.getOrderId());
        return restTemplate.postForObject(
            "http://payment-service/api/v1/payments",
            request,
            PaymentResponse.class
        );
    }

    private PaymentResponse paymentFallback(PaymentRequest request, Throwable ex) {
        log.warn("Payment fallback for order: {}. Reason: {}",
                 request.getOrderId(), ex.getMessage());
        return PaymentResponse.builder()
            .orderId(request.getOrderId())
            .status("PAYMENT_DEFERRED")
            .message("Payment queued for retry")
            .timestamp(LocalDateTime.now())
            .build();
    }
}
```

**PendingPaymentRetryJob.java:**

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class PendingPaymentRetryJob {

    private final OrderRepository orderRepository;
    private final PaymentService paymentService;

    @Scheduled(fixedDelay = 60000)  // Every 60 seconds
    public void retryPendingPayments() {
        List<Order> pendingOrders = orderRepository
            .findByStatus(OrderStatus.PAYMENT_PENDING);

        for (Order order : pendingOrders) {
            try {
                PaymentResponse response = paymentService.processPayment(
                    PaymentRequest.builder()
                        .orderId(order.getId())
                        .amount(order.getAmount())
                        .build()
                );

                if ("SUCCESS".equals(response.getStatus())) {
                    order.setStatus(OrderStatus.CONFIRMED);
                    order.setTransactionId(response.getTransactionId());
                    orderRepository.save(order);
                    log.info("Pending payment resolved for order: {}",
                             order.getId());
                }
                // If still deferred, will be retried next cycle
            } catch (Exception e) {
                log.error("Retry failed for order: {}", order.getId(), e);
            }
        }
    }
}
```

---

## SECTION 11: QUICK REFERENCE CHEAT SHEET

### Circuit Breaker Cheat Sheet

**STATES:** CLOSED (healthy) → OPEN (failing) → HALF-OPEN (testing)

**WHEN IT OPENS:**

- `failureRateThreshold` exceeded (default: 50%)
- OR `slowCallRateThreshold` exceeded (default: 100%)

**WHEN IT CLOSES:**

- Test calls in HALF-OPEN succeed below failure threshold

**KEY CONFIG:**

| Configuration | Description |
|---------------|-------------|
| `slidingWindowSize` | How many calls to track |
| `failureRateThreshold` | % failures to trip |
| `waitDurationInOpenState` | How long to stay open |
| `minimumNumberOfCalls` | Min calls before evaluating |
| `permittedCallsInHalfOpen` | Test calls allowed |

**ANNOTATIONS:**

```java
@CircuitBreaker(name="x", fallbackMethod="y")
@Retry + @Bulkhead + @RateLimiter + @TimeLimiter
```

**EXECUTION ORDER:**

```
Retry → CB → RateLimiter → Bulkhead → TimeLimiter → Function
```

**MUST-HAVE DEPENDENCIES:**

- `resilience4j-spring-boot3`
- `spring-boot-starter-aop`

**GOLDEN RULES:**

| Rule | Description |
|------|-------------|
| ✓ | Always configure timeouts on HTTP clients |
| ✓ | Fallbacks should be fast (no network calls) |
| ✓ | Tune config per service, don't use defaults |
| ✓ | Monitor state transitions and alert on OPEN |
| ✓ | Combine with retry + bulkhead for defense in depth |
| ✗ | Don't use CB for database calls within same service |
| ✗ | Don't ignore the proxy bypass problem (self-invocation) |

---

## Conclusion

The **Circuit Breaker pattern** is essential for building resilient microservices. It prevents cascading failures, protects your threads, and provides graceful degradation when dependencies fail.

**Key Takeaways:**

- **Three states:** CLOSED → OPEN → HALF-OPEN
- **Always configure timeouts** on HTTP clients
- **Tune configuration** per service based on traffic and criticality
- **Keep fallbacks fast** — no network calls
- **Monitor state transitions** and alert on OPEN state
- **Combine with Retry, Bulkhead, Rate Limiter** for defense in depth
- **Test regularly** with chaos engineering

By implementing Circuit Breaker correctly, you can save millions in revenue during outages, maintain system health, and provide better user experiences even when dependencies fail.

---

*💡 Found this guide helpful? Follow me for more backend engineering content and share it with your engineering team!*

---

**Total Coverage:** 11 Sections | Beginner to Advanced | Production Ready
