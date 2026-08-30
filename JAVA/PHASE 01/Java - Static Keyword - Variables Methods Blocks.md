---
title: "Java - Static Keyword - Variables Methods Blocks"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - static
  - class-members
status: "not-started"
---

# Java - Static Keyword - Variables Methods Blocks

> [!abstract] Overview
> The `static` keyword in Java marks a member as belonging to the **class itself** rather than to any individual object (instance). Static variables are shared across all instances of a class. Static methods can be called without creating an object. Static blocks execute once when the class is first loaded by the JVM. In backend development, `static` is used for constants, utility methods, configuration values, factory methods, singleton patterns, and Spring's static helper methods. Understanding `static` deeply is essential because misusing it is one of the most common sources of concurrency bugs in multi-threaded backend servers.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Constructors - Default Parameterized Copy]]
> - [[Java - Encapsulation - Getters Setters Access Modifiers]]
> - [[Java - Abstraction - Abstract Classes and Interfaces]]

---

## Theory

### What Does `static` Mean?

In Java, every member of a class (field, method, nested class, or initializer block) is either an **instance member** or a **class member**. The `static` keyword makes a member a class member.

- **Instance members** belong to individual objects. Each object has its own copy of instance fields, and instance methods operate on the specific object they are called on (via the `this` reference).

- **Static (class) members** belong to the class itself. There is exactly one copy of each static field, shared by all objects of that class. Static methods do not have a `this` reference and cannot access instance members directly.

The analogy: think of a university. The **university name** is a static property. There is only one name, shared by all students. Each student's **name** is an instance property. Every student has their own name. Changing the university name affects everyone. Changing one student's name affects only that student.

### Static Variables (Class Variables)

A static variable is declared with the `static` keyword. The JVM allocates memory for it once when the class is loaded, and all instances of the class share the same variable.

```java
public class User {
    // Instance variable: each User object has its own copy
    private String username;

    // Static variable: shared by ALL User objects
    private static int totalUsers = 0;

    public User(String username) {
        this.username = username;
        totalUsers++;  // Incremented every time a new User is created
    }

    public static int getTotalUsers() {
        return totalUsers;
    }
}

User u1 = new User("saad");
User u2 = new User("rahim");
User u3 = new User("karim");

System.out.println(User.getTotalUsers());  // 3
// All three User objects share the same totalUsers variable.
```

**When to use static variables:**

- **Constants**: Values that never change and are shared across the application. Declared as `public static final`.

  ```java
  public static final double TAX_RATE = 0.15;
  public static final String DEFAULT_CURRENCY = "BDT";
  ```

- **Counters and caches**: Tracking the number of instances, caching shared data, or maintaining application-wide state. Use with extreme caution in multi-threaded environments (more on this later).

- **Configuration**: Values loaded once at startup and read by all instances.

**When NOT to use static variables:**

- Do not use static variables to store per-request or per-user data in a backend server. Since static variables are shared across all threads, one user's data will overwrite another user's data. This is the most common concurrency bug in Java backends.

### Static Methods

A static method belongs to the class, not to any object. It can be called using the class name without creating an instance.

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class MathUtils {
    // Static method: called as MathUtils.calculateTax(...)
    // No object creation needed.
    public static BigDecimal calculateTax(BigDecimal amount, BigDecimal rate) {
        return amount.multiply(rate).setScale(2, RoundingMode.HALF_UP);
    }
}

// Usage:
BigDecimal tax = MathUtils.calculateTax(new BigDecimal("1000"), new BigDecimal("0.15"));
```

**Rules for static methods:**

1. A static method **cannot** access instance variables or call instance methods directly. It has no `this` reference because it is not associated with any object.
2. A static method **can** access other static variables and call other static methods of the same class.
3. A static method **can** be called through an object reference (`obj.staticMethod()`), but this is discouraged because it is misleading. Always call static methods through the class name.
4. Static methods **cannot** be overridden. If a subclass defines a static method with the same signature, it **hides** the parent's method rather than overriding it. The method that gets called is determined at compile time by the reference type, not at runtime by the object type.
5. Static methods **cannot** use the `this` or `super` keywords.

```java
public class Calculator {
    private int memory = 0;  // Instance variable

    // COMPILATION ERROR: static method cannot access instance variable
    public static int getMemory() {
        return memory;  // ERROR: non-static variable memory cannot be referenced from a static context
    }

    // OK: static method accessing static variable
    private static int operationCount = 0;

