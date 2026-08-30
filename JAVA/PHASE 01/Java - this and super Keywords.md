---
title: "Java - this and super Keywords"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - this
  - super
  - keywords
status: "not-started"
---

# Java - this and super Keywords

> [!abstract] Overview
> The `this` and `super` keywords are reference variables provided by Java to navigate the object hierarchy. `this` refers to the current object instance and is used to resolve naming conflicts, chain constructors, pass the current object to other methods, and return the current object from methods. `super` refers to the immediate parent class and is used to call parent constructors, invoke parent methods, and access parent fields that are hidden by subclass fields. Both keywords are fundamental to writing correct OOP code and appear extensively in Spring Boot entities, services, and framework extension points.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Constructors - Default Parameterized Copy]]
> - [[Java - Inheritance - Single Multilevel Hierarchical]]
> - [[Java - Encapsulation - Getters Setters Access Modifiers]]

---

## Theory

### The `this` Keyword

The `this` keyword is a reference variable that points to the **current object** -- the object on which the method or constructor was invoked. It exists only in instance contexts (instance methods, instance initializers, and constructors). It does not exist in static methods because static methods are not associated with any object.

**Memory perspective:** When you call `order.confirm()`, the JVM passes the reference to the `order` object as an implicit first argument to the `confirm()` method. Inside the method, `this` holds that reference. At the bytecode level, `this` is stored in local variable slot 0 of every instance method.

```text
order.confirm()

JVM internally:
  confirm(Order this)   <-- 'this' is slot 0, pointing to the order object
    this.status = CONFIRMED
```

### Uses of `this`

**1. Resolving naming conflicts (field shadowing):**

When a method parameter or local variable has the same name as an instance field, the local name "shadows" the field. `this` disambiguates.

```java
public class User {
    private String name;
    private String email;

    public void setName(String name) {
        // 'name' refers to the parameter (local variable)
        // 'this.name' refers to the instance field
        this.name = name;
    }
}
```

Without `this`, the assignment `name = name` assigns the parameter to itself, leaving the instance field unchanged. This is the most common use of `this` and the reason it appears in virtually every Java class.

**2. Constructor chaining:**

`this(arguments)` calls another constructor in the same class. It must be the first statement in the constructor.

```java
import java.math.BigDecimal;

public class Order {
    private String orderNumber;
    private BigDecimal total;
    private OrderStatus status;

    // Master constructor
    public Order(String orderNumber, BigDecimal total, OrderStatus status) {
        this.orderNumber = orderNumber;
        this.total = total;
        this.status = status;
    }

    // Chains to the master constructor with a default status
    public Order(String orderNumber, BigDecimal total) {
        this(orderNumber, total, OrderStatus.PENDING);  // Must be first statement
    }

    // Chains to the two-argument constructor
    public Order(String orderNumber) {
        this(orderNumber, BigDecimal.ZERO);  // Chains to the constructor above
    }
}
```

**3. Passing the current object to other methods:**

`this` can be passed as an argument to methods that need a reference to the current object.

```java
public class Order {
    public void process() {
        // Pass this order to the payment service
        paymentService.charge(this);

        // Pass this order to the event publisher
        eventPublisher.publish(new OrderProcessedEvent(this));
    }
}
```

**4. Returning the current object (method chaining / fluent API):**

Returning `this` from a method allows callers to chain multiple method calls on the same object. This is the foundation of the Builder pattern and fluent APIs.

```java
public class QueryBuilder {
    private StringBuilder query = new StringBuilder();

    public QueryBuilder select(String columns) {
        query.append("SELECT ").append(columns);
        return this;  // Return the current object for chaining
    }

    public QueryBuilder from(String table) {
        query.append(" FROM ").append(table);
        return this;
    }

    public QueryBuilder where(String condition) {
        query.append(" WHERE ").append(condition);
        return this;
    }

    public String build() {
        return query.toString();
    }
}

// Usage: fluent method chaining
String sql = new QueryBuilder()
    .select("id, name, email")
    .from("users")
    .where("active = true")
    .build();
// "SELECT id, name, email FROM users WHERE active = true"
```

**5. Distinguishing between the current object and other objects:**

`this` is useful in `equals()` methods to check if the argument is the same object.

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;  // Same reference? Equal immediately.
    if (obj == null || getClass() != obj.getClass()) return false;
    Order other = (Order) obj;
    return this.orderNumber.equals(other.orderNumber);
}
```

### The `super` Keyword

The `super` keyword is a reference variable that points to the **immediate parent class** of the current object. It provides access to the parent's constructors, methods, and fields from within the subclass.

### Uses of `super`

**1. Calling the parent constructor:**

`super(arguments)` calls a constructor of the immediate parent class. It must be the first statement in the subclass constructor. If you do not write `super()` explicitly, the compiler inserts `super()` (no arguments) automatically.

```java
import java.time.LocalDateTime;

