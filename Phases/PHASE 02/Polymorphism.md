**Phase:** Level 2 - OOP
**Date Studied:**

---
## What Problem Does This Solve?

```text
You just learned inheritance. Now look at this common problem:

You have a notification system that sends different types
of notifications: Email, SMS, Push, WhatsApp.

Without polymorphism:

void sendAll(List<Object> notifications) {
    for (Object n : notifications) {
        if (n instanceof EmailNotification) {
            ((EmailNotification) n).sendEmail();
        } else if (n instanceof SmsNotification) {
            ((SmsNotification) n).sendSms();
        } else if (n instanceof PushNotification) {
            ((PushNotification) n).sendPush();
        } else if (n instanceof WhatsAppNotification) {
            ((WhatsAppNotification) n).sendWhatsApp();
        }
        // Adding a new type? Edit THIS code. And every place like it.
        // Forget one? Bug. Always.
    }
}

With polymorphism:

void sendAll(List<Notification> notifications) {
    for (Notification n : notifications) {
        n.send();  // ONE LINE. Works for ALL types.
        // Adding a new type? Just create a new class.
        // This code NEVER changes.
    }
}

Polymorphism means: "one interface, many implementations."
The SAME method call (n.send()) behaves DIFFERENTLY
depending on the ACTUAL type of the object at runtime.

This is the Open/Closed Principle in action:
  Open for extension (add new types)
  Closed for modification (don't touch existing code)

This is how Spring Boot routes HTTP requests,
how Java collections work, how JPA loads entities,
how design patterns are implemented.
Polymorphism is everywhere in professional Java.
```

---

## 1. What Is Polymorphism?

```text
Polymorphism comes from Greek: "poly" (many) + "morphe" (form).
"Many forms."

An object can take many forms.
A method can behave differently on different objects.
The SAME code works with DIFFERENT types.

Java has TWO types of polymorphism:

┌─────────────────────────────────────────────────────────────┐
│  TYPE 1: COMPILE-TIME POLYMORPHISM                          │
│  Also called: Static binding, Early binding                 │
│  Mechanism: Method Overloading                              │
│  Decision made: At COMPILE time (before program runs)       │
│  Based on: Method signature (name + parameters)             │
│                                                             │
│  Example: calculateArea(double r) vs                        │
│           calculateArea(double w, double h)                 │
│  Java knows at compile time which one to call.              │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  TYPE 2: RUNTIME POLYMORPHISM                               │
│  Also called: Dynamic binding, Late binding                 │
│  Mechanism: Method Overriding + Inheritance/Interface       │
│  Decision made: At RUNTIME (while program runs)             │
│  Based on: Actual object type (not reference type)          │
│                                                             │
│  Example: notification.send() — which implementation?       │
│  Java decides at runtime based on what 'notification'       │
│  actually IS (Email? SMS? Push?).                           │
│                                                             │
│  THIS is the powerful one. This is what people mean when    │
│  they say "polymorphism" in Java interviews.                │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Compile-Time Polymorphism (Method Overloading Review)

```java
public class CompileTimePolymorphism {
    
    // COMPILE-TIME: Java knows which method to call
    // at compile time based on the argument types.
    
    static class Printer {
        
        // Java picks based on argument type/count at compile time:
        
        public void print(int value) {
            System.out.println("Printing int: " + value);
        }
        
        public void print(double value) {
            System.out.println("Printing double: " + value);
        }
        
        public void print(String value) {
            System.out.println("Printing String: " + value);
        }
        
        public void print(String value, int copies) {
            System.out.println("Printing '" + value + "' × " + copies);
        }
        
        public void print(boolean value) {
            System.out.println("Printing boolean: " + value);
        }
    }
    
    // Another example: Logger with different signatures
    static class Logger {
        
        public static void log(String message) {
            System.out.println("[INFO] " + message);
        }
        
        public static void log(String message, Exception e) {
            System.out.println("[ERROR] " + message + ": " + e.getMessage());
        }
        
        public static void log(String level, String message) {
            System.out.println("[" + level + "] " + message);
        }
        
        public static void log(String message, Object... args) {
            // varargs: print message with formatted args
            System.out.printf("[INFO] " + message + "%n", args);
        }
    }
    
    public static void main(String[] args) {
        Printer p = new Printer();
        
        // Compiler sees the argument type and picks the right method:
        p.print(42);            // int version
        p.print(3.14);         // double version
        p.print("Hello");      // String version
        p.print("Hello", 3);   // String + int version
        p.print(true);         // boolean version
        
        System.out.println();
        
        // Logger overloads:
        Logger.log("Server started");
        Logger.log("Connection failed", new RuntimeException("Timeout"));
        Logger.log("DEBUG", "Processing request #123");
        Logger.log("User %s logged in from %s", "Rahim", "Dhaka");
        
        // ─────────────────────────────────────────
        // HOW JAVA RESOLVES OVERLOADS:
        // Step 1: Exact match
        // Step 2: Widening (int → long → double)
        // Step 3: Autoboxing (int → Integer)
        // Step 4: Varargs
        // ─────────────────────────────────────────
        
        // Widening example:
        byte b = 42;
        p.print(b);   // byte widens to int → calls print(int)
        
        float f = 3.14f;
        p.print(f);   // float widens to double → calls print(double)
        
        System.out.println("\nCompile-time polymorphism: Java " +
                           "decided which method to call BEFORE running.");
    }
}
```

---

## 3. Runtime Polymorphism — The Heart of OOP

```java
public class RuntimePolymorphism {
    
    // ─────────────────────────────────────────
    // THE FOUNDATION: Reference type vs Object type
    // ─────────────────────────────────────────
    //
    // Reference type: what the variable is declared as
    // Object type: what the variable actually CONTAINS at runtime
    //
    // Animal animal = new Dog("Rex");
    //         │               │
    //         └─ reference    └─ actual object
    //            type: Animal    type: Dog
    //
    // Method resolution uses the ACTUAL object type.
    // NOT the reference type.
    // ─────────────────────────────────────────
    
    static class Animal {
        protected String name;
        
        public Animal(String name) {
            this.name = name;
        }
        
        public void makeSound() {
            System.out.println(name + " makes a sound.");
        }
        
        public void move() {
            System.out.println(name + " moves.");
        }
        
        public String describe() {
            return "I am " + name + ", an Animal.";
        }
    }
    
    static class Dog extends Animal {
        public Dog(String name) { super(name); }
        
        @Override
        public void makeSound() {
            System.out.println(name + ": Woof! 🐶");
        }
        
        @Override
        public void move() {
            System.out.println(name + " runs on four legs. 🏃");
        }
        
        public void fetch() {
            System.out.println(name + " fetches! 🎾");
        }
    }
    
    static class Cat extends Animal {
        public Cat(String name) { super(name); }
        
        @Override
        public void makeSound() {
            System.out.println(name + ": Meow! 🐱");
        }
        
        @Override
        public void move() {
            System.out.println(name + " slinks gracefully. 🐈");
        }
        
        public void purr() {
            System.out.println(name + " purrs...");
        }
    }
    
    static class Bird extends Animal {
        public Bird(String name) { super(name); }
        
        @Override
        public void makeSound() {
            System.out.println(name + ": Tweet! Tweet! 🐦");
        }
        
        @Override
        public void move() {
            System.out.println(name + " flies through the air! ✈️");
        }
        
        @Override
        public String describe() {
            return super.describe() + " (and I can fly!)";
        }
    }
    
