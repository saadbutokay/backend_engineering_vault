---
title: "Java - List - ArrayList LinkedList"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - collections
  - list
  - arraylist
  - linkedlist
status: "not-started"
---

# Java - List - ArrayList LinkedList

> [!abstract] Overview
> The `List` interface represents an ordered collection (sequence) that allows duplicate elements and provides positional access via integer indices. It is the most commonly used collection type in backend development. The two primary implementations are `ArrayList` (backed by a resizable array, optimized for random access) and `LinkedList` (backed by a doubly-linked list, optimized for insertion and deletion at the ends). In Spring Boot, `List` is the default return type for database queries, the standard container for API request and response payloads, and the workhorse of every service layer that processes collections of records.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Arrays - 1D and 2D]]
> - [[Java - Collections Framework Overview]]
> - [[Java - Abstraction - Abstract Classes and Interfaces]]
> - [[Java - Generics - Classes Methods Wildcards]] (basic understanding of `<T>` syntax)

---

## Theory

### The `List` Interface

The `List` interface extends `Collection` and adds operations that depend on the **positional index** of elements. The key contract of a `List` is:

1. **Ordered**: Elements maintain their insertion order. The first element added is at index 0, the second at index 1, and so on.
2. **Allows duplicates**: Unlike `Set`, a `List` can contain multiple equal elements. `[1, 2, 2, 3]` is a valid list.
3. **Positional access**: Elements can be accessed, inserted, and removed by their integer index using `get(i)`, `set(i, element)`, `add(i, element)`, and `remove(i)`.
4. **Searchable**: `indexOf()` and `lastIndexOf()` find the position of elements.

**Core `List` methods:**

| Method | Description | Return |
|--------|-------------|--------|
| `add(E e)` | Appends to the end | `boolean` |
| `add(int index, E e)` | Inserts at the specified position | `void` |
| `get(int index)` | Returns the element at the position | `E` |
| `set(int index, E e)` | Replaces the element at the position | `E` (old value) |
| `remove(int index)` | Removes the element at the position | `E` (removed value) |
| `remove(Object o)` | Removes the first occurrence of the object | `boolean` |
| `indexOf(Object o)` | Returns the index of the first occurrence | `int` (-1 if not found) |
| `lastIndexOf(Object o)` | Returns the index of the last occurrence | `int` (-1 if not found) |
| `size()` | Returns the number of elements | `int` |
| `isEmpty()` | Returns true if the list has no elements | `boolean` |
| `contains(Object o)` | Returns true if the list contains the element | `boolean` |
| `subList(int from, int to)` | Returns a view of the portion of the list | `List<E>` |
| `sort(Comparator)` | Sorts the list in place | `void` |
| `toArray()` | Converts the list to an array | `Object[]` |

### `ArrayList` Internals

`ArrayList` is backed by a **resizable array** (`Object[]`). It is the default choice for most use cases and the most commonly used collection in all of Java.

**How it works:**

1. **Initial capacity**: When you create `new ArrayList<>()`, the internal array starts with a default capacity of 10 (in OpenJDK). You can specify a different initial capacity with `new ArrayList<>(100)`.

2. **Adding elements**: When you call `add(element)`, the element is placed at the next available index in the internal array. This is an O(1) operation as long as the array has free space.

3. **Resizing (growth)**: When the internal array is full and you add another element, `ArrayList` creates a new, larger array and copies all existing elements into it. The new capacity is approximately 1.5x the old capacity (`oldCapacity + oldCapacity >> 1`). This resizing is an O(n) operation, but it happens infrequently enough that the **amortized** cost of `add()` is still O(1).

4. **Random access**: `get(index)` directly accesses the internal array at the given index. This is O(1) because arrays support constant-time index access.

5. **Insertion in the middle**: `add(index, element)` requires shifting all elements from the index to the end one position to the right. This is O(n) because every element after the insertion point must be moved.

6. **Deletion**: `remove(index)` requires shifting all elements after the removed index one position to the left. This is O(n). `remove(Object)` first searches for the element (O(n)) and then shifts (O(n)), so it is O(n) overall.

**Memory layout:**

```text
ArrayList<String> list = new ArrayList<>(4);
list.add("Dhaka");
list.add("Sylhet");
list.add("Rajshahi");

Internal state:
  Object[] elementData = ["Dhaka", "Sylhet", "Rajshahi", null]
  int size = 3
  int capacity = 4

After list.add("Khulna"):
  Object[] elementData = ["Dhaka", "Sylhet", "Rajshahi", "Khulna"]
  int size = 4
  int capacity = 4

After list.add("Barishal"):  // Triggers resize!
  Object[] elementData = ["Dhaka", "Sylhet", "Rajshahi", "Khulna", "Barishal", null]
  int size = 5
  int capacity = 6  (4 * 1.5 = 6)
```

