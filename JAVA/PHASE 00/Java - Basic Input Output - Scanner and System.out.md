---
title: "Java - Basic Input Output - Scanner and System.out"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - io
  - scanner
status: "not-started"
---

# Java - Basic Input Output - Scanner and System.out

> [!abstract] Overview
> Input and Output (I/O) is how a program communicates with the outside world. In Java, `System.out` handles standard output, `System.in` handles standard input, and the `Scanner` class provides a convenient way to parse user input from the console. While backend engineers rarely use `Scanner` in production code (backend systems receive input through HTTP requests, not console prompts), understanding Java's I/O model is essential because the same stream-based architecture underpins file reading, network communication, logging, and every data pipeline you will build.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]
> - [[Java - Strings and String Methods]]
> - [[Java - Methods - Parameters Return Types Overloading]]

---

## Theory

### What is I/O?

I/O stands for Input/Output. It is the mechanism by which a program reads data from external sources (input) and writes data to external destinations (output). External sources and destinations include:

- **Console/terminal**: The keyboard and screen (what we cover in this note)
- **Files**: Reading from and writing to files on disk
- **Network**: Sending and receiving data over the internet (HTTP requests, database connections, message queues)
- **Memory**: Reading from and writing to in-memory buffers

In Java, all I/O is built on the concept of **streams**. A stream is an abstraction that represents a flow of data. Think of it like a pipe: data flows into one end and out the other. Java provides two fundamental stream hierarchies:

- **Byte streams** (`InputStream`, `OutputStream`): Handle raw binary data, one byte at a time. Used for images, audio, video, and any non-text data.
- **Character streams** (`Reader`, `Writer`): Handle text data, one character at a time. Used for text files, JSON, XML, and console I/O. Character streams handle encoding (UTF-8, UTF-16, etc.) automatically.

`System.in` is an `InputStream` (byte stream). `Scanner` wraps `System.in` and adds the ability to parse text into Java types. `System.out` is a `PrintStream` (a specialized byte stream that can also print text).

### The `System` Class

The `System` class in `java.lang` provides access to three standard streams that are available to every Java program from the moment it starts:

| Stream | Type | Default Destination | Purpose |
|--------|------|-------------------|--------|
| `System.in` | `InputStream` | Keyboard | Standard input. Where the program reads data from. |
| `System.out` | `PrintStream` | Console/terminal | Standard output. Where the program writes normal output. |
| `System.err` | `PrintStream` | Console/terminal | Standard error. Where the program writes error messages. |

Although `System.out` and `System.err` both print to the console by default, they are separate streams. This separation matters in production backend systems where standard output and standard error are redirected to different log files. For example, a Linux server might redirect `stdout` to `app.log` and `stderr` to `error.log`, allowing the operations team to monitor errors separately from normal activity.

### `System.out` Methods

`System.out` is a `PrintStream` object that provides several methods for writing output:

- **`print(Object)`**: Prints the string representation of the argument without a newline.
- **`println(Object)`**: Prints the string representation followed by a newline character.
- **`printf(String format, Object... args)`**: Prints formatted text using format specifiers, similar to C's `printf`.
- **`format(String format, Object... args)`**: Identical to `printf`. Exists for readability.

**Format specifiers for `printf`:**

| Specifier | Type | Example | Output |
|-----------|------|---------|--------|
| `%d` | Integer | `printf("%d", 42)` | `42` |
| `%f` | Floating point | `printf("%.2f", 3.14159)` | `3.14` |
| `%s` | String | `printf("%s", "Hello")` | `Hello` |
| `%c` | Character | `printf("%c", 'A')` | `A` |
| `%b` | Boolean | `printf("%b", true)` | `true` |
| `%n` | Newline | `printf("Line1%nLine2")` | Two lines |
| `%10d` | Padded integer | `printf("%10d", 42)` | `        42` |
| `%-10s` | Left-aligned string | `printf("%-10s", "Hi")` | `Hi        ` |
| `%,d` | Comma-separated | `printf("%,d", 1000000)` | `1,000,000` |

