---
title: "Java - Final Keyword"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - final
  - immutability
status: "not-started"
---

# Java - Final Keyword

> [!abstract] Overview
> The `final` keyword in Java is a restriction modifier that can be applied to variables, methods, and classes. A `final` variable cannot be reassigned after initialization. A `final` method cannot be overridden by subclasses. A `final` class cannot be subclassed. In backend development, `final` is the primary tool for enforcing immutability, securing dependency injection in Spring Boot, preventing accidental modification of configuration values, and communicating design intent to other developers. Understanding `final` deeply is essential for writing thread-safe, maintainable backend code.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]
> - [[Java - Classes and Objects]]
> - [[Java - Constructors - Default Parameterized Copy]]
> - [[Java - Inheritance - Single Multilevel Hierarchical]]
> - [[Java - Static Keyword - Variables Methods Blocks]]

---

## Theory

### What Does `final` Mean?

The `final` keyword means "this cannot change." It is a promise to the compiler, the JVM, and other developers that a particular element is fixed after initialization. The specific meaning depends on what `final` is applied to:

| Applied To | Meaning |
|-----------|---------|
| **Local variable** | The variable cannot be reassigned after its first assignment. |
| **Instance field** | The field must be assigned exactly once (in the declaration or constructor) and cannot be changed afterward. |
| **Static field** | The field must be assigned once (in the declaration or static block) and cannot be changed afterward. Combined with `static`, this creates a constant. |
| **Method parameter** | The parameter cannot be reassigned within the method body. |
| **Method** | The method cannot be overridden by subclasses. |
| **Class** | The class cannot be extended (subclassed). |

The `final` keyword is not just a compiler restriction. It communicates **design intent**. When a developer sees `final`, they immediately know that the value, behavior, or type hierarchy is intentionally fixed. This reduces cognitive load and prevents an entire category of bugs.

### Final Variables

**Final local variables:**

A final local variable must be assigned exactly once. After assignment, any attempt to reassign it causes a compilation error.

```java
import java.math.BigDecimal;

public void processOrder(BigDecimal amount) {
    final double TAX_RATE = 0.15;
    final BigDecimal tax = amount.multiply(BigDecimal.valueOf(TAX_RATE));

    TAX_RATE = 0.20;  // COMPILATION ERROR: cannot assign a value to final variable
    tax = new BigDecimal("100");  // COMPILATION ERROR
}
```

**Final instance fields (blank finals):**

A final instance field that is not initialized in its declaration is called a **blank final**. It must be assigned in every constructor of the class. The compiler verifies that every possible constructor path assigns the field exactly once.

```java
import java.math.BigDecimal;
import java.time.LocalDateTime;

public class Order {
    private final String orderNumber;  // Blank final: no initializer
    private final LocalDateTime createdAt;  // Blank final
    private final BigDecimal totalAmount;  // Blank final

    // Constructor 1: assigns all blank finals
    public Order(String orderNumber, BigDecimal totalAmount) {
        this.orderNumber = orderNumber;  // First and only assignment
        this.createdAt = LocalDateTime.now();  // First and only assignment
        this.totalAmount = totalAmount;  // First and only assignment
    }

    // Constructor 2: must ALSO assign all blank finals
    public Order(String orderNumber) {
        this.orderNumber = orderNumber;
        this.createdAt = LocalDateTime.now();
        this.totalAmount = BigDecimal.ZERO;
        // If you forget to assign any final field, COMPILATION ERROR.
    }
}
```

**Final reference variables vs immutable objects:**

This is the most misunderstood aspect of `final`. A `final` reference variable cannot be reassigned to point to a different object, but the **internal state of the object it points to can still change** if the object is mutable.

```java
import java.util.ArrayList;
import java.util.List;

final List<String> items = new ArrayList<>();
items.add("Laptop");  // OK: modifying the object's internal state
items.add("Mouse");   // OK: the reference is final, not the object
items = new ArrayList<>();  // COMPILATION ERROR: cannot reassign the reference

final StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // OK: StringBuilder is mutable
sb = new StringBuilder("Goodbye");  // COMPILATION ERROR
```

`final` guarantees **reference immutability**, not **object immutability**. To make an object truly immutable, you need `final` fields AND a class that does not expose any mutation methods (no setters, no add/remove methods on internal collections).

### Final Methods

A `final` method cannot be overridden by any subclass. The subclass inherits the method and can call it, but it cannot provide its own implementation.

