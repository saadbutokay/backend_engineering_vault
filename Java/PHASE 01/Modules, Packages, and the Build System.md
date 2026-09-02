## Overview

This section is a consolidation and expansion of the packaging concepts introduced in 01.09, with a focus on how packages and modules fit into the larger build system that produces deployable Java applications. In 01.09, we covered the mechanics of packages, imports, JPMS directives, and JAR structure. This section takes a step back to look at the conceptual model: how source code is organized, how the compiler and JVM find classes, how build tools automate the process, and how these pieces fit together to produce the fat JARs and Docker images you will deploy to production. Understanding this conceptual model is essential for diagnosing build failures, dependency conflicts, and classpath issues — problems that occupy a significant portion of every Java developer's debugging time.

---

## Core Concepts

### The Compilation and Execution Pipeline

Java has a three-stage pipeline from source code to running application:

```
Source Code (.java files)
        │
        │ javac (Java Compiler)
        ▼
Bytecode (.class files)
        │
        │ jar (or automatic in build tools)
        ▼
Distribution Archive (.jar, .war, .ear)
        │
        │ java (JVM)
        ▼
Running Application
```

Each stage has its own concerns:

- **Compilation** requires access to all classes referenced in the source code. This is the **compile-time classpath**.
- **Packaging** bundles compiled classes with resources and metadata into a distributable archive.
- **Execution** requires the JVM to locate all classes needed at runtime. This is the **runtime classpath**.

The compile-time and runtime classpaths are often different. A class may be needed at compile time (`compileOnly` in Gradle, `provided` in Maven) but not at runtime (because the runtime environment provides it). Conversely, a class may be needed at runtime (`runtime` scope) but not at compile time (because your code accesses it via reflection or a dynamically loaded configuration).

### The Package as an Organizational Unit

Packages serve four purposes:

1. **Namespace management.** Prevents naming collisions. Two classes named `Transaction` can coexist if they are in different packages (`com.example.payment.Transaction` and `com.example.audit.Transaction`).

2. **Access control.** The package-private (default) access modifier makes a class or member visible only within its package. This is the foundation of package-level encapsulation.

3. **Physical organization.** The package structure maps directly to the directory structure. This makes it easy to locate a class by its fully qualified name and vice versa.

4. **Logical grouping.** Related classes are placed together, communicating intent to future readers. A file in `com.example.fintech.payment` is clearly related to payment processing.

**Package granularity guidelines:**

- **Too coarse:** All classes in `com.example`. Loses all organizational benefit.
- **Too fine:** Each class in its own package. Loses cohesion and creates administrative overhead.
- **Right:** Group classes that change together, are used together, or represent a single concept. Aim for 5-20 classes per package in mature applications.

### The Module as an Encapsulation Boundary

A module is a higher-level grouping of packages. It adds two capabilities beyond packages:

1. **Explicit dependency declaration.** A module declares which other modules it requires. The compiler and JVM verify that all dependencies are present. This eliminates entire categories of `NoClassDefFoundError` at compile time.

2. **Strong encapsulation.** A module can declare that only specific packages are exported (visible to other modules). Non-exported packages are truly internal — they cannot be accessed by any code outside the module, even via reflection (unless explicitly `opens`).

**When modules matter conceptually:**

Modules answer questions that packages cannot:

- "This library has 500 public classes. Which ones am I supposed to use?" (Answer: only the ones in exported packages.)
- "I updated my dependency and now my build breaks with cryptic errors. Which of the dependency's classes did I accidentally rely on?" (Answer: modules force this to be explicit.)
- "How do I prevent our microservices team from accidentally importing our internal database utility?" (Answer: put it in a non-exported package of a module.)

**The reality in modern Java:**

Despite the theoretical benefits, most production Spring Boot applications do not use JPMS. Reasons:

- Spring Boot's classpath scanning and auto-configuration predate JPMS and are not fully compatible with modular projects.
- Adding `module-info.java` files to every module in a large application is administrative overhead.
- Many libraries (especially older ones) are not modularized. Mixing modular and non-modular JARs produces "automatic modules" with subtle behavior differences.
- The build tool ecosystem (Maven, Gradle) supports JPMS but not as smoothly as the classpath.