    public static void main(String[] args) {
        
        System.out.println("=== Variable Types ===");
        
        // Reference type = Animal, but actual object = Dog
        Animal a1 = new Dog("Rex");
        // Reference type = Animal, but actual object = Cat
        Animal a2 = new Cat("Whiskers");
        // Reference type = Animal, but actual object = Bird
        Animal a3 = new Bird("Tweety");
        // Reference type = Animal, actual object = Animal (no override)
        Animal a4 = new Animal("Generic");
        
        // ─────────────────────────────────────────
        // DYNAMIC DISPATCH — Java picks method at RUNTIME
        // ─────────────────────────────────────────
        
        System.out.println("\n=== makeSound() — Runtime Polymorphism ===");
        
        // Same call: a.makeSound()
        // Different behavior based on actual object:
        a1.makeSound(); // Calls Dog's makeSound()   → "Rex: Woof!"
        a2.makeSound(); // Calls Cat's makeSound()   → "Whiskers: Meow!"
        a3.makeSound(); // Calls Bird's makeSound()  → "Tweety: Tweet!"
        a4.makeSound(); // Calls Animal's makeSound() → "Generic makes a sound."
        
        // The reference type is Animal for all four.
        // But Java looks at the ACTUAL object and calls the right method.
        
        System.out.println("\n=== move() ===");
        a1.move(); // Dog's move()
        a2.move(); // Cat's move()
        a3.move(); // Bird's move()
        
        System.out.println("\n=== describe() — super + override ===");
        System.out.println(a1.describe()); // Animal's (Dog didn't override)
        System.out.println(a3.describe()); // Bird's overridden version
        
        System.out.println("\n=== The Power: ARRAY of mixed types ===");
        
        Animal[] zoo = {
            new Dog("Rex"),
            new Cat("Luna"),
            new Bird("Polly"),
            new Dog("Max"),
            new Cat("Mimi"),
            new Bird("Kiwi"),
            new Animal("Unknown")
        };
        
        System.out.println("All animals making sounds:");
        for (Animal animal : zoo) {
            animal.makeSound(); // Each makes its OWN sound — ONE LINE handles all!
        }
        
        System.out.println("\nAll animals moving:");
        for (Animal animal : zoo) {
            animal.move(); // Each moves its OWN way
        }
        
        System.out.println("\n=== Cannot call child-specific methods ===");
        
        // a1 is declared as Animal — can only call Animal methods:
        a1.makeSound(); // ✅ Animal has this
        // a1.fetch();  // ❌ COMPILE ERROR: Animal doesn't have fetch()
        
        // Must downcast to access Dog-specific methods:
        if (a1 instanceof Dog d) {
            d.fetch(); // ✅ Now we can call fetch()
        }
        
        System.out.println("\n=== Why this is powerful ===");
        System.out.println("The zoo array works with ANY Animal subtype.");
        System.out.println("Add 100 new animal types — the loop NEVER changes.");
    }
}
```

---

## 4. Polymorphism with Interfaces

```java
// Interfaces make polymorphism even more powerful.
// A class can implement MULTIPLE interfaces,
// and objects can be treated as different interface types.

public class InterfacePolymorphism {
    
    // ─────────────────────────────────────────
    // INTERFACES: Pure contracts — only behavior, no state
    // ─────────────────────────────────────────
    
    interface Printable {
        void print();
        
        default String getDocumentType() {
            return "Generic Document";
        }
    }
    
    interface Saveable {
        void save(String location);
        boolean load(String location);
    }
    
    interface Shareable {
        void share(String recipient);
        void shareWithAll(String[] recipients);
    }
    
    // ─────────────────────────────────────────
    // CLASSES IMPLEMENTING MULTIPLE INTERFACES
    // ─────────────────────────────────────────
    
    static class Report implements Printable, Saveable {
        private String title;
        private String content;
        
        public Report(String title, String content) {
            this.title   = title;
            this.content = content;
        }
        
        @Override
        public void print() {
            System.out.println("=== REPORT: " + title + " ===");
            System.out.println(content);
        }
        
        @Override
        public void save(String location) {
            System.out.println("Report '" + title + "' saved to: " + location);
        }
        
        @Override
        public boolean load(String location) {
            System.out.println("Loading report from: " + location);
            return true;
        }
        
        @Override
        public String getDocumentType() {
            return "Business Report";
        }
    }
    
    static class Invoice implements Printable, Saveable, Shareable {
        private String invoiceNumber;
        private double amount;
        
        public Invoice(String number, double amount) {
            this.invoiceNumber = number;
            this.amount        = amount;
        }
        
        @Override
        public void print() {
            System.out.printf("INVOICE #%s — Amount: ৳%.2f%n",
                              invoiceNumber, amount);
        }
        
        @Override
        public void save(String location) {
            System.out.println("Invoice #" + invoiceNumber + " saved to: " + location);
        }
        
        @Override
        public boolean load(String location) {
            return true;
        }
        
        @Override
        public void share(String recipient) {
            System.out.println("Invoice #" + invoiceNumber +
                               " shared with: " + recipient);
        }
        
        @Override
        public void shareWithAll(String[] recipients) {
            for (String r : recipients) {
                share(r);
            }
        }
        
        @Override
        public String getDocumentType() { return "Financial Invoice"; }
    }
    
    static class Photo implements Printable, Shareable {
        private String filename;
        private String resolution;
        
        public Photo(String filename, String resolution) {
            this.filename   = filename;
            this.resolution = resolution;
        }
        
        @Override
        public void print() {
            System.out.println("Printing photo: " + filename + " at " + resolution);
        }
        
        @Override
        public void share(String recipient) {
            System.out.println("Photo '" + filename + "' shared with " + recipient);
        }
        
        @Override
        public void shareWithAll(String[] recipients) {
            System.out.println("Sharing '" + filename + "' with " + recipients.length + " people");
        }
    }
    
    // ─────────────────────────────────────────
    // POLYMORPHIC METHODS USING INTERFACES
    // ─────────────────────────────────────────
    
    static void printAll(Printable[] documents) {
        System.out.println("\n--- Printing All Documents ---");
        for (Printable doc : documents) {
            System.out.println("Type: " + doc.getDocumentType());
            doc.print();
            System.out.println();
        }
    }
    
    static void saveAll(Saveable[] documents, String directory) {
        System.out.println("--- Saving All Saveable Documents ---");
        for (Saveable doc : documents) {
            doc.save(directory + "/" + System.currentTimeMillis());
        }
    }
    
    public static void main(String[] args) {
        Report  report  = new Report("Q4 Revenue", "Total: ৳10M, Growth: 15%");
        Invoice invoice = new Invoice("INV-001", 15000.0);
        Photo   photo   = new Photo("product.jpg", "4K");
        
        // ─────────────────────────────────────────
        // SAME OBJECT, DIFFERENT INTERFACE VIEWS
        // ─────────────────────────────────────────
        
        // Invoice can be viewed as any interface it implements:
        Printable  p1 = invoice; // Invoice viewed as Printable
        Saveable   s1 = invoice; // Invoice viewed as Saveable
        Shareable  sh = invoice; // Invoice viewed as Shareable
        // All point to the SAME invoice object!
        
        System.out.println("Same object?");
        System.out.println(p1 == s1);  // true — same object
        System.out.println(s1 == sh);  // true — same object
        
        // ─────────────────────────────────────────
        // POLYMORPHIC COLLECTIONS
        // ─────────────────────────────────────────
        
        // All Printable things in one array:
        Printable[] printables = { report, invoice, photo };
        printAll(printables); // ONE method handles ALL types
        
        // All Saveable things:
        Saveable[] saveables = { report, invoice };
        saveAll(saveables, "/documents");
        
        // Share invoice with multiple recipients:
        invoice.shareWithAll(new String[]{"boss@company.com",
                                           "accounting@company.com",
                                           "client@client.com"});
        
        // ─────────────────────────────────────────
        // INTERFACE AS METHOD PARAMETER (very common in Spring)
        // ─────────────────────────────────────────
        
        // Method accepts any Printable — maximum flexibility:
        processPrintable(new Report("Monthly", "content..."));
        processPrintable(new Invoice("INV-002", 5000.0));
        processPrintable(new Photo("team.jpg", "HD"));
        
        // Method accepts any Saveable:
        processAndSave(new Report("Annual", "content..."), "/backup");
        processAndSave(new Invoice("INV-003", 8000.0), "/invoices");
        // Photo is NOT Saveable — cannot pass photo here
    }
    