```java
public class BaseEntity {
    private Long id;

    // Final method: subclasses CANNOT override this.
    // The ID generation logic is locked and cannot be changed.
    public final Long getId() {
        return id;
    }

    // Non-final method: subclasses CAN override this.
    public String getDisplayName() {
        return "Entity #" + id;
    }
}

public class Order extends BaseEntity {
    // COMPILATION ERROR: cannot override final method
    // @Override
    // public Long getId() { return 999L; }

    // OK: getDisplayName() is not final
    @Override
    public String getDisplayName() {
        return "Order #" + getId();  // Can call the final method, just not override it
    }
}
```

**When to use final methods:**

- When the method's behavior is critical to the class's correctness and should not be changed by subclasses. For example, a security validation method or a core algorithm.
- When the method is part of a **Template Method** pattern and the algorithm skeleton should not be altered.
- When you want to allow the JIT compiler to inline the method for performance (the JIT can inline final methods more aggressively because it knows they will not be overridden).

**When NOT to use final methods:**

- When you want subclasses to customize the behavior. Making methods final by default prevents extension and violates the Open/Closed Principle.
- On methods in interfaces (interface methods are implicitly non-final unless they are `static` or `private`).

### Final Classes

A `final` class cannot be extended. No class can use `extends` to inherit from a final class.

```java
import java.math.BigDecimal;
import java.time.LocalDateTime;

public final class PaymentReceipt {
    private final String transactionId;
    private final BigDecimal amount;
    private final LocalDateTime timestamp;

    public PaymentReceipt(String transactionId, BigDecimal amount) {
        this.transactionId = transactionId;
        this.amount = amount;
        this.timestamp = LocalDateTime.now();
    }

    // Getters only. No setters. The class is final and immutable.
    public String getTransactionId() { return transactionId; }
    public BigDecimal getAmount() { return amount; }
    public LocalDateTime getTimestamp() { return timestamp; }
}

// COMPILATION ERROR: cannot inherit from final class
// public class RefundReceipt extends PaymentReceipt { }
```

**When to use final classes:**

- **Immutable value objects**: Classes like `String`, `BigDecimal`, `LocalDateTime`, and `Integer` are all final. Making an immutable class final prevents subclasses from adding mutable state, which would break the immutability contract.
- **Security-sensitive classes**: Preventing subclassing stops malicious code from overriding methods to bypass security checks.
- **Utility classes**: Classes with only static methods (like `MathUtils`) should be final to prevent meaningless subclassing.
- **Sealed hierarchies**: When you have enumerated all possible subclasses using `sealed` classes (Java 17+), the leaf classes are typically `final`.

**Famous final classes in the Java standard library:**

| Class | Why it is final |
|-------|----------------|
| `String` | Immutability and String pool security |
| `Integer`, `Long`, `Double`, etc. | Wrapper class immutability |
| `BigDecimal`, `BigInteger` | Numeric precision guarantees |
| `LocalDate`, `LocalTime`, `LocalDateTime` | Date/time immutability |
| `Math` | Utility class, no instances |
| `System` | Security, controls I/O streams |

### Final Parameters

Method parameters can be declared `final`, which prevents reassignment within the method body.

```java
import java.math.BigDecimal;

public BigDecimal calculateDiscount(final BigDecimal amount, final double rate) {
    // amount = new BigDecimal("0");  // COMPILATION ERROR: cannot reassign final parameter
    // rate = 0.5;  // COMPILATION ERROR

    return amount.multiply(BigDecimal.valueOf(rate));
}
```

**When to use final parameters:**

- Some coding standards (like the older Checkstyle rules) require all parameters to be final to prevent accidental reassignment.
- In practice, most modern Java teams do not use final parameters because they add visual noise without significant benefit. The method body should be short enough that reassignment is obvious.
- Final parameters are required when the parameter is used inside an anonymous inner class or lambda expression (though Java 8 relaxed this to "effectively final").

### Effectively Final (Java 8+)

Starting from Java 8, a variable that is not declared `final` but is assigned only once is called **effectively final**. Effectively final variables can be used in lambda expressions and anonymous inner classes, just like explicitly final variables.

```java
public void processOrders(List<Order> orders) {
    double taxRate = 0.15;  // Not declared final, but assigned only once

    // Lambda can access taxRate because it is effectively final
    orders.forEach(order -> {
        double tax = order.getTotal() * taxRate;  // OK
        System.out.println("Tax: " + tax);
    });

    // taxRate = 0.20;  // If you uncomment this, taxRate is no longer effectively final,
    // and the lambda above will cause a COMPILATION ERROR.
}
```

