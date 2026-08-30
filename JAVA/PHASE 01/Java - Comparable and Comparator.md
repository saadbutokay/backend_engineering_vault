---
title: "Java - Comparable and Comparator"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - comparable
  - comparator
  - sorting
  - ordering
status: "not-started"
---

# Java - Comparable and Comparator

> [!abstract] Overview
> `Comparable` and `Comparator` are the two interfaces that define how Java objects are ordered and sorted. `Comparable` defines the **natural ordering** of a class (the default way objects of that type should be sorted), while `Comparator` defines **external, customizable orderings** that can be applied without modifying the class itself. Together, they power every sorting operation in Java: `Collections.sort()`, `List.sort()`, `TreeSet`, `TreeMap`, `PriorityQueue`, `Arrays.sort()`, and the Streams API's `sorted()` method. In backend development, sorting is essential for paginated API responses, leaderboard rankings, chronological event processing, and database query result ordering.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Abstraction - Abstract Classes and Interfaces]]
> - [[Java - Generics - Classes Methods Wildcards]]
> - [[Java - List - ArrayList LinkedList]]
> - [[Java - Set - HashSet TreeSet LinkedHashSet]]
> - [[Java - Map - HashMap TreeMap LinkedHashMap]]

---

## Theory

### The `Comparable<T>` Interface

The `Comparable<T>` interface defines the **natural ordering** of a class. A class implements `Comparable<T>` when there is a single, obvious, default way to sort its instances. The interface has a single method:

```java
public interface Comparable<T> {
    int compareTo(T other);
}
```

The `compareTo()` method returns:

| Return Value | Meaning |
|-------------|---------|
| Negative integer | `this` is less than `other` (comes before) |
| Zero | `this` is equal to `other` (same position) |
| Positive integer | `this` is greater than `other` (comes after) |

**Classes that already implement `Comparable`:**

| Class | Natural Ordering |
|-------|-----------------|
| `String` | Lexicographic (alphabetical, case-sensitive, based on Unicode values) |
| `Integer`, `Long`, `Double` | Numeric ascending |
| `BigDecimal` | Numeric ascending (by value, not scale) |
| `LocalDate`, `LocalDateTime` | Chronological ascending |
| `Boolean` | `false` before `true` |
| `UUID` | Lexicographic by UUID string |

**Example of natural ordering:**

```java
String a = "Apple";
String b = "Banana";
System.out.println(a.compareTo(b));  // Negative (A < B alphabetically)
System.out.println(b.compareTo(a));  // Positive (B > A)
System.out.println(a.compareTo("Apple"));  // 0 (equal)
```

### The `Comparator<T>` Interface

The `Comparator<T>` interface defines an **external ordering** that can be applied to objects without modifying their class. This is essential when:

1. The class does not implement `Comparable` (e.g., a third-party class you cannot modify).
2. You need multiple different orderings for the same class (e.g., sort users by name, by age, or by registration date).
3. You want to override the natural ordering (e.g., sort strings by length instead of alphabetically).

The interface has one abstract method and many default/static methods (Java 8+):

```java
public interface Comparator<T> {
    int compare(T o1, T o2);  // Abstract method

    // Java 8+ default and static methods (covered below)
    default Comparator<T> reversed() { ... }
    default Comparator<T> thenComparing(...) { ... }
    static <T> Comparator<T> comparing(...) { ... }
    static <T> Comparator<T> naturalOrder() { ... }
    static <T> Comparator<T> reverseOrder() { ... }
    static <T> Comparator<T> nullsFirst(...) { ... }
    static <T> Comparator<T> nullsLast(...) { ... }
}
```

The `compare()` method follows the same contract as `compareTo()`:

| Return Value | Meaning |
|-------------|---------|
| Negative integer | `o1` comes before `o2` |
| Zero | `o1` and `o2` are equal in this ordering |
| Positive integer | `o1` comes after `o2` |

### `Comparable` vs `Comparator`

| Aspect | Comparable | Comparator |
|--------|-----------|------------|
| Package | `java.lang` (auto-imported) | `java.util` |
| Method | `compareTo(T other)` | `compare(T o1, T o2)` |
| Defined in | The class itself (internal) | A separate class or lambda (external) |
| Number of orderings | One (natural ordering) | Unlimited (multiple comparators) |
| Modifies the class | Yes (implements interface) | No (external to the class) |
| Use when | There is one obvious default ordering | You need custom or multiple orderings |
| Used by | `Collections.sort(list)`, `TreeSet`, `TreeMap` | `Collections.sort(list, comparator)`, `list.sort(comparator)` |
| Example | `String`, `Integer`, `LocalDate` | Sort users by age, by name length, by email |

**When to implement `Comparable`:**

