**Phase:** Level 1 - Java Fundamentals  
**Date Studied:**   

---
## What Problem Does This Solve?
Most beginners just install Java and start typing code.  
They have no idea what happens when they press "Run".

This is a problem because:
```
When something breaks → you don't know WHERE it broke
When performance is bad → you don't know WHY
When JVM errors appear → they look like random gibberish
When someone asks "why is Java platform independent?"
→ you can't answer

Understanding HOW Java works makes you:
  → Better at debugging (you know where to look)
  → Better at performance (you know what costs what)
  → Better at interviews (this is asked constantly)
  → Better at reading error messages (stack traces make sense)
  → A real engineer instead of someone who just types code
```
This topic is foundational.  
Everything else in Java sits on top of this.

---

## The Big Picture
Before Any Details.
Let's start with the most important question:
```
What actually happens when you write Java and press Run?

You write this:
┌─────────────────────────────────┐
│  public class Main {            │
│    public static void main      │
│      (String[] args) {          │
│      System.out.println(        │
│        "Hello, World!");        │
│    }                            │
│  }                              │
└─────────────────────────────────┘
        │
        │  Step 1: You save this as Main.java
        │          (plain text file - human readable)
        ▼
┌─────────────────────────────────┐
│         Java Compiler           │
│           (javac)               │
│                                 │
│  Reads your .java file          │
│  Checks for syntax errors       │
│  Translates to bytecode         │
└─────────────────────────────────┘
        │
        │  Step 2: Compiler produces Main.class
        │          (bytecode - NOT human readable)
        │          (NOT machine code either)
        │          (something in between)
        ▼
┌─────────────────────────────────┐
│       Java Virtual Machine      │
│             (JVM)               │
│                                 │
│  Reads the .class bytecode      │
│  Interprets OR compiles it      │
│  to actual machine code         │
│  Executes it on YOUR computer   │
└─────────────────────────────────┘
        │
        │  Step 3: Your program runs
        │          Output appears on screen
        ▼
   Hello, World!

This three-step process is the entire secret of Java.
Everything else is details on top of this.
```

---
## Step 1 - You Write Source Code (`.java`)
Source code is the code YOU write.
It is just a text file with `.java` extension.
```
Rules for .java files:
  → File name MUST match the public class name
     If class is: public class UserService
     File must be: UserService.java
     Case sensitive! UserService ≠ userservice
  
  → One public class per file
     You can have multiple classes in one file
     But only ONE can be public
     And the file name matches the public one
  
  → Java is case-sensitive
     String ≠ string ≠ STRING
     main ≠ Main ≠ MAIN
     This causes many beginner bugs
```

Example file structure:
```
  src/
    main/
      java/
        com/
          yourname/
            Main.java          ← contains: public class Main
            UserService.java   ← contains: public class UserService
            models/
              User.java        ← contains: public class User
```
- The package structure (`com/yourname/`) matters.
- It prevents naming conflicts.
- Two companies can both have a class called "User" as long as they have different packages.
- (com.google.User vs com.yourname.User)

---
## Step 2 - The Java Compiler (`javac`)

### What Does the Compiler Do?
```
The compiler does three things:

1. SYNTAX CHECK
   Reads your code and checks:
   → Are all braces {} balanced?
   → Are all statements ended with ;?
   → Are you using variables before declaring them?
   → Are types correct? (can't assign String to int)
   → Are methods called with correct number of arguments?
   
   If ANY syntax error exists → compilation FAILS
   No .class file is created
   Compiler tells you exactly what line has the error
   
   THIS IS A GOOD THING.
   The compiler catches mistakes BEFORE your program runs.
   Strongly-typed languages like Java catch more errors
   at compile time than languages like Python.

2. TRANSLATION TO BYTECODE
   Source code (human readable):
     int x = 5;
     int y = 10;
     System.out.println(x + y);
   
   Bytecode (JVM readable, not human readable):
     0: iconst_5          ← push 5 onto stack
     1: istore_1          ← store in variable 1
     2: bipush 10         ← push 10 onto stack
     4: istore_2          ← store in variable 2
     5: getstatic ...     ← get System.out
     8: iload_1           ← load variable 1
     9: iload_2           ← load variable 2
    10: iadd              ← add them
    11: invokevirtual ... ← call println
   
   You don't need to understand bytecode.
   Just know: it's an intermediate language.
   Not your code. Not machine code. Something in between.

3. PRODUCES .class FILES
   One .class file per class.
   Main.java → Main.class
   UserService.java → UserService.class
   
   These .class files are what you DEPLOY.
   Not your .java source code.
   (Though in practice, Maven packages them into .jar files)
```

