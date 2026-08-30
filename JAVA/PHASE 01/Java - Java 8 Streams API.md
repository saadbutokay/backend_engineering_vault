---
title: "Java - Java 8 Streams API"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - streams
  - java8
  - functional-programming
  - collectors
status: "not-started"
---

# Java - Java 8 Streams API

> [!abstract] Overview
> The Streams API, introduced in Java 8, provides a declarative, functional approach to processing sequences of elements. A stream is not a data structure; it is a pipeline of operations that transforms a source of data (a collection, an array, a file, or a generator) into a result. Streams enable you to express complex data processing logic -- filtering, mapping, grouping, reducing, sorting -- in a single, readable chain of method calls. In backend development, the Streams API is the primary tool for transforming database query results into API responses, aggregating metrics, filtering records, building lookup maps, and processing large datasets. It replaces the verbose `for` loop patterns that dominated pre-Java 8 code.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Java 8 Lambdas and Functional Interfaces]]
> - [[Java - Collections Framework Overview]]
> - [[Java - List - ArrayList LinkedList]]
> - [[Java - Map - HashMap TreeMap LinkedHashMap]]
> - [[Java - Generics - Classes Methods Wildcards]]
> - [[Java - Comparable and Comparator]]

---

## Theory

### What is a Stream?

A stream is a **sequence of elements** that supports sequential and parallel aggregate operations. It is not a data structure -- it does not store elements. Instead, it conveys elements from a source through a pipeline of computational operations.

The key characteristics of streams:

1. **Not a data structure**: A stream does not store data. It pulls data from a source (collection, array, I/O channel) on demand.
2. **Functional in nature**: Stream operations produce results without modifying the source. Filtering a stream does not remove elements from the underlying collection.
3. **Lazy evaluation**: Intermediate operations are not executed until a terminal operation is invoked. This allows the stream to optimize the entire pipeline (e.g., short-circuiting a `findFirst()` after the first match).
4. **Possibly unbounded**: Streams can represent infinite sequences (e.g., `Stream.generate(() -> Math.random())`). Operations like `limit()` and `findFirst()` make infinite streams practical.
5. **Consumable**: A stream can be traversed only once. After a terminal operation, the stream is "consumed" and cannot be reused. Attempting to reuse a consumed stream throws `IllegalStateException`.

### The Stream Pipeline

A stream pipeline consists of three parts:

1. **Source**: The origin of the data (a collection, array, file, or generator).
2. **Intermediate operations**: Zero or more operations that transform the stream into another stream. These are lazy and return a new stream. Examples: `filter()`, `map()`, `sorted()`, `distinct()`, `flatMap()`, `limit()`, `skip()`.
3. **Terminal operation**: Exactly one operation that produces a result or a side effect. This triggers the execution of the entire pipeline. Examples: `collect()`, `forEach()`, `reduce()`, `count()`, `findFirst()`, `anyMatch()`.

```text
Source -> Intermediate Op 1 -> Intermediate Op 2 -> ... -> Terminal Op -> Result
  |                                                          |
  Collection, Array, etc.                              List, Map, int, boolean, void
```

### Creating Streams

| Source | Method | Example |
|--------|--------|---------|
| Collection | `.stream()` | `list.stream()` |
| Collection (parallel) | `.parallelStream()` | `list.parallelStream()` |
| Array | `Arrays.stream()` | `Arrays.stream(array)` |
| Individual values | `Stream.of()` | `Stream.of("A", "B", "C")` |
| Empty stream | `Stream.empty()` | `Stream.empty()` |
| Infinite (generate) | `Stream.generate()` | `Stream.generate(Math::random)` |
| Infinite (iterate) | `Stream.iterate()` | `Stream.iterate(0, n -> n + 1)` |
| Range (int) | `IntStream.range()` | `IntStream.range(1, 10)` |
| Range (long) | `LongStream.range()` | `LongStream.rangeClosed(1, 100)` |
| File lines | `Files.lines()` | `Files.lines(Path.of("data.txt"))` |
| String chars | `.chars()` | `"Hello".chars()` |

### Intermediate Operations

Intermediate operations transform a stream into another stream. They are **lazy**: no processing happens until a terminal operation is invoked.

| Operation | Signature | Description |
|-----------|-----------|-------------|
| `filter` | `Stream<T> filter(Predicate<T>)` | Keeps elements matching the predicate |
| `map` | `Stream<R> map(Function<T, R>)` | Transforms each element |
| `flatMap` | `Stream<R> flatMap(Function<T, Stream<R>>)` | Transforms and flattens nested streams |
| `sorted` | `Stream<T> sorted()` / `sorted(Comparator)` | Sorts elements |
| `distinct` | `Stream<T> distinct()` | Removes duplicates (uses `equals()`) |
| `peek` | `Stream<T> peek(Consumer<T>)` | Performs action on each element (debugging) |
| `limit` | `Stream<T> limit(long n)` | Truncates to the first n elements |
| `skip` | `Stream<T> skip(long n)` | Discards the first n elements |
| `takeWhile` | `Stream<T> takeWhile(Predicate<T>)` | Takes elements while predicate is true (Java 9+) |
| `dropWhile` | `Stream<T> dropWhile(Predicate<T>)` | Drops elements while predicate is true (Java 9+) |

