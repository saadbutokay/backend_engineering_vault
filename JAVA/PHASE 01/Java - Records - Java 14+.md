---
title: "Java - Records - Java 14+"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - records
  - java14
  - java16
  - immutability
  - dto
status: "not-started"
---

# Java - Records - Java 14+

> [!abstract] Overview
> Records are a special kind of class introduced as a preview in Java 14 and finalized in Java 16. A record is a concise, immutable data carrier that automatically generates a constructor, accessor methods, `equals()`, `hashCode()`, and `toString()` from its component declarations. Records eliminate the boilerplate that has plagued Java for decades: the dozens of lines of getters, setters, constructors, and object methods required for simple data classes. In backend development, records are the modern standard for DTOs (Data Transfer Objects), API request and response bodies, configuration properties, event payloads, and any class whose primary purpose is to hold data.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Constructors - Default Parameterized Copy]]
> - [[Java - Encapsulation - Getters Setters Access Modifiers]]
> - [[Java - Final Keyword]]
> - [[Java - Generics - Classes Methods Wildcards]]
> - [[Java - Enum and Enum Methods]]

---

## Theory

### What is a Record?

A record is a restricted form of class designed for **transparent data carriers** -- classes whose primary purpose is to hold data. When you declare a record, the compiler automatically generates:

1. **Private final fields** for each component.
2. **A canonical constructor** that initializes all fields.
3. **Public accessor methods** named after the components (not `getXxx()`, just `xxx()`).
4. **`equals()`** that compares all components by value.
5. **`hashCode()`** that is consistent with `equals()`.
6. **`toString()`** that includes the class name and all component values.

**The boilerplate problem records solve:**

```java
// Traditional Java class: 40+ lines of boilerplate
public class OrderResponse {
    private final Long id;
    private final String orderNumber;
    private final BigDecimal totalAmount;
    private final String status;

    public OrderResponse(Long id, String orderNumber, BigDecimal totalAmount, String status) {
        this.id = id;
        this.orderNumber = orderNumber;
        this.totalAmount = totalAmount;
        this.status = status;
    }

    public Long getId() { return id; }
    public String getOrderNumber() { return orderNumber; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public String getStatus() { return status; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof OrderResponse that)) return false;
        return Objects.equals(id, that.id)
            && Objects.equals(orderNumber, that.orderNumber)
            && Objects.equals(totalAmount, that.totalAmount)
            && Objects.equals(status, that.status);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, orderNumber, totalAmount, status);
    }

    @Override
    public String toString() {
        return "OrderResponse[id=" + id + ", orderNumber=" + orderNumber
            + ", totalAmount=" + totalAmount + ", status=" + status + "]";
    }
}
```

```java
// Record: 1 line, same functionality
public record OrderResponse(Long id, String orderNumber, BigDecimal totalAmount, String status) {}
```

The record declaration generates all 40+ lines of boilerplate automatically. The generated code is identical to what you would write by hand (and what IDEs generate for you), but it is maintained by the compiler, so it never goes out of sync when you add or remove a field.

### Record Syntax

```java
public record RecordName(Type1 component1, Type2 component2, ...) {
    // Optional: compact constructor, additional constructors, methods, static fields
}
```

**Key characteristics:**

1. **Implicitly final**: Records cannot be subclassed. The class is implicitly `final`.
2. **Implicitly extends `java.lang.Record`**: All records extend the abstract `Record` class, which provides the default `equals()`, `hashCode()`, and `toString()` implementations.
3. **Components are private final fields**: Each component declared in the header becomes a `private final` instance field.
4. **Accessors have no `get` prefix**: The accessor for a component named `name` is `name()`, not `getName()`. This is a deliberate design choice to distinguish records from JavaBeans.
5. **No instance fields beyond components**: You cannot add additional instance fields to a record. Records are transparent carriers of their declared components.
6. **No setters**: Records are immutable by design. There are no setter methods.

### The Canonical Constructor and Compact Constructor

The **canonical constructor** is the constructor whose parameter list matches the record components exactly. The compiler generates it automatically, but you can override it to add validation.

**Explicit canonical constructor:**

```java
public record Money(BigDecimal amount, String currency) {
    // Explicit canonical constructor: must assign all components
    public Money(BigDecimal amount, String currency) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        if (currency == null || !currency.matches("[A-Z]{3}")) {
            throw new IllegalArgumentException("Currency must be a 3-letter uppercase code");
        }
        this.amount = amount.setScale(2, RoundingMode.HALF_UP);
        this.currency = currency;
    }
}
```

**Compact constructor (preferred):**

