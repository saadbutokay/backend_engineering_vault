**Phase:** Level 1 - Java Fundamentals
**Date Studied:**

---
## What Problem Does This Solve?

```text
Programs fail. Always.

  The database goes down.
  The user enters "abc" where a number is expected.
  The file doesn't exist.
  The network times out.
  A service returns unexpected null.
  Division by zero in a calculation.
  Memory runs out.

Without exception handling:
  Your program CRASHES. Completely. No warning.

  int value = Integer.parseInt(userInput); // "abc" → CRASH
  String name = user.getName();            // user is null → CRASH
  File file = new File("data.txt");        // doesn't exist → CRASH

  The entire application stops.
  Users see a cryptic error.
  Data might be corrupted.
  Other users are affected.
  Production is down.

With exception handling:
  Errors are CAUGHT and handled GRACEFULLY.
  The application keeps running.
  Users see a meaningful message.
  Errors are logged for engineers to investigate.
  The system tries to recover where possible.

In backend engineering:
  → APIs must NEVER crash — return proper error responses instead
  → Databases failing must be caught and retried
  → Invalid input must be rejected with clear error messages
  → All failures must be logged for investigation
  → Some errors need alerts sent to the team

Exception handling is not optional in production code.
Every Spring Boot endpoint, service, and repository
has exception handling. It's foundational.
```

---

## 1. The Exception Hierarchy

```text
ALL throwable things in Java extend Throwable.

                        Throwable
                           │
              ┌────────────┴────────────┐
              │                         │
            Error                   Exception
              │                         │
    (JVM-level problems)        ┌───────┴────────┐
    StackOverflowError          │                │
    OutOfMemoryError     RuntimeException    (Checked Exceptions)
    VirtualMachineError         │            IOException
                                │            SQLException
           (Unchecked/Runtime)  │            FileNotFoundException
           NullPointerException │            ClassNotFoundException
           ArrayIndexOutOf...   │
           ClassCastException   │
           IllegalArgument...   │
           NumberFormat...      │
           IllegalState...      │

TWO KEY CATEGORIES:

1. CHECKED EXCEPTIONS (extend Exception but NOT RuntimeException)
   → Java FORCES you to handle them (compile error if you don't)
   → Represent recoverable situations: file missing, network error
   → You must: declare with 'throws' OR wrap in try-catch
   → Examples: IOException, SQLException, FileNotFoundException

2. UNCHECKED EXCEPTIONS (extend RuntimeException or Error)
   → Java does NOT force you to handle them
   → Represent programming errors or unrecoverable situations
   → Examples: NullPointerException, IllegalArgumentException
   → YOU decide whether to catch them

3. ERRORS (extend Error)
   → JVM-level, usually unrecoverable
   → NEVER catch Error (except in very specific framework code)
   → StackOverflowError, OutOfMemoryError

THE PROFESSIONAL RULE:
  Use UNCHECKED exceptions for most things in modern Java.
  (Spring Boot, modern libraries all use unchecked)
  Use CHECKED for things the CALLER truly needs to handle
  (IO, network) where the caller can meaningfully recover.
```

---

## 2. try-catch-finally — Complete Guide

```java
public class TryCatchGuide {
    
    // ─────────────────────────────────────────
    // BASIC try-catch
    // ─────────────────────────────────────────
    
    public static void basicTryCatch() {
        System.out.println("=== Basic try-catch ===");
        
        String input = "abc"; // not a number
        
        try {
            // Code that might throw:
            int number = Integer.parseInt(input);
            System.out.println("Parsed: " + number); // never reaches here
        } catch (NumberFormatException e) {
            // Runs ONLY if NumberFormatException is thrown:
            System.out.println("❌ Cannot parse '" + input + "' as integer");
            System.out.println("   Error: " + e.getMessage());
        }
        
        System.out.println("Program continues after catch block.");
        // Without try-catch, this line would never execute.
    }
    
    // ─────────────────────────────────────────
    // MULTIPLE catch BLOCKS
    // ─────────────────────────────────────────
    
    public static void multipleCatch(String input, int[] array, int index) {
        System.out.println("\n=== Multiple catch blocks ===");
        
        try {
            int value    = Integer.parseInt(input);      // might throw NumberFormatException
            int result   = array[index];                 // might throw ArrayIndexOutOfBoundsException
            int division = value / result;               // might throw ArithmeticException
            System.out.println("Result: " + division);
        } catch (NumberFormatException e) {
            // Caught FIRST if parsing fails:
            System.out.println("❌ Invalid number: " + input);
        } catch (ArrayIndexOutOfBoundsException e) {
            // Caught if index is out of bounds:
            System.out.println("❌ Index " + index + " out of bounds");
        } catch (ArithmeticException e) {
            // Caught if division by zero:
            System.out.println("❌ Cannot divide by zero");
        } catch (Exception e) {
            // Catch-all: catches anything not caught above
            // MUST be last — catches parent type
            System.out.println("❌ Unexpected error: " + e.getMessage());
        }
    }
    
    // ─────────────────────────────────────────
    // MULTI-CATCH (Java 7+) — catch multiple types in one block
    // ─────────────────────────────────────────
    
    public static void multiCatch(String input) {
        System.out.println("\n=== Multi-catch ===");
        
        try {
            int value = Integer.parseInt(input);
            System.out.println(100 / value);
        } catch (NumberFormatException | ArithmeticException e) {
            // Both exceptions handled the same way — no duplication:
            System.out.println("❌ Input error: " + e.getMessage());
        }
    }
    
    // ─────────────────────────────────────────
    // finally — ALWAYS runs (cleanup code)
    // ─────────────────────────────────────────
    
    public static void finallyDemo() {
        System.out.println("\n=== finally block ===");
        
        // finally runs:
        // → After try block completes normally
        // → After catch block completes
        // → Even if exception is thrown and not caught
        // → Even if there's a return statement in try or catch
        
        // Use for: closing resources, releasing locks, cleanup
        
        java.io.BufferedReader reader = null;
        
        try {
            System.out.println("Opening resource...");
            reader = new java.io.BufferedReader(
                new java.io.StringReader("test data"));
            
            String line = reader.readLine();
            System.out.println("Read: " + line);
            
            // Simulate an error:
            if (line != null) throw new RuntimeException("Simulated error");
            
        } catch (java.io.IOException e) {
            System.out.println("IO Error: " + e.getMessage());
        } catch (RuntimeException e) {
            System.out.println("Runtime Error: " + e.getMessage());
        } finally {
            // This ALWAYS runs — good for cleanup:
            System.out.println("Closing resource (finally)...");
            if (reader != null) {
                try { reader.close(); }
                catch (java.io.IOException e) { /* ignore close errors */ }
            }
            System.out.println("Resource closed.");
        }
        
        System.out.println("After try-catch-finally.");
    }
    
    // finally with return — tricky!
    public static int finallyWithReturn() {
        try {
            System.out.println("In try");
            return 1;          // try to return 1
        } finally {
            System.out.println("In finally"); // this runs BEFORE the return!
            return 2;          // overrides the return from try!
        }
    }
    // This returns 2! finally's return overrides try's return.
    // AVOID: never return from finally — confusing and dangerous.
    
    // ─────────────────────────────────────────
    // ORDER OF EXECUTION
    // ─────────────────────────────────────────
    
    public static void executionOrder() {
        System.out.println("\n=== Execution order ===");
        
        try {
            System.out.println("1. try starts");
            System.out.println("2. before throw");
            throw new RuntimeException("test");
            // System.out.println("UNREACHABLE"); // compiler error
        } catch (RuntimeException e) {
            System.out.println("3. catch: " + e.getMessage());
        } finally {
            System.out.println("4. finally");
        }
        System.out.println("5. after try-catch-finally");
        
        // Output:
        // 1. try starts
        // 2. before throw
        // 3. catch: test
        // 4. finally
        // 5. after try-catch-finally
    }
    
    public static void main(String[] args) {
        basicTryCatch();
        multipleCatch("abc", new int[]{10, 20, 30}, 5);
        multipleCatch("5",   new int[]{10, 20, 30}, 1);
        multipleCatch("5",   new int[]{10, 0, 30},  1);
        multiCatch("abc");
        multiCatch("0");
        finallyDemo();
        
        System.out.println("\nfinallyWithReturn: " + finallyWithReturn()); // 2
        executionOrder();
    }
}
```