### Terminal Operations

Terminal operations produce a result or side effect and trigger the execution of the entire pipeline.

| Operation | Return Type | Description |
|-----------|-------------|-------------|
| `collect` | `R` | Accumulates elements into a collection or other result |
| `forEach` | `void` | Performs an action on each element |
| `reduce` | `Optional<T>` / `T` | Combines elements into a single value |
| `count` | `long` | Counts the elements |
| `min` | `Optional<T>` | Finds the minimum element |
| `max` | `Optional<T>` | Finds the maximum element |
| `anyMatch` | `boolean` | True if any element matches |
| `allMatch` | `boolean` | True if all elements match |
| `noneMatch` | `boolean` | True if no elements match |
| `findFirst` | `Optional<T>` | Returns the first element |
| `findAny` | `Optional<T>` | Returns any element (useful for parallel streams) |
| `toArray` | `T[]` | Collects elements into an array |

### The `collect()` Operation and Collectors

`collect()` is the most versatile terminal operation. It uses a `Collector` to accumulate stream elements into a result container. The `Collectors` utility class provides factory methods for the most common collectors.

| Collector | Result | Example |
|-----------|--------|---------|
| `Collectors.toList()` | `List<T>` | `stream.collect(Collectors.toList())` |
| `Collectors.toSet()` | `Set<T>` | `stream.collect(Collectors.toSet())` |
| `Collectors.toMap(key, value)` | `Map<K, V>` | `stream.collect(Collectors.toMap(User::getId, User::getName))` |
| `Collectors.joining(delimiter)` | `String` | `stream.collect(Collectors.joining(", "))` |
| `Collectors.groupingBy(classifier)` | `Map<K, List<T>>` | `stream.collect(Collectors.groupingBy(Order::getStatus))` |
| `Collectors.partitioningBy(predicate)` | `Map<Boolean, List<T>>` | `stream.collect(Collectors.partitioningBy(Order::isPaid))` |
| `Collectors.counting()` | `long` | `stream.collect(Collectors.counting())` |
| `Collectors.summingInt(mapper)` | `int` | `stream.collect(Collectors.summingInt(Order::getQuantity))` |
| `Collectors.averagingDouble(mapper)` | `double` | `stream.collect(Collectors.averagingDouble(Product::getPrice))` |
| `Collectors.maxBy(comparator)` | `Optional<T>` | `stream.collect(Collectors.maxBy(Comparator.comparing(Order::getTotal)))` |
| `Collectors.mapping(mapper, downstream)` | Depends on downstream | Nested transformation within grouping |
| `Collectors.flatMapping(mapper, downstream)` | Depends on downstream | Nested flat-mapping within grouping (Java 9+) |

**Java 16+ shortcut**: `Stream.toList()` returns an unmodifiable list, replacing `stream.collect(Collectors.toList())` for the common case.

### `map()` vs `flatMap()`

This is the most confusing distinction for beginners.

- **`map()`** transforms each element into exactly one new element. The result is a one-to-one mapping.
- **`flatMap()`** transforms each element into a **stream** of elements and then flattens all the resulting streams into a single stream. The result is a one-to-many mapping.

```java
// map(): one-to-one
List<String> names = List.of("Saad", "Rahim");
names.stream()
    .map(String::toUpperCase)  // Each name -> one uppercase name
    .toList();  // ["SAAD", "RAHIM"]

// flatMap(): one-to-many
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4), List.of(5));
nested.stream()
    .flatMap(List::stream)  // Each inner list -> a stream of integers, then flattened
    .toList();  // [1, 2, 3, 4, 5]
```

### Parallel Streams

Streams can be executed in parallel by calling `.parallelStream()` or `.parallel()`. The stream pipeline is automatically split across multiple threads using the Fork/Join framework.

**When to use parallel streams:**

- Large datasets (typically > 10,000 elements).
- CPU-intensive operations (complex calculations, not I/O).
- Stateless operations (no shared mutable state).
- When the source is efficiently splittable (ArrayList, arrays).

**When NOT to use parallel streams:**

- Small datasets (the thread management overhead exceeds the computation savings).
- I/O-bound operations (database queries, HTTP calls). Parallel I/O can overwhelm the connection pool.
- Operations that depend on ordering (`limit()`, `findFirst()`, `sorted()`).
- Operations with shared mutable state (race conditions).
- When the source is a `LinkedList` (poor splitting performance).

> [!tip] Key Insight
> The Streams API is **declarative**, not imperative. Instead of telling the computer HOW to process the data (loop, check condition, add to list), you tell it WHAT you want (filter by status, map to DTO, collect to list). The stream implementation handles the iteration, optimization, and even parallelization for you. This shift from imperative to declarative is the most important conceptual change in modern Java and the foundation of readable, maintainable backend code.

---

## Syntax and Basic Examples