public class BaseEntity {
    protected Long id;
    protected LocalDateTime createdAt;

    public BaseEntity() {
        this.createdAt = LocalDateTime.now();
    }

    public BaseEntity(Long id) {
        this.id = id;
        this.createdAt = LocalDateTime.now();
    }
}

public class Order extends BaseEntity {
    private String orderNumber;

    public Order(Long id, String orderNumber) {
        super(id);  // Calls BaseEntity(Long id). Must be first!
        this.orderNumber = orderNumber;
    }
}
```

**2. Calling a parent method:**

`super.methodName()` calls the parent's version of a method, even if the subclass has overridden it. This is essential when you want to extend the parent's behavior rather than replace it entirely.

```java
public class Notification {
    public void send(String recipient, String message) {
        System.out.println("Logging: Sending to " + recipient);
        // Base sending logic...
    }
}

public class EmailNotification extends Notification {
    @Override
    public void send(String recipient, String message) {
        super.send(recipient, message);  // Call parent's logging and base logic first
        System.out.println("Sending email via SMTP to " + recipient);
        // Email-specific logic...
    }
}
```

**3. Accessing a hidden parent field:**

When a subclass declares a field with the same name as a parent field, the subclass field "hides" the parent field. `super.fieldName` accesses the parent's version.

```java
public class Animal {
    protected String name = "Animal";
}

public class Dog extends Animal {
    protected String name = "Dog";  // Hides Animal's 'name'

    public void printNames() {
        System.out.println("this.name: " + this.name);    // "Dog"
        System.out.println("super.name: " + super.name);  // "Animal"
    }
}
```

> [!warning] Field hiding is almost always a design mistake
> In practice, you should avoid declaring fields in subclasses that have the same name as parent fields. Field hiding creates confusion and bugs. If you need to change a parent field's value, use a setter method instead of redeclaring the field.

### `this` vs `super` Comparison

| Aspect | `this` | `super` |
|--------|--------|---------|
| Refers to | Current object | Immediate parent class |
| Constructor call | `this(args)` calls another constructor in the same class | `super(args)` calls a constructor in the parent class |
| Method call | `this.method()` calls the current class's method (or overridden version) | `super.method()` calls the parent's version, bypassing overrides |
| Field access | `this.field` accesses the current class's field | `super.field` accesses the parent's field (if hidden) |
| Available in | Instance methods, constructors, instance initializers | Instance methods, constructors, instance initializers |
| Available in static context | No | No |
| Must be first in constructor | `this()` must be first if used | `super()` must be first if used |
| Can use both in same constructor | No (both must be first, only one first statement allowed) | No |

### How `this` and `super` Work Internally

At the bytecode level, `this` is not a keyword but a convention. The JVM reserves local variable slot 0 in every instance method for the object reference. When you write `this.name`, the compiler generates:

```text
aload_0          // Load 'this' (slot 0) onto the stack
getfield User.name  // Access the 'name' field of the object on the stack
```

For `super.method()`, the compiler generates an `invokespecial` instruction instead of `invokevirtual`. The `invokevirtual` instruction uses dynamic dispatch (runtime polymorphism), while `invokespecial` bypasses the vtable and calls the specific method in the specified class directly.

```text
// this.method() compiles to:
aload_0
invokevirtual Order.method()  // Dynamic dispatch: JVM checks the vtable

// super.method() compiles to:
aload_0
invokespecial BaseEntity.method()  // Direct call: bypasses the vtable
```

This is why `super.method()` always calls the parent's version, even if the method is overridden in the current class. The `invokespecial` instruction tells the JVM to skip the vtable lookup and call the method in the specified class directly.

**Constructor chaining at the bytecode level:**

When you write `super(id)` in a constructor, the compiler generates:

```text
aload_0          // Load 'this'
iload_1          // Load the 'id' parameter
invokespecial BaseEntity.<init>(Long)  // Call parent constructor
```

The parent constructor runs first, initializing the parent's fields. Then the subclass constructor continues, initializing the subclass's fields. This top-down initialization order ensures that the parent's state is fully established before the child's constructor accesses it.

> [!tip] Key Insight
> `this` and `super` are not objects. They are reference variables that the JVM provides implicitly. You cannot assign to them (`this = new Order()` is a compilation error). You cannot use them in static contexts (there is no object for `this` to refer to, and no parent instance for `super` to refer to). They exist solely to navigate the object hierarchy during instance method execution and construction.

---

## Syntax and Basic Examples

### Example 1: All uses of `this` in a single class

```java
public class Product {
    private long id;
    private String name;
    private double price;
    private int stock;

