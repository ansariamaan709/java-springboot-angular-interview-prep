# Design Patterns in Java - Complete Interview Guide

## Table of Contents

1. [Introduction to Design Patterns](#introduction-to-design-patterns)
2. [Creational Patterns](#creational-patterns)
3. [Structural Patterns](#structural-patterns)
4. [Behavioral Patterns](#behavioral-patterns)
5. [Real-World Usage Examples](#real-world-usage-examples)
6. [Interview Questions](#interview-questions)

---

## Introduction to Design Patterns

### What are Design Patterns?

Design patterns are **reusable solutions** to commonly occurring problems in software design. They are templates that can be applied in many situations.

### Categories of Design Patterns

```
                    Design Patterns
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   Creational        Structural       Behavioral
        │                 │                 │
  ┌─────┴─────┐     ┌─────┴─────┐     ┌─────┴─────┐
  │ Singleton │     │  Adapter  │     │ Observer  │
  │  Factory  │     │  Bridge   │     │ Strategy  │
  │  Builder  │     │ Composite │     │  Command  │
  │ Prototype │     │ Decorator │     │ Iterator  │
  └───────────┘     │  Facade   │     │ Template  │
                    │  Flyweight│     │   State   │
                    │   Proxy   │     │ Chain of  │
                    └───────────┘     │   Resp.   │
                                      └───────────┘
```

---

## Creational Patterns

### 1. Singleton Pattern

**Purpose:** Ensure a class has only one instance and provide global access to it.

```java
// 1. Eager Initialization (Thread-safe)
public class EagerSingleton {
    private static final EagerSingleton INSTANCE = new EagerSingleton();

    private EagerSingleton() {}

    public static EagerSingleton getInstance() {
        return INSTANCE;
    }
}

// 2. Lazy Initialization with Double-Checked Locking (Thread-safe)
public class LazyDCLSingleton {
    private static volatile LazyDCLSingleton instance;

    private LazyDCLSingleton() {}

    public static LazyDCLSingleton getInstance() {
        if (instance == null) {
            synchronized (LazyDCLSingleton.class) {
                if (instance == null) {
                    instance = new LazyDCLSingleton();
                }
            }
        }
        return instance;
    }
}

// 3. Bill Pugh Singleton (Recommended - Thread-safe, Lazy)
public class BillPughSingleton {
    private BillPughSingleton() {}

    private static class SingletonHolder {
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }

    public static BillPughSingleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}

// 4. Enum Singleton (Best - Thread-safe, Serialization-safe)
public enum EnumSingleton {
    INSTANCE;

    private int value;

    public void setValue(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

**Breaking Singleton (and Prevention):**

```java
// Breaking via Reflection
public class BreakSingleton {
    public static void main(String[] args) throws Exception {
        BillPughSingleton s1 = BillPughSingleton.getInstance();

        // Break via reflection
        Constructor<BillPughSingleton> constructor =
            BillPughSingleton.class.getDeclaredConstructor();
        constructor.setAccessible(true);
        BillPughSingleton s2 = constructor.newInstance();

        System.out.println(s1 == s2);  // false - Singleton broken!
    }
}

// Prevention: Throw exception in constructor
public class SafeSingleton {
    private static volatile SafeSingleton instance;

    private SafeSingleton() {
        if (instance != null) {
            throw new IllegalStateException("Singleton already instantiated");
        }
    }
}

// Breaking via Serialization
// Prevention: Implement readResolve()
public class SerializableSingleton implements Serializable {
    private static final long serialVersionUID = 1L;
    private static final SerializableSingleton INSTANCE = new SerializableSingleton();

    private SerializableSingleton() {}

    public static SerializableSingleton getInstance() {
        return INSTANCE;
    }

    // Prevent creating new instance during deserialization
    protected Object readResolve() {
        return INSTANCE;
    }
}
```

**Real-World Examples:**

- `java.lang.Runtime.getRuntime()`
- `java.awt.Desktop.getDesktop()`
- Logger instances
- Configuration managers
- Database connection pools

---

### 2. Factory Pattern

**Purpose:** Create objects without exposing creation logic to the client.

#### Simple Factory

```java
// Product interface
interface Shape {
    void draw();
}

// Concrete products
class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Circle");
    }
}

class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Rectangle");
    }
}

class Triangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Triangle");
    }
}

// Simple Factory
class ShapeFactory {
    public static Shape createShape(String type) {
        return switch (type.toLowerCase()) {
            case "circle" -> new Circle();
            case "rectangle" -> new Rectangle();
            case "triangle" -> new Triangle();
            default -> throw new IllegalArgumentException("Unknown shape: " + type);
        };
    }
}

