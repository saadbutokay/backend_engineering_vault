---
title: "Java - Optional Class"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - optional
  - java8
  - null-safety
status: "not-started"
---

# Java - Optional Class

> [!abstract] Overview
> `Optional<T>` is a container object introduced in Java 8 that may or may not contain a non-null value. It provides a type-safe, explicit alternative to returning `null` from methods, forcing callers to handle the "value absent" case consciously rather than risking a `NullPointerException` at runtime. In backend development, `Optional` is the standard return type for repository lookups (`findById()`), search operations, configuration values, and any method where "not found" is a normal, expected outcome rather than an error.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Java 8 Lambdas and Functional Interfaces]]
> - [[Java - Java 8 Streams API]]
> - [[Java - Generics - Classes Methods Wildcards]]
> - [[Java - Exception Handling - Try Catch Finally Throw Throws]]

---

## Theory

### What is Optional?

`Optional<T>` is a final class in the `java.util` package that wraps a value of type `T` or represents the absence of a value. It is a **container** with two possible states:

1. **Present**: The container holds a non-null value of type `T`.
2. **Empty**: The container holds no value (equivalent to null, but explicit and type-safe).

Before Java 8, the standard way to indicate "no result" was to return `null`:

```java
// Pre-Java 8: null return (error-prone)
public User findUserByEmail(String email) {
    // ... database query ...
    if (noResult) {
        return null;  // Caller might forget to check for null
    }
    return user;
}

// Caller:
User user = findUserByEmail("saad@example.com");
System.out.println(user.getName());  // NullPointerException if user is null!
```

With `Optional`:

```java
import java.util.Optional;

// Java 8+: Optional return (explicit)
public Optional<User> findUserByEmail(String email) {
    // ... database query ...
    if (noResult) {
        return Optional.empty();  // Explicitly signals "no value"
    }
    return Optional.of(user);
}

// Caller: forced to handle the absent case
Optional<User> user = findUserByEmail("saad@example.com");
user.ifPresent(u -> System.out.println(u.getName()));  // Safe: only runs if present
```

### Why Does Optional Exist?

`NullPointerException` is the most common runtime error in Java. Tony Hoare, the inventor of the null reference in 1965, called it his "billion-dollar mistake" because it has caused countless bugs, crashes, and security vulnerabilities over the decades.

`Optional` addresses this problem by:

1. **Making absence explicit in the type system**: A method returning `Optional<User>` communicates to the caller that the result might be absent. A method returning `User` implies the result is always present. The type signature becomes documentation.

2. **Forcing conscious handling**: The caller cannot accidentally call methods on an `Optional` as if it were the wrapped value. They must explicitly unwrap it using `get()`, `orElse()`, `ifPresent()`, or other methods. This makes the "absent" case a deliberate choice, not an oversight.

3. **Enabling functional composition**: `Optional` provides `map()`, `flatMap()`, `filter()`, and `orElse()` methods that allow you to chain operations without null checks. This replaces nested `if (x != null)` blocks with clean, declarative pipelines.

### Creating Optional Instances

| Method | Description | When to Use |
|--------|-------------|-------------|
| `Optional.empty()` | Returns an empty Optional | When there is no value |
| `Optional.of(T value)` | Returns an Optional with the given non-null value | When the value is guaranteed non-null. Throws `NullPointerException` if value is null. |
| `Optional.ofNullable(T value)` | Returns an Optional with the value if non-null, or empty if null | When the value might be null (e.g., from a legacy API or database query) |

```java
import java.util.Optional;

Optional<String> empty = Optional.empty();
Optional<String> present = Optional.of("Hello");  // "Hello" is non-null
Optional<String> maybe = Optional.ofNullable(null);  // Returns empty, no exception
```

### Checking and Extracting Values

| Method | Description | Risk |
|--------|-------------|------|
| `isPresent()` | Returns true if a value is present | Safe, but leads to verbose if-else patterns |
| `isEmpty()` | Returns true if no value is present (Java 11+) | Safe |
| `get()` | Returns the value if present | **DANGEROUS**: throws `NoSuchElementException` if empty. Avoid unless you have already checked `isPresent()`. |
| `orElse(T other)` | Returns the value if present, otherwise returns `other` | Safe. The `other` value is always evaluated. |
| `orElseGet(Supplier<T>)` | Returns the value if present, otherwise calls the supplier | Safe. The supplier is called only if empty (lazy evaluation). |
| `orElseThrow()` | Returns the value if present, otherwise throws `NoSuchElementException` (Java 10+) | Safe if absence is truly exceptional. |
| `orElseThrow(Supplier<X>)` | Returns the value if present, otherwise throws the supplied exception | Safe. Preferred for domain-specific exceptions. |
| `ifPresent(Consumer<T>)` | Executes the consumer if a value is present | Safe. Does nothing if empty. |
| `ifPresentOrElse(Consumer<T>, Runnable)` | Executes the consumer if present, otherwise executes the runnable (Java 9+) | Safe. Handles both cases. |