    // Use 1: Constructor chaining with this()
    public Product(long id, String name, double price, int stock) {
        this.id = id;            // Use 2: Resolving field shadowing
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

    public Product(String name, double price) {
        this(0, name, price, 0);  // Chains to the 4-argument constructor
    }

    public Product(String name) {
        this(name, 0.0);  // Chains to the 2-argument constructor
    }

    // Use 3: Returning 'this' for method chaining (fluent API)
    public Product setPrice(double price) {
        if (price < 0) throw new IllegalArgumentException("Price cannot be negative");
        this.price = price;
        return this;  // Enables chaining: product.setPrice(100).setStock(50)
    }

    public Product setStock(int stock) {
        if (stock < 0) throw new IllegalArgumentException("Stock cannot be negative");
        this.stock = stock;
        return this;
    }

    // Use 4: Passing 'this' to another object
    public void registerInCatalog(ProductCatalog catalog) {
        catalog.addProduct(this);  // Pass the current Product object
    }

    // Use 5: Using 'this' in equals()
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;  // Same reference check
        if (obj == null || getClass() != obj.getClass()) return false;
        Product other = (Product) obj;
        return this.id == other.id;
    }

    @Override
    public int hashCode() {
        return Long.hashCode(id);
    }

    @Override
    public String toString() {
        return "Product{id=" + id + ", name='" + name + "', price=" + price + "}";
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        // Constructor chaining
        Product p1 = new Product("Laptop");  // Chains: 1-arg -> 2-arg -> 4-arg
        System.out.println(p1);

        // Method chaining (fluent API)
        Product p2 = new Product(1, "Mouse", 0, 0)
            .setPrice(1500)
            .setStock(50);
        System.out.println(p2);
    }
}
```

**Output:**

```text
Product{id=0, name='Laptop', price=0.0}
Product{id=1, name='Mouse', price=1500.0}
```

### Example 2: All uses of `super` in an inheritance hierarchy

```java
import java.time.LocalDateTime;

public class BaseEntity {
    protected Long id;
    protected LocalDateTime createdAt;
    protected LocalDateTime updatedAt;

    public BaseEntity() {
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
        System.out.println("BaseEntity constructor called");
    }

    public BaseEntity(Long id) {
        this();  // Chains to the no-arg constructor
        this.id = id;
    }

    public void audit(String action) {
        this.updatedAt = LocalDateTime.now();
        System.out.println("[AUDIT] Entity " + id + ": " + action + " at " + updatedAt);
    }

    public String getEntityInfo() {
        return "BaseEntity{id=" + id + ", created=" + createdAt + "}";
    }
}
```

```java
import java.math.BigDecimal;

public class Order extends BaseEntity {
    private String orderNumber;
    private BigDecimal totalAmount;
    private String status;

    // Use 1: Calling parent constructor with super()
    public Order(Long id, String orderNumber, BigDecimal totalAmount) {
        super(id);  // Calls BaseEntity(Long id). Must be first statement!
        this.orderNumber = orderNumber;
        this.totalAmount = totalAmount;
        this.status = "PENDING";
        System.out.println("Order constructor called");
    }

    // Use 2: Calling parent method with super.method()
    @Override
    public void audit(String action) {
        super.audit(action);  // Call BaseEntity's audit() first for base logging
        System.out.println("[ORDER AUDIT] Order " + orderNumber + ": " + action);
        // Additional order-specific audit logic
    }

    // Use 3: Extending parent method behavior
    @Override
    public String getEntityInfo() {
        // Call parent's version and append order-specific info
        return super.getEntityInfo() + " | Order{number=" + orderNumber
            + ", total=" + totalAmount + ", status=" + status + "}";
    }

    public void confirm() {
        this.status = "CONFIRMED";
        this.audit("Order confirmed");  // Calls Order's overridden audit()
    }
}
```

```java
import java.math.BigDecimal;

public class InternationalOrder extends Order {
    private String destinationCountry;
    private BigDecimal customsDuty;

    // Multilevel super() chaining:
    // InternationalOrder() -> super() -> Order() -> super() -> BaseEntity()
    public InternationalOrder(Long id, String orderNumber, BigDecimal totalAmount,
                               String destinationCountry, BigDecimal customsDuty) {
        super(id, orderNumber, totalAmount);  // Calls Order's constructor
        this.destinationCountry = destinationCountry;
        this.customsDuty = customsDuty;
        System.out.println("InternationalOrder constructor called");
    }

    @Override
    public void audit(String action) {
        super.audit(action);  // Calls Order's audit(), which calls BaseEntity's audit()
        System.out.println("[INTL AUDIT] Destination: " + destinationCountry);
    }

    @Override
    public String getEntityInfo() {
        return super.getEntityInfo() + " | Intl{country=" + destinationCountry
            + ", duty=" + customsDuty + "}";
    }
}
```

```java
import java.math.BigDecimal;