### `LinkedList` Internals

`LinkedList` is backed by a **doubly-linked list**. Each element is stored in a `Node` object that contains the element value and references to the previous and next nodes.

**How it works:**

1. **Node structure**: Each node has three fields: `E item` (the element), `Node<E> next` (reference to the next node), and `Node<E> prev` (reference to the previous node).

2. **Adding at the ends**: `add(element)` appends a new node at the tail. `addFirst(element)` prepends at the head. Both are O(1) because the `LinkedList` maintains direct references to the `first` and `last` nodes.

3. **Adding in the middle**: `add(index, element)` first traverses the list to find the node at the given index (O(n)), then inserts a new node between the found node and its predecessor (O(1)). The total cost is O(n) due to the traversal.

4. **Random access**: `get(index)` traverses the list from the head (or tail, whichever is closer) to the given index. This is O(n) because there is no direct index access. For a list of 1 million elements, accessing the 500,000th element requires 500,000 node traversals.

5. **Deletion**: `remove(index)` traverses to the node (O(n)) and then unlinks it by updating the previous and next pointers (O(1)). If you already have a reference to the node (via an `Iterator` or `ListIterator`), removal is O(1).

**Memory layout:**

```text
LinkedList<String> list = new LinkedList<>();
list.add("Dhaka");
list.add("Sylhet");
list.add("Rajshahi");

Internal state:
  Node first ----> [prev:null | "Dhaka" | next:--]
                                        |
  Node last  <---- [prev:-- | "Rajshahi" | next:null]
                          |
                   [prev:-- | "Sylhet" | next:--]

Each node is a separate object on the heap with three fields.
This means LinkedList uses significantly more memory per element than ArrayList.
```

### `ArrayList` vs `LinkedList` Performance

