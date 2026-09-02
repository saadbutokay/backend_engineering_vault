# THE ROADMAP

---
## PHASE 00 / ENGINEERING FUNDAMENTALS
**Follow the structure down.**
*Duration: 2-3 weeks at 6 hours/day*

This phase is identical to the Python roadmap. If you completed it there, skip it. If this is your first vault, start here. The concepts are language-agnostic.

---
### 01 - [[What Is Computing]]

```
- What a computer actually does (input, processing, storage, output)
- Binary and data representation (bits, bytes, encoding, ASCII, UTF-8, Unicode)
- CPU, RAM, storage — how they interact
- What an operating system does
- Processes vs threads (conceptual)
- What a program is (source code → compilation → bytecode → JVM execution)
- Compiled vs interpreted languages
- What Java is: compiled to bytecode, runs on JVM
- Java vs Python vs C vs Go (high-level positioning)
- JVM, JDK, JRE — what each is
```

### 02 - [[How The Internet Works]]

```
- Client-server model
- IP addresses, DNS, ports
- TCP/IP stack (conceptual)
- HTTP/HTTPS — request/response cycle
- What a URL is (scheme, host, path, query, fragment)
- What an API is (conceptual)
- JSON and XML as data interchange formats
- Latency, bandwidth, throughput (definitions)
```

### 03 - [[Operating System Basics]] (macOS and Linux)

```
- File system hierarchy (/, /home, /etc, /usr, /var, /tmp)
- Users, groups, permissions (rwx, chmod, chown)
- Environment variables
- PATH and how the shell resolves commands
- JAVA_HOME and why it matters
- Process management (ps, top, kill)
- Package managers (brew for macOS, apt for Linux)
- SSH basics
```

### 04 - [[What Is Software Engineering]]

```
- Software engineering vs programming
- Software Development Life Cycle (SDLC)
- Agile, Scrum, Kanban (overview)
- What version control is and why it matters
- What testing is and why it matters
- What deployment means
- Roles: frontend, backend, fullstack, DevOps, SRE, data engineer
- What a backend engineer specifically does day-to-day
- Why Java dominates enterprise and fintech backend
- Reading documentation as a skill (Javadoc, Oracle docs)
- How to ask technical questions
```

### 05 - [[Engineering Culture And Mindset]]

```
- How to read error messages and stack traces
- How to search for solutions effectively
- How to read source code you did not write
- Technical debt (concept)
- Code review culture
- Writing things down (why Obsidian matters)
- The compounding nature of fundamentals
```

### 06 Phase 00 Projects

```
Project: "My Machine" Documentation
- Document your machine's specs (CPU, RAM, storage, OS version)
- Map your file system hierarchy
- Set up Homebrew (macOS) or apt (Linux)
- Install JDK (via SDKMAN)
- Set JAVA_HOME
- Verify: java -version, javac -version
- Install and configure: terminal emulator, Obsidian
- Write your first 5 Obsidian notes using the templates above
- Create a personal glossary note with every new term from Phase 00
- Deliverable: A fully structured Obsidian vault ready for Phase 01
```

---
## PHASE 01 / JAVA CORE
**Follow the structure down.**
*Duration: 6-7 weeks at 6 hours/day*

Java has more upfront ceremony than Python. The type system, compilation model, memory model, and OOP depth are all heavier. This phase is longer because Java demands it. Do not rush. Every enterprise Java codebase you will ever touch relies on these fundamentals.

---
### 1.01 - [[Environment Setup]]

```
- SDKMAN: managing multiple JDK versions
- JDK distributions: Oracle, OpenJDK, Adoptium (Eclipse Temurin), Amazon Corretto, GraalVM
- JDK versions: LTS strategy (Java 17, Java 21)
- Choosing a version (Java 21 LTS recommended for new projects)
- JVM architecture:
    - ClassLoader
    - Runtime data areas (heap, stack, method area, PC register)
    - Execution engine (interpreter, JIT compiler)
    - Garbage collector (overview)
- Compiling and running from command line:
    - javac HelloWorld.java
    - java HelloWorld
    - classpath (-cp)
- .java files, .class files, .jar files
- Project folder structure (src/main/java, src/test/java convention)
- Your first program: public static void main(String[] args)
```

### 1.02 - [[Syntax And Basics]]

```
- Java syntax structure (class-based, every file needs a class)
- Comments: //, /* */, /** */ (Javadoc)
- Variables and assignment
- Naming conventions:
    - camelCase for variables and methods
    - PascalCase for classes
    - UPPER_SNAKE_CASE for constants
    - Package naming (reverse domain: com.example.project)
- Primitive data types:
    - byte (8-bit), short (16-bit), int (32-bit), long (64-bit)
    - float (32-bit), double (64-bit)
    - char (16-bit Unicode)
    - boolean
    - Default values
    - Literal suffixes (L for long, F for float, D for double)
- Wrapper classes: Integer, Long, Double, Float, Character, Boolean, Byte, Short
    - Autoboxing and unboxing
    - Caching (Integer cache -128 to 127)
    - Why primitives vs wrappers matters for performance
- Type casting:
    - Widening (implicit)
    - Narrowing (explicit)
- Operators: arithmetic, comparison, logical, assignment, bitwise, ternary
- String:
    - String immutability
    - String pool
    - String methods (length, charAt, substring, indexOf, contains, equals, equalsIgnoreCase,
      trim, strip, toUpperCase, toLowerCase, replace, split, join, format, formatted)
    - String comparison (== vs .equals())
    - StringBuilder and StringBuffer
    - Text blocks (Java 13+, """ """)
    - String.format and formatted()
- System.out.println, System.out.print, System.out.printf
- Scanner for input
- var (local variable type inference, Java 10+)
- Constants (final keyword)
```

### 1.03 - [[Control Flow]]

```
- if / else if / else
- Ternary operator
- switch statement (traditional)
- switch expressions (Java 14+, arrow syntax, yield)
- Pattern matching in switch (Java 21)
- while loop
- do-while loop
- for loop (traditional C-style)
- Enhanced for loop (for-each)
- break, continue
- Labeled break and continue
- Nested loops
```

### 1.04 - [[Arrays]]

```
- Array declaration, initialization, access
- Array length (property, not method)
- Multi-dimensional arrays
- Arrays utility class (Arrays.sort, Arrays.fill, Arrays.copyOf,
  Arrays.equals, Arrays.deepEquals, Arrays.toString, Arrays.asList)
- Array limitations (fixed size, no generics for primitives)
- Varargs (Type... args)
- Command-line arguments (args array)
```

### 1.05 - [[Object Oriented Programming]] (Deep Dive)

```
This is the most critical section for Java. Java is OOP to its core.

Classes and Objects:
    - Class declaration
    - Fields (instance variables)
    - Methods (instance methods)
    - Constructors:
        - Default constructor
        - Parameterized constructors
        - Constructor overloading
        - Constructor chaining (this())
        - Copy constructors
    - this keyword
    - static keyword:
        - Static fields
        - Static methods
        - Static blocks
        - Static imports
        - When to use static
    - Object creation (new keyword, heap allocation)
    - null and NullPointerException
    - toString() method
    - equals() and hashCode():
        - Contract between equals and hashCode
        - Implementing properly
        - Why this matters for collections
    - Immutable objects (how to create, why they matter)
    - Record classes (Java 16+):
        - Syntax
        - Generated methods (constructor, getters, equals, hashCode, toString)
        - Compact constructors
        - When to use records

Encapsulation:
    - Access modifiers: private, default (package-private), protected, public
    - Getter and setter methods
    - Encapsulation best practices
    - Information hiding principle

Inheritance:
    - extends keyword
    - Single inheritance (Java does not support multiple class inheritance)
    - super keyword (accessing parent constructor, methods, fields)
    - Constructor chaining with super()
    - Method overriding:
        - @Override annotation
        - Rules for overriding
        - Covariant return types
    - Object class (root of all classes):
        - toString, equals, hashCode, clone, finalize, getClass
    - final keyword with classes and methods (preventing inheritance/overriding)
    - Sealed classes (Java 17):
        - permits clause
        - Use cases (restricted hierarchies)

Polymorphism:
    - Compile-time polymorphism (method overloading)
    - Runtime polymorphism (method overriding)
    - Upcasting and downcasting
    - instanceof operator
    - Pattern matching for instanceof (Java 16+)

Abstraction:
    - Abstract classes:
        - Abstract methods
        - Concrete methods in abstract classes
        - When to use abstract classes
    - Interfaces:
        - Interface declaration
        - Implementing interfaces
        - Multiple interface implementation
        - Default methods (Java 8+)
        - Static methods in interfaces
        - Private methods in interfaces (Java 9+)
        - Functional interfaces (@FunctionalInterface)
        - Marker interfaces (Serializable, Cloneable)
    - Abstract class vs interface (decision criteria)

Inner Classes:
    - Member inner class
    - Static nested class
    - Local inner class
    - Anonymous inner class
    - When to use each

Enums:
    - Enum declaration
    - Enum constructors, fields, methods
    - Enum with abstract methods
    - values(), valueOf(), ordinal(), name()
    - EnumSet, EnumMap
    - Enums implementing interfaces
```

### 1.06 - [[Java Collections Framework]]

```
This is the equivalent of Python's built-in data structures but far more extensive.

Collection hierarchy:
    Iterable
    └── Collection
        ├── List
        │   ├── ArrayList
        │   ├── LinkedList
        │   └── Vector (legacy, avoid)
        ├── Set
        │   ├── HashSet
        │   ├── LinkedHashSet
        │   └── TreeSet (SortedSet)
        └── Queue
            ├── PriorityQueue
            ├── ArrayDeque
            └── LinkedList (also implements Queue)

    Map (not part of Collection interface)
        ├── HashMap
        ├── LinkedHashMap
        ├── TreeMap (SortedMap)
        ├── Hashtable (legacy, avoid)
        └── ConcurrentHashMap

List:
    - ArrayList: dynamic array, when to use, performance characteristics
    - LinkedList: doubly-linked list, when to use
    - List.of() (immutable list, Java 9+)
    - List.copyOf()
    - Methods: add, get, set, remove, contains, indexOf, size, isEmpty,
      subList, sort, iterator, listIterator
    - Iteration: for-each, iterator, listIterator, forEach

Set:
    - HashSet: hash table backed, O(1) operations
    - LinkedHashSet: insertion-ordered
    - TreeSet: sorted, O(log n) operations, NavigableSet
    - Set.of() (immutable)
    - Methods: add, remove, contains, size, isEmpty
    - Set operations: retainAll (intersection), addAll (union), removeAll (difference)

Map:
    - HashMap: hash table, O(1) average
    - LinkedHashMap: insertion or access ordered
    - TreeMap: sorted by keys, O(log n)
    - Map.of(), Map.ofEntries(), Map.entry() (immutable)
    - Methods: put, get, getOrDefault, containsKey, containsValue, remove,
      putIfAbsent, compute, computeIfAbsent, computeIfPresent, merge,
      keySet, values, entrySet, forEach, replaceAll
    - Map iteration patterns

Queue and Deque:
    - Queue interface: offer, poll, peek
    - Deque interface: offerFirst, offerLast, pollFirst, pollLast
    - ArrayDeque (stack and queue usage)
    - PriorityQueue (min-heap by default)

Collections utility class:
    - sort, reverse, shuffle, min, max, frequency
    - unmodifiableList/Set/Map
    - synchronizedList/Set/Map
    - singletonList, emptyList

Comparable and Comparator:
    - Comparable<T>: compareTo, natural ordering
    - Comparator<T>: compare, custom ordering
    - Comparator.comparing, thenComparing, reversed
    - Sorting with Comparator

Iterator and Iterable:
    - Iterator interface: hasNext, next, remove
    - Implementing Iterable for custom classes
    - ConcurrentModificationException and how to avoid it
    - ListIterator
```