---

## 3. try-with-resources — The Modern Way

```java
public class TryWithResources {
    
    // ─────────────────────────────────────────
    // THE PROBLEM with manual close:
    // ─────────────────────────────────────────
    //
    // Old way — error-prone:
    //   Connection conn = null;
    //   try {
    //     conn = getConnection();
    //     doWork(conn);
    //   } catch (SQLException e) {
    //     handle(e);
    //   } finally {
    //     if (conn != null) {
    //       try { conn.close(); }
    //       catch (SQLException e) { /* ignore */ }
    //     }
    //   }
    //
    // Problems:
    //   → Verbose (many lines for simple task)
    //   → Easy to forget null check
    //   → Exception in finally can HIDE original exception
    //   → If close() throws, you lose the original error!
    // ─────────────────────────────────────────
    
    // RESOURCE = any class implementing AutoCloseable
    // AutoCloseable has: void close() throws Exception
    // Closeable (from IO) extends AutoCloseable
    
    // Custom AutoCloseable:
    static class DatabaseConnection implements AutoCloseable {
        private final String url;
        private boolean open;
        
        public DatabaseConnection(String url) {
            this.url  = url;
            this.open = true;
            System.out.println("📂 Connection opened: " + url);
        }
        
        public String query(String sql) {
            if (!open) throw new IllegalStateException("Connection is closed");
            System.out.println("🔍 Querying: " + sql);
            return "result data";
        }
        
        @Override
        public void close() {
            // Called AUTOMATICALLY when try-with-resources block ends
            this.open = false;
            System.out.println("🔒 Connection closed: " + url);
        }
    }
    
    static class FileProcessor implements AutoCloseable {
        private final String filename;
        
        public FileProcessor(String filename) {
            this.filename = filename;
            System.out.println("📄 File opened: " + filename);
        }
        
        public String process() {
            return "processed content of " + filename;
        }
        
        @Override
        public void close() {
            System.out.println("📄 File closed: " + filename);
        }
    }
    
    // ─────────────────────────────────────────
    // BASIC try-with-resources
    // ─────────────────────────────────────────
    
    public static void basicExample() {
        System.out.println("=== Basic try-with-resources ===");
        
        // Resources declared in try() are auto-closed when block exits:
        try (DatabaseConnection conn = new DatabaseConnection("localhost:5432/mydb")) {
            String result = conn.query("SELECT * FROM users");
            System.out.println("Got: " + result);
            // conn.close() called AUTOMATICALLY here — even if exception thrown
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
        // After this: conn is guaranteed to be closed
        
        System.out.println("After try-with-resources block.");
    }
    
    // ─────────────────────────────────────────
    // MULTIPLE RESOURCES
    // ─────────────────────────────────────────
    
    public static void multipleResources() {
        System.out.println("\n=== Multiple resources ===");
        
        // Closed in REVERSE order of declaration:
        // conn opened first, file opened second
        // file closed first, conn closed second
        try (DatabaseConnection conn = new DatabaseConnection("db:5432/app");
             FileProcessor file = new FileProcessor("output.csv")) {
            
            String data = conn.query("SELECT * FROM products");
            String processed = file.process();
            System.out.println("Data: " + data);
            System.out.println("File: " + processed);
            
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
    
    // ─────────────────────────────────────────
    // SUPPRESSED EXCEPTIONS
    // ─────────────────────────────────────────
    
    static class FailingClose implements AutoCloseable {
        @Override
        public void close() throws Exception {
            throw new Exception("Error during close!");
        }
    }
    
    public static void suppressedExceptions() {
        System.out.println("\n=== Suppressed exceptions ===");
        
        try (FailingClose resource = new FailingClose()) {
            throw new RuntimeException("Primary exception");
            // When exiting try: close() is called → throws too
            // PRIMARY exception is the RuntimeException
            // close() exception is SUPPRESSED (attached to primary)
        } catch (Exception e) {
            System.out.println("Primary: " + e.getMessage());
            for (Throwable suppressed : e.getSuppressed()) {
                System.out.println("Suppressed: " + suppressed.getMessage());
            }
        }
        // Primary: Primary exception
        // Suppressed: Error during close!
        // Old way: close() exception would HIDE the primary!
    }
    
    // ─────────────────────────────────────────
    // REAL-WORLD: Reading a file
    // ─────────────────────────────────────────
    
    public static void readFile(String path) {
        System.out.println("\n=== Reading file: " + path + " ===");
        
        try (java.io.BufferedReader reader = new java.io.BufferedReader(
                new java.io.FileReader(path))) {
            
            String line;
            int lineNumber = 0;
            while ((line = reader.readLine()) != null) {
                lineNumber++;
                System.out.printf("Line %d: %s%n", lineNumber, line);
            }
            
        } catch (java.io.FileNotFoundException e) {
            System.out.println("File not found: " + path);
        } catch (java.io.IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }
        // reader is GUARANTEED to be closed even if exception thrown
    }
    
    public static void main(String[] args) {
        basicExample();
        multipleResources();
        suppressedExceptions();
        readFile("nonexistent.txt");
    }
}
```

---

## 4. throws — Declaring Exceptions

```java
public class ThrowsDeclaration {
    
    // 'throws' in a method signature = "I might throw this, caller must handle it"
    // REQUIRED for checked exceptions (compile error if missing)
    // OPTIONAL for unchecked exceptions (but sometimes good documentation)
    
    // ─────────────────────────────────────────
    // CHECKED EXCEPTION: must declare with throws
    // ─────────────────────────────────────────
    
    // This method reads a file — can throw checked IOException:
    public static String readFileContent(String path) throws java.io.IOException {
        // If this throws IOException: caller MUST handle it
        java.io.BufferedReader reader = new java.io.BufferedReader(
            new java.io.FileReader(path)); // FileNotFoundException (checked!)
        
        StringBuilder content = new StringBuilder();
        String line;
        while ((line = reader.readLine()) != null) {
            content.append(line).append("\n");
        }
        reader.close();
        return content.toString();
    }
    
    // Caller must handle the checked exception:
    public static void callerMustHandle() {
        // Option 1: Catch it here
        try {
            String content = readFileContent("data.txt");
            System.out.println(content);
        } catch (java.io.IOException e) {
            System.out.println("Cannot read file: " + e.getMessage());
        }
        
        // Option 2: Propagate it (declare in THIS method too)
        // → See propagateException() below
    }
    
    // Propagating: method also declares throws
    public static void propagateException() throws java.io.IOException {
        String content = readFileContent("data.txt"); // propagated up
        System.out.println(content);
    }
    
    // ─────────────────────────────────────────
    // MULTIPLE throws
    // ─────────────────────────────────────────
    
    public static void multipleThrows() throws java.io.IOException,
                                                java.sql.SQLException,
                                                InterruptedException {
        // This method might throw any of these checked exceptions
    }
    
    // ─────────────────────────────────────────
    // UNCHECKED: throws is optional documentation
    // ─────────────────────────────────────────
    
    // Not required, but good documentation:
    public static int divide(int a, int b) throws ArithmeticException {
        if (b == 0) throw new ArithmeticException("Division by zero");
        return a / b;
    }
    
    // ─────────────────────────────────────────
    // THE MODERN APPROACH: Wrap checked in unchecked
    // (What Spring Boot does everywhere)
    // ─────────────────────────────────────────
    
    // Instead of propagating checked exceptions everywhere:
    // Wrap them in a RuntimeException
    
    public static String readFileSafely(String path) {
        try {
            return readFileContent(path);
        } catch (java.io.IOException e) {
            // Wrap checked in unchecked — cleaner for callers:
            throw new RuntimeException("Failed to read file: " + path, e);
            // The original exception is preserved as the CAUSE
        }
    }
    
    // Now callers don't have to handle checked exceptions:
    public static void cleanCaller() {
        String content = readFileSafely("data.txt"); // no try-catch needed!
        // If it fails, RuntimeException propagates up
        System.out.println(content);
    }
    
    // ─────────────────────────────────────────
    // EXCEPTION TRANSLATION (important pattern)
    // ─────────────────────────────────────────
    
    // Low-level: SQL exception
    public static Object queryDatabase(String sql) throws java.sql.SQLException {
        throw new java.sql.SQLException("Table 'users' not found");
    }
    
    // High-level: translates to domain exception
    public static Object findUser(long id) {
        try {
            return queryDatabase("SELECT * FROM users WHERE id = " + id);
        } catch (java.sql.SQLException e) {
            // Don't expose low-level details — translate to domain exception:
            throw new RuntimeException("User lookup failed for ID: " + id, e);
        }
    }
    // Callers see "UserNotFoundException" not "SQLException"
    // Implementation details are hidden — good abstraction
    
    public static void main(String[] args) {
        callerMustHandle();
        
        try {
            propagateException();
        } catch (java.io.IOException e) {
            System.out.println("Handled at top: " + e.getMessage());
        }
        
        System.out.println(divide(10, 2));  // 5
        
        try {
            divide(10, 0); // ArithmeticException — no need to declare
        } catch (ArithmeticException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        
        // readFileSafely — cleaner for callers:
        try {
            readFileSafely("missing.txt");
        } catch (RuntimeException e) {
            System.out.println("Wrapped: " + e.getMessage());
            System.out.println("Cause: " + e.getCause().getMessage());
        }
    }
}
```

