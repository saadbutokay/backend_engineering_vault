---
title: "Java - Collections Framework Overview"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - collections
  - data-structures
  - list
  - set
  - map
status: "not-started"
---

# Java - Collections Framework Overview

> [!abstract] Overview
> The Java Collections Framework (JCF) is a unified architecture of interfaces and classes for storing, retrieving, manipulating, and aggregating groups of objects. It replaces the legacy data structures (arrays, Vector, Hashtable) with a consistent, type-safe, and high-performance set of collections. In backend development, collections are used everywhere: storing query results from the database, caching frequently accessed data, managing user sessions, processing batch operations, and building in-memory data structures for business logic. This note provides a comprehensive overview of the entire framework. The subsequent notes dive deep into each collection type.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Arrays - 1D and 2D]]
> - [[Java - Classes and Objects]]
> - [[Java - Inheritance - Single Multilevel Hierarchical]]
> - [[Java - Abstraction - Abstract Classes and Interfaces]]
> - [[Java - Custom Exceptions]]

---

## Theory

### What is the Collections Framework?

The Java Collections Framework is a set of interfaces and classes in the `java.util` package that provide a standard way to work with groups of objects. Before the Collections Framework was introduced in Java 1.2 (1998), Java had a scattered set of data structures: `Vector`, `Stack`, `Hashtable`, `Dictionary`, and arrays. Each had its own API, naming conventions, and behavior. The Collections Framework unified all of these under a common set of interfaces, making it possible to write algorithms that work with any collection type.

The framework has three main components:

1. **Interfaces**: Abstract types that define the contract for each collection category (`List`, `Set`, `Map`, `Queue`, `Deque`).
2. **Implementations**: Concrete classes that implement the interfaces (`ArrayList`, `HashSet`, `HashMap`, `LinkedList`, `TreeMap`, etc.).
3. **Algorithms**: Utility methods in the `Collections` and `Arrays` classes for sorting, searching, shuffling, and other operations.

### The Collection Hierarchy

The hierarchy is rooted at the `Iterable` interface, which allows any collection to be used in a for-each loop. Below `Iterable` is the `Collection` interface, which branches into three main sub-interfaces: `List`, `Set`, and `Queue`. The `Map` interface is **not** part of the `Collection` hierarchy because it stores key-value pairs rather than individual elements, but it is considered part of the Collections Framework.

```text
Iterable<T>
    └── Collection<T>
            ├── List<T> (ordered, allows duplicates)
            │       ├── ArrayList
            │       ├── LinkedList
            │       ├── Vector (legacy, synchronized)
            │       └── Stack (legacy, extends Vector)
            │
            ├── Set<T> (unordered, no duplicates)
            │       ├── HashSet
            │       ├── LinkedHashSet (insertion order)
            │       └── TreeSet (sorted order)
            │
            └── Queue<T> (FIFO processing)
                    ├── PriorityQueue (sorted by priority)
                    └── Deque<T> (double-ended queue)
                            ├── ArrayDeque
                            └── LinkedList (implements both List and Deque)

Map<K, V> (key-value pairs, NOT a Collection)
    ├── HashMap
    ├── LinkedHashMap (insertion order)
    ├── TreeMap (sorted by key)
    ├── Hashtable (legacy, synchronized)
    └── ConcurrentHashMap (thread-safe)
```

### The Core Interfaces

**`Iterable<T>`**: The root of the hierarchy. Any class that implements `Iterable` can be used in a for-each loop. It defines a single method: `Iterator<T> iterator()`.

**`Collection<T>`**: The base interface for all single-element collections. Defines the fundamental operations: `add()`, `remove()`, `contains()`, `size()`, `isEmpty()`, `clear()`, `iterator()`, and bulk operations like `addAll()`, `removeAll()`, `retainAll()`.

**`List<T>`**: An ordered collection that allows duplicate elements. Elements can be accessed by their integer index (position). Think of it as a resizable array. The two most common implementations are `ArrayList` (fast random access, slow insertion/deletion in the middle) and `LinkedList` (fast insertion/deletion, slow random access).

**`Set<T>`**: A collection that contains no duplicate elements. It models the mathematical concept of a set. The three main implementations are `HashSet` (fastest, no ordering), `LinkedHashSet` (maintains insertion order), and `TreeSet` (maintains sorted order).

**`Queue<T>`**: A collection designed for holding elements prior to processing. Typically follows FIFO (First-In-First-Out) ordering, though priority queues order by priority. The `Deque` sub-interface supports insertion and removal at both ends.

**`Map<K, V>`**: An object that maps keys to values. A map cannot contain duplicate keys; each key maps to at most one value. The three main implementations are `HashMap` (fastest, no ordering), `LinkedHashMap` (maintains insertion order), and `TreeMap` (maintains sorted order by key).

### Choosing the Right Collection

