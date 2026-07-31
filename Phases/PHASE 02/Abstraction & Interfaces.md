**Phase:** Level 2 - OOP
**Date Studied:**

---
## What Problem Does This Solve?

```text
You've learned:
Classes → blueprints for objects
Encapsulation → hide internal details
Inheritance → reuse and extend behavior
Polymorphism → one interface, many implementations

Now the deepest question:
How do you design systems that are FLEXIBLE and STABLE at the same time?
How do you write code that WORKS with things that don't exist yet?
How do you ENFORCE that every implementation does what's required?

Real scenario — you're building a payment gateway:
Today: credit cards, bKash, Nagad
Next month: adding cryptocurrency
Next quarter: adding Stripe, PayPal

If your code is tied to specific implementations:
PaymentService {
  void processCreditCard(CreditCard card) { ... }
  void processBkash(BkashAccount acc) { ... }
  // Every new payment type = new method = change existing code
}

Every new payment type breaks your existing code.

With abstraction:
interface PaymentGateway {
  PaymentResult process(PaymentRequest request);
  boolean refund(String transactionId, double amount);
  String getGatewayName();
}

PaymentService {
  void process(PaymentGateway gateway, PaymentRequest request) {
    // Works with ANY gateway — now and in the future
    // This code NEVER changes when new gateways are added
  }
}

CryptoGateway implements PaymentGateway { ... } // add this next month
StripeGateway implements PaymentGateway { ... } // add next quarter

PaymentService NEVER changes.
It was written once, works forever.

Abstraction = show WHAT, hide HOW.
The "what": process a payment (interface)
The "how": credit card vs bKash vs crypto (implementations)

This is the highest-level OOP concept.
Understanding this makes you think like a senior engineer.
```

---

## 1. What Is Abstraction?

```text
Abstraction = focusing on WHAT something does,
not HOW it does it.

Real-world analogies:
TV remote control:
  WHAT: change channel, adjust volume, power on/off
  HOW: infrared signals, circuit board logic — hidden
  You use the "interface" (remote) without knowing the internals.

Car steering wheel:
  WHAT: turn left, turn right
  HOW: hydraulic power steering, rack and pinion — hidden
  Driver uses the interface without knowing the mechanism.

Database:
  WHAT: save data, find data, delete data
  HOW: B-tree indexes, WAL logs, buffer pools — hidden
  You write SQL (the interface) without knowing the storage engine.

In Java, abstraction is achieved through:
1. ABSTRACT CLASSES — partially abstract, can have state
2. INTERFACES — fully abstract (mostly), pure contracts

The principle: program to the ABSTRACTION, not the implementation.
Don't write: PostgreSQLUserRepository repo = new PostgreSQLUserRepository();
Write: UserRepository repo = new PostgreSQLUserRepository();

Now repo works with ANY implementation.
Tests can swap to InMemoryUserRepository.
Production can use PostgreSQL.
Same code works with both.
```

---

## 2. Abstract Classes — The Complete Guide

```java
public class AbstractClassGuide {

    // ─────────────────────────────────────────
    // WHAT IS AN ABSTRACT CLASS?
    //
    // A class that:
    // → Has 'abstract' keyword
    // → CANNOT be instantiated directly (no 'new AbstractClass()')
    // → Can have abstract methods (no body — subclasses MUST implement)
    // → Can have concrete methods (with body — subclasses inherit)
    // → Can have fields, constructors, static methods
    // → Sits between interface (pure contract) and concrete class
    // ─────────────────────────────────────────

    // Abstract class: defines structure, leaves details to subclasses
    abstract static class Report {

        // ── Fields (shared by all reports) ──
        protected final String title;
        protected final String author;
        protected final java.time.LocalDate generatedDate;
        protected String format; // PDF, CSV, HTML, etc.

        // ── Constructor (abstract classes CAN have constructors) ──
        // Subclasses call this via super()
        public Report(String title, String author) {
            this.title = title;
            this.author = author;
            this.generatedDate = java.time.LocalDate.now();
            this.format = "PDF"; // default
        }

        // ── ABSTRACT METHODS (no body — subclasses MUST implement) ──
        // These define WHAT every report must do, not HOW

        public abstract String generateContent(); // each report has own content
        public abstract String getReportType(); // return "FINANCIAL", "SALES", etc.
        public abstract void exportToCsv(); // each exports differently

        // ── CONCRETE METHODS (with body — subclasses inherit) ──
        // These define common behavior shared by ALL reports

        public final void generate() { // final: subclasses cannot change this
            System.out.println("╔══════════════════════════════════╗");
            System.out.println("║ GENERATING " + getReportType() + " REPORT");
            System.out.println("╠══════════════════════════════════╣");
            System.out.println("║ Title : " + title);
            System.out.println("║ Author : " + author);
            System.out.println("║ Date : " + generatedDate);
            System.out.println("║ Format : " + format);
            System.out.println("╠══════════════════════════════════╣");
            System.out.println(generateContent()); // polymorphic — subclass provides
            System.out.println("╚══════════════════════════════════╝");
        }

        // Concrete method using abstract method (polymorphism!):
        public void preview() {
            System.out.println("[PREVIEW] " + getReportType() + ": " + title);
            System.out.println(generateContent().substring(
                0, Math.min(100, generateContent().length())) + "...");
        }

        // Concrete setters:
        public void setFormat(String format) {
            java.util.List valid = java.util.List.of("PDF","CSV","HTML","EXCEL");
            if (!valid.contains(format))
                throw new IllegalArgumentException("Invalid format: " + format);
            this.format = format;
        }

        // Shared utility:
        protected String formatCurrency(double amount) {
            return String.format("৳%,.2f", amount);
        }

        // Getters:
        public String getTitle() { return title; }
        public String getAuthor() { return author; }
        public String getFormat() { return format; }

        @Override
        public String toString() {
            return getReportType() + " Report: '" + title + "' by " + author;
        }
    }

    // CONCRETE SUBCLASS 1: Financial Report
    static class FinancialReport extends Report {
        private double revenue;
        private double expenses;
        private double profit;

        public FinancialReport(String title, String author,
                               double revenue, double expenses) {
            super(title, author);
            this.revenue = revenue;
            this.expenses = expenses;
            this.profit = revenue - expenses;
        }

        @Override
        public String generateContent() {
            return String.format(
                "║ Revenue : %s%n║ Expenses : %s%n║ Profit : %s%n" +
                "║ Margin : %.1f%%",
                formatCurrency(revenue), formatCurrency(expenses),
                formatCurrency(profit), (profit / revenue) * 100);
        }

        @Override
        public String getReportType() { return "FINANCIAL"; }

        @Override
        public void exportToCsv() {
            System.out.println("Exporting financial data: Revenue, Expenses, Profit...");
        }

        public double getProfit() { return profit; }
        public double getMargin() { return profit / revenue; }
    }

    // CONCRETE SUBCLASS 2: Sales Report
    static class SalesReport extends Report {
        private int unitsSold;
        private double avgOrderValue;
        private String topProduct;
        private String topRegion;

        public SalesReport(String title, String author,
                           int unitsSold, double avgOrderValue,
                           String topProduct, String topRegion) {
            super(title, author);
            this.unitsSold = unitsSold;
            this.avgOrderValue = avgOrderValue;
            this.topProduct = topProduct;
            this.topRegion = topRegion;
        }

        @Override
        public String generateContent() {
            return String.format(
                "║ Units Sold : %,d%n" +
                "║ Avg Order : %s%n" +
                "║ Total Revenue : %s%n" +
                "║ Top Product : %s%n" +
                "║ Top Region : %s",
                unitsSold, formatCurrency(avgOrderValue),
                formatCurrency(unitsSold * avgOrderValue),
                topProduct, topRegion);
        }

        @Override
        public String getReportType() { return "SALES"; }

        @Override
        public void exportToCsv() {
            System.out.println("Exporting sales data: Units, Region, Product...");
        }
    }

    // CONCRETE SUBCLASS 3: User Activity Report
    static class UserActivityReport extends Report {
        private int activeUsers;
        private int newSignups;
        private double retentionRate;
        private String topFeature;

        public UserActivityReport(String title, String author,
                                  int activeUsers, int newSignups,
                                  double retentionRate, String topFeature) {
            super(title, author);
            this.activeUsers = activeUsers;
            this.newSignups = newSignups;
            this.retentionRate = retentionRate;
            this.topFeature = topFeature;
        }

        @Override
        public String generateContent() {
            return String.format(
                "║ Active Users : %,d%n" +
                "║ New Sign-ups : %,d%n" +
                "║ Retention : %.1f%%%n" +
                "║ Top Feature : %s",
                activeUsers, newSignups, retentionRate * 100, topFeature);
        }

        @Override
        public String getReportType() { return "USER_ACTIVITY"; }

        @Override
        public void exportToCsv() {
            System.out.println("Exporting user data: Active, Signups, Retention...");
        }
    }

    public static void main(String[] args) {

        // Cannot instantiate abstract class:
        // Report r = new Report("Test", "Author"); // COMPILE ERROR!

        // Create concrete instances:
        FinancialReport finance = new FinancialReport("Q4 2024", "CFO", 5_000_000, 3_200_000);
        SalesReport sales = new SalesReport("November Sales", "Sales Manager",
                                            1250, 1800.0, "Laptop Pro", "Dhaka");
        UserActivityReport activity = new UserActivityReport("Monthly Users", "Product Team",
                                                             45000, 3200, 0.87, "Dashboard");

        // Generate each — same interface, different content (polymorphism!):
        finance.generate();
        System.out.println();
        sales.generate();
        System.out.println();
        activity.generate();

        // Use abstract reference for polymorphism:
        System.out.println("\n=== Polymorphic Processing ===");
        Report[] reports = { finance, sales, activity };
        for (Report report : reports) {
            report.preview(); // each generates its own preview
            report.exportToCsv(); // each exports its own way
        }

        // Abstract class reference — access common behavior:
        Report r = new FinancialReport("Annual", "CEO", 10_000_000, 7_000_000);
        r.setFormat("EXCEL");
        System.out.println("Format: " + r.getFormat());
        System.out.println(r); // toString() works

        // Check profit (need downcast for specific method):
        if (r instanceof FinancialReport fr) {
            System.out.printf("Profit: %s (%.1f%% margin)%n",
                              fr.formatCurrency(fr.getProfit()),
                              fr.getMargin() * 100);
        }
    }
}
```

