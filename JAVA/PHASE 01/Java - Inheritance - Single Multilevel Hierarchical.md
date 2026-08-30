---
title: "Java - Inheritance - Single Multilevel Hierarchical"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - inheritance
  - extends
  - super
status: "not-started"
---

# Java - Inheritance - Single Multilevel Hierarchical

> [!abstract] Overview
> Inheritance is the OOP mechanism that allows a class (the child or subclass) to acquire the fields and methods of another class (the parent or superclass). It models "is-a" relationships: an `Order` is a `BaseEntity`, a `PaymentException` is a `RuntimeException`, a `SavingsAccount` is a `BankAccount`. Java supports single inheritance (one parent per class), multilevel inheritance (chains of parent-child relationships), and hierarchical inheritance (one parent with multiple children). In backend development, inheritance is the foundation of exception hierarchies, entity base classes, Spring's framework extension points, and the entire Java Collections Framework.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Constructors - Default Parameterized Copy]]
> - [[Java - Encapsulation - Getters Setters Access Modifiers]]

---

## Theory

### What is Inheritance?

Inheritance is a mechanism where a new class is built on top of an existing class, automatically acquiring all of its non-private fields and methods. The existing class is called the **superclass** (or parent class or base class). The new class is called the **subclass** (or child class or derived class).

```java
// Superclass (parent)
public class Animal {
    String name;
    int age;

    void eat() {
        System.out.println(name + " is eating");
    }
}

// Subclass (child): inherits name, age, and eat() from Animal
public class Dog extends Animal {
    String breed;

    void bark() {
        System.out.println(name + " is barking");
        // 'name' is inherited from Animal. Dog does not declare it.
    }
}
```

The `extends` keyword establishes the inheritance relationship. When you write `class Dog extends Animal`, you are saying "a Dog is an Animal." The `Dog` class automatically gets the `name` field, the `age` field, and the `eat()` method without having to declare them again. It then adds its own `breed` field and `bark()` method on top.

**What is inherited:**
- All `public` and `protected` fields and methods.
- All default (package-private) fields and methods if the subclass is in the same package.
- Constructors are NOT inherited (but the parent constructor is called via `super()`).
- `private` fields and methods are NOT directly accessible in the subclass, but they still exist in the object's memory. The subclass can access them indirectly through inherited public or protected methods.

### Why Does Inheritance Exist?

Inheritance solves the problem of **code duplication** when multiple classes share common behavior. Without inheritance, you would copy and paste the same fields and methods into every class that needs them.

Consider a backend system with three entity types: `Order`, `User`, and `Product`. All three have an `id`, a `createdAt` timestamp, and an `updatedAt` timestamp. Without inheritance:

```java
// Without inheritance: 3 classes, each with duplicated fields
public class Order {
    Long id; LocalDateTime createdAt; LocalDateTime updatedAt;
    String orderNumber; BigDecimal total;
}
public class User {
    Long id; LocalDateTime createdAt; LocalDateTime updatedAt;
    String username; String email;
}
public class Product {
    Long id; LocalDateTime createdAt; LocalDateTime updatedAt;
    String name; double price;
}
```

