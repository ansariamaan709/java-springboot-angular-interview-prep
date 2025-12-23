# Spring Boot Complete Interview Guide

## Table of Contents

1. [Spring Core Concepts](#spring-core-concepts)
2. [Spring Boot Fundamentals](#spring-boot-fundamentals)
3. [Dependency Injection & IoC](#dependency-injection--ioc)
4. [Spring Boot Auto-Configuration](#spring-boot-auto-configuration)
5. [Spring MVC & REST APIs](#spring-mvc--rest-apis)
6. [Spring Data JPA](#spring-data-jpa)
7. [Spring Security](#spring-security)
8. [Microservices with Spring Boot](#microservices-with-spring-boot)
9. [Testing in Spring Boot](#testing-in-spring-boot)
10. [Interview Questions](#interview-questions)

---

## Spring Core Concepts

### What is Spring Framework?

Spring is a comprehensive framework for enterprise Java development that provides:

- **Dependency Injection (DI)** - Manages object creation and wiring
- **Aspect-Oriented Programming (AOP)** - Cross-cutting concerns
- **Transaction Management** - Declarative transactions
- **MVC Framework** - Web application development
- **Data Access** - JDBC, ORM integration

### Spring Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SPRING FRAMEWORK                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                        WEB LAYER                                │ │
│  │  Spring MVC │ Spring WebFlux │ WebSocket │ Servlet              │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                    DATA ACCESS LAYER                            │ │
│  │  JDBC │ ORM │ Transactions │ JPA │ Spring Data                  │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                     CORE CONTAINER                              │ │
│  │  Beans │ Core │ Context │ Expression Language (SpEL)            │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                     CROSS-CUTTING                               │ │
│  │  AOP │ Aspects │ Instrumentation │ Messaging                    │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                          TEST                                   │ │
│  │  Unit Testing │ Integration Testing │ Mock Objects              │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Spring Boot Fundamentals

### What is Spring Boot?

Spring Boot is an opinionated framework built on top of Spring that:

- Provides **auto-configuration** for common scenarios
- Eliminates boilerplate XML configuration
- Offers **embedded servers** (Tomcat, Jetty, Undertow)
- Supports **production-ready** features (metrics, health checks)
- Uses **convention over configuration**

### Spring vs Spring Boot Comparison

| Feature           | Spring Framework  | Spring Boot                       |
| ----------------- | ----------------- | --------------------------------- |
| **Configuration** | Manual (XML/Java) | Auto-configuration                |
| **Setup**         | Complex           | Simple (starters)                 |
| **Server**        | External required | Embedded                          |
| **Dependencies**  | Manual management | Starter POMs                      |
| **Main Method**   | Optional          | Required (@SpringBootApplication) |
| **Production**    | Manual setup      | Actuator built-in                 |

### Creating a Spring Boot Application

```java
// Main Application Class
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// @SpringBootApplication is equivalent to:
@Configuration           // Java-based configuration
@EnableAutoConfiguration // Enable auto-config
@ComponentScan          // Scan for components
```

### Project Structure

```
my-spring-boot-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/
│   │   │       ├── MyApplication.java
│   │   │       ├── controller/
│   │   │       │   └── UserController.java
│   │   │       ├── service/
│   │   │       │   ├── UserService.java
│   │   │       │   └── impl/
│   │   │       │       └── UserServiceImpl.java
│   │   │       ├── repository/
│   │   │       │   └── UserRepository.java
│   │   │       ├── model/
│   │   │       │   └── User.java
│   │   │       ├── dto/
│   │   │       │   └── UserDTO.java
│   │   │       ├── exception/
│   │   │       │   └── UserNotFoundException.java
│   │   │       └── config/
│   │   │           └── AppConfig.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── static/
│   └── test/
│       └── java/
│           └── com/example/
│               └── controller/
│                   └── UserControllerTest.java
├── pom.xml
└── README.md
```

### application.yml Configuration

```yaml
# Application settings
spring:
  application:
    name: my-service

  # Database configuration
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER:postgres}
    password: ${DB_PASS:password}
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      idle-timeout: 300000
      connection-timeout: 20000

  # JPA settings
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect

  # Profile activation
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}

# Server configuration
server:
  port: 8080
  servlet:
    context-path: /api
  error:
    include-message: always
    include-binding-errors: always

# Logging
logging:
  level:
    root: INFO
    com.example: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"

# Custom properties
app:
  jwt:
    secret: ${JWT_SECRET:mySecretKey}
    expiration: 86400000
  cors:
    allowed-origins: http://localhost:3000
```

---

## Dependency Injection & IoC

### What is Inversion of Control (IoC)?

IoC is a design principle where the control of object creation and lifecycle is transferred from the application code to a framework (Spring Container).

```
Traditional Control:
┌───────────────┐      creates      ┌──────────────┐
│  Application  │ ───────────────► │  Dependencies │
└───────────────┘                   └──────────────┘

Inversion of Control:
┌───────────────┐      injects     ┌───────────────┐
│   Container   │ ───────────────► │  Application  │
│  (Spring)     │                  │               │
└───────────────┘                  └───────────────┘
       │
       │ manages
       ▼
┌──────────────┐
│ Dependencies │
└──────────────┘
```

### Types of Dependency Injection

#### 1. Constructor Injection (Recommended)

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final NotificationService notificationService;

    // Constructor injection - @Autowired optional for single constructor
    public OrderService(OrderRepository orderRepository,
                       PaymentService paymentService,
                       NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }

    public Order placeOrder(OrderRequest request) {
        Order order = new Order(request);
        paymentService.processPayment(order);
        orderRepository.save(order);
        notificationService.sendConfirmation(order);
        return order;
    }
}
```

**Benefits of Constructor Injection:**

- Immutable dependencies (final fields)
- Required dependencies guaranteed
- Easy to test (mock in constructor)
- Clear dependency list
- Fails fast if dependency missing

#### 2. Setter Injection

```java
@Service
public class ReportService {

    private ReportRepository reportRepository;
    private EmailService emailService;

    @Autowired
    public void setReportRepository(ReportRepository reportRepository) {
        this.reportRepository = reportRepository;
    }

    @Autowired(required = false)  // Optional dependency
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

#### 3. Field Injection (Not Recommended)

```java
@Service
public class UserService {

    @Autowired  // Not recommended - hard to test
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;
}
```

### Bean Scopes

```java
@Component
@Scope("singleton")  // Default - one instance per container
public class SingletonBean { }

@Component
@Scope("prototype")  // New instance for each request
public class PrototypeBean { }

@Component
@Scope("request")    // One instance per HTTP request
public class RequestBean { }

@Component
@Scope("session")    // One instance per HTTP session
public class SessionBean { }

@Component
@Scope("application") // One instance per ServletContext
public class ApplicationBean { }
```

### Bean Scope Comparison

| Scope           | Instances            | Lifecycle                  | Use Case           |
| --------------- | -------------------- | -------------------------- | ------------------ |
| **singleton**   | 1 per container      | Container lifetime         | Stateless services |
| **prototype**   | New each time        | Not managed after creation | Stateful beans     |
| **request**     | 1 per HTTP request   | Request duration           | Request data       |
| **session**     | 1 per HTTP session   | Session duration           | User-specific data |
| **application** | 1 per ServletContext | Application lifetime       | Shared state       |

### @Qualifier and @Primary

```java
// Multiple implementations
public interface MessageService {
    void sendMessage(String message);
}

@Service
@Primary  // Default choice when no qualifier specified
public class EmailService implements MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("Email: " + message);
    }
}

@Service
@Qualifier("sms")
public class SMSService implements MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("SMS: " + message);
    }
}

@Service
public class NotificationService {

    private final MessageService emailService;
    private final MessageService smsService;

    public NotificationService(
            MessageService emailService,  // Gets @Primary
            @Qualifier("sms") MessageService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }
}
```

---

## Spring Boot Auto-Configuration

### How Auto-Configuration Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AUTO-CONFIGURATION PROCESS                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Application starts with @EnableAutoConfiguration                 │
│                            │                                         │
│                            ▼                                         │
│  2. Spring scans META-INF/spring/                                   │
│     org.springframework.boot.autoconfigure.AutoConfiguration.imports│
│                            │                                         │
│                            ▼                                         │
│  3. Evaluates @Conditional annotations                              │
│     - @ConditionalOnClass                                           │
│     - @ConditionalOnMissingBean                                     │
│     - @ConditionalOnProperty                                        │
│                            │                                         │
│                            ▼                                         │
│  4. Creates beans that pass conditions                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Common Conditional Annotations

```java
@Configuration
@ConditionalOnClass(DataSource.class)  // Only if class is on classpath
@ConditionalOnProperty(
    prefix = "spring.datasource",
    name = "enabled",
    havingValue = "true",
    matchIfMissing = true
)
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean  // Only if user hasn't defined this bean
    public DataSource dataSource(DataSourceProperties properties) {
        return DataSourceBuilder.create()
            .url(properties.getUrl())
            .username(properties.getUsername())
            .password(properties.getPassword())
            .build();
    }
}
```

### Creating Custom Auto-Configuration

```java
// 1. Create configuration class
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyServiceProperties.class)
public class MyServiceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyServiceProperties properties) {
        return new MyService(properties.getApiKey());
    }
}

// 2. Create properties class
@ConfigurationProperties(prefix = "myservice")
public class MyServiceProperties {
    private String apiKey;
    private int timeout = 30;

    // Getters and setters
}

// 3. Register in META-INF/spring/
// org.springframework.boot.autoconfigure.AutoConfiguration.imports
// com.example.MyServiceAutoConfiguration

// 4. Usage in application.yml
// myservice:
//   api-key: my-secret-key
//   timeout: 60
```

### Excluding Auto-Configuration

```java
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    SecurityAutoConfiguration.class
})
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// Or in application.yml
// spring:
//   autoconfigure:
//     exclude:
//       - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

---

## Spring MVC & REST APIs

### REST Controller Basics

```java
@RestController
@RequestMapping("/api/v1/users")
@Validated
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    // GET /api/v1/users
    @GetMapping
    public ResponseEntity<List<UserDTO>> getAllUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy) {

        Page<UserDTO> users = userService.getAllUsers(page, size, sortBy);
        return ResponseEntity.ok(users.getContent());
    }

    // GET /api/v1/users/{id}
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUserById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    // POST /api/v1/users
    @PostMapping
    public ResponseEntity<UserDTO> createUser(
            @Valid @RequestBody CreateUserRequest request) {

        UserDTO created = userService.createUser(request);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();

        return ResponseEntity.created(location).body(created);
    }

    // PUT /api/v1/users/{id}
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserRequest request) {

        return ResponseEntity.ok(userService.updateUser(id, request));
    }

    // PATCH /api/v1/users/{id}
    @PatchMapping("/{id}")
    public ResponseEntity<UserDTO> partialUpdateUser(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {

        return ResponseEntity.ok(userService.partialUpdate(id, updates));
    }

    // DELETE /api/v1/users/{id}
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Request/Response DTOs with Validation

```java
// Request DTO
public class CreateUserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).*$",
        message = "Password must contain uppercase, lowercase, and digit"
    )
    private String password;

    @NotNull(message = "Age is required")
    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 150, message = "Age must be less than 150")
    private Integer age;

    // Getters and setters
}

// Response DTO
@Data
@Builder
public class UserDTO {
    private Long id;
    private String name;
    private String email;
    private Integer age;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

### Global Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.NOT_FOUND.value())
            .error("Not Found")
            .message(ex.getMessage())
            .path(request.getRequestURI())
            .build();

        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(
            MethodArgumentNotValidException ex, HttpServletRequest request) {

        Map<String, String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage,
                (existing, replacement) -> existing
            ));

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.BAD_REQUEST.value())
            .error("Validation Failed")
            .message("Invalid request parameters")
            .path(request.getRequestURI())
            .validationErrors(errors)
            .build();

        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ErrorResponse> handleDataIntegrityViolation(
            DataIntegrityViolationException ex, HttpServletRequest request) {

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.CONFLICT.value())
            .error("Data Integrity Violation")
            .message("Database constraint violation")
            .path(request.getRequestURI())
            .build();

        return new ResponseEntity<>(error, HttpStatus.CONFLICT);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAllExceptions(
            Exception ex, HttpServletRequest request) {

        log.error("Unhandled exception", ex);

        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
            .error("Internal Server Error")
            .message("An unexpected error occurred")
            .path(request.getRequestURI())
            .build();

        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}

@Data
@Builder
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private Map<String, String> validationErrors;
}
```

### CORS Configuration

```java
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("http://localhost:3000", "https://myapp.com")
                    .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
                    .allowedHeaders("*")
                    .exposedHeaders("Authorization", "Content-Type")
                    .allowCredentials(true)
                    .maxAge(3600);
            }
        };
    }
}
```

---

## Spring Data JPA

### Entity Mapping

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email", unique = true)
})
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserRole role = UserRole.USER;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<Order> orders = new ArrayList<>();

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();

    @CreationTimestamp
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;

    @Version  // Optimistic locking
    private Long version;

    // Helper methods for bidirectional relationships
    public void addOrder(Order order) {
        orders.add(order);
        order.setUser(this);
    }

    public void removeOrder(Order order) {
        orders.remove(order);
        order.setUser(null);
    }
}
```

### Repository Interface

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Derived query methods
    Optional<User> findByEmail(String email);

    List<User> findByNameContainingIgnoreCase(String name);

    List<User> findByRoleOrderByCreatedAtDesc(UserRole role);

    boolean existsByEmail(String email);

    // Custom JPQL query
    @Query("SELECT u FROM User u WHERE u.createdAt >= :startDate")
    List<User> findUsersCreatedAfter(@Param("startDate") LocalDateTime startDate);

    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE email LIKE %:domain%", nativeQuery = true)
    List<User> findByEmailDomain(@Param("domain") String domain);

    // Modifying query
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.role = :role WHERE u.id = :id")
    int updateUserRole(@Param("id") Long id, @Param("role") UserRole role);

    // Projection
    @Query("SELECT u.email FROM User u WHERE u.role = :role")
    List<String> findEmailsByRole(@Param("role") UserRole role);

    // Pagination and sorting
    Page<User> findByRole(UserRole role, Pageable pageable);

    // EntityGraph for eager loading
    @EntityGraph(attributePaths = {"orders", "roles"})
    Optional<User> findWithOrdersAndRolesById(Long id);
}
```

### Service Layer with Transactions

```java
@Service
@Transactional(readOnly = true)
public class UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final UserMapper userMapper;

    public UserService(UserRepository userRepository,
                      PasswordEncoder passwordEncoder,
                      UserMapper userMapper) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.userMapper = userMapper;
    }

    public UserDTO getUserById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
        return userMapper.toDTO(user);
    }

    public Page<UserDTO> getAllUsers(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return userRepository.findAll(pageable).map(userMapper::toDTO);
    }

    @Transactional  // Override readOnly for write operation
    public UserDTO createUser(CreateUserRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new DuplicateResourceException("Email already exists");
        }

        User user = User.builder()
            .name(request.getName())
            .email(request.getEmail())
            .password(passwordEncoder.encode(request.getPassword()))
            .role(UserRole.USER)
            .build();

        User saved = userRepository.save(user);
        return userMapper.toDTO(saved);
    }

    @Transactional
    public UserDTO updateUser(Long id, UpdateUserRequest request) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));

        user.setName(request.getName());
        if (request.getEmail() != null && !request.getEmail().equals(user.getEmail())) {
            if (userRepository.existsByEmail(request.getEmail())) {
                throw new DuplicateResourceException("Email already exists");
            }
            user.setEmail(request.getEmail());
        }

        return userMapper.toDTO(user);  // Dirty checking will save automatically
    }

    @Transactional
    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User not found: " + id);
        }
        userRepository.deleteById(id);
    }
}
```

### Transaction Propagation

```java
@Service
public class OrderService {

    // REQUIRED (default): Join existing or create new
    @Transactional(propagation = Propagation.REQUIRED)
    public void createOrder() { }

    // REQUIRES_NEW: Always create new, suspend existing
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void auditLog() { }

    // NESTED: Create savepoint within existing transaction
    @Transactional(propagation = Propagation.NESTED)
    public void updateInventory() { }

    // SUPPORTS: Use existing if present, else non-transactional
    @Transactional(propagation = Propagation.SUPPORTS)
    public void readData() { }

    // MANDATORY: Must have existing transaction
    @Transactional(propagation = Propagation.MANDATORY)
    public void requiresTransaction() { }

    // NEVER: Must NOT have existing transaction
    @Transactional(propagation = Propagation.NEVER)
    public void nonTransactional() { }

    // NOT_SUPPORTED: Suspend existing transaction
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void suspendTransaction() { }
}
```

---

## Spring Security

### Security Configuration (Spring Security 6+)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final UserDetailsService userDetailsService;

    public SecurityConfig(JwtAuthenticationFilter jwtAuthFilter,
                         UserDetailsService userDetailsService) {
        this.jwtAuthFilter = jwtAuthFilter;
        this.userDetailsService = userDetailsService;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/users/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("http://localhost:3000"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

### JWT Authentication Filter

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    public JwtAuthenticationFilter(JwtService jwtService,
                                  UserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain filterChain)
            throws ServletException, IOException {

        final String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        try {
            final String jwt = authHeader.substring(7);
            final String username = jwtService.extractUsername(jwt);

            if (username != null &&
                SecurityContextHolder.getContext().getAuthentication() == null) {

                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                if (jwtService.isTokenValid(jwt, userDetails)) {
                    UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                            userDetails,
                            null,
                            userDetails.getAuthorities()
                        );

                    authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                    );

                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (Exception e) {
            // Log and continue without authentication
        }

        filterChain.doFilter(request, response);
    }
}
```

### JWT Service

```java
@Service
public class JwtService {

    @Value("${app.jwt.secret}")
    private String secretKey;

    @Value("${app.jwt.expiration}")
    private long jwtExpiration;

    public String generateToken(UserDetails userDetails) {
        return generateToken(new HashMap<>(), userDetails);
    }

    public String generateToken(Map<String, Object> extraClaims,
                               UserDetails userDetails) {
        return Jwts.builder()
            .setClaims(extraClaims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }

    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

---

## Microservices with Spring Boot

### Service Discovery with Eureka

```java
// Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

// application.yml for Eureka Server
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
```

```java
// Eureka Client (Microservice)
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// application.yml for Eureka Client
spring:
  application:
    name: user-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    prefer-ip-address: true
```

### API Gateway (Spring Cloud Gateway)

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

```yaml
# application.yml for API Gateway
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: userServiceCB
                fallbackUri: forward:/fallback/users

        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1

      default-filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
```

### Circuit Breaker with Resilience4j

```java
@Service
public class OrderService {

    private final UserServiceClient userServiceClient;

    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    @Retry(name = "userService")
    @TimeLimiter(name = "userService")
    public CompletableFuture<UserDTO> getUser(Long userId) {
        return CompletableFuture.supplyAsync(() ->
            userServiceClient.getUserById(userId));
    }

    public CompletableFuture<UserDTO> getUserFallback(Long userId, Exception e) {
        return CompletableFuture.completedFuture(
            UserDTO.builder()
                .id(userId)
                .name("Unknown User")
                .build()
        );
    }
}
```

```yaml
# Resilience4j configuration
resilience4j:
  circuitbreaker:
    instances:
      userService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        permitted-number-of-calls-in-half-open-state: 3

  retry:
    instances:
      userService:
        max-attempts: 3
        wait-duration: 500ms
        exponential-backoff-multiplier: 2

  timelimiter:
    instances:
      userService:
        timeout-duration: 3s
```

### Inter-Service Communication with OpenFeign

```java
@FeignClient(
    name = "user-service",
    fallbackFactory = UserServiceFallbackFactory.class
)
public interface UserServiceClient {

    @GetMapping("/api/users/{id}")
    UserDTO getUserById(@PathVariable Long id);

    @GetMapping("/api/users")
    List<UserDTO> getAllUsers();

    @PostMapping("/api/users")
    UserDTO createUser(@RequestBody CreateUserRequest request);
}

@Component
public class UserServiceFallbackFactory implements FallbackFactory<UserServiceClient> {

    @Override
    public UserServiceClient create(Throwable cause) {
        return new UserServiceClient() {
            @Override
            public UserDTO getUserById(Long id) {
                return UserDTO.builder()
                    .id(id)
                    .name("Fallback User")
                    .build();
            }

            @Override
            public List<UserDTO> getAllUsers() {
                return Collections.emptyList();
            }

            @Override
            public UserDTO createUser(CreateUserRequest request) {
                throw new ServiceUnavailableException("User service unavailable");
            }
        };
    }
}
```

---

## Testing in Spring Boot

### Unit Testing with Mockito

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @Mock
    private UserMapper userMapper;

    @InjectMocks
    private UserService userService;

    private User testUser;
    private UserDTO testUserDTO;

    @BeforeEach
    void setUp() {
        testUser = User.builder()
            .id(1L)
            .name("John Doe")
            .email("john@example.com")
            .password("encoded-password")
            .build();

        testUserDTO = UserDTO.builder()
            .id(1L)
            .name("John Doe")
            .email("john@example.com")
            .build();
    }

    @Test
    @DisplayName("Should return user when found by ID")
    void getUserById_WhenUserExists_ReturnsUser() {
        // Given
        when(userRepository.findById(1L)).thenReturn(Optional.of(testUser));
        when(userMapper.toDTO(testUser)).thenReturn(testUserDTO);

        // When
        UserDTO result = userService.getUserById(1L);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getName()).isEqualTo("John Doe");

        verify(userRepository).findById(1L);
        verify(userMapper).toDTO(testUser);
    }

    @Test
    @DisplayName("Should throw exception when user not found")
    void getUserById_WhenUserNotExists_ThrowsException() {
        // Given
        when(userRepository.findById(999L)).thenReturn(Optional.empty());

        // When & Then
        assertThatThrownBy(() -> userService.getUserById(999L))
            .isInstanceOf(ResourceNotFoundException.class)
            .hasMessageContaining("User not found");

        verify(userRepository).findById(999L);
        verifyNoInteractions(userMapper);
    }

    @Test
    @DisplayName("Should create user successfully")
    void createUser_WithValidRequest_CreatesUser() {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setName("Jane Doe");
        request.setEmail("jane@example.com");
        request.setPassword("password123");

        when(userRepository.existsByEmail("jane@example.com")).thenReturn(false);
        when(passwordEncoder.encode("password123")).thenReturn("encoded");
        when(userRepository.save(any(User.class))).thenReturn(testUser);
        when(userMapper.toDTO(any(User.class))).thenReturn(testUserDTO);

        // When
        UserDTO result = userService.createUser(request);

        // Then
        assertThat(result).isNotNull();

        verify(userRepository).existsByEmail("jane@example.com");
        verify(passwordEncoder).encode("password123");
        verify(userRepository).save(any(User.class));
    }
}
```

### Integration Testing

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@Testcontainers
@ActiveProfiles("test")
class UserControllerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    @DisplayName("POST /api/users - Should create user")
    void createUser_WithValidRequest_Returns201() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setName("John Doe");
        request.setEmail("john@example.com");
        request.setPassword("Password123");

        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("John Doe"))
            .andExpect(jsonPath("$.email").value("john@example.com"))
            .andExpect(jsonPath("$.id").isNumber());

        assertThat(userRepository.findByEmail("john@example.com")).isPresent();
    }

    @Test
    @DisplayName("GET /api/users/{id} - Should return user")
    void getUserById_WhenExists_Returns200() throws Exception {
        // Given
        User user = userRepository.save(User.builder()
            .name("Jane Doe")
            .email("jane@example.com")
            .password("password")
            .build());

        // When & Then
        mockMvc.perform(get("/api/users/{id}", user.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(user.getId()))
            .andExpect(jsonPath("$.name").value("Jane Doe"));
    }

    @Test
    @DisplayName("GET /api/users/{id} - Should return 404 when not found")
    void getUserById_WhenNotExists_Returns404() throws Exception {
        mockMvc.perform(get("/api/users/{id}", 999))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.message").value("User not found: 999"));
    }
}
```

### @WebMvcTest for Controller Layer

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("Should return 400 for invalid request")
    void createUser_WithInvalidRequest_Returns400() throws Exception {
        CreateUserRequest request = new CreateUserRequest();
        request.setName("");  // Invalid
        request.setEmail("invalid-email");  // Invalid
        request.setPassword("short");  // Invalid

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.validationErrors.name").exists())
            .andExpect(jsonPath("$.validationErrors.email").exists())
            .andExpect(jsonPath("$.validationErrors.password").exists());

        verifyNoInteractions(userService);
    }
}
```

---

## Interview Questions

### Q1: What is the difference between @Component, @Service, @Repository, and @Controller?

**Answer:**
All are specializations of `@Component`:

| Annotation        | Purpose              | Extra Features              |
| ----------------- | -------------------- | --------------------------- |
| `@Component`      | Generic component    | Base annotation             |
| `@Service`        | Business logic layer | Semantic only               |
| `@Repository`     | Data access layer    | Exception translation       |
| `@Controller`     | Web controller       | Request mapping             |
| `@RestController` | REST API controller  | @Controller + @ResponseBody |

```java
@Component      // Generic bean
@Service        // Business logic
@Repository     // DAO - translates persistence exceptions
@Controller     // MVC controller - returns views
@RestController // REST - returns JSON/XML directly
```

---

### Q2: Explain Bean lifecycle in Spring.

**Answer:**

```
Container Start
      │
      ▼
Instantiation (Constructor)
      │
      ▼
Populate Properties (@Autowired)
      │
      ▼
BeanNameAware.setBeanName()
      │
      ▼
BeanFactoryAware.setBeanFactory()
      │
      ▼
ApplicationContextAware.setApplicationContext()
      │
      ▼
BeanPostProcessor.postProcessBeforeInitialization()
      │
      ▼
@PostConstruct / InitializingBean.afterPropertiesSet() / init-method
      │
      ▼
BeanPostProcessor.postProcessAfterInitialization()
      │
      ▼
Bean Ready to Use
      │
      ▼ (Container Shutdown)
      │
@PreDestroy / DisposableBean.destroy() / destroy-method
      │
      ▼
Bean Destroyed
```

---

### Q3: What is the difference between @Autowired and @Inject?

| Feature         | @Autowired              | @Inject                         |
| --------------- | ----------------------- | ------------------------------- |
| **Source**      | Spring                  | JSR-330 (Java standard)         |
| **Required**    | `required=false` option | No (use @Nullable)              |
| **Portability** | Spring-specific         | Portable to other DI frameworks |
| **Qualifiers**  | @Qualifier              | @Named                          |

**Recommendation:** Use `@Autowired` for Spring projects, `@Inject` for portability.

---

### Q4: How does Spring Boot auto-configuration work?

**Answer:**

1. `@EnableAutoConfiguration` triggers scanning
2. Reads `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
3. Each configuration class has `@Conditional` annotations
4. Conditions evaluated (class present, bean missing, property set)
5. Matching configurations create beans

```java
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnProperty(prefix = "spring.datasource", name = "url")
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource() { ... }
}
```

---

### Q5: Explain @Transactional and its propagation levels.

**Answer:**

| Propagation       | Behavior                                |
| ----------------- | --------------------------------------- |
| **REQUIRED**      | Join existing or create new (default)   |
| **REQUIRES_NEW**  | Always create new, suspend existing     |
| **NESTED**        | Create savepoint within existing        |
| **SUPPORTS**      | Use existing or run non-transactional   |
| **MANDATORY**     | Must have existing transaction          |
| **NEVER**         | Must NOT have existing transaction      |
| **NOT_SUPPORTED** | Suspend existing, run non-transactional |

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    timeout = 30,
    readOnly = false,
    rollbackFor = Exception.class
)
public void method() { }
```

---

### Q6: How do you handle exceptions in Spring Boot REST APIs?

**Answer:**

```java
// 1. @ExceptionHandler in controller
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<Error> handleNotFound(UserNotFoundException ex) {
    return ResponseEntity.notFound().build();
}

// 2. @ControllerAdvice for global handling
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAll(Exception ex) {
        return ResponseEntity.status(500).body(new ErrorResponse(ex));
    }
}

// 3. ResponseStatusException
throw new ResponseStatusException(HttpStatus.NOT_FOUND, "User not found");
```

---

### Q7: What is the N+1 problem in JPA and how to solve it?

**Answer:**
N+1 problem: Fetching parent entity + N additional queries for children.

```java
// Problem: N+1 queries
List<User> users = userRepository.findAll(); // 1 query
for (User user : users) {
    user.getOrders().size(); // N queries (one per user)
}

// Solutions:

// 1. JOIN FETCH
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();

// 2. @EntityGraph
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();

// 3. @BatchSize
@OneToMany
@BatchSize(size = 25)
private List<Order> orders;

// 4. Projection (fetch only needed data)
@Query("SELECT new UserOrderCount(u.id, u.name, SIZE(u.orders)) FROM User u")
List<UserOrderCount> getUserOrderCounts();
```

---

### Q8: How does Spring Security authentication work?

**Answer:**

```
Request
   │
   ▼
SecurityFilterChain
   │
   ▼
AuthenticationFilter (extracts credentials)
   │
   ▼
AuthenticationManager
   │
   ▼
AuthenticationProvider
   │
   ▼
UserDetailsService.loadUserByUsername()
   │
   ▼
PasswordEncoder.matches()
   │
   ▼
Authentication object created
   │
   ▼
SecurityContextHolder.setContext()
   │
   ▼
Access granted / denied
```

---

### Q9: Explain Circuit Breaker pattern.

**Answer:**
Circuit breaker prevents cascading failures in distributed systems:

```
         ┌─────────────────────────────────────┐
         │                                     │
         ▼                                     │
      CLOSED ──────(failures reach threshold)──► OPEN
         │                                     │
         │                                     │
    (success)                           (timeout expires)
         │                                     │
         │                                     ▼
         ◄──────(success)──────────────── HALF-OPEN
                                               │
                                          (failure)
                                               │
                                               ▼
                                             OPEN
```

States:

- **CLOSED**: Normal operation
- **OPEN**: Fail fast, return fallback
- **HALF-OPEN**: Test if service recovered

---

### Q10: What are Spring Profiles and how to use them?

**Answer:**
Profiles allow different configurations for different environments:

```java
// Bean specific to profile
@Profile("dev")
@Bean
public DataSource devDataSource() { ... }

@Profile("prod")
@Bean
public DataSource prodDataSource() { ... }

// Conditional component
@Service
@Profile("!prod")  // Not in production
public class MockPaymentService { }
```

```yaml
# application.yml
spring:
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}

---
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:h2:mem:testdb
---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://prod-db:5432/mydb
```

Activation:

- `application.yml`: `spring.profiles.active=prod`
- Command line: `--spring.profiles.active=prod`
- Environment: `SPRING_PROFILES_ACTIVE=prod`

---

### Q11: What is the difference between @RequestParam, @PathVariable, and @RequestBody?

**Answer:**

```java
// @PathVariable - from URL path
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) { }
// GET /users/123 → id = 123

// @RequestParam - from query string
@GetMapping("/users")
public List<User> search(@RequestParam String name,
                         @RequestParam(defaultValue = "0") int page) { }
// GET /users?name=John&page=2 → name="John", page=2

// @RequestBody - from request body (JSON)
@PostMapping("/users")
public User create(@RequestBody @Valid UserDto dto) { }
// POST /users with JSON body
```

| Annotation    | Source       | Required           | Default      |
| ------------- | ------------ | ------------------ | ------------ |
| @PathVariable | URL path     | Yes                | -            |
| @RequestParam | Query string | Yes (configurable) | defaultValue |
| @RequestBody  | Body         | Yes                | -            |

---

### Q12: How do you implement pagination and sorting in Spring Data JPA?

**Answer:**

```java
// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByStatus(String status, Pageable pageable);
}

// Controller
@GetMapping("/users")
public Page<User> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "createdAt,desc") String[] sort) {

    List<Sort.Order> orders = new ArrayList<>();
    for (String s : sort) {
        String[] parts = s.split(",");
        orders.add(new Sort.Order(
            parts.length > 1 && parts[1].equals("desc") ?
                Sort.Direction.DESC : Sort.Direction.ASC,
            parts[0]));
    }

    Pageable pageable = PageRequest.of(page, size, Sort.by(orders));
    return userRepository.findAll(pageable);
}

