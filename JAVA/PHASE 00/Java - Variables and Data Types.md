---
title: "Java - Variables and Data Types"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - variables
  - data-types
status: "not-started"
---

# Java - Variables and Data Types

> [!abstract] Overview
> Variables are named containers that store data in memory. Java is a statically typed language, which means every variable must have a declared type that determines what kind of data it can hold, how much memory it occupies, and what operations are valid on it. Understanding Java's type system is critical because type errors are the most common source of bugs in backend systems that handle money, dates, IDs, and user input.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - How Java Works - JVM JRE JDK]]
> - [[Java - Setting Up Environment - IntelliJ and JDK]]

---

## Theory

### What is a Variable?

A variable is a named location in your computer's memory that holds a value. Think of it like a labeled box. The label is the variable name, the size of the box is determined by the data type, and the contents of the box are the value.

In Java, you must declare a variable before you use it. Declaration means telling the compiler three things:

1. **The type**: What kind of data will this variable hold? (a number, a letter, true/false, etc.)
2. **The name**: What will you call this variable so you can refer to it later?
3. **The value** (optional at declaration): What is the initial data inside the box?

```java
int age = 22;
//  ^    ^    ^
// type  name  value
```

Once you declare a variable with a type, that type cannot change. You cannot later store a text string in an `int` variable. This is what "statically typed" means, and it is one of the biggest differences between Java and Python.

### Why Does Java Have So Many Data Types?

Java provides eight **primitive data types** built into the language. You might wonder why Java needs `byte`, `short`, `int`, and `long` when they all store whole numbers. The answer is memory efficiency.

In backend systems, you often deal with millions of records. If you have a database table with 100 million users and you store each user's age as a `long` (8 bytes) instead of a `byte` (1 byte), you waste 700 megabytes of memory on a single column. In a large-scale system with hundreds of such columns, this adds up to gigabytes of wasted RAM.

Java gives you the choice so you can optimize when it matters. For most everyday programming, you will use `int` for whole numbers, `double` for decimals, `boolean` for true/false, and `String` for text. The smaller and larger types exist for specific situations.

### How Does It Work Internally?

When you declare a variable, the JVM allocates memory for it. Where that memory lives depends on the type of variable:

**Stack Memory**: Local variables (declared inside methods) are stored on the stack. The stack is fast, organized, and automatically cleaned up when the method finishes. Primitive values are stored directly on the stack.

```java
public void calculate() {
    int x = 10;      // x lives on the stack, value 10 is stored directly
    double price = 9.99; // price lives on the stack, value 9.99 is stored directly
}
// When calculate() finishes, x and price are destroyed automatically.
```

**Heap Memory**: Objects (like `String`, arrays, and custom classes) are stored on the heap. The heap is larger but slower. When you create an object, the actual data lives on the heap, and a **reference** (like a pointer or address) to that data is stored on the stack.

```java
public void greet() {
    String name = "Saad";
    // "Saad" lives on the heap.
    // The variable 'name' lives on the stack and holds a reference (address) to "Saad".
}
```

This distinction between stack and heap becomes very important when you study memory leaks, garbage collection, and performance tuning in later phases.

### The Eight Primitive Data Types

| Type | Size | Range | Default Value | Use Case |
|------|------|-------|---------------|----------|
| `byte` | 1 byte | -128 to 127 | 0 | Small numbers, raw binary data, file I/O |
| `short` | 2 bytes | -32,768 to 32,767 | 0 | Rarely used, sometimes for memory optimization in large arrays |
| `int` | 4 bytes | -2,147,483,648 to 2,147,483,647 | 0 | Default choice for whole numbers. IDs, counts, ages |
| `long` | 8 bytes | -9.2 x 10^18 to 9.2 x 10^18 | 0L | Large numbers. Timestamps, database primary keys, file sizes |
| `float` | 4 bytes | ~6-7 decimal digits of precision | 0.0f | Rarely used in backend. Graphics and scientific computing |
| `double` | 8 bytes | ~15-16 decimal digits of precision | 0.0d | Default choice for decimal numbers. Prices, coordinates, rates |
| `char` | 2 bytes | 0 to 65,535 (Unicode) | '\u0000' | Single characters. Rarely used alone in backend |
| `boolean` | 1 bit (JVM dependent) | true or false | false | Flags, conditions, status indicators |

> [!warning] Important: Floating Point Precision
> Never use `float` or `double` to represent money in a backend system. Floating-point arithmetic produces rounding errors. `0.1 + 0.2` does not equal `0.3` in floating-point math. For money, use `BigDecimal` (a reference type you will learn later). This is one of the most common and expensive mistakes in backend development.

