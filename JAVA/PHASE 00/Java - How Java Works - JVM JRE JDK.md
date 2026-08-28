---
title: "Java - How Java Works - JVM JRE JDK"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - jvm
status: "not-started"
---

# Java - How Java Works - JVM JRE JDK

> [!abstract] Overview
> Java is a compiled and interpreted language that runs on a virtual machine called the JVM. Understanding the relationship between JVM, JRE, and JDK is the first step to understanding why Java is called "write once, run anywhere" and why it dominates enterprise backend development.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - Basic idea of what a programming language is
> - What an operating system does (very high level)
> - No prior Java knowledge required

---

## Theory

### What is Java?
Java is a general-purpose, object-oriented programming language created by James Gosling at Sun Microsystems in 1995. Sun was later acquired by Oracle, which now maintains Java. It was designed with one massive goal in mind: to allow developers to write code once and run it on any machine, regardless of the underlying operating system or hardware.

Most programming languages work in one of two ways. **Compiled languages** like C and C++ convert your source code directly into machine code that runs on a specific operating system and processor. If you compile a C program on Windows, it will not run on Linux without recompilation. **Interpreted languages** like Python read your source code line by line at runtime and execute it, which is portable but slow.

Java takes a hybrid approach. Your Java source code is compiled into an intermediate format called **bytecode**, which is not tied to any specific operating system. This bytecode is then executed by a special program called the **Java Virtual Machine (JVM)**, which exists for every major operating system. The JVM translates bytecode into machine code that your specific computer understands.

### Why does it exist?
Before Java, software companies had a painful problem. If they wrote a program in C++ for Windows, they had to rewrite significant portions of it to run on Mac or Linux or Solaris. This meant maintaining multiple codebases, hiring multiple teams, and shipping different products for different platforms. It was expensive and error-prone.

Java solved this by adding a layer of abstraction between your code and the operating system. That layer is the JVM. As long as a JVM exists for a platform, your Java code runs on it without any changes. This is why Java became the dominant language for enterprise systems, banking software, telecom systems, and Android applications. Companies could write one backend system and deploy it on Windows servers, Linux servers, or anything else.

For backend engineering specifically, Java's portability, stability, mature ecosystem, and strong performance made it the default choice for large-scale systems at banks, e-commerce companies, streaming platforms (Netflix built much of its backend in Java), and payment processors.

### How does it work internally?
The journey of your Java code from writing to execution has three stages:

**Stage 1: Writing and Compilation**
You write your code in a file with a `.java` extension. For example, `HelloWorld.java`. This is human-readable source code.

You then run the Java compiler called `javac` on this file. The compiler checks your syntax, verifies types, and if everything is correct, produces a new file called `HelloWorld.class`. This `.class` file contains **bytecode**, which is a low-level, platform-independent instruction set. Bytecode is not machine code. No processor can execute it directly.

**Stage 2: Class Loading**
When you run your program using the `java` command, the JVM starts up. The first thing it does is load your `.class` files into memory using a component called the **Class Loader**. The Class Loader also loads all the core Java library classes your program depends on, like `String`, `System`, `ArrayList`, and so on.

**Stage 3: Bytecode Verification and Execution**
The JVM verifies the bytecode to make sure it is safe. It checks that the code does not access memory it should not, does not violate type safety, and follows all the rules of the Java language. This verification is one reason Java is considered secure.

Then the **Execution Engine** takes over. It uses two techniques:
- **Interpreter**: Reads bytecode instructions one at a time and executes them. This is slow but starts immediately.
- **JIT (Just-In-Time) Compiler**: Watches your program run. When it notices certain methods being called many times (called "hot code"), it compiles those methods into native machine code and caches them. Future calls to that method skip the interpreter and run at native speed.

This combination gives Java both fast startup (interpreter) and high runtime performance (JIT compiler).

### JDK vs JRE vs JVM
These three terms confuse beginners, but the relationship is simple.

**JVM (Java Virtual Machine)** is the engine that runs bytecode. It is a specification, and there are multiple implementations of it, such as HotSpot (from Oracle), OpenJ9 (from IBM), and GraalVM. When you "run" a Java program, you are really running a JVM that loads your bytecode.

**JRE (Java Runtime Environment)** is a bundle that contains the JVM plus the core Java libraries (like `java.util`, `java.io`, etc.). If you only want to run Java programs (not develop them), you only need the JRE. Note that starting from Java 11, Oracle no longer distributes a separate JRE. Modern Java simply distributes the JDK, and you can create a smaller runtime image using a tool called `jlink` if you need one.

**JDK (Java Development Kit)** is the full development package. It contains the JRE plus development tools like the compiler (`javac`), the debugger (`jdb`), the documentation generator (`javadoc`), and other utilities. If you want to write Java code, you install the JDK.

