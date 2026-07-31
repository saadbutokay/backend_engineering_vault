**Phase:** Level 1 - Java Fundamentals
**Date Studied:**

---
## What Problem Does This Solve?
Imagine you're building a payment system.
Without methods, this is what happens:
```java
main() {
    // Validate user
    if (user == null) { ... }
    if (user.email == null) { ... }
    if (!user.email.contains("@")) { ... }
    
    // Calculate order total
    double total = 0;
    for (item : cart) { total += item.price * item.qty; }
    if (isPremium) { total *= 0.9; }
    total += calculateTax(total);
    
    // Process payment
    if (total <= 0) { ... }
    if (paymentMethod == null) { ... }
    // ... 50 more lines of payment logic
    
    // Send confirmation email
    // ... 30 more lines of email logic
}
```

```
main() becomes 500+ lines of code.
You can't find anything.
You can't test anything in isolation.
You repeat the same validation logic in 10 places.
When validation rules change, you update 10 places.
You miss one. Bug in production. Angry customers.
```

With methods, this becomes:
```java
main() {
    validateUser(user);
    double total = calculateOrderTotal(cart, isPremium);
    processPayment(total, paymentMethod);
    sendConfirmationEmail(user, order);
}

// 4 lines. Clear intent. Each piece tested independently.
// Change validation? Update ONE method. Done.
```
Methods are the fundamental unit of code organization.
Everything you write in Java from now on is built from methods.
Every Spring Boot endpoint, every service method, every utility —
all methods.

---
## What Is a Method?
```text
A method is a NAMED BLOCK of code that:
  → Does ONE specific thing (Single Responsibility Principle)
  → Can be CALLED (invoked) from anywhere with access
  → Can accept INPUT (parameters)
  → Can return OUTPUT (return value)
  → Can be called REPEATEDLY without rewriting the code

The relationship between a METHOD and a FUNCTION:
  In Java: they're called METHODS (inside a class)
  In other languages: they might be called FUNCTIONS
  Functionally: the same concept
  Technically: a method is a function that belongs to a class

ANATOMY OF A METHOD:

  access   return
  modifier type    name      parameters
     │       │      │             │
     ▼       ▼      ▼             ▼
  public  double  calculateTax(double amount, double rate) {
      
      double tax = amount * rate;   ← method BODY
      return tax;                   ← return statement
      
  }
  
  Every part has a purpose.
  Every part you must understand.
```

---
## 1. Method Structure - Every Part Explained
```java
public class MethodAnatomy {
    
    // ─────────────────────────────────────────────────────────
    // THE FULL METHOD SIGNATURE BREAKDOWN
    // ─────────────────────────────────────────────────────────
    
    /**
     * Calculates the tax amount for a given price.
     * (Javadoc comment — documents what the method does)
     *
     * @param amount  the base amount to calculate tax on
     * @param rate    the tax rate as a decimal (e.g., 0.15 for 15%)
     * @return        the tax amount (not the total, just the tax)
     */
    public static double calculateTax(double amount, double rate) {
        //  │       │      │             │
        //  │       │      │             └─ PARAMETERS: inputs the method needs
        //  │       │      └─────────────── NAME: verb describing what it does
        //  │       └────────────────────── RETURN TYPE: what it gives back
        //  └────────────────────────────── ACCESS + MODIFIER
        
        // METHOD BODY: the actual logic
        if (amount <= 0) {
            return 0; // early return for invalid input
        }
        
        double tax = amount * rate;
        return tax; // RETURN STATEMENT: sends value back to caller
    }
    
    public static void main(String[] args) {
        // CALLING the method:
        double myTax = calculateTax(1000.0, 0.15);
        //                              │        │
        //                              │        └─ ARGUMENT 2 (value for rate)
        //                              └────────── ARGUMENT 1 (value for amount)
        
        System.out.println("Tax: " + myTax); // Tax: 150.0
        
        // Can call it MULTIPLE TIMES with different values:
        System.out.println(calculateTax(5000.0, 0.15)); // 750.0
        System.out.println(calculateTax(250.0, 0.05));  // 12.5
        System.out.println(calculateTax(-100.0, 0.15)); // 0.0 (invalid)
    }
}
```

### Access Modifiers for Methods
```java
public class AccessModifiers {
    
    // public: anyone, anywhere can call this method
    // Most common for service methods, API endpoints
    public static void publicMethod() {
        System.out.println("Anyone can call me");
    }
    
    // private: ONLY this class can call this method
    // Use for internal helper methods
    // Hide implementation details
    private static void privateMethod() {
        System.out.println("Only this class can call me");
    }
    
    // protected: this class + subclasses + same package
    // Use when extending classes (inheritance)
    protected static void protectedMethod() {
        System.out.println("Subclasses can call me");
    }
    
    // default (no modifier): same package only
    static void packageMethod() {
        System.out.println("Same package only");
    }
    
    // The static modifier:
    // static method  → belongs to CLASS, call without object: ClassName.method()
    // instance method → belongs to OBJECT, need object: object.method()
    
    // static (class-level, no object needed):
    public static int add(int a, int b) {
        return a + b;
    }
    
    // instance (object-level, need an object):
    public int multiply(int a, int b) {
        return a * b; // could also use 'this' to reference the object
    }
    
    public static void main(String[] args) {
        // Static method: call directly on class
        int sum = AccessModifiers.add(3, 4); // or just: add(3, 4)
        System.out.println("Sum: " + sum);
        
        // Instance method: need an object first
        AccessModifiers obj = new AccessModifiers();
        int product = obj.multiply(3, 4);
        System.out.println("Product: " + product);
        
        // Private method: only callable inside this class
        privateMethod(); // valid here (we're inside the same class)
        
        // publicMethod(); // valid anywhere
    }
}
```

