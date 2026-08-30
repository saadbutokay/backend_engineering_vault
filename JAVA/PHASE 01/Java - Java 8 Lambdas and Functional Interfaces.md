---
title: "Java - Java 8 Lambdas and Functional Interfaces"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - lambdas
  - functional-interfaces
  - java8
  - method-references
status: "not-started"
---

# Java - Java 8 Lambdas and Functional Interfaces

> [!abstract] Overview
> Lambdas and functional interfaces are the foundation of functional programming in Java, introduced in Java 8 (2014). A **lambda expression** is a concise way to represent an anonymous function -- a block of code that can be passed as an argument, returned from a method, or stored in a variable. A **functional interface** is an interface with exactly one abstract method, which serves as the target type for lambda expressions. Together, they enable behavior parameterization: instead of passing data to methods, you pass behavior. In backend development, lambdas are used everywhere: filtering collections, transforming DTOs, handling events, configuring Spring beans, defining validation rules, and building the Streams API pipelines that process millions of records.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Abstraction - Abstract Classes and Interfaces]]
> - [[Java - Generics - Classes Methods Wildcards]]
> - [[Java - Comparable and Comparator]]
> - [[Java - Collections Framework Overview]]

---

## Theory

### What is a Lambda Expression?

A lambda expression is a compact syntax for creating an anonymous implementation of a functional interface. Before Java 8, if you wanted to pass a block of behavior to a method, you had to create an anonymous inner class:

```java
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

// Pre-Java 8: anonymous inner class (verbose)
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
});
```

With Java 8 lambdas, the same code becomes:

```java
import java.util.Collections;
import java.util.List;

// Java 8+: lambda expression (concise)
Collections.sort(names, (a, b) -> a.length() - b.length());
```

The lambda `(a, b) -> a.length() - b.length()` is a function that takes two strings and returns an integer. It implements the `Comparator<String>` interface's single abstract method `compare()`. The compiler infers the interface, the method, and the parameter types from the context.

### The Lambda Syntax

A lambda expression has three parts: **parameters**, an **arrow** (`->`), and a **body**.

```text
(parameters) -> expression
(parameters) -> { statements; }
```

**Syntax variations:**

```java
// 1. No parameters
() -> System.out.println("Hello")

// 2. One parameter (parentheses optional)
name -> name.toUpperCase()
(name) -> name.toUpperCase()

// 3. Multiple parameters (parentheses required)
(a, b) -> a + b

// 4. Explicit parameter types (usually inferred, rarely needed)
(String a, String b) -> a.compareTo(b)

// 5. Single expression (return is implicit, no curly braces)
x -> x * 2

// 6. Block body (explicit return required for non-void)
(x, y) -> {
    int sum = x + y;
    return sum * 2;
}

// 7. Void return
(String message) -> { System.out.println(message); }
```

### What is a Functional Interface?

A functional interface is an interface that has **exactly one abstract method**. It can have any number of `default` methods and `static` methods, but only one abstract method. The `@FunctionalInterface` annotation is optional but recommended -- it tells the compiler to verify that the interface has exactly one abstract method.

```java
@FunctionalInterface
public interface Validator<T> {
    boolean isValid(T value);  // The single abstract method (SAM)

    // Default methods do NOT count toward the SAM limit
    default Validator<T> and(Validator<T> other) {
        return value -> this.isValid(value) && other.isValid(value);
    }

    // Static methods do NOT count either
    static <T> Validator<T> alwaysTrue() {
        return value -> true;
    }
}
```

**Why exactly one abstract method?**

A lambda expression provides the implementation for a single method. If the interface had two abstract methods, the compiler would not know which method the lambda is implementing. The single abstract method (SAM) rule eliminates this ambiguity.

**Examples of functional interfaces you have already used:**

| Interface | Abstract Method | Used In |
|-----------|----------------|---------|
| `Comparator<T>` | `int compare(T o1, T o2)` | `list.sort()`, `TreeSet`, `TreeMap` |
| `Runnable` | `void run()` | `Thread`, `ExecutorService` |
| `Comparable<T>` | `int compareTo(T o)` | Sorting (but rarely used as a lambda target) |

### Built-in Functional Interfaces (java.util.function)

Java 8 introduced 43 functional interfaces in the `java.util.function` package. The four core interfaces cover the most common use cases:

