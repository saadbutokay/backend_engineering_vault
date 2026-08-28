---
title: "Java - Encapsulation - Getters Setters Access Modifiers"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - encapsulation
  - access-modifiers
status: "not-started"
---

# Java - Encapsulation - Getters Setters Access Modifiers

> [!abstract] Overview
> Encapsulation is the OOP principle of bundling data and the methods that operate on that data inside a single unit (the class) and restricting direct access to the internal state from outside code. In Java, encapsulation is achieved through access modifiers (`private`, `protected`, `public`, and package-private) combined with getter and setter methods that provide controlled access to an object's fields. In backend development, encapsulation is what prevents a controller from directly modifying a database entity's primary key, stops external code from setting an order's status to an invalid value, and ensures that business rules are enforced in exactly one place.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Constructors - Default Parameterized Copy]]
> - [[Java - Methods - Parameters Return Types Overloading]]

---

## Theory

### What is Encapsulation?

Encapsulation is one of the four pillars of Object-Oriented Programming (the others are Inheritance, Polymorphism, and Abstraction). It has two complementary meanings:

**1. Data bundling**: Grouping related data (fields) and behavior (methods) together inside a class. You already did this in the Classes and Objects note when you put `name`, `price`, and `calculateStockValue()` inside the `Product` class.

**2. Data hiding**: Restricting direct access to an object's internal state from outside the class. Instead of letting any code read or write the fields directly, you provide controlled access through public methods (getters and setters). The internal representation is hidden, and the class can change its implementation without breaking external code.

The analogy is a bank account. You cannot walk into a bank and directly modify the database row that stores your balance. You must go through a teller (a method) who verifies your identity, checks the rules, and then updates the balance on your behalf. The teller is the getter/setter. The database row is the private field. The bank's internal procedures are the hidden implementation.

### Why Does Encapsulation Exist?

Consider what happens without encapsulation:

```java
public class Order {
    public String orderNumber;
    public BigDecimal totalAmount;
    public OrderStatus status;
}
```

```java
// Any code anywhere in the application can do this:
Order order = new Order();
order.totalAmount = new BigDecimal("-500");  // Negative total! No validation!
order.status = OrderStatus.DELIVERED;         // Skipped PENDING, CONFIRMED, SHIPPED!
order.orderNumber = null;                     // Null order number!
```

Without encapsulation, every piece of code in your entire application can set any field to any value at any time. There is no way to enforce business rules, no way to validate data, and no way to track when and why a field changed. In a backend system with 50 developers and 200 classes, this leads to data corruption, impossible-to-debug errors, and security vulnerabilities.

Encapsulation solves this by making fields private and providing public methods that enforce rules:

```java
public class Order {
    private BigDecimal totalAmount;
    private OrderStatus status;

    public void setTotalAmount(BigDecimal totalAmount) {
        if (totalAmount == null || totalAmount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Total must be positive");
        }
        this.totalAmount = totalAmount;
    }

    public void updateStatus(OrderStatus newStatus) {
        if (!isValidTransition(this.status, newStatus)) {
            throw new IllegalStateException(
                "Cannot transition from " + this.status + " to " + newStatus
            );
        }
        this.status = newStatus;
    }
}
```

Now it is impossible to set a negative total or skip order statuses. The rules are enforced in exactly one place (the setter methods), and every caller must go through them.

### Access Modifiers

Java provides four levels of access control, from most restrictive to least restrictive:

| Modifier     | Class | Package | Subclass | World |
| ------------ | ----- | ------- | -------- | ----- |
| `private`    | Yes   | No      | No       | No    |
| *(default)*  | Yes   | Yes     | No       | No    |
| `protected`  | Yes   | Yes     | Yes      | No    |
| `public`     | Yes   | Yes     | Yes      | Yes   |

**private**: Accessible only within the same class. This is the default choice for fields. No other class, not even a subclass or a class in the same package, can access a private member directly.

```java
public class BankAccount {
    private double balance;  // Only BankAccount methods can access this

    public double getBalance() {
        return balance;  // OK: same class
    }
}

public class Auditor {
    void audit(BankAccount account) {
        double b = account.balance;      // COMPILATION ERROR: balance is private
        double b = account.getBalance(); // OK: public method
    }
}
```

**Default (package-private)**: No keyword. Accessible within the same class and any class in the same package. This is useful for classes that collaborate closely within a single package but should not be exposed to the rest of the application.

```java
// In package com.company.orderservice.util
class OrderValidator {  // No public keyword: package-private class
    boolean isValid(Order order) { ... }
}

// In the same package: can access OrderValidator
class OrderService {
    OrderValidator validator = new OrderValidator();  // OK
}

// In a different package: cannot access OrderValidator
class ExternalService {
    OrderValidator validator = new OrderValidator();  // COMPILATION ERROR
}
```

