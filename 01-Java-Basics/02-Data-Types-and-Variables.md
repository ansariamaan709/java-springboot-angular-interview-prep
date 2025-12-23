# Data Types and Variables in Java - Complete Guide

## Table of Contents

1. [Introduction to Data Types](#introduction-to-data-types)
2. [Primitive Data Types](#primitive-data-types)
3. [Reference Data Types](#reference-data-types)
4. [Variables in Java](#variables-in-java)
5. [Type Casting and Conversion](#type-casting-and-conversion)
6. [Wrapper Classes](#wrapper-classes)
7. [Autoboxing and Unboxing](#autoboxing-and-unboxing)
8. [var Keyword (Java 10+)](#var-keyword-java-10)
9. [Interview Questions](#interview-questions)

---

## Introduction to Data Types

### What are Data Types?

Data types specify the **size and type of values** that can be stored in a variable. Java is a **strongly typed language**, meaning every variable must have a declared data type.

### Why Data Types Matter?

1. **Memory Allocation**: JVM knows how much memory to allocate
2. **Type Safety**: Prevents incorrect operations on data
3. **Performance**: Primitive types are more efficient
4. **Correctness**: Compiler catches type mismatches

### Categories of Data Types

```
                    Data Types in Java
                           │
          ┌────────────────┴────────────────┐
          │                                 │
    Primitive Types                  Reference Types
          │                                 │
    ┌─────┴─────┐                    ┌──────┴──────┐
    │           │                    │             │
  Numeric    Boolean               Classes     Interfaces
    │                                │
  ┌─┴─────────┐                    Arrays
  │           │                    Enums
Integer   Floating                 Strings
  │       Point
  │         │
byte      float
short     double
int
long
char
```

---

## Primitive Data Types

### Overview of All Primitive Types

| Type      | Size    | Range                   | Default  | Wrapper Class |
| --------- | ------- | ----------------------- | -------- | ------------- |
| `byte`    | 8 bits  | -128 to 127             | 0        | Byte          |
| `short`   | 16 bits | -32,768 to 32,767       | 0        | Short         |
| `int`     | 32 bits | -2³¹ to 2³¹-1           | 0        | Integer       |
| `long`    | 64 bits | -2⁶³ to 2⁶³-1           | 0L       | Long          |
| `float`   | 32 bits | ±3.4E38 (6-7 digits)    | 0.0f     | Float         |
| `double`  | 64 bits | ±1.7E308 (15-16 digits) | 0.0d     | Double        |
| `char`    | 16 bits | 0 to 65,535             | '\u0000' | Character     |
| `boolean` | 1 bit\* | true/false              | false    | Boolean       |

\*JVM implementation dependent, often uses 32 bits

### 1. Integer Types

#### byte (8 bits)

```java
public class ByteDemo {
    public static void main(String[] args) {
        byte minByte = -128;
        byte maxByte = 127;

        // Common use: Reading binary data, network protocols
        byte[] buffer = new byte[1024];

        // Overflow example
        byte overflow = 127;
        overflow++; // Becomes -128 (wraps around)
        System.out.println(overflow); // -128

        // Memory efficient for arrays
        byte[] imageData = new byte[1000000]; // 1MB
        int[] intData = new int[1000000];     // 4MB
    }
}
```

**When to use byte:**

- File I/O operations
- Network socket programming
- Image/audio processing
- Memory-critical applications with small values

#### short (16 bits)

```java
public class ShortDemo {
    public static void main(String[] args) {
        short minShort = -32768;
        short maxShort = 32767;

        // Rarely used in modern Java
        // Legacy compatibility, specific protocols

        // Array of ports (0-65535 won't fit, need int)
        short port = 8080; // OK for common ports
    }
}
```

**When to use short:**

- Rarely used in modern applications
- Legacy system integration
- Memory-critical with medium-range values

#### int (32 bits) - Most Common

```java
public class IntDemo {
    public static void main(String[] args) {
        int minInt = Integer.MIN_VALUE; // -2147483648
        int maxInt = Integer.MAX_VALUE; // 2147483647

        // Default for integer literals
        int count = 100;

        // Binary, octal, hexadecimal literals
        int binary = 0b1010;      // 10 in decimal
        int octal = 012;          // 10 in decimal
        int hex = 0xA;            // 10 in decimal

        // Underscores for readability (Java 7+)
        int million = 1_000_000;
        int creditCard = 1234_5678_9012_3456; // Won't fit, need long

        // Overflow example
        int overflow = Integer.MAX_VALUE;
        System.out.println(overflow + 1); // -2147483648 (wraps to MIN)
    }
}
```

**When to use int:**

- Default choice for integers
- Array indices
- Loop counters
- Most numerical calculations

#### long (64 bits)

```java
public class LongDemo {
    public static void main(String[] args) {
        long minLong = Long.MIN_VALUE;
        long maxLong = Long.MAX_VALUE;

        // Must use L suffix for long literals
        long bigNumber = 9999999999L; // L suffix required
        long anotherWay = 9999999999l; // lowercase l works but discouraged

        // Common use cases
        long timestamp = System.currentTimeMillis();
        long fileSize = 10_000_000_000L; // 10 GB
        long populationWorld = 8_000_000_000L;

        // Database IDs (especially in distributed systems)
        long userId = 1234567890123456789L;
    }
}
```

**When to use long:**

- Timestamps (milliseconds since epoch)
- File sizes
- Database IDs (especially auto-generated)
- Large counts exceeding int range
- Financial calculations (cents)

### 2. Floating-Point Types

#### float (32 bits)

```java
public class FloatDemo {
    public static void main(String[] args) {
        // Must use f suffix
        float pi = 3.14159f;
        float temperature = 98.6f;

        // Scientific notation
        float scientific = 1.5e10f;

        // Precision issues
        float f1 = 0.1f + 0.2f;
        System.out.println(f1); // 0.30000001 (not exactly 0.3)

        // Special values
        float positiveInfinity = Float.POSITIVE_INFINITY;
        float negativeInfinity = Float.NEGATIVE_INFINITY;
        float nan = Float.NaN;

        // NaN comparisons
        System.out.println(nan == nan);        // false
        System.out.println(Float.isNaN(nan));  // true
    }
}
```

#### double (64 bits) - Default for Decimals

```java
public class DoubleDemo {
    public static void main(String[] args) {
        // Default for floating-point literals
        double pi = 3.141592653589793;
        double e = 2.718281828459045;

        // Precision comparison
        float fPi = 3.14159265358979323846f;
        double dPi = 3.14159265358979323846;
        System.out.println(fPi);  // 3.1415927
        System.out.println(dPi);  // 3.141592653589793

        // Never use for money!
        double price = 0.1 + 0.2;
        System.out.println(price); // 0.30000000000000004

        // Use BigDecimal for money
        // BigDecimal price = new BigDecimal("0.1").add(new BigDecimal("0.2"));
    }
}
```

**Critical: Why NOT to use float/double for Money**

```java
public class MoneyProblem {
    public static void main(String[] args) {
        double total = 0.0;
        for (int i = 0; i < 10; i++) {
            total += 0.1;
        }
        System.out.println(total);        // 0.9999999999999999
        System.out.println(total == 1.0); // false

        // Correct way: Use BigDecimal or store cents as long
        long totalCents = 0;
        for (int i = 0; i < 10; i++) {
            totalCents += 10; // 10 cents
        }
        System.out.println(totalCents / 100.0); // 1.0
    }
}
```

### 3. char Type (16 bits)

```java
public class CharDemo {
    public static void main(String[] args) {
        // Character literal
        char letter = 'A';
        char digit = '5';
        char symbol = '@';

        // Unicode representation
        char unicode = '\u0041';  // 'A'
        char copyrightSymbol = '\u00A9';  // ©
        char rupeeSymbol = '\u20B9';  // ₹

        // char is numeric (unsigned 16-bit)
        char c = 65;  // 'A'
        int ascii = 'A';  // 65

        // Arithmetic with char
        char a = 'A';
        a++;
        System.out.println(a);  // 'B'

        // Escape sequences
        char newline = '\n';
        char tab = '\t';
        char backslash = '\\';
        char singleQuote = '\'';
        char doubleQuote = '\"';

        // char to int conversion
        char ch = '7';
        int num = ch - '0';  // Convert char digit to int: 7
    }
}
```

### 4. boolean Type

```java
public class BooleanDemo {
    public static void main(String[] args) {
        boolean isJavaFun = true;
        boolean isCodingEasy = false;

        // Cannot be converted to/from numeric types
        // boolean b = 1;  // Compilation error
        // int i = true;   // Compilation error

        // Boolean expressions
        boolean result = (10 > 5);  // true
        boolean complex = (10 > 5) && (20 < 30);  // true

        // Common mistake from C/C++ programmers
        int x = 5;
        // if (x = 10) { }  // Error in Java, assignment returns int, not boolean
        if (x == 10) { }    // Correct
    }
}
```

---

## Reference Data Types

### Characteristics of Reference Types

```java
public class ReferenceDemo {
    public static void main(String[] args) {
        // Reference variables store memory addresses
        String str = "Hello";  // str stores address of String object
        int[] arr = {1, 2, 3}; // arr stores address of array object

        // Default value is null
        String nullStr = null;
        Integer nullInt = null;

        // Reference comparison vs value comparison
        String s1 = new String("Hello");
        String s2 = new String("Hello");
        System.out.println(s1 == s2);       // false (different addresses)
        System.out.println(s1.equals(s2));  // true (same content)

        // String pool optimization
        String s3 = "Hello";
        String s4 = "Hello";
        System.out.println(s3 == s4);  // true (same pool reference)
    }
}
```

### Reference Types Categories

1. **Class Types**

```java
String name = "John";
Date today = new Date();
ArrayList<String> list = new ArrayList<>();
```

2. **Interface Types**

```java
List<String> list = new ArrayList<>();
Runnable task = () -> System.out.println("Running");
```

3. **Array Types**

```java
int[] numbers = {1, 2, 3};
String[] names = new String[10];
int[][] matrix = new int[3][3];
```

4. **Enum Types**

```java
enum Day { MONDAY, TUESDAY, WEDNESDAY }
Day today = Day.MONDAY;
```

### Primitive vs Reference Types

| Aspect     | Primitive          | Reference                        |
| ---------- | ------------------ | -------------------------------- |
| Storage    | Stack (usually)    | Heap (object), Stack (reference) |
| Default    | 0, false, '\u0000' | null                             |
| Size       | Fixed              | Variable                         |
| Null       | Cannot be null     | Can be null                      |
| Methods    | No methods         | Has methods                      |
| Comparison | == compares values | == compares addresses            |

---

## Variables in Java

### Types of Variables

```java
public class VariableTypes {
    // 1. Instance Variables (Non-static fields)
    // - One copy per object instance
    // - Stored in heap (inside object)
    // - Default values applied
    private String instanceVar = "Instance";

    // 2. Class Variables (Static fields)
    // - One copy shared by all instances
    // - Stored in Method Area (Metaspace)
    // - Default values applied
    private static int classVar = 100;

    // 3. Local Variables
    // - Declared inside methods
    // - Stored in stack
    // - NO default values (must initialize before use)
    public void method() {
        int localVar = 10;  // Must initialize
        // int uninitialized;
        // System.out.println(uninitialized);  // Compilation error
    }

    // 4. Parameters
    // - Passed to methods
    // - Stored in stack
    public void methodWithParam(int param) {
        System.out.println(param);
    }
}
```

### Variable Scope

```java
public class ScopeDemo {
    private int instanceScope;  // Accessible throughout the class
    private static int classScope;  // Accessible via class name

    public void methodScope() {
        int methodLocal = 10;  // Accessible only in this method

        {
            int blockLocal = 20;  // Accessible only in this block
            System.out.println(methodLocal);  // OK
            System.out.println(blockLocal);   // OK
        }

        // System.out.println(blockLocal);  // Error: out of scope

        for (int i = 0; i < 5; i++) {
            int loopLocal = i;  // Accessible only in this loop
        }
        // System.out.println(loopLocal);  // Error: out of scope
    }
}
```

### Variable Initialization

```java
public class InitializationDemo {
    // Instance variables - default initialized
    int intDefault;           // 0
    double doubleDefault;     // 0.0
    boolean boolDefault;      // false
    char charDefault;         // '\u0000'
    String stringDefault;     // null
    int[] arrayDefault;       // null

    public void showDefaults() {
        System.out.println("int: " + intDefault);         // 0
        System.out.println("double: " + doubleDefault);   // 0.0
        System.out.println("boolean: " + boolDefault);    // false
        System.out.println("char: '" + charDefault + "'");// ''
        System.out.println("String: " + stringDefault);   // null
        System.out.println("Array: " + arrayDefault);     // null
    }

    public void localVariables() {
        // Local variables MUST be initialized before use
        int x;
        // System.out.println(x);  // Compilation error

        x = 10;
        System.out.println(x);  // OK: 10

        // Compiler tracks definite assignment
        int y;
        if (true) {
            y = 20;
        }
        // System.out.println(y);  // Error: might not be initialized

        int z;
        if (true) {
            z = 30;
        } else {
            z = 40;
        }
        System.out.println(z);  // OK: definitely assigned
    }
}
```

### Constants (final variables)

```java
public class ConstantsDemo {
    // Class-level constant
    public static final double PI = 3.14159265359;
    public static final int MAX_SIZE = 100;

    // Instance-level final (set once per object)
    private final String id;

    public ConstantsDemo(String id) {
        this.id = id;  // Must be set in constructor
    }

    public void method() {
        // Local final
        final int localConst = 10;
        // localConst = 20;  // Compilation error

        // Effectively final (Java 8+) - for lambdas
        int effectivelyFinal = 30;
        // Used in lambda but never reassigned
        Runnable r = () -> System.out.println(effectivelyFinal);

        // effectivelyFinal = 40;  // This would make it NOT effectively final
    }
}
```

---

## Type Casting and Conversion

### Implicit Casting (Widening)

```java
public class WideningDemo {
    public static void main(String[] args) {
        // Smaller type to larger type - automatic
        byte b = 10;
        short s = b;     // byte -> short
        int i = s;       // short -> int
        long l = i;      // int -> long
        float f = l;     // long -> float
        double d = f;    // float -> double

        // Widening hierarchy:
        // byte -> short -> int -> long -> float -> double
        //          char ↗

        // char to int (widening)
        char c = 'A';
        int charToInt = c;  // 65

        // int to double in expression
        int x = 5;
        double result = x / 2.0;  // x widened to double: 2.5
    }
}
```

### Explicit Casting (Narrowing)

```java
public class NarrowingDemo {
    public static void main(String[] args) {
        // Larger type to smaller type - explicit cast required
        double d = 100.99;
        float f = (float) d;   // 100.99
        long l = (long) f;     // 100 (truncated)
        int i = (int) l;       // 100
        short s = (short) i;   // 100
        byte b = (byte) s;     // 100

        // Data loss examples
        int largeInt = 130;
        byte smallByte = (byte) largeInt;  // -126 (overflow)

        double pi = 3.14159;
        int truncated = (int) pi;  // 3 (decimal lost)

        // char to byte requires cast
        char ch = 'A';
        byte byteFromChar = (byte) ch;  // 65

        // int to char
        int num = 66;
        char charFromInt = (char) num;  // 'B'
    }
}
```

### Type Promotion in Expressions

```java
public class TypePromotionDemo {
    public static void main(String[] args) {
        // In expressions, smaller types promoted to at least int
        byte b1 = 10;
        byte b2 = 20;
        // byte b3 = b1 + b2;  // Error: result is int
        int b3 = b1 + b2;      // OK
        byte b4 = (byte) (b1 + b2);  // OK with cast

        // Mixed type promotion
        int i = 10;
        long l = 20L;
        float f = 30.0f;
        double d = 40.0;

        // Result type is the largest type in expression
        long result1 = i + l;      // int + long = long
        float result2 = l + f;     // long + float = float
        double result3 = f + d;    // float + double = double

        // Compound assignment handles casting automatically
        byte b = 10;
        b += 5;  // Equivalent to: b = (byte)(b + 5)

        // This works:
        byte x = 10;
        x += 128;  // No error, auto-cast

        // But this doesn't:
        // x = x + 128;  // Error: int to byte
    }
}
```

### String Conversion

```java
public class StringConversionDemo {
    public static void main(String[] args) {
        // Primitive to String
        String s1 = String.valueOf(42);           // "42"
        String s2 = Integer.toString(42);         // "42"
        String s3 = "" + 42;                      // "42" (concatenation)

        // String to primitive
        int i = Integer.parseInt("42");           // 42
        double d = Double.parseDouble("3.14");    // 3.14
        boolean b = Boolean.parseBoolean("true"); // true

        // String to primitive (wrapper methods)
        Integer intObj = Integer.valueOf("42");   // Integer object
        int primitive = intObj;                   // auto-unboxing

        // NumberFormatException
        try {
            int invalid = Integer.parseInt("abc");
        } catch (NumberFormatException e) {
            System.out.println("Invalid number format");
        }

        // Radix conversion
        int binary = Integer.parseInt("1010", 2);   // 10
        int octal = Integer.parseInt("12", 8);      // 10
        int hex = Integer.parseInt("A", 16);        // 10

        String toBinary = Integer.toBinaryString(10);  // "1010"
        String toOctal = Integer.toOctalString(10);    // "12"
        String toHex = Integer.toHexString(10);        // "a"
    }
}
```

---

## Wrapper Classes

### Overview

```java
// Each primitive has a corresponding wrapper class
// Primitive -> Wrapper
// byte      -> Byte
// short     -> Short
// int       -> Integer
// long      -> Long
// float     -> Float
// double    -> Double
// char      -> Character
// boolean   -> Boolean
```

### Why Wrapper Classes?

1. **Generics require objects**: `List<Integer>` not `List<int>`
2. **Null values**: Primitives can't be null, wrappers can
3. **Utility methods**: `Integer.parseInt()`, `Integer.toBinaryString()`
4. **Collections**: All collections store objects
5. **Reflection**: Works with Class objects

### Wrapper Class Features

```java
public class WrapperDemo {
    public static void main(String[] args) {
        // Creating wrapper objects
        Integer i1 = Integer.valueOf(100);  // Preferred
        Integer i2 = Integer.valueOf("100");
        // Integer i3 = new Integer(100);  // Deprecated since Java 9

        // Useful constants
        System.out.println(Integer.MAX_VALUE);  // 2147483647
        System.out.println(Integer.MIN_VALUE);  // -2147483648
        System.out.println(Integer.SIZE);       // 32 (bits)
        System.out.println(Integer.BYTES);      // 4 (bytes)

        // Parsing methods
        int parsed = Integer.parseInt("42");
        int withRadix = Integer.parseInt("2A", 16);  // 42

        // Comparison
        int cmp = Integer.compare(10, 20);  // -1 (10 < 20)

        // Bit manipulation
        int bits = Integer.bitCount(7);     // 3 (binary: 111)
        int highest = Integer.highestOneBit(10);  // 8
        int lowest = Integer.lowestOneBit(10);    // 2

        // Reverse bits
        int reversed = Integer.reverse(1);  // -2147483648
    }
}
```

### Integer Cache (Important for Interviews!)

```java
public class IntegerCacheDemo {
    public static void main(String[] args) {
        // Integer caches values from -128 to 127
        Integer a = 100;
        Integer b = 100;
        System.out.println(a == b);  // true (same cached object)

        Integer c = 200;
        Integer d = 200;
        System.out.println(c == d);  // false (different objects)

        // Always use equals() for wrapper comparison
        System.out.println(c.equals(d));  // true

        // valueOf uses cache, new doesn't
        Integer e = Integer.valueOf(100);
        Integer f = Integer.valueOf(100);
        System.out.println(e == f);  // true (cached)

        // The cache range can be increased via JVM argument:
        // -XX:AutoBoxCacheMax=<size>
    }
}
```

---

## Autoboxing and Unboxing

### What is Autoboxing and Unboxing?

```java
public class AutoboxingDemo {
    public static void main(String[] args) {
        // Autoboxing: primitive -> wrapper (automatic)
        Integer wrapped = 42;  // int -> Integer (autoboxing)

        // Unboxing: wrapper -> primitive (automatic)
        int unwrapped = wrapped;  // Integer -> int (unboxing)

        // How it works internally
        Integer manual = Integer.valueOf(42);  // What compiler does for autoboxing
        int manualUnbox = wrapped.intValue();  // What compiler does for unboxing

        // Works with all wrapper types
        Double d = 3.14;      // double -> Double
        Boolean b = true;     // boolean -> Boolean
        Character c = 'A';    // char -> Character
    }
}
```

### Autoboxing in Collections

```java
public class CollectionAutoboxingDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();

        // Autoboxing when adding
        numbers.add(1);   // int -> Integer
        numbers.add(2);
        numbers.add(3);

        // Unboxing when retrieving
        int first = numbers.get(0);  // Integer -> int

        // Mixed operations
        int sum = 0;
        for (Integer num : numbers) {
            sum += num;  // Unboxing in each iteration
        }

        // Autoboxing in method calls
        processNumber(42);  // Autoboxed to Integer
    }

    static void processNumber(Integer num) {
        System.out.println(num);
    }
}
```

### Autoboxing Pitfalls

```java
public class AutoboxingPitfalls {
    public static void main(String[] args) {
        // Pitfall 1: NullPointerException
        Integer nullInteger = null;
        // int value = nullInteger;  // NullPointerException during unboxing

        // Safe way:
        int value = (nullInteger != null) ? nullInteger : 0;

        // Pitfall 2: Performance in loops
        Long sum = 0L;  // Using wrapper instead of primitive
        for (long i = 0; i < 1000000; i++) {
            sum += i;  // Creates new Long object each iteration!
        }
        // Better: use primitive long for sum

        // Pitfall 3: == vs equals()
        Integer x = 1000;
        Integer y = 1000;
        System.out.println(x == y);       // false (outside cache range)
        System.out.println(x.equals(y));  // true

        // Pitfall 4: Overloaded methods
        overloaded(100);     // Calls primitive version
        overloaded((Integer) 100);  // Calls wrapper version
    }

    static void overloaded(int x) {
        System.out.println("Primitive: " + x);
    }

    static void overloaded(Integer x) {
        System.out.println("Wrapper: " + x);
    }
}
```

### Comparison with Autoboxing

```java
public class ComparisonDemo {
    public static void main(String[] args) {
        // Comparing with primitives (unboxing happens)
        Integer a = 100;
        int b = 100;
        System.out.println(a == b);  // true (a is unboxed)

        // Comparing two wrappers (reference comparison)
        Integer c = 100;
        Integer d = 100;
        System.out.println(c == d);  // true (cached)

        Integer e = 1000;
        Integer f = 1000;
        System.out.println(e == f);  // false (not cached)

        // Comparing different wrapper types
        Integer intVal = 100;
        Long longVal = 100L;
        // System.out.println(intVal == longVal);  // Compilation error
        System.out.println(intVal.equals(longVal));  // false (different types)
        System.out.println(intVal.longValue() == longVal);  // true
    }
}
```

---

## var Keyword (Java 10+)

### What is var?

```java
public class VarDemo {
    public static void main(String[] args) {
        // Type inference - compiler determines the type
        var message = "Hello";        // Inferred as String
        var count = 42;               // Inferred as int
        var price = 19.99;            // Inferred as double
        var flag = true;              // Inferred as boolean

        // Works with complex types
        var list = new ArrayList<String>();  // ArrayList<String>
        var map = new HashMap<String, Integer>();  // HashMap<String, Integer>

        // With method return types
        var stream = list.stream();  // Stream<String>
        var optional = Optional.of("value");  // Optional<String>

        // The type is fixed after inference
        var x = 10;
        // x = "Hello";  // Compilation error: incompatible types
    }
}
```

### var Rules and Restrictions

```java
public class VarRestrictionsDemo {
    // Cannot use var for instance variables
    // var instanceVar = 10;  // Compilation error

    // Cannot use var for method parameters
    // void method(var param) { }  // Compilation error

    // Cannot use var for method return types
    // var method() { return 10; }  // Compilation error

    public void validUsages() {
        // Must have initializer
        // var x;  // Error: cannot use var without initializer

        // Cannot initialize with null
        // var y = null;  // Error: cannot infer type

        // Cannot use with array initializer
        // var arr = {1, 2, 3};  // Error
        var arr = new int[]{1, 2, 3};  // OK

        // Works in for loops
        for (var i = 0; i < 10; i++) {
            System.out.println(i);
        }

        // Works in enhanced for loops
        var list = List.of("a", "b", "c");
        for (var item : list) {
            System.out.println(item);
        }

        // Works in try-with-resources
        try (var reader = new BufferedReader(new FileReader("file.txt"))) {
            var line = reader.readLine();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### var Best Practices

```java
public class VarBestPractices {
    public void examples() {
        // GOOD: Type is obvious from right side
        var list = new ArrayList<String>();
        var map = new HashMap<String, List<Integer>>();
        var stream = files.stream();

        // BAD: Type is not obvious
        var result = someMethod();  // What type is result?
        var data = process(input);  // What type is data?

        // GOOD: Use meaningful variable names with var
        var customerNames = getCustomerNames();  // List<String> implied
        var orderTotal = calculateTotal();       // Probably numeric

        // BAD: Vague names with var
        var x = getData();
        var y = compute();

        // AVOID: When it reduces readability
        var value = (flag) ? someMethod() : anotherMethod();  // What type?
    }

    // Use var when it IMPROVES readability, not reduces it
}
```

---

## Interview Questions

### Q1: What is the difference between primitive and reference types?

**Answer:**
| Aspect | Primitive | Reference |
|--------|-----------|-----------|
| Storage | Value directly in stack | Reference in stack, object in heap |
| Default | 0, false, '\u0000' | null |
| Null | Cannot be null | Can be null |
| Methods | No methods | Has methods |
| == | Compares values | Compares references |
| Pass by | Value (copy) | Value (reference copy) |
| Memory | Fixed size | Variable size |
| Generics | Cannot use directly | Can use |

---

### Q2: Why is String immutable in Java?

**Answer:**

1. **Security**: Strings used in network connections, database URLs, usernames can't be changed
2. **Caching**: String pool works because strings are immutable
3. **Thread Safety**: Immutable objects are inherently thread-safe
4. **HashCode Caching**: hashCode can be cached since string won't change
5. **Class Loading**: Class names are strings; mutability would be a security risk

```java
String s1 = "Hello";
String s2 = s1;
s1 = "World";  // Creates new object, s2 still points to "Hello"
```

---

### Q3: What happens when you do Integer a = 100?

**Answer:**
This is **autoboxing**. The compiler converts it to:

```java
Integer a = Integer.valueOf(100);
```

`Integer.valueOf()` uses an **Integer Cache** for values from -128 to 127, so:

```java
Integer a = 100;  // Returns cached object
Integer b = 100;  // Returns same cached object
System.out.println(a == b);  // true

Integer c = 200;  // Creates new object (outside cache)
Integer d = 200;  // Creates different new object
System.out.println(c == d);  // false
```

---

### Q4: What is the output?

```java
public class Test {
    public static void main(String[] args) {
        Integer a = new Integer(100);
        Integer b = new Integer(100);
        Integer c = 100;
        Integer d = 100;

        System.out.println(a == b);
        System.out.println(c == d);
        System.out.println(a == c);
    }
}
```

**Answer:**

```
false  // a and b are different objects (new creates new object)
true   // c and d are same cached object (autoboxing uses cache)
false  // a is new object, c is cached object
```

---

### Q5: Why can't primitives be used with generics?

**Answer:**
Generics in Java use **type erasure** - generic type information is removed at compile time and replaced with Object. Since primitives are not Objects, they cannot be used.

```java
// Cannot do this:
// List<int> numbers;  // Compilation error

// Must use wrapper:
List<Integer> numbers = new ArrayList<>();

// At runtime, it becomes:
List numbers = new ArrayList();  // Type erased to Object
```

This is why we have wrapper classes and autoboxing.

---

### Q6: What is the output?

```java
public class Test {
    public static void main(String[] args) {
        byte b = 127;
        b++;
        System.out.println(b);

        byte x = 10;
        byte y = 20;
        // byte z = x + y;  // Would this compile?
        byte z = (byte) (x + y);
        System.out.println(z);
    }
}
```

**Answer:**

```
-128    // Overflow: 127 + 1 wraps to -128
30      // Cast required because x + y results in int
```

The commented line `byte z = x + y;` would **NOT compile** because:

- In expressions, bytes are promoted to int
- The result is int, which cannot be assigned to byte without explicit cast

---

### Q7: What is the difference between float and double?

**Answer:**
| Aspect | float | double |
|--------|-------|--------|
| Size | 32 bits | 64 bits |
| Precision | ~6-7 decimal digits | ~15-16 decimal digits |
| Default | No (need f suffix) | Yes (for decimal literals) |
| Memory | Less | More |
| Use case | Memory-constrained, graphics | Most calculations |

```java
float f = 3.14f;   // Needs 'f' suffix
double d = 3.14;   // No suffix needed (default)

// Precision difference
float fPi = 3.14159265358979323846f;
double dPi = 3.14159265358979323846;
System.out.println(fPi);  // 3.1415927
System.out.println(dPi);  // 3.141592653589793
```

---

### Q8: What is type promotion in expressions?

**Answer:**
When performing operations on different types, Java promotes smaller types to larger types:

1. If any operand is `double`, the other is converted to `double`
2. Else if any operand is `float`, the other is converted to `float`
3. Else if any operand is `long`, the other is converted to `long`
4. Else both are converted to `int`

```java
byte b = 10;
short s = 20;
int i = 30;
long l = 40L;
float f = 50.0f;
double d = 60.0;

// Type of result depends on operands
int r1 = b + s;      // byte + short = int (both promoted to int)
long r2 = i + l;     // int + long = long
float r3 = l + f;    // long + float = float
double r4 = f + d;   // float + double = double
```

---

### Q9: Can we use var with lambda expressions?

**Answer:**
Starting from **Java 11**, you can use `var` with lambda parameters:

```java
// Java 11+
BiFunction<Integer, Integer, Integer> add = (var a, var b) -> a + b;

// Why would you want this?
// To add annotations:
BiFunction<Integer, Integer, Integer> addWithAnnotation =
    (@NonNull var a, @NonNull var b) -> a + b;
```

However, you cannot:

```java
// Cannot do this - var with implicit type
// var fn = (a, b) -> a + b;  // Error: cannot infer type

// Must specify target type
BinaryOperator<Integer> fn = (a, b) -> a + b;  // OK
```

---

### Q10: What are effectively final variables?

**Answer:**
A variable is **effectively final** if it's never reassigned after initialization. This concept was introduced in **Java 8** for use with lambdas and anonymous classes.

```java
public void example() {
    int x = 10;  // Effectively final (never reassigned)
    int y = 20;
    y = 30;      // Not effectively final (reassigned)

    // Lambda can use effectively final variable
    Runnable r = () -> System.out.println(x);  // OK

    // Cannot use non-effectively final variable
    // Runnable r2 = () -> System.out.println(y);  // Error
}
```

---

### Q11: What is the output of this code?

```java
public class Test {
    public static void main(String[] args) {
        double d = 0.1 + 0.2;
        System.out.println(d);
        System.out.println(d == 0.3);
    }
}
```

**Answer:**

```
0.30000000000000004
false
```

**Explanation:** Floating-point numbers cannot represent 0.1 and 0.2 exactly in binary. This causes precision issues.

**Solution for money/precision:**

```java
BigDecimal bd1 = new BigDecimal("0.1");
BigDecimal bd2 = new BigDecimal("0.2");
BigDecimal sum = bd1.add(bd2);
System.out.println(sum.equals(new BigDecimal("0.3"))); // true
```

---

### Q12: What is the difference between int and Integer?

| int (Primitive)        | Integer (Wrapper)                |
| ---------------------- | -------------------------------- |
| 32-bit value           | Object on heap                   |
| Cannot be null         | Can be null                      |
| No methods             | Has methods (parseInt, toString) |
| Faster operations      | Slower (boxing overhead)         |
| Cannot use in generics | Can use in generics              |
| Default value: 0       | Default value: null              |

```java
int a = 10;                   // Primitive
Integer b = 10;               // Autoboxed to Integer
Integer c = null;             // Valid
// int d = null;              // Compilation error

List<Integer> list = new ArrayList<>();  // OK
// List<int> list2;           // Compilation error
```

---

### Q13: What is NaN in Java?

**Answer:**
**NaN** (Not a Number) represents undefined results in floating-point operations.

```java
double nan1 = 0.0 / 0.0;           // NaN
double nan2 = Math.sqrt(-1);       // NaN
double nan3 = Double.NaN;          // NaN constant

// NaN properties
System.out.println(nan1 == nan1);  // false (NaN != NaN)
System.out.println(Double.isNaN(nan1));  // true

// Infinity
double posInf = 1.0 / 0.0;         // Infinity
double negInf = -1.0 / 0.0;        // -Infinity
System.out.println(Double.isInfinite(posInf));  // true
```

---

### Q14: What is the output?

```java
public class Test {
    public static void main(String[] args) {
        int x = 5;
        System.out.println(x++ + ++x);

        int y = 5;
        y = y++;
        System.out.println(y);
    }
}
```

**Answer:**

```
12
5
```

**Explanation:**

- `x++ + ++x`: x++ is 5 (x becomes 6), ++x is 7 (x becomes 7). Result: 5 + 7 = 12
- `y = y++`: y++ returns 5, then increments y to 6, but assignment overwrites with 5

---

### Q15: How does BigDecimal differ from double?

| double           | BigDecimal          |
| ---------------- | ------------------- |
| Approximate      | Exact               |
| Faster           | Slower              |
| Fixed precision  | Arbitrary precision |
| Native type      | Object              |
| Good for science | Good for finance    |

```java
// double precision issue
double d1 = 1.0 - 0.9;
System.out.println(d1);  // 0.09999999999999998

// BigDecimal is exact
BigDecimal bd1 = new BigDecimal("1.0");
BigDecimal bd2 = new BigDecimal("0.9");
System.out.println(bd1.subtract(bd2));  // 0.1

// IMPORTANT: Use String constructor!
new BigDecimal("0.1");  // Correct
new BigDecimal(0.1);    // Wrong - inherits double imprecision
```

---

### Q16: What is the size and range of each primitive type?

| Type    | Size          | Min       | Max      |
| ------- | ------------- | --------- | -------- |
| byte    | 8 bits        | -128      | 127      |
| short   | 16 bits       | -32,768   | 32,767   |
| int     | 32 bits       | -2^31     | 2^31-1   |
| long    | 64 bits       | -2^63     | 2^63-1   |
| float   | 32 bits       | ~-3.4E38  | ~3.4E38  |
| double  | 64 bits       | ~-1.8E308 | ~1.8E308 |
| char    | 16 bits       | 0         | 65,535   |
| boolean | JVM dependent | false     | true     |

---

### Q17: What is widening vs narrowing conversion?

**Widening (Implicit):** Smaller → Larger (no data loss)

```java
byte b = 10;
int i = b;       // byte → int (implicit)
long l = i;      // int → long (implicit)
double d = l;    // long → double (implicit)
```

**Narrowing (Explicit):** Larger → Smaller (potential data loss)

```java
double d = 10.99;
int i = (int) d;    // 10 (fractional part lost)
byte b = (byte) 200; // -56 (overflow)
```

---

### Q18: What happens when Integer wrapper is null and unboxed?

**Answer:** `NullPointerException` is thrown.

```java
Integer wrapper = null;
int primitive = wrapper;  // NullPointerException!

// Safe approach
Integer wrapper2 = null;
int primitive2 = (wrapper2 != null) ? wrapper2 : 0;

// Or use Optional (Java 8+)
int value = Optional.ofNullable(wrapper2).orElse(0);
```

---

### Q19: What is the output?

```java
public class Test {
    public static void main(String[] args) {
        Short s = 128;
        Integer i = 128;

        System.out.println(s.equals(i));
        System.out.println(s.intValue() == i);
    }
}
```

**Answer:**

```
false
true
```

**Explanation:**

- `s.equals(i)`: Different types (Short vs Integer), equals returns false
- `s.intValue() == i`: Both are 128 (int comparison), returns true

---

### Q20: What is the difference between == and equals() for wrapper classes?

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b);      // true (cached)
System.out.println(a.equals(b)); // true

Integer c = 128;
Integer d = 128;
System.out.println(c == d);      // false (not cached)
System.out.println(c.equals(d)); // true
```

**Rule:** Always use `equals()` for wrapper class comparison.

---

## Common Interview Traps

### Trap 1: "What is the default value of a local variable?"

**Answer:** Local variables have **no default value**. They must be initialized before use, or you get a compilation error.

### Trap 2: "Is boolean size 1 bit?"

**Answer:** The JVM specification doesn't define the size. In practice, JVMs typically use 1 byte (8 bits) or even 32 bits (int) for boolean to align with memory.

### Trap 3: "Can char store negative values?"

**Answer:** No, `char` is an **unsigned** 16-bit type (0 to 65,535). For negative values, use `short`.

### Trap 4: "What is the type of 1 + 2L + 3.0f + 4.0?"

**Answer:** `double`. The expression has double (4.0), so everything is promoted to double.

### Trap 5: "Is String a primitive type?"

**Answer:** No, `String` is a **class** (reference type), even though it has special syntax support like literals and the + operator.

---

## Key Takeaways

1. **8 primitive types**: byte, short, int, long, float, double, char, boolean
2. **Wrapper classes** allow primitives to be used as objects
3. **Integer cache** caches -128 to 127 (important for == comparisons)
4. **Type promotion** happens in expressions (byte/short → int)
5. **var** enables type inference (Java 10+) for local variables
6. **Autoboxing/Unboxing** is automatic but has performance implications
7. **Local variables** must be explicitly initialized
8. **float** needs f suffix; **long** needs L suffix