If you need to add an `isActive` field to all entities, you must modify three classes. If you fix a bug in the timestamp logic, you must fix it in three places. This violates the **DRY principle** (Don't Repeat Yourself).

With inheritance:

```java
// With inheritance: common fields in one place
public class BaseEntity {
    Long id;
    LocalDateTime createdAt;
    LocalDateTime updatedAt;
}

public class Order extends BaseEntity {
    String orderNumber;
    BigDecimal total;
}
public class User extends BaseEntity {
    String username;
    String email;
}
public class Product extends BaseEntity {
    String name;
    double price;
}
```

Now adding `isActive` requires changing only `BaseEntity`. The change automatically propagates to all three subclasses.

### Types of Inheritance in Java

**1. Single Inheritance**: One subclass extends one superclass. This is the most common form.

```text
Animal
  |
  Dog
```

```java
class Dog extends Animal { }
```

**2. Multilevel Inheritance**: A chain of inheritance where a subclass becomes the superclass of another subclass.

```text
BaseEntity
    |
  Order
    |
InternationalOrder
```

```java
class BaseEntity { }
class Order extends BaseEntity { }
class InternationalOrder extends Order { }
// InternationalOrder inherits from Order AND BaseEntity
```

**3. Hierarchical Inheritance**: One superclass has multiple subclasses.

```text
    BaseEntity
    /    |    \
 Order  User  Product
```

```java
class BaseEntity { }
class Order extends BaseEntity { }
class User extends BaseEntity { }
class Product extends BaseEntity { }
```

> [!warning] Java Does NOT Support Multiple Inheritance of Classes
> A Java class can extend only ONE superclass. You cannot write `class Dog extends Animal, Pet`. This restriction exists because multiple inheritance creates the **diamond problem**: if both `Animal` and `Pet` define a method `getName()`, which one does `Dog` inherit? Different languages solve this differently (C++ uses virtual inheritance, Python uses MRO). Java avoids the problem entirely by allowing only single class inheritance. However, Java does support multiple inheritance of **interfaces**, which you will learn in the Abstraction note.

### The `super` Keyword

The `super` keyword refers to the immediate parent class. It has three uses:

**1. Calling the parent constructor**: `super(arguments)` must be the first statement in the subclass constructor.

```java
public class Dog extends Animal {
    String breed;

    Dog(String name, int age, String breed) {
        super(name, age);  // Calls Animal's constructor. Must be first!
        this.breed = breed;
    }
}
```

If you do not write `super()` explicitly, the compiler inserts `super()` (no arguments) automatically. This means the parent class must have a no-argument constructor (either explicit or compiler-generated). If the parent class only has parameterized constructors, you must call one of them explicitly with `super(arguments)`.

**2. Calling a parent method**: `super.methodName()` calls the parent's version of a method, even if the subclass has overridden it.

```java
public class Dog extends Animal {
    @Override
    void makeSound() {
        super.makeSound();  // Calls Animal's makeSound() first
        System.out.println("Bark!");  // Then adds Dog-specific behavior
    }
}
```

**3. Accessing a parent field**: `super.fieldName` accesses the parent's version of a field when the subclass has a field with the same name (field shadowing).

```java
public class Dog extends Animal {
    String name;  // Shadows Animal's 'name' field

    void printNames() {
        System.out.println("Dog name: " + this.name);
        System.out.println("Animal name: " + super.name);
    }
}
```

### Method Overriding

Method overriding occurs when a subclass provides its own implementation of a method that is already defined in its superclass. The method in the subclass must have the **same name, same parameter list, and same return type** (or a covariant return type, which is a subclass of the original return type).

```java
public class Animal {
    void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

public class Dog extends Animal {
    @Override  // This annotation is optional but strongly recommended
    void makeSound() {
        System.out.println("Woof!");
    }
}

public class Cat extends Animal {
    @Override
    void makeSound() {
        System.out.println("Meow!");
    }
}
```

**Rules for overriding:**

1. The method name, parameter list, and return type must match the parent method exactly (or the return type must be a subtype).
2. The access modifier in the subclass cannot be more restrictive than in the parent. If the parent method is `protected`, the override can be `protected` or `public`, but not `private` or default.
3. The override cannot throw new or broader checked exceptions than the parent method. It can throw fewer or narrower exceptions.
4. `static` methods cannot be overridden. If a subclass defines a static method with the same signature, it **hides** the parent's method, which is a different mechanism.
5. `final` methods cannot be overridden.
6. `private` methods cannot be overridden because they are not visible to the subclass.

**The `@Override` annotation**: While optional, you should always use `@Override` on methods that are intended to override a parent method. The compiler checks that the method actually overrides something. If you misspell the method name or get the parameter types wrong, the compiler will catch the error. Without `@Override`, the compiler silently creates a new method instead of overriding the parent's, leading to subtle bugs.

### The `Object` Class

Every class in Java implicitly extends `java.lang.Object`, the root of the entire class hierarchy. Even if you do not write `extends`, your class inherits from `Object`.

```java
public class Student { }
// Is equivalent to:
public class Student extends Object { }
```

The `Object` class provides several methods that every Java object inherits:

| Method | Purpose |
|--------|---------|
| `toString()` | Returns a string representation of the object. Default: `ClassName@hashCode`. |
| `equals(Object)` | Checks if two objects are equal. Default: reference equality (`==`). |
| `hashCode()` | Returns an integer hash code for the object. Used by HashMap and HashSet. |
| `getClass()` | Returns the runtime class of the object. |
| `clone()` | Creates a copy of the object (requires implementing `Cloneable`). |
| `finalize()` | Called before garbage collection (deprecated since Java 9). |

In backend development, you will override `toString()`, `equals()`, and `hashCode()` frequently. JPA entities, DTOs, and domain objects all need proper implementations of these methods.

> [!note] Missing imports in this section's examples
> `equals()`/`hashCode()` examples use `Objects.hash(...)`, which requires `import java.util.Objects;`. The `LocalDateTime` examples require `import java.time.LocalDateTime;`, and `BigDecimal` requires `import java.math.BigDecimal;`.

### The `final` Keyword in Inheritance

The `final` keyword restricts inheritance and overriding:

- **`final` class**: Cannot be subclassed. `public final class String { }` means no class can extend `String`.
- **`final` method**: Cannot be overridden by subclasses. The subclass inherits the method but cannot change its behavior.
- **`final` field**: Cannot be reassigned after initialization (covered in the Variables note).

```java
public final class PaymentReceipt {
    // No class can extend PaymentReceipt.
    // This is useful for security-sensitive or immutable classes.
}

public class BaseEntity {
    public final Long getId() {
        return id;  // Subclasses cannot override getId(). The ID logic is locked.
    }
}
```

### How Inheritance Works Internally

When the JVM loads a subclass, it performs the following steps:

1. **Loads the superclass first**. The superclass must be fully loaded and initialized before the subclass. This happens recursively up to `Object`.

2. **Creates the object layout**. The JVM allocates memory for all fields from the entire inheritance chain. A `Dog` object contains the fields declared in `Animal` followed by the fields declared in `Dog`. The memory layout is a single contiguous block, not separate objects.

```text
Dog object in memory:
+-------------------+
| Animal fields     |  (name, age)
+-------------------+
| Dog fields        |  (breed)
+-------------------+
| Object header     |  (class pointer, lock info, GC metadata)
+-------------------+
```

3. **Resolves method calls using a virtual method table (vtable)**. Each class has a vtable that maps method signatures to their actual implementations. When you call `animal.makeSound()`, the JVM looks up the vtable of the object's actual runtime class (which might be `Dog`, not `Animal`) and dispatches to the correct implementation. This is how polymorphism works at the JVM level, and you will study it in detail in the Polymorphism note.

4. **Executes constructors from top to bottom**. When you create a `new Dog()`, the JVM first calls `Object()`'s constructor, then `Animal()`'s constructor, then `Dog()`'s constructor. This ensures that the parent's state is fully initialized before the child's constructor runs.

> [!tip] Key Insight
> Inheritance creates an "is-a" relationship, not a "has-a" relationship. A `Dog` IS an `Animal`. A `Car` HAS an `Engine` (composition, not inheritance). A common mistake is using inheritance when composition is more appropriate. For example, a `User` should not extend `Address`. A user HAS an address; a user IS NOT an address. Use inheritance for true type hierarchies and composition for part-whole relationships. You will learn composition in detail in later notes.

---

## Syntax and Basic Examples

### Example 1: Single inheritance with constructor chaining

```java
public class Animal {
    String name;
    int age;

    Animal(String name, int age) {
        this.name = name;
        this.age = age;
        System.out.println("Animal constructor: " + name);
    }

    void eat() {
        System.out.println(name + " is eating");
    }

    void makeSound() {
        System.out.println(name + " makes a generic sound");
    }

    @Override
    public String toString() {
        return "Animal{name='" + name + "', age=" + age + "}";
    }
}
```

```java
public class Dog extends Animal {
    String breed;
    boolean isTrained;

    Dog(String name, int age, String breed, boolean isTrained) {
        super(name, age);  // MUST be first. Calls Animal(String, int).
        this.breed = breed;
        this.isTrained = isTrained;
        System.out.println("Dog constructor: " + breed);
    }

    @Override
    void makeSound() {
        System.out.println(name + " says: Woof! Woof!");
    }

    void fetch() {
        if (isTrained) {
            System.out.println(name + " fetches the ball!");
        } else {
            System.out.println(name + " is not trained to fetch");
        }
    }

    @Override
    public String toString() {
        return "Dog{name='" + name + "', age=" + age
            + ", breed='" + breed + "', trained=" + isTrained + "}";
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Dog rex = new Dog("Rex", 3, "German Shepherd", true);
        // Output:
        // Animal constructor: Rex
        // Dog constructor: German Shepherd

        rex.eat();        // Inherited from Animal: "Rex is eating"
        rex.makeSound();  // Overridden in Dog: "Rex says: Woof! Woof!"
        rex.fetch();      // Defined in Dog: "Rex fetches the ball!"

        System.out.println(rex);  // Uses Dog's toString()
    }
}
```

**Output:**

```text
Animal constructor: Rex
Dog constructor: German Shepherd
Rex is eating
Rex says: Woof! Woof!
Rex fetches the ball!
Dog{name='Rex', age=3, breed='German Shepherd', trained=true}
```

### Example 2: Multilevel inheritance

```java
public class BaseEntity {
    protected Long id;
    protected LocalDateTime createdAt;
    protected LocalDateTime updatedAt;

    BaseEntity() {
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }

    public Long getId() { return id; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }

    protected void touch() {
        this.updatedAt = LocalDateTime.now();
    }
}
```

```java
public class Order extends BaseEntity {
    private String orderNumber;
    private BigDecimal totalAmount;
    private OrderStatus status;

    Order(String orderNumber, BigDecimal totalAmount) {
        super();  // Calls BaseEntity() constructor
        this.orderNumber = orderNumber;
        this.totalAmount = totalAmount;
        this.status = OrderStatus.PENDING;
    }

    public void confirm() {
        this.status = OrderStatus.CONFIRMED;
        this.touch();  // Inherited from BaseEntity: updates the updatedAt timestamp
    }

    public String getOrderNumber() { return orderNumber; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public OrderStatus getStatus() { return status; }
}
```

```java
public class InternationalOrder extends Order {
    private String destinationCountry;
    private BigDecimal customsDuty;

    InternationalOrder(String orderNumber, BigDecimal totalAmount,
                       String destinationCountry, BigDecimal customsDuty) {
        super(orderNumber, totalAmount);  // Calls Order() constructor
        this.destinationCountry = destinationCountry;
        this.customsDuty = customsDuty;
    }

    public BigDecimal getTotalWithDuty() {
        return this.getTotalAmount().add(this.customsDuty);
        // getTotalAmount() is inherited from Order, which inherited id from BaseEntity.
        // Three levels of inheritance working together.
    }

    public String getDestinationCountry() { return destinationCountry; }
}
```

```java
public class Main {
    public static void main(String[] args) {
        InternationalOrder order = new InternationalOrder(
            "INT-001", new BigDecimal("5000"), "Bangladesh", new BigDecimal("750")
        );

        System.out.println("Order: " + order.getOrderNumber());
        System.out.println("Subtotal: " + order.getTotalAmount());
        System.out.println("Customs: " + order.getDestinationCountry());
        System.out.println("Total with duty: " + order.getTotalWithDuty());
        System.out.println("Created at: " + order.getCreatedAt());  // From BaseEntity
        System.out.println("Status: " + order.getStatus());          // From Order

        order.confirm();  // From Order, which calls touch() from BaseEntity
        System.out.println("Updated at: " + order.getUpdatedAt());
    }
}
```

### Example 3: Hierarchical inheritance

```java
public class Notification {
    protected String recipient;
    protected String message;
    protected LocalDateTime sentAt;

    Notification(String recipient, String message) {
        this.recipient = recipient;
        this.message = message;
    }

    void send() {
        this.sentAt = LocalDateTime.now();
        System.out.println("Sending notification to " + recipient);
    }

    boolean isSent() {
        return sentAt != null;
    }
}
```

```java
public class EmailNotification extends Notification {
    private String subject;

    EmailNotification(String email, String subject, String message) {
        super(email, message);  // Calls Notification constructor
        this.subject = subject;
    }

    @Override
    void send() {
        super.send();  // Call parent's send() first
        System.out.println("  Email Subject: " + subject);
        System.out.println("  Email Body: " + message);
        System.out.println("  Sent via SMTP to " + recipient);
    }
}
```

```java
public class SmsNotification extends Notification {
    private String phoneNumber;

    SmsNotification(String phoneNumber, String message) {
        super(phoneNumber, message);
        this.phoneNumber = phoneNumber;
    }

    @Override
    void send() {
        super.send();
        System.out.println("  SMS: " + message);
        System.out.println("  Sent via SMS gateway to " + phoneNumber);
    }
}
```

```java
public class PushNotification extends Notification {
    private String deviceToken;

    PushNotification(String deviceToken, String message) {
        super(deviceToken, message);
        this.deviceToken = deviceToken;
    }

    @Override
    void send() {
        super.send();
        System.out.println("  Push: " + message);
        System.out.println("  Sent via FCM to device " + deviceToken);
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Notification[] notifications = {
            new EmailNotification("saad@example.com", "Order Confirmed", "Your order #123 is confirmed."),
            new SmsNotification("+8801712345678", "Your OTP is 4829."),
            new PushNotification("device_token_abc", "Your order has been shipped!")
        };

        for (Notification n : notifications) {
            n.send();
            System.out.println("  Sent: " + n.isSent());
            System.out.println();
        }
    }
}
```

**Output:**

```text
Sending notification to saad@example.com
  Email Subject: Order Confirmed
  Email Body: Your order #123 is confirmed.
  Sent via SMTP to saad@example.com
  Sent: true

Sending notification to +8801712345678
  SMS: Your OTP is 4829.
  Sent via SMS gateway to +8801712345678
  Sent: true

Sending notification to device_token_abc
  Push: Your order has been shipped!
  Sent via FCM to device token_abc
  Sent: true
```

### Example 4: Overriding `equals()`, `hashCode()`, and `toString()`

```java
import java.math.BigDecimal;
import java.util.Objects;

public class Money {
    private BigDecimal amount;
    private String currency;

    public Money(BigDecimal amount, String currency) {
        this.amount = amount;
        this.currency = currency.toUpperCase();
    }

    // Override toString() for readable output
    @Override
    public String toString() {
        return amount + " " + currency;
    }

    // Override equals() for value-based comparison
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;           // Same reference
        if (obj == null) return false;           // Null check
        if (getClass() != obj.getClass()) return false;  // Same class check

        Money other = (Money) obj;              // Cast to Money
        return amount.compareTo(other.amount) == 0
            && currency.equals(other.currency);
    }

    // Override hashCode() whenever you override equals()
    // Contract: equal objects must have equal hash codes
    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }

    public BigDecimal getAmount() { return amount; }
    public String getCurrency() { return currency; }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Money price1 = new Money(new BigDecimal("1500.00"), "BDT");
        Money price2 = new Money(new BigDecimal("1500.00"), "BDT");
        Money price3 = new Money(new BigDecimal("2000.00"), "BDT");

        System.out.println(price1);  // 1500.00 BDT (toString)
        System.out.println(price1.equals(price2));  // true (same value)
        System.out.println(price1.equals(price3));  // false (different amount)
        System.out.println(price1 == price2);       // false (different objects)
        System.out.println(price1.hashCode() == price2.hashCode());  // true
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Inheritance is deeply embedded in every Spring Boot application. Here are three realistic scenarios.

### Scenario 1: Exception hierarchy for structured error handling

Every well-designed backend defines a custom exception hierarchy that extends Java's built-in exception classes. This allows the global exception handler to catch exceptions at different levels of specificity.

```java
package com.company.orderservice.exception;

// Base exception for the entire application.
// Extends RuntimeException so it is an unchecked exception
// (callers are not forced to catch it with try-catch).
public class AppException extends RuntimeException {
    private final int httpStatusCode;
    private final String errorCode;

    public AppException(String message, int httpStatusCode, String errorCode) {
        super(message);  // Calls RuntimeException's constructor
        this.httpStatusCode = httpStatusCode;
        this.errorCode = errorCode;
    }

    public AppException(String message, int httpStatusCode, String errorCode, Throwable cause) {
        super(message, cause);  // Passes the cause to the parent for stack trace chaining
        this.httpStatusCode = httpStatusCode;
        this.errorCode = errorCode;
    }

    public int getHttpStatusCode() { return httpStatusCode; }
    public String getErrorCode() { return errorCode; }
}
```

```java
package com.company.orderservice.exception;

// Specific exception for "resource not found" errors.
// Inherits httpStatusCode and errorCode from AppException.
public class ResourceNotFoundException extends AppException {
    public ResourceNotFoundException(String resourceName, Long id) {
        super(
            resourceName + " not found with id: " + id,
            404,
            "RESOURCE_NOT_FOUND"
        );
    }

    public ResourceNotFoundException(String resourceName, String field, String value) {
        super(
            resourceName + " not found with " + field + ": " + value,
            404,
            "RESOURCE_NOT_FOUND"
        );
    }
}
```

```java
package com.company.orderservice.exception;

// Specific exception for business rule violations.
public class BusinessRuleException extends AppException {
    public BusinessRuleException(String message) {
        super(message, 422, "BUSINESS_RULE_VIOLATION");
    }
}
```

```java
package com.company.orderservice.exception;

// Specific exception for payment failures.
// Extends BusinessRuleException, creating a multilevel hierarchy:
// RuntimeException -> AppException -> BusinessRuleException -> PaymentException
public class PaymentException extends BusinessRuleException {
    private final String transactionId;

    public PaymentException(String message, String transactionId) {
        super(message);  // Calls BusinessRuleException -> AppException -> RuntimeException
        this.transactionId = transactionId;
    }

    public String getTransactionId() { return transactionId; }
}
```

```java
// The global exception handler catches exceptions at the appropriate level.
// Because of inheritance, catching AppException catches ALL custom exceptions.
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(
            new ErrorResponse(ex.getErrorCode(), ex.getMessage())
        );
    }

    @ExceptionHandler(PaymentException.class)
    public ResponseEntity<ErrorResponse> handlePayment(PaymentException ex) {
        // Specific handling for payment failures: log the transaction ID
        // logger.error("Payment failed: txn={}", ex.getTransactionId());
        return ResponseEntity.status(422).body(
            new ErrorResponse(ex.getErrorCode(), ex.getMessage())
        );
    }

    @ExceptionHandler(AppException.class)
    public ResponseEntity<ErrorResponse> handleAppException(AppException ex) {
        // Catch-all for any other custom exception
        return ResponseEntity.status(ex.getHttpStatusCode()).body(
            new ErrorResponse(ex.getErrorCode(), ex.getMessage())
        );
    }
}
```

**What to notice:**

- The exception hierarchy is: `RuntimeException` -> `AppException` -> `BusinessRuleException` -> `PaymentException`. Each level adds specificity. The global handler can catch exceptions at any level.
- `super(message)` chains the constructor calls up the hierarchy. Each constructor passes the message to its parent, which passes it to `RuntimeException`, which passes it to `Exception`, which passes it to `Throwable`. The entire chain is initialized correctly.
- The `AppException` constructor with a `Throwable cause` parameter enables **exception chaining**. When a low-level exception (like a database `SQLException`) causes a high-level exception (like `PaymentException`), you pass the original exception as the cause. The stack trace then shows both exceptions, making debugging much easier.

### Scenario 2: Mapped Superclass in JPA

JPA provides the `@MappedSuperclass` annotation for base entity classes that share common fields but are not entities themselves (they do not have their own database table).

```java
package com.company.orderservice.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;