This is one of the most important decisions you will make in backend development. The wrong choice can turn a 1-millisecond operation into a 10-second operation when your data grows from 100 records to 10 million.

**Decision flowchart:**

```text
Do you need key-value pairs?
├── YES -> Map
│       ├── Need sorted keys? -> TreeMap
│       ├── Need insertion order? -> LinkedHashMap
│       └── Default (fastest) -> HashMap
│
└── NO -> Collection
        ├── Need to maintain order and allow duplicates? -> List
        │       ├── Frequent random access by index? -> ArrayList
        │       └── Frequent insertion/deletion at ends? -> LinkedList / ArrayDeque
        │
        ├── Need to prevent duplicates? -> Set
        │       ├── Need sorted order? -> TreeSet
        │       ├── Need insertion order? -> LinkedHashSet
        │       └── Default (fastest) -> HashSet
        │
        └── Need FIFO/LIFO processing? -> Queue / Deque
                ├── Priority-based processing? -> PriorityQueue
                └── Add/remove from both ends? -> ArrayDeque
```

### Performance Comparison (Big-O Notation)

Big-O notation describes how an operation's time grows as the collection size (n) increases. O(1) means constant time (does not grow with n). O(n) means linear time (doubles when n doubles). O(log n) means logarithmic time (grows very slowly).

| Operation | ArrayList | LinkedList | HashSet | TreeSet | HashMap | TreeMap |
|-----------|-----------|------------|---------|---------|---------|---------|
| Add (end) | O(1)* | O(1) | O(1)* | O(log n) | O(1)* | O(log n) |
| Add (middle) | O(n) | O(1)** | N/A | N/A | N/A | N/A |
| Get by index | O(1) | O(n) | N/A | N/A | N/A | N/A |
| Get by key | N/A | N/A | N/A | N/A | O(1)* | O(log n) |
| Remove | O(n) | O(1)** | O(1)* | O(log n) | O(1)* | O(log n) |
| Contains | O(n) | O(n) | O(1)* | O(log n) | O(1)* | O(log n) |
| Search | O(n) | O(n) | O(1)* | O(log n) | O(1)* | O(log n) |

\* Amortized average case. Worst case is O(n) due to resizing or hash collisions.
\** O(1) if you already have a reference to the node. Finding the node is O(n).

### Arrays vs Collections

| Feature | Arrays | Collections |
|---------|--------|-------------|
| Size | Fixed at creation | Dynamic (grow and shrink) |
| Type safety | Primitives and objects | Objects only (primitives are autoboxed) |
| Performance | Fastest (no overhead) | Slightly slower (object overhead) |
| Methods | None (use `Arrays` utility) | Rich API (add, remove, sort, search, etc.) |
| Generics | No (arrays are covariant, which is unsafe) | Yes (compile-time type safety) |
| Use case | Fixed-size data, performance-critical code | Most backend data processing |

**When to use arrays:** When the size is known and fixed, when you need to store primitives without autoboxing overhead, or when you are interfacing with legacy APIs that require arrays.

**When to use collections:** Almost everywhere else. Collections provide dynamic sizing, type safety through generics, a rich API, and integration with the Streams API. In backend development, you will use collections 95% of the time and arrays 5% of the time.

### The `Collections` Utility Class

The `java.util.Collections` class (note the plural "s") provides static utility methods for working with collections. Do not confuse it with the `Collection` interface (singular).

| Method | Description |
|--------|-------------|
| `sort(List)` | Sorts a list in natural order |
| `sort(List, Comparator)` | Sorts a list using a custom comparator |
| `binarySearch(List, key)` | Searches a sorted list for a key |
| `reverse(List)` | Reverses the order of elements |
| `shuffle(List)` | Randomly shuffles the elements |
| `unmodifiableList(List)` | Returns an unmodifiable view |
| `unmodifiableMap(Map)` | Returns an unmodifiable view |
| `emptyList()` | Returns an immutable empty list |
| `emptyMap()` | Returns an immutable empty map |
| `singletonList(T)` | Returns an immutable list with one element |
| `frequency(Collection, Object)` | Counts occurrences of an object |
| `disjoint(Collection, Collection)` | Checks if two collections share no elements |
| `max(Collection)` | Returns the maximum element |
| `min(Collection)` | Returns the minimum element |

### Thread Safety and Concurrent Collections

The standard collections (`ArrayList`, `HashMap`, `HashSet`) are **not thread-safe**. If multiple threads modify a collection concurrently, the internal data structures can become corrupted, leading to infinite loops, lost data, or `ConcurrentModificationException`.

For multi-threaded backend applications, Java provides concurrent collection implementations in the `java.util.concurrent` package:

| Standard Collection | Concurrent Alternative |
|--------------------|----------------------|
| `HashMap` | `ConcurrentHashMap` |
| `ArrayList` | `CopyOnWriteArrayList` |
| `HashSet` | `ConcurrentHashMap.newKeySet()` |
| `LinkedList` (Queue) | `ConcurrentLinkedQueue` |
| `ArrayDeque` | `ConcurrentLinkedDeque` |
| `PriorityQueue` | `PriorityBlockingQueue` |

