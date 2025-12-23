# Strings in Java - Complete Deep Dive

## Table of Contents

1. [String Fundamentals](#string-fundamentals)
2. [String Pool and Memory](#string-pool-and-memory)
3. [String Immutability](#string-immutability)
4. [String Methods](#string-methods)
5. [StringBuilder and StringBuffer](#stringbuilder-and-stringbuffer)
6. [String Comparison](#string-comparison)
7. [String Formatting](#string-formatting)
8. [Regular Expressions](#regular-expressions)
9. [Java 11+ String Methods](#java-11-string-methods)
10. [Performance Considerations](#performance-considerations)
11. [Interview Questions](#interview-questions)

---

## String Fundamentals

### What is String in Java?

String is a **sequence of characters** represented as an object in Java. It's one of the most used classes and has special support from the language.

```java
// String is a class in java.lang package
// It's NOT a primitive type, but has special treatment

// Multiple ways to create strings
String s1 = "Hello";                    // String literal (recommended)
String s2 = new String("Hello");        // Using constructor (creates new object)
String s3 = new String(new char[]{'H', 'e', 'l', 'l', 'o'});  // From char array
String s4 = String.valueOf(123);        // From other types
```

### String Internal Implementation

```java
// Before Java 9: backed by char[]
// private final char value[];

// Java 9+: backed by byte[] for memory efficiency
// private final byte[] value;
// private final byte coder;  // LATIN1 or UTF16

// Why byte[]?
// Most strings use only ASCII characters (1 byte each)
// UTF-16 (char) uses 2 bytes per character
// Saves ~50% memory for ASCII-only strings
```

### String as Final Class

```java
public final class String  // Cannot be extended
    implements java.io.Serializable,
               Comparable<String>,
               CharSequence {

    private final byte[] value;  // Immutable - cannot change after creation
    // ...
}

// Why final?
// 1. Security - Strings used for passwords, network connections, file paths
// 2. Thread safety - Immutable objects are inherently thread-safe
// 3. Performance - Can be cached (String pool)
// 4. Hashcode caching - hashCode computed once and cached
```

---

## String Pool and Memory

### What is String Pool?

String Pool (String Intern Pool) is a special memory region in the **heap** (part of Metaspace in Java 8+) where Java stores string literals.

```
┌─────────────────────────────────────────────────────────────────┐
│                          HEAP MEMORY                             │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    STRING POOL                             │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │  │
│  │  │ "Hello" │  │ "World" │  │ "Java"  │  │  "123"  │       │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                 REGULAR HEAP OBJECTS                       │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  String Object (new String("Hello"))                 │  │  │
│  │  │  - References "Hello" in pool for value              │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### String Creation and Memory

```java
public class StringMemoryDemo {
    public static void main(String[] args) {
        // Literal - stored in String Pool
        String s1 = "Hello";

        // Literal - reuses existing "Hello" from pool
        String s2 = "Hello";

        // Using new - creates NEW object in heap (outside pool)
        String s3 = new String("Hello");

        // Another new object
        String s4 = new String("Hello");

        // Check references
        System.out.println(s1 == s2);  // true (same pool reference)
        System.out.println(s1 == s3);  // false (different objects)
        System.out.println(s3 == s4);  // false (different objects)

        // Content comparison
        System.out.println(s1.equals(s3));  // true (same content)
    }
}
```

### Memory Visualization

```
Stack                          Heap
┌─────────────────┐           ┌─────────────────────────────────────┐
│ s1 [ref: 0x100] │──────────►│ String Pool                         │
├─────────────────┤           │ ┌─────────────────────────────────┐ │
│ s2 [ref: 0x100] │──────────►│ │ 0x100: "Hello"                  │ │
├─────────────────┤           │ └─────────────────────────────────┘ │
│ s3 [ref: 0x200] │───────┐   │                                     │
├─────────────────┤       │   │ Regular Heap                        │
│ s4 [ref: 0x300] │───┐   │   │ ┌─────────────────────────────────┐ │
└─────────────────┘   │   └──►│ │ 0x200: String obj               │ │
                      │       │ │        value -> 0x100            │ │
                      │       │ └─────────────────────────────────┘ │
                      │       │ ┌─────────────────────────────────┐ │
                      └──────►│ │ 0x300: String obj               │ │
                              │ │        value -> 0x100            │ │
                              │ └─────────────────────────────────┘ │
                              └─────────────────────────────────────┘
```

### String.intern() Method

```java
public class InternDemo {
    public static void main(String[] args) {
        String s1 = new String("Hello");  // Creates 2 objects: pool + heap
        String s2 = s1.intern();          // Returns reference from pool
        String s3 = "Hello";              // Reference from pool

        System.out.println(s1 == s2);  // false (heap vs pool)
        System.out.println(s2 == s3);  // true (both pool references)

        // Use case: Save memory when many duplicate strings
        List<String> names = new ArrayList<>();
        // Instead of storing many "Active" strings
        String status = "Active".intern();  // Reuse single pool instance
    }
}
```

### How Many Objects Created?

```java
// Question: How many objects are created?
String s1 = "Hello";
// Answer: 1 object (in pool, if not already exists)
// If "Hello" already in pool: 0 new objects

String s2 = new String("Hello");
// Answer: Could be 1 or 2
// - If "Hello" not in pool: 2 objects (one in pool, one in heap)
// - If "Hello" already in pool: 1 object (only heap)

String s3 = new String("Hello") + new String("World");
// Answer: Approximately 5 objects
// - "Hello" in pool (if not exists)
// - new String("Hello") in heap
// - "World" in pool (if not exists)
// - new String("World") in heap
// - StringBuilder internally
// - Result String "HelloWorld" in heap
```

---

## String Immutability

### What is Immutability?

Once a String object is created, its content cannot be changed. Any modification creates a new String object.

```java
public class ImmutabilityDemo {
    public static void main(String[] args) {
        String s = "Hello";
        System.out.println("Original: " + s);
        System.out.println("Hashcode: " + System.identityHashCode(s));

        // This doesn't modify 's', creates NEW string
        s = s.concat(" World");
        System.out.println("After concat: " + s);
        System.out.println("Hashcode: " + System.identityHashCode(s));  // Different!

        // Original "Hello" still exists in pool (eligible for GC if no refs)

        // All these create new strings
        String s2 = s.toUpperCase();
        String s3 = s.substring(0, 5);
        String s4 = s.replace('H', 'J');
    }
}
```

### Why String is Immutable?

#### 1. Security

```java
// Strings used in security-critical areas
String databaseUrl = "jdbc:mysql://localhost:3306/db";
String username = "admin";
String password = "secret";
String filePath = "/etc/passwd";
Class<?> clazz = Class.forName("java.lang.String");

// If mutable, these could be changed after validation
// but before use - security vulnerability!
```

#### 2. Thread Safety

```java
// Immutable objects are inherently thread-safe
// Multiple threads can share the same String without synchronization

class ThreadSafeExample {
    private final String config = "production";  // Safe to share

    public void processRequest() {
        // Multiple threads can read 'config' safely
        System.out.println(config);
    }
}
```

#### 3. Hashcode Caching

```java
// String caches its hashCode after first computation
public int hashCode() {
    int h = hash;  // cached value
    if (h == 0 && value.length > 0) {
        h = computeHashCode();
        hash = h;  // cache for future
    }
    return h;
}

// Critical for HashMap/HashSet performance
Map<String, Object> map = new HashMap<>();
map.put("key", value);  // hashCode computed once
map.get("key");         // uses cached hashCode
```

#### 4. String Pool

```java
// Immutability enables string interning
String s1 = "Hello";
String s2 = "Hello";
// Safe to share because neither can modify the content
```

### Attempting to Modify (Using Reflection - Don't Do This!)

```java
// WARNING: This is just for understanding, NEVER do this in real code
public class ReflectionHack {
    public static void main(String[] args) throws Exception {
        String s = "Hello";
        System.out.println("Before: " + s);

        // Get the value field
        Field field = String.class.getDeclaredField("value");
        field.setAccessible(true);  // Bypass private

        // In Java 8: char[]
        // char[] value = (char[]) field.get(s);
        // value[0] = 'J';

        System.out.println("After: " + s);  // "Jello"

        // This also affects the pool entry!
        String s2 = "Hello";
        System.out.println("Pool string: " + s2);  // Also "Jello"!
    }
}
// Note: Java 9+ has stronger encapsulation, this may fail
```

---

## String Methods

### Common String Methods

```java
public class StringMethodsDemo {
    public static void main(String[] args) {
        String str = "  Hello World  ";

        // Length
        System.out.println(str.length());  // 15 (includes spaces)

        // Character access
        System.out.println(str.charAt(2));  // 'H' (0-indexed)

        // Substring
        System.out.println(str.substring(2));      // "Hello World  "
        System.out.println(str.substring(2, 7));   // "Hello" (end exclusive)

        // Trimming
        System.out.println(str.trim());           // "Hello World"
        System.out.println(str.strip());          // "Hello World" (Java 11+)
        System.out.println(str.stripLeading());   // "Hello World  " (Java 11+)
        System.out.println(str.stripTrailing());  // "  Hello World" (Java 11+)

        // Case conversion
        System.out.println(str.toUpperCase());    // "  HELLO WORLD  "
        System.out.println(str.toLowerCase());    // "  hello world  "

        // Search
        System.out.println(str.indexOf('o'));          // 6
        System.out.println(str.lastIndexOf('o'));      // 9
        System.out.println(str.indexOf("World"));      // 8
        System.out.println(str.contains("Hello"));     // true
        System.out.println(str.startsWith("  H"));     // true
        System.out.println(str.endsWith("  "));        // true

        // Replace
        System.out.println(str.replace('l', 'L'));           // "  HeLLo WorLd  "
        System.out.println(str.replace("World", "Java"));    // "  Hello Java  "
        System.out.println(str.replaceAll("\\s+", "_"));     // "_Hello_World_" (regex)
        System.out.println(str.replaceFirst("\\s+", "_"));   // "_Hello World  "

        // Split
        String csv = "apple,banana,cherry";
        String[] fruits = csv.split(",");  // ["apple", "banana", "cherry"]

        // Join (Java 8+)
        String joined = String.join("-", "2023", "12", "25");  // "2023-12-25"
        String fromArray = String.join(", ", fruits);  // "apple, banana, cherry"

        // Comparison
        String a = "Hello";
        String b = "hello";
        System.out.println(a.equals(b));            // false
        System.out.println(a.equalsIgnoreCase(b));  // true
        System.out.println(a.compareTo(b));         // negative (H < h in ASCII)
        System.out.println(a.compareToIgnoreCase(b));  // 0 (equal)

        // Empty and blank
        String empty = "";
        String blank = "   ";
        System.out.println(empty.isEmpty());   // true (length == 0)
        System.out.println(blank.isEmpty());   // false
        System.out.println(blank.isBlank());   // true (Java 11+, only whitespace)

        // Character array conversion
        char[] chars = "Hello".toCharArray();
        String fromChars = new String(chars);

        // Concatenation
        String concat = "Hello" + " " + "World";  // Compiler optimizes
        String concat2 = "Hello".concat(" World");
    }
}
```

### String Comparison Methods

```java
public class StringComparisonDemo {
    public static void main(String[] args) {
        String s1 = "Apple";
        String s2 = "Banana";
        String s3 = "apple";
        String s4 = "APPLE";

        // compareTo - lexicographic comparison
        // Returns: negative if s1 < s2, 0 if equal, positive if s1 > s2
        System.out.println(s1.compareTo(s2));    // -1 (A < B)
        System.out.println(s2.compareTo(s1));    // 1 (B > A)
        System.out.println(s1.compareTo("Apple")); // 0 (equal)

        // Case-sensitive vs case-insensitive
        System.out.println(s1.compareTo(s3));           // -32 (A=65, a=97)
        System.out.println(s1.compareToIgnoreCase(s3)); // 0

        // contentEquals - compare with CharSequence
        StringBuilder sb = new StringBuilder("Apple");
        System.out.println(s1.contentEquals(sb));  // true

        // regionMatches - compare regions
        String str1 = "Welcome to Java";
        String str2 = "Java Programming";
        // Compare "Java" in both (case-sensitive)
        System.out.println(str1.regionMatches(11, str2, 0, 4)); // true
        // Compare with ignoreCase
        System.out.println(str1.regionMatches(true, 11, str2, 0, 4)); // true
    }
}
```

### valueOf() and toString()

```java
public class ValueOfDemo {
    public static void main(String[] args) {
        // Static valueOf() - converts to String
        String s1 = String.valueOf(123);        // "123"
        String s2 = String.valueOf(45.67);      // "45.67"
        String s3 = String.valueOf(true);       // "true"
        String s4 = String.valueOf('A');        // "A"
        String s5 = String.valueOf(new char[]{'H', 'i'});  // "Hi"

        // valueOf with null
        Object obj = null;
        String s6 = String.valueOf(obj);  // "null" (not NPE!)

        // vs toString()
        Integer num = 123;
        String s7 = num.toString();       // "123"
        // String s8 = null.toString();   // NPE!

        // Object's toString
        Object o = new Object();
        System.out.println(o.toString());  // className@hashCode
    }
}
```

---

## StringBuilder and StringBuffer

### Why StringBuilder/StringBuffer?

```java
// Problem with String concatenation in loops
public class ConcatenationProblem {
    public static void main(String[] args) {
        // BAD - Creates many intermediate String objects
        String result = "";
        for (int i = 0; i < 10000; i++) {
            result += i;  // Each + creates new String
        }
        // Creates approximately 10000 String objects!

        // GOOD - Uses StringBuilder
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            sb.append(i);  // Modifies same object
        }
        String result2 = sb.toString();
        // Creates only 1 StringBuilder + 1 final String
    }
}
```

### StringBuilder (Not Thread-Safe, Fast)

```java
public class StringBuilderDemo {
    public static void main(String[] args) {
        // Creating StringBuilder
        StringBuilder sb1 = new StringBuilder();           // Empty, capacity 16
        StringBuilder sb2 = new StringBuilder(50);         // Empty, capacity 50
        StringBuilder sb3 = new StringBuilder("Hello");    // "Hello", capacity 21

        // Append (returns same StringBuilder - method chaining)
        sb1.append("Hello")
           .append(" ")
           .append("World")
           .append(123)
           .append(true);
        System.out.println(sb1);  // "Hello World123true"

        // Insert
        StringBuilder sb = new StringBuilder("Hello World");
        sb.insert(5, " Beautiful");  // "Hello Beautiful World"

        // Delete
        sb.delete(5, 16);  // "Hello World"
        sb.deleteCharAt(5);  // "HelloWorld"

        // Replace
        sb = new StringBuilder("Hello World");
        sb.replace(6, 11, "Java");  // "Hello Java"

        // Reverse
        sb = new StringBuilder("Hello");
        sb.reverse();  // "olleH"

        // Length and Capacity
        sb = new StringBuilder("Hello");
        System.out.println(sb.length());    // 5 (current chars)
        System.out.println(sb.capacity());  // 21 (initial 16 + "Hello" length)

        // Set length (truncate or pad with null chars)
        sb.setLength(3);  // "Hel"

        // Ensure capacity
        sb.ensureCapacity(100);  // Capacity at least 100

        // charAt, setCharAt
        sb = new StringBuilder("Hello");
        System.out.println(sb.charAt(0));  // 'H'
        sb.setCharAt(0, 'J');              // "Jello"

        // Substring (returns String, doesn't modify)
        String sub = sb.substring(1, 4);  // "ell"

        // Convert to String
        String result = sb.toString();
    }
}
```

### StringBuffer (Thread-Safe, Slower)

```java
public class StringBufferDemo {
    // StringBuffer has same methods as StringBuilder
    // but synchronized (thread-safe)

    public static void main(String[] args) {
        StringBuffer sb = new StringBuffer("Hello");
        sb.append(" World");

        // Thread-safe operations
        synchronized(sb) {
            // Not needed - methods are already synchronized
        }
    }

    // Use case: When multiple threads modify same buffer
    private StringBuffer log = new StringBuffer();

    public void appendLog(String message) {
        log.append(message).append("\n");  // Thread-safe
    }
}
```

### Comparison: String vs StringBuilder vs StringBuffer

| Feature       | String                    | StringBuilder                 | StringBuffer                 |
| ------------- | ------------------------- | ----------------------------- | ---------------------------- |
| Mutability    | Immutable                 | Mutable                       | Mutable                      |
| Thread Safety | Yes (immutable)           | No                            | Yes (synchronized)           |
| Performance   | Slow for concatenation    | Fast                          | Slower than StringBuilder    |
| Memory        | Pool optimization         | No pool                       | No pool                      |
| When to Use   | Small, infrequent changes | Single-threaded heavy changes | Multi-threaded heavy changes |
| Since         | JDK 1.0                   | JDK 1.5                       | JDK 1.0                      |

### Performance Comparison

```java
public class PerformanceTest {
    public static void main(String[] args) {
        int iterations = 100000;

        // String concatenation
        long start = System.currentTimeMillis();
        String str = "";
        for (int i = 0; i < iterations; i++) {
            str += "a";
        }
        System.out.println("String: " + (System.currentTimeMillis() - start) + "ms");

        // StringBuilder
        start = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < iterations; i++) {
            sb.append("a");
        }
        String result = sb.toString();
        System.out.println("StringBuilder: " + (System.currentTimeMillis() - start) + "ms");

        // StringBuffer
        start = System.currentTimeMillis();
        StringBuffer sbuf = new StringBuffer();
        for (int i = 0; i < iterations; i++) {
            sbuf.append("a");
        }
        String result2 = sbuf.toString();
        System.out.println("StringBuffer: " + (System.currentTimeMillis() - start) + "ms");
    }
}

// Typical results:
// String: ~5000ms
// StringBuilder: ~5ms
// StringBuffer: ~10ms
```

---

## String Comparison

### == vs equals() vs compareTo()

```java
public class StringComparisonTypes {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = "Hello";
        String s3 = new String("Hello");
        String s4 = new String("Hello");

        // == : Reference comparison
        System.out.println(s1 == s2);  // true (same pool reference)
        System.out.println(s1 == s3);  // false (different objects)
        System.out.println(s3 == s4);  // false (different heap objects)

        // equals() : Content comparison
        System.out.println(s1.equals(s2));  // true
        System.out.println(s1.equals(s3));  // true
        System.out.println(s3.equals(s4));  // true

        // compareTo() : Lexicographic comparison
        // Returns: 0 if equal, negative if less, positive if greater
        System.out.println(s1.compareTo(s2));  // 0
        System.out.println("A".compareTo("B"));  // -1
        System.out.println("B".compareTo("A"));  // 1
        System.out.println("A".compareTo("a"));  // -32 (ASCII difference)
    }
}
```

### Best Practices for Comparison

```java
public class ComparisonBestPractices {
    // BAD - Potential NullPointerException
    public boolean badComparison(String input) {
        return input.equals("expected");  // NPE if input is null
    }

    // GOOD - Literal first (null-safe)
    public boolean goodComparison(String input) {
        return "expected".equals(input);  // Safe even if input is null
    }

    // GOOD - Using Objects.equals (null-safe)
    public boolean nullSafeComparison(String a, String b) {
        return Objects.equals(a, b);  // Handles null for both
    }

    // For case-insensitive
    public boolean caseInsensitive(String input) {
        return "expected".equalsIgnoreCase(input);
    }
}
```

---

## String Formatting

### printf/format Style

```java
public class StringFormattingDemo {
    public static void main(String[] args) {
        // String.format()
        String formatted = String.format("Name: %s, Age: %d", "John", 25);
        System.out.println(formatted);  // "Name: John, Age: 25"

        // Common format specifiers
        // %s - String
        // %d - Integer (decimal)
        // %f - Floating point
        // %c - Character
        // %b - Boolean
        // %n - Line separator
        // %% - Literal %

        // Width and precision
        System.out.printf("|%10s|%n", "Hi");      // |        Hi|
        System.out.printf("|%-10s|%n", "Hi");     // |Hi        |
        System.out.printf("|%10.5f|%n", 3.14159); // |   3.14159|
        System.out.printf("|%-10.2f|%n", 3.14159);// |3.14      |

        // Numbers
        System.out.printf("%d%n", 1000);          // 1000
        System.out.printf("%,d%n", 1000000);      // 1,000,000
        System.out.printf("%08d%n", 123);         // 00000123
        System.out.printf("%+d%n", 123);          // +123

        // Floating point
        System.out.printf("%f%n", 3.14159);       // 3.141590
        System.out.printf("%.2f%n", 3.14159);     // 3.14
        System.out.printf("%e%n", 12345.6789);    // 1.234568e+04

        // Date/Time (with %t)
        Date now = new Date();
        System.out.printf("%tF%n", now);  // 2023-12-25 (ISO date)
        System.out.printf("%tT%n", now);  // 14:30:45 (24h time)
        System.out.printf("%tc%n", now);  // Mon Dec 25 14:30:45 EST 2023

        // Argument index
        System.out.printf("%2$s, %1$s!%n", "World", "Hello"); // Hello, World!
        System.out.printf("%1$s %1$s %1$s%n", "Hello");       // Hello Hello Hello
    }
}
```

### Text Blocks (Java 15+)

```java
public class TextBlockDemo {
    public static void main(String[] args) {
        // Traditional multi-line string
        String traditional = "{\n" +
                "  \"name\": \"John\",\n" +
                "  \"age\": 25\n" +
                "}";

        // Text block (Java 15+)
        String textBlock = """
                {
                  "name": "John",
                  "age": 25
                }
                """;

        // Incidental whitespace is removed
        // Essential whitespace is preserved

        // HTML
        String html = """
                <html>
                    <body>
                        <p>Hello, World!</p>
                    </body>
                </html>
                """;

        // SQL
        String sql = """
                SELECT id, name, email
                FROM users
                WHERE status = 'active'
                ORDER BY name
                """;

        // Escape sequences still work
        String withEscape = """
                Line 1\nLine 2
                Tab:\tHere
                Quote: \"quoted\"
                """;

        // Formatting with text blocks
        String template = """
                Hello, %s!
                Today is %s.
                Temperature: %.1f°C
                """.formatted("John", "Monday", 23.5);
    }
}
```

### MessageFormat

```java
import java.text.MessageFormat;

public class MessageFormatDemo {
    public static void main(String[] args) {
        // Basic usage
        String result = MessageFormat.format(
            "Hello, {0}! You have {1} messages.",
            "John", 5
        );
        System.out.println(result);  // Hello, John! You have 5 messages.

        // With formatting
        String formatted = MessageFormat.format(
            "Price: {0,number,currency}",
            1234.56
        );
        System.out.println(formatted);  // Price: $1,234.56

        // Date formatting
        String dateFormat = MessageFormat.format(
            "Today is {0,date,long}",
            new Date()
        );
        System.out.println(dateFormat);  // Today is December 25, 2023

        // Choice format (pluralization)
        String choice = MessageFormat.format(
            "You have {0,choice,0#no messages|1#one message|1<{0} messages}",
            5
        );
        System.out.println(choice);  // You have 5 messages
    }
}
```

---

## Regular Expressions

### Pattern and Matcher

```java
import java.util.regex.*;

public class RegexDemo {
    public static void main(String[] args) {
        // Basic matching
        String text = "Hello World";
        boolean matches = text.matches("Hello.*");  // true

        // Pattern and Matcher for reuse
        Pattern pattern = Pattern.compile("\\d+");  // One or more digits
        Matcher matcher = pattern.matcher("abc123def456");

        // Find all matches
        while (matcher.find()) {
            System.out.println("Found: " + matcher.group());
            System.out.println("Start: " + matcher.start());
            System.out.println("End: " + matcher.end());
        }
        // Found: 123, Start: 3, End: 6
        // Found: 456, Start: 9, End: 12

        // Replace
        String result = matcher.replaceAll("NUM");  // "abcNUMdefNUM"

        // String methods with regex
        String[] parts = "a,b;c:d".split("[,;:]");  // ["a", "b", "c", "d"]
        String replaced = "hello123".replaceAll("\\d+", "#");  // "hello#"
    }
}
```

### Common Regex Patterns

```java
public class RegexPatterns {
    // Email validation
    public static final String EMAIL =
        "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$";

    // Phone number (US format)
    public static final String PHONE =
        "^\\(?\\d{3}\\)?[-. ]?\\d{3}[-. ]?\\d{4}$";

    // URL
    public static final String URL =
        "^(https?://)?(www\\.)?[a-zA-Z0-9-]+\\.[a-zA-Z]{2,}(/\\S*)?$";

    // Date (YYYY-MM-DD)
    public static final String DATE =
        "^\\d{4}-\\d{2}-\\d{2}$";

    // Password (min 8 chars, 1 upper, 1 lower, 1 digit, 1 special)
    public static final String PASSWORD =
        "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$";

    // IP Address
    public static final String IP =
        "^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)$";

    public static void main(String[] args) {
        System.out.println("test@email.com".matches(EMAIL));  // true
        System.out.println("(123) 456-7890".matches(PHONE));  // true
        System.out.println("2023-12-25".matches(DATE));       // true
    }
}
```

### Named Groups (Java 7+)

```java
public class NamedGroupsDemo {
    public static void main(String[] args) {
        String pattern = "(?<year>\\d{4})-(?<month>\\d{2})-(?<day>\\d{2})";
        String date = "2023-12-25";

        Pattern p = Pattern.compile(pattern);
        Matcher m = p.matcher(date);

        if (m.matches()) {
            System.out.println("Year: " + m.group("year"));   // 2023
            System.out.println("Month: " + m.group("month")); // 12
            System.out.println("Day: " + m.group("day"));     // 25
        }
    }
}
```

---

## Java 11+ String Methods

```java
public class Java11StringMethods {
    public static void main(String[] args) {
        // isBlank() - true if empty or only whitespace
        System.out.println("".isBlank());      // true
        System.out.println("   ".isBlank());   // true
        System.out.println("  a  ".isBlank()); // false

        // lines() - returns Stream of lines
        String multiline = "line1\nline2\nline3";
        multiline.lines().forEach(System.out::println);
        // line1
        // line2
        // line3

        // strip(), stripLeading(), stripTrailing()
        String s = "  Hello World  ";
        System.out.println(s.strip());          // "Hello World"
        System.out.println(s.stripLeading());   // "Hello World  "
        System.out.println(s.stripTrailing());  // "  Hello World"

        // strip() vs trim() - strip handles Unicode whitespace
        String unicode = "\u2003Hello\u2003";  // Em space
        System.out.println(unicode.trim());    // Still has em spaces
        System.out.println(unicode.strip());   // No em spaces

        // repeat()
        System.out.println("Ha".repeat(3));    // "HaHaHa"
        System.out.println("-".repeat(20));    // "--------------------"
    }
}
```

### Java 12+ String Methods

```java
public class Java12StringMethods {
    public static void main(String[] args) {
        // indent() - adds/removes indentation
        String s = "Hello\nWorld";
        System.out.println(s.indent(4));
        //     Hello
        //     World

        System.out.println("    Hello".indent(-2));  // "  Hello"

        // transform() - apply function to string
        String result = "hello"
            .transform(String::toUpperCase)
            .transform(s2 -> s2 + "!")
            .transform(s2 -> s2.repeat(2));
        System.out.println(result);  // "HELLO!HELLO!"
    }
}
```

### Java 15+ String Methods (with Text Blocks)

```java
public class Java15StringMethods {
    public static void main(String[] args) {
        // formatted() - instance method version of format()
        String template = "Hello, %s! You are %d years old.";
        String result = template.formatted("John", 25);
        System.out.println(result);

        // stripIndent() - removes incidental whitespace
        String indented = """
                Hello
                World
                """;
        System.out.println(indented.stripIndent());

        // translateEscapes() - translate escape sequences
        String withEscapes = "Hello\\nWorld";
        System.out.println(withEscapes.translateEscapes());
        // Hello
        // World
    }
}
```

---

## Performance Considerations

### String Concatenation Optimization

```java
public class ConcatenationOptimization {
    public static void main(String[] args) {
        // Compiler optimizes literals
        String s1 = "Hello" + " " + "World";  // Computed at compile time
        // Bytecode: String s1 = "Hello World";

        // But NOT with variables
        String a = "Hello";
        String b = "World";
        String s2 = a + " " + b;  // Creates StringBuilder at runtime

        // This is what happens:
        // String s2 = new StringBuilder().append(a).append(" ").append(b).toString();

        // In loops, use StringBuilder explicitly
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 100; i++) {
            sb.append(i).append(",");
        }
        String result = sb.toString();
    }
}
```

### String Deduplication (G1 GC)

```java
// JVM option: -XX:+UseG1GC -XX:+UseStringDeduplication

// G1 GC can deduplicate strings in the heap
// Strings with same content share the same char[]/byte[]
// Reduces memory usage for applications with many duplicate strings

// Especially useful for:
// - Large-scale applications
// - Applications reading lots of data (CSV, JSON parsing)
// - Long-running applications
```

### Compact Strings (Java 9+)

```java
// Java 9 introduced compact strings
// Strings containing only Latin-1 characters use byte[] (1 byte per char)
// instead of char[] (2 bytes per char)

// Latin-1 characters: ASCII (0-127) + Latin-1 supplement (128-255)
String ascii = "Hello";        // Uses 1 byte per char internally
String unicode = "Hello世界";  // Uses 2 bytes per char internally

// This is transparent to the programmer
// Same API, automatic optimization
```

---

## Interview Questions

### Q1: What is the String Pool?

**Answer:**
String Pool is a special memory area in the heap where Java stores **string literals**. When you create a string literal, JVM first checks the pool. If the string exists, it returns the reference; otherwise, it creates a new string in the pool.

```java
String s1 = "Hello";  // Created in pool
String s2 = "Hello";  // Reuses from pool
System.out.println(s1 == s2);  // true

String s3 = new String("Hello");  // New object in heap
System.out.println(s1 == s3);  // false
```

---

### Q2: Why is String immutable in Java?

**Answer:**

1. **Security**: Strings used for passwords, network connections, file paths
2. **Thread Safety**: Immutable objects are inherently thread-safe
3. **Caching**: String pool works because strings can't change
4. **HashCode Caching**: hashCode is computed once and cached
5. **Class Loading**: Class names are strings; mutability would be a security risk

---

### Q3: Difference between == and equals() for Strings?

**Answer:**

- `==` compares **references** (memory addresses)
- `equals()` compares **content** (characters)

```java
String s1 = "Hello";
String s2 = new String("Hello");

s1 == s2;       // false (different references)
s1.equals(s2);  // true (same content)
```

---

### Q4: How many objects are created?

```java
String s1 = "Hello";
String s2 = new String("Hello");
String s3 = s1 + s2;
```

**Answer:**

- Line 1: 1 object (in pool, if not exists)
- Line 2: 1 object (in heap) - "Hello" already in pool
- Line 3: 1 or 2 objects (StringBuilder + result String)

**Total: 3-4 objects** (depends on JVM optimization)

---

### Q5: Difference between String, StringBuilder, and StringBuffer?

**Answer:**

| Feature       | String                   | StringBuilder      | StringBuffer                |
| ------------- | ------------------------ | ------------------ | --------------------------- |
| Mutability    | Immutable                | Mutable            | Mutable                     |
| Thread Safety | Yes                      | No                 | Yes (synchronized)          |
| Performance   | Slow for concatenation   | Fastest            | Slower than StringBuilder   |
| Memory        | Uses pool                | No pool            | No pool                     |
| When to Use   | Constants, small changes | Heavy manipulation | Multi-threaded manipulation |

---

### Q6: What is the output?

```java
String s1 = "Hello";
String s2 = "Hel" + "lo";
String s3 = "Hel";
String s4 = s3 + "lo";

System.out.println(s1 == s2);
System.out.println(s1 == s4);
```

**Answer:**

```
true   // s2 is compile-time constant, same as s1 in pool
false  // s4 involves variable, creates new object at runtime
```

---

### Q7: How to check if String is palindrome?

```java
public static boolean isPalindrome(String str) {
    // Method 1: StringBuilder reverse
    String reversed = new StringBuilder(str).reverse().toString();
    return str.equals(reversed);

    // Method 2: Two pointers
    int left = 0, right = str.length() - 1;
    while (left < right) {
        if (str.charAt(left) != str.charAt(right)) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

---

### Q8: What is intern() method?

**Answer:**
`intern()` returns a canonical representation of the string from the String Pool.

```java
String s1 = new String("Hello");  // Heap object
String s2 = s1.intern();          // Pool reference
String s3 = "Hello";              // Pool reference

System.out.println(s1 == s2);  // false
System.out.println(s2 == s3);  // true
```

**Use case**: Memory optimization when many duplicate strings exist.

---

### Q9: String s = new String("Hello"); How many objects?

**Answer:**

- If "Hello" not in pool: **2 objects** (one in pool for literal, one in heap)
- If "Hello" already in pool: **1 object** (only heap object)

The literal "Hello" in the constructor argument goes to the pool (if not exists), and `new String()` creates another object in the heap.

---

### Q10: Difference between trim() and strip()?

**Answer:**

- `trim()` removes ASCII whitespace (characters <= ' ')
- `strip()` (Java 11+) removes Unicode whitespace

```java
String s = "\u2003Hello\u2003";  // Em space (Unicode)
s.trim();   // Still has em spaces
s.strip();  // Em spaces removed
```

---

## Common Interview Traps

### Trap 1: "String is primitive"

**No!** String is a class, but has special language support (literals, +).

### Trap 2: "StringBuilder is thread-safe"

**No!** StringBuilder is NOT thread-safe. StringBuffer is.

### Trap 3: "== works for Strings"

Only for pool strings. Always use `equals()` for comparison.

### Trap 4: "String + creates new String always"

Compiler optimizes concatenation of literals:

```java
String s = "Hello" + " " + "World";  // Computed at compile time
```

### Trap 5: "String concatenation in loop is fine"

Very inefficient! Creates many intermediate objects. Use StringBuilder.

---

## Key Takeaways

1. **String is immutable** - every modification creates new object
2. **String Pool** saves memory for literals
3. **Use StringBuilder** for concatenation in loops
4. **Use equals()** for content comparison, not ==
5. **intern()** adds string to pool and returns pool reference
6. **Java 9+**: Compact strings save memory for ASCII
7. **Java 11+**: New methods like strip(), isBlank(), lines()
8. **Java 15+**: Text blocks for multi-line strings
9. **String is final** - cannot be extended
10. **HashCode is cached** - efficient for HashMap keys
