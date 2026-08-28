---
title: "Java - Methods - Parameters Return Types Overloading"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - methods
  - oop-basics
status: "not-started"
---

# Java - Methods - Parameters Return Types Overloading

> [!abstract] Overview
> A method is a named block of code that performs a specific task, accepts input through parameters, and optionally returns a result. Methods are the fundamental unit of code organization in Java. Every backend system is composed of thousands of methods: controller methods that handle HTTP requests, service methods that contain business logic, repository methods that query databases, and utility methods that perform common operations. Understanding method design, parameter passing, return types, and overloading is essential before moving into object-oriented programming in Phase 1.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]
> - [[Java - Control Flow - If Else Switch]]
> - [[Java - Loops - For While Do-While]]
> - [[Java - Arrays - 1D and 2D]]

---

## Theory

### What is a Method?

A method is a reusable block of code that is defined once and can be called (invoked) from anywhere in your program. Methods allow you to break a large, complex program into smaller, manageable, and testable pieces.

The anatomy of a method declaration:

```java
accessModifier returnType methodName(parameterList) {
    // Method body: the code that executes when the method is called
    // Optional: return statement to send a value back to the caller
}
```

Breaking down each component:

- **Access modifier**: Controls who can call this method. `public` means anyone can call it. `private` means only code within the same class can call it. `protected` means the same class, subclasses, and classes in the same package. Default (no modifier) means only classes in the same package. You will learn the full implications in Phase 1 when you study encapsulation.

- **Return type**: The data type of the value the method sends back to the caller. If the method does not return anything, the return type is `void`.

- **Method name**: A descriptive name following camelCase convention. The name should be a verb or verb phrase that describes what the method does: `calculateTotal()`, `findUserById()`, `isValidEmail()`.

- **Parameter list**: A comma-separated list of input variables enclosed in parentheses. Each parameter has a type and a name. If the method takes no input, the parentheses are empty.

- **Method body**: The code block enclosed in curly braces that defines what the method does.

### Why Do Methods Exist?

Methods solve four fundamental problems in software development:

**1. Code reuse**: Without methods, you would copy and paste the same logic everywhere it is needed. If the logic changes, you would have to find and update every copy. With methods, you define the logic once and call it from multiple places. A single fix in the method fixes all callers.

**2. Abstraction**: Methods let you hide complex implementation details behind a simple name. When you call `orderService.calculateTax(order)`, you do not need to know how the tax is calculated. You only need to know the method name, what input it expects, and what output it returns. This is the foundation of building large backend systems where no single developer understands every line of code.

**3. Testability**: A well-designed method does one thing and does it well. This makes it easy to write unit tests. You can test `calculateTax()` in isolation without needing a running web server, database, or payment gateway. In Phase 2, you will learn JUnit testing, and you will see that every test targets a specific method.

**4. Readability**: A method named `isEligibleForDiscount(user, order)` is self-documenting. A reader understands the intent without reading the implementation. Compare this to a 20-line `if-else` block embedded in the middle of a controller method. Methods turn complex code into readable narratives.

### How Does It Work Internally?

When you call a method, the JVM performs several operations behind the scenes:

**1. Stack frame creation**: The JVM creates a new **stack frame** (also called an activation record) on the call stack. This frame contains:
- The method's local variables (including parameters)
- The return address (where to resume execution after the method finishes)
- A reference to the calling method's stack frame

**2. Parameter passing**: Java passes all parameters **by value**. This means the method receives a copy of the argument, not the original variable. For primitive types, the actual value is copied. For reference types (objects, arrays, Strings), the reference (memory address) is copied, not the object itself.

This distinction is critical:

```java
void modifyPrimitive(int x) {
    x = 100;  // Modifies the local copy. The caller's variable is unchanged.
}

void modifyArray(int[] arr) {
    arr[0] = 100;  // Modifies the actual array on the heap!
    // The reference was copied, but both the caller and the method
    // point to the same array object in memory.
}

void reassignArray(int[] arr) {
    arr = new int[]{100, 200};  // Reassigns the local copy of the reference.
    // The caller's array is unchanged because the caller still holds
    // the original reference.
}
```

**3. Execution**: The JVM executes the method body instruction by instruction.

**4. Return**: When the method reaches a `return` statement (or the closing brace for `void` methods), the JVM:
- Pops the stack frame off the call stack
- Passes the return value (if any) back to the caller
- Resumes execution at the return address