// Usage
public class FactoryDemo {
    public static void main(String[] args) {
        Shape circle = ShapeFactory.createShape("circle");
        circle.draw();  // Drawing Circle
    }
}
```

#### Factory Method Pattern

```java
// Product
interface Document {
    void open();
    void save();
}

// Concrete Products
class WordDocument implements Document {
    @Override
    public void open() { System.out.println("Opening Word document"); }

    @Override
    public void save() { System.out.println("Saving Word document"); }
}

class PdfDocument implements Document {
    @Override
    public void open() { System.out.println("Opening PDF document"); }

    @Override
    public void save() { System.out.println("Saving PDF document"); }
}

// Creator (Factory)
abstract class Application {
    // Factory method
    abstract Document createDocument();

    // Template method using factory method
    public void newDocument() {
        Document doc = createDocument();
        doc.open();
    }
}

// Concrete Creators
class WordApplication extends Application {
    @Override
    Document createDocument() {
        return new WordDocument();
    }
}

class PdfApplication extends Application {
    @Override
    Document createDocument() {
        return new PdfDocument();
    }
}
```

#### Abstract Factory Pattern

```java
// Abstract products
interface Button {
    void render();
}

interface Checkbox {
    void render();
}

// Concrete products - Windows family
class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows button");
    }
}

class WindowsCheckbox implements Checkbox {
    @Override
    public void render() {
        System.out.println("Rendering Windows checkbox");
    }
}

// Concrete products - Mac family
class MacButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Mac button");
    }
}

class MacCheckbox implements Checkbox {
    @Override
    public void render() {
        System.out.println("Rendering Mac checkbox");
    }
}

// Abstract factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete factories
class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Client
class Application {
    private Button button;
    private Checkbox checkbox;

    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void render() {
        button.render();
        checkbox.render();
    }
}
```

---

### 3. Builder Pattern

**Purpose:** Construct complex objects step by step, separating construction from representation.

```java
// Product
class Computer {
    private String cpu;
    private String ram;
    private String storage;
    private String gpu;
    private String os;

    private Computer(Builder builder) {
        this.cpu = builder.cpu;
        this.ram = builder.ram;
        this.storage = builder.storage;
        this.gpu = builder.gpu;
        this.os = builder.os;
    }

    // Builder class
    public static class Builder {
        // Required parameters
        private final String cpu;
        private final String ram;

        // Optional parameters with defaults
        private String storage = "256GB SSD";
        private String gpu = "Integrated";
        private String os = "Windows 11";

        public Builder(String cpu, String ram) {
            this.cpu = cpu;
            this.ram = ram;
        }

        public Builder storage(String storage) {
            this.storage = storage;
            return this;
        }

        public Builder gpu(String gpu) {
            this.gpu = gpu;
            return this;
        }

        public Builder os(String os) {
            this.os = os;
            return this;
        }

        public Computer build() {
            return new Computer(this);
        }
    }

    @Override
    public String toString() {
        return "Computer{cpu='" + cpu + "', ram='" + ram +
               "', storage='" + storage + "', gpu='" + gpu + "', os='" + os + "'}";
    }
}

// Usage
public class BuilderDemo {
    public static void main(String[] args) {
        Computer gaming = new Computer.Builder("Intel i9", "32GB")
            .storage("2TB NVMe")
            .gpu("RTX 4090")
            .os("Windows 11 Pro")
            .build();

        Computer basic = new Computer.Builder("Intel i3", "8GB")
            .build();  // Uses defaults for optional params

        System.out.println(gaming);
        System.out.println(basic);
    }
}
```

**With Lombok:**

```java
@Builder
@Data
public class User {
    private String firstName;
    private String lastName;
    private String email;
    @Builder.Default
    private String role = "USER";
}

// Usage
User user = User.builder()
    .firstName("John")
    .lastName("Doe")
    .email("john@example.com")
    .build();
```

---

### 4. Prototype Pattern

**Purpose:** Create new objects by copying existing ones without depending on their classes.

```java
// Prototype interface
interface Prototype extends Cloneable {
    Prototype clone();
}

// Concrete prototype
class Document implements Prototype {
    private String title;
    private String content;
    private List<String> comments;

    public Document() {
        this.comments = new ArrayList<>();
    }

    // Copy constructor for deep copy
    private Document(Document source) {
        this.title = source.title;
        this.content = source.content;
        this.comments = new ArrayList<>(source.comments);  // Deep copy of list
    }

    @Override
    public Document clone() {
        return new Document(this);
    }

