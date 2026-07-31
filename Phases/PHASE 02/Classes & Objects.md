**Phase:** Level 2 - OOP
**Date Studied:**

---
## What Problem Does This Solve?
```text
So far you've written code like this:

  String name = "Rahim";
  String email = "rahim@example.com";
  int age = 21;
  String city = "Dhaka";
  boolean isActive = true;
  
  String name2 = "Karim";
  String email2 = "karim@example.com";
  int age2 = 25;
  String city2 = "Chittagong";
  boolean isActive2 = true;
  
  // 100 users later:
  String name100 = ...
  String email100 = ...

This is chaos. No structure. No safety. No scalability.
What if you need to pass a "user" to a method?
void processUser(String name, String email, int age, String city, boolean isActive)
// 5 parameters just for one user — and growing

Real programs model the REAL WORLD.
The real world has:
  Users (with name, email, password, address)
  Products (with name, price, stock, category)
  Orders (with items, total, status, customer)
  Bank accounts (with balance, owner, transactions)

Object-Oriented Programming solves this by letting you
CREATE YOUR OWN TYPES that model real-world concepts:

  class User {
      String name;
      String email;
      int age;
      String city;
      boolean isActive;
  }

  User user1 = new User();  // ONE variable for everything
  User user2 = new User();  // Another independent user

  processUser(user1);       // ONE parameter — clean!

This is the foundation of how ALL Java applications are built.
Every Spring Boot service, entity, DTO, repository — all classes.
Understanding OOP is understanding professional Java.
```

---

## 1. What Is OOP?
```text
Object-Oriented Programming (OOP) is a paradigm
where you organize code around OBJECTS that represent
real-world things or concepts.

The four pillars of OOP:
  1. Encapsulation  → hide internal details (this note + 1.11)
  2. Inheritance    → reuse and extend behavior (Level 1.12)
  3. Polymorphism   → many forms (Level 1.13)
  4. Abstraction    → show only what's needed (Level 1.14)

We start with the foundation: Classes and Objects.

The core analogy — use this to explain to anyone:

  CLASS  = Blueprint (design/plan)
  OBJECT = Instance (the actual thing built from the plan)

  ┌────────────────────────────────────────────────────┐
  │              BLUEPRINT: User                       │
  │  Properties: name, email, age, isActive            │
  │  Behaviors: login(), logout(), updateProfile()     │
  └──────────────────────┬─────────────────────────────┘
                         │ build many instances from one blueprint
              ┌──────────┼──────────┐
              ▼          ▼          ▼
         User #1     User #2     User #3
         name=Rahim  name=Karim  name=Hasan
         email=...   email=...   email=...
         age=21      age=25      age=28

  One blueprint → unlimited objects.
  Each object has its OWN copy of the properties.
  All objects share the same behaviors (methods).
```

---

## 2. Defining a Class

```java
// File: User.java
// A class definition — the blueprint

public class User {
    
    // ─────────────────────────────────────────
    // FIELDS (instance variables / properties)
    // What each User object HAS
    // ─────────────────────────────────────────
    
    String name;        // The user's full name
    String email;       // Login identifier
    int age;            // User's age
    String city;        // Current city
    boolean isActive;   // Account status
    
    // Fields have DEFAULT VALUES when not set:
    // int → 0
    // double → 0.0
    // boolean → false
    // String/Object → null
    
    // ─────────────────────────────────────────
    // METHODS (behaviors)
    // What each User object CAN DO
    // ─────────────────────────────────────────
    
    void displayInfo() {
        System.out.println("User: " + name);
        System.out.println("Email: " + email);
        System.out.println("Age: " + age);
        System.out.println("City: " + city);
        System.out.println("Active: " + isActive);
    }
    
    void activate() {
        isActive = true;
        System.out.println(name + " has been activated.");
    }
    
    void deactivate() {
        isActive = false;
        System.out.println(name + " has been deactivated.");
    }
    
    boolean canVote() {
        return age >= 18;
    }
    
    String getSummary() {
        return name + " (" + email + ") - " + (isActive ? "Active" : "Inactive");
    }
}
```

---

## 3. Creating Objects (Instantiation)

```java
public class ObjectCreation {
    public static void main(String[] args) {
        
        // Creating an object from the User blueprint:
        //
        //  new User()
        //   │
        //   │  1. JVM allocates memory on the HEAP for the object
        //   │  2. All fields set to defaults (0, false, null)
        //   │  3. Returns a REFERENCE (memory address) to the object
        //   │
        //   ▼
        //  user1 variable on STACK holds the REFERENCE
        
        User user1 = new User(); // create first User object
        
        // Set field values using dot notation:
        user1.name     = "Rahim Ahmed";
        user1.email    = "rahim@example.com";
        user1.age      = 21;
        user1.city     = "Dhaka";
        user1.isActive = true;
        
        // Call methods using dot notation:
        user1.displayInfo();
        System.out.println("Can vote: " + user1.canVote());
        System.out.println("Summary: " + user1.getSummary());
        
        System.out.println();
        
        // Create a SECOND object — completely independent:
        User user2 = new User();
        user2.name     = "Karim Hassan";
        user2.email    = "karim@example.com";
        user2.age      = 16;
        user2.city     = "Chittagong";
        user2.isActive = false;
        
        user2.displayInfo();
        System.out.println("Can vote: " + user2.canVote()); // false (age 16)
        
        // Changing one object does NOT affect the other:
        user1.city = "Sylhet";
        System.out.println("\nAfter changing user1's city:");
        System.out.println("user1.city: " + user1.city); // Sylhet
        System.out.println("user2.city: " + user2.city); // Chittagong — unchanged
        
        // ─────────────────────────────────────────
        // MEMORY PICTURE
        // ─────────────────────────────────────────
        
        // STACK:                    HEAP:
        // user1 → address-001       [User Object @ 001]
        //                             name: "Rahim Ahmed"
        //                             email: "rahim@example.com"
        //                             age: 21
        //                             city: "Sylhet"
        //                             isActive: true
        //
        // user2 → address-002       [User Object @ 002]
        //                             name: "Karim Hassan"
        //                             email: "karim@example.com"
        //                             age: 16
        //                             city: "Chittagong"
        //                             isActive: false
        
        // ─────────────────────────────────────────
        // REFERENCE ASSIGNMENT
        // ─────────────────────────────────────────
        
        // Assigning one reference to another copies the ADDRESS,
        // not the object:
        User user3 = user1; // user3 points to the SAME object as user1
        
        user3.name = "Modified Name";
        System.out.println("\nuser1.name: " + user1.name); // "Modified Name"!
        // user1 and user3 point to the SAME object — changing via user3 affects user1
        
        System.out.println("Same object? " + (user1 == user3)); // true
        System.out.println("Same object? " + (user1 == user2)); // false
        
        // ─────────────────────────────────────────
        // NULL REFERENCE
        // ─────────────────────────────────────────
        
        User user4 = null; // no object — reference points to nothing
        System.out.println("\nuser4 is null: " + (user4 == null)); // true
        
        // Calling any method/field on null → NullPointerException:
        try {
            user4.displayInfo(); // NPE!
        } catch (NullPointerException e) {
            System.out.println("NullPointerException: user4 has no object!");
        }
        
        // Always check for null before using a reference:
        if (user4 != null) {
            user4.displayInfo();
        } else {
            System.out.println("No user to display.");
        }
        
        // ─────────────────────────────────────────
        // ARRAY OF OBJECTS
        // ─────────────────────────────────────────
        
        User[] team = new User[3]; // array of 3 User references (all null initially)
        
        team[0] = new User();
        team[0].name = "Alice";
        team[0].age  = 28;
        
        team[1] = new User();
        team[1].name = "Bob";
        team[1].age  = 32;
        
        team[2] = new User();
        team[2].name = "Charlie";
        team[2].age  = 25;
        
        System.out.println("\nTeam members:");
        for (User member : team) {
            System.out.println("  " + member.name + " (age " + member.age + ")");
        }
    }
}
```

