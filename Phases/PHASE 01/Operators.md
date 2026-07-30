Phase:** Level 1 - Java Fundamentals
**Date Studied:** 

---
## 🤔What Problem Does This Solve?
You have variables. You have data stored in them.
But data sitting in boxes doing nothing is useless.
```text
You need to:
  CALCULATE things   → what's the total price after discount?
  COMPARE things     → is this user older than 18?
  COMBINE conditions → is the user logged in AND has permission?
  MAKE DECISIONS     → if balance > 0 then allow withdrawal
  MODIFY values      → add interest to balance, increment counter

All of these require OPERATORS.

Operators are the verbs of programming.
Variables are nouns (they hold things).
Operators are verbs (they DO things with those nouns).

Without operators:
  You can store a price. You can store a discount.
  But you cannot calculate the final price.
  You have data but no computation.

With operators:
  finalPrice = originalPrice - (originalPrice * discountRate)
  This single expression calculates the answer.
```

Operators are used in literally EVERY line of real code.
This is not optional knowledge — this is foundational.

---
## The Full Operator Map

```text
JAVA OPERATORS (all categories):

┌─────────────────────────────────────────────────────────┐
│  ARITHMETIC    + - * / % ++ --                          │
│  ASSIGNMENT    = += -= *= /= %= &= |= ^= <<= >>= >>>=  │
│  COMPARISON    == != > < >= <=                          │
│  LOGICAL       && || ! & | ^                            │
│  BITWISE       & | ^ ~ << >> >>>                        │
│  TERNARY       condition ? valueIfTrue : valueIfFalse   │
│  INSTANCEOF    instanceof (type check)                  │
│  STRING        + (concatenation)                        │
└─────────────────────────────────────────────────────────┘

We'll cover all of these in order.
Bitwise is last (less commonly used in backend web dev).
```

---

## 1. Arithmetic Operators

```java
public class ArithmeticOperators {
    public static void main(String[] args) {
        
        int a = 17;
        int b = 5;
        
        // ─────────────────────────────────────────
        // BASIC ARITHMETIC
        // ─────────────────────────────────────────
        
        int sum         = a + b;   // 22   addition
        int difference  = a - b;   // 12   subtraction
        int product     = a * b;   // 85   multiplication
        int quotient    = a / b;   // 3    integer division (truncates!)
        int remainder   = a % b;   // 2    modulo (remainder after division)
        
        System.out.println("a + b = " + sum);         // 22
        System.out.println("a - b = " + difference);  // 12
        System.out.println("a * b = " + product);     // 85
        System.out.println("a / b = " + quotient);    // 3
        System.out.println("a % b = " + remainder);   // 2
        
        // ─────────────────────────────────────────
        // INTEGER DIVISION — THE TRAP
        // ─────────────────────────────────────────
        
        // Integer / Integer = Integer (decimal part DISCARDED, not rounded)
        System.out.println(17 / 5);    // 3   (not 3.4)
        System.out.println(1 / 2);     // 0   (not 0.5) ← common mistake!
        System.out.println(7 / 2);     // 3   (not 3.5)
        System.out.println(-7 / 2);    // -3  (truncates toward zero)
        
        // Fix: make at least ONE operand a double/float
        System.out.println(17.0 / 5);      // 3.4
        System.out.println(17 / 5.0);      // 3.4
        System.out.println((double) 17 / 5); // 3.4 (cast one to double)
        
        // Real-world mistake this causes:
        int totalScore = 7;
        int totalStudents = 2;
        double average = totalScore / totalStudents; // 3.0 ← WRONG!
        double correctAverage = (double) totalScore / totalStudents; // 3.5 ✓
        System.out.println("Wrong average: " + average);    // 3.0
        System.out.println("Correct average: " + correctAverage); // 3.5
        
        // ─────────────────────────────────────────
        // MODULO (%) — REMAINDER OPERATOR
        // ─────────────────────────────────────────
        
        // Returns the REMAINDER after division
        // 17 / 5 = 3 remainder 2 → 17 % 5 = 2
        
        System.out.println(10 % 3);  // 1  (10 = 3×3 + 1)
        System.out.println(10 % 2);  // 0  (10 = 5×2 + 0 → even number!)
        System.out.println(10 % 5);  // 0  (exactly divisible)
        System.out.println(3 % 10);  // 3  (3 < 10, so 0 times with 3 left)
        System.out.println(7 % 7);   // 0  (always 0 when n % n)
        System.out.println(-7 % 3);  // -1 (sign follows the dividend in Java)
        
        // MODULO USE CASES — this operator is incredibly useful:
        
        // 1. Check even or odd
        int num = 17;
        if (num % 2 == 0) {
            System.out.println(num + " is even");
        } else {
            System.out.println(num + " is odd"); // prints: 17 is odd
        }
        
        // 2. Check divisibility
        int n = 100;
        System.out.println(n + " divisible by 4: " + (n % 4 == 0)); // true
        System.out.println(n + " divisible by 7: " + (n % 7 == 0)); // false
        
        // 3. Wrapping around (circular index)
        int arraySize = 5;
        for (int i = 0; i < 12; i++) {
            int index = i % arraySize; // wraps: 0,1,2,3,4,0,1,2,3,4,0,1
            System.out.print(index + " ");
        }
        System.out.println();
        // Use: round-robin load balancing, rotating arrays, clock arithmetic
        
        // 4. Get last N digits
        int bigNumber = 123456789;
        int lastThreeDigits = bigNumber % 1000; // 789
        System.out.println("Last 3 digits: " + lastThreeDigits);
        
        // 5. Converting seconds to hours:minutes:seconds
        int totalSeconds = 3661;
        int hours   = totalSeconds / 3600;
        int minutes = (totalSeconds % 3600) / 60;
        int seconds = totalSeconds % 60;
        System.out.printf("Time: %02d:%02d:%02d%n", hours, minutes, seconds);
        // Output: 01:01:01
        
        // ─────────────────────────────────────────
        // ARITHMETIC WITH DIFFERENT TYPES
        // ─────────────────────────────────────────
        
        // When mixing types, result is the WIDER type
        int intVal = 5;
        long longVal = 10L;
        long result1 = intVal + longVal;  // int promoted to long → long result
        
        int intVal2 = 5;
        double doubleVal = 2.5;
        double result2 = intVal2 + doubleVal; // int promoted to double → double
        
        System.out.println(result1); // 15
        System.out.println(result2); // 7.5
        
        // byte and short in arithmetic get promoted to int
        byte byte1 = 10;
        byte byte2 = 20;
        // byte result3 = byte1 + byte2; // COMPILE ERROR: result is int!
        int result3 = byte1 + byte2;     // must store in int or cast
        byte result4 = (byte)(byte1 + byte2); // explicit cast back to byte
        System.out.println(result3); // 30
        System.out.println(result4); // 30
    }
}
```