    // Getters and setters
    public void setTitle(String title) { this.title = title; }
    public void setContent(String content) { this.content = content; }
    public void addComment(String comment) { comments.add(comment); }
}

// Prototype registry
class DocumentRegistry {
    private Map<String, Document> registry = new HashMap<>();

    public void register(String name, Document doc) {
        registry.put(name, doc);
    }

    public Document create(String name) {
        Document prototype = registry.get(name);
        return prototype != null ? prototype.clone() : null;
    }
}

// Usage
public class PrototypeDemo {
    public static void main(String[] args) {
        // Create and register prototypes
        Document memo = new Document();
        memo.setTitle("Memo Template");
        memo.setContent("Dear [Name],...");

        DocumentRegistry registry = new DocumentRegistry();
        registry.register("memo", memo);

        // Create new documents from prototype
        Document newMemo = registry.create("memo");
        newMemo.setTitle("Project Update");
    }
}
```

---

## Structural Patterns

### 1. Adapter Pattern

**Purpose:** Allow incompatible interfaces to work together.

```java
// Target interface (what client expects)
interface MediaPlayer {
    void play(String filename);
}

// Adaptee (incompatible interface)
class VLCPlayer {
    public void playVLC(String filename) {
        System.out.println("Playing VLC: " + filename);
    }
}

class MP4Player {
    public void playMP4(String filename) {
        System.out.println("Playing MP4: " + filename);
    }
}

// Adapter (Object Adapter)
class MediaAdapter implements MediaPlayer {
    private VLCPlayer vlcPlayer;
    private MP4Player mp4Player;

    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("vlc")) {
            vlcPlayer = new VLCPlayer();
        } else if (audioType.equalsIgnoreCase("mp4")) {
            mp4Player = new MP4Player();
        }
    }

    @Override
    public void play(String filename) {
        if (vlcPlayer != null) {
            vlcPlayer.playVLC(filename);
        } else if (mp4Player != null) {
            mp4Player.playMP4(filename);
        }
    }
}

// Client
class AudioPlayer implements MediaPlayer {
    @Override
    public void play(String filename) {
        String extension = filename.substring(filename.lastIndexOf('.') + 1);

        if (extension.equalsIgnoreCase("mp3")) {
            System.out.println("Playing MP3: " + filename);
        } else if (extension.equalsIgnoreCase("vlc") || extension.equalsIgnoreCase("mp4")) {
            MediaAdapter adapter = new MediaAdapter(extension);
            adapter.play(filename);
        } else {
            System.out.println("Unsupported format: " + extension);
        }
    }
}
```

**Real-World Examples:**

- `java.util.Arrays.asList()`
- `java.io.InputStreamReader`
- `java.io.OutputStreamWriter`

---

### 2. Decorator Pattern

**Purpose:** Add new behaviors to objects dynamically by wrapping them.

```java
// Component interface
interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete component
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }

    @Override
    public double getCost() {
        return 2.00;
    }
}

// Base decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }

    @Override
    public String getDescription() {
        return coffee.getDescription();
    }

    @Override
    public double getCost() {
        return coffee.getCost();
    }
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 0.50;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 0.20;
    }
}

class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Whip";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 0.70;
    }
}

// Usage
public class DecoratorDemo {
    public static void main(String[] args) {
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        // Simple Coffee $2.0

        coffee = new MilkDecorator(coffee);
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        // Simple Coffee, Milk $2.5

        coffee = new SugarDecorator(coffee);
        coffee = new WhipDecorator(coffee);
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        // Simple Coffee, Milk, Sugar, Whip $3.4
    }
}
```

**Real-World Examples:**

- `java.io.BufferedInputStream(InputStream)`
- `java.io.DataInputStream(InputStream)`
- `java.util.Collections.synchronizedList(List)`

---

### 3. Facade Pattern

**Purpose:** Provide a simplified interface to a complex subsystem.

```java
// Complex subsystem classes
class CPU {
    public void freeze() { System.out.println("CPU: Freeze"); }
    public void jump(long position) { System.out.println("CPU: Jump to " + position); }
    public void execute() { System.out.println("CPU: Execute"); }
}

class Memory {
    public void load(long position, byte[] data) {
        System.out.println("Memory: Load data at " + position);
    }
}

class HardDrive {
    public byte[] read(long lba, int size) {
        System.out.println("HardDrive: Read from " + lba);
        return new byte[size];
    }
}

// Facade
class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;

    private static final long BOOT_ADDRESS = 0x0000;
    private static final long BOOT_SECTOR = 0x001;
    private static final int SECTOR_SIZE = 512;

    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }

    public void start() {
        System.out.println("Computer starting...");
        cpu.freeze();
        memory.load(BOOT_ADDRESS, hardDrive.read(BOOT_SECTOR, SECTOR_SIZE));
        cpu.jump(BOOT_ADDRESS);
        cpu.execute();
        System.out.println("Computer started!");
    }
}