In Spring Boot, most collections are used within a single request thread, so standard collections are sufficient. Concurrent collections are needed for shared caches, session stores, and background task queues.

> [!tip] Key Insight
> The Collections Framework is built on **interfaces**, not implementations. Your method signatures should use interface types (`List<Order>`, `Map<String, User>`, `Set<Long>`) rather than implementation types (`ArrayList<Order>`, `HashMap<String, User>`, `HashSet<Long>`). This allows you to change the implementation without modifying the callers. For example, if you start with `ArrayList` and later need sorted order, you can switch to `LinkedList` or wrap the list in a sorted view without changing any method signatures. This is the Dependency Inversion Principle applied to data structures.

---

## Syntax and Basic Examples

### Example 1: List operations

```java
import java.util.*;

public class ListDemo {
    public static void main(String[] args) {
        // Creating lists
        List<String> cities = new ArrayList<>();
        cities.add("Dhaka");
        cities.add("Chittagong");
        cities.add("Sylhet");
        cities.add("Dhaka");  // Duplicates allowed in List

        System.out.println("Cities: " + cities);        // [Dhaka, Chittagong, Sylhet, Dhaka]
        System.out.println("Size: " + cities.size());    // 4
        System.out.println("Get(1): " + cities.get(1));  // Chittagong
        System.out.println("Contains Sylhet: " + cities.contains("Sylhet"));  // true
        System.out.println("Index of Dhaka: " + cities.indexOf("Dhaka"));     // 0 (first occurrence)

        // Modifying
        cities.set(2, "Rajshahi");  // Replace Sylhet with Rajshahi
        cities.remove("Dhaka");     // Removes the FIRST occurrence of Dhaka
        System.out.println("After modifications: " + cities);  // [Chittagong, Rajshahi, Dhaka]

        // Immutable list (Java 9+)
        List<String> immutable = List.of("Dhaka", "Chittagong", "Sylhet");
        // immutable.add("Rajshahi");  // UnsupportedOperationException!

        // Iteration
        for (String city : cities) {
            System.out.println("City: " + city);
        }
    }
}
```

### Example 2: Set operations

```java
import java.util.*;

public class SetDemo {
    public static void main(String[] args) {
        // HashSet: no ordering, no duplicates, fastest
        Set<String> tags = new HashSet<>();
        tags.add("java");
        tags.add("spring");
        tags.add("backend");
        tags.add("java");  // Duplicate! Ignored.

        System.out.println("Tags: " + tags);      // [java, backend, spring] (order may vary)
        System.out.println("Size: " + tags.size()); // 3 (duplicate not counted)

        // TreeSet: sorted order
        Set<String> sortedTags = new TreeSet<>();
        sortedTags.add("java");
        sortedTags.add("spring");
        sortedTags.add("backend");
        System.out.println("Sorted: " + sortedTags);  // [backend, java, spring]

        // LinkedHashSet: insertion order
        Set<String> orderedTags = new LinkedHashSet<>();
        orderedTags.add("java");
        orderedTags.add("spring");
        orderedTags.add("backend");
        System.out.println("Ordered: " + orderedTags);  // [java, spring, backend]

        // Set operations (mathematical)
        Set<Integer> setA = new HashSet<>(Set.of(1, 2, 3, 4, 5));
        Set<Integer> setB = new HashSet<>(Set.of(4, 5, 6, 7, 8));

        Set<Integer> union = new HashSet<>(setA);
        union.addAll(setB);
        System.out.println("Union: " + union);  // [1, 2, 3, 4, 5, 6, 7, 8]

        Set<Integer> intersection = new HashSet<>(setA);
        intersection.retainAll(setB);
        System.out.println("Intersection: " + intersection);  // [4, 5]

        Set<Integer> difference = new HashSet<>(setA);
        difference.removeAll(setB);
        System.out.println("Difference (A-B): " + difference);  // [1, 2, 3]
    }
}
```

### Example 3: Map operations

