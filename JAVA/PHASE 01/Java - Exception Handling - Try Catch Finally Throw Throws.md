---
title: "Java - Exception Handling - Try Catch Finally Throw Throws"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - exceptions
  - error-handling
  - try-catch
status: "not-started"
---

# Java - Exception Handling - Try Catch Finally Throw Throws

> [!abstract] Overview
> Exception handling is Java's mechanism for dealing with errors and unexpected conditions that occur during program execution. Instead of crashing silently or returning error codes that callers might ignore, Java uses an object-oriented approach: errors are represented as **exception objects** that are **thrown** when something goes wrong and **caught** by code that knows how to handle them. In backend development, exception handling is critical for every layer of the application: controllers catch exceptions and return appropriate HTTP status codes, services throw business rule violations, repositories handle database errors, and global exception handlers translate exceptions into structured API responses.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Inheritance - Single Multilevel Hierarchical]]
> - [[Java - Polymorphism - Compile Time and Runtime]]
> - [[Java - this and super Keywords]]

---

## Theory

### What is an Exception?

An exception is an **object** that represents an error or unexpected condition that occurs during the execution of a program. When an error occurs, the JVM creates an exception object containing information about the error (the error message, the stack trace, and optionally the cause) and **throws** it. The normal flow of execution is interrupted, and the JVM searches for a **catch block** that can handle the exception. If a handler is found, execution resumes in the catch block. If no handler is found, the exception propagates up the call stack until it reaches the `main` method, at which point the program terminates and the JVM prints the stack trace to `System.err`.

The key insight is that exceptions are **objects**, not magic. They are instances of classes that extend `java.lang.Throwable`. You can create your own exception classes, throw them, catch them, and inspect them just like any other Java object.

### The Exception Hierarchy

Java's exception classes form a hierarchy rooted at `Throwable`. Understanding this hierarchy is essential because it determines how exceptions behave and how you should handle them.

```text
Throwable
├── Error (DO NOT catch these)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   ├── NoClassDefFoundError
│   └── ...
└── Exception
    ├── RuntimeException (unchecked exceptions)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── IllegalArgumentException
    │   ├── IllegalStateException
    │   ├── ClassCastException
    │   ├── ArithmeticException
    │   ├── NumberFormatException
    │   └── ...
    └── Checked exceptions (everything else)
        ├── IOException
        │   ├── FileNotFoundException
        │   └── ...
        ├── SQLException
        ├── InterruptedException
        ├── ParseException
        └── ...
```

**`Error`**: Represents serious problems that a reasonable application should not try to catch. These are JVM-level failures like running out of memory or a stack overflow. You cannot recover from these errors, so catching them is pointless and dangerous. Let the JVM handle them.

**`Exception`**: Represents conditions that a reasonable application might want to catch and handle. Divided into two subcategories:

**Checked exceptions** (compile-time exceptions): Any exception that extends `Exception` but NOT `RuntimeException`. The compiler **forces** you to handle these exceptions, either by catching them with a `try-catch` block or by declaring them in the method signature with `throws`. Examples: `IOException`, `SQLException`, `InterruptedException`.

**Unchecked exceptions** (runtime exceptions): Any exception that extends `RuntimeException`. The compiler does **not** force you to handle these. They represent programming errors (null references, invalid arguments, out-of-bounds access) that should be prevented by writing correct code, not caught and handled. Examples: `NullPointerException`, `IllegalArgumentException`, `ArrayIndexOutOfBoundsException`.

### Checked vs Unchecked Exceptions

This is one of the most debated topics in Java. Here is the practical guideline for backend development:

| Aspect | Checked Exceptions | Unchecked Exceptions |
|--------|-------------------|---------------------|
| Extends | `Exception` | `RuntimeException` |
| Compiler enforcement | Must be caught or declared | No enforcement |
| Represents | Recoverable external failures | Programming errors |
| Examples | `IOException`, `SQLException` | `NullPointerException`, `IllegalArgumentException` |
| Backend usage | Rare (Spring wraps most in unchecked) | Very common |
| Modern consensus | Overused in early Java; prefer unchecked | Preferred for most application errors |

**Why modern Spring Boot prefers unchecked exceptions:**

Spring Boot wraps almost all checked exceptions in unchecked wrappers. `JdbcTemplate` wraps `SQLException` in `DataAccessException` (a `RuntimeException`). `RestTemplate` wraps `IOException` in `ResourceAccessException`. This means you rarely need to write `try-catch` blocks for checked exceptions in Spring Boot. The framework handles the checked-to-unchecked conversion for you.

For your own application code, the recommendation is to extend `RuntimeException` for all custom exceptions. This keeps your method signatures clean and avoids forcing callers to write boilerplate `try-catch` blocks for errors they cannot meaningfully handle.

