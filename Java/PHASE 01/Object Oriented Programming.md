## Overview

Java is an object-oriented language at its foundation. Every piece of code lives inside a class. Every value you manipulate (beyond primitives) is an object. The entire Java standard library, every Spring framework component, and every enterprise application you will build is structured around OOP principles. This is the most important section in Phase 01. Mastery of OOP in Java is not optional — it is the prerequisite for everything that follows.

---

## Core Concepts

### Classes and Objects

A **class** is a blueprint. An **object** is an instance of that blueprint.

```java
public class Account {
    // Fields (state)
    private String accountId;
    private String ownerName;
    private BigDecimal balance;
    private boolean isActive;

    // Constructor (initialization)
    public Account(String accountId, String ownerName, BigDecimal initialDeposit) {
        this.accountId = accountId;
        this.ownerName = ownerName;
        this.balance = initialDeposit;
        this.isActive = true;
    }

    // Methods (behavior)
    public void deposit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        this.balance = this.balance.add(amount);
    }

    public void withdraw(BigDecimal amount) {
        if (amount.compareTo(balance) > 0) {
            throw new IllegalStateException("Insufficient funds");
        }
        this.balance = this.balance.subtract(amount);
    }

    public BigDecimal getBalance() {
        return balance;
    }
}
```

**Creating objects:**

```java
Account savings = new Account("SAV-001", "Alice", new BigDecimal("5000.00"));
Account checking = new Account("CHK-001", "Bob", new BigDecimal("1200.00"));

savings.deposit(new BigDecimal("500.00"));
System.out.println(savings.getBalance());  // 5500.00
```

The `new` keyword allocates memory on the heap, calls the constructor, and returns a reference to the new object.

### Fields

Fields store an object's state. They are declared at the class level, outside any method.

```java
public class Transaction {
    // Instance fields (each object has its own copy)
    private String transactionId;
    private BigDecimal amount;
    private LocalDateTime timestamp;

    // Static field (shared across ALL instances of the class)
    private static long transactionCount = 0;

    // Final field (cannot be reassigned after initialization)
    private final String currency;
}
```

**Instance vs static fields:**

- **Instance fields** belong to a specific object. Each `Account` has its own `balance`.
- **Static fields** belong to the class itself. All `Transaction` objects share the same `transactionCount`.

### Methods

Methods define an object's behavior. They operate on the object's fields and can accept parameters and return values.

```java
public class Account {
    private BigDecimal balance;

    // Instance method (operates on a specific object's state)
    public BigDecimal getBalance() {
        return balance;
    }

    // Method with parameters and validation
    public void transfer(Account target, BigDecimal amount) {
        this.withdraw(amount);
        target.deposit(amount);
    }

    // Static method (does not operate on instance state)
    public static Account createSavingsAccount(String owner) {
        return new Account(generateId(), owner, BigDecimal.ZERO);
    }
}
```

### Constructors

Constructors initialize new objects. They have the same name as the class and no return type.

```java
public class Account {
    private String id;
    private String owner;
    private BigDecimal balance;
    private AccountType type;

    // Default constructor (no arguments)
    public Account() {
        this.id = UUID.randomUUID().toString();
        this.owner = "Unknown";
        this.balance = BigDecimal.ZERO;
        this.type = AccountType.CHECKING;
    }

    // Parameterized constructor
    public Account(String owner, BigDecimal initialDeposit) {
        this.id = UUID.randomUUID().toString();
        this.owner = owner;
        this.balance = initialDeposit;
        this.type = AccountType.CHECKING;
    }

    // Full constructor
    public Account(String id, String owner, BigDecimal balance, AccountType type) {
        this.id = id;
        this.owner = owner;
        this.balance = balance;
        this.type = type;
    }

    // Constructor chaining (calling another constructor with this())
    public Account(String owner, AccountType type) {
        this(owner, BigDecimal.ZERO);  // Calls the 2-arg constructor
        this.type = type;              // Overrides the type
    }
}
```

Rules:
- If you do not define any constructor, Java provides a default no-argument constructor that initializes all fields to their default values.
- If you define any constructor, the default constructor is no longer generated automatically.
- `this()` must be the first statement in a constructor when chaining.
- Constructor overloading (multiple constructors with different parameters) is common and expected.