### 1.07 - [[Generics]]

```
- Why generics exist (type safety, code reuse)
- Generic classes
- Generic methods
- Generic interfaces
- Bounded type parameters:
    - Upper bound (<T extends Number>)
    - Multiple bounds (<T extends Comparable<T> & Serializable>)
- Wildcards:
    - Unbounded (?)
    - Upper bounded (? extends T) — producer
    - Lower bounded (? super T) — consumer
    - PECS principle (Producer Extends, Consumer Super)
- Type erasure:
    - What it is
    - Why you cannot do new T() or T.class
    - Bridge methods
    - Implications for runtime
- Generic methods with varargs (@SafeVarargs)
- Recursive type bounds
- Diamond operator (<>)
```

### 1.08 - [[Exception Handling]]

```
- Exception hierarchy:
    Throwable
    ├── Error (OutOfMemoryError, StackOverflowError — do not catch)
    └── Exception
        ├── Checked exceptions (IOException, SQLException)
        └── RuntimeException (unchecked)
            ├── NullPointerException
            ├── IllegalArgumentException
            ├── IllegalStateException
            ├── IndexOutOfBoundsException
            ├── ClassCastException
            ├── ArithmeticException
            ├── UnsupportedOperationException
            └── ConcurrentModificationException

- try / catch / finally
- Multi-catch (catch (IOException | SQLException e))
- try-with-resources (AutoCloseable):
    - What it solves
    - Implementing AutoCloseable
    - Suppressed exceptions
- Checked vs unchecked exceptions:
    - When to use each
    - throws declaration
    - Why checked exceptions are controversial
- Throwing exceptions (throw)
- Custom exceptions:
    - Extending Exception (checked)
    - Extending RuntimeException (unchecked)
    - Adding fields and constructors
    - Exception message best practices
- Exception chaining (cause)
- Best practices:
    - Never catch Exception or Throwable generically
    - Never swallow exceptions silently
    - Fail fast
    - Use specific exception types
    - Document exceptions with @throws in Javadoc
- Optional<T> (Java 8+):
    - What it is (alternative to null)
    - of, ofNullable, empty
    - isPresent, isEmpty, ifPresent, ifPresentOrElse
    - get (avoid), orElse, orElseGet, orElseThrow
    - map, flatMap, filter
    - When to use Optional (return types, not fields or parameters)
```

### 1.09 - [[File IO And Serialization]]

```
- Java I/O evolution:
    - java.io (legacy streams)
    - java.nio (New I/O, buffers, channels)
    - java.nio.file (modern file operations)

- java.nio.file.Path:
    - Creating paths
    - Path operations (resolve, relativize, getParent, getFileName)
    - Paths.get vs Path.of

- java.nio.file.Files:
    - readString, writeString (Java 11+)
    - readAllLines, write
    - exists, isDirectory, isRegularFile
    - createDirectory, createDirectories
    - copy, move, delete
    - walk, list (directory traversal)
    - newBufferedReader, newBufferedWriter

- Reading/writing with streams:
    - InputStream, OutputStream (byte streams)
    - Reader, Writer (character streams)
    - BufferedReader, BufferedWriter
    - InputStreamReader, OutputStreamWriter (bridge)
    - try-with-resources for all I/O

- Serialization:
    - JSON with Jackson:
        - ObjectMapper
        - Serialization (writeValueAsString)
        - Deserialization (readValue)
        - Annotations: @JsonProperty, @JsonIgnore, @JsonInclude,
          @JsonCreator, @JsonFormat, @JsonNaming
        - Custom serializers/deserializers
        - Handling dates
        - Tree model (JsonNode)
    - JSON with Gson (overview, comparison with Jackson)
    - CSV reading/writing (OpenCSV or Apache Commons CSV)
    - YAML with Jackson (jackson-dataformat-yaml)
    - Properties files (java.util.Properties)
    - Java serialization (Serializable) — understand but avoid in production
```

### 1.10 - [[Functional Programming in Java]]

```
- Functional interfaces:
    - @FunctionalInterface annotation
    - Predefined functional interfaces (java.util.function):
        - Function<T, R> — apply
        - Predicate<T> — test
        - Consumer<T> — accept
        - Supplier<T> — get
        - UnaryOperator<T> — apply (Function<T, T>)
        - BinaryOperator<T> — apply (BiFunction<T, T, T>)
        - BiFunction<T, U, R> — apply
        - BiPredicate<T, U> — test
        - BiConsumer<T, U> — accept

- Lambda expressions:
    - Syntax: (parameters) -> expression or (parameters) -> { statements }
    - Single parameter shorthand
    - Effectively final variables (closures)
    - Lambda vs anonymous class

- Method references:
    - Static method reference (Class::method)
    - Instance method reference (object::method)
    - Instance method of arbitrary object (Class::method)
    - Constructor reference (Class::new)

- Stream API:
    - What streams are (not data structures, lazy pipelines)
    - Creating streams:
        - Collection.stream()
        - Stream.of()
        - Arrays.stream()
        - Stream.iterate, Stream.generate
        - IntStream, LongStream, DoubleStream
        - Files.lines()
    - Intermediate operations (lazy):
        - filter, map, flatMap, distinct, sorted
        - peek, limit, skip, takeWhile, dropWhile
        - mapToInt, mapToLong, mapToDouble
    - Terminal operations (eager):
        - forEach, forEachOrdered
        - collect (Collectors)
        - reduce
        - count, min, max, sum, average
        - findFirst, findAny
        - anyMatch, allMatch, noneMatch
        - toArray, toList (Java 16+)
    - Collectors:
        - toList, toSet, toMap, toUnmodifiableList
        - joining
        - groupingBy, partitioningBy
        - counting, summingInt, averagingInt
        - mapping, flatMapping
        - reducing
        - collectingAndThen
    - Parallel streams:
        - parallelStream()
        - When to use (CPU-bound, large datasets)
        - When to avoid (small datasets, I/O-bound, ordering matters)
        - Thread safety concerns
    - Stream pipeline best practices

- Optional (revisited in functional context):
    - Chaining with map, flatMap, filter
    - Using with streams
```

### 1.11 - [[Concurrency and Multithreading]]

```
Java concurrency is a critical differentiator from Python. This is why Java dominates fintech.

Fundamentals:
    - Processes vs threads (revisited)
    - Creating threads:
        - Extending Thread
        - Implementing Runnable
        - Lambda with Runnable
    - Thread lifecycle: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED
    - Thread methods: start, run, sleep, join, interrupt, isInterrupted
    - Daemon threads

Synchronization:
    - Race conditions
    - Critical sections
    - synchronized keyword (method and block)
    - Object locking (intrinsic locks / monitors)
    - Reentrant locking
    - volatile keyword
    - Happens-before relationship
    - Deadlock: what it is, how to prevent
    - Livelock, starvation

java.util.concurrent:
    - Executor framework:
        - Executor, ExecutorService, ScheduledExecutorService
        - Executors factory methods (newFixedThreadPool, newCachedThreadPool,
          newSingleThreadExecutor, newScheduledThreadPool)
        - ThreadPoolExecutor (custom configuration)
        - Submitting tasks (submit, invokeAll, invokeAny)
        - Shutting down executors
    - Future and Callable:
        - Callable<V> vs Runnable
        - Future: get, isDone, cancel
        - Blocking nature of Future.get()
    - CompletableFuture:
        - Creating: supplyAsync, runAsync
        - Chaining: thenApply, thenAccept, thenRun, thenCompose, thenCombine
        - Error handling: exceptionally, handle, whenComplete
        - Combining: allOf, anyOf
        - Async variants: thenApplyAsync
    - Concurrent collections:
        - ConcurrentHashMap
        - CopyOnWriteArrayList
        - BlockingQueue (ArrayBlockingQueue, LinkedBlockingQueue)
        - ConcurrentLinkedQueue
    - Locks (java.util.concurrent.locks):
        - ReentrantLock
        - ReadWriteLock, ReentrantReadWriteLock
        - StampedLock
        - Lock vs synchronized (comparison)
        - Condition objects
    - Atomic classes:
        - AtomicInteger, AtomicLong, AtomicBoolean, AtomicReference
        - Compare-and-swap (CAS)
        - When to use atomics vs locks
    - CountDownLatch, CyclicBarrier, Semaphore, Phaser
    - Fork/Join framework (overview)

Virtual Threads (Java 21):
    - What virtual threads are (Project Loom)
    - Thread.startVirtualThread()
    - Executors.newVirtualThreadPerTaskExecutor()
    - Structured concurrency (preview)
    - Why this changes Java's concurrency story
    - Virtual threads vs platform threads
    - Impact on backend frameworks (Spring Boot integration)

Thread safety best practices:
    - Immutable objects
    - ThreadLocal
    - Avoid shared mutable state
    - Prefer higher-level abstractions (ExecutorService, CompletableFuture)
```

### 1.12 - [[Modules, Packages, and the Build System]] (Conceptual)

```
- Packages:
    - Package declaration
    - Import statements
    - Static imports
    - Package naming conventions (reverse domain)
    - Access control across packages

- Java Module System (JPMS, Java 9+):
    - What modules are
    - module-info.java
    - requires, exports, opens
    - When modules matter (large applications, libraries)
    - When to skip modules (most Spring Boot apps)

- JAR files:
    - What a JAR is
    - Creating JARs (jar command)
    - Executable JARs (MANIFEST.MF, Main-Class)
    - Fat/uber JARs

- Classpath vs module path
```

### 1.13 - [[Annotations]]

```
- What annotations are (metadata)
- Built-in annotations:
    - @Override, @Deprecated, @SuppressWarnings, @FunctionalInterface
    - @SafeVarargs
- Meta-annotations:
    - @Target, @Retention, @Documented, @Inherited, @Repeatable
- Retention policies: SOURCE, CLASS, RUNTIME
- Creating custom annotations
- Processing annotations at runtime (with reflection)
- Annotations in frameworks (Spring, JPA, Jackson — preview)
```