// Client uses simple interface
public class FacadeDemo {
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start();  // Simple method hides complex operations
    }
}
```

---

### 4. Proxy Pattern

**Purpose:** Provide a placeholder for another object to control access to it.

```java
// Subject interface
interface Image {
    void display();
}

// Real subject
class RealImage implements Image {
    private String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println("Loading image: " + filename);
        // Expensive operation
    }

    @Override
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}

// Virtual Proxy (lazy loading)
class ProxyImage implements Image {
    private String filename;
    private RealImage realImage;

    public ProxyImage(String filename) {
        this.filename = filename;
    }

    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);  // Load on demand
        }
        realImage.display();
    }
}

// Protection Proxy (access control)
class ProtectedImage implements Image {
    private Image image;
    private String userRole;

    public ProtectedImage(Image image, String userRole) {
        this.image = image;
        this.userRole = userRole;
    }

    @Override
    public void display() {
        if ("ADMIN".equals(userRole)) {
            image.display();
        } else {
            System.out.println("Access denied. Admin role required.");
        }
    }
}

// Caching Proxy
class CachedImage implements Image {
    private Image image;
    private boolean cached = false;

    public CachedImage(Image image) {
        this.image = image;
    }

    @Override
    public void display() {
        if (!cached) {
            System.out.println("Loading into cache...");
            cached = true;
        }
        image.display();
    }
}
```

---

## Behavioral Patterns

### 1. Observer Pattern

**Purpose:** Define a one-to-many dependency so that when one object changes state, all dependents are notified.

```java
// Observer interface
interface Observer {
    void update(String message);
}

// Subject interface
interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// Concrete Subject
class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;

    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }

    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

// Concrete Observers
class NewsChannel implements Observer {
    private String name;

    public NewsChannel(String name) {
        this.name = name;
    }

    @Override
    public void update(String message) {
        System.out.println(name + " received news: " + message);
    }
}

// Usage
public class ObserverDemo {
    public static void main(String[] args) {
        NewsAgency agency = new NewsAgency();

        Observer cnn = new NewsChannel("CNN");
        Observer bbc = new NewsChannel("BBC");

        agency.attach(cnn);
        agency.attach(bbc);

        agency.setNews("Breaking: Design Patterns are awesome!");
        // CNN received news: Breaking: Design Patterns are awesome!
        // BBC received news: Breaking: Design Patterns are awesome!

        agency.detach(bbc);

        agency.setNews("Update: Observer pattern demonstrated!");
        // CNN received news: Update: Observer pattern demonstrated!
    }
}
```

**Using Java Built-in (PropertyChangeSupport):**

```java
import java.beans.PropertyChangeListener;
import java.beans.PropertyChangeSupport;

class Person {
    private PropertyChangeSupport support = new PropertyChangeSupport(this);
    private String name;

    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }

    public void setName(String name) {
        String oldName = this.name;
        this.name = name;
        support.firePropertyChange("name", oldName, name);
    }
}
```

---

### 2. Strategy Pattern

**Purpose:** Define a family of algorithms, encapsulate each one, and make them interchangeable.

```java
// Strategy interface
interface PaymentStrategy {
    void pay(double amount);
}

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;

    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with credit card " +
                          cardNumber.substring(cardNumber.length() - 4));
    }
}

class PayPalPayment implements PaymentStrategy {
    private String email;

    public PayPalPayment(String email) {
        this.email = email;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with PayPal account " + email);
    }
}

class CryptoPayment implements PaymentStrategy {
    private String walletAddress;

    public CryptoPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with crypto wallet " +
                          walletAddress.substring(0, 8) + "...");
    }
}

// Context
class ShoppingCart {
    private List<Double> items = new ArrayList<>();
    private PaymentStrategy paymentStrategy;

    public void addItem(double price) {
        items.add(price);
    }

    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }

    public void checkout() {
        double total = items.stream().mapToDouble(Double::doubleValue).sum();
        paymentStrategy.pay(total);
    }
}

// Usage
public class StrategyDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addItem(100.0);
        cart.addItem(50.0);

        // Pay with credit card
        cart.setPaymentStrategy(new CreditCardPayment("1234567890123456"));
        cart.checkout();  // Paid $150.0 with credit card 3456

        // Change strategy at runtime
        cart.setPaymentStrategy(new PayPalPayment("user@example.com"));
        cart.checkout();  // Paid $150.0 with PayPal account user@example.com
    }
}
```

**With Lambda (Functional Strategy):**

```java
// Using functional interface
@FunctionalInterface
interface SortStrategy<T> {
    void sort(List<T> list);
}

