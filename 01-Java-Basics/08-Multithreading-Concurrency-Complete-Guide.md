# Java Multithreading and Concurrency - Complete Deep Dive

## Table of Contents

1. [Introduction to Multithreading](#introduction-to-multithreading)
2. [Creating Threads](#creating-threads)
3. [Thread Lifecycle](#thread-lifecycle)
4. [Thread Synchronization](#thread-synchronization)
5. [Inter-Thread Communication](#inter-thread-communication)
6. [Concurrency Utilities](#concurrency-utilities)
7. [Executor Framework](#executor-framework)
8. [Concurrent Collections](#concurrent-collections)
9. [Atomic Variables](#atomic-variables)
10. [Locks and Conditions](#locks-and-conditions)
11. [CompletableFuture](#completablefuture)
12. [Common Concurrency Problems](#common-concurrency-problems)
13. [Interview Questions](#interview-questions)

---

## Introduction to Multithreading

### What is a Thread?

A **thread** is the smallest unit of execution within a process. Multiple threads can exist within a single process, sharing the same memory space but executing independently.

### Process vs Thread

```
┌──────────────────────────────────────────────────────────┐
│                      PROCESS                              │
│  ┌─────────────────────────────────────────────────────┐ │
│  │                    Memory Space                      │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────────┐  │ │
│  │  │  Heap   │  │ Method  │  │   Static Variables  │  │ │
│  │  │(Shared) │  │  Area   │  │      (Shared)       │  │ │
│  │  └─────────┘  └─────────┘  └─────────────────────┘  │ │
│  └─────────────────────────────────────────────────────┘ │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐│
│  │    Thread 1    │ │    Thread 2    │ │    Thread 3    ││
│  │ ┌────────────┐ │ │ ┌────────────┐ │ │ ┌────────────┐ ││
│  │ │   Stack    │ │ │ │   Stack    │ │ │ │   Stack    │ ││
│  │ │  (Local)   │ │ │ │  (Local)   │ │ │ │  (Local)   │ ││
│  │ └────────────┘ │ │ └────────────┘ │ │ └────────────┘ ││
│  │ ┌────────────┐ │ │ ┌────────────┐ │ │ ┌────────────┐ ││
│  │ │     PC     │ │ │ │     PC     │ │ │ │     PC     │ ││
│  │ └────────────┘ │ │ └────────────┘ │ │ └────────────┘ ││
│  └────────────────┘ └────────────────┘ └────────────────┘│
└──────────────────────────────────────────────────────────┘
```

### Why Multithreading?

1. **Better resource utilization** - Use CPU while waiting for I/O
2. **Responsiveness** - UI remains responsive during long operations
3. **Parallel processing** - Utilize multi-core processors
4. **Simplified modeling** - Some problems naturally multi-threaded

### Java Memory Model (JMM)

```
Thread 1 Working Memory          Main Memory          Thread 2 Working Memory
┌─────────────────────┐     ┌─────────────────┐     ┌─────────────────────┐
│  Local Cache        │     │   Shared Vars   │     │  Local Cache        │
│  ┌───────────────┐  │     │  ┌───────────┐  │     │  ┌───────────────┐  │
│  │ x = 5 (copy)  │◄─┼─────┼──│   x = 5   │──┼─────┼─►│ x = ? (stale) │  │
│  └───────────────┘  │     │  └───────────┘  │     │  └───────────────┘  │
└─────────────────────┘     └─────────────────┘     └─────────────────────┘
         │                          ▲                         │
         │       read               │         write           │
         └──────────────────────────┴─────────────────────────┘

Without synchronization/volatile:
- Threads may see stale values
- Writes may not be visible to other threads
- Instruction reordering can occur
```

---

## Creating Threads

### Method 1: Extending Thread Class

```java
public class ExtendThread extends Thread {
    private String threadName;

    public ExtendThread(String name) {
        this.threadName = name;
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(threadName + ": " + i);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                System.out.println(threadName + " interrupted");
            }
        }
    }

    public static void main(String[] args) {
        ExtendThread t1 = new ExtendThread("Thread-1");
        ExtendThread t2 = new ExtendThread("Thread-2");

        t1.start();  // Creates new thread
        t2.start();

        // t1.run();  // Wrong! Runs in current thread
    }
}
```

### Method 2: Implementing Runnable Interface (Preferred)

```java
public class ImplementRunnable implements Runnable {
    private String threadName;

    public ImplementRunnable(String name) {
        this.threadName = name;
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(threadName + ": " + i);
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(new ImplementRunnable("Thread-1"));
        Thread t2 = new Thread(new ImplementRunnable("Thread-2"));

        t1.start();
        t2.start();

        // Using lambda (Java 8+)
        Thread t3 = new Thread(() -> {
            System.out.println("Lambda thread running");
        });
        t3.start();
    }
}
```

### Method 3: Implementing Callable Interface (Returns Value)

```java
import java.util.concurrent.*;

public class CallableExample implements Callable<Integer> {
    private int number;

    public CallableExample(int number) {
        this.number = number;
    }

    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 1; i <= number; i++) {
            sum += i;
            Thread.sleep(100);
        }
        return sum;
    }

    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        Future<Integer> future = executor.submit(new CallableExample(10));

        // Do other work...
        System.out.println("Doing other work...");

        // Get result (blocks until complete)
        Integer result = future.get();  // Can throw ExecutionException
        System.out.println("Sum: " + result);

        executor.shutdown();
    }
}
```

### Runnable vs Callable

| Feature      | Runnable             | Callable          |
| ------------ | -------------------- | ----------------- |
| Method       | run()                | call()            |
| Return value | void                 | V (generic)       |
| Exception    | Cannot throw checked | Can throw checked |
| Since        | Java 1.0             | Java 5            |

---

## Thread Lifecycle

### Thread States

```
                  ┌───────────────────────────────────────────────────┐
                  │                                                   │
                  ▼                                                   │
┌──────────┐    start()   ┌──────────────┐  scheduler  ┌───────────┐ │
│   NEW    │──────────────►│   RUNNABLE   │◄───────────►│  RUNNING  │─┼─► TERMINATED
└──────────┘              └──────────────┘             └───────────┘ │
                                 ▲                           │       │
                                 │                           │       │
                                 │   notify()                │       │
                                 │   notifyAll()             │       │
                           ┌─────┴─────┐           wait()    │       │
                           │  WAITING  │◄────────────────────┤       │
                           └───────────┘                     │       │
                                 ▲                           │       │
                                 │                           │       │
                           ┌─────┴─────┐      sleep()        │       │
                           │  TIMED    │◄────────────────────┤       │
                           │  WAITING  │      join(time)     │       │
                           └───────────┘                     │       │
                                 ▲                           │       │
                                 │                           │       │
                           ┌─────┴─────┐   synchronized      │       │
                           │  BLOCKED  │◄────────────────────┘       │
                           └───────────┘                             │
                                 │                                   │
                                 └───────────────────────────────────┘
```

### Thread States Code Example

```java
public class ThreadStates {
    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();

        Thread thread = new Thread(() -> {
            synchronized (lock) {
                try {
                    lock.wait();  // WAITING state
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        System.out.println("State after creation: " + thread.getState());  // NEW

        thread.start();
        System.out.println("State after start: " + thread.getState());     // RUNNABLE

        Thread.sleep(100);
        System.out.println("State while waiting: " + thread.getState());   // WAITING

        synchronized (lock) {
            lock.notify();
        }

        thread.join();
        System.out.println("State after completion: " + thread.getState()); // TERMINATED
    }
}
```

### Important Thread Methods

```java
public class ThreadMethods {
    public static void main(String[] args) throws InterruptedException {
        Thread current = Thread.currentThread();

        // Thread identification
        System.out.println("Name: " + current.getName());
        System.out.println("ID: " + current.getId());
        System.out.println("Priority: " + current.getPriority());

        Thread worker = new Thread(() -> {
            System.out.println("Worker running");
            try {
                Thread.sleep(2000);  // Sleep for 2 seconds
            } catch (InterruptedException e) {
                System.out.println("Worker interrupted");
            }
        });

        worker.setName("Worker-Thread");
        worker.setPriority(Thread.MAX_PRIORITY);  // 1-10 (10 highest)
        worker.setDaemon(true);  // Daemon thread

        worker.start();

        // Check if alive
        System.out.println("Is alive: " + worker.isAlive());

        // Wait for thread to complete
        worker.join();  // Or worker.join(1000) for timeout

        // yield() - hint to scheduler to give up CPU
        Thread.yield();
    }
}
```

### Daemon Threads

```java
public class DaemonThreadExample {
    public static void main(String[] args) throws InterruptedException {
        Thread daemon = new Thread(() -> {
            while (true) {
                System.out.println("Daemon running...");
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        daemon.setDaemon(true);  // Must set before start()
        daemon.start();

        Thread.sleep(2000);  // Main thread sleeps
        System.out.println("Main thread ending");

        // When main ends, daemon also terminates
        // JVM exits when all non-daemon threads complete
    }
}
```

---

## Thread Synchronization

### Race Condition Problem

```java
public class RaceCondition {
    private int count = 0;

    public void increment() {
        count++;  // Not atomic: read → modify → write
    }

    public static void main(String[] args) throws InterruptedException {
        RaceCondition rc = new RaceCondition();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) rc.increment();
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) rc.increment();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Count: " + rc.count);  // < 20000 due to race!
    }
}
```

### synchronized Keyword

```java
public class SynchronizedExample {
    private int count = 0;

    // Synchronized instance method
    public synchronized void increment() {
        count++;
    }

    // Synchronized block
    public void incrementBlock() {
        synchronized (this) {  // Monitor is 'this' object
            count++;
        }
    }

    // Different lock object
    private final Object lock = new Object();
    public void incrementWithLock() {
        synchronized (lock) {
            count++;
        }
    }

    // Synchronized static method (locks on Class object)
    private static int staticCount = 0;
    public static synchronized void incrementStatic() {
        staticCount++;
    }

    // Equivalent to:
    public static void incrementStaticBlock() {
        synchronized (SynchronizedExample.class) {
            staticCount++;
        }
    }
}
```

### volatile Keyword

```java
public class VolatileExample {
    // Without volatile: thread may never see updated value
    private volatile boolean running = true;

    public void stop() {
        running = false;
    }

    public void run() {
        while (running) {  // Without volatile, might be infinite loop
            // Do work
        }
    }
}
```

### volatile vs synchronized

| Feature    | volatile             | synchronized      |
| ---------- | -------------------- | ----------------- |
| Visibility | Yes                  | Yes               |
| Atomicity  | No (only read/write) | Yes (block)       |
| Blocking   | No                   | Yes               |
| Deadlock   | No                   | Possible          |
| Use case   | Flags, status        | Critical sections |

```java
// volatile is NOT enough for compound actions
private volatile int count = 0;

public void increment() {
    count++;  // Still race condition! (read-modify-write)
}

// Use synchronized or AtomicInteger instead
```

### Deadlock

```java
public class DeadlockExample {
    private static final Object LOCK_A = new Object();
    private static final Object LOCK_B = new Object();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            synchronized (LOCK_A) {
                System.out.println("T1: Holding LOCK_A");
                try { Thread.sleep(100); } catch (InterruptedException e) {}

                synchronized (LOCK_B) {  // Waiting for LOCK_B
                    System.out.println("T1: Holding both locks");
                }
            }
        });

        Thread t2 = new Thread(() -> {
            synchronized (LOCK_B) {
                System.out.println("T2: Holding LOCK_B");
                try { Thread.sleep(100); } catch (InterruptedException e) {}

                synchronized (LOCK_A) {  // Waiting for LOCK_A - DEADLOCK!
                    System.out.println("T2: Holding both locks");
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

### Preventing Deadlock

```java
public class DeadlockPrevention {
    private static final Object LOCK_A = new Object();
    private static final Object LOCK_B = new Object();

    // Solution 1: Consistent lock ordering
    public void method1() {
        synchronized (LOCK_A) {
            synchronized (LOCK_B) {
                // Work
            }
        }
    }

    public void method2() {
        synchronized (LOCK_A) {  // Same order as method1
            synchronized (LOCK_B) {
                // Work
            }
        }
    }

    // Solution 2: Lock timeout with tryLock()
    private final ReentrantLock lockA = new ReentrantLock();
    private final ReentrantLock lockB = new ReentrantLock();

    public void safeMethod() throws InterruptedException {
        boolean gotBothLocks = false;
        while (!gotBothLocks) {
            if (lockA.tryLock(100, TimeUnit.MILLISECONDS)) {
                try {
                    if (lockB.tryLock(100, TimeUnit.MILLISECONDS)) {
                        try {
                            gotBothLocks = true;
                            // Work with both locks
                        } finally {
                            lockB.unlock();
                        }
                    }
                } finally {
                    if (!gotBothLocks) {
                        lockA.unlock();
                    }
                }
            }
        }
    }
}
```

---

## Inter-Thread Communication

### wait(), notify(), notifyAll()

```java
public class ProducerConsumer {
    private final List<Integer> buffer = new ArrayList<>();
    private final int capacity = 5;

    public synchronized void produce(int item) throws InterruptedException {
        while (buffer.size() == capacity) {
            wait();  // Wait until space available
        }
        buffer.add(item);
        System.out.println("Produced: " + item);
        notifyAll();  // Notify consumers
    }

    public synchronized int consume() throws InterruptedException {
        while (buffer.isEmpty()) {
            wait();  // Wait until item available
        }
        int item = buffer.remove(0);
        System.out.println("Consumed: " + item);
        notifyAll();  // Notify producers
        return item;
    }

    public static void main(String[] args) {
        ProducerConsumer pc = new ProducerConsumer();

        Thread producer = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    pc.produce(i);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        Thread consumer = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    pc.consume();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        producer.start();
        consumer.start();
    }
}
```

### Important Rules for wait/notify

```java
// 1. Must be called within synchronized block
synchronized (lock) {
    lock.wait();
    lock.notify();
}

// 2. Always use while loop, not if
synchronized (lock) {
    while (!condition) {  // NOT if (!condition)
        lock.wait();
    }
    // Process
}

// 3. Prefer notifyAll() over notify()
// notify() wakes one random thread
// notifyAll() wakes all waiting threads (safer)
```

---

## Concurrency Utilities

### CountDownLatch

```java
public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        int numberOfServices = 3;
        CountDownLatch latch = new CountDownLatch(numberOfServices);

        // Start multiple services
        for (int i = 0; i < numberOfServices; i++) {
            final int serviceId = i;
            new Thread(() -> {
                try {
                    System.out.println("Service " + serviceId + " starting...");
                    Thread.sleep(1000 * (serviceId + 1));
                    System.out.println("Service " + serviceId + " ready");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    latch.countDown();  // Decrement count
                }
            }).start();
        }

        System.out.println("Waiting for all services...");
        latch.await();  // Wait until count reaches 0
        System.out.println("All services ready! Starting main application...");
    }
}
```

### CyclicBarrier

```java
public class CyclicBarrierExample {
    public static void main(String[] args) {
        int parties = 3;
        CyclicBarrier barrier = new CyclicBarrier(parties, () -> {
            System.out.println("All parties arrived! Proceeding...");
        });

        for (int i = 0; i < parties; i++) {
            final int id = i;
            new Thread(() -> {
                try {
                    System.out.println("Party " + id + " doing work...");
                    Thread.sleep(1000 * (id + 1));
                    System.out.println("Party " + id + " waiting at barrier");

                    barrier.await();  // Wait for all parties

                    System.out.println("Party " + id + " passed barrier");
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

### Semaphore

```java
public class SemaphoreExample {
    public static void main(String[] args) {
        // Limit concurrent access to 3
        Semaphore semaphore = new Semaphore(3);

        for (int i = 0; i < 10; i++) {
            final int id = i;
            new Thread(() -> {
                try {
                    System.out.println("Thread " + id + " waiting for permit");
                    semaphore.acquire();  // Get permit

                    System.out.println("Thread " + id + " acquired permit");
                    Thread.sleep(2000);  // Do work

                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    System.out.println("Thread " + id + " releasing permit");
                    semaphore.release();  // Release permit
                }
            }).start();
        }
    }
}
```

### CountDownLatch vs CyclicBarrier

| Feature    | CountDownLatch  | CyclicBarrier      |
| ---------- | --------------- | ------------------ |
| Reusable   | No (one-time)   | Yes (reusable)     |
| Count down | countDown()     | await()            |
| Wait       | await()         | await()            |
| Action     | None            | Barrier action     |
| Use case   | Wait for events | Coordinate threads |

---

## Executor Framework

### ExecutorService

```java
public class ExecutorExample {
    public static void main(String[] args) throws Exception {
        // Different executor types
        ExecutorService singleThread = Executors.newSingleThreadExecutor();
        ExecutorService fixedPool = Executors.newFixedThreadPool(4);
        ExecutorService cachedPool = Executors.newCachedThreadPool();
        ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);

        try {
            // Submit Runnable
            fixedPool.submit(() -> {
                System.out.println("Running task");
            });

            // Submit Callable, get Future
            Future<Integer> future = fixedPool.submit(() -> {
                Thread.sleep(1000);
                return 42;
            });

            System.out.println("Result: " + future.get());  // Blocks until done
            System.out.println("Result with timeout: " + future.get(2, TimeUnit.SECONDS));

            // Execute multiple tasks
            List<Callable<String>> tasks = Arrays.asList(
                () -> { Thread.sleep(500); return "Task 1"; },
                () -> { Thread.sleep(300); return "Task 2"; },
                () -> { Thread.sleep(400); return "Task 3"; }
            );

            // invokeAll - wait for all
            List<Future<String>> results = fixedPool.invokeAll(tasks);
            for (Future<String> result : results) {
                System.out.println(result.get());
            }

            // invokeAny - return first completed
            String first = fixedPool.invokeAny(tasks);
            System.out.println("First completed: " + first);

        } finally {
            // Proper shutdown
            fixedPool.shutdown();  // Graceful shutdown
            if (!fixedPool.awaitTermination(60, TimeUnit.SECONDS)) {
                fixedPool.shutdownNow();  // Force shutdown
            }
        }
    }
}
```

### ThreadPoolExecutor Configuration

```java
public class ThreadPoolConfig {
    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2,                      // corePoolSize
            4,                      // maximumPoolSize
            60L,                    // keepAliveTime
            TimeUnit.SECONDS,       // unit
            new LinkedBlockingQueue<>(100),  // workQueue
            Executors.defaultThreadFactory(), // threadFactory
            new ThreadPoolExecutor.AbortPolicy()  // rejectionPolicy
        );

        // Core threads - always kept alive
        // Max threads - spawned when queue is full
        // Keep alive - how long idle threads wait before termination
        // Work queue - holds tasks waiting for execution
        // Rejection policy - what to do when queue is full and max threads reached

        // Rejection policies:
        // AbortPolicy - throws RejectedExecutionException (default)
        // CallerRunsPolicy - runs in calling thread
        // DiscardPolicy - silently discards
        // DiscardOldestPolicy - discards oldest task, retries
    }
}
```

### ScheduledExecutorService

```java
public class ScheduledExecutorExample {
    public static void main(String[] args) throws Exception {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

        // Schedule one-time task after delay
        ScheduledFuture<?> oneTime = scheduler.schedule(
            () -> System.out.println("One-time task"),
            5,
            TimeUnit.SECONDS
        );

        // Schedule at fixed rate (every 2 seconds)
        // If task takes longer, next starts immediately after
        ScheduledFuture<?> fixedRate = scheduler.scheduleAtFixedRate(
            () -> System.out.println("Fixed rate: " + System.currentTimeMillis()),
            0,      // initial delay
            2,      // period
            TimeUnit.SECONDS
        );

        // Schedule with fixed delay (2 seconds between end and start)
        ScheduledFuture<?> fixedDelay = scheduler.scheduleWithFixedDelay(
            () -> System.out.println("Fixed delay: " + System.currentTimeMillis()),
            0,      // initial delay
            2,      // delay between completion and next start
            TimeUnit.SECONDS
        );

        // Let it run for a while
        Thread.sleep(10000);

        // Cancel scheduled tasks
        fixedRate.cancel(false);
        fixedDelay.cancel(false);

        scheduler.shutdown();
    }
}
```

---

## Concurrent Collections

### ConcurrentHashMap

```java
public class ConcurrentHashMapExample {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

        // Basic operations (thread-safe)
        map.put("A", 1);
        map.get("A");
        map.remove("A");

        // Atomic compound operations
        map.putIfAbsent("B", 2);
        map.computeIfAbsent("C", k -> k.length());
        map.computeIfPresent("B", (k, v) -> v + 1);
        map.compute("D", (k, v) -> v == null ? 1 : v + 1);
        map.merge("B", 10, Integer::sum);

        // Bulk operations (Java 8+)
        map.forEach(1, (k, v) -> System.out.println(k + ": " + v));

        long count = map.reduceValues(1, v -> (long) v, Long::sum);

        // Search (returns null if not found)
        String result = map.search(1, (k, v) -> v > 5 ? k : null);

        // Note: Size might be approximate during concurrent modification
        int size = map.size();
        long mappingCount = map.mappingCount();  // More accurate
    }
}
```

### CopyOnWriteArrayList

```java
public class CopyOnWriteExample {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

        // Writes create new internal array copy
        list.add("A");
        list.add("B");

        // Safe iteration (uses snapshot)
        for (String s : list) {
            list.add("C");  // No ConcurrentModificationException
            System.out.println(s);  // Won't see "C"
        }

        // Good for: read-heavy, write-light scenarios
        // Bad for: frequent writes (expensive copy)
    }
}
```

### BlockingQueue

```java
public class BlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

        // Producer
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    queue.put("Item-" + i);  // Blocks if full
                    System.out.println("Produced: Item-" + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();

        // Consumer
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    String item = queue.take();  // Blocks if empty
                    System.out.println("Consumed: " + item);
                    Thread.sleep(500);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();
    }
}
```

### BlockingQueue Implementations

| Implementation        | Bound     | Notes                          |
| --------------------- | --------- | ------------------------------ |
| ArrayBlockingQueue    | Bounded   | Fixed capacity, fair option    |
| LinkedBlockingQueue   | Optional  | Unbounded by default           |
| PriorityBlockingQueue | Unbounded | Priority ordering              |
| SynchronousQueue      | 0         | Direct handoff                 |
| DelayQueue            | Unbounded | Elements available after delay |

---

## Atomic Variables

### AtomicInteger, AtomicLong, AtomicBoolean

```java
public class AtomicExample {
    public static void main(String[] args) throws InterruptedException {
        AtomicInteger counter = new AtomicInteger(0);

        // Atomic operations
        int value = counter.get();
        counter.set(10);

        int oldValue = counter.getAndSet(20);
        int newValue = counter.incrementAndGet();  // ++counter
        int oldAndIncr = counter.getAndIncrement();  // counter++

        int added = counter.addAndGet(5);
        int oldAndAdd = counter.getAndAdd(5);

        // Compare-and-set (CAS)
        boolean success = counter.compareAndSet(30, 40);  // If value is 30, set to 40

        // Update with function
        counter.updateAndGet(v -> v * 2);
        counter.getAndUpdate(v -> v + 10);

        // Accumulate
        counter.accumulateAndGet(10, Integer::sum);

        // Thread-safe counter
        AtomicInteger safeCounter = new AtomicInteger(0);
        Thread[] threads = new Thread[100];

        for (int i = 0; i < 100; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    safeCounter.incrementAndGet();
                }
            });
            threads[i].start();
        }

        for (Thread t : threads) {
            t.join();
        }

        System.out.println("Final count: " + safeCounter.get());  // Always 100000
    }
}
```

### AtomicReference

```java
public class AtomicReferenceExample {
    public static void main(String[] args) {
        AtomicReference<String> ref = new AtomicReference<>("initial");

        // Get and set
        String current = ref.get();
        ref.set("new value");

        // CAS
        boolean updated = ref.compareAndSet("new value", "updated");

        // Update
        ref.updateAndGet(s -> s.toUpperCase());

        // Immutable object updates
        AtomicReference<ImmutablePerson> personRef = new AtomicReference<>(
            new ImmutablePerson("John", 30)
        );

        personRef.updateAndGet(p -> new ImmutablePerson(p.name, p.age + 1));
    }

    record ImmutablePerson(String name, int age) {}
}
```

### LongAdder and LongAccumulator

```java
public class LongAdderExample {
    public static void main(String[] args) throws InterruptedException {
        // LongAdder - better than AtomicLong for high contention
        LongAdder adder = new LongAdder();

        Thread[] threads = new Thread[100];
        for (int i = 0; i < 100; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    adder.increment();  // Low contention
                }
            });
            threads[i].start();
        }

        for (Thread t : threads) {
            t.join();
        }

        System.out.println("Sum: " + adder.sum());  // 1000000

        // LongAccumulator - custom accumulation
        LongAccumulator accumulator = new LongAccumulator(Long::max, Long.MIN_VALUE);
        accumulator.accumulate(10);
        accumulator.accumulate(5);
        accumulator.accumulate(15);
        System.out.println("Max: " + accumulator.get());  // 15
    }
}
```

---

## Locks and Conditions

### ReentrantLock

```java
public class ReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock();
    private int count = 0;

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();  // Always in finally!
        }
    }

    public void tryIncrementWithTimeout() {
        try {
            if (lock.tryLock(1, TimeUnit.SECONDS)) {
                try {
                    count++;
                } finally {
                    lock.unlock();
                }
            } else {
                System.out.println("Could not acquire lock");
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // Interruptible lock
    public void interruptibleIncrement() throws InterruptedException {
        lock.lockInterruptibly();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

### ReentrantReadWriteLock

```java
public class ReadWriteLockExample {
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    private Map<String, String> data = new HashMap<>();

    // Multiple threads can read simultaneously
    public String read(String key) {
        readLock.lock();
        try {
            return data.get(key);
        } finally {
            readLock.unlock();
        }
    }

    // Only one thread can write (exclusive)
    public void write(String key, String value) {
        writeLock.lock();
        try {
            data.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }
}
```

### Condition

```java
public class ConditionExample {
    private final Lock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();

    private final Queue<Integer> queue = new LinkedList<>();
    private final int capacity = 10;

    public void produce(int item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await();  // Wait for space
            }
            queue.add(item);
            notEmpty.signal();  // Signal consumer
        } finally {
            lock.unlock();
        }
    }

    public int consume() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();  // Wait for item
            }
            int item = queue.poll();
            notFull.signal();  // Signal producer
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

### synchronized vs ReentrantLock

| Feature             | synchronized      | ReentrantLock               |
| ------------------- | ----------------- | --------------------------- |
| Lock acquisition    | Implicit          | Explicit lock()/unlock()    |
| Interruptible       | No                | Yes (lockInterruptibly())   |
| Timeout             | No                | Yes (tryLock(time))         |
| Fairness            | No                | Yes (fair mode)             |
| Multiple conditions | No (one wait-set) | Yes (multiple Conditions)   |
| Try lock            | No                | Yes (tryLock())             |
| Automatic release   | Yes               | No (must unlock in finally) |

---

## CompletableFuture

### Creating CompletableFutures

```java
public class CompletableFutureExample {
    public static void main(String[] args) throws Exception {
        // Create completed future
        CompletableFuture<String> completed = CompletableFuture.completedFuture("Done");

        // Run async (no return value)
        CompletableFuture<Void> runAsync = CompletableFuture.runAsync(() -> {
            System.out.println("Running async");
        });

        // Supply async (with return value)
        CompletableFuture<String> supplyAsync = CompletableFuture.supplyAsync(() -> {
            return "Result";
        });

        // With custom executor
        ExecutorService executor = Executors.newFixedThreadPool(4);
        CompletableFuture<String> withExecutor = CompletableFuture.supplyAsync(
            () -> "Custom executor result",
            executor
        );

        // Get result
        String result = supplyAsync.get();  // Blocking
        String resultTimeout = supplyAsync.get(1, TimeUnit.SECONDS);  // With timeout
        String resultNow = completed.getNow("default");  // Return default if not done
        String resultJoin = supplyAsync.join();  // Like get() but unchecked exception
    }
}
```

### Chaining Operations

```java
public class CompletableFutureChaining {
    public static void main(String[] args) {
        CompletableFuture.supplyAsync(() -> "Hello")
            // Transform result
            .thenApply(s -> s + " World")

            // Consume result
            .thenAccept(System.out::println)

            // Run action (no input)
            .thenRun(() -> System.out.println("Done"));

        // Async variants (run in different thread)
        CompletableFuture.supplyAsync(() -> "Hello")
            .thenApplyAsync(s -> s + " World")
            .thenAcceptAsync(System.out::println);

        // Compose (flatMap for futures)
        CompletableFuture<String> composed = CompletableFuture
            .supplyAsync(() -> "user123")
            .thenCompose(userId -> fetchUserDetails(userId));

        // Combine two futures
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

        CompletableFuture<String> combined = future1.thenCombine(
            future2,
            (s1, s2) -> s1 + " " + s2
        );
    }

    static CompletableFuture<String> fetchUserDetails(String userId) {
        return CompletableFuture.supplyAsync(() -> "Details for " + userId);
    }
}
```

### Error Handling

```java
public class CompletableFutureErrors {
    public static void main(String[] args) {
        CompletableFuture<String> future = CompletableFuture
            .supplyAsync(() -> {
                if (true) throw new RuntimeException("Error!");
                return "Success";
            })
            // Handle exception, provide recovery value
            .exceptionally(ex -> {
                System.out.println("Error: " + ex.getMessage());
                return "Default";
            });

        // Handle both result and exception
        CompletableFuture<String> handled = CompletableFuture
            .supplyAsync(() -> "Success")
            .handle((result, ex) -> {
                if (ex != null) {
                    return "Error: " + ex.getMessage();
                }
                return result;
            });

        // whenComplete (doesn't transform result)
        CompletableFuture<String> withCallback = CompletableFuture
            .supplyAsync(() -> "Success")
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    System.out.println("Failed: " + ex);
                } else {
                    System.out.println("Succeeded: " + result);
                }
            });
    }
}
```

### Combining Multiple Futures

```java
public class CombiningFutures {
    public static void main(String[] args) throws Exception {
        CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> {
            sleep(1000);
            return "Result 1";
        });
        CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> {
            sleep(500);
            return "Result 2";
        });
        CompletableFuture<String> f3 = CompletableFuture.supplyAsync(() -> {
            sleep(750);
            return "Result 3";
        });

        // Wait for all
        CompletableFuture<Void> allOf = CompletableFuture.allOf(f1, f2, f3);
        allOf.join();

        // Get all results
        List<String> results = Stream.of(f1, f2, f3)
            .map(CompletableFuture::join)
            .collect(Collectors.toList());

        // Wait for any (first to complete)
        CompletableFuture<Object> anyOf = CompletableFuture.anyOf(f1, f2, f3);
        System.out.println("First completed: " + anyOf.join());
    }

    static void sleep(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException e) {}
    }
}
```

---

## Common Concurrency Problems

### 1. Race Condition

```java
// Problem: Non-atomic compound action
private int count = 0;
public void increment() {
    count++;  // Read → Modify → Write
}

// Solution 1: synchronized
public synchronized void increment() {
    count++;
}

// Solution 2: AtomicInteger
private AtomicInteger count = new AtomicInteger(0);
public void increment() {
    count.incrementAndGet();
}
```

### 2. Visibility Problem

```java
// Problem: Visibility not guaranteed
private boolean running = true;
public void stop() { running = false; }
public void run() {
    while (running) { /* may never see update */ }
}

// Solution: volatile
private volatile boolean running = true;
```

### 3. Deadlock

```java
// Problem: Circular wait
synchronized (lockA) {
    synchronized (lockB) { }
}
// Thread 2:
synchronized (lockB) {
    synchronized (lockA) { }  // DEADLOCK
}

// Solution: Consistent lock ordering
synchronized (lockA) {
    synchronized (lockB) { }
}
// Thread 2:
synchronized (lockA) {  // Same order
    synchronized (lockB) { }
}
```

### 4. Livelock

```java
// Both threads keep yielding to each other
// Neither makes progress
// Solution: Random backoff, priority
```

### 5. Starvation

```java
// Low-priority thread never gets CPU
// Solution: Fair locks, priority inversion prevention
ReentrantLock fairLock = new ReentrantLock(true);  // Fair mode
```

---

## Interview Questions

### Q1: What is the difference between process and thread?

| Process                     | Thread                  |
| --------------------------- | ----------------------- |
| Separate memory space       | Shared memory space     |
| Heavyweight                 | Lightweight             |
| Inter-process communication | Direct shared memory    |
| Crash doesn't affect others | Crash can affect others |

---

### Q2: What is the difference between start() and run()?

- `start()` - Creates new thread, invokes run() in that thread
- `run()` - Just a regular method call in current thread

```java
Thread t = new Thread(() -> System.out.println(Thread.currentThread().getName()));
t.run();    // Prints "main"
t.start();  // Prints "Thread-0"
```

---

### Q3: Can a thread be started twice?

**No.** IllegalThreadStateException is thrown.

```java
Thread t = new Thread(() -> {});
t.start();
t.start();  // IllegalThreadStateException
```

---

### Q4: What is thread starvation and how to prevent it?

**Starvation**: Thread never gets CPU time due to other high-priority threads.

**Prevention:**

- Use fair locks: `new ReentrantLock(true)`
- Avoid long-held locks
- Use proper thread priorities

---

### Q5: Explain volatile vs synchronized.

| volatile        | synchronized           |
| --------------- | ---------------------- |
| Visibility only | Visibility + Atomicity |
| No blocking     | Blocking               |
| Single variable | Code block             |
| Read/write only | Compound actions       |

---

### Q6: What is the output?

```java
public class Test {
    private static int count = 0;

    public static void main(String[] args) throws Exception {
        Thread t1 = new Thread(() -> { for (int i = 0; i < 10000; i++) count++; });
        Thread t2 = new Thread(() -> { for (int i = 0; i < 10000; i++) count++; });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println(count);
    }
}
```

**Answer**: Less than 20000 (race condition).
Use `AtomicInteger` or `synchronized` to fix.

---

### Q7: What is deadlock? How to detect and prevent?

**Deadlock conditions:**

1. Mutual exclusion
2. Hold and wait
3. No preemption
4. Circular wait

**Prevention:**

- Consistent lock ordering
- Lock timeout (tryLock)
- Avoid nested locks

**Detection:**

- Thread dump (jstack)
- ThreadMXBean

---

### Q8: Explain thread pool benefits.

1. **Reduced overhead** - No thread creation per task
2. **Resource management** - Limit concurrent threads
3. **Improved response time** - Threads ready to execute
4. **Better system stability** - Prevents resource exhaustion

---

### Q9: What is the difference between Callable and Runnable?

| Runnable              | Callable               |
| --------------------- | ---------------------- |
| run() method          | call() method          |
| No return value       | Returns value (Future) |
| No checked exceptions | Can throw exceptions   |

---

### Q10: What are the different ways to create a thread-safe singleton?

```java
// 1. Eager initialization
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    public static Singleton getInstance() { return INSTANCE; }
}

// 2. Double-checked locking
public class Singleton {
    private static volatile Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// 3. Bill Pugh (inner class)
public class Singleton {
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() { return Holder.INSTANCE; }
}

// 4. Enum (Best)
public enum Singleton {
    INSTANCE;
}
```

---

### Q11: What is the difference between wait() and sleep()?

| wait()                         | sleep()             |
| ------------------------------ | ------------------- |
| Releases lock                  | Retains lock        |
| Object method                  | Thread method       |
| Must be in synchronized        | Can be anywhere     |
| Wakes on notify/notifyAll      | Wakes after time    |
| For inter-thread communication | For pause execution |

```java
// wait() - releases lock
synchronized (lock) {
    while (!condition) {
        lock.wait();  // Releases lock, waits for notify
    }
}

// sleep() - keeps lock
synchronized (lock) {
    Thread.sleep(1000);  // Still holds lock!
}
```

---

### Q12: What is ThreadLocal and when to use it?

**Answer:**
ThreadLocal provides thread-confined variables. Each thread has its own copy.

```java
public class UserContext {
    private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

    public static void setUser(User user) {
        currentUser.set(user);
    }

    public static User getUser() {
        return currentUser.get();
    }

    public static void clear() {
        currentUser.remove();  // Important! Prevent memory leaks
    }
}

// Usage
UserContext.setUser(loggedInUser);
try {
    // Access user anywhere in this thread
    User user = UserContext.getUser();
} finally {
    UserContext.clear();  // Always clean up!
}
```

**Use cases:**

- User session/context
- Database connections
- Transaction management
- SimpleDateFormat (not thread-safe)

---

### Q13: What is the difference between notify() and notifyAll()?

| notify()                 | notifyAll()               |
| ------------------------ | ------------------------- |
| Wakes one waiting thread | Wakes all waiting threads |
| Arbitrary selection      | All compete for lock      |
| Risk of missed signals   | Safer but less efficient  |

```java
// Producer-Consumer
synchronized (queue) {
    while (queue.isEmpty()) {
        queue.wait();
    }
    item = queue.poll();
    queue.notifyAll();  // Wake all consumers
}
```

**Best practice:** Use `notifyAll()` unless you're certain only one thread should wake up.

---

### Q14: What is the Java Memory Model (JMM)?

**Answer:**
JMM defines how threads interact through memory and what behaviors are allowed.

**Key concepts:**

1. **Visibility:** Changes by one thread may not be visible to others
2. **Ordering:** Instructions may be reordered
3. **Happens-before:** Guarantees ordering relationships

```java
// Without synchronization
class SharedData {
    boolean flag = false;
    int value = 0;

    void writer() {
        value = 42;      // May be reordered
        flag = true;
    }

    void reader() {
        if (flag) {
            // value might still be 0!
        }
    }
}

// With volatile
class SharedData {
    volatile boolean flag = false;
    int value = 0;

    void writer() {
        value = 42;      // Happens-before flag write
        flag = true;
    }

    void reader() {
        if (flag) {
            // value is guaranteed to be 42
        }
    }
}
```

---

### Q15: What is CompletableFuture and how to use it?

**Answer:**
CompletableFuture (Java 8+) enables asynchronous, non-blocking programming.

```java
// Simple async execution
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return fetchDataFromAPI();
});

// Chaining
CompletableFuture.supplyAsync(() -> getUserId())
    .thenApply(id -> getUser(id))             // Transform
    .thenAccept(user -> sendEmail(user))      // Consume
    .exceptionally(ex -> {                    // Handle error
        log.error("Failed", ex);
        return null;
    });

// Combining
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2)
       .thenAccept(System.out::println);  // "Hello World"

// Wait for all
CompletableFuture.allOf(future1, future2).join();

// Wait for any
CompletableFuture.anyOf(future1, future2).join();
```

---

### Q16: What is the difference between ReentrantLock and synchronized?

| synchronized          | ReentrantLock              |
| --------------------- | -------------------------- |
| Built-in keyword      | Explicit class             |
| Auto unlock           | Manual unlock (in finally) |
| No fairness           | Fairness option            |
| No tryLock            | Has tryLock                |
| No interruptible lock | lockInterruptibly()        |

```java
// synchronized
synchronized (lock) {
    // Critical section
}

// ReentrantLock
ReentrantLock lock = new ReentrantLock(true);  // Fair mode
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();
}

// Advanced features
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        // Got lock within 1 second
    } finally {
        lock.unlock();
    }
}
```

---

### Q17: What is a CountDownLatch?

**Answer:**
CountDownLatch allows threads to wait for a count to reach zero.

```java
CountDownLatch latch = new CountDownLatch(3);

// Worker threads
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        doWork();
        latch.countDown();  // Decrement count
    }).start();
}

// Main thread waits
latch.await();  // Blocks until count reaches 0
System.out.println("All workers done!");

// Timeout version
if (latch.await(5, TimeUnit.SECONDS)) {
    System.out.println("All done");
} else {
    System.out.println("Timeout!");
}
```

**Use cases:**

- Wait for services to start
- Parallel test initialization
- Batch processing synchronization

---

### Q18: What is CyclicBarrier?

**Answer:**
CyclicBarrier allows threads to wait for each other at a barrier point.

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("All threads reached barrier!");
});

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        System.out.println("Working...");
        barrier.await();  // Wait for all threads
        System.out.println("Continuing...");
    }).start();
}
```

| CountDownLatch                    | CyclicBarrier               |
| --------------------------------- | --------------------------- |
| One-time use                      | Reusable                    |
| Count down from N                 | Wait for N parties          |
| No barrier action                 | Optional barrier action     |
| Threads don't wait for each other | Threads wait for each other |

---

### Q19: What is Semaphore?

**Answer:**
Semaphore controls access to a limited number of resources.

```java
// Connection pool with max 3 connections
Semaphore semaphore = new Semaphore(3);

void getConnection() {
    semaphore.acquire();  // Get permit (blocks if none available)
    try {
        useConnection();
    } finally {
        semaphore.release();  // Return permit
    }
}

// Non-blocking version
if (semaphore.tryAcquire(1, TimeUnit.SECONDS)) {
    try {
        useConnection();
    } finally {
        semaphore.release();
    }
} else {
    System.out.println("No connection available");
}
```

---

### Q20: What is the ForkJoinPool?

**Answer:**
ForkJoinPool is designed for divide-and-conquer parallel processing.

```java
// RecursiveTask returns a value
class SumTask extends RecursiveTask<Long> {
    private final int[] array;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // Base case: compute directly
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        }

        // Divide
        int mid = (start + end) / 2;
        SumTask left = new SumTask(array, start, mid);
        SumTask right = new SumTask(array, mid, end);

        left.fork();  // Execute asynchronously
        long rightResult = right.compute();  // Execute directly
        long leftResult = left.join();  // Wait for fork

        return leftResult + rightResult;
    }
}

// Usage
ForkJoinPool pool = ForkJoinPool.commonPool();
long sum = pool.invoke(new SumTask(array, 0, array.length));
```

---

## Key Takeaways

1. **Prefer Runnable over Thread** - More flexible, can extend other classes
2. **Use volatile for flags** - Ensures visibility
3. **Use synchronized for compound actions** - Atomicity needed
4. **Prefer concurrent collections** - Better scalability
5. **Use ExecutorService** - Don't create threads manually
6. **Always unlock in finally** - Prevents deadlock
7. **Use AtomicXxx for counters** - Lock-free performance
8. **CompletableFuture for async** - Modern approach
9. **Avoid nested locks** - Deadlock prevention
10. **Thread pools over thread-per-task** - Resource management
