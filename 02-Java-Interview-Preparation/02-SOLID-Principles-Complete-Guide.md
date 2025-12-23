# SOLID Principles - Complete Interview Guide

## Table of Contents

1. [Introduction to SOLID](#introduction-to-solid)
2. [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
3. [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
4. [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
5. [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
6. [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
7. [Real-World Examples](#real-world-examples)
8. [Interview Questions](#interview-questions)

---

## Introduction to SOLID

### What is SOLID?

**SOLID** is an acronym for five object-oriented design principles introduced by Robert C. Martin (Uncle Bob). These principles help create software that is:

- **Maintainable** - Easy to modify and extend
- **Flexible** - Adapts to changing requirements
- **Testable** - Easy to write unit tests
- **Understandable** - Clear and readable code

### SOLID Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        SOLID PRINCIPLES                          │
├─────────────────────────────────────────────────────────────────┤
│  S  │ Single Responsibility  │ One class, one reason to change │
├─────┼────────────────────────┼──────────────────────────────────┤
│  O  │ Open/Closed            │ Open for extension, closed for  │
│     │                        │ modification                     │
├─────┼────────────────────────┼──────────────────────────────────┤
│  L  │ Liskov Substitution    │ Subtypes must be substitutable  │
│     │                        │ for their base types             │
├─────┼────────────────────────┼──────────────────────────────────┤
│  I  │ Interface Segregation  │ Many specific interfaces better │
│     │                        │ than one general interface       │
├─────┼────────────────────────┼──────────────────────────────────┤
│  D  │ Dependency Inversion   │ Depend on abstractions, not     │
│     │                        │ on concretions                   │
└─────┴────────────────────────┴──────────────────────────────────┘
```

---

## Single Responsibility Principle (SRP)

### Definition

> "A class should have one, and only one, reason to change."

A class should have only one responsibility, meaning it should have only one job or purpose.

### Violation Example

```java
// BAD: Class has multiple responsibilities
public class Employee {
    private String name;
    private String email;
    private double salary;

    // Responsibility 1: Employee data management
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public double getSalary() { return salary; }
    public void setSalary(double salary) { this.salary = salary; }

    // Responsibility 2: Salary calculation (should be separate)
    public double calculateBonus() {
        return salary * 0.1;
    }

    public double calculateTax() {
        return salary * 0.3;
    }

    // Responsibility 3: Persistence (should be separate)
    public void saveToDatabase() {
        // Database connection and save logic
        System.out.println("Saving " + name + " to database");
    }

    public void loadFromDatabase(int id) {
        // Database connection and load logic
        System.out.println("Loading employee " + id + " from database");
    }

    // Responsibility 4: Reporting (should be separate)
    public String generateReport() {
        return "Employee Report: " + name + ", Salary: $" + salary;
    }

    public void printReport() {
        System.out.println(generateReport());
    }

    // Responsibility 5: Email notification (should be separate)
    public void sendEmail(String subject, String body) {
        // Email sending logic
        System.out.println("Sending email to " + email);
    }
}
```

### Correct Implementation

```java
// GOOD: Each class has a single responsibility

// Responsibility 1: Employee data (Entity/Model)
public class Employee {
    private Long id;
    private String name;
    private String email;
    private double salary;

    public Employee(String name, String email, double salary) {
        this.name = name;
        this.email = email;
        this.salary = salary;
    }

    // Only getters and setters - pure data holder
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public double getSalary() { return salary; }
    public void setSalary(double salary) { this.salary = salary; }
}

// Responsibility 2: Salary calculations
public class SalaryCalculator {
    public double calculateBonus(Employee employee) {
        return employee.getSalary() * 0.1;
    }

    public double calculateTax(Employee employee) {
        return employee.getSalary() * 0.3;
    }

    public double calculateNetSalary(Employee employee) {
        double bonus = calculateBonus(employee);
        double tax = calculateTax(employee);
        return employee.getSalary() + bonus - tax;
    }
}

// Responsibility 3: Persistence
public class EmployeeRepository {
    public void save(Employee employee) {
        // Database logic
        System.out.println("Saving " + employee.getName() + " to database");
    }

    public Employee findById(Long id) {
        // Database logic
        System.out.println("Loading employee " + id + " from database");
        return new Employee("John", "john@example.com", 50000);
    }

    public List<Employee> findAll() {
        // Database logic
        return new ArrayList<>();
    }

    public void delete(Employee employee) {
        // Database logic
    }
}

// Responsibility 4: Reporting
public class EmployeeReportGenerator {
    public String generateSummaryReport(Employee employee) {
        return String.format("Employee Report:\nName: %s\nEmail: %s\nSalary: $%.2f",
            employee.getName(), employee.getEmail(), employee.getSalary());
    }

    public String generateDetailedReport(Employee employee, SalaryCalculator calculator) {
        return String.format("""
            Detailed Employee Report:
            Name: %s
            Email: %s
            Gross Salary: $%.2f
            Bonus: $%.2f
            Tax: $%.2f
            Net Salary: $%.2f
            """,
            employee.getName(),
            employee.getEmail(),
            employee.getSalary(),
            calculator.calculateBonus(employee),
            calculator.calculateTax(employee),
            calculator.calculateNetSalary(employee)
        );
    }
}

// Responsibility 5: Email notifications
public class EmailService {
    public void sendEmail(String to, String subject, String body) {
        System.out.println("Sending email to " + to);
        System.out.println("Subject: " + subject);
        System.out.println("Body: " + body);
    }

    public void sendWelcomeEmail(Employee employee) {
        sendEmail(employee.getEmail(), "Welcome!",
            "Welcome to the company, " + employee.getName() + "!");
    }

    public void sendSalarySlip(Employee employee, String report) {
        sendEmail(employee.getEmail(), "Salary Slip", report);
    }
}

// Usage - composition of single-responsibility classes
public class EmployeeService {
    private EmployeeRepository repository;
    private SalaryCalculator calculator;
    private EmployeeReportGenerator reportGenerator;
    private EmailService emailService;

    public EmployeeService(EmployeeRepository repository,
                          SalaryCalculator calculator,
                          EmployeeReportGenerator reportGenerator,
                          EmailService emailService) {
        this.repository = repository;
        this.calculator = calculator;
        this.reportGenerator = reportGenerator;
        this.emailService = emailService;
    }

    public void processNewEmployee(Employee employee) {
        repository.save(employee);
        emailService.sendWelcomeEmail(employee);
    }

    public void sendMonthlySalarySlip(Employee employee) {
        String report = reportGenerator.generateDetailedReport(employee, calculator);
        emailService.sendSalarySlip(employee, report);
    }
}
```

### Benefits of SRP

1. **Easier testing** - Each class can be tested independently
2. **Lower coupling** - Changes in one area don't affect others
3. **Better organization** - Easy to find and understand code
4. **Reusability** - Single-purpose classes are more reusable
5. **Easier maintenance** - Fix bugs in isolation

---

## Open/Closed Principle (OCP)

### Definition

> "Software entities should be open for extension but closed for modification."

You should be able to add new functionality without changing existing code.

### Violation Example

```java
// BAD: Need to modify class to add new discount types
public class DiscountCalculator {

    public double calculateDiscount(Order order) {
        double discount = 0;

        // Adding new customer type requires modifying this method
        switch (order.getCustomerType()) {
            case "REGULAR":
                discount = order.getTotal() * 0.05;
                break;
            case "PREMIUM":
                discount = order.getTotal() * 0.10;
                break;
            case "VIP":
                discount = order.getTotal() * 0.20;
                break;
            // To add GOLD customer, must modify this class!
            case "GOLD":
                discount = order.getTotal() * 0.25;
                break;
        }

        return discount;
    }
}
```

### Correct Implementation

```java
// GOOD: Open for extension, closed for modification

// Abstraction
public interface DiscountStrategy {
    double calculateDiscount(Order order);
    boolean isApplicable(Order order);
}

// Extensions (new discount types don't require modifying existing code)
public class RegularCustomerDiscount implements DiscountStrategy {
    @Override
    public double calculateDiscount(Order order) {
        return order.getTotal() * 0.05;
    }

    @Override
    public boolean isApplicable(Order order) {
        return "REGULAR".equals(order.getCustomerType());
    }
}

public class PremiumCustomerDiscount implements DiscountStrategy {
    @Override
    public double calculateDiscount(Order order) {
        return order.getTotal() * 0.10;
    }

    @Override
    public boolean isApplicable(Order order) {
        return "PREMIUM".equals(order.getCustomerType());
    }
}

public class VIPCustomerDiscount implements DiscountStrategy {
    @Override
    public double calculateDiscount(Order order) {
        return order.getTotal() * 0.20;
    }

    @Override
    public boolean isApplicable(Order order) {
        return "VIP".equals(order.getCustomerType());
    }
}

// Adding new discount - no modification to existing code!
public class GoldCustomerDiscount implements DiscountStrategy {
    @Override
    public double calculateDiscount(Order order) {
        return order.getTotal() * 0.25;
    }

    @Override
    public boolean isApplicable(Order order) {
        return "GOLD".equals(order.getCustomerType());
    }
}

// Seasonal discount - extend without modifying!
public class SeasonalDiscount implements DiscountStrategy {
    @Override
    public double calculateDiscount(Order order) {
        return order.getTotal() * 0.15;
    }

    @Override
    public boolean isApplicable(Order order) {
        LocalDate now = LocalDate.now();
        return now.getMonth() == Month.DECEMBER;
    }
}

// Calculator is closed for modification
public class DiscountCalculator {
    private List<DiscountStrategy> discountStrategies;

    public DiscountCalculator(List<DiscountStrategy> strategies) {
        this.discountStrategies = strategies;
    }

    public double calculateDiscount(Order order) {
        return discountStrategies.stream()
            .filter(strategy -> strategy.isApplicable(order))
            .mapToDouble(strategy -> strategy.calculateDiscount(order))
            .max()
            .orElse(0.0);
    }
}

// Configuration (wiring)
public class Application {
    public static void main(String[] args) {
        List<DiscountStrategy> strategies = Arrays.asList(
            new RegularCustomerDiscount(),
            new PremiumCustomerDiscount(),
            new VIPCustomerDiscount(),
            new GoldCustomerDiscount(),
            new SeasonalDiscount()
        );

        DiscountCalculator calculator = new DiscountCalculator(strategies);

        Order order = new Order("VIP", 1000.0);
        double discount = calculator.calculateDiscount(order);
        System.out.println("Discount: $" + discount);
    }
}
```

### OCP with Inheritance

```java
// Base class - closed for modification
public abstract class Shape {
    public abstract double calculateArea();
}

// Extensions - open for extension
public class Rectangle extends Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return width * height;
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

// New shape - no modification to existing code!
public class Triangle extends Shape {
    private double base;
    private double height;

    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

// AreaCalculator works with any shape - closed for modification
public class AreaCalculator {
    public double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
            .mapToDouble(Shape::calculateArea)
            .sum();
    }
}
```

---

## Liskov Substitution Principle (LSP)

### Definition

> "Objects of a superclass should be replaceable with objects of its subclasses without breaking the application."

Derived classes must be substitutable for their base classes without altering the correctness of the program.

### Violation Example

```java
// BAD: Square is not a proper substitute for Rectangle

public class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getWidth() { return width; }
    public int getHeight() { return height; }

    public int calculateArea() {
        return width * height;
    }
}

// Violates LSP - changes behavior of setters
public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;  // Also sets height!
    }

    @Override
    public void setHeight(int height) {
        this.height = height;
        this.width = height;  // Also sets width!
    }
}

// Client code breaks with Square
public class AreaCalculatorTest {
    public static void main(String[] args) {
        Rectangle rectangle = new Rectangle();
        rectangle.setWidth(5);
        rectangle.setHeight(10);
        System.out.println("Area: " + rectangle.calculateArea()); // 50 ✓

        // Substituting with Square breaks expectations
        Rectangle square = new Square();  // LSP violation!
        square.setWidth(5);
        square.setHeight(10);
        // Expectation: 50, but actual: 100 (because width was also set to 10)
        System.out.println("Area: " + square.calculateArea()); // 100 ✗
    }
}
```

### Correct Implementation

```java
// GOOD: Proper abstraction respecting LSP

// Common abstraction
public interface Shape {
    double calculateArea();
}

// Rectangle - independent implementation
public class Rectangle implements Shape {
    private final int width;
    private final int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    public int getWidth() { return width; }
    public int getHeight() { return height; }

    @Override
    public double calculateArea() {
        return width * height;
    }
}

// Square - independent implementation (not extending Rectangle)
public class Square implements Shape {
    private final int side;

    public Square(int side) {
        this.side = side;
    }

    public int getSide() { return side; }

    @Override
    public double calculateArea() {
        return side * side;
    }
}

// Client code works correctly with any Shape
public class ShapeService {
    public double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
            .mapToDouble(Shape::calculateArea)
            .sum();
    }
}
```

### Another LSP Violation Example

```java
// BAD: Bird hierarchy violating LSP

public class Bird {
    public void fly() {
        System.out.println("Flying...");
    }

    public void eat() {
        System.out.println("Eating...");
    }
}

// Violates LSP - Ostrich can't fly!
public class Ostrich extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Ostriches can't fly!");
    }
}

// Client code breaks
public class BirdWatcher {
    public void watchBirdsFly(List<Bird> birds) {
        for (Bird bird : birds) {
            bird.fly();  // Throws exception for Ostrich!
        }
    }
}
```

### Correct Bird Hierarchy

```java
// GOOD: Proper abstraction

public interface Bird {
    void eat();
}

public interface FlyingBird extends Bird {
    void fly();
}

public interface SwimmingBird extends Bird {
    void swim();
}

public class Sparrow implements FlyingBird {
    @Override
    public void eat() {
        System.out.println("Sparrow eating seeds");
    }

    @Override
    public void fly() {
        System.out.println("Sparrow flying");
    }
}

public class Ostrich implements Bird {
    @Override
    public void eat() {
        System.out.println("Ostrich eating");
    }
    // No fly method - ostrich can't fly, and that's fine!
}

public class Penguin implements SwimmingBird {
    @Override
    public void eat() {
        System.out.println("Penguin eating fish");
    }

    @Override
    public void swim() {
        System.out.println("Penguin swimming");
    }
}

// Client code works correctly
public class BirdWatcher {
    public void watchBirdsFly(List<FlyingBird> birds) {
        for (FlyingBird bird : birds) {
            bird.fly();  // All birds here can fly!
        }
    }
}
```

### LSP Rules

1. **Signature Rule**: Subtype methods should have compatible signatures
2. **Precondition Rule**: Subtypes cannot strengthen preconditions
3. **Postcondition Rule**: Subtypes cannot weaken postconditions
4. **Invariant Rule**: Subtypes must maintain supertype invariants
5. **History Rule**: Subtypes cannot violate supertype history constraints

---

## Interface Segregation Principle (ISP)

### Definition

> "Clients should not be forced to depend on interfaces they do not use."

Many specific interfaces are better than one general-purpose interface.

### Violation Example

```java
// BAD: Fat interface with too many methods

public interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void writeReport();
    void manageTeam();
    void reviewCode();
}

// Human worker can do everything
public class HumanWorker implements Worker {
    @Override
    public void work() { System.out.println("Working..."); }

    @Override
    public void eat() { System.out.println("Eating..."); }

    @Override
    public void sleep() { System.out.println("Sleeping..."); }

    @Override
    public void attendMeeting() { System.out.println("In meeting..."); }

    @Override
    public void writeReport() { System.out.println("Writing report..."); }

    @Override
    public void manageTeam() { System.out.println("Managing team..."); }

    @Override
    public void reviewCode() { System.out.println("Reviewing code..."); }
}

// Robot forced to implement methods it doesn't need!
public class RobotWorker implements Worker {
    @Override
    public void work() { System.out.println("Robot working..."); }

    @Override
    public void eat() {
        throw new UnsupportedOperationException("Robots don't eat!");
    }

    @Override
    public void sleep() {
        throw new UnsupportedOperationException("Robots don't sleep!");
    }

    @Override
    public void attendMeeting() {
        throw new UnsupportedOperationException("Robots don't attend meetings!");
    }

    // ... more unnecessary methods
}
```

### Correct Implementation

```java
// GOOD: Segregated interfaces

// Core work interface
public interface Workable {
    void work();
}

// Biological needs interface
public interface Feedable {
    void eat();
    void sleep();
}

// Communication interface
public interface Communicable {
    void attendMeeting();
    void writeReport();
}

// Management interface
public interface Manageable {
    void manageTeam();
}

// Technical interface
public interface TechnicalReviewer {
    void reviewCode();
}

// Human worker implements what makes sense
public class Developer implements Workable, Feedable, Communicable, TechnicalReviewer {
    @Override
    public void work() { System.out.println("Coding..."); }

    @Override
    public void eat() { System.out.println("Eating lunch..."); }

    @Override
    public void sleep() { System.out.println("Sleeping..."); }

    @Override
    public void attendMeeting() { System.out.println("In standup..."); }

    @Override
    public void writeReport() { System.out.println("Writing PR description..."); }

    @Override
    public void reviewCode() { System.out.println("Reviewing PR..."); }
}

// Manager implements different set
public class Manager implements Workable, Feedable, Communicable, Manageable {
    @Override
    public void work() { System.out.println("Planning..."); }

    @Override
    public void eat() { System.out.println("Business lunch..."); }

    @Override
    public void sleep() { System.out.println("Sleeping..."); }

    @Override
    public void attendMeeting() { System.out.println("Leading meeting..."); }

    @Override
    public void writeReport() { System.out.println("Writing status report..."); }

    @Override
    public void manageTeam() { System.out.println("Managing team..."); }
}

// Robot implements only what it can do
public class RobotWorker implements Workable {
    @Override
    public void work() { System.out.println("Robot assembling parts..."); }
}

// Client code uses only what it needs
public class WorkScheduler {
    public void scheduleWork(List<Workable> workers) {
        workers.forEach(Workable::work);
    }
}

public class CafeteriaService {
    public void serveMeals(List<Feedable> feedables) {
        feedables.forEach(Feedable::eat);
    }
}
```

### Another Example: Printer Interface

```java
// BAD: One fat interface
public interface MultiFunctionDevice {
    void print(Document doc);
    void scan(Document doc);
    void fax(Document doc);
    void copy(Document doc);
}

// Simple printer forced to implement everything!
public class SimplePrinter implements MultiFunctionDevice {
    @Override
    public void print(Document doc) { /* Works */ }

    @Override
    public void scan(Document doc) {
        throw new UnsupportedOperationException();
    }

    @Override
    public void fax(Document doc) {
        throw new UnsupportedOperationException();
    }

    @Override
    public void copy(Document doc) {
        throw new UnsupportedOperationException();
    }
}

// GOOD: Segregated interfaces
public interface Printer {
    void print(Document doc);
}

public interface Scanner {
    void scan(Document doc);
}

public interface Fax {
    void fax(Document doc);
}

// Simple printer only implements what it needs
public class SimplePrinter implements Printer {
    @Override
    public void print(Document doc) {
        System.out.println("Printing: " + doc.getName());
    }
}

// Advanced device implements multiple interfaces
public class MultiFunctionPrinter implements Printer, Scanner, Fax {
    @Override
    public void print(Document doc) {
        System.out.println("Printing: " + doc.getName());
    }

    @Override
    public void scan(Document doc) {
        System.out.println("Scanning: " + doc.getName());
    }

    @Override
    public void fax(Document doc) {
        System.out.println("Faxing: " + doc.getName());
    }
}
```

---

## Dependency Inversion Principle (DIP)

### Definition

> "High-level modules should not depend on low-level modules. Both should depend on abstractions."
> "Abstractions should not depend on details. Details should depend on abstractions."

### Violation Example

```java
// BAD: High-level module depends on low-level module

// Low-level module (implementation detail)
public class MySQLDatabase {
    public void connect() {
        System.out.println("Connecting to MySQL...");
    }

    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }

    public String retrieve(int id) {
        return "Data from MySQL";
    }
}

