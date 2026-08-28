---
title: "Java - Classes and Objects"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - classes
  - objects
status: "not-started"
---

# Java - Classes and Objects

> [!abstract] Overview
> A class is a blueprint that defines the structure and behavior of a type of object. An object is a concrete instance created from that blueprint. Together, classes and objects form the foundation of Object-Oriented Programming (OOP), the paradigm that dominates Java and all enterprise backend development. Every Spring Boot controller, service, repository, entity, DTO, and exception is a class. Every HTTP request your backend processes is handled by objects instantiated from those classes. Understanding classes and objects deeply is the single most important step in your transition from writing scripts to engineering systems.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]
> - [[Java - Methods - Parameters Return Types Overloading]]
> - [[Java - Arrays - 1D and 2D]]
> - [[Java - Strings and String Methods]]
> - All of Phase 0

---

## Theory

### What is a Class?

A class is a user-defined data type that bundles **data** (called fields or attributes) and **behavior** (called methods) into a single unit. Think of a class as an architectural blueprint for a house. The blueprint defines how many rooms the house has, what color the walls are, and where the doors are located. But the blueprint itself is not a house. You cannot live in a blueprint. You need to **build** a house from the blueprint to have something real.

In Java, the class is the blueprint. The object is the house.

```java
// This is a class (blueprint). It defines what a Student looks like.
// No actual student exists yet.
public class Student {
    // Fields (data): what a student HAS
    String name;
    int age;
    String department;
    double cgpa;

    // Methods (behavior): what a student DOES
    void study() {
        System.out.println(name + " is studying " + department);
    }

    boolean isEligibleForScholarship() {
        return cgpa >= 3.5;
    }
}
```

### What is an Object?

An object is a concrete instance of a class. When you create an object, the JVM allocates memory on the heap for all the fields defined in the class and returns a reference to that memory location.

```java
// Creating objects (building houses from the blueprint)
Student saad = new Student();
saad.name = "Abdullah Al Sayb Saad";
saad.age = 22;
saad.department = "CSE";
saad.cgpa = 3.72;

Student rahim = new Student();
rahim.name = "Rahim Uddin";
rahim.age = 21;
rahim.department = "EEE";
rahim.cgpa = 3.45;
```

Each object has its own independent copy of the fields. Changing `saad.cgpa` does not affect `rahim.cgpa`. They are separate objects living at separate locations on the heap.

**Memory layout:**

```text
Stack                          Heap
+-----------+                 +-----------------------------+
| saad      |---------------> | Student@0x1A2B              |
| (ref)     |                 | name: "Abdullah..."         |
+-----------+                 | age: 22                     |
                              | department: "CSE"           |
+-----------+                 | cgpa: 3.72                  |
| rahim     |---------------> +-----------------------------+
| (ref)     |                 
+-----------+                 +-----------------------------+
                              | Student@0x3C4D              |
                              | name: "Rahim Uddin"         |
                              | age: 21                     |
                              | department: "EEE"           |
                              | cgpa: 3.45                  |
                              +-----------------------------+
```

### Why Does OOP Exist?

Before OOP, programs were written in a procedural style: a collection of functions that operate on separate data structures. This works for small programs but breaks down for large systems. Consider a backend e-commerce system written procedurally:

```java
// Procedural style (how you would write it in C)
String[] userNames = {"Alice", "Bob"};
String[] userEmails = {"alice@test.com", "bob@test.com"};
double[] userBalances = {1000.0, 500.0};

void updateUserEmail(int userIndex, String newEmail) {
    userEmails[userIndex] = newEmail;
    // What if userIndex is out of bounds?
    // What if the arrays get out of sync?
    // What if you add a new field and forget to update all functions?
}
```

This approach has three fatal problems at scale:

1. **Data and behavior are separated.** The user data lives in arrays, and the user logic lives in functions. There is no guarantee that the functions will operate on the correct data. In a system with 500 functions and 50 data arrays, keeping everything synchronized is impossible.

2. **No encapsulation.** Any function can modify any array directly. There is no way to enforce rules like "a user's balance cannot go negative" because any piece of code can write to the `userBalances` array.