### 1.14 - Reflection (Overview)

```
- What reflection is
- Class object: getClass(), Class.forName()
- Inspecting: fields, methods, constructors, annotations
- Modifying accessibility (setAccessible)
- Why reflection matters (frameworks use it heavily)
- Performance cost of reflection
- When NOT to use reflection in application code
```

### 1.15 - Logging

```
- Why logging matters
- SLF4J (Simple Logging Facade for Java):
    - What a logging facade is
    - LoggerFactory.getLogger()
    - Log levels: TRACE, DEBUG, INFO, WARN, ERROR
    - Parameterized logging (avoid string concatenation)
- Logback:
    - Configuration (logback.xml, logback-spring.xml)
    - Appenders: ConsoleAppender, FileAppender, RollingFileAppender
    - Patterns and layouts
    - MDC (Mapped Diagnostic Context) for request tracing
- Log4j2 (overview, comparison)
- Logging best practices:
    - What to log at each level
    - Never log sensitive data
    - Structured logging (JSON format)
    - Correlation IDs
```

### 1.16 - Testing in Java

```
- JUnit 5 (Jupiter):
    - @Test annotation
    - Assertions: assertEquals, assertTrue, assertFalse, assertNull,
      assertNotNull, assertThrows, assertDoesNotThrow, assertAll,
      assertTimeout, assertIterableEquals
    - Lifecycle: @BeforeEach, @AfterEach, @BeforeAll, @AfterAll
    - @DisplayName
    - @Disabled
    - @Nested (nested test classes)
    - @ParameterizedTest:
        - @ValueSource, @CsvSource, @MethodSource, @EnumSource
    - @RepeatedTest
    - Assumptions: assumeTrue, assumeFalse
    - Test ordering
    - Test instance lifecycle

- Mockito:
    - What mocking is
    - @Mock, @InjectMocks, @Spy
    - when/thenReturn, when/thenThrow
    - doReturn/when (for spies and void methods)
    - verify (times, never, atLeast, atMost)
    - ArgumentCaptor
    - BDD style: given/willReturn, then/should
    - Mocking static methods (mockStatic, Java 11+)
    - @ExtendWith(MockitoExtension.class)

- AssertJ (fluent assertions):
    - assertThat().isEqualTo, isNotNull, contains, hasSize, etc.
    - Why AssertJ over JUnit assertions (readability)

- Test structure:
    - Given-When-Then / Arrange-Act-Assert
    - Test naming conventions
    - Unit test vs integration test separation

- Code coverage:
    - JaCoCo
    - Coverage reports
    - What coverage numbers mean (and do not mean)

- Test-Driven Development (TDD):
    - Red-Green-Refactor cycle
    - When TDD helps, when it hinders
```

### 1.17 - Date and Time API

```
- Why java.util.Date and Calendar are broken (legacy, avoid)
- java.time (Java 8+):
    - LocalDate, LocalTime, LocalDateTime
    - ZonedDateTime, OffsetDateTime
    - Instant (machine time, UTC)
    - Duration (time-based) vs Period (date-based)
    - DateTimeFormatter:
        - Predefined (ISO_LOCAL_DATE, etc.)
        - Custom patterns (ofPattern)
    - Parsing and formatting
    - ZoneId, ZoneOffset
    - TemporalAdjusters
    - ChronoUnit
- Working with timestamps in databases
- Timezone handling best practices (store UTC, display local)
```

### 1.18 - Regular Expressions in Java

```
- java.util.regex: Pattern, Matcher
- Pattern.compile, matcher, matches, find, group
- Pattern syntax (same core regex, Java-specific escaping)
- Named groups
- String methods: matches, replaceAll, replaceFirst, split
- Precompiled patterns for performance
- Common patterns
```

### 1.19 - Phase 01 Projects

```
Project 01A: "Java Fundamentals Drill Suite"
- 60+ exercises covering every syntax and OOP topic
- Organized by sub-topic
- Each exercise has JUnit 5 tests
- Run all tests with a single command (Maven/Gradle)

Project 01B: "Personal Finance Calculator"
- CLI application
- Read transactions from CSV (using Jackson or OpenCSV)
- Categorize expenses
- Calculate: total income, total expenses, savings rate, category breakdown
- Output results to JSON and formatted terminal output
- Use OOP extensively:
    - Transaction, Account, Category, Budget, Report classes
    - Interfaces for calculation strategies
    - Enums for categories and transaction types
    - Records for DTOs
    - Custom exceptions (InsufficientFundsException, InvalidTransactionException)
- Use Streams API for data processing
- Full JUnit 5 + Mockito test suite
- Logging with SLF4J + Logback
- This is the seed for Portfolio Project 01
```

---

## PHASE 02 / DEVELOPER TOOLING
**Follow the structure down.**
*Duration: 2-3 weeks at 6 hours/day*

Java tooling is heavier than Python. Build tools are non-negotiable.

---

### 02.01 — Terminal Mastery (Bash/Zsh)

```
- (Same as Python roadmap — if completed, skip)
- Shell vs terminal vs console
- Bash vs Zsh
- Navigation, file operations, pipes, redirection
- grep, sed, awk basics
- Shell scripting basics
- curl and wget
- tmux
- Dotfiles management
```

### 02.02 — Git and GitHub

```
- (Same as Python roadmap — if completed, skip)
- Git architecture and all commands
- GitHub workflows
- Branching strategies
- Conventional commits
- PR workflows
- SSH setup
```

### 02.03 — Build Tools

```
Maven:
    - What Maven is (build tool + dependency manager + project structure)
    - POM (pom.xml):
        - groupId, artifactId, version
        - Properties
        - Dependencies
        - Dependency scope (compile, test, provided, runtime)
        - Plugins
        - Profiles
    - Maven lifecycle: validate, compile, test, package, verify, install, deploy
    - Common commands: mvn clean, mvn compile, mvn test, mvn package, mvn install
    - Maven Central Repository
    - Multi-module projects
    - Maven wrapper (mvnw)
    - Dependency management (BOM — Bill of Materials)
    - Dependency conflicts and resolution
    - Effective POM (mvn help:effective-pom)

Gradle:
    - What Gradle is (Groovy/Kotlin DSL, more flexible than Maven)
    - build.gradle (Groovy DSL) and build.gradle.kts (Kotlin DSL)
    - Settings file
    - Dependencies: implementation, testImplementation, compileOnly, runtimeOnly
    - Tasks and task graph
    - Common commands: gradle build, gradle test, gradle clean, gradle bootRun
    - Gradle wrapper (gradlew)
    - Plugins
    - Multi-project builds
    - Build cache
    - Maven vs Gradle (comparison, when to use each)

- Choosing: Spring Boot defaults to Gradle (Kotlin DSL) in Spring Initializr.
  Both are industry standard. Learn Maven first (more common in enterprise/fintech),
  then Gradle.
```

### 02.04 — Code Quality Tools

```
- Code style:
    - Google Java Style Guide
    - Checkstyle (enforcing style)
    - .editorconfig
- Static analysis:
    - SpotBugs (successor to FindBugs)
    - PMD
    - SonarLint / SonarQube (overview)
    - Error Prone (Google's compile-time checks)
- Formatting:
    - google-java-format
    - Spotless plugin (Maven/Gradle)
- Pre-commit hooks:
    - pre-commit framework (same as Python roadmap)
    - Hooks for Java: checkstyle, spotless
- Dependency vulnerability scanning:
    - OWASP Dependency-Check plugin
    - Snyk (overview)
```

### 02.05 — IDE

```
- IntelliJ IDEA:
    - Community vs Ultimate (Ultimate for Spring, database tools)
    - Essential shortcuts:
        - Navigate: Cmd+O (class), Cmd+Shift+O (file), Cmd+E (recent)
        - Refactor: Shift+F6 (rename), Cmd+Alt+M (extract method),
          Cmd+Alt+V (extract variable), Cmd+Alt+F (extract field)
        - Code: Alt+Enter (quick fix), Cmd+N (generate), Ctrl+Space (complete)
        - Run: Ctrl+Shift+R (run), Ctrl+Shift+D (debug)
        - Search: Cmd+Shift+F (find in project), Double Shift (search everywhere)
    - Debugging:
        - Breakpoints (line, conditional, exception)
        - Step over, step into, step out
        - Evaluate expression
        - Watch variables
    - Database tools (Ultimate)
    - Git integration
    - Running tests
    - Live templates
    - Plugins:
        - Lombok
        - SonarLint
        - .env files
        - Docker
        - Kubernetes
    - Project structure configuration
    - Memory settings for large projects

- VS Code with Java:
    - Extension Pack for Java
    - Spring Boot Extension Pack
    - When VS Code is sufficient vs when IntelliJ is necessary
```

### PHASE 02 CHAPTER PROJECT

```
Project 02: "Development Environment and Workflow"
- Create a dotfiles repository on GitHub
- Shell configuration files (.zshrc)
- Git configuration (.gitconfig)
- IntelliJ settings export
- A setup.sh script that installs:
    - SDKMAN, JDK 21
    - Maven, Gradle
    - Docker
    - IntelliJ (via brew cask)
    - All CLI tools
- Create a Maven project template with:
    - Checkstyle configured
    - Spotless configured
    - JaCoCo configured
    - JUnit 5 dependency
    - SLF4J + Logback
    - .gitignore for Java
    - README template
- Conventional commits throughout
```

---

## PHASE 03 / DATA STRUCTURES AND ALGORITHMS
**Follow the structure down.**
*Duration: 4-5 weeks at 6 hours/day*

Implement everything in Java. Java interviews are more algorithm-heavy than Python interviews for backend roles, especially in fintech.

---

### 03.01 — Complexity Analysis

```
- Big-O notation (same theory as Python roadmap)
- Time and space complexity
- Best, worst, average case
- Amortized analysis
- Java Collections complexity:
    - ArrayList: O(1) access, O(n) insert/delete middle, O(1) amortized add
    - LinkedList: O(n) access, O(1) insert/delete at known position
    - HashMap: O(1) average get/put, O(n) worst case
    - TreeMap: O(log n) all operations
    - HashSet: O(1) average
    - TreeSet: O(log n)
    - PriorityQueue: O(log n) offer/poll, O(1) peek
```

### 03.02 — Arrays and Strings

```
- Array manipulation in Java
- Two-pointer technique
- Sliding window technique
- Prefix sums
- StringBuilder for string manipulation
- String pool implications
- Practice problems (10 minimum)
```

### 03.03 — Linked Lists

```
- Singly linked list (implementation from scratch with generics)
- Doubly linked list (implementation from scratch with generics)
- Operations: insert, delete, search, reverse
- Cycle detection (Floyd's algorithm)
- Merge two sorted lists
- LRU Cache (LinkedHashMap or manual implementation)
- Practice problems (8 minimum)
```