**Memory visualization of a method call:**

```
Call Stack (grows downward)
+---------------------------+
| main() stack frame        |
|   int age = 22            |
|   return address: line 15 |
+---------------------------+
| calculateDiscount() frame |  <-- Created when calculateDiscount() is called
|   double price = 500.0    |
|   double rate = 0.10      |
|   return address: line 8  |
+---------------------------+  <-- Popped when calculateDiscount() returns
```

> [!tip] Key Insight
> Java is strictly pass-by-value. There is no pass-by-reference in Java. When you pass an object to a method, you pass a copy of the reference, not a copy of the object. This means the method can modify the object's internal state (because both references point to the same object on the heap), but it cannot reassign the caller's variable to point to a different object. This is one of the most misunderstood concepts in Java and a frequent interview question.

### Parameters: Types and Patterns

**Formal parameters vs actual arguments**: The variables declared in the method signature are called **formal parameters** (or just parameters). The values passed when calling the method are called **actual arguments** (or just arguments).

```java
// 'price' and 'quantity' are formal parameters
double calculateTotal(double price, int quantity) {
    return price * quantity;
}

// 99.50 and 3 are actual arguments
double total = calculateTotal(99.50, 3);
```

**Varargs (variable-length arguments)**: Java allows a method to accept a variable number of arguments of the same type using the `...` syntax. Internally, the varargs parameter is treated as an array.

```java
double sum(double... numbers) {
    double total = 0;
    for (double n : numbers) {
        total += n;
    }
    return total;
}

// All of these calls are valid:
sum(1.0);
sum(1.0, 2.0, 3.0);
sum(1.0, 2.0, 3.0, 4.0, 5.0);
```

A method can have at most one varargs parameter, and it must be the last parameter in the list.

### Return Types

A method can return any data type: primitives, objects, arrays, or `void` (nothing).

```java
int getCount() { ... }              // Returns a primitive
String getName() { ... }            // Returns an object
int[] getScores() { ... }           // Returns an array
List<Order> getOrders() { ... }     // Returns a collection
void sendEmail(String to) { ... }   // Returns nothing
```

A method with a non-void return type **must** return a value on every possible execution path. The compiler checks this and will reject code where a path exists that does not return a value.

```java
// COMPILATION ERROR: not all paths return a value
String getGrade(int score) {
    if (score >= 80) {
        return "A";
    }
    // What happens if score < 80? No return statement for this path.
}

// CORRECT:
String getGrade(int score) {
    if (score >= 80) {
        return "A";
    }
    return "B";  // All paths now return a value
}
```

### Method Overloading

Method overloading allows you to define multiple methods with the **same name** but **different parameter lists** in the same class. The compiler distinguishes them by the number, types, or order of parameters.

```java
// Three overloaded methods, all named "calculateArea"
double calculateArea(double side) {
    return side * side;  // Square
}

double calculateArea(double length, double width) {
    return length * width;  // Rectangle
}

double calculateArea(double radius, boolean isCircle) {
    return Math.PI * radius * radius;  // Circle
}
```

**Rules for overloading:**

- The method name must be the same.
- The parameter list must differ in number, type, or order of parameters.
- The return type alone is NOT sufficient to distinguish overloaded methods. You cannot have two methods that differ only in return type.
- Overloading is resolved at **compile time** (static binding). The compiler looks at the argument types in the method call and selects the matching overload.

**What does NOT count as overloading:**

```java
int add(int a, int b) { return a + b; }
double add(int a, int b) { return a + b; }  // COMPILATION ERROR!
// Same name, same parameters, different return type. Not valid overloading.
```

---

## Syntax and Basic Examples

### Example 1: Basic method definitions and calls