- When your class has a single, universally agreed-upon natural ordering.
- When you want your objects to work automatically with `TreeSet`, `TreeMap`, and `Collections.sort()` without specifying a comparator.
- When the ordering is an inherent property of the domain concept (e.g., dates are naturally chronological, numbers are naturally numeric).

**When to use `Comparator`:**

- When you need multiple orderings for the same class.
- When you cannot modify the class (third-party library).
- When the ordering is context-dependent (e.g., sort products by price in one view and by rating in another).
- When the natural ordering does not match the sorting requirement.

### The `compareTo()` Contract

The `compareTo()` method must satisfy three mathematical properties to ensure consistent sorting:

1. **Antisymmetry**: `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`. If x is less than y, then y is greater than x.
2. **Transitivity**: If `x.compareTo(y) > 0` and `y.compareTo(z) > 0`, then `x.compareTo(z) > 0`. If A beats B and B beats C, then A beats C.
3. **Consistency with equals** (strongly recommended): `x.compareTo(y) == 0` should imply `x.equals(y)`. If two objects compare as equal, they should also be `equals()`. This is critical for `TreeSet` and `TreeMap`, which use `compareTo()` instead of `equals()` to determine equality.

**Violation of the contract leads to unpredictable behavior:**

- `Collections.sort()` may produce incorrect results or throw `IllegalArgumentException` ("Comparison method violates its general contract").
- `TreeSet` may silently drop elements that it considers duplicates based on `compareTo()` but are not `equals()`.
- `TreeMap` may lose entries or fail to find existing keys.

### Java 8+ Comparator Factory Methods

Java 8 introduced powerful static and default methods on the `Comparator` interface that make creating comparators concise and readable. These methods are the preferred way to create comparators in modern Java.

| Method | Description | Example |
|--------|-------------|---------|
| `Comparator.comparing(keyExtractor)` | Compare by a single field | `comparing(User::getName)` |
| `Comparator.comparingInt(keyExtractor)` | Compare by an int field (avoids boxing) | `comparingInt(User::getAge)` |
| `Comparator.comparingDouble(keyExtractor)` | Compare by a double field | `comparingDouble(Product::getPrice)` |
| `Comparator.comparingLong(keyExtractor)` | Compare by a long field | `comparingLong(Order::getId)` |
| `Comparator.naturalOrder()` | Use the natural ordering | `Comparator.naturalOrder()` |
| `Comparator.reverseOrder()` | Reverse the natural ordering | `Comparator.reverseOrder()` |
| `Comparator.nullsFirst(comp)` | Nulls sort before non-nulls | `nullsFirst(comparing(User::getName))` |
| `Comparator.nullsLast(comp)` | Nulls sort after non-nulls | `nullsLast(comparing(User::getName))` |
| `comparator.reversed()` | Reverse the comparator | `comparing(User::getAge).reversed()` |
| `comparator.thenComparing(keyExtractor)` | Secondary sort key | `comparing(User::getAge).thenComparing(User::getName)` |

### How Sorting Works Internally

Java's `Collections.sort()` and `List.sort()` use **TimSort**, a hybrid sorting algorithm derived from merge sort and insertion sort. It was designed by Tim Peters for Python and adopted by Java in Java 7.

**TimSort characteristics:**

- **Time complexity**: O(n log n) worst case, O(n) best case (already sorted data).
- **Space complexity**: O(n) auxiliary space.
- **Stable**: Equal elements maintain their relative order. This is important when sorting by multiple criteria.
- **Adaptive**: It detects and exploits existing order in the data (runs of already-sorted elements), making it extremely fast for partially sorted data, which is common in backend systems.

When you call `list.sort(comparator)`, the JVM:

1. Converts the list to an array internally.
2. Runs TimSort on the array, using the comparator's `compare()` method to determine element order.
3. Copies the sorted array back into the list.

For `TreeSet` and `TreeMap`, the ordering is maintained by a **red-black tree**. Each insertion calls `compareTo()` (or the `Comparator.compare()`) to find the correct position in the tree. The tree self-balances to guarantee O(log n) operations.

> [!tip] Key Insight
> The most important practical difference between `Comparable` and `Comparator` is **ownership**. `Comparable` is implemented by the class author and defines the "one true ordering" for that type. `Comparator` is created by the class consumer and defines a "context-specific ordering." In backend development, you will use `Comparator` far more often than `Comparable` because API endpoints frequently need to sort the same data in different ways depending on the client's request (e.g., `?sortBy=price&sortDir=desc`).

---

## Syntax and Basic Examples

### Example 1: Implementing Comparable