### Example 1: Basic stream pipeline

```java
import java.util.*;
import java.util.stream.*;

public class StreamBasics {
    public static void main(String[] args) {
        List<String> cities = List.of(
            "Dhaka", "Chittagong", "Sylhet", "Rajshahi",
            "Khulna", "Barishal", "Rangpur", "Comilla"
        );

        // Filter cities with more than 6 characters, sort alphabetically, collect to list
        List<String> longCities = cities.stream()
            .filter(city -> city.length() > 6)      // Intermediate: filter
            .sorted()                                 // Intermediate: sort
            .toList();                                // Terminal: collect (Java 16+)

        System.out.println("Long city names: " + longCities);
        // [Barishal, Chittagong, Comilla, Rajshahi, Rangpur]

        // Count cities starting with 'R'
        long rCount = cities.stream()
            .filter(city -> city.startsWith("R"))
            .count();  // Terminal: count

        System.out.println("Cities starting with R: " + rCount);  // 2

        // Check if any city starts with 'Z'
        boolean hasZ = cities.stream()
            .anyMatch(city -> city.startsWith("Z"));  // Terminal: short-circuit

        System.out.println("Any city starting with Z: " + hasZ);  // false
    }
}
```

### Example 2: map() and flatMap()

```java
import java.util.*;
import java.util.stream.*;

public class MapAndFlatMap {
    public static void main(String[] args) {
        // map(): transform each element
        List<String> names = List.of("saad", "rahim", "karim");
        List<String> upperNames = names.stream()
            .map(String::toUpperCase)
            .toList();
        System.out.println("Upper: " + upperNames);  // [SAAD, RAHIM, KARIM]

        // map(): extract a field from objects
        record Product(String name, double price) {}
        List<Product> products = List.of(
            new Product("Laptop", 85000),
            new Product("Mouse", 1500),
            new Product("Keyboard", 3200)
        );

        List<String> productNames = products.stream()
            .map(Product::name)  // Extract the name field
            .toList();
        System.out.println("Names: " + productNames);  // [Laptop, Mouse, Keyboard]

        List<Double> prices = products.stream()
            .map(Product::price)
            .toList();
        System.out.println("Prices: " + prices);  // [85000.0, 1500.0, 3200.0]

        // flatMap(): flatten nested structures
        record Order(String id, List<String> items) {}
        List<Order> orders = List.of(
            new Order("ORD-1", List.of("Laptop", "Mouse")),
            new Order("ORD-2", List.of("Keyboard")),
            new Order("ORD-3", List.of("Monitor", "Webcam", "Mouse"))
        );

        // Get all unique items across all orders
        List<String> allItems = orders.stream()
            .flatMap(order -> order.items().stream())  // Flatten lists into a single stream
            .distinct()
            .sorted()
            .toList();
        System.out.println("All items: " + allItems);
        // [Keyboard, Laptop, Monitor, Mouse, Webcam]

        // flatMap(): split strings into words
        List<String> sentences = List.of("Java is great", "Spring Boot is powerful");
        List<String> words = sentences.stream()
            .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
            .distinct()
            .toList();
        System.out.println("Words: " + words);
        // [Java, is, great, Spring, Boot, powerful]
    }
}
```

### Example 3: Collectors in depth

```java
import java.util.*;
import java.util.stream.*;

public class CollectorsDemo {
    record Employee(String name, String department, double salary, int yearsOfService) {}

    public static void main(String[] args) {
        List<Employee> employees = List.of(
            new Employee("Saad", "Engineering", 85000, 3),
            new Employee("Rahim", "Engineering", 95000, 5),
            new Employee("Karim", "Marketing", 70000, 3),
            new Employee("Nila", "Marketing", 90000, 7),
            new Employee("Arif", "Engineering", 90000, 4),
            new Employee("Fatima", "HR", 60000, 2),
            new Employee("Hasan", "HR", 65000, 4)
        );

        // 1. toList() and toSet()
        List<String> names = employees.stream()
            .map(Employee::name)
            .toList();
        System.out.println("Names: " + names);

        Set<String> departments = employees.stream()
            .map(Employee::department)
            .collect(Collectors.toSet());
        System.out.println("Departments: " + departments);

        // 2. toMap(): name -> salary
        Map<String, Double> salaryMap = employees.stream()
            .collect(Collectors.toMap(Employee::name, Employee::salary));
        System.out.println("Salary map: " + salaryMap);

        // 3. toMap() with merge function (handles duplicate keys)
        Map<String, Double> deptAvgSalary = employees.stream()
            .collect(Collectors.toMap(
                Employee::department,
                Employee::salary,
                (existing, replacement) -> (existing + replacement) / 2
                // This is a simplified average; for real averages, use groupingBy + averagingDouble
            ));
        System.out.println("Dept salary (simplified): " + deptAvgSalary);

        // 4. groupingBy(): group employees by department
        Map<String, List<Employee>> byDepartment = employees.stream()
            .collect(Collectors.groupingBy(Employee::department));
        System.out.println("\nBy department:");
        byDepartment.forEach((dept, emps) ->
            System.out.println("  " + dept + ": " + emps.size() + " employees")
        );

        // 5. groupingBy() with downstream collector: count per department
        Map<String, Long> countByDept = employees.stream()
            .collect(Collectors.groupingBy(Employee::department, Collectors.counting()));
        System.out.println("Count by dept: " + countByDept);

        // 6. groupingBy() with averagingDouble: average salary per department
        Map<String, Double> avgSalaryByDept = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::department,
                Collectors.averagingDouble(Employee::salary)
            ));
        System.out.println("Avg salary by dept: " + avgSalaryByDept);

        // 7. partitioningBy(): split into two groups based on a predicate
        Map<Boolean, List<Employee>> bySalary = employees.stream()
            .collect(Collectors.partitioningBy(e -> e.salary() >= 80000));
        System.out.println("\nHigh earners (>=80k): " + bySalary.get(true).size());
        System.out.println("Others: " + bySalary.get(false).size());

        // 8. joining(): concatenate strings
        String allNames = employees.stream()
            .map(Employee::name)
            .collect(Collectors.joining(", ", "[", "]"));
        System.out.println("All names: " + allNames);
        // [Saad, Rahim, Karim, Nila, Arif, Fatima, Hasan]

        // 9. Nested groupingBy(): department -> years of service -> list of names
        Map<String, Map<Integer, List<String>>> nested = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::department,
                Collectors.groupingBy(
                    Employee::yearsOfService,
                    Collectors.mapping(Employee::name, Collectors.toList())
                )
            ));
        System.out.println("\nNested grouping: " + nested);
    }
}
```