    static void processPrintable(Printable item) {
        System.out.println("\nProcessing: " + item.getDocumentType());
        item.print(); // polymorphic call
    }
    
    static void processAndSave(Saveable item, String path) {
        item.save(path);
    }
}
```

---

## 5. Virtual Method Table (vtable) — How JVM Does It

```text
This is what actually happens inside the JVM.
You don't need to code this — just understand the mechanism.

When the JVM loads a class, it builds a VIRTUAL METHOD TABLE (vtable)
for each class. This table maps method names to implementations.

Example:

Animal vtable:
  makeSound → Animal.makeSound()
  move      → Animal.move()
  describe  → Animal.describe()

Dog vtable:
  makeSound → Dog.makeSound()    ← overridden! Points to Dog's version
  move      → Dog.move()         ← overridden!
  describe  → Animal.describe()  ← not overridden, inherited from Animal
  fetch     → Dog.fetch()        ← Dog-specific method

Cat vtable:
  makeSound → Cat.makeSound()    ← overridden!
  move      → Cat.move()         ← overridden!
  describe  → Animal.describe()  ← inherited
  purr      → Cat.purr()         ← Cat-specific

When you write:
  Animal a = new Dog("Rex");
  a.makeSound();

JVM does:
  1. Look at 'a' — it's an Animal reference
  2. Look at the ACTUAL object on the heap — it's a Dog
  3. Look up Dog's vtable for makeSound
  4. Find Dog.makeSound() — call it!

This lookup happens at RUNTIME — hence "runtime polymorphism".
This is why it's also called "dynamic dispatch" or "late binding".
The binding of the method call to the implementation is delayed
until runtime.

Performance note:
  vtable lookup adds a tiny overhead vs direct call.
  JIT compiler optimizes frequently called virtual methods
  to direct calls (devirtualization) — so in practice,
  the overhead is minimal in hot code.
```

---

## 6. Polymorphism in Real Spring Boot Code

```java
// These patterns appear CONSTANTLY in Spring Boot.
// Understanding polymorphism makes Spring Boot obvious.

public class SpringBootPolymorphism {
    
    // ─────────────────────────────────────────
    // PATTERN 1: Repository Pattern
    // Spring Data uses polymorphism heavily
    // ─────────────────────────────────────────
    
    interface UserRepository {
        void save(User user);
        User findById(Long id);
        java.util.List<User> findAll();
        void delete(Long id);
    }
    
    // Different implementations (Spring creates these):
    static class PostgresUserRepository implements UserRepository {
        @Override public void save(User user) {
            System.out.println("Saving to PostgreSQL: " + user.getName());
        }
        @Override public User findById(Long id) {
            return new User("DB User " + id, "db@example.com");
        }
        @Override public java.util.List<User> findAll() {
            return java.util.List.of(new User("User1", "u1@test.com"));
        }
        @Override public void delete(Long id) {
            System.out.println("Deleted user " + id + " from PostgreSQL");
        }
    }
    
    static class InMemoryUserRepository implements UserRepository {
        private final java.util.Map<Long, User> store = new java.util.HashMap<>();
        private long nextId = 1;
        
        @Override public void save(User user) {
            store.put(nextId++, user);
            System.out.println("Saved in memory: " + user.getName());
        }
        @Override public User findById(Long id) {
            return store.get(id);
        }
        @Override public java.util.List<User> findAll() {
            return new java.util.ArrayList<>(store.values());
        }
        @Override public void delete(Long id) {
            store.remove(id);
        }
    }
    
    // UserService doesn't know (or care) which repository it uses:
    static class UserService {
        private final UserRepository repository; // interface type!
        
        // Dependency injection — Spring provides the implementation:
        public UserService(UserRepository repository) {
            this.repository = repository;
        }
        
        public void registerUser(String name, String email) {
            User user = new User(name, email);
            repository.save(user); // polymorphic! which impl? doesn't matter
            System.out.println("Registration complete for: " + name);
        }
        
        public User getUser(Long id) {
            User user = repository.findById(id);
            if (user == null) throw new RuntimeException("User not found: " + id);
            return user;
        }
    }
    
    // ─────────────────────────────────────────
    // PATTERN 2: Strategy Pattern
    // Different algorithms behind same interface
    // ─────────────────────────────────────────
    
    interface PricingStrategy {
        double calculatePrice(double basePrice, String customerType, int quantity);
    }
    
    static class StandardPricing implements PricingStrategy {
        @Override
        public double calculatePrice(double basePrice, String type, int qty) {
            return basePrice * qty;
        }
    }
    
    static class PremiumPricing implements PricingStrategy {
        @Override
        public double calculatePrice(double basePrice, String type, int qty) {
            double discount = 0.15; // 15% off for premium
            return basePrice * qty * (1 - discount);
        }
    }
    
    static class BulkPricing implements PricingStrategy {
        @Override
        public double calculatePrice(double basePrice, String type, int qty) {
            double discount = qty >= 100 ? 0.20 : qty >= 50 ? 0.10 : 0.0;
            return basePrice * qty * (1 - discount);
        }
    }
    
    static class SeasonalPricing implements PricingStrategy {
        private double seasonalMultiplier;
        
        public SeasonalPricing(double multiplier) {
            this.seasonalMultiplier = multiplier;
        }
        
        @Override
        public double calculatePrice(double base, String type, int qty) {
            return base * qty * seasonalMultiplier;
        }
    }
    
    static class PricingService {
        // Switch strategy at runtime:
        private PricingStrategy strategy;
        
        public PricingService(PricingStrategy strategy) {
            this.strategy = strategy;
        }
        
        public void setStrategy(PricingStrategy strategy) {
            this.strategy = strategy; // swap without changing logic
        }
        
        public double getPrice(double basePrice, String customerType, int qty) {
            return strategy.calculatePrice(basePrice, customerType, qty);
        }
    }
    
    // ─────────────────────────────────────────
    // PATTERN 3: Event handling
    // ─────────────────────────────────────────
    
    interface EventHandler<T> {
        void handle(T event);
    }
    
    static class UserCreatedEvent {
        public final String userId;
        public final String email;
        public UserCreatedEvent(String userId, String email) {
            this.userId = userId;
            this.email  = email;
        }
    }
    
    static class SendWelcomeEmailHandler implements EventHandler<UserCreatedEvent> {
        @Override
        public void handle(UserCreatedEvent event) {
            System.out.println("📧 Welcome email sent to: " + event.email);
        }
    }
    
    static class CreateDefaultSettingsHandler implements EventHandler<UserCreatedEvent> {
        @Override
        public void handle(UserCreatedEvent event) {
            System.out.println("⚙️  Default settings created for: " + event.userId);
        }
    }
    
    static class AuditLogHandler implements EventHandler<UserCreatedEvent> {
        @Override
        public void handle(UserCreatedEvent event) {
            System.out.println("📋 Audit: User " + event.userId + " created at "
                               + java.time.LocalDateTime.now());
        }
    }
    
    static class EventBus {
        private final java.util.List<EventHandler<UserCreatedEvent>> handlers =
            new java.util.ArrayList<>();
        
        public void subscribe(EventHandler<UserCreatedEvent> handler) {
            handlers.add(handler);
        }
        
