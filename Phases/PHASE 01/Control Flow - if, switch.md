**Phase:** Level 1 - Java Fundamentals
**Date Studied:**

---
## What Problem Does This Solve?
Every program you will ever write needs to make decisions.
```text
Without control flow, your program is a straight line:
  Do this → Do this → Do this → Done.
  No decisions. No branches. No conditions.
  Like a recipe that can't handle substitutions.
  
Real applications make THOUSANDS of decisions every second:
  Is this user authenticated?         → show page or redirect to login
  Does the user have enough balance?  → allow or reject withdrawal
  Is this email already registered?   → create account or show error
  Is the product in stock?            → add to cart or show "out of stock"
  Is it past midnight?                → show today's menu or tomorrow's
  Did the payment succeed?            → confirm order or retry
  Is the user an admin?               → show admin panel or hide it
  
ALL of these are control flow decisions.
  
Control flow = directing the path of execution
based on CONDITIONS that are evaluated at runtime.
```
You've seen variables (data) and operators (expressions).
Now you learn how to USE those expressions to control
what your program actually does.

---
## The Big Picture - All Control Flow in Java

```text
CONTROL FLOW TOOLS IN JAVA:
  
┌─────────────────────────────────────────────────────────┐
│  DECISION MAKING (this note):                           │
│    if / else if / else                                  │
│    switch statement (traditional)                       │
│    switch expression (modern Java 14+)                  │
│                                                         │
│  LOOPS (next note — Level 1.6):                         │
│    for, while, do-while, for-each                       │
│                                                         │
│  JUMPING (covered in loops):                            │
│    break, continue, return                              │
└─────────────────────────────────────────────────────────┘
  
TODAY: Decision making — if and switch.
Every program needs to choose between paths.
These are the tools for making that choice.
```

---
## 1. The if Statement - Core Decision Making

### Basic if
```java
public class IfBasics {
    public static void main(String[] args) {
        
        // SYNTAX:
        // if (condition) {
        //     // code runs ONLY if condition is true
        // }
        
        // The condition MUST be a boolean expression
        // (something that evaluates to true or false)
        
        int balance = 1000;
        int withdrawAmount = 500;
        
        // Simple if: runs only when true
        if (balance >= withdrawAmount) {
            System.out.println("Withdrawal approved.");
            balance = balance - withdrawAmount;
            System.out.println("New balance: " + balance);
        }
        // If balance < withdrawAmount, the block is SKIPPED entirely
        // Program continues after the closing }
        
        System.out.println("Program continues here regardless.");
        
        // ─────────────────────────────────────────
        // WHAT CAN BE THE CONDITION?
        // ─────────────────────────────────────────
        
        // Any expression that evaluates to boolean:
        
        int x = 10;
        
        if (x > 5) { System.out.println("x > 5"); }        // comparison
        if (x == 10) { System.out.println("x is 10"); }    // equality
        if (x != 0) { System.out.println("x is not 0"); }  // inequality
        
        boolean isActive = true;
        if (isActive) { System.out.println("Active"); }     // direct boolean
        if (!isActive) { System.out.println("Not active"); } // negated boolean
        
        String name = "Rahim";
        if (name != null) { System.out.println("Has name"); } // null check
        if (name.equals("Rahim")) { System.out.println("It's Rahim"); }
        
        // Combined conditions:
        int age = 25;
        double income = 50000;
        if (age >= 21 && income >= 30000) {
            System.out.println("Loan eligible");
        }
        
        // ─────────────────────────────────────────
        // SINGLE-LINE if — WHEN NO BRACES
        // ─────────────────────────────────────────
        
        // Java allows if without braces for single statements:
        if (x > 5) System.out.println("x is greater than 5"); // valid
        
        // ⚠️  BUT: this is a common source of bugs
        // Only the IMMEDIATELY NEXT line is in the if block:
        
        if (x > 100)
            System.out.println("This is in the if"); // only this line
            System.out.println("This ALWAYS runs!"); // NOT in the if block!
        // The indentation is DECEPTIVE — Java doesn't care about indentation
        
        // PROFESSIONAL RULE:
        // ALWAYS use braces {} even for single-line if bodies.
        // Prevents bugs when you add more lines later.
        // Most style guides (Google, Oracle) require this.
        
        // ✅ Always do this:
        if (x > 100) {
            System.out.println("x is over 100");
        }
    }
}
```

### if-else
```java
public class IfElse {
    public static void main(String[] args) {
        
        // SYNTAX:
        // if (condition) {
        //     // runs when condition is TRUE
        // } else {
        //     // runs when condition is FALSE
        // }
        // Exactly ONE of these two blocks ALWAYS runs.
        
        int age = 16;
        
        if (age >= 18) {
            System.out.println("You can vote.");
        } else {
            System.out.println("You cannot vote yet.");
            int yearsLeft = 18 - age;
            System.out.println("Years until eligible: " + yearsLeft);
        }
        // Output: You cannot vote yet.
        //         Years until eligible: 2
        
        // ─────────────────────────────────────────
        // REAL-WORLD PATTERN: Validate then proceed
        // ─────────────────────────────────────────
        
        double balance = 1000.0;
        double withdrawAmount = 1500.0;
        
        if (balance >= withdrawAmount) {
            balance -= withdrawAmount;
            System.out.printf("Withdrawn: %.2f%n", withdrawAmount);
            System.out.printf("Remaining: %.2f%n", balance);
        } else {
            System.out.println("Insufficient funds.");
            System.out.printf("Need %.2f more.%n", withdrawAmount - balance);
        }
        
        // ─────────────────────────────────────────
        // EARLY RETURN PATTERN (used heavily in Spring Boot)
        // ─────────────────────────────────────────
        
        // Instead of:
        validateUser_bad(null);
        validateUser_bad("Rahim");
        
        // Professional code uses early returns:
        validateUser_good(null);
        validateUser_good("Rahim");
    }
    
    // BAD: deep nesting
    static void validateUser_bad(String username) {
        if (username != null) {
            if (!username.isEmpty()) {
                if (username.length() >= 3) {
                    System.out.println("Valid user: " + username);
                } else {
                    System.out.println("Username too short");
                }
            } else {
                System.out.println("Username is empty");
            }
        } else {
            System.out.println("Username is null");
        }
    }
    
    // GOOD: early returns (guard clauses)
    // Each guard clause handles an error case and exits early
    // The "happy path" is at the bottom, not nested
    static void validateUser_good(String username) {
        if (username == null) {
            System.out.println("Username is null");
            return; // exit method immediately
        }
        
        if (username.isEmpty()) {
            System.out.println("Username is empty");
            return;
        }
        
        if (username.length() < 3) {
            System.out.println("Username too short");
            return;
        }
        
        // Happy path: we know username is valid here
        System.out.println("Valid user: " + username);
    }
}
```

