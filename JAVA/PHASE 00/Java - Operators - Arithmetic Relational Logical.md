---
title: "Java - Operators - Arithmetic Relational Logical"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - operators
status: "not-started"
---

# Java - Operators - Arithmetic Relational Logical

> [!abstract] Overview
> Operators are symbols that perform operations on variables and values. Java provides arithmetic operators for math, relational operators for comparison, logical operators for combining conditions, and several others. Every backend system uses operators constantly: calculating totals, validating user input, checking permissions, and controlling the flow of business logic.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]

---

## Theory

### What is an Operator?

An operator is a symbol that tells the compiler or JVM to perform a specific operation on one or more values. The values that operators act on are called **operands**.

```java
int result = 10 + 5;
//          ^^ ^ ^^
//       operand operator operand
//       The '+' operator adds the two operands.
```

Java operators are grouped into categories based on what they do and how many operands they require:

- **Unary operators**: Act on a single operand (e.g., `-x`, `!flag`, `count++`).
- **Binary operators**: Act on two operands (e.g., `a + b`, `x > y`, `p && q`).
- **Ternary operator**: Acts on three operands (e.g., `condition ? valueIfTrue : valueIfFalse`).

### Arithmetic Operators

Arithmetic operators perform mathematical calculations. These are the operators you learned in school, plus a few extras.

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `10 + 3` | `13` |
| `-` | Subtraction | `10 - 3` | `7` |
| `*` | Multiplication | `10 * 3` | `30` |
| `/` | Division | `10 / 3` | `3` (integer division truncates) |
| `%` | Modulus (remainder) | `10 % 3` | `1` |
| `++` | Increment | `x++` or `++x` | Adds 1 to x |
| `--` | Decrement | `x--` or `--x` | Subtracts 1 from x |

**Integer Division**: When you divide two integers in Java, the result is an integer. The decimal part is truncated (cut off), not rounded. `10 / 3` gives `3`, not `3.333`. This is one of the most common sources of bugs for beginners.

**Modulus**: The `%` operator returns the remainder after division. `10 % 3` is `1` because 10 divided by 3 is 3 with a remainder of 1. Modulus is extremely useful in backend programming for tasks like checking if a number is even or odd (`n % 2 == 0`), implementing circular buffers, and pagination logic.

**Prefix vs Postfix Increment/Decrement**:

- `++x` (prefix): Increment first, then use the value.
- `x++` (postfix): Use the value first, then increment.

```java
int a = 5;
int b = ++a;  // a becomes 6, then b is assigned 6. Result: a=6, b=6.

int c = 5;
int d = c++;  // d is assigned 5, then c becomes 6. Result: c=6, d=5.
```

### Relational (Comparison) Operators

Relational operators compare two values and return a `boolean` result: either `true` or `false`.

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `==` | Equal to | `5 == 5` | `true` |
| `!=` | Not equal to | `5 != 3` | `true` |
| `>` | Greater than | `5 > 3` | `true` |
| `<` | Less than | `5 < 3` | `false` |
| `>=` | Greater than or equal to | `5 >= 5` | `true` |
| `<=` | Less than or equal to | `3 <= 5` | `true` |

> [!warning] Critical Distinction: == vs .equals()
> The `==` operator checks if two **references** point to the exact same object in memory when used with reference types like `String`. It does NOT check if the contents are the same. For comparing the actual content of objects, you must use the `.equals()` method. This is the single most common bug related to operators in Java.

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);       // false (different objects in memory)
System.out.println(a.equals(b));  // true (same content)
```

For primitive types (`int`, `double`, `boolean`, etc.), `==` works correctly because primitives store values directly, not references.

### Logical Operators

Logical operators combine multiple boolean expressions. They are the foundation of all conditional logic in your backend code.

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `&&` | Logical AND | `true && false` | `false` |
| `\|\|` | Logical OR | `true \|\| false` | `true` |
| `!` | Logical NOT | `!true` | `false` |

**Short-Circuit Evaluation**: Both `&&` and `||` use short-circuit evaluation, which means they stop evaluating as soon as the result is determined.

- `&&`: If the left side is `false`, the right side is never evaluated because the entire expression must be `false`.
- `||`: If the left side is `true`, the right side is never evaluated because the entire expression must be `true`.

```java
int x = 5;
// The second condition is never checked because the first is false.
// This prevents a NullPointerException or division by zero.
if (x > 10 && (100 / 0) > 1) {
    // This block is never reached, and no ArithmeticException is thrown.
}
```

This behavior is not just an optimization. It is a safety feature that backend developers rely on to prevent errors.

### Assignment Operators

Assignment operators assign values to variables. The compound assignment operators combine an arithmetic operation with assignment.

| Operator | Example | Equivalent To |
|----------|---------|---------------|
| `=` | `x = 5` | Assign 5 to x |
| `+=` | `x += 3` | `x = x + 3` |
| `-=` | `x -= 3` | `x = x - 3` |
| `*=` | `x *= 3` | `x = x * 3` |
| `/=` | `x /= 3` | `x = x / 3` |
| `%=` | `x %= 3` | `x = x % 3` |

### Ternary Operator

The ternary operator is a compact alternative to an `if-else` statement for simple conditions.

```java
// Syntax: condition ? valueIfTrue : valueIfFalse