---

## 3. Interfaces — The Complete Guide

```java
public class InterfaceCompleteGuide {

    // ─────────────────────────────────────────
    // WHAT IS AN INTERFACE?
    //
    // A pure CONTRACT — defines WHAT, never HOW.
    // A collection of method signatures that a class agrees to implement.
    // The most powerful abstraction tool in Java.
    //
    // RULES:
    // → Cannot be instantiated (no 'new Interface()')
    // → All fields are automatically: public static final (constants)
    // → All abstract methods are automatically: public abstract
    // → A class can implement MULTIPLE interfaces
    // → Can have: default methods (Java 8+), static methods (Java 8+),
    //   private methods (Java 9+)
    // ─────────────────────────────────────────

    // ─────────────────────────────────────────
    // BASIC INTERFACE
    // ─────────────────────────────────────────

    interface Drawable {
        // These are automatically: public static final
        int DEFAULT_COLOR = 0xFF000000; // black
        String DEFAULT_STYLE = "SOLID";

        // These are automatically: public abstract
        void draw();
        void erase();

        // Default method (Java 8+) — has a body, can be overridden:
        default void resize(double factor) {
            System.out.println("Resizing by factor: " + factor);
        }

        // Static method (Java 8+) — belongs to the interface:
        static Drawable createDefault() {
            return new Drawable() {
                @Override public void draw() { System.out.println("Drawing default"); }
                @Override public void erase() { System.out.println("Erasing default"); }
            };
        }
    }

    interface Colorable {
        void setColor(String color);
        String getColor();

        default void resetColor() {
            setColor("BLACK"); // calls the abstract setColor()
        }
    }

    interface Resizable {
        void resize(int width, int height);
        int getWidth();
        int getHeight();

        default double getAspectRatio() {
            return (double) getWidth() / getHeight();
        }
    }

    // ─────────────────────────────────────────
    // CLASS IMPLEMENTING MULTIPLE INTERFACES
    // ─────────────────────────────────────────

    static class Canvas implements Drawable, Colorable, Resizable {
        private String color;
        private int width;
        private int height;
        private java.util.List<String> elements;

        public Canvas(int width, int height) {
            this.width = width;
            this.height = height;
            this.color = "WHITE";
            this.elements = new java.util.ArrayList<>();
        }

        // Drawable:
        @Override public void draw() {
            System.out.printf("Drawing canvas %dx%d in %s with %d elements%n",
                              width, height, color, elements.size());
        }
        @Override public void erase() {
            elements.clear();
            System.out.println("Canvas erased.");
        }

        // Override default resize from Drawable:
        @Override public void resize(double factor) {
            this.width = (int)(width * factor);
            this.height = (int)(height * factor);
            System.out.println("Canvas resized to " + width + "x" + height);
        }

        // Colorable:
        @Override public void setColor(String color) { this.color = color; }
        @Override public String getColor() { return color; }

        // Resizable:
        @Override public void resize(int w, int h) { width = w; height = h; }
        @Override public int getWidth() { return width; }
        @Override public int getHeight() { return height; }

        // Canvas-specific:
        public void addElement(String element) { elements.add(element); }
    }

    // ─────────────────────────────────────────
    // DEFAULT METHOD CONFLICT RESOLUTION
    // ─────────────────────────────────────────

    interface A {
        default void greet() { System.out.println("Hello from A"); }
    }

    interface B {
        default void greet() { System.out.println("Hello from B"); }
    }

    // Class implementing both MUST resolve the conflict:
    static class C implements A, B {
        @Override
        public void greet() {
            A.super.greet(); // explicitly choose A's version
            // OR: B.super.greet();
            // OR: provide your own implementation entirely
            System.out.println("Hello from C (resolved conflict)");
        }
    }

    // ─────────────────────────────────────────
    // FUNCTIONAL INTERFACES — Single Abstract Method
    // Foundation of lambdas (Level 2.9)
    // ─────────────────────────────────────────

    @FunctionalInterface // optional annotation — enforces single abstract method
    interface PaymentProcessor {
        boolean process(double amount); // the ONE abstract method

        // Default methods are fine (don't violate functional interface rule):
        default void onSuccess(double amount) {
            System.out.printf("Payment of ৳%.2f processed successfully%n", amount);
        }

        default void onFailure(double amount) {
            System.out.printf("Payment of ৳%.2f FAILED%n", amount);
        }
    }

    // ─────────────────────────────────────────
    // INTERFACE INHERITANCE
    // ─────────────────────────────────────────

    interface Vehicle {
        void start();
        void stop();
        int getSpeed();
    }

    // Interface extending another interface:
    interface ElectricVehicle extends Vehicle {
        int getBatteryLevel();
        void charge(int minutes);

        default boolean needsCharging() {
            return getBatteryLevel() < 20;
        }
    }

    // Interface extending MULTIPLE interfaces:
    interface HybridVehicle extends Vehicle, ElectricVehicle {
        double getFuelLevel();
        void refuel(double liters);

        default String getCurrentMode() {
            return getBatteryLevel() > 30 ? "ELECTRIC" : "HYBRID";
        }
    }

    static class TeslaModel3 implements ElectricVehicle {
        private int speed = 0;
        private int battery = 90;

        @Override public void start() { System.out.println("Tesla: silently starting"); }
        @Override public void stop() { speed = 0; System.out.println("Tesla: stopped"); }
        @Override public int getSpeed() { return speed; }
        @Override public int getBatteryLevel() { return battery; }

        @Override
        public void charge(int minutes) {
            int added = minutes / 5; // 1% per 5 minutes (simplified)
            battery = Math.min(100, battery + added);
            System.out.printf("Charged for %d min. Battery: %d%%%n", minutes, battery);
        }
    }

    public static void main(String[] args) {

        System.out.println("=== Canvas (Multiple Interfaces) ===");
        Canvas canvas = new Canvas(1920, 1080);
        canvas.addElement("Rectangle");
        canvas.addElement("Circle");
        canvas.draw();

        canvas.setColor("BLUE"); // Colorable
        canvas.resize(0.5); // Drawable.resize()
        canvas.draw();

        canvas.resize(800, 600); // Resizable.resize()
        System.out.printf("Aspect ratio: %.2f%n", canvas.getAspectRatio());

        canvas.resetColor(); // Colorable default method

        // Same canvas, different interface views:
        Drawable d = canvas;
        Colorable c = canvas;
        Resizable r = canvas;
        System.out.println("Width: " + r.getWidth()); // 800

        System.out.println("\n=== Default Method Conflict ===");
        C obj = new C();
        obj.greet(); // resolved in C

        System.out.println("\n=== Functional Interface ===");
        // Implementation via anonymous class:
        PaymentProcessor bkash = new PaymentProcessor() {
            @Override
            public boolean process(double amount) {
                System.out.println("Processing via bKash: ৳" + amount);
                return amount <= 25000;
            }
        };

        // Implementation via lambda (Level 2.9 preview):
        PaymentProcessor card = amount -> {
            System.out.println("Processing via card: ৳" + amount);
            return amount <= 500000;
        };

        processPayment(bkash, 5000.0);
        processPayment(card, 50000.0);

        System.out.println("\n=== Interface Hierarchy ===");
        TeslaModel3 tesla = new TeslaModel3();
        tesla.start();
        System.out.println("Needs charging: " + tesla.needsCharging()); // false (90%)
        tesla.charge(25);

        // Treat Tesla as just Vehicle:
        Vehicle v = tesla;
        v.start();
        v.stop();

        // Treat Tesla as ElectricVehicle:
        ElectricVehicle ev = tesla;
        System.out.println("Battery: " + ev.getBatteryLevel() + "%");
    }

    static void processPayment(PaymentProcessor processor, double amount) {
        boolean success = processor.process(amount);
        if (success) processor.onSuccess(amount);
        else processor.onFailure(amount);
    }
}
```