### if-else if-else Chain
```java
public class IfElseIfChain {
    public static void main(String[] args) {
        
        // SYNTAX:
        // if (condition1) {
        //     // runs if condition1 is true
        // } else if (condition2) {
        //     // runs if condition1 false AND condition2 true
        // } else if (condition3) {
        //     // runs if condition1,2 false AND condition3 true
        // } else {
        //     // runs if ALL conditions above are false
        //     // (the "default" / "catch-all" case)
        // }
        //
        // EXACTLY ONE block runs. Always.
        // Once a true condition is found → runs that block → skips rest
        
        // ─────────────────────────────────────────
        // EXAMPLE 1: Grade classification
        // ─────────────────────────────────────────
        
        int score = 78;
        String grade;
        
        if (score >= 90) {
            grade = "A";
        } else if (score >= 80) {
            grade = "B";
        } else if (score >= 70) {
            grade = "C";  // 78 lands here ← first true condition wins
        } else if (score >= 60) {
            grade = "D";
        } else {
            grade = "F";
        }
        
        System.out.println("Score: " + score + " → Grade: " + grade);
        // Score: 78 → Grade: C
        
        // ─────────────────────────────────────────
        // EXAMPLE 2: HTTP Status Code handler
        // (You'll write this in Spring Boot constantly)
        // ─────────────────────────────────────────
        
        int statusCode = 404;
        String statusMessage;
        
        if (statusCode == 200) {
            statusMessage = "OK - Request successful";
        } else if (statusCode == 201) {
            statusMessage = "Created - Resource created successfully";
        } else if (statusCode == 400) {
            statusMessage = "Bad Request - Invalid input";
        } else if (statusCode == 401) {
            statusMessage = "Unauthorized - Authentication required";
        } else if (statusCode == 403) {
            statusMessage = "Forbidden - Access denied";
        } else if (statusCode == 404) {
            statusMessage = "Not Found - Resource does not exist"; // ← this
        } else if (statusCode == 409) {
            statusMessage = "Conflict - Resource already exists";
        } else if (statusCode >= 500) {
            statusMessage = "Server Error - Something went wrong on server";
        } else {
            statusMessage = "Unknown status code: " + statusCode;
        }
        
        System.out.println(statusCode + ": " + statusMessage);
        // 404: Not Found - Resource does not exist
        
        // ─────────────────────────────────────────
        // EXAMPLE 3: bKash-style transaction fee
        // ─────────────────────────────────────────
        
        double transactionAmount = 5000.0; // BDT
        double fee;
        
        if (transactionAmount <= 100) {
            fee = 0;                         // free for small transactions
        } else if (transactionAmount <= 1000) {
            fee = transactionAmount * 0.005; // 0.5%
        } else if (transactionAmount <= 5000) {
            fee = transactionAmount * 0.01;  // 1% ← 5000 lands here
        } else if (transactionAmount <= 25000) {
            fee = transactionAmount * 0.015; // 1.5%
        } else {
            fee = transactionAmount * 0.02;  // 2% for large amounts
        }
        
        System.out.printf("Transaction: %.2f BDT%n", transactionAmount);
        System.out.printf("Fee: %.2f BDT%n", fee);
        System.out.printf("You receive: %.2f BDT%n", transactionAmount - fee);
        
        // ─────────────────────────────────────────
        // ORDER OF CONDITIONS MATTERS
        // ─────────────────────────────────────────
        
        int value = 95;
        
        // WRONG ORDER — bug!
        if (value >= 60) {
            System.out.println("D or above"); // catches EVERYTHING >= 60
        } else if (value >= 70) {
            System.out.println("C or above"); // NEVER reached!
        } else if (value >= 90) {
            System.out.println("A");          // NEVER reached!
        }
        // Output: D or above (WRONG for 95)
        
        // CORRECT ORDER — specific to general
        if (value >= 90) {
            System.out.println("A");          // catches 90-100 ← 95 here
        } else if (value >= 70) {
            System.out.println("C or above"); // catches 70-89
        } else if (value >= 60) {
            System.out.println("D or above"); // catches 60-69
        }
        // Output: A (correct!)
        
        // RULE: order conditions from MOST SPECIFIC to LEAST SPECIFIC
    }
}
```

