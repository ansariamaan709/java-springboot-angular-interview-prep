# Java Memory Management & Garbage Collection - Complete Interview Guide

## Table of Contents

1. [JVM Memory Architecture](#jvm-memory-architecture)
2. [Heap Memory Structure](#heap-memory-structure)
3. [Garbage Collection Fundamentals](#garbage-collection-fundamentals)
4. [Types of Garbage Collectors](#types-of-garbage-collectors)
5. [GC Algorithms](#gc-algorithms)
6. [Memory Leaks in Java](#memory-leaks-in-java)
7. [JVM Tuning Parameters](#jvm-tuning-parameters)
8. [Monitoring & Profiling](#monitoring--profiling)
9. [Interview Questions](#interview-questions)

---

## JVM Memory Architecture

### Memory Area Overview

```
┌────────────────────────────────────────────────────────────────────────┐
│                           JVM MEMORY STRUCTURE                          │
├────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │                         HEAP MEMORY                           │      │
│  │  (Shared across all threads - Object storage)                 │      │
│  │  ┌─────────────────┐  ┌──────────────────────────────────┐   │      │
│  │  │  Young Gen      │  │         Old Generation            │   │      │
│  │  │  ┌───┬───┬───┐  │  │  (Tenured/Long-lived objects)     │   │      │
│  │  │  │ E │S0 │S1 │  │  │                                   │   │      │
│  │  │  └───┴───┴───┘  │  └──────────────────────────────────┘   │      │
│  │  └─────────────────┘                                          │      │
│  └──────────────────────────────────────────────────────────────┘      │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │                       METASPACE (Java 8+)                     │      │
│  │  (Class metadata, method bytecode, constant pool)             │      │
│  │  Uses native memory - auto-grows                              │      │
│  └──────────────────────────────────────────────────────────────┘      │
│                                                                         │
│  ┌────────────────────┐  ┌────────────────────┐                        │
│  │   Stack (per thread)│  │  PC Register        │                        │
│  │   - Local variables │  │  (per thread)       │                        │
│  │   - Method calls    │  │  - Current          │                        │
│  │   - Operand stack   │  │    instruction      │                        │
│  └────────────────────┘  └────────────────────┘                        │
│                                                                         │
│  ┌────────────────────────────────────────────────────────────────┐    │
│  │                    Native Method Stack                          │    │
│  │  (For native method execution - JNI calls)                      │    │
│  └────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└────────────────────────────────────────────────────────────────────────┘
```

### Memory Areas Explained

#### 1. Heap Memory

```java
// Objects created on the heap
public class HeapExample {
    public static void main(String[] args) {
        // All objects are allocated on heap
        String str = new String("Hello");      // Heap allocation
        Employee emp = new Employee("John");    // Heap allocation
        int[] array = new int[100];             // Heap allocation

        // Even wrapper objects
        Integer num = Integer.valueOf(1000);    // Heap allocation

        // String literal pool (special area in heap)
        String literal = "Hello World";         // String pool
    }
}
```

#### 2. Stack Memory

```java
// Stack stores local variables and method call frames
public class StackExample {
    public static void main(String[] args) {
        int a = 10;          // Primitive on stack
        int b = 20;          // Primitive on stack
        int sum = add(a, b); // New stack frame created
    }

    public static int add(int x, int y) {
        // x, y are on this method's stack frame
        int result = x + y;  // result on stack
        return result;       // Stack frame removed after return
    }
}

/*
Stack visualization during add() execution:

│ add() frame      │  ← Top of stack
│  - x = 10        │
│  - y = 20        │
│  - result = 30   │
├──────────────────┤
│ main() frame     │
│  - args          │
│  - a = 10        │
│  - b = 20        │
│  - sum           │
└──────────────────┘
*/
```

#### 3. Metaspace (Java 8+)

```java
// Metaspace stores class metadata
public class MetaspaceExample {
    // Class definition stored in Metaspace
    // Method bytecode stored in Metaspace
    // Constant pool stored in Metaspace

    private static final String CONSTANT = "Value"; // Constant pool

    public void method() {
        // Method bytecode in Metaspace
    }
}

// Before Java 8: PermGen (fixed size, prone to OOM)
// Java 8+: Metaspace (uses native memory, auto-grows)
```

### Stack vs Heap Comparison

| Feature               | Stack                  | Heap               |
| --------------------- | ---------------------- | ------------------ |
| **Storage**           | Primitives, references | Objects, arrays    |
| **Access**            | Fast (LIFO)            | Slower             |
| **Size**              | Limited (per thread)   | Large (shared)     |
| **Memory Management** | Automatic (on return)  | Garbage Collected  |
| **Thread Safety**     | Thread-local           | Shared, needs sync |
| **Errors**            | StackOverflowError     | OutOfMemoryError   |

---

## Heap Memory Structure

### Generational Heap Layout

```
┌──────────────────────────────────────────────────────────────────────┐
│                          HEAP MEMORY                                  │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │              YOUNG GENERATION (~1/3 of heap)                    │  │
│  │                                                                 │  │
│  │  ┌────────────────┐  ┌─────────────┐  ┌─────────────┐          │  │
│  │  │    EDEN        │  │ SURVIVOR    │  │ SURVIVOR    │          │  │
│  │  │    (~80%)      │  │    S0       │  │    S1       │          │  │
│  │  │                │  │   (~10%)    │  │   (~10%)    │          │  │
│  │  │  New objects   │  │  (From)     │  │   (To)      │          │  │
│  │  │  allocated     │  │             │  │             │          │  │
│  │  │  here          │  │             │  │             │          │  │
│  │  └────────────────┘  └─────────────┘  └─────────────┘          │  │
│  │                                                                 │  │
│  │  Minor GC occurs here (fast, frequent)                         │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                       │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │              OLD GENERATION (~2/3 of heap)                      │  │
│  │                     (TENURED SPACE)                             │  │
│  │                                                                 │  │
│  │    Objects that survived multiple GC cycles live here           │  │
│  │    Major GC / Full GC occurs here (slow, infrequent)           │  │
│  │                                                                 │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### Object Lifecycle in Heap

```java
public class ObjectLifecycle {
    public static void main(String[] args) {
        // Step 1: Object created in EDEN space
        Object obj = new Object();

        // Step 2: After Minor GC, if still referenced,
        //         moves to Survivor space (S0)

        // Step 3: After next Minor GC, if still alive,
        //         moves to other Survivor (S1)

        // Step 4: After multiple GC cycles (threshold ~15),
        //         promoted to Old Generation

        // Step 5: Eventually, during Major GC,
        //         reclaimed if no longer referenced
    }
}
```

### Object Allocation Flow

```
New Object Request
       │
       ▼
┌──────────────┐
│ Try EDEN     │
│ Allocation   │
└──────────────┘
       │
       ├─── Success → Object created in Eden
       │
       ▼ (Eden full)
┌──────────────┐
│ Minor GC     │
│ triggered    │
└──────────────┘
       │
       ▼
┌──────────────────────────────┐
│ Live objects → Survivor      │
│ Dead objects → Reclaimed     │
│ Age++ for surviving objects  │
└──────────────────────────────┘
       │
       ▼ (Age > threshold)
┌──────────────┐
│ Promote to   │
│ Old Gen      │
└──────────────┘
       │
       ▼ (Old Gen full)
┌──────────────┐
│ Major GC /   │
│ Full GC      │
└──────────────┘
```

---

## Garbage Collection Fundamentals

### What is Garbage Collection?

**Garbage Collection (GC)** is automatic memory management that reclaims memory occupied by objects that are no longer referenced.

### How GC Identifies Garbage

```java
public class GCEligibility {
    public static void main(String[] args) {
        // Case 1: Nullifying reference
        Object obj1 = new Object();
        obj1 = null;  // Original object eligible for GC

        // Case 2: Reassigning reference
        Object obj2 = new Object();  // Object A
        obj2 = new Object();         // Object A eligible for GC

        // Case 3: Objects inside method
        createObject();  // Object created inside is eligible after method returns

        // Case 4: Island of isolation
        IslandExample a = new IslandExample();
        IslandExample b = new IslandExample();
        a.ref = b;
        b.ref = a;
        a = null;
        b = null;
        // Both objects are eligible - island of isolation
    }

    static void createObject() {
        Object localObj = new Object();
        // localObj eligible after method returns
    }
}

class IslandExample {
    IslandExample ref;
}
```

### GC Roots

Objects reachable from GC roots are NOT garbage collected.

```
                    GC ROOTS
                       │
    ┌─────────┬────────┼────────┬─────────┐
    │         │        │        │         │
    ▼         ▼        ▼        ▼         ▼
┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
│Static │ │Local  │ │Active │ │JNI    │ │System │
│Fields │ │Vars   │ │Threads│ │refs   │ │Class  │
└───────┘ └───────┘ └───────┘ └───────┘ └───────┘
```

**GC Roots include:**

1. **Local variables** in active threads
2. **Static fields** of loaded classes
3. **Active threads** themselves
4. **JNI references** (native code)
5. **Class loaders** and classes they loaded

### Mark and Sweep Algorithm

```
Phase 1: MARK
─────────────
Starting from GC roots, mark all reachable objects

         GC Root
            │
            ▼
         ┌─────┐
         │  A  │ ← Marked
         └─────┘
          /   \
         ▼     ▼
     ┌─────┐ ┌─────┐
     │  B  │ │  C  │ ← Both Marked
     └─────┘ └─────┘
                │
                ▼
            ┌─────┐
            │  D  │ ← Marked
            └─────┘

     ┌─────┐ ┌─────┐
     │  X  │ │  Y  │ ← NOT Marked (unreachable)
     └─────┘ └─────┘


Phase 2: SWEEP
──────────────
Reclaim memory from unmarked objects

Before:  │ A │ X │ B │ Y │ C │   │ D │

After:   │ A │   │ B │   │ C │   │ D │
              ↑       ↑
         (Free space added to free list)


Phase 3: COMPACT (optional)
───────────────────────────
Move live objects together to reduce fragmentation

Before:  │ A │   │ B │   │ C │   │ D │

After:   │ A │ B │ C │ D │               │
                         ↑───────────────↑
                         (Contiguous free space)
```

### finalize() Method

```java
public class FinalizeExample {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("Object being garbage collected");
        super.finalize();
    }

    public static void main(String[] args) {
        FinalizeExample obj = new FinalizeExample();
        obj = null;

        // Request GC (not guaranteed)
        System.gc();

        // Give GC time to run
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

/*
WARNING: finalize() is DEPRECATED since Java 9!

Problems with finalize():
1. No guarantee when or if it will be called
2. Slows down GC (objects need two GC cycles)
3. Can resurrect objects (bad practice)
4. May cause memory leaks if not careful

Use instead:
- try-with-resources for cleanup
- Cleaner class (Java 9+)
- explicit close() methods
*/
```

### Modern Cleanup Approaches

```java
// 1. try-with-resources (preferred for I/O)
public class CleanupExample {
    public void readFile() {
        try (FileInputStream fis = new FileInputStream("file.txt")) {
            // Use resource
        } catch (IOException e) {
            e.printStackTrace();
        }
        // Automatically closed
    }
}

// 2. Cleaner (Java 9+)
import java.lang.ref.Cleaner;

public class ResourceHolder {
    private static final Cleaner CLEANER = Cleaner.create();

    private final Cleaner.Cleanable cleanable;
    private final Resource resource;

    public ResourceHolder() {
        this.resource = new Resource();
        this.cleanable = CLEANER.register(this, resource::cleanup);
    }

    public void close() {
        cleanable.clean();
    }

    private static class Resource {
        void cleanup() {
            System.out.println("Cleaning up resource");
        }
    }
}
```

---

## Types of Garbage Collectors

### 1. Serial GC

```
┌──────────────────────────────────────────────────────┐
│                    SERIAL GC                          │
├──────────────────────────────────────────────────────┤
│                                                       │
│    Application    ████████████████                   │
│    Threads        ░░░░░░░░░░░░░░░░                   │
│                                                       │
│    GC Thread      ░░░░░░░░░░░░░░░░████░░░░░░░░░░░░  │
│                              STOP     RESUME          │
│                              THE                      │
│                              WORLD                    │
│                                                       │
│    Single thread for both Minor and Major GC          │
│                                                       │
└──────────────────────────────────────────────────────┘

Enable: -XX:+UseSerialGC

Best for:
- Single CPU machines
- Small heaps (< 100MB)
- Client applications
```

### 2. Parallel GC (Throughput Collector)

```
┌──────────────────────────────────────────────────────┐
│                    PARALLEL GC                        │
├──────────────────────────────────────────────────────┤
│                                                       │
│    Application    ██████████████                     │
│    Threads        ░░░░░░░░░░░░░░                     │
│                                                       │
│    GC Thread 1    ░░░░░░░░░░░░░░███░░░░░░░░░░░░░░   │
│    GC Thread 2    ░░░░░░░░░░░░░░███░░░░░░░░░░░░░░   │
│    GC Thread 3    ░░░░░░░░░░░░░░███░░░░░░░░░░░░░░   │
│    GC Thread 4    ░░░░░░░░░░░░░░███░░░░░░░░░░░░░░   │
│                              STOP    RESUME           │
│                              THE                      │
│                              WORLD                    │
│                                                       │
│    Multiple threads for GC (parallel collection)      │
│    Default GC for Java 8                             │
│                                                       │
└──────────────────────────────────────────────────────┘

Enable: -XX:+UseParallelGC

Best for:
- Multi-CPU machines
- Throughput-sensitive applications
- Batch processing
```

### 3. G1 GC (Garbage First)

```
┌──────────────────────────────────────────────────────┐
│                      G1 GC                            │
├──────────────────────────────────────────────────────┤
│                                                       │
│    Heap divided into regions (~2048 regions)          │
│                                                       │
│    ┌───┬───┬───┬───┬───┬───┬───┬───┐                │
│    │ E │ E │ S │ O │ O │ H │ E │   │                │
│    ├───┼───┼───┼───┼───┼───┼───┼───┤                │
│    │ O │ E │ O │   │ E │ S │ O │ O │                │
│    ├───┼───┼───┼───┼───┼───┼───┼───┤                │
│    │ E │ O │ H │ H │ O │ E │   │ O │                │
│    └───┴───┴───┴───┴───┴───┴───┴───┘                │
│                                                       │
│    E = Eden, S = Survivor, O = Old, H = Humongous    │
│                                                       │
│    Features:                                          │
│    - Concurrent marking                               │
│    - Incremental compaction                           │
│    - Predictable pause times                          │
│    - Collects "garbage first" (most garbage regions)  │
│                                                       │
└──────────────────────────────────────────────────────┘

Enable: -XX:+UseG1GC (default in Java 9+)

Best for:
- Large heaps (> 4GB)
- Applications requiring predictable latency
- Server applications
```

### 4. ZGC (Z Garbage Collector)

```
┌──────────────────────────────────────────────────────┐
│                      ZGC                              │
├──────────────────────────────────────────────────────┤
│                                                       │
│    Application    ████████████████████████████████   │
│    Threads                                            │
│                                                       │
│    GC Threads     ╔══╗  ╔══╗  ╔══╗  ╔══╗  ╔══╗     │
│    (concurrent)   ║GC║  ║GC║  ║GC║  ║GC║  ║GC║     │
│                   ╚══╝  ╚══╝  ╚══╝  ╚══╝  ╚══╝     │
│                                                       │
│    STW pauses: < 1ms (regardless of heap size!)      │
│                                                       │
│    Features:                                          │
│    - Colored pointers                                 │
│    - Load barriers                                    │
│    - Concurrent compaction                            │
│    - Region-based                                     │
│    - Supports heaps up to 16TB                       │
│                                                       │
└──────────────────────────────────────────────────────┘

Enable: -XX:+UseZGC (Java 11+, production-ready Java 15+)

Best for:
- Ultra-low latency requirements
- Very large heaps (TB scale)
- Real-time applications
```

### 5. Shenandoah GC

```
┌──────────────────────────────────────────────────────┐
│                  SHENANDOAH GC                        │
├──────────────────────────────────────────────────────┤
│                                                       │
│    Similar to ZGC but with different approach         │
│                                                       │
│    Features:                                          │
│    - Concurrent compaction                            │
│    - Brooks forwarding pointers                       │
│    - Low pause times (< 10ms typically)              │
│    - Developed by Red Hat                             │
│                                                       │
│    Phases:                                            │
│    1. Initial Mark (STW - brief)                     │
│    2. Concurrent Mark                                 │
│    3. Final Mark (STW - brief)                       │
│    4. Concurrent Cleanup                              │
│    5. Concurrent Evacuation                           │
│    6. Init Update Refs (STW - brief)                 │
│    7. Concurrent Update Refs                          │
│    8. Final Update Refs (STW - brief)                │
│    9. Concurrent Cleanup                              │
│                                                       │
└──────────────────────────────────────────────────────┘

Enable: -XX:+UseShenandoahGC (Java 12+, backported to 8/11)

Best for:
- Low latency requirements
- Large heaps
- Not in Oracle JDK (OpenJDK only)
```

### GC Comparison Table

| Feature        | Serial | Parallel | G1          | ZGC        | Shenandoah |
| -------------- | ------ | -------- | ----------- | ---------- | ---------- |
| **Threads**    | Single | Multiple | Multiple    | Multiple   | Multiple   |
| **STW Pauses** | Long   | Medium   | Predictable | < 1ms      | < 10ms     |
| **Throughput** | Low    | High     | Good        | Good       | Good       |
| **Heap Size**  | Small  | Any      | Large       | Very Large | Large      |
| **Latency**    | High   | Medium   | Low         | Ultra-Low  | Very Low   |
| **Default**    | -      | Java 8   | Java 9+     | -          | -          |
| **Compaction** | Yes    | Yes      | Incremental | Concurrent | Concurrent |

---

## GC Algorithms

### Copying Algorithm (Young Gen)

```
Used in Young Generation (Minor GC)

Before GC:
EDEN:       [A][B][C][D][E][F][G]
SURVIVOR 0: [H][I]
SURVIVOR 1: (empty - To space)

                    │
                    ▼ Minor GC

Live objects: A, C, F, H (others are garbage)

After GC:
EDEN:       (empty)
SURVIVOR 0: (empty - now becomes To)
SURVIVOR 1: [A][C][F][H] ← All live objects copied here

Advantages:
- Fast for high mortality rate (most objects die young)
- No fragmentation (compacted by copying)

Disadvantages:
- Requires double the memory
- Copying cost for long-lived objects
```

### Mark-Sweep Algorithm (Old Gen)

```
Used in Old Generation

Phase 1: Mark reachable objects
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ B │ C │ D │ E │ F │ G │ H │
│ ✓ │   │ ✓ │   │ ✓ │   │ ✓ │   │
└───┴───┴───┴───┴───┴───┴───┴───┘
  ↑       ↑       ↑       ↑
  Marked  Marked  Marked  Marked

Phase 2: Sweep unmarked objects
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ A │   │ C │   │ E │   │ G │   │
└───┴───┴───┴───┴───┴───┴───┴───┘
      ↑       ↑       ↑       ↑
      Free    Free    Free    Free

Problem: Fragmentation!
Cannot allocate object size 3:
│ A │   │ C │   │ E │   │ G │   │
      ↑ Only 1 slot free in each gap
```

### Mark-Sweep-Compact Algorithm

```
Phase 3: Compact (after Mark-Sweep)

Before Compaction:
│ A │   │ C │   │ E │   │ G │   │

After Compaction:
│ A │ C │ E │ G │                   │
                 ↑───────────────────↑
                 Contiguous free space

Now can allocate larger objects!
```

### Tri-color Marking (G1, ZGC, Shenandoah)

```
Colors:
- WHITE: Not yet visited (potentially garbage)
- GRAY:  Visited but children not processed
- BLACK: Visited and all children processed

Algorithm:
1. Initially all objects are WHITE
2. Start from GC roots, mark them GRAY
3. Process GRAY objects:
   - Mark all references GRAY
   - Mark current object BLACK
4. Repeat until no GRAY objects
5. All WHITE objects are garbage

          Step 1           Step 2           Step 3           Final

    ○ Root            ● Root            ● Root            ● Root
    │                 │                 │                 │
    ▼                 ▼                 ▼                 ▼
    ○ A               ◐ A               ● A               ● A
   / \               / \               / \               / \
  ○   ○             ○   ○             ◐   ◐             ●   ●
  B   C             B   C             B   C             B   C

    ○ X               ○ X               ○ X               ○ X ← Garbage
    ↓                 ↓                 ↓                 ↓
    ○ Y               ○ Y               ○ Y               ○ Y ← Garbage

○ = WHITE, ◐ = GRAY, ● = BLACK
```

---

## Memory Leaks in Java

### What is a Memory Leak?

A memory leak occurs when objects are no longer needed but cannot be garbage collected because they're still referenced.

### Common Memory Leak Patterns

#### 1. Static Field Holding References

```java
// MEMORY LEAK: Static collection grows forever
public class StaticCollectionLeak {
    private static List<Object> cache = new ArrayList<>();

    public void addToCache(Object obj) {
        cache.add(obj);  // Never removed!
    }

    // Objects in cache can never be GC'd while class is loaded
}

// FIX: Use bounded cache or WeakReference
public class FixedCache {
    private static Map<String, WeakReference<Object>> cache =
        new ConcurrentHashMap<>();

    public void addToCache(String key, Object obj) {
        cache.put(key, new WeakReference<>(obj));
    }

    public Object getFromCache(String key) {
        WeakReference<Object> ref = cache.get(key);
        return ref != null ? ref.get() : null;
    }
}
```

#### 2. Unclosed Resources

```java
// MEMORY LEAK: Unclosed connection
public class ResourceLeak {
    public void readFile() {
        try {
            FileInputStream fis = new FileInputStream("file.txt");
            // If exception occurs here, fis is never closed!
            int data = fis.read();
            fis.close();  // May not be reached
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// FIX: Use try-with-resources
public class FixedResource {
    public void readFile() {
        try (FileInputStream fis = new FileInputStream("file.txt")) {
            int data = fis.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
        // Automatically closed even if exception occurs
    }
}
```

#### 3. Inner Class Holding Outer Reference

```java
// MEMORY LEAK: Inner class holds reference to outer
public class OuterClass {
    private byte[] largeData = new byte[10_000_000]; // 10MB

    public Object getInner() {
        // Non-static inner class holds implicit reference to OuterClass
        return new Object() {
            @Override
            public String toString() {
                return "Inner object";
            }
        };
    }
}

// FIX: Use static inner class
public class FixedOuter {
    private byte[] largeData = new byte[10_000_000];

    public Object getInner() {
        return new StaticInner();
    }

    private static class StaticInner {
        @Override
        public String toString() {
            return "Static inner object";
        }
    }
}
```

#### 4. Listener/Callback Not Deregistered

```java
// MEMORY LEAK: Listener never removed
public class ListenerLeak {
    private List<EventListener> listeners = new ArrayList<>();

    public void addListener(EventListener listener) {
        listeners.add(listener);
    }

    // Missing removeListener() or listeners never cleaned up
}

// FIX: Provide removal and use WeakReference
public class FixedListener {
    private List<WeakReference<EventListener>> listeners =
        new ArrayList<>();

    public void addListener(EventListener listener) {
        listeners.add(new WeakReference<>(listener));
    }

    public void removeListener(EventListener listener) {
        listeners.removeIf(ref ->
            ref.get() == null || ref.get() == listener);
    }

    private void notifyListeners() {
        listeners.removeIf(ref -> ref.get() == null);
        listeners.forEach(ref -> {
            EventListener l = ref.get();
            if (l != null) l.onEvent();
        });
    }
}
```

#### 5. String.intern() Abuse

```java
// MEMORY LEAK: Too many interned strings
public class InternLeak {
    public void processData(List<String> data) {
        for (String s : data) {
            String interned = s.intern();  // Adds to string pool
            // String pool is in Metaspace, grows unbounded
        }
    }
}

// FIX: Only intern truly needed strings
public class FixedIntern {
    private Set<String> importantStrings = new HashSet<>();

    public String internIfImportant(String s) {
        if (isImportant(s)) {
            return s.intern();
        }
        return s;
    }
}
```

#### 6. ThreadLocal Not Cleaned

```java
// MEMORY LEAK: ThreadLocal not removed
public class ThreadLocalLeak {
    private static ThreadLocal<byte[]> threadLocalData =
        ThreadLocal.withInitial(() -> new byte[1_000_000]);

    public void processRequest() {
        byte[] data = threadLocalData.get();
        // Use data...
        // Thread returns to pool but ThreadLocal value remains!
    }
}

// FIX: Always remove ThreadLocal value
public class FixedThreadLocal {
    private static ThreadLocal<byte[]> threadLocalData =
        ThreadLocal.withInitial(() -> new byte[1_000_000]);

    public void processRequest() {
        try {
            byte[] data = threadLocalData.get();
            // Use data...
        } finally {
            threadLocalData.remove();  // Always clean up!
        }
    }
}
```

### Reference Types

```java
// Strong Reference (default) - Never GC'd while reachable
Object strong = new Object();

// Soft Reference - GC'd when memory is low
SoftReference<Object> soft = new SoftReference<>(new Object());
// Good for caches - cleared before OutOfMemoryError

// Weak Reference - GC'd at next collection
WeakReference<Object> weak = new WeakReference<>(new Object());
// Good for canonicalizing mappings

// Phantom Reference - For cleanup actions
PhantomReference<Object> phantom = new PhantomReference<>(
    new Object(), new ReferenceQueue<>());
// Object is already finalized when enqueued

// WeakHashMap - Keys are weak references
Map<Object, String> weakMap = new WeakHashMap<>();
Object key = new Object();
weakMap.put(key, "value");
key = null;  // Entry can be removed by GC
```

---

## JVM Tuning Parameters

### Heap Size Parameters

```bash
# Initial heap size
-Xms512m       # 512MB initial heap

# Maximum heap size
-Xmx4g         # 4GB maximum heap

# Young generation size
-Xmn256m       # 256MB young generation

# Metaspace size
-XX:MetaspaceSize=128m        # Initial metaspace
-XX:MaxMetaspaceSize=256m     # Maximum metaspace

# Stack size per thread
-Xss256k       # 256KB per thread stack
```

### GC Selection

```bash
# Serial GC
-XX:+UseSerialGC

# Parallel GC
-XX:+UseParallelGC
-XX:ParallelGCThreads=4       # Number of GC threads

# G1 GC
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200      # Target pause time
-XX:G1HeapRegionSize=16m      # Region size

# ZGC
-XX:+UseZGC
-XX:ZCollectionInterval=5     # Max interval between collections

# Shenandoah
-XX:+UseShenandoahGC
```

### G1 Specific Tuning

```bash
# G1 GC tuning parameters
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200         # Target max pause
-XX:G1HeapRegionSize=32m         # Region size (1-32MB)
-XX:G1ReservePercent=10          # Reserve for promotions
-XX:InitiatingHeapOccupancyPercent=45  # Start concurrent cycle
-XX:G1MixedGCLiveThresholdPercent=65   # Old region inclusion threshold
-XX:G1MixedGCCountTarget=8       # Number of mixed GCs
```

### GC Logging

```bash
# Java 8 style
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:/path/to/gc.log

# Java 9+ unified logging
-Xlog:gc*:file=/path/to/gc.log:time,uptime,level,tags
-Xlog:gc+heap=debug
-Xlog:gc+phases=debug
```

### Memory Tuning Best Practices

```bash
# Production-ready G1 configuration example
java \
  -Xms4g \
  -Xmx4g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:+UseStringDeduplication \
  -XX:+ParallelRefProcEnabled \
  -XX:+DisableExplicitGC \
  -XX:+AlwaysPreTouch \
  -Xlog:gc*:file=gc.log:time,uptime,level,tags \
  -jar application.jar
```

### Common JVM Flags Table

| Flag                       | Purpose               | Default             |
| -------------------------- | --------------------- | ------------------- |
| `-Xms`                     | Initial heap size     | Platform specific   |
| `-Xmx`                     | Maximum heap size     | 1/4 physical memory |
| `-Xmn`                     | Young generation size | -                   |
| `-Xss`                     | Thread stack size     | 1MB (64-bit)        |
| `-XX:NewRatio`             | Old/Young ratio       | 2                   |
| `-XX:SurvivorRatio`        | Eden/Survivor ratio   | 8                   |
| `-XX:MaxTenuringThreshold` | Promotion threshold   | 15                  |
| `-XX:+DisableExplicitGC`   | Ignore System.gc()    | false               |

---

## Monitoring & Profiling

### JVM Tools

```bash
# jstat - GC statistics
jstat -gc <pid> 1000       # GC stats every 1 second
jstat -gcutil <pid> 1000   # GC utilization percentages

# jmap - Memory map
jmap -heap <pid>           # Heap summary
jmap -histo <pid>          # Object histogram
jmap -dump:live,format=b,file=heap.hprof <pid>  # Heap dump

# jcmd - Diagnostic commands
jcmd <pid> GC.heap_info    # Heap information
jcmd <pid> GC.run          # Trigger GC
jcmd <pid> VM.flags        # All VM flags

# jinfo - Configuration info
jinfo -flags <pid>         # All flags
jinfo -sysprops <pid>      # System properties
```

### VisualVM / JConsole

```java
// Enable JMX for remote monitoring
// Add to JVM args:
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=9010
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

### Programmatic GC Monitoring

```java
import java.lang.management.*;
import java.util.List;

public class GCMonitor {
    public static void main(String[] args) {
        // Get memory MXBean
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();

        // Heap memory
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        System.out.println("Heap Memory:");
        System.out.println("  Init: " + heapUsage.getInit() / 1024 / 1024 + " MB");
        System.out.println("  Used: " + heapUsage.getUsed() / 1024 / 1024 + " MB");
        System.out.println("  Committed: " + heapUsage.getCommitted() / 1024 / 1024 + " MB");
        System.out.println("  Max: " + heapUsage.getMax() / 1024 / 1024 + " MB");

        // GC beans
        List<GarbageCollectorMXBean> gcBeans =
            ManagementFactory.getGarbageCollectorMXBeans();

        for (GarbageCollectorMXBean gcBean : gcBeans) {
            System.out.println("\nGC: " + gcBean.getName());
            System.out.println("  Collection count: " + gcBean.getCollectionCount());
            System.out.println("  Collection time: " + gcBean.getCollectionTime() + " ms");
        }

        // Memory pools
        List<MemoryPoolMXBean> poolBeans =
            ManagementFactory.getMemoryPoolMXBeans();

        for (MemoryPoolMXBean poolBean : poolBeans) {
            System.out.println("\nPool: " + poolBean.getName());
            System.out.println("  Type: " + poolBean.getType());
            MemoryUsage usage = poolBean.getUsage();
            System.out.println("  Used: " + usage.getUsed() / 1024 / 1024 + " MB");
        }
    }
}
```

---

## Interview Questions

### Q1: What are the different memory areas in JVM?

**Answer:**

1. **Heap** - Object storage (shared across threads)
   - Young Generation (Eden + Survivors)
   - Old Generation (Tenured)
2. **Stack** - Per-thread, stores local variables and method calls
3. **Metaspace** - Class metadata (replaced PermGen in Java 8)
4. **PC Register** - Per-thread, current instruction address
5. **Native Method Stack** - For native (JNI) methods

---

### Q2: Explain the generational garbage collection concept.

**Answer:**
Based on the **weak generational hypothesis** - most objects die young.

**Generations:**

1. **Young Gen** - New objects allocated here
   - Eden: Initial allocation
   - Survivor spaces: Objects that survive GC
   - Minor GC: Frequent, fast collection
2. **Old Gen** - Long-lived objects
   - Objects promoted after surviving threshold
   - Major GC: Less frequent, slower

**Benefits:**

- Optimizes for common case (short-lived objects)
- Reduces GC pause times
- Different algorithms for different generations

---

### Q3: What is the difference between Minor GC and Major GC?

| Aspect        | Minor GC            | Major GC                 |
| ------------- | ------------------- | ------------------------ |
| **Area**      | Young Generation    | Old Generation (+ Young) |
| **Frequency** | Very frequent       | Less frequent            |
| **Speed**     | Fast (milliseconds) | Slow (seconds possible)  |
| **Algorithm** | Copying             | Mark-Sweep-Compact       |
| **Trigger**   | Eden full           | Old Gen full             |
| **STW**       | Short pause         | Longer pause             |

---

### Q4: How does G1 GC work?

**Answer:**
G1 (Garbage First) divides heap into equal-sized regions:

1. **Region-based** - Heap split into ~2048 regions
2. **Regions can be** - Eden, Survivor, Old, or Humongous
3. **Concurrent marking** - Identifies garbage during application run
4. **Priority collection** - Collects regions with most garbage first
5. **Predictable pauses** - Target pause time with `-XX:MaxGCPauseMillis`
6. **Incremental compaction** - Compacts portions each cycle

**Phases:**

1. Initial Mark (STW)
2. Root Region Scan
3. Concurrent Mark
4. Remark (STW)
5. Cleanup (STW + Concurrent)
6. Copy/Evacuation (STW)

---

### Q5: What causes OutOfMemoryError?

**Answer:**
Different OOM errors have different causes:

1. **Java heap space**

   - Heap exhausted
   - Memory leak
   - Insufficient `-Xmx`

2. **Metaspace** (Java 8+)

   - Too many classes loaded
   - Class loader leak
   - Increase `-XX:MaxMetaspaceSize`

3. **GC overhead limit exceeded**

   - 98% time spent in GC, <2% heap recovered
   - Memory leak or heap too small

4. **Unable to create new native thread**

   - Too many threads
   - OS thread limit
   - Reduce `-Xss` or thread count

5. **Direct buffer memory**
   - NIO direct buffers exhausted
   - Increase `-XX:MaxDirectMemorySize`

---

### Q6: How do you detect and fix memory leaks?

**Detection:**

1. Monitor heap usage over time (should not grow indefinitely)
2. Analyze heap dumps with MAT or VisualVM
3. Use profilers (JProfiler, YourKit)
4. GC logs showing increasing old gen

**Common fixes:**

1. Remove objects from collections when done
2. Use weak references for caches
3. Close resources in finally/try-with-resources
4. Remove listeners when no longer needed
5. Use static inner classes instead of non-static
6. Always call `ThreadLocal.remove()`

---

### Q7: What is the difference between Soft, Weak, and Phantom references?

| Reference Type | GC Behavior                     | Use Case                |
| -------------- | ------------------------------- | ----------------------- |
| **Strong**     | Never collected while reachable | Normal references       |
| **Soft**       | Collected when memory is low    | Caches                  |
| **Weak**       | Collected at next GC            | Canonicalizing mappings |
| **Phantom**    | After finalization, for cleanup | Resource cleanup        |

```java
SoftReference<Object> soft = new SoftReference<>(obj);
WeakReference<Object> weak = new WeakReference<>(obj);
PhantomReference<Object> phantom = new PhantomReference<>(obj, queue);
```

---

### Q8: What JVM parameters would you use for a production application?

**Answer:**

```bash
java \
  -Xms4g -Xmx4g \                    # Fixed heap size
  -XX:+UseG1GC \                      # G1 collector
  -XX:MaxGCPauseMillis=200 \          # Target pause
  -XX:+UseStringDeduplication \       # Save memory
  -XX:+DisableExplicitGC \            # Ignore System.gc()
  -XX:+HeapDumpOnOutOfMemoryError \   # Dump on OOM
  -XX:HeapDumpPath=/path/to/dumps \   # Dump location
  -Xlog:gc*:file=gc.log:time \        # GC logging
  -XX:+AlwaysPreTouch \               # Pre-touch heap pages
  -jar app.jar
```

---

### Q9: Why is `System.gc()` discouraged?

**Answer:**

1. **Not guaranteed** - JVM may ignore the request
2. **Full GC** - Usually triggers expensive Full GC
3. **Unpredictable timing** - Can cause pause at wrong time
4. **Already optimized** - JVM knows best when to collect
5. **Production impact** - Can cause latency spikes

Best practice: Use `-XX:+DisableExplicitGC` to ignore calls.

---

### Q10: Compare ZGC vs G1 GC

| Feature        | G1                | ZGC                      |
| -------------- | ----------------- | ------------------------ |
| **Max pause**  | 200ms target      | < 1ms                    |
| **Heap size**  | Up to TBs         | Up to 16TB               |
| **Compaction** | Incremental (STW) | Concurrent               |
| **Technique**  | Regions           | Colored pointers         |
| **Available**  | Java 7+           | Java 11+ (prod Java 15+) |
| **Use case**   | General purpose   | Ultra-low latency        |

---

## Key Takeaways

| Topic              | Key Point                                          |
| ------------------ | -------------------------------------------------- |
| **Heap Structure** | Young (Eden + Survivors) + Old Generation          |
| **GC Roots**       | Static fields, local vars, threads, JNI            |
| **Minor GC**       | Young gen, fast, frequent, copying algorithm       |
| **Major GC**       | Old gen, slow, mark-sweep-compact                  |
| **G1 GC**          | Region-based, predictable pauses, default Java 9+  |
| **ZGC**            | Sub-millisecond pauses, concurrent compaction      |
| **Memory Leaks**   | Collections, listeners, ThreadLocal, inner classes |
| **Tuning**         | -Xms/-Xmx same, GC logging, monitor before tuning  |
