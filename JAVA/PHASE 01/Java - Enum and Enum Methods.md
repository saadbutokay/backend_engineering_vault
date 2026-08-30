---
title: "Java - Enum and Enum Methods"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - enum
  - type-safety
  - jpa
status: "not-started"
---

# Java - Enum and Enum Methods

> [!abstract] Overview
> An `enum` (enumeration) is a special Java type that represents a fixed set of named constants. Unlike integer constants or string literals, enums provide compile-time type safety, namespace isolation, and the ability to attach fields, methods, and behavior to each constant. In backend development, enums are the standard way to model finite, well-defined sets of values: order statuses, user roles, payment methods, notification channels, HTTP methods, and error codes. They integrate seamlessly with JPA for database persistence, Jackson for JSON serialization, and Spring for request parameter binding.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Constructors - Default Parameterized Copy]]
> - [[Java - Abstraction - Abstract Classes and Interfaces]]
> - [[Java - Control Flow - If Else Switch]]
> - [[Java - Static Keyword - Variables Methods Blocks]]

---

## Theory

### What is an Enum?

An enum is a special class that defines a **fixed set of instances** (constants). Each constant is a singleton object of the enum type, created automatically by the JVM when the enum class is loaded.

Before enums (pre-Java 5), developers used integer or string constants:

```java
// Pre-Java 5: integer constants (error-prone)
public static final int ORDER_PENDING = 0;
public static final int ORDER_CONFIRMED = 1;
public static final int ORDER_SHIPPED = 2;

int status = ORDER_PENDING;
status = 99;  // No compile-time error! Invalid value accepted.
if (status == ORDER_CONFIRMED) { ... }  // What does 1 mean? Not self-documenting.
```

With enums:

```java
// Java 5+: type-safe enum
public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}

OrderStatus status = OrderStatus.PENDING;
// status = 99;  // COMPILATION ERROR! Only enum constants are allowed.
if (status == OrderStatus.CONFIRMED) { ... }  // Self-documenting and type-safe.
```

**Key properties of enums:**

1. **Fixed set of instances**: The constants are defined at compile time. No new instances can be created at runtime (the constructor is implicitly private).
2. **Singleton**: Each constant is a single, shared instance. `OrderStatus.PENDING == OrderStatus.PENDING` is always `true`.
3. **Type-safe**: A variable of type `OrderStatus` can only hold one of the defined constants. Assigning an integer, string, or unrelated enum is a compilation error.
4. **Full-fledged classes**: Enums can have fields, constructors, methods, and even implement interfaces. Each constant can have its own state and behavior.
5. **Implicitly final**: Enum classes are implicitly `final` (unless they have abstract methods, in which case each constant is an anonymous subclass). They cannot be instantiated with `new` or subclassed externally.
6. **Extend `java.lang.Enum`**: All enums implicitly extend `java.lang.Enum<E>`, which provides `name()`, `ordinal()`, `compareTo()`, and `equals()`.

### Basic Enum Syntax

```java
public enum OrderStatus {
    PENDING,        // Constant 0
    CONFIRMED,      // Constant 1
    SHIPPED,        // Constant 2
    DELIVERED,      // Constant 3
    CANCELLED       // Constant 4
    // Semicolon is optional when there are no fields or methods
}
```

### Enum with Fields and Constructors

Enums can have instance fields and constructors to associate data with each constant. The constructor is called once for each constant when the enum class is loaded.

```java
public enum OrderStatus {
    PENDING("Pending Review", 1),
    CONFIRMED("Order Confirmed", 2),
    SHIPPED("In Transit", 3),
    DELIVERED("Delivered", 4),
    CANCELLED("Cancelled", 0);  // Semicolon required before fields/methods

    private final String displayName;
    private final int priority;

    // Constructor is implicitly private. Cannot be called from outside.
    OrderStatus(String displayName, int priority) {
        this.displayName = displayName;
        this.priority = priority;
    }

    public String getDisplayName() { return displayName; }
    public int getPriority() { return priority; }
}
```

### Enum with Abstract Methods (Constant-Specific Behavior)

Each enum constant can override abstract methods to provide its own behavior. This is the **Strategy pattern** implemented with enums.

```java
import java.math.BigDecimal;

public enum PaymentMethod {
    CREDIT_CARD {
        @Override
        public BigDecimal calculateFee(BigDecimal amount) {
            return amount.multiply(new BigDecimal("0.025"));  // 2.5%
        }

        @Override
        public String getGateway() {
            return "Stripe";
        }
    },

    BKASH {
        @Override
        public BigDecimal calculateFee(BigDecimal amount) {
            return amount.multiply(new BigDecimal("0.015"));  // 1.5%
        }

        @Override
        public String getGateway() {
            return "bKash Gateway";
        }
    },

    CASH_ON_DELIVERY {
        @Override
        public BigDecimal calculateFee(BigDecimal amount) {
            return BigDecimal.ZERO;  // No fee
        }

        @Override
        public String getGateway() {
            return "N/A";
        }
    };

    // Abstract methods: each constant MUST implement these
    public abstract BigDecimal calculateFee(BigDecimal amount);
    public abstract String getGateway();
}
```