### Return Types
```java
public class ReturnTypes {
    
    // void: returns NOTHING
    public static void printDivider(int width) {
        for (int i = 0; i < width; i++) {
            System.out.print("─");
        }
        System.out.println();
    }
    // No return statement needed (or bare 'return;' to exit early)
    
    // Primitive return types:
    public static int    getAge()      { return 21; }
    public static double getPrice()    { return 9.99; }
    public static boolean isValid()    { return true; }
    public static char    getGrade()   { return 'A'; }
    public static long    getUserId()  { return 1234567890L; }
    
    // Reference return types:
    public static String  getName()    { return "Rahim"; }
    
    // Return an array:
    public static int[] getTopScores() {
        return new int[]{95, 88, 92, 79, 85};
    }
    
    // Return can appear MULTIPLE TIMES (early returns):
    public static String classifyAge(int age) {
        if (age < 0)  return "Invalid";   // early exit
        if (age < 13) return "Child";     // early exit
        if (age < 18) return "Teenager";  // early exit
        if (age < 60) return "Adult";     // early exit
        return "Senior";                  // default case
        // After return → execution stops. Nothing else runs.
    }
    
    // ⚠️ Every code path must return a value:
    // public static int badMethod(int x) {
    //     if (x > 0) {
    //         return 1;
    //     }
    //     // COMPILE ERROR: missing return statement!
    //     // What if x <= 0? Java demands all paths return.
    // }
    
    // ✅ Fixed:
    public static int goodMethod(int x) {
        if (x > 0) {
            return 1;
        }
        return -1; // covers the else case
    }
    
    // Returning null (reference types can return null):
    public static String findUser(int id) {
        if (id == 1) return "Rahim";
        return null; // user not found → callers must check for null!
    }
    
    public static void main(String[] args) {
        printDivider(30);
        
        System.out.println(classifyAge(5));  // Child
        System.out.println(classifyAge(16)); // Teenager
        System.out.println(classifyAge(25)); // Adult
        System.out.println(classifyAge(70)); // Senior
        
        int[] scores = getTopScores();
        System.out.println("Top score: " + scores[0]); // 95
        
        // Handle potential null return:
        String user = findUser(99);
        if (user != null) {
            System.out.println("Found: " + user);
        } else {
            System.out.println("User not found");
        }
    }
}
```

---

## 2. Parameters & Arguments - The Difference
```java
public class ParametersAndArguments {
    
    // PARAMETER: the variable in the method DEFINITION
    //            It's a placeholder — doesn't have a value yet
    //            Lives only while the method runs
    
    // ARGUMENT: the actual VALUE passed when CALLING the method
    //           It fills in the parameter placeholder
    
    //                  PARAMETERS (definitions)
    //                     ↓      ↓
    public static double calculateDiscount(double price, double discountRate) {
        // Inside here, price and discountRate are local variables
        // They have the values passed by the caller
        return price * discountRate;
    }
    
    public static void main(String[] args) {
        double originalPrice = 1000.0;
        double rate = 0.15;
        
        //                    ARGUMENTS (actual values)
        //                      ↓          ↓
        double discount = calculateDiscount(originalPrice, rate);
        // originalPrice (1000.0) → price
        // rate (0.15) → discountRate
        
        System.out.println("Discount: " + discount); // 150.0
        
        // Arguments can be literals, variables, or expressions:
        calculateDiscount(500.0, 0.10);              // literals
        calculateDiscount(originalPrice, rate);       // variables
        calculateDiscount(originalPrice * 2, 0.20);  // expression
    }
    
    // ─────────────────────────────────────────
    // PASS BY VALUE — CRITICAL CONCEPT
    // ─────────────────────────────────────────
    
    // Java is ALWAYS pass by value.
    // For primitives: the VALUE is copied.
    // For objects: the REFERENCE (address) is copied.
    
    static void tryToModifyPrimitive(int x) {
        x = 999; // modifies the LOCAL COPY only
        System.out.println("Inside method: x = " + x); // 999
    }
    
    static void tryToModifyArray(int[] arr) {
        arr[0] = 999; // modifies the ACTUAL ARRAY (via reference)
        System.out.println("Inside method: arr[0] = " + arr[0]); // 999
    }
    
    static void tryToReassignArray(int[] arr) {
        arr = new int[]{1, 2, 3}; // reassigns LOCAL reference only
        // The original array is unaffected
        System.out.println("Inside method: arr[0] = " + arr[0]); // 1
    }
    
    // Test pass by value:
    static void demonstratePassByValue() {
        // Primitives — changes DON'T affect original
        int num = 5;
        System.out.println("Before: " + num);   // 5
        tryToModifyPrimitive(num);
        System.out.println("After: " + num);    // 5 — unchanged!
        
        System.out.println();
        
        // Arrays — changes DO affect original (reference shared)
        int[] arr = {10, 20, 30};
        System.out.println("Before: " + arr[0]);  // 10
        tryToModifyArray(arr);
        System.out.println("After: " + arr[0]);   // 999 — changed!
        
        System.out.println();
        
        // Reassigning reference — does NOT affect original
        int[] arr2 = {10, 20, 30};
        System.out.println("Before: " + arr2[0]); // 10
        tryToReassignArray(arr2);
        System.out.println("After: " + arr2[0]);  // 10 — unchanged!
    }
}
```

### Multiple Parameters & Parameter Types
```java
public class ParameterTypes {
    
    // ─────────────────────────────────────────
    // MULTIPLE PARAMETERS
    // ─────────────────────────────────────────
    
    public static double calculateBill(
            String customerName,
            int quantity,
            double unitPrice,
            boolean isPremium,
            String couponCode) {
        
        double subtotal = quantity * unitPrice;
        
        // Premium discount
        if (isPremium) {
            subtotal *= 0.90; // 10% off
        }
        
        // Coupon discount
        if ("SAVE20".equals(couponCode)) {
            subtotal *= 0.80; // 20% off
        }
        
        System.out.printf("Bill for %s: ৳%.2f%n", customerName, subtotal);
        return subtotal;
    }
    
    // ─────────────────────────────────────────
    // VARARGS — Variable Number of Arguments
    // ─────────────────────────────────────────
    
    // ... (three dots) = varargs
    // Accept ANY number of arguments of that type
    // Java bundles them into an array internally
    
    public static double sum(double... numbers) {
        // 'numbers' is actually a double[] inside the method
        double total = 0;
        for (double n : numbers) {
            total += n;
        }
        return total;
    }
    
    public static String joinStrings(String separator, String... parts) {
        // Mix normal params with varargs (varargs MUST be last)
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < parts.length; i++) {
            sb.append(parts[i]);
            if (i < parts.length - 1) {
                sb.append(separator);
            }
        }
        return sb.toString();
    }
    
    // ─────────────────────────────────────────
    // ARRAY AS PARAMETER
    // ─────────────────────────────────────────
    
    public static double average(int[] numbers) {
        if (numbers == null || numbers.length == 0) {
            return 0;
        }
        int sum = 0;
        for (int n : numbers) {
            sum += n;
        }
        return (double) sum / numbers.length;
    }
    
    public static void main(String[] args) {
        // Calling with multiple parameters:
        calculateBill("Rahim", 3, 500.0, true, "SAVE20");
        
        // Varargs — pass any number of values:
        System.out.println(sum(1.0, 2.0, 3.0));           // 6.0
        System.out.println(sum(10.0, 20.0));               // 30.0
        System.out.println(sum(5.0, 5.0, 5.0, 5.0, 5.0)); // 25.0
        System.out.println(sum());                          // 0.0 (zero args)
        
        // Can also pass array to varargs:
        double[] prices = {100.0, 200.0, 300.0};
        System.out.println(sum(prices)); // 600.0
        
        // joinStrings:
        System.out.println(joinStrings(", ", "Java", "Spring", "PostgreSQL"));
        // Java, Spring, PostgreSQL
        
        System.out.println(joinStrings(" | ", "GET", "POST", "PUT", "DELETE"));
        // GET | POST | PUT | DELETE
        
        // Array parameter:
        int[] scores = {85, 92, 78, 95, 88};
        System.out.printf("Average: %.2f%n", average(scores)); // 87.60
    }
}
```