        public void publish(UserCreatedEvent event) {
            System.out.println("\n=== Publishing UserCreatedEvent ===");
            for (EventHandler<UserCreatedEvent> h : handlers) {
                h.handle(event); // polymorphic! each handler does its thing
            }
        }
    }
    
    // Simple User class for examples:
    static class User {
        private final String name;
        private final String email;
        public User(String name, String email) {
            this.name = name; this.email = email;
        }
        public String getName()  { return name; }
        public String getEmail() { return email; }
    }
    
    public static void main(String[] args) {
        
        System.out.println("=== Repository Pattern ===");
        
        // Swap implementations — UserService doesn't change:
        UserRepository inMemory = new InMemoryUserRepository();
        UserService service1 = new UserService(inMemory);
        service1.registerUser("Rahim", "rahim@test.com");
        
        UserRepository postgres = new PostgresUserRepository();
        UserService service2 = new UserService(postgres);
        service2.registerUser("Karim", "karim@test.com");
        
        System.out.println("\n=== Strategy Pattern ===");
        
        PricingService pricing = new PricingService(new StandardPricing());
        System.out.printf("Standard (5 items @ ৳100): ৳%.2f%n",
                          pricing.getPrice(100, "REGULAR", 5));
        
        pricing.setStrategy(new PremiumPricing());
        System.out.printf("Premium  (5 items @ ৳100): ৳%.2f%n",
                          pricing.getPrice(100, "PREMIUM", 5));
        
        pricing.setStrategy(new BulkPricing());
        System.out.printf("Bulk 100 (100 items @ ৳100): ৳%.2f%n",
                          pricing.getPrice(100, "BULK", 100));
        
        pricing.setStrategy(new SeasonalPricing(1.20));
        System.out.printf("Seasonal (5 items @ ৳100): ৳%.2f%n",
                          pricing.getPrice(100, "REGULAR", 5));
        
        System.out.println("\n=== Event Bus Pattern ===");
        
        EventBus bus = new EventBus();
        bus.subscribe(new SendWelcomeEmailHandler());
        bus.subscribe(new CreateDefaultSettingsHandler());
        bus.subscribe(new AuditLogHandler());
        
        // Publishing one event triggers ALL handlers:
        bus.publish(new UserCreatedEvent("USER-001", "hasan@test.com"));
        // Output: 3 handlers all called with the same event — polymorphic!
    }
}
```

---

## 7. Polymorphism with Abstract Classes

```java
// Abstract classes provide a middle ground:
// - Can have abstract methods (must be overridden)
// - Can have concrete methods (may be overridden or inherited)
// - Cannot be instantiated directly

public class AbstractPolymorphism {
    
    abstract static class DatabaseConnector {
        // Common state:
        protected String host;
        protected int port;
        protected String database;
        protected boolean connected;
        
        public DatabaseConnector(String host, int port, String database) {
            this.host     = host;
            this.port     = port;
            this.database = database;
            this.connected = false;
        }
        
        // Template method (final — sequence cannot change):
        public final boolean connect() {
            System.out.println("Connecting to " + getDbType() +
                               " at " + host + ":" + port);
            
            if (!validateConnection()) {
                System.out.println("  ❌ Validation failed");
                return false;
            }
            
            if (!establishConnection()) {
                System.out.println("  ❌ Connection failed");
                return false;
            }
            
            connected = true;
            System.out.println("  ✅ Connected to " + database);
            onConnect();
            return true;
        }
        
        public final void disconnect() {
            if (!connected) return;
            closeConnection();
            connected = false;
            System.out.println("Disconnected from " + database);
        }
        
        // Abstract: subclasses MUST implement:
        protected abstract String getDbType();
        protected abstract boolean validateConnection();
        protected abstract boolean establishConnection();
        protected abstract void closeConnection();
        protected abstract java.util.List<java.util.Map<String, Object>>
                          query(String sql);
        
        // Optional override (default behavior provided):
        protected void onConnect() { }
        
        // Concrete method using abstract method (polymorphism!):
        public void printInfo() {
            System.out.println(getDbType() + " at " + host + ":" + port +
                               "/" + database + " [" +
                               (connected ? "CONNECTED" : "DISCONNECTED") + "]");
        }
    }
    
    static class PostgreSQLConnector extends DatabaseConnector {
        private String schema;
        
        public PostgreSQLConnector(String host, int port, String db, String schema) {
            super(host, port, db);
            this.schema = schema;
        }
        
        @Override
        protected String getDbType() { return "PostgreSQL"; }
        
        @Override
        protected boolean validateConnection() {
            return host != null && port == 5432; // PostgreSQL default port
        }
        
        @Override
        protected boolean establishConnection() {
            System.out.println("  Authenticating with PostgreSQL...");
            System.out.println("  Schema: " + schema);
            return true;
        }
        
        @Override
        protected void closeConnection() {
            System.out.println("  Closing PostgreSQL connection gracefully.");
        }
        
        @Override
        protected java.util.List<java.util.Map<String, Object>> query(String sql) {
            System.out.println("PostgreSQL executing: " + sql);
            return new java.util.ArrayList<>();
        }
        
        @Override
        protected void onConnect() {
            System.out.println("  Setting schema: " + schema);
        }
    }
    
    static class MongoDBConnector extends DatabaseConnector {
        private String collection;
        
        public MongoDBConnector(String host, int port, String db, String collection) {
            super(host, port, db);
            this.collection = collection;
        }
        
        @Override
        protected String getDbType() { return "MongoDB"; }
        
        @Override
        protected boolean validateConnection() {
            return host != null && port == 27017; // MongoDB default port
        }
        
        @Override
        protected boolean establishConnection() {
            System.out.println("  Connecting to MongoDB replica set...");
            return true;
        }
        
        @Override
        protected void closeConnection() {
            System.out.println("  Closing MongoDB connection pool.");
        }
        
        @Override
        protected java.util.List<java.util.Map<String, Object>> query(String filter) {
            System.out.println("MongoDB querying collection '" + collection +
                               "' with: " + filter);
            return new java.util.ArrayList<>();
        }
    }
    
    static class RedisConnector extends DatabaseConnector {
        private int db;
        
        public RedisConnector(String host, int port, int db) {
            super(host, port, "Redis");
            this.db = db;
        }
        
        @Override
        protected String getDbType() { return "Redis"; }
        
        @Override
        protected boolean validateConnection() {
            return host != null && port == 6379; // Redis default port
        }
        
        @Override
        protected boolean establishConnection() {
            System.out.println("  Connecting to Redis, DB #" + db);
            return true;
        }
        
        @Override
        protected void closeConnection() {
            System.out.println("  Closing Redis connection.");
        }
        
        @Override
        protected java.util.List<java.util.Map<String, Object>> query(String key) {
            System.out.println("Redis GET: " + key);
            return new java.util.ArrayList<>();
        }
    }
    
    public static void main(String[] args) {
        
        // DatabaseConnector db = new DatabaseConnector(...); // COMPILE ERROR: abstract!
        
        // All connectors treated as DatabaseConnector:
        DatabaseConnector[] connectors = {
            new PostgreSQLConnector("localhost", 5432, "myapp", "public"),
            new MongoDBConnector("localhost", 27017, "myapp", "users"),
            new RedisConnector("localhost", 6379, 0)
        };
        
        System.out.println("=== Connecting All Databases ===");
        for (DatabaseConnector conn : connectors) {
            conn.connect();    // template method — polymorphic internals
            conn.printInfo();  // uses getDbType() — polymorphic
            System.out.println();
        }
        
        System.out.println("=== Querying ===");
        for (DatabaseConnector conn : connectors) {
            if (conn.connected) {
                conn.query("SELECT * FROM users");
            }
        }
        
        System.out.println("\n=== Disconnecting ===");
        for (DatabaseConnector conn : connectors) {
            conn.disconnect(); // each disconnects its own way
        }
    }
}
```

---

## 8. Common Polymorphism Patterns

```java
// Patterns you will use every day in Spring Boot

