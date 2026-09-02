## Overview

The Java Collections Framework is the backbone of data manipulation in Java. Every backend system you build will use collections to store, retrieve, sort, filter, and transform data. Whether you are processing a list of transactions, caching user sessions in a map, deduplicating events with a set, or managing a task queue, you are using the Collections Framework. Understanding the performance characteristics, thread safety guarantees, and appropriate use cases for each collection type is what separates engineers who write correct, performant code from those who introduce subtle bugs and bottlenecks.

---

## Core Concepts

### The Collection Hierarchy

The framework is organized around two root interfaces: `Collection` and `Map`.

```
Iterable
└── Collection<E>
    ├── List<E>                    (ordered, allows duplicates)
    │   ├── ArrayList<E>           (dynamic array, fast random access)
    │   ├── LinkedList<E>          (doubly-linked list, fast insert/remove at ends)
    │   └── Vector<E>              (legacy, synchronized — avoid)
    │
    ├── Set<E>                     (unordered, no duplicates)
    │   ├── HashSet<E>             (hash table, O(1) operations)
    │   ├── LinkedHashSet<E>       (hash table + linked list, insertion order)
    │   └── TreeSet<E>             (red-black tree, sorted, O(log n))
    │
    └── Queue<E>                   (FIFO processing)
        ├── PriorityQueue<E>       (heap, priority-based ordering)
        ├── ArrayDeque<E>          (resizable array, stack or queue)
        └── LinkedList<E>          (also implements Queue)

Map<K, V>                          (key-value pairs, NOT a Collection)
├── HashMap<K, V>                  (hash table, O(1) average)
├── LinkedHashMap<K, V>            (hash table + linked list, insertion/access order)
├── TreeMap<K, V>                  (red-black tree, sorted by key, O(log n))
├── Hashtable<K, V>                (legacy, synchronized — avoid)
└── ConcurrentHashMap<K, V>        (thread-safe, high concurrency)
```

### List

A `List` is an ordered collection that allows duplicate elements. Elements are accessed by integer index (zero-based).

**ArrayList:**

The default choice for most use cases. Backed by a dynamically resizing array.

```java
// Creation
List<String> names = new ArrayList<>();
List<String> namesWithCapacity = new ArrayList<>(100);  // Pre-allocate for known size
List<String> fromArray = new ArrayList<>(Arrays.asList("Alice", "Bob", "Charlie"));

// Adding
names.add("Alice");              // Append to end — O(1) amortized
names.add(0, "Zara");            // Insert at index — O(n), shifts elements
names.addAll(List.of("Bob", "Charlie"));  // Append all

// Accessing
String first = names.get(0);     // O(1) — direct array access
String last = names.get(names.size() - 1);

// Searching
boolean hasAlice = names.contains("Alice");  // O(n) — linear scan
int index = names.indexOf("Bob");            // O(n) — returns first occurrence
int lastIndex = names.lastIndexOf("Bob");    // O(n) — returns last occurrence

// Modifying
names.set(0, "Alicia");          // Replace at index — O(1)
names.remove("Alice");           // Remove by value — O(n), shifts elements
names.remove(0);                 // Remove by index — O(n), shifts elements
names.removeAll(List.of("Bob")); // Remove all occurrences of elements in collection

// Size and state
int size = names.size();         // O(1)
boolean empty = names.isEmpty(); // O(1)

// Sublist (view of the original list — modifications affect both)
List<String> subset = names.subList(1, 3);  // Elements at index 1 and 2

// Sorting
names.sort(Comparator.naturalOrder());       // In-place sort
names.sort(Comparator.reverseOrder());
names.sort(Comparator.comparing(String::length));

// Iteration
for (String name : names) {
    System.out.println(name);
}

names.forEach(name -> System.out.println(name));
names.forEach(System.out::println);  // Method reference
```

**Performance characteristics of ArrayList:**

| Operation | Time Complexity | Notes |
|-----------|----------------|-------|
| `get(index)` | O(1) | Direct array access |
| `set(index, value)` | O(1) | Direct array access |
| `add(value)` | O(1) amortized | O(n) when resizing |
| `add(index, value)` | O(n) | Shifts elements right |
| `remove(index)` | O(n) | Shifts elements left |
| `remove(value)` | O(n) | Linear search + shift |
| `contains(value)` | O(n) | Linear search |
| `indexOf(value)` | O(n) | Linear search |
| `size()` | O(1) | Stored field |
| `sort()` | O(n log n) | TimSort |

**Internal resizing mechanism:**

When an `ArrayList` exceeds its capacity, it creates a new array with 1.5x the current capacity and copies all elements. This is why pre-allocating with `new ArrayList<>(expectedSize)` improves performance when you know the approximate number of elements.