```java
public class MethodBasics {

    // A void method with no parameters
    static void printWelcome() {
        System.out.println("Welcome to the Order Management System");
        System.out.println("======================================");
    }

    // A method with parameters and a return value
    static double calculateDiscount(double price, double discountPercent) {
        if (discountPercent < 0 || discountPercent > 100) {
            return 0;  // Invalid discount, return no discount
        }
        return price * (discountPercent / 100);
    }

    // A method that returns a boolean
    static boolean isEligibleForFreeShipping(double orderTotal, boolean isMember) {
        if (isMember) {
            return orderTotal >= 500;  // Members get free shipping above 500 BDT
        }
        return orderTotal >= 1000;     // Non-members need 1000 BDT
    }

    // A method that returns a String
    static String formatCurrency(double amount) {
        return String.format("%.2f BDT", amount);
    }

    public static void main(String[] args) {
        printWelcome();

        double originalPrice = 2500.0;
        double discount = calculateDiscount(originalPrice, 15);
        double finalPrice = originalPrice - discount;

        System.out.println("Original: " + formatCurrency(originalPrice));
        System.out.println("Discount: " + formatCurrency(discount));
        System.out.println("Final: " + formatCurrency(finalPrice));

        boolean freeShipping = isEligibleForFreeShipping(finalPrice, true);
        System.out.println("Free shipping: " + freeShipping);
    }
}
```

**Output:**
```
Welcome to the Order Management System
======================================
Original: 2500.00 BDT
Discount: 375.00 BDT
Final: 2125.00 BDT
Free shipping: true
```

### Example 2: Pass-by-value demonstration

```java
public class PassByValueDemo {

    static void tryToChangePrimitive(int number) {
        number = 999;
        System.out.println("Inside method (primitive): " + number);  // 999
    }

    static void tryToChangeArray(int[] numbers) {
        numbers[0] = 999;  // This DOES modify the original array
        System.out.println("Inside method (array[0]): " + numbers[0]);  // 999
    }

    static void tryToReassignArray(int[] numbers) {
        numbers = new int[]{999, 888, 777};  // This does NOT affect the caller
        System.out.println("Inside method (reassigned): " + numbers[0]);  // 999
    }

    static void tryToChangeString(String text) {
        text = "Modified";  // This does NOT affect the caller
        // Strings are immutable, and the reference is passed by value.
        System.out.println("Inside method (String): " + text);  // Modified
    }

    public static void main(String[] args) {
        int x = 10;
        tryToChangePrimitive(x);
        System.out.println("After method (primitive): " + x);  // 10 (unchanged!)

        int[] arr = {1, 2, 3};
        tryToChangeArray(arr);
        System.out.println("After method (array[0]): " + arr[0]);  // 999 (changed!)

        int[] arr2 = {1, 2, 3};
        tryToReassignArray(arr2);
        System.out.println("After method (reassigned): " + arr2[0]);  // 1 (unchanged!)

        String name = "Original";
        tryToChangeString(name);
        System.out.println("After method (String): " + name);  // Original (unchanged!)
    }
}
```

**Output:**
```
Inside method (primitive): 999
After method (primitive): 10
Inside method (array[0]): 999
After method (array[0]): 999
Inside method (reassigned): 999
After method (reassigned): 1
Inside method (String): Modified
After method (String): Original
```

### Example 3: Method overloading

```java
public class OverloadingDemo {

    // Overloaded search methods: same name, different parameters
    static String searchProduct(int productId) {
        return "Searching by ID: " + productId;
    }

    static String searchProduct(String productName) {
        return "Searching by name: " + productName;
    }

    static String searchProduct(String category, double maxPrice) {
        return "Searching in category '" + category + "' under " + maxPrice + " BDT";
    }

    static String searchProduct(String category, double minPrice, double maxPrice) {
        return "Searching in category '" + category + "' between "
            + minPrice + " and " + maxPrice + " BDT";
    }

    // Overloaded calculateTax methods
    static double calculateTax(double amount) {
        return amount * 0.15;  // Default 15% VAT
    }

    static double calculateTax(double amount, double taxRate) {
        return amount * taxRate;  // Custom tax rate
    }

    static double calculateTax(double amount, boolean isExport) {
        if (isExport) {
            return 0;  // Export orders are tax-exempt
        }
        return amount * 0.15;
    }

    public static void main(String[] args) {
        System.out.println(searchProduct(1042));
        System.out.println(searchProduct("Wireless Mouse"));
        System.out.println(searchProduct("Electronics", 5000.0));
        System.out.println(searchProduct("Electronics", 1000.0, 5000.0));

        System.out.println("\nTax on 1000 BDT: " + calculateTax(1000));
        System.out.println("Tax at 5%: " + calculateTax(1000, 0.05));
        System.out.println("Tax on export: " + calculateTax(1000, true));
    }
}
```

