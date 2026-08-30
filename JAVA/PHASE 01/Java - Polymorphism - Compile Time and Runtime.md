---
title: "Java - Polymorphism - Compile Time and Runtime"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - polymorphism
  - overloading
  - overriding
  - dynamic-dispatch
status: "not-started"
---

# Java - Polymorphism - Compile Time and Runtime

> [!abstract] Overview
> Polymorphism means "many forms." It is the OOP principle that allows a single interface to represent different underlying types. In Java, polymorphism manifests in two ways: **compile-time polymorphism** (method overloading, where the compiler selects the correct method based on argument types) and **runtime polymorphism** (method overriding with dynamic dispatch, where the JVM selects the correct method based on the actual object type at runtime). Runtime polymorphism is the mechanism that makes Spring Boot's dependency injection, JPA entity hierarchies, and exception handling work. It is the single most powerful feature of OOP and the one that separates procedural code from truly object-oriented design.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Methods - Parameters Return Types Overloading]]
> - [[Java - Classes and Objects]]
> - [[Java - Inheritance - Single Multilevel Hierarchical]]

---

## Theory

### What is Polymorphism?

The word "polymorphism" comes from Greek: "poly" (many) + "morph" (form). In programming, it means that a single variable, method, or object can take on multiple forms depending on the context.

A real-world analogy: the command "speak" means different things depending on who you say it to. If you say "speak" to a dog, it barks. If you say "speak" to a cat, it meows. If you say "speak" to a human, they talk. The command is the same, but the behavior changes based on the type of the receiver. This is polymorphism.

In Java, polymorphism allows you to write code that works with a parent type but automatically adapts to the specific child type at runtime. This is what makes it possible to write a single method that processes any type of `Notification` (email, SMS, push) without knowing the specific type in advance.

### Compile-Time Polymorphism (Static Binding)

Compile-time polymorphism is resolved by the **compiler** before the program runs. The compiler looks at the method name and the argument types in the method call and selects the matching overloaded method. This is also called **static binding** or **early binding** because the decision is made at compile time and never changes.

The only mechanism for compile-time polymorphism in Java is **method overloading** (and constructor overloading, which you studied in the Constructors note).

```java
public class PaymentProcessor {
    void processPayment(double amount) {
        System.out.println("Processing card payment: " + amount);
    }

    void processPayment(double amount, String couponCode) {
        System.out.println("Processing card payment with coupon: " + amount);
    }

    void processPayment(String bankAccount, double amount) {
        System.out.println("Processing bank transfer: " + amount);
    }
}

// At compile time, the compiler sees the argument types and selects the method:
processor.processPayment(500.0);              // Calls method 1 (double)
processor.processPayment(500.0, "SAVE20");    // Calls method 2 (double, String)
processor.processPayment("ACC-001", 500.0);   // Calls method 3 (String, double)
```

The compiler makes this decision based on the **declared types** of the arguments, not their runtime types. The selection is fixed in the compiled bytecode and does not change regardless of what happens at runtime.

**Characteristics of compile-time polymorphism:**

- Resolved at compile time by the compiler.
- Based on the number, types, and order of parameters.
- Faster at runtime because there is no lookup overhead. The bytecode contains a direct call to the specific method.
- Less flexible because the decision is hardcoded at compile time.

### Runtime Polymorphism (Dynamic Binding)

Runtime polymorphism is resolved by the **JVM** while the program is running. The JVM looks at the **actual type** of the object (not the declared type of the reference variable) and calls the appropriate overridden method. This is also called **dynamic binding** or **late binding** because the decision is deferred until runtime.

The mechanism for runtime polymorphism is **method overriding** combined with **upcasting** (storing a child object in a parent reference).

```java
public class Notification {
    void send() {
        System.out.println("Sending generic notification");
    }
}

public class EmailNotification extends Notification {
    @Override
    void send() {
        System.out.println("Sending email via SMTP");
    }
}

public class SmsNotification extends Notification {
    @Override
    void send() {
        System.out.println("Sending SMS via gateway");
    }
}

// The magic of runtime polymorphism:
Notification n1 = new EmailNotification();  // Parent reference, child object
Notification n2 = new SmsNotification();    // Parent reference, child object

n1.send();  // JVM sees the actual object is EmailNotification. Calls Email's send().
n2.send();  // JVM sees the actual object is SmsNotification. Calls SMS's send().
```

Even though both `n1` and `n2` are declared as `Notification`, the JVM calls different `send()` methods based on the actual object type. The compiler does not know which `send()` will be called. It only knows that `Notification` has a `send()` method. The JVM figures out the rest at runtime.

**Characteristics of runtime polymorphism:**

- Resolved at runtime by the JVM.
- Based on the actual object type, not the reference type.
- Slightly slower than static binding because the JVM must look up the method in the virtual method table (vtable) at runtime. However, the JIT compiler optimizes this heavily, and the overhead is negligible in practice.
- Extremely flexible because new subclasses can be added without modifying existing code. This is the **Open/Closed Principle**: open for extension, closed for modification.