// @MappedSuperclass means: "These fields will be inherited by subclasses
// and mapped to columns in the subclass's table. But BaseEntity itself
// does not have a table in the database."
@MappedSuperclass
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(nullable = false)
    private LocalDateTime updatedAt;

    @Column(nullable = false)
    private boolean isActive = true;

    @PrePersist
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        this.updatedAt = LocalDateTime.now();
    }

    // Getters
    public Long getId() { return id; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public boolean isActive() { return isActive; }

    // Controlled mutation
    public void deactivate() {
        this.isActive = false;
        this.onUpdate();
    }
}
```

```java
@Entity
@Table(name = "orders")
public class Order extends BaseEntity {
    // Inherits: id, createdAt, updatedAt, isActive
    // These columns appear in the 'orders' table automatically.

    @Column(nullable = false, unique = true)
    private String orderNumber;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal totalAmount;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    // Constructor, getters, business methods...
}
```

```java
@Entity
@Table(name = "users")
public class User extends BaseEntity {
    // Inherits: id, createdAt, updatedAt, isActive
    // These columns appear in the 'users' table automatically.

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String username;

    @Column(nullable = false)
    private String passwordHash;

    // Constructor, getters, business methods...
}
```

**What to notice:**

- `BaseEntity` is `abstract`. You cannot create a `new BaseEntity()` because it represents a concept, not a concrete entity. Only concrete subclasses like `Order` and `User` can be instantiated.
- The `@PrePersist` and `@PreUpdate` annotations are JPA lifecycle callbacks. They are inherited by all subclasses. When any entity is saved to the database, the `onCreate()` method runs automatically and sets the timestamps. This eliminates the need to set timestamps manually in every service method.
- The `id` field and its `@GeneratedValue` annotation are inherited. Every entity gets an auto-generated primary key without declaring it. This is the DRY principle in action at the database level.

### Scenario 3: Extending Spring's framework classes

Spring Boot is designed to be extended through inheritance. Many of its core features work by having you extend a base class and override specific methods.

```java
package com.company.orderservice.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