**protected**: Accessible within the same class, same package, and any subclass (even in a different package). This is primarily used for methods and fields that a parent class wants to share with its children but not with the rest of the world. You will use protected extensively when you study Inheritance.

```java
public class BaseEntity {
    protected LocalDateTime createdAt;  // Subclasses can access this
    protected void setCreatedAt(LocalDateTime time) {
        this.createdAt = time;
    }
}

public class Order extends BaseEntity {
    void initialize() {
        this.createdAt = LocalDateTime.now();       // OK: subclass
        this.setCreatedAt(LocalDateTime.now());     // OK: subclass
    }
}
```

**public**: Accessible from anywhere. Use this for methods that form the class's public API (the contract with the outside world). Avoid making fields public unless they are constants (`public static final`).

### The Principle of Least Privilege

The guiding principle for choosing access modifiers is: make every member as restrictive as possible. Start with `private` and only widen the access if there is a specific reason.

- **Fields**: Almost always `private`.
- **Helper methods used only within the class**: `private`.
- **Methods used by classes in the same package**: default (package-private).
- **Methods that subclasses need to override or call**: `protected`.
- **Methods that external code needs to call**: `public`.

This principle minimizes the "attack surface" of your class. The fewer public methods a class has, the fewer ways external code can misuse it, and the easier it is to change the internal implementation without breaking anything.

### Getters and Setters

Getters (also called accessors) are public methods that return the value of a private field. By convention, they are named `getFieldName()`.

Setters (also called mutators) are public methods that set the value of a private field after performing validation. By convention, they are named `setFieldName(Type value)`.

```java
public class User {
    private String email;

    // Getter
    public String getEmail() {
        return email;
    }

    // Setter with validation
    public void setEmail(String email) {
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email: " + email);
        }
        this.email = email.toLowerCase();  // Normalize to lowercase
    }
}
```

Important conventions:

- For boolean fields, the getter is named `isFieldName()` instead of `getFieldName()`. For example, `isActive()` instead of `getActive()`.
- Not every field needs a getter and setter. If a field should never change after construction, provide only a getter (or no getter at all if external code does not need to read it).
- Getters and setters are **not** just pass-through methods. They are an opportunity to add validation, logging, lazy initialization, caching, and transformation.

### Defensive Copying in Getters

A subtle but critical aspect of encapsulation is that getters for mutable reference types (arrays, lists, dates) should return a copy of the internal data, not the original reference. If you return the original reference, external code can modify the internal state of your object without going through the setter, completely bypassing your validation.

```java
public class Team {
    private List<String> members;

    // BAD: returns the internal list directly
    public List<String> getMembersBad() {
        return members;  // Caller can do: team.getMembersBad().clear()
    }

    // GOOD: returns a copy
    public List<String> getMembersCopy() {
        return new ArrayList<>(members);  // Caller modifies the copy, not the original
    }

    // Also good: return an unmodifiable view
    public List<String> getMembers() {
        return Collections.unmodifiableList(members);  // Throws exception if caller tries to modify
    }
}
```

This is one of the most overlooked encapsulation mistakes in backend code. A getter that returns a mutable internal reference is a security hole.

### How It Works Internally

At the bytecode level, access modifiers are enforced by the JVM, not just the compiler. When the JVM loads a class, it reads the access flags from the class file and checks them during linking and execution. If code in class A tries to access a private field in class B, the JVM throws an `IllegalAccessError` at runtime (even if the code somehow bypassed the compiler, for example through bytecode manipulation).

However, Java's reflection API (`java.lang.reflect`) can bypass access modifiers. The `setAccessible(true)` method on a `Field` or `Method` object disables access checks. This is how frameworks like Spring and Hibernate access private fields to inject dependencies and hydrate entities from database rows.

```java
Field balanceField = BankAccount.class.getDeclaredField("balance");
balanceField.setAccessible(true);         // Bypass private access
balanceField.set(account, 1000000.0);     // Modify the private field directly
```

This is powerful but dangerous. Frameworks use it responsibly. Application code should never use reflection to bypass encapsulation.

> [!tip] Key Insight
> Encapsulation is not about hiding data for the sake of hiding. It is about **controlling how data changes**. A private field with a public setter that does nothing but `this.field = value` provides no real encapsulation. The value of encapsulation comes from the validation, transformation, and invariants enforced inside the setter. If a setter has no logic beyond assignment, consider whether the field should be `public final` (immutable) or whether the class needs a richer API that expresses business operations (like `order.cancel()` instead of `order.setStatus(CANCELLED)`).

---

## Syntax and Basic Examples

### Example 1: Full encapsulation with validation

