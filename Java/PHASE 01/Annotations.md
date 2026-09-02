## Overview

Annotations are a form of metadata that you attach to Java declarations — classes, methods, fields, parameters, and packages. They do not change the behavior of the code they annotate by themselves. Instead, they provide information that tools, frameworks, and the compiler can read and act upon. Annotations are the backbone of modern Java frameworks. Every Spring Boot controller, every JPA entity, every Jackson serialization rule, and every JUnit test is driven by annotations. Understanding how annotations work — how to read them, how to create them, and how frameworks process them — is essential for understanding how Spring, Hibernate, and every other Java framework operates under the hood.

---

## Core Concepts

### What Annotations Are

An annotation is a tag that you place on a Java declaration to associate metadata with it. The annotation itself does nothing at runtime unless something reads it and acts on it.

```java
@Override  // This annotation tells the compiler: "I intend to override a superclass method"
public String toString() {
    return "Account{" + id + "}";
}
```

The `@Override` annotation does not change how `toString()` executes. It tells the compiler to verify that a method with this signature actually exists in the superclass. If it does not (e.g., you misspelled the method name), the compiler produces an error. The annotation is a compile-time check, not a runtime behavior.

**Where annotations can be applied:**

```java
@MyAnnotation                          // Class
public class Account {

    @MyAnnotation                      // Field
    private BigDecimal balance;

    @MyAnnotation                      // Constructor
    public Account(@MyAnnotation String id) {  // Parameter
        // ...
    }

    @MyAnnotation                      // Method
    public void deposit(@MyAnnotation BigDecimal amount) {
        // ...
    }

    @MyAnnotation                      // Local variable (Java 8+)
    public void process() {
        @MyAnnotation String temp = "value";
    }
}

@MyAnnotation                          // Package (in package-info.java)
package com.example.fintech;

@MyAnnotation                          // Type use (Java 8+)
List<@NonNull String> names;
```

### Built-in Annotations

Java provides several built-in annotations in `java.lang`. These are the annotations you will use daily.

**@Override:**

Indicates that a method is intended to override a method in a superclass or implement a method from an interface. The compiler verifies this. If the method does not actually override anything, a compilation error is produced.

```java
public class SavingsAccount extends Account {
    @Override  // Compiler checks: does Account have a getMonthlyFee() method?
    public BigDecimal getMonthlyFee() {
        return BigDecimal.ZERO;
    }

    // @Override  // If you uncomment this, compilation error:
    // public BigDecimal getMnthlyFee() {  // Typo — no such method in superclass
    //     return BigDecimal.ZERO;
    // }
}
```

Always use `@Override` when overriding a method. It catches typos and signature mismatches at compile time. If the superclass method is later renamed or removed, the compiler flags the orphaned override.

**@Deprecated:**

Marks a class, method, or field as deprecated — still functional but discouraged for new code. The compiler produces a warning when deprecated elements are used.

```java
public class PaymentGateway {

    /**
     * @deprecated Use {@link #processPayment(PaymentRequest)} instead.
     * This method does not support idempotency and will be removed in v3.0.
     */
    @Deprecated(since = "2.1", forRemoval = true)
    public String charge(String cardNumber, BigDecimal amount) {
        // Legacy implementation
    }

    public PaymentResult processPayment(PaymentRequest request) {
        // Modern implementation with idempotency
    }
}

// Usage produces a compiler warning:
String txId = gateway.charge("4111111111111111", new BigDecimal("100"));
// Warning: [deprecation] charge(String, BigDecimal) in PaymentGateway has been deprecated
```

The `since` and `forRemoval` attributes (Java 9+) provide structured information about when the element was deprecated and whether it will be removed in a future version.

**@SuppressWarnings:**

Tells the compiler to suppress specific warnings for the annotated element. Use sparingly and with the narrowest possible scope.

```java
@SuppressWarnings("unchecked")
public <T> List<T> castList(Object obj) {
    return (List<T>) obj;  // Unchecked cast — we know it is safe in this context
}

@SuppressWarnings({"rawtypes", "unchecked"})
public void legacyInterop() {
    List rawList = new ArrayList();  // Raw type warning suppressed
    rawList.add("string");
}

// Common warning keys:
// "unchecked"     — unchecked type conversions (generics)
// "rawtypes"      — use of raw types
// "deprecation"   — use of deprecated elements
// "serial"        — serializable class without serialVersionUID
// "all"           — suppress all warnings (avoid this)
```