```
Running the compiler manually:
  javac Main.java
  → produces Main.class in same directory
  
  javac -d out/ src/Main.java
  → produces Main.class in out/ directory
  
  (IntelliJ and Maven do this automatically for you)
```

### Why Not Compile Directly to Machine Code?
This is the KEY question that explains Java's design.
```
Machine code is different for every CPU:
  Intel x86 code ≠ ARM code (phone chips) ≠ Apple M1 code

If Java compiled to machine code:
  → Compile on Windows Intel → runs on Windows Intel only
  → Need different compilation for Linux, Mac, Phone
  → Need recompilation every time hardware changes
  → This was the nightmare of C/C++ programs in the 90s

Java's solution: Bytecode + JVM
  → Java compiles to BYTECODE (neutral intermediate format)
  → Each platform has its own JVM
  → JVM translates bytecode to THAT platform's machine code
  → Same .class file runs on Windows, Linux, Mac, Phone
  → "Write Once, Run Anywhere" — Java's famous slogan
  
  The bytecode is the universal language.
  The JVM is the translator for each platform.

Visual:
         Your .class file (bytecode)
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
    Windows JVM  Linux JVM  Mac JVM
          │         │         │
          ▼         ▼         ▼
    Windows    Linux      Mac
    Machine    Machine    Machine
    Code       Code       Code
```

---
## Step 3 - The JVM
JVM - Java Virtual Machine.
### What Is the Java Virtual Machine?
- "Virtual" machine because it's a SOFTWARE machine.
- It simulates a computer inside your computer.
- It has its own instruction set (bytecode).
- It manages its own memory.
- It runs your programs.
```
The JVM does:
  1. Class Loading      → finds and loads .class files
  2. Bytecode Verification → checks bytecode is safe/valid
  3. Execution          → runs your program
  4. Memory Management  → allocates and frees memory (GC)
  5. Security           → sandboxes code from OS
  6. Optimization       → makes your code run faster over time

The JVM is NOT Java-specific.
Other languages compile to JVM bytecode too:
  → Kotlin (Android development — also used in Spring Boot)
  → Scala (big data — Apache Spark)
  → Groovy (scripting, Gradle build files)
  → Clojure (functional programming)

This means: a Kotlin class and a Java class can
call each other directly — they both run on the JVM.
This is why Spring Boot works with Kotlin.
```

### How the JVM Executes Code
Interpretation vs Compilation.
```
The JVM has TWO ways to run bytecode:

METHOD 1: Interpretation
  JVM reads bytecode line by line.
  Translates each instruction to machine code.
  Executes it immediately.
  
  Advantage: starts fast (no upfront compilation wait)
  Disadvantage: slower overall (translating every time)

METHOD 2: JIT Compilation (Just-In-Time)
  JVM watches which code runs FREQUENTLY (hot code).
  Compiles that hot code to native machine code.
  Stores the compiled version.
  Next time that code runs → uses compiled version directly.
  
  Advantage: fast after warmup period
  Disadvantage: needs warmup time
```

```
WHAT ACTUALLY HAPPENS (both combined):
  
  App starts → JVM interprets code (fast startup)
       │
       ▼
  JVM monitors: which methods run most often?
       │
       ▼
  Hot methods → JIT compiles them to machine code
       │
       ▼
  App runs faster and faster over time
  (This is called "JVM warmup")

  This is why:
  → Java apps are sometimes slow at startup
  → But fast after running for a while
  → This is why servers run Java continuously
    rather than starting/stopping constantly
  → Why AWS Lambda (serverless) cold starts are 
    slower for Java than for Python/Node
```

```
MODERN JIT (Java 21):
  C1 Compiler (Client) → quick compilation, basic optimization
  C2 Compiler (Server) → slower compilation, heavy optimization
  GraalVM              → even more aggressive optimization
                         can also compile AHEAD of time (AOT)
  
  HotSpot JVM (what you use):
  Uses both C1 and C2 based on how "hot" the code is.
  Code that runs millions of times → heavily optimized.
  Code that runs once → barely optimized.
  Smart resource usage.
```