### The `try-catch` Block

The `try-catch` block is the primary mechanism for handling exceptions. Code that might throw an exception is placed inside the `try` block. If an exception occurs, the JVM jumps to the matching `catch` block.

```java
try {
    // Code that might throw an exception
    int result = 10 / 0;
} catch (ArithmeticException e) {
    // Code that handles the specific exception
    System.out.println("Cannot divide by zero: " + e.getMessage());
}
```

**Multiple catch blocks:**

You can catch different exception types in separate catch blocks. The JVM matches the exception to the first compatible catch block, so more specific exceptions must come before more general ones.

```java
try {
    String number = "abc";
    int[] arr = new int[3];
    arr[5] = Integer.parseInt(number);
} catch (NumberFormatException e) {
    System.out.println("Invalid number format");
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Array index out of bounds");
} catch (RuntimeException e) {
    System.out.println("Some other runtime error: " + e.getMessage());
}
```

**Multi-catch (Java 7+):**

When multiple exception types require the same handling logic, you can combine them in a single catch block using the `|` operator.

```java
try {
    // Code that might throw either exception
} catch (IOException | SQLException e) {
    // Same handling for both
    System.out.println("Data access error: " + e.getMessage());
}
```

### The `finally` Block

The `finally` block contains code that **always executes**, regardless of whether an exception was thrown or caught. It is used for cleanup operations like closing files, database connections, and network sockets.

```java
try {
    // Code that might throw an exception
    FileReader reader = new FileReader("data.txt");
    // ... read from the file ...
} catch (FileNotFoundException e) {
    System.out.println("File not found");
} finally {
    // This ALWAYS executes, even if an exception was thrown
    // and even if the catch block rethrows the exception.
    System.out.println("Cleanup complete");
}
```

**Execution order:**

1. If no exception: `try` -> `finally`
2. If exception caught: `try` -> `catch` -> `finally`
3. If exception not caught: `try` -> `finally` -> exception propagates up
4. If `return` in `try` or `catch`: `finally` executes BEFORE the method actually returns

> [!warning] Avoid `return` in `finally` blocks
> If you put a `return` statement in a `finally` block, it will override any `return` or exception from the `try` or `catch` blocks. This silently swallows exceptions and is almost always a bug.

### Try-with-Resources (Java 7+)

The try-with-resources statement automatically closes resources (objects that implement `AutoCloseable`) when the try block exits, whether normally or due to an exception. This eliminates the need for explicit `finally` blocks for resource cleanup.

```java
// Old way (before Java 7):
FileReader reader = null;
try {
    reader = new FileReader("data.txt");
    // ... read ...
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// Modern way (Java 7+):
try (FileReader reader = new FileReader("data.txt")) {
    // ... read ...
} catch (IOException e) {
    e.printStackTrace();
}
// reader is automatically closed here, even if an exception occurred
```

### The `throw` Keyword

The `throw` keyword is used to explicitly throw an exception object. You create the exception object with `new` and throw it with `throw`.

```java
public void setAge(int age) {
    if (age < 0 || age > 150) {
        throw new IllegalArgumentException("Age must be between 0 and 150. Received: " + age);
    }
    this.age = age;
}
```

When the JVM encounters a `throw` statement, it immediately stops executing the current method and begins searching for a matching `catch` block up the call stack.

### The `throws` Keyword

The `throws` keyword is used in a method signature to declare that the method might throw one or more checked exceptions. It is a contract that tells callers: "If you call this method, you must handle these exceptions."

```java
// This method declares that it might throw IOException.
// Callers must either catch it or declare it in their own throws clause.
public String readFile(String path) throws IOException {
    return Files.readString(Path.of(path));
}

// The caller handles the exception:
public void processFile() {
    try {
        String content = readFile("data.txt");
    } catch (IOException e) {
        System.out.println("Failed to read file: " + e.getMessage());
    }
}

// Or the caller propagates it:
public void processFile() throws IOException {
    String content = readFile("data.txt");  // No try-catch needed
}
```

**Key distinction:**

- `throw` (no 's'): Throws an exception **object** inside a method body. `throw new IllegalArgumentException("...")`.
- `throws` (with 's'): Declares exception **types** in a method signature. `public void method() throws IOException`.

### Custom Exceptions

In backend development, you should define custom exception classes for your application's specific error conditions. Custom exceptions make your error handling more expressive and allow the global exception handler to map different errors to different HTTP status codes.

**Guidelines for custom exceptions:**

1. Extend `RuntimeException` for most application errors (unchecked).
2. Extend a base application exception class for consistency.
3. Include an error code and HTTP status code in the base class.
4. Provide meaningful error messages that help with debugging.
5. Support exception chaining by accepting a `Throwable cause` parameter.

### How Exceptions Work Internally