    public static int getOperationCount() {
        return operationCount;  // OK: both are static
    }
}
```

### Static Blocks (Static Initializers)

A static block is a block of code enclosed in `static { ... }` that executes **exactly once** when the class is first loaded by the JVM. It is used to initialize static variables that require complex logic beyond a simple assignment.

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

public class AppConfig {
    private static final Map<String, String> SETTINGS;

    // Static block: runs once when the class is loaded
    static {
        SETTINGS = new HashMap<>();
        try {
            Properties props = new Properties();
            props.load(AppConfig.class.getResourceAsStream("/config.properties"));
            for (String key : props.stringPropertyNames()) {
                SETTINGS.put(key, props.getProperty(key));
            }
        } catch (Exception e) {
            throw new RuntimeException("Failed to load configuration", e);
        }
    }

    public static String getSetting(String key) {
        return SETTINGS.get(key);
    }
}
```

**Key facts about static blocks:**

- A class can have multiple static blocks. They execute in the order they appear in the source code.
- Static blocks execute before any constructor is called and before any static method is invoked.
- Static blocks execute only once per classloader. In a typical Spring Boot application, this means they execute once at startup.
- If a static block throws an exception, the class fails to load, and any subsequent attempt to use the class throws a `NoClassDefFoundError`.

### Static Nested Classes

A static nested class is a class defined inside another class with the `static` keyword. Unlike inner classes (non-static nested classes), a static nested class does not have a reference to the enclosing class's instance.

```java
import java.math.BigDecimal;

public class Order {
    private String orderNumber;
    private BigDecimal total;

    // Static nested class: does NOT have access to Order's instance fields
    public static class OrderItem {
        private String productName;
        private int quantity;
        private BigDecimal unitPrice;

        public OrderItem(String productName, int quantity, BigDecimal unitPrice) {
            this.productName = productName;
            this.quantity = quantity;
            this.unitPrice = unitPrice;
        }

        public BigDecimal getSubtotal() {
            return unitPrice.multiply(new BigDecimal(quantity));
        }
    }
}

// Usage: no Order instance needed to create an OrderItem
Order.OrderItem item = new Order.OrderItem("Laptop", 1, new BigDecimal("85000"));
```

**When to use static nested classes:**

- When the nested class does not need access to the enclosing class's instance fields. This is the most common case.
- For grouping related classes together for organizational purposes (e.g., `Map.Entry`, `AbstractMap.SimpleEntry`).
- For builder patterns (e.g., `HttpRequest.Builder`).

**Static nested class vs inner class:**

| Feature | Static Nested Class | Inner Class (non-static) |
|---------|--------------------|-------------------------|
| Reference to outer instance | No | Yes (implicit `OuterClass.this`) |
| Can access outer instance fields | No (only static fields) | Yes (all fields) |
| Instantiation | `new Outer.Inner()` | `outer.new Inner()` |
| Memory overhead | None extra | Holds a reference to the outer object |
| Use case | Logical grouping, builders | Event handlers, callbacks |

### Static Imports

Static imports allow you to use static members of a class without qualifying them with the class name.

```java
// Without static import:
double result = Math.sqrt(Math.pow(3, 2) + Math.pow(4, 2));

// With static import:
import static java.lang.Math.sqrt;
import static java.lang.Math.pow;

double result = sqrt(pow(3, 2) + pow(4, 2));
```

**When to use static imports:**

- For frequently used constants like `HttpStatus.OK`, `Collections.emptyList()`, or `Assertions.assertEquals()` in test classes.
- When the meaning is clear without the class name qualifier.

**When NOT to use static imports:**

- When the method name is ambiguous or unclear without the class name. `sort(list)` is less readable than `Collections.sort(list)`.
- When importing too many static members from different classes, which makes the code harder to understand.

### How `static` Works Internally

When the JVM starts, it uses a **classloader** to load classes into memory. The loading process has three phases: loading, linking, and initialization.

**1. Loading**: The classloader reads the `.class` file and creates a `Class` object in the **method area** (also called metaspace in Java 8+). The method area is a region of the heap that stores class metadata, including static variables, static methods, and the runtime constant pool.

**2. Linking**: The JVM verifies the bytecode, allocates memory for static variables, and resolves symbolic references. Static variables are initialized to their default values during this phase.

**3. Initialization**: The JVM executes the static initializers and static blocks in the order they appear. This is when static variables get their actual values.

**Memory layout:**

```text
Heap
├── Metaspace (Method Area)
│   ├── User class metadata
│   │   ├── static int totalUsers = 3     <-- ONE copy, shared by all instances
│   │   ├── static final String APP = "MyApp"
│   │   └── static methods: getTotalUsers()
│   └── Order class metadata
│       └── ...
├── Object instances
│   ├── User@0x1A  { username: "saad" }   <-- Each has its own instance fields
│   ├── User@0x2B  { username: "rahim" }
│   └── User@0x3C  { username: "karim" }
```

Static variables live in the metaspace, not in the individual objects. This is why they are shared across all instances and all threads.