---

## 2. Increment & Decrement Operators

```java
public class IncrementDecrement {
    public static void main(String[] args) {
        
        // ++ adds 1 to a variable
        // -- subtracts 1 from a variable
        
        int x = 5;
        
        // PRE-increment: ++x
        // INCREMENT FIRST, then use the value
        int a = ++x;           // x becomes 6, then a = 6
        System.out.println("x = " + x); // 6
        System.out.println("a = " + a); // 6
        
        // PRE-decrement: --x
        // DECREMENT FIRST, then use the value
        int b = --x;           // x becomes 5, then b = 5
        System.out.println("x = " + x); // 5
        System.out.println("b = " + b); // 5
        
        // POST-increment: x++
        // USE the value first, THEN increment
        int c = x++;           // c = 5 (old value), then x becomes 6
        System.out.println("x = " + x); // 6
        System.out.println("c = " + c); // 5
        
        // POST-decrement: x--
        // USE the value first, THEN decrement
        int d = x--;           // d = 6 (old value), then x becomes 5
        System.out.println("x = " + x); // 5
        System.out.println("d = " + d); // 6
        
        // ─────────────────────────────────────────
        // THE PRE vs POST DIFFERENCE — Visual
        // ─────────────────────────────────────────
        
        int counter = 10;
        
        // These are EQUIVALENT (in isolation — not in expression):
        counter++;      // same as: counter = counter + 1;
        counter--;      // same as: counter = counter - 1;
        ++counter;      // same as: counter = counter + 1;
        --counter;      // same as: counter = counter - 1;
        
        // Difference ONLY matters inside expressions:
        int original = 5;
        int postResult = original++; // postResult = 5, original = 6
        int preResult  = ++original; // original = 7, preResult = 7
        
        System.out.println("postResult: " + postResult); // 5
        System.out.println("preResult: " + preResult);   // 7
        System.out.println("original: " + original);     // 7
        
        // ─────────────────────────────────────────
        // WHERE THEY'RE MOST COMMONLY USED
        // ─────────────────────────────────────────
        
        // In for loops (extremely common):
        for (int i = 0; i < 5; i++) {  // i++ is most common
            System.out.print(i + " "); // 0 1 2 3 4
        }
        System.out.println();
        
        // Counting something:
        int errorCount = 0;
        boolean error1 = true, error2 = false, error3 = true;
        if (error1) errorCount++;
        if (error2) errorCount++;
        if (error3) errorCount++;
        System.out.println("Errors: " + errorCount); // 2
        
        // ─────────────────────────────────────────
        // PROFESSIONAL ADVICE
        // ─────────────────────────────────────────
        
        // Don't be "clever" with pre/post in complex expressions.
        // This is unreadable:
        int ugly = 5;
        int uglyResult = ++ugly + ugly++; // What is this? Don't do this.
        
        // This is readable:
        int clean = 5;
        clean++; // increment first
        int cleanResult = clean + clean; // then use
        
        // Rule: use i++ or i-- standalone (on its own line).
        // Only use in expressions when it's obvious and necessary.
    }
}
```

---

## 3. Assignment Operators

```java
public class AssignmentOperators {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // BASIC ASSIGNMENT
        // ─────────────────────────────────────────
        
        int x = 10; // assign 10 to x
        
        // ─────────────────────────────────────────
        // COMPOUND ASSIGNMENT OPERATORS
        // shorthand: variable op= value
        // means: variable = variable op value
        // ─────────────────────────────────────────
        
        int n = 20;
        
        n += 5;   // n = n + 5   → n = 25
        System.out.println("n += 5  → " + n); // 25
        
        n -= 3;   // n = n - 3   → n = 22
        System.out.println("n -= 3  → " + n); // 22
        
        n *= 2;   // n = n * 2   → n = 44
        System.out.println("n *= 2  → " + n); // 44
        
        n /= 4;   // n = n / 4   → n = 11
        System.out.println("n /= 4  → " + n); // 11
        
        n %= 3;   // n = n % 3   → n = 2 (11 % 3 = 2)
        System.out.println("n %= 3  → " + n); // 2
        
        // ─────────────────────────────────────────
        // HIDDEN FEATURE: compound assignment includes implicit cast
        // ─────────────────────────────────────────
        
        byte b = 10;
        // b = b + 5; // COMPILE ERROR: b + 5 is int, can't assign to byte
        b += 5;       // ✓ WORKS: compound assignment auto-casts back to byte
        // This is: b = (byte)(b + 5) internally
        System.out.println("byte b += 5 → " + b); // 15
        
        // Same for short:
        short s = 100;
        // s = s + 50; // COMPILE ERROR
        s += 50;        // ✓ Works: auto-casts back to short
        System.out.println("short s += 50 → " + s); // 150
        
        // ─────────────────────────────────────────
        // REAL-WORLD USAGE PATTERNS
        // ─────────────────────────────────────────
        
        // Accumulating a total:
        double totalPrice = 0.0;
        totalPrice += 9.99;   // first item
        totalPrice += 4.50;   // second item
        totalPrice += 12.00;  // third item
        System.out.printf("Total: %.2f%n", totalPrice); // 26.49
        
        // Applying discount:
        double price = 1000.0;
        double discountRate = 0.15; // 15%
        price -= price * discountRate; // price = price - price * 0.15
        System.out.printf("After 15%% discount: %.2f%n", price); // 850.00
        
        // Applying interest:
        double balance = 10000.0;
        double monthlyInterestRate = 0.005; // 0.5% per month
        balance *= (1 + monthlyInterestRate); // balance *= 1.005
        System.out.printf("After monthly interest: %.2f%n", balance); // 10050.00
        
        // Counting down:
        int countdown = 10;
        countdown -= 3;  // skip 3
        System.out.println("Countdown now at: " + countdown); // 7
        
        // ─────────────────────────────────────────
        // CHAINED ASSIGNMENT
        // ─────────────────────────────────────────
        
        // Multiple variables can be assigned the same value:
        int p, q, r;
        p = q = r = 0; // right-to-left: r=0, q=0, p=0
        System.out.println(p + " " + q + " " + r); // 0 0 0
        
        // Rarely used but valid Java.
        // More common pattern:
        int val1 = 0, val2 = 0, val3 = 0; // clearer
        
        // ─────────────────────────────────────────
        // BITWISE ASSIGNMENT (brief mention)
        // ─────────────────────────────────────────
        // These exist but are less common in backend web dev:
        // &=   bitwise AND assignment
        // |=   bitwise OR assignment
        // ^=   bitwise XOR assignment
        // <<=  left shift assignment
        // >>=  right shift assignment
        // >>>= unsigned right shift assignment
        // We'll cover these in bitwise section below.
    }
}
```

---

## 4. Comparison (Relational) Operators