```java
import java.util.*;

public class MapDemo {
    public static void main(String[] args) {
        // HashMap: key-value pairs, no ordering, fastest
        Map<String, Double> productPrices = new HashMap<>();
        productPrices.put("Laptop", 85000.0);
        productPrices.put("Mouse", 1500.0);
        productPrices.put("Keyboard", 3200.0);
        productPrices.put("Laptop", 79999.0);  // Overwrites the previous value!

        System.out.println("Prices: " + productPrices);
        System.out.println("Mouse price: " + productPrices.get("Mouse"));       // 1500.0
        System.out.println("Monitor price: " + productPrices.get("Monitor"));   // null (not found)
        System.out.println("Contains Laptop: " + productPrices.containsKey("Laptop"));  // true
        System.out.println("Size: " + productPrices.size());  // 3

        // Safe retrieval with default value
        double monitorPrice = productPrices.getOrDefault("Monitor", 0.0);
        System.out.println("Monitor (default): " + monitorPrice);  // 0.0

        // Iterating over a map
        System.out.println("\n--- All Products ---");
        for (Map.Entry<String, Double> entry : productPrices.entrySet()) {
            System.out.printf("%-12s: %,.2f BDT%n", entry.getKey(), entry.getValue());
        }

        // Iterating over keys only
        for (String product : productPrices.keySet()) {
            System.out.println("Product: " + product);
        }

        // Iterating over values only
        for (Double price : productPrices.values()) {
            System.out.println("Price: " + price);
        }

        // TreeMap: sorted by key
        Map<String, Double> sortedPrices = new TreeMap<>(productPrices);
        System.out.println("\nSorted: " + sortedPrices);
        // {Keyboard=3200.0, Laptop=79999.0, Mouse=1500.0}

        // Immutable map (Java 9+)
        Map<String, Integer> httpStatus = Map.of(
            "OK", 200,
            "NOT_FOUND", 404,
            "SERVER_ERROR", 500
        );
        // httpStatus.put("CREATED", 201);  // UnsupportedOperationException!
    }
}
```

### Example 4: Queue and Deque operations

```java
import java.util.*;

public class QueueDemo {
    public static void main(String[] args) {
        // PriorityQueue: elements ordered by priority (natural ordering by default)
        Queue<Integer> taskQueue = new PriorityQueue<>();
        taskQueue.add(30);
        taskQueue.add(10);
        taskQueue.add(20);

        System.out.println("Queue: " + taskQueue);  // [10, 30, 20] (internal order)
        System.out.println("Peek: " + taskQueue.peek());  // 10 (smallest = highest priority)
        System.out.println("Poll: " + taskQueue.poll());  // 10 (removes and returns)
        System.out.println("Poll: " + taskQueue.poll());  // 20
        System.out.println("Poll: " + taskQueue.poll());  // 30
        System.out.println("Poll: " + taskQueue.poll());  // null (empty)

        // ArrayDeque: double-ended queue, faster than LinkedList for stack/queue operations
        Deque<String> deque = new ArrayDeque<>();

        // Use as a queue (FIFO)
        deque.addLast("First");
        deque.addLast("Second");
        deque.addLast("Third");
        System.out.println("\nQueue front: " + deque.removeFirst());  // First

        // Use as a stack (LIFO)
        deque.push("Bottom");
        deque.push("Middle");
        deque.push("Top");
        System.out.println("Stack pop: " + deque.pop());  // Top
        System.out.println("Stack pop: " + deque.pop());  // Middle
    }
}
```

### Example 5: Collections utility methods

```java
import java.util.*;

public class CollectionsUtilDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(List.of(45, 12, 78, 3, 56, 91, 23));

        // Sorting
        Collections.sort(numbers);
        System.out.println("Sorted: " + numbers);  // [3, 12, 23, 45, 56, 78, 91]

        // Reverse
        Collections.reverse(numbers);
        System.out.println("Reversed: " + numbers);  // [91, 78, 56, 45, 23, 12, 3]

        // Binary search (list must be sorted first!)
        Collections.sort(numbers);
        int index = Collections.binarySearch(numbers, 56);
        System.out.println("56 found at index: " + index);  // 4

        // Min and Max
        System.out.println("Min: " + Collections.min(numbers));  // 3
        System.out.println("Max: " + Collections.max(numbers));  // 91

        // Frequency
        List<String> words = List.of("java", "spring", "java", "backend", "java");
        System.out.println("Frequency of 'java': " + Collections.frequency(words, "java"));  // 3

        // Unmodifiable view
        List<String> mutable = new ArrayList<>(List.of("A", "B", "C"));
        List<String> unmodifiable = Collections.unmodifiableList(mutable);
        // unmodifiable.add("D");  // UnsupportedOperationException!
        mutable.add("D");  // OK: the original list is still mutable
        System.out.println("Unmodifiable view: " + unmodifiable);  // [A, B, C, D] (reflects changes)
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Collections are the backbone of every backend service. Here are three realistic scenarios.

### Scenario 1: Service layer returning collections from the database

In a Spring Boot service, repository queries return collections of entities. The service layer processes, filters, and transforms these collections before returning them to the controller.

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    // Returns a List: ordered, may contain duplicates (though unlikely for orders)
    public List<OrderResponse> getOrdersByUser(Long userId) {
        // Spring Data returns a List<Order> from the database
        List<Order> orders = orderRepository.findByUserIdOrderByCreatedAtDesc(userId);

        // Transform entities to DTOs using Streams (covered in a later note)
        return orders.stream()
            .map(OrderResponse::fromEntity)
            .toList();  // Returns an unmodifiable List (Java 16+)
    }

    // Returns a Set: no duplicates, used for unique values
    public Set<String> getUniqueOrderStatuses(Long userId) {
        List<Order> orders = orderRepository.findByUserId(userId);

        // Collect unique statuses into a Set
        Set<String> statuses = new HashSet<>();
        for (Order order : orders) {
            statuses.add(order.getStatus().name());
        }
        return statuses;
        // Or with Streams: orders.stream().map(o -> o.getStatus().name()).collect(Collectors.toSet());
    }

    // Returns a Map: key-value pairs for fast lookup
    public Map<Long, BigDecimal> getOrderTotalsByUser(Long userId) {
        List<Order> orders = orderRepository.findByUserId(userId);

        // Build a map of orderId -> totalAmount
        Map<Long, BigDecimal> totals = new HashMap<>();
        for (Order order : orders) {
            totals.put(order.getId(), order.getTotalAmount());
        }
        return totals;
        // Or with Streams: orders.stream().collect(Collectors.toMap(Order::getId, Order::getTotalAmount));
    }

    // Returns a Map grouped by status
    public Map<OrderStatus, List<Order>> getOrdersGroupedByStatus(Long userId) {
        List<Order> orders = orderRepository.findByUserId(userId);

        Map<OrderStatus, List<Order>> grouped = new HashMap<>();
        for (Order order : orders) {
            grouped.computeIfAbsent(order.getStatus(), k -> new ArrayList<>()).add(order);
        }
        return grouped;
        // computeIfAbsent() creates a new ArrayList for the key if it does not exist yet.
        // This is the standard pattern for grouping elements into a map of lists.
    }
}
```