### Upcasting and Downcasting

**Upcasting** is assigning a child object to a parent reference. It is implicit, automatic, and always safe.

```java
Dog dog = new Dog("Rex", 3, "German Shepherd");
Animal animal = dog;  // Upcasting: Dog -> Animal. Automatic and safe.
// A Dog IS an Animal, so this is always valid.
```

After upcasting, you can only call methods that are defined in the parent class. The `Animal` reference does not know about `Dog`-specific methods like `fetch()`. However, if a method is overridden, the JVM still calls the child's version (runtime polymorphism).

```java
animal.eat();       // OK: eat() is defined in Animal
animal.makeSound(); // OK: makeSound() is defined in Animal, but Dog's version runs
// animal.fetch();  // COMPILATION ERROR: fetch() is not defined in Animal
```

**Downcasting** is assigning a parent reference back to a child reference. It requires an explicit cast and can fail at runtime if the object is not actually of the child type.

```java
Animal animal = new Dog("Rex", 3, "German Shepherd");
Dog dog = (Dog) animal;  // Downcasting: Animal -> Dog. Explicit cast required.
dog.fetch();  // OK: dog is actually a Dog object

Animal animal2 = new Cat("Whiskers", 2);
Dog dog2 = (Dog) animal2;  // RUNTIME ERROR: ClassCastException!
// The object is a Cat, not a Dog. The cast fails.
```

### The `instanceof` Operator

The `instanceof` operator checks whether an object is an instance of a specific class or interface before downcasting. This prevents `ClassCastException`.

```java
Animal animal = getSomeAnimal();  // Could return a Dog, Cat, or any Animal

if (animal instanceof Dog) {
    Dog dog = (Dog) animal;  // Safe: we verified the type first
    dog.fetch();
} else if (animal instanceof Cat) {
    Cat cat = (Cat) animal;
    cat.purr();
}
```

**Pattern matching with instanceof (Java 16+):**

Java 16 introduced a cleaner syntax that combines the type check and the cast into a single expression:

```java
if (animal instanceof Dog dog) {
    dog.fetch();  // 'dog' is automatically cast and available in this block
} else if (animal instanceof Cat cat) {
    cat.purr();
}
```

This eliminates the redundant cast and is the recommended style in modern Java.

### How Runtime Polymorphism Works Internally

The JVM implements runtime polymorphism using a **virtual method table** (vtable). Every class that has overridable methods has a vtable, which is an array of method pointers created when the class is loaded.

**Step-by-step mechanism:**

1. When the JVM loads the `Animal` class, it creates a vtable:

```text
Animal vtable:
[0] -> Animal.eat()
[1] -> Animal.makeSound()
[2] -> Animal.toString()
```

2. When the JVM loads the `Dog` class, it creates a new vtable that starts as a copy of `Animal`'s vtable and replaces entries for overridden methods:

```text
Dog vtable:
[0] -> Animal.eat()       (inherited, not overridden)
[1] -> Dog.makeSound()    (overridden! Points to Dog's version)
[2] -> Dog.toString()     (overridden!)
[3] -> Dog.fetch()        (new method, added at the end)
```

3. When you write `animal.makeSound()`, the JVM:
   - Looks at the actual object's class (Dog, not Animal).
   - Finds the Dog vtable.
   - Looks up index [1] in the vtable.
   - Finds `Dog.makeSound()`.
   - Calls it.

This vtable lookup is an O(1) operation (a single array access), which is why runtime polymorphism is fast despite being resolved at runtime. The JIT compiler further optimizes this by inlining frequently called virtual methods when it can determine the actual type.

**Memory layout:**

```text
Dog object in heap:
+------------------+
| Object header    |
|  -> class ptr    |-----> Dog class metadata
|  -> lock word    |            |
+------------------+            v
| Animal fields    |       Dog vtable
|  name: "Rex"     |       [0] -> Animal.eat()
|  age: 3          |       [1] -> Dog.makeSound()  <-- overridden
+------------------+       [2] -> Dog.toString()
| Dog fields       |       [3] -> Dog.fetch()
|  breed: "GSD"    |
+------------------+
```

> [!tip] Key Insight
> Runtime polymorphism is what makes the **Open/Closed Principle** possible. You can add new subclasses (e.g., `PushNotification`, `WhatsAppNotification`) without modifying any existing code that works with the parent type. The existing `sendAllNotifications(List<Notification> notifications)` method will automatically call the correct `send()` method for the new subclass without any changes. This is the foundation of extensible backend architectures and plugin systems.

---

## Syntax and Basic Examples

### Example 1: Compile-time polymorphism (overloading)

