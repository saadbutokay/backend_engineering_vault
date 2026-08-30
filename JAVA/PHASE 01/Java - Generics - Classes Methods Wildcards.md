---
title: "Java - Generics - Classes Methods Wildcards"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - generics
  - type-safety
  - wildcards
status: "not-started"
---

# Java - Generics - Classes Methods Wildcards

> [!abstract] Overview
> Generics allow you to parameterize types, enabling classes, interfaces, and methods to operate on objects of various types while providing compile-time type safety. Before generics (pre-Java 5), collections stored `Object` references and required explicit casting, which was error-prone and verbose. Generics eliminate unsafe casts, catch type errors at compile time instead of runtime, and make code self-documenting. In backend development, generics are the foundation of Spring Data repositories (`JpaRepository<Order, Long>`), generic DTO wrappers (`ApiResponse<T>`), type-safe query builders, and the entire Java Collections Framework.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Inheritance - Single Multilevel Hierarchical]]
> - [[Java - Abstraction - Abstract Classes and Interfaces]]
> - [[Java - Collections Framework Overview]]
> - [[Java - List - ArrayList LinkedList]]

---

## Theory

### What Are Generics?

Generics are a mechanism for **parameterizing types**. Instead of hardcoding a specific type into a class or method, you use a **type parameter** (a placeholder) that is replaced with a concrete type when the class is instantiated or the method is called.

Before generics (Java 1.0-1.4), collections stored `Object` references:

```java
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

// Pre-generics (Java 1.4 and earlier):
List list = new ArrayList();
list.add("Hello");
list.add(42);  // No compile-time error! Mixed types allowed.
list.add(new Date());

String s = (String) list.get(0);  // Explicit cast required
String s2 = (String) list.get(1);  // ClassCastException at runtime!
// The compiler could not catch this error. It only crashed at runtime.
```

With generics (Java 5+):

```java
import java.util.ArrayList;
import java.util.List;

// Modern Java:
List<String> list = new ArrayList<>();
list.add("Hello");
list.add(42);  // COMPILATION ERROR! Cannot add Integer to List<String>.

String s = list.get(0);  // No cast needed. The compiler knows it is a String.
```

Generics provide three benefits:

1. **Compile-time type safety**: The compiler catches type mismatches before the program runs. A `List<String>` cannot accidentally contain an `Integer`.
2. **Elimination of casts**: You no longer need to cast when retrieving elements from a collection. The compiler inserts the cast automatically and safely.
3. **Self-documenting code**: `List<Order>` is more readable than `List` (which could contain anything). The type parameter communicates intent.

### Generic Classes

A generic class declares one or more type parameters in angle brackets after the class name. These type parameters act as placeholders that are replaced with concrete types when the class is instantiated.

```java
// T is a type parameter (convention: single uppercase letter)
public class Box<T> {
    private T content;

    public void put(T item) {
        this.content = item;
    }

    public T get() {
        return content;
    }
}

// Usage: T is replaced with String
Box<String> stringBox = new Box<>();
stringBox.put("Hello");
String s = stringBox.get();  // No cast needed

// Usage: T is replaced with Integer
Box<Integer> intBox = new Box<>();
intBox.put(42);
int n = intBox.get();  // No cast needed

// stringBox.put(42);  // COMPILATION ERROR! Box<String> only accepts Strings
```

**Type parameter naming conventions:**

| Letter | Meaning | Example |
|--------|---------|---------|
| `T` | Type | `Box<T>`, `Repository<T>` |
| `E` | Element | `List<E>`, `Set<E>` |
| `K` | Key | `Map<K, V>` |
| `V` | Value | `Map<K, V>` |
| `N` | Number | `MathUtils<N extends Number>` |
| `R` | Return type | `Function<T, R>` |
| `S`, `U` | Additional types | `BiFunction<T, U, R>` |

A class can have multiple type parameters:

```java
public class Pair<K, V> {
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}

Pair<String, Integer> pair = new Pair<>("age", 22);
String key = pair.getKey();      // String
Integer value = pair.getValue(); // Integer
```

### Generic Methods

A generic method declares its own type parameters, independent of the class's type parameters. The type parameters are declared before the return type.

```java
import java.util.*;

public class ArrayUtils {

    // Generic method: T is determined by the arguments at the call site
    public static <T> void printArray(T[] array) {
        for (T element : array) {
            System.out.print(element + " ");
        }
        System.out.println();
    }

    // Generic method with two type parameters
    public static <K, V> void printMap(Map<K, V> map) {
        for (Map.Entry<K, V> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
    }

    // Generic method that returns a generic type
    public static <T> List<T> toList(T... elements) {
        return new ArrayList<>(Arrays.asList(elements));
    }
}

// Usage: the compiler infers T from the argument types
ArrayUtils.printArray(new String[]{"A", "B", "C"});  // T = String
ArrayUtils.printArray(new Integer[]{1, 2, 3});        // T = Integer

List<String> list = ArrayUtils.toList("X", "Y", "Z");  // T = String (inferred)
```