| Interface | Method | Input | Output | Use Case |
|-----------|--------|-------|--------|----------|
| `Predicate<T>` | `boolean test(T t)` | One argument | `boolean` | Filtering, validation |
| `Function<T, R>` | `R apply(T t)` | One argument | One result | Transformation, mapping |
| `Consumer<T>` | `void accept(T t)` | One argument | Nothing | Side effects, logging, printing |
| `Supplier<T>` | `T get()` | Nothing | One result | Lazy creation, factories |

**Specialized variants for primitives** (avoid autoboxing overhead):

| Interface | Method | Use Case |
|-----------|--------|----------|
| `IntPredicate` | `boolean test(int value)` | Filtering integers |
| `IntFunction<R>` | `R apply(int value)` | Transforming integers |
| `IntConsumer` | `void accept(int value)` | Processing integers |
| `IntSupplier` | `int getAsInt()` | Generating integers |
| `LongPredicate`, `DoubleFunction`, etc. | Similar patterns | Other primitive types |

**Two-argument variants:**

| Interface | Method | Use Case |
|-----------|--------|----------|
| `BiPredicate<T, U>` | `boolean test(T t, U u)` | Two-argument filtering |
| `BiFunction<T, U, R>` | `R apply(T t, U u)` | Two-argument transformation |
| `BiConsumer<T, U>` | `void accept(T t, U u)` | Two-argument side effects (e.g., `Map.forEach`) |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | Combining two values of the same type |
| `UnaryOperator<T>` | `T apply(T t)` | Transforming a value to the same type |

### Method References

Method references are a shorthand for lambdas that call a single existing method. They use the `::` operator.

| Type | Syntax | Lambda Equivalent | Example |
|------|--------|-------------------|---------|
| Static method | `ClassName::methodName` | `(args) -> ClassName.methodName(args)` | `Integer::parseInt` |
| Instance method of a specific object | `object::methodName` | `(args) -> object.methodName(args)` | `System.out::println` |
| Instance method of an arbitrary object | `ClassName::methodName` | `(obj, args) -> obj.methodName(args)` | `String::toUpperCase` |
| Constructor | `ClassName::new` | `(args) -> new ClassName(args)` | `ArrayList::new` |

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Supplier;

// Lambda vs Method Reference equivalents:

// Static method
Function<String, Integer> parser = s -> Integer.parseInt(s);
Function<String, Integer> parser = Integer::parseInt;

// Instance method of a specific object
Consumer<String> printer = s -> System.out.println(s);
Consumer<String> printer = System.out::println;

// Instance method of an arbitrary object
Function<String, String> upper = s -> s.toUpperCase();
Function<String, String> upper = String::toUpperCase;

// Constructor
Supplier<List<String>> listFactory = () -> new ArrayList<>();
Supplier<List<String>> listFactory = ArrayList::new;

// Constructor with arguments
Function<String, User> userFactory = name -> new User(name);
Function<String, User> userFactory = User::new;
```

### Variable Capture and Effectively Final

Lambdas can access variables from their enclosing scope, but those variables must be **final** or **effectively final** (assigned only once). This restriction exists because the lambda captures a copy of the variable's value, not a reference to the variable itself. If the variable could change after the lambda is created, the lambda would hold a stale copy.

```java
import java.util.List;

public void processOrders(List<Order> orders) {
    double taxRate = 0.15;  // Effectively final: assigned once

    orders.forEach(order -> {
        double tax = order.getTotal() * taxRate;  // OK: taxRate is effectively final
        System.out.println("Tax: " + tax);
    });

    // taxRate = 0.20;  // If you uncomment this, taxRate is no longer effectively final,
    // and the lambda above will cause a COMPILATION ERROR.
}
```

**Instance and static fields** can be accessed freely from lambdas because they are accessed through `this` (which is effectively final as a reference) or through the class name.

```java
import java.util.List;

public class OrderService {
    private double taxRate = 0.15;  // Instance field: accessible from lambdas

    public void process(List<Order> orders) {
        orders.forEach(order -> {
            double tax = order.getTotal() * this.taxRate;  // OK
            System.out.println("Tax: " + tax);
        });
    }
}
```

### How Lambdas Work Internally

Lambdas are NOT syntactic sugar for anonymous inner classes. They are implemented using the `invokedynamic` bytecode instruction (introduced in Java 7 for dynamic languages). When the JVM encounters a lambda for the first time, it:

1. Generates a synthetic method containing the lambda body.
2. Uses `LambdaMetafactory` to create an instance of the functional interface that delegates to the synthetic method.
3. Caches the generated class for subsequent invocations.

This is more efficient than anonymous inner classes because:

- No `.class` file is generated for each lambda (anonymous inner classes generate `OuterClass$1.class`, `OuterClass$2.class`, etc.).
- The lambda instance can be a singleton if it does not capture any variables (stateless lambdas).
- The JVM can optimize lambda instantiation more aggressively than anonymous class creation.

**Stateless lambda (singleton):**

```java
import java.util.function.Predicate;

