## Overview

Exceptions are Java's mechanism for signaling that something has gone wrong during program execution. Unlike return codes or null checks, exceptions cannot be silently ignored. They force the caller to acknowledge the failure, either by handling it or by declaring that it propagates upward. In a fintech backend, a failed database connection, an invalid transaction amount, or a timeout from a payment gateway must be handled explicitly and correctly. Mishandled exceptions lead to lost payments, corrupted data, and production incidents. This section covers the complete exception hierarchy, the mechanics of try-catch-finally, checked versus unchecked exceptions, custom exception design, try-with-resources, and the best practices that separate robust production code from fragile prototypes.

---

## Core Concepts

### The Exception Hierarchy

All exceptions and errors in Java inherit from `java.lang.Throwable`. The hierarchy has two main branches: `Error` and `Exception`.

```
Throwable
├── Error (do NOT catch)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   ├── NoClassDefFoundError
│   ├── LinkageError
│   └── VirtualMachineError
│
└── Exception
    ├── Checked Exceptions (compiler forces you to handle)
    │   ├── IOException
    │   │   ├── FileNotFoundException
    │   │   ├── EOFException
    │   │   └── SocketException
    │   ├── SQLException
    │   ├── ClassNotFoundException
    │   ├── InterruptedException
    │   ├── ParseException
    │   └── TimeoutException
    │
    └── RuntimeException (unchecked — compiler does NOT force handling)
        ├── NullPointerException
        ├── IllegalArgumentException
        ├── IllegalStateException
        ├── IndexOutOfBoundsException
        │   ├── ArrayIndexOutOfBoundsException
        │   └── StringIndexOutOfBoundsException
        ├── ClassCastException
        ├── ArithmeticException
        ├── NumberFormatException
        ├── UnsupportedOperationException
        ├── ConcurrentModificationException
        ├── NoSuchElementException
        └── SecurityException
```

**Error vs Exception:**

- **Error** represents serious problems that a reasonable application should not try to catch. `OutOfMemoryError`, `StackOverflowError`, and `NoClassDefFoundError` indicate that the JVM itself is in a compromised state. Catching an `Error` is almost always wrong because the JVM may not be able to continue executing reliably. Let errors crash the application and let the monitoring system alert you.

- **Exception** represents conditions that a reasonable application might want to catch and recover from. A file not found, a network timeout, or an invalid user input are all exceptional but recoverable conditions.

### Checked vs Unchecked Exceptions

This distinction is unique to Java and is one of the most debated design decisions in the language.

**Checked exceptions** extend `Exception` but NOT `RuntimeException`. The compiler forces you to either catch them or declare them in the method signature with `throws`.

```java
// The compiler knows that Files.readString can throw IOException.
// You MUST handle it or declare it.

// Option 1: Catch it
public String readConfig(Path path) {
    try {
        return Files.readString(path);
    } catch (IOException e) {
        log.error("Failed to read config: {}", path, e);
        return "{}";  // Fallback
    }
}

// Option 2: Declare it
public String readConfig(Path path) throws IOException {
    return Files.readString(path);  // Caller must now handle IOException
}
```

**Unchecked exceptions** extend `RuntimeException`. The compiler does not require you to catch or declare them. They represent programming errors (null references, invalid arguments, illegal state) that should be fixed in the code, not handled at runtime.

```java
// The compiler does NOT force you to handle NullPointerException.
// It is your responsibility to write code that does not produce null references.
public BigDecimal getBalance(Account account) {
    return account.getBalance();  // May throw NPE if account is null
}
```

**When to use each:**

| Scenario | Exception Type | Rationale |
|----------|---------------|-----------|
| Invalid method argument | Unchecked (`IllegalArgumentException`) | Programming error — fix the caller |
| Invalid object state | Unchecked (`IllegalStateException`) | Programming error — fix the lifecycle |
| Null reference | Unchecked (`NullPointerException`) | Programming error — fix the null |
| File not found | Checked (`FileNotFoundException`) | External condition — caller decides recovery |
| Network timeout | Checked (`TimeoutException`) | External condition — caller decides retry |
| SQL error | Checked (`SQLException`) | External condition — caller decides rollback |
| JSON parse error | Unchecked (`JsonProcessingException` in Jackson is actually checked, but many wrap it) | Depends on context |
| Payment gateway failure | Unchecked (custom `PaymentException`) | Business logic — propagate to controller |