---

## 5. throw — Manually Throwing Exceptions

```java
public class ThrowKeyword {
    
    // 'throw' manually creates and throws an exception
    // Used to signal that something went wrong in YOUR code
    // Enforces preconditions and business rules
    
    // ─────────────────────────────────────────
    // BASIC throw
    // ─────────────────────────────────────────
    
    public static int divide(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("Cannot divide " + a + " by zero");
            // throw exits the method immediately
            // NOTHING after throw is reached
        }
        return a / b;
    }
    
    // ─────────────────────────────────────────
    // VALIDATING INPUTS (very common pattern)
    // ─────────────────────────────────────────
    
    public static void createUser(String name, String email, int age) {
        // Guard clauses — validate before doing real work:
        
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (name.length() < 2 || name.length() > 100) {
            throw new IllegalArgumentException(
                "Name must be 2-100 characters, got: " + name.length());
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email: " + email);
        }
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException(
                "Age must be 0-150, got: " + age);
        }
        
        // Happy path — all inputs are valid:
        System.out.println("Creating user: " + name + " (" + email + "), age " + age);
    }
    
    // ─────────────────────────────────────────
    // BUSINESS RULE VIOLATIONS
    // ─────────────────────────────────────────
    
    static class BankAccount {
        private double balance;
        private boolean isFrozen;
        private static final double MIN_BALANCE = 500.0;
        
        public BankAccount(double initialBalance) {
            if (initialBalance < MIN_BALANCE) {
                throw new IllegalArgumentException(
                    "Initial balance must be at least ৳" + MIN_BALANCE);
            }
            this.balance  = initialBalance;
            this.isFrozen = false;
        }
        
        public void withdraw(double amount) {
            if (isFrozen) {
                throw new IllegalStateException("Account is frozen, cannot withdraw");
            }
            if (amount <= 0) {
                throw new IllegalArgumentException("Amount must be positive: " + amount);
            }
            if (amount > balance - MIN_BALANCE) {
                throw new IllegalStateException(
                    String.format("Insufficient balance. Available: ৳%.2f (min balance: ৳%.2f)",
                                  balance - MIN_BALANCE, MIN_BALANCE));
            }
            balance -= amount;
            System.out.printf("Withdrawn ৳%.2f. Balance: ৳%.2f%n", amount, balance);
        }
        
        public void freeze() {
            if (isFrozen) {
                throw new IllegalStateException("Account is already frozen");
            }
            isFrozen = true;
            System.out.println("Account frozen.");
        }
    }
    
    // ─────────────────────────────────────────
    // NULL CHECKS (prevent NPE with clear message)
    // ─────────────────────────────────────────
    
    public static void processOrder(Object order, Object customer, Object payment) {
        // Objects.requireNonNull — throws NullPointerException with message:
        java.util.Objects.requireNonNull(order,    "Order cannot be null");
        java.util.Objects.requireNonNull(customer, "Customer cannot be null");
        java.util.Objects.requireNonNull(payment,  "Payment cannot be null");
        
        // After these checks: order, customer, payment are guaranteed non-null
        System.out.println("Processing order...");
    }
    
    // ─────────────────────────────────────────
    // RE-THROWING (adding context then rethrowing)
    // ─────────────────────────────────────────
    
    public static void processPayment(double amount) {
        try {
            // Simulate calling external payment API:
            if (amount > 100_000) {
                throw new RuntimeException("Payment gateway rejected large amount");
            }
            System.out.println("Payment processed: ৳" + amount);
        } catch (RuntimeException e) {
            // Add context and rethrow:
            throw new RuntimeException(
                "Payment failed for amount ৳" + amount + ": " + e.getMessage(), e);
        }
    }
    
    // ─────────────────────────────────────────
    // throw vs throws — NEVER CONFUSE THESE
    // ─────────────────────────────────────────
    //
    // throw  → ACTION: actually throws an exception right now
    //          Used inside method body
    //          throw new SomeException("message");
    //
    // throws → DECLARATION: warns callers this might be thrown
    //          Used in method signature
    //          public void method() throws SomeException { }
    
    public static void main(String[] args) {
        
        System.out.println("=== divide ===");
        System.out.println(divide(10, 2)); // 5
        try { divide(10, 0); } catch (ArithmeticException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        
        System.out.println("\n=== createUser ===");
        createUser("Rahim Ahmed", "rahim@test.com", 21); // ✅
        
        try { createUser("", "email@test.com", 21); }
        catch (IllegalArgumentException e) { System.out.println("❌ " + e.getMessage()); }
        
        try { createUser("Rahim", "notanemail", 21); }
        catch (IllegalArgumentException e) { System.out.println("❌ " + e.getMessage()); }
        
        try { createUser("Rahim", "r@test.com", 200); }
        catch (IllegalArgumentException e) { System.out.println("❌ " + e.getMessage()); }
        
        System.out.println("\n=== BankAccount ===");
        BankAccount account = new BankAccount(5000.0);
        account.withdraw(2000.0);
        
        try { account.withdraw(10000.0); }   // too much
        catch (IllegalStateException e) { System.out.println("❌ " + e.getMessage()); }
        
        account.freeze();
        try { account.withdraw(100.0); }     // frozen
        catch (IllegalStateException e) { System.out.println("❌ " + e.getMessage()); }
        
        try { account.freeze(); }            // already frozen
        catch (IllegalStateException e) { System.out.println("❌ " + e.getMessage()); }
        
        System.out.println("\n=== Null checks ===");
        try { processOrder(null, "customer", "payment"); }
        catch (NullPointerException e) { System.out.println("❌ " + e.getMessage()); }
        
        processOrder("order", "customer", "payment"); // ✅
    }
}
```

---

## 6. Custom Exceptions — The Professional Standard