**Class loading triggers:**

A class is loaded and initialized when any of the following occurs for the first time:

1. An object of the class is created (`new User()`).
2. A static method of the class is called (`MathUtils.calculateTax(...)`).
3. A static field of the class is accessed (`User.totalUsers`).
4. A static field of the class is assigned (unless it is a `static final` constant).
5. The class is the startup class (contains `main()`).
6. A subclass is initialized (the parent class is initialized first).

> [!tip] Key Insight
> Static members are shared across all threads in a Java application. In a Spring Boot backend, the server handles thousands of concurrent requests, each on a different thread. If you store request-specific or user-specific data in a static variable, one thread will overwrite another thread's data, causing data corruption, security vulnerabilities, and impossible-to-reproduce bugs. The rule is simple: **never store mutable per-request state in static variables**. Use request-scoped beans, ThreadLocal, or pass data through method parameters instead.

---

## Syntax and Basic Examples

### Example 1: Static variables and methods

```java
public class DatabaseConnection {
    // Static constants: shared, immutable
    public static final String DEFAULT_HOST = "localhost";
    public static final int DEFAULT_PORT = 5432;
    public static final String DEFAULT_DATABASE = "orders_db";

    // Static variable: tracks total connections across all instances
    private static int activeConnections = 0;
    private static int totalConnectionsCreated = 0;

    // Instance variables: unique to each connection
    private String host;
    private int port;
    private String database;
    private boolean isConnected;

    public DatabaseConnection(String host, int port, String database) {
        this.host = host;
        this.port = port;
        this.database = database;
        this.isConnected = false;
    }

    // Instance method: operates on a specific connection
    public void connect() {
        if (!isConnected) {
            isConnected = true;
            activeConnections++;
            totalConnectionsCreated++;
            System.out.println("Connected to " + host + ":" + port + "/" + database);
            System.out.println("  Active connections: " + activeConnections);
        }
    }

    public void disconnect() {
        if (isConnected) {
            isConnected = false;
            activeConnections--;
            System.out.println("Disconnected from " + host);
            System.out.println("  Active connections: " + activeConnections);
        }
    }

    // Static method: reports global state, does not need an instance
    public static void printStats() {
        System.out.println("=== Connection Pool Stats ===");
        System.out.println("Active: " + activeConnections);
        System.out.println("Total created: " + totalConnectionsCreated);
    }

    // Static factory method: creates a connection with default settings
    public static DatabaseConnection createDefault() {
        return new DatabaseConnection(DEFAULT_HOST, DEFAULT_PORT, DEFAULT_DATABASE);
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        // Use static factory method (no 'new' keyword needed by the caller)
        DatabaseConnection conn1 = DatabaseConnection.createDefault();
        DatabaseConnection conn2 = new DatabaseConnection("prod-server", 5432, "orders_prod");

        conn1.connect();
        conn2.connect();

        // Static method called on the class, not on an instance
        DatabaseConnection.printStats();

        conn1.disconnect();
        DatabaseConnection.printStats();

        // Accessing static constants
        System.out.println("Default host: " + DatabaseConnection.DEFAULT_HOST);
    }
}
```

**Output:**

```text
Connected to localhost:5432/orders_db
  Active connections: 1
Connected to prod-server:5432/orders_prod
  Active connections: 2
=== Connection Pool Stats ===
Active: 2
Total created: 2
Disconnected from localhost
  Active connections: 1
=== Connection Pool Stats ===
Active: 1
Total created: 2
Default host: localhost
```

### Example 2: Static blocks and initialization order

```java
import java.util.HashMap;
import java.util.Map;

public class AppConfig {
    private static final String APP_NAME;
    private static final String VERSION;
    private static final Map<String, String> FEATURES;

    // Static block 1: runs first
    static {
        System.out.println("Static block 1: Loading app metadata...");
        APP_NAME = "OrderService";
        VERSION = "2.1.0";
    }

    // Static block 2: runs second
    static {
        System.out.println("Static block 2: Loading feature flags...");
        FEATURES = new HashMap<>();
        FEATURES.put("dark_mode", "true");
        FEATURES.put("new_checkout", "false");
        FEATURES.put("free_shipping", "true");
    }

    // Instance initializer (runs for each object, AFTER static blocks)
    {
        System.out.println("Instance initializer: Creating AppConfig instance");
    }

    public AppConfig() {
        System.out.println("Constructor: AppConfig ready");
    }

    public static String getAppName() { return APP_NAME; }
    public static String getVersion() { return VERSION; }
    public static boolean isFeatureEnabled(String feature) {
        return "true".equals(FEATURES.get(feature));
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("--- First access ---");
        System.out.println("App: " + AppConfig.getAppName() + " v" + AppConfig.getVersion());

        System.out.println("\n--- Creating instance ---");
        AppConfig config = new AppConfig();

        System.out.println("\n--- Second access (static blocks do NOT run again) ---");
        System.out.println("Dark mode: " + AppConfig.isFeatureEnabled("dark_mode"));
        System.out.println("New checkout: " + AppConfig.isFeatureEnabled("new_checkout"));
    }
}
```