### The `this` Keyword

`this` refers to the current object instance.

```java
public class Account {
    private String owner;
    private BigDecimal balance;

    public Account(String owner, BigDecimal balance) {
        this.owner = owner;      // Distinguishes field from parameter
        this.balance = balance;
    }

    public Account deposit(BigDecimal amount) {
        this.balance = this.balance.add(amount);
        return this;  // Enables method chaining
    }

    public Account withdraw(BigDecimal amount) {
        this.balance = this.balance.subtract(amount);
        return this;
    }
}

// Method chaining usage:
account.deposit(new BigDecimal("500")).withdraw(new BigDecimal("100"));
```

### The `static` Keyword

`static` members belong to the class, not to any instance.

```java
public class CurrencyConverter {
    // Static field — shared across all instances
    private static Map<String, BigDecimal> rates = new HashMap<>();

    // Static initializer block — runs once when the class is loaded
    static {
        rates.put("USD", BigDecimal.ONE);
        rates.put("EUR", new BigDecimal("0.92"));
        rates.put("GBP", new BigDecimal("0.79"));
        rates.put("JPY", new BigDecimal("149.50"));
    }

    // Static method — can be called without an instance
    public static BigDecimal convert(BigDecimal amount, String from, String to) {
        BigDecimal usdAmount = amount.divide(rates.get(from), 10, RoundingMode.HALF_UP);
        return usdAmount.multiply(rates.get(to)).setScale(2, RoundingMode.HALF_UP);
    }

    // Static nested class
    public static class ConversionResult {
        private final BigDecimal amount;
        private final String currency;

        public ConversionResult(BigDecimal amount, String currency) {
            this.amount = amount;
            this.currency = currency;
        }
    }
}

// Usage — no instance needed:
BigDecimal euros = CurrencyConverter.convert(new BigDecimal("1000"), "USD", "EUR");
```

**When to use static:**
- Utility methods that do not depend on instance state (`Math.abs()`, `Collections.sort()`)
- Constants (`public static final`)
- Factory methods (`List.of()`, `Optional.of()`)
- Singleton pattern (covered later)

**When NOT to use static:**
- Business logic that operates on instance data
- Methods you might want to override in subclasses (static methods cannot be overridden)
- Anything that makes testing difficult (static dependencies are hard to mock)

### Access Modifiers

Access modifiers control visibility. They are the foundation of encapsulation.

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `public` | Yes | Yes | Yes | Yes |
| `protected` | Yes | Yes | Yes | No |
| (default) | Yes | Yes | No | No |
| `private` | Yes | No | No | No |

```java
public class Account {
    private BigDecimal balance;       // Only this class
    protected String accountType;     // This class + subclasses + same package
    String branchCode;                // This class + same package (default/package-private)
    public String accountId;          // Everyone (avoid public fields in practice)
}
```

**Best practice:** Make all fields `private`. Expose behavior through public methods. Use `protected` sparingly for methods that subclasses need to override. Use default (package-private) for classes and methods that are internal to a module.

### Getters and Setters

Getters and setters provide controlled access to private fields.

```java
public class Account {
    private BigDecimal balance;
    private String owner;

    // Getter
    public BigDecimal getBalance() {
        return balance;
    }

    // Setter with validation
    public void setOwner(String owner) {
        if (owner == null || owner.isBlank()) {
            throw new IllegalArgumentException("Owner name cannot be blank");
        }
        this.owner = owner;
    }

    // Read-only property (getter without setter)
    public String getAccountId() {
        return accountId;
    }

    // Do NOT provide a setter for balance.
    // Balance changes only through deposit() and withdraw().
    // This is encapsulation in action.
}
```

**Important:** Not every field needs a getter and setter. Expose only what the caller needs. If a field should not be modified externally, do not provide a setter. The balance example above is critical — allowing direct `setBalance()` would bypass all business rules.

### The `equals()` and `hashCode()` Contract

This is one of the most important concepts in Java. Getting it wrong causes subtle, hard-to-debug failures in collections.

**The contract:**

1. If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true.
2. If `a.hashCode() == b.hashCode()`, `a.equals(b)` is NOT necessarily true (hash collisions are allowed).
3. `equals()` must be reflexive (`a.equals(a)`), symmetric (`a.equals(b) == b.equals(a)`), and transitive.
4. `equals(null)` must return false.