```java
// Every real application has custom exceptions.
// They make error handling clear, specific, and searchable.
// Spring Boot apps have dozens of custom exceptions.

public class CustomExceptions {
    
    // ─────────────────────────────────────────
    // CUSTOM EXCEPTION HIERARCHY
    // ─────────────────────────────────────────
    
    // Base exception for ALL app-specific exceptions:
    static class AppException extends RuntimeException {
        private final String errorCode;
        private final int httpStatus;
        
        public AppException(String errorCode, String message, int httpStatus) {
            super(message);
            this.errorCode  = errorCode;
            this.httpStatus = httpStatus;
        }
        
        public AppException(String errorCode, String message,
                            int httpStatus, Throwable cause) {
            super(message, cause);
            this.errorCode  = errorCode;
            this.httpStatus = httpStatus;
        }
        
        public String getErrorCode()  { return errorCode; }
        public int getHttpStatus()    { return httpStatus; }
    }
    
    // ── Domain-specific exceptions ──
    
    static class ResourceNotFoundException extends AppException {
        private final String resourceType;
        private final Object resourceId;
        
        public ResourceNotFoundException(String resourceType, Object resourceId) {
            super("RESOURCE_NOT_FOUND",
                  resourceType + " not found with ID: " + resourceId,
                  404); // HTTP 404
            this.resourceType = resourceType;
            this.resourceId   = resourceId;
        }
        
        public String getResourceType() { return resourceType; }
        public Object getResourceId()   { return resourceId; }
    }
    
    static class DuplicateResourceException extends AppException {
        public DuplicateResourceException(String resourceType, String field, Object value) {
            super("DUPLICATE_RESOURCE",
                  resourceType + " already exists with " + field + ": " + value,
                  409); // HTTP 409 Conflict
        }
    }
    
    static class ValidationException extends AppException {
        private final java.util.List<String> errors;
        
        public ValidationException(java.util.List<String> errors) {
            super("VALIDATION_FAILED",
                  "Validation failed: " + String.join(", ", errors),
                  400); // HTTP 400 Bad Request
            this.errors = errors;
        }
        
        public ValidationException(String error) {
            this(java.util.List.of(error));
        }
        
        public java.util.List<String> getErrors() { return errors; }
    }
    
    static class UnauthorizedException extends AppException {
        public UnauthorizedException(String message) {
            super("UNAUTHORIZED", message, 401);
        }
    }
    
    static class ForbiddenException extends AppException {
        public ForbiddenException(String action) {
            super("FORBIDDEN",
                  "You don't have permission to: " + action,
                  403);
        }
    }
    
    static class InsufficientFundsException extends AppException {
        private final double requested;
        private final double available;
        
        public InsufficientFundsException(double requested, double available) {
            super("INSUFFICIENT_FUNDS",
                  String.format("Insufficient funds. Requested: ৳%.2f, Available: ৳%.2f",
                                requested, available),
                  400);
            this.requested = requested;
            this.available = available;
        }
        
        public double getRequested() { return requested; }
        public double getAvailable() { return available; }
        public double getShortfall() { return requested - available; }
    }
    
    static class ExternalServiceException extends AppException {
        private final String serviceName;
        
        public ExternalServiceException(String serviceName, String message,
                                         Throwable cause) {
            super("EXTERNAL_SERVICE_ERROR",
                  serviceName + " failed: " + message,
                  503, // HTTP 503 Service Unavailable
                  cause);
            this.serviceName = serviceName;
        }
        
        public String getServiceName() { return serviceName; }
    }
    
    // ─────────────────────────────────────────
    // USING CUSTOM EXCEPTIONS IN SERVICES
    // ─────────────────────────────────────────
    
    static class User {
        Long id;
        String name;
        String email;
        String role;
        
        User(Long id, String name, String email, String role) {
            this.id = id; this.name = name;
            this.email = email; this.role = role;
        }
    }
    
    static class UserService {
        private final java.util.Map<Long, User> users = new java.util.HashMap<>();
        private final java.util.Set<String> emails   = new java.util.HashSet<>();
        private long nextId = 1;
        
        public User createUser(String name, String email, String role) {
            // Validate:
            java.util.List<String> errors = new java.util.ArrayList<>();
            if (name == null || name.isBlank())      errors.add("Name is required");
            if (email == null || !email.contains("@")) errors.add("Valid email is required");
            if (role == null || role.isBlank())      errors.add("Role is required");
            
            if (!errors.isEmpty()) throw new ValidationException(errors);
            
            // Check duplicate:
            if (emails.contains(email.toLowerCase())) {
                throw new DuplicateResourceException("User", "email", email);
            }
            
            User user = new User(nextId++, name.trim(), email.toLowerCase(), role);
            users.put(user.id, user);
            emails.add(user.email);
            return user;
        }
        
        public User getUser(Long id) {
            User user = users.get(id);
            if (user == null) {
                throw new ResourceNotFoundException("User", id);
            }
            return user;
        }
        
        public User requireAdminAccess(Long userId, Long requesterId) {
            User requester = getUser(requesterId); // might throw ResourceNotFoundException
            if (!"ADMIN".equals(requester.role)) {
                throw new ForbiddenException("manage users");
            }
            return getUser(userId);
        }
        
        public void deleteUser(Long userId, String requesterToken) {
            if (requesterToken == null || !requesterToken.startsWith("Bearer ")) {
                throw new UnauthorizedException("Authentication required");
            }
            User user = getUser(userId);
            users.remove(userId);
            emails.remove(user.email);
            System.out.println("User deleted: " + user.name);
        }
    }
    
    static class PaymentService {
        public void transfer(Long fromId, Long toId, double amount,
                              double fromBalance) {
            if (amount <= 0) {
                throw new ValidationException("Amount must be positive");
            }
            if (amount > fromBalance) {
                throw new InsufficientFundsException(amount, fromBalance);
            }
            
            // Simulate external payment call:
            try {
                if (Math.random() < 0.1) { // 10% failure rate for demo
                    throw new RuntimeException("Gateway timeout");
                }
                System.out.println("Transfer successful: ৳" + amount);
            } catch (RuntimeException e) {
                throw new ExternalServiceException("PaymentGateway", e.getMessage(), e);
            }
        }
    }
    
    // ─────────────────────────────────────────
    // GLOBAL EXCEPTION HANDLER (Spring Boot style)
    // ─────────────────────────────────────────
    
    static class GlobalExceptionHandler {
        // In Spring Boot: @RestControllerAdvice @ExceptionHandler
        // Here: simulated handler
        
        public void handle(AppException e) {
            System.out.println("═".repeat(50));
            System.out.printf("HTTP %d: [%s] %s%n",
                              e.getHttpStatus(), e.getErrorCode(), e.getMessage());
            if (e instanceof ValidationException ve) {
                System.out.println("Validation errors:");
                ve.getErrors().forEach(err -> System.out.println("  • " + err));
            }
            if (e instanceof InsufficientFundsException ife) {
                System.out.printf("Shortfall: ৳%.2f%n", ife.getShortfall());
            }
            if (e.getCause() != null) {
                System.out.println("Caused by: " + e.getCause().getMessage());
            }
            System.out.println("═".repeat(50));
        }
    }
    
    public static void main(String[] args) {
        UserService userService = new UserService();
        PaymentService paymentService = new PaymentService();
        GlobalExceptionHandler handler = new GlobalExceptionHandler();
        
        System.out.println("=== Creating users ===");
        User admin = userService.createUser("Admin User", "admin@test.com", "ADMIN");
        User rahim = userService.createUser("Rahim Ahmed", "rahim@test.com", "USER");
        System.out.println("Created: " + admin.name + " (ADMIN)");
        System.out.println("Created: " + rahim.name + " (USER)");
        
        System.out.println("\n=== Validation error ===");
        try {
            userService.createUser("", "bad-email", "");
        } catch (ValidationException e) {
            handler.handle(e);
        }
        
        System.out.println("\n=== Duplicate email ===");
        try {
            userService.createUser("Rahim 2", "rahim@test.com", "USER");
        } catch (DuplicateResourceException e) {
            handler.handle(e);
        }
        
        System.out.println("\n=== Resource not found ===");
        try {
            userService.getUser(999L);
        } catch (ResourceNotFoundException e) {
            handler.handle(e);
        }
        
        System.out.println("\n=== Unauthorized ===");
        try {
            userService.deleteUser(rahim.id, null);
        } catch (UnauthorizedException e) {
            handler.handle(e);
        }
        
        System.out.println("\n=== Forbidden ===");
        try {
            userService.requireAdminAccess(admin.id, rahim.id);
        } catch (ForbiddenException e) {
            handler.handle(e);
        }
        
        System.out.println("\n=== Insufficient funds ===");
        try {
            paymentService.transfer(1L, 2L, 10000.0, 5000.0);
        } catch (InsufficientFundsException e) {
            handler.handle(e);
        }
        
        System.out.println("\n=== Successful payment ===");
        try {
            paymentService.transfer(1L, 2L, 2000.0, 5000.0);
        } catch (ExternalServiceException e) {
            handler.handle(e);
        }
    }
}
```