class Sorter<T> {
    public void sort(List<T> list, SortStrategy<T> strategy) {
        strategy.sort(list);
    }
}

// Usage with lambdas
Sorter<Integer> sorter = new Sorter<>();
List<Integer> numbers = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5, 9));

sorter.sort(numbers, Collections::sort);  // Natural order
sorter.sort(numbers, list -> Collections.sort(list, Collections.reverseOrder()));  // Reverse
```

---

### 3. Template Method Pattern

**Purpose:** Define the skeleton of an algorithm, deferring some steps to subclasses.

```java
// Abstract class with template method
abstract class DataMiner {
    // Template method
    public final void mine(String path) {
        openFile(path);
        extractData();
        parseData();
        analyzeData();
        sendReport();
        closeFile();
    }

    // Steps that vary - abstract methods
    protected abstract void openFile(String path);
    protected abstract void extractData();
    protected abstract void parseData();
    protected abstract void closeFile();

    // Steps with default implementation - can be overridden (hooks)
    protected void analyzeData() {
        System.out.println("Analyzing data...");
    }

    protected void sendReport() {
        System.out.println("Sending report via email...");
    }
}

// Concrete implementations
class PDFDataMiner extends DataMiner {
    @Override
    protected void openFile(String path) {
        System.out.println("Opening PDF file: " + path);
    }

    @Override
    protected void extractData() {
        System.out.println("Extracting data from PDF...");
    }

    @Override
    protected void parseData() {
        System.out.println("Parsing PDF data...");
    }

    @Override
    protected void closeFile() {
        System.out.println("Closing PDF file");
    }
}

class CSVDataMiner extends DataMiner {
    @Override
    protected void openFile(String path) {
        System.out.println("Opening CSV file: " + path);
    }

    @Override
    protected void extractData() {
        System.out.println("Reading CSV rows...");
    }

    @Override
    protected void parseData() {
        System.out.println("Parsing CSV columns...");
    }

    @Override
    protected void closeFile() {
        System.out.println("Closing CSV file");
    }

    @Override
    protected void sendReport() {
        System.out.println("Sending report via Slack...");  // Override hook
    }
}
```

---

### 4. Chain of Responsibility Pattern

**Purpose:** Pass request along a chain of handlers until one handles it.

```java
// Handler interface
abstract class SupportHandler {
    protected SupportHandler nextHandler;

    public void setNext(SupportHandler handler) {
        this.nextHandler = handler;
    }

    public abstract void handleRequest(Ticket ticket);
}

// Request class
class Ticket {
    private String issue;
    private int priority;  // 1 = Low, 2 = Medium, 3 = High

    public Ticket(String issue, int priority) {
        this.issue = issue;
        this.priority = priority;
    }

    public String getIssue() { return issue; }
    public int getPriority() { return priority; }
}

// Concrete handlers
class Level1Support extends SupportHandler {
    @Override
    public void handleRequest(Ticket ticket) {
        if (ticket.getPriority() == 1) {
            System.out.println("Level 1 Support handling: " + ticket.getIssue());
        } else if (nextHandler != null) {
            System.out.println("Level 1: Escalating ticket...");
            nextHandler.handleRequest(ticket);
        }
    }
}

class Level2Support extends SupportHandler {
    @Override
    public void handleRequest(Ticket ticket) {
        if (ticket.getPriority() == 2) {
            System.out.println("Level 2 Support handling: " + ticket.getIssue());
        } else if (nextHandler != null) {
            System.out.println("Level 2: Escalating ticket...");
            nextHandler.handleRequest(ticket);
        }
    }
}

class Level3Support extends SupportHandler {
    @Override
    public void handleRequest(Ticket ticket) {
        System.out.println("Level 3 Expert Support handling: " + ticket.getIssue());
    }
}

// Usage
public class ChainDemo {
    public static void main(String[] args) {
        // Build chain
        SupportHandler level1 = new Level1Support();
        SupportHandler level2 = new Level2Support();
        SupportHandler level3 = new Level3Support();

        level1.setNext(level2);
        level2.setNext(level3);

        // Test tickets
        level1.handleRequest(new Ticket("Password reset", 1));
        // Level 1 Support handling: Password reset

        level1.handleRequest(new Ticket("Software installation", 2));
        // Level 1: Escalating ticket...
        // Level 2 Support handling: Software installation

        level1.handleRequest(new Ticket("System crash", 3));
        // Level 1: Escalating ticket...
        // Level 2: Escalating ticket...
        // Level 3 Expert Support handling: System crash
    }
}
```

**Real-World Examples:**

- Servlet Filters
- Spring Security Filter Chain
- Exception handling (try-catch chain)

---

### 5. Command Pattern

**Purpose:** Encapsulate a request as an object, allowing parameterization and queuing.

```java
// Command interface
interface Command {
    void execute();
    void undo();
}