### Reference Types

Everything in Java that is not a primitive is a **reference type**. Reference types include:

- **String**: Text data. `"Hello"` is a String object.
- **Arrays**: Collections of elements. `int[] numbers = {1, 2, 3};`
- **Classes**: Objects you create from classes. `User user = new User();`
- **Interfaces, Enums, Records**: You will learn these in Phase 1.

The critical difference: primitive variables hold the actual value, reference variables hold a memory address pointing to the actual value on the heap.

### Type Casting

Sometimes you need to convert a value from one type to another. Java has two kinds of casting:

**Implicit Casting (Widening)**: Happens automatically when you assign a smaller type to a larger type. No data is lost.

```java
int count = 100;
long bigCount = count;  // int automatically becomes long. Safe.
double price = count;   // int automatically becomes double. Safe.
```

**Explicit Casting (Narrowing)**: You must manually tell Java to convert a larger type to a smaller type. Data might be lost.

```java
long population = 7_000_000_000L;
int smallPop = (int) population;  // You must write (int) explicitly.
// Result: smallPop is NOT 7 billion. It overflows and becomes a wrong number.
```

### The `var` Keyword (Java 10+)

Starting from Java 10, you can use `var` for local variables. The compiler infers the type from the value you assign.

```java
var name = "Saad";       // Compiler infers String
var age = 22;            // Compiler infers int
var price = 9.99;        // Compiler infers double
var isActive = true;     // Compiler infers boolean
```

`var` does not make Java dynamically typed. The type is still fixed at compile time. `var` is just syntactic sugar to reduce verbosity. You cannot reassign a `var` variable to a different type later.

> [!tip] Key Insight
> Use `var` when the type is obvious from the right-hand side (`var users = new ArrayList<User>()`). Do not use `var` when the type is unclear (`var result = service.process(data)` -- what type is result?). In professional backend code, readability matters more than brevity.

### Constants with `final`

A variable declared with the `final` keyword cannot be reassigned after initialization. By convention, constant names use UPPER_SNAKE_CASE.

```java
final double TAX_RATE = 0.15;
final int MAX_RETRY_ATTEMPTS = 3;
final String DEFAULT_CURRENCY = "BDT";

TAX_RATE = 0.20;  // COMPILATION ERROR. Cannot reassign a final variable.
```

---

## Syntax and Basic Examples

### Example 1: Declaring and Using All Primitive Types

```java
public class PrimitiveDemo {
    public static void main(String[] args) {
        // Whole numbers
        byte age = 22;                    // Small number, fits in 1 byte
        short year = 2025;                // Fits in 2 bytes
        int population = 170_000_000;     // Bangladesh population. Underscores for readability.
        long worldPopulation = 8_000_000_000L;  // Note the L suffix. Required for long literals.

        // Decimal numbers
        float temperature = 36.6f;        // Note the f suffix. Required for float literals.
        double pi = 3.141592653589793;    // No suffix needed. Decimal literals are double by default.

        // Character
        char grade = 'A';                 // Single quotes for char. Double quotes for String.
        char banglaLetter = '\u0985';     // Unicode escape for the Bangla letter 'অ'

        // Boolean
        boolean isStudent = true;
        boolean hasGraduated = false;

        // Printing everything
        System.out.println("Age: " + age);
        System.out.println("Year: " + year);
        System.out.println("BD Population: " + population);
        System.out.println("World Population: " + worldPopulation);
        System.out.println("Temperature: " + temperature);
        System.out.println("Pi: " + pi);
        System.out.println("Grade: " + grade);
        System.out.println("Bangla Letter: " + banglaLetter);
        System.out.println("Is Student: " + isStudent);
    }
}
```

**Output:**
```
Age: 22
Year: 2025
BD Population: 170000000
World Population: 8000000000
Temperature: 36.6
Pi: 3.141592653589793
Grade: A
Bangla Letter: অ
Is Student: true
```

### Example 2: Type Casting in Action

```java
public class CastingDemo {
    public static void main(String[] args) {
        // Implicit casting (safe, automatic)
        int smallNumber = 42;
        double convertedToDouble = smallNumber;
        System.out.println("int to double: " + convertedToDouble);  // 42.0

        // Explicit casting (manual, potential data loss)
        double precisePrice = 99.99;
        int roundedPrice = (int) precisePrice;
        System.out.println("double to int: " + roundedPrice);  // 99 (decimal part is TRUNCATED, not rounded)

        // Overflow example
        int maxInt = 2_147_483_647;
        int overflowed = maxInt + 1;
        System.out.println("Max int + 1: " + overflowed);  // -2147483648 (wraps around to negative!)

        // char to int and back
        char letter = 'A';
        int asciiValue = letter;  // Implicit: char to int
        System.out.println("ASCII of A: " + asciiValue);  // 65

        char backToChar = (char) 66;  // Explicit: int to char
        System.out.println("Character 66: " + backToChar);  // B
    }
}
```

