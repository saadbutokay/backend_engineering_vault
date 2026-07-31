**Phase:** Level 2 - OOP
**Date Studied:**

---
## What Problem Does This Solve?

```text
In Level 1.10, you built a BankAccount class.
But there was a dangerous problem:

  BankAccount account = new BankAccount("Rahim", 5000.0);
  account.balance = -999999; // anyone can set any value!
  account.isActive = false;  // bypass the freeze logic!
  account.accountNumber = "HACKED"; // change supposedly fixed ID!

Without protection:
  → Any code anywhere can set balance to negative
  → Business rules (minimum balance, freeze logic) can be bypassed
  → Internal implementation details are exposed
  → You can't change how the class works without breaking all callers
  → Data is NEVER in a guaranteed valid state

This is like building a bank vault with no door.
The money (data) is completely unprotected.

Encapsulation solves this:
  → Fields are PRIVATE — only the class itself can access them
  → Controlled access through PUBLIC methods (getters/setters)
  → Setters VALIDATE before allowing changes
  → Business rules are ENFORCED — impossible to bypass
  → Internal implementation can CHANGE without breaking callers
  → Data is ALWAYS in a valid, known state

After encapsulation:
  account.balance = -999999;  // COMPILE ERROR — field is private!
  account.setBalance(-999999); // no such method — controlled access
  account.deposit(-999999);    // method validates → rejects invalid input
```

Encapsulation is not optional in professional code.
Every Spring Boot entity, service, DTO you write will use it.

---

## 1. What Is Encapsulation?

```text
Encapsulation = bundling DATA and the METHODS that operate
on that data into one unit, and RESTRICTING direct access
to the data from outside that unit.

Two aspects:
  1. DATA HIDING: make fields private
  2. CONTROLLED ACCESS: provide public methods as the interface

The analogy: your phone's battery.
  The battery (data) is inside your phone — you can't touch it directly.
  You interact with it through the charging port and power button (methods).
  This prevents you from accidentally damaging the battery.
  The manufacturer can change battery design without changing your charging behavior.

In code:
  Private fields = battery inside the phone
  Public methods = charging port and power button
  Class = the phone encasing everything

WHAT YOU CONTROL:
  ✅ WHO can access data (access modifiers)
  ✅ HOW data can be read (getters — may transform)
  ✅ HOW data can be modified (setters — must validate)
  ✅ WHAT internal state is hidden from outside
  ✅ WHEN state can change (enforced through methods)
```

---

## 2. Access Modifiers - The Four Levels

```java
package com.example.demo;

public class AccessModifierDemo {
    
    // ─────────────────────────────────────────
    // PRIVATE — Most restrictive
    // Visible ONLY within this class
    // ─────────────────────────────────────────
    
    private String secretKey;        // only this class
    private int failedAttempts;      // only this class
    
    // ─────────────────────────────────────────
    // DEFAULT (package-private) — No modifier
    // Visible within the SAME PACKAGE only
    // ─────────────────────────────────────────
    
    String packageField;             // same package only
    void packageMethod() { }         // same package only
    
    // ─────────────────────────────────────────
    // PROTECTED — Package + subclasses
    // Visible in same package AND subclasses (even different packages)
    // ─────────────────────────────────────────
    
    protected String protectedField; // same package + subclasses
    protected void protectedMethod() { }
    
    // ─────────────────────────────────────────
    // PUBLIC — Least restrictive
    // Visible EVERYWHERE
    // ─────────────────────────────────────────
    
    public String publicField;       // everywhere (avoid public fields!)
    public void publicMethod() { }   // everywhere
    
    // ─────────────────────────────────────────
    // VISIBILITY TABLE:
    // ─────────────────────────────────────────
    //
    // Modifier      │ Class │ Package │ Subclass │ World
    // ──────────────┼───────┼─────────┼──────────┼──────
    // private       │  ✅   │   ❌    │    ❌    │  ❌
    // (default)     │  ✅   │   ✅    │    ❌    │  ❌
    // protected     │  ✅   │   ✅    │    ✅    │  ❌
    // public        │  ✅   │   ✅    │    ✅    │  ✅
    //
    // ─────────────────────────────────────────
    // PROFESSIONAL RULE:
    // Fields: ALWAYS private (with rare exceptions)
    // Methods: public (API), private (internal helpers), protected (inheritance)
    // ─────────────────────────────────────────
}
```

### Access Modifiers in Practice

```java
// File: Order.java (in package com.example.shop)
package com.example.shop;

public class Order {
    
    // Private fields — protected from direct manipulation:
    private final String orderId;    // immutable after creation
    private String status;           // controlled change via methods
    private double totalAmount;      // validated on change
    private String customerId;       // set once, never changes
    private java.time.LocalDateTime createdAt;
    
    // Package-private — used by other classes in same package (e.g., OrderRepository)
    String internalTrackingCode;
    
    // Protected — subclasses (like PriorityOrder) can access:
    protected String warehouse;
    
    public Order(String orderId, String customerId, double totalAmount) {
        this.orderId    = orderId;
        this.customerId = customerId;
        this.totalAmount = totalAmount;
        this.status     = "PENDING";
        this.createdAt  = java.time.LocalDateTime.now();
    }
    
    // Public API — what other classes interact with:
    public String getOrderId()     { return orderId; }
    public String getStatus()      { return status; }
    public double getTotalAmount() { return totalAmount; }
    public String getCustomerId()  { return customerId; }
    
    // Private helper — internal logic only:
    private boolean isValidStatusTransition(String newStatus) {
        return switch (status) {
            case "PENDING"    -> newStatus.equals("CONFIRMED") || newStatus.equals("CANCELLED");
            case "CONFIRMED"  -> newStatus.equals("SHIPPED") || newStatus.equals("CANCELLED");
            case "SHIPPED"    -> newStatus.equals("DELIVERED");
            case "DELIVERED",
                 "CANCELLED"  -> false; // terminal states
            default           -> false;
        };
    }
    
    // Public method that enforces business rules:
    public boolean updateStatus(String newStatus) {
        if (!isValidStatusTransition(newStatus)) { // calls private method
            System.out.println("❌ Invalid transition: " + status + " → " + newStatus);
            return false;
        }
        String old = this.status;
        this.status = newStatus;
        System.out.println("✅ Order " + orderId + ": " + old + " → " + newStatus);
        return true;
    }
    
    @Override
    public String toString() {
        return String.format("Order{id='%s', status='%s', amount=৳%.2f}",
                             orderId, status, totalAmount);
    }
}

// In a DIFFERENT class, trying to access private fields:
class OrderTest {
    static void test() {
        Order order = new Order("ORD-001", "CUST-123", 1500.0);
        
        // order.status = "DELIVERED"; // COMPILE ERROR: private
        // order.totalAmount = -999;   // COMPILE ERROR: private
        // order.orderId = "HACKED";   // COMPILE ERROR: final + private
        
        // Only through methods:
        System.out.println(order.getStatus()); // OK
        order.updateStatus("CONFIRMED");        // OK — validates first
        order.updateStatus("PENDING");          // rejected — invalid transition
    }
}
```