```java
import java.util.List;

public class SearchService {

    // Overloaded search methods: same name, different parameters.
    // The compiler selects the correct one based on argument types.

    List<Product> search(String keyword) {
        System.out.println("Searching by keyword: " + keyword);
        return List.of();  // Simplified
    }

    List<Product> search(String category, double maxPrice) {
        System.out.println("Searching category '" + category + "' under " + maxPrice);
        return List.of();
    }

    List<Product> search(String category, double minPrice, double maxPrice) {
        System.out.println("Searching category '" + category + "' between " + minPrice + " and " + maxPrice);
        return List.of();
    }

    List<Product> search(long productId) {
        System.out.println("Searching by product ID: " + productId);
        return List.of();
    }

    public static void main(String[] args) {
        SearchService service = new SearchService();

        // The compiler resolves each call at compile time:
        service.search("laptop");              // String -> method 1
        service.search("electronics", 5000);   // String, double -> method 2
        service.search("electronics", 1000, 5000);  // String, double, double -> method 3
        service.search(1042L);                 // long -> method 4
    }
}
```

### Example 2: Runtime polymorphism (overriding with dynamic dispatch)

```java
public class PaymentMethod {
    String name;

    PaymentMethod(String name) {
        this.name = name;
    }

    void processPayment(double amount) {
        System.out.println("Processing " + amount + " BDT via " + name);
    }

    double getTransactionFee(double amount) {
        return 0;  // Default: no fee
    }
}
```

```java
public class CreditCardPayment extends PaymentMethod {
    String cardNumber;

    CreditCardPayment(String cardNumber) {
        super("Credit Card");
        this.cardNumber = cardNumber;
    }

    @Override
    void processPayment(double amount) {
        System.out.println("Charging " + amount + " BDT to card ending in "
            + cardNumber.substring(cardNumber.length() - 4));
        System.out.println("  Authorization code: AUTH-" + System.currentTimeMillis());
    }

    @Override
    double getTransactionFee(double amount) {
        return amount * 0.025;  // 2.5% fee
    }
}
```

```java
public class BkashPayment extends PaymentMethod {
    String phoneNumber;

    BkashPayment(String phoneNumber) {
        super("bKash");
        this.phoneNumber = phoneNumber;
    }

    @Override
    void processPayment(double amount) {
        System.out.println("Sending " + amount + " BDT to bKash number " + phoneNumber);
        System.out.println("  Transaction ID: BK-" + System.currentTimeMillis());
    }

    @Override
    double getTransactionFee(double amount) {
        return amount * 0.015;  // 1.5% fee
    }
}
```

```java
public class CashOnDelivery extends PaymentMethod {
    CashOnDelivery() {
        super("Cash on Delivery");
    }

    @Override
    void processPayment(double amount) {
        System.out.println("COD: Customer will pay " + amount + " BDT upon delivery");
    }

    // Does NOT override getTransactionFee(). Inherits the default (0 fee).
}
```

```java
public class Main {
    // This method demonstrates runtime polymorphism.
    // It accepts ANY PaymentMethod subclass without knowing the specific type.
    static void checkout(PaymentMethod payment, double orderTotal) {
        double fee = payment.getTransactionFee(orderTotal);
        double total = orderTotal + fee;

        System.out.println("=== Checkout ===");
        System.out.println("Order total: " + orderTotal + " BDT");
        System.out.println("Transaction fee: " + fee + " BDT");
        System.out.println("Grand total: " + total + " BDT");

        // The JVM calls the correct processPayment() based on the actual object type.
        // The compiler only knows that 'payment' is a PaymentMethod.
        // At runtime, the JVM checks the vtable and dispatches to the right method.
        payment.processPayment(total);
        System.out.println();
    }

    public static void main(String[] args) {
        // All three variables are declared as PaymentMethod (parent type).
        // But the actual objects are different subclasses.
        PaymentMethod p1 = new CreditCardPayment("4111111111111234");
        PaymentMethod p2 = new BkashPayment("01712345678");
        PaymentMethod p3 = new CashOnDelivery();

        // Same method call, different behavior at runtime.
        checkout(p1, 5000);  // JVM calls CreditCardPayment.processPayment()
        checkout(p2, 5000);  // JVM calls BkashPayment.processPayment()
        checkout(p3, 5000);  // JVM calls CashOnDelivery.processPayment()
    }
}
```

**Output:**

```text
=== Checkout ===
Order total: 5000.0 BDT
Transaction fee: 125.0 BDT
Grand total: 5125.0 BDT
Charging 5125.0 BDT to card ending in 1234
  Authorization code: AUTH-1720620000000

=== Checkout ===
Order total: 5000.0 BDT
Transaction fee: 75.0 BDT
Grand total: 5075.0 BDT
Sending 5075.0 BDT to bKash number 01712345678
  Transaction ID: BK-1720620000001

=== Checkout ===
Order total: 5000.0 BDT
Transaction fee: 0.0 BDT
Grand total: 5000.0 BDT
COD: Customer will pay 5000.0 BDT upon delivery
```

### Example 3: Upcasting, downcasting, and instanceof