3. **No abstraction.** To add a new feature (e.g., user roles), you must modify every function that touches user data. In a large codebase, this means changing hundreds of files and introducing dozens of new bugs.

OOP solves all three problems by bundling data and behavior together inside classes, restricting access to internal data through encapsulation, and hiding implementation details through abstraction. This is why every enterprise backend framework (Spring Boot, Jakarta EE, Quarkus, Micronaut) is built entirely on OOP principles.

### The `new` Keyword and Object Creation

When you write `new Student()`, the JVM performs the following steps:

1. **Allocates memory** on the heap for a new Student object. The size is determined by the number and types of fields in the class.
2. **Initializes all fields** to their default values (null for references, 0 for numbers, false for booleans).
3. **Calls the constructor** (a special method that initializes the object). If you did not write a constructor, Java provides a default one that does nothing.
4. **Returns a reference** to the newly created object. This reference is stored in the variable on the left side of the assignment.

### The `this` Keyword

Inside a method, `this` refers to the current object — the object on which the method was called. It is used to distinguish between instance fields and local variables (or parameters) that have the same name.

```java
public class Student {
    String name;

    void setName(String name) {
        // 'name' refers to the parameter (local variable)
        // 'this.name' refers to the instance field
        this.name = name;
    }
}
```

Without `this`, the assignment `name = name` would assign the parameter to itself, leaving the instance field unchanged. The `this` keyword resolves the ambiguity.

### Instance Variables vs Local Variables vs Class Variables

| Type | Where declared | Where stored | Lifetime | Default value |
|------|----------------|--------------|----------|---------------|
| Instance variable (field) | Inside a class, outside methods | Heap (inside the object) | As long as the object exists | Yes (0, null, false) |
| Local variable | Inside a method | Stack | Until the method returns | No (must be initialized) |
| Class variable (static) | Inside a class with `static` keyword | Heap (in the class metadata area) | As long as the class is loaded | Yes (0, null, false) |

```java
public class Student {
    static int totalStudents = 0;  // Class variable: shared by ALL students
    String name;                    // Instance variable: unique to EACH student

    void enroll() {
        int semester = 6;           // Local variable: exists only during this method call
        totalStudents++;            // Incrementing the shared class variable
        this.name = "New Student";  // Setting the instance variable
    }
}
```

### Null References

A reference variable that does not point to any object holds the value `null`. Attempting to access a field or call a method on a null reference throws a `NullPointerException`, the most common runtime error in Java backend systems.

```java
Student ghost = null;
System.out.println(ghost.name);  // NullPointerException!
ghost.study();                   // NullPointerException!
```

In backend development, null references come from database queries that find no matching record, JSON fields that are missing from a request body, and optional configuration values that were not set. You will learn strategies for handling null safely in later notes (Optional class, null checks, defensive programming).

> [!tip] Key Insight
> A class is a type, and an object is a value of that type. Just as `int` is a type and `42` is a value of that type, `Student` is a type and `new Student()` is a value of that type. This mental model helps you understand why you can use classes everywhere you use primitive types: as method parameters, return types, array elements, and generic type arguments.

---

## Syntax and Basic Examples

### Example 1: Defining a class and creating objects

```java
public class Product {
    // Instance fields (each product has its own copy)
    String name;
    double price;
    int stockQuantity;
    String category;

    // A method that calculates the total value of the stock
    double calculateStockValue() {
        return price * stockQuantity;
    }

    // A method that checks if the product is available for purchase
    boolean isInStock() {
        return stockQuantity > 0;
    }

    // A method that reduces stock after a purchase
    void purchase(int quantity) {
        if (quantity <= 0) {
            System.out.println("Invalid quantity");
            return;
        }
        if (quantity > stockQuantity) {
            System.out.println("Not enough stock. Available: " + stockQuantity);
            return;
        }
        stockQuantity -= quantity;
        System.out.println("Purchased " + quantity + " units of " + name
            + ". Remaining stock: " + stockQuantity);
    }
}
```

Using the class:

```java
public class Main {
    public static void main(String[] args) {
        // Create the first product
        Product laptop = new Product();
        laptop.name = "ThinkPad T14";
        laptop.price = 85000.0;
        laptop.stockQuantity = 15;
        laptop.category = "Electronics";

        // Create the second product
        Product mouse = new Product();
        mouse.name = "Logitech MX Master";
        mouse.price = 7500.0;
        mouse.stockQuantity = 50;
        mouse.category = "Accessories";

        // Use the objects
        System.out.println(laptop.name + " in stock: " + laptop.isInStock());
        System.out.println("Stock value: " + laptop.calculateStockValue() + " BDT");

        laptop.purchase(3);
        laptop.purchase(20);  // Not enough stock

        System.out.println(mouse.name + " stock value: " + mouse.calculateStockValue() + " BDT");
    }
}
```

**Output:**

```text
ThinkPad T14 in stock: true
Stock value: 1275000.0 BDT
Purchased 3 units of ThinkPad T14. Remaining stock: 12
Not enough stock. Available: 12
Logitech MX Master stock value: 375000.0 BDT
```

### Example 2: The `this` keyword and field shadowing

```java
public class BankAccount {
    String accountNumber;
    String holderName;
    double balance;

    // Method with parameters that have the same names as fields
    void initialize(String accountNumber, String holderName, double balance) {
        this.accountNumber = accountNumber;  // this.field = parameter
        this.holderName = holderName;
        this.balance = balance;
    }

    void deposit(double amount) {
        if (amount <= 0) {
            System.out.println("Deposit amount must be positive");
            return;
        }
        this.balance += amount;
        System.out.println("Deposited " + amount + " BDT. New balance: " + this.balance);
    }

    void withdraw(double amount) {
        if (amount > this.balance) {
            System.out.println("Insufficient funds. Balance: " + this.balance);
            return;
        }
        this.balance -= amount;
        System.out.println("Withdrew " + amount + " BDT. New balance: " + this.balance);
    }

    void displayInfo() {
        System.out.println("Account: " + this.accountNumber);
        System.out.println("Holder: " + this.holderName);
        System.out.printf("Balance: %,.2f BDT%n", this.balance);
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        account.initialize("ACC-2025-001", "Abdullah Al Sayb Saad", 50000.0);
        account.displayInfo();
        account.deposit(15000);
        account.withdraw(70000);  // Insufficient
        account.withdraw(20000);
    }
}
```

**Output:**

```text
Account: ACC-2025-001
Holder: Abdullah Al Sayb Saad
Balance: 50,000.00 BDT
Deposited 15000.0 BDT. New balance: 65000.0
Insufficient funds. Balance: 65000.0
Withdrew 20000.0 BDT. New balance: 45000.0
```

### Example 3: Arrays of objects

```java
public class Main {
    public static void main(String[] args) {
        // Create an array of Product objects
        Product[] inventory = new Product[3];

        inventory[0] = new Product();
        inventory[0].name = "Keyboard";
        inventory[0].price = 3200;
        inventory[0].stockQuantity = 30;

        inventory[1] = new Product();
        inventory[1].name = "Monitor";
        inventory[1].price = 25000;
        inventory[1].stockQuantity = 8;

        inventory[2] = new Product();
        inventory[2].name = "Webcam";
        inventory[2].price = 4500;
        inventory[2].stockQuantity = 0;

        // Process all products using a for-each loop
        double totalInventoryValue = 0;
        for (Product p : inventory) {
            System.out.printf("%-12s | %8.2f BDT | Stock: %3d | Value: %10.2f BDT | %s%n",
                p.name, p.price, p.stockQuantity,
                p.calculateStockValue(),
                p.isInStock() ? "Available" : "Out of Stock");
            totalInventoryValue += p.calculateStockValue();
        }

        System.out.printf("%nTotal Inventory Value: %,.2f BDT%n", totalInventoryValue);
    }
}
```

**Output:**

```text
Keyboard     |  3200.00 BDT | Stock:  30 | Value:   96000.00 BDT | Available
Monitor      | 25000.00 BDT | Stock:   8 | Value:  200000.00 BDT | Available
Webcam       |  4500.00 BDT | Stock:   0 | Value:       0.00 BDT | Out of Stock

Total Inventory Value: 296,000.00 BDT
```

