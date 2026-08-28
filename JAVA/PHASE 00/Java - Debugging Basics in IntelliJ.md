---
title: "Java - Debugging Basics in IntelliJ"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - debugging
  - intellij
status: "not-started"
---

# Java - Debugging Basics in IntelliJ

> [!abstract] Overview
> Debugging is the process of finding and fixing bugs in your code. It is arguably the most important skill a backend engineer develops. You will spend more time debugging than writing new code throughout your career. IntelliJ IDEA provides a powerful visual debugger that lets you pause execution, inspect variables, step through code line by line, and evaluate expressions at runtime. This note covers the three types of bugs, how to read stack traces, and how to use every essential feature of the IntelliJ debugger.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Setting Up Environment - IntelliJ and JDK]]
> - [[Java - Control Flow - If Else Switch]]
> - [[Java - Loops - For While Do-While]]
> - [[Java - Methods - Parameters Return Types Overloading]]
> - [[Java - Basic Input Output - Scanner and System.out]]

---

## Theory

### What is Debugging?

Debugging is the systematic process of identifying, isolating, and fixing defects (bugs) in your code. The term "bug" has been used in engineering since the 1800s, but it became famous in computing in 1947 when Grace Hopper's team found an actual moth trapped in a relay of the Harvard Mark II computer. They taped the moth into the logbook with the note "First actual case of bug being found."

Debugging is not a sign of failure. Every programmer, from first-semester students to principal engineers at Google, writes buggy code. The difference between a junior and a senior engineer is not that the senior writes bug-free code. It is that the senior finds and fixes bugs faster.

### The Three Types of Bugs

**1. Compile-time errors (syntax errors)**

These are errors that the Java compiler catches before your program runs. They include missing semicolons, mismatched brackets, type mismatches, and undefined variables. The compiler tells you exactly which line has the error and what is wrong.

```java
int x = "hello";  // COMPILATION ERROR: incompatible types
System.out.println(y);  // COMPILATION ERROR: cannot find symbol 'y'
if (x = 5) { }  // COMPILATION ERROR: incompatible types (int cannot be boolean)
```

These are the easiest bugs to fix because the compiler does the work for you. IntelliJ highlights them in red before you even compile.

**2. Runtime errors (exceptions)**

These occur while your program is running. The code compiles successfully, but something goes wrong during execution. The JVM throws an exception, which, if not caught, crashes your program and prints a stack trace.

```java
int[] arr = {1, 2, 3};
System.out.println(arr[5]);  // RUNTIME ERROR: ArrayIndexOutOfBoundsException

int result = 10 / 0;  // RUNTIME ERROR: ArithmeticException

String name = null;
System.out.println(name.length());  // RUNTIME ERROR: NullPointerException
```

Runtime errors are harder than compile-time errors because the compiler cannot predict them. They depend on the actual data your program processes at runtime.

**3. Logical errors**

These are the hardest bugs to find. The code compiles, runs without crashing, but produces the wrong result. There is no error message, no stack trace, and no red highlighting. The only clue is that the output is incorrect.

```java
// Bug: should calculate average but calculates sum instead
double average(int[] scores) {
    int sum = 0;
    for (int score : scores) {
        sum += score;
    }
    return sum;  // LOGICAL ERROR: forgot to divide by scores.length
}
```

Logical errors require you to trace the execution of your program and compare what the code actually does with what you intended it to do. This is where the debugger becomes essential.

### Stack Traces

When a runtime error occurs, the JVM prints a stack trace (also called a stack backtrace). A stack trace shows the sequence of method calls that led to the error, starting from the point of failure and going back to the `main` method.

```text
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "String.length()" because "name" is null
    at com.company.orderservice.service.UserService.validateName(UserService.java:25)
    at com.company.orderservice.service.UserService.createUser(UserService.java:15)
    at com.company.orderservice.controller.UserController.handleRequest(UserController.java:42)
    at com.company.orderservice.Application.main(Application.java:10)
```