**Why does Java require final/effectively final for lambdas?**

Lambda expressions capture variables from their enclosing scope. The captured variable is copied into the lambda's internal state. If the original variable could change after the lambda is created, the lambda would hold a stale copy, leading to confusing behavior. By requiring the variable to be final (or effectively final), Java guarantees that the captured value is consistent.

### How `final` Works Internally

At the bytecode level, `final` is represented by the `ACC_FINAL` flag in the class file. The JVM and JIT compiler use this flag for several optimizations:

**1. Constant folding**: If a `static final` field is initialized with a compile-time constant (a primitive or a String literal), the compiler replaces all references to the field with the actual value. This is called a **constant variable**.

```java
public static final int MAX_RETRIES = 3;

// The compiler replaces all occurrences of MAX_RETRIES with 3 in the bytecode.
// The field is not even read at runtime.
```

**2. Method inlining**: The JIT compiler can inline `final` methods more aggressively because it knows they will never be overridden. Inlining eliminates the method call overhead and enables further optimizations.

**3. Escape analysis**: When a `final` field is assigned in a constructor, the JVM guarantees that the field's value is visible to all threads after the constructor completes (the **safe publication** guarantee of the Java Memory Model). This is critical for thread-safe immutable objects.

**4. Class loading**: `final` classes cannot have subclasses, so the JVM can skip certain virtual method dispatch checks and devirtualize method calls, improving performance.

> [!tip] Key Insight
> The most important use of `final` in Spring Boot is for **dependency injection fields**. When you declare a service's dependencies as `private final` and inject them through the constructor, you guarantee three things: (1) the dependency is always present (the constructor would fail otherwise), (2) the dependency cannot be accidentally reassigned during the object's lifetime, and (3) the field is safely published to all threads. This is why constructor injection with `final` fields is the recommended pattern in Spring Boot.

---

## Syntax and Basic Examples

### Example 1: Final variables in different contexts

```java
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

public class FinalDemo {

    // Final static field (constant): assigned once, shared by all instances
    public static final String APP_NAME = "OrderService";
    public static final int MAX_RETRY = 3;

    // Final instance field (blank final): assigned in constructor
    private final String instanceId;

    // Final instance field with initializer: assigned at declaration
    private final LocalDateTime createdAt = LocalDateTime.now();

    public FinalDemo(String instanceId) {
        this.instanceId = instanceId;  // Blank final: must be assigned here
        // this.createdAt = LocalDateTime.now();  // COMPILATION ERROR: already initialized
    }

    public void demonstrate() {
        // Final local variable
        final double PI = 3.14159;
        // PI = 3.14;  // COMPILATION ERROR

        // Final reference variable: the reference is fixed, the object is mutable
        final List<String> items = new ArrayList<>();
        items.add("Laptop");  // OK: modifying the list's contents
        items.add("Mouse");   // OK
        // items = new ArrayList<>();  // COMPILATION ERROR: cannot reassign the reference

        System.out.println("App: " + APP_NAME);
        System.out.println("Instance: " + instanceId);
        System.out.println("Created: " + createdAt);
        System.out.println("Items: " + items);
    }
}
```

### Example 2: Final methods and final classes

```java
public class SecurityValidator {

    // Final method: the validation logic cannot be overridden by subclasses.
    // This prevents a malicious subclass from weakening the security checks.
    public final boolean isValidToken(String token) {
        if (token == null || token.length() < 32) {
            return false;
        }
        // Complex token validation logic...
        return token.startsWith("Bearer ");
    }

    // Non-final method: subclasses can customize the error message
    public String getErrorMessage() {
        return "Invalid authentication token";
    }
}
```

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.Objects;

// Final class: cannot be subclassed.
// This ensures that Money objects are always immutable and behave consistently.
public final class Money {
    private final BigDecimal amount;
    private final String currency;

    public Money(BigDecimal amount, String currency) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        this.amount = amount.setScale(2, RoundingMode.HALF_UP);
        this.currency = currency.toUpperCase();
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }

    public Money subtract(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot subtract different currencies");
        }
        return new Money(this.amount.subtract(other.amount), this.currency);
    }

    public BigDecimal getAmount() { return amount; }
    public String getCurrency() { return currency; }

    @Override
    public String toString() {
        return amount + " " + currency;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Money money)) return false;
        return amount.compareTo(money.amount) == 0 && currency.equals(money.currency);
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }
}
```

### Example 3: Final fields for dependency injection (Spring Boot pattern)

```java
@Service
public class OrderService {