```java
public record Money(BigDecimal amount, String currency) {
    // Compact constructor: no parentheses, no parameter list, no this assignments.
    // The compiler assigns the components automatically AFTER this block runs.
    public Money {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        if (currency == null || !currency.matches("[A-Z]{3}")) {
            throw new IllegalArgumentException("Currency must be a 3-letter uppercase code");
        }
        amount = amount.setScale(2, RoundingMode.HALF_UP);  // Reassign the parameter
        // No 'this.amount = amount' needed. The compiler does it automatically.
    }
}
```

The compact constructor is unique to records. It allows you to validate and normalize the component values before the compiler assigns them to the fields. You modify the parameters directly (e.g., `amount = amount.setScale(...)`) and the compiler assigns the final values to `this.amount` after the block completes.

### Additional Constructors

Records can have additional constructors, but they must delegate to the canonical constructor using `this(...)`.

```java
public record Point(int x, int y) {
    // Additional constructor: delegates to the canonical constructor
    public Point(int value) {
        this(value, value);  // Must delegate to canonical
    }

    public Point() {
        this(0, 0);  // Must delegate to canonical
    }
}

Point p1 = new Point(5, 10);  // Canonical
Point p2 = new Point(5);      // Additional: Point(5, 5)
Point p3 = new Point();       // Additional: Point(0, 0)
```

### Methods and Static Members

Records can have instance methods, static methods, and static fields. They cannot have instance fields beyond the declared components.

```java
public record Rectangle(double width, double height) {
    // Static field (allowed)
    public static final Rectangle UNIT = new Rectangle(1, 1);

    // Instance method (allowed)
    public double area() {
        return width * height;
    }

    public double perimeter() {
        return 2 * (width + height);
    }

    public boolean isSquare() {
        return Double.compare(width, height) == 0;
    }

    // Static method (allowed)
    public static Rectangle ofSide(double side) {
        return new Rectangle(side, side);
    }

    // Instance field (NOT allowed)
    // private String color;  // COMPILATION ERROR!
}
```

### Records and Interfaces

Records can implement interfaces. This is useful when records need to participate in polymorphic hierarchies.

```java
public interface Identifiable {
    Long id();
}

public interface Auditable {
    LocalDateTime createdAt();
    LocalDateTime updatedAt();
}

public record User(
    Long id,
    String username,
    String email,
    LocalDateTime createdAt,
    LocalDateTime updatedAt
) implements Identifiable, Auditable {
    // The record components automatically satisfy the interface contracts.
    // id() satisfies Identifiable.id()
    // createdAt() and updatedAt() satisfy Auditable
}
```

### Local Records (Java 16+)

Records can be declared inside methods, making them ideal for intermediate data structures that are only needed within a single method scope.

```java
public List<OrderSummary> getOrderSummaries(List<Order> orders) {
    // Local record: only visible within this method
    record OrderGroup(String status, long count, BigDecimal total) {}

    return orders.stream()
        .collect(Collectors.groupingBy(
            o -> o.getStatus().name(),
            Collectors.collectingAndThen(
                Collectors.toList(),
                group -> new OrderGroup(
                    group.get(0).getStatus().name(),
                    group.size(),
                    group.stream()
                        .map(Order::getTotalAmount)
                        .reduce(BigDecimal.ZERO, BigDecimal::add)
                )
            )
        ))
        .values().stream()
        .map(g -> new OrderSummary(g.status(), g.count(), g.total()))
        .toList();
}
```

### Records with Generics

Records support generic type parameters, just like regular classes.

```java
public record ApiResponse<T>(
    boolean success,
    String message,
    T data,
    List<String> errors
) {
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data, List.of());
    }

    public static <T> ApiResponse<T> error(String message, List<String> errors) {
        return new ApiResponse<>(false, message, null, errors);
    }
}

// Usage:
ApiResponse<User> response = ApiResponse.success(user);
ApiResponse<Void> error = ApiResponse.error("Not found", List.of("User not found"));
```

### Records and Pattern Matching (Java 16+)

Records integrate beautifully with pattern matching in `instanceof` checks and `switch` expressions.

```java
// Sealed interface with record implementations (Java 17+)
public sealed interface Shape permits Circle, Rectangle, Triangle {
    double area();
}

public record Circle(double radius) implements Shape {
    @Override
    public double area() { return Math.PI * radius * radius; }
}

public record Rectangle(double width, double height) implements Shape {
    @Override
    public double area() { return width * height; }
}

public record Triangle(double base, double height) implements Shape {
    @Override
    public double area() { return 0.5 * base * height; }
}
```

```java
// Pattern matching in switch (Java 21+)
public String describe(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle with radius " + c.radius();
        case Rectangle r -> "Rectangle " + r.width() + " x " + r.height();
        case Triangle t -> "Triangle with base " + t.base();
    };
    // The compiler verifies exhaustiveness because Shape is sealed.
    // If you add a new Shape subtype, the compiler warns about the missing case.
}
```