---
## 2. Nested if - When Decisions Have Sub-decisions
```java
public class NestedIf {
    public static void main(String[] args) {
        
        // if inside another if = nested if
        // Use carefully — too many levels = "pyramid of doom"
        
        boolean isLoggedIn = true;
        boolean isAdmin = false;
        boolean isAccountVerified = true;
        String targetUserId = "user-123";
        String currentUserId = "user-456";
        
        // ─────────────────────────────────────────
        // EXAMPLE: Role-based access control
        // (Simplified version of what Spring Security does)
        // ─────────────────────────────────────────
        
        if (isLoggedIn) {
            System.out.println("User is logged in.");
            
            if (!isAccountVerified) {
                System.out.println("Please verify your email first.");
            } else if (isAdmin) {
                System.out.println("Admin: Full access granted.");
            } else {
                // Regular user
                if (currentUserId.equals(targetUserId)) {
                    System.out.println("You can view your own profile.");
                } else {
                    System.out.println("You can only view public data.");
                }
            }
        } else {
            System.out.println("Please log in to continue.");
        }
        // Output:
        // User is logged in.
        // You can only view public data.
        
        // ─────────────────────────────────────────
        // THE PYRAMID OF DOOM — AVOID THIS
        // ─────────────────────────────────────────
        
        // This is hard to read and maintain:
        if (isLoggedIn) {
            if (isAccountVerified) {
                if (isAdmin) {
                    if (targetUserId != null) {
                        if (!targetUserId.isEmpty()) {
                            System.out.println("Finally doing the thing");
                        }
                    }
                }
            }
        }
        // 5 levels of nesting → PYRAMID OF DOOM
        
        // REFACTORED with guard clauses (early returns):
        // (would be in a method — shown as concept)
        // if (!isLoggedIn) return;
        // if (!isAccountVerified) return;
        // if (!isAdmin) return;
        // if (targetUserId == null || targetUserId.isEmpty()) return;
        // System.out.println("Finally doing the thing"); // clean!
        
        // RULE: Maximum 2-3 levels of nesting.
        // If you're going deeper → extract to methods or use guard clauses.
    }
}
```

---
## 3. Common if Patterns in Real Backend Code
```java
public class RealWorldIfPatterns {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // PATTERN 1: Null check before use
        // (The most common pattern in Java)
        // ─────────────────────────────────────────
        
        String email = null; // simulate missing email
        
        if (email != null && !email.isEmpty()) {
            System.out.println("Email: " + email.toLowerCase());
        } else {
            System.out.println("Email not provided");
        }
        
        // ─────────────────────────────────────────
        // PATTERN 2: Range check
        // ─────────────────────────────────────────
        
        int port = 8080;
        
        if (port >= 1 && port <= 65535) {
            System.out.println("Valid port: " + port);
        } else {
            System.out.println("Invalid port number");
        }
        
        // ─────────────────────────────────────────
        // PATTERN 3: Collection null + empty check
        // ─────────────────────────────────────────
        
        java.util.List<String> items = new java.util.ArrayList<>();
        
        if (items == null || items.isEmpty()) {
            System.out.println("No items to process");
        } else {
            System.out.println("Processing " + items.size() + " items");
        }
        
        // ─────────────────────────────────────────
        // PATTERN 4: Feature flag / toggle
        // ─────────────────────────────────────────
        
        boolean darkModeEnabled = true;
        boolean betaFeaturesEnabled = false;
        
        String theme = darkModeEnabled ? "dark" : "light";
        System.out.println("Theme: " + theme);
        
        if (betaFeaturesEnabled) {
            System.out.println("Beta features: ON");
        }
        
        // ─────────────────────────────────────────
        // PATTERN 5: Business rule validation
        // ─────────────────────────────────────────
        
        double loanAmount = 500_000;
        int creditScore = 720;
        double monthlyIncome = 75_000;
        int employmentYears = 2;
        boolean hasExistingLoan = false;
        
        boolean isEligible = true;
        String rejectionReason = "";
        
        if (loanAmount > monthlyIncome * 60) {
            isEligible = false;
            rejectionReason = "Loan amount exceeds 60x monthly income";
        } else if (creditScore < 650) {
            isEligible = false;
            rejectionReason = "Credit score below minimum (650)";
        } else if (employmentYears < 1) {
            isEligible = false;
            rejectionReason = "Minimum 1 year employment required";
        } else if (hasExistingLoan) {
            isEligible = false;
            rejectionReason = "Cannot have existing active loan";
        }
        
        if (isEligible) {
            System.out.println("Loan APPROVED: ৳" + loanAmount);
        } else {
            System.out.println("Loan REJECTED: " + rejectionReason);
        }
        // Loan APPROVED: ৳500000.0
        
        // ─────────────────────────────────────────
        // PATTERN 6: Checking String values
        // (DON'T use == for strings)
        // ─────────────────────────────────────────
        
        String orderStatus = "PROCESSING";
        
        // ❌ Wrong:
        if (orderStatus == "DELIVERED") { // unreliable!
            System.out.println("Order delivered");
        }
        
        // ✅ Correct:
        if (orderStatus.equals("DELIVERED")) {
            System.out.println("Order delivered");
        } else if (orderStatus.equals("PROCESSING")) {
            System.out.println("Order is being processed"); // ← this
        } else if (orderStatus.equals("CANCELLED")) {
            System.out.println("Order was cancelled");
        }
        
        // ✅ Even better: put the literal FIRST (prevents NPE if var is null)
        if ("DELIVERED".equals(orderStatus)) { // safe even if orderStatus is null
            System.out.println("Order delivered");
        }
        
        // ─────────────────────────────────────────
        // PATTERN 7: Multiple conditions same variable
        // ─────────────────────────────────────────
        
        String role = "MANAGER";
        
        // Verbose:
        if (role.equals("ADMIN") || role.equals("MANAGER") || role.equals("SUPERVISOR")) {
            System.out.println("Has management access");
        }
        
        // Cleaner with List.of().contains():
        java.util.List<String> managementRoles = java.util.List.of("ADMIN", "MANAGER", "SUPERVISOR");
        if (managementRoles.contains(role)) {
            System.out.println("Has management access"); // same result
        }
    }
}
```