```java
public class ComparisonOperators {
    public static void main(String[] args) {
        
        // Comparison operators ALWAYS return boolean (true or false)
        // They are used to BUILD CONDITIONS for if/while/for etc.
        
        int a = 10;
        int b = 20;
        int c = 10;
        
        // ─────────────────────────────────────────
        // EQUALITY AND INEQUALITY
        // ─────────────────────────────────────────
        
        System.out.println(a == b);  // false (10 equals 20? No)
        System.out.println(a == c);  // true  (10 equals 10? Yes)
        System.out.println(a != b);  // true  (10 not equals 20? Yes)
        System.out.println(a != c);  // false (10 not equals 10? No)
        
        // ─────────────────────────────────────────
        // MAGNITUDE COMPARISON
        // ─────────────────────────────────────────
        
        System.out.println(a > b);   // false (10 > 20? No)
        System.out.println(a < b);   // true  (10 < 20? Yes)
        System.out.println(a >= c);  // true  (10 >= 10? Yes, equal counts)
        System.out.println(a <= b);  // true  (10 <= 20? Yes)
        System.out.println(b >= b);  // true  (20 >= 20? Yes)
        System.out.println(b > b);   // false (20 > 20? No, not strictly greater)
        
        // ─────────────────────────────────────────
        // STORING COMPARISON RESULTS
        // ─────────────────────────────────────────
        
        int userAge = 21;
        int legalAge = 18;
        int seniorAge = 60;
        
        boolean canVote  = userAge >= legalAge;  // true
        boolean isSenior = userAge >= seniorAge; // false
        boolean isExact  = userAge == legalAge;  // false
        
        System.out.println("Can vote: " + canVote);   // true
        System.out.println("Is senior: " + isSenior); // false
        
        // ─────────────────────────────────────────
        // COMPARING DOUBLES — THE PROBLEM
        // ─────────────────────────────────────────
        
        double x = 0.1 + 0.2;
        double y = 0.3;
        
        System.out.println(x == y);   // false! (floating point precision error)
        System.out.println(x);        // 0.30000000000000004
        
        // FIX: use epsilon comparison for doubles
        double epsilon = 1e-9; // tolerance (0.000000001)
        boolean almostEqual = Math.abs(x - y) < epsilon;
        System.out.println("Almost equal: " + almostEqual); // true
        
        // In real code:
        // Use BigDecimal.compareTo() for exact decimal comparison
        // Or store money as long (no floating point issues)
        
        // ─────────────────────────────────────────
        // COMPARING OBJECTS — THE BIG TRAP
        // ─────────────────────────────────────────
        
        // == compares REFERENCES (memory addresses) for objects
        // == compares VALUES for primitives
        
        // PRIMITIVES — == compares values correctly
        int p1 = 100;
        int p2 = 100;
        System.out.println(p1 == p2); // true ✓ (same value)
        
        // STRINGS — == compares references (addresses), NOT content!
        String s1 = new String("hello"); // creates new object on heap
        String s2 = new String("hello"); // creates ANOTHER new object on heap
        String s3 = "hello";             // from String pool
        String s4 = "hello";             // same object from String pool
        
        System.out.println(s1 == s2);      // false (different objects!)
        System.out.println(s3 == s4);      // true  (same pool object)
        System.out.println(s1 == s3);      // false (pool vs heap)
        
        // ALWAYS use .equals() for String content comparison:
        System.out.println(s1.equals(s2)); // true  ✓ (same content)
        System.out.println(s1.equals(s3)); // true  ✓
        
        // equals() vs equalsIgnoreCase()
        String name1 = "Rahim";
        String name2 = "rahim";
        System.out.println(name1.equals(name2));            // false
        System.out.println(name1.equalsIgnoreCase(name2)); // true
        
        // OBJECTS — use .equals() ALWAYS (unless checking if same object)
        // Integer wrapper:
        Integer w1 = new Integer(200);
        Integer w2 = new Integer(200);
        System.out.println(w1 == w2);      // false (different objects)
        System.out.println(w1.equals(w2)); // true ✓
        
        // RULE TO REMEMBER:
        // == for primitives ✓
        // == for checking if two variables point to SAME object (rare)
        // .equals() for comparing CONTENT of objects ✓
        // NEVER == for String comparison
    }
}
```

---

## 5. Logical Operators

```java
public class LogicalOperators {
    public static void main(String[] args) {
        
        // Logical operators combine boolean expressions
        // Input: booleans   Output: boolean
        
        boolean isLoggedIn = true;
        boolean hasPermission = false;
        boolean isAdmin = true;
        boolean isActive = true;
        
        // ─────────────────────────────────────────
        // && (AND) — BOTH must be true
        // ─────────────────────────────────────────
        
        boolean canAccess = isLoggedIn && hasPermission;
        System.out.println("Can access: " + canAccess); // false
        // false because hasPermission is false
        
        boolean fullAccess = isLoggedIn && isAdmin;
        System.out.println("Full access: " + fullAccess); // true
        
        // Truth table for &&:
        // true  && true  = true
        // true  && false = false
        // false && true  = false
        // false && false = false
        // ONLY true when BOTH are true
        
        // ─────────────────────────────────────────
        // || (OR) — AT LEAST ONE must be true
        // ─────────────────────────────────────────
        
        boolean canView = isAdmin || hasPermission;
        System.out.println("Can view: " + canView); // true
        // true because isAdmin is true (doesn't matter that hasPermission is false)
        
        boolean canEdit = hasPermission || isAdmin;
        System.out.println("Can edit: " + canEdit); // true
        
        // Truth table for ||:
        // true  || true  = true
        // true  || false = true
        // false || true  = true
        // false || false = false
        // false ONLY when BOTH are false
        
        // ─────────────────────────────────────────
        // ! (NOT) — REVERSES the boolean
        // ─────────────────────────────────────────
        
        boolean isGuest = !isLoggedIn; // !true = false
        boolean isDisabled = !isActive; // !true = false
        
        System.out.println("Is guest: " + isGuest);       // false
        System.out.println("Is disabled: " + isDisabled); // false
        
        boolean shouldRetry = !isActive || !hasPermission;
        System.out.println("Should retry: " + shouldRetry); // true
        
        // Double negation (avoid — confusing):
        boolean notNotLoggedIn = !!isLoggedIn; // same as isLoggedIn
        
        // ─────────────────────────────────────────
        // SHORT-CIRCUIT EVALUATION ← CRITICAL CONCEPT
        // ─────────────────────────────────────────
        
        // && short-circuits on FALSE:
        // If LEFT side is false → RIGHT side is NEVER evaluated
        // Because false && anything = false (no need to check right)
        
        // || short-circuits on TRUE:
        // If LEFT side is true → RIGHT side is NEVER evaluated
        // Because true || anything = true (no need to check right)
        
        // WHY THIS MATTERS:
        
        String name = null;
        
        // This would CRASH:
        // if (name.length() > 0) {...} // NullPointerException!
        
        // This is SAFE because of short-circuit:
        if (name != null && name.length() > 0) {
            System.out.println("Name: " + name);
        } else {
            System.out.println("Name is null or empty"); // prints this
        }
        // name != null → false → short-circuits
        // name.length() is NEVER called → no NullPointerException
        
        // Another example:
        int[] numbers = null;
        // Safe because of short-circuit:
        if (numbers != null && numbers.length > 0) {
            System.out.println("First: " + numbers[0]);
        } else {
            System.out.println("Array is null or empty"); // prints this
        }
        
        // || short-circuit example:
        boolean hasCache = true;
        // If hasCache is true → doesn't call the expensive database method
        boolean hasData = hasCache || fetchFromDatabase(); // DB not called!
        System.out.println("Has data: " + hasData); // true
        
        // ─────────────────────────────────────────
        // NON-SHORT-CIRCUIT: & and | (rare)
        // ─────────────────────────────────────────
        
        // & (non-short-circuit AND): evaluates BOTH sides always
        // | (non-short-circuit OR): evaluates BOTH sides always
        // Use these only when you NEED both sides evaluated (rare)
        // Usually you want && and || (short-circuit versions)
        
        boolean result1 = false & someMethod(); // someMethod() IS called
        boolean result2 = false && someMethod(); // someMethod() NOT called
        
        // ─────────────────────────────────────────
        // COMBINING LOGICAL OPERATORS
        // ─────────────────────────────────────────
        
        int age = 25;
        double income = 50000.0;
        boolean hasCreditHistory = true;
        boolean hasDebt = false;
        
        // Loan eligibility check
        boolean isEligible = 
            age >= 21 &&
            age <= 65 &&
            income >= 30000 &&
            hasCreditHistory &&
            !hasDebt;
        
        System.out.println("Loan eligible: " + isEligible); // true
        
        // Password strength check
        String password = "MyPass123!";
        int length = password.length();
        boolean hasUpper = !password.equals(password.toLowerCase());
        boolean hasLower = !password.equals(password.toUpperCase());
        boolean hasDigit = password.matches(".*\\d.*");
        boolean isStrong = length >= 8 && hasUpper && hasLower && hasDigit;
        
        System.out.println("Strong password: " + isStrong); // true
    }
    
    // Helper methods for short-circuit demo
    static boolean fetchFromDatabase() {
        System.out.println("DATABASE CALLED!"); // proves it was called
        return true;
    }
    
    static boolean someMethod() {
        System.out.println("someMethod CALLED!");
        return true;
    }
}
```

