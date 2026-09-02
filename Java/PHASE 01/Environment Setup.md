## Overview

Before writing any Java code, you need a working development environment. This means installing a JDK, understanding the JVM architecture that will execute your programs, and learning how to compile and run Java from the command line. Every IDE, build tool, and framework you use later builds on top of this foundation. If your environment is misconfigured, nothing else will work. Get this right first.

---

## Core Concepts

### JDK Distributions

The JDK (Java Development Kit) is not a single product. Multiple vendors produce compatible JDK builds from the same OpenJDK source code. They differ in licensing, support, and performance optimizations.

**Major distributions:**

| Distribution | Vendor | License | Notes |
|-------------|--------|---------|-------|
| Oracle JDK | Oracle | Commercial (free for dev) | The original. Requires license for production use in some cases. |
| Eclipse Temurin (Adoptium) | Eclipse Foundation | GPLv2 + CE | The community standard. Free for all uses. Recommended default. |
| Amazon Corretto | Amazon | GPLv2 + CE | Free. Long-term support. Optimized for AWS. |
| Azul Zulu | Azul Systems | GPLv2 + CE | Free community builds. Commercial support available. |
| GraalVM | Oracle | GPLv2 + CE / Commercial | Supports ahead-of-time compilation to native images. |
| Microsoft Build of OpenJDK | Microsoft | GPLv2 + CE | Free. Optimized for Azure. |

**For this roadmap, use Eclipse Temurin.** It is free, widely supported, and the default in most enterprise environments. Switch to GraalVM later when you study native compilation in Phase 08.

### JDK Versions and LTS

Java releases a new version every six months. Not all versions receive long-term support.

**LTS (Long-Term Support) versions** receive security patches and bug fixes for years. Non-LTS versions are supported for only six months.

| Version | Release | LTS | Support Until |
|---------|---------|-----|---------------|
| Java 8 | 2014 | Yes | 2030+ (still widely used in legacy fintech) |
| Java 11 | 2018 | Yes | 2026+ |
| Java 17 | 2021 | Yes | 2029+ |
| Java 21 | 2023 | Yes | 2031+ |
| Java 22 | 2024 | No | September 2024 (already EOL) |
| Java 23 | 2024 | No | March 2025 |

**Use Java 21 LTS for all new projects.** It is the current standard. It includes virtual threads (Project Loom), pattern matching for switch, record patterns, and sequenced collections. You will encounter Java 8 and Java 17 in existing fintech codebases, so awareness of older versions matters, but you should learn on 21.

### SDKMAN

**SDKMAN** (Software Development Kit Manager) is a command-line tool for managing multiple JDK versions and other JVM-based tools (Maven, Gradle, etc.) on macOS and Linux. It is the Java equivalent of `pyenv` for Python.

**Why SDKMAN:**

- Install and switch between multiple JDK versions instantly.
- Manage Maven, Gradle, Spring Boot CLI, and other tools from the same interface.
- No root privileges required.
- Works in your shell without modifying system paths manually.

**Installation:**

```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

**Verify:**

```bash
sdk version
# SDKMAN 5.18.2
```

**Core commands:**

```bash
# List all available JDK distributions
sdk list java

# Install a specific JDK
sdk install java 21.0.2-tem

# List installed JDKs
sdk list java | grep installed

# Switch to a specific version (current shell only)
sdk use java 21.0.2-tem

# Set the default version (all new shells)
sdk default java 21.0.2-tem

# Install other tools
sdk install maven
sdk install gradle