**@FunctionalInterface:**

Indicates that an interface is intended to be a functional interface (exactly one abstract method). The compiler verifies this constraint.

```java
@FunctionalInterface
public interface TransactionFilter {
    boolean test(Transaction transaction);
    // If you add a second abstract method, the compiler produces an error:
    // "Multiple non-overriding abstract methods found in interface TransactionFilter"

    // Default and static methods do NOT count:
    default TransactionFilter and(TransactionFilter other) {
        return tx -> this.test(tx) && other.test(tx);
    }
}
```

**@SafeVarargs:**

Suppresses the "possible heap pollution" warning for varargs methods with generic type parameters. Can only be applied to `static`, `final`, or `private` methods and constructors.

```java
@SafeVarargs
public static <T> List<T> immutableList(T... elements) {
    return List.of(elements);
}
```

### Meta-Annotations

Meta-annotations are annotations that you apply to other annotation declarations. They define how the annotation behaves. Java provides five meta-annotations in `java.lang.annotation`.

**@Target:**

Specifies where the annotation can be applied. Without `@Target`, the annotation can be applied to any declaration.

```java
import java.lang.annotation.ElementType;

@Target({ElementType.METHOD, ElementType.FIELD})
public @interface Audited {
    // Can only be placed on methods and fields
}

// Valid ElementType values:
// ElementType.TYPE           — class, interface, enum, record
// ElementType.FIELD          — field (including enum constants)
// ElementType.METHOD         — method
// ElementType.PARAMETER      — method/constructor parameter
// ElementType.CONSTRUCTOR    — constructor
// ElementType.LOCAL_VARIABLE — local variable
// ElementType.ANNOTATION_TYPE — annotation declaration
// ElementType.PACKAGE        — package (in package-info.java)
// ElementType.TYPE_PARAMETER — generic type parameter (Java 8+)
// ElementType.TYPE_USE       — any type use (Java 8+)
// ElementType.MODULE         — module declaration (Java 9+)
// ElementType.RECORD_COMPONENT — record component (Java 16+)
```

**@Retention:**

Specifies how long the annotation is retained. This is the most important meta-annotation because it determines whether the annotation is available at runtime.

```java
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Audited {
    // Retained at runtime — can be read via reflection
}

// RetentionPolicy values:
// SOURCE   — Discarded by the compiler. Not in the .class file.
//            Used for compile-time checks only (e.g., @Override, @SuppressWarnings).
// CLASS    — Recorded in the .class file but not available at runtime.
//            This is the default if @Retention is not specified.
// RUNTIME  — Recorded in the .class file AND available at runtime via reflection.
//            Required for framework annotations (Spring, JPA, Jackson).
```

**When to use each retention policy:**

| Policy | Use When | Example |
|--------|----------|---------|
| `SOURCE` | The annotation is only for the compiler or source-level tools | `@Override`, `@SuppressWarnings`, Lombok annotations |
| `CLASS` | The annotation is for bytecode-level tools (static analyzers, obfuscators) | Rarely used directly |
| `RUNTIME` | The annotation must be read by the application or framework at runtime | `@RestController`, `@Entity`, `@JsonProperty`, `@Transactional` |

**@Documented:**

Indicates that the annotation should be included in Javadoc. Without `@Documented`, the annotation is invisible in generated documentation.

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Audited {
    // This annotation will appear in the Javadoc of annotated methods
}
```

**@Inherited:**

Indicates that the annotation is inherited by subclasses. Without `@Inherited`, annotations on a superclass are not visible on subclasses.

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Transactional {
    // If @Transactional is on AccountService, it is also present on
    // PremiumAccountService extends AccountService
}

@Transactional
public class AccountService { }

public class PremiumAccountService extends AccountService {
    // Inherits @Transactional from AccountService
}
```

**Important:** `@Inherited` only works for class-level annotations. It does not apply to method, field, or interface annotations.

**@Repeatable (Java 8+):**

Allows the same annotation to be applied multiple times to the same declaration.