**Output:**
```
Searching by ID: 1042
Searching by name: Wireless Mouse
Searching in category 'Electronics' under 5000.0 BDT
Searching in category 'Electronics' between 1000.0 and 5000.0 BDT

Tax on 1000 BDT: 150.0
Tax at 5%: 50.0
Tax on export: 0.0
```

### Example 4: Varargs and multiple return values

```java
import java.util.Arrays;

public class VarargsDemo {

    // Varargs: accepts any number of double values
    static double average(double... values) {
        if (values.length == 0) {
            throw new IllegalArgumentException("Cannot calculate average of zero values");
        }
        double sum = 0;
        for (double v : values) {
            sum += v;
        }
        return sum / values.length;
    }

    // Varargs with a regular parameter (varargs must be last)
    static String joinWithPrefix(String prefix, String... items) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < items.length; i++) {
            sb.append(prefix).append(items[i]);
            if (i < items.length - 1) {
                sb.append(", ");
            }
        }
        return sb.toString();
    }

    // Java methods can only return one value.
    // To return multiple values, return an array or an object.
    static int[] findMinAndMax(int[] numbers) {
        if (numbers == null || numbers.length == 0) {
            throw new IllegalArgumentException("Array must not be empty");
        }
        int min = numbers[0];
        int max = numbers[0];
        for (int n : numbers) {
            if (n < min) min = n;
            if (n > max) max = n;
        }
        return new int[]{min, max};  // Return both values in an array
    }

    public static void main(String[] args) {
        System.out.println("Average of 3: " + average(10, 20, 30));
        System.out.println("Average of 5: " + average(10, 20, 30, 40, 50));
        System.out.println("Average of 1: " + average(42));

        System.out.println(joinWithPrefix("Item: ", "Laptop", "Mouse", "Keyboard"));

        int[] data = {45, 12, 78, 3, 91, 56};
        int[] minMax = findMinAndMax(data);
        System.out.println("Min: " + minMax[0] + ", Max: " + minMax[1]);
    }
}
```

**Output:**
```
Average of 3: 20.0
Average of 5: 30.0
Average of 1: 42.0
Item: Laptop, Item: Mouse, Item: Keyboard
Min: 3, Max: 91
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Methods are the building blocks of every backend application. Here are three realistic scenarios showing how methods are designed and used in production Spring Boot systems.

### Scenario 1: Service layer methods with clear responsibilities

In a Spring Boot backend, the service layer contains the business logic. Each method in a service class should do exactly one thing. This is the **Single Responsibility Principle** (covered in detail in Phase 2).

```java
package com.company.orderservice.service;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.time.LocalDateTime;
import java.util.List;

public class OrderService {

    // Method 1: Creates an order. Returns the created order object.
    public Order createOrder(Long userId, List<OrderItem> items) {
        validateOrderItems(items);  // Delegates validation to another method

        BigDecimal subtotal = calculateSubtotal(items);
        BigDecimal tax = calculateTax(subtotal);
        BigDecimal total = subtotal.add(tax);

        Order order = new Order();
        order.setOrderNumber(generateOrderNumber());
        order.setUserId(userId);
        order.setItems(items);
        order.setSubtotal(subtotal);
        order.setTax(tax);
        order.setTotal(total);
        order.setStatus(OrderStatus.PENDING);
        order.setCreatedAt(LocalDateTime.now());

        return orderRepository.save(order);
    }

    // Method 2: Validates order items. Throws exception if invalid.
    // Notice: void return type. This method either succeeds silently
    // or throws an exception. This is a common pattern for validation.
    private void validateOrderItems(List<OrderItem> items) {
        if (items == null || items.isEmpty()) {
            throw new IllegalArgumentException("Order must contain at least one item");
        }
        if (items.size() > 100) {
            throw new IllegalArgumentException("Order cannot exceed 100 items");
        }
        for (OrderItem item : items) {
            if (item.getQuantity() <= 0) {
                throw new IllegalArgumentException(
                    "Item quantity must be positive: " + item.getProductName()
                );
            }
        }
    }

    // Method 3: Calculates subtotal. Pure computation, no side effects.
    private BigDecimal calculateSubtotal(List<OrderItem> items) {
        BigDecimal subtotal = BigDecimal.ZERO;
        for (OrderItem item : items) {
            BigDecimal itemTotal = item.getUnitPrice()
                .multiply(new BigDecimal(item.getQuantity()));
            subtotal = subtotal.add(itemTotal);
        }
        return subtotal.setScale(2, RoundingMode.HALF_UP);
    }