```java
public class Money {
    private final BigDecimal amount;
    private final String currency;

    public Money(BigDecimal amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                          // Same reference
        if (o == null || getClass() != o.getClass()) return false;  // Null or different type
        Money money = (Money) o;
        return amount.compareTo(money.amount) == 0
            && currency.equals(money.currency);
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount.stripTrailingZeros(), currency);
    }
}
```

**Why this matters:**

```java
Set<Money> uniqueAmounts = new HashSet<>();
uniqueAmounts.add(new Money(new BigDecimal("100.00"), "USD"));
uniqueAmounts.add(new Money(new BigDecimal("100.00"), "USD"));

System.out.println(uniqueAmounts.size());
// 1 if equals/hashCode are correct
// 2 if equals/hashCode are missing (uses Object's default — reference equality)
```

If you use an object as a key in a `HashMap` or an element in a `HashSet`, you MUST override both `equals()` and `hashCode()`. Failing to do so is one of the most common bugs in Java.

**Modern shortcut — Records (Java 16+):**

Records automatically generate `equals()`, `hashCode()`, `toString()`, getters, and a constructor.

```java
public record Money(BigDecimal amount, String currency) {}

// That single line gives you:
// - Constructor: new Money(new BigDecimal("100"), "USD")
// - Getters: money.amount(), money.currency()
// - equals(), hashCode(), toString() — all correct
```

Use records for immutable data carriers (DTOs, value objects, configuration). They eliminate boilerplate and guarantee correctness.

### Immutable Objects

An immutable object cannot be modified after creation. Immutable objects are inherently thread-safe, simpler to reason about, and safe to use as HashMap keys.

**Rules for creating immutable objects:**

1. Declare the class `final` (or use a record).
2. Make all fields `private` and `final`.
3. Do not provide setters.
4. Initialize all fields in the constructor.
5. If a field is a mutable object (List, Date, array), make a defensive copy in the constructor and in the getter.

```java
public final class Transaction {
    private final String id;
    private final BigDecimal amount;
    private final String currency;
    private final LocalDateTime timestamp;
    private final List<String> tags;

    public Transaction(String id, BigDecimal amount, String currency,
                       LocalDateTime timestamp, List<String> tags) {
        this.id = id;
        this.amount = amount;
        this.currency = currency;
        this.timestamp = timestamp;
        this.tags = List.copyOf(tags);  // Defensive copy — unmodifiable
    }

    public String getId() { return id; }
    public BigDecimal getAmount() { return amount; }
    public String getCurrency() { return currency; }
    public LocalDateTime getTimestamp() { return timestamp; }
    public List<String> getTags() { return tags; }  // Already unmodifiable
}
```

---

## Inheritance

### The `extends` Keyword

A subclass inherits fields and methods from a superclass.

```java
public class BankAccount {
    protected String accountId;
    protected BigDecimal balance;

    public BankAccount(String accountId, BigDecimal initialBalance) {
        this.accountId = accountId;
        this.balance = initialBalance;
    }

    public void deposit(BigDecimal amount) {
        this.balance = this.balance.add(amount);
    }

    public BigDecimal getBalance() {
        return balance;
    }
}

public class SavingsAccount extends BankAccount {
    private BigDecimal interestRate;

    public SavingsAccount(String accountId, BigDecimal initialBalance,
                          BigDecimal interestRate) {
        super(accountId, initialBalance);  // Call superclass constructor
        this.interestRate = interestRate;
    }

    public void applyInterest() {
        BigDecimal interest = balance.multiply(interestRate)
            .divide(new BigDecimal("100"), 2, RoundingMode.HALF_UP);
        deposit(interest);  // Inherited method
    }
}
```

**Key rules:**
- Java supports **single inheritance** only. A class can extend only one superclass.
- The `super()` call must be the first statement in the subclass constructor.
- If you do not call `super()` explicitly, Java inserts `super()` (no-arg) automatically. If the superclass has no no-arg constructor, this causes a compilation error.
- `private` members of the superclass are NOT accessible in the subclass. Use `protected` for members that subclasses need.
- All classes implicitly extend `java.lang.Object`.