| Operation | ArrayList | LinkedList | Winner |
|-----------|-----------|------------|--------|
| `add(element)` at end | O(1) amortized | O(1) | Tie (ArrayList faster in practice due to cache locality) |
| `add(index, element)` in middle | O(n) shift | O(n) traverse + O(1) insert | Tie (both O(n), but ArrayList's shift is a fast `System.arraycopy`) |
| `get(index)` random access | O(1) | O(n) | **ArrayList** (by a huge margin) |
| `remove(index)` | O(n) shift | O(n) traverse + O(1) unlink | Tie (both O(n)) |
| `remove(Object)` | O(n) search + O(n) shift | O(n) search + O(1) unlink | LinkedList (slightly, if the element is near the head) |
| `contains(Object)` | O(n) | O(n) | Tie |
| `iterator.remove()` | O(n) shift | O(1) unlink | **LinkedList** |
| Memory per element | ~4 bytes (reference in array) | ~24 bytes (Node object + 2 references) | **ArrayList** (6x less memory) |
| Cache locality | Excellent (contiguous memory) | Poor (scattered nodes) | **ArrayList** |

**The practical verdict**: `ArrayList` is faster than `LinkedList` for almost all real-world use cases, even for operations where `LinkedList` has a theoretical advantage. The reasons are:

1. **CPU cache locality**: `ArrayList`'s elements are stored in a contiguous array, which fits neatly into CPU cache lines. `LinkedList`'s nodes are scattered across the heap, causing frequent cache misses that are far more expensive than the theoretical O(n) vs O(1) difference.
2. **Memory overhead**: Each `LinkedList` node is a separate object with object header overhead (12-16 bytes) plus two reference fields (8 bytes each) plus the element reference (8 bytes). This is roughly 6x more memory per element than `ArrayList`.
3. **Garbage collection pressure**: `LinkedList` creates a new `Node` object for every element, generating more garbage for the GC to collect.

> [!tip] Key Insight
> In modern Java backend development, **always default to `ArrayList`**. The only scenario where `LinkedList` might be preferable is when you need frequent insertion and deletion at both ends of the list and you never need random access by index. Even then, `ArrayDeque` is usually a better choice than `LinkedList` for queue/deque operations. The Java community's consensus, backed by extensive benchmarking, is that `LinkedList` should be considered a legacy data structure for most application code.

### Immutable Lists (Java 9+)

Java 9 introduced factory methods for creating immutable lists. These lists cannot be modified after creation: `add()`, `remove()`, and `set()` all throw `UnsupportedOperationException`.

```java
// Immutable list with specific elements
List<String> cities = List.of("Dhaka", "Chittagong", "Sylhet");

// Immutable list from an existing collection
List<String> copy = List.copyOf(mutableList);

// Empty immutable list
List<String> empty = List.of();
```

Immutable lists are useful for:
- Returning unmodifiable results from service methods (prevents callers from accidentally modifying internal state).
- Defining constant lists (configuration values, default settings).
- Thread safety (immutable objects are inherently thread-safe).

---

## Syntax and Basic Examples

### Example 1: ArrayList creation and basic operations

```java
import java.util.*;

public class ArrayListDemo {
    public static void main(String[] args) {
        // Creation
        List<String> cities = new ArrayList<>();
        List<Integer> numbers = new ArrayList<>(100);  // Initial capacity of 100

        // Adding elements
        cities.add("Dhaka");           // Add at end: O(1)
        cities.add("Chittagong");
        cities.add("Sylhet");
        cities.add(1, "Rajshahi");     // Insert at index 1: O(n) shift
        System.out.println("Cities: " + cities);
        // [Dhaka, Rajshahi, Chittagong, Sylhet]

        // Accessing elements
        System.out.println("First: " + cities.get(0));      // Dhaka: O(1)
        System.out.println("Last: " + cities.get(cities.size() - 1));  // Sylhet
        // cities.get(10);  // IndexOutOfBoundsException!

        // Modifying elements
        cities.set(2, "Khulna");  // Replace Chittagong with Khulna: O(1)
        System.out.println("After set: " + cities);
        // [Dhaka, Rajshahi, Khulna, Sylhet]

        // Removing elements
        cities.remove(1);           // Remove by index (Rajshahi): O(n) shift
        cities.remove("Khulna");    // Remove by value: O(n) search + O(n) shift
        System.out.println("After remove: " + cities);
        // [Dhaka, Sylhet]

        // Searching
        System.out.println("Contains Dhaka: " + cities.contains("Dhaka"));  // true: O(n)
        System.out.println("Index of Sylhet: " + cities.indexOf("Sylhet")); // 1: O(n)
        System.out.println("Index of Rajshahi: " + cities.indexOf("Rajshahi"));  // -1: not found

        // Size and emptiness
        System.out.println("Size: " + cities.size());      // 2
        System.out.println("Empty: " + cities.isEmpty());  // false

        // Clearing
        cities.clear();
        System.out.println("After clear: " + cities.isEmpty());  // true
    }
}
```

### Example 2: LinkedList as a queue and deque

```java
import java.util.*;

public class LinkedListDemo {
    public static void main(String[] args) {
        // LinkedList implements both List and Deque interfaces
        LinkedList<String> deque = new LinkedList<>();

        // Use as a Queue (FIFO)
        deque.addLast("Order-001");
        deque.addLast("Order-002");
        deque.addLast("Order-003");
        System.out.println("Queue front: " + deque.removeFirst());  // Order-001
        System.out.println("Queue front: " + deque.removeFirst());  // Order-002

        // Use as a Stack (LIFO)
        deque.push("Task-A");
        deque.push("Task-B");
        deque.push("Task-C");
        System.out.println("Stack top: " + deque.pop());  // Task-C
        System.out.println("Stack top: " + deque.pop());  // Task-B

        // Use as a List (positional access, but O(n)!)
        List<String> list = new LinkedList<>(List.of("A", "B", "C", "D", "E"));
        System.out.println("Get(2): " + list.get(2));  // C (traverses 2 nodes)
        list.add(2, "X");  // Insert at index 2 (traverses 2 nodes, then O(1) insert)
        System.out.println("After insert: " + list);  // [A, B, X, C, D, E]
    }
}
```

### Example 3: Iteration patterns

```java
import java.util.*;

public class ListIteration {
    public static void main(String[] args) {
        List<String> frameworks = new ArrayList<>(
            List.of("Spring Boot", "Hibernate", "Quarkus", "Micronaut", "Dropwizard")
        );

        // Pattern 1: Enhanced for-each (read-only iteration)
        System.out.println("--- For-each ---");
        for (String fw : frameworks) {
            System.out.println(fw);
        }

        // Pattern 2: Indexed for loop (when you need the index)
        System.out.println("\n--- Indexed ---");
        for (int i = 0; i < frameworks.size(); i++) {
            System.out.println(i + ": " + frameworks.get(i));
        }

        // Pattern 3: Iterator (safe removal during iteration)
        System.out.println("\n--- Iterator with removal ---");
        Iterator<String> it = frameworks.iterator();
        while (it.hasNext()) {
            String fw = it.next();
            if (fw.equals("Dropwizard")) {
                it.remove();  // Safe removal! No ConcurrentModificationException.
            }
        }
        System.out.println("After removal: " + frameworks);

        // Pattern 4: ListIterator (bidirectional traversal)
        System.out.println("\n--- ListIterator (reverse) ---");
        ListIterator<String> lit = frameworks.listIterator(frameworks.size());
        while (lit.hasPrevious()) {
            System.out.println(lit.previous());
        }

        // Pattern 5: forEach with lambda (Java 8+)
        System.out.println("\n--- Lambda ---");
        frameworks.forEach(fw -> System.out.println("Framework: " + fw));

        // Pattern 6: removeIf (Java 8+, cleanest for conditional removal)
        frameworks.removeIf(fw -> fw.startsWith("Q"));
        System.out.println("After removeIf: " + frameworks);
    }
}
```

### Example 4: Sorting lists

```java
import java.util.*;

public class ListSorting {
    public static void main(String[] args) {
        // Sorting strings (natural order: alphabetical)
        List<String> names = new ArrayList<>(List.of("Saad", "Rahim", "Karim", "Arif", "Nila"));
        Collections.sort(names);
        System.out.println("Sorted: " + names);  // [Arif, Karim, Nila, Rahim, Saad]

        // Sorting in reverse order
        names.sort(Comparator.reverseOrder());
        System.out.println("Reverse: " + names);  // [Saad, Rahim, Nila, Karim, Arif]

        // Sorting objects by a specific field
        record Student(String name, double cgpa) {}

        List<Student> students = new ArrayList<>(List.of(
            new Student("Saad", 3.72),
            new Student("Rahim", 3.45),
            new Student("Karim", 3.90),
            new Student("Nila", 3.80)
        ));

        // Sort by CGPA ascending
        students.sort(Comparator.comparingDouble(Student::cgpa));
        System.out.println("\nBy CGPA (asc):");
        students.forEach(s -> System.out.println("  " + s.name() + ": " + s.cgpa()));

        // Sort by CGPA descending
        students.sort(Comparator.comparingDouble(Student::cgpa).reversed());
        System.out.println("\nBy CGPA (desc):");
        students.forEach(s -> System.out.println("  " + s.name() + ": " + s.cgpa()));

        // Sort by name length, then alphabetically
        students.sort(Comparator.comparingInt((Student s) -> s.name().length())
            .thenComparing(Student::name));
        System.out.println("\nBy name length, then alpha:");
        students.forEach(s -> System.out.println("  " + s.name() + " (" + s.name().length() + ")"));
    }
}
```

### Example 5: Sublists and immutable lists

```java
import java.util.*;

public class SublistDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(List.of(10, 20, 30, 40, 50, 60, 70));

        // subList returns a VIEW, not a copy.
        // Changes to the sublist are reflected in the original list and vice versa.
        List<Integer> middle = numbers.subList(2, 5);  // indices 2, 3, 4
        System.out.println("Sublist: " + middle);  // [30, 40, 50]

        middle.set(0, 99);  // Modifies the original list!
        System.out.println("Original after sublist modification: " + numbers);
        // [10, 20, 99, 40, 50, 60, 70]

        // Immutable lists (Java 9+)
        List<String> immutable = List.of("Dhaka", "Sylhet", "Rajshahi");
        System.out.println("\nImmutable: " + immutable);
        // immutable.add("Khulna");  // UnsupportedOperationException!
        // immutable.set(0, "Barishal");  // UnsupportedOperationException!
        // immutable.remove(0);  // UnsupportedOperationException!

        // List.copyOf creates an immutable copy of a mutable list
        List<String> mutable = new ArrayList<>(List.of("A", "B", "C"));
        List<String> frozen = List.copyOf(mutable);
        mutable.add("D");
        System.out.println("Mutable: " + mutable);  // [A, B, C, D]
        System.out.println("Frozen: " + frozen);    // [A, B, C] (unaffected)
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> `List` is the most frequently used collection in Spring Boot backends. Here are three realistic scenarios.

### Scenario 1: Pagination with Spring Data

Every backend API that returns lists of records implements pagination to avoid sending millions of rows in a single response. Spring Data's `Page` and `Slice` abstractions are built on top of `List`.

```java
package com.company.orderservice.service;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    // Returns a paginated list of orders for a user
    public PageResponse<OrderResponse> getUserOrders(Long userId, int page, int size) {
        // Spring Data returns a Page<Order>, which contains a List<Order>
        // plus pagination metadata (total pages, total elements, etc.)
        Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
        Page<Order> orderPage = orderRepository.findByUserId(userId, pageable);

        // Transform the List<Order> to List<OrderResponse>
        List<OrderResponse> content = orderPage.getContent().stream()
            .map(OrderResponse::fromEntity)
            .toList();  // toList() returns an unmodifiable list (Java 16+)

        return new PageResponse<>(
            content,
            orderPage.getNumber(),
            orderPage.getSize(),
            orderPage.getTotalElements(),
            orderPage.getTotalPages(),
            orderPage.hasNext()
        );
    }
}