---
## 4. switch Statement - Traditional
```java
public class SwitchStatement {
    public static void main(String[] args) {
        
        // switch is an alternative to long if-else if chains
        // when you're checking ONE variable against MULTIPLE specific values
        
        // SYNTAX:
        // switch (variable) {
        //     case value1:
        //         // code
        //         break;
        //     case value2:
        //         // code
        //         break;
        //     default:
        //         // code (runs if no case matched)
        //         break;
        // }
        
        // WHAT CAN BE SWITCHED ON:
        // ✅ byte, short, int, char
        // ✅ String (Java 7+)
        // ✅ Enum
        // ✅ Wrapper types: Byte, Short, Integer, Character
        // ❌ boolean, long, float, double (not allowed)
        
        // ─────────────────────────────────────────
        // EXAMPLE 1: Day of week
        // ─────────────────────────────────────────
        
        int dayNumber = 3; // 1=Monday, 2=Tuesday, etc.
        String dayName;
        
        switch (dayNumber) {
            case 1:
                dayName = "Monday";
                break;
            case 2:
                dayName = "Tuesday";
                break;
            case 3:
                dayName = "Wednesday"; // ← matched
                break;
            case 4:
                dayName = "Thursday";
                break;
            case 5:
                dayName = "Friday";
                break;
            case 6:
                dayName = "Saturday";
                break;
            case 7:
                dayName = "Sunday";
                break;
            default:
                dayName = "Invalid day";
                break;
        }
        
        System.out.println("Day: " + dayName); // Day: Wednesday
        
        // ─────────────────────────────────────────
        // THE BREAK KEYWORD — CRITICAL
        // ─────────────────────────────────────────
        
        // WITHOUT break: FALL-THROUGH happens
        // After matching case, execution CONTINUES into next cases
        // until a break is found or switch ends
        
        int num = 2;
        
        System.out.println("--- Without break (fall-through) ---");
        switch (num) {
            case 1:
                System.out.println("One");
            case 2:
                System.out.println("Two");   // ← matched, starts here
            case 3:
                System.out.println("Three"); // ← falls through (no break after case 2)
            case 4:
                System.out.println("Four");  // ← falls through
            default:
                System.out.println("Default"); // ← falls through
        }
        // Output:
        // Two
        // Three
        // Four
        // Default
        // This is usually a BUG, but sometimes intentional (see below)
        
        System.out.println("--- With break (normal) ---");
        switch (num) {
            case 1:
                System.out.println("One");
                break;
            case 2:
                System.out.println("Two");   // ← matched
                break;                       // ← stops here
            case 3:
                System.out.println("Three");
                break;
            default:
                System.out.println("Default");
                break;
        }
        // Output: Two
        
        // ─────────────────────────────────────────
        // INTENTIONAL FALL-THROUGH — Grouping cases
        // ─────────────────────────────────────────
        
        String day = "SATURDAY";
        
        switch (day) {
            case "MONDAY":
            case "TUESDAY":
            case "WEDNESDAY":
            case "THURSDAY":
            case "FRIDAY":
                System.out.println("Weekday - work day"); // all weekdays reach this
                break;
            case "SATURDAY":
            case "SUNDAY":
                System.out.println("Weekend - rest day"); // ← SATURDAY matches
                break;
            default:
                System.out.println("Invalid day");
        }
        // Output: Weekend - rest day
        
        // ─────────────────────────────────────────
        // EXAMPLE 2: HTTP Method routing
        // (Simplified version of Spring Boot routing)
        // ─────────────────────────────────────────
        
        String httpMethod = "POST";
        String endpoint = "/api/users";
        
        switch (httpMethod) {
            case "GET":
                System.out.println("Fetching resource from " + endpoint);
                break;
            case "POST":
                System.out.println("Creating resource at " + endpoint); // ←
                break;
            case "PUT":
                System.out.println("Updating resource at " + endpoint);
                break;
            case "PATCH":
                System.out.println("Partially updating resource at " + endpoint);
                break;
            case "DELETE":
                System.out.println("Deleting resource at " + endpoint);
                break;
            default:
                System.out.println("Unsupported HTTP method: " + httpMethod);
        }
        
        // ─────────────────────────────────────────
        // EXAMPLE 3: switch with Enum (very common)
        // ─────────────────────────────────────────
        
        OrderStatus status = OrderStatus.PROCESSING;
        
        switch (status) {
            case PENDING:
                System.out.println("Order received, awaiting confirmation");
                break;
            case PROCESSING:
                System.out.println("Order is being prepared"); // ←
                break;
            case SHIPPED:
                System.out.println("Order is on the way");
                break;
            case DELIVERED:
                System.out.println("Order delivered successfully");
                break;
            case CANCELLED:
                System.out.println("Order was cancelled");
                break;
            default:
                System.out.println("Unknown status");
        }
    }
}

enum OrderStatus {
    PENDING, PROCESSING, SHIPPED, DELIVERED, CANCELLED
}
```