### Transforming Optional Values

| Method | Description | Analogy |
|--------|-------------|---------|
| `map(Function<T, R>)` | Applies the function to the value if present, wraps the result in a new Optional | Like `Stream.map()` but for a single element |
| `flatMap(Function<T, Optional<R>>)` | Applies the function that returns an Optional, flattens the result | Like `Stream.flatMap()` but for a single element |
| `filter(Predicate<T>)` | Returns the Optional if the value matches the predicate, otherwise returns empty | Like `Stream.filter()` but for a single element |
| `stream()` | Returns a Stream of zero or one elements (Java 9+) | Bridges Optional to the Streams API |

### `orElse()` vs `orElseGet()`

This is the most commonly misunderstood distinction:

- **`orElse(value)`**: The `value` argument is **always evaluated**, even if the Optional is present. Use this for cheap, pre-computed defaults.

- **`orElseGet(supplier)`**: The `supplier` is called **only if the Optional is empty**. Use this for expensive defaults (database queries, object creation, API calls).

```java
import java.util.Optional;

Optional<String> present = Optional.of("Saad");

// orElse: "default" is always created, even though it is not used
String result1 = present.orElse(expensiveDefault());  // expensiveDefault() IS called!

// orElseGet: the supplier is NOT called because the Optional is present
String result2 = present.orElseGet(() -> expensiveDefault());  // expensiveDefault() is NOT called
```

### Optional and Streams (Java 9+)

The `stream()` method converts an `Optional` into a `Stream` of zero or one elements. This is useful for integrating Optional results into stream pipelines.

```java
import java.util.List;
import java.util.Optional;

List<Long> userIds = List.of(1L, 2L, 3L, 999L);

List<User> foundUsers = userIds.stream()
    .map(userRepository::findById)      // Stream<Optional<User>>
    .flatMap(Optional::stream)          // Stream<User> (empties are filtered out)
    .toList();
```

### How Optional Works Internally

`Optional` is a simple final class with a single private final field:

```java
public final class Optional<T> {
    private static final Optional<?> EMPTY = new Optional<>(null);
    private final T value;  // null means empty

    private Optional(T value) {
        this.value = value;
    }

    public static <T> Optional<T> of(T value) {
        return new Optional<>(Objects.requireNonNull(value));
    }

    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
    // ... other methods
}
```

**Memory overhead**: An `Optional` object adds approximately 16 bytes of overhead (object header + reference field) compared to a raw reference. For most backend use cases, this overhead is negligible. However, you should not use `Optional` as a field type in large collections or high-performance data structures where millions of instances would be created.

> [!tip] Key Insight
> `Optional` is designed primarily as a **return type** for methods that might not find a result. It is NOT designed to be used as a method parameter, a field in a class, or a collection element. Using `Optional` in these contexts adds complexity without benefit. The Java language architect Brian Goetz stated: "Optional was designed to provide a limited mechanism for library method return types where there needed to be a clear way to represent 'no result,' and using null for such was overwhelmingly likely to cause errors."

---

## Syntax and Basic Examples

### Example 1: Creating and checking Optional values

```java
import java.util.Optional;

public class OptionalBasics {
    public static void main(String[] args) {
        // Creating Optionals
        Optional<String> present = Optional.of("Hello");
        Optional<String> empty = Optional.empty();
        Optional<String> fromNull = Optional.ofNullable(null);
        Optional<String> fromValue = Optional.ofNullable("World");

        System.out.println("present.isPresent(): " + present.isPresent());   // true
        System.out.println("empty.isPresent(): " + empty.isPresent());       // false
        System.out.println("fromNull.isEmpty(): " + fromNull.isEmpty());     // true (Java 11+)
        System.out.println("fromValue.isPresent(): " + fromValue.isPresent()); // true

        // Extracting values safely
        System.out.println("orElse: " + empty.orElse("default"));           // default
        System.out.println("orElse: " + present.orElse("default"));         // Hello

        System.out.println("orElseGet: " + empty.orElseGet(() -> "computed"));  // computed
        System.out.println("orElseGet: " + present.orElseGet(() -> "computed")); // Hello

        // orElseThrow
        try {
            String value = empty.orElseThrow(() ->
                new RuntimeException("Value is required"));
        } catch (RuntimeException e) {
            System.out.println("Caught: " + e.getMessage());  // Value is required
        }
    }
}
```