```java
public class Main {
    public static void main(String[] args) {
        // Upcasting: implicit, always safe
        PaymentMethod payment = new BkashPayment("01712345678");

        // We can call PaymentMethod methods:
        payment.processPayment(1000);  // Calls BkashPayment's version (runtime polymorphism)

        // We CANNOT call BkashPayment-specific methods:
        // payment.getPhoneNumber();  // COMPILATION ERROR

        // Downcasting: explicit, can fail
        if (payment instanceof BkashPayment bkash) {
            // Java 16+ pattern matching: 'bkash' is automatically cast
            System.out.println("bKash number: " + bkash.phoneNumber);
        }

        // Unsafe downcast (would throw ClassCastException):
        // CreditCardPayment cc = (CreditCardPayment) payment;  // CRASH!

        // Safe downcast with instanceof check:
        if (payment instanceof CreditCardPayment cc) {
            System.out.println("Card: " + cc.cardNumber);
        } else {
            System.out.println("Not a credit card payment");  // This prints
        }

        // instanceof also checks the inheritance chain:
        System.out.println(payment instanceof PaymentMethod);  // true (same class)
        System.out.println(payment instanceof BkashPayment);   // true (actual type)
        System.out.println(payment instanceof Object);         // true (all objects are Objects)
        System.out.println(payment instanceof CreditCardPayment);  // false (different branch)
    }
}
```

### Example 4: Polymorphism with arrays and collections

```java
public class Main {
    public static void main(String[] args) {
        // An array of the parent type can hold any subclass objects.
        // This is the most common use of polymorphism in backend code.
        Notification[] notifications = {
            new EmailNotification("saad@example.com", "Order Confirmed", "Your order is confirmed."),
            new SmsNotification("+8801712345678", "OTP: 4829"),
            new PushNotification("device_abc", "Your order has shipped!"),
            new EmailNotification("admin@example.com", "Daily Report", "Revenue: 50,000 BDT"),
        };

        // A single loop processes all notification types.
        // The JVM dispatches to the correct send() method for each object.
        // If you add a WhatsAppNotification subclass tomorrow, this loop
        // does not need to change. That is the power of polymorphism.
        int successCount = 0;
        for (Notification n : notifications) {
            try {
                n.send();  // Runtime polymorphism: different behavior for each type
                successCount++;
            } catch (Exception e) {
                System.out.println("Failed to send: " + e.getMessage());
            }
        }

        System.out.println("Sent " + successCount + " of " + notifications.length + " notifications");
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Runtime polymorphism is the engine that drives Spring Boot's architecture. Here are three realistic scenarios.

### Scenario 1: Strategy pattern for discount calculation

The Strategy pattern uses polymorphism to swap algorithms at runtime. Different discount strategies implement the same interface, and the service selects the appropriate one based on business rules.

```java
package com.company.orderservice.discount;

import java.math.BigDecimal;
import java.math.RoundingMode;

// The strategy interface (covered in detail in the Abstraction note).
// All discount strategies implement this interface.
public interface DiscountStrategy {
    BigDecimal calculateDiscount(BigDecimal orderTotal);
    String getStrategyName();
}
```

```java
public class PercentageDiscount implements DiscountStrategy {
    private final double percentage;

    public PercentageDiscount(double percentage) {
        this.percentage = percentage;
    }

    @Override
    public BigDecimal calculateDiscount(BigDecimal orderTotal) {
        return orderTotal.multiply(BigDecimal.valueOf(percentage / 100))
            .setScale(2, RoundingMode.HALF_UP);
    }

    @Override
    public String getStrategyName() {
        return percentage + "% Off";
    }
}
```

```java
public class FixedAmountDiscount implements DiscountStrategy {
    private final BigDecimal discountAmount;

    public FixedAmountDiscount(BigDecimal discountAmount) {
        this.discountAmount = discountAmount;
    }

    @Override
    public BigDecimal calculateDiscount(BigDecimal orderTotal) {
        // Cannot discount more than the order total
        return orderTotal.compareTo(discountAmount) > 0
            ? discountAmount
            : orderTotal;
    }

    @Override
    public String getStrategyName() {
        return discountAmount + " BDT Off";
    }
}
```

```java
public class FreeShippingDiscount implements DiscountStrategy {
    private static final BigDecimal SHIPPING_COST = new BigDecimal("120.00");

    @Override
    public BigDecimal calculateDiscount(BigDecimal orderTotal) {
        return SHIPPING_COST;  // Flat discount equal to shipping cost
    }

    @Override
    public String getStrategyName() {
        return "Free Shipping";
    }
}
```

```java
@Service
public class PricingService {

    // This method uses polymorphism. It does not know or care which
    // DiscountStrategy implementation it receives. It just calls the
    // interface methods, and the JVM dispatches to the correct implementation.
    public BigDecimal calculateFinalPrice(BigDecimal orderTotal, DiscountStrategy strategy) {
        BigDecimal discount = strategy.calculateDiscount(orderTotal);
        BigDecimal finalPrice = orderTotal.subtract(discount);

        // logger.info("Applied {} discount: {} BDT off. Final: {} BDT",
        //     strategy.getStrategyName(), discount, finalPrice);

        return finalPrice.max(BigDecimal.ZERO);  // Price cannot go below zero
    }

