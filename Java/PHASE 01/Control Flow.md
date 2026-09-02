## Overview

Control flow determines the order in which statements execute. Without it, your program would run from top to bottom and stop. With it, your program can make decisions, repeat operations, and handle different scenarios. Every piece of business logic you will write — validating a transaction, routing a payment, calculating tiered interest rates, retrying a failed API call — relies on control flow constructs. This section covers every branching and looping mechanism in Java, from the basic `if` statement to Java 21's pattern-matching switch expressions.

---

## Core Concepts

### if / else if / else

The most fundamental branching construct. Evaluates a boolean condition and executes the corresponding block.

```java
if (condition) {
    // Executes when condition is true
} else if (anotherCondition) {
    // Executes when first condition is false AND this condition is true
} else {
    // Executes when all conditions above are false
}
```

Rules:
- The condition must evaluate to a `boolean`. Java does not accept integers, strings, or null as conditions (unlike C or Python).
- Curly braces are technically optional for single-statement blocks, but always use them. Omitting braces is a leading cause of bugs in production code.
- Conditions are evaluated top to bottom. The first matching branch executes. All subsequent branches are skipped.
- The `else` block is optional. The `else if` blocks are optional and can be repeated any number of times.

**Common mistake — assignment instead of comparison:**

```java
boolean isActive = false;

// BUG: This assigns true to isActive, then evaluates the result (true)
if (isActive = true) {
    System.out.println("This always prints!");
}

// CORRECT: Comparison
if (isActive == true) {
    System.out.println("This does not print");
}

// BETTER: Direct boolean evaluation
if (isActive) {
    System.out.println("This does not print");
}
```

### Ternary Operator

A compact alternative to `if/else` for simple value assignments.

```java
// Syntax: condition ? valueIfTrue : valueIfFalse

int age = 20;
String status = (age >= 18) ? "adult" : "minor";

// Equivalent to:
// String status;
// if (age >= 18) {
//     status = "adult";
// } else {
//     status = "minor";
// }
```

Rules:
- Both branches must return compatible types.
- Use only for simple, readable expressions. If the ternary spans multiple lines or contains complex logic, use `if/else` instead.
- Ternaries can be nested, but nested ternaries are almost always unreadable. Avoid them.

```java
// Acceptable
double fee = (amount > 10_000) ? 0.0 : 2.50;

// Unreadable — do not do this
String tier = (balance > 100_000) ? "platinum" :
              (balance > 50_000) ? "gold" :
              (balance > 10_000) ? "silver" : "bronze";
// Use if/else if/else instead for multi-branch logic.
```

### Traditional switch Statement

The `switch` statement branches based on the value of an expression. It is cleaner than long `if/else if` chains when comparing a single variable against multiple constant values.

```java
String currency = "USD";
String symbol;

switch (currency) {
    case "USD":
        symbol = "$";
        break;
    case "EUR":
        symbol = "€";
        break;
    case "GBP":
        symbol = "£";
        break;
    case "JPY":
        symbol = "¥";
        break;
    default:
        symbol = "?";
        break;
}
```

Rules:
- The switch expression must be of type `byte`, `short`, `char`, `int`, `String`, an enum, or a wrapper type (`Integer`, `Character`, etc.).
- Each `case` label must be a compile-time constant.
- The `break` statement exits the switch. Without it, execution **falls through** to the next case.
- The `default` case executes when no other case matches. It is optional but strongly recommended.
- `default` can appear anywhere in the switch, but convention places it last.

**Fall-through behavior (intentional and unintentional):**

```java
int statusCode = 201;
String category;

switch (statusCode) {
    case 200:
    case 201:
    case 204:
        category = "success";
        break;
    case 400:
    case 401:
    case 403:
    case 404:
    case 422:
        category = "client_error";
        break;
    case 500:
    case 502:
    case 503:
        category = "server_error";
        break;
    default:
        category = "unknown";
        break;
}
```

