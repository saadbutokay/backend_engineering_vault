---
title: "Java - Abstraction - Abstract Classes and Interfaces"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - abstraction
  - abstract-class
  - interface
status: "not-started"
---

# Java - Abstraction - Abstract Classes and Interfaces

> [!abstract] Overview
> Abstraction is the OOP principle of hiding implementation details and exposing only the essential behavior through a contract. In Java, abstraction is achieved through **abstract classes** (classes that cannot be instantiated and may contain abstract methods) and **interfaces** (pure contracts that define what a class must do without specifying how). Abstraction is the backbone of every Spring Boot application: Spring Data JPA repositories are interfaces, Spring Security filters are abstract classes, and every service contract your team defines is an interface. Understanding when to use abstract classes vs interfaces is one of the most important architectural decisions you will make as a backend engineer.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Inheritance - Single Multilevel Hierarchical]]
> - [[Java - Polymorphism - Compile Time and Runtime]]
> - [[Java - Encapsulation - Getters Setters Access Modifiers]]

---

## Theory

### What is Abstraction?

Abstraction means focusing on **what** something does rather than **how** it does it. When you drive a car, you use the steering wheel, accelerator, and brake. You do not need to understand how the fuel injection system works, how the transmission shifts gears, or how the ABS modulates brake pressure. The car's controls are an **abstraction** over the complex mechanical systems underneath.

In programming, abstraction lets you define a contract (a set of methods) that any implementing class must fulfill, without specifying the implementation details. This separates the **interface** (what the client sees) from the **implementation** (how the work is done).

Consider a payment system. The business logic needs to charge a customer. It does not care whether the payment goes through Stripe, bKash, or a bank transfer. It only cares that the payment is processed and a result is returned. Abstraction lets you define a `PaymentGateway` contract with a `charge()` method, and each payment provider implements it differently. The business logic works with the contract, not the specific provider.

### Abstract Classes

An abstract class is a class that **cannot be instantiated** directly. It serves as a partial blueprint for its subclasses. An abstract class can contain:

- **Abstract methods**: Methods declared without a body. Subclasses must provide the implementation.
- **Concrete methods**: Fully implemented methods that subclasses inherit as-is.
- **Fields**: Instance variables, static variables, and constants.
- **Constructors**: Called by subclasses via `super()` during object creation.

```java
import java.math.BigDecimal;

public abstract class PaymentGateway {
    // Field: shared by all subclasses
    protected String gatewayName;
    protected boolean isSandbox;

    // Constructor: called by subclasses
    protected PaymentGateway(String gatewayName, boolean isSandbox) {
        this.gatewayName = gatewayName;
        this.isSandbox = isSandbox;
    }

    // Abstract method: subclasses MUST implement this
    public abstract PaymentResult charge(BigDecimal amount, String customerId);

    // Abstract method: different gateways refund differently
    public abstract PaymentResult refund(String transactionId, BigDecimal amount);

    // Concrete method: shared logic inherited by all subclasses
    public void validateAmount(BigDecimal amount) {
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }

    // Concrete method: shared logging logic
    public void logTransaction(String transactionId, BigDecimal amount, String status) {
        System.out.printf("[%s] Transaction %s: %s BDT - %s%n",
            gatewayName, transactionId, amount, status);
    }
}
```

**Key rules for abstract classes:**

1. You cannot create an object of an abstract class: `new PaymentGateway()` is a compilation error.
2. A class that contains even one abstract method must be declared `abstract`.
3. A subclass of an abstract class must either implement all abstract methods or be declared `abstract` itself.
4. Abstract classes can have constructors, but they are only called by subclass constructors via `super()`.
5. Abstract methods cannot be `private` (subclasses would not be able to override them) or `static` (static methods belong to the class, not the object, and cannot be overridden).

**When to use an abstract class:**

- When multiple classes share common state (fields) and behavior (concrete methods) but differ in specific implementations.
- When you want to provide a partial implementation that subclasses extend and complete.
- When the classes are closely related in an "is-a" hierarchy (e.g., `SavingsAccount` is a `BankAccount`).

### Interfaces

An interface is a **pure contract** that defines a set of methods that a class must implement. Before Java 8, interfaces could only contain abstract methods and constants. Starting from Java 8, interfaces can also contain `default` methods (with implementations) and `static` methods.

```java
import java.util.List;

public interface Searchable {
    // Abstract method (implicitly public and abstract)
    List<Product> search(String keyword);

    // Abstract method
    List<Product> searchByCategory(String category);

    // Default method (Java 8+): provides a default implementation
    // that implementing classes can override if needed
    default List<Product> searchWithPagination(String keyword, int page, int size) {
        List<Product> allResults = search(keyword);
        int fromIndex = page * size;
        int toIndex = Math.min(fromIndex + size, allResults.size());
        if (fromIndex >= allResults.size()) {
            return List.of();
        }
        return allResults.subList(fromIndex, toIndex);
    }

    // Static method (Java 8+): utility method that belongs to the interface
    static String normalizeKeyword(String keyword) {
        return keyword.strip().toLowerCase();
    }
}
```

**Key rules for interfaces:**

1. All methods in an interface are implicitly `public`. You cannot use `private` or `protected` for abstract interface methods (Java 9 added `private` methods for internal use within default methods).
2. All fields in an interface are implicitly `public static final` (constants). You cannot have instance fields.
3. A class can implement **multiple** interfaces. This is Java's way of achieving multiple inheritance of type.
4. A class implements an interface using the `implements` keyword.
5. If a class implements an interface but does not implement all abstract methods, the class must be declared `abstract`.

**When to use an interface:**

- When you want to define a contract that unrelated classes can implement (e.g., both `User` and `Product` can implement `Searchable`).
- When you need multiple inheritance of type (a class can implement many interfaces but extend only one class).
- When you want to define a capability or role (e.g., `Serializable`, `Comparable`, `Cloneable`).
- When you want to decouple the API from the implementation (e.g., Spring Data repositories are interfaces; Spring generates the implementation at runtime).

### Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Instantiation | Cannot be instantiated | Cannot be instantiated |
| Methods | Abstract + concrete | Abstract + default + static |
| Fields | Instance, static, final | Only `public static final` constants |
| Constructors | Yes | No |
| Inheritance | Single (`extends`) | Multiple (`implements`) |
| Access modifiers | All (`private`, `protected`, `public`) | Methods are `public` by default |
| Use case | "Is-a" relationship with shared code | "Can-do" capability or contract |
| State | Can maintain state (instance fields) | Cannot maintain state (no instance fields) |
| Evolution | Adding a method may break subclasses | Adding a `default` method does not break implementors |

**The modern guideline (Java 8+):**

- Use an **interface** when you want to define a contract that multiple unrelated classes can implement, especially when the contract might evolve over time (add new default methods without breaking existing code).
- Use an **abstract class** when you want to share state and implementation among closely related classes in an inheritance hierarchy.
- In many cases, you use both together: an interface defines the public contract, and an abstract class provides a partial implementation that concrete classes extend.

### Default Methods in Interfaces (Java 8+)

Default methods were introduced in Java 8 to allow interfaces to evolve without breaking existing implementations. Before Java 8, adding a new method to an interface would break every class that implemented it. With default methods, you can add new methods with default implementations, and existing classes continue to work without modification.

```java
import java.util.List;

public interface Repository<T> {
    T findById(Long id);
    List<T> findAll();
    T save(T entity);
    void deleteById(Long id);

    // Added in version 2.0. Existing implementations do not break
    // because they inherit this default implementation.
    default boolean existsById(Long id) {
        return findById(id) != null;
    }

    // Added in version 3.0.
    default long count() {
        return findAll().size();
    }
}
```

**The diamond problem with default methods:**

If a class implements two interfaces that both define a default method with the same signature, the compiler forces the class to override the method and resolve the ambiguity.

```java
interface A {
    default void greet() { System.out.println("Hello from A"); }
}

interface B {
    default void greet() { System.out.println("Hello from B"); }
}

class C implements A, B {
    // COMPILATION ERROR unless C overrides greet()
    @Override
    public void greet() {
        A.super.greet();  // Explicitly choose A's version
        // Or provide a completely new implementation
    }
}
```

### Sealed Classes and Interfaces (Java 17+)

Sealed classes and interfaces restrict which classes can extend or implement them. This gives you the benefits of abstraction while maintaining control over the type hierarchy.

```java
import java.math.BigDecimal;
import java.time.LocalDateTime;

// Only Order, Refund, and Subscription can extend this class.
// No other class in the entire codebase can extend Payment.
public sealed class Payment permits Order, Refund, Subscription {
    protected BigDecimal amount;
    protected LocalDateTime timestamp;
}

public final class Order extends Payment { }      // final: cannot be extended further
public final class Refund extends Payment { }      // final: cannot be extended further
public non-sealed class Subscription extends Payment { }  // non-sealed: opens the hierarchy again
```

> [!note] Sealed hierarchy requirement
> All permitted subclasses must be in the same package as the sealed class (or the same module, when using the module system). They must also be `final`, `sealed`, or `non-sealed` — the modifier is mandatory, not optional.

Sealed types are useful in backend systems when you want to model a closed set of possibilities (like order statuses, payment types, or notification channels) and ensure that the compiler catches any missing cases in switch expressions.

### How Abstraction Works Internally

At the bytecode level, abstract classes and interfaces are represented differently:

**Abstract classes** are compiled like regular classes with the `ACC_ABSTRACT` flag set in the class file header. The JVM prevents instantiation by checking this flag when `new` is executed. Abstract methods have the `ACC_ABSTRACT` flag and no `Code` attribute (no bytecode instructions).

**Interfaces** are compiled with both the `ACC_INTERFACE` and `ACC_ABSTRACT` flags. Interface methods are stored in the implementing class's vtable. When you call a method through an interface reference, the JVM uses an `invokeinterface` bytecode instruction instead of `invokevirtual`. The `invokeinterface` instruction performs a vtable lookup on the actual object's class, similar to `invokevirtual`, but with an additional indirection because the interface's method index may differ from the class's vtable index.

```text
Interface reference call:
Searchable s = new ProductSearchService();
s.search("laptop");

Bytecode:
aload_1          // Load 's' reference
ldc "laptop"     // Load argument
invokeinterface Searchable.search(String)  // JVM looks up the actual class's vtable
```

> [!tip] Key Insight
> The most important abstraction in Spring Boot is the **repository interface**. When you write `public interface OrderRepository extends JpaRepository<Order, Long>`, you define only the contract. Spring Data generates the implementation at runtime using dynamic proxies. You never write the SQL queries, the connection management, or the transaction handling. Spring generates all of that based on the interface method names and annotations. This is abstraction at its most powerful: you define what you want, and the framework figures out how to do it.

---

## Syntax and Basic Examples

### Example 1: Abstract class with abstract and concrete methods

```java
public abstract class NotificationService {
    protected String senderName;
    protected int maxRetries;

    protected NotificationService(String senderName, int maxRetries) {
        this.senderName = senderName;
        this.maxRetries = maxRetries;
    }

    // Abstract methods: each channel implements these differently
    public abstract void send(String recipient, String message);
    public abstract String getChannelName();

    // Concrete method: shared retry logic
    public void sendWithRetry(String recipient, String message) {
        int attempts = 0;
        while (attempts < maxRetries) {
            try {
                send(recipient, message);  // Calls the subclass's implementation
                System.out.println("[" + getChannelName() + "] Sent successfully to " + recipient);
                return;
            } catch (Exception e) {
                attempts++;
                System.out.println("[" + getChannelName() + "] Attempt " + attempts + " failed: " + e.getMessage());
            }
        }
        System.out.println("[" + getChannelName() + "] Failed after " + maxRetries + " attempts");
    }

    // Concrete method: shared validation
    protected void validateRecipient(String recipient) {
        if (recipient == null || recipient.isBlank()) {
            throw new IllegalArgumentException("Recipient cannot be empty");
        }
    }
}
```