public class PolymorphismPatterns {
    
    // ─────────────────────────────────────────
    // PATTERN 1: Null Object Pattern
    // Avoids null checks with polymorphism
    // ─────────────────────────────────────────
    
    interface Logger {
        void info(String message);
        void error(String message, Exception e);
        boolean isEnabled();
    }
    
    static class ConsoleLogger implements Logger {
        @Override public void info(String message) {
            System.out.println("[INFO] " + message);
        }
        @Override public void error(String message, Exception e) {
            System.out.println("[ERROR] " + message + ": " + e.getMessage());
        }
        @Override public boolean isEnabled() { return true; }
    }
    
    // Null Object: does nothing, but avoids NullPointerException
    static class NoOpLogger implements Logger {
        @Override public void info(String message)             { } // does nothing
        @Override public void error(String message, Exception e) { } // does nothing
        @Override public boolean isEnabled()                   { return false; }
    }
    
    static class Service {
        private final Logger logger;
        
        // Can inject real logger or NoOpLogger — service doesn't care:
        public Service(Logger logger) {
            this.logger = logger != null ? logger : new NoOpLogger();
        }
        
        public void doWork() {
            logger.info("Starting work..."); // never throws NPE — polymorphism!
            // ... do work ...
            logger.info("Work complete.");
        }
    }
    
    // ─────────────────────────────────────────
    // PATTERN 2: Decorator Pattern
    // Add behavior without changing the class
    // ─────────────────────────────────────────
    
    interface TextProcessor {
        String process(String text);
    }
    
    static class PlainTextProcessor implements TextProcessor {
        @Override
        public String process(String text) {
            return text;
        }
    }
    
    // Decorator wraps another TextProcessor:
    abstract static class TextProcessorDecorator implements TextProcessor {
        protected final TextProcessor wrapped;
        
        public TextProcessorDecorator(TextProcessor wrapped) {
            this.wrapped = wrapped;
        }
    }
    
    static class TrimDecorator extends TextProcessorDecorator {
        public TrimDecorator(TextProcessor wrapped) { super(wrapped); }
        
        @Override
        public String process(String text) {
            return wrapped.process(text).trim(); // delegate + add trim
        }
    }
    
    static class UpperCaseDecorator extends TextProcessorDecorator {
        public UpperCaseDecorator(TextProcessor wrapped) { super(wrapped); }
        
        @Override
        public String process(String text) {
            return wrapped.process(text).toUpperCase();
        }
    }
    
    static class SanitizeDecorator extends TextProcessorDecorator {
        public SanitizeDecorator(TextProcessor wrapped) { super(wrapped); }
        
        @Override
        public String process(String text) {
            return wrapped.process(text)
                          .replaceAll("[<>\"'&]", ""); // remove HTML special chars
        }
    }
    
    // ─────────────────────────────────────────
    // PATTERN 3: Chain of Responsibility
    // Used in Spring Security filter chain!
    // ─────────────────────────────────────────
    
    abstract static class RequestHandler {
        protected RequestHandler next;
        
        public RequestHandler setNext(RequestHandler next) {
            this.next = next;
            return next; // return next for fluent chaining
        }
        
        public abstract boolean handle(Request request);
        
        protected boolean passToNext(Request request) {
            if (next != null) return next.handle(request);
            System.out.println("  ✅ Request approved by all handlers");
            return true;
        }
    }
    
    static class Request {
        final String ip;
        final String token;
        final String path;
        final boolean hasValidBody;
        
        Request(String ip, String token, String path, boolean validBody) {
            this.ip = ip; this.token = token;
            this.path = path; this.hasValidBody = validBody;
        }
    }
    
    static class RateLimitHandler extends RequestHandler {
        private final java.util.Map<String, Integer> requestCounts = new java.util.HashMap<>();
        private final int maxRequests;
        
        public RateLimitHandler(int maxRequests) { this.maxRequests = maxRequests; }
        
        @Override
        public boolean handle(Request req) {
            int count = requestCounts.merge(req.ip, 1, Integer::sum);
            if (count > maxRequests) {
                System.out.println("  ❌ Rate limit exceeded for: " + req.ip);
                return false;
            }
            System.out.println("  ✅ Rate limit OK (" + count + "/" + maxRequests + ")");
            return passToNext(req);
        }
    }
    
    static class AuthenticationHandler extends RequestHandler {
        @Override
        public boolean handle(Request req) {
            if (req.token == null || !req.token.startsWith("Bearer ")) {
                System.out.println("  ❌ Authentication failed: no valid token");
                return false;
            }
            System.out.println("  ✅ Authentication passed");
            return passToNext(req);
        }
    }
    
    static class ValidationHandler extends RequestHandler {
        @Override
        public boolean handle(Request req) {
            if (!req.hasValidBody) {
                System.out.println("  ❌ Validation failed: invalid request body");
                return false;
            }
            System.out.println("  ✅ Validation passed");
            return passToNext(req);
        }
    }
    
    static class AuthorizationHandler extends RequestHandler {
        private final java.util.List<String> allowedPaths =
            java.util.List.of("/api/users", "/api/products", "/api/orders");
        
        @Override
        public boolean handle(Request req) {
            boolean allowed = allowedPaths.stream()
                .anyMatch(p -> req.path.startsWith(p));
            if (!allowed) {
                System.out.println("  ❌ Authorization failed: path not allowed: " + req.path);
                return false;
            }
            System.out.println("  ✅ Authorization passed");
            return passToNext(req);
        }
    }
    
    public static void main(String[] args) {
        
        System.out.println("=== Null Object Pattern ===");
        Service withLogger = new Service(new ConsoleLogger());
        Service withoutLogger = new Service(null); // NoOpLogger used automatically
        withLogger.doWork();
        withoutLogger.doWork(); // no output — but no NPE either!
        
        System.out.println("\n=== Decorator Pattern ===");
        
        // Build processing pipeline by wrapping:
        TextProcessor processor = new SanitizeDecorator(    // outer
                                  new TrimDecorator(          // middle
                                  new UpperCaseDecorator(     // inner
                                  new PlainTextProcessor()))   // core
                                  );
        
        String raw = "  hello <world> & java  ";
        System.out.println("Input:  '" + raw + "'");
        System.out.println("Output: '" + processor.process(raw) + "'");
        // Input:  '  hello <world> & java  '
        // Output: 'HELLO  JAVA' (trimmed, uppercased, sanitized)
        
        System.out.println("\n=== Chain of Responsibility (Spring Security-like) ===");
        
        // Build the filter chain:
        RequestHandler chain = new RateLimitHandler(3);
        chain.setNext(new AuthenticationHandler())
             .setNext(new ValidationHandler())
             .setNext(new AuthorizationHandler());
        
        // Valid request:
        System.out.println("--- Valid Request ---");
        chain.handle(new Request("192.168.1.1", "Bearer token123",
                                  "/api/users/123", true));
        
        // No token:
        System.out.println("\n--- No Token ---");
        chain.handle(new Request("192.168.1.2", null,
                                  "/api/products", true));
        
        // Invalid path:
        System.out.println("\n--- Invalid Path ---");
        chain.handle(new Request("192.168.1.3", "Bearer token456",
                                  "/admin/secret", true));
        
        // Rate limited:
        System.out.println("\n--- Rate Limited (4th request from same IP) ---");
        for (int i = 0; i < 4; i++) {
            System.out.println("Request " + (i+1) + ":");
            chain.handle(new Request("192.168.1.1", "Bearer token",
                                      "/api/orders", true));
        }
    }
}
```

---

## Build This — Complete Polymorphism Practice

```java
// File: NotificationSystem.java
// A notification system demonstrating polymorphism
// at every level