```java
// Bad: starts with capacity 10, resizes multiple times
List<Transaction> transactions = new ArrayList<>();
for (int i = 0; i < 100_000; i++) {
    transactions.add(fetchTransaction(i));  // Multiple resizes
}

// Good: pre-allocate
List<Transaction> transactions = new ArrayList<>(100_000);
for (int i = 0; i < 100_000; i++) {
    transactions.add(fetchTransaction(i));  // No resizes
}
```

**LinkedList:**

Backed by a doubly-linked list. Each element is a node with pointers to the previous and next nodes.

```java
List<String> linked = new LinkedList<>();
linked.add("Alice");
linked.addFirst("Zara");    // O(1) — LinkedList-specific
linked.addLast("Bob");      // O(1)
linked.removeFirst();        // O(1)
linked.removeLast();         // O(1)
```

**Performance characteristics of LinkedList:**

| Operation | Time Complexity | Notes |
|-----------|----------------|-------|
| `get(index)` | O(n) | Must traverse from head or tail |
| `add(value)` | O(1) | Append to tail |
| `add(index, value)` | O(n) | Must traverse to index, then O(1) insert |
| `addFirst(value)` | O(1) | Direct pointer manipulation |
| `addLast(value)` | O(1) | Direct pointer manipulation |
| `removeFirst()` | O(1) | Direct pointer manipulation |
| `removeLast()` | O(1) | Direct pointer manipulation |
| `remove(index)` | O(n) | Must traverse to index |
| `contains(value)` | O(n) | Linear traversal |

**ArrayList vs LinkedList:**

In practice, `ArrayList` outperforms `LinkedList` in almost every scenario, even for insertions and deletions in the middle of the list. This is because `ArrayList` benefits from CPU cache locality (contiguous memory), while `LinkedList` suffers from pointer chasing (each node is scattered across the heap). `LinkedList` also has significantly higher memory overhead (two pointers per element plus object header).

Use `ArrayList` as your default. Use `LinkedList` only when you specifically need a deque (double-ended queue) and even then, prefer `ArrayDeque`.

### Set

A `Set` is a collection that contains no duplicate elements. It models the mathematical set abstraction.

**HashSet:**

The default choice. Backed by a `HashMap`. Offers O(1) average performance for add, remove, and contains.

```java
Set<String> currencies = new HashSet<>();
currencies.add("USD");           // O(1) average
currencies.add("EUR");
currencies.add("GBP");
currencies.add("USD");           // Duplicate — ignored
System.out.println(currencies.size());  // 3

boolean hasUsd = currencies.contains("USD");  // O(1) average
currencies.remove("GBP");                     // O(1) average

// Iteration order is NOT guaranteed
for (String currency : currencies) {
    System.out.println(currency);  // Order may vary between runs
}
```

**How HashSet works internally:**

`HashSet` uses a `HashMap` under the hood. Each element is stored as a key in the map with a dummy value (`PRESENT`). This means:

- Elements must implement `hashCode()` and `equals()` correctly.
- If two objects are equal according to `equals()`, they must have the same `hashCode()`.
- If you modify an object's fields after adding it to a `HashSet`, and those fields are used in `hashCode()`, the object becomes "lost" in the set.

```java
// DANGER: Mutable object in a HashSet
Account account = new Account("ACC-001", "Alice");
Set<Account> accounts = new HashSet<>();
accounts.add(account);

account.setName("Bob");  // If name is part of hashCode, the account is now lost
System.out.println(accounts.contains(account));  // May return false!
```

**LinkedHashSet:**

A `HashSet` that maintains insertion order. Slightly slower than `HashSet` due to the linked list overhead, but the ordering guarantee is valuable.

```java
Set<String> ordered = new LinkedHashSet<>();
ordered.add("USD");
ordered.add("EUR");
ordered.add("GBP");

for (String currency : ordered) {
    System.out.println(currency);
}
// Guaranteed output: USD, EUR, GBP (insertion order)
```

Use `LinkedHashSet` when you need uniqueness AND predictable iteration order. Common in caching and deduplication scenarios where order matters.

**TreeSet:**

A sorted set backed by a `TreeMap` (red-black tree). Elements are maintained in sorted order. All operations are O(log n).

```java
Set<BigDecimal> prices = new TreeSet<>();
prices.add(new BigDecimal("29.99"));
prices.add(new BigDecimal("9.99"));
prices.add(new BigDecimal("149.99"));
prices.add(new BigDecimal("4.99"));

for (BigDecimal price : prices) {
    System.out.println(price);
}
// Output: 4.99, 9.99, 29.99, 149.99 (sorted)

// TreeSet-specific operations (NavigableSet interface)
BigDecimal lowest = prices.first();          // 4.99
BigDecimal highest = prices.last();          // 149.99
BigDecimal lower = prices.lower(new BigDecimal("30"));   // 29.99
BigDecimal higher = prices.higher(new BigDecimal("10")); // 29.99
BigDecimal floor = prices.floor(new BigDecimal("30"));   // 29.99
BigDecimal ceiling = prices.ceiling(new BigDecimal("30")); // 149.99

SortedSet<BigDecimal> range = prices.subSet(
    new BigDecimal("10"), new BigDecimal("100")
);  // [29.99]

SortedSet<BigDecimal> head = prices.headSet(new BigDecimal("30"));  // [4.99, 9.99]
SortedSet<BigDecimal> tail = prices.tailSet(new BigDecimal("30"));  // [149.99]
```