// This lambda captures no variables. The JVM creates a single instance
// and reuses it for every invocation.
Predicate<String> isNotEmpty = s -> !s.isEmpty();
```

**Stateful lambda (new instance per capture):**

```java
import java.util.function.Predicate;

// This lambda captures 'prefix'. The JVM creates a new instance
// each time because the captured value may differ.
String prefix = "ORD-";
Predicate<String> hasPrefix = s -> s.startsWith(prefix);
```

> [!tip] Key Insight
> Lambdas enable **behavior parameterization**: instead of hardcoding logic inside a method, you pass the logic as an argument. This is the foundation of the Strategy pattern, the Template Method pattern, and the entire Streams API. In backend development, behavior parameterization allows you to write generic methods that work with any filtering, mapping, or reduction logic without duplicating code. A single `filterAndTransform()` method can handle any business rule when the filter and transform logic are passed as lambdas.

---

## Syntax and Basic Examples

### Example 1: All four core functional interfaces

```java
import java.util.function.*;

public class FunctionalInterfaceDemo {
    public static void main(String[] args) {
        // Predicate: takes T, returns boolean
        Predicate<String> isValidEmail = email ->
            email != null && email.contains("@") && email.contains(".");

        System.out.println("saad@test.com valid: " + isValidEmail.test("saad@test.com"));  // true
        System.out.println("invalid valid: " + isValidEmail.test("invalid"));              // false

        // Function: takes T, returns R
        Function<String, String> normalizeEmail = email ->
            email.strip().toLowerCase();

        System.out.println("Normalized: " + normalizeEmail.apply("  SAAD@TEST.COM  "));
        // saad@test.com

        // Consumer: takes T, returns void
        Consumer<String> logger = message ->
            System.out.println("[LOG] " + message);

        logger.accept("Order created");  // [LOG] Order created

        // Supplier: takes nothing, returns T
        Supplier<String> orderIdGenerator = () ->
            "ORD-" + System.currentTimeMillis();

        System.out.println("Order ID: " + orderIdGenerator.get());
        // ORD-1720620000000
    }
}
```

### Example 2: Composing functional interfaces

```java
import java.util.function.*;

public class CompositionDemo {
    public static void main(String[] args) {
        // Predicate composition: and(), or(), negate()
        Predicate<Integer> isPositive = n -> n > 0;
        Predicate<Integer> isEven = n -> n % 2 == 0;
        Predicate<Integer> isLessThan100 = n -> n < 100;

        Predicate<Integer> isPositiveEvenUnder100 = isPositive.and(isEven).and(isLessThan100);

        System.out.println("50: " + isPositiveEvenUnder100.test(50));   // true
        System.out.println("51: " + isPositiveEvenUnder100.test(51));   // false (odd)
        System.out.println("-2: " + isPositiveEvenUnder100.test(-2));   // false (negative)
        System.out.println("200: " + isPositiveEvenUnder100.test(200)); // false (> 100)

        Predicate<Integer> isNegativeOrOdd = isPositive.negate().or(isEven.negate());
        System.out.println("-3: " + isNegativeOrOdd.test(-3));  // true
        System.out.println("4: " + isNegativeOrOdd.test(4));    // false

        // Function composition: andThen(), compose()
        Function<String, String> trim = String::strip;
        Function<String, String> toLower = String::toLowerCase;
        Function<String, String> addPrefix = s -> "user_" + s;

        // andThen: apply this first, then the next
        Function<String, String> normalize = trim.andThen(toLower).andThen(addPrefix);
        System.out.println(normalize.apply("  SAAD  "));  // user_saad

        // compose: apply the argument first, then this
        Function<String, String> normalize2 = addPrefix.compose(toLower).compose(trim);
        System.out.println(normalize2.apply("  SAAD  "));  // user_saad (same result)
    }
}
```

### Example 3: Lambdas with collections

```java
import java.util.*;