**Output:**

```text
--- First access ---
Static block 1: Loading app metadata...
Static block 2: Loading feature flags...
App: OrderService v2.1.0

--- Creating instance ---
Instance initializer: Creating AppConfig instance
Constructor: AppConfig ready

--- Second access (static blocks do NOT run again) ---
Dark mode: true
New checkout: false
```

### Example 3: Static nested classes and the Builder pattern

```java
import java.util.HashMap;
import java.util.Map;

public class HttpRequest {
    private final String method;
    private final String url;
    private final Map<String, String> headers;
    private final String body;
    private final int timeoutMs;

    // Private constructor: only the Builder can create HttpRequest objects
    private HttpRequest(Builder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.headers = Map.copyOf(builder.headers);
        this.body = builder.body;
        this.timeoutMs = builder.timeoutMs;
    }

    // Getters
    public String getMethod() { return method; }
    public String getUrl() { return url; }
    public Map<String, String> getHeaders() { return headers; }
    public String getBody() { return body; }
    public int getTimeoutMs() { return timeoutMs; }

    @Override
    public String toString() {
        return method + " " + url + " | Headers: " + headers.size()
            + " | Timeout: " + timeoutMs + "ms"
            + (body != null ? " | Body: " + body.length() + " chars" : "");
    }

    // Static nested class: the Builder
    // It is static because it does not need access to HttpRequest's instance fields.
    // It only needs to set the fields during construction.
    public static class Builder {
        private String method = "GET";
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private String body;
        private int timeoutMs = 5000;

        public Builder(String url) {
            this.url = url;
        }

        public Builder method(String method) {
            this.method = method;
            return this;  // Return 'this' for method chaining
        }

        public Builder header(String key, String value) {
            this.headers.put(key, value);
            return this;
        }

        public Builder body(String body) {
            this.body = body;
            return this;
        }

        public Builder timeout(int timeoutMs) {
            this.timeoutMs = timeoutMs;
            return this;
        }

        public HttpRequest build() {
            if (url == null || url.isBlank()) {
                throw new IllegalStateException("URL is required");
            }
            return new HttpRequest(this);
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        // The Builder pattern uses a static nested class for clean object construction.
        // This is the same pattern used by OkHttp, Spring's RestTemplate, and
        // Java's HttpClient.
        HttpRequest request = new HttpRequest.Builder("https://api.example.com/orders")
            .method("POST")
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer token123")
            .body("{\"userId\": 1, \"items\": [1, 2, 3]}")
            .timeout(10000)
            .build();

        System.out.println(request);
    }
}
```

**Output:**

```text
POST https://api.example.com/orders | Headers: 2 | Timeout: 10000ms | Body: 38 chars
```

### Example 4: Static imports

```java
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;
import static java.lang.Math.pow;
import static java.util.Collections.unmodifiableList;
import static java.util.Collections.emptyList;

import java.util.List;

public class GeometryService {
    public double circleArea(double radius) {
        return PI * pow(radius, 2);  // Instead of Math.PI * Math.pow(...)
    }

    public double hypotenuse(double a, double b) {
        return sqrt(pow(a, 2) + pow(b, 2));  // Instead of Math.sqrt(Math.pow(...))
    }

    public List<String> getDefaultCategories() {
        return emptyList();  // Instead of Collections.emptyList()
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> The `static` keyword is used extensively in Spring Boot and all Java backend frameworks. Here are three realistic scenarios.

### Scenario 1: Constants and utility classes

Backend applications define constants in dedicated utility classes with static fields and static methods. These classes are never instantiated.

```java
package com.company.orderservice.constants;

import java.math.BigDecimal;

public final class OrderConstants {
    // Private constructor prevents instantiation.
    // This class only holds constants. Creating an object of it makes no sense.
    private OrderConstants() {
        throw new UnsupportedOperationException("Utility class cannot be instantiated");
    }

    // Order status values
    public static final String STATUS_PENDING = "PENDING";
    public static final String STATUS_CONFIRMED = "CONFIRMED";
    public static final String STATUS_SHIPPED = "SHIPPED";
    public static final String STATUS_DELIVERED = "DELIVERED";
    public static final String STATUS_CANCELLED = "CANCELLED";

