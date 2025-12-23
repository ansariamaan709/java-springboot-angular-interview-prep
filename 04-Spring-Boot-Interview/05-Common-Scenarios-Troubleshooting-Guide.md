# Spring Boot Common Scenarios & Troubleshooting - Complete Interview Guide

## Table of Contents

1. [Database Connection Issues](#database-connection-issues)
2. [Application Startup Failures](#application-startup-failures)
3. [Memory Leaks & OutOfMemoryError](#memory-leaks--outofmemoryerror)
4. [Performance Issues](#performance-issues)
5. [Security Issues](#security-issues)
6. [Microservices Communication Failures](#microservices-communication-failures)
7. [Transaction Management Issues](#transaction-management-issues)
8. [Common Configuration Mistakes](#common-configuration-mistakes)
9. [Debugging Techniques](#debugging-techniques)
10. [Production Incidents & Resolution](#production-incidents--resolution)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## Database Connection Issues

### 1. Connection Pool Exhausted

**Symptom:**

```
Caused by: java.sql.SQLTransientConnectionException:
HikariPool-1 - Connection is not available, request timed out after 30000ms.
```

**Root Causes:**

1. Connection leaks (not closing connections)
2. Too many concurrent requests
3. Small connection pool size
4. Long-running transactions

**Solution:**

```yaml
# application.yml - Tune HikariCP
spring:
  datasource:
    hikari:
      maximum-pool-size: 20 # Increase from default 10
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      leak-detection-threshold: 60000 # Detect leaks after 60s
```

**Diagnosis:**

```java
// Enable leak detection
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setLeakDetectionThreshold(60000); // 60 seconds
        config.setRegisterMbeans(true); // Enable JMX monitoring

        // Log pool stats
        HikariDataSource ds = new HikariDataSource(config);
        logPoolStats(ds);
        return ds;
    }

    @Scheduled(fixedDelay = 30000)
    public void logPoolStats(HikariDataSource dataSource) {
        HikariPoolMXBean poolMBean = dataSource.getHikariPoolMXBean();
        log.info("Pool stats - Active: {}, Idle: {}, Total: {}, Waiting: {}",
                poolMBean.getActiveConnections(),
                poolMBean.getIdleConnections(),
                poolMBean.getTotalConnections(),
                poolMBean.getThreadsAwaitingConnection());
    }
}
```

**Fix Connection Leaks:**

```java
// BAD - Connection leak
public List<User> getUsers() {
    Connection conn = dataSource.getConnection();
    // ... query logic
    return users; // Connection never closed!
}

// GOOD - Use try-with-resources
public List<User> getUsers() {
    try (Connection conn = dataSource.getConnection();
         PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users");
         ResultSet rs = stmt.executeQuery()) {
        // Process results
        return users;
    }
}

// BETTER - Use Spring Data JPA (auto-manages connections)
public interface UserRepository extends JpaRepository<User, Long> {
}
```

---

### 2. Database Lock Timeout

**Symptom:**

```
ERROR: deadlock detected
Detail: Process 12345 waits for ShareLock on transaction 67890
```

**Solution:**

```java
@Service
public class OrderService {

    // Acquire locks in consistent order to prevent deadlocks
    @Transactional
    public void transferFunds(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        // ALWAYS lock accounts in same order (ascending ID)
        Long firstId = Math.min(fromAccountId, toAccountId);
        Long secondId = Math.max(fromAccountId, toAccountId);

        Account firstAccount = accountRepository.findByIdForUpdate(firstId);
        Account secondAccount = accountRepository.findByIdForUpdate(secondId);

        if (fromAccountId.equals(firstId)) {
            firstAccount.debit(amount);
            secondAccount.credit(amount);
        } else {
            secondAccount.credit(amount);
            firstAccount.debit(amount);
        }
    }
}

// Repository with pessimistic locking
public interface AccountRepository extends JpaRepository<Account, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT a FROM Account a WHERE a.id = :id")
    Account findByIdForUpdate(@Param("id") Long id);
}
```

---

## Application Startup Failures

### 1. Port Already in Use

**Symptom:**

```
Web server failed to start. Port 8080 was already in use.
```

**Solutions:**

```powershell
# Option 1: Kill process on port 8080 (Windows)
netstat -ano | findstr :8080
taskkill /PID <PID> /F

# Option 2: Change port
# application.yml
server:
  port: 8081

# Option 3: Random port (for tests)
server:
  port: 0  # OS assigns random available port
```

---

### 2. Circular Dependency

**Symptom:**

```
The dependencies of some of the beans in the application context form a cycle:
┌─────┐
|  userService defined in file [UserService.class]
↑     ↓
|  orderService defined in file [OrderService.class]
└─────┘
```

**Solutions:**

```java
// PROBLEM - Circular dependency
@Service
public class UserService {
    private final OrderService orderService;

    public UserService(OrderService orderService) {
        this.orderService = orderService;
    }
}

@Service
public class OrderService {
    private final UserService userService;

    public OrderService(UserService userService) {
        this.userService = userService;
    }
}

// SOLUTION 1 - Use @Lazy
@Service
public class UserService {
    private final OrderService orderService;

    public UserService(@Lazy OrderService orderService) {
        this.orderService = orderService;
    }
}

// SOLUTION 2 - Setter injection
@Service
public class UserService {
    private OrderService orderService;

    @Autowired
    public void setOrderService(OrderService orderService) {
        this.orderService = orderService;
    }
}

// SOLUTION 3 - Refactor (BEST)
// Extract common functionality to a separate service
@Service
public class UserService {
    private final NotificationService notificationService;
}

@Service
public class OrderService {
    private final NotificationService notificationService;
}

@Service
public class NotificationService {
    // Common logic
}
```

---

### 3. Bean Not Found

**Symptom:**

```
Field userRepository in UserService required a bean of type
'UserRepository' that could not be found.
```

**Solutions:**

```java
// PROBLEM - Repository not scanned
package com.example.repositories; // Outside component scan

@Repository
public interface UserRepository extends JpaRepository<User, Long> {}

// SOLUTION 1 - Enable JPA repositories
@SpringBootApplication
@EnableJpaRepositories(basePackages = "com.example.repositories")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// SOLUTION 2 - Move to scanned package
// Move UserRepository to com.example.myapp.repositories
// (same package or sub-package as @SpringBootApplication)

// SOLUTION 3 - Explicit component scan
@SpringBootApplication
@ComponentScan(basePackages = {"com.example.myapp", "com.example.repositories"})
public class Application {}
```

---

## Memory Leaks & OutOfMemoryError

### 1. Heap Memory Exhaustion

**Symptom:**

```
java.lang.OutOfMemoryError: Java heap space
```

**Diagnosis:**

```bash
# Enable heap dump on OOM
java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heapdump.hprof -jar app.jar

# Analyze with VisualVM, MAT (Memory Analyzer Tool), or jhat
jhat heapdump.hprof
```

**Common Causes & Fixes:**

```java
// LEAK 1 - Static collections growing unbounded
public class UserCache {
    private static final Map<Long, User> cache = new HashMap<>(); // LEAK!

    public static void cacheUser(User user) {
        cache.put(user.getId(), user); // Never removed!
    }
}

// FIX - Use bounded cache with eviction
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        return new CaffeineCacheManager("users") {
            @Override
            protected Cache createNativeCaffeineCache(String name) {
                return Caffeine.newBuilder()
                        .maximumSize(10000) // Bounded
                        .expireAfterWrite(10, TimeUnit.MINUTES) // TTL
                        .build();
            }
        };
    }
}

// LEAK 2 - Large result sets loaded into memory
public List<User> getAllUsers() {
    return userRepository.findAll(); // Loads millions of rows!
}

// FIX - Use pagination
public Page<User> getUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
}

// Or streaming for batch processing
@Transactional(readOnly = true)
public void processAllUsers() {
    try (Stream<User> userStream = userRepository.streamAllUsers()) {
        userStream.forEach(user -> {
            // Process one at a time
        });
    }
}

@Query("SELECT u FROM User u")
Stream<User> streamAllUsers();
```

---

### 2. Metaspace OutOfMemoryError

**Symptom:**

```
java.lang.OutOfMemoryError: Metaspace
```

**Cause:** Too many classes loaded (common with class reloading in dev tools).

**Solution:**

```bash
# Increase metaspace size
java -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m -jar app.jar

# For dev - disable devtools auto-restart
# application-dev.yml
spring:
  devtools:
    restart:
      enabled: false
```

---

## Performance Issues

### 1. N+1 Query Problem

**Symptom:** Slow performance with many database queries.

```sql
-- 1 query to fetch orders
SELECT * FROM orders;

-- N queries to fetch items for each order
SELECT * FROM order_items WHERE order_id = 1;
SELECT * FROM order_items WHERE order_id = 2;
-- ... (100 more queries)
```

**Detection:**

```yaml
# Enable query logging
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        use_sql_comments: true

logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

**Solution:**

```java
// BAD - N+1 queries
@Entity
public class Order {
    @OneToMany(mappedBy = "order")
    private List<OrderItem> items; // Lazy loaded
}

List<Order> orders = orderRepository.findAll();
for (Order order : orders) {
    order.getItems().size(); // Each access triggers a query!
}

// GOOD - Fetch join
@Query("SELECT o FROM Order o LEFT JOIN FETCH o.items")
List<Order> findAllWithItems();

// Or use EntityGraph
@EntityGraph(attributePaths = {"items"})
List<Order> findAll();

// Or Projections (if you don't need full entities)
@Query("SELECT new com.example.OrderSummary(o.id, o.total, COUNT(i)) " +
       "FROM Order o LEFT JOIN o.items i GROUP BY o.id, o.total")
List<OrderSummary> findOrderSummaries();
```

---

### 2. Missing Database Indexes

**Symptom:** Slow queries on large tables.

```sql
-- Slow query (full table scan)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';
-- Seq Scan on users (cost=0.00..18334.00 rows=1)
```

**Solution:**

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email", unique = true),
    @Index(name = "idx_created_at", columnList = "created_at"),
    @Index(name = "idx_status_created", columnList = "status, created_at")
})
public class User {
    @Column(unique = true, nullable = false)
    private String email;

    private LocalDateTime createdAt;
    private String status;
}

// Or use Flyway migration
-- V2__add_user_indexes.sql
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_created_at ON users(created_at);
CREATE INDEX idx_status_created ON users(status, created_at);
```

---

### 3. Blocking I/O in Event Loop (WebFlux)

**Symptom:** Slow reactive application.

```java
// BAD - Blocking call in reactive chain
public Mono<User> getUser(Long id) {
    return Mono.fromCallable(() -> {
        // BLOCKING JDBC call in reactive context!
        return jdbcTemplate.queryForObject("SELECT * FROM users WHERE id = ?",
                new Object[]{id}, User.class);
    });
}
```

**Solution:**

```java
// GOOD - Use R2DBC (reactive database driver)
public Mono<User> getUser(Long id) {
    return userRepository.findById(id); // Non-blocking
}

// Or wrap blocking calls in bounded elastic scheduler
public Mono<User> getUser(Long id) {
    return Mono.fromCallable(() -> blockingRepository.findById(id))
            .subscribeOn(Schedulers.boundedElastic()); // Dedicated thread pool
}
```

---

## Security Issues

### 1. JWT Token Stolen/Replay Attack

**Scenario:** Attacker intercepts JWT and uses it to impersonate user.

**Solutions:**

```java
// 1. Short-lived access tokens + refresh tokens
@Service
public class TokenService {

    public String generateAccessToken(UserDetails user) {
        return Jwts.builder()
                .setSubject(user.getUsername())
                .setIssuedAt(new Date())
                .setExpiration(Date.from(Instant.now().plusSeconds(900))) // 15 min
                .signWith(key, SignatureAlgorithm.HS512)
                .compact();
    }

    public String generateRefreshToken(UserDetails user) {
        String token = Jwts.builder()
                .setSubject(user.getUsername())
                .setExpiration(Date.from(Instant.now().plusSeconds(604800))) // 7 days
                .signWith(key, SignatureAlgorithm.HS512)
                .compact();

        // Store refresh token in database for revocation
        refreshTokenRepository.save(new RefreshToken(token, user.getUsername()));
        return token;
    }
}

// 2. Token rotation (invalidate old refresh token on use)
public AuthResponse refreshAccessToken(String refreshToken) {
    RefreshToken storedToken = refreshTokenRepository.findByToken(refreshToken)
            .orElseThrow(() -> new InvalidTokenException("Invalid refresh token"));

    // Invalidate old refresh token
    refreshTokenRepository.delete(storedToken);

    // Issue new access token AND new refresh token
    String newAccessToken = generateAccessToken(storedToken.getUser());
    String newRefreshToken = generateRefreshToken(storedToken.getUser());

    return new AuthResponse(newAccessToken, newRefreshToken);
}

// 3. Device fingerprinting
public String generateToken(UserDetails user, HttpServletRequest request) {
    String deviceFingerprint = generateFingerprint(
            request.getHeader("User-Agent"),
            request.getRemoteAddr()
    );

    return Jwts.builder()
            .setSubject(user.getUsername())
            .claim("fingerprint", hashFingerprint(deviceFingerprint))
            .signWith(key)
            .compact();
}

public void validateToken(String token, HttpServletRequest request) {
    Claims claims = Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody();
    String storedFingerprint = claims.get("fingerprint", String.class);
    String currentFingerprint = generateFingerprint(
            request.getHeader("User-Agent"),
            request.getRemoteAddr()
    );

    if (!storedFingerprint.equals(hashFingerprint(currentFingerprint))) {
        throw new SecurityException("Token fingerprint mismatch");
    }
}
```

---

### 2. SQL Injection Despite Using JPA

**Vulnerable Code:**

```java
// VULNERABLE - String concatenation in JPQL
@Query("SELECT u FROM User u WHERE u.status = '" + status + "'")
List<User> findByStatus(String status); // SQL Injection risk!

// VULNERABLE - Native query with concatenation
@Query(value = "SELECT * FROM users WHERE email = '" + email + "'", nativeQuery = true)
User findByEmail(String email);
```

**Secure Code:**

```java
// SAFE - Parameterized query
@Query("SELECT u FROM User u WHERE u.status = :status")
List<User> findByStatus(@Param("status") String status);

// SAFE - Positional parameter
@Query("SELECT u FROM User u WHERE u.email = ?1")
User findByEmail(String email);

// SAFE - Criteria API
public List<User> findByStatus(String status) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> user = query.from(User.class);
    query.select(user).where(cb.equal(user.get("status"), status));
    return entityManager.createQuery(query).getResultList();
}
```

---

## Microservices Communication Failures

### 1. Service Discovery Failure

**Symptom:**

```
No instances available for user-service
```

**Diagnosis:**

```java
// Check Eureka dashboard: http://localhost:8761/

// Check service registration
@RestController
public class DiagnosticsController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/diagnostics/services")
    public Map<String, List<String>> getServices() {
        Map<String, List<String>> services = new HashMap<>();
        discoveryClient.getServices().forEach(serviceName -> {
            List<String> instances = discoveryClient.getInstances(serviceName)
                    .stream()
                    .map(si -> si.getUri().toString())
                    .collect(Collectors.toList());
            services.put(serviceName, instances);
        });
        return services;
    }
}
```

**Solutions:**

```yaml
# Ensure services are registering with Eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
    fetchRegistry: true
    registerWithEureka: true
  instance:
    preferIpAddress: true
    lease-renewal-interval-in-seconds: 5 # Heartbeat frequency
    lease-expiration-duration-in-seconds: 10 # Eviction time

# Health check endpoint
management:
  endpoints:
    web:
      exposure:
        include: health, info
  health:
    defaults:
      enabled: true
```

---

### 2. Circuit Breaker Not Opening

**Symptom:** Cascading failures, service keeps calling failing dependency.

**Diagnosis:**

```java
// Check circuit breaker state
@RestController
public class CircuitBreakerController {

    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;

    @GetMapping("/diagnostics/circuit-breakers")
    public Map<String, String> getCircuitBreakerStates() {
        return circuitBreakerRegistry.getAllCircuitBreakers()
                .stream()
                .collect(Collectors.toMap(
                        CircuitBreaker::getName,
                        cb -> cb.getState().toString()
                ));
    }
}
```

**Fix Configuration:**

```yaml
resilience4j:
  circuitbreaker:
    instances:
      userService:
        slidingWindowSize: 10 # Last 10 calls
        failureRateThreshold: 50 # 50% failures -> OPEN
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 3
        minimumNumberOfCalls: 5 # IMPORTANT: Need at least 5 calls
        recordExceptions:
          - java.net.ConnectException
          - java.util.concurrent.TimeoutException
        ignoreExceptions:
          - com.example.BusinessException # Don't count as failures
```

```java
@Service
public class OrderService {

    // Ensure @CircuitBreaker is applied
    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    public User getUser(Long userId) {
        return userClient.getUser(userId);
    }

    // Fallback must have SAME signature + Throwable parameter
    private User getUserFallback(Long userId, Throwable t) {
        log.error("Circuit breaker fallback for user {}: {}", userId, t.getMessage());
        return User.builder()
                .id(userId)
                .name("Unknown User")
                .build();
    }
}
```

---

## Transaction Management Issues

### 1. Transaction Not Rolling Back

**Symptom:** Data saved to database despite exception.

```java
// PROBLEM - Checked exception doesn't rollback by default
@Transactional
public void createOrder(OrderRequest request) throws OrderException {
    Order order = orderRepository.save(new Order(request));

    if (inventoryService.checkStock(order) < order.getQuantity()) {
        throw new OrderException("Insufficient stock"); // Checked exception - NO ROLLBACK!
    }
}
```

**Solutions:**

```java
// SOLUTION 1 - Use rollbackFor
@Transactional(rollbackFor = Exception.class)
public void createOrder(OrderRequest request) throws OrderException {
    // Now rolls back on checked exceptions
}

// SOLUTION 2 - Throw unchecked exception
@Transactional
public void createOrder(OrderRequest request) {
    Order order = orderRepository.save(new Order(request));

    if (inventoryService.checkStock(order) < order.getQuantity()) {
        throw new InsufficientStockException("Insufficient stock"); // RuntimeException
    }
}

// SOLUTION 3 - Manual rollback
@Transactional
public void createOrder(OrderRequest request) throws OrderException {
    try {
        // ... business logic
    } catch (Exception e) {
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
        throw e;
    }
}
```

---

### 2. Lost Updates (Concurrent Modifications)

**Scenario:** Two users update same entity concurrently, one update is lost.

```
Time    User A                          User B
T1      Read account (balance=100)
T2                                      Read account (balance=100)
T3      Debit 20 (balance=80)
T4                                      Debit 30 (balance=70)
T5      Save (balance=80)
T6                                      Save (balance=70)  <- Lost User A's update!
```

**Solution: Optimistic Locking**

```java
@Entity
public class Account {
    @Id
    private Long id;

    private BigDecimal balance;

    @Version  // Optimistic locking
    private Long version;
}

@Service
public class AccountService {

    @Transactional
    public void debit(Long accountId, BigDecimal amount) {
        Account account = accountRepository.findById(accountId)
                .orElseThrow(() -> new AccountNotFoundException(accountId));

        account.setBalance(account.getBalance().subtract(amount));

        accountRepository.save(account); // Throws OptimisticLockException if version changed
    }
}

// Handle concurrent modification
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OptimisticLockException.class)
    public ResponseEntity<ErrorResponse> handleOptimisticLock(OptimisticLockException e) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
                .body(new ErrorResponse("The resource was modified by another user. Please refresh and try again."));
    }
}
```

---

## Common Configuration Mistakes

### 1. Profiles Not Activated

**Symptom:** Application uses default config instead of prod config.

```bash
# Wrong - profile not activated
java -jar app.jar

# Correct
java -jar app.jar --spring.profiles.active=prod

# Or environment variable
export SPRING_PROFILES_ACTIVE=prod
java -jar app.jar

# Or in application.yml
spring:
  profiles:
    active: prod
```

---

### 2. Property Not Overriding

**Issue:** External config not overriding packaged config.

**Property Precedence (highest to lowest):**

1. Command line arguments: `--server.port=9090`
2. Java system properties: `-Dserver.port=9090`
3. OS environment variables: `SERVER_PORT=9090`
4. `application-{profile}.yml` in `/config` subdirectory
5. `application-{profile}.yml` in current directory
6. `application-{profile}.yml` in classpath `/config` package
7. `application-{profile}.yml` in classpath root
8. `application.yml` (same locations as above)

```bash
# Override from external config file
java -jar app.jar --spring.config.location=file:./config/application-prod.yml

# Or environment variable (note naming convention)
export SERVER_PORT=9090  # Maps to server.port
export SPRING_DATASOURCE_URL=jdbc:postgresql://prod-db:5432/mydb
```

---

### 3. Binding Null Values

**Symptom:**

```
Binding to target org.springframework.boot.context.properties.bind.BindException:
Failed to bind properties under 'app.feature-flags' to FeatureFlags
```

**Fix:**

```java
@ConfigurationProperties(prefix = "app.feature-flags")
public class FeatureFlags {
    private boolean enableNewUI = false;  // Default value
    private boolean enableBetaFeatures = false;

    // Getters/setters
}

// Or use @DefaultValue
@ConfigurationProperties(prefix = "app")
public class AppConfig {
    @DefaultValue("false")
    private boolean debugMode;
}
```

---

## Debugging Techniques

### 1. Actuator Endpoints

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*" # Expose all endpoints (secure in production!)
  endpoint:
    health:
      show-details: always
```

**Useful endpoints:**

- `/actuator/health` - Health status
- `/actuator/metrics` - Application metrics
- `/actuator/env` - Environment properties
- `/actuator/beans` - All Spring beans
- `/actuator/mappings` - All HTTP endpoints
- `/actuator/loggers` - Change log levels at runtime

```bash
# Change log level at runtime
curl -X POST http://localhost:8080/actuator/loggers/com.example.myapp \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}'
```

---

### 2. Request/Response Logging

```java
@Component
public class LoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        // Wrap to allow reading body multiple times
        ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper(httpRequest);
        ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper(httpResponse);

        long startTime = System.currentTimeMillis();

        try {
            chain.doFilter(requestWrapper, responseWrapper);
        } finally {
            long duration = System.currentTimeMillis() - startTime;

            log.info("Request: {} {} - Status: {} - Duration: {}ms",
                    httpRequest.getMethod(),
                    httpRequest.getRequestURI(),
                    httpResponse.getStatus(),
                    duration);

            if (log.isDebugEnabled()) {
                log.debug("Request body: {}", new String(requestWrapper.getContentAsByteArray()));
                log.debug("Response body: {}", new String(responseWrapper.getContentAsByteArray()));
            }

            responseWrapper.copyBodyToResponse(); // Important: Copy body to actual response
        }
    }
}
```

---

### 3. Thread Dump Analysis

```bash
# Generate thread dump
jstack <PID> > threaddump.txt

# Or via Actuator
curl http://localhost:8080/actuator/threaddump > threaddump.json
```

**Analyze deadlocks:**

```
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f8a3c004e80 (object 0x00000000d5f3a0a0, a java.lang.Object),
  which is held by "Thread-2"
"Thread-2":
  waiting to lock monitor 0x00007f8a3c007330 (object 0x00000000d5f3a0b0, a java.lang.Object),
  which is held by "Thread-1"
```

---

## Production Incidents & Resolution

### Incident 1: Sudden Traffic Spike → 503 Service Unavailable

**Symptoms:**

- Response time increased from 200ms to 5s
- Error rate spiked to 30%
- CPU at 90%, memory stable

**Investigation:**

```bash
# Check active requests
curl http://localhost:8080/actuator/metrics/http.server.requests.active

# Check thread pool
curl http://localhost:8080/actuator/metrics/executor.active

# Check circuit breaker state
curl http://localhost:8080/actuator/circuitbreakers
```

**Root Cause:** Tomcat thread pool exhausted (default 200 threads).

**Immediate Fix:**

```yaml
server:
  tomcat:
    threads:
      max: 400 # Increase from 200
      min-spare: 50
    max-connections: 10000
    accept-count: 200 # Queue size for requests when all threads busy
```

**Long-term Fix:**

- Implement rate limiting
- Add caching for frequently accessed data
- Scale horizontally (add more instances)

---

### Incident 2: Memory Leak → OOM After 3 Days

**Symptoms:**

- Heap usage grows linearly over time
- Frequent full GCs, long GC pauses
- Eventually crashes with OOM

**Investigation:**

```bash
# Enable GC logging
java -Xlog:gc*:file=/tmp/gc.log -XX:+HeapDumpOnOutOfMemoryError -jar app.jar

# Analyze heap dump with VisualVM or MAT
```

**Root Cause:** ThreadLocal not cleaned up in async executor.

```java
// LEAK - ThreadLocal in controller
@RestController
public class UserController {

    private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        User user = userService.getUser(id);
        currentUser.set(user); // Never removed!
        return user;
    }
}

// FIX - Always clean up ThreadLocal
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    try {
        User user = userService.getUser(id);
        currentUser.set(user);
        return user;
    } finally {
        currentUser.remove(); // Clean up
    }
}

// BETTER - Avoid ThreadLocal, use request scope
@RestController
public class UserController {
    @Autowired
    private HttpServletRequest request;

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        User user = userService.getUser(id);
        request.setAttribute("currentUser", user);
        return user;
    }
}
```

---

### Incident 3: Database Connection Leak

**Symptoms:**

- Intermittent "Connection timeout" errors
- Pool exhaustion after few hours
- Errors during peak hours

**Investigation:**

```java
// Enable HikariCP leak detection
spring.datasource.hikari.leak-detection-threshold=60000

// Check pool metrics
curl http://localhost:8080/actuator/metrics/hikaricp.connections.active
curl http://localhost:8080/actuator/metrics/hikaricp.connections.idle
```

**Logs showed:**

```
Connection leak detection triggered for connection com.zaxxer.hikari.proxy.HikariProxyConnection@abc123
Apparently, the connection was used at this location:
  at com.example.UserService.getUsers(UserService.java:45)
```

**Root Cause:** Using `EntityManager` directly without closing.

```java
// LEAK
@Service
public class UserService {
    @PersistenceContext
    private EntityManager entityManager;

    public List<User> getActiveUsers() {
        Query query = entityManager.createQuery("SELECT u FROM User u WHERE u.active = true");
        return query.getResultList(); // EntityManager not closed
    }
}

// FIX - Use repository (auto-managed)
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public List<User> getActiveUsers() {
        return userRepository.findByActiveTrue(); // Spring Data manages lifecycle
    }
}
```

---

## Interview Questions & Answers

### Q1: How do you troubleshoot a slow-performing endpoint?

**Answer:**

**Step-by-step approach:**

1. **Identify the bottleneck:**

```java
@RestController
public class OrderController {

    @GetMapping("/orders/{id}")
    public Order getOrder(@PathVariable Long id) {
        long start = System.currentTimeMillis();

        Order order = orderService.getOrder(id);
        log.info("Service call: {}ms", System.currentTimeMillis() - start);

        orderService.enrichWithUserDetails(order);
        log.info("Enrichment: {}ms", System.currentTimeMillis() - start);

        return order;
    }
}
```

2. **Check database queries:**

```yaml
spring:
  jpa:
    show-sql: true # Log SQL
    properties:
      hibernate:
        format_sql: true
        use_sql_comments: true

logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE # Log parameters
```

3. **Profile with APM tools:**

- New Relic, Datadog, AppDynamics
- Identify slow database queries, external API calls

4. **Common fixes:**

- Add database indexes
- Fix N+1 queries (use JOIN FETCH)
- Add caching
- Optimize algorithms (O(n²) → O(n))
- Use async processing for non-critical tasks

---

### Q2: Application fails to start. How do you diagnose?

**Answer:**

**Diagnosis steps:**

1. **Check stack trace:**

```
***************************
APPLICATION FAILED TO START
***************************

Description:
Field userRepository in UserService required a bean of type 'UserRepository'
that could not be found.
```

2. **Enable debug logging:**

```bash
java -jar app.jar --debug
# Or
logging.level.org.springframework=DEBUG
```

3. **Common startup issues:**

| Error                      | Cause                          | Fix                                                   |
| -------------------------- | ------------------------------ | ----------------------------------------------------- |
| `Port 8080 already in use` | Another process using port     | Kill process or change port                           |
| `Bean not found`           | Component not scanned          | Add `@EnableJpaRepositories` or fix package structure |
| `Circular dependency`      | ServiceA → ServiceB → ServiceA | Use `@Lazy` or refactor                               |
| `Unable to create bean`    | Constructor injection failure  | Check dependencies                                    |
| `Unsatisfied dependency`   | Missing required property      | Add to `application.yml`                              |

4. **Verify configuration:**

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(Application.class, args);

        // Print all beans (debug)
        String[] beanNames = ctx.getBeanDefinitionNames();
        Arrays.sort(beanNames);
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
    }
}
```

---

### Q3: How do you debug intermittent failures in production?

**Answer:**

**Challenges:** Hard to reproduce, no stack traces, happens randomly.

**Strategies:**

1. **Structured logging with correlation ID:**

```java
@Component
public class CorrelationIdFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        String correlationId = UUID.randomUUID().toString();
        MDC.put("correlationId", correlationId);
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.remove("correlationId");
        }
    }
}

// Logback configuration
<pattern>%d{ISO8601} [%thread] %-5level %logger{36} [%X{correlationId}] - %msg%n</pattern>

// All logs for a single request will have same correlationId
2024-01-15 10:23:45 [http-nio-8080-exec-1] INFO  c.e.OrderController [abc-123] - Received order request
2024-01-15 10:23:46 [http-nio-8080-exec-1] ERROR c.e.PaymentService [abc-123] - Payment failed
```

2. **Detailed error tracking:**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex, HttpServletRequest request) {
        String errorId = UUID.randomUUID().toString();

        // Log full details
        log.error("Error ID: {} | URL: {} | User: {} | Details: {}",
                errorId, request.getRequestURI(), getCurrentUser(), ex);

        // Return error ID to user
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(new ErrorResponse("An error occurred. Error ID: " + errorId));
    }
}
```

3. **Application Performance Monitoring (APM):**

- Datadog, New Relic, AppDynamics
- Distributed tracing (Zipkin, Jaeger)
- Captures stack traces, slow queries, errors

4. **Feature flags for risky changes:**

```java
@Service
public class OrderService {
    @Autowired
    private FeatureFlagService featureFlags;

    public Order createOrder(OrderRequest request) {
        if (featureFlags.isEnabled("new-payment-flow")) {
            return createOrderV2(request); // New code path
        } else {
            return createOrderV1(request); // Old code path
        }
    }
}
```

---

### Q4: How do you handle database deadlocks?

**Answer:**

**Deadlock scenario:**

```
Transaction 1: Lock Row A → Wait for Row B
Transaction 2: Lock Row B → Wait for Row A
Result: DEADLOCK!
```

**Detection:**

```
ERROR: deadlock detected
Detail: Process 123 waits for ShareLock on transaction 456
Hint: See server log for query details
```

**Prevention strategies:**

1. **Consistent lock ordering:**

```java
// BAD - Inconsistent lock order (can deadlock)
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {
    Account from = accountRepository.findByIdWithLock(fromId); // Lock account 100
    Account to = accountRepository.findByIdWithLock(toId); // Lock account 50
    // ...
}

// Thread 1: transfer(100, 50) → Locks 100, waits for 50
// Thread 2: transfer(50, 100) → Locks 50, waits for 100
// DEADLOCK!

// GOOD - Always lock in ascending ID order
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {
    Long firstId = Math.min(fromId, toId);
    Long secondId = Math.max(fromId, toId);

    Account first = accountRepository.findByIdWithLock(firstId);
    Account second = accountRepository.findByIdWithLock(secondId);

    // Perform transfer logic
}
```

2. **Use optimistic locking instead of pessimistic:**

```java
@Entity
public class Account {
    @Version
    private Long version; // Optimistic locking (no database locks)
}
```

3. **Reduce transaction scope:**

```java
// BAD - Long transaction holding locks
@Transactional
public void processOrder(OrderRequest request) {
    Order order = createOrder(request); // Locks order table
    sendEmail(order); // External call - holding locks!
    updateInventory(order); // More locks
}

// GOOD - Minimize transaction scope
public void processOrder(OrderRequest request) {
    Order order = createOrderTransactional(request); // Short transaction
    sendEmail(order); // Outside transaction
    updateInventoryTransactional(order); // Separate transaction
}

@Transactional
private Order createOrderTransactional(OrderRequest request) {
    return orderRepository.save(new Order(request));
}
```

4. **Retry logic:**

```java
@Retryable(
    value = DeadlockLoserDataAccessException.class,
    maxAttempts = 3,
    backoff = @Backoff(delay = 100)
)
@Transactional
public void processPayment(Long orderId) {
    // Retry on deadlock
}
```

---

### Q5: Memory keeps growing. How do you find the leak?

**Answer:**

**Step-by-step investigation:**

1. **Enable heap dump on OOM:**

```bash
java -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/tmp/heapdump.hprof \
     -Xms512m -Xmx1g \
     -jar app.jar
```

2. **Take periodic heap dumps:**

```bash
# Take heap dump
jmap -dump:live,format=b,file=heapdump-$(date +%s).hprof <PID>

# Or via Actuator
curl -X POST http://localhost:8080/actuator/heapdump -o heapdump.hprof
```

3. **Analyze with VisualVM or Eclipse MAT:**

- Open heap dump in MAT
- Look for **Leak Suspects Report**
- Find objects consuming most memory
- Check **Dominator Tree** to see what's holding references

4. **Common leak patterns:**

```java
// LEAK 1 - Static collection
public class UserCache {
    private static final Map<Long, User> CACHE = new HashMap<>();

    public static void cache(User user) {
        CACHE.put(user.getId(), user); // Never evicted!
    }
}

// FIX - Bounded cache
@Cacheable(value = "users", unless = "#result == null")
public User getUser(Long id) {
    return userRepository.findById(id).orElse(null);
}

// LEAK 2 - ThreadLocal not cleaned
private static final ThreadLocal<List<String>> CONTEXT = new ThreadLocal<>();

// FIX
try {
    CONTEXT.set(new ArrayList<>());
    // ... use context
} finally {
    CONTEXT.remove(); // Always clean up
}

// LEAK 3 - Event listeners never removed
applicationEventMulticaster.addListener(event -> {
    // Listener holds reference to controller (never removed)
});

// FIX - Use weak references or remove listeners
```

5. **Monitor in production:**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: metrics, heapdump
  metrics:
    enable:
      jvm: true

# Track metrics
curl http://localhost:8080/actuator/metrics/jvm.memory.used
curl http://localhost:8080/actuator/metrics/jvm.gc.pause
```

---

## Summary: Production-Ready Checklist

✅ **Logging & Monitoring**

- Structured logging with correlation IDs
- Centralized log aggregation (ELK, Splunk)
- APM tools (Datadog, New Relic)
- Distributed tracing (Zipkin, Jaeger)

✅ **Error Handling**

- Global exception handler
- Custom error responses
- Error tracking (Sentry, Rollbar)
- Circuit breakers for external services

✅ **Performance**

- Database connection pooling tuned
- Caching strategy implemented
- Database indexes optimized
- N+1 queries eliminated

✅ **Security**

- Input validation
- SQL injection prevention
- XSS/CSRF protection
- Secrets externalized (not in code)

✅ **Resilience**

- Health checks configured
- Graceful shutdown enabled
- Circuit breakers configured
- Retry logic with exponential backoff

✅ **Observability**

- Actuator endpoints enabled (secured)
- Custom metrics exposed
- Thread dumps accessible
- Heap dumps on OOM

---

**Key Takeaway:** Most production issues can be **prevented** with proper configuration, testing, and observability. When they occur, **structured logging** and **APM tools** are your best friends for rapid diagnosis and resolution.