### 03.04 — Stacks and Queues

```
- Stack (implementation with ArrayList and LinkedList)
- Queue (implementation with LinkedList and ArrayDeque)
- Applications: balanced parentheses, expression evaluation
- Monotonic stack
- Priority queue (PriorityQueue, custom Comparator)
- Min-heap, max-heap
- Implement a min-heap from scratch
- Practice problems (8 minimum)
```

### 03.05 — Hash Tables

```
- How hashing works in Java (hashCode, equals contract)
- HashMap internals:
    - Buckets, load factor, rehashing
    - Java 8+ treeification (when chains become trees)
- Implementing a hash table from scratch
- Hash collisions: chaining, open addressing
- Frequency counting patterns
- Two-sum and variants
- Practice problems (8 minimum)
```

### 03.06 — Trees

```
- Binary trees (implementation with generics)
- Binary search trees:
    - Insert, search, delete
    - In-order, pre-order, post-order traversal (recursive and iterative)
    - Level-order traversal (BFS with Queue)
- Balanced BSTs:
    - AVL tree (conceptual, optional implementation)
    - Red-Black tree (conceptual — this is what TreeMap uses)
- Tries (prefix trees):
    - Implementation
    - Autocomplete use case
- Segment trees (overview for range queries)
- Practice problems (10 minimum)
```

### 03.07 — Graphs

```
- Graph representations: adjacency matrix, adjacency list
- BFS (using Queue)
- DFS (recursive and iterative using Stack)
- Topological sort (Kahn's algorithm, DFS-based)
- Dijkstra's shortest path (PriorityQueue)
- Bellman-Ford (overview)
- Cycle detection (directed and undirected)
- Connected components
- Union-Find (disjoint set with path compression and union by rank)
- Minimum spanning tree (Kruskal's, Prim's — overview)
- Practice problems (10 minimum)
```

### 03.08 — Sorting and Searching

```
- Merge sort (implementation)
- Quick sort (implementation, pivot strategies)
- Counting sort, radix sort
- Java's built-in sorting (Arrays.sort — dual-pivot quicksort for primitives,
  TimSort for objects)
- Binary search (iterative and recursive)
- Arrays.binarySearch, Collections.binarySearch
- Binary search variations
- Practice problems (8 minimum)
```

### 03.09 — Dynamic Programming

```
- Memoization (top-down, HashMap or array)
- Tabulation (bottom-up)
- Classic problems:
    - Fibonacci, climbing stairs, coin change
    - Longest common subsequence
    - 0/1 Knapsack
    - Edit distance
    - Longest increasing subsequence
    - Maximum subarray (Kadane's)
    - Matrix chain multiplication
    - Unique paths
- Space optimization techniques
- Practice problems (10 minimum)
```

### 03.10 — Greedy Algorithms and Backtracking

```
- Greedy strategy
- Activity selection, interval scheduling
- Huffman coding (conceptual)
- Backtracking:
    - N-Queens
    - Sudoku solver
    - Permutations and combinations (with streams and without)
- Practice problems (6 minimum)
```

### PHASE 03 CHAPTER PROJECT

```
Project 03: "Data Structures Library"
- Implement from scratch (with generics):
    - Dynamic array (ArrayList equivalent)
    - Singly and doubly linked list
    - Stack, Queue
    - Hash map
    - Binary search tree
    - Min-heap / Priority queue
    - Trie
    - Graph (adjacency list)
- Each implementation:
    - Implements appropriate Java interfaces (Iterable, Comparable)
    - Full JUnit 5 test suite
    - Javadoc documentation
    - Complexity annotations on each method
- CLI benchmarking tool:
    - Compares custom implementations vs Java Collections
    - Measures time for various operations
- Published to GitHub with comprehensive README
```

---

## PHASE 04 / DATABASES
**Follow the structure down.**
*Duration: 4-5 weeks at 6 hours/day*

Same database theory as the Python roadmap but with Java-specific connectivity (JDBC, JPA/Hibernate).

---

### 04.01 — Database Fundamentals

```
- (Same theory as Python roadmap — if completed, review briefly)
- Relational vs non-relational
- ACID properties
- CAP theorem
- OLTP vs OLAP
- Data modeling, ER diagrams
- Normalization (1NF through BCNF)
- Denormalization
```

### 04.02 — SQL Fundamentals

```
- (Same as Python roadmap — if completed, review briefly)
- DDL, DML, DQL
- Data types (PostgreSQL)
- Constraints
- JOINs (all types)
- Subqueries and CTEs
- Window functions
- Transactions and isolation levels
- Views and materialized views
```

### 04.03 — PostgreSQL Deep Dive

```
- (Same as Python roadmap — if completed, review briefly)
- PostgreSQL-specific features: JSONB, arrays, UUID, full-text search
- Indexing: B-tree, Hash, GIN, GiST, BRIN
- EXPLAIN ANALYZE
- VACUUM, ANALYZE
- Connection pooling (HikariCP is the Java standard)
- Partitioning
- Roles and permissions
```

### 04.04 — JDBC (Java Database Connectivity)

```
- What JDBC is
- JDBC architecture: Driver, Connection, Statement, ResultSet
- Loading JDBC drivers
- Connection management:
    - DriverManager.getConnection
    - Connection URL format
    - Connection properties
- Statement types:
    - Statement (avoid — SQL injection risk)
    - PreparedStatement (always use):
        - Parameterized queries
        - Setting parameters by type
        - Batch operations
    - CallableStatement (stored procedures)
- ResultSet:
    - Navigating results
    - Getting values by column name or index
    - ResultSet types and concurrency
- Transaction management:
    - setAutoCommit(false)
    - commit()
    - rollback()
    - Savepoints
- Connection pooling:
    - Why connection pooling matters
    - HikariCP:
        - Configuration
        - Pool sizing
        - Connection timeout, idle timeout, max lifetime
- Resource management (try-with-resources for all JDBC objects)
- DAO pattern (Data Access Object):
    - Interface design
    - Implementation
    - Separation from business logic
```

### 04.05 — JPA and Hibernate

```
- What JPA is (specification, not implementation)
- What Hibernate is (JPA implementation)
- Why ORM exists (impedance mismatch)

Entity mapping:
    - @Entity, @Table
    - @Id, @GeneratedValue (strategies: IDENTITY, SEQUENCE, UUID)
    - @Column (name, nullable, unique, length, precision, scale)
    - Field access vs property access
    - @Transient

Relationships:
    - @OneToOne
    - @OneToMany / @ManyToOne
    - @ManyToMany
    - @JoinColumn, @JoinTable
    - Bidirectional vs unidirectional
    - mappedBy
    - Cascade types (PERSIST, MERGE, REMOVE, REFRESH, DETACH, ALL)
    - orphanRemoval
    - Fetch types (EAGER vs LAZY):
        - Default fetch types per relationship
        - N+1 query problem
        - Solutions: JOIN FETCH, @EntityGraph, @BatchSize

Inheritance mapping:
    - Single table (@Inheritance(strategy = SINGLE_TABLE), @DiscriminatorColumn)
    - Joined table (@Inheritance(strategy = JOINED))
    - Table per class (@Inheritance(strategy = TABLE_PER_CLASS))
    - When to use each

EntityManager:
    - persist, merge, remove, find, getReference
    - flush, clear, detach
    - Entity lifecycle: transient, managed, detached, removed

JPQL (Java Persistence Query Language):
    - SELECT, WHERE, JOIN, ORDER BY, GROUP BY
    - Named queries (@NamedQuery)
    - Parameter binding (positional and named)
    - Aggregate functions
    - Pagination (setFirstResult, setMaxResults)

Criteria API:
    - CriteriaBuilder, CriteriaQuery, Root
    - Type-safe queries
    - Dynamic query construction
    - Metamodel (static metamodel generation)

Native queries:
    - When to drop to SQL
    - @SqlResultSetMapping

Second-level cache:
    - What it is
    - Ehcache or Caffeine integration
    - @Cacheable, cache regions
    - When to cache, when not to

Hibernate-specific features:
    - @NaturalId
    - @Formula
    - @Where
    - @Filter
    - Envers (auditing — overview)
    - Hibernate Validator (Bean Validation):
        - @NotNull, @NotBlank, @NotEmpty
        - @Size, @Min, @Max
        - @Email, @Pattern
        - @Past, @Future
        - Custom validators
        - Validation groups
```

### 04.06 — Database Migrations

```
- Why migrations matter
- Flyway:
    - Installation and configuration
    - SQL-based migrations (V1__description.sql naming)
    - Java-based migrations
    - Baseline
    - Repair
    - Migration ordering
    - Repeatable migrations
    - Environment-specific migrations
- Liquibase (overview, comparison with Flyway):
    - Changelog formats (XML, YAML, SQL)
    - Changesets
    - Rollback support
- Migration strategies in team environments
- Zero-downtime migrations:
    - Additive changes only
    - Multi-phase migrations
    - Expand and contract pattern
```

### 04.07 — Redis

```
- (Same theory as Python roadmap)
- Redis data types and operations
- Use cases: caching, sessions, rate limiting, leaderboards

- Java Redis clients:
    - Jedis:
        - Connection
        - Basic operations
        - Pipelining
        - Transactions
        - JedisPool
    - Lettuce:
        - Reactive and async support
        - Connection
        - Cluster support
    - Spring Data Redis:
        - RedisTemplate
        - StringRedisTemplate
        - @Cacheable, @CacheEvict, @CachePut (Spring Cache abstraction)
        - Serialization (JSON vs JDK)
    - Redisson (overview — distributed data structures)
```

### 04.08 — MongoDB (Overview)

```
- (Same concepts as Python roadmap)
- Document databases, when to use
- MongoDB Java Driver:
    - MongoClient
    - CRUD operations
    - Queries and filters
    - Aggregation pipeline
- Spring Data MongoDB (overview):
    - @Document, @Id
    - MongoRepository
    - Query methods
- When MongoDB is appropriate and when it is not
```

### PHASE 04 CHAPTER PROJECT

```
Project 04: "Financial Data Store"
- Design normalized schema for:
    - Users, accounts, transactions, categories, budgets
- Implement in PostgreSQL
- JDBC layer: DAO pattern with raw SQL for complex queries
- JPA/Hibernate layer: Entity classes for all tables
- Flyway migrations for schema management
- Implement complex queries:
    - Monthly spending by category (JPQL and native SQL)
    - Running account balance over time (window functions)
    - Top merchants by transaction volume
    - Year-over-year comparison
- Redis caching for:
    - Account balance lookups
    - Recent transactions
- Benchmark: JDBC vs JPA query performance
- Full JUnit 5 test suite (H2 in-memory for unit tests, Testcontainers for integration)
- Testcontainers:
    - What it is (Docker-based test dependencies)
    - PostgreSQL Testcontainer
    - Redis Testcontainer
```