### Built-in Enum Methods

Every enum inherits these methods from `java.lang.Enum<E>`:

| Method | Return | Description |
|--------|--------|-------------|
| `name()` | `String` | Returns the exact name of the constant as declared (e.g., `"PENDING"`) |
| `ordinal()` | `int` | Returns the zero-based position of the constant (e.g., `0` for the first) |
| `toString()` | `String` | Returns `name()` by default. Can be overridden. |
| `compareTo(E other)` | `int` | Compares by ordinal. Returns negative, zero, or positive. |
| `equals(Object)` | `boolean` | Identity comparison (`==`). Enum constants are singletons. |
| `hashCode()` | `int` | Based on identity. Consistent with `equals()`. |
| `getDeclaringClass()` | `Class<E>` | Returns the enum class that declared this constant. |

**Static methods generated by the compiler:**

| Method | Return | Description |
|--------|--------|-------------|
| `values()` | `E[]` | Returns an array of all constants in declaration order |
| `valueOf(String)` | `E` | Returns the constant with the matching name. Throws `IllegalArgumentException` if not found. |

```java
OrderStatus[] allStatuses = OrderStatus.values();
// [PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED]

OrderStatus status = OrderStatus.valueOf("SHIPPED");  // OrderStatus.SHIPPED
OrderStatus invalid = OrderStatus.valueOf("REFUNDED");  // IllegalArgumentException!
```

### `EnumSet` and `EnumMap`

Java provides specialized collection implementations optimized for enum keys:

**`EnumSet<E>`**: A high-performance `Set` implementation for enum types. Internally backed by a bit vector, making it extremely fast and memory-efficient.

```java
import java.util.EnumSet;

// Create an EnumSet with specific elements
EnumSet<OrderStatus> activeStatuses = EnumSet.of(
    OrderStatus.PENDING, OrderStatus.CONFIRMED, OrderStatus.SHIPPED
);

// All constants
EnumSet<OrderStatus> all = EnumSet.allOf(OrderStatus.class);

// Range of constants (in declaration order)
EnumSet<OrderStatus> inProgress = EnumSet.range(
    OrderStatus.CONFIRMED, OrderStatus.SHIPPED
);

// Complement (all constants NOT in the set)
EnumSet<OrderStatus> terminal = EnumSet.complementOf(activeStatuses);
// [DELIVERED, CANCELLED]
```

**`EnumMap<K, V>`**: A high-performance `Map` implementation with enum keys. Internally backed by an array, making it faster than `HashMap` for enum keys.

```java
import java.util.EnumMap;

EnumMap<OrderStatus, Integer> statusCounts = new EnumMap<>(OrderStatus.class);
statusCounts.put(OrderStatus.PENDING, 15);
statusCounts.put(OrderStatus.CONFIRMED, 42);
statusCounts.put(OrderStatus.SHIPPED, 28);
// Iteration order is the declaration order of the enum constants
```

### Enums in Switch Statements

Enums work naturally with `switch` statements and expressions. The compiler can verify exhaustiveness when using switch expressions (Java 14+).

```java
// Traditional switch
switch (status) {
    case PENDING:
        System.out.println("Awaiting confirmation");
        break;
    case CONFIRMED:
        System.out.println("Order confirmed");
        break;
    case SHIPPED:
        System.out.println("In transit");
        break;
    case DELIVERED:
        System.out.println("Delivered");
        break;
    case CANCELLED:
        System.out.println("Cancelled");
        break;
}

// Switch expression (Java 14+)
String message = switch (status) {
    case PENDING -> "Awaiting confirmation";
    case CONFIRMED -> "Order confirmed";
    case SHIPPED -> "In transit";
    case DELIVERED -> "Delivered successfully";
    case CANCELLED -> "Order cancelled";
};
// The compiler warns if a case is missing (when the enum changes).
```

### Enums Implementing Interfaces

Enums can implement interfaces, which is useful when different enum types share a common contract.

```java
public interface Displayable {
    String getDisplayName();
    String getDescription();
}

public enum UserRole implements Displayable {
    ADMIN("Administrator", "Full system access"),
    MANAGER("Manager", "Department-level access"),
    CUSTOMER("Customer", "Own orders and profile only");

    private final String displayName;
    private final String description;

    UserRole(String displayName, String description) {
        this.displayName = displayName;
        this.description = description;
    }

    @Override
    public String getDisplayName() { return displayName; }

    @Override
    public String getDescription() { return description; }
}
```

### How Enums Work Internally

At the bytecode level, an enum is compiled into a final class that extends `java.lang.Enum<E>`. Each constant is a `public static final` field initialized in a static block.