```java
public class Student implements Comparable<Student> {
    private String name;
    private double cgpa;
    private int semester;

    public Student(String name, double cgpa, int semester) {
        this.name = name;
        this.cgpa = cgpa;
        this.semester = semester;
    }

    // Natural ordering: sort by CGPA descending (highest first),
    // then by name ascending (alphabetical) for ties.
    @Override
    public int compareTo(Student other) {
        // Primary sort: CGPA descending
        int cgpaComparison = Double.compare(other.cgpa, this.cgpa);
        // Note: other.cgpa comes first to reverse the order (descending)

        if (cgpaComparison != 0) {
            return cgpaComparison;
        }

        // Secondary sort: name ascending (alphabetical)
        return this.name.compareTo(other.name);
    }

    @Override
    public String toString() {
        return String.format("%s (CGPA: %.2f, Sem: %d)", name, cgpa, semester);
    }

    // Getters
    public String getName() { return name; }
    public double getCgpa() { return cgpa; }
    public int getSemester() { return semester; }
}
```

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>(List.of(
            new Student("Saad", 3.72, 6),
            new Student("Rahim", 3.45, 5),
            new Student("Karim", 3.90, 7),
            new Student("Arif", 3.72, 6),
            new Student("Nila", 3.80, 4)
        ));

        // Collections.sort() uses the natural ordering (compareTo)
        Collections.sort(students);

        System.out.println("Sorted by natural ordering (CGPA desc, name asc):");
        for (Student s : students) {
            System.out.println("  " + s);
        }
        // Karim (CGPA: 3.90, Sem: 7)
        // Nila (CGPA: 3.80, Sem: 4)
        // Arif (CGPA: 3.72, Sem: 6)    <- Arif before Saad (alphabetical tiebreaker)
        // Saad (CGPA: 3.72, Sem: 6)
        // Rahim (CGPA: 3.45, Sem: 5)
    }
}
```

### Example 2: Comparator with Java 8+ factory methods

```java
import java.util.*;

public class ComparatorDemo {

    record Product(String name, double price, int rating, String category) {}

    public static void main(String[] args) {
        List<Product> products = new ArrayList<>(List.of(
            new Product("Laptop", 85000, 4, "Electronics"),
            new Product("Mouse", 1500, 5, "Accessories"),
            new Product("Keyboard", 3200, 4, "Accessories"),
            new Product("Monitor", 25000, 3, "Electronics"),
            new Product("Webcam", 4500, 4, "Accessories")
        ));

        // 1. Sort by price ascending
        products.sort(Comparator.comparingDouble(Product::price));
        System.out.println("By price (asc):");
        products.forEach(p -> System.out.printf("  %-10s %,.0f BDT%n", p.name(), p.price()));

        // 2. Sort by price descending
        products.sort(Comparator.comparingDouble(Product::price).reversed());
        System.out.println("\nBy price (desc):");
        products.forEach(p -> System.out.printf("  %-10s %,.0f BDT%n", p.name(), p.price()));

        // 3. Sort by rating descending, then by price ascending
        products.sort(
            Comparator.comparingInt(Product::rating).reversed()
                .thenComparingDouble(Product::price)
        );
        System.out.println("\nBy rating (desc), then price (asc):");
        products.forEach(p -> System.out.printf("  %-10s Rating: %d, %,.0f BDT%n",
            p.name(), p.rating(), p.price()));

        // 4. Sort by category, then by name
        products.sort(
            Comparator.comparing(Product::category)
                .thenComparing(Product::name)
        );
        System.out.println("\nBy category, then name:");
        products.forEach(p -> System.out.printf("  [%s] %s%n", p.category(), p.name()));

        // 5. Sort by name length
        products.sort(Comparator.comparingInt(p -> p.name().length()));
        System.out.println("\nBy name length:");
        products.forEach(p -> System.out.printf("  %s (%d chars)%n", p.name(), p.name().length()));
    }
}
```

### Example 3: Comparator with Collections and sorted data structures

```java
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.*;

public class ComparatorWithDataStructures {

    record Order(long id, BigDecimal total, LocalDateTime createdAt) {}