### JVM Memory
Where Your Data Lives!
```
The JVM manages memory in separate regions.
Understanding this prevents bugs and performance issues.

JVM MEMORY STRUCTURE:
┌──────────────────────────────────────────────────────┐
│                    JVM Memory                        │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │                   HEAP                          │ │
│  │                                                 │ │
│  │  Where OBJECTS live                             │ │
│  │  new User() → created here                      │ │
│  │  new ArrayList() → created here                 │ │
│  │  All instances of classes → here                │ │
│  │                                                 │ │
│  │  ┌──────────────┐  ┌────────────────────────┐   │ │
│  │  │ Young Gen    │  │    Old Generation      │   │ │
│  │  │              │  │                        │   │ │
│  │  │ New objects  │  │ Objects that survived  │   │ │
│  │  │ start here   │  │ many GC cycles         │   │ │
│  │  │              │  │ Long-lived objects     │   │ │
│  │  │ Eden Space   │  │                        │   │ │
│  │  │ Survivor 0   │  │                        │   │ │
│  │  │ Survivor 1   │  │                        │   │ │
│  │  └──────────────┘  └────────────────────────┘   │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │                   STACK                         │ │
│  │                                                 │ │
│  │  One stack per THREAD                           │ │
│  │  Local variables live here                      │ │
│  │  Method calls create "stack frames"             │ │
│  │  When method returns → frame deleted            │ │
│  │  Automatically managed (no GC needed)           │ │
│  │  Fast but LIMITED in size                       │ │
│  │  Too deep recursion → StackOverflowError        │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │                 METASPACE                       │ │
│  │  (called PermGen in Java 7 and earlier)         │ │
│  │                                                 │ │
│  │  Class definitions (metadata) stored here       │ │
│  │  Method bytecode                                │ │
│  │  Static variables                               │ │
│  │  Grows dynamically (unlike old PermGen)         │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │              STRING POOL                        │ │
│  │  (part of Heap in Java 7+)                      │ │
│  │                                                 │ │
│  │  String literals stored here                    │ │
│  │  "Hello" used 100 times → stored ONCE           │ │
│  │  Memory optimization for Strings                │ │
│  └─────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

```
STACK vs HEAP in simple terms:
  
  int x = 5;
  → x lives on the STACK (primitive, fixed size, in method)
  
  User user = new User("Rahim");
  → user variable (the reference) lives on STACK
  → The actual User OBJECT lives on HEAP
  → Stack holds the "address" of where to find the object
  
  This is critical:
  Stack: small, fast, automatic cleanup
  Heap: large, slower, needs garbage collection
```

### Garbage Collection
Automatic Memory Management (GC).
```
In C/C++: you manually allocate and free memory.
  malloc() to allocate
  free() to deallocate
  Forget to free → memory leak
  Free too early → crash
  This is one of the hardest problems in programming.

In Java: Garbage Collector (GC) handles this automatically.

HOW GC WORKS (simplified):
  
  Step 1: Mark
    GC starts from "root" references
    (local variables, static fields, active threads)
    Follows every reference to objects
    Marks every object that is REACHABLE
  
  Step 2: Sweep
    Any object NOT marked = unreachable = garbage
    GC deletes it and reclaims memory
  
  Step 3: Compact (sometimes)
    Moves surviving objects together
    Eliminates fragmentation
    Makes allocation faster
```

```
WHEN DOES GC RUN?
  The JVM decides. Not you.
  Usually when heap is getting full.
  Can be triggered manually (not recommended):
    System.gc(); ← hint to JVM, not a command
  
  GC pauses your application while it runs.
  ("Stop the world" pause)
  Modern GCs minimize these pauses.

TYPES OF GC IN JAVA 21:
  G1GC (Garbage First) → default, balanced
  ZGC → ultra-low latency, <1ms pauses (Java 21: production ready)
  Shenandoah → similar to ZGC
  Parallel GC → maximum throughput, higher pauses
  Serial GC → single-threaded, small apps only