---
## 5. switch Expression - Modern Java (Java 14+)
```java
public class SwitchExpression {
    public static void main(String[] args) {
        
        // Java 14+ introduced the SWITCH EXPRESSION
        // Key improvements over traditional switch statement:
        //   ✅ Returns a VALUE (can be used in assignment)
        //   ✅ Arrow syntax (->) eliminates fall-through and break
        //   ✅ Multiple labels in one case
        //   ✅ Exhaustiveness check (compiler ensures all cases covered)
        //   ✅ Much cleaner and less error-prone
        
        // ─────────────────────────────────────────
        // SYNTAX 1: Arrow (->)  — most common
        // ─────────────────────────────────────────
        
        int dayNum = 3;
        
        // Old way (switch statement):
        String dayOld;
        switch (dayNum) {
            case 1: dayOld = "Monday"; break;
            case 2: dayOld = "Tuesday"; break;
            case 3: dayOld = "Wednesday"; break;
            default: dayOld = "Other";
        }
        
        // New way (switch expression with ->):
        String dayNew = switch (dayNum) {
            case 1 -> "Monday";
            case 2 -> "Tuesday";
            case 3 -> "Wednesday"; // ← matched, returns this value
            case 4 -> "Thursday";
            case 5 -> "Friday";
            case 6 -> "Saturday";
            case 7 -> "Sunday";
            default -> "Invalid day";
        }; // note the semicolon — it's an expression/assignment
        
        System.out.println(dayNew); // Wednesday
        
        // ─────────────────────────────────────────
        // MULTIPLE LABELS PER CASE
        // ─────────────────────────────────────────
        
        String day = "SATURDAY";
        String dayType = switch (day) {
            case "MONDAY", "TUESDAY", "WEDNESDAY",
                 "THURSDAY", "FRIDAY"   -> "Weekday";
            case "SATURDAY", "SUNDAY"   -> "Weekend"; // ← matched
            default                     -> "Unknown";
        };
        
        System.out.println(day + " is a " + dayType); // SATURDAY is a Weekend
        
        // ─────────────────────────────────────────
        // WITH ENUM (very clean)
        // ─────────────────────────────────────────
        
        OrderStatus status = OrderStatus.SHIPPED;
        
        String message = switch (status) {
            case PENDING    -> "Awaiting confirmation";
            case PROCESSING -> "Being prepared";
            case SHIPPED    -> "On the way to you!"; // ←
            case DELIVERED  -> "Successfully delivered";
            case CANCELLED  -> "This order was cancelled";
            // No default needed! Compiler checks ALL enum values covered.
            // If you add a new enum value and forget this switch → compile error!
            // This is a HUGE safety advantage over if-else chains.
        };
        
        System.out.println(message); // On the way to you!
        
        // ─────────────────────────────────────────
        // YIELD — returning value from a block
        // ─────────────────────────────────────────
        
        // Sometimes you need multiple lines in a case branch
        // Use yield to return the value from a block:
        
        int score = 85;
        String grade = switch (score / 10) { // integer division
            case 10, 9 -> "A";
            case 8     -> {
                // multiple lines — need yield to return value
                System.out.println("Good score in the 80s!");
                yield "B"; // yield = return value from this switch block
            }
            case 7     -> "C";
            case 6     -> "D";
            default    -> {
                System.out.println("Score needs improvement.");
                yield "F";
            }
        };
        
        System.out.println("Grade: " + grade); // Grade: B
        // Also prints: Good score in the 80s!
        
        // ─────────────────────────────────────────
        // SWITCH EXPRESSION FOR COMPLEX MAPPINGS
        // ─────────────────────────────────────────
        
        // HTTP status code to category
        int httpStatus = 404;
        
        String category = switch (httpStatus / 100) { // get first digit
            case 1 -> "Informational";
            case 2 -> "Success";
            case 3 -> "Redirection";
            case 4 -> "Client Error";   // ← 404/100 = 4
            case 5 -> "Server Error";
            default -> "Unknown";
        };
        
        System.out.println(httpStatus + " is a " + category + " response");
        // 404 is a Client Error response
        
        // ─────────────────────────────────────────
        // SWITCH AS STATEMENT (with ->, no assignment)
        // ─────────────────────────────────────────
        
        // You can also use switch expression as a STATEMENT (no assignment)
        String role = "ADMIN";
        
        switch (role) {
            case "ADMIN"   -> System.out.println("Full system access");
            case "MANAGER" -> System.out.println("Department access");
            case "STAFF"   -> System.out.println("Limited access");
            default        -> System.out.println("No access");
        }
        // Output: Full system access
    }
}
```

---
## 6. Pattern Matching in switch (Java 21)
```java
public class PatternMatchingSwitch {
    public static void main(String[] args) {
        
        // Java 21: switch can match on TYPES (not just values)
        // This is extremely powerful for working with heterogeneous data
        
        // ─────────────────────────────────────────
        // TYPE PATTERN MATCHING
        // ─────────────────────────────────────────
        
        Object[] data = { "Hello", 42, 3.14, true, null };
        
        for (Object obj : data) {
            String description = switch (obj) {
                case Integer i  -> "Integer: " + i * 2;
                case String s   -> "String of length " + s.length();
                case Double d   -> String.format("Double: %.2f", d);
                case Boolean b  -> "Boolean: " + (b ? "YES" : "NO");
                case null       -> "Null value"; // null case (Java 21)
                default         -> "Unknown type: " + obj.getClass().getSimpleName();
            };
            System.out.println(description);
        }
        // Output:
        // String of length 5
        // Integer: 84
        // Double: 3.14
        // Boolean: YES
        // Null value
        
        // ─────────────────────────────────────────
        // GUARDED PATTERNS (when clause)
        // ─────────────────────────────────────────
        
        // Add conditions to pattern cases with 'when':
        Object value = -5;
        
        String result = switch (value) {
            case Integer i when i > 0  -> "Positive integer: " + i;
            case Integer i when i < 0  -> "Negative integer: " + i; // ←
            case Integer i             -> "Zero";
            case String s              -> "String: " + s;
            default                    -> "Something else";
        };
        
        System.out.println(result); // Negative integer: -5
        
        // This is a preview of the kind of patterns
        // you'll use when working with sealed classes,
        // processing different event types in Kafka,
        // handling different API response types, etc.
    }
}
```

---