**The modern consensus:** Most modern Java frameworks (Spring, Jakarta EE) favor unchecked exceptions. Spring wraps all checked `SQLException`s in unchecked `DataAccessException`s. The reasoning is that checked exceptions add boilerplate without improving reliability. Most callers cannot meaningfully recover from a checked exception and simply wrap it in a `RuntimeException` anyway. However, checked exceptions are still appropriate when the caller is expected to take a specific recovery action (e.g., retry, fallback, prompt the user).

### try / catch / finally

The `try-catch-finally` block is the fundamental exception handling construct.

```java
try {
    // Code that might throw an exception
    Transaction tx = paymentGateway.process(request);
    repository.save(tx);
} catch (PaymentDeclinedException e) {
    // Handle specific exception
    log.warn("Payment declined: {}", e.getReason());
    notifyCustomer(request.getCustomerId(), "Payment declined");
} catch (TimeoutException e) {
    // Handle another specific exception
    log.error("Payment gateway timeout", e);
    scheduleRetry(request);
} catch (Exception e) {
    // Catch-all for unexpected exceptions
    log.error("Unexpected error processing payment", e);
    throw new PaymentProcessingException("Internal error", e);
} finally {
    // ALWAYS executes, regardless of whether an exception was thrown
    // Use for cleanup: closing connections, releasing locks, resetting state
    metrics.recordPaymentAttempt(request.getPaymentMethod());
}
```

**Execution flow:**

1. The `try` block executes.
2. If no exception is thrown, the `catch` blocks are skipped. The `finally` block executes.
3. If an exception is thrown, the JVM searches the `catch` blocks in order for a matching type. The first matching `catch` block executes. The `finally` block executes.
4. If no `catch` block matches, the exception propagates to the caller. The `finally` block still executes before the exception leaves the method.
5. The `finally` block executes even if the `try` or `catch` block contains a `return` statement.

**Critical rule:** Order your `catch` blocks from most specific to most general. A `catch (Exception e)` block will catch everything, making subsequent `catch` blocks unreachable (compilation error).

```java
// WRONG — compilation error: FileNotFoundException is already caught by IOException
try {
    Files.readString(path);
} catch (Exception e) {
    // This catches everything — FileNotFoundException is unreachable below
} catch (FileNotFoundException e) {  // ERROR: unreachable
    // ...
}

// CORRECT — most specific first
try {
    Files.readString(path);
} catch (FileNotFoundException e) {
    log.error("File not found: {}", path);
} catch (IOException e) {
    log.error("I/O error reading file: {}", path, e);
} catch (Exception e) {
    log.error("Unexpected error", e);
}
```

### Multi-Catch (Java 7+)

When multiple exception types require the same handling logic, combine them in a single `catch` block using the `|` operator.

```java
try {
    Connection conn = DriverManager.getConnection(url, user, password);
    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery(query);
} catch (SQLException | TimeoutException e) {
    // Same handling for both types
    log.error("Database access failed", e);
    throw new DataAccessException("Cannot access database", e);
}
```

Rules:
- The exception variable (`e`) is implicitly `final`. You cannot reassign it.
- The catch parameter type is the nearest common supertype of all listed exceptions. In the example above, `e` is of type `Exception` (the common supertype of `SQLException` and `TimeoutException`).
- You cannot combine exceptions that have a parent-child relationship. `catch (IOException | FileNotFoundException e)` is a compilation error because `FileNotFoundException` is a subclass of `IOException`.

### try-with-resources (Java 7+)

The `try-with-resources` statement automatically closes resources that implement `AutoCloseable` (or `Closeable`). This eliminates the need for explicit `finally` blocks to close resources.

```java
// AutoCloseable interface
public interface AutoCloseable {
    void close() throws Exception;
}
```

**Basic usage:**

```java
// Single resource
try (BufferedReader reader = Files.newBufferedReader(path)) {
    String line = reader.readLine();
    processLine(line);
}  // reader.close() is called automatically here

// Multiple resources (closed in reverse order of declaration)
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql);
     ResultSet rs = stmt.executeQuery()) {
    while (rs.next()) {
        processRow(rs);
    }
}  // rs.close(), then stmt.close(), then conn.close() — automatically

// With catch and finally
try (InputStream in = Files.newInputStream(path)) {
    byte[] data = in.readAllBytes();
    processData(data);
} catch (IOException e) {
    log.error("Failed to read file: {}", path, e);
    throw new DataLoadException("Cannot load data", e);
} finally {
    log.info("File processing complete: {}", path);
}
```

**Why try-with-resources matters:**

