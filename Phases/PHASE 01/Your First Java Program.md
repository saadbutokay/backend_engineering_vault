**Phase:** Level 1 - Java Fundamentals  
**Date Studied:** 31st July, 2026

---
## What Problem Does This Solve?
You understand HOW Java works (Level 1.1).  
Now you need to understand what you're actually WRITING.

Most beginners type their first program without understanding:
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```
They just copy it. It works. They move on.

Six months later in an interview:  
- "What does `static` mean in the main method?"  
- "Why is main `void`?"  
- "What is `String[] args`?"

Silence.
This note explains EVERY word in your first program. Not just what to type - but WHY every single piece exists. When you finish this note, you will understand every character in that program completely.

---
## The Full First Program
Every Word Explained.
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```
Let's break this into layers.

---

## Layer 1 - The Class
```java
public class Main {
    // everything inside here
}
```

**WHAT IT IS:**
- The fundamental building block of Java.
- Everything in Java lives inside a class.
- Unlike Python or JavaScript where you can write
- code outside of any class - in Java you CANNOT.
- There is no such thing as "code outside a class" in Java.
- Every line of executable code must be inside a class.

**WHAT IT DOES:**
- A class is a blueprint.
- It defines what something IS and what it CAN DO.
- Right now, Main is just a container for our program.
- Later, classes will represent real things:
    - class User { ... }        → blueprint for a user
    - class BankAccount { ... } → blueprint for an account
    - class Order { ... }       → blueprint for an order

```
SYNTAX RULE:
  class keyword → space → ClassName → opening brace {
  Everything inside → closing brace }
  
  The brace { opens a "block" of code.
  Everything inside belongs to this class.
  The brace } closes it.
  Every opening { needs a closing }.
  This is the #1 cause of syntax errors for beginners.
```

---
### `public` (before class)

```
WHAT IT IS:
  An access modifier.
  Controls who can SEE and USE this class.

public means:
  This class is visible to EVERYONE.
  Any other class in any package can use it.
  
  Java has 4 access levels (we'll cover all in Level 1.11):
  public    → everyone can access
  protected → same package + subclasses
  (default) → same package only
  private   → only this class

WHY Main IS public:
  The JVM needs to find and run your main method.
  If Main wasn't public, the JVM couldn't access it
  from outside your package.
  
  RULE FOR NOW:
  Every class you write that contains the main method
  should be public.
  Every class definition that goes in its own file
  should be public (and filename must match).
```

---
### `Main` (the class name)

```
WHAT IT IS:
  The name you give your class.
  
NAMING RULES (enforced by Java — breaks if violated):
  → Must start with a letter, $ or _ (not a number)
  → Can contain letters, numbers, $, _
  → Cannot be a Java keyword (class, public, void, etc.)
  → Case sensitive (Main ≠ main ≠ MAIN)

NAMING CONVENTIONS (not enforced, but ALWAYS follow):
  → PascalCase: every word starts with capital
    ✅ Main, UserService, BankAccount, OrderController
    ❌ main, userservice, bankaccount, ordercontroller
    ❌ user_service, bank-account (don't use these)
  
  → Should be a NOUN (what is this thing?)
    ✅ User, Product, OrderService, PaymentProcessor
    ❌ DoStuff, RunThis (bad names)
  
  → Should be descriptive, not abbreviated
    ✅ UserAuthentication
    ❌ UsrAuth, UA

FILE NAME RULE:
  If the class is public:
  File MUST be named exactly: ClassName.java
  public class Main → file must be: Main.java
  public class UserService → file must be: UserService.java
  Case sensitive. Main.java ≠ main.java
```

---
### The Braces `{ }`
WHAT THEY ARE:
- Block delimiters. They define a "scope" or "block."
- Everything between { and } belongs to that block.

IN JAVA:
```java
  class body → { }
  method body → { }
  if block → { }
  for loop → { }
  try block → { }
```