```java
public class EmailService extends NotificationService {
    private String smtpServer;

    public EmailService(String smtpServer) {
        super("EmailSystem", 3);  // Calls NotificationService constructor
        this.smtpServer = smtpServer;
    }

    @Override
    public void send(String recipient, String message) {
        validateRecipient(recipient);  // Uses inherited concrete method
        if (!recipient.contains("@")) {
            throw new IllegalArgumentException("Invalid email: " + recipient);
        }
        System.out.println("  Connecting to " + smtpServer);
        System.out.println("  Sending email to " + recipient + ": " + message);
    }

    @Override
    public String getChannelName() {
        return "EMAIL";
    }
}
```

```java
public class SmsService extends NotificationService {
    private String apiGateway;

    public SmsService(String apiGateway) {
        super("SmsSystem", 2);
        this.apiGateway = apiGateway;
    }

    @Override
    public void send(String recipient, String message) {
        validateRecipient(recipient);
        if (!recipient.startsWith("+88")) {
            throw new IllegalArgumentException("Only Bangladeshi numbers supported");
        }
        System.out.println("  Calling API: " + apiGateway);
        System.out.println("  Sending SMS to " + recipient + ": " + message);
    }

    @Override
    public String getChannelName() {
        return "SMS";
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        // Cannot instantiate abstract class:
        // NotificationService ns = new NotificationService("test", 3);  // ERROR

        NotificationService email = new EmailService("smtp.gmail.com");
        NotificationService sms = new SmsService("https://sms-gateway.com/api");

        // Polymorphism: same method call, different behavior
        email.sendWithRetry("saad@example.com", "Your order is confirmed");
        System.out.println();
        sms.sendWithRetry("+8801712345678", "Your OTP is 4829");
    }
}
```

**Output:**

```text
  Connecting to smtp.gmail.com
  Sending email to saad@example.com: Your order is confirmed
[EMAIL] Sent successfully to saad@example.com

  Calling API: https://sms-gateway.com/api
  Sending SMS to +8801712345678: Your OTP is 4829
[SMS] Sent successfully to +8801712345678
```

### Example 2: Interface with default and static methods

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.function.Supplier;

public interface Cacheable {
    // Abstract methods: each implementation handles caching differently
    void put(String key, Object value, int ttlSeconds);
    Object get(String key);
    void evict(String key);

    // Default method: provides a common "get or compute" pattern
    default Object getOrCompute(String key, Supplier<Object> supplier, int ttlSeconds) {
        Object cached = get(key);
        if (cached != null) {
            System.out.println("Cache HIT for key: " + key);
            return cached;
        }
        System.out.println("Cache MISS for key: " + key);
        Object computed = supplier.get();
        put(key, computed, ttlSeconds);
        return computed;
    }

    // Default method: clear all keys matching a prefix
    default void evictByPrefix(String prefix, List<String> allKeys) {
        for (String key : allKeys) {
            if (key.startsWith(prefix)) {
                evict(key);
            }
        }
    }

    // Static method: utility for generating cache keys
    static String generateKey(String entityType, Long entityId) {
        return entityType.toLowerCase() + ":" + entityId;
    }
}
```

```java
import java.util.HashMap;
import java.util.Map;

public class RedisCache implements Cacheable {
    // In a real application, this would use a Redis client library.
    // Simplified here with a HashMap for demonstration.
    private final Map<String, Object> store = new HashMap<>();

    @Override
    public void put(String key, Object value, int ttlSeconds) {
        store.put(key, value);
        System.out.println("Redis: Stored " + key + " (TTL: " + ttlSeconds + "s)");
    }

    @Override
    public Object get(String key) {
        return store.get(key);
    }

    @Override
    public void evict(String key) {
        store.remove(key);
        System.out.println("Redis: Evicted " + key);
    }
}
```

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class InMemoryCache implements Cacheable {
    private final Map<String, Object> store = new HashMap<>();

    @Override
    public void put(String key, Object value, int ttlSeconds) {
        store.put(key, value);
        System.out.println("InMemory: Stored " + key);
    }

    @Override
    public Object get(String key) {
        return store.get(key);
    }

    @Override
    public void evict(String key) {
        store.remove(key);
        System.out.println("InMemory: Evicted " + key);
    }

    // Override the default method with a more efficient implementation
    @Override
    public void evictByPrefix(String prefix, List<String> allKeys) {
        System.out.println("InMemory: Bulk evicting prefix " + prefix);
        store.keySet().removeIf(key -> key.startsWith(prefix));
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Cacheable cache = new RedisCache();

        // Use the static utility method
        String key = Cacheable.generateKey("Product", 42L);
        System.out.println("Generated key: " + key);  // product:42

        // Use the default method
        Object product = cache.getOrCompute(key, () -> {
            System.out.println("  Fetching product from database...");
            return "ThinkPad T14";
        }, 300);
        System.out.println("Result: " + product);

        // Second call hits the cache
        Object cached = cache.getOrCompute(key, () -> {
            System.out.println("  This should NOT print");
            return "ThinkPad T14";
        }, 300);
        System.out.println("Result: " + cached);
    }
}
```

**Output:**

```text
Generated key: product:42
Cache MISS for key: product:42
  Fetching product from database...
Redis: Stored product:42 (TTL: 300s)
Result: ThinkPad T14
Cache HIT for key: product:42
Result: ThinkPad T14
```

### Example 3: Multiple interfaces on a single class