### Example 4: reduce() and numeric streams

```java
import java.util.*;
import java.util.stream.*;

public class ReduceDemo {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // reduce(): sum all numbers
        int sum = numbers.stream()
            .reduce(0, Integer::sum);  // identity=0, accumulator=sum
        System.out.println("Sum: " + sum);  // 55

        // reduce(): multiply all numbers
        int product = numbers.stream()
            .reduce(1, (a, b) -> a * b);
        System.out.println("Product: " + product);  // 3628800

        // reduce(): find maximum (without identity, returns Optional)
        Optional<Integer> max = numbers.stream()
            .reduce(Integer::max);
        System.out.println("Max: " + max.orElse(0));  // 10

        // reduce(): concatenate strings
        List<String> words = List.of("Java", "Spring", "Boot");
        String sentence = words.stream()
            .reduce("", (a, b) -> a.isEmpty() ? b : a + " " + b);
        System.out.println("Sentence: " + sentence);  // Java Spring Boot

        // Numeric streams: IntStream, LongStream, DoubleStream
        // These avoid autoboxing overhead for primitive types
        int intSum = IntStream.rangeClosed(1, 100).sum();
        System.out.println("Sum 1-100: " + intSum);  // 5050

        double avg = IntStream.rangeClosed(1, 100).average().orElse(0);
        System.out.println("Average 1-100: " + avg);  // 50.5

        IntSummaryStatistics stats = IntStream.of(10, 20, 30, 40, 50).summaryStatistics();
        System.out.println("Count: " + stats.getCount());      // 5
        System.out.println("Sum: " + stats.getSum());          // 150
        System.out.println("Min: " + stats.getMin());          // 10
        System.out.println("Max: " + stats.getMax());          // 50
        System.out.println("Average: " + stats.getAverage());  // 30.0
    }
}
```

### Example 5: Debugging with peek() and understanding laziness