When an exception is thrown, the JVM performs the following steps:

1. **Creates the exception object**: The `new` keyword allocates memory for the exception on the heap. The constructor captures the current stack trace by calling `Throwable.fillInStackTrace()`, which walks the call stack and records each frame.

2. **Searches for a handler**: The JVM walks the call stack from the current method upward, looking for a `catch` block whose exception type matches the thrown exception (using `instanceof` semantics). This is called **stack unwinding**.

3. **Executes the handler**: If a matching `catch` block is found, the JVM jumps to it. Any `finally` blocks between the throw point and the catch block are executed during the unwinding process.

4. **Terminates if unhandled**: If the exception reaches the top of the stack without being caught, the JVM calls the thread's `UncaughtExceptionHandler`, which by default prints the stack trace to `System.err` and terminates the thread.

**Performance note:** Creating an exception object is relatively expensive because `fillInStackTrace()` walks the entire call stack. In performance-critical code, avoid using exceptions for normal control flow. Use them for truly exceptional conditions.

> [!tip] Key Insight
> Exceptions are not just error reporting mechanisms. They are a **control flow** construct that allows you to separate the "happy path" (normal execution) from the "error path" (exception handling). Without exceptions, every method would need to check return codes after every call, cluttering the code with error-checking logic. Exceptions let you write clean business logic in the `try` block and handle all errors in one place in the `catch` block. In Spring Boot, the `@ControllerAdvice` global exception handler takes this to the extreme: your controller and service code throws exceptions freely, and a single centralized handler translates them into HTTP responses.

---

## Syntax and Basic Examples

### Example 1: Basic try-catch-finally

```java
public class ExceptionBasics {
    public static void main(String[] args) {
        // Example 1: Catching a specific exception
        try {
            int[] numbers = {1, 2, 3};
            System.out.println(numbers[5]);  // ArrayIndexOutOfBoundsException
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Caught: " + e.getMessage());
            System.out.println("Exception class: " + e.getClass().getSimpleName());
        }

        // Example 2: Multiple catch blocks (specific before general)
        try {
            String input = "abc";
            int number = Integer.parseInt(input);  // NumberFormatException
            int result = 10 / number;
        } catch (NumberFormatException e) {
            System.out.println("Invalid number: " + e.getMessage());
        } catch (ArithmeticException e) {
            System.out.println("Math error: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Unexpected error: " + e.getMessage());
        }

        // Example 3: finally block always executes
        try {
            System.out.println("In try block");
            int result = 10 / 0;
        } catch (ArithmeticException e) {
            System.out.println("In catch block");
            return;  // Even with return, finally executes!
        } finally {
            System.out.println("In finally block (always runs)");
        }
    }
}
```

**Output:**

```text
Caught: Index 5 out of bounds for length 3
Exception class: ArrayIndexOutOfBoundsException
Invalid number: For input string: "abc"
In try block
In catch block
In finally block (always runs)
```

### Example 2: throw and throws

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.nio.file.Files;

public class ValidationService {

    // 'throws' declares that this method might throw a checked exception.
    // Callers must handle it or propagate it.
    public String readConfigFile(String path) throws IOException {
        File file = new File(path);
        if (!file.exists()) {
            // 'throw' creates and throws an exception object
            throw new FileNotFoundException("Config file not found: " + path);
        }
        return Files.readString(file.toPath());
    }

    // Unchecked exception: no 'throws' declaration needed
    public void validateEmail(String email) {
        if (email == null || email.isBlank()) {
            throw new IllegalArgumentException("Email cannot be empty");
        }
        if (!email.contains("@") || !email.contains(".")) {
            throw new IllegalArgumentException("Invalid email format: " + email);
        }
    }

    public void validateAge(int age) {
        if (age < 0) {
            // Throwing with a cause (exception chaining)
            throw new IllegalArgumentException(
                "Age cannot be negative: " + age,
                new ArithmeticException("Negative value detected")
            );
        }
        if (age > 150) {
            throw new IllegalArgumentException("Age seems unrealistic: " + age);
        }
    }

    public static void main(String[] args) {
        ValidationService service = new ValidationService();

        // Handling the checked exception
        try {
            String config = service.readConfigFile("/nonexistent/path.yml");
        } catch (IOException e) {
            System.out.println("Config error: " + e.getMessage());
        }

        // Unchecked exceptions: no try-catch required, but good practice to validate
        try {
            service.validateEmail("invalid-email");
        } catch (IllegalArgumentException e) {
            System.out.println("Validation error: " + e.getMessage());
        }
    }
}
```

### Example 3: Try-with-resources

```java
import java.io.*;
import java.nio.file.*;

public class FileProcessor {