---

## 4. Constructors - Initializing Objects Properly

```java
public class ConstructorDemo {
    
    // ─────────────────────────────────────────
    // WHAT IS A CONSTRUCTOR?
    // ─────────────────────────────────────────
    //
    // A constructor is a special method that runs when
    // you create an object with 'new'.
    //
    // RULES:
    //  → Same name as the class
    //  → No return type (not even void)
    //  → Called automatically by 'new'
    //  → Purpose: initialize the object's fields
    
    // ─────────────────────────────────────────
    // NO-ARG CONSTRUCTOR (default)
    // ─────────────────────────────────────────
    
    String name;
    String email;
    int age;
    boolean isActive;
    
    // Default no-arg constructor
    public ConstructorDemo() {
        // If you don't write ANY constructor, Java provides this automatically
        // But if you write any constructor, Java NO LONGER provides the default
        name     = "Unknown";
        email    = "";
        age      = 0;
        isActive = false;
    }
    
    public static void main(String[] args) {
        ConstructorDemo obj = new ConstructorDemo(); // calls the no-arg constructor
        System.out.println(obj.name); // "Unknown"
    }
}

// Better example — a class with multiple constructors:
class Product {
    
    // ─────────────────────────────────────────
    // FIELDS
    // ─────────────────────────────────────────
    
    String name;
    String description;
    double price;
    int stockQuantity;
    String category;
    boolean isAvailable;
    String productId;
    
    // ─────────────────────────────────────────
    // CONSTRUCTOR 1: No arguments (minimal)
    // ─────────────────────────────────────────
    
    public Product() {
        this.name         = "Unnamed Product";
        this.price        = 0.0;
        this.stockQuantity = 0;
        this.isAvailable  = false;
        this.productId    = generateId();
    }
    
    // ─────────────────────────────────────────
    // CONSTRUCTOR 2: Essential fields only
    // ─────────────────────────────────────────
    
    public Product(String name, double price) {
        this.name         = name;
        this.price        = price;
        this.stockQuantity = 0;
        this.isAvailable  = false;
        this.productId    = generateId();
    }
    
    // ─────────────────────────────────────────
    // CONSTRUCTOR 3: All fields
    // ─────────────────────────────────────────
    
    public Product(String name, String description, double price,
                   int stockQuantity, String category) {
        this.name          = name;
        this.description   = description;
        this.price         = price;
        this.stockQuantity = stockQuantity;
        this.category      = category;
        this.isAvailable   = stockQuantity > 0;
        this.productId     = generateId();
    }
    
    // ─────────────────────────────────────────
    // CONSTRUCTOR CHAINING with this()
    // ─────────────────────────────────────────
    // Call another constructor from within a constructor.
    // this() must be the FIRST statement.
    // Avoids code duplication.
    
    public Product(String name, double price, int stock) {
        this(name, null, price, stock, "General"); // calls 5-param constructor
    }
    
    // ─────────────────────────────────────────
    // METHODS
    // ─────────────────────────────────────────
    
    void addStock(int quantity) {
        if (quantity <= 0) {
            System.out.println("Quantity must be positive");
            return;
        }
        stockQuantity += quantity;
        isAvailable = true;
        System.out.printf("Added %d units. Total stock: %d%n", quantity, stockQuantity);
    }
    
    boolean purchase(int quantity) {
        if (!isAvailable || stockQuantity < quantity) {
            System.out.println("Cannot purchase: insufficient stock");
            return false;
        }
        stockQuantity -= quantity;
        isAvailable = stockQuantity > 0;
        System.out.printf("Purchased %d units. Remaining: %d%n", quantity, stockQuantity);
        return true;
    }
    
    double getTotalValue() {
        return price * stockQuantity;
    }
    
    void display() {
        System.out.println("─".repeat(40));
        System.out.println("ID       : " + productId);
        System.out.println("Name     : " + name);
        System.out.println("Category : " + (category != null ? category : "None"));
        System.out.printf( "Price    : ৳%.2f%n", price);
        System.out.println("Stock    : " + stockQuantity);
        System.out.println("Available: " + (isAvailable ? "✅ Yes" : "❌ No"));
        System.out.printf( "Value    : ৳%.2f%n", getTotalValue());
    }
    
    private String generateId() {
        return "PRD-" + (int)(Math.random() * 9000 + 1000);
    }
    
    public static void main(String[] args) {
        // Using different constructors:
        Product p1 = new Product();
        p1.display();
        
        System.out.println();
        
        Product p2 = new Product("Laptop", 75000.0);
        p2.display();
        
        System.out.println();
        
        Product p3 = new Product(
            "Samsung Galaxy A55",
            "6.6\" AMOLED, 8GB RAM, 256GB storage",
            45000.0,
            50,
            "Smartphones"
        );
        p3.display();
        
        System.out.println();
        p3.purchase(5);
        p3.addStock(20);
        p3.display();
    }
}
```

---

## 5. The `this` Keyword - Complete Guide