// Receiver
class Light {
    private String location;

    public Light(String location) {
        this.location = location;
    }

    public void on() {
        System.out.println(location + " light is ON");
    }

    public void off() {
        System.out.println(location + " light is OFF");
    }
}

// Concrete commands
class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.on();
    }

    @Override
    public void undo() {
        light.off();
    }
}

class LightOffCommand implements Command {
    private Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.off();
    }

    @Override
    public void undo() {
        light.on();
    }
}

// Invoker
class RemoteControl {
    private Command[] onCommands;
    private Command[] offCommands;
    private Command lastCommand;

    public RemoteControl(int slots) {
        onCommands = new Command[slots];
        offCommands = new Command[slots];
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void onButtonPressed(int slot) {
        if (onCommands[slot] != null) {
            onCommands[slot].execute();
            lastCommand = onCommands[slot];
        }
    }

    public void offButtonPressed(int slot) {
        if (offCommands[slot] != null) {
            offCommands[slot].execute();
            lastCommand = offCommands[slot];
        }
    }

    public void undoButtonPressed() {
        if (lastCommand != null) {
            lastCommand.undo();
        }
    }
}

// Usage
public class CommandDemo {
    public static void main(String[] args) {
        RemoteControl remote = new RemoteControl(3);

        Light livingRoom = new Light("Living Room");
        Light kitchen = new Light("Kitchen");

        remote.setCommand(0, new LightOnCommand(livingRoom), new LightOffCommand(livingRoom));
        remote.setCommand(1, new LightOnCommand(kitchen), new LightOffCommand(kitchen));

        remote.onButtonPressed(0);   // Living Room light is ON
        remote.offButtonPressed(0);  // Living Room light is OFF
        remote.undoButtonPressed();  // Living Room light is ON (undo)
    }
}
```

---

## Real-World Usage Examples

### Patterns in JDK

| Pattern   | JDK Examples                                            |
| --------- | ------------------------------------------------------- |
| Singleton | `Runtime.getRuntime()`, `System.getSecurityManager()`   |
| Factory   | `Calendar.getInstance()`, `NumberFormat.getInstance()`  |
| Builder   | `StringBuilder`, `Stream.Builder`, `Locale.Builder`     |
| Adapter   | `Arrays.asList()`, `InputStreamReader`                  |
| Decorator | `BufferedInputStream`, `Collections.synchronizedList()` |
| Proxy     | `java.lang.reflect.Proxy`, RMI stubs                    |
| Observer  | `java.util.Observer`, `PropertyChangeListener`          |
| Strategy  | `Comparator`, `java.util.logging.Handler`               |
| Template  | `InputStream.read()`, `AbstractList`                    |
| Iterator  | `java.util.Iterator`, `Enumeration`                     |

### Patterns in Spring Framework

| Pattern   | Spring Usage                                  |
| --------- | --------------------------------------------- |
| Singleton | Bean scope (default)                          |
| Factory   | `BeanFactory`, `ApplicationContext`           |
| Proxy     | AOP proxies, `@Transactional`                 |
| Template  | `JdbcTemplate`, `RestTemplate`, `JmsTemplate` |
| Observer  | `ApplicationEvent`, `@EventListener`          |
| Strategy  | `Resource` implementations, `ViewResolver`    |
| Decorator | `BeanPostProcessor`                           |

---

## Interview Questions

### Q1: What is the difference between Factory and Abstract Factory?

| Factory Method      | Abstract Factory           |
| ------------------- | -------------------------- |
| Creates one product | Creates family of products |
| Single method       | Multiple methods           |
| Subclass decides    | Factory object decides     |
| Inheritance-based   | Composition-based          |

---

### Q2: When to use Builder vs Constructor?

**Use Builder when:**

- Many constructor parameters (> 4)
- Many optional parameters
- Object is immutable
- Need validation before creation
- Readable, fluent API needed

**Use Constructor when:**

- Few required parameters
- Simple object creation
- No optional parameters

---

### Q3: What is the difference between Adapter and Decorator?

| Adapter                       | Decorator               |
| ----------------------------- | ----------------------- |
| Changes interface             | Same interface          |
| Makes incompatible compatible | Adds functionality      |
| Works with existing code      | Enhances behavior       |
| Usually wraps one object      | Can wrap multiple times |

---

### Q4: What is the difference between Strategy and State?

| Strategy                   | State                        |
| -------------------------- | ---------------------------- |
| Client chooses algorithm   | Object changes internally    |
| Algorithms independent     | States know about each other |
| Interchangeable algorithms | Represent object states      |
| One strategy at a time     | Transitions between states   |

---

### Q5: What is the difference between Proxy and Decorator?

| Proxy                  | Decorator            |
| ---------------------- | -------------------- |
| Controls access        | Adds behavior        |
| May create real object | Requires real object |
| Same interface         | Same interface       |
| Client unaware         | Client may compose   |

---

### Q6: How to make Singleton thread-safe?

1. **Eager initialization** - Thread-safe by default
2. **Double-checked locking** - With volatile keyword
3. **Bill Pugh** - Static inner class
4. **Enum** - Best approach, handles serialization too

---

### Q7: What design patterns are used in Java I/O?

- **Decorator**: `BufferedInputStream`, `DataInputStream`
- **Adapter**: `InputStreamReader` (byte to char)
- **Template**: `InputStream.read()` abstract methods
- **Factory**: `Files.newInputStream()`

---

### Q8: How does Spring use design patterns?

- **Singleton**: Default bean scope
- **Factory**: `BeanFactory`, `ApplicationContext`
- **Proxy**: AOP, `@Transactional`, `@Async`
- **Template**: `JdbcTemplate`, `RestTemplate`
- **Observer**: Event handling with `ApplicationEvent`
- **Strategy**: `ViewResolver`, `HandlerMapping`

---

### Q9: Explain the Observer pattern with Java example.

**Answer:**
Observer pattern defines one-to-many dependency so that when one object changes state, all dependents are notified.

```java
// Subject (Observable)
interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// Observer
interface Observer {
    void update(String message);
}

// Concrete Subject
class NewsPublisher implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;