public class Main {
    public static void main(String[] args) {
        System.out.println("--- Creating InternationalOrder ---");
        InternationalOrder order = new InternationalOrder(
            1L, "INT-001", new BigDecimal("5000"), "Bangladesh", new BigDecimal("750")
        );
        // Constructor output:
        // BaseEntity constructor called
        // Order constructor called
        // InternationalOrder constructor called

        System.out.println("\n--- Entity Info ---");
        System.out.println(order.getEntityInfo());

        System.out.println("\n--- Confirming Order ---");
        order.confirm();
        // Audit output:
        // [AUDIT] Entity 1: Order confirmed at 2025-07-10T14:30:00
        // [ORDER AUDIT] Order INT-001: Order confirmed
        // [INTL AUDIT] Destination: Bangladesh
    }
}
```

### Example 3: `this` and `super` with field hiding (and why to avoid it)

```java
public class Parent {
    protected String type = "PARENT";
    protected int version = 1;

    public void display() {
        System.out.println("Parent display: type=" + type + ", version=" + version);
    }
}

public class Child extends Parent {
    protected String type = "CHILD";  // HIDES Parent's 'type' field (bad practice)
    // Note: 'version' is NOT redeclared, so it is inherited normally.

    @Override
    public void display() {
        // 'this.type' refers to Child's 'type' (the hiding field)
        // 'super.type' refers to Parent's 'type' (the hidden field)
        System.out.println("this.type: " + this.type);    // CHILD
        System.out.println("super.type: " + super.type);  // PARENT

        // 'this.version' and 'super.version' are the same because
        // Child did not redeclare 'version'.
        System.out.println("this.version: " + this.version);    // 1
        System.out.println("super.version: " + super.version);  // 1
    }

    public void demonstrateFieldHiding() {
        this.type = "MODIFIED CHILD";
        super.type = "MODIFIED PARENT";

        System.out.println("After modification:");
        System.out.println("this.type: " + this.type);    // MODIFIED CHILD
        System.out.println("super.type: " + super.type);  // MODIFIED PARENT
        // Both fields exist independently in memory!
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Child child = new Child();
        child.display();
        child.demonstrateFieldHiding();

        // The hidden field is still accessible through a parent reference
        Parent parentRef = child;
        System.out.println("Via parent ref: " + parentRef.type);  // MODIFIED PARENT
        // Field access is resolved at compile time based on the reference type,
        // NOT the object type. This is different from method overriding.
    }
}
```

### Example 4: Builder pattern combining `this` and constructor chaining

```java
import java.util.ArrayList;
import java.util.List;

public class EmailMessage {
    private final String from;
    private final String to;
    private final String subject;
    private final String body;
    private final boolean isHtml;
    private final List<String> ccRecipients;

    // Private constructor: only the Builder can create EmailMessage objects
    private EmailMessage(Builder builder) {
        this.from = builder.from;
        this.to = builder.to;
        this.subject = builder.subject;
        this.body = builder.body;
        this.isHtml = builder.isHtml;
        this.ccRecipients = List.copyOf(builder.ccRecipients);
    }

    // Getters
    public String getFrom() { return from; }
    public String getTo() { return to; }
    public String getSubject() { return subject; }
    public String getBody() { return body; }
    public boolean isHtml() { return isHtml; }
    public List<String> getCcRecipients() { return ccRecipients; }

    // Static nested Builder class
    public static class Builder {
        private String from;
        private String to;
        private String subject;
        private String body = "";
        private boolean isHtml = false;
        private List<String> ccRecipients = new ArrayList<>();

        public Builder(String from, String to) {
            this.from = from;  // 'this' resolves shadowing
            this.to = to;
        }

        // Each setter returns 'this' (the Builder) for method chaining
        public Builder subject(String subject) {
            this.subject = subject;
            return this;
        }

        public Builder body(String body) {
            this.body = body;
            return this;
        }

        public Builder html(boolean isHtml) {
            this.isHtml = isHtml;
            return this;
        }

        public Builder cc(String... recipients) {
            this.ccRecipients.addAll(List.of(recipients));
            return this;
        }

        public EmailMessage build() {
            if (from == null || to == null || subject == null) {
                throw new IllegalStateException("from, to, and subject are required");
            }
            return new EmailMessage(this);  // Pass 'this' Builder to the constructor
        }
    }