```java
public class ThisKeyword {
    
    // 'this' refers to the CURRENT OBJECT.
    // The object whose method/constructor is currently executing.
    
    // ─────────────────────────────────────────
    // USE 1: Resolve field/parameter name conflicts
    // ─────────────────────────────────────────
    
    String name;
    int age;
    String email;
    
    // Parameter names shadow (hide) field names when same name:
    public ThisKeyword(String name, int age, String email) {
        // Without 'this': name = name would assign parameter to itself!
        this.name  = name;  // this.name = field, name = parameter
        this.age   = age;   // this.age = field, age = parameter
        this.email = email; // this.email = field, email = parameter
    }
    
    // Field names different from params — this. not needed (but still valid):
    public void setNameByDifferentParam(String newName) {
        name = newName;        // OK — no conflict
        this.name = newName;   // Same thing — just more explicit
    }
    
    // ─────────────────────────────────────────
    // USE 2: Call another method on current object
    // ─────────────────────────────────────────
    
    public void validate() {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name required");
        }
        if (age < 0 || age > 120) {
            throw new IllegalArgumentException("Invalid age: " + age);
        }
        if (!email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
    }
    
    public void display() {
        this.validate(); // calls validate() on THIS object
        // Could also just write: validate(); (same thing)
        System.out.println(name + " (" + age + ") - " + email);
    }
    
    // ─────────────────────────────────────────
    // USE 3: Constructor chaining with this()
    // ─────────────────────────────────────────
    
    public ThisKeyword() {
        this("Unknown", 0, ""); // calls the 3-param constructor
    }
    
    public ThisKeyword(String name) {
        this(name, 0, ""); // calls the 3-param constructor
    }
    
    // this() MUST be first statement in constructor
    // Cannot have this() and super() in same constructor
    
    // ─────────────────────────────────────────
    // USE 4: Return current object (method chaining / Builder pattern)
    // ─────────────────────────────────────────
    
    public ThisKeyword setName(String name) {
        this.name = name;
        return this; // return the CURRENT object
    }
    
    public ThisKeyword setAge(int age) {
        this.age = age;
        return this;
    }
    
    public ThisKeyword setEmail(String email) {
        this.email = email;
        return this;
    }
    
    // Now you can CHAIN method calls:
    // user.setName("Rahim").setAge(21).setEmail("r@test.com")
    // Each method returns 'this' so you can call next method on the result
    
    // ─────────────────────────────────────────
    // USE 5: Pass current object to another method
    // ─────────────────────────────────────────
    
    public void registerWith(UserRegistry registry) {
        registry.register(this); // pass THIS object to registry
    }
    
    public static void main(String[] args) {
        
        // Normal construction:
        ThisKeyword u1 = new ThisKeyword("Rahim", 21, "rahim@test.com");
        u1.display();
        
        // Constructor chaining:
        ThisKeyword u2 = new ThisKeyword("Karim");
        System.out.println(u2.name); // "Karim"
        System.out.println(u2.age);  // 0 (from chained constructor)
        
        // Method chaining with 'this':
        ThisKeyword u3 = new ThisKeyword()
            .setName("Hasan")
            .setAge(25)
            .setEmail("hasan@test.com");
        u3.display();
    }
}

// Dummy class for the example:
class UserRegistry {
    void register(ThisKeyword user) {
        System.out.println("Registered: " + user.name);
    }
}
```

---

## 6. Static vs Instance Members

```java
public class StaticVsInstance {
    
    // ─────────────────────────────────────────
    // INSTANCE MEMBERS (non-static)
    // → Belong to each OBJECT
    // → Each object has its OWN COPY
    // → Accessed through object reference: object.field
    // ─────────────────────────────────────────
    
    String name;          // instance field — each object has own name
    double balance;       // instance field — each object has own balance
    boolean isActive;     // instance field
    
    public void deposit(double amount) { // instance method
        this.balance += amount;
        System.out.println(name + " deposited ৳" + amount);
    }
    
    public double getBalance() { // instance method
        return this.balance;
    }
    
    // ─────────────────────────────────────────
    // STATIC MEMBERS (class-level)
    // → Belong to the CLASS itself, not any object
    // → ONE copy shared by ALL objects
    // → Accessed through class name: ClassName.field
    // → Exist without any object being created
    // ─────────────────────────────────────────
    
    static int totalAccounts = 0;           // ONE counter for ALL accounts
    static final double INTEREST_RATE = 0.045; // class constant
    static final String BANK_NAME = "Java National Bank"; // class constant
    
    public StaticVsInstance(String name, double initialBalance) {
        this.name    = name;
        this.balance = initialBalance;
        this.isActive = true;
        totalAccounts++; // increment class-level counter
        System.out.println("Account created. Total accounts: " + totalAccounts);
    }
    
    // Static method — can only access static members directly
    // Cannot access instance fields/methods (no 'this' exists)
    public static int getTotalAccounts() {
        return totalAccounts;
        // return name; // COMPILE ERROR: name is an instance field!
    }
    
    public static double getInterestRate() {
        return INTEREST_RATE;
    }
    
    // Static utility methods (don't need object state):
    public static double calculateTax(double amount) {
        return amount * 0.15;
    }
    
    public static boolean isValidAmount(double amount) {
        return amount > 0 && amount <= 1_000_000;
    }
    
    // Instance method CAN access both static and instance members:
    public void applyMonthlyInterest() {
        double interest = balance * (INTEREST_RATE / 12); // uses static INTEREST_RATE
        balance += interest;
        System.out.printf("%s interest: ৳%.2f (rate: %.1f%%)%n",
                          name, interest, INTEREST_RATE * 100);
    }
    
    public static void main(String[] args) {
        
        System.out.println("=== Static Members ===");
        
        // Access static members WITHOUT any object:
        System.out.println("Bank: " + StaticVsInstance.BANK_NAME);
        System.out.println("Rate: " + (StaticVsInstance.INTEREST_RATE * 100) + "%");
        System.out.println("Accounts: " + StaticVsInstance.getTotalAccounts()); // 0
        System.out.println("Tax on 1000: " + StaticVsInstance.calculateTax(1000));
        
        System.out.println("\n=== Creating Objects ===");
        
        // Create objects — totalAccounts increments each time:
        StaticVsInstance acc1 = new StaticVsInstance("Rahim", 10000);
        StaticVsInstance acc2 = new StaticVsInstance("Karim", 25000);
        StaticVsInstance acc3 = new StaticVsInstance("Hasan", 5000);
        
        System.out.println("\n=== Instance Members ===");
        
        // Each object has OWN balance:
        acc1.deposit(5000);
        acc2.deposit(10000);
        
        System.out.println("acc1 balance: ৳" + acc1.getBalance()); // 15000
        System.out.println("acc2 balance: ৳" + acc2.getBalance()); // 35000
        // Modifying acc1 does NOT affect acc2
        
        System.out.println("\n=== Static Counter ===");
        
        // ONE shared counter — reflects ALL objects:
        System.out.println("Total accounts: " + StaticVsInstance.getTotalAccounts()); // 3
        System.out.println("Total accounts: " + acc1.totalAccounts); // 3 (same value, warn: use class name)
        System.out.println("Total accounts: " + acc2.totalAccounts); // 3 (same value)
        // They all see the same static field!
        
        System.out.println("\n=== Monthly Interest ===");
        acc1.applyMonthlyInterest();
        acc2.applyMonthlyInterest();
        acc3.applyMonthlyInterest();
        
        System.out.println("\n=== SUMMARY ===");
        System.out.println("Total accounts at " + StaticVsInstance.BANK_NAME + ": "
                          + StaticVsInstance.getTotalAccounts());
    }
}
```