```java
import java.time.LocalDateTime;

public interface Auditable {
    LocalDateTime getCreatedAt();
    LocalDateTime getUpdatedAt();
    String getCreatedBy();
}

public interface SoftDeletable {
    boolean isDeleted();
    void markAsDeleted();
    void restore();
}

public interface Versioned {
    int getVersion();
    void incrementVersion();
}
```

```java
import java.time.LocalDateTime;

// A single class implementing three interfaces.
// This is multiple inheritance of type, which Java supports through interfaces.
public class Order implements Auditable, SoftDeletable, Versioned {
    private Long id;
    private String orderNumber;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private String createdBy;
    private boolean deleted;
    private int version;

    public Order(String orderNumber, String createdBy) {
        this.orderNumber = orderNumber;
        this.createdBy = createdBy;
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
        this.deleted = false;
        this.version = 1;
    }

    // Auditable implementation
    @Override public LocalDateTime getCreatedAt() { return createdAt; }
    @Override public LocalDateTime getUpdatedAt() { return updatedAt; }
    @Override public String getCreatedBy() { return createdBy; }

    // SoftDeletable implementation
    @Override public boolean isDeleted() { return deleted; }
    @Override public void markAsDeleted() {
        this.deleted = true;
        this.updatedAt = LocalDateTime.now();
        this.version++;
    }
    @Override public void restore() {
        this.deleted = false;
        this.updatedAt = LocalDateTime.now();
        this.version++;
    }

    // Versioned implementation
    @Override public int getVersion() { return version; }
    @Override public void incrementVersion() { this.version++; }
}
```

### Example 4: Combining abstract class and interface

> [!note] `PaymentResult` is assumed defined
> The examples below use a `PaymentResult` record that is not declared in this note. The `Main` at the end calls `result.status()` and `result.message()`, so define it as:
> ```java
> public record PaymentResult(String transactionId, String status, String message) { }
> ```

```java
import java.math.BigDecimal;

// Interface: defines the public contract
public interface PaymentProcessor {
    PaymentResult processPayment(BigDecimal amount, String customerId);
    PaymentResult refund(String transactionId);
    String getProcessorName();
}
```

```java
import java.math.BigDecimal;

// Abstract class: provides shared implementation for all payment processors
public abstract class BasePaymentProcessor implements PaymentProcessor {
    protected final String apiKey;
    protected final boolean isSandbox;

    protected BasePaymentProcessor(String apiKey, boolean isSandbox) {
        this.apiKey = apiKey;
        this.isSandbox = isSandbox;
    }

    // Concrete method: shared validation logic
    protected void validatePayment(BigDecimal amount, String customerId) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        if (customerId == null || customerId.isBlank()) {
            throw new IllegalArgumentException("Customer ID is required");
        }
    }

    // Concrete method: shared logging
    protected void logPayment(String action, BigDecimal amount, String status) {
        System.out.printf("[%s] %s: %s BDT - %s%n",
            getProcessorName(), action, amount, status);
    }

    // Concrete implementation of a common pattern
    @Override
    public PaymentResult refund(String transactionId) {
        logPayment("REFUND", BigDecimal.ZERO, "Processing");
        // Default refund logic: subclasses can override if needed
        return new PaymentResult(transactionId, "REFUNDED", "Refund processed");
    }

    // processPayment() remains abstract: each processor charges differently
}
```

```java
import java.math.BigDecimal;

// Concrete class: implements the remaining abstract method
public class StripeProcessor extends BasePaymentProcessor {

    public StripeProcessor(String apiKey, boolean isSandbox) {
        super(apiKey, isSandbox);
    }

    @Override
    public PaymentResult processPayment(BigDecimal amount, String customerId) {
        validatePayment(amount, customerId);  // Uses inherited concrete method
        logPayment("CHARGE", amount, "Initiated");

        // Stripe-specific implementation
        String transactionId = "ch_" + System.currentTimeMillis();
        logPayment("CHARGE", amount, "Success");
        return new PaymentResult(transactionId, "SUCCESS", "Charged via Stripe");
    }

    @Override
    public String getProcessorName() {
        return "Stripe" + (isSandbox ? " (Sandbox)" : "");
    }
}
```

```java
import java.math.BigDecimal;

public class Main {
    public static void main(String[] args) {
        // The business logic works with the interface, not the concrete class.
        // This makes it trivial to swap Stripe for bKash or any other processor.
        PaymentProcessor processor = new StripeProcessor("sk_test_abc123", true);

        PaymentResult result = processor.processPayment(
            new BigDecimal("5000.00"), "cust_12345"
        );
        System.out.println("Result: " + result.status() + " - " + result.message());
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Abstraction is the architectural backbone of Spring Boot. Here are three realistic scenarios.

### Scenario 1: Spring Data JPA Repository interfaces

Spring Data JPA is the most striking example of abstraction in the Java ecosystem. You define an interface, and Spring generates the implementation at runtime.

```java
package com.company.orderservice.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