// High-level module directly depends on low-level module
public class UserService {
    private MySQLDatabase database;  // Direct dependency on concrete class!

    public UserService() {
        this.database = new MySQLDatabase();  // Tight coupling!
    }

    public void saveUser(User user) {
        database.connect();
        database.save(user.toString());
    }

    public User getUser(int id) {
        database.connect();
        String data = database.retrieve(id);
        return new User(data);
    }
}

// Problems:
// 1. Can't easily switch to PostgreSQL or MongoDB
// 2. Can't unit test without real database
// 3. UserService must change if database changes
```

### Correct Implementation

```java
// GOOD: Both depend on abstraction

// Abstraction (interface)
public interface Database {
    void connect();
    void save(String data);
    String retrieve(int id);
}

// Low-level modules implement the abstraction
public class MySQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL...");
    }

    @Override
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }

    @Override
    public String retrieve(int id) {
        return "Data from MySQL";
    }
}

public class PostgreSQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to PostgreSQL...");
    }

    @Override
    public void save(String data) {
        System.out.println("Saving to PostgreSQL: " + data);
    }

    @Override
    public String retrieve(int id) {
        return "Data from PostgreSQL";
    }
}

public class MongoDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MongoDB...");
    }

    @Override
    public void save(String data) {
        System.out.println("Saving to MongoDB: " + data);
    }

    @Override
    public String retrieve(int id) {
        return "Data from MongoDB";
    }
}