---

## PHASE 05 / BACKEND FRAMEWORKS
**Follow the structure down.**
*Duration: 6-8 weeks at 6 hours/day*

Spring Boot is the undisputed king of Java backend. This phase is long because Spring is an ecosystem, not just a framework.

---

### 05.01 — HTTP Deep Dive

```
- (Same as Python roadmap — if completed, review briefly)
- HTTP methods, status codes, headers
- Content types
- Cookies and sessions
- HTTPS and TLS
- REST constraints and API design
- Request/response lifecycle in a servlet container
```

### 05.02 — Servlets and Jakarta EE (Foundation)

```
- What a servlet is
- Servlet lifecycle: init, service, destroy
- HttpServletRequest, HttpServletResponse
- Servlet container (Tomcat, Jetty, Undertow)
- What Jakarta EE is (formerly Java EE)
- Why you need to understand this (Spring abstracts it)
- Filters and listeners
- WAR files vs embedded servers
- You will NOT build applications with raw servlets — this is for understanding what Spring does underneath
```

### 05.03 — Spring Core

```
- What Spring Framework is
- Inversion of Control (IoC) and Dependency Injection (DI):
    - What DI is and why it matters
    - Spring IoC container (ApplicationContext)
    - Bean definition
    - Bean scopes: singleton, prototype, request, session
    - Bean lifecycle: instantiation → population → initialization → destruction
    - @PostConstruct, @PreDestroy

- Configuration:
    - Annotation-based configuration:
        - @Component, @Service, @Repository, @Controller, @RestController
        - @Autowired (constructor injection — preferred)
        - @Qualifier
        - @Primary
        - @Value
    - Java-based configuration:
        - @Configuration
        - @Bean
        - @Import
        - @Profile
        - @Conditional
    - Component scanning (@ComponentScan)

- Constructor injection vs field injection vs setter injection:
    - Why constructor injection is preferred
    - How Spring resolves dependencies
    - Circular dependencies

- Spring AOP (Aspect-Oriented Programming):
    - What AOP is
    - Cross-cutting concerns
    - Terminology: aspect, advice, pointcut, join point
    - Advice types: @Before, @After, @AfterReturning, @AfterThrowing, @Around
    - Creating custom aspects
    - Use cases: logging, security, transactions, metrics
    - How Spring AOP works (proxies)
    - Limitations of proxy-based AOP

- Spring Expression Language (SpEL) — basics

- Events:
    - ApplicationEvent
    - @EventListener
    - @Async event handling
    - Custom events
```

### 05.04 — Spring Boot

```
- What Spring Boot is (opinionated Spring configuration)
- Spring Initializr (start.spring.io)
- Project structure:
    /src/main/java/com/example/project
        /config
        /controller
        /service
        /repository
        /model (entity)
        /dto
        /exception
        /mapper
        /util
        Application.java
    /src/main/resources
        application.yml (or .properties)
        /db/migration (Flyway)
    /src/test/java
    pom.xml or build.gradle.kts

- Auto-configuration:
    - What it is
    - How it works (@EnableAutoConfiguration, spring.factories)
    - Excluding auto-configurations

- application.yml / application.properties:
    - Property binding
    - @ConfigurationProperties (type-safe configuration)
    - Profiles (application-dev.yml, application-prod.yml)
    - Profile activation
    - Environment-specific configuration
    - Externalized configuration hierarchy

- Starters:
    - spring-boot-starter-web
    - spring-boot-starter-data-jpa
    - spring-boot-starter-data-redis
    - spring-boot-starter-security
    - spring-boot-starter-validation
    - spring-boot-starter-test
    - spring-boot-starter-actuator
    - spring-boot-starter-cache

- Embedded server (Tomcat, Jetty, Undertow):
    - Configuration
    - Port, context path
    - Server tuning

- Spring Boot DevTools:
    - Automatic restart
    - LiveReload
    - Property defaults

- Actuator:
    - Health endpoint (/actuator/health)
    - Info endpoint
    - Metrics endpoint (/actuator/metrics)
    - Custom health indicators
    - Securing actuator endpoints
    - Prometheus endpoint (/actuator/prometheus)
```

### 05.05 — Spring Web (REST API Development)

```
- @RestController:
    - @GetMapping, @PostMapping, @PutMapping, @PatchMapping, @DeleteMapping
    - @RequestMapping

- Request handling:
    - @PathVariable
    - @RequestParam (required, defaultValue)
    - @RequestBody
    - @RequestHeader
    - @CookieValue
    - @ModelAttribute

- Response handling:
    - ResponseEntity<T>
    - Status codes
    - Response headers
    - @ResponseStatus

- Validation:
    - @Valid and @Validated
    - Bean Validation annotations on DTOs
    - Validation groups
    - Custom validators
    - MethodArgumentNotValidException handling

- Exception handling:
    - @ExceptionHandler
    - @ControllerAdvice / @RestControllerAdvice
    - Global exception handling
    - ProblemDetail (RFC 7807, Spring 6+)
    - Custom error response structure
    - Mapping exceptions to HTTP status codes

- DTO pattern:
    - Why DTOs exist (never expose entities directly)
    - Request DTOs, response DTOs
    - Mapping: MapStruct (recommended):
        - @Mapper, @Mapping
        - Component model (Spring)
        - Nested mapping
        - Custom mapping methods
    - Mapping: ModelMapper (overview)
    - Manual mapping (when to use)

- Pagination and sorting:
    - Pageable, Page<T>, Slice<T>
    - PageRequest
    - Sort
    - Custom pagination response

- Content negotiation
- File upload and download
- Async controllers (@Async, DeferredResult, CompletableFuture)
- API versioning strategies (URL, header)
- HATEOAS (overview)
- OpenAPI / Swagger:
    - springdoc-openapi
    - @Operation, @ApiResponse, @Schema, @Tag
    - Swagger UI configuration
    - Generating API documentation
```

### 05.06 — Spring Data

```
- What Spring Data is (unified data access abstraction)
- Spring Data JPA:
    - JpaRepository<T, ID>
    - CrudRepository, PagingAndSortingRepository
    - Query derivation (method name queries):
        - findByName, findByEmailAndStatus
        - findByCreatedAtBetween, findByNameContaining
        - countBy, existsBy, deleteBy
    - @Query (JPQL and native)
    - @Modifying (for UPDATE/DELETE queries)
    - Specifications (dynamic queries):
        - Specification<T>
        - JpaSpecificationExecutor
        - Combining specifications (and, or)
    - Projections:
        - Interface-based projections
        - Class-based projections (DTOs)
    - Auditing:
        - @CreatedDate, @LastModifiedDate
        - @CreatedBy, @LastModifiedBy
        - @EnableJpaAuditing
        - AuditorAware
    - Custom repository implementations
    - @EntityGraph (solving N+1)
    - Pagination (revisited with repository methods)
    - Transactions (@Transactional):
        - Propagation types (REQUIRED, REQUIRES_NEW, NESTED, etc.)
        - Isolation levels
        - readOnly
        - Rollback rules
        - Transaction boundaries (service layer)
        - Common pitfalls (self-invocation, checked exceptions)

- Spring Data Redis:
    - RedisTemplate
    - StringRedisTemplate
    - Operations: opsForValue, opsForHash, opsForList, opsForSet, opsForZSet
    - Repository support (@RedisHash)
    - Spring Cache abstraction (@Cacheable, @CacheEvict, @CachePut, @Caching)
    - Cache managers (RedisCacheManager)
    - TTL configuration

- Spring Data MongoDB (overview):
    - MongoTemplate
    - MongoRepository
    - Query methods and custom queries
```

### 05.07 — Spring Security

```
- What Spring Security does
- SecurityFilterChain (modern configuration, Spring Security 6+)
- Authentication:
    - Authentication object
    - UserDetailsService, UserDetails
    - PasswordEncoder (BCryptPasswordEncoder)
    - DaoAuthenticationProvider
    - In-memory authentication (testing only)
    - Database-backed authentication
- Authorization:
    - Role-based: hasRole, hasAnyRole
    - Authority-based: hasAuthority, hasAnyAuthority
    - @PreAuthorize, @PostAuthorize (method-level security)
    - @Secured
    - SecurityContext and SecurityContextHolder
    - Custom authorization logic

- JWT authentication:
    - JWT structure (header, payload, signature)
    - Access tokens and refresh tokens
    - JWT filter implementation:
        - Extracting token from Authorization header
        - Validating token
        - Setting authentication in SecurityContext
    - Login endpoint (authentication, token generation)
    - Token refresh endpoint
    - Token expiration handling
    - Libraries: jjwt (io.jsonwebtoken) or Nimbus JOSE+JWT
    - Stateless session configuration

- OAuth2:
    - OAuth2 concepts (authorization code, client credentials, resource owner)
    - Spring Security OAuth2 Resource Server
    - JWT decoder configuration
    - OAuth2 with external providers (conceptual)

- CORS configuration (CorsConfigurationSource)
- CSRF (when to enable, when to disable for APIs)
- Security headers
- Rate limiting (Bucket4j or custom filter)
- Method-level security
- Custom filters in the security filter chain
- Testing with security (@WithMockUser, @WithUserDetails)
```

### 05.08 — Spring Boot Advanced

```
- Async processing:
    - @Async annotation
    - @EnableAsync
    - Custom TaskExecutor
    - CompletableFuture with @Async
    - Exception handling in async methods

- Scheduling:
    - @Scheduled (fixedRate, fixedDelay, cron)
    - @EnableScheduling
    - Dynamic scheduling

- WebSocket support:
    - STOMP over WebSocket
    - @MessageMapping, @SendTo, @SendToUser
    - SimpMessagingTemplate
    - WebSocket security

- Server-Sent Events (SSE):
    - SseEmitter
    - Streaming endpoints

- Caching (deep dive):
    - @Cacheable strategies
    - Cache key generation
    - Conditional caching
    - Cache sync
    - Multiple cache managers

- Interceptors:
    - HandlerInterceptor
    - preHandle, postHandle, afterCompletion
    - Use cases: logging, auth, metrics

- Filters vs interceptors vs AOP (when to use each)

- Configuration:
    - @ConfigurationProperties in depth
    - Validation on config properties (@Validated)
    - Custom property sources
    - Profiles for different environments

- Spring Boot testing:
    - @SpringBootTest (full application context)
    - @WebMvcTest (controller layer only)
    - @DataJpaTest (repository layer only)
    - @MockBean, @SpyBean
    - TestRestTemplate, WebTestClient
    - MockMvc:
        - perform (get, post, put, delete)
        - andExpect (status, jsonPath, content)
        - contentType, content
    - Testcontainers integration:
        - @Testcontainers, @Container
        - @DynamicPropertySource
        - PostgreSQL, Redis, Kafka containers
    - Test slicing strategy:
        - Unit tests (Mockito, no Spring context)
        - Integration tests (@SpringBootTest with Testcontainers)
        - Controller tests (@WebMvcTest)
        - Repository tests (@DataJpaTest)
    - Test configuration:
        - application-test.yml
        - @TestConfiguration
        - @ActiveProfiles("test")

- Spring Boot 3 features:
    - Jakarta EE namespace (javax → jakarta)
    - Native compilation (GraalVM — overview)
    - Observability (Micrometer, OpenTelemetry)
    - ProblemDetail for error responses
    - HTTP interfaces (@HttpExchange)
```