// This interface extends JpaRepository, which itself extends
// PagingAndSortingRepository, which extends CrudRepository.
// Each level adds more abstract methods to the contract.
//
// You write ZERO implementation code. Spring generates the
// implementation class at runtime using dynamic proxies.
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Spring generates the SQL from the method name:
    // SELECT * FROM orders WHERE user_id = ?
    List<Order> findByUserId(Long userId);

    // SELECT * FROM orders WHERE user_id = ? AND status = ?
    List<Order> findByUserIdAndStatus(Long userId, OrderStatus status);

    // SELECT * FROM orders WHERE created_at BETWEEN ? AND ?
    List<Order> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    // SELECT * FROM orders WHERE order_number = ?
    Optional<Order> findByOrderNumber(String orderNumber);

    // Custom JPQL query for complex cases
    @Query("SELECT o FROM Order o WHERE o.totalAmount > :minAmount " +
           "AND o.status = :status ORDER BY o.totalAmount DESC")
    List<Order> findHighValueOrders(
        @Param("minAmount") BigDecimal minAmount,
        @Param("status") OrderStatus status
    );

    // Count query
    long countByStatus(OrderStatus status);

    // Delete query
    void deleteByUserIdAndStatus(Long userId, OrderStatus status);
}
```

**What to notice:**

- `OrderRepository` is an interface, not a class. You never write `class OrderRepositoryImpl`. Spring creates the implementation at startup using a dynamic proxy that intercepts method calls and translates them into JPA queries.
- The method names follow Spring Data's naming convention. `findByUserIdAndStatus` is automatically translated to `SELECT * FROM orders WHERE user_id = ? AND status = ?`. This is abstraction at its most extreme: the method name IS the implementation.
- `JpaRepository<Order, Long>` is itself an interface that extends `PagingAndSortingRepository<Order, Long>`, which extends `CrudRepository<Order, Long>`, which extends `Repository<Order, Long>`. This is a hierarchy of interfaces, each adding more methods to the contract. Your `OrderRepository` inherits `save()`, `findById()`, `findAll()`, `deleteById()`, `count()`, and dozens of other methods without declaring them.

### Scenario 2: Service interface with multiple implementations

In a well-architected backend, services are defined as interfaces with concrete implementations. This allows you to swap implementations for testing, different environments, or different business requirements.

```java
package com.company.orderservice.service;

// The interface defines the contract. The rest of the application
// (controllers, other services) depends on this interface, not on
// the concrete implementation. This is the Dependency Inversion Principle.
public interface EmailService {
    void sendOrderConfirmation(Order order);
    void sendPasswordReset(String email, String token);
    void sendPromotional(String email, String campaignId);
    boolean isHealthy();
}
```

```java
// Production implementation: sends real emails via SendGrid
@Service
@Profile("production")  // Only active in the production environment
public class SendGridEmailService implements EmailService {
    private final SendGridClient client;

    public SendGridEmailService(SendGridClient client) {
        this.client = client;
    }

    @Override
    public void sendOrderConfirmation(Order order) {
        String html = buildOrderConfirmationTemplate(order);
        client.send(order.getUserEmail(), "Order Confirmed", html);
    }

    @Override
    public void sendPasswordReset(String email, String token) {
        String html = buildPasswordResetTemplate(token);
        client.send(email, "Reset Your Password", html);
    }

    @Override
    public void sendPromotional(String email, String campaignId) {
        String html = fetchCampaignTemplate(campaignId);
        client.send(email, "Special Offer", html);
    }

    @Override
    public boolean isHealthy() {
        return client.ping();
    }
}
```

```java
// Development implementation: logs emails instead of sending them
@Service
@Profile("development")  // Only active during local development
public class LoggingEmailService implements EmailService {

    private static final Logger logger = LoggerFactory.getLogger(LoggingEmailService.class);

    @Override
    public void sendOrderConfirmation(Order order) {
        logger.info("DEV EMAIL - Order Confirmation to {}: Order #{}",
            order.getUserEmail(), order.getOrderNumber());
    }

    @Override
    public void sendPasswordReset(String email, String token) {
        logger.info("DEV EMAIL - Password Reset to {}: Token={}", email, token);
    }

    @Override
    public void sendPromotional(String email, String campaignId) {
        logger.info("DEV EMAIL - Promotional to {}: Campaign={}", email, campaignId);
    }

    @Override
    public boolean isHealthy() {
        return true;  // Always healthy in development
    }
}
```

```java
// Test implementation: records sent emails for assertions
public class MockEmailService implements EmailService {
    private final List<String> sentEmails = new ArrayList<>();

    @Override
    public void sendOrderConfirmation(Order order) {
        sentEmails.add("CONFIRMATION:" + order.getUserEmail());
    }

    @Override
    public void sendPasswordReset(String email, String token) {
        sentEmails.add("RESET:" + email);
    }

    @Override
    public void sendPromotional(String email, String campaignId) {
        sentEmails.add("PROMO:" + email);
    }

    @Override
    public boolean isHealthy() { return true; }

    // Test helper: verify that specific emails were sent
    public boolean wasSentTo(String email) {
        return sentEmails.stream().anyMatch(e -> e.contains(email));
    }
}
```

**What to notice:**

- The controller and other services depend on the `EmailService` interface, not on `SendGridEmailService`. This means you can swap the implementation without changing any calling code.
- Spring's `@Profile` annotation selects the implementation based on the active environment. In production, real emails are sent. In development, emails are logged. In tests, emails are recorded for assertions. All three implementations satisfy the same interface contract.
- This pattern is called **Dependency Inversion**: high-level modules (controllers) depend on abstractions (interfaces), not on low-level modules (SendGrid client). This is one of the SOLID principles and the foundation of testable backend architecture.

### Scenario 3: Abstract base class for event handlers

Event-driven architectures use abstract classes to define common event handling patterns while allowing specific handlers to implement domain-specific logic.

```java
package com.company.orderservice.events;

// Abstract base class for all event handlers.
// Provides shared infrastructure: logging, error handling, retry logic.
public abstract class EventHandler<T extends DomainEvent> {

    protected abstract Class<T> getEventType();
    protected abstract void handleEvent(T event);

    // Template Method pattern: the algorithm skeleton is defined here.
    // Subclasses fill in the specific steps (getEventType, handleEvent).
    public final void process(DomainEvent event) {
        if (!getEventType().isInstance(event)) {
            return;  // Not our event type, skip
        }

        T typedEvent = getEventType().cast(event);
        String eventId = typedEvent.getEventId();

        try {
            // logger.info("Processing {} event: {}", getEventType().getSimpleName(), eventId);
            long start = System.currentTimeMillis();

            handleEvent(typedEvent);  // Calls the subclass's implementation

            long duration = System.currentTimeMillis() - start;
            // logger.info("Event {} processed in {}ms", eventId, duration);

        } catch (RetryableException e) {
            // logger.warn("Event {} failed, scheduling retry", eventId);
            scheduleRetry(typedEvent, e.getRetryAfter());
        } catch (Exception e) {
            // logger.error("Event {} failed permanently", eventId, e);
            moveToDeadLetterQueue(typedEvent, e);
        }
    }