### Records vs Regular Classes

| Feature | Record | Regular Class |
|---------|--------|---------------|
| Boilerplate | Minimal (auto-generated) | Verbose (manual) |
| Immutability | Enforced (final fields, no setters) | Manual (must declare final, omit setters) |
| Inheritance | Cannot extend classes (extends Record implicitly) | Full inheritance |
| Instance fields | Only declared components | Any number |
| Accessors | `componentName()` | `getComponentName()` (convention) |
| `equals()`/`hashCode()` | Auto-generated from components | Manual or IDE-generated |
| `toString()` | Auto-generated | Manual or IDE-generated |
| Subclassing | Not allowed (implicitly final) | Allowed (unless final) |
| JPA entities | Not supported (JPA requires no-arg constructor and setters) | Fully supported |
| Use case | Data carriers, DTOs, events, config | Entities, services, complex domain objects |

### Records vs Lombok `@Value`

Before records, many Java projects used Lombok's `@Value` annotation to generate immutable classes. Records make Lombok unnecessary for data carriers:

| Feature | Record | Lombok `@Value` |
|---------|--------|----------------|
| Language support | Built into Java | Requires annotation processor |
| IDE support | Native | Requires Lombok plugin |
| Build complexity | None | Adds a dependency and annotation processor |
| Reflection | Standard Java reflection | Lombok-generated code can confuse reflection |
| Pattern matching | Full support (Java 16+) | No special support |
| Sealed hierarchies | Full support | No special support |

> [!tip] Key Insight
> Records are the single biggest reduction in Java boilerplate since generics. A typical Spring Boot backend has hundreds of DTOs, request objects, response objects, and event payloads. Before records, each of these required 30-50 lines of boilerplate. With records, each is a single line. This eliminates thousands of lines of generated code from a typical codebase, making the code easier to read, review, and maintain. The rule of thumb is simple: if a class is primarily a data carrier with no complex behavior, it should be a record.

---

## Syntax and Basic Examples

### Example 1: Basic record declaration and usage

```java
public record Student(String name, int age, String department, double cgpa) {}

public class Main {
    public static void main(String[] args) {
        // Creation
        Student saad = new Student("Saad", 22, "CSE", 3.72);

        // Accessors (note: no "get" prefix)
        System.out.println("Name: " + saad.name());         // Saad
        System.out.println("Age: " + saad.age());           // 22
        System.out.println("Department: " + saad.department()); // CSE
        System.out.println("CGPA: " + saad.cgpa());         // 3.72

        // Auto-generated toString()
        System.out.println(saad);
        // Student[name=Saad, age=22, department=CSE, cgpa=3.72]

        // Auto-generated equals() (value-based comparison)
        Student saad2 = new Student("Saad", 22, "CSE", 3.72);
        System.out.println("Equals: " + saad.equals(saad2));  // true
        System.out.println("== : " + (saad == saad2));        // false (different objects)

        // Auto-generated hashCode()
        System.out.println("Hash match: " + (saad.hashCode() == saad2.hashCode()));  // true
    }
}
```

### Example 2: Record with compact constructor validation

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public record Product(
    Long id,
    String name,
    BigDecimal price,
    String category,
    int stockQuantity
) {
    // Compact constructor: validates and normalizes components
    public Product {
        if (id != null && id <= 0) {
            throw new IllegalArgumentException("ID must be positive");
        }
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Product name cannot be empty");
        }
        if (price == null || price.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Price cannot be negative");
        }
        if (stockQuantity < 0) {
            throw new IllegalArgumentException("Stock cannot be negative");
        }

        // Normalize: strip whitespace and uppercase the category
        name = name.strip();
        category = category != null ? category.strip().toUpperCase() : "UNCATEGORIZED";
        price = price.setScale(2, RoundingMode.HALF_UP);
        // The compiler assigns these normalized values to the fields automatically.
    }

    // Additional convenience constructor
    public Product(String name, BigDecimal price) {
        this(null, name, price, null, 0);
    }

    // Instance methods
    public boolean isInStock() {
        return stockQuantity > 0;
    }

    public BigDecimal getStockValue() {
        return price.multiply(BigDecimal.valueOf(stockQuantity));
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Product laptop = new Product(1L, "  ThinkPad T14  ", new BigDecimal("85000"), "electronics", 15);
        System.out.println(laptop);
        // Product[id=1, name=ThinkPad T14, price=85000.00, category=ELECTRONICS, stockQuantity=15]
        // Note: name is stripped, category is uppercased, price has 2 decimal places

        Product mouse = new Product("Mouse", new BigDecimal("1500"));
        System.out.println(mouse);
        // Product[id=null, name=Mouse, price=1500.00, category=UNCATEGORIZED, stockQuantity=0]

        System.out.println("In stock: " + laptop.isInStock());  // true
        System.out.println("Stock value: " + laptop.getStockValue());  // 1275000.00
    }
}
```

### Example 3: Nested records for complex DTOs

```java
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