public class LambdaCollectionsDemo {
    public static void main(String[] args) {
        List<String> cities = new ArrayList<>(
            List.of("Dhaka", "Chittagong", "Sylhet", "Rajshahi", "Khulna", "Barishal")
        );

        // sort() with lambda
        cities.sort((a, b) -> a.length() - b.length());
        System.out.println("By length: " + cities);
        // [Dhaka, Sylhet, Khulna, Rajshahi, Barishal, Chittagong]

        // removeIf() with lambda (removes elements matching the predicate)
        cities.removeIf(city -> city.length() > 6);
        System.out.println("After removeIf: " + cities);
        // [Dhaka, Sylhet, Khulna]

        // forEach() with lambda
        System.out.println("All cities:");
        cities.forEach(city -> System.out.println("  - " + city));

        // replaceAll() with lambda
        cities.replaceAll(String::toUpperCase);
        System.out.println("Uppercase: " + cities);
        // [DHAKA, SYLHET, KHULNA]

        // Map operations with lambdas
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Saad", 85);
        scores.put("Rahim", 72);
        scores.put("Karim", 90);

        // forEach with BiConsumer
        scores.forEach((name, score) ->
            System.out.printf("  %s: %d%n", name, score)
        );

        // computeIfAbsent with lambda
        scores.computeIfAbsent("Nila", name -> 0);
        System.out.println("After computeIfAbsent: " + scores);

        // merge with lambda
        scores.merge("Saad", 10, (oldVal, newVal) -> oldVal + newVal);
        System.out.println("After merge: " + scores);  // Saad: 95
    }
}
```

### Example 4: Method references in detail

```java
import java.util.*;
import java.util.function.*;

public class MethodReferenceDemo {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(List.of("saad", "rahim", "karim", "nila"));

        // 1. Static method reference
        // Lambda: s -> Integer.parseInt(s)
        Function<String, Integer> parser = Integer::parseInt;
        System.out.println("Parsed: " + parser.apply("42"));  // 42

        // 2. Instance method of a specific object
        // Lambda: s -> System.out.println(s)
        Consumer<String> printer = System.out::println;
        names.forEach(printer);

        // 3. Instance method of an arbitrary object of a particular type
        // Lambda: s -> s.toUpperCase()
        names.replaceAll(String::toUpperCase);
        System.out.println("Upper: " + names);  // [SAAD, RAHIM, KARIM, NILA]

        // 4. Constructor reference
        // Lambda: () -> new ArrayList<>()
        Supplier<List<String>> listFactory = ArrayList::new;
        List<String> newList = listFactory.get();

        // Lambda: capacity -> new ArrayList<>(capacity)
        Function<Integer, List<String>> sizedListFactory = ArrayList::new;
        List<String> sizedList = sizedListFactory.apply(100);

        // 5. Constructor reference with a custom class
        record User(String name, String email) {}
        Function<String, User> userFactory = name -> new User(name, name + "@test.com");
        // Note: constructor references only work when the lambda arguments
        // match the constructor parameters exactly. For custom mapping,
        // use a lambda instead.

        User user = userFactory.apply("saad");
        System.out.println("User: " + user);
    }
}
```

### Example 5: Custom functional interfaces

```java
// Custom functional interface for validation
@FunctionalInterface
interface Validator<T> {
    boolean validate(T value);

    // Default methods for composition (do NOT count as the SAM)
    default Validator<T> and(Validator<T> other) {
        return value -> this.validate(value) && other.validate(value);
    }

    default Validator<T> or(Validator<T> other) {
        return value -> this.validate(value) || other.validate(value);
    }

    default Validator<T> negate() {
        return value -> !this.validate(value);
    }
}