### Example 4: Objects as method parameters and return types

```java
public class OrderProcessor {

    // Method that takes an object as a parameter
    static double calculateOrderTotal(Product[] items, int[] quantities) {
        double total = 0;
        for (int i = 0; i < items.length; i++) {
            total += items[i].price * quantities[i];
        }
        return total;
    }

    // Method that returns an object
    static Product findMostExpensive(Product[] inventory) {
        if (inventory == null || inventory.length == 0) {
            return null;  // Return null when no product exists
        }

        Product mostExpensive = inventory[0];
        for (Product p : inventory) {
            if (p.price > mostExpensive.price) {
                mostExpensive = p;  // Assign the reference, not a copy
            }
        }
        return mostExpensive;
    }

    // Method that modifies an object passed as a parameter
    // Remember: Java passes the reference by value, so the method
    // can modify the object's fields but cannot reassign the caller's variable.
    static void applyDiscount(Product product, double discountPercent) {
        double discount = product.price * (discountPercent / 100);
        product.price -= discount;  // This modifies the ORIGINAL object
    }

    public static void main(String[] args) {
        Product laptop = new Product();
        laptop.name = "Laptop";
        laptop.price = 80000;
        laptop.stockQuantity = 10;

        Product phone = new Product();
        phone.name = "Phone";
        phone.price = 45000;
        phone.stockQuantity = 25;

        Product[] inventory = {laptop, phone};
        int[] quantities = {2, 3};

        double total = calculateOrderTotal(inventory, quantities);
        System.out.printf("Order total: %,.2f BDT%n", total);

        Product expensive = findMostExpensive(inventory);
        System.out.println("Most expensive: " + expensive.name);

        System.out.println("Phone price before discount: " + phone.price);
        applyDiscount(phone, 10);
        System.out.println("Phone price after discount: " + phone.price);
        // The original phone object is modified because the method
        // received a copy of the reference, which points to the same object.
    }
}
```

**Output:**

```text
Order total: 295,000.00 BDT
Most expensive: Laptop
Phone price before discount: 45000.0
Phone price after discount: 40500.0
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Every component in a Spring Boot backend is a class. Here are three realistic examples showing how classes and objects form the architecture of a real system.

### Scenario 1: JPA Entity class (the data layer)

In Spring Boot, a JPA entity class maps directly to a database table. Each field maps to a column. Each object instance represents a row in the database.

```java
package com.company.orderservice.model;

import jakarta.persistence.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