### Example 2: Transforming with map, flatMap, and filter

```java
import java.util.Optional;

public class OptionalTransformations {

    record User(String name, String email, Address address) {}
    record Address(String city, String zipCode) {}

    public static void main(String[] args) {
        User userWithAddress = new User("Saad", "saad@test.com",
            new Address("Dhaka", "1200"));
        User userWithoutAddress = new User("Rahim", "rahim@test.com", null);

        // map(): transform the value if present
        Optional<String> upperName = Optional.of(userWithAddress)
            .map(User::name)
            .map(String::toUpperCase);
        System.out.println("Upper name: " + upperName.orElse("N/A"));  // SAAD

        // map() on empty: returns empty without calling the function
        Optional<String> emptyName = Optional.<User>empty()
            .map(User::name)
            .map(String::toUpperCase);
        System.out.println("Empty name: " + emptyName.orElse("N/A"));  // N/A

        // flatMap(): avoid Optional<Optional<T>> nesting
        // map() would return Optional<Optional<Address>> because getAddress returns Optional
        Optional<String> city = Optional.of(userWithAddress)
            .flatMap(u -> Optional.ofNullable(u.address()))
            .map(Address::city);
        System.out.println("City: " + city.orElse("Unknown"));  // Dhaka

        Optional<String> noCity = Optional.of(userWithoutAddress)
            .flatMap(u -> Optional.ofNullable(u.address()))
            .map(Address::city);
        System.out.println("No city: " + noCity.orElse("Unknown"));  // Unknown

        // filter(): keep the value only if it matches the predicate
        Optional<User> validEmail = Optional.of(userWithAddress)
            .filter(u -> u.email().contains("@"));
        System.out.println("Valid email: " + validEmail.isPresent());  // true

        Optional<User> invalidEmail = Optional.of(userWithAddress)
            .filter(u -> u.email().contains("@gmail"));
        System.out.println("Gmail: " + invalidEmail.isPresent());  // false
    }
}
```

### Example 3: ifPresent and ifPresentOrElse

```java
import java.util.Optional;

public class OptionalActions {
    public static void main(String[] args) {
        Optional<String> found = Optional.of("Saad");
        Optional<String> notFound = Optional.empty();

        // ifPresent(): execute action only if value is present
        found.ifPresent(name -> System.out.println("Found: " + name));
        // Found: Saad

        notFound.ifPresent(name -> System.out.println("Found: " + name));
        // (nothing printed)

        // ifPresentOrElse(): handle both cases (Java 9+)
        found.ifPresentOrElse(
            name -> System.out.println("Welcome, " + name),
            () -> System.out.println("User not found")
        );
        // Welcome, Saad

        notFound.ifPresentOrElse(
            name -> System.out.println("Welcome, " + name),
            () -> System.out.println("User not found")
        );
        // User not found
    }
}
```

### Example 4: Optional with Streams

```java
import java.util.*;
import java.util.stream.*;

public class OptionalWithStreams {

    record Product(Long id, String name, double price) {}

    static Optional<Product> findProduct(Long id) {
        Map<Long, Product> catalog = Map.of(
            1L, new Product(1L, "Laptop", 85000),
            2L, new Product(2L, "Mouse", 1500),
            3L, new Product(3L, "Keyboard", 3200)
        );
        return Optional.ofNullable(catalog.get(id));
    }

    public static void main(String[] args) {
        // Convert Optional to Stream (Java 9+)
        List<Long> productIds = List.of(1L, 2L, 999L, 3L, 888L);

        // flatMap(Optional::stream) filters out empty Optionals
        List<Product> foundProducts = productIds.stream()
            .map(OptionalWithStreams::findProduct)  // Stream<Optional<Product>>
            .flatMap(Optional::stream)              // Stream<Product> (empties removed)
            .toList();

        System.out.println("Found " + foundProducts.size() + " products:");
        foundProducts.forEach(p ->
            System.out.printf("  %s: %,.0f BDT%n", p.name(), p.price())
        );
        // Found 3 products:
        //   Laptop: 85,000 BDT
        //   Mouse: 1,500 BDT
        //   Keyboard: 3,200 BDT

        // or() method: provide a fallback Optional (Java 9+)
        Optional<Product> product = findProduct(999L)
            .or(() -> findProduct(1L));  // Falls back to product 1 if 999 not found
        System.out.println("Fallback: " + product.map(Product::name).orElse("None"));
        // Fallback: Laptop
    }
}
```