---

## 3. Getters and Setters - The Complete Guide

```java
public class GettersAndSetters {
    
    // Getters: read-only access to a field
    // Setters: write access with validation
    
    // ─────────────────────────────────────────
    // BASIC GETTER/SETTER PATTERN
    // ─────────────────────────────────────────
    
    private String firstName;
    private String lastName;
    private int age;
    private String email;
    private double salary;
    
    // GETTER naming: get + FieldName (in CamelCase)
    public String getFirstName() { return firstName; }
    public String getLastName()  { return lastName; }
    public int getAge()          { return age; }
    public String getEmail()     { return email; }
    public double getSalary()    { return salary; }
    
    // Boolean getter: is + FieldName (convention for booleans)
    private boolean isActive;
    private boolean hasSubscription;
    
    public boolean isActive()          { return isActive; }
    public boolean hasSubscription()   { return hasSubscription; }
    // NOTE: getIsActive() also compiles but isActive() is the standard
    
    // SETTER naming: set + FieldName
    // ALWAYS validate in setters!
    
    public void setFirstName(String firstName) {
        if (firstName == null || firstName.isBlank()) {
            throw new IllegalArgumentException("First name cannot be blank");
        }
        this.firstName = firstName.trim();
    }
    
    public void setLastName(String lastName) {
        if (lastName == null || lastName.isBlank()) {
            throw new IllegalArgumentException("Last name cannot be blank");
        }
        this.lastName = lastName.trim();
    }
    
    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Age must be between 0 and 150: " + age);
        }
        this.age = age;
    }
    
    public void setEmail(String email) {
        if (email == null || !email.contains("@") || !email.contains(".")) {
            throw new IllegalArgumentException("Invalid email: " + email);
        }
        this.email = email.trim().toLowerCase();
    }
    
    public void setSalary(double salary) {
        if (salary < 0) {
            throw new IllegalArgumentException("Salary cannot be negative: " + salary);
        }
        this.salary = salary;
    }
    
    public void setActive(boolean active) {
        this.isActive = active;
    }
    
    // ─────────────────────────────────────────
    // COMPUTED GETTERS — Return derived values
    // (no corresponding field — calculated on the fly)
    // ─────────────────────────────────────────
    
    // No 'fullName' field — computed from firstName + lastName:
    public String getFullName() {
        return firstName + " " + lastName;
    }
    
    // No 'annualSalary' field:
    public double getAnnualSalary() {
        return salary * 12;
    }
    
    // No 'isAdult' field:
    public boolean isAdult() {
        return age >= 18;
    }
    
    // ─────────────────────────────────────────
    // GETTERS THAT TRANSFORM / PROTECT DATA
    // ─────────────────────────────────────────
    
    private String passwordHash;
    private String creditCardNumber;
    private java.util.List<String> roles;
    
    // NEVER return the real password — only a masked version:
    public String getMaskedPassword() {
        return passwordHash != null ? "****" : null;
    }
    
    // Return only last 4 digits of credit card:
    public String getMaskedCardNumber() {
        if (creditCardNumber == null || creditCardNumber.length() < 4) return null;
        return "****-****-****-" + creditCardNumber.substring(creditCardNumber.length() - 4);
    }
    
    // Return COPY of mutable collection — defensive copy:
    public java.util.List<String> getRoles() {
        if (roles == null) return java.util.Collections.emptyList();
        return java.util.Collections.unmodifiableList(roles); // or: new ArrayList<>(roles)
    }
    // If you returned 'roles' directly, callers could modify your internal list!
    
    // ─────────────────────────────────────────
    // WHEN NOT TO HAVE A SETTER (read-only fields)
    // ─────────────────────────────────────────
    
    private final String userId;     // set in constructor, NEVER changes — no setter
    private java.time.LocalDateTime createdAt; // set once at creation — no setter
    
    // These have GETTERS but NO SETTERS:
    public String getUserId()                      { return userId; }
    public java.time.LocalDateTime getCreatedAt()  { return createdAt; }
    
    public GettersAndSetters(String userId) {
        this.userId    = userId;
        this.createdAt = java.time.LocalDateTime.now();
        this.roles     = new java.util.ArrayList<>();
    }
    
    public static void main(String[] args) {
        GettersAndSetters user = new GettersAndSetters("USER-001");
        
        // Using setters with validation:
        user.setFirstName("Rahim");
        user.setLastName("Ahmed");
        user.setAge(21);
        user.setEmail("rahim@example.com");
        user.setSalary(60000);
        
        // Using getters:
        System.out.println(user.getFullName());    // "Rahim Ahmed"
        System.out.println(user.getEmail());       // "rahim@example.com"
        System.out.println(user.getAnnualSalary()); // 720000.0
        System.out.println(user.isAdult());        // true
        
        // Validation in action:
        try {
            user.setAge(-5); // throws IllegalArgumentException
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        
        try {
            user.setEmail("notanemail"); // throws
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        
        // Read-only field — no setter:
        System.out.println("User ID: " + user.getUserId());
        // user.userId = "HACKED"; // COMPILE ERROR: private + final
    }
}
```

---

## 4. Encapsulation in Real Spring Boot Code