**Output:**
```
int to double: 42.0
double to int: 99
Max int + 1: -2147483648
ASCII of A: 65
Character 66: B
```

### Example 3: Using `var` and `final`

```java
public class VarAndFinalDemo {
    public static void main(String[] args) {
        // var: compiler infers the type
        var greeting = "Hello, Backend!";  // String
        var count = 10;                    // int
        var rate = 0.05;                   // double

        System.out.println(greeting);
        System.out.println("Count: " + count);
        System.out.println("Rate: " + rate);

        // var type is fixed. This would cause a compilation error:
        // count = "ten";  // ERROR: incompatible types

        // final: value cannot change
        final String APP_NAME = "OrderService";
        final int HTTP_OK = 200;
        final double PI = 3.14159;

        System.out.println("App: " + APP_NAME);
        System.out.println("HTTP OK: " + HTTP_OK);

        // This would cause a compilation error:
        // HTTP_OK = 201;  // ERROR: cannot assign a value to final variable
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Variables and data types are everywhere in backend code. Here are three realistic scenarios showing how type choices affect real systems.

### Scenario 1: A JPA Entity representing a database table

In Spring Boot, you map Java classes to database tables using JPA annotations. The data types you choose for your fields directly determine the database column types.

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
    // Long, not int. Database primary keys can exceed 2 billion over time.
    // Using int for an ID column is a common junior mistake that causes
    // catastrophic failures when the table grows beyond 2.1 billion rows.

    @Column(nullable = false, length = 50)
    private String orderNumber;
    // String for alphanumeric identifiers like "ORD-2025-00123"

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal totalAmount;
    // BigDecimal, NOT double. This is an e-commerce order.
    // Using double for money causes rounding errors.
    // Imagine charging a customer 99.99999999999997 instead of 100.00.
    // precision=10 means up to 10 digits total, scale=2 means 2 decimal places.

    @Column(nullable = false)
    private Integer quantity;
    // Integer (wrapper type) instead of int (primitive).
    // Wrapper types can be null, which is important for database columns
    // that allow NULL values. Primitives cannot be null.

    @Column(nullable = false)
    private boolean isPaid;
    // boolean for a simple yes/no flag.
    // Maps to a BOOLEAN or TINYINT column in the database.

    @Column(nullable = false)
    private LocalDateTime createdAt;
    // LocalDateTime for timestamps. Never use java.util.Date in modern Java.
    // The java.time API (Java 8+) is thread-safe and much more reliable.

    private String customerEmail;
    // No @Column annotation means JPA uses defaults.
    // Nullable by default. VARCHAR(255) by default.
}
```

**What to notice:**

- `Long` for IDs, not `int`. This is a rule you should follow in every backend project.
- `BigDecimal` for money, never `double`. This is non-negotiable in production systems.
- `Integer` (wrapper) vs `int` (primitive). Wrapper types support `null`, which maps to SQL `NULL`. Primitives always have a default value (0 for int) and cannot represent missing data.
- `LocalDateTime` for timestamps. The old `java.util.Date` class is mutable, not thread-safe, and poorly designed. Modern Java backends exclusively use the `java.time` package.

### Scenario 2: Constants in a configuration class

Real backend applications define constants for configuration values, HTTP status codes, and business rules.

```java
package com.company.orderservice.config;

public final class AppConstants {
    // Private constructor prevents instantiation.
    // This class only holds constants, so creating objects of it makes no sense.
    private AppConstants() {}

    // Pagination defaults
    public static final int DEFAULT_PAGE_SIZE = 20;
    public static final int MAX_PAGE_SIZE = 100;

    // Business rules
    public static final double MIN_ORDER_AMOUNT = 50.0;
    public static final int MAX_ITEMS_PER_ORDER = 500;
    public static final int ORDER_EXPIRY_MINUTES = 30;

    // API paths
    public static final String API_BASE_PATH = "/api/v1";
    public static final String ORDERS_PATH = API_BASE_PATH + "/orders";

    // Rate limiting
    public static final int MAX_REQUESTS_PER_MINUTE = 60;
}
```

**What to notice:**