// Response contains: content, totalElements, totalPages, size, number
```

---

### Q13: What is Spring AOP and when to use it?

**Answer:**
AOP (Aspect-Oriented Programming) handles cross-cutting concerns:

```java
@Aspect
@Component
public class LoggingAspect {

    // Pointcut - where to apply
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}

    // Before advice
    @Before("serviceMethods()")
    public void logBefore(JoinPoint jp) {
        log.info("Calling: {}", jp.getSignature().getName());
    }

    // Around advice - most powerful
    @Around("serviceMethods()")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed();
        } finally {
            log.info("{} took {}ms", pjp.getSignature(),
                     System.currentTimeMillis() - start);
        }
    }

    // After returning
    @AfterReturning(pointcut = "serviceMethods()", returning = "result")
    public void logAfter(Object result) {
        log.info("Returned: {}", result);
    }
}
```

**Use cases:** Logging, security, transactions, caching, auditing.

---

### Q14: What is the difference between @Component, @Bean, and @Configuration?

**Answer:**
| Annotation | Level | Purpose |
|------------|-------|---------|
| @Component | Class | Auto-detected by component scan |
| @Bean | Method | Manual bean creation in config class |
| @Configuration | Class | Declares config class with @Bean methods |

```java
// @Component - automatic
@Component
public class UserValidator { }