## 7. if vs switch - When to Use Which
```text
DECISION GUIDE:
  
USE if-else when:
  ✅ Conditions are RANGES or COMPLEX expressions
     (age >= 18 && age <= 65) — not just single values
  ✅ Checking different variables in different branches
  ✅ Complex boolean logic (&&, ||, !)
  ✅ Only 2-3 branches (if/else or if/else-if)
  ✅ Checking for null
  
USE switch when:
  ✅ Comparing ONE variable against MANY specific VALUES
  ✅ Working with Enum types (switch expression is perfect)
  ✅ Working with String status codes, HTTP methods, roles
  ✅ Many (4+) branches all checking same variable
  ✅ You need compiler exhaustiveness check (switch on Enum)
  ✅ Modern Java — prefer switch expression over statement
  
PREFER switch EXPRESSION (Java 14+) over switch STATEMENT:
  ✅ No fall-through bugs (arrow syntax prevents it)
  ✅ Can return a value directly
  ✅ Cleaner, less code
  ✅ Compiler catches missing cases for Enum
  ✅ More modern and readable
```

**Comparison:**
```java
// if-else (always valid):
if (status.equals("PENDING")) {
    // ...
} else if (status.equals("PROCESSING")) {
    // ...
} else if (status.equals("SHIPPED")) {
    // ...
}

// switch statement (old way — avoid new code):
switch (status) {
    case "PENDING": /* ... */ break;
    case "PROCESSING": /* ... */ break;
    case "SHIPPED": /* ... */ break;
    default: /* ... */ break;
}

// switch expression (modern — PREFER THIS):
String message = switch (status) {
    case "PENDING"     -> "Awaiting";
    case "PROCESSING"  -> "Preparing";
    case "SHIPPED"     -> "Shipped";
    default            -> "Unknown";
};
```

```text
WHICH ONE IN YOUR REAL SPRING BOOT CODE?
  Enum status fields → switch expression (exhaustive!)
  String matching → either (switch expression cleaner for many cases)
  Range conditions → always if-else (switch can't do ranges)
  Complex conditions → always if-else
```

---
## Build This - Complete Control Flow Practice
```java
// File: OnlineStoreDecisions.java
// An online store system using all control flow concepts

public class OnlineStoreDecisions {
    
    // ─────────────────────────────────────────
    // CONSTANTS
    // ─────────────────────────────────────────
    static final double FREE_SHIPPING_THRESHOLD = 1000.0; // BDT
    static final double EXPRESS_SURCHARGE       = 150.0;
    static final double PREMIUM_DISCOUNT_RATE   = 0.10;
    static final int    MAX_QUANTITY            = 100;
    
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // ORDER 1: Regular customer, normal delivery
        // ─────────────────────────────────────────
        processOrder(
            "Karim",     // name
            false,       // isPremium
            "STANDARD",  // deliveryType
            3,           // quantity
            350.0,       // unitPrice BDT
            "VISA"       // paymentMethod
        );
        
        System.out.println("\n" + "═".repeat(50) + "\n");
        
        // ─────────────────────────────────────────
        // ORDER 2: Premium customer, express delivery
        // ─────────────────────────────────────────
        processOrder(
            "Rahim",
            true,
            "EXPRESS",
            1,
            800.0,
            "BKASH"
        );
        
        System.out.println("\n" + "═".repeat(50) + "\n");
        
        // ─────────────────────────────────────────
        // ORDER 3: Invalid order
        // ─────────────────────────────────────────
        processOrder(
            "Hasan",
            false,
            "DRONE",     // invalid delivery type
            0,           // invalid quantity
            -50.0,       // invalid price
            "CRYPTO"     // unsupported payment
        );
    }
    
    static void processOrder(String customerName,
                              boolean isPremiumMember,
                              String deliveryType,
                              int quantity,
                              double unitPrice,
                              String paymentMethod) {
        
        System.out.println("Processing order for: " + customerName);
        System.out.println("─".repeat(40));
        
        // ─────────────────────────────────────────
        // STEP 1: Validate inputs (guard clauses)
        // ─────────────────────────────────────────
        
        if (customerName == null || customerName.isBlank()) {
            System.out.println("❌ ERROR: Customer name is required.");
            return;
        }
        
        if (quantity <= 0 || quantity > MAX_QUANTITY) {
            System.out.printf("❌ ERROR: Quantity must be 1-%d. Got: %d%n",
                              MAX_QUANTITY, quantity);
            return;
        }
        
        if (unitPrice <= 0) {
            System.out.println("❌ ERROR: Unit price must be positive.");
            return;
        }
        
        // ─────────────────────────────────────────
        // STEP 2: Validate payment method
        // ─────────────────────────────────────────
        
        java.util.List<String> validPayments = 
            java.util.List.of("VISA", "MASTERCARD", "BKASH", "NAGAD", "COD");
        
        if (!validPayments.contains(paymentMethod)) {
            System.out.println("❌ ERROR: Unsupported payment: " + paymentMethod);
            System.out.println("   Accepted: " + String.join(", ", validPayments));
            return;
        }
        
        // ─────────────────────────────────────────
        // STEP 3: Calculate pricing
        // ─────────────────────────────────────────
        
        double subtotal = unitPrice * quantity;
        
        // Premium discount
        double discountAmount = isPremiumMember ? subtotal * PREMIUM_DISCOUNT_RATE : 0;
        double afterDiscount  = subtotal - discountAmount;
        
        // Delivery fee using switch expression
        double deliveryFee = switch (deliveryType) {
            case "STANDARD" -> afterDiscount >= FREE_SHIPPING_THRESHOLD ? 0 : 60.0;
            case "EXPRESS"  -> 60.0 + EXPRESS_SURCHARGE;
            case "PICKUP"   -> 0.0;
            default         -> {
                System.out.println("⚠️  Unknown delivery type: " + deliveryType);
                yield -1.0; // sentinel value for invalid
            }
        };
        
        if (deliveryFee < 0) {
            System.out.println("❌ ERROR: Invalid delivery type.");
            return;
        }
        
        double totalAmount = afterDiscount + deliveryFee;
        
        // ─────────────────────────────────────────
        // STEP 4: Payment processing info
        // ─────────────────────────────────────────
        
        String paymentInfo = switch (paymentMethod) {
            case "VISA", "MASTERCARD" -> "Card ending in ****";
            case "BKASH"              -> "bKash mobile payment";
            case "NAGAD"              -> "Nagad digital wallet";
            case "COD"                -> "Cash on delivery";
            default                   -> "Unknown"; // won't reach here (validated above)
        };
        
        // ─────────────────────────────────────────
        // STEP 5: Estimated delivery days
        // ─────────────────────────────────────────
        
        int deliveryDays = switch (deliveryType) {
            case "EXPRESS"  -> 1;
            case "STANDARD" -> {
                // Business logic: weekends don't count
                // Simplified version
                if (quantity > 10) {
                    yield 5; // bulk orders take longer
                } else {
                    yield 3;
                }
            }
            case "PICKUP"   -> 0;
            default         -> -1;
        };
        
        // ─────────────────────────────────────────
        // STEP 6: Order summary
        // ─────────────────────────────────────────
        
        String memberStatus = isPremiumMember ? "⭐ Premium" : "Regular";
        
        System.out.println("✅ ORDER DETAILS:");
        System.out.printf("   Customer     : %s (%s)%n", customerName, memberStatus);
        System.out.printf("   Quantity     : %d × ৳%.2f%n", quantity, unitPrice);
        System.out.printf("   Subtotal     : ৳%.2f%n", subtotal);
        
        if (isPremiumMember) {
            System.out.printf("   Discount(10%%): -৳%.2f%n", discountAmount);
        }
        
        System.out.printf("   Delivery     : %s", deliveryType);
        if (deliveryFee == 0 && deliveryType.equals("STANDARD")) {
            System.out.print(" (FREE - above threshold!)");
        }
        System.out.printf(" ৳%.2f%n", deliveryFee);
        
        System.out.printf("   ─────────────────────────────%n");
        System.out.printf("   TOTAL        : ৳%.2f%n", totalAmount);
        System.out.printf("   Payment      : %s%n", paymentInfo);
        System.out.printf("   Estimated    : ");
        
        if (deliveryDays == 0) {
            System.out.println("Ready for pickup now");
        } else if (deliveryDays == 1) {
            System.out.println("Tomorrow");
        } else {
            System.out.println(deliveryDays + " business days");
        }
    }
}
```

