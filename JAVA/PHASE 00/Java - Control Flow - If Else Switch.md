---
title: "Java - Control Flow - If Else Switch"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - control-flow
  - conditionals
status: "not-started"
---

# Java - Control Flow - If Else Switch

> [!abstract] Overview
> Control flow statements determine which blocks of code execute based on conditions. The `if-else` statement evaluates boolean expressions to branch execution, while the `switch` statement selects a branch based on the value of a single expression. Every backend system relies on control flow for request routing, input validation, business rule enforcement, error handling, and state management.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]
> - [[Java - Operators - Arithmetic Relational Logical]]

---

## Theory

### What is Control Flow?

By default, a Java program executes statements sequentially from top to bottom, one line after another. Control flow statements break this linear execution by allowing the program to make decisions, repeat actions, or jump to different sections of code.

There are three categories of control flow in Java:

1. **Decision-making**: `if`, `if-else`, `if-else if`, `switch` (covered in this note)
2. **Looping**: `for`, `while`, `do-while` (covered in the next note)
3. **Branching**: `break`, `continue`, `return` (covered across both notes)

This note focuses entirely on decision-making statements.

### The `if` Statement

The `if` statement evaluates a boolean condition. If the condition is `true`, the code block inside the curly braces executes. If the condition is `false`, the block is skipped entirely.

```java
if (condition) {
    // This code runs only when condition is true
}
```

The condition must be a boolean expression. Unlike C or C++, Java does not allow integers or other types as conditions. You cannot write `if (1)` or `if (x)` where `x` is an integer. This strictness prevents an entire class of bugs.

### The `if-else` Statement

The `else` block provides an alternative path when the `if` condition is `false`.

```java
if (condition) {
    // Runs when condition is true
} else {
    // Runs when condition is false
}
```

### The `if-else if-else` Chain

When you have multiple mutually exclusive conditions, you chain `else if` blocks. Java evaluates them from top to bottom and executes the first block whose condition is `true`. All remaining blocks are skipped.

```java
if (condition1) {
    // Runs when condition1 is true
} else if (condition2) {
    // Runs when condition1 is false AND condition2 is true
} else if (condition3) {
    // Runs when condition1 and condition2 are both false AND condition3 is true
} else {
    // Runs when ALL conditions above are false
}
```

The order of conditions matters. Java stops at the first `true` condition. If you place a broad condition before a specific one, the specific one will never be reached.

```java
int score = 95;

// WRONG ORDER: The first condition catches everything >= 50,
// so the A+ check is never reached.
if (score >= 50) {
    System.out.println("Pass");       // This prints, even though score is 95
} else if (score >= 80) {
    System.out.println("A+");         // Never reached
}

// CORRECT ORDER: Most specific conditions first.
if (score >= 80) {
    System.out.println("A+");         // This prints correctly
} else if (score >= 50) {
    System.out.println("Pass");
}
```

### The `switch` Statement

The `switch` statement is an alternative to long `if-else if` chains when you are comparing a single variable against multiple constant values. It is cleaner and, in some cases, faster because the JVM can optimize it into a jump table.

**Traditional switch (Java 1.0+):**

```java
switch (expression) {
    case value1:
        // Code for value1
        break;
    case value2:
        // Code for value2
        break;
    default:
        // Code when no case matches
        break;
}
```

The `expression` must evaluate to one of these types: `byte`, `short`, `int`, `char`, `String`, or an `enum`. You cannot switch on `long`, `float`, `double`, or `boolean`.

The `break` statement at the end of each case is critical. Without it, execution "falls through" to the next case, which is almost always a bug.

**Enhanced switch expression (Java 14+):**

Java 14 introduced a modern switch syntax that eliminates fall-through bugs and can return a value directly.

```java
String result = switch (statusCode) {
    case 200 -> "OK";
    case 404 -> "Not Found";
    case 500 -> "Internal Server Error";
    default -> "Unknown Status";
};
```

The arrow syntax `->` replaces the colon and eliminates the need for `break`. Each branch is a single expression or a block. This is the recommended style for new code.

### When to Use `if` vs `switch`

| Use `if-else` when | Use `switch` when |
|---------------------|-------------------|
| Conditions involve ranges (`score >= 80`) | Comparing a single variable against exact values |
| Conditions involve multiple variables (`age > 18 && hasLicense`) | The variable is a `String`, `int`, `char`, or `enum` |
| Conditions involve complex boolean logic | You have 3 or more discrete values to check |
| You need to check inequalities (`<`, `>`, `<=`, `>=`) | You want cleaner, more readable code for discrete choices |