```
EVERY { MUST HAVE A MATCHING }
  This is the most common beginner syntax error.
  
  IntelliJ helps: when you type {, it auto-types }
  Also shows matching braces when you click on one.
```

INDENTATION is your visual guide:
```java
  public class Main {           // opens class block
      public static void main   // inside class
          (String[] args) {     // opens method block
          System.out.println(); // inside method
      }                         // closes method block
  }                             // closes class block
```
4 spaces per indent level (Java standard). IntelliJ does this automatically. Ctrl+Alt+L reformats code to correct indentation.


---
## Layer 2 - The main Method
```java
public static void main(String[] args) {
    // your code goes here
}
```
This is the **entry point** of every Java application. When you run `java Main`, the JVM looks for EXACTLY this method. Not just any method called "main" - THIS specific signature.

Let's understand every word:
### `public` (before main)

```
Same access modifier concept as for the class.

WHY main MUST BE public:
  The JVM calls main from OUTSIDE your class.
  If main were private, the JVM couldn't call it.
  If main were package-private, the JVM couldn't call it.
  
  It MUST be public so the JVM can reach it.
  
  Try making it private → you'll get:
  "Error: Main method not found in class Main,
   please define the main method as:
   public static void main(String[] args)"
  
  The JVM is very specific about this.
```

---
### `static`
WHAT IT MEANS:
- This method belongs to the CLASS itself.
- Not to any specific OBJECT (instance) of the class.

WITHOUT static:
```java
  // To call a regular method, you need an object:
  Main obj = new Main();  // create object first
  obj.someMethod();       // then call method on it
```

WITH static:
```java
  // You call it on the class itself, no object needed:
  Main.main(args);        // directly on the class
```

WHY main MUST BE static:
- Think about it: When the JVM starts running your program, NO objects exist yet. Nothing has been created. The JVM can't create a Main object to call main on it because it doesn't even know how to do that yet.

```  
  The JVM needs a starting point BEFORE any objects exist.
  
  Solution: make main static.
  The JVM calls: Main.main(args)
  No object needed. Directly on the class.
  
  After main starts running, YOU can create objects:
  Main obj = new Main(); // now inside main, this works
  
  This is why:
  static → class level (exists without any object)
  non-static → instance level (needs an object)
```
We'll explore this much more in [[Classes & Objects]].

---
### `void`
WHAT IT MEANS:
- The return type of this method.
- void means: "this method returns NOTHING."

METHODS CAN RETURN VALUES:
```java
  int add(int a, int b) { return a + b; } // returns int
  String getName() { return "Rahim"; }     // returns String
  boolean isAdult(int age) { return age >= 18; } // returns boolean
```

void MEANS NO RETURN:
```java
  void printHello() { System.out.println("Hello"); }
  // prints something but returns nothing
```

```
void methods:
  → Cannot have a return statement with a value
  → Can have bare return; (just exits the method early)
  → Caller cannot use the result (there is none)

WHY main IS void:
  When main finishes, your program ends.
  There's nobody to return a value TO.
  What would the JVM do with it?
  
  Well, actually: the program exit code matters.
  Exit code 0 = success (Java does this automatically)
  Exit code non-zero = error
  
  If you want to exit with a specific code:
  System.exit(0);   // success
  System.exit(1);   // generic error
  System.exit(2);   // specific error code
  
  But normally you just let main return normally (void).
```

---
### `main`
WHAT IT IS:
- The method name.

**WHY "main"?**
Convention that Java requires. The JVM specifically looks for a method called "main". It's not a keyword - it's just the name the JVM searches for.

You can have other methods called main in other classes - but only ONE class is the "entry point" class. You specify which class when running:
  `java Main`  ← this tells JVM: find `main()` in Main class

In Spring Boot, you'll see:
```java
  @SpringBootApplication
  public class MyApp {
      public static void main(String[] args) {
          SpringApplication.run(MyApp.class, args);
      }
  }
  // Same pattern. Same main method. Spring Boot takes over from here.
```

