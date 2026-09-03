## Overview

An Integrated Development Environment (IDE) is the **primary workspace** where you write, navigate, debug, test, and refactor Java code. While the terminal handles builds and deployments, the IDE is where you spend 80% of your working hours. A Java backend engineer who has not mastered their IDE is like a surgeon who has not mastered their scalpel—technically capable, but operating at a fraction of their potential speed and precision.

For Java backend development, **IntelliJ IDEA** by JetBrains is the undisputed industry standard. It dominates enterprise Java, fintech, and Spring Boot development to a degree that no other IDE approaches. Its deep understanding of Java's type system, its Spring-aware code intelligence, its integrated debugger, and its refactoring capabilities are unmatched. If you walk into a bank's engineering floor, 95% of the screens will show IntelliJ.

**VS Code** with the Java Extension Pack is a viable alternative for lighter workloads, quick edits, and polyglot development. It is faster to launch, uses less memory, and excels when you work across multiple languages (Java + Python + TypeScript + Terraform). However, for deep Spring Boot development, complex refactoring, and large enterprise codebases, IntelliJ remains the superior choice.

This note covers IntelliJ IDEA in exhaustive detail—shortcuts, debugging, refactoring, Spring Boot integration, plugins, performance tuning—and provides a practical guide to VS Code with Java for the scenarios where it makes sense.

---

## Core Concepts

### IDE vs Text Editor vs Terminal

```
Terminal (zsh/bash):
  → Build, test, deploy, version control, process management
  → No code intelligence, no GUI
  → Essential for automation and CI/CD

Text Editor (VS Code, Sublime, Vim):
  → Lightweight, fast startup, extensible via plugins
  → Code intelligence depends on language server plugins
  → Good for quick edits, scripting, polyglot projects

IDE (IntelliJ IDEA, Eclipse):
  → Full code intelligence: type inference, refactoring, navigation
  → Integrated debugger, test runner, profiler, database client
  → Deep framework awareness (Spring, JPA, Maven/Gradle)
  → Heavier resource usage (RAM, CPU, startup time)
  → The right tool for sustained, complex Java development
```

### IntelliJ IDEA Editions

```
Community Edition (Free, Open Source):
  → Full Java SE support
  → Maven, Gradle, Ant build tools
  → JUnit 5, TestNG test runners
  → Git, GitHub, Mercurial integration
  → Basic refactoring (rename, extract method, extract variable)
  → Debugger (breakpoints, step, evaluate)
  → Android development
  → Kotlin, Groovy, Scala (basic)
  → LIMITATIONS:
      ✗ No Spring/Spring Boot support
      ✗ No Jakarta EE / Java EE support
      ✗ No database tools (DataGrip integration)
      ✗ No HTTP client
      ✗ No Docker/Kubernetes integration (basic only)
      ✗ No profiler integration
      ✗ No advanced refactoring (extract interface, migrate type)

Ultimate Edition (Paid, ~$149/year individual, free for students):
  → Everything in Community, PLUS:
  → Spring Boot, Spring MVC, Spring Security, Spring Data (full support)
      - Spring bean detection and navigation
      - application.yml autocompletion and validation
      - @Autowired injection analysis
      - Spring endpoint detection and HTTP client generation
  → Jakarta EE (JPA, CDI, JAX-RS, JSF, EJB)
  → Database tools (full DataGrip integration)
      - SQL autocompletion in JPA @Query strings
      - Database schema visualization
      - Query execution and result browsing
      - Migration tool integration
  → HTTP Client (test REST APIs directly from the IDE)
  → Docker and Docker Compose integration
  → Kubernetes integration
  → Profiler (CPU, memory, allocation)
  → Advanced refactoring
  → Microservices tooling (service discovery, endpoint mapping)
  → JavaScript/TypeScript (full WebStorm integration)
  → Database migrations (Flyway, Liquibase awareness)

Recommendation:
  → Start with Community Edition for Phase 01-03 (core Java, DSA)
  → Upgrade to Ultimate for Phase 04+ (databases, Spring Boot)
  → Students: get Ultimate for free at https://www.jetbrains.com/student/
  → Open source contributors: free Ultimate license available
  → If your employer provides licenses, use Ultimate from day one
```

### IntelliJ Project Model

```
IntelliJ's project model differs from Eclipse and VS Code:

Project:
  → The top-level container (usually maps to your repository root)
  → One project window = one project
  → Configuration stored in .idea/ directory (partially version-controlled)

Module:
  → A sub-unit within a project with its own source roots, dependencies, and SDK
  → Maps to Maven modules or Gradle subprojects
  → A multi-module Maven project opens as one IntelliJ project with multiple modules

SDK:
  → The JDK version used for compilation and running
  → Configured per-project or per-module
  → IntelliJ can download and manage JDKs directly (Settings → Build → SDKs)

Facet:
  → Additional framework configuration attached to a module
  → Spring facet, JPA facet, Web facet
  → Usually auto-detected from pom.xml / build.gradle

Run Configuration:
  → A saved configuration for running or debugging your application
  → Application, JUnit, Maven, Gradle, Docker, Remote JVM Debug
  → Stored in .idea/runConfigurations/ (can be version-controlled)
```