```java
import java.util.*;
import java.util.stream.*;

public class LazinessDemo {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // peek() lets you observe the stream without modifying it.
        // Use it for debugging, NOT for production logic.
        System.out.println("--- Stream with peek ---");
        List<Integer> result = numbers.stream()
            .peek(n -> System.out.println("Source: " + n))
            .filter(n -> n % 2 == 0)
            .peek(n -> System.out.println("After filter: " + n))
            .map(n -> n * 10)
            .peek(n -> System.out.println("After map: " + n))
            .limit(3)
            .peek(n -> System.out.println("After limit: " + n))
            .toList();

        System.out.println("Result: " + result);  // [20, 40, 60]

        // Laziness demonstration: no output until terminal operation
        System.out.println("\n--- Laziness ---");
        Stream<Integer> lazyStream = numbers.stream()
            .filter(n -> {
                System.out.println("Filtering: " + n);  // This does NOT print yet
                return n > 5;
            });
        System.out.println("Stream created but not executed yet.");

        // NOW the pipeline executes
        long count = lazyStream.count();
        System.out.println("Count: " + count);  // 5
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> The Streams API is the backbone of data transformation in Spring Boot backends. Here are three realistic scenarios.

### Scenario 1: Entity-to-DTO transformation pipeline

The most common use of streams in backend code is transforming database entities into API response DTOs.

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public OrderListResponse getUserOrders(Long userId) {
        List<Order> orders = orderRepository.findByUserId(userId);

        // Single stream pipeline: filter, transform, sort, collect
        List<OrderSummaryDTO> summaries = orders.stream()
            .filter(order -> order.getStatus() != OrderStatus.CANCELLED)
            .sorted(Comparator.comparing(Order::getCreatedAt).reversed())
            .map(order -> new OrderSummaryDTO(
                order.getId(),
                order.getOrderNumber(),
                order.getTotalAmount(),
                order.getStatus().name(),
                order.getItems().size(),
                order.getCreatedAt()
            ))
            .toList();

        // Aggregate statistics using a separate stream
        BigDecimal totalSpent = orders.stream()
            .filter(order -> order.getStatus() == OrderStatus.DELIVERED)
            .map(Order::getTotalAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);

        long activeOrders = orders.stream()
            .filter(order -> order.getStatus() == OrderStatus.PENDING
                || order.getStatus() == OrderStatus.CONFIRMED)
            .count();

        return new OrderListResponse(summaries, totalSpent, activeOrders);
    }

    public Map<OrderStatus, OrderStatsDTO> getOrderStats(Long userId) {
        // Group orders by status and compute statistics for each group
        return orderRepository.findByUserId(userId).stream()
            .collect(Collectors.groupingBy(
                Order::getStatus,
                Collectors.collectingAndThen(
                    Collectors.toList(),
                    group -> new OrderStatsDTO(
                        group.size(),
                        group.stream()
                            .map(Order::getTotalAmount)
                            .reduce(BigDecimal.ZERO, BigDecimal::add),
                        group.stream()
                            .map(Order::getTotalAmount)
                            .max(Comparator.naturalOrder())
                            .orElse(BigDecimal.ZERO)
                    )
                )
            ));
    }
}

record OrderSummaryDTO(
    Long id, String orderNumber, BigDecimal total,
    String status, int itemCount, LocalDateTime createdAt
) {}

record OrderStatsDTO(int count, BigDecimal totalAmount, BigDecimal maxAmount) {}

record OrderListResponse(
    List<OrderSummaryDTO> orders, BigDecimal totalSpent, long activeOrders
) {}
```

**What to notice:**

- The main transformation pipeline is a single, readable chain: `stream().filter().sorted().map().toList()`. This replaces what would be a 15-line `for` loop with `if` statements and a temporary list.
- Aggregate statistics (`totalSpent`, `activeOrders`) use separate stream pipelines over the same data. This is acceptable for small-to-medium datasets. For large datasets, a single `collect()` with a custom collector would be more efficient.
- The `getOrderStats()` method uses `Collectors.groupingBy()` with `Collectors.collectingAndThen()` to compute per-group statistics. This is the most powerful collector pattern for backend reporting and analytics.

### Scenario 2: Data validation and deduplication

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

@Service
public class ImportService {

    public ImportResult importProducts(List<ProductImportDTO> rawImports) {
        // Step 1: Validate and filter invalid records
        List<ProductImportDTO> validImports = rawImports.stream()
            .filter(dto -> dto.name() != null && !dto.name().isBlank())
            .filter(dto -> dto.price() != null && dto.price().compareTo(BigDecimal.ZERO) > 0)
            .filter(dto -> dto.sku() != null && dto.sku().matches("^[A-Z0-9]{8,12}$"))
            .toList();

        // Step 2: Detect duplicates by SKU
        Set<String> seenSkus = new HashSet<>();
        List<ProductImportDTO> uniqueImports = validImports.stream()
            .filter(dto -> seenSkus.add(dto.sku()))  // add() returns false for duplicates
            .toList();

        int duplicateCount = validImports.size() - uniqueImports.size();

        // Step 3: Transform DTOs to entities
        List<Product> products = uniqueImports.stream()
            .map(dto -> new Product(
                dto.sku(),
                dto.name().strip(),
                dto.price().setScale(2, RoundingMode.HALF_UP),
                dto.category() != null ? dto.category().toUpperCase() : "UNCATEGORIZED"
            ))
            .toList();

        // Step 4: Save in batch
        List<Product> savedProducts = productRepository.saveAll(products);

        return new ImportResult(
            savedProducts.size(),
            rawImports.size() - validImports.size(),  // Invalid count
            duplicateCount
        );
    }
}