    // Method 4: Calculates tax. Isolated so the tax rate can change
    // without affecting the rest of the order creation logic.
    private BigDecimal calculateTax(BigDecimal subtotal) {
        BigDecimal taxRate = new BigDecimal("0.15");
        return subtotal.multiply(taxRate).setScale(2, RoundingMode.HALF_UP);
    }

    // Method 5: Generates a unique order number.
    private String generateOrderNumber() {
        String timestamp = String.valueOf(System.currentTimeMillis());
        String random = String.valueOf((int) (Math.random() * 10000));
        return "ORD-" + timestamp + "-" + random;
    }
}
```

**What to notice:**

- The `createOrder()` method is the **public** entry point. It orchestrates the process by calling several **private** helper methods. This is the standard pattern: one public method that coordinates, multiple private methods that do the actual work.
- `validateOrderItems()` returns `void`. Validation methods either succeed (return nothing) or fail (throw an exception). This is a clean contract that callers can rely on.
- `calculateSubtotal()` and `calculateTax()` are **pure functions**. They take input, compute a result, and return it without modifying any external state. Pure functions are the easiest to test and the least likely to contain bugs.
- Each method has a single, clear responsibility. If the tax rate changes, you only modify `calculateTax()`. If the order number format changes, you only modify `generateOrderNumber()`. This is the power of good method design.

### Scenario 2: Overloaded repository methods for flexible querying

Spring Data JPA allows you to define overloaded methods that query the database with different criteria.

```java
package com.company.orderservice.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

public interface OrderRepository extends JpaRepository<Order, Long> {

    // Find by exact ID (inherited from JpaRepository)
    // Optional<Order> findById(Long id);

    // Overloaded find methods with different criteria
    List<Order> findByUserId(Long userId);

    List<Order> findByUserIdAndStatus(Long userId, OrderStatus status);

    List<Order> findByUserIdAndStatusAndCreatedAtBetween(
        Long userId, OrderStatus status, LocalDateTime start, LocalDateTime end
    );

    List<Order> findByOrderNumber(String orderNumber);

    @Query("SELECT o FROM Order o WHERE o.total > :minAmount ORDER BY o.total DESC")
    List<Order> findHighValueOrders(BigDecimal minAmount);

    @Query("SELECT o FROM Order o WHERE o.total > :minAmount AND o.status = :status")
    List<Order> findHighValueOrders(BigDecimal minAmount, OrderStatus status);
    // Overloaded: same name, different parameters. Spring Data resolves
    // the correct query based on the arguments you pass.
}
```

**What to notice:**

- The method names follow Spring Data's naming convention. `findByUserIdAndStatus` automatically generates the SQL query `SELECT * FROM orders WHERE user_id = ? AND status = ?`. No SQL code is needed.
- The two `findHighValueOrders()` methods are overloaded. One takes a minimum amount, the other takes a minimum amount and a status filter. This gives callers flexibility without requiring them to pass null for optional parameters.
- The return type `Optional<Order>` for single results prevents NullPointerException. The caller must explicitly handle the case where no order is found.

### Scenario 3: Utility methods with static and varargs patterns

```java
package com.company.orderservice.util;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Collection;

public final class AppUtils {

    // Private constructor prevents instantiation.
    // This class only holds static utility methods.
    private AppUtils() {}

    // Null-safe string check
    public static boolean isNullOrEmpty(String value) {
        return value == null || value.isBlank();
    }

    // Null-safe collection check
    public static boolean isNullOrEmpty(Collection<?> collection) {
        return collection == null || collection.isEmpty();
    }
    // Overloaded: same name, different parameter type (String vs Collection)

    // Safe BigDecimal parsing with a default fallback
    public static BigDecimal parseBigDecimal(String value, BigDecimal defaultValue) {
        if (isNullOrEmpty(value)) {
            return defaultValue;
        }
        try {
            return new BigDecimal(value.strip()).setScale(2, RoundingMode.HALF_UP);
        } catch (NumberFormatException e) {
            return defaultValue;
        }
    }

    // Overloaded: no default value, throws exception on failure
    public static BigDecimal parseBigDecimal(String value) {
        if (isNullOrEmpty(value)) {
            throw new IllegalArgumentException("Cannot parse null or blank string to BigDecimal");
        }
        return new BigDecimal(value.strip()).setScale(2, RoundingMode.HALF_UP);
    }