---

## 7. Exception Best Practices

```java
public class ExceptionBestPractices {
    
    // ─────────────────────────────────────────
    // ✅ DO: Specific exception types
    // ─────────────────────────────────────────
    
    // ❌ BAD: Too generic
    public static void badValidation(String email) throws Exception {
        if (!email.contains("@")) throw new Exception("Invalid email");
    }
    
    // ✅ GOOD: Specific type with clear message
    public static void goodValidation(String email) {
        if (!email.contains("@"))
            throw new IllegalArgumentException("Email must contain '@': " + email);
    }
    
    // ─────────────────────────────────────────
    // ✅ DO: Preserve the original cause
    // ─────────────────────────────────────────
    
    // ❌ BAD: Loses the original exception
    public static void badCatch() {
        try {
            throw new java.io.IOException("File not found");
        } catch (java.io.IOException e) {
            throw new RuntimeException("Error processing file"); // cause LOST!
        }
    }
    
    // ✅ GOOD: Preserves original as cause
    public static void goodCatch() {
        try {
            throw new java.io.IOException("File not found");
        } catch (java.io.IOException e) {
            throw new RuntimeException("Error processing file", e); // cause PRESERVED!
            // e.getCause() returns the IOException
        }
    }
    
    // ─────────────────────────────────────────
    // ✅ DO: Catch specific, not everything
    // ─────────────────────────────────────────
    
    // ❌ BAD: Swallowing all exceptions silently
    public static int badParsing(String s) {
        try {
            return Integer.parseInt(s);
        } catch (Exception e) {
            return 0; // SILENT FAILURE — bugs hidden!
        }
    }
    
    // ✅ GOOD: Handle specifically, log meaningfully
    public static int goodParsing(String s) {
        try {
            return Integer.parseInt(s);
        } catch (NumberFormatException e) {
            System.out.println("[WARN] Cannot parse '" + s + "' as int, using 0");
            return 0; // at least we LOG it
        }
    }
    
    // ─────────────────────────────────────────
    // ✅ DO: Don't use exceptions for control flow
    // ─────────────────────────────────────────
    
    // ❌ BAD: Exception as control flow (slow + unreadable)
    public static boolean userExistsBad(String email,
                                         java.util.List<String> emails) {
        try {
            int idx = emails.indexOf(email);
            if (idx < 0) throw new RuntimeException("Not found");
            return true;
        } catch (RuntimeException e) {
            return false;
        }
    }
    
    // ✅ GOOD: Use return values for expected conditions
    public static boolean userExistsGood(String email,
                                          java.util.List<String> emails) {
        return emails.contains(email); // simple, fast, readable
    }
    
    // ─────────────────────────────────────────
    // ✅ DO: Log at the right level
    // ─────────────────────────────────────────
    
    public static void properLogging(String userId) {
        try {
            // ... operation ...
            System.out.println("[INFO] Processing user: " + userId);
        } catch (IllegalArgumentException e) {
            // WARN: expected business error — not a system problem
            System.out.println("[WARN] Invalid user input: " + e.getMessage());
        } catch (RuntimeException e) {
            // ERROR: unexpected error — needs investigation
            System.out.println("[ERROR] Unexpected error for user " + userId +
                               ": " + e.getMessage());
            // In real code: log.error("...", e); — logs full stack trace
        }
    }
    
    // ─────────────────────────────────────────
    // ✅ DO: Never catch Error
    // ─────────────────────────────────────────
    
    // ❌ BAD: Catching Error (don't do this!)
    public static void badErrorCatch() {
        try {
            // some code
        } catch (Error e) {
            // You cannot recover from StackOverflowError or OutOfMemoryError!
            // Don't catch Error
        }
    }
    
    // ─────────────────────────────────────────
    // ✅ DO: Early return / guard clauses
    // ─────────────────────────────────────────
    
    // ❌ BAD: nested try-catch
    public static String badProcess(String input) {
        String result = null;
        try {
            if (input != null) {
                try {
                    result = input.trim();
                    try {
                        result = result.toLowerCase();
                    } catch (Exception e3) { }
                } catch (Exception e2) { }
            }
        } catch (Exception e1) { }
        return result;
    }
    
    // ✅ GOOD: fail fast with guard clauses
    public static String goodProcess(String input) {
        if (input == null) {
            throw new IllegalArgumentException("Input cannot be null");
        }
        return input.trim().toLowerCase();
    }
    
    // ─────────────────────────────────────────
    // ✅ DO: Meaningful exception messages
    // ─────────────────────────────────────────
    
    public static void setAge(int age) {
        // ❌ BAD: No context
        // if (age < 0) throw new IllegalArgumentException("Invalid age");
        
        // ✅ GOOD: Full context — what was received, what's expected
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException(
                "Age must be between 0 and 150, received: " + age);
        }
    }
    
    // ─────────────────────────────────────────
    // QUICK REFERENCE — Exception selection
    // ─────────────────────────────────────────
    //
    // Null argument          → NullPointerException or
    //                          IllegalArgumentException("X cannot be null")
    // Invalid argument value → IllegalArgumentException
    // Invalid state          → IllegalStateException
    // Index out of bounds    → IndexOutOfBoundsException
    // Not found              → Custom: ResourceNotFoundException (extends RuntimeException)
    // Not authorized         → Custom: UnauthorizedException
    // Not permitted          → Custom: ForbiddenException
    // Network/external fail  → Custom: ExternalServiceException
    // Not implemented yet    → UnsupportedOperationException
    // Arithmetic error       → ArithmeticException
    // Conversion failed      → NumberFormatException
    // Concurrent modification→ ConcurrentModificationException
    
    public static void main(String[] args) {
        
        System.out.println("=== Exception selection demo ===");
        
        // goodValidation:
        try { goodValidation("notanemail"); }
        catch (IllegalArgumentException e) { System.out.println("✅ " + e.getMessage()); }
        
        // goodParsing:
        System.out.println(goodParsing("42"));   // 42
        System.out.println(goodParsing("abc"));  // 0 (with warning)
        
        // goodProcess:
        System.out.println(goodProcess("  HELLO  ")); // hello
        try { goodProcess(null); }
        catch (IllegalArgumentException e) { System.out.println("✅ " + e.getMessage()); }
        
        // setAge:
        try { setAge(-5); }
        catch (IllegalArgumentException e) { System.out.println("✅ " + e.getMessage()); }
    }
}
```

---

## Build This — Complete Exception Handling System