// The pagination response DTO
public record PageResponse<T>(
    List<T> content,       // The actual list of items for this page
    int pageNumber,        // Current page (0-indexed)
    int pageSize,          // Items per page
    long totalElements,    // Total items across all pages
    int totalPages,        // Total number of pages
    boolean hasNext        // Whether there is a next page
) {}
```

**What to notice:**

- `orderPage.getContent()` returns a `List<Order>`. This is the sublist of orders for the current page, not the entire database table.
- `.toList()` (Java 16+) returns an unmodifiable list. This prevents the controller or the JSON serializer from accidentally modifying the list after the service returns it.
- The `PageResponse` record wraps the list with pagination metadata. The frontend uses this metadata to render pagination controls (page numbers, next/previous buttons).

### Scenario 2: Building response lists with filtering and transformation

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

@Service
public class ProductService {

    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public List<ProductResponse> searchProducts(String keyword, String category,
                                                  Double minPrice, Double maxPrice) {
        // Fetch all products matching the category (database-level filtering)
        List<Product> products;
        if (category != null) {
            products = productRepository.findByCategory(category);
        } else {
            products = productRepository.findAll();
        }

        // Apply additional filters in memory (application-level filtering)
        // In production, these filters would be pushed to the database query.
        // Shown here to demonstrate List operations.
        List<ProductResponse> results = new ArrayList<>();

        for (Product product : products) {
            // Keyword filter
            if (keyword != null && !product.getName().toLowerCase()
                    .contains(keyword.toLowerCase())) {
                continue;
            }

            // Price range filter
            if (minPrice != null && product.getPrice() < minPrice) {
                continue;
            }
            if (maxPrice != null && product.getPrice() > maxPrice) {
                continue;
            }

            results.add(ProductResponse.fromEntity(product));
        }

        // Sort by price ascending
        results.sort(Comparator.comparingDouble(ProductResponse::price));

        // Return an unmodifiable view to prevent callers from modifying the list
        return Collections.unmodifiableList(results);
    }

    // Batch update: demonstrates modifying a list of entities
    public int applyDiscount(String category, double discountPercent) {
        List<Product> products = productRepository.findByCategory(category);
        int updatedCount = 0;

        for (Product product : products) {
            double newPrice = product.getPrice() * (1 - discountPercent / 100);
            product.setPrice(newPrice);
            updatedCount++;
        }

        // Save all modified products in a single batch
        // saveAll() accepts a List and executes a batch INSERT/UPDATE
        productRepository.saveAll(products);

        return updatedCount;
    }
}
```