    // Try-with-resources: the BufferedReader is automatically closed
    // when the try block exits, even if an exception occurs.
    public int countLines(String filePath) {
        int lineCount = 0;

        try (BufferedReader reader = Files.newBufferedReader(Path.of(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                lineCount++;
            }
        } catch (NoSuchFileException e) {
            System.out.println("File not found: " + filePath);
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }
        // reader.close() is called automatically here

        return lineCount;
    }

    // Multiple resources in a single try-with-resources
    public void copyFile(String source, String destination) {
        try (
            BufferedReader reader = Files.newBufferedReader(Path.of(source));
            BufferedWriter writer = Files.newBufferedWriter(Path.of(destination))
        ) {
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }
            System.out.println("File copied successfully");
        } catch (IOException e) {
            System.out.println("Copy failed: " + e.getMessage());
        }
        // Both reader and writer are closed automatically, in reverse order
    }
}
```

### Example 4: Custom exception hierarchy

```java
// Base exception for the entire application
public class AppException extends RuntimeException {
    private final int httpStatus;
    private final String errorCode;

    public AppException(String message, int httpStatus, String errorCode) {
        super(message);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public AppException(String message, int httpStatus, String errorCode, Throwable cause) {
        super(message, cause);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public int getHttpStatus() { return httpStatus; }
    public String getErrorCode() { return errorCode; }
}
```

```java
// Specific exception: resource not found
public class ResourceNotFoundException extends AppException {
    public ResourceNotFoundException(String resource, Long id) {
        super(resource + " not found with id: " + id, 404, "RESOURCE_NOT_FOUND");
    }

    public ResourceNotFoundException(String resource, String field, String value) {
        super(resource + " not found with " + field + ": " + value, 404, "RESOURCE_NOT_FOUND");
    }
}
```

```java
// Specific exception: business rule violation
public class BusinessRuleException extends AppException {
    public BusinessRuleException(String message) {
        super(message, 422, "BUSINESS_RULE_VIOLATION");
    }
}
```

```java
// Specific exception: insufficient stock
public class InsufficientStockException extends BusinessRuleException {
    private final String productName;
    private final int requested;
    private final int available;

    public InsufficientStockException(String productName, int requested, int available) {
        super("Insufficient stock for '" + productName
            + "': requested " + requested + ", available " + available);
        this.productName = productName;
        this.requested = requested;
        this.available = available;
    }

    public String getProductName() { return productName; }
    public int getRequested() { return requested; }
    public int getAvailable() { return available; }
}
```

```java
// Usage in a service:
public class InventoryService {
    public void reserveStock(String productName, int quantity) {
        int available = getAvailableStock(productName);

        if (available < quantity) {
            throw new InsufficientStockException(productName, quantity, available);
            // This exception propagates up to the controller,
            // where the global exception handler converts it to an HTTP 422 response.
        }

        // Reserve the stock...
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Exception handling is one of the most important aspects of backend development. Here are three realistic scenarios from production Spring Boot applications.

### Scenario 1: Global exception handler with `@ControllerAdvice`

In a Spring Boot backend, you do not write `try-catch` blocks in every controller method. Instead, you define a global exception handler that catches all exceptions and converts them into structured HTTP responses.

```java
package com.company.orderservice.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.time.LocalDateTime;
import java.util.Map;
import java.util.stream.Collectors;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // Handle custom application exceptions
    @ExceptionHandler(AppException.class)
    public ResponseEntity<ErrorResponse> handleAppException(AppException ex) {
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            ex.getHttpStatus(),
            ex.getErrorCode(),
            ex.getMessage()
        );
        return ResponseEntity.status(ex.getHttpStatus()).body(error);
    }

    // Handle validation errors from @Valid annotations
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        // Extract field-level validation errors
        Map<String, String> fieldErrors = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(
                error -> error.getField(),
                error -> error.getDefaultMessage(),
                (existing, replacement) -> existing  // Keep first error per field
            ));

        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            400,
            "VALIDATION_ERROR",
            "Request validation failed",
            fieldErrors
        );
        return ResponseEntity.badRequest().body(error);
    }

    // Handle resource not found specifically
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            404,
            ex.getErrorCode(),
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    // Catch-all for unexpected exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        // Log the full exception for the development team
        // logger.error("Unhandled exception", ex);

        // Never expose internal details to the client
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            500,
            "INTERNAL_ERROR",
            "An unexpected error occurred. Please contact support."
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

```java
import java.time.LocalDateTime;
import java.util.Map;

// The ErrorResponse DTO
public record ErrorResponse(
    LocalDateTime timestamp,
    int status,
    String error,
    String message,
    Map<String, String> fieldErrors
) {
    // Overloaded constructor for errors without field details
    public ErrorResponse(LocalDateTime timestamp, int status, String error, String message) {
        this(timestamp, status, error, message, null);
    }
}
```

**What to notice:**

- The `@RestControllerAdvice` annotation makes this class a global exception handler for all controllers. Spring automatically routes any uncaught exception to the appropriate `@ExceptionHandler` method.
- The handler uses polymorphism: `@ExceptionHandler(AppException.class)` catches all subclasses of `AppException` (including `ResourceNotFoundException`, `BusinessRuleException`, etc.) unless a more specific handler exists.
- The catch-all `@ExceptionHandler(Exception.class)` ensures that no exception ever reaches the client as a raw stack trace. It returns a generic 500 error while logging the full details for the development team.
- The `MethodArgumentNotValidException` handler extracts field-level validation errors from Spring's `@Valid` annotation and returns them as a structured map. This allows the frontend to display specific error messages next to each form field.

### Scenario 2: Service layer exception throwing

In the service layer, you throw exceptions to signal business rule violations. The service does not catch its own exceptions; it lets them propagate to the global handler.

```java
package com.company.orderservice.service;

@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    public OrderService(OrderRepository orderRepository,
                        InventoryService inventoryService,
                        PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
    }

    public Order createOrder(CreateOrderRequest request) {
        // Validate: user exists
        User user = userRepository.findById(request.getUserId())
            .orElseThrow(() -> new ResourceNotFoundException("User", request.getUserId()));
            // The lambda () -> new ResourceNotFoundException(...) is called only
            // if the Optional is empty. This is lazy exception creation.

        // Validate: items are in stock (throws InsufficientStockException if not)
        inventoryService.reserveStock(request.getItems());

        // Create the order
        Order order = new Order(
            OrderUtils.generateOrderNumber(),
            user.getId(),
            calculateTotal(request.getItems())
        );
        Order savedOrder = orderRepository.save(order);

        // Process payment (throws PaymentException if it fails)
        try {
            paymentService.charge(savedOrder);
        } catch (PaymentException e) {
            // If payment fails, cancel the order and rethrow
            savedOrder.cancel();
            orderRepository.save(savedOrder);
            throw e;  // Rethrow the original exception with its stack trace intact
        }

        return savedOrder;
    }

    public Order cancelOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new ResourceNotFoundException("Order", orderId));

        // This method throws IllegalStateException if the order cannot be cancelled
        // (e.g., it has already been shipped). The exception propagates to the
        // global handler, which returns a 422 response.
        order.cancel();

        return orderRepository.save(order);
    }
}
```

**What to notice:**

- The service throws exceptions but does not catch them (except for the payment rollback scenario). This keeps the service code clean and focused on business logic. Error handling is centralized in the global exception handler.
- `orElseThrow(() -> new ResourceNotFoundException(...))` uses a lambda for lazy exception creation. The exception object is only created if the Optional is empty. This avoids the performance cost of creating an exception object (including the stack trace) when the resource is found.
- The payment failure scenario demonstrates **exception rethrowing**. When payment fails, the service cancels the order (cleanup) and then rethrows the original exception. The `throw e` statement preserves the original stack trace, so the global handler can log the full error chain.

### Scenario 3: Exception chaining in repository and integration layers

When a low-level exception occurs (e.g., a database error), you wrap it in a higher-level application exception that provides context while preserving the original cause.

```java
package com.company.orderservice.repository;

@Repository
public class OrderRepositoryImpl implements OrderRepository {