FOR YOU NOW:
  Don't stress about GC details.
  Just know:
  → Java manages memory automatically (unlike C/C++)
  → Objects on heap are cleaned up when unreachable
  → Setting an object to null makes it eligible for GC
  → Memory leaks can still happen (if you hold references
    to objects you don't need anymore)
  
  We'll go deep on GC in Level 3 (Java Advanced).
```

---
## JDK vs JRE vs JVM - Crystal Clear
```
One final time, with full clarity:

JVM (Java Virtual Machine)
  What: The engine that RUNS Java bytecode
  Who uses it: Everyone (it's inside JRE and JDK)
  Can you install it alone: Not usually — comes with JRE/JDK
  Analogy: The car's engine

JRE (Java Runtime Environment)  
  What: JVM + standard library classes (java.lang, java.util, etc.)
  Who uses it: End users who want to RUN Java apps
  Can you install alone: Yes, but Oracle stopped shipping it standalone
  Contains: JVM + rt.jar (all standard Java classes)
  Analogy: The car with engine + all standard features

JDK (Java Development Kit)
  What: JRE + compiler (javac) + debugger + other dev tools
  Who uses it: DEVELOPERS (YOU)
  Must install: YES — this is what you have
  Contains: Everything in JRE + javac + jdb + javadoc + more
  Analogy: The car + engine + mechanic's toolbox

SIMPLE RULE:
  You develop → install JDK (contains everything)
  User runs your app → only needs JRE (or JDK also works)
  Java runs → always using JVM (inside JDK/JRE)
```

```
┌─────────────────────────────────┐
│              JDK                │
│  ┌───────────────────────────┐  │
│  │           JRE             │  │
│  │  ┌─────────────────────┐  │  │
│  │  │        JVM          │  │  │
│  │  │   (runs bytecode)   │  │  │
│  │  └─────────────────────┘  │  │
│  │  Standard Libraries       │  │
│  │  (java.lang, java.util..) │  │
│  └───────────────────────────┘  │
│  javac (compiler)               │
│  jdb (debugger)                 │
│  javadoc (documentation gen)    │
│  jconsole (monitoring)          │
│  jps, jstat, jmap (diagnostics) │
└─────────────────────────────────┘
```

---
## Java Versions - What You Need to Know
```
Java releases every 6 months (since Java 9).
Not all versions are equal.

VERSION TIMELINE (relevant versions only):
  
  Java 8  (2014) ── LTS ── Still in MILLIONS of systems
    Added: Lambdas, Streams, Optional, default methods
    Why it matters: Huge amount of existing code uses Java 8
    You'll see Java 8 code in legacy codebases
  
  Java 11 (2018) ── LTS ── Was standard for years
    Added: var keyword improvements, new String methods,
           HttpClient API, removed some old stuff
    Why it matters: Many companies still on Java 11
  
  Java 17 (2021) ── LTS ── Very widely used RIGHT NOW
    Added: Sealed classes, Records, Pattern matching basics,
           TextBlocks (multiline Strings), Switch expressions
    Why it matters: Spring Boot 3.x requires 17+
                    Most "modern Java" code targets 17
  
  Java 21 (2023) ── LTS ── Current. This is YOURS.
    Added: Virtual Threads (Project Loom — revolutionary!),
           Record Patterns, Pattern matching for switch,
           Sequenced Collections, String Templates (preview)
    Why it matters: Most modern, best performance,
                    companies migrating to this now
  
  Java 25 (2025) ── LTS ── Future
    Not released yet at time of writing
```

```
WHICH VERSION TO CARE ABOUT:
  Write code in: Java 21
  Must understand: Java 8 features (lambdas, streams)
    → Because 80% of Java tutorials/code you find online
      uses Java 8 style. You need to read it.
  Should know exists: Java 17 features (records, sealed)
  
  When you see Java 8 code in tutorials:
  → It still works in Java 21 (backward compatible)
  → Java is ALWAYS backward compatible
  → Code from 1998 can still compile and run in Java 21

VIRTUAL THREADS (Java 21 — Brief Introduction):
  Traditional threads = OS threads (heavy, limited)
  Virtual threads = JVM-managed threads (lightweight, millions possible)
  
  Why this matters for backend:
  A Spring Boot app handling 10,000 requests simultaneously
  → Old: needed complex async code (CompletableFuture, reactive)
  → Java 21: simple blocking code works because virtual threads
             are so lightweight the JVM handles the concurrency
  
  Spring Boot 3.2 + Java 21 = virtual thread support built in
  This is the future of Java backend development.
  We'll use this in Level 6 (Spring Boot).
```

---
## Java Distributions - Which One Are You Running?
```
"Java" is a specification. Multiple companies implement it.
They are all compatible (same bytecode, same APIs).

DISTRIBUTIONS:
  
  Eclipse Temurin (Adoptium) ← YOU HAVE THIS
    Free, open source, production-grade
    Used by: most companies that don't use Oracle JDK
    Recommended for: everyone who doesn't have a license

  Oracle JDK
    Official Oracle distribution
    Free for development and personal use
    Requires paid license for commercial use (enterprise)
    Used by: large enterprises with Oracle licenses

  Amazon Corretto
    Amazon's distribution, optimized for AWS
    Free, production-ready
    Used by: companies on AWS (Chaldal, Shohoz likely)

  Azul Zulu
    Azul's distribution, good support options
    Used by: companies wanting commercial support without Oracle

  GraalVM (GraalVM Community)
    Special: can AOT compile to native image (fast startup!)
    Used by: Quarkus framework, performance-critical apps
    Advanced topic — come back to this later

ALL OF THESE:
  → Run the same Java bytecode
  → Implement the same Java APIs
  → Are interchangeable for learning
  → Your code written for one works on all

For you: Temurin 21 is perfect.
```

---
## The Complete Journey - Source to Running Program
Let's trace your Hello World completely.
YOU WRITE (Main.java):
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

```
STEP 1: IntelliJ (or you) runs:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
javac Main.java

STEP 2: Compiler checks syntax:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Class name matches file name (Main = Main.java) ✓
✓ All braces balanced ✓
✓ main method signature correct ✓
✓ System.out.println exists ✓
→ No errors. Proceed.

STEP 3: Compiler produces:
━━━━━━━━━━━━━━━━━━━━━━━━━
Main.class (bytecode file)

STEP 4: You (or IntelliJ) runs:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
java Main

STEP 5: JVM Class Loader finds Main.class:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Looks in current directory
Finds Main.class
Loads it into memory

STEP 6: JVM Bytecode Verifier checks:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Is this valid bytecode? (not corrupted/malicious)
✓ Valid. Proceed.

STEP 7: JVM finds main method:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Looks for: public static void main(String[] args)
This is the ENTRY POINT of every Java application.
Found it. Start executing here.

STEP 8: JVM executes:
━━━━━━━━━━━━━━━━━━━━━
System.out.println("Hello, World!")
→ JVM calls System class
→ Accesses out field (a PrintStream object)
→ Calls println method on it
→ PrintStream writes to standard output (your terminal)

STEP 9: Output appears:
━━━━━━━━━━━━━━━━━━━━━━
Hello, World!

STEP 10: main method returns:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
No more code to run.
JVM shuts down.
Process exits with code 0 (success).
(Exit code 1 or other non-zero = error)

TOTAL TIME: Milliseconds.
This entire process happens before you can blink.
```

---
## What Happens in a Spring Boot App (Preview)
```
You don't need to understand this fully yet.
But seeing the bigger picture now helps later.

When you eventually run a Spring Boot app:

java -jar myapp.jar
        │
        ▼
JVM starts, loads your jar file
        │
        ▼
Your main class runs:
  SpringApplication.run(MyApp.class, args);
        │
        ▼
Spring Boot starts:
  → Scans for classes with @Component, @Service etc.
  → Creates objects (beans) for them
  → Wires dependencies between objects
  → Starts embedded Tomcat server (on port 8080)
  → Runs database migrations (Flyway)
  → Connects to database, Redis, Kafka etc.
        │
        ▼
Server is running:
  Started MyApp in 2.345 seconds
        │
        ▼
JVM is still running (not exiting!)
Waiting for HTTP requests
When request arrives → JVM processes it → sends response
JVM keeps running until you stop it (Ctrl+C)

This is different from Hello World:
  Hello World: JVM starts → runs → exits
  Spring Boot: JVM starts → runs → KEEPS RUNNING → serves requests

The JVM is a long-running process for server applications.
This is important for:
  → Understanding memory (heap fills up over time)
  → Understanding warmup (JIT optimizes after requests come in)
  → Understanding deployment (starting/stopping the JVM = deployment)
```

---
## Code Examples

### Example 1
Prove Compilation Creates `.class` Files
```java
// Save as: Hello.java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Compiled successfully!");
    }
}
```

In terminal:
```bash
javac Hello.java     # compile
ls                   # see Hello.class was created
java Hello           # run the .class file