```java
// File: RobustSystem.java
// A robust system with comprehensive exception handling
// Simulating a real Spring Boot service layer

import java.util.*;
import java.time.*;

public class RobustSystem {

    // ═══════════════════════════════════════
    // EXCEPTION HIERARCHY
    // ═══════════════════════════════════════
    
    static class AppException extends RuntimeException {
        private final String code;
        private final int status;
        
        public AppException(String code, String message, int status) {
            super(message);
            this.code = code; this.status = status;
        }
        public AppException(String code, String message, int status, Throwable cause) {
            super(message, cause);
            this.code = code; this.status = status;
        }
        
        public String getCode()   { return code; }
        public int getStatus()    { return status; }
        
        @Override
        public String toString() {
            return String.format("[%d][%s] %s", status, code, getMessage());
        }
    }
    
    static class NotFoundException      extends AppException {
        public NotFoundException(String resource, Object id) {
            super("NOT_FOUND", resource + " not found: " + id, 404);
        }
    }
    static class ConflictException      extends AppException {
        public ConflictException(String message) {
            super("CONFLICT", message, 409);
        }
    }
    static class BadRequestException    extends AppException {
        public BadRequestException(String message) {
            super("BAD_REQUEST", message, 400);
        }
    }
    static class UnauthorizedException  extends AppException {
        public UnauthorizedException()  { super("UNAUTHORIZED", "Authentication required", 401); }
    }
    static class ServiceUnavailableException extends AppException {
        public ServiceUnavailableException(String service, Throwable cause) {
            super("SERVICE_UNAVAILABLE", service + " is temporarily unavailable", 503, cause);
        }
    }

    // ═══════════════════════════════════════
    // DOMAIN ENTITIES
    // ═══════════════════════════════════════
    
    record Product(String id, String name, double price, int stock) {
        Product {
            if (name == null || name.isBlank()) throw new BadRequestException("Name required");
            if (price <= 0) throw new BadRequestException("Price must be positive");
            if (stock < 0)  throw new BadRequestException("Stock cannot be negative");
        }
        
        Product withStock(int newStock) {
            return new Product(id, name, price, newStock);
        }
    }
    
    record Order(String id, String productId, int quantity,
                 double total, String status, LocalDateTime createdAt) { }

    // ═══════════════════════════════════════
    // REPOSITORY LAYER
    // ═══════════════════════════════════════
    
    static class ProductRepository {
        private final Map<String, Product> store = new HashMap<>();
        private int counter = 1;
        
        public Product save(Product product) {
            String id = product.id() != null ? product.id() : "PRD-" + counter++;
            Product saved = new Product(id, product.name(), product.price(), product.stock());
            store.put(id, saved);
            return saved;
        }
        
        public Optional<Product> findById(String id) {
            return Optional.ofNullable(store.get(id));
        }
        
        public Product getById(String id) {
            return findById(id).orElseThrow(() -> new NotFoundException("Product", id));
        }
        
        public List<Product> findAll() {
            return new ArrayList<>(store.values());
        }
        
        public boolean existsByName(String name) {
            return store.values().stream()
                .anyMatch(p -> p.name().equalsIgnoreCase(name));
        }
    }
    
    static class OrderRepository {
        private final Map<String, Order> store = new HashMap<>();
        private int counter = 1;
        
        public Order save(Order order) {
            String id = "ORD-" + String.format("%05d", counter++);
            Order saved = new Order(id, order.productId(), order.quantity(),
                                    order.total(), order.status(), order.createdAt());
            store.put(id, saved);
            return saved;
        }
        
        public Optional<Order> findById(String id) {
            return Optional.ofNullable(store.get(id));
        }
        
        public Order getById(String id) {
            return findById(id).orElseThrow(() -> new NotFoundException("Order", id));
        }
        
        public void updateStatus(String id, String status) {
            Order order = getById(id);
            store.put(id, new Order(id, order.productId(), order.quantity(),
                                    order.total(), status, order.createdAt()));
        }
        
        public List<Order> findAll() { return new ArrayList<>(store.values()); }
    }

    // ═══════════════════════════════════════
    // EXTERNAL SERVICE (simulated)
    // ═══════════════════════════════════════
    
    static class PaymentGateway {
        public boolean charge(double amount, String token) {
            // Simulate: 5% of calls fail
            if (Math.random() < 0.05) {
                throw new RuntimeException("Gateway connection timeout");
            }
            System.out.printf("  💳 Charged ৳%.2f via token %s%n", amount, token);
            return true;
        }
    }

    // ═══════════════════════════════════════
    // SERVICE LAYER
    // ═══════════════════════════════════════
    
    static class ProductService {
        private final ProductRepository repo;
        
        public ProductService(ProductRepository repo) { this.repo = repo; }
        
        public Product createProduct(String name, double price, int stock) {
            Objects.requireNonNull(name, "Name cannot be null");
            
            if (repo.existsByName(name)) {
                throw new ConflictException("Product already exists with name: " + name);
            }
            
            // Constructor validates — throws BadRequestException if invalid:
            Product product = new Product(null, name, price, stock);
            Product saved   = repo.save(product);
            System.out.println("✅ Product created: " + saved.id());
            return saved;
        }
        
        public Product getProduct(String id) {
            return repo.getById(id); // throws NotFoundException if missing
        }
        
        public List<Product> getAllProducts() {
            return repo.findAll();
        }
        
        public Product restockProduct(String id, int quantity) {
            if (quantity <= 0) {
                throw new BadRequestException("Restock quantity must be positive: " + quantity);
            }
            Product product = repo.getById(id);
            Product restocked = product.withStock(product.stock() + quantity);
            repo.save(restocked);
            System.out.printf("✅ Restocked %s: +%d units (total: %d)%n",
                              product.name(), quantity, restocked.stock());
            return restocked;
        }
    }
    
    static class OrderService {
        private final ProductRepository productRepo;
        private final OrderRepository orderRepo;
        private final PaymentGateway paymentGateway;
        private int retryCount = 3;
        
        public OrderService(ProductRepository productRepo,
                            OrderRepository orderRepo,
                            PaymentGateway paymentGateway) {
            this.productRepo    = productRepo;
            this.orderRepo      = orderRepo;
            this.paymentGateway = paymentGateway;
        }
        
        public Order placeOrder(String productId, int quantity,
                                 String authToken) {
            // Authentication check:
            if (authToken == null || !authToken.startsWith("Bearer ")) {
                throw new UnauthorizedException();
            }
            
            // Validate quantity:
            if (quantity <= 0) {
                throw new BadRequestException("Quantity must be positive: " + quantity);
            }
            
            // Get product — throws NotFoundException if not found:
            Product product = productRepo.getById(productId);
            
            // Check stock:
            if (product.stock() < quantity) {
                throw new BadRequestException(
                    String.format("Insufficient stock. Requested: %d, Available: %d",
                                  quantity, product.stock()));
            }
            
            double total = product.price() * quantity;
            
            // Process payment with retry:
            processPaymentWithRetry(total, authToken);
            
            // Deduct stock:
            productRepo.save(product.withStock(product.stock() - quantity));
            
            // Create order:
            Order order = orderRepo.save(new Order(null, productId, quantity,
                                                    total, "PLACED", LocalDateTime.now()));
            System.out.printf("✅ Order placed: %s (৳%.2f)%n", order.id(), total);
            return order;
        }
        
        private void processPaymentWithRetry(double amount, String token) {
            int attempts = 0;
            Exception lastException = null;
            
            while (attempts < retryCount) {
                try {
                    paymentGateway.charge(amount, token.replace("Bearer ", ""));
                    return; // success — exit
                } catch (RuntimeException e) {
                    attempts++;
                    lastException = e;
                    System.out.printf("  ⚠️  Payment attempt %d failed: %s%n",
                                      attempts, e.getMessage());
                    if (attempts < retryCount) {
                        System.out.println("  🔄 Retrying...");
                    }
                }
            }
            
            // All retries exhausted:
            throw new ServiceUnavailableException("PaymentGateway", lastException);
        }
        
        public Order cancelOrder(String orderId, String authToken) {
            if (authToken == null || !authToken.startsWith("Bearer ")) {
                throw new UnauthorizedException();
            }
            
            Order order = orderRepo.getById(orderId);
            
            if ("CANCELLED".equals(order.status())) {
                throw new ConflictException("Order already cancelled: " + orderId);
            }
            if ("DELIVERED".equals(order.status())) {
                throw new ConflictException("Cannot cancel delivered order: " + orderId);
            }
            
            // Restore stock:
            Product product = productRepo.getById(order.productId());
            productRepo.save(product.withStock(product.stock() + order.quantity()));
            
            orderRepo.updateStatus(orderId, "CANCELLED");
            System.out.println("✅ Order cancelled: " + orderId);
            return orderRepo.getById(orderId);
        }
        
        public List<Order> getAllOrders() { return orderRepo.findAll(); }
    }

    // ═══════════════════════════════════════
    // EXCEPTION HANDLER (Spring-style)
    // ═══════════════════════════════════════
    
    static class ExceptionHandler {
        public void handle(String operation, Runnable action) {
            try {
                action.run();
            } catch (NotFoundException e) {
                System.out.printf("🔍 [%s] %s%n", operation, e);
            } catch (ConflictException e) {
                System.out.printf("⚡ [%s] %s%n", operation, e);
            } catch (BadRequestException e) {
                System.out.printf("❌ [%s] %s%n", operation, e);
            } catch (UnauthorizedException e) {
                System.out.printf("🔐 [%s] %s%n", operation, e);
            } catch (ServiceUnavailableException e) {
                System.out.printf("🔥 [%s] %s%n", operation, e);
                System.out.printf("   Caused by: %s%n", e.getCause().getMessage());
            } catch (AppException e) {
                System.out.printf("⚠️  [%s] %s%n", operation, e);
            }
        }
    }

    // ═══════════════════════════════════════
    // MAIN
    // ═══════════════════════════════════════
    
    public static void main(String[] args) {
        
        // Setup:
        ProductRepository productRepo = new ProductRepository();
        OrderRepository   orderRepo   = new OrderRepository();
        PaymentGateway    gateway     = new PaymentGateway();
        ProductService    products    = new ProductService(productRepo);
        OrderService      orders      = new OrderService(productRepo, orderRepo, gateway);
        ExceptionHandler  handler     = new ExceptionHandler();
        
        String token = "Bearer valid-jwt-token";
        
        System.out.println("╔══════════════════════════════════════════╗");
        System.out.println("║         ROBUST SYSTEM DEMO               ║");
        System.out.println("╚══════════════════════════════════════════╝\n");
        
        // ── Product management ──
        System.out.println("── Creating Products ──");
        Product laptop  = products.createProduct("Laptop Pro", 75000.0, 10);
        Product phone   = products.createProduct("Smartphone X", 45000.0, 25);
        Product earbuds = products.createProduct("Earbuds Pro", 8500.0, 50);
        
        // Duplicate name:
        handler.handle("CREATE_DUPLICATE",
            () -> products.createProduct("Laptop Pro", 80000.0, 5));
        
        // Invalid price:
        handler.handle("CREATE_INVALID",
            () -> products.createProduct("Test", -100.0, 5));
        
        // ── Order placement ──
        System.out.println("\n── Placing Orders ──");
        Order order1 = orders.placeOrder(laptop.id(), 2, token);
        Order order2 = orders.placeOrder(phone.id(), 1, token);
        Order order3 = orders.placeOrder(earbuds.id(), 5, token);
        
        // No auth:
        handler.handle("NO_AUTH",
            () -> orders.placeOrder(laptop.id(), 1, null));
        
        // Not found:
        handler.handle("NOT_FOUND",
            () -> orders.placeOrder("NONEXISTENT", 1, token));
        
        // Insufficient stock (2 laptops ordered already, 8 remain):
        handler.handle("INSUFFICIENT_STOCK",
            () -> orders.placeOrder(laptop.id(), 9, token)); // only 8 left
        
        // ── Cancellation ──
        System.out.println("\n── Cancellations ──");
        orders.cancelOrder(order2.id(), token);
        
        // Cancel again:
        handler.handle("DOUBLE_CANCEL",
            () -> orders.cancelOrder(order2.id(), token));
        
        // Cancel non-existent:
        handler.handle("CANCEL_MISSING",
            () -> orders.cancelOrder("ORD-99999", token));
        
        // ── Summary ──
        System.out.println("\n── Product Stock After Orders ──");
        products.getAllProducts().forEach(p ->
            System.out.printf("  %-20s: %d units%n", p.name(), p.stock()));
        
        System.out.println("\n── All Orders ──");
        orders.getAllOrders().forEach(o ->
            System.out.printf("  %s: %-12s qty=%d total=৳%.2f%n",
                              o.id(), o.status(), o.quantity(), o.total()));
    }
}
```