### How It Works Internally

At the bytecode level, `if` statements compile to conditional jump instructions. When the JVM encounters `if (x > 10)`, it generates an instruction like `if_icmple` (if integer compare less than or equal) that compares two values on the stack and jumps to a different bytecode offset if the condition is false.

`switch` statements compile to one of two bytecode instructions depending on the case values:

- **`tableswitch`**: Used when case values are contiguous or nearly contiguous (e.g., 1, 2, 3, 4, 5). The JVM creates a jump table in memory and performs an O(1) lookup. This is extremely fast.
- **`lookupswitch`**: Used when case values are sparse (e.g., 10, 1000, 50000). The JVM performs a binary search through the sorted case values. This is O(log n).

Both are faster than a long chain of `if-else if` statements, which the JVM must evaluate sequentially. This performance difference is negligible for small numbers of cases but matters in high-throughput backend systems processing millions of requests.

> [!tip] Key Insight
> The enhanced switch expression (Java 14+) is not just syntactic sugar. It is an **expression**, meaning it returns a value. This makes it composable: you can assign its result to a variable, return it from a method, or pass it as an argument. Traditional switch is a **statement** that cannot return a value. In modern Java backend code, prefer the enhanced switch whenever possible.

---

## Syntax and Basic Examples

### Example 1: Basic if-else

```java
public class BasicIfElse {
    public static void main(String[] args) {
        int temperature = 35;

        // Simple if
        if (temperature > 30) {
            System.out.println("It is hot outside. Stay hydrated.");
        }

        // if-else
        if (temperature > 40) {
            System.out.println("Extreme heat warning.");
        } else {
            System.out.println("Temperature is manageable.");
        }

        // if-else if-else chain
        if (temperature >= 40) {
            System.out.println("Extreme heat");
        } else if (temperature >= 30) {
            System.out.println("Hot");            // This prints
        } else if (temperature >= 20) {
            System.out.println("Warm");
        } else if (temperature >= 10) {
            System.out.println("Cool");
        } else {
            System.out.println("Cold");
        }
    }
}
```

**Output:**
```
It is hot outside. Stay hydrated.
Temperature is manageable.
Hot
```

### Example 2: Traditional switch

```java
public class TraditionalSwitch {
    public static void main(String[] args) {
        String day = "WEDNESDAY";

        switch (day) {
            case "MONDAY":
                System.out.println("Start of the work week");
                break;
            case "TUESDAY":
            case "WEDNESDAY":
            case "THURSDAY":
                // Multiple cases sharing the same code block.
                // This is intentional fall-through, one of the few valid uses.
                System.out.println("Midweek grind");
                break;
            case "FRIDAY":
                System.out.println("Almost weekend!");
                break;
            case "SATURDAY":
            case "SUNDAY":
                System.out.println("Weekend!");
                break;
            default:
                System.out.println("Invalid day: " + day);
                break;
        }
    }
}
```

**Output:**
```
Midweek grind
```

### Example 3: Enhanced switch expression (Java 14+)

```java
public class EnhancedSwitch {
    public static void main(String[] args) {
        int httpStatus = 404;

        // Switch as an expression that returns a value
        String message = switch (httpStatus) {
            case 200 -> "OK";
            case 201 -> "Created";
            case 400 -> "Bad Request";
            case 401 -> "Unauthorized";
            case 403 -> "Forbidden";
            case 404 -> "Not Found";
            case 500 -> "Internal Server Error";
            case 503 -> "Service Unavailable";
            default -> "Unknown Status: " + httpStatus;
        };

        System.out.println("HTTP " + httpStatus + ": " + message);

        // Switch with blocks for multi-line logic
        String category = switch (httpStatus / 100) {
            case 2 -> {
                System.out.println("Success category");
                yield "Success";  // 'yield' returns a value from a switch block
            }
            case 3 -> "Redirection";
            case 4 -> {
                System.out.println("Client error category");
                yield "Client Error";
            }
            case 5 -> "Server Error";
            default -> "Unknown";
        };

        System.out.println("Category: " + category);
    }
}
```

**Output:**
```
HTTP 404: Not Found
Client error category
Category: Client Error
```

### Example 4: Nested conditions