public record OrderResponse(
    Long id,
    String orderNumber,
    BigDecimal totalAmount,
    String status,
    CustomerInfo customer,
    List<OrderItemResponse> items,
    ShippingInfo shipping,
    LocalDateTime createdAt
) {
    // Nested records: each is a self-contained data carrier
    public record CustomerInfo(
        Long id,
        String name,
        String email
    ) {}

    public record OrderItemResponse(
        String productName,
        int quantity,
        BigDecimal unitPrice,
        BigDecimal subtotal
    ) {}

    public record ShippingInfo(
        String method,
        String address,
        String city,
        String zipCode,
        LocalDateTime estimatedDelivery
    ) {}

    // Factory method: converts an entity to a response DTO
    public static OrderResponse fromEntity(Order order) {
        return new OrderResponse(
            order.getId(),
            order.getOrderNumber(),
            order.getTotalAmount(),
            order.getStatus().name(),
            new CustomerInfo(
                order.getUser().getId(),
                order.getUser().getName(),
                order.getUser().getEmail()
            ),
            order.getItems().stream()
                .map(item -> new OrderItemResponse(
                    item.getProduct().getName(),
                    item.getQuantity(),
                    item.getUnitPrice(),
                    item.getUnitPrice().multiply(BigDecimal.valueOf(item.getQuantity()))
                ))
                .toList(),
            new ShippingInfo(
                order.getShippingMethod().name(),
                order.getShippingAddress().getStreet(),
                order.getShippingAddress().getCity(),
                order.getShippingAddress().getZipCode(),
                order.getEstimatedDelivery()
            ),
            order.getCreatedAt()
        );
    }
}
```

### Example 4: Records with Jackson JSON serialization

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;

public record CreateUserRequest(
    @JsonProperty("first_name") String firstName,
    @JsonProperty("last_name") String lastName,
    String email,
    int age
) {
    // Compact constructor for validation
    public CreateUserRequest {
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Valid email is required");
        }
        if (age < 13 || age > 120) {
            throw new IllegalArgumentException("Age must be between 13 and 120");
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        // Deserialize JSON to record
        String json = """
            {
                "first_name": "Abdullah",
                "last_name": "Saad",
                "email": "saad@example.com",
                "age": 22
            }
            """;

        CreateUserRequest request = mapper.readValue(json, CreateUserRequest.class);
        System.out.println("Parsed: " + request);
        // CreateUserRequest[firstName=Abdullah, lastName=Saad, email=saad@example.com, age=22]

        // Serialize record to JSON
        String output = mapper.writeValueAsString(request);
        System.out.println("JSON: " + output);
        // {"first_name":"Abdullah","last_name":"Saad","email":"saad@example.com","age":22}
    }
}
```

**What to notice:**

- Jackson (the JSON library used by Spring Boot) supports records natively since Jackson 2.12. It uses the canonical constructor for deserialization and the accessor methods for serialization.
- `@JsonProperty` annotations on record components control the JSON field names. The record uses `firstName` internally but serializes to `first_name` in JSON.
- The compact constructor validates the input during deserialization. If the JSON contains invalid data, Jackson throws an exception that Spring Boot converts to a 400 Bad Request response.

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Records have become the standard for DTOs in modern Spring Boot applications. Here are three realistic scenarios.

### Scenario 1: Complete API request/response DTO layer

```java
package com.company.orderservice.dto;

import jakarta.validation.Valid;
import jakarta.validation.constraints.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

// Request DTOs: validated by Spring's @Valid annotation
public record CreateOrderRequest(
    @NotNull(message = "User ID is required")
    @Positive(message = "User ID must be positive")
    Long userId,

    @NotEmpty(message = "Order must contain at least one item")
    @Size(max = 100, message = "Maximum 100 items per order")
    List<@Valid OrderItemRequest> items,

    @Pattern(regexp = "^[A-Z0-9]{5,10}$", message = "Invalid coupon code format")
    String couponCode
) {}

public record OrderItemRequest(
    @NotNull(message = "Product ID is required")
    @Positive(message = "Product ID must be positive")
    Long productId,

    @Min(value = 1, message = "Quantity must be at least 1")
    @Max(value = 50, message = "Maximum 50 units per item")
    int quantity
) {}

// Response DTOs: immutable data carriers
public record OrderResponse(
    Long id,
    String orderNumber,
    BigDecimal totalAmount,
    String status,
    List<OrderItemResponse> items,
    LocalDateTime createdAt
) {
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

public record OrderItemResponse(
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

// Paginated response wrapper (generic record)
public record PageResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages,
    boolean hasNext
) {}

// Error response (consistent across all endpoints)
public record ErrorResponse(
    LocalDateTime timestamp,
    int status,
    String error,
    String message,
    List<FieldError> fieldErrors
) {
    public record FieldError(String field, String message) {}

    public static ErrorResponse of(int status, String error, String message) {
        return new ErrorResponse(LocalDateTime.now(), status, error, message, null);
    }
}
```