// Custom functional interface for transformation with error handling
@FunctionalInterface
interface CheckedFunction<T, R> {
    R apply(T input) throws Exception;
}
```

```java
public class CustomFunctionalDemo {
    public static void main(String[] args) {
        // Compose validators using the custom functional interface
        Validator<String> notNull = s -> s != null;
        Validator<String> notEmpty = s -> !s.strip().isEmpty();
        Validator<String> validLength = s -> s.length() >= 3 && s.length() <= 50;
        Validator<String> validEmail = s -> s.contains("@") && s.contains(".");

        Validator<String> emailValidator = notNull.and(notEmpty).and(validLength).and(validEmail);

        System.out.println("saad@test.com: " + emailValidator.validate("saad@test.com"));  // true
        System.out.println("ab: " + emailValidator.validate("ab"));                         // false
        System.out.println("null: " + emailValidator.validate(null));                       // false
        System.out.println("no-at-sign: " + emailValidator.validate("no-at-sign"));         // false
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Lambdas are pervasive in modern Spring Boot applications. Here are three realistic scenarios.

### Scenario 1: DTO transformation with Function

Backend services frequently transform entities into DTOs using lambdas and the Streams API (covered in the next note).

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

    public List<OrderResponse> getUserOrders(Long userId) {
        return orderRepository.findByUserId(userId).stream()
            .map(this::toResponse)  // Method reference: equivalent to order -> this.toResponse(order)
            .toList();
    }

    public PageResponse<OrderResponse> getUserOrdersPaginated(Long userId, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
        Page<Order> orderPage = orderRepository.findByUserId(userId, pageable);

        return new PageResponse<>(
            orderPage.getContent().stream()
                .map(this::toResponse)  // Method reference for transformation
                .toList(),
            orderPage.getNumber(),
            orderPage.getSize(),
            orderPage.getTotalElements()
        );
    }

    // The transformation method that the method reference points to
    private OrderResponse toResponse(Order order) {
        return new OrderResponse(
            order.getId(),
            order.getOrderNumber(),
            order.getTotalAmount(),
            order.getStatus().name(),
            order.getItems().stream()
                .map(this::toItemResponse)  // Nested method reference
                .toList(),
            order.getCreatedAt()
        );
    }

    private OrderItemResponse toItemResponse(OrderItem item) {
        return new OrderItemResponse(
            item.getProduct().getName(),
            item.getQuantity(),
            item.getUnitPrice()
        );
    }
}
```

**What to notice:**

- `.map(this::toResponse)` is a method reference that transforms each `Order` entity into an `OrderResponse` DTO. This is equivalent to `.map(order -> this.toResponse(order))` but more concise.
- The transformation logic is extracted into a private method (`toResponse`) rather than being inlined as a lambda. This is preferred when the transformation is complex (more than one line) because it keeps the stream pipeline readable and the transformation logic testable.
- Nested method references (`.map(this::toItemResponse)` inside `.map(this::toResponse)`) are common when transforming nested objects.

### Scenario 2: Validation with Predicate composition

Backend services use composed predicates to build complex validation rules from simple, reusable components.

```java
package com.company.orderservice.validation;

import java.util.function.Predicate;
import java.math.BigDecimal;

public class OrderValidators {

    // Reusable predicate components
    public static final Predicate<CreateOrderRequest> HAS_ITEMS =
        request -> request.items() != null && !request.items().isEmpty();

    public static final Predicate<CreateOrderRequest> VALID_USER_ID =
        request -> request.userId() != null && request.userId() > 0;

    public static final Predicate<CreateOrderRequest> MAX_ITEMS =
        request -> request.items().size() <= 100;

    public static final Predicate<CreateOrderRequest> POSITIVE_QUANTITIES =
        request -> request.items().stream()
            .allMatch(item -> item.quantity() > 0);

    public static final Predicate<CreateOrderRequest> POSITIVE_PRICES =
        request -> request.items().stream()
            .allMatch(item -> item.unitPrice().compareTo(BigDecimal.ZERO) > 0);

    public static final Predicate<CreateOrderRequest> VALID_COUPON =
        request -> request.couponCode() == null
            || request.couponCode().matches("^[A-Z0-9]{5,10}$");

    // Composed validator: all rules must pass
    public static final Predicate<CreateOrderRequest> FULL_VALIDATION =
        HAS_ITEMS
            .and(VALID_USER_ID)
            .and(MAX_ITEMS)
            .and(POSITIVE_QUANTITIES)
            .and(POSITIVE_PRICES)
            .and(VALID_COUPON);
}
```

```java
// Usage in the service:
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    public Order createOrder(CreateOrderRequest request) {
        if (!OrderValidators.FULL_VALIDATION.test(request)) {
            throw new ValidationException("Order validation failed");
        }

        // Proceed with order creation...
        return orderRepository.save(buildOrder(request));
    }
}
```

**What to notice:**

- Each validation rule is a standalone `Predicate<CreateOrderRequest>` that can be tested independently. This makes unit testing trivial: `assertTrue(VALID_USER_ID.test(validRequest))`.
- The `and()` method composes predicates into a single validator. The composed predicate short-circuits: if `HAS_ITEMS` fails, the remaining predicates are not evaluated.
- This pattern is the **Strategy pattern** implemented with lambdas. Each predicate is a strategy, and the composed predicate is a composite strategy.

### Scenario 3: Spring configuration with lambdas

Spring Boot uses lambdas extensively in configuration classes for defining beans, customizing behavior, and registering handlers.

```java
package com.company.orderservice.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    // Lambda-based CORS configuration
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("https://example.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .maxAge(3600);
    }
}
```

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // Lambda-based security configuration (Spring Security 5.2+)
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.GET, "/api/v1/products/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint((request, response, authException) -> {
                    response.setContentType("application/json");
                    response.setStatus(401);
                    response.getWriter().write("{\"error\":\"Unauthorized\"}");
                })
            );

        return http.build();
    }
}
```