---

## 7. The Object Class - What Every Class Inherits

```java
public class ObjectClassDemo {
    
    // Every class in Java implicitly extends Object.
    // Object provides several methods all classes inherit:
    
    // ─────────────────────────────────────────
    // toString() — String representation
    // ─────────────────────────────────────────
    
    static class WithoutToString {
        String name;
        int value;
        
        WithoutToString(String name, int value) {
            this.name = name;
            this.value = value;
        }
        // No toString() override — uses Object's default
    }
    
    static class WithToString {
        String name;
        int value;
        
        WithToString(String name, int value) {
            this.name = name;
            this.value = value;
        }
        
        @Override
        public String toString() {
            return "WithToString{name='" + name + "', value=" + value + "}";
        }
    }
    
    // ─────────────────────────────────────────
    // equals() — content comparison
    // ─────────────────────────────────────────
    
    static class Point {
        int x, y;
        
        Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
        
        // Without equals(): uses Object's which compares references
        // With equals(): we define what "equal" means for our class
        
        @Override
        public boolean equals(Object obj) {
            // Step 1: check if same reference (optimization)
            if (this == obj) return true;
            // Step 2: check if null
            if (obj == null) return false;
            // Step 3: check if same class
            if (getClass() != obj.getClass()) return false;
            // Step 4: cast and compare fields
            Point other = (Point) obj;
            return this.x == other.x && this.y == other.y;
        }
        
        // RULE: Always override hashCode() when you override equals()!
        // If two objects are equal, they MUST have the same hashCode.
        // This is required for HashMap, HashSet to work correctly.
        @Override
        public int hashCode() {
            return 31 * x + y; // simple hash combining both fields
            // Better: Objects.hash(x, y); // use this in real code
        }
        
        @Override
        public String toString() {
            return "Point(" + x + ", " + y + ")";
        }
    }
    
    public static void main(String[] args) {
        
        System.out.println("=== toString() ===");
        
        WithoutToString wo = new WithoutToString("test", 42);
        System.out.println(wo); // "ObjectClassDemo$WithoutToString@hashCode"
        // Useless output — just the class name and hash
        
        WithToString wt = new WithToString("test", 42);
        System.out.println(wt); // "WithToString{name='test', value=42}"
        // Useful! Shows actual field values
        
        // toString() is called automatically by println, + concatenation, etc.
        System.out.println("Object: " + wt); // toString() called automatically
        
        System.out.println("\n=== equals() and hashCode() ===");
        
        Point p1 = new Point(3, 4);
        Point p2 = new Point(3, 4); // same values, different objects
        Point p3 = new Point(5, 6); // different values
        
        // Reference equality:
        System.out.println("p1 == p2: " + (p1 == p2)); // false (different objects)
        System.out.println("p1 == p3: " + (p1 == p3)); // false
        
        // Content equality (our custom equals):
        System.out.println("p1.equals(p2): " + p1.equals(p2)); // true (same x,y)
        System.out.println("p1.equals(p3): " + p1.equals(p3)); // false (different x,y)
        System.out.println("p1.equals(null): " + p1.equals(null)); // false (null-safe)
        
        // hashCode:
        System.out.println("p1.hashCode(): " + p1.hashCode()); // same as p2!
        System.out.println("p2.hashCode(): " + p2.hashCode()); // same as p1!
        // Equal objects MUST have equal hash codes (required contract)
        
        // Practical impact — works correctly in HashMap/HashSet:
        java.util.Set<Point> points = new java.util.HashSet<>();
        points.add(p1);
        points.add(p2); // same as p1 by equals → NOT added (already in set)
        points.add(p3);
        System.out.println("Set size: " + points.size()); // 2 (not 3!)
        
        System.out.println("\n=== getClass() and instanceof ===");
        
        System.out.println(p1.getClass()); // class ObjectClassDemo$Point
        System.out.println(p1.getClass().getSimpleName()); // "Point"
        System.out.println(p1 instanceof Point); // true
        System.out.println(p1 instanceof Object); // true (everything extends Object)
        
        System.out.println("\n=== Other Object methods ===");
        
        // hashCode() — unique(ish) number for each object
        WithoutToString obj1 = new WithoutToString("a", 1);
        WithoutToString obj2 = new WithoutToString("b", 2);
        System.out.println("obj1.hashCode(): " + obj1.hashCode());
        System.out.println("obj2.hashCode(): " + obj2.hashCode());
        // Without override: usually memory-address-based
    }
}
```

---

## 8. Designing Classes Well