How to read a stack trace:

- **First line**: The exception type and message. This tells you WHAT went wrong. In this case, a `NullPointerException` because `name` is null.
- **Second line (top of the stack)**: The exact location where the error occurred. This tells you WHERE it went wrong. `UserService.java:25` means line 25 of the `UserService` class, inside the `validateName` method.
- **Remaining lines**: The call chain that led to the error. `main` called `handleRequest`, which called `createUser`, which called `validateName`, which crashed. This tells you HOW the program reached the error.

Read stack traces from top to bottom. The first line of the stack (after the exception message) is the most important. It is the exact line where the error occurred. The lines below it show the path the program took to get there.

> [!tip] Key Insight
> A stack trace is a snapshot of the call stack at the moment the exception was thrown. Each line represents a stack frame. The top frame is the method that crashed. The bottom frame is usually `main` or a framework entry point (like Spring's DispatcherServlet). In a Spring Boot backend, stack traces can be 50-100 lines long because the framework adds many layers between the HTTP request and your code. Learn to ignore the framework frames and focus on the lines that reference your package name (e.g., `com.company.orderservice`).

### The IntelliJ Debugger

The IntelliJ debugger lets you run your program in a controlled mode where you can pause execution at any line, inspect the state of all variables, and step through the code one line at a time. This is far more powerful than adding `System.out.println()` statements everywhere (a technique called "print debugging" that beginners rely on but professionals avoid for complex bugs).

**Debugger terminology:**

| Term | Meaning |
|------|---------|
| Breakpoint | A marker you place on a line of code telling the debugger to pause execution when that line is reached. |
| Debug mode | Running the program with the debugger attached (as opposed to normal "Run" mode). |
| Step Over | Execute the current line and move to the next line in the same method. If the current line calls a method, execute the entire method without stepping into it. |
| Step Into | If the current line calls a method, move the debugger into that method so you can step through it line by line. |
| Step Out | Execute the rest of the current method and pause at the line after the method call in the calling method. |
| Resume | Continue execution until the next breakpoint or until the program ends. |
| Watches | Variables or expressions you want to monitor continuously as you step through the code. |
| Evaluate Expression | A tool that lets you type any Java expression and see its result at the current point of execution. |
| Call Stack (Frames) | The list of active method calls at the current point of execution. |

**Keyboard shortcuts (Mac / Linux):**

| Action | Mac | Linux |
|--------|-----|-------|
| Toggle breakpoint | `Cmd + F8` | `Ctrl + F8` |
| Debug program | `Ctrl + Shift + D` | `Shift + F9` |
| Step Over | `F8` | `F8` |
| Step Into | `F7` | `F7` |
| Step Out | `Shift + F8` | `Shift + F8` |
| Resume | `Cmd + Option + R` | `F9` |
| Evaluate Expression | `Option + F8` | `Alt + F8` |
| Stop | `Cmd + F2` | `Ctrl + F2` |

---

## Syntax and Basic Examples

### Example 1: Setting breakpoints and stepping through code

Consider this buggy program that is supposed to calculate the factorial of a number but returns the wrong result:

```java
public class FactorialDebugger {

    static int factorial(int n) {
        int result = 0;  // BUG: should be 1, not 0
        for (int i = 1; i <= n; i++) {
            result *= i;
        }
        return result;
    }

    public static void main(String[] args) {
        int number = 5;
        int answer = factorial(number);
        System.out.println(number + "! = " + answer);
        // Expected: 120
        // Actual: 0
    }
}
```

How to debug this in IntelliJ:

1. **Set a breakpoint**: Click in the gutter (left margin) next to the line `int answer = factorial(number);` in `main()`. A red dot appears.

2. **Start debugging**: Click the bug icon (green bug with a play button) in the toolbar, or press `Shift + F9` (Linux) / `Ctrl + Shift + D` (Mac). Do NOT click the green play button (that runs without the debugger).

3. The program pauses at the breakpoint. The Debug tool window opens at the bottom of IntelliJ. You can see:
   - **Frames panel** (left): Shows the call stack. Currently `main()`.
   - **Variables panel** (center): Shows `number = 5`, `answer = 0` (not yet assigned).
   - **Code editor**: The current line is highlighted in blue.

4. **Step Into (F7)**: Since the current line calls `factorial()`, pressing Step Into takes you inside the `factorial` method. The Variables panel now shows `n = 5`, `result = 0`.

5. **Step Over (F8)**: Execute `int result = 0;`. The Variables panel shows `result = 0`. Notice that `result` starts at 0, which is suspicious. A factorial calculation should start at 1 because `n! = 1 * 2 * 3 * ... * n`. Starting at 0 means every multiplication will produce 0.

6. **You found the bug!** Change `int result = 0;` to `int result = 1;`, stop the debugger, and run again. The output is now `5! = 120`.

### Example 2: Using watches and conditional breakpoints

```java
public class SearchDebugger {

    static int findFirstNegative(int[] numbers) {
        for (int i = 0; i < numbers.length; i++) {
            if (numbers[i] < 0) {
                return i;
            }
        }
        return -1;
    }

    public static void main(String[] args) {
        int[] data = {10, 25, -3, 42, -7, 15, -1};
        int index = findFirstNegative(data);
        System.out.println("First negative at index: " + index);
    }
}
```

Debugging with watches:

1. Set a breakpoint on the line `if (numbers[i] < 0)` inside the loop.
2. Start debugging. The program pauses on the first iteration (`i=0`).
3. In the Variables panel, right-click and select **Add to Watches** for `numbers[i]` and `i`.
4. The Watches panel now shows the current values. Press `F9` (Resume) to continue to the next breakpoint hit.
5. On each iteration, the Watches panel updates automatically. You can see `i` incrementing and `numbers[i]` changing.

Conditional breakpoints:

If the array had 10,000 elements, pressing Resume thousands of times would be tedious. Instead, right-click the breakpoint red dot and set a condition:

```text
numbers[i] < 0
```

Now the debugger only pauses when the condition is true (when a negative number is found). This is extremely useful for debugging loops in backend systems that process large datasets.

### Example 3: Evaluating expressions at runtime

```java
public class ExpressionDebugger {

    static double calculateDiscount(double price, String customerTier, boolean isHoliday) {
        double discountRate = 0.0;

        if ("GOLD".equals(customerTier)) {
            discountRate = 0.20;
        } else if ("SILVER".equals(customerTier)) {
            discountRate = 0.10;
        }

        if (isHoliday) {
            discountRate += 0.05;
        }

        return price * discountRate;
    }

    public static void main(String[] args) {
        double price = 5000.0;
        String tier = "GOLD";
        boolean holiday = true;

        double discount = calculateDiscount(price, tier, holiday);
        System.out.println("Discount: " + discount);
    }
}
```

Using Evaluate Expression:

1. Set a breakpoint on the line `return price * discountRate;`.
2. Start debugging and step through until you reach the `return` statement.
3. Press `Alt + F8` (Linux) or `Option + F8` (Mac) to open the Evaluate Expression dialog.
4. Type any valid Java expression using the current variables:
   - `price * 0.25` -> evaluates to `1250.0`
   - `discountRate > 0.15` -> evaluates to `true`
   - `customerTier.toLowerCase()` -> evaluates to `"gold"`

This lets you test hypotheses about the bug without modifying the code and restarting.

### Example 4: Reading and understanding a stack trace

```java
public class StackTraceDemo {

    static int divide(int a, int b) {
        return a / b;  // Line 4: ArithmeticException when b is 0
    }

    static int calculateAverage(int[] scores) {
        int sum = 0;
        for (int score : scores) {
            sum += score;
        }
        return divide(sum, scores.length);  // Line 11: crashes if array is empty
    }

    static void processStudent(String name, int[] scores) {
        System.out.println("Processing: " + name);
        int avg = calculateAverage(scores);  // Line 16
        System.out.println("Average: " + avg);
    }

    public static void main(String[] args) {
        int[] emptyScores = {};
        processStudent("Saad", emptyScores);  // Line 22
    }
}
```

Output when run:

```text
Processing: Saad
Exception in thread "main" java.lang.ArithmeticException: / by zero
    at StackTraceDemo.divide(StackTraceDemo.java:4)
    at StackTraceDemo.calculateAverage(StackTraceDemo.java:11)
    at StackTraceDemo.processStudent(StackTraceDemo.java:16)
    at StackTraceDemo.main(StackTraceDemo.java:22)
```

How to read this:

- **What happened?** `ArithmeticException: / by zero`. Someone divided by zero.
- **Where exactly?** `StackTraceDemo.java:4`, inside the `divide` method. The line `return a / b;` executed with `b = 0`.
- **How did we get there?** `main` (line 22) called `processStudent` (line 16), which called `calculateAverage` (line 11), which called `divide` (line 4). The empty array has `length = 0`, which was passed as the divisor.
- **The fix:** Add a check in `calculateAverage` to handle empty arrays before calling `divide`.

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Debugging in a Spring Boot backend is more complex than debugging a console application because the framework adds many layers between the HTTP request and your code. Here are three realistic scenarios.

### Scenario 1: Debugging a NullPointerException in a Spring Boot service

NullPointerExceptions are the most common runtime error in Java backend systems. They occur when you try to call a method on a null reference.

```java
package com.company.orderservice.service;

public class OrderService {

    public OrderResponse getOrderDetails(Long orderId) {
        // This line might return null if the order does not exist in the database
        Order order = orderRepository.findById(orderId).orElse(null);

        // BUG: If order is null, the next line throws NullPointerException
        String customerName = order.getCustomer().getName();
        // The stack trace will point to THIS line.
        // But the real bug is on the line above: we did not check for null.

        BigDecimal total = order.getTotal();
        String status = order.getStatus().name();

        return new OrderResponse(orderId, customerName, total, status);
    }
}
```

The stack trace in production:

```text
java.lang.NullPointerException: Cannot invoke "Order.getCustomer()" because "order" is null
    at com.company.orderservice.service.OrderService.getOrderDetails(OrderService.java:10)
    at com.company.orderservice.controller.OrderController.getOrder(OrderController.java:25)
    at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:897)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
    ... 45 more
```

How to debug this:

- **Focus on your code, not the framework.** Ignore all lines that start with `org.springframework`, `org.apache`, or `java.base`. Focus on the lines in your package: `com.company.orderservice`.
- **The top frame is the crash site.** `OrderService.java:10` is where the NPE occurred. The variable `order` is null.
- **Trace backward.** Why is `order` null? Because `findById(orderId)` did not find a matching record. The `orderId` passed by the client does not exist in the database.

The fix:

```java
public OrderResponse getOrderDetails(Long orderId) {
    Order order = orderRepository.findById(orderId)
        .orElseThrow(() -> new ResourceNotFoundException("Order not found: " + orderId));
    // Now order is guaranteed to be non-null.
    // If it was null, an exception with a clear message is thrown instead of a cryptic NPE.

    String customerName = order.getCustomer().getName();
    BigDecimal total = order.getTotal();
    String status = order.getStatus().name();

    return new OrderResponse(orderId, customerName, total, status);
}
```

### Scenario 2: Debugging a logical error in pricing calculation

A customer reports that their order total is wrong. The code compiles and runs without errors, but the result is incorrect. This is a logical error, and the debugger is the best tool to find it.

```java
package com.company.orderservice.service;

public class PricingService {

    public BigDecimal calculateOrderTotal(List<OrderItem> items, String couponCode) {
        BigDecimal subtotal = BigDecimal.ZERO;

        for (OrderItem item : items) {
            BigDecimal itemTotal = item.getUnitPrice()
                .multiply(new BigDecimal(item.getQuantity()));
            subtotal = subtotal.add(itemTotal);
        }

        BigDecimal discount = applyDiscount(subtotal, couponCode);
        BigDecimal afterDiscount = subtotal.subtract(discount);

        // BUG: Tax is calculated on the original subtotal, not the discounted amount.
        // Customers are being overcharged.
        BigDecimal tax = subtotal.multiply(new BigDecimal("0.15"));

        return afterDiscount.add(tax);
    }

    private BigDecimal applyDiscount(BigDecimal subtotal, String couponCode) {
        if ("SAVE20".equals(couponCode)) {
            return subtotal.multiply(new BigDecimal("0.20"));
        }
        return BigDecimal.ZERO;
    }
}
```

How to debug this:

1. Set a breakpoint on the line `BigDecimal tax = subtotal.multiply(...)`.
2. Start debugging with a test case: 2 items at 1000 BDT each, coupon "SAVE20".
3. Inspect variables when the breakpoint hits:
   - `subtotal = 2000.00` (correct: 2 x 1000)
   - `discount = 400.00` (correct: 20% of 2000)
   - `afterDiscount = 1600.00` (correct: 2000 - 400)
   - `tax = 300.00` (WRONG! 15% of 2000 instead of 15% of 1600)
4. The expected tax should be 15% of 1600 = 240.00. The customer is being overcharged by 60 BDT.
5. **The fix:** Change `subtotal.multiply(...)` to `afterDiscount.multiply(...)`.

This type of bug is invisible to the compiler and does not throw any exceptions. Without the debugger, you would have to add `System.out.println()` statements throughout the method, recompile, rerun, and try to trace the values manually. The debugger lets you see all variable values instantly at any point in the execution.

### Scenario 3: Debugging with IntelliJ's HTTP Client

Spring Boot projects in IntelliJ Ultimate include an HTTP Client that lets you send real HTTP requests to your running backend and debug the server-side code simultaneously.

```text
### Create a new order
POST http://localhost:8080/api/v1/orders
Content-Type: application/json

{
  "userId": 12345,
  "items": [
    {"productId": 1, "quantity": 2, "unitPrice": 1500.00},
    {"productId": 5, "quantity": 1, "unitPrice": 3200.00}
  ],
  "couponCode": "SAVE20"
}
```

Debugging workflow:

1. Set breakpoints in your controller, service, and repository methods.
2. Start the Spring Boot application in debug mode (bug icon).
3. Run the HTTP request from IntelliJ's HTTP Client.
4. The debugger pauses at your first breakpoint. You can inspect the deserialized request body, step through the business logic, and watch the database query being constructed.

This is the closest simulation of a real debugging session you will experience in a professional backend team.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using println debugging instead of the debugger

**Wrong approach:**

```java
public Order processOrder(Order order) {
    System.out.println("DEBUG 1: order = " + order);
    System.out.println("DEBUG 2: items = " + order.getItems());
    System.out.println("DEBUG 3: total = " + order.getTotal());
    System.out.println("DEBUG 4: discount = " + discount);
    System.out.println("DEBUG 5: tax = " + tax);
    // ... 20 more println statements scattered throughout the method ...
}
```

**Right approach:** Set a breakpoint at the beginning of the method and use the debugger to inspect all variables in the Variables panel. Step through the code line by line. No code modifications needed.

**Why it is wrong:** Println debugging requires you to modify the code, recompile, and rerun for every investigation. You cannot inspect intermediate values inside complex expressions. You cannot evaluate arbitrary expressions at runtime. You have to remember to remove all the `println` statements before committing (and you will forget). The debugger does all of this without touching your code.

### Mistake 2: Ignoring the stack trace and guessing the bug

**Wrong approach:** Seeing a `NullPointerException`, guessing that "something is null somewhere," and randomly adding null checks throughout the code.

**Right approach:** Read the stack trace carefully. The first line tells you the exact file, method, and line number where the error occurred. Go to that line, identify which variable is null, and then trace backward to understand why it is null.

**Why it is wrong:** Random guessing wastes time and often introduces new bugs. A stack trace is a precise diagnostic tool. It tells you exactly where the problem is. Ignoring it is like ignoring a GPS and driving randomly when you are lost.

### Mistake 3: Stepping into framework code

**Wrong approach:** When debugging a Spring Boot application, pressing Step Into (F7) on a Spring method call and ending up deep inside the Spring framework source code, trying to understand how Spring's internal DispatcherServlet works.

**Right approach:** Use Step Over (F8) for framework method calls. Only Step Into methods that you wrote yourself. If you accidentally step into framework code, use Step Out (Shift + F8) to return to your code immediately.

**Why it is wrong:** Spring's internal code is complex and not relevant to your bug. The bug is in your code, not in Spring. Stepping into framework code wastes time and confuses beginners. The framework has been tested by thousands of developers; your code has not.

### Mistake 4: Not reproducing the bug before debugging

**Wrong approach:** A user reports a bug. You immediately open the code and start reading it, trying to spot the error by visual inspection.

**Right approach:**

1. **Reproduce the bug.** Get the exact input that causes the error. If the user says "the order total is wrong," find out which order, what items, and what coupon they used.
2. **Write a test case** or set up the exact scenario in your development environment.
3. **Set breakpoints** and run the debugger with the reproducing input.
4. **Observe the actual values** and compare them with the expected values.

**Why it is wrong:** You cannot debug a bug you cannot reproduce. Reading code by eye is slow and unreliable for logical errors. The debugger shows you what the code actually does, which is often different from what you think it does.

---

## Key Takeaways

> [!tip] Remember these points
> 1. There are three types of bugs: compile-time (caught by the compiler, easiest to fix), runtime (exceptions during execution, identified by stack traces), and logical (wrong output, no error message, requires the debugger).
> 2. Stack traces are your primary diagnostic tool for runtime errors. Read them from top to bottom. Focus on the lines in your package, not the framework lines. The first line after the exception message is the exact crash location.
> 3. The IntelliJ debugger lets you pause execution (breakpoints), inspect variables (Variables panel), step through code (Step Over/Into/Out), monitor expressions (Watches), and test hypotheses (Evaluate Expression). Learn the keyboard shortcuts to debug efficiently.
> 4. Conditional breakpoints pause only when a condition is true. Use them for debugging loops that process large datasets. Watches let you monitor specific variables or expressions without cluttering the Variables panel.
> 5. The debugging workflow is: reproduce the bug, set breakpoints, run in debug mode, inspect state, identify the discrepancy, fix the code, verify the fix. Do not guess. Do not add random println statements. Use the tools.

---

## Phase 0 Summary

> [!success] Congratulations! You have completed Phase 0: Programming Foundations.

You now understand:

- How Java works (JVM, JRE, JDK)
- How to set up a professional development environment
- Variables, data types, and type casting
- Operators (arithmetic, relational, logical)
- Control flow (if-else, switch)
- Loops (for, while, do-while, for-each)
- Arrays (1D and 2D)
- Strings and String manipulation
- Methods, parameters, return types, and overloading
- Console I/O with Scanner and System.out
- How to debug code using IntelliJ

**Next: Phase 1 - Java Core and OOP**

In Phase 1, you will learn Object-Oriented Programming: classes, objects, inheritance, polymorphism, abstraction, encapsulation, exception handling, the Collections Framework, and Java 8 features like lambdas and streams. This is the foundation of all Spring Boot development.

---

## Exercise

> [!abstract] Practice before moving to Phase 1

### Exercise 1: Debug a Broken Calculator (Easy)

The following program is supposed to be a simple calculator but produces wrong results for all operations. Set breakpoints, step through the code, and find all three bugs. Fix them and verify with the debugger.

```java
public class BrokenCalculator {
    static double calculate(double a, double b, String operation) {
        double result = 0;
        if (operation == "add") {
            result = a - b;
        } else if (operation == "subtract") {
            result = a + b;
        } else if (operation.equals("multiply")) {
            result = a * b;
        } else if (operation.equals("divide")) {
            result = a / b;
        }
        return result;
    }

    public static void main(String[] args) {
        System.out.println("5 + 3 = " + calculate(5, 3, "add"));
        System.out.println("10 - 4 = " + calculate(10, 4, "subtract"));
        System.out.println("6 * 7 = " + calculate(6, 7, "multiply"));
        System.out.println("20 / 4 = " + calculate(20, 4, "divide"));
    }
}
```

**Hint:** There are three bugs. Two are logical (swapped operations) and one is a String comparison issue. Use the debugger to watch the `result` variable after each `if` branch.

### Exercise 2: Debug an Array Processing Bug (Medium)

The following program is supposed to find the second largest number in an array but returns the wrong answer. Use the IntelliJ debugger to step through the loop and watch how `largest` and `secondLargest` change on each iteration.

```java
public class SecondLargest {
    static int findSecondLargest(int[] arr) {
        int largest = arr[0];
        int secondLargest = arr[0];  // BUG: should be initialized differently

        for (int i = 1; i < arr.length; i++) {
            if (arr[i] > largest) {
                secondLargest = largest;
                largest = arr[i];
            } else if (arr[i] > secondLargest) {
                secondLargest = arr[i];
            }
        }
        return secondLargest;
    }

    public static void main(String[] args) {
        int[] numbers = {10, 5, 20, 8, 15};
        System.out.println("Second largest: " + findSecondLargest(numbers));
        // Expected: 15
        // What does it actually print?
    }
}
```

**Hint:** Set a breakpoint inside the `for` loop. Add watches for `arr[i]`, `largest`, and `secondLargest`. Step through each iteration and observe when the logic fails.

### Exercise 3: Read and Interpret a Stack Trace (Medium)

Run the following program and read the stack trace. Answer these questions:

1. What exception was thrown?
2. On which exact line did the crash occur?
3. What was the sequence of method calls that led to the crash?
4. What is the root cause of the bug?
5. How would you fix it?

```java
public class StackTraceExercise {
    static String getFirstCharacter(String text) {
        return text.substring(0, 1);
    }

    static String formatName(String firstName, String lastName) {
        String firstInitial = getFirstCharacter(firstName);
        String lastInitial = getFirstCharacter(lastName);
        return firstInitial + ". " + lastInitial + ".";
    }

    static void printGreeting(String fullName) {
        String[] parts = fullName.split(" ");
        String initials = formatName(parts[0], parts[2]);  // What if there is no middle name?
        System.out.println("Hello, " + initials);
    }

    public static void main(String[] args) {
        printGreeting("Abdullah Saad");  // Only two parts, no middle name
    }
}
```

**Hint:** The stack trace will show an `ArrayIndexOutOfBoundsException`. Trace the call chain from `main` to the crash site and identify which array access is invalid.

### Exercise 4: Debug a Real-World Scenario (Hard, Optional)

The following program simulates a simple e-commerce checkout. It compiles and runs but produces incorrect totals for some orders. Use the debugger to find the logical error.

```java
import java.util.Arrays;

public class CheckoutDebugger {
    static double calculateTotal(double[] prices, int[] quantities, String[] categories) {
        double total = 0;

        for (int i = 0; i < prices.length; i++) {
            double itemTotal = prices[i] * quantities[i];

            // Apply category discount
            if ("Electronics".equals(categories[i])) {
                itemTotal *= 0.90;  // 10% off electronics
            } else if ("Books".equals(categories[i])) {
                itemTotal *= 0.85;  // 15% off books
            }

            total += itemTotal;
        }

        // Apply bulk discount: 5% off if more than 10 total items
        int totalItems = 0;
        for (int qty : quantities) {
            totalItems += qty;
        }
        if (totalItems > 10) {
            total *= 0.95;
        }

        // Add delivery charge
        if (total < 1000) {
            total += 120;  // 120 BDT delivery for orders under 1000
        }

        return total;
    }

    public static void main(String[] args) {
        double[] prices = {25000, 350, 1200, 450};
        int[] quantities = {1, 3, 2, 5};
        String[] categories = {"Electronics", "Books", "Clothing", "Books"};

        double total = calculateTotal(prices, quantities, categories);
        System.out.printf("Order Total: %,.2f BDT%n", total);

        // Manually calculate the expected total:
        // Laptop: 25000 * 1 * 0.90 = 22500
        // Books: 350 * 3 * 0.85 = 892.50
        // Clothing: 1200 * 2 = 2400
        // Books: 450 * 5 * 0.85 = 1912.50
        // Subtotal: 27705.00
        // Total items: 11 (> 10, so 5% bulk discount)
        // After bulk: 27705 * 0.95 = 26319.75
        // Delivery: free (total > 1000)
        // Expected: 26319.75
        // Does the program output match?
    }
}
```

**Hint:** Set a breakpoint inside the `for` loop and watch `itemTotal` for each item. Then watch `total` after the bulk discount and delivery charge. Compare with the manual calculation in the comments.

### Solution
For Exercise 1:

The three bugs are:

1. `operation == "add"` uses `==` instead of `.equals()` for String comparison. This will always be `false` for Strings created at runtime.
2. The "add" branch performs subtraction (`a - b`) instead of addition (`a + b`).
3. The "subtract" branch performs addition (`a + b`) instead of subtraction (`a - b`).

Fixed code:

```java
static double calculate(double a, double b, String operation) {
    if ("add".equals(operation)) {
        return a + b;
    } else if ("subtract".equals(operation)) {
        return a - b;
    } else if ("multiply".equals(operation)) {
        return a * b;
    } else if ("divide".equals(operation)) {
        return a / b;
    }
    throw new IllegalArgumentException("Unknown operation: " + operation);
}
```

For Exercise 3:
- **Exception:** `ArrayIndexOutOfBoundsException: Index 2 out of bounds for length 2`
- **Crash line:** `StackTraceExercise.java:14` inside `printGreeting`, at `parts[2]`.
- **Call chain:** `main` (line 21) -> `printGreeting` (line 14) -> crash at `parts[2]`.
- **Root cause:** `"Abdullah Saad"` splits into only 2 parts: `["Abdullah", "Saad"]`. The code tries to access `parts[2]` (a third part), which does not exist.
- **Fix:** Check `parts.length` before accessing indices. Handle names with and without middle names.

---

## Related Notes

- [[Java - Basic Input Output - Scanner and System.out]]
- [[Java - Classes and Objects]] (Phase 1, next note)

---

## Resources

- [IntelliJ IDEA Debugging Guide](https://www.jetbrains.com/help/idea/debugging-code.html) - Official JetBrains documentation covering all debugger features with screenshots.
- [IntelliJ IDEA Breakpoints](https://www.jetbrains.com/help/idea/using-breakpoints.html) - Detailed guide on breakpoint types: line, conditional, exception, field watchpoints, and method breakpoints.
- [Baeldung: Debugging in IntelliJ IDEA](https://www.baeldung.com/intellij-debugging) - Step-by-step tutorial with examples.
- [Oracle Java Tutorials: Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/) - Official guide to understanding exceptions and stack traces. You will study this in depth in Phase 1.