```java
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.time.Duration;

@Configuration
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager("products", "users");
        // Lambda-based cache configuration
        manager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(Duration.ofMinutes(30))
            .removalListener((key, value, cause) -> {
                // Lambda for cache eviction logging
                // logger.debug("Cache entry evicted: {} ({})", key, cause);
            })
        );
        return manager;
    }
}
```

**What to notice:**

- Spring Security's lambda DSL (`csrf -> csrf.disable()`, `auth -> auth.requestMatchers(...)`) replaced the older chained method style. Each lambda receives a configurer object that you customize. This pattern makes the configuration more readable and IDE-friendly.
- The `authenticationEntryPoint` lambda is an inline implementation of the `AuthenticationEntryPoint` functional interface. It handles unauthenticated requests by returning a JSON 401 response.
- The `removalListener` lambda is a `RemovalListener<Object, Object>` that logs cache evictions. This demonstrates how lambdas simplify callback registration.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using lambdas for complex logic

**Wrong:**

```java
orders.stream()
    .filter(order -> {
        if (order.getStatus() == OrderStatus.PENDING) {
            if (order.getTotalAmount().compareTo(BigDecimal.ZERO) > 0) {
                if (order.getItems().size() <= 100) {
                    return true;
                }
            }
        }
        return false;
    })
    .map(order -> {
        OrderResponse response = new OrderResponse();
        response.setId(order.getId());
        response.setOrderNumber(order.getOrderNumber());
        response.setTotal(order.getTotalAmount());
        response.setStatus(order.getStatus().name());
        response.setItems(order.getItems().stream()
            .map(item -> new OrderItemResponse(item.getName(), item.getQty()))
            .collect(Collectors.toList()));
        return response;
    })
    .collect(Collectors.toList());
```

**Right:**

```java
orders.stream()
    .filter(this::isValidPendingOrder)  // Extracted to a named method
    .map(this::toResponse)              // Extracted to a named method
    .toList();

private boolean isValidPendingOrder(Order order) {
    return order.getStatus() == OrderStatus.PENDING
        && order.getTotalAmount().compareTo(BigDecimal.ZERO) > 0
        && order.getItems().size() <= 100;
}

private OrderResponse toResponse(Order order) {
    // Transformation logic here
}
```

**Why it is wrong:** Lambdas should be short and readable. A lambda with more than 2-3 lines of logic is a code smell. Extract complex logic into named methods and use method references. This improves readability, testability, and reusability. The stream pipeline should read like a narrative: filter, map, collect. The implementation details belong in separate methods.

### Mistake 2: Modifying external state from a lambda (side effects)

**Wrong:**

```java
List<String> results = new ArrayList<>();
orders.forEach(order -> {
    if (order.isActive()) {
        results.add(order.getOrderNumber());  // Side effect: modifying external list
    }
});
```

**Right:**

```java
List<String> results = orders.stream()
    .filter(Order::isActive)
    .map(Order::getOrderNumber)
    .toList();
```

**Why it is wrong:** Lambdas that modify external state (side effects) defeat the purpose of functional programming and can cause bugs in parallel streams. The functional approach (filter + map + collect) is declarative, thread-safe, and more readable. Use `forEach` only for terminal side effects like logging or sending notifications, not for building collections.

### Mistake 3: Confusing `Consumer` and `Function`

**Wrong:**

```java
// Trying to use a Consumer where a Function is expected
Function<String, String> upper = s -> { System.out.println(s.toUpperCase()); };
// COMPILATION ERROR: Consumer returns void, but Function must return a String.
```

**Right:**

```java
// Consumer: performs an action, returns nothing
Consumer<String> printer = s -> System.out.println(s.toUpperCase());

// Function: transforms input to output
Function<String, String> upper = s -> s.toUpperCase();
```