```java
// What you write:
public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED
}

// What the compiler generates (conceptually):
public final class OrderStatus extends Enum<OrderStatus> {
    public static final OrderStatus PENDING = new OrderStatus("PENDING", 0);
    public static final OrderStatus CONFIRMED = new OrderStatus("CONFIRMED", 1);
    public static final OrderStatus SHIPPED = new OrderStatus("SHIPPED", 2);

    private static final OrderStatus[] $VALUES = {PENDING, CONFIRMED, SHIPPED};

    public static OrderStatus[] values() {
        return $VALUES.clone();
    }

    public static OrderStatus valueOf(String name) {
        return Enum.valueOf(OrderStatus.class, name);
    }

    private OrderStatus(String name, int ordinal) {
        super(name, ordinal);
    }
}
```

This means enum constants are **singletons** created at class loading time. They are thread-safe by default because they are immutable and initialized before any thread can access them.

> [!tip] Key Insight
> Enums are the most underused feature in Java backend development. Many developers still use string constants (`"PENDING"`, `"CONFIRMED"`) or integer codes (`0`, `1`, `2`) for status fields, missing out on compile-time type safety, IDE autocompletion, refactoring support, and the ability to attach behavior to each status. If a value has a finite, well-known set of possibilities, it should be an enum. The only exception is when the set of values is truly dynamic and can change at runtime (e.g., user-defined tags), in which case a database table or a `String` is more appropriate.

---

## Syntax and Basic Examples

### Example 1: Basic enum with fields and methods

```java
import java.util.Optional;

public enum HttpStatus {
    OK(200, "Success"),
    CREATED(201, "Resource created"),
    BAD_REQUEST(400, "Invalid request"),
    UNAUTHORIZED(401, "Authentication required"),
    FORBIDDEN(403, "Access denied"),
    NOT_FOUND(404, "Resource not found"),
    CONFLICT(409, "Resource conflict"),
    INTERNAL_ERROR(500, "Internal server error");

    private final int code;
    private final String reasonPhrase;

    HttpStatus(int code, String reasonPhrase) {
        this.code = code;
        this.reasonPhrase = reasonPhrase;
    }

    public int getCode() { return code; }
    public String getReasonPhrase() { return reasonPhrase; }

    public boolean isSuccess() {
        return code >= 200 && code < 300;
    }

    public boolean isClientError() {
        return code >= 400 && code < 500;
    }

    public boolean isServerError() {
        return code >= 500;
    }

    // Custom lookup method (safer than valueOf)
    public static Optional<HttpStatus> fromCode(int code) {
        for (HttpStatus status : values()) {
            if (status.code == code) {
                return Optional.of(status);
            }
        }
        return Optional.empty();
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        HttpStatus status = HttpStatus.NOT_FOUND;

        System.out.println("Name: " + status.name());           // NOT_FOUND
        System.out.println("Code: " + status.getCode());        // 404
        System.out.println("Reason: " + status.getReasonPhrase()); // Resource not found
        System.out.println("Is success: " + status.isSuccess());   // false
        System.out.println("Is client error: " + status.isClientError()); // true

        // values() and valueOf()
        System.out.println("\nAll statuses:");
        for (HttpStatus s : HttpStatus.values()) {
            System.out.printf("  %d %s%n", s.getCode(), s.getReasonPhrase());
        }

        // Custom lookup
        HttpStatus.fromCode(200).ifPresent(s ->
            System.out.println("\n200 = " + s.name()));  // OK

        System.out.println("999 = " + HttpStatus.fromCode(999));  // Optional.empty
    }
}
```

### Example 2: Enum with constant-specific behavior

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public enum ShippingMethod {
    STANDARD(5, "Standard Delivery (5-7 days)") {
        @Override
        public BigDecimal calculateCost(BigDecimal orderTotal) {
            return orderTotal.compareTo(new BigDecimal("1000")) >= 0
                ? BigDecimal.ZERO  // Free shipping over 1000 BDT
                : new BigDecimal("120");
        }
    },

    EXPRESS(2, "Express Delivery (2-3 days)") {
        @Override
        public BigDecimal calculateCost(BigDecimal orderTotal) {
            return new BigDecimal("250");
        }
    },

    SAME_DAY(0, "Same Day Delivery") {
        @Override
        public BigDecimal calculateCost(BigDecimal orderTotal) {
            return new BigDecimal("500");
        }
    },

    PICKUP(0, "Store Pickup") {
        @Override
        public BigDecimal calculateCost(BigDecimal orderTotal) {
            return BigDecimal.ZERO;  // Always free
        }
    };

    private final int estimatedDays;
    private final String description;

    ShippingMethod(int estimatedDays, String description) {
        this.estimatedDays = estimatedDays;
        this.description = description;
    }

    // Abstract method: each constant provides its own implementation
    public abstract BigDecimal calculateCost(BigDecimal orderTotal);

    public int getEstimatedDays() { return estimatedDays; }
    public String getDescription() { return description; }
}
```

```java
import java.math.BigDecimal;