### The .idea Directory

```
.idea/                          ← IntelliJ project configuration
├── .gitignore                  ← should ignore most .idea files
├── codeStyles/                 ← code style settings (SHARE THIS)
│   ├── codeStyleConfig.xml
│   └── Project.xml
├── runConfigurations/          ← run/debug configurations (SHARE THIS)
│   └── BankingApplication.xml
├── inspectionProfiles/         ← inspection settings (SHARE THIS)
│   └── Project_Default.xml
├── compiler.xml                ← compiler settings
├── encodings.xml               ← file encoding settings
├── jarRepositories.xml         ← Maven repository settings
├── misc.xml                    ← project SDK, language level
├── modules.xml                 ← module definitions
├── vcs.xml                     ← version control mapping
├── workspace.xml               ← YOUR local workspace state (DO NOT SHARE)
└── *.iml                       ← module files (legacy, mostly replaced by Maven/Gradle)

What to version-control:
  ✓ .idea/codeStyles/           → team-wide formatting consistency
  ✓ .idea/runConfigurations/    → shared run configurations
  ✓ .idea/inspectionProfiles/   → shared inspection rules
  ✓ .idea/externalDependencies.xml → required plugins
  ✗ .idea/workspace.xml         → personal window layout, cursor positions
  ✗ .idea/tasks.xml             → personal task history
  ✗ .idea/shelf/                → personal shelved changes
  ✗ *.iml                       → if using Maven/Gradle (regenerated on import)

Recommended .gitignore entries:
  .idea/workspace.xml
  .idea/tasks.xml
  .idea/shelf/
  .idea/dataSources/
  .idea/dataSources.local.xml
  *.iml
```

---

## Code Examples

### Essential Keyboard Shortcuts

Mastering shortcuts is the single highest-ROI investment in IDE productivity. Every mouse click you replace with a keyboard shortcut saves 2-5 seconds. Across hundreds of actions per day, this compounds into hours per week.

**The shortcuts below use macOS key bindings. For Windows/Linux, substitute:**
- `Cmd` → `Ctrl`
- `Option (Alt)` → `Alt`
- `Ctrl` → `Ctrl` (mostly the same)

#### Navigation

```
Cmd + O              → Go to Class (type class name, fuzzy match)
Cmd + Shift + O      → Go to File (type filename, fuzzy match)
Cmd + Option + O     → Go to Symbol (any method, field, variable)
Cmd + E              → Recent Files (popup of recently opened files)
Cmd + Shift + E      → Recent Edited Files
Cmd + B              → Go to Declaration (jump to where a method/class is defined)
Cmd + Option + B     → Go to Implementation (jump to implementation of interface)
Cmd + U              → Go to Super Method/Class
Cmd + Shift + I      → Quick Definition (preview without leaving current file)
Cmd + [              → Navigate Back (to previous cursor position)
Cmd + ]              → Navigate Forward
Cmd + F12            → File Structure (popup of methods/fields in current file)
Cmd + Shift + A      → Find Action (search any IDE action by name — THE most useful shortcut)
Double Shift         → Search Everywhere (files, classes, symbols, actions, settings)
Cmd + G              → Go to Line Number
Cmd + Shift + F      → Find in Files (search across entire project)
Cmd + Shift + R      → Replace in Files
Ctrl + H             → Type Hierarchy (show class hierarchy)
Ctrl + Shift + H     → Method Hierarchy
Ctrl + Option + H    → Call Hierarchy (who calls this method?)
Cmd + Shift + G      → Find Usages (where is this class/method/field used?)
Option + F7          → Find Usages (with more options)
Cmd + Click          → Go to Declaration (mouse alternative to Cmd+B)
Cmd + Shift + Click  → Go to Implementation
```

#### Editing

```
Cmd + D              → Duplicate current line or selection
Cmd + Y              → Delete current line
Cmd + Shift + Up     → Move statement up
Cmd + Shift + Down   → Move statement down
Option + Shift + Up  → Move line up
Option + Shift + Down → Move line down
Cmd + /              → Toggle line comment (// ...)
Cmd + Option + /     → Toggle block comment (/* ... */)
Cmd + Shift + J      → Join lines (merge current line with next)
Cmd + Enter          → Complete statement (add semicolon, braces, etc.)
Shift + Enter        → Start new line (regardless of cursor position)
Cmd + Option + L     → Reformat Code (apply code style)
Cmd + Option + O     → Optimize Imports (remove unused, reorder)
Ctrl + Space         → Basic Code Completion
Ctrl + Shift + Space → Smart Type Completion (filters by expected type)
Cmd + Shift + Enter  → Complete Current Statement
Cmd + J              → Insert Live Template
Cmd + Option + T     → Surround With (if/else, try/catch, for loop)
Cmd + Shift + V      → Paste from History (clipboard ring)
Cmd + Shift + U      → Toggle Case (UPPER/lower)
Cmd + W              → Extend Selection (select word → expression → statement → block)
Cmd + Shift + W      → Shrink Selection
Option + Enter       → Show Intention Actions (quick fix — THE most-used shortcut)
```

