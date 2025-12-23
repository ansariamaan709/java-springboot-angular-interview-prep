# System Design for Java Applications (Application Lead) — Complete Interview Guide

## Goal

Help you answer senior/lead system design questions with clear trade-offs, production concerns, and Java-oriented implementation choices.

## Table of Contents

1. [How to Drive the Interview](#how-to-drive-the-interview)
2. [Requirements → Architecture](#requirements--architecture)
3. [Core Building Blocks](#core-building-blocks)
4. [API Design (REST) & Idempotency](#api-design-rest--idempotency)
5. [Data Modeling, Indexing, and Query Patterns](#data-modeling-indexing-and-query-patterns)
6. [Scaling: Vertical, Horizontal, Sharding](#scaling-vertical-horizontal-sharding)
7. [Caching Strategies](#caching-strategies)
8. [Messaging & Event-Driven Systems](#messaging--event-driven-systems)
9. [Consistency, Transactions, and Distributed Data](#consistency-transactions-and-distributed-data)
10. [Resilience: Timeouts, Retries, Circuit Breakers](#resilience-timeouts-retries-circuit-breakers)
11. [Concurrency, Threading, and Backpressure](#concurrency-threading-and-backpressure)
12. [Observability: Logs, Metrics, Traces](#observability-logs-metrics-traces)
13. [Security for System Design](#security-for-system-design)
14. [Capacity Estimation (Practical)](#capacity-estimation-practical)
15. [Common System Design Questions (Java flavor)](#common-system-design-questions-java-flavor)
16. [Lead-Level Interview Q&A](#lead-level-interview-qa)

---

## How to Drive the Interview

### A reliable 7-step flow

1. **Clarify requirements** (functional + non-functional)
2. **Define APIs** (request/response, status codes, error shapes)
3. **Define data model** (entities, keys, indexes)
4. **High-level architecture** (services, DB, cache, queues)
5. **Deep dive** the hottest path (read/write flow)
6. **Failure modes** (timeouts, retries, partial failure, consistency)
7. **Operational plan** (observability, rollout, SLOs)

### What interviewers want from an Application Lead

- You make trade-offs explicit.
- You anticipate production incidents.
- You propose an MVP first, then scaling improvements.

---

## Requirements → Architecture

### Functional requirements examples

- Create order
- Get order status
- Cancel order

### Non-functional requirements (NFRs)

- Latency (p99)
- Availability (e.g., 99.9%)
- Consistency level
- Throughput (RPS)
- Durability (data loss tolerance)
- Security/compliance

### Translate NFRs to concrete decisions

- p99 latency tight → caching, denormalization, precomputation
- Very high availability → multi-AZ, failover, stateless services
- Strong consistency → single-writer, transactional boundaries, careful replication

---

## Core Building Blocks

### Baseline architecture (common)

```
Clients
  |
[API Gateway / LB]
  |
[Stateless Service(s)]  ----->  [Cache]
  |
  +--------------------->  [DB]
  |
  +--------------------->  [Message Broker]
                                   |
                              [Workers]
```

### What “stateless service” means in practice

- No session state in memory that you can’t lose.
- Any state needed for requests lives in DB/cache/token.

---

## API Design (REST) & Idempotency

### Idempotency (must-know)

- “Same request repeated should not create duplicate side effects.”

#### Practical approach

- Client sends an `Idempotency-Key` header.
- Server stores key + request hash + result for a time window.

### Safe retries

- Network issues happen; clients retry.
- Your system must not double-charge or double-create.

### Status codes (common)

- `201 Created` for create
- `200 OK` for read
- `202 Accepted` for async processing
- `409 Conflict` for version conflicts / duplicate operations

### Optimistic concurrency control

- Use version fields (e.g., `version` column).
- Fail with conflict when version mismatch.

### Java Code: Idempotency Filter (Spring Boot)

```java
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import java.io.IOException;
import java.time.Duration;
import java.util.concurrent.ConcurrentHashMap;

@Component
public class IdempotencyFilter extends OncePerRequestFilter {

    // Production: use Redis with TTL
    private final ConcurrentHashMap<String, CachedResponse> cache = new ConcurrentHashMap<>();
    private static final Duration TTL = Duration.ofMinutes(10);

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {

        String idempotencyKey = request.getHeader("Idempotency-Key");

        // Only apply to mutating methods
        if (idempotencyKey == null || "GET".equalsIgnoreCase(request.getMethod())) {
            chain.doFilter(request, response);
            return;
        }

        CachedResponse cached = cache.get(idempotencyKey);
        if (cached != null && !cached.isExpired()) {
            // Return cached response (idempotent replay)
            response.setStatus(cached.status);
            response.getWriter().write(cached.body);
            return;
        }

        // Wrap response to capture output
        ContentCachingResponseWrapper wrappedResponse =
                new ContentCachingResponseWrapper(response);

        chain.doFilter(request, wrappedResponse);

        // Cache successful responses
        if (wrappedResponse.getStatus() >= 200 && wrappedResponse.getStatus() < 300) {
            String body = new String(wrappedResponse.getContentAsByteArray());
            cache.put(idempotencyKey, new CachedResponse(
                    wrappedResponse.getStatus(), body, System.currentTimeMillis()));
        }

        wrappedResponse.copyBodyToResponse();
    }

    private record CachedResponse(int status, String body, long createdAt) {
        boolean isExpired() {
            return System.currentTimeMillis() - createdAt > TTL.toMillis();
        }
    }
}
```

### Java Code: Optimistic Locking with JPA

```java
import jakarta.persistence.*;

@Entity
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerId;
    private String status;

    @Version  // Optimistic locking
    private Long version;

    // getters, setters
}

// Service layer
@Service
@Transactional
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    public Order updateStatus(Long orderId, String newStatus) {
        Order order = orderRepository.findById(orderId)
                .orElseThrow(() -> new EntityNotFoundException("Order not found"));

        order.setStatus(newStatus);
        // On concurrent update, JPA throws OptimisticLockException
        return orderRepository.save(order);
    }
}

// Controller: handle conflict
@ExceptionHandler(OptimisticLockException.class)
public ResponseEntity<String> handleConflict(OptimisticLockException ex) {
    return ResponseEntity.status(HttpStatus.CONFLICT)
            .body("Resource was modified by another request. Please retry.");
}
```

---

## Data Modeling, Indexing, and Query Patterns

### Start from access patterns

- “How do we query?” comes before “how do we store?”

### Common mistakes

- Over-normalization when read traffic dominates.
- Missing indexes for critical queries.
- Unbounded result sets (no pagination).

### Indexing rules of thumb

- Index on high-selectivity columns used in WHERE / JOIN.
- Composite index order matters.
- Validate with `EXPLAIN`.

---

## Scaling: Vertical, Horizontal, Sharding

### Vertical scaling

- Faster machine, bigger RAM.
- Simple, but has limits.

### Horizontal scaling

- Add instances; use load balancer.
- Requires statelessness and shared data stores.

### Sharding (partitioning)

- Partition by a key (userId, tenantId, orderId).

**Trade-offs:**

- Cross-shard queries become harder.
- Rebalancing shards is operationally complex.

### Hot partitions

If partition key has skew (celebrity users):

- Use composite keys
- Add random suffix
- Separate hot users/tenants

---

## Caching Strategies

### Cache patterns

1. **Cache-aside** (most common)

   - Read: check cache → if miss, fetch DB → fill cache.
   - Write: write DB → invalidate/update cache.

2. **Write-through**

   - Write cache and DB as one operation (cache mediates).

3. **Write-behind**
   - Write cache first; async flush to DB (riskier).

### TTL and invalidation

- TTL prevents stale data living forever.
- Invalidation is hard; prefer **versioned keys** for some problems.

### Cache stampede protection

- Per-key lock (single flight)
- Request coalescing
- Jitter TTL

### Caching and consistency

Be explicit:

- Strong consistency with cache is expensive.
- Often accept eventual consistency for reads.

---

## Messaging & Event-Driven Systems

### When to add a queue/broker

- You need decoupling.
- You need async work (email, notifications, analytics).
- You need smoothing of spikes.

### Delivery semantics

- At-most-once: can lose messages.
- At-least-once: duplicates possible.
- Exactly-once: usually “effectively once” (idempotent consumers) in real systems.

### Idempotent consumers

- Store processed event IDs (or dedupe key) with TTL.
- Use upserts.

### Outbox pattern (high value)

Problem: DB write succeeded but message publish failed.
Solution: write domain change + outbox event in same DB transaction; a relay publishes.

### Java Code: Outbox Pattern Implementation

```java
// 1. Outbox Entity
@Entity
@Table(name = "outbox_events")
public class OutboxEvent {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String aggregateType;   // e.g., "Order"
    private String aggregateId;     // e.g., "12345"
    private String eventType;       // e.g., "OrderCreated"

    @Column(columnDefinition = "TEXT")
    private String payload;         // JSON payload

    private Instant createdAt;
    private boolean published;

    // constructors, getters, setters
}

// 2. Service: Write domain + outbox in same transaction
@Service
public class OrderService {

    @Autowired private OrderRepository orderRepository;
    @Autowired private OutboxRepository outboxRepository;
    @Autowired private ObjectMapper objectMapper;

    @Transactional  // Single transaction!
    public Order createOrder(CreateOrderRequest request) {
        // Domain write
        Order order = new Order(request.getCustomerId(), request.getItems());
        order = orderRepository.save(order);

        // Outbox write (same transaction)
        OutboxEvent event = new OutboxEvent();
        event.setAggregateType("Order");
        event.setAggregateId(order.getId().toString());
        event.setEventType("OrderCreated");
        event.setPayload(objectMapper.writeValueAsString(new OrderCreatedEvent(order)));
        event.setCreatedAt(Instant.now());
        event.setPublished(false);
        outboxRepository.save(event);

        return order;
    }
}

// 3. Relay: Poll and publish (or use Debezium CDC)
@Scheduled(fixedDelay = 1000)
@Transactional
public void publishOutboxEvents() {
    List<OutboxEvent> pending = outboxRepository.findByPublishedFalse();
    for (OutboxEvent event : pending) {
        try {
            kafkaTemplate.send(event.getAggregateType(), event.getPayload());
            event.setPublished(true);
            outboxRepository.save(event);
        } catch (Exception e) {
            log.warn("Failed to publish event {}: {}", event.getId(), e.getMessage());
            // Will retry on next poll
        }
    }
}
```

---

## Consistency, Transactions, and Distributed Data

### CAP in plain terms

- Under partition, you choose **Consistency** or **Availability**.

### Sagas (distributed transactions)

- Break a large transaction into steps.
- Each step has a compensating action.

### Read-your-writes

- If users must see their update immediately, you may need:
  - sticky sessions to a leader
  - read from primary
  - or client-side hints

---

## Resilience: Timeouts, Retries, Circuit Breakers

### Timeouts

- Always set timeouts on downstream calls.
- Unbounded waits cause thread exhaustion.

### Retries

- Retry only on transient errors.
- Use exponential backoff + jitter.
- Avoid retry storms (add circuit breakers).

### Circuit breaker

- Stop calling a failing dependency for a short window.
- Fail fast and use fallbacks.

### Java Code: Circuit Breaker with Resilience4j

```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.retry.annotation.Retry;
import io.github.resilience4j.timelimiter.annotation.TimeLimiter;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class PaymentGatewayClient {

    private final RestTemplate restTemplate;

    public PaymentGatewayClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    @TimeLimiter(name = "paymentService")
    public PaymentResponse processPayment(PaymentRequest request) {
        return restTemplate.postForObject(
                "https://payment-gateway/api/charge",
                request,
                PaymentResponse.class
        );
    }

    // Fallback when circuit is open or all retries exhausted
    public PaymentResponse paymentFallback(PaymentRequest request, Throwable ex) {
        log.warn("Payment service unavailable, using fallback: {}", ex.getMessage());
        return PaymentResponse.pending(request.getOrderId(),
                "Payment queued for processing");
    }
}
```

**application.yml configuration:**

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 3
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 500ms
        exponentialBackoffMultiplier: 2
        retryExceptions:
          - java.net.ConnectException
          - java.net.SocketTimeoutException
  timelimiter:
    instances:
      paymentService:
        timeoutDuration: 2s
```

### Bulkheads

- Isolate resources per dependency: separate pools/queues.

---

## Concurrency, Threading, and Backpressure

### Common failure mode: thread pool exhaustion

- Too many blocking calls; requests pile up.

### Backpressure options

- Bounded queues
- Caller-runs policy
- Shed load (429/503)
- Token bucket / rate limiting

### Async vs blocking

- Blocking I/O needs careful pool sizing.
- Async pipelines reduce threads, but increase complexity.

---

## Observability: Logs, Metrics, Traces

### Logs

- Structured logs (key-value)
- Correlation IDs (traceId/requestId)
- Avoid logging secrets

### Metrics

- RED method: Rate, Errors, Duration
- Saturation metrics: CPU, heap, thread pools, queue length

### Tracing

- Distributed tracing for microservices.
- Identify latency contributors.

### SLOs / SLIs

- SLI: measurable (p99 latency)
- SLO: target (p99 < 200ms)

---

## Security for System Design

### Authentication & Authorization

- AuthN: who are you?
- AuthZ: what can you do?

### Service-to-service security

- Mutual TLS (mTLS)
- Short-lived tokens

### Data security

- Encrypt in transit + at rest
- Least privilege for DB/service accounts

### Rate limiting

- Protects against abuse and prevents cascading failure.

---

## Capacity Estimation (Practical)

### What to estimate quickly

- Requests per second
- Storage per day/month
- Peak vs average

### Example template

Assume:

- 1M daily active users
- 10 actions/user/day → 10M actions/day
- Peak is 10x average

Average RPS: $10M / 86400 \approx 116$ RPS
Peak RPS: $\approx 1,160$ RPS

Then discuss:

- cache hit rate needed
- DB QPS
- queue sizing

---

## Common System Design Questions (Java flavor)

### 1) Design URL shortener

Key points:

- ID generation (snowflake/segment)
- DB key-value store
- caching hot links
- rate limiting

### 2) Design order placement

Key points:

- idempotent create
- outbox for events
- inventory reservation saga
- consistency and failure handling

### 3) Design notification service

Key points:

- queues + workers
- retries and DLQ
- templates, localization

### 4) Design rate limiter

Key points:

- token bucket/leaky bucket
- distributed counters
- per-user/per-IP

---

## Lead-Level Interview Q&A

1. **How do you choose between sync REST and async messaging?**

   - Sync for immediate user response; async for long-running work and decoupling. Often: sync “accept” + async processing.

2. **How do you avoid double-processing in event-driven systems?**

   - Assume at-least-once delivery; build idempotent consumers (dedupe keys, upserts).

3. **What’s your approach to reliability under dependency failures?**

   - Timeouts, bounded retries, circuit breaker, bulkheads, and graceful degradation with clear SLOs.

4. **How do you prevent cache stampedes?**

   - Single-flight per key, TTL jitter, and prewarming.

5. **How do you handle schema evolution safely?**

   - Backward-compatible changes, expand/contract migrations, dual writes when required, and versioned APIs/events.

6. **How do you debug p99 latency spikes?**

   - Correlate traces, check saturation (thread pools/GC), analyze slow DB queries, and identify contention hotspots.

7. **What’s your deployment risk strategy?**
   - Gradual rollout, canary, feature flags, fast rollback, and safe defaults.

---

## What to practice next

- Do 2 designs end-to-end: URL shortener + order processing.
- In every design, explicitly discuss: idempotency, backpressure, observability, and failure modes.