```java
public class Product {
    private long id;
    private String name;
    private double price;
    private int stockQuantity;
    private String category;

    public Product(long id, String name, double price, int stockQuantity, String category) {
        setId(id);
        setName(name);
        setPrice(price);
        setStockQuantity(stockQuantity);
        setCategory(category);
    }

    // --- Getters ---

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public double getPrice() {
        return price;
    }

    public int getStockQuantity() {
        return stockQuantity;
    }

    public String getCategory() {
        return category;
    }

    // --- Setters with validation ---

    private void setId(long id) {
        // ID is set only during construction. The setter is private.
        if (id <= 0) {
            throw new IllegalArgumentException("ID must be positive");
        }
        this.id = id;
    }

    public void setName(String name) {
        if (name == null || name.strip().isEmpty()) {
            throw new IllegalArgumentException("Product name cannot be empty");
        }
        if (name.length() > 200) {
            throw new IllegalArgumentException("Product name too long (max 200 chars)");
        }
        this.name = name.strip();
    }

    public void setPrice(double price) {
        if (price < 0) {
            throw new IllegalArgumentException("Price cannot be negative: " + price);
        }
        this.price = price;
    }

    public void setStockQuantity(int stockQuantity) {
        if (stockQuantity < 0) {
            throw new IllegalArgumentException("Stock cannot be negative: " + stockQuantity);
        }
        this.stockQuantity = stockQuantity;
    }

    public void setCategory(String category) {
        if (category == null || category.strip().isEmpty()) {
            this.category = "UNCATEGORIZED";  // Default value instead of throwing
        } else {
            this.category = category.strip().toUpperCase();  // Normalize
        }
    }

    // --- Business methods (preferred over setters for state changes) ---

    public void reduceStock(int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        if (quantity > this.stockQuantity) {
            throw new IllegalStateException(
                "Insufficient stock. Available: " + this.stockQuantity
            );
        }
        this.stockQuantity -= quantity;
    }

    public boolean isInStock() {
        return this.stockQuantity > 0;
    }

    @Override
    public String toString() {
        return String.format("Product{id=%d, name='%s', price=%.2f, stock=%d, category='%s'}",
            id, name, price, stockQuantity, category);
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Product laptop = new Product(1, "ThinkPad T14", 85000, 15, "electronics");
        System.out.println(laptop);

        laptop.setPrice(79999);  // Valid: price update
        System.out.println("New price: " + laptop.getPrice());

        laptop.reduceStock(3);  // Valid: purchase 3 units
        System.out.println("Stock after purchase: " + laptop.getStockQuantity());

        // Invalid operations are caught:
        try {
            laptop.setPrice(-100);
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }

        try {
            laptop.reduceStock(100);
        } catch (IllegalStateException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Output:**

```text
Product{id=1, name='ThinkPad T14', price=85000.00, stock=15, category='ELECTRONICS'}
New price: 79999.0
Stock after purchase: 12
Error: Price cannot be negative: -100.0
Error: Insufficient stock. Available: 12
```

### Example 2: Access modifiers across classes

```java
// File: com/company/orderservice/model/BaseEntity.java
package com.company.orderservice.model;

import java.time.LocalDateTime;

public class BaseEntity {
    private Long id;                    // private: only BaseEntity can access
    protected LocalDateTime createdAt;  // protected: BaseEntity + subclasses + same package
    LocalDateTime updatedAt;            // default: BaseEntity + same package only

    public Long getId() { return id; }  // public: anyone can read the ID

    protected void setId(Long id) {     // protected: only subclasses can set the ID
        this.id = id;
    }

    public LocalDateTime getCreatedAt() { return createdAt; }
}
```

```java
// File: com/company/orderservice/model/Order.java
package com.company.orderservice.model;

import java.time.LocalDateTime;

public class Order extends BaseEntity {
    private String orderNumber;  // private: only Order can access

    public Order(String orderNumber) {
        this.orderNumber = orderNumber;
        this.createdAt = LocalDateTime.now();  // OK: protected, accessible from subclass
        this.updatedAt = LocalDateTime.now();  // OK: default, same package
        // this.id = 1L;  // COMPILATION ERROR: id is private in BaseEntity
        this.setId(1L);  // OK: setId is protected, accessible from subclass
    }

    public String getOrderNumber() { return orderNumber; }
}
```

```java
// File: com/company/orderservice/service/OrderService.java
package com.company.orderservice.service;

import com.company.orderservice.model.Order;