    public static void main(String[] args) {
        // TreeSet with custom comparator: orders sorted by total descending
        TreeSet<Order> ordersByTotal = new TreeSet<>(
            Comparator.comparing(Order::total).reversed()
                .thenComparingLong(Order::id)  // Tiebreaker to prevent lost entries
        );

        ordersByTotal.add(new Order(1, new BigDecimal("5000"), LocalDateTime.now()));
        ordersByTotal.add(new Order(2, new BigDecimal("12000"), LocalDateTime.now()));
        ordersByTotal.add(new Order(3, new BigDecimal("3000"), LocalDateTime.now()));
        ordersByTotal.add(new Order(4, new BigDecimal("12000"), LocalDateTime.now()));

        System.out.println("Orders by total (desc):");
        for (Order o : ordersByTotal) {
            System.out.printf("  #%d: %s BDT%n", o.id(), o.total());
        }
        // #2: 12000 BDT
        // #4: 12000 BDT  (same total, sorted by ID)
        // #1: 5000 BDT
        // #3: 3000 BDT

        // TreeMap with custom comparator: keys sorted by string length
        TreeMap<String, Integer> byLength = new TreeMap<>(
            Comparator.comparingInt(String::length).thenComparing(Comparator.naturalOrder())
        );
        byLength.put("Dhaka", 1);
        byLength.put("Sylhet", 2);
        byLength.put("Rajshahi", 3);
        byLength.put("Khulna", 4);

        System.out.println("\nCities by name length:");
        byLength.forEach((city, code) ->
            System.out.printf("  %s (%d)%n", city, city.length())
        );
        // Dhaka (5)
        // Khulna (6)
        // Sylhet (6)
        // Rajshahi (8)

        // PriorityQueue with custom comparator: highest priority first
        PriorityQueue<String> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
        maxHeap.addAll(List.of("Banana", "Apple", "Cherry", "Date"));
        System.out.println("\nMax-heap poll order:");
        while (!maxHeap.isEmpty()) {
            System.out.print(maxHeap.poll() + " ");
        }
        // Date Cherry Banana Apple
    }
}
```

### Example 4: Handling nulls in sorting

```java
import java.util.*;

public class NullHandlingDemo {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(Arrays.asList(
            "Saad", null, "Rahim", null, "Karim", "Arif"
        ));

        // Without null handling: NullPointerException!
        // names.sort(Comparator.naturalOrder());  // CRASH

        // Nulls first
        names.sort(Comparator.nullsFirst(Comparator.naturalOrder()));
        System.out.println("Nulls first: " + names);
        // [null, null, Arif, Karim, Rahim, Saad]

        // Nulls last
        names.sort(Comparator.nullsLast(Comparator.naturalOrder()));
        System.out.println("Nulls last: " + names);
        // [Arif, Karim, Rahim, Saad, null, null]

        // Nulls last, reverse order
        names.sort(Comparator.nullsLast(Comparator.reverseOrder()));
        System.out.println("Nulls last, reverse: " + names);
        // [Saad, Rahim, Karim, Arif, null, null]
    }
}
```

### Example 5: Multi-level sorting with thenComparing

```java
import java.util.*;

public class MultiLevelSortDemo {

    record Employee(String name, String department, int yearsOfService, double salary) {}

    public static void main(String[] args) {
        List<Employee> employees = new ArrayList<>(List.of(
            new Employee("Saad", "Engineering", 3, 85000),
            new Employee("Rahim", "Engineering", 5, 95000),
            new Employee("Karim", "Marketing", 3, 70000),
            new Employee("Nila", "Marketing", 7, 90000),
            new Employee("Arif", "Engineering", 3, 90000),
            new Employee("Fatima", "HR", 2, 60000)
        ));

        // Sort by department (asc), then years of service (desc), then salary (desc)
        employees.sort(
            Comparator.comparing(Employee::department)
                .thenComparing(Comparator.comparingInt(Employee::yearsOfService).reversed())
                .thenComparing(Comparator.comparingDouble(Employee::salary).reversed())
        );

        System.out.println("Sorted by dept, then experience (desc), then salary (desc):");
        for (Employee e : employees) {
            System.out.printf("  %-8s | %-12s | %d yrs | %,.0f BDT%n",
                e.name(), e.department(), e.yearsOfService(), e.salary());
        }
        // Rahim    | Engineering  | 5 yrs | 95,000 BDT
        // Arif     | Engineering  | 3 yrs | 90,000 BDT
        // Saad     | Engineering  | 3 yrs | 85,000 BDT
        // Fatima   | HR           | 2 yrs | 60,000 BDT
        // Nila     | Marketing    | 7 yrs | 90,000 BDT
        // Karim    | Marketing    | 3 yrs | 70,000 BDT
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Sorting is a fundamental requirement in almost every backend API. Here are three realistic scenarios.

### Scenario 1: Dynamic sorting in REST API responses

Backend APIs often allow clients to specify the sort field and direction via query parameters. This requires building comparators dynamically at runtime.

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.util.Comparator;
import java.util.List;

@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public List<OrderResponse> getOrders(String sortBy, String sortDir) {
        List<Order> orders = orderRepository.findAll();

        // Build the comparator dynamically based on client request
        Comparator<Order> comparator = buildComparator(sortBy);

        // Apply sort direction
        if ("desc".equalsIgnoreCase(sortDir)) {
            comparator = comparator.reversed();
        }

        // Sort and transform
        return orders.stream()
            .sorted(comparator)
            .map(OrderResponse::fromEntity)
            .toList();
    }

    private Comparator<Order> buildComparator(String sortBy) {
        if (sortBy == null) sortBy = "createdAt";  // Default sort

        return switch (sortBy.toLowerCase()) {
            case "total", "amount" ->
                Comparator.comparing(Order::getTotalAmount);
            case "status" ->
                Comparator.comparing(o -> o.getStatus().name());
            case "ordernumber", "order_number" ->
                Comparator.comparing(Order::getOrderNumber);
            case "createdat", "created_at", "date" ->
                Comparator.comparing(Order::getCreatedAt);
            default ->
                Comparator.comparing(Order::getCreatedAt);  // Fallback
        };
    }
}
```

```java
// Controller:
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    // GET /api/v1/orders?sortBy=total&sortDir=desc
    @GetMapping
    public ResponseEntity<List<OrderResponse>> getOrders(
            @RequestParam(defaultValue = "createdAt") String sortBy,
            @RequestParam(defaultValue = "desc") String sortDir) {
        return ResponseEntity.ok(orderService.getOrders(sortBy, sortDir));
    }
}
```

**What to notice:**

- The `buildComparator()` method uses a `switch` expression to map the client's sort field string to a `Comparator<Order>`. This is the standard pattern for dynamic sorting in backend APIs.
- `comparator.reversed()` flips the sort direction based on the `sortDir` parameter. This avoids duplicating the comparator logic for ascending and descending.
- In production, sorting is usually done at the database level using Spring Data's `Sort` object (`PageRequest.of(page, size, Sort.by("totalAmount").descending())`), which generates an `ORDER BY` clause in the SQL query. Application-level sorting (as shown here) is only appropriate for small datasets or when the data is already in memory.

### Scenario 2: Comparable for domain entities with natural ordering

Some domain entities have a clear natural ordering that is used consistently across the application.

```java
package com.company.orderservice.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.ManyToOne;
import jakarta.persistence.Table;
import java.math.BigDecimal;