---

## 4. Abstract Class vs Interface — Decision Guide

```java
public class AbstractVsInterface {

    // ─────────────────────────────────────────
    // THE COMPARISON TABLE
    // ─────────────────────────────────────────
    //
    // Feature                 │ Abstract Class │ Interface
    // ────────────────────────┼────────────────┼──────────────────
    // Instantiation           │ ❌ Cannot      │ ❌ Cannot
    // Methods with body       │ ✅ Yes         │ ✅ default/static
    // Abstract methods        │ ✅ Yes         │ ✅ Yes (implicit)
    // Fields / State          │ ✅ Any type    │ ✅ Only constants
    // Constructor             │ ✅ Yes         │ ❌ No
    // Access modifiers        │ ✅ Any         │ Public only
    // Multiple inheritance    │ ❌ One parent  │ ✅ Many interfaces
    // IS-A relationship       │ Strong (extends) │ Can-do (implements)
    // Purpose                 │ Partial reuse  │ Pure contract
    //
    // ─────────────────────────────────────────

    // ─────────────────────────────────────────
    // WHEN TO USE ABSTRACT CLASS:
    //
    // ✅ Classes are closely related and share state
    // ✅ You want to provide common base implementation
    // ✅ Subclasses need to access protected fields/methods
    // ✅ Template Method pattern (skeleton algorithm)
    // ✅ Common constructor logic needed
    // ─────────────────────────────────────────

    abstract static class HttpHandler {
        protected final String path;
        protected final String method;

        // Shared state + constructor:
        public HttpHandler(String path, String method) {
            this.path = path;
            this.method = method;
        }

        // Template method — fixed algorithm:
        public final void handle(String requestBody) {
            logRequest();
            Object result = processRequest(requestBody); // subclass implements
            logResponse(result);
        }

        // Abstract: subclasses define specific logic:
        protected abstract Object processRequest(String requestBody);

        // Shared implementations:
        private void logRequest() {
            System.out.println("[REQUEST] " + method + " " + path);
        }
        private void logResponse(Object result) {
            System.out.println("[RESPONSE] " + result);
        }

        public String getPath() { return path; }
        public String getMethod() { return method; }
    }

    static class UserHandler extends HttpHandler {
        public UserHandler() { super("/api/users", "GET"); }

        @Override
        protected Object processRequest(String body) {
            return "{\"users\": [\"rahim\", \"karim\"]}";
        }
    }

    static class OrderHandler extends HttpHandler {
        public OrderHandler() { super("/api/orders", "POST"); }

        @Override
        protected Object processRequest(String body) {
            return "{\"orderId\": \"ORD-001\", \"status\": \"CREATED\"}";
        }
    }

    // ─────────────────────────────────────────
    // WHEN TO USE INTERFACE:
    //
    // ✅ Define a capability/behavior (can-do relationship)
    // ✅ Unrelated classes need same behavior
    // ✅ Multiple inheritance of behavior needed
    // ✅ Plugin/extension points (anyone can implement)
    // ✅ Type for dependency injection
    // ✅ API contracts (Spring @Repository, @Service)
    // ─────────────────────────────────────────

    // Completely unrelated classes sharing Exportable behavior:
    interface Exportable {
        String toJson();
        String toCsv();
        byte[] toBinary();
    }

    static class User implements Exportable {
        private String name, email;
        public User(String name, String email) { this.name=name; this.email=email; }

        @Override public String toJson() {
            return "{\"name\":\"" + name + "\",\"email\":\"" + email + "\"}";
        }
        @Override public String toCsv() { return name + "," + email; }
        @Override public byte[] toBinary() { return (name+email).getBytes(); }
    }

    static class Transaction implements Exportable {
        private String id;
        private double amount;
        public Transaction(String id, double amount) { this.id=id; this.amount=amount; }

        @Override public String toJson() {
            return "{\"id\":\"" + id + "\",\"amount\":" + amount + "}";
        }
        @Override public String toCsv() { return id + "," + amount; }
        @Override public byte[] toBinary() { return (id+amount).toString().getBytes(); }
    }

    // Export engine works with ANY Exportable:
    static void exportAll(java.util.List<Exportable> items, String format) {
        System.out.println("=== Exporting " + items.size() + " items as " + format + " ===");
        for (Exportable item : items) {
            System.out.println(switch (format) {
                case "JSON" -> item.toJson();
                case "CSV" -> item.toCsv();
                default -> "Unknown format";
            });
        }
    }

    // ─────────────────────────────────────────
    // COMBINING ABSTRACT CLASS + INTERFACE
    // ─────────────────────────────────────────

    interface DataSource {
        java.util.List<java.util.Map<String, Object>> query(String sql);
        int execute(String sql);
        boolean isConnected();
    }

    // Abstract class provides common behavior,
    // implements the interface contract:
    abstract static class BaseDataSource implements DataSource {
        protected String connectionString;
        protected boolean connected;

        public BaseDataSource(String connectionString) {
            this.connectionString = connectionString;
            this.connected = false;
        }

        // Common behavior:
        public void connect() {
            System.out.println("Connecting to: " + connectionString);
            this.connected = doConnect();
            if (connected) System.out.println("Connected!");
        }

        @Override
        public boolean isConnected() { return connected; }

        // Hook for subclasses:
        protected abstract boolean doConnect();

        // Default implementation for query logging:
        @Override
        public java.util.List<java.util.Map<String, Object>> query(String sql) {
            if (!connected) throw new IllegalStateException("Not connected");
            System.out.println("Executing query: " + sql);
            return doQuery(sql);
        }

        protected abstract java.util.List<java.util.Map<String, Object>> doQuery(String sql);
    }

    static class PostgresDataSource extends BaseDataSource {
        public PostgresDataSource(String host, int port, String db) {
            super("jdbc:postgresql://" + host + ":" + port + "/" + db);
        }

        @Override
        protected boolean doConnect() { return true; } // simulate success

        @Override
        protected java.util.List<java.util.Map<String, Object>> doQuery(String sql) {
            return new java.util.ArrayList<>(); // simulate result
        }

        @Override
        public int execute(String sql) {
            System.out.println("PostgreSQL executing: " + sql);
            return 1;
        }
    }

    public static void main(String[] args) {

        System.out.println("=== Abstract Class (Template Method) ===");
        HttpHandler[] handlers = { new UserHandler(), new OrderHandler() };
        for (HttpHandler h : handlers) {
            h.handle("{\"request\":\"data\"}");
        }

        System.out.println("\n=== Interface (Cross-Type Behavior) ===");
        java.util.List<Exportable> items = java.util.List.of(
            new User("Rahim", "rahim@test.com"),
            new Transaction("TXN-001", 1500.0),
            new User("Karim", "karim@test.com"),
            new Transaction("TXN-002", 3000.0)
        );
        exportAll(items, "JSON");
        System.out.println();
        exportAll(items, "CSV");

        System.out.println("\n=== Abstract Class + Interface Combined ===");
        PostgresDataSource ds = new PostgresDataSource("localhost", 5432, "myapp");
        ds.connect();

        DataSource source = ds; // interface reference
        source.query("SELECT * FROM users LIMIT 10");
        source.execute("INSERT INTO logs VALUES (...)");
        System.out.println("Is connected: " + source.isConnected());
    }
}
```

