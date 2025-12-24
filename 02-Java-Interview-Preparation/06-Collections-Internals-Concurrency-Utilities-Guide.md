# Java Collections Internals & Concurrency Utilities (java.util.concurrent) â€” Complete Interview Guide

## Audience

Senior Java developers / Application Leads targeting product + enterprise interviews.

## Table of Contents

1. [Big Picture](#big-picture)
2. [Core Contracts: equals/hashCode/compareTo](#core-contracts-equalshashcodecompareto)
3. [ArrayList vs LinkedList](#arraylist-vs-linkedlist)
4. [HashMap Internals (Java 8+)](#hashmap-internals-java-8)
5. [TreeMap / TreeSet Internals](#treemap--treeset-internals)
6. [HashSet / LinkedHashMap / LinkedHashSet](#hashset--linkedhashmap--linkedhashset)
7. [Fail-Fast vs Fail-Safe Iterators](#fail-fast-vs-fail-safe-iterators)
8. [ConcurrentHashMap Internals (Java 8+)](#concurrenthashmap-internals-java-8)
9. [CopyOnWrite Collections](#copyonwrite-collections)
10. [BlockingQueue & Producerâ€“Consumer](#blockingqueue--producerconsumer)
11. [Executors & ThreadPoolExecutor Deep Dive](#executors--threadpoolexecutor-deep-dive)
12. [CompletableFuture Essentials](#completablefuture-essentials)
13. [Locks, Atomics, and Synchronizers](#locks-atomics-and-synchronizers)
14. [Common Concurrency Bugs (and how to detect)](#common-concurrency-bugs-and-how-to-detect)
15. [Interview Questions (with crisp answers)](#interview-questions-with-crisp-answers)

---

## Big Picture

### Choose the collection by _dominant_ operations

- Frequent indexed reads: `ArrayList`
- Frequent middle insert/remove: `LinkedList` (but beware cache-unfriendly traversal)
- Uniqueness: `HashSet` / `LinkedHashSet` / `TreeSet`
- Key-value lookup: `HashMap` / `LinkedHashMap` / `TreeMap`
- Thread-safe high-throughput maps: `ConcurrentHashMap`
- Producer-consumer: `BlockingQueue`

### Complexity (typical)

| Structure           | get/contains |              add |               remove | notes                                |
| ------------------- | -----------: | ---------------: | -------------------: | ------------------------------------ |
| `ArrayList`         |       $O(1)$ | amortized $O(1)$ |               $O(n)$ | shifting costs                       |
| `LinkedList`        |       $O(n)$ |   $O(1)$ at ends | $O(1)$ with node ref | traversal dominates                  |
| `HashMap`           |   avg $O(1)$ |       avg $O(1)$ |           avg $O(1)$ | worst $O(n)$, mitigated by tree bins |
| `TreeMap`           |  $O(\log n)$ |      $O(\log n)$ |          $O(\log n)$ | sorted by comparator                 |
| `ConcurrentHashMap` |   avg $O(1)$ |       avg $O(1)$ |           avg $O(1)$ | scalable concurrency                 |

---

## Core Contracts: equals/hashCode/compareTo

### equals/hashCode rules (must know)

- If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true.
- `equals` must be: reflexive, symmetric, transitive, consistent; and `x.equals(null)` is false.

### compareTo/comparator rules

- If using `TreeMap`/`TreeSet`, ordering defines uniqueness.
- If comparator says `compare(a,b)==0`, then the structure considers them equal keys/elements.

### Common interview trap

`HashSet`/`HashMap` rely on `hashCode` for bucket selection, then `equals` for exact match.

```java
final class User {
    private final long id;
    private final String email;

    User(long id, String email) {
        this.id = id;
        this.email = email;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User other = (User) o;
        return id == other.id;
    }

    @Override
    public int hashCode() {
        return Long.hashCode(id);
    }
}
```

**Rule of thumb:** base equality on immutable identity fields.

---

## ArrayList vs LinkedList

### ArrayList internals

- Backed by resizable array.
- Growth: typically grows by ~1.5x (implementation detail).
- `add(e)` amortized $O(1)$; `add(index,e)` shifts elements $O(n)$.

### LinkedList internals

- Doubly-linked list of nodes.
- Great for frequent insert/remove _if you already have the node reference_.
- Random access via index is $O(n)$.

### Practical guidance

- Most apps should default to `ArrayList`.
- Use `LinkedList` primarily for deque semantics (`addFirst/removeFirst`) or when list splicing is key.

---

## HashMap Internals (Java 8+)

### Data structure

- Table is an array of bins.
- Each bin holds:
  - `null` (empty)
  - a linked list of nodes
  - or a balanced tree (red-black) after treeification.

### Put flow (high level)

1. Compute `hash(key)` (spread bits).
2. Index = `(n - 1) & hash`.
3. If bin empty â†’ place node.
4. If key match in bin â†’ replace value.
5. Else append to list; possibly treeify.
6. If size exceeds threshold â†’ resize/rehash.

### Why Java 8 added tree bins

Worst-case collision lists used to allow near $O(n)$ lookups under adversarial keys. Tree bins improve worst-case to $O(\log n)$ after treeification.

### Treeification thresholds (conceptual)

- Treeify when bin gets â€œtoo longâ€ **and** table is â€œlarge enoughâ€.
- Otherwise resize first.

### Resize insights

- Resize doubles table size.
- Rehashing is expensive; performance depends on load factor and key distribution.

### Interview hotspot: mutable keys

If key fields change after insertion:

- `hashCode` changes â†’ lookup fails.
- bucket location becomes wrong.

**Avoid:** using mutable objects as map keys.

---

## TreeMap / TreeSet Internals

- Backed by Red-Black Tree.
- Sorted by natural ordering (`Comparable`) or provided `Comparator`.
- Operations: $O(\log n)$.

### Common traps

- Comparator inconsistent with equals â†’ â€œmissingâ€ elements or overwrites.
- Null keys: `TreeMap` supports null only in some cases (implementation-dependent); in practice avoid.

---

## HashSet / LinkedHashMap / LinkedHashSet

### HashSet

- Usually wraps a `HashMap` with a dummy value.

### LinkedHashMap

- HashMap + doubly linked list of entries to preserve insertion order.
- Supports access-order (LRU-ish) by moving accessed entry to tail.

#### Simple LRU using LinkedHashMap

```java
import java.util.*;

public class LruCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LruCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder=true
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

---

## Fail-Fast vs Fail-Safe Iterators

### Fail-fast (most non-concurrent collections)

- Detect structural modification (via `modCount`) and throw `ConcurrentModificationException`.
- Not guaranteed (best effort), but expected behavior.

### Fail-safe (concurrent collections)

- Iterate over a snapshot or weakly consistent view.
- No CME, but may or may not see concurrent updates.

---

## ConcurrentHashMap Internals (Java 8+)

### High-level idea

- Uses lock striping + CAS + bin-level synchronization.
- Read operations are largely lock-free.

### Put mechanics (simplified)

- If bin empty â†’ CAS insert.
- If bin non-empty â†’ synchronize on bin head, mutate list/tree.
- Resizing happens cooperatively (threads help transfer bins).

### Why it scales

- Locks are localized to a bin (not a global map lock).
- Reduced contention compared to `Collections.synchronizedMap(new HashMap<>())`.

### Donâ€™t do this

```java
// Anti-pattern: check-then-act race
if (!map.containsKey(k)) {
    map.put(k, v);
}
```

Use atomic operations:

```java
map.putIfAbsent(k, v);
map.computeIfAbsent(k, key -> expensiveCompute(key));
```

---

## CopyOnWrite Collections

### CopyOnWriteArrayList

- Writes copy the entire backing array; reads are very fast.
- Great for â€œread-mostly, write-rarelyâ€ workloads (listeners, config snapshots).
- Iterators are snapshot-based.

Trade-off: heavy memory churn on frequent writes.

---

## BlockingQueue & Producerâ€“Consumer

### Key types

- `ArrayBlockingQueue` (bounded, array-backed)
- `LinkedBlockingQueue` (optionally bounded, linked nodes)
- `PriorityBlockingQueue` (ordering)
- `DelayQueue` (scheduled-like semantics)
- `SynchronousQueue` (handoff, no capacity)

### Correct producer-consumer (bounded)

```java
import java.util.concurrent.*;

public class ProducerConsumer {
    private final BlockingQueue<String> queue = new ArrayBlockingQueue<>(100);

    public void producer() throws InterruptedException {
        queue.put("task"); // blocks if full
    }

    public String consumer() throws InterruptedException {
        return queue.take(); // blocks if empty
    }
}
```

### Interview note

Prefer `BlockingQueue` over manual `wait/notify`.

---

## Executors & ThreadPoolExecutor Deep Dive

### The _why_

Thread creation is expensive; pools reuse threads and control concurrency.

### Core knobs

- `corePoolSize`: threads kept even when idle
- `maximumPoolSize`: cap
- `keepAliveTime`: idle timeout for non-core threads (and sometimes core)
- `workQueue`: `SynchronousQueue`, bounded queue, etc.
- `RejectedExecutionHandler`: when saturated

### Common pool types

- `newFixedThreadPool(n)`: unbounded queue; can hide backpressure.
- `newCachedThreadPool()`: unbounded threads; can blow up under load.
- `newSingleThreadExecutor()`: ordering and isolation.

### Safer explicit pool (with backpressure)

```java
import java.util.concurrent.*;

public class Pools {
    public static ExecutorService ioBoundPool() {
        int core = 16;
        int max = 64;
        int queueCapacity = 1000;

        return new ThreadPoolExecutor(
            core,
            max,
            30, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(queueCapacity),
            new ThreadFactory() {
                private final ThreadFactory d = Executors.defaultThreadFactory();
                @Override public Thread newThread(Runnable r) {
                    Thread t = d.newThread(r);
                    t.setName("io-pool-" + t.getId());
                    t.setDaemon(false);
                    return t;
                }
            },
            new ThreadPoolExecutor.CallerRunsPolicy() // backpressure
        );
    }
}
```

### CallerRunsPolicy meaning

When the pool is saturated, the submitting thread executes the task â†’ slows producers â†’ backpressure.

---

## CompletableFuture Essentials

### When to use

- Compose async stages without callback hell.
- Parallelize independent work.

```java
import java.util.concurrent.*;

public class CF {
    public CompletableFuture<String> pipeline(Executor executor) {
        return CompletableFuture.supplyAsync(() -> "userId", executor)
            .thenCompose(id -> CompletableFuture.supplyAsync(() -> "profile:" + id, executor))
            .thenApply(String::toUpperCase)
            .orTimeout(500, TimeUnit.MILLISECONDS)
            .exceptionally(ex -> "FALLBACK");
    }
}
```

Interview must-know:

- `thenApply` transforms value; `thenCompose` flattens nested futures.

---

## Locks, Atomics, and Synchronizers

### ReentrantLock vs synchronized

- `ReentrantLock` supports tryLock, interruptible lock acquisition, fairness policy.
- `synchronized` is simpler and often optimized by JVM.

### Atomic types

- `AtomicInteger`, `AtomicLong`, `AtomicReference`
- Use CAS to avoid locking for simple state.

### Synchronizers

- `CountDownLatch`: one-time gate
- `CyclicBarrier`: reusable barrier
- `Semaphore`: permits
- `Phaser`: flexible barrier for dynamic parties

---

## Common Concurrency Bugs (and how to detect)

### Bugs

- Deadlock (lock ordering)
- Livelock (threads active but no progress)
- Starvation (unfair scheduling)
- Visibility issues (missing happens-before)
- Race conditions (check-then-act)

### Detection tools (JDK)

- `jstack` for thread dumps
- `jcmd <pid> Thread.print`
- Java Flight Recorder (JFR) for contention

---

## Interview Questions (with crisp answers)

### Q1: Why can HashMap become slow?

Collisions create long bins; Java 8 mitigates via tree bins. Poor `hashCode` or adversarial keys degrade performance.

```java
// Bad hashCode - all objects go to same bucket
class BadKey {
    @Override public int hashCode() { return 1; } // O(n) lookups!
}

// Good hashCode - well distributed
class GoodKey {
    private final int id;
    private final String name;
    
    @Override public int hashCode() {
        return Objects.hash(id, name); // Uses 31-based algorithm
    }
}
```

---

### Q2: What's the difference between `Collections.synchronizedMap` and `ConcurrentHashMap`?

| Aspect | synchronizedMap | ConcurrentHashMap |
|--------|-----------------|-------------------|
| Locking | Single global lock | Lock striping / CAS |
| Read concurrency | Blocked by writes | Lock-free reads |
| Iteration | Fail-fast (CME risk) | Weakly consistent |
| Null keys/values | Depends on backing map | Not allowed |
| Atomic operations | Manual sync needed | Built-in (computeIfAbsent) |

---

### Q3: Why is `CopyOnWriteArrayList` iteration safe?

Iterators hold a snapshot array; writes copy and swap.

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.addAll(List.of("a", "b", "c"));

// Safe iteration - uses snapshot
for (String s : list) {
    list.add("new"); // Creates new array, doesn't affect iteration
    System.out.println(s); // Prints: a, b, c
}
// After loop: ["a", "b", "c", "new", "new", "new"]
```

---

### Q4: What's the biggest pitfall with `ThreadPoolExecutor`?

Misconfigured queue/capacity causing unbounded memory growth or unbounded thread creation; also missing backpressure.

```java
// DANGEROUS: Unbounded queue can cause OOM
ExecutorService bad = Executors.newFixedThreadPool(10); // Uses LinkedBlockingQueue()

// SAFE: Bounded queue with rejection policy
ExecutorService safe = new ThreadPoolExecutor(
    10, 20, 60L, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(100),
    new ThreadPoolExecutor.CallerRunsPolicy() // Backpressure
);
```

---

### Q5: Why is check-then-act dangerous in concurrency?

Another thread can change state between check and action. Use atomic map ops or locks.

```java
// RACE CONDITION
if (!map.containsKey(key)) {   // Thread A checks
    // Thread B inserts here!
    map.put(key, value);       // Thread A overwrites
}

// CORRECT: Atomic operation
map.putIfAbsent(key, value);
map.computeIfAbsent(key, k -> computeValue(k));
```

---

### Q6: What does "weakly consistent iterator" mean?

It doesn't throw CME and may reflect some concurrent updates, but not guaranteed.

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);

Iterator<String> it = map.keySet().iterator();
map.put("b", 2); // May or may not be seen by iterator

while (it.hasNext()) {
    System.out.println(it.next()); // No ConcurrentModificationException
}
```

---

### Q7: When would you prefer `TreeMap` over `HashMap`?

When you need sorted keys, range queries (`subMap`), or ordered iteration.

```java
TreeMap<LocalDate, Event> events = new TreeMap<>();
events.put(LocalDate.of(2024, 1, 15), new Event("A"));
events.put(LocalDate.of(2024, 2, 20), new Event("B"));
events.put(LocalDate.of(2024, 3, 10), new Event("C"));

// Range query: events in February
SortedMap<LocalDate, Event> febEvents = events.subMap(
    LocalDate.of(2024, 2, 1),
    LocalDate.of(2024, 3, 1)
);
```

---

### Q8: Why should map keys be immutable?

Changing fields affecting `hashCode/equals` breaks retrieval and violates map invariants.

```java
class MutableKey {
    String name;
    @Override public int hashCode() { return name.hashCode(); }
    @Override public boolean equals(Object o) { 
        return o instanceof MutableKey && ((MutableKey)o).name.equals(name);
    }
}

Map<MutableKey, String> map = new HashMap<>();
MutableKey key = new MutableKey();
key.name = "original";
map.put(key, "value");

key.name = "changed"; // hashCode changes!
map.get(key);         // Returns null - lost forever!
```

---

### Q9: How does HashMap handle hash collisions in Java 8+?

Java 8 introduced tree bins to handle high-collision scenarios. When bin length exceeds TREEIFY_THRESHOLD (8) AND table size >= MIN_TREEIFY_CAPACITY (64), the bin converts from linked list to red-black tree.

| Bin Type | Threshold | Lookup Complexity |
|----------|-----------|-------------------|
| Linked List | < 8 nodes | O(n) |
| Red-Black Tree | >= 8 nodes | O(log n) |
| Untreeify | < 6 nodes | Back to list |

---

### Q10: How do you implement a thread-safe lazy singleton with double-checked locking?

```java
public class Singleton {
    private static volatile Singleton instance; // volatile is CRITICAL
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {                    // First check (no locking)
            synchronized (Singleton.class) {
                if (instance == null) {            // Second check (with lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
// Why volatile? Without it, constructor may not complete before reference is assigned
```

---

### Q11: What's the difference between `ReentrantLock` and `synchronized`?

| Feature | synchronized | ReentrantLock |
|---------|--------------|---------------|
| Lock acquisition | Implicit | Explicit lock()/unlock() |
| Try lock | No | tryLock(timeout) |
| Interruptible | No | lockInterruptibly() |
| Fairness | No control | Fair/unfair option |
| Condition variables | Single (wait/notify) | Multiple Conditions |

```java
ReentrantLock lock = new ReentrantLock(true); // fair lock
Condition notEmpty = lock.newCondition();

lock.lock();
try {
    while (count == 0) notEmpty.await();
    // consume...
} finally {
    lock.unlock();
}
```

---

### Q12: How does `LinkedHashMap` support LRU cache implementation?

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        // accessOrder=true: iteration order = access order (LRU)
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}

// Usage
LRUCache<String, String> cache = new LRUCache<>(3);
cache.put("a", "1"); cache.put("b", "2"); cache.put("c", "3");
cache.get("a");        // Access "a" - moves to end
cache.put("d", "4");   // Evicts "b" (least recently used)
```

---

### Q13: What are the differences between different `BlockingQueue` implementations?

| Implementation | Bounded | Ordering | Use Case |
|----------------|---------|----------|----------|
| `ArrayBlockingQueue` | Yes (fixed) | FIFO | Standard producer-consumer |
| `LinkedBlockingQueue` | Optional | FIFO | Higher throughput (separate locks) |
| `PriorityBlockingQueue` | No | Priority | Task scheduling |
| `SynchronousQueue` | Zero capacity | Direct handoff | Thread pools |

---

### Q14: How do you prevent deadlock in concurrent code?

```java
// Deadlock scenario:
// Thread 1: lock(A) -> lock(B)
// Thread 2: lock(B) -> lock(A)

// Solution 1: Lock ordering (always acquire in same order)
void safeMethod() {
    synchronized (LOCK_A) {  // Always A first
        synchronized (LOCK_B) { /* work */ }
    }
}

// Solution 2: tryLock with timeout
boolean acquireBoth() throws InterruptedException {
    while (true) {
        if (lock1.tryLock(100, TimeUnit.MILLISECONDS)) {
            try {
                if (lock2.tryLock(100, TimeUnit.MILLISECONDS)) {
                    return true;
                }
            } finally { lock1.unlock(); }
        }
        Thread.sleep(50); // Back off
    }
}
```

---

### Q15: How does `CompletableFuture` differ from `Future`?

| Feature | Future | CompletableFuture |
|---------|--------|-------------------|
| Completion | Cannot complete manually | complete(), completeExceptionally() |
| Chaining | No | thenApply, thenCompose, thenCombine |
| Exception handling | get() throws | exceptionally(), handle() |
| Combining | Manual | allOf(), anyOf() |

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchUser())
    .thenCompose(user -> fetchOrders(user))
    .exceptionally(ex -> "Error: " + ex.getMessage())
    .orTimeout(5, TimeUnit.SECONDS);
```

---

### Q16: What's the purpose of `Phaser` vs `CountDownLatch` vs `CyclicBarrier`?

| Synchronizer | Reusable | Dynamic Parties | Use Case |
|--------------|----------|-----------------|----------|
| `CountDownLatch` | No | No | One-time event gate |
| `CyclicBarrier` | Yes | No | Fixed threads, repeated sync |
| `Phaser` | Yes | Yes | Dynamic participants, phases |

---

### Q17: How do you implement a rate limiter using `Semaphore`?

```java
public class RateLimiter {
    private final Semaphore semaphore;
    private final ScheduledExecutorService scheduler;
    
    public RateLimiter(int permitsPerSecond) {
        this.semaphore = new Semaphore(permitsPerSecond);
        this.scheduler = Executors.newSingleThreadScheduledExecutor();
        
        scheduler.scheduleAtFixedRate(() -> {
            int toAdd = permitsPerSecond - semaphore.availablePermits();
            semaphore.release(toAdd);
        }, 1, 1, TimeUnit.SECONDS);
    }
    
    public boolean tryAcquire() { return semaphore.tryAcquire(); }
}
```

---

### Q18: What are visibility issues in concurrent code and how to fix them?

```java
// PROBLEM: Visibility issue without volatile
class Visibility {
    private boolean running = true; // Should be volatile!
    void stop() { running = false; }
    void run() {
        while (running) { } // May never see update!
    }
}

// SOLUTIONS:
// 1. volatile keyword
private volatile boolean running = true;

// 2. AtomicBoolean
private final AtomicBoolean running = new AtomicBoolean(true);

// 3. synchronized access
synchronized void stop() { running = false; }
```

---

### Q19: How do you safely publish an object in concurrent code?

```java
// UNSAFE: Object may be seen partially constructed
public static Holder holder;
public static void init() { holder = new Holder(42); }

// SAFE PUBLICATION TECHNIQUES:
// 1. Static initializer
public static final Holder holder = new Holder(42);

// 2. Volatile field
private volatile Holder holder;

// 3. Final field (immutable object)
public class SafeHolder {
    private final int value; // Safe via final
    public SafeHolder(int v) { this.value = v; }
}
```

---

### Q20: What's the difference between `parallelStream()` and using an `ExecutorService`?

| Aspect | parallelStream() | ExecutorService |
|--------|------------------|-----------------|
| Thread pool | Common ForkJoinPool | Custom pool |
| Control | Limited | Full control |
| Best for | CPU-bound, stateless | I/O-bound, custom requirements |
| Blocking ops | Blocks FJP threads | Dedicated threads |

```java
// parallelStream: Uses common ForkJoinPool
List<Result> results = data.parallelStream()
    .map(this::cpuIntensiveTask)
    .collect(Collectors.toList());

// Custom pool for I/O operations
ExecutorService ioPool = Executors.newFixedThreadPool(50);
List<CompletableFuture<Result>> futures = data.stream()
    .map(item -> CompletableFuture.supplyAsync(() -> ioOp(item), ioPool))
    .collect(Collectors.toList());
```

---
## Quick mental model (one-liners)

- `HashMap`: fast average lookup; correctness hinges on key contracts.
- `ConcurrentHashMap`: scalable shared map; prefer atomic APIs (`computeIfAbsent`).
- `BlockingQueue`: correct producer-consumer without manual coordination.
- `ThreadPoolExecutor`: concurrency + backpressure configuration is the real interview.