Before Java 7, the correct pattern for closing resources was verbose and error-prone:

```java
// Legacy pattern — DO NOT WRITE CODE LIKE THIS
BufferedReader reader = null;
try {
    reader = Files.newBufferedReader(path);
    String line = reader.readLine();
} catch (IOException e) {
    log.error("Read failed", e);
} finally {
    if (reader != null) {
        try {
            reader.close();  // close() itself can throw!
        } catch (IOException e) {
            log.error("Close failed", e);  // This exception masks the original!
        }
    }
}
```

The legacy pattern has two problems: the `close()` call can throw an exception that masks the original exception, and the boilerplate is so verbose that developers often skip it, causing resource leaks. `try-with-resources` solves both problems. If both the `try` block and the `close()` method throw exceptions, the `close()` exception is **suppressed** and attached to the primary exception.

**Suppressed exceptions:**

```java
try (FaultyResource resource = new FaultyResource()) {
    resource.doWork();  // Throws PrimaryException
}  // resource.close() throws CloseException
// The PrimaryException is thrown.
// The CloseException is attached as a suppressed exception.

// Accessing suppressed exceptions:
try {
    processWithResource();
} catch (PrimaryException e) {
    for (Throwable suppressed : e.getSuppressed()) {
        log.error("Suppressed: {}", suppressed.getMessage());
    }
}
```

**Implementing AutoCloseable:**

```java
public class DatabaseTransaction implements AutoCloseable {
    private final Connection connection;
    private boolean committed = false;

    public DatabaseTransaction(Connection connection) throws SQLException {
        this.connection = connection;
        this.connection.setAutoCommit(false);
    }

    public void commit() throws SQLException {
        connection.commit();
        committed = true;
    }

    @Override
    public void close() throws SQLException {
        if (!committed) {
            connection.rollback();  // Auto-rollback if not committed
            log.warn("Transaction rolled back — commit() was not called");
        }
        connection.setAutoCommit(true);
    }
}

// Usage
try (DatabaseTransaction tx = new DatabaseTransaction(connection)) {
    accountRepository.debit(sourceAccount, amount);
    accountRepository.credit(targetAccount, amount);
    tx.commit();
}  // If commit() was not called (exception occurred), auto-rollback happens
```

### Throwing Exceptions

Use the `throw` keyword to explicitly throw an exception. Use the `throws` keyword in the method signature to declare that a method may throw a checked exception.

```java
// throw — creates and throws an exception instance
public void withdraw(BigDecimal amount) {
    if (amount.compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("Withdrawal amount must be positive");
    }
    if (amount.compareTo(balance) > 0) {
        throw new InsufficientFundsException(
            "Insufficient funds: requested %s, available %s".formatted(amount, balance)
        );
    }
    this.balance = this.balance.subtract(amount);
}

// throws — declares that this method may propagate a checked exception
public void processFile(Path path) throws IOException, ParseException {
    String content = Files.readString(path);  // May throw IOException
    Transaction tx = parseTransaction(content);  // May throw ParseException
    repository.save(tx);
}
```

**Rules for `throws`:**

- You must declare all checked exceptions that a method can throw (either directly or by propagation).
- You do not need to declare unchecked exceptions (`RuntimeException` and its subclasses), though you may document them with `@throws` in Javadoc.
- A method that overrides a superclass method cannot declare new checked exceptions that the superclass method does not declare. It can declare fewer or more specific checked exceptions.

```java
public interface PaymentProcessor {
    PaymentResult process(PaymentRequest request) throws PaymentException;
}

public class StripeProcessor implements PaymentProcessor {
    @Override
    public PaymentResult process(PaymentRequest request) throws PaymentException {
        // Can throw PaymentException or any subclass of PaymentException
        // Cannot throw IOException (not declared in the interface)
        // Can throw RuntimeException (unchecked, no declaration needed)
    }
}
```

### Custom Exception Classes

Custom exceptions communicate domain-specific errors. They make your code self-documenting and allow callers to handle specific failure modes.

**Designing a custom exception hierarchy:**