    private final JdbcTemplate jdbcTemplate;

    public OrderRepositoryImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public Order findById(Long id) {
        try {
            return jdbcTemplate.queryForObject(
                "SELECT * FROM orders WHERE id = ?",
                new OrderRowMapper(),
                id
            );
        } catch (EmptyResultDataAccessException e) {
            // Spring's JdbcTemplate throws this checked-like exception
            // when no row is found. We convert it to our custom exception.
            throw new ResourceNotFoundException("Order", id);
        } catch (DataAccessException e) {
            // Wrap the database exception in an application exception.
            // The original exception is passed as the 'cause', preserving
            // the full stack trace for debugging.
            throw new RepositoryException(
                "Failed to fetch order " + id + " from database",
                e  // Exception chaining: the original DataAccessException is the cause
            );
        }
    }
}
```

```java
package com.company.orderservice.integration;

import java.io.IOException;
import java.math.BigDecimal;

public class PaymentGatewayClient {

    public PaymentResult charge(BigDecimal amount, String customerId) {
        try {
            // Call the external payment API
            HttpResponse<String> response = httpClient.send(request, BodyHandlers.ofString());

            if (response.statusCode() == 402) {
                throw new InsufficientFundsException(amount, customerId);
            }
            if (response.statusCode() >= 500) {
                throw new PaymentGatewayException(
                    "Payment gateway returned " + response.statusCode()
                );
            }

            return parseResponse(response.body());

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();  // Restore the interrupt flag
            throw new PaymentProcessingException("Payment interrupted", e);
        } catch (IOException e) {
            // Wrap the I/O exception in a domain-specific exception.
            // The original IOException is preserved as the cause.
            throw new PaymentProcessingException(
                "Failed to connect to payment gateway", e
            );
        }
    }
}
```

**What to notice:**

- **Exception chaining** (`new PaymentProcessingException("message", e)`) preserves the full error history. When you print the stack trace, you will see both the `PaymentProcessingException` and the original `IOException` that caused it. This is invaluable for debugging production issues.
- The `InterruptedException` handling follows Java best practices: restore the interrupt flag with `Thread.currentThread().interrupt()` before throwing a new exception. This ensures that the thread's interrupted status is not lost.
- The repository converts Spring's `EmptyResultDataAccessException` into the application's `ResourceNotFoundException`. This decouples the service layer from the database technology. If you switch from JdbcTemplate to JPA, the service layer does not change.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Catching `Exception` or `Throwable` too broadly

**Wrong:**

```java
try {
    Order order = orderService.createOrder(request);
    paymentService.charge(order);
    emailService.sendConfirmation(order);
} catch (Exception e) {
    // This catches EVERYTHING: NullPointerException, IllegalArgumentException,
    // database errors, payment failures, email errors.
    // You have no idea what went wrong, and you cannot recover appropriately.
    System.out.println("Something went wrong");
}
```

**Right:**

```java
try {
    Order order = orderService.createOrder(request);
    paymentService.charge(order);
    emailService.sendConfirmation(order);
} catch (InsufficientStockException e) {
    // Specific handling: inform the user which product is out of stock
    return ResponseEntity.status(422).body(e.getMessage());
} catch (PaymentException e) {
    // Specific handling: cancel the order and inform the user
    orderService.cancelOrder(order.getId());
    return ResponseEntity.status(402).body("Payment failed");
} catch (Exception e) {
    // Catch-all for truly unexpected errors. Log the full exception.
    // logger.error("Unexpected error during order creation", e);
    return ResponseEntity.status(500).body("Internal server error");
}
```

**Why it is wrong:** Catching `Exception` broadly hides specific errors and prevents appropriate handling. A `NullPointerException` (a programming bug) is treated the same as an `InsufficientStockException` (a normal business condition). In a backend, this means you might return a 500 error for a situation that should be a 422, or worse, silently swallow a critical error.

### Mistake 2: Swallowing exceptions (empty catch blocks)

**Wrong:**

```java
try {
    paymentService.charge(order);
} catch (PaymentException e) {
    // Empty catch block! The exception is silently ignored.
    // The order is marked as paid even though the payment failed.
    // This is a data corruption bug that will be extremely hard to find.
}
```

**Right:**

```java
try {
    paymentService.charge(order);
} catch (PaymentException e) {
    // At minimum, log the exception
    // logger.error("Payment failed for order {}", order.getId(), e);
    throw e;  // Or rethrow, or handle appropriately
}
```

**Why it is wrong:** An empty catch block is the most dangerous anti-pattern in Java. It silently swallows errors, making bugs invisible. The program continues as if nothing happened, but the state is now inconsistent. In a backend, this can lead to orders marked as paid without actual payment, users created without verification emails, and inventory decremented without actual sales. **Never leave a catch block empty.** At minimum, log the exception.

### Mistake 3: Using exceptions for normal control flow

**Wrong:**

```java
public User findUserByEmail(String email) {
    try {
        return userRepository.findByEmail(email);
    } catch (ResourceNotFoundException e) {
        return null;  // Using exceptions for normal "not found" logic
    }
}

// The caller then checks for null, defeating the purpose of the exception.
```

**Right:**

```java
// Option 1: Return Optional (preferred for "might not exist" scenarios)
public Optional<User> findUserByEmail(String email) {
    return userRepository.findByEmail(email);  // Returns Optional.empty() if not found
}

// Option 2: Throw the exception if "not found" is truly exceptional
public User getUserByEmail(String email) {
    return userRepository.findByEmail(email)
        .orElseThrow(() -> new ResourceNotFoundException("User", "email", email));
}
```

**Why it is wrong:** Exceptions are expensive to create (the stack trace capture is costly). Using them for normal control flow (like "user not found") degrades performance and makes the code harder to understand. Use `Optional` for expected "not found" scenarios and exceptions for truly unexpected conditions.

### Mistake 4: Losing the original exception cause

**Wrong:**

```java
try {
    paymentGateway.charge(amount);
} catch (IOException e) {
    // Creating a new exception WITHOUT passing the original as the cause.
    // The original stack trace (which contains the actual network error) is lost.
    throw new PaymentException("Payment failed");
}
```

**Right:**

```java
try {
    paymentGateway.charge(amount);
} catch (IOException e) {
    // Pass the original exception as the cause.
    // The stack trace will show both exceptions.
    throw new PaymentException("Payment failed", e);
}
```

**Why it is wrong:** When you catch an exception and throw a new one without passing the original as the cause, you lose the entire stack trace of the original error. In production, this makes debugging nearly impossible because the log only shows "Payment failed" with no information about why the network call failed. Always use the constructor that accepts a `Throwable cause` parameter.

---

## Key Takeaways

> [!tip] Remember these points
> 1. **Exceptions are objects** that extend `Throwable`. The hierarchy is `Throwable` -> `Error` (do not catch) and `Throwable` -> `Exception` -> `RuntimeException` (unchecked) and `Exception` (checked). In modern Spring Boot, prefer unchecked exceptions (`RuntimeException`) for application errors.
> 2. **`try-catch-finally`** handles exceptions. The `try` block contains risky code, `catch` blocks handle specific exception types (most specific first), and `finally` always executes for cleanup. Use **try-with-resources** for `AutoCloseable` resources instead of manual `finally` blocks.
> 3. **`throw`** throws an exception object inside a method body. **`throws`** declares exception types in a method signature. Checked exceptions require `throws` or `try-catch`. Unchecked exceptions do not.
> 4. **Custom exceptions** should extend a base application exception class that includes an HTTP status code and error code. This allows the global exception handler to translate exceptions into structured API responses automatically.
> 5. **Never swallow exceptions** (empty catch blocks), **never catch too broadly** (`catch (Exception e)` as the first handler), and **never lose the original cause** when wrapping exceptions. Always pass the original exception as the `cause` parameter to preserve the full stack trace.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Exception Hierarchy and Catch Order (Easy)

Write a program that demonstrates the importance of catch block ordering. Create a `try` block that throws an `IllegalArgumentException`. Write three `catch` blocks: one for `IllegalArgumentException`, one for `RuntimeException`, and one for `Exception`. Verify that the most specific catch block is executed.

Then rearrange the catch blocks so that `Exception` comes first. Observe the compilation warning or the behavior change. Explain in comments why the order matters.

> **Hint:** The compiler will warn you if a catch block is unreachable because a previous catch block already handles the same exception type or its supertype.

### Exercise 2: Custom Exception Hierarchy (Medium)

Create a complete exception hierarchy for a library management system:

1. `LibraryException extends RuntimeException` with fields `errorCode` (String) and `httpStatus` (int).
2. `BookNotFoundException extends LibraryException` (404, "BOOK_NOT_FOUND").
3. `BookAlreadyBorrowedException extends LibraryException` (409, "BOOK_ALREADY_BORROWED").
4. `MemberSuspendedException extends LibraryException` (403, "MEMBER_SUSPENDED").
5. `OverdueException extends LibraryException` (422, "OVERDUE_BOOK") with an additional field `daysOverdue` (int).

Write a `LibraryService` class with a method `borrowBook(String memberId, String bookId)` that throws the appropriate exceptions based on simulated conditions. Write a `main()` method that calls `borrowBook()` with different inputs and catches each exception type, printing the error code, HTTP status, and message.

> **Hint:** Use `if` statements in `borrowBook()` to simulate different error conditions based on the input values. For example, if `bookId` is "999", throw `BookNotFoundException`.

### Exercise 3: Try-with-Resources and Exception Chaining (Medium)

Write a program that reads a CSV file and parses each line into a `Student` object. Use try-with-resources to manage the `BufferedReader`. Handle the following exceptions:

- `NoSuchFileException`: print a user-friendly message.
- `NumberFormatException`: log the line number and the invalid field, then continue processing the next line (do not stop the entire file).
- `IOException`: wrap it in a custom `DataProcessingException` with the original exception as the cause, and throw it.

Create a sample CSV file with some valid and some invalid lines to test all scenarios.

> **Hint:** Use a `try-catch` inside the `while` loop for `NumberFormatException` so that one bad line does not stop the entire file processing. The outer `try-with-resources` handles the `IOException` for file-level errors.

### Exercise 4: Global Exception Handler Simulation (Hard, Optional)

Simulate Spring Boot's `@ControllerAdvice` pattern without using Spring. Create:

1. A custom exception hierarchy (base `AppException` with `httpStatus` and `errorCode`, plus 3-4 specific subclasses).
2. A `GlobalExceptionHandler` class with a method `handle(Exception e)` that uses `instanceof` checks (or pattern matching) to determine the exception type and returns an `ErrorResponse` record with the appropriate HTTP status, error code, and message.
3. A `Controller` class with methods that throw different exceptions.
4. A `main()` method that calls the controller methods inside `try` blocks and passes any caught exceptions to the `GlobalExceptionHandler`.

This exercise demonstrates how Spring's exception handling works under the hood.

> **Hint:** The `GlobalExceptionHandler.handle()` method should check exceptions from most specific to least specific, similar to how `@ExceptionHandler` methods are ordered in Spring.

<details>
<summary>Solution for Exercise 1</summary>

```java
public class CatchOrderDemo {
    public static void main(String[] args) {
        // Correct order: most specific first
        try {
            throw new IllegalArgumentException("Invalid input");
        } catch (IllegalArgumentException e) {
            System.out.println("Caught IllegalArgumentException: " + e.getMessage());
            // This block executes because it is the most specific match.
        } catch (RuntimeException e) {
            System.out.println("Caught RuntimeException");
            // This block is NOT reached because IllegalArgumentException
            // was already caught above.
        } catch (Exception e) {
            System.out.println("Caught Exception");
            // This block is NOT reached.
        }

        // Wrong order: general first (compiler warning in some IDEs)
        try {
            throw new IllegalArgumentException("Invalid input");
        } catch (Exception e) {
            System.out.println("Caught Exception (too broad!): " + e.getClass().getSimpleName());
            // This catches EVERYTHING, including IllegalArgumentException.
            // The specific catch blocks below would be unreachable.
        }
        // catch (IllegalArgumentException e) { }  // COMPILATION ERROR: unreachable
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
class LibraryException extends RuntimeException {
    private final String errorCode;
    private final int httpStatus;

    LibraryException(String message, int httpStatus, String errorCode) {
        super(message);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public String getErrorCode() { return errorCode; }
    public int getHttpStatus() { return httpStatus; }
}

class BookNotFoundException extends LibraryException {
    BookNotFoundException(String bookId) {
        super("Book not found: " + bookId, 404, "BOOK_NOT_FOUND");
    }
}

class BookAlreadyBorrowedException extends LibraryException {
    BookAlreadyBorrowedException(String bookId) {
        super("Book already borrowed: " + bookId, 409, "BOOK_ALREADY_BORROWED");
    }
}

class MemberSuspendedException extends LibraryException {
    MemberSuspendedException(String memberId) {
        super("Member is suspended: " + memberId, 403, "MEMBER_SUSPENDED");
    }
}

class OverdueException extends LibraryException {
    private final int daysOverdue;

    OverdueException(String memberId, int daysOverdue) {
        super("Member " + memberId + " has overdue books (" + daysOverdue + " days)", 422, "OVERDUE_BOOK");
        this.daysOverdue = daysOverdue;
    }

    public int getDaysOverdue() { return daysOverdue; }
}

class LibraryService {
    void borrowBook(String memberId, String bookId) {
        if ("SUSPENDED".equals(memberId)) throw new MemberSuspendedException(memberId);
        if ("OVERDUE".equals(memberId)) throw new OverdueException(memberId, 14);
        if ("999".equals(bookId)) throw new BookNotFoundException(bookId);
        if ("BORROWED".equals(bookId)) throw new BookAlreadyBorrowedException(bookId);
        System.out.println("Book " + bookId + " borrowed by " + memberId);
    }
}

public class Main {
    public static void main(String[] args) {
        LibraryService service = new LibraryService();
        String[][] tests = {{"M001", "B001"}, {"SUSPENDED", "B001"}, {"OVERDUE", "B001"}, {"M001", "999"}, {"M001", "BORROWED"}};

        for (String[] test : tests) {
            try {
                service.borrowBook(test[0], test[1]);
            } catch (LibraryException e) {
                System.out.printf("[%d %s] %s%n", e.getHttpStatus(), e.getErrorCode(), e.getMessage());
            }
        }
    }
}
```

</details>

---

## Related Notes

- [[Java - this and super Keywords]]
- [[Java - Custom Exceptions]] (next note)
- [[Java - Comparable and Comparator]]

---

## Resources

- [Oracle Java Tutorials: Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/) - Official documentation covering the entire exception mechanism.
- [Oracle Java Tutorials: The try-with-resources Statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html) - Official guide to automatic resource management.
- [Baeldung: Java Exceptions](https://www.baeldung.com/java-exceptions) - Comprehensive guide covering checked vs unchecked, custom exceptions, and best practices.
- [Baeldung: Exception Handling in Spring MVC](https://www.baeldung.com/exception-handling-for-rest-with-spring) - Detailed guide to `@ControllerAdvice` and `@ExceptionHandler`. Read this when you reach Phase 4.
- [Effective Java by Joshua Bloch - Item 69: Use Exceptions Only for Exceptional Conditions](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive argument against using exceptions for control flow.
- [Effective Java by Joshua Bloch - Item 71: Avoid Unnecessary Use of Checked Exceptions](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Why modern Java prefers unchecked exceptions for application errors.
