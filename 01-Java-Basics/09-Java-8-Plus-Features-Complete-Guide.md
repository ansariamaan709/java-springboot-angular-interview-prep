# Java 8+ Features - Complete Deep Dive

## Table of Contents

1. [Lambda Expressions](#lambda-expressions)
2. [Functional Interfaces](#functional-interfaces)
3. [Method References](#method-references)
4. [Stream API](#stream-api)
5. [Optional Class](#optional-class)
6. [Date and Time API](#date-and-time-api)
7. [Default and Static Methods in Interfaces](#default-and-static-methods-in-interfaces)
8. [Java 9-17 Features](#java-9-17-features)
9. [Interview Questions](#interview-questions)

---

## Lambda Expressions

### What is a Lambda Expression?

A **lambda expression** is a concise way to represent an anonymous function (a method without a name) that can be passed around as a value.

### Lambda Syntax

```java
// Full syntax
(parameters) -> { statements; }

// Simplified forms
(a, b) -> a + b                    // Multiple params, expression
a -> a * 2                         // Single param (no parentheses)
() -> System.out.println("Hello")  // No params
(String s) -> s.length()           // With type declaration
```

### Lambda Examples

```java
public class LambdaExamples {
    public static void main(String[] args) {
        // Without lambda (anonymous class)
        Runnable oldWay = new Runnable() {
            @Override
            public void run() {
                System.out.println("Running");
            }
        };

        // With lambda
        Runnable newWay = () -> System.out.println("Running");

        // Comparator without lambda
        Comparator<String> oldComparator = new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return s1.compareTo(s2);
            }
        };

        // With lambda
        Comparator<String> newComparator = (s1, s2) -> s1.compareTo(s2);

        // Multi-line lambda
        Comparator<String> multiLine = (s1, s2) -> {
            System.out.println("Comparing: " + s1 + " and " + s2);
            return s1.compareTo(s2);
        };

        // Using lambdas
        List<String> names = Arrays.asList("Charlie", "Alice", "Bob");

        // Sort
        names.sort((a, b) -> a.compareTo(b));

        // forEach
        names.forEach(name -> System.out.println(name));
        names.forEach(System.out::println);  // Method reference

        // removeIf
        names.removeIf(name -> name.startsWith("A"));

        // replaceAll
        names.replaceAll(name -> name.toUpperCase());
    }
}
```

### Variable Capture

```java
public class VariableCapture {
    public static void main(String[] args) {
        // Local variables must be effectively final
        String prefix = "Hello, ";
        // prefix = "Hi, ";  // Would cause compilation error

        Consumer<String> greeter = name -> System.out.println(prefix + name);
        greeter.accept("World");

        // Instance variables can be modified
        Counter counter = new Counter();
        Runnable r = () -> counter.count++;  // OK - instance variable

        // Workaround for modifying local variable
        int[] count = {0};  // Array is effectively final
        Runnable incrementer = () -> count[0]++;
        incrementer.run();
        System.out.println(count[0]);  // 1
    }
}

class Counter {
    int count = 0;
}
```

### Lambda Scoping

```java
public class LambdaScoping {
    private String instanceVar = "instance";

    public void demo() {
        String localVar = "local";

        Consumer<String> consumer = s -> {
            // Access parameter
            System.out.println(s);

            // Access local variable (must be effectively final)
            System.out.println(localVar);

            // Access instance variable
            System.out.println(instanceVar);

            // Access 'this' (refers to enclosing class, not lambda)
            System.out.println(this.instanceVar);
        };
    }
}
```

---

## Functional Interfaces

### What is a Functional Interface?

A **functional interface** is an interface with exactly **one abstract method**. It can have multiple default or static methods.

```java
@FunctionalInterface  // Optional but recommended
public interface Calculator {
    int calculate(int a, int b);  // Single abstract method

    // Default and static methods allowed
    default void printInfo() {
        System.out.println("Calculator");
    }

    static Calculator getAdder() {
        return (a, b) -> a + b;
    }
}
```

### Built-in Functional Interfaces (java.util.function)

```java
public class BuiltInFunctionalInterfaces {
    public static void main(String[] args) {
        // 1. Predicate<T> - T -> boolean
        Predicate<String> isEmpty = s -> s.isEmpty();
        Predicate<Integer> isPositive = n -> n > 0;

        System.out.println(isEmpty.test(""));  // true

        // Predicate combinations
        Predicate<Integer> isEven = n -> n % 2 == 0;
        Predicate<Integer> isPositiveEven = isPositive.and(isEven);
        Predicate<Integer> isNegativeOrOdd = isPositive.negate().or(isEven.negate());

        // 2. Function<T, R> - T -> R
        Function<String, Integer> length = s -> s.length();
        Function<Integer, Integer> doubler = n -> n * 2;

        System.out.println(length.apply("Hello"));  // 5

        // Function composition
        Function<String, Integer> doubleLength = length.andThen(doubler);  // length then double
        Function<String, Integer> same = doubler.compose(length);  // length then double

        // 3. Consumer<T> - T -> void
        Consumer<String> printer = s -> System.out.println(s);
        Consumer<String> uppercasePrinter = s -> System.out.println(s.toUpperCase());

        printer.accept("Hello");

        // Consumer chaining
        Consumer<String> combined = printer.andThen(uppercasePrinter);
        combined.accept("Hello");  // Prints "Hello" then "HELLO"

        // 4. Supplier<T> - () -> T
        Supplier<Double> randomSupplier = () -> Math.random();
        Supplier<List<String>> listSupplier = ArrayList::new;

        System.out.println(randomSupplier.get());

        // 5. BiFunction<T, U, R> - (T, U) -> R
        BiFunction<Integer, Integer, Integer> adder = (a, b) -> a + b;
        System.out.println(adder.apply(5, 3));  // 8

        // 6. BiPredicate<T, U> - (T, U) -> boolean
        BiPredicate<String, Integer> hasLength = (s, len) -> s.length() == len;
        System.out.println(hasLength.test("Hello", 5));  // true

        // 7. BiConsumer<T, U> - (T, U) -> void
        BiConsumer<String, Integer> printRepeat = (s, n) -> {
            for (int i = 0; i < n; i++) System.out.print(s);
        };
        printRepeat.accept("Hi", 3);  // HiHiHi

        // 8. UnaryOperator<T> - T -> T (extends Function<T, T>)
        UnaryOperator<String> toUpper = s -> s.toUpperCase();
        System.out.println(toUpper.apply("hello"));  // HELLO

        // 9. BinaryOperator<T> - (T, T) -> T (extends BiFunction<T, T, T>)
        BinaryOperator<Integer> sum = (a, b) -> a + b;
        System.out.println(sum.apply(5, 3));  // 8
    }
}
```

### Primitive Functional Interfaces

```java
public class PrimitiveFunctionalInterfaces {
    public static void main(String[] args) {
        // Avoid boxing/unboxing overhead

        // IntPredicate, LongPredicate, DoublePredicate
        IntPredicate isEven = n -> n % 2 == 0;

        // IntFunction<R>, LongFunction<R>, DoubleFunction<R>
        IntFunction<String> intToString = n -> "Number: " + n;

        // ToIntFunction<T>, ToLongFunction<T>, ToDoubleFunction<T>
        ToIntFunction<String> stringLength = s -> s.length();

        // IntConsumer, LongConsumer, DoubleConsumer
        IntConsumer printInt = n -> System.out.println(n);

        // IntSupplier, LongSupplier, DoubleSupplier
        IntSupplier randomInt = () -> (int)(Math.random() * 100);

        // IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator
        IntUnaryOperator square = n -> n * n;

        // IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator
        IntBinaryOperator add = (a, b) -> a + b;

        // ObjIntConsumer<T>, ObjLongConsumer<T>, ObjDoubleConsumer<T>
        ObjIntConsumer<String> printWithIndex = (s, i) -> System.out.println(i + ": " + s);
    }
}
```

### Functional Interface Summary

| Interface           | Method | Signature   |
| ------------------- | ------ | ----------- |
| `Predicate<T>`      | test   | T → boolean |
| `Function<T,R>`     | apply  | T → R       |
| `Consumer<T>`       | accept | T → void    |
| `Supplier<T>`       | get    | () → T      |
| `BiFunction<T,U,R>` | apply  | (T, U) → R  |
| `UnaryOperator<T>`  | apply  | T → T       |
| `BinaryOperator<T>` | apply  | (T, T) → T  |

---

## Method References

### Types of Method References

```java
public class MethodReferences {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

        // 1. Static method reference
        // Class::staticMethod
        Function<String, Integer> parser = Integer::parseInt;
        int num = parser.apply("123");  // Same as s -> Integer.parseInt(s)

        // 2. Instance method of particular object
        // object::instanceMethod
        String prefix = "Hello, ";
        Function<String, String> greeter = prefix::concat;
        // Same as s -> prefix.concat(s)

        // 3. Instance method of arbitrary object of particular type
        // Class::instanceMethod
        Function<String, String> toUpper = String::toUpperCase;
        // Same as s -> s.toUpperCase()

        BiPredicate<String, String> startsWith = String::startsWith;
        // Same as (s1, s2) -> s1.startsWith(s2)

        // 4. Constructor reference
        // Class::new
        Supplier<List<String>> listCreator = ArrayList::new;
        Function<Integer, List<String>> listWithSize = ArrayList::new;

        // Array constructor reference
        IntFunction<int[]> arrayCreator = int[]::new;
        int[] arr = arrayCreator.apply(5);  // new int[5]
    }
}
```

### Method Reference Examples

```java
public class MethodReferenceExamples {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Charlie", "Alice", "Bob");

        // Lambda vs Method Reference
        names.forEach(name -> System.out.println(name));
        names.forEach(System.out::println);  // Equivalent

        names.sort((a, b) -> a.compareToIgnoreCase(b));
        names.sort(String::compareToIgnoreCase);  // Equivalent

        // Stream operations
        List<String> upper = names.stream()
            .map(String::toUpperCase)
            .collect(Collectors.toList());

        // Constructor reference with streams
        List<Person> people = names.stream()
            .map(Person::new)  // Constructor that takes String
            .collect(Collectors.toList());

        // Method reference to instance method
        Printer printer = new Printer();
        names.forEach(printer::print);

        // Method reference to static method
        names.stream()
            .filter(MethodReferenceExamples::isNotEmpty)
            .forEach(System.out::println);
    }

    private static boolean isNotEmpty(String s) {
        return !s.isEmpty();
    }
}

class Person {
    private String name;
    public Person(String name) { this.name = name; }
}

class Printer {
    public void print(String s) { System.out.println(s); }
}
```

---

## Stream API

### What is a Stream?

A **Stream** is a sequence of elements supporting sequential and parallel aggregate operations. Streams don't store data; they carry values from a source through a pipeline of operations.

### Stream Characteristics

1. **Not a data structure** - Doesn't store elements
2. **Functional** - Operations don't modify the source
3. **Lazy** - Intermediate operations are lazy
4. **Possibly unbounded** - Can be infinite
5. **Consumable** - Elements can only be visited once

### Creating Streams

```java
public class StreamCreation {
    public static void main(String[] args) {
        // From Collection
        List<String> list = Arrays.asList("a", "b", "c");
        Stream<String> streamFromList = list.stream();
        Stream<String> parallelStream = list.parallelStream();

        // From Array
        String[] array = {"a", "b", "c"};
        Stream<String> streamFromArray = Arrays.stream(array);
        Stream<String> streamOf = Stream.of("a", "b", "c");

        // Empty stream
        Stream<String> emptyStream = Stream.empty();

        // Infinite streams
        Stream<Integer> infinite = Stream.iterate(0, n -> n + 2);  // 0, 2, 4, ...
        Stream<Double> randoms = Stream.generate(Math::random);

        // Java 9+ iterate with predicate
        Stream<Integer> limited = Stream.iterate(0, n -> n < 100, n -> n + 10);

        // From primitives
        IntStream intStream = IntStream.range(1, 5);      // 1, 2, 3, 4
        IntStream intStreamClosed = IntStream.rangeClosed(1, 5);  // 1, 2, 3, 4, 5
        LongStream longStream = LongStream.of(1L, 2L, 3L);
        DoubleStream doubleStream = DoubleStream.of(1.0, 2.0, 3.0);

        // From String
        IntStream chars = "Hello".chars();

        // From files (Java NIO)
        // Stream<String> lines = Files.lines(Paths.get("file.txt"));

        // Builder pattern
        Stream<String> built = Stream.<String>builder()
            .add("a")
            .add("b")
            .add("c")
            .build();

        // Concatenate streams
        Stream<String> concat = Stream.concat(
            Stream.of("a", "b"),
            Stream.of("c", "d")
        );
    }
}
```

### Intermediate Operations (Lazy)

```java
public class IntermediateOperations {
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
            new Person("Alice", 30),
            new Person("Bob", 25),
            new Person("Charlie", 35),
            new Person("Diana", 25)
        );

        // filter - keep elements matching predicate
        Stream<Person> adults = people.stream()
            .filter(p -> p.getAge() >= 30);

        // map - transform elements
        Stream<String> names = people.stream()
            .map(Person::getName);

        // mapToInt, mapToLong, mapToDouble - to primitive streams
        IntStream ages = people.stream()
            .mapToInt(Person::getAge);

        // flatMap - one-to-many transformation
        List<List<Integer>> nested = Arrays.asList(
            Arrays.asList(1, 2),
            Arrays.asList(3, 4, 5)
        );
        Stream<Integer> flattened = nested.stream()
            .flatMap(List::stream);  // 1, 2, 3, 4, 5

        // distinct - remove duplicates
        Stream<Integer> distinct = Stream.of(1, 2, 2, 3, 3, 3)
            .distinct();  // 1, 2, 3

        // sorted - sort elements
        Stream<Person> sortedByName = people.stream()
            .sorted(Comparator.comparing(Person::getName));

        // sorted - natural order
        Stream<Integer> sortedNums = Stream.of(3, 1, 2)
            .sorted();  // 1, 2, 3

        // peek - debug/side effects (don't modify!)
        Stream<String> peeked = people.stream()
            .peek(p -> System.out.println("Processing: " + p))
            .map(Person::getName);

        // limit - truncate stream
        Stream<Integer> limited = Stream.iterate(0, n -> n + 1)
            .limit(5);  // 0, 1, 2, 3, 4

        // skip - skip first n elements
        Stream<Integer> skipped = Stream.of(1, 2, 3, 4, 5)
            .skip(2);  // 3, 4, 5

        // takeWhile (Java 9+) - take while predicate is true
        Stream<Integer> taken = Stream.of(1, 2, 3, 4, 5, 1, 2)
            .takeWhile(n -> n < 4);  // 1, 2, 3

        // dropWhile (Java 9+) - drop while predicate is true
        Stream<Integer> dropped = Stream.of(1, 2, 3, 4, 5, 1, 2)
            .dropWhile(n -> n < 4);  // 4, 5, 1, 2
    }
}

class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
}
```

### Terminal Operations

```java
public class TerminalOperations {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // forEach - perform action on each element
        numbers.stream().forEach(System.out::println);

        // forEachOrdered - preserve encounter order in parallel
        numbers.parallelStream().forEachOrdered(System.out::println);

        // collect - accumulate into collection
        List<Integer> collected = numbers.stream()
            .filter(n -> n > 2)
            .collect(Collectors.toList());

        // toArray - convert to array
        Integer[] array = numbers.stream().toArray(Integer[]::new);

        // reduce - combine elements
        Optional<Integer> sum = numbers.stream()
            .reduce((a, b) -> a + b);

        int sumWithIdentity = numbers.stream()
            .reduce(0, Integer::sum);

        // count - count elements
        long count = numbers.stream().count();

        // min, max - find extremes
        Optional<Integer> min = numbers.stream().min(Integer::compareTo);
        Optional<Integer> max = numbers.stream().max(Integer::compareTo);

        // findFirst - find first element
        Optional<Integer> first = numbers.stream()
            .filter(n -> n > 3)
            .findFirst();

        // findAny - find any element (good for parallel)
        Optional<Integer> any = numbers.parallelStream()
            .filter(n -> n > 3)
            .findAny();

        // allMatch, anyMatch, noneMatch - predicates
        boolean allPositive = numbers.stream().allMatch(n -> n > 0);
        boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);
        boolean noNegative = numbers.stream().noneMatch(n -> n < 0);
    }
}
```

### Collectors

```java
public class CollectorsDemo {
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
            new Person("Alice", 30, "Engineering"),
            new Person("Bob", 25, "Sales"),
            new Person("Charlie", 35, "Engineering"),
            new Person("Diana", 25, "Sales")
        );

        // toList, toSet
        List<String> names = people.stream()
            .map(Person::getName)
            .collect(Collectors.toList());

        Set<String> uniqueNames = people.stream()
            .map(Person::getName)
            .collect(Collectors.toSet());

        // toCollection - specific collection
        TreeSet<String> sortedNames = people.stream()
            .map(Person::getName)
            .collect(Collectors.toCollection(TreeSet::new));

        // toMap
        Map<String, Integer> nameToAge = people.stream()
            .collect(Collectors.toMap(
                Person::getName,
                Person::getAge
            ));

        // toMap with merge function (handle duplicates)
        Map<Integer, String> ageToNames = people.stream()
            .collect(Collectors.toMap(
                Person::getAge,
                Person::getName,
                (name1, name2) -> name1 + ", " + name2
            ));

        // joining
        String joined = people.stream()
            .map(Person::getName)
            .collect(Collectors.joining(", ", "[", "]"));
        // "[Alice, Bob, Charlie, Diana]"

        // counting
        long count = people.stream()
            .collect(Collectors.counting());

        // summingInt, summingLong, summingDouble
        int totalAge = people.stream()
            .collect(Collectors.summingInt(Person::getAge));

        // averagingInt, averagingLong, averagingDouble
        double avgAge = people.stream()
            .collect(Collectors.averagingInt(Person::getAge));

        // summarizingInt - all statistics
        IntSummaryStatistics stats = people.stream()
            .collect(Collectors.summarizingInt(Person::getAge));
        // stats.getSum(), stats.getAverage(), stats.getMin(), stats.getMax(), stats.getCount()

        // maxBy, minBy
        Optional<Person> oldest = people.stream()
            .collect(Collectors.maxBy(Comparator.comparingInt(Person::getAge)));

        // groupingBy
        Map<String, List<Person>> byDept = people.stream()
            .collect(Collectors.groupingBy(Person::getDepartment));

        // groupingBy with downstream collector
        Map<String, Long> countByDept = people.stream()
            .collect(Collectors.groupingBy(
                Person::getDepartment,
                Collectors.counting()
            ));

        Map<String, Double> avgAgeByDept = people.stream()
            .collect(Collectors.groupingBy(
                Person::getDepartment,
                Collectors.averagingInt(Person::getAge)
            ));

        // partitioningBy (group by boolean)
        Map<Boolean, List<Person>> partitioned = people.stream()
            .collect(Collectors.partitioningBy(p -> p.getAge() >= 30));

        // collectingAndThen - transform result
        List<String> unmodifiable = people.stream()
            .map(Person::getName)
            .collect(Collectors.collectingAndThen(
                Collectors.toList(),
                Collections::unmodifiableList
            ));

        // mapping
        Map<String, List<String>> namesByDept = people.stream()
            .collect(Collectors.groupingBy(
                Person::getDepartment,
                Collectors.mapping(
                    Person::getName,
                    Collectors.toList()
                )
            ));

        // reducing
        Optional<Integer> sumAges = people.stream()
            .collect(Collectors.reducing(
                0,
                Person::getAge,
                Integer::sum
            ));
    }
}

class Person {
    private String name;
    private int age;
    private String department;

    public Person(String name, int age, String department) {
        this.name = name;
        this.age = age;
        this.department = department;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
    public String getDepartment() { return department; }
}
```

### Parallel Streams

```java
public class ParallelStreams {
    public static void main(String[] args) {
        List<Integer> numbers = IntStream.rangeClosed(1, 1000000)
            .boxed()
            .collect(Collectors.toList());

        // Create parallel stream
        long sum1 = numbers.parallelStream()
            .mapToLong(Integer::longValue)
            .sum();

        // Convert to parallel
        long sum2 = numbers.stream()
            .parallel()
            .mapToLong(Integer::longValue)
            .sum();

        // Convert back to sequential
        long sum3 = numbers.parallelStream()
            .sequential()
            .mapToLong(Integer::longValue)
            .sum();

        // Check if parallel
        boolean isParallel = numbers.parallelStream().isParallel();

        // Parallel stream considerations:
        // 1. Use for CPU-intensive operations
        // 2. Data should be large enough
        // 3. Source should be efficiently splittable (ArrayList > LinkedList)
        // 4. Avoid stateful operations
        // 5. Use forEachOrdered if order matters
    }
}
```

---

## Optional Class

### Creating Optional

```java
public class OptionalCreation {
    public static void main(String[] args) {
        // Create with value (throws NPE if null)
        Optional<String> opt1 = Optional.of("Hello");

        // Create with possible null
        Optional<String> opt2 = Optional.ofNullable(null);  // Empty
        Optional<String> opt3 = Optional.ofNullable("World");

        // Create empty
        Optional<String> empty = Optional.empty();
    }
}
```

### Using Optional

```java
public class OptionalUsage {
    public static void main(String[] args) {
        Optional<String> optional = Optional.of("Hello");

        // Check presence
        if (optional.isPresent()) {
            System.out.println(optional.get());
        }

        // isEmpty (Java 11+)
        if (optional.isEmpty()) {
            System.out.println("Empty!");
        }

        // ifPresent - consume if present
        optional.ifPresent(System.out::println);

        // ifPresentOrElse (Java 9+)
        optional.ifPresentOrElse(
            System.out::println,
            () -> System.out.println("Empty!")
        );

        // orElse - default value
        String value = optional.orElse("Default");

        // orElseGet - lazy default (supplier)
        String lazyValue = optional.orElseGet(() -> computeDefault());

        // orElseThrow - throw if empty
        String throwValue = optional.orElseThrow();  // NoSuchElementException
        String customThrow = optional.orElseThrow(
            () -> new IllegalStateException("Value required")
        );

        // or (Java 9+) - alternative Optional
        Optional<String> alt = optional.or(() -> Optional.of("Alternative"));

        // map - transform value
        Optional<Integer> length = optional.map(String::length);

        // flatMap - for nested Optionals
        Optional<Optional<String>> nested = Optional.of(optional);
        Optional<String> flattened = optional.flatMap(s -> Optional.of(s.toUpperCase()));

        // filter - filter value
        Optional<String> filtered = optional.filter(s -> s.length() > 3);

        // stream (Java 9+) - to Stream
        Stream<String> stream = optional.stream();  // 0 or 1 element
    }

    private static String computeDefault() {
        System.out.println("Computing default...");
        return "Computed Default";
    }
}
```

### Optional Best Practices

```java
public class OptionalBestPractices {
    // DO: Use as return type
    public Optional<User> findUserById(Long id) {
        // Return Optional.empty() or Optional.of(user)
        return Optional.ofNullable(getUserFromDb(id));
    }

    // DON'T: Use as parameter
    // Bad: public void setName(Optional<String> name)
    // Good: public void setName(String name)

    // DON'T: Use for fields
    // Bad: private Optional<String> name;
    // Good: private String name;  // Can be null

    // DON'T: Use get() without checking
    // Bad: optional.get()
    // Good: optional.orElse("default")

    // DO: Chain operations
    public String getUserName(Long id) {
        return findUserById(id)
            .map(User::getName)
            .map(String::toUpperCase)
            .orElse("UNKNOWN");
    }

    // orElse vs orElseGet
    public void orElseVsOrElseGet() {
        Optional<String> opt = Optional.of("Value");

        // orElse - default always evaluated
        String v1 = opt.orElse(expensiveOperation());  // expensiveOperation() called!

        // orElseGet - default only evaluated if needed
        String v2 = opt.orElseGet(() -> expensiveOperation());  // NOT called!
    }

    private String expensiveOperation() {
        System.out.println("Expensive operation!");
        return "Expensive Result";
    }

    private User getUserFromDb(Long id) { return null; }
}

class User {
    private String name;
    public String getName() { return name; }
}
```

---

## Date and Time API

### Overview

Java 8 introduced `java.time` package as a replacement for `java.util.Date` and `java.util.Calendar`.

### Key Classes

```java
import java.time.*;
import java.time.format.*;
import java.time.temporal.*;

public class DateTimeAPI {
    public static void main(String[] args) {
        // LocalDate - date without time
        LocalDate today = LocalDate.now();
        LocalDate birthday = LocalDate.of(1990, Month.JANUARY, 15);
        LocalDate parsed = LocalDate.parse("2023-12-25");

        // LocalTime - time without date
        LocalTime now = LocalTime.now();
        LocalTime noon = LocalTime.of(12, 0);
        LocalTime precise = LocalTime.of(12, 30, 45, 123456789);

        // LocalDateTime - date and time without timezone
        LocalDateTime dateTime = LocalDateTime.now();
        LocalDateTime specific = LocalDateTime.of(2023, 12, 25, 10, 30);
        LocalDateTime combined = LocalDateTime.of(birthday, noon);

        // ZonedDateTime - date and time with timezone
        ZonedDateTime zonedNow = ZonedDateTime.now();
        ZonedDateTime newYork = ZonedDateTime.now(ZoneId.of("America/New_York"));
        ZonedDateTime tokyo = zonedNow.withZoneSameInstant(ZoneId.of("Asia/Tokyo"));

        // Instant - point in time (epoch-based)
        Instant instant = Instant.now();
        Instant epoch = Instant.EPOCH;  // 1970-01-01T00:00:00Z
        Instant fromEpoch = Instant.ofEpochMilli(1000000000000L);

        // Duration - time-based amount
        Duration hours = Duration.ofHours(5);
        Duration between = Duration.between(LocalTime.NOON, LocalTime.MIDNIGHT);

        // Period - date-based amount
        Period months = Period.ofMonths(3);
        Period years = Period.between(birthday, today);
    }
}
```

### Manipulating Dates and Times

```java
public class DateTimeManipulation {
    public static void main(String[] args) {
        LocalDate date = LocalDate.of(2023, 6, 15);
        LocalTime time = LocalTime.of(10, 30);
        LocalDateTime dateTime = LocalDateTime.of(date, time);

        // Getting components
        int year = date.getYear();
        Month month = date.getMonth();
        int dayOfMonth = date.getDayOfMonth();
        DayOfWeek dayOfWeek = date.getDayOfWeek();
        int dayOfYear = date.getDayOfYear();

        // Adding/Subtracting (returns new instance - immutable!)
        LocalDate tomorrow = date.plusDays(1);
        LocalDate nextMonth = date.plusMonths(1);
        LocalDate lastYear = date.minusYears(1);
        LocalDate plusWeeks = date.plus(2, ChronoUnit.WEEKS);

        LocalTime later = time.plusHours(3);
        LocalTime earlier = time.minusMinutes(45);

        // Modifying specific fields
        LocalDate modified = date.withYear(2024)
                                 .withMonth(12)
                                 .withDayOfMonth(25);

        // Temporal adjusters
        LocalDate firstDayOfMonth = date.with(TemporalAdjusters.firstDayOfMonth());
        LocalDate lastDayOfMonth = date.with(TemporalAdjusters.lastDayOfMonth());
        LocalDate nextMonday = date.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
        LocalDate firstDayOfNextYear = date.with(TemporalAdjusters.firstDayOfNextYear());

        // Comparing
        boolean isBefore = date.isBefore(LocalDate.now());
        boolean isAfter = date.isAfter(LocalDate.now());
        boolean isEqual = date.isEqual(LocalDate.now());

        // Leap year check
        boolean isLeapYear = date.isLeapYear();

        // Length
        int lengthOfMonth = date.lengthOfMonth();  // Days in month
        int lengthOfYear = date.lengthOfYear();    // 365 or 366
    }
}
```

### Formatting and Parsing

```java
public class DateTimeFormatting {
    public static void main(String[] args) {
        LocalDateTime dateTime = LocalDateTime.of(2023, 12, 25, 10, 30, 45);

        // Predefined formatters
        String iso = dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);
        String isoDate = dateTime.format(DateTimeFormatter.ISO_DATE);

        // Pattern-based formatting
        DateTimeFormatter custom = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
        String formatted = dateTime.format(custom);  // "25/12/2023 10:30:45"

        // Localized formatting
        DateTimeFormatter localized = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM)
                                                       .withLocale(Locale.US);
        String localizedStr = dateTime.format(localized);

        // Parsing
        LocalDate parsed = LocalDate.parse("2023-12-25");
        LocalDate parsedCustom = LocalDate.parse("25/12/2023",
                                                 DateTimeFormatter.ofPattern("dd/MM/yyyy"));
        LocalDateTime parsedDateTime = LocalDateTime.parse("25/12/2023 10:30:45", custom);

        // Common patterns
        // y - year         M - month        d - day
        // H - hour (24)    h - hour (12)    m - minute        s - second
        // E - day name     a - AM/PM        z - timezone

        DateTimeFormatter fancy = DateTimeFormatter.ofPattern("EEEE, MMMM d, yyyy 'at' h:mm a");
        System.out.println(dateTime.format(fancy));
        // "Monday, December 25, 2023 at 10:30 AM"
    }
}
```

### Time Zones

```java
public class TimeZones {
    public static void main(String[] args) {
        // List all zones
        Set<String> zones = ZoneId.getAvailableZoneIds();

        // Create ZoneId
        ZoneId newYork = ZoneId.of("America/New_York");
        ZoneId utc = ZoneId.of("UTC");
        ZoneId systemDefault = ZoneId.systemDefault();

        // ZonedDateTime operations
        ZonedDateTime nowNY = ZonedDateTime.now(newYork);
        ZonedDateTime nowUTC = ZonedDateTime.now(utc);

        // Convert between zones
        ZonedDateTime tokyoTime = nowNY.withZoneSameInstant(ZoneId.of("Asia/Tokyo"));

        // Keep local time, change zone
        ZonedDateTime sameLocal = nowNY.withZoneSameLocal(ZoneId.of("Europe/London"));

        // OffsetDateTime - fixed offset from UTC
        OffsetDateTime offset = OffsetDateTime.of(
            LocalDateTime.now(),
            ZoneOffset.ofHours(5)
        );

        // Daylight saving time handling
        ZonedDateTime beforeDST = ZonedDateTime.of(
            LocalDateTime.of(2023, 3, 12, 1, 30),
            ZoneId.of("America/New_York")
        );
        ZonedDateTime afterDST = beforeDST.plusHours(2);
        // DST gap is automatically handled
    }
}
```

### Legacy Conversion

```java
public class LegacyConversion {
    public static void main(String[] args) {
        // java.util.Date <-> Instant
        Date date = new Date();
        Instant instant = date.toInstant();
        Date backToDate = Date.from(instant);

        // java.util.Date <-> LocalDateTime
        LocalDateTime ldt = LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
        Date backFromLdt = Date.from(ldt.atZone(ZoneId.systemDefault()).toInstant());

        // java.sql.Date <-> LocalDate
        java.sql.Date sqlDate = java.sql.Date.valueOf(LocalDate.now());
        LocalDate localDate = sqlDate.toLocalDate();

        // java.sql.Timestamp <-> LocalDateTime
        java.sql.Timestamp timestamp = java.sql.Timestamp.valueOf(LocalDateTime.now());
        LocalDateTime fromTimestamp = timestamp.toLocalDateTime();

        // Calendar <-> ZonedDateTime
        Calendar calendar = Calendar.getInstance();
        ZonedDateTime zdt = ZonedDateTime.ofInstant(calendar.toInstant(),
                                                     calendar.getTimeZone().toZoneId());
    }
}
```

---

## Default and Static Methods in Interfaces

### Default Methods

```java
public interface Vehicle {
    void start();
    void stop();

    // Default method - provides implementation
    default void horn() {
        System.out.println("Beep!");
    }

    default void info() {
        System.out.println("This is a vehicle");
    }
}

public interface Electric {
    default void info() {
        System.out.println("This is electric");
    }
}

// Diamond problem resolution
public class ElectricCar implements Vehicle, Electric {
    @Override
    public void start() {
        System.out.println("Starting electric car");
    }

    @Override
    public void stop() {
        System.out.println("Stopping electric car");
    }

    // Must override when multiple interfaces have same default method
    @Override
    public void info() {
        Vehicle.super.info();  // Call specific interface's default
        Electric.super.info();
        System.out.println("This is an electric car");
    }
}
```

### Static Methods

```java
public interface Calculator {
    int calculate(int a, int b);

    // Static method
    static Calculator getAdder() {
        return (a, b) -> a + b;
    }

    static Calculator getMultiplier() {
        return (a, b) -> a * b;
    }

    // Static helper method
    static int add(int a, int b) {
        return a + b;
    }
}

// Usage
public class StaticMethodDemo {
    public static void main(String[] args) {
        Calculator adder = Calculator.getAdder();
        System.out.println(adder.calculate(5, 3));  // 8

        int sum = Calculator.add(5, 3);  // 8
    }
}
```

### Private Methods (Java 9+)

```java
public interface ModernInterface {
    default void method1() {
        helper();
        staticHelper();
    }

    default void method2() {
        helper();
    }

    // Private instance method (Java 9+)
    private void helper() {
        System.out.println("Helper method");
    }

    // Private static method (Java 9+)
    private static void staticHelper() {
        System.out.println("Static helper");
    }
}
```

---

## Java 9-17 Features

### Java 9 Features

```java
// 1. Collection factory methods
List<String> list = List.of("a", "b", "c");  // Immutable
Set<Integer> set = Set.of(1, 2, 3);
Map<String, Integer> map = Map.of("a", 1, "b", 2);
Map<String, Integer> mapEntries = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2)
);

// 2. Stream improvements
Stream.of(1, 2, 3, 4, 5)
    .takeWhile(n -> n < 4)      // 1, 2, 3
    .forEach(System.out::println);

Stream.of(1, 2, 3, 4, 5, 1, 2)
    .dropWhile(n -> n < 4)      // 4, 5, 1, 2
    .forEach(System.out::println);

Stream.ofNullable(null);  // Empty stream instead of NPE

Stream.iterate(0, n -> n < 10, n -> n + 1);  // With predicate

// 3. Optional improvements
Optional.of("value").ifPresentOrElse(
    System.out::println,
    () -> System.out.println("Empty")
);

Optional.empty().or(() -> Optional.of("fallback"));
Optional.of("value").stream();  // Stream of 0 or 1 element

// 4. Interface private methods
// (shown in previous section)

// 5. Try-with-resources improvement
BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
try (reader) {  // Effectively final variable allowed
    // Use reader
}
```

### Java 10 Features

```java
// 1. Local variable type inference (var)
var list = new ArrayList<String>();  // Type inferred as ArrayList<String>
var map = new HashMap<String, Integer>();

// var in for loops
for (var item : list) {
    System.out.println(item);
}

// var in try-with-resources
try (var reader = new BufferedReader(new FileReader("file.txt"))) {
    // ...
}

// Limitations:
// - Must have initializer
// - Cannot be null
// - Not for fields, parameters, or return types
// - Not for lambda parameters (until Java 11)

// 2. Unmodifiable collections from stream
List<Integer> unmodifiable = Stream.of(1, 2, 3)
    .collect(Collectors.toUnmodifiableList());

// 3. Optional.orElseThrow()
String value = Optional.of("value").orElseThrow();  // No argument version
```

### Java 11 Features

```java
// 1. var in lambda parameters
BiFunction<String, String, String> concat = (var a, var b) -> a + b;

// 2. String methods
String str = "  Hello World  ";
str.isBlank();           // false
str.strip();             // "Hello World" (Unicode-aware trim)
str.stripLeading();      // "Hello World  "
str.stripTrailing();     // "  Hello World"
str.lines();             // Stream of lines
str.repeat(3);           // "  Hello World    Hello World    Hello World  "

"".isBlank();            // true
"   ".isBlank();         // true

// 3. File methods
String content = Files.readString(Path.of("file.txt"));
Files.writeString(Path.of("output.txt"), "content");

// 4. Optional isEmpty()
Optional.empty().isEmpty();  // true

// 5. HTTP Client (standardized)
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .GET()
    .build();
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

// 6. Collection to array
List<String> list = List.of("a", "b", "c");
String[] array = list.toArray(String[]::new);  // Improved syntax
```

### Java 12-14 Features

```java
// 1. Switch expressions (preview 12, standard 14)
int day = 3;
String dayName = switch (day) {
    case 1, 2, 3, 4, 5 -> "Weekday";
    case 6, 7 -> "Weekend";
    default -> "Invalid";
};

// With yield for blocks
String result = switch (day) {
    case 1 -> "Monday";
    case 2 -> {
        System.out.println("Processing...");
        yield "Tuesday";  // yield for returning value from block
    }
    default -> "Other";
};

// 2. Text blocks (preview 13, standard 15)
String json = """
    {
        "name": "John",
        "age": 30,
        "city": "New York"
    }
    """;

String html = """
    <html>
        <body>
            <h1>Hello</h1>
        </body>
    </html>
    """;

// 3. Helpful NullPointerExceptions (14)
// a.b.c.d = 5;  // If b is null:
// "Cannot invoke ... because 'a.b' is null"

// 4. Records (preview 14, standard 16)
record Point(int x, int y) {
    // Compact constructor
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordinates must be positive");
        }
    }

    // Additional methods
    public double distance() {
        return Math.sqrt(x * x + y * y);
    }
}

Point p = new Point(3, 4);
int x = p.x();           // Accessor method
System.out.println(p);   // "Point[x=3, y=4]"
```

### Java 15-17 Features

```java
// 1. Sealed classes (preview 15, standard 17)
public sealed class Shape permits Circle, Rectangle, Triangle {
    // Common shape methods
}

public final class Circle extends Shape {
    // Cannot be extended further
}

public non-sealed class Rectangle extends Shape {
    // Can be freely extended
}

public sealed class Triangle extends Shape permits EquilateralTriangle {
    // Limited extension
}

// 2. Pattern matching for instanceof (preview 14, standard 16)
Object obj = "Hello";
if (obj instanceof String s) {
    System.out.println(s.length());  // s is String, no cast needed
}

// With logical operators
if (obj instanceof String s && s.length() > 5) {
    System.out.println(s);
}

// 3. Pattern matching for switch (preview 17)
Object o = 123;
String formatted = switch (o) {
    case Integer i -> "Integer: " + i;
    case Long l -> "Long: " + l;
    case Double d -> "Double: " + d;
    case String s -> "String: " + s;
    case null -> "null";
    default -> "Unknown";
};

// With guards (Java 21+)
String result = switch (o) {
    case Integer i when i > 100 -> "Large integer";
    case Integer i -> "Small integer: " + i;
    default -> "Not an integer";
};

// 4. Enhanced pseudo-random number generators (17)
RandomGenerator generator = RandomGenerator.of("L64X128MixRandom");
int random = generator.nextInt();

// 5. Foreign Function & Memory API (incubator)
// For calling native code and managing off-heap memory
```

---

## Interview Questions

### Q1: What is a lambda expression and when to use it?

**Answer:**
A lambda is an anonymous function implementing a functional interface.

```java
// Use for:
// 1. Short, single-use implementations
names.forEach(name -> System.out.println(name));

// 2. Callbacks
button.addActionListener(e -> handleClick());

// 3. Stream operations
list.stream().filter(x -> x > 5).collect(Collectors.toList());

// Don't use for:
// 1. Complex logic (use method reference or named class)
// 2. When you need to throw checked exceptions
// 3. When readability suffers
```

---

### Q2: What is the difference between map() and flatMap()?

```java
// map() - one-to-one transformation
List<String> names = people.stream()
    .map(Person::getName)      // Stream<String>
    .collect(Collectors.toList());

// flatMap() - one-to-many, flattens nested structures
List<List<String>> nested = List.of(List.of("a", "b"), List.of("c", "d"));
List<String> flat = nested.stream()
    .flatMap(List::stream)     // Stream<String> (flattened)
    .collect(Collectors.toList());  // [a, b, c, d]
```

---

### Q3: What is the difference between findFirst() and findAny()?

| findFirst()            | findAny()                     |
| ---------------------- | ----------------------------- |
| Returns first element  | Returns any element           |
| Deterministic          | Non-deterministic in parallel |
| Slower in parallel     | Faster in parallel            |
| Use when order matters | Use for existence check       |

---

### Q4: What are the advantages of the new Date/Time API?

1. **Immutable and thread-safe** - No synchronization needed
2. **Clear separation** - LocalDate vs LocalTime vs LocalDateTime
3. **Timezone support** - ZonedDateTime, ZoneId
4. **Fluent API** - Chaining methods
5. **No deprecated methods** - Unlike java.util.Date
6. **Consistent** - All in java.time package

---

### Q5: What is the output?

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
int sum = list.stream()
    .peek(System.out::println)
    .filter(n -> n > 2)
    .mapToInt(n -> n)
    .sum();
```

**Answer:**

```
1
2
3
4
5
```

Sum = 12 (3+4+5). All elements are peeked because `sum()` is terminal.

---

### Q6: Why should Optional.get() be avoided?

`get()` throws `NoSuchElementException` if empty. Use alternatives:

```java
// Bad
String value = optional.get();

// Good
String value = optional.orElse("default");
String value = optional.orElseThrow(() -> new CustomException());
optional.ifPresent(v -> process(v));
```

---

### Q7: What is the difference between Collectors.toList() and Stream.toList()?

| Collectors.toList()  | Stream.toList() (Java 16+) |
| -------------------- | -------------------------- |
| Modifiable list      | Unmodifiable list          |
| java.util.ArrayList  | Unknown implementation     |
| Allows null elements | Allows null elements       |
| More flexible        | More concise               |

---

### Q8: What is a record and when to use it?

```java
// Record is a compact immutable data carrier
record Person(String name, int age) {}

// Equivalent to writing:
// - private final fields
// - public accessor methods (name(), age())
// - equals(), hashCode(), toString()
// - All-args constructor

// Use for:
// - DTOs
// - Value objects
// - Immutable data carriers

// Don't use for:
// - Mutable classes
// - Classes with complex logic
// - Classes requiring inheritance
```

---

### Q9: What is the difference between intermediate and terminal operations?

| Intermediate                    | Terminal                        |
| ------------------------------- | ------------------------------- |
| Returns Stream                  | Returns value/void              |
| Lazy (not executed immediately) | Eager (triggers execution)      |
| Can be chained                  | Ends the pipeline               |
| filter, map, flatMap, sorted    | collect, forEach, reduce, count |

---

### Q10: Explain var keyword limitations.

```java
// Cannot use var:
// 1. Without initializer
var x;  // Error

// 2. With null
var y = null;  // Error

// 3. For fields
class C { var field = 1; }  // Error

// 4. For parameters
void method(var param) {}  // Error

// 5. For return types
var method() { return 1; }  // Error

// 6. With array initializer
var arr = {1, 2, 3};  // Error
var arr = new int[]{1, 2, 3};  // OK
```

---

## Key Takeaways

1. **Lambda + Functional Interfaces** - Core of modern Java
2. **Streams** - Declarative data processing
3. **Optional** - Null-safe programming
4. **Date/Time API** - Immutable, thread-safe dates
5. **Default methods** - Interface evolution
6. **var** - Type inference for local variables
7. **Records** - Immutable data carriers
8. **Sealed classes** - Controlled inheritance
9. **Pattern matching** - Cleaner type checking
10. **Text blocks** - Multi-line strings