// @Bean - manual creation, third-party classes
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();  // Can't add @Component to RestTemplate
    }

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper()
            .registerModule(new JavaTimeModule())
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    }
}
```

**Key difference:** @Configuration uses CGLIB proxy for singleton semantics within the class.

---

### Q15: How do you handle validation in Spring Boot?

**Answer:**

```java
// DTO with validation annotations
public class UserDto {
    @NotBlank(message = "Name is required")
    private String name;

    @Email(message = "Invalid email format")
    @NotNull
    private String email;

    @Size(min = 8, max = 100, message = "Password must be 8-100 chars")
    private String password;

    @Past(message = "Birth date must be in past")
    private LocalDate birthDate;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone")
    private String phone;
}

// Controller with @Valid
@PostMapping("/users")
public ResponseEntity<User> create(@Valid @RequestBody UserDto dto) {
    return ResponseEntity.ok(userService.create(dto));
}

// Global exception handler
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<Map<String, String>> handleValidation(
        MethodArgumentNotValidException ex) {
    Map<String, String> errors = ex.getBindingResult()
        .getFieldErrors()
        .stream()
        .collect(Collectors.toMap(
            FieldError::getField,
            FieldError::getDefaultMessage
        ));
    return ResponseEntity.badRequest().body(errors);
}
```

---

### Q16: What is Spring WebFlux and when to use it?

**Answer:**
WebFlux is Spring's reactive web framework for non-blocking I/O.

| Aspect       | Spring MVC              | Spring WebFlux              |
| ------------ | ----------------------- | --------------------------- |
| **Model**    | Blocking, servlet-based | Non-blocking, reactive      |
| **Threads**  | Thread per request      | Event loop (few threads)    |
| **Server**   | Tomcat, Jetty           | Netty, Undertow             |
| **Use case** | Traditional apps        | High concurrency, streaming |

```java
// Reactive controller
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable Long id) {
        return userRepository.findById(id);
    }

    @GetMapping("/users")
    public Flux<User> getAllUsers() {
        return userRepository.findAll();
    }

    @GetMapping(value = "/users/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamUsers() {
        return userRepository.findAll()
            .delayElements(Duration.ofSeconds(1));
    }
}
```

**Use WebFlux when:**

- High number of concurrent connections
- Streaming data
- Microservices calling multiple external services

---

### Q17: How do you configure and use caching in Spring Boot?

**Answer:**

```java
// Enable caching
@SpringBootApplication
@EnableCaching
public class Application { }

