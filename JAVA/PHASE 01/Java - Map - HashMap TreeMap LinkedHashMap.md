---
title: "Java - Map - HashMap TreeMap LinkedHashMap"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - collections
  - map
  - hashmap
  - treemap
  - linkedhashmap
status: "not-started"
---

# Java - Map - HashMap TreeMap LinkedHashMap

> [!abstract] Overview
> The `Map` interface represents a collection of key-value pairs where each key maps to exactly one value. Unlike `List` and `Set`, `Map` does not extend `Collection` because it operates on pairs rather than individual elements. Maps are the most versatile collection type in backend development. They power database result caching, configuration management, HTTP header processing, JSON deserialization, session storage, rate limiting, and virtually every lookup operation in a Spring Boot application. The three primary implementations are `HashMap` (O(1) operations, no ordering), `LinkedHashMap` (O(1) operations, insertion or access order), and `TreeMap` (O(log n) operations, sorted by key).

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Collections Framework Overview]]
> - [[Java - Set - HashSet TreeSet LinkedHashSet]]
> - [[Java - List - ArrayList LinkedList]]
> - [[Java - Abstraction - Abstract Classes and Interfaces]]

---

## Theory

### The `Map` Interface

A `Map` is an object that maps **keys** to **values**. The fundamental contract is:

1. **Unique keys**: A map cannot contain duplicate keys. Each key maps to at most one value. If you call `put(key, value)` with a key that already exists, the old value is replaced and returned.
2. **Values can be duplicated**: Multiple keys can map to the same value.
3. **Null handling**: `HashMap` and `LinkedHashMap` allow one null key and multiple null values. `TreeMap` does not allow null keys (because it uses `compareTo()` which throws `NullPointerException` on null).

**Core `Map` methods:**

| Method | Description | Return |
|--------|-------------|--------|
| `put(K key, V value)` | Associates the key with the value | `V` (old value, or null) |
| `get(Object key)` | Returns the value for the key | `V` (or null if not found) |
| `getOrDefault(K key, V default)` | Returns the value or a default | `V` |
| `remove(Object key)` | Removes the mapping for the key | `V` (removed value, or null) |
| `containsKey(Object key)` | Checks if the key exists | `boolean` |
| `containsValue(Object value)` | Checks if the value exists | `boolean` |
| `size()` | Returns the number of key-value pairs | `int` |
| `isEmpty()` | Returns true if the map is empty | `boolean` |
| `keySet()` | Returns a `Set` view of the keys | `Set<K>` |
| `values()` | Returns a `Collection` view of the values | `Collection<V>` |
| `entrySet()` | Returns a `Set` view of key-value pairs | `Set<Map.Entry<K,V>>` |
| `putAll(Map)` | Copies all mappings from another map | `void` |
| `clear()` | Removes all mappings | `void` |

**Java 8+ default methods (extremely important for backend code):**

| Method | Description | Use Case |
|--------|-------------|----------|
| `computeIfAbsent(K, Function)` | Computes and inserts a value only if the key is absent | Building maps of lists, caching |
| `computeIfPresent(K, BiFunction)` | Computes a new value only if the key is present | Conditional updates |
| `compute(K, BiFunction)` | Computes a new value regardless of presence | General-purpose updates |
| `merge(K, V, BiFunction)` | Merges a new value with an existing value | Counting, accumulating |
| `putIfAbsent(K, V)` | Inserts only if the key is not already mapped | Thread-safe initialization |
| `replace(K, V)` | Replaces the value only if the key exists | Safe updates |
| `forEach(BiConsumer)` | Iterates over all key-value pairs | Processing all entries |

### `HashMap` Internals

`HashMap` is the most commonly used `Map` implementation. It is backed by a **hash table** (an array of buckets, where each bucket holds a linked list or a balanced tree of entries).

**How it works step by step:**

**1. Hashing the key:**

When you call `map.put(key, value)`, the `HashMap` first computes the key's hash code by calling `key.hashCode()`. It then applies a supplemental hash function to reduce collisions:

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    // XOR the upper 16 bits with the lower 16 bits to spread the hash
}
```

**2. Selecting the bucket:**

The hash is mapped to a bucket index using a bitwise AND:

```java
int index = hash & (capacity - 1);
// capacity is always a power of 2, so this is equivalent to hash % capacity
// but much faster.
```

**3. Handling collisions:**

If the bucket is empty, a new `Node` is created. If the bucket already has entries, the `HashMap` traverses the linked list and checks each node's key using `==` (reference equality) and `equals()` (value equality). If a matching key is found, the value is replaced. If no match is found, a new node is appended to the list.

**4. Treeification (Java 8+):**

When a bucket's linked list grows beyond 8 nodes (and the total capacity is at least 64), the list is converted into a **red-black tree**. This improves worst-case lookup from O(n) to O(log n) for buckets with many collisions. When the tree shrinks below 6 nodes, it is converted back to a linked list.

**5. Resizing:**

When the number of entries exceeds `capacity * loadFactor` (default load factor is 0.75), the hash table doubles in capacity and all entries are redistributed into new buckets. This is an O(n) operation but happens infrequently.

**Memory layout:**

```text
HashMap<String, Integer> map = new HashMap<>(4);
map.put("Dhaka", 1);
map.put("Sylhet", 2);
map.put("Rajshahi", 3);