    // Business rules
    public static final int MAX_ITEMS_PER_ORDER = 100;
    public static final BigDecimal MIN_ORDER_AMOUNT = new BigDecimal("50.00");
    public static final BigDecimal MAX_ORDER_AMOUNT = new BigDecimal("500000.00");
    public static final int ORDER_EXPIRY_MINUTES = 30;

    // API paths
    public static final String API_V1 = "/api/v1";
    public static final String ORDERS_PATH = API_V1 + "/orders";
    public static final String USERS_PATH = API_V1 + "/users";

    // Pagination
    public static final int DEFAULT_PAGE_SIZE = 20;
    public static final int MAX_PAGE_SIZE = 100;
}
```

```java
package com.company.orderservice.util;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.UUID;

// Utility class: all methods are static, no instances needed.
// This is the most common use of static in backend code.
public final class OrderUtils {

    private OrderUtils() {}

    // Static method: generates a unique order number
    public static String generateOrderNumber() {
        String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd"));
        String uuid = UUID.randomUUID().toString().substring(0, 6).toUpperCase();
        return "ORD-" + timestamp + "-" + uuid;
    }

    // Static method: validates an order amount
    public static boolean isValidAmount(BigDecimal amount) {
        return amount != null
            && amount.compareTo(OrderConstants.MIN_ORDER_AMOUNT) >= 0
            && amount.compareTo(OrderConstants.MAX_ORDER_AMOUNT) <= 0;
    }

    // Static method: rounds a monetary value to 2 decimal places
    public static BigDecimal roundCurrency(BigDecimal amount) {
        if (amount == null) return BigDecimal.ZERO;
        return amount.setScale(2, RoundingMode.HALF_UP);
    }

    // Static method: checks if an order has expired
    public static boolean isOrderExpired(LocalDateTime createdAt) {
        return createdAt.plusMinutes(OrderConstants.ORDER_EXPIRY_MINUTES)
            .isBefore(LocalDateTime.now());
    }
}
```

**What to notice:**

- Both classes are `final` with private constructors. This prevents instantiation and subclassing. They are pure containers for static members.
- All constants are `public static final`. They are accessible from anywhere via `OrderConstants.MAX_ITEMS_PER_ORDER`.
- Utility methods are `public static`. They are stateless and thread-safe because they do not modify any shared state. `generateOrderNumber()` creates a new String each time. `isValidAmount()` only reads its parameter. These methods can be called safely from any thread.

### Scenario 2: Static factory methods in Spring Boot

Spring Boot uses static factory methods extensively to create objects with complex initialization logic.

```java
package com.company.orderservice.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
import org.springframework.http.client.SimpleClientHttpRequestFactory;

@Configuration
public class RestClientConfig {

    // @Bean methods are essentially static factory methods managed by Spring.
    // Spring calls this method once at startup and caches the result.
    // Every component that needs a RestTemplate gets the same instance.
    @Bean
    public RestTemplate restTemplate() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(5000);
        factory.setReadTimeout(10000);
        return new RestTemplate(factory);
    }
}
```

```java
package com.company.orderservice.util;

import org.springframework.http.ResponseEntity;

// Spring's ResponseEntity uses static factory methods instead of constructors.
// This is a common pattern in the Java ecosystem.
public class ApiResponse {

    // Instead of: new ResponseEntity<>(body, HttpStatus.OK)
    // Spring provides: ResponseEntity.ok(body)
    // The static method is more readable and discoverable.

    public static <T> ResponseEntity<T> success(T data) {
        return ResponseEntity.ok(data);
    }

    public static <T> ResponseEntity<T> created(T data, String location) {
        return ResponseEntity
            .status(201)
            .header("Location", location)
            .body(data);
    }

    public static ResponseEntity<ErrorResponse> error(int status, String message) {
        return ResponseEntity
            .status(status)
            .body(new ErrorResponse(status, message));
    }

    public static ResponseEntity<Void> noContent() {
        return ResponseEntity.noContent().build();
    }
}
```

```java
// Usage in a controller:
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        Order order = orderService.createOrder(request);
        // Static factory method: clean and readable
        return ApiResponse.created(
            OrderResponse.fromEntity(order),
            "/api/v1/orders/" + order.getId()
        );
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteOrder(@PathVariable Long id) {
        orderService.deleteOrder(id);
        return ApiResponse.noContent();
    }
}
```

**What to notice:**

- Static factory methods (`ApiResponse.ok()`, `ApiResponse.created()`) are more readable than constructors. `ApiResponse.created(data, location)` clearly communicates intent, while `new ApiResponse(201, data, location)` requires you to remember what `201` means.
- Static factory methods can return subtypes, cache instances, and have meaningful names. Constructors cannot do any of these things. This is why many Java libraries (Guava, Spring, Jackson) prefer static factories over public constructors.

### Scenario 3: The Singleton pattern (and why Spring makes it mostly unnecessary)

The Singleton pattern ensures that only one instance of a class exists. It uses a static variable to hold the single instance and a static method to provide access to it.

```java
package com.company.orderservice.util;