---

## 5. Marker Interfaces and Sealed Interfaces

```java
public class SpecialInterfaces {

    // ─────────────────────────────────────────
    // MARKER INTERFACE — No methods, marks a class
    // Examples: Serializable, Cloneable, RandomAccess
    // ─────────────────────────────────────────

    // Used to tag classes with metadata:
    interface Auditable { } // no methods — just a tag
    interface Cacheable { } // no methods — just a tag

    static class UserEntity implements Auditable {
        private String name;
        public UserEntity(String name) { this.name = name; }
        public String getName() { return name; }
    }

    static class ProductEntity implements Auditable, Cacheable {
        private String sku;
        public ProductEntity(String sku) { this.sku = sku; }
        public String getSku() { return sku; }
    }

    static class OrderEntity { // NOT auditable
        private String id;
        public OrderEntity(String id) { this.id = id; }
    }

    // Check at runtime using instanceof:
    static void process(Object entity) {
        System.out.print("Processing " + entity.getClass().getSimpleName() + ": ");
        if (entity instanceof Auditable) {
            System.out.print("[AUDITED] ");
        }
        if (entity instanceof Cacheable) {
            System.out.print("[CACHED] ");
        }
        System.out.println();
    }

    // ─────────────────────────────────────────
    // SEALED INTERFACE (Java 17+)
    // Restricts which classes can implement it
    // Enables exhaustive pattern matching
    // ─────────────────────────────────────────

    // Only these classes can implement ApiResponse:
    sealed interface ApiResponse
        permits SuccessResponse, ErrorResponse, PendingResponse {
        int getStatusCode();
        String getMessage();
    }

    // Permitted implementations:
    record SuccessResponse(int statusCode, String message, Object data)
        implements ApiResponse { }

    record ErrorResponse(int statusCode, String message, String errorCode)
        implements ApiResponse { }

    record PendingResponse(int statusCode, String message, String jobId)
        implements ApiResponse { }

    // Exhaustive pattern matching — no default needed:
    static void handleResponse(ApiResponse response) {
        String description = switch (response) {
            case SuccessResponse s -> "✅ Success [" + s.statusCode() + "]: " +
                                      s.message() + " | Data: " + s.data();
            case ErrorResponse e -> "❌ Error [" + e.statusCode() + "]: " +
                                     e.message() + " | Code: " + e.errorCode();
            case PendingResponse p -> "⏳ Pending [" + p.statusCode() + "]: " +
                                      p.message() + " | Job: " + p.jobId();
            // No default needed! Compiler knows all possibilities.
            // Add a new permitted type → compiler forces you to handle it here!
        };
        System.out.println(description);
    }

    public static void main(String[] args) {

        System.out.println("=== Marker Interfaces ===");
        process(new UserEntity("Rahim"));
        process(new ProductEntity("SKU-001"));
        process(new OrderEntity("ORD-001"));

        System.out.println("\n=== Sealed Interface + Pattern Matching ===");
        ApiResponse[] responses = {
            new SuccessResponse(200, "User created", "{\"id\": 1}"),
            new ErrorResponse(404, "User not found", "USER_NOT_FOUND"),
            new PendingResponse(202, "Processing...", "job-abc123"),
            new ErrorResponse(400, "Validation failed", "INVALID_EMAIL"),
            new SuccessResponse(200, "OK", null)
        };

        for (ApiResponse response : responses) {
            handleResponse(response);
        }

        // Sealed + records = powerful combination for API design
        System.out.println("\n=== Type safety in sealed hierarchy ===");

        // Compiler knows all types — no surprises:
        ApiResponse resp = new SuccessResponse(201, "Created", "data");
        if (resp instanceof SuccessResponse s) {
            System.out.println("Success with data: " + s.data());
        }
    }
}
```

---

## 6. Design with Abstraction — Real Spring Boot Patterns