import java.time.*;
import java.util.*;

public class NotificationSystem {

    // ═══════════════════════════════════════
    // NOTIFICATION INTERFACE
    // ═══════════════════════════════════════
    
    interface Notification {
        void send();
        boolean canSend();
        String getType();
        String getRecipient();
        String preview();
        
        default String getStatus() {
            return canSend() ? "READY" : "BLOCKED";
        }
    }

    // ═══════════════════════════════════════
    // CONCRETE NOTIFICATION TYPES
    // ═══════════════════════════════════════
    
    static class EmailNotification implements Notification {
        private final String toEmail;
        private final String subject;
        private final String body;
        private boolean isHtml;
        
        public EmailNotification(String toEmail, String subject,
                                  String body, boolean isHtml) {
            this.toEmail  = toEmail;
            this.subject  = subject;
            this.body     = body;
            this.isHtml   = isHtml;
        }
        
        @Override
        public void send() {
            System.out.printf("📧 EMAIL → %s%n   Subject: %s%n   Format: %s%n",
                              toEmail, subject, isHtml ? "HTML" : "Plain");
        }
        
        @Override
        public boolean canSend() {
            return toEmail != null && toEmail.contains("@") && !subject.isBlank();
        }
        
        @Override public String getType()      { return "EMAIL"; }
        @Override public String getRecipient() { return toEmail; }
        
        @Override
        public String preview() {
            return String.format("Email to %s: '%s'",
                                 toEmail, subject.substring(0, Math.min(30, subject.length())));
        }
    }
    
    static class SmsNotification implements Notification {
        private final String phoneNumber;
        private final String message;
        private static final int MAX_LENGTH = 160;
        
        public SmsNotification(String phoneNumber, String message) {
            this.phoneNumber = phoneNumber;
            this.message     = message.length() > MAX_LENGTH
                               ? message.substring(0, MAX_LENGTH - 3) + "..."
                               : message;
        }
        
        @Override
        public void send() {
            System.out.printf("📱 SMS → %s%n   Message: %s%n   Length: %d/%d chars%n",
                              phoneNumber, message, message.length(), MAX_LENGTH);
        }
        
        @Override
        public boolean canSend() {
            String cleaned = phoneNumber.replaceAll("[\\s\\-]", "");
            return cleaned.matches("(\\+88)?01[3-9]\\d{8}");
        }
        
        @Override public String getType()      { return "SMS"; }
        @Override public String getRecipient() { return phoneNumber; }
        
        @Override
        public String preview() {
            return String.format("SMS to %s: '%s'",
                                 phoneNumber,
                                 message.substring(0, Math.min(20, message.length())));
        }
    }
    
    static class PushNotification implements Notification {
        private final String deviceToken;
        private final String title;
        private final String body;
        private final Map<String, String> data;
        
        public PushNotification(String deviceToken, String title,
                                  String body, Map<String, String> data) {
            this.deviceToken = deviceToken;
            this.title       = title;
            this.body        = body;
            this.data        = data != null ? data : new HashMap<>();
        }
        
        @Override
        public void send() {
            System.out.printf("🔔 PUSH → %s%n   Title: %s%n   Body: %s%n",
                              deviceToken.substring(0, 8) + "...", title, body);
            if (!data.isEmpty()) {
                System.out.println("   Data: " + data);
            }
        }
        
        @Override
        public boolean canSend() {
            return deviceToken != null && deviceToken.length() >= 20
                && !title.isBlank();
        }
        
        @Override public String getType()      { return "PUSH"; }
        @Override public String getRecipient() { return deviceToken.substring(0, 8) + "..."; }
        
        @Override
        public String preview() {
            return String.format("Push to device %s: '%s'",
                                 deviceToken.substring(0, 8), title);
        }
    }
    
    static class InAppNotification implements Notification {
        private final long userId;
        private final String message;
        private final String category;
        private final String actionUrl;
        private boolean isRead;
        private final LocalDateTime createdAt;
        
        public InAppNotification(long userId, String message,
                                   String category, String actionUrl) {
            this.userId    = userId;
            this.message   = message;
            this.category  = category;
            this.actionUrl = actionUrl;
            this.isRead    = false;
            this.createdAt = LocalDateTime.now();
        }
        
        @Override
        public void send() {
            System.out.printf("🏠 IN-APP → User #%d%n   [%s] %s%n   URL: %s%n",
                              userId, category, message, actionUrl);
        }
        
        @Override public boolean canSend() { return userId > 0 && !message.isBlank(); }
        @Override public String getType()      { return "IN_APP"; }
        @Override public String getRecipient() { return "User#" + userId; }
        
        @Override
        public String preview() {
            return String.format("In-App for user %d: [%s] %s",
                                 userId, category, message.substring(0, Math.min(25, message.length())));
        }
        
        public void markRead() { isRead = true; }
        public boolean isRead() { return isRead; }
    }

    // ═══════════════════════════════════════
    // NOTIFICATION DISPATCHER (uses polymorphism)
    // ═══════════════════════════════════════
    
    static class NotificationDispatcher {
        private final List<Notification> queue = new ArrayList<>();
        private int sent = 0, failed = 0, skipped = 0;
        
        public void add(Notification notification) {
            queue.add(notification);
        }
        
        public void addAll(List<Notification> notifications) {
            queue.addAll(notifications);
        }
        
        public void dispatchAll() {
            System.out.println("\n╔══════════════════════════════════════════╗");
            System.out.println("║      DISPATCHING ALL NOTIFICATIONS       ║");
            System.out.println("╠══════════════════════════════════════════╣");
            
            for (Notification n : queue) {
                System.out.printf("║  [%s] %s%n", n.getType(), n.preview());
                
                if (!n.canSend()) {
                    System.out.println("║  ⚠️  Skipped: cannot send (invalid data)");
                    skipped++;
                    continue;
                }
                
                try {
                    n.send();  // POLYMORPHIC — each type sends differently
                    sent++;
                    System.out.println("║  ✅ Sent successfully");
                } catch (Exception e) {
                    failed++;
                    System.out.println("║  ❌ Failed: " + e.getMessage());
                }
                System.out.println("║");
            }
            
            System.out.println("╠══════════════════════════════════════════╣");
            System.out.printf( "║  Sent: %-5d Skipped: %-5d Failed: %-5d║%n",
                               sent, skipped, failed);
            System.out.println("╚══════════════════════════════════════════╝");
        }
        
        public void dispatchByType(String type) {
            System.out.println("\n--- Dispatching " + type + " notifications ---");
            int count = 0;
            for (Notification n : queue) {
                if (n.getType().equals(type) && n.canSend()) {
                    n.send();
                    count++;
                }
            }
            System.out.println("Dispatched " + count + " " + type + " notifications.");
        }
        
        public Map<String, Long> getStats() {
            Map<String, Long> stats = new LinkedHashMap<>();
            stats.put("total", (long) queue.size());
            stats.put("sent", (long) sent);
            stats.put("skipped", (long) skipped);
            stats.put("failed", (long) failed);
            
            // Count by type using polymorphism:
            Map<String, Long> byType = new LinkedHashMap<>();
            for (Notification n : queue) {
                byType.merge(n.getType(), 1L, Long::sum);
            }
            stats.putAll(byType);
            
            return stats;
        }
    }

    // ═══════════════════════════════════════
    // NOTIFICATION BUILDER (fluent API)
    // ═══════════════════════════════════════
    