```java
@Repeatable(Schedules.class)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Schedule {
    String cron();
    String timezone() default "UTC";
}

// Container annotation (required by @Repeatable)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Schedules {
    Schedule[] value();
}

// Usage — apply the same annotation multiple times
@Schedule(cron = "0 0 9 * * ?", timezone = "America/New_York")
@Schedule(cron = "0 0 18 * * ?", timezone = "Europe/London")
public void generateDailyReport() {
    // Runs at 9 AM ET and 6 PM London time
}
```

### Creating Custom Annotations

Custom annotations are declared with the `@interface` keyword. They can define elements (attributes) with optional default values.

**Basic custom annotation:**

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
@Documented
public @interface Audited {
    String action();                    // Required element (no default)
    String actor() default "system";    // Optional element with default
    boolean includePayload() default false;
}

// Usage
@Audited(action = "PAYMENT_PROCESSED")
public PaymentResult processPayment(PaymentRequest request) {
    // ...
}

@Audited(action = "ACCOUNT_CREATED", actor = "admin", includePayload = true)
public Account createAccount(AccountRequest request) {
    // ...
}
```

**Annotation element types:**

Annotation elements can be of the following types only:

- Primitives (`int`, `long`, `boolean`, `double`, etc.)
- `String`
- `Class` (`Class<?>`, `Class<? extends SomeType>`)
- Enum types
- Other annotation types
- Arrays of any of the above

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface EntityConfig {
    String tableName();                          // String
    int maxRetries() default 3;                  // Primitive
    Class<? extends Exception>[] retryOn() default {SQLException.class};  // Class array
    TransactionIsolation isolation() default TransactionIsolation.READ_COMMITTED;  // Enum
    Audited audit() default @Audited(action = "DEFAULT");  // Nested annotation
    String[] tags() default {};                  // String array
}

// Usage
@EntityConfig(
    tableName = "transactions",
    maxRetries = 5,
    retryOn = {SQLException.class, TimeoutException.class},
    isolation = TransactionIsolation.SERIALIZABLE,
    audit = @Audited(action = "TX_CREATED"),
    tags = {"fintech", "critical"}
)
public class Transaction { }
```

**Single-element annotation shorthand:**

If an annotation has a single element named `value`, you can omit the element name when using it.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface TableName {
    String value();  // Must be named "value" for the shorthand
}

// Usage — shorthand
@TableName("transactions")
public class Transaction { }

// Equivalent to:
@TableName(value = "transactions")
public class Transaction { }
```

**Marker annotation (no elements):**

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Immutable {
    // No elements — this is a marker annotation
    // Its presence alone conveys meaning
}

@Immutable
public record Money(BigDecimal amount, String currency) { }
```

### Processing Annotations at Runtime (Reflection)

Annotations with `RetentionPolicy.RUNTIME` can be read at runtime using Java's Reflection API. This is how Spring, JPA, Jackson, and every other framework discovers and processes annotations.

**Reading class-level annotations:**

```java
@TableName("accounts")
public class Account { }

// Check if an annotation is present
boolean hasTable = Account.class.isAnnotationPresent(TableName.class);  // true

// Get the annotation instance
TableName tableName = Account.class.getAnnotation(TableName.class);
if (tableName != null) {
    System.out.println("Table: " + tableName.value());  // "accounts"
}

// Get all annotations on a class
Annotation[] allAnnotations = Account.class.getAnnotations();
for (Annotation ann : allAnnotations) {
    System.out.println(ann.annotationType().getSimpleName());
}
```

**Reading method-level annotations:**

```java
public class PaymentService {

    @Audited(action = "PAYMENT_PROCESSED", actor = "gateway")
    public PaymentResult processPayment(PaymentRequest request) {
        return PaymentResult.success("TX-001");
    }
}

// Get the method via reflection
Method method = PaymentService.class.getMethod("processPayment", PaymentRequest.class);

// Read the annotation
Audited audited = method.getAnnotation(Audited.class);
if (audited != null) {
    System.out.println("Action: " + audited.action());          // "PAYMENT_PROCESSED"
    System.out.println("Actor: " + audited.actor());            // "gateway"
    System.out.println("Include payload: " + audited.includePayload());  // false
}
```

**Reading field-level annotations:**

