# 11 — Observability & Performance Tuning

> **Goal:** Master logging, metrics, tracing, monitoring, and JVM performance tuning — the pillars of running reliable Java backends in production.

---

## Table of Contents

1. [Observability Overview](#1-observability-overview)
2. [Structured Logging](#2-structured-logging)
3. [Metrics](#3-metrics)
4. [Distributed Tracing](#4-distributed-tracing)
5. [Monitoring & Alerting](#5-monitoring--alerting)
6. [JVM Tuning](#6-jvm-tuning)
7. [Thread Pool Tuning](#7-thread-pool-tuning)
8. [Latency Analysis](#8-latency-analysis)
9. [Throughput Optimization](#9-throughput-optimization)
10. [Interview Notes](#10-interview-notes)
11. [Summary](#11-summary)
12. [References](#12-references)

---

## 1. Observability Overview

**Definition:**
Observability is the ability to understand a system's internal state by examining its outputs. The three pillars are **Logs** (what happened), **Metrics** (how much/how fast), and **Traces** (the request journey across services).

**Why it exists:**
In production, you can't attach a debugger or add print statements. Observability tools let you diagnose issues, track performance, and understand system behavior without modifying code.

**Real-life analogy:**
Running a factory: Logs = CCTV recordings (detailed events), Metrics = dashboard gauges (temperature, speed, output counts), Traces = tracking a specific product through the assembly line.

```
┌─────────────────────────────────────────────┐
│              OBSERVABILITY                    │
│                                               │
│  LOGS          METRICS        TRACES          │
│  What happened? How much?     Where did the   │
│  Error details  QPS, latency  request go?     │
│                                               │
│  SLF4J/Logback  Micrometer    OpenTelemetry   │
│  ELK Stack      Prometheus    Zipkin/Jaeger   │
│                 Grafana                        │
└─────────────────────────────────────────────┘
```

---

## 2. Structured Logging

**Definition:**
Structured logging outputs log events as key-value pairs (JSON) instead of unstructured text, making them machine-parsable and searchable in log aggregation systems.

**Why it exists:**
Unstructured logs like `"User 42 placed order 123"` are human-readable but hard to search, filter, and analyze at scale. Structured logs enable queries like `user_id=42 AND action=order_placed`.

**Backend example:**
```java
// Using SLF4J with MDC (Mapped Diagnostic Context)
@Component
public class RequestLoggingFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
            FilterChain chain) throws Exception {
        String requestId = req.getHeader("X-Request-Id");
        if (requestId == null) requestId = UUID.randomUUID().toString();

        MDC.put("requestId", requestId);
        MDC.put("clientIp", req.getRemoteAddr());
        MDC.put("method", req.getMethod());
        MDC.put("path", req.getRequestURI());

        long start = System.currentTimeMillis();
        try {
            chain.doFilter(req, res);
        } finally {
            long duration = System.currentTimeMillis() - start;
            MDC.put("status", String.valueOf(res.getStatus()));
            MDC.put("durationMs", String.valueOf(duration));
            log.info("Request completed");
            MDC.clear();
        }
    }
}
```

```xml
<!-- logback-spring.xml — JSON output -->
<appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>
```

**Output:**
```json
{
    "timestamp": "2026-02-23T16:48:46.123",
    "level": "INFO",
    "logger": "RequestLoggingFilter",
    "message": "Request completed",
    "requestId": "abc-123",
    "method": "GET",
    "path": "/api/v1/users/42",
    "status": "200",
    "durationMs": "45",
    "clientIp": "192.168.1.1"
}
```

**Logging best practices:**

| Level | Use For | Example |
|-------|---------|---------|
| ERROR | Unrecoverable failures, requires attention | DB connection lost, payment failed |
| WARN | Unexpected but handled situations | Retry succeeded, fallback used |
| INFO | Business events, request lifecycle | Order placed, user registered |
| DEBUG | Diagnostic detail for development | SQL queries, method parameters |
| TRACE | Very fine-grained debugging | Loop iterations, cache lookups |

---

## 3. Metrics

**Definition:**
Metrics are numerical measurements collected over time that represent system behavior — request counts, response times, error rates, JVM memory usage, database connection pool stats.

**Spring Boot + Micrometer + Prometheus:**
```java
// Custom business metrics
@Service
@RequiredArgsConstructor
public class OrderService {
    private final MeterRegistry meterRegistry;
    private final Counter orderCounter;
    private final Timer orderProcessingTimer;

    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.orderCounter = Counter.builder("orders.created.total")
            .description("Total orders created")
            .tag("service", "order-service")
            .register(meterRegistry);
        this.orderProcessingTimer = Timer.builder("orders.processing.duration")
            .description("Order processing time")
            .register(meterRegistry);
    }

    public OrderDto createOrder(CreateOrderRequest req) {
        return orderProcessingTimer.record(() -> {
            OrderDto order = processOrder(req);
            orderCounter.increment();
            meterRegistry.gauge("orders.pending",
                orderRepository.countByStatus(OrderStatus.PENDING));
            return order;
        });
    }
}
```

```yaml
# application.yml — expose Prometheus endpoint
management:
  endpoints:
    web:
      exposure:
        include: health, info, prometheus, metrics
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: my-api
```

**Key metrics to monitor:**

| Metric | What It Tells You |
|--------|-------------------|
| `http_server_requests_seconds` | Request latency (p50, p95, p99) |
| `jvm_memory_used_bytes` | Heap/non-heap memory usage |
| `jvm_gc_pause_seconds` | GC pause duration and frequency |
| `hikaricp_connections_active` | Active DB connections |
| `hikaricp_connections_pending` | Threads waiting for connection |
| `system_cpu_usage` | CPU utilization |
| Custom counters/timers | Business metrics |

---

## 4. Distributed Tracing

**Definition:**
Distributed tracing tracks a single request as it flows across multiple services, using a trace ID and span IDs. Each service adds a span with timing data.

**Spring Boot 3 + Micrometer Tracing:**
```yaml
management:
  tracing:
    sampling:
      probability: 1.0 # 100% sampling (reduce in production)
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
```

```java
// Trace propagation happens automatically with Spring Boot 3
// Just add micrometer-tracing-bridge-brave and zipkin-reporter-brave dependencies

// Manual span creation for custom operations
@Service
public class PaymentService {
    private final Tracer tracer;

    public PaymentResult charge(PaymentRequest req) {
        Span span = tracer.nextSpan().name("payment-gateway-call").start();
        try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
            return paymentGateway.charge(req);
        } finally {
            span.end();
        }
    }
}
```

---

## 5. Monitoring & Alerting

**Standard monitoring stack:**

```
App (Spring Boot + Micrometer)
    │
    ▼ (scrapes /actuator/prometheus)
Prometheus (metrics storage & queries)
    │
    ▼
Grafana (dashboards & visualization)
    │
    ▼
AlertManager (notifications: Slack, PagerDuty, email)
```

**Key alerts to configure:**
- Error rate > 5% for 5 minutes
- p99 latency > 2 seconds
- CPU > 80% for 10 minutes
- Heap usage > 85%
- DB connection pool exhausted
- Circuit breaker OPEN

---

## 6. JVM Tuning

**Definition:**
JVM tuning involves configuring heap size, garbage collector, and JVM flags to optimize application performance for your specific workload.

**Essential JVM flags for production Spring Boot:**
```bash
java -jar app.jar \
  -Xms2g -Xmx2g \                    # Heap size (min=max to avoid resizing)
  -XX:+UseG1GC \                      # G1 GC (default JDK 9+)
  -XX:MaxGCPauseMillis=200 \          # Target GC pause time
  -XX:+UseStringDeduplication \       # Share identical strings
  -XX:+HeapDumpOnOutOfMemoryError \   # Dump heap on OOM
  -XX:HeapDumpPath=/tmp/heapdump.hprof \
  -Xlog:gc*:file=gc.log:time,level    # GC logging
```

**Heap sizing guidelines:**
- Start with 2x your app's steady-state live data set
- Set `-Xms` = `-Xmx` (avoids heap resizing overhead)
- Monitor and adjust based on GC metrics
- Too small → frequent GC → high latency
- Too large → long GC pauses → memory waste

---

## 7. Thread Pool Tuning

| Pool | Sizing Strategy |
|------|----------------|
| Tomcat threads | Default 200. For I/O-bound: increase. For CPU-bound: ≈ cores × 2 |
| HikariCP connections | `(cores × 2) + effective_spindle_count` ≈ 10-20 |
| @Async thread pool | Core = 5-10, Max = 20-50, Queue = 100-500 |
| Kafka consumers | Match partition count |

```yaml
server:
  tomcat:
    threads:
      max: 200        # Max worker threads
      min-spare: 10   # Minimum idle threads
    accept-count: 100  # OS-level queue when all threads busy
```

---

## 8. Latency Analysis

**Where latency hides:**
```
Client ──(network)──→ LB ──→ App ──(business logic)──→ DB ──(query)──→ Response
         ~5ms         ~1ms    ~10ms                      ~50ms

Total: ~66ms
Bottleneck: Database query (76% of total time)
```

**Tools for analysis:**
- **Distributed tracing** (Zipkin/Jaeger) — cross-service latency
- **Slow query log** — database bottlenecks
- **JVM profiler** (async-profiler, JFR) — CPU/allocation hotspots
- **Flame graphs** — visualize where CPU time is spent

---

## 9. Throughput Optimization

| Technique | Impact |
|-----------|--------|
| Connection pooling (HikariCP) | Eliminates connection overhead |
| Caching hot data (Redis/Caffeine) | Reduces DB load by >90% |
| Async processing (@Async, Kafka) | Non-blocking for non-critical paths |
| Batch operations | Reduce round trips |
| Virtual threads (JDK 21+) | Handle more concurrent I/O |
| Database query optimization | Reduce query time |
| Response compression (gzip) | Reduce network transfer |
| HTTP/2 | Multiplexed connections |

---

## 10. Interview Notes

1. **What are the three pillars of observability?** — Logs, Metrics, Traces. Logs = what happened. Metrics = how much. Traces = where.
2. **How do you monitor a Spring Boot app in production?** — Actuator + Micrometer → Prometheus → Grafana. Alerts for error rate, latency, CPU, memory.
3. **How do you diagnose a slow API endpoint?** — Distributed tracing → identify slow span → slow query log → explain plan → add index or optimize.
4. **What JVM flags do you set in production?** — `-Xms=Xmx`, G1GC, HeapDumpOnOOM, GC logging.
5. **How do you tune thread pools?** — I/O-bound: more threads. CPU-bound: cores × 2. Monitor utilization, adjust.

---

## 11. Summary

| Topic | Key Takeaway |
|-------|-------------|
| Logging | Structured JSON + MDC for correlation |
| Metrics | Micrometer → Prometheus → Grafana |
| Tracing | Micrometer Tracing → Zipkin/Jaeger |
| JVM Tuning | Set Xms=Xmx, use G1/ZGC, enable heap dumps |
| Thread Pools | Size based on workload type (I/O vs CPU) |
| Latency | Trace → identify bottleneck → optimize |

---

## 12. References

- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Micrometer Documentation](https://micrometer.io/docs)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [OpenTelemetry](https://opentelemetry.io/)
- [Baeldung — JVM Tuning](https://www.baeldung.com/jvm-parameters)
- [async-profiler](https://github.com/async-profiler/async-profiler)

---

> **Previous:** [← 10 — Caching, Messaging & Async](./10_Caching_Messaging_Async.md)
> **Next:** [12 — Real-World Backend Issues & Fixes →](./12_Real_World_Backend_Issues_Fixes.md)