// Cache annotations
@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        return userRepository.findById(id).orElseThrow();
    }

    @CachePut(value = "users", key = "#user.id")
    public User save(User user) {
        return userRepository.save(user);
    }

    @CacheEvict(value = "users", key = "#id")
    public void delete(Long id) {
        userRepository.deleteById(id);
    }

    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() { }
}

// Redis cache configuration
spring:
  cache:
    type: redis
  redis:
    host: localhost
    port: 6379
```

---

### Q18: What is @Conditional and how does Spring Boot use it?

**Answer:**
@Conditional creates beans only when conditions are met.

```java
// Custom condition
public class OnWindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment().getProperty("os.name").contains("Windows");
    }
}

@Bean
@Conditional(OnWindowsCondition.class)
public FileService windowsFileService() { }

// Built-in conditions (Spring Boot)
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
@ConditionalOnBean(JpaRepository.class)
@ConditionalOnMissingClass("com.example.SomeClass")
@ConditionalOnWebApplication
@ConditionalOnNotWebApplication
```

**Auto-configuration uses these to enable features only when needed.**

---

### Q19: How do you implement rate limiting in Spring Boot?

**Answer:**

```java
// Using Bucket4j
@Configuration
public class RateLimitConfig {
    @Bean
    public Bucket bucket() {
        return Bucket.builder()
            .addLimit(Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(1))))
            .build();
    }
}