**What to notice:**

- All DTOs are records. The entire request/response layer is defined in a fraction of the code that traditional classes would require.
- Jakarta Validation annotations (`@NotNull`, `@Positive`, `@Size`, `@Pattern`, `@Min`, `@Max`) work on record components. Spring Boot validates the request body automatically when the controller parameter is annotated with `@Valid`.
- Nested records (`OrderItemRequest` inside the same file, `FieldError` inside `ErrorResponse`) keep related data structures together and reduce the number of files.
- The `fromEntity()` factory methods convert JPA entities to DTOs. This is the standard pattern for separating the persistence layer from the API layer.

### Scenario 2: Event-driven architecture with record events

```java
package com.company.orderservice.events;

import java.math.BigDecimal;
import java.time.Instant;
import java.util.List;

// Sealed interface for type-safe event handling
public sealed interface DomainEvent {
    String eventId();
    Instant timestamp();
    String eventType();
}

// Each event type is a record: immutable, serializable, self-documenting
public record OrderCreatedEvent(
    String eventId,
    Instant timestamp,
    Long orderId,
    String orderNumber,
    Long userId,
    BigDecimal totalAmount,
    List<OrderItemSnapshot> items
) implements DomainEvent {
    @Override
    public String eventType() { return "ORDER_CREATED"; }

    public record OrderItemSnapshot(
        Long productId,
        String productName,
        int quantity,
        BigDecimal unitPrice
    ) {}
}

public record PaymentCompletedEvent(
    String eventId,
    Instant timestamp,
    Long orderId,
    String transactionId,
    BigDecimal amount,
    String paymentMethod
) implements DomainEvent {
    @Override
    public String eventType() { return "PAYMENT_COMPLETED"; }
}

public record OrderCancelledEvent(
    String eventId,
    Instant timestamp,
    Long orderId,
    String reason
) implements DomainEvent {
    @Override
    public String eventType() { return "ORDER_CANCELLED"; }
}
```

```java
// Event handler using pattern matching (Java 21+)
@Service
public class EventDispatcher {

    public void dispatch(DomainEvent event) {
        switch (event) {
            case OrderCreatedEvent e -> {
                inventoryService.reserveStock(e.orderId(), e.items());
                emailService.sendOrderConfirmation(e.orderNumber(), e.userId());
            }
            case PaymentCompletedEvent e -> {
                orderService.confirmOrder(e.orderId());
                notificationService.notifyPaymentSuccess(e.transactionId());
            }
            case OrderCancelledEvent e -> {
                inventoryService.releaseStock(e.orderId());
                refundService.processRefund(e.orderId(), e.reason());
            }
        }
        // The compiler warns if a new event type is added to the sealed interface
        // but not handled in this switch.
    }
}
```

**What to notice:**

- Events are records because they are immutable data carriers. Once an event is created, its data should never change. Records enforce this at the language level.
- The sealed interface + record combination provides exhaustive pattern matching. The compiler knows all possible event types and warns you if you forget to handle one. This is impossible with traditional class hierarchies.
- Nested records (`OrderItemSnapshot`) keep event-related data structures scoped to the event that uses them.

### Scenario 3: Configuration properties with records (Spring Boot 3+)

```java
package com.company.orderservice.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import java.time.Duration;
import java.util.List;
import java.util.Map;

// Spring Boot 3 supports records for configuration properties.
// The constructor parameters map to the YAML properties automatically.
@ConfigurationProperties(prefix = "app")
public record AppProperties(
    String name,
    String version,
    PaymentConfig payment,
    CacheConfig cache,
    List<String> allowedOrigins
) {
    public record PaymentConfig(
        String gatewayUrl,
        String apiKey,
        Duration timeout,
        int maxRetries,
        boolean sandboxMode
    ) {}

    public record CacheConfig(
        Duration ttl,
        int maxSize,
        Map<String, Duration> perEntityTtl
    ) {}
}
```

```yaml
# application.yml
app:
  name: OrderService
  version: 2.1.0
  payment:
    gateway-url: https://api.stripe.com
    api-key: sk_test_abc123
    timeout: 10s
    max-retries: 3
    sandbox-mode: true
  cache:
    ttl: 30m
    max-size: 1000
    per-entity-ttl:
      product: 1h
      user: 15m
      order: 5m
  allowed-origins:
    - https://example.com
    - https://admin.example.com
```