**TreeSet requirements:**

Elements must be `Comparable` (implement `compareTo()`) or you must provide a `Comparator` at construction time. If neither is available, a `ClassCastException` is thrown at runtime.

```java
// Custom comparator
Set<Transaction> byAmount = new TreeSet<>(
    Comparator.comparing(Transaction::getAmount)
);

// Reverse order
Set<String> reverse = new TreeSet<>(Comparator.reverseOrder());
```

**Set comparison:**

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|--------------|---------|
| Backing structure | Hash table | Hash table + linked list | Red-black tree |
| add/remove/contains | O(1) average | O(1) average | O(log n) |
| Ordering | None | Insertion order | Sorted |
| Null elements | One null allowed | One null allowed | No nulls |
| Memory overhead | Low | Medium | High |
| Use when | Fast lookups, order irrelevant | Fast lookups, order matters | Sorted order needed |

**Set operations:**

```java
Set<String> setA = new HashSet<>(Set.of("USD", "EUR", "GBP"));
Set<String> setB = new HashSet<>(Set.of("EUR", "JPY", "CHF"));

// Union (A ∪ B)
Set<String> union = new HashSet<>(setA);
union.addAll(setB);
// [USD, EUR, GBP, JPY, CHF]

// Intersection (A ∩ B)
Set<String> intersection = new HashSet<>(setA);
intersection.retainAll(setB);
// [EUR]

// Difference (A - B)
Set<String> difference = new HashSet<>(setA);
difference.removeAll(setB);
// [USD, GBP]

// Symmetric difference (A △ B)
Set<String> symmetric = new HashSet<>(setA);
symmetric.addAll(setB);
Set<String> temp = new HashSet<>(setA);
temp.retainAll(setB);
symmetric.removeAll(temp);
// [USD, GBP, JPY, CHF]
```

### Queue and Deque

A `Queue` is a collection designed for holding elements prior to processing. Typically FIFO (first-in, first-out).

**PriorityQueue:**

A heap-based priority queue. Elements are ordered by natural ordering or a custom comparator. The head of the queue is the least element.

```java
// Min-heap (default)
Queue<Integer> minHeap = new PriorityQueue<>();
minHeap.add(5);
minHeap.add(1);
minHeap.add(3);
System.out.println(minHeap.poll());  // 1 (smallest)
System.out.println(minHeap.poll());  // 3
System.out.println(minHeap.poll());  // 5

// Max-heap
Queue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.add(5);
maxHeap.add(1);
maxHeap.add(3);
System.out.println(maxHeap.poll());  // 5 (largest)

// Custom priority: process high-value transactions first
Queue<Transaction> txQueue = new PriorityQueue<>(
    Comparator.comparing(Transaction::getAmount).reversed()
);
txQueue.add(new Transaction("TX-1", new BigDecimal("100")));
txQueue.add(new Transaction("TX-2", new BigDecimal("5000")));
txQueue.add(new Transaction("TX-3", new BigDecimal("250")));

Transaction highest = txQueue.poll();  // TX-2 ($5000)
```

**Performance:** `offer()` and `poll()` are O(log n). `peek()` is O(1). `contains()` and `remove(Object)` are O(n).

**ArrayDeque:**

A resizable array-based deque (double-ended queue). Faster than `LinkedList` for both stack and queue operations. This is the recommended replacement for both `Stack` and `LinkedList` as a queue.

```java
Deque<String> deque = new ArrayDeque<>();

// As a Queue (FIFO)
deque.offerLast("first");    // Add to tail
deque.offerLast("second");
deque.offerLast("third");
System.out.println(deque.pollFirst());  // "first"

// As a Stack (LIFO)
deque.push("bottom");        // Add to head
deque.push("middle");
deque.push("top");
System.out.println(deque.pop());  // "top"
System.out.println(deque.peek()); // "middle"

// Deque-specific operations
deque.offerFirst("head");    // Add to head — O(1)
deque.offerLast("tail");     // Add to tail — O(1)
deque.pollFirst();            // Remove from head — O(1)
deque.pollLast();             // Remove from tail — O(1)
deque.peekFirst();            // View head without removing — O(1)
deque.peekLast();             // View tail without removing — O(1)
```

**Queue methods comparison:**

| Operation | Throws Exception | Returns Special Value |
|-----------|-----------------|----------------------|
| Insert | `add(e)` | `offer(e)` (returns false if full) |
| Remove | `remove()` | `poll()` (returns null if empty) |
| Examine | `element()` | `peek()` (returns null if empty) |