### 05.09 — Quarkus (Overview)

```
- What Quarkus is (supersonic, subatomic Java)
- When to choose Quarkus over Spring Boot:
    - Container-first design
    - GraalVM native compilation (fast startup, low memory)
    - Serverless and Kubernetes-native
    - Cloud functions
- Key differences from Spring Boot:
    - CDI vs Spring DI
    - Build-time processing vs runtime reflection
    - Dev mode
- RESTEasy (JAX-RS)
- Panache (simplified Hibernate)
- Extension model
- When Quarkus wins: serverless, startup-sensitive, resource-constrained
```

### 05.10 — Micronaut (Overview)

```
- What Micronaut is
- Compile-time DI (vs Spring runtime DI)
- Low memory footprint
- Comparison with Spring Boot and Quarkus
- When to consider Micronaut
- Awareness level only (Spring Boot is the primary framework for this roadmap)
```

### PHASE 05 CHAPTER PROJECT

```
Project 05: "Banking REST API"
- Full REST API using Spring Boot 3
- Features:
    - User registration and login (JWT with Spring Security)
    - Account management (create, view, close)
    - Transaction processing (deposit, withdraw, transfer)
    - Transaction history with filtering, sorting, pagination (Specifications)
    - Budget creation and tracking
    - Spending analytics endpoints (window functions via native queries)
    - Real-time balance updates via WebSocket (STOMP)
- PostgreSQL with Spring Data JPA
- Flyway migrations
- Redis caching for balance lookups and recent transactions
- Rate limiting on sensitive endpoints
- RBAC (ADMIN, USER roles)
- MapStruct for DTO mapping
- Bean Validation on all inputs
- Global exception handling with ProblemDetail
- Structured logging with MDC
- Actuator health and metrics endpoints
- springdoc-openapi for API documentation
- Full test suite:
    - Unit tests (Mockito, 80%+ coverage)
    - Controller tests (@WebMvcTest)
    - Repository tests (@DataJpaTest)
    - Integration tests (Testcontainers)
- This is Portfolio Project 02
```

---

## PHASE 06 / SYSTEM DESIGN
**Follow the structure down.**
*Duration: 4-5 weeks at 6 hours/day*

Theory is identical to the Python roadmap. Implementation details change for Java.

---

### 06.01 — System Design Fundamentals

```
- (Same theory as Python roadmap)
- Functional vs non-functional requirements
- Back-of-the-envelope estimation
- Latency numbers
- Horizontal vs vertical scaling
- Stateless services
- Redundancy and replication
- CAP revisited
- SLAs, SLOs, SLIs
```

### 06.02 — Caching Strategies

```
- (Same theory as Python roadmap)
- Cache-aside, write-through, write-behind, read-through
- Cache invalidation
- Cache stampede
- Caching layers
- Java-specific:
    - Caffeine (local cache, successor to Guava cache)
    - Spring Cache abstraction (revisited)
    - Redis (revisited for distributed caching)
    - Hazelcast (distributed cache and data grid — overview)
```

### 06.03 — Message Queues and Async Processing

```
- (Same theory as Python roadmap)
- Producer-consumer, pub/sub
- Dead letter queues
- Idempotency

RabbitMQ:
    - Exchanges, queues, bindings
    - Spring AMQP (spring-boot-starter-amqp):
        - RabbitTemplate
        - @RabbitListener
        - Message conversion (Jackson)
        - Exchange types (direct, topic, fanout, headers)
        - Acknowledgments and error handling
        - Retry with DLQ
        - Manual acknowledgment

Apache Kafka:
    - Topics, partitions, consumer groups, offsets
    - Spring Kafka (spring-boot-starter-kafka or spring-kafka):
        - KafkaTemplate
        - @KafkaListener
        - Consumer group configuration
        - Serialization/deserialization (JSON, Avro)
        - Error handling (SeekToCurrentErrorHandler, DefaultErrorHandler)
        - Exactly-once semantics
        - Transactional Kafka
    - Confluent Schema Registry (Avro schemas)
    - Kafka Streams:
        - KStream, KTable, GlobalKTable
        - Windowed aggregations
        - Joins
        - State stores
        - Spring Cloud Stream with Kafka Streams
    - When Kafka vs RabbitMQ

Celery equivalent in Java:
    - There is no direct equivalent
    - Spring @Async + CompletableFuture
    - Spring Integration (overview)
    - Quartz Scheduler (for scheduled jobs)
```

### 06.04 — Load Balancing and Reverse Proxies

```
- (Same as Python roadmap)
- NGINX configuration
- Load balancing algorithms
- Health checks
- SSL termination
- Java-specific: embedded Tomcat configuration, thread pool tuning
```

### 06.05 — API Gateway Pattern

```
- (Same as Python roadmap)
- Spring Cloud Gateway:
    - Route definitions
    - Predicates
    - Filters (pre and post)
    - Rate limiting
    - Circuit breaker integration
    - Load balancing
- Kong, AWS API Gateway (conceptual)
```

### 06.06 — Microservices Architecture

```
- (Same theory as Python roadmap)
- Monolith vs microservices
- Service boundaries, bounded contexts
- Inter-service communication

Java microservices ecosystem:
    - Spring Cloud:
        - Spring Cloud Config (centralized configuration)
        - Spring Cloud Netflix (Eureka — service discovery)
        - Spring Cloud Gateway (API gateway)
        - Spring Cloud CircuitBreaker (Resilience4j):
            - Circuit breaker
            - Retry
            - Rate limiter
            - Bulkhead
            - TimeLimiter
        - Spring Cloud OpenFeign (declarative REST client):
            - @FeignClient
            - Request/response interceptors
            - Error handling
            - Circuit breaker integration
        - Spring Cloud Sleuth / Micrometer Tracing (distributed tracing)
        - Spring Cloud Stream (messaging abstraction)

    - gRPC in Java:
        - Protocol Buffers (.proto files)
        - grpc-java
        - grpc-spring-boot-starter
        - Defining services
        - Server and client implementation
        - Interceptors
        - Deadline/timeout
        - Streaming (server, client, bidirectional)
        - When gRPC vs REST

    - Saga pattern implementation:
        - Orchestration (central coordinator)
        - Choreography (event-based)
        - Compensating transactions
        - State machines for saga orchestration

    - API composition pattern
    - Strangler fig pattern
    - Database per service
    - Event-driven communication between services
```

### 06.07 — Event-Driven Architecture

```
- (Same theory as Python roadmap)
- Event sourcing
- CQRS
- Domain events
- Integration events
- Eventual consistency

Java-specific:
    - Axon Framework (overview — event sourcing + CQRS framework for Java)
    - Spring Application Events for intra-service events
    - Kafka for inter-service events
    - Event schema evolution and versioning
    - Outbox pattern (transactional outbox):
        - Write event to outbox table in same transaction as state change
        - Separate process publishes outbox events to Kafka
        - Debezium CDC (Change Data Capture — overview)
```

### 06.08 — System Design Case Studies

```
- (Same as Python roadmap)
- Design a URL shortener
- Design a rate limiter
- Design a notification system
- Design a payment processing system
- Design a real-time stock price feed
- Design a transaction ledger system
- Each with Java stack recommendations
```

### PHASE 06 CHAPTER PROJECT

```
Project 06: "Real-Time Data Pipeline"
- Ingest financial market data (mock or free API)
- Kafka producer (Spring Kafka): publishes price updates
- Kafka consumer: processes and stores in PostgreSQL
- Kafka Streams: computes real-time aggregations (OHLCV candles)
- Redis: caches latest prices
- Spring Boot REST API: serves historical data
- WebSocket (STOMP): streams real-time prices to clients
- Quartz Scheduler: periodic aggregation and cleanup jobs
- Resilience4j: circuit breakers for external API calls
- Docker Compose for all services
- Full documentation
- This is Portfolio Project 03
```

---

## PHASE 07 / DEVOPS AND CLOUD
**Follow the structure down.**
*Duration: 5-6 weeks at 6 hours/day*

Mostly identical to the Python roadmap with Java-specific build and deployment considerations.

---

### 07.01 — Linux Administration

```
- (Same as Python roadmap)
- systemd, user management, file permissions
- Networking tools
- Firewall basics
- Log management
- Shell scripting for automation
```

### 07.02 — Docker

```
- (Same as Python roadmap — theory)
- Java-specific Dockerfile:
    - Multi-stage build:
        Stage 1: Maven/Gradle build (builder)
        Stage 2: JRE-only runtime (eclipse-temurin:21-jre-alpine)
    - Layer optimization for Java:
        - Spring Boot layered JARs
        - exploded JAR layout
        - Paketo buildpacks (alternative to Dockerfile)
    - JVM memory settings in containers:
        - -XX:MaxRAMPercentage
        - Container-aware JVM (Java 10+)
        - Heap vs non-heap memory
    - .dockerignore for Java projects
    - Security: non-root user, minimal base images
- Docker Compose for Java microservices
- Jib (Google's container image builder for Java — no Dockerfile needed)
```

### 07.03 — CI/CD

```
- (Same as Python roadmap — theory)
- GitHub Actions for Java:
    - Setup JDK action
    - Maven/Gradle caching
    - Pipeline stages: checkstyle → test → build → docker → deploy
    - Matrix builds (multiple Java versions)
    - Testcontainers in CI
    - Publishing Docker images
    - Deploy to AWS ECS/EKS
- Jenkins (overview — still common in Java enterprise):
    - Jenkinsfile (declarative pipeline)
    - Pipeline stages
    - Why you should know Jenkins exists
```

### 07.04 — AWS (Core Services)

```
- (Same as Python roadmap)
- IAM, EC2, ECS, Lambda, VPC, S3, RDS, ElastiCache, SQS, SNS, CloudWatch
- Java-specific:
    - AWS SDK for Java v2:
        - S3Client, SqsClient, DynamoDbClient
        - Async clients
        - Credential providers
    - Lambda with Java:
        - RequestHandler interface
        - Cold start problem (worse in Java than Python)
        - SnapStart (AWS Lambda SnapStart for Java)
        - GraalVM native image for Lambda
        - When Java Lambda makes sense vs not
    - Elastic Beanstalk (overview — easy deployment for Java apps)
    - MSK (Managed Streaming for Kafka)
```