    // Format a timestamp for API responses
    public static String formatTimestamp(LocalDateTime dateTime) {
        if (dateTime == null) {
            return null;
        }
        return dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);
    }

    // Varargs: concatenate multiple strings with a separator, skipping nulls
    public static String concatNonNull(String separator, String... parts) {
        StringBuilder sb = new StringBuilder();
        for (String part : parts) {
            if (part != null && !part.isBlank()) {
                if (sb.length() > 0) {
                    sb.append(separator);
                }
                sb.append(part);
            }
        }
        return sb.toString();
    }
}
```

**What to notice:**

- All methods are `public static` because utility methods do not need an object instance. They belong to the class itself.
- The class is `final` with a private constructor. This prevents anyone from creating an `AppUtils` object or subclassing it. This is the standard pattern for utility classes in Java.
- The overloaded `isNullOrEmpty()` methods handle both Strings and Collections. The compiler selects the correct version based on the argument type.
- The overloaded `parseBigDecimal()` methods demonstrate a common API design pattern: one version with a default value for lenient parsing, and one version that throws an exception for strict parsing. Callers choose the version that matches their needs.
- The `concatNonNull()` method uses varargs to accept any number of String arguments and filters out nulls. This is useful for building addresses, full names, or log messages where some parts might be missing.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Writing methods that do too many things

**Wrong:**
```java
// This method does five different things. It is impossible to test,
// hard to read, and will break when any single requirement changes.
public String processOrder(String userId, String productId, int qty,
                            String couponCode, String paymentMethod) {
    // Validate user
    // Fetch product from database
    // Check inventory
    // Apply coupon discount
    // Calculate tax
    // Process payment
    // Send confirmation email
    // Update inventory
    // Generate invoice
    // Return order confirmation string
    // ... 200 lines of code ...
}
```

**Right:**
```java
public Order processOrder(Long userId, CartItem[] items, String couponCode) {
    User user = userService.findById(userId);
    List<OrderItem> orderItems = inventoryService.reserveItems(items);
    BigDecimal discount = couponService.applyCoupon(couponCode, orderItems);
    BigDecimal total = pricingService.calculateTotal(orderItems, discount);
    PaymentResult payment = paymentService.charge(user, total);
    Order order = orderRepository.createOrder(user, orderItems, total, payment);
    emailService.sendConfirmation(order);
    return order;
}
```

**Why it is wrong:** A method that does too many things violates the Single Responsibility Principle. It is hard to test (you need to mock five different systems), hard to debug (a bug could be in any of the five steps), and hard to reuse (what if you want to calculate the total without processing payment?). Break complex logic into small, focused methods.

### Mistake 2: Ignoring the return value of a method

**Wrong:**
```java
String email = "  SAAD@EXAMPLE.COM  ";
email.strip();          // Return value is discarded!
email.toLowerCase();    // Return value is discarded!
System.out.println(email);  // Still "  SAAD@EXAMPLE.COM  "
```

**Right:**
```java
String email = "  SAAD@EXAMPLE.COM  ";
email = email.strip().toLowerCase();  // Assign the return value back
System.out.println(email);  // "saad@example.com"
```

**Why it is wrong:** This is the same immutability issue from the Strings note, but it applies to any method that returns a new value. If a method returns a result and you do not capture it, the result is lost. The compiler does not warn you about this because there are legitimate cases where you call a method for its side effects only (like `list.add(item)` where you ignore the boolean return).

### Mistake 3: Confusing method overloading with method overriding

**Wrong thinking:** "I changed the return type of the method, so it is overloaded."

**Right thinking:** "Overloading means same name, different parameters, resolved at compile time. Overriding means same name, same parameters, in a subclass, resolved at runtime. I will learn overriding in Phase 1."

**Why it is wrong:** Overloading and overriding are fundamentally different mechanisms. Overloading is about providing multiple versions of a method in the same class. Overriding is about a subclass replacing a parent class's method. Changing only the return type does not create a valid overload and will cause a compilation error.

### Mistake 4: Passing too many parameters (long parameter lists)

**Wrong:**
```java
public Order createOrder(String firstName, String lastName, String email,
                          String phone, String address, String city,
                          String zip, String country, Long productId,
                          int quantity, String couponCode, String paymentMethod,
                          String cardNumber, String expiryDate, String cvv) {
    // 15 parameters! This is unreadable, error-prone, and impossible to maintain.
}
```

**Right:**
```java
public Order createOrder(Customer customer, List<OrderItem> items,
                          PaymentDetails payment, String couponCode) {
    // 4 parameters, each a meaningful object that groups related data.
}
```

**Why it is wrong:** Methods with more than 3-4 parameters are a code smell. Long parameter lists are hard to read, easy to mix up (especially when multiple parameters have the same type), and indicate that the method is doing too much or that related data should be grouped into objects. In backend development, this pattern is solved using **DTOs** (Data Transfer Objects), which you will learn in Phase 4.

---

## Key Takeaways

> [!tip] Remember these points
> 1. A method is a named, reusable block of code with a return type, a name, and a parameter list. Methods are the fundamental unit of organization in Java backend systems.
> 2. Java is strictly **pass-by-value**. Primitives are copied by value. Object references are copied by value (the reference is copied, not the object). This means methods can modify an object's internal state but cannot reassign the caller's variable.
> 3. **Method overloading** allows multiple methods with the same name but different parameter lists in the same class. The return type alone does not distinguish overloads. Overloading is resolved at compile time.
> 4. Design methods with a **single responsibility**. Each method should do one thing well. Use `private` helper methods to break complex logic into small, testable pieces. Avoid methods with more than 3-4 parameters.
> 5. Use `void` for methods that perform actions without returning data (validation, sending emails, logging). Use specific return types for methods that compute or retrieve data. Use `Optional<T>` for methods that might not find a result.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Temperature Converter (Easy)
Write three overloaded methods named `convertTemperature`:
1. `convertTemperature(double celsius, String toUnit)` where `toUnit` is `"F"` or `"K"`.
2. `convertTemperature(double value, String fromUnit, String toUnit)` that handles any combination of `"C"`, `"F"`, and `"K"`.
3. `convertTemperature(double... celsiusValues)` that converts an array of Celsius values to Fahrenheit and returns a `double[]`.

Test all three versions in `main()`.

**Hint:** Celsius to Fahrenheit: `F = C * 9/5 + 32`. Celsius to Kelvin: `K = C + 273.15`.

### Exercise 2: Array Utility Methods (Medium)
Write a utility class `ArrayUtils` with the following static methods:
1. `sum(int[] arr)` - returns the sum of all elements.
2. `sum(double[] arr)` - overloaded for doubles.
3. `contains(int[] arr, int target)` - returns true if the target exists in the array.
4. `reverse(int[] arr)` - returns a new reversed array (does not modify the original).
5. `toString(int[] arr)` - returns a formatted string like `"[1, 2, 3]"` without using `Arrays.toString()`.

Test all methods in `main()`.

**Hint:** For `reverse()`, create a new array of the same length and copy elements in reverse order using a loop.

### Exercise 3: Pass-by-Value Proof (Medium)
Write a program that proves Java is pass-by-value. Create a `Student` class (just a simple class with a `name` field and a constructor). Write three methods:
1. `changeName(Student s, String newName)` - modifies `s.name`. Show that the original object IS affected.
2. `reassignStudent(Student s)` - assigns `s = new Student("New")`. Show that the original reference is NOT affected.
3. `changePrimitive(int x)` - modifies `x`. Show that the original variable is NOT affected.

Print the state of all variables before and after each method call to demonstrate the behavior.

**Hint:** You can define a simple class inside the same file:
```java
class Student {
    String name;
    Student(String name) { this.name = name; }
}
```

### Exercise 4: Order Processing Pipeline (Hard, Optional)
Write a program that simulates an order processing pipeline using methods. Define the following methods:
1. `validateOrder(String[] items, int[] quantities, double[] prices)` - checks that arrays are the same length, quantities are positive, and prices are non-negative. Returns `boolean`.
2. `calculateSubtotal(int[] quantities, double[] prices)` - returns the subtotal as a `double`.
3. `applyDiscount(double subtotal, String couponCode)` - returns the discounted amount. Use a switch on the coupon code: "SAVE10" gives 10% off, "SAVE20" gives 20% off, "EID50" gives 50% off, anything else gives 0% off.
4. `calculateTax(double amount, String region)` - returns tax. "DHAKA" is 15%, "CHITTAGONG" is 12%, "SYLHET" is 10%, default is 15%.
5. `generateReceipt(String customerName, String[] items, int[] quantities, double[] prices, String coupon, String region)` - calls all the above methods and prints a formatted receipt.

Test with a sample order in `main()`.

**Hint:** The `generateReceipt()` method should call the other methods in sequence, composing the final output. This mirrors how a real backend service method orchestrates helper methods.

### Solution
For Exercise 1:

```java
public class TemperatureConverter {