Internal state:
  capacity = 4, loadFactor = 0.75, size = 3, threshold = 3

  Bucket 0: null
  Bucket 1: [hash=... | "Rajshahi" -> 3 | next: null]
  Bucket 2: [hash=... | "Dhaka" -> 1 | next: -> [hash=... | "Sylhet" -> 2 | next: null]]
  Bucket 3: null
  (Dhaka and Sylhet collided in bucket 2)
```

**Performance:**

| Operation | Average | Worst Case |
|-----------|---------|------------|
| `put()` | O(1) | O(log n) with treeification, O(n) without |
| `get()` | O(1) | O(log n) with treeification, O(n) without |
| `remove()` | O(1) | O(log n) with treeification, O(n) without |
| `containsKey()` | O(1) | O(log n) |
| `containsValue()` | O(n) | O(n) (must scan all entries) |

### `LinkedHashMap` Internals

`LinkedHashMap` extends `HashMap` and adds a **doubly-linked list** that connects all entries in a specific order. It supports two ordering modes:

**1. Insertion order (default):** Entries are linked in the order they were first inserted. Iterating over the map returns entries in insertion order.

**2. Access order:** Entries are reordered on every `get()` and `put()` call so that the most recently accessed entry is at the tail. This mode is used to implement LRU (Least Recently Used) caches.

**How it works:**

- Each `Entry` node has two additional fields: `before` and `after`, which link it to the previous and next entries in the linked list.
- The `HashMap`'s hash table still provides O(1) `get()` and `put()`.
- The linked list adds a small memory overhead (two extra references per entry) but does not affect time complexity.
- The `removeEldestEntry()` method can be overridden to automatically evict the oldest entry when the map exceeds a certain size. This is the standard pattern for implementing an LRU cache.

**When to use `LinkedHashMap`:**

- When you need to preserve the insertion order of key-value pairs (e.g., maintaining the order of HTTP response headers, JSON fields, or configuration properties).
- When you need an LRU cache (with access-order mode and `removeEldestEntry()` override).
- When you need predictable iteration order for testing or debugging.

### `TreeMap` Internals

`TreeMap` is backed by a **red-black tree** (a self-balancing binary search tree). Entries are sorted by key according to their natural ordering (`Comparable`) or a custom `Comparator`.

**How it works:**

1. **Insertion:** The tree compares the new key with existing keys using `compareTo()` (or the `Comparator`) and places the new entry in the correct position to maintain sorted order. The tree then rebalances to ensure O(log n) height.

2. **Search:** The tree traverses from the root, going left or right based on the comparison. O(log n).

3. **Deletion:** The tree finds the node, removes it, and rebalances. O(log n).

4. **Range queries:** `TreeMap` supports efficient range operations: `subMap()`, `headMap()`, `tailMap()`, `firstKey()`, `lastKey()`, `higherKey()`, `lowerKey()`, `ceilingKey()`, `floorKey()`. These are impossible with `HashMap`.

**Performance:**

| Operation | Time |
|-----------|------|
| `put()` | O(log n) |
| `get()` | O(log n) |
| `remove()` | O(log n) |
| `containsKey()` | O(log n) |
| `firstKey()` / `lastKey()` | O(log n) |
| `subMap()` / `headMap()` / `tailMap()` | O(log n) to create the view |

**When to use `TreeMap`:**

- When you need keys in sorted order (e.g., a leaderboard, a timeline of events, a price index).
- When you need range queries (e.g., "find all orders between $100 and $500").
- When you need to find the nearest key (e.g., "find the closest timestamp to a given time").

### Comparison of All Three Implementations

| Feature | HashMap | LinkedHashMap | TreeMap |
|---------|---------|---------------|---------|
| Backing structure | Hash table | Hash table + linked list | Red-black tree |
| Key ordering | None (unpredictable) | Insertion or access order | Sorted (natural or custom) |
| `put()` | O(1) | O(1) | O(log n) |
| `get()` | O(1) | O(1) | O(log n) |
| `remove()` | O(1) | O(1) | O(log n) |
| Null keys | One null allowed | One null allowed | No null keys |
| Null values | Allowed | Allowed | Allowed |
| Memory overhead | Low | Medium | High |
| Range queries | No | No | Yes |
| LRU cache | No | Yes (access order) | No |
| Use case | Fast lookups | Ordered lookups, LRU cache | Sorted keys, range queries |

**Decision guide:**

- Need fast key-value lookups and do not care about order? Use `HashMap`.
- Need fast lookups AND insertion order? Use `LinkedHashMap`.
- Need sorted keys or range queries? Use `TreeMap`.
- Need an LRU cache? Use `LinkedHashMap` with access-order mode.
- Need thread safety? Use `ConcurrentHashMap` (hash-based, O(1), thread-safe).

### Iterating Over a Map

There are four ways to iterate over a map, each with different use cases:

```java
import java.util.Map;

Map<String, Integer> map = Map.of("A", 1, "B", 2, "C", 3);

// 1. Iterate over entries (BEST for accessing both key and value)
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}

// 2. Iterate over keys (when you only need keys)
for (String key : map.keySet()) {
    System.out.println(key);
}

// 3. Iterate over values (when you only need values)
for (Integer value : map.values()) {
    System.out.println(value);
}