```java
// Base exception for the entire application
public class FintechException extends RuntimeException {
    private final String errorCode;

    public FintechException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }

    public FintechException(String message, String errorCode, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
    }

    public String getErrorCode() {
        return errorCode;
    }
}

// Domain-specific exceptions extending the base
public class PaymentException extends FintechException {
    public PaymentException(String message, String errorCode) {
        super(message, errorCode);
    }

    public PaymentException(String message, String errorCode, Throwable cause) {
        super(message, errorCode, cause);
    }
}

public class InsufficientFundsException extends PaymentException {
    private final BigDecimal requested;
    private final BigDecimal available;

    public InsufficientFundsException(BigDecimal requested, BigDecimal available) {
        super(
            "Insufficient funds: requested %s, available %s".formatted(requested, available),
            "PAYMENT_INSUFFICIENT_FUNDS"
        );
        this.requested = requested;
        this.available = available;
    }

    public BigDecimal getRequested() { return requested; }
    public BigDecimal getAvailable() { return available; }
}

public class AccountNotFoundException extends FintechException {
    public AccountNotFoundException(String accountId) {
        super("Account not found: " + accountId, "ACCOUNT_NOT_FOUND");
    }
}

public class DuplicateTransactionException extends PaymentException {
    public DuplicateTransactionException(String transactionId) {
        super("Duplicate transaction: " + transactionId, "PAYMENT_DUPLICATE");
    }
}
```

**Rules for custom exceptions:**

1. **Extend `RuntimeException` for business logic exceptions.** Most modern Java applications use unchecked exceptions for domain errors. The caller at the controller layer handles them globally via `@ExceptionHandler`.

2. **Extend `Exception` only when the caller must explicitly handle the error.** Checked exceptions are appropriate when there is a clear, specific recovery action the caller should take.

3. **Always provide a constructor that accepts a `Throwable cause`.** This enables exception chaining, which preserves the original stack trace.

4. **Include domain-relevant fields.** An `InsufficientFundsException` should carry the requested and available amounts. An `AccountNotFoundException` should carry the account ID. This information is essential for logging, error responses, and debugging.

5. **Include an error code.** Error codes are stable identifiers that API clients can use to handle specific errors programmatically. They are more reliable than parsing error messages.

6. **Do not use exceptions for flow control.** Exceptions are expensive to create (the JVM captures the full stack trace). Do not throw exceptions for expected conditions like "user not found" in a search. Use `Optional` or return null for expected absence.

### Exception Chaining

Exception chaining preserves the original cause of an error when you wrap a low-level exception in a higher-level one. This is critical for debugging because the root cause is often several layers deep.

```java
// Without chaining — the original cause is lost
public Account findAccount(String id) {
    try {
        return repository.findById(id);
    } catch (SQLException e) {
        throw new AccountNotFoundException(id);
        // The SQLException is gone. You have no idea WHY the query failed.
    }
}

// With chaining — the original cause is preserved
public Account findAccount(String id) {
    try {
        return repository.findById(id);
    } catch (SQLException e) {
        throw new AccountNotFoundException(id, e);
        // The SQLException is attached as the cause.
        // The stack trace shows both the AccountNotFoundException AND the SQLException.
    }
}
```

**The `cause` mechanism:**

```java
public class AccountNotFoundException extends FintechException {
    public AccountNotFoundException(String accountId, Throwable cause) {
        super("Account not found: " + accountId, "ACCOUNT_NOT_FOUND", cause);
        // The cause is passed to the superclass constructor,
        // which passes it to RuntimeException(String, Throwable),
        // which passes it to Exception(String, Throwable),
        // which stores it in the Throwable.cause field.
    }
}

// Accessing the cause
try {
    accountService.findAccount("ACC-999");
} catch (AccountNotFoundException e) {
    System.out.println(e.getMessage());       // "Account not found: ACC-999"
    System.out.println(e.getCause());         // java.sql.SQLException: Connection refused
    System.out.println(e.getCause().getMessage());  // "Connection refused"
}
```

**Stack trace with chaining:**

```
com.example.fintech.exception.AccountNotFoundException: Account not found: ACC-999
    at com.example.fintech.service.AccountService.findAccount(AccountService.java:23)
    at com.example.fintech.controller.AccountController.getAccount(AccountController.java:15)
    ...
Caused by: java.sql.SQLException: Connection refused
    at org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:315)
    at org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:51)
    ...
```

The `Caused by:` section is the chained exception. Without chaining, you would see only the `AccountNotFoundException` and have no idea that the real problem was a database connection failure.

### Best Practices

**1. Catch specific exceptions, not generic ones.**

```java
// BAD — catches everything, including NullPointerException and programming errors
try {
    processPayment(request);
} catch (Exception e) {
    log.error("Error", e);
}

// GOOD — catches only the exceptions you expect and can handle
try {
    processPayment(request);
} catch (PaymentDeclinedException e) {
    notifyCustomer(request.getCustomerId(), e.getReason());
} catch (TimeoutException e) {
    scheduleRetry(request);
} catch (IOException e) {
    log.error("Network error during payment processing", e);
    throw new PaymentProcessingException("Network error", e);
}
```