**What to notice:**

- The `results` list is built incrementally using `add()` inside a loop. This is the most common pattern for filtering and transforming collections in backend services.
- `Collections.unmodifiableList(results)` wraps the list in a read-only view. If the controller tries to call `results.add()` or `results.remove()`, it will throw `UnsupportedOperationException`. This is a defensive programming practice that prevents accidental mutation.
- `productRepository.saveAll(products)` accepts a `List<Product>` and persists all changes in a single batch. This is much more efficient than calling `save()` in a loop because it reduces the number of database round-trips.

### Scenario 3: Request body deserialization into Lists

When a client sends a JSON array in a POST request, Spring Boot deserializes it into a `List` automatically.

```java
package com.company.orderservice.controller;

import jakarta.validation.Valid;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.ArrayList;
import java.util.List;

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    // Client sends: [{"productId": 1, "qty": 2}, {"productId": 5, "qty": 1}]
    // Spring deserializes the JSON array into a List<OrderItemRequest>
    @PostMapping("/batch")
    public ResponseEntity<BatchResponse> createBatchOrder(
            @Valid @RequestBody List<@Valid OrderItemRequest> items) {

        if (items.isEmpty()) {
            throw new ValidationException("items", "Order must contain at least one item");
        }

        if (items.size() > 100) {
            throw new ValidationException("items", "Maximum 100 items per batch");
        }

        List<OrderResponse> createdOrders = new ArrayList<>();
        List<String> errors = new ArrayList<>();

        for (int i = 0; i < items.size(); i++) {
            try {
                Order order = orderService.createSingleItemOrder(items.get(i));
                createdOrders.add(OrderResponse.fromEntity(order));
            } catch (Exception e) {
                errors.add("Item " + i + ": " + e.getMessage());
            }
        }

        return ResponseEntity.status(207).body(
            new BatchResponse(createdOrders, errors)
        );
    }
}

// The batch response contains two lists: successes and failures
public record BatchResponse(
    List<OrderResponse> successful,
    List<String> errors
) {}
```