### Method Overriding

A subclass can provide its own implementation of a method defined in the superclass.

```java
public class BankAccount {
    public String getAccountType() {
        return "STANDARD";
    }

    public BigDecimal calculateMonthlyFee() {
        return new BigDecimal("5.00");
    }
}

public class PremiumAccount extends BankAccount {
    @Override  // Always use this annotation
    public String getAccountType() {
        return "PREMIUM";
    }

    @Override
    public BigDecimal calculateMonthlyFee() {
        return BigDecimal.ZERO;  // No fee for premium accounts
    }
}
```

**Rules for overriding:**
- The method signature (name + parameter types) must match exactly.
- The return type must be the same or a subtype (covariant return).
- The access modifier cannot be more restrictive (you can widen from `protected` to `public`, but not from `public` to `protected`).
- You cannot override `final` methods.
- You cannot override `static` methods (they are hidden, not overridden).
- Always use `@Override`. The compiler will flag errors if the method does not actually override anything.

### The `Object` Class

Every class in Java inherits from `java.lang.Object`. Its key methods:

| Method | Purpose | Override? |
|--------|---------|-----------|
| `equals(Object)` | Equality comparison | Yes, for value-based classes |
| `hashCode()` | Hash code for collections | Yes, whenever you override equals |
| `toString()` | String representation | Yes, for debugging and logging |
| `getClass()` | Runtime class information | No (final) |
| `clone()` | Object copying | Rarely (prefer copy constructors) |
| `finalize()` | Cleanup before GC | No (deprecated since Java 9) |

```java
@Override
public String toString() {
    return "Account{id='%s', owner='%s', balance=%s}".formatted(
        accountId, ownerName, balance
    );
}
```

### Sealed Classes (Java 17)

Sealed classes restrict which classes can extend them. This is useful for modeling fixed hierarchies.

```java
public sealed interface PaymentMethod
    permits CreditCard, BankTransfer, CryptoWallet {
    BigDecimal getAmount();
}

public record CreditCard(String cardNumber, BigDecimal amount) implements PaymentMethod {}
public record BankTransfer(String routingNumber, BigDecimal amount) implements PaymentMethod {}
public record CryptoWallet(String address, BigDecimal amount) implements PaymentMethod {}

// No other class can implement PaymentMethod.
// The compiler enforces this.
```

Sealed classes work exceptionally well with pattern-matching switch (covered in 01.03):

```java
String description = switch (payment) {
    case CreditCard cc -> "Card ending in " + cc.cardNumber().substring(12);
    case BankTransfer bt -> "Transfer from routing " + bt.routingNumber();
    case CryptoWallet cw -> "Wallet " + cw.address();
};
// No default needed — the compiler knows all possible subtypes.
```

---

## Polymorphism

### Compile-Time Polymorphism (Method Overloading)

Multiple methods with the same name but different parameter lists.

```java
public class AccountService {
    public Account findAccount(String accountId) { ... }
    public Account findAccount(String accountId, boolean includeClosed) { ... }
    public List<Account> findAccount(String ownerName, AccountType type) { ... }
}
```

### Runtime Polymorphism (Method Overriding)

The actual method called is determined by the runtime type of the object, not the compile-time type of the reference.

```java
BankAccount account = new SavingsAccount("SAV-001", new BigDecimal("1000"), new BigDecimal("2.5"));

// The compiler sees BankAccount, but the JVM calls SavingsAccount's methods
System.out.println(account.getAccountType());  // "SAVINGS" (if overridden)
account.deposit(new BigDecimal("500"));         // Calls inherited method
```

### `instanceof` and Pattern Matching (Java 16+)

```java
// Old style (pre-Java 16)
if (event instanceof PaymentEvent) {
    PaymentEvent pe = (PaymentEvent) event;  // Explicit cast
    processPayment(pe.getTransactionId());
}

// Modern style (Java 16+)
if (event instanceof PaymentEvent pe) {
    processPayment(pe.getTransactionId());  // pe is already cast
}

// With pattern variable in condition
if (event instanceof PaymentEvent pe && pe.getAmount().compareTo(BigDecimal.ZERO) > 0) {
    processPayment(pe.getTransactionId());
}
```

---

## Abstraction

### Abstract Classes