```java
// These patterns appear in EVERY professional Spring Boot codebase.
// Understanding abstraction makes these patterns obvious.
public class SpringAbstractionPatterns {

    // ─────────────────────────────────────────
    // PATTERN 1: Repository abstraction
    // Spring Data JPA is built exactly like this
    // ─────────────────────────────────────────

    // The contract — what ALL repositories must do:
    interface Repository<T, ID> {
        T save(T entity);
        java.util.Optional<T> findById(ID id);
        java.util.List<T> findAll();
        void deleteById(ID id);
        boolean existsById(ID id);
        long count();
    }

    static class Product {
        private Long id;
        private String name;
        private double price;

        public Product(Long id, String name, double price) {
            this.id = id; this.name = name; this.price = price;
        }
        public Long getId() { return id; }
        public String getName() { return name; }
        public double getPrice(){ return price; }

        @Override
        public String toString() {
            return "Product{id=" + id + ", name='" + name + "', price=" + price + "}";
        }
    }

    // Specific repository:
    interface ProductRepository extends Repository<Product, Long> {
        java.util.List<Product> findByPriceBelow(double maxPrice);
        java.util.List<Product> findByName(String name);
    }

    // In-memory implementation (for tests):
    static class InMemoryProductRepository implements ProductRepository {
        private final java.util.Map<Long, Product> store = new java.util.HashMap<>();
        private long nextId = 1;

        @Override
        public Product save(Product p) {
            if (p.getId() == null) {
                Product withId = new Product(nextId++, p.getName(), p.getPrice());
                store.put(withId.getId(), withId);
                return withId;
            }
            store.put(p.getId(), p);
            return p;
        }

        @Override
        public java.util.Optional<Product> findById(Long id) {
            return java.util.Optional.ofNullable(store.get(id));
        }

        @Override
        public java.util.List<Product> findAll() {
            return new java.util.ArrayList<>(store.values());
        }

        @Override
        public void deleteById(Long id) { store.remove(id); }

        @Override
        public boolean existsById(Long id) { return store.containsKey(id); }

        @Override
        public long count() { return store.size(); }

        @Override
        public java.util.List<Product> findByPriceBelow(double max) {
            java.util.List<Product> result = new java.util.ArrayList<>();
            for (Product p : store.values()) {
                if (p.getPrice() < max) result.add(p);
            }
            return result;
        }

        @Override
        public java.util.List<Product> findByName(String name) {
            java.util.List<Product> result = new java.util.ArrayList<>();
            for (Product p : store.values()) {
                if (p.getName().equalsIgnoreCase(name)) result.add(p);
            }
            return result;
        }
    }

    // ─────────────────────────────────────────
    // PATTERN 2: Service abstraction
    // ─────────────────────────────────────────

    interface ProductService {
        Product createProduct(String name, double price);
        Product getProduct(Long id);
        java.util.List<Product> getAffordable(double budget);
        void deleteProduct(Long id);
    }

    static class ProductServiceImpl implements ProductService {
        private final ProductRepository repository; // interface dependency!

        public ProductServiceImpl(ProductRepository repository) {
            this.repository = repository;
        }

        @Override
        public Product createProduct(String name, double price) {
            if (name == null || name.isBlank())
                throw new IllegalArgumentException("Name required");
            if (price <= 0)
                throw new IllegalArgumentException("Price must be positive");
            return repository.save(new Product(null, name.trim(), price));
        }

        @Override
        public Product getProduct(Long id) {
            return repository.findById(id)
                .orElseThrow(() -> new RuntimeException("Product not found: " + id));
        }

        @Override
        public java.util.List<Product> getAffordable(double budget) {
            return repository.findByPriceBelow(budget);
        }

        @Override
        public void deleteProduct(Long id) {
            if (!repository.existsById(id))
                throw new RuntimeException("Product not found: " + id);
            repository.deleteById(id);
        }
    }

    // ─────────────────────────────────────────
    // PATTERN 3: Notification abstraction
    // (Simplified version of Spring's ApplicationEvent)
    // ─────────────────────────────────────────

    interface NotificationChannel {
        void send(String recipient, String subject, String message);
        boolean isAvailable();
        String getChannelName();
    }

    static class EmailChannel implements NotificationChannel {
        @Override public void send(String r, String s, String m) {
            System.out.println("📧 EMAIL to " + r + ": [" + s + "] " + m);
        }
        @Override public boolean isAvailable() { return true; }
        @Override public String getChannelName() { return "EMAIL"; }
    }

    static class SmsChannel implements NotificationChannel {
        @Override public void send(String r, String s, String m) {
            System.out.println("📱 SMS to " + r + ": " + m.substring(0, Math.min(160, m.length())));
        }
        @Override public boolean isAvailable() { return true; }
        @Override public String getChannelName() { return "SMS"; }
    }

    static class SlackChannel implements NotificationChannel {
        private boolean available;

        public SlackChannel(boolean available) { this.available = available; }

        @Override public void send(String r, String s, String m) {
            System.out.println("💬 SLACK to #" + r + ": " + s + " - " + m);
        }
        @Override public boolean isAvailable() { return available; }
        @Override public String getChannelName() { return "SLACK"; }
    }

    static class NotificationService {
        private final java.util.List<NotificationChannel> channels;

        public NotificationService(java.util.List<NotificationChannel> channels) {
            this.channels = channels;
        }

        // Sends through ALL available channels:
        public void notifyAll(String recipient, String subject, String message) {
            for (NotificationChannel channel : channels) {
                if (channel.isAvailable()) {
                    channel.send(recipient, subject, message); // polymorphic!
                } else {
                    System.out.println("⚠️ " + channel.getChannelName() + " unavailable");
                }
            }
        }

        // Sends through a SPECIFIC channel type:
        public void notifyVia(String channelName, String recipient,
                              String subject, String message) {
            channels.stream()
                .filter(c -> c.getChannelName().equals(channelName))
                .filter(NotificationChannel::isAvailable)
                .findFirst()
                .ifPresentOrElse(
                    c -> c.send(recipient, subject, message),
                    () -> System.out.println("Channel not found: " + channelName));
        }
    }

    public static void main(String[] args) {

        System.out.println("=== Repository + Service Abstraction ===");

        // Inject in-memory repo (for demo — Spring would inject PostgreSQL):
        ProductRepository repo = new InMemoryProductRepository();
        ProductService service = new ProductServiceImpl(repo);

        service.createProduct("Laptop", 75000.0);
        service.createProduct("Phone", 45000.0);
        service.createProduct("Tablet", 35000.0);
        service.createProduct("Watch", 15000.0);
        service.createProduct("Earbuds", 8000.0);

        System.out.println("\nAll products:");
        repo.findAll().forEach(p ->
            System.out.println("  " + p));

        System.out.println("\nAffordable (under ৳40k):");
        service.getAffordable(40000).forEach(p ->
            System.out.println("  " + p));

        System.out.println("\nProduct #2: " + service.getProduct(2L));

        try {
            service.getProduct(99L); // doesn't exist
        } catch (RuntimeException e) {
            System.out.println("Not found: " + e.getMessage());
        }

        System.out.println("\n=== Notification Abstraction ===");

        NotificationService notifier = new NotificationService(
            java.util.List.of(
                new EmailChannel(),
                new SmsChannel(),
                new SlackChannel(false) // unavailable today
            )
        );

        System.out.println("--- Notifying all channels ---");
        notifier.notifyAll("rahim@test.com / 01712345678 / #engineering",
                           "Order Confirmed",
                           "Your order ORD-001 has been confirmed! Total: ৳1500.");

        System.out.println("\n--- Notify via EMAIL only ---");
        notifier.notifyVia("EMAIL", "karim@test.com",
                           "Welcome!", "Thanks for signing up!");

        System.out.println("\n--- Try unavailable channel ---");
        notifier.notifyVia("SLACK", "#devs", "Deploy", "Deployment complete");
    }
}
```

---

## Build This — Complete Abstraction Practice