public class OrderService {
    void processOrder(Order order) {
        Long id = order.getId();                 // OK: public getter
        String number = order.getOrderNumber();  // OK: public getter
        // order.setId(999L);                    // COMPILATION ERROR: setId is protected
        // order.orderNumber = "HACKED";         // COMPILATION ERROR: orderNumber is private
        // order.createdAt = null;               // COMPILATION ERROR: createdAt is protected,
                                                 // and OrderService is not a subclass
    }
}
```

### Example 3: Read-only and write-only properties

```java
public class ApiKey {
    private final String key;
    private final LocalDateTime createdAt;
    private LocalDateTime lastUsedAt;
    private boolean isRevoked;

    public ApiKey(String key) {
        if (key == null || key.length() < 32) {
            throw new IllegalArgumentException("API key must be at least 32 characters");
        }
        this.key = key;
        this.createdAt = LocalDateTime.now();
        this.lastUsedAt = null;
        this.isRevoked = false;
    }

    // Read-only: getter but no setter. The key cannot change after creation.
    public String getKey() {
        return key;
    }

    // Read-only: creation time is immutable.
    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    // Read-only: external code can check when the key was last used
    // but cannot set it directly.
    public LocalDateTime getLastUsedAt() {
        return lastUsedAt;
    }

    // Read-only: external code can check if the key is revoked.
    public boolean isRevoked() {
        return isRevoked;
    }

    // Business method: the only way to update lastUsedAt.
    // External code cannot set it to an arbitrary time.
    public void recordUsage() {
        if (isRevoked) {
            throw new IllegalStateException("Cannot use a revoked API key");
        }
        this.lastUsedAt = LocalDateTime.now();
    }

    // Business method: the only way to revoke a key.
    // Once revoked, it cannot be un-revoked (no setRevoked(false) exists).
    public void revoke() {
        this.isRevoked = true;
    }
}
```

### Example 4: Defensive copying

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Team {
    private String teamName;
    private List<String> members;

    public Team(String teamName, List<String> members) {
        this.teamName = teamName;
        // Defensive copy in the constructor: do not store the caller's list reference
        this.members = new ArrayList<>(members);
    }

    public String getTeamName() {
        return teamName;
    }

    // Defensive copy in the getter: return a copy, not the internal list
    public List<String> getMembers() {
        return new ArrayList<>(members);
        // Alternative: return Collections.unmodifiableList(members);
    }

    // Controlled modification through a business method
    public void addMember(String member) {
        if (member == null || member.isBlank()) {
            throw new IllegalArgumentException("Member name cannot be empty");
        }
        if (members.contains(member)) {
            throw new IllegalArgumentException(member + " is already a team member");
        }
        if (members.size() >= 10) {
            throw new IllegalStateException("Team is full (max 10 members)");
        }
        members.add(member);
    }

    public int getMemberCount() {
        return members.size();
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        List<String> initialMembers = new ArrayList<>(List.of("Alice", "Bob"));
        Team team = new Team("Backend", initialMembers);

        // Modifying the original list does NOT affect the team
        initialMembers.add("Charlie");
        System.out.println("Team members: " + team.getMembers());  // [Alice, Bob]

        // Modifying the returned list does NOT affect the team
        List<String> returnedMembers = team.getMembers();
        returnedMembers.add("Dave");
        System.out.println("Team members: " + team.getMembers());  // [Alice, Bob]

        // The only way to add members is through the business method
        team.addMember("Charlie");
        System.out.println("Team members: " + team.getMembers());  // [Alice, Bob, Charlie]
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Encapsulation is the backbone of every well-designed Spring Boot application. Here are three realistic scenarios.

### Scenario 1: JPA Entity with controlled state transitions

In a production backend, entity fields are private, and state changes go through business methods that enforce rules. Direct setters are avoided for fields that have complex transition rules.

```java
package com.company.orderservice.model;