### Example 5: Chaining Optional operations

```java
import java.util.Optional;

public class OptionalChaining {

    record Order(Long id, String couponCode, Double discount) {}

    static Optional<String> validateCoupon(String code) {
        if (code != null && code.matches("^[A-Z0-9]{5,10}$")) {
            return Optional.of(code);
        }
        return Optional.empty();
    }

    static Optional<Double> calculateDiscount(String couponCode) {
        return switch (couponCode) {
            case "SAVE10" -> Optional.of(0.10);
            case "SAVE20" -> Optional.of(0.20);
            case "EID50" -> Optional.of(0.50);
            default -> Optional.empty();
        };
    }

    public static void main(String[] args) {
        // Chain: validate coupon -> calculate discount -> apply to total
        double orderTotal = 5000.0;

        double finalPrice = Optional.ofNullable("SAVE20")
            .flatMap(OptionalChaining::validateCoupon)
            .flatMap(OptionalChaining::calculateDiscount)
            .map(discount -> orderTotal * (1 - discount))
            .orElse(orderTotal);  // No discount if any step fails

        System.out.println("Final price: " + finalPrice);  // 4000.0

        // With an invalid coupon
        double noDiscountPrice = Optional.ofNullable("invalid!")
            .flatMap(OptionalChaining::validateCoupon)
            .flatMap(OptionalChaining::calculateDiscount)
            .map(discount -> orderTotal * (1 - discount))
            .orElse(orderTotal);

        System.out.println("No discount price: " + noDiscountPrice);  // 5000.0

        // With a null coupon
        double nullCouponPrice = Optional.<String>ofNullable(null)
            .flatMap(OptionalChaining::validateCoupon)
            .flatMap(OptionalChaining::calculateDiscount)
            .map(discount -> orderTotal * (1 - discount))
            .orElse(orderTotal);

        System.out.println("Null coupon price: " + nullCouponPrice);  // 5000.0
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> `Optional` is deeply integrated into Spring Boot's data access layer. Here are three realistic scenarios.

### Scenario 1: Spring Data JPA repository lookups

Spring Data JPA returns `Optional<T>` from all single-entity lookup methods. This is the most common use of `Optional` in backend development.

```java
package com.company.orderservice.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface OrderRepository extends JpaRepository<Order, Long> {

    // Inherited from CrudRepository:
    // Optional<Order> findById(Long id);

    // Custom single-result queries also return Optional:
    Optional<Order> findByOrderNumber(String orderNumber);

    Optional<Order> findByUserIdAndStatus(Long userId, OrderStatus status);
}
```

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.util.Optional;

@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    // Pattern 1: orElseThrow for required resources
    public OrderResponse getOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new ResourceNotFoundException("Order", orderId));
        return OrderResponse.fromEntity(order);
    }

    // Pattern 2: orElseThrow with a different exception for unique lookups
    public OrderResponse getOrderByNumber(String orderNumber) {
        Order order = orderRepository.findByOrderNumber(orderNumber)
            .orElseThrow(() -> new ResourceNotFoundException("Order", "orderNumber", orderNumber));
        return OrderResponse.fromEntity(order);
    }

    // Pattern 3: ifPresentOrElse for conditional logic
    public void cancelOrderIfPending(Long orderId) {
        orderRepository.findById(orderId)
            .ifPresentOrElse(
                order -> {
                    if (order.getStatus() == OrderStatus.PENDING) {
                        order.cancel();
                        orderRepository.save(order);
                    } else {
                        throw new OrderStateException(orderId, order.getStatus().name(), "cancel");
                    }
                },
                () -> {
                    throw new ResourceNotFoundException("Order", orderId);
                }
            );
    }

    // Pattern 4: map + orElse for safe transformation
    public BigDecimal getOrderTotal(Long orderId) {
        return orderRepository.findById(orderId)
            .map(Order::getTotalAmount)
            .orElse(BigDecimal.ZERO);
    }

    // Pattern 5: flatMap for chained lookups
    public Optional<User> getOrderOwner(Long orderId) {
        return orderRepository.findById(orderId)
            .map(Order::getUserId)
            .flatMap(userRepository::findById);
    }
}
```