```java
// File: AbstractionMasterpiece.java
// A complete system demonstrating abstraction at every level
import java.util.*;
import java.time.*;

public class AbstractionMasterpiece {
    // ═══════════════════════════════════════
    // LAYER 1: DATA INTERFACES
    // ═══════════════════════════════════════

    interface Identifiable {
        String getId();
    }

    interface Timestamped {
        LocalDateTime getCreatedAt();
        LocalDateTime getUpdatedAt();
    }

    interface Validatable {
        boolean isValid();
        List<String> getValidationErrors();
    }

    // Marker:
    interface Archivable { }
    // ═══════════════════════════════════════
    // LAYER 2: DOMAIN INTERFACES
    // ═══════════════════════════════════════

    interface OrderProcessor {
        Order processOrder(Cart cart, Customer customer);
        boolean cancelOrder(String orderId, String reason);
        Order getOrder(String orderId);
    }

    interface PriceCalculator {
        double calculateSubtotal(Cart cart);
        double calculateDiscount(Cart cart, Customer customer);
        double calculateTax(double amount);
        double calculateTotal(Cart cart, Customer customer);
    }

    interface InventoryManager {
        boolean isAvailable(String productId, int quantity);
        boolean reserve(String productId, int quantity);
        boolean release(String productId, int quantity);
        int getStock(String productId);
    }
    // ═══════════════════════════════════════
    // LAYER 3: ABSTRACT BASE
    // ═══════════════════════════════════════

    abstract static class BaseEntity implements Identifiable, Timestamped {
        protected final String id;
        protected final LocalDateTime createdAt;
        protected LocalDateTime updatedAt;

        protected BaseEntity() {
            this.id = java.util.UUID.randomUUID().toString().substring(0, 8);
            this.createdAt = LocalDateTime.now();
            this.updatedAt = LocalDateTime.now();
        }

        @Override public String getId() { return id; }
        @Override public LocalDateTime getCreatedAt() { return createdAt; }
        @Override public LocalDateTime getUpdatedAt() { return updatedAt; }

        protected void markUpdated() { updatedAt = LocalDateTime.now(); }
    }
    // ═══════════════════════════════════════
    // LAYER 4: CONCRETE ENTITIES
    // ═══════════════════════════════════════

    static class Product extends BaseEntity implements Archivable {
        private final String sku;
        private String name;
        private double price;
        private String category;

        public Product(String sku, String name, double price, String category) {
            super();
            this.sku = sku;
            this.name = name;
            this.price = price;
            this.category = category;
        }

        public String getSku() { return sku; }
        public String getName() { return name; }
        public double getPrice() { return price; }
        public String getCategory() { return category; }

        public void setPrice(double price) {
            if (price <= 0) throw new IllegalArgumentException("Price must be positive");
            this.price = price;
            markUpdated();
        }

        @Override
        public String toString() {
            return String.format("%-30s SKU:%-10s ৳%,.2f", name, sku, price);
        }
    }

    static class CartItem {
        private final Product product;
        private int quantity;

        public CartItem(Product product, int quantity) {
            this.product = product;
            this.quantity = quantity;
        }

        public Product getProduct() { return product; }
        public int getQuantity() { return quantity; }
        public double getSubtotal() { return product.getPrice() * quantity; }

        public void setQuantity(int qty) {
            if (qty <= 0) throw new IllegalArgumentException("Quantity must be positive");
            this.quantity = qty;
        }

        @Override
        public String toString() {
            return String.format(" %-30s × %-4d = ৳%,.2f",
                                 product.getName(), quantity, getSubtotal());
        }
    }

    static class Cart extends BaseEntity {
        private final List<CartItem> items = new ArrayList<>();

        public void addItem(Product product, int quantity) {
            for (CartItem item : items) {
                if (item.getProduct().getSku().equals(product.getSku())) {
                    item.setQuantity(item.getQuantity() + quantity);
                    markUpdated();
                    return;
                }
            }
            items.add(new CartItem(product, quantity));
            markUpdated();
        }

        public boolean removeItem(String sku) {
            boolean removed = items.removeIf(i -> i.getProduct().getSku().equals(sku));
            if (removed) markUpdated();
            return removed;
        }

        public List<CartItem> getItems() { return Collections.unmodifiableList(items); }
        public boolean isEmpty() { return items.isEmpty(); }
        public int getTotalItems() { return items.stream().mapToInt(CartItem::getQuantity).sum(); }
    }

    static class Customer extends BaseEntity implements Validatable {
        private final String name;
        private final String email;
        private final String phone;
        private boolean isPremium;

        public Customer(String name, String email, String phone, boolean isPremium) {
            super();
            this.name = name;
            this.email = email;
            this.phone = phone;
            this.isPremium = isPremium;
        }

        @Override
        public boolean isValid() { return getValidationErrors().isEmpty(); }

        @Override
        public List<String> getValidationErrors() {
            List<String> errors = new ArrayList<>();
            if (name == null || name.isBlank()) errors.add("Name required");
            if (email == null || !email.contains("@")) errors.add("Valid email required");
            if (phone == null || phone.isBlank()) errors.add("Phone required");
            return errors;
        }

        public String getName() { return name; }
        public String getEmail() { return email; }
        public String getPhone() { return phone; }
        public boolean isPremium() { return isPremium; }

        @Override
        public String toString() {
            return String.format("Customer{name='%s', email='%s', premium=%b}",
                                 name, email, isPremium);
        }
    }

    static class Order extends BaseEntity {
        private final String customerId;
        private final List<CartItem> items;
        private final double subtotal;
        private final double discount;
        private final double tax;
        private final double total;
        private String status;
        private String cancelReason;

        Order(String customerId, List<CartItem> items,
              double subtotal, double discount, double tax) {
            super();
            this.customerId = customerId;
            this.items = new ArrayList<>(items);
            this.subtotal = subtotal;
            this.discount = discount;
            this.tax = tax;
            this.total = subtotal - discount + tax;
            this.status = "PLACED";
        }

        public String getStatus() { return status; }
        public double getTotal() { return total; }
        public String getCustomerId() { return customerId; }

        void cancel(String reason) {
            status = "CANCELLED";
            cancelReason = reason;
            markUpdated();
        }

        public void printReceipt() {
            System.out.println("┌─────────────────────────────────────────────┐");
            System.out.printf( "│ ORDER: %-36s│%n", id);
            System.out.printf( "│ Status: %-35s│%n", status);
            System.out.println("├─────────────────────────────────────────────┤");
            for (CartItem item : items) {
                System.out.printf("│%s│%n",
                    item.toString().length() >= 45
                    ? item.toString().substring(0, 45)
                    : String.format("%-45s", item.toString()));
            }
            System.out.println("├─────────────────────────────────────────────┤");
            System.out.printf( "│ Subtotal: %-34s│%n", String.format("৳%,.2f", subtotal));
            if (discount > 0)
                System.out.printf("│ Discount: %-34s│%n", String.format("-৳%,.2f", discount));
            System.out.printf( "│ Tax: %-34s│%n", String.format("৳%,.2f", tax));
            System.out.printf( "│ TOTAL: %-34s│%n", String.format("৳%,.2f", total));
            System.out.println("└─────────────────────────────────────────────┘");
        }
    }
    // ═══════════════════════════════════════
    // LAYER 5: IMPLEMENTATIONS
    // ═══════════════════════════════════════

    static class StandardPriceCalculator implements PriceCalculator {
        private static final double TAX_RATE = 0.15;
        private static final double PREMIUM_DISC = 0.10;
        private static final double BULK_DISC = 0.05; // 10+ items

        @Override
        public double calculateSubtotal(Cart cart) {
            return cart.getItems().stream()
                .mapToDouble(CartItem::getSubtotal)
                .sum();
        }

        @Override
        public double calculateDiscount(Cart cart, Customer customer) {
            double subtotal = calculateSubtotal(cart);
            double discount = 0;
            if (customer.isPremium()) discount += subtotal * PREMIUM_DISC;
            if (cart.getTotalItems() >= 10) discount += subtotal * BULK_DISC;
            return discount;
        }

        @Override
        public double calculateTax(double amount) {
            return Math.round(amount * TAX_RATE * 100) / 100.0;
        }

        @Override
        public double calculateTotal(Cart cart, Customer customer) {
            double sub = calculateSubtotal(cart);
            double disc = calculateDiscount(cart, customer);
            double tax = calculateTax(sub - disc);
            return sub - disc + tax;
        }
    }

    static class SimpleInventoryManager implements InventoryManager {
        private final Map<String, Integer> stock = new HashMap<>();
        private final Map<String, Integer> reserved = new HashMap<>();

        public void setStock(String sku, int quantity) {
            stock.put(sku, quantity);
            reserved.put(sku, 0);
        }

        @Override
        public boolean isAvailable(String sku, int qty) {
            int available = stock.getOrDefault(sku, 0) - reserved.getOrDefault(sku, 0);
            return available >= qty;
        }

        @Override
        public boolean reserve(String sku, int qty) {
            if (!isAvailable(sku, qty)) return false;
            reserved.merge(sku, qty, Integer::sum);
            return true;
        }

        @Override
        public boolean release(String sku, int qty) {
            int current = reserved.getOrDefault(sku, 0);
            if (current < qty) return false;
            reserved.put(sku, current - qty);
            return true;
        }

        @Override
        public int getStock(String sku) {
            return stock.getOrDefault(sku, 0) - reserved.getOrDefault(sku, 0);
        }
    }

    static class EcommerceOrderProcessor implements OrderProcessor {
        private final PriceCalculator calculator;
        private final InventoryManager inventory;
        private final Map<String, Order> orders = new HashMap<>();

        // Dependencies injected as INTERFACES — not concrete types:
        public EcommerceOrderProcessor(PriceCalculator calculator,
                                       InventoryManager inventory) {
            this.calculator = calculator;
            this.inventory = inventory;
        }

        @Override
        public Order processOrder(Cart cart, Customer customer) {
            System.out.println("Processing order for: " + customer.getName());

            if (!customer.isValid()) {
                System.out.println("❌ Invalid customer: " + customer.getValidationErrors());
                return null;
            }
            if (cart.isEmpty()) {
                System.out.println("❌ Cart is empty");
                return null;
            }

            // Reserve inventory for all items:
            List<String> reserved = new ArrayList<>();
            for (CartItem item : cart.getItems()) {
                String sku = item.getProduct().getSku();
                int qty = item.getQuantity();

                if (!inventory.isAvailable(sku, qty)) {
                    // Rollback any reservations already made:
                    for (String r : reserved) inventory.release(r, qty);
                    System.out.println("❌ Out of stock: " + item.getProduct().getName());
                    return null;
                }
                inventory.reserve(sku, qty);
                reserved.add(sku);
            }

            // Calculate pricing:
            double subtotal = calculator.calculateSubtotal(cart);
            double discount = calculator.calculateDiscount(cart, customer);
            double tax = calculator.calculateTax(subtotal - discount);

            Order order = new Order(customer.getId(), cart.getItems(),
                                    subtotal, discount, tax);
            orders.put(order.getId(), order);

            System.out.println("✅ Order created: " + order.getId());
            return order;
        }

        @Override
        public boolean cancelOrder(String orderId, String reason) {
            Order order = orders.get(orderId);
            if (order == null) { System.out.println("Order not found"); return false; }
            if ("CANCELLED".equals(order.getStatus())) {
                System.out.println("Already cancelled");
                return false;
            }

            // Release reserved inventory:
            // (simplified — in real code we'd track the reservation)

            order.cancel(reason);
            System.out.println("✅ Order " + orderId + " cancelled: " + reason);
            return true;
        }

        @Override
        public Order getOrder(String orderId) {
            return orders.get(orderId);
        }
    }
    // ═══════════════════════════════════════
    // MAIN
    // ═══════════════════════════════════════

    public static void main(String[] args) {

        // Setup inventory:
        SimpleInventoryManager inventory = new SimpleInventoryManager();
        inventory.setStock("LAPTOP-PRO", 5);
        inventory.setStock("PHONE-S24", 20);
        inventory.setStock("TABLET-X1", 8);
        inventory.setStock("WATCH-GT4", 15);
        inventory.setStock("EARBUDS-W1", 50);

        // Products:
        Product laptop = new Product("LAPTOP-PRO", "Laptop Pro 14\" i7", 95000.0, "Electronics");
        Product phone = new Product("PHONE-S24", "Smartphone S24 Ultra", 145000.0, "Electronics");
        Product tablet = new Product("TABLET-X1", "Tablet X1 12\"", 55000.0, "Electronics");
        Product watch = new Product("WATCH-GT4", "SmartWatch GT4", 18000.0, "Wearables");
        Product earbuds = new Product("EARBUDS-W1", "Wireless Earbuds W1", 8500.0, "Audio");

        // Customers:
        Customer rahim = new Customer("Rahim Ahmed", "rahim@test.com", "01712345678", true);
        Customer karim = new Customer("Karim Hassan", "karim@test.com", "01812345678", false);
        Customer invalid = new Customer("", "notanemail", "", false);

        // Services — ALL via interfaces:
        PriceCalculator calculator = new StandardPriceCalculator();
        OrderProcessor processor = new EcommerceOrderProcessor(calculator, inventory);

        System.out.println("╔══════════════════════════════════════════╗");
        System.out.println("║     ECOMMERCE ORDER PROCESSING           ║");
        System.out.println("╚══════════════════════════════════════════╝\n");

        // Order 1: Premium customer
        System.out.println("── Order 1: Premium Customer ──");
        Cart cart1 = new Cart();
        cart1.addItem(laptop, 1);
        cart1.addItem(watch, 2);
        cart1.addItem(earbuds, 3);

        Order order1 = processor.processOrder(cart1, rahim);
        if (order1 != null) order1.printReceipt();

        // Order 2: Regular customer
        System.out.println("\n── Order 2: Regular Customer ──");
        Cart cart2 = new Cart();
        cart2.addItem(phone, 1);
        cart2.addItem(tablet, 1);

        Order order2 = processor.processOrder(cart2, karim);
        if (order2 != null) order2.printReceipt();

        // Order 3: Invalid customer
        System.out.println("\n── Order 3: Invalid Customer ──");
        Cart cart3 = new Cart();
        cart3.addItem(earbuds, 1);
        processor.processOrder(cart3, invalid);

        // Order 4: Out of stock
        System.out.println("\n── Order 4: Inventory check ──");
        Cart cart4 = new Cart();
        cart4.addItem(laptop, 10); // only 5 minus what rahim took
        processor.processOrder(cart4, karim);

        // Cancel order 1:
        System.out.println("\n── Cancellation ──");
        if (order1 != null) {
            processor.cancelOrder(order1.getId(), "Customer changed mind");
        }

        // Inventory after orders:
        System.out.println("\n── Stock Levels ──");
        for (Product p : List.of(laptop, phone, tablet, watch, earbuds)) {
            System.out.printf("%-30s: %d units%n",
                              p.getName(), inventory.getStock(p.getSku()));
        }
    }
}
```