# Switch Maven versions
sdk use maven 3.9.6
```

**Per-project JDK version:**

Create a `.sdkmanrc` file in your project root:

```
java=21.0.2-tem
maven=3.9.6
```

Then run `sdk env` in that directory to switch automatically. This is critical when you work on multiple projects that require different Java versions.

### JVM Architecture

The JVM is the engine that executes your Java programs. Understanding its architecture helps you reason about performance, memory, and debugging throughout your career.

**The JVM has three major subsystems:**

```
┌─────────────────────────────────────────────────┐
│                    JVM                          │
│                                                 │
│  ┌───────────────┐  ┌────────────────────────┐  │
│  │  ClassLoader  │  │   Runtime Data Areas   │  │
│  │  Subsystem    │  │                        │  │
│  │               │  │  ┌──────┐  ┌────────┐  │  │
│  │  Loading      │  │  │ Heap │  │ Method │  │  │
│  │  Linking      │  │  │      │  │ Area   │  │  │
│  │  Initializing │  │  └──────┘  └────────┘  │  │
│  │               │  │  ┌──────┐  ┌────────┐  │  │
│  │               │  │  │Stack │  │   PC   │  │  │
│  │               │  │  │      │  │Register│  │  │
│  └───────────────┘  │  └──────┘  └────────┘  │  │
│                     │  ┌──────────────────┐  │  │
│                     │  │ Native Method    │  │  │
│                     │  │ Stack            │  │  │
│                     │  └──────────────────┘  │  │
│                     └────────────────────────┘  │
│                                                 │
│  ┌────────────────────────────────────────────┐ │
│  │         Execution Engine                   │ │
│  │                                            │ │
│  │  ┌────────────┐  ┌──────────────────────┐  │ │
│  │  │Interpreter │  │   JIT Compiler       │  │ │
│  │  │            │  │   (C1, C2, Graal)    │  │ │
│  │  └────────────┘  └──────────────────────┘  │ │
│  │                                            │ │
│  │  ┌──────────────────────────────────────┐  │ │
│  │  │     Garbage Collector                │  │ │
│  │  │     (G1, ZGC, Shenandoah, etc.)      │  │ │
│  │  └──────────────────────────────────────┘  │ │
│  └────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

**ClassLoader Subsystem:**

The ClassLoader loads `.class` files into memory. It operates in three phases:

1. **Loading** — Reads the `.class` file bytes and creates a `Class` object in the Method Area.
2. **Linking** — Verifies the bytecode, allocates memory for static variables, and resolves symbolic references.
3. **Initializing** — Executes static initializers and static blocks.

ClassLoader hierarchy:

```
Bootstrap ClassLoader (loads java.lang.*, java.util.*, core JDK classes)
    └── Platform ClassLoader (loads platform modules)
        └── Application ClassLoader (loads your application classes from classpath)
```

**Runtime Data Areas:**

| Area | Shared? | Purpose |
|------|---------|---------|
| **Heap** | Yes (all threads) | Stores all objects and arrays. This is where `new` allocates memory. The garbage collector manages this area. |
| **Method Area (Metaspace)** | Yes (all threads) | Stores class metadata, static variables, method bytecode, constant pool. In Java 8+, backed by native memory (not heap). |
| **Stack (JVM Stack)** | No (per thread) | Stores local variables, method call frames, operand stacks. Each method call creates a new frame. When the method returns, the frame is popped. |
| **PC Register** | No (per thread) | Stores the address of the current instruction being executed by the thread. |
| **Native Method Stack** | No (per thread) | Supports native methods written in C/C++ (JNI). |

**Execution Engine:**

- **Interpreter** — Reads bytecode instructions one at a time and executes them. Fast startup, slow execution.
- **JIT (Just-In-Time) Compiler** — Identifies "hot" methods (frequently executed) and compiles them to native machine code at runtime. Slow startup, fast execution. The JVM uses both: the interpreter handles initial execution, and the JIT optimizes hot paths over time.
    - **C1 (Client Compiler)** — Fast compilation, moderate optimization. Used for less critical code.
    - **C2 (Server Compiler)** — Slow compilation, aggressive optimization. Used for hot paths.
    - **GraalVM Compiler** — Alternative JIT written in Java. Available in GraalVM distributions.
- **Garbage Collector** — Automatically reclaims memory from objects that are no longer reachable. You do not manually free memory in Java. The GC runs in the background. Different GC algorithms (G1, ZGC, Shenandoah) offer different tradeoffs between throughput and latency. Covered in depth in Phase 08.

**Why this matters now:**