**What to notice:**

- `orElseThrow()` is the most common pattern for required resources. If the order must exist, throw a `ResourceNotFoundException` when it does not. The global exception handler converts this to a 404 HTTP response.
- `ifPresentOrElse()` handles both the "found" and "not found" cases in a single expression, replacing the verbose `if (optional.isPresent()) { ... } else { ... }` pattern.
- `map()` transforms the wrapped value without unwrapping it. `orderRepository.findById(orderId).map(Order::getTotalAmount)` returns `Optional<BigDecimal>`, not `BigDecimal`. The `orElse()` at the end provides the default.
- `flatMap()` chains multiple Optional-returning operations without nesting. `findById().map(getUserId).flatMap(userRepository::findById)` reads cleanly and short-circuits if any step returns empty.

### Scenario 2: Configuration and feature flags

```java
package com.company.orderservice.config;

import org.springframework.stereotype.Service;
import java.util.Map;
import java.util.Optional;

@Service
public class FeatureFlagService {

    private final Map<String, Boolean> flags;

    public FeatureFlagService(AppProperties properties) {
        this.flags = properties.getFeatureFlags();
    }

    // Returns Optional because a flag might not be configured
    public Optional<Boolean> getFlag(String featureName) {
        return Optional.ofNullable(flags.get(featureName));
    }

    // Convenience method with a default value
    public boolean isEnabled(String featureName) {
        return getFlag(featureName).orElse(false);  // Default: disabled
    }

    // Conditional execution based on feature flag
    public void executeIfEnabled(String featureName, Runnable action) {
        getFlag(featureName)
            .filter(enabled -> enabled)  // Only proceed if the flag is true
            .ifPresent(enabled -> action.run());
    }
}
```

```java
// Usage in a service:
import org.springframework.stereotype.Service;
import java.math.BigDecimal;

@Service
public class PricingService {

    private final FeatureFlagService featureFlags;

    public BigDecimal calculatePrice(Order order) {
        BigDecimal basePrice = calculateBasePrice(order);

        // Apply holiday discount only if the feature flag is enabled
        if (featureFlags.isEnabled("holiday_discount")) {
            basePrice = basePrice.multiply(new BigDecimal("0.85"));
        }

        // Apply new pricing algorithm if the flag is set
        return featureFlags.getFlag("new_pricing_v2")
            .filter(enabled -> enabled)
            .map(enabled -> applyNewPricing(basePrice, order))
            .orElse(basePrice);
    }
}
```

**What to notice:**

- `Optional.ofNullable()` bridges the gap between legacy APIs that return null (like `Map.get()`) and the Optional-based API. This is the most common use of `ofNullable()` in backend code.
- `filter(enabled -> enabled)` keeps the Optional only if the flag is `true`. If the flag is `false` or absent, the chain falls through to `orElse()`.
- The `isEnabled()` convenience method provides a simple boolean for callers that do not need the full Optional API. This is a common pattern: expose both the Optional version (for chaining) and the boolean version (for simple checks).

### Scenario 3: Safe navigation through nested objects

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.util.Optional;

@Service
public class ShippingService {

    // Without Optional: nested null checks (the "pyramid of doom")
    public String getShippingCity_OLD(Order order) {
        if (order != null) {
            if (order.getShippingAddress() != null) {
                if (order.getShippingAddress().getCity() != null) {
                    return order.getShippingAddress().getCity();
                }
            }
        }
        return "Unknown";
    }

    // With Optional: clean, flat chain
    public String getShippingCity(Order order) {
        return Optional.ofNullable(order)
            .map(Order::getShippingAddress)
            .map(Address::getCity)
            .orElse("Unknown");
    }

    // More complex: extract the zip code and validate it
    public Optional<String> getValidZipCode(Order order) {
        return Optional.ofNullable(order)
            .map(Order::getShippingAddress)
            .map(Address::getZipCode)
            .filter(zip -> zip.matches("^\\d{4}$"));  // Bangladeshi zip: 4 digits
    }