An abstract class cannot be instantiated. It may contain abstract methods (no implementation) that subclasses must implement.

```java
public abstract class Notification {
    protected final String recipient;
    protected final String message;

    protected Notification(String recipient, String message) {
        this.recipient = recipient;
        this.message = message;
    }

    // Abstract method — subclasses MUST implement
    public abstract void send();

    // Concrete method — shared logic
    public void log() {
        System.out.printf("[%s] Notification to %s: %s%n",
            getClass().getSimpleName(), recipient, message);
    }
}

public class EmailNotification extends Notification {
    public EmailNotification(String email, String message) {
        super(email, message);
    }

    @Override
    public void send() {
        log();
        // Send email via SMTP
        System.out.println("Sending email to " + recipient);
    }
}

public class SmsNotification extends Notification {
    public SmsNotification(String phoneNumber, String message) {
        super(phoneNumber, message);
    }

    @Override
    public void send() {
        log();
        // Send SMS via gateway
        System.out.println("Sending SMS to " + recipient);
    }
}
```

### Interfaces

An interface defines a contract. A class can implement multiple interfaces (unlike single class inheritance).

```java
public interface Auditable {
    void recordAudit(String action, String actor);
    List<AuditEntry> getAuditTrail();
}

public interface Encryptable {
    void encrypt();
    void decrypt();
}

// A class can implement multiple interfaces
public class Transaction implements Auditable, Encryptable {
    @Override
    public void recordAudit(String action, String actor) { ... }

    @Override
    public List<AuditEntry> getAuditTrail() { ... }

    @Override
    public void encrypt() { ... }

    @Override
    public void decrypt() { ... }
}
```

**Interface features (Java 8+):**

```java
public interface Repository<T, ID> {
    // Abstract method (implicitly public abstract)
    T findById(ID id);

    // Default method (Java 8+) — provides a default implementation
    default List<T> findAll() {
        return findAll(0, Integer.MAX_VALUE);
    }

    default List<T> findAll(int offset, int limit) {
        throw new UnsupportedOperationException("Pagination not supported");
    }

    // Static method (Java 8+)
    static <T> Repository<T, Long> empty() {
        return id -> null;
    }

    // Private method (Java 9+) — shared logic between default methods
    private void validateId(ID id) {
        if (id == null) throw new IllegalArgumentException("ID cannot be null");
    }
}
```

**Abstract class vs interface:**

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Multiple inheritance | No | Yes |
| State (fields) | Yes | Only `public static final` constants |
| Constructors | Yes | No |
| Access modifiers | All | Only `public` (implicitly) |
| When to use | Shared implementation + state | Contract / capability |

**Guideline:** Use interfaces to define what an object can do (`Auditable`, `Serializable`, `Comparable`). Use abstract classes to define what an object is when there is shared implementation (`BankAccount` → `SavingsAccount`).

### Functional Interfaces

A functional interface has exactly one abstract method. It can be used with lambda expressions.

```java
@FunctionalInterface
public interface TransactionValidator {
    boolean validate(Transaction transaction);
    // Only one abstract method allowed
    // Default and static methods do not count
}

// Usage with lambda:
TransactionValidator largeAmountValidator = tx ->
    tx.getAmount().compareTo(new BigDecimal("10000")) <= 0;

TransactionValidator activeAccountValidator = tx ->
    tx.getAccount().isActive();

// Combining:
TransactionValidator combined = tx ->
    largeAmountValidator.validate(tx) && activeAccountValidator.validate(tx);
```

The `java.util.function` package provides many built-in functional interfaces: `Function<T,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>`, `BiFunction<T,U,R>`, and more. Covered in depth in 01.10.

---

## Inner Classes