@Entity
@Table(name = "order_items")
public class OrderItem implements Comparable<OrderItem> {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    private Product product;

    private int quantity;

    private BigDecimal unitPrice;

    // Natural ordering: sort by line total descending (highest value items first).
    // This is the most common way to display order items in invoices and receipts.
    @Override
    public int compareTo(OrderItem other) {
        BigDecimal thisTotal = this.unitPrice.multiply(BigDecimal.valueOf(this.quantity));
        BigDecimal otherTotal = other.unitPrice.multiply(BigDecimal.valueOf(other.quantity));
        int totalComparison = otherTotal.compareTo(thisTotal);  // Descending

        if (totalComparison != 0) {
            return totalComparison;
        }

        // Tiebreaker: sort by product name ascending
        return this.product.getName().compareTo(other.product.getName());
    }

    // Getters, setters, constructors...
}
```

```java
// Usage: Order items are automatically sorted when displayed
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public OrderResponse buildOrderResponse(Order order) {
    List<OrderItem> items = new ArrayList<>(order.getItems());
    Collections.sort(items);  // Uses compareTo(): highest value items first

    return new OrderResponse(
        order.getId(),
        order.getOrderNumber(),
        items.stream().map(OrderItemResponse::fromEntity).toList(),
        order.getTotalAmount()
    );
}
```

**What to notice:**

- `OrderItem` implements `Comparable<OrderItem>` because there is a single, universally agreed-upon ordering: highest value items first. This ordering is used in invoices, receipts, and order detail pages throughout the application.
- The tiebreaker (product name) ensures consistent ordering even when two items have the same total. Without a tiebreaker, the sort order would be unstable and could change between requests.
- `Collections.sort(items)` uses the natural ordering automatically. No comparator needs to be specified.

### Scenario 3: Priority-based task processing with Comparator

```java
package com.company.orderservice.scheduler;

import org.springframework.stereotype.Service;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.PriorityQueue;

@Service
public class TaskQueue {

    // Comparator: highest priority first, then earliest deadline
    private static final Comparator<Task> TASK_COMPARATOR =
        Comparator.comparingInt(Task::priority)  // Lower number = higher priority
            .thenComparing(Task::deadline)        // Earlier deadline first
            .thenComparingLong(Task::createdAt);  // FIFO for ties

    private final PriorityQueue<Task> queue = new PriorityQueue<>(TASK_COMPARATOR);

    public synchronized void submit(Task task) {
        queue.offer(task);
    }

    public synchronized Task pollNext() {
        return queue.poll();  // Returns the highest-priority task
    }

    public synchronized List<Task> pollBatch(int maxTasks) {
        List<Task> batch = new ArrayList<>();
        while (batch.size() < maxTasks && !queue.isEmpty()) {
            batch.add(queue.poll());
        }
        return batch;
    }

    public synchronized int size() {
        return queue.size();
    }
}

record Task(
    String name,
    int priority,       // 1 = critical, 2 = high, 3 = normal, 4 = low
    LocalDateTime deadline,
    long createdAt,
    Runnable action
) {}
```

```java
// Usage:
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.util.List;