```java
public class Transaction {
    @Column(name = "tx_amount", precision = 19, scale = 2)
    private BigDecimal amount;
}

Field field = Transaction.class.getDeclaredField("amount");
Column column = field.getAnnotation(Column.class);
if (column != null) {
    System.out.println("Column name: " + column.name());      // "tx_amount"
    System.out.println("Precision: " + column.precision());    // 19
    System.out.println("Scale: " + column.scale());            // 2
}
```

**Reading parameter-level annotations:**

```java
public class PaymentController {
    public void process(@Valid @RequestBody PaymentRequest request) { }
}

Method method = PaymentController.class.getMethod("process", PaymentRequest.class);
Annotation[][] paramAnnotations = method.getParameterAnnotations();

for (Annotation[] annotations : paramAnnotations) {
    for (Annotation ann : annotations) {
        System.out.println("Parameter annotation: " + ann.annotationType().getSimpleName());
        // "Valid", "RequestBody"
    }
}
```

**A complete annotation-driven audit framework:**

```java
package com.example.fintech.audit;

import java.lang.annotation.*;
import java.lang.reflect.Method;
import java.time.LocalDateTime;

// 1. Define the annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface Audited {
    String action();
    String description() default "";
    AuditLevel level() default AuditLevel.INFO;
}

public enum AuditLevel {
    DEBUG, INFO, WARN, CRITICAL
}

// 2. Define the audit log entry
public record AuditLogEntry(
    String action,
    String description,
    AuditLevel level,
    String methodName,
    LocalDateTime timestamp
) {}

// 3. Build a processor that reads annotations and acts on them
public class AuditProcessor {

    public void processAuditedMethods(Class<?> clazz) {
        for (Method method : clazz.getDeclaredMethods()) {
            Audited audited = method.getAnnotation(Audited.class);
            if (audited != null) {
                AuditLogEntry entry = new AuditLogEntry(
                    audited.action(),
                    audited.description().isEmpty() ? audited.action() : audited.description(),
                    audited.level(),
                    clazz.getSimpleName() + "." + method.getName(),
                    LocalDateTime.now()
                );
                log(entry);
            }
        }
    }

    private void log(AuditLogEntry entry) {
        System.out.printf("[%s] %s | %s | %s | %s%n",
            entry.level(),
            entry.timestamp(),
            entry.action(),
            entry.description(),
            entry.methodName()
        );
    }
}

// 4. Apply annotations to business methods
public class PaymentService {

    @Audited(action = "PAYMENT_INITIATED", level = AuditLevel.INFO)
    public void initiatePayment(PaymentRequest request) {
        // Business logic
    }

    @Audited(
        action = "PAYMENT_COMPLETED",
        description = "Payment successfully processed and settled",
        level = AuditLevel.CRITICAL
    )
    public void completePayment(String transactionId) {
        // Business logic
    }

    @Audited(action = "REFUND_ISSUED", level = AuditLevel.WARN)
    public void issueRefund(String transactionId, BigDecimal amount) {
        // Business logic
    }

    public void internalHelper() {
        // No @Audited — will not be logged
    }
}

// 5. Run the processor
public class Main {
    public static void main(String[] args) {
        AuditProcessor processor = new AuditProcessor();
        processor.processAuditedMethods(PaymentService.class);
        // Output:
        // [INFO] 2024-03-15T10:30:00 | PAYMENT_INITIATED | PAYMENT_INITIATED | PaymentService.initiatePayment
        // [CRITICAL] 2024-03-15T10:30:00 | PAYMENT_COMPLETED | Payment successfully processed and settled | PaymentService.completePayment
        // [WARN] 2024-03-15T10:30:00 | REFUND_ISSUED | REFUND_ISSUED | PaymentService.issueRefund
    }
}
```

### Annotations in Frameworks (Preview)

This is a preview of the annotations you will use extensively from Phase 04 onward. Understanding the annotation mechanics above will help you understand how these frameworks work internally.

**Spring Boot annotations (Phase 05):**