```java
// This is what your Spring Boot entities, DTOs, and services look like
// Understanding encapsulation makes Spring Boot patterns obvious

// ─────────────────────────────────────────
// JPA ENTITY (database table mapping)
// In Spring Boot, JPA uses reflection to access private fields
// via getters/setters — so you MUST follow the pattern
// ─────────────────────────────────────────

// @Entity  ← Spring annotation (comes in Level 6)
// @Table(name = "users")
public class UserEntity {
    
    // @Id @GeneratedValue
    private Long id;               // auto-generated by DB
    
    // @Column(nullable = false, unique = true)
    private String email;
    
    // @Column(nullable = false)
    private String passwordHash;   // never store plain password!
    
    private String firstName;
    private String lastName;
    
    // @Enumerated(EnumType.STRING)
    private String role;           // "ADMIN", "USER", "MODERATOR"
    
    private boolean isActive;
    
    // @CreationTimestamp
    private java.time.LocalDateTime createdAt;
    
    // @UpdateTimestamp
    private java.time.LocalDateTime updatedAt;
    
    // JPA requires no-arg constructor:
    protected UserEntity() { }
    
    // Your constructor for creating new users:
    public UserEntity(String email, String passwordHash,
                      String firstName, String lastName) {
        this.email        = email.toLowerCase().trim();
        this.passwordHash = passwordHash;
        this.firstName    = firstName.trim();
        this.lastName     = lastName.trim();
        this.role         = "USER";
        this.isActive     = true;
        this.createdAt    = java.time.LocalDateTime.now();
    }
    
    // Getters — all public:
    public Long getId()              { return id; }
    public String getEmail()         { return email; }
    public String getPasswordHash()  { return passwordHash; }
    public String getFirstName()     { return firstName; }
    public String getLastName()      { return lastName; }
    public String getRole()          { return role; }
    public boolean isActive()        { return isActive; }
    public java.time.LocalDateTime getCreatedAt() { return createdAt; }
    
    // Computed:
    public String getFullName() { return firstName + " " + lastName; }
    
    // Setters only for mutable fields — with validation:
    public void setEmail(String email) {
        if (email == null || !email.contains("@"))
            throw new IllegalArgumentException("Invalid email");
        this.email = email.toLowerCase().trim();
        this.updatedAt = java.time.LocalDateTime.now();
    }
    
    public void setFirstName(String firstName) {
        if (firstName == null || firstName.isBlank())
            throw new IllegalArgumentException("First name required");
        this.firstName = firstName.trim();
        this.updatedAt = java.time.LocalDateTime.now();
    }
    
    public void setLastName(String lastName) {
        if (lastName == null || lastName.isBlank())
            throw new IllegalArgumentException("Last name required");
        this.lastName = lastName.trim();
        this.updatedAt = java.time.LocalDateTime.now();
    }
    
    // No setId() — ID is auto-generated, never manually set
    // No setCreatedAt() — set at creation, never changes
    // No setPasswordHash() — change password through a dedicated method:
    
    public void changePassword(String newPasswordHash) {
        if (newPasswordHash == null || newPasswordHash.length() < 20) {
            throw new IllegalArgumentException("Invalid password hash");
        }
        this.passwordHash = newPasswordHash;
        this.updatedAt = java.time.LocalDateTime.now();
    }
    
    public void deactivate() {
        this.isActive = false;
        this.updatedAt = java.time.LocalDateTime.now();
    }
    
    public void promoteToAdmin() {
        this.role = "ADMIN";
        this.updatedAt = java.time.LocalDateTime.now();
    }
    
    @Override
    public String toString() {
        return String.format("User{id=%d, email='%s', name='%s', role='%s', active=%b}",
                             id, email, getFullName(), role, isActive);
    }
}

// ─────────────────────────────────────────
// DTO (Data Transfer Object)
// What you send/receive in API requests/responses
// Different from Entity — no DB concerns, just data transfer
// ─────────────────────────────────────────

class CreateUserRequest {
    // Input DTO — what comes FROM the client
    // In Spring Boot, Jackson deserializes JSON to this
    
    private String firstName;
    private String lastName;
    private String email;
    private String password;
    
    // Spring Boot's Jackson needs a no-arg constructor:
    public CreateUserRequest() { }
    
    public CreateUserRequest(String firstName, String lastName,
                              String email, String password) {
        this.firstName = firstName;
        this.lastName  = lastName;
        this.email     = email;
        this.password  = password;
    }
    
    // Getters — Jackson reads these to build JSON:
    public String getFirstName() { return firstName; }
    public String getLastName()  { return lastName; }
    public String getEmail()     { return email; }
    public String getPassword()  { return password; }
    
    // Setters — Jackson writes these when deserializing JSON:
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public void setLastName(String lastName)   { this.lastName = lastName; }
    public void setEmail(String email)         { this.email = email; }
    public void setPassword(String password)   { this.password = password; }
}

class UserResponse {
    // Output DTO — what you SEND to the client
    // NEVER includes password hash or sensitive data
    
    private Long id;
    private String fullName;
    private String email;
    private String role;
    private boolean active;
    
    public UserResponse() { }
    
    // Built from an entity — controls what the client sees:
    public UserResponse(UserEntity user) {
        this.id       = user.getId();
        this.fullName = user.getFullName(); // computed
        this.email    = user.getEmail();
        this.role     = user.getRole();
        this.active   = user.isActive();
        // passwordHash deliberately NOT included!
    }
    
    public Long getId()       { return id; }
    public String getFullName(){ return fullName; }
    public String getEmail()  { return email; }
    public String getRole()   { return role; }
    public boolean isActive() { return active; }
    
    // Setters needed for Jackson:
    public void setId(Long id)             { this.id = id; }
    public void setFullName(String name)   { this.fullName = name; }
    public void setEmail(String email)     { this.email = email; }
    public void setRole(String role)       { this.role = role; }
    public void setActive(boolean active)  { this.active = active; }
}
```

---

## 5. Immutable Classes - Ultimate Encapsulation

```java
// An immutable class: once created, its state NEVER changes.
// All fields are final and private.
// No setters.
// Safe to share across threads.
// Java's String, Integer, LocalDate — all immutable.

public final class Money {
    // final class — cannot be subclassed (prevents subclass from making it mutable)
    
    private final long amountInPoisha; // final — set once, never changed
    private final String currency;     // final
    
    public Money(long amountInPoisha, String currency) {
        if (amountInPoisha < 0)
            throw new IllegalArgumentException("Amount cannot be negative");
        if (currency == null || currency.isBlank())
            throw new IllegalArgumentException("Currency required");
        
        this.amountInPoisha = amountInPoisha;
        this.currency = currency.toUpperCase();
    }
    
    // Convenience constructor:
    public static Money of(double amount, String currency) {
        return new Money((long)(amount * 100), currency); // static factory method
    }
    
    // Getters only — no setters:
    public long getAmountInPoisha() { return amountInPoisha; }
    public String getCurrency()     { return currency; }
    
    // Computed getter:
    public double getAmount() { return amountInPoisha / 100.0; }
    
    // Operations return NEW objects — never modify 'this':
    public Money add(Money other) {
        if (!this.currency.equals(other.currency))
            throw new IllegalArgumentException("Cannot add different currencies");
        return new Money(this.amountInPoisha + other.amountInPoisha, this.currency);
    }
    
    public Money subtract(Money other) {
        if (!this.currency.equals(other.currency))
            throw new IllegalArgumentException("Cannot subtract different currencies");
        if (this.amountInPoisha < other.amountInPoisha)
            throw new IllegalArgumentException("Insufficient amount");
        return new Money(this.amountInPoisha - other.amountInPoisha, this.currency);
    }
    
    public Money multiply(int factor) {
        if (factor < 0) throw new IllegalArgumentException("Factor cannot be negative");
        return new Money(this.amountInPoisha * factor, this.currency);
    }
    
    public Money applyDiscount(double discountRate) {
        if (discountRate < 0 || discountRate > 1)
            throw new IllegalArgumentException("Discount must be 0.0 to 1.0");
        long discounted = (long)(amountInPoisha * (1 - discountRate));
        return new Money(discounted, currency);
    }
    
    public boolean isGreaterThan(Money other) {
        return this.amountInPoisha > other.amountInPoisha;
    }
    
    public boolean isZero() { return amountInPoisha == 0; }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Money)) return false;
        Money other = (Money) obj;
        return this.amountInPoisha == other.amountInPoisha
            && this.currency.equals(other.currency);
    }
    
    @Override
    public int hashCode() {
        return java.util.Objects.hash(amountInPoisha, currency);
    }
    
    @Override
    public String toString() {
        return String.format("%s %.2f", currency, getAmount());
    }
    
    public static void main(String[] args) {
        Money price  = Money.of(1500.0, "BDT");
        Money tax    = Money.of(225.0, "BDT");
        Money total  = price.add(tax); // returns NEW Money — price unchanged!
        Money discounted = price.applyDiscount(0.10); // 10% off
        
        System.out.println("Price    : " + price);      // BDT 1500.00
        System.out.println("Tax      : " + tax);        // BDT 225.00
        System.out.println("Total    : " + total);      // BDT 1725.00
        System.out.println("Discounted: " + discounted); // BDT 1350.00
        System.out.println("Price unchanged: " + price); // BDT 1500.00 — immutable!
        
        // Perfect for financial systems:
        // Threads can share Money objects safely
        // No accidental mutation bugs
        // Easy to reason about correctness
    }
}
```