---

## Exercises

```text
EXERCISE 1: Parse and Validate
  Create ParseAndValidate.java
  Build a DataParser class with these SAFE parsing methods:
  - parseInt(String s) → Optional<Integer> (empty on failure)
  - parseDouble(String s) → Optional<Double>
  - parseBoolean(String s) → Optional<Boolean>
    (accepts: true/false, yes/no, 1/0, on/off)
  - parseDate(String s, String format) → Optional<LocalDate>
  - parseEnum(String s, Class<E> enumClass) → Optional<E>

  Build a UserInputValidator:
  - validateAge(String input) → int (throw with specific message if invalid)
  - validateEmail(String input) → String (cleaned)
  - validatePassword(String input) → validate strength, throw listing all failures
  - validatePhoneBD(String input) → normalized format
  - validateAmount(String input, double min, double max) → double

  Test with valid and invalid inputs for each.

EXERCISE 2: Custom Exception Hierarchy
  Create ExceptionHierarchy.java
  Build a complete exception hierarchy for a banking system:

  BankException (base)
  ├── AccountException
  │   ├── AccountNotFoundException
  │   ├── AccountFrozenException
  │   └── AccountClosedException
  ├── TransactionException
  │   ├── InsufficientFundsException
  │   ├── DailyLimitExceededException
  │   └── InvalidAmountException
  ├── AuthException
  │   ├── InvalidPinException(attemptsLeft int)
  │   ├── SessionExpiredException
  │   └── AccountLockedAfterMaxAttempts
  └── ExternalException
      ├── PaymentGatewayException
      └── CbsConnectionException (Core Banking System)

  Each exception has appropriate fields, constructors, messages.
  Build a BankATM simulation using all exceptions.
  Global handler catches by hierarchy level.

EXERCISE 3: Retry with Exception Handling
  Create RetrySystem.java

  interface Operation<T> { T execute() throws Exception; }

  class Retry {
    static <T> T withRetry(
      Operation<T> op,
      int maxAttempts,
      Class<? extends Exception>[] retryOn,
      long delayMs
    ) throws Exception

    static <T> T withExponentialBackoff(
      Operation<T> op,
      int maxAttempts,
      long initialDelayMs
    ) throws Exception
  }

  Test with:
  - HTTP call that fails first 2 times (mock)
  - Database connection that fails intermittently
  - Operation that always fails
  - Operation that succeeds on 3rd try

  Log each attempt.

EXERCISE 4: Exception-Safe Resource Pool
  Create ResourcePool.java

  Generic resource pool that:
  - Holds pool of AutoCloseable resources
  - Provides: acquire(), release(), executeWithResource(Operation<T>)
  - executeWithResource() always returns resource to pool
  - Even if operation throws, resource is released
  - Pool tracks: available, in-use, total
  - Throws PoolExhaustedException if no resource available
  - Throws ResourceCreationException if can't create new one

  Test with DatabaseConnection pool.
  Demonstrate resource always returned even on exception.

EXERCISE 5: Spring-Style Exception System
  Create SpringStyleExceptions.java

  Simulate Spring Boot's exception handling:

  @ControllerAdvice equivalent → GlobalExceptionHandler class
  @ExceptionHandler equivalent → handle(ExceptionType e) methods

  ErrorResponse record(timestamp, status, error, message, path)

  Handler for:
  - NotFoundException → 404 with resource name
  - ValidationException → 400 with List<FieldError>
  - ConflictException → 409
  - UnauthorizedException → 401
  - RuntimeException → 500 (never expose internal details!)

  Build a ProductController simulation that:
  - Creates, finds, updates, deletes products
  - All operations wrapped in exception handling
  - Returns appropriate ErrorResponse for each failure

  Push to GitHub: "feat: robust system with exception handling"
```

---

## Common Mistakes