**Type inference**: In most cases, the compiler can infer the type parameter from the method arguments or the assignment context. You rarely need to specify the type explicitly. However, you can specify it if needed:

```java
// Explicit type specification (rarely needed)
List<String> list = ArrayUtils.<String>toList("A", "B");
```

### Bounded Type Parameters

Bounded type parameters restrict the types that can be used as type arguments. They are declared using the `extends` keyword (which, in the context of generics, means "extends or implements").

**Upper bound**: The type argument must be a subclass of (or implement) the specified type.

```java
// T must be a subclass of Number (Integer, Double, Long, etc.)
public class NumericBox<T extends Number> {
    private T value;

    public NumericBox(T value) {
        this.value = value;
    }

    public double toDouble() {
        return value.doubleValue();  // Safe: Number has doubleValue()
    }
}

NumericBox<Integer> intBox = new NumericBox<>(42);      // OK
NumericBox<Double> doubleBox = new NumericBox<>(3.14);  // OK
// NumericBox<String> strBox = new NumericBox<>("hello");  // COMPILATION ERROR!
// String does not extend Number.
```

**Multiple bounds**: A type parameter can have multiple bounds (one class and multiple interfaces), separated by `&`.

```java
import java.io.Serializable;
import java.util.List;

// T must extend Comparable AND implement Serializable
public <T extends Comparable<T> & Serializable> T findMax(List<T> list) {
    T max = list.get(0);
    for (T item : list) {
        if (item.compareTo(max) > 0) {
            max = item;
        }
    }
    return max;
}
```

### Wildcards

Wildcards (`?`) represent an **unknown type**. They are used in method parameters and variable declarations when you want to accept a range of generic types rather than a specific one.

**Three types of wildcards:**

**1. Unbounded wildcard (`?`)**: Accepts any type. Equivalent to `? extends Object`.

```java
import java.util.List;

public static void printList(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}

printList(List.of("A", "B"));  // OK
printList(List.of(1, 2, 3));   // OK
```

**2. Upper-bounded wildcard (`? extends T`)**: Accepts `T` or any subclass of `T`. Use when you only **read** from the collection.

```java
import java.util.List;

public static double sumOfList(List<? extends Number> list) {
    double sum = 0;
    for (Number n : list) {  // Safe: every element is at least a Number
        sum += n.doubleValue();
    }
    return sum;
}

sumOfList(List.of(1, 2, 3));          // OK: List<Integer>
sumOfList(List.of(1.5, 2.5, 3.5));    // OK: List<Double>
// sumOfList(List.of("A", "B"));      // ERROR: String is not a Number
```

**3. Lower-bounded wildcard (`? super T`)**: Accepts `T` or any superclass of `T`. Use when you only **write** to the collection.

```java
import java.util.ArrayList;
import java.util.List;

public static void addIntegers(List<? super Integer> list) {
    list.add(1);  // Safe: the list accepts Integer or any supertype
    list.add(2);
    list.add(3);
}

List<Number> numbers = new ArrayList<>();
addIntegers(numbers);  // OK: Number is a supertype of Integer

List<Object> objects = new ArrayList<>();
addIntegers(objects);  // OK: Object is a supertype of Integer
```

### The PECS Principle

**PECS** stands for **Producer Extends, Consumer Super**. It is the golden rule for choosing wildcards:

- If a parameterized type **produces** values (you read from it), use `? extends T`.
- If a parameterized type **consumes** values (you write to it), use `? super T`.
- If you both read and write, do not use a wildcard. Use an exact type parameter.

```java
import java.util.ArrayList;
import java.util.List;

// Producer: reading from source (extends)
public static <T> void copy(List<? extends T> source, List<? super T> destination) {
    for (T item : source) {       // Reading from source: ? extends T
        destination.add(item);    // Writing to destination: ? super T
    }
}

List<Integer> ints = List.of(1, 2, 3);
List<Number> nums = new ArrayList<>();
copy(ints, nums);  // OK: Integer extends Number, Number super Integer
```

### Type Erasure

Java generics are implemented through **type erasure**. This means that generic type information exists only at compile time. The compiler removes (erases) all type parameters and replaces them with their bounds (or `Object` if unbounded) during compilation. The resulting bytecode contains no generic type information.

**What the compiler does:**

```java
import java.util.ArrayList;
import java.util.List;

// What you write:
List<String> list = new ArrayList<>();
list.add("Hello");
String s = list.get(0);

// What the compiler generates (conceptually):
List list = new ArrayList();
list.add("Hello");
String s = (String) list.get(0);  // Compiler inserts the cast automatically
```

**Consequences of type erasure:**

1. **You cannot create instances of type parameters**: `new T()` is illegal because `T` does not exist at runtime.
2. **You cannot create arrays of type parameters**: `new T[10]` is illegal.
3. **You cannot use `instanceof` with parameterized types**: `list instanceof List<String>` is illegal. You can only check `list instanceof List`.
4. **You cannot use primitives as type arguments**: `List<int>` is illegal. Use `List<Integer>` instead (autoboxing handles the conversion).
5. **Generic types with different type arguments have the same runtime class**: `List<String>.class == List<Integer>.class` is true. Both are just `List.class` at runtime.

