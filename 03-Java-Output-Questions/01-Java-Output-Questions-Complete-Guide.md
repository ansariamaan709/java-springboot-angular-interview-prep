# Java Output Questions - Complete Interview Guide

## Table of Contents

1. [String Output Questions](#string-output-questions)
2. [Inheritance & Polymorphism Questions](#inheritance--polymorphism-questions)
3. [Exception Handling Questions](#exception-handling-questions)
4. [Operators & Expressions Questions](#operators--expressions-questions)
5. [Collections Output Questions](#collections-output-questions)
6. [Multithreading Output Questions](#multithreading-output-questions)
7. [Static & Initialization Questions](#static--initialization-questions)
8. [Tricky Interview Questions](#tricky-interview-questions)

---

## String Output Questions

### Question 1: String Pool & == vs equals()

```java
public class StringPoolQuestion {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = "Hello";
        String s3 = new String("Hello");
        String s4 = new String("Hello");

        System.out.println(s1 == s2);        // Line 1
        System.out.println(s1 == s3);        // Line 2
        System.out.println(s3 == s4);        // Line 3
        System.out.println(s1.equals(s3));   // Line 4
        System.out.println(s3.equals(s4));   // Line 5
    }
}
```

<details>
<summary>Click to see Output</summary>

```
true    // Line 1: Both point to same string pool object
false   // Line 2: s3 is new object on heap
false   // Line 3: Different heap objects
true    // Line 4: Same content
true    // Line 5: Same content
```

**Explanation:**

- `s1` and `s2` are string literals → stored in String Pool → same reference
- `s3` and `s4` use `new` → creates new objects on heap
- `==` compares references, `equals()` compares content

</details>

---

### Question 2: String intern()

```java
public class StringInternQuestion {
    public static void main(String[] args) {
        String s1 = "Java";
        String s2 = new String("Java");
        String s3 = s2.intern();

        System.out.println(s1 == s2);      // Line 1
        System.out.println(s1 == s3);      // Line 2
        System.out.println(s2 == s3);      // Line 3
    }
}
```

<details>
<summary>Click to see Output</summary>

```
false   // Line 1: s2 is on heap, s1 is in pool
true    // Line 2: intern() returns pool reference = s1
false   // Line 3: s2 is heap, s3 is pool
```

**Explanation:**

- `intern()` returns the String Pool reference for the string
- Since "Java" already exists in pool, `s3` points to same object as `s1`

</details>

---

### Question 3: String Concatenation

```java
public class StringConcatQuestion {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = "World";
        String s3 = "HelloWorld";
        String s4 = "Hello" + "World";
        String s5 = s1 + "World";
        String s6 = s1 + s2;

        System.out.println(s3 == s4);      // Line 1
        System.out.println(s3 == s5);      // Line 2
        System.out.println(s3 == s6);      // Line 3
        System.out.println(s5 == s6);      // Line 4
    }
}
```

<details>
<summary>Click to see Output</summary>

```
true    // Line 1: Compile-time constant folding
false   // Line 2: s5 created at runtime (s1 is variable)
false   // Line 3: s6 created at runtime
false   // Line 4: Both are different runtime objects
```

**Explanation:**

- `"Hello" + "World"` is evaluated at compile time → goes to pool
- When variable is involved in concatenation → runtime object on heap
- Compiler optimization only works with literals/final constants

</details>

---

### Question 4: String with final

```java
public class StringFinalQuestion {
    public static void main(String[] args) {
        final String s1 = "Hello";
        final String s2 = "World";
        String s3 = "HelloWorld";
        String s4 = s1 + s2;

        System.out.println(s3 == s4);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
true
```

**Explanation:**

- `s1` and `s2` are `final` → compile-time constants
- `s1 + s2` is evaluated at compile time as `"HelloWorld"`
- Both `s3` and `s4` point to same string pool object

</details>

---

### Question 5: StringBuilder vs StringBuffer

```java
public class StringBuilderQuestion {
    public static void main(String[] args) {
        StringBuilder sb1 = new StringBuilder("Hello");
        StringBuilder sb2 = new StringBuilder("Hello");

        System.out.println(sb1 == sb2);                    // Line 1
        System.out.println(sb1.equals(sb2));               // Line 2
        System.out.println(sb1.toString().equals(sb2.toString())); // Line 3
    }
}
```

<details>
<summary>Click to see Output</summary>

```
false   // Line 1: Different objects
false   // Line 2: StringBuilder doesn't override equals()!
true    // Line 3: String's equals() compares content
```

**Explanation:**

- `StringBuilder` does NOT override `equals()` from Object
- `equals()` uses default Object behavior (reference comparison)
- Must convert to String for content comparison

</details>

---

## Inheritance & Polymorphism Questions

### Question 6: Method Overriding with Return Types

```java
class Parent {
    public Number getValue() {
        return 10;
    }
}

class Child extends Parent {
    @Override
    public Integer getValue() {  // Covariant return type
        return 20;
    }
}

public class CovariantQuestion {
    public static void main(String[] args) {
        Parent p = new Child();
        System.out.println(p.getValue());
    }
}
```

<details>
<summary>Click to see Output</summary>

```
20
```

**Explanation:**

- Covariant return type allows overriding with more specific return type
- `Integer` is subtype of `Number` → valid override
- Runtime polymorphism → Child's method is called

</details>

---

### Question 7: Method Hiding (Static Methods)

```java
class Animal {
    public static void eat() {
        System.out.println("Animal is eating");
    }

    public void sleep() {
        System.out.println("Animal is sleeping");
    }
}

class Dog extends Animal {
    public static void eat() {
        System.out.println("Dog is eating");
    }

    @Override
    public void sleep() {
        System.out.println("Dog is sleeping");
    }
}

public class StaticHidingQuestion {
    public static void main(String[] args) {
        Animal a = new Dog();
        a.eat();     // Line 1
        a.sleep();   // Line 2

        Dog d = new Dog();
        d.eat();     // Line 3
        d.sleep();   // Line 4
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Animal is eating    // Line 1: Static - resolved at compile time
Dog is sleeping     // Line 2: Instance - runtime polymorphism
Dog is eating       // Line 3: Called on Dog reference
Dog is sleeping     // Line 4: Called on Dog reference
```

**Explanation:**

- Static methods are resolved at **compile time** based on reference type
- Instance methods are resolved at **runtime** based on object type
- Static methods can be "hidden" but not "overridden"

</details>

---

### Question 8: Field Hiding

```java
class Base {
    public int x = 10;

    public int getX() {
        return x;
    }
}

class Derived extends Base {
    public int x = 20;

    @Override
    public int getX() {
        return x;
    }
}

public class FieldHidingQuestion {
    public static void main(String[] args) {
        Base b = new Derived();
        System.out.println(b.x);        // Line 1
        System.out.println(b.getX());   // Line 2

        Derived d = new Derived();
        System.out.println(d.x);        // Line 3
        System.out.println(d.getX());   // Line 4
    }
}
```

<details>
<summary>Click to see Output</summary>

```
10    // Line 1: Field resolved by reference type
20    // Line 2: Method - runtime polymorphism
20    // Line 3: Derived's field
20    // Line 4: Derived's method
```

**Explanation:**

- Fields are NOT polymorphic - resolved at compile time by reference type
- Methods ARE polymorphic - resolved at runtime by object type
- `b.x` uses Base's field, `b.getX()` uses Derived's method

</details>

---

### Question 9: Constructor Chaining

```java
class A {
    public A() {
        System.out.println("A constructor");
    }
}

class B extends A {
    public B() {
        System.out.println("B constructor");
    }
}

class C extends B {
    public C() {
        System.out.println("C constructor");
    }
}

public class ConstructorChainQuestion {
    public static void main(String[] args) {
        C c = new C();
    }
}
```

<details>
<summary>Click to see Output</summary>

```
A constructor
B constructor
C constructor
```

**Explanation:**

- Constructor chaining: parent constructor called before child
- Implicit `super()` at the beginning of each constructor
- Order: A → B → C (top-down in hierarchy)

</details>

---

### Question 10: Overriding with Exceptions

```java
class Parent {
    public void method() throws Exception {
        System.out.println("Parent method");
    }
}

class Child extends Parent {
    @Override
    public void method() throws RuntimeException {  // More specific exception
        System.out.println("Child method");
    }
}

public class ExceptionOverrideQuestion {
    public static void main(String[] args) {
        Parent p = new Child();
        try {
            p.method();
        } catch (Exception e) {
            System.out.println("Caught exception");
        }
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Child method
```

**Explanation:**

- Override can throw same, narrower, or NO checked exception
- `RuntimeException` is narrower than `Exception`
- Child's method is called (polymorphism)

</details>

---

## Exception Handling Questions

### Question 11: try-catch-finally Order

```java
public class TryCatchFinallyQuestion {
    public static void main(String[] args) {
        System.out.println(getValue());
    }

    public static int getValue() {
        try {
            System.out.println("try block");
            return 1;
        } catch (Exception e) {
            System.out.println("catch block");
            return 2;
        } finally {
            System.out.println("finally block");
        }
    }
}
```

<details>
<summary>Click to see Output</summary>

```
try block
finally block
1
```

**Explanation:**

- `try` executes, prepares to return 1
- `finally` ALWAYS executes before method returns
- Return value was already determined as 1

</details>

---

### Question 12: Return in finally

```java
public class FinallyReturnQuestion {
    public static void main(String[] args) {
        System.out.println(getValue());
    }

    public static int getValue() {
        try {
            return 1;
        } finally {
            return 2;  // WARNING: finally shouldn't have return
        }
    }
}
```

<details>
<summary>Click to see Output</summary>

```
2
```

**Explanation:**

- Return in `finally` OVERRIDES the return in `try`
- This is bad practice - should be avoided
- `finally` return masks any exception or previous return value

</details>

---

### Question 13: Exception in finally

```java
public class ExceptionInFinallyQuestion {
    public static void main(String[] args) {
        try {
            method();
        } catch (Exception e) {
            System.out.println("Main caught: " + e.getMessage());
        }
    }

    public static void method() {
        try {
            throw new RuntimeException("From try");
        } finally {
            throw new RuntimeException("From finally");
        }
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Main caught: From finally
```

**Explanation:**

- Exception in `finally` SUPPRESSES the exception from `try`
- Original exception is lost (bad practice)
- Only "From finally" exception propagates

</details>

---

### Question 14: Multiple Exceptions

```java
public class MultipleExceptionQuestion {
    public static void main(String[] args) {
        try {
            try {
                throw new RuntimeException("Inner");
            } catch (RuntimeException e) {
                System.out.println("Inner catch: " + e.getMessage());
                throw new RuntimeException("Re-thrown");
            } finally {
                System.out.println("Inner finally");
            }
        } catch (RuntimeException e) {
            System.out.println("Outer catch: " + e.getMessage());
        } finally {
            System.out.println("Outer finally");
        }
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Inner catch: Inner
Inner finally
Outer catch: Re-thrown
Outer finally
```

**Explanation:**

1. Inner try throws "Inner" exception
2. Inner catch handles it and re-throws "Re-thrown"
3. Inner finally executes
4. Outer catch catches "Re-thrown"
5. Outer finally executes

</details>

---

### Question 15: Catch Order

```java
public class CatchOrderQuestion {
    public static void main(String[] args) {
        try {
            int[] arr = new int[5];
            arr[10] = 50;  // ArrayIndexOutOfBoundsException
        } catch (RuntimeException e) {
            System.out.println("RuntimeException caught");
        } catch (Exception e) {
            System.out.println("Exception caught");
        }
    }
}
```

<details>
<summary>Click to see Output</summary>

```
RuntimeException caught
```

**Explanation:**

- `ArrayIndexOutOfBoundsException` is a `RuntimeException`
- First matching catch block is executed
- More specific exceptions must come before generic ones

</details>

---

## Operators & Expressions Questions

### Question 16: Post vs Pre Increment

```java
public class IncrementQuestion {
    public static void main(String[] args) {
        int i = 5;
        System.out.println(i++);   // Line 1
        System.out.println(++i);   // Line 2
        System.out.println(i++);   // Line 3
        System.out.println(i);     // Line 4
    }
}
```

<details>
<summary>Click to see Output</summary>

```
5    // Line 1: Post-increment, use then increment (i becomes 6)
7    // Line 2: Pre-increment, increment then use (i becomes 7)
7    // Line 3: Post-increment, use then increment (i becomes 8)
8    // Line 4: Current value
```

**Explanation:**

- `i++` (post): use current value, then increment
- `++i` (pre): increment first, then use new value

</details>

---

### Question 17: Tricky Increment Expression

```java
public class TrickyIncrementQuestion {
    public static void main(String[] args) {
        int x = 5;
        x = x++;  // What happens here?
        System.out.println(x);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
5
```

**Explanation:**

1. `x++` returns 5 (current value), then x becomes 6
2. Assignment `x = 5` overwrites x with the returned value
3. Final value is 5

Step by step:

- Save current x (5) for return
- Increment x to 6
- Assign saved value (5) back to x
- x is now 5

</details>

---

### Question 18: Short-Circuit Evaluation

```java
public class ShortCircuitQuestion {
    public static void main(String[] args) {
        int a = 5, b = 10;

        if (a > 3 || ++b > 10) {
            System.out.println("a = " + a + ", b = " + b);
        }

        if (a < 3 && ++b > 10) {
            System.out.println("Won't print");
        }
        System.out.println("After: a = " + a + ", b = " + b);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
a = 5, b = 10
After: a = 5, b = 10
```

**Explanation:**

- `||` short-circuits: if first is true, second not evaluated
- `&&` short-circuits: if first is false, second not evaluated
- `++b` is NEVER evaluated because of short-circuit

</details>

---

### Question 19: Integer Caching

```java
public class IntegerCacheQuestion {
    public static void main(String[] args) {
        Integer a = 100;
        Integer b = 100;
        Integer c = 200;
        Integer d = 200;

        System.out.println(a == b);    // Line 1
        System.out.println(c == d);    // Line 2
        System.out.println(a.equals(b)); // Line 3
        System.out.println(c.equals(d)); // Line 4
    }
}
```

<details>
<summary>Click to see Output</summary>

```
true    // Line 1: -128 to 127 cached
false   // Line 2: 200 outside cache range
true    // Line 3: Same value
true    // Line 4: Same value
```

**Explanation:**

- Java caches Integer objects from -128 to 127
- Within range: `==` returns true (same cached object)
- Outside range: new objects created
- Always use `equals()` for comparing wrapper objects

</details>

---

### Question 20: Ternary Operator Type

```java
public class TernaryTypeQuestion {
    public static void main(String[] args) {
        int x = 5;
        Object result = (x > 3) ? 1 : 2.0;
        System.out.println(result);
        System.out.println(result.getClass().getName());
    }
}
```

<details>
<summary>Click to see Output</summary>

```
1.0
java.lang.Double
```

**Explanation:**

- Ternary operator requires both branches to have compatible types
- `int` and `double` → result type is `double`
- 1 is promoted to 1.0
- Autoboxed to `Double` when assigned to `Object`

</details>

---

## Collections Output Questions

### Question 21: ArrayList vs LinkedList

```java
import java.util.*;

public class ListRemovalQuestion {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 2, 4));
        list.remove(2);           // Remove by index
        System.out.println(list);

        list.remove(Integer.valueOf(2));  // Remove by value
        System.out.println(list);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
[1, 2, 2, 4]
[1, 2, 4]
```

**Explanation:**

- `remove(2)` removes element at index 2 (value 3)
- `remove(Integer.valueOf(2))` removes first occurrence of value 2
- Ambiguity with Integer lists - index vs value

</details>

---

### Question 22: HashSet Uniqueness

```java
import java.util.*;

class Person {
    String name;

    Person(String name) {
        this.name = name;
    }
}

public class HashSetQuestion {
    public static void main(String[] args) {
        Set<Person> set = new HashSet<>();
        set.add(new Person("John"));
        set.add(new Person("John"));
        System.out.println(set.size());
    }
}
```

<details>
<summary>Click to see Output</summary>

```
2
```

**Explanation:**

- `Person` class doesn't override `hashCode()` and `equals()`
- Uses default Object implementation (reference comparison)
- Two different objects = two different entries in HashSet

</details>

---

### Question 23: ConcurrentModificationException

```java
import java.util.*;

public class ConcurrentModQuestion {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));

        for (String s : list) {
            if (s.equals("B")) {
                list.remove(s);
            }
        }
        System.out.println(list);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Exception in thread "main" java.util.ConcurrentModificationException
```

**Explanation:**

- For-each uses Iterator internally
- Modifying collection during iteration causes CME
- Use Iterator.remove() or CopyOnWriteArrayList

**Fixed version:**

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("B")) {
        it.remove();  // Safe removal
    }
}
```

</details>

---

### Question 24: HashMap Null Keys

```java
import java.util.*;

public class HashMapNullQuestion {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put(null, 1);
        map.put(null, 2);
        map.put("key", null);

        System.out.println(map.size());
        System.out.println(map.get(null));
        System.out.println(map.get("key"));
    }
}
```

<details>
<summary>Click to see Output</summary>

```
2
2
null
```

**Explanation:**

- HashMap allows ONE null key (overwrites previous)
- HashMap allows multiple null values
- First `null` key entry is overwritten by second

</details>

---

### Question 25: TreeSet Ordering

```java
import java.util.*;

public class TreeSetQuestion {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        set.add(5);
        set.add(2);
        set.add(8);
        set.add(1);
        set.add(3);

        System.out.println(set);
        System.out.println(((TreeSet<Integer>) set).first());
        System.out.println(((TreeSet<Integer>) set).last());
    }
}
```

<details>
<summary>Click to see Output</summary>

```
[1, 2, 3, 5, 8]
1
8
```

**Explanation:**

- TreeSet maintains natural ordering (sorted)
- `first()` returns smallest element
- `last()` returns largest element

</details>

---

## Multithreading Output Questions

### Question 26: Basic Thread Output

```java
public class ThreadOrderQuestion {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            for (int i = 1; i <= 3; i++) {
                System.out.println("Thread: " + i);
            }
        });

        t.start();

        for (int i = 1; i <= 3; i++) {
            System.out.println("Main: " + i);
        }
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Output is NON-DETERMINISTIC!
Possible output:
Main: 1
Main: 2
Thread: 1
Main: 3
Thread: 2
Thread: 3

Or any other interleaving!
```

**Explanation:**

- Thread execution order is not guaranteed
- OS scheduler decides which thread runs when
- Both threads compete for CPU time

</details>

---

### Question 27: start() vs run()

```java
public class StartVsRunQuestion {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            System.out.println("Running in: " + Thread.currentThread().getName());
        });

        System.out.println("Before: " + Thread.currentThread().getName());
        t.run();   // Direct call - NOT starting thread!
        System.out.println("After run()");
        t.start(); // Actual thread start
        System.out.println("After start()");
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Before: main
Running in: main
After run()
After start()
Running in: Thread-0
```

**Explanation:**

- `run()` - Executes in CURRENT thread (synchronous)
- `start()` - Creates NEW thread (asynchronous)
- `t.run()` runs in main thread
- `t.start()` runs in Thread-0

</details>

---

### Question 28: synchronized Block

```java
public class SynchronizedQuestion {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public static void main(String[] args) throws InterruptedException {
        SynchronizedQuestion obj = new SynchronizedQuestion();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                obj.increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                obj.increment();
            }
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Count: " + obj.count);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Count: 2000
```

**Explanation:**

- `synchronized` ensures thread-safe increment
- Without synchronized, count would be < 2000 (race condition)
- `join()` waits for threads to complete

</details>

---

### Question 29: Volatile Variable

```java
public class VolatileQuestion {
    private static volatile boolean flag = false;

    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            while (!flag) {
                // Busy waiting
            }
            System.out.println("Flag changed, thread exiting");
        });

        t.start();
        Thread.sleep(100);
        flag = true;
        System.out.println("Flag set to true");
        t.join();
        System.out.println("Main done");
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Flag set to true
Flag changed, thread exiting
Main done
```

**Explanation:**

- `volatile` ensures visibility across threads
- Without `volatile`, thread might cache `flag` and loop forever
- Change is immediately visible to all threads

</details>

---

### Question 30: Thread States

```java
public class ThreadStateQuestion {
    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();

        Thread t = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("State inside sync: " +
                        Thread.currentThread().getState());
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        System.out.println("Before start: " + t.getState());
        t.start();
        Thread.sleep(100);
        System.out.println("After wait called: " + t.getState());

        synchronized (lock) {
            lock.notify();
        }
        t.join();
        System.out.println("After join: " + t.getState());
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Before start: NEW
State inside sync: RUNNABLE
After wait called: WAITING
After join: TERMINATED
```

**Explanation:**

- NEW: Thread created but not started
- RUNNABLE: Thread is executing
- WAITING: Thread is waiting (after `wait()`)
- TERMINATED: Thread has finished

</details>

---

## Static & Initialization Questions

### Question 31: Static Block Execution Order

```java
class Parent {
    static {
        System.out.println("Parent static block");
    }

    {
        System.out.println("Parent instance block");
    }

    public Parent() {
        System.out.println("Parent constructor");
    }
}

class Child extends Parent {
    static {
        System.out.println("Child static block");
    }

    {
        System.out.println("Child instance block");
    }

    public Child() {
        System.out.println("Child constructor");
    }
}

public class InitOrderQuestion {
    public static void main(String[] args) {
        System.out.println("Creating first Child");
        new Child();
        System.out.println("\nCreating second Child");
        new Child();
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Parent static block
Child static block
Creating first Child
Parent instance block
Parent constructor
Child instance block
Child constructor

Creating second Child
Parent instance block
Parent constructor
Child instance block
Child constructor
```

**Explanation:**

1. Static blocks execute ONCE when class is loaded (parent first)
2. For each object:
   - Parent instance block
   - Parent constructor
   - Child instance block
   - Child constructor

</details>

---

### Question 32: Static Variable Initialization

```java
public class StaticInitQuestion {
    static int x = getValue();
    static int y = 10;

    static int getValue() {
        return y;  // y not initialized yet!
    }

    public static void main(String[] args) {
        System.out.println("x = " + x);
        System.out.println("y = " + y);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
x = 0
y = 10
```

**Explanation:**

- Static variables initialized in declaration order
- When `getValue()` is called, `y` has default value (0)
- Later `y` is set to 10

</details>

---

### Question 33: Variable Shadowing

```java
public class ShadowingQuestion {
    static int x = 10;

    public static void main(String[] args) {
        int x = 20;  // Local shadows static
        System.out.println(x);
        System.out.println(ShadowingQuestion.x);

        method(30);
    }

    static void method(int x) {  // Parameter shadows static
        System.out.println(x);
        System.out.println(ShadowingQuestion.x);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
20
10
30
10
```

**Explanation:**

- Local variables shadow class variables
- Use class name to access shadowed static variable

</details>

---

### Question 34: Instance Initializer Execution

```java
public class InitializerQuestion {
    int x = 10;

    {
        System.out.println("Instance block 1: x = " + x);
        x = 20;
    }

    int y = x + 5;

    {
        System.out.println("Instance block 2: y = " + y);
    }

    public InitializerQuestion() {
        System.out.println("Constructor: x = " + x + ", y = " + y);
    }

    public static void main(String[] args) {
        new InitializerQuestion();
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Instance block 1: x = 10
Instance block 2: y = 25
Constructor: x = 20, y = 25
```

**Explanation:**

- Initialization order: field initializers and instance blocks in declaration order
- Then constructor executes
- y = 20 + 5 = 25 (uses x after first block modifies it)

</details>

---

## Tricky Interview Questions

### Question 35: Overloading vs Overriding

```java
class Base {
    public void method(int x) {
        System.out.println("Base int: " + x);
    }
}

class Derived extends Base {
    public void method(Integer x) {
        System.out.println("Derived Integer: " + x);
    }
}

public class OverloadQuestion {
    public static void main(String[] args) {
        Base b = new Derived();
        b.method(10);

        Derived d = new Derived();
        d.method(10);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Base int: 10
Base int: 10
```

**Explanation:**

- `method(Integer)` is OVERLOADING, not overriding
- `int` parameter prefers `int` version (exact match)
- Compiler chooses method based on reference type
- For b: only `method(int)` is visible
- For d: `int` version still preferred over `Integer` version

</details>

---

### Question 36: Varargs and Overloading

```java
public class VarargsQuestion {
    public static void method(int... args) {
        System.out.println("Varargs: " + args.length);
    }

    public static void method(int a, int b) {
        System.out.println("Two args: " + a + ", " + b);
    }

    public static void main(String[] args) {
        method(1, 2);
        method(1, 2, 3);
        method();
    }
}
```

<details>
<summary>Click to see Output</summary>

```
Two args: 1, 2
Varargs: 3
Varargs: 0
```

**Explanation:**

- Exact match is preferred over varargs
- `method(1, 2)` matches the two-arg version
- Varargs is used when no exact match exists

</details>

---

### Question 37: Autoboxing and Method Resolution

```java
public class AutoboxingQuestion {
    public static void method(Object o) {
        System.out.println("Object");
    }

    public static void method(Integer i) {
        System.out.println("Integer");
    }

    public static void method(long l) {
        System.out.println("long");
    }

    public static void main(String[] args) {
        int x = 10;
        method(x);
    }
}
```

<details>
<summary>Click to see Output</summary>

```
long
```

**Explanation:**
Method resolution priority:

1. Exact match (int) - Not found
2. Widening (int → long) - **Found!**
3. Autoboxing (int → Integer)
4. Varargs

Widening is preferred over autoboxing.

</details>

---

### Question 38: Lambda and Effectively Final

```java
public class LambdaFinalQuestion {
    public static void main(String[] args) {
        int x = 10;
        // x = 20;  // Uncommenting this would cause compilation error

        Runnable r = () -> {
            System.out.println("x = " + x);
            // x = 30;  // Cannot modify - x must be effectively final
        };

        r.run();
    }
}
```

<details>
<summary>Click to see Output</summary>

```
x = 10
```

**Explanation:**

- Variables used in lambda must be effectively final
- Cannot modify `x` after initialization
- If `x` is modified, lambda cannot capture it

</details>

---

### Question 39: Method Reference Binding

```java
import java.util.function.*;

public class MethodRefQuestion {
    public String value = "Hello";

    public String getValue() {
        return value;
    }

    public static void main(String[] args) {
        MethodRefQuestion obj1 = new MethodRefQuestion();
        Supplier<String> supplier = obj1::getValue;

        obj1.value = "World";

        System.out.println(supplier.get());
    }
}
```

<details>
<summary>Click to see Output</summary>

```
World
```

**Explanation:**

- Method reference captures the object reference, not the value
- When `supplier.get()` is called, it uses current value
- `obj1.value` was changed before calling `get()`

</details>

---

### Question 40: equals() Contract

```java
public class EqualsQuestion {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = "HELLO".toLowerCase();

        System.out.println(s1 == s2);
        System.out.println(s1.equals(s2));
        System.out.println(s1.hashCode() == s2.hashCode());
    }
}
```

<details>
<summary>Click to see Output</summary>

```
false
true
true
```

**Explanation:**

- `toLowerCase()` creates new String object (not in pool)
- `==` compares references → false
- `equals()` compares content → true
- Equal objects must have same hashCode → true

</details>

---

## Summary: Key Traps to Remember

| Topic                 | Trap                 | Key Point                                                  |
| --------------------- | -------------------- | ---------------------------------------------------------- |
| **String Pool**       | `==` vs `equals()`   | Literals share pool, `new` creates heap object             |
| **String concat**     | Variable involvement | Only literals get compile-time optimization                |
| **Static methods**    | "Overriding"         | Static methods hide, don't override                        |
| **Fields**            | Polymorphism         | Fields resolved by reference type, not object type         |
| **finally**           | Return override      | finally's return overrides try's return                    |
| **Increment**         | `x = x++`            | Returns old value, then increments, then assigns           |
| **Integer cache**     | -128 to 127          | Only this range shares objects                             |
| **Collection remove** | Index vs value       | Integer lists: `remove(2)` vs `remove(Integer.valueOf(2))` |
| **HashSet**           | equals/hashCode      | Must override both for custom objects                      |
| **Thread start**      | start() vs run()     | Only start() creates new thread                            |
| **Static blocks**     | Execution order      | Once per class, parent before child                        |
| **Autoboxing**        | Method resolution    | Widening preferred over boxing                             |