---

## Exercises

```text
EXERCISE 1: Plugin System
  Create PluginSystem.java
  Design a plugin architecture:

  interface Plugin {
    String getName();
    String getVersion();
    void initialize();
    void execute(Map<String, Object> context);
    void cleanup();
    boolean isEnabled();
  }

  abstract class BasePlugin implements Plugin {
    // Common lifecycle management
    // isEnabled defaults to true
    // Template method pattern for execute()
  }

  Implement 5 plugins:
  - LoggingPlugin → logs context to console
  - CachePlugin → caches results in Map
  - ValidationPlugin → validates context fields
  - MetricsPlugin → counts executions, tracks timing
  - TransformPlugin → transforms values in context

  PluginManager:
  - register(Plugin p)
  - unregister(String name)
  - executeAll(Map<String, Object> context)
  - getEnabled()
  - disable(String name)

  Run a complete demo.

EXERCISE 2: Abstract Data Types
  Create AbstractDataTypes.java
  Implement these abstract data structures:

  interface Stack<T> {
    void push(T item);
    T pop();
    T peek();
    boolean isEmpty();
    int size();
  }

  interface Queue<T> {
    void enqueue(T item);
    T dequeue();
    T front();
    boolean isEmpty();
    int size();
  }

  Implementations:
  - ArrayStack — backed by array
  - LinkedStack — backed by linked list nodes
  - ArrayQueue — backed by circular array
  - PriorityQueue<T extends Comparable<T>> — min heap

  Test each with same data.
  Show polymorphic usage.

EXERCISE 3: Sealed Hierarchy
  Create OrderStatusMachine.java
  Model order states as sealed interface:

  sealed interface OrderState permits
    Pending, Confirmed, Processing, Shipped, Delivered, Cancelled, Refunded

  Each state as record with relevant data:
  - Pending: createdAt
  - Confirmed: confirmedAt, estimatedDelivery
  - Processing: startedAt, warehouseId
  - Shipped: shippedAt, trackingNumber, carrier
  - Delivered: deliveredAt, signedBy
  - Cancelled: cancelledAt, reason, cancelledBy
  - Refunded: refundedAt, amount, refundId

  State machine:
  - transitions(OrderState current) → List<Class<? extends OrderState>>
  - transition(OrderState from, Class to) → OrderState
  - describe(OrderState state) → String (exhaustive pattern matching, no default!)

  Simulate a complete order journey.

EXERCISE 4: Generic Repository
  Create GenericRepository.java

  interface Repository<T, ID> {
    T save(T entity);
    Optional<T> findById(ID id);
    List<T> findAll();
    List<T> findWhere(Predicate<T> condition);
    void deleteById(ID id);
    long count();
  }

  abstract class InMemoryRepository<T, ID>
    implements Repository<T, ID> {
    // Common in-memory storage logic
    // Abstract: getId(T entity)
  }

  Implement for: Product, Customer, Order
  Use the repositories in a small service.

EXERCISE 5: Observer + Abstract
  Create EventDrivenSystem.java

  abstract class Event {
    final String id = UUID.randomUUID().toString();
    final LocalDateTime timestamp = LocalDateTime.now();
    abstract String getEventType();
    abstract String getSummary();
  }

  interface EventListener<T extends Event> {
    void onEvent(T event);
    boolean canHandle(Event event);
    default int getPriority() { return 0; }
  }

  abstract class BaseEventListener<T extends Event>
    implements EventListener<T> {
    protected abstract Class<T> getSupportedEventType();

    @Override
    public boolean canHandle(Event event) {
      return getSupportedEventType().isInstance(event);
    }
  }

  Events: UserRegistered, OrderPlaced, PaymentProcessed,
          InventoryUpdated, EmailFailed

  3+ listeners per event.
  EventDispatcher with priority ordering.
  Push to GitHub: "feat: abstraction masterpiece ecommerce system"
```

---

## Common Mistakes