```java
@RestController                          // Marks class as a REST controller
@RequestMapping("/api/v1/accounts")      // Maps URL paths
public class AccountController {

    @GetMapping("/{id}")                 // Maps GET requests
    public Account getAccount(@PathVariable String id) {  // Binds URL parameter
        return accountService.findById(id);
    }

    @PostMapping                         // Maps POST requests
    public Account createAccount(@Valid @RequestBody AccountRequest request) {
        return accountService.create(request);
    }
}

@Service                                 // Marks class as a service bean
public class AccountService {

    @Transactional                       // Wraps method in a database transaction
    public Account create(AccountRequest request) {
        // ...
    }
}

@Configuration                           // Marks class as a configuration source
public class DatabaseConfig {

    @Bean                                // Declares a Spring-managed bean
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}
```

**JPA annotations (Phase 04):**

```java
@Entity                                  // Marks class as a JPA entity (database table)
@Table(name = "accounts")                // Specifies table name
public class Account {

    @Id                                  // Primary key
    @GeneratedValue(strategy = GenerationType.UUID)  // Auto-generated
    private String id;

    @Column(name = "owner_name", nullable = false, length = 100)
    private String ownerName;

    @Column(precision = 19, scale = 2)
    private BigDecimal balance;

    @OneToMany(mappedBy = "account", cascade = CascadeType.ALL)
    private List<Transaction> transactions;

    @Enumerated(EnumType.STRING)         // Store enum as string, not ordinal
    private AccountStatus status;
}
```

**Jackson annotations (Phase 01, 01.08):**

```java
public record PaymentResponse(
    @JsonProperty("transaction_id")      // Custom JSON property name
    String transactionId,

    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss'Z'", timezone = "UTC")
    LocalDateTime timestamp,

    @JsonInclude(JsonInclude.Include.NON_NULL)  // Omit if null
    String errorCode,

    @JsonIgnore                          // Exclude from JSON entirely
    String internalDebugInfo
) {}
```

**JUnit annotations (Phase 01, 01.16):**

```java
class PaymentServiceTest {

    @Test                                // Marks method as a test
    void shouldProcessPaymentSuccessfully() {
        // ...
    }

    @ParameterizedTest                   // Runs test multiple times with different inputs
    @ValueSource(strings = {"USD", "EUR", "GBP"})
    void shouldAcceptValidCurrencies(String currency) {
        // ...
    }

    @BeforeEach                          // Runs before each test method
    void setUp() {
        // ...
    }

    @Disabled("Pending gateway integration")  // Skips the test
    @Test
    void shouldConnectToLiveGateway() {
        // ...
    }
}
```

All of these framework annotations use `RetentionPolicy.RUNTIME`. The framework reads them at startup (or at request time) via reflection and configures its behavior accordingly. When you write `@RestController`, Spring's component scanner finds the class via reflection, reads the annotation, and registers it as an HTTP request handler. When you write `@Entity`, Hibernate reads the annotation and generates the corresponding SQL table definition.

---

## Code Examples

**A complete custom annotation framework for input validation:**