In production code, prefer the `offer/poll/peek` variants. They handle empty/full conditions gracefully instead of throwing exceptions.

### Map

A `Map` is a collection of key-value pairs. Each key maps to exactly one value. Keys are unique; values may be duplicated. `Map` does NOT extend `Collection`.

**HashMap:**

The default choice. Backed by an array of buckets, each containing a linked list (or tree for large buckets).

```java
Map<String, BigDecimal> exchangeRates = new HashMap<>();

// Putting entries
exchangeRates.put("USD", BigDecimal.ONE);
exchangeRates.put("EUR", new BigDecimal("0.92"));
exchangeRates.put("GBP", new BigDecimal("0.79"));
exchangeRates.put("JPY", new BigDecimal("149.50"));

// Getting values
BigDecimal eurRate = exchangeRates.get("EUR");         // 0.92
BigDecimal chfRate = exchangeRates.get("CHF");         // null (key not found)
BigDecimal chfOrDefault = exchangeRates.getOrDefault("CHF", BigDecimal.ONE);  // 1.0

// Checking
boolean hasUsd = exchangeRates.containsKey("USD");     // true
boolean hasRate = exchangeRates.containsValue(BigDecimal.ONE);  // true

// Removing
exchangeRates.remove("GBP");
exchangeRates.remove("JPY", new BigDecimal("149.50"));  // Remove only if value matches

// Size
int size = exchangeRates.size();
boolean empty = exchangeRates.isEmpty();

// Iteration
// Over entries (most efficient)
for (Map.Entry<String, BigDecimal> entry : exchangeRates.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

// Over keys
for (String currency : exchangeRates.keySet()) {
    System.out.println(currency);
}

// Over values
for (BigDecimal rate : exchangeRates.values()) {
    System.out.println(rate);
}

// forEach (Java 8+)
exchangeRates.forEach((currency, rate) ->
    System.out.println(currency + " = " + rate)
);
```

**Advanced Map operations (Java 8+):**

```java
Map<String, Integer> transactionCounts = new HashMap<>();

// putIfAbsent — only puts if key is not already present
transactionCounts.putIfAbsent("Alice", 0);
transactionCounts.putIfAbsent("Alice", 5);  // Ignored — Alice already has 0

// compute — compute a new value based on the current value
transactionCounts.compute("Alice", (key, currentCount) ->
    currentCount == null ? 1 : currentCount + 1
);

// computeIfAbsent — compute only if key is absent
transactionCounts.computeIfAbsent("Bob", key -> 0);

// computeIfPresent — compute only if key is present
transactionCounts.computeIfPresent("Alice", (key, count) -> count + 1);

// merge — merge a new value with the existing value
transactionCounts.merge("Alice", 1, Integer::sum);
// If Alice exists: newValue = oldValue + 1
// If Alice does not exist: newValue = 1

// replaceAll — transform all values
transactionCounts.replaceAll((key, count) -> count * 2);
```

**HashMap internals:**

- Default initial capacity: 16 buckets.
- Default load factor: 0.75. When the number of entries exceeds `capacity * loadFactor`, the map resizes (doubles capacity and rehashes all entries).
- In Java 8+, when a bucket's linked list exceeds 8 nodes and the total capacity is at least 64, the list is converted to a red-black tree. This improves worst-case lookup from O(n) to O(log n) for buckets with many hash collisions.
- Keys must implement `hashCode()` and `equals()` correctly.
- One `null` key is allowed. Multiple `null` values are allowed.

```java
// Pre-allocate for known size to avoid resizing
// Formula: expectedEntries / loadFactor + 1
Map<String, Account> accounts = new HashMap<>(256);
```

**LinkedHashMap:**

A `HashMap` that maintains insertion order (or access order). Useful for LRU caches.

```java
// Insertion order (default)
Map<String, Integer> insertionOrdered = new LinkedHashMap<>();
insertionOrdered.put("C", 3);
insertionOrdered.put("A", 1);
insertionOrdered.put("B", 2);
System.out.println(insertionOrdered.keySet());  // [C, A, B]

// Access order (for LRU cache)
Map<String, Integer> accessOrdered = new LinkedHashMap<>(16, 0.75f, true);
accessOrdered.put("A", 1);
accessOrdered.put("B", 2);
accessOrdered.put("C", 3);
accessOrdered.get("A");  // Access A, moves it to end
System.out.println(accessOrdered.keySet());  // [B, C, A]

// LRU Cache using LinkedHashMap
public class LruCache<K, V> extends LinkedHashMap<K, V> {
    private final int maxSize;

    public LruCache(int maxSize) {
        super(maxSize, 0.75f, true);  // access order
        this.maxSize = maxSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxSize;  // Remove oldest when exceeding max size
    }
}

LruCache<String, Account> cache = new LruCache<>(100);
```