### 07.05 — Infrastructure as Code

```
- (Same as Python roadmap)
- Terraform: all same concepts
- Java-specific:
    - CDK (AWS Cloud Development Kit) — Java version:
        - Define infrastructure in Java
        - Constructs
        - Stacks
        - Comparison with Terraform
```

### 07.06 — Monitoring, Logging, Observability

```
- (Same theory as Python roadmap)

Java-specific:
    - Micrometer:
        - What it is (metrics facade, like SLF4J for metrics)
        - Meter types: Counter, Gauge, Timer, DistributionSummary
        - Micrometer + Prometheus
        - Micrometer + Spring Boot Actuator
        - Custom metrics
        - @Timed annotation
    - Prometheus endpoint via Actuator
    - Grafana dashboards for JVM metrics:
        - Heap memory usage
        - GC pauses
        - Thread count
        - HTTP request duration
        - Database connection pool stats
    - Structured logging:
        - Logback with JSON encoder (logstash-logback-encoder)
        - MDC for request tracing
        - Correlation IDs (Spring Cloud Sleuth / Micrometer Tracing)
    - Distributed tracing:
        - Micrometer Tracing (successor to Spring Cloud Sleuth)
        - OpenTelemetry Java agent
        - Zipkin or Jaeger
        - Trace context propagation across services
    - ELK Stack:
        - Filebeat shipping logs to Elasticsearch
        - Kibana dashboards
    - JVM monitoring:
        - JMX (Java Management Extensions)
        - VisualVM
        - JFR (Java Flight Recorder)
        - JMC (Java Mission Control)
    - Health checks:
        - Actuator health indicators
        - Custom health indicators (database, Redis, Kafka)
        - Liveness vs readiness probes
```

### 07.07 — Kubernetes

```
- (Same as Python roadmap — all theory and core objects)

Java-specific:
    - JVM resource configuration in Kubernetes:
        - Resource requests and limits
        - JVM heap vs container memory
        - -XX:MaxRAMPercentage=75.0
    - Startup probes (important for Java — slower startup)
    - Graceful shutdown:
        - server.shutdown=graceful in Spring Boot
        - preStop hooks
    - ConfigMaps and Secrets for Spring Boot:
        - spring.config.import=configtree:
        - Kubernetes-native configuration
    - Spring Cloud Kubernetes (service discovery via Kubernetes API)
    - Helm charts for Spring Boot applications
    - Skaffold (local Kubernetes development — overview)
    - EKS deployment
```

### PHASE 07 CHAPTER PROJECT

```
Project 07: "Deploy the Banking API"
- Dockerize the Phase 05 Banking API:
    - Multi-stage build
    - Layered JAR
    - JVM memory configuration
- Docker Compose for local development (app, PostgreSQL, Redis, Kafka)
- GitHub Actions CI/CD pipeline:
    - Checkstyle, test (Testcontainers), build on PR
    - Build and push Docker image on merge to main
    - Deploy to AWS ECS Fargate
- Terraform for AWS infrastructure:
    - VPC, subnets, security groups
    - ECS cluster and service
    - RDS PostgreSQL
    - ElastiCache Redis
    - ALB
    - ECR
- Monitoring:
    - Actuator + Micrometer + Prometheus
    - CloudWatch logging (JSON structured logs)
    - Health check and readiness endpoints
    - Grafana dashboard (local) with JVM and application metrics
- Full documentation: architecture diagram, deployment guide, runbook
- This completes Portfolio Project 02 (now production-grade)
```

---

## PHASE 08 / FINTECH, DATA SYSTEMS, AND SPECIALIZATION
**Follow the structure down.**
*Duration: 5-6 weeks at 6 hours/day*

---

### 08.01 — Financial Data Concepts

```
- (Same as Python roadmap — all finance theory)
- Financial instruments, market data, time series
- Currency handling:
    - BigDecimal (not double, not float — ever):
        - Scale and precision
        - Rounding modes (HALF_UP, HALF_EVEN — banker's rounding)
        - Arithmetic operations
        - Comparison (compareTo, not equals for BigDecimal)
    - JSR 354 (JavaMoney API — overview)
    - Storing money in smallest unit (cents) as long
    - Currency codes (java.util.Currency, ISO 4217)
- Double-entry bookkeeping
- Payment processing
- Regulatory concepts (KYC, AML, PCI DSS, SOC 2)
- Audit trails
```

### 08.02 — Data Pipeline Architecture

```
- (Same theory as Python roadmap)
- ETL vs ELT
- Batch vs stream processing

Java-specific:
    - Apache Kafka deep dive (revisited):
        - Producer API (fine-grained control: acks, retries, idempotence)
        - Consumer API (manual offset management, rebalance listeners)
        - Kafka Streams (deep dive):
            - Topologies
            - State stores (RocksDB)
            - Interactive queries
            - Windowed aggregations (tumbling, hopping, sliding, session)
            - Exactly-once processing
            - Testing (TopologyTestDriver)
        - Kafka Connect:
            - Source and sink connectors
            - JDBC connector
            - Debezium CDC connector
            - Schema Registry and Avro
    - Apache Flink (overview):
        - What it is (stream processing framework)
        - DataStream API
        - When Flink vs Kafka Streams
    - Spring Batch:
        - What it is (batch processing framework)
        - Job, Step, ItemReader, ItemProcessor, ItemWriter
        - Chunk-oriented processing
        - Job parameters and scheduling
        - Restart and retry
        - Use cases: end-of-day processing, report generation, data migration
    - Apache Airflow (awareness — more Python-centric, but used in mixed stacks)
    - Data validation frameworks
    - Idempotent processing patterns
```

### 08.03 — Streaming and Real-Time Systems

```
- (Same theory as Python roadmap)
- WebSocket with STOMP (revisited)
- Server-Sent Events
- Kafka Streams for real-time aggregation
- Redis Streams
- Back-pressure handling (Project Reactor — Mono and Flux)
- Spring WebFlux (reactive web framework):
    - What reactive programming is
    - Project Reactor:
        - Mono<T> (0 or 1 element)
        - Flux<T> (0 to N elements)
        - Operators: map, flatMap, filter, zip, merge, concat
        - Schedulers
        - Error handling
        - Backpressure
    - When to use WebFlux vs Spring MVC
    - R2DBC (Reactive Relational Database Connectivity)
    - Reactive Redis (ReactiveRedisTemplate)
    - NOT the default choice — understand when reactive is justified
```

### 08.04 — High-Performance Java

```
- JVM tuning:
    - Garbage collectors:
        - Serial GC
        - Parallel GC
        - G1 GC (default since Java 9)
        - ZGC (low-latency, Java 15+)
        - Shenandoah GC
        - When to use each
    - GC tuning flags:
        - -Xms, -Xmx (heap size)
        - -XX:+UseG1GC, -XX:+UseZGC
        - -XX:MaxGCPauseMillis
        - -XX:NewRatio, -XX:SurvivorRatio
    - GC log analysis
    - JVM memory model:
        - Young generation (Eden, Survivor)
        - Old generation
        - Metaspace
        - Stack

- Profiling:
    - JFR (Java Flight Recorder):
        - Starting recordings
        - Analyzing with JMC
    - VisualVM:
        - Heap dumps
        - Thread dumps
        - CPU profiling
    - async-profiler (low-overhead profiler)
    - YourKit, JProfiler (commercial — awareness)

- Performance optimization:
    - Object creation and GC pressure
    - String pooling and StringBuilder
    - Primitive vs wrapper types (autoboxing overhead)
    - Data structure choice (ArrayList vs LinkedList, HashMap capacity)
    - Stream API overhead (when to use, when to avoid for hot paths)
    - Connection pooling (HikariCP tuning)
    - Thread pool sizing (CPU-bound vs I/O-bound formulas)
    - Batch operations (JDBC batch, JPA batch)
    - Caching strategies (Caffeine, Redis)
    - Lazy initialization

- Concurrency performance:
    - Lock contention profiling
    - Lock-free data structures
    - False sharing
    - ThreadLocal for thread-safe caching
    - Virtual threads for I/O-bound scaling

- JIT compilation:
    - How JIT works
    - Warm-up effects
    - -XX:+PrintCompilation
    - Tiered compilation

- Benchmarking:
    - JMH (Java Microbenchmark Harness):
        - @Benchmark, @State, @Setup
        - Modes: Throughput, AverageTime, SampleTime
        - Avoiding common pitfalls (dead code elimination, constant folding)
        - Interpreting results

- GraalVM native image:
    - What it is (ahead-of-time compilation)
    - Benefits: instant startup, low memory
    - Limitations: reflection, dynamic proxies
    - Spring Boot native image support
    - When to use (serverless, CLI tools, containers)
```

### 08.05 — Security for Fintech

```
- (Same theory as Python roadmap — OWASP, encryption, audit)

Java-specific:
    - Spring Security (revisited for advanced security):
        - Method-level security in depth
        - Custom security expressions
        - Security events and auditing
    - Encryption in Java:
        - javax.crypto (Cipher, SecretKey, KeyGenerator)
        - AES encryption/decryption
        - RSA key pairs
        - Digital signatures
        - Java KeyStore (JKS, PKCS12)
    - Hashing:
        - MessageDigest (SHA-256)
        - BCrypt (Spring Security)
        - Argon2 (Spring Security)
    - Secrets management:
        - Spring Cloud Config with encryption
        - AWS Secrets Manager with Spring Boot
        - Vault (HashiCorp) with Spring Cloud Vault
    - Input sanitization:
        - OWASP Java HTML Sanitizer
        - Prepared statements (always)
    - Dependency scanning:
        - OWASP Dependency-Check Maven/Gradle plugin
        - Snyk
        - Dependabot
    - mTLS between services
    - Security testing:
        - OWASP ZAP (overview)
        - Penetration testing concepts
```

### PHASE 08 CHAPTER PROJECTS

```
Project 08A: "Transaction Ledger System"
- Double-entry bookkeeping engine
- PostgreSQL with BigDecimal handling
- Immutable transaction log (append-only table)
- Balance calculation from journal entries
- Reconciliation endpoint
- Audit trail with timestamps, actor, IP address
- Idempotent transaction processing (idempotency keys)
- Optimistic locking for concurrent balance updates
- Full test suite with edge cases:
    - Overdrafts
    - Concurrent transfers
    - Currency handling
    - Idempotency verification
- JMH benchmarks for critical paths

Project 08B: "Market Data Pipeline"
- Ingest mock stock market data (or free API)
- Kafka producer publishing tick data
- Kafka Streams computing OHLCV candles (1-min, 5-min, 1-hour):
    - Windowed aggregations
    - State stores
    - Exactly-once processing
- PostgreSQL (time-partitioned tables) for candle storage
- Spring Batch for end-of-day aggregation
- Redis for latest price cache
- Spring Boot REST endpoints for historical candle data
- WebSocket (STOMP) for real-time price streaming
- Resilience4j circuit breakers for external API calls
- Micrometer + Prometheus for pipeline metrics
- Docker Compose for entire stack
- This is Portfolio Project 03 (evolved from Phase 06)
```