---
## Exercises
```text
EXERCISE 1: Temperature Advisory System
  Create TempAdvisory.java
  Given a temperature (double, Celsius):
  - Below 0:   "Freezing - wear heavy coat"
  - 0-10:      "Very cold - wear coat"
  - 10-20:     "Cool - wear jacket"
  - 20-30:     "Comfortable - light clothing"
  - 30-40:     "Hot - stay hydrated"
  - Above 40:  "Extreme heat - stay indoors"
  Also: if temp > 35, add "⚠️ Heat warning"
  Also: if temp < 0, add "⚠️ Frost warning"
  Print appropriate message.
  
EXERCISE 2: ATM Machine Simulator
  Create ATMSimulator.java
  Use if-else for all decisions:
  - User has: boolean isCardInserted, String pin, double balance
  - Correct PIN is "1234"
  - Operations: check balance, withdraw, deposit
  - For withdrawal:
    if !isCardInserted → "Please insert card"
    if wrong pin → "Wrong PIN"
    if amount <= 0 → "Invalid amount"
    if amount > balance → "Insufficient funds"
    if amount > 25000 → "Daily limit exceeded (max ৳25,000)"
    if amount % 100 != 0 → "Amount must be multiple of 100"
    else → approve and update balance
  Test 6 different scenarios.
  
EXERCISE 3: switch Expression Practice
  Create PaymentRouter.java
  Using ONLY switch expressions (no if-else):
  - Map payment method to: processing time, max limit, fee %
    BKASH:      instant,  ৳25000,  1.5%
    NAGAD:      instant,  ৳20000,  1.5%
    VISA:       1-2 days, ৳500000, 2.5%
    MASTERCARD: 1-2 days, ৳500000, 2.5%
    COD:        on delivery, ৳10000, 0%
    BANK_TRANSFER: 3 days, unlimited, 0.5%
  Print payment routing info for each method.
  
EXERCISE 4: Grade Report
  Create GradeReport.java
  Given 5 subject scores (hardcoded):
  - Calculate average
  - Determine grade using if-else (A/B/C/D/F)
  - Determine CGPA equivalent (4.0 scale)
  - Determine scholarship eligibility:
    GPA >= 3.75: Full scholarship
    GPA >= 3.5: Partial scholarship (50%)
    GPA >= 3.0: Tuition waiver (25%)
    Below 3.0: No scholarship
  - Print detailed report
  Push to GitHub with commit: "feat: add grade report system"
  
EXERCISE 5: Predict the output (no running first!)
  What is the output of each snippet?
  Write your answer THEN run to verify.
  
  a) int x = 5;
     if (x > 3)
         System.out.println("A");
         System.out.println("B");
  
  b) int n = 2;
     switch (n) {
         case 1: System.out.println("One");
         case 2: System.out.println("Two");
         case 3: System.out.println("Three");
         default: System.out.println("Default");
     }
  
  c) String s = switch (3) {
         case 1, 2 -> "low";
         case 3, 4 -> "mid";
         default   -> "high";
     };
     System.out.println(s);
  
  d) Object obj = "test";
     if (obj instanceof String str && str.length() > 3) {
         System.out.println("Long string: " + str);
     }
```