```java
package com.example.fintech.validation;

import java.lang.annotation.*;
import java.lang.reflect.Field;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

// 1. Define validation annotations
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface NotNull {
    String message() default "Field cannot be null";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Positive {
    String message() default "Value must be positive";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface MaxLength {
    int value();
    String message() default "Value exceeds maximum length";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Currency {
    String[] allowed() default {"USD", "EUR", "GBP", "JPY", "CHF"};
    String message() default "Invalid currency code";
}

// 2. Define the validation result
public record ValidationResult(boolean valid, List<String> errors) {
    public static ValidationResult success() {
        return new ValidationResult(true, List.of());
    }

    public static ValidationResult failure(List<String> errors) {
        return new ValidationResult(false, errors);
    }
}

// 3. Build the validator that reads annotations via reflection
public class AnnotationValidator {

    public ValidationResult validate(Object obj) {
        List<String> errors = new ArrayList<>();
        Class<?> clazz = obj.getClass();

        for (Field field : clazz.getDeclaredFields()) {
            field.setAccessible(true);  // Access private fields
            Object value;
            try {
                value = field.get(obj);
            } catch (IllegalAccessException e) {
                errors.add("Cannot access field: " + field.getName());
                continue;
            }

            // Process @NotNull
            if (field.isAnnotationPresent(NotNull.class) && value == null) {
                NotNull ann = field.getAnnotation(NotNull.class);
                errors.add(field.getName() + ": " + ann.message());
            }

            // Process @Positive
            if (field.isAnnotationPresent(Positive.class) && value instanceof BigDecimal bd) {
                if (bd.compareTo(BigDecimal.ZERO) <= 0) {
                    Positive ann = field.getAnnotation(Positive.class);
                    errors.add(field.getName() + ": " + ann.message());
                }
            }

            // Process @MaxLength
            if (field.isAnnotationPresent(MaxLength.class) && value instanceof String s) {
                MaxLength ann = field.getAnnotation(MaxLength.class);
                if (s.length() > ann.value()) {
                    errors.add(field.getName() + ": " + ann.message()
                        + " (max " + ann.value() + ", got " + s.length() + ")");
                }
            }

            // Process @Currency
            if (field.isAnnotationPresent(Currency.class) && value instanceof String s) {
                Currency ann = field.getAnnotation(Currency.class);
                boolean found = false;
                for (String allowed : ann.allowed()) {
                    if (allowed.equals(s)) {
                        found = true;
                        break;
                    }
                }
                if (!found) {
                    errors.add(field.getName() + ": " + ann.message()
                        + " (allowed: " + String.join(", ", ann.allowed()) + ")");
                }
            }
        }

        return errors.isEmpty() ? ValidationResult.success() : ValidationResult.failure(errors);
    }
}

// 4. Apply annotations to a domain object
public class PaymentRequest {
    @NotNull
    @MaxLength(36)
    private String transactionId;

    @NotNull
    @Positive
    private BigDecimal amount;

    @NotNull
    @Currency(allowed = {"USD", "EUR", "GBP"})
    private String currency;

    @MaxLength(255)
    private String description;

    public PaymentRequest(String transactionId, BigDecimal amount,
                          String currency, String description) {
        this.transactionId = transactionId;
        this.amount = amount;
        this.currency = currency;
        this.description = description;
    }
}

// 5. Use the validator
public class Main {
    public static void main(String[] args) {
        AnnotationValidator validator = new AnnotationValidator();

        // Valid request
        PaymentRequest valid = new PaymentRequest(
            "TX-001", new BigDecimal("150.00"), "USD", "Invoice payment"
        );
        ValidationResult result1 = validator.validate(valid);
        System.out.println("Valid: " + result1.valid());  // true

        // Invalid request
        PaymentRequest invalid = new PaymentRequest(
            null, new BigDecimal("-50.00"), "XYZ", "x".repeat(300)
        );
        ValidationResult result2 = validator.validate(invalid);
        System.out.println("Valid: " + result2.valid());  // false
        result2.errors().forEach(System.out::println);
        // transactionId: Field cannot be null
        // amount: Value must be positive
        // currency: Invalid currency code (allowed: USD, EUR, GBP)
        // description: Value exceeds maximum length (max 255, got 300)
    }
}
```

This is a simplified version of what Hibernate Validator (Bean Validation) does internally. The real framework is far more sophisticated, but the core mechanism — read annotations via reflection, evaluate constraints, collect violations — is the same.

---

## Important Notes