```java
public class PaymentProcessor {
    private String gatewayUrl;

    // 1. Member inner class (has access to outer class instance)
    public class PaymentResult {
        private boolean success;
        private String message;

        public PaymentResult(boolean success, String message) {
            this.success = success;
            this.message = message;
        }

        public String getGateway() {
            return gatewayUrl;  // Accesses outer class field
        }
    }

    // 2. Static nested class (no access to outer instance — preferred)
    public static class PaymentRequest {
        private final BigDecimal amount;
        private final String currency;

        public PaymentRequest(BigDecimal amount, String currency) {
            this.amount = amount;
            this.currency = currency;
        }
    }

    // 3. Local inner class (defined inside a method)
    public void process(BigDecimal amount) {
        class Logger {
            void log(String msg) {
                System.out.println("[PaymentProcessor] " + msg);
            }
        }
        Logger logger = new Logger();
        logger.log("Processing " + amount);
    }

    // 4. Anonymous inner class (one-time use, largely replaced by lambdas)
    public void addListener() {
        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("Clicked");
            }
        });
        // Modern equivalent with lambda:
        // button.addActionListener(e -> System.out.println("Clicked"));
    }
}
```

**In practice:** Static nested classes are the most common. Use them for closely related helper classes (e.g., `Map.Entry`, `Builder` pattern). Avoid non-static inner classes unless you specifically need access to the outer instance.

---

## Enums

Enums are special classes that represent a fixed set of constants. They are far more powerful in Java than in most languages.

```java
public enum TransactionStatus {
    PENDING("Transaction is awaiting processing"),
    PROCESSING("Transaction is being processed"),
    COMPLETED("Transaction completed successfully"),
    FAILED("Transaction failed"),
    CANCELLED("Transaction was cancelled"),
    REFUNDED("Transaction has been refunded");

    private final String description;

    // Enum constructor (implicitly private)
    TransactionStatus(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }

    // Enum methods
    public boolean isTerminal() {
        return this == COMPLETED || this == FAILED || this == CANCELLED || this == REFUNDED;
    }

    public boolean canTransitionTo(TransactionStatus target) {
        return switch (this) {
            case PENDING -> target == PROCESSING || target == CANCELLED;
            case PROCESSING -> target == COMPLETED || target == FAILED;
            case COMPLETED -> target == REFUNDED;
            case FAILED, CANCELLED, REFUNDED -> false;
        };
    }
}

// Usage:
TransactionStatus status = TransactionStatus.PENDING;
System.out.println(status.name());         // "PENDING"
System.out.println(status.ordinal());      // 0
System.out.println(status.getDescription()); // "Transaction is awaiting processing"
System.out.println(status.isTerminal());   // false

// Iterating:
for (TransactionStatus s : TransactionStatus.values()) {
    System.out.println(s + ": " + s.getDescription());
}

// Parsing from string:
TransactionStatus parsed = TransactionStatus.valueOf("COMPLETED");
```

**Enums implementing interfaces:**

```java
public interface Calculable {
    BigDecimal calculate(BigDecimal principal);
}

public enum FeeType implements Calculable {
    FLAT {
        @Override
        public BigDecimal calculate(BigDecimal principal) {
            return new BigDecimal("2.50");
        }
    },
    PERCENTAGE {
        @Override
        public BigDecimal calculate(BigDecimal principal) {
            return principal.multiply(new BigDecimal("0.015"))
                .setScale(2, RoundingMode.HALF_UP);
        }
    },
    TIERED {
        @Override
        public BigDecimal calculate(BigDecimal principal) {
            if (principal.compareTo(new BigDecimal("10000")) > 0) {
                return principal.multiply(new BigDecimal("0.005"));
            }
            return principal.multiply(new BigDecimal("0.01"));
        }
    }
}
```

**EnumMap and EnumSet:**

```java
// EnumMap — faster than HashMap for enum keys
Map<TransactionStatus, Integer> statusCounts = new EnumMap<>(TransactionStatus.class);
statusCounts.put(TransactionStatus.COMPLETED, 150);
statusCounts.put(TransactionStatus.FAILED, 3);

// EnumSet — faster than HashSet for enum values
Set<TransactionStatus> activeStatuses = EnumSet.of(
    TransactionStatus.PENDING,
    TransactionStatus.PROCESSING
);
Set<TransactionStatus> allTerminal = EnumSet.range(
    TransactionStatus.COMPLETED,
    TransactionStatus.REFUNDED
);
```

---

## Code Examples

**A complete OOP hierarchy for a banking domain:**

