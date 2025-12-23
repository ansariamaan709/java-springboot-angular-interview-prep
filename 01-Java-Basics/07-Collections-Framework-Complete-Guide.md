# Java Collections Framework - Complete Deep Dive

## Table of Contents

1. [Collections Framework Overview](#collections-framework-overview)
2. [Collection Interface Hierarchy](#collection-interface-hierarchy)
3. [List Interface](#list-interface)
4. [Set Interface](#set-interface)
5. [Queue Interface](#queue-interface)
6. [Map Interface](#map-interface)
7. [Iterators and Iteration](#iterators-and-iteration)
8. [Collections Utility Class](#collections-utility-class)
9. [Comparable vs Comparator](#comparable-vs-comparator)
10. [Thread-Safe Collections](#thread-safe-collections)
11. [Interview Questions](#interview-questions)

---

## Collections Framework Overview

### What is the Collections Framework?

The **Java Collections Framework (JCF)** is a unified architecture for representing and manipulating collections of objects. It provides:

- **Interfaces**: Abstract data types (List, Set, Map, Queue)
- **Implementations**: Concrete classes (ArrayList, HashSet, HashMap)
- **Algorithms**: Static methods for manipulation (sort, search, shuffle)

### Why Use Collections Framework?

1. **Reduces programming effort** - Ready-to-use data structures
2. **Increases performance** - High-quality implementations
3. **Interoperability** - Common interfaces allow interchange
4. **Reduces learning effort** - Consistent API design
5. **Enables software reuse** - Standard interfaces

### Collections Framework Hierarchy

```
                    Iterable
                       │
                    Collection
        ┌──────────────┼──────────────┐
        │              │              │
       List           Set          Queue
        │              │              │
    ┌───┴───┐     ┌────┴────┐    ┌───┴───┐
ArrayList  LinkedList  HashSet  SortedSet  PriorityQueue  Deque
Vector              LinkedHashSet  TreeSet               ArrayDeque
Stack                                                   LinkedList

                      Map (separate hierarchy)
                       │
         ┌─────────────┼─────────────┐
      HashMap    LinkedHashMap    SortedMap
      Hashtable                    TreeMap
      WeakHashMap
```

---

## Collection Interface Hierarchy

### Core Interfaces

```java
// Iterable - root interface
public interface Iterable<T> {
    Iterator<T> iterator();
    default void forEach(Consumer<? super T> action) {...}
    default Spliterator<T> spliterator() {...}
}

// Collection - foundation for collections
public interface Collection<E> extends Iterable<E> {
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    void clear();
    // Java 8+ default methods
    default boolean removeIf(Predicate<? super E> filter) {...}
    default Spliterator<E> spliterator() {...}
    default Stream<E> stream() {...}
    default Stream<E> parallelStream() {...}
}
```

### Basic Collection Operations

```java
public class CollectionBasics {
    public static void main(String[] args) {
        Collection<String> collection = new ArrayList<>();

        // Adding elements
        collection.add("Apple");
        collection.addAll(Arrays.asList("Banana", "Cherry"));

        // Checking elements
        System.out.println(collection.size());          // 3
        System.out.println(collection.isEmpty());       // false
        System.out.println(collection.contains("Apple")); // true

        // Removing elements
        collection.remove("Banana");
        collection.removeIf(s -> s.startsWith("C"));    // Java 8+

        // Iteration
        for (String item : collection) {
            System.out.println(item);
        }

        // Stream operations
        collection.stream()
                  .filter(s -> s.length() > 4)
                  .forEach(System.out::println);

        // Converting to array
        String[] arr = collection.toArray(new String[0]);
        String[] arr2 = collection.toArray(String[]::new);  // Java 11+

        // Clear all
        collection.clear();
    }
}
```

---

## List Interface

### List Characteristics

- **Ordered** - Elements maintain insertion order
- **Indexed** - Access by position (0-based)
- **Duplicates allowed** - Can contain same element multiple times
- **Null allowed** - Can contain null elements

### ArrayList

```java
public class ArrayListDemo {
    public static void main(String[] args) {
        // Creation
        List<String> list = new ArrayList<>();
        List<String> withCapacity = new ArrayList<>(100);  // Initial capacity
        List<String> fromCollection = new ArrayList<>(Arrays.asList("A", "B"));

        // Adding elements
        list.add("First");              // Append at end
        list.add(0, "Zero");            // Insert at index
        list.addAll(Arrays.asList("X", "Y"));

        // Accessing elements
        String first = list.get(0);     // O(1) - random access
        int index = list.indexOf("X");  // First occurrence
        int lastIndex = list.lastIndexOf("X");

        // Modifying elements
        list.set(1, "Modified");        // Replace at index

        // Removing elements
        list.remove(0);                 // By index
        list.remove("X");               // By object (first occurrence)
        list.removeAll(Arrays.asList("Y"));

        // Sublist (view, not copy!)
        List<String> sub = list.subList(0, 2);
        sub.clear();  // Also clears from original list!

        // Java 9+ factory methods
        List<String> immutable = List.of("A", "B", "C");
        // immutable.add("D");  // UnsupportedOperationException
    }
}
```

### ArrayList Internal Working

```
ArrayList Internal Structure:
┌────────────────────────────────────────────────────────┐
│ ArrayList Object                                        │
│ ┌────────────────────────────────────────────────────┐ │
│ │ Object[] elementData (internal array)               │ │
│ │ ┌────┬────┬────┬────┬────┬────┬────┬────┬────┐    │ │
│ │ │ E0 │ E1 │ E2 │ E3 │null│null│null│null│null│    │ │
│ │ └────┴────┴────┴────┴────┴────┴────┴────┴────┘    │ │
│ │ size = 4       capacity = 9                         │ │
│ └────────────────────────────────────────────────────┘ │
│ int size = 4                                            │
│ int modCount (for fail-fast iteration)                  │
└────────────────────────────────────────────────────────┘

Growth Strategy:
- Default initial capacity: 10
- New capacity = oldCapacity + (oldCapacity >> 1) = 1.5x
```

```java
// ArrayList performance characteristics
// add(E)        - O(1) amortized (O(n) when resizing)
// add(index, E) - O(n) (shift elements)
// get(index)    - O(1) (direct array access)
// remove(index) - O(n) (shift elements)
// remove(Object)- O(n) (search + shift)
// contains()    - O(n) (linear search)
// size()        - O(1)
```

### LinkedList

```java
public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<String> list = new LinkedList<>();

        // List operations
        list.add("Middle");
        list.addFirst("First");         // O(1)
        list.addLast("Last");           // O(1)

        // Deque operations (LinkedList implements Deque)
        list.push("Stack Top");         // Same as addFirst
        list.pop();                     // Same as removeFirst

        list.offer("Queue End");        // Same as addLast
        list.poll();                    // Same as removeFirst

        // Peek operations
        String first = list.peekFirst();    // Returns null if empty
        String last = list.peekLast();

        // Get operations
        String firstGet = list.getFirst();  // Throws if empty
        String lastGet = list.getLast();

        // Remove operations
        list.removeFirst();             // O(1)
        list.removeLast();              // O(1)

        // Index-based operations (slower than ArrayList!)
        String middle = list.get(2);    // O(n) - traversal required
        list.set(2, "New");             // O(n)
    }
}
```

### LinkedList Internal Structure

```
LinkedList Internal Structure (Doubly Linked):
┌────────────────────────────────────────────────────────────┐
│ LinkedList Object                                          │
│ ┌──────────┐                                               │
│ │ first ───┼──►┌────────────────┐                          │
│ └──────────┘   │ Node           │                          │
│ ┌──────────┐   │ ┌────────────┐ │   ┌────────────────┐     │
│ │ last ────┼─┐ │ │ prev: null │ │   │ Node           │     │
│ └──────────┘ │ │ │ item: "A"  │ │   │ ┌────────────┐ │     │
│ size = 3     │ │ │ next: ─────┼─┼──►│ │ prev: ←────┼─┼─┐   │
│              │ │ └────────────┘ │   │ │ item: "B"  │ │ │   │
│              │ └────────────────┘   │ │ next: ─────┼─┼─┼─► │
│              │                      │ └────────────┘ │ │   │
│              │                      └────────────────┘ │   │
│              └─────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

### ArrayList vs LinkedList

| Operation          | ArrayList      | LinkedList          |
| ------------------ | -------------- | ------------------- |
| get(index)         | O(1) ✓         | O(n)                |
| add(end)           | O(1) amortized | O(1)                |
| add(index)         | O(n)           | O(n)\*              |
| remove(index)      | O(n)           | O(n)\*              |
| remove(first/last) | O(n)           | O(1) ✓              |
| Memory             | Less overhead  | More (node objects) |
| Cache locality     | Better         | Worse               |

\*LinkedList: O(n) to find, O(1) to insert/remove

**Best Practice**: Use ArrayList by default. Use LinkedList only when you need frequent insertions/deletions at both ends.

### Vector and Stack (Legacy)

```java
// Vector - synchronized ArrayList (legacy, avoid)
Vector<String> vector = new Vector<>();
vector.add("A");
// Use Collections.synchronizedList(new ArrayList<>()) or CopyOnWriteArrayList instead

// Stack - LIFO structure (legacy, extends Vector)
Stack<String> stack = new Stack<>();
stack.push("First");
stack.push("Second");
String top = stack.pop();   // "Second"
String peek = stack.peek(); // View without removing

// Modern alternative: ArrayDeque
Deque<String> modernStack = new ArrayDeque<>();
modernStack.push("First");
modernStack.push("Second");
modernStack.pop();
```

---

## Set Interface

### Set Characteristics

- **No duplicates** - Each element appears at most once
- **No index** - Cannot access by position
- **At most one null** - (HashSet allows one, TreeSet doesn't)

### HashSet

```java
public class HashSetDemo {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();

        // Adding elements
        set.add("Apple");
        set.add("Banana");
        set.add("Apple");  // Ignored - duplicate

        System.out.println(set.size());  // 2

        // Checking elements
        boolean contains = set.contains("Apple");  // O(1)

        // Removing elements
        set.remove("Banana");  // O(1)

        // No guaranteed order!
        for (String item : set) {
            System.out.println(item);  // Order may vary
        }

        // Null is allowed
        set.add(null);

        // Set operations
        Set<String> other = new HashSet<>(Arrays.asList("Apple", "Orange"));

        // Union
        Set<String> union = new HashSet<>(set);
        union.addAll(other);

        // Intersection
        Set<String> intersection = new HashSet<>(set);
        intersection.retainAll(other);

        // Difference
        Set<String> difference = new HashSet<>(set);
        difference.removeAll(other);
    }
}
```

### HashSet Internal Working

```
HashSet is backed by HashMap:
┌──────────────────────────────────────────────────────────┐
│ HashSet                                                  │
│ ┌──────────────────────────────────────────────────────┐ │
│ │ HashMap<E, Object> map                                │ │
│ │ ┌────────────────────────────────────────────────┐   │ │
│ │ │ Key     │ Value (PRESENT - dummy object)        │   │ │
│ │ ├─────────┼───────────────────────────────────────┤   │ │
│ │ │ "Apple" │ PRESENT                               │   │ │
│ │ │ "Banana"│ PRESENT                               │   │ │
│ │ │ "Cherry"│ PRESENT                               │   │ │
│ │ └────────────────────────────────────────────────┘   │ │
│ └──────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘

// HashSet.add() implementation:
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

### LinkedHashSet

```java
public class LinkedHashSetDemo {
    public static void main(String[] args) {
        // Maintains insertion order
        Set<String> linkedSet = new LinkedHashSet<>();

        linkedSet.add("Third");
        linkedSet.add("First");
        linkedSet.add("Second");

        // Iteration preserves insertion order
        for (String item : linkedSet) {
            System.out.println(item);  // Third, First, Second
        }

        // Slightly slower than HashSet due to linked list overhead
        // But faster iteration (predictable order)
    }
}
```

### TreeSet

```java
public class TreeSetDemo {
    public static void main(String[] args) {
        // Sorted order (natural ordering or comparator)
        Set<Integer> treeSet = new TreeSet<>();

        treeSet.add(5);
        treeSet.add(2);
        treeSet.add(8);
        treeSet.add(1);

        System.out.println(treeSet);  // [1, 2, 5, 8] - sorted!

        // NavigableSet methods
        NavigableSet<Integer> navSet = new TreeSet<>(treeSet);

        System.out.println(navSet.first());     // 1
        System.out.println(navSet.last());      // 8
        System.out.println(navSet.lower(5));    // 2 (strictly less)
        System.out.println(navSet.floor(5));    // 5 (less or equal)
        System.out.println(navSet.ceiling(5));  // 5 (greater or equal)
        System.out.println(navSet.higher(5));   // 8 (strictly greater)

        // Subsets
        SortedSet<Integer> sub = navSet.subSet(2, 6);  // [2, 5]
        SortedSet<Integer> head = navSet.headSet(5);   // [1, 2]
        SortedSet<Integer> tail = navSet.tailSet(5);   // [5, 8]

        // Reverse order
        NavigableSet<Integer> descending = navSet.descendingSet();

        // Custom comparator
        Set<String> caseInsensitive = new TreeSet<>(String.CASE_INSENSITIVE_ORDER);
        caseInsensitive.add("Apple");
        caseInsensitive.add("apple");  // Duplicate (case-insensitive)
        System.out.println(caseInsensitive.size());  // 1

        // TreeSet doesn't allow null
        // treeSet.add(null);  // NullPointerException
    }
}
```

### Set Comparison

| Feature             | HashSet          | LinkedHashSet    | TreeSet        |
| ------------------- | ---------------- | ---------------- | -------------- |
| Order               | None             | Insertion order  | Sorted         |
| add/remove/contains | O(1)             | O(1)             | O(log n)       |
| Null                | One null allowed | One null allowed | No null        |
| Underlying          | HashMap          | LinkedHashMap    | Red-Black Tree |
| Memory              | Less             | More (links)     | More (tree)    |

---

## Queue Interface

### Queue Characteristics

- **FIFO** (First-In-First-Out) by default
- **Special operations** - offer, poll, peek vs add, remove, element

### Queue Methods

| Throws Exception | Returns Special Value    |
| ---------------- | ------------------------ |
| add(e)           | offer(e) - returns false |
| remove()         | poll() - returns null    |
| element()        | peek() - returns null    |

```java
public class QueueDemo {
    public static void main(String[] args) {
        Queue<String> queue = new LinkedList<>();

        // Adding elements
        queue.add("First");      // Throws if capacity restricted
        queue.offer("Second");   // Returns false if full

        // Examining head
        String head = queue.element();  // Throws if empty
        String headSafe = queue.peek(); // Returns null if empty

        // Removing head
        String removed = queue.remove();  // Throws if empty
        String removedSafe = queue.poll(); // Returns null if empty

        // Queue size
        System.out.println(queue.size());
        System.out.println(queue.isEmpty());
    }
}
```

### PriorityQueue

```java
public class PriorityQueueDemo {
    public static void main(String[] args) {
        // Min-heap by default (natural ordering)
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();

        minHeap.offer(5);
        minHeap.offer(2);
        minHeap.offer(8);
        minHeap.offer(1);

        // Elements come out in sorted order
        while (!minHeap.isEmpty()) {
            System.out.print(minHeap.poll() + " ");  // 1 2 5 8
        }

        // Max-heap using reverseOrder comparator
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        maxHeap.addAll(Arrays.asList(5, 2, 8, 1));

        while (!maxHeap.isEmpty()) {
            System.out.print(maxHeap.poll() + " ");  // 8 5 2 1
        }

        // Custom objects with comparator
        PriorityQueue<Task> taskQueue = new PriorityQueue<>(
            Comparator.comparingInt(Task::getPriority)
        );

        taskQueue.offer(new Task("Low", 3));
        taskQueue.offer(new Task("High", 1));
        taskQueue.offer(new Task("Medium", 2));

        // High priority tasks come first
        while (!taskQueue.isEmpty()) {
            System.out.println(taskQueue.poll().getName());
            // High, Medium, Low
        }
    }
}

class Task {
    private String name;
    private int priority;

    public Task(String name, int priority) {
        this.name = name;
        this.priority = priority;
    }

    public String getName() { return name; }
    public int getPriority() { return priority; }
}
```

### Deque (Double-Ended Queue)

```java
public class DequeDemo {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();

        // Add at both ends
        deque.addFirst("First");
        deque.addLast("Last");
        deque.offerFirst("New First");
        deque.offerLast("New Last");

        // Examine both ends
        System.out.println(deque.peekFirst());  // New First
        System.out.println(deque.peekLast());   // New Last

        // Remove from both ends
        String first = deque.pollFirst();
        String last = deque.pollLast();

        // Use as Stack (LIFO)
        Deque<String> stack = new ArrayDeque<>();
        stack.push("Bottom");
        stack.push("Middle");
        stack.push("Top");

        while (!stack.isEmpty()) {
            System.out.println(stack.pop());  // Top, Middle, Bottom
        }

        // Use as Queue (FIFO)
        Deque<String> queue = new ArrayDeque<>();
        queue.offer("First");
        queue.offer("Second");

        while (!queue.isEmpty()) {
            System.out.println(queue.poll());  // First, Second
        }
    }
}
```

---

## Map Interface

### Map Characteristics

- **Key-Value pairs** - Each key maps to one value
- **Unique keys** - No duplicate keys allowed
- **One null key** (HashMap) or no null key (TreeMap)
- **Not part of Collection** - Separate hierarchy

### HashMap

```java
public class HashMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();

        // Adding entries
        map.put("Apple", 100);
        map.put("Banana", 200);
        map.put("Apple", 150);  // Replaces existing value

        // Getting values
        Integer value = map.get("Apple");         // 150
        Integer defaultVal = map.getOrDefault("Orange", 0);  // 0

        // Checking
        boolean hasKey = map.containsKey("Apple");    // true
        boolean hasValue = map.containsValue(200);    // true

        // Removing
        map.remove("Banana");
        map.remove("Apple", 100);  // Only if value matches

        // Size
        int size = map.size();
        boolean empty = map.isEmpty();

        // Iteration methods
        // 1. entrySet
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }

        // 2. keySet
        for (String key : map.keySet()) {
            System.out.println(key + ": " + map.get(key));
        }

        // 3. values
        for (Integer val : map.values()) {
            System.out.println(val);
        }

        // 4. forEach (Java 8+)
        map.forEach((k, v) -> System.out.println(k + ": " + v));

        // Java 8+ methods
        map.putIfAbsent("Orange", 300);  // Add only if absent
        map.computeIfAbsent("Grape", k -> k.length() * 10);
        map.computeIfPresent("Apple", (k, v) -> v + 10);
        map.compute("Mango", (k, v) -> v == null ? 1 : v + 1);
        map.merge("Apple", 50, Integer::sum);  // Add 50 to existing
        map.replaceAll((k, v) -> v * 2);  // Double all values
    }
}
```

### HashMap Internal Working

```
HashMap Structure (Java 8+):
┌────────────────────────────────────────────────────────────────┐
│ HashMap                                                        │
│ ┌────────────────────────────────────────────────────────────┐ │
│ │ Node<K,V>[] table (buckets)                                 │ │
│ │ ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐ │ │
│ │ │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │10 │11 │12 │...│ │ │
│ │ └─┬─┴───┴───┴─┬─┴───┴───┴─┬─┴───┴───┴───┴───┴───┴───┴───┘ │ │
│ │   │           │           │                                 │ │
│ │   ▼           ▼           ▼                                 │ │
│ │ ┌───────┐  ┌───────┐  ┌───────┐                            │ │
│ │ │K1: V1 │  │K2: V2 │  │K3: V3 │                            │ │
│ │ │ next ─┼─►│K4: V4 │  │ next ─┼─► TreeNode (if > 8)        │ │
│ │ └───────┘  │ next  │  └───────┘                            │ │
│ │            └───────┘                                        │ │
│ └────────────────────────────────────────────────────────────┘ │
│ int size                                                       │
│ float loadFactor = 0.75                                        │
│ int threshold = capacity * loadFactor                          │
└────────────────────────────────────────────────────────────────┘

Bucket selection: index = hash(key) & (table.length - 1)

Tree conversion:
- Linked list → Red-Black Tree when bucket size > 8
- Tree → Linked list when bucket size < 6
```

```java
// How put() works:
public V put(K key, V value) {
    // 1. Calculate hash
    int hash = hash(key);  // Spreads high bits to lower

    // 2. Find bucket index
    int index = hash & (table.length - 1);

    // 3. If bucket empty, create new node
    // 4. If key exists, replace value
    // 5. If collision, add to linked list/tree
    // 6. If size > threshold, resize (double capacity)
}

// Hash collision handling:
// 1. Separate chaining (linked list → tree)
// 2. Good hashCode() minimizes collisions
```

### LinkedHashMap

```java
public class LinkedHashMapDemo {
    public static void main(String[] args) {
        // Maintains insertion order
        Map<String, Integer> linkedMap = new LinkedHashMap<>();
        linkedMap.put("Third", 3);
        linkedMap.put("First", 1);
        linkedMap.put("Second", 2);

        linkedMap.forEach((k, v) -> System.out.println(k));
        // Third, First, Second (insertion order)

        // Access-order mode (LRU cache)
        Map<String, Integer> lruCache = new LinkedHashMap<>(16, 0.75f, true);
        lruCache.put("A", 1);
        lruCache.put("B", 2);
        lruCache.put("C", 3);

        lruCache.get("A");  // Access A, moves to end

        lruCache.forEach((k, v) -> System.out.print(k + " "));
        // B C A (A moved to end due to access)

        // LRU Cache with size limit
        Map<String, Integer> boundedCache = new LinkedHashMap<>(16, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, Integer> eldest) {
                return size() > 3;  // Max 3 entries
            }
        };
    }
}
```

### TreeMap

```java
public class TreeMapDemo {
    public static void main(String[] args) {
        // Sorted by keys (natural ordering)
        TreeMap<String, Integer> treeMap = new TreeMap<>();

        treeMap.put("Banana", 2);
        treeMap.put("Apple", 1);
        treeMap.put("Cherry", 3);

        // Keys are sorted
        treeMap.forEach((k, v) -> System.out.println(k));
        // Apple, Banana, Cherry

        // NavigableMap methods
        System.out.println(treeMap.firstKey());     // Apple
        System.out.println(treeMap.lastKey());      // Cherry
        System.out.println(treeMap.lowerKey("Banana"));   // Apple
        System.out.println(treeMap.higherKey("Banana"));  // Cherry

        // Submap operations
        SortedMap<String, Integer> sub = treeMap.subMap("Apple", "Cherry");
        NavigableMap<String, Integer> desc = treeMap.descendingMap();

        // Custom comparator
        TreeMap<String, Integer> reverseMap = new TreeMap<>(Collections.reverseOrder());

        // TreeMap doesn't allow null keys
        // treeMap.put(null, 0);  // NullPointerException
    }
}
```

### Map Comparison

| Feature     | HashMap    | LinkedHashMap     | TreeMap        |
| ----------- | ---------- | ----------------- | -------------- |
| Order       | None       | Insertion/Access  | Sorted         |
| get/put     | O(1)       | O(1)              | O(log n)       |
| Null keys   | One        | One               | None           |
| Null values | Multiple   | Multiple          | Multiple       |
| Thread-safe | No         | No                | No             |
| Underlying  | Hash Table | Hash + LinkedList | Red-Black Tree |

---

## Iterators and Iteration

### Iterator Interface

```java
public class IteratorDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));

        // Basic iteration
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String element = iterator.next();
            System.out.println(element);
        }

        // Remove during iteration (safe)
        iterator = list.iterator();
        while (iterator.hasNext()) {
            String element = iterator.next();
            if (element.equals("B")) {
                iterator.remove();  // Safe removal
            }
        }

        // Fail-fast behavior
        try {
            for (String s : list) {
                if (s.equals("A")) {
                    list.remove(s);  // ConcurrentModificationException!
                }
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("Cannot modify during for-each!");
        }

        // Java 8+ removeIf (safe)
        list.removeIf(s -> s.equals("C"));

        // forEachRemaining (Java 8+)
        iterator = list.iterator();
        iterator.forEachRemaining(System.out::println);
    }
}
```

### ListIterator

```java
public class ListIteratorDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));

        // ListIterator - bidirectional
        ListIterator<String> listIterator = list.listIterator();

        // Forward iteration
        while (listIterator.hasNext()) {
            int index = listIterator.nextIndex();
            String element = listIterator.next();
            System.out.println(index + ": " + element);
        }

        // Backward iteration
        while (listIterator.hasPrevious()) {
            int index = listIterator.previousIndex();
            String element = listIterator.previous();
            System.out.println(index + ": " + element);
        }

        // Modification during iteration
        listIterator = list.listIterator();
        while (listIterator.hasNext()) {
            String element = listIterator.next();
            if (element.equals("B")) {
                listIterator.set("B-Modified");  // Replace current
                listIterator.add("B2");           // Add after current
            }
        }
        System.out.println(list);  // [A, B-Modified, B2, C]

        // Start from specific index
        ListIterator<String> fromMiddle = list.listIterator(2);
    }
}
```

### Fail-Fast vs Fail-Safe

```java
public class FailFastVsFailSafe {
    public static void main(String[] args) {
        // Fail-Fast (ArrayList, HashMap, etc.)
        List<String> failFast = new ArrayList<>(Arrays.asList("A", "B", "C"));
        try {
            for (String s : failFast) {
                failFast.add("D");  // ConcurrentModificationException
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("Fail-fast iterator detected modification");
        }

        // Fail-Safe (CopyOnWriteArrayList, ConcurrentHashMap)
        List<String> failSafe = new CopyOnWriteArrayList<>(Arrays.asList("A", "B", "C"));
        for (String s : failSafe) {
            failSafe.add("D");  // No exception, works on snapshot
        }
        System.out.println(failSafe);  // [A, B, C, D, D, D]

        // ConcurrentHashMap - weakly consistent
        Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
        concurrentMap.put("A", 1);
        concurrentMap.put("B", 2);

        for (String key : concurrentMap.keySet()) {
            concurrentMap.put("C", 3);  // May or may not be seen
        }
    }
}
```

---

## Collections Utility Class

### Common Operations

```java
import java.util.Collections;

public class CollectionsUtility {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9));

        // Sorting
        Collections.sort(list);                    // [1, 2, 5, 8, 9]
        Collections.sort(list, Collections.reverseOrder());  // [9, 8, 5, 2, 1]

        // Searching (requires sorted list)
        Collections.sort(list);
        int index = Collections.binarySearch(list, 5);  // Returns index

        // Min/Max
        Integer min = Collections.min(list);
        Integer max = Collections.max(list);
        Integer maxByComparator = Collections.max(list, Collections.reverseOrder());

        // Shuffling
        Collections.shuffle(list);
        Collections.shuffle(list, new Random(42));  // Reproducible

        // Reversing
        Collections.reverse(list);

        // Rotating
        Collections.rotate(list, 2);  // Rotate right by 2

        // Swapping
        Collections.swap(list, 0, 4);  // Swap positions

        // Filling
        List<String> stringList = new ArrayList<>(Arrays.asList("A", "B", "C"));
        Collections.fill(stringList, "X");  // [X, X, X]

        // Copying
        List<Integer> dest = new ArrayList<>(Arrays.asList(0, 0, 0, 0, 0));
        Collections.copy(dest, list);  // dest must be >= src size

        // Frequency
        List<String> words = Arrays.asList("A", "B", "A", "C", "A");
        int frequency = Collections.frequency(words, "A");  // 3

        // Disjoint (no common elements)
        boolean disjoint = Collections.disjoint(
            Arrays.asList(1, 2, 3),
            Arrays.asList(4, 5, 6)
        );  // true

        // nCopies (immutable list)
        List<String> copies = Collections.nCopies(5, "X");  // [X, X, X, X, X]
    }
}
```

### Creating Immutable Collections

```java
public class ImmutableCollections {
    public static void main(String[] args) {
        // Collections utility wrappers
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
        List<String> unmodifiable = Collections.unmodifiableList(list);
        // unmodifiable.add("D");  // UnsupportedOperationException
        // But original list can still be modified!

        Set<String> unmodifiableSet = Collections.unmodifiableSet(new HashSet<>(list));
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(new HashMap<>());

        // Empty collections (immutable)
        List<String> emptyList = Collections.emptyList();
        Set<String> emptySet = Collections.emptySet();
        Map<String, Integer> emptyMap = Collections.emptyMap();

        // Singleton collections (immutable)
        List<String> singletonList = Collections.singletonList("Only");
        Set<String> singletonSet = Collections.singleton("Only");
        Map<String, Integer> singletonMap = Collections.singletonMap("Only", 1);

        // Java 9+ factory methods (truly immutable)
        List<String> immutableList = List.of("A", "B", "C");
        Set<String> immutableSet = Set.of("A", "B", "C");
        Map<String, Integer> immutableMap = Map.of("A", 1, "B", 2);

        // Java 10+ copyOf (immutable copy)
        List<String> immutableCopy = List.copyOf(list);
    }
}
```

### Synchronized Collections

```java
public class SynchronizedCollections {
    public static void main(String[] args) {
        // Synchronized wrappers
        List<String> syncList = Collections.synchronizedList(new ArrayList<>());
        Set<String> syncSet = Collections.synchronizedSet(new HashSet<>());
        Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());

        // Important: Iteration still needs external synchronization!
        synchronized (syncList) {
            for (String s : syncList) {
                // Safe iteration
            }
        }

        // Better alternatives for concurrent access:
        List<String> copyOnWrite = new CopyOnWriteArrayList<>();
        Set<String> concurrentSet = ConcurrentHashMap.newKeySet();
        Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
    }
}
```

---

## Comparable vs Comparator

### Comparable Interface

```java
// Comparable - natural ordering (single way)
public class Employee implements Comparable<Employee> {
    private String name;
    private int salary;
    private int age;

    public Employee(String name, int salary, int age) {
        this.name = name;
        this.salary = salary;
        this.age = age;
    }

    // Natural ordering by salary
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }

    // Getters
    public String getName() { return name; }
    public int getSalary() { return salary; }
    public int getAge() { return age; }

    @Override
    public String toString() {
        return name + "($" + salary + ", " + age + "y)";
    }
}

// Usage
public class ComparableDemo {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Alice", 50000, 30),
            new Employee("Bob", 60000, 25),
            new Employee("Charlie", 45000, 35)
        );

        Collections.sort(employees);  // Uses compareTo
        System.out.println(employees);
        // [Charlie($45000, 35y), Alice($50000, 30y), Bob($60000, 25y)]
    }
}
```

### Comparator Interface

```java
// Comparator - custom ordering (multiple ways)
public class ComparatorDemo {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Alice", 50000, 30),
            new Employee("Bob", 60000, 25),
            new Employee("Charlie", 45000, 35)
        );

        // Anonymous class comparator
        Comparator<Employee> byName = new Comparator<Employee>() {
            @Override
            public int compare(Employee e1, Employee e2) {
                return e1.getName().compareTo(e2.getName());
            }
        };

        // Lambda comparator
        Comparator<Employee> byAge = (e1, e2) -> Integer.compare(e1.getAge(), e2.getAge());

        // Method reference comparator
        Comparator<Employee> bySalary = Comparator.comparingInt(Employee::getSalary);

        // Sort by different criteria
        employees.sort(byName);
        System.out.println("By name: " + employees);

        employees.sort(byAge);
        System.out.println("By age: " + employees);

        employees.sort(bySalary.reversed());
        System.out.println("By salary (desc): " + employees);

        // Chained comparators
        Comparator<Employee> bySalaryThenAge = Comparator
            .comparingInt(Employee::getSalary)
            .thenComparingInt(Employee::getAge);

        // Handling nulls
        Comparator<Employee> nullSafe = Comparator
            .nullsFirst(Comparator.comparing(Employee::getName));

        // Reverse order
        Comparator<Employee> descending = Collections.reverseOrder(bySalary);
    }
}
```

### Comparable vs Comparator Comparison

| Feature             | Comparable             | Comparator                         |
| ------------------- | ---------------------- | ---------------------------------- |
| Package             | java.lang              | java.util                          |
| Method              | compareTo(T o)         | compare(T o1, T o2)                |
| Sorting             | Natural/default        | Custom                             |
| Class modification  | Required               | Not required                       |
| Number of orderings | One                    | Multiple                           |
| Usage               | Collections.sort(list) | Collections.sort(list, comparator) |

---

## Thread-Safe Collections

### Legacy Synchronized Collections

```java
// Vector - synchronized ArrayList
Vector<String> vector = new Vector<>();

// Hashtable - synchronized HashMap
Hashtable<String, Integer> hashtable = new Hashtable<>();

// Problems:
// 1. Coarse-grained locking (entire collection locked)
// 2. Iteration still not thread-safe
// 3. Poor performance under contention
```

### Synchronized Wrappers

```java
// Collections.synchronizedXxx()
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());

// Must synchronize during iteration!
synchronized (syncList) {
    Iterator<String> it = syncList.iterator();
    while (it.hasNext()) {
        System.out.println(it.next());
    }
}
```

### Concurrent Collections (java.util.concurrent)

```java
public class ConcurrentCollectionsDemo {
    public static void main(String[] args) {
        // ConcurrentHashMap - segment-based locking, high concurrency
        ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
        concurrentMap.put("A", 1);
        concurrentMap.putIfAbsent("B", 2);
        concurrentMap.compute("A", (k, v) -> v + 1);

        // Safe iteration (weakly consistent)
        for (Map.Entry<String, Integer> entry : concurrentMap.entrySet()) {
            // No ConcurrentModificationException
        }

        // CopyOnWriteArrayList - snapshot iteration
        CopyOnWriteArrayList<String> copyOnWriteList = new CopyOnWriteArrayList<>();
        copyOnWriteList.add("A");
        // Good for read-heavy, write-light scenarios

        // CopyOnWriteArraySet
        CopyOnWriteArraySet<String> copyOnWriteSet = new CopyOnWriteArraySet<>();

        // ConcurrentSkipListMap - concurrent sorted map
        ConcurrentSkipListMap<String, Integer> skipListMap = new ConcurrentSkipListMap<>();

        // ConcurrentSkipListSet - concurrent sorted set
        ConcurrentSkipListSet<String> skipListSet = new ConcurrentSkipListSet<>();

        // BlockingQueue implementations
        BlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(100);
        BlockingQueue<String> linkedBlockingQueue = new LinkedBlockingQueue<>();
        BlockingQueue<String> priorityBlockingQueue = new PriorityBlockingQueue<>();

        // ConcurrentLinkedQueue - non-blocking
        ConcurrentLinkedQueue<String> concurrentQueue = new ConcurrentLinkedQueue<>();
    }
}
```

---

## Interview Questions

### Q1: What is the difference between ArrayList and LinkedList?

| ArrayList                 | LinkedList                   |
| ------------------------- | ---------------------------- |
| Dynamic array             | Doubly linked list           |
| O(1) random access        | O(n) random access           |
| O(n) insert/delete middle | O(1) insert/delete at ends   |
| Better cache locality     | Poor cache locality          |
| Less memory per element   | More memory (prev/next refs) |

**Best Practice**: Use ArrayList by default.

---

### Q2: How does HashMap work internally?

**Answer:**

1. Uses array of buckets (Node[])
2. Key's hashCode() determines bucket index
3. Collisions handled by linked list (→ tree if > 8 elements)
4. Load factor 0.75 triggers resize (double capacity)
5. Java 8+: Treeification when bucket size > 8

---

### Q3: What is the difference between HashMap and Hashtable?

| HashMap                       | Hashtable              |
| ----------------------------- | ---------------------- |
| Not synchronized              | Synchronized           |
| Allows one null key           | No null keys/values    |
| Fail-fast iterator            | Enumerator + fail-fast |
| Preferred for single-threaded | Legacy, avoid          |

**Modern alternative**: ConcurrentHashMap for thread-safety.

---

### Q4: What happens if you put duplicate key in HashMap?

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("A", 2);  // Replaces value

System.out.println(map.get("A"));  // 2
System.out.println(map.size());    // 1
```

The old value is replaced; key remains the same.

---

### Q5: Why is String commonly used as HashMap key?

1. **Immutable** - hashCode won't change after insertion
2. **Caches hashCode** - computed once, reused
3. **Proper equals/hashCode** - correct behavior guaranteed

```java
// If key were mutable and changed after insertion:
// Original bucket calculation becomes invalid!
// Key cannot be found, causes memory leak
```

---

### Q6: What is the difference between fail-fast and fail-safe iterators?

| Fail-Fast                              | Fail-Safe                               |
| -------------------------------------- | --------------------------------------- |
| Throws ConcurrentModificationException | No exception                            |
| Works on original collection           | Works on clone/snapshot                 |
| ArrayList, HashMap                     | CopyOnWriteArrayList, ConcurrentHashMap |
| Detects structural modification        | Allows modification                     |

---

### Q7: How to remove elements during iteration?

```java
// Method 1: Iterator.remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("remove")) {
        it.remove();  // Safe
    }
}

// Method 2: removeIf() - Java 8+
list.removeIf(s -> s.equals("remove"));

// Method 3: Use CopyOnWriteArrayList
List<String> cowList = new CopyOnWriteArrayList<>(list);
for (String s : cowList) {
    cowList.remove(s);  // Safe but inefficient
}
```

---

### Q8: What is the output?

```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("B");
set.add("A");
set.add(null);
set.add(null);

System.out.println(set.size());
```

**Answer**: `3`

HashSet stores unique elements. "A" added twice counts once, null added twice counts once.

---

### Q9: Explain equals() and hashCode() contract.

**Contract:**

1. If `a.equals(b)`, then `a.hashCode() == b.hashCode()`
2. If hashCodes equal, equals may or may not be true
3. hashCode must be consistent during object lifetime

```java
// Violating contract breaks HashMap/HashSet!
class BadKey {
    int id;

    @Override
    public boolean equals(Object o) {
        return this.id == ((BadKey)o).id;
    }

    // Missing hashCode override!
    // Two equal objects may have different hashCodes
    // HashMap will treat them as different keys
}
```

---

### Q10: How to make a collection thread-safe?

```java
// Method 1: Synchronized wrapper
List<String> syncList = Collections.synchronizedList(new ArrayList<>());

// Method 2: Concurrent collection
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();

// Method 3: External synchronization
synchronized (list) {
    // Operations
}
```

---

### Q11: What is the difference between Collection and Collections?

| Collection                   | Collections             |
| ---------------------------- | ----------------------- |
| Interface                    | Utility class           |
| Root of collection hierarchy | Static helper methods   |
| Declares methods             | sort(), shuffle(), etc. |

---

### Q12: Which collection for each scenario?

| Scenario                          | Best Collection          |
| --------------------------------- | ------------------------ |
| Ordered, duplicates, fast access  | ArrayList                |
| Ordered, frequent add/remove ends | LinkedList               |
| Unique, fast lookup               | HashSet                  |
| Unique, sorted                    | TreeSet                  |
| Key-value, fast lookup            | HashMap                  |
| Key-value, sorted keys            | TreeMap                  |
| Key-value, insertion order        | LinkedHashMap            |
| Thread-safe map                   | ConcurrentHashMap        |
| LIFO stack                        | ArrayDeque               |
| FIFO queue                        | LinkedList or ArrayDeque |
| Priority queue                    | PriorityQueue            |

---

### Q13: What is the difference between LinkedHashMap and HashMap?

| HashMap            | LinkedHashMap             |
| ------------------ | ------------------------- |
| No order guarantee | Maintains insertion order |
| Slightly faster    | Slightly slower           |
| Less memory        | More memory (linked list) |
| Use for most cases | Use when order matters    |

```java
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("B", 2);
hashMap.put("A", 1);
hashMap.put("C", 3);
// Order not guaranteed

Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
linkedHashMap.put("B", 2);
linkedHashMap.put("A", 1);
linkedHashMap.put("C", 3);
// Order: B, A, C (insertion order)

// Access order mode
Map<String, Integer> accessOrder = new LinkedHashMap<>(16, 0.75f, true);
// Useful for LRU cache implementation
```

---

### Q14: How does TreeMap work?

**Answer:**
TreeMap is a Red-Black tree based NavigableMap implementation.

| Feature     | Details                       |
| ----------- | ----------------------------- |
| Ordering    | Natural order or Comparator   |
| Operations  | O(log n) for get, put, remove |
| Null keys   | Not allowed (throws NPE)      |
| Null values | Allowed                       |

```java
TreeMap<String, Integer> map = new TreeMap<>();
map.put("C", 3);
map.put("A", 1);
map.put("B", 2);

// Automatically sorted by key
System.out.println(map);  // {A=1, B=2, C=3}

// Navigation methods
map.firstKey();      // "A"
map.lastKey();       // "C"
map.lowerKey("B");   // "A" (strictly less)
map.floorKey("B");   // "B" (less or equal)
map.higherKey("B");  // "C" (strictly greater)
map.subMap("A", "C"); // {A=1, B=2}
```

---

### Q15: What is CopyOnWriteArrayList and when to use it?

**Answer:**
CopyOnWriteArrayList creates a new copy of the array for every write operation.

| Feature  | Details                            |
| -------- | ---------------------------------- |
| Reads    | O(1), no locking                   |
| Writes   | O(n), copies entire array          |
| Iterator | Snapshot - doesn't reflect changes |
| Use case | Read-heavy, write-rare scenarios   |

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("A");
list.add("B");

// Safe iteration during modification
for (String s : list) {
    list.add("C");  // No ConcurrentModificationException
    System.out.println(s);  // Only prints A, B (snapshot)
}
```

**When to use:**

- Event listener lists
- Configuration caching
- High read, low write scenarios

---

### Q16: What is the output?

```java
Set<Integer> set = new TreeSet<>();
set.add(5);
set.add(2);
set.add(8);
set.add(1);

Iterator<Integer> it = set.iterator();
while (it.hasNext()) {
    System.out.print(it.next() + " ");
}
```

**Answer:** `1 2 5 8`

TreeSet maintains elements in sorted order.

---

### Q17: What is the difference between Comparable and Comparator?

| Comparable                  | Comparator            |
| --------------------------- | --------------------- |
| In the class being compared | Separate class        |
| `compareTo(T o)`            | `compare(T o1, T o2)` |
| Single natural ordering     | Multiple orderings    |
| Modifies original class     | Doesn't modify        |

```java
// Comparable - natural ordering
class Employee implements Comparable<Employee> {
    String name;
    int salary;

    @Override
    public int compareTo(Employee other) {
        return this.salary - other.salary;  // By salary
    }
}

// Comparator - custom ordering
Comparator<Employee> byName = (e1, e2) -> e1.name.compareTo(e2.name);
Comparator<Employee> bySalaryDesc = Comparator.comparingInt(Employee::getSalary).reversed();

// Multiple orderings
Collections.sort(employees);                    // Natural order (by salary)
Collections.sort(employees, byName);            // By name
Collections.sort(employees, bySalaryDesc);      // By salary descending
```

---

### Q18: What is PriorityQueue?

**Answer:**
PriorityQueue is a heap-based queue where elements are ordered by priority.

```java
// Min-heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.addAll(List.of(5, 2, 8, 1));
minHeap.poll();  // 1 (smallest)
minHeap.poll();  // 2

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.addAll(List.of(5, 2, 8, 1));
maxHeap.poll();  // 8 (largest)

// Custom priority
PriorityQueue<Task> taskQueue = new PriorityQueue<>(
    Comparator.comparingInt(Task::getPriority)
);
```

**Operations:**

- `offer()` / `add()`: O(log n)
- `poll()` / `remove()`: O(log n)
- `peek()`: O(1)

---

### Q19: How to convert between Collection types?

```java
List<String> list = List.of("A", "B", "C");

// List → Set (removes duplicates)
Set<String> set = new HashSet<>(list);

// Set → List
List<String> backToList = new ArrayList<>(set);

// List → Array
String[] array = list.toArray(new String[0]);

// Array → List (fixed-size)
List<String> fixedList = Arrays.asList(array);

// Array → List (modifiable)
List<String> modifiableList = new ArrayList<>(Arrays.asList(array));

// Map → Set of entries
Set<Map.Entry<K, V>> entries = map.entrySet();

// Map → Collection of values
Collection<V> values = map.values();

// Map → Set of keys
Set<K> keys = map.keySet();
```

---

### Q20: What is EnumSet and EnumMap?

**Answer:**
Specialized collections for enum types with better performance.

```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

// EnumSet - high-performance Set for enums
EnumSet<Day> weekdays = EnumSet.range(Day.MON, Day.FRI);
EnumSet<Day> weekend = EnumSet.of(Day.SAT, Day.SUN);
EnumSet<Day> allDays = EnumSet.allOf(Day.class);
EnumSet<Day> noDays = EnumSet.noneOf(Day.class);

// EnumMap - high-performance Map for enum keys
EnumMap<Day, String> activities = new EnumMap<>(Day.class);
activities.put(Day.MON, "Work");
activities.put(Day.SAT, "Rest");
```

**Benefits:**

- Internally uses bit vectors (EnumSet)
- Extremely fast operations
- Compact memory representation
- Type-safe

---

## Key Takeaways

1. **ArrayList** - Default choice for List (O(1) access)
2. **HashSet/HashMap** - O(1) operations with good hashCode
3. **TreeSet/TreeMap** - Sorted, O(log n) operations
4. **LinkedHashMap** - Maintains insertion order
5. **ConcurrentHashMap** - Thread-safe alternative to HashMap
6. **Iterator.remove()** - Safe way to remove during iteration
7. **equals/hashCode contract** - Critical for hash-based collections
8. **Immutable collections** - Use List.of(), Set.of(), Map.of()
9. **Fail-fast** - Standard collections throw on concurrent modification
10. **Choose wisely** - Performance varies dramatically by use case
