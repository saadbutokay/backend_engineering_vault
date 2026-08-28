---
title: "Java - Loops - For While Do-While"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - loops
  - iteration
status: "not-started"
---

# Java - Loops - For While Do-While

> [!abstract] Overview
> Loops allow you to execute a block of code repeatedly based on a condition or a count. Java provides three loop constructs: `for`, `while`, and `do-while`, plus the enhanced `for-each` loop for iterating over collections. In backend engineering, loops process database result sets, iterate over lists of users or orders, handle batch operations, retry failed network calls, and power scheduled background tasks.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]
> - [[Java - Operators - Arithmetic Relational Logical]]
> - [[Java - Control Flow - If Else Switch]]

---

## Theory

### What is a Loop?

A loop is a control flow statement that repeats a block of code as long as a specified condition remains `true`. Without loops, you would have to write the same code over and over manually. If you needed to process 10,000 orders, you would need 10,000 nearly identical lines of code. Loops reduce this to a few lines.

Every loop has three conceptual components:

1. **Initialization**: The starting state. What is the counter or condition before the loop begins?
2. **Condition**: The boolean expression checked before (or after) each iteration. When this becomes `false`, the loop stops.
3. **Update**: How the state changes after each iteration. This must eventually cause the condition to become `false`, or the loop runs forever.

### The `while` Loop

The `while` loop is the simplest loop. It checks the condition **before** each iteration. If the condition is `false` from the start, the loop body never executes.

```java
while (condition) {
    // Code to repeat
    // Must include something that eventually makes condition false
}
```

Use `while` when you do not know in advance how many times the loop will run. For example, reading lines from a file until the end, or retrying a database connection until it succeeds.

### The `do-while` Loop

The `do-while` loop checks the condition **after** each iteration. This guarantees that the loop body executes **at least once**, even if the condition is `false` from the start.

```java
do {
    // Code to repeat
} while (condition);  // Note the semicolon here
```

Use `do-while` when you need to execute the body at least once before checking the condition. A common example is a menu-driven console application where you show the menu, get input, and then check if the user wants to quit.

### The `for` Loop

The `for` loop combines initialization, condition, and update into a single line. It is the most commonly used loop when you know exactly how many times you want to iterate.

```java
for (initialization; condition; update) {
    // Code to repeat
}
```

The execution order is:

1. Run `initialization` once.
2. Check `condition`. If `false`, exit the loop.
3. Execute the loop body.
4. Run `update`.
5. Go back to step 2.

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
// Prints: 0, 1, 2, 3, 4
```

### The Enhanced `for` Loop (For-Each)

Java 5 introduced the enhanced `for` loop, also called the for-each loop. It iterates over every element in an array or collection without using an index variable.

```java
for (ElementType element : collection) {
    // Use element directly
}
```

```java
String[] cities = {"Dhaka", "Chittagong", "Sylhet", "Rajshahi"};
for (String city : cities) {
    System.out.println(city);
}
```

The for-each loop is cleaner and less error-prone than an indexed `for` loop because you cannot accidentally access an out-of-bounds index. However, it has limitations: you cannot modify the collection during iteration, you do not have access to the index, and you cannot iterate in reverse.

### Loop Control Statements

**`break`**: Immediately exits the innermost loop. Execution continues with the first statement after the loop.

```java
for (int i = 0; i < 100; i++) {
    if (i == 5) {
        break;  // Exits the loop when i reaches 5
    }
    System.out.println(i);  // Prints 0, 1, 2, 3, 4
}
```

**`continue`**: Skips the rest of the current iteration and jumps to the next iteration. In a `for` loop, it jumps to the update step. In a `while` loop, it jumps to the condition check.

```java
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) {
        continue;  // Skips even numbers
    }
    System.out.println(i);  // Prints 1, 3, 5, 7, 9
}
```

**Labeled `break` and `continue`**: When you have nested loops, you can use labels to break or continue an outer loop from inside an inner loop.

```java
outer:
for (int i = 0; i < 5; i++) {
    for (int j = 0; j < 5; j++) {
        if (i * j > 6) {
            break outer;  // Breaks out of BOTH loops
        }
    }
}
```

Labeled breaks are rare in professional code. If you find yourself needing them, it is usually a sign that the logic should be extracted into a separate method with a `return` statement.

### How Loops Work Internally

At the bytecode level, all loops compile down to conditional jump instructions, the same ones used by `if` statements. A `for` loop and a `while` loop with equivalent logic produce nearly identical bytecode.

Consider this `for` loop:

```java
for (int i = 0; i < 3; i++) {
    System.out.println(i);
}
```

The JVM bytecode looks conceptually like this:

```
0: iconst_0          // i = 0
1: istore_1          // store i
2: iload_1           // load i
3: iconst_3          // push 3
4: if_icmpge 15      // if i >= 3, jump to instruction 15 (exit loop)
7: getstatic ...     // System.out
10: iload_1          // load i
11: invokevirtual    // println(i)
14: iinc 1, 1        // i++
17: goto 2           // jump back to condition check
15: return           // exit
```

The loop is just a `goto` instruction that jumps back to the condition check. This is why infinite loops happen: if the condition never becomes `false`, the `goto` keeps jumping back forever.

The enhanced for-each loop compiles differently depending on the type. For arrays, it compiles to an indexed `for` loop. For collections (like `ArrayList`), it compiles to use an `Iterator` object behind the scenes:

```java
// What you write:
for (String name : names) { ... }