// Traditional Singleton pattern (before Spring)
public class DatabaseConnectionPool {
    // Static variable: holds the single instance
    private static volatile DatabaseConnectionPool instance;

    private final int maxConnections;
    private final String connectionString;

    // Private constructor: prevents external instantiation
    private DatabaseConnectionPool() {
        this.maxConnections = 20;
        this.connectionString = "jdbc:postgresql://localhost:5432/orders";
        System.out.println("Connection pool initialized");
    }

    // Static factory method: provides access to the single instance
    // Double-checked locking for thread safety
    public static DatabaseConnectionPool getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnectionPool.class) {
                if (instance == null) {
                    instance = new DatabaseConnectionPool();
                }
            }
        }
        return instance;
    }

    public int getMaxConnections() { return maxConnections; }
}
```

```java
// In Spring Boot, you almost never write Singletons manually.
// Spring manages object lifecycle and creates singletons by default.
@Service
public class OrderService {
    // Spring creates exactly ONE instance of OrderService at startup.
    // Every controller and component that depends on OrderService
    // receives the same instance. This is the Singleton pattern
    // managed by the framework, not by your code.

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    // All methods are instance methods, but since there is only one
    // instance, the behavior is equivalent to a Singleton.
    public Order createOrder(CreateOrderRequest request) {
        // ...
    }
}
```

**What to notice:**

- The traditional Singleton pattern uses a `private static` variable and a `public static` factory method. The `volatile` keyword ensures thread-safe publication of the instance across CPU caches.
- In Spring Boot, all `@Service`, `@Component`, `@Repository`, and `@Controller` beans are singletons by default. Spring creates one instance at startup and injects it everywhere. You do not need to write Singleton boilerplate. This is one of the biggest advantages of using a dependency injection framework.
- The manual Singleton pattern is still useful for utility classes that are not managed by Spring (e.g., a custom logger, a configuration loader that runs before Spring starts).

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Storing per-request state in static variables (CRITICAL)

**Wrong:**

```java
@Service
public class UserService {
    // DISASTER: This static variable is shared by ALL threads.
    // In a server handling 1000 concurrent requests, every request
    // overwrites this variable. User A might see User B's data.
    private static User currentUser;

    public void processRequest(User user) {
        currentUser = user;  // Thread A sets "Alice"
        // ... Thread B sets "Bob", overwriting Alice ...
        sendEmail(currentUser);  // Thread A sends email to Bob!
    }
}
```

**Right:**

```java
@Service
public class UserService {
    // Instance method with parameter: each thread has its own stack,
    // so the 'user' parameter is isolated per request.
    public void processRequest(User user) {
        sendEmail(user);  // Safe: 'user' is a local parameter, not shared
    }

    // Or use Spring's request-scoped beans for per-request state
    // Or use SecurityContextHolder for authentication context
}
```

**Why it is wrong:** This is the most dangerous mistake a backend developer can make with `static`. A Spring Boot server handles thousands of requests concurrently on different threads. All threads share the same static variables. Storing per-request data in a static variable causes data leakage between users, security vulnerabilities (one user seeing another user's data), and race conditions that are nearly impossible to reproduce in testing. **Never store mutable per-request state in static variables.**

### Mistake 2: Accessing instance members from a static context

**Wrong:**

```java
public class OrderService {
    private OrderRepository repository;  // Instance field

    public static List<Order> getAllOrders() {
        return repository.findAll();  // COMPILATION ERROR!
        // Cannot access instance field 'repository' from a static method.
    }
}
```

**Right:**

```java
public class OrderService {
    private OrderRepository repository;

    // Option 1: Make the method an instance method (preferred in Spring)
    public List<Order> getAllOrders() {
        return repository.findAll();
    }

    // Option 2: Pass the dependency as a parameter
    public static List<Order> getAllOrders(OrderRepository repository) {
        return repository.findAll();
    }
}
```

**Why it is wrong:** Static methods belong to the class, not to any object. They have no `this` reference and no access to instance fields. If a method needs to access instance state, it should be an instance method. In Spring Boot, almost all service methods are instance methods because they depend on injected repositories and other services.

### Mistake 3: Calling static methods on object references

**Wrong (technically compiles, but misleading):**

```java
MathUtils utils = new MathUtils();
double result = utils.calculateTax(amount, rate);  // Compiles but misleading
// This looks like an instance method call, but calculateTax is static.
// The 'utils' object is completely irrelevant. The method is called on the class.
```

**Right:**

```java
double result = MathUtils.calculateTax(amount, rate);  // Clear: static method on the class
```

**Why it is wrong:** Calling a static method on an object reference compiles but is misleading. It suggests that the method operates on the specific object, which it does not. The JVM ignores the object reference entirely and calls the method on the class. If the reference is `null`, the static method still executes without throwing a NullPointerException (because the object is never accessed). This behavior is confusing and should be avoided.

### Mistake 4: Using static methods when instance methods are more testable

**Wrong:**

```java
public class OrderService {
    public Order createOrder(CreateOrderRequest request) {
        String orderNumber = OrderUtils.generateOrderNumber();  // Static call
        BigDecimal tax = TaxCalculator.calculate(request.getTotal());  // Static call
        // ...
    }
}