```java
public class NestedConditions {
    public static void main(String[] args) {
        boolean isMember = true;
        int purchaseAmount = 5000;
        String couponCode = "EID2025";

        if (isMember) {
            if (purchaseAmount >= 3000) {
                if ("EID2025".equals(couponCode)) {
                    System.out.println("20% discount applied. Final amount: " + (purchaseAmount * 0.8));
                } else {
                    System.out.println("10% member discount applied. Final amount: " + (purchaseAmount * 0.9));
                }
            } else {
                System.out.println("5% member discount applied. Final amount: " + (purchaseAmount * 0.95));
            }
        } else {
            System.out.println("No discount. Final amount: " + purchaseAmount);
        }
    }
}
```

**Output:**
```
20% discount applied. Final amount: 4000.0
```

> [!warning] Avoid deep nesting
> The example above has three levels of nesting, which is already hard to read. In real backend code, deeply nested `if` statements are a code smell. You will learn techniques to flatten nested conditions using early returns, guard clauses, and polymorphism in later phases.

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Control flow is the backbone of all business logic in backend systems. Here are three realistic scenarios.

### Scenario 1: HTTP exception handler in Spring Boot

Every Spring Boot backend needs a global exception handler that maps different exception types to appropriate HTTP status codes and error messages. This is a classic use case for `if-else` chains.

```java
package com.company.orderservice.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex) {
        HttpStatus status;
        String message;

        if (ex instanceof IllegalArgumentException) {
            // Client sent invalid data (e.g., negative quantity, empty email)
            status = HttpStatus.BAD_REQUEST;
            message = "Invalid input: " + ex.getMessage();

        } else if (ex instanceof ResourceNotFoundException) {
            // Client requested something that does not exist (e.g., order ID 999)
            status = HttpStatus.NOT_FOUND;
            message = ex.getMessage();

        } else if (ex instanceof UnauthorizedAccessException) {
            // Client is authenticated but lacks permission
            status = HttpStatus.FORBIDDEN;
            message = "You do not have permission to perform this action.";

        } else if (ex instanceof DuplicateResourceException) {
            // Client tried to create something that already exists (e.g., duplicate email)
            status = HttpStatus.CONFLICT;
            message = "Resource already exists: " + ex.getMessage();

        } else if (ex instanceof RateLimitExceededException) {
            // Client is sending too many requests
            status = HttpStatus.TOO_MANY_REQUESTS;
            message = "Rate limit exceeded. Try again later.";

        } else {
            // Unexpected error. Never expose internal details to the client.
            status = HttpStatus.INTERNAL_SERVER_ERROR;
            message = "An unexpected error occurred. Please contact support.";
            // Log the full exception for the development team
            // logger.error("Unhandled exception", ex);
        }

        ErrorResponse error = new ErrorResponse(status.value(), message);
        return new ResponseEntity<>(error, status);
    }
}
```

**What to notice:**

- The `if-else if` chain checks the exception type using `instanceof`. This is appropriate because the conditions involve type checking, not simple value matching, so `switch` would not work here (prior to Java 21 pattern matching).
- The `else` block at the end is a catch-all for unexpected errors. In production, you never expose stack traces or internal error details to the client because that information can be exploited by attackers.
- Each branch maps to a specific HTTP status code. This mapping is the core responsibility of a backend exception handler.

### Scenario 2: Order status state machine using enhanced switch

E-commerce backends manage order lifecycles through a series of states. The `switch` statement is ideal for state transitions because each state is a discrete value.

```java
package com.company.orderservice.service;

public class OrderStateMachine {

    public enum OrderStatus {
        PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED, REFUNDED
    }

    public OrderStatus getNextStatus(OrderStatus currentStatus, String action) {
        return switch (currentStatus) {
            case PENDING -> switch (action) {
                case "CONFIRM" -> OrderStatus.CONFIRMED;
                case "CANCEL" -> OrderStatus.CANCELLED;
                default -> throw new IllegalStateException(
                    "Invalid action '" + action + "' for PENDING order"
                );
            };

            case CONFIRMED -> switch (action) {
                case "PROCESS" -> OrderStatus.PROCESSING;
                case "CANCEL" -> OrderStatus.CANCELLED;
                default -> throw new IllegalStateException(
                    "Invalid action '" + action + "' for CONFIRMED order"
                );
            };

            case PROCESSING -> switch (action) {
                case "SHIP" -> OrderStatus.SHIPPED;
                case "CANCEL" -> OrderStatus.CANCELLED;
                default -> throw new IllegalStateException(
                    "Invalid action '" + action + "' for PROCESSING order"
                );
            };

            case SHIPPED -> switch (action) {
                case "DELIVER" -> OrderStatus.DELIVERED;
                default -> throw new IllegalStateException(
                    "Invalid action '" + action + "' for SHIPPED order"
                );
            };

            case DELIVERED -> switch (action) {
                case "REFUND" -> OrderStatus.REFUNDED;
                default -> throw new IllegalStateException(
                    "Invalid action '" + action + "' for DELIVERED order"
                );
            };

            case CANCELLED, REFUNDED -> throw new IllegalStateException(
                "Cannot transition from terminal status: " + currentStatus
            );
        };
    }
}
```