---

## 3. Method Overloading
```java
public class MethodOverloading {
    
    // OVERLOADING: Same method NAME, different PARAMETERS
    // Java determines which one to call based on argument types/count
    // This is COMPILE-TIME polymorphism
    
    // Three versions of 'calculateArea' — same name, different signatures:
    
    // Circle area
    public static double calculateArea(double radius) {
        return Math.PI * radius * radius;
    }
    
    // Rectangle area
    public static double calculateArea(double width, double height) {
        return width * height;
    }
    
    // Square area (int parameter — different type)
    public static double calculateArea(int side) {
        return (double) side * side;
    }
    
    // Triangle area
    public static double calculateArea(double base, double height, boolean isTriangle) {
        return 0.5 * base * height;
        // (boolean param distinguishes from rectangle overload)
    }
    
    // ─────────────────────────────────────────
    // REAL-WORLD OVERLOADING EXAMPLE
    // ─────────────────────────────────────────
    
    // Log method — same concept, different input types:
    public static void log(String message) {
        System.out.println("[INFO] " + message);
    }
    
    public static void log(String message, Exception e) {
        System.out.println("[ERROR] " + message + ": " + e.getMessage());
    }
    
    public static void log(String level, String message) {
        System.out.println("[" + level + "] " + message);
    }
    
    // Format price — different currencies/precision:
    public static String formatPrice(double amount) {
        return String.format("৳%.2f", amount);
    }
    
    public static String formatPrice(double amount, String currency) {
        return String.format("%s %.2f", currency, amount);
    }
    
    public static String formatPrice(long amountInPoisha) {
        return String.format("৳%.2f", amountInPoisha / 100.0);
    }
    
    // ─────────────────────────────────────────
    // OVERLOADING RULES
    // ─────────────────────────────────────────
    
    // ✅ Valid overloads (different parameter LIST):
    // void send(String message)
    // void send(String message, String recipient)    ← different count
    // void send(int code)                            ← different type
    // void send(String message, int priority)        ← different types
    
    // ❌ NOT a valid overload (same parameters, different return type):
    // int process(String data)
    // String process(String data)  ← COMPILE ERROR: same signature!
    // Return type is NOT part of the method signature for overloading.
    
    // ❌ NOT a valid overload (same parameters, different parameter names):
    // void create(String name)
    // void create(String username)  ← SAME signature! compile error
    // Parameter names are NOT part of the method signature.
    
    public static void main(String[] args) {
        // Java picks the right version automatically:
        System.out.println(calculateArea(5.0));          // circle: 78.54
        System.out.println(calculateArea(4.0, 6.0));     // rectangle: 24.0
        System.out.println(calculateArea(5));             // square: 25.0 (int)
        System.out.println(calculateArea(3.0, 4.0, true)); // triangle: 6.0
        
        log("Server started");                           // [INFO] Server started
        log("DB failed", new RuntimeException("Conn")); // [ERROR] DB failed: Conn
        log("DEBUG", "Processing request");              // [DEBUG] Processing request
        
        System.out.println(formatPrice(1500.0));          // ৳1500.00
        System.out.println(formatPrice(1500.0, "USD"));   // USD 1500.00
        System.out.println(formatPrice(150000L));          // ৳1500.00
    }
}
```