    // All dependencies are private final.
    // They are assigned once in the constructor and never change.
    // This is the recommended Spring Boot pattern.
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final EmailService emailService;
    private final InventoryService inventoryService;

    // Constructor injection: Spring calls this once at startup.
    // The final keyword guarantees these fields are never null
    // and never reassigned during the application's lifetime.
    public OrderService(OrderRepository orderRepository,
                        PaymentService paymentService,
                        EmailService emailService,
                        InventoryService inventoryService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.emailService = emailService;
        this.inventoryService = inventoryService;
    }

    public Order placeOrder(CreateOrderRequest request) {
        Order order = new Order(request);
        inventoryService.reserveStock(order.getItems());
        Order savedOrder = orderRepository.save(order);
        paymentService.charge(savedOrder);
        emailService.sendConfirmation(savedOrder);
        return savedOrder;
    }
}
```

### Example 4: Building a truly immutable class

```java
import java.time.LocalDateTime;
import java.util.List;

// A truly immutable class requires ALL of the following:
// 1. The class is final (cannot be subclassed to add mutable state)
// 2. All fields are private and final (cannot be reassigned)
// 3. No setter methods (no way to modify state after construction)
// 4. Defensive copies for mutable fields (callers cannot modify internal state)
// 5. The constructor fully initializes all fields

public final class CustomerProfile {
    private final Long customerId;
    private final String name;
    private final String email;
    private final List<String> preferredCategories;  // Mutable type, needs defensive copy
    private final LocalDateTime memberSince;

    public CustomerProfile(Long customerId, String name, String email,
                           List<String> preferredCategories) {
        this.customerId = customerId;
        this.name = name;
        this.email = email;
        // Defensive copy: the caller's list might change after construction.
        // We store our own independent copy.
        this.preferredCategories = List.copyOf(preferredCategories);
        this.memberSince = LocalDateTime.now();
    }

    // Getters only. No setters.

    public Long getCustomerId() { return customerId; }
    public String getName() { return name; }
    public String getEmail() { return email; }

    // Defensive copy in the getter: return an unmodifiable view.
    // If we returned the internal list directly, the caller could modify it.
    public List<String> getPreferredCategories() {
        return preferredCategories;  // Already unmodifiable from List.copyOf()
    }

    public LocalDateTime getMemberSince() { return memberSince; }

    // "Modification" methods return NEW objects instead of changing this one.
    // This is the functional approach to immutability.
    public CustomerProfile withName(String newName) {
        return new CustomerProfile(this.customerId, newName, this.email, this.preferredCategories);
    }

    public CustomerProfile withEmail(String newEmail) {
        return new CustomerProfile(this.customerId, this.name, newEmail, this.preferredCategories);
    }
}
```

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> categories = new ArrayList<>(List.of("Electronics", "Books"));
        CustomerProfile profile = new CustomerProfile(1L, "Saad", "saad@example.com", categories);

        // Modifying the original list does NOT affect the profile
        categories.add("Clothing");
        System.out.println("Profile categories: " + profile.getPreferredCategories());
        // [Electronics, Books] (unchanged)

        // "Modifying" the profile creates a new object
        CustomerProfile updated = profile.withName("Abdullah Al Sayb Saad");
        System.out.println("Original name: " + profile.getName());  // Saad
        System.out.println("Updated name: " + updated.getName());   // Abdullah Al Sayb Saad
        System.out.println("Same object? " + (profile == updated)); // false
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> The `final` keyword is pervasive in production Spring Boot codebases. Here are three realistic scenarios.

### Scenario 1: Immutable DTOs with Java Records

Java records (Java 14+) are implicitly final classes with final fields. They are the modern way to create immutable DTOs in Spring Boot.

```java
package com.company.orderservice.dto;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

// A record is implicitly:
// - final (cannot be subclassed)
// - All fields are private final
// - Constructor assigns all fields
// - Getters are generated (without "get" prefix)
// - equals(), hashCode(), toString() are generated
public record OrderResponse(
    Long id,
    String orderNumber,
    BigDecimal totalAmount,
    String status,
    List<OrderItemResponse> items,
    LocalDateTime createdAt
) {
    // Compact constructor for validation and defensive copying
    public OrderResponse {
        if (orderNumber == null || orderNumber.isBlank()) {
            throw new IllegalArgumentException("Order number is required");
        }
        // Defensive copy of the mutable list
        items = items != null ? List.copyOf(items) : List.of();
    }

    // Static factory method: converts an entity to a DTO
    public static OrderResponse fromEntity(Order order) {
        return new OrderResponse(
            order.getId(),
            order.getOrderNumber(),
            order.getTotalAmount(),
            order.getStatus().name(),
            order.getItems().stream()
                .map(OrderItemResponse::fromEntity)
                .toList(),
            order.getCreatedAt()
        );
    }
}