---

## 6. The Ternary Operator

```java
public class TernaryOperator {
    public static void main(String[] args) {
        
        // SYNTAX:
        // condition ? valueIfTrue : valueIfFalse
        //
        // Read as: "if condition then valueIfTrue else valueIfFalse"
        
        // ─────────────────────────────────────────
        // BASIC USAGE
        // ─────────────────────────────────────────
        
        int age = 20;
        
        // Without ternary (verbose):
        String status1;
        if (age >= 18) {
            status1 = "Adult";
        } else {
            status1 = "Minor";
        }
        
        // With ternary (concise):
        String status2 = age >= 18 ? "Adult" : "Minor";
        
        System.out.println(status1); // Adult
        System.out.println(status2); // Adult (same result)
        
        // ─────────────────────────────────────────
        // PRACTICAL EXAMPLES
        // ─────────────────────────────────────────
        
        // 1. Absolute value (Math.abs() exists, but good example)
        int num = -15;
        int absValue = num < 0 ? -num : num;
        System.out.println("Absolute value: " + absValue); // 15
        
        // 2. Max of two numbers (Math.max() exists, but good example)
        int a = 42, b = 73;
        int max = a > b ? a : b;
        System.out.println("Max: " + max); // 73
        
        // 3. Null check (VERY common in real code)
        String name = null;
        String displayName = name != null ? name : "Anonymous";
        System.out.println("Display: " + displayName); // Anonymous
        
        // 4. Formatting output
        int items = 1;
        System.out.println(items + " " + (items == 1 ? "item" : "items"));
        // prints: 1 item
        
        items = 5;
        System.out.println(items + " " + (items == 1 ? "item" : "items"));
        // prints: 5 items
        
        // 5. Choosing between two values
        boolean isPremium = true;
        double price = isPremium ? 99.99 : 9.99;
        System.out.printf("Price: %.2f%n", price); // 99.99
        
        // 6. In method arguments (common in Spring Boot)
        System.out.println("Status: " + (age >= 18 ? "ADULT" : "MINOR"));
        
        // ─────────────────────────────────────────
        // NESTED TERNARY — USE WITH EXTREME CAUTION
        // ─────────────────────────────────────────
        
        int score = 75;
        
        // AVOID this — nearly unreadable:
        String grade = score >= 90 ? "A" : score >= 80 ? "B" : 
                       score >= 70 ? "C" : score >= 60 ? "D" : "F";
        
        // BETTER: use if-else if chain for multiple conditions
        String grade2;
        if (score >= 90)      grade2 = "A";
        else if (score >= 80) grade2 = "B";
        else if (score >= 70) grade2 = "C";
        else if (score >= 60) grade2 = "D";
        else                  grade2 = "F";
        
        System.out.println("Grade: " + grade);  // C
        System.out.println("Grade: " + grade2); // C (same)
        
        // RULE FOR TERNARY:
        // ✅ Use when replacing a simple if-else with two clear outcomes
        // ✅ Use for inline value selection in expressions
        // ❌ Avoid nesting ternary operators (kills readability)
        // ❌ Avoid when the expressions are long/complex
        
        // ─────────────────────────────────────────
        // TERNARY FOR NULL HANDLING (preview of Optional)
        // ─────────────────────────────────────────
        
        // Pattern you'll see constantly in real code:
        String userInput = null;
        int inputLength = userInput != null ? userInput.length() : 0;
        System.out.println("Length: " + inputLength); // 0 (safe, no NPE)
        
        String userEmail = null;
        String email = userEmail != null ? userEmail : "no-reply@example.com";
        System.out.println("Email: " + email); // no-reply@example.com
    }
}
```

---

## 7. Operator Precedence