// What the compiler generates (conceptually):
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    ...
}
```

> [!tip] Key Insight
> All loops in Java are syntactic variations of the same underlying mechanism: a conditional jump backward in the bytecode. The `for` loop is not faster than `while`. The enhanced for-each loop is not slower than an indexed loop for arrays. Choose the loop type based on readability and intent, not performance. The JIT compiler optimizes all of them equally well.

---

## Syntax and Basic Examples

### Example 1: All four loop types doing the same thing

```java
public class LoopComparison {
    public static void main(String[] args) {
        // Goal: Print numbers 1 through 5

        // --- while loop ---
        System.out.println("--- while ---");
        int i = 1;
        while (i <= 5) {
            System.out.print(i + " ");
            i++;
        }
        System.out.println();

        // --- do-while loop ---
        System.out.println("--- do-while ---");
        int j = 1;
        do {
            System.out.print(j + " ");
            j++;
        } while (j <= 5);
        System.out.println();

        // --- for loop ---
        System.out.println("--- for ---");
        for (int k = 1; k <= 5; k++) {
            System.out.print(k + " ");
        }
        System.out.println();

        // --- enhanced for-each loop ---
        System.out.println("--- for-each ---");
        int[] numbers = {1, 2, 3, 4, 5};
        for (int n : numbers) {
            System.out.print(n + " ");
        }
        System.out.println();
    }
}
```

**Output:**
```
--- while ---
1 2 3 4 5 
--- do-while ---
1 2 3 4 5 
--- for-each ---
1 2 3 4 5 
--- for ---
1 2 3 4 5 
```

### Example 2: When `while` and `do-while` behave differently

```java
public class WhileVsDoWhile {
    public static void main(String[] args) {
        int x = 10;

        // while: condition is false from the start, body never runs
        System.out.println("--- while (x < 5) ---");
        while (x < 5) {
            System.out.println("This will NOT print");
            x++;
        }
        System.out.println("After while: x = " + x);  // x is still 10

        // do-while: body runs once even though condition is false
        System.out.println("--- do-while (x < 5) ---");
        do {
            System.out.println("This WILL print once");
            x++;
        } while (x < 5);
        System.out.println("After do-while: x = " + x);  // x is now 11
    }
}
```

**Output:**
```
--- while (x < 5) ---
After while: x = 10
--- do-while (x < 5) ---
This WILL print once
After do-while: x = 11
```

### Example 3: Nested loops and break/continue

```java
public class NestedLoops {
    public static void main(String[] args) {
        // Multiplication table (1 to 5)
        System.out.println("--- Multiplication Table ---");
        for (int row = 1; row <= 5; row++) {
            for (int col = 1; col <= 5; col++) {
                System.out.printf("%4d", row * col);  // %4d formats to 4-character width
            }
            System.out.println();  // New line after each row
        }

        // Using continue to skip specific iterations
        System.out.println("\n--- Odd numbers only (using continue) ---");
        for (int i = 1; i <= 10; i++) {
            if (i % 2 == 0) {
                continue;  // Skip even numbers
            }
            System.out.print(i + " ");
        }
        System.out.println();

        // Using break to stop early
        System.out.println("\n--- First number divisible by 7 after 50 ---");
        for (int i = 51; i <= 100; i++) {
            if (i % 7 == 0) {
                System.out.println("Found: " + i);  // 56
                break;  // Stop searching
            }
        }
    }
}
```

**Output:**
```
--- Multiplication Table ---
   1   2   3   4   5
   2   4   6   8  10
   3   6   9  12  15
   4   8  12  16  20
   5  10  15  20  25