### The `Scanner` Class

The `Scanner` class in `java.util` is a text parser that can read input from various sources (console, files, strings) and convert the text into Java primitive types and Strings.

**How Scanner works internally:**

1. Scanner reads raw bytes from the input source (e.g., `System.in`).
2. It decodes the bytes into characters using a character encoding (UTF-8 by default).
3. It splits the character stream into **tokens** using a delimiter pattern. The default delimiter is whitespace (spaces, tabs, newlines).
4. When you call a method like `nextInt()` or `nextDouble()`, Scanner attempts to parse the next token into the requested type. If the token cannot be parsed (e.g., calling `nextInt()` when the next token is "hello"), it throws an `InputMismatchException`.

**Key Scanner methods:**

| Method | Returns | Reads |
|--------|---------|-------|
| `next()` | `String` | The next whitespace-delimited token |
| `nextLine()` | `String` | The entire remaining line (up to the newline character) |
| `nextInt()` | `int` | The next token parsed as an integer |
| `nextLong()` | `long` | The next token parsed as a long |
| `nextDouble()` | `double` | The next token parsed as a double |
| `nextBoolean()` | `boolean` | The next token parsed as a boolean |
| `hasNext()` | `boolean` | True if there is another token available |
| `hasNextInt()` | `boolean` | True if the next token can be parsed as an int |
| `hasNextLine()` | `boolean` | True if there is another line available |
| `useDelimiter(String)` | `Scanner` | Changes the delimiter pattern |
| `close()` | `void` | Closes the scanner and the underlying input source |

### The `nextLine()` Trap

The most common Scanner bug involves mixing `nextInt()` (or `nextDouble()`, etc.) with `nextLine()`. Here is why it happens:

When you type `22` and press Enter, the input stream contains `"22\n"`. The `nextInt()` method reads the `22` but leaves the `\n` (newline character) in the stream. If you immediately call `nextLine()`, it reads the leftover `\n` and returns an empty string, skipping the line you actually wanted to read.

```java
Scanner scanner = new Scanner(System.in);

System.out.print("Enter your age: ");
int age = scanner.nextInt();  // Reads "22", leaves "\n" in the stream

System.out.print("Enter your name: ");
String name = scanner.nextLine();  // Reads the leftover "\n", returns ""
// The user never gets a chance to type their name!
```

The fix is to add an extra `scanner.nextLine()` after `nextInt()` to consume the leftover newline:

```java
int age = scanner.nextInt();
scanner.nextLine();  // Consume the leftover newline
String name = scanner.nextLine();  // Now this reads the actual name
```