Here, fall-through is intentional. Multiple cases share the same logic. This is the one scenario where fall-through is acceptable. Always add a comment when fall-through is intentional so reviewers know it is not a bug.

**Unintentional fall-through (a classic bug):**

```java
String action = "WITHDRAW";

switch (action) {
    case "DEPOSIT":
        System.out.println("Processing deposit");
        // BUG: missing break — falls through to WITHDRAW
    case "WITHDRAW":
        System.out.println("Processing withdrawal");
        break;
    case "TRANSFER":
        System.out.println("Processing transfer");
        break;
}
// Output:
// Processing deposit    ← This should NOT have printed
// Processing withdrawal
```

### Switch Expressions (Java 14+)

Switch expressions are a modern replacement for the traditional switch statement. They return a value, do not fall through by default, and use arrow syntax.

```java
String currency = "EUR";

// Arrow syntax — no break needed, no fall-through
String symbol = switch (currency) {
    case "USD" -> "$";
    case "EUR" -> "€";
    case "GBP" -> "£";
    case "JPY" -> "¥";
    default -> "?";
};

System.out.println(symbol);  // €
```

**Multiple labels per case:**

```java
int statusCode = 404;

String category = switch (statusCode) {
    case 200, 201, 204 -> "success";
    case 400, 401, 403, 404, 422 -> "client_error";
    case 500, 502, 503 -> "server_error";
    default -> "unknown";
};
```

**Multi-line case blocks with `yield`:**

When a case requires multiple statements, use a block with `yield` to return the value.

```java
String tier = switch (accountBalance) {
    case BigDecimal b when b.compareTo(new BigDecimal("100000")) >= 0 -> {
        log.info("Platinum tier customer");
        yield "platinum";
    }
    case BigDecimal b when b.compareTo(new BigDecimal("50000")) >= 0 -> {
        log.info("Gold tier customer");
        yield "gold";
    }
    case BigDecimal b when b.compareTo(new BigDecimal("10000")) >= 0 -> {
        yield "silver";
    }
    default -> {
        yield "bronze";
    }
};
```

**Key differences from traditional switch:**

| Feature | Traditional switch | Switch expression |
|---------|-------------------|-------------------|
| Returns a value | No | Yes |
| Fall-through | Yes (requires `break`) | No (arrow syntax prevents it) |
| Exhaustiveness | Not checked | Checked by compiler for enums and sealed types |
| Syntax | `case X:` with `break` | `case X ->` or `case X:` with `yield` |
| Multiple labels | Separate `case` lines | Comma-separated: `case A, B, C ->` |

**Exhaustiveness checking:**

When switching over an enum, the compiler verifies that all values are covered. If you add a new enum constant later and forget to update the switch, the compiler will flag it.

```java
public enum TransactionStatus {
    PENDING, COMPLETED, FAILED, CANCELLED
}

TransactionStatus status = TransactionStatus.COMPLETED;

// Compiler error if any enum value is missing (when no default is present)
String message = switch (status) {
    case PENDING -> "Transaction is pending";
    case COMPLETED -> "Transaction completed successfully";
    case FAILED -> "Transaction failed";
    case CANCELLED -> "Transaction was cancelled";
    // No default needed — all enum values are covered
};
```

### Pattern Matching in switch (Java 21)

Java 21 extends switch to support type patterns and guarded patterns. This is one of the most powerful features in modern Java and eliminates long chains of `instanceof` checks.

```java
// Type patterns
Object payment = getPaymentMethod();

String description = switch (payment) {
    case CreditCard cc -> "Credit card ending in " + cc.getLastFourDigits();
    case BankTransfer bt -> "Bank transfer from " + bt.getBankName();
    case CryptoWallet cw -> "Crypto wallet " + cw.getAddress();
    case null -> "No payment method";
    default -> "Unknown payment type";
};
```

**Guarded patterns (when clause):**