    private void scheduleRetry(T event, int delaySeconds) {
        System.out.println("Scheduling retry for " + event.getEventId() + " in " + delaySeconds + "s");
    }

    private void moveToDeadLetterQueue(T event, Exception cause) {
        System.out.println("Moving " + event.getEventId() + " to DLQ: " + cause.getMessage());
    }
}
```

```java
// Concrete handler: processes order created events
@Component
public class OrderCreatedHandler extends EventHandler<OrderCreatedEvent> {

    private final InventoryService inventoryService;
    private final EmailService emailService;

    public OrderCreatedHandler(InventoryService inventoryService, EmailService emailService) {
        this.inventoryService = inventoryService;
        this.emailService = emailService;
    }

    @Override
    protected Class<OrderCreatedEvent> getEventType() {
        return OrderCreatedEvent.class;
    }

    @Override
    protected void handleEvent(OrderCreatedEvent event) {
        inventoryService.reserveStock(event.getOrderId(), event.getItems());
        emailService.sendOrderConfirmation(event.getOrder());
    }
}
```

```java
// Concrete handler: processes payment completed events
@Component
public class PaymentCompletedHandler extends EventHandler<PaymentCompletedEvent> {

    private final OrderService orderService;

    public PaymentCompletedHandler(OrderService orderService) {
        this.orderService = orderService;
    }

    @Override
    protected Class<PaymentCompletedEvent> getEventType() {
        return PaymentCompletedEvent.class;
    }

    @Override
    protected void handleEvent(PaymentCompletedEvent event) {
        orderService.confirmOrder(event.getOrderId());
    }
}
```

**What to notice:**

- The `process()` method in the abstract class is declared `final`. Subclasses cannot override it. This ensures that logging, error handling, and retry logic are consistent across all event handlers. This is the **Template Method** design pattern.
- Subclasses only implement `getEventType()` and `handleEvent()`. The infrastructure (logging, retries, dead letter queue) is inherited from the abstract class. This eliminates hundreds of lines of duplicated boilerplate across handlers.
- Adding a new event handler requires creating a new class that extends `EventHandler<NewEventType>` and implementing two methods. The retry logic, error handling, and logging come for free.

---

## Java vs Python Comparison

> [!note] Cross-language perspective
> Python uses Abstract Base Classes (ABCs) and duck typing instead of Java-style interfaces.

| Aspect | Java | Python |
|--------|------|--------|
| Abstract class | `abstract class` + `abstract` methods | `ABC` module + `@abstractmethod` |
| Interface | `interface` keyword | No dedicated keyword; use ABCs or Protocols |
| Multiple inheritance | Classes: No. Interfaces: Yes | Yes (classes, with MRO) |
| Default methods | `default` keyword (Java 8+) | Regular methods in ABCs |
| Enforcement | Compile-time | Runtime (on instantiation) |
| Duck typing | No (nominal typing) | Yes ("if it walks like a duck...") |

```java
// Java: explicit interface implementation
public interface PaymentGateway {
    PaymentResult charge(BigDecimal amount);
}

public class StripeGateway implements PaymentGateway {
    @Override
    public PaymentResult charge(BigDecimal amount) {
        return new PaymentResult("SUCCESS");
    }
}
```

```python
# Python: ABC-based abstraction
from abc import ABC, abstractmethod
from decimal import Decimal

class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: Decimal) -> dict:
        pass

class StripeGateway(PaymentGateway):
    def charge(self, amount: Decimal) -> dict:
        return {"status": "SUCCESS"}

# Python also supports duck typing (no ABC needed):
# Any class with a charge() method works, even without inheriting PaymentGateway.
```

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using abstract classes when interfaces are more appropriate

**Wrong:**

```java
public abstract class Flyable {
    public abstract void fly();
}

public class Bird extends Flyable {
    // Bird can only extend Flyable. If Bird also needs to extend Animal,
    // it cannot, because Java does not support multiple class inheritance.
    @Override
    public void fly() { System.out.println("Flapping wings"); }
}
```

**Right:**

```java
public interface Flyable {
    void fly();
}

public class Bird extends Animal implements Flyable {
    // Bird extends Animal AND implements Flyable.
    // Interfaces allow multiple inheritance of type.
    @Override
    public void fly() { System.out.println("Flapping wings"); }
}
```

**Why it is wrong:** Abstract classes consume the single inheritance slot. If `Bird` extends `Flyable` (an abstract class), it cannot also extend `Animal`. Interfaces do not consume the inheritance slot, so a class can extend one class and implement many interfaces. Use interfaces for capabilities (`Flyable`, `Swimmable`, `Serializable`) and abstract classes for shared base implementations in a type hierarchy.

### Mistake 2: Putting implementation logic in interfaces when an abstract class is better

**Wrong:**

```java
public interface OrderService {
    // Interface with lots of default methods containing complex business logic.
    // This is hard to test, hard to maintain, and violates the purpose of interfaces.
    default Order createOrder(Long userId, List<OrderItem> items) {
        // 50 lines of business logic...
    }
    default void cancelOrder(Long orderId) {
        // 30 lines of business logic...
    }
}
```

**Right:**

```java
// Interface: pure contract
public interface OrderService {
    Order createOrder(Long userId, List<OrderItem> items);
    void cancelOrder(Long orderId);
}

// Abstract class: shared implementation
public abstract class BaseOrderService implements OrderService {
    protected void validateItems(List<OrderItem> items) { /* ... */ }
    protected BigDecimal calculateTotal(List<OrderItem> items) { /* ... */ }
}