// 4. forEach with lambda (Java 8+, cleanest syntax)
map.forEach((key, value) -> System.out.println(key + " -> " + value));
```

> [!tip] Key Insight
> The most important `Map` methods for backend development are the Java 8+ default methods: `computeIfAbsent()`, `merge()`, and `getOrDefault()`. These methods eliminate the most common map-related boilerplate (null checks, conditional inserts, and accumulation logic) and make your code significantly cleaner. `computeIfAbsent()` alone replaces the entire "check if key exists, create new list if not, add to list" pattern that used to require 4-5 lines of code.

---

## Syntax and Basic Examples

### Example 1: HashMap basics

```java
import java.util.*;

public class HashMapDemo {
    public static void main(String[] args) {
        // Creation
        Map<String, Double> prices = new HashMap<>();

        // Adding entries
        prices.put("Laptop", 85000.0);
        prices.put("Mouse", 1500.0);
        prices.put("Keyboard", 3200.0);
        prices.put("Monitor", 25000.0);

        // Overwriting an existing key
        Double oldPrice = prices.put("Laptop", 79999.0);
        System.out.println("Old laptop price: " + oldPrice);  // 85000.0

        System.out.println("Prices: " + prices);
        System.out.println("Size: " + prices.size());  // 4

        // Retrieving values
        System.out.println("Mouse: " + prices.get("Mouse"));        // 1500.0
        System.out.println("Webcam: " + prices.get("Webcam"));      // null (not found)
        System.out.println("Webcam (default): " + prices.getOrDefault("Webcam", 0.0));  // 0.0

        // Checking existence
        System.out.println("Has Laptop: " + prices.containsKey("Laptop"));    // true
        System.out.println("Has 1500: " + prices.containsValue(1500.0));      // true

        // Removing
        Double removed = prices.remove("Monitor");
        System.out.println("Removed: " + removed);  // 25000.0
        System.out.println("After remove: " + prices);

        // Null handling
        prices.put(null, 0.0);       // One null key is allowed
        prices.put("Free Item", null);  // Null values are allowed
        System.out.println("Null key value: " + prices.get(null));  // 0.0
    }
}
```

### Example 2: Java 8+ Map methods (the most important methods for backend code)

```java
import java.util.*;

public class MapJava8Demo {
    public static void main(String[] args) {
        // computeIfAbsent: the most useful Map method in backend development
        // Groups orders by user ID
        Map<Long, List<String>> userOrders = new HashMap<>();

        // Old way (before Java 8):
        // if (!userOrders.containsKey(userId)) {
        //     userOrders.put(userId, new ArrayList<>());
        // }
        // userOrders.get(userId).add(orderNumber);

        // New way (Java 8+):
        userOrders.computeIfAbsent(1L, k -> new ArrayList<>()).add("ORD-001");
        userOrders.computeIfAbsent(1L, k -> new ArrayList<>()).add("ORD-002");
        userOrders.computeIfAbsent(2L, k -> new ArrayList<>()).add("ORD-003");
        // computeIfAbsent creates the ArrayList only on the first call for each key.
        // Subsequent calls return the existing list.

        System.out.println("User orders: " + userOrders);
        // {1=[ORD-001, ORD-002], 2=[ORD-003]}

        // merge: combines a new value with an existing value
        // Counts word frequencies
        Map<String, Integer> wordCount = new HashMap<>();
        String[] words = {"java", "spring", "java", "backend", "spring", "java"};

        for (String word : words) {
            wordCount.merge(word, 1, Integer::sum);
            // If 'word' is absent: puts (word, 1)
            // If 'word' is present: replaces with Integer.sum(oldValue, 1)
        }
        System.out.println("Word count: " + wordCount);
        // {java=3, spring=2, backend=1}

        // computeIfPresent: updates only if the key exists
        Map<String, Double> cart = new HashMap<>();
        cart.put("Laptop", 85000.0);
        cart.put("Mouse", 1500.0);

        // Apply 10% discount to Laptop (only if it exists in the cart)
        cart.computeIfPresent("Laptop", (key, price) -> price * 0.90);
        cart.computeIfPresent("Monitor", (key, price) -> price * 0.90);  // No effect, key absent
        System.out.println("Cart after discount: " + cart);
        // {Laptop=76500.0, Mouse=1500.0}

        // putIfAbsent: inserts only if the key is not already mapped
        Map<String, String> config = new HashMap<>();
        config.put("db.host", "localhost");
        config.putIfAbsent("db.host", "production-server");  // No effect, key exists
        config.putIfAbsent("db.port", "5432");  // Inserted, key was absent
        System.out.println("Config: " + config);
        // {db.host=localhost, db.port=5432}

        // replace: replaces the value only if the key exists
        config.replace("db.host", "staging-server");  // Replaces
        config.replace("db.name", "orders");  // No effect, key absent
        System.out.println("Config after replace: " + config);

        // forEach: iterate with a lambda
        System.out.println("\n--- Cart Items ---");
        cart.forEach((item, price) ->
            System.out.printf("  %-10s: %,.2f BDT%n", item, price)
        );
    }
}
```

### Example 3: LinkedHashMap for insertion order and LRU cache

```java
import java.util.*;