```java
Object transaction = getTransaction();

String riskLevel = switch (transaction) {
    case Transaction t when t.getAmount().compareTo(new BigDecimal("10000")) > 0
        -> "HIGH";
    case Transaction t when t.getAmount().compareTo(new BigDecimal("1000")) > 0
        -> "MEDIUM";
    case Transaction t -> "LOW";
    case Refund r -> "REVIEW";
    case null -> "INVALID";
};
```

**Null handling:**

Traditional switch throws a `NullPointerException` if the switch expression is null. Switch expressions and pattern-matching switch allow explicit null handling.

```java
String input = null;

// Traditional switch — throws NullPointerException
// switch (input) { case "hello": ... }

// Pattern-matching switch — handles null explicitly
String result = switch (input) {
    case null -> "Input was null";
    case "" -> "Input was empty";
    case String s when s.length() > 100 -> "Input too long";
    case String s -> "Valid input: " + s;
};
```

### while Loop

Executes a block repeatedly as long as a condition is true. The condition is checked **before** each iteration.

```java
int attempts = 0;
int maxAttempts = 3;
boolean success = false;

while (attempts < maxAttempts && !success) {
    attempts++;
    System.out.println("Attempt " + attempts);
    success = processPayment();  // Returns true on success
}

if (!success) {
    System.out.println("Payment failed after " + maxAttempts + " attempts");
}
```

Rules:
- If the condition is false initially, the loop body never executes.
- The condition must be a `boolean` expression.
- Ensure the loop will eventually terminate. An infinite `while(true)` loop without a `break` will hang your application.

### do-while Loop

Executes a block **at least once**, then checks the condition. The condition is checked **after** each iteration.

```java
String input;
do {
    System.out.print("Enter a valid transaction amount (> 0): ");
    input = scanner.nextLine();
} while (input.isEmpty() || new BigDecimal(input).compareTo(BigDecimal.ZERO) <= 0);
```

Rules:
- The loop body always executes at least once, regardless of the condition.
- Use when you need to perform an action before checking whether to repeat.
- Less common than `while` in production backend code.

### Traditional for Loop

The C-style for loop gives you full control over initialization, condition, and iteration.

```java
// Syntax: for (initialization; condition; update) { body }

for (int i = 0; i < 10; i++) {
    System.out.println("Iteration " + i);
}

// Multiple variables
for (int i = 0, j = 10; i < j; i++, j--) {
    System.out.println(i + " " + j);
}

// Counting down
for (int i = 100; i >= 0; i -= 10) {
    System.out.println(i + "%");
}
```

Rules:
- The initialization runs once before the loop starts.
- The condition is checked before each iteration.
- The update runs after each iteration.
- Any of the three parts can be omitted (but the semicolons are required).
- Variables declared in the initialization are scoped to the loop.

### Enhanced for Loop (for-each)

Iterates over elements of an array or any `Iterable` collection. Cleaner and less error-prone than the traditional for loop.

```java
// Arrays
int[] amounts = {100, 250, 500, 1000};
for (int amount : amounts) {
    System.out.println("Processing: $" + amount);
}

// Collections
List<String> currencies = List.of("USD", "EUR", "GBP", "JPY");
for (String currency : currencies) {
    System.out.println("Loading exchange rate for " + currency);
}

// Maps (iterate over entries)
Map<String, BigDecimal> balances = Map.of(
    "Alice", new BigDecimal("1500.00"),
    "Bob", new BigDecimal("3200.50")
);
for (Map.Entry<String, BigDecimal> entry : balances.entrySet()) {
    System.out.println(entry.getKey() + ": $" + entry.getValue());
}
```

Rules:
- You cannot modify the collection structure (add/remove elements) during iteration. Doing so throws `ConcurrentModificationException`.
- You do not have access to the index. If you need the index, use the traditional for loop or a stream with `IntStream.range()`.
- The loop variable is a copy of the reference (for objects) or a copy of the value (for primitives). Reassigning the loop variable does not modify the original collection.