// The @Entity annotation tells Spring that this class maps to a database table.
// The @Table annotation specifies the table name.
@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // This field maps to the 'id' column, which is the primary key.
    // @GeneratedValue means the database auto-generates the ID.

    @Column(nullable = false, unique = true, length = 20)
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

    @Column
    private LocalDateTime updatedAt;

    // Business logic method inside the entity
    public boolean canBeCancelled() {
        return this.status == OrderStatus.PENDING
            || this.status == OrderStatus.CONFIRMED;
    }

    // State transition method
    public void cancel() {
        if (!canBeCancelled()) {
            throw new IllegalStateException(
                "Cannot cancel order in status: " + this.status
            );
        }
        this.status = OrderStatus.CANCELLED;
        this.updatedAt = LocalDateTime.now();
    }

    // Getters and setters (required by JPA)
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getOrderNumber() { return orderNumber; }
    public void setOrderNumber(String orderNumber) { this.orderNumber = orderNumber; }
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public void setTotalAmount(BigDecimal totalAmount) { this.totalAmount = totalAmount; }
    public OrderStatus getStatus() { return status; }
    public void setStatus(OrderStatus status) { this.status = status; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

**What to notice:**

- This class is the bridge between Java objects and database rows. When you call `orderRepository.findById(1L)`, Spring creates an `Order` object and populates its fields with data from the database row where `id = 1`.
- The fields are `private`. External code accesses them through getter and setter methods. This is encapsulation, which you will study in detail in a later note.
- The `cancel()` method contains business logic inside the entity. This is a debated practice. Some teams prefer "anemic entities" (data only, no logic) and put all logic in service classes. Others prefer "rich entities" (data + logic). Both approaches are valid; the choice depends on the team's conventions.
- Every field has a corresponding database column. The `@Column` annotations control constraints like `nullable`, `unique`, `length`, `precision`, and `scale`.

### Scenario 2: Service class (the business logic layer)

The service class contains the business logic of your application. It creates, reads, updates, and deletes objects by coordinating between the controller (input) and the repository (database).

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;
import java.util.UUID;

// The @Service annotation tells Spring to create a single instance (object)
// of this class at startup and manage its lifecycle. This is called a "bean."
@Service
public class OrderService {

    // Spring injects the repository object automatically.
    // You do not call "new OrderRepository()". Spring creates the object
    // and assigns it to this field. This is called Dependency Injection.
    private final OrderRepository orderRepository;

    // Constructor: Spring calls this when creating the OrderService object
    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public Order createOrder(Long userId, List<OrderItem> items) {
        // Create a new Order object
        Order order = new Order();
        order.setOrderNumber(generateOrderNumber());
        order.setUserId(userId);
        order.setTotalAmount(calculateTotal(items));
        order.setStatus(OrderStatus.PENDING);
        order.setCreatedAt(LocalDateTime.now());

        // Save the object to the database.
        // The repository converts the Java object into an INSERT SQL statement.
        Order savedOrder = orderRepository.save(order);

        return savedOrder;
    }

    public Order cancelOrder(Long orderId) {
        // Retrieve the Order object from the database
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Order not found: " + orderId
            ));

        // Call the business logic method on the Order object
        order.cancel();  // This checks if cancellation is allowed and updates the status

        // Save the modified object back to the database
        return orderRepository.save(order);
    }

    private String generateOrderNumber() {
        return "ORD-" + UUID.randomUUID().toString().substring(0, 8).toUpperCase();
    }

    private BigDecimal calculateTotal(List<OrderItem> items) {
        BigDecimal total = BigDecimal.ZERO;
        for (OrderItem item : items) {
            total = total.add(item.getUnitPrice()
                .multiply(new BigDecimal(item.getQuantity())));
        }
        return total;
    }
}
```

**What to notice:**

- The `@Service` annotation makes this class a Spring-managed bean. Spring creates exactly one instance of this class when the application starts. Every HTTP request that needs order processing uses the same `OrderService` object. This is why service classes must be thread-safe (no mutable instance fields that change per request).
- The `OrderRepository` is injected through the constructor. You never write `new OrderRepository()`. Spring creates the repository object and passes it to the constructor. This is **Dependency Injection**, a core OOP pattern that you will study in depth in Phase 4.
- The `createOrder` method creates a new `Order` object, populates its fields, and passes it to the repository. The repository translates the object into a SQL `INSERT` statement. This is the Object-Relational Mapping (ORM) pattern: Java objects map to database rows.
- The `cancelOrder` method retrieves an existing `Order` object from the database, calls a method on it (`order.cancel()`), and saves it back. The repository translates this into a SQL `UPDATE` statement.

### Scenario 3: DTO class (the API layer)

A Data Transfer Object (DTO) is a class that defines the shape of data sent to and from your API. It is separate from the entity class to prevent exposing internal database details to clients.

```java
package com.company.orderservice.dto;

import java.math.BigDecimal;
import java.time.LocalDateTime;

// A DTO is a simple class with fields, getters, and setters.
// It contains no business logic. Its only job is to carry data
// between the client and the server.
public class OrderResponseDTO {
    private Long id;
    private String orderNumber;
    private BigDecimal totalAmount;
    private String status;
    private LocalDateTime createdAt;

    // Static factory method: converts an Order entity into an OrderResponseDTO.
    // This is a common pattern for converting between layers.
    public static OrderResponseDTO fromEntity(Order order) {
        OrderResponseDTO dto = new OrderResponseDTO();
        dto.id = order.getId();
        dto.orderNumber = order.getOrderNumber();
        dto.totalAmount = order.getTotalAmount();
        dto.status = order.getStatus().name();
        dto.createdAt = order.getCreatedAt();
        return dto;
    }