**2. Never swallow exceptions silently.**

```java
// BAD — the exception is caught and ignored. The bug is invisible.
try {
    processPayment(request);
} catch (Exception e) {
    // TODO: handle this later  (you will forget)
}

// BAD — printing to stdout is not logging. It disappears in production.
try {
    processPayment(request);
} catch (Exception e) {
    e.printStackTrace();  // Goes to stderr, not your log aggregation system
}

// GOOD — log the exception with full stack trace and re-throw or handle
try {
    processPayment(request);
} catch (PaymentException e) {
    log.error("Payment failed for request {}: {}", request.getId(), e.getMessage(), e);
    throw e;
}
```

**3. Fail fast.**

```java
// BAD — validates deep inside the processing logic
public void processPayment(PaymentRequest request) {
    connectToGateway();
    authenticateMerchant();
    if (request.getAmount().compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("Invalid amount");  // Too late!
    }
    // ...
}

// GOOD — validates at the entry point
public void processPayment(PaymentRequest request) {
    Objects.requireNonNull(request, "Request cannot be null");
    if (request.getAmount().compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("Amount must be positive");
    }
    if (request.getCurrency() == null || request.getCurrency().isBlank()) {
        throw new IllegalArgumentException("Currency is required");
    }
    // Now proceed with confidence
    connectToGateway();
    authenticateMerchant();
    // ...
}
```

**4. Use `@RestControllerAdvice` for global exception handling in Spring Boot.**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(InsufficientFundsException.class)
    public ResponseEntity<ProblemDetail> handleInsufficientFunds(InsufficientFundsException e) {
        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.UNPROCESSABLE_ENTITY, e.getMessage()
        );
        detail.setTitle("Insufficient Funds");
        detail.setProperty("errorCode", e.getErrorCode());
        detail.setProperty("requested", e.getRequested());
        detail.setProperty("available", e.getAvailable());
        return ResponseEntity.unprocessableEntity().body(detail);
    }

    @ExceptionHandler(AccountNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleNotFound(AccountNotFoundException e) {
        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.NOT_FOUND, e.getMessage()
        );
        detail.setTitle("Account Not Found");
        detail.setProperty("errorCode", e.getErrorCode());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(detail);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ProblemDetail> handleUnexpected(Exception e) {
        log.error("Unexpected error", e);
        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR, "An unexpected error occurred"
        );
        detail.setTitle("Internal Server Error");
        // Do NOT expose internal details to the client
        return ResponseEntity.internalServerError().body(detail);
    }
}
```

**5. Do not use exceptions for expected control flow.**

```java
// BAD — exception for expected "not found" scenario
public Account getAccount(String id) {
    try {
        return repository.findById(id);
    } catch (EntityNotFoundException e) {
        return null;  // Using exception for flow control
    }
}

// GOOD — Optional for expected absence
public Optional<Account> getAccount(String id) {
    return repository.findById(id);  // Returns Optional.empty() if not found
}
```

### Optional<T>

`Optional<T>` is a container object that may or may not contain a non-null value. It is Java's answer to the `NullPointerException` problem and an alternative to returning `null` or throwing exceptions for expected absence.

**Creating Optionals:**

```java
Optional<String> present = Optional.of("Hello");       // Value must be non-null
Optional<String> nullable = Optional.ofNullable(null);  // Value may be null → empty
Optional<String> empty = Optional.empty();              // Explicitly empty
```

**Using Optionals:**

```java
Optional<Account> account = accountRepository.findById("ACC-001");

// Check presence
if (account.isPresent()) {
    System.out.println(account.get().getOwnerName());
}

if (account.isEmpty()) {  // Java 11+
    System.out.println("Account not found");
}

// Provide defaults
Account result = account.orElse(defaultAccount);
Account result2 = account.orElseGet(() -> createDefaultAccount());  // Lazy evaluation
Account result3 = account.orElseThrow(() ->
    new AccountNotFoundException("ACC-001")
);

// Transform
Optional<String> ownerName = account.map(Account::getOwnerName);
Optional<BigDecimal> balance = account.map(Account::getBalance);

// Chain transformations
Optional<String> upperName = account
    .map(Account::getOwnerName)
    .map(String::toUpperCase)
    .filter(name -> name.length() > 3);