```text
MISTAKE 1: Instantiating abstract class or interface
  new AbstractClass(); // COMPILE ERROR
  new Interface(); // COMPILE ERROR
  Fix: new ConcreteSubclass(); // implement the abstract class/interface

MISTAKE 2: Forgetting to implement all abstract methods
  abstract class A { abstract void doA(); abstract void doB(); }
  class B extends A { void doA() { } } // COMPILE ERROR: doB() not implemented
  Fix: implement ALL abstract methods, or make B also abstract.

MISTAKE 3: Modifying interface constants
  interface Config {
    int MAX = 100; // implicitly: public static final
  }
  Config.MAX = 200; // COMPILE ERROR: final!
  Fix: constants are final. Use class with static fields for mutable config.

MISTAKE 4: Trying to have state in interface
  interface Stateful {
    int count = 0; // public static final — shared constant, not instance state!
    void increment(); // can't have instance state here
  }
  Fix: use abstract class for shared mutable state.

MISTAKE 5: Not checking @Override on abstract method implementation
  abstract class A { abstract void process(); }
  class B extends A {
    void processs() { } // typo! NOT implementing process()!
    // COMPILE ERROR: B is abstract (process() not implemented)
    // @Override would have caught the typo
  }
  Fix: always @Override when implementing abstract methods.

MISTAKE 6: Using abstract class when interface is sufficient
  abstract class Flyable { abstract void fly(); }
  // If no state or shared implementation: use interface!
  interface Flyable { void fly(); }
  Fix: prefer interface unless shared state or protected methods needed.

MISTAKE 7: Deep abstract class hierarchy
  AbstractBase → AbstractMiddle → AbstractTop → Concrete
  Hard to understand. Hard to change. Hard to test.
  Fix: max 2 levels of abstraction. Extract interfaces.

MISTAKE 8: Sealed interface without handling all cases
  sealed interface Shape permits Circle, Rectangle { }

  // COMPILE ERROR: switch must be exhaustive:
  switch (shape) {
    case Circle c -> "circle"; // missing Rectangle!
  }
  Fix: handle all permitted types, or add default.

MISTAKE 9: Making every interface a functional interface
  @FunctionalInterface
  interface UserValidator { // too narrow!
    boolean validate(User user);
    boolean validateEmail(String email); // COMPILE ERROR: two abstract methods
  }
  Fix: @FunctionalInterface only for intentionally single-method interfaces.

MISTAKE 10: Private methods in interface not being callable
  interface Service {
    private void helper() { } // Java 9+
    default void doWork() { helper(); } // OK — default can call private
  }
  class Impl implements Service {
    // helper() is NOT inherited — it's private to the interface
    // Cannot call helper() here
  }
  Fix: private interface methods are internal helpers for default methods only.
```

---

## Interview Questions

**Q: What is abstraction in OOP? How is it different from encapsulation?**
A: Abstraction focuses on WHAT an object does — its behavior and interface — hiding the implementation details. It answers: "What can this do?" Encapsulation focuses on HOW data is protected — bundling data and methods, restricting direct access. It answers: "How is this data protected?" Example: A Car interface (abstraction) exposes start(), stop(), accelerate(). Encapsulation hides the engine's internal state behind private fields and validated setters. Together, they create clean, maintainable APIs. Abstraction is achieved through abstract classes and interfaces. Encapsulation through access modifiers and getters/setters.

**Q: When would you use an abstract class vs an interface?**
A: Use INTERFACE when: - You're defining a pure capability/behavior contract - Unrelated classes need to share the same behavior - Multiple inheritance of behavior is needed - You want a stable public API (anyone can implement) - Spring dependency injection type (always program to interface)
Use ABSTRACT CLASS when: - Closely related classes share common state AND behavior - You need shared constructor logic - Protected fields/methods are needed by subclasses - Template Method pattern (fixed algorithm skeleton) - Some methods have shared default behavior; others are abstract
Modern rule: prefer interfaces. Add abstract class only when you genuinely need shared state or protected methods.

**Q: What is a functional interface and why does it matter?**
A: A functional interface has exactly ONE abstract method (though it can have default and static methods). The @FunctionalInterface annotation enforces this at compile time. It matters because functional interfaces can be implemented with lambda expressions instead of verbose anonymous inner classes. java.util.function package has built-ins: Predicate, Function, Consumer, Supplier. In Spring Boot: @Bean factories, event handlers, Spring Data specifications — all use functional interfaces with lambdas. Functional interfaces bridge OOP and functional programming in Java.

**Q: What are default methods in interfaces? Why were they added?**
A: Default methods (Java 8+) are interface methods with a body, implemented using the 'default' keyword. They can be overridden by implementing classes or inherited as-is. They were added to enable backward compatibility: adding new methods to existing interfaces without breaking all implementing classes. Before Java 8: adding a method to an interface broke ALL implementations. With default methods: existing code continues to work, getting the default behavior. If a class implements two interfaces with the same default method, it MUST override to resolve the conflict. Classes always win over interfaces — a class method beats a default method.

**Q: What are sealed classes/interfaces? What problem do they solve?**
A: Sealed classes/interfaces (Java 17+) restrict which classes can extend/implement them using the 'permits' clause. Problem solved: previously, any class anywhere could extend an open class/interface — making exhaustive handling of all subtypes impossible. With sealed: the compiler knows ALL permitted subtypes at compile time, enabling exhaustive pattern matching in switch expressions with no 'default' case needed. If you add a new permitted type, all switch expressions must be updated → compiler enforces it. Use cases: modeling finite state machines (OrderStatus), algebraic data types (Result), discriminated unions, sealed API response types (SuccessResponse | ErrorResponse).

**Q: Explain the principle "program to an interface, not an implementation." Why is it important?**
A: Instead of: PostgreSQLUserRepository repo = new PostgreSQLUserRepository() Write: UserRepository repo = new PostgreSQLUserRepository()
Why important: 1) FLEXIBILITY: Switch from PostgreSQL to MongoDB by changing one line (the constructor call). All code using the repo variable works unchanged. 2) TESTABILITY: In tests, inject InMemoryUserRepository — fast, no database needed. Production code never changes. 3) DEPENDENCY INJECTION: Spring Boot injects implementations automatically — you declare the interface, Spring provides the concrete class. 4) OPEN/CLOSED PRINCIPLE: Add new implementations without modifying code that uses the interface. 5) TEAM COLLABORATION: Define the interface (contract) first. Different team members implement it. Others code against the interface while implementation is being built. This is the foundation of Spring's entire dependency injection system.

---

## Key Takeaways

```text
1. ABSTRACTION = WHAT (not HOW).
   Interfaces and abstract classes define the contract.
   Implementations provide the details.
   Users of abstractions never need to know the details.

2. ABSTRACT CLASS — partial abstraction:
   Has abstract methods (must override) AND concrete methods.
   Can have state (fields), constructors.
   Single inheritance only.
   Use for: related classes sharing state + behavior.
   Template Method pattern: final skeleton + abstract hooks.

3. INTERFACE — pure abstraction:
   Defines a contract — WHAT classes can do.
   Fields = constants (public static final).
   Methods = public abstract (default: Java 8+ has body).
   Can implement MULTIPLE interfaces.
   Use for: capabilities, unrelated classes, dependency injection.

4. FUNCTIONAL INTERFACE:
   Exactly ONE abstract method.
   @FunctionalInterface enforces this.
   Can be implemented with lambda expressions.
   Built-ins: Predicate, Function, Consumer, Supplier.

5. DEFAULT METHODS (Java 8+):
   Interface methods with body — for backward compatibility.
   Implementing class can override or inherit.
   Conflict (two interfaces same default): class MUST override.

6. SEALED INTERFACES (Java 17+):
   Restrict which classes can implement.
   Enables exhaustive switch pattern matching.
   Compiler enforces all cases handled.
   Use for: finite state machines, discriminated unions.

7. ABSTRACT vs INTERFACE decision:
   Has state? → Abstract class
   Pure behavior contract? → Interface
   Multiple inheritance needed? → Interface
   Template Method? → Abstract class
   Dependency injection type? → Interface (always)

8. "Program to interface, not implementation":
   UserRepository repo = new PostgreSQLRepo(); // correct
   NOT: PostgreSQLRepo repo = new PostgreSQLRepo(); // too specific
   This enables: swapping, testing, Spring DI, flexibility.

9. In Spring Boot — interfaces EVERYWHERE:
   @Repository extends JpaRepository (interface)
   @Service depends on Repository (interface)
   @Controller depends on Service (interface)
   Everything wired by Spring through interfaces.

10. Abstraction + Polymorphism + Interfaces:
    The holy trinity of flexible, maintainable Java design.
    Together they enable the Open/Closed Principle:
    Open for extension (new implementations),
    Closed for modification (existing code unchanged).
```

---