#### Refactoring

```
Shift + F6           → Rename (class, method, variable — updates all usages)
Cmd + Option + M     → Extract Method (select code → extract to new method)
Cmd + Option + V     → Extract Variable (select expression → assign to variable)
Cmd + Option + F     → Extract Field (select expression → assign to class field)
Cmd + Option + C     → Extract Constant (select expression → assign to static final)
Cmd + Option + P     → Extract Parameter (select expression → add as method parameter)
Cmd + Option + N     → Inline (reverse of extract: inline a variable or method)
Cmd + F6             → Change Method Signature (add/remove/reorder parameters)
Cmd + Option + Shift + T → Refactor This (popup with all applicable refactorings)
F6                   → Move (move class to different package)
F5                   → Copy (copy class to different package)
Cmd + Shift + F6     → Type Migration (change type of variable/field/parameter)
```

#### Build and Run

```
Ctrl + Shift + R     → Run current configuration
Ctrl + Shift + D     → Debug current configuration
Ctrl + R             → Run (context-sensitive: runs test under cursor, or app)
Ctrl + D             → Debug (context-sensitive)
Cmd + F2             → Stop running process
Ctrl + Shift + F2    → Stop all running processes
Cmd + Shift + F9     → Build Project (compile without running)
Cmd + F9             → Build Module
```

#### Debugging

```
Cmd + F8             → Toggle Breakpoint (on current line)
Cmd + Shift + F8     → View Breakpoints (list all breakpoints)
F8                   → Step Over (execute current line, don't enter methods)
F7                   → Step Into (enter the method being called)
Shift + F8           → Step Out (exit current method, return to caller)
Option + F9          → Run to Cursor (execute until cursor position)
Option + F8          → Evaluate Expression (evaluate arbitrary code in debug context)
Cmd + Option + R     → Resume Program (continue until next breakpoint)
```

#### Testing

```
Ctrl + Shift + R     → Run all tests in current file
Ctrl + Shift + D     → Debug all tests in current file
Ctrl + R             → Run test under cursor (place cursor on @Test method)
Ctrl + Shift + R     → Re-run last test configuration
Cmd + Shift + F10    → Run context configuration
```

#### Search and Replace

```
Cmd + F              → Find in current file
Cmd + G              → Find Next
Cmd + Shift + G      → Find Previous
Cmd + R              → Replace in current file
Cmd + Shift + F      → Find in Files (project-wide search)
Cmd + Shift + R      → Replace in Files (project-wide replace)
Cmd + Shift + A      → Find Action (search IDE commands)
Double Shift         → Search Everywhere
```

#### Window Management

```
Cmd + 1              → Project tool window (file tree)
Cmd + 2              → Favorites
Cmd + 3              → Find tool window
Cmd + 4              → Run tool window
Cmd + 5              → Debug tool window
Cmd + 6              → Problems tool window
Cmd + 7              → Structure tool window
Cmd + 8              → Git tool window
Cmd + 9              → Version Control / Git Log
Cmd + 0              → Commit tool window
Cmd + Shift + F12    → Hide All Tool Windows (maximize editor)
Shift + Esc          → Hide active tool window
Cmd + Shift + [      → Previous editor tab
Cmd + Shift + ]      → Next editor tab
Cmd + W              → Close active editor tab
```

### Debugging — Deep Dive

IntelliJ's debugger is one of the most powerful in any IDE. It goes far beyond simple breakpoints and step-through.

#### Breakpoint Types

```
Line Breakpoint:
  → Click in the gutter (left margin) on a line number
  → Red dot appears
  → Execution pauses when this line is reached
  → Cmd + F8 to toggle

Conditional Breakpoint:
  → Right-click the breakpoint dot → Condition
  → Enter a boolean expression: userId.equals("admin")
  → Breakpoint only triggers when the condition is true
  → Essential for loops that run thousands of times

Exception Breakpoint:
  → Run → View Breakpoints → + → Java Exception Breakpoints
  → Choose exception class (e.g., NullPointerException)
  → Pauses execution whenever this exception is thrown
  → "Caught" and "Uncaught" checkboxes control when it triggers
  → CRITICAL for debugging unexpected NPEs in production-like scenarios

Method Breakpoint:
  → Place on a method declaration line
  → Pauses on method entry and/or exit
  → Useful for tracing interface implementations

Field Watchpoint:
  → Right-click a field declaration → Toggle Field Watchpoint
  → Pauses when the field is read or written
  → Essential for debugging "who changed this value?"

Temporary Breakpoint:
  → Right-click breakpoint → "Remove once hit"
  → Automatically deleted after first trigger
  → Useful for one-time debugging sessions
```