**What to notice:**

- `@RequestBody List<OrderItemRequest> items` tells Spring to deserialize the JSON array into a `List`. The framework handles the parsing automatically using Jackson.
- The `@Valid` annotation on both the `List` and the generic type (`List<@Valid OrderItemRequest>`) triggers validation for each element in the list. If any item has invalid data, Spring throws a `MethodArgumentNotValidException`.
- The response contains two lists: one for successful orders and one for error messages. This is a common pattern for batch APIs where partial success is acceptable.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using `LinkedList` when `ArrayList` is better

**Wrong:**

```java
// Using LinkedList "because we add and remove elements frequently"
List<Order> orders = new LinkedList<>();
for (Order order : allOrders) {
    if (order.isActive()) {
        orders.add(order);  // Adding at the end: ArrayList is faster due to cache locality
    }
}
Order first = orders.get(0);  // O(n) in LinkedList! O(1) in ArrayList!
```

**Right:**

```java
List<Order> orders = new ArrayList<>();
for (Order order : allOrders) {
    if (order.isActive()) {
        orders.add(order);  // O(1) amortized, cache-friendly
    }
}
Order first = orders.get(0);  // O(1)
```

**Why it is wrong:** The common belief that "LinkedList is faster for insertion and deletion" is a myth for most real-world scenarios. `ArrayList`'s `System.arraycopy()` is a highly optimized native method that moves memory blocks much faster than `LinkedList`'s node-by-node traversal. Combined with CPU cache locality, `ArrayList` outperforms `LinkedList` in almost every benchmark. Default to `ArrayList` unless you have a specific, measured reason to use `LinkedList`.

### Mistake 2: Removing elements by value vs by index

**Wrong:**

```java
List<Integer> numbers = new ArrayList<>(List.of(10, 20, 30, 40, 50));

// Intending to remove the element at index 2 (value 30)
numbers.remove(2);  // This removes by INDEX. Removes 30. Correct by accident.

List<Integer> numbers2 = new ArrayList<>(List.of(10, 20, 30, 40, 50));
// Intending to remove the value 20
numbers2.remove(20);  // COMPILATION ERROR or IndexOutOfBoundsException!
// 20 is an int, so Java calls remove(int index), not remove(Object o).
// It tries to remove the element at index 20, which does not exist.
```

**Right:**

```java
List<Integer> numbers = new ArrayList<>(List.of(10, 20, 30, 40, 50));

// To remove by value, use Integer.valueOf() to force the Object overload
numbers.remove(Integer.valueOf(20));  // Removes the value 20

// Or use the boxed type explicitly
Integer target = 20;
numbers.remove(target);  // Calls remove(Object o)
```

**Why it is wrong:** The `List` interface has two `remove` methods: `remove(int index)` and `remove(Object o)`. When you pass an `int` literal, Java always calls `remove(int index)` because the primitive type matches exactly. To call `remove(Object o)`, you must pass an `Integer` object. This ambiguity only affects `List<Integer>` and is a common source of `IndexOutOfBoundsException` bugs.

### Mistake 3: Modifying a list returned by `Arrays.asList()` or `List.of()`

**Wrong:**

```java
// Arrays.asList() returns a fixed-size list backed by the original array.
// You can modify elements (set) but cannot add or remove.
List<String> list = Arrays.asList("A", "B", "C");
list.set(0, "X");     // OK
list.add("D");        // UnsupportedOperationException!
list.remove(0);       // UnsupportedOperationException!

// List.of() returns a fully immutable list.
List<String> immutable = List.of("A", "B", "C");
immutable.set(0, "X");  // UnsupportedOperationException!
```

**Right:**

```java
// If you need a modifiable list, wrap it in a new ArrayList
List<String> modifiable = new ArrayList<>(Arrays.asList("A", "B", "C"));
modifiable.add("D");  // OK
modifiable.remove(0);  // OK

// Or use the ArrayList constructor with List.of()
List<String> modifiable2 = new ArrayList<>(List.of("A", "B", "C"));
```

**Why it is wrong:** `Arrays.asList()` returns a `java.util.Arrays$ArrayList` (an inner class of `Arrays`), not a `java.util.ArrayList`. It is a fixed-size wrapper around the original array. `List.of()` returns a truly immutable list. If you need a fully modifiable list, always wrap the result in `new ArrayList<>(...)`.

### Mistake 4: Ignoring the cost of `contains()` on large lists

**Wrong:**

```java
List<Long> activeUserIds = userRepository.findAllActiveUserIds();  // 1 million IDs

for (Order order : orders) {
    if (activeUserIds.contains(order.getUserId())) {  // O(n) per call!
        processOrder(order);
    }
}
// Total cost: O(orders * activeUserIds) = O(10,000 * 1,000,000) = 10 billion operations!
```