```java
public class ClassDesign {
    
    // ─────────────────────────────────────────
    // EXAMPLE: A well-designed BankAccount class
    // Applies everything learned in this note
    // ─────────────────────────────────────────
    
    static class BankAccount {
        
        // ── Constants (static final) ──
        private static final double MIN_DEPOSIT = 10.0;
        private static final double MAX_WITHDRAWAL = 50_000.0;
        private static final double MINIMUM_BALANCE = 500.0;
        
        // ── Class-level tracking (static) ──
        private static int accountCounter = 0;
        private static double totalDepositsAllAccounts = 0;
        
        // ── Instance fields ──
        private final String accountNumber; // final — never changes after creation
        private final String holderName;    // final — never changes
        private double balance;
        private boolean isFrozen;
        private int transactionCount;
        
        // ── Constructors ──
        public BankAccount(String holderName, double initialDeposit) {
            if (holderName == null || holderName.isBlank()) {
                throw new IllegalArgumentException("Holder name cannot be blank");
            }
            if (initialDeposit < MINIMUM_BALANCE) {
                throw new IllegalArgumentException(
                    "Initial deposit must be at least ৳" + MINIMUM_BALANCE);
            }
            this.holderName     = holderName.trim();
            this.balance        = initialDeposit;
            this.isFrozen       = false;
            this.transactionCount = 1;
            this.accountNumber  = generateAccountNumber();
            
            accountCounter++;
            totalDepositsAllAccounts += initialDeposit;
        }
        
        // ── Static factory method (alternative to constructor) ──
        public static BankAccount createSavingsAccount(String name) {
            return new BankAccount(name, MINIMUM_BALANCE);
        }
        
        // ── Business methods ──
        public boolean deposit(double amount) {
            if (isFrozen) {
                System.out.println("❌ Account is frozen. Cannot deposit.");
                return false;
            }
            if (amount < MIN_DEPOSIT) {
                System.out.printf("❌ Minimum deposit is ৳%.2f%n", MIN_DEPOSIT);
                return false;
            }
            balance += amount;
            transactionCount++;
            totalDepositsAllAccounts += amount;
            System.out.printf("✅ Deposited ৳%.2f. New balance: ৳%.2f%n", amount, balance);
            return true;
        }
        
        public boolean withdraw(double amount) {
            if (isFrozen) {
                System.out.println("❌ Account is frozen. Cannot withdraw.");
                return false;
            }
            if (amount <= 0 || amount > MAX_WITHDRAWAL) {
                System.out.printf("❌ Amount must be between ৳1 and ৳%.2f%n", MAX_WITHDRAWAL);
                return false;
            }
            if (balance - amount < MINIMUM_BALANCE) {
                System.out.printf("❌ Cannot withdraw. Minimum balance is ৳%.2f%n", MINIMUM_BALANCE);
                return false;
            }
            balance -= amount;
            transactionCount++;
            System.out.printf("✅ Withdrawn ৳%.2f. New balance: ৳%.2f%n", amount, balance);
            return true;
        }
        
        public void freeze() {
            isFrozen = true;
            System.out.println("🔒 Account " + accountNumber + " frozen.");
        }
        
        public void unfreeze() {
            isFrozen = false;
            System.out.println("🔓 Account " + accountNumber + " unfrozen.");
        }
        
        // ── Getters (controlled access — no setters for sensitive fields) ──
        public String getAccountNumber() { return accountNumber; }
        public String getHolderName()    { return holderName; }
        public double getBalance()       { return balance; }
        public boolean isFrozen()        { return isFrozen; }
        public int getTransactionCount() { return transactionCount; }
        
        // ── Static getters (class-level info) ──
        public static int getTotalAccounts()           { return accountCounter; }
        public static double getTotalDeposits()        { return totalDepositsAllAccounts; }
        
        // ── Object description ──
        @Override
        public String toString() {
            return String.format(
                "BankAccount{acc='%s', holder='%s', balance=৳%.2f, frozen=%b, txns=%d}",
                accountNumber, holderName, balance, isFrozen, transactionCount);
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (!(obj instanceof BankAccount)) return false;
            BankAccount other = (BankAccount) obj;
            return accountNumber.equals(other.accountNumber); // accounts are equal if same number
        }
        
        @Override
        public int hashCode() {
            return accountNumber.hashCode();
        }
        
        // ── Private helpers ──
        private static String generateAccountNumber() {
            return String.format("ACC%07d", accountCounter + 1);
        }
    }
    
    public static void main(String[] args) {
        
        System.out.println("=== Creating Accounts ===");
        BankAccount acc1 = new BankAccount("Rahim Ahmed", 5000.0);
        BankAccount acc2 = new BankAccount("Karim Hassan", 10000.0);
        BankAccount acc3 = BankAccount.createSavingsAccount("Hasan Ali");
        
        System.out.println("\n=== Transactions ===");
        acc1.deposit(2000.0);
        acc1.withdraw(1000.0);
        acc1.withdraw(10000.0); // will fail — exceeds balance minus minimum
        acc2.deposit(5.0);      // will fail — below minimum deposit
        
        System.out.println("\n=== Freeze/Unfreeze ===");
        acc2.freeze();
        acc2.deposit(1000.0);  // will fail — frozen
        acc2.unfreeze();
        acc2.deposit(1000.0);  // succeeds now
        
        System.out.println("\n=== Account Details ===");
        System.out.println(acc1);
        System.out.println(acc2);
        System.out.println(acc3);
        
        System.out.println("\n=== Class-Level Statistics ===");
        System.out.printf("Total accounts    : %d%n", BankAccount.getTotalAccounts());
        System.out.printf("Total deposited   : ৳%.2f%n", BankAccount.getTotalDeposits());
        
        System.out.println("\n=== Equals and HashCode ===");
        BankAccount ref1 = acc1;
        System.out.println("acc1 == ref1: " + (acc1 == ref1)); // true (same ref)
        System.out.println("acc1.equals(ref1): " + acc1.equals(ref1)); // true
        System.out.println("acc1.equals(acc2): " + acc1.equals(acc2)); // false
        
        System.out.println("\n=== Invalid Creation ===");
        try {
            BankAccount bad = new BankAccount("", 5000.0);
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        try {
            BankAccount bad2 = new BankAccount("Valid Name", 100.0);
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

---

## Build This - Complete OOP Design

```java
// File: LibrarySystem.java
// A library management system using OOP principles

import java.util.ArrayList;
import java.util.List;

public class LibrarySystem {

    // ═══════════════════════════════════════════
    // BOOK CLASS
    // ═══════════════════════════════════════════
    
    static class Book {
        
        private static int idCounter = 1;
        
        private final int id;
        private final String isbn;
        private String title;
        private String author;
        private int year;
        private String genre;
        private int totalCopies;
        private int availableCopies;
        
        public Book(String isbn, String title, String author,
                    int year, String genre, int copies) {
            this.id              = idCounter++;
            this.isbn            = isbn;
            this.title           = title;
            this.author          = author;
            this.year            = year;
            this.genre           = genre;
            this.totalCopies     = copies;
            this.availableCopies = copies;
        }
        
        public boolean checkout() {
            if (availableCopies <= 0) return false;
            availableCopies--;
            return true;
        }
        
        public boolean returnBook() {
            if (availableCopies >= totalCopies) return false;
            availableCopies++;
            return true;
        }
        