---
## 4. Scope - Where Variables Live and Die
```java
public class VariableScope {
    
    // ─────────────────────────────────────────
    // WHAT IS SCOPE?
    // ─────────────────────────────────────────
    // Scope = the region of code where a variable is ACCESSIBLE
    // Outside its scope → COMPILE ERROR (variable not found)
    
    // SCOPE LEVELS (from largest to smallest):
    // 1. Class scope (fields)
    // 2. Method scope (local variables, parameters)
    // 3. Block scope (inside {})
    
    // ─────────────────────────────────────────
    // CLASS SCOPE: Fields
    // ─────────────────────────────────────────
    
    // These are visible to ALL methods in this class:
    static int classVariable = 100;       // static field (class level)
    int instanceVariable = 200;           // instance field (object level)
    static final String CLASS_CONST = "CONSTANT"; // constant
    
    public static void methodA() {
        System.out.println(classVariable);   // ✅ accessible
        System.out.println(CLASS_CONST);     // ✅ accessible
        // System.out.println(instanceVariable); // ❌ static method can't access non-static field
    }
    
    public void instanceMethodA() {
        System.out.println(classVariable);   // ✅ accessible
        System.out.println(instanceVariable); // ✅ accessible
        System.out.println(CLASS_CONST);     // ✅ accessible
    }
    
    // ─────────────────────────────────────────
    // METHOD SCOPE: Local Variables & Parameters
    // ─────────────────────────────────────────
    
    public static void demonstrateMethodScope() {
        // localVar is created when method starts, destroyed when method ends
        int localVar = 50;
        System.out.println(localVar); // ✅ accessible
        
        // Parameters are also local variables:
    }
    
    public static void methodWithParam(int param) {
        // param exists here
        System.out.println(param); // ✅
    }
    // After this method ends:
    // localVar is GONE
    // param is GONE
    
    // Can two methods have a local variable with the same name? YES!
    public static void methodOne() {
        int value = 10; // completely separate from methodTwo's value
        System.out.println("methodOne value: " + value);
    }
    
    public static void methodTwo() {
        int value = 999; // completely separate from methodOne's value
        System.out.println("methodTwo value: " + value);
    }
    
    // ─────────────────────────────────────────
    // BLOCK SCOPE: Inside { }
    // ─────────────────────────────────────────
    
    public static void demonstrateBlockScope() {
        int outerVar = 10; // method scope
        
        if (outerVar > 5) {
            int innerVar = 20; // block scope (lives only inside this if)
            System.out.println(outerVar);  // ✅ outer visible inside block
            System.out.println(innerVar);  // ✅ visible inside its block
        }
        
        System.out.println(outerVar);      // ✅ outer still alive
        // System.out.println(innerVar);   // ❌ COMPILE ERROR: innerVar is gone!
        
        // for loop variable scope:
        for (int i = 0; i < 3; i++) {
            // i exists only inside the for loop
            System.out.println(i);
        }
        // System.out.println(i); // ❌ COMPILE ERROR: i is gone!
        
        // while loop:
        int count = 0;  // declared BEFORE loop — accessible after
        while (count < 3) {
            int temp = count * 2; // temp: block scope — only inside while
            System.out.println(temp);
            count++;
        }
        System.out.println("count after: " + count); // ✅ count still alive
        // System.out.println(temp); // ❌ COMPILE ERROR: temp is gone!
        
        // Nested blocks:
        {
            int blockVar = 42; // anonymous block scope
            System.out.println(blockVar); // ✅ inside the block
        }
        // System.out.println(blockVar); // ❌ COMPILE ERROR
    }
    
    // ─────────────────────────────────────────
    // VARIABLE SHADOWING (avoid this!)
    // ─────────────────────────────────────────
    
    static int shadowVar = 100; // class-level
    
    public static void shadowingDemo() {
        // Local variable with SAME NAME as class field:
        int shadowVar = 999; // shadows the class-level shadowVar
        System.out.println(shadowVar); // 999 — local takes precedence!
        System.out.println(VariableScope.shadowVar); // 100 — class field
        
        // Shadowing causes confusion — AVOID IT.
        // In instance methods, use 'this' to distinguish:
        // this.shadowVar = class field
        // shadowVar = local variable
    }
    
    // ─────────────────────────────────────────
    // SCOPE IN REAL BACKEND CODE
    // ─────────────────────────────────────────
    
    public static double processOrder(double price, int qty, String code) {
        // All parameters are local to this method
        
        double subtotal = price * qty;          // local
        boolean hasDiscount = code != null && !code.isEmpty(); // local
        
        if (hasDiscount) {
            double discount = subtotal * 0.10;  // scoped to if block
            subtotal -= discount;
            System.out.printf("Discount applied: ৳%.2f%n", discount);
            // discount is only needed here — block scope is appropriate
        }
        // discount is gone here — that's fine, we don't need it anymore
        
        double tax = subtotal * 0.15;           // local
        double total = subtotal + tax;           // local
        
        return total; // all locals destroyed after this return
    }
    
    public static void main(String[] args) {
        demonstrateBlockScope();
        shadowingDemo();
        
        System.out.printf("Order total: ৳%.2f%n",
                          processOrder(500.0, 3, "SAVE10"));
    }
}
```

---
## 5. Method Design — How Professionals Think
```java
public class MethodDesign {
    
    // ─────────────────────────────────────────
    // SINGLE RESPONSIBILITY PRINCIPLE
    // One method → one job. Always.
    // ─────────────────────────────────────────
    
    // ❌ BAD: One method doing too many things
    public static void badProcessUser(String name, String email,
                                       String password, int age) {
        // Validate name:
        if (name == null || name.isBlank()) {
            System.out.println("Invalid name");
            return;
        }
        // Validate email:
        if (!email.contains("@")) {
            System.out.println("Invalid email");
            return;
        }
        // Validate password:
        if (password.length() < 8) {
            System.out.println("Password too short");
            return;
        }
        // Hash password:
        String hashed = "hashed_" + password; // simplified
        // Save to database:
        System.out.println("Saving: " + name + ", " + email + ", " + hashed);
        // Send welcome email:
        System.out.println("Sending welcome email to: " + email);
        // Log the event:
        System.out.println("AUDIT: New user created: " + name);
    }
    
    // ✅ GOOD: Separate responsibilities
    
    private static boolean isValidName(String name) {
        return name != null && !name.isBlank() && name.length() >= 2;
    }
    
    private static boolean isValidEmail(String email) {
        return email != null
            && email.contains("@")
            && email.contains(".")
            && !email.startsWith("@");
    }
    
    private static boolean isValidPassword(String password) {
        if (password == null || password.length() < 8) return false;
        boolean hasUpper = !password.equals(password.toLowerCase());
        boolean hasDigit = password.matches(".*\\d.*");
        return hasUpper && hasDigit;
    }
    
    private static String hashPassword(String password) {
        // In real code: BCryptPasswordEncoder.encode(password)
        return "bcrypt_" + password.hashCode(); // simplified
    }
    
    private static void sendWelcomeEmail(String email, String name) {
        System.out.println("Email sent to " + email + ": Welcome, " + name + "!");
    }
    
    private static void auditLog(String event) {
        System.out.println("AUDIT [" + java.time.LocalDateTime.now() + "]: " + event);
    }
    
    // The orchestrator — calls other focused methods:
    public static boolean registerUser(String name, String email,
                                        String password, int age) {
        // Validate — each check is clear and testable:
        if (!isValidName(name)) {
            System.out.println("❌ Invalid name");
            return false;
        }
        if (!isValidEmail(email)) {
            System.out.println("❌ Invalid email");
            return false;
        }
        if (!isValidPassword(password)) {
            System.out.println("❌ Password must be 8+ chars with uppercase and digit");
            return false;
        }
        
        // Process:
        String hashedPassword = hashPassword(password);
        
        // Would save to DB here:
        System.out.printf("✅ User saved: name=%s, email=%s%n", name, email);
        
        // Side effects:
        sendWelcomeEmail(email, name);
        auditLog("User registered: " + email);
        
        return true;
    }
    
    // ─────────────────────────────────────────
    // NAMING CONVENTIONS
    // ─────────────────────────────────────────
    
    // Methods should be VERBS (they DO things):
    // ✅ calculateTotal(), validateEmail(), getUserById()
    // ✅ sendNotification(), processPayment(), createOrder()
    // ❌ total(), email(), user() (not verbs)
    
    // Boolean methods: use is/has/can/should:
    // ✅ isEmailVerified(), hasPermission(), canAccess()
    // ❌ emailVerified(), permission(), access()
    
    // Methods returning a value: the name should hint at what's returned:
    // ✅ getOrderById(), findUserByEmail(), calculateTax()
    // ❌ order(), user(), tax() (too vague)
    
    // ─────────────────────────────────────────
    // METHOD LENGTH GUIDELINE
    // ─────────────────────────────────────────
    
    // Rule of thumb: if a method doesn't fit on ONE screen → too long
    // Aim for: 5-20 lines for most methods
    // Maximum: 30-40 lines before you should refactor
    
    // If a method needs a comment to explain each "section" →
    // those sections should be separate methods with descriptive names
    
    // ─────────────────────────────────────────
    // DEFENSIVE PROGRAMMING IN METHODS
    // ─────────────────────────────────────────
    
    public static double divide(double dividend, double divisor) {
        // Validate inputs FIRST (guard clauses):
        if (divisor == 0) {
            throw new IllegalArgumentException(
                "Divisor cannot be zero");
        }
        return dividend / divisor;
    }
    
    public static String processName(String name) {
        // Null check first:
        if (name == null) {
            throw new IllegalArgumentException("Name cannot be null");
        }
        // Range check:
        if (name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (name.length() > 100) {
            throw new IllegalArgumentException("Name too long (max 100 chars)");
        }
        // Happy path:
        return name.trim().toLowerCase();
    }
    
    public static void main(String[] args) {
        // Test our well-designed methods:
        System.out.println("=== User Registration ===");
        registerUser("Rahim Ahmed", "rahim@example.com", "SecurePass1", 25);
        System.out.println();
        registerUser("", "invalid", "short", 25); // validation fails
        System.out.println();
        
        System.out.println("=== Defensive Methods ===");
        System.out.println(divide(10, 3));     // 3.333...
        
        try {
            divide(10, 0); // will throw
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        
        System.out.println(processName("  RAHIM  ")); // rahim (trimmed, lowercased)
    }
}
```