> [!tip] Key Insight
> Type erasure means that generics are a **compile-time** feature. They provide type safety during development but have zero runtime overhead. The JVM does not know about generics. This is why Java generics are sometimes called "syntactic sugar" -- they make the source code safer and cleaner, but the bytecode is identical to what you would write with manual casts. This design decision was made for backward compatibility: generic code can interoperate with pre-Java 5 code that uses raw types.

---

## Syntax and Basic Examples

### Example 1: Generic class with multiple type parameters

```java
import java.util.List;

public class ApiResponse<T> {
    private final boolean success;
    private final String message;
    private final T data;
    private final List<String> errors;

    private ApiResponse(boolean success, String message, T data, List<String> errors) {
        this.success = success;
        this.message = message;
        this.data = data;
        this.errors = errors;
    }

    // Static factory methods with generic return types
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data, List.of());
    }

    public static <T> ApiResponse<T> success(String message, T data) {
        return new ApiResponse<>(true, message, data, List.of());
    }

    public static <T> ApiResponse<T> error(String message, List<String> errors) {
        return new ApiResponse<>(false, message, null, errors);
    }

    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, message, null, List.of(message));
    }

    // Getters
    public boolean isSuccess() { return success; }
    public String getMessage() { return message; }
    public T getData() { return data; }
    public List<String> getErrors() { return errors; }
}
```

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        // ApiResponse<User>
        ApiResponse<User> userResponse = ApiResponse.success(
            new User(1L, "Saad", "saad@example.com")
        );
        System.out.println("Success: " + userResponse.isSuccess());
        System.out.println("User: " + userResponse.getData().name());

        // ApiResponse<List<Order>>
        ApiResponse<List<Order>> ordersResponse = ApiResponse.success(
            "Found 3 orders",
            List.of(new Order(1L), new Order(2L), new Order(3L))
        );
        System.out.println("Orders: " + ordersResponse.getData().size());

        // ApiResponse<Void> (no data, just success/failure)
        ApiResponse<Void> deleteResponse = ApiResponse.success("Deleted", null);
        System.out.println("Delete: " + deleteResponse.getMessage());

        // Error response
        ApiResponse<User> errorResponse = ApiResponse.error(
            "Validation failed",
            List.of("Email is required", "Name must be at least 2 characters")
        );
        System.out.println("Errors: " + errorResponse.getErrors());
    }
}

record User(Long id, String name, String email) {}
record Order(Long id) {}
```

### Example 2: Generic methods and type inference

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;

public class GenericMethods {

    // Generic method: swaps two elements in a list
    public static <T> void swap(List<T> list, int i, int j) {
        T temp = list.get(i);
        list.set(i, list.get(j));
        list.set(j, temp);
    }

    // Generic method: finds the first element matching a predicate
    public static <T> T findFirst(List<T> list, Predicate<T> predicate) {
        for (T item : list) {
            if (predicate.test(item)) {
                return item;
            }
        }
        return null;
    }

    // Generic method: converts a list of one type to a list of another type
    public static <T, R> List<R> transform(List<T> source, Function<T, R> mapper) {
        List<R> result = new ArrayList<>();
        for (T item : source) {
            result.add(mapper.apply(item));
        }
        return result;
    }

    public static void main(String[] args) {
        // Type inference: the compiler infers T from the arguments
        List<String> names = new ArrayList<>(List.of("Saad", "Rahim", "Karim"));
        swap(names, 0, 2);
        System.out.println("After swap: " + names);  // [Karim, Rahim, Saad]

        // Type inference with predicate
        String found = findFirst(names, n -> n.startsWith("R"));
        System.out.println("Found: " + found);  // Rahim

        // Type inference with two type parameters
        List<Integer> lengths = transform(names, String::length);
        System.out.println("Lengths: " + lengths);  // [5, 5, 5]

        List<String> upper = transform(names, String::toUpperCase);
        System.out.println("Upper: " + upper);  // [KARIM, RAHIM, SAAD]
    }
}
```

### Example 3: Bounded type parameters

```java
import java.util.List;

public class BoundedGenerics {

    // T must be Comparable (so we can compare elements)
    public static <T extends Comparable<T>> T findMax(List<T> list) {
        if (list.isEmpty()) {
            throw new IllegalArgumentException("List is empty");
        }
        T max = list.get(0);
        for (T item : list) {
            if (item.compareTo(max) > 0) {
                max = item;
            }
        }
        return max;
    }

    // T must be a Number (so we can call doubleValue())
    public static <T extends Number> double average(List<T> numbers) {
        if (numbers.isEmpty()) return 0;
        double sum = 0;
        for (T n : numbers) {
            sum += n.doubleValue();
        }
        return sum / numbers.size();
    }

    // T must implement both Comparable and CharSequence
    public static <T extends Comparable<T> & CharSequence> T longestString(List<T> strings) {
        T longest = strings.get(0);
        for (T s : strings) {
            if (s.length() > longest.length()) {
                longest = s;
            }
        }
        return longest;
    }

    public static void main(String[] args) {
        System.out.println("Max int: " + findMax(List.of(3, 1, 4, 1, 5, 9)));      // 9
        System.out.println("Max string: " + findMax(List.of("apple", "cherry", "banana")));  // cherry

        System.out.println("Avg int: " + average(List.of(10, 20, 30)));        // 20.0
        System.out.println("Avg double: " + average(List.of(1.5, 2.5, 3.5)));  // 2.5

        System.out.println("Longest: " + longestString(List.of("Hi", "Hello", "Hey")));  // Hello
    }
}
```