> [!tip] Key Insight
> Scanner is a teaching tool, not a production tool. In real backend systems, you will never use Scanner to read user input because backend systems do not interact with users through a console. Input comes from HTTP request bodies (parsed by Spring Boot into Java objects), database queries (returned as result sets), message queues (deserialized from JSON), and configuration files (loaded by Spring's property system). However, the stream-based model that Scanner uses is the same model that underpins all of these production I/O mechanisms. Understanding Scanner helps you understand how Spring reads request bodies, how JDBC reads database rows, and how Kafka reads messages.

---

## Syntax and Basic Examples

### Example 1: System.out methods

```java
public class OutputDemo {
    public static void main(String[] args) {
        // print vs println
        System.out.print("Hello ");     // No newline
        System.out.print("World");      // Continues on the same line
        System.out.println("!");        // Adds newline
        System.out.println("New line"); // Starts on a new line

        // println with different types
        System.out.println(42);          // int
        System.out.println(3.14);        // double
        System.out.println(true);        // boolean
        System.out.println('A');         // char
        System.out.println((Object) null); // null (prints "null", does not crash)

        // printf / format
        String name = "Saad";
        int age = 22;
        double cgpa = 3.72;

        System.out.printf("Name: %s, Age: %d, CGPA: %.2f%n", name, age, cgpa);
        // Name: Saad, Age: 22, CGPA: 3.72

        // Formatting numbers
        long population = 170_000_000L;
        double price = 1234.5;
        System.out.printf("Population: %,d%n", population);   // 170,000,000
        System.out.printf("Price: %,.2f BDT%n", price);       // 1,234.50 BDT

        // Alignment
        System.out.printf("%-15s %10s %10s%n", "Item", "Qty", "Price");
        System.out.printf("%-15s %10d %10.2f%n", "Laptop", 1, 85000.00);
        System.out.printf("%-15s %10d %10.2f%n", "Mouse", 2, 1500.50);
        System.out.printf("%-15s %10d %10.2f%n", "Keyboard", 1, 3200.00);
    }
}
```

**Output:**

```text
Hello World!
New line
42
3.14
true
A
null
Name: Saad, Age: 22, CGPA: 3.72
Population: 170,000,000
Price: 1,234.50 BDT
Item                   Qty      Price
Laptop                   1   85000.00
Mouse                    2    1500.50
Keyboard                 1    3200.00
```

### Example 2: Basic Scanner input

```java
import java.util.Scanner;

public class InputDemo {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter your name: ");
        String name = scanner.nextLine();

        System.out.print("Enter your age: ");
        int age = scanner.nextInt();

        System.out.print("Enter your CGPA: ");
        double cgpa = scanner.nextDouble();

        System.out.print("Are you a CSE student? (true/false): ");
        boolean isCse = scanner.nextBoolean();

        System.out.println("\n--- Student Profile ---");
        System.out.printf("Name: %s%n", name);
        System.out.printf("Age: %d%n", age);
        System.out.printf("CGPA: %.2f%n", cgpa);
        System.out.printf("CSE: %b%n", isCse);

        scanner.close();
    }
}
```

### Example 3: The nextLine() trap and its fix

```java
import java.util.Scanner;

public class NextLineTrap {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // --- WRONG WAY ---
        System.out.print("Enter your age: ");
        int age = scanner.nextInt();
        // The newline character from pressing Enter is still in the stream!

        System.out.print("Enter your name: ");
        String name = scanner.nextLine();
        // This reads the leftover newline, NOT what the user types next.
        System.out.println("Name is: '" + name + "'");  // Prints empty string!

        // --- CORRECT WAY ---
        // Create a new scanner to reset (or just add an extra nextLine)
        Scanner scanner2 = new Scanner(System.in);

        System.out.print("Enter your age: ");
        int age2 = scanner2.nextInt();
        scanner2.nextLine();  // Consume the leftover newline

        System.out.print("Enter your name: ");
        String name2 = scanner2.nextLine();  // Now reads correctly
        System.out.println("Name is: '" + name2 + "'");

        // --- BEST WAY: Read everything as lines, then parse ---
        Scanner scanner3 = new Scanner(System.in);

        System.out.print("Enter your age: ");
        int age3 = Integer.parseInt(scanner3.nextLine().strip());

        System.out.print("Enter your CGPA: ");
        double cgpa3 = Double.parseDouble(scanner3.nextLine().strip());

        System.out.println("Age: " + age3 + ", CGPA: " + cgpa3);
    }
}
```

### Example 4: Input validation with Scanner

```java
import java.util.Scanner;

public class ValidatedInput {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // Validate integer input
        int age = 0;
        while (true) {
            System.out.print("Enter your age (1-120): ");
            if (scanner.hasNextInt()) {
                age = scanner.nextInt();
                scanner.nextLine();  // Consume newline
                if (age >= 1 && age <= 120) {
                    break;  // Valid input, exit the loop
                }
                System.out.println("Age must be between 1 and 120.");
            } else {
                System.out.println("Invalid input. Please enter a number.");
                scanner.nextLine();  // Consume the invalid token
            }
        }

        System.out.println("Your age: " + age);
        scanner.close();
    }
}
```

### Example 5: Reading from a String instead of the console

```java
import java.util.Scanner;

public class StringScanner {
    public static void main(String[] args) {
        // Scanner can parse any text source, not just console input.
        // This is closer to how backend systems process incoming data.
        String csvData = "Saad,22,3.72,true";

        Scanner scanner = new Scanner(csvData);
        scanner.useDelimiter(",");  // Change delimiter from whitespace to comma

        String name = scanner.next();
        int age = scanner.nextInt();
        double cgpa = scanner.nextDouble();
        boolean isActive = scanner.nextBoolean();

        System.out.printf("Name: %s, Age: %d, CGPA: %.2f, Active: %b%n",
            name, age, cgpa, isActive);

        scanner.close();
    }
}
```

**Output:**

```text
Name: Saad, Age: 22, CGPA: 3.72, Active: true
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Backend engineers do not use Scanner or System.out.println() in production code. However, the concepts behind them (streams, parsing, formatted output) appear everywhere. Here are three realistic scenarios.

### Scenario 1: Logging replaces System.out in production

In a real Spring Boot backend, you never use `System.out.println()` for output. You use a logging framework like SLF4J with Logback. Logging provides log levels, timestamps, thread names, and the ability to redirect output to files, databases, or monitoring systems.

```java
package com.company.orderservice.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class OrderService {

    // This is the production equivalent of System.out.
    // Every class that needs to produce output creates a logger.
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);

    public Order createOrder(Long userId, List<OrderItem> items) {
        // Instead of System.out.println("Creating order..."):
        logger.info("Creating order for user {} with {} items", userId, items.size());
        // {} is a placeholder, similar to %s in printf but more efficient
        // because the string is only constructed if the log level is enabled.

        try {
            Order order = orderRepository.save(buildOrder(userId, items));

            logger.info("Order {} created successfully. Total: {} BDT",
                order.getOrderNumber(), order.getTotal());

            return order;

        } catch (Exception e) {
            // Instead of System.err.println("Error: " + e):
            logger.error("Failed to create order for user {}. Error: {}",
                userId, e.getMessage(), e);
            // The last argument 'e' prints the full stack trace.
            throw new OrderCreationException("Order creation failed", e);
        }
    }

    public void processBatchOrders(List<Order> orders) {
        logger.debug("Starting batch processing of {} orders", orders.size());
        // DEBUG level is only visible during development.
        // In production, the log level is set to INFO or WARN,
        // so this line produces no output and no performance cost.

        int successCount = 0;
        int failCount = 0;

        for (Order order : orders) {
            try {
                processSingleOrder(order);
                successCount++;
            } catch (Exception e) {
                failCount++;
                logger.warn("Order {} failed: {}", order.getOrderNumber(), e.getMessage());
                // WARN level indicates something unexpected but not critical.
            }
        }

        logger.info("Batch complete. Success: {}, Failed: {}", successCount, failCount);
    }
}
```

**What to notice:**

- `logger.info()` replaces `System.out.println()`. `logger.error()` replaces `System.err.println()`. The logging framework handles timestamps, formatting, and output destinations automatically.
- Log levels (DEBUG, INFO, WARN, ERROR) let you control verbosity without changing code. In development, you see everything. In production, you only see warnings and errors. This is impossible with `System.out.println()`.
- The `{}` placeholder syntax is more efficient than `String.format()` or string concatenation because the message string is only constructed if the log level is enabled. If DEBUG is disabled, `logger.debug("Processing {}", expensiveComputation())` still evaluates the argument. To avoid this, use `logger.isDebugEnabled()` checks or lambda suppliers.

### Scenario 2: Reading request bodies replaces Scanner input

In a backend API, input comes from HTTP request bodies, not from the console. Spring Boot automatically deserializes JSON into Java objects, which is conceptually similar to how Scanner parses text into types.

```java
package com.company.orderservice.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    // Instead of reading input with Scanner:
    //   String name = scanner.nextLine();
    //   int qty = scanner.nextInt();
    //
    // Spring Boot reads the HTTP request body and deserializes it
    // into a Java object automatically. This is the backend equivalent
    // of Scanner parsing console input into typed variables.

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @RequestBody CreateOrderRequest request) {
        // The @RequestBody annotation tells Spring to:
        // 1. Read the raw bytes from the HTTP request body (like System.in)
        // 2. Decode the bytes as UTF-8 text (like Scanner's character decoding)
        // 3. Parse the JSON text into a CreateOrderRequest object
        //    (like Scanner's nextInt(), nextDouble(), etc.)

        // The CreateOrderRequest class defines the expected structure:
        // {
        //   "userId": 12345,
        //   "items": [{"productId": 1, "quantity": 2}],
        //   "couponCode": "EID2025"
        // }
        // Spring maps "userId" to request.getUserId(), etc.

        Order order = orderService.createOrder(
            request.getUserId(),
            request.getItems(),
            request.getCouponCode()
        );

        return ResponseEntity.status(201).body(new OrderResponse(order));
    }
}

