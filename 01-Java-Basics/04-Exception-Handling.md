# Exception Handling in Java - Complete Guide

## Table of Contents

1. [Introduction to Exceptions](#introduction-to-exceptions)
2. [Exception Hierarchy](#exception-hierarchy)
3. [Types of Exceptions](#types-of-exceptions)
4. [Exception Handling Keywords](#exception-handling-keywords)
5. [try-catch-finally](#try-catch-finally)
6. [try-with-resources](#try-with-resources)
7. [throw and throws](#throw-and-throws)
8. [Custom Exceptions](#custom-exceptions)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction to Exceptions

### What is an Exception?

An **exception** is an event that disrupts the normal flow of program execution. It's an object that wraps an error event and contains information about the error.

### Why Exception Handling?

1. **Graceful Degradation**: Handle errors without crashing
2. **Separation of Concerns**: Separate error-handling code from business logic
3. **Error Propagation**: Pass errors up the call stack
4. **Meaningful Messages**: Provide useful error information
5. **Resource Management**: Ensure cleanup even when errors occur

### What Happens Without Exception Handling?

```java
public class NoExceptionHandling {
    public static void main(String[] args) {
        int result = 10 / 0;  // ArithmeticException
        System.out.println("This will never execute");
    }
}

// Output:
// Exception in thread "main" java.lang.ArithmeticException: / by zero
//     at NoExceptionHandling.main(NoExceptionHandling.java:3)
// Program terminates abruptly!
```

### With Exception Handling

```java
public class WithExceptionHandling {
    public static void main(String[] args) {
        try {
            int result = 10 / 0;
        } catch (ArithmeticException e) {
            System.out.println("Cannot divide by zero!");
        }
        System.out.println("Program continues normally");
    }
}

// Output:
// Cannot divide by zero!
// Program continues normally
```

---

## Exception Hierarchy

```
                        java.lang.Object
                              │
                        java.lang.Throwable
                              │
              ┌───────────────┴───────────────┐
              │                               │
        java.lang.Error              java.lang.Exception
              │                               │
    (Unchecked - System)         ┌────────────┴────────────┐
              │                  │                         │
    ┌─────────┼─────────┐   RuntimeException      Checked Exceptions
    │         │         │   (Unchecked)                    │
OutOfMemory  Stack    Virtual       │           ┌─────────┼─────────┐
  Error    Overflow  Machine      ┌─┼─┐       IOException SQLException
             Error    Error       │ │ │       ClassNotFound  etc.
                                  │ │ │       Exception
                        NullPointer │ ArrayIndexOutOf
                        Exception   │   BoundsException
                                    │
                        IllegalArgumentException
                        NumberFormatException
                        ClassCastException
```

### Throwable Class

```java
public class ThrowableDemo {
    public static void main(String[] args) {
        try {
            throw new Exception("Something went wrong");
        } catch (Exception e) {
            // getMessage() - Returns detail message
            System.out.println("Message: " + e.getMessage());

            // toString() - Returns short description
            System.out.println("String: " + e.toString());

            // printStackTrace() - Prints stack trace
            e.printStackTrace();

            // getStackTrace() - Returns array of stack trace elements
            StackTraceElement[] stackTrace = e.getStackTrace();
            for (StackTraceElement element : stackTrace) {
                System.out.println("Class: " + element.getClassName());
                System.out.println("Method: " + element.getMethodName());
                System.out.println("Line: " + element.getLineNumber());
            }

            // getCause() - Returns cause of exception
            Throwable cause = e.getCause();
        }
    }
}
```

---

## Types of Exceptions

### 1. Checked Exceptions (Compile-time)

Must be handled or declared. Compiler forces you to deal with them.

```java
public class CheckedExceptionDemo {
    // Must declare throws or handle in try-catch
    public void readFile(String path) throws IOException {
        FileReader reader = new FileReader(path);  // FileNotFoundException
        BufferedReader br = new BufferedReader(reader);
        String line = br.readLine();  // IOException
        br.close();
    }

    public void parseDate(String date) throws ParseException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        Date d = sdf.parse(date);  // ParseException
    }

    public void loadClass(String name) throws ClassNotFoundException {
        Class.forName(name);  // ClassNotFoundException
    }
}

// Common Checked Exceptions:
// - IOException, FileNotFoundException
// - SQLException
// - ParseException
// - ClassNotFoundException
// - InterruptedException
// - CloneNotSupportedException
```

### 2. Unchecked Exceptions (Runtime)

Not required to be caught or declared. Usually indicate programming errors.

```java
public class UncheckedExceptionDemo {
    public void demonstrateRuntimeExceptions() {
        // NullPointerException
        String str = null;
        // str.length();  // NPE

        // ArrayIndexOutOfBoundsException
        int[] arr = {1, 2, 3};
        // int x = arr[5];  // AIOOBE

        // ArithmeticException
        // int y = 10 / 0;  // AE

        // NumberFormatException
        // int z = Integer.parseInt("abc");  // NFE

        // ClassCastException
        Object obj = "Hello";
        // Integer num = (Integer) obj;  // CCE

        // IllegalArgumentException
        // Thread.sleep(-1000);  // IAE

        // IllegalStateException
        List<String> list = new ArrayList<>();
        Iterator<String> it = list.iterator();
        // it.remove();  // ISE - remove before next

        // ConcurrentModificationException
        List<String> names = new ArrayList<>(Arrays.asList("A", "B"));
        // for (String name : names) {
        //     names.remove(name);  // CME
        // }
    }
}

// Common Unchecked Exceptions:
// - NullPointerException
// - ArrayIndexOutOfBoundsException
// - ArithmeticException
// - NumberFormatException
// - ClassCastException
// - IllegalArgumentException
// - IllegalStateException
// - UnsupportedOperationException
```

### 3. Errors

Serious problems that applications should not try to catch.

```java
public class ErrorDemo {
    // StackOverflowError - Infinite recursion
    public void infiniteRecursion() {
        infiniteRecursion();  // SOE
    }

    // OutOfMemoryError - Heap exhausted
    public void outOfMemory() {
        List<byte[]> list = new ArrayList<>();
        while (true) {
            list.add(new byte[1024 * 1024]);  // OOME
        }
    }

    // NoClassDefFoundError - Class found at compile, not at runtime
    // VirtualMachineError - JVM issues
}
```

### Checked vs Unchecked Comparison

| Aspect             | Checked                   | Unchecked               | Error                      |
| ------------------ | ------------------------- | ----------------------- | -------------------------- |
| Superclass         | Exception                 | RuntimeException        | Error                      |
| Compile-time check | Yes                       | No                      | No                         |
| Must handle        | Yes                       | No                      | No                         |
| Recovery           | Usually possible          | Usually programming bug | Usually not possible       |
| Examples           | IOException, SQLException | NPE, ArrayIndex         | OutOfMemory, StackOverflow |

---

## Exception Handling Keywords

### Five Keywords

| Keyword   | Purpose                                 |
| --------- | --------------------------------------- |
| `try`     | Block of code to monitor for exceptions |
| `catch`   | Block to handle specific exception      |
| `finally` | Block that always executes              |
| `throw`   | Explicitly throw an exception           |
| `throws`  | Declare exceptions a method can throw   |

---

## try-catch-finally

### Basic Structure

```java
public class TryCatchFinallyDemo {
    public static void main(String[] args) {
        try {
            // Code that might throw exception
            int result = 10 / 0;
            System.out.println("Result: " + result);
        } catch (ArithmeticException e) {
            // Handle the exception
            System.out.println("Cannot divide by zero: " + e.getMessage());
        } finally {
            // Always executes (cleanup code)
            System.out.println("Finally block executed");
        }
        System.out.println("Program continues");
    }
}

// Output:
// Cannot divide by zero: / by zero
// Finally block executed
// Program continues
```

### Multiple Catch Blocks

```java
public class MultipleCatchDemo {
    public static void main(String[] args) {
        try {
            String str = null;
            int[] arr = {1, 2, 3};

            // This could throw multiple types of exceptions
            System.out.println(str.length());  // NullPointerException
            System.out.println(arr[5]);        // ArrayIndexOutOfBoundsException
            int x = Integer.parseInt("abc");   // NumberFormatException

        } catch (NullPointerException e) {
            System.out.println("Null pointer: " + e.getMessage());
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Array index: " + e.getMessage());
        } catch (NumberFormatException e) {
            System.out.println("Number format: " + e.getMessage());
        } catch (Exception e) {
            // Catch-all for any other exceptions
            System.out.println("General exception: " + e.getMessage());
        }
    }
}
```

### Multi-Catch (Java 7+)

```java
public class MultiCatchDemo {
    public static void main(String[] args) {
        try {
            // Some code that can throw multiple exceptions
            riskyOperation();
        } catch (IOException | SQLException e) {
            // Handle both exceptions the same way
            System.out.println("IO or SQL error: " + e.getMessage());
            // Note: 'e' is implicitly final
            // e = new IOException();  // Compilation error
        } catch (Exception e) {
            System.out.println("Other error: " + e.getMessage());
        }
    }

    static void riskyOperation() throws IOException, SQLException {
        // ...
    }
}

// Rules for multi-catch:
// 1. Exceptions in multi-catch cannot have inheritance relationship
// catch (Exception | IOException e) { }  // ERROR: IOException is subclass
// 2. The exception variable is implicitly final
```

### Catch Block Order

```java
public class CatchOrderDemo {
    public static void main(String[] args) {
        try {
            throw new FileNotFoundException("File not found");
        }
        // WRONG ORDER - compilation error
        // catch (Exception e) { }
        // catch (IOException e) { }  // Unreachable
        // catch (FileNotFoundException e) { }  // Unreachable

        // CORRECT ORDER - most specific first
        catch (FileNotFoundException e) {
            System.out.println("File not found");
        } catch (IOException e) {
            System.out.println("IO error");
        } catch (Exception e) {
            System.out.println("General error");
        }
    }
}
```

### finally Block Behavior

```java
public class FinallyDemo {
    // Finally always executes (with one exception - System.exit())

    public static void main(String[] args) {
        // Case 1: No exception
        System.out.println("Case 1: " + case1());

        // Case 2: Exception caught
        System.out.println("Case 2: " + case2());

        // Case 3: Return in try
        System.out.println("Case 3: " + case3());

        // Case 4: Return in finally (overwrites try's return)
        System.out.println("Case 4: " + case4());
    }

    static int case1() {
        try {
            return 1;
        } finally {
            System.out.println("Finally in case1");
        }
    }

    static int case2() {
        try {
            int x = 10 / 0;
            return 1;
        } catch (Exception e) {
            return 2;
        } finally {
            System.out.println("Finally in case2");
        }
    }

    static int case3() {
        try {
            return 1;
        } finally {
            System.out.println("Finally in case3");
            // return 3;  // Bad practice - overrides try's return
        }
    }

    static int case4() {
        try {
            return 1;
        } finally {
            return 4;  // WARNING: This overrides the return in try
        }
    }
}

// Output:
// Finally in case1
// Case 1: 1
// Finally in case2
// Case 2: 2
// Finally in case3
// Case 3: 1
// Case 4: 4
```

### finally NOT Executed

```java
public class FinallyNotExecuted {
    public static void main(String[] args) {
        try {
            System.out.println("In try");
            System.exit(0);  // JVM terminates
        } finally {
            System.out.println("In finally");  // Never executes
        }
    }
}

// Also won't execute if:
// - JVM crashes
// - Thread is killed
// - Infinite loop in try/catch
// - Power failure
```

---

## try-with-resources

### What is try-with-resources?

Introduced in Java 7, it automatically closes resources that implement `AutoCloseable` interface.

### Without try-with-resources (Old Way)

```java
public class OldResourceHandling {
    public void readFile(String path) {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader(path));
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();  // Can also throw exception!
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### With try-with-resources (Modern Way)

```java
public class ModernResourceHandling {
    public void readFile(String path) {
        try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // reader is automatically closed here
    }

    // Multiple resources
    public void copyFile(String src, String dest) {
        try (FileInputStream in = new FileInputStream(src);
             FileOutputStream out = new FileOutputStream(dest)) {

            byte[] buffer = new byte[1024];
            int length;
            while ((length = in.read(buffer)) > 0) {
                out.write(buffer, 0, length);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // Both streams closed automatically (in reverse order)
    }
}
```

### AutoCloseable Interface

```java
public class CustomResource implements AutoCloseable {
    private String name;

    public CustomResource(String name) {
        this.name = name;
        System.out.println(name + " opened");
    }

    public void doWork() {
        System.out.println(name + " working");
    }

    @Override
    public void close() {
        System.out.println(name + " closed");
    }
}

// Usage
public class CustomResourceDemo {
    public static void main(String[] args) {
        try (CustomResource r1 = new CustomResource("Resource1");
             CustomResource r2 = new CustomResource("Resource2")) {

            r1.doWork();
            r2.doWork();
        }
        // Output:
        // Resource1 opened
        // Resource2 opened
        // Resource1 working
        // Resource2 working
        // Resource2 closed  (reverse order)
        // Resource1 closed
    }
}
```

### Effectively Final Variables (Java 9+)

```java
// Java 9 allows effectively final variables in try-with-resources
public class Java9TryWithResources {
    public void process(BufferedReader reader) throws IOException {
        // reader is effectively final
        try (reader) {  // Java 9+ syntax
            String line = reader.readLine();
            System.out.println(line);
        }
    }
}
```

### Suppressed Exceptions

```java
public class SuppressedExceptionsDemo {
    public static void main(String[] args) {
        try (ProblemResource r = new ProblemResource()) {
            r.doWork();  // Throws Exception A
        } catch (Exception e) {
            System.out.println("Main exception: " + e.getMessage());

            // Get suppressed exceptions
            Throwable[] suppressed = e.getSuppressed();
            for (Throwable t : suppressed) {
                System.out.println("Suppressed: " + t.getMessage());
            }
        }
    }
}

class ProblemResource implements AutoCloseable {
    public void doWork() throws Exception {
        throw new Exception("Exception A: Work failed");
    }

    @Override
    public void close() throws Exception {
        throw new Exception("Exception B: Close failed");
    }
}

// Output:
// Main exception: Exception A: Work failed
// Suppressed: Exception B: Close failed
```

---

## throw and throws

### throw Keyword

Used to explicitly throw an exception.

```java
public class ThrowDemo {
    public void validateAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
        if (age < 18) {
            throw new IllegalArgumentException("Must be 18 or older");
        }
        System.out.println("Valid age: " + age);
    }

    public void validateNotNull(Object obj) {
        if (obj == null) {
            throw new NullPointerException("Object cannot be null");
        }
    }

    public void processFile(String path) throws IOException {
        if (path == null || path.isEmpty()) {
            throw new IllegalArgumentException("Path cannot be null or empty");
        }
        File file = new File(path);
        if (!file.exists()) {
            throw new FileNotFoundException("File not found: " + path);
        }
    }
}
```

### throws Keyword

Used in method signature to declare exceptions that method might throw.

```java
public class ThrowsDemo {
    // Declare single exception
    public void method1() throws IOException {
        throw new IOException("IO error");
    }

    // Declare multiple exceptions
    public void method2() throws IOException, SQLException {
        // Method can throw either exception
    }

    // Propagating exceptions up the call stack
    public void caller() throws IOException {
        method1();  // Don't handle, let it propagate
    }

    // Handling vs Declaring
    public void handler() {
        try {
            method1();  // Handle here
        } catch (IOException e) {
            System.out.println("Handled: " + e.getMessage());
        }
    }
}
```

### throw vs throws

| Aspect      | throw                      | throws                      |
| ----------- | -------------------------- | --------------------------- |
| Usage       | Inside method body         | In method declaration       |
| Purpose     | Actually throw exception   | Declare possible exceptions |
| Followed by | Exception object           | Exception class names       |
| Count       | Single exception at a time | Multiple exceptions         |
| Checked     | Both checked and unchecked | Mainly for checked          |

### Re-throwing Exceptions

```java
public class RethrowDemo {
    // Simple rethrow
    public void method1() throws IOException {
        try {
            // Some code
            throw new IOException("Original error");
        } catch (IOException e) {
            System.out.println("Logging: " + e.getMessage());
            throw e;  // Rethrow same exception
        }
    }

    // Wrap in different exception
    public void method2() throws ServiceException {
        try {
            // Some code
            throw new SQLException("Database error");
        } catch (SQLException e) {
            throw new ServiceException("Service failed", e);  // Wrap
        }
    }

    // More precise rethrow (Java 7+)
    public void method3(int type) throws FirstException, SecondException {
        try {
            if (type == 1) throw new FirstException();
            if (type == 2) throw new SecondException();
        } catch (Exception e) {
            // Compiler knows exactly which exceptions can be thrown
            throw e;  // No need to declare Exception in throws
        }
    }
}
```

---

## Custom Exceptions

### When to Create Custom Exceptions?

1. When no existing exception fits the scenario
2. When you need additional information
3. When you want domain-specific exceptions
4. When you need exception chaining

### Creating Custom Checked Exception

```java
// Custom checked exception
public class InsufficientFundsException extends Exception {
    private double amount;
    private double balance;

    public InsufficientFundsException(String message) {
        super(message);
    }

    public InsufficientFundsException(String message, double amount, double balance) {
        super(message);
        this.amount = amount;
        this.balance = balance;
    }

    public InsufficientFundsException(String message, Throwable cause) {
        super(message, cause);
    }

    public double getAmount() {
        return amount;
    }

    public double getBalance() {
        return balance;
    }

    public double getShortfall() {
        return amount - balance;
    }
}

// Usage
public class BankAccount {
    private double balance;

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(
                "Cannot withdraw " + amount + ". Balance is " + balance,
                amount,
                balance
            );
        }
        balance -= amount;
    }
}
```

### Creating Custom Unchecked Exception

```java
// Custom unchecked exception
public class InvalidOrderException extends RuntimeException {
    private String orderId;
    private String reason;

    public InvalidOrderException(String message) {
        super(message);
    }

    public InvalidOrderException(String message, String orderId, String reason) {
        super(message);
        this.orderId = orderId;
        this.reason = reason;
    }

    public InvalidOrderException(String message, Throwable cause) {
        super(message, cause);
    }

    public String getOrderId() {
        return orderId;
    }

    public String getReason() {
        return reason;
    }
}

// Usage
public class OrderService {
    public void processOrder(Order order) {
        if (order == null) {
            throw new InvalidOrderException("Order cannot be null");
        }
        if (order.getItems().isEmpty()) {
            throw new InvalidOrderException(
                "Order has no items",
                order.getId(),
                "Empty order"
            );
        }
    }
}
```

### Exception Hierarchy for Application

```java
// Base exception for application
public abstract class ApplicationException extends Exception {
    private String errorCode;
    private LocalDateTime timestamp;

    public ApplicationException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
        this.timestamp = LocalDateTime.now();
    }

    public ApplicationException(String message, String errorCode, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
        this.timestamp = LocalDateTime.now();
    }

    public String getErrorCode() { return errorCode; }
    public LocalDateTime getTimestamp() { return timestamp; }
}

// Specific exceptions
public class ValidationException extends ApplicationException {
    private List<String> violations;

    public ValidationException(String message, List<String> violations) {
        super(message, "ERR_VALIDATION");
        this.violations = violations;
    }

    public List<String> getViolations() { return violations; }
}

public class DataNotFoundException extends ApplicationException {
    private String entityType;
    private Object entityId;

    public DataNotFoundException(String entityType, Object entityId) {
        super(entityType + " not found with id: " + entityId, "ERR_NOT_FOUND");
        this.entityType = entityType;
        this.entityId = entityId;
    }
}

public class ServiceUnavailableException extends ApplicationException {
    private String serviceName;

    public ServiceUnavailableException(String serviceName, Throwable cause) {
        super("Service unavailable: " + serviceName, "ERR_SERVICE", cause);
        this.serviceName = serviceName;
    }
}
```

---

## Best Practices

### 1. Use Specific Exceptions

```java
// BAD - Too generic
public void process() throws Exception {
    // ...
}

// GOOD - Specific exception
public void process() throws ValidationException, DataNotFoundException {
    // ...
}
```

### 2. Catch Specific Exceptions

```java
// BAD - Catches everything including programming errors
try {
    riskyOperation();
} catch (Exception e) {
    // NPE would also be caught here!
}

// GOOD - Catch what you expect
try {
    riskyOperation();
} catch (IOException e) {
    // Handle IO error
} catch (SQLException e) {
    // Handle SQL error
}
```

### 3. Don't Swallow Exceptions

```java
// BAD - Silent failure
try {
    riskyOperation();
} catch (Exception e) {
    // Empty catch block - exception ignored!
}

// ALSO BAD - Only logging
try {
    riskyOperation();
} catch (Exception e) {
    e.printStackTrace();  // Then what?
}

// GOOD - Handle appropriately
try {
    riskyOperation();
} catch (IOException e) {
    logger.error("Failed to process: " + e.getMessage(), e);
    throw new ServiceException("Processing failed", e);
}
```

### 4. Use Finally or try-with-resources

```java
// BAD - Resource might not be closed
FileInputStream fis = new FileInputStream("file.txt");
// If exception here, stream is never closed

// GOOD - try-with-resources
try (FileInputStream fis = new FileInputStream("file.txt")) {
    // Use the resource
}  // Automatically closed
```

### 5. Don't Use Exceptions for Flow Control

```java
// BAD - Using exception for normal flow
public boolean containsElement(int[] arr, int element) {
    try {
        int index = findIndex(arr, element);
        return true;
    } catch (ElementNotFoundException e) {
        return false;
    }
}

// GOOD - Normal control flow
public boolean containsElement(int[] arr, int element) {
    for (int value : arr) {
        if (value == element) return true;
    }
    return false;
}
```

### 6. Document Exceptions with Javadoc

```java
/**
 * Withdraws money from the account.
 *
 * @param amount the amount to withdraw (must be positive)
 * @throws IllegalArgumentException if amount is not positive
 * @throws InsufficientFundsException if account balance is less than amount
 */
public void withdraw(double amount) throws InsufficientFundsException {
    if (amount <= 0) {
        throw new IllegalArgumentException("Amount must be positive");
    }
    if (amount > balance) {
        throw new InsufficientFundsException("Insufficient funds");
    }
    balance -= amount;
}
```

### 7. Preserve the Original Exception

```java
// BAD - Original exception lost
try {
    database.query(sql);
} catch (SQLException e) {
    throw new ServiceException("Query failed");  // Original cause lost!
}

// GOOD - Chain exceptions
try {
    database.query(sql);
} catch (SQLException e) {
    throw new ServiceException("Query failed", e);  // Original preserved
}
```

### 8. Log at the Right Level

```java
try {
    process();
} catch (RecoverableException e) {
    logger.warn("Recoverable error, retrying: {}", e.getMessage());
    retry();
} catch (CriticalException e) {
    logger.error("Critical failure: {}", e.getMessage(), e);
    throw e;
}
```

### 9. Clean Up in Finally (Pre-Java 7)

```java
Connection conn = null;
PreparedStatement stmt = null;
ResultSet rs = null;

try {
    conn = dataSource.getConnection();
    stmt = conn.prepareStatement(sql);
    rs = stmt.executeQuery();
    // Process results
} catch (SQLException e) {
    throw new DataAccessException("Query failed", e);
} finally {
    // Close in reverse order
    closeQuietly(rs);
    closeQuietly(stmt);
    closeQuietly(conn);
}

private void closeQuietly(AutoCloseable resource) {
    if (resource != null) {
        try {
            resource.close();
        } catch (Exception e) {
            logger.warn("Failed to close resource", e);
        }
    }
}
```

### 10. Fail Fast

```java
// Validate inputs at the beginning
public void processOrder(Order order) {
    // Fail fast with clear messages
    Objects.requireNonNull(order, "Order cannot be null");

    if (order.getItems() == null || order.getItems().isEmpty()) {
        throw new IllegalArgumentException("Order must have at least one item");
    }

    if (order.getCustomerId() <= 0) {
        throw new IllegalArgumentException("Invalid customer ID");
    }

    // Now process valid order
    doProcessOrder(order);
}
```

---

## Interview Questions

### Q1: What is the difference between throw and throws?

**Answer:**

| throw                          | throws                       |
| ------------------------------ | ---------------------------- |
| Used inside method body        | Used in method signature     |
| Throws one exception at a time | Declares multiple exceptions |
| Followed by exception instance | Followed by exception class  |
| Used to actually throw         | Used to declare possibility  |

```java
// throw - actually throwing
public void validate(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Invalid age");
    }
}

// throws - declaring
public void readFile(String path) throws IOException, FileNotFoundException {
    // ...
}
```

---

### Q2: What is the difference between final, finally, and finalize?

**Answer:**

| Keyword      | Type     | Purpose                                                              |
| ------------ | -------- | -------------------------------------------------------------------- |
| `final`      | Modifier | Constant (variable), Cannot override (method), Cannot extend (class) |
| `finally`    | Block    | Guaranteed execution after try/catch                                 |
| `finalize()` | Method   | Called by GC before object destruction (deprecated)                  |

```java
// final
final int x = 10;  // Constant
final void method() { }  // Cannot override
final class MyClass { }  // Cannot extend

// finally
try {
    // ...
} finally {
    // Always executes
}

// finalize (deprecated)
@Override
protected void finalize() throws Throwable {
    // Cleanup before GC
    super.finalize();
}
```

---

### Q3: Can we have try without catch?

**Answer:**
Yes, but only in two ways:

1. **try-finally** (without catch)

```java
try {
    // Code
} finally {
    // Cleanup
}
```

2. **try-with-resources** (without catch and finally)

```java
try (Resource r = new Resource()) {
    // Use resource
}
// Resource automatically closed
```

---

### Q4: What is exception propagation?

**Answer:**
When an exception occurs and is not handled, it propagates up the call stack until it's handled or reaches the main method.

```java
public class PropagationDemo {
    public static void main(String[] args) {
        try {
            method1();
        } catch (Exception e) {
            System.out.println("Caught in main");
        }
    }

    static void method1() {
        method2();  // Exception propagates from here
    }

    static void method2() {
        method3();  // Exception propagates from here
    }

    static void method3() {
        int x = 10 / 0;  // ArithmeticException thrown
    }
}
```

**Propagation path:** method3 → method2 → method1 → main (caught)

---

### Q5: What is the output?

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(test());
    }

    static int test() {
        try {
            return 1;
        } catch (Exception e) {
            return 2;
        } finally {
            return 3;
        }
    }
}
```

**Answer:** `3`

The finally block's return overwrites the try block's return. This is why returning from finally is considered bad practice.

---

### Q6: Can we catch Error?

**Answer:**
Yes, but you **shouldn't**. Errors indicate serious problems that cannot be recovered from.

```java
try {
    // Infinite recursion
    recursiveMethod();
} catch (StackOverflowError e) {
    // Technically possible but not recommended
    System.out.println("Stack overflow caught");
}
```

**Why not catch Error:**

- Errors indicate JVM/system problems
- Recovery is usually not possible
- Catching may hide serious issues
- May leave system in inconsistent state

---

### Q7: What happens if exception is thrown from finally block?

**Answer:**
If both try/catch and finally throw exceptions, the finally exception masks the original.

```java
try {
    throw new Exception("From try");
} finally {
    throw new RuntimeException("From finally");  // This is thrown
}
// RuntimeException is thrown, original Exception is lost
```

**Solution:** Never throw from finally, or use try-with-resources with suppressed exceptions.

---

### Q8: Difference between ClassNotFoundException and NoClassDefFoundError?

**Answer:**

| ClassNotFoundException       | NoClassDefFoundError                   |
| ---------------------------- | -------------------------------------- |
| Checked Exception            | Error                                  |
| Class.forName(), loadClass() | Class loading at runtime               |
| Class not found at runtime   | Class found at compile, not at runtime |
| Can be handled               | Usually fatal                          |

```java
// ClassNotFoundException
try {
    Class.forName("com.NonExistent");
} catch (ClassNotFoundException e) {
    // Handle
}

// NoClassDefFoundError
// Occurs when a class was present during compilation
// but missing at runtime (dependency issue)
```

---

### Q9: What is exception chaining?

**Answer:**
Exception chaining links exceptions to show the cause-effect relationship.

```java
public void processData() throws ServiceException {
    try {
        database.query();
    } catch (SQLException e) {
        // Chain the original exception
        throw new ServiceException("Failed to process data", e);
    }
}

// Later, get the chain
catch (ServiceException e) {
    System.out.println("Service error: " + e.getMessage());
    Throwable cause = e.getCause();
    System.out.println("Caused by: " + cause.getMessage());
}
```

---

### Q10: What are suppressed exceptions?

**Answer:**
When try-with-resources has exceptions in both the try block and close(), the close() exception is added as suppressed.

```java
try (Resource r = new Resource()) {
    throw new Exception("Primary");  // Main exception
}
// If r.close() throws, it becomes suppressed

// Access suppressed exceptions
catch (Exception e) {
    for (Throwable t : e.getSuppressed()) {
        System.out.println("Suppressed: " + t.getMessage());
    }
}
```

---

### Q11: Can we override a method that throws checked exception with a method that doesn't throw?

**Answer:**
Yes. Subclass method can throw:

- Same exception
- Subclass exception
- No exception
- Cannot throw new/broader checked exception

```java
class Parent {
    void method() throws IOException { }
}

class Child extends Parent {
    @Override
    void method() { }  // OK - no exception

    // @Override
    // void method() throws Exception { }  // ERROR - broader

    @Override
    void method() throws FileNotFoundException { }  // OK - subclass
}
```

---

### Q12: What is a multi-catch block and its limitations?

**Answer:**
Multi-catch (Java 7+) allows catching multiple exception types in one catch block.

```java
try {
    // Code
} catch (IOException | SQLException e) {
    // Handle both the same way
}
```

**Limitations:**

1. Exception types cannot have inheritance relationship
2. Exception variable is implicitly final
3. Cannot modify the caught exception

```java
// ERROR - FileNotFoundException is subclass of IOException
catch (IOException | FileNotFoundException e) { }

// ERROR - variable is final
catch (IOException | SQLException e) {
    e = new IOException();  // Compilation error
}
```

---

## Common Interview Traps

### Trap 1: "Finally always executes"

Not if System.exit() is called, JVM crashes, or thread is killed.

### Trap 2: "RuntimeException must be declared"

No, unchecked exceptions don't need to be declared or caught.

### Trap 3: "Catch Exception catches everything"

No, it doesn't catch Error. Only Throwable catches everything.

### Trap 4: "Return in finally is good practice"

No! Return in finally masks the original return value.

### Trap 5: "Custom exception should always extend Exception"

Extend RuntimeException for unchecked, Exception for checked based on use case.

---

## Key Takeaways

1. **Checked vs Unchecked**: Know when to use each
2. **try-with-resources**: Prefer over try-finally for resource management
3. **Specific exceptions**: Catch and throw specific, not generic
4. **Exception chaining**: Preserve original cause
5. **Finally**: Use for cleanup, never return from it
6. **Don't swallow**: Always handle or propagate meaningfully
7. **Custom exceptions**: Create when domain-specific handling needed
8. **Fail fast**: Validate early, throw meaningful exceptions
9. **Documentation**: Document exceptions in Javadoc
10. **Logging**: Log with appropriate levels and context