```java
public class OperatorPrecedence {
    public static void main(String[] args) {
        
        // PRECEDENCE = which operator runs first (like math order of operations)
        // Higher precedence = runs first
        // Same precedence = left to right (usually)
        
        // PRECEDENCE TABLE (high to low):
        // 1. Postfix:     x++  x--
        // 2. Prefix/Unary: ++x  --x  +x  -x  !  ~
        // 3. Multiplicative: * / %
        // 4. Additive:    +  -
        // 5. Shift:       <<  >>  >>>
        // 6. Relational:  <  >  <=  >=  instanceof
        // 7. Equality:    ==  !=
        // 8. Bitwise AND: &
        // 9. Bitwise XOR: ^
        // 10. Bitwise OR: |
        // 11. Logical AND: &&
        // 12. Logical OR:  ||
        // 13. Ternary:    ?:
        // 14. Assignment: = += -= *= /= %= etc.
        
        // ─────────────────────────────────────────
        // EXAMPLES
        // ─────────────────────────────────────────
        
        // Multiplication before addition (like math):
        int result1 = 2 + 3 * 4;       // 2 + 12 = 14 (not 20!)
        int result2 = (2 + 3) * 4;     // 5 * 4 = 20 (parentheses force order)
        System.out.println(result1);    // 14
        System.out.println(result2);    // 20
        
        // Comparison before equality:
        boolean check = 5 + 3 > 4 * 2; // (5+3) > (4*2) → 8 > 8 → false
        System.out.println(check);      // false
        
        // Logical && before ||:
        boolean logic = true || false && false;
        // false && false = false (evaluated first, && has higher precedence)
        // true || false = true
        System.out.println(logic);      // true
        
        boolean logic2 = (true || false) && false;
        // (true || false) = true (parentheses force this first)
        // true && false = false
        System.out.println(logic2);     // false
        
        // NOT (!) before AND:
        boolean not1 = !true && false;   // (!true) && false → false && false → false
        boolean not2 = !(true && false); // !(false) → true
        System.out.println(not1);        // false
        System.out.println(not2);        // true
        
        // Assignment is RIGHT-to-LEFT associative:
        int x, y, z;
        x = y = z = 5; // z=5, then y=5, then x=5
        System.out.println(x + " " + y + " " + z); // 5 5 5
        
        // ─────────────────────────────────────────
        // THE GOLDEN RULE
        // ─────────────────────────────────────────
        
        // When in doubt → USE PARENTHESES
        // Parentheses always override precedence
        // They also make code MORE READABLE
        
        int a = 5, b = 7, c = 3;
        
        // ❌ Unclear (relies on precedence knowledge):
        boolean unclear = a > 0 && b < 10 || c == 5;
        
        // ✅ Clear (parentheses show intent):
        boolean clear = (a > 0 && b < 10) || (c == 5);
        
        System.out.println(unclear); // true
        System.out.println(clear);   // true (same but readable)
        
        // Real-world example:
        double price = 100.0;
        double taxRate = 0.15;
        double discount = 10.0;
        
        // Without parentheses (confusing):
        double total1 = price + price * taxRate - discount;
        
        // With parentheses (clear intent):
        double total2 = price + (price * taxRate) - discount;
        
        System.out.println(total1); // 105.0
        System.out.println(total2); // 105.0 (same, but total2 is clearer)
    }
}
```

---

## 8. String Concatenation with +

```java
public class StringConcatenation {
    public static void main(String[] args) {
        
        // The + operator is OVERLOADED for Strings
        // With numbers: addition
        // With Strings: concatenation (joining)
        
        // ─────────────────────────────────────────
        // BASIC CONCATENATION
        // ─────────────────────────────────────────
        
        String firstName = "Rahim";
        String lastName = "Ahmed";
        String fullName = firstName + " " + lastName;
        System.out.println(fullName); // Rahim Ahmed
        
        // Concatenating with non-String types (auto-converts to String)
        int age = 21;
        String message = "Age: " + age; // int → String automatically
        System.out.println(message);    // Age: 21
        
        double gpa = 3.15;
        System.out.println("GPA: " + gpa); // GPA: 3.15
        
        boolean isActive = true;
        System.out.println("Active: " + isActive); // Active: true
        
        // ─────────────────────────────────────────
        // THE CRITICAL TRAP — ORDER MATTERS
        // ─────────────────────────────────────────
        
        // Left to right evaluation!
        System.out.println("Sum: " + 1 + 2);   // "Sum: 12" ← WRONG!
        // "Sum: " + 1 = "Sum: 1" (String + int = String)
        // "Sum: 1" + 2 = "Sum: 12" (String + int = String)
        
        System.out.println("Sum: " + (1 + 2)); // "Sum: 3" ← CORRECT
        // (1 + 2) = 3 (int + int = int) — evaluated first due to ()
        // "Sum: " + 3 = "Sum: 3"
        
        System.out.println(1 + 2 + " is the sum"); // "3 is the sum" ✓
        // 1 + 2 = 3 (both int, addition happens first, left to right)
        // 3 + " is the sum" = "3 is the sum"
        
        System.out.println(1 + 2 + "3"); // "33" ← careful!
        // 1 + 2 = 3 (int addition)
        // 3 + "3" = "33" (String concatenation)
        
        System.out.println("" + 1 + 2); // "12" ← careful!
        // "" + 1 = "1" (String concat)
        // "1" + 2 = "12" (String concat)
        
        // ─────────────────────────────────────────
        // PERFORMANCE ISSUE WITH + IN LOOPS
        // ─────────────────────────────────────────
        
        // ❌ BAD: String concatenation in loops creates many objects
        String result = "";
        for (int i = 0; i < 5; i++) {
            result = result + i; // creates NEW String each iteration!
        }
        System.out.println(result); // "01234"
        // With 5 iterations: creates 5 intermediate String objects
        // With 10,000 iterations: creates 10,000 objects → SLOW
        
        // ✅ GOOD: Use StringBuilder (mutable String)
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 5; i++) {
            sb.append(i); // modifies same object, no new objects created
        }
        System.out.println(sb.toString()); // "01234"
        // Much faster for large loops
        
        // StringBuilder is the professional approach for building strings:
        StringBuilder profile = new StringBuilder();
        profile.append("Name: ").append("Rahim").append("\n");
        profile.append("Age: ").append(21).append("\n");
        profile.append("GPA: ").append(3.15).append("\n");
        System.out.println(profile.toString());
        
        // ─────────────────────────────────────────
        // MODERN STRING APPROACHES
        // ─────────────────────────────────────────
        
        // String.format() — like printf but returns String
        String formatted = String.format("Name: %s, Age: %d", "Rahim", 21);
        System.out.println(formatted);
        
        // Formatted strings (Java 15+) — same as String.format()
        String modern = "Name: %s, Age: %d".formatted("Rahim", 21);
        System.out.println(modern);
        
        // Text blocks (Java 15+) — multiline strings
        String json = """
                {
                    "name": "Rahim",
                    "age": 21,
                    "city": "Dhaka"
                }
                """;
        System.out.println(json);
        
        // Use text blocks for:
        // SQL queries in Java
        // JSON templates
        // HTML templates
        // Long multiline strings
    }
}
```

---

## 9. Bitwise Operators (Awareness Level)