    // Getters (required for JSON serialization by Jackson)
    public Long getId() { return id; }
    public String getOrderNumber() { return orderNumber; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public String getStatus() { return status; }
    public LocalDateTime getCreatedAt() { return createdAt; }
}
```

**What to notice:**

- The DTO does not expose the `userId` field. This is intentional. The API client does not need to know the internal user ID. DTOs let you control exactly what data is visible to the outside world.
- The `fromEntity()` method is a **static factory method**. It creates and returns a new DTO object from an entity object. This pattern keeps the conversion logic in one place and makes the controller code cleaner.
- When Spring returns this DTO from a controller method, the Jackson library automatically converts it into JSON: `{"id": 1, "orderNumber": "ORD-A3F2B1C4", "totalAmount": 5000.00, "status": "PENDING", "createdAt": "2025-07-10T14:30:00"}`.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Confusing the class with the object

**Wrong thinking:** "I created a Student class, so I have a student."

**Right thinking:** "I created a Student class, which is a blueprint. I need to call `new Student()` to create an actual student object."

```java
Student s;  // This declares a reference variable but does NOT create an object.
            // s is null. Calling s.study() here would throw NullPointerException.

s = new Student();  // NOW an object exists on the heap, and s points to it.
```

**Why it is wrong:** Declaring a variable of a class type only creates a reference on the stack. No object is created until you use the `new` keyword. This is different from primitive types, where `int x = 5;` creates the value immediately.

### Mistake 2: Modifying objects through shared references unintentionally

**Wrong:**

```java
Product original = new Product();
original.name = "Laptop";
original.price = 80000;

Product copy = original;  // This does NOT create a new object!
                          // Both variables point to the SAME object.

copy.price = 50000;
System.out.println(original.price);  // 50000! The "original" is also modified!
```

**Right:**

```java
Product original = new Product();
original.name = "Laptop";
original.price = 80000;

// Create a new object and copy the field values manually
Product copy = new Product();
copy.name = original.name;
copy.price = original.price;
copy.stockQuantity = original.stockQuantity;

copy.price = 50000;
System.out.println(original.price);  // 80000 (unchanged)
```

**Why it is wrong:** The assignment `Product copy = original` copies the **reference**, not the object. Both variables point to the same object on the heap. Modifying the object through one reference is visible through the other. This is a common source of bugs in backend systems when multiple services share references to the same entity object.

### Mistake 3: Putting all logic in the main method

**Wrong:**

```java
public class Main {
    public static void main(String[] args) {
        // 500 lines of code creating objects, processing data,
        // printing results, handling errors, all in one method.
        // This is procedural programming dressed up as OOP.
    }
}
```

**Right:**

```java
// Separate classes with clear responsibilities
public class Order { /* data + business rules */ }
public class OrderService { /* business logic */ }
public class OrderRepository { /* database access */ }
public class OrderController { /* HTTP request handling */ }
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        // main() only starts the application. All logic is in other classes.
    }
}
```

**Why it is wrong:** OOP is about distributing responsibility across multiple classes, not putting everything in one place. A class should have a single, clear purpose. If your `main` method is longer than 20 lines, you are not using OOP effectively.

### Mistake 4: Forgetting to initialize fields

**Wrong:**

```java
public class Order {
    String orderNumber;
    BigDecimal totalAmount;

    void printSummary() {
        System.out.println("Order: " + orderNumber);
        System.out.println("Total: " + totalAmount.toString());
        // NullPointerException! totalAmount is null because it was never initialized.
        // BigDecimal is a reference type, so its default value is null, not 0.
    }
}
```

**Right:**

```java
public class Order {
    String orderNumber;
    BigDecimal totalAmount = BigDecimal.ZERO;  // Initialize to a safe default