**TreeMap:**

A sorted map backed by a red-black tree. Keys are maintained in sorted order. All operations are O(log n).

```java
Map<LocalDate, BigDecimal> dailyBalances = new TreeMap<>();
dailyBalances.put(LocalDate.of(2024, 3, 15), new BigDecimal("5000"));
dailyBalances.put(LocalDate.of(2024, 1, 1), new BigDecimal("3000"));
dailyBalances.put(LocalDate.of(2024, 2, 10), new BigDecimal("4500"));

// Iteration is in key order
for (Map.Entry<LocalDate, BigDecimal> entry : dailyBalances.entrySet()) {
    System.out.println(entry.getKey() + ": $" + entry.getValue());
}
// 2024-01-01: $3000
// 2024-02-10: $4500
// 2024-03-15: $5000

// NavigableMap operations
Map.Entry<LocalDate, BigDecimal> first = dailyBalances.firstEntry();
Map.Entry<LocalDate, BigDecimal> last = dailyBalances.lastEntry();
Map.Entry<LocalDate, BigDecimal> floor = dailyBalances.floorEntry(LocalDate.of(2024, 2, 15));
Map.Entry<LocalDate, BigDecimal> ceiling = dailyBalances.ceilingEntry(LocalDate.of(2024, 2, 15));

SortedMap<LocalDate, BigDecimal> q1 = dailyBalances.subMap(
    LocalDate.of(2024, 1, 1),
    LocalDate.of(2024, 4, 1)
);

// Custom comparator (reverse chronological)
Map<LocalDate, BigDecimal> reverse = new TreeMap<>(Comparator.reverseOrder());
```

**Map comparison:**

| Feature | HashMap | LinkedHashMap | TreeMap |
|---------|---------|--------------|---------|
| Backing structure | Hash table | Hash table + linked list | Red-black tree |
| get/put/remove | O(1) average | O(1) average | O(log n) |
| Ordering | None | Insertion or access | Sorted by key |
| Null keys | One allowed | One allowed | Not allowed |
| Null values | Allowed | Allowed | Allowed |
| Use when | Fast lookups | Fast lookups + order | Sorted keys |

### Immutable Collections (Java 9+)

Java 9 introduced factory methods for creating truly immutable collections. These are more memory-efficient and safer than `Collections.unmodifiableList()`.

```java
// Lists
List<String> names = List.of("Alice", "Bob", "Charlie");
List<String> empty = List.of();
List<String> single = List.of("Alice");
List<String> many = List.of("A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K");

// Sets
Set<String> currencies = Set.of("USD", "EUR", "GBP");
Set<String> emptySet = Set.of();

// Maps
Map<String, Integer> ages = Map.of("Alice", 30, "Bob", 25);
Map<String, Integer> moreAges = Map.ofEntries(
    Map.entry("Alice", 30),
    Map.entry("Bob", 25),
    Map.entry("Charlie", 35)
);
Map<String, Integer> emptyMap = Map.of();

// Copy of existing collection (defensive copy)
List<String> mutable = new ArrayList<>(List.of("A", "B", "C"));
List<String> immutable = List.copyOf(mutable);
mutable.add("D");
System.out.println(immutable);  // [A, B, C] — unaffected

// Properties of immutable collections:
// - Cannot add, remove, or replace elements (throws UnsupportedOperationException)
// - Cannot contain null elements (throws NullPointerException)
// - Thread-safe (immutable = inherently thread-safe)
// - Identity-based deduplication for sets (uses equals())
```

### Collections Utility Class

`java.util.Collections` provides static utility methods for collections.

```java
List<Integer> numbers = new ArrayList<>(List.of(5, 2, 8, 1, 9, 3));

// Sorting
Collections.sort(numbers);                    // Ascending
Collections.sort(numbers, Comparator.reverseOrder());  // Descending
Collections.shuffle(numbers);                 // Random order

// Searching
int index = Collections.binarySearch(numbers, 5);  // List must be sorted
int min = Collections.min(numbers);
int max = Collections.max(numbers);
int freq = Collections.frequency(numbers, 3);

// Unmodifiable views (legacy — prefer List.of() in Java 9+)
List<String> readOnly = Collections.unmodifiableList(mutableList);
Set<String> readOnlySet = Collections.unmodifiableSet(mutableSet);
Map<String, Integer> readOnlyMap = Collections.unmodifiableMap(mutableMap);

// Synchronized wrappers (legacy — prefer ConcurrentHashMap etc.)
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());

// Singletons
List<String> single = Collections.singletonList("only");
Set<String> singleSet = Collections.singleton("only");
Map<String, Integer> singleMap = Collections.singletonMap("key", 1);

// Empty collections
List<String> empty = Collections.emptyList();
Set<String> emptySet = Collections.emptySet();
Map<String, Integer> emptyMap = Collections.emptyMap();

// Fill and replace
Collections.fill(numbers, 0);              // All elements become 0
Collections.replaceAll(numbers, 5, 99);    // Replace all 5s with 99

// Reverse and rotate
Collections.reverse(numbers);
Collections.rotate(numbers, 2);  // Shift elements right by 2 positions
```