# Output:
# Compiled successfully!

# Try a syntax error:
# Change println to printlnn (typo)
javac Hello.java

# Output (compile error - will NOT create .class):
# Hello.java:3: error: cannot find symbol
#         System.out.printlnn("Compiled successfully!");
#                   ^
#   symbol:   method printlnn(String)
#   location: variable out of type PrintStream
# 1 error
```

### Example 2
See the Stack in Action.
```java
// Save as: StackDemo.java
public class StackDemo {
    
    public static void main(String[] args) {
        // main() creates a stack frame
        // args lives on the stack (it's a local variable)
        
        int result = addNumbers(5, 10); // creates another frame
        System.out.println("Result: " + result);
        // addNumbers frame is gone after it returns
    }
    
    public static int addNumbers(int a, int b) {
        // addNumbers() creates its own stack frame
        // a, b, sum all live on THIS frame
        int sum = a + b;
        return sum;
        // this frame is DELETED when we return
        // sum, a, b are gone
    }
}

// Stack visualization as program runs:
//
// When main() starts:
// ┌─────────────────┐
// │ main() frame    │
// │  args = []      │
// │  result = ???   │  ← not yet set
// └─────────────────┘
//
// When addNumbers(5,10) is called:
// ┌─────────────────┐
// │ addNumbers()    │  ← top of stack
// │  a = 5          │
// │  b = 10         │
// │  sum = 15       │
// └─────────────────┘
// ┌─────────────────┐
// │ main() frame    │
// │  args = []      │
// │  result = ???   │
// └─────────────────┘
//
// When addNumbers() returns:
// ┌─────────────────┐
// │ main() frame    │  ← addNumbers frame DELETED
// │  args = []      │
// │  result = 15    │  ← now set
// └─────────────────┘
```

### Example 3
See Stack vs Heap.
```java
// Save as: MemoryDemo.java
public class MemoryDemo {
    