**What to notice:**

- Records work as `@ConfigurationProperties` classes in Spring Boot 3+. Spring uses the canonical constructor to bind YAML properties to record components.
- Nested records map to nested YAML structures. `PaymentConfig` maps to `app.payment.*`.
- Spring Boot automatically converts YAML values to the correct types: `"10s"` to `Duration.ofSeconds(10)`, `"30m"` to `Duration.ofMinutes(30)`, lists, and maps.
- The record is immutable, so configuration values cannot be accidentally modified at runtime. This is a safety improvement over mutable `@ConfigurationProperties` classes.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Trying to use records as JPA entities

**Wrong:**

```java
@Entity
@Table(name = "orders")
public record Order(
    @Id @GeneratedValue Long id,
    String orderNumber,
    BigDecimal totalAmount
) {}
// COMPILATION/RUNTIME ERROR! JPA requires:
// 1. A no-argument constructor (records do not have one)
// 2. Non-final fields (record fields are final)
// 3. Setter methods (records do not have setters)
```

**Right:**

```java
// JPA entities must be regular classes
@Entity
@Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String orderNumber;
    private BigDecimal totalAmount;
    // Getters, setters, no-arg constructor...
}

// Use records for DTOs that carry data FROM the entity TO the API
public record OrderResponse(Long id, String orderNumber, BigDecimal totalAmount) {
    public static OrderResponse fromEntity(Order order) {
        return new OrderResponse(order.getId(), order.getOrderNumber(), order.getTotalAmount());
    }
}
```

**Why it is wrong:** JPA (Hibernate) requires entities to have a no-argument constructor, mutable fields, and setter methods so it can create instances via reflection and populate fields from database rows. Records are immutable and have no no-arg constructor, making them fundamentally incompatible with JPA. Use regular classes for entities and records for DTOs.

### Mistake 2: Expecting `getXxx()` accessor names

**Wrong:**

```java
public record User(String name, String email) {}

User user = new User("Saad", "saad@test.com");
System.out.println(user.getName());   // COMPILATION ERROR! No getName() method.
System.out.println(user.getEmail());  // COMPILATION ERROR! No getEmail() method.
```

**Right:**

```java
User user = new User("Saad", "saad@test.com");
System.out.println(user.name());   // Saad (accessor is the component name)
System.out.println(user.email());  // saad@test.com
```

**Why it is wrong:** Record accessors are named after the components without the `get` prefix. This is a deliberate design decision to distinguish records from JavaBeans. If your codebase uses frameworks or libraries that expect JavaBean naming conventions (`getName()`, `getEmail()`), you may need to add explicit getter methods to the record or use Jackson's `@JsonProperty` annotations for JSON serialization.

### Mistake 3: Adding instance fields beyond the declared components

**Wrong:**

```java
public record Order(Long id, String orderNumber) {
    private BigDecimal totalAmount;  // COMPILATION ERROR!
    // Records cannot have instance fields beyond the declared components.
}
```

**Right:**

```java
// Option 1: Add the field as a component
public record Order(Long id, String orderNumber, BigDecimal totalAmount) {}

// Option 2: Compute the value in a method
public record Order(Long id, String orderNumber, BigDecimal subtotal, BigDecimal tax) {
    public BigDecimal totalAmount() {
        return subtotal.add(tax);
    }
}
```

**Why it is wrong:** Records are transparent data carriers. Their state is defined entirely by their components. Adding hidden instance fields would break the transparency contract: `equals()`, `hashCode()`, and `toString()` would not account for the hidden field, leading to subtle bugs. If you need additional state, use a regular class.

### Mistake 4: Using records for classes with complex behavior

**Wrong:**

```java
public record OrderService(
    OrderRepository repository,
    PaymentService paymentService,
    EmailService emailService
) {
    // 200 lines of business logic...
    // Records are not designed for service classes with complex behavior.
}
```

**Right:**

```java
// Services should be regular classes with injected dependencies
@Service
public class OrderService {
    private final OrderRepository repository;
    private final PaymentService paymentService;
    private final EmailService emailService;
    // Constructor injection, business methods...
}

// Records are for data, not behavior
public record OrderData(Long id, String orderNumber, BigDecimal total) {}
```

**Why it is wrong:** Records are designed for data carriers, not for classes with complex behavior, mutable state, or lifecycle management. While records can have methods, a record with hundreds of lines of business logic defeats the purpose of the feature. Use regular classes for services, repositories, and controllers. Use records for DTOs, events, configuration, and intermediate data structures.

---

## Key Takeaways