```java
public class BitwiseOperators {
    public static void main(String[] args) {
        
        // Bitwise operators work on individual BITS of integers
        // Less commonly used in high-level backend web dev
        // BUT appears in: flags, permissions, networking, encryption,
        //                 performance optimization, interview questions
        
        // Numbers in binary:
        // 5  = 0101
        // 3  = 0011
        // 12 = 1100
        // 10 = 1010
        
        int a = 5;  // 0000 0101
        int b = 3;  // 0000 0011
        
        // ─────────────────────────────────────────
        // & (Bitwise AND) — bit is 1 only if BOTH bits are 1
        // ─────────────────────────────────────────
        System.out.println(a & b); // 1
        // 0101
        // 0011 &
        // ----
        // 0001 = 1
        
        // USE: checking if a specific bit is set
        // (we'll see this with permission flags below)
        
        // ─────────────────────────────────────────
        // | (Bitwise OR) — bit is 1 if EITHER bit is 1
        // ─────────────────────────────────────────
        System.out.println(a | b); // 7
        // 0101
        // 0011 |
        // ----
        // 0111 = 7
        
        // ─────────────────────────────────────────
        // ^ (Bitwise XOR) — bit is 1 if bits are DIFFERENT
        // ─────────────────────────────────────────
        System.out.println(a ^ b); // 6
        // 0101
        // 0011 ^
        // ----
        // 0110 = 6
        
        // ─────────────────────────────────────────
        // ~ (Bitwise NOT) — flips all bits
        // ─────────────────────────────────────────
        System.out.println(~a); // -6
        // 0000...0101 → 1111...1010 = -6 (two's complement)
        
        // ─────────────────────────────────────────
        // << (Left Shift) — shift bits left, multiply by 2^n
        // ─────────────────────────────────────────
        System.out.println(1 << 0);  // 1  (1 × 2^0)
        System.out.println(1 << 1);  // 2  (1 × 2^1)
        System.out.println(1 << 2);  // 4  (1 × 2^2)
        System.out.println(1 << 3);  // 8  (1 × 2^3)
        System.out.println(1 << 4);  // 16 (1 × 2^4)
        System.out.println(5 << 1);  // 10 (5 × 2 = 10)
        // Fast multiplication by powers of 2
        
        // ─────────────────────────────────────────
        // >> (Right Shift) — shift bits right, divide by 2^n
        // ─────────────────────────────────────────
        System.out.println(16 >> 1); // 8  (16 / 2)
        System.out.println(16 >> 2); // 4  (16 / 4)
        System.out.println(16 >> 3); // 2  (16 / 8)
        // Sign bit is preserved (arithmetic shift)
        
        // ─────────────────────────────────────────
        // REAL-WORLD USE: Permission Flags
        // ─────────────────────────────────────────
        // Each permission is a separate bit
        // Common pattern in Linux file permissions, databases
        
        final int READ    = 1;  // 001
        final int WRITE   = 2;  // 010
        final int EXECUTE = 4;  // 100
        final int DELETE  = 8;  // 1000
        
        // Assign permissions using OR (combine bits)
        int userPermissions = READ | WRITE; // 011 = 3
        System.out.println("User permissions: " + userPermissions); // 3
        
        // Add execute permission:
        userPermissions = userPermissions | EXECUTE; // 111 = 7
        System.out.println("With execute: " + userPermissions); // 7
        
        // Check if user HAS a permission using AND:
        boolean canRead    = (userPermissions & READ) != 0;
        boolean canWrite   = (userPermissions & WRITE) != 0;
        boolean canExecute = (userPermissions & EXECUTE) != 0;
        boolean canDelete  = (userPermissions & DELETE) != 0;
        
        System.out.println("Can read: " + canRead);       // true
        System.out.println("Can write: " + canWrite);     // true
        System.out.println("Can execute: " + canExecute); // true
        System.out.println("Can delete: " + canDelete);   // false
        
        // Remove a permission using AND NOT:
        userPermissions = userPermissions & ~WRITE; // remove write bit
        boolean canWriteNow = (userPermissions & WRITE) != 0;
        System.out.println("Can write after removal: " + canWriteNow); // false
        
        // Toggle a permission using XOR:
        userPermissions = userPermissions ^ READ; // toggle read bit
        // If read was on → turn off. If off → turn on.
        System.out.println("After toggling read: " + ((userPermissions & READ) != 0));
    }
}
```

---

## 10. `instanceof` Operator

```java
public class InstanceofOperator {
    public static void main(String[] args) {
        
        // instanceof checks if an object is of a specific type
        // Returns: true or false
        
        Object str = "Hello";
        Object num = 42;
        Object list = new java.util.ArrayList<>();
        
        System.out.println(str instanceof String);       // true
        System.out.println(str instanceof Integer);      // false
        System.out.println(num instanceof Integer);      // true
        System.out.println(list instanceof java.util.List); // true
        
        // null instanceof AnyType = always false (no exception)
        String nullStr = null;
        System.out.println(nullStr instanceof String); // false (safe!)
        
        // ─────────────────────────────────────────
        // PATTERN MATCHING instanceof (Java 16+)
        // ─────────────────────────────────────────
        // Old way:
        Object obj = "Hello World";
        if (obj instanceof String) {
            String s = (String) obj; // manual cast needed
            System.out.println(s.length()); // 11
        }
        
        // New way (Java 16+) — pattern matching:
        if (obj instanceof String s) { // type check + cast + name in ONE
            System.out.println(s.length()); // 11 — s is already a String!
        }
        
        // Real-world use: processing different event types
        // (You'll see this in event-driven systems)
        processEvent("UserCreated");
        processEvent(404);
        processEvent(3.14);
    }
    
    static void processEvent(Object event) {
        // Pattern matching makes this clean:
        if (event instanceof String message) {
            System.out.println("String event: " + message.toUpperCase());
        } else if (event instanceof Integer code) {
            System.out.println("Integer event: code " + code);
        } else if (event instanceof Double value) {
            System.out.printf("Double event: %.2f%n", value);
        } else {
            System.out.println("Unknown event type");
        }
    }
}
```

---

## Build This - Complete Operators Practice