### Comparable and Comparator

**Comparable** defines the natural ordering of a class. A class implements `Comparable<T>` and overrides `compareTo()`.

```java
public class Transaction implements Comparable<Transaction> {
    private final LocalDateTime timestamp;
    private final BigDecimal amount;

    @Override
    public int compareTo(Transaction other) {
        // Natural order: by timestamp (oldest first)
        return this.timestamp.compareTo(other.timestamp);
    }
}

// Now you can sort without specifying a comparator
List<Transaction> sorted = new ArrayList<>(transactions);
Collections.sort(sorted);  // Uses compareTo()
TreeSet<Transaction> treeSet = new TreeSet<>(transactions);  // Uses compareTo()
```

**Comparator** defines custom orderings external to the class. Use when you need multiple sort orders or cannot modify the class.

```java
// Lambda comparator
Comparator<Transaction> byAmount = (t1, t2) ->
    t1.getAmount().compareTo(t2.getAmount());

// Method reference with Comparator.comparing
Comparator<Transaction> byAmountDesc =
    Comparator.comparing(Transaction::getAmount).reversed();

// Chained comparators
Comparator<Transaction> byDateThenAmount =
    Comparator.comparing(Transaction::getTimestamp)
        .thenComparing(Transaction::getAmount);

// Null handling
Comparator<Transaction> byNullableField =
    Comparator.comparing(
        Transaction::getDescription,
        Comparator.nullsLast(Comparator.naturalOrder())
    );

// Multi-field sorting
transactions.sort(
    Comparator.comparing(Transaction::getStatus)
        .thenComparing(Transaction::getAmount, Comparator.reverseOrder())
        .thenComparing(Transaction::getTimestamp)
);
```

**Key Comparator factory methods:**

| Method | Purpose |
|--------|---------|
| `Comparator.comparing(keyExtractor)` | Sort by a single field |
| `Comparator.comparingInt/Long/Double` | Sort by primitive field (avoids boxing) |
| `.thenComparing(keyExtractor)` | Secondary sort field |
| `.reversed()` | Reverse the order |
| `Comparator.naturalOrder()` | Natural ordering |
| `Comparator.reverseOrder()` | Reverse natural ordering |
| `Comparator.nullsFirst(cmp)` | Nulls sort first |
| `Comparator.nullsLast(cmp)` | Nulls sort last |

### Iterator and Iterable

The `Iterator` interface provides a uniform way to traverse any collection.

```java
List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Charlie"));

// Explicit iterator
Iterator<String> iterator = names.iterator();
while (iterator.hasNext()) {
    String name = iterator.next();
    if (name.equals("Bob")) {
        iterator.remove();  // Safe removal during iteration
    }
}
System.out.println(names);  // [Alice, Charlie]

// Enhanced for loop (uses Iterator internally)
for (String name : names) {
    System.out.println(name);
}

// forEach (uses Iterable.forEach, Java 8+)
names.forEach(System.out::println);
```

**ConcurrentModificationException:**

If you modify a collection structurally (add/remove) while iterating over it with a for-each loop or iterator (other than through `iterator.remove()`), Java throws `ConcurrentModificationException`.

```java
// WRONG — throws ConcurrentModificationException
for (String name : names) {
    if (name.equals("Bob")) {
        names.remove(name);  // Structural modification during iteration!
    }
}

// CORRECT — use iterator.remove()
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    if (it.next().equals("Bob")) {
        it.remove();
    }
}

// CORRECT — use removeIf (Java 8+)
names.removeIf(name -> name.equals("Bob"));

// CORRECT — use streams
List<String> filtered = names.stream()
    .filter(name -> !name.equals("Bob"))
    .toList();
```

**Implementing Iterable for custom classes:**

```java
public class TransactionLog implements Iterable<Transaction> {
    private final List<Transaction> transactions = new ArrayList<>();

    public void add(Transaction tx) {
        transactions.add(tx);
    }

    @Override
    public Iterator<Transaction> iterator() {
        return transactions.iterator();
    }
}

// Now you can use for-each:
TransactionLog log = new TransactionLog();
log.add(tx1);
log.add(tx2);
for (Transaction tx : log) {
    System.out.println(tx);
}
```

---

## Code Examples

**A complete example demonstrating collection selection in a financial context:**