- Annotations are metadata, not code. They do not execute logic by themselves. Something must read the annotation and act on it. The compiler reads `@Override`. The JVM reads `@Deprecated`. Spring reads `@RestController`. Hibernate reads `@Entity`. Jackson reads `@JsonProperty`. If nothing reads your custom annotation, it has no effect.
- `RetentionPolicy.RUNTIME` is required for any annotation that a framework must read at runtime. If you create a custom annotation for Spring or JPA and forget to set the retention to `RUNTIME`, the framework will silently ignore it because the annotation is not present in the `.class` file at runtime.
- `RetentionPolicy.SOURCE` is appropriate for annotations that are only meaningful during compilation. `@Override`, `@SuppressWarnings`, and Lombok annotations (`@Data`, `@Builder`) are source-level. Lombok processes these annotations at compile time via the annotation processing API and generates bytecode. The annotations do not exist in the compiled `.class` files.
- The `@Target` meta-annotation should always be specified on custom annotations. Without it, the annotation can be applied to any declaration, which leads to misuse. Restrict the target to the specific element types where the annotation is meaningful.
- `@Inherited` only works for class-level annotations on classes. It does not apply to interfaces, methods, fields, or parameters. If you annotate an interface with `@Inherited @MyAnnotation`, classes implementing that interface do NOT inherit the annotation.
- `@Repeatable` requires a container annotation. The container annotation must have a single element named `value` that is an array of the repeatable annotation type. This is a syntactic requirement, not a design choice.
- Reflection-based annotation processing is slow compared to direct method calls. Frameworks mitigate this by reading annotations once at startup and caching the results. You should follow the same pattern in your own annotation processors — read annotations during initialization, not on every request.
- The `field.setAccessible(true)` call in reflection bypasses Java's access control. This is necessary to read private fields but should be used carefully. In modular applications (JPMS), `setAccessible` may throw `InaccessibleObjectException` if the module does not `opens` the relevant package.
- Annotation elements cannot be `null`. If you need to represent the absence of a value, use a sentinel value (e.g., empty string `""`, `-1` for integers, `Void.class` for class elements) or wrap the element in an `Optional`-like pattern using a separate boolean element.
- Spring's annotation model is built on **meta-annotations**. Annotations like `@RestController` are themselves annotated with `@Controller` and `@ResponseBody`. Spring reads the meta-annotation hierarchy to determine the full set of behaviors. You can create your own composed annotations using this pattern:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@RestController
@RequestMapping("/api/v1")
public @interface V1RestController {
    @AliasFor(annotation = RequestMapping.class, attribute = "value")
    String[] value() default {};
}

// Usage — equivalent to @RestController + @RequestMapping("/api/v1/accounts")
@V1RestController("/accounts")
public class AccountController { }
```

- The annotation processing API (`javax.annotation.processing`) is a compile-time mechanism that allows you to generate source code, configuration files, or other artifacts based on annotations. Lombok, MapStruct, and JPA metamodel generators use this API. This is different from runtime reflection — annotation processors run during `javac`, not during application execution.
- In professional Java development, you will write custom annotations rarely but read framework annotations constantly. The most important skill is understanding what each framework annotation does and how it affects the behavior of your code. When an annotation does not behave as expected, the first debugging step is to verify the retention policy and target, then check whether the framework's component scanning or configuration is set up correctly.

---

## Practice

1. Create a custom `@RateLimited` annotation with elements for `maxRequests` (int, default 100) and `windowSeconds` (int, default 60). Apply it to three methods in a service class. Write a reflection-based processor that reads the annotation from each method and prints the rate limit configuration.

2. Create a `@Retry` annotation with elements for `maxAttempts` (int, default 3), `delayMs` (long, default 1000), and `retryOn` (Class array, default `{Exception.class}`). Write a method that uses reflection to read this annotation and implement a retry loop that respects the configuration.

3. Demonstrate the difference between `RetentionPolicy.SOURCE`, `CLASS`, and `RUNTIME`. Create three annotations, one for each retention policy. Apply all three to a class. Compile the class and use `javap -v` to inspect the bytecode. Verify which annotations appear in the `.class` file. Then write a reflection program and verify which annotations are readable at runtime.

4. Create a `@Repeatable` annotation called `@Role` that specifies a user role required to access a method. Apply multiple `@Role` annotations to a single method. Write a processor that reads all roles and checks whether a given user has at least one of the required roles.

5. Build a simple dependency injection container using annotations. Create `@Inject` (field-level) and `@Component` (class-level) annotations. Write a container class that scans for `@Component` classes, instantiates them, and injects dependencies into `@Inject` fields using reflection.

6. In your Obsidian vault, create a reference table of all Spring Boot annotations you have encountered so far. For each, note the target (class, method, field, parameter), the retention policy, and what the framework does when it reads the annotation.

---

## References

- Java Language Specification — Chapter 9.6 (Annotation Types): https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html#jls-9.6
- java.lang.annotation Package: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/annotation/package-summary.html
- Reflection API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/package-summary.html
- Java Tutorial — Annotations: https://docs.oracle.com/javase/tutorial/java/annotations/
- Spring Framework Annotations: https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html
- Jakarta Bean Validation Specification: https://beanvalidation.org/
- "Effective Java" by Joshua Bloch — Item 39 (Prefer annotations to naming patterns), Item 40 (Consistently use the Override annotation)