// By implementing WebMvcConfigurer (an interface, covered in the next note),
// you extend Spring's default web configuration. The @Configuration annotation
// tells Spring to use your customizations instead of the defaults.
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // Override the default CORS configuration.
        // The parent interface defines this method with an empty default implementation.
        // By overriding it, you customize the behavior.
        registry.addMapping("/api/**")
            .allowedOrigins("https://example.com", "https://admin.example.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .maxAge(3600);
    }
}
```

```java
package com.company.orderservice.security;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // HttpSecurity is a builder class provided by Spring Security.
        // You customize it by calling its methods, which is a form of
        // extending the framework's default security configuration.
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```

**What to notice:**

- Spring's architecture is built on inheritance and interfaces. You extend its base classes and override its methods to customize behavior. This is the **Template Method** design pattern: the framework defines the algorithm skeleton, and you fill in the specific steps.
- The `@Override` annotation on `addCorsMappings()` tells the compiler that you are intentionally replacing the default implementation. If Spring changes the method signature in a future version, the compiler will alert you immediately.

---

## Java vs Python Comparison

> [!note] Cross-language perspective
> Both Java and Python support inheritance, but Python supports multiple inheritance while Java does not.

| Aspect | Java | Python |
|--------|------|--------|
| Syntax | `class Dog extends Animal` | `class Dog(Animal):` |
| Multiple inheritance | No (classes), Yes (interfaces) | Yes (classes) |
| Constructor chaining | `super(name, age)` | `super().__init__(name, age)` |
| Method overriding | Same name, same params, `@Override` | Same name, same params (no annotation needed) |
| Root class | `java.lang.Object` (implicit) | `object` (implicit in Python 3) |
| Access control | `private`, `protected`, `public` | Convention: `_protected`, `__private` (name mangling) |
| Abstract classes | `abstract class` + `abstract` methods | `ABC` module + `@abstractmethod` |

```java
// Java: single inheritance
class Animal {
    String name;
    Animal(String name) { this.name = name; }
    void speak() { System.out.println("..."); }
}