- Every constant is `public static final`. `public` so other classes can access it. `static` so it belongs to the class, not to an instance. `final` so it cannot be changed.
- The class itself is `final` and has a private constructor. This is a standard Java pattern for utility/constant classes.
- Constants use UPPER_SNAKE_CASE. This is a universal Java convention that makes constants immediately recognizable in code.

### Scenario 3: Processing user input in a service method

```java
package com.company.orderservice.service;

import java.math.BigDecimal;
import java.math.RoundingMode;

public class PricingService {

    private static final BigDecimal TAX_RATE = new BigDecimal("0.15");
    private static final BigDecimal DISCOUNT_RATE = new BigDecimal("0.10");

    public BigDecimal calculateFinalPrice(BigDecimal basePrice, int quantity, boolean applyDiscount) {
        // Validate input
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive. Received: " + quantity);
        }

        // Calculate subtotal
        BigDecimal subtotal = basePrice.multiply(new BigDecimal(quantity));

        // Apply discount if eligible
        if (applyDiscount) {
            BigDecimal discount = subtotal.multiply(DISCOUNT_RATE);
            subtotal = subtotal.subtract(discount);
        }

        // Apply tax
        BigDecimal tax = subtotal.multiply(TAX_RATE);
        BigDecimal total = subtotal.add(tax);

        // Round to 2 decimal places (standard for currency)
        return total.setScale(2, RoundingMode.HALF_UP);
    }
}
```

**What to notice:**

- The method takes `BigDecimal` for price and `int` for quantity. The type choices reflect the real-world nature of the data.
- `boolean applyDiscount` is a clean flag. No need for a String like "yes"/"no" or an int like 0/1.
- Notice `new BigDecimal("0.15")` uses a String constructor, not `new BigDecimal(0.15)`. The double constructor introduces floating-point imprecision. This is a subtle but critical detail in financial backend systems.
- `setScale(2, RoundingMode.HALF_UP)` ensures the final price always has exactly two decimal places, which is what payment gateways expect.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using `double` for money

**Wrong:**
```java
double price = 19.99;
double quantity = 3;
double total = price * quantity;
System.out.println(total);  // 59.970000000000006 (not 59.97!)
```

**Right:**
```java
BigDecimal price = new BigDecimal("19.99");
BigDecimal quantity = new BigDecimal("3");
BigDecimal total = price.multiply(quantity);
System.out.println(total);  // 59.97 (exact)
```

**Why it is wrong:** Floating-point numbers cannot represent most decimal fractions exactly in binary. The error is tiny in a single calculation but accumulates across millions of transactions. A payment system that loses fractions of a cent per transaction will have significant discrepancies in its accounting over time.

### Mistake 2: Integer overflow in production counters

**Wrong:**
```java
int viewCount = 2_147_483_647;  // Maximum int value
viewCount++;  // Silently wraps to -2,147,483,648
System.out.println(viewCount);  // -2147483648
```

**Right:**
```java
long viewCount = 2_147_483_647L;
viewCount++;  // 2147483648. No overflow.
System.out.println(viewCount);  // 2147483648
```

**Why it is wrong:** Java does not throw an error on integer overflow. It silently wraps around. If your backend tracks page views, order counts, or user IDs with `int`, your system will break when the count exceeds 2.1 billion. Use `long` for any counter that could grow large over the lifetime of your application.

### Mistake 3: Confusing `char` and `String`

**Wrong:**
```java
char letter = "A";   // COMPILATION ERROR. Double quotes create a String, not a char.
String word = 'Hello'; // COMPILATION ERROR. Single quotes are only for single characters.
```

**Right:**
```java
char letter = 'A';     // Single quotes for a single character
String word = "Hello"; // Double quotes for a sequence of characters
```

**Why it is wrong:** `char` and `String` are fundamentally different types in Java. `char` is a primitive that holds exactly one Unicode character. `String` is a reference type that holds a sequence of characters. They are not interchangeable.

### Mistake 4: Forgetting the `L` suffix for long literals

**Wrong:**
```java
long timestamp = 1720000000000;  // COMPILATION ERROR.
// The literal 1720000000000 exceeds int range, and Java treats number literals as int by default.
```

**Right:**
```java
long timestamp = 1720000000000L;  // The L suffix tells Java this is a long literal.
```

**Why it is wrong:** Java interprets numeric literals without a decimal point as `int` by default. If the number is too large for `int`, the compiler rejects it even if you are assigning it to a `long` variable. The `L` suffix resolves this.

---

## Key Takeaways

