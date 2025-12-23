# Advanced Java: Generics, Reflection, Annotations & Serialization — Complete Interview Guide

## Table of Contents

1. [Generics Deep Dive](#generics-deep-dive)
2. [Type Erasure & Reification](#type-erasure--reification)
3. [Wildcards: PECS Principle](#wildcards-pecs-principle)
4. [Reflection API](#reflection-api)
5. [Annotations & Custom Annotations](#annotations--custom-annotations)
6. [Annotation Processing](#annotation-processing)
7. [Serialization & Deserialization](#serialization--deserialization)
8. [Java Records (Java 14+)](#java-records-java-14)
9. [Sealed Classes (Java 17+)](#sealed-classes-java-17)
10. [Pattern Matching](#pattern-matching)
11. [Interview Questions](#interview-questions)

---

## Generics Deep Dive

### What are Generics?

Generics enable **type parameterization** — writing code that works with different types while maintaining compile-time type safety.

### Why Generics exist

- **Type safety**: Catch type errors at compile time, not runtime
- **Elimination of casts**: No explicit casting needed
- **Code reuse**: Single implementation works for multiple types

### Basic Syntax

```java
// Generic class
public class Box<T> {
    private T content;

    public void set(T content) { this.content = content; }
    public T get() { return content; }
}

// Generic method
public class Utility {
    public static <T> T getFirst(List<T> list) {
        return list.isEmpty() ? null : list.get(0);
    }

    // Multiple type parameters
    public static <K, V> Map<K, V> createMap(K key, V value) {
        Map<K, V> map = new HashMap<>();
        map.put(key, value);
        return map;
    }
}

// Usage
Box<String> stringBox = new Box<>();
stringBox.set("Hello");
String s = stringBox.get();  // No cast needed

Box<Integer> intBox = new Box<>();
intBox.set(42);
Integer i = intBox.get();
```

### Bounded Type Parameters

```java
// Upper bound: T must be Number or subclass
public class NumericBox<T extends Number> {
    private T value;

    public double doubleValue() {
        return value.doubleValue();  // Can call Number methods
    }
}

// Multiple bounds (class first, then interfaces)
public class Calculator<T extends Number & Comparable<T>> {
    public T max(T a, T b) {
        return a.compareTo(b) >= 0 ? a : b;
    }
}

// Usage
NumericBox<Integer> intBox = new NumericBox<>();  // OK
NumericBox<Double> doubleBox = new NumericBox<>();  // OK
// NumericBox<String> stringBox = new NumericBox<>();  // Compile error!
```

### Generic Interfaces

```java
public interface Repository<T, ID> {
    T findById(ID id);
    List<T> findAll();
    T save(T entity);
    void delete(T entity);
}

public class UserRepository implements Repository<User, Long> {
    @Override
    public User findById(Long id) { /* ... */ }

    @Override
    public List<User> findAll() { /* ... */ }

    @Override
    public User save(User entity) { /* ... */ }

    @Override
    public void delete(User entity) { /* ... */ }
}
```

---

## Type Erasure & Reification

### What is Type Erasure?

Java generics are implemented via **type erasure** — generic type information is removed at compile time and replaced with bounds or Object.

### How Erasure Works

```java
// What you write
public class Box<T> {
    private T content;
    public T get() { return content; }
}

// After erasure (what JVM sees)
public class Box {
    private Object content;
    public Object get() { return content; }
}

// With bounds
public class NumericBox<T extends Number> {
    private T value;
}

// After erasure
public class NumericBox {
    private Number value;  // Erased to bound
}
```

### Implications of Type Erasure

```java
// 1. Cannot create generic arrays
// T[] array = new T[10];  // Compile error

// Workaround
@SuppressWarnings("unchecked")
T[] array = (T[]) new Object[10];

// 2. Cannot use instanceof with generic types
// if (obj instanceof List<String>) {}  // Compile error
if (obj instanceof List<?>) {}  // OK (unbounded)

// 3. Cannot create instances of type parameters
// T obj = new T();  // Compile error

// Workaround: pass Class<T>
public static <T> T create(Class<T> clazz) throws Exception {
    return clazz.getDeclaredConstructor().newInstance();
}

// 4. Static members cannot use class type parameters
public class Box<T> {
    // private static T staticField;  // Compile error
    private static Object staticField;  // OK
}
```

### Bridge Methods (Interview favorite)

```java
public class IntegerBox extends Box<Integer> {
    @Override
    public Integer get() { return super.get(); }
}

// Compiler generates bridge method for polymorphism
// public Object get() { return get(); }  // Bridge calls Integer get()
```

### Reifiable vs Non-Reifiable Types

| Reifiable (type info at runtime) | Non-Reifiable (erased) |
| -------------------------------- | ---------------------- |
| Primitives (int, boolean)        | List<String>           |
| Non-generic types (String)       | List<? extends Number> |
| Raw types (List)                 | T (type variables)     |
| Unbounded wildcards (List<?>)    | Map<K, V>              |
| Arrays of reifiable (String[])   |                        |

---

## Wildcards: PECS Principle

### Wildcard Types

```java
// Unbounded wildcard: unknown type
List<?> unknownList;

// Upper bounded: ? is some subtype of Number
List<? extends Number> numbers;

// Lower bounded: ? is some supertype of Integer
List<? super Integer> integers;
```

### PECS: Producer Extends, Consumer Super

**The single most important generics rule for interviews.**

```java
// PRODUCER: When you READ from a structure, use extends
public double sumOfList(List<? extends Number> list) {
    double sum = 0;
    for (Number n : list) {  // Safe to read as Number
        sum += n.doubleValue();
    }
    return sum;
}

// Works with List<Integer>, List<Double>, List<Number>
sumOfList(Arrays.asList(1, 2, 3));
sumOfList(Arrays.asList(1.5, 2.5, 3.5));

// CONSUMER: When you WRITE to a structure, use super
public void addIntegers(List<? super Integer> list) {
    list.add(1);    // Safe to add Integer
    list.add(2);
    list.add(3);
}

// Works with List<Integer>, List<Number>, List<Object>
List<Number> numbers = new ArrayList<>();
addIntegers(numbers);
```

### Why PECS Works (Covariance & Contravariance)

```java
// EXTENDS (Covariance) - Read-only
List<? extends Number> nums = new ArrayList<Integer>();
Number n = nums.get(0);  // OK: Integer IS-A Number
// nums.add(1);  // ERROR: Can't add - might be List<Double>

// SUPER (Contravariance) - Write-only
List<? super Integer> ints = new ArrayList<Number>();
ints.add(42);  // OK: Integer fits in Number list
// Integer i = ints.get(0);  // ERROR: Might return Long
Object obj = ints.get(0);  // Only safe to read as Object
```

### Real-World Example: Collections.copy()

```java
// Signature from Java Collections
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (int i = 0; i < src.size(); i++) {
        dest.set(i, src.get(i));  // Read from src, write to dest
    }
}

// Usage
List<Number> numbers = new ArrayList<>(Arrays.asList(0.0, 0.0, 0.0));
List<Integer> integers = Arrays.asList(1, 2, 3);
Collections.copy(numbers, integers);  // Copy integers to numbers
```

---

## Reflection API

### What is Reflection?

Reflection allows examining and modifying program behavior at **runtime** — classes, methods, fields, annotations.

### When to Use

- Frameworks (Spring, Hibernate, JUnit)
- Serialization libraries
- Dependency injection
- Dynamic proxy creation

### When NOT to Use

- Performance-critical code (reflection is slow)
- When compile-time safety is preferred
- Simple cases where direct calls work

### Core Reflection Classes

```java
import java.lang.reflect.*;

public class ReflectionDemo {

    public static void main(String[] args) throws Exception {
        // Get Class object
        Class<?> clazz = User.class;
        // or: Class.forName("com.example.User")
        // or: user.getClass()

        // Class information
        System.out.println("Name: " + clazz.getName());
        System.out.println("Simple Name: " + clazz.getSimpleName());
        System.out.println("Package: " + clazz.getPackage().getName());
        System.out.println("Superclass: " + clazz.getSuperclass());
        System.out.println("Interfaces: " + Arrays.toString(clazz.getInterfaces()));
        System.out.println("Modifiers: " + Modifier.toString(clazz.getModifiers()));
    }
}
```

### Accessing Fields

```java
public class FieldReflection {

    public static void main(String[] args) throws Exception {
        User user = new User("John", "secret123");
        Class<?> clazz = user.getClass();

        // Get all declared fields (including private)
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);  // Bypass access control
            System.out.println(field.getName() + " = " + field.get(user));
        }

        // Modify private field
        Field passwordField = clazz.getDeclaredField("password");
        passwordField.setAccessible(true);
        passwordField.set(user, "newPassword");

        System.out.println("New password: " + passwordField.get(user));
    }
}

class User {
    private String name;
    private String password;

    public User(String name, String password) {
        this.name = name;
        this.password = password;
    }
}
```

### Invoking Methods

```java
public class MethodReflection {

    public static void main(String[] args) throws Exception {
        Calculator calc = new Calculator();
        Class<?> clazz = calc.getClass();

        // Get method by name and parameter types
        Method addMethod = clazz.getMethod("add", int.class, int.class);

        // Invoke method
        Object result = addMethod.invoke(calc, 5, 3);
        System.out.println("5 + 3 = " + result);  // 8

        // Invoke private method
        Method privateMethod = clazz.getDeclaredMethod("secretMethod");
        privateMethod.setAccessible(true);
        privateMethod.invoke(calc);

        // Get all methods
        Method[] methods = clazz.getDeclaredMethods();
        for (Method m : methods) {
            System.out.println(m.getName() + " - params: " +
                Arrays.toString(m.getParameterTypes()));
        }
    }
}

class Calculator {
    public int add(int a, int b) { return a + b; }
    private void secretMethod() { System.out.println("Secret!"); }
}
```

### Creating Instances

```java
public class InstantiationReflection {

    public static void main(String[] args) throws Exception {
        Class<?> clazz = Person.class;

        // Using no-arg constructor
        Object person1 = clazz.getDeclaredConstructor().newInstance();

        // Using parameterized constructor
        Constructor<?> constructor = clazz.getConstructor(String.class, int.class);
        Object person2 = constructor.newInstance("Alice", 30);

        System.out.println(person2);  // Person{name='Alice', age=30}
    }
}

class Person {
    private String name;
    private int age;

    public Person() {}

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}
```

### Dynamic Proxy (Interview favorite)

```java
import java.lang.reflect.*;

public class DynamicProxyDemo {

    public static void main(String[] args) {
        UserService realService = new UserServiceImpl();

        UserService proxy = (UserService) Proxy.newProxyInstance(
            UserService.class.getClassLoader(),
            new Class<?>[] { UserService.class },
            new LoggingHandler(realService)
        );

        proxy.createUser("John");
        // Output:
        // Before: createUser
        // Creating user: John
        // After: createUser, result: true
    }
}

interface UserService {
    boolean createUser(String name);
}

class UserServiceImpl implements UserService {
    public boolean createUser(String name) {
        System.out.println("Creating user: " + name);
        return true;
    }
}

class LoggingHandler implements InvocationHandler {
    private final Object target;

    LoggingHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before: " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("After: " + method.getName() + ", result: " + result);
        return result;
    }
}
```

---

## Annotations & Custom Annotations

### What are Annotations?

Metadata attached to code elements that can be processed at compile time or runtime.

### Built-in Annotations

```java
public class BuiltInAnnotations {

    @Override  // Compiler checks if actually overriding
    public String toString() {
        return "Demo";
    }

    @Deprecated(since = "1.5", forRemoval = true)
    public void oldMethod() {}

    @SuppressWarnings("unchecked")
    public void uncheckedCode() {
        List rawList = new ArrayList();
        List<String> stringList = rawList;  // Warning suppressed
    }

    @FunctionalInterface  // Compiler enforces single abstract method
    interface MyFunction {
        void apply();
    }

    @SafeVarargs  // Suppress heap pollution warning
    public final <T> void safeVarargs(T... elements) {}
}
```

### Creating Custom Annotations

```java
import java.lang.annotation.*;

// Meta-annotations define annotation behavior
@Retention(RetentionPolicy.RUNTIME)  // Available at runtime
@Target({ElementType.METHOD, ElementType.TYPE})  // Where it can be used
@Documented  // Include in Javadoc
@Inherited   // Subclasses inherit this annotation
public @interface Auditable {
    String action() default "ACCESS";
    boolean logParams() default false;
    String[] roles() default {};
}

// Usage
@Auditable(action = "CREATE", logParams = true, roles = {"ADMIN"})
public void createUser(String name) {
    // ...
}
```

### Retention Policies

| Policy  | Description                    | Use Case                     |
| ------- | ------------------------------ | ---------------------------- |
| SOURCE  | Discarded by compiler          | @Override, @SuppressWarnings |
| CLASS   | In .class file, not at runtime | Bytecode analysis tools      |
| RUNTIME | Available via reflection       | Frameworks, DI, validation   |

### Target Element Types

```java
@Target({
    ElementType.TYPE,           // Class, interface, enum
    ElementType.FIELD,          // Field
    ElementType.METHOD,         // Method
    ElementType.PARAMETER,      // Method parameter
    ElementType.CONSTRUCTOR,    // Constructor
    ElementType.LOCAL_VARIABLE, // Local variable
    ElementType.ANNOTATION_TYPE,// Another annotation
    ElementType.PACKAGE,        // Package declaration
    ElementType.TYPE_PARAMETER, // Generic type parameter <T>
    ElementType.TYPE_USE        // Any type usage
})
public @interface MyAnnotation {}
```

### Processing Annotations at Runtime

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Cacheable {
    int ttlSeconds() default 300;
    String keyPrefix() default "";
}

public class CacheProcessor {

    public static void processCacheableMethod(Object target, String methodName,
                                               Object... args) throws Exception {
        Method method = target.getClass().getMethod(methodName);

        if (method.isAnnotationPresent(Cacheable.class)) {
            Cacheable cacheable = method.getAnnotation(Cacheable.class);

            String cacheKey = cacheable.keyPrefix() + "_" + methodName + "_" +
                              Arrays.hashCode(args);
            int ttl = cacheable.ttlSeconds();

            // Check cache, invoke method, store result...
            System.out.println("Cache key: " + cacheKey + ", TTL: " + ttl + "s");
        }

        method.invoke(target, args);
    }
}

class UserService {
    @Cacheable(ttlSeconds = 600, keyPrefix = "user")
    public User findById(Long id) {
        return new User(id, "John");
    }
}
```

---

## Annotation Processing

### Compile-Time Processing

Used by Lombok, MapStruct, Dagger for code generation.

```java
// Simple compile-time processor
@SupportedAnnotationTypes("com.example.GenerateBuilder")
@SupportedSourceVersion(SourceVersion.RELEASE_17)
public class BuilderProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations,
                          RoundEnvironment roundEnv) {

        for (Element element : roundEnv.getElementsAnnotatedWith(GenerateBuilder.class)) {
            if (element.getKind() == ElementKind.CLASS) {
                TypeElement classElement = (TypeElement) element;
                generateBuilder(classElement);
            }
        }
        return true;
    }

    private void generateBuilder(TypeElement classElement) {
        // Generate builder class source code
        // Write to processingEnv.getFiler().createSourceFile(...)
    }
}
```

---

## Serialization & Deserialization

### Java Serialization Basics

```java
import java.io.*;

public class SerializationDemo {

    public static void main(String[] args) throws Exception {
        Employee emp = new Employee(1L, "John", "secret");

        // Serialize
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("employee.ser"))) {
            oos.writeObject(emp);
        }

        // Deserialize
        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream("employee.ser"))) {
            Employee loaded = (Employee) ois.readObject();
            System.out.println(loaded);
            // Output: Employee{id=1, name='John', password='null'}
            // password is null because it's transient
        }
    }
}

class Employee implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;  // Version control

    private Long id;
    private String name;
    private transient String password;  // Not serialized

    public Employee(Long id, String name, String password) {
        this.id = id;
        this.name = name;
        this.password = password;
    }

    // Custom serialization hooks
    @Serial
    private void writeObject(ObjectOutputStream oos) throws IOException {
        oos.defaultWriteObject();
        // Custom: encrypt password before writing
        oos.writeObject(encrypt(password));
    }

    @Serial
    private void readObject(ObjectInputStream ois)
            throws IOException, ClassNotFoundException {
        ois.defaultReadObject();
        // Custom: decrypt password after reading
        this.password = decrypt((String) ois.readObject());
    }

    private String encrypt(String s) { return s; /* actual encryption */ }
    private String decrypt(String s) { return s; /* actual decryption */ }

    @Override
    public String toString() {
        return "Employee{id=" + id + ", name='" + name +
               "', password='" + password + "'}";
    }
}
```

### serialVersionUID Importance

```java
// If you don't declare serialVersionUID:
// - JVM computes one based on class structure
// - Any class change = different UID = InvalidClassException on deserialize

// Best practice: Always declare explicitly
private static final long serialVersionUID = 1L;

// Increment when making incompatible changes
```

### Externalization (Full Control)

```java
public class CustomEmployee implements Externalizable {
    private Long id;
    private String name;

    public CustomEmployee() {}  // Required no-arg constructor

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeLong(id);
        out.writeUTF(name);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException {
        this.id = in.readLong();
        this.name = in.readUTF();
    }
}
```

### Modern Alternative: JSON Serialization

```java
// Jackson (industry standard)
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.annotation.*;

public class JsonSerializationDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        User user = new User(1L, "John", "secret");

        // Serialize to JSON
        String json = mapper.writeValueAsString(user);
        System.out.println(json);
        // {"id":1,"name":"John"}  (password excluded)

        // Deserialize from JSON
        User loaded = mapper.readValue(json, User.class);
    }
}

class User {
    private Long id;
    private String name;

    @JsonIgnore  // Exclude from JSON
    private String password;

    @JsonProperty("full_name")  // Custom JSON key
    public String getName() { return name; }

    // constructors, getters, setters
}
```

---

## Java Records (Java 14+)

### What are Records?

Immutable data carriers with auto-generated constructors, getters, equals, hashCode, toString.

```java
// Traditional class: ~50 lines
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int x() { return x; }
    public int y() { return y; }

    @Override
    public boolean equals(Object o) { /* ... */ }
    @Override
    public int hashCode() { /* ... */ }
    @Override
    public String toString() { /* ... */ }
}

// Record: 1 line
public record Point(int x, int y) {}

// Usage
Point p = new Point(10, 20);
System.out.println(p.x());     // 10 (accessor, not getX())
System.out.println(p);         // Point[x=10, y=20]
```

### Custom Constructors in Records

```java
public record Range(int start, int end) {

    // Compact constructor (validation)
    public Range {
        if (start > end) {
            throw new IllegalArgumentException("start > end");
        }
    }

    // Additional constructor
    public Range(int end) {
        this(0, end);
    }
}

// With derived fields
public record Person(String firstName, String lastName) {
    public String fullName() {
        return firstName + " " + lastName;
    }
}
```

### Records with Generics

```java
public record Pair<L, R>(L left, R right) {}

Pair<String, Integer> pair = new Pair<>("age", 25);
System.out.println(pair.left());   // "age"
System.out.println(pair.right());  // 25
```

---

## Sealed Classes (Java 17+)

### What are Sealed Classes?

Restrict which classes can extend/implement a type.

```java
// Only these three can extend Shape
public sealed class Shape
    permits Circle, Rectangle, Triangle {
}

public final class Circle extends Shape {
    private final double radius;
    // ...
}

public final class Rectangle extends Shape {
    private final double width, height;
    // ...
}

// Can be sealed itself
public sealed class Triangle extends Shape
    permits EquilateralTriangle, RightTriangle {
}

public final class EquilateralTriangle extends Triangle {}
public final class RightTriangle extends Triangle {}

// non-sealed: opens for extension
public non-sealed class OpenShape extends Shape {}
```

### Sealed Interfaces

```java
public sealed interface Result<T>
    permits Success, Failure {
}

public record Success<T>(T value) implements Result<T> {}
public record Failure<T>(String error) implements Result<T> {}
```

---

## Pattern Matching

### instanceof Pattern Matching (Java 16+)

```java
// Old way
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// New way
if (obj instanceof String s) {
    System.out.println(s.length());  // s already cast
}

// With negation
if (!(obj instanceof String s)) {
    return;
}
// s is in scope here
System.out.println(s.length());
```

### Switch Pattern Matching (Java 21+)

```java
public String describe(Object obj) {
    return switch (obj) {
        case null -> "null value";
        case String s -> "String of length " + s.length();
        case Integer i when i > 0 -> "Positive integer: " + i;
        case Integer i -> "Non-positive integer: " + i;
        case int[] arr -> "int array of length " + arr.length;
        case Point(int x, int y) -> "Point at (" + x + ", " + y + ")";
        default -> "Unknown type";
    };
}

// With sealed classes (exhaustive)
public double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Triangle t -> /* calculate */;
        // No default needed - compiler knows all subtypes
    };
}
```

### Record Patterns (Java 21+)

```java
record Point(int x, int y) {}
record Line(Point start, Point end) {}

