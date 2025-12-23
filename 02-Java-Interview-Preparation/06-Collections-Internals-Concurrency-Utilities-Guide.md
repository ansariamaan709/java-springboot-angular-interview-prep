# Java Collections Internals & Concurrency Utilities (java.util.concurrent) — Complete Interview Guide

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
10. [BlockingQueue & Producer–Consumer](#blockingqueue--producerconsumer)
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
3. If bin empty → place node.
4. If key match in bin → replace value.
5. Else append to list; possibly treeify.
6. If size exceeds threshold → resize/rehash.

### Why Java 8 added tree bins

Worst-case collision lists used to allow near $O(n)$ lookups under adversarial keys. Tree bins improve worst-case to $O(\log n)$ after treeification.

### Treeification thresholds (conceptual)

- Treeify when bin gets “too long” **and** table is “large enough”.
- Otherwise resize first.

### Resize insights

- Resize doubles table size.
- Rehashing is expensive; performance depends on load factor and key distribution.

### Interview hotspot: mutable keys

If key fields change after insertion:

- `hashCode` changes → lookup fails.
- bucket location becomes wrong.

**Avoid:** using mutable objects as map keys.

---

## TreeMap / TreeSet Internals

- Backed by Red-Black Tree.
- Sorted by natural ordering (`Comparable`) or provided `Comparator`.
- Operations: $O(\log n)$.

### Common traps

- Comparator inconsistent with equals → “missing” elements or overwrites.
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

- If bin empty → CAS insert.
- If bin non-empty → synchronize on bin head, mutate list/tree.
- Resizing happens cooperatively (threads help transfer bins).

### Why it scales

- Locks are localized to a bin (not a global map lock).
- Reduced contention compared to `Collections.synchronizedMap(new HashMap<>())`.

### Don’t do this

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
- Great for “read-mostly, write-rarely” workloads (listeners, config snapshots).
- Iterators are snapshot-based.

Trade-off: heavy memory churn on frequent writes.

---

## BlockingQueue & Producer–Consumer

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

When the pool is saturated, the submitting thread executes the task → slows producers → backpressure.

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

1. **Why can HashMap become slow?**

   - Collisions create long bins; Java 8 mitigates via tree bins. Poor `hashCode` or adversarial keys degrade performance.

2. **What’s the difference between `Collections.synchronizedMap` and `ConcurrentHashMap`?**

   - SynchronizedMap uses a single lock → contention. CHM uses finer-grained synchronization/CAS → higher throughput.

3. **Why is `CopyOnWriteArrayList` iteration safe?**

   - Iterators hold a snapshot array; writes copy and swap.

4. **What’s the biggest pitfall with `ThreadPoolExecutor`?**

   - Misconfigured queue/capacity causing unbounded memory growth or unbounded thread creation; also missing backpressure.

5. **Why is check-then-act dangerous in concurrency?**

   - Another thread can change state between check and action. Use atomic map ops or locks.

6. **What does “weakly consistent iterator” mean?**

   - It doesn’t throw CME and may reflect some concurrent updates, but not guaranteed.

7. **When would you prefer `TreeMap` over `HashMap`?**

   - When you need sorted keys, range queries (`subMap`), or ordered iteration.

8. **Why should map keys be immutable?**
   - Changing fields affecting `hashCode/equals` breaks retrieval and violates map invariants.

---

## Quick mental model (one-liners)

- `HashMap`: fast average lookup; correctness hinges on key contracts.
- `ConcurrentHashMap`: scalable shared map; prefer atomic APIs (`computeIfAbsent`).
- `BlockingQueue`: correct producer-consumer without manual coordination.
- `ThreadPoolExecutor`: concurrency + backpressure configuration is the real interview.