// In-memory for testing
public class InMemoryDatabase implements Database {
    private Map<Integer, String> storage = new HashMap<>();
    private int nextId = 1;

    @Override
    public void connect() {
        System.out.println("Using in-memory database");
    }

    @Override
    public void save(String data) {
        storage.put(nextId++, data);
    }

    @Override
    public String retrieve(int id) {
        return storage.get(id);
    }
}

// High-level module depends on abstraction (via constructor injection)
public class UserService {
    private final Database database;  // Depends on abstraction!

    // Dependency Injection via constructor
    public UserService(Database database) {
        this.database = database;
    }

    public void saveUser(User user) {
        database.connect();
        database.save(user.toString());
    }

    public User getUser(int id) {
        database.connect();
        String data = database.retrieve(id);
        return new User(data);
    }
}

// Usage - inject the dependency
public class Application {
    public static void main(String[] args) {
        // Production: Use MySQL
        Database mysqlDb = new MySQLDatabase();
        UserService productionService = new UserService(mysqlDb);

        // Testing: Use in-memory
        Database testDb = new InMemoryDatabase();
        UserService testService = new UserService(testDb);

        // Easy to switch implementations!
        Database postgresDb = new PostgreSQLDatabase();
        UserService postgresService = new UserService(postgresDb);
    }
}
```

### Dependency Injection Types

```java
// 1. Constructor Injection (Preferred)
public class OrderService {
    private final OrderRepository repository;
    private final PaymentGateway gateway;
    private final NotificationService notifier;