// The DTO class that defines the input structure.
// This is the backend equivalent of defining what types
// Scanner should parse from the input.
record CreateOrderRequest(
    Long userId,
    List<OrderItemRequest> items,
    String couponCode
) {}
```

**What to notice:**

- `@RequestBody` is the backend equivalent of Scanner. It reads raw input, parses it, and converts it into typed Java objects. The difference is that Scanner parses whitespace-delimited text from the console, while Spring parses JSON from an HTTP request body.
- The `CreateOrderRequest` record defines the expected input structure, similar to how you decide which Scanner methods to call (`nextInt()`, `nextLine()`, etc.) based on the expected input format.
- Validation happens automatically if you add `@Valid` and Jakarta Validation annotations to the DTO fields. This is the backend equivalent of the input validation loop we wrote with `hasNextInt()`.

### Scenario 3: Reading configuration files

Backend applications read configuration from files at startup. This is another form of input processing that uses the same stream-based model as Scanner.

```java
package com.company.orderservice.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.beans.factory.annotation.Value;
import jakarta.annotation.PostConstruct;
import java.io.InputStream;
import java.util.Properties;

@Configuration
public class AppConfig {

    // Spring reads application.yml or application.properties
    // and injects values into these fields.
    // This is the production equivalent of reading input from a file.

