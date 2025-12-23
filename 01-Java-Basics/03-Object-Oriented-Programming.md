# Object-Oriented Programming (OOP) in Java - Complete Guide

## Table of Contents

1. [Introduction to OOP](#introduction-to-oop)
2. [Classes and Objects](#classes-and-objects)
3. [Encapsulation](#encapsulation)
4. [Inheritance](#inheritance)
5. [Polymorphism](#polymorphism)
6. [Abstraction](#abstraction)
7. [Association, Aggregation, and Composition](#association-aggregation-and-composition)
8. [Constructors](#constructors)
9. [Static vs Instance Members](#static-vs-instance-members)
10. [Object Class and Its Methods](#object-class-and-its-methods)
11. [Interview Questions](#interview-questions)

---

## Introduction to OOP

### What is Object-Oriented Programming?

OOP is a programming paradigm based on the concept of **objects**, which contain data (attributes/fields) and code (methods/behavior). It organizes software design around data, rather than functions and logic.

### Why OOP?

| Problem with Procedural                | Solution with OOP                  |
| -------------------------------------- | ---------------------------------- |
| Data and functions are separate        | Data and behavior bundled together |
| Global data can be modified anywhere   | Encapsulation protects data        |
| Code duplication                       | Inheritance enables reuse          |
| Difficult to model real-world entities | Objects represent real entities    |
| Hard to manage large codebases         | Modular and maintainable           |

### Four Pillars of OOP

```
                    Object-Oriented Programming
                              │
    ┌─────────────┬───────────┼───────────┬─────────────┐
    │             │           │           │             │
Encapsulation  Inheritance  Polymorphism  Abstraction
    │             │           │           │
Hiding data   Reusing code  Many forms   Hiding complexity
Protecting    IS-A relation Overloading  Abstract classes
state                       Overriding   Interfaces
```

---

## Classes and Objects

### What is a Class?

A **class** is a blueprint or template that defines the structure (attributes) and behavior (methods) of objects. It is a logical entity.

### What is an Object?

An **object** is a runtime instance of a class. It is a physical entity that occupies memory and has state and behavior.

```java
// Class definition - blueprint
public class Car {
    // Attributes (state)
    private String brand;
    private String model;
    private int year;
    private double speed;

    // Constructor
    public Car(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.speed = 0;
    }

    // Methods (behavior)
    public void accelerate(double increment) {
        this.speed += increment;
        System.out.println(brand + " accelerating to " + speed + " km/h");
    }

    public void brake(double decrement) {
        this.speed = Math.max(0, this.speed - decrement);
        System.out.println(brand + " slowing to " + speed + " km/h");
    }

    public void displayInfo() {
        System.out.println(year + " " + brand + " " + model);
    }
}

// Creating objects - instances
public class Main {
    public static void main(String[] args) {
        // car1 and car2 are references to Car objects
        Car car1 = new Car("Toyota", "Camry", 2023);
        Car car2 = new Car("Honda", "Accord", 2022);

        car1.accelerate(50);  // Toyota accelerating to 50.0 km/h
        car2.accelerate(60);  // Honda accelerating to 60.0 km/h
    }
}
```

### Memory Representation

```
Stack Memory                      Heap Memory
┌─────────────────┐              ┌──────────────────────────────┐
│ car1 (reference)│──────────────│ Car Object                   │
│    [0x1234]     │              │ brand: "Toyota"              │
├─────────────────┤              │ model: "Camry"               │
│ car2 (reference)│──────────┐   │ year: 2023                   │
│    [0x5678]     │          │   │ speed: 0.0                   │
└─────────────────┘          │   └──────────────────────────────┘
                             │   ┌──────────────────────────────┐
                             └───│ Car Object                   │
                                 │ brand: "Honda"               │
                                 │ model: "Accord"              │
                                 │ year: 2022                   │
                                 │ speed: 0.0                   │
                                 └──────────────────────────────┘
```

### Class vs Object Comparison

| Aspect     | Class                         | Object                   |
| ---------- | ----------------------------- | ------------------------ |
| Definition | Blueprint/Template            | Instance of class        |
| Memory     | No memory allocated           | Memory allocated in heap |
| Creation   | Once at design time           | Many at runtime          |
| Keyword    | `class`                       | `new`                    |
| Contains   | Fields, methods, constructors | State values             |

---

## Encapsulation

### What is Encapsulation?

Encapsulation is the mechanism of **bundling data (variables) and methods** that operate on the data into a single unit (class), and **restricting direct access** to some components.

### Why Encapsulation?

1. **Data Protection**: Prevents unauthorized access and modification
2. **Flexibility**: Internal implementation can change without affecting external code
3. **Maintainability**: Changes are localized
4. **Testability**: Easier to test isolated units

### How to Achieve Encapsulation

```java
// POOR ENCAPSULATION - Public fields (Anti-pattern)
public class BadBankAccount {
    public double balance;  // Anyone can modify directly!

    public BadBankAccount(double balance) {
        this.balance = balance;
    }
}

// Usage of bad design
BadBankAccount bad = new BadBankAccount(1000);
bad.balance = -5000;  // Allowed but invalid!
```

```java
// GOOD ENCAPSULATION - Private fields with controlled access
public class BankAccount {
    // Private fields - hidden from outside
    private String accountNumber;
    private double balance;
    private String accountHolderName;

    // Constructor with validation
    public BankAccount(String accountNumber, String name, double initialDeposit) {
        if (initialDeposit < 0) {
            throw new IllegalArgumentException("Initial deposit cannot be negative");
        }
        this.accountNumber = accountNumber;
        this.accountHolderName = name;
        this.balance = initialDeposit;
    }

    // Getter - controlled read access
    public double getBalance() {
        return balance;
    }

    // No setter for balance - must use deposit/withdraw

    // Controlled modification with validation
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        balance += amount;
        System.out.println("Deposited: " + amount + ", New Balance: " + balance);
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        if (amount > balance) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        balance -= amount;
        System.out.println("Withdrawn: " + amount + ", New Balance: " + balance);
    }

    // Read-only getter
    public String getAccountNumber() {
        // Return masked account number for security
        return "XXXX" + accountNumber.substring(accountNumber.length() - 4);
    }
}

// Usage
BankAccount account = new BankAccount("1234567890", "John Doe", 1000);
account.deposit(500);    // Deposited: 500, New Balance: 1500
account.withdraw(200);   // Withdrawn: 200, New Balance: 1300
// account.balance = -5000;  // Compilation error - balance is private
```

### Access Modifiers

| Modifier                | Class | Package | Subclass | World |
| ----------------------- | ----- | ------- | -------- | ----- |
| `public`                | ✓     | ✓       | ✓        | ✓     |
| `protected`             | ✓     | ✓       | ✓        | ✗     |
| `default` (no modifier) | ✓     | ✓       | ✗        | ✗     |
| `private`               | ✓     | ✗       | ✗        | ✗     |

```java
public class AccessDemo {
    public int publicVar = 1;       // Accessible everywhere
    protected int protectedVar = 2; // Same package + subclasses
    int defaultVar = 3;             // Same package only
    private int privateVar = 4;     // This class only
}
```

### JavaBeans Convention (Getters/Setters)

```java
public class Employee {
    // Private fields
    private String firstName;
    private String lastName;
    private int age;
    private double salary;
    private boolean active;  // Note: boolean uses 'is' prefix

    // Default constructor
    public Employee() {}

    // Parameterized constructor
    public Employee(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        setAge(age);  // Use setter for validation
    }

    // Getters
    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public int getAge() {
        return age;
    }

    public double getSalary() {
        return salary;
    }

    // Boolean getter uses 'is' prefix
    public boolean isActive() {
        return active;
    }

    // Setters with validation
    public void setFirstName(String firstName) {
        if (firstName == null || firstName.trim().isEmpty()) {
            throw new IllegalArgumentException("First name cannot be empty");
        }
        this.firstName = firstName.trim();
    }

    public void setLastName(String lastName) {
        if (lastName == null || lastName.trim().isEmpty()) {
            throw new IllegalArgumentException("Last name cannot be empty");
        }
        this.lastName = lastName.trim();
    }

    public void setAge(int age) {
        if (age < 18 || age > 65) {
            throw new IllegalArgumentException("Age must be between 18 and 65");
        }
        this.age = age;
    }

    public void setSalary(double salary) {
        if (salary < 0) {
            throw new IllegalArgumentException("Salary cannot be negative");
        }
        this.salary = salary;
    }

    public void setActive(boolean active) {
        this.active = active;
    }

    // Derived property (no field)
    public String getFullName() {
        return firstName + " " + lastName;
    }
}
```

---

## Inheritance

### What is Inheritance?

Inheritance is a mechanism where a new class (**subclass/child**) is derived from an existing class (**superclass/parent**), inheriting its fields and methods.

### Why Inheritance?

1. **Code Reuse**: Don't repeat common code
2. **Extensibility**: Add new features to existing classes
3. **Polymorphism**: Enable runtime behavior changes
4. **Hierarchical Classification**: Model IS-A relationships

### Inheritance Syntax and Example

```java
// Parent class (Superclass)
public class Animal {
    protected String name;
    protected int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void eat() {
        System.out.println(name + " is eating");
    }

    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    public void displayInfo() {
        System.out.println("Name: " + name + ", Age: " + age);
    }
}

// Child class (Subclass) - inherits from Animal
public class Dog extends Animal {
    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }

    // Additional method specific to Dog
    public void bark() {
        System.out.println(name + " is barking: Woof!");
    }

    // Method overriding
    @Override
    public void eat() {
        System.out.println(name + " is eating dog food");
    }

    @Override
    public void displayInfo() {
        super.displayInfo();  // Call parent method
        System.out.println("Breed: " + breed);
    }
}

// Another child class
public class Cat extends Animal {
    private boolean isIndoor;

    public Cat(String name, int age, boolean isIndoor) {
        super(name, age);
        this.isIndoor = isIndoor;
    }

    public void meow() {
        System.out.println(name + " is meowing: Meow!");
    }

    @Override
    public void eat() {
        System.out.println(name + " is eating cat food");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog("Buddy", 3, "Labrador");
        Cat cat = new Cat("Whiskers", 2, true);

        dog.eat();         // Buddy is eating dog food (overridden)
        dog.sleep();       // Buddy is sleeping (inherited)
        dog.bark();        // Buddy is barking: Woof! (Dog specific)
        dog.displayInfo(); // Name: Buddy, Age: 3
                           // Breed: Labrador

        cat.eat();         // Whiskers is eating cat food (overridden)
        cat.meow();        // Whiskers is meowing: Meow!
    }
}
```

### Inheritance Hierarchy

```
                    Object (Root of all classes)
                       │
                    Animal
                    /    \
                 Dog      Cat
                /   \
           Labrador  Bulldog
```

### Types of Inheritance in Java

```java
// 1. SINGLE INHERITANCE
class A { }
class B extends A { }  // B inherits from A

// 2. MULTILEVEL INHERITANCE
class A { }
class B extends A { }
class C extends B { }  // C inherits from B which inherits from A

// 3. HIERARCHICAL INHERITANCE
class A { }
class B extends A { }  // B inherits from A
class C extends A { }  // C also inherits from A

// 4. MULTIPLE INHERITANCE (NOT supported with classes)
// class C extends A, B { }  // COMPILATION ERROR

// Multiple inheritance achieved through INTERFACES
interface A { }
interface B { }
class C implements A, B { }  // OK
```

### Why Java Doesn't Support Multiple Inheritance with Classes

```java
// Diamond Problem Example (Hypothetical - Not valid Java)
class A {
    void display() {
        System.out.println("A");
    }
}

class B extends A {
    @Override
    void display() {
        System.out.println("B");
    }
}

class C extends A {
    @Override
    void display() {
        System.out.println("C");
    }
}

// If this were allowed:
// class D extends B, C { }  // Which display() to use? Ambiguity!

// Solution: Use interfaces with default methods (Java 8+)
interface InterfaceB {
    default void display() {
        System.out.println("B");
    }
}

interface InterfaceC {
    default void display() {
        System.out.println("C");
    }
}

// Must resolve ambiguity explicitly
class D implements InterfaceB, InterfaceC {
    @Override
    public void display() {
        InterfaceB.super.display();  // Choose which one to call
    }
}
```

### super Keyword

```java
public class Parent {
    protected String name = "Parent";

    public Parent() {
        System.out.println("Parent constructor");
    }

    public Parent(String name) {
        this.name = name;
        System.out.println("Parent parameterized constructor");
    }

    public void display() {
        System.out.println("Parent display: " + name);
    }
}

public class Child extends Parent {
    private String name = "Child";  // Shadows parent's name

    public Child() {
        super();  // Calls Parent()
        System.out.println("Child constructor");
    }

    public Child(String name) {
        super(name);  // Calls Parent(String)
        System.out.println("Child parameterized constructor");
    }

    public void display() {
        System.out.println("Child name: " + this.name);
        System.out.println("Parent name: " + super.name);  // Access parent's field
        super.display();  // Call parent's method
    }
}

// Output when: new Child("Test")
// Parent parameterized constructor
// Child parameterized constructor
```

### final Keyword with Inheritance

```java
// Final class cannot be extended
public final class ImmutableClass {
    // This class cannot be inherited
}

// class ExtendedImmutable extends ImmutableClass { }  // ERROR

// Final method cannot be overridden
public class Parent {
    public final void criticalMethod() {
        // Cannot be overridden in child classes
        System.out.println("This implementation is fixed");
    }
}

public class Child extends Parent {
    // @Override
    // public void criticalMethod() { }  // ERROR: Cannot override final method
}
```

---

## Polymorphism

### What is Polymorphism?

Polymorphism means **"many forms"**. It allows objects to be treated as instances of their parent class rather than their actual class, enabling one interface to be used for different underlying forms.

### Types of Polymorphism

```
                    Polymorphism
                         │
         ┌───────────────┴───────────────┐
         │                               │
   Compile-time                    Runtime
  (Static Binding)              (Dynamic Binding)
         │                               │
  Method Overloading              Method Overriding
  Operator Overloading (N/A)
```

### 1. Compile-Time Polymorphism (Method Overloading)

**What**: Same method name, different parameters (number, type, or order).

```java
public class Calculator {
    // Overloaded methods - different parameter count
    public int add(int a, int b) {
        System.out.println("Two int parameters");
        return a + b;
    }

    public int add(int a, int b, int c) {
        System.out.println("Three int parameters");
        return a + b + c;
    }

    // Different parameter types
    public double add(double a, double b) {
        System.out.println("Two double parameters");
        return a + b;
    }

    // Different parameter order
    public String add(String a, int b) {
        System.out.println("String then int");
        return a + b;
    }

    public String add(int a, String b) {
        System.out.println("Int then String");
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator();

        // Compiler decides which method to call based on arguments
        System.out.println(calc.add(5, 10));           // Two int: 15
        System.out.println(calc.add(5, 10, 15));       // Three int: 30
        System.out.println(calc.add(5.0, 10.0));       // Two double: 15.0
        System.out.println(calc.add("Value: ", 10));   // String then int: Value: 10
        System.out.println(calc.add(10, " is the value")); // Int then String
    }
}
```

### Overloading Rules

```java
public class OverloadingRules {
    // VALID overloading
    void method(int a) { }
    void method(String a) { }           // Different type
    void method(int a, int b) { }       // Different count
    void method(int a, String b) { }    // Different types
    void method(String a, int b) { }    // Different order

    // INVALID overloading - return type alone doesn't count
    // int method(int a) { return a; }  // ERROR: already defined

    // INVALID - access modifier alone doesn't count
    // private void method(int a) { }   // ERROR: already defined

    // Varargs - treated as last resort
    void process(int... nums) { }
    void process(int a) { }  // This is called for process(5)
}
```

### Type Promotion in Overloading

```java
public class TypePromotionDemo {
    void method(long a) {
        System.out.println("long: " + a);
    }

    void method(double a) {
        System.out.println("double: " + a);
    }

    public static void main(String[] args) {
        TypePromotionDemo demo = new TypePromotionDemo();

        // int is promoted to long (not double)
        demo.method(5);  // Output: long: 5

        // Promotion hierarchy:
        // byte -> short -> int -> long -> float -> double
        //          char  ↗
    }
}
```

### 2. Runtime Polymorphism (Method Overriding)

**What**: Child class provides specific implementation for method defined in parent class.

```java
public class Shape {
    protected String color;

    public Shape(String color) {
        this.color = color;
    }

    public double calculateArea() {
        return 0;  // Base implementation
    }

    public void display() {
        System.out.println("This is a " + color + " shape");
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }

    @Override
    public void display() {
        System.out.println("This is a " + color + " circle with radius " + radius);
    }
}

public class Rectangle extends Shape {
    private double length;
    private double width;

    public Rectangle(String color, double length, double width) {
        super(color);
        this.length = length;
        this.width = width;
    }

    @Override
    public double calculateArea() {
        return length * width;
    }

    @Override
    public void display() {
        System.out.println("This is a " + color + " rectangle " + length + "x" + width);
    }
}

public class Main {
    public static void main(String[] args) {
        // Runtime polymorphism - reference type is Shape, actual type varies
        Shape shape1 = new Circle("red", 5.0);
        Shape shape2 = new Rectangle("blue", 4.0, 6.0);

        // Method called depends on actual object type, not reference type
        shape1.calculateArea();  // Circle's calculateArea (78.54...)
        shape2.calculateArea();  // Rectangle's calculateArea (24.0)

        // Polymorphic collection
        Shape[] shapes = {
            new Circle("green", 3.0),
            new Rectangle("yellow", 5.0, 3.0),
            new Circle("purple", 7.0)
        };

        for (Shape s : shapes) {
            s.display();  // Calls respective overridden method
            System.out.println("Area: " + s.calculateArea());
        }
    }
}
```

### Overriding Rules

```java
public class Parent {
    public void publicMethod() { }
    protected void protectedMethod() { }
    void defaultMethod() { }
    private void privateMethod() { }  // Not inherited, can't be overridden

    public Object returnObject() { return new Object(); }
}

public class Child extends Parent {
    // Rule 1: Same method signature (name + parameters)
    @Override
    public void publicMethod() { }

    // Rule 2: Access modifier must be same or more accessible
    @Override
    public void protectedMethod() { }  // OK: public > protected
    // protected void publicMethod() { }  // ERROR: can't reduce visibility

    // Rule 3: Return type must be same or covariant (subtype)
    @Override
    public String returnObject() { return "Hello"; }  // String is subtype of Object

    // Rule 4: Can throw same, fewer, or narrower checked exceptions
    // (covered in Exception Handling)

    // Rule 5: Final methods cannot be overridden
    // Rule 6: Static methods cannot be overridden (but can be hidden)
    // Rule 7: Private methods cannot be overridden (not inherited)
}
```

### Covariant Return Types

```java
public class Animal {
    public Animal reproduce() {
        return new Animal();
    }
}

public class Dog extends Animal {
    @Override
    public Dog reproduce() {  // Covariant return - Dog is subtype of Animal
        return new Dog();
    }
}
```

### Method Hiding vs Overriding (Static Methods)

```java
public class Parent {
    public static void staticMethod() {
        System.out.println("Parent static");
    }

    public void instanceMethod() {
        System.out.println("Parent instance");
    }
}

public class Child extends Parent {
    // Static method HIDING (not overriding)
    public static void staticMethod() {
        System.out.println("Child static");
    }

    // Instance method OVERRIDING
    @Override
    public void instanceMethod() {
        System.out.println("Child instance");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent p = new Child();

        // Static - resolved by reference type (compile-time)
        p.staticMethod();    // "Parent static" - hiding, not polymorphism

        // Instance - resolved by object type (runtime)
        p.instanceMethod();  // "Child instance" - true polymorphism
    }
}
```

---

## Abstraction

### What is Abstraction?

Abstraction is the process of **hiding implementation details** and showing only the functionality to the user. It lets you focus on **what** an object does instead of **how** it does it.

### Why Abstraction?

1. **Reduces complexity**: Users don't need to know internal workings
2. **Increases security**: Hides implementation details
3. **Supports maintainability**: Implementation can change without affecting users
4. **Enables loose coupling**: Depend on abstractions, not implementations

### Abstraction in Java: Two Ways

```
            Abstraction
                 │
     ┌───────────┴───────────┐
     │                       │
Abstract Classes         Interfaces
(0-100% abstraction)    (100% abstraction*)
```

\*Before Java 8; now interfaces can have default and static methods.

### Abstract Classes

```java
// Abstract class - cannot be instantiated
public abstract class Vehicle {
    // Regular fields
    protected String brand;
    protected int year;

    // Constructor (called by subclasses)
    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }

    // Abstract method - no implementation, must be overridden
    public abstract void start();
    public abstract void stop();
    public abstract double calculateFuelEfficiency();

    // Concrete method - has implementation
    public void displayInfo() {
        System.out.println(year + " " + brand);
    }

    // Final method - cannot be overridden
    public final String getBrand() {
        return brand;
    }
}

// Concrete class must implement all abstract methods
public class Car extends Vehicle {
    private int numberOfDoors;
    private double fuelCapacity;
    private double milesPerGallon;

    public Car(String brand, int year, int doors, double mpg) {
        super(brand, year);
        this.numberOfDoors = doors;
        this.milesPerGallon = mpg;
    }

    @Override
    public void start() {
        System.out.println("Car starting: Turn key or push button");
    }

    @Override
    public void stop() {
        System.out.println("Car stopping: Apply brakes and turn off engine");
    }

    @Override
    public double calculateFuelEfficiency() {
        return milesPerGallon;
    }
}

public class Motorcycle extends Vehicle {
    private int engineCC;

    public Motorcycle(String brand, int year, int cc) {
        super(brand, year);
        this.engineCC = cc;
    }

    @Override
    public void start() {
        System.out.println("Motorcycle starting: Kick start or electric start");
    }

    @Override
    public void stop() {
        System.out.println("Motorcycle stopping: Apply brakes");
    }

    @Override
    public double calculateFuelEfficiency() {
        // Simplified calculation
        return 50.0 - (engineCC / 100.0);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Vehicle v = new Vehicle("Test", 2023);  // ERROR: Cannot instantiate abstract class

        Vehicle car = new Car("Toyota", 2023, 4, 35.0);
        Vehicle bike = new Motorcycle("Honda", 2023, 600);

        // Polymorphism with abstraction
        Vehicle[] vehicles = {car, bike};
        for (Vehicle v : vehicles) {
            v.displayInfo();
            v.start();
            System.out.println("Fuel Efficiency: " + v.calculateFuelEfficiency());
        }
    }
}
```

### Interfaces

```java
// Interface - defines a contract
public interface Drivable {
    // Constants (implicitly public static final)
    int MAX_SPEED = 200;

    // Abstract methods (implicitly public abstract)
    void accelerate(int speed);
    void brake();
    void steer(String direction);

    // Default method (Java 8+) - has implementation
    default void honk() {
        System.out.println("Honking...");
    }

    // Static method (Java 8+)
    static void safetyCheck() {
        System.out.println("Performing safety check...");
    }

    // Private method (Java 9+) - helper for default methods
    private void logAction(String action) {
        System.out.println("Action: " + action);
    }
}

public interface Electric {
    void charge();
    int getBatteryLevel();
}

// Implementing multiple interfaces
public class ElectricCar implements Drivable, Electric {
    private int speed;
    private int batteryLevel;

    public ElectricCar() {
        this.speed = 0;
        this.batteryLevel = 100;
    }

    @Override
    public void accelerate(int speed) {
        this.speed = Math.min(speed, MAX_SPEED);
        System.out.println("Accelerating to " + this.speed);
    }

    @Override
    public void brake() {
        this.speed = 0;
        System.out.println("Stopping");
    }

    @Override
    public void steer(String direction) {
        System.out.println("Steering " + direction);
    }

    @Override
    public void charge() {
        batteryLevel = 100;
        System.out.println("Charging complete");
    }

    @Override
    public int getBatteryLevel() {
        return batteryLevel;
    }
}
```

### Abstract Class vs Interface

| Feature          | Abstract Class              | Interface                             |
| ---------------- | --------------------------- | ------------------------------------- |
| Instantiation    | Cannot instantiate          | Cannot instantiate                    |
| Methods          | Abstract + concrete         | Abstract + default + static (Java 8+) |
| Fields           | Any type (instance, static) | Only public static final              |
| Constructor      | Can have                    | Cannot have                           |
| Inheritance      | Single (extends)            | Multiple (implements)                 |
| Access Modifiers | Any                         | Public only (methods)                 |
| When to use      | IS-A with shared code       | CAN-DO capability                     |

### When to Use What?

```java
// Use ABSTRACT CLASS when:
// 1. You want to share code among related classes
// 2. You expect classes to have common methods or fields
// 3. You need non-public members
// 4. You want to declare non-static or non-final fields

public abstract class DatabaseConnection {
    protected String connectionString;

    public DatabaseConnection(String connectionString) {
        this.connectionString = connectionString;
    }

    // Shared implementation
    public void connect() {
        System.out.println("Connecting to: " + connectionString);
        // Common connection logic
    }

    // Each database type implements differently
    public abstract void executeQuery(String query);
}

// Use INTERFACE when:
// 1. You want to define a contract for unrelated classes
// 2. You need multiple inheritance
// 3. You want to define behavior that can be mixed in

public interface Serializable {
    byte[] serialize();
    void deserialize(byte[] data);
}

public interface Comparable<T> {
    int compareTo(T other);
}

// A class can implement both
public class Employee implements Serializable, Comparable<Employee> {
    // ...
}
```

---

## Association, Aggregation, and Composition

### Association

A general relationship between two classes where one class uses another.

```java
// Association: Teacher teaches Student (loose relationship)
public class Teacher {
    private String name;

    public void teach(Student student) {
        System.out.println(name + " teaches " + student.getName());
    }
}

public class Student {
    private String name;

    public String getName() {
        return name;
    }
}
```

### Aggregation (HAS-A - Weak)

A specialized form of association where objects have their own lifecycle. The contained object can exist independently.

```java
// Aggregation: Department has Teachers
// Teachers can exist without Department
public class Department {
    private String name;
    private List<Teacher> teachers;  // Department HAS teachers

    public Department(String name) {
        this.name = name;
        this.teachers = new ArrayList<>();
    }

    // Teachers are added from outside - they exist independently
    public void addTeacher(Teacher teacher) {
        teachers.add(teacher);
    }

    public void removeTeacher(Teacher teacher) {
        teachers.remove(teacher);
    }
}

// Teachers can exist without Department
Teacher teacher = new Teacher("John");
Department dept = new Department("Computer Science");
dept.addTeacher(teacher);
// If dept is destroyed, teacher still exists
```

### Composition (HAS-A - Strong)

A stronger form of aggregation where the contained object cannot exist without the container.

```java
// Composition: House has Rooms
// Rooms cannot exist without House
public class House {
    private List<Room> rooms;  // House owns rooms

    public House(int numberOfRooms) {
        rooms = new ArrayList<>();
        // Rooms are created as part of House
        for (int i = 0; i < numberOfRooms; i++) {
            rooms.add(new Room("Room " + (i + 1)));  // Created inside
        }
    }

    // Inner class - Room is part of House
    private class Room {
        private String name;

        public Room(String name) {
            this.name = name;
        }
    }
}
// When House is destroyed, all Rooms are destroyed too
```

### Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Relationship Strength                             │
│                                                                      │
│  Association ←───── Aggregation ←───── Composition                  │
│     (uses)         (has, weak)        (has, strong)                 │
│                                                                      │
│  Teacher uses      Department has     House has                     │
│  Student           Teachers           Rooms                         │
│                                                                      │
│  Objects exist     Objects can exist  Objects cannot                │
│  independently     independently      exist independently           │
└─────────────────────────────────────────────────────────────────────┘
```

### Real-World Example

```java
// Enterprise Application Example
public class Company {
    private String name;
    private List<Department> departments;  // Composition - departments belong to company

    public Company(String name) {
        this.name = name;
        this.departments = new ArrayList<>();
    }

    public void addDepartment(String deptName) {
        departments.add(new Department(deptName));  // Created inside
    }
}

public class Department {
    private String name;
    private List<Employee> employees;  // Aggregation - employees can exist without dept

    public Department(String name) {
        this.name = name;
        this.employees = new ArrayList<>();
    }

    public void addEmployee(Employee employee) {
        employees.add(employee);  // Employee exists independently
    }
}

public class Employee {
    private String name;
    private Address address;  // Composition - address is part of employee

    public Employee(String name, String street, String city) {
        this.name = name;
        this.address = new Address(street, city);  // Created inside
    }

    private class Address {
        private String street;
        private String city;

        public Address(String street, String city) {
            this.street = street;
            this.city = city;
        }
    }
}
```

---

## Constructors

### What is a Constructor?

A constructor is a special method that is called when an object is created. It initializes the object's state.

### Constructor Rules

1. Same name as class
2. No return type (not even void)
3. Called automatically when object is created
4. Can be overloaded
5. If no constructor defined, compiler provides default constructor

### Types of Constructors

```java
public class Person {
    private String name;
    private int age;
    private String email;

    // 1. DEFAULT CONSTRUCTOR (no parameters)
    public Person() {
        this.name = "Unknown";
        this.age = 0;
        this.email = "";
        System.out.println("Default constructor called");
    }

    // 2. PARAMETERIZED CONSTRUCTOR
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
        this.email = "";
        System.out.println("Parameterized constructor called");
    }

    // 3. CONSTRUCTOR OVERLOADING
    public Person(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }

    // 4. COPY CONSTRUCTOR (not built-in, but commonly implemented)
    public Person(Person other) {
        this.name = other.name;
        this.age = other.age;
        this.email = other.email;
        System.out.println("Copy constructor called");
    }
}
```

### Constructor Chaining

```java
public class Employee {
    private String name;
    private int age;
    private String department;
    private double salary;

    // Primary constructor - all parameters
    public Employee(String name, int age, String department, double salary) {
        this.name = name;
        this.age = age;
        this.department = department;
        this.salary = salary;
    }

    // Constructor chaining with this()
    public Employee(String name, int age, String department) {
        this(name, age, department, 50000.0);  // Calls primary constructor
    }

    public Employee(String name, int age) {
        this(name, age, "General");  // Calls 3-param constructor
    }

    public Employee(String name) {
        this(name, 25);  // Calls 2-param constructor
    }

    public Employee() {
        this("New Employee");  // Calls 1-param constructor
    }
}

// All ultimately call the primary constructor
Employee e1 = new Employee();
Employee e2 = new Employee("John");
Employee e3 = new Employee("Jane", 30);
Employee e4 = new Employee("Bob", 35, "IT");
Employee e5 = new Employee("Alice", 28, "HR", 75000);
```

### Constructor Chaining with Inheritance

```java
public class Animal {
    protected String name;

    public Animal() {
        System.out.println("Animal default constructor");
    }

    public Animal(String name) {
        this.name = name;
        System.out.println("Animal parameterized constructor: " + name);
    }
}

public class Dog extends Animal {
    private String breed;

    public Dog() {
        // super() is called implicitly if not specified
        System.out.println("Dog default constructor");
    }

    public Dog(String name) {
        super(name);  // Explicitly call parent constructor
        System.out.println("Dog constructor with name");
    }

    public Dog(String name, String breed) {
        super(name);
        this.breed = breed;
        System.out.println("Dog constructor with name and breed");
    }
}

// Execution order:
// new Dog("Buddy", "Labrador") ->
// 1. Animal parameterized constructor: Buddy
// 2. Dog constructor with name and breed
```

### Private Constructor (Singleton Pattern)

```java
public class Singleton {
    // Single instance
    private static Singleton instance;

    // Private constructor prevents external instantiation
    private Singleton() {
        System.out.println("Singleton instance created");
    }

    // Public method to get the instance
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    public void doSomething() {
        System.out.println("Doing something...");
    }
}

// Usage
Singleton s1 = Singleton.getInstance();
Singleton s2 = Singleton.getInstance();
System.out.println(s1 == s2);  // true - same instance
```

---

## Static vs Instance Members

### Instance Members

Belong to each object instance. Each object has its own copy.

### Static Members

Belong to the class itself. Shared by all instances.

```java
public class Counter {
    // Instance variable - each object has its own
    private int instanceCount = 0;

    // Static variable - shared by all objects
    private static int staticCount = 0;

    // Instance method
    public void incrementInstance() {
        instanceCount++;
    }

    // Static method
    public static void incrementStatic() {
        staticCount++;
    }

    public void display() {
        System.out.println("Instance: " + instanceCount + ", Static: " + staticCount);
    }

    public static void main(String[] args) {
        Counter c1 = new Counter();
        Counter c2 = new Counter();

        c1.incrementInstance();  // c1.instanceCount = 1
        c1.incrementStatic();    // Counter.staticCount = 1

        c2.incrementInstance();  // c2.instanceCount = 1
        c2.incrementStatic();    // Counter.staticCount = 2

        c1.display();  // Instance: 1, Static: 2
        c2.display();  // Instance: 1, Static: 2
    }
}
```

### Memory Allocation

```
Method Area (Metaspace)                    Heap
┌────────────────────────┐          ┌─────────────────────┐
│ Counter class          │          │ c1 object           │
│ - staticCount: 2       │          │ - instanceCount: 1  │
│ - static methods       │          └─────────────────────┘
│ - instance methods     │          ┌─────────────────────┐
│   (shared code)        │          │ c2 object           │
└────────────────────────┘          │ - instanceCount: 1  │
                                    └─────────────────────┘
```

### Static Block

```java
public class StaticBlockDemo {
    private static int value;
    private static List<String> list;

    // Static block - executed once when class is loaded
    static {
        System.out.println("Static block 1 executed");
        value = 100;
        list = new ArrayList<>();
    }

    // Multiple static blocks executed in order
    static {
        System.out.println("Static block 2 executed");
        list.add("Item 1");
        list.add("Item 2");
    }

    public static void main(String[] args) {
        System.out.println("Main method");
        System.out.println("Value: " + value);
        System.out.println("List: " + list);
    }
}

// Output:
// Static block 1 executed
// Static block 2 executed
// Main method
// Value: 100
// List: [Item 1, Item 2]
```

### Static vs Instance Comparison

| Aspect     | Static                            | Instance                 |
| ---------- | --------------------------------- | ------------------------ |
| Belongs to | Class                             | Object                   |
| Memory     | Method Area                       | Heap                     |
| Access     | ClassName.member or object.member | object.member only       |
| Created    | When class loads                  | When object created      |
| Copies     | One per class                     | One per object           |
| Can access | Other static members only         | Both static and instance |

### Static Method Restrictions

```java
public class StaticRestrictions {
    private int instanceVar = 10;
    private static int staticVar = 20;

    public void instanceMethod() {
        System.out.println(instanceVar);  // OK
        System.out.println(staticVar);    // OK
    }

    public static void staticMethod() {
        // System.out.println(instanceVar);  // ERROR: Non-static field
        System.out.println(staticVar);       // OK

        // System.out.println(this.staticVar);  // ERROR: 'this' in static

        // instanceMethod();  // ERROR: Non-static method
        staticMethod2();      // OK
    }

    public static void staticMethod2() { }
}
```

---

## Object Class and Its Methods

### The Object Class

Every class in Java implicitly extends `java.lang.Object`. It's the root of the class hierarchy.

```java
// These are equivalent:
public class MyClass { }
public class MyClass extends Object { }
```

### Important Methods of Object Class

```java
public class Employee {
    private int id;
    private String name;
    private double salary;

    public Employee(int id, String name, double salary) {
        this.id = id;
        this.name = name;
        this.salary = salary;
    }

    // 1. toString() - String representation of object
    @Override
    public String toString() {
        return "Employee{id=" + id + ", name='" + name + "', salary=" + salary + "}";
    }

    // 2. equals() - Check equality based on content
    @Override
    public boolean equals(Object obj) {
        // Same reference
        if (this == obj) return true;

        // Null check and type check
        if (obj == null || getClass() != obj.getClass()) return false;

        // Cast and compare fields
        Employee employee = (Employee) obj;
        return id == employee.id &&
               Double.compare(employee.salary, salary) == 0 &&
               Objects.equals(name, employee.name);
    }

    // 3. hashCode() - Must override if equals() is overridden
    @Override
    public int hashCode() {
        return Objects.hash(id, name, salary);
    }

    // 4. getClass() - Returns runtime class
    // Inherited from Object, cannot override (final method)

    // 5. clone() - Creates copy of object
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();  // Shallow copy
    }

    // 6. finalize() - Called before garbage collection (deprecated in Java 9+)
    @Override
    protected void finalize() throws Throwable {
        // Cleanup code (prefer try-with-resources instead)
        super.finalize();
    }
}
```

### equals() and hashCode() Contract

```java
// The contract:
// 1. If a.equals(b) is true, then a.hashCode() == b.hashCode() must be true
// 2. If a.hashCode() != b.hashCode(), then a.equals(b) must be false
// 3. If a.hashCode() == b.hashCode(), a.equals(b) may or may not be true

// Example: HashSet uses this contract
Set<Employee> employees = new HashSet<>();
Employee e1 = new Employee(1, "John", 50000);
Employee e2 = new Employee(1, "John", 50000);

// Without proper equals() and hashCode():
// Both would be added (different objects)

// With proper equals() and hashCode():
employees.add(e1);
employees.add(e2);
System.out.println(employees.size());  // 1 (duplicates detected)
```

### Shallow Copy vs Deep Copy

```java
public class Person implements Cloneable {
    private String name;
    private Address address;  // Reference type

    public Person(String name, String city) {
        this.name = name;
        this.address = new Address(city);
    }

    // Shallow Copy - address reference is copied
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    // Deep Copy - new address object created
    public Person deepClone() {
        Person cloned = new Person(this.name, this.address.getCity());
        return cloned;
    }
}

public class Address {
    private String city;

    public Address(String city) {
        this.city = city;
    }

    public String getCity() { return city; }
    public void setCity(String city) { this.city = city; }
}

// Demonstration
Person p1 = new Person("John", "New York");
Person p2 = (Person) p1.clone();  // Shallow copy
Person p3 = p1.deepClone();       // Deep copy

p1.getAddress().setCity("Boston");

// p2's address also changed (shallow copy - same reference)
// p3's address unchanged (deep copy - different object)
```

---

## Interview Questions

### Q1: What is the difference between abstraction and encapsulation?

**Answer:**

| Aspect         | Abstraction                                       | Encapsulation                                    |
| -------------- | ------------------------------------------------- | ------------------------------------------------ |
| Definition     | Hiding complexity, showing only relevant features | Bundling data and methods, hiding internal state |
| Focus          | What an object does                               | How an object does it                            |
| Implementation | Abstract classes, interfaces                      | Access modifiers, getters/setters                |
| Level          | Design level                                      | Implementation level                             |
| Example        | Vehicle interface with drive()                    | Private fields with public methods               |

---

### Q2: Can we override static methods? What about private methods?

**Answer:**

- **Static methods**: Cannot be overridden, only **hidden**. Method resolution happens at compile-time based on reference type.
- **Private methods**: Cannot be overridden because they are not inherited. Subclass can define a method with the same name, but it's a new method, not an override.

```java
class Parent {
    public static void staticMethod() { System.out.println("Parent static"); }
    private void privateMethod() { System.out.println("Parent private"); }
}

class Child extends Parent {
    public static void staticMethod() { System.out.println("Child static"); }  // Hiding
    private void privateMethod() { System.out.println("Child private"); }      // New method
}

Parent p = new Child();
p.staticMethod();  // "Parent static" (resolved by reference type)
```

---

### Q3: What is the diamond problem and how does Java solve it?

**Answer:**
Diamond problem occurs in multiple inheritance when a class inherits from two classes that have a common ancestor with a method, causing ambiguity about which method to use.

Java solves this by:

1. **Not allowing multiple class inheritance** - Only single inheritance with classes
2. **Allowing multiple interface inheritance** - But requires explicit resolution if there's a conflict

```java
interface A {
    default void display() { System.out.println("A"); }
}

interface B extends A {
    default void display() { System.out.println("B"); }
}

interface C extends A {
    default void display() { System.out.println("C"); }
}

// Must resolve ambiguity
class D implements B, C {
    @Override
    public void display() {
        B.super.display();  // Explicitly choose
    }
}
```

---

### Q4: What happens if we don't override equals() and hashCode()?

**Answer:**

- **equals()**: Uses `==` by default, comparing references. Two objects with same content are considered different.
- **hashCode()**: Returns different values for different objects, even if they have same content.

**Problems:**

1. Objects won't work correctly in HashMap, HashSet
2. Contains() won't find equivalent objects
3. Duplicates won't be detected

```java
// Without override
Employee e1 = new Employee(1, "John");
Employee e2 = new Employee(1, "John");

Set<Employee> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());  // 2 (both added, duplicates not detected)
```

---

### Q5: What is the output?

```java
public class Test {
    public static void main(String[] args) {
        Base b = new Derived();
        b.display();
    }
}

class Base {
    public void display() {
        System.out.println("Base display");
        show();
    }

    public void show() {
        System.out.println("Base show");
    }
}

class Derived extends Base {
    public void display() {
        System.out.println("Derived display");
        super.display();
    }

    public void show() {
        System.out.println("Derived show");
    }
}
```

**Answer:**

```
Derived display
Base display
Derived show
```

**Explanation:**

1. `b.display()` calls `Derived.display()` (runtime polymorphism)
2. Prints "Derived display"
3. `super.display()` calls `Base.display()`
4. Prints "Base display"
5. `show()` in Base.display() calls `Derived.show()` (runtime polymorphism applies to `this`)
6. Prints "Derived show"

---

### Q6: Why is multiple inheritance not supported with classes but supported with interfaces?

**Answer:**
**With classes:**

- Diamond problem causes ambiguity
- Complex state management (which parent's fields to inherit?)
- Constructor chaining becomes ambiguous

**With interfaces:**

- Interfaces don't have state (until Java 8 default methods)
- No constructor issues
- For default methods, explicit resolution is required
- Methods are implicitly public, reducing access conflicts

---

### Q7: Can a constructor be private? When would you use it?

**Answer:**
Yes, constructors can be private. Use cases:

1. **Singleton Pattern**: Ensure only one instance
2. **Factory Pattern**: Control object creation
3. **Utility Classes**: Prevent instantiation (like Math class)
4. **Builder Pattern**: Force use of builder for object creation

```java
// Utility class
public final class MathUtils {
    private MathUtils() {
        throw new AssertionError("Cannot instantiate");
    }

    public static int add(int a, int b) { return a + b; }
}
```

---

### Q8: What is covariant return type?

**Answer:**
Covariant return type allows an overriding method to return a subtype of the return type declared in the parent method.

```java
class Animal {
    Animal reproduce() { return new Animal(); }
}

class Dog extends Animal {
    @Override
    Dog reproduce() {  // Returns Dog instead of Animal
        return new Dog();
    }
}
```

Benefits:

- Avoids unnecessary casting
- More specific return type for subclasses
- Type-safe

---

### Q9: What is method hiding vs method overriding?

**Answer:**

| Aspect       | Method Hiding     | Method Overriding |
| ------------ | ----------------- | ----------------- |
| Applies to   | Static methods    | Instance methods  |
| Binding      | Compile-time      | Runtime           |
| Resolution   | By reference type | By object type    |
| @Override    | Not applicable    | Recommended       |
| Polymorphism | No                | Yes               |

```java
Parent p = new Child();
p.staticMethod();   // Parent's method (hiding - compile-time)
p.instanceMethod(); // Child's method (overriding - runtime)
```

---

### Q10: Can we have a class inside an interface?

**Answer:**
Yes! Interfaces can contain:

- Constants (public static final)
- Abstract methods
- Default methods (Java 8+)
- Static methods (Java 8+)
- Private methods (Java 9+)
- **Nested classes, interfaces, enums**

```java
public interface MyInterface {
    // Nested class in interface (implicitly static)
    class Helper {
        public static void help() {
            System.out.println("Helping...");
        }
    }

    // Nested interface
    interface SubInterface {
        void method();
    }

    // Nested enum
    enum Status { ACTIVE, INACTIVE }
}

// Usage
MyInterface.Helper.help();
MyInterface.Status status = MyInterface.Status.ACTIVE;
```

---

## Key Takeaways

1. **Four Pillars**: Encapsulation, Inheritance, Polymorphism, Abstraction
2. **Encapsulation**: Use private fields + public methods
3. **Inheritance**: Use `extends` for classes, `implements` for interfaces
4. **Polymorphism**: Overloading (compile-time), Overriding (runtime)
5. **Abstraction**: Abstract classes (partial), Interfaces (full)
6. **Composition over Inheritance**: Prefer HAS-A over IS-A when possible
7. **Always override**: `equals()`, `hashCode()`, `toString()` for value classes
8. **Static**: Class level, one copy; Instance: Object level, multiple copies
9. **Constructor chaining**: Use `this()` and `super()` effectively
10. **Final**: Cannot extend (class), cannot override (method), cannot reassign (variable)