    void printSummary() {
        System.out.println("Order: " + orderNumber);
        System.out.println("Total: " + totalAmount.toString());  // Safe
    }
}
```

**Why it is wrong:** Primitive fields default to 0 or false, which are usually safe. Reference type fields default to `null`, which causes `NullPointerException` when you try to call methods on them. Always initialize reference type fields to a meaningful default value or ensure they are set before use.

---

## Key Takeaways

> [!tip] Remember these points
> 1. A class is a blueprint that defines fields (data) and methods (behavior). An object is a concrete instance of a class created with the `new` keyword. Each object has its own independent copy of the instance fields.
> 2. Objects live on the heap. Reference variables live on the stack and hold memory addresses pointing to objects. Assigning one reference to another (`a = b`) copies the address, not the object. Both references then point to the same object.
> 3. The `this` keyword refers to the current object inside an instance method. Use it to resolve naming conflicts between fields and parameters (`this.name = name`).
> 4. Instance variables belong to individual objects. Class variables (`static`) belong to the class and are shared by all objects. Local variables belong to a single method call and live on the stack.
> 5. In Spring Boot backends, every component is a class: entities map to database tables, services contain business logic, controllers handle HTTP requests, and DTOs define API payloads. Objects are created and managed by the Spring framework through Dependency Injection.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Library Book Class (Easy)

Create a class `Book` with the following fields: `title` (String), `author` (String), `isbn` (String), `totalCopies` (int), `availableCopies` (int). Add the following methods:

- `borrow()`: Decreases `availableCopies` by 1 if copies are available. Prints an error if no copies are available.
- `returnBook()`: Increases `availableCopies` by 1 but not beyond `totalCopies`.
- `isAvailable()`: Returns true if `availableCopies > 0`.
- `displayInfo()`: Prints all book details in a formatted manner.

Create three `Book` objects in `main()`, borrow and return books, and verify the available copies change correctly.

**Hint:** The `borrow()` method should check `availableCopies > 0` before decrementing. The `returnBook()` method should check `availableCopies < totalCopies` before incrementing.

### Exercise 2: Shopping Cart with Objects (Medium)

Create two classes:

- `CartItem` with fields: `productName` (String), `unitPrice` (double), `quantity` (int). Add a method `getSubtotal()` that returns `unitPrice * quantity`.
- `ShoppingCart` with a field `items` (an array of `CartItem`, maximum 10 items) and a field `itemCount` (int, tracks how many items have been added). Add methods:
  - `addItem(CartItem item)`: Adds an item to the array if there is space.
  - `getTotal()`: Returns the sum of all item subtotals.
  - `displayCart()`: Prints all items with their subtotals and the grand total.

In `main()`, create a cart, add 3-4 items, and display the cart.

**Hint:** The `ShoppingCart` class uses an array of `CartItem` objects internally. The `itemCount` field tracks the next available index in the array. This is a simplified version of how `ArrayList` works internally.

### Exercise 3: Object References and Aliasing (Medium)

Write a program that demonstrates the difference between reference assignment and object copying. Create a `Student` class with `name` and `cgpa` fields. In `main()`:

1. Create a `Student` object `s1` with name "Saad" and CGPA 3.72.
2. Assign `s2 = s1`. Modify `s2.cgpa` to 3.80. Print both `s1.cgpa` and `s2.cgpa`. Explain in a comment why they are the same.
3. Create a new `Student` object `s3` and manually copy the fields from `s1`. Modify `s3.cgpa` to 3.50. Print both `s1.cgpa` and `s3.cgpa`. Explain in a comment why they are different.
4. Print `s1 == s2` and `s1 == s3`. Explain the results.

**Hint:** `s1 == s2` compares references (memory addresses), not field values.

### Exercise 4: Mini Backend Simulation (Hard, Optional)

Simulate a simplified backend system with three classes:

- `User` with fields: `id` (long), `username` (String), `email` (String), `isActive` (boolean). Add methods `activate()` and `deactivate()`.
- `UserRepository` with a field `users` (array of `User`, max 100) and `count` (int). Add methods:
  - `save(User user)`: Adds the user to the array. Assigns an auto-incremented ID.
  - `findById(long id)`: Returns the `User` with the matching ID, or null if not found.
  - `findByEmail(String email)`: Returns the `User` with the matching email, or null.
  - `findAll()`: Returns an array of all saved users (only the non-null portion).
- `UserService` with a `UserRepository` field (set through a constructor). Add methods:
  - `registerUser(String username, String email)`: Creates a `User`, saves it via the repository. Checks for duplicate emails before saving.
  - `getUser(long id)`: Retrieves a user by ID. Throws an exception if not found.
  - `deactivateUser(long id)`: Finds the user and calls `deactivate()`.

In `main()`, create a repository and service, register 3 users, try to register a duplicate, retrieve users by ID, and deactivate one user.

**Hint:** This exercise simulates the Controller -> Service -> Repository architecture that you will use in Spring Boot. The `UserRepository` replaces the database, the `UserService` contains business logic, and `main()` acts as the controller.

<details>
<summary>Solution for Exercise 1</summary>

```java
public class Book {
    String title;
    String author;
    String isbn;
    int totalCopies;
    int availableCopies;