// Nested record for order items
record OrderItemResponse(
    Long productId,
    String productName,
    int quantity,
    BigDecimal unitPrice,
    BigDecimal subtotal
) {
    public static OrderItemResponse fromEntity(OrderItem item) {
        return new OrderItemResponse(
            item.getProduct().getId(),
            item.getProduct().getName(),
            item.getQuantity(),
            item.getUnitPrice(),
            item.getUnitPrice().multiply(BigDecimal.valueOf(item.getQuantity()))
        );
    }
}
```

**What to notice:**

- Records are `final` by default. You cannot extend a record. This guarantees that the DTO's structure is fixed and cannot be altered by subclasses.
- All record fields are `private final`. Once the record is constructed, its fields cannot change. This makes records inherently thread-safe.
- The compact constructor validates input and creates defensive copies of mutable fields (the `List`). This ensures that the record is truly immutable, not just reference-immutable.

### Scenario 2: Immutable configuration properties

Spring Boot configuration classes use `final` fields to ensure that configuration values cannot be changed after startup.

```java
package com.company.orderservice.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;

@ConfigurationProperties(prefix = "app.payment")
@ConstructorBinding  // Tells Spring to use the constructor for binding
public final class PaymentProperties {

    private final String gatewayUrl;
    private final String apiKey;
    private final int timeoutSeconds;
    private final int maxRetries;
    private final boolean sandboxMode;

    // All fields are final. Spring calls this constructor once at startup.
    // After construction, the configuration cannot change.
    // This prevents accidental modification during request processing.
    public PaymentProperties(String gatewayUrl, String apiKey,
                              int timeoutSeconds, int maxRetries,
                              boolean sandboxMode) {
        if (gatewayUrl == null || gatewayUrl.isBlank()) {
            throw new IllegalArgumentException("Payment gateway URL is required");
        }
        if (timeoutSeconds < 1 || timeoutSeconds > 60) {
            throw new IllegalArgumentException("Timeout must be 1-60 seconds");
        }

        this.gatewayUrl = gatewayUrl;
        this.apiKey = apiKey;
        this.timeoutSeconds = timeoutSeconds;
        this.maxRetries = maxRetries;
        this.sandboxMode = sandboxMode;
    }

    public String getGatewayUrl() { return gatewayUrl; }
    public String getApiKey() { return apiKey; }
    public int getTimeoutSeconds() { return timeoutSeconds; }
    public int getMaxRetries() { return maxRetries; }
    public boolean isSandboxMode() { return sandboxMode; }
}
```

**What to notice:**

- The class is `final`. No subclass can override the configuration values or add mutable state.
- All fields are `private final`. Once Spring constructs the object at startup, the configuration is frozen. A service cannot accidentally change the gateway URL during request processing.
- The `@ConstructorBinding` annotation tells Spring to use constructor injection instead of setter injection. This is the immutable configuration pattern recommended by the Spring Boot team.

### Scenario 3: Thread-safe singleton services

In Spring Boot, all `@Service` beans are singletons by default. Making their fields `final` ensures thread safety because the fields are assigned once during construction and never modified.

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class RateLimitService {

    // Final fields: assigned once in the constructor, never reassigned.
    // This makes the service thread-safe for these references.
    private final int maxRequestsPerMinute;
    private final ConcurrentHashMap<String, AtomicLong> requestCounts;
    private final ConcurrentHashMap<String, Long> windowStartTimes;

    public RateLimitService() {
        this.maxRequestsPerMinute = 60;
        this.requestCounts = new ConcurrentHashMap<>();
        this.windowStartTimes = new ConcurrentHashMap<>();
    }

    public boolean isAllowed(String clientId) {
        long now = System.currentTimeMillis();
        long windowStart = windowStartTimes.computeIfAbsent(clientId, k -> now);

        // Reset the window if a minute has passed
        if (now - windowStart > 60_000) {
            windowStartTimes.put(clientId, now);
            requestCounts.put(clientId, new AtomicLong(0));
        }

        AtomicLong count = requestCounts.computeIfAbsent(clientId, k -> new AtomicLong(0));
        return count.incrementAndGet() <= maxRequestsPerMinute;
    }
}
```