---

## 6. The Builder Pattern - When Constructors Get Too Long

```java
// When a class has many optional fields,
// constructors become unwieldy:
// new User("Rahim", "Ahmed", null, null, null, 21, "Dhaka", true, false, null)
// What does each null mean? Which fields are optional?

// The Builder Pattern solves this elegantly.
// Spring Boot uses this everywhere (HttpEntity.Builder, etc.)

public class UserProfile {
    
    // All fields private — set only through Builder
    private final String username;    // required
    private final String email;       // required
    private final String firstName;   // required
    private final String lastName;    // required
    private final Integer age;        // optional
    private final String bio;         // optional
    private final String city;        // optional
    private final String website;     // optional
    private final boolean isPublic;   // optional, default true
    private final boolean emailNotifications; // optional, default true
    
    // Private constructor — ONLY Builder can call it
    private UserProfile(Builder builder) {
        this.username           = builder.username;
        this.email              = builder.email;
        this.firstName          = builder.firstName;
        this.lastName           = builder.lastName;
        this.age                = builder.age;
        this.bio                = builder.bio;
        this.city               = builder.city;
        this.website            = builder.website;
        this.isPublic           = builder.isPublic;
        this.emailNotifications = builder.emailNotifications;
    }
    
    // Getters only — immutable:
    public String getUsername()           { return username; }
    public String getEmail()              { return email; }
    public String getFirstName()          { return firstName; }
    public String getLastName()           { return lastName; }
    public Integer getAge()               { return age; }
    public String getBio()                { return bio; }
    public String getCity()               { return city; }
    public String getWebsite()            { return website; }
    public boolean isPublic()             { return isPublic; }
    public boolean isEmailNotifications() { return emailNotifications; }
    
    public String getFullName() { return firstName + " " + lastName; }
    
    @Override
    public String toString() {
        return String.format(
            "UserProfile{username='%s', email='%s', name='%s', " +
            "age=%s, city='%s', public=%b}",
            username, email, getFullName(), age, city, isPublic);
    }
    
    // ─────────────────────────────────────────
    // STATIC INNER BUILDER CLASS
    // ─────────────────────────────────────────
    
    public static class Builder {
        
        // Required fields:
        private final String username;
        private final String email;
        private final String firstName;
        private final String lastName;
        
        // Optional fields with defaults:
        private Integer age                  = null;
        private String bio                   = null;
        private String city                  = null;
        private String website               = null;
        private boolean isPublic             = true;
        private boolean emailNotifications   = true;
        
        // Required fields in Builder constructor:
        public Builder(String username, String email,
                       String firstName, String lastName) {
            if (username == null || username.isBlank())
                throw new IllegalArgumentException("Username required");
            if (email == null || !email.contains("@"))
                throw new IllegalArgumentException("Valid email required");
            if (firstName == null || firstName.isBlank())
                throw new IllegalArgumentException("First name required");
            if (lastName == null || lastName.isBlank())
                throw new IllegalArgumentException("Last name required");
            
            this.username  = username.toLowerCase().trim();
            this.email     = email.toLowerCase().trim();
            this.firstName = firstName.trim();
            this.lastName  = lastName.trim();
        }
        
        // Each optional setter returns 'this' for chaining:
        public Builder age(int age) {
            if (age < 0 || age > 120)
                throw new IllegalArgumentException("Invalid age: " + age);
            this.age = age;
            return this;
        }
        
        public Builder bio(String bio) {
            if (bio != null && bio.length() > 500)
                throw new IllegalArgumentException("Bio max 500 characters");
            this.bio = bio;
            return this;
        }
        
        public Builder city(String city) {
            this.city = city != null ? city.trim() : null;
            return this;
        }
        
        public Builder website(String website) {
            this.website = website;
            return this;
        }
        
        public Builder makePrivate() {
            this.isPublic = false;
            return this;
        }
        
        public Builder disableEmailNotifications() {
            this.emailNotifications = false;
            return this;
        }
        
        // Build the final object:
        public UserProfile build() {
            return new UserProfile(this);
        }
    }
    
    public static void main(String[] args) {
        
        // Minimal profile (only required fields):
        UserProfile minimal = new Builder("rahim123", "rahim@test.com",
                                          "Rahim", "Ahmed")
            .build();
        System.out.println(minimal);
        
        // Full profile:
        UserProfile full = new Builder("karim.dev", "karim@example.com",
                                        "Karim", "Hassan")
            .age(25)
            .bio("Backend engineer | Java | Spring Boot | Open source contributor")
            .city("Chittagong")
            .website("https://karim.dev")
            .makePrivate()
            .disableEmailNotifications()
            .build();
        System.out.println(full);
        
        // Clear, readable — each parameter is labeled:
        UserProfile readable = new Builder("hasan.ali", "hasan@test.com",
                                           "Hasan", "Ali")
            .age(28)
            .city("Dhaka")
            .bio("Software developer at Pathao")
            .build();
        System.out.println(readable);
        
        // Compare to the constructor nightmare:
        // new UserProfile("hasan.ali", "hasan@test.com", "Hasan", "Ali",
        //                 28, "Software developer at Pathao", "Dhaka",
        //                 null, true, true);
        // Which argument is what? No idea without looking at the class.
    }
}
```

---

## 7. Records - Concise Immutable Data Classes (Java 16+)