// Testing this is hard because you cannot mock static methods easily.
// The static calls are hardcoded and cannot be replaced with test doubles.
```

**Right:**

```java
public class OrderService {
    private final OrderNumberGenerator orderNumberGenerator;  // Interface
    private final TaxCalculator taxCalculator;  // Interface

    public OrderService(OrderNumberGenerator orderNumberGenerator, TaxCalculator taxCalculator) {
        this.orderNumberGenerator = orderNumberGenerator;
        this.taxCalculator = taxCalculator;
    }

    public Order createOrder(CreateOrderRequest request) {
        String orderNumber = orderNumberGenerator.generate();  // Instance call
        BigDecimal tax = taxCalculator.calculate(request.getTotal());  // Instance call
        // ...
    }
}

// Testing is easy: inject mock implementations of the interfaces.
```

**Why it is wrong:** Static methods are hard to mock in unit tests. Before Java 16 (when Mockito added `mockStatic()`), mocking static methods was impossible without special libraries. Even with `mockStatic()`, it is slower and more fragile than mocking instance methods. By wrapping static utility calls behind interfaces and injecting them, you make your code testable and flexible. This is the Dependency Inversion Principle in action.

---

## Key Takeaways

> [!tip] Remember these points
> 1. The `static` keyword makes a member belong to the **class** rather than to individual objects. Static variables are shared across all instances. Static methods can be called without creating an object. Static blocks execute once when the class is loaded.
> 2. Use `static` for **constants** (`public static final`), **utility methods** (stateless helper functions), **factory methods** (creating objects with complex initialization), and **shared counters/caches** (with thread-safety considerations).
> 3. **Never store mutable per-request or per-user state in static variables** in a backend server. Static variables are shared across all threads, which causes data corruption and security vulnerabilities in concurrent environments.
> 4. Static methods cannot access instance members, cannot use `this` or `super`, and cannot be overridden (only hidden). If a method needs instance state, make it an instance method.
> 5. Static nested classes do not hold a reference to the enclosing class's instance. Use them for builders, logical grouping, and helper classes that do not need access to outer instance fields. They are more memory-efficient than inner classes.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Static Counter and Constants (Easy)

Create a `Product` class with:

- A `private static int productCount` that increments every time a new `Product` is created.
- A `public static final double TAX_RATE = 0.15`.
- Instance fields: `name` (String), `price` (double).
- A constructor that validates the price and increments `productCount`.
- A static method `getProductCount()` that returns the total number of products created.
- A static method `calculateTax(double price)` that returns `price * TAX_RATE`.
- An instance method `getPriceWithTax()` that uses the static `calculateTax()` method.

In `main()`, create 5 products, print the count, and calculate the tax-inclusive price for each.

> **Hint:** The static counter is incremented in the constructor. The static `calculateTax()` method can be called from both static and instance contexts.

### Exercise 2: Static Block Configuration Loader (Medium)

Create a `FeatureFlags` class that uses a static block to initialize a `Map<String, Boolean>` of feature flags. The static block should:

1. Print "Loading feature flags..." to simulate loading from a file.
2. Populate the map with at least 5 feature flags (e.g., "dark_mode", "new_checkout", "free_shipping", "sms_notifications", "multi_currency").
3. Provide a static method `isEnabled(String flag)` that returns the flag's value or `false` if the flag does not exist.
4. Provide a static method `getAllFlags()` that returns an unmodifiable copy of the map.

In `main()`, access the flags multiple times and verify that the static block runs only once.

> **Hint:** Use `Collections.unmodifiableMap()` in the `getAllFlags()` method to prevent external modification of the internal map.

### Exercise 3: Builder Pattern with Static Nested Class (Medium)

Create an `EmailMessage` class using the Builder pattern with a static nested `Builder` class. The `EmailMessage` should have fields: `from` (String), `to` (String), `subject` (String), `body` (String), `cc` (List of Strings), `isHtml` (boolean). The Builder should:

- Require `from`, `to`, and `subject` (throw `IllegalStateException` in `build()` if any are missing).
- Default `isHtml` to `false` and `cc` to an empty list.
- Support method chaining for all fields.

In `main()`, build several email messages with different combinations of fields and print them.

> **Hint:** The `EmailMessage` constructor should be private and accept the `Builder` as a parameter. The `Builder` copies all its fields into the `EmailMessage` during `build()`.

### Exercise 4: Thread Safety Demonstration (Hard, Optional)

Write a program that demonstrates why static variables are dangerous in multi-threaded environments:

1. Create a class `RequestCounter` with a `private static int count = 0` and a static method `increment()` that reads the count, adds 1, and writes it back (not using `++`, but explicitly: `int temp = count; temp = temp + 1; count = temp;`).
2. In `main()`, create 10 threads, each calling `increment()` 10,000 times.
3. After all threads finish, print the final count. The expected value is 100,000, but the actual value will likely be less due to race conditions.
4. Fix the race condition using `synchronized` on the `increment()` method and verify that the count is now exactly 100,000.
5. Fix it again using `AtomicInteger` instead of `int` and compare the approaches.

> **Hint:** Use `Thread.start()` and `Thread.join()` to run and wait for all threads. The explicit read-modify-write in step 1 makes the race condition more likely to manifest.

<details>
<summary>Solution for Exercise 1</summary>

```java
public class Product {
    private static int productCount = 0;
    public static final double TAX_RATE = 0.15;