**Right:**

```java
List<Long> activeUserIds = userRepository.findAllActiveUserIds();
Set<Long> activeUserIdSet = new HashSet<>(activeUserIds);  // O(n) to build the set

for (Order order : orders) {
    if (activeUserIdSet.contains(order.getUserId())) {  // O(1) per call!
        processOrder(order);
    }
}
// Total cost: O(activeUserIds + orders) = O(1,010,000) operations.
// 10,000x faster!
```

**Why it is wrong:** `List.contains()` performs a linear search (O(n)). When called inside a loop over another collection, the total cost becomes O(n * m), which is catastrophic for large datasets. Converting the list to a `HashSet` takes O(n) once, and then each `contains()` call is O(1). This single change can reduce a 10-minute operation to a 1-second operation.

---

## Key Takeaways

> [!tip] Remember these points
> 1. `List` is an ordered collection that allows duplicates and provides positional access by index. It is the most commonly used collection type in backend development. Always declare variables and method parameters as `List<T>`, not `ArrayList<T>`.
> 2. **`ArrayList`** is backed by a resizable array. It provides O(1) random access, O(1) amortized append, and O(n) insertion/deletion in the middle. It has excellent cache locality and low memory overhead. **Default to `ArrayList` for almost all use cases.**
> 3. **`LinkedList`** is backed by a doubly-linked list. It provides O(1) insertion/deletion at the ends but O(n) random access. It uses 6x more memory per element than `ArrayList` and has poor cache locality. In practice, `ArrayList` outperforms `LinkedList` for almost all operations due to CPU cache effects.
> 4. Use `List.of()` and `List.copyOf()` for **immutable lists** that cannot be modified after creation. Use `Collections.unmodifiableList()` to create a read-only view of a mutable list. Use `new ArrayList<>(existingList)` to create a modifiable copy.
> 5. **Never use `List.contains()` for membership testing on large collections inside a loop.** Convert the list to a `HashSet` first for O(1) lookups. Be careful with `remove()` on `List<Integer>`: use `remove(Integer.valueOf(n))` to remove by value and `remove(n)` to remove by index.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: ArrayList CRUD Operations (Easy)

Create an `ArrayList` of `String` representing a to-do list. Implement the following operations using `ArrayList` methods:

1. Add 5 tasks.
2. Insert a high-priority task at the beginning (index 0).
3. Replace the third task with a new task.
4. Remove a task by value.
5. Remove a task by index.
6. Check if a specific task exists.
7. Find the index of a specific task.
8. Print the final list with indices.

> **Hint:** Use `add(index, element)` for insertion, `set(index, element)` for replacement, and `indexOf()` for searching.

### Exercise 2: List Sorting and Filtering (Medium)

Create a `Student` record with fields `name`, `department`, and `cgpa`. Create a list of 8-10 students with varying data. Then:

1. Sort the list by CGPA descending.
2. Filter the list to include only students with CGPA >= 3.5 (store in a new list).
3. Sort the filtered list by name alphabetically.
4. Extract a sublist of the top 3 students.
5. Print all results.

> **Hint:** Use `Comparator.comparingDouble(Student::cgpa).reversed()` for sorting. Use a loop or `removeIf()` for filtering. Use `subList(0, 3)` for the top 3.

### Exercise 3: ArrayList vs LinkedList Benchmark (Medium)

Write a program that benchmarks `ArrayList` vs `LinkedList` for the following operations with 1,000,000 elements:

1. Append 1,000,000 integers to the end.
2. Access the element at index 500,000 (1000 times).
3. Insert an element at index 0 (1000 times).
4. Remove the element at index 0 (1000 times).

Use `System.nanoTime()` for timing. Print the results in a formatted table. Write comments explaining why each implementation wins or loses each benchmark.

> **Hint:** For the insertion and deletion benchmarks, use a small number of operations (1000) because `LinkedList`'s O(n) traversal to find the index makes large numbers impractical. The key insight is that `ArrayList`'s `System.arraycopy()` is faster than `LinkedList`'s node traversal even for middle insertions.

### Exercise 4: Order Processing Pipeline with Lists (Hard, Optional)

Build a simplified order processing pipeline:

1. Create an `Order` record with fields `orderId`, `userId`, `totalAmount`, `status`.
2. Create a list of 20 orders with random data (different users, amounts, and statuses: PENDING, CONFIRMED, SHIPPED, CANCELLED).
3. Implement the following methods:
   - `getPendingOrders(List<Order>)`: returns a new list of only PENDING orders, sorted by amount descending.
   - `getOrdersByUser(List<Order>, Long userId)`: returns a list of orders for a specific user.
   - `getOrderSummary(List<Order>)`: returns a `Map<String, List<Order>>` grouped by status.
   - `cancelAllPending(List<Order>)`: modifies the original list, changing all PENDING orders to CANCELLED. Returns the count of cancelled orders.