```java
List<String> names = new ArrayList<>(List.of("alice", "bob"));
for (String name : names) {
    name = name.toUpperCase();  // Does NOT modify the list
}
System.out.println(names);  // [alice, bob] — unchanged
```

### break and continue

**`break`** exits the innermost loop or switch immediately.

```java
List<Transaction> transactions = getTransactions();
Transaction target = null;

for (Transaction t : transactions) {
    if (t.getId().equals(targetId)) {
        target = t;
        break;  // Stop searching — we found it
    }
}
```

**`continue`** skips the rest of the current iteration and moves to the next.

```java
List<Transaction> transactions = getTransactions();
BigDecimal total = BigDecimal.ZERO;

for (Transaction t : transactions) {
    if (t.getStatus() == TransactionStatus.CANCELLED) {
        continue;  // Skip cancelled transactions
    }
    if (t.getStatus() == TransactionStatus.FAILED) {
        continue;  // Skip failed transactions
    }
    total = total.add(t.getAmount());
}
```

### Labeled break and continue

Java allows you to label loops and break or continue to a specific outer loop. This is useful in nested loops but should be used sparingly.

```java
// Labeled break: exit the outer loop from inside the inner loop
outer:
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if (i * j > 50) {
            break outer;  // Exits BOTH loops
        }
        System.out.println(i + " * " + j + " = " + (i * j));
    }
}

// Labeled continue: skip to the next iteration of the outer loop
outer:
for (int row = 0; row < 5; row++) {
    for (int col = 0; col < 5; col++) {
        if (col == row) {
            continue outer;  // Skips to next row
        }
        System.out.print("(" + row + "," + col + ") ");
    }
    System.out.println();
}
```

In practice, labeled breaks and continues are rare in production code. If you find yourself needing them, consider extracting the nested loop into a separate method and using `return` instead.

### Nested Loops

Loops inside loops. The inner loop completes all its iterations for each iteration of the outer loop.

```java
// Multiplication table
for (int i = 1; i <= 5; i++) {
    for (int j = 1; j <= 5; j++) {
        System.out.printf("%4d", i * j);
    }
    System.out.println();
}
// Output:
//    1   2   3   4   5
//    2   4   6   8  10
//    3   6   9  12  15
//    4   8  12  16  20
//    5  10  15  20  25
```

**Performance warning:** Nested loops multiply their iteration counts. Two nested loops each running 10,000 times execute 100,000,000 iterations. Three nested loops of 1,000 each execute 1,000,000,000 iterations. In backend systems processing large datasets, nested loops are often the cause of performance bottlenecks. Consider using a `HashMap` lookup (O(1)) instead of an inner loop search (O(n)) when possible.

---

## Code Examples

**Tiered fee calculation using if/else if/else:**

```java
public BigDecimal calculateTransferFee(BigDecimal amount) {
    if (amount.compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("Amount must be positive");
    } else if (amount.compareTo(new BigDecimal("1000")) <= 0) {
        return new BigDecimal("1.00");           // Flat $1 for transfers up to $1,000
    } else if (amount.compareTo(new BigDecimal("10000")) <= 0) {
        return amount.multiply(new BigDecimal("0.005"));  // 0.5% for $1,001-$10,000
    } else if (amount.compareTo(new BigDecimal("100000")) <= 0) {
        return amount.multiply(new BigDecimal("0.0025")); // 0.25% for $10,001-$100,000
    } else {
        return amount.multiply(new BigDecimal("0.001"));  // 0.1% for $100,001+
    }
}
```

**Transaction routing using switch expression (Java 14+):**