Think of it like this. If Java were a car:
- The **JVM** is the engine.
- The **JRE** is the fully assembled car ready to drive.
- The **JDK** is the car plus the mechanic's toolkit to build and repair cars.

> [!tip] Key Insight
> Java's "write once, run anywhere" promise is delivered by the JVM. Your `.class` bytecode files are portable, but the JVM itself is not. Different JVMs exist for different operating systems, and each one knows how to translate the same bytecode into instructions its host system understands.

---
## Syntax and Basic Examples

### Basic Syntax
Here is a minimal Java program to see the full flow in action.

```java
// File name: HelloWorld.java
// The file name must match the public class name exactly, including case.

public class HelloWorld {
    // The main method is the entry point of every Java application.
    // The JVM looks for this exact signature to start execution.
    public static void main(String[] args) {
        // System.out.println prints text to the console followed by a newline.
        System.out.println("Hello, backend world!");
    }
}
```

How to compile and run:

```bash
# Step 1: Compile the source file into bytecode.
# This produces a file named HelloWorld.class in the same directory.
javac HelloWorld.java

# Step 2: Run the bytecode with the JVM.
# Notice you do NOT type the .class extension.
java HelloWorld
```

Output:

```text
Hello, backend world!
```

### Example 1: Checking Your Java Version

Before writing any code, it is important to verify your installation. Every professional Java project specifies which Java version it targets.

```bash
# Check the version of the runtime (JVM).
java --version

# Check the version of the compiler.
javac --version
```

Example output:

```text
openjdk 21.0.2 2024-01-16
OpenJDK Runtime Environment (build 21.0.2+13)
OpenJDK 64-Bit Server VM (build 21.0.2+13, mixed mode)
```

Both versions should match. If they do not, you have multiple JDKs installed and your system is pointing to different ones for compiling and running.

### Example 2: Seeing Bytecode with Your Own Eyes

Java includes a tool called `javap` that lets you look at the bytecode of a compiled class. This helps you understand what the compiler actually produces.

```java
// File: Adder.java
public class Adder {
    public static void main(String[] args) {
        int a = 5;
        int b = 10;
        int sum = a + b;
        System.out.println(sum);
    }
}
```

Compile it and then disassemble it:

```bash
javac Adder.java
javap -c Adder
```

Output (bytecode):

```text
public class Adder {
  public Adder();
    Code:
       0: aload_0
       1: invokespecial #1  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_5              // push integer 5 onto the stack
       1: istore_1              // store into local variable 1 (a)
       2: bipush        10      // push integer 10 onto the stack
       4: istore_2              // store into local variable 2 (b)
       5: iload_1               // load variable 1 (a) onto the stack
       6: iload_2               // load variable 2 (b) onto the stack
       7: iadd                  // pop top two, add them, push result
       8: istore_3              // store result into variable 3 (sum)
       9: getstatic     #7      // get System.out
      12: iload_3               // load sum onto the stack
      13: invokevirtual #13     // call println
      16: return
}
```

You do not need to memorize this. The point is to see that the JVM is a stack-based machine. It pushes values, performs operations, and pops results. This is what "bytecode" actually looks like.

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Every Spring Boot backend application has an entry point that looks similar to the main method you saw above. Here is what the starting point of a real backend service looks like.

**Scenario: A minimal Spring Boot application starting up**

```java
// File: OrderServiceApplication.java
// This is the entry point of an entire backend microservice
// that might handle thousands of e-commerce orders per second.

package com.company.orderservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrderServiceApplication {

    public static void main(String[] args) {
        // This single line starts an embedded web server (Tomcat),
        // loads all your controllers, services, and database connections,
        // and begins listening for HTTP requests.
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

What to notice:
- The `main` method signature `public static void main(String[] args)` is identical to your first Hello World. Every Java program, from a small script to a massive backend microservice, starts here. The JVM does not care whether your program prints text or serves millions of API requests. It always starts by finding `main`.
- The `@SpringBootApplication` annotation is a signal to the Spring framework, not to the JVM. The JVM only sees a class with a `main` method. Spring does its magic on top of Java, not inside it.
- When you deploy this backend service to a production Linux server, the exact same `.class` files that ran on your Mac during development will run there. This is the JVM's portability guarantee in action.
- The command to run this in production looks like `java -jar order-service.jar`, which invokes the JVM to execute the packaged bytecode.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Filename does not match class name

Wrong:
```java
// File saved as: hello.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

Compiling this gives:
```text
hello.java:1: error: class HelloWorld is public, should be declared in a file named HelloWorld.java
```