**What to notice:**

- The repository returns `List<Order>`. The service transforms it into different collection types depending on what the caller needs: a `List` for ordered results, a `Set` for unique values, a `Map` for key-value lookups.
- `computeIfAbsent()` is one of the most useful Map methods in backend code. It eliminates the null-check boilerplate when building maps of lists (grouping operations).
- The method signatures use interface types (`List`, `Set`, `Map`), not implementation types (`ArrayList`, `HashSet`, `HashMap`). This allows the implementation to change without affecting callers.

### Scenario 2: Caching with collections

In-memory caching uses collections to store frequently accessed data and avoid repeated database queries.

```java
package com.company.orderservice.cache;

import org.springframework.stereotype.Service;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

@Service
public class ProductCache {

    // HashMap for O(1) lookup by product ID
    private final Map<Long, Product> cacheById = new HashMap<>();

    // LinkedHashMap for LRU (Least Recently Used) eviction
    // The 'true' parameter enables access-order mode: entries are reordered
    // on every access, so the least recently accessed entry is at the head.
    private final Map<Long, Product> lruCache = new LinkedHashMap<>(100, 0.75f, true) {
        @Override
        protected boolean removeEldestEntry(Map.Entry<Long, Product> eldest) {
            return size() > 1000;  // Evict the oldest entry when size exceeds 1000
        }
    };

    public Product getProduct(Long productId) {
        // Check cache first (O(1) lookup)
        Product cached = cacheById.get(productId);
        if (cached != null) {
            return cached;  // Cache hit
        }

        // Cache miss: fetch from database and store in cache
        Product product = productRepository.findById(productId)
            .orElseThrow(() -> new ResourceNotFoundException("Product", productId));

        cacheById.put(productId, product);
        lruCache.put(productId, product);

        return product;
    }

    public void invalidate(Long productId) {
        cacheById.remove(productId);
        lruCache.remove(productId);
    }

    public void invalidateAll() {
        cacheById.clear();
        lruCache.clear();
    }

    public int getCacheSize() {
        return cacheById.size();
    }
}
```

**What to notice:**

- `HashMap` provides O(1) lookup by product ID, making cache hits extremely fast compared to a database query.
- `LinkedHashMap` with access-order mode and `removeEldestEntry()` override implements an LRU cache. When the cache exceeds 1000 entries, the least recently accessed entry is automatically evicted. This prevents the cache from consuming all available memory.
- In production, you would use Redis or Caffeine instead of a raw `HashMap` for caching. But understanding the underlying collection mechanics helps you configure these tools correctly.

### Scenario 3: Batch processing with collections

Backend systems frequently process large batches of records. Collections are used to accumulate, partition, and process these batches.