public class Main {
    public static void main(String[] args) {
        BigDecimal orderTotal = new BigDecimal("800");

        for (ShippingMethod method : ShippingMethod.values()) {
            BigDecimal cost = method.calculateCost(orderTotal);
            System.out.printf("%-25s | Days: %d | Cost: %s BDT%n",
                method.getDescription(), method.getEstimatedDays(), cost);
        }
        // Standard Delivery (5-7 days) | Days: 5 | Cost: 120 BDT
        // Express Delivery (2-3 days)  | Days: 2 | Cost: 250 BDT
        // Same Day Delivery            | Days: 0 | Cost: 500 BDT
        // Store Pickup                 | Days: 0 | Cost: 0 BDT
    }
}
```

### Example 3: EnumSet and EnumMap

```java
import java.util.*;

public class EnumCollectionsDemo {
    public static void main(String[] args) {
        // EnumSet: efficient set of enum constants
        EnumSet<OrderStatus> activeStatuses = EnumSet.of(
            OrderStatus.PENDING, OrderStatus.CONFIRMED, OrderStatus.SHIPPED
        );

        EnumSet<OrderStatus> terminalStatuses = EnumSet.complementOf(activeStatuses);
        System.out.println("Active: " + activeStatuses);    // [PENDING, CONFIRMED, SHIPPED]
        System.out.println("Terminal: " + terminalStatuses); // [DELIVERED, CANCELLED]

        EnumSet<OrderStatus> inProgress = EnumSet.range(
            OrderStatus.CONFIRMED, OrderStatus.SHIPPED
        );
        System.out.println("In progress: " + inProgress);  // [CONFIRMED, SHIPPED]

        // EnumMap: efficient map with enum keys
        EnumMap<OrderStatus, Long> statusCounts = new EnumMap<>(OrderStatus.class);
        statusCounts.put(OrderStatus.PENDING, 15L);
        statusCounts.put(OrderStatus.CONFIRMED, 42L);
        statusCounts.put(OrderStatus.SHIPPED, 28L);
        statusCounts.put(OrderStatus.DELIVERED, 156L);
        statusCounts.put(OrderStatus.CANCELLED, 8L);

        System.out.println("\nOrder counts by status:");
        statusCounts.forEach((status, count) ->
            System.out.printf("  %-12s: %d%n", status.name(), count)
        );
        // Iteration follows the enum declaration order
    }
}
```

### Example 4: Enum with reverse lookup and safe parsing

```java
import java.util.*;

public enum PaymentStatus {
    INITIATED("init"),
    PROCESSING("proc"),
    COMPLETED("comp"),
    FAILED("fail"),
    REFUNDED("ref");

    private final String code;

    // Static map for O(1) reverse lookup
    private static final Map<String, PaymentStatus> CODE_MAP = new HashMap<>();

    // Static initializer: runs once when the enum is loaded
    static {
        for (PaymentStatus status : values()) {
            CODE_MAP.put(status.code, status);
        }
    }

    PaymentStatus(String code) {
        this.code = code;
    }

    public String getCode() { return code; }

    // Safe reverse lookup: returns Optional instead of throwing
    public static Optional<PaymentStatus> fromCode(String code) {
        return Optional.ofNullable(CODE_MAP.get(code));
    }