    @Override
    public void attach(Observer observer) { observers.add(observer); }

    @Override
    public void detach(Observer observer) { observers.remove(observer); }

    @Override
    public void notifyObservers() {
        for (Observer o : observers) {
            o.update(news);
        }
    }

    public void publishNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

// Concrete Observer
class NewsSubscriber implements Observer {
    private String name;

    public NewsSubscriber(String name) { this.name = name; }

    @Override
    public void update(String message) {
        System.out.println(name + " received: " + message);
    }
}
```

**Real-world uses:**

- Event listeners (Swing, JavaFX)
- Message queues
- Reactive programming (RxJava, Project Reactor)

---

### Q10: What is the Chain of Responsibility pattern?

**Answer:**
Chain of Responsibility passes request along a chain of handlers until one handles it.

```java
abstract class Handler {
    protected Handler next;

    public Handler setNext(Handler next) {
        this.next = next;
        return next;
    }

    public abstract void handle(Request request);
}

class AuthenticationHandler extends Handler {
    @Override
    public void handle(Request request) {
        if (!request.isAuthenticated()) {
            throw new SecurityException("Not authenticated");
        }
        if (next != null) next.handle(request);
    }
}

class AuthorizationHandler extends Handler {
    @Override
    public void handle(Request request) {
        if (!request.isAuthorized()) {
            throw new SecurityException("Not authorized");
        }
        if (next != null) next.handle(request);
    }
}

class LoggingHandler extends Handler {
    @Override
    public void handle(Request request) {
        System.out.println("Logging request: " + request);
        if (next != null) next.handle(request);
    }
}

// Usage
Handler chain = new AuthenticationHandler();
chain.setNext(new AuthorizationHandler())
     .setNext(new LoggingHandler());
chain.handle(request);
```

**Real-world uses:**

- Servlet filters
- Spring interceptors
- Exception handling chains

---

### Q11: Explain Prototype pattern.

**Answer:**
Prototype creates new objects by cloning existing ones.

```java
interface Prototype extends Cloneable {
    Prototype clone();
}

class Document implements Prototype {
    private String content;
    private List<String> attachments;

    public Document(String content) {
        this.content = content;
        this.attachments = new ArrayList<>();
    }

    // Copy constructor for deep clone
    private Document(Document other) {
        this.content = other.content;
        this.attachments = new ArrayList<>(other.attachments);
    }

    @Override
    public Document clone() {
        return new Document(this);  // Deep copy
    }
}

// Usage
Document original = new Document("Template");
original.addAttachment("file1.pdf");

Document copy = original.clone();  // Independent copy
```

**When to use:**

- Object creation is expensive
- Need variations of complex objects
- Avoiding subclass explosion

---

### Q12: What is the Flyweight pattern?

**Answer:**
Flyweight shares common state among multiple objects to reduce memory.

```java
class CharacterFlyweight {
    // Intrinsic state (shared)
    private final char character;
    private final String font;