**Why it is wrong:** `Consumer<T>` has a `void` return type. `Function<T, R>` must return a value of type `R`. If your lambda has a `return` statement (or an expression that produces a value), you need a `Function`. If your lambda performs a side effect (printing, logging, saving) and returns nothing, you need a `Consumer`.

### Mistake 4: Forgetting that lambdas capture values, not variables

**Wrong:**

```java
List<Runnable> tasks = new ArrayList<>();
for (int i = 0; i < 5; i++) {
    tasks.add(() -> System.out.println(i));  // COMPILATION ERROR!
    // 'i' is not effectively final because it changes in each loop iteration.
}
```

**Right:**

```java
List<Runnable> tasks = new ArrayList<>();
for (int i = 0; i < 5; i++) {
    final int index = i;  // Create an effectively final copy
    tasks.add(() -> System.out.println(index));
}

tasks.forEach(Runnable::run);  // Prints 0, 1, 2, 3, 4
```

**Why it is wrong:** The loop variable `i` changes on each iteration, so it is not effectively final. The lambda needs to capture a value that will not change. The fix is to create a final copy of the variable inside the loop body. Note that enhanced for-each loops do not have this problem because the loop variable is reassigned (not mutated) on each iteration, making it effectively final within each iteration.

---

## Key Takeaways

> [!tip] Remember these points
> 1. A **lambda expression** `(params) -> body` is a concise way to implement a functional interface. It enables behavior parameterization: passing logic as arguments to methods. Use lambdas for short, single-purpose logic; extract complex logic into named methods and use method references.
> 2. A **functional interface** has exactly one abstract method (SAM). Use the `@FunctionalInterface` annotation to enforce this at compile time. The four core interfaces are `Predicate<T>` (filtering), `Function<T, R>` (transformation), `Consumer<T>` (side effects), and `Supplier<T>` (lazy creation).
> 3. **Method references** (`ClassName::method`, `object::method`, `ClassName::new`) are shorthand for lambdas that call a single existing method. Use them when the lambda body is a single method call with matching parameters.
> 4. Lambdas capture variables from their enclosing scope, but those variables must be **final or effectively final**. Lambdas capture values, not references to variables. Instance and static fields can be accessed freely because `this` is effectively final.
> 5. Functional interfaces support **composition** through default methods: `Predicate.and()`, `Predicate.or()`, `Function.andThen()`, `Function.compose()`. This allows you to build complex behavior from simple, reusable components. In backend development, composed predicates are the standard pattern for validation, and composed functions are the standard pattern for DTO transformation pipelines.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Functional Interface Basics (Easy)

Write a program that demonstrates all four core functional interfaces:

1. `Predicate<String>`: checks if a string is a valid Bangladeshi phone number (starts with "+880" or "01", followed by 10 digits).
2. `Function<String, String>`: converts a name to the format "LAST, First" (e.g., "saad abdullah" -> "ABDULLAH, Saad").
3. `Consumer<Map.Entry<String, Integer>>`: prints a map entry in the format "key: value".
4. `Supplier<LocalDateTime>`: returns the current time.

Test each one with sample inputs and print the results.

> **Hint:** Use `String.matches()` with a regex for the phone number validator. Use `String.split(" ")` for the name formatter.

### Exercise 2: Composing Predicates for Validation (Medium)

Create a validation framework using composed predicates:

1. Define predicates for an `Order` record: `hasValidId`, `hasPositiveTotal`, `hasItems`, `hasValidStatus`, `isNotExpired`.
2. Compose them into a `FULL_VALIDATION` predicate using `and()`.
3. Create a `validate(Order order, Predicate<Order> validator)` method that returns a list of validation error messages (test each predicate individually and collect the failures).
4. Test with valid and invalid orders.

> **Hint:** To collect individual error messages, test each predicate separately in a loop or chain, rather than using the composed predicate (which short-circuits). Store each predicate with its error message in a `Map<Predicate<Order>, String>` or a list of pairs.

### Exercise 3: Method Reference Conversion (Medium)

Convert the following lambdas to method references (where possible). For lambdas that cannot be converted to method references, explain why.

1. `(String s) -> Integer.parseInt(s)`
2. `(String s) -> System.out.println(s)`
3. `(String s) -> s.toUpperCase()`
4. `(String a, String b) -> a.compareTo(b)`
5. `() -> new ArrayList<String>()`
6. `(String s) -> s.length() > 5`
7. `(Order o) -> o.getTotalAmount()`
8. `(int x) -> Math.abs(x)`