class Dog extends Animal {
    Dog(String name) { super(name); }
    @Override
    void speak() { System.out.println("Woof!"); }
}
```

```python
# Python: supports multiple inheritance
class Animal:
    def __init__(self, name):
        self.name = name
    def speak(self):
        print("...")

class Pet:
    def __init__(self, owner):
        self.owner = owner
    def play(self):
        print("Playing!")

# Python allows this. Java does not (for classes).
class Dog(Animal, Pet):
    def __init__(self, name, owner):
        Animal.__init__(self, name)
        Pet.__init__(self, owner)
    def speak(self):
        print("Woof!")
```

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using inheritance for code reuse instead of "is-a" relationships

**Wrong:**

```java
public class User extends Address {
    // A User IS NOT an Address. A User HAS an Address.
    // This inheritance makes no logical sense, even though it
    // lets User reuse Address's fields (street, city, zip).
    String username;
    String email;
}
```

**Right:**

```java
public class User {
    String username;
    String email;
    Address address;  // Composition: a User HAS an Address

    // Or embed the address fields directly if the relationship is simple
    String street;
    String city;
    String zipCode;
}
```

**Why it is wrong:** Inheritance should model "is-a" relationships. A `Dog` is an `Animal`. An `Order` is a `BaseEntity`. A `PaymentException` is a `RuntimeException`. When you use inheritance purely to avoid typing duplicate fields, you create fragile class hierarchies that break when the parent class changes. Use **composition** (having a field of the other type) for "has-a" relationships.

### Mistake 2: Forgetting to call `super()` when the parent has no default constructor

**Wrong:**

```java
public class Animal {
    String name;
    Animal(String name) {
        this.name = name;
    }
    // No default constructor! The compiler stopped generating one
    // because you wrote a parameterized constructor.
}

