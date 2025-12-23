# JVM Internals & Performance Tuning - Complete Interview Guide

## Table of Contents

1. [JVM Architecture Deep Dive](#jvm-architecture-deep-dive)
2. [Class Loading Mechanism](#class-loading-mechanism)
3. [JIT Compilation](#jit-compilation)
4. [Memory Model (JMM)](#memory-model-jmm)
5. [JVM Performance Tuning](#jvm-performance-tuning)
6. [Profiling and Diagnostics](#profiling-and-diagnostics)
7. [Common JVM Issues](#common-jvm-issues)
8. [Interview Questions](#interview-questions)

---

## JVM Architecture Deep Dive

### Complete JVM Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              JAVA VIRTUAL MACHINE                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         CLASS LOADER SUBSYSTEM                        │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐                      │   │
│  │  │  Loading   │→ │  Linking   │→ │Initialization│                    │   │
│  │  │            │  │            │  │             │                      │   │
│  │  │ Bootstrap  │  │ Verify     │  │ Static      │                      │   │
│  │  │ Extension  │  │ Prepare    │  │ initializers│                      │   │
│  │  │ Application│  │ Resolve    │  │ executed    │                      │   │
│  │  └────────────┘  └────────────┘  └────────────┘                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                      │                                       │
│                                      ▼                                       │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                        RUNTIME DATA AREAS                             │   │
│  │                                                                       │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │   │
│  │  │                    SHARED ACROSS THREADS                         │ │   │
│  │  │  ┌───────────────────────┐  ┌────────────────────────────────┐  │ │   │
│  │  │  │      HEAP MEMORY      │  │         METASPACE              │  │ │   │
│  │  │  │  ┌─────────────────┐  │  │  - Class metadata              │  │ │   │
│  │  │  │  │   Young Gen     │  │  │  - Method bytecode             │  │ │   │
│  │  │  │  │ Eden│ S0 │ S1   │  │  │  - Constant pool               │  │ │   │
│  │  │  │  └─────────────────┘  │  │  - Annotations                 │  │ │   │
│  │  │  │  ┌─────────────────┐  │  │                                │  │ │   │
│  │  │  │  │   Old Gen       │  │  │  (Native memory, auto-grows)   │  │ │   │
│  │  │  │  └─────────────────┘  │  └────────────────────────────────┘  │ │   │
│  │  │  └───────────────────────┘                                      │ │   │
│  │  └─────────────────────────────────────────────────────────────────┘ │   │
│  │                                                                       │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │   │
│  │  │                     PER-THREAD AREAS                             │ │   │
│  │  │                                                                  │ │   │
│  │  │  Thread 1            Thread 2            Thread N                │ │   │
│  │  │  ┌──────────┐        ┌──────────┐        ┌──────────┐           │ │   │
│  │  │  │ PC Reg   │        │ PC Reg   │        │ PC Reg   │           │ │   │
│  │  │  ├──────────┤        ├──────────┤        ├──────────┤           │ │   │
│  │  │  │ JVM Stack│        │ JVM Stack│        │ JVM Stack│           │ │   │
│  │  │  │ ┌──────┐ │        │ ┌──────┐ │        │ ┌──────┐ │           │ │   │
│  │  │  │ │Frame │ │        │ │Frame │ │        │ │Frame │ │           │ │   │
│  │  │  │ ├──────┤ │        │ ├──────┤ │        │ ├──────┤ │           │ │   │
│  │  │  │ │Frame │ │        │ │Frame │ │        │ │Frame │ │           │ │   │
│  │  │  │ └──────┘ │        │ └──────┘ │        │ └──────┘ │           │ │   │
│  │  │  ├──────────┤        ├──────────┤        ├──────────┤           │ │   │
│  │  │  │ Native   │        │ Native   │        │ Native   │           │ │   │
│  │  │  │ Method   │        │ Method   │        │ Method   │           │ │   │
│  │  │  │ Stack    │        │ Stack    │        │ Stack    │           │ │   │
│  │  │  └──────────┘        └──────────┘        └──────────┘           │ │   │
│  │  └─────────────────────────────────────────────────────────────────┘ │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                      │                                       │
│                                      ▼                                       │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                        EXECUTION ENGINE                               │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────────────┐  │   │
│  │  │  Interpreter   │  │  JIT Compiler  │  │  Garbage Collector     │  │   │
│  │  │                │  │                │  │                        │  │   │
│  │  │  Bytecode      │  │  C1 (Client)   │  │  Serial, Parallel,     │  │   │
│  │  │  execution     │  │  C2 (Server)   │  │  G1, ZGC, Shenandoah   │  │   │
│  │  │  line by line  │  │  Graal         │  │                        │  │   │
│  │  └────────────────┘  └────────────────┘  └────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                      │                                       │
│                                      ▼                                       │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    NATIVE METHOD INTERFACE (JNI)                      │   │
│  │              Connects JVM with Native Libraries (C/C++)               │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Stack Frame Structure

```
┌─────────────────────────────────────────────────┐
│                  STACK FRAME                     │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │          LOCAL VARIABLES ARRAY            │  │
│  │  ┌─────┬─────┬─────┬─────┬─────┬─────┐   │  │
│  │  │ 0   │ 1   │ 2   │ 3   │ 4   │ ... │   │  │
│  │  │this │arg1 │arg2 │var1 │var2 │     │   │  │
│  │  └─────┴─────┴─────┴─────┴─────┴─────┘   │  │
│  │  (For instance methods, slot 0 = this)   │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │            OPERAND STACK                  │  │
│  │  ┌─────┬─────┬─────┬─────┐               │  │
│  │  │ val │ val │ val │ ... │  ← Top        │  │
│  │  └─────┴─────┴─────┴─────┘               │  │
│  │  (Used for computation, method calls)     │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │       FRAME DATA / CONSTANT POOL REF      │  │
│  │  - Reference to runtime constant pool     │  │
│  │  - Exception table reference              │  │
│  │  - Return address                         │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
└─────────────────────────────────────────────────┘
```

### Bytecode Example

```java
// Java Source Code
public int add(int a, int b) {
    int sum = a + b;
    return sum;
}

// Compiled Bytecode (javap -c)
public int add(int, int);
  Code:
     0: iload_1        // Load int from local variable 1 (a) onto stack
     1: iload_2        // Load int from local variable 2 (b) onto stack
     2: iadd           // Add top two ints on stack, push result
     3: istore_3       // Store result into local variable 3 (sum)
     4: iload_3        // Load sum onto stack
     5: ireturn        // Return int from method

/*
Local Variables:
  Slot 0: this (implicit for instance methods)
  Slot 1: a
  Slot 2: b
  Slot 3: sum

Operand Stack at each step:
  Step 0: []
  Step 1: [a]
  Step 2: [a, b]
  Step 3: [sum]
  Step 4: []
  Step 5: [sum]
*/
```

---

## Class Loading Mechanism

### Class Loader Hierarchy

```
                    ┌──────────────────────────┐
                    │   Bootstrap ClassLoader  │ (Native Code)
                    │   Loads: rt.jar, core    │
                    │   Java classes           │
                    └────────────┬─────────────┘
                                 │ parent
                                 ▼
                    ┌──────────────────────────┐
                    │  Platform ClassLoader    │ (Java 9+)
                    │  (Extension ClassLoader) │ (Java 8)
                    │  Loads: ext/*.jar        │
                    └────────────┬─────────────┘
                                 │ parent
                                 ▼
                    ┌──────────────────────────┐
                    │  Application ClassLoader │
                    │  (System ClassLoader)    │
                    │  Loads: classpath        │
                    └────────────┬─────────────┘
                                 │ parent
                                 ▼
                    ┌──────────────────────────┐
                    │   Custom ClassLoaders    │
                    │   (User defined)         │
                    └──────────────────────────┘
```

### Class Loading Process

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      CLASS LOADING PHASES                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  LOADING                                                                 │
│  ├── Find .class file (classpath, network, generated)                   │
│  ├── Read byte array                                                    │
│  └── Create java.lang.Class object in Metaspace                         │
│                            │                                             │
│                            ▼                                             │
│  LINKING                                                                 │
│  ├── VERIFICATION                                                       │
│  │   ├── Bytecode verification                                          │
│  │   ├── Type checking                                                  │
│  │   └── Stack map frame validation                                     │
│  │                                                                      │
│  ├── PREPARATION                                                        │
│  │   ├── Allocate memory for static variables                           │
│  │   └── Set default values (0, null, false)                           │
│  │                                                                      │
│  └── RESOLUTION (Optional - can be lazy)                                │
│      ├── Replace symbolic references with direct references             │
│      └── Resolve class, method, field references                        │
│                            │                                             │
│                            ▼                                             │
│  INITIALIZATION                                                          │
│  ├── Execute static initializers (<clinit>)                             │
│  ├── Initialize static variables with actual values                     │
│  └── Run static blocks in order                                         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Parent Delegation Model

```java
// How class loading works with parent delegation
public class ClassLoaderDemo {
    public static void main(String[] args) {
        // String.class loaded by Bootstrap (null returned)
        System.out.println(String.class.getClassLoader());  // null

        // Application class loaded by Application ClassLoader
        System.out.println(ClassLoaderDemo.class.getClassLoader());
        // jdk.internal.loader.ClassLoaders$AppClassLoader

        // Parent chain
        ClassLoader cl = ClassLoaderDemo.class.getClassLoader();
        while (cl != null) {
            System.out.println(cl);
            cl = cl.getParent();
        }
        /*
        Output:
        jdk.internal.loader.ClassLoaders$AppClassLoader@...
        jdk.internal.loader.ClassLoaders$PlatformClassLoader@...
        null (Bootstrap - native)
        */
    }
}
```

### Custom ClassLoader

```java
public class CustomClassLoader extends ClassLoader {

    private String classPath;

    public CustomClassLoader(String classPath, ClassLoader parent) {
        super(parent);
        this.classPath = classPath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            // Convert class name to file path
            String fileName = classPath + "/" +
                name.replace('.', '/') + ".class";

            // Read class bytes
            byte[] classBytes = Files.readAllBytes(Paths.get(fileName));

            // Define the class
            return defineClass(name, classBytes, 0, classBytes.length);

        } catch (IOException e) {
            throw new ClassNotFoundException("Could not load " + name, e);
        }
    }

    // Usage
    public static void main(String[] args) throws Exception {
        CustomClassLoader loader = new CustomClassLoader(
            "/custom/classes",
            ClassLoader.getSystemClassLoader()
        );

        Class<?> clazz = loader.loadClass("com.example.MyClass");
        Object instance = clazz.getDeclaredConstructor().newInstance();

        // Classes loaded by different loaders are different!
        CustomClassLoader loader2 = new CustomClassLoader(
            "/custom/classes",
            ClassLoader.getSystemClassLoader()
        );
        Class<?> clazz2 = loader2.loadClass("com.example.MyClass");

        System.out.println(clazz == clazz2);  // false - different Class objects!
    }
}
```

### When Classes are Loaded

```java
public class ClassLoadingTriggers {

    // Classes are loaded when:

    // 1. Creating instance with 'new'
    MyClass obj = new MyClass();

    // 2. Accessing static field (not final compile-time constant)
    int value = MyClass.staticField;

    // 3. Calling static method
    MyClass.staticMethod();

    // 4. Reflection
    Class.forName("com.example.MyClass");

    // 5. Subclass initialization loads parent
    class Child extends Parent { }  // Parent loaded first

    // 6. Main class at startup

    // Classes are NOT loaded for:
    // - Declaring a variable (no initialization)
    // - Accessing final static compile-time constants
    // - Array of class type (only array class loaded)
}
```

---

## JIT Compilation

### JIT Compilation Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       JIT COMPILATION PROCESS                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                         Method Execution                                 │
│                              │                                           │
│                              ▼                                           │
│                    ┌─────────────────┐                                  │
│                    │   Interpreter   │                                  │
│                    │  (Initial exec) │                                  │
│                    └────────┬────────┘                                  │
│                             │                                           │
│                    Invocation Counter++                                 │
│                             │                                           │
│              ┌──────────────┴──────────────┐                           │
│              │                              │                           │
│              ▼                              ▼                           │
│     Counter < Threshold          Counter >= Threshold                   │
│              │                              │                           │
│              ▼                              ▼                           │
│     Continue with                   Trigger JIT                         │
│     Interpreter                     Compilation                         │
│                                            │                            │
│                              ┌─────────────┼─────────────┐              │
│                              ▼             ▼             ▼              │
│                          ┌─────┐       ┌─────┐       ┌─────┐           │
│                          │ C1  │       │ C2  │       │Graal│           │
│                          │Comp.│       │Comp.│       │Comp.│           │
│                          └──┬──┘       └──┬──┘       └──┬──┘           │
│                             │             │             │               │
│                      Quick compile  Deep optimize  Advanced             │
│                      Less optimize  Slower compile optimize             │
│                                            │                            │
│                                            ▼                            │
│                                   Native Machine Code                   │
│                                    (Cached in Code Cache)               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Tiered Compilation

```
Level 0: Interpreter
    │
    ▼ (After ~1,500 invocations)
Level 1: C1 with full optimization
    │
    ▼ (After ~10,000 invocations)
Level 2: C1 with invocation and backedge counters
    │
    ▼
Level 3: C1 with full profiling
    │
    ▼ (After profiling complete)
Level 4: C2 with full optimization
```

### JIT Optimizations

```java
// 1. METHOD INLINING
// Before inlining:
public int calculate(int x) {
    return square(x) + 1;
}

private int square(int n) {
    return n * n;
}

// After inlining:
public int calculate(int x) {
    return x * x + 1;  // square() inlined
}


// 2. ESCAPE ANALYSIS
public long sumOfSquares() {
    long sum = 0;
    for (int i = 0; i < 1000; i++) {
        Point p = new Point(i, i);  // Object doesn't escape
        sum += p.x * p.x + p.y * p.y;
    }
    return sum;
}
// JIT can allocate Point on stack or eliminate allocation entirely


// 3. LOOP UNROLLING
// Before:
for (int i = 0; i < 4; i++) {
    sum += array[i];
}

// After unrolling:
sum += array[0];
sum += array[1];
sum += array[2];
sum += array[3];


// 4. DEAD CODE ELIMINATION
public int deadCode(int x) {
    int unused = x * 2;  // Never used - eliminated
    return x + 1;
}


// 5. NULL CHECK ELIMINATION
public void process(Object obj) {
    if (obj != null) {
        obj.method1();  // Null check
        obj.method2();  // JIT eliminates redundant null check
        obj.method3();  // JIT eliminates redundant null check
    }
}
```

### JIT Flags

```bash
# Print compilation events
-XX:+PrintCompilation

# Print inlining decisions
-XX:+PrintInlining

# Unlock diagnostic options
-XX:+UnlockDiagnosticVMOptions

# Adjust compilation threshold
-XX:CompileThreshold=10000

# Enable tiered compilation (default in Java 8+)
-XX:+TieredCompilation

# Set code cache size
-XX:ReservedCodeCacheSize=256m

# Disable C2 compiler
-XX:TieredStopAtLevel=1

# Use Graal compiler (Java 11+)
-XX:+UseJVMCICompiler
```

---

## Memory Model (JMM)

### Java Memory Model Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     JAVA MEMORY MODEL (JMM)                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Thread 1                  MAIN MEMORY                  Thread 2        │
│  ┌──────────────┐                                  ┌──────────────┐     │
│  │ Local Memory │       ┌─────────────────┐        │ Local Memory │     │
│  │              │       │                 │        │              │     │
│  │ ┌──────────┐ │       │  Shared         │        │ ┌──────────┐ │     │
│  │ │ x = 1    │ │ ────► │  Variables      │ ◄───── │ │ x = ?    │ │     │
│  │ │ y = 2    │ │       │  ┌─────────┐    │        │ │ y = ?    │ │     │
│  │ └──────────┘ │       │  │ x = 0   │    │        │ └──────────┘ │     │
│  │              │       │  │ y = 0   │    │        │              │     │
│  │  (CPU Cache, │       │  └─────────┘    │        │  (CPU Cache, │     │
│  │   Registers) │       │                 │        │   Registers) │     │
│  └──────────────┘       └─────────────────┘        └──────────────┘     │
│                                                                          │
│  Problems without synchronization:                                       │
│  1. Visibility - Thread 2 may not see Thread 1's changes                │
│  2. Ordering - Operations may be reordered                              │
│  3. Atomicity - Compound operations not atomic                          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Happens-Before Relationships

```java
// 1. Program Order Rule
// Within a thread, each action happens-before subsequent actions
int x = 1;  // happens-before
int y = 2;  // this line

// 2. Monitor Lock Rule
synchronized(lock) {
    // Everything inside happens-before
}
// unlock happens-before subsequent lock

// 3. Volatile Variable Rule
volatile boolean flag = false;
// Write to volatile happens-before subsequent read

// 4. Thread Start Rule
thread.start();  // happens-before any action in the thread

// 5. Thread Join Rule
thread.join();
// All actions in thread happen-before join returns

// 6. Transitivity
// If A happens-before B and B happens-before C
// Then A happens-before C
```

### Volatile Deep Dive

```java
public class VolatileExample {

    private volatile boolean running = true;
    private int counter = 0;

    public void stop() {
        running = false;  // Write to volatile
    }

    public void run() {
        while (running) {  // Read from volatile
            counter++;
        }
    }

    /*
    What volatile guarantees:
    1. VISIBILITY - All threads see the latest value
    2. ORDERING - Prevents reordering of reads/writes around volatile

    What volatile does NOT guarantee:
    - Atomicity of compound operations (counter++ is NOT atomic)
    */
}

// Volatile for double-checked locking
public class Singleton {
    private static volatile Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {                    // First check (no lock)
            synchronized (Singleton.class) {
                if (instance == null) {            // Second check (with lock)
                    instance = new Singleton();    // Must be volatile!
                }
            }
        }
        return instance;
    }

    /*
    Why volatile is required:
    Without volatile, instance = new Singleton() can be reordered:
    1. Allocate memory
    2. Assign reference to instance  ← Other thread sees non-null
    3. Call constructor              ← But object not initialized!

    Volatile prevents this reordering.
    */
}
```

### Memory Barriers

```
┌─────────────────────────────────────────────────────────────────┐
│                     MEMORY BARRIERS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LoadLoad Barrier:   Load1; LoadLoad; Load2                     │
│  - Ensures Load1 completes before Load2                         │
│                                                                  │
│  StoreStore Barrier: Store1; StoreStore; Store2                 │
│  - Ensures Store1 is visible before Store2                      │
│                                                                  │
│  LoadStore Barrier:  Load1; LoadStore; Store2                   │
│  - Ensures Load1 completes before Store2                        │
│                                                                  │
│  StoreLoad Barrier:  Store1; StoreLoad; Load2                   │
│  - Ensures Store1 is visible to all processors before Load2     │
│  - Most expensive barrier                                        │
│                                                                  │
│  Volatile Write: StoreStore + StoreLoad barriers                │
│  Volatile Read:  LoadLoad + LoadStore barriers                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## JVM Performance Tuning

### Performance Tuning Methodology

```
┌─────────────────────────────────────────────────────────────────┐
│                 PERFORMANCE TUNING STEPS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. DEFINE GOALS                                                │
│     ├── Throughput target (transactions/sec)                    │
│     ├── Latency target (p99 < 100ms)                           │
│     └── Memory footprint limits                                 │
│                                                                  │
│  2. ESTABLISH BASELINE                                          │
│     ├── Measure current performance                             │
│     ├── Profile application behavior                            │
│     └── Identify bottlenecks                                    │
│                                                                  │
│  3. ANALYZE                                                     │
│     ├── GC logs analysis                                        │
│     ├── Thread dumps                                            │
│     ├── Heap dumps                                              │
│     └── CPU/Memory profiling                                    │
│                                                                  │
│  4. TUNE                                                        │
│     ├── Adjust heap size                                        │
│     ├── Select appropriate GC                                   │
│     ├── Tune GC parameters                                      │
│     └── Optimize application code                               │
│                                                                  │
│  5. VALIDATE                                                    │
│     ├── Measure improvement                                     │
│     ├── Test under production-like load                         │
│     └── Monitor for regressions                                 │
│                                                                  │
│  6. ITERATE                                                     │
│     └── Repeat until goals are met                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Heap Sizing Guidelines

```bash
# General Guidelines:

# 1. Set -Xms = -Xmx (avoid resize overhead)
-Xms4g -Xmx4g

# 2. Young Generation sizing
# - Higher ratio = more frequent Minor GC but faster
# - Default NewRatio=2 means Old:Young = 2:1
-XX:NewRatio=2
# Or set directly
-Xmn1g

# 3. Survivor ratio
# - Default SurvivorRatio=8 means Eden:S0:S1 = 8:1:1
-XX:SurvivorRatio=8

# 4. Metaspace
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m

# Common configurations by application type:

# Web Application (latency-sensitive)
java -Xms4g -Xmx4g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:+UseStringDeduplication \
     -jar app.jar

# Batch Processing (throughput-oriented)
java -Xms8g -Xmx8g \
     -XX:+UseParallelGC \
     -XX:ParallelGCThreads=8 \
     -jar batch.jar

# Low-Latency Application
java -Xms16g -Xmx16g \
     -XX:+UseZGC \
     -XX:ZCollectionInterval=5 \
     -jar trading.jar
```

### GC Tuning by Collector

```bash
# G1 GC Tuning (Recommended for most applications)
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200          # Target pause time
-XX:G1HeapRegionSize=16m          # Region size (1-32MB)
-XX:G1ReservePercent=10           # Reserve for promotions
-XX:InitiatingHeapOccupancyPercent=45  # Start marking cycle
-XX:G1MixedGCLiveThresholdPercent=65   # Old region eligibility
-XX:G1HeapWastePercent=5          # Tolerable waste
-XX:+G1UseAdaptiveIHOP            # Auto-tune IHOP

# ZGC Tuning (Ultra-low latency)
-XX:+UseZGC
-XX:+ZGenerational                # Generational ZGC (Java 21+)
-XX:SoftMaxHeapSize=8g            # Soft limit before aggressive GC
-XX:ZCollectionInterval=5         # Max seconds between GCs
-XX:ZAllocationSpikeTolerance=2   # Allocation spike handling

# Parallel GC Tuning (Throughput)
-XX:+UseParallelGC
-XX:ParallelGCThreads=8           # GC threads
-XX:MaxGCPauseMillis=500          # Target pause
-XX:GCTimeRatio=19                # 1/(1+19) = 5% GC time target
```

### Application-Level Optimizations

```java
// 1. Object Pooling (for expensive objects)
public class ConnectionPool {
    private final BlockingQueue<Connection> pool;

    public ConnectionPool(int size) {
        pool = new ArrayBlockingQueue<>(size);
        for (int i = 0; i < size; i++) {
            pool.offer(createConnection());
        }
    }

    public Connection borrow() throws InterruptedException {
        return pool.take();
    }

    public void release(Connection conn) {
        pool.offer(conn);
    }
}

// 2. Avoid creating unnecessary objects
// Bad:
public boolean isValidId(String id) {
    return id.matches("\\d+");  // Creates Pattern every time!
}

// Good:
private static final Pattern ID_PATTERN = Pattern.compile("\\d+");
public boolean isValidId(String id) {
    return ID_PATTERN.matcher(id).matches();
}

// 3. Use primitives instead of wrappers
// Bad:
Long sum = 0L;
for (long i = 0; i < 1_000_000; i++) {
    sum += i;  // Auto-boxing overhead
}

// Good:
long sum = 0L;
for (long i = 0; i < 1_000_000; i++) {
    sum += i;
}

// 4. StringBuilder for string concatenation
// Bad:
String result = "";
for (String s : list) {
    result += s;  // Creates new String each iteration
}

// Good:
StringBuilder sb = new StringBuilder();
for (String s : list) {
    sb.append(s);
}
String result = sb.toString();

// 5. Right-size collections
// Bad:
List<User> users = new ArrayList<>();  // Default capacity 10, grows

// Good:
List<User> users = new ArrayList<>(expectedSize);

// 6. Use lazy initialization
// Eager (always allocated):
private final ExpensiveObject resource = new ExpensiveObject();

// Lazy (allocated on demand):
private ExpensiveObject resource;
public ExpensiveObject getResource() {
    if (resource == null) {
        resource = new ExpensiveObject();
    }
    return resource;
}
```

---

## Profiling and Diagnostics

### Essential JVM Tools

```bash
# 1. jps - List Java processes
jps -l -v

# 2. jstat - GC statistics
jstat -gc <pid> 1000     # Every 1 second
jstat -gcutil <pid> 1000 # Utilization percentages
jstat -gccause <pid>     # GC cause

# 3. jstack - Thread dump
jstack <pid>              # Print thread dump
jstack -l <pid>           # Include locks
jstack -F <pid>           # Force dump (hung process)

# 4. jmap - Memory map
jmap -heap <pid>          # Heap summary
jmap -histo <pid>         # Object histogram
jmap -histo:live <pid>    # Only live objects (triggers GC)
jmap -dump:format=b,file=heap.hprof <pid>  # Heap dump

# 5. jcmd - Unified diagnostic commands
jcmd <pid> help           # List available commands
jcmd <pid> VM.flags       # All JVM flags
jcmd <pid> VM.system_properties  # System properties
jcmd <pid> GC.heap_info   # Heap information
jcmd <pid> GC.run         # Trigger GC
jcmd <pid> Thread.print   # Thread dump
jcmd <pid> VM.native_memory summary  # Native memory usage

# 6. jinfo - Configuration info
jinfo -flags <pid>        # All flags
jinfo -sysprops <pid>     # System properties

# 7. jfr - Java Flight Recorder
jcmd <pid> JFR.start duration=60s filename=recording.jfr
jcmd <pid> JFR.dump name=1 filename=dump.jfr
jcmd <pid> JFR.stop name=1
```

### Reading Thread Dumps

```
"http-nio-8080-exec-1" #25 daemon prio=5 os_prio=0
    tid=0x00007f8a3c001800 nid=0x1a03 waiting on condition [0x00007f8a1c5fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000006c0a85c10> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:103)

Thread States:
┌─────────────────────────────────────────────────────────────────┐
│ State              │ Description                                │
├─────────────────────────────────────────────────────────────────┤
│ NEW                │ Not yet started                            │
│ RUNNABLE           │ Running or ready to run                    │
│ BLOCKED            │ Waiting for monitor lock                   │
│ WAITING            │ wait(), join(), park()                     │
│ TIMED_WAITING      │ sleep(), wait(timeout), join(timeout)     │
│ TERMINATED         │ Completed execution                        │
└─────────────────────────────────────────────────────────────────┘

Deadlock Detection:
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f8a3c003100 (object 0x000000076b9a0f20),
  which is held by "Thread-2"
"Thread-2":
  waiting to lock monitor 0x00007f8a3c003200 (object 0x000000076b9a0f30),
  which is held by "Thread-1"
```

### Analyzing Heap Dumps

```bash
# Generate heap dump
jmap -dump:format=b,file=heap.hprof <pid>

# Or on OutOfMemoryError
java -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/path/to/dumps \
     -jar app.jar

# Analyze with Eclipse MAT (Memory Analyzer Tool)
# Key reports:
# 1. Leak Suspects - Identifies potential memory leaks
# 2. Dominator Tree - Shows memory by object ownership
# 3. Histogram - Object counts and sizes by class
# 4. Path to GC Roots - Shows why object is retained
```

### GC Log Analysis

```bash
# Enable GC logging (Java 9+)
-Xlog:gc*:file=gc.log:time,uptime,level,tags

# Sample G1 GC log entry:
[2024-01-15T10:30:45.123+0000][25.456s][info][gc]
    GC(45) Pause Young (Normal) (G1 Evacuation Pause)
    45M->12M(256M) 12.345ms

# Understanding:
# GC(45)      - 45th GC event
# Pause Young - Young generation collection
# 45M->12M    - Heap usage before->after
# (256M)      - Total heap size
# 12.345ms    - Pause time

# Use tools for analysis:
# - GCViewer
# - GCEasy.io (online)
# - HP jmeter GC log analyzer
```

---

## Common JVM Issues

### OutOfMemoryError Types and Solutions

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     OutOfMemoryError Types                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. Java heap space                                                     │
│     Cause: Heap exhausted                                               │
│     Solutions:                                                          │
│     - Increase -Xmx                                                     │
│     - Find and fix memory leaks                                         │
│     - Optimize object creation                                          │
│                                                                          │
│  2. GC overhead limit exceeded                                          │
│     Cause: 98% time in GC, <2% heap recovered                          │
│     Solutions:                                                          │
│     - Fix memory leak (most likely cause)                               │
│     - Increase heap size                                                │
│     - Disable limit: -XX:-UseGCOverheadLimit (not recommended)         │
│                                                                          │
│  3. Metaspace                                                           │
│     Cause: Class metadata space exhausted                               │
│     Solutions:                                                          │
│     - Increase -XX:MaxMetaspaceSize                                     │
│     - Fix classloader leaks                                             │
│     - Reduce number of loaded classes                                   │
│                                                                          │
│  4. Unable to create new native thread                                  │
│     Cause: OS thread limit reached                                      │
│     Solutions:                                                          │
│     - Reduce thread count in application                                │
│     - Decrease -Xss (thread stack size)                                │
│     - Increase OS thread limits (ulimit -u)                            │
│                                                                          │
│  5. Direct buffer memory                                                │
│     Cause: NIO direct buffers exhausted                                 │
│     Solutions:                                                          │
│     - Increase -XX:MaxDirectMemorySize                                  │
│     - Properly release direct buffers                                   │
│     - Use heap buffers instead                                          │
│                                                                          │
│  6. Map failed                                                          │
│     Cause: Memory-mapped file failed                                    │
│     Solutions:                                                          │
│     - Increase OS virtual memory limits                                 │
│     - Reduce mapped file sizes                                          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Diagnosing High CPU Usage

```java
// Steps to diagnose:
// 1. Find the Java process
// $ top -c | grep java
// PID 12345, CPU 95%

// 2. Find thread consuming CPU
// $ top -H -p 12345
// Thread 12378, CPU 90%

// 3. Convert to hex
// $ printf "%x\n" 12378
// 305a

// 4. Take thread dump
// $ jstack 12345 | grep -A 50 "nid=0x305a"

// Common causes:
// - Infinite loop
// - Expensive computation
// - Excessive GC (check with jstat -gc)
// - Lock contention (check thread dump for BLOCKED)

// Example problematic code:
public void infiniteLoop() {
    while (true) {
        // No sleep, yields 100% CPU
    }
}

// Fix:
public void betterLoop() {
    while (running) {
        // Do work
        Thread.sleep(100);  // Yield CPU
    }
}
```

### Deadlock Detection and Prevention

```java
// Deadlock Example
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void method1() {
        synchronized (lock1) {           // Acquire lock1
            sleep(100);
            synchronized (lock2) {       // Wait for lock2
                // Never reached
            }
        }
    }

    public void method2() {
        synchronized (lock2) {           // Acquire lock2
            sleep(100);
            synchronized (lock1) {       // Wait for lock1
                // Never reached
            }
        }
    }
}

// Prevention Strategies:

// 1. Lock Ordering - Always acquire locks in same order
public void method1Fixed() {
    synchronized (lock1) {              // Always lock1 first
        synchronized (lock2) {
            // Safe
        }
    }
}

public void method2Fixed() {
    synchronized (lock1) {              // Same order
        synchronized (lock2) {
            // Safe
        }
    }
}

// 2. Try-Lock with Timeout
public boolean method1WithTimeout() {
    try {
        if (lock1.tryLock(1, TimeUnit.SECONDS)) {
            try {
                if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                    try {
                        // Do work
                        return true;
                    } finally {
                        lock2.unlock();
                    }
                }
            } finally {
                lock1.unlock();
            }
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
    return false;
}

// 3. Use higher-level concurrency utilities
private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
private final ConcurrentHashMap<K, V> map = new ConcurrentHashMap<>();
```

---

## Interview Questions

### Q1: Explain JVM memory areas and their purposes.

**Answer:**

| Area             | Purpose               | Shared/Per-Thread | Errors             |
| ---------------- | --------------------- | ----------------- | ------------------ |
| **Heap**         | Object storage        | Shared            | OutOfMemoryError   |
| **Metaspace**    | Class metadata        | Shared            | OutOfMemoryError   |
| **Stack**        | Method frames, locals | Per-thread        | StackOverflowError |
| **PC Register**  | Current instruction   | Per-thread        | None               |
| **Native Stack** | Native method calls   | Per-thread        | StackOverflowError |

---

### Q2: What is the difference between ClassNotFoundException and NoClassDefFoundError?

**Answer:**

| Aspect       | ClassNotFoundException       | NoClassDefFoundError                         |
| ------------ | ---------------------------- | -------------------------------------------- |
| **Type**     | Checked Exception            | Error                                        |
| **When**     | Class.forName(), loadClass() | Class was present at compile but not runtime |
| **Cause**    | Class not on classpath       | Class removed, or static initializer failed  |
| **Recovery** | Can be caught and handled    | Usually fatal                                |

```java
// ClassNotFoundException
try {
    Class.forName("com.nonexistent.Class");  // Throws CNFE
} catch (ClassNotFoundException e) {
    // Handle
}

// NoClassDefFoundError
// Compile with Util.class present
// Run without Util.class → NoClassDefFoundError
```

---

### Q3: Explain the volatile keyword and its guarantees.

**Answer:**

**Guarantees:**

1. **Visibility**: Writes are immediately visible to all threads
2. **Ordering**: Prevents reordering of reads/writes around volatile

**Does NOT guarantee:**

- Atomicity of compound operations

```java
volatile int counter = 0;

// This is NOT thread-safe:
counter++;  // Read + Increment + Write (3 operations)

// Use AtomicInteger instead:
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();  // Atomic
```

---

### Q4: What is escape analysis and how does it help?

**Answer:**
Escape analysis determines if an object escapes the method or thread.

**Optimizations enabled:**

1. **Stack allocation**: Object allocated on stack instead of heap
2. **Scalar replacement**: Object fields used directly, no object created
3. **Lock elision**: Synchronization removed for thread-local objects

```java
public long sumOfSquares(int n) {
    long sum = 0;
    for (int i = 0; i < n; i++) {
        Point p = new Point(i, i);  // Doesn't escape
        sum += p.x * p.x + p.y * p.y;
    }
    return sum;
}
// JIT can eliminate Point allocation entirely
```

---

### Q5: How do you diagnose a memory leak?

**Answer:**

**Steps:**

1. Monitor heap growth over time (should stabilize)
2. Enable GC logging, look for growing old gen
3. Take heap dumps at intervals
4. Analyze with Eclipse MAT
5. Look for:
   - Leak Suspects report
   - Growing collections (Map, List)
   - Objects with unexpected retention paths

**Common causes:**

- Static collections not cleared
- Listeners not deregistered
- ThreadLocal not removed
- Unclosed resources
- Cache without eviction

---

### Q6: What JVM flags would you use for a production application?

**Answer:**

```bash
java \
  # Memory
  -Xms4g -Xmx4g \                       # Fixed heap
  -XX:MaxMetaspaceSize=512m \           # Limit metaspace

  # GC
  -XX:+UseG1GC \                        # G1 collector
  -XX:MaxGCPauseMillis=200 \            # Pause target

  # Diagnostics
  -XX:+HeapDumpOnOutOfMemoryError \     # Dump on OOM
  -XX:HeapDumpPath=/var/dumps \
  -Xlog:gc*:file=/var/log/gc.log:time \

  # Performance
  -XX:+UseStringDeduplication \         # Save memory
  -XX:+DisableExplicitGC \              # Ignore System.gc()
  -XX:+AlwaysPreTouch \                 # Pre-touch pages

  -jar app.jar
```

---

### Q7: Explain the difference between C1 and C2 compilers.

**Answer:**

| Aspect           | C1 (Client)        | C2 (Server)        |
| ---------------- | ------------------ | ------------------ |
| **Speed**        | Fast compilation   | Slow compilation   |
| **Optimization** | Light optimization | Heavy optimization |
| **Use case**     | Quick startup      | Peak performance   |
| **Profiling**    | Basic              | Extensive          |

**Tiered Compilation** (default): Uses both

- C1 for quick initial compilation
- C2 for hot methods after profiling

---

### Q8: How does the class loading parent delegation model work?

**Answer:**

```
Load Request for class X
        │
        ▼
Application ClassLoader
        │ delegate to parent
        ▼
Platform/Extension ClassLoader
        │ delegate to parent
        ▼
Bootstrap ClassLoader
        │ try to load
        ├── Found → Return class
        └── Not found → Return to child
                            │
Platform tries to load ←────┘
        │
        ├── Found → Return class
        └── Not found → Return to child
                            │
Application tries to load ←─┘
        │
        ├── Found → Return class
        └── Not found → ClassNotFoundException
```

**Benefits:**

- Prevents core class tampering (can't replace java.lang.String)
- Ensures class uniqueness (same class from same loader)

---

### Q9: What causes StackOverflowError?

**Answer:**

**Cause:** Stack space exhausted, usually due to deep/infinite recursion.

```java
// Infinite recursion
public void infinite() {
    infinite();  // Each call adds a stack frame
}

// Deep recursion without tail-call optimization
public int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);  // Stack frame for each call
}
```

**Solutions:**

1. Fix recursion logic (base case)
2. Convert to iteration
3. Increase stack size: `-Xss2m`
4. Use tail recursion (JVM doesn't optimize, but helps readability)

---

### Q10: Explain happens-before relationship.

**Answer:**
Happens-before defines ordering guarantees between operations across threads.

**Rules:**

1. **Program order**: Within thread, earlier → later
2. **Monitor lock**: Unlock → subsequent lock
3. **Volatile**: Write → subsequent read
4. **Thread start**: start() → thread's first action
5. **Thread join**: Thread's last action → join() return
6. **Transitivity**: If A→B and B→C, then A→C

```java
// Example with volatile
volatile boolean ready = false;
int data = 0;

// Thread 1
data = 42;        // A
ready = true;     // B (volatile write)

// Thread 2
while (!ready);   // C (volatile read, waits for B)
print(data);      // D - guaranteed to see 42

// B happens-before C (volatile rule)
// A happens-before B (program order)
// C happens-before D (program order)
// Therefore: A happens-before D (data = 42 visible)
```

---

### Q11: What is JIT compilation and how does it optimize code?

**Answer:**
JIT (Just-In-Time) compiles bytecode to native code at runtime for frequently executed code.

**Optimization techniques:**

| Technique                 | Description                              |
| ------------------------- | ---------------------------------------- |
| **Method inlining**       | Replace call with method body            |
| **Escape analysis**       | Stack-allocate objects that don't escape |
| **Loop unrolling**        | Reduce loop overhead                     |
| **Dead code elimination** | Remove unreachable code                  |
| **Branch prediction**     | Optimize hot paths                       |
| **Lock elision**          | Remove unnecessary synchronization       |

```java
// Before JIT optimization
public int sum(int[] arr) {
    int sum = 0;
    for (int i = 0; i < arr.length; i++) {
        sum += arr[i];
    }
    return sum;
}

// After JIT (conceptually):
// - Array bounds check hoisted out of loop
// - Loop unrolled (process multiple elements per iteration)
// - SIMD instructions used if available
```

**JIT flags:**

```
-XX:+PrintCompilation     # Show compiled methods
-XX:CompileThreshold=10000  # Invocations before compile
-XX:+TieredCompilation    # Default in Java 8+
```

---

### Q12: What are the different class loaders and their hierarchy?

**Answer:**

```
Bootstrap ClassLoader (null)
        ↓
Platform/Extension ClassLoader
        ↓
Application/System ClassLoader
        ↓
Custom ClassLoaders
```

| ClassLoader     | Loads              | Example                  |
| --------------- | ------------------ | ------------------------ |
| **Bootstrap**   | Core Java (rt.jar) | java.lang._, java.util._ |
| **Platform**    | Extensions         | javax._, java.sql._      |
| **Application** | Classpath          | Your application classes |
| **Custom**      | Special sources    | Plugins, servlets        |

**Parent delegation:**

```java
// When loading class "com.example.MyClass":
1. ApplicationClassLoader asks Platform
2. Platform asks Bootstrap
3. Bootstrap doesn't have it
4. Platform doesn't have it
5. Application loads from classpath
```

**Why parent delegation:**

- Security: Can't replace core Java classes
- Consistency: Same class loaded once
- Isolation: Avoid conflicts

---

### Q13: What is the difference between -Xms and -Xmx, and why set them equal?

**Answer:**

```
-Xms  = Initial heap size
-Xmx  = Maximum heap size
```

**Why set them equal:**

```
-Xms2g -Xmx2g  # Recommended for production
```

**Benefits of equal values:**

1. **No runtime resizing** - Avoids GC pauses for heap growth
2. **Predictable memory** - Know exactly what app needs
3. **Performance** - No allocation/deallocation overhead
4. **GC efficiency** - Better heap layout planning

**When to differ:**

- Development environment (save memory)
- Applications with variable load
- Container with memory limits

**Other important flags:**

```
-Xmn512m          # Young generation size
-XX:MetaspaceSize=256m  # Initial metaspace
-Xss256k          # Thread stack size
```

---

### Q14: How do you analyze a thread dump?

**Answer:**

**Capturing thread dump:**

```bash
jstack <pid> > threads.txt
kill -3 <pid>    # Unix
jcmd <pid> Thread.print
```

**Thread states:**
| State | Meaning |
|-------|---------|
| RUNNABLE | Executing or ready to run |
| BLOCKED | Waiting for monitor lock |
| WAITING | Wait for notify/signal |
| TIMED_WAITING | Wait with timeout |

**Common issues to look for:**

```
// 1. DEADLOCK
"Thread-1" waiting for lock held by "Thread-2"
"Thread-2" waiting for lock held by "Thread-1"

// 2. THREAD STARVATION - Many threads BLOCKED on same lock
"Thread-1" BLOCKED waiting for <0x0000...>
"Thread-2" BLOCKED waiting for <0x0000...>
"Thread-3" BLOCKED waiting for <0x0000...>

// 3. INFINITE LOOP - RUNNABLE with high CPU
"Thread-1" RUNNABLE
   at MyClass.infiniteMethod(MyClass.java:42)
```

**Tools:**

- fastThread.io (online analyzer)
- Thread Dump Analyzer (TDA)
- IntelliJ/Eclipse built-in viewers

---

### Q15: Explain Compressed OOPs and why they matter.

**Answer:**
Compressed Ordinary Object Pointers use 32-bit references in 64-bit JVM.

**Without compression:**

- 64-bit pointers = 8 bytes per reference
- More memory, more cache misses

**With compression (default < 32GB heap):**

- 32-bit pointers = 4 bytes per reference
- Shifted and decoded on access

```
Actual address = compressed_oop << 3 + heap_base
```

**Memory savings:**

```
Object with 10 reference fields:
Without compression: 10 × 8 = 80 bytes
With compression:    10 × 4 = 40 bytes
Savings: 50%!
```

**JVM flags:**

```
-XX:+UseCompressedOops    # Default if heap < 32GB
-XX:+UseCompressedClassPointers  # Compress class pointers
```

**Important:** Keep heap under 32GB to enable compression.

---

### Q16: What is the safepoint and why does it matter?

**Answer:**
A safepoint is a point in code where all threads are stopped for GC or other JVM operations.

**Safepoint operations:**

- Garbage collection
- Code deoptimization
- Thread dump
- Heap dump
- Class redefinition

**Safepoint locations:**

- Method calls/returns
- Loop back edges (not always!)
- JNI calls

**Problem - counted loops:**

```java
// No safepoint in loop!
for (int i = 0; i < 1_000_000_000; i++) {
    // JVM can't stop this thread
}

// Solution: Use long
for (long i = 0; i < 1_000_000_000; i++) {
    // Safepoints inserted
}
```

**Monitoring:**

```
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintSafepointStatistics
```

---

### Q17: How does Method Inlining work and when is it applied?

**Answer:**
Inlining replaces method call with method body.

```java
// Before inlining
int result = add(a, b);

int add(int x, int y) { return x + y; }

// After inlining
int result = a + b;  // No call overhead!
```

**Inlining conditions:**

- Method is small (< 35 bytecodes by default)
- Method is hot (frequently called)
- Method is final, private, or effectively final
- Not too deep call chain (< 9 levels)

**JVM flags:**

```
-XX:MaxInlineSize=35       # Max bytecode size
-XX:FreqInlineSize=325     # Max for hot methods
-XX:MaxInlineLevel=9       # Max call depth
-XX:+PrintInlining         # Show inlining decisions
```

**Why it matters:**

- Eliminates call overhead
- Enables further optimizations (constant folding, etc.)
- Most important JIT optimization

---

### Q18: What tools do you use for JVM monitoring and profiling?

**Answer:**

| Tool               | Purpose             | Use Case             |
| ------------------ | ------------------- | -------------------- |
| **jps**            | List JVM processes  | Find process IDs     |
| **jstat**          | GC statistics       | Monitor memory       |
| **jstack**         | Thread dump         | Deadlock analysis    |
| **jmap**           | Heap dump           | Memory leak analysis |
| **jcmd**           | Diagnostic commands | All-in-one tool      |
| **VisualVM**       | Visual profiling    | CPU, memory, threads |
| **JFR**            | Flight Recorder     | Production profiling |
| **async-profiler** | Sampling profiler   | CPU hotspots         |

**Common commands:**

```bash
# Process list
jps -lv

# GC monitoring
jstat -gcutil <pid> 1000 10

# Thread dump
jstack <pid>

# Heap dump
jmap -dump:format=b,file=heap.hprof <pid>

# JFR recording
jcmd <pid> JFR.start duration=60s filename=recording.jfr
```

---

### Q19: Explain biased locking and why it was removed in Java 15+.

**Answer:**
Biased locking optimizes uncontended synchronization by "biasing" lock to first acquiring thread.

**How it worked:**

```java
synchronized (obj) {  // First thread: Mark as biased
    // ...
}

synchronized (obj) {  // Same thread: No atomic ops needed!
    // ...
}
```

**Why removed (deprecated Java 15, removed 17):**

1. **Complexity** - Hard to maintain
2. **Modern hardware** - Atomic operations are fast
3. **Safepoint overhead** - Revocation requires safepoint
4. **Diminishing returns** - Modern apps use concurrent utilities

**Alternative:**

```java
// Use explicit locks with tryLock()
ReentrantLock lock = new ReentrantLock();
if (lock.tryLock()) {
    try { /* ... */ }
    finally { lock.unlock(); }
}

// Or use concurrent collections
ConcurrentHashMap, CopyOnWriteArrayList
```

---

### Q20: How do you tune JVM for a Spring Boot microservice?

**Answer:**

```bash
java \
  # Memory
  -Xms512m -Xmx512m \              # Fixed heap
  -XX:MaxMetaspaceSize=128m \      # Limit metaspace

  # GC (choose one)
  -XX:+UseG1GC \                   # Balanced
  -XX:MaxGCPauseMillis=200 \       # Target pause

  # Container-aware (Java 10+)
  -XX:+UseContainerSupport \
  -XX:MaxRAMPercentage=75 \        # 75% of container memory

  # Diagnostics
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/logs/heap.hprof \
  -Xlog:gc*:file=/logs/gc.log \

  # Performance
  -XX:+AlwaysPreTouch \            # Pre-allocate heap
  -XX:+UseStringDeduplication \    # Save memory

  -jar app.jar
```

**Container-specific:**

```
# Set memory based on container limit
-XX:MaxRAMPercentage=75.0
# Leave 25% for native memory, other processes
```

---

## Key Takeaways

| Topic               | Key Point                                              |
| ------------------- | ------------------------------------------------------ |
| **Memory Areas**    | Heap (shared), Stack (per-thread), Metaspace (classes) |
| **Class Loading**   | Parent delegation prevents core class tampering        |
| **JIT**             | Tiered compilation: C1 fast, C2 optimized              |
| **Memory Model**    | Volatile for visibility, synchronized for atomicity    |
| **GC Tuning**       | Set -Xms=-Xmx, choose GC by latency/throughput needs   |
| **Diagnostics**     | jstack for threads, jmap for memory, jstat for GC      |
| **Escape Analysis** | Enables stack allocation and lock elision              |