```java
// Records are a Java 16+ feature that generates:
// - Private final fields
// - Constructor with all fields
// - Getters (fieldName() not getFieldName())
// - equals(), hashCode(), toString()
// All automatically!

// Before Records (lots of boilerplate):
class PointOldWay {
    private final int x;
    private final int y;
    
    public PointOldWay(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public int getX() { return x; }
    public int getY() { return y; }
    
    // @Override equals/hashCode/toString...
}

// With Record (compact):
public record Point(int x, int y) {
    // That's it! Everything above is auto-generated.
    
    // You can add compact constructor for validation:
    public Point {
        if (x < 0 || y < 0)
            throw new IllegalArgumentException("Coordinates must be non-negative");
        // no assignment needed — done automatically
    }
    
    // Can add instance methods:
    public double distanceTo(Point other) {
        int dx = this.x - other.x;
        int dy = this.y - other.y;
        return Math.sqrt(dx * dx + dy * dy);
    }
    
    public Point translate(int dx, int dy) {
        return new Point(x + dx, y + dy); // returns new Point — immutable!
    }
}

// Records are perfect for:
// - DTOs (data transfer objects)
// - Value objects (Money, Coordinates, Color)
// - Database query results
// - API request/response classes

// Common Spring Boot DTO examples using Records:
record LoginRequest(String email, String password) {
    public LoginRequest {
        if (email == null || !email.contains("@"))
            throw new IllegalArgumentException("Invalid email");
        if (password == null || password.length() < 6)
            throw new IllegalArgumentException("Password too short");
    }
}

record LoginResponse(String token, String tokenType,
                     long expiresIn, String userEmail) { }

record ErrorResponse(int status, String error,
                     String message, String timestamp) {
    public static ErrorResponse of(int status, String error, String message) {
        return new ErrorResponse(status, error, message,
                                 java.time.LocalDateTime.now().toString());
    }
}

record ProductSummary(Long id, String name, double price, boolean available) { }

class RecordDemo {
    public static void main(String[] args) {
        
        // Point record:
        Point p1 = new Point(3, 4);
        Point p2 = new Point(3, 4);
        Point p3 = new Point(6, 8);
        
        // Getters: fieldName() — no 'get' prefix!
        System.out.println("x: " + p1.x()); // 3
        System.out.println("y: " + p1.y()); // 4
        
        // Auto-generated toString():
        System.out.println(p1); // Point[x=3, y=4]
        
        // Auto-generated equals():
        System.out.println(p1.equals(p2)); // true
        System.out.println(p1.equals(p3)); // false
        
        // Custom methods:
        System.out.printf("Distance p1 to p3: %.2f%n", p1.distanceTo(p3)); // 5.00
        System.out.println("Translated: " + p1.translate(1, 1)); // Point[x=4, y=5]
        
        // DTO records:
        LoginRequest request = new LoginRequest("rahim@test.com", "password123");
        System.out.println("Email: " + request.email()); // rahim@test.com
        
        LoginResponse response = new LoginResponse(
            "jwt-token-here", "Bearer", 3600, "rahim@test.com");
        System.out.println(response);
        
        ErrorResponse error = ErrorResponse.of(404, "Not Found", "User not found");
        System.out.println(error);
        
        // Invalid:
        try {
            new Point(-1, 5); // compact constructor validates
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

---

## 8. Encapsulation Anti-Patterns to Avoid

```java
public class EncapsulationAntiPatterns {
    
    // ─────────────────────────────────────────
    // ANTI-PATTERN 1: Public fields
    // ─────────────────────────────────────────
    
    // ❌ BAD:
    public class BadAccount {
        public double balance;  // anyone can set this to anything!
        public String status;   // no validation, no business rules
    }
    
    // Usage:
    // BadAccount acc = new BadAccount();
    // acc.balance = -999999; // works! data corrupted
    // acc.status = "INVALID_STATUS"; // no validation
    
    // ✅ GOOD: private fields + validated setters
    
    // ─────────────────────────────────────────
    // ANTI-PATTERN 2: Setter for everything (anemic domain model)
    // ─────────────────────────────────────────
    
    // ❌ BAD: Setter for every field with no validation
    public class AnemicUser {
        private String status;
        
        // Just sets the value — no business rules:
        public void setStatus(String status) {
            this.status = status; // accepts "ACTIVE", "INVALID", "🤡", anything
        }
    }
    
    // ✅ GOOD: Methods that enforce domain rules
    public class RichUser {
        private String status = "PENDING";
        
        public void activate() {
            if (!"PENDING".equals(status))
                throw new IllegalStateException("Can only activate PENDING users");
            this.status = "ACTIVE";
        }
        
        public void suspend(String reason) {
            if ("SUSPENDED".equals(status))
                throw new IllegalStateException("Already suspended");
            this.status = "SUSPENDED";
            // log reason, notify user, etc.
        }
        
        public String getStatus() { return status; }
    }
    
    // ─────────────────────────────────────────
    // ANTI-PATTERN 3: Returning mutable internal state
    // ─────────────────────────────────────────
    
    private java.util.List<String> items = new java.util.ArrayList<>();
    
    // ❌ BAD: Returns the actual internal list
    public java.util.List<String> getItemsBad() {
        return items; // caller can do: getItemsBad().clear() — wipes your data!
    }
    
    // ✅ GOOD: Return unmodifiable view or copy
    public java.util.List<String> getItems() {
        return java.util.Collections.unmodifiableList(items);
        // OR: return new java.util.ArrayList<>(items);
    }
    
    // ─────────────────────────────────────────
    // ANTI-PATTERN 4: Setter that does too much (unpredictable)
    // ─────────────────────────────────────────
    
    // ❌ BAD: Setter with complex side effects
    public void setEmailBad(String email_) {
        this.email = email_;
        sendVerificationEmail(email_); // surprise! setter sends email!
        updateAllRelatedUsers(email_); // surprise! updates other records!
        clearAllSessions();           // surprise! logs out everywhere!
    }
    
    // ✅ GOOD: Setter just sets, separate methods for behavior
    private String email;
    
    public void setEmail(String email) {
        if (email == null || !email.contains("@"))
            throw new IllegalArgumentException("Invalid email");
        this.email = email.toLowerCase().trim();
    }
    
    public void changeEmail(String newEmail) {
        setEmail(newEmail);           // validation through setter
        sendVerificationEmail(newEmail);
        clearAllSessions();
        // now it's explicit and documented
    }
    
    // ─────────────────────────────────────────
    // ANTI-PATTERN 5: Leaking 'this' from constructor
    // ─────────────────────────────────────────
    
    // ❌ BAD:
    public class BadService {
        public BadService(EventBus bus) {
            bus.register(this); // 'this' leaks before constructor finishes!
            // Other fields might not be set yet
            // Other threads could see partially-constructed object
        }
    }
    
    // ✅ GOOD: Register after construction is complete
    // BadService service = new BadService();
    // bus.register(service); // fully constructed before registering
    
    // Placeholder methods for compilation:
    private void sendVerificationEmail(String email) { }
    private void updateAllRelatedUsers(String email) { }
    private void clearAllSessions() { }
    
    class EventBus { void register(Object o) {} }
}
```

---

## Build This - Complete Encapsulation Practice

```java
// File: BankingEncapsulation.java
// A fully encapsulated banking system demonstrating
// all encapsulation concepts

import java.time.LocalDateTime;
import java.util.*;

public class BankingEncapsulation {

    // ═══════════════════════════════════════
    // MONEY VALUE OBJECT (Immutable)
    // ═══════════════════════════════════════
    
    static final class Taka {
        private final long poisha; // store in smallest unit
        
        private Taka(long poisha) {
            if (poisha < 0)
                throw new IllegalArgumentException("Amount cannot be negative");
            this.poisha = poisha;
        }
        
        public static Taka of(double taka) {
            return new Taka(Math.round(taka * 100));
        }
        
        public static Taka ofPoisha(long poisha) {
            return new Taka(poisha);
        }
        
        public static Taka zero() { return new Taka(0); }
        
        public Taka add(Taka other)      { return new Taka(this.poisha + other.poisha); }
        public Taka subtract(Taka other) {
            if (other.poisha > this.poisha)
                throw new IllegalArgumentException("Insufficient funds");
            return new Taka(this.poisha - other.poisha);
        }
        public Taka percentage(double pct) {
            return new Taka((long)(this.poisha * pct / 100));
        }
        