import jakarta.persistence.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // No setter for id. The database generates it. External code reads it via getId().

    @Column(nullable = false, unique = true)
    private String orderNumber;
    // No setter for orderNumber. It is set once in the constructor and never changes.

    @Column(nullable = false)
    private Long userId;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal totalAmount;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;
    // No public setter for status. State transitions go through business methods.

    @Column(nullable = false)
    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    protected Order() {}  // For JPA

    public Order(String orderNumber, Long userId, BigDecimal totalAmount) {
        this.orderNumber = orderNumber;
        this.userId = userId;
        this.totalAmount = totalAmount;
        this.status = OrderStatus.PENDING;
        this.createdAt = LocalDateTime.now();
    }

    // --- Getters (read-only access to internal state) ---

    public Long getId() { return id; }
    public String getOrderNumber() { return orderNumber; }
    public Long getUserId() { return userId; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public OrderStatus getStatus() { return status; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }

    // --- Business methods (controlled state changes) ---

    // Instead of setStatus(OrderStatus.CONFIRMED), the caller uses confirm().
    // This method enforces the rule that only PENDING orders can be confirmed.
    public void confirm() {
        if (this.status != OrderStatus.PENDING) {
            throw new IllegalStateException(
                "Only PENDING orders can be confirmed. Current status: " + this.status
            );
        }
        this.status = OrderStatus.CONFIRMED;
        this.updatedAt = LocalDateTime.now();
    }

    public void ship() {
        if (this.status != OrderStatus.CONFIRMED) {
            throw new IllegalStateException(
                "Only CONFIRMED orders can be shipped. Current status: " + this.status
            );
        }
        this.status = OrderStatus.SHIPPED;
        this.updatedAt = LocalDateTime.now();
    }

    public void cancel() {
        if (this.status == OrderStatus.SHIPPED || this.status == OrderStatus.DELIVERED) {
            throw new IllegalStateException(
                "Cannot cancel an order that has already been shipped or delivered"
            );
        }
        if (this.status == OrderStatus.CANCELLED) {
            throw new IllegalStateException("Order is already cancelled");
        }
        this.status = OrderStatus.CANCELLED;
        this.updatedAt = LocalDateTime.now();
    }

    // The total amount can only be adjusted before the order is confirmed.
    public void adjustTotal(BigDecimal newTotal) {
        if (this.status != OrderStatus.PENDING) {
            throw new IllegalStateException(
                "Cannot adjust total after order is confirmed"
            );
        }
        if (newTotal.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Total must be positive");
        }
        this.totalAmount = newTotal;
        this.updatedAt = LocalDateTime.now();
    }
}
```

**What to notice:**

- There is no `setStatus()` method. The status can only change through `confirm()`, `ship()`, and `cancel()`. Each method enforces the valid state transitions. This makes it impossible for any caller to set the status to an invalid value.
- The `id` and `orderNumber` have getters but no setters. They are set once (by the database and the constructor, respectively) and never change. This is immutability by encapsulation.
- The `adjustTotal()` method has a business rule: you can only adjust the total while the order is still pending. Once confirmed, the total is locked. This rule is enforced inside the method, not scattered across the codebase.
- The `updatedAt` field is set automatically inside every state-changing method. Callers do not need to remember to update the timestamp. This is the power of encapsulation: the class manages its own consistency.

### Scenario 2: Configuration class with read-only access

Backend applications load configuration at startup and expose it to services through encapsulated configuration classes.

```java
package com.company.orderservice.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "app.payment")
public class PaymentConfiguration {

    private String gatewayUrl;
    private String apiKey;
    private int timeoutSeconds;
    private int maxRetries;
    private boolean sandboxMode;

    // Spring calls these setters during startup to populate the configuration
    // from application.yml. After startup, the values should not change.

    public void setGatewayUrl(String gatewayUrl) {
        this.gatewayUrl = gatewayUrl;
    }

    public void setApiKey(String apiKey) {
        this.apiKey = apiKey;
    }

    public void setTimeoutSeconds(int timeoutSeconds) {
        if (timeoutSeconds < 1 || timeoutSeconds > 60) {
            throw new IllegalArgumentException("Timeout must be between 1 and 60 seconds");
        }
        this.timeoutSeconds = timeoutSeconds;
    }

    public void setMaxRetries(int maxRetries) {
        if (maxRetries < 0 || maxRetries > 5) {
            throw new IllegalArgumentException("Max retries must be between 0 and 5");
        }
        this.maxRetries = maxRetries;
    }

    public void setSandboxMode(boolean sandboxMode) {
        this.sandboxMode = sandboxMode;
    }

    // Public getters: services can read the configuration
    public String getGatewayUrl() { return gatewayUrl; }
    public int getTimeoutSeconds() { return timeoutSeconds; }
    public int getMaxRetries() { return maxRetries; }
    public boolean isSandboxMode() { return sandboxMode; }

    // Security: the API key getter masks the value in logs
    public String getMaskedApiKey() {
        if (apiKey == null || apiKey.length() < 8) return "****";
        return apiKey.substring(0, 4) + "****" + apiKey.substring(apiKey.length() - 4);
    }

    // The raw API key is only accessible to the payment client,
    // not to logging or debugging code.
    String getApiKey() {  // Package-private: only classes in this package can access
        return apiKey;
    }
}
```

**What to notice:**

- The `apiKey` getter is package-private (no access modifier). Only the payment client class in the same package can access the raw API key. Logging code, controllers, and other services cannot accidentally log the full API key.
- The `getMaskedApiKey()` method provides a safe version for display and logging. This is encapsulation serving security.
- The setters validate the configuration values. If someone sets `timeoutSeconds` to 300 in `application.yml`, the application fails fast at startup with a clear error message instead of silently using an unreasonable timeout.

### Scenario 3: DTO with controlled serialization

```java
package com.company.orderservice.dto;

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonProperty;
import java.math.BigDecimal;
import java.time.LocalDateTime;