record ImportResult(int imported, int invalid, int duplicates) {}
```

**What to notice:**

- The validation pipeline uses chained `filter()` calls, each checking one validation rule. This is more readable than a single complex `if` condition and makes it easy to add or remove rules.
- The deduplication trick `seenSkus.add(dto.sku())` exploits the fact that `Set.add()` returns `false` when the element already exists. This is a common pattern for order-preserving deduplication in streams.
- The transformation step (`map()`) normalizes the data: stripping whitespace, rounding prices, and uppercasing categories. This ensures that all data entering the database is clean and consistent.

### Scenario 3: Building lookup maps and indexes

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

@Service
public class CatalogService {

    private final ProductRepository productRepository;

    public CatalogService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    // Build a product lookup map for fast access during order processing
    public Map<Long, Product> buildProductLookup() {
        return productRepository.findAll().stream()
            .collect(Collectors.toMap(Product::getId, Function.identity()));
            // Function.identity() is equivalent to p -> p
    }

    // Build a category index for faceted search
    public Map<String, List<ProductSummary>> getCategoryIndex() {
        return productRepository.findAll().stream()
            .filter(Product::isActive)
            .collect(Collectors.groupingBy(
                Product::getCategory,
                Collectors.mapping(
                    product -> new ProductSummary(
                        product.getId(),
                        product.getName(),
                        product.getPrice()
                    ),
                    Collectors.toList()
                )
            ));
    }

    // Build a price range index for filtering
    public Map<String, Long> getPriceRangeCounts() {
        return productRepository.findAll().stream()
            .filter(Product::isActive)
            .collect(Collectors.groupingBy(
                product -> {
                    double price = product.getPrice().doubleValue();
                    if (price < 1000) return "Under 1,000";
                    if (price < 5000) return "1,000 - 5,000";
                    if (price < 20000) return "5,000 - 20,000";
                    if (price < 50000) return "20,000 - 50,000";
                    return "50,000+";
                },
                Collectors.counting()
            ));
    }

    // Find products matching multiple filter criteria
    public List<Product> searchProducts(String keyword, String category,
                                         Double minPrice, Double maxPrice) {
        return productRepository.findAll().stream()
            .filter(Product::isActive)
            .filter(p -> keyword == null ||
                p.getName().toLowerCase().contains(keyword.toLowerCase()))
            .filter(p -> category == null ||
                p.getCategory().equalsIgnoreCase(category))
            .filter(p -> minPrice == null ||
                p.getPrice().doubleValue() >= minPrice)
            .filter(p -> maxPrice == null ||
                p.getPrice().doubleValue() <= maxPrice)
            .sorted(Comparator.comparing(Product::getName))
            .toList();
    }
}

record ProductSummary(Long id, String name, BigDecimal price) {}
```

**What to notice:**

- `Collectors.toMap(Product::getId, Function.identity())` builds a lookup map in a single line. `Function.identity()` is a built-in function that returns its argument unchanged (`p -> p`). This is the standard pattern for building ID-to-entity maps.
- `Collectors.groupingBy()` with `Collectors.mapping()` transforms the grouped elements before collecting them. This avoids creating intermediate lists of full entities when you only need a summary.
- The `searchProducts()` method uses conditional filters: if a filter parameter is null, the filter passes all elements. This is a common pattern for building dynamic search queries in memory. In production, these filters would be pushed to the database using Spring Data's `Specification` or `QueryDSL`.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using streams for simple operations

**Wrong:**

```java
// Over-engineering a simple lookup
Optional<User> user = users.stream()
    .filter(u -> u.getId().equals(targetId))
    .findFirst();
```

**Right:**

```java
// For a single lookup, use a Map or a direct repository call
User user = userMap.get(targetId);  // O(1) instead of O(n)
// Or: userRepository.findById(targetId);
```

**Why it is wrong:** Streams are not a replacement for all loops. For simple lookups, a `Map.get()` is O(1) while a stream filter is O(n). For simple iterations with side effects, a `for` loop is more readable and faster. Use streams for data transformation pipelines (filter + map + collect), not for single lookups or simple iterations.

### Mistake 2: Modifying the source collection during a stream operation

**Wrong:**

```java
List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Charlie"));

names.stream()
    .filter(name -> name.startsWith("A"))
    .forEach(name -> names.remove(name));  // ConcurrentModificationException!
```

**Right:**

```java
// Use removeIf() for in-place removal
names.removeIf(name -> name.startsWith("A"));

// Or create a new filtered list
List<String> filtered = names.stream()
    .filter(name -> !name.startsWith("A"))
    .toList();
```

**Why it is wrong:** Streams operate on a snapshot of the source. Modifying the source collection while a stream is processing it causes `ConcurrentModificationException`. Streams are designed to be non-mutating: they produce new results without changing the source.

### Mistake 3: Ignoring Optional from terminal operations

**Wrong:**

```java
Order mostExpensive = orders.stream()
    .max(Comparator.comparing(Order::getTotalAmount))
    .get();  // NoSuchElementException if the stream is empty!
```

**Right:**

```java
Order mostExpensive = orders.stream()
    .max(Comparator.comparing(Order::getTotalAmount))
    .orElseThrow(() -> new ResourceNotFoundException("No orders found"));

// Or provide a default
Order mostExpensive = orders.stream()
    .max(Comparator.comparing(Order::getTotalAmount))
    .orElse(null);  // Handle null downstream
```

**Why it is wrong:** Terminal operations like `findFirst()`, `findAny()`, `min()`, `max()`, and `reduce()` (without identity) return `Optional<T>` because the stream might be empty. Calling `.get()` on an empty `Optional` throws `NoSuchElementException`. Always use `.orElse()`, `.orElseThrow()`, or `.ifPresent()` to handle the empty case explicitly.

### Mistake 4: Using parallel streams without understanding the trade-offs

**Wrong:**