**What to notice:**

- The `maxRequestsPerMinute` field is `final`. It is set once and never changes. Multiple threads can read it safely without synchronization.
- The `ConcurrentHashMap` references are `final`. The maps themselves are thread-safe (they handle concurrent access internally), and the `final` keyword ensures that no thread can replace the map with a different instance.
- The combination of `final` references and thread-safe data structures (`ConcurrentHashMap`, `AtomicLong`) makes this service safe for concurrent access without explicit `synchronized` blocks.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Thinking `final` makes objects immutable

**Wrong:**

```java
final List<String> users = new ArrayList<>();
users.add("Alice");
users.add("Bob");
users.clear();  // This works! The list is now empty.
// The 'final' keyword only prevents reassignment of the 'users' variable.
// It does NOT prevent modification of the ArrayList object.
```

**Right:**

```java
// To make the list truly immutable:
final List<String> users = List.of("Alice", "Bob");  // Unmodifiable list
// users.add("Charlie");  // UnsupportedOperationException at runtime

// Or wrap a mutable list:
final List<String> users = Collections.unmodifiableList(new ArrayList<>(List.of("Alice", "Bob")));
```

**Why it is wrong:** `final` on a reference variable means the variable cannot point to a different object. It does not freeze the object's internal state. An `ArrayList` is still mutable even if the reference to it is `final`. To achieve true immutability, you need both `final` references AND immutable objects (no setters, unmodifiable collections, defensive copies).

### Mistake 2: Forgetting to initialize blank final fields in all constructors

**Wrong:**

```java
public class Order {
    private final String orderNumber;
    private final BigDecimal total;

    public Order(String orderNumber, BigDecimal total) {
        this.orderNumber = orderNumber;
        this.total = total;
    }

    public Order(String orderNumber) {
        this.orderNumber = orderNumber;
        // COMPILATION ERROR: final field 'total' is not initialized in this constructor.
        // Every constructor must assign every blank final field.
    }
}
```

**Right:**

```java
public class Order {
    private final String orderNumber;
    private final BigDecimal total;

    public Order(String orderNumber, BigDecimal total) {
        this.orderNumber = orderNumber;
        this.total = total;
    }

    public Order(String orderNumber) {
        this(orderNumber, BigDecimal.ZERO);  // Chain to the full constructor
    }
}
```

**Why it is wrong:** The compiler enforces that every blank final field is assigned in every constructor. If you have multiple constructors, each one must assign all blank finals, either directly or by chaining to another constructor that does. Constructor chaining with `this()` is the cleanest solution.

### Mistake 3: Making everything final without understanding the trade-offs

**Wrong:**

```java
public final class OrderService {
    private final OrderRepository repository;

    public final Order createOrder(final CreateOrderRequest request) {
        final Order order = new Order(request);
        final Order saved = repository.save(order);
        return saved;
    }
}
```

**Right thinking:** Use `final` intentionally, not mechanically. Ask yourself:

- Should this class be subclassed? If yes, do not make it `final`.
- Should this method be overridden? If yes, do not make it `final`.
- Will this variable be reassigned? If no, make it `final`.
- Is this a dependency that should never change? Make it `final`.

**Why it is wrong:** Blindly adding `final` everywhere makes the code rigid and hard to extend. A `final` class cannot be proxied by Spring (which breaks AOP, caching, and transaction management). A `final` method cannot be overridden in tests. Use `final` where it adds value (immutability, thread safety, design intent), not as a default for everything.

### Mistake 4: Not using `final` for Spring-managed dependencies

**Wrong:**

```java
@Service
public class OrderService {
    // Without final: the field can be accidentally reassigned at any point.
    // Without final: Spring can use field injection (@Autowired on the field),
    // which makes the class harder to test.
    private OrderRepository orderRepository;
    private PaymentService paymentService;
}
```

