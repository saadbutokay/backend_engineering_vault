## Overview

A computer is a machine that accepts input, processes it according to a set of instructions, stores results, and produces output. Every program you will ever write, from a simple calculator to a distributed payment system processing millions of transactions per second, reduces to this cycle. Understanding what happens beneath the abstractions is what separates engineers who debug effectively from those who guess.

---

## Core Concepts

### The Four Functions of a Computer

1. **Input** — Data enters the system (keyboard, network, file, sensor)
2. **Processing** — The CPU executes instructions on that data
3. **Storage** — Data is held temporarily (RAM) or permanently (disk)
4. **Output** — Results leave the system (screen, network, file, printer)

Every backend request follows this pattern. A client sends JSON (input), your Java application parses and validates it (processing), writes to PostgreSQL (storage), and returns a response (output).

### Binary and Data Representation

Computers operate on electrical signals: on or off, 1 or 0. These are **bits**.

- 1 bit = 0 or 1
- 1 byte = 8 bits (values 0-255)
- 1 kilobyte (KB) = 1,024 bytes
- 1 megabyte (MB) = 1,024 KB
- 1 gigabyte (GB) = 1,024 MB
- 1 terabyte (TB) = 1,024 GB

**Character encoding** maps numbers to characters:

- **ASCII** — 7-bit, 128 characters (English letters, digits, basic symbols)
- **UTF-8** — Variable-width (1-4 bytes), encodes all Unicode characters. This is the standard for the web and for Java's `String` internal representation in modern JVMs.

### The CPU

The Central Processing Unit executes instructions. Key components:

- **ALU (Arithmetic Logic Unit)** — Performs math and logic operations
- **Control Unit** — Fetches, decodes, and dispatches instructions
- **Registers** — Tiny, ultra-fast storage inside the CPU (nanosecond access)
- **Cache (L1, L2, L3)** — Small, fast memory between registers and RAM

The CPU executes a **fetch-decode-execute cycle** billions of times per second. Clock speed (e.g., 3.5 GHz) measures cycles per second. Modern CPUs have multiple **cores**, each capable of independent execution.

### RAM (Random Access Memory)

- Volatile storage (data is lost when power is off)
- Much faster than disk (nanoseconds vs milliseconds)
- Holds the operating system, running programs, and their data
- Java's **heap** and **stack** live here
- Typical modern machines: 8 GB to 64 GB

### Storage (Disk)

- Non-volatile (data persists without power)
- HDD (mechanical, slower) vs SSD (solid-state, faster)
- Holds the operating system, applications, databases, files
- Typical modern machines: 256 GB to 2 TB SSD

### The Memory Hierarchy (Critical for Backend Engineers)

From fastest to slowest:

```
Registers        ~1 ns      (inside CPU)
L1 Cache         ~1-2 ns    (inside CPU)
L2 Cache         ~3-10 ns   (inside CPU)
L3 Cache         ~10-20 ns  (shared across cores)
RAM              ~50-100 ns
SSD              ~50-150 us (microseconds)
HDD              ~1-10 ms   (milliseconds)
Network (local)  ~100 us
Network (remote) ~10-100 ms
```

This hierarchy explains why caching exists, why database queries are expensive, and why in-memory data structures outperform disk reads. You will revisit this in system design.

### Processes and Threads (Conceptual)

- A **process** is a running instance of a program. It has its own memory space.
- A **thread** is a unit of execution within a process. Threads share the process's memory.
- A single Java application runs as one process (the JVM) but can spawn many threads.
- Java's concurrency model is built on threads. This is a major topic in Phase 01.

### What a Program Is

A program is a set of instructions written by a human, translated into a form the machine can execute.

The translation path differs by language:

**Compiled languages (C, Go, Rust):**
```
Source Code → Compiler → Machine Code (binary) → CPU executes directly
```

**Interpreted languages (Python, Ruby):**
```
Source Code → Interpreter → Executes line by line at runtime
```

**Java's hybrid model (compiled + interpreted + JIT):**
```
Source Code (.java)
    → javac compiler
    → Bytecode (.class files)
    → JVM loads bytecode
    → Interpreter executes bytecode
    → JIT compiler hotspots bytecode into native machine code at runtime
```

This is why Java is described as "write once, run anywhere." The bytecode is platform-independent. The JVM is platform-specific. As long as a JVM exists for the target OS, the same `.class` files run without recompilation.

### The JVM, JDK, and JRE

These three terms are foundational and frequently confused:

- **JVM (Java Virtual Machine)** — The runtime engine that executes bytecode. It handles memory management, garbage collection, and JIT compilation. Each operating system has its own JVM implementation.

- **JRE (Java Runtime Environment)** — The JVM plus the standard class libraries (java.lang, java.util, java.io, etc.). This is what you need to *run* Java programs. Oracle has deprecated standalone JRE distributions since Java 11.

- **JDK (Java Development Kit)** — The JRE plus development tools: `javac` (compiler), `javadoc`, `jdb` (debugger), `jar`, `jshell`, `jlink`, `jpackage`. This is what you need to *write and compile* Java programs.

As a backend engineer, you install the JDK. The JDK contains everything.

### Java's Position in the Landscape

| Language | Primary Use | Execution Model | Type System |
|----------|------------|-----------------|-------------|
| Java | Enterprise backend, fintech, Android | JVM bytecode | Static, strong |
| Python | Data science, ML, scripting, web | Interpreted | Dynamic, strong |
| Go | Cloud infrastructure, microservices | Compiled to binary | Static, strong |
| C | Operating systems, embedded | Compiled to binary | Static, weak |
| JavaScript | Frontend, Node.js backend | Interpreted/JIT | Dynamic, weak |
| Rust | Systems programming, performance | Compiled to binary | Static, strong |

Java dominates enterprise backend and fintech because of its maturity, type safety, concurrency model, massive ecosystem (Spring), and long-term support commitments from Oracle and the OpenJDK community.

---

## Code Examples

Even at this conceptual stage, here is what the compilation and execution pipeline looks like in practice:

**Step 1: Write source code**

```java
// HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Step 2: Compile to bytecode**

```bash
javac HelloWorld.java
# Produces HelloWorld.class (bytecode)
```

**Step 3: Run on the JVM**

```bash
java HelloWorld
# Output: Hello, World!
```

**Step 4: Inspect the bytecode (optional, for understanding)**

```bash
javap -c HelloWorld
```

Output (simplified):

```
public static void main(java.lang.String[]);
    Code:
       0: getstatic     #7    // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #13   // String Hello, World!
       5: invokevirtual #15   // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
```

This bytecode is what the JVM interprets and eventually JIT-compiles to native machine code. You do not need to read bytecode daily, but understanding that it exists helps you reason about performance later.

**Verifying your JDK installation:**

```bash
java -version
# openjdk version "21.0.2" 2024-01-16 LTS
# OpenJDK Runtime Environment Temurin-21.0.2+13 (build 21.0.2+13-LTS)
# OpenJDK 64-Bit Server VM Temurin-21.0.2+13 (build 21.0.2+13-LTS, mixed mode, sharing)

javac -version
# javac 21.0.2
```

---

## Important Notes

- The JVM is not just for Java. Kotlin, Scala, Clojure, and Groovy all compile to JVM bytecode. Understanding the JVM makes you effective across all these languages.
- Java's "write once, run anywhere" promise holds true in practice for server-side applications. The same JAR file runs on Linux, macOS, and Windows without modification.
- The JIT compiler is a major reason Java performance rivals C++ in long-running server applications. Short-lived scripts (where Python excels) do not benefit from JIT warmup.
- Memory management in Java is automatic (garbage collection). You do not manually allocate and free memory as in C. However, understanding how the GC works is critical for high-performance fintech systems. This is covered in Phase 01 and revisited in Phase 08.
- The memory hierarchy numbers above are approximate but the relative orders of magnitude are stable. A network call to a database is roughly 100,000x slower than reading from L1 cache. This fact drives nearly every caching and architectural decision you will make.

---

## Practice

1. Open your terminal. Run `java -version` and `javac -version`. Record the output in your Obsidian vault.
2. Write the `HelloWorld.java` program above. Compile it with `javac`. Run it with `java`. Verify the output.
3. Run `javap -c HelloWorld` and study the bytecode output. You will not understand all of it yet. Note what you recognize and what you do not.
4. Calculate: if a RAM access takes 100 ns and a disk access takes 1 ms, how many times slower is disk? Write the answer in your notes.
5. In your own words, write a 3-5 sentence explanation of the difference between the JDK, JRE, and JVM. Add this to your glossary note.

---

## References

- Oracle Java Documentation: https://docs.oracle.com/en/java/
- OpenJDK: https://openjdk.org/
- "Effective Java" by Joshua Bloch — Chapter 1 (read when you reach Phase 01)
- JEP 1: JDK Enhancement Proposals: https://openjdk.org/jeps/1