        public boolean isGreaterThan(Taka other) { return this.poisha > other.poisha; }
        public boolean isLessThan(Taka other)    { return this.poisha < other.poisha; }
        public boolean isZero()                  { return this.poisha == 0; }
        
        public long getPoisha()  { return poisha; }
        public double getTaka()  { return poisha / 100.0; }
        
        @Override
        public boolean equals(Object obj) {
            if (!(obj instanceof Taka)) return false;
            return this.poisha == ((Taka) obj).poisha;
        }
        
        @Override
        public int hashCode() { return Long.hashCode(poisha); }
        
        @Override
        public String toString() { return String.format("৳%.2f", getTaka()); }
    }

    // ═══════════════════════════════════════
    // TRANSACTION RECORD (Immutable Record)
    // ═══════════════════════════════════════
    
    record Transaction(String id, String type, Taka amount,
                       Taka balanceAfter, LocalDateTime timestamp,
                       String description) {
        
        public static Transaction of(String type, Taka amount,
                                      Taka balanceAfter, String description) {
            return new Transaction(
                "TXN-" + System.currentTimeMillis(),
                type, amount, balanceAfter,
                LocalDateTime.now(), description);
        }
        
        @Override
        public String toString() {
            return String.format("[%s] %-8s %8s → Balance: %s | %s",
                                 timestamp.toLocalTime(), type, amount,
                                 balanceAfter, description);
        }
    }

    // ═══════════════════════════════════════
    // ACCOUNT CLASS (Fully Encapsulated)
    // ═══════════════════════════════════════
    
    static class Account {
        
        // Class-level:
        private static int counter = 1;
        private static Taka totalBankAssets = Taka.zero();
        
        // Constants:
        private static final Taka MINIMUM_BALANCE = Taka.of(500);
        private static final Taka MAX_DAILY_WITHDRAWAL = Taka.of(50_000);
        private static final double INTEREST_RATE_ANNUAL = 4.5;
        
        // Instance fields — all private:
        private final String accountNumber;
        private final String holderName;
        private final LocalDateTime openedAt;
        private Taka balance;
        private boolean isFrozen;
        private final List<Transaction> history;
        private Taka totalWithdrawnToday;
        
        // ── Constructor ──
        public Account(String holderName, Taka initialDeposit) {
            validateHolderName(holderName);
            if (initialDeposit.isLessThan(MINIMUM_BALANCE))
                throw new IllegalArgumentException(
                    "Initial deposit must be at least " + MINIMUM_BALANCE);
            
            this.accountNumber    = String.format("ACC%06d", counter++);
            this.holderName       = holderName.trim();
            this.balance          = initialDeposit;
            this.isFrozen         = false;
            this.history          = new ArrayList<>();
            this.totalWithdrawnToday = Taka.zero();
            this.openedAt         = LocalDateTime.now();
            
            totalBankAssets = totalBankAssets.add(initialDeposit);
            
            history.add(Transaction.of("OPEN", initialDeposit, balance,
                                        "Account opened"));
        }
        
        // ── Public operations ──
        
        public boolean deposit(Taka amount) {
            if (isFrozen) return fail("Account is frozen");
            if (amount.isZero())
                return fail("Deposit amount must be positive");
            if (amount.isLessThan(Taka.of(10)))
                return fail("Minimum deposit is ৳10");
            
            balance = balance.add(amount);
            totalBankAssets = totalBankAssets.add(amount);
            history.add(Transaction.of("DEPOSIT", amount, balance, "Cash deposit"));
            return success("Deposited " + amount);
        }
        
        public boolean withdraw(Taka amount, String reason) {
            if (isFrozen) return fail("Account is frozen");
            if (amount.isZero()) return fail("Amount must be positive");
            if (amount.isGreaterThan(MAX_DAILY_WITHDRAWAL))
                return fail("Exceeds daily limit (" + MAX_DAILY_WITHDRAWAL + ")");
            
            Taka newWithdrawnToday = totalWithdrawnToday.add(amount);
            if (newWithdrawnToday.isGreaterThan(MAX_DAILY_WITHDRAWAL))
                return fail("Daily withdrawal limit would be exceeded");
            
            Taka balanceAfterWithdrawal = balance.subtract(
                amount); // throws if negative
            if (balanceAfterWithdrawal.isLessThan(MINIMUM_BALANCE))
                return fail("Would breach minimum balance (" + MINIMUM_BALANCE + ")");
            
            balance = balanceAfterWithdrawal;
            totalWithdrawnToday = newWithdrawnToday;
            totalBankAssets = totalBankAssets.subtract(amount);
            history.add(Transaction.of("WITHDRAW", amount, balance, reason));
            return success("Withdrawn " + amount + " for: " + reason);
        }
        
        public boolean applyMonthlyInterest() {
            if (isFrozen) return fail("Account is frozen — no interest");
            Taka interest = balance.percentage(INTEREST_RATE_ANNUAL / 12);
            balance = balance.add(interest);
            totalBankAssets = totalBankAssets.add(interest);
            history.add(Transaction.of("INTEREST", interest, balance,
                                        String.format("Monthly interest (%.2f%% annual)",
                                                      INTEREST_RATE_ANNUAL)));
            return success("Interest applied: " + interest);
        }
        
        public void freeze(String reason) {
            if (isFrozen) { System.out.println("⚠️  Already frozen"); return; }
            isFrozen = true;
            history.add(Transaction.of("FREEZE", Taka.zero(), balance,
                                        "Account frozen: " + reason));
            System.out.println("🔒 " + accountNumber + " frozen: " + reason);
        }
        
        public void unfreeze() {
            isFrozen = false;
            history.add(Transaction.of("UNFREEZE", Taka.zero(), balance,
                                        "Account unfrozen"));
            System.out.println("🔓 " + accountNumber + " unfrozen");
        }
        
        // ── Getters (controlled access) ──
        
        public String getAccountNumber()    { return accountNumber; }
        public String getHolderName()       { return holderName; }
        public Taka getBalance()            { return balance; }
        public boolean isFrozen()           { return isFrozen; }
        public LocalDateTime getOpenedAt()  { return openedAt; }
        
        // Return unmodifiable view — callers cannot tamper with history:
        public List<Transaction> getHistory() {
            return Collections.unmodifiableList(history);
        }
        
        // Computed:
        public String getMaskedAccountNumber() {
            return "****" + accountNumber.substring(accountNumber.length() - 4);
        }
        
        // Static class-level info:
        public static int getTotalAccounts()   { return counter - 1; }
        public static Taka getTotalAssets()    { return totalBankAssets; }
        
        // ── Private helpers ──
        
        private void validateHolderName(String name) {
            if (name == null || name.isBlank())
                throw new IllegalArgumentException("Holder name required");
            if (name.trim().length() < 2)
                throw new IllegalArgumentException("Name too short");
        }
        
        private boolean fail(String msg) {
            System.out.println("❌ " + accountNumber + ": " + msg);
            return false;
        }
        