int age = 20;
String status = (age >= 18) ? "Adult" : "Minor";
// status is "Adult"
```

Use the ternary operator only for simple, readable expressions. If the logic is complex, use a regular `if-else` block instead. Readability matters more than cleverness in backend code.

### Operator Precedence

When an expression contains multiple operators, Java evaluates them in a specific order called **precedence**. Higher precedence operators are evaluated first.

From highest to lowest precedence (simplified):

1. Postfix: `x++`, `x--`
2. Unary: `++x`, `--x`, `!`, `-` (negation)
3. Multiplicative: `*`, `/`, `%`
4. Additive: `+`, `-`
5. Relational: `<`, `>`, `<=`, `>=`
6. Equality: `==`, `!=`
7. Logical AND: `&&`
8. Logical OR: `||`
9. Ternary: `?:`
10. Assignment: `=`, `+=`, `-=`, etc.

When in doubt, use parentheses to make the order explicit. Parentheses have the highest precedence and make your code easier to read.

```java
// Unclear: what is evaluated first?
boolean result = a + b > c * d && e == f;

// Clear: parentheses make the intent obvious
boolean result = ((a + b) > (c * d)) && (e == f);
```

### How Operators Work Internally

At the bytecode level, the JVM uses a stack-based model for arithmetic. When you write `int result = a + b`, the JVM:

1. Pushes the value of `a` onto the stack.
2. Pushes the value of `b` onto the stack.
3. Executes the `iadd` instruction, which pops both values, adds them, and pushes the result.
4. Stores the result into the local variable `result` using `istore`.

For relational operators, the JVM uses comparison instructions like `if_icmpgt` (if integer compare greater than) that compare two values on the stack and jump to a different bytecode instruction based on the result. This is how `if` statements work at the lowest level.

> [!tip] Key Insight
> Operators are not just syntax. They map directly to JVM bytecode instructions. Understanding that `+` on integers becomes `iadd` and `+` on Strings becomes a `StringBuilder.append()` call helps you write more efficient code. String concatenation in a loop using `+` creates a new StringBuilder object on every iteration, which is wasteful. Use `StringBuilder` explicitly for loops.

---

## Syntax and Basic Examples

### Example 1: Arithmetic Operators

```java
public class ArithmeticDemo {
    public static void main(String[] args) {
        int a = 17;
        int b = 5;

        System.out.println("a + b = " + (a + b));   // 22
        System.out.println("a - b = " + (a - b));   // 12
        System.out.println("a * b = " + (a * b));   // 85
        System.out.println("a / b = " + (a / b));   // 3 (NOT 3.4, integer division!)
        System.out.println("a % b = " + (a % b));   // 2 (remainder)

        // To get a decimal result, cast one operand to double
        System.out.println("a / b (decimal) = " + ((double) a / b));  // 3.4

        // Increment and decrement
        int count = 10;
        System.out.println("count++ = " + count++);  // Prints 10, then count becomes 11
        System.out.println("count is now = " + count); // 11
        System.out.println("++count = " + ++count);  // count becomes 12, then prints 12
    }
}
```

**Output:**
```
a + b = 22
a - b = 12
a * b = 85
a / b = 3
a % b = 2
a / b (decimal) = 3.4
count++ = 10
count is now = 11
++count = 12
```

### Example 2: Relational and Logical Operators

```java
public class ComparisonDemo {
    public static void main(String[] args) {
        int age = 22;
        double cgpa = 3.5;
        boolean hasInternship = true;

        // Relational operators
        System.out.println("age >= 18: " + (age >= 18));        // true
        System.out.println("cgpa > 3.8: " + (cgpa > 3.8));      // false
        System.out.println("age != 20: " + (age != 20));         // true

        // Logical operators
        boolean eligibleForJob = age >= 18 && cgpa >= 3.0 && hasInternship;
        System.out.println("Eligible for job: " + eligibleForJob);  // true

        boolean canApplyForScholarship = cgpa >= 3.8 || hasInternship;
        System.out.println("Can apply for scholarship: " + canApplyForScholarship);  // true

        boolean isNotGraduated = !hasInternship;
        System.out.println("Is not graduated: " + isNotGraduated);  // false

        // String comparison (the right way)
        String role = "admin";
        System.out.println("role == \"admin\": " + (role == "admin"));           // true (string pool)
        System.out.println("role.equals(\"admin\"): " + role.equals("admin"));   // true (always reliable)

        String inputRole = new String("admin");
        System.out.println("inputRole == \"admin\": " + (inputRole == "admin"));         // false (different object!)
        System.out.println("inputRole.equals(\"admin\"): " + inputRole.equals("admin")); // true (correct way)
    }
}
```

**Output:**
```
age >= 18: true
cgpa > 3.8: false
age != 20: true
Eligible for job: true
Can apply for scholarship: true
Is not graduated: false
role == "admin": true
role.equals("admin"): true
inputRole == "admin": false
inputRole.equals("admin"): true
```

### Example 3: Ternary and Assignment Operators

```java
public class TernaryDemo {
    public static void main(String[] args) {
        int score = 75;

        // Ternary operator
        String grade = (score >= 80) ? "A" : (score >= 70) ? "B" : (score >= 60) ? "C" : "F";
        System.out.println("Grade: " + grade);  // B

        // Assignment operators
        int total = 100;
        total += 50;   // total = total + 50 = 150
        System.out.println("After += 50: " + total);

        total -= 30;   // total = total - 30 = 120
        System.out.println("After -= 30: " + total);

        total *= 2;    // total = total * 2 = 240
        System.out.println("After *= 2: " + total);

        total /= 4;    // total = total / 4 = 60
        System.out.println("After /= 4: " + total);

        total %= 7;    // total = total % 7 = 4
        System.out.println("After %= 7: " + total);
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Operators are embedded in every line of backend logic. Here are three realistic scenarios from production systems.

### Scenario 1: Pagination calculation in a REST API

Every backend API that returns lists of data (users, products, orders) needs pagination. Pagination math relies entirely on arithmetic operators.

```java
package com.company.orderservice.service;

import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

public class PaginationService {

    private static final int MAX_PAGE_SIZE = 100;
    private static final int DEFAULT_PAGE_SIZE = 20;

    public Pageable createPageable(int page, int size) {
        // Validate page number. Pages are zero-indexed in Spring Data.
        // If the client sends a negative page, default to 0.
        int safePage = (page < 0) ? 0 : page;

        // Clamp the page size between 1 and MAX_PAGE_SIZE.
        // This prevents a malicious client from requesting size=999999
        // and crashing your database.
        int safeSize = (size <= 0) ? DEFAULT_PAGE_SIZE : Math.min(size, MAX_PAGE_SIZE);

        return PageRequest.of(safePage, safeSize);
    }

    public int calculateTotalPages(long totalRecords, int pageSize) {
        // Ceiling division: how many pages do we need?
        // For 105 records with page size 20, we need 6 pages (not 5).
        // Integer division truncates, so we add (pageSize - 1) before dividing.
        return (int) ((totalRecords + pageSize - 1) / pageSize);
    }

    public long calculateOffset(int page, int size) {
        // The offset tells the database how many rows to skip.
        // Page 0, size 20 -> offset 0 (rows 1-20)
        // Page 1, size 20 -> offset 20 (rows 21-40)
        // Page 2, size 20 -> offset 40 (rows 41-60)
        return (long) page * size;
    }
}
```

**What to notice:**

- The ternary operator `(page < 0) ? 0 : page` is used for input validation. This pattern appears constantly in backend code to sanitize client input.
- The ceiling division formula `(totalRecords + pageSize - 1) / pageSize` is a classic arithmetic trick. It avoids floating-point math entirely, which is faster and more reliable.
- The cast `(long) page * size` prevents integer overflow. If `page` is 200000 and `size` is 100, the product is 20 million, which fits in `int`. But if `page` is 30 million and `size` is 100, the product overflows `int`. Casting to `long` before multiplication prevents this.

### Scenario 2: Input validation with logical operators

Backend services must validate every piece of data that comes from the client. Logical operators combine multiple validation rules.

```java
package com.company.orderservice.service;

import java.math.BigDecimal;

public class OrderValidationService {

    public void validateOrder(int itemCount, BigDecimal totalAmount, String customerEmail, boolean isVerified) {
        // Rule 1: Order must have at least 1 item and no more than 500
        if (itemCount <= 0 || itemCount > 500) {
            throw new IllegalArgumentException(
                "Item count must be between 1 and 500. Received: " + itemCount
            );
        }

        // Rule 2: Total amount must be positive and not exceed the maximum order limit
        BigDecimal maxOrderAmount = new BigDecimal("500000.00");
        if (totalAmount.compareTo(BigDecimal.ZERO) <= 0 || totalAmount.compareTo(maxOrderAmount) > 0) {
            throw new IllegalArgumentException(
                "Order amount must be between 0.01 and 500,000.00 BDT."
            );
        }

        // Rule 3: Email must not be null, not empty, and must contain '@'
        if (customerEmail == null || customerEmail.isEmpty() || !customerEmail.contains("@")) {
            throw new IllegalArgumentException("A valid email address is required.");
        }

        // Rule 4: Unverified users can only place orders below 5000 BDT
        BigDecimal unverifiedLimit = new BigDecimal("5000.00");
        if (!isVerified && totalAmount.compareTo(unverifiedLimit) > 0) {
            throw new IllegalStateException(
                "Unverified users cannot place orders above 5,000 BDT. Please verify your account."
            );
        }

        // Rule 5: Short-circuit evaluation prevents NullPointerException
        // If customerEmail is null, the .length() call would crash.
        // But || short-circuits: if the left side is true, the right side is never evaluated.
        if (customerEmail == null || customerEmail.length() > 255) {
            throw new IllegalArgumentException("Email must not exceed 255 characters.");
        }
    }
}
```

**What to notice:**

- The `||` operator chains multiple rejection conditions. If any single condition is true, the entire expression is true and the exception is thrown.
- The `&&` operator chains multiple acceptance conditions. All conditions must be true for the code to proceed.
- Short-circuit evaluation in Rule 5 is a safety mechanism. Because `customerEmail == null` is checked first with `||`, the `customerEmail.length()` call on the right side is never executed if the email is null. If you used `|` (bitwise OR, which does not short-circuit) instead of `||`, this code would throw a `NullPointerException`.
- `BigDecimal.compareTo()` returns an `int`, not a `boolean`. It returns -1, 0, or 1. You must use relational operators (`<= 0`, `> 0`) to interpret the result. You cannot use `==` to compare BigDecimal values because `==` checks reference equality.

### Scenario 3: Role-based access check

```java
package com.company.orderservice.security;

public class AuthorizationService {

    public boolean canDeleteOrder(String userRole, boolean isOrderOwner, boolean isOrderShipped) {
        // Admins can delete any order that has not been shipped
        if ("ADMIN".equals(userRole) && !isOrderShipped) {
            return true;
        }

        // Order owners can delete their own order only if it has not been shipped
        if (isOrderOwner && !isOrderShipped) {
            return true;
        }

        // Everyone else is denied
        return false;
    }

    // The above can be simplified to a single return statement:
    public boolean canDeleteOrderConcise(String userRole, boolean isOrderOwner, boolean isOrderShipped) {
        boolean isAdmin = "ADMIN".equals(userRole);
        boolean hasPermission = isAdmin || isOrderOwner;
        return hasPermission && !isOrderShipped;
    }
}
```

**What to notice:**

- `"ADMIN".equals(userRole)` instead of `userRole.equals("ADMIN")`. This is a defensive pattern. If `userRole` is null, `userRole.equals("ADMIN")` throws a `NullPointerException`. But `"ADMIN".equals(null)` safely returns `false`. This pattern is called a "Yoda condition" and is standard practice in backend Java.
- The concise version breaks the logic into named boolean variables. This makes the code self-documenting. In a code review, `hasPermission && !isOrderShipped` is immediately understandable.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using `==` to compare Strings

**Wrong:**
```java
String input = scanner.nextLine();  // User types "admin"
if (input == "admin") {
    System.out.println("Welcome, admin!");  // This will NEVER print
}
```

**Right:**
```java
String input = scanner.nextLine();
if ("admin".equals(input)) {
    System.out.println("Welcome, admin!");  // This works correctly
}
```

**Why it is wrong:** `==` compares memory addresses for reference types. Two String objects with the same text can live at different memory locations. `.equals()` compares the actual character content. This mistake is so common that it causes real production bugs in authentication systems where login checks silently fail.

### Mistake 2: Integer division when you need decimals

**Wrong:**
```java
int totalScore = 85;
int maxScore = 100;
double percentage = totalScore / maxScore;
System.out.println("Percentage: " + percentage);  // 0.0 (not 0.85!)
// 85 / 100 in integer math is 0. The .0 is added after the division is already done.
```

**Right:**
```java
int totalScore = 85;
int maxScore = 100;
double percentage = (double) totalScore / maxScore;
System.out.println("Percentage: " + percentage);  // 0.85
```

**Why it is wrong:** When both operands of `/` are integers, Java performs integer division and discards the remainder. The result is `0`, and then `0` is implicitly cast to `0.0` when assigned to the `double` variable. The cast must happen before the division.

### Mistake 3: Confusing `=` (assignment) with `==` (comparison)

**Wrong:**
```java
int x = 5;
if (x = 10) {  // COMPILATION ERROR in Java (unlike C/C++)
    System.out.println("This won't compile");
}
```

**Right:**
```java
int x = 5;
if (x == 10) {  // Comparison. Evaluates to false.
    System.out.println("x is 10");
}
```

**Why it is wrong:** `=` assigns a value. `==` compares values. Java's compiler is stricter than C/C++ and will reject `if (x = 10)` because the result of an assignment is an `int`, not a `boolean`. However, this mistake IS possible with boolean variables: `if (isActive = true)` compiles but assigns `true` to `isActive` instead of checking it. Always use `if (isActive)` instead of `if (isActive == true)`.

### Mistake 4: Ignoring short-circuit evaluation

**Wrong:**
```java
String name = null;
// Using & (bitwise AND) instead of && (logical AND)
// Bitwise AND does NOT short-circuit, so both sides are always evaluated.
if (name != null & name.length() > 3) {  // NullPointerException!
    System.out.println("Valid name");
}
```

**Right:**
```java
String name = null;
if (name != null && name.length() > 3) {  // Safe. Short-circuits after first false.
    System.out.println("Valid name");
}
```

**Why it is wrong:** `&` and `|` are bitwise operators that evaluate both sides unconditionally. `&&` and `||` are logical operators that short-circuit. In conditional statements, always use `&&` and `||`. The only time you need `&` or `|` in a boolean context is when you specifically want both sides evaluated (which is rare).

---

## Key Takeaways

> [!tip] Remember these points
> 1. Arithmetic operators (`+`, `-`, `*`, `/`, `%`) perform math. Integer division truncates decimals. Use casting to `double` when you need fractional results.
> 2. Relational operators (`==`, `!=`, `>`, `<`, `>=`, `<=`) return `boolean`. For primitive types, `==` compares values. For reference types like `String`, `==` compares memory addresses. Always use `.equals()` for object comparison.
> 3. Logical operators (`&&`, `||`, `!`) combine boolean expressions. `&&` and `||` short-circuit, which prevents null pointer exceptions and unnecessary computation.
> 4. The ternary operator `condition ? a : b` is a compact if-else for simple cases. Do not nest ternaries deeply.
> 5. Use parentheses to make operator precedence explicit. Code is read far more often than it is written, so clarity is more important than brevity.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Basic Calculator (Easy)
Write a program that takes two numbers and an operator (`+`, `-`, `*`, `/`, `%`) as input from the user using `Scanner`. Perform the operation and print the result. Handle division by zero by printing an error message instead of crashing.

**Hint:** Use `scanner.nextDouble()` for the numbers and `scanner.next()` for the operator. Use an `if-else` chain to check which operator was entered.

### Exercise 2: Grade Classifier (Medium)
Write a program that takes a student's marks (0-100) as input and prints the grade using the following scale:
- 80-100: A+
- 75-79: A
- 70-74: A-
- 65-69: B+
- 60-64: B
- 55-59: B-
- 50-54: C
- 45-49: D
- 0-44: F

Use logical operators (`&&`) to check ranges. Also validate that the input is between 0 and 100, and print an error if it is not.

**Hint:** `if (marks >= 80 && marks <= 100)` checks the A+ range. Chain these conditions with `else if`.

### Exercise 3: Leap Year Checker (Medium)
Write a program that takes a year as input and determines whether it is a leap year. A year is a leap year if:
- It is divisible by 4 AND not divisible by 100, OR
- It is divisible by 400.

Use modulus (`%`) and logical operators (`&&`, `||`) to implement this in a single boolean expression.

**Hint:** The entire leap year check can be written as one line: `boolean isLeap = (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);`

### Exercise 4: Debug This Code (Hard, Optional)
The following code has four operator-related bugs. Find and fix all four. Explain each fix in a comment.

```java
public class BuggyOperators {
    public static void main(String[] args) {
        int totalMarks = 85;
        int maxMarks = 100;
        double percentage = totalMarks / maxMarks * 100;
        System.out.println("Percentage: " + percentage);

        String grade = "A";
        if (grade == "A") {
            System.out.println("Excellent!");
        }

        int age = 20;
        boolean isAdult = (age = 18);
        System.out.println("Is adult: " + isAdult);

        String username = null;
        if (username != null & username.length() > 0) {
            System.out.println("Hello, " + username);
        }
    }
}
```

**Hint:** One bug involves integer division, one involves String comparison, one involves assignment vs comparison, and one involves short-circuit evaluation.

### Solution
For Exercise 1:
```java
import java.util.Scanner;

public class Calculator {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter first number: ");
        double num1 = scanner.nextDouble();

        System.out.print("Enter operator (+, -, *, /, %): ");
        String operator = scanner.next();

        System.out.print("Enter second number: ");
        double num2 = scanner.nextDouble();

        double result;

        if (operator.equals("+")) {
            result = num1 + num2;
        } else if (operator.equals("-")) {
            result = num1 - num2;
        } else if (operator.equals("*")) {
            result = num1 * num2;
        } else if (operator.equals("/")) {
            if (num2 == 0) {
                System.out.println("Error: Division by zero is not allowed.");
                return;
            }
            result = num1 / num2;
        } else if (operator.equals("%")) {
            if (num2 == 0) {
                System.out.println("Error: Modulus by zero is not allowed.");
                return;
            }
            result = num1 % num2;
        } else {
            System.out.println("Error: Invalid operator.");
            return;
        }

        System.out.println("Result: " + num1 + " " + operator + " " + num2 + " = " + result);
        scanner.close();
    }
}
```


For Exercise 3:
```java
import java.util.Scanner;

public class LeapYear {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter a year: ");
        int year = scanner.nextInt();

        boolean isLeap = (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);

        if (isLeap) {
            System.out.println(year + " is a leap year.");
        } else {
            System.out.println(year + " is not a leap year.");
        }
        scanner.close();
    }
}
```


For Exercise 4:
```java
public class BuggyOperators {
    public static void main(String[] args) {
        int totalMarks = 85;
        int maxMarks = 100;

        // Bug 1: Integer division. 85/100 = 0 in integer math.
        // Fix: Cast to double before division.
        double percentage = (double) totalMarks / maxMarks * 100;
        System.out.println("Percentage: " + percentage);  // Now prints 85.0

        String grade = "A";

        // Bug 2: Using == for String comparison.
        // Fix: Use .equals() method.
        if ("A".equals(grade)) {
            System.out.println("Excellent!");
        }

        int age = 20;

        // Bug 3: Using = (assignment) instead of >= (comparison).
        // Fix: Use >= to compare. Also, boolean cannot be assigned an int.
        boolean isAdult = (age >= 18);
        System.out.println("Is adult: " + isAdult);

        String username = null;

        // Bug 4: Using & (bitwise AND) instead of && (logical AND).
        // & does not short-circuit, so username.length() is called even when username is null.
        // Fix: Use && so the second condition is skipped when the first is false.
        if (username != null && username.length() > 0) {
            System.out.println("Hello, " + username);
        }
    }
}
```

---

## Related Notes

- [[Java - Variables and Data Types]]
- [[Java - Control Flow - If Else Switch]]

---

## Resources

- [Oracle Java Tutorials: Operators](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html) - Official documentation covering all Java operators with examples.
- [Oracle Java Tutorials: Operator Precedence](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html) - The precedence table at the bottom of the page.
- [Baeldung: Java == vs equals()](https://www.baeldung.com/java-compare-objects) - Detailed explanation of why == and .equals() behave differently for objects.