    // The strategy is selected based on the coupon code.
    // New coupon types can be added without modifying this method.
    public DiscountStrategy resolveStrategy(String couponCode) {
        if (couponCode == null) return new PercentageDiscount(0);

        return switch (couponCode.toUpperCase()) {
            case "SAVE10" -> new PercentageDiscount(10);
            case "SAVE20" -> new PercentageDiscount(20);
            case "FLAT500" -> new FixedAmountDiscount(new BigDecimal("500"));
            case "FREESHIP" -> new FreeShippingDiscount();
            default -> new PercentageDiscount(0);
        };
    }
}
```

**What to notice:**

- The `calculateFinalPrice()` method accepts a `DiscountStrategy` interface. It does not know whether the actual object is a `PercentageDiscount`, `FixedAmountDiscount`, or `FreeShippingDiscount`. It calls `strategy.calculateDiscount()` and the JVM dispatches to the correct implementation at runtime.
- Adding a new discount type (e.g., `BuyOneGetOneDiscount`) requires creating a new class that implements `DiscountStrategy` and adding one line to the `resolveStrategy()` switch. The `calculateFinalPrice()` method does not change at all. This is the Open/Closed Principle enabled by polymorphism.

### Scenario 2: Spring's HandlerExceptionResolver (framework-level polymorphism)

Spring Boot's exception handling uses polymorphism extensively. The framework defines a `HandlerExceptionResolver` interface, and multiple implementations handle different types of exceptions.

```java
package com.company.orderservice.exception;

import java.util.Map;

// Custom exception hierarchy (from the Inheritance note)
public class AppException extends RuntimeException {
    private final int httpStatus;
    private final String errorCode;

    public AppException(String message, int httpStatus, String errorCode) {
        super(message);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public int getHttpStatus() { return httpStatus; }
    public String getErrorCode() { return errorCode; }
}

public class ValidationException extends AppException {
    private final Map<String, String> fieldErrors;

    public ValidationException(Map<String, String> fieldErrors) {
        super("Validation failed", 400, "VALIDATION_ERROR");
        this.fieldErrors = fieldErrors;
    }

    public Map<String, String> getFieldErrors() { return fieldErrors; }
}
```

```java
// The global exception handler uses polymorphism through method overloading
// (compile-time) and exception hierarchy matching (runtime).
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Spring matches the exception type at runtime using instanceof checks.
    // The most specific handler is selected, similar to how catch blocks work.

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<Map<String, Object>> handleValidation(ValidationException ex) {
        // This handler is selected ONLY for ValidationException objects.
        Map<String, Object> response = new HashMap<>();
        response.put("error", ex.getErrorCode());
        response.put("message", ex.getMessage());
        response.put("fieldErrors", ex.getFieldErrors());
        return ResponseEntity.status(400).body(response);
    }