---
## 6. Recursion - Methods Calling Themselves
```java
public class Recursion {
    
    // RECURSION: A method that calls ITSELF.
    // Used when a problem can be broken into smaller versions
    // of the SAME problem.
    
    // EVERY recursive method MUST have:
    // 1. BASE CASE: condition where recursion STOPS (no more self-call)
    // 2. RECURSIVE CASE: calls itself with SMALLER/SIMPLER input
    
    // Without base case → infinite recursion → StackOverflowError!
    
    // ─────────────────────────────────────────
    // EXAMPLE 1: Factorial
    // 5! = 5 × 4 × 3 × 2 × 1 = 120
    // n! = n × (n-1)!
    // ─────────────────────────────────────────
    
    public static long factorial(int n) {
        // BASE CASE: 0! = 1, 1! = 1
        if (n <= 1) {
            return 1; // ← stops recursion
        }
        // RECURSIVE CASE: n! = n × (n-1)!
        return n * factorial(n - 1); // ← calls itself with smaller n
    }
    
    // HOW factorial(5) EXECUTES:
    // factorial(5) → 5 × factorial(4)
    //                    → 4 × factorial(3)
    //                           → 3 × factorial(2)
    //                                  → 2 × factorial(1)
    //                                         → return 1  [BASE CASE]
    //                                  → 2 × 1 = 2
    //                           → 3 × 2 = 6
    //                    → 4 × 6 = 24
    // factorial(5) → 5 × 24 = 120
    
    // ─────────────────────────────────────────
    // EXAMPLE 2: Fibonacci Sequence
    // 0, 1, 1, 2, 3, 5, 8, 13, 21, 34...
    // fib(n) = fib(n-1) + fib(n-2)
    // ─────────────────────────────────────────
    
    public static long fibonacci(int n) {
        // BASE CASES:
        if (n == 0) return 0;
        if (n == 1) return 1;
        // RECURSIVE CASE:
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
    // ⚠️ WARNING: This is O(2^n) — VERY SLOW for large n!
    // fibonacci(50) would take minutes.
    // Real code uses dynamic programming or iteration instead.
    // This is for understanding recursion, not production use.
    
    // ─────────────────────────────────────────
    // EXAMPLE 3: Binary Search (recursive)
    // Search sorted array for target value
    // ─────────────────────────────────────────
    
    public static int binarySearch(int[] arr, int target, int left, int right) {
        // BASE CASE: search space exhausted
        if (left > right) {
            return -1; // not found
        }
        
        int mid = left + (right - left) / 2; // avoid integer overflow
        
        if (arr[mid] == target) {
            return mid; // BASE CASE: found it!
        } else if (arr[mid] < target) {
            return binarySearch(arr, target, mid + 1, right); // search right half
        } else {
            return binarySearch(arr, target, left, mid - 1);  // search left half
        }
    }
    
    // ─────────────────────────────────────────
    // EXAMPLE 4: Sum of digits (elegant recursion)
    // ─────────────────────────────────────────
    
    public static int digitSum(int n) {
        n = Math.abs(n); // handle negatives
        if (n < 10) return n; // BASE CASE: single digit
        return n % 10 + digitSum(n / 10); // last digit + sum of rest
    }
    
    // ─────────────────────────────────────────
    // EXAMPLE 5: Flatten nested structure
    // (Real use in parsing JSON-like structures)
    // ─────────────────────────────────────────
    
    // Count all files in a directory (simulated with depth)
    public static int countItems(int depth, int itemsPerLevel) {
        if (depth == 0) return itemsPerLevel; // BASE CASE: leaf level
        // Items at this level + items in each sub-directory
        return itemsPerLevel + itemsPerLevel * countItems(depth - 1, itemsPerLevel);
    }
    
    // ─────────────────────────────────────────
    // RECURSION vs ITERATION
    // ─────────────────────────────────────────
    
    // Same factorial — iterative version:
    public static long factorialIterative(int n) {
        long result = 1;
        for (int i = 2; i <= n; i++) {
            result *= i;
        }
        return result;
    }
    
    // When to use recursion:
    // ✅ Tree traversal (category trees, file systems, org charts)
    // ✅ Graph traversal (BFS, DFS)
    // ✅ Divide and conquer algorithms (merge sort, quick sort)
    // ✅ Parsing nested structures (JSON, XML, recursive data)
    // ✅ When recursive solution is significantly cleaner
    
    // When to use iteration:
    // ✅ Simple loops (array processing, counting)
    // ✅ When stack depth could be large (stack overflow risk)
    // ✅ Performance critical code (iteration is usually faster)
    // ✅ When you need to maintain state across iterations
    
    public static void main(String[] args) {
        // Factorial:
        for (int i = 0; i <= 10; i++) {
            System.out.printf("%2d! = %d%n", i, factorial(i));
        }
        
        // Fibonacci:
        System.out.print("\nFibonacci: ");
        for (int i = 0; i < 12; i++) {
            System.out.print(fibonacci(i) + " ");
        }
        System.out.println();
        
        // Binary search:
        int[] sorted = {2, 5, 8, 12, 16, 23, 38, 56, 72, 91};
        int idx = binarySearch(sorted, 23, 0, sorted.length - 1);
        System.out.println("\nBinary search for 23: index " + idx); // 5
        System.out.println("Binary search for 99: index " +
                           binarySearch(sorted, 99, 0, sorted.length - 1)); // -1
        
        // Digit sum:
        System.out.println("\nDigit sum of 123456: " + digitSum(123456)); // 21
        System.out.println("Digit sum of 9999: " + digitSum(9999));       // 36
        
        // Compare recursive vs iterative:
        System.out.println("\n15! recursive:  " + factorial(15));
        System.out.println("15! iterative:  " + factorialIterative(15));
    }
}
```