The JDK itself is fully modular (since Java 9). Understanding how the JDK is organized helps you interpret errors like `module java.base does not export sun.misc to unnamed module`. But writing your own `module-info.java` files is uncommon in application code.

### The Build System

A build system automates the process of compiling source code, resolving dependencies, running tests, and packaging deployable artifacts. In Java, the two dominant build systems are Maven and Gradle. Both will be covered in depth in Phase 02. Here, we focus on the conceptual model.

**What a build tool does:**

```
1. Read project configuration (pom.xml for Maven, build.gradle for Gradle)
2. Download dependencies from repositories (Maven Central, private Nexus/Artifactory)
3. Resolve dependency versions and detect conflicts
4. Compile source code with the resolved compile-time classpath
5. Run unit tests
6. Package the compiled code + resources + dependencies into a distributable artifact
7. Optionally: run integration tests, generate documentation, publish artifacts
```

**Why manual compilation does not scale:**

For a "Hello World" program, `javac` and `java` are sufficient. For a real application with 50-500 dependencies (Spring Boot, Jackson, Hibernate, PostgreSQL driver, Kafka client, etc.), manual dependency management is impossible:

- Each dependency has its own transitive dependencies (dependencies of dependencies).
- Dependencies must be downloaded from repositories and stored locally.
- Versions must be compatible with each other.
- The classpath must include every dependency's JAR file in the correct order.

A build tool handles all of this automatically. When you declare `spring-boot-starter-web` as a dependency, the build tool downloads Spring Boot, Spring MVC, Tomcat, Jackson, and 20-30 other JARs, resolves version conflicts, and constructs the classpath for you.

### Dependency Scopes

Not every dependency is needed in every phase. Build tools distinguish between dependency scopes to keep artifacts lean and avoid conflicts.

**Common scopes (Maven terminology, similar concepts in Gradle):**

| Scope | Compile-time | Runtime | Test | In final JAR | Example |
|-------|-------------|---------|------|--------------|---------|
| `compile` (default) | Yes | Yes | Yes | Yes | `spring-boot-starter-web`, `postgresql` |
| `provided` | Yes | No | Yes | No | `servlet-api` (provided by Tomcat) |
| `runtime` | No | Yes | Yes | Yes | JDBC drivers |
| `test` | No | No | Yes | No | `junit-jupiter`, `mockito-core` |
| `system` | Yes | Yes | Yes | No | Local JAR files (rare, discouraged) |
| `import` | Bill of Materials only | No | No | No | Spring Boot BOM |

**Gradle equivalent:**

| Gradle Configuration | Maven Scope | Purpose |
|---------------------|-------------|---------|
| `implementation` | `compile` | Available at compile and runtime, not exposed to consumers |
| `api` | `compile` | Available at compile and runtime, exposed to consumers |
| `compileOnly` | `provided` | Available at compile time only |
| `runtimeOnly` | `runtime` | Available at runtime only |
| `testImplementation` | `test` | Available in test source set only |

**Why scopes matter:**