--- Odd numbers only (using continue) ---
1 3 5 7 9 

--- First number divisible by 7 after 50 ---
Found: 56
```

### Example 4: Looping with user input

```java
import java.util.Scanner;

public class InputLoop {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int sum = 0;
        int count = 0;

        System.out.println("Enter positive numbers to sum. Enter -1 to stop.");

        while (true) {  // Infinite loop, controlled by break
            System.out.print("Enter a number: ");
            int number = scanner.nextInt();

            if (number == -1) {
                break;  // Exit condition
            }

            if (number < 0) {
                System.out.println("Ignoring negative number.");
                continue;  // Skip to next iteration
            }

            sum += number;
            count++;
        }

        if (count > 0) {
            double average = (double) sum / count;
            System.out.println("Sum: " + sum);
            System.out.println("Count: " + count);
            System.out.println("Average: " + average);
        } else {
            System.out.println("No numbers were entered.");
        }

        scanner.close();
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Loops in backend systems process collections of data, retry operations, and handle batch tasks. Here are three realistic scenarios.

### Scenario 1: Processing a list of orders from the database

When a Spring Boot service fetches a list of orders from the database, it often needs to transform, filter, or enrich each order before returning it to the client.

```java
package com.company.orderservice.service;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

public class OrderReportService {

    public OrderReport generateDailyReport(List<Order> orders) {
        BigDecimal totalRevenue = BigDecimal.ZERO;
        int paidOrders = 0;
        int pendingOrders = 0;
        int cancelledOrders = 0;
        BigDecimal largestOrder = BigDecimal.ZERO;

        // Enhanced for-each loop to process each order
        for (Order order : orders) {
            // Skip null entries (defensive programming)
            if (order == null) {
                continue;
            }

            // Accumulate totals based on order status
            switch (order.getStatus()) {
                case PAID -> {
                    totalRevenue = totalRevenue.add(order.getTotalAmount());
                    paidOrders++;
                    if (order.getTotalAmount().compareTo(largestOrder) > 0) {
                        largestOrder = order.getTotalAmount();
                    }
                }
                case PENDING -> pendingOrders++;
                case CANCELLED -> cancelledOrders++;
            }
        }

        return new OrderReport(
            orders.size(),
            paidOrders,
            pendingOrders,
            cancelledOrders,
            totalRevenue,
            largestOrder
        );
    }
}
```

**What to notice:**

- The enhanced for-each loop `for (Order order : orders)` is the standard way to iterate over collections in backend Java. You will see this pattern in virtually every service class.
- The `continue` statement skips null entries. In real systems, collections can contain null elements due to database joins or deserialization issues. Defensive null checks inside loops prevent `NullPointerException` crashes.
- The `switch` inside the `for` loop is a common pattern: iterate over a collection and categorize each element. This is the imperative style. In Phase 1, you will learn the Streams API, which provides a declarative alternative.

### Scenario 2: Retry logic for external API calls

Backend services frequently call external APIs (payment gateways, SMS providers, email services). These calls can fail due to network issues. A retry loop with exponential backoff is a standard pattern.

```java
package com.company.orderservice.integration;

import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.URI;
import java.time.Duration;

public class PaymentGatewayClient {

    private static final int MAX_RETRIES = 3;
    private static final long INITIAL_DELAY_MS = 1000;

    public HttpResponse<String> processPayment(String paymentRequestJson) {
        HttpClient client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.paymentgateway.com/v1/charge"))
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer " + apiKey)
            .POST(HttpRequest.BodyPublishers.ofString(paymentRequestJson))
            .build();

        int attempt = 0;
        long delay = INITIAL_DELAY_MS;

        while (attempt < MAX_RETRIES) {
            attempt++;
            try {
                HttpResponse<String> response = client.send(
                    request, HttpResponse.BodyHandlers.ofString()
                );

                // Success: 2xx status code
                if (response.statusCode() >= 200 && response.statusCode() < 300) {
                    return response;
                }

                // Client error (4xx): do not retry, the request is invalid
                if (response.statusCode() >= 400 && response.statusCode() < 500) {
                    throw new PaymentException(
                        "Payment rejected: " + response.body()
                    );
                }

                // Server error (5xx): retry
                System.out.println("Attempt " + attempt + " failed with status "
                    + response.statusCode() + ". Retrying in " + delay + "ms...");

            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new PaymentException("Payment processing interrupted", e);
            } catch (Exception e) {
                System.out.println("Attempt " + attempt + " failed: " + e.getMessage());
            }

            // Wait before retrying (exponential backoff)
            if (attempt < MAX_RETRIES) {
                try {
                    Thread.sleep(delay);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    throw new PaymentException("Retry interrupted", e);
                }
                delay *= 2;  // 1s -> 2s -> 4s
            }
        }

        throw new PaymentException(
            "Payment failed after " + MAX_RETRIES + " attempts"
        );
    }
}
```

**What to notice:**

- The `while` loop is the right choice here because the number of iterations depends on runtime conditions (whether the API call succeeds or fails). A `for` loop would also work, but `while` reads more naturally for retry logic.
- The `break` is implicit: the method returns the response on success, which exits the loop and the method simultaneously.
- Exponential backoff (`delay *= 2`) is a critical pattern in distributed systems. If the payment gateway is overloaded, retrying immediately makes the problem worse. Increasing the delay between retries gives the external service time to recover.
- The `if (attempt < MAX_RETRIES)` check before sleeping prevents an unnecessary delay after the final failed attempt.

### Scenario 3: Batch processing with pagination

When a backend system needs to process a large number of records (e.g., sending emails to all users, generating monthly invoices), it processes them in batches to avoid overwhelming the database and running out of memory.

```java
package com.company.orderservice.batch;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import java.util.List;

public class InvoiceBatchProcessor {

    private static final int BATCH_SIZE = 500;

    public BatchResult generateMonthlyInvoices(int year, int month) {
        int page = 0;
        int totalProcessed = 0;
        int totalFailed = 0;
        boolean hasMoreData = true;

        while (hasMoreData) {
            // Fetch one batch of orders from the database
            Page<Order> orderPage = orderRepository.findUninvoicedOrders(
                year, month, PageRequest.of(page, BATCH_SIZE)
            );

            List<Order> orders = orderPage.getContent();

            if (orders.isEmpty()) {
                hasMoreData = false;
                break;
            }

            // Process each order in the current batch
            for (Order order : orders) {
                try {
                    invoiceService.generateInvoice(order);
                    totalProcessed++;
                } catch (Exception e) {
                    totalFailed++;
                    // Log the error but continue processing other orders.
                    // One failed invoice should not stop the entire batch.
                    // logger.error("Failed to generate invoice for order {}", order.getId(), e);
                }
            }

            // Move to the next page
            page++;
            hasMoreData = orderPage.hasNext();

            // Log progress
            System.out.println("Processed page " + page + ". "
                + totalProcessed + " succeeded, " + totalFailed + " failed so far.");
        }

        return new BatchResult(totalProcessed, totalFailed);
    }
}
```

**What to notice:**

- The outer `while` loop handles pagination. It keeps fetching pages until there is no more data. This pattern is essential for processing millions of records without loading them all into memory at once.
- The inner `for-each` loop processes each order within the current batch.
- The `try-catch` inside the loop ensures that a single failure does not crash the entire batch. This is a standard resilience pattern in backend batch processing.
- The `BATCH_SIZE` constant controls memory usage. A batch of 500 orders might use 50 MB of memory. A batch of 500,000 might use 50 GB and crash the server. Choosing the right batch size is an important engineering decision.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Infinite loop due to missing update

**Wrong:**
```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    // Forgot i++. The condition i < 10 is always true.
    // This loop runs forever and crashes your program.
}
```

**Right:**
```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;  // Update the counter
}
```

**Why it is wrong:** If the loop variable never changes, the condition never becomes `false`, and the loop runs indefinitely. In a backend server, an infinite loop in a request handler will consume 100% of a CPU core and eventually cause the server to become unresponsive. Always verify that your loop has a clear exit condition.

### Mistake 2: Off-by-one error in `for` loops

**Wrong:**
```java
int[] prices = {100, 200, 300, 400, 500};  // 5 elements, indices 0-4

// Trying to access index 5 causes ArrayIndexOutOfBoundsException
for (int i = 0; i <= prices.length; i++) {  // <= is wrong here
    System.out.println(prices[i]);  // Crashes on the last iteration
}
```

**Right:**
```java
int[] prices = {100, 200, 300, 400, 500};

for (int i = 0; i < prices.length; i++) {  // < is correct
    System.out.println(prices[i]);
}

// Even better: use for-each to eliminate the possibility of off-by-one errors
for (int price : prices) {
    System.out.println(price);
}
```

**Why it is wrong:** Arrays in Java are zero-indexed. An array of length 5 has valid indices 0, 1, 2, 3, 4. Using `<= length` tries to access index 5, which does not exist. Off-by-one errors are the most common loop bug in all of programming. The for-each loop eliminates this class of error entirely.

### Mistake 3: Modifying a collection while iterating with for-each

**Wrong:**
```java
List<String> users = new ArrayList<>(List.of("Alice", "Bob", "Charlie", "Dave"));

for (String user : users) {
    if ("Bob".equals(user)) {
        users.remove(user);  // ConcurrentModificationException!
    }
}
```

**Right:**
```java
// Option 1: Use an Iterator explicitly
Iterator<String> iterator = users.iterator();
while (iterator.hasNext()) {
    String user = iterator.next();
    if ("Bob".equals(user)) {
        iterator.remove();  // Safe removal through the iterator
    }
}

// Option 2: Use removeIf (Java 8+, cleanest approach)
users.removeIf("Bob"::equals);

// Option 3: Collect items to remove in a separate list, then remove after the loop
List<String> toRemove = new ArrayList<>();
for (String user : users) {
    if ("Bob".equals(user)) {
        toRemove.add(user);
    }
}
users.removeAll(toRemove);
```

**Why it is wrong:** The enhanced for-each loop uses an `Iterator` internally. If you modify the collection directly (add or remove elements) while the iterator is active, the iterator detects the structural change and throws a `ConcurrentModificationException`. This is a safety mechanism to prevent unpredictable behavior.

### Mistake 4: Using `==` instead of `.equals()` inside loop conditions with Strings

**Wrong:**
```java
String[] commands = {"start", "stop", "pause", "quit"};

for (String command : commands) {
    if (command == "quit") {  // Unreliable for String comparison
        System.out.println("Shutting down...");
        break;
    }
}
```

**Right:**
```java
String[] commands = {"start", "stop", "pause", "quit"};

for (String command : commands) {
    if ("quit".equals(command)) {  // Always use .equals() for Strings
        System.out.println("Shutting down...");
        break;
    }
}
```

**Why it is wrong:** This is the same `==` vs `.equals()` issue from the operators note, but it is especially dangerous inside loops because the bug may only manifest with certain data. String literals from the same source file might share the same memory location (string pool), making `==` appear to work during testing. But strings from user input, database queries, or API responses will be different objects, and `==` will fail silently.

---

## Key Takeaways

> [!tip] Remember these points
> 1. Use `for` when you know the number of iterations in advance. Use `while` when the number of iterations depends on a runtime condition. Use `do-while` when the body must execute at least once.
> 2. Use the enhanced for-each loop `for (Type item : collection)` whenever you need to iterate over all elements of an array or collection without modifying it. It is cleaner and prevents off-by-one errors.
> 3. Every loop must have a clear exit condition. Verify that the loop variable is updated in a way that eventually makes the condition `false`. Infinite loops in backend servers cause outages.
> 4. `break` exits the loop entirely. `continue` skips to the next iteration. Use them sparingly. If your loop has many `break` and `continue` statements, consider refactoring the logic into a separate method.
> 5. Never modify a collection (add or remove elements) while iterating over it with a for-each loop. Use `Iterator.remove()`, `removeIf()`, or collect changes in a separate list and apply them after the loop.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: FizzBuzz (Easy)
Write a program that prints numbers from 1 to 100 with the following rules:
- If the number is divisible by 3, print "Fizz" instead of the number.
- If the number is divisible by 5, print "Buzz" instead of the number.
- If the number is divisible by both 3 and 5, print "FizzBuzz".
- Otherwise, print the number.

This is one of the most common interview screening questions. You should be able to write it in under 5 minutes.

**Hint:** Check the most specific condition first (divisible by both 3 and 5) before checking the individual conditions. Use `%` (modulus) to check divisibility.

### Exercise 2: Prime Number Checker (Medium)
Write a program that takes a number as input and determines whether it is a prime number. A prime number is a number greater than 1 that has no divisors other than 1 and itself. Use a `for` loop to check divisibility from 2 up to the square root of the number.

Then extend the program to print all prime numbers between 1 and 100 using a nested loop structure.

**Hint:** You only need to check divisors up to `Math.sqrt(n)`. If no divisor is found in that range, the number is prime. Use a `boolean` flag variable to track whether a divisor was found.

### Exercise 3: Input Validation with do-while (Medium)
Write a program that asks the user to enter a valid email address. The program should keep asking until the user enters an email that contains exactly one `@` symbol and at least one `.` after the `@`. Use a `do-while` loop because you want to ask at least once.

**Hint:** Use `scanner.nextLine()` to read the email. Use `email.indexOf("@")` and `email.lastIndexOf("@")` to check for exactly one `@`. Use `email.substring(email.indexOf("@")).contains(".")` to check for a dot after the `@`.

### Exercise 4: Matrix Operations (Hard, Optional)
Write a program that creates a 2D array (matrix) of size 4x4 and fills it with random integers between 1 and 100. Then use nested loops to:
1. Print the matrix in a formatted grid.
2. Calculate and print the sum of each row.
3. Calculate and print the sum of the main diagonal (top-left to bottom-right).
4. Find and print the maximum value in the entire matrix and its position (row, column).

**Hint:** Use `int[][] matrix = new int[4][4]` to create the 2D array. Use `Math.random()` or `java.util.Random` to generate random numbers. The main diagonal elements are at positions where row index equals column index (`matrix[i][i]`).

### Solution
For Exercise 1:
```java
public class FizzBuzz {
    public static void main(String[] args) {
        for (int i = 1; i <= 100; i++) {
            if (i % 15 == 0) {
                System.out.println("FizzBuzz");
            } else if (i % 3 == 0) {
                System.out.println("Fizz");
            } else if (i % 5 == 0) {
                System.out.println("Buzz");
            } else {
                System.out.println(i);
            }
        }
    }
}
```


For Exercise 2:
```java
import java.util.Scanner;

public class PrimeChecker {
    public static void main(String[] args) {
        // Check a single number
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter a number: ");
        int n = scanner.nextInt();

        if (isPrime(n)) {
            System.out.println(n + " is prime.");
        } else {
            System.out.println(n + " is not prime.");
        }

        // Print all primes from 1 to 100
        System.out.println("\nPrimes from 1 to 100:");
        for (int i = 2; i <= 100; i++) {
            if (isPrime(i)) {
                System.out.print(i + " ");
            }
        }
        System.out.println();
        scanner.close();
    }

    static boolean isPrime(int n) {
        if (n <= 1) return false;
        if (n <= 3) return true;
        if (n % 2 == 0) return false;

        for (int i = 3; i <= Math.sqrt(n); i += 2) {
            if (n % i == 0) {
                return false;
            }
        }
        return true;
    }
}
```

For Exercise 3:

```java
import java.util.Scanner;

public class EmailValidator {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String email;

        do {
            System.out.print("Enter a valid email address: ");
            email = scanner.nextLine().trim();

            int firstAt = email.indexOf("@");
            int lastAt = email.lastIndexOf("@");
            boolean hasExactlyOneAt = firstAt > 0 && firstAt == lastAt;
            boolean hasDotAfterAt = hasExactlyOneAt
                && email.substring(firstAt).contains(".");

            if (hasExactlyOneAt && hasDotAfterAt) {
                System.out.println("Valid email: " + email);
            } else {
                System.out.println("Invalid email. Please try again.");
            }

        } while (email.indexOf("@") != email.lastIndexOf("@")
            || email.indexOf("@") <= 0
            || !email.substring(email.indexOf("@")).contains("."));

        scanner.close();
    }
}
```

---

## Related Notes

- [[Java - Control Flow - If Else Switch]]
- [[Java - Arrays - 1D and 2D]]

---

## Resources

- [Oracle Java Tutorials: The while and do-while Statements](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/while.html) - Official documentation with examples.
- [Oracle Java Tutorials: The for Statement](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/for.html) - Official documentation covering both traditional and enhanced for loops.
- [Baeldung: Java Break and Continue](https://www.baeldung.com/java-break-continue) - Detailed guide on loop control statements with labeled examples.
- [Baeldung: Iterate Over a Collection in Java](https://www.baeldung.com/java-iterate-collection) - Comprehensive comparison of all iteration methods including Iterator, for-each, and Streams.