    public static void main(String[] args) {
        
        // STACK: primitive values live here directly
        int age = 25;           // 25 stored ON stack
        double price = 9.99;    // 9.99 stored ON stack
        boolean isActive = true; // true stored ON stack
        
        // HEAP: objects live here
        // 'name' variable is on STACK
        // but the String object "Rahim" is on HEAP
        // 'name' holds the MEMORY ADDRESS of the object
        String name = new String("Rahim");
        
        // String literals use String Pool (part of heap)
        // These two point to the SAME object in pool
        String s1 = "Hello"; // stored in String Pool
        String s2 = "Hello"; // reuses same object from pool!
        
        // These are different objects on heap
        String s3 = new String("Hello"); // forced new object
        String s4 = new String("Hello"); // another new object
        
        // Demonstrate: s1 and s2 are same reference
        System.out.println(s1 == s2);      // true (same object!)
        System.out.println(s3 == s4);      // false (different objects)
        System.out.println(s3.equals(s4)); // true (same content)
        
        // This is why you use .equals() for String comparison
        // not == (which compares references/addresses, not content)
        
        // When this method ends:
        // age, price, isActive, name, s1,s2,s3,s4
        // (the variables/references) are GONE from stack
        // But the String objects on heap remain
        // until Garbage Collector cleans them up
    }
}
```

### Example 4
Trigger a StackOverflowError.
```java
// Save as: OverflowDemo.java
// WARNING: This will crash intentionally — that's the point
public class OverflowDemo {
    
    public static void main(String[] args) {
        // This calls itself forever
        // Stack fills up → StackOverflowError
        countdown(1000000);
    }
    
    public static void countdown(int n) {
        // No base case to stop recursion!
        System.out.println(n);
        countdown(n - 1); // keeps calling itself forever
    }
}