public class LinkedHashMapDemo {
    public static void main(String[] args) {
        // Insertion order (default)
        Map<String, Integer> insertionOrder = new LinkedHashMap<>();
        insertionOrder.put("Dhaka", 1);
        insertionOrder.put("Chittagong", 2);
        insertionOrder.put("Sylhet", 3);
        insertionOrder.put("Rajshahi", 4);

        System.out.println("Insertion order: " + insertionOrder);
        // {Dhaka=1, Chittagong=2, Sylhet=3, Rajshahi=4} (order preserved)

        // Compare with HashMap (order is unpredictable)
        Map<String, Integer> noOrder = new HashMap<>(insertionOrder);
        System.out.println("HashMap order: " + noOrder);
        // {Sylhet=3, Dhaka=1, Rajshahi=4, Chittagong=2} (order may vary)

        // Access order (LRU cache pattern)
        // Constructor: LinkedHashMap(initialCapacity, loadFactor, accessOrder)
        Map<String, String> lruCache = new LinkedHashMap<>(16, 0.75f, true) {
            private static final int MAX_SIZE = 3;

            @Override
            protected boolean removeEldestEntry(Map.Entry<String, String> eldest) {
                boolean shouldRemove = size() > MAX_SIZE;
                if (shouldRemove) {
                    System.out.println("  Evicting: " + eldest.getKey());
                }
                return shouldRemove;
            }
        };

        System.out.println("\n--- LRU Cache (max 3 entries) ---");
        lruCache.put("user:1", "Alice");
        lruCache.put("user:2", "Bob");
        lruCache.put("user:3", "Charlie");
        System.out.println("Cache: " + lruCache);

        // Access user:1, moving it to the end (most recently used)
        lruCache.get("user:1");
        System.out.println("After accessing user:1: " + lruCache);

        // Add user:4, which triggers eviction of the least recently used (user:2)
        lruCache.put("user:4", "Dave");
        System.out.println("After adding user:4: " + lruCache);
        // user:2 was evicted because it was the least recently accessed
    }
}
```

### Example 4: TreeMap for sorted keys and range queries

```java
import java.util.*;

public class TreeMapDemo {
    public static void main(String[] args) {
        // Natural ordering (String keys sorted alphabetically)
        TreeMap<String, Double> sortedPrices = new TreeMap<>();
        sortedPrices.put("Monitor", 25000.0);
        sortedPrices.put("Laptop", 85000.0);
        sortedPrices.put("Keyboard", 3200.0);
        sortedPrices.put("Mouse", 1500.0);
        sortedPrices.put("Webcam", 4500.0);

        System.out.println("Sorted prices: " + sortedPrices);
        // {Keyboard=3200.0, Laptop=85000.0, Monitor=25000.0, Mouse=1500.0, Webcam=4500.0}

        // First and last keys
        System.out.println("First: " + sortedPrices.firstKey());  // Keyboard
        System.out.println("Last: " + sortedPrices.lastKey());    // Webcam

        // Range queries
        System.out.println("\nSubMap (Laptop to Mouse): " +
            sortedPrices.subMap("Laptop", "Mouse"));
        // {Laptop=85000.0, Monitor=25000.0} (from inclusive, to exclusive)

        System.out.println("HeadMap (before Monitor): " +
            sortedPrices.headMap("Monitor"));
        // {Keyboard=3200.0, Laptop=85000.0}

        System.out.println("TailMap (from Monitor): " +
            sortedPrices.tailMap("Monitor"));
        // {Monitor=25000.0, Mouse=1500.0, Webcam=4500.0}

        // Nearest key queries
        System.out.println("\nHigher than 'Laptop': " + sortedPrices.higherKey("Laptop"));
        // Monitor
        System.out.println("Lower than 'Monitor': " + sortedPrices.lowerKey("Monitor"));
        // Laptop
        System.out.println("Ceiling of 'M': " + sortedPrices.ceilingKey("M"));
        // Monitor (smallest key >= "M")
        System.out.println("Floor of 'M': " + sortedPrices.floorKey("M"));
        // Laptop (largest key <= "M")

        // Custom comparator: reverse alphabetical
        TreeMap<String, Double> reversePrices = new TreeMap<>(Comparator.reverseOrder());
        reversePrices.putAll(sortedPrices);
        System.out.println("\nReverse order: " + reversePrices);
        // {Webcam=4500.0, Mouse=1500.0, Monitor=25000.0, Laptop=85000.0, Keyboard=3200.0}

        // TreeMap does NOT allow null keys
        try {
            sortedPrices.put(null, 0.0);  // NullPointerException!
        } catch (NullPointerException e) {
            System.out.println("\nTreeMap does not allow null keys");
        }
    }
}
```

### Example 5: Map views and iteration patterns

```java
import java.util.*;