        public boolean isAvailable()   { return availableCopies > 0; }
        public int getId()             { return id; }
        public String getIsbn()        { return isbn; }
        public String getTitle()       { return title; }
        public String getAuthor()      { return author; }
        public int getYear()           { return year; }
        public String getGenre()       { return genre; }
        public int getTotalCopies()    { return totalCopies; }
        public int getAvailableCopies(){ return availableCopies; }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (!(obj instanceof Book)) return false;
            Book other = (Book) obj;
            return this.isbn.equals(other.isbn);
        }
        
        @Override
        public int hashCode() { return isbn.hashCode(); }
        
        @Override
        public String toString() {
            return String.format("[%03d] \"%s\" by %s (%d) - %s | %d/%d available",
                                 id, title, author, year, genre,
                                 availableCopies, totalCopies);
        }
    }
    
    // ═══════════════════════════════════════════
    // MEMBER CLASS
    // ═══════════════════════════════════════════
    
    static class Member {
        
        private static int idCounter = 1000;
        static int totalMembers = 0;
        
        private final int memberId;
        private String name;
        private String email;
        private List<Book> borrowedBooks;
        private int totalBooksEverBorrowed;
        private boolean isActive;
        
        public static final int MAX_BORROW_LIMIT = 3;
        
        public Member(String name, String email) {
            if (name == null || name.isBlank())
                throw new IllegalArgumentException("Name required");
            if (email == null || !email.contains("@"))
                throw new IllegalArgumentException("Valid email required");
            
            this.memberId    = idCounter++;
            this.name        = name.trim();
            this.email       = email.trim().toLowerCase();
            this.borrowedBooks = new ArrayList<>();
            this.totalBooksEverBorrowed = 0;
            this.isActive    = true;
            totalMembers++;
        }
        
        public boolean canBorrow() {
            return isActive && borrowedBooks.size() < MAX_BORROW_LIMIT;
        }
        
        public boolean borrow(Book book) {
            if (!isActive) {
                System.out.println("❌ " + name + ": account inactive");
                return false;
            }
            if (!canBorrow()) {
                System.out.printf("❌ %s: borrow limit reached (%d/%d)%n",
                                  name, borrowedBooks.size(), MAX_BORROW_LIMIT);
                return false;
            }
            if (borrowedBooks.contains(book)) {
                System.out.println("❌ " + name + ": already has this book");
                return false;
            }
            if (!book.checkout()) {
                System.out.println("❌ No copies available: " + book.getTitle());
                return false;
            }
            borrowedBooks.add(book);
            totalBooksEverBorrowed++;
            System.out.printf("✅ %s borrowed \"%s\"%n", name, book.getTitle());
            return true;
        }
        
        public boolean returnBook(Book book) {
            if (!borrowedBooks.contains(book)) {
                System.out.println("❌ " + name + " didn't borrow: " + book.getTitle());
                return false;
            }
            borrowedBooks.remove(book);
            book.returnBook();
            System.out.printf("✅ %s returned \"%s\"%n", name, book.getTitle());
            return true;
        }
        
        // Getters
        public int getMemberId()          { return memberId; }
        public String getName()           { return name; }
        public String getEmail()          { return email; }
        public List<Book> getBorrowed()   { return new ArrayList<>(borrowedBooks); }
        public int getTotalBorrowed()     { return totalBooksEverBorrowed; }
        public boolean isActive()         { return isActive; }
        public void setActive(boolean a)  { isActive = a; }
        
        @Override
        public String toString() {
            return String.format("Member{id=%d, name='%s', borrowed=%d/%d, active=%b}",
                                 memberId, name, borrowedBooks.size(),
                                 MAX_BORROW_LIMIT, isActive);
        }
    }
    
    // ═══════════════════════════════════════════
    // LIBRARY CLASS (orchestrator)
    // ═══════════════════════════════════════════
    
    static class Library {
        
        private final String libraryName;
        private final List<Book> books;
        private final List<Member> members;
        
        public Library(String name) {
            this.libraryName = name;
            this.books       = new ArrayList<>();
            this.members     = new ArrayList<>();
        }
        
        public void addBook(Book book)     { books.add(book); }
        public void registerMember(Member m) { members.add(m); }
        
        public Book findBookByTitle(String title) {
            for (Book book : books) {
                if (book.getTitle().equalsIgnoreCase(title)) return book;
            }
            return null;
        }
        
        public Book findBookByIsbn(String isbn) {
            for (Book book : books) {
                if (book.getIsbn().equals(isbn)) return book;
            }
            return null;
        }
        
        public Member findMemberByName(String name) {
            for (Member m : members) {
                if (m.getName().equalsIgnoreCase(name)) return m;
            }
            return null;
        }
        
        public List<Book> getAvailableBooks() {
            List<Book> available = new ArrayList<>();
            for (Book book : books) {
                if (book.isAvailable()) available.add(book);
            }
            return available;
        }
        
        public List<Book> getBooksByGenre(String genre) {
            List<Book> result = new ArrayList<>();
            for (Book book : books) {
                if (book.getGenre().equalsIgnoreCase(genre)) result.add(book);
            }
            return result;
        }
        
        public void printCatalog() {
            System.out.println("\n╔═══════════════════════════════════════════════════╗");
            System.out.println("║        " + libraryName + " — CATALOG        ║");
            System.out.println("╠═══════════════════════════════════════════════════╣");
            for (Book book : books) {
                String avail = book.isAvailable() ? "✅" : "❌";
                System.out.printf("║ %s %s%n", avail, book);
            }
            System.out.println("╠═══════════════════════════════════════════════════╣");
            System.out.printf( "║ Total: %d books | Available: %d%n",
                               books.size(), getAvailableBooks().size());
            System.out.println("╚═══════════════════════════════════════════════════╝");
        }
        
        public void printMemberList() {
            System.out.println("\n╔═══════════════════════════════════════════════════╗");
            System.out.println("║              MEMBER DIRECTORY                     ║");
            System.out.println("╠═══════════════════════════════════════════════════╣");
            for (Member m : members) {
                System.out.println("║ " + m);
            }
            System.out.printf( "║ Total members: %d%n", Member.totalMembers);
            System.out.println("╚═══════════════════════════════════════════════════╝");
        }
        
        public void printStats() {
            int totalBooks   = 0, availBooks = 0;
            for (Book b : books) {
                totalBooks   += b.getTotalCopies();
                availBooks   += b.getAvailableCopies();
            }
            int borrowed = totalBooks - availBooks;
            
            System.out.println("\n=== LIBRARY STATISTICS ===");
            System.out.println("Library    : " + libraryName);
            System.out.printf( "Book titles: %d%n", books.size());
            System.out.printf( "Total copies: %d%n", totalBooks);
            System.out.printf( "Borrowed   : %d%n", borrowed);
            System.out.printf( "Available  : %d%n", availBooks);
            System.out.printf( "Members    : %d%n", members.size());
        }
    }
    
    // ═══════════════════════════════════════════
    // MAIN — DEMONSTRATE THE SYSTEM
    // ═══════════════════════════════════════════
    
    public static void main(String[] args) {
        
        Library library = new Library("Dhaka Public Library");
        
        // ── Add books ──
        library.addBook(new Book("978-0-13-110362-7", "The C Programming Language",
                                  "Kernighan & Ritchie", 1988, "Programming", 3));
        library.addBook(new Book("978-0-596-00712-6", "Clean Code",
                                  "Robert C. Martin", 2008, "Programming", 2));
        library.addBook(new Book("978-0-13-468599-1", "Effective Java",
                                  "Joshua Bloch", 2018, "Programming", 4));
        library.addBook(new Book("978-0-201-63361-0", "Design Patterns",
                                  "Gang of Four", 1994, "Programming", 2));
        library.addBook(new Book("978-0-00-655-4", "Ikigai",
                                  "Héctor García", 2016, "Self-Help", 5));
        
        // ── Register members ──
        Member rahim  = new Member("Rahim Ahmed", "rahim@test.com");
        Member karim  = new Member("Karim Hassan", "karim@test.com");
        Member hasan  = new Member("Hasan Ali", "hasan@test.com");
        
        library.registerMember(rahim);
        library.registerMember(karim);
        library.registerMember(hasan);
        
        // ── Display catalog ──
        library.printCatalog();
        library.printMemberList();
        
        // ── Transactions ──
        System.out.println("\n=== BORROWING TRANSACTIONS ===");
        
        Book cleanCode   = library.findBookByTitle("Clean Code");
        Book effectiveJava = library.findBookByTitle("Effective Java");
        Book designPatterns = library.findBookByTitle("Design Patterns");
        Book cProgramming = library.findBookByTitle("The C Programming Language");
        
        rahim.borrow(cleanCode);
        rahim.borrow(effectiveJava);
        rahim.borrow(designPatterns);
        rahim.borrow(cProgramming);   // will fail — limit reached
        
        karim.borrow(cleanCode);
        karim.borrow(cleanCode);      // will fail — already borrowed (by karim)
        karim.borrow(cProgramming);
        
        hasan.borrow(effectiveJava);
        
        // ── View after borrowing ──
        library.printCatalog();
        
        // ── Returns ──
        System.out.println("\n=== RETURNS ===");
        rahim.returnBook(cleanCode);
        karim.borrow(cleanCode);      // now available again
        
        // ── Try to borrow non-existent ──
        System.out.println("\n=== EDGE CASES ===");
        Book notExist = library.findBookByTitle("Python Cookbook");
        if (notExist == null) {
            System.out.println("Book not found in library.");
        }
        
        // ── Final stats ──
        library.printStats();
        
        System.out.println("\n=== MEMBER DETAILS ===");
        System.out.printf("Rahim  : %d total borrowed%n", rahim.getTotalBorrowed());
        System.out.printf("Karim  : %d total borrowed%n", karim.getTotalBorrowed());
        System.out.printf("Hasan  : %d total borrowed%n", hasan.getTotalBorrowed());
        System.out.println("\nRahim's current books:");
        rahim.getBorrowed().forEach(b -> System.out.println("  - " + b.getTitle()));
    }
}
```

---

## Exercises

```text
EXERCISE 1: Student Grade Tracker
  Create StudentTracker.java
  Design a Student class with:
  Fields: id (auto-generated), name, email, year, major
  Fields: int[] scores (up to 10 assignments)
  Methods:
  - addScore(double score) → validate 0-100, add to array
  - getAverage() → calculate average of added scores
  - getHighest() / getLowest()
  - getGrade() → A/B/C/D/F based on average
  - getGpaEquivalent() → 4.0 scale
  - toString() → formatted summary
  - equals() + hashCode() → based on email
  Static: totalStudents counter, class average across all students
  Test with 5 students, various scores.