---
## Common Mistakes
```text
MISTAKE 1: Assignment instead of comparison in condition
  if (x = 5) { ... }   // COMPILE ERROR in Java (unlike C/C++)
  if (x == 5) { ... }  // ✓ correct
  Java saves you here — won't compile.
  
MISTAKE 2: Missing break in switch statement
  switch (day) {
      case 1: System.out.println("Mon");  // no break!
      case 2: System.out.println("Tue");  // falls through
  }
  // Both print if day == 1
  Fix: add break; or use switch expression (->)
  
MISTAKE 3: Wrong order in if-else if
  if (score >= 60) { grade = "D"; }      // catches everything >= 60
  else if (score >= 90) { grade = "A"; } // NEVER reached
  Fix: most specific first: 90, 80, 70, 60
  
MISTAKE 4: No braces on single-line if
  if (x > 0)
      doThis();
      alsoThis(); // NOT in the if block! Always runs!
  Fix: ALWAYS use braces {}
  
MISTAKE 5: Using == for String in switch... wait, actually:
  switch on String uses .equals() internally — that's fine
  But never use == to compare String outside switch
  
MISTAKE 6: switch without default
  switch (status) {
      case "ACTIVE": ... break;
      case "INACTIVE": ... break;
      // What if status is "PENDING"? Silent! Nothing happens.
  }
  Always have default unless using switch expression on Enum
  (Enum switch expression is exhaustiveness-checked)
  
MISTAKE 7: Null in switch (traditional)
  String s = null;
  switch (s) { ... } // NullPointerException at runtime!
  Fix: null-check before switch, or use switch with null case (Java 21)
  
MISTAKE 8: Forgetting yield in switch expression block
  String result = switch (x) {
      case 1 -> {
          System.out.println("One");
          "one"; // COMPILE ERROR: needs yield
      }
  };
  Fix: yield "one";
  
MISTAKE 9: Comparing with literals the wrong way
  if (orderStatus.equals("ACTIVE")) // NPE if orderStatus is null!
  Fix: if ("ACTIVE".equals(orderStatus)) // safe, literal can't be null
  
MISTAKE 10: Deep nesting instead of guard clauses
  if (isLoggedIn) {
      if (isVerified) {
          if (hasPermission) {
              // actual logic buried deep
          }
      }
  }
  Fix: guard clauses (early returns for failure cases)
```

---
## Interview Questions

**Q: What is the difference between if-else and switch?**
A: if-else evaluates boolean expressions and can handle ranges, complex conditions, and multiple variables. switch compares one variable against specific constant values and is cleaner when you have many branches checking the same variable. Modern switch expressions (Java 14+) return values, use arrow syntax to prevent fall-through, and provide exhaustiveness checks for enums. For ranges or complex conditions, use if-else. For many specific value matches, prefer switch expression.

**Q: What is fall-through in a switch statement and how do you prevent it?**
A: Fall-through occurs when a matched case doesn't have a break statement — execution continues into the next case(s) until a break or end of switch is reached. It's usually a bug. Prevention: always add break at the end of each case, or better — use the modern switch expression with arrow syntax (->) which has NO fall-through by design. Intentional fall-through is sometimes used to group cases that share the same action.

**Q: What is a guard clause and why is it preferred over nested if statements?**
A: A guard clause is an early return statement that handles an error or edge case at the top of a method, before the main logic. Instead of wrapping the main logic in nested ifs, you check for invalid conditions first and return immediately. This inverts the condition — instead of if (valid) { logic } you write if (!valid) return; then logic. It reduces nesting, makes the happy path obvious, and makes code easier to read and maintain. Used heavily in production Spring Boot code for input validation.

**Q: Can you use a switch on a String in Java?**
A: Yes, since Java 7. switch on String uses .equals() internally for comparison, so it's content-based. However, if the switch variable is null, it throws NullPointerException — always null-check before switching on a String. In Java 21, you can handle null explicitly with a null case in switch expressions.

**Q: What is pattern matching for switch in Java 21?**
A: Java 21 extended switch to support type patterns — you can match on the TYPE of an object, not just its value. Combined with when guards, you can write: case Integer i when i > 0 -> which matches integers greater than zero. This replaces verbose instanceof chains and makes code much cleaner when processing heterogeneous data (like different event types in a Kafka consumer or API responses with different structures).

**Q: When would you choose an if-else chain over a switch expression?**
A: Use if-else when: conditions involve ranges (score >= 90), different variables are checked in different branches, complex boolean logic is needed (&&, ||), or there are only 2-3 branches. Use switch expression when: comparing one variable against many specific values, working with enums (where compiler checks exhaustiveness), many branches all checking the same variable, or you want a clean value-returning expression.

---
## Key Takeaways

```text
1. if-else controls execution path based on boolean conditions.
   if → else if → else: only ONE block runs, always.
   Order conditions MOST SPECIFIC to LEAST SPECIFIC.

2. ALWAYS use braces {} even for single-line if bodies.
   Omitting braces = future bug waiting to happen.

3. Guard clauses (early returns) beat deep nesting.
   Handle error cases first, happy path last.
   Max 2-3 levels of nesting in real code.

4. Use "literal".equals(variable) not variable.equals("literal")
   Literals can never be null — prevents NullPointerException.

5. Traditional switch: needs break to prevent fall-through.
   Missing break = fall-through = almost always a bug.
   Default case handles unmatched values — always include it.

6. switch EXPRESSION (Java 14+) is the modern standard:
   → Arrow syntax (->) prevents fall-through automatically
   → Returns a value (can be used in assignment)
   → Multiple labels: case "A", "B" -> ...
   → yield to return from a block case
   → Exhaustiveness check for Enum (no default needed)

7. Short-circuit in if: obj != null && obj.method()
   If obj is null, the method is never called.
   This is the standard null-safety pattern in Java.

8. switch on Enum with switch expression = safest combo.
   Compiler tells you if you forgot a case.
   Add new enum value → all switches need updating → compile error.

9. Pattern matching switch (Java 21):
   Match on types + conditions in one expression.
   Replaces instanceof chains.

10. if vs switch decision:
    Ranges/complex conditions → if-else
    Many specific values, one variable → switch expression
    Enum states → switch expression (always)
```

---