// Nested deconstruction
public void printLine(Object obj) {
    if (obj instanceof Line(Point(int x1, int y1), Point(int x2, int y2))) {
        System.out.println("Line from (" + x1 + "," + y1 +
                          ") to (" + x2 + "," + y2 + ")");
    }
}
```

---

## Interview Questions

### Generics

1. **What is type erasure and why does Java use it?**

   - Type info removed at compile time for backward compatibility with pre-generics code. Generic List<String> becomes raw List at runtime.

2. **Explain PECS principle with example.**

   - Producer Extends, Consumer Super. Use `? extends T` when reading (producing values), `? super T` when writing (consuming values). Collections.copy() demonstrates both.

3. **Can you create a generic array? Why or why not?**

   - No. Arrays are reifiable (know their type at runtime), generics are erased. `new T[10]` is illegal. Workaround: cast from Object array or use List.

4. **What's the difference between List<?> and List<Object>?**
   - `List<?>` accepts any List (List<String>, List<Integer>), read-only (can only add null). `List<Object>` only accepts List<Object>, can add any Object.

### Reflection

5. **What are the performance implications of reflection?**

   - Slower than direct calls (no JIT optimization, runtime checks). Use sparingly, cache Method/Field objects, consider alternatives like MethodHandles.

6. **How does Spring use reflection?**

   - Bean instantiation, dependency injection, annotation processing, AOP proxies, accessing private fields for configuration.

7. **Can reflection access private members? Is it secure?**
   - Yes with setAccessible(true). Security managers can restrict this. Java 9+ module system adds stronger encapsulation.

### Annotations

8. **Difference between @Retention SOURCE, CLASS, RUNTIME?**

   - SOURCE: only at compile (IDE warnings). CLASS: in .class but not runtime (bytecode tools). RUNTIME: available via reflection (frameworks).

9. **How do frameworks like Spring process annotations?**
   - At startup, scan classpath for annotated classes. Use reflection to read annotations and configure beans, inject dependencies, set up proxies.

### Serialization

10. **What happens if serialVersionUID doesn't match?**

    - InvalidClassException on deserialize. Always declare explicitly to control versioning.

11. **How do you exclude a field from serialization?**
    - Mark as `transient`. For JSON: @JsonIgnore (Jackson), @Transient (JPA).

### Records & Sealed

12. **When would you use a record vs a regular class?**

    - Record for immutable data carriers (DTOs, value objects). Regular class when you need mutability, inheritance, or complex behavior.

13. **What problem do sealed classes solve?**
    - Controlled inheritance for domain modeling. Enables exhaustive pattern matching. Better than marker interfaces or enums for type hierarchies.