// FlatMap (when the mapping function returns an Optional)
Optional<BigDecimal> savingsBalance = account
    .flatMap(acc -> accountRepository.findSavingsAccount(acc.getOwnerId()))
    .map(Account::getBalance);

// Execute side effects
account.ifPresent(acc -> log.info("Found account: {}", acc.getAccountId()));
account.ifPresentOrElse(
    acc -> log.info("Found: {}", acc.getAccountId()),
    () -> log.warn("Account not found")
);  // Java 9+
```

**Optional best practices:**

- Use `Optional` as a **return type** for methods that may not find a result.
- Do NOT use `Optional` as a method parameter, a field, or a collection element. It adds complexity without benefit in those contexts.
- Do NOT call `Optional.get()` without first checking `isPresent()`. Use `orElseThrow()` instead — it is more explicit and produces a better error message.
- `Optional` is not a replacement for null checks in performance-critical code. It creates an object allocation per call. For hot paths, null checks are more efficient.

---

## Code Examples

**A complete exception handling example for a payment processing service:**

```java
package com.example.fintech.service;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.Objects;

public class PaymentService {

    private final AccountRepository accountRepository;
    private final PaymentGateway paymentGateway;
    private final TransactionRepository transactionRepository;

    public PaymentService(AccountRepository accountRepository,
                          PaymentGateway paymentGateway,
                          TransactionRepository transactionRepository) {
        this.accountRepository = accountRepository;
        this.paymentGateway = paymentGateway;
        this.transactionRepository = transactionRepository;
    }

    public PaymentResult processPayment(PaymentRequest request) {
        // 1. Fail fast — validate inputs
        Objects.requireNonNull(request, "Payment request cannot be null");
        if (request.getAmount().compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Payment amount must be positive");
        }
        if (request.getCurrency() == null || request.getCurrency().isBlank()) {
            throw new IllegalArgumentException("Currency is required");
        }

        // 2. Look up accounts — use Optional for expected absence
        Account sourceAccount = accountRepository.findById(request.getSourceAccountId())
            .orElseThrow(() -> new AccountNotFoundException(request.getSourceAccountId()));

        Account targetAccount = accountRepository.findById(request.getTargetAccountId())
            .orElseThrow(() -> new AccountNotFoundException(request.getTargetAccountId()));

        // 3. Check funds — domain-specific exception with context
        if (sourceAccount.getBalance().compareTo(request.getAmount()) < 0) {
            throw new InsufficientFundsException(
                request.getAmount(), sourceAccount.getBalance()
            );
        }

        // 4. Process via gateway — handle external failures with chaining
        GatewayResponse response;
        try {
            response = paymentGateway.charge(
                request.getSourceAccountId(),
                request.getAmount(),
                request.getCurrency()
            );
        } catch (TimeoutException e) {
            throw new PaymentProcessingException(
                "Payment gateway timeout after 30 seconds", "GATEWAY_TIMEOUT", e
            );
        } catch (GatewayConnectionException e) {
            throw new PaymentProcessingException(
                "Cannot connect to payment gateway", "GATEWAY_UNAVAILABLE", e
            );
        }

        if (!response.isSuccessful()) {
            throw new PaymentDeclinedException(
                response.getDeclineReason(), response.getDeclineCode()
            );
        }

        // 5. Update balances — use try-with-resources for transaction
        try (DatabaseTransaction tx = new DatabaseTransaction(dataSource.getConnection())) {
            sourceAccount.debit(request.getAmount());
            targetAccount.credit(request.getAmount());

            Transaction transaction = new Transaction(
                response.getTransactionId(),
                request.getAmount(),
                request.getCurrency(),
                LocalDateTime.now(),
                TransactionStatus.COMPLETED
            );
            transactionRepository.save(transaction);

            tx.commit();
            return PaymentResult.success(transaction);
        } catch (SQLException e) {
            throw new PaymentProcessingException(
                "Failed to persist transaction", "PERSISTENCE_ERROR", e
            );
        }
        // If we reach here without commit(), the DatabaseTransaction
        // auto-rolls back in its close() method.
    }
}
```

**Custom exception hierarchy:**

```java
// Base
public class FintechException extends RuntimeException {
    private final String errorCode;

    public FintechException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }

    public FintechException(String message, String errorCode, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
    }

    public String getErrorCode() { return errorCode; }
}

// Payment domain
public class PaymentException extends FintechException {
    public PaymentException(String message, String errorCode) {
        super(message, errorCode);
    }
    public PaymentException(String message, String errorCode, Throwable cause) {
        super(message, errorCode, cause);
    }
}