---
## Build This - Complete Methods Practice
```java
// File: BankingSystem.java
// A simplified banking system using well-designed methods

public class BankingSystem {
    
    // ─────────────────────────────────────────
    // CONSTANTS
    // ─────────────────────────────────────────
    static final double MIN_BALANCE          = 500.0;
    static final double MAX_DAILY_WITHDRAWAL = 50_000.0;
    static final double SAVINGS_INTEREST     = 0.045;
    static final double TRANSFER_FEE_RATE   = 0.01;
    static final int    MAX_PIN_ATTEMPTS     = 3;
    
    // ─────────────────────────────────────────
    // VALIDATION METHODS (private helpers)
    // ─────────────────────────────────────────
    
    private static boolean isValidAmount(double amount) {
        return amount > 0 && amount <= MAX_DAILY_WITHDRAWAL;
    }
    
    private static boolean isValidPin(String pin) {
        return pin != null
            && pin.length() == 4
            && pin.matches("\\d{4}"); // exactly 4 digits
    }
    
    private static boolean isValidAccountNumber(String accNum) {
        return accNum != null
            && accNum.length() == 10
            && accNum.matches("\\d{10}");
    }
    
    private static boolean hasSufficientBalance(double balance, double amount) {
        return (balance - amount) >= MIN_BALANCE;
    }
    
    // ─────────────────────────────────────────
    // CALCULATION METHODS
    // ─────────────────────────────────────────
    
    private static double calculateTransferFee(double amount) {
        return amount * TRANSFER_FEE_RATE;
    }
    
    private static double calculateMonthlyInterest(double balance) {
        double monthlyRate = SAVINGS_INTEREST / 12;
        return Math.round(balance * monthlyRate * 100.0) / 100.0;
    }
    
    private static double calculateCompoundInterest(double principal,
                                                     double annualRate,
                                                     int years) {
        // A = P(1 + r/n)^(nt), where n=1 (annual compounding)
        return principal * Math.pow(1 + annualRate, years);
    }
    
    // ─────────────────────────────────────────
    // FORMATTING METHODS
    // ─────────────────────────────────────────
    
    private static String formatAmount(double amount) {
        return String.format("৳%,.2f", amount);
    }
    
    private static String maskAccountNumber(String accNum) {
        if (!isValidAccountNumber(accNum)) return "Invalid";
        return "******" + accNum.substring(6); // show last 4 digits
    }
    
    private static void printSectionHeader(String title) {
        System.out.println("\n" + "═".repeat(50));
        System.out.println("  " + title);
        System.out.println("═".repeat(50));
    }
    
    private static void printTransaction(String type, double amount,
                                          double balance, boolean success) {
        String status = success ? "✅ SUCCESS" : "❌ FAILED";
        System.out.printf("%-12s │ %12s │ Balance: %12s │ %s%n",
                          type, formatAmount(amount),
                          formatAmount(balance), status);
    }
    
    // ─────────────────────────────────────────
    // OPERATION METHODS
    // ─────────────────────────────────────────
    
    public static double deposit(double balance, double amount) {
        if (!isValidAmount(amount)) {
            printTransaction("DEPOSIT", amount, balance, false);
            System.out.println("  Reason: Invalid amount");
            return balance; // unchanged
        }
        double newBalance = balance + amount;
        printTransaction("DEPOSIT", amount, newBalance, true);
        return newBalance;
    }
    
    public static double withdraw(double balance, double amount) {
        if (!isValidAmount(amount)) {
            printTransaction("WITHDRAW", amount, balance, false);
            System.out.println("  Reason: Amount out of valid range");
            return balance;
        }
        if (!hasSufficientBalance(balance, amount)) {
            printTransaction("WITHDRAW", amount, balance, false);
            System.out.printf("  Reason: Would breach minimum balance (%s)%n",
                              formatAmount(MIN_BALANCE));
            return balance;
        }
        double newBalance = balance - amount;
        printTransaction("WITHDRAW", amount, newBalance, true);
        return newBalance;
    }
    
    public static double[] transfer(double senderBalance,
                                     double receiverBalance,
                                     double amount,
                                     String toAccount) {
        double fee = calculateTransferFee(amount);
        double totalDeduction = amount + fee;
        
        if (!isValidAmount(amount)) {
            System.out.println("❌ TRANSFER FAILED: Invalid amount");
            return new double[]{senderBalance, receiverBalance};
        }
        
        if (!isValidAccountNumber(toAccount)) {
            System.out.println("❌ TRANSFER FAILED: Invalid account number");
            return new double[]{senderBalance, receiverBalance};
        }
        
        if (!hasSufficientBalance(senderBalance, totalDeduction)) {
            System.out.println("❌ TRANSFER FAILED: Insufficient balance (including fee)");
            return new double[]{senderBalance, receiverBalance};
        }
        
        double newSenderBalance = senderBalance - totalDeduction;
        double newReceiverBalance = receiverBalance + amount;
        
        System.out.println("✅ TRANSFER SUCCESS:");
        System.out.printf("   Amount     : %s%n", formatAmount(amount));
        System.out.printf("   Fee (1%%)   : %s%n", formatAmount(fee));
        System.out.printf("   Total      : %s%n", formatAmount(totalDeduction));
        System.out.printf("   To Account : %s%n", maskAccountNumber(toAccount));
        System.out.printf("   New Balance: %s%n", formatAmount(newSenderBalance));
        
        return new double[]{newSenderBalance, newReceiverBalance};
    }
    
    // ─────────────────────────────────────────
    // INTEREST PROJECTION (uses recursion)
    // ─────────────────────────────────────────
    
    private static void printInterestProjection(double balance,
                                                 double rate,
                                                 int years) {
        System.out.println("\nInterest Projection:");
        System.out.printf("%-6s %-15s %-15s%n", "Year", "Balance", "Interest");
        System.out.println("─".repeat(38));
        
        double currentBalance = balance;
        for (int year = 1; year <= years; year++) {
            double projected = calculateCompoundInterest(balance, rate, year);
            double interest = projected - currentBalance;
            System.out.printf("%-6d %-15s %-15s%n",
                              year,
                              formatAmount(projected),
                              "+" + formatAmount(interest));
            currentBalance = projected;
        }
    }
    
    // ─────────────────────────────────────────
    // MAIN — DEMONSTRATE THE SYSTEM
    // ─────────────────────────────────────────
    
    public static void main(String[] args) {
        double aliceBalance = 25_000.0;
        double bobBalance   = 10_000.0;
        
        String aliceAccount = "1234567890";
        String bobAccount   = "0987654321";
        
        printSectionHeader("ACCOUNT OVERVIEW");
        System.out.printf("Alice: %s → Balance: %s%n",
                          maskAccountNumber(aliceAccount),
                          formatAmount(aliceBalance));
        System.out.printf("Bob  : %s → Balance: %s%n",
                          maskAccountNumber(bobAccount),
                          formatAmount(bobBalance));
        
        // ─────────────────────────────────────────
        // DEPOSITS
        // ─────────────────────────────────────────
        printSectionHeader("DEPOSITS");
        aliceBalance = deposit(aliceBalance, 5_000.0);
        aliceBalance = deposit(aliceBalance, -500.0);  // invalid
        aliceBalance = deposit(aliceBalance, 0);        // invalid
        
        // ─────────────────────────────────────────
        // WITHDRAWALS
        // ─────────────────────────────────────────
        printSectionHeader("WITHDRAWALS");
        aliceBalance = withdraw(aliceBalance, 3_000.0);
        aliceBalance = withdraw(aliceBalance, 60_000.0); // exceeds limit
        aliceBalance = withdraw(aliceBalance, 26_000.0); // would breach minimum
        
        // ─────────────────────────────────────────
        // TRANSFERS
        // ─────────────────────────────────────────
        printSectionHeader("TRANSFERS");
        double[] result = transfer(aliceBalance, bobBalance,
                                   10_000.0, bobAccount);
        aliceBalance = result[0];
        bobBalance   = result[1];
        
        System.out.printf("%nAlice after transfer: %s%n", formatAmount(aliceBalance));
        System.out.printf("Bob after transfer  : %s%n", formatAmount(bobBalance));
        
        // ─────────────────────────────────────────
        // INTEREST
        // ─────────────────────────────────────────
        printSectionHeader("SAVINGS INTEREST PROJECTION");
        System.out.printf("Current Balance: %s%n", formatAmount(aliceBalance));
        System.out.printf("Annual Rate    : %.1f%%%n", SAVINGS_INTEREST * 100);
        printInterestProjection(aliceBalance, SAVINGS_INTEREST, 5);
        
        // Monthly interest:
        double monthlyInterest = calculateMonthlyInterest(aliceBalance);
        System.out.printf("%nThis month's interest: %s%n",
                          formatAmount(monthlyInterest));
    }
}
```