> **Hint:** Method references work when the lambda simply delegates to an existing method with matching parameters. Lambdas that contain additional logic (like `s.length() > 5`) cannot be converted directly.

### Exercise 4: Generic Functional Utility Class (Hard, Optional)

Create a `FunctionalUtils` class with the following static methods that use functional interfaces:

1. `<T> List<T> filter(List<T> list, Predicate<T> predicate)`: returns a new filtered list.
2. `<T, R> List<R> map(List<T> list, Function<T, R> mapper)`: returns a new transformed list.
3. `<T> void forEach(List<T> list, Consumer<T> action)`: applies the action to each element.
4. `<T, R> R reduce(List<T> list, R identity, BiFunction<R, T, R> accumulator)`: reduces the list to a single value.
5. `<T> Predicate<T> not(Predicate<T> predicate)`: returns the negation of a predicate.
6. `<T> Function<T, T> compose(Function<T, T>... functions)`: composes multiple functions into one.

Test all methods with sample data. This exercise essentially rebuilds the core of the Streams API using lambdas and functional interfaces.

> **Hint:** The `reduce` method starts with the `identity` value and applies the `accumulator` to each element: `result = accumulator.apply(result, element)`. For example, `reduce(List.of(1, 2, 3), 0, Integer::sum)` returns 6.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.time.LocalDateTime;
import java.util.Map;
import java.util.function.*;

public class Main {
    public static void main(String[] args) {
        Predicate<String> isValidPhone = phone ->
            phone != null && phone.matches("(\\+880|01)\\d{10}");

        System.out.println("+8801712345678: " + isValidPhone.test("+8801712345678"));  // true
        System.out.println("01712345678: " + isValidPhone.test("01712345678"));        // true
        System.out.println("12345: " + isValidPhone.test("12345"));                    // false

        Function<String, String> formatName = name -> {
            String[] parts = name.strip().split("\\s+");
            return parts[parts.length - 1].toUpperCase() + ", " + parts[0];
        };
        System.out.println(formatName.apply("saad abdullah"));  // ABDULLAH, saad

        Consumer<Map.Entry<String, Integer>> printEntry =
            entry -> System.out.println(entry.getKey() + ": " + entry.getValue());
        Map.of("Saad", 92, "Rahim", 78).forEach(printEntry);

        Supplier<LocalDateTime> now = LocalDateTime::now;
        System.out.println("Current time: " + now.get());
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```text
1. Integer::parseInt                          (static method)
2. System.out::println                        (instance method of specific object)
3. String::toUpperCase                        (instance method of arbitrary object)
4. String::compareTo                          (instance method of arbitrary object)
5. ArrayList::new                             (constructor reference)
6. CANNOT convert: contains additional logic (s.length() > 5)
7. Order::getTotalAmount                      (instance method of arbitrary object)
8. Math::abs                                  (static method, but needs cast: (IntUnaryOperator) Math::abs)
```

</details>

---

## Related Notes

- [[Java - Comparable and Comparator]]
- [[Java - Java 8 Streams API]] (next note)
- [[Java - Optional Class]]
- [[Java - Generics - Classes Methods Wildcards]]

---

## Resources

- [Oracle Java Tutorials: Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) - Official documentation covering lambda syntax, target typing, and variable capture.
- [Oracle Java Tutorials: Method References](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html) - Official guide to all four types of method references.
- [Oracle Java Documentation: java.util.function](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html) - Complete API reference for all 43 functional interfaces.
- [Baeldung: Java 8 Lambda Expressions](https://www.baeldung.com/java-8-lambda-expressions-tips) - Comprehensive guide with practical examples and best practices.
- [Baeldung: Java 8 Functional Interfaces](https://www.baeldung.com/java-8-functional-interfaces) - Detailed guide to the core functional interfaces and their composition methods.
- [Baeldung: Java Method References](https://www.baeldung.com/java-method-references) - Guide to all four types of method references with examples.
- [Effective Java by Joshua Bloch - Item 42: Prefer Lambdas to Anonymous Classes](https://www.oreilly.com/library/view/effective-java/9780134686097/) - When to use lambdas and when to stick with named classes.
- [Effective Java by Joshua Bloch - Item 43: Prefer Method References to Lambdas](https://www.oreilly.com/library/view/effective-java/9780134686097/) - When method references are clearer than lambdas.