```java
public enum PaymentMethod {
    CREDIT_CARD, DEBIT_CARD, BANK_TRANSFER, ACH, WIRE, CRYPTO
}

public PaymentProcessor getProcessor(PaymentMethod method) {
    return switch (method) {
        case CREDIT_CARD, DEBIT_CARD -> new CardProcessor();
        case BANK_TRANSFER, ACH -> new AchProcessor();
        case WIRE -> new WireProcessor();
        case CRYPTO -> new CryptoProcessor();
    };
    // No default needed — all enum values covered.
    // If a new PaymentMethod is added to the enum, the compiler
    // will flag this switch as non-exhaustive.
}
```

**Retry logic using while loop:**

```java
public PaymentResponse processWithRetry(PaymentRequest request, int maxRetries) {
    int attempt = 0;
    PaymentResponse response = null;
    Exception lastException = null;

    while (attempt < maxRetries) {
        attempt++;
        try {
            response = paymentGateway.process(request);
            if (response.isSuccess()) {
                return response;
            }
            if (!response.isRetryable()) {
                return response;  // Do not retry permanent failures
            }
        } catch (TimeoutException e) {
            lastException = e;
            log.warn("Timeout on attempt {}/{}", attempt, maxRetries);
        } catch (ConnectionException e) {
            lastException = e;
            log.warn("Connection error on attempt {}/{}", attempt, maxRetries);
        }

        if (attempt < maxRetries) {
            long backoffMs = (long) Math.pow(2, attempt) * 1000;
            Thread.sleep(backoffMs);  // Exponential backoff
        }
    }

    throw new PaymentProcessingException(
        "Failed after " + maxRetries + " attempts", lastException
    );
}
```

**Processing a batch of transactions using for-each with continue and break:**

```java
public BatchResult processBatch(List<Transaction> transactions) {
    int processed = 0;
    int skipped = 0;
    BigDecimal totalAmount = BigDecimal.ZERO;

    for (Transaction tx : transactions) {
        if (tx.getStatus() == TransactionStatus.CANCELLED) {
            skipped++;
            continue;
        }

        if (tx.getAmount().compareTo(new BigDecimal("1000000")) > 0) {
            log.error("Transaction {} exceeds single-transaction limit", tx.getId());
            break;  // Halt the entire batch — manual review required
        }

        try {
            processTransaction(tx);
            totalAmount = totalAmount.add(tx.getAmount());
            processed++;
        } catch (ProcessingException e) {
            log.error("Failed to process transaction {}", tx.getId(), e);
            skipped++;
        }
    }

    return new BatchResult(processed, skipped, totalAmount);
}
```

**Pattern-matching switch for event handling (Java 21):**

```java
public void handleEvent(Object event) {
    switch (event) {
        case PaymentCompleted pc -> {
            updateLedger(pc.getTransactionId(), pc.getAmount());
            sendReceipt(pc.getCustomerId());
            notifyFraudSystem(pc);
        }
        case PaymentFailed pf when pf.getRetryCount() < 3 -> {
            scheduleRetry(pf.getTransactionId(), pf.getRetryCount() + 1);
        }
        case PaymentFailed pf -> {
            markAsPermanentlyFailed(pf.getTransactionId());
            notifyCustomer(pf.getCustomerId(), "Payment failed permanently");
        }
        case RefundRequested rr -> {
            initiateRefund(rr.getTransactionId(), rr.getAmount());
        }
        case AccountSuspended as -> {
            freezeAccount(as.getAccountId());
            cancelPendingTransactions(as.getAccountId());
        }
        case null -> log.warn("Received null event");
        default -> log.warn("Unknown event type: {}", event.getClass().getName());
    }
}
```

**FizzBuzz using traditional for loop (a common interview problem):**

```java
public void fizzBuzz(int n) {
    for (int i = 1; i <= n; i++) {
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
```

---

## Important Notes