```java
// File: OperatorsPractice.java
// Build a simple order calculation system using operators

public class OperatorsPractice {
    
    // Constants
    static final double TAX_RATE        = 0.15;  // 15%
    static final double SENIOR_DISCOUNT = 0.10;  // 10%
    static final double BULK_DISCOUNT   = 0.05;  // 5% for 10+ items
    static final int    MIN_BULK_QTY    = 10;
    static final long   MAX_ORDER_BDT   = 100_000_00L; // 1 lakh BDT in poisha
    
    public static void main(String[] args) {
        
        // ═══════════════════════════════════════
        // ORDER INFORMATION
        // ═══════════════════════════════════════
        String customerName = "Rahim Ahmed";
        int    customerAge  = 65;
        int    quantity     = 12;
        long   unitPricePoisha = 50_00L; // 50 BDT in poisha
        
        // ═══════════════════════════════════════
        // CALCULATIONS USING OPERATORS
        // ═══════════════════════════════════════
        
        // Subtotal
        long subtotalPoisha = unitPricePoisha * quantity;
        
        // Check eligibility using comparison + logical operators
        boolean isSenior    = customerAge >= 60;
        boolean isBulkOrder = quantity >= MIN_BULK_QTY;
        boolean isLargeOrder = subtotalPoisha > MAX_ORDER_BDT;
        
        // Calculate discounts using ternary
        double seniorDiscountRate = isSenior ? SENIOR_DISCOUNT : 0.0;
        double bulkDiscountRate   = isBulkOrder ? BULK_DISCOUNT : 0.0;
        
        // Total discount rate (cannot exceed 20%)
        double totalDiscountRate = seniorDiscountRate + bulkDiscountRate;
        totalDiscountRate = totalDiscountRate > 0.20 ? 0.20 : totalDiscountRate;
        
        // Apply discount
        long discountPoisha = (long)(subtotalPoisha * totalDiscountRate);
        long afterDiscountPoisha = subtotalPoisha - discountPoisha;
        
        // Apply tax
        long taxPoisha = (long)(afterDiscountPoisha * TAX_RATE);
        long totalPoisha = afterDiscountPoisha + taxPoisha;
        
        // Can we fulfill the order?
        boolean canFulfill = quantity > 0 && unitPricePoisha > 0 && !isLargeOrder;
        
        // ═══════════════════════════════════════
        // ORDER SUMMARY
        // ═══════════════════════════════════════
        System.out.println("╔═══════════════════════════════════════╗");
        System.out.println("║           ORDER SUMMARY               ║");
        System.out.println("╠═══════════════════════════════════════╣");
        System.out.printf( "║ Customer  : %-25s ║%n", customerName);
        System.out.printf( "║ Age       : %-25d ║%n", customerAge);
        System.out.printf( "║ Quantity  : %-25d ║%n", quantity);
        System.out.printf( "║ Unit Price: %-25s ║%n", 
                           "৳" + (unitPricePoisha / 100.0));
        System.out.println("╠═══════════════════════════════════════╣");
        System.out.printf( "║ Subtotal  : %-25s ║%n", 
                           "৳" + String.format("%.2f", subtotalPoisha / 100.0));
        System.out.printf( "║ Senior?   : %-25s ║%n", 
                           isSenior ? "Yes (-" + (int)(SENIOR_DISCOUNT*100) + "%)" : "No");
        System.out.printf( "║ Bulk?     : %-25s ║%n", 
                           isBulkOrder ? "Yes (-" + (int)(BULK_DISCOUNT*100) + "%)" : "No");
        System.out.printf( "║ Discount  : %-25s ║%n", 
                           "৳" + String.format("%.2f", discountPoisha / 100.0) 
                           + " (" + (int)(totalDiscountRate*100) + "%)");
        System.out.printf( "║ Tax (15%%) : %-25s ║%n", 
                           "৳" + String.format("%.2f", taxPoisha / 100.0));
        System.out.println("╠═══════════════════════════════════════╣");
        System.out.printf( "║ TOTAL     : %-25s ║%n", 
                           "৳" + String.format("%.2f", totalPoisha / 100.0));
        System.out.printf( "║ Status    : %-25s ║%n", 
                           canFulfill ? "✓ APPROVED" : "✗ REJECTED");
        System.out.println("╚═══════════════════════════════════════╝");
        
        // ═══════════════════════════════════════
        // BITWISE DEMO: Permission Check
        // ═══════════════════════════════════════
        final int VIEW   = 1;
        final int CREATE = 2;
        final int EDIT   = 4;
        final int DELETE = 8;
        
        int adminPermissions = VIEW | CREATE | EDIT | DELETE; // 15
        int staffPermissions = VIEW | CREATE | EDIT;           // 7
        int guestPermissions = VIEW;                           // 1
        
        System.out.println("\n╔═══════════════════════════════════════╗");
        System.out.println("║        PERMISSION MATRIX              ║");
        System.out.println("╠══════════╦═══════╦═══════╦════════════╣");
        System.out.println("║ Role     ║ View  ║ Create║ Edit Delete║");
        System.out.println("╠══════════╬═══════╬═══════╬════════════╣");
        
        printPermRow("Admin", adminPermissions, VIEW, CREATE, EDIT, DELETE);
        printPermRow("Staff", staffPermissions, VIEW, CREATE, EDIT, DELETE);
        printPermRow("Guest", guestPermissions, VIEW, CREATE, EDIT, DELETE);
        
        System.out.println("╚══════════╩═══════╩═══════╩════════════╝");
    }
    
    static void printPermRow(String role, int perms, 
                              int view, int create, int edit, int delete) {
        System.out.printf("║ %-8s ║  %-4s ║  %-4s ║  %-4s  %-4s ║%n",
            role,
            (perms & view)   != 0 ? "✓" : "✗",
            (perms & create) != 0 ? "✓" : "✗",
            (perms & edit)   != 0 ? "✓" : "✗",
            (perms & delete) != 0 ? "✓" : "✗"
        );
    }
}
```

---

## Exercises

```text
EXERCISE 1: Arithmetic Deep Dive
  Create ArithmeticExercise.java
  Write a program that:
  - Given a number of seconds (e.g. 86400), converts to
    days, hours, minutes, seconds using only / and %
  - Calculates average of 5 numbers WITHOUT using array
  - Demonstrates integer division trap with a real example
  - Shows why (double)a/b differs from a/b for int variables
  Push with good commit message.

EXERCISE 2: Modulo Magic
  Create ModuloMagic.java
  Use modulo to:
  - Determine if a year is a leap year
    (divisible by 4, except centuries unless divisible by 400)
  - Check if a number is divisible by both 3 and 7
  - Implement a circular buffer index (size 5, indices 0-14)
  - Get the last 4 digits of a large number

EXERCISE 3: Logical Operator Challenge
  Create AccessControl.java
  Build an access control system:
  - User has: age, isVerified, isPremium, isBanned
  - canView   = not banned AND verified
  - canPost   = canView AND age >= 16
  - canUpload = canPost AND premium
  - canAdmin  = canUpload AND age >= 18
  Test with 4 different users (different combinations)
  Print access table for each user

EXERCISE 4: Operator Precedence Quiz
  Without running the code, calculate the result.
  Then run to verify.
  a) int x = 2 + 3 * 4 - 1;
  b) boolean y = 5 > 3 && 2 < 1 || 8 > 6;
  c) int z = 10 / 3 * 3 + 10 % 3;
  d) boolean w = !true || false && !false;
  e) String s = "A" + 1 + 2 + "B";
  f) String t = "A" + (1 + 2) + "B";
  Write expected values first, THEN run.

EXERCISE 5: Build a Grade Calculator
  Create GradeCalculator.java
  - Input: 5 assignment scores (hardcoded)
  - Calculate: total, average, percentage
  - Determine grade using ternary operator:
    90+ = "A", 80+ = "B", 70+ = "C", 60+ = "D", else = "F"
  - Determine: pass/fail, needs improvement
  - Print formatted report using printf
  - Push to GitHub
```