public class PaymentProcessingException extends PaymentException {
    public PaymentProcessingException(String message, String errorCode, Throwable cause) {
        super(message, errorCode, cause);
    }
}

public class PaymentDeclinedException extends PaymentException {
    private final String declineCode;

    public PaymentDeclinedException(String reason, String declineCode) {
        super("Payment declined: " + reason, "PAYMENT_DECLINED");
        this.declineCode = declineCode;
    }

    public String getDeclineCode() { return declineCode; }
}

public class InsufficientFundsException extends PaymentException {
    private final BigDecimal requested;
    private final BigDecimal available;

    public InsufficientFundsException(BigDecimal requested, BigDecimal available) {
        super(
            "Insufficient funds: requested %s, available %s".formatted(requested, available),
            "INSUFFICIENT_FUNDS"
        );
        this.requested = requested;
        this.available = available;
    }

    public BigDecimal getRequested() { return requested; }
    public BigDecimal getAvailable() { return available; }
}

// Account domain
public class AccountNotFoundException extends FintechException {
    public AccountNotFoundException(String accountId) {
        super("Account not found: " + accountId, "ACCOUNT_NOT_FOUND");
    }
}
```

**Global exception handler (Spring Boot):**

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(InsufficientFundsException.class)
    public ResponseEntity<ProblemDetail> handleInsufficientFunds(InsufficientFundsException e) {
        log.warn("Insufficient funds: requested={}, available={}",
            e.getRequested(), e.getAvailable());

        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.UNPROCESSABLE_ENTITY, e.getMessage()
        );
        detail.setTitle("Insufficient Funds");
        detail.setProperty("errorCode", e.getErrorCode());
        detail.setProperty("requested", e.getRequested());
        detail.setProperty("available", e.getAvailable());
        return ResponseEntity.unprocessableEntity().body(detail);
    }

    @ExceptionHandler(AccountNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleNotFound(AccountNotFoundException e) {
        log.warn("Account not found: {}", e.getMessage());

        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.NOT_FOUND, e.getMessage()
        );
        detail.setTitle("Not Found");
        detail.setProperty("errorCode", e.getErrorCode());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(detail);
    }

    @ExceptionHandler(PaymentDeclinedException.class)
    public ResponseEntity<ProblemDetail> handleDeclined(PaymentDeclinedException e) {
        log.warn("Payment declined: code={}", e.getDeclineCode());

        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.PAYMENT_REQUIRED, e.getMessage()
        );
        detail.setTitle("Payment Declined");
        detail.setProperty("errorCode", e.getErrorCode());
        detail.setProperty("declineCode", e.getDeclineCode());
        return ResponseEntity.status(HttpStatus.PAYMENT_REQUIRED).body(detail);
    }

    @ExceptionHandler(PaymentProcessingException.class)
    public ResponseEntity<ProblemDetail> handleProcessing(PaymentProcessingException e) {
        log.error("Payment processing error: {}", e.getMessage(), e);

        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.SERVICE_UNAVAILABLE, "Payment processing temporarily unavailable"
        );
        detail.setTitle("Service Unavailable");
        detail.setProperty("errorCode", e.getErrorCode());
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body(detail);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ProblemDetail> handleBadRequest(IllegalArgumentException e) {
        log.warn("Bad request: {}", e.getMessage());

        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.BAD_REQUEST, e.getMessage()
        );
        detail.setTitle("Bad Request");
        return ResponseEntity.badRequest().body(detail);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ProblemDetail> handleUnexpected(Exception e) {
        log.error("Unexpected error", e);

        ProblemDetail detail = ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR, "An unexpected error occurred"
        );
        detail.setTitle("Internal Server Error");
        return ResponseEntity.internalServerError().body(detail);
    }
}
```

---

## Important Notes