### Example 4: Wildcards in action (PECS)

```java
import java.util.ArrayList;
import java.util.List;

public class WildcardDemo {

    // Producer: reads from the list (? extends Number)
    // We can read Numbers from this list, but we cannot add to it
    // (because we do not know the exact type: it might be List<Integer>,
    // List<Double>, etc., and adding the wrong type would corrupt it).
    public static double sum(List<? extends Number> numbers) {
        double total = 0;
        for (Number n : numbers) {
            total += n.doubleValue();
        }
        return total;
    }

    // Consumer: writes to the list (? super Integer)
    // We can add Integers to this list because it accepts Integer
    // or any supertype (Number, Object).
    public static void addDefaults(List<? super Integer> list) {
        list.add(1);
        list.add(2);
        list.add(3);
    }

    // Both read and write: exact type parameter (no wildcard)
    public static <T> void rotate(List<T> list, int positions) {
        if (list.isEmpty()) return;
        for (int i = 0; i < positions; i++) {
            T first = list.remove(0);  // Read
            list.add(first);           // Write
        }
    }

    public static void main(String[] args) {
        // Producer example
        List<Integer> ints = List.of(1, 2, 3);
        List<Double> doubles = List.of(1.5, 2.5, 3.5);
        System.out.println("Sum ints: " + sum(ints));        // 6.0
        System.out.println("Sum doubles: " + sum(doubles));  // 7.5

        // Consumer example
        List<Number> numbers = new ArrayList<>();
        List<Object> objects = new ArrayList<>();
        addDefaults(numbers);  // OK: Number is a supertype of Integer
        addDefaults(objects);  // OK: Object is a supertype of Integer
        System.out.println("Numbers: " + numbers);  // [1, 2, 3]
        System.out.println("Objects: " + objects);  // [1, 2, 3]

        // Read and write example
        List<String> names = new ArrayList<>(List.of("A", "B", "C", "D"));
        rotate(names, 2);
        System.out.println("Rotated: " + names);  // [C, D, A, B]
    }
}
```

### Example 5: Raw types (what NOT to do)

```java
import java.util.ArrayList;
import java.util.List;

public class RawTypeDemo {
    public static void main(String[] args) {
        // RAW TYPE: no type parameter specified.
        // This disables all generic type checking.
        List rawList = new ArrayList();  // WARNING: raw use of parameterized class
        rawList.add("Hello");
        rawList.add(42);
        rawList.add(new Object());

        // Retrieving elements requires explicit casting and is unsafe
        String s = (String) rawList.get(0);  // OK by luck
        String s2 = (String) rawList.get(1);  // ClassCastException at runtime!

        // PARAMETERIZED TYPE: type-safe
        List<String> safeList = new ArrayList<>();
        safeList.add("Hello");
        // safeList.add(42);  // COMPILATION ERROR! Type safety enforced.
        String safe = safeList.get(0);  // No cast needed
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Generics are deeply embedded in Spring Boot's architecture. Here are three realistic scenarios.

### Scenario 1: Spring Data JPA generic repository

Spring Data JPA's entire repository abstraction is built on generics. The `JpaRepository<T, ID>` interface uses two type parameters: `T` for the entity type and `ID` for the primary key type.

```java
package com.company.orderservice.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;
import java.util.Optional;

// T = Order (entity type), ID = Long (primary key type)
// Spring Data generates the implementation at runtime using dynamic proxies.
// All CRUD methods are inherited from JpaRepository with the correct types.
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Inherited methods from JpaRepository<Order, Long>:
    // Optional<Order> findById(Long id);       // T = Order, ID = Long
    // List<Order> findAll();                    // T = Order
    // Order save(Order entity);                 // T = Order
    // void deleteById(Long id);                 // ID = Long
    // long count();
    // boolean existsById(Long id);              // ID = Long

    // Custom query methods: Spring generates the SQL from the method name
    List<Order> findByUserId(Long userId);
    List<Order> findByStatus(OrderStatus status);
    Optional<Order> findByOrderNumber(String orderNumber);
}
```

```text
// The JpaRepository interface hierarchy (simplified):
//
// Repository<T, ID>                          (marker interface)
//   └── CrudRepository<T, ID>                (basic CRUD: save, findById, delete)
//         └── PagingAndSortingRepository<T, ID>  (adds findAll(Pageable))
//               └── JpaRepository<T, ID>     (adds flush, saveAll, batch operations)
//
// Each level uses the same type parameters T and ID, so the types
// flow through the entire hierarchy. When you declare
// OrderRepository extends JpaRepository<Order, Long>, the compiler
// knows that findById returns Optional<Order>, not Optional<Object>.
```

**What to notice:**

- The type parameters `T` and `ID` propagate through the entire repository hierarchy. `CrudRepository<T, ID>` defines `Optional<T> findById(ID id)`. When you extend `JpaRepository<Order, Long>`, the compiler resolves this to `Optional<Order> findById(Long id)`. No casts needed.
- Spring Data generates the implementation at runtime. The generic type information is used by Spring's reflection-based proxy to determine the entity class and construct the correct JPA queries.
- This is the most powerful example of generics in the Java ecosystem. A single generic interface provides type-safe CRUD operations for any entity type without writing any implementation code.

### Scenario 2: Generic base service class

Backend applications often share common CRUD logic across multiple services. A generic base service class eliminates this duplication.

```java
package com.company.orderservice.service;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;
import java.util.Optional;