// Concrete class: specific implementation
public class OrderServiceImpl extends BaseOrderService {
    @Override
    public Order createOrder(Long userId, List<OrderItem> items) { /* ... */ }
    @Override
    public void cancelOrder(Long orderId) { /* ... */ }
}
```

**Why it is wrong:** Interfaces should define contracts, not contain complex business logic. Default methods are meant for backward-compatible evolution (adding new methods without breaking existing implementations), not for sharing large blocks of code. If you find yourself writing many default methods with complex logic, you need an abstract class.

### Mistake 3: Creating interfaces with only one implementation

**Wrong:**

```java
public interface IUserService {
    User findById(Long id);
    User create(User user);
}

public class UserServiceImpl implements IUserService {
    // The only implementation. The interface adds no value.
    // It just adds an extra file and an extra layer of indirection.
}
```

**Right:**

```java
// If there is only one implementation and no plan for more,
// just use the class directly.
public class UserService {
    public User findById(Long id) { /* ... */ }
    public User create(User user) { /* ... */ }
}

// Create an interface only when you have (or anticipate) multiple implementations:
// - SendGridEmailService, LoggingEmailService, MockEmailService
// - StripePaymentProcessor, BkashPaymentProcessor
// - RedisCacheService, InMemoryCacheService
```

**Why it is wrong:** An interface with a single implementation is premature abstraction. It adds complexity without benefit. The "always code to an interface" advice is overapplied. Create interfaces when you have a genuine need for multiple implementations, testability requirements, or framework contracts (like Spring Data repositories). Do not create interfaces just because a coding standard says so.

### Mistake 4: Forgetting that interface fields are constants

**Wrong:**

```java
public interface Configurable {
    int MAX_RETRIES = 3;  // This is public static final, not an instance field!

    default void setMaxRetries(int retries) {
        MAX_RETRIES = retries;  // COMPILATION ERROR! Cannot assign to a final variable.
    }
}
```

**Right:**

```java
public interface Configurable {
    int DEFAULT_MAX_RETRIES = 3;  // Constant: public static final
    String DEFAULT_TIMEOUT = "30s";

    // If you need mutable state, use an abstract class, not an interface.
}
```

**Why it is wrong:** All fields in an interface are implicitly `public static final`. They are compile-time constants shared by all implementing classes. You cannot have instance fields in an interface because interfaces do not have instances. If you need mutable state, use an abstract class.

---

## Key Takeaways

> [!tip] Remember these points
> 1. **Abstraction** hides implementation details and exposes only the essential behavior through a contract. It separates what a component does from how it does it.
> 2. **Abstract classes** cannot be instantiated, can contain both abstract and concrete methods, can have instance fields and constructors, and support single inheritance. Use them when closely related classes share state and behavior.
> 3. **Interfaces** define pure contracts with abstract methods, default methods, and static methods. They cannot have instance fields or constructors. A class can implement multiple interfaces. Use them for capabilities, contracts, and decoupling.
> 4. **Default methods** (Java 8+) allow interfaces to evolve without breaking existing implementations. Use them for backward-compatible additions, not for complex business logic.
> 5. The most powerful abstraction in Spring Boot is the **repository interface**. You define the contract (method names), and Spring generates the implementation at runtime. Service interfaces with multiple implementations (production, development, test) are the foundation of testable backend architecture.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Shape Interface and Abstract Class (Easy)

Refactor the Shape hierarchy from previous exercises to use both an interface and an abstract class:

1. Create an interface `Drawable` with methods `double area()`, `double perimeter()`, and `void draw()` (prints the shape's details).
2. Create an abstract class `AbstractShape` that implements `Drawable`. It has a `color` field, a constructor, and a concrete `draw()` method that prints the color and delegates to `toString()`. It leaves `area()` and `perimeter()` abstract.
3. Create `Circle`, `Rectangle`, and `Triangle` classes that extend `AbstractShape` and implement `area()` and `perimeter()`.

In `main()`, create a `Drawable[]` array with one of each shape and call `draw()`, `area()`, and `perimeter()` on each.

> **Hint:** The abstract class provides the shared `color` field and `draw()` implementation. The interface provides the contract that allows unrelated classes (if any) to also be `Drawable`.

### Exercise 2: Plugin System with Interfaces (Medium)

Build a simple plugin system using interfaces:

1. Create an interface `Plugin` with methods `String getName()`, `void initialize()`, `void execute(String input)`, and `void shutdown()`.
2. Add a default method `String getVersion()` that returns "1.0.0".
3. Create three implementations:
   - `LoggingPlugin`: logs the input to the console.
   - `EncryptionPlugin`: "encrypts" the input by reversing it.
   - `CompressionPlugin`: "compresses" the input by removing all vowels.
4. Create a `PluginManager` class that holds a `List<Plugin>`. It has methods `registerPlugin(Plugin)`, `initializeAll()`, `executeAll(String input)`, and `shutdownAll()`.

In `main()`, register all three plugins, initialize them, execute a sample input through all of them, and shut them down.

> **Hint:** The `PluginManager` works with the `Plugin` interface. It does not know or care about the specific plugin implementations. This is runtime polymorphism through interfaces.

### Exercise 3: Repository Pattern with Abstract Base (Medium)

Simulate the Spring Data repository pattern:

1. Create an interface `Repository<T>` with methods `T findById(Long id)`, `List<T> findAll()`, `T save(T entity)`, `void deleteById(Long id)`, and default methods `boolean existsById(Long id)` and `long count()`.
2. Create an abstract class `InMemoryRepository<T>` that implements `Repository<T>`. It uses a `Map<Long, T>` internally to store entities. It implements all methods except `findById()` (which subclasses might want to customize with caching or logging).
3. Create `UserRepository extends InMemoryRepository<User>` and `ProductRepository extends InMemoryRepository<Product>`. Override `findById()` in `UserRepository` to add logging.

In `main()`, create repositories, save entities, find them, and test the default methods.

> **Hint:** Use Java generics (`<T>`) to make the repository type-safe. The `InMemoryRepository` provides the shared Map-based storage, and concrete repositories add domain-specific behavior.

### Exercise 4: Sealed Interface for Order Events (Hard, Optional)

Use Java 17 sealed interfaces to model a closed set of order events:

1. Create a sealed interface `OrderEvent` that permits `OrderCreated`, `OrderConfirmed`, `OrderShipped`, `OrderDelivered`, and `OrderCancelled`.
2. Each event is a record that implements `OrderEvent` and contains relevant fields (orderId, timestamp, and event-specific data).
3. Create a method `processEvent(OrderEvent event)` that uses a switch expression with pattern matching to handle each event type. The compiler should warn you if you forget a case (because the interface is sealed).
4. Try to create a new class that implements `OrderEvent` and observe the compilation error.

> **Hint:** Sealed interfaces + switch pattern matching = exhaustive type checking. The compiler knows all possible implementations and can verify that your switch covers all of them.

<details>
<summary>Solution for Exercise 1</summary>

```java
interface Drawable {
    double area();
    double perimeter();
    void draw();
}