    @Value("${app.name}")
    private String appName;

    @Value("${app.version}")
    private String appVersion;

    @Value("${app.max-order-limit}")
    private int maxOrderLimit;

    @Value("${app.database.url}")
    private String databaseUrl;

    @PostConstruct
    public void logStartupInfo() {
        // In production, this uses a logger, not System.out.
        // Shown here to illustrate the connection.
        System.out.printf("Starting %s v%s%n", appName, appVersion);
        System.out.printf("Max order limit: %d%n", maxOrderLimit);
        System.out.printf("Database: %s%n", databaseUrl);
        // Output:
        // Starting OrderService v2.1.0
        // Max order limit: 1000
        // Database: jdbc:postgresql://localhost:5432/orders
    }

    // Manual properties file reading (rarely needed in Spring Boot,
    // but shows the stream-based I/O model)
    public Properties loadProperties(String filename) {
        Properties props = new Properties();
        try (InputStream input = getClass().getClassLoader()
                .getResourceAsStream(filename)) {
            // getResourceAsStream returns an InputStream (like System.in)
            // Properties.load() reads from the stream (like Scanner reads from System.in)
            if (input != null) {
                props.load(input);
            }
        } catch (Exception e) {
            throw new RuntimeException("Failed to load properties: " + filename, e);
        }
        return props;
    }
}
```

**What to notice:**

- `InputStream` is the same base class that `System.in` belongs to. Whether you are reading from the keyboard, a file, a network socket, or a classpath resource, the underlying mechanism is the same: a stream of bytes flowing into your program.
- The try-with-resources block (`try (InputStream input = ...)`) automatically closes the stream when the block exits, even if an exception occurs. This is the modern Java way to handle I/O resources and prevents resource leaks. You will use this pattern extensively when working with files, database connections, and HTTP clients.

---

## Java vs Python Comparison

> [!note] Cross-language perspective
> Both Java and Python provide console I/O, but Python's approach is significantly simpler due to its dynamic typing.

| Aspect | Java | Python |
|--------|------|--------|
| Output | `System.out.println("text")` | `print("text")` |
| Formatted output | `System.out.printf("%s %d", name, age)` | `print(f"{name} {age}")` |
| Input | `Scanner scanner = new Scanner(System.in); String s = scanner.nextLine();` | `s = input()` |
| Typed input | `int n = scanner.nextInt();` | `n = int(input())` |
| Input source | Scanner wraps any InputStream | `input()` only reads from stdin |
| Production logging | SLF4J + Logback | `logging` module |

```java
// Java: Reading typed input requires Scanner and explicit parsing
Scanner scanner = new Scanner(System.in);
System.out.print("Enter your name: ");
String name = scanner.nextLine();
System.out.print("Enter your age: ");
int age = Integer.parseInt(scanner.nextLine().strip());
System.out.printf("Hello %s, you are %d years old.%n", name, age);
scanner.close();
```

```python
# Python: input() always returns a string, cast as needed
name = input("Enter your name: ")
age = int(input("Enter your age: "))
print(f"Hello {name}, you are {age} years old.")
```

**Key difference:** Java's Scanner provides type-specific methods (`nextInt()`, `nextDouble()`) that parse and validate in one step. Python's `input()` always returns a string, and you must explicitly cast it. Java's approach catches type errors at parse time (throwing `InputMismatchException`), while Python's approach throws `ValueError` at cast time. Both achieve the same result, but Java's Scanner is more verbose.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: The nextLine() trap after nextInt()

**Wrong:**

```java
Scanner scanner = new Scanner(System.in);
System.out.print("Age: ");
int age = scanner.nextInt();
System.out.print("Name: ");
String name = scanner.nextLine();  // Reads the leftover newline, returns ""
System.out.println("Hello, " + name);  // Prints "Hello, " (empty name)
```

**Right:**

```java
Scanner scanner = new Scanner(System.in);
System.out.print("Age: ");
int age = scanner.nextInt();
scanner.nextLine();  // Consume the leftover newline
System.out.print("Name: ");
String name = scanner.nextLine();  // Now reads correctly
System.out.println("Hello, " + name);
```

**Why it is wrong:** `nextInt()` reads the integer token but does not consume the newline character that follows it. `nextLine()` reads everything up to the next newline, which in this case is the leftover newline from the previous input. The result is an empty string. This is the single most common Scanner bug and will waste hours of your debugging time if you do not understand it.

### Mistake 2: Not closing the Scanner

**Wrong:**

```java
Scanner scanner = new Scanner(System.in);
String name = scanner.nextLine();
// Scanner is never closed. The underlying stream remains open.
```

**Right:**

```java
// Option 1: Explicit close
Scanner scanner = new Scanner(System.in);
String name = scanner.nextLine();
scanner.close();