    public OrderService(OrderRepository repository,
                       PaymentGateway gateway,
                       NotificationService notifier) {
        this.repository = repository;
        this.gateway = gateway;
        this.notifier = notifier;
    }
}

// 2. Setter Injection
public class OrderService {
    private OrderRepository repository;

    public void setRepository(OrderRepository repository) {
        this.repository = repository;
    }
}

// 3. Interface Injection
public interface RepositoryInjector {
    void injectRepository(OrderRepository repository);
}

public class OrderService implements RepositoryInjector {
    private OrderRepository repository;

    @Override
    public void injectRepository(OrderRepository repository) {
        this.repository = repository;
    }
}

// With Spring Framework
@Service
public class OrderService {
    private final OrderRepository repository;

    @Autowired  // Spring injects the dependency
    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
```

### DIP with Notifications Example

```java
// Abstraction
public interface NotificationSender {
    void send(String to, String message);
}

// Implementations
public class EmailSender implements NotificationSender {
    @Override
    public void send(String to, String message) {
        System.out.println("Sending email to " + to + ": " + message);
    }
}

public class SMSSender implements NotificationSender {
    @Override
    public void send(String to, String message) {
        System.out.println("Sending SMS to " + to + ": " + message);
    }
}

public class PushNotificationSender implements NotificationSender {
    @Override
    public void send(String to, String message) {
        System.out.println("Sending push notification to " + to + ": " + message);
    }
}

// High-level module
public class NotificationService {
    private final List<NotificationSender> senders;

