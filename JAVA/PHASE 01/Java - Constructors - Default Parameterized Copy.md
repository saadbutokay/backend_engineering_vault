---
title: "Java - Constructors - Default Parameterized Copy"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - constructors
status: "not-started"
---

# Java - Constructors - Default Parameterized Copy

> [!abstract] Overview
> A constructor is a special method that is called automatically when an object is created using the `new` keyword. Its purpose is to initialize the object's fields to valid, meaningful values before the object is used. Java provides three types of constructors: default (no arguments), parameterized (accepts arguments to set initial state), and copy (creates a new object as a duplicate of an existing one). In backend development, constructors are used everywhere: Spring creates controller and service objects through constructors, JPA uses constructors to hydrate entities from database rows, and DTOs use constructors to enforce that required fields are always present.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Methods - Parameters Return Types Overloading]]
> - [[Java - Variables and Data Types]]

---

## Theory

### What is a Constructor?

A constructor is a special block of code that looks like a method but has two critical differences:

1. **Its name must exactly match the class name**, including capitalization.
2. **It has no return type**, not even `void`. If you write a return type, it becomes a regular method, not a constructor.

```java
public class Student {
    String name;
    int age;

    // This is a constructor. No return type. Name matches the class.
    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // This is NOT a constructor. It has a return type (void).
    // This is a regular method that happens to have the same name as the class.
    void Student() {
        System.out.println("This is a method, not a constructor!");
    }
}
```

When you write `new Student("Saad", 22)`, the JVM:

1. Allocates memory on the heap for a new `Student` object.
2. Initializes all fields to default values (`null`, `0`, `false`).
3. Calls the matching constructor, passing `"Saad"` and `22` as arguments.
4. The constructor sets `this.name = "Saad"` and `this.age = 22`.
5. Returns the reference to the fully initialized object.

The constructor is the only opportunity to guarantee that an object starts its life in a valid state. Without constructors, every object would start with null and zero values, and you would have to remember to call setter methods after every `new` call. This is error-prone and violates the principle that an object should always be in a consistent state.

### The Default Constructor

A default constructor is a constructor that takes no arguments. Java has two kinds of default constructors:

**1. Compiler-generated default constructor**: If you do not write any constructor in your class, the Java compiler automatically generates a no-argument constructor that does nothing except call the parent class constructor (`super()`). This generated constructor is invisible in your source code but exists in the compiled bytecode.

```java
public class Product {
    String name;
    double price;
    // No constructor written. The compiler generates:
    // Product() { super(); }
}

// This works because the compiler-generated constructor exists.
Product p = new Product();
// p.name is null, p.price is 0.0
```

**2. Explicit default constructor**: A no-argument constructor that you write yourself. You do this when you want to set specific default values.

```java
public class Product {
    String name;
    double price;
    int stockQuantity;

    // Explicit default constructor with meaningful defaults
    Product() {
        this.name = "Unknown Product";
        this.price = 0.0;
        this.stockQuantity = 0;
    }
}
```

> [!warning] Critical Rule
> The compiler only generates a default constructor if you have NOT written any constructor at all. The moment you write even one constructor (parameterized or otherwise), the compiler stops generating the default constructor. If you still need a no-argument constructor, you must write it explicitly.

```java
public class Product {
    String name;
    double price;

    // You wrote a parameterized constructor.
    // The compiler will NOT generate a default constructor.
    Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
}

Product p1 = new Product("Laptop", 85000);  // Works
Product p2 = new Product();  // COMPILATION ERROR! No default constructor exists.
```

This rule causes more compilation errors for beginners than almost any other Java feature.

### The Parameterized Constructor

A parameterized constructor accepts arguments that are used to initialize the object's fields. This is the most commonly used type of constructor in backend development because it ensures that an object is created with all required data from the start.

```java
public class Order {
    String orderNumber;
    Long userId;
    BigDecimal totalAmount;
    OrderStatus status;
    LocalDateTime createdAt;

    // Parameterized constructor: forces the caller to provide all required data
    Order(String orderNumber, Long userId, BigDecimal totalAmount) {
        if (orderNumber == null || orderNumber.isBlank()) {
            throw new IllegalArgumentException("Order number cannot be empty");
        }
        if (userId == null || userId <= 0) {
            throw new IllegalArgumentException("Valid user ID is required");
        }
        if (totalAmount == null || totalAmount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Total amount must be positive");
        }

        this.orderNumber = orderNumber;
        this.userId = userId;
        this.totalAmount = totalAmount;
        this.status = OrderStatus.PENDING;  // Default status for new orders
        this.createdAt = LocalDateTime.now();  // Automatically set
    }
}
```