> [!tip] Remember these points
> 1. Java has 8 primitive types (`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`) and an unlimited number of reference types (`String`, arrays, objects).
> 2. Use `int` for general whole numbers, `long` for IDs and large counters, `double` for general decimals, and `BigDecimal` for money. Never use `double` for financial calculations.
> 3. Primitive variables store values directly on the stack. Reference variables store a memory address on the stack that points to the actual object on the heap.
> 4. Implicit casting (small to large) is automatic and safe. Explicit casting (large to small) requires `(type)` syntax and can lose data.
> 5. Use `final` for constants and follow UPPER_SNAKE_CASE naming. Use `var` sparingly and only when the type is obvious from context.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Type Identification (Easy)
Declare one variable of each of the 8 primitive types and one `String` variable. Assign meaningful values related to a university student profile (name, age, CGPA, student ID, enrollment year, is active, blood group initial, semester number). Print all of them.

**Hint:** Use `long` for the student ID (they can be large numbers). Use `double` for CGPA. Use `char` for blood group initial.

### Exercise 2: Casting and Overflow (Medium)
Write a program that demonstrates the following:
1. Assign an `int` value to a `double` variable (implicit).
2. Assign a `double` value to an `int` variable (explicit). Print the result and explain in a comment what happened to the decimal part.
3. Add 1 to `Integer.MAX_VALUE` and print the result. Explain in a comment why the result is negative.

**Hint:** `Integer.MAX_VALUE` is a built-in constant that gives you the maximum value an `int` can hold.

### Exercise 3: BigDecimal Money Calculator (Medium)
Write a program that calculates the total cost of ordering items from a restaurant. Use `BigDecimal` for all monetary values. The program should:
1. Define a unit price (e.g., 149.50 BDT for a plate of biryani).
2. Define a quantity (e.g., 3).
3. Calculate the subtotal.
4. Add 15% VAT.
5. Print the final amount rounded to 2 decimal places.

**Hint:** Use `new BigDecimal("149.50")` with a String argument, not a double argument. Use `.multiply()`, `.add()`, and `.setScale()`.

### Exercise 4: Debug This Code (Hard, Optional)
The following code has three bugs related to data types. Find and fix all three. Explain each fix in a comment.

```java
public class BuggyCode {
    public static void main(String[] args) {
        int userId = 3_000_000_000;
        double price = 99.99;
        double total = price * 3;
        System.out.println("Total: " + total);

        char initial = "S";
        boolean isVerified = 1;
    }
}
```

**Hint:** One bug is a range issue, one is a type mismatch with quotes, and one is a type mismatch with a number.

### Solution
**For Exercise 1:**
```java
public class StudentProfile {
    public static void main(String[] args) {
        byte semester = 6;
        short enrollmentYear = 2023;
        int rollNumber = 230145;
        long studentId = 2023140045L;
        float attendance = 85.5f;
        double cgpa = 3.72;
        char bloodGroup = 'O';
        boolean isActive = true;
        String name = "Abdullah Al Sayb Saad";

        System.out.println("Name: " + name);
        System.out.println("Student ID: " + studentId);
        System.out.println("Roll: " + rollNumber);
        System.out.println("Semester: " + semester);
        System.out.println("Enrollment Year: " + enrollmentYear);
        System.out.println("CGPA: " + cgpa);
        System.out.println("Attendance: " + attendance + "%");
        System.out.println("Blood Group: " + bloodGroup);
        System.out.println("Active: " + isActive);
    }
}
```


**For Exercise 4:**
```java
public class BuggyCode {
    public static void main(String[] args) {
        // Bug 1: 3 billion exceeds int range (max ~2.1 billion). Use long.
        long userId = 3_000_000_000L;

        double price = 99.99;
        double total = price * 3;
        System.out.println("Total: " + total);

        // Bug 2: Double quotes create a String, not a char. Use single quotes.
        char initial = 'S';

        // Bug 3: boolean cannot hold an int. Use true or false.
        boolean isVerified = true;
    }
}
```

---

## Related Notes

- [[Java - Setting Up Environment - IntelliJ and JDK]]
- [[Java - Operators - Arithmetic Relational Logical]]

---

## Resources

- [Oracle Java Tutorials: Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html) - Official Oracle documentation on all primitive types.
- [Baeldung: Java BigDecimal Guide](https://www.baeldung.com/java-bigdecimal) - Comprehensive guide on why and how to use BigDecimal for financial calculations.
- [Baeldung: Java Primitive vs Wrapper Types](https://www.baeldung.com/java-primitives-vs-objects) - Deep dive into the difference between int and Integer, double and Double, etc.