#### Debug Workflow Example

```java
// Scenario: a transfer is failing with an unexpected balance

@Service
public class TransferService {

    private final AccountRepository accountRepository;

    public TransferResult transfer(String fromId, String toId, BigDecimal amount) {
        Account from = accountRepository.findById(fromId)  // ← BREAKPOINT HERE
            .orElseThrow(() -> new AccountNotFoundException(fromId));
        Account to = accountRepository.findById(toId)
            .orElseThrow(() -> new AccountNotFoundException(toId));

        if (from.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException(fromId, from.getBalance(), amount);
        }

        from.withdraw(amount);   // ← CONDITIONAL BREAKPOINT: from.getBalance().compareTo(new BigDecimal("100")) < 0
        to.deposit(amount);

        accountRepository.save(from);
        accountRepository.save(to);

        return new TransferResult(fromId, toId, amount, from.getBalance());
    }
}
```

**Debug session steps:**

```
1. Set breakpoint on the findById line
2. Start debug: Ctrl + Shift + D (or click the bug icon)
3. Send a test request: curl -X POST localhost:8080/api/transfers ...
4. Debugger pauses at the breakpoint
5. Inspect variables in the "Variables" panel:
   → fromId = "ACC-001"
   → toId = "ACC-002"
   → amount = 500.00
6. Step Over (F8) to execute findById
7. Inspect the "from" object:
   → from.balance = 300.00  ← this is the problem!
8. Evaluate Expression (Option + F8):
   → from.getBalance().compareTo(amount)
   → Result: -1 (balance < amount)
9. Now you understand the bug: the balance check should have caught this,
   but the exception message was misleading. Fix the error message.
10. Resume (Cmd + Option + R) or stop and fix the code.
```

#### Advanced Debugging Features

```
Evaluate Expression (Option + F8):
  → Execute arbitrary Java code in the current debug context
  → Call methods, create objects, modify variables
  → Example: accountRepository.count() to check DB state mid-execution
  → Example: new BigDecimal("100").compareTo(amount) to test logic

Watches:
  → In the Variables panel, click "+" to add a watch expression
  → The expression is evaluated at every step
  → Example: from.getBalance() to watch how balance changes

Force Return:
  → Right-click the current stack frame → Force Return
  → Immediately returns from the current method with a specified value
  → Useful for bypassing problematic code during debugging

Drop Frame:
  → Right-click a stack frame → Drop Frame
  → "Rewinds" execution to the beginning of the method
  → All side effects of the method are NOT undone (database writes persist!)
  → Use with caution

HotSwap (Hot Code Replace):
  → Modify code while debugging, then Cmd + Shift + F9 (Build)
  → IntelliJ reloads the changed classes without restarting the JVM
  → Works for method body changes (not signature changes, not new classes)
  → Spring Boot DevTools enhances this with automatic restart

Stream Debugger:
  → When paused inside a Stream pipeline, click the "Trace Current Stream Chain" icon
  → Visualizes each intermediate operation and its output
  → Incredibly useful for debugging complex Stream chains

Memory View:
  → Debug → Show Memory View
  → See all live objects grouped by class
  → Detect memory leaks during debugging sessions

Async Stack Traces:
  → IntelliJ can trace execution across thread boundaries
  → When a CompletableFuture completes, the stack trace shows
    where the async task was originally submitted
  → Enabled by default in recent versions
```

### Spring Boot Integration (Ultimate Edition)

IntelliJ Ultimate provides **deep Spring awareness** that transforms the development experience:

```
Spring Bean Detection:
  → @Component, @Service, @Repository, @Controller, @Configuration
    classes are marked with Spring icons in the gutter
  → Click the icon to see where the bean is injected
  → @Autowired fields show the resolved bean type

Endpoint Detection:
  → @GetMapping, @PostMapping, etc. show HTTP method icons in the gutter
  → Click to generate an HTTP request in the built-in HTTP Client
  → Services tool window shows all endpoints grouped by controller

application.yml Intelligence:
  → Full autocompletion for all Spring Boot properties
  → Type "server." and see all server.* properties with descriptions
  → Validation: warns about unknown or deprecated properties
  → Navigation: Cmd+Click on a property to jump to its source definition
  → Profile awareness: shows which properties are active per profile

Spring Data JPA:
  → @Query strings get SQL syntax highlighting and autocompletion
  → JPQL validation: warns about invalid entity names or fields
  → Cmd+Click on an entity name in @Query to jump to the entity class
  → Repository method name validation: findByNonExistentField shows a warning

Spring Security:
  → @PreAuthorize expressions are validated
  → Security filter chain visualization

Run Configurations:
  → Auto-detects Spring Boot application classes
  → One-click run with Spring Boot Run Configuration
  → JMX bean browsing during debug sessions
  → Live Beans graph (visualize bean dependencies)
```