    CharacterFlyweight(char character, String font) {
        this.character = character;
        this.font = font;
    }

    // Extrinsic state (passed in)
    public void display(int x, int y, int size) {
        System.out.println("Char: " + character + " at (" + x + "," + y + ")");
    }
}

class FlyweightFactory {
    private static final Map<String, CharacterFlyweight> pool = new HashMap<>();

    public static CharacterFlyweight getCharacter(char c, String font) {
        String key = c + "-" + font;
        return pool.computeIfAbsent(key, k -> new CharacterFlyweight(c, font));
    }
}

// Usage - same object reused
CharacterFlyweight a1 = FlyweightFactory.getCharacter('A', "Arial");
CharacterFlyweight a2 = FlyweightFactory.getCharacter('A', "Arial");
System.out.println(a1 == a2);  // true (same object)
```

**Real-world uses:**

- String pool in Java
- Integer cache (-128 to 127)
- Connection pools

---

### Q13: Compare Abstract Factory vs Factory Method.

| Abstract Factory            | Factory Method             |
| --------------------------- | -------------------------- |
| Creates families of objects | Creates one type of object |
| Multiple factory methods    | Single factory method      |
| Composition-based           | Inheritance-based          |
| `GUIFactory.createButton()` | `Dialog.createButton()`    |

```java
// Factory Method
abstract class Dialog {
    abstract Button createButton();  // Factory method

    public void render() {
        Button btn = createButton();
        btn.render();
    }
}

class WindowsDialog extends Dialog {
    @Override
    Button createButton() { return new WindowsButton(); }
}

// Abstract Factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

class WindowsFactory implements GUIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}
```

---

### Q14: What is the Command pattern?

**Answer:**
Command encapsulates a request as an object, allowing parameterization and queuing.

```java
interface Command {
    void execute();
    void undo();
}

class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) { this.light = light; }

    @Override
    public void execute() { light.turnOn(); }

    @Override
    public void undo() { light.turnOff(); }
}

class RemoteControl {
    private Command[] commands = new Command[7];
    private Stack<Command> history = new Stack<>();

    public void setCommand(int slot, Command cmd) {
        commands[slot] = cmd;
    }

    public void pressButton(int slot) {
        commands[slot].execute();
        history.push(commands[slot]);
    }

    public void undoLast() {
        if (!history.isEmpty()) {
            history.pop().undo();
        }
    }
}
```

**Real-world uses:**

- GUI actions (cut, copy, paste)
- Transaction rollback
- Task scheduling
- Macro recording

---

### Q15: Explain the State pattern.

**Answer:**
State allows an object to alter its behavior when its internal state changes.

```java
interface OrderState {
    void next(Order order);
    void prev(Order order);
    void printStatus();
}

class Order {
    private OrderState state = new OrderedState();

    public void setState(OrderState state) { this.state = state; }
    public void nextState() { state.next(this); }
    public void prevState() { state.prev(this); }
    public void printStatus() { state.printStatus(); }
}

class OrderedState implements OrderState {
    @Override
    public void next(Order order) { order.setState(new ShippedState()); }
    @Override
    public void prev(Order order) { System.out.println("Already at beginning"); }
    @Override
    public void printStatus() { System.out.println("Order placed"); }
}

class ShippedState implements OrderState {
    @Override
    public void next(Order order) { order.setState(new DeliveredState()); }
    @Override
    public void prev(Order order) { order.setState(new OrderedState()); }
    @Override
    public void printStatus() { System.out.println("Order shipped"); }
}

class DeliveredState implements OrderState {
    @Override
    public void next(Order order) { System.out.println("Already delivered"); }
    @Override
    public void prev(Order order) { order.setState(new ShippedState()); }
    @Override
    public void printStatus() { System.out.println("Order delivered"); }
}
```

**Difference from Strategy:**

- **State:** Behavior changes based on internal state
- **Strategy:** Behavior selected externally

---

## Key Takeaways

1. **Singleton** - One instance, global access (use Enum)
2. **Factory** - Decouple object creation
3. **Builder** - Complex object construction
4. **Adapter** - Interface compatibility
5. **Decorator** - Dynamic behavior addition
6. **Facade** - Simplified interface
7. **Proxy** - Access control/lazy loading
8. **Observer** - Event notification
9. **Strategy** - Interchangeable algorithms
10. **Template** - Algorithm skeleton