When your application throws an `OutOfMemoryError: Java heap space`, you need to know that the heap is where objects live. When you see a `StackOverflowError`, you need to know that the stack is where method frames live and that infinite recursion exhausts it. When your application is slow on startup but fast after a few minutes, you need to know that the JIT compiler is warming up. These are not abstract concepts. They are daily debugging realities.

### Compiling and Running from the Command Line

Before using Maven, Gradle, or an IDE, you must understand the raw compilation and execution process.

**The two-step process:**

```
Source code (.java)  →  javac  →  Bytecode (.class)  →  java  →  Output
```

**Step 1: Write a Java source file.**

The file name must match the public class name exactly, including capitalization.

```java
// File: HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Step 2: Compile.**

```bash
javac HelloWorld.java
```

This produces `HelloWorld.class` in the same directory. The `.class` file contains bytecode, not machine code. It is platform-independent.

**Step 3: Run.**

```bash
java HelloWorld
# Output: Hello, World!
```

Note: you pass the class name, not the file name. No `.class` extension.

**Key compiler flags:**

```bash
# Specify output directory
javac -d out/ HelloWorld.java

# Specify source and target version
javac --release 21 HelloWorld.java

# Enable all warnings
javac -Xlint:all HelloWorld.java

# Compile multiple files
javac src/com/example/*.java

# Specify classpath (where to find dependencies)
javac -cp "lib/*" HelloWorld.java
```

**Key runtime flags:**

```bash
# Specify classpath
java -cp "out/:lib/*" HelloWorld

# Pass arguments to the program
java HelloWorld arg1 arg2 arg3

# Set heap size
java -Xms256m -Xmx1g HelloWorld

# Enable assertions
java -ea HelloWorld

# Run a single-file program directly (Java 11+, no compilation step)
java HelloWorld.java
```

### The Classpath

The **classpath** tells the JVM where to find `.class` files and JAR files at runtime. It is one of the most common sources of confusion for beginners.

```bash
# The JVM searches for classes in this order:
# 1. Bootstrap classes (java.lang.*, etc.) — always available
# 2. Extension/platform classes
# 3. Classpath entries — your application code and dependencies

# Set classpath via flag
java -cp "/path/to/classes:/path/to/lib/*" com.example.Main

# Set classpath via environment variable (less common, not recommended)
export CLASSPATH="/path/to/classes:/path/to/lib/*"
java com.example.Main
```

The classpath separator is `:` on macOS/Linux and `;` on Windows.

In practice, you will rarely set the classpath manually. Maven, Gradle, and your IDE handle it automatically. But understanding what the classpath is will save you hours of debugging when something breaks.

### File Types

| Extension | What It Is |
|-----------|-----------|
| `.java` | Source code. Human-readable. Written by you. |
| `.class` | Bytecode. Machine-readable by the JVM. Produced by `javac`. |
| `.jar` | Java ARchive. A ZIP file containing `.class` files, resources, and metadata. Distributable unit. |
| `.war` | Web Application Archive. A JAR for web applications deployed to servlet containers. |
| `.jmod` | Java Module. Used by the Java Platform Module System (Java 9+). |

### Project Folder Structure

Java projects follow a strict directory convention. This is not optional. Build tools, IDEs, and frameworks all expect this layout.

```
my-project/
├── pom.xml                          # Maven config (or build.gradle.kts for Gradle)
├── src/
│   ├── main/
│   │   ├── java/                    # Application source code
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── myapp/
│   │   │               ├── Application.java
│   │   │               ├── controller/
│   │   │               │   └── UserController.java
│   │   │               ├── service/
│   │   │               │   └── UserService.java
│   │   │               ├── repository/
│   │   │               │   └── UserRepository.java
│   │   │               └── model/
│   │   │                   └── User.java
│   │   └── resources/               # Configuration files, static assets
│   │       ├── application.yml
│   │       └── db/
│   │           └── migration/
│   │               └── V1__init.sql
│   └── test/
│       ├── java/                    # Test source code (mirrors main structure)
│       │   └── com/
│       │       └── example/
│       │           └── myapp/
│       │               ├── service/
│       │               │   └── UserServiceTest.java
│       │               └── controller/
│       │                   └── UserControllerTest.java
│       └── resources/               # Test-specific configuration
│           └── application-test.yml
├── target/                          # Build output (Maven) — generated, do not edit
│   └── classes/
│       └── com/example/myapp/
│           └── Application.class
├── .gitignore
└── README.md
```

**Key rules:**
- The directory structure under `src/main/java/` must match the package declaration. If your class declares `package com.example.myapp.service;`, the file must be at `src/main/java/com/example/myapp/service/UserService.java`.
- Test code mirrors the main code structure under `src/test/java/`.
- The `target/` directory (Maven) or `build/` directory (Gradle) is generated by the build tool. Never edit files there. Always add it to `.gitignore`.
- Resources (configuration files, SQL migrations, templates) go in `src/main/resources/`, not in the Java source directory.

### Your First Program

The `main` method is the entry point of every Java application. The JVM looks for this exact signature to start execution.

```java
public static void main(String[] args)
```

Breaking down each keyword:

- **`public`** — Accessible from outside the class. The JVM must be able to call it.
- **`static`** — Belongs to the class, not to an instance. The JVM calls it without creating an object first.
- **`void`** — Returns nothing.
- **`main`** — The method name the JVM looks for. Must be exactly `main`.
- **`String[] args`** — Command-line arguments passed to the program.

---

## Code Examples

**Complete setup verification script:**

```bash
#!/bin/bash
# verify-java-setup.sh

echo "=== Java Environment Verification ==="
echo ""

echo "1. SDKMAN version:"
sdk version
echo ""

echo "2. Installed JDKs:"
sdk list java | grep installed
echo ""

echo "3. Current Java version:"
java -version
echo ""

echo "4. Current javac version:"
javac -version
echo ""

echo "5. JAVA_HOME:"
echo $JAVA_HOME
echo ""

echo "6. Java location:"
which java
echo ""

echo "7. Maven version (if installed):"
mvn -version 2>/dev/null || echo "Maven not installed yet"
echo ""

echo "8. Gradle version (if installed):"
gradle -version 2>/dev/null || echo "Gradle not installed yet"
echo ""

echo "=== Verification Complete ==="
```

**Compiling and running a multi-file project from the command line:**

```java
// src/com/example/Greeter.java
package com.example;

public class Greeter {
    private final String name;

    public Greeter(String name) {
        this.name = name;
    }

    public String greet() {
        return "Hello, " + name + "!";
    }
}
```

```java
// src/com/example/Main.java
package com.example;

public class Main {
    public static void main(String[] args) {
        String name = args.length > 0 ? args[0] : "World";
        Greeter greeter = new Greeter(name);
        System.out.println(greeter.greet());
    }
}
```

```bash
# Compile (from the project root)
javac -d out src/com/example/Greeter.java src/com/example/Main.java

# This creates:
# out/com/example/Greeter.class
# out/com/example/Main.class

# Run
java -cp out com.example.Main
# Output: Hello, World!

java -cp out com.example.Main Alice
# Output: Hello, Alice!
```

**Running a single-file program (Java 11+):**

```java
// Script.java (no package declaration needed for single-file programs)
public class Script {
    public static void main(String[] args) {
        System.out.println("Running directly without compilation!");
        System.out.println("Java version: " + Runtime.version());
        System.out.println("Available processors: " + Runtime.getRuntime().availableProcessors());
        System.out.println("Max memory: " + Runtime.getRuntime().maxMemory() / (1024 * 1024) + " MB");
    }
}
```

```bash
# Run directly (no javac step)
java Script.java
# Output:
# Running directly without compilation!
# Java version: 21.0.2+13-LTS
# Available processors: 10
# Max memory: 4096 MB
```

This single-file execution mode is useful for quick scripts and learning. Production applications always use the compile-then-run workflow with a build tool.

**Creating and running a JAR:**

```bash
# Compile
javac -d out src/com/example/*.java

# Create a manifest file
echo "Main-Class: com.example.Main" > MANIFEST.MF

# Package into a JAR
jar cfm myapp.jar MANIFEST.MF -C out .

# Run the JAR
java -jar myapp.jar
# Output: Hello, World!

java -jar myapp.jar Bob
# Output: Hello, Bob!
```

---

## Important Notes

- Always use an LTS version of Java for production applications. Non-LTS versions stop receiving security patches after six months. In fintech, running an unsupported JDK is a compliance violation.
- SDKMAN is the recommended way to manage JDK versions on macOS and Linux. Do not install JDKs manually via `.dmg` or `.deb` files unless you have a specific reason. SDKMAN makes version switching trivial and prevents PATH conflicts.
- The `JAVA_HOME` environment variable must point to the JDK root directory, not the `bin` subdirectory. Maven, Gradle, and most IDEs rely on this variable. If `JAVA_HOME` is wrong, builds will fail with cryptic errors.
- The JVM's JIT compiler is why Java performance improves over time. A freshly started Java application is slower than the same application after five minutes of operation. This "warmup" period matters in serverless environments (AWS Lambda) where cold starts are a concern. You will revisit this in Phase 08.
- The heap is where all objects created with `new` live. The stack is where local variables and method call frames live. When you get a `StackOverflowError`, it means your call stack is too deep (usually infinite recursion). When you get an `OutOfMemoryError: Java heap space`, it means you have created more objects than the heap can hold. These are distinct problems with distinct solutions.
- The classpath is the single most common source of "it works on my machine" problems in Java. If a class is not found at runtime (`ClassNotFoundException` or `NoClassDefFoundError`), the classpath is almost always the cause. Build tools eliminate most classpath issues, but you will encounter them when debugging production deployments.
- The `src/main/java/` directory structure must match your package declarations exactly. A class in `package com.example.service;` must live in `src/main/java/com/example/service/`. If the directory and package do not match, the code will compile but fail at runtime.
- Never commit the `target/` or `build/` directory to Git. These contain compiled output that is regenerated by the build tool. Add them to `.gitignore` immediately when creating a new project.
- The `main` method signature is rigid. `public static void main(String[] args)` is the only form the JVM recognizes as an entry point. Changing the return type, removing `static`, or renaming the parameter type will cause the JVM to report `Main method not found in class`.

---

## Practice

1. Install SDKMAN if you have not already. Run `sdk version` to verify.
2. Install JDK 21 via SDKMAN: `sdk install java 21.0.2-tem`. Set it as default: `sdk default java 21.0.2-tem`.
3. Run `java -version` and `javac -version`. Verify both report version 21.
4. Set `JAVA_HOME` in your shell configuration file. Verify with `echo $JAVA_HOME`.
5. Write the `HelloWorld.java` program. Compile it with `javac`. Run it with `java`. Verify the output.
6. Write the two-file project (`Greeter.java` and `Main.java`) from the code examples. Compile both files to an `out/` directory. Run the program with and without a command-line argument.
7. Package the two-file project into a JAR and run it with `java -jar`.
8. Create the full project folder structure shown above (`src/main/java/`, `src/test/java/`, `src/main/resources/`). Create a `.gitignore` that excludes `target/`, `build/`, `.idea/`, `*.class`, and `*.jar`.
9. In your Obsidian vault, write a note explaining the difference between the heap and the stack in your own words. Include what lives in each and what errors occur when each is exhausted.

---

## References

- SDKMAN: https://sdkman.io/
- Eclipse Temurin (Adoptium): https://adoptium.net/
- Oracle JDK Documentation: https://docs.oracle.com/en/java/javase/21/
- OpenJDK: https://openjdk.org/
- JVM Specification: https://docs.oracle.com/javase/specs/jvms/se21/html/
- JEP 330 (Launch Single-File Source-Code Programs): https://openjdk.org/jeps/330
- JEP 444 (Virtual Threads, Java 21): https://openjdk.org/jeps/444