---
## Exercises
```text
EXERCISE 1: Math Utilities
  Create MathUtils.java
  Write these static methods (no importing Math class):
  - int abs(int n)
  - int max(int a, int b, int c)  ← three values
  - int min(int a, int b, int c)  ← three values  
  - boolean isPrime(int n)
  - int[] getPrimes(int limit)    ← all primes up to limit
  - double power(double base, int exp)  ← without Math.pow
  - int gcd(int a, int b)         ← greatest common divisor (use recursion)
  - int lcm(int a, int b)         ← least common multiple
  Test each method with at least 3 test cases.
  
EXERCISE 2: String Utilities
  Create StringUtils.java
  Write these static methods:
  - boolean isPalindrome(String s)     ← "racecar" = true
  - String reverse(String s)
  - int countOccurrences(String s, char c)
  - String capitalize(String s)        ← first letter of each word
  - boolean isAnagram(String a, String b)  ← "listen" & "silent" = true
  - String truncate(String s, int max) ← cut to max chars, add "..."
  - String repeat(String s, int times) ← without built-in repeat()
  Test each thoroughly.
  
EXERCISE 3: Overloading Practice
  Create Calculator.java
  Overload an 'add' method to handle:
  - add(int a, int b)
  - add(int a, int b, int c)
  - add(double a, double b)
  - add(long a, long b)
  - add(int... numbers)           ← varargs, any count
  - add(String a, String b)       ← concatenation (returns String)
  Print results and show which overload was called.
  
EXERCISE 4: Scope Investigator
  Create ScopeDemo.java
  Write a program that demonstrates:
  - A variable in outer scope visible in inner block
  - A variable in inner block NOT visible outside
  - Two methods with same local variable name (no conflict)
  - Variable shadowing (field vs local)
  - for loop variable scope
  - Method parameter scope
  Add comments explaining EXACTLY what's in scope at each line.
  
EXERCISE 5: Recursive Problems
  Create RecursiveProblems.java
  Implement recursively:
  - int sum(int n)           ← 1+2+3+...+n
  - boolean isPalindrome(String s)  ← using recursion
  - int countDigits(int n)   ← number of digits in n
  - void printReverse(int n) ← print n to 1
  - String toBinary(int n)   ← convert to binary string
  For each: show the call stack trace in comments.
  Compare each with an iterative version.
  Push to GitHub: "feat: add math and string utility methods"
```