```java
// Parallel stream for a small list with I/O operations
List<OrderResponse> responses = orders.parallelStream()
    .map(order -> {
        PaymentStatus status = paymentClient.checkStatus(order.getId());  // I/O!
        return new OrderResponse(order, status);
    })
    .toList();
// This creates 100 concurrent HTTP requests, overwhelming the payment API
// and the connection pool. It is SLOWER than sequential processing.
```

**Right:**

```java
// Sequential stream for I/O operations
List<OrderResponse> responses = orders.stream()
    .map(order -> {
        PaymentStatus status = paymentClient.checkStatus(order.getId());
        return new OrderResponse(order, status);
    })
    .toList();

// Or use a bounded thread pool with CompletableFuture for controlled parallelism
List<CompletableFuture<OrderResponse>> futures = orders.stream()
    .map(order -> CompletableFuture.supplyAsync(
        () -> new OrderResponse(order, paymentClient.checkStatus(order.getId())),
        executorService  // Bounded thread pool with 10 threads
    ))
    .toList();
```

**Why it is wrong:** Parallel streams use the common ForkJoinPool, which has a limited number of threads (typically equal to the number of CPU cores). I/O operations block these threads, starving other parallel streams in the application. For I/O-bound work, use a dedicated thread pool with `CompletableFuture` or a reactive framework (WebFlux). Reserve parallel streams for CPU-bound operations on large in-memory datasets.

---

## Key Takeaways

> [!tip] Remember these points
> 1. A **stream** is a declarative pipeline of operations on a sequence of elements. It consists of a source, zero or more intermediate operations (lazy), and exactly one terminal operation (triggers execution). Streams do not modify the source and can be traversed only once.
> 2. **Intermediate operations** (`filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`, `skip`) transform the stream and return a new stream. **Terminal operations** (`collect`, `forEach`, `reduce`, `count`, `findFirst`, `anyMatch`) produce a result and trigger the pipeline.
> 3. **`collect()`** is the most versatile terminal operation. Use `Collectors.toList()`, `toSet()`, `toMap()`, `groupingBy()`, `partitioningBy()`, and `joining()` for the most common accumulation patterns. `groupingBy()` with downstream collectors is the standard tool for backend reporting and analytics.
> 4. **`map()`** transforms each element one-to-one. **`flatMap()`** transforms each element into a stream and flattens the results. Use `flatMap()` when each input element produces multiple output elements (e.g., extracting items from a list of orders).
> 5. Use **sequential streams** by default. Use **parallel streams** only for CPU-bound operations on large datasets (> 10,000 elements) with stateless operations. Never use parallel streams for I/O-bound operations. Always handle `Optional` results from terminal operations explicitly.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Stream Basics (Easy)

Given a list of integers from 1 to 50, use streams to:

1. Find all even numbers greater than 20.
2. Calculate the sum of all odd numbers.
3. Find the maximum number divisible by 7.
4. Check if all numbers are positive.
5. Get the first 5 numbers that are perfect squares.

Print all results.

> **Hint:** Use `IntStream.rangeClosed(1, 50)` as the source. For perfect squares, check if `Math.sqrt(n) % 1 == 0`.

### Exercise 2: Collectors Practice (Medium)

Given a list of `Student` records (name, department, cgpa, semester), use streams and collectors to:

1. Group students by department and count them.
2. Find the average CGPA per department.
3. Find the top student (highest CGPA) in each department.
4. Partition students into "honors" (CGPA >= 3.5) and "regular" groups.
5. Create a map of student name to CGPA.
6. Join all student names into a single comma-separated string.

> **Hint:** Use `Collectors.groupingBy()` with downstream collectors like `Collectors.counting()`, `Collectors.averagingDouble()`, and `Collectors.maxBy()`. For the top student per department, use `Collectors.collectingAndThen()` to unwrap the `Optional`.

### Exercise 3: FlatMap and Complex Pipelines (Medium)

Given a list of `Order` records, each containing a list of `OrderItem` records:

1. Extract all unique product names across all orders.
2. Calculate the total revenue (sum of quantity * unitPrice for all items in all orders).
3. Find the most frequently ordered product.
4. Group orders by month and calculate monthly revenue.

> **Hint:** Use `flatMap()` to flatten the nested `OrderItem` lists. For the most frequent product, use `Collectors.groupingBy()` with `Collectors.counting()` and then find the entry with the maximum count. For monthly grouping, use `order.createdAt().getMonth()` as the grouping key.

### Exercise 4: Stream-Based Data Processing Pipeline (Hard, Optional)

Build a complete data processing pipeline that reads a list of raw CSV-like strings, parses them, validates them, transforms them, and produces a summary report:

Input: `List<String>` of CSV lines like `"Saad,CSE,3.72,6"` and some invalid lines like `"Rahim,EEE,-1.0,5"` or `"broken,line"`.

Pipeline steps:

1. Parse each line into a `Student` record (split by comma, parse fields).
2. Filter out lines that do not have exactly 4 fields.
3. Filter out students with invalid CGPA (< 0 or > 4.0).
4. Normalize names to title case.
5. Group by department and calculate statistics (count, average CGPA, max CGPA).
6. Produce a formatted report string.