// Filter or Interceptor
@Component
public class RateLimitFilter extends OncePerRequestFilter {
    @Autowired private Bucket bucket;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain chain) throws IOException {
        if (bucket.tryConsume(1)) {
            chain.doFilter(request, response);
        } else {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Rate limit exceeded");
        }
    }
}

// With Spring Cloud Gateway
spring:
  cloud:
    gateway:
      routes:
        - id: api_route
          uri: lb://api-service
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

---

### Q20: Explain the difference between @RestController and @Controller.

**Answer:**
| @Controller | @RestController |
|-------------|-----------------|
| Returns view names | Returns data directly |
| Needs @ResponseBody on methods | @ResponseBody implicit |
| For MVC web apps | For REST APIs |

```java
// @Controller - MVC
@Controller
public class WebController {
    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("message", "Hello");
        return "home";  // View name
    }

    @GetMapping("/api/data")
    @ResponseBody  // Needed to return JSON
    public DataDto getData() {
        return new DataDto();
    }
}

// @RestController = @Controller + @ResponseBody
@RestController
@RequestMapping("/api")
public class ApiController {
    @GetMapping("/users")
    public List<User> getUsers() {  // Returns JSON directly
        return userService.findAll();
    }
}
```

---

## Key Takeaways

| Topic             | Key Point                                                    |
| ----------------- | ------------------------------------------------------------ |
| **DI**            | Prefer constructor injection                                 |
| **Scopes**        | Singleton is default, use prototype for stateful beans       |
| **Auto-config**   | Conditional beans based on classpath and properties          |
| **Transactions**  | Use @Transactional, understand propagation                   |
| **Security**      | Filter chain, stateless JWT for APIs                         |
| **Testing**       | @WebMvcTest for controllers, @SpringBootTest for integration |
| **Microservices** | Use Service Discovery, API Gateway, Circuit Breaker          |
