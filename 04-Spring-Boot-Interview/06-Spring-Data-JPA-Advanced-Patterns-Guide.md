# Spring Data JPA Advanced Patterns — Complete Interview Guide

## Table of Contents

1. [Repository Patterns & Custom Queries](#repository-patterns--custom-queries)
2. [Projections (Interface, Class, Dynamic)](#projections-interface-class-dynamic)
3. [Specifications & Criteria API](#specifications--criteria-api)
4. [Auditing & Entity Listeners](#auditing--entity-listeners)
5. [Transaction Management Deep Dive](#transaction-management-deep-dive)
6. [Connection Pooling (HikariCP)](#connection-pooling-hikaricp)
7. [Database Migrations (Flyway/Liquibase)](#database-migrations-flywayliquibase)
8. [Performance Optimization](#performance-optimization)
9. [Multi-Tenancy Patterns](#multi-tenancy-patterns)
10. [Common Pitfalls & Solutions](#common-pitfalls--solutions)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## Repository Patterns & Custom Queries

### Repository Hierarchy

```
                    Repository<T, ID>
                          │
                          ▼
                 CrudRepository<T, ID>
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
   PagingAndSortingRepository    ListCrudRepository
              │
              ▼
       JpaRepository<T, ID>
              │
              ▼
    JpaSpecificationExecutor<T>
```

### 1. Query Derivation Methods

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Simple property matching
    List<User> findByEmail(String email);
    List<User> findByEmailAndStatus(String email, Status status);
    
    // Comparison operators
    List<User> findByAgeGreaterThan(int age);
    List<User> findByAgeBetween(int start, int end);
    List<User> findByCreatedAtAfter(LocalDateTime date);
    
    // String matching
    List<User> findByNameContaining(String name);          // LIKE %name%
    List<User> findByNameStartingWith(String prefix);      // LIKE prefix%
    List<User> findByEmailEndingWith(String domain);       // LIKE %domain
    List<User> findByNameIgnoreCase(String name);
    
    // Collection operations
    List<User> findByStatusIn(Collection<Status> statuses);
    List<User> findByStatusNotIn(Collection<Status> statuses);
    
    // Null checks
    List<User> findByDeletedAtIsNull();
    List<User> findByDeletedAtIsNotNull();
    
    // Boolean
    List<User> findByActiveTrue();
    List<User> findByActiveFalse();
    
    // Ordering
    List<User> findByStatusOrderByCreatedAtDesc(Status status);
    
    // Limiting results
    User findFirstByOrderByCreatedAtDesc();
    List<User> findTop10ByStatusOrderByScoreDesc(Status status);
    
    // Distinct
    List<User> findDistinctByDepartment(String department);
    
    // Counting & existence
    long countByStatus(Status status);
    boolean existsByEmail(String email);
    
    // Delete
    void deleteByStatus(Status status);
    long deleteByLastLoginBefore(LocalDateTime date);
}
```

### 2. @Query Annotation (JPQL & Native)

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // JPQL query
    @Query("SELECT o FROM Order o WHERE o.status = :status AND o.createdAt > :date")
    List<Order> findRecentOrdersByStatus(
            @Param("status") OrderStatus status,
            @Param("date") LocalDateTime date);

    // JPQL with JOIN FETCH (eager loading)
    @Query("SELECT o FROM Order o " +
           "LEFT JOIN FETCH o.items " +
           "LEFT JOIN FETCH o.customer " +
           "WHERE o.id = :id")
    Optional<Order> findByIdWithDetails(@Param("id") Long id);

    // Native SQL query
    @Query(value = "SELECT * FROM orders WHERE status = ?1 LIMIT ?2", 
           nativeQuery = true)
    List<Order> findTopOrdersByStatus(String status, int limit);

    // Native query with pagination
    @Query(value = "SELECT * FROM orders WHERE customer_id = :customerId",
           countQuery = "SELECT COUNT(*) FROM orders WHERE customer_id = :customerId",
           nativeQuery = true)
    Page<Order> findByCustomerId(@Param("customerId") Long customerId, Pageable pageable);

    // Modifying queries (UPDATE/DELETE)
    @Modifying
    @Query("UPDATE Order o SET o.status = :status WHERE o.id IN :ids")
    int updateStatusForOrders(@Param("status") OrderStatus status, 
                              @Param("ids") List<Long> ids);

    @Modifying
    @Query("DELETE FROM Order o WHERE o.status = 'CANCELLED' AND o.createdAt < :date")
    int deleteCancelledOrdersBefore(@Param("date") LocalDateTime date);

    // SpEL expressions
    @Query("SELECT o FROM #{#entityName} o WHERE o.createdBy = ?#{principal.username}")
    List<Order> findCurrentUserOrders();
}
```

### 3. Custom Repository Implementation

```java
// Step 1: Define custom interface
public interface OrderRepositoryCustom {
    List<Order> findOrdersWithComplexCriteria(OrderSearchCriteria criteria);
    void batchInsertOrders(List<Order> orders);
}

// Step 2: Implement custom interface
@Repository
public class OrderRepositoryImpl implements OrderRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<Order> findOrdersWithComplexCriteria(OrderSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Order> query = cb.createQuery(Order.class);
        Root<Order> order = query.from(Order.class);

        List<Predicate> predicates = new ArrayList<>();

        if (criteria.getStatus() != null) {
            predicates.add(cb.equal(order.get("status"), criteria.getStatus()));
        }
        if (criteria.getMinAmount() != null) {
            predicates.add(cb.greaterThanOrEqualTo(order.get("totalAmount"), 
                    criteria.getMinAmount()));
        }
        if (criteria.getCustomerId() != null) {
            predicates.add(cb.equal(order.get("customer").get("id"), 
                    criteria.getCustomerId()));
        }

        query.where(predicates.toArray(new Predicate[0]));
        query.orderBy(cb.desc(order.get("createdAt")));

        return entityManager.createQuery(query)
                .setMaxResults(criteria.getLimit())
                .getResultList();
    }

    @Override
    @Transactional
    public void batchInsertOrders(List<Order> orders) {
        int batchSize = 50;
        for (int i = 0; i < orders.size(); i++) {
            entityManager.persist(orders.get(i));
            if (i > 0 && i % batchSize == 0) {
                entityManager.flush();
                entityManager.clear();
            }
        }
        entityManager.flush();
        entityManager.clear();
    }
}

// Step 3: Extend both interfaces
public interface OrderRepository extends JpaRepository<Order, Long>, 
                                         OrderRepositoryCustom {
    // Query methods from JpaRepository + custom methods
}
```

---

## Projections (Interface, Class, Dynamic)

### Why Projections?

- **Performance**: Fetch only needed columns
- **Security**: Don't expose sensitive fields
- **DTO mapping**: Transform data for API responses

### 1. Interface-Based Projections (Closed)

```java
// Projection interface
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
    
    // Nested projection
    DepartmentInfo getDepartment();
    
    interface DepartmentInfo {
        String getName();
        String getCode();
    }
}

// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Returns only id, name, email, department.name, department.code
    List<UserSummary> findByStatus(Status status);
    
    // With pagination
    Page<UserSummary> findByDepartmentId(Long deptId, Pageable pageable);
}

// Usage
List<UserSummary> users = userRepository.findByStatus(Status.ACTIVE);
users.forEach(u -> {
    System.out.println(u.getName());  // Works
    // u.getPassword();  // Not available - not in projection!
});
```

### 2. Open Projections (Computed Properties)

```java
public interface UserDetails {
    String getFirstName();
    String getLastName();
    
    // Computed property using SpEL
    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
    
    // Call a method
    @Value("#{target.getAge()}")
    int getAge();
    
    // Access bean
    @Value("#{@userFormatter.format(target)}")
    String getFormattedUser();
}
```

### 3. Class-Based Projections (DTOs)

```java
// DTO class
public class UserDTO {
    private final Long id;
    private final String name;
    private final String email;
    private final String departmentName;

    // Constructor must match SELECT clause order!
    public UserDTO(Long id, String name, String email, String departmentName) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.departmentName = departmentName;
    }

    // Getters only (immutable)
    public Long getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public String getDepartmentName() { return departmentName; }
}

// Repository with constructor expression
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query("SELECT new com.example.dto.UserDTO(u.id, u.name, u.email, u.department.name) " +
           "FROM User u WHERE u.status = :status")
    List<UserDTO> findUserDTOsByStatus(@Param("status") Status status);
}

// Or with records (Java 16+)
public record UserRecord(Long id, String name, String email) {}

@Query("SELECT new com.example.dto.UserRecord(u.id, u.name, u.email) FROM User u")
List<UserRecord> findAllAsRecords();
```

### 4. Dynamic Projections

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Generic projection method
    <T> List<T> findByStatus(Status status, Class<T> type);
    
    <T> Optional<T> findById(Long id, Class<T> type);
}

// Usage - choose projection at runtime
List<UserSummary> summaries = userRepository.findByStatus(Status.ACTIVE, UserSummary.class);
List<UserDTO> dtos = userRepository.findByStatus(Status.ACTIVE, UserDTO.class);
List<User> entities = userRepository.findByStatus(Status.ACTIVE, User.class);
```

---

## Specifications & Criteria API

### JPA Specification Pattern

Specifications enable **composable, reusable query predicates**.

```java
// Entity
@Entity
public class Product {
    @Id
    private Long id;
    private String name;
    private String category;
    private BigDecimal price;
    private boolean inStock;
    private LocalDateTime createdAt;
    
    @ManyToOne
    private Brand brand;
}

// Specification class
public class ProductSpecifications {

    public static Specification<Product> hasCategory(String category) {
        return (root, query, cb) -> 
            category == null ? null : cb.equal(root.get("category"), category);
    }

    public static Specification<Product> hasPriceBetween(BigDecimal min, BigDecimal max) {
        return (root, query, cb) -> {
            if (min == null && max == null) return null;
            if (min == null) return cb.lessThanOrEqualTo(root.get("price"), max);
            if (max == null) return cb.greaterThanOrEqualTo(root.get("price"), min);
            return cb.between(root.get("price"), min, max);
        };
    }

    public static Specification<Product> isInStock() {
        return (root, query, cb) -> cb.isTrue(root.get("inStock"));
    }

    public static Specification<Product> hasNameLike(String name) {
        return (root, query, cb) -> 
            name == null ? null : cb.like(cb.lower(root.get("name")), 
                    "%" + name.toLowerCase() + "%");
    }

    public static Specification<Product> hasBrandIn(List<String> brands) {
        return (root, query, cb) -> {
            if (brands == null || brands.isEmpty()) return null;
            return root.get("brand").get("name").in(brands);
        };
    }

    public static Specification<Product> createdAfter(LocalDateTime date) {
        return (root, query, cb) ->
            date == null ? null : cb.greaterThan(root.get("createdAt"), date);
    }
}

// Repository
public interface ProductRepository extends JpaRepository<Product, Long>,
                                           JpaSpecificationExecutor<Product> {
}

// Service - Compose specifications dynamically
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Page<Product> searchProducts(ProductSearchRequest request, Pageable pageable) {
        
        Specification<Product> spec = Specification
            .where(ProductSpecifications.hasCategory(request.getCategory()))
            .and(ProductSpecifications.hasPriceBetween(request.getMinPrice(), request.getMaxPrice()))
            .and(ProductSpecifications.hasNameLike(request.getSearchTerm()))
            .and(ProductSpecifications.hasBrandIn(request.getBrands()));

        if (request.isOnlyInStock()) {
            spec = spec.and(ProductSpecifications.isInStock());
        }

        return productRepository.findAll(spec, pageable);
    }
}
```

### Advanced Specifications with Joins

```java
public class OrderSpecifications {

    // Join to related entity
    public static Specification<Order> hasCustomerName(String customerName) {
        return (root, query, cb) -> {
            if (customerName == null) return null;
            Join<Order, Customer> customerJoin = root.join("customer");
            return cb.like(cb.lower(customerJoin.get("name")), 
                    "%" + customerName.toLowerCase() + "%");
        };
    }

    // Subquery
    public static Specification<Order> hasItemWithProduct(Long productId) {
        return (root, query, cb) -> {
            if (productId == null) return null;
            
            Subquery<Long> subquery = query.subquery(Long.class);
            Root<OrderItem> itemRoot = subquery.from(OrderItem.class);
            subquery.select(itemRoot.get("order").get("id"))
                    .where(cb.equal(itemRoot.get("product").get("id"), productId));
            
            return root.get("id").in(subquery);
        };
    }

    // Aggregate in specification
    public static Specification<Order> hasTotalGreaterThan(BigDecimal minTotal) {
        return (root, query, cb) -> {
            if (minTotal == null) return null;
            
            // For aggregate, need to use subquery or handle in service layer
            return cb.greaterThan(root.get("totalAmount"), minTotal);
        };
    }

    // Fetch join to avoid N+1
    public static Specification<Order> withItems() {
        return (root, query, cb) -> {
            if (Long.class != query.getResultType()) {
                // Don't fetch for count queries
                root.fetch("items", JoinType.LEFT);
            }
            return null; // No filtering, just fetching
        };
    }
}
```

---

## Auditing & Entity Listeners

### 1. Spring Data JPA Auditing

```java
// Enable auditing
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            if (authentication == null || !authentication.isAuthenticated()) {
                return Optional.of("SYSTEM");
            }
            return Optional.of(authentication.getName());
        };
    }
}

// Base auditable entity
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @CreatedBy
    @Column(name = "created_by", nullable = false, updatable = false)
    private String createdBy;

    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;

    @Version
    private Long version;

    // Getters and setters
}

// Entity extending base
@Entity
@Table(name = "orders")
public class Order extends BaseEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String orderNumber;
    private BigDecimal totalAmount;
    
    // Other fields...
}
```

### 2. Custom Entity Listeners

```java
// Custom listener
public class OrderEntityListener {

    @PrePersist
    public void prePersist(Order order) {
        if (order.getOrderNumber() == null) {
            order.setOrderNumber(generateOrderNumber());
        }
        order.setStatus(OrderStatus.PENDING);
    }

    @PreUpdate
    public void preUpdate(Order order) {
        order.setLastModifiedAt(LocalDateTime.now());
    }

    @PostPersist
    public void postPersist(Order order) {
        // Publish event, send notification, etc.
        ApplicationContextProvider.getBean(EventPublisher.class)
                .publish(new OrderCreatedEvent(order));
    }

    @PostLoad
    public void postLoad(Order order) {
        // Initialize transient fields
        order.initializeCalculatedFields();
    }

    @PreRemove
    public void preRemove(Order order) {
        // Validate deletion is allowed
        if (order.getStatus() == OrderStatus.SHIPPED) {
            throw new IllegalStateException("Cannot delete shipped orders");
        }
    }

    private String generateOrderNumber() {
        return "ORD-" + System.currentTimeMillis();
    }
}

// Apply listener to entity
@Entity
@EntityListeners({AuditingEntityListener.class, OrderEntityListener.class})
public class Order extends BaseEntity {
    // ...
}
```

### 3. Soft Delete Pattern

```java
@MappedSuperclass
public abstract class SoftDeletableEntity extends BaseEntity {

    @Column(name = "deleted_at")
    private LocalDateTime deletedAt;

    @Column(name = "deleted_by")
    private String deletedBy;

    public boolean isDeleted() {
        return deletedAt != null;
    }

    public void softDelete(String deletedBy) {
        this.deletedAt = LocalDateTime.now();
        this.deletedBy = deletedBy;
    }
}

// Repository with soft delete filter
public interface UserRepository extends JpaRepository<User, Long> {

    // Only active users
    @Query("SELECT u FROM User u WHERE u.deletedAt IS NULL")
    List<User> findAllActive();

    // Override default methods
    @Override
    @Query("SELECT u FROM User u WHERE u.id = :id AND u.deletedAt IS NULL")
    Optional<User> findById(@Param("id") Long id);

    // Include deleted
    @Query("SELECT u FROM User u WHERE u.id = :id")
    Optional<User> findByIdIncludingDeleted(@Param("id") Long id);
}

// Or use Hibernate @Where (global filter)
@Entity
@Where(clause = "deleted_at IS NULL")
@SQLDelete(sql = "UPDATE users SET deleted_at = NOW() WHERE id = ?")
public class User extends SoftDeletableEntity {
    // Soft delete automatically applied to all queries
}
```

---

## Transaction Management Deep Dive

### Transaction Propagation Types

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    // REQUIRED (Default) - Join existing or create new
    @Transactional(propagation = Propagation.REQUIRED)
    public void createOrder(OrderRequest request) {
        // Runs in same transaction as caller (if exists)
        // Or creates new transaction
    }

    // REQUIRES_NEW - Always create new, suspend existing
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrderAudit(Order order) {
        // Commits independently - even if outer transaction fails
        auditRepository.save(new AuditLog(order));
    }

    // NESTED - Create savepoint within existing transaction
    @Transactional(propagation = Propagation.NESTED)
    public void processPayment(Order order) {
        // Can rollback to savepoint without rolling back outer transaction
        paymentService.charge(order);
    }

    // MANDATORY - Must run within existing transaction
    @Transactional(propagation = Propagation.MANDATORY)
    public void updateInventory(Order order) {
        // Throws exception if no transaction exists
    }

    // SUPPORTS - Use transaction if exists, otherwise non-transactional
    @Transactional(propagation = Propagation.SUPPORTS, readOnly = true)
    public Order getOrder(Long id) {
        return orderRepository.findById(id).orElseThrow();
    }

    // NOT_SUPPORTED - Suspend existing, run non-transactional
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void sendNotification(Order order) {
        // External API call - don't hold transaction
        emailService.send(order);
    }

    // NEVER - Must NOT run within transaction
    @Transactional(propagation = Propagation.NEVER)
    public void generateReport() {
        // Throws if transaction exists
    }
}
```

### Transaction Isolation Levels

```java
@Service
public class AccountService {

    // READ_UNCOMMITTED - Can read uncommitted changes (dirty reads)
    @Transactional(isolation = Isolation.READ_UNCOMMITTED)
    public BigDecimal getBalanceQuick(Long accountId) {
        // Fastest, but may read uncommitted data
        return accountRepository.findBalance(accountId);
    }

    // READ_COMMITTED (Default for most DBs) - Only read committed data
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public BigDecimal getBalance(Long accountId) {
        // No dirty reads, but non-repeatable reads possible
        return accountRepository.findBalance(accountId);
    }

    // REPEATABLE_READ - Same query returns same results within transaction
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        // Prevents non-repeatable reads
        // But phantom reads possible
    }

    // SERIALIZABLE - Highest isolation, like serial execution
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void criticalFinancialOperation() {
        // Full isolation - slowest, may cause deadlocks
    }
}
```

### Rollback Configuration

```java
@Service
public class PaymentService {

    // Rollback on specific exceptions
    @Transactional(rollbackFor = PaymentException.class)
    public void processPayment(Payment payment) {
        // Rolls back on PaymentException (checked exception)
    }

    // Don't rollback on specific exceptions
    @Transactional(noRollbackFor = {ValidationException.class})
    public void validateAndProcess(Payment payment) {
        // ValidationException won't cause rollback
    }

    // Rollback on any exception
    @Transactional(rollbackFor = Exception.class)
    public void criticalOperation() {
        // Rolls back on any exception including checked
    }

    // Programmatic rollback
    @Transactional
    public void manualRollback() {
        try {
            // ... operations
        } catch (Exception e) {
            TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
            throw e;
        }
    }
}
```

### Common Transaction Pitfalls

```java
@Service
public class UserService {

    // PITFALL 1: Self-invocation bypasses proxy
    @Transactional
    public void createUser(UserRequest request) {
        User user = saveUser(request);
        sendWelcomeEmail(user);  // @Transactional ignored! Same class call.
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendWelcomeEmail(User user) {
        // This @Transactional is IGNORED when called from createUser()
        // Because it bypasses the proxy
    }

    // FIX: Inject self or use separate service
    @Autowired
    private UserService self;  // Inject proxy

    @Transactional
    public void createUserFixed(UserRequest request) {
        User user = saveUser(request);
        self.sendWelcomeEmail(user);  // Now goes through proxy
    }

    // PITFALL 2: Catching exception prevents rollback
    @Transactional
    public void createWithCatch() {
        try {
            riskyOperation();
        } catch (RuntimeException e) {
            log.error("Error", e);
            // Transaction NOT rolled back - exception was caught!
        }
    }

    // FIX: Re-throw or mark for rollback
    @Transactional
    public void createWithCatchFixed() {
        try {
            riskyOperation();
        } catch (RuntimeException e) {
            log.error("Error", e);
            throw e;  // Re-throw to trigger rollback
        }
    }

    // PITFALL 3: @Transactional on private method (ignored)
    @Transactional  // IGNORED - Spring AOP can't proxy private methods
    private void privateMethod() {
    }
}
```

---

## Connection Pooling (HikariCP)

### HikariCP Configuration

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver
    
    hikari:
      # Pool sizing
      maximum-pool-size: 20          # Max connections in pool
      minimum-idle: 10               # Min idle connections maintained
      
      # Timeouts
      connection-timeout: 30000      # Max wait for connection (30s)
      idle-timeout: 600000           # Max idle time before eviction (10min)
      max-lifetime: 1800000          # Max connection lifetime (30min)
      
      # Validation
      validation-timeout: 5000       # Max time for connection validation
      
      # Leak detection
      leak-detection-threshold: 60000  # Log warning if connection held > 60s
      
      # Performance
      auto-commit: false             # Better performance with explicit transactions
      
      # Pool name (for monitoring)
      pool-name: MyAppHikariPool
      
      # Register MBeans for JMX monitoring
      register-mbeans: true
```

### Calculating Pool Size

**Formula:** `connections = (core_count * 2) + effective_spindle_count`

For SSD with 4 cores: `(4 * 2) + 1 = 9` connections per instance

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties("spring.datasource.hikari")
    public HikariConfig hikariConfig() {
        HikariConfig config = new HikariConfig();
        
        // Dynamic pool size based on CPU cores
        int cores = Runtime.getRuntime().availableProcessors();
        int poolSize = (cores * 2) + 1;
        
        config.setMaximumPoolSize(Math.max(poolSize, 10));
        config.setMinimumIdle(Math.max(poolSize / 2, 5));
        
        return config;
    }

    @Bean
    public DataSource dataSource(HikariConfig config) {
        return new HikariDataSource(config);
    }
}
```

### Monitoring Pool Metrics

```java
@Component
public class HikariMetricsLogger {

    @Autowired
    private DataSource dataSource;

    @Scheduled(fixedDelay = 30000)
    public void logPoolMetrics() {
        if (dataSource instanceof HikariDataSource) {
            HikariDataSource hikariDS = (HikariDataSource) dataSource;
            HikariPoolMXBean poolMXBean = hikariDS.getHikariPoolMXBean();

            log.info("HikariCP Metrics - " +
                    "Active: {}, Idle: {}, Total: {}, Waiting: {}",
                    poolMXBean.getActiveConnections(),
                    poolMXBean.getIdleConnections(),
                    poolMXBean.getTotalConnections(),
                    poolMXBean.getThreadsAwaitingConnection());

            // Alert if pool is exhausted
            if (poolMXBean.getThreadsAwaitingConnection() > 0) {
                log.warn("Threads waiting for connection! Consider increasing pool size.");
            }
        }
    }
}
```

---

## Database Migrations (Flyway/Liquibase)

### Flyway Setup & Usage

```yaml
# application.yml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true       # Create baseline for existing DBs
    validate-on-migrate: true       # Validate migrations before running
    out-of-order: false             # Don't allow out-of-order migrations
    clean-disabled: true            # Disable clean in production!
```

**Migration file naming:** `V{version}__{description}.sql`

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'ACTIVE',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);

-- V2__add_department_to_users.sql
ALTER TABLE users ADD COLUMN department_id BIGINT;

ALTER TABLE users ADD CONSTRAINT fk_users_department 
    FOREIGN KEY (department_id) REFERENCES departments(id);

-- V3__add_soft_delete.sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP;
ALTER TABLE users ADD COLUMN deleted_by VARCHAR(255);

CREATE INDEX idx_users_deleted_at ON users(deleted_at) WHERE deleted_at IS NULL;

-- R__seed_data.sql (Repeatable migration - runs every time checksum changes)
INSERT INTO roles (name, description) VALUES ('ADMIN', 'Administrator')
ON CONFLICT (name) DO UPDATE SET description = EXCLUDED.description;
```

### Liquibase Setup & Usage

```yaml
# application.yml
spring:
  liquibase:
    enabled: true
    change-log: classpath:db/changelog/db.changelog-master.xml
```

**Master changelog:**
```xml
<!-- db/changelog/db.changelog-master.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <include file="db/changelog/changes/001-create-users.xml"/>
    <include file="db/changelog/changes/002-add-orders.xml"/>
</databaseChangeLog>
```

**Individual changeset:**
```xml
<!-- db/changelog/changes/001-create-users.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="1" author="developer">
        <createTable tableName="users">
            <column name="id" type="BIGINT" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="email" type="VARCHAR(255)">
                <constraints nullable="false" unique="true"/>
            </column>
            <column name="name" type="VARCHAR(255)">
                <constraints nullable="false"/>
            </column>
            <column name="created_at" type="TIMESTAMP" defaultValueComputed="CURRENT_TIMESTAMP">
                <constraints nullable="false"/>
            </column>
        </createTable>
        
        <createIndex tableName="users" indexName="idx_users_email">
            <column name="email"/>
        </createIndex>
        
        <rollback>
            <dropTable tableName="users"/>
        </rollback>
    </changeSet>

    <changeSet id="2" author="developer" context="!prod">
        <!-- Only runs in non-prod environments -->
        <insert tableName="users">
            <column name="email" value="test@example.com"/>
            <column name="name" value="Test User"/>
        </insert>
    </changeSet>
</databaseChangeLog>
```

---

## Performance Optimization

### 1. Batch Processing

```java
@Repository
public class BatchRepository {

    @PersistenceContext
    private EntityManager entityManager;

    // Batch insert
    @Transactional
    public void batchInsert(List<User> users, int batchSize) {
        for (int i = 0; i < users.size(); i++) {
            entityManager.persist(users.get(i));
            
            if (i > 0 && i % batchSize == 0) {
                entityManager.flush();
                entityManager.clear();  // Detach entities to free memory
            }
        }
        entityManager.flush();
        entityManager.clear();
    }

    // Batch update with JPQL
    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id IN :ids")
    int batchUpdateStatus(@Param("status") Status status, @Param("ids") List<Long> ids);
}

// application.yml for batch optimization
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 50
          batch_versioned_data: true
        order_inserts: true
        order_updates: true
```

### 2. Read-Only Transactions

```java
@Service
public class ReportService {

    // Read-only optimization
    @Transactional(readOnly = true)
    public List<OrderSummary> generateReport(LocalDate from, LocalDate to) {
        // Hibernate skips dirty checking
        // Some DBs route to read replica
        return orderRepository.findOrderSummaries(from, to);
    }
}
```

### 3. Entity Graphs

```java
@Entity
@NamedEntityGraph(
    name = "Order.withItemsAndCustomer",
    attributeNodes = {
        @NamedAttributeNode("items"),
        @NamedAttributeNode(value = "customer", subgraph = "customer-subgraph")
    },
    subgraphs = {
        @NamedSubgraph(
            name = "customer-subgraph",
            attributeNodes = {@NamedAttributeNode("address")}
        )
    }
)
public class Order {
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Customer customer;
}

// Repository
public interface OrderRepository extends JpaRepository<Order, Long> {

    @EntityGraph(value = "Order.withItemsAndCustomer")
    Optional<Order> findById(Long id);

    // Or inline definition
    @EntityGraph(attributePaths = {"items", "customer", "customer.address"})
    List<Order> findByStatus(OrderStatus status);
}
```

### 4. Query Hints

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    // Cache query results (second-level cache)
    @QueryHints({
        @QueryHint(name = "org.hibernate.cacheable", value = "true"),
        @QueryHint(name = "org.hibernate.cacheRegion", value = "product-cache")
    })
    List<Product> findByCategory(String category);

    // Read-only hint (no dirty checking)
    @QueryHints(@QueryHint(name = "org.hibernate.readOnly", value = "true"))
    List<Product> findAllForDisplay();

    // Timeout hint
    @QueryHints(@QueryHint(name = "jakarta.persistence.query.timeout", value = "5000"))
    List<Product> findExpensiveQuery();
}
```

---

## Multi-Tenancy Patterns

### 1. Separate Schema Per Tenant

```java
@Configuration
public class MultiTenantConfig {

    @Bean
    public MultiTenantConnectionProvider multiTenantConnectionProvider(DataSource dataSource) {
        return new SchemaMultiTenantConnectionProvider(dataSource);
    }

    @Bean
    public CurrentTenantIdentifierResolver currentTenantIdentifierResolver() {
        return () -> TenantContext.getCurrentTenant();
    }
}

public class SchemaMultiTenantConnectionProvider implements MultiTenantConnectionProvider {

    private final DataSource dataSource;

    @Override
    public Connection getConnection(String tenantIdentifier) throws SQLException {
        Connection connection = dataSource.getConnection();
        connection.setSchema(tenantIdentifier);
        return connection;
    }

    @Override
    public void releaseConnection(String tenantIdentifier, Connection connection) throws SQLException {
        connection.setSchema("public");  // Reset to default
        connection.close();
    }
}

// Tenant context holder
public class TenantContext {
    private static final ThreadLocal<String> CURRENT_TENANT = new ThreadLocal<>();

    public static String getCurrentTenant() {
        return CURRENT_TENANT.get();
    }

    public static void setCurrentTenant(String tenant) {
        CURRENT_TENANT.set(tenant);
    }

    public static void clear() {
        CURRENT_TENANT.remove();
    }
}

// Filter to set tenant from request
@Component
public class TenantFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String tenantId = httpRequest.getHeader("X-Tenant-ID");
        
        try {
            TenantContext.setCurrentTenant(tenantId);
            chain.doFilter(request, response);
        } finally {
            TenantContext.clear();
        }
    }
}
```

### 2. Discriminator Column (Shared Table)

```java
@MappedSuperclass
@FilterDef(name = "tenantFilter", parameters = @ParamDef(name = "tenantId", type = String.class))
@Filter(name = "tenantFilter", condition = "tenant_id = :tenantId")
public abstract class TenantAwareEntity {

    @Column(name = "tenant_id", nullable = false, updatable = false)
    private String tenantId;

    @PrePersist
    public void prePersist() {
        this.tenantId = TenantContext.getCurrentTenant();
    }
}

@Entity
public class Order extends TenantAwareEntity {
    @Id
    private Long id;
    // Other fields...
}

// Enable filter in service
@Service
public class OrderService {

    @PersistenceContext
    private EntityManager entityManager;

    @PostConstruct
    public void enableTenantFilter() {
        Session session = entityManager.unwrap(Session.class);
        session.enableFilter("tenantFilter")
               .setParameter("tenantId", TenantContext.getCurrentTenant());
    }
}
```

---

## Common Pitfalls & Solutions

### Pitfall 1: LazyInitializationException

```java
// PROBLEM
@Entity
public class Order {
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
}

@Service
public class OrderService {
    public Order getOrder(Long id) {
        Order order = orderRepository.findById(id).orElseThrow();
        return order;  // items not loaded
    }
}

// Controller
order.getItems().size();  // LazyInitializationException! Session closed.

// SOLUTIONS

// Solution 1: JOIN FETCH
@Query("SELECT o FROM Order o LEFT JOIN FETCH o.items WHERE o.id = :id")
Optional<Order> findByIdWithItems(@Param("id") Long id);

// Solution 2: EntityGraph
@EntityGraph(attributePaths = {"items"})
Optional<Order> findById(Long id);

// Solution 3: Open Session in View (NOT recommended for APIs)
spring.jpa.open-in-view=true  # Default is true, but avoid in production

// Solution 4: DTO projection
@Query("SELECT new OrderDTO(o.id, o.status, SIZE(o.items)) FROM Order o WHERE o.id = :id")
Optional<OrderDTO> findOrderSummary(@Param("id") Long id);
```

### Pitfall 2: N+1 Query Problem

```java
// PROBLEM
List<Order> orders = orderRepository.findAll();
for (Order order : orders) {
    order.getCustomer().getName();  // N additional queries!
}

// SOLUTION: Batch fetching
@Entity
public class Order {
    @ManyToOne(fetch = FetchType.LAZY)
    @BatchSize(size = 20)  // Load customers in batches of 20
    private Customer customer;
}

// Or global setting
spring.jpa.properties.hibernate.default_batch_fetch_size=20

// Or JOIN FETCH
@Query("SELECT o FROM Order o JOIN FETCH o.customer")
List<Order> findAllWithCustomer();
```

### Pitfall 3: Detached Entity Passed to Persist

```java
// PROBLEM
User user = new User();
user.setId(1L);  // Setting ID makes it "detached"
userRepository.save(user);  // Tries SELECT then INSERT - fails if exists

// SOLUTION: Use merge for detached entities
@Transactional
public User saveOrUpdate(User user) {
    if (user.getId() == null) {
        entityManager.persist(user);
        return user;
    } else {
        return entityManager.merge(user);
    }
}
```

---

## Interview Questions & Answers

### Q1: Explain the difference between `findById()` and `getById()` (getReferenceById)?

**Answer:**

| Method | Behavior | Use Case |
|--------|----------|----------|
| `findById(id)` | Executes SELECT immediately, returns `Optional<T>` | When you need the entity data |
| `getReferenceById(id)` | Returns proxy, no DB hit until property accessed | When you only need reference (e.g., setting FK) |

```java
// findById - Immediate database query
Optional<User> user = userRepository.findById(1L);  // SELECT * FROM users WHERE id = 1

// getReferenceById - Returns proxy, no query yet
User userRef = userRepository.getReferenceById(1L);  // No query!
order.setUser(userRef);  // Still no query - just sets FK
orderRepository.save(order);  // Only order INSERT, no user SELECT

// But if you access a property...
userRef.getName();  // NOW it queries the database
```

**Use `getReferenceById` when:**
- Setting foreign key relationships
- You don't need entity data, just the reference
- Performance optimization

---

### Q2: How do you handle optimistic vs pessimistic locking?

**Answer:**

**Optimistic Locking** (preferred for low contention):
```java
@Entity
public class Product {
    @Id
    private Long id;
    
    @Version
    private Long version;  // Auto-incremented on update
    
    private int stock;
}

// On concurrent update:
// User A reads product (version=1)
// User B reads product (version=1)
// User A updates stock (version becomes 2)
// User B tries to update → OptimisticLockException (version mismatch)
```

**Pessimistic Locking** (for high contention):
```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdForUpdate(@Param("id") Long id);

    @Lock(LockModeType.PESSIMISTIC_READ)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdWithSharedLock(@Param("id") Long id);
}

// Usage
@Transactional
public void decrementStock(Long productId) {
    Product product = productRepository.findByIdForUpdate(productId)
            .orElseThrow();  // Locks row until transaction commits
    product.setStock(product.getStock() - 1);
}
```

**When to use:**
- **Optimistic**: Read-heavy, rare conflicts, better throughput
- **Pessimistic**: Write-heavy, frequent conflicts, data integrity critical

---

### Q3: What's the difference between JPQL, Criteria API, and native queries?

**Answer:**

| Aspect | JPQL | Criteria API | Native SQL |
|--------|------|--------------|------------|
| **Type Safety** | No (string-based) | Yes (compile-time) | No |
| **Dynamic Queries** | Concatenation (risky) | Built programmatically | Concatenation |
| **Database Portable** | Yes | Yes | No |
| **Complex Queries** | Limited | Full power | Full power |
| **Readability** | High | Lower | Depends |

```java
// JPQL - Simple, readable
@Query("SELECT u FROM User u WHERE u.status = :status AND u.age > :age")
List<User> findByStatusAndAge(@Param("status") Status status, @Param("age") int age);

// Criteria API - Type-safe, dynamic
public List<User> findUsers(UserSearchCriteria criteria) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> user = query.from(User.class);
    
    List<Predicate> predicates = new ArrayList<>();
    if (criteria.getStatus() != null) {
        predicates.add(cb.equal(user.get("status"), criteria.getStatus()));
    }
    if (criteria.getMinAge() != null) {
        predicates.add(cb.greaterThan(user.get("age"), criteria.getMinAge()));
    }
    
    query.where(predicates.toArray(new Predicate[0]));
    return em.createQuery(query).getResultList();
}

// Native SQL - Full database power
@Query(value = "SELECT * FROM users WHERE status = ?1 FOR UPDATE SKIP LOCKED", 
       nativeQuery = true)
List<User> findForProcessing(String status);
```

---

### Q4: How do you optimize a slow JPA query?

**Answer:**

**Step-by-step optimization:**

1. **Enable SQL logging to identify the problem:**
```yaml
spring.jpa.show-sql=true
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

2. **Fix N+1 with JOIN FETCH or EntityGraph:**
```java
@EntityGraph(attributePaths = {"items", "customer"})
List<Order> findByStatus(OrderStatus status);
```

3. **Use projections instead of full entities:**
```java
@Query("SELECT new OrderDTO(o.id, o.total, c.name) FROM Order o JOIN o.customer c")
List<OrderDTO> findOrderSummaries();
```

4. **Add database indexes:**
```java
@Table(indexes = {
    @Index(name = "idx_order_status", columnList = "status"),
    @Index(name = "idx_order_customer", columnList = "customer_id, status")
})
```

5. **Use read-only transactions:**
```java
@Transactional(readOnly = true)
public List<Order> getOrders() { ... }
```

6. **Batch fetching for collections:**
```java
@BatchSize(size = 20)
private List<OrderItem> items;
```

7. **Pagination for large result sets:**
```java
Page<Order> findByStatus(OrderStatus status, Pageable pageable);
```

---

### Q5: Explain `@Transactional` proxy behavior and its limitations.

**Answer:**

Spring `@Transactional` works via **AOP proxy**:

```
Client → Proxy → Target Bean
         ↓
    Begin Transaction
         ↓
    Call actual method
         ↓
    Commit/Rollback
```

**Limitations:**

1. **Self-invocation bypasses proxy:**
```java
@Service
public class OrderService {
    @Transactional
    public void createOrder() {
        // ...
        this.sendNotification();  // BYPASSES PROXY! @Transactional ignored
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendNotification() { }
}

// FIX: Inject self
@Autowired
private OrderService self;

public void createOrder() {
    self.sendNotification();  // Goes through proxy
}
```

2. **Only works on public methods:**
```java
@Transactional  // IGNORED on private/protected
private void internalMethod() { }
```

3. **Only works on Spring-managed beans:**
```java
new OrderService().createOrder();  // No proxy, no transaction!
```

4. **Checked exceptions don't rollback by default:**
```java
@Transactional  // Won't rollback on IOException
public void process() throws IOException { }

@Transactional(rollbackFor = Exception.class)  // FIX
public void process() throws IOException { }
```

---

## Summary: Best Practices Checklist

✅ **Repository Design**
- Use query derivation for simple queries
- Use `@Query` for complex JPQL
- Implement custom repository for dynamic queries
- Use Specifications for composable filters

✅ **Performance**
- Always use projections when full entity not needed
- Fix N+1 with JOIN FETCH or EntityGraph
- Use `@Transactional(readOnly = true)` for reads
- Configure batch fetching globally
- Add appropriate database indexes

✅ **Transactions**
- Understand propagation types
- Use `REQUIRES_NEW` for independent operations (audit logs)
- Don't catch exceptions without re-throwing
- Avoid self-invocation issues

✅ **Connection Pool**
- Size pool based on CPU cores
- Monitor active/idle connections
- Enable leak detection in dev
- Set appropriate timeouts

✅ **Migrations**
- Use Flyway or Liquibase
- Never modify existing migrations
- Test migrations in CI/CD
- Disable clean in production

---

**Key Takeaway:** Spring Data JPA is powerful but has many pitfalls. Understanding the proxy mechanism, lazy loading, and transaction management is crucial for building performant, reliable applications.