4. In `main()`, test all methods and print the results.

> **Hint:** For `getOrderSummary()`, use `computeIfAbsent()` to build the map of lists. For `cancelAllPending()`, iterate with a regular for loop (not for-each) since you are modifying elements in place (not adding/removing from the list).

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.*;

public class TodoList {
    public static void main(String[] args) {
        List<String> tasks = new ArrayList<>();

        // 1. Add 5 tasks
        tasks.add("Study Java Collections");
        tasks.add("Complete Spring Boot project");
        tasks.add("Write unit tests");
        tasks.add("Review pull requests");
        tasks.add("Update documentation");
        System.out.println("Initial: " + tasks);

        // 2. Insert at beginning
        tasks.add(0, "URGENT: Fix production bug");
        System.out.println("After insert: " + tasks);

        // 3. Replace third task (index 2)
        tasks.set(2, "Complete Spring Boot project (revised deadline)");
        System.out.println("After replace: " + tasks);

        // 4. Remove by value
        tasks.remove("Update documentation");
        System.out.println("After remove by value: " + tasks);

        // 5. Remove by index (remove index 3)
        tasks.remove(3);
        System.out.println("After remove by index: " + tasks);

        // 6. Check existence
        System.out.println("Contains 'Write unit tests': " + tasks.contains("Write unit tests"));

        // 7. Find index
        System.out.println("Index of 'Write unit tests': " + tasks.indexOf("Write unit tests"));

        // 8. Print with indices
        System.out.println("\n--- Final To-Do List ---");
        for (int i = 0; i < tasks.size(); i++) {
            System.out.println("  [" + i + "] " + tasks.get(i));
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;

record Student(String name, String department, double cgpa) {}

public class Main {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>(List.of(
            new Student("Saad", "CSE", 3.72),
            new Student("Rahim", "CSE", 3.45),
            new Student("Karim", "EEE", 3.90),
            new Student("Fatima", "EEE", 3.80),
            new Student("Nila", "BBA", 3.60),
            new Student("Arif", "CSE", 3.55),
            new Student("Tania", "BBA", 3.30),
            new Student("Hasan", "EEE", 3.95)
        ));

        // 1. Sort by CGPA descending
        students.sort(Comparator.comparingDouble(Student::cgpa).reversed());
        System.out.println("By CGPA (desc):");
        students.forEach(s -> System.out.printf("  %s: %.2f%n", s.name(), s.cgpa()));

        // 2. Filter CGPA >= 3.5
        List<Student> topStudents = new ArrayList<>();
        for (Student s : students) {
            if (s.cgpa() >= 3.5) topStudents.add(s);
        }

        // 3. Sort filtered by name
        topStudents.sort(Comparator.comparing(Student::name));
        System.out.println("\nTop students (>= 3.5) by name:");
        topStudents.forEach(s -> System.out.printf("  %s: %.2f%n", s.name(), s.cgpa()));

        // 4. Top 3 sublist
        List<Student> top3 = topStudents.subList(0, Math.min(3, topStudents.size()));
        System.out.println("\nTop 3:");
        top3.forEach(s -> System.out.printf("  %s: %.2f%n", s.name(), s.cgpa()));
    }
}
```

</details>

---

## Related Notes

- [[Java - Collections Framework Overview]]
- [[Java - Set - HashSet TreeSet LinkedHashSet]] (next note)
- [[Java - Map - HashMap TreeMap LinkedHashMap]]
- [[Java - Java 8 Streams API]]

---

## Resources

- [Oracle Java Tutorials: The List Interface](https://docs.oracle.com/javase/tutorial/collections/interfaces/list.html) - Official documentation covering the List contract and operations.
- [Oracle Java Documentation: ArrayList](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html) - Complete API reference for ArrayList.
- [Baeldung: ArrayList vs LinkedList](https://www.baeldung.com/java-arraylist-linkedlist) - Detailed performance comparison with benchmarks.
- [Baeldung: Java List Guide](https://www.baeldung.com/java-list) - Comprehensive guide covering all List operations and implementations.
- [Baeldung: Java Immutable List](https://www.baeldung.com/java-immutable-list) - Guide to `List.of()`, `List.copyOf()`, and `Collections.unmodifiableList()`.
- [Effective Java by Joshua Bloch - Item 28: Prefer Lists to Arrays](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Why Lists are safer and more flexible than arrays.