    static double convertTemperature(double celsius, String toUnit) {
        return switch (toUnit.toUpperCase()) {
            case "F" -> celsius * 9.0 / 5.0 + 32;
            case "K" -> celsius + 273.15;
            default -> throw new IllegalArgumentException("Unknown unit: " + toUnit);
        };
    }

    static double convertTemperature(double value, String fromUnit, String toUnit) {
        double celsius = switch (fromUnit.toUpperCase()) {
            case "C" -> value;
            case "F" -> (value - 32) * 5.0 / 9.0;
            case "K" -> value - 273.15;
            default -> throw new IllegalArgumentException("Unknown unit: " + fromUnit);
        };
        return convertTemperature(celsius, toUnit);
    }

    static double[] convertTemperature(double... celsiusValues) {
        double[] fahrenheit = new double[celsiusValues.length];
        for (int i = 0; i < celsiusValues.length; i++) {
            fahrenheit[i] = celsiusValues[i] * 9.0 / 5.0 + 32;
        }
        return fahrenheit;
    }

    public static void main(String[] args) {
        System.out.println("100C to F: " + convertTemperature(100, "F"));
        System.out.println("100C to K: " + convertTemperature(100, "K"));
        System.out.println("212F to C: " + convertTemperature(212, "F", "C"));
        System.out.println("300K to F: " + convertTemperature(300, "K", "F"));

        double[] results = convertTemperature(0, 25, 37, 100);
        System.out.print("Batch C to F: ");
        for (double r : results) {
            System.out.print(r + " ");
        }
        System.out.println();
    }
}
```

For Exercise 2:

```java
public class ArrayUtils {