abstract class AbstractShape implements Drawable {
    protected String color;

    protected AbstractShape(String color) {
        this.color = color;
    }

    @Override
    public void draw() {
        System.out.printf("%s %s | Area: %.2f | Perimeter: %.2f%n",
            getClass().getSimpleName(), color, area(), perimeter());
    }
}

class Circle extends AbstractShape {
    private double radius;
    Circle(String color, double radius) { super(color); this.radius = radius; }
    @Override public double area() { return Math.PI * radius * radius; }
    @Override public double perimeter() { return 2 * Math.PI * radius; }
}

class Rectangle extends AbstractShape {
    private double width, height;
    Rectangle(String color, double w, double h) { super(color); width = w; height = h; }
    @Override public double area() { return width * height; }
    @Override public double perimeter() { return 2 * (width + height); }
}

class Triangle extends AbstractShape {
    private double a, b, c;
    Triangle(String color, double a, double b, double c) {
        super(color); this.a = a; this.b = b; this.c = c;
    }
    @Override public double area() {
        double s = (a + b + c) / 2;
        return Math.sqrt(s * (s - a) * (s - b) * (s - c));
    }
    @Override public double perimeter() { return a + b + c; }
}

public class Main {
    public static void main(String[] args) {
        Drawable[] shapes = {
            new Circle("Red", 5),
            new Rectangle("Blue", 4, 6),
            new Triangle("Green", 3, 4, 5)
        };
        for (Drawable d : shapes) {
            d.draw();
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;

interface Plugin {
    String getName();
    void initialize();
    void execute(String input);
    void shutdown();
    default String getVersion() { return "1.0.0"; }
}

class LoggingPlugin implements Plugin {
    public String getName() { return "Logger"; }
    public void initialize() { System.out.println("[Logger] Initialized"); }
    public void execute(String input) { System.out.println("[Logger] Input: " + input); }
    public void shutdown() { System.out.println("[Logger] Shutdown"); }
}

class EncryptionPlugin implements Plugin {
    public String getName() { return "Encryptor"; }
    public void initialize() { System.out.println("[Encryptor] Initialized"); }
    public void execute(String input) {
        String reversed = new StringBuilder(input).reverse().toString();
        System.out.println("[Encryptor] Encrypted: " + reversed);
    }
    public void shutdown() { System.out.println("[Encryptor] Shutdown"); }
}

class CompressionPlugin implements Plugin {
    public String getName() { return "Compressor"; }
    public void initialize() { System.out.println("[Compressor] Initialized"); }
    public void execute(String input) {
        String compressed = input.replaceAll("[aeiouAEIOU]", "");
        System.out.println("[Compressor] Compressed: " + compressed);
    }
    public void shutdown() { System.out.println("[Compressor] Shutdown"); }
}

class PluginManager {
    private final List<Plugin> plugins = new ArrayList<>();

    void registerPlugin(Plugin plugin) {
        plugins.add(plugin);
        System.out.println("Registered: " + plugin.getName() + " v" + plugin.getVersion());
    }

    void initializeAll() { plugins.forEach(Plugin::initialize); }
    void executeAll(String input) { plugins.forEach(p -> p.execute(input)); }
    void shutdownAll() { plugins.forEach(Plugin::shutdown); }
}

public class Main {
    public static void main(String[] args) {
        PluginManager manager = new PluginManager();
        manager.registerPlugin(new LoggingPlugin());
        manager.registerPlugin(new EncryptionPlugin());
        manager.registerPlugin(new CompressionPlugin());

        manager.initializeAll();
        System.out.println("\n--- Processing ---");
        manager.executeAll("Hello Backend World");
        System.out.println("\n--- Shutting Down ---");
        manager.shutdownAll();
    }
}
```

</details>

---

## Related Notes

- [[Java - Polymorphism - Compile Time and Runtime]]
- [[Java - Static Keyword - Variables Methods Blocks]] (next note)
- [[Java - Collections Framework Overview]]

---

## Resources

- [Oracle Java Tutorials: Abstract Methods and Classes](https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html) - Official documentation on abstract classes and methods.
- [Oracle Java Tutorials: Interfaces](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html) - Official documentation covering interface declaration, implementation, and default methods.
- [Baeldung: Abstract Class vs Interface in Java](https://www.baeldung.com/java-abstract-class-vs-interface) - Comprehensive comparison with decision guidelines.
- [Baeldung: Java 8 Default Methods](https://www.baeldung.com/java-default-methods) - Detailed guide on default methods and the diamond problem.
- [Baeldung: Java Sealed Classes and Interfaces](https://www.baeldung.com/java-sealed-classes-interfaces) - Guide to Java 17 sealed types with pattern matching examples.
- [Effective Java by Joshua Bloch - Item 20: Prefer Interfaces to Abstract Classes](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive argument for when to choose interfaces over abstract classes.