- A test framework like JUnit is only needed to compile and run tests. Including it in the production JAR would bloat the artifact and expose test utilities to production code.
- A servlet API is needed at compile time so your code can reference `HttpServletRequest`, but at runtime, the servlet container (Tomcat, Jetty) provides its own implementation. Including it in the JAR would cause version conflicts.
- A JDBC driver is needed at runtime but not at compile time (your code uses the JDBC API, not the driver's classes directly). This allows swapping the driver without recompiling.

### Transitive Dependencies

When you depend on a library, you automatically depend on the libraries that library depends on. This is called **transitive dependency resolution**.

```
Your App
├── spring-boot-starter-web
│   ├── spring-boot
│   │   └── spring-core
│   ├── spring-web
│   │   └── spring-beans
│   ├── spring-webmvc
│   ├── tomcat-embed-core
│   ├── tomcat-embed-el
│   └── jackson-databind
│       ├── jackson-core
│       ├── jackson-annotations
│       └── ...
├── postgresql
│   └── (no dependencies)
└── redis.clients:jedis
    └── slf4j-api
```

Your `pom.xml` declares 3 dependencies. Your actual classpath contains 30-50 JARs after transitive resolution.

**Benefits:**
- You do not need to know every JAR your dependencies need.
- Adding a new dependency automatically pulls in everything required.

**Problems:**
- Version conflicts. Two of your dependencies may require different versions of the same transitive dependency.
- Bloat. Your JAR includes libraries you never directly use.
- Security. A vulnerability in a transitive dependency is a vulnerability in your application, even if you never wrote code that uses it.

**Version conflict resolution:**

When two dependencies require different versions of the same library, the build tool must choose one. Different tools have different strategies:

- **Maven:** Nearest wins. The version closest to the root in the dependency tree is chosen.
- **Gradle:** Highest version wins by default. Can be configured with resolution strategies.

Both tools allow you to override the resolved version explicitly using a **Bill of Materials (BOM)** or dependency management section. Spring Boot's BOM specifies compatible versions for hundreds of libraries, eliminating most conflicts.

**Inspecting the dependency tree:**

```bash
# Maven
mvn dependency:tree

# Gradle
gradle dependencies
```

Reading these outputs is a critical skill for diagnosing classpath issues.

### The Classpath in Depth

The classpath is a list of locations (directories and JAR files) where the JVM searches for classes at runtime. It is the runtime equivalent of the module path in JPMS.

**How the JVM resolves a class:**

When your code references `com.example.service.PaymentService`, the JVM:

1. Consults the **Bootstrap ClassLoader** for core JDK classes (`java.lang.*`, etc.).
2. If not found, consults the **Platform ClassLoader** for platform module classes.
3. If not found, consults the **Application ClassLoader** for classpath classes.
4. If still not found, throws `NoClassDefFoundError` or `ClassNotFoundException`.

The Application ClassLoader searches each entry on the classpath in order:

```
classpath entries: /app/classes:/lib/spring-boot.jar:/lib/jackson.jar:/lib/postgresql.jar
```

For `com.example.service.PaymentService`:
- Look for `/app/classes/com/example/service/PaymentService.class` — found? Use it. Not found? Continue.
- Look inside `spring-boot.jar` for `com/example/service/PaymentService.class` — usually not found.
- Continue with each JAR.

**Classpath ordering matters:**

If two JARs contain a class with the same fully qualified name, the JVM uses the one from the earlier classpath entry. This is the root of "JAR hell" — version conflicts.

```
classpath: lib/jackson-2.10.jar:lib/jackson-2.17.jar
```

The JVM uses classes from `jackson-2.10.jar`, even if you intended to use 2.17. Build tools prevent this by ensuring only one version of each library is on the classpath after conflict resolution.

**The "shaded" or "shadow" pattern:**

Sometimes different applications on the same JVM require conflicting versions of the same library. The solution is to **shade** (or **shadow**) dependencies — rename their packages to avoid conflicts.

```
Original: com.google.guava.common.collect.ImmutableList
Shaded:   com.example.myapp.shaded.google.guava.common.collect.ImmutableList
```

This is done with the Maven Shade Plugin or the Gradle Shadow Plugin. Fat JARs for Spring Boot are not shaded — they use a special classloader instead — but library JARs distributed to third parties often are.

### The Runtime Environment

Once your code is compiled and packaged, it runs inside the JVM. The JVM startup process:

```
1. Read JVM arguments (heap size, GC settings, system properties)
2. Bootstrap the JVM (load core classes, initialize primitive types)
3. Load the main class specified by:
   - The -cp/--class-path flag (classpath)
   - The --module-path flag (modules)
   - The -jar flag (JAR manifest)
4. Initialize static fields and run static initializers
5. Call the main() method
```

**JVM configuration for production:**

```bash
java \
  -Xms1g \                              # Initial heap size
  -Xmx4g \                              # Maximum heap size
  -XX:+UseG1GC \                        # Garbage collector
  -XX:MaxGCPauseMillis=200 \            # GC pause target
  -XX:+HeapDumpOnOutOfMemoryError \     # Dump heap on OOM
  -XX:HeapDumpPath=/var/log/heapdumps \
  -Dspring.profiles.active=production \  # System property
  -jar myapp.jar
```

These flags are not part of your application code but are critical to how it runs. Managing them is a DevOps concern (covered in Phase 07) but backend engineers must understand what they do.

### The Deployment Artifact

The final output of a build is a **deployment artifact** — a self-contained package ready to run in a target environment.

**Common artifact types:**

| Type | Description | Use Case |
|------|-------------|----------|
| **JAR** | Java Archive | Libraries, standalone applications |
| **Fat/Uber JAR** | JAR containing all dependencies | Spring Boot applications, microservices |
| **WAR** | Web Application Archive | Deployment to external servlet containers (Tomcat, Jetty) |
| **EAR** | Enterprise Application Archive | Legacy Java EE application servers |
| **Docker Image** | Container image with JVM + application | Modern cloud deployments (Kubernetes, ECS) |
| **Native Image** | Ahead-of-time compiled binary | GraalVM native applications (fast startup, low memory) |

**The Spring Boot fat JAR:**

A Spring Boot fat JAR contains:

```
myapp-1.0.0.jar
├── META-INF/
│   └── MANIFEST.MF                (specifies Spring Boot's launcher as Main-Class)
├── org/springframework/boot/loader/  (Spring Boot's custom classloader)
├── BOOT-INF/
│   ├── classes/                   (your application code and resources)
│   │   ├── com/example/myapp/Application.class
│   │   └── application.yml
│   └── lib/                        (all dependency JARs)
│       ├── spring-boot-3.2.0.jar
│       ├── spring-web-6.1.0.jar
│       ├── jackson-databind-2.17.0.jar
│       ├── postgresql-42.7.0.jar
│       └── ... (40+ more JARs)
```

When you run `java -jar myapp.jar`, Spring Boot's `JarLauncher` reads the nested JARs from `BOOT-INF/lib/`, sets up a custom classloader, and calls your `Application.main()`. This entire mechanism is transparent to you as a developer.

**The Docker image:**

For containerized deployments, the artifact is a Docker image:

```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY target/myapp-1.0.0.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

The image contains:
- A minimal JRE (only the runtime, not the JDK)
- Your fat JAR
- OS-level dependencies (libc, etc.)

This is the standard deployment unit for modern cloud-native applications. Covered in depth in Phase 07.

### Build Lifecycles

Both Maven and Gradle define a **lifecycle** — a sequence of phases that execute in order. Understanding the lifecycle helps you know what happens when you run a build command.

**Maven lifecycle (simplified):**

```
validate    → Verify project structure and configuration
compile     → Compile main source code
test        → Compile and run tests
package     → Package compiled code into a JAR/WAR
verify      → Run integration tests
install     → Install artifact into local Maven repository (~/.m2/repository)
deploy      → Upload artifact to remote repository
```

Running `mvn package` executes all phases up to and including `package`. Running `mvn install` executes everything up to `install`.

**Gradle lifecycle (task-based, not phase-based):**

Gradle uses a **directed acyclic graph** of tasks. Each task declares its dependencies. Running `gradle build` executes all tasks required for the `build` task, including `compileJava`, `test`, `jar`, `check`, and their transitive dependencies.

**Common commands:**

```bash
# Maven
mvn clean              # Delete the target/ directory
mvn compile            # Compile source code
mvn test               # Compile and run unit tests
mvn package            # Build the JAR
mvn install            # Install to local repo
mvn spring-boot:run    # Run the Spring Boot app (dev mode)

# Gradle
gradle clean           # Delete the build/ directory
gradle compileJava     # Compile source code
gradle test            # Run unit tests
gradle build           # Full build (compile, test, package)
gradle bootJar         # Build the Spring Boot fat JAR
gradle bootRun         # Run the Spring Boot app (dev mode)
```

### The Local Repository Cache

Build tools cache downloaded dependencies locally to avoid re-downloading them for every build.

**Maven local repository:** `~/.m2/repository/`

```
~/.m2/repository/
├── org/springframework/boot/
│   └── spring-boot/
│       ├── 3.2.0/
│       │   ├── spring-boot-3.2.0.jar
│       │   ├── spring-boot-3.2.0.pom
│       │   └── spring-boot-3.2.0.jar.sha1
│       └── 3.2.1/
│           └── ...
└── com/fasterxml/jackson/core/
    └── jackson-databind/
        └── 2.17.0/
            └── ...
```

**Gradle local repository:** `~/.gradle/caches/`

The structure is similar but organized differently.

**Implications:**
- The first build downloads dependencies. Subsequent builds are much faster.
- If your local cache is corrupted, you may see mysterious errors. Deleting the cache and re-downloading often resolves them.
- In CI/CD environments, the cache is often shared across builds to reduce build times.

### Environment Configuration

Applications behave differently in development, staging, and production. Java provides several mechanisms to inject environment-specific configuration:

**1. System properties (`-D` flags):**

```bash
java -Dspring.profiles.active=production -Dserver.port=8080 -jar myapp.jar
```

```java
String profile = System.getProperty("spring.profiles.active");  // "production"
```

**2. Environment variables:**

```bash
export DATABASE_URL="jdbc:postgresql://prod-db:5432/payments"
export DATABASE_PASSWORD="secret"
java -jar myapp.jar
```

```java
String dbUrl = System.getenv("DATABASE_URL");
```

**3. Configuration files:**

- `application.properties` or `application.yml` (Spring Boot)
- `logback.xml` (logging configuration)
- Custom properties files loaded via `Properties.load()`

**4. External configuration servers:**

- Spring Cloud Config
- HashiCorp Vault
- AWS Parameter Store / Secrets Manager

**Precedence (in Spring Boot):**

Higher-priority sources override lower-priority sources:

1. Command-line arguments (highest)
2. System properties (`-D` flags)
3. Environment variables
4. `application-{profile}.yml`
5. `application.yml`
6. Default values (lowest)

This precedence order is important. It allows you to deploy the same JAR to multiple environments and configure each environment with environment variables or command-line arguments.

---

## Code Examples

**A complete project structure showing packages, resources, and build configuration:**

```
myapp/
├── pom.xml                                    # Maven build configuration
├── mvnw                                        # Maven wrapper (executable)
├── .gitignore                                  # Git ignore rules
├── README.md
├── Dockerfile
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── fintech/
│   │   │               ├── Application.java              # Main entry point
│   │   │               ├── config/
│   │   │               │   ├── DatabaseConfig.java
│   │   │               │   ├── SecurityConfig.java
│   │   │               │   └── WebConfig.java
│   │   │               ├── controller/
│   │   │               │   ├── AccountController.java
│   │   │               │   └── PaymentController.java
│   │   │               ├── service/
│   │   │               │   ├── AccountService.java
│   │   │               │   └── PaymentService.java
│   │   │               ├── repository/
│   │   │               │   ├── AccountRepository.java
│   │   │               │   └── TransactionRepository.java
│   │   │               ├── model/
│   │   │               │   ├── Account.java
│   │   │               │   └── Transaction.java
│   │   │               ├── dto/
│   │   │               │   ├── PaymentRequest.java
│   │   │               │   └── PaymentResponse.java
│   │   │               └── exception/
│   │   │                   ├── FintechException.java
│   │   │                   └── GlobalExceptionHandler.java
│   │   └── resources/
│   │       ├── application.yml                # Main configuration
│   │       ├── application-dev.yml            # Dev profile
│   │       ├── application-prod.yml           # Prod profile
│   │       ├── logback-spring.xml             # Logging config
│   │       ├── db/
│   │       │   └── migration/                 # Flyway migrations
│   │       │       ├── V1__create_accounts.sql
│   │       │       └── V2__create_transactions.sql
│   │       └── static/                        # Static web resources
│   └── test/
│       ├── java/
│       │   └── com/
│       │       └── example/
│       │           └── fintech/
│       │               ├── controller/
│       │               │   └── AccountControllerTest.java
│       │               └── service/
│       │                   └── PaymentServiceTest.java
│       └── resources/
│           └── application-test.yml           # Test configuration
└── target/                                    # Build output (gitignored)
    ├── classes/                               # Compiled main classes
    ├── test-classes/                          # Compiled test classes
    ├── myapp-1.0.0.jar                        # Final artifact
    └── myapp-1.0.0.jar.original               # Original JAR (before Spring Boot repackaging)
```

**A minimal pom.xml illustrating the build system model:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <!-- Project identity -->
    <groupId>com.example</groupId>
    <artifactId>fintech-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <!-- Inherit sensible defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <!-- Project-wide properties -->
    <properties>
        <java.version>21</java.version>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- Dependencies with explicit scopes -->
    <dependencies>
        <!-- Compile scope (default) — needed at compile and runtime -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- Runtime scope — needed only at runtime -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Test scope — needed only for tests -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <version>1.19.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- Build configuration -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <!-- Repackages the JAR as a Spring Boot fat JAR -->
            </plugin>
        </plugins>
    </build>
</project>
```

**Inspecting a build:**

```bash
# See all dependencies (including transitive)
mvn dependency:tree

# Output (abbreviated):
# [INFO] com.example:fintech-app:jar:1.0.0
# [INFO] +- org.springframework.boot:spring-boot-starter-web:jar:3.2.0:compile
# [INFO] |  +- org.springframework.boot:spring-boot-starter:jar:3.2.0:compile
# [INFO] |  |  +- org.springframework.boot:spring-boot:jar:3.2.0:compile
# [INFO] |  |  +- org.springframework:spring-core:jar:6.1.0:compile
# [INFO] |  |  +- ch.qos.logback:logback-classic:jar:1.4.11:compile
# [INFO] |  +- org.springframework:spring-web:jar:6.1.0:compile
# [INFO] |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:10.1.16:compile
# [INFO] |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.15.3:compile
# ...

# See dependencies that have known vulnerabilities
mvn dependency:analyze

# See which dependencies are actually used
mvn dependency:analyze -DignoreNonCompile=true

# Build and see the resulting JAR
mvn clean package
ls -la target/
# fintech-app-1.0.0.jar         <-- Fat JAR ready to deploy
# fintech-app-1.0.0.jar.original <-- Original JAR without dependencies

# Run the JAR
java -jar target/fintech-app-1.0.0.jar

# Inspect the JAR contents
jar tf target/fintech-app-1.0.0.jar | head -20
```

---

## Important Notes

- This section is intentionally conceptual because Phase 02 covers the practical mechanics of Maven, Gradle, and dependency management in depth. The goal here is to understand the model, not to memorize configuration syntax.
- The most common failure mode for Java developers is confusion between compile-time and runtime classpaths. If a class compiles but throws `NoClassDefFoundError` at runtime, it means the class is on the compile-time classpath but not the runtime classpath. This often happens when a dependency is declared as `provided` or `test` scope but is needed at runtime.
- Dependency version conflicts are one of the most frequent sources of production bugs. A method that exists in version 2.10 of a library may have been removed in 2.17. If your build resolves to 2.10 but you wrote code assuming 2.17, you get `NoSuchMethodError` at runtime. Always inspect the dependency tree when adding new dependencies to a mature project.
- The `provided` scope exists for a specific reason: some libraries are provided by the runtime environment (servlet containers provide the servlet API, application servers provide Java EE APIs). Including them in the deployment artifact causes classloader conflicts and version incompatibilities.
- Spring Boot's fat JAR mechanism is a workaround for the fact that Java's built-in JAR format does not natively support nested JARs. Spring Boot's `JarLauncher` reads the nested JARs from `BOOT-INF/lib/` and constructs a custom classloader that can find classes inside them. This is why you cannot simply extract a Spring Boot JAR and expect Java's default classloader to find the dependency classes.
- The local Maven/Gradle repository (`~/.m2/repository/` or `~/.gradle/caches/`) can grow to several gigabytes over time as you work on multiple projects with different dependency versions. If your disk fills up, you can safely delete this directory — it will be re-downloaded on the next build.
- The `.gitignore` file must exclude the `target/` (Maven) or `build/` (Gradle) directory. Committing build outputs pollutes the repository, causes merge conflicts, and slows down cloning. The convention is to check in only source code, configuration files, and build scripts — never generated files.
- Environment-specific configuration should never be hardcoded. A JAR built for staging must be deployable to production without modification. Use system properties, environment variables, or external configuration servers to inject environment-specific values. This is the foundation of the Twelve-Factor App methodology (covered in Phase 09).
- The distinction between `implementation` and `api` in Gradle is important for library authors. If you declare a dependency as `api`, it becomes part of your library's public API, and consumers of your library will also have that dependency on their compile-time classpath. If you declare it as `implementation`, consumers do not see it. For applications (as opposed to libraries), always use `implementation` to minimize coupling.
- Reading a dependency tree is a critical debugging skill. When you encounter a `ClassCastException` or `NoSuchMethodError` at runtime, the first step is usually to run `mvn dependency:tree` or `gradle dependencies` and look for version conflicts on the affected class.
- Build tools support **build caching** to speed up incremental builds. Gradle's cache is particularly aggressive — it can share cache entries across projects and even across machines (via remote build caches). Understanding when and why builds are cached (or not) helps diagnose "it built yesterday, why is it broken now?" scenarios.
- The compilation pipeline is not just `javac`. Modern Java builds also run annotation processors (Lombok, MapStruct, JPA metamodel generators), which can generate additional source code during compilation. If your IDE cannot find a generated class, it usually means the annotation processor is not configured correctly.

---

## Practice

1. Create a Maven project from scratch (without using Spring Initializr). Write a `pom.xml` that declares dependencies on JUnit 5 (test scope), Jackson (compile scope), and PostgreSQL driver (runtime scope). Compile the project and run `mvn dependency:tree`. Identify the transitive dependencies of each direct dependency.

2. Deliberately introduce a dependency conflict. Add two dependencies that require different versions of the same transitive dependency (e.g., older Spring Web and newer Jackson). Run `mvn dependency:tree -Dverbose` to see how Maven resolves the conflict. Override the resolution using `<dependencyManagement>`.

3. Build a Spring Boot fat JAR. Extract it using `unzip -d extracted target/myapp.jar`. Explore the directory structure. Locate one of your application classes and one of the dependency JARs. Read the `MANIFEST.MF` and identify the `Main-Class` and `Start-Class` entries.

4. Create a project with three packages: `com.example.api`, `com.example.internal`, and `com.example.util`. Place a public class in each package. Demonstrate that classes in `com.example.api` cannot access package-private members of `com.example.internal`, even though they are in the same top-level package hierarchy.

5. Write a Dockerfile that packages a Spring Boot fat JAR into a Docker image. Use `eclipse-temurin:21-jre-alpine` as the base image. Build the image with `docker build`. Run it with `docker run -p 8080:8080 myapp`. Verify it works.

6. Configure the same Spring Boot application to run with three different profiles: `dev`, `staging`, and `prod`. Each profile should use a different database URL. Demonstrate switching profiles using: (a) a system property (`-Dspring.profiles.active=prod`), (b) an environment variable (`SPRING_PROFILES_ACTIVE=prod`), and (c) a command-line argument (`--spring.profiles.active=prod`).

7. In your Obsidian vault, draw a diagram showing the flow from source code to running application. Include: source files, `javac`, `.class` files, dependency resolution, JAR packaging, JVM startup, and classloader hierarchy. This is a reference you will consult when debugging build and runtime issues.

---

## References

- Maven Getting Started Guide: https://maven.apache.org/guides/getting-started/index.html
- Maven POM Reference: https://maven.apache.org/pom.html
- Maven Dependency Scopes: https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope
- Gradle User Manual: https://docs.gradle.org/current/userguide/userguide.html
- Gradle Java Plugin: https://docs.gradle.org/current/userguide/java_plugin.html
- Spring Boot Executable JAR Format: https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html
- Twelve-Factor App Methodology: https://12factor.net/
- Java Class Loading Documentation: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ClassLoader.html
- "Java Application Architecture" by Kirk Knoernschild (for module-level design principles)
