# System Design for Java Applications (Application Lead) â€” Complete Interview Guide

## Goal

Help you answer senior/lead system design questions with clear trade-offs, production concerns, and Java-oriented implementation choices.

## Table of Contents

1. [How to Drive the Interview](#how-to-drive-the-interview)
2. [Requirements â†’ Architecture](#requirements--architecture)
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

## Requirements â†’ Architecture

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

- p99 latency tight â†’ caching, denormalization, precomputation
- Very high availability â†’ multi-AZ, failover, stateless services
- Strong consistency â†’ single-writer, transactional boundaries, careful replication

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

### What â€œstateless serviceâ€ means in practice

- No session state in memory that you canâ€™t lose.
- Any state needed for requests lives in DB/cache/token.

---

## API Design (REST) & Idempotency

### Idempotency (must-know)

- â€œSame request repeated should not create duplicate side effects.â€

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

- â€œHow do we query?â€ comes before â€œhow do we store?â€

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

   - Read: check cache â†’ if miss, fetch DB â†’ fill cache.
   - Write: write DB â†’ invalidate/update cache.

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
- Exactly-once: usually â€œeffectively onceâ€ (idempotent consumers) in real systems.

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
- 10 actions/user/day â†’ 10M actions/day
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

### Q1: How do you choose between sync REST and async messaging?

Sync for immediate user response; async for long-running work and decoupling. Often: sync "accept" + async processing.

```java
// Sync: User needs immediate confirmation
@PostMapping("/orders")
public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest req) {
    Order order = orderService.create(req);
    return ResponseEntity.ok(new OrderResponse(order.getId(), "ACCEPTED"));
}

// Async: Background processing
@PostMapping("/orders")
public ResponseEntity<OrderResponse> createOrderAsync(@RequestBody OrderRequest req) {
    String orderId = UUID.randomUUID().toString();
    kafkaTemplate.send("orders", orderId, req); // Async processing
    return ResponseEntity.accepted().body(new OrderResponse(orderId, "PROCESSING"));
}
```

---

### Q2: How do you avoid double-processing in event-driven systems?

Assume at-least-once delivery; build idempotent consumers (dedupe keys, upserts).

```java
@KafkaListener(topics = "payments")
public void processPayment(PaymentEvent event) {
    String idempotencyKey = event.getTransactionId();
    
    // Check if already processed
    if (processedEvents.contains(idempotencyKey)) {
        log.info("Duplicate event ignored: {}", idempotencyKey);
        return;
    }
    
    // Use database upsert for idempotency
    paymentRepository.upsert(Payment.builder()
        .transactionId(idempotencyKey)
        .amount(event.getAmount())
        .status(PaymentStatus.COMPLETED)
        .build());
    
    processedEvents.add(idempotencyKey);
}
```

---

### Q3: What's your approach to reliability under dependency failures?

Timeouts, bounded retries, circuit breaker, bulkheads, and graceful degradation with clear SLOs.

```java
@Service
public class ResilientPaymentService {
    
    @CircuitBreaker(name = "payment", fallbackMethod = "fallbackPayment")
    @Retry(name = "payment", fallbackMethod = "fallbackPayment")
    @TimeLimiter(name = "payment")
    public CompletableFuture<PaymentResult> processPayment(PaymentRequest req) {
        return CompletableFuture.supplyAsync(() -> 
            paymentGateway.charge(req));
    }
    
    public CompletableFuture<PaymentResult> fallbackPayment(PaymentRequest req, Throwable t) {
        log.warn("Payment fallback triggered", t);
        return CompletableFuture.completedFuture(
            PaymentResult.deferred(req.getOrderId()));
    }
}
```

---

### Q4: How do you prevent cache stampedes?

Single-flight per key, TTL jitter, and prewarming.

```java
@Service
public class CacheService {
    private final LoadingCache<String, Product> cache;
    
    public CacheService(ProductRepository repo) {
        this.cache = Caffeine.newBuilder()
            .maximumSize(10_000)
            .expireAfterWrite(Duration.ofMinutes(5))
            // Jitter: randomize TTL to prevent synchronized expiration
            .expireAfterWrite(Duration.ofMinutes(4 + ThreadLocalRandom.current().nextInt(2)))
            // Single-flight: only one thread loads, others wait
            .build(key -> repo.findById(key).orElseThrow());
    }
    
    // Prewarm cache on startup
    @EventListener(ApplicationReadyEvent.class)
    public void prewarmCache() {
        productRepository.findTopProducts(1000)
            .forEach(p -> cache.put(p.getId(), p));
    }
}
```

---

### Q5: How do you handle schema evolution safely?

Backward-compatible changes, expand/contract migrations, dual writes when required, and versioned APIs/events.

```java
// Expand/Contract Pattern for DB migrations

// Step 1: EXPAND - Add new column (nullable)
// ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

// Step 2: Code writes to BOTH columns
@Entity
public class User {
    private String firstName;
    private String lastName;
    private String fullName; // New field
    
    @PrePersist @PreUpdate
    void syncFullName() {
        this.fullName = firstName + " " + lastName;
    }
}

// Step 3: Backfill existing data
// UPDATE users SET full_name = CONCAT(first_name, ' ', last_name);

// Step 4: CONTRACT - Remove old columns after all services updated
```

---

### Q6: How do you debug p99 latency spikes?

Correlate traces, check saturation (thread pools/GC), analyze slow DB queries, and identify contention hotspots.

```java
// Key metrics to monitor
@Configuration
public class LatencyMonitoring {
    
    @Bean
    MeterBinder threadPoolMetrics(ThreadPoolTaskExecutor executor) {
        return registry -> {
            Gauge.builder("executor.queue.size", executor, e -> e.getThreadPoolExecutor().getQueue().size())
                .register(registry);
            Gauge.builder("executor.active", executor, ThreadPoolTaskExecutor::getActiveCount)
                .register(registry);
        };
    }
}

// Trace slow operations
@Around("@annotation(Timed)")
public Object traceSlowMethods(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.nanoTime();
    try {
        return pjp.proceed();
    } finally {
        long duration = System.nanoTime() - start;
        if (duration > SLOW_THRESHOLD_NS) {
            log.warn("Slow operation: {} took {}ms", 
                pjp.getSignature(), duration / 1_000_000);
        }
    }
}
```

---

### Q7: What's your deployment risk strategy?

Gradual rollout, canary, feature flags, fast rollback, and safe defaults.

```java
// Feature flag for gradual rollout
@Service
public class FeatureFlagService {
    
    public boolean isEnabled(String feature, String userId) {
        FeatureConfig config = featureRepository.findByName(feature);
        
        // Percentage rollout
        if (config.getRolloutPercentage() < 100) {
            int hash = Math.abs(userId.hashCode() % 100);
            return hash < config.getRolloutPercentage();
        }
        
        // Allowlist check
        return config.getAllowedUsers().contains(userId);
    }
}

// Usage in controller
@GetMapping("/checkout")
public CheckoutResponse checkout(@RequestParam String userId) {
    if (featureFlagService.isEnabled("new-checkout-flow", userId)) {
        return newCheckoutService.process();
    }
    return legacyCheckoutService.process();
}
```

---

### Q8: How do you design a scalable API rate limiter?

Use token bucket or sliding window algorithms with distributed storage (Redis).

```java
@Component
public class DistributedRateLimiter {
    private final StringRedisTemplate redis;
    
    public boolean tryAcquire(String clientId, int limit, Duration window) {
        String key = "ratelimit:" + clientId;
        long now = System.currentTimeMillis();
        long windowStart = now - window.toMillis();
        
        // Sliding window using Redis sorted set
        return redis.execute((RedisCallback<Boolean>) conn -> {
            byte[] keyBytes = key.getBytes();
            
            // Remove old entries
            conn.zRemRangeByScore(keyBytes, 0, windowStart);
            
            // Count current window
            Long count = conn.zCard(keyBytes);
            
            if (count != null && count < limit) {
                // Add new request
                conn.zAdd(keyBytes, now, String.valueOf(now).getBytes());
                conn.expire(keyBytes, window.getSeconds());
                return true;
            }
            return false;
        });
    }
}
```

---

### Q9: How do you ensure data consistency in distributed systems?

Choose consistency model based on requirements: strong (2PC/Saga), eventual (events + reconciliation), or CQRS for read/write separation.

```java
// Saga Pattern for distributed transactions
@Service
public class OrderSaga {
    
    public void createOrder(OrderRequest request) {
        String sagaId = UUID.randomUUID().toString();
        
        try {
            // Step 1: Reserve inventory
            inventoryService.reserve(sagaId, request.getItems());
            
            // Step 2: Process payment
            paymentService.charge(sagaId, request.getPayment());
            
            // Step 3: Create order
            orderService.create(sagaId, request);
            
        } catch (Exception e) {
            // Compensating transactions
            compensate(sagaId);
            throw e;
        }
    }
    
    private void compensate(String sagaId) {
        orderService.cancel(sagaId);
        paymentService.refund(sagaId);
        inventoryService.release(sagaId);
    }
}
```

---

### Q10: How do you handle service versioning in microservices?

Support multiple versions simultaneously, use semantic versioning, and deprecate gradually.

```java
// API versioning strategies
@RestController
public class UserController {
    
    // URL versioning
    @GetMapping("/v1/users/{id}")
    public UserV1Response getUserV1(@PathVariable String id) {
        return userService.getUserV1(id);
    }
    
    @GetMapping("/v2/users/{id}")
    public UserV2Response getUserV2(@PathVariable String id) {
        return userService.getUserV2(id);
    }
    
    // Header versioning
    @GetMapping(value = "/users/{id}", headers = "X-API-Version=2")
    public UserV2Response getUserByHeader(@PathVariable String id) {
        return userService.getUserV2(id);
    }
}

// Deprecation with sunset header
@GetMapping("/v1/users/{id}")
public ResponseEntity<UserV1Response> getDeprecatedUser(@PathVariable String id) {
    return ResponseEntity.ok()
        .header("Sunset", "Sat, 31 Dec 2024 23:59:59 GMT")
        .header("Deprecation", "true")
        .body(userService.getUserV1(id));
}
```

---

### Q11: How do you design for high-throughput event processing?

Partition for parallelism, batch for efficiency, and use backpressure mechanisms.

```java
@Configuration
public class KafkaConsumerConfig {
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Event> kafkaListenerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, Event> factory = 
            new ConcurrentKafkaListenerContainerFactory<>();
        
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(12); // Parallel consumers
        factory.setBatchListener(true); // Batch processing
        factory.getContainerProperties().setAckMode(AckMode.MANUAL_IMMEDIATE);
        
        return factory;
    }
}

@KafkaListener(topics = "events", containerFactory = "kafkaListenerFactory")
public void processBatch(List<Event> events, Acknowledgment ack) {
    try {
        // Process in parallel with bounded concurrency
        events.parallelStream()
            .forEach(this::processEvent);
        ack.acknowledge();
    } catch (Exception e) {
        // Don't acknowledge - will retry
        log.error("Batch processing failed", e);
    }
}
```

---

### Q12: How do you implement observability in distributed systems?

Three pillars: metrics (Micrometer), logs (structured), and traces (OpenTelemetry).

```java
@Service
public class OrderService {
    private final MeterRegistry meterRegistry;
    private final Tracer tracer;
    
    public Order createOrder(OrderRequest req) {
        Span span = tracer.nextSpan().name("createOrder").start();
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
            // Structured logging with trace context
            log.info("Creating order", 
                kv("orderId", req.getOrderId()),
                kv("customerId", req.getCustomerId()),
                kv("traceId", span.context().traceId()));
            
            Order order = processOrder(req);
            
            meterRegistry.counter("orders.created", 
                "status", "success").increment();
            
            return order;
        } catch (Exception e) {
            meterRegistry.counter("orders.created", 
                "status", "failure").increment();
            span.error(e);
            throw e;
        } finally {
            sample.stop(meterRegistry.timer("order.creation.time"));
            span.end();
        }
    }
}
```

---

### Q13: How do you design a notification system at scale?

Fan-out on write vs fan-out on read, prioritization, and delivery guarantees.

```java
@Service
public class NotificationService {
    
    // Fan-out on write for small audiences
    public void notifyFollowers(String userId, Notification notification) {
        List<String> followers = followerService.getFollowers(userId);
        
        if (followers.size() < 1000) {
            // Direct delivery for small fan-out
            followers.forEach(f -> 
                notificationQueue.send(f, notification));
        } else {
            // Async batch processing for large fan-out
            kafkaTemplate.send("bulk-notifications", 
                new BulkNotification(userId, notification));
        }
    }
    
    // Priority queue for urgent notifications
    @KafkaListener(topics = "notifications")
    public void processNotification(Notification n) {
        if (n.getPriority() == Priority.HIGH) {
            pushService.sendImmediately(n);
        } else {
            batchBuffer.add(n); // Batch for efficiency
        }
    }
}
```

---

### Q14: How do you handle capacity planning and auto-scaling?

Define SLOs, measure utilization, and configure proactive scaling based on leading indicators.

```yaml
# Kubernetes HPA configuration
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
```

---

### Q15: How do you implement distributed locking?

Use Redis or Zookeeper with proper TTL and fencing tokens.

```java
@Component
public class RedisDistributedLock {
    private final StringRedisTemplate redis;
    
    public Optional<Lock> tryAcquire(String resource, Duration ttl) {
        String lockId = UUID.randomUUID().toString();
        String key = "lock:" + resource;
        
        Boolean acquired = redis.opsForValue()
            .setIfAbsent(key, lockId, ttl);
        
        if (Boolean.TRUE.equals(acquired)) {
            return Optional.of(new Lock(key, lockId));
        }
        return Optional.empty();
    }
    
    public void release(Lock lock) {
        // Lua script for atomic check-and-delete
        String script = 
            "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "  return redis.call('del', KEYS[1]) " +
            "else return 0 end";
        
        redis.execute(new DefaultRedisScript<>(script, Long.class),
            List.of(lock.key()), lock.lockId());
    }
    
    public record Lock(String key, String lockId) {}
}
```

---
## What to practice next

- Do 2 designs end-to-end: URL shortener + order processing.
- In every design, explicitly discuss: idempotency, backpressure, observability, and failure modes.