- The exception hierarchy is not arbitrary. `Error` means the JVM is broken — do not catch it. Checked exceptions mean an external condition failed — the caller must decide how to recover. Unchecked exceptions mean the code has a bug — fix the code, do not catch the exception.
- `try-with-resources` is mandatory for all I/O operations, database connections, and any resource that implements `AutoCloseable`. Failing to close resources causes file descriptor leaks, connection pool exhaustion, and eventual application crashes. There is no valid reason to use manual `finally` blocks for resource cleanup in modern Java.
- Exception chaining is non-negotiable in production code. When you catch a low-level exception and throw a higher-level one, always pass the original exception as the `cause`. Without chaining, the root cause of a production incident is lost, and debugging becomes a guessing game.
- Never catch `Exception` or `Throwable` at the service or repository layer. Catch specific exceptions that you can meaningfully handle. Let everything else propagate to the global exception handler at the controller layer. A blanket `catch (Exception e)` hides bugs and makes the system fail silently.
- Never use `e.printStackTrace()` in production code. It writes to `System.err`, which is not captured by your logging framework, not sent to your log aggregation system, and not correlated with request IDs. Always use `log.error("message", e)` with SLF4J.
- `Optional` is a return type, not a field type, not a parameter type, and not a collection element type. Using `Optional` as a field in a JPA entity or as a method parameter creates unnecessary complexity and breaks serialization. Use it only as a return type for methods that may not find a result.
- Do not use exceptions for expected control flow. Throwing and catching exceptions is expensive because the JVM captures the full stack trace at the point of creation. For expected conditions like "record not found," use `Optional`, return null, or return a result object. Reserve exceptions for truly exceptional situations.
- Custom exceptions should carry domain-relevant context. An `InsufficientFundsException` without the requested and available amounts is useless for debugging and error reporting. The exception object should contain everything needed to understand and respond to the error without reading the log message.
- Error codes in custom exceptions are more reliable than error messages for programmatic handling. API clients should check `errorCode == "INSUFFICIENT_FUNDS"`, not parse the human-readable message string. Messages change for localization and clarity. Error codes are stable contracts.
- The `finally` block executes even when the `try` block contains a `return` statement. This is a common source of confusion. If the `finally` block also contains a `return`, it overrides the `try` block's return value. Avoid returning from `finally` blocks — it masks exceptions and makes the code harder to reason about.
- In Spring Boot, the `@RestControllerAdvice` annotation creates a global exception handler that intercepts all exceptions thrown by controllers. This is the correct place to map exceptions to HTTP status codes and error response bodies. Do not handle exceptions in individual controller methods unless the handling is specific to that endpoint.
- The `ProblemDetail` class (Spring 6+, RFC 7807) is the standard format for error responses in modern REST APIs. It provides a consistent structure with `type`, `title`, `status`, `detail`, and `instance` fields, plus custom properties. Use it instead of inventing your own error response format.

---

## Practice

1. Create a custom exception hierarchy for a banking application. Include a base `BankingException`, then domain-specific exceptions for `AccountNotFoundException`, `InsufficientFundsException`, `AccountFrozenException`, `DailyLimitExceededException`, and `DuplicateTransactionException`. Each exception should carry relevant context fields and an error code.

2. Write a `transfer(String fromId, String toId, BigDecimal amount)` method that demonstrates all exception handling best practices: input validation (fail fast), account lookup with `Optional`, funds checking, gateway call with try-catch and chaining, and database transaction with try-with-resources.

3. Write a `@RestControllerAdvice` class that handles all the exceptions from exercise 1. Map each exception to the appropriate HTTP status code and return a `ProblemDetail` response with the error code and relevant context fields.

4. Demonstrate the difference between checked and unchecked exceptions. Write a method that throws a checked `IOException` and observe how the compiler forces the caller to handle it. Then write a method that throws an unchecked `IllegalArgumentException` and observe that the compiler does not complain. Document the tradeoffs.

5. Write a program that demonstrates suppressed exceptions. Create an `AutoCloseable` resource whose `close()` method throws an exception. Use it in a try-with-resources block where the `try` body also throws an exception. Catch the primary exception and inspect `getSuppressed()` to find the close exception.

6. Demonstrate the performance cost of exceptions. Write a benchmark that compares two approaches for looking up a value in a map: (1) using `map.get()` with a null check, and (2) using `map.get()` with a try-catch for a custom `KeyNotFoundException`. Run each approach 1,000,000 times and compare the execution time. Document the results.

7. In your Obsidian vault, create a decision flowchart: "Should I throw a checked exception, an unchecked exception, return Optional, or return null?" Include the criteria for each choice and examples from the fintech domain.

---

## References

- Java Language Specification — Chapter 11 (Exceptions): https://docs.oracle.com/javase/specs/jls/se21/html/jls-11.html
- Throwable API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html
- Optional API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html
- Spring Framework — Exception Handling: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html
- RFC 7807 (Problem Details for HTTP APIs): https://datatracker.ietf.org/doc/html/rfc7807
- "Effective Java" by Joshua Bloch — Items 69-77 (Exceptions)