Notice that the constructor does more than just assign values. It **validates** the input. This is a powerful pattern: by putting validation in the constructor, you guarantee that no `Order` object can ever exist in an invalid state. Any code that receives an `Order` object can trust that the order number is non-empty, the user ID is positive, and the total amount is greater than zero.

### The Copy Constructor

A copy constructor creates a new object by copying the fields of an existing object of the same class. Java does not provide a built-in copy constructor (unlike C++), so you must write it yourself if you need one.

```java
public class Product {
    String name;
    double price;
    int stockQuantity;

    // Parameterized constructor
    Product(String name, double price, int stockQuantity) {
        this.name = name;
        this.price = price;
        this.stockQuantity = stockQuantity;
    }

    // Copy constructor: creates a new Product with the same field values
    Product(Product other) {
        this.name = other.name;
        this.price = other.price;
        this.stockQuantity = other.stockQuantity;
    }
}
```

**When to use a copy constructor:**

- When you need to create an independent copy of an object so that modifications to the copy do not affect the original.
- When working with immutable objects that need to be "modified" by creating a new version (like the Builder pattern).
- When passing objects to methods that might modify them and you want to protect the original.

**Shallow copy vs deep copy:**

The copy constructor above performs a **shallow copy**. For primitive fields (`double`, `int`), the actual values are copied. For reference fields (`String`, arrays, objects), only the reference is copied, meaning both the original and the copy point to the same object on the heap.

For classes with mutable reference fields (arrays, lists, custom objects), you need a **deep copy** that creates new copies of the referenced objects as well.

```java
public class ShoppingCart {
    String customerName;
    List<String> items;  // Mutable reference field

    // Shallow copy constructor (DANGEROUS for mutable references)
    ShoppingCart(ShoppingCart other) {
        this.customerName = other.customerName;
        this.items = other.items;  // Both carts share the SAME list!
    }

    // Deep copy constructor (SAFE)
    ShoppingCart deepCopy(ShoppingCart other) {
        ShoppingCart copy = new ShoppingCart();
        copy.customerName = other.customerName;
        copy.items = new ArrayList<>(other.items);  // New list with the same elements
        return copy;
    }
}
```

### Constructor Overloading

Just like methods, constructors can be overloaded. You can define multiple constructors with different parameter lists in the same class. The compiler selects the appropriate constructor based on the arguments you pass to `new`.

```java
public class User {
    String username;
    String email;
    String role;
    boolean isActive;

    // Constructor 1: Full details
    User(String username, String email, String role, boolean isActive) {
        this.username = username;
        this.email = email;
        this.role = role;
        this.isActive = isActive;
    }

    // Constructor 2: Default role and active status
    User(String username, String email) {
        this(username, email, "USER", true);  // Calls Constructor 1
    }

    // Constructor 3: Copy constructor
    User(User other) {
        this(other.username, other.email, other.role, other.isActive);  // Calls Constructor 1
    }
}
```

### Constructor Chaining with `this()`

Constructor chaining is the practice of one constructor calling another constructor in the same class using the `this()` keyword. This eliminates code duplication when multiple constructors share common initialization logic.

```java
public class Order {
    String orderNumber;
    Long userId;
    BigDecimal totalAmount;
    OrderStatus status;

    // The "master" constructor that does all the work
    Order(String orderNumber, Long userId, BigDecimal totalAmount, OrderStatus status) {
        this.orderNumber = orderNumber;
        this.userId = userId;
        this.totalAmount = totalAmount;
        this.status = status;
    }

    // Chains to the master constructor with a default status
    Order(String orderNumber, Long userId, BigDecimal totalAmount) {
        this(orderNumber, userId, totalAmount, OrderStatus.PENDING);
        // this() must be the FIRST statement in the constructor
    }

    // Chains to the three-argument constructor
    Order(String orderNumber, Long userId) {
        this(orderNumber, userId, BigDecimal.ZERO);
    }
}
```