    void borrow() {
        if (availableCopies > 0) {
            availableCopies--;
            System.out.println("Borrowed '" + title + "'. Available: " + availableCopies);
        } else {
            System.out.println("No copies available for '" + title + "'");
        }
    }

    void returnBook() {
        if (availableCopies < totalCopies) {
            availableCopies++;
            System.out.println("Returned '" + title + "'. Available: " + availableCopies);
        } else {
            System.out.println("All copies of '" + title + "' are already returned");
        }
    }

    boolean isAvailable() {
        return availableCopies > 0;
    }

    void displayInfo() {
        System.out.printf("'%s' by %s (ISBN: %s) | %d/%d copies available%n",
            title, author, isbn, availableCopies, totalCopies);
    }

    public static void main(String[] args) {
        Book book1 = new Book();
        book1.title = "Clean Code";
        book1.author = "Robert C. Martin";
        book1.isbn = "978-0132350884";
        book1.totalCopies = 3;
        book1.availableCopies = 3;

        Book book2 = new Book();
        book2.title = "Effective Java";
        book2.author = "Joshua Bloch";
        book2.isbn = "978-0134685991";
        book2.totalCopies = 2;
        book2.availableCopies = 2;

        book1.displayInfo();
        book1.borrow();
        book1.borrow();
        book1.borrow();
        book1.borrow();  // No copies available
        book1.returnBook();
        book1.displayInfo();
    }
}
```

</details>

<details>
<summary>Solution for Exercise 4 (partial - UserService)</summary>

```java
public class UserService {
    private UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }

    public User registerUser(String username, String email) {
        User existing = repository.findByEmail(email);
        if (existing != null) {
            throw new IllegalArgumentException("Email already registered: " + email);
        }
        User user = new User();
        user.username = username;
        user.email = email;
        user.isActive = true;
        repository.save(user);
        return user;
    }

    public User getUser(long id) {
        User user = repository.findById(id);
        if (user == null) {
            throw new RuntimeException("User not found: " + id);
        }
        return user;
    }

    public void deactivateUser(long id) {
        User user = getUser(id);
        user.deactivate();
        System.out.println("User " + user.username + " deactivated");
    }
}
```

</details>

---

## Related Notes

- [[Java - Methods - Parameters Return Types Overloading]] (Phase 0)
- [[Java - Constructors - Default Parameterized Copy]] (next note)
- [[Java - Encapsulation - Getters Setters Access Modifiers]]

---

## Resources

- [Oracle Java Tutorials: Classes and Objects](https://docs.oracle.com/javase/tutorial/java/javaOO/classes.html) - Official documentation covering class declaration, object creation, and field initialization.
- [Oracle Java Tutorials: Objects](https://docs.oracle.com/javase/tutorial/java/javaOO/objects.html) - Official guide on creating and using objects.
- [Baeldung: Java Objects and Classes](https://www.baeldung.com/java-classes-objects) - Comprehensive guide with memory diagrams.
- [Baeldung: this Keyword in Java](https://www.baeldung.com/java-this-keyword) - Detailed explanation of all uses of the `this` keyword.
- Head First Java by Kathy Sierra - Chapter 3 - The best beginner-friendly explanation of classes, objects, and references with visual diagrams. Highly recommended.