```java
package com.example.collections;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.*;
import java.util.stream.Collectors;

public class CollectionsDemo {

    public static void main(String[] args) {
        // 1. ArrayList — storing and processing transactions
        List<Transaction> transactions = new ArrayList<>();
        transactions.add(new Transaction("TX-001", new BigDecimal("150.00"),
            LocalDate.of(2024, 1, 15), "FOOD"));
        transactions.add(new Transaction("TX-002", new BigDecimal("1200.00"),
            LocalDate.of(2024, 1, 1), "RENT"));
        transactions.add(new Transaction("TX-003", new BigDecimal("45.50"),
            LocalDate.of(2024, 1, 20), "FOOD"));
        transactions.add(new Transaction("TX-004", new BigDecimal("89.99"),
            LocalDate.of(2024, 2, 5), "UTILITIES"));
        transactions.add(new Transaction("TX-005", new BigDecimal("3500.00"),
            LocalDate.of(2024, 1, 1), "SALARY"));

        // Sort by date, then by amount descending
        transactions.sort(
            Comparator.comparing(Transaction::date)
                .thenComparing(Transaction::amount, Comparator.reverseOrder())
        );

        System.out.println("Sorted transactions:");
        transactions.forEach(tx ->
            System.out.printf("  %s | %s | $%10.2f | %s%n",
                tx.id(), tx.date(), tx.amount(), tx.category())
        );

        // 2. HashMap — aggregating spending by category
        Map<String, BigDecimal> spendingByCategory = new HashMap<>();
        for (Transaction tx : transactions) {
            if (tx.amount().compareTo(BigDecimal.ZERO) > 0 &&
                !tx.category().equals("SALARY")) {
                spendingByCategory.merge(tx.category(), tx.amount(), BigDecimal::add);
            }
        }

        System.out.println("\nSpending by category:");
        spendingByCategory.forEach((cat, total) ->
            System.out.printf("  %-12s $%10.2f%n", cat, total)
        );

        // 3. TreeSet — unique sorted dates
        Set<LocalDate> uniqueDates = new TreeSet<>();
        transactions.forEach(tx -> uniqueDates.add(tx.date()));
        System.out.println("\nUnique transaction dates (sorted):");
        uniqueDates.forEach(date -> System.out.println("  " + date));

        // 4. LinkedHashMap — preserving insertion order for report
        Map<String, List<Transaction>> byCategory = new LinkedHashMap<>();
        for (Transaction tx : transactions) {
            byCategory.computeIfAbsent(tx.category(), k -> new ArrayList<>()).add(tx);
        }

        System.out.println("\nTransactions grouped by category:");
        byCategory.forEach((category, txList) -> {
            System.out.println("  " + category + ":");
            txList.forEach(tx ->
                System.out.printf("    %s $%.2f%n", tx.id(), tx.amount())
            );
        });

        // 5. PriorityQueue — processing highest-value transactions first
        Queue<Transaction> priorityQueue = new PriorityQueue<>(
            Comparator.comparing(Transaction::amount).reversed()
        );
        priorityQueue.addAll(transactions);

        System.out.println("\nProcessing by priority (highest amount first):");
        while (!priorityQueue.isEmpty()) {
            Transaction tx = priorityQueue.poll();
            System.out.printf("  Processing %s: $%.2f%n", tx.id(), tx.amount());
        }

        // 6. ArrayDeque — undo stack for transaction operations
        Deque<String> undoStack = new ArrayDeque<>();
        undoStack.push("Created TX-001");
        undoStack.push("Updated TX-002 amount");
        undoStack.push("Deleted TX-003");

        System.out.println("\nUndo operations:");
        while (!undoStack.isEmpty()) {
            System.out.println("  Undoing: " + undoStack.pop());
        }

        // 7. Immutable collections — configuration
        Set<String> supportedCurrencies = Set.of("USD", "EUR", "GBP", "JPY", "CHF");
        Map<String, BigDecimal> fixedRates = Map.of(
            "EUR", new BigDecimal("0.92"),
            "GBP", new BigDecimal("0.79")
        );

        System.out.println("\nSupported currencies: " + supportedCurrencies);
        System.out.println("Fixed rates: " + fixedRates);
    }

    public record Transaction(
        String id,
        BigDecimal amount,
        LocalDate date,
        String category
    ) {}
}
```

**Thread-safe collection usage:**

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;

public class ConcurrentCollectionsDemo {

    // ConcurrentHashMap — thread-safe, high-performance map
    private final Map<String, BigDecimal> accountBalances = new ConcurrentHashMap<>();

    public void updateBalance(String accountId, BigDecimal delta) {
        // Atomic compute — thread-safe without external synchronization
        accountBalances.compute(accountId, (key, currentBalance) -> {
            BigDecimal balance = currentBalance != null ? currentBalance : BigDecimal.ZERO;
            return balance.add(delta);
        });
    }

    // CopyOnWriteArrayList — thread-safe, optimized for reads
    // Writes create a new copy of the underlying array
    // Use when reads vastly outnumber writes (e.g., listener lists)
    private final List<String> activeListeners = new CopyOnWriteArrayList<>();

    public void addListener(String listener) {
        activeListeners.add(listener);
    }