public class Dog extends Animal {
    String breed;
    Dog(String name, String breed) {
        // COMPILATION ERROR! The compiler tries to insert super() automatically,
        // but Animal has no no-argument constructor.
        this.breed = breed;
    }
}
```

**Right:**

```java
public class Dog extends Animal {
    String breed;
    Dog(String name, String breed) {
        super(name);  // Explicitly call Animal's parameterized constructor
        this.breed = breed;
    }
}
```

**Why it is wrong:** When a parent class does not have a no-argument constructor, every subclass constructor must explicitly call one of the parent's parameterized constructors using `super(arguments)`. The compiler cannot guess which arguments to pass. This is one of the most common compilation errors when working with inheritance.

### Mistake 3: Overriding a method with a more restrictive access modifier

**Wrong:**

```java
public class Animal {
    public void eat() {
        System.out.println("Eating");
    }
}

public class Dog extends Animal {
    @Override
    private void eat() {  // COMPILATION ERROR! Cannot reduce visibility.
        System.out.println("Dog eating");
    }
}
```

**Right:**

```java
public class Dog extends Animal {
    @Override
    public void eat() {  // Same or wider access is allowed
        System.out.println("Dog eating");
    }
}
```

**Why it is wrong:** The Liskov Substitution Principle states that a subclass must be usable wherever the parent class is expected. If external code calls `animal.eat()` on a reference that happens to point to a `Dog`, the call must succeed. If `Dog` makes `eat()` private, the call would fail, violating the contract established by the parent class.

### Mistake 4: Confusing method overriding with method overloading

**Wrong thinking:** "I changed the parameter types, so I overrode the parent method."

**Right thinking:** "Overriding means same name AND same parameters in a subclass. Overloading means same name but different parameters in the same class. If I change the parameters, I am creating a new overloaded method, not overriding the parent's method."

```java
public class Animal {
    void eat(String food) {
        System.out.println("Eating " + food);
    }
}