// Option 2: try-with-resources (preferred)
try (Scanner scanner = new Scanner(System.in)) {
    String name = scanner.nextLine();
}  // Scanner is automatically closed here
```

**Why it is wrong:** An open Scanner holds a reference to the underlying input stream. If you are reading from a file, the file handle remains open, which can prevent other programs from accessing the file and can lead to resource exhaustion if you open many files without closing them. For `System.in`, the impact is minimal, but it is a good habit to close resources. In production backend code, unclosed streams (database connections, HTTP clients, file handles) are a leading cause of memory leaks and server crashes.

### Mistake 3: Using System.out.println() in production backend code

**Wrong:**

```java
public Order createOrder(Long userId) {
    System.out.println("Creating order for user " + userId);  // Bad
    // ...
    System.out.println("Order created: " + order.getId());    // Bad
}
```

**Right:**

```java
private static final Logger logger = LoggerFactory.getLogger(OrderService.class);

public Order createOrder(Long userId) {
    logger.info("Creating order for user {}", userId);
    // ...
    logger.info("Order created: {}", order.getId());
}
```

**Why it is wrong:** `System.out.println()` has no log levels, no timestamps, no thread information, and no way to redirect output to different destinations. In a production backend processing thousands of requests per second, System.out output is unstructured, unsearchable, and impossible to filter. Logging frameworks provide all of these features and are a requirement for any production system.

### Mistake 4: Not validating input before parsing

**Wrong:**

```java
Scanner scanner = new Scanner(System.in);
System.out.print("Enter a number: ");
int number = scanner.nextInt();  // Crashes with InputMismatchException if user types "abc"
```

**Right:**

```java
Scanner scanner = new Scanner(System.in);
System.out.print("Enter a number: ");
if (scanner.hasNextInt()) {
    int number = scanner.nextInt();
    System.out.println("You entered: " + number);
} else {
    System.out.println("Invalid input. Expected a number.");
    scanner.nextLine();  // Consume the invalid token
}
```

**Why it is wrong:** Scanner's type-specific methods (`nextInt()`, `nextDouble()`) throw `InputMismatchException` if the next token cannot be parsed as the expected type. In a console application, this crashes the program. In a backend API, the equivalent is returning a 400 Bad Request response when the client sends invalid JSON. Always validate before parsing.

---

## Key Takeaways

> [!tip] Remember these points
> 1. `System.out` and `System.err` are `PrintStream` objects for standard output and error. Use `print()`, `println()`, and `printf()` for console output. In production backend code, replace all `System.out` calls with a logging framework (SLF4J + Logback).
> 2. Scanner wraps an input stream and provides methods to parse text into Java types: `nextInt()`, `nextDouble()`, `nextLine()`, `next()`, etc. Always check with `hasNextInt()` before calling `nextInt()` to avoid exceptions.
> 3. The `nextLine()` trap occurs when you mix `nextInt()` with `nextLine()`. The fix is to add an extra `scanner.nextLine()` after `nextInt()` to consume the leftover newline, or to read everything as lines and parse manually with `Integer.parseInt()`.
> 4. Java's I/O is built on streams. `System.in` is an `InputStream`. Scanner wraps any `InputStream` or `Readable`. This same stream model underpins file I/O, network communication, and every data pipeline in backend systems.
> 5. Always close I/O resources using try-with-resources or explicit `close()` calls. Unclosed streams cause resource leaks that can crash production servers.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Formatted Receipt (Easy)

Write a program that asks the user for their name, the number of items purchased, and the price per item. Use Scanner for input. Then print a formatted receipt using `printf` that looks like this:

```text
================================
        PURCHASE RECEIPT