```java
package com.company.orderservice.batch;

import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;

@Service
public class InvoiceBatchProcessor {

    private static final int BATCH_SIZE = 500;

    public BatchResult generateMonthlyInvoices(int year, int month) {
        // Fetch all uninvoiced orders for the month
        List<Order> allOrders = orderRepository.findUninvoicedOrders(year, month);

        int totalProcessed = 0;
        int totalFailed = 0;
        List<Long> failedOrderIds = new ArrayList<>();

        // Partition the list into batches of BATCH_SIZE
        // This prevents memory issues and database lock contention
        for (int i = 0; i < allOrders.size(); i += BATCH_SIZE) {
            int endIndex = Math.min(i + BATCH_SIZE, allOrders.size());
            List<Order> batch = allOrders.subList(i, endIndex);
            // subList() returns a VIEW of the original list, not a copy.
            // This is memory-efficient for large lists.

            for (Order order : batch) {
                try {
                    invoiceService.generateInvoice(order);
                    totalProcessed++;
                } catch (Exception e) {
                    totalFailed++;
                    failedOrderIds.add(order.getId());
                    // logger.error("Failed to invoice order {}", order.getId(), e);
                }
            }

            // logger.info("Processed batch {}/{}. Success: {}, Failed: {}",
            //     (i / BATCH_SIZE) + 1,
            //     (allOrders.size() + BATCH_SIZE - 1) / BATCH_SIZE,
            //     totalProcessed, totalFailed);
        }

        return new BatchResult(totalProcessed, totalFailed, failedOrderIds);
    }
}
```

**What to notice:**

- `subList()` creates a lightweight view of the original list without copying elements. This is critical for memory efficiency when processing millions of records.
- The `failedOrderIds` list accumulates the IDs of failed orders for later retry or manual review. Using a `List` (not a `Set`) preserves the order of failures and allows duplicates if the same order fails multiple times in different batches.
- The batch size constant controls the trade-off between throughput (larger batches) and memory usage (smaller batches). This is a tuning parameter that depends on the available heap size and database connection pool.

---

## Java vs Python Comparison

> [!note] Cross-language perspective
> Python's built-in data structures map closely to Java's Collections Framework, but Python's dynamic typing makes them simpler to use.

| Java | Python | Notes |
|------|--------|-------|
| `ArrayList<T>` | `list` | Both are dynamic arrays. Python lists can hold mixed types. |
| `LinkedList<T>` | `collections.deque` | Python's deque is optimized for both ends. |
| `HashSet<T>` | `set` | Both provide O(1) lookup and no duplicates. |
| `TreeSet<T>` | `sortedcontainers.SortedSet` | Not built-in in Python. Requires a third-party library. |
| `HashMap<K,V>` | `dict` | Both provide O(1) key-value lookup. Python dicts maintain insertion order (3.7+). |
| `TreeMap<K,V>` | `sortedcontainers.SortedDict` | Not built-in in Python. |
| `PriorityQueue<T>` | `heapq` module | Python's heapq is a module, not a class. |
| `ArrayDeque<T>` | `collections.deque` | Same data structure, different name. |

```java
// Java: explicit types, generics, interface-based
List<String> names = new ArrayList<>();
names.add("Saad");
names.add("Rahim");
Map<String, Integer> ages = new HashMap<>();
ages.put("Saad", 22);
Set<String> unique = new HashSet<>(names);
```

```python
# Python: dynamic types, built-in literals
names = ["Saad", "Rahim"]
ages = {"Saad": 22}
unique = set(names)
```

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using the wrong collection type for the use case

**Wrong:**

```java
// Using a List when you need fast lookup by ID
List<Order> orders = orderRepository.findAll();
Order target = null;
for (Order order : orders) {
    if (order.getId().equals(targetId)) {  // O(n) search through the entire list
        target = order;
        break;
    }
}
```

**Right:**

```java
// Using a Map for O(1) lookup by ID
Map<Long, Order> orderMap = new HashMap<>();
for (Order order : orderRepository.findAll()) {
    orderMap.put(order.getId(), order);
}
Order target = orderMap.get(targetId);  // O(1) lookup
```

**Why it is wrong:** Searching a `List` for an element by a field value is O(n). If the list has 1 million orders, you might scan all 1 million to find the one you need. A `HashMap` lookup is O(1), meaning it takes the same time regardless of the collection size. Choosing the right collection type is the single most impactful performance decision in backend code.

### Mistake 2: Using implementation types in method signatures

**Wrong:**

```java
public ArrayList<Order> getOrders() { ... }
public HashMap<String, User> getUserMap() { ... }
public void processOrders(LinkedList<Order> orders) { ... }
```

**Right:**

```java
public List<Order> getOrders() { ... }
public Map<String, User> getUserMap() { ... }
public void processOrders(List<Order> orders) { ... }
```

**Why it is wrong:** Using implementation types in method signatures couples the caller to a specific data structure. If you later need to change from `ArrayList` to `LinkedList` for performance reasons, every caller must be updated. Using interface types (`List`, `Map`, `Set`) gives you the freedom to change the implementation without breaking the API.

### Mistake 3: Modifying a collection while iterating over it

**Wrong:**

```java
List<String> users = new ArrayList<>(List.of("Alice", "Bob", "Charlie", "Dave"));

for (String user : users) {
    if ("Bob".equals(user)) {
        users.remove(user);  // ConcurrentModificationException!
    }
}
```