---

## Common Mistakes

```text
MISTAKE 1: Integer division when expecting decimal
  int a = 5, b = 2;
  double result = a / b; // 2.0 NOT 2.5!
  Fix: double result = (double) a / b; // 2.5 ✓

MISTAKE 2: Precedence surprise with +
  System.out.println("Value: " + 3 + 4); // "Value: 34" not "Value: 7"
  Fix: System.out.println("Value: " + (3 + 4)); // "Value: 7" ✓

MISTAKE 3: Using == for String comparison
  String s = "hello";
  if (s == "hello") { ... } // sometimes works (pool), unreliable
  Fix: if (s.equals("hello")) { ... } // always correct ✓

MISTAKE 4: Forgetting short-circuit saves from NullPointerException
  if (name.length() > 0) // NPE if name is null!
  Fix: if (name != null && name.length() > 0) // short-circuits safely ✓

MISTAKE 5: && vs & confusion
  && short-circuits (right not evaluated if left is false) → use this
  & evaluates both sides always → only when you need side effects (rare)

MISTAKE 6: Modulo with negative numbers
  -7 % 3 = -1 in Java (not 2!)
  Sign follows the dividend.
  Fix if you need positive: ((n % m) + m) % m

MISTAKE 7: Compound assignment narrowing (actually a feature)
  byte b = 10;
  b = b + 5;  // COMPILE ERROR (b + 5 is int)
  b += 5;     // ✓ Works (implicit cast included)
  Understanding this prevents confusion.

MISTAKE 8: Integer overflow silently
  int max = Integer.MAX_VALUE;
  int overflow = max + 1; // = Integer.MIN_VALUE, NO ERROR
  Fix: use long for potentially large values
  Or: Math.addExact(max, 1) → throws exception on overflow ✓

MISTAKE 9: Confusing pre/post increment in expressions
  int x = 5;
  int y = x++ + ++x; // x=5 then 7, y=5+7=12? Actually: 5+7=12
  Just avoid mixing pre/post in same expression.
  Use them standalone where possible.

MISTAKE 10: Comparing doubles with ==
  0.1 + 0.2 == 0.3 → false
  Fix: Math.abs((0.1 + 0.2) - 0.3) < 1e-9 → true ✓
```

---

## Interview Questions

**Q: What is the difference between == and .equals() in Java?**
A: `==` compares memory addresses (references) for objects and actual values for primitives. `.equals()` compares the content of objects. For primitives: `int a=5, b=5 → a==b` is true (same value). For objects: two String objects with same content can have different addresses, so `==` may be false while `.equals()` is true. Always use `.equals()` for String and object comparison. Never use `==` for String content comparison.

**Q: What is short-circuit evaluation?**
A: With `&&` (AND), if the left operand is false, the right is never evaluated because false `&&` anything is always false. With `||` (OR), if the left is true, the right is never evaluated because true `||` anything is always true. This is used for null safety: (`obj != null && obj.method()`) - if obj is null, the method is never called, preventing NullPointerException. It also provides performance benefits by skipping expensive operations when unnecessary.

**Q: What does the `%` (modulo) operator do?**
A: Returns the remainder after integer division. 17 % 5 = 2 because 17 / 5 = 3 with 2 left over. Common uses: checking even/odd (n % 2 == 0), checking divisibility, circular array indexing (i % arrayLength), and extracting digits (n % 10 gives last digit). In Java, the sign of the result follows the dividend: -7 % 3 = -1.

**Q: What is the difference between i++ and ++i?**
A: Both increment i by 1. The difference is when the value is used in an expression. i++ (post-increment) uses the current value first, then increments: int a = i++ with i=5 gives a=5, then i=6. ++i (pre-increment) increments first, then uses the value: int a = ++i with i=5 gives i=6, then a=6. In standalone statements like i++ or ++i alone on a line, there is no difference.

**Q: Why should you use parentheses even when you know operator precedence?**
A: Readability and maintainability. Code is read many more times than it is written. When you add parentheses to show intent, future readers (including yourself in 6 months) don't have to mentally parse precedence rules. It also prevents bugs when code is refactored. The compiler optimizes away parentheses, so there is no performance cost. Always prioritize clarity over cleverness.

**Q: What happens when you do integer division in Java?**
A: The decimal part is truncated (not rounded) toward zero. 7 / 2 = 3 (not 3.5). -7 / 2 = -3 (not -4, truncates toward zero). This is a common bug source when you accidentally use integer division expecting a decimal result. Fix by ensuring at least one operand is a floating point type: (double) a / b or a / 2.0.

---

## Key Takeaways

```text
1. ARITHMETIC: + - * / % ++ --
   Integer division TRUNCATES (7/2=3, not 3.5)
   Cast to double first if you need decimal result
   Never use double/float for money (precision errors)

2. MODULO (%) = remainder. Incredibly useful:
   Even/odd, divisibility, circular indexing, digit extraction
   Sign follows dividend in Java: -7 % 3 = -1

3. ASSIGNMENT shortcuts: += -= *= /= %=
   Include implicit cast back to left-hand type
   b += 5 works even when b is byte (b = b + 5 doesn't)

4. COMPARISON: == != > < >= <=
   For PRIMITIVES: == compares values correctly
   For OBJECTS/STRINGS: == compares addresses (wrong!)
   ALWAYS use .equals() for object content comparison

5. LOGICAL: && || !
   && = BOTH true. || = AT LEAST ONE true. ! = flip
   SHORT-CIRCUIT: && stops at first false, || at first true
   Use (obj != null && obj.method()) for null safety

6. TERNARY: condition ? valueIfTrue : valueIfFalse
   Replaces simple if-else for value selection
   Don't nest them (kills readability)
   Perfect for null fallbacks and binary choices

7. OPERATOR PRECEDENCE:
   * / % before + -
   Comparison before equality before logical
   && before ||
   When in doubt: USE PARENTHESES

8. STRING + operator:
   Evaluates LEFT to RIGHT
   "A" + 1 + 2 = "A12" (not "A3")
   "A" + (1 + 2) = "A3" (parentheses force math first)

9. BITWISE: & | ^ ~ << >> >>>
   Used for permission flags, networking, optimization
   Know the patterns: check (n & FLAG) != 0
   Add FLAG: perms |= FLAG
   Remove FLAG: perms &= ~FLAG

10. instanceof checks type at runtime safely
    null instanceof AnyType = false (no exception)
    Pattern matching: instanceof String s (Java 16+)
```

---