public class Dog extends Animal {
    // This is OVERLOADING, not overriding. Different parameter type.
    void eat(int quantity) {
        System.out.println("Eating " + quantity + " bowls");
    }

    // This is OVERRIDING. Same name, same parameters.
    @Override
    void eat(String food) {
        System.out.println("Dog eating " + food);
    }
}
```

**Why it is wrong:** If you think you are overriding a method but you accidentally change the parameter types, the parent's method is still active and will be called in polymorphic contexts. The `@Override` annotation catches this mistake at compile time. Always use `@Override`.

---

## Key Takeaways

> [!tip] Remember these points
> 1. **Inheritance** lets a subclass acquire fields and methods from a superclass using the `extends` keyword. Java supports single, multilevel, and hierarchical inheritance but NOT multiple inheritance of classes.
> 2. The `super` keyword calls the parent constructor (`super(args)`), parent methods (`super.method()`), and parent fields (`super.field`). The `super()` call must be the first statement in the subclass constructor.
> 3. **Method overriding** replaces a parent method's implementation in the subclass. The method signature must match exactly. Always use the `@Override` annotation to catch mistakes at compile time. The access modifier cannot be more restrictive than the parent's.
> 4. Every Java class implicitly extends `java.lang.Object`. Override `toString()` for readable output, `equals()` for value comparison, and `hashCode()` for use in HashMaps and HashSets. Always override `equals()` and `hashCode()` together.
> 5. Use inheritance for true "is-a" relationships (Order is a BaseEntity, PaymentException is a RuntimeException). Use **composition** for "has-a" relationships (User has an Address). When in doubt, prefer composition over inheritance.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Shape Hierarchy (Easy)

Create a class hierarchy for geometric shapes:

1. `Shape` (superclass) with fields `color` (String) and a method `double area()` that returns 0 by default. Override `toString()` to return the color and area.
2. `Circle extends Shape` with field `radius`. Override `area()` to return `Math.PI * radius * radius`.
3. `Rectangle extends Shape` with fields `width` and `height`. Override `area()` to return `width * height`.
4. `Square extends Rectangle` (multilevel inheritance) with a constructor that takes a single `side` parameter and passes it as both width and height to `super()`.

Create an array of `Shape` objects containing one of each type. Loop through the array and print each shape's `toString()`.

> **Hint:** The `Square` constructor should call `super(side, side)` to reuse `Rectangle`'s area calculation.

### Exercise 2: Employee Hierarchy with Salary Calculation (Medium)

Create a multilevel inheritance hierarchy:

1. `Employee` (base) with fields `name` (String), `employeeId` (String), `baseSalary` (double). Constructor validates that `baseSalary > 0`. Method `double calculateSalary()` returns `baseSalary`.
2. `FullTimeEmployee extends Employee` with field `bonus` (double). Override `calculateSalary()` to return `baseSalary + bonus`.
3. `PartTimeEmployee extends Employee` with field `hoursWorked` (int) and `hourlyRate` (double). Override `calculateSalary()` to return `hoursWorked * hourlyRate`.
4. `Manager extends FullTimeEmployee` with field `teamSize` (int). Override `calculateSalary()` to return `super.calculateSalary() + (teamSize * 1000)` (management allowance).

In `main()`, create one of each type, store them in an `Employee[]` array, and print each employee's name and calculated salary.

> **Hint:** The `Manager` class extends `FullTimeEmployee`, which extends `Employee`. The constructor chain is `Manager()` -> `super()` -> `FullTimeEmployee()` -> `super()` -> `Employee()`.

### Exercise 3: Custom Exception Hierarchy (Medium)

Create an exception hierarchy for an e-commerce backend:

1. `ECommerceException extends RuntimeException` with fields `errorCode` (String) and `httpStatus` (int).
2. `OrderException extends ECommerceException` (for order-related errors).
3. `OrderNotFoundException extends OrderException` (404, "ORDER_NOT_FOUND").
4. `OrderAlreadyShippedException extends OrderException` (409, "ORDER_ALREADY_SHIPPED").
5. `PaymentException extends ECommerceException` (for payment-related errors).
6. `InsufficientFundsException extends PaymentException` (402, "INSUFFICIENT_FUNDS").

Write a method `processOrder(String orderId, double amount)` that throws the appropriate exceptions based on simulated conditions. Write a `main()` method with a try-catch block that catches each exception type and prints the error code, HTTP status, and message.

> **Hint:** Each specific exception's constructor should call `super(message, httpStatus, errorCode)` to pass the details up the chain.

### Exercise 4: BaseEntity with Audit Logging (Hard, Optional)

Create a `BaseEntity` class that tracks creation and modification metadata:

- Fields: `id` (long, auto-incremented via static counter), `createdBy` (String), `createdAt` (LocalDateTime), `updatedBy` (String), `updatedAt` (LocalDateTime), `version` (int, starts at 1).
- Constructor takes `createdBy` and sets all timestamps.
- Method `update(String updatedBy)` increments `version` and updates `updatedBy` and `updatedAt`.
- Method `toString()` returns all audit fields.

Create two subclasses: `Product extends BaseEntity` and `Category extends BaseEntity`. Each adds its own fields. Override `toString()` in each subclass to call `super.toString()` and append the subclass-specific fields.

In `main()`, create products and categories, update them multiple times, and print the audit trail.

> **Hint:** Use `super.toString()` in the subclass `toString()` to include the base audit information, then append the subclass fields. This avoids duplicating the audit formatting logic.

<details>
<summary>Solution for Exercise 1</summary>

```java
class Shape {
    String color;