public class UserResponseDTO {

    private Long id;
    private String username;
    private String email;

    @JsonProperty("is_active")
    private boolean active;

    @JsonIgnore
    private String passwordHash;
    // Even if this field is accidentally populated, Jackson will not
    // include it in the JSON response. This is encapsulation at the
    // serialization layer.

    @JsonIgnore
    private String internalNotes;
    // Internal notes are visible to admins in the database but should
    // never be sent to the end user through the API.

    public UserResponseDTO(Long id, String username, String email, boolean active) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.active = active;
        // passwordHash and internalNotes are intentionally not set
    }

    // Getters for JSON serialization
    public Long getId() { return id; }
    public String getUsername() { return username; }
    public String getEmail() { return email; }
    public boolean isActive() { return active; }

    // No setters. This DTO is read-only. It is created once and sent to the client.
    // Making it immutable prevents accidental modification during request processing.
}
```

**What to notice:**

- The `@JsonIgnore` annotation prevents sensitive fields from appearing in the API response, even if they are populated. This is a form of encapsulation enforced by the serialization framework.
- The DTO has getters but no setters. It is a read-only data carrier. Once created, its state cannot change. This prevents bugs where a service accidentally modifies a DTO after it has been partially processed.
- The `@JsonProperty("is_active")` annotation controls the JSON field name. The internal Java field is `active`, but the API exposes it as `is_active` to match the frontend's naming convention. This is encapsulation allowing the internal representation to differ from the external API.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Making all fields public

**Wrong:**

```java
public class User {
    public String username;
    public String email;
    public String passwordHash;
    public boolean isAdmin;
}

// Any code can do this:
user.passwordHash = "plaintext123";  // Security disaster
user.isAdmin = true;                  // Privilege escalation
```

**Right:**

```java
public class User {
    private String username;
    private String email;
    private String passwordHash;
    private boolean isAdmin;

    // Controlled access through methods
    public void changePassword(String oldPassword, String newPassword) {
        if (!verifyPassword(oldPassword)) {
            throw new SecurityException("Incorrect current password");
        }
        this.passwordHash = hashPassword(newPassword);
    }

    public void grantAdmin() {
        // Only callable by authorized service methods, not by any random code
        this.isAdmin = true;
    }
}
```

**Why it is wrong:** Public fields provide zero encapsulation. Any code in the application can read or write any field without validation, logging, or authorization checks. This is a security vulnerability and a maintenance nightmare. Fields should be `private` by default.

### Mistake 2: Generating getters and setters for every field without thinking

**Wrong:**

```java
public class Order {
    private Long id;
    private String orderNumber;
    private OrderStatus status;
    private LocalDateTime createdAt;