    public void notifyAll(String message) {
        // Safe to iterate while other threads may be adding/removing
        for (String listener : activeListeners) {
            System.out.println("Notifying " + listener + ": " + message);
        }
    }
}
```

---

## Important Notes

- `ArrayList` is the default collection for ordered sequences. Use it unless you have a specific reason not to. It outperforms `LinkedList` in nearly all real-world scenarios due to CPU cache locality.
- `HashMap` is the default collection for key-value mappings. Use it unless you need ordering (`LinkedHashMap`, `TreeMap`) or thread safety (`ConcurrentHashMap`).
- Always override `equals()` and `hashCode()` on classes used as elements in `HashSet` or keys in `HashMap`. Failing to do so causes duplicate entries, failed lookups, and memory leaks.
- Never modify an object's hash-relevant fields after adding it to a hash-based collection. The object becomes unreachable because its hash code no longer matches its bucket location.
- Use `List.of()`, `Set.of()`, and `Map.of()` (Java 9+) for immutable collections. They are more concise, more memory-efficient, and safer than `Collections.unmodifiableList(new ArrayList<>(...))`.
- `Collections.unmodifiableList()` creates a read-only view, not a true copy. If the underlying list is modified, the "unmodifiable" view reflects the changes. `List.of()` and `List.copyOf()` create truly independent immutable collections.
- `TreeSet` and `TreeMap` require elements/keys to be `Comparable` or a `Comparator` must be provided. If neither is available, a `ClassCastException` is thrown at runtime, not at compile time.
- `PriorityQueue` does not guarantee iteration order. The only guarantee is that `poll()` returns the least element according to the comparator. If you iterate over a `PriorityQueue` with a for-each loop, the elements will NOT be in priority order.
- `ConcurrentHashMap` is the correct choice for thread-safe maps in concurrent applications. Do not use `Collections.synchronizedMap(new HashMap<>())` — it synchronizes on every operation and is significantly slower under contention. `ConcurrentHashMap` uses fine-grained locking (segment-level in Java 7, CAS + synchronized on individual nodes in Java 8+).
- `CopyOnWriteArrayList` is appropriate only when reads vastly outnumber writes. Every write creates a full copy of the underlying array. For a list with 10,000 elements, each `add()` copies all 10,000 references. Use it for listener lists, configuration caches, and similar read-heavy scenarios.
- The `removeIf()` method (Java 8+) is the cleanest way to remove elements from a collection based on a condition. It handles the iterator mechanics internally and avoids `ConcurrentModificationException`.
- When pre-allocating a `HashMap` for a known number of entries, use the formula `expectedEntries / 0.75 + 1` to avoid resizing. For 1000 entries: `new HashMap<>(1334)`.
- `EnumMap` and `EnumSet` are specialized implementations for enum keys/elements. They are faster and more memory-efficient than `HashMap` and `HashSet` for enum types. Always use them when the key type is an enum.
- The `Collections` utility class and the `Collection` interface are different things. `Collection` (singular) is the root interface. `Collections` (plural) is a utility class with static methods. Do not confuse them.

---

## Practice

1. Create a `List<Transaction>` with at least 20 transactions spanning multiple categories and dates. Sort it by date ascending, then by amount descending within each date. Print the sorted list.

2. Write a method that takes a `List<Transaction>` and returns a `Map<String, BigDecimal>` containing the total spending per category. Use `HashMap` with `merge()`. Then rewrite it using streams and `Collectors.groupingBy()`.

3. Implement an LRU cache using `LinkedHashMap` that stores the 10 most recently accessed account balances. Demonstrate that the least recently used entry is evicted when the cache exceeds its capacity.

4. Create a `PriorityQueue<Transaction>` that processes transactions in order of: (1) status (FAILED first, then PENDING, then COMPLETED), (2) amount descending within each status. Add 10 transactions and poll them all, verifying the order.

5. Demonstrate the `ConcurrentModificationException` by modifying a list during a for-each loop. Then fix it using three different approaches: `Iterator.remove()`, `removeIf()`, and streams.

6. Create a class `Money` with `BigDecimal amount` and `String currency`. Override `equals()` and `hashCode()`. Add instances to a `HashSet` and verify that duplicates are correctly detected. Then break the `hashCode()` implementation and observe the incorrect behavior.

7. In your Obsidian vault, create a decision table: "Which collection should I use?" with rows for common scenarios (ordered list, unique elements, key-value lookup, sorted data, priority processing, FIFO queue, LIFO stack, thread-safe map) and columns recommending the specific implementation.

---

## References

- Java Collections Framework Overview: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/package-summary.html#CollectionsFramework
- List API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html
- Map API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Map.html
- Set API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Set.html
- Collections Utility: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html
- "Effective Java" by Joshua Bloch — Items 25-31 (Collections), Item 55 (Return optionals judiciously)
- Java Tutorial — Collections: https://docs.oracle.com/javase/tutorial/collections/