    Shape(String color) {
        this.color = color;
    }

    double area() {
        return 0;
    }

    @Override
    public String toString() {
        return getClass().getSimpleName() + "{color='" + color + "', area=" + String.format("%.2f", area()) + "}";
    }
}

class Circle extends Shape {
    double radius;

    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

class Rectangle extends Shape {
    double width;
    double height;

    Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }

    @Override
    double area() {
        return width * height;
    }
}

class Square extends Rectangle {
    Square(String color, double side) {
        super(color, side, side);
    }
}

public class Main {
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle("Red", 5),
            new Rectangle("Blue", 4, 6),
            new Square("Green", 3)
        };

        for (Shape s : shapes) {
            System.out.println(s);
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
class ECommerceException extends RuntimeException {
    private final String errorCode;
    private final int httpStatus;

    ECommerceException(String message, int httpStatus, String errorCode) {
        super(message);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public String getErrorCode() { return errorCode; }
    public int getHttpStatus() { return httpStatus; }
}

class OrderException extends ECommerceException {
    OrderException(String message, int httpStatus, String errorCode) {
        super(message, httpStatus, errorCode);
    }
}

class OrderNotFoundException extends OrderException {
    OrderNotFoundException(String orderId) {
        super("Order not found: " + orderId, 404, "ORDER_NOT_FOUND");
    }
}

class OrderAlreadyShippedException extends OrderException {
    OrderAlreadyShippedException(String orderId) {
        super("Order already shipped: " + orderId, 409, "ORDER_ALREADY_SHIPPED");
    }
}

class PaymentException extends ECommerceException {
    PaymentException(String message, int httpStatus, String errorCode) {
        super(message, httpStatus, errorCode);
    }
}

class InsufficientFundsException extends PaymentException {
    InsufficientFundsException(double required, double available) {
        super("Required: " + required + ", Available: " + available, 402, "INSUFFICIENT_FUNDS");
    }
}

public class Main {
    static void processOrder(String orderId, double amount) {
        if ("999".equals(orderId)) throw new OrderNotFoundException(orderId);
        if ("100".equals(orderId)) throw new OrderAlreadyShippedException(orderId);
        if (amount > 10000) throw new InsufficientFundsException(amount, 5000);
        System.out.println("Order " + orderId + " processed successfully for " + amount);
    }

    public static void main(String[] args) {
        String[] testOrders = {"001", "999", "100", "002"};
        double[] testAmounts = {500, 200, 300, 15000};

        for (int i = 0; i < testOrders.length; i++) {
            try {
                processOrder(testOrders[i], testAmounts[i]);
            } catch (ECommerceException e) {
                System.out.printf("Error [%d %s]: %s%n",
                    e.getHttpStatus(), e.getErrorCode(), e.getMessage());
            }
        }
    }
}
```

</details>

---

## Related Notes

- [[Java - Encapsulation - Getters Setters Access Modifiers]]
- [[Java - Polymorphism - Compile Time and Runtime]] (next note)
- [[Java - Abstraction - Abstract Classes and Interfaces]]

---

## Resources

- [Oracle Java Tutorials: Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html) - Official documentation covering extends, super, and method overriding.
- [Oracle Java Tutorials: Overriding and Hiding Methods](https://docs.oracle.com/javase/tutorial/java/IandI/override.html) - Detailed explanation of overriding rules and the difference between overriding and hiding.
- [Baeldung: Java Inheritance](https://www.baeldung.com/java-inheritance) - Comprehensive guide with examples of all inheritance types.
- [Baeldung: Java super Keyword](https://www.baeldung.com/java-super-keyword) - Detailed explanation of all three uses of super.
- [Effective Java by Joshua Bloch - Item 18: Favor Composition Over Inheritance](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive argument for when NOT to use inheritance. Essential reading for backend engineers.