public class MapViewsDemo {
    public static void main(String[] args) {
        Map<String, Integer> scores = new LinkedHashMap<>();
        scores.put("Saad", 92);
        scores.put("Rahim", 78);
        scores.put("Karim", 85);
        scores.put("Nila", 95);

        // keySet(): returns a Set VIEW of the keys
        Set<String> names = scores.keySet();
        System.out.println("Names: " + names);  // [Saad, Rahim, Karim, Nila]
        // WARNING: This is a VIEW, not a copy. Modifying the map affects the set.
        scores.put("Arif", 88);
        System.out.println("Names after map change: " + names);  // Includes Arif

        // values(): returns a Collection VIEW of the values
        Collection<Integer> allScores = scores.values();
        System.out.println("Scores: " + allScores);

        // entrySet(): returns a Set VIEW of key-value pairs (most efficient for iteration)
        System.out.println("\n--- All Entries ---");
        for (Map.Entry<String, Integer> entry : scores.entrySet()) {
            System.out.printf("  %s: %d%n", entry.getKey(), entry.getValue());
        }

        // Modifying through the entry set
        for (Map.Entry<String, Integer> entry : scores.entrySet()) {
            if (entry.getValue() < 80) {
                entry.setValue(entry.getValue() + 5);  // Bonus 5 marks
            }
        }
        System.out.println("\nAfter bonus: " + scores);

        // Immutable map (Java 9+)
        Map<String, Integer> immutable = Map.of("A", 1, "B", 2, "C", 3);
        // immutable.put("D", 4);  // UnsupportedOperationException!

        // Map.ofEntries for more than 10 entries
        Map<String, Integer> largeImmutable = Map.ofEntries(
            Map.entry("A", 1), Map.entry("B", 2), Map.entry("C", 3),
            Map.entry("D", 4), Map.entry("E", 5), Map.entry("F", 6)
        );
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Maps are the backbone of every Spring Boot application. Here are three realistic scenarios.

### Scenario 1: Configuration properties as maps

Spring Boot loads configuration from `application.yml` into `Map` objects for flexible, key-value-based configuration.

```java
package com.company.orderservice.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import java.util.HashMap;
import java.util.Map;

@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    // Loaded from application.yml:
    // app:
    //   rate-limits:
    //     default: 100
    //     premium: 1000
    //     admin: 10000
    private Map<String, Integer> rateLimits = new HashMap<>();

    // Loaded from application.yml:
    // app:
    //   feature-flags:
    //     dark-mode: true
    //     new-checkout: false
    //     free-shipping: true
    private Map<String, Boolean> featureFlags = new HashMap<>();

    // Loaded from application.yml:
    // app:
    //   payment-gateways:
    //     stripe:
    //       api-key: sk_test_...
    //       webhook-secret: whsec_...
    //     bkash:
    //       api-key: bk_test_...
    //       webhook-secret: bk_whsec_...
    private Map<String, Map<String, String>> paymentGateways = new HashMap<>();

    public int getRateLimit(String tier) {
        return rateLimits.getOrDefault(tier, rateLimits.getOrDefault("default", 100));
    }

    public boolean isFeatureEnabled(String feature) {
        return featureFlags.getOrDefault(feature, false);
    }

    public String getGatewayApiKey(String gateway) {
        Map<String, String> config = paymentGateways.get(gateway);
        if (config == null) {
            throw new IllegalArgumentException("Unknown payment gateway: " + gateway);
        }
        return config.get("api-key");
    }

    // Getters and setters for Spring Boot property binding
    public Map<String, Integer> getRateLimits() { return rateLimits; }
    public void setRateLimits(Map<String, Integer> rateLimits) { this.rateLimits = rateLimits; }
    public Map<String, Boolean> getFeatureFlags() { return featureFlags; }
    public void setFeatureFlags(Map<String, Boolean> featureFlags) { this.featureFlags = featureFlags; }
    public Map<String, Map<String, String>> getPaymentGateways() { return paymentGateways; }
    public void setPaymentGateways(Map<String, Map<String, String>> paymentGateways) {
        this.paymentGateways = paymentGateways;
    }
}
```

**What to notice:**

- `Map<String, Integer>` and `Map<String, Boolean>` are used for flat key-value configuration. Spring Boot automatically maps YAML keys to map keys and YAML values to map values.
- `Map<String, Map<String, String>>` is used for nested configuration. This allows each payment gateway to have its own set of configuration properties without creating a separate class for each gateway.
- `getOrDefault()` provides fallback values when a key is not present. This prevents `NullPointerException` and allows the application to function with partial configuration.

### Scenario 2: Grouping and aggregation in service layers

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.TreeMap;

@Service
public class ReportService {

    private final OrderRepository orderRepository;

    public ReportService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    // Group orders by status and count them
    public Map<OrderStatus, Long> getOrderCountByStatus() {
        List<Order> orders = orderRepository.findAll();

        Map<OrderStatus, Long> countByStatus = new HashMap<>();
        for (Order order : orders) {
            countByStatus.merge(order.getStatus(), 1L, Long::sum);
            // merge() is perfect for counting:
            // - First occurrence: puts (status, 1)
            // - Subsequent occurrences: adds 1 to the existing count
        }
        return countByStatus;
        // {PENDING=15, CONFIRMED=42, SHIPPED=28, DELIVERED=156, CANCELLED=8}
    }

    // Group orders by user and calculate total spending per user
    public Map<Long, BigDecimal> getTotalSpendingByUser() {
        List<Order> orders = orderRepository.findByStatus(OrderStatus.DELIVERED);

        Map<Long, BigDecimal> spendingByUser = new HashMap<>();
        for (Order order : orders) {
            spendingByUser.merge(
                order.getUserId(),
                order.getTotalAmount(),
                BigDecimal::add
            );
            // merge() accumulates the total:
            // - First order for this user: puts (userId, orderTotal)
            // - Subsequent orders: adds orderTotal to the existing sum
        }
        return spendingByUser;
        // {1=15000.00, 2=8500.50, 3=42000.00}
    }

    // Group orders by month for a revenue chart
    public Map<String, BigDecimal> getMonthlyRevenue(int year) {
        List<Order> orders = orderRepository.findByYearAndStatus(year, OrderStatus.DELIVERED);

        // TreeMap ensures the months are sorted chronologically
        Map<String, BigDecimal> monthlyRevenue = new TreeMap<>();

        for (Order order : orders) {
            String monthKey = order.getCreatedAt().getMonth().name();
            monthlyRevenue.merge(monthKey, order.getTotalAmount(), BigDecimal::add);
        }
        return monthlyRevenue;
        // {JANUARY=125000.00, FEBRUARY=98000.00, MARCH=145000.00, ...}
        // TreeMap ensures alphabetical (and roughly chronological) ordering
    }

    // Build a lookup map for fast access by order number
    public Map<String, Order> getOrderLookup(List<Order> orders) {
        Map<String, Order> lookup = new HashMap<>();
        for (Order order : orders) {
            lookup.put(order.getOrderNumber(), order);
        }
        return lookup;
        // O(1) lookup by order number instead of O(n) list scan
    }

    // Group orders by status with full order lists (using computeIfAbsent)
    public Map<OrderStatus, List<Order>> groupOrdersByStatus(List<Order> orders) {
        Map<OrderStatus, List<Order>> grouped = new HashMap<>();
        for (Order order : orders) {
            grouped.computeIfAbsent(order.getStatus(), k -> new ArrayList<>()).add(order);
        }
        return grouped;
        // {PENDING=[Order1, Order5], CONFIRMED=[Order2, Order3, Order7], ...}
    }
}
```

**What to notice:**

- `merge()` is the most powerful method for aggregation. It handles both the "first insertion" and "subsequent accumulation" cases in a single call. The `BigDecimal::add` method reference makes the accumulation logic clean and readable.
- `computeIfAbsent()` is the standard pattern for grouping elements into a map of lists. It creates the list only when a new key is encountered, eliminating the null-check boilerplate.
- `TreeMap` is used for `getMonthlyRevenue()` to ensure the months are returned in sorted order. This is important for chart rendering on the frontend.

### Scenario 3: HTTP header processing and JSON deserialization

```java
package com.company.orderservice.controller;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.http.ResponseEntity;
import java.math.BigDecimal;
import java.util.Map;

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    // Spring Boot deserializes JSON objects into Map<String, Object>
    // when the structure is dynamic or unknown at compile time.
    @PostMapping("/webhook")
    public ResponseEntity<String> handleWebhook(
            @RequestHeader Map<String, String> headers,
            @RequestBody Map<String, Object> payload) {

        // Access HTTP headers as a map
        String contentType = headers.getOrDefault("content-type", "application/json");
        String signature = headers.get("x-webhook-signature");

        if (signature == null) {
            return ResponseEntity.status(401).body("Missing signature");
        }

        // Access the dynamic JSON payload as a map
        String eventType = (String) payload.getOrDefault("type", "unknown");
        Map<String, Object> data = (Map<String, Object>) payload.get("data");

        if (data == null) {
            return ResponseEntity.badRequest().body("Missing data field");
        }

        // Process based on event type
        switch (eventType) {
            case "payment.completed" -> {
                String transactionId = (String) data.get("transaction_id");
                Double amount = (Double) data.get("amount");
                orderService.confirmPayment(transactionId, BigDecimal.valueOf(amount));
            }
            case "order.shipped" -> {
                String orderNumber = (String) data.get("order_number");
                String trackingNumber = (String) data.get("tracking_number");
                orderService.updateTracking(orderNumber, trackingNumber);
            }
            default -> {
                // logger.warn("Unknown webhook event type: {}", eventType);
            }
        }

        return ResponseEntity.ok("Webhook processed");
    }
}
```

**What to notice:**

- `@RequestHeader Map<String, String> headers` tells Spring to populate a map with all HTTP request headers. Header names are keys, header values are values. This is useful when you need to access dynamic or optional headers.
- `@RequestBody Map<String, Object> payload` tells Spring to deserialize the JSON body into a generic map. This is used when the JSON structure is dynamic or when you do not want to create a dedicated DTO class.
- `getOrDefault()` provides safe access to map entries that might not exist. This prevents `NullPointerException` when processing webhooks from external services that may send inconsistent payloads.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using mutable objects as map keys

**Wrong:**

```java
Map<List<String>, Integer> cache = new HashMap<>();
List<String> key = new ArrayList<>(List.of("java", "spring"));
cache.put(key, 42);

// Modifying the key AFTER inserting it into the map
key.add("hibernate");  // The key's hashCode changes!

System.out.println(cache.get(key));  // null! The map cannot find the entry.
// The entry is in the bucket for the old hash code, but the lookup
// uses the new hash code. The entry is lost.
```

**Right:**

```java
// Use immutable objects as map keys
Map<List<String>, Integer> cache = new HashMap<>();
List<String> key = List.of("java", "spring");  // Immutable list
cache.put(key, 42);

System.out.println(cache.get(key));  // 42 (works correctly)
// The key cannot be modified, so its hash code is stable.
```

**Why it is wrong:** This is the same issue as with `HashSet` (covered in the Set note). Map keys are stored in hash buckets based on their hash code. If the key's hash code changes after insertion, the map looks in the wrong bucket and cannot find the entry. Always use immutable objects as map keys: `String`, `Integer`, `Long`, `UUID`, or custom immutable classes with `final` fields.

### Mistake 2: Not handling null returns from `get()`

**Wrong:**

```java
Map<String, User> userCache = getUserCache();
User user = userCache.get(userId);
System.out.println(user.getName());  // NullPointerException if userId is not in the cache!
```

**Right:**

```java
// Option 1: Use getOrDefault()
User user = userCache.getOrDefault(userId, User.ANONYMOUS);

// Option 2: Check for null
User user = userCache.get(userId);
if (user == null) {
    throw new ResourceNotFoundException("User", userId);
}

// Option 3: Use computeIfAbsent() to load on cache miss
User user = userCache.computeIfAbsent(userId, id -> userRepository.findById(id));
```

**Why it is wrong:** `Map.get()` returns `null` when the key is not found. If you immediately call a method on the returned value without checking for null, you get a `NullPointerException`. This is one of the most common bugs in backend code. Always handle the "key not found" case explicitly using `getOrDefault()`, a null check, or `computeIfAbsent()`.

### Mistake 3: Using `containsKey()` + `get()` instead of `getOrDefault()` or `computeIfAbsent()`

**Wrong:**

```java
// Verbose and inefficient: two hash lookups instead of one
if (map.containsKey(key)) {
    List<String> list = map.get(key);  // Second hash lookup!
    list.add(value);
} else {
    List<String> list = new ArrayList<>();
    list.add(value);
    map.put(key, list);
}
```

**Right:**

```java
// Single hash lookup, clean and efficient
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

**Why it is wrong:** The `containsKey()` + `get()` pattern performs two hash lookups for the same key, which is wasteful. `computeIfAbsent()` performs a single lookup and handles both the "present" and "absent" cases in one call. This is not just cleaner; it is measurably faster for large maps.

### Mistake 4: Iterating over `keySet()` and calling `get()` for each key

**Wrong:**

```java
for (String key : map.keySet()) {
    Integer value = map.get(key);  // Extra hash lookup for every key!
    System.out.println(key + " -> " + value);
}
```

**Right:**

```java
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
    // No extra lookup: the entry already contains both key and value.
}

// Or even cleaner with forEach:
map.forEach((key, value) -> System.out.println(key + " -> " + value));
```

**Why it is wrong:** Iterating over `keySet()` and calling `get()` for each key performs an unnecessary hash lookup on every iteration. The `entrySet()` view provides direct access to both the key and the value without any additional lookups. For a map with 1 million entries, this difference is significant.

---

## Key Takeaways

> [!tip] Remember these points
> 1. `Map` stores key-value pairs with unique keys. It does not extend `Collection`. The three primary implementations are `HashMap` (O(1), unordered), `LinkedHashMap` (O(1), insertion/access order), and `TreeMap` (O(log n), sorted keys).
> 2. **`HashMap`** is the default choice for most backend use cases. It provides O(1) `get()` and `put()` using a hash table with collision handling via linked lists and red-black trees (Java 8+). It allows one null key and multiple null values.
> 3. **`LinkedHashMap`** preserves insertion order by default and can be configured for access order to implement LRU caches. Override `removeEldestEntry()` to automatically evict old entries when the map exceeds a size limit.
> 4. **`TreeMap`** maintains keys in sorted order and supports range queries (`subMap`, `headMap`, `tailMap`) and nearest-key queries (`higherKey`, `lowerKey`, `ceilingKey`, `floorKey`). It does not allow null keys. Use it when sorted order or range queries are required.
> 5. The Java 8+ methods `computeIfAbsent()`, `merge()`, `getOrDefault()`, and `forEach()` are the most important `Map` methods for backend development. They eliminate null-check boilerplate, simplify grouping and aggregation, and make map operations more readable and efficient. Always use `entrySet()` or `forEach()` for iteration, not `keySet()` + `get()`.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Word Frequency Counter (Easy)

Write a program that takes a paragraph of text as a String, splits it into words, and counts the frequency of each word using a `HashMap`. Ignore case and punctuation. Print the results sorted by frequency (highest first).

Example input: `"Java is great. Java is fast. Spring Boot is built with Java."`
Expected output: `java=3, is=3, great=1, fast=1, spring=1, boot=1, built=1, with=1`

> **Hint:** Use `String.toLowerCase().replaceAll("[^a-z\\s]", "")` to clean the text. Use `split("\\s+")` to split into words. Use `merge(word, 1, Integer::sum)` to count frequencies. Sort the entries using a `List` and `Comparator`.

### Exercise 2: Student Grade Book with Maps (Medium)

Create a grade book system using nested maps:

1. Use a `Map<String, Map<String, Double>>` where the outer key is the student name and the inner map maps subject names to grades.
2. Implement methods:
   - `addGrade(String student, String subject, double grade)`: adds a grade using `computeIfAbsent()`.
   - `getAverage(String student)`: calculates the average grade for a student.
   - `getSubjectAverage(String subject)`: calculates the average grade across all students for a subject.
   - `getTopStudent()`: returns the student with the highest overall average.
3. Add grades for 5 students across 4 subjects and test all methods.

> **Hint:** For `addGrade()`, use `gradeBook.computeIfAbsent(student, k -> new HashMap<>()).put(subject, grade)`. For `getSubjectAverage()`, iterate over all students' inner maps and collect grades for the target subject.

### Exercise 3: LRU Cache Implementation (Medium)

Implement a fully functional LRU (Least Recently Used) cache using `LinkedHashMap`:

1. Create a class `LRUCache<K, V>` with a constructor that takes the maximum capacity.
2. Implement `get(K key)`: returns the value and marks the entry as recently used. Returns null if the key is not found.
3. Implement `put(K key, V value)`: inserts or updates the entry. If the cache exceeds capacity, evicts the least recently used entry.
4. Implement `size()` and `containsKey(K key)`.
5. In `main()`, test the cache with a capacity of 3. Insert 5 entries, access some entries, and verify that the correct entries are evicted.

> **Hint:** Extend `LinkedHashMap<K, V>` with `accessOrder = true` and override `removeEldestEntry()`. Wrap it in your `LRUCache` class to hide the `LinkedHashMap` implementation details.

### Exercise 4: E-Commerce Analytics Dashboard (Hard, Optional)

Build a simplified analytics service that processes a list of orders and produces various reports using maps:

1. Create an `Order` record with fields `orderId`, `userId`, `category`, `amount`, `status`, `createdAt`.
2. Generate 30 random orders with varying data.
3. Implement the following reports using maps:
   - **Revenue by category**: `Map<String, BigDecimal>` using `merge()`.
   - **Order count by status**: `Map<String, Long>` using `merge()`.
   - **Top 5 customers by spending**: Use a `Map<Long, BigDecimal>` to aggregate, then sort and take the top 5.
   - **Monthly revenue trend**: `TreeMap<String, BigDecimal>` sorted by month.
   - **Category breakdown per user**: `Map<Long, Map<String, BigDecimal>>` using nested `computeIfAbsent()`.
4. Print all reports in a formatted manner.

> **Hint:** The nested map for category breakdown per user requires two levels of `computeIfAbsent()`: `map.computeIfAbsent(userId, k -> new HashMap<>()).merge(category, amount, BigDecimal::add)`.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.*;

public class WordFrequency {
    public static void main(String[] args) {
        String text = "Java is great. Java is fast. Spring Boot is built with Java.";
        String[] words = text.toLowerCase().replaceAll("[^a-z\\s]", "").split("\\s+");

        Map<String, Integer> freq = new HashMap<>();
        for (String word : words) {
            if (!word.isEmpty()) {
                freq.merge(word, 1, Integer::sum);
            }
        }

        // Sort by frequency descending
        List<Map.Entry<String, Integer>> sorted = new ArrayList<>(freq.entrySet());
        sorted.sort(Map.Entry.<String, Integer>comparingByValue().reversed());

        System.out.println("Word frequencies:");
        for (var entry : sorted) {
            System.out.printf("  %-10s: %d%n", entry.getKey(), entry.getValue());
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
import java.util.*;

public class LRUCache<K, V> {
    private final LinkedHashMap<K, V> map;
    private final int capacity;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new LinkedHashMap<>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > LRUCache.this.capacity;
            }
        };
    }