// Generic base service that provides common CRUD operations for any entity.
// T = entity type, ID = primary key type.
public abstract class BaseService<T, ID> {

    // The repository is injected by the concrete subclass
    protected abstract JpaRepository<T, ID> getRepository();

    public Optional<T> findById(ID id) {
        return getRepository().findById(id);
    }

    public T getById(ID id) {
        return getRepository().findById(id)
            .orElseThrow(() -> new ResourceNotFoundException(
                getEntityName(), id
            ));
    }

    public List<T> findAll() {
        return getRepository().findAll();
    }

    public T create(T entity) {
        return getRepository().save(entity);
    }

    public T update(ID id, T entity) {
        if (!getRepository().existsById(id)) {
            throw new ResourceNotFoundException(getEntityName(), id);
        }
        return getRepository().save(entity);
    }

    public void delete(ID id) {
        if (!getRepository().existsById(id)) {
            throw new ResourceNotFoundException(getEntityName(), id);
        }
        getRepository().deleteById(id);
    }

    public long count() {
        return getRepository().count();
    }

    protected abstract String getEntityName();
}
```

```java
// Concrete service for Order entities:
package com.company.orderservice.service;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class OrderService extends BaseService<Order, Long> {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Override
    protected JpaRepository<Order, Long> getRepository() {
        return orderRepository;
    }

    @Override
    protected String getEntityName() {
        return "Order";
    }

    // Order-specific business logic (not shared with other services)
    public Order cancelOrder(Long orderId) {
        Order order = getById(orderId);  // Inherited from BaseService
        order.cancel();
        return orderRepository.save(order);
    }

    public List<Order> getUserOrders(Long userId) {
        return orderRepository.findByUserId(userId);
    }
}

// Concrete service for Product entities (reuses the same CRUD logic):
@Service
public class ProductService extends BaseService<Product, Long> {

    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Override
    protected JpaRepository<Product, Long> getRepository() {
        return productRepository;
    }

    @Override
    protected String getEntityName() {
        return "Product";
    }
}
```

**What to notice:**

- `BaseService<T, ID>` defines generic CRUD operations that work for any entity type. `OrderService` extends `BaseService<Order, Long>`, so `findById()` returns `Optional<Order>` and `delete()` accepts a `Long` ID. `ProductService` extends `BaseService<Product, Long>` and gets the same type-safe CRUD for products.
- The `getById()` method uses the generic `T` type parameter in its return type and the generic `ID` type parameter in its argument. The compiler ensures type safety across all subclasses.
- Concrete services only need to implement `getRepository()` and `getEntityName()`, plus any entity-specific business logic. All shared CRUD operations are inherited from the generic base class.

### Scenario 3: Generic DTO wrapper and response builder

```java
package com.company.orderservice.dto;

import java.time.LocalDateTime;
import java.util.List;

// Generic paginated response wrapper
public record PaginatedResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages,
    boolean hasNext
) {
    // Static factory method with generic type inference
    public static <T> PaginatedResponse<T> of(List<T> content, int page, int size, long total) {
        int totalPages = (int) Math.ceil((double) total / size);
        return new PaginatedResponse<>(
            content, page, size, total, totalPages, page < totalPages - 1
        );
    }

    public static <T> PaginatedResponse<T> empty(int page, int size) {
        return new PaginatedResponse<>(List.of(), page, size, 0, 0, false);
    }
}
```

```java
// Generic API response envelope
import java.time.LocalDateTime;