> [!tip] Remember these points
> 1. **Records** are concise, immutable data carriers introduced in Java 16. A single line `record Name(Type field)` generates a constructor, accessors, `equals()`, `hashCode()`, and `toString()` automatically. They eliminate the boilerplate that has plagued Java data classes for decades.
> 2. Record components become **private final fields** with accessor methods named after the component (no `get` prefix). Records are implicitly `final` and cannot be subclassed. They cannot have instance fields beyond the declared components.
> 3. The **compact constructor** allows validation and normalization without boilerplate assignments. Modify the parameters directly, and the compiler assigns them to the fields after the block completes. Additional constructors must delegate to the canonical constructor via `this(...)`.
> 4. Records are the modern standard for **DTOs, API request/response bodies, event payloads, and configuration properties** in Spring Boot. They integrate natively with Jackson (JSON), Jakarta Validation, and Spring's `@ConfigurationProperties`. They are NOT compatible with JPA entities.
> 5. Records support **generics, interfaces, nested declarations, local scope, and pattern matching**. Combined with sealed interfaces (Java 17+) and switch pattern matching (Java 21+), records enable exhaustive type-safe data modeling that was previously impossible in Java.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Basic Records (Easy)

Convert the following traditional classes to records:

1. A `Point` class with `x` and `y` fields (doubles). Add a method `distanceTo(Point other)` that calculates the Euclidean distance.

2. A `Money` class with `amount` (BigDecimal) and `currency` (String). Add a compact constructor that validates the amount is non-negative and the currency is a 3-letter uppercase code. Add methods `add(Money other)` and `subtract(Money other)` that return new `Money` records.

3. A `DateRange` class with `start` (LocalDate) and `end` (LocalDate). Add a compact constructor that validates `start` is before `end`. Add a method `contains(LocalDate date)` and `daysBetween()`.

Test all records with valid and invalid inputs.

> **Hint:** Records are immutable, so `add()` and `subtract()` must return new `Money` records, not modify the existing one.

### Exercise 2: Nested Record DTOs (Medium)

Create a complete API response structure using nested records for a product catalog:

1. `ProductCatalogResponse` with fields: `categoryName`, `totalProducts`, `products` (list of `ProductSummary`).

2. `ProductSummary` with fields: `id`, `name`, `price`, `rating`, `inStock`.

3. `ProductDetailResponse` with fields: all `ProductSummary` fields plus `description`, `specifications` (map of String to String), `reviews` (list of `Review`).

4. `Review` with fields: `author`, `rating`, `comment`, `createdAt`.

Add `fromEntity()` factory methods to each record. Create sample data and serialize to JSON using Jackson.

> **Hint:** Use `Map.of()` and `List.of()` to create sample data. Use Jackson's `ObjectMapper` to serialize the records to JSON strings.

### Exercise 3: Sealed Interface with Record Implementations (Medium)

Create a sealed interface `Notification` with three record implementations:

1. `EmailNotification(String to, String subject, String body)`

2. `SmsNotification(String phoneNumber, String message)`

3. `PushNotification(String deviceToken, String title, String body)`

Each record should implement a `String channel()` method from the interface. Create a method `sendNotification(Notification notification)` that uses pattern matching in a switch expression to handle each type differently. Add a new notification type and observe the compiler warning about the missing case.

> **Hint:** Use `sealed interface Notification permits EmailNotification, SmsNotification, PushNotification`. The switch expression should be exhaustive if all permitted types are covered.

### Exercise 4: Event Sourcing with Records (Hard, Optional)

Build a simplified event sourcing system using records:

1. Create a sealed interface `Event` with a `String aggregateId()` and `Instant timestamp()` method.

2. Create event records: `OrderPlaced`, `PaymentReceived`, `OrderShipped`, `OrderDelivered`, `OrderCancelled`.

3. Create an `EventStore` class that stores events in a `List<Event>` and provides methods to append events and retrieve events by aggregate ID.

4. Create an `OrderProjection` class that rebuilds the current state of an order by replaying its events. Use pattern matching to handle each event type.

5. Test by creating an order, processing a payment, shipping it, and then replaying all events to reconstruct the order state.

> **Hint:** The `OrderProjection` should maintain mutable state internally (current status, total, etc.) and update it by processing each event in chronological order. The events themselves are immutable records, but the projection is a mutable read model.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.time.LocalDate;
import java.time.temporal.ChronoUnit;

record Point(double x, double y) {
    double distanceTo(Point other) {
        return Math.sqrt(Math.pow(this.x - other.x, 2) + Math.pow(this.y - other.y, 2));
    }
}

record Money(BigDecimal amount, String currency) {
    public Money {
        if (amount.compareTo(BigDecimal.ZERO) < 0)
            throw new IllegalArgumentException("Amount cannot be negative");
        if (currency == null || !currency.matches("[A-Z]{3}"))
            throw new IllegalArgumentException("Invalid currency code");
        amount = amount.setScale(2, RoundingMode.HALF_UP);
    }
    Money add(Money other) {
        if (!this.currency.equals(other.currency))
            throw new IllegalArgumentException("Currency mismatch");
        return new Money(this.amount.add(other.amount), this.currency);
    }
    Money subtract(Money other) {
        if (!this.currency.equals(other.currency))
            throw new IllegalArgumentException("Currency mismatch");
        return new Money(this.amount.subtract(other.amount), this.currency);
    }
}