    public NotificationService(List<NotificationSender> senders) {
        this.senders = senders;
    }

    public void notifyAll(String to, String message) {
        senders.forEach(sender -> sender.send(to, message));
    }
}

// Configuration
public class Application {
    public static void main(String[] args) {
        List<NotificationSender> senders = Arrays.asList(
            new EmailSender(),
            new SMSSender(),
            new PushNotificationSender()
        );

        NotificationService service = new NotificationService(senders);
        service.notifyAll("user@example.com", "Welcome!");
    }
}
```

---

## Real-World Examples

### Spring Framework and SOLID

```java
// SRP: Each layer has single responsibility
@Entity  // Data representation
public class Order { }

@Repository  // Data access
public interface OrderRepository extends JpaRepository<Order, Long> { }

@Service  // Business logic
public class OrderService { }

@RestController  // HTTP handling
public class OrderController { }

// OCP: Extend behavior without modification
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter { }

// LSP: All implementations are substitutable
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username);
}

// ISP: Specific interfaces
public interface InitializingBean {
    void afterPropertiesSet();
}

public interface DisposableBean {
    void destroy();
}

// DIP: Spring DI container
@Service
public class UserService {
    private final UserRepository repository;

    @Autowired
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

---

## Interview Questions

### Q1: What is SOLID and why is it important?

**Answer:**
SOLID is a set of five object-oriented design principles:

- **S** - Single Responsibility
- **O** - Open/Closed
- **L** - Liskov Substitution
- **I** - Interface Segregation
- **D** - Dependency Inversion

**Importance:**

1. Creates maintainable, flexible code
2. Reduces coupling between components
3. Makes testing easier
4. Facilitates code reuse
5. Improves readability

---

### Q2: How does SRP differ from "one method per class"?

**Answer:**
SRP means **one reason to change**, not one method. A class can have multiple methods that serve the same responsibility.

Example: A `UserValidator` class might have `validateEmail()`, `validatePassword()`, and `validateUsername()` - all serving the single responsibility of user validation.

---

### Q3: How do you identify SRP violations?

**Signs of SRP violation:**

1. Class has too many dependencies
2. Class changes for multiple reasons
3. Class name includes "And" or "Manager"
4. Hard to describe class purpose in one sentence
5. Class has unrelated methods

---

### Q4: What's the difference between OCP and inheritance?

**Answer:**
OCP is a principle; inheritance is one mechanism to achieve it.

OCP can be achieved via:

- **Inheritance** - Extend base class
- **Composition** - Strategy pattern
- **Interfaces** - Plugin architecture

The goal is to add new behavior without modifying existing code.

---

### Q5: Give a real-world LSP violation example.

**Answer:**
Classic example: **Square extending Rectangle**

```java
Rectangle r = new Square();
r.setWidth(5);
r.setHeight(10);
// Expecting area = 50, but square forces width = height = 10
// Area becomes 100, violating client expectations
```

---

### Q6: How does ISP relate to SRP?

**Answer:**
Both promote separation of concerns:

- **SRP** - Separates classes by responsibility
- **ISP** - Separates interfaces by client needs

ISP is about ensuring clients don't depend on methods they don't use, while SRP is about ensuring classes don't have multiple reasons to change.

---

### Q7: What is Dependency Injection? How does it relate to DIP?

**Answer:**

- **DIP** is the principle (depend on abstractions)
- **DI** is a technique to implement DIP

DI provides dependencies from outside the class rather than creating them inside. This allows high-level modules to depend on abstractions, with concrete implementations injected at runtime.

---

### Q8: How do SOLID principles work together?

**Answer:**
They complement each other:

1. **SRP + ISP** → Small, focused classes and interfaces
2. **OCP + LSP** → Safe extension via inheritance
3. **DIP + OCP** → Add implementations without modification
4. **All five** → Clean, maintainable architecture

---

## Key Takeaways

| Principle | Key Point                  | Technique                          |
| --------- | -------------------------- | ---------------------------------- |
| SRP       | One reason to change       | Separate concerns into classes     |
| OCP       | Extend, don't modify       | Strategy, Template Method patterns |
| LSP       | Subtypes substitutable     | Proper inheritance, composition    |
| ISP       | Client-specific interfaces | Split fat interfaces               |
| DIP       | Depend on abstractions     | Dependency Injection               |

### Best Practices

1. **Start simple** - Don't over-engineer from day one
2. **Refactor when needed** - Apply SOLID when complexity grows
3. **Use interfaces** - Define contracts before implementations
4. **Prefer composition** - Over inheritance where possible
5. **Inject dependencies** - Don't instantiate inside classes
6. **Test-driven development** - Tests reveal SOLID violations