@Component
public class TaskProcessor {

    private final TaskQueue taskQueue;

    @Scheduled(fixedDelay = 1000)
    public void processTasks() {
        List<Task> batch = taskQueue.pollBatch(10);

        for (Task task : batch) {
            try {
                task.action().run();
            } catch (Exception e) {
                // logger.error("Task failed: {}", task.name(), e);
            }
        }
    }
}
```

**What to notice:**

- The `TASK_COMPARATOR` is defined as a `static final` constant because it is immutable and shared across all instances. This avoids creating a new comparator object on every use.
- The three-level comparator chain (`priority` -> `deadline` -> `createdAt`) ensures that critical tasks are processed first, then tasks with earlier deadlines, and finally tasks in FIFO order for ties. This is a common pattern in job scheduling systems.
- The `PriorityQueue` uses the comparator to maintain the heap invariant. `poll()` always returns the task with the highest priority (lowest priority number, earliest deadline).

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Inconsistent `compareTo()` and `equals()`

**Wrong:**

```java
public class Order implements Comparable<Order> {
    private Long id;
    private BigDecimal total;

    @Override
    public int compareTo(Order other) {
        return this.total.compareTo(other.total);  // Sort by total
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Order other)) return false;
        return this.id.equals(other.id);  // Equal by ID
    }

    // Problem: two orders with the same total but different IDs
    // compareTo() returns 0 (equal), but equals() returns false (different).
    // TreeSet will treat them as duplicates and silently drop one!
}
```

**Right:**

```java
@Override
public int compareTo(Order other) {
    int totalComparison = this.total.compareTo(other.total);
    if (totalComparison != 0) return totalComparison;
    return this.id.compareTo(other.id);  // Tiebreaker: ensures compareTo == 0 only when equals is true
}
```

**Why it is wrong:** `TreeSet` and `TreeMap` use `compareTo()` (not `equals()`) to determine equality. If `compareTo()` returns 0 for two objects that `equals()` considers different, the `TreeSet` will silently drop one of them. The fix is to add a tiebreaker field (usually the ID) to `compareTo()` so that it returns 0 only when the objects are truly equal.

### Mistake 2: Using subtraction for comparison (integer overflow)

**Wrong:**

```java
@Override
public int compareTo(Student other) {
    return this.age - other.age;  // DANGEROUS! Integer overflow!
    // If this.age = 2,000,000,000 and other.age = -2,000,000,000,
    // the subtraction overflows and returns a negative number,
    // incorrectly indicating that this.age < other.age.
}
```

**Right:**

```java
@Override
public int compareTo(Student other) {
    return Integer.compare(this.age, other.age);  // Safe: handles overflow correctly
}
```

**Why it is wrong:** The subtraction trick (`a - b`) works for small positive numbers but fails catastrophically for large numbers or negative numbers due to integer overflow. `Integer.compare()`, `Long.compare()`, and `Double.compare()` are specifically designed to handle all edge cases correctly, including overflow, NaN, and negative zero. Always use the static `compare()` methods instead of subtraction.

### Mistake 3: Forgetting the tiebreaker in multi-level sorting

**Wrong:**

```java
// Sorting products by rating only
products.sort(Comparator.comparingInt(Product::rating));
// Products with the same rating appear in an unpredictable order.
// The sort is stable (preserves original order for equal elements),
// but the original order may vary between database queries.
```

**Right:**

```java
// Sorting by rating, then by name for consistent results
products.sort(
    Comparator.comparingInt(Product::rating)
        .thenComparing(Product::name)
);
// Products with the same rating are always in alphabetical order.
// The result is deterministic and reproducible across requests.
```

**Why it is wrong:** Without a tiebreaker, elements with the same sort key appear in an implementation-dependent order that can change between requests, between JVM restarts, or between database query results. This causes confusing UI behavior where the order of items changes on page refresh. Always add a unique tiebreaker (like an ID or name) as the final sort key to ensure deterministic ordering.

### Mistake 4: Implementing Comparable when multiple orderings are needed

**Wrong:**

```java
public class Product implements Comparable<Product> {
    // Natural ordering by price
    @Override
    public int compareTo(Product other) {
        return Double.compare(this.price, other.price);
    }
}

// But now the marketing team wants to sort by rating,
// and the warehouse team wants to sort by stock quantity.
// The single compareTo() cannot serve all three use cases.
```

**Right:**

```java
// Do NOT implement Comparable. Use external Comparators instead.
public class Product {
    // No compareTo(). The class does not impose a single ordering.
}

// Define comparators where they are needed:
public static final Comparator<Product> BY_PRICE =
    Comparator.comparingDouble(Product::getPrice);