```
NAMING RULES FOR METHODS:
  → camelCase: first word lowercase, subsequent words capitalized
    ✅ main, getUserById, calculateTotalPrice, sendEmail
    ❌ GetUserById, getuserbyid, get_user_by_id
  
  → Should be a VERB (what does this method DO?)
    ✅ calculatePrice(), sendEmail(), validateUser()
    ❌ price(), email(), user() (not verbs)
  
  → Descriptive, not abbreviated
    ✅ calculateTotalOrderPrice()
    ❌ calcTotOrdPrc()
```

---
### `(String[] args)`
WHAT IT IS:
- The parameter list of the main method.
- Parameters = input values that a method receives.

```
BREAKING IT DOWN:
 String[]  → the TYPE: an array of Strings
 args      → the NAME: the variable name for this array
              (short for "arguments" — could be named anything)
              convention is args
```

WHAT IS `String[]`?
- String = text data type (we'll cover fully in [[Strings Deep Dive]])
- `[]` = array (a collection of items of same type)
- `String[]` = an array (list) of String items

```java
// Example String array:
  String[] fruits = {"apple", "banana", "mango"};
  fruits[0] = "apple"   // access by index (starts at 0!)
  fruits[1] = "banana"
  fruits[2] = "mango"
```

WHAT IS `args`?
- When you run a Java program from the command line, you can pass arguments (extra information) to it:
```java
// java Main hello world 123
  
// Inside your program:
  args[0] = "hello"
  args[1] = "world"
  args[2] = "123"
```

```
This is how you pass configuration to a program from the command line.

WHY main NEEDS String[] args:
  The JVM specification says main MUST have this parameter.
  Even if you don't use it, it must be there.
  
  If you write: public static void main() {
  → JVM can't find the main method → error
  
  However in Java 21, there's a new feature (preview):
  Unnamed main methods (JEP 445) — for very simple programs.
  But for now, always write the full signature.
```

```java
// USING args IN PRACTICE:
  public static void main(String[] args) {
      if (args.length > 0) {
          System.out.println("First argument: " + args[0]);
      } else {
          System.out.println("No arguments provided");
      }
  }
```

```
  Run: java Main Dhaka
  Output: First argument: Dhaka
  
  Run: java Main
  Output: No arguments provided
  
  In Spring Boot, args are passed to SpringApplication.run()
  This allows runtime configuration.
```

---
## Layer 3 - Printing Output

```java
System.out.println("Hello, World!");
```
This looks simple. It's actually three nested references.  
Let's understand it fully.

### Breaking Down `System.out.println`

```
System
  → A CLASS in the java.lang package
  → Automatically imported (you don't need to import it)
  → Contains useful utilities for interacting with the system
  → Has fields: in, out, err
  → Has methods: exit(), gc(), currentTimeMillis(), arraycopy()

System.out
  → out is a FIELD (variable) inside System class
  → It is static (accessed on the class, not an object)
  → Its type is: PrintStream
  → It represents STANDARD OUTPUT (your terminal/console)
  → This is the object we call methods on

System.out.println()
  → println is a METHOD on the PrintStream object
  → print + ln = print then new Line
  → Prints the content AND moves to next line

The DOT (.) operator:
  dot means "access something inside"
  System.out = access out field inside System
  System.out.println = access println method on that object
  
  Reading left to right:
  "Get System, get its out field, call println on that"
```

---
### Output Methods - All Variations
```java
public class OutputDemo {
    public static void main(String[] args) {
        
        // println: prints + new line (moves cursor to next line)
        System.out.println("Line 1");
        System.out.println("Line 2");
        // Output:
        // Line 1
        // Line 2
        
        // print: prints WITHOUT new line
        System.out.print("Hello ");
        System.out.print("World");
        System.out.println(); // just a newline, no content
        // Output:
        // Hello World
        
        // printf: formatted output (like printf in C)
        // %s = String placeholder
        // %d = integer placeholder
        // %f = float/double placeholder
        // %n = newline (platform-independent)
        String name = "Rahim";
        int age = 21;
        double gpa = 3.15;
        System.out.printf("Name: %s, Age: %d, GPA: %.2f%n", 
                           name, age, gpa);
        // Output:
        // Name: Rahim, Age: 21, GPA: 3.15
        // %.2f = float with 2 decimal places
        
        // formatted() - modern Java way (Java 15+)
        String message = "Name: %s, Age: %d".formatted(name, age);
        System.out.println(message);
        // Output:
        // Name: Rahim, Age: 21
        
        // Printing different types
        System.out.println(42);           // int
        System.out.println(3.14);         // double
        System.out.println(true);         // boolean
        System.out.println('A');          // char
        System.out.println(100L);         // long
        
        // Printing expressions
        System.out.println(5 + 3);        // 8
        System.out.println(10 > 5);       // true
        System.out.println("Sum: " + (5 + 3)); // Sum: 8
        
        // Standard ERROR output (shows in red in IntelliJ)
        // Use this for error messages, not normal output
        System.err.println("This is an error message");
        
        // Standard INPUT
        // System.in → used for reading from keyboard
        // We'll use Scanner for this (Level 1.7)
    }
}
```

---
## Comments - Documenting Your Code
```java
public class CommentsDemo {
    
    // This is a single-line comment
    // Everything after // on this line is ignored by compiler
    // Use for short explanations of WHAT or WHY
    
    /*
     * This is a multi-line comment.
     * Can span multiple lines.
     * Everything between the markers is ignored.
     * Use for longer explanations.
     */
    
    /**
     * This is a Javadoc comment.
     * Used to document classes, methods, fields.
     * Special tags add structured information:
     *
     * @param args  command line arguments
     * @return      nothing (void method)
     * @throws      IllegalArgumentException if args is null
     * @author      Your Name
     * @version     1.0
     * @since       2025-01-01
     */
    public static void main(String[] args) {
        
        int x = 5; // inline comment: x holds the count of items
        
        // Good comment: explains WHY (not obvious from code)
        // We add 1 because the API uses 1-based indexing
        int apiIndex = x + 1;
        
        // Bad comment: explains WHAT (obvious from code)
        // Set x to 5   ← bad, anyone can see x = 5
        // Increment i  ← bad, anyone can see i++
        
        // Temporarily disable code during debugging:
        // System.out.println("debug: " + x);
        
        /*
         * This block was removed because:
         * The new validation handles this case.
         * Keeping as reference for 2 weeks then delete.
         */
        // if (x < 0) {
        //     throw new IllegalArgumentException("x cannot be negative");
        // }
        
        System.out.println("Running with x = " + x);
    }
}
```

### When to Write Comments (and When NOT To)
```
WRITE COMMENTS WHEN:
  ✅ Explaining WHY (business reason not obvious from code)
     // We use 15-minute intervals because the payment provider
     // only allows one retry per 15 minutes
  
  ✅ Warning about non-obvious behavior
     // Don't use parallelStream() here — order matters
  
  ✅ TODO notes (but fix them, don't leave forever)
     // TODO: add pagination when user count exceeds 1000
  
  ✅ Explaining complex algorithms
     // Using Floyd's cycle detection algorithm:
     // Fast pointer moves 2 steps, slow moves 1 step
  
  ✅ Javadoc for public APIs (what others will call)

DON'T WRITE COMMENTS WHEN:
  ❌ The code is self-explanatory
     // Get user by ID  ← obvious from: getUserById(id)
     // Add 1 to count ← obvious from: count++
  
  ❌ Describing WHAT code does (code should show this)
     // Create a new user and set their name and email
     // instead: just write clean code that's obvious
  
  ❌ Outdated comments (lies are worse than no comments)
     // Returns the user's email  ← but actually returns username
  
GOLDEN RULE:
  "Good code is self-documenting.
   Comments explain WHY, not WHAT.
   If you need a comment to explain WHAT,
   consider renaming things to make it obvious."
```

---
## Understanding the Semicolon `;`
In Java, every STATEMENT ends with a semicolon `;`. A statement is a complete instruction.
```java
// Examples of statements (need semicolons):
  int x = 5;
  System.out.println("hello");
  String name = "Rahim";
  x = x + 1;
  return value;
```

```java
// Examples of NOT statements (no semicolons):
  public class Main { ... }       // class declaration
  public static void main() {     // method declaration
  if (condition) {                // if statement header
  for (int i = 0; i < 10; i++) {  // for loop header
  }                               // closing brace
```

```java
// COMMON SEMICOLON ERRORS:
  // Missing semicolon:
    int x = 5        // error: ';' expected
    int x = 5        // error on NEXT line (confusing!)
    System.out.println("hi")
    // Java reads until end → reports error on next line
    // Always look at the line BEFORE the reported error
```

```java
  // Semicolon after if/for/while (logic bug, not syntax error!):
    if (x > 0);       // VALID SYNTAX but almost always a bug!
    {                  // this block runs regardless of condition
        System.out.println("this always prints");
    }
    // The ; after if creates an empty if body
    // The { } block has nothing to do with the if
    // IntelliJ warns about this — pay attention to warnings
```

---
## The Full Anatomy - Putting It All Together
```java
// File: Main.java
// Package declaration (optional for now, required in real projects)
// package com.yourname.firstproject;

// Import statements (bring in other classes)
// java.lang is auto-imported — System, String, Math etc.
// import java.util.Scanner; ← you'll add when needed

/**
 * Entry point for our first Java application.
 * Demonstrates the structure of a Java program.
 */
public class Main {           // ← public class, name = filename
    
    // CLASS BODY starts here
    // Fields (variables) would go here
    // Methods go here
    
    /**
     * JVM entry point. Every Java application starts here.
     * @param args command line arguments (can be empty)
     */
    public static void main(String[] args) {
        // METHOD BODY starts here
        // This is where your program's logic goes
        
        System.out.println("Hello, World!");
        
        // METHOD BODY ends here
    }               // ← closes main method
    
    // CLASS BODY ends here
}                   // ← closes Main class
```

---
## How IntelliJ Helps You Write This
```
IntelliJ shortcuts that save you time from day 1:

CREATING main METHOD FAST:
  Inside a class, type: main
  Press Tab
  IntelliJ expands to full main method signature!
  
  Or type: psvm (public static void main)
  Press Tab → full main method!

CREATING println FAST:
  Type: sout
  Press Tab
  IntelliJ expands to: System.out.println()
  Cursor is placed inside the parentheses.
  
  Other shortcuts:
  soutv → System.out.println("variable = " + variable);
  soutm → prints current method name
  souf  → System.out.printf("")

CREATING CLASS:
  Right-click folder → New → Java Class
  Type class name → Enter
  IntelliJ creates the class with proper structure

CODE COMPLETION:
  Type: Sys
  IntelliJ suggests: System
  Press Enter → accepts suggestion
  
  Type: System.
  IntelliJ shows all fields and methods
  Type: out → narrows to System.out
  Press Enter

ERROR HINTS:
  Red squiggly underline = syntax error
  Yellow underline = warning (not wrong, but suspicious)
  Hover over error → see explanation
  Alt+Enter → quick fix suggestions

REFORMAT CODE:
  Ctrl+Alt+L → fix all indentation and spacing
  Use this constantly. Good formatting is professional habit.
```

---
## Program Structure - Bigger Picture
As your programs grow, here's the full structure:
```java
// 1. Package declaration (first line, if any)
package com.yourname.myapp;

// 2. Import statements
import java.util.List;
import java.util.ArrayList;

// 3. Class declaration
public class UserManager {
    
// 4. Fields (class-level variables)
    private static int totalUsers = 0;  // static field
    private String name;                 // instance field
    
// 5. Constructors (how to create objects of this class)
    public UserManager(String name) {
        this.name = name;
        totalUsers++;
    }
    
// 6. Methods (behaviors)
    public void displayInfo() {
        System.out.println("Manager: " + name);
    }
    
    public static int getTotalUsers() {
        return totalUsers;
    }
    
// 7. main method (only in one class — the entry point)
    public static void main(String[] args) {
        UserManager manager = new UserManager("Rahim");
        manager.displayInfo();
        System.out.println("Total: " + getTotalUsers());
    }
}
```

```
RIGHT NOW you only need:
  → Package (skip for now)
  → Import (we'll add when needed)
  → Class declaration
  → main method
  → println statements inside main

Everything else comes in upcoming levels.
```

---
## Code Examples
Build These...
### Program 1: Hello World (The Professional Version)
```java
// File: Main.java
/**
 * My first Java program.
 * Demonstrates basic Java program structure.
 * 
 * @author Your Name
 */
public class Main {
    
    public static void main(String[] args) {
        // Print a greeting
        System.out.println("Hello, World!");
        System.out.println("I am learning Java.");
        System.out.println("My goal: Backend Engineer.");
    }
}
```

### Program 2: Using Command Line Arguments
```java
// File: Greeter.java
public class Greeter {
    
    public static void main(String[] args) {
        // Check if a name was provided
        if (args.length == 0) {
            System.out.println("Hello, Stranger!");
            System.out.println("Usage: java Greeter <your-name>");
        } else {
            // Use the first argument as the name
            String name = args[0];
            System.out.println("Hello, " + name + "!");
            System.out.println("Welcome to Java.");
        }
        
        // Print all arguments received
        System.out.println("Total arguments received: " + args.length);
        for (int i = 0; i < args.length; i++) {
            System.out.println("Argument " + i + ": " + args[i]);
        }
    }
}

// Run: java Greeter Rahim
// Output:
// Hello, Rahim!
// Welcome to Java.
// Total arguments received: 1
// Argument 0: Rahim

// Run: java Greeter Rahim Backend Engineer
// Output:
// Hello, Rahim!
// Welcome to Java.
// Total arguments received: 3
// Argument 0: Rahim
// Argument 1: Backend
// Argument 2: Engineer
```

### Program 3: Formatted Output
```java
// File: FormattedOutput.java
public class FormattedOutput {
    
    public static void main(String[] args) {
        
        // Your profile as a formatted table
        System.out.println("================================");
        System.out.println("     ENGINEER PROFILE           ");
        System.out.println("================================");
        
        String name     = "Your Name";
        String goal     = "Backend Engineer";
        String language = "Java";
        int    year     = 2;
        double cgpa     = 2.83;
        double target   = 3.15;
        
        // printf for aligned output
        System.out.printf("Name    : %s%n", name);
        System.out.printf("Goal    : %s%n", goal);
        System.out.printf("Language: %s%n", language);
        System.out.printf("Year    : %d%n", year);
        System.out.printf("CGPA    : %.2f%n", cgpa);
        System.out.printf("Target  : %.2f%n", target);
        
        System.out.println("================================");
        System.out.println("Status  : In Progress");
        System.out.println("================================");
    }
}

// Output:
// ================================
//      ENGINEER PROFILE           
// ================================
// Name    : Your Name
// Goal    : Backend Engineer
// Language: Java
// Year    : 2
// CGPA    : 2.83
// Target  : 3.15
// ================================
// Status  : In Progress
// ================================
```

### Program 4: Multiple Classes in One Concept
```java
// File: MultiClassDemo.java

// Second class in same file (not public — only Main is public here)
class Helper {
    // A simple helper class
    static void printSeparator() {
        System.out.println("─────────────────────");
    }
    
    static void printHeader(String title) {
        printSeparator();
        System.out.println("  " + title);
        printSeparator();
    }
}

// Main class (public, matches filename)
public class MultiClassDemo {
    
    public static void main(String[] args) {
        Helper.printHeader("My Java Journey");
        System.out.println("Day 1: Completed setup");
        System.out.println("Day 2: Learned how Java works");
        System.out.println("Day 3: Writing first programs");
        Helper.printSeparator();
        System.out.println("Status: On track!");
    }
}

// IMPORTANT: This file is MultiClassDemo.java
// because MultiClassDemo is the public class
// Helper is a "package-private" class (no access modifier)
// Both classes CAN be in the same file when only one is public
// But in real projects: ONE class per file (always)
```

### Program 5: Understanding Static - Side by Side
```java
// File: StaticDemo.java
public class StaticDemo {
    
    // Static field: belongs to the CLASS
    // ONE copy shared by ALL objects
    static int objectCount = 0;
    
    // Instance field: belongs to each OBJECT
    // Each object has its OWN copy
    String name;
    
    // Constructor (we'll learn this properly in 1.10)
    // For now: it runs when you create a new object
    StaticDemo(String name) {
        this.name = name;
        objectCount++; // increment the shared counter
    }
    
    // Static method: can be called without an object
    static void showCount() {
        System.out.println("Objects created: " + objectCount);
    }
    
    // Instance method: needs an object to call
    void showName() {
        System.out.println("My name is: " + name);
    }
    
    public static void main(String[] args) {
        // Call static method: directly on class
        StaticDemo.showCount(); // Objects created: 0
        
        // Create first object
        StaticDemo obj1 = new StaticDemo("Rahim");
        StaticDemo.showCount(); // Objects created: 1
        obj1.showName();        // My name is: Rahim
        
        // Create second object
        StaticDemo obj2 = new StaticDemo("Karim");
        StaticDemo.showCount(); // Objects created: 2
        obj2.showName();        // My name is: Karim
        
        // objectCount is SHARED — both objects affect it
        // name is SEPARATE — each object has its own
        
        // This is the core difference:
        // static = class level = shared = ONE copy
        // non-static = instance level = per-object = MANY copies
    }
}
```

---
## Exercises
Do These Before Moving On:
```
EXERCISE 1: Modify Hello World
  Add more println statements.
  Print your name, university, goal, today's date.
  Practice formatting with printf.

EXERCISE 2: Experiment with main signature
  a) Remove "public" from main → run → read the error
  b) Remove "static" from main → run → read the error  
  c) Change "void" to "int" → run → what happens?
  d) Remove String[] args → run → read the error
  e) Restore to correct version
  GOAL: Understand WHY each word is required.

EXERCISE 3: Break it and fix it
  Intentionally create these errors, read them, fix them:
  a) Remove a semicolon from System.out.println
  b) Remove the closing } of main method
  c) Remove the closing } of the class
  d) Misspell "class" as "Class" (capital C)
  e) Misspell "println" as "printLn"
  GOAL: Recognize common error messages.

EXERCISE 4: Command line arguments
  a) Copy the Greeter.java program
  b) Run it with no arguments
  c) Run it with your name as argument
  d) Run it with 3 different arguments
  e) Modify it to also print the args in UPPERCASE
     (use args[0].toUpperCase() — we'll learn this properly soon)

EXERCISE 5: Your Engineer Profile
  Build on FormattedOutput.java.
  Add:
  - Your target job company (pick one: bKash, Pathao, etc.)
  - The number of days into your journey
  - Your current topic (Hello World!)
  - A motivational line at the bottom
  Push this to GitHub.
```

---
## Common Mistakes
```
MISTAKE 1: Wrong file name
  Class: public class UserProfile
  File:  UserProfile.java ← correct
  File:  userprofile.java ← wrong (case sensitive!)
  File:  User.java        ← wrong (doesn't match class name)
  
  Error: class UserProfile is public, should be declared
         in a file named UserProfile.java

MISTAKE 2: Forgetting semicolons
  System.out.println("Hello")   ← missing ;
  Error reported on NEXT line (confusing)
  Always look at the line BEFORE the reported error

MISTAKE 3: Wrong main signature
  public void main(String[] args)  ← missing static
  public static void Main(...)     ← Main ≠ main (case!)
  public static void main()        ← missing String[] args
  public static int main(String[] args) ← wrong return type
  
  All cause: "Main method not found" error

MISTAKE 4: Mismatched braces
  public class Main {
      public static void main(String[] args) {
          System.out.println("Hello");
      }
  // Missing closing } for class
  
  IntelliJ highlights unmatched braces.
  Click on a brace → IntelliJ highlights its match.

MISTAKE 5: Using == for String comparison
  String s = "hello";
  if (s == "hello") { ... }     ← works sometimes but WRONG
  if (s.equals("hello")) { ... } ← CORRECT, always use this
  
  We'll understand fully in Level 1.9 (Strings).
  For now: ALWAYS use .equals() for String comparison.

MISTAKE 6: Printing to err instead of out (accidental)
  System.err.println("Hello");  ← shows in red in IntelliJ
  System.out.println("Hello");  ← normal output
  Both print to terminal but err is for errors.

MISTAKE 7: Semicolon after class/method declaration
  public class Main; {   ← WRONG
  public class Main {    ← correct
  
  public static void main(String[] args); {  ← WRONG
  public static void main(String[] args) {   ← correct
```

---
## Interview Questions
```
Q: Why does the main method need to be static?
A: When the JVM starts, no objects exist yet. To call a
   non-static method, you need an instance of the class.
   But the JVM can't create an instance before it has a
   starting point. Making main static means the JVM can call
   it directly on the class — Main.main(args) — without
   creating any object first. After main starts, you can
   create objects normally.

Q: What does public static void main(String[] args) mean
   word by word?
A: public means accessible from anywhere — needed so the JVM
   can call it from outside the class.
   static means it belongs to the class, not an object —
   needed so JVM can call it without creating an instance.
   void means it returns no value — when main ends,
   the program ends, nothing to return to.
   main is the specific method name the JVM looks for.
   String[] args is an array of command line arguments —
   required by the JVM specification even if unused.

Q: What is the difference between System.out.print
   and System.out.println?
A: print outputs the text and leaves the cursor at the end
   of the same line. println outputs the text and then adds
   a newline character, moving the cursor to the start of
   the next line. println is equivalent to print + "\n".

Q: Can you have multiple classes in one .java file?
A: Yes, but only ONE class can be public, and the filename
   must match that public class. The other classes in the
   file are package-private (no access modifier). In practice,
   professional code always puts each class in its own file
   for clarity and maintainability. The one exception is
   nested classes (a class inside a class), which are common
   and fine.

Q: What is System.out?
A: System is a class in java.lang (auto-imported).
   out is a static field in System of type PrintStream.
   It represents the standard output stream — your terminal.
   println() and print() are methods on that PrintStream
   object. So System.out.println() means: access the System
   class, get its out field (a PrintStream), call println on it.
```

---
## Key Takeaways
```
1. public class ClassName {  }
   → Every Java program is inside a class.
   → Filename must match the public class name exactly.
   → Case sensitive.

2. public static void main(String[] args) {  }
   → The ONLY entry point the JVM looks for.
   → public: JVM must access it from outside.
   → static: JVM calls it before any objects exist.
   → void: returns nothing (program ends after main).
   → String[] args: command line arguments (always required).

3. System.out.println() prints with newline.
   System.out.print() prints without newline.
   System.out.printf() prints with formatting.

4. Every statement ends with semicolon ;
   Class declarations, method declarations, blocks { }
   do NOT get semicolons.

5. Comments:
   // single line (use for short explanations of WHY)
   /* multi-line */
   /** Javadoc (for public APIs) */

6. static = class level (one copy, no object needed)
   non-static = instance level (per object, needs new)
   main must be static. This is non-negotiable.

7. The . (dot) operator means "access something inside"
   System.out = get out from System
   System.out.println = get println from that object

8. IntelliJ shortcuts:
   psvm + Tab → full main method
   sout + Tab → System.out.println()
   Ctrl+Alt+L → format code
```

---