================================
Customer:   Abdullah Al Sayb Saad
Items:      3
Unit Price: 1,250.00 BDT
Subtotal:   3,750.00 BDT
VAT (15%):    562.50 BDT
Total:      4,312.50 BDT
================================
```

Use proper formatting: comma-separated numbers, two decimal places, and aligned columns.

**Hint:** Use `%,.2f` for currency formatting and `%-15s` for left-aligned strings.

### Exercise 2: Input Validation Loop (Medium)

Write a program that asks the user to enter a positive integer between 1 and 100. The program should keep asking until the user enters a valid number. Handle the following cases:

- User enters a non-integer (e.g., `"abc"`): Print "Please enter a valid number."
- User enters a number outside the range (e.g., 150): Print "Number must be between 1 and 100."
- User enters a valid number: Print "Accepted: X" and exit the loop.

Use `hasNextInt()` for validation and `nextLine()` to consume invalid input.

**Hint:** Use a `while (true)` loop with a `break` statement when valid input is received. Remember to consume the newline after `nextInt()`.

### Exercise 3: CSV Line Parser (Medium)

Write a program that takes a CSV line as input from the user (using `nextLine()`) and parses it into individual fields. The CSV line may contain fields with spaces. Use a Scanner with a comma delimiter to parse the line.

Example input: `Saad,22,Computer Science,3.72`
Expected output:

```text
Field 1: Saad
Field 2: 22
Field 3: Computer Science
Field 4: 3.72
```

Then extend the program to handle multiple lines of CSV input until the user types "quit".

**Hint:** Create a new Scanner from the input string: `Scanner lineScanner = new Scanner(line); lineScanner.useDelimiter(",");`

### Exercise 4: Student Grade Processor (Hard, Optional)

Write a program that processes student grades from console input. The program should:

1. Ask how many students to process.
2. For each student, read their name (may contain spaces) and marks for 3 subjects.
3. Calculate the average for each student.
4. Print a formatted grade report showing all students with their averages and letter grades.
5. Print the class average, highest average, and lowest average.

Handle the `nextLine()` trap correctly when mixing `nextInt()` and `nextLine()`. Use `printf` for formatted output.

**Hint:** After reading the number of students with `nextInt()`, call `nextLine()` to consume the newline before reading student names.

### Solution
For Exercise 1:

```java
import java.util.Scanner;