    // Chaining multiple Optional lookups
    public Optional<String> getCustomerLoyaltyTier(Long orderId) {
        return orderRepository.findById(orderId)
            .map(Order::getUserId)
            .flatMap(userRepository::findById)
            .map(User::getLoyaltyAccount)
            .map(LoyaltyAccount::getTier);
    }
}
```

**What to notice:**

- The `map()` chain replaces nested `if (x != null)` blocks with a flat, readable pipeline. If any step returns null, the chain short-circuits and returns `Optional.empty()`.
- `filter()` adds validation to the chain. The zip code is only returned if it matches the expected format.
- The `getCustomerLoyaltyTier()` method chains four lookups (order -> user -> loyalty account -> tier) using `map()` and `flatMap()`. Each step might return empty, and the chain handles all cases gracefully without a single null check.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Calling `get()` without checking `isPresent()`

**Wrong:**

```java
Optional<User> user = userRepository.findById(userId);
String name = user.get().getName();  // NoSuchElementException if user is empty!
// This is WORSE than a NullPointerException because it is less informative.
```

**Right:**

```java
// Option 1: orElseThrow (preferred for required resources)
User user = userRepository.findById(userId)
    .orElseThrow(() -> new ResourceNotFoundException("User", userId));
String name = user.getName();

// Option 2: orElse (for optional resources with a default)
String name = userRepository.findById(userId)
    .map(User::getName)
    .orElse("Anonymous");

// Option 3: ifPresent (for side effects)
userRepository.findById(userId)
    .ifPresent(user -> sendWelcomeEmail(user));
```

**Why it is wrong:** Calling `get()` on an empty Optional throws `NoSuchElementException`, which is just as bad as a `NullPointerException`. The entire purpose of Optional is to avoid unchecked access to absent values. If you find yourself calling `get()`, you are using Optional incorrectly. Use `orElse()`, `orElseThrow()`, `ifPresent()`, or `map()` instead.

### Mistake 2: Using Optional as a method parameter

**Wrong:**

```java
public List<Order> getOrders(Optional<Long> userId, Optional<String> status) {
    // Now the caller has to wrap every argument in Optional.ofNullable()
    // This makes the API harder to use, not easier.
}

// Caller:
getOrders(Optional.ofNullable(userId), Optional.ofNullable(status));  // Verbose!
```

**Right:**

```java
// Use null for optional parameters (or method overloading)
public List<Order> getOrders(Long userId, String status) {
    if (userId != null && status != null) {
        return orderRepository.findByUserIdAndStatus(userId, OrderStatus.valueOf(status));
    }
    if (userId != null) {
        return orderRepository.findByUserId(userId);
    }
    return orderRepository.findAll();
}

// Caller:
getOrders(userId, status);  // Clean and simple
```

**Why it is wrong:** Optional was designed as a return type, not a parameter type. Using Optional as a parameter forces every caller to wrap their values in `Optional.ofNullable()`, adding verbosity without benefit. For optional parameters, use method overloading, nullable parameters, or the Builder pattern.

### Mistake 3: Using Optional as a class field

**Wrong:**

```java
@Entity
public class Order {
    private Optional<Address> shippingAddress;  // BAD!
    // JPA cannot persist Optional fields.
    // Serialization frameworks (Jackson) do not handle Optional fields well.
    // Optional is not Serializable, which breaks caching and clustering.
}
```

**Right:**

```java
@Entity
public class Order {
    private Address shippingAddress;  // Use null for absent values in fields

    // Return Optional from the getter if you want to signal optionality to callers
    public Optional<Address> getShippingAddress() {
        return Optional.ofNullable(shippingAddress);
    }
}
```

**Why it is wrong:** Optional is not designed for fields. It is not `Serializable`, which breaks JPA persistence, JSON serialization, and distributed caching. It adds memory overhead to every object. It complicates `equals()` and `hashCode()` implementations. Use null for fields and return Optional from getters if you want to communicate optionality.

### Mistake 4: Using `isPresent()` + `get()` instead of functional methods

**Wrong:**

```java
Optional<User> userOpt = userRepository.findById(userId);
String email;
if (userOpt.isPresent()) {
    email = userOpt.get().getEmail();
} else {
    email = "unknown@example.com";
}
```

**Right:**

```java
String email = userRepository.findById(userId)
    .map(User::getEmail)
    .orElse("unknown@example.com");
