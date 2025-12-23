# Spring Boot Production Readiness — Complete Interview Guide

## Table of Contents

1. [Production Checklist Overview](#production-checklist-overview)
2. [Spring Boot Actuator Deep Dive](#spring-boot-actuator-deep-dive)
3. [Health Checks & Probes](#health-checks--probes)
4. [Metrics with Micrometer](#metrics-with-micrometer)
5. [Logging Best Practices](#logging-best-practices)
6. [Configuration Management](#configuration-management)
7. [Security Hardening](#security-hardening)
8. [Performance Optimization](#performance-optimization)
9. [Connection Pool Tuning](#connection-pool-tuning)
10. [Graceful Shutdown](#graceful-shutdown)
11. [Containerization (Docker)](#containerization-docker)
12. [Kubernetes Deployment](#kubernetes-deployment)
13. [Production Debugging](#production-debugging)
14. [Incident Response](#incident-response)
15. [Interview Questions](#interview-questions)

---

## Production Checklist Overview

### Before Going to Production

```
┌────────────────────────────────────────────────────────────────────┐
│                    PRODUCTION READINESS CHECKLIST                   │
├────────────────────────────────────────────────────────────────────┤
│ □ Health endpoints configured (/health, /ready, /live)            │
│ □ Metrics exposed (Prometheus/Micrometer)                          │
│ □ Structured logging with correlation IDs                          │
│ □ Externalized configuration (env vars, config server)             │
│ □ Secrets management (Vault, K8s secrets)                          │
│ □ Connection pools properly sized (DB, HTTP clients)               │
│ □ Timeouts configured on all external calls                        │
│ □ Circuit breakers for critical dependencies                        │
│ □ Graceful shutdown enabled                                        │
│ □ Security headers and HTTPS enforced                              │
│ □ Rate limiting configured                                         │
│ □ Error handling and meaningful error responses                    │
│ □ API documentation (OpenAPI/Swagger)                              │
│ □ Containerized with optimized Docker image                        │
│ □ Resource limits defined (CPU, memory)                            │
│ □ Alerting configured for critical metrics                         │
└────────────────────────────────────────────────────────────────────┘
```

---

## Spring Boot Actuator Deep Dive

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Configuration

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,env,loggers,threaddump,heapdump
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized
      show-components: always
      probes:
        enabled: true
    shutdown:
      enabled: true # Enable graceful shutdown endpoint
  info:
    env:
      enabled: true
    git:
      enabled: true
      mode: full
    build:
      enabled: true
```

### Key Actuator Endpoints

| Endpoint            | Purpose                   | Production Use          |
| ------------------- | ------------------------- | ----------------------- |
| `/health`           | Application health        | Load balancer checks    |
| `/health/liveness`  | Is app running?           | K8s liveness probe      |
| `/health/readiness` | Can accept traffic?       | K8s readiness probe     |
| `/metrics`          | Application metrics       | Monitoring              |
| `/prometheus`       | Prometheus format metrics | Scraping                |
| `/info`             | Build/git info            | Deployment verification |
| `/loggers`          | View/change log levels    | Runtime debugging       |
| `/threaddump`       | Thread dump               | Debugging hangs         |
| `/heapdump`         | Heap dump                 | Memory analysis         |
| `/env`              | Environment properties    | Config verification     |

### Securing Actuator Endpoints

```java
@Configuration
@EnableWebSecurity
public class ActuatorSecurityConfig {

    @Bean
    public SecurityFilterChain actuatorSecurity(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/actuator/**")
            .authorizeHttpRequests(auth -> auth
                // Public health endpoints for load balancers
                .requestMatchers("/actuator/health/**").permitAll()
                .requestMatchers("/actuator/info").permitAll()
                .requestMatchers("/actuator/prometheus").permitAll()
                // Restricted endpoints
                .requestMatchers("/actuator/env/**").hasRole("ADMIN")
                .requestMatchers("/actuator/loggers/**").hasRole("ADMIN")
                .requestMatchers("/actuator/threaddump").hasRole("ADMIN")
                .requestMatchers("/actuator/heapdump").hasRole("ADMIN")
                .requestMatchers("/actuator/shutdown").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());

        return http.build();
    }
}
```

---

## Health Checks & Probes

### Custom Health Indicator

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    @Autowired
    private DataSource dataSource;

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(5)) {
                return Health.up()
                        .withDetail("database", "PostgreSQL")
                        .withDetail("validationTime", "5s")
                        .build();
            }
        } catch (SQLException e) {
            return Health.down()
                    .withException(e)
                    .build();
        }
        return Health.down().build();
    }
}

// Reactive health indicator
@Component
public class ExternalServiceHealthIndicator implements ReactiveHealthIndicator {

    @Autowired
    private WebClient webClient;

    @Override
    public Mono<Health> health() {
        return webClient.get()
                .uri("https://api.external.com/health")
                .retrieve()
                .bodyToMono(String.class)
                .map(response -> Health.up().withDetail("service", "available").build())
                .onErrorResume(e -> Mono.just(
                        Health.down().withDetail("error", e.getMessage()).build()))
                .timeout(Duration.ofSeconds(3));
    }
}
```

### Health Groups (Kubernetes Probes)

```yaml
management:
  endpoint:
    health:
      group:
        liveness:
          include: livenessState
        readiness:
          include: readinessState, db, redis, kafka
          show-details: always
```

```java
// Custom liveness/readiness contribution
@Component
public class StartupHealthContributor implements ApplicationListener<ApplicationReadyEvent> {

    @Autowired
    private ApplicationAvailability availability;

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        // Signal that app is ready to accept traffic
        eventPublisher.publishEvent(
            AvailabilityChangeEvent.publish(event.getApplicationContext(),
                ReadinessState.ACCEPTING_TRAFFIC));
    }

    // Programmatically change availability
    public void setNotReady(String reason) {
        eventPublisher.publishEvent(
            AvailabilityChangeEvent.publish(this, ReadinessState.REFUSING_TRAFFIC));
    }
}
```

---

## Metrics with Micrometer

### Setup

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### Custom Metrics

```java
@Component
public class OrderMetrics {

    private final Counter ordersCreated;
    private final Counter ordersFailed;
    private final Timer orderProcessingTime;
    private final AtomicInteger activeOrders;
    private final DistributionSummary orderAmount;

    public OrderMetrics(MeterRegistry registry) {
        // Counter: monotonically increasing
        this.ordersCreated = Counter.builder("orders.created")
                .description("Total orders created")
                .tag("type", "all")
                .register(registry);

        this.ordersFailed = Counter.builder("orders.failed")
                .description("Total orders failed")
                .register(registry);

        // Timer: duration + count
        this.orderProcessingTime = Timer.builder("orders.processing.time")
                .description("Order processing time")
                .publishPercentiles(0.5, 0.95, 0.99)
                .publishPercentileHistogram()
                .register(registry);

        // Gauge: current value
        this.activeOrders = registry.gauge("orders.active", new AtomicInteger(0));

        // Distribution Summary: value distribution
        this.orderAmount = DistributionSummary.builder("orders.amount")
                .description("Order amounts")
                .baseUnit("USD")
                .publishPercentiles(0.5, 0.75, 0.95)
                .register(registry);
    }

    public void recordOrderCreated() {
        ordersCreated.increment();
    }

    public void recordOrderFailed(String reason) {
        ordersFailed.increment();
    }

    public void recordOrderAmount(double amount) {
        orderAmount.record(amount);
    }

    public Timer.Sample startTimer() {
        return Timer.start();
    }

    public void stopTimer(Timer.Sample sample) {
        sample.stop(orderProcessingTime);
    }

    public void incrementActiveOrders() {
        activeOrders.incrementAndGet();
    }

    public void decrementActiveOrders() {
        activeOrders.decrementAndGet();
    }
}
```

### Using @Timed Annotation

```java
@Service
public class OrderService {

    @Timed(value = "order.create",
           description = "Time taken to create order",
           percentiles = {0.5, 0.95, 0.99})
    public Order createOrder(OrderRequest request) {
        // Business logic
        return new Order();
    }
}

// Enable @Timed
@Configuration
public class MetricsConfig {

    @Bean
    public TimedAspect timedAspect(MeterRegistry registry) {
        return new TimedAspect(registry);
    }
}
```

### Key Metrics to Monitor

```yaml
# Essential Spring Boot metrics
jvm.memory.used
jvm.memory.max
jvm.gc.pause
jvm.threads.live
jvm.threads.peak

# HTTP metrics
http.server.requests (count, sum, max by uri, method, status)

# Database connection pool
hikaricp.connections.active
hikaricp.connections.idle
hikaricp.connections.pending
hikaricp.connections.timeout

# Custom business metrics
orders.created
orders.processing.time
orders.amount
```

---

## Logging Best Practices

### Structured Logging with Logback

```xml
<!-- logback-spring.xml -->
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <!-- JSON format for production -->
    <springProfile name="prod">
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder">
                <includeMdcKeyName>traceId</includeMdcKeyName>
                <includeMdcKeyName>spanId</includeMdcKeyName>
                <includeMdcKeyName>userId</includeMdcKeyName>
                <customFields>{"service":"order-service","env":"prod"}</customFields>
            </encoder>
        </appender>
    </springProfile>

    <!-- Human-readable for dev -->
    <springProfile name="dev">
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] [%X{traceId:-}] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
    </springProfile>

    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>

    <!-- Reduce noise -->
    <logger name="org.hibernate.SQL" level="WARN"/>
    <logger name="org.springframework.security" level="WARN"/>
</configuration>
```

### MDC for Request Context

```java
@Component
public class RequestContextFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {
        try {
            // Add context to all log statements in this request
            MDC.put("requestId", getOrGenerateRequestId(request));
            MDC.put("userId", extractUserId(request));
            MDC.put("clientIp", request.getRemoteAddr());
            MDC.put("uri", request.getRequestURI());

            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }

    private String getOrGenerateRequestId(HttpServletRequest request) {
        String requestId = request.getHeader("X-Request-ID");
        return requestId != null ? requestId : UUID.randomUUID().toString();
    }
}
```

### Logging Guidelines

```java
@Service
@Slf4j
public class OrderService {

    public Order createOrder(OrderRequest request) {
        // DO: Log entry with relevant context
        log.info("Creating order for customer: {}, items: {}",
                request.getCustomerId(), request.getItems().size());

        // DON'T: Log sensitive data
        // log.info("Order details: {}", request);  // May contain PII

        try {
            Order order = processOrder(request);

            // DO: Log success with key identifiers
            log.info("Order created successfully: orderId={}, amount={}",
                    order.getId(), order.getTotalAmount());

            return order;
        } catch (Exception e) {
            // DO: Log errors with context
            log.error("Failed to create order for customer {}: {}",
                    request.getCustomerId(), e.getMessage(), e);
            throw e;
        }
    }
}
```

### Dynamic Log Level Change

```bash
# View current level
curl http://localhost:8080/actuator/loggers/com.example.service

# Change level at runtime
curl -X POST http://localhost:8080/actuator/loggers/com.example.service \
  -H 'Content-Type: application/json' \
  -d '{"configuredLevel": "DEBUG"}'
```

---

## Configuration Management

### Externalized Configuration

```yaml
# application.yml - defaults
spring:
  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/mydb}
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:}

order:
  max-items: ${ORDER_MAX_ITEMS:100}
  processing:
    timeout: ${ORDER_TIMEOUT:30s}
```

### Type-Safe Configuration

```java
@Configuration
@ConfigurationProperties(prefix = "order")
@Validated
public class OrderProperties {

    @NotNull
    @Min(1)
    @Max(1000)
    private Integer maxItems;

    @Valid
    private Processing processing = new Processing();

    @Valid
    private Retry retry = new Retry();

    // Nested configuration
    public static class Processing {
        @DurationUnit(ChronoUnit.SECONDS)
        private Duration timeout = Duration.ofSeconds(30);

        @Min(1)
        private int threadPoolSize = 10;

        // getters/setters
    }

    public static class Retry {
        private int maxAttempts = 3;
        private Duration delay = Duration.ofMillis(100);

        // getters/setters
    }

    // getters/setters
}

// Usage
@Service
public class OrderService {

    private final OrderProperties properties;

    public OrderService(OrderProperties properties) {
        this.properties = properties;
    }

    public void validateOrder(OrderRequest request) {
        if (request.getItems().size() > properties.getMaxItems()) {
            throw new ValidationException("Too many items");
        }
    }
}
```

### Profile-Based Configuration

```yaml
# application-prod.yml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5

logging:
  level:
    root: WARN
    com.example: INFO
```

---

## Security Hardening

### Security Headers

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers(headers -> headers
                .contentTypeOptions(Customizer.withDefaults())
                .xssProtection(Customizer.withDefaults())
                .frameOptions(frame -> frame.sameOrigin())
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000))
                .contentSecurityPolicy(csp ->
                    csp.policyDirectives("default-src 'self'"))
            )
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            );

        return http.build();
    }
}
```

### Input Validation

```java
@RestController
@Validated
public class OrderController {

    @PostMapping("/orders")
    public ResponseEntity<Order> createOrder(
            @Valid @RequestBody OrderRequest request) {
        return ResponseEntity.ok(orderService.create(request));
    }
}

public class OrderRequest {

    @NotNull(message = "Customer ID is required")
    @Size(max = 50)
    private String customerId;

    @NotEmpty(message = "At least one item required")
    @Size(max = 100, message = "Maximum 100 items allowed")
    @Valid
    private List<OrderItem> items;

    @Email
    private String email;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$")
    private String phone;
}

public class OrderItem {
    @NotNull
    private String productId;

    @Min(1)
    @Max(1000)
    private int quantity;
}
```

### Secrets Management

```yaml
# Using environment variables (K8s secrets)
spring:
  datasource:
    password: ${DB_PASSWORD}

# Using Vault
spring:
  cloud:
    vault:
      uri: https://vault.example.com
      token: ${VAULT_TOKEN}
      kv:
        enabled: true
        backend: secret
        default-context: order-service
```

---

## Performance Optimization

### JVM Tuning

```dockerfile
# Dockerfile
ENV JAVA_OPTS="-XX:+UseG1GC \
    -XX:MaxGCPauseMillis=200 \
    -XX:+UseStringDeduplication \
    -XX:+HeapDumpOnOutOfMemoryError \
    -XX:HeapDumpPath=/tmp/heapdump.hprof \
    -Xms512m \
    -Xmx2g"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### Virtual Threads (Java 21+)

```yaml
spring:
  threads:
    virtual:
      enabled: true # Enable virtual threads for request handling
```

```java
// Or configure manually
@Bean
public TomcatProtocolHandlerCustomizer<?> protocolHandlerCustomizer() {
    return handler -> handler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
}
```

### Response Compression

```yaml
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/xml,text/plain
    min-response-size: 1024
```

### Caching

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                .maximumSize(1000)
                .expireAfterWrite(Duration.ofMinutes(10))
                .recordStats());
        return cacheManager;
    }
}

@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) {
        return productRepository.findById(id).orElseThrow();
    }

    @CacheEvict(value = "products", key = "#product.id")
    public Product update(Product product) {
        return productRepository.save(product);
    }

    @CacheEvict(value = "products", allEntries = true)
    public void clearCache() {
        // Clears all cached products
    }
}
```

---

## Connection Pool Tuning

### HikariCP Configuration

```yaml
spring:
  datasource:
    hikari:
      # Pool sizing
      maximum-pool-size: 20 # Max connections
      minimum-idle: 5 # Min idle connections

      # Timeouts
      connection-timeout: 30000 # Wait for connection (ms)
      idle-timeout: 600000 # Idle connection lifetime (ms)
      max-lifetime: 1800000 # Max connection lifetime (ms)

      # Validation
      validation-timeout: 5000

      # Leak detection (for debugging)
      leak-detection-threshold: 60000

      # Pool name for metrics
      pool-name: order-service-pool
```

### Sizing Guidelines

```
Formula: pool size = (core_count * 2) + spindle_count

For typical web apps:
- CPU-bound: cores × 2
- I/O-bound: Higher, but watch DB connection limits

Example:
- 8-core server, PostgreSQL (1 spindle)
- Base pool: (8 × 2) + 1 = 17
- With connection overhead: ~20 connections
```

### HTTP Client Pool (RestTemplate/WebClient)

```java
@Configuration
public class HttpClientConfig {

    @Bean
    public RestTemplate restTemplate() {
        HttpComponentsClientHttpRequestFactory factory =
            new HttpComponentsClientHttpRequestFactory();

        PoolingHttpClientConnectionManager connectionManager =
            new PoolingHttpClientConnectionManager();
        connectionManager.setMaxTotal(100);           // Max total connections
        connectionManager.setDefaultMaxPerRoute(20);  // Max per host

        CloseableHttpClient httpClient = HttpClients.custom()
            .setConnectionManager(connectionManager)
            .setDefaultRequestConfig(RequestConfig.custom()
                .setConnectTimeout(Timeout.ofSeconds(5))
                .setResponseTimeout(Timeout.ofSeconds(30))
                .build())
            .build();

        factory.setHttpClient(httpClient);
        return new RestTemplate(factory);
    }

    @Bean
    public WebClient webClient() {
        HttpClient httpClient = HttpClient.create()
            .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
            .responseTimeout(Duration.ofSeconds(30))
            .doOnConnected(conn -> conn
                .addHandlerLast(new ReadTimeoutHandler(30))
                .addHandlerLast(new WriteTimeoutHandler(30)));

        return WebClient.builder()
            .clientConnector(new ReactorClientHttpConnector(httpClient))
            .build();
    }
}
```

---

## Graceful Shutdown

### Configuration

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

### Custom Shutdown Logic

```java
@Component
public class GracefulShutdownHandler implements DisposableBean {

    @Autowired
    private ExecutorService executorService;

    @Autowired
    private KafkaListenerEndpointRegistry kafkaRegistry;

    @Override
    public void destroy() throws Exception {
        log.info("Starting graceful shutdown...");

        // 1. Stop accepting new messages
        kafkaRegistry.stop();
        log.info("Kafka listeners stopped");

        // 2. Wait for in-flight requests
        executorService.shutdown();
        if (!executorService.awaitTermination(25, TimeUnit.SECONDS)) {
            log.warn("Forcing executor shutdown");
            executorService.shutdownNow();
        }

        log.info("Graceful shutdown complete");
    }
}

// SmartLifecycle for ordered shutdown
@Component
public class OrderedShutdown implements SmartLifecycle {

    private volatile boolean running = false;

    @Override
    public void start() {
        running = true;
    }

    @Override
    public void stop() {
        // Cleanup logic
        running = false;
    }

    @Override
    public boolean isRunning() {
        return running;
    }

    @Override
    public int getPhase() {
        return Integer.MAX_VALUE - 1;  // Shut down early
    }
}
```

---

## Containerization (Docker)

### Optimized Dockerfile

```dockerfile
# Multi-stage build
FROM eclipse-temurin:21-jdk-alpine AS builder

WORKDIR /app
COPY pom.xml .
COPY src ./src

# Cache dependencies
RUN --mount=type=cache,target=/root/.m2 \
    mvn package -DskipTests

# Extract layers for better caching
RUN java -Djarmode=layertools -jar target/*.jar extract

# Production image
FROM eclipse-temurin:21-jre-alpine

# Security: non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

WORKDIR /app

# Copy layers in order of change frequency
COPY --from=builder /app/dependencies/ ./
COPY --from=builder /app/spring-boot-loader/ ./
COPY --from=builder /app/snapshot-dependencies/ ./
COPY --from=builder /app/application/ ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=30s --retries=3 \
    CMD wget -q --spider http://localhost:8080/actuator/health || exit 1

EXPOSE 8080

ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-Djava.security.egd=file:/dev/./urandom", \
    "org.springframework.boot.loader.launch.JarLauncher"]
```

### Docker Compose for Local Development

```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - DB_URL=jdbc:postgresql://db:5432/mydb
      - DB_USERNAME=postgres
      - DB_PASSWORD=secret
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

---

## Kubernetes Deployment

### Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/actuator/prometheus"
    spec:
      containers:
        - name: order-service
          image: order-service:1.0.0
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: password
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "2Gi"
              cpu: "1000m"
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command: ["sh", "-c", "sleep 10"] # Allow time for deregistration
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

---

## Interview Questions

### Production Readiness

1. **What's the difference between liveness and readiness probes?**

   - Liveness: Is the app running? Fail = restart container. Readiness: Can it accept traffic? Fail = remove from load balancer. Both essential for K8s deployments.

2. **How do you handle configuration in production?**

   - Environment variables, Config Server, K8s ConfigMaps/Secrets. Never hardcode secrets. Use @ConfigurationProperties for type safety.

3. **What metrics should you monitor in production?**
   - RED: Rate, Errors, Duration. Plus: JVM heap, GC pauses, connection pool utilization, thread counts, business metrics.

### Performance

4. **How do you size a connection pool?**

   - Formula: `(cores × 2) + spindle_count`. Consider: DB max connections shared across instances, I/O vs CPU bound workload.

5. **How do you debug memory issues in production?**

   - Monitor JVM metrics (heap, GC). Enable heap dump on OOM. Use `/actuator/heapdump` for analysis. Consider tools like MAT, VisualVM.

6. **What is graceful shutdown and why is it important?**
   - Complete in-flight requests before terminating. Prevents data loss and failed requests. Configure `server.shutdown=graceful` and proper timeouts.

### Security

7. **How do you secure actuator endpoints?**

   - Different security for different endpoints. Health/metrics public for load balancers/Prometheus. Admin endpoints require authentication.

8. **How do you manage secrets in Spring Boot?**
   - Never in source code. Use env vars, K8s secrets, Vault. Spring Cloud Vault for automatic secret injection.

### Observability

9. **What is structured logging and why use it?**

   - JSON format with consistent fields. Enables machine parsing, searching, alerting. Essential for centralized logging (ELK, Splunk).

10. **How do you change log level without restart?**
    - Use `/actuator/loggers` endpoint. POST with new level. Useful for production debugging without deployment.