**Right:**

```java
// Option 1: Iterator.remove()
Iterator<String> it = users.iterator();
while (it.hasNext()) {
    if ("Bob".equals(it.next())) {
        it.remove();  // Safe removal through the iterator
    }
}

// Option 2: removeIf() (Java 8+, cleanest)
users.removeIf("Bob"::equals);

// Option 3: Collect items to remove, then remove after iteration
List<String> toRemove = new ArrayList<>();
for (String user : users) {
    if ("Bob".equals(user)) {
        toRemove.add(user);
    }
}
users.removeAll(toRemove);
```

**Why it is wrong:** The for-each loop uses an `Iterator` internally. If you modify the collection directly (add or remove elements) while the iterator is active, the iterator detects the structural change and throws a `ConcurrentModificationException`. This is a safety mechanism to prevent unpredictable behavior.

### Mistake 4: Not considering thread safety for shared collections

**Wrong:**

```java
@Service
public class SessionStore {
    // Shared across all threads! Multiple HTTP requests modify this concurrently.
    private final Map<String, UserSession> sessions = new HashMap<>();

    public void addSession(String sessionId, UserSession session) {
        sessions.put(sessionId, session);  // Race condition!
    }

    public UserSession getSession(String sessionId) {
        return sessions.get(sessionId);  // May return corrupted data
    }
}
```

**Right:**

```java
@Service
public class SessionStore {
    // ConcurrentHashMap is designed for concurrent access from multiple threads.
    private final Map<String, UserSession> sessions = new ConcurrentHashMap<>();

    public void addSession(String sessionId, UserSession session) {
        sessions.put(sessionId, session);  // Thread-safe
    }

    public UserSession getSession(String sessionId) {
        return sessions.get(sessionId);  // Thread-safe
    }
}
```

**Why it is wrong:** `HashMap` is not thread-safe. When multiple threads modify it concurrently, the internal hash table can become corrupted, leading to infinite loops, lost entries, or `NullPointerException` inside the HashMap's internal code. In a Spring Boot server handling thousands of concurrent requests, any collection shared across requests must be thread-safe. Use `ConcurrentHashMap`, `CopyOnWriteArrayList`, or synchronize access explicitly.

---

## Key Takeaways

> [!tip] Remember these points
> 1. The Java Collections Framework provides a unified set of interfaces (`List`, `Set`, `Map`, `Queue`, `Deque`) and implementations (`ArrayList`, `HashSet`, `HashMap`, `TreeMap`, etc.) for working with groups of objects. It replaces arrays for most backend data processing tasks.
> 2. **Choose the right collection type** based on your access patterns: `List` for ordered sequences with duplicates, `Set` for unique elements, `Map` for key-value lookups, `Queue` for FIFO processing. Within each category, choose the implementation based on performance needs (hash-based for O(1), tree-based for O(log n) sorted order).
> 3. **Use interface types in method signatures** (`List<Order>`, not `ArrayList<Order>`). This follows the Dependency Inversion Principle and allows you to change implementations without modifying callers.
> 4. **Never modify a collection while iterating** over it with a for-each loop. Use `Iterator.remove()`, `removeIf()`, or collect changes in a separate list and apply them after iteration.
> 5. **Standard collections are not thread-safe**. In a multi-threaded backend server, shared collections must use concurrent implementations (`ConcurrentHashMap`, `CopyOnWriteArrayList`) or explicit synchronization. Collections that are local to a single request thread (the most common case) do not need thread safety.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Collection Type Selection (Easy)

For each of the following scenarios, identify the best collection type (interface and implementation) and explain why:

1. Storing a list of product names in the order they were added, allowing duplicates.
2. Storing unique email addresses of registered users for fast lookup.
3. Storing product IDs mapped to their prices for O(1) price lookup.
4. Processing customer support tickets in the order they were received (FIFO).
5. Maintaining a leaderboard of top 10 players sorted by score.
6. Storing HTTP response headers (key-value pairs, preserving insertion order).

> **Hint:** Use the decision flowchart from the Theory section. Consider the access patterns (lookup, insertion, ordering) and constraints (duplicates, sorting, thread safety).

### Exercise 2: Collection Operations Practice (Medium)

Write a program that processes a list of student records. Each student has a name, department, and CGPA. Use collections to:

1. Store all students in a `List`.
2. Extract unique departments into a `Set`.
3. Build a `Map` of department -> list of students in that department (grouping).
4. Build a `Map` of department -> average CGPA for that department.
5. Find the student with the highest CGPA using `Collections.max()` with a custom `Comparator`.

Print all results in a formatted manner.

> **Hint:** Use `computeIfAbsent()` for the grouping map. For the average CGPA map, iterate over the grouped map and calculate the average for each department's student list.

### Exercise 3: Collection Performance Comparison (Medium)