---

## PHASE 09 / SENIOR TO PRINCIPAL TRAJECTORY
**Follow the structure down.**
*Duration: Ongoing — this is the rest of your career*

---

### 09.01 — Architecture Patterns (Senior Level)

```
- (Same theory as Python roadmap)
- Layered, hexagonal, clean architecture
- Domain-Driven Design (DDD) in Java:
    - DDD is more natural in Java than Python:
        - Rich domain models (entities with behavior)
        - Value objects (records in Java 16+)
        - Aggregates and aggregate roots
        - Domain events
        - Repositories (interface in domain, implementation in infrastructure)
        - Application services
        - Domain services
    - Bounded contexts and context mapping
    - Anti-corruption layer
    - DDD with Spring Boot:
        - Module structure by bounded context
        - Package structure:
            /domain (entities, value objects, repositories, domain services)
            /application (use cases, command/query handlers)
            /infrastructure (JPA repositories, Kafka producers, external APIs)
            /api (controllers, DTOs)

- SOLID principles in Java:
    - Single Responsibility (separate concerns into classes)
    - Open/Closed (strategy pattern, inheritance)
    - Liskov Substitution (proper inheritance hierarchies)
    - Interface Segregation (small, focused interfaces)
    - Dependency Inversion (depend on abstractions, Spring DI)

- Design patterns in Java:
    - Creational:
        - Factory Method, Abstract Factory
        - Builder (Lombok @Builder, manual builders)
        - Singleton (Spring beans are singletons by default)
        - Prototype
    - Structural:
        - Adapter, Decorator, Facade, Proxy, Composite
        - Proxy pattern (Spring AOP uses it)
    - Behavioral:
        - Observer (Spring Events)
        - Strategy (interface + implementations + DI)
        - Command, Chain of Responsibility
        - Template Method
        - State, Visitor
    - Repository, Unit of Work, Specification (revisited)

- Twelve-Factor App methodology (revisited with Spring Boot mapping)
- Clean Code principles (Robert C. Martin):
    - Naming
    - Functions (small, one purpose)
    - Comments (why, not what)
    - Error handling
    - Classes (small, single responsibility)
    - Tests (FIRST: Fast, Independent, Repeatable, Self-validating, Timely)
```

### 09.02 — Distributed Systems (Staff Level)

```
- (Same theory as Python roadmap)
- Fallacies of distributed computing
- Consensus: Paxos, Raft
- Replication and partitioning
- Distributed transactions (2PC, Saga)
- Distributed locking

Java-specific:
    - ZooKeeper (Java-native, used for leader election, distributed locking)
    - etcd (comparison)
    - Hazelcast (distributed data structures, computing)
    - Apache Curator (ZooKeeper recipes)
    - Consistency patterns in Spring Boot microservices:
        - Outbox pattern implementation
        - Saga orchestrator implementation
        - Idempotent consumer implementation

- Reactive systems (Reactive Manifesto):
    - Responsive, resilient, elastic, message-driven
    - Akka (overview — actor model for JVM)
    - When reactive architecture matters

- Books:
    - "Designing Data-Intensive Applications" (Kleppmann)
    - "System Design Interview" (Alex Xu)
    - "Release It!" (Michael Nygard)
    - "Building Microservices" (Sam Newman)
    - "Domain-Driven Design" (Eric Evans)
    - "Implementing Domain-Driven Design" (Vaughn Vernon)
    - "Effective Java" (Joshua Bloch) — essential Java book
    - "Java Concurrency in Practice" (Goetz)
```

### 09.03 — Technical Leadership (Staff Level)

```
- (Same as Python roadmap)
- Writing RFCs and design documents
- Architecture Decision Records (ADRs)
- Code review practices
- Technical debt management
- Incident management and post-mortems

Java-specific:
    - JEP (JDK Enhancement Proposal) awareness — tracking Java evolution
    - Evaluating frameworks and libraries (Spring vs Quarkus, Hibernate vs jOOQ)
    - Migration planning (Java version upgrades, Spring Boot major versions)
    - Performance review processes for backend systems
    - Capacity planning for JVM-based services
```

### 09.04 — Organizational Impact (Principal Level)

```
- (Same as Python roadmap)
- Technical vision and strategy
- Technology radar
- Cross-team alignment
- Build vs buy
- Platform engineering
- Developer experience
- Mentoring and growing engineers

Java-specific:
    - Platform team: shared libraries, starter templates, internal frameworks
    - Java ecosystem governance:
        - JDK version policy across teams
        - Shared BOM (Bill of Materials) for dependency alignment
        - Internal Maven/Gradle plugins
        - Architecture fitness functions (ArchUnit):
            - Enforcing layer dependencies
            - Enforcing naming conventions
            - Detecting circular dependencies
    - Performance culture:
        - SLA-driven development
        - Performance budgets
        - Capacity planning models

- Books:
    - "Staff Engineer" (Will Larson)
    - "An Elegant Puzzle" (Will Larson)
    - "The Staff Engineer's Path" (Tanya Reilly)
    - "Accelerate" (Forsgren, Humble, Kim)
    - "Team Topologies" (Skelton, Pais)
    - "Fundamentals of Software Architecture" (Richards, Ford)
```

### 09.05 — Continuous Learning Map

```
- Databases to learn when needed:
    - TimescaleDB, ClickHouse, Cassandra, Neo4j (same as Python roadmap)

- Languages to learn alongside Java:
    - Kotlin (fully interoperable with Java, increasingly used in Spring Boot)
    - Go (high-performance services, DevOps tooling)
    - Python (data engineering, ML integration, scripting)
    - SQL (you should already be strong)
    - TypeScript (if touching frontend or Node.js)

- Frameworks to explore:
    - jOOQ (type-safe SQL in Java — alternative to JPA for complex queries)
    - Vert.x (reactive toolkit)
    - Helidon (Oracle's microservices framework)

- Cloud certifications:
    - AWS Solutions Architect Associate
    - AWS Developer Associate
    - AWS Solutions Architect Professional

- Tools:
    - Pulumi (IaC in Java/TypeScript)
    - ArgoCD (GitOps)
    - Istio (service mesh)
    - Vault, Consul

- Open source contribution (Spring ecosystem, Apache projects)
- Speaking, writing, building in public
```

---

## PORTFOLIO PROJECTS — SUMMARY

---

### Portfolio Project 01: CLI Finance Tracker
```
Phase built in: 01
Stack: Java 21, Jackson, JUnit 5, Mockito, Maven/Gradle
Demonstrates: Java fundamentals, OOP, collections, streams, file I/O, testing
GitHub: clean README, CI with GitHub Actions, Javadoc
```

### Portfolio Project 02: Banking REST API (Production-Grade)
```
Phases built in: 05 + 07
Stack: Spring Boot 3, Spring Security, Spring Data JPA, PostgreSQL, Redis,
       JWT, Flyway, MapStruct, Docker, AWS ECS, Terraform, GitHub Actions,
       Micrometer, Prometheus
Demonstrates: API design, security, database design, caching, containerization,
              CI/CD, cloud deployment, monitoring, observability
GitHub: architecture diagram, API docs (Swagger), deployment guide, test coverage
```

### Portfolio Project 03: Real-Time Market Data Pipeline
```
Phases built in: 06 + 08
Stack: Spring Boot, Kafka, Kafka Streams, Spring Batch, PostgreSQL, Redis,
       WebSocket (STOMP), Docker, Resilience4j, Micrometer, Prometheus
Demonstrates: event-driven architecture, stream processing, data pipelines,
              real-time systems, time-series data, batch processing
GitHub: architecture diagram, data flow docs, performance benchmarks
```

### Portfolio Project 04: Microservices Payment Platform
```
Phase built in: 08 + 09
Stack: Spring Boot (multiple services), Spring Cloud, gRPC, Kafka,
       PostgreSQL, Redis, Docker, Kubernetes, Terraform, Resilience4j
Services:
    - User service
    - Account service
    - Payment service (double-entry ledger)
    - Notification service
    - API Gateway (Spring Cloud Gateway)
    - Config Server (Spring Cloud Config)
Demonstrates: microservices, distributed transactions (saga), inter-service
              communication, service discovery, circuit breakers,
              container orchestration, DDD
GitHub: mono-repo with clear service boundaries, ADRs, deployment manifests
```

### Portfolio Project 05: Distributed Fintech System
```
Phase built in: 09
Stack: Everything from previous projects + CQRS, event sourcing,
       distributed tracing, chaos testing
This is the evolution of Project 04 with:
    - Event sourcing for the ledger (Axon or custom)
    - CQRS for read/write separation
    - Outbox pattern for reliable event publishing
    - OpenTelemetry distributed tracing (Jaeger)
    - Chaos testing (Chaos Monkey for Spring Boot)
    - ArchUnit fitness functions
    - Comprehensive observability dashboard
    - Technical design document
    - Architecture Decision Records
Demonstrates: principal-level system design, distributed systems,
              DDD, technical documentation
```

---

## TIMELINE ESTIMATE (at 6 hours/day)

```
Phase 00: 2-3 weeks
Phase 01: 6-7 weeks (Java has more upfront ceremony)
Phase 02: 2-3 weeks
Phase 03: 4-5 weeks
Phase 04: 4-5 weeks
Phase 05: 6-8 weeks (Spring ecosystem is massive)
Phase 06: 4-5 weeks
Phase 07: 5-6 weeks
Phase 08: 5-6 weeks
---
Total to job-ready (junior-mid): ~40-48 weeks (9-12 months)

Phase 09: Ongoing
- Senior: 2-4 years of professional experience
- Staff: 5-8 years of professional experience
- Principal: 10+ years of professional experience
```

---

## KEY DIFFERENCES FROM THE PYTHON ROADMAP

```
1. Java Phase 01 is longer: OOP depth, generics, concurrency, type system
2. Build tools are mandatory (Maven/Gradle) — Python has pip/poetry but they are simpler
3. Spring Boot replaces FastAPI/Django — Spring is a larger ecosystem
4. Concurrency is a first-class topic in Java (threads, locks, concurrent collections,
   virtual threads) — Python has GIL limitations
5. JVM tuning and GC knowledge is required at senior+ levels
6. Testing uses JUnit 5 + Mockito + Testcontainers (vs pytest)
7. Java is the dominant language in enterprise fintech — Python dominates
   data/ML/startups
8. Kafka Streams is a native Java library — better integration than Python
9. DDD is more naturally expressed in Java's type system
10. Interview preparation: Java interviews tend to be more algorithm and
    system-design heavy
```