    // IDE-generated getters and setters for ALL fields.
    // Now any code can call order.setId(999L) or order.setCreatedAt(null).
    // The encapsulation is meaningless because the setters do no validation.
    public void setId(Long id) { this.id = id; }
    public void setOrderNumber(String orderNumber) { this.orderNumber = orderNumber; }
    public void setStatus(OrderStatus status) { this.status = status; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

**Right:**

```java
public class Order {
    private Long id;
    private String orderNumber;
    private OrderStatus status;
    private LocalDateTime createdAt;

    // Only provide getters for fields that external code needs to read.
    public Long getId() { return id; }
    public String getOrderNumber() { return orderNumber; }
    public OrderStatus getStatus() { return status; }
    public LocalDateTime getCreatedAt() { return createdAt; }

    // No setters for id, orderNumber, createdAt. They are immutable after creation.
    // Status changes through business methods, not a generic setter.
    public void confirm() { /* ... */ }
    public void cancel() { /* ... */ }
}
```

**Why it is wrong:** Blindly generating getters and setters for every field gives the illusion of encapsulation without the substance. If a setter does nothing but assign the value, it is functionally identical to a public field. Think about which fields should be readable, which should be writable, and which should be immutable. Only provide the access that is actually needed.

### Mistake 3: Exposing mutable internal state through getters

**Wrong:**

```java
public class ShoppingCart {
    private List<OrderItem> items = new ArrayList<>();

    public List<OrderItem> getItems() {
        return items;  // Returns the internal list directly!
    }
}

// External code can bypass all validation:
cart.getItems().clear();  // Empties the cart without going through removeItem()
cart.getItems().add(new OrderItem(null, -1, BigDecimal.ZERO));  // Adds invalid item
```

**Right:**

```java
public class ShoppingCart {
    private List<OrderItem> items = new ArrayList<>();

    public List<OrderItem> getItems() {
        return Collections.unmodifiableList(items);
        // Or: return new ArrayList<>(items);
    }

    public void addItem(OrderItem item) {
        // Validation happens here
        if (item == null || item.getQuantity() <= 0) {
            throw new IllegalArgumentException("Invalid item");
        }
        items.add(item);
    }
}
```

**Why it is wrong:** Returning a mutable internal reference from a getter completely breaks encapsulation. External code can modify the internal state without going through your validation methods. This is especially dangerous for collections (`List`, `Set`, `Map`), arrays, and mutable objects like `Date`. Always return a defensive copy or an unmodifiable view.

### Mistake 4: Using public when protected or private would suffice

**Wrong:**

```java
public class OrderService {
    public OrderRepository orderRepository;  // Public field! Anyone can access it.
    public PaymentService paymentService;

    public void validateOrder(Order order) {  // Public method called only internally
        // ...
    }
}
```

**Right:**

```java
public class OrderService {
    private final OrderRepository orderRepository;  // Private, injected via constructor
    private final PaymentService paymentService;

    private void validateOrder(Order order) {  // Private: only used internally
        // ...
    }
}
```

**Why it is wrong:** Every public member is a commitment. External code can depend on it, and changing or removing it becomes a breaking change. By making members more restrictive than necessary, you retain the freedom to refactor your code without affecting other classes. Follow the principle of least privilege.

---

## Key Takeaways

> [!tip] Remember these points

- Encapsulation means bundling data and behavior inside a class and restricting direct access to the internal state. It is achieved through `private` fields and `public` methods (getters, setters, and business methods).
- Java has four access levels: `private` (class only), default/package-private (class + package), `protected` (class + package + subclasses), and `public` (everywhere). Use the most restrictive level that meets your needs.
- Getters provide read access to private fields. Setters provide write access with validation. Not every field needs both. Immutable fields should have only a getter (or neither). Boolean getters use the `is` prefix.
- Defensive copying is essential for getters that return mutable reference types (lists, arrays, objects). Return a copy or an unmodifiable view to prevent external code from modifying your internal state.
- Prefer business methods over generic setters for complex state changes. `order.confirm()` is better than `order.setStatus(CONFIRMED)` because the method name expresses intent and the method body enforces transition rules.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Bank Account with Full Encapsulation (Easy)

Create a `BankAccount` class with the following encapsulated fields:

- `accountNumber` (String, private, read-only after construction)
- `holderName` (String, private, can be updated with validation)
- `balance` (double, private, cannot be set directly)
- `isActive` (boolean, private)

Provide:

- A constructor that validates the account number (must start with `"ACC-"`) and initial deposit (minimum 500 BDT).
- Getters for all fields. Use `isActive()` for the boolean.
- A setter for `holderName` that validates the name is at least 2 characters.
- Business methods: `deposit(double amount)`, `withdraw(double amount)`, `deactivate()`.
- No setter for `balance`, `accountNumber`, or `isActive`. These change only through business methods.

Test by creating an account, making deposits and withdrawals, and verifying that invalid operations throw exceptions.

> **Hint:** The `withdraw()` method should check for sufficient funds and throw `IllegalStateException` if the balance is too low. The `deactivate()` method should set `isActive` to false and prevent further transactions.

### Exercise 2: Immutable Configuration Class (Medium)

Create an `AppConfig` class where all fields are `private final` and set only through the constructor. Fields: `appName` (String), `version` (String), `maxConnections` (int), `debugMode` (boolean). The constructor should validate:

- `appName` is not null or blank.
- `version` matches the pattern `"X.Y.Z"` (e.g., `"2.1.0"`). You can use a simple check: split by `"."` and verify there are exactly 3 numeric parts.
- `maxConnections` is between 1 and 1000.

Provide getters for all fields. No setters. The object's state cannot change after creation.

> **Hint:** Use `String.split("\\.")` to split the version string. Check that the resulting array has length 3 and each part is a valid integer using `Integer.parseInt()`.

### Exercise 3: Defensive Copying Demonstration (Medium)

Create a `StudentRecord` class with fields: `studentId` (String), `grades` (int array, private). Provide:

- A constructor that takes a String ID and an `int[]` of grades. Make a defensive copy of the array in the constructor.
- A getter `getGrades()` that returns a defensive copy of the array.
- A method `addGrade(int grade)` that validates the grade is between 0 and 100 before adding it.
- A method `getAverage()` that calculates the average grade.

In `main()`, prove that modifying the original array after construction does not affect the `StudentRecord`, and modifying the returned array from `getGrades()` does not affect the internal state.

> **Hint:** Use `Arrays.copyOf()` for defensive copying of arrays. To add a grade, create a new array one element larger, copy the old elements, and add the new grade.

### Exercise 4: Access Modifier Challenge (Hard, Optional)

Create a mini package structure to demonstrate all four access modifiers:

**Package `com.company.model`:**

- `BaseEntity` class with `private Long id`, `protected LocalDateTime createdAt`, `String createdBy` (default), `public String getAuditInfo()`.
- `Order` class extending `BaseEntity` with `private String orderNumber`. Try to access all inherited fields from within `Order` and note which ones compile.

**Package `com.company.service`:**

- `OrderService` class that creates an `Order` object. Try to access all fields and methods of `Order` and `BaseEntity` from this different package. Note which ones compile.

Write comments explaining why each access attempt succeeds or fails.

> **Hint:** You will need to create the package directories in your project: `src/com/company/model/` and `src/com/company/service/`. In IntelliJ, right-click the `src` folder and select New > Package.

<details>
<summary>Solution for Exercise 1</summary>

```java
public class BankAccount {
    private final String accountNumber;
    private String holderName;
    private double balance;
    private boolean isActive;

    public BankAccount(String accountNumber, String holderName, double initialDeposit) {
        if (accountNumber == null || !accountNumber.startsWith("ACC-")) {
            throw new IllegalArgumentException("Account number must start with 'ACC-'");
        }
        if (holderName == null || holderName.strip().length() < 2) {
            throw new IllegalArgumentException("Holder name must be at least 2 characters");
        }
        if (initialDeposit < 500) {
            throw new IllegalArgumentException("Minimum initial deposit is 500 BDT");
        }

        this.accountNumber = accountNumber;
        this.holderName = holderName.strip();
        this.balance = initialDeposit;
        this.isActive = true;
    }

    public String getAccountNumber() { return accountNumber; }
    public String getHolderName() { return holderName; }
    public double getBalance() { return balance; }
    public boolean isActive() { return isActive; }

    public void setHolderName(String holderName) {
        if (holderName == null || holderName.strip().length() < 2) {
            throw new IllegalArgumentException("Holder name must be at least 2 characters");
        }
        this.holderName = holderName.strip();
    }

    public void deposit(double amount) {
        checkActive();
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        this.balance += amount;
    }

    public void withdraw(double amount) {
        checkActive();
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        if (amount > this.balance) {
            throw new IllegalStateException(
                "Insufficient funds. Balance: " + this.balance + " BDT"
            );
        }
        this.balance -= amount;
    }

    public void deactivate() {
        this.isActive = false;
    }

    private void checkActive() {
        if (!this.isActive) {
            throw new IllegalStateException("Account is deactivated");
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
public class AppConfig {
    private final String appName;
    private final String version;
    private final int maxConnections;
    private final boolean debugMode;

    public AppConfig(String appName, String version, int maxConnections, boolean debugMode) {
        if (appName == null || appName.isBlank()) {
            throw new IllegalArgumentException("App name cannot be empty");
        }

        if (!isValidVersion(version)) {
            throw new IllegalArgumentException(
                "Version must match X.Y.Z format. Received: " + version
            );
        }

        if (maxConnections < 1 || maxConnections > 1000) {
            throw new IllegalArgumentException(
                "Max connections must be between 1 and 1000. Received: " + maxConnections
            );
        }

        this.appName = appName.strip();
        this.version = version;
        this.maxConnections = maxConnections;
        this.debugMode = debugMode;
    }

    private static boolean isValidVersion(String version) {
        if (version == null) return false;
        String[] parts = version.split("\\.");
        if (parts.length != 3) return false;
        for (String part : parts) {
            try {
                int num = Integer.parseInt(part);
                if (num < 0) return false;
            } catch (NumberFormatException e) {
                return false;
            }
        }
        return true;
    }

    public String getAppName() { return appName; }
    public String getVersion() { return version; }
    public int getMaxConnections() { return maxConnections; }
    public boolean isDebugMode() { return debugMode; }
}
```

</details>

---

## Related Notes

- [[Java - Constructors - Default Parameterized Copy]]
- [[Java - Inheritance - Single Multilevel Hierarchical]] (next note)
- [[Java - Static Keyword - Variables Methods Blocks]]

---

## Resources

- **Oracle Java Tutorials: Controlling Access to Members of a Class** - Official documentation on all four access modifiers with a summary table.
- **Oracle Java Tutorials: Encapsulation** - Official explanation of encapsulation with getter/setter examples.
- **Baeldung: Java Access Modifiers** - Comprehensive guide with code examples for each modifier level.
- **Baeldung: Getters and Setters in Java** - Detailed discussion of when to use getters/setters and when to avoid them.
- **Effective Java by Joshua Bloch - Item 15: Minimize the Accessibility of Classes and Members** - The definitive guide on choosing access modifiers. Essential reading for backend engineers.
- **Effective Java by Joshua Bloch - Item 50: Make Defensive Copies When Needed** - Deep dive into defensive copying in constructors and getters.
