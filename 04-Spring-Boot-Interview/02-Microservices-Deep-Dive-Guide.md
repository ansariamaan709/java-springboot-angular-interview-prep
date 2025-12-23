# Spring Boot Microservices Deep Dive — Complete Interview Guide

## Table of Contents

1. [Microservices Fundamentals](#microservices-fundamentals)
2. [Service Discovery with Eureka](#service-discovery-with-eureka)
3. [API Gateway (Spring Cloud Gateway)](#api-gateway-spring-cloud-gateway)
4. [Load Balancing](#load-balancing)
5. [Inter-Service Communication](#inter-service-communication)
6. [Circuit Breaker & Resilience Patterns](#circuit-breaker--resilience-patterns)
7. [Distributed Configuration](#distributed-configuration)
8. [Distributed Tracing](#distributed-tracing)
9. [Event-Driven Microservices](#event-driven-microservices)
10. [Saga Pattern for Distributed Transactions](#saga-pattern-for-distributed-transactions)
11. [Security in Microservices](#security-in-microservices)
12. [Containerization & Kubernetes Basics](#containerization--kubernetes-basics)
13. [Testing Microservices](#testing-microservices)
14. [Common Pitfalls & Best Practices](#common-pitfalls--best-practices)
15. [Interview Questions](#interview-questions)

---

## Microservices Fundamentals

### What are Microservices?

An architectural style where an application is built as a collection of **loosely coupled, independently deployable services**, each owning its data and business capability.

### Monolith vs Microservices

| Aspect         | Monolith                | Microservices                 |
| -------------- | ----------------------- | ----------------------------- |
| Deployment     | Single unit             | Independent services          |
| Scaling        | Scale entire app        | Scale individual services     |
| Technology     | Single stack            | Polyglot                      |
| Database       | Shared                  | Database per service          |
| Team structure | Large team              | Small, autonomous teams       |
| Failure        | Single point of failure | Isolated failures             |
| Complexity     | Simpler initially       | Distributed system complexity |

### When to Use Microservices

- Large, complex applications
- Need for independent scaling
- Multiple teams working in parallel
- Different components need different tech stacks
- High availability requirements

### When NOT to Use

- Small applications or MVPs
- Small team (< 5 developers)
- Simple, well-understood domain
- When you can't afford operational overhead

### Spring Cloud Components Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Spring Cloud Ecosystem                       │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   Eureka    │  │   Gateway   │  │   Config    │  │  Sleuth/    │ │
│  │  (Service   │  │   (API      │  │   Server    │  │  Micrometer │ │
│  │  Discovery) │  │   Gateway)  │  │             │  │  (Tracing)  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ Resilience4j│  │   Feign     │  │  Stream     │  │   Bus       │ │
│  │  (Circuit   │  │  (HTTP      │  │  (Kafka/    │  │  (Config    │ │
│  │   Breaker)  │  │   Client)   │  │   Rabbit)   │  │   Refresh)  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Service Discovery with Eureka

### Why Service Discovery?

In dynamic environments (cloud, containers), service instances come and go. Hardcoded IPs don't work.

### Eureka Server Setup

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

// Main class
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false # Don't register itself
    fetch-registry: false
  server:
    enable-self-preservation: false # For dev; enable in prod
```

### Eureka Client (Service Registration)

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

// Main class (auto-registration with @SpringBootApplication)
@SpringBootApplication
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

```yaml
# application.yml
spring:
  application:
    name: order-service # Service name for discovery

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 30
```

### Discovery Client Usage

```java
@Service
public class ServiceLocator {

    @Autowired
    private DiscoveryClient discoveryClient;

    public List<String> getServiceInstances(String serviceName) {
        return discoveryClient.getInstances(serviceName)
                .stream()
                .map(instance -> instance.getUri().toString())
                .collect(Collectors.toList());
    }
}
```

---

## API Gateway (Spring Cloud Gateway)

### Why API Gateway?

- Single entry point
- Cross-cutting concerns (auth, rate limiting, logging)
- Request routing
- Protocol translation
- Response aggregation

### Gateway Setup

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
# application.yml
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # Auto-route by service name
          lower-case-service-id: true
      routes:
        - id: order-service
          uri: lb://order-service # Load-balanced
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - AddRequestHeader=X-Gateway-Source, api-gateway

        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
            - Method=GET,POST
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: userServiceCB
                fallbackUri: forward:/fallback/users
```

### Custom Gateway Filter

```java
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {

    @Autowired
    private JwtUtil jwtUtil;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        // Skip auth for public endpoints
        if (isPublicEndpoint(request.getPath().value())) {
            return chain.filter(exchange);
        }

        String authHeader = request.getHeaders().getFirst(HttpHeaders.AUTHORIZATION);

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return onError(exchange, "Missing or invalid Authorization header",
                          HttpStatus.UNAUTHORIZED);
        }

        String token = authHeader.substring(7);

        try {
            Claims claims = jwtUtil.validateToken(token);

            // Add user info to headers for downstream services
            ServerHttpRequest modifiedRequest = request.mutate()
                    .header("X-User-Id", claims.getSubject())
                    .header("X-User-Roles", claims.get("roles", String.class))
                    .build();

            return chain.filter(exchange.mutate().request(modifiedRequest).build());

        } catch (JwtException e) {
            return onError(exchange, "Invalid token", HttpStatus.UNAUTHORIZED);
        }
    }

    private boolean isPublicEndpoint(String path) {
        return path.startsWith("/api/auth/") || path.equals("/health");
    }

    private Mono<Void> onError(ServerWebExchange exchange, String message,
                               HttpStatus status) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(status);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);

        String body = "{\"error\": \"" + message + "\"}";
        DataBuffer buffer = response.bufferFactory().wrap(body.getBytes());
        return response.writeWith(Mono.just(buffer));
    }

    @Override
    public int getOrder() {
        return -1;  // Run first
    }
}
```

### Rate Limiting

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: rate-limited-route
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10 # requests per second
                redis-rate-limiter.burstCapacity: 20
                key-resolver: "#{@userKeyResolver}"
```

```java
@Configuration
public class RateLimiterConfig {

    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest().getHeaders().getFirst("X-User-Id")
        );
    }

    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest().getRemoteAddress().getAddress().getHostAddress()
        );
    }
}
```

---

## Load Balancing

### Client-Side Load Balancing with Spring Cloud LoadBalancer

```java
// Replaces Ribbon (deprecated)
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>

@Configuration
public class LoadBalancerConfig {

    @Bean
    @LoadBalanced  // Enables load balancing for RestTemplate
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    @LoadBalanced
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}

@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    public User getUser(Long userId) {
        // "user-service" is resolved via service discovery
        return restTemplate.getForObject(
            "http://user-service/api/users/" + userId,
            User.class
        );
    }
}
```

### Custom Load Balancer Strategy

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    ReactorLoadBalancer<ServiceInstance> customLoadBalancer(
            Environment environment,
            LoadBalancerClientFactory clientFactory) {

        String serviceId = environment.getProperty(
            LoadBalancerClientFactory.PROPERTY_NAME);

        return new RandomLoadBalancer(
            clientFactory.getLazyProvider(serviceId, ServiceInstanceListSupplier.class),
            serviceId
        );
    }
}

// Apply to specific service
@LoadBalancerClient(name = "user-service",
                    configuration = CustomLoadBalancerConfiguration.class)
public class UserServiceConfig {}
```

---

## Inter-Service Communication

### Synchronous: OpenFeign

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

// Enable Feign
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication {}

// Feign Client
@FeignClient(name = "user-service", fallbackFactory = UserClientFallbackFactory.class)
public interface UserClient {

    @GetMapping("/api/users/{id}")
    User getUserById(@PathVariable("id") Long id);

    @PostMapping("/api/users")
    User createUser(@RequestBody CreateUserRequest request);

    @GetMapping("/api/users")
    List<User> getUsersByIds(@RequestParam("ids") List<Long> ids);
}

// Fallback Factory (provides fallback with exception access)
@Component
public class UserClientFallbackFactory implements FallbackFactory<UserClient> {

    @Override
    public UserClient create(Throwable cause) {
        return new UserClient() {
            @Override
            public User getUserById(Long id) {
                log.error("Failed to get user {}: {}", id, cause.getMessage());
                return User.unknown(id);  // Return default user
            }

            @Override
            public User createUser(CreateUserRequest request) {
                throw new ServiceUnavailableException("User service unavailable");
            }

            @Override
            public List<User> getUsersByIds(List<Long> ids) {
                return Collections.emptyList();
            }
        };
    }
}
```

### Feign Configuration

```yaml
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: FULL
      user-service:
        connectTimeout: 3000
        readTimeout: 3000
  circuitbreaker:
    enabled: true
```

### Asynchronous: Message-Based Communication

```java
// See Event-Driven Microservices section
```

---

## Circuit Breaker & Resilience Patterns

### Resilience4j Setup

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

### Circuit Breaker States

```
       ┌──────────────────────────────────────────────────┐
       │                                                  │
       ▼                                                  │
   ┌────────┐    failure threshold     ┌──────┐         │
   │ CLOSED │ ──────exceeded──────────>│ OPEN │         │
   └────────┘                          └──────┘         │
       ▲                                   │            │
       │                          wait duration        │
       │                                   │            │
       │     success          ┌───────────▼──────────┐ │
       └─────────────────────│    HALF_OPEN         │──┘
                    threshold └──────────────────────┘
                    reached          failure
```

### Complete Resilience4j Implementation

```java
@Service
public class PaymentService {

    private final PaymentGatewayClient paymentClient;

    @CircuitBreaker(name = "payment", fallbackMethod = "paymentFallback")
    @Retry(name = "payment")
    @TimeLimiter(name = "payment")
    @Bulkhead(name = "payment")
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() ->
            paymentClient.charge(request)
        );
    }

    // Fallback method - same signature + Throwable
    public CompletableFuture<PaymentResponse> paymentFallback(
            PaymentRequest request, Throwable throwable) {

        log.warn("Payment fallback triggered for order {}: {}",
                request.getOrderId(), throwable.getMessage());

        // Queue for retry or return pending status
        return CompletableFuture.completedFuture(
            PaymentResponse.pending(request.getOrderId(),
                "Payment processing delayed")
        );
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      payment:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        waitDurationInOpenState: 30s
        failureRateThreshold: 50
        slowCallRateThreshold: 50
        slowCallDurationThreshold: 2s
        recordExceptions:
          - java.net.ConnectException
          - java.net.SocketTimeoutException
          - org.springframework.web.client.HttpServerErrorException
        ignoreExceptions:
          - com.example.BusinessException

  retry:
    instances:
      payment:
        maxAttempts: 3
        waitDuration: 1s
        exponentialBackoffMultiplier: 2
        retryExceptions:
          - java.net.ConnectException
        ignoreExceptions:
          - com.example.InvalidRequestException

  timelimiter:
    instances:
      payment:
        timeoutDuration: 3s
        cancelRunningFuture: true

  bulkhead:
    instances:
      payment:
        maxConcurrentCalls: 10
        maxWaitDuration: 500ms
```

### Monitoring Circuit Breaker State

```java
@RestController
@RequestMapping("/actuator/circuitbreaker")
public class CircuitBreakerController {

    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;

    @GetMapping("/status")
    public Map<String, Object> getStatus() {
        return circuitBreakerRegistry.getAllCircuitBreakers()
                .stream()
                .collect(Collectors.toMap(
                    CircuitBreaker::getName,
                    cb -> Map.of(
                        "state", cb.getState(),
                        "failureRate", cb.getMetrics().getFailureRate(),
                        "slowCallRate", cb.getMetrics().getSlowCallRate(),
                        "bufferedCalls", cb.getMetrics().getNumberOfBufferedCalls()
                    )
                ));
    }
}
```

---

## Distributed Configuration

### Spring Cloud Config Server

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>

@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/company/config-repo
          default-label: main
          search-paths: "{application}"
          clone-on-start: true

        # Or use native file system
        native:
          search-locations: file:///config-repo/
```

### Config Client

```yaml
# bootstrap.yml (or spring.config.import in newer versions)
spring:
  application:
    name: order-service
  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true
      retry:
        max-attempts: 5
        initial-interval: 1000

# Or using spring.config.import (Spring Boot 2.4+)
spring:
  config:
    import: configserver:http://localhost:8888
```

### Dynamic Configuration Refresh

```java
@RestController
@RefreshScope  // Beans are recreated on refresh
public class OrderController {

    @Value("${order.max-items:10}")
    private int maxItems;

    @GetMapping("/config")
    public Map<String, Object> getConfig() {
        return Map.of("maxItems", maxItems);
    }
}

// Trigger refresh via POST /actuator/refresh
// Or use Spring Cloud Bus for cluster-wide refresh
```

---

## Distributed Tracing

### Micrometer Tracing (Replaces Sleuth)

```java
// pom.xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

```yaml
management:
  tracing:
    sampling:
      probability: 1.0 # 100% sampling for dev
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans

logging:
  pattern:
    level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]"
```

### Custom Span Creation

```java
@Service
public class OrderService {

    @Autowired
    private Tracer tracer;

    public Order processOrder(OrderRequest request) {
        Span span = tracer.nextSpan().name("process-order");

        try (Tracer.SpanInScope ws = tracer.withSpan(span.start())) {
            span.tag("order.customer", request.getCustomerId());
            span.tag("order.items", String.valueOf(request.getItems().size()));

            // Business logic
            Order order = createOrder(request);

            span.tag("order.id", order.getId().toString());
            span.event("order-created");

            return order;
        } catch (Exception e) {
            span.error(e);
            throw e;
        } finally {
            span.end();
        }
    }
}
```

### Propagating Trace Context

```java
// Automatic with RestTemplate/WebClient/Feign when configured

// Manual propagation
@Service
public class AsyncService {

    @Autowired
    private Tracer tracer;

    @Async
    public CompletableFuture<Void> processAsync(String data) {
        // Context is automatically propagated with proper configuration
        Span currentSpan = tracer.currentSpan();
        log.info("Processing in span: {}", currentSpan.context().traceId());

        return CompletableFuture.completedFuture(null);
    }
}
```

---

## Event-Driven Microservices

### Spring Cloud Stream with Kafka

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
```

### Functional Style (Modern Approach)

```java
// Producer
@Component
public class OrderEventPublisher {

    @Autowired
    private StreamBridge streamBridge;

    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getCustomerId(),
            order.getTotalAmount(),
            Instant.now()
        );

        streamBridge.send("order-events-out-0", event);
    }
}

// Consumer
@Configuration
public class OrderEventConsumer {

    @Bean
    public Consumer<OrderCreatedEvent> orderCreated() {
        return event -> {
            log.info("Received order created event: {}", event.orderId());
            // Process event
        };
    }

    @Bean
    public Function<OrderCreatedEvent, InventoryReservationEvent> reserveInventory() {
        return event -> {
            // Process and produce new event
            return new InventoryReservationEvent(
                event.orderId(),
                "RESERVED",
                Instant.now()
            );
        };
    }
}
```

```yaml
spring:
  cloud:
    stream:
      bindings:
        order-events-out-0:
          destination: order-events
          content-type: application/json
        orderCreated-in-0:
          destination: order-events
          group: inventory-service
          content-type: application/json
        reserveInventory-in-0:
          destination: order-events
          group: inventory-service
        reserveInventory-out-0:
          destination: inventory-events

      kafka:
        binder:
          brokers: localhost:9092
          auto-create-topics: true
        bindings:
          orderCreated-in-0:
            consumer:
              enable-dlq: true
              dlq-name: order-events-dlq
```

### Dead Letter Queue Handling

```java
@Bean
public Consumer<Message<OrderCreatedEvent>> orderCreatedWithDlq() {
    return message -> {
        try {
            processOrder(message.getPayload());
        } catch (Exception e) {
            // Will be sent to DLQ automatically
            throw e;
        }
    };
}

// Manual DLQ processing
@Bean
public Consumer<Message<OrderCreatedEvent>> processDlq() {
    return message -> {
        log.error("Processing DLQ message: {}", message.getPayload());
        // Alert, store for manual review, etc.
    };
}
```

---

## Saga Pattern for Distributed Transactions

### Choreography-Based Saga

```java
// Each service listens and reacts to events

// Order Service
@Component
public class OrderSaga {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private StreamBridge streamBridge;

    // Step 1: Create order and publish event
    public Order createOrder(OrderRequest request) {
        Order order = new Order(request);
        order.setStatus(OrderStatus.PENDING);
        order = orderRepository.save(order);

        streamBridge.send("order-created", new OrderCreatedEvent(order));
        return order;
    }

    // Step 4: Handle payment result
    @Bean
    public Consumer<PaymentCompletedEvent> paymentCompleted() {
        return event -> {
            Order order = orderRepository.findById(event.orderId()).orElseThrow();
            order.setStatus(OrderStatus.CONFIRMED);
            orderRepository.save(order);

            streamBridge.send("order-confirmed", new OrderConfirmedEvent(order));
        };
    }

    // Compensation: Handle payment failure
    @Bean
    public Consumer<PaymentFailedEvent> paymentFailed() {
        return event -> {
            Order order = orderRepository.findById(event.orderId()).orElseThrow();
            order.setStatus(OrderStatus.CANCELLED);
            orderRepository.save(order);

            // Trigger inventory release
            streamBridge.send("inventory-release",
                new InventoryReleaseEvent(order.getId(), order.getItems()));
        };
    }
}

// Inventory Service
@Component
public class InventorySaga {

    // Step 2: Reserve inventory
    @Bean
    public Consumer<OrderCreatedEvent> reserveInventory() {
        return event -> {
            try {
                inventoryService.reserve(event.orderId(), event.items());
                streamBridge.send("inventory-reserved",
                    new InventoryReservedEvent(event.orderId()));
            } catch (InsufficientInventoryException e) {
                streamBridge.send("inventory-failed",
                    new InventoryFailedEvent(event.orderId(), e.getMessage()));
            }
        };
    }
}

// Payment Service
@Component
public class PaymentSaga {

    // Step 3: Process payment after inventory reserved
    @Bean
    public Consumer<InventoryReservedEvent> processPayment() {
        return event -> {
            try {
                paymentService.charge(event.orderId());
                streamBridge.send("payment-completed",
                    new PaymentCompletedEvent(event.orderId()));
            } catch (PaymentException e) {
                streamBridge.send("payment-failed",
                    new PaymentFailedEvent(event.orderId(), e.getMessage()));
            }
        };
    }
}
```

### Orchestration-Based Saga (Saga Orchestrator)

```java
@Service
public class OrderSagaOrchestrator {

    @Autowired
    private OrderRepository orderRepository;
    @Autowired
    private InventoryClient inventoryClient;
    @Autowired
    private PaymentClient paymentClient;
    @Autowired
    private ShippingClient shippingClient;

    @Transactional
    public Order executeOrderSaga(OrderRequest request) {
        Order order = new Order(request);
        order.setStatus(OrderStatus.PENDING);
        order = orderRepository.save(order);

        try {
            // Step 1: Reserve inventory
            InventoryReservation reservation = inventoryClient.reserve(
                order.getId(), order.getItems());
            order.setInventoryReservationId(reservation.getId());

            // Step 2: Process payment
            PaymentResult payment = paymentClient.charge(
                order.getId(), order.getTotalAmount());
            order.setPaymentId(payment.getId());

            // Step 3: Create shipment
            Shipment shipment = shippingClient.createShipment(order);
            order.setShipmentId(shipment.getId());

            order.setStatus(OrderStatus.CONFIRMED);
            return orderRepository.save(order);

        } catch (InventoryException e) {
            order.setStatus(OrderStatus.FAILED);
            orderRepository.save(order);
            throw new OrderFailedException("Inventory unavailable", e);

        } catch (PaymentException e) {
            // Compensate: Release inventory
            compensateInventory(order);
            order.setStatus(OrderStatus.FAILED);
            orderRepository.save(order);
            throw new OrderFailedException("Payment failed", e);

        } catch (ShippingException e) {
            // Compensate: Refund payment, release inventory
            compensatePayment(order);
            compensateInventory(order);
            order.setStatus(OrderStatus.FAILED);
            orderRepository.save(order);
            throw new OrderFailedException("Shipping failed", e);
        }
    }

    private void compensateInventory(Order order) {
        if (order.getInventoryReservationId() != null) {
            inventoryClient.release(order.getInventoryReservationId());
        }
    }

    private void compensatePayment(Order order) {
        if (order.getPaymentId() != null) {
            paymentClient.refund(order.getPaymentId());
        }
    }
}
```

---

## Security in Microservices

### OAuth2 Resource Server

```java
// pom.xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://auth.example.com/realms/myapp
          # or jwk-set-uri: https://auth.example.com/.well-known/jwks.json
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/**").authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            );

        return http.build();
    }

    private JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter =
            new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");

        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
        return converter;
    }
}
```

### Service-to-Service Authentication

```java
// Propagate JWT token via Feign
@Component
public class FeignAuthInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate template) {
        SecurityContext context = SecurityContextHolder.getContext();
        if (context.getAuthentication() instanceof JwtAuthenticationToken jwt) {
            template.header("Authorization", "Bearer " + jwt.getToken().getTokenValue());
        }
    }
}
```

---

## Interview Questions

### Fundamentals

1. **When would you choose microservices over monolith?**

   - Large teams, complex domains, need independent scaling, different tech requirements. NOT for small apps, MVPs, or small teams.

2. **What is service discovery and why is it needed?**

   - Dynamic IP resolution. Services register with discovery server, clients query for available instances. Essential in containerized/cloud environments.

3. **What's the difference between API Gateway and Load Balancer?**
   - LB distributes traffic. Gateway handles cross-cutting concerns (auth, rate limiting, routing, protocol translation). Gateway typically sits in front of LB.

### Resilience

4. **Explain circuit breaker states and transitions.**

   - CLOSED: Normal operation. OPEN: After failure threshold, rejects calls. HALF_OPEN: After wait, allows test requests to determine recovery.

5. **What's the difference between retry and circuit breaker?**

   - Retry: Repeat failed calls immediately/with backoff. Circuit breaker: Prevent calls to failing service entirely for a duration.

6. **How do you prevent cascading failures?**
   - Circuit breakers, timeouts, bulkheads (isolate thread pools), graceful degradation, fallbacks.

### Communication

7. **When to use sync vs async communication?**

   - Sync (REST/gRPC): Immediate response needed, simple request-response. Async (messaging): Decoupling, eventual consistency OK, long-running operations.

8. **What is the outbox pattern?**
   - Write domain event to outbox table in same DB transaction as business data. Separate relay publishes to message broker. Ensures consistency.

### Transactions

9. **How do you handle distributed transactions?**

   - Avoid if possible. Use sagas (choreography or orchestration), eventual consistency, compensating transactions.

10. **Choreography vs Orchestration saga - when to use which?**
    - Choreography: Simple flows, services emit events. Orchestration: Complex flows, central coordinator manages steps. Orchestration better for visibility and complex compensation.

### Deployment

11. **How do you deploy microservices safely?**

    - Blue-green, canary, rolling deployments. Feature flags. Health checks. Rollback capability.

12. **How do you debug issues across services?**
    - Distributed tracing (Zipkin/Jaeger), correlation IDs in logs, centralized logging (ELK), metrics dashboards.