// Run this and observe:
// Numbers count down...
// Then:
// Exception in thread "main" java.lang.StackOverflowError
//     at java.base/java.io.PrintStream.write(PrintStream.java:...)
//     at OverflowDemo.countdown(OverflowDemo.java:11)
//     at OverflowDemo.countdown(OverflowDemo.java:12)
//     at OverflowDemo.countdown(OverflowDemo.java:12)
//     ... (hundreds of lines)
//
// You can see: countdown is calling itself until stack is full.
// The fix: add a base case:
//   if (n <= 0) return; // stop recursion here
```

### Example 5
See Garbage Collection
```java
// Save as: GCDemo.java
public class GCDemo {
    
    // This is called when GC is about to delete this object
    // Don't rely on this in real code — just for demonstration
    @Override
    protected void finalize() {
        System.out.println("Object being garbage collected!");
    }
    
    public static void main(String[] args) throws InterruptedException {
        
        // Create an object on the heap
        GCDemo obj = new GCDemo();
        System.out.println("Object created: " + obj);
        
        // Remove the reference — object is now unreachable
        obj = null;
        System.out.println("Reference set to null");
        System.out.println("Object is now eligible for GC");
        
        // Suggest GC runs (just a hint — JVM may ignore it)
        System.gc();
        
        // Give GC time to run
        Thread.sleep(100);
        
        System.out.println("End of program");
    }
}

// The object on the heap has no more references.
// GC can now reclaim that memory.
// finalize() might print the GC message — or might not.
// (GC timing is unpredictable — that's the point)
```

---
## What to Run Today

- EXERCISE 1: Compile and run manually (not using IntelliJ run button)
  1. Create Hello.java in a folder
  2. Open terminal in that folder
  3. javac Hello.java → see Hello.class created
  4. java Hello → see output
  5. Delete Hello.class
  6. java Hello → see "class not found" error
  7. Understand: you need BOTH the compile AND run step

- EXERCISE 2: Intentionally break compilation
  1. Remove a semicolon from Hello.java
  2. javac Hello.java → read the error carefully
  3. Fix it
  4. Add an extra brace → compile → read error
  5. Fix it
  6. Misspell System as Sistem → compile → read error
  This teaches you to read compiler errors

- EXERCISE 3: Run the StackOverflowError demo
  1. Copy OverflowDemo.java
  2. Compile and run
  3. Watch the numbers count down
  4. See the StackOverflowError
  5. Add the base case (if n <= 0 return;)
  6. Run again → works correctly now

- EXERCISE 4: Understand == vs .equals() with Strings
  1. Copy MemoryDemo.java
  2. Run it
  3. Understand WHY s1 == s2 is true but s3 == s4 is false
  4. Write in your Obsidian notes:
  - "Always use `.equals()` to compare Strings, never"
  - "Because `==` compares memory addresses, not content"

---
## Common Misconceptions
```
MISCONCEPTION 1: "Java is slow"
REALITY: Modern Java (17, 21) is extremely fast.
  JIT compilation makes hot code native-speed.
  G1GC and ZGC minimize pauses.
  Netflix, Amazon, LinkedIn handle millions of requests with Java.
  The "Java is slow" reputation is from Java 1.x in the 90s.
  Not true today.

MISCONCEPTION 2: "The JVM compiles Java"
REALITY: javac (the Java compiler) compiles your code to bytecode.
  The JVM runs that bytecode.
  The JVM's JIT compiler compiles HOT bytecode to machine code.
  Two different things. javac is the compiler. JVM runs it.

MISCONCEPTION 3: "Java 8 and Java 21 are completely different"
REALITY: Java is backward compatible.
  Java 8 code compiles and runs on Java 21 JVM.
  Java 21 adds features. It doesn't remove Java 8 features.
  (Rare exceptions exist, but 99% of code is compatible)

MISCONCEPTION 4: ".class files are your source code"
REALITY: .class files are bytecode. Not source code.
  You can't read them (they're binary).
  But you can decompile them back to Java code.
  This is why you ship .jar files (containing .class files).
  Your actual .java source is not needed at runtime.

MISCONCEPTION 5: "Garbage Collection means no memory leaks"
REALITY: Memory leaks can still happen in Java.
  If you hold a reference to an object you no longer need,
  GC won't collect it (it's still "reachable").
  Classic example: adding to a static List and never removing.
  We'll cover this in Level 3.