```

**Why it is wrong:** The `isPresent()` + `get()` pattern is the imperative style that Optional was designed to replace. It is more verbose, harder to read, and provides no advantage over a plain null check. The functional methods (`map`, `flatMap`, `filter`, `orElse`, `ifPresent`) express the same logic more concisely and declaratively. If your Optional usage looks like a null check, you are not getting the benefits of Optional.

---

## Key Takeaways

> [!tip] Remember these points
> 1. `Optional<T>` is a container that may or may not hold a non-null value. It makes the absence of a value **explicit in the type system**, forcing callers to handle the "not found" case consciously.
> 2. Use `Optional` primarily as a **method return type** for operations that might not find a result (repository lookups, searches, configuration values). Do NOT use it as a method parameter, class field, or collection element.
> 3. Create Optionals with `Optional.of()` (non-null guaranteed), `Optional.ofNullable()` (might be null), or `Optional.empty()` (definitely absent). Never use `new Optional<>()`.
> 4. Extract values with `orElse()` (cheap default), `orElseGet()` (expensive default), `orElseThrow()` (absence is an error), or `ifPresent()` (side effects). **Never call `get()` without first checking `isPresent()`**, and prefer the functional methods over `isPresent()` + `get()`.
> 5. Transform Optional values with `map()` (one-to-one), `flatMap()` (one-to-Optional), and `filter()` (conditional). Chain these methods to replace nested null checks with clean, flat pipelines. Use `Optional.stream()` (Java 9+) to integrate with the Streams API.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Optional Basics (Easy)

Write a program that demonstrates all ways to create and extract values from Optional:

1. Create an `Optional<String>` with a value using `of()`.
2. Create an empty `Optional<String>` using `empty()`.
3. Create an `Optional<String>` from a nullable variable using `ofNullable()`.
4. Extract values using `orElse()`, `orElseGet()`, and `orElseThrow()`.
5. Use `ifPresent()` and `ifPresentOrElse()` to perform actions.

Print the results of each operation with descriptive labels.

> **Hint:** For `orElseGet()`, use a lambda that prints a message to demonstrate lazy evaluation. Compare with `orElse()` to show that the argument is always evaluated.

### Exercise 2: Optional Chaining for Safe Navigation (Medium)

Create a nested object structure: `Company` -> `Department` -> `Team` -> `Employee`, where any level might be null. Write two versions of a method that extracts the employee's name:

1. The old way: nested `if (x != null)` checks.
2. The Optional way: a flat chain of `map()` and `flatMap()` calls.

Test both versions with fully populated objects and with nulls at each level. Verify that both produce the same results.

> **Hint:** Use `Optional.ofNullable()` at the top level and `map()` for each navigation step. If any field returns an `Optional` itself, use `flatMap()` to avoid nesting.

### Exercise 3: Optional with Spring Data Pattern (Medium)

Simulate a Spring Data repository pattern:

1. Create a `UserRepository` class with an internal `Map<Long, User>` and a method `Optional<User> findById(Long id)`.
2. Create a `UserService` class with methods:
   - `getUser(Long id)`: returns the user or throws `ResourceNotFoundException`.
   - `getUserEmail(Long id)`: returns the email or "unknown@example.com".
   - `getActiveUser(Long id)`: returns the user only if `isActive` is true.
   - `getUserDepartment(Long id)`: chains `findById` with a department lookup using `flatMap()`.
3. Test all methods with existing and non-existing user IDs.

> **Hint:** Use `orElseThrow()` for required lookups, `map().orElse()` for safe transformations, and `filter()` for conditional presence.

### Exercise 4: Optional Pipeline for Order Processing (Hard, Optional)

Build an order processing pipeline that uses Optional at every step:

1. `findOrder(String orderNumber)` -> `Optional<Order>`
2. `validateOrder(Order)` -> `Optional<Order>` (empty if the order is cancelled or expired)
3. `findCustomer(Long customerId)` -> `Optional<Customer>`
4. `calculateDiscount(Customer, Order)` -> `Optional<BigDecimal>` (empty if no discount applies)
5. `processPayment(Order, BigDecimal discount)` -> `Optional<PaymentResult>`

Chain all steps using `flatMap()` and `map()`. If any step returns empty, the entire pipeline should return empty without executing subsequent steps. Use `orElseThrow()` at the end to convert the final empty Optional into a meaningful exception.

> **Hint:** The pipeline should look like: `findOrder(number).flatMap(this::validateOrder).flatMap(order -> findCustomer(order.getUserId()).map(customer -> ...))`. Each `flatMap()` short-circuits if the previous step returned empty.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.Optional;

public class Main {
    public static void main(String[] args) {
        Optional<String> present = Optional.of("Hello");
        Optional<String> empty = Optional.empty();
        Optional<String> fromNull = Optional.ofNullable(null);

        System.out.println("orElse (present): " + present.orElse("default"));
        System.out.println("orElse (empty): " + empty.orElse("default"));

        System.out.println("orElseGet (present): " + present.orElseGet(() -> {
            System.out.println("  [Supplier called]");
            return "computed";
        }));
        System.out.println("orElseGet (empty): " + empty.orElseGet(() -> {
            System.out.println("  [Supplier called]");
            return "computed";
        }));

        present.ifPresentOrElse(
            v -> System.out.println("Present: " + v),
            () -> System.out.println("Empty!")
        );

        empty.ifPresentOrElse(
            v -> System.out.println("Present: " + v),
            () -> System.out.println("Empty!")
        );
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
import java.util.*;

record User(Long id, String name, String email, boolean active, Long departmentId) {}
record Department(Long id, String name) {}

class UserRepository {
    private final Map<Long, User> store = Map.of(
        1L, new User(1L, "Saad", "saad@test.com", true, 10L),
        2L, new User(2L, "Rahim", "rahim@test.com", false, 20L)
    );
    Optional<User> findById(Long id) { return Optional.ofNullable(store.get(id)); }
}

class DepartmentRepository {
    private final Map<Long, Department> store = Map.of(
        10L, new Department(10L, "Engineering")
    );
    Optional<Department> findById(Long id) { return Optional.ofNullable(store.get(id)); }
}

class UserService {
    private final UserRepository userRepo = new UserRepository();
    private final DepartmentRepository deptRepo = new DepartmentRepository();

    User getUser(Long id) {
        return userRepo.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found: " + id));
    }

    String getUserEmail(Long id) {
        return userRepo.findById(id)
            .map(User::email)
            .orElse("unknown@example.com");
    }

    Optional<User> getActiveUser(Long id) {
        return userRepo.findById(id)
            .filter(User::active);
    }

    Optional<String> getUserDepartment(Long id) {
        return userRepo.findById(id)
            .map(User::departmentId)
            .flatMap(deptRepo::findById)
            .map(Department::name);
    }
}

public class Main {
    public static void main(String[] args) {
        UserService service = new UserService();
        System.out.println("Email 1: " + service.getUserEmail(1L));
        System.out.println("Email 99: " + service.getUserEmail(99L));
        System.out.println("Active 1: " + service.getActiveUser(1L).isPresent());
        System.out.println("Active 2: " + service.getActiveUser(2L).isPresent());
        System.out.println("Dept 1: " + service.getUserDepartment(1L).orElse("N/A"));
        System.out.println("Dept 2: " + service.getUserDepartment(2L).orElse("N/A"));
    }
}
```