    static int sum(int[] arr) {
        int total = 0;
        for (int n : arr) total += n;
        return total;
    }

    static double sum(double[] arr) {
        double total = 0;
        for (double n : arr) total += n;
        return total;
    }

    static boolean contains(int[] arr, int target) {
        for (int n : arr) {
            if (n == target) return true;
        }
        return false;
    }

    static int[] reverse(int[] arr) {
        int[] reversed = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            reversed[i] = arr[arr.length - 1 - i];
        }
        return reversed;
    }

    static String toString(int[] arr) {
        StringBuilder sb = new StringBuilder("[");
        for (int i = 0; i < arr.length; i++) {
            sb.append(arr[i]);
            if (i < arr.length - 1) sb.append(", ");
        }
        sb.append("]");
        return sb.toString();
    }

    public static void main(String[] args) {
        int[] nums = {1, 2, 3, 4, 5};
        System.out.println("Sum int: " + sum(nums));
        System.out.println("Sum double: " + sum(new double[]{1.5, 2.5, 3.0}));
        System.out.println("Contains 3: " + contains(nums, 3));
        System.out.println("Contains 9: " + contains(nums, 9));
        System.out.println("Reverse: " + toString(reverse(nums)));
        System.out.println("Original: " + toString(nums));
    }
}
```

---

## Related Notes

- [[Java - Strings and String Methods]]
- [[Java - Basic Input Output - Scanner and System.out]]
- [[Java - Classes and Objects]] (Phase 1)

---

## Resources

- [Oracle Java Tutorials: Defining Methods](https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html) - Official documentation covering method declaration, naming, and body.
- [Oracle Java Tutorials: Passing Information to a Method](https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html) - Official explanation of parameters, arguments, and pass-by-value.
- [Baeldung: Java Method Overloading](https://www.baeldung.com/java-method-overload-override) - Clear comparison of overloading vs overriding with examples.
- [Baeldung: Java Pass by Value or Pass by Reference](https://www.baeldung.com/java-pass-by-value-or-pass-by-reference) - Definitive explanation with memory diagrams proving Java is pass-by-value.
- [Clean Code by Robert C. Martin - Chapter 3: Functions](https://www.oreilly.com/library/view/clean-code/9780136083238/) - The best resource on how to design good methods. Highly recommended reading when you reach Phase 2.