---
## Common Mistakes
```text
MISTAKE 1: Method does too many things
  public static void processOrder() {
      // validates, calculates, saves to DB,
      // sends email, logs audit, updates inventory...
  }
  Fix: one method = one responsibility.
  Split into validateOrder(), calculateTotal(), saveOrder(), etc.

MISTAKE 2: Not returning from all code paths
  public static int getDiscount(String code) {
      if (code.equals("SAVE10")) return 10;
      if (code.equals("SAVE20")) return 20;
      // COMPILE ERROR: no return for other codes!
  }
  Fix: add default return at end, or return 0 as default.

MISTAKE 3: Modifying and returning (confusing callers)
  static double[] process(double[] arr) {
      arr[0] = 999;    // modifies original
      return arr;      // AND returns it
  }
  // Caller thinks they get a new array, but original is modified!
  Fix: either modify in place OR return a new array, not both.

MISTAKE 4: Forgetting base case in recursion
  static int factorial(int n) {
      return n * factorial(n - 1); // no base case! StackOverflowError!
  }
  Fix: if (n <= 1) return 1;

MISTAKE 5: Variable used before initialization (in method)
  public static void method() {
      int x;
      System.out.println(x); // COMPILE ERROR: x not initialized
  }
  Fix: int x = 0; (local variables MUST be initialized before use)

MISTAKE 6: Shadowing fields with local variables
  static int value = 100;
  public static void method() {
      int value = 999; // shadows field — confusing
      System.out.println(value); // prints 999, not field!
  }
  Fix: rename local variable to avoid shadowing.

MISTAKE 7: Treating pass-by-value as pass-by-reference
  static void addOne(int n) { n++; } // changes LOCAL copy only
  int x = 5;
  addOne(x);
  System.out.println(x); // still 5! addOne changed nothing.
  Fix: return the new value: static int addOne(int n) { return n + 1; }

MISTAKE 8: Recursive call without reducing toward base case
  static void countdown(int n) {
      System.out.println(n);
      countdown(n); // calls with SAME n! infinite recursion!
  }
  Fix: countdown(n - 1); // reduce n each time

MISTAKE 9: Varargs not last parameter
  // COMPILE ERROR:
  public static void method(String... items, int count) { }
  Fix: varargs MUST be the LAST parameter:
  public static void method(int count, String... items) { }

MISTAKE 10: Returning null without documentation
  public static String findUser(int id) {
      if (id == 1) return "Rahim";
      return null; // caller might not check!
  }
  Fix: document that null is a valid return value.
  Better: return Optional<String> (Level 2.11)
```

---
## Interview Questions

**Q: What is the difference between a parameter and an argument?**
A: A parameter is the variable declared in the method DEFINITION — it's a placeholder that exists when the method runs. An argument is the actual VALUE passed when CALLING the method. In: `public static void greet(String name) { }` — name is a parameter. In: `greet("Rahim")` — "Rahim" is the argument. Parameters define what the method needs. Arguments provide it.

**Q: Is Java pass-by-value or pass-by-reference?**
A: Java is strictly pass-by-value. For primitives, the actual value is copied — changes inside the method don't affect the original. For objects and arrays, the REFERENCE (memory address) is copied — so the method can modify the object's contents (because it has the same address), but reassigning the reference inside the method doesn't affect the original variable outside. This is a common interview trap — Java is NOT pass-by-reference even though object modifications inside methods DO affect the original object's state.

**Q: What is method overloading?**
A: Overloading means having multiple methods with the SAME name but DIFFERENT parameter lists (different number of parameters, different parameter types, or different order). The compiler determines which method to call based on the arguments provided. Return type alone CANNOT distinguish overloaded methods — parameter list must differ. It's compile-time polymorphism. Example: `calculateArea(double radius)` for circle and `calculateArea(double width, double height)` for rectangle.

**Q: What is the scope of a variable in Java?**
A: Scope is the region of code where a variable is accessible. Class-level fields: accessible throughout the class. Method parameters and local variables: accessible only within that method. Block scope (inside {}): accessible only within that block (if, for, while bodies). A variable ceases to exist when execution leaves its scope. Two methods can have local variables with the same name without conflict — they are completely separate.

**Q: What is recursion and what are its requirements?**
A: Recursion is when a method calls itself to solve a problem by breaking it into smaller instances of the same problem. Requirements: 1) A BASE CASE that stops the recursion (returns without calling itself), and 2) A RECURSIVE CASE that calls the method with a SMALLER or SIMPLER input, progressing toward the base case. Without a base case, recursion is infinite and causes StackOverflowError. Recursion is natural for tree traversal, divide-and-conquer algorithms, and parsing nested structures.

**Q: What is the difference between static and instance methods?**
A: A static method belongs to the CLASS and can be called without creating an object: `ClassName.method()`. It cannot access instance fields (non-static fields) because no object context exists. An instance method belongs to an OBJECT and requires an instance to call: `object.method()`. It can access both static and instance fields. Use static for utility methods that don't depend on object state (`Math.abs()`, `String.valueOf()`). Use instance methods when behavior depends on the object's state.

---
## Key Takeaways

```text
1. A method = named block of code that does ONE thing.
   Name it with a verb. Keep it short (5-20 lines ideal).
   One method = one responsibility. Always.

2. METHOD SIGNATURE:
   accessModifier returnType methodName(parameters)
   All parts matter. All have rules.

3. return TYPE:
   void     → returns nothing
   anything → must have matching return statement
   All code paths MUST return the correct type.
   Early returns are fine and often cleaner.

4. PARAMETERS vs ARGUMENTS:
   Parameters = placeholders in definition
   Arguments = actual values when calling
   MUST match in count and type (widening is automatic)

5. Java is PASS BY VALUE always:
   Primitives: copy of value → original unchanged
   Objects/arrays: copy of REFERENCE → can modify contents
   Reassigning a reference inside method → doesn't affect caller

6. METHOD OVERLOADING:
   Same name, different parameter LIST.
   Return type does NOT distinguish overloads.
   Compile-time decision — compiler picks the right one.

7. SCOPE rules:
   Variable lives and dies within its { } block.
   Local variables MUST be initialized before use (unlike fields).
   Parameters are local to the method.
   Avoid shadowing (same name for field and local variable).

8. RECURSION needs:
   BASE CASE (stops recursion — required!)
   RECURSIVE CASE (calls itself with smaller input)
   No base case = StackOverflowError

9. Design principles:
   Private for internal helpers (hide implementation)
   Guard clauses over deep nesting
   Descriptive names: calculateTax() not calc()
   Validate inputs first (defensive programming)

10. Every Spring Boot endpoint, service method, repository
    method = a method. You'll write thousands.
    Good method design = good Spring Boot code.
    Start building the habit now.
```

---