Use a single stream pipeline where possible. Handle parsing errors gracefully using `flatMap()` with `Optional` or try-catch inside a helper method.

> **Hint:** Create a `parseStudent(String line)` method that returns `Optional<Student>`. Use `flatMap(Optional::stream)` to filter out parse failures. This is a common pattern for error-tolerant stream processing.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.stream.*;

public class Main {
    public static void main(String[] args) {
        // 1. Even numbers > 20
        var evens = IntStream.rangeClosed(1, 50)
            .filter(n -> n > 20 && n % 2 == 0)
            .boxed().toList();
        System.out.println("Even > 20: " + evens);

        // 2. Sum of odd numbers
        int oddSum = IntStream.rangeClosed(1, 50)
            .filter(n -> n % 2 != 0).sum();
        System.out.println("Odd sum: " + oddSum);

        // 3. Max divisible by 7
        int maxDiv7 = IntStream.rangeClosed(1, 50)
            .filter(n -> n % 7 == 0).max().orElse(0);
        System.out.println("Max div 7: " + maxDiv7);

        // 4. All positive
        boolean allPos = IntStream.rangeClosed(1, 50).allMatch(n -> n > 0);
        System.out.println("All positive: " + allPos);

        // 5. First 5 perfect squares
        var squares = IntStream.rangeClosed(1, 50)
            .filter(n -> Math.sqrt(n) % 1 == 0)
            .limit(5).boxed().toList();
        System.out.println("Perfect squares: " + squares);
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;
import java.util.stream.*;

record Student(String name, String dept, double cgpa, int sem) {}

public class Main {
    public static void main(String[] args) {
        List<Student> students = List.of(
            new Student("Saad", "CSE", 3.72, 6),
            new Student("Rahim", "CSE", 3.45, 5),
            new Student("Karim", "EEE", 3.90, 7),
            new Student("Nila", "EEE", 3.80, 4),
            new Student("Arif", "CSE", 3.55, 6),
            new Student("Fatima", "BBA", 3.60, 3)
        );

        // 1. Count by department
        Map<String, Long> countByDept = students.stream()
            .collect(Collectors.groupingBy(Student::dept, Collectors.counting()));
        System.out.println("Count: " + countByDept);

        // 2. Average CGPA by department
        Map<String, Double> avgByDept = students.stream()
            .collect(Collectors.groupingBy(Student::dept,
                Collectors.averagingDouble(Student::cgpa)));
        System.out.println("Avg CGPA: " + avgByDept);

        // 3. Top student per department
        Map<String, Student> topByDept = students.stream()
            .collect(Collectors.toMap(
                Student::dept, s -> s,
                (s1, s2) -> s1.cgpa() > s2.cgpa() ? s1 : s2
            ));
        System.out.println("Top per dept: " + topByDept);

        // 4. Partition by honors
        Map<Boolean, List<Student>> partitioned = students.stream()
            .collect(Collectors.partitioningBy(s -> s.cgpa() >= 3.5));
        System.out.println("Honors: " + partitioned.get(true).size());
        System.out.println("Regular: " + partitioned.get(false).size());

        // 5. Name -> CGPA map
        Map<String, Double> nameToCgpa = students.stream()
            .collect(Collectors.toMap(Student::name, Student::cgpa));
        System.out.println("Name-CGPA: " + nameToCgpa);

        // 6. Joined names
        String joined = students.stream()
            .map(Student::name)
            .collect(Collectors.joining(", "));
        System.out.println("Names: " + joined);
    }
}
```

</details>

---

## Related Notes

- [[Java - Java 8 Lambdas and Functional Interfaces]]
- [[Java - Optional Class]] (next note)
- [[Java - Date and Time API - LocalDate LocalDateTime]]
- [[Java - Collections Framework Overview]]

---

## Resources

- [Oracle Java Tutorials: Streams](https://docs.oracle.com/javase/tutorial/collections/streams/) - Official documentation covering the Streams API with detailed examples.
- [Oracle Java Documentation: java.util.stream](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html) - Complete API reference for all stream classes and interfaces.
- [Oracle Java Documentation: Collectors](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html) - Complete reference for all built-in collectors.
- [Baeldung: Java 8 Streams Guide](https://www.baeldung.com/java-8-streams) - Comprehensive guide covering all stream operations with examples.
- [Baeldung: Java 8 Collectors](https://www.baeldung.com/java-8-collectors) - Detailed guide to all collector types including groupingBy, partitioningBy, and custom collectors.
- [Baeldung: Java 8 Stream flatMap](https://www.baeldung.com/java-stream-flatmap) - In-depth explanation of flatMap with practical examples.
- [Effective Java by Joshua Bloch - Item 45: Use Streams Judiciously](https://www.oreilly.com/library/view/effective-java/9780134686097/) - When to use streams and when to stick with loops. Essential reading.
- [Modern Java in Action by Raoul-Gabriel Urma - Chapters 4-6](https://www.manning.com/books/modern-java-in-action) - The best book-length treatment of streams, lambdas, and collectors.