    @Override
    public String toString() {
        return "Email{from=" + from + ", to=" + to + ", subject='" + subject + "'}";
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        EmailMessage email = new EmailMessage.Builder("noreply@example.com", "saad@example.com")
            .subject("Order Confirmation")
            .body("<h1>Your order is confirmed!</h1>")
            .html(true)
            .cc("admin@example.com", "archive@example.com")
            .build();

        System.out.println(email);
        System.out.println("CC: " + email.getCcRecipients());
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Both `this` and `super` are used extensively in Spring Boot applications. Here are three realistic scenarios.

### Scenario 1: JPA Entity with `super` constructor chaining

In a JPA entity hierarchy, subclass constructors chain to the parent constructor using `super()`, and business methods use `this` to reference the current entity.

```java
package com.company.orderservice.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@MappedSuperclass
public abstract class AuditableEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(nullable = false)
    private LocalDateTime updatedAt;

    @Column(nullable = false)
    private String createdBy;

    protected AuditableEntity() {
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }

    protected AuditableEntity(String createdBy) {
        this();  // Chains to the no-arg constructor using this()
        this.createdBy = createdBy;
    }

    @PreUpdate
    protected void onUpdate() {
        this.updatedAt = LocalDateTime.now();  // 'this' refers to the current entity
    }

    public Long getId() { return id; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public String getCreatedBy() { return createdBy; }
}
```

```java
import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity
@Table(name = "orders")
public class Order extends AuditableEntity {
    @Column(nullable = false, unique = true)
    private String orderNumber;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal totalAmount;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    protected Order() {
        super();  // Calls AuditableEntity()
    }

    public Order(String orderNumber, BigDecimal totalAmount, String createdBy) {
        super(createdBy);  // Calls AuditableEntity(String createdBy)
        this.orderNumber = orderNumber;
        this.totalAmount = totalAmount;
        this.status = OrderStatus.PENDING;
    }

    public void confirm() {
        if (this.status != OrderStatus.PENDING) {
            throw new IllegalStateException(
                "Cannot confirm order in status: " + this.status
            );
        }
        this.status = OrderStatus.CONFIRMED;
        this.onUpdate();  // Calls inherited method using 'this'
    }

    // 'this' is passed to the DTO factory method
    public OrderResponseDTO toDTO() {
        return OrderResponseDTO.fromEntity(this);
    }

    public String getOrderNumber() { return orderNumber; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public OrderStatus getStatus() { return status; }
}
```

**What to notice:**

- `super(createdBy)` chains the constructor to the parent, ensuring that `createdAt`, `updatedAt`, and `createdBy` are initialized before the `Order`-specific fields.
- `this.status`, `this.onUpdate()`, and `this.toDTO()` all use `this` to reference the current entity instance. In JPA, the entity object represents a row in the database, and `this` refers to that specific row's Java representation.
- `OrderResponseDTO.fromEntity(this)` passes the current entity to a static factory method that converts it into a DTO. This is a common pattern in Spring Boot controllers.

### Scenario 2: Extending Spring's exception classes with `super`

Custom exception classes use `super()` to chain to the parent exception's constructor, passing the message and cause up the hierarchy.

```java
package com.company.orderservice.exception;

public class AppException extends RuntimeException {
    private final int httpStatus;
    private final String errorCode;

    public AppException(String message, int httpStatus, String errorCode) {
        super(message);  // Calls RuntimeException(String message)
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public AppException(String message, int httpStatus, String errorCode, Throwable cause) {
        super(message, cause);  // Calls RuntimeException(String, Throwable)
        // 'super(message, cause)' passes both the message and the original exception
        // up to the parent. This preserves the full stack trace chain.
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public int getHttpStatus() { return httpStatus; }
    public String getErrorCode() { return errorCode; }
}
```

```java
public class OrderNotFoundException extends AppException {
    public OrderNotFoundException(Long orderId) {
        // Chains to AppException(String, int, String)
        super("Order not found: " + orderId, 404, "ORDER_NOT_FOUND");
    }
}
```

```java
public class PaymentProcessingException extends AppException {
    public PaymentProcessingException(String message, Throwable cause) {
        // Chains to AppException(String, int, String, Throwable)
        // The 'cause' parameter preserves the original exception (e.g., a Stripe API error)
        super("Payment processing failed: " + message, 502, "PAYMENT_ERROR", cause);
    }
}
```

```java
// Usage in a service:
@Service
public class PaymentService {
    public void charge(Order order) {
        try {
            stripeClient.charge(order.getTotalAmount(), order.getUserId());
        } catch (StripeApiException e) {
            // Wrap the Stripe exception in our custom exception.
            // 'e' is passed as the cause, preserving the original stack trace.
            throw new PaymentProcessingException(e.getMessage(), e);
        }
    }
}
```

**What to notice:**

- `super(message)` and `super(message, cause)` chain to `RuntimeException`'s constructors. The `cause` parameter enables **exception chaining**, which preserves the full error history. When you print the stack trace of a `PaymentProcessingException`, you will see both the custom exception and the original `StripeApiException` that caused it.
- The constructor chain is: `OrderNotFoundException()` -> `super()` -> `AppException()` -> `super()` -> `RuntimeException()` -> `super()` -> `Exception()` -> `super()` -> `Throwable()`. Each level initializes its own fields and passes the message up.

### Scenario 3: Fluent API with `this` return in Spring's RestTemplate

Spring's `ResponseEntity` and many other Spring classes use the `this` return pattern for fluent method chaining.

```java
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

// Simplified version of how Spring's ResponseEntity.Builder works internally
public class ApiResponse<T> {
    private int statusCode;
    private Map<String, String> headers;
    private T body;

    private ApiResponse() {
        this.headers = new HashMap<>();
    }

    // Each method returns 'this' for chaining
    public ApiResponse<T> status(int statusCode) {
        this.statusCode = statusCode;
        return this;
    }

    public ApiResponse<T> header(String name, String value) {
        this.headers.put(name, value);
        return this;
    }

    public ApiResponse<T> body(T body) {
        this.body = body;
        return this;
    }

    public ResponseEntity<T> build() {
        HttpHeaders httpHeaders = new HttpHeaders();
        this.headers.forEach(httpHeaders::add);
        return new ResponseEntity<>(this.body, httpHeaders, HttpStatus.valueOf(this.statusCode));
    }

    // Static factory methods that return a pre-configured builder
    public static <T> ApiResponse<T> ok() {
        return new ApiResponse<T>().status(200);
    }

    public static <T> ApiResponse<T> created() {
        return new ApiResponse<T>().status(201);
    }

    public static <T> ApiResponse<T> notFound() {
        return new ApiResponse<T>().status(404);
    }
}
```

```java
// Usage in a controller:
@RestController
public class OrderController {

    @PostMapping("/orders")
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        Order order = orderService.createOrder(request);

        return ApiResponse.<OrderResponse>created()
            .header("Location", "/orders/" + order.getId())
            .header("X-Request-Id", UUID.randomUUID().toString())
            .body(OrderResponse.fromEntity(order))
            .build();
    }
}
```

**What to notice:**

- Every setter method returns `this`, enabling the fluent chaining pattern: `ApiResponse.created().header(...).body(...).build()`.
- The `this` return type is `ApiResponse<T>`, which preserves the generic type through the chain. This is important for type safety in generic fluent APIs.
- This pattern is used extensively in the Java ecosystem: `StringBuilder.append()`, `Stream.filter()`, Spring's `WebClient`, OkHttp's `Request.Builder`, and JPA's `CriteriaBuilder`.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using `this` in a static method

**Wrong:**

```java
public class OrderService {
    private OrderRepository repository;

    public static List<Order> getAllOrders() {
        return this.repository.findAll();  // COMPILATION ERROR!
        // 'this' does not exist in static methods.
        // Static methods belong to the class, not to any object.
    }
}
```

**Right:**

```java
public class OrderService {
    private OrderRepository repository;

    // Option 1: Make it an instance method (preferred in Spring)
    public List<Order> getAllOrders() {
        return this.repository.findAll();  // OK: 'this' exists in instance methods
    }

    // Option 2: Pass the dependency as a parameter
    public static List<Order> getAllOrders(OrderRepository repository) {
        return repository.findAll();  // No 'this' needed
    }
}
```

**Why it is wrong:** `this` refers to the current object instance. Static methods are not associated with any object. They belong to the class itself. There is no `this` to refer to. The compiler will reject any use of `this` in a static context.

### Mistake 2: Putting code before `super()` or `this()` in a constructor

**Wrong:**

```java
public class Order extends BaseEntity {
    private String orderNumber;

    public Order(String orderNumber) {
        System.out.println("Creating order");  // Code before super()
        super();  // COMPILATION ERROR! super() must be the first statement.
        this.orderNumber = orderNumber;
    }
}
```

**Right:**

```java
public class Order extends BaseEntity {
    private String orderNumber;

    public Order(String orderNumber) {
        super();  // Must be first
        System.out.println("Creating order");  // Code after super() is fine
        this.orderNumber = orderNumber;
    }
}
```

**Why it is wrong:** Java requires that `super()` or `this()` be the very first statement in a constructor. This ensures that the parent class (or the chained constructor) fully initializes the object before any subclass code runs. If you need to compute a value before calling `super()`, extract the computation into a static helper method:

```java
public Order(String rawInput) {
    super(parseId(rawInput));  // Static method call is allowed before super()
    this.orderNumber = rawInput;
}

private static Long parseId(String rawInput) {
    return Long.parseLong(rawInput.substring(0, 8));
}
```

### Mistake 3: Confusing `this.method()` with `super.method()` in overridden methods

**Wrong:**

```java
public class EmailNotification extends Notification {
    @Override
    public void send(String recipient, String message) {
        this.send(recipient, message);  // INFINITE RECURSION!
        // 'this.send()' calls the current class's send(), which is THIS method.
        // The method calls itself forever until StackOverflowError.
    }
}
```

**Right:**

```java
public class EmailNotification extends Notification {
    @Override
    public void send(String recipient, String message) {
        super.send(recipient, message);  // Calls the PARENT's send() method
        // Email-specific logic here
    }
}
```

**Why it is wrong:** `this.send()` resolves to the current class's overridden method, creating infinite recursion. `super.send()` bypasses the override and calls the parent's version directly. When you want to extend the parent's behavior, always use `super.method()`. When you want to call the current class's version (including overrides), use `this.method()` or just `method()`.

### Mistake 4: Using `super` to access private parent members

**Wrong:**

```java
public class BaseEntity {
    private Long id;  // Private!
}

public class Order extends BaseEntity {
    public void printId() {
        System.out.println(super.id);  // COMPILATION ERROR!
        // 'id' is private in BaseEntity. 'super' does not bypass access control.
    }
}
```

**Right:**

```java
public class BaseEntity {
    private Long id;
    public Long getId() { return id; }  // Provide a public or protected getter
}

public class Order extends BaseEntity {
    public void printId() {
        System.out.println(this.getId());  // Access through the inherited getter
        // Or: System.out.println(super.getId());  // Also works
    }
}
```

**Why it is wrong:** `super` provides access to the parent's members, but it does not bypass access modifiers. Private members are still inaccessible from the subclass, even through `super`. Use `protected` or `public` access modifiers for members that subclasses need to access, or provide getter methods.

---

## Key Takeaways

> [!tip] Remember these points
> 1. `this` refers to the **current object instance**. Use it to resolve field shadowing (`this.name = name`), chain constructors (`this(args)`), pass the current object to other methods, return the current object for fluent APIs, and check reference equality in `equals()`.
> 2. `super` refers to the **immediate parent class**. Use it to call parent constructors (`super(args)`), invoke parent methods that have been overridden (`super.method()`), and access parent fields that are hidden by subclass fields (`super.field`).
> 3. Both `this()` and `super()` must be the **first statement** in a constructor. You cannot use both in the same constructor. If you write neither, the compiler inserts `super()` automatically.
> 4. Neither `this` nor `super` is available in **static contexts**. Static methods belong to the class, not to any object, so there is no current instance for `this` to refer to and no parent instance for `super` to refer to.
> 5. At the bytecode level, `this` is local variable slot 0 in every instance method. `super.method()` compiles to `invokespecial` (direct call to the parent), while `this.method()` compiles to `invokevirtual` (dynamic dispatch through the vtable). This is why `super.method()` always calls the parent's version, even in polymorphic contexts.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Constructor Chaining with `this` and `super` (Easy)

Create a three-level inheritance hierarchy:

1. `Vehicle` with fields `make` (String), `year` (int). Two constructors: no-arg (defaults) and parameterized.
2. `Car extends Vehicle` with field `numDoors` (int). Two constructors: one that chains to `Vehicle(String, int)` using `super()`, and one that chains to the other `Car` constructor using `this()`.
3. `ElectricCar extends Car` with field `batteryCapacityKwh` (double). One constructor that chains to `Car` using `super()`.

In each constructor, print a message like "Vehicle constructor called" so you can see the initialization order. In `main()`, create an `ElectricCar` and observe the constructor chain output.

> **Hint:** The output should show constructors executing from top to bottom: Vehicle -> Car -> ElectricCar.

### Exercise 2: Fluent Query Builder with `this` (Medium)

Create a `SqlQueryBuilder` class that uses `this` return for fluent method chaining. Support:

- `select(String... columns)` - adds SELECT clause
- `from(String table)` - adds FROM clause
- `where(String condition)` - adds WHERE clause (can be called multiple times, joining with AND)
- `orderBy(String column, boolean ascending)` - adds ORDER BY clause
- `limit(int count)` - adds LIMIT clause
- `build()` - returns the final SQL string

All methods except `build()` should return `this`. In `main()`, build a complex query using chaining and print the result.

> **Hint:** Use a `List<String>` internally to collect WHERE conditions. In `build()`, join them with " AND ".

### Exercise 3: Overriding with `super` Extension (Medium)

Create a notification hierarchy:

1. `Notification` with method `send(String recipient, String message)` that prints a generic log message.
2. `EmailNotification extends Notification` that overrides `send()`, calls `super.send()` first, then adds email-specific logic (SMTP connection, subject line).
3. `PriorityEmailNotification extends EmailNotification` that overrides `send()`, calls `super.send()` first, then adds priority headers.

In `main()`, create a `PriorityEmailNotification` and call `send()`. Observe how the call chain goes: PriorityEmail -> Email -> Notification, with each level adding its own behavior.

> **Hint:** Each `send()` method should call `super.send()` as its first action, then add its own logic. This creates a layered behavior where each level contributes to the final output.

### Exercise 4: Immutable Builder with `this` and Private Constructor (Hard, Optional)

Create an `HttpRequest` class that is fully immutable (all fields `final`, class `final`). Use a static nested `Builder` class where all setter methods return `this` (the Builder). The `HttpRequest` constructor should be private and accept the Builder.

Fields: `method` (String), `url` (String), `headers` (Map<String, String>), `body` (String), `timeoutMs` (int).

The Builder should:

- Require `url` in its constructor.
- Default `method` to "GET" and `timeoutMs` to 5000.
- Validate in `build()` that `url` is not blank and `timeoutMs` is positive.
- Make a defensive copy of the headers map in the `HttpRequest` constructor.

In `main()`, build several requests with different configurations and print them.

> **Hint:** The Builder's `header(String key, String value)` method adds to an internal map and returns `this`. The `build()` method creates a new `HttpRequest(this)`, passing the Builder itself to the private constructor.

<details>
<summary>Solution for Exercise 1</summary>

```java
class Vehicle {
    String make;
    int year;

    Vehicle() {
        this.make = "Unknown";
        this.year = 2025;
        System.out.println("Vehicle() constructor called");
    }

    Vehicle(String make, int year) {
        this.make = make;
        this.year = year;
        System.out.println("Vehicle(" + make + ", " + year + ") constructor called");
    }
}

class Car extends Vehicle {
    int numDoors;

    Car(String make, int year, int numDoors) {
        super(make, year);  // Chains to Vehicle(String, int)
        this.numDoors = numDoors;
        System.out.println("Car(" + make + ", " + year + ", " + numDoors + ") constructor called");
    }

    Car(String make, int year) {
        this(make, year, 4);  // Chains to the 3-arg Car constructor using this()
        System.out.println("Car(" + make + ", " + year + ") constructor called (defaulted to 4 doors)");
    }
}

class ElectricCar extends Car {
    double batteryCapacityKwh;

    ElectricCar(String make, int year, int numDoors, double batteryCapacityKwh) {
        super(make, year, numDoors);  // Chains to Car(String, int, int)
        this.batteryCapacityKwh = batteryCapacityKwh;
        System.out.println("ElectricCar constructor called");
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println("--- Creating ElectricCar ---");
        ElectricCar tesla = new ElectricCar("Tesla", 2025, 4, 75.0);
    }
}
```

**Output:**

```text
--- Creating ElectricCar ---
Vehicle(Tesla, 2025) constructor called
Car(Tesla, 2025, 4) constructor called
ElectricCar constructor called
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;

public class SqlQueryBuilder {
    private String selectClause = "*";
    private String fromClause;
    private final List<String> whereConditions = new ArrayList<>();
    private String orderByClause;
    private int limitValue = -1;

    public SqlQueryBuilder select(String... columns) {
        this.selectClause = String.join(", ", columns);
        return this;
    }

    public SqlQueryBuilder from(String table) {
        this.fromClause = table;
        return this;
    }

    public SqlQueryBuilder where(String condition) {
        this.whereConditions.add(condition);
        return this;
    }

    public SqlQueryBuilder orderBy(String column, boolean ascending) {
        this.orderByClause = column + (ascending ? " ASC" : " DESC");
        return this;
    }

    public SqlQueryBuilder limit(int count) {
        this.limitValue = count;
        return this;
    }

    public String build() {
        if (fromClause == null) throw new IllegalStateException("FROM clause is required");

        StringBuilder sql = new StringBuilder();
        sql.append("SELECT ").append(selectClause);
        sql.append(" FROM ").append(fromClause);

        if (!whereConditions.isEmpty()) {
            sql.append(" WHERE ").append(String.join(" AND ", whereConditions));
        }
        if (orderByClause != null) {
            sql.append(" ORDER BY ").append(orderByClause);
        }
        if (limitValue > 0) {
            sql.append(" LIMIT ").append(limitValue);
        }

        return sql.toString();
    }

    public static void main(String[] args) {
        String query = new SqlQueryBuilder()
            .select("id", "name", "email", "created_at")
            .from("users")
            .where("active = true")
            .where("age >= 18")
            .orderBy("created_at", false)
            .limit(50)
            .build();

        System.out.println(query);
        // SELECT id, name, email, created_at FROM users WHERE active = true AND age >= 18 ORDER BY created_at DESC LIMIT 50
    }
}
```

</details>

---

## Related Notes

- [[Java - Inheritance - Single Multilevel Hierarchical]]
- [[Java - Final Keyword]]
- [[Java - Exception Handling - Try Catch Finally Throw Throws]] (next note)

---

## Resources

- [Oracle Java Tutorials: Using the this Keyword](https://docs.oracle.com/javase/tutorial/java/javaOO/thiskey.html) - Official documentation on all uses of `this`.
- [Oracle Java Tutorials: Using the super Keyword](https://docs.oracle.com/javase/tutorial/java/IandI/super.html) - Official documentation on `super` for constructors, methods, and fields.
- [Baeldung: this Keyword in Java](https://www.baeldung.com/java-this) - Comprehensive guide with examples of all `this` use cases.
- [Baeldung: super Keyword in Java](https://www.baeldung.com/java-super-keyword) - Detailed explanation of `super` in constructors, methods, and field access.
- [Effective Java by Joshua Bloch - Item 2: Consider a Builder When Faced with Many Constructor Parameters](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive guide to the Builder pattern, which relies heavily on `this` return for fluent chaining.