### Database Tools (Ultimate Edition)

IntelliJ's built-in database tools eliminate the need for a separate SQL client like DBeaver or pgAdmin:

```
Setup:
  → View → Tool Windows → Database
  → + → Data Source → PostgreSQL
  → Enter host, port, database, user, password
  → Download missing driver (IntelliJ prompts automatically)
  → Test Connection → OK

Features:
  → Schema browser: tables, views, indexes, sequences, functions
  → SQL editor with autocompletion (table names, column names, functions)
  → Query execution with result grid (sortable, filterable, exportable)
  → EXPLAIN ANALYZE visualization
  → Table data editor (inline editing, like a spreadsheet)
  → DDL generation: right-click table → Generate DDL
  → Schema comparison: compare two databases or a database with Flyway scripts
  → SQL dialect detection: knows PostgreSQL vs MySQL vs Oracle syntax
  → JPA integration: @Query strings autocomplete against the actual database schema

JPA Console:
  → Write JPQL queries and execute them against the database
  → Autocompletion uses your JPA entity model
  → Useful for testing complex JPQL before putting it in @Query annotations

Database in Spring Boot:
  → When a DataSource is configured in application.yml, IntelliJ
    auto-detects the connection and links it to the database tools
  → SQL in @Query annotations is validated against the real schema
```

### HTTP Client (Ultimate Edition)

IntelliJ's built-in HTTP Client replaces Postman for API testing during development:

```http
### Create a new account
POST http://localhost:8080/api/accounts
Content-Type: application/json
Authorization: Bearer {{auth_token}}

{
  "ownerName": "Alice Johnson",
  "accountType": "CHECKING",
  "initialDeposit": 1000.00
}

### Get account balance
GET http://localhost:8080/api/accounts/ACC-001/balance
Authorization: Bearer {{auth_token}}

### Transfer funds
POST http://localhost:8080/api/transfers
Content-Type: application/json
Authorization: Bearer {{auth_token}}

{
  "fromAccountId": "ACC-001",
  "toAccountId": "ACC-002",
  "amount": 250.00,
  "description": "Rent payment"
}

### Login and capture token
POST http://localhost:8080/api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "secret"
}

> {%
    client.global.set("auth_token", response.body.token);
%}
```

**Features:**
```
→ .http files are version-controlled (unlike Postman collections)
→ Environment variables: http-client.env.json for dev/staging/prod
→ Response handling scripts (> {% %}) for extracting tokens
→ Request history and replay
→ Convert from cURL: paste a curl command and IntelliJ converts it
→ WebSocket testing (STOMP support)
→ gRPC testing (Ultimate 2023.2+)
```

### Essential Plugins

```
Must-Have (install immediately):
  → Lombok (Community: required for @Data, @Builder, etc.)
  → SonarLint (real-time code quality feedback)
  → .env files support (syntax highlighting for .env files)
  → Key Promoter X (shows keyboard shortcut when you use the mouse — trains you)
  → Rainbow Brackets (colorizes matching brackets — saves your sanity in nested code)

Highly Recommended:
  → Docker (container management, Dockerfile editing, Compose support)
  → Kubernetes (K8s manifest editing, cluster browsing)
  → GitToolBox (inline blame annotations, ahead/behind status)
  → String Manipulation (case conversion, escaping, sorting)
  → JPA Buddy (JPA entity generation, Liquibase/Flyway integration)
  → Maven Helper (visualize dependency tree, find conflicts)
  → JRebel (advanced hot-reload — commercial, but transformative for Spring Boot)
  → AI Assistant (JetBrains AI — code generation, explanation, refactoring)

For Fintech / Data:
  → CSV Editor (view and edit CSV files in a table)
  → JSON Parser (validate and format JSON)
  → Big Data Tools (Spark, Hadoop, S3 browsing — Ultimate)
  → Database Navigator (alternative to built-in DB tools for Community Edition)

Theme and Appearance:
  → One Dark Theme / Dracula Theme / Catppuccin (easier on the eyes)
  → Atom Material Icons (better file type icons)
  → Extra Icons (icons for .env, Dockerfile, etc.)
```

### Live Templates

Live Templates are **code snippets** that expand from abbreviations. They eliminate boilerplate typing:

```
Built-in templates (type the abbreviation and press Tab):

sout       → System.out.println();
soutv      → System.out.println("variable = " + variable);
serr       → System.err.println();
psvm       → public static void main(String[] args) { }
psf        → public static final
prsf       → private static final
thr        → throw new
try        → try { } catch (Exception e) { }
twr        → try-with-resources
fori       → for (int i = 0; i < ; i++) { }
fore       → for (Type item : collection) { }
ifn        → if (var == null) { }
inn        → if (var != null) { }
inst       → if (var instanceof Type) { }
log        → (with Logger field) log.info("");

Custom templates (Settings → Editor → Live Templates):

Example: Create a "test" template for JUnit 5 test methods
  Abbreviation: test
  Template text:
    @Test
    @DisplayName("$DESCRIPTION$")
    void $METHOD_NAME$() {
        // Given
        $END$
        // When

        // Then

    }
  Applicable in: Java → Declaration

Example: Create a "logi" template for SLF4J logging
  Abbreviation: logi
  Template text:
    log.info("$MESSAGE$ {}", $VARS$);
  Applicable in: Java → Statement

Example: Create a "mock" template for Mockito setup
  Abbreviation: mock
  Template text:
    when($MOCK$.$METHOD$($ARGS$)).thenReturn($RETURN$);
  Applicable in: Java → Statement
```

### Project Structure Configuration

```
Settings → Project Structure (Cmd + ;)

Project:
  → SDK: select JDK 21 (or download via IntelliJ)
  → Language Level: 21 (enables pattern matching, records, sealed classes)

Modules:
  → Sources tab:
      src/main/java → Sources (blue)
      src/main/resources → Resources (green)
      src/test/java → Tests (green)
      src/test/resources → Test Resources (green)
      target/ → Excluded (orange)
  → Dependencies tab:
      Module SDK
      Maven/Gradle dependencies (auto-populated)
      Module dependencies (for multi-module projects)

Facets:
  → Spring facet (auto-detected from Spring Boot starter)
  → JPA facet (auto-detected from spring-boot-starter-data-jpa)
  → Web facet (auto-detected from spring-boot-starter-web)

Artifacts:
  → JAR, WAR, exploded archive configurations
  → Usually managed by Maven/Gradle, not configured manually
```

### Memory and Performance Settings

IntelliJ is a memory-hungry application. For large Spring Boot projects, the default settings are often insufficient:

```
Help → Edit Custom VM Options (or idea.vmoptions):

# Heap size (increase for large projects)
-Xms1g          # initial heap (default: 256m)
-Xmx4g          # maximum heap (default: 2g, increase to 4-8g for large projects)

# Garbage collector (G1 is default, good for most cases)
-XX:+UseG1GC

# GC tuning for responsiveness
-XX:SoftRefLRUPolicyMSPerMB=50
-XX:ReservedCodeCacheSize=512m
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow

# For very large projects (1M+ lines of Java):
-Xmx8g
-XX:+UseZGC        # low-latency GC (Java 17+)
-XX:+ZGenerational  # generational ZGC (Java 21+)

Signs you need more memory:
  → IDE freezes during indexing
  → "Low Memory" notification in the status bar
  → Slow code completion
  → Frequent GC pauses (visible in the memory indicator)

Other performance tips:
  → Exclude large directories from indexing:
    Right-click → Mark Directory as → Excluded
    Exclude: target/, build/, node_modules/, .gradle/, logs/
  → Disable unused plugins (Settings → Plugins)
  → Increase shared index storage: Settings → Tools → Shared Indexes
  → Use SSD (IntelliJ on an HDD is painfully slow)
  → Close unused projects (each open project consumes memory)
  → Invalidate Caches (File → Invalidate Caches) when things get weird
```

### VS Code with Java

VS Code is a lightweight alternative that works well for specific scenarios:

**Setup:**

```
1. Install VS Code: https://code.visualstudio.com/

2. Install the Extension Pack for Java (Microsoft):
   → Includes:
      - Language Support for Java (Red Hat) — LSP-based Java intelligence
      - Debugger for Java — breakpoints, stepping, variables
      - Test Runner for Java — JUnit 5, TestNG
      - Maven for Java — project management
      - Project Manager for Java — project explorer
      - IntelliCode — AI-assisted completions

3. Install Spring Boot Extension Pack (VMware/Broadcom):
   → Spring Boot Tools — application.yml support, bean detection
   → Spring Initializr — create new Spring Boot projects
   → Spring Boot Dashboard — run/debug Spring Boot apps

4. Install additional recommended extensions:
   → Gradle for Java
   → Docker
   → GitLens (Git blame, history, comparison)
   → SonarLint
   → YAML (Red Hat)
   → XML (Red Hat)
   → EditorConfig for VS Code
```

**VS Code Java Capabilities:**