- Java requires boolean conditions in all control flow constructs. You cannot write `if (1)` or `if (value)` as you can in C, Python, or JavaScript. The condition must explicitly evaluate to `true` or `false`. This eliminates an entire class of bugs where assignment (`=`) is accidentally used instead of comparison (`==`).
- Always use curly braces for `if`, `else`, `for`, and `while` blocks, even when the body is a single statement. The Apple "goto fail" SSL bug (2014) was caused by a missing brace in an `if` statement. This is enforced by checkstyle in professional codebases.
- Switch expressions (Java 14+) are preferred over traditional switch statements in new code. They are safer (no fall-through), more concise, and return values directly. Use traditional switch only when maintaining legacy code.
- Pattern-matching switch (Java 21) is a significant language evolution. It replaces chains of `if (x instanceof Type t)` checks with a single, readable switch expression. Use it whenever you are dispatching on type.
- The enhanced for loop (for-each) should be your default choice for iterating over collections. It eliminates off-by-one errors and index-out-of-bounds exceptions. Use the traditional for loop only when you need the index or need to iterate in a non-sequential order.
- `break` and `continue` are acceptable in moderation. If a method has more than two or three of them, the logic is likely too complex and should be refactored into smaller methods.
- Labeled breaks and continues are almost never necessary in production code. If you need them, extract the nested loop into a method and use `return` instead. Labeled jumps make code harder to read and reason about.
- Nested loops have multiplicative time complexity. A loop of size n inside a loop of size m is O(n * m). When processing large datasets, replace inner loop lookups with `HashMap` or `HashSet` lookups to reduce O(n * m) to O(n + m). This is a common optimization in backend systems.
- The `while(true)` pattern is acceptable for server loops (event loops, message consumers) as long as there is a clear exit condition (shutdown signal, interrupt). In application logic, prefer bounded loops with explicit termination conditions.
- The `do-while` loop is the least commonly used loop in backend Java. Its primary use case is input validation where you need to prompt at least once. If you find yourself using `do-while` frequently, consider whether a `while` loop with initialization would be clearer.

---

## Practice

1. Write a method `String classifyTransaction(BigDecimal amount)` that returns `"micro"` for amounts under $1, `"small"` for $1-$99.99, `"medium"` for $100-$9,999.99, `"large"` for $10,000-$99,999.99, and `"wholesale"` for $100,000 and above. Implement it first with `if/else if/else`, then rewrite it with a switch expression and guarded patterns.

2. Write a method that simulates processing a list of transactions. Use a for-each loop. Skip transactions with a `CANCELLED` status using `continue`. Stop processing entirely if a transaction amount exceeds $1,000,000 using `break`. Count and return the number of processed and skipped transactions.

3. Write a retry mechanism using a `while` loop. The method should attempt to call a mock `processPayment()` method up to 5 times. On each failure, wait with exponential backoff (1s, 2s, 4s, 8s). If all retries fail, throw a custom exception.

4. Write a program that uses a switch expression with pattern matching (Java 21) to handle different shapes. Create a sealed interface `Shape` with records `Circle(double radius)`, `Rectangle(double width, double height)`, and `Triangle(double base, double height)`. Use a switch expression to calculate the area of each shape.

5. Write a nested loop that finds all pairs of transactions in a list that sum to a target amount. Then rewrite it using a `HashMap` for O(n) performance. Benchmark both approaches with a list of 10,000 transactions.

6. In your Obsidian vault, create a decision table comparing `if/else`, `switch statement`, `switch expression`, and `pattern-matching switch`. For each, note when to use it, when to avoid it, and the Java version required.

---

## References

- Java Language Specification — Chapter 14 (Blocks and Statements): https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html
- JEP 361 (Switch Expressions, Java 14): https://openjdk.org/jeps/361
- JEP 441 (Pattern Matching for switch, Java 21): https://openjdk.org/jeps/441
- JEP 427 (Pattern Matching for switch, Third Preview, Java 19): https://openjdk.org/jeps/427
- "Effective Java" by Joshua Bloch — Item 104 (Prefer switch expressions to switch statements)