EXERCISE 2: E-Commerce Product
  Create ProductCatalog.java
  Design a Product class with:
  Fields: id, sku, name, description, price, discountRate,
          stockQuantity, category, isActive
  Constructors: minimal (name, price) and full
  Methods:
  - getDiscountedPrice()
  - applyDiscount(double rate) → validate 0-1
  - addStock(int qty) / removeStock(int qty)
  - isAvailable() → stock > 0 AND isActive
  - getTotalValue() → price * stock
  - toString() / equals() / hashCode()
  Static: totalProducts, total inventory value
  Build a "catalog" with 5 products.
  Print: all available products, products by category,
         total catalog value, most valuable product.

EXERCISE 3: Constructor Overloading Practice
  Create VehicleRegistrar.java
  Design Vehicle class with constructors:
  - Vehicle(String brand) → only brand, rest defaults
  - Vehicle(String brand, String model, int year)
  - Vehicle(String brand, String model, int year, String color, double price)
  Use this() for constructor chaining.
  Each constructor validates its inputs.
  Show how different constructors create different levels of detail.

EXERCISE 4: Static Tracker
  Create CompanyTracker.java
  Design Employee class:
  Static: totalEmployees, totalSalaryBill, nextEmployeeId
  Instance: id, name, department, salary, isActive
  When an employee is created: increment totalEmployees, add to totalSalaryBill
  When deactivated: subtract from totalSalaryBill
  When salary changes: update totalSalaryBill accordingly
  Methods: promote(double newSalary), deactivate(), getAnnualSalary()
  Create 6 employees, change some salaries, deactivate one.
  Print running statistics after each operation.

EXERCISE 5: Design Your Own
  Create a class that models something from YOUR life:
  Ideas: CourseSchedule, GPACalculator, ExamTracker, ProjectTracker
  Must include:
  - At least 5 fields (mix of types)
  - 2+ constructors with chaining
  - 5+ instance methods with real logic
  - 2+ static fields/methods
  - Proper equals(), hashCode(), toString()
  - Input validation in constructors and methods
  Push to GitHub: "feat: OOP library system and student tracker"
