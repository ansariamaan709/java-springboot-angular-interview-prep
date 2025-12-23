# Spring Boot Testing Strategies - Complete Interview Guide

## Table of Contents

1. [Testing Pyramid & Strategy](#testing-pyramid--strategy)
2. [Unit Testing (@SpringBootTest vs @WebMvcTest)](#unit-testing-springboottest-vs-webmvctest)
3. [Integration Testing](#integration-testing)
4. [Contract Testing](#contract-testing)
5. [Testcontainers](#testcontainers)
6. [Mocking with Mockito](#mocking-with-mockito)
7. [REST API Testing](#rest-api-testing)
8. [Security Testing](#security-testing)
9. [Performance Testing](#performance-testing)
10. [Interview Questions & Answers](#interview-questions--answers)

---

## Testing Pyramid & Strategy

### Testing Pyramid

```
        /\
       /  \           E2E Tests (Few)
      /----\          - Slow, brittle, expensive
     /      \         - Test critical user journeys
    /--------\
   / Integration\     Integration Tests (Some)
  /    Tests     \    - Medium speed
 /--------------\
/  Unit Tests    \   Unit Tests (Many)
/                 \  - Fast, isolated, cheap
--------------------
```

### Spring Boot Testing Annotations

| Annotation        | Purpose                  | What It Loads                    | Use Case             |
| ----------------- | ------------------------ | -------------------------------- | -------------------- |
| `@SpringBootTest` | Full application context | Entire Spring context            | Integration tests    |
| `@WebMvcTest`     | Web layer only           | Controllers, filters, validation | Controller tests     |
| `@DataJpaTest`    | JPA layer only           | Repositories, EntityManager      | Repository tests     |
| `@RestClientTest` | REST client testing      | RestTemplate, WebClient          | Client tests         |
| `@JsonTest`       | JSON serialization       | Jackson ObjectMapper             | JSON testing         |
| `@MockBean`       | Mockito mock in Spring   | Creates mock bean                | Replace dependencies |
| `@SpyBean`        | Mockito spy in Spring    | Spies on real bean               | Partial mocking      |

---

## Unit Testing (@SpringBootTest vs @WebMvcTest)

### 1. Controller Unit Testing with @WebMvcTest

**FAST, ISOLATED** - Only loads web layer (controllers, filters, advice, converters).

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void createUser_ValidInput_ReturnsCreated() throws Exception {
        // Given
        UserRequest request = new UserRequest("john@example.com", "John Doe");
        UserResponse response = new UserResponse(1L, "john@example.com", "John Doe");

        when(userService.createUser(any(UserRequest.class))).thenReturn(response);

        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"john@example.com\",\"name\":\"John Doe\"}"))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.email").value("john@example.com"))
                .andExpect(jsonPath("$.name").value("John Doe"));

        verify(userService).createUser(any(UserRequest.class));
    }

    @Test
    void createUser_InvalidEmail_ReturnsBadRequest() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"invalid-email\",\"name\":\"John Doe\"}"))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.errors[0].field").value("email"))
                .andExpect(jsonPath("$.errors[0].message").value("must be a well-formed email address"));
    }

    @Test
    void getUser_NotFound_Returns404() throws Exception {
        when(userService.getUser(999L)).thenThrow(new UserNotFoundException(999L));

        mockMvc.perform(get("/api/users/999"))
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.message").value("User not found with id: 999"));
    }
}
```

### 2. Service Unit Testing (Pure Unit Test)

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Test
    void createUser_ValidInput_SavesAndSendsEmail() {
        // Given
        UserRequest request = new UserRequest("john@example.com", "John Doe");
        User user = new User(null, "john@example.com", "John Doe");
        User savedUser = new User(1L, "john@example.com", "John Doe");

        when(userRepository.save(any(User.class))).thenReturn(savedUser);

        // When
        UserResponse response = userService.createUser(request);

        // Then
        assertThat(response.id()).isEqualTo(1L);
        assertThat(response.email()).isEqualTo("john@example.com");

        verify(userRepository).save(any(User.class));
        verify(emailService).sendWelcomeEmail("john@example.com", "John Doe");
    }

    @Test
    void createUser_DuplicateEmail_ThrowsException() {
        UserRequest request = new UserRequest("john@example.com", "John Doe");
        when(userRepository.save(any(User.class)))
                .thenThrow(new DataIntegrityViolationException("Duplicate email"));

        assertThatThrownBy(() -> userService.createUser(request))
                .isInstanceOf(DuplicateUserException.class)
                .hasMessageContaining("john@example.com");
    }
}
```

### 3. Repository Testing with @DataJpaTest

**IN-MEMORY DATABASE** - Uses H2 by default, auto-rollback.

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void findByEmail_ExistingUser_ReturnsUser() {
        // Given
        User user = new User(null, "john@example.com", "John Doe");
        entityManager.persistAndFlush(user);

        // When
        Optional<User> found = userRepository.findByEmail("john@example.com");

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("John Doe");
    }

    @Test
    void findByEmailContaining_PartialMatch_ReturnsUsers() {
        entityManager.persist(new User(null, "john@example.com", "John Doe"));
        entityManager.persist(new User(null, "jane@example.com", "Jane Doe"));
        entityManager.persist(new User(null, "bob@gmail.com", "Bob Smith"));
        entityManager.flush();

        List<User> users = userRepository.findByEmailContaining("example");

        assertThat(users).hasSize(2);
        assertThat(users).extracting(User::getEmail)
                .containsExactlyInAnyOrder("john@example.com", "jane@example.com");
    }

    @Test
    void customQuery_FindActiveUsers_ReturnsCorrectUsers() {
        entityManager.persist(new User(null, "active1@example.com", "Active One", true));
        entityManager.persist(new User(null, "active2@example.com", "Active Two", true));
        entityManager.persist(new User(null, "inactive@example.com", "Inactive", false));
        entityManager.flush();

        List<User> activeUsers = userRepository.findActiveUsers();

        assertThat(activeUsers).hasSize(2);
        assertThat(activeUsers).allMatch(User::isActive);
    }
}
```

---

## Integration Testing

### 1. Full Integration Test with @SpringBootTest

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class UserIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void registerPgProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void createUser_EndToEnd_SavesInDatabaseAndReturns201() {
        // Given
        UserRequest request = new UserRequest("john@example.com", "John Doe");

        // When
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
                "/api/users",
                request,
                UserResponse.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody()).isNotNull();
        assertThat(response.getBody().email()).isEqualTo("john@example.com");

        // Verify database
        List<User> users = userRepository.findAll();
        assertThat(users).hasSize(1);
        assertThat(users.get(0).getEmail()).isEqualTo("john@example.com");
    }

    @Test
    void createUser_DuplicateEmail_Returns409Conflict() {
        // Given - existing user
        userRepository.save(new User(null, "john@example.com", "Existing User"));

        UserRequest request = new UserRequest("john@example.com", "John Doe");

        // When
        ResponseEntity<ErrorResponse> response = restTemplate.postForEntity(
                "/api/users",
                request,
                ErrorResponse.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CONFLICT);
        assertThat(response.getBody().message()).contains("already exists");
    }
}
```

### 2. Testing with Profiles

```java
@SpringBootTest
@ActiveProfiles("test")
class OrderServiceIntegrationTest {

    @Autowired
    private OrderService orderService;

    @Autowired
    private PaymentClient paymentClient; // Mocked in test profile

    @Test
    void createOrder_WithTestProfile_UsesTestConfiguration() {
        // Test profile uses mock payment client
        Order order = orderService.createOrder(new OrderRequest(100.0));

        assertThat(order.getStatus()).isEqualTo(OrderStatus.PENDING);
    }
}
```

**application-test.yml**:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
  jpa:
    hibernate:
      ddl-auto: create-drop
  kafka:
    bootstrap-servers: ${spring.embedded.kafka.brokers}

payment:
  client:
    url: http://localhost:${wiremock.server.port}
```

---

## Contract Testing

### 1. Producer Contract Testing (Spring Cloud Contract)

**Producer Side** - Define contracts in `src/test/resources/contracts`:

```groovy
// src/test/resources/contracts/user/shouldReturnUserById.groovy
package contracts.user

import org.springframework.cloud.contract.spec.Contract

Contract.make {
    description "should return user by id"
    request {
        method GET()
        url("/api/users/1")
        headers {
            contentType(applicationJson())
        }
    }
    response {
        status 200
        body([
            id: 1,
            email: "john@example.com",
            name: "John Doe"
        ])
        headers {
            contentType(applicationJson())
        }
    }
}
```

**Base Test Class**:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public abstract class BaseContractTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @BeforeEach
    void setup() {
        UserResponse user = new UserResponse(1L, "john@example.com", "John Doe");
        when(userService.getUser(1L)).thenReturn(user);
    }
}
```

**Generated Test** (auto-generated):

```java
public class ContractVerifierTest extends BaseContractTest {
    @Test
    public void validate_shouldReturnUserById() throws Exception {
        // given:
        MockMvcRequestSpecification request = given();

        // when:
        ResponseOptions response = given().spec(request)
                .get("/api/users/1");

        // then:
        assertThat(response.statusCode()).isEqualTo(200);
        // ... assertions
    }
}
```

### 2. Consumer Contract Testing (Pact)

```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "UserService", port = "8080")
class UserClientPactTest {

    @Pact(consumer = "OrderService")
    public RequestResponsePact createPact(PactDslWithProvider builder) {
        return builder
                .given("user with id 1 exists")
                .uponReceiving("a request to get user by id")
                .path("/api/users/1")
                .method("GET")
                .willRespondWith()
                .status(200)
                .body(new PactDslJsonBody()
                        .numberType("id", 1L)
                        .stringType("email", "john@example.com")
                        .stringType("name", "John Doe"))
                .toPact();
    }

    @Test
    @PactTestFor(pactMethod = "createPact")
    void testGetUser(MockServer mockServer) {
        UserClient client = new UserClient(mockServer.getUrl());
        UserResponse user = client.getUser(1L);

        assertThat(user.id()).isEqualTo(1L);
        assertThat(user.email()).isEqualTo("john@example.com");
    }
}
```

---

## Testcontainers

### 1. Single Container Testing

```java
@SpringBootTest
@Testcontainers
class PostgresIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test")
            .withInitScript("init.sql"); // Optional

    @DynamicPropertySource
    static void registerProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void testDatabaseOperations() {
        User user = userRepository.save(new User(null, "test@example.com", "Test User"));
        assertThat(user.getId()).isNotNull();
    }
}
```

### 2. Multiple Containers with Docker Compose

```java
@SpringBootTest
@Testcontainers
class MultiContainerIntegrationTest {

    @Container
    static DockerComposeContainer<?> environment =
            new DockerComposeContainer<>(new File("src/test/resources/docker-compose-test.yml"))
                    .withExposedService("postgres", 5432)
                    .withExposedService("redis", 6379)
                    .withExposedService("kafka", 9092);

    @DynamicPropertySource
    static void registerProperties(DynamicPropertyRegistry registry) {
        String postgresUrl = String.format("jdbc:postgresql://%s:%d/testdb",
                environment.getServiceHost("postgres", 5432),
                environment.getServicePort("postgres", 5432));

        registry.add("spring.datasource.url", () -> postgresUrl);
        registry.add("spring.redis.host", () -> environment.getServiceHost("redis", 6379));
        registry.add("spring.kafka.bootstrap-servers",
                () -> environment.getServiceHost("kafka", 9092) + ":" +
                        environment.getServicePort("kafka", 9092));
    }

    @Test
    void testFullStack() {
        // Test with all services running
    }
}
```

### 3. Reusable Containers (Performance Optimization)

```java
public abstract class AbstractIntegrationTest {

    static PostgreSQLContainer<?> postgres;
    static RedisContainer redis;

    static {
        postgres = new PostgreSQLContainer<>("postgres:15-alpine")
                .withReuse(true);
        redis = new RedisContainer("redis:7-alpine")
                .withReuse(true);

        postgres.start();
        redis.start();
    }

    @DynamicPropertySource
    static void registerProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.redis.host", redis::getHost);
    }
}

// Usage
class UserServiceTest extends AbstractIntegrationTest {
    @Test
    void test() {
        // Containers are shared across test classes
    }
}
```

---

## Mocking with Mockito

### 1. Argument Captors

```java
@Test
void createOrder_SendsEmailWithCorrectDetails() {
    // Given
    ArgumentCaptor<EmailMessage> emailCaptor = ArgumentCaptor.forClass(EmailMessage.class);
    OrderRequest request = new OrderRequest(100.0);

    // When
    orderService.createOrder(request);

    // Then
    verify(emailService).sendEmail(emailCaptor.capture());
    EmailMessage sentEmail = emailCaptor.getValue();

    assertThat(sentEmail.getRecipient()).isEqualTo("customer@example.com");
    assertThat(sentEmail.getSubject()).contains("Order Confirmation");
    assertThat(sentEmail.getBody()).contains("$100.0");
}
```

### 2. Custom Argument Matchers

```java
@Test
void createOrder_CallsPaymentServiceWithCorrectAmount() {
    verify(paymentService).processPayment(argThat(payment ->
            payment.getAmount().equals(100.0) &&
            payment.getCurrency().equals("USD") &&
            payment.getType().equals(PaymentType.CREDIT_CARD)
    ));
}
```

### 3. Mockito BDD Style

```java
@Test
void getUserOrders_ReturnsFormattedOrders() {
    // Given (BDD style)
    given(orderRepository.findByUserId(1L))
            .willReturn(Arrays.asList(
                    new Order(1L, 100.0),
                    new Order(2L, 200.0)
            ));

    // When
    List<OrderResponse> orders = orderService.getUserOrders(1L);

    // Then
    then(orderRepository).should().findByUserId(1L);
    assertThat(orders).hasSize(2);
}
```

---

## REST API Testing

### 1. REST Assured

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserApiTest {

    @LocalServerPort
    private int port;

    @BeforeEach
    void setUp() {
        RestAssured.port = port;
        RestAssured.basePath = "/api";
    }

    @Test
    void createUser_ValidInput_Returns201() {
        given()
                .contentType(ContentType.JSON)
                .body(new UserRequest("john@example.com", "John Doe"))
        .when()
                .post("/users")
        .then()
                .statusCode(201)
                .body("email", equalTo("john@example.com"))
                .body("name", equalTo("John Doe"))
                .body("id", notNullValue());
    }

    @Test
    void getUsers_WithPagination_ReturnsCorrectPage() {
        given()
                .queryParam("page", 0)
                .queryParam("size", 10)
                .queryParam("sort", "name,asc")
        .when()
                .get("/users")
        .then()
                .statusCode(200)
                .body("content", hasSize(lessThanOrEqualTo(10)))
                .body("totalPages", greaterThanOrEqualTo(1))
                .body("number", equalTo(0));
    }

    @Test
    void updateUser_WithETag_ReturnsCorrectResponse() {
        // Get user with ETag
        String etag = given()
                .get("/users/1")
                .then()
                .statusCode(200)
                .extract()
                .header("ETag");

        // Update with If-Match header
        given()
                .contentType(ContentType.JSON)
                .header("If-Match", etag)
                .body(new UserUpdateRequest("Updated Name"))
        .when()
                .put("/users/1")
        .then()
                .statusCode(200);
    }
}
```

### 2. WebTestClient (WebFlux)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureWebTestClient
class ReactiveUserControllerTest {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void getAllUsers_ReturnsFlux() {
        webTestClient.get()
                .uri("/api/users")
                .exchange()
                .expectStatus().isOk()
                .expectHeader().contentType(MediaType.APPLICATION_JSON)
                .expectBodyList(UserResponse.class)
                .hasSize(3)
                .consumeWith(response -> {
                    List<UserResponse> users = response.getResponseBody();
                    assertThat(users).extracting(UserResponse::email)
                            .contains("user1@example.com", "user2@example.com");
                });
    }

    @Test
    void streamUsers_ServerSentEvents() {
        webTestClient.get()
                .uri("/api/users/stream")
                .accept(MediaType.TEXT_EVENT_STREAM)
                .exchange()
                .expectStatus().isOk()
                .returnResult(UserResponse.class)
                .getResponseBody()
                .take(5)
                .collectList()
                .block();
    }
}
```

---

## Security Testing

### 1. Testing with Spring Security

```java
@WebMvcTest(UserController.class)
@Import(SecurityConfig.class)
class SecureControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    @WithMockUser(username = "admin", roles = {"ADMIN"})
    void deleteUser_AsAdmin_Returns204() throws Exception {
        mockMvc.perform(delete("/api/users/1"))
                .andExpect(status().isNoContent());
    }

    @Test
    @WithMockUser(username = "user", roles = {"USER"})
    void deleteUser_AsRegularUser_Returns403() throws Exception {
        mockMvc.perform(delete("/api/users/1"))
                .andExpect(status().isForbidden());
    }

    @Test
    void deleteUser_Unauthenticated_Returns401() throws Exception {
        mockMvc.perform(delete("/api/users/1"))
                .andExpect(status().isUnauthorized());
    }
}
```

### 2. Custom Security Test Annotations

```java
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(username = "admin", roles = {"ADMIN"})
public @interface WithMockAdmin {
}

@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(username = "user", roles = {"USER"})
public @interface WithMockRegularUser {
}

// Usage
@Test
@WithMockAdmin
void adminOnlyEndpoint_AsAdmin_Returns200() throws Exception {
    // Test code
}
```

### 3. Testing JWT Authentication

```java
@Test
void getUserProfile_WithValidJWT_Returns200() throws Exception {
    String token = jwtTokenProvider.createToken("john@example.com", List.of("ROLE_USER"));

    mockMvc.perform(get("/api/users/me")
            .header("Authorization", "Bearer " + token))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.email").value("john@example.com"));
}

@Test
void getUserProfile_WithExpiredJWT_Returns401() throws Exception {
    String expiredToken = createExpiredToken();

    mockMvc.perform(get("/api/users/me")
            .header("Authorization", "Bearer " + expiredToken))
            .andExpect(status().isUnauthorized());
}
```

---

## Performance Testing

### 1. JMH Microbenchmarking

```java
@State(Scope.Benchmark)
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
@Fork(1)
@Warmup(iterations = 3, time = 1)
@Measurement(iterations = 5, time = 1)
public class UserServiceBenchmark {

    private UserService userService;
    private UserRepository userRepository;

    @Setup
    public void setup() {
        userRepository = mock(UserRepository.class);
        userService = new UserService(userRepository);
    }

    @Benchmark
    public UserResponse testCreateUser() {
        UserRequest request = new UserRequest("test@example.com", "Test User");
        return userService.createUser(request);
    }
}
```

### 2. Load Testing with Gatling

```java
public class UserLoadSimulation extends Simulation {

    HttpProtocolBuilder httpProtocol = http
            .baseUrl("http://localhost:8080")
            .acceptHeader("application/json");

    ScenarioBuilder scn = scenario("User Creation")
            .exec(http("Create User")
                    .post("/api/users")
                    .body(StringBody(
                            "{\"email\":\"user${randomEmail}@example.com\",\"name\":\"User ${randomId}\"}"
                    ))
                    .check(status().is(201)))
            .pause(1);

    {
        setUp(
                scn.injectOpen(
                        rampUsersPerSec(10).to(100).during(Duration.ofMinutes(5)),
                        constantUsersPerSec(100).during(Duration.ofMinutes(10))
                )
        ).protocols(httpProtocol);
    }
}
```

---

## Interview Questions & Answers

### Q1: What's the difference between @SpringBootTest and @WebMvcTest?

**Answer:**

| Aspect           | @SpringBootTest                | @WebMvcTest                              |
| ---------------- | ------------------------------ | ---------------------------------------- |
| **Context**      | Full application context       | Web layer only                           |
| **Speed**        | Slower (loads everything)      | Faster (sliced test)                     |
| **Beans Loaded** | All beans                      | Controllers, filters, validators, advice |
| **Use Case**     | Integration tests              | Controller unit tests                    |
| **Database**     | Full database connection       | No database beans                        |
| **MockMvc**      | Requires @AutoConfigureMockMvc | Auto-configured                          |

```java
// @WebMvcTest - Fast, isolated controller testing
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean UserService userService; // Mock dependencies
}

// @SpringBootTest - Full integration test
@SpringBootTest(webEnvironment = RANDOM_PORT)
class UserIntegrationTest {
    @Autowired TestRestTemplate restTemplate;
    // Real beans, real database
}
```

**When to use:**

- **@WebMvcTest**: Unit test controllers, validate request/response, test validation
- **@SpringBootTest**: Integration tests, end-to-end flows, test real database

---

### Q2: How do you test asynchronous methods?

**Answer:**

```java
@Service
public class EmailService {
    @Async
    public CompletableFuture<Void> sendEmail(String to, String subject) {
        // Send email
        return CompletableFuture.completedFuture(null);
    }
}

// Test
@SpringBootTest
@EnableAsync
class EmailServiceTest {

    @Autowired
    private EmailService emailService;

    @Test
    void testAsyncMethod() throws Exception {
        CompletableFuture<Void> future = emailService.sendEmail("test@example.com", "Test");

        // Wait for completion
        future.get(5, TimeUnit.SECONDS);

        // Verify email was sent
        verify(mailSender).send(any(SimpleMailMessage.class));
    }

    // Alternative: Use Awaitility
    @Test
    void testAsyncWithAwaitility() {
        emailService.sendEmail("test@example.com", "Test");

        await().atMost(5, SECONDS).untilAsserted(() ->
                verify(mailSender).send(any(SimpleMailMessage.class))
        );
    }
}
```

---

### Q3: How do you test Kafka producers and consumers?

**Answer:**

```java
@SpringBootTest
@EmbeddedKafka(partitions = 1, topics = {"order-events"})
class OrderKafkaTest {

    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;

    @Autowired
    private OrderEventConsumer consumer;

    @Test
    void testKafkaProducerConsumer() throws Exception {
        // Produce
        OrderEvent event = new OrderEvent(1L, "CREATED");
        kafkaTemplate.send("order-events", event);

        // Consume - wait for message
        await().atMost(5, SECONDS).untilAsserted(() ->
                assertThat(consumer.getReceivedEvents()).hasSize(1)
        );

        OrderEvent received = consumer.getReceivedEvents().get(0);
        assertThat(received.getOrderId()).isEqualTo(1L);
    }
}

// With Testcontainers
@Testcontainers
class OrderKafkaTestcontainersTest {

    @Container
    static KafkaContainer kafka = new KafkaContainer(
            DockerImageName.parse("confluentinc/cp-kafka:7.5.0")
    );

    @DynamicPropertySource
    static void kafkaProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }
}
```

---

### Q4: How do you test caching?

**Answer:**

```java
@SpringBootTest
@EnableCaching
class CacheTest {

    @Autowired
    private UserService userService;

    @Autowired
    private CacheManager cacheManager;

    @BeforeEach
    void clearCache() {
        cacheManager.getCacheNames().forEach(cacheName ->
                Objects.requireNonNull(cacheManager.getCache(cacheName)).clear()
        );
    }

    @Test
    void getUserById_CachesResult() {
        // First call - hits database
        UserResponse user1 = userService.getUserById(1L);
        verify(userRepository, times(1)).findById(1L);

        // Second call - hits cache
        UserResponse user2 = userService.getUserById(1L);
        verify(userRepository, times(1)).findById(1L); // Still only 1 call

        assertThat(user1).isEqualTo(user2);

        // Verify cache contains the user
        Cache userCache = cacheManager.getCache("users");
        assertThat(userCache.get(1L)).isNotNull();
    }

    @Test
    void updateUser_EvictsCache() {
        // Populate cache
        userService.getUserById(1L);

        // Update user (should evict cache)
        userService.updateUser(1L, new UserUpdateRequest("New Name"));

        // Next call should hit database again
        userService.getUserById(1L);
        verify(userRepository, times(2)).findById(1L);
    }
}
```

---

### Q5: How do you test database transactions and rollback?

**Answer:**

```java
@SpringBootTest
@Transactional // Auto-rollback after each test
class TransactionTest {

    @Autowired
    private OrderService orderService;

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private PaymentRepository paymentRepository;

    @Test
    void createOrder_RollbackOnPaymentFailure() {
        // Given
        OrderRequest request = new OrderRequest(100.0);
        when(paymentService.processPayment(any()))
                .thenThrow(new PaymentException("Insufficient funds"));

        // When
        assertThatThrownBy(() -> orderService.createOrder(request))
                .isInstanceOf(PaymentException.class);

        // Then - transaction should be rolled back
        assertThat(orderRepository.findAll()).isEmpty();
        assertThat(paymentRepository.findAll()).isEmpty();
    }

    @Test
    @Commit // Override auto-rollback
    void createOrder_CommitsOnSuccess() {
        OrderRequest request = new OrderRequest(100.0);
        orderService.createOrder(request);

        // Data is committed to database
        assertThat(orderRepository.findAll()).hasSize(1);
    }
}
```

---

### Q6: How do you test file uploads/downloads?

**Answer:**

```java
@WebMvcTest(FileController.class)
class FileControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private FileStorageService fileStorageService;

    @Test
    void uploadFile_ValidFile_Returns200() throws Exception {
        MockMultipartFile file = new MockMultipartFile(
                "file",
                "test.txt",
                MediaType.TEXT_PLAIN_VALUE,
                "Hello World".getBytes()
        );

        when(fileStorageService.store(any(MultipartFile.class)))
                .thenReturn("file-12345.txt");

        mockMvc.perform(multipart("/api/files/upload")
                .file(file))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.filename").value("file-12345.txt"));
    }

    @Test
    void downloadFile_ExistingFile_ReturnsFile() throws Exception {
        byte[] content = "File content".getBytes();
        Resource resource = new ByteArrayResource(content);

        when(fileStorageService.loadAsResource("test.txt")).thenReturn(resource);

        mockMvc.perform(get("/api/files/download/test.txt"))
                .andExpect(status().isOk())
                .andExpect(header().string("Content-Disposition", "attachment; filename=\"test.txt\""))
                .andExpect(content().bytes(content));
    }
}
```

---

### Q7: How do you test scheduled tasks?

**Answer:**

```java
@SpringBootTest
class ScheduledTaskTest {

    @Autowired
    private ScheduledReportService scheduledReportService;

    @SpyBean
    private ReportGenerator reportGenerator;

    @Test
    void scheduledTask_ExecutesAtScheduledTime() throws Exception {
        // Trigger manually for testing
        scheduledReportService.generateDailyReport();

        verify(reportGenerator).generateReport(any());
    }

    // Alternative: Use Awaitility to wait for scheduled execution
    @Test
    @EnableScheduling
    void scheduledTask_ExecutesAutomatically() {
        await().atMost(65, SECONDS)
                .untilAsserted(() ->
                        verify(reportGenerator, atLeastOnce()).generateReport(any())
                );
    }
}

// Or use @Scheduled(fixedDelay = 100) in test profile
@Configuration
@Profile("test")
public class TestSchedulingConfig {
    @Bean
    public ScheduledReportService scheduledReportService() {
        return new ScheduledReportService() {
            @Scheduled(fixedDelay = 100) // Fast execution for testing
            public void generateDailyReport() {
                super.generateDailyReport();
            }
        };
    }
}
```

---

### Q8: What is contract testing and why is it important in microservices?

**Answer:**

**Contract Testing** verifies that services can communicate correctly by testing the **contract** (API specification) between producer and consumer.

**Problem it solves:**

```
Consumer Service                  Producer Service
    |                                    |
    |------- GET /api/users/1 --------->|
    |                                    |
    |<------ { id, email, name } --------|
```

**Without contract testing:**

- Producer changes response format → Consumer breaks in production
- Integration tests catch this too late
- E2E tests are slow and brittle

**With contract testing (Producer-Driven):**

```java
// Producer defines contract
Contract.make {
    request {
        method GET()
        url("/api/users/1")
    }
    response {
        status 200
        body([id: 1, email: "john@example.com", name: "John Doe"])
    }
}

// Test verifies producer honors contract
// Stub is generated for consumer to test against
```

**Benefits:**

1. **Fast feedback**: Catch breaking changes before deployment
2. **Independent testing**: Teams test in isolation
3. **Living documentation**: Contracts document API behavior
4. **CI/CD integration**: Automated verification

---

### Q9: How do you handle flaky tests?

**Answer:**

**Causes of flaky tests:**

1. **Race conditions**: Async operations, timing issues
2. **External dependencies**: Database, network, third-party APIs
3. **Shared state**: Tests interfere with each other
4. **Non-deterministic data**: Random values, timestamps

**Solutions:**

```java
// 1. Use Awaitility for async operations
@Test
void asyncOperation_CompletesSuccessfully() {
    service.triggerAsyncOperation();

    // DON'T: Thread.sleep(1000); // Flaky!

    // DO: Wait with timeout
    await().atMost(5, SECONDS)
            .pollInterval(100, MILLISECONDS)
            .untilAsserted(() ->
                    assertThat(service.isCompleted()).isTrue()
            );
}

// 2. Isolate test data
@BeforeEach
void setUp() {
    repository.deleteAll(); // Clean slate
    // Or use @DirtiesContext (expensive)
}

// 3. Use Testcontainers for real dependencies
@Container
static PostgreSQLContainer<?> postgres = ...;

// 4. Mock time-dependent code
@MockBean
private Clock clock;

@Test
void testTimeDependent() {
    when(clock.instant()).thenReturn(Instant.parse("2024-01-01T00:00:00Z"));
    // Test with fixed time
}

// 5. Retry flaky tests (last resort)
@RepeatedTest(3) // Repeat up to 3 times
void potentiallyFlakyTest() {
    // Test code
}
```

---

### Q10: How do you test exception handling and error responses?

**Answer:**

```java
@WebMvcTest(UserController.class)
class ExceptionHandlingTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void getUser_NotFound_Returns404WithErrorDetails() throws Exception {
        when(userService.getUser(999L))
                .thenThrow(new UserNotFoundException("User not found with id: 999"));

        mockMvc.perform(get("/api/users/999"))
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.status").value(404))
                .andExpect(jsonPath("$.error").value("Not Found"))
                .andExpect(jsonPath("$.message").value("User not found with id: 999"))
                .andExpect(jsonPath("$.timestamp").exists())
                .andExpect(jsonPath("$.path").value("/api/users/999"));
    }

    @Test
    void createUser_ValidationFails_Returns400WithFieldErrors() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"invalid\",\"name\":\"\"}"))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.errors", hasSize(2)))
                .andExpect(jsonPath("$.errors[*].field", containsInAnyOrder("email", "name")))
                .andExpect(jsonPath("$.errors[?(@.field == 'email')].message")
                        .value("must be a well-formed email address"))
                .andExpect(jsonPath("$.errors[?(@.field == 'name')].message")
                        .value("must not be blank"));
    }

    @Test
    void createUser_InternalError_Returns500() throws Exception {
        when(userService.createUser(any()))
                .thenThrow(new RuntimeException("Database connection failed"));

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"john@example.com\",\"name\":\"John\"}"))
                .andExpect(status().isInternalServerError())
                .andExpect(jsonPath("$.message").value("An unexpected error occurred"));
        // Don't expose internal error details to client
    }
}
```

---

## Summary & Best Practices

### Testing Strategy Checklist

✅ **Unit Tests (70%)**

- Use `@WebMvcTest`, `@DataJpaTest` for sliced testing
- Mock all external dependencies
- Fast execution (< 1 second per test)
- Focus on business logic

✅ **Integration Tests (20%)**

- Use `@SpringBootTest` with Testcontainers
- Test real database, cache, messaging
- Cover critical user journeys
- Validate cross-layer interactions

✅ **Contract Tests (5%)**

- Verify API contracts between services
- Generate stubs for consumers
- Catch breaking changes early

✅ **E2E Tests (5%)**

- Test critical business flows
- Run in CI before production
- Keep minimal and maintainable

### Performance Tips

| Technique                      | Impact                          |
| ------------------------------ | ------------------------------- |
| Use sliced tests (@WebMvcTest) | 10x faster than @SpringBootTest |
| Reuse Testcontainers           | 5x faster (shared containers)   |
| Parallel test execution        | 2-4x faster                     |
| Mock external services         | 100x faster than real calls     |

### Common Pitfalls

❌ **Don't:**

- Use `Thread.sleep()` for waiting
- Share mutable state between tests
- Test framework code (Spring Boot auto-configuration)
- Write E2E tests for everything

✅ **Do:**

- Use Awaitility for async operations
- Isolate test data (`@BeforeEach` cleanup)
- Test your business logic
- Follow the testing pyramid

---

**Key Takeaway:** Testing is NOT about coverage percentage—it's about **confidence**. Write tests that catch real bugs and enable safe refactoring.