**What to notice:**

- The enhanced switch with arrow syntax makes the state machine extremely readable. Each state maps to its valid transitions.
- Nested switch expressions handle the combination of current state and action. This would be much messier with `if-else` chains.
- The `default` branch throws an exception for invalid transitions. This is a defensive programming pattern. In a real system, invalid state transitions indicate a bug or a malicious request, and the system should fail loudly rather than silently ignore the problem.
- `CANCELLED` and `REFUNDED` are terminal states. The comma syntax `case CANCELLED, REFUNDED ->` groups them together, which is cleaner than listing them separately.

### Scenario 3: Request routing based on HTTP method

```java
package com.company.orderservice.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    // In a real Spring Boot app, you use annotations like @GetMapping, @PostMapping, etc.
    // But internally, Spring's DispatcherServlet uses logic similar to this
    // to route incoming HTTP requests to the correct handler method.

    public ResponseEntity<?> handleRequest(String httpMethod, String orderId) {
        return switch (httpMethod.toUpperCase()) {
            case "GET" -> {
                if (orderId == null || orderId.isBlank()) {
                    yield ResponseEntity.ok(orderService.getAllOrders());
                } else {
                    yield ResponseEntity.ok(orderService.getOrderById(orderId));
                }
            }
            case "POST" -> {
                // Create a new order
                yield ResponseEntity.status(201).body(orderService.createOrder(requestBody));
            }
            case "PUT" -> {
                if (orderId == null) {
                    yield ResponseEntity.badRequest().body("Order ID is required for update");
                }
                yield ResponseEntity.ok(orderService.updateOrder(orderId, requestBody));
            }
            case "DELETE" -> {
                if (orderId == null) {
                    yield ResponseEntity.badRequest().body("Order ID is required for deletion");
                }
                orderService.deleteOrder(orderId);
                yield ResponseEntity.noContent().build();
            }
            default -> ResponseEntity.status(405).body("Method Not Allowed: " + httpMethod);
        };
    }
}
```

**What to notice:**

- The `switch` routes based on the HTTP method. This mirrors what Spring's internal `DispatcherServlet` does for every incoming request.
- Inside the `GET` case, an `if-else` checks whether an order ID was provided. This demonstrates how `switch` and `if-else` work together naturally: `switch` for discrete value matching, `if-else` for conditional logic within each branch.
- The `yield` keyword returns a value from a multi-line switch block. This is different from `return`, which would exit the entire method. `yield` only exits the switch expression.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Forgetting `break` in traditional switch (fall-through)

**Wrong:**
```java
int month = 2;
switch (month) {
    case 1:
        System.out.println("January");
    case 2:
        System.out.println("February");    // This prints (correct)
    case 3:
        System.out.println("March");       // This ALSO prints (bug!)
    case 4:
        System.out.println("April");       // This ALSO prints (bug!)
    default:
        System.out.println("Unknown");     // This ALSO prints (bug!)
}
```

**Output of wrong code:**
```
February
March
April
Unknown
```

**Right:**
```java
int month = 2;
switch (month) {
    case 1:
        System.out.println("January");
        break;
    case 2:
        System.out.println("February");
        break;
    case 3:
        System.out.println("March");
        break;
    default:
        System.out.println("Unknown");
        break;
}
```

**Why it is wrong:** Without `break`, execution falls through to the next case regardless of whether it matches. This is one of the most notorious sources of bugs in C, C++, and Java. The enhanced switch expression (Java 14+) eliminates this problem entirely by not requiring `break`.

### Mistake 2: Dangling else ambiguity