    private String name;
    private double price;

    public Product(String name, double price) {
        if (price < 0) throw new IllegalArgumentException("Price cannot be negative");
        this.name = name;
        this.price = price;
        productCount++;
    }

    public static int getProductCount() { return productCount; }

    public static double calculateTax(double price) {
        return price * TAX_RATE;
    }

    public double getPriceWithTax() {
        return price + calculateTax(price);
    }

    @Override
    public String toString() {
        return String.format("%s: %.2f BDT (tax: %.2f, total: %.2f)",
            name, price, calculateTax(price), getPriceWithTax());
    }

    public static void main(String[] args) {
        Product[] products = {
            new Product("Laptop", 85000),
            new Product("Mouse", 1500),
            new Product("Keyboard", 3200),
            new Product("Monitor", 25000),
            new Product("Webcam", 4500)
        };

        System.out.println("Total products created: " + Product.getProductCount());
        for (Product p : products) {
            System.out.println(p);
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 4</summary>

```java
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadSafetyDemo {

    // UNSAFE: race condition
    private static int unsafeCount = 0;
    public static void unsafeIncrement() {
        int temp = unsafeCount;
        temp = temp + 1;
        unsafeCount = temp;
    }

    // SAFE: synchronized
    private static int syncCount = 0;
    public static synchronized void syncIncrement() {
        int temp = syncCount;
        temp = temp + 1;
        syncCount = temp;
    }

    // SAFE: atomic
    private static AtomicInteger atomicCount = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        int numThreads = 10;
        int incrementsPerThread = 10000;
        Thread[] threads = new Thread[numThreads];

        for (int i = 0; i < numThreads; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < incrementsPerThread; j++) {
                    unsafeIncrement();
                    syncIncrement();
                    atomicCount.incrementAndGet();
                }
            });
            threads[i].start();
        }

        for (Thread t : threads) {
            t.join();
        }

        int expected = numThreads * incrementsPerThread;
        System.out.println("Expected:    " + expected);
        System.out.println("Unsafe:      " + unsafeCount + " (likely less than expected!)");
        System.out.println("Synchronized:" + syncCount);
        System.out.println("Atomic:      " + atomicCount.get());
    }
}
```

</details>

---

## Related Notes

- [[Java - Abstraction - Abstract Classes and Interfaces]]
- [[Java - Final Keyword]] (next note)
- [[Java - Exception Handling - Try Catch Finally Throw Throws]]

---

## Resources

- [Oracle Java Tutorials: Understanding Class Members](https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html) - Official documentation on static variables and methods.
- [Oracle Java Tutorials: Initializing Fields](https://docs.oracle.com/javase/tutorial/java/javaOO/initial.html) - Official guide on static and instance initializers.
- [Baeldung: Static Keyword in Java](https://www.baeldung.com/java-static) - Comprehensive guide covering all uses of static with examples.
- [Baeldung: Static Nested Classes in Java](https://www.baeldung.com/java-static-nested-classes) - Detailed comparison of static nested classes vs inner classes.
- [Effective Java by Joshua Bloch - Item 1: Consider Static Factory Methods Instead of Constructors](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive argument for static factory methods.
- [Java Concurrency in Practice by Brian Goetz - Chapter 2: Thread Safety](https://www.oreilly.com/library/view/java-concurrency-in/0321349601/) - Essential reading on why static mutable state is dangerous in concurrent systems.