public class Receipt {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter your name: ");
        String name = scanner.nextLine();

        System.out.print("Number of items: ");
        int items = Integer.parseInt(scanner.nextLine().strip());

        System.out.print("Price per item: ");
        double price = Double.parseDouble(scanner.nextLine().strip());

        double subtotal = items * price;
        double vat = subtotal * 0.15;
        double total = subtotal + vat;

        System.out.println("================================");
        System.out.println("        PURCHASE RECEIPT");
        System.out.println("================================");
        System.out.printf("Customer:   %s%n", name);
        System.out.printf("Items:      %d%n", items);
        System.out.printf("Unit Price: %,.2f BDT%n", price);
        System.out.printf("Subtotal:   %,.2f BDT%n", subtotal);
        System.out.printf("VAT (15%%):  %,.2f BDT%n", vat);
        System.out.printf("Total:      %,.2f BDT%n", total);
        System.out.println("================================");

        scanner.close();
    }
}
```

For Exercise 2:

```java
import java.util.Scanner;

public class InputValidation {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int number = 0;

        while (true) {
            System.out.print("Enter a number (1-100): ");

            if (scanner.hasNextInt()) {
                number = scanner.nextInt();
                scanner.nextLine();  // Consume newline

                if (number >= 1 && number <= 100) {
                    System.out.println("Accepted: " + number);
                    break;
                } else {
                    System.out.println("Number must be between 1 and 100.");
                }
            } else {
                System.out.println("Please enter a valid number.");
                scanner.nextLine();  // Consume invalid token
            }
        }

        scanner.close();
    }
}
```

---

## Related Notes

- [[Java - Methods - Parameters Return Types Overloading]]
- [[Java - Debugging Basics in IntelliJ]]
- [[Java - File I-O - FileReader FileWriter BufferedReader]]

---

## Resources

- [Oracle Java Tutorials: I/O Streams](https://docs.oracle.com/javase/tutorial/essential/io/) - Official documentation on the Java I/O stream architecture.
- [Oracle Java Documentation: java.util.Scanner](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Scanner.html) - Complete API reference for all Scanner methods.
- [Oracle Java Tutorials: Formatting Numeric Print Output](https://docs.oracle.com/javase/tutorial/java/data/numberformat.html) - Guide to printf format specifiers.
- [Baeldung: Java Scanner Guide](https://www.baeldung.com/java-scanner) - Comprehensive guide covering Scanner usage, pitfalls, and alternatives.
- [Baeldung: Java Logging with SLF4J and Logback](https://www.baeldung.com/slf4j-with-log4j2-logback) - Introduction to production logging. Read this when you reach Phase 2.