```

---

## Common Mistakes

```text
MISTAKE 1: Forgetting 'this.' when parameter shadows field
  public Account(String name, double balance) {
      name = name;       // assigns param to itself! field unchanged!
      balance = balance; // same bug!
  }
  Fix: this.name = name; this.balance = balance;

MISTAKE 2: Calling instance method from static context
  public static void main(String[] args) {
      displayInfo(); // COMPILE ERROR if displayInfo() is instance method
  }
  Fix: create an object first: new MyClass().displayInfo();
  OR make displayInfo() static.

MISTAKE 3: Not overriding equals() and hashCode() together
  Override equals() without hashCode() → HashSet/HashMap breaks!
  Two "equal" objects with different hashCodes → stored as duplicates.
  Always override BOTH or NEITHER.

MISTAKE 4: Returning mutable collection directly from getter
  public List<Book> getBorrowedBooks() {
      return borrowedBooks; // caller can modify the internal list!
  }
  Fix: return new ArrayList<>(borrowedBooks); // defensive copy

MISTAKE 5: Using default toString() and wondering about output
  System.out.println(myObject); // "ClassName@7ef88735" — useless!
  Fix: always override toString() in your classes.

MISTAKE 6: Calling this() not as first statement
  public MyClass(String name) {
      System.out.println("hi"); // COMPILE ERROR: this() must be first!
      this();
  }
  Fix: this() must be the VERY FIRST statement.

MISTAKE 7: Accessing static member through object reference
  BankAccount acc = new BankAccount(...);
  acc.totalAccounts; // works but MISLEADING — looks like instance field
  Fix: BankAccount.totalAccounts; // always use class name for static

MISTAKE 8: Modifying final field after construction
  private final String id = "123";
  id = "456"; // COMPILE ERROR: final field cannot be reassigned
  // This is actually a FEATURE — prevents accidental ID changes

MISTAKE 9: Constructor does too much work
  public User(String name) {
      this.name = name;
      this.id = database.getNextId(); // database call in constructor!
      this.sendWelcomeEmail();        // side effect in constructor!
  }
  Fix: constructors should set fields. Heavy work goes in methods.

MISTAKE 10: Comparing objects with ==
  User u1 = new User("Rahim");
  User u2 = new User("Rahim");
  u1 == u2 → false (different objects)
  Fix: implement equals(): u1.equals(u2) → true (same content)
```

---

## Interview Questions

**Q: What is the difference between a class and an object?**
A: A class is a blueprint or template that defines the structure and behavior of objects — what fields (data) they have and what methods (behaviors) they can perform. An object is an instance of a class — an actual thing created from that blueprint. A class is defined once; you can create unlimited objects from it. Analogy: class = architectural blueprint, objects = the actual houses built from that blueprint. Each object has its own copy of instance fields but shares the class's method implementations.

**Q: What is a constructor? How does it differ from a method?**
A: A constructor is special code that runs when you create an object with 'new'. It shares the class name, has no return type (not even void), and cannot be called explicitly after object creation. Methods have their own names, explicit return types, and can be called any number of times. Constructors are for initialization — setting up the object in a valid state. If you write no constructor, Java provides a default no-arg one. Once you write any constructor, Java no longer provides the default.

**Q: What is the difference between static and instance members?**
A: Instance members (fields and methods without 'static') belong to each individual object. Every object has its own copy of instance fields. Instance methods have access to 'this' and can use both instance and static members. Static members belong to the CLASS itself. There is ONE copy shared by all objects. Static methods have no 'this' — they cannot access instance members directly. Static is used for: counters tracking all objects, constants, utility methods that don't need object state. Example: instance field = each account's balance. Static field = total number of accounts.

**Q: What is 'this' keyword and what are its uses?**
A: 'this' is a reference to the current object — the object on which the current method or constructor is executing. Main uses: 1) Disambiguate when a parameter shadows a field: this.name = name. 2) Call another constructor from a constructor: this() or this(args) — must be first statement. 3) Return the current object for method chaining: return this. 4) Pass the current object to another method: doSomething(this). You cannot use 'this' in static methods because static context has no associated object.

**Q: Why should you override equals() and hashCode() together?**
A: The Java contract requires: if a.equals(b) is true, then a.hashCode() must equal b.hashCode(). HashSet and HashMap use hashCode first to find the "bucket", then use equals to confirm equality. If you override equals() without hashCode(), two "logically equal" objects get different hash codes, land in different buckets, and the Set/Map treats them as different objects — so you can store duplicates in a HashSet or lose keys in a HashMap. Always override both together, or use Objects.hash(field1, field2) and Objects.equals() for convenience.

**Q: What is constructor chaining with this()?**
A: Calling this() inside a constructor invokes another constructor of the same class. This eliminates code duplication when multiple constructors share common initialization logic. this() must be the first statement in the constructor. Pattern: a minimal constructor calls a fuller one with defaults, which calls the most complete one. The most complete constructor does the actual work. Example: this("Unknown", 0) calls the two-parameter constructor from a no-arg constructor, reusing the initialization logic rather than duplicating it.

---

## Key Takeaways

```text
1. CLASS = blueprint. OBJECT = instance built from it.
   One class → unlimited objects.
   Each object has its OWN copy of instance fields.

2. FIELDS define what an object HAS.
   METHODS define what an object CAN DO.
   Both together model real-world things.

3. CONSTRUCTOR initializes an object when 'new' is used.
   Same name as class. No return type.
   If you write any constructor → no automatic default.
   Use this() for constructor chaining (must be first line).

4. 'this' = reference to current object.
   Use to: disambiguate field/param names, chain constructors,
   return current object, pass self to other methods.

5. INSTANCE members (no static):
   Belong to each object. Each object has own copy.
   Accessed via: object.field, object.method()

6. STATIC members (with static):
   Belong to the class. ONE copy shared by all objects.
   Accessed via: ClassName.field, ClassName.method()
   Cannot access instance members in static methods.

7. Always override toString() for meaningful printing.
   Default: "ClassName@hashCode" — useless.
   Custom: show the actual field values.

8. Override equals() AND hashCode() together. Always.
   equals() alone breaks HashSet and HashMap behavior.
   Two equal objects MUST have the same hashCode.

9. NullPointerException comes from calling methods
   on a null reference. Always check: if (ref != null)
   before using a reference that might be null.

10. Object reference assignment copies the ADDRESS.
    User a = b; means a and b point to the SAME object.
    Changing via 'a' affects 'b' too — they're the same thing.
    This catches many beginners off-guard.
```

---