Write a program that compares the performance of `ArrayList`, `LinkedList`, `HashSet`, and `TreeSet` for the following operations with 100,000 elements:

1. Insertion (adding 100,000 elements).
2. Search (checking if a specific element exists).
3. Deletion (removing 10,000 elements).

Use `System.nanoTime()` to measure each operation. Print the results in a formatted table and explain why each collection performs the way it does based on its internal data structure.

> **Hint:** For the search test, search for an element near the end of the collection to highlight the difference between O(1) and O(n) lookups.

### Exercise 4: In-Memory Order Management System (Hard, Optional)

Build a simplified in-memory order management system using collections:

1. `OrderStore` class with internal collections to store orders. Use a `Map<Long, Order>` for O(1) lookup by ID and a `List<Order>` for maintaining chronological order.
2. Methods: `addOrder(Order)`, `getOrder(Long id)`, `getOrdersByUser(Long userId)`, `getOrdersByStatus(OrderStatus)`, `cancelOrder(Long id)`, `getOrderSummary()` (returns a Map of status -> count).
3. The `getOrdersByUser()` method should return a `List` sorted by creation date (newest first).
4. The `getOrderSummary()` method should return a `Map<OrderStatus, Integer>` using `computeIfAbsent()` or `merge()`.

In `main()`, add 10-15 orders with different users and statuses, then test all methods.

> **Hint:** Keep the `Map` and `List` in sync: when you add an order, add it to both collections. When you cancel an order, update it in the map (the list holds the same reference, so it is automatically updated).

<details>
<summary>Solution for Exercise 1</summary>

1. **`ArrayList<String>`**: Ordered, allows duplicates, fast insertion at the end.
2. **`HashSet<String>`**: No duplicates, O(1) lookup by email.
3. **`HashMap<Long, Double>`**: O(1) lookup by product ID.
4. **`ArrayDeque<Ticket>`** or **`LinkedList<Ticket>`**: FIFO queue, fast add/remove at ends.
5. **`TreeSet<Player>`** with a custom Comparator sorting by score descending: maintains sorted order automatically.
6. **`LinkedHashMap<String, String>`**: Key-value pairs with insertion order preserved.

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;

record Student(String name, String department, double cgpa) {}

public class Main {
    public static void main(String[] args) {
        List<Student> students = List.of(
            new Student("Saad", "CSE", 3.72),
            new Student("Rahim", "CSE", 3.45),
            new Student("Karim", "EEE", 3.80),
            new Student("Fatima", "EEE", 3.90),
            new Student("Nila", "BBA", 3.60),
            new Student("Arif", "CSE", 3.55)
        );

        // Unique departments
        Set<String> departments = new HashSet<>();
        for (Student s : students) departments.add(s.department());
        System.out.println("Departments: " + departments);

        // Group by department
        Map<String, List<Student>> byDept = new HashMap<>();
        for (Student s : students) {
            byDept.computeIfAbsent(s.department(), k -> new ArrayList<>()).add(s);
        }
        System.out.println("\nGrouped: " + byDept);

        // Average CGPA by department
        Map<String, Double> avgCgpa = new HashMap<>();
        for (var entry : byDept.entrySet()) {
            double avg = entry.getValue().stream()
                .mapToDouble(Student::cgpa).average().orElse(0);
            avgCgpa.put(entry.getKey(), Math.round(avg * 100.0) / 100.0);
        }
        System.out.println("\nAvg CGPA: " + avgCgpa);

        // Highest CGPA student
        Student top = Collections.max(students, Comparator.comparingDouble(Student::cgpa));
        System.out.println("\nTop student: " + top.name() + " (" + top.cgpa() + ")");
    }
}
```

</details>

---

## Related Notes

- [[Java - Custom Exceptions]]
- [[Java - List - ArrayList LinkedList]] (next note)
- [[Java - Set - HashSet TreeSet LinkedHashSet]]
- [[Java - Map - HashMap TreeMap LinkedHashMap]]

---

## Resources

- [Oracle Java Tutorials: Collections Framework](https://docs.oracle.com/javase/tutorial/collections/) - Official documentation covering the entire framework with tutorials for each collection type.
- [Oracle Java Documentation: java.util Package](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/package-summary.html) - Complete API reference for all collection interfaces and classes.
- [Baeldung: Java Collections Guide](https://www.baeldung.com/java-collections) - Comprehensive overview with decision trees and performance comparisons.
- [Baeldung: Java Collections Cheat Sheet](https://www.baeldung.com/java-collections-cheat-sheet) - Quick reference for choosing the right collection type.
- [Effective Java by Joshua Bloch - Item 25: Prefer Lists to Arrays](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive argument for using collections over arrays in most situations.
- [Java Concurrency in Practice by Brian Goetz - Chapter 5: Building Blocks](https://www.oreilly.com/library/view/java-concurrency-in/0321349601/) - Essential reading on concurrent collections for multi-threaded backend systems.