**Rules for `this()`:**

1. `this()` must be the **first statement** in the constructor. You cannot put any code before it.
2. You cannot use `this()` and `super()` in the same constructor (both must be first, and there can only be one first statement).
3. Constructor chaining must not create a cycle. `A()` calling `B()` calling `A()` will cause a compilation error.

### How Constructors Work Internally

At the bytecode level, constructors are compiled into special methods named `<init>`. When you write `new Student("Saad", 22)`, the JVM executes the following bytecode sequence:

1. `new Student` -- allocates memory for the object on the heap and pushes an uninitialized reference onto the stack.
2. `dup` -- duplicates the reference on the stack (one copy for the constructor, one for the assignment).
3. `ldc "Saad"` -- pushes the string argument onto the stack.
4. `bipush 22` -- pushes the integer argument onto the stack.
5. `invokespecial Student.<init>(String, int)` -- calls the constructor, which pops the arguments and the reference, initializes the fields, and returns.
6. `astore_1` -- stores the initialized reference into the local variable.

The `<init>` method is not a regular method. It cannot be called directly, it cannot be overridden, and it has no return type in the bytecode (though the JVM implicitly returns the initialized reference to the `new` expression).

> [!tip] Key Insight
> Constructors are not inherited. If a parent class has a parameterized constructor, the child class does not automatically get it. The child class must define its own constructors and explicitly call the parent constructor using `super()`. You will learn this in detail in the Inheritance note. For now, remember: constructors belong to the class that defines them, and every constructor (except `Object`'s) begins with an implicit or explicit call to `super()`.

---

## Syntax and Basic Examples

### Example 1: All three constructor types in one class

```java
public class Student {
    String name;
    int age;
    String department;
    double cgpa;

    // 1. Default constructor (no arguments)
    Student() {
        this.name = "Unknown";
        this.age = 0;
        this.department = "Undeclared";
        this.cgpa = 0.0;
        System.out.println("Default constructor called");
    }

    // 2. Parameterized constructor (all fields)
    Student(String name, int age, String department, double cgpa) {
        this.name = name;
        this.age = age;
        this.department = department;
        this.cgpa = cgpa;
        System.out.println("Parameterized constructor called for " + name);
    }

    // 3. Parameterized constructor (partial fields, chaining to the full one)
    Student(String name, String department) {
        this(name, 0, department, 0.0);  // Chains to the 4-argument constructor
        System.out.println("Partial constructor called");
    }

    // 4. Copy constructor
    Student(Student other) {
        this.name = other.name;
        this.age = other.age;
        this.department = other.department;
        this.cgpa = other.cgpa;
        System.out.println("Copy constructor called for " + name);
    }

    void display() {
        System.out.printf("Name: %s | Age: %d | Dept: %s | CGPA: %.2f%n",
            name, age, department, cgpa);
    }

    public static void main(String[] args) {
        Student s1 = new Student();
        s1.display();

        Student s2 = new Student("Abdullah Al Sayb Saad", 22, "CSE", 3.72);
        s2.display();

        Student s3 = new Student("Rahim", "EEE");
        s3.display();

        Student s4 = new Student(s2);  // Copy of s2
        s4.display();

        // Prove that s4 is independent of s2
        s4.cgpa = 3.90;
        System.out.println("\nAfter modifying s4:");
        System.out.print("s2: "); s2.display();
        System.out.print("s4: "); s4.display();
    }
}
```

**Output:**
```
Default constructor called
Name: Unknown | Age: 0 | Dept: Undeclared | CGPA: 0.00
Parameterized constructor called for Abdullah Al Sayb Saad
Name: Abdullah Al Sayb Saad | Age: 22 | Dept: CSE | CGPA: 3.72
Parameterized constructor called for Rahim
Partial constructor called
Name: Rahim | Age: 0 | Dept: EEE | CGPA: 0.00
Copy constructor called for Abdullah Al Sayb Saad
Name: Abdullah Al Sayb Saad | Age: 22 | Dept: CSE | CGPA: 3.72

After modifying s4:
s2: Name: Abdullah Al Sayb Saad | Age: 22 | Dept: CSE | CGPA: 3.72
s4: Name: Abdullah Al Sayb Saad | Age: 22 | Dept: CSE | CGPA: 3.90
```

### Example 2: Constructor with validation

```java
public class BankAccount {
    String accountNumber;
    String holderName;
    double balance;

    BankAccount(String accountNumber, String holderName, double initialDeposit) {
        // Validate account number
        if (accountNumber == null || accountNumber.isBlank()) {
            throw new IllegalArgumentException("Account number cannot be empty");
        }
        if (accountNumber.length() < 10) {
            throw new IllegalArgumentException("Account number must be at least 10 characters");
        }

        // Validate holder name
        if (holderName == null || holderName.strip().length() < 2) {
            throw new IllegalArgumentException("Holder name must be at least 2 characters");
        }

        // Validate initial deposit
        if (initialDeposit < 500) {
            throw new IllegalArgumentException(
                "Minimum initial deposit is 500 BDT. Received: " + initialDeposit
            );
        }

        this.accountNumber = accountNumber;
        this.holderName = holderName.strip();
        this.balance = initialDeposit;
    }

    void display() {
        System.out.printf("Account: %s | Holder: %s | Balance: %,.2f BDT%n",
            accountNumber, holderName, balance);
    }

    public static void main(String[] args) {
        // Valid account
        BankAccount acc1 = new BankAccount("ACC-2025-001", "Abdullah Al Sayb Saad", 10000);
        acc1.display();

        // Invalid: deposit too low
        try {
            BankAccount acc2 = new BankAccount("ACC-2025-002", "Rahim", 100);
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }

        // Invalid: empty name
        try {
            BankAccount acc3 = new BankAccount("ACC-2025-003", "  ", 5000);
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Output:**
```
Account: ACC-2025-001 | Holder: Abdullah Al Sayb Saad | Balance: 10,000.00 BDT
Error: Minimum initial deposit is 500 BDT. Received: 100.0
Error: Holder name must be at least 2 characters
```

### Example 3: Constructor chaining in action

```java
public class HttpRequest {
    String method;
    String url;
    String body;
    String contentType;
    int timeoutMs;

    // Master constructor with all parameters
    HttpRequest(String method, String url, String body, String contentType, int timeoutMs) {
        this.method = method;
        this.url = url;
        this.body = body;
        this.contentType = contentType;
        this.timeoutMs = timeoutMs;
    }

    // GET request: no body, default content type and timeout
    HttpRequest(String url) {
        this("GET", url, null, "application/json", 5000);
    }

    // POST request: has body, default content type and timeout
    HttpRequest(String url, String body) {
        this("POST", url, body, "application/json", 5000);
    }

    // Custom method and URL with default timeout
    HttpRequest(String method, String url, String body) {
        this(method, url, body, "application/json", 5000);
    }

    void display() {
        System.out.printf("%s %s | Content-Type: %s | Timeout: %dms | Body: %s%n",
            method, url, contentType, timeoutMs,
            body != null ? body.substring(0, Math.min(body.length(), 30)) + "..." : "none");
    }

    public static void main(String[] args) {
        HttpRequest get = new HttpRequest("https://api.example.com/orders");
        get.display();

        HttpRequest post = new HttpRequest(
            "https://api.example.com/orders",
            "{\"userId\": 123, \"items\": [1, 2]}"
        );
        post.display();

        HttpRequest put = new HttpRequest(
            "PUT",
            "https://api.example.com/orders/1",
            "{\"status\": \"SHIPPED\"}"
        );
        put.display();
    }
}
```

**Output:**
```
GET https://api.example.com/orders | Content-Type: application/json | Timeout: 5000ms | Body: none
POST https://api.example.com/orders | Content-Type: application/json | Timeout: 5000ms | Body: {"userId": 123, "items": [1, 2]...
PUT https://api.example.com/orders/1 | Content-Type: application/json | Timeout: 5000ms | Body: {"status": "SHIPPED"}...
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Constructors are fundamental to every layer of a Spring Boot application. Here are three realistic scenarios.

### Scenario 1: Entity constructors with JPA

JPA entities require a no-argument constructor because the JPA framework uses reflection to create entity objects when reading from the database. The framework calls the no-arg constructor first, then sets the fields using reflection. However, you should also provide a parameterized constructor for creating new entities in your service layer.

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

    @Column(nullable = false, unique = true)
    private String orderNumber;

    @Column(nullable = false)
    private Long userId;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal totalAmount;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;

    @Column(nullable = false)
    private LocalDateTime createdAt;

    // REQUIRED by JPA. The framework calls this when loading from the database.
    // It must be public or protected. Do not remove it even if your IDE says it is unused.
    protected Order() {
    }

    // Used by your service layer when creating new orders.
    // Notice that 'id' and 'createdAt' are not parameters.
    // The database generates the ID, and the constructor sets the timestamp.
    public Order(String orderNumber, Long userId, BigDecimal totalAmount) {
        if (orderNumber == null || orderNumber.isBlank()) {
            throw new IllegalArgumentException("Order number is required");
        }
        if (userId == null || userId <= 0) {
            throw new IllegalArgumentException("Valid user ID is required");
        }
        if (totalAmount == null || totalAmount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Total amount must be positive");
        }

        this.orderNumber = orderNumber;
        this.userId = userId;
        this.totalAmount = totalAmount;
        this.status = OrderStatus.PENDING;
        this.createdAt = LocalDateTime.now();
    }

    // Getters (no setters for fields that should not change after creation)
    public Long getId() { return id; }
    public String getOrderNumber() { return orderNumber; }
    public Long getUserId() { return userId; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public OrderStatus getStatus() { return status; }
    public LocalDateTime getCreatedAt() { return createdAt; }

    // Only the status can change after creation, through a controlled method
    public void updateStatus(OrderStatus newStatus) {
        this.status = newStatus;
    }
}
```

**What to notice:**

- The `protected Order()` no-arg constructor exists solely for JPA. Your application code should never call it. Making it `protected` (instead of `public`) prevents accidental use while still allowing JPA's reflection-based instantiation.
- The parameterized constructor enforces business rules: order number cannot be empty, user ID must be positive, total must be greater than zero. These rules are enforced at object creation time, making it impossible to create an invalid `Order` object.
- The `id` field is not in the constructor because the database generates it when the entity is saved. The `createdAt` field is set automatically inside the constructor. This is a common pattern: the constructor only accepts the data that the caller must provide.

### Scenario 2: DTO constructors with Java Records

Java 14 introduced **records**, which are a compact syntax for immutable data classes. Records automatically generate a constructor, getters, `equals()`, `hashCode()`, and `toString()`. DTOs are the perfect use case for records.

```java
package com.company.orderservice.dto;

import java.math.BigDecimal;
import java.time.LocalDateTime;

// A record automatically generates:
// 1. A constructor: CreateOrderRequest(Long userId, List<OrderItemDTO> items, String couponCode)
// 2. Getters: userId(), items(), couponCode() (note: no "get" prefix)
// 3. equals(), hashCode(), toString()
public record CreateOrderRequest(
    Long userId,
    List<OrderItemDTO> items,
    String couponCode
) {
    // Compact constructor for validation.
    // This runs BEFORE the fields are assigned.
    // If validation fails, the object is never created.
    public CreateOrderRequest {
        if (userId == null || userId <= 0) {
            throw new IllegalArgumentException("Valid user ID is required");
        }
        if (items == null || items.isEmpty()) {
            throw new IllegalArgumentException("Order must contain at least one item");
        }
        if (items.size() > 100) {
            throw new IllegalArgumentException("Maximum 100 items per order");
        }
        // couponCode can be null (optional field), so no validation needed
    }
}

// Nested record for order items
record OrderItemDTO(
    Long productId,
    int quantity,
    BigDecimal unitPrice
) {
    public OrderItemDTO {
        if (productId == null || productId <= 0) {
            throw new IllegalArgumentException("Valid product ID is required");
        }
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        if (unitPrice == null || unitPrice.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Unit price must be positive");
        }
    }
}
```

**What to notice:**

- The record's compact constructor (`public CreateOrderRequest { ... }`) does not have parentheses or parameters. It validates the arguments before the record assigns them to fields. This is the cleanest way to enforce validation on DTOs.
- Records are **immutable**. Once created, the fields cannot be changed. This makes them thread-safe and ideal for DTOs, which should be read-only data carriers.
- When Spring receives a JSON request body, it uses the record's constructor to create the object. If the JSON contains invalid data (e.g., negative quantity), the constructor throws an exception, and Spring automatically returns a 400 Bad Request response.

### Scenario 3: Dependency Injection through constructors

In Spring Boot, services and controllers receive their dependencies through constructor injection. Spring calls the constructor and passes the required objects as arguments.

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final EmailService emailService;

    // Spring calls this constructor when creating the OrderService bean.
    // It automatically finds the OrderRepository, PaymentService, and EmailService
    // beans in the application context and passes them as arguments.
    // You never call this constructor yourself.
    public OrderService(OrderRepository orderRepository,
                        PaymentService paymentService,
                        EmailService emailService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.emailService = emailService;
    }

    public Order placeOrder(Long userId, List<OrderItem> items) {
        Order order = new Order(generateOrderNumber(), userId, calculateTotal(items));
        Order savedOrder = orderRepository.save(order);
        paymentService.processPayment(savedOrder);
        emailService.sendOrderConfirmation(savedOrder);
        return savedOrder;
    }

    // ... other methods ...
}
```

**What to notice:**

- The constructor parameters are the service's **dependencies**. The `OrderService` depends on a repository, a payment service, and an email service. By declaring them in the constructor, you make these dependencies explicit and mandatory.
- The fields are declared `final`. This means they can only be set once (in the constructor) and cannot be changed afterward. This is a best practice for dependency injection because it guarantees that the dependencies are always present and never accidentally reassigned.
- Constructor injection is preferred over field injection (`@Autowired` on fields) because it makes the class easier to test. In a unit test, you can create an `OrderService` by passing mock objects to the constructor without needing the Spring framework.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Losing the default constructor by adding a parameterized one

**Wrong:**
```java
public class User {
    String name;
    String email;

    User(String name, String email) {
        this.name = name;
        this.email = email;
    }
}

User u = new User();  // COMPILATION ERROR! No default constructor.
// The compiler stopped generating one because you wrote a parameterized constructor.
```

**Right:**
```java
public class User {
    String name;
    String email;

    // Explicitly provide the default constructor if you need it
    User() {
        this.name = "Unknown";
        this.email = "";
    }

    User(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

**Why it is wrong:** This is the most common constructor-related compilation error. The compiler's rule is simple: if you write zero constructors, it generates a default one. If you write one or more constructors, it generates nothing. Always think about whether you need a no-argument constructor when you add a parameterized one. JPA entities, in particular, always require a no-argument constructor.

### Mistake 2: Adding a return type to a constructor

**Wrong:**
```java
public class Product {
    String name;

    // This looks like a constructor but is actually a regular method
    // because it has a return type (void).
    void Product(String name) {
        this.name = name;
    }
}

Product p = new Product("Laptop");  // COMPILATION ERROR!
// The compiler sees no constructor that takes a String argument.
// The void Product(String) method is not a constructor.
```

**Right:**
```java
public class Product {
    String name;

    // No return type. This is a proper constructor.
    Product(String name) {
        this.name = name;
    }
}
```

**Why it is wrong:** A constructor must not have a return type. Adding `void` (or any other type) turns it into a regular method with the same name as the class, which is legal Java but almost certainly a mistake. The compiler will not warn you about this because it is technically valid syntax.

### Mistake 3: Not calling `this()` as the first statement

**Wrong:**
```java
public class Order {
    String orderNumber;
    BigDecimal total;

    Order(String orderNumber, BigDecimal total) {
        this.orderNumber = orderNumber;
        this.total = total;
    }

    Order(String orderNumber) {
        System.out.println("Creating order");  // Code before this()
        this(orderNumber, BigDecimal.ZERO);    // COMPILATION ERROR!
        // this() must be the FIRST statement.
    }
}
```

**Right:**
```java
public class Order {
    String orderNumber;
    BigDecimal total;

    Order(String orderNumber, BigDecimal total) {
        this.orderNumber = orderNumber;
        this.total = total;
    }

    Order(String orderNumber) {
        this(orderNumber, BigDecimal.ZERO);    // Must be first
        System.out.println("Creating order");  // Code after this() is fine
    }
}
```

**Why it is wrong:** Java requires that `this()` (or `super()`) be the very first statement in a constructor. This ensures that the object is fully initialized by the chained constructor before any additional code runs. If you need to perform logic before the chained call, extract it into a static helper method.

### Mistake 4: Shallow copy when deep copy is needed

**Wrong:**
```java
public class Team {
    String teamName;
    String[] members;

    Team(String teamName, String[] members) {
        this.teamName = teamName;
        this.members = members;  // Shallow copy: both teams share the same array!
    }

    Team(Team other) {
        this.teamName = other.teamName;
        this.members = other.members;  // BUG: shallow copy of the array
    }
}

Team original = new Team("Backend", new String[]{"Alice", "Bob"});
Team copy = new Team(original);
copy.members[0] = "Charlie";
System.out.println(original.members[0]);  // "Charlie"! The original is modified!
```

**Right:**
```java
Team(Team other) {
    this.teamName = other.teamName;
    this.members = Arrays.copyOf(other.members, other.members.length);  // Deep copy
}
```

**Why it is wrong:** Arrays and objects are reference types. Assigning `this.members = other.members` copies the reference, not the array contents. Both the original and the copy point to the same array on the heap. Modifying the array through one reference affects the other. Use `Arrays.copyOf()` for arrays and create new instances for mutable objects.

---

## Key Takeaways

> [!tip] Remember these points
> 1. A constructor is a special method with the same name as the class and no return type. It is called automatically when you use `new` to create an object. Its purpose is to initialize the object to a valid state.
> 2. The **default constructor** takes no arguments. The compiler generates one automatically only if you have not written any constructor. The moment you write a parameterized constructor, you must explicitly write a default constructor if you still need one.
> 3. **Parameterized constructors** accept arguments to set initial field values. Use them to enforce validation rules at creation time, making it impossible to create objects in an invalid state.
> 4. **Copy constructors** create a new object by copying an existing one. Be careful with mutable reference fields (arrays, lists, objects): a shallow copy shares the same references, while a deep copy creates independent duplicates.
> 5. **Constructor chaining** with `this()` eliminates code duplication by having simpler constructors delegate to more comprehensive ones. `this()` must always be the first statement in the constructor.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Employee Class with Constructors (Easy)
Create an `Employee` class with fields: `employeeId` (String), `name` (String), `department` (String), `salary` (double). Write four constructors:
1. Default constructor that sets all fields to reasonable defaults.
2. Parameterized constructor that accepts all four fields with validation (salary must be positive, name must not be empty).
3. Parameterized constructor that accepts `name` and `department` only, chaining to the full constructor with a generated ID and a default salary of 30000.
4. Copy constructor that creates a deep copy.

Test all four constructors in `main()` and verify that the copy is independent of the original.

**Hint:** For the generated ID in constructor 3, use `"EMP-" + System.currentTimeMillis()`.

### Exercise 2: Immutable Product with Validation (Medium)
Create an `ImmutableProduct` class where all fields are `final` and can only be set through the constructor. Fields: `id` (long), `name` (String), `price` (double), `category` (String). The constructor should:
- Throw `IllegalArgumentException` if `name` is null or blank.
- Throw `IllegalArgumentException` if `price` is negative.
- Automatically convert `category` to uppercase.
- Automatically assign the `id` using a static counter that increments with each new object.

Since all fields are `final`, there should be no setter methods. The object's state cannot change after creation.

**Hint:** Use a `private static long nextId = 1;` class variable. In the constructor, assign `this.id = nextId++;`.

### Exercise 3: Constructor Chaining for HTTP Response (Medium)
Create an `HttpResponse` class with fields: `statusCode` (int), `statusMessage` (String), `body` (String), `contentType` (String). Create a chain of constructors:
1. Full constructor: all four fields.
2. `HttpResponse(int statusCode, String body)`: chains to the full constructor with auto-detected status message ("OK" for 200, "Not Found" for 404, "Internal Server Error" for 500, etc.) and default content type "application/json".
3. `HttpResponse(int statusCode)`: chains to constructor 2 with an empty body.
4. `HttpResponse()`: chains to constructor 3 with status code 200.

Test all constructors and print the results.

**Hint:** Use a private static helper method `getStatusMessage(int code)` to map status codes to messages. Call it from the constructor chain.

### Exercise 4: Deep Copy of a Complex Object (Hard, Optional)
Create a `Course` class with fields: `courseCode` (String), `courseName` (String), `enrolledStudents` (an array of `Student` objects from the previous note). Write:
1. A parameterized constructor.
2. A copy constructor that performs a **deep copy**: the new `Course` object should have its own independent array of `Student` objects, not shared references with the original.

Prove the deep copy works by modifying a student's CGPA in the copied course and verifying that the original course's student is unaffected.

**Hint:** In the copy constructor, create a new array and use a loop to create new `Student` objects (using the `Student` copy constructor) for each element.

<details>
<summary>Solution for Exercise 2</summary>

```java
public class ImmutableProduct {
    private static long nextId = 1;

    private final long id;
    private final String name;
    private final double price;
    private final String category;

    public ImmutableProduct(String name, double price, String category) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Product name cannot be empty");
        }
        if (price < 0) {
            throw new IllegalArgumentException("Price cannot be negative: " + price);
        }

        this.id = nextId++;
        this.name = name.strip();
        this.price = price;
        this.category = category != null ? category.toUpperCase() : "UNCATEGORIZED";
    }

    public long getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
    public String getCategory() { return category; }

    @Override
    public String toString() {
        return String.format("Product{id=%d, name='%s', price=%.2f, category='%s'}",
            id, name, price, category);
    }

    public static void main(String[] args) {
        ImmutableProduct p1 = new ImmutableProduct("Laptop", 85000, "electronics");
        ImmutableProduct p2 = new ImmutableProduct("Mouse", 1500, "accessories");
        ImmutableProduct p3 = new ImmutableProduct("Clean Code Book", 1200, "books");

        System.out.println(p1);
        System.out.println(p2);
        System.out.println(p3);
        // IDs are auto-assigned: 1, 2, 3
        // Categories are auto-uppercased: ELECTRONICS, ACCESSORIES, BOOKS
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
public class HttpResponse {
    int statusCode;
    String statusMessage;
    String body;
    String contentType;

    HttpResponse(int statusCode, String statusMessage, String body, String contentType) {
        this.statusCode = statusCode;
        this.statusMessage = statusMessage;
        this.body = body;
        this.contentType = contentType;
    }

    HttpResponse(int statusCode, String body) {
        this(statusCode, getStatusMessage(statusCode), body, "application/json");
    }

    HttpResponse(int statusCode) {
        this(statusCode, "");
    }

    HttpResponse() {
        this(200);
    }

    private static String getStatusMessage(int code) {
        return switch (code) {
            case 200 -> "OK";
            case 201 -> "Created";
            case 400 -> "Bad Request";
            case 401 -> "Unauthorized";
            case 403 -> "Forbidden";
            case 404 -> "Not Found";
            case 500 -> "Internal Server Error";
            case 503 -> "Service Unavailable";
            default -> "Unknown";
        };
    }

    void display() {
        System.out.printf("HTTP %d %s | Content-Type: %s | Body: %s%n",
            statusCode, statusMessage, contentType,
            body.isEmpty() ? "(empty)" : body);
    }

    public static void main(String[] args) {
        new HttpResponse().display();
        new HttpResponse(404).display();
        new HttpResponse(201, "{\"id\": 1}").display();
        new HttpResponse(500, "Server Error", "Database connection failed", "text/plain").display();
    }
}
```

</details>

---

## Related Notes

- [[Java - Classes and Objects]]
- [[Java - Encapsulation - Getters Setters Access Modifiers]] (next note)
- [[Java - Inheritance - Single Multilevel Hierarchical]]

---

## Resources

- [Oracle Java Tutorials: Providing Constructors for Your Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/constructors.html) - Official documentation covering default and parameterized constructors.
- [Baeldung: Java Constructors](https://www.baeldung.com/java-constructors) - Comprehensive guide with examples of all constructor types.
- [Baeldung: Java Copy Constructor](https://www.baeldung.com/java-copy-constructor) - Detailed explanation of shallow vs deep copy constructors.
- [Baeldung: Java Records](https://www.baeldung.com/java-record-keyword) - Guide to Java records and their compact constructors. Read this when you start using DTOs in Phase 4.
- [Effective Java by Joshua Bloch - Item 1: Consider Static Factory Methods Instead of Constructors](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Advanced reading on when to use constructors vs factory methods. Highly recommended for Phase 2 and beyond.