record DateRange(LocalDate start, LocalDate end) {
    public DateRange {
        if (!start.isBefore(end))
            throw new IllegalArgumentException("Start must be before end");
    }
    boolean contains(LocalDate date) {
        return !date.isBefore(start) && !date.isAfter(end);
    }
    long daysBetween() {
        return ChronoUnit.DAYS.between(start, end);
    }
}

public class Main {
    public static void main(String[] args) {
        Point p1 = new Point(0, 0);
        Point p2 = new Point(3, 4);
        System.out.println("Distance: " + p1.distanceTo(p2));  // 5.0

        Money m1 = new Money(new BigDecimal("100"), "BDT");
        Money m2 = new Money(new BigDecimal("50.5"), "BDT");
        System.out.println("Sum: " + m1.add(m2));  // Money[amount=150.50, currency=BDT]

        DateRange range = new DateRange(LocalDate.of(2025, 1, 1), LocalDate.of(2025, 12, 31));
        System.out.println("Days: " + range.daysBetween());  // 364
        System.out.println("Contains Jul 10: " + range.contains(LocalDate.of(2025, 7, 10)));  // true
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
import java.time.Instant;

sealed interface Notification {
    String channel();
}

record EmailNotification(String to, String subject, String body) implements Notification {
    @Override public String channel() { return "EMAIL"; }
}

record SmsNotification(String phoneNumber, String message) implements Notification {
    @Override public String channel() { return "SMS"; }
}

record PushNotification(String deviceToken, String title, String body) implements Notification {
    @Override public String channel() { return "PUSH"; }
}

public class Main {
    static void send(Notification n) {
        String result = switch (n) {
            case EmailNotification e -> "Email to " + e.to() + ": " + e.subject();
            case SmsNotification s -> "SMS to " + s.phoneNumber() + ": " + s.message();
            case PushNotification p -> "Push to " + p.deviceToken() + ": " + p.title();
        };
        System.out.println("[" + n.channel() + "] " + result);
    }

    public static void main(String[] args) {
        send(new EmailNotification("saad@test.com", "Order Confirmed", "Your order is confirmed."));
        send(new SmsNotification("+8801712345678", "OTP: 4829"));
        send(new PushNotification("device_abc", "Shipped!", "Your order has shipped."));
    }
}
```

</details>

---

## Related Notes

- [[Java - Enum and Enum Methods]]
- [[Java - Final Keyword]]
- [[Java - Generics - Classes Methods Wildcards]]
- [[Java - Java 8 Lambdas and Functional Interfaces]]

---

## Phase 1 Summary

> [!success] Congratulations! You have completed Phase 1: Java Core and OOP.
> You now understand:
> - Classes, objects, constructors, and encapsulation
> - Inheritance, polymorphism, and abstraction
> - Static and final keywords, `this` and `super`
> - Exception handling and custom exceptions
> - The Collections Framework (List, Set, Map, Queue, Deque)
> - Generics, Comparable, and Comparator
> - Lambdas, functional interfaces, and the Streams API
> - Optional, File I/O, and the Date and Time API
> - Enums and Records
>
> **Next: Phase 2 - Java Advanced and Build Tools**
> In Phase 2, you will learn multithreading, Maven, Gradle, logging, unit testing with JUnit 5 and Mockito, design patterns, SOLID principles, and clean code. This phase bridges the gap between knowing Java and writing production-quality backend code.

---

## Resources

- [Oracle Java Tutorials: Records](https://docs.oracle.com/en/java/javase/21/language/records.html) - Official documentation covering record syntax, semantics, and restrictions.
- [Oracle Java Documentation: java.lang.Record](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html) - API reference for the Record base class.
- [Baeldung: Java Records](https://www.baeldung.com/java-record-keyword) - Comprehensive guide covering all record features with examples.
- [Baeldung: Java Records with Jackson](https://www.baeldung.com/java-record-jackson) - Guide to JSON serialization and deserialization of records.
- [Baeldung: Java Sealed Classes and Interfaces](https://www.baeldung.com/java-sealed-classes-interfaces) - Guide to sealed types with record implementations.
- [Effective Java by Joshua Bloch - Item 1: Consider Static Factory Methods Instead of Constructors](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Related to record factory methods.
- [JEP 395: Records (Final)](https://openjdk.org/jeps/395) - The official Java Enhancement Proposal that finalized records in Java 16.