    public V get(K key) {
        return map.get(key);  // Access-order mode moves this to the end
    }

    public void put(K key, V value) {
        map.put(key, value);  // May trigger eviction via removeEldestEntry
    }

    public int size() { return map.size(); }
    public boolean containsKey(K key) { return map.containsKey(key); }

    @Override
    public String toString() { return map.toString(); }

    public static void main(String[] args) {
        LRUCache<String, Integer> cache = new LRUCache<>(3);
        cache.put("A", 1);
        cache.put("B", 2);
        cache.put("C", 3);
        System.out.println("Cache: " + cache);  // {A=1, B=2, C=3}

        cache.get("A");  // Access A, making it most recently used
        cache.put("D", 4);  // Evicts B (least recently used)
        System.out.println("After D: " + cache);  // {C=3, A=1, D=4}

        cache.put("E", 5);  // Evicts C
        System.out.println("After E: " + cache);  // {A=1, D=4, E=5}
    }
}
```

</details>

---

## Related Notes

- [[Java - Set - HashSet TreeSet LinkedHashSet]]
- [[Java - Queue and Deque]] (next note)
- [[Java - Comparable and Comparator]]
- [[Java - Java 8 Streams API]]

---

## Resources

- [Oracle Java Tutorials: The Map Interface](https://docs.oracle.com/javase/tutorial/collections/interfaces/map.html) - Official documentation covering the Map contract and all implementations.
- [Oracle Java Documentation: HashMap](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html) - Complete API reference with implementation details.
- [Baeldung: Java Map Guide](https://www.baeldung.com/java-map) - Comprehensive guide covering all Map operations and implementations.
- [Baeldung: HashMap vs TreeMap vs LinkedHashMap](https://www.baeldung.com/java-hashmap-vs-treemap-vs-linkedhashmap) - Detailed comparison with performance benchmarks.
- [Baeldung: Java computeIfAbsent](https://www.baeldung.com/java-map-computeifabsent) - Deep dive into the most useful Map method for backend development.
- [Baeldung: Java Map merge](https://www.baeldung.com/java-map-merge) - Guide to the merge method with practical examples for counting and aggregation.
- [Effective Java by Joshua Bloch - Item 11: Always Override hashCode When You Override equals](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Critical for using objects as map keys.