```
Works well:
  → Writing and editing Java code (syntax highlighting, basic completion)
  → Running and debugging single applications
  → Running JUnit 5 tests
  → Maven and Gradle project management
  → Spring Boot application.yml editing
  → Git integration (with GitLens)
  → Quick edits across multiple languages (Java + Python + Terraform)
  → Remote development (SSH, WSL, Dev Containers)
  → Lightweight resource usage (500MB-1GB vs IntelliJ's 2-4GB)

Limitations compared to IntelliJ:
  → Refactoring is basic (rename works, extract method is limited)
  → No deep Spring bean navigation (can't click @Autowired to see the bean)
  → No SQL autocompletion in @Query strings
  → No built-in database client (use DBeaver or pgAdmin separately)
  → No HTTP client (use REST Client extension or Postman)
  → No profiler integration
  → No stream debugger
  → No decompiler (IntelliJ decompiles .class files on the fly)
  → Slower indexing on very large projects (100K+ files)
  → Less reliable code completion for complex generic types
  → No JPA entity relationship visualization
```

**VS Code settings.json for Java:**

```json
{
    "java.configuration.runtimes": [
        {
            "name": "JavaSE-21",
            "path": "~/.sdkman/candidates/java/21.0.2-tem",
            "default": true
        }
    ],
    "java.jdt.ls.vmargs": "-Xmx2G -XX:+UseG1GC",
    "java.compile.nullAnalysis.mode": "automatic",
    "java.format.settings.url": "https://raw.githubusercontent.com/google/styleguide/gh-pages/eclipse-java-google-style.xml",
    "java.saveActions.organizeImports": true,
    "java.completion.importOrder": ["java", "javax", "org", "com", ""],
    "java.test.defaultConfig": "JUnit 5",
    "editor.formatOnSave": true,
    "[java]": {
        "editor.defaultFormatter": "redhat.java"
    }
}
```

### When to Use Which IDE

```
Use IntelliJ IDEA when:
  → You are doing serious Spring Boot development (Phase 05+)
  → You need deep refactoring capabilities (extract interface, migrate type)
  → You work with JPA/Hibernate and need SQL autocompletion in @Query
  → You need the built-in database client
  → You are debugging complex multi-threaded or async code
  → You work on a large enterprise codebase (500K+ lines)
  → Your team uses IntelliJ (shared run configs, code styles)
  → You need the Spring Boot Dashboard and endpoint mapping
  → You want the best possible Java code completion and navigation

Use VS Code when:
  → You are learning Java basics (Phase 01-03) and want a lightweight editor
  → You work across many languages daily (Java + Python + Go + TypeScript)
  → You are doing quick edits on a remote server via SSH
  → Your machine has limited RAM (< 8GB)
  → You prefer a minimal, customizable editor over a full IDE
  → You are working with Dev Containers or GitHub Codespaces
  → You need to edit configuration files (YAML, JSON, Terraform) alongside Java

Use both:
  → Many senior engineers use IntelliJ for Java development and VS Code
    for everything else (scripts, configs, documentation, polyglot projects)
  → This is a perfectly valid and common workflow
```

---

## Important Notes

### IntelliJ Configuration Best Practices

```
1. Sync code style to the project:
   → Settings → Editor → Code Style → Java → Set from... → Google Java Style
   → Export to .idea/codeStyles/ and commit to Git
   → Ensures all team members format code identically

2. Enable auto-import:
   → Settings → Editor → General → Auto Import
   → "Add unambiguous imports on the fly" → ON
   → "Optimize imports on the fly" → ON
   → Eliminates manual import management

3. Configure inspection severity:
   → Settings → Editor → Inspections
   → Adjust severity levels for your team's preferences
   → Export profile to .idea/inspectionProfiles/ and commit

4. Set up file templates:
   → Settings → Editor → File and Code Templates
   → Customize the default class template to include:
      - SLF4J logger field
      - @Slf4j annotation (if using Lombok)
      - Javadoc header with author and date

5. Enable annotation processing:
   → Settings → Build → Compiler → Annotation Processors
   → "Enable annotation processing" → ON
   → Required for MapStruct, Lombok, JPA Metamodel Generator

6. Configure Maven/Gradle auto-import:
   → Settings → Build → Build Tools → Maven/Gradle
   → "Reload project after changes in the build scripts" → Any changes
   → Ensures dependencies update automatically when pom.xml changes

7. Set up external tools:
   → Settings → Tools → External Tools
   → Add: Spotless Apply (command: ./mvnw spotless:apply)
   → Add: Open in Terminal (command: open -a Terminal $FileDir$)
   → Bind to keyboard shortcuts for quick access
```

### Common IntelliJ Pitfalls