**Wrong:**
```java
int x = 10;
int y = 20;

if (x > 5)
    if (y > 25)
        System.out.println("Both large");
    else
        System.out.println("x is large but y is not");
// Which 'if' does the 'else' belong to?
// It belongs to the INNER if (y > 25), not the outer if (x > 5).
// This is confusing and error-prone.
```

**Right:**
```java
int x = 10;
int y = 20;

if (x > 5) {
    if (y > 25) {
        System.out.println("Both large");
    } else {
        System.out.println("x is large but y is not");
    }
}
```

**Why it is wrong:** Java's rule is that `else` always binds to the nearest unmatched `if`. This is called the "dangling else" problem. Without curly braces, the visual indentation can mislead you about which `if` the `else` belongs to. Always use curly braces, even for single-line blocks.

### Mistake 3: Using `switch` for range checks

**Wrong:**
```java
int score = 85;
// You cannot do this with switch. Switch only matches exact values.
switch (score) {
    case (score >= 80):  // COMPILATION ERROR
        System.out.println("A");
        break;
}
```

**Right:**
```java
int score = 85;
if (score >= 80) {
    System.out.println("A");
} else if (score >= 70) {
    System.out.println("B");
}
```

**Why it is wrong:** `switch` cases must be compile-time constants (literals, final variables, or enum values). You cannot use boolean expressions, ranges, or variables as case labels in a traditional switch. Use `if-else` for range checks.

### Mistake 4: Not handling the `default` case

**Wrong:**
```java
String role = getUserRole();  // Could return anything
switch (role) {
    case "ADMIN" -> grantFullAccess();
    case "USER" -> grantLimitedAccess();
    // No default. If role is "GUEST" or null or "MODERATOR",
    // nothing happens. The user gets no access and no error message.
    // This is a silent failure, which is the hardest kind of bug to find.
}
```

**Right:**
```java
String role = getUserRole();
switch (role) {
    case "ADMIN" -> grantFullAccess();
    case "USER" -> grantLimitedAccess();
    case "GUEST" -> grantReadOnlyAccess();
    default -> throw new SecurityException("Unknown role: " + role);
}
```

**Why it is wrong:** Omitting `default` means unexpected values are silently ignored. In backend systems, this can lead to security vulnerabilities (a user with an unknown role gets no access check at all) or data corruption (an order with an unknown status is skipped by the processing logic). Always include a `default` branch that either handles the unexpected case or throws an exception.

---

## Key Takeaways

> [!tip] Remember these points
> 1. `if-else` evaluates boolean conditions and is best for range checks, complex logic, and multiple variables. `switch` matches a single expression against discrete values and is best for enums, status codes, and fixed sets of options.
> 2. Always use curly braces `{}` with `if-else` blocks, even for single statements. This prevents dangling else bugs and makes the code easier to modify later.
> 3. In traditional `switch`, always include `break` at the end of each case unless you intentionally want fall-through. Better yet, use the enhanced switch expression (Java 14+) which eliminates `break` entirely.
> 4. Always include a `default` case in every `switch` statement. Handle unexpected values explicitly rather than silently ignoring them.
> 5. Order your `if-else if` conditions from most specific to most general. A broad condition placed first will catch everything and prevent more specific conditions from ever being evaluated.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Traffic Light Simulator (Easy)
Write a program that takes a traffic light color as input (`"RED"`, `"YELLOW"`, `"GREEN"`) and prints the appropriate action:
- RED: "Stop"
- YELLOW: "Slow down and prepare to stop"
- GREEN: "Go"
- Any other input: "Invalid signal"

Use a `switch` statement. Then rewrite the same logic using `if-else`. Compare the two versions and note which one is more readable.

**Hint:** Use `scanner.nextLine().toUpperCase()` to handle case-insensitive input.

### Exercise 2: BMI Calculator with Category (Medium)
Write a program that takes a person's weight in kilograms and height in meters as input. Calculate BMI using the formula `weight / (height * height)`. Then use `if-else if` to classify the BMI:
- Below 18.5: Underweight
- 18.5 to 24.9: Normal weight
- 25.0 to 29.9: Overweight
- 30.0 and above: Obese

Print the BMI value (rounded to 1 decimal place) and the category.

**Hint:** Use `double` for weight, height, and BMI. Use `Math.round(bmi * 10.0) / 10.0` to round to one decimal place.

### Exercise 3: Enhanced Switch Calculator (Medium)
Rewrite the calculator from the previous note's Exercise 1 using an enhanced switch expression (Java 14+). The switch should take the operator string and return the result as a `double`. Handle division by zero inside the switch block using a block with `yield`.