        private boolean success(String msg) {
            System.out.println("✅ " + accountNumber + ": " + msg);
            return true;
        }
        
        // ── Display ──
        
        public void printStatement() {
            System.out.println("\n╔══════════════════════════════════════════════╗");
            System.out.printf( "║  Account  : %-32s║%n", accountNumber);
            System.out.printf( "║  Holder   : %-32s║%n", holderName);
            System.out.printf( "║  Balance  : %-32s║%n", balance);
            System.out.printf( "║  Status   : %-32s║%n",
                               isFrozen ? "🔒 FROZEN" : "✅ Active");
            System.out.println("╠══════════════════════════════════════════════╣");
            System.out.println("║  TRANSACTION HISTORY                         ║");
            System.out.println("╠══════════════════════════════════════════════╣");
            for (Transaction t : history) {
                System.out.printf("║  %-44s║%n",
                                  t.toString().length() > 44
                                      ? t.toString().substring(0, 41) + "..."
                                      : t.toString());
            }
            System.out.println("╚══════════════════════════════════════════════╝");
        }
        
        @Override
        public boolean equals(Object obj) {
            if (!(obj instanceof Account)) return false;
            return accountNumber.equals(((Account) obj).accountNumber);
        }
        
        @Override
        public int hashCode() { return accountNumber.hashCode(); }
        
        @Override
        public String toString() {
            return String.format("Account{%s, holder='%s', balance=%s, frozen=%b}",
                                 accountNumber, holderName, balance, isFrozen);
        }
    }

    // ═══════════════════════════════════════
    // MAIN
    // ═══════════════════════════════════════
    
    public static void main(String[] args) {
        
        System.out.println("╔══════════════════════════════════════════════╗");
        System.out.println("║        JAVA NATIONAL BANK DEMO               ║");
        System.out.println("╚══════════════════════════════════════════════╝\n");
        
        // Open accounts:
        Account rahim  = new Account("Rahim Ahmed",  Taka.of(10_000));
        Account karim  = new Account("Karim Hassan", Taka.of(25_000));
        Account hasan  = new Account("Hasan Ali",    Taka.of(500));   // minimum
        
        // Transactions:
        System.out.println("\n── Rahim's transactions ──");
        rahim.deposit(Taka.of(5_000));
        rahim.withdraw(Taka.of(2_000), "Shopping");
        rahim.withdraw(Taka.of(60_000), "Exceeds limit");       // fails
        rahim.withdraw(Taka.of(13_000), "Would breach minimum"); // fails
        rahim.applyMonthlyInterest();
        
        System.out.println("\n── Karim's transactions ──");
        karim.deposit(Taka.of(10_000));
        karim.freeze("Suspicious activity detected");
        karim.deposit(Taka.of(5_000)); // fails — frozen
        karim.withdraw(Taka.of(1_000), "Test"); // fails — frozen
        karim.unfreeze();
        karim.deposit(Taka.of(5_000)); // now works
        
        System.out.println("\n── Invalid operations ──");
        try {
            new Account("", Taka.of(5000));
        } catch (IllegalArgumentException e) {
            System.out.println("❌ Caught: " + e.getMessage());
        }
        try {
            new Account("Valid Name", Taka.of(100));
        } catch (IllegalArgumentException e) {
            System.out.println("❌ Caught: " + e.getMessage());
        }
        
        // Statements:
        rahim.printStatement();
        karim.printStatement();
        
        // Bank summary:
        System.out.println("\n═══════════════════════════════════════════");
        System.out.println("           BANK SUMMARY");
        System.out.println("═══════════════════════════════════════════");
        System.out.printf("Total Accounts : %d%n", Account.getTotalAccounts());
        System.out.printf("Total Assets   : %s%n", Account.getTotalAssets());
    }
}
```

---

## Exercises

```text
EXERCISE 1: Encapsulate a Broken Class
  Create EncapsulationFix.java
  Start with this broken "public fields" class:
  
  class Student {
      public String name;
      public int age;
      public double cgpa;
      public String email;
      public boolean isActive;
      public String[] courses; // mutable array exposed
  }
  
  Refactor it to be fully encapsulated:
  - Make all fields private
  - Add constructors with validation
  - Add getters with appropriate visibility
  - Add setters only where mutation makes sense
  - Add defensive copy for the array
  - Add computed getters: getGpaCategory(), isGoodStanding()
  - Override toString(), equals(), hashCode()
  - Demonstrate the before and after difference.

EXERCISE 2: Temperature Sensor
  Create TemperatureSensor.java
  Build a fully encapsulated Temperature class:
  - Private double value in Celsius
  - Private String unit ("CELSIUS", "FAHRENHEIT", "KELVIN")
  - Static final constants for freezing/boiling points
  - Constructor validates: Kelvin >= 0, all other reasonable ranges
  - getAsCelsius(), getAsFahrenheit(), getAsKelvin() — convert on demand
  - add(double amount) — returns NEW Temperature (immutable-style)
  - isAboveBoiling(), isBelowFreezing() — business logic
  - toString() shows value + unit
  Build a ThermostatController that tracks min/max readings.

EXERCISE 3: Builder Pattern Practice
  Create HttpRequestBuilder.java
  Implement a Builder for an HttpRequest class:
  Required: method (GET/POST/PUT/PATCH/DELETE), url
  Optional: headers (Map<String, String>),
            body (String),
            timeout (int, default 30),
            followRedirects (boolean, default true),
            authToken (String)
  Validate:
  - URL must start with http:// or https://
  - method must be valid HTTP verb
  - timeout must be 1-300 seconds
  Build 4 different requests using the builder.
  Print each request's details.

EXERCISE 4: Record Practice
  Create DataTransferRecords.java
  Design these Records for a REST API:
  - ProductDTO (id, name, price, category, available)
  - OrderDTO (orderId, products List<ProductDTO>, total, status)
  - UserDTO (id, name, email, role)
  - ApiResponse<T>(success, data T, message, timestamp)
  - PageResponse<T>(content List<T>, page, size, total, totalPages)
  Add compact constructors for validation where needed.
  Add useful instance methods.
  Demonstrate creating and using each.

EXERCISE 5: Full Encapsulation Challenge
  Create HospitalManagement.java
  Design a Patient class:
  - id (auto-generated, immutable)
  - name, dateOfBirth (calculate age from this)
  - bloodType (A+/A-/B+/B-/O+/O-/AB+/AB-)
  - List<String> allergies (defensive copy)
  - List<Prescription> prescriptions (defensive copy)
  - boolean isAdmitted
  - LocalDateTime admittedAt (null if not admitted)
  
  Prescription record: medicationName, dosage, frequency, prescribedBy
  
  Methods: admit(), discharge(), addAllergy(String),
           prescribe(Prescription), getAge(), getMedicalSummary()
  All with proper validation and encapsulation.

  Push to GitHub: "feat: encapsulation banking system"
```

---

## Common Mistakes

```text
MISTAKE 1: Public fields (most common encapsulation violation)
  public String name; // anyone can set any value
  Fix: private String name; + getName() + setName() with validation