public record ApiEnvelope<T>(
    LocalDateTime timestamp,
    String status,
    T data,
    String error
) {
    public static <T> ApiEnvelope<T> success(T data) {
        return new ApiEnvelope<>(LocalDateTime.now(), "SUCCESS", data, null);
    }

    public static <T> ApiEnvelope<T> error(String message) {
        return new ApiEnvelope<>(LocalDateTime.now(), "ERROR", null, message);
    }
}
```

```java
// Usage in a controller:
import org.springframework.data.domain.Page;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    @GetMapping
    public ApiEnvelope<PaginatedResponse<OrderResponse>> getOrders(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {

        Page<Order> orderPage = orderService.getOrders(page, size);

        List<OrderResponse> content = orderPage.getContent().stream()
            .map(OrderResponse::fromEntity)
            .toList();

        PaginatedResponse<OrderResponse> paginated = PaginatedResponse.of(
            content, page, size, orderPage.getTotalElements()
        );

        return ApiEnvelope.success(paginated);
    }
}
```

**JSON response:**

```text
// {
//   "timestamp": "2025-07-10T14:30:00",
//   "status": "SUCCESS",
//   "data": {
//     "content": [{"id": 1, "orderNumber": "ORD-001", ...}],
//     "page": 0,
//     "size": 20,
//     "totalElements": 150,
//     "totalPages": 8,
//     "hasNext": true
//   },
//   "error": null
// }
```

**What to notice:**

- `PaginatedResponse<T>` is parameterized by the content type. `PaginatedResponse<OrderResponse>` and `PaginatedResponse<UserResponse>` are different types at compile time, providing type safety across different API endpoints.
- `ApiEnvelope<T>` wraps any response type in a consistent envelope. The `data` field is of type `T`, so it can hold a single object, a list, a paginated response, or any other type.
- The nested generics `ApiEnvelope<PaginatedResponse<OrderResponse>>` demonstrate how generic types compose. The compiler tracks the full type chain and ensures that the JSON serializer produces the correct structure.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using raw types instead of parameterized types

**Wrong:**

```java
List list = new ArrayList();  // Raw type: no type safety
Map map = new HashMap();      // Raw type: no type safety
list.add("Hello");
list.add(42);  // No error! Mixed types allowed.
String s = (String) list.get(1);  // ClassCastException at runtime
```

**Right:**

```java
List<String> list = new ArrayList<>();  // Parameterized: type-safe
Map<String, Integer> map = new HashMap<>();  // Parameterized: type-safe
list.add("Hello");
// list.add(42);  // COMPILATION ERROR! Type safety enforced.
String s = list.get(0);  // No cast needed
```

**Why it is wrong:** Raw types disable all generic type checking. The compiler treats a raw `List` as `List<Object>`, allowing any type to be added. This defeats the entire purpose of generics and reintroduces the `ClassCastException` bugs that generics were designed to eliminate. Raw types exist only for backward compatibility with pre-Java 5 code. Never use them in new code.

### Mistake 2: Trying to create instances or arrays of type parameters

**Wrong:**

```java
public class Container<T> {
    public T create() {
        return new T();  // COMPILATION ERROR! Cannot instantiate type parameter.
    }

    public T[] createArray(int size) {
        return new T[size];  // COMPILATION ERROR! Cannot create generic array.
    }

    public void checkType(Object obj) {
        if (obj instanceof T) {  // COMPILATION ERROR! Cannot use instanceof with T.
            // ...
        }
    }
}
```

**Right:**

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Supplier;

public class Container<T> {
    // Use a Class object or Supplier to create instances
    private final Supplier<T> factory;

    public Container(Supplier<T> factory) {
        this.factory = factory;
    }

    public T create() {
        return factory.get();  // Creates a new T using the supplier
    }

    // Use a List instead of an array
    public List<T> createList(int size) {
        List<T> list = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            list.add(factory.get());
        }
        return list;
    }

    // Use Class<T> for type checking
    public boolean checkType(Object obj, Class<T> type) {
        return type.isInstance(obj);
    }
}
```

**Why it is wrong:** Type erasure removes all generic type information at runtime. The JVM does not know what `T` is, so it cannot create instances, allocate arrays, or perform type checks with `T`. The workarounds are to pass a `Class<T>` object, a `Supplier<T>`, or use a `List<T>` instead of an array.

### Mistake 3: Confusing `List<Object>` with `List<?>`

**Wrong thinking:** "I want a method that accepts any list, so I will use `List<Object>`."

```java
public void printAll(List<Object> list) { ... }

List<String> names = List.of("Saad", "Rahim");
printAll(names);  // COMPILATION ERROR! List<String> is NOT a List<Object>.
```

**Right:**

```java
public void printAll(List<?> list) { ... }  // Accepts any List

List<String> names = List.of("Saad", "Rahim");
printAll(names);  // OK! List<String> is a List<?>

List<Integer> numbers = List.of(1, 2, 3);
printAll(numbers);  // OK! List<Integer> is a List<?>
```

**Why it is wrong:** `List<String>` is NOT a subtype of `List<Object>`, even though `String` is a subtype of `Object`. This is because generics are **invariant**: `List<A>` and `List<B>` have no subtype relationship regardless of the relationship between `A` and `B`. This invariance is a safety feature. If `List<String>` were a subtype of `List<Object>`, you could add an `Integer` to a `List<String>` through the `List<Object>` reference, corrupting the list. The wildcard `List<?>` is the correct way to express "a list of some unknown type."

### Mistake 4: Adding elements to a `List<? extends T>`

**Wrong:**