**Hint:**
```java
double result = switch (operator) {
    case "+" -> num1 + num2;
    case "/" -> {
        if (num2 == 0) yield Double.NaN;
        yield num1 / num2;
    }
    // ... other cases
    default -> throw new IllegalArgumentException("Unknown operator");
};
```

### Exercise 4: Order Status Validator (Hard, Optional)
Write a program that simulates an order status validator. Define an enum `OrderStatus` with values `PENDING`, `CONFIRMED`, `SHIPPED`, `DELIVERED`, `CANCELLED`. Take a current status and a requested new status as input. Use nested switch expressions to determine if the transition is valid:
- PENDING can go to CONFIRMED or CANCELLED
- CONFIRMED can go to SHIPPED or CANCELLED
- SHIPPED can go to DELIVERED
- DELIVERED and CANCELLED are terminal (no transitions allowed)

Print whether the transition is valid or not, and if invalid, explain why.

**Hint:** The outer switch matches the current status. The inner switch matches the requested status. Use the enhanced switch with arrow syntax for clean code.

### Solution
For Exercise 1: (switch version)
```java
import java.util.Scanner;

public class TrafficLight {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter traffic light color: ");
        String color = scanner.nextLine().toUpperCase().trim();

        String action = switch (color) {
            case "RED" -> "Stop";
            case "YELLOW" -> "Slow down and prepare to stop";
            case "GREEN" -> "Go";
            default -> "Invalid signal: " + color;
        };

        System.out.println(action);
        scanner.close();
    }
}
```

For Exercise 2:
```java
import java.util.Scanner;

public class BMICalculator {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter weight in kg: ");
        double weight = scanner.nextDouble();

        System.out.print("Enter height in meters: ");
        double height = scanner.nextDouble();

        if (weight <= 0 || height <= 0) {
            System.out.println("Error: Weight and height must be positive.");
            return;
        }

        double bmi = weight / (height * height);
        double roundedBmi = Math.round(bmi * 10.0) / 10.0;

        String category;
        if (bmi < 18.5) {
            category = "Underweight";
        } else if (bmi < 25.0) {
            category = "Normal weight";
        } else if (bmi < 30.0) {
            category = "Overweight";
        } else {
            category = "Obese";
        }

        System.out.println("Your BMI: " + roundedBmi);
        System.out.println("Category: " + category);
        scanner.close();
    }
}
```

For Exercise 4:
```java
import java.util.Scanner;

public class OrderStatusValidator {

    enum OrderStatus {
        PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Current status: ");
        OrderStatus current = OrderStatus.valueOf(scanner.nextLine().toUpperCase());

        System.out.print("Requested status: ");
        OrderStatus requested = OrderStatus.valueOf(scanner.nextLine().toUpperCase());

        String result = switch (current) {
            case PENDING -> switch (requested) {
                case CONFIRMED, CANCELLED -> "Valid transition";
                default -> "Invalid. PENDING can only go to CONFIRMED or CANCELLED.";
            };
            case CONFIRMED -> switch (requested) {
                case SHIPPED, CANCELLED -> "Valid transition";
                default -> "Invalid. CONFIRMED can only go to SHIPPED or CANCELLED.";
            };
            case SHIPPED -> switch (requested) {
                case DELIVERED -> "Valid transition";
                default -> "Invalid. SHIPPED can only go to DELIVERED.";
            };
            case DELIVERED -> "Invalid. DELIVERED is a terminal status.";
            case CANCELLED -> "Invalid. CANCELLED is a terminal status.";
        };

        System.out.println(current + " -> " + requested + ": " + result);
        scanner.close();
    }
}
```


---

## Related Notes

- [[Java - Operators - Arithmetic Relational Logical]]
- [[Java - Loops - For While Do-While]]

---

## Resources

- [Oracle Java Tutorials: The if-then and if-then-else Statements](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/if.html) - Official documentation with examples.
- [Oracle Java Tutorials: The switch Statement](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/switch.html) - Official documentation covering traditional switch.
- [Baeldung: Java Switch Statement](https://www.baeldung.com/java-switch) - Comprehensive guide covering both traditional and enhanced switch.
- [Baeldung: Java 14 Switch Expressions](https://www.baeldung.com/java-switch) - Detailed explanation of the modern switch syntax with yield and arrow notation.