public static final Comparator<Product> BY_RATING =
    Comparator.comparingInt(Product::getRating).reversed();
public static final Comparator<Product> BY_STOCK =
    Comparator.comparingInt(Product::getStockQuantity);

// Usage:
products.sort(Product.BY_PRICE);
products.sort(Product.BY_RATING);
```

**Why it is wrong:** `Comparable` defines a single natural ordering. If your class needs to be sorted in multiple ways depending on the context, `Comparable` is the wrong tool. Use `Comparator` instances instead, which can be defined externally and composed freely. Implement `Comparable` only when there is one universally agreed-upon ordering (like chronological for dates or alphabetical for names).

---

## Key Takeaways

> [!tip] Remember these points
> 1. **`Comparable<T>`** defines the natural ordering of a class via `compareTo()`. Implement it when there is one obvious default ordering. Classes like `String`, `Integer`, and `LocalDate` already implement it. `TreeSet` and `TreeMap` use it by default.
> 2. **`Comparator<T>`** defines external, customizable orderings via `compare()`. Use it when you need multiple orderings, cannot modify the class, or want to override the natural ordering. Java 8+ factory methods (`comparing()`, `thenComparing()`, `reversed()`) make creating comparators concise and readable.
> 3. The `compareTo()`/`compare()` contract requires **antisymmetry**, **transitivity**, and (strongly recommended) **consistency with equals**. Violating this contract causes `TreeSet` to drop elements, `TreeMap` to lose entries, and `Collections.sort()` to throw `IllegalArgumentException`.
> 4. Always use `Integer.compare()`, `Long.compare()`, and `Double.compare()` instead of subtraction (`a - b`) to avoid integer overflow. Always add a **unique tiebreaker** (like an ID) as the final sort key to ensure deterministic ordering.
> 5. In backend APIs, sorting is usually done at the **database level** using Spring Data's `Sort` object, which generates `ORDER BY` clauses. Application-level sorting with `Comparator` is appropriate for small in-memory datasets, dynamic sorting based on client parameters, and priority queues.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Comparable Student Class (Easy)

Create a `Student` class that implements `Comparable<Student>`. The natural ordering should be:

1. By CGPA descending (highest first).
2. By semester ascending (lower semester first) for ties.
3. By name ascending (alphabetical) for further ties.

Create a list of 6 students with varying data, sort it using `Collections.sort()`, and print the results. Verify that the ordering is correct.

> **Hint:** Use `Double.compare(other.cgpa, this.cgpa)` for descending CGPA (note the reversed argument order). Use `Integer.compare(this.semester, other.semester)` for ascending semester. Use `this.name.compareTo(other.name)` for alphabetical name.

### Exercise 2: Multiple Comparators for Products (Medium)

Create a `Product` record with fields `name`, `price`, `rating`, `category`, and `reviewCount`. Do NOT implement `Comparable`. Instead, define the following static `Comparator` constants in the class:

1. `BY_PRICE_ASC`: by price ascending.
2. `BY_PRICE_DESC`: by price descending.
3. `BY_RATING`: by rating descending, then by review count descending (more reviews = more reliable rating).
4. `BY_NAME`: by name alphabetically, case-insensitive.
5. `BY_CATEGORY_THEN_PRICE`: by category alphabetically, then by price ascending within each category.

Create a list of 8 products and sort it using each comparator. Print the results for each sorting.

> **Hint:** Use `Comparator.comparing(String::toLowerCase)` for case-insensitive string comparison. Use `thenComparing()` to chain secondary sort keys.

### Exercise 3: Dynamic Sort Builder (Medium)

Create a `SortBuilder<T>` class that builds a `Comparator<T>` dynamically from a list of sort criteria. Each criterion specifies a field name and a direction (ASC or DESC).

Implement a method `buildComparator(List<SortCriterion> criteria, Map<String, Function<T, Comparable>> fieldExtractors)` that constructs a chained comparator from the criteria.

Test it with a `User` record and sort criteria like `[("name", ASC), ("age", DESC)]`.

> **Hint:** Start with a `null` comparator. For each criterion, look up the field extractor from the map, create a comparator using `Comparator.comparing()`, reverse it if the direction is DESC, and chain it using `thenComparing()`. Handle the first criterion specially (no `thenComparing` needed).

### Exercise 4: Leaderboard System with TreeSet (Hard, Optional)

Build a leaderboard system for a gaming platform:

1. Create a `Player` record with fields `id`, `username`, `score`, `gamesPlayed`, and `lastActive` (LocalDateTime).
2. Create a `Leaderboard` class that uses a `TreeSet<Player>` with a custom comparator to maintain players in ranked order. The ranking criteria are:
   - Score descending (highest first).
   - Games played ascending (fewer games = more efficient, used as tiebreaker).
   - Last active descending (more recent = higher rank for further ties).
   - ID ascending (final unique tiebreaker to prevent lost entries).
3. Implement methods: `addPlayer(Player)`, `removePlayer(long id)`, `getTopN(int n)`, `getRank(long playerId)`, `getPlayersAround(long playerId, int range)`.
4. Add 15 players with varying scores and test all methods.

> **Hint:** The `getRank()` method can use `headSet()` to count how many players are ranked above the target player. The `getPlayersAround()` method can use `headSet()` and `tailSet()` to find the neighbors. Remember that `TreeSet` uses the comparator for equality, so the ID tiebreaker is critical to prevent players with the same score from being treated as duplicates.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.*;

class Student implements Comparable<Student> {
    String name;
    double cgpa;
    int semester;

    Student(String name, double cgpa, int semester) {
        this.name = name;
        this.cgpa = cgpa;
        this.semester = semester;
    }

    @Override
    public int compareTo(Student other) {
        int cgpaCmp = Double.compare(other.cgpa, this.cgpa);  // Descending
        if (cgpaCmp != 0) return cgpaCmp;
        int semCmp = Integer.compare(this.semester, other.semester);  // Ascending
        if (semCmp != 0) return semCmp;
        return this.name.compareTo(other.name);  // Ascending
    }

    @Override
    public String toString() {
        return String.format("%-8s CGPA: %.2f Sem: %d", name, cgpa, semester);
    }

    public static void main(String[] args) {
        List<Student> students = new ArrayList<>(List.of(
            new Student("Saad", 3.72, 6),
            new Student("Rahim", 3.45, 5),
            new Student("Karim", 3.90, 7),
            new Student("Arif", 3.72, 4),
            new Student("Nila", 3.80, 4),
            new Student("Fatima", 3.72, 6)
        ));

        Collections.sort(students);
        System.out.println("Sorted students:");
        students.forEach(s -> System.out.println("  " + s));
        // Karim 3.90, Nila 3.80, Arif 3.72/4, Fatima 3.72/6, Saad 3.72/6, Rahim 3.45
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;

record Product(String name, double price, int rating, String category, int reviewCount) {

    static final Comparator<Product> BY_PRICE_ASC =
        Comparator.comparingDouble(Product::price);

    static final Comparator<Product> BY_PRICE_DESC =
        BY_PRICE_ASC.reversed();

    static final Comparator<Product> BY_RATING =
        Comparator.comparingInt(Product::rating).reversed()
            .thenComparingInt(Product::reviewCount).reversed();

    static final Comparator<Product> BY_NAME =
        Comparator.comparing(Product::name, String.CASE_INSENSITIVE_ORDER);

    static final Comparator<Product> BY_CATEGORY_THEN_PRICE =
        Comparator.comparing(Product::category)
            .thenComparingDouble(Product::price);

    public static void main(String[] args) {
        List<Product> products = new ArrayList<>(List.of(
            new Product("Laptop", 85000, 4, "Electronics", 120),
            new Product("Mouse", 1500, 5, "Accessories", 300),
            new Product("Keyboard", 3200, 4, "Accessories", 80),
            new Product("Monitor", 25000, 3, "Electronics", 45),
            new Product("Webcam", 4500, 4, "Accessories", 60)
        ));

        products.sort(BY_RATING);
        System.out.println("By rating:");
        products.forEach(p -> System.out.printf("  %-10s R:%d (%d reviews)%n",
            p.name(), p.rating(), p.reviewCount()));
    }
}
```