```text
MISTAKE 1: Catching Exception (too broad) and doing nothing
  try { ... }
  catch (Exception e) { } // SILENT FAILURE — worst pattern in Java
  // Bugs are hidden. Nothing is logged. Nightmare to debug.
  Fix: at minimum, log the exception. Better: handle specifically.

MISTAKE 2: Losing the original cause
  catch (IOException e) {
      throw new RuntimeException("Failed"); // cause LOST!
  }
  Fix: throw new RuntimeException("Failed", e); // preserve cause

MISTAKE 3: Catching Error
  catch (Error e) { ... } // Never do this
  // OutOfMemoryError, StackOverflowError — you cannot recover.
  Fix: let Error propagate. Let the JVM/container handle it.

MISTAKE 4: Using exceptions for normal control flow
  try {
      int val = Integer.parseInt(input);
  } catch (NumberFormatException e) {
      return false; // exception as if-else — slow and unreadable
  }
  Fix: validate before parsing, or use methods that return Optional.

MISTAKE 5: Not closing resources (resource leak)
  Connection conn = getConnection();
  conn.doWork(); // if this throws, conn is NEVER closed!
  conn.close();
  Fix: always use try-with-resources.

MISTAKE 6: Order of catch blocks (broader catches narrower)
  catch (Exception e) { ... } // catches EVERYTHING
  catch (IOException e) { ... } // COMPILE ERROR: unreachable
  Fix: specific exceptions BEFORE general ones.

MISTAKE 7: Rethrowing without the original
  catch (SQLException e) {
      throw new RuntimeException("DB error"); // original stack trace LOST
  }
  Fix: throw new RuntimeException("DB error", e);

MISTAKE 8: Empty finally or return from finally
  finally {
      return "default"; // OVERRIDES return from try! Silent bug.
  }
  Fix: never return from finally. Only cleanup code in finally.

MISTAKE 9: Checked exceptions everywhere (exception hell)
  void a() throws A {}
  void b() throws A, B {} // must declare everything
  void c() throws A, B, C {} // grows and grows
  Fix: wrap checked in RuntimeException early.
  Modern Java: use unchecked exceptions primarily.

MISTAKE 10: Null message in exceptions
  throw new IllegalArgumentException(); // no message!
  // How will anyone know what was illegal?
  Fix: always provide a clear, specific message:
  throw new IllegalArgumentException(
    "Age must be between 0-150, received: " + age);
```

---

## Interview Questions

**Q: What is the difference between checked and unchecked exceptions?**
A: Checked exceptions extend Exception (but not RuntimeException). Java FORCES you to handle them: either with try-catch or by declaring 'throws' in the method signature. They represent conditions the caller might reasonably recover from: file not found, network timeout, database error. Unchecked exceptions extend RuntimeException (or Error). Java does NOT force handling — they can propagate freely. They represent programming errors or truly unexpected conditions: NullPointerException, IllegalArgumentException, ArrayIndexOutOfBounds. Modern Java practice (and Spring Boot) uses unchecked exceptions almost exclusively, as checked exceptions can lead to cluttered code when propagated through many layers.

**Q: What is try-with-resources and why is it better than finally?**
A: try-with-resources (Java 7+) automatically calls close() on resources that implement AutoCloseable when the try block exits, whether normally or via exception. Unlike manual finally blocks: 1) It's cleaner — no null checks before close(). 2) If the try block throws AND close() throws, the close() exception is SUPPRESSED (attached to primary) instead of HIDING the primary exception (which happens with finally). 3) Multiple resources closed in reverse declaration order automatically. Always prefer try-with-resources for any Closeable/AutoCloseable resource: connections, streams, files.

**Q: What is the difference between throw and throws?**
A: throw is an ACTION inside a method body that creates and throws an exception instance: throw new IllegalArgumentException("bad"). Execution stops immediately and the exception propagates up. throws is a DECLARATION in a method signature that warns callers this method might throw that checked exception type, requiring them to handle or propagate it: public void method() throws IOException { } throw is used at runtime; throws is a compile-time declaration.

**Q: Why should you always preserve the original exception as cause?**
A: When wrapping exceptions, the original exception contains the full stack trace and context of what actually went wrong — the "root cause." If you throw new RuntimeException("error") without including the original, you lose: the original exception type, the original message, and the original stack trace showing exactly where the problem occurred. Debugging becomes extremely difficult. Always use: throw new RuntimeException("message", e) where e is the original exception. This keeps the full chain accessible via getCause() and in stack traces.

**Q: When should you create custom exceptions?**
A: Create custom exceptions when: 1) A built-in type doesn't clearly express the domain error (ResourceNotFoundException is clearer than RuntimeException). 2) Callers need to distinguish your errors from others (catch ResourceNotFoundException separately from other errors). 3) You need to carry extra information with the error (InsufficientFundsException carrying requested vs available amounts). 4) You're building a library or API (custom exceptions document the API's failure modes clearly). 5) You need HTTP status mapping (each exception maps to a status code — very common in Spring Boot REST APIs). Naming convention: end with Exception or Error. Extend RuntimeException for most cases (unchecked — cleaner APIs).

**Q: What is the exception handling pattern in Spring Boot?**
A: Spring Boot uses @ControllerAdvice and @ExceptionHandler for global exception handling. A class annotated with @RestControllerAdvice has methods annotated with @ExceptionHandler(ExceptionType.class) — Spring automatically calls the matching handler when that exception type is thrown from any controller, service, or repository. Each handler returns an appropriate HTTP response (status + body). Best practice: define a custom exception hierarchy where each exception maps to an HTTP status (NotFoundException → 404, BadRequestException → 400). The global handler catches each type and returns a structured error response (timestamp, status, code, message). NEVER expose internal details (stack traces, SQL errors) in API responses — return clean, user-friendly error messages.

---

## Key Takeaways

```text
1. EXCEPTION HIERARCHY:
   Throwable → Error (JVM, never catch)
   Throwable → Exception → checked (must handle)
   Throwable → Exception → RuntimeException → unchecked (optional handle)

2. CHECKED vs UNCHECKED:
   Checked: compiler forces handling (IOException, SQLException)
   Unchecked: optional handling (RuntimeException, NullPointerException)
   Modern Java: prefer unchecked — cleaner code, no exception hell.

3. try-catch-finally:
   try: code that might throw
   catch: handle specific exception (most specific FIRST)
   finally: ALWAYS runs — use for cleanup
   Never return from finally — it overrides try's return!

4. try-with-resources (Java 7+):
   For AutoCloseable resources — ALWAYS use this.
   Automatically calls close() when block exits.
   Handles suppressed exceptions correctly.
   No more "forget to close" bugs.

5. throw: manually throw an exception (action, in method body)
   throws: declare exceptions method might throw (signature, compile-time)
   NEVER confuse these two.

6. ALWAYS preserve original cause:
   throw new RuntimeException("message", originalException);
   getCause() gives access to the chain.
   Lose the cause → lose the stack trace → debugging hell.

7. CUSTOM EXCEPTIONS:
   One base AppException with errorCode and httpStatus.
   Domain-specific children: NotFoundException, ValidationException, etc.
   Always extend RuntimeException in modern Java.
   Add relevant fields for context (requestedAmount, resourceId, etc.).

8. BEST PRACTICES:
   ✅ Catch specific types first, general last
   ✅ Always log exceptions (never silent catch)
   ✅ Meaningful messages with context
   ✅ Use try-with-resources for any resource
   ✅ Fail fast with guard clauses
   ❌ Never catch Error
   ❌ Never swallow exceptions silently
   ❌ Never use exceptions for control flow
   ❌ Never expose internal details in API responses

9. EXCEPTION HIERARCHY in Spring Boot:
   Custom exceptions → Spring @ExceptionHandler → HTTP response
   Each exception type maps to an HTTP status code.
   Global handler catches everything — consistent error responses.

10. throw EXITS immediately. Code after throw is unreachable.
    Always throw from validation, never return null.
    "Crash loudly early" is better than "silently corrupt data".
```

---