Right:
```java
// File saved as: HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

Why it is wrong: In Java, the filename of a `.java` file must exactly match the name of the public class it contains, including capitalization. This is a rule the compiler enforces so that other classes can find yours reliably.

### Mistake 2: Running with the wrong command

Wrong:
```bash
java HelloWorld.java   # Trying to run source code (works in Java 11+ but not always what you want)
java HelloWorld.class  # Adding the .class extension
```

Right:
```bash
javac HelloWorld.java  # Compile first
java HelloWorld        # Then run the class name without extension
```

Why it is wrong: The `java` command expects a class name, not a filename. Java 11 introduced single-file source-code execution, so `java HelloWorld.java` does work as a shortcut for simple scripts, but it recompiles every time and is not used in real projects.

### Mistake 3: Confusing JDK, JRE, and JVM

- **Wrong thinking:** "I installed the JVM, so I can start writing Java code."
- **Right thinking:** "I need the JDK to write and compile Java code. The JDK includes the compiler `javac` and also includes a JVM to run the code."
- **Why it is wrong:** If you only have a JRE or a bare JVM, you cannot compile `.java` files because there is no `javac`. Always install the full JDK for development.

### Mistake 4: Assuming Java is fully interpreted or fully compiled

- **Wrong thinking:** "Java is slow because it is interpreted like Python."
- **Right thinking:** "Java is compiled to bytecode ahead of time, and the JIT compiler further compiles hot bytecode to native machine code at runtime."
- **Why it is wrong:** Modern Java performance is close to C++ for long-running server applications precisely because of the JIT compiler. This is why Java is trusted for high-throughput backend systems at Netflix, Uber, LinkedIn, and thousands of banks.

---

## Key Takeaways

> [!tip] Remember these points

- Java source code (`.java`) is compiled by `javac` into bytecode (`.class`), which is then executed by the JVM.
- The JVM runs bytecode, the JRE is the JVM plus core libraries for running Java, and the JDK is the JRE plus tools for developing Java. Always install the JDK.
- Bytecode is platform-independent. The same `.class` file runs on any operating system that has a JVM. This is the "write once, run anywhere" principle.
- The JIT compiler makes long-running Java applications fast by translating frequently used bytecode into native machine code at runtime.
- Every Java program, from Hello World to a Netflix microservice, starts execution at `public static void main(String[] args)`.

---
## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Install and Verify (Easy)
Install JDK 21 (LTS) on your Mac using Homebrew or SDKMAN. Verify that both `java --version` and `javac --version` return matching version numbers. Take a screenshot or write down the output in this note.

**Hint:** SDKMAN is the recommended way for developers because it lets you switch between multiple JDK versions easily. Try running `sdk install java 21.0.2-tem`.

### Exercise 2: Write, Compile, Run (Easy)
Create a file called `AboutMe.java`. Inside, print three lines: your name, your university, and your dream job title. Compile it with `javac`, run it with `java`, and confirm the output.

**Hint:** You will need three `System.out.println` statements inside `main`.

### Exercise 3: Explore Bytecode (Medium)
Create a small Java program that adds two numbers and prints the result. Compile it. Use `javap -c YourClassName` to see the bytecode. Try to identify which bytecode instruction performs the addition.

**Hint:** Look for an instruction that starts with `i` and ends with `add`.

### Exercise 4: Break Something Intentionally (Medium, Optional)
Create a file named `Test.java` but name the class inside it `Different`. Try to compile it. Read the error message carefully and understand what it tells you.

**Hint:** The class must not be `public` if the filename does not match. Try adding and removing the `public` keyword and observe when compilation succeeds and when it fails.


### Solution
**For Exercise 2:**
```java
// File: AboutMe.java
public class AboutMe {
    public static void main(String[] args) {
        System.out.println("Abdullah Al Sayb Saad");
        System.out.println("Your University Name");
        System.out.println("Backend Engineer");
    }
}
```

Commands:
```bash
javac AboutMe.java
java AboutMe
```


**For Exercise 3:**
```java
// File: Sum.java
public class Sum {
    public static void main(String[] args) {
        int result = 7 + 3;
        System.out.println(result);
    }
}
```
After running `javap -c Sum`, look for the `iadd` instruction. That is the JVM's integer addition operation.

---
## Related Notes
- [[Java - Setting Up Environment - IntelliJ and JDK]]
- [[Java - Variables and Data Types]]

---

## Resources
- [Oracle Java Documentation](https://docs.oracle.com/en/java/) - The official reference for the language and standard library.
- [SDKMAN](https://sdkman.io) - The best tool to install and manage multiple JDK versions on Mac and Linux.
- [Adoptium](https://adoptium.net) - Free, high-quality builds of OpenJDK used in production by many companies.
- [Baeldung: JVM vs JRE vs JDK](https://www.baeldung.com/jvm-vs-jre-vs-jdk) - Deeper article on the differences with diagrams.

---