    static class NotificationBuilder {
        private final List<Notification> notifications = new ArrayList<>();
        
        public NotificationBuilder email(String to, String subject, String body) {
            notifications.add(new EmailNotification(to, subject, body, false));
            return this;
        }
        
        public NotificationBuilder htmlEmail(String to, String subject, String body) {
            notifications.add(new EmailNotification(to, subject, body, true));
            return this;
        }
        
        public NotificationBuilder sms(String phone, String message) {
            notifications.add(new SmsNotification(phone, message));
            return this;
        }
        
        public NotificationBuilder push(String token, String title, String body) {
            notifications.add(new PushNotification(token, title, body, null));
            return this;
        }
        
        public NotificationBuilder inApp(long userId, String message,
                                          String category, String url) {
            notifications.add(new InAppNotification(userId, message, category, url));
            return this;
        }
        
        public List<Notification> build() {
            return new ArrayList<>(notifications);
        }
    }

    // ═══════════════════════════════════════
    // MAIN
    // ═══════════════════════════════════════
    
    public static void main(String[] args) {
        
        NotificationDispatcher dispatcher = new NotificationDispatcher();
        
        // Order confirmation — multiple notification types for ONE event:
        List<Notification> orderConfirmation = new NotificationBuilder()
            .email("rahim@example.com",
                   "Order Confirmed - ORD-2024-001",
                   "<h1>Your order has been confirmed!</h1>")
            .htmlEmail("rahim@example.com",
                       "Order #ORD-2024-001 confirmed",
                       "<p>Thank you for your purchase!</p>")
            .sms("01712345678",
                 "Order ORD-2024-001 confirmed! Total: ৳1500. Track at app.com/track")
            .push("device_token_abc123xyz_very_long_token_here",
                  "Order Confirmed! 🎉",
                  "Your order is being prepared")
            .inApp(1001L,
                   "Your order ORD-2024-001 has been confirmed",
                   "ORDER",
                   "/orders/ORD-2024-001")
            .build();
        
        dispatcher.addAll(orderConfirmation);
        
        // Payment failure — urgent notifications:
        dispatcher.add(new SmsNotification(
            "01812345678",
            "URGENT: Payment failed for order ORD-2024-002. Please retry."));
        dispatcher.add(new EmailNotification(
            "karim@example.com",
            "Payment Failed - Action Required",
            "Your payment could not be processed...", false));
        
        // Invalid data (should be skipped):
        dispatcher.add(new SmsNotification("invalid-phone", "Test")); // bad phone
        dispatcher.add(new EmailNotification("notanemail", "Test", "body", false)); // bad email
        dispatcher.add(new PushNotification("short", "Title", "Body", null)); // token too short
        
        // Dispatch all:
        dispatcher.dispatchAll();
        
        // Stats:
        System.out.println("\n=== STATISTICS ===");
        dispatcher.getStats().forEach((k, v) ->
            System.out.printf("  %-15s: %d%n", k, v));
        
        // Dispatch only a specific type:
        dispatcher.dispatchByType("EMAIL");
    }
}
```

---

## Exercises

```text
EXERCISE 1: Shape Area Calculator (Runtime Polymorphism)
  Create ShapeCalculator.java

  Interface Shape:
  - double getArea()
  - double getPerimeter()
  - String getType()
  - default String describe() → formats area + perimeter

  Implement: Circle, Rectangle, Triangle, Pentagon, Hexagon

  Create a ShapeCollection class that:
  - Holds a List<Shape>
  - add(Shape s)
  - getTotalArea() → sum of all areas
  - getLargest() → shape with max area
  - getSmallest() → shape with min area
  - printAll() → print each shape's details
  - filterByType(String type) → return List<Shape>
  - sortByArea() → sort in place

  Create 10+ shapes of mixed types, test all methods.

EXERCISE 2: Media Player (Compile + Runtime)
  Create MediaPlayer.java

  Compile-time polymorphism:
  class AudioPlayer {
    void play(String filename)
    void play(String filename, int startSecond)
    void play(String filename, int start, int end)
    void play(String[] playlist)
  }

  Runtime polymorphism:
  interface MediaFile { void play(); String getInfo(); int getDurationSeconds(); }

  class Mp3File implements MediaFile { ... }
  class FlacFile implements MediaFile { ... }
  class WavFile implements MediaFile { ... }
  class VideoFile implements MediaFile { ... }
  class StreamingFile implements MediaFile { ... }

  Playlist that:
  - Holds List<MediaFile>
  - playAll() → polymorphic play
  - getTotalDuration() → sum of all durations
  - shuffle() → random order then play
  - filterByType(Class<?> type) → return matching files

EXERCISE 3: Strategy Pattern for Sorting
  Create SortingStrategies.java

  interface SortStrategy<T extends Comparable<T>> {
    List<T> sort(List<T> items);
    String getName();
    int getTimeComplexityN(); // big O coefficient
  }

  Implement:
  - BubbleSort
  - SelectionSort
  - InsertionSort
  - MergeSort (simplified)
  - QuickSort (simplified)

  SortingBenchmark class:
  - Takes List<SortStrategy<Integer>> strategies
  - Runs each on same data
  - Records time taken
  - Prints comparison table

  Test with lists of size: 100, 1000, 10000

EXERCISE 4: Observer Pattern (Event System)
  Create EventSystem.java

  Build a full Observer/Event system:

  interface Event { String getType(); LocalDateTime getTimestamp(); }
  interface EventListener<T extends Event> { void onEvent(T event); }

  Events: UserRegistered, OrderPlaced, PaymentReceived, ItemShipped

  Listeners for each event:
  UserRegistered → WelcomeEmailListener, TrialActivationListener
  OrderPlaced → InventoryUpdateListener, InvoiceListener
  PaymentReceived → OrderConfirmationListener, ReceiptListener
  ItemShipped → TrackingCodeListener, ShipmentEmailListener

  EventBus that:
  - subscribe(String eventType, EventListener<?> listener)
  - publish(Event event)
  - getListenerCount(String eventType)
  - unsubscribe(String eventType, EventListener<?> listener)

  Simulate a complete order flow.

EXERCISE 5: Polymorphism Reflection
  Without running code, answer:

  class A {
    void method() { System.out.println("A"); }
    static void staticMethod() { System.out.println("A-static"); }
  }
  class B extends A {
    @Override
    void method() { System.out.println("B"); }
    static void staticMethod() { System.out.println("B-static"); }
  }

  A obj1 = new A();
  A obj2 = new B();
  B obj3 = new B();

  What prints?
  a) obj1.method()
  b) obj2.method()
  c) obj3.method()
  d) A.staticMethod()
  e) B.staticMethod()
  f) obj2.staticMethod()  ← tricky!

  Then verify. Explain WHY each answer is what it is.
  Push to GitHub: "feat: notification system with polymorphism"
```

---

## Common Mistakes

```text
MISTAKE 1: Expecting reference type to determine method call
  Animal a = new Dog("Rex");
  a.makeSound(); // Calls DOG's makeSound — NOT Animal's!
  // The ACTUAL object type determines which method runs.
  // Reference type (Animal) only determines what methods are AVAILABLE.

MISTAKE 2: Using instanceof everywhere (defeats polymorphism)
  for (Animal a : animals) {
      if (a instanceof Dog) { ((Dog)a).fetch(); }
      else if (a instanceof Cat) { ((Cat)a).purr(); }
  }
  // This means you're NOT using polymorphism properly.
  // Fix: add fetch()/purr() to Animal (or an interface),
  //      override in each class. Then: a.doYourThing();

MISTAKE 3: Forgetting @Override — typo creates new method
  class Dog extends Animal {
      void makeSOUND() { ... } // typo! NOT overriding makeSound()
  }
  // Fix: always @Override — catches this at compile time.