```java
package com.example.bank;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.time.LocalDateTime;
import java.util.*;

// Interface
public interface InterestBearing {
    BigDecimal calculateInterest();
    BigDecimal getInterestRate();
}

// Abstract class
public abstract class Account {
    private final String accountId;
    private final String ownerName;
    protected BigDecimal balance;
    private final LocalDateTime createdAt;
    private final List<Transaction> history;

    protected Account(String accountId, String ownerName, BigDecimal initialDeposit) {
        this.accountId = accountId;
        this.ownerName = ownerName;
        this.balance = initialDeposit;
        this.createdAt = LocalDateTime.now();
        this.history = new ArrayList<>();
        history.add(new Transaction("INITIAL_DEPOSIT", initialDeposit));
    }

    public void deposit(BigDecimal amount) {
        validatePositive(amount);
        balance = balance.add(amount);
        history.add(new Transaction("DEPOSIT", amount));
    }

    public void withdraw(BigDecimal amount) {
        validatePositive(amount);
        if (amount.compareTo(balance) > 0) {
            throw new IllegalStateException("Insufficient funds");
        }
        balance = balance.subtract(amount);
        history.add(new Transaction("WITHDRAWAL", amount.negate()));
    }

    public abstract BigDecimal getMonthlyFee();

    protected void validatePositive(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }

    public String getAccountId() { return accountId; }
    public String getOwnerName() { return ownerName; }
    public BigDecimal getBalance() { return balance; }
    public List<Transaction> getHistory() { return Collections.unmodifiableList(history); }

    @Override
    public String toString() {
        return "%s{id='%s', owner='%s', balance=%s}".formatted(
            getClass().getSimpleName(), accountId, ownerName, balance);
    }
}

// Concrete subclass
public class SavingsAccount extends Account implements InterestBearing {
    private final BigDecimal interestRate;

    public SavingsAccount(String accountId, String ownerName,
                          BigDecimal initialDeposit, BigDecimal interestRate) {
        super(accountId, ownerName, initialDeposit);
        this.interestRate = interestRate;
    }

    @Override
    public BigDecimal getMonthlyFee() {
        return BigDecimal.ZERO;
    }

    @Override
    public BigDecimal calculateInterest() {
        return getBalance().multiply(interestRate)
            .divide(new BigDecimal("12"), 2, RoundingMode.HALF_UP);
    }

    @Override
    public BigDecimal getInterestRate() {
        return interestRate;
    }
}

// Concrete subclass
public class CheckingAccount extends Account {
    private final int freeTransactions;
    private int transactionCount;

    public CheckingAccount(String accountId, String ownerName,
                           BigDecimal initialDeposit, int freeTransactions) {
        super(accountId, ownerName, initialDeposit);
        this.freeTransactions = freeTransactions;
        this.transactionCount = 0;
    }

    @Override
    public void deposit(BigDecimal amount) {
        super.deposit(amount);
        transactionCount++;
    }

    @Override
    public void withdraw(BigDecimal amount) {
        super.withdraw(amount);
        transactionCount++;
    }

    @Override
    public BigDecimal getMonthlyFee() {
        if (transactionCount > freeTransactions) {
            return new BigDecimal("0.25")
                .multiply(BigDecimal.valueOf(transactionCount - freeTransactions));
        }
        return BigDecimal.ZERO;
    }
}

// Record for immutable transaction data
public record Transaction(String type, BigDecimal amount, LocalDateTime timestamp) {
    public Transaction(String type, BigDecimal amount) {
        this(type, amount, LocalDateTime.now());
    }
}

// Demonstration
public class BankDemo {
    public static void main(String[] args) {
        // Polymorphism: Account reference, different runtime types
        List<Account> accounts = List.of(
            new SavingsAccount("SAV-001", "Alice", new BigDecimal("10000"), new BigDecimal("0.04")),
            new CheckingAccount("CHK-001", "Bob", new BigDecimal("2500"), 10)
        );

        for (Account account : accounts) {
            account.deposit(new BigDecimal("500"));
            System.out.println(account);
            System.out.println("Monthly fee: $" + account.getMonthlyFee());

            if (account instanceof InterestBearing ib) {
                System.out.println("Monthly interest: $" + ib.calculateInterest());
            }
            System.out.println();
        }
    }
}
```

---

## Important Notes