```java
public void addElement(List<? extends Number> list) {
    list.add(42);  // COMPILATION ERROR!
    // The list might be a List<Double>. Adding an Integer would corrupt it.
    // The compiler prevents this because it does not know the exact type.
}
```

**Right:**

```java
// If you need to add elements, use ? super T (consumer)
public void addElement(List<? super Integer> list) {
    list.add(42);  // OK! The list accepts Integer or any supertype.
}

// If you need to both read and write, use an exact type parameter
public <T extends Number> void process(List<T> list, T element) {
    list.add(element);  // OK! T is a specific type known at the call site.
    T first = list.get(0);  // OK!
}
```

**Why it is wrong:** `List<? extends Number>` means "a list of some specific but unknown subtype of Number." It could be `List<Integer>`, `List<Double>`, or `List<Long>`. The compiler cannot allow you to add an `Integer` because the list might actually be a `List<Double>`. The PECS principle explains this: `? extends T` is for producers (reading), not consumers (writing).

---

## Key Takeaways

> [!tip] Remember these points
> 1. **Generics** parameterize types, enabling compile-time type safety without runtime overhead. They eliminate unsafe casts and make code self-documenting. Use parameterized types (`List<String>`) everywhere; never use raw types (`List`).
> 2. **Generic classes** declare type parameters after the class name (`class Box<T>`). **Generic methods** declare type parameters before the return type (`<T> void method(T item)`). The compiler infers type parameters from the context in most cases.
> 3. **Bounded type parameters** (`<T extends Number>`) restrict the types that can be used as type arguments. Use them when your generic code needs to call methods on the type parameter (e.g., `doubleValue()` on `Number`, `compareTo()` on `Comparable`).
> 4. **Wildcards** (`?`, `? extends T`, `? super T`) represent unknown types in method parameters. Follow the **PECS principle**: use `? extends T` for producers (reading), `? super T` for consumers (writing), and exact type parameters for both.
> 5. **Type erasure** means generics exist only at compile time. The JVM does not know about type parameters at runtime. This is why you cannot instantiate type parameters (`new T()`), create generic arrays (`new T[10]`), or use `instanceof` with parameterized types. Use `Class<T>`, `Supplier<T>`, or `List<T>` as workarounds.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Generic Pair and Triple Classes (Easy)

Create a generic `Pair<A, B>` class with fields `first` and `second`, a constructor, getters, `equals()`, `hashCode()`, and `toString()`. Then create a generic `Triple<A, B, C>` class that extends the concept to three fields. Test both classes with different type combinations: `Pair<String, Integer>`, `Pair<Long, Boolean>`, `Triple<String, Integer, Double>`.

> **Hint:** Use `Objects.equals()` and `Objects.hash()` for the `equals()` and `hashCode()` implementations. The type parameters allow the same class to work with any combination of types.

### Exercise 2: Generic Utility Methods (Medium)

Create a `CollectionUtils` class with the following static generic methods:

1. `<T> T first(List<T> list)`: returns the first element or null if empty.
2. `<T> T last(List<T> list)`: returns the last element or null if empty.
3. `<T extends Comparable<T>> T min(List<T> list)`: returns the minimum element.
4. `<T> List<T> filter(List<T> list, Predicate<T> predicate)`: returns a new list with only the matching elements.
5. `<T, R> List<R> map(List<T> list, Function<T, R> mapper)`: transforms each element.
6. `<T> List<T> concat(List<? extends T> list1, List<? extends T> list2)`: concatenates two lists (note the wildcard for flexibility).

Test all methods with `List<String>` and `List<Integer>`.

> **Hint:** The `concat` method uses `? extends T` so you can concatenate a `List<Integer>` with a `List<Number>` into a `List<Number>`. This is the PECS principle in action.

### Exercise 3: Generic Repository Simulation (Medium)

Simulate Spring Data's generic repository pattern without Spring:

1. Create an interface `Repository<T, ID>` with methods `save(T entity)`, `findById(ID id)`, `findAll()`, `deleteById(ID id)`, and `count()`.
2. Create an abstract class `InMemoryRepository<T, ID>` that implements `Repository<T, ID>` using a `Map<ID, T>` internally. It should have an abstract method `ID getId(T entity)` that subclasses implement to extract the ID from the entity.
3. Create `UserRepository extends InMemoryRepository<User, Long>` and `ProductRepository extends InMemoryRepository<Product, String>`.
4. Test both repositories with CRUD operations.

> **Hint:** The `InMemoryRepository` uses the generic `ID` type as the map key and `T` as the map value. The `getId()` method is the bridge between the entity object and its ID.

### Exercise 4: Type-Safe Event Bus with Generics (Hard, Optional)

Build a type-safe event bus that routes events to the correct handlers based on the event type:

1. Create a base `Event` interface and several event types: `OrderCreatedEvent`, `PaymentCompletedEvent`, `UserRegisteredEvent`.
2. Create a generic `EventHandler<T extends Event>` interface with a `handle(T event)` method.
3. Create an `EventBus` class that maintains a `Map<Class<? extends Event>, List<EventHandler<?>>>` of registered handlers.
4. Implement `register(Class<T> eventType, EventHandler<T> handler)` and `publish(Event event)` methods. The `publish` method should find all handlers registered for the event's class and invoke them.
5. Test by registering handlers for different event types and publishing events. Verify that each handler only receives events of its registered type.

> **Hint:** The `Map` key is `Class<? extends Event>`, which allows you to look up handlers by the event's runtime class. The wildcard `EventHandler<?>` in the value type is necessary because the map stores handlers for different event types. You will need an unchecked cast inside `publish()` to call the handler with the correct type, which is safe because you verified the class match.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.Objects;

class Pair<A, B> {
    private final A first;
    private final B second;

    Pair(A first, B second) {
        this.first = first;
        this.second = second;
    }

    public A first() { return first; }
    public B second() { return second; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Pair<?, ?> p)) return false;
        return Objects.equals(first, p.first) && Objects.equals(second, p.second);
    }

    @Override
    public int hashCode() { return Objects.hash(first, second); }

    @Override
    public String toString() { return "(" + first + ", " + second + ")"; }
}

class Triple<A, B, C> {
    private final A first;
    private final B second;
    private final C third;

    Triple(A first, B second, C third) {
        this.first = first; this.second = second; this.third = third;
    }

    public A first() { return first; }
    public B second() { return second; }
    public C third() { return third; }

    @Override
    public String toString() { return "(" + first + ", " + second + ", " + third + ")"; }
}

public class Main {
    public static void main(String[] args) {
        Pair<String, Integer> p1 = new Pair<>("Saad", 22);
        Pair<String, Integer> p2 = new Pair<>("Saad", 22);
        System.out.println(p1);  // (Saad, 22)
        System.out.println("Equal: " + p1.equals(p2));  // true

        Triple<String, Integer, Double> t = new Triple<>("Saad", 22, 3.72);
        System.out.println(t);  // (Saad, 22, 3.72)
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;
import java.util.function.*;

public class CollectionUtils {

    public static <T> T first(List<T> list) {
        return list.isEmpty() ? null : list.get(0);
    }

    public static <T> T last(List<T> list) {
        return list.isEmpty() ? null : list.get(list.size() - 1);
    }

    public static <T extends Comparable<T>> T min(List<T> list) {
        if (list.isEmpty()) throw new IllegalArgumentException("Empty list");
        T min = list.get(0);
        for (T item : list) {
            if (item.compareTo(min) < 0) min = item;
        }
        return min;
    }

    public static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
        List<T> result = new ArrayList<>();
        for (T item : list) {
            if (predicate.test(item)) result.add(item);
        }
        return result;
    }

    public static <T, R> List<R> map(List<T> list, Function<T, R> mapper) {
        List<R> result = new ArrayList<>();
        for (T item : list) {
            result.add(mapper.apply(item));
        }
        return result;
    }

    public static <T> List<T> concat(List<? extends T> a, List<? extends T> b) {
        List<T> result = new ArrayList<>(a);
        result.addAll(b);
        return result;
    }

    public static void main(String[] args) {
        List<Integer> nums = List.of(5, 2, 8, 1, 9);
        System.out.println("First: " + first(nums));
        System.out.println("Last: " + last(nums));
        System.out.println("Min: " + min(nums));
        System.out.println("Even: " + filter(nums, n -> n % 2 == 0));
        System.out.println("Doubled: " + map(nums, n -> n * 2));

        List<Integer> a = List.of(1, 2);
        List<Integer> b = List.of(3, 4);
        System.out.println("Concat: " + concat(a, b));
    }
}
```

</details>

---

## Related Notes

- [[Java - Queue and Deque]]
- [[Java - Comparable and Comparator]] (next note)
- [[Java - Java 8 Lambdas and Functional Interfaces]]
- [[Java - Java 8 Streams API]]

---

## Resources

- [Oracle Java Tutorials: Generics](https://docs.oracle.com/javase/tutorial/java/generics/) - Official documentation covering all aspects of generics with detailed examples.
- [Oracle Java Tutorials: Wildcards](https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html) - Official guide to upper-bounded, lower-bounded, and unbounded wildcards.
- [Oracle Java Tutorials: Type Erasure](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html) - Official explanation of how type erasure works and its consequences.
- [Baeldung: Java Generics Guide](https://www.baeldung.com/java-generics) - Comprehensive guide covering generic classes, methods, bounds, and wildcards.
- [Baeldung: Java Generics PECS](https://www.baeldung.com/java-generics-pecs) - Detailed explanation of the Producer Extends, Consumer Super principle.
- [Effective Java by Joshua Bloch - Item 26: Don't Use Raw Types](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive argument against raw types.
- [Effective Java by Joshua Bloch - Item 28: Prefer Lists to Arrays](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Explains why generics and arrays do not mix well due to type erasure.
- [Effective Java by Joshua Bloch - Item 31: Use Bounded Wildcards to Increase API Flexibility](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive guide to using wildcards in public APIs.