    // Safe parsing from string: handles null and invalid values
    public static Optional<PaymentStatus> fromName(String name) {
        if (name == null || name.isBlank()) {
            return Optional.empty();
        }
        try {
            return Optional.of(valueOf(name.toUpperCase().strip()));
        } catch (IllegalArgumentException e) {
            return Optional.empty();
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        // Reverse lookup by code
        PaymentStatus.fromCode("comp").ifPresent(s ->
            System.out.println("Code 'comp' = " + s.name()));  // COMPLETED

        System.out.println("Code 'xyz' = " + PaymentStatus.fromCode("xyz"));
        // Optional.empty

        // Safe parsing from name
        System.out.println("Name 'FAILED' = " + PaymentStatus.fromName("FAILED"));
        // Optional[FAILED]

        System.out.println("Name 'invalid' = " + PaymentStatus.fromName("invalid"));
        // Optional.empty

        System.out.println("Name null = " + PaymentStatus.fromName(null));
        // Optional.empty
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Enums are used extensively in Spring Boot backends for modeling domain concepts. Here are three realistic scenarios.

### Scenario 1: JPA entity with enum fields

JPA supports two strategies for persisting enums to the database: by name (string) or by ordinal (integer).

```java
package com.company.orderservice.model;

import jakarta.persistence.*;

@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String orderNumber;

    // EnumType.STRING: stores the enum name as a VARCHAR (e.g., "PENDING")
    // This is the RECOMMENDED approach because it survives enum reordering.
    // If you add a new constant or reorder the enum, the database data is unaffected.
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private OrderStatus status;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private PaymentMethod paymentMethod;

    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private ShippingMethod shippingMethod;

    // WARNING: EnumType.ORDINAL stores the enum's position as an INTEGER (0, 1, 2...)
    // This is DANGEROUS because reordering the enum constants changes the meaning
    // of existing database rows. Avoid EnumType.ORDINAL in production.
    // @Enumerated(EnumType.ORDINAL)  // BAD!
    // private OrderStatus status;

    // Business methods using enum comparison
    public boolean canBeCancelled() {
        return this.status == OrderStatus.PENDING
            || this.status == OrderStatus.CONFIRMED;
    }

    public boolean isTerminal() {
        return this.status == OrderStatus.DELIVERED
            || this.status == OrderStatus.CANCELLED;
    }

    public void transitionTo(OrderStatus newStatus) {
        if (!this.status.canTransitionTo(newStatus)) {
            throw new OrderStateException(this.id, this.status.name(), newStatus.name());
        }
        this.status = newStatus;
    }

    // Getters, setters, constructors...
}
```

**What to notice:**

- `@Enumerated(EnumType.STRING)` stores the enum constant's name as a string in the database. This is the recommended approach because it is human-readable and survives enum reordering. The trade-off is slightly more storage space compared to integers.
- `@Enumerated(EnumType.ORDINAL)` stores the enum's position (0, 1, 2...) as an integer. This is fragile: if you add a new constant in the middle of the enum or reorder the constants, existing database rows will map to the wrong enum values. **Never use `EnumType.ORDINAL` in production.**
- Enum comparison uses `==` instead of `.equals()`. This is safe because enum constants are singletons. `this.status == OrderStatus.PENDING` is the idiomatic way to compare enums in Java.

### Scenario 2: Enum with state transition validation

```java
package com.company.orderservice.model;

import java.util.*;

public enum OrderStatus {
    PENDING {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return EnumSet.of(CONFIRMED, CANCELLED);
        }
    },
    CONFIRMED {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return EnumSet.of(PROCESSING, CANCELLED);
        }
    },
    PROCESSING {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return EnumSet.of(SHIPPED, CANCELLED);
        }
    },
    SHIPPED {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return EnumSet.of(DELIVERED);
        }
    },
    DELIVERED {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return EnumSet.of(REFUNDED);
        }
    },
    CANCELLED {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return EnumSet.noneOf(OrderStatus.class);  // Terminal state
        }
    },
    REFUNDED {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return EnumSet.noneOf(OrderStatus.class);  // Terminal state
        }
    };

    // Abstract method: each constant defines its own valid transitions
    public abstract Set<OrderStatus> allowedTransitions();

    public boolean canTransitionTo(OrderStatus target) {
        return allowedTransitions().contains(target);
    }

    public boolean isTerminal() {
        return allowedTransitions().isEmpty();
    }
}
```

**What to notice:**

- Each enum constant overrides `allowedTransitions()` to define its valid state transitions using `EnumSet`. This encodes the business rules directly in the enum, making it impossible to perform an invalid transition without going through `canTransitionTo()`.
- `EnumSet.of()` and `EnumSet.noneOf()` create efficient bit-vector sets. Checking `contains()` on an `EnumSet` is an O(1) bitwise operation.
- This pattern eliminates the need for a separate state machine class. The enum IS the state machine.

### Scenario 3: Enum for API error codes

```java
package com.company.orderservice.exception;

public enum ErrorCode {
    // Authentication
    INVALID_TOKEN(1001, "Authentication token is invalid or expired", 401),
    TOKEN_EXPIRED(1002, "Authentication token has expired", 401),
    INSUFFICIENT_PERMISSIONS(1003, "You do not have permission to perform this action", 403),

    // Validation
    VALIDATION_ERROR(2001, "Request validation failed", 400),
    INVALID_EMAIL(2002, "Invalid email format", 400),
    INVALID_PHONE(2003, "Invalid phone number format", 400),

    // Resources
    ORDER_NOT_FOUND(3001, "Order not found", 404),
    USER_NOT_FOUND(3002, "User not found", 404),
    PRODUCT_NOT_FOUND(3003, "Product not found", 404),
    ORDER_ALREADY_EXISTS(3004, "Order with this number already exists", 409),

    // Business rules
    INSUFFICIENT_STOCK(4001, "Insufficient stock for the requested items", 422),
    ORDER_ALREADY_CANCELLED(4002, "Order is already cancelled", 422),
    ORDER_ALREADY_SHIPPED(4003, "Cannot modify a shipped order", 422),
    COUPON_EXPIRED(4004, "Coupon code has expired", 422),

    // External services
    PAYMENT_FAILED(5001, "Payment processing failed", 502),
    EMAIL_SERVICE_UNAVAILABLE(5002, "Email service is temporarily unavailable", 503),

    // Internal
    INTERNAL_ERROR(9999, "An unexpected error occurred", 500);

    private final int code;
    private final String defaultMessage;
    private final int httpStatus;

    ErrorCode(int code, String defaultMessage, int httpStatus) {
        this.code = code;
        this.defaultMessage = defaultMessage;
        this.httpStatus = httpStatus;
    }

    public int getCode() { return code; }
    public String getDefaultMessage() { return defaultMessage; }
    public int getHttpStatus() { return httpStatus; }
}
```

```java
// Usage in a custom exception:
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;

public class AppException extends RuntimeException {
    private final ErrorCode errorCode;

    public AppException(ErrorCode errorCode) {
        super(errorCode.getDefaultMessage());
        this.errorCode = errorCode;
    }

    public AppException(ErrorCode errorCode, String customMessage) {
        super(customMessage);
        this.errorCode = errorCode;
    }

    public ErrorCode getErrorCode() { return errorCode; }
    public int getHttpStatus() { return errorCode.getHttpStatus(); }
}

// Usage in a service:
throw new AppException(ErrorCode.INSUFFICIENT_STOCK,
    "Only 3 units of 'Laptop' available, requested 5");

// Usage in the global exception handler:
@ExceptionHandler(AppException.class)
public ResponseEntity<ErrorResponse> handleAppException(AppException ex) {
    return ResponseEntity.status(ex.getHttpStatus()).body(new ErrorResponse(
        ex.getErrorCode().getCode(),
        ex.getErrorCode().name(),
        ex.getMessage()
    ));
}
```

**What to notice:**

- The `ErrorCode` enum centralizes all error codes, messages, and HTTP statuses in a single, searchable location. This makes it easy to maintain a consistent API error contract and generate API documentation.
- Each error code has a unique integer code (for machine consumption), a default message (for human consumption), and an HTTP status (for the response). This three-part structure is the industry standard for REST API error handling.
- The enum's `name()` method provides a machine-readable string identifier (e.g., `"INSUFFICIENT_STOCK"`) that frontend applications can use for localization and conditional logic.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using `EnumType.ORDINAL` in JPA

**Wrong:**

```java
@Enumerated(EnumType.ORDINAL)  // Stores 0, 1, 2, 3, 4 in the database
@Column(nullable = false)
private OrderStatus status;

// If the enum is later reordered:
// PENDING(0), CONFIRMED(1), SHIPPED(2), CANCELLED(3), DELIVERED(4)
// (CANCELLED and DELIVERED swapped positions)
// Now all DELIVERED orders in the database are read as CANCELLED!
```

**Right:**

```java
@Enumerated(EnumType.STRING)  // Stores "PENDING", "CONFIRMED", etc.
@Column(nullable = false, length = 20)
private OrderStatus status;

// Reordering the enum constants has no effect on the database.
// The string "DELIVERED" still maps to OrderStatus.DELIVERED.
```

**Why it is wrong:** `EnumType.ORDINAL` stores the enum's position as an integer. If you add, remove, or reorder enum constants, the ordinal values shift, and existing database rows map to the wrong enum values. This is a silent data corruption bug that is extremely difficult to detect. Always use `EnumType.STRING`.

### Mistake 2: Using `valueOf()` without error handling

**Wrong:**

```java
// Client sends ?status=pendng (typo)
OrderStatus status = OrderStatus.valueOf(requestParam);  // IllegalArgumentException!
// The application crashes with a 500 error instead of a helpful 400.
```

**Right:**

```java
// Safe parsing with error handling
public static Optional<OrderStatus> parseStatus(String value) {
    if (value == null || value.isBlank()) {
        return Optional.empty();
    }
    try {
        return Optional.of(OrderStatus.valueOf(value.toUpperCase().strip()));
    } catch (IllegalArgumentException e) {
        return Optional.empty();
    }
}

// Usage:
OrderStatus status = parseStatus(requestParam)
    .orElseThrow(() -> new ValidationException("status",
        "Invalid status. Valid values: " + Arrays.toString(OrderStatus.values())));
```

**Why it is wrong:** `valueOf()` throws `IllegalArgumentException` when the input does not match any constant name. In a backend API, this unhandled exception results in a 500 Internal Server Error with a stack trace, which is both unhelpful to the client and a potential security risk. Always wrap `valueOf()` in a try-catch or use a custom lookup method that returns `Optional`.

### Mistake 3: Relying on `ordinal()` for business logic

**Wrong:**

```java
// Using ordinal to determine priority
if (status1.ordinal() < status2.ordinal()) {
    // Assumes the declaration order matches the business priority.
    // This breaks if someone reorders the constants or adds a new one.
}
```

**Right:**

```java
// Use an explicit priority field
public enum OrderStatus {
    PENDING(1), CONFIRMED(2), SHIPPED(3), DELIVERED(4), CANCELLED(0);

    private final int priority;
    OrderStatus(int priority) { this.priority = priority; }
    public int getPriority() { return priority; }
}

if (status1.getPriority() < status2.getPriority()) {
    // Explicit, self-documenting, and survives reordering.
}
```

**Why it is wrong:** `ordinal()` returns the zero-based position of the constant in the enum declaration. It is an implementation detail, not a business concept. If someone adds a new constant or reorders the enum, the ordinal values change and the business logic breaks silently. Use an explicit field for any business-relevant ordering.

### Mistake 4: Using strings instead of enums for fixed value sets

**Wrong:**

```java
public class Order {
    private String status;  // "PENDING", "CONFIRMED", "SHIPPED"...

    public void setStatus(String status) {
        this.status = status;  // Accepts ANY string: "PENDNG", "pizza", "42"...
    }
}
```

**Right:**

```java
public class Order {
    private OrderStatus status;  // Type-safe: only valid enum constants

    public void setStatus(OrderStatus status) {
        this.status = status;  // Compiler enforces valid values
    }
}
```

**Why it is wrong:** String constants provide no compile-time type safety. Any string can be assigned, including typos and completely invalid values. The error is only discovered at runtime, often in production. Enums catch these errors at compile time, provide IDE autocompletion, and make the code self-documenting.

---

## Key Takeaways

> [!tip] Remember these points
> 1. An **enum** is a type-safe, fixed set of named constants. Each constant is a singleton instance of the enum class. Use enums for any value that has a finite, well-known set of possibilities: statuses, roles, types, methods, channels.
> 2. Enums can have **fields, constructors, and methods**. Each constant can have its own state and behavior. Use abstract methods in the enum to implement the Strategy pattern, where each constant provides its own implementation.
> 3. The built-in methods `values()` and `valueOf(String)` are generated by the compiler. `name()` and `ordinal()` are inherited from `java.lang.Enum`. Always use `name()` for serialization, never `ordinal()`. Wrap `valueOf()` in error handling for user input.
> 4. **`EnumSet`** and **`EnumMap`** are high-performance collection implementations optimized for enum keys. Use `EnumSet` instead of `HashSet<EnumType>` and `EnumMap` instead of `HashMap<EnumType, V>` for better performance and memory efficiency.
> 5. In JPA, always use `@Enumerated(EnumType.STRING)` to persist enums as their name strings. Never use `EnumType.ORDINAL` because reordering enum constants silently corrupts database data. In Spring Boot APIs, enums integrate automatically with JSON serialization (Jackson) and request parameter binding.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Basic Enum with Fields (Easy)

Create an enum `DayOfWeek` (or use the built-in `java.time.DayOfWeek`) with custom fields: `displayName` (String, e.g., "Monday"), `isWeekend` (boolean), and `shortName` (String, e.g., "Mon"). Add methods:

- `isWeekday()`: returns the opposite of `isWeekend`.
- `nextDay()`: returns the next day of the week (wrapping from Sunday to Monday).
- `daysUntil(DayOfWeek target)`: calculates the number of days until the target day.

Test all methods and print the results.

> **Hint:** For `nextDay()`, use `values()[(this.ordinal() + 1) % 7]`. For `daysUntil()`, calculate the difference in ordinals with wrapping.

### Exercise 2: Enum with Constant-Specific Behavior (Medium)

Create an enum `FileType` with constants `PDF`, `CSV`, `JSON`, `XML`, `IMAGE`. Each constant should have:

- A `mimeType` field (e.g., "application/pdf").
- A `maxSizeMb` field (e.g., 10 for PDF, 5 for CSV, 1 for JSON, 2 for XML, 20 for IMAGE).
- An abstract method `boolean isValidContent(byte[] content)` that performs a simple validation (e.g., check the magic bytes for PDF, check for valid JSON structure, etc.). For simplicity, you can check the first few bytes or just return true/false based on content length.
- A method `boolean isWithinSizeLimit(long sizeBytes)` that checks if the file size is within the limit.

Test with simulated file sizes and content.

> **Hint:** Use constant-specific class bodies for the `isValidContent()` method. Each constant overrides it with its own validation logic.

### Exercise 3: Enum-Based State Machine (Medium)

Create an enum `TicketStatus` for a customer support ticket system with constants: `OPEN`, `IN_PROGRESS`, `WAITING_FOR_CUSTOMER`, `RESOLVED`, `CLOSED`, `REOPENED`. Each constant should define its allowed transitions using `EnumSet`:

- OPEN -> IN_PROGRESS, CLOSED
- IN_PROGRESS -> WAITING_FOR_CUSTOMER, RESOLVED, CLOSED
- WAITING_FOR_CUSTOMER -> IN_PROGRESS, CLOSED
- RESOLVED -> CLOSED, REOPENED
- CLOSED -> REOPENED
- REOPENED -> IN_PROGRESS

Add methods `canTransitionTo(TicketStatus target)` and `getAllowedTransitions()`. Write a `Ticket` class that uses this enum and validates transitions. Test valid and invalid transitions.

> **Hint:** Use the same pattern as the `OrderStatus` example in the Real Backend Code section. Each constant overrides an abstract `allowedTransitions()` method.

### Exercise 4: API Error Code Registry (Hard, Optional)

Build a complete error code system using enums:

1. Create an enum `ErrorCategory` with constants: `AUTHENTICATION`, `VALIDATION`, `RESOURCE`, `BUSINESS`, `EXTERNAL`, `INTERNAL`. Each has a numeric prefix (1xxx, 2xxx, 3xxx, 4xxx, 5xxx, 9xxx).
2. Create an enum `ErrorCode` that associates each error with a category, a numeric code, a default message, and an HTTP status.
3. Add a static method `ErrorCode.fromCode(int code)` that performs O(1) reverse lookup using a pre-built `Map`.
4. Add a static method `List<ErrorCode> getByCategory(ErrorCategory category)` that returns all errors in a category.
5. Create an `ApiException` class that takes an `ErrorCode` and optional custom message.
6. Create a `GlobalExceptionHandler` that converts `ApiException` into a structured JSON error response.

Test by throwing and catching various exceptions and printing the structured error responses.

> **Hint:** Use a static initializer block to build the reverse lookup map. Use `EnumMap<ErrorCategory, List<ErrorCode>>` for the category grouping.

<details>
<summary>Solution for Exercise 1</summary>

```java
public enum Day {
    MONDAY("Monday", "Mon", false),
    TUESDAY("Tuesday", "Tue", false),
    WEDNESDAY("Wednesday", "Wed", false),
    THURSDAY("Thursday", "Thu", false),
    FRIDAY("Friday", "Fri", false),
    SATURDAY("Saturday", "Sat", true),
    SUNDAY("Sunday", "Sun", true);

    private final String displayName;
    private final String shortName;
    private final boolean weekend;

    Day(String displayName, String shortName, boolean weekend) {
        this.displayName = displayName;
        this.shortName = shortName;
        this.weekend = weekend;
    }

    public boolean isWeekday() { return !weekend; }

    public Day nextDay() {
        return values()[(this.ordinal() + 1) % 7];
    }

    public int daysUntil(Day target) {
        int diff = target.ordinal() - this.ordinal();
        return diff > 0 ? diff : diff + 7;
    }

    public static void main(String[] args) {
        for (Day d : values()) {
            System.out.printf("%-10s (%s) | Weekend: %-5s | Next: %-10s | Days to Friday: %d%n",
                d.displayName, d.shortName, d.weekend, d.nextDay(), d.daysUntil(FRIDAY));
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
import java.util.*;

enum TicketStatus {
    OPEN {
        @Override public Set<TicketStatus> allowedTransitions() {
            return EnumSet.of(IN_PROGRESS, CLOSED);
        }
    },
    IN_PROGRESS {
        @Override public Set<TicketStatus> allowedTransitions() {
            return EnumSet.of(WAITING_FOR_CUSTOMER, RESOLVED, CLOSED);
        }
    },
    WAITING_FOR_CUSTOMER {
        @Override public Set<TicketStatus> allowedTransitions() {
            return EnumSet.of(IN_PROGRESS, CLOSED);
        }
    },
    RESOLVED {
        @Override public Set<TicketStatus> allowedTransitions() {
            return EnumSet.of(CLOSED, REOPENED);
        }
    },
    CLOSED {
        @Override public Set<TicketStatus> allowedTransitions() {
            return EnumSet.of(REOPENED);
        }
    },
    REOPENED {
        @Override public Set<TicketStatus> allowedTransitions() {
            return EnumSet.of(IN_PROGRESS);
        }
    };

    public abstract Set<TicketStatus> allowedTransitions();
    public boolean canTransitionTo(TicketStatus target) {
        return allowedTransitions().contains(target);
    }
}

class Ticket {
    private final String id;
    private TicketStatus status;

    Ticket(String id) { this.id = id; this.status = TicketStatus.OPEN; }

    void transition(TicketStatus newStatus) {
        if (!status.canTransitionTo(newStatus)) {
            throw new IllegalStateException(
                "Cannot transition from " + status + " to " + newStatus);
        }
        System.out.println(id + ": " + status + " -> " + newStatus);
        this.status = newStatus;
    }
}

public class Main {
    public static void main(String[] args) {
        Ticket t = new Ticket("TKT-001");
        t.transition(TicketStatus.IN_PROGRESS);
        t.transition(TicketStatus.RESOLVED);
        t.transition(TicketStatus.CLOSED);
        try { t.transition(TicketStatus.IN_PROGRESS); }
        catch (IllegalStateException e) { System.out.println("Error: " + e.getMessage()); }
    }
}
```

</details>

---

## Related Notes

- [[Java - Date and Time API - LocalDate LocalDateTime]]
- [[Java - Control Flow - If Else Switch]]
- [[Java - Abstraction - Abstract Classes and Interfaces]]
- [[Java - Records - Java 14+]] (next note)

---

## Resources

- [Oracle Java Tutorials: Enum Types](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html) - Official documentation covering basic and advanced enum features.
- [Oracle Java Documentation: java.lang.Enum](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html) - Complete API reference for the Enum base class.
- [Oracle Java Documentation: EnumSet](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumSet.html) - API reference for the high-performance enum set.
- [Baeldung: Java Enums](https://www.baeldung.com/a-guide-to-java-enums) - Comprehensive guide covering all enum features with examples.
- [Baeldung: JPA Enum Mappings](https://www.baeldung.com/jpa-persisting-enums-in-jpa) - Detailed guide to `@Enumerated`, `EnumType.STRING`, `EnumType.ORDINAL`, and custom converters. Essential for backend development.
- [Effective Java by Joshua Bloch - Item 34: Use Enums Instead of int Constants](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive argument for using enums over integer or string constants.
- [Effective Java by Joshua Bloch - Item 35: Use Instance Fields Instead of Ordinals](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Why you should never use ordinal() for business logic.