- Every Java application is built on OOP. Spring Boot controllers, services, repositories, entities, DTOs — they are all classes. Understanding OOP is not an academic exercise. It is the daily reality of Java development.
- Make fields `private` by default. Expose behavior through methods, not data through getters. A class that is nothing but getters and setters is a procedural data structure wearing an OOP costume. Real OOP encapsulates behavior with data.
- The `equals()`/`hashCode()` contract is non-negotiable. If you use an object in a `HashMap`, `HashSet`, or any hash-based collection, you must override both methods correctly. Use your IDE's generation feature or use records to avoid manual errors.
- Records (Java 16+) are the preferred way to model immutable data carriers. Use them for DTOs, value objects, API responses, and configuration objects. They eliminate boilerplate and guarantee correct `equals()`/`hashCode()`.
- Prefer composition over inheritance. Inheritance creates tight coupling between parent and child. If you find yourself building deep inheritance hierarchies (more than 2-3 levels), reconsider. Composition (having a field of another class) is more flexible and easier to test.
- Interfaces define capabilities. Abstract classes define identity. A `Transaction` is not a type of `Auditable` — it has the capability of being audited. Use interfaces for capabilities. Use abstract classes when subclasses share significant implementation and state.
- Static methods and fields should be used sparingly in business logic. They create hidden dependencies that make testing difficult. A static call to `CurrencyConverter.convert()` cannot be mocked in a unit test. An injected `CurrencyConverter` instance can.
- Enums in Java are full-featured classes. They can have fields, constructors, methods, and implement interfaces. Use enums whenever you have a fixed set of related constants. Never use string constants (`public static final String STATUS_PENDING = "PENDING"`) when an enum is appropriate.
- The `@Override` annotation is mandatory in professional code. It catches typos and signature mismatches at compile time. If you intend to override a method and forget `@Override`, a subtle change in the superclass signature will silently break your code.
- Sealed classes (Java 17) combined with pattern-matching switch (Java 21) provide exhaustive type checking at compile time. This is the closest Java gets to algebraic data types in functional languages. Use them for domain models with a fixed set of variants (payment methods, event types, transaction statuses).
- Immutable objects are thread-safe by definition. In concurrent fintech systems, immutability eliminates entire categories of race conditions. Favor immutability wherever possible. Use `final` fields, `List.of()`, `Map.of()`, and records.

---

## Practice

1. Create a class hierarchy for a payment system. Start with an abstract `Payment` class with subclasses `CreditCardPayment`, `BankTransferPayment`, and `CryptoPayment`. Each subclass should have unique fields and override a `process()` method. Add a sealed interface `PaymentStatus` with records for `Success`, `Failed`, and `Pending`.

2. Implement a `Money` class with `BigDecimal amount` and `String currency`. Override `equals()`, `hashCode()`, and `toString()`. Add methods for `add(Money)`, `subtract(Money)`, and `convertTo(String targetCurrency)`. Ensure that adding two `Money` objects with different currencies throws an exception. Then rewrite the entire class as a record and compare the code size.

3. Create an interface `Taxable` with a method `BigDecimal calculateTax()`. Create an abstract class `Product` that implements `Taxable`. Create subclasses `DigitalProduct` (no tax) and `PhysicalProduct` (8% tax). Demonstrate polymorphism by storing both types in a `List<Product>` and calculating total tax.

4. Build an enum `Currency` with at least five currencies. Each enum constant should have a symbol, a decimal places count, and a conversion rate to USD. Add a method `convert(BigDecimal amount, Currency target)` that converts between any two currencies.

5. Create an immutable `Transaction` class following all five rules of immutability. Include a `List<String> tags` field and demonstrate the defensive copy in the constructor and getter. Write a test that proves the returned list cannot be modified.

6. In your Obsidian vault, create a decision flowchart: "Should I use an abstract class, an interface, a record, or an enum?" Include the criteria for each choice.

---

## References

- Java Language Specification — Chapter 8 (Classes): https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html
- Java Language Specification — Chapter 9 (Interfaces): https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html
- JEP 395 (Records, Java 16): https://openjdk.org/jeps/395
- JEP 409 (Sealed Classes, Java 17): https://openjdk.org/jeps/409
- "Effective Java" by Joshua Bloch — Items 10-14 (equals, hashCode, toString, Comparable), Item 18 (Prefer composition to inheritance), Item 20 (Prefer interfaces to abstract classes)