MISCONCEPTION 6: "Java is only used for Android"
REALITY: Java is used for:
  → Enterprise backend (banking, fintech, e-commerce)
  → Android (historically — now Kotlin preferred)
  → Big data (Apache Spark, Hadoop core)
  → Cloud infrastructure (many AWS services)
  → Trading systems (ultra-low latency finance)
  → Scientific computing
  Android is a small fraction of Java's usage.
```

---
## Interview Questions
```
Q: Explain the difference between JDK, JRE, and JVM.
A: JVM is the Java Virtual Machine — the engine that executes
   Java bytecode. It handles memory management, garbage collection,
   and JIT compilation. JRE is the Java Runtime Environment —
   the JVM plus the standard library classes needed to run Java
   programs. It's for end users. JDK is the Java Development Kit —
   the JRE plus development tools like the javac compiler,
   debugger, and documentation generator. Developers install the JDK.

Q: What is bytecode and why does Java use it?
A: Bytecode is an intermediate representation of Java code —
   neither human-readable source nor machine-specific binary.
   The Java compiler translates source code to bytecode.
   Then each platform's JVM translates that bytecode to
   platform-specific machine code at runtime.
   This gives Java its "write once, run anywhere" property —
   the same .class bytecode runs on Windows, Linux, Mac,
   because each has its own JVM implementation.

Q: What is JIT compilation?
A: Just-In-Time compilation is where the JVM monitors which code
   runs most frequently (hot code) and compiles that bytecode
   directly to native machine code at runtime. Initially code is
   interpreted (slower), but after warmup the JIT-compiled hot
   paths run at near-native speed. This is why Java apps often
   start slightly slow but perform very well under sustained load.

Q: What is garbage collection in Java?
A: Garbage collection is automatic memory management in the JVM.
   When objects on the heap have no more references pointing to
   them — they're unreachable — the GC identifies and deletes them,
   reclaiming memory. This prevents the manual memory management
   bugs common in C/C++ (dangling pointers, memory leaks from
   forgetting to free). Modern GCs like G1 and ZGC minimize
   pause times to keep applications responsive.

Q: Where do local variables live vs objects in Java?
A: Local variables (primitives) live on the stack — a fast,
   automatically managed memory area tied to the current thread
   and method call. Object instances live on the heap — a larger,
   shared memory area managed by garbage collection. When you do
   String s = new String("hello"), the variable s (the reference)
   is on the stack, but the actual String object is on the heap.
   The stack holds the address of the heap object.

Q: Why is Java platform independent?
A: Java compiles to bytecode rather than platform-specific machine
   code. Bytecode is a neutral intermediate format that any JVM
   can understand. Each operating system and CPU architecture has
   its own JVM implementation, which translates bytecode to
   that platform's native machine instructions. So the same
   compiled Java program runs on Windows, Linux, and Mac without
   recompilation — you just need the appropriate JVM installed.
```

---
## Key Takeaways
```
1. Java's execution is a THREE-STEP process:
   Source code (.java) → Compiler (javac) → Bytecode (.class)
   → JVM → Machine code → Execution

2. Bytecode is the SECRET to Java's portability.
   Not machine code. Not source code.
   A neutral format every JVM can run.

3. JVM = engine. JRE = engine + standard library.
   JDK = everything. Install JDK. Always.

4. Stack = fast, automatic, per-thread, for local variables.
   Heap = managed by GC, for objects (new SomeClass()).
   Stack reference → points to → Heap object.

5. Garbage Collection = automatic memory management.
   Objects with no references → eligible for GC.
   GC runs when JVM decides → not when you decide.
   Memory leaks still possible if you hold references.

6. JIT compilation makes Java fast.
   Starts interpreted (slow) → hot code compiled (fast).
   JVM warmup = the time to reach peak performance.

7. Java is BACKWARD COMPATIBLE.
   Java 8 code runs on Java 21 JVM.
   This is why 30-year-old Java codebases still run.

8. Always use .equals() to compare String content.
   Never == for Strings (compares memory addresses).

9. StackOverflowError = infinite recursion (no base case).
   OutOfMemoryError = heap is full (GC can't keep up).
   These are JVM-level errors, not exceptions.

10. Java 21's Virtual Threads are a revolution.
    Millions of lightweight threads. Simple blocking code.
    Will change how Spring Boot apps are written.
```

---