    @ExceptionHandler(AppException.class)
    public ResponseEntity<Map<String, String>> handleAppException(AppException ex) {
        // This handler catches ALL AppException subclasses that were not
        // caught by a more specific handler above.
        // Polymorphism: the 'ex' variable could be a ResourceNotFoundException,
        // BusinessRuleException, or any other AppException subclass.
        Map<String, String> response = Map.of(
            "error", ex.getErrorCode(),
            "message", ex.getMessage()
        );
        return ResponseEntity.status(ex.getHttpStatus()).body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, String>> handleGeneric(Exception ex) {
        // Catch-all for any exception not handled above.
        // The actual object could be NullPointerException, SQLException, etc.
        // logger.error("Unhandled exception", ex);
        return ResponseEntity.status(500).body(Map.of(
            "error", "INTERNAL_ERROR",
            "message", "An unexpected error occurred"
        ));
    }
}
```

**What to notice:**

- Spring's `@ExceptionHandler` mechanism is polymorphic. When an exception is thrown, Spring walks the exception class hierarchy from most specific to least specific and selects the first matching handler. This is runtime polymorphism applied to exception handling.
- The `AppException` handler receives any subclass of `AppException` that was not caught by a more specific handler. The `ex.getErrorCode()` and `ex.getHttpStatus()` calls dispatch to the correct implementation based on the actual exception type, even though the parameter is declared as `AppException`.

### Scenario 3: JPA entity inheritance with polymorphic queries

JPA supports polymorphic queries, where a query against a parent entity automatically returns all matching subclass entities.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "notification_type")
@Table(name = "notifications")
public abstract class Notification {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String recipient;
    private String message;
    private LocalDateTime sentAt;

    // Abstract method: each subclass must provide its own implementation
    public abstract String getChannel();
}
```

```java
@Entity
@DiscriminatorValue("EMAIL")
public class EmailNotification extends Notification {
    private String subject;

    @Override
    public String getChannel() {
        return "EMAIL";
    }
}
```

```java
@Entity
@DiscriminatorValue("SMS")
public class SmsNotification extends Notification {
    private String phoneNumber;

    @Override
    public String getChannel() {
        return "SMS";
    }
}
```

```java
@Repository
public interface NotificationRepository extends JpaRepository<Notification, Long> {

    // Polymorphic query: returns ALL notification types (Email, SMS, Push)
    // that match the recipient. The JVM creates the correct subclass object
    // for each row based on the discriminator column.
    List<Notification> findByRecipient(String recipient);

    // This also works with specific subclasses:
    List<EmailNotification> findByRecipientAndSubjectContaining(String recipient, String keyword);
}
```

```java
@Service
public class NotificationService {

    public void resendFailedNotifications(String recipient) {
        // This query returns a mix of EmailNotification, SmsNotification, etc.
        List<Notification> notifications = notificationRepository.findByRecipient(recipient);

        for (Notification n : notifications) {
            // Runtime polymorphism: getChannel() returns "EMAIL" or "SMS"
            // depending on the actual object type, even though 'n' is declared
            // as the parent type Notification.
            System.out.println("Resending via " + n.getChannel() + ": " + n.getMessage());
        }
    }
}
```

**What to notice:**

- JPA's `@Inheritance` annotation enables polymorphic persistence. A single database table stores all notification types, and the `notification_type` discriminator column tells JPA which subclass to instantiate when reading rows.
- The `findByRecipient()` query returns a `List<Notification>` that contains a mix of `EmailNotification` and `SmsNotification` objects. The JVM creates the correct subclass based on the discriminator value. This is runtime polymorphism at the database level.
- The `getChannel()` method is abstract in the parent and overridden in each subclass. The for loop calls `n.getChannel()` without knowing the specific type, and the JVM dispatches to the correct implementation.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Confusing the reference type with the object type

**Wrong thinking:** "I declared the variable as `Animal`, so `Animal`'s methods will be called."

**Right thinking:** "The reference type (`Animal`) determines which methods the compiler allows me to call. The object type (`Dog`) determines which implementation the JVM actually executes at runtime."

```java
Animal animal = new Dog("Rex", 3, "German Shepherd");

animal.makeSound();  // Compiler checks: does Animal have makeSound()? Yes.
                     // JVM checks: what is the actual object? Dog.
                     // JVM calls: Dog.makeSound(). Output: "Woof!"
```

**Why it is wrong:** The reference type is a compile-time concept. It restricts which methods you can call (you cannot call `animal.fetch()` because `Animal` does not have `fetch()`). But for methods that exist in both the parent and the child (overridden methods), the JVM always calls the child's version based on the actual object type. This is the essence of runtime polymorphism.

### Mistake 2: Expecting polymorphism to work with static methods

**Wrong:**

```java
public class Animal {
    static void speak() {
        System.out.println("Animal speaks");
    }
}

public class Dog extends Animal {
    static void speak() {
        System.out.println("Dog barks");
    }
}

Animal animal = new Dog();
animal.speak();  // Prints "Animal speaks", NOT "Dog barks"!
```

**Right thinking:** Static methods belong to the class, not the object. They are resolved at compile time based on the reference type, not the object type. This is called **method hiding**, not method overriding. Polymorphism does not apply to static methods.

```java
Animal animal = new Dog();
animal.speak();  // "Animal speaks" (compile-time resolution based on reference type)
Dog.speak();     // "Dog barks" (direct class call)
Animal.speak();  // "Animal speaks" (direct class call)
```

**Why it is wrong:** Static methods are not part of the vtable. They are bound at compile time. If you need polymorphic behavior, use instance methods, not static methods.

### Mistake 3: Expecting polymorphism to work with fields

**Wrong:**

```java
public class Animal {
    String sound = "generic";
}

public class Dog extends Animal {
    String sound = "woof";  // This HIDES the parent's field, it does not override it
}

Animal animal = new Dog();
System.out.println(animal.sound);  // Prints "generic", NOT "woof"!
```

**Right thinking:** Fields are not polymorphic in Java. Field access is resolved at compile time based on the reference type, not the object type. Only methods participate in runtime polymorphism. If you need polymorphic access to data, use getter methods.

```java
public class Animal {
    String getSound() { return "generic"; }
}

public class Dog extends Animal {
    @Override
    String getSound() { return "woof"; }
}

Animal animal = new Dog();
System.out.println(animal.getSound());  // Prints "woof" (polymorphism works with methods)
```

**Why it is wrong:** Fields are resolved statically. The JVM does not use the vtable for field access. This is a deliberate design decision in Java to keep field access fast. Always use getter methods when you need polymorphic behavior.

### Mistake 4: Unsafe downcasting without instanceof check

**Wrong:**

```java
public void processPayment(PaymentMethod payment) {
    // Assumes all payments are credit cards. Crashes for bKash, COD, etc.
    CreditCardPayment cc = (CreditCardPayment) payment;  // ClassCastException!
    System.out.println("Card: " + cc.cardNumber);
}
```

**Right:**

```java
public void processPayment(PaymentMethod payment) {
    if (payment instanceof CreditCardPayment cc) {
        System.out.println("Card: " + cc.cardNumber);
    } else if (payment instanceof BkashPayment bkash) {
        System.out.println("bKash: " + bkash.phoneNumber);
    } else {
        System.out.println("Payment via: " + payment.name);
    }
}

// Even better: use polymorphism to avoid the instanceof chain entirely.
// Add a getPaymentDetails() method to PaymentMethod and override it in each subclass.
```

**Why it is wrong:** Downcasting without a type check is fragile and defeats the purpose of polymorphism. If a new payment method is added, every downcast location must be updated. The polymorphic solution (adding a method to the parent and overriding it in each subclass) is more maintainable and follows the Open/Closed Principle.

---

## Key Takeaways

> [!tip] Remember these points
> 1. **Compile-time polymorphism** (method overloading) is resolved by the compiler based on the declared types of the arguments. The decision is fixed in the bytecode and does not change at runtime.
> 2. **Runtime polymorphism** (method overriding with dynamic dispatch) is resolved by the JVM based on the actual object type at runtime. The reference type determines which methods you can call; the object type determines which implementation runs.
> 3. **Upcasting** (child to parent) is implicit and safe. **Downcasting** (parent to child) requires an explicit cast and should always be guarded by an `instanceof` check to prevent `ClassCastException`.
> 4. Polymorphism applies only to **instance methods**. Static methods and fields are resolved at compile time based on the reference type, not the object type. Use getter methods instead of direct field access when you need polymorphic behavior.
> 5. Runtime polymorphism enables the **Open/Closed Principle**: you can add new subclasses without modifying existing code that works with the parent type. This is the foundation of extensible backend architectures, Spring's plugin model, and the Strategy, Template Method, and Factory design patterns.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Polymorphic Shape Calculator (Easy)

Using the Shape hierarchy from the Inheritance note's Exercise 1 (`Shape`, `Circle`, `Rectangle`, `Square`), write a method `printAllAreas(Shape[] shapes)` that loops through the array and prints each shape's name, type, and area. The method should work with any current or future subclass of `Shape` without modification.

Then add a new subclass `Triangle extends Shape` with fields `base` and `height`. Override `area()` to return `0.5 * base * height`. Add a `Triangle` to the array and verify that `printAllAreas()` handles it correctly without any changes.

> **Hint:** Use `shape.getClass().getSimpleName()` to get the class name at runtime. This demonstrates runtime type information, which is a form of polymorphism.

### Exercise 2: Polymorphic Notification Dispatcher (Medium)

Create a `NotificationDispatcher` class with a method `dispatchAll(List<Notification> notifications)` that sends all notifications and returns a summary report. The summary should include:

- Total notifications sent
- Count per channel (Email, SMS, Push)
- Total characters sent

Use runtime polymorphism to determine the channel. Add a `getChannel()` method to the `Notification` base class and override it in each subclass. The dispatcher should not use `instanceof` checks.

> **Hint:** The `getChannel()` method returns a String like "EMAIL", "SMS", or "PUSH". The dispatcher uses this string to count notifications per channel. This is the polymorphic alternative to `instanceof` chains.

### Exercise 3: Compile-Time vs Runtime Polymorphism Demonstration (Medium)

Write a program that demonstrates the difference between compile-time and runtime polymorphism side by side. Create:

1. A class `Calculator` with overloaded methods: `add(int, int)`, `add(double, double)`, `add(String, String)` (concatenation). Show that the compiler selects the method based on the declared argument types.
2. A class hierarchy `Vehicle` -> `Car`, `Vehicle` -> `Motorcycle` with an overridden method `describe()`. Show that the JVM selects the method based on the actual object type, not the reference type.
3. A tricky case: `Vehicle v = new Car(); v.describe();` and explain which method is called and why.

Print clear labels showing "Compile-time resolution" and "Runtime resolution" for each case.

> **Hint:** For the compile-time demonstration, try passing a variable declared as `Number` to the overloaded `add()` methods and observe which overload the compiler selects.

### Exercise 4: Polymorphic Order Processing Pipeline (Hard, Optional)

Build a simplified order processing pipeline using the Strategy pattern:

1. Create an interface `OrderProcessor` with methods `boolean canProcess(Order order)` and `void process(Order order)`.
2. Create three implementations:
   - `DomesticOrderProcessor`: processes orders with a domestic address. Adds 15% VAT.
   - `InternationalOrderProcessor`: processes orders with an international address. Adds customs duty.
   - `DigitalOrderProcessor`: processes digital orders (no shipping). Instant delivery.
3. Create an `Order` class with fields: `orderId`, `totalAmount`, `shippingAddress`, `isDigital`.
4. Create an `OrderPipeline` class that holds a `List<OrderProcessor>`. Its `processOrder(Order order)` method iterates through the processors, finds the first one where `canProcess()` returns true, and calls `process()`. This is runtime polymorphism: the pipeline does not know which processor will handle the order.
5. In `main()`, create orders of each type and process them through the pipeline.

> **Hint:** The `canProcess()` method encapsulates the selection logic inside each processor. The pipeline just iterates and delegates. Adding a new processor (e.g., `SubscriptionOrderProcessor`) requires only creating a new class and adding it to the list.

<details>
<summary>Solution for Exercise 1</summary>

```java
class Shape {
    String color;
    Shape(String color) { this.color = color; }
    double area() { return 0; }
    @Override
    public String toString() {
        return getClass().getSimpleName() + "{color='" + color + "', area=" + String.format("%.2f", area()) + "}";
    }
}

class Circle extends Shape {
    double radius;
    Circle(String color, double radius) { super(color); this.radius = radius; }
    @Override double area() { return Math.PI * radius * radius; }
}

class Rectangle extends Shape {
    double width, height;
    Rectangle(String color, double w, double h) { super(color); width = w; height = h; }
    @Override double area() { return width * height; }
}

class Square extends Rectangle {
    Square(String color, double side) { super(color, side, side); }
}

class Triangle extends Shape {
    double base, height;
    Triangle(String color, double base, double height) {
        super(color); this.base = base; this.height = height;
    }
    @Override double area() { return 0.5 * base * height; }
}

public class Main {
    static void printAllAreas(Shape[] shapes) {
        for (Shape s : shapes) {
            System.out.printf("%-10s | Color: %-6s | Area: %8.2f%n",
                s.getClass().getSimpleName(), s.color, s.area());
        }
    }

    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle("Red", 5),
            new Rectangle("Blue", 4, 6),
            new Square("Green", 3),
            new Triangle("Yellow", 8, 5)  // New subclass, no changes to printAllAreas()
        };
        printAllAreas(shapes);
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;

abstract class Notification {
    String recipient;
    String message;
    Notification(String recipient, String message) {
        this.recipient = recipient; this.message = message;
    }
    abstract String getChannel();
    void send() { System.out.println("[" + getChannel() + "] To: " + recipient + " | " + message); }
}

class EmailNotification extends Notification {
    EmailNotification(String r, String m) { super(r, m); }
    @Override String getChannel() { return "EMAIL"; }
}

class SmsNotification extends Notification {
    SmsNotification(String r, String m) { super(r, m); }
    @Override String getChannel() { return "SMS"; }
}

class PushNotification extends Notification {
    PushNotification(String r, String m) { super(r, m); }
    @Override String getChannel() { return "PUSH"; }
}

class NotificationDispatcher {
    static Map<String, Object> dispatchAll(List<Notification> notifications) {
        Map<String, Integer> channelCounts = new HashMap<>();
        int totalChars = 0;

        for (Notification n : notifications) {
            n.send();
            String channel = n.getChannel();  // Polymorphic call
            channelCounts.merge(channel, 1, Integer::sum);
            totalChars += n.message.length();
        }

        Map<String, Object> report = new HashMap<>();
        report.put("totalSent", notifications.size());
        report.put("channelBreakdown", channelCounts);
        report.put("totalCharacters", totalChars);
        return report;
    }
}

public class Main {
    public static void main(String[] args) {
        List<Notification> notifications = List.of(
            new EmailNotification("saad@test.com", "Order confirmed"),
            new SmsNotification("+8801712345678", "OTP: 1234"),
            new PushNotification("device_1", "Order shipped"),
            new EmailNotification("admin@test.com", "Daily report")
        );

        Map<String, Object> report = NotificationDispatcher.dispatchAll(notifications);
        System.out.println("\nReport: " + report);
    }
}
```

</details>

---

## Related Notes

- [[Java - Inheritance - Single Multilevel Hierarchical]]
- [[Java - Abstraction - Abstract Classes and Interfaces]] (next note)
- [[Java - Static Keyword - Variables Methods Blocks]]

---

## Resources

- [Oracle Java Tutorials: Polymorphism](https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html) - Official documentation explaining runtime polymorphism with the bicycle example.
- [Oracle Java Tutorials: Overriding and Hiding Methods](https://docs.oracle.com/javase/tutorial/java/IandI/override.html) - Detailed comparison of overriding (instance methods, runtime) vs hiding (static methods, compile time).
- [Baeldung: Java Polymorphism](https://www.baeldung.com/java-polymorphism) - Comprehensive guide covering both compile-time and runtime polymorphism with examples.
- [Baeldung: Java instanceof Operator](https://www.baeldung.com/java-instanceof) - Guide to instanceof including Java 16+ pattern matching syntax.
- [Head First Design Patterns by Eric Freeman - Chapter 1: Strategy Pattern](https://www.oreilly.com/library/view/head-first-design/9781098118907/) - The best introduction to the Strategy pattern, which is the most common application of polymorphism in backend development.