**Right:**

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;

    public OrderService(OrderRepository orderRepository, PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }
}
```

**Why it is wrong:** Without `final`, there is nothing preventing a developer from accidentally writing `this.orderRepository = null` somewhere in the service. The `final` keyword makes this a compilation error. It also forces constructor injection, which makes the class testable (you can pass mock objects to the constructor in unit tests without needing Spring).

---

## Key Takeaways

> [!tip] Remember these points
> 1. `final` on a **variable** means it cannot be reassigned after initialization. For reference types, the reference is fixed but the object's internal state may still be mutable. Use defensive copies and unmodifiable collections for true immutability.
> 2. `final` on a **method** means it cannot be overridden by subclasses. Use it for security-critical methods and Template Method skeletons. Avoid it when you want to allow extension.
> 3. `final` on a **class** means it cannot be subclassed. Use it for immutable value objects, utility classes, and security-sensitive classes. Be aware that Spring cannot proxy final classes for AOP.
> 4. **Blank finals** (final fields without initializers) must be assigned in every constructor. This is the foundation of immutable objects and constructor-based dependency injection in Spring Boot.
> 5. The most important use of `final` in backend development is for **Spring service dependencies**: `private final` fields injected through the constructor. This guarantees non-null dependencies, prevents reassignment, and ensures thread-safe publication.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Immutable Student Record (Easy)

Create a `final` class `Student` with `private final` fields: `studentId` (String), `name` (String), `department` (String), `cgpa` (double). The constructor should validate that `cgpa` is between 0.0 and 4.0 and that `name` is not blank. Provide getters only (no setters). Override `toString()`, `equals()`, and `hashCode()`.

In `main()`, create two `Student` objects with the same data and verify that `equals()` returns `true` and `hashCode()` values match. Try to modify a field and observe the compilation error.

> **Hint:** Use `Objects.hash()` for `hashCode()` and `Objects.equals()` for `equals()`. Remember to compare `cgpa` with a tolerance or use `Double.compare()`.

### Exercise 2: Immutable Money Class with Operations (Medium)

Create a `final` class `Money` with `private final` fields `amount` (BigDecimal) and `currency` (String). Implement:

- Constructor that validates amount is non-negative and currency is a 3-letter uppercase code.
- `add(Money other)`: returns a new `Money` object with the sum. Throws if currencies differ.
- `subtract(Money other)`: returns a new `Money` object with the difference. Throws if the result would be negative.
- `multiply(int quantity)`: returns a new `Money` object.
- `split(int ways)`: returns a `List<Money>` with the amount divided equally.

All methods return **new** `Money` objects. The original object is never modified. This is the functional approach to immutability.

> **Hint:** Use `BigDecimal` methods (`add`, `subtract`, `multiply`, `divide`) which already return new objects. The `split` method should use `divide` with `RoundingMode.HALF_UP` and handle the remainder by adding it to the last share.

### Exercise 3: Final Fields and Thread Safety (Medium)

Create a `Configuration` class that loads settings in a static block and exposes them through final fields:

- `private static final Map<String, String> SETTINGS` initialized in a static block.
- `public static String get(String key)` that returns the value or a default.
- `public static Map<String, String> getAll()` that returns an unmodifiable view.

Prove that the map cannot be modified from outside the class by attempting to call `getAll().put("key", "value")` and catching the `UnsupportedOperationException`.

> **Hint:** Use `Collections.unmodifiableMap()` or `Map.copyOf()` to create the unmodifiable view.

### Exercise 4: Immutable Order with Defensive Copies (Hard, Optional)

Create a `final` class `ImmutableOrder` with fields: `orderId` (long), `customerName` (String), `items` (List of `OrderItem` objects, where `OrderItem` is also a final immutable class with `productName`, `quantity`, `unitPrice`), and `createdAt` (LocalDateTime).

The constructor must:

1. Validate all fields.
2. Make a defensive copy of the `items` list.
3. Make defensive copies of each `OrderItem` if `OrderItem` were mutable (since it is immutable, storing the references is safe, but document this decision in a comment).

Provide a `withStatus(String newStatus)` method that returns a new `ImmutableOrder` with the updated status (add a `status` field). The original order must remain unchanged.

In `main()`, create an order, modify the original items list, and verify that the order's items are unaffected.

> **Hint:** Use `List.copyOf()` for the defensive copy. The `withStatus()` method creates a new `ImmutableOrder` with all the same fields except the status. This is the "wither" pattern used in Java records and functional programming.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.Objects;

public final class Student {
    private final String studentId;
    private final String name;
    private final String department;
    private final double cgpa;

    public Student(String studentId, String name, String department, double cgpa) {
        if (studentId == null || studentId.isBlank()) {
            throw new IllegalArgumentException("Student ID is required");
        }
        if (name == null || name.strip().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (cgpa < 0.0 || cgpa > 4.0) {
            throw new IllegalArgumentException("CGPA must be between 0.0 and 4.0");
        }

        this.studentId = studentId;
        this.name = name.strip();
        this.department = department != null ? department.toUpperCase() : "UNDECLARED";
        this.cgpa = cgpa;
    }

    public String getStudentId() { return studentId; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getCgpa() { return cgpa; }

    @Override
    public String toString() {
        return String.format("Student{id='%s', name='%s', dept='%s', cgpa=%.2f}",
            studentId, name, department, cgpa);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student s)) return false;
        return Double.compare(s.cgpa, cgpa) == 0
            && studentId.equals(s.studentId)
            && name.equals(s.name)
            && department.equals(s.department);
    }

    @Override
    public int hashCode() {
        return Objects.hash(studentId, name, department, cgpa);
    }

    public static void main(String[] args) {
        Student s1 = new Student("230145", "Saad", "CSE", 3.72);
        Student s2 = new Student("230145", "Saad", "CSE", 3.72);

        System.out.println(s1);
        System.out.println("equals: " + s1.equals(s2));
        System.out.println("hashCode match: " + (s1.hashCode() == s2.hashCode()));

        // s1.cgpa = 3.80;  // COMPILATION ERROR: cannot assign final field
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.ArrayList;
import java.util.List;

public final class Money {
    private final BigDecimal amount;
    private final String currency;

    public Money(BigDecimal amount, String currency) {
        if (amount == null || amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount must be non-negative");
        }
        if (currency == null || !currency.matches("[A-Z]{3}")) {
            throw new IllegalArgumentException("Currency must be a 3-letter uppercase code");
        }
        this.amount = amount.setScale(2, RoundingMode.HALF_UP);
        this.currency = currency;
    }

    public Money add(Money other) {
        requireSameCurrency(other);
        return new Money(this.amount.add(other.amount), this.currency);
    }

    public Money subtract(Money other) {
        requireSameCurrency(other);
        BigDecimal result = this.amount.subtract(other.amount);
        if (result.compareTo(BigDecimal.ZERO) < 0) {
            throw new ArithmeticException("Result would be negative");
        }
        return new Money(result, this.currency);
    }

    public Money multiply(int quantity) {
        if (quantity < 0) throw new IllegalArgumentException("Quantity must be non-negative");
        return new Money(this.amount.multiply(BigDecimal.valueOf(quantity)), this.currency);
    }

    public List<Money> split(int ways) {
        if (ways <= 0) throw new IllegalArgumentException("Ways must be positive");
        BigDecimal share = this.amount.divide(BigDecimal.valueOf(ways), 2, RoundingMode.HALF_UP);
        BigDecimal remainder = this.amount.subtract(share.multiply(BigDecimal.valueOf(ways)));

        List<Money> shares = new ArrayList<>();
        for (int i = 0; i < ways; i++) {
            BigDecimal thisShare = (i == ways - 1) ? share.add(remainder) : share;
            shares.add(new Money(thisShare, this.currency));
        }
        return List.copyOf(shares);
    }

    private void requireSameCurrency(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException(
                "Currency mismatch: " + this.currency + " vs " + other.currency);
        }
    }

    public BigDecimal getAmount() { return amount; }
    public String getCurrency() { return currency; }

    @Override
    public String toString() { return amount + " " + currency; }
}
```

</details>

---

## Related Notes

- [[Java - Static Keyword - Variables Methods Blocks]]
- [[Java - Exception Handling - Try Catch Finally Throw Throws]] (next note)
- [[Java - Comparable and Comparator]]

---

## Resources

- [Oracle Java Tutorials: Understanding Instance and Class Members](https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html) - Official documentation covering the `final` keyword in the context of class members.
- [Baeldung: A Guide to the Final Keyword in Java](https://www.baeldung.com/java-final) - Comprehensive guide covering final variables, methods, and classes with examples.
- [Baeldung: Immutable Objects in Java](https://www.baeldung.com/java-immutable-object) - Step-by-step guide to creating truly immutable classes using final.
- [Baeldung: Java Effectively Final](https://www.baeldung.com/java-effectively-final) - Explanation of effectively final variables and their role in lambdas.
- [Effective Java by Joshua Bloch - Item 17: Minimize Mutability](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive guide on when and how to make classes immutable. Essential reading for backend engineers.
- [Java Concurrency in Practice by Brian Goetz - Chapter 3: Sharing Objects](https://www.oreilly.com/library/view/java-concurrency-in/0321349601/) - Explains the safe publication guarantee of final fields in the Java Memory Model.