MISTAKE 2: Setter with no validation
  public void setAge(int age) { this.age = age; } // accepts -999!
  Fix: validate before setting:
  if (age < 0 || age > 150) throw new IllegalArgumentException("...");

MISTAKE 3: Returning mutable internal collection directly
  public List<Item> getItems() { return items; } // caller can clear()!
  Fix: return Collections.unmodifiableList(items);
  or return new ArrayList<>(items);

MISTAKE 4: Getter for everything including sensitive data
  public String getPasswordHash() { return passwordHash; } // exposes hash!
  Fix: no getter for password hash in API responses.
  In UserResponse DTO: never include passwordHash.

MISTAKE 5: Not using 'this.' when parameter names shadow fields
  public void setName(String name) { name = name; } // sets param to itself!
  Fix: this.name = name;

MISTAKE 6: Making final fields settable
  private final String id;
  public void setId(String id) { this.id = id; } // COMPILE ERROR
  final = set once in constructor, never again. No setter.

MISTAKE 7: Constructor that accepts invalid state
  public Account(String holder, double balance) {
      this.holder = holder;  // what if null?
      this.balance = balance; // what if negative?
  }
  Fix: validate ALL inputs in constructor.
  An object should NEVER be in an invalid state.

MISTAKE 8: Builder without validation
  public Builder url(String url) {
      this.url = url; // accepts null, "not-a-url", ""
      return this;
  }
  Fix: validate in each builder method, not just build().

MISTAKE 9: Making the Builder class public but constructor private
  // This means no one can create objects except through Builder — intentional!
  // But sometimes people accidentally make the Builder private too:
  private static class Builder { ... } // now no one can use it!
  Fix: static nested Builder is typically public.

MISTAKE 10: Record with mutable field types
  record Order(List<String> items) {
      // items IS the internal list — callers can modify it!
  }
  Fix: defensive copy in compact constructor:
  record Order(List<String> items) {
      public Order { items = List.copyOf(items); } // immutable copy
  }
```

---

## Interview Questions

**Q: What is encapsulation and why is it important?**
A: Encapsulation is the OOP principle of bundling data (fields) and the methods that operate on that data into one unit (class), and restricting direct external access to the data. Fields are made private; access is through controlled public methods. Importance: 1) Data integrity — setters validate before changing fields, preventing invalid states. 2) Change freedom — internal implementation can change without breaking code that uses the class (as long as the public API stays the same). 3) Security — sensitive data like passwords can't be accidentally exposed or modified. 4) Simpler debugging — data changes go through a controlled path, easier to trace. In Spring Boot, this is fundamental — JPA entities, DTOs, services all use encapsulation.

**Q: What is the difference between private, protected, default, and public access modifiers?**
A: private: accessible only within the declaring class — most restrictive. Use for fields and internal helper methods. (no modifier/default): accessible within the same package only — used when classes in a package collaborate closely. protected: accessible in the same package AND in subclasses anywhere — used when designing classes meant to be extended. public: accessible everywhere — use for the class's public API. Rule of thumb: minimize visibility. Start with private and open up only when there's a clear reason.

**Q: Should you always provide getters and setters for all fields?**
A: No. This is a common misunderstanding. Rules: - Only add a getter if external code NEEDS to read the value. - Only add a setter if the field should be changeable AFTER construction, and include validation in it. - Fields that never change after construction (like id, createdAt) should have getters but NO setters — mark the field final. - Never add a setter just because "it might be useful." This exposes implementation details and undermines encapsulation. - Prefer domain-specific methods: activate(), changePassword() over generic setStatus(), setPasswordHash().

**Q: What is the Builder pattern and when would you use it?**
A: Builder is a creational design pattern where object construction is separated into a Builder class. You call Builder methods to set optional parameters (each returns 'this' for chaining), then call build() to get the final object. Use it when a class has more than 3-4 parameters especially with many optional ones, when parameter combination creates confusing constructors, or when you want an immutable object but readable construction code. Lombok's @Builder annotation generates this automatically in Spring Boot — a very common usage in real projects.

**Q: What is an immutable class? How do you create one?**
A: An immutable class's state cannot change after construction. Rules: 1) Make the class final (prevents mutable subclass). 2) Make all fields private and final. 3) No setters. 4) Operations return new instances instead of modifying self. 5) If fields are mutable objects (List, array), make defensive copies in constructor AND in getters. Examples: Java's String, Integer, LocalDate, BigDecimal. Benefits: thread-safe by nature, safe to share references, hashCode can be cached, simpler to reason about. In Spring Boot: DTOs and value objects (Money, Coordinate) are often designed as immutable records or immutable classes.

**Q: What are Java Records and when do you use them?**
A: Records (Java 16+) are a compact syntax for immutable data classes. Declaring record Point(int x, int y) {} automatically generates: private final fields, canonical constructor, getters named x() and y() (not getX()), equals(), hashCode(), and toString(). Use records for: DTOs (data transfer objects in Spring Boot APIs), value objects (Money, Coordinate), method return types carrying multiple values, database projection results. Records cannot extend other classes (they implicitly extend Record), cannot have non-final instance fields, but can have methods and implement interfaces.

---

## Key Takeaways

```text
1. Encapsulation = private fields + controlled public access.
   Data hiding prevents invalid state. Methods enforce business rules.
   Every professional Java class uses this pattern.

2. ACCESS MODIFIERS — most to least restrictive:
   private → (default) → protected → public
   Fields: ALWAYS private.
   Methods: public for API, private for helpers, protected for inheritance.

3. GETTERS provide read access. Name: getFieldName() or isFieldName().
   Not every field needs a getter.
   Getters can compute, transform, or mask data.
   Return defensive copies of mutable collections.

4. SETTERS provide write access WITH VALIDATION.
   Not every field needs a setter.
   Validate before setting. Throw IllegalArgumentException on bad input.
   Prefer domain methods (activate(), suspend()) over generic setters.

5. IMMUTABLE FIELDS: use final + no setter.
   IDs, creation timestamps — set once, never changed.
   Immutable classes: final class + all fields final + no setters.

6. BUILDER PATTERN for complex objects with many optional fields.
   Each builder method returns 'this' for chaining.
   Build validates and creates the final immutable object.
   Lombok @Builder generates this automatically in Spring Boot.

7. RECORDS (Java 16+) for concise immutable data classes.
   Perfect for DTOs, value objects.
   Auto-generates: fields, constructor, getters, equals, hashCode, toString.
   Getters named fieldName() not getFieldName().

8. DEFENSIVE COPIES for mutable return values.
   return new ArrayList<>(internalList) or unmodifiableList().
   Prevents callers from tampering with internal state.

9. CONSTRUCTOR validates everything.
   Object should NEVER be in an invalid state.
   Better to throw in constructor than have corrupted data.

10. In Spring Boot: entities, DTOs, services — ALL use encapsulation.
    JPA entities: private fields + getters/setters (Jackson/JPA need them).
    DTOs: private fields + getter/setter or Records.
    Domain objects: private fields + domain methods.
```

---