MISTAKE 4: Assuming static methods are polymorphic
  Animal a = new Dog("Rex");
  a.staticMethod(); // Calls ANIMAL's static method, NOT Dog's!
  // Static methods are resolved by REFERENCE type (compile time).
  // They are HIDDEN not OVERRIDDEN.
  // Fix: call static methods through the class: Dog.staticMethod()

MISTAKE 5: Calling subclass-specific methods through parent reference
  Animal a = new Dog("Rex");
  a.fetch(); // COMPILE ERROR: Animal has no fetch()
  // Fix: downcast (safely): if (a instanceof Dog d) { d.fetch(); }

MISTAKE 6: Downcasting without instanceof check
  Animal a = new Cat("Luna");
  Dog d = (Dog) a; // COMPILE ERROR? No! RUNTIME ClassCastException!
  // Compiler allows it. JVM crashes at runtime.
  // Fix: ALWAYS check: if (a instanceof Dog d) { d.doThing(); }

MISTAKE 7: Thinking overloading is runtime polymorphism
  void process(Animal a) { ... }
  void process(Dog d) { ... }

  Animal a = new Dog("Rex");
  process(a); // Calls process(Animal) — compile-time decision!
  // Even though actual object is Dog, overload picks Animal (reference type).
  // Overloading is compile-time. Overriding is runtime.

MISTAKE 8: Not using interfaces for dependencies
  class Service {
      private MySQLRepository repo; // tight coupling!
  }
  // Fix:
  class Service {
      private UserRepository repo; // interface — flexible!
  }
  // Now you can inject any implementation.

MISTAKE 9: Breaking the Liskov Substitution Principle
  // If subclass breaks parent's contract, polymorphism breaks:
  class Square extends Rectangle {
      @Override
      public void setWidth(double w) {
          super.setWidth(w);
          super.setHeight(w); // Square forces equal sides
      }
  }
  // Code expecting Rectangle behavior breaks with Square!
  // Fix: Square should NOT extend Rectangle.

MISTAKE 10: Not understanding default methods in interfaces
  interface Flyable {
      void fly();
      default void land() { System.out.println("Landing..."); }
  }
  class Bird implements Flyable {
      public void fly() { System.out.println("Flapping wings"); }
      // land() is inherited from interface — no override needed!
  }
  // default methods in interfaces provide base behavior.
  // Override them when you need different behavior.
```

---

## Interview Questions

**Q: What is polymorphism in Java? What are the two types?**
A: Polymorphism means "many forms" — the same interface or method behaves differently depending on the actual type involved. Type 1: Compile-time polymorphism (static binding) through method overloading. The compiler picks the right method based on the argument types at compile time. Type 2: Runtime polymorphism (dynamic binding) through method overriding with inheritance or interfaces. The JVM picks the right method at runtime based on the actual object type, not the reference type. Example: Animal a = new Dog(); a.makeSound() calls Dog's makeSound() even though 'a' is declared as Animal. Runtime polymorphism is what's meant when interviewers ask about polymorphism in professional Java.

**Q: How does Java resolve method calls at runtime? (vtable)**
A: When the JVM loads a class, it builds a virtual method table (vtable) that maps each method to its implementation for that class. When a method is called on a reference, the JVM looks up the ACTUAL object's class (not the reference type) and finds the correct implementation in that class's vtable. If the method is overridden in the child class, the vtable points to the child's version. If not overridden, it points to the inherited parent's version. This lookup happens at runtime — hence "dynamic dispatch." The JIT compiler can optimize frequently called virtual methods by inlining them (devirtualization) when the type is deterministic.

**Q: What is the difference between method overriding and hiding?**
A: Instance method overriding: child provides a new implementation, resolved at RUNTIME based on actual object type (polymorphism). Animal a = new Dog(); a.move() calls Dog's move(). Static method hiding: child declares a static method with the same signature as parent's static method. Resolved at COMPILE TIME based on the REFERENCE type — not polymorphic. Animal a = new Dog(); a.staticMove() calls ANIMAL's staticMove()! Dog.staticMove() calls Dog's. Static methods are not polymorphic. This is a common interview trap question.

**Q: Can you achieve polymorphism without inheritance?**
A: Yes — through interfaces. A class doesn't need to extend another class. It just needs to implement an interface. Multiple unrelated classes can implement the same interface and be treated polymorphically. Example: String, File, and SocketStream all implement Serializable — you can pass any of them where Serializable is expected, even though they share no parent class. In fact, programming to interfaces (rather than abstract classes) is often preferred because Java allows implementing multiple interfaces but only extending one class.

**Q: What is the Liskov Substitution Principle and how does it relate to polymorphism?**
A: LSP states: if S is a subtype of T, then objects of type T can be replaced with objects of type S without altering the correctness of the program. In simple terms: a child class should be fully substitutable for its parent class. Polymorphism relies on LSP — if you write code using Animal references, it should work correctly with any Animal subtype (Dog, Cat, Bird). LSP is violated when a subclass changes behavior in a way that breaks code expecting the parent's contract. Example: if Rectangle has setWidth() and setHeight() independently, Square violating this by forcing equal sides breaks code expecting Rectangle behavior. Well-designed polymorphic systems follow LSP.

**Q: When should you use abstract classes vs interfaces for polymorphism?**
A: Use interfaces when: - Unrelated classes need to share behavior (Comparable, Serializable) - You want to define a pure contract with no state - A class needs to implement multiple behaviors (multiple interfaces) - Maximum flexibility is needed
Use abstract classes when: - Related classes share common state AND behavior - You want to provide default implementations to avoid duplication - You need the template method pattern (define algorithm skeleton, subclasses fill in steps) - You want to share protected helper methods
Modern Java (8+): interfaces can have default methods, so the line is blurrier. General advice: prefer interfaces for type hierarchies, use abstract classes when significant shared state or protected helper methods are needed.

---

## Key Takeaways

```text
1. Polymorphism = "one interface, many implementations"
   Same code works with different types. Open for extension,
   closed for modification.

2. TWO TYPES:
   Compile-time (overloading): compiler picks method by argument types
   Runtime (overriding): JVM picks method by ACTUAL object type

3. THE GOLDEN RULE of runtime polymorphism:
   The ACTUAL object type determines which method runs.
   NOT the reference type.
   Animal a = new Dog(); a.makeSound() → Dog's makeSound()

4. Reference type determines WHAT METHODS ARE AVAILABLE.
   Actual object type determines WHICH IMPLEMENTATION RUNS.
   These two facts together explain all polymorphism behavior.

5. STATIC METHODS are NOT polymorphic.
   They are resolved by reference type at compile time (hiding, not overriding).
   Never rely on static method polymorphism.

6. Using instanceof everywhere = polymorphism not working.
   Fix by moving type-specific logic into overridden methods.
   If you're checking the type to call different methods →
   those methods should be in the interface/parent class.

7. Interface polymorphism is MORE FLEXIBLE than class inheritance.
   Multiple interfaces, unrelated classes, maximum reusability.
   "Program to an interface, not an implementation."

8. Design patterns built on polymorphism:
   Strategy → swap algorithms at runtime
   Observer → multiple handlers for one event
   Decorator → add behavior without changing class
   Chain of Responsibility → Spring Security filter chain
   Template Method → define skeleton, subclasses fill gaps

9. In Spring Boot:
   Repository interfaces → swap implementations (test vs prod)
   Service dependencies → inject any implementation
   Event handlers → any number of listeners per event
   Filters → chain of handlers (Spring Security)

10. Polymorphism enables the Open/Closed Principle:
    Add new behavior by creating new classes.
    Existing code never changes.
    This is the heart of maintainable, extensible software.
```

---