</details>

---

## Related Notes

- [[Java - Generics - Classes Methods Wildcards]]
- [[Java - Java 8 Lambdas and Functional Interfaces]] (next note)
- [[Java - Java 8 Streams API]]
- [[Java - List - ArrayList LinkedList]]

---

## Resources

- [Oracle Java Tutorials: Object Ordering](https://docs.oracle.com/javase/tutorial/collections/interfaces/order.html) - Official documentation covering Comparable and Comparator with examples.
- [Oracle Java Documentation: Comparable](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Comparable.html) - Complete API reference for the Comparable interface.
- [Oracle Java Documentation: Comparator](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Comparator.html) - Complete API reference including all Java 8+ factory methods.
- [Baeldung: Java Comparable and Comparator](https://www.baeldung.com/java-comparator-comparable) - Comprehensive comparison with practical examples.
- [Baeldung: Java 8 Comparator](https://www.baeldung.com/java-8-comparator-comparing) - Detailed guide to the Java 8 Comparator factory methods.
- [Effective Java by Joshua Bloch - Item 14: Consider Implementing Comparable](https://www.oreilly.com/library/view/effective-java/9780134686097/) - When and how to implement Comparable correctly.
- [Effective Java by Joshua Bloch - Item 13: Override compareTo Judiciously](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive guide to the compareTo contract and common pitfalls.