</details>

---

## Related Notes

- [[Java - Java 8 Streams API]]
- [[Java - Java 8 Lambdas and Functional Interfaces]]
- [[Java - File I-O - FileReader FileWriter BufferedReader]]
- [[Java - Date and Time API - LocalDate LocalDateTime]] (next note)

---

## Resources

- [Oracle Java Documentation: Optional](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html) - Complete API reference for all Optional methods.
- [Oracle Java Tutorials: Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) - Official documentation with usage examples.
- [Baeldung: Java Optional](https://www.baeldung.com/java-optional) - Comprehensive guide covering all Optional operations and best practices.
- [Baeldung: Java Optional orElse vs orElseGet](https://www.baeldung.com/java-optional-or-else-vs-or-else-get) - Detailed comparison with performance implications.
- [Baeldung: Java 9 Optional Improvements](https://www.baeldung.com/java-9-optional) - Guide to `ifPresentOrElse()`, `or()`, and `stream()` methods.
- [Effective Java by Joshua Bloch - Item 55: Return Optionals Judiciously](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive guide on when to use Optional and when not to. Essential reading.
- [Brian Goetz: Optional - The Mother of All Bikesheds](https://www.youtube.com/watch?v=Ej0sss6cq14) - The Java language architect explains the design intent behind Optional. Highly recommended video.