```
1. "My code compiles in IntelliJ but not in Maven/Gradle"
   → IntelliJ's internal compiler may be more lenient than javac
   → Always verify with mvn clean compile or ./gradlew build
   → Check that IntelliJ's SDK matches your pom.xml java.version

2. "IntelliJ is not picking up my dependency changes"
   → Right-click pom.xml → Maven → Reload Project
   → Or click the Maven refresh icon in the Maven tool window
   → For Gradle: click the Gradle sync elephant icon

3. "My breakpoints are not being hit"
   → Ensure you are running in Debug mode (bug icon), not Run mode
   → Check that the source code matches the compiled classes (rebuild)
   → Verify the breakpoint is on an executable line (not a declaration)
   → Check for conditional breakpoints that never evaluate to true

4. "IntelliJ is using too much memory"
   → Increase -Xmx in Help → Edit Custom VM Options
   → Exclude large directories from indexing
   → Disable unused plugins
   → Close projects you are not actively working on
   → Invalidate Caches and Restart (File → Invalidate Caches)

5. "Code completion is slow or missing"
   → Wait for indexing to complete (progress bar at bottom)
   → Check that the correct SDK is configured
   → Invalidate Caches if the index is corrupted
   → Ensure annotation processing is enabled

6. "My run configuration disappeared"
   → Check .idea/runConfigurations/ — is it still there?
   → If using temporary configurations, they are deleted after a limit
   → Save important configurations: Run → Edit Configurations → check "Store as project file"
```

### Productivity Multipliers

```
1. Learn one new shortcut per day:
   → Install Key Promoter X to shame you into using shortcuts
   → After 30 days, your navigation speed will double

2. Use Structural Search and Replace (Edit → Find → Search Structurally):
   → Search by code structure, not text
   → Example: find all try-catch blocks that swallow exceptions
   → Pattern: try { $stmt$; } catch ($ExceptionType$ $e$) { }
   → Far more powerful than regex for code searches

3. Use Scratch Files (Cmd + Shift + N → Scratch File):
   → Temporary files for experimenting with code snippets
   → Can be Java, SQL, JSON, HTTP, etc.
   → Not part of the project — won't be compiled or committed
   → Perfect for testing regex, JSON parsing, SQL queries

4. Use the Services Tool Window (Cmd + 8):
   → Unified view of all running services, databases, and Docker containers
   → Spring Boot endpoints, run configurations, database connections
   → The "mission control" for backend development

5. Use Bookmarks (F3 / Cmd + F3):
   → Mark important locations in your codebase
   → Navigate between them instantly
   → Mnemonic bookmarks: assign a number (Cmd + 1-9) for instant jump

6. Use the Terminal inside IntelliJ (Option + F12):
   → Integrated terminal shares the project's environment
   → No need to switch between IDE and terminal app
   → Supports multiple tabs, split panes
```

---

## Practice

```
1. Install IntelliJ IDEA Community Edition and configure it with JDK 21 via SDKMAN
2. Create a new Maven project and explore the project structure (modules, SDK, facets)
3. Practice the 10 most important navigation shortcuts until they are muscle memory:
   Cmd+O, Cmd+Shift+O, Cmd+B, Cmd+Shift+B, Cmd+E, Cmd+Shift+A, Double Shift,
   Cmd+F12, Cmd+Shift+F, Cmd+G
4. Practice the 5 most important refactoring shortcuts:
   Shift+F6, Cmd+Option+M, Cmd+Option+V, Cmd+Option+F, Cmd+Option+N
5. Write a Java class with intentional bugs and use the debugger to find them:
   → Set a conditional breakpoint
   → Use Evaluate Expression to inspect variables
   → Use Step Into, Step Over, and Step Out
6. Configure Checkstyle and SonarLint plugins and observe the real-time feedback
7. Create 3 custom Live Templates for patterns you type frequently
8. Set up a Spring Boot Run Configuration and debug a REST endpoint
9. (Ultimate) Connect to a PostgreSQL database and write a query using the SQL console
10. (Ultimate) Create an .http file and test your REST API endpoints
11. Configure IntelliJ's code style to match Google Java Style and export to .idea/codeStyles/
12. Increase IntelliJ's heap size to 4GB and observe the performance difference
13. Install VS Code with the Java Extension Pack and compare the experience with IntelliJ
    for a simple Spring Boot project
14. Use Structural Search to find all methods in your project that throw RuntimeException
15. Set up a complete IntelliJ workspace for the Phase 01 Banking Calculator project:
    run configuration, test configuration, debugger breakpoints, database connection
```

---

## References

- IntelliJ IDEA Documentation: https://www.jetbrains.com/help/idea/
- IntelliJ Keyboard Shortcuts (PDF): https://resources.jetbrains.com/storage/products/intellij-idea/docs/IntelliJIDEA_ReferenceCard.pdf
- IntelliJ Spring Boot Support: https://www.jetbrains.com/help/idea/spring-boot.html
- IntelliJ Database Tools: https://www.jetbrains.com/help/idea/database-tool-window.html
- IntelliJ HTTP Client: https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html
- VS Code Java Documentation: https://code.visualstudio.com/docs/java/java-tutorial
- VS Code Spring Boot Extension: https://marketplace.visualstudio.com/items?itemName=vmware.vscode-boot-dev-pack
- IntelliJ Performance Tuning: https://www.jetbrains.com/help/idea/tuning-the-ide.html
- "Mastering IntelliJ IDEA" — Packt Publishing
- Key Promoter X Plugin: https://plugins.jetbrains.com/plugin/9792-key-promoter-x
