## Overview

A build tool is the **engine that transforms your source code into a running application**. It compiles Java files into bytecode, resolves and downloads dependencies from remote repositories, runs your test suites, packages everything into deployable artifacts (JARs, WARs), and orchestrates the entire pipeline from `src/main/java` to production deployment.

In the Java ecosystem, there are two dominant build tools: **Maven** and **Gradle**. Unlike Python's relatively simple `pip`/`poetry` dependency management, Java build tools are heavyweight systems that manage compilation, testing, packaging, dependency resolution, plugin execution, and multi-module project orchestration. You cannot build a professional Java project without one.

**Maven** is the older, more established tool. It dominates enterprise Java and fintech because of its convention-over-configuration philosophy, predictable behavior, and massive plugin ecosystem. If you walk into a bank's engineering team, their projects are almost certainly Maven-based.

**Gradle** is the newer, more flexible alternative. It uses a real programming language (Groovy or Kotlin) for its build scripts instead of XML, supports incremental builds that are dramatically faster for large projects, and is the default choice for Android development and many modern Spring Boot projects. Spring Initializr now defaults to Gradle with Kotlin DSL.

The roadmap's recommendation: **learn Maven first** (it is the enterprise standard and easier to understand initially), **then learn Gradle** (you will encounter it in modern projects and need to be fluent in both). This note covers both in depth.

---

## Core Concepts

### What a Build Tool Actually Does

```
Source Code (.java files)
    │
    ▼
┌─────────────────────────────────────────────────┐
│                  BUILD TOOL                      │
│                                                   │
│  1. RESOLVE DEPENDENCIES                         │
│     → Read pom.xml / build.gradle                │
│     → Download JARs from Maven Central / repos    │
│     → Resolve version conflicts (transitive deps) │
│     → Cache locally (~/.m2/repository)            │
│                                                   │
│  2. COMPILE                                       │
│     → Invoke javac on src/main/java               │
│     → Output .class files to target/classes        │
│     → Process annotations (MapStruct, Lombok)      │
│                                                   │
│  3. TEST                                          │
│     → Compile src/test/java                       │
│     → Run JUnit 5 tests via Surefire/Fork         │
│     → Generate test reports                       │
│     → Check coverage thresholds (JaCoCo)          │
│                                                   │
│  4. PACKAGE                                       │
│     → Bundle .class files + resources into JAR     │
│     → Create fat/uber JAR with all dependencies   │
│     → Generate source and javadoc JARs            │
│                                                   │
│  5. VERIFY                                        │
│     → Run integration tests (Failsafe)            │
│     → Run static analysis (Checkstyle, SpotBugs)  │
│     → Enforce quality gates                       │
│                                                   │
│  6. INSTALL / DEPLOY                              │
│     → Install to local repository (~/.m2)          │
│     → Deploy to remote repository (Artifactory)   │
└─────────────────────────────────────────────────┘
    │
    ▼
Deployable Artifact (.jar, Docker image)
```

### Maven Architecture

```
Maven is built on three pillars:

1. Project Object Model (POM):
   → pom.xml declares everything about your project
   → Dependencies, plugins, build configuration, metadata
   → Declarative: you describe WHAT you want, not HOW to do it

2. Lifecycle:
   → A fixed sequence of phases that every build follows
   → validate → compile → test → package → verify → install → deploy
   → You cannot change the order of phases (convention over configuration)
   → Plugins bind their goals to specific lifecycle phases

3. Plugin System:
   → Maven itself does very little — plugins do the actual work
   → maven-compiler-plugin compiles Java
   → maven-surefire-plugin runs unit tests
   → maven-failsafe-plugin runs integration tests
   → spring-boot-maven-plugin creates executable JARs
   → Hundreds of community plugins available
```

### Gradle Architecture

```
Gradle is built on three pillars:

1. Build Scripts (DSL):
   → build.gradle (Groovy) or build.gradle.kts (Kotlin)
   → Imperative: you write actual code to configure the build
   → Full programming language: loops, conditionals, functions

2. Task Graph:
   → Builds are composed of tasks with dependencies
   → Gradle constructs a DAG (directed acyclic graph) of tasks
   → Only executes tasks whose inputs have changed (incremental builds)
   → Parallel task execution across modules

3. Plugin System:
   → Plugins add tasks, configurations, and conventions
   → java plugin, application plugin, spring-boot plugin
   → More flexible than Maven plugins (can modify the task graph)
```

### Dependency Resolution

Both tools solve the same fundamental problem: your project depends on libraries, those libraries depend on other libraries, and somewhere in that tree there are version conflicts.

```
Your Project
  ├── spring-boot-starter-web 3.2.3
  │     ├── spring-web 6.1.4
  │     │     └── spring-beans 6.1.4
  │     ├── spring-webmvc 6.1.4
  │     └── jackson-databind 2.15.4
  │           ├── jackson-core 2.15.4
  │           └── jackson-annotations 2.15.4
  ├── spring-boot-starter-data-jpa 3.2.3
  │     ├── hibernate-core 6.4.4.Final
  │     └── spring-data-jpa 3.2.3
  └── postgresql 42.7.2

Transitive dependencies:
  → You declare spring-boot-starter-web
  → Maven/Gradle automatically downloads spring-web, jackson, etc.
  → You do NOT need to declare every transitive dependency manually

Version conflicts:
  → Library A needs jackson 2.15.4
  → Library B needs jackson 2.14.2
  → The build tool must pick one version (nearest-wins, or forced)
  → This is where most build headaches come from
```

### The Local Repository Cache

```
Maven: ~/.m2/repository/
  → All downloaded JARs are cached here
  → Organized by groupId/artifactId/version
  → Example: ~/.m2/repository/org/springframework/spring-core/6.1.4/spring-core-6.1.4.jar
  → Shared across all Maven projects on your machine
  → Can grow to several GB over time
  → Safe to delete: rm -rf ~/.m2/repository (Maven re-downloads as needed)

Gradle: ~/.gradle/caches/
  → Similar concept, different directory structure
  → Also caches compiled build scripts and task outputs
  → Can be cleaned: gradle cleanBuildCache
```

---

## Code Examples

### Maven — Project Structure

Maven enforces a **standard directory layout**. Deviating from this convention is possible but strongly discouraged:

```
banking-api/
├── pom.xml                          ← the POM (project configuration)
├── mvnw                             ← Maven wrapper script (Unix)
├── mvnw.cmd                         ← Maven wrapper script (Windows)
├── .mvn/
│   └── wrapper/
│       └── maven-wrapper.properties ← wrapper version config
├── src/
│   ├── main/
│   │   ├── java/                    ← application source code
│   │   │   └── com/example/banking/
│   │   │       ├── BankingApplication.java
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── model/
│   │   │       ├── dto/
│   │   │       ├── exception/
│   │   │       └── config/
│   │   └── resources/               ← configuration files, templates
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       ├── logback-spring.xml
│   │       └── db/migration/        ← Flyway migrations
│   │           ├── V1__create_accounts.sql
│   │           └── V2__create_transactions.sql
│   └── test/
│       ├── java/                    ← test source code
│       │   └── com/example/banking/
│       │       ├── controller/
│       │       ├── service/
│       │       └── repository/
│       └── resources/               ← test configuration
│           └── application-test.yml
└── target/                          ← build output (git-ignored!)
    ├── classes/                     ← compiled .class files
    ├── test-classes/                ← compiled test .class files
    ├── surefire-reports/            ← test results
    └── banking-api-0.0.1-SNAPSHOT.jar  ← packaged artifact
```

### Maven — The POM (pom.xml)

The POM is the heart of every Maven project. Here is a complete, annotated example for a Spring Boot banking API:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- PARENT: inherit Spring Boot defaults -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <!-- PROJECT COORDINATES (GAV): uniquely identify this artifact -->
    <groupId>com.example</groupId>
    <artifactId>banking-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>banking-api</name>
    <description>Banking REST API for transaction processing</description>
    <packaging>jar</packaging> <!-- jar (default), war, pom (for parent modules) -->

    <!-- PROPERTIES: centralize version numbers and configuration -->
    <properties>
        <java.version>21</java.version>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
        <jjwt.version>0.12.5</jjwt.version>
        <testcontainers.version>1.19.7</testcontainers.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- DEPENDENCY MANAGEMENT: declare versions without importing -->
    <dependencyManagement>
        <dependencies>
            <!-- BOM (Bill of Materials): import a set of compatible versions -->
            <dependency>
                <groupId>org.testcontainers</groupId>
                <artifactId>testcontainers-bom</artifactId>
                <version>${testcontainers.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- DEPENDENCIES: what this project needs -->
    <dependencies>
        <!-- Spring Boot Starters (bundles of related dependencies) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- Database -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope> <!-- only needed at runtime, not compile time -->
        </dependency>
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>

        <!-- Utilities -->
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>

        <!-- API Documentation -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.3.0</version>
        </dependency>

        <!-- Development tools (not included in production JAR) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
            <!-- version inherited from BOM -->
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- BUILD CONFIGURATION -->
    <build>
        <plugins>
            <!-- Spring Boot Maven Plugin: creates executable fat JAR -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>

            <!-- Compiler Plugin: configure Java version and annotation processors -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${mapstruct.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>

            <!-- Surefire Plugin: runs unit tests during the test phase -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <includes>
                        <include>**/*Test.java</include>
                        <include>**/*Tests.java</include>
                    </includes>
                    <excludes>
                        <exclude>**/*IntegrationTest.java</exclude>
                    </excludes>
                </configuration>
            </plugin>

            <!-- Failsafe Plugin: runs integration tests during the verify phase -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <includes>
                        <include>**/*IntegrationTest.java</include>
                    </includes>
                </configuration>
            </plugin>

            <!-- JaCoCo Plugin: code coverage -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.12</version>
                <executions>
                    <execution>
                        <goals><goal>prepare-agent</goal></goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>test</phase>
                        <goals><goal>report</goal></goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <!-- PROFILES: environment-specific configurations -->
    <profiles>
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <spring.profiles.active>dev</spring.profiles.active>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <spring.profiles.active>prod</spring.profiles.active>
            </properties>
            <build>
                <plugins>
                    <!-- Additional production-only plugins -->
                </plugins>
            </build>
        </profile>
    </profiles>

</project>
```

### Maven — Dependency Scopes

Scopes control **when** a dependency is available (compile time, runtime, test time):

```
compile (default):
  → Available at compile time, runtime, and in the final package
  → Example: spring-boot-starter-web, mapstruct
  → Transitive: YES (consumers of your library also get this)

provided:
  → Available at compile time but NOT included in the final package
  → The runtime environment is expected to provide it
  → Example: jakarta.servlet-api (provided by Tomcat at runtime)
  → Transitive: NO

runtime:
  → NOT available at compile time, but included at runtime and in the package
  → Example: postgresql JDBC driver (you code against JDBC interfaces,
    the driver is loaded at runtime via reflection)
  → Transitive: YES

test:
  → Only available during test compilation and execution
  → NOT included in the final package
  → Example: junit-jupiter, mockito, testcontainers
  → Transitive: NO

system:
  → Like provided, but you specify the JAR path manually
  → AVOID: non-portable, deprecated in practice
  → Use a local repository or install:install-file instead

import (BOM only):
  → Only used in <dependencyManagement> with <type>pom</type>
  → Imports version declarations from another POM
  → Does not add actual dependencies
```

### Maven — The Build Lifecycle

Maven has three built-in lifecycles. The **default lifecycle** is the one you use 99% of the time:

```
DEFAULT LIFECYCLE (in order):
  validate      → verify the project is correct and all necessary info is available
  compile       → compile src/main/java → target/classes
  test          → compile src/test/java and run unit tests (Surefire)
  package       → package compiled code into JAR/WAR → target/*.jar
  verify        → run integration tests and quality checks (Failsafe, Checkstyle)
  install       → install the package to ~/.m2/repository (available to other local projects)
  deploy        → upload the package to a remote repository (Artifactory, Nexus)

CLEAN LIFECYCLE:
  pre-clean     → execute processes needed prior to clean
  clean         → remove target/ directory (all build artifacts)
  post-clean    → execute processes needed to finalize clean

SITE LIFECYCLE:
  pre-site      → execute processes needed prior to site generation
  site          → generate project documentation
  post-site     → execute processes needed to finalize site
  site-deploy   → deploy site documentation to a web server

KEY CONCEPT: phases are cumulative.
  "mvn package" runs: validate → compile → test → package
  "mvn install" runs: validate → compile → test → package → verify → install
  You never need to run "mvn compile test package" — just "mvn package" does all three.
```

### Maven — Common Commands

```bash
# --- Clean and Build ---
mvn clean                    # delete target/ directory
mvn compile                  # compile main sources
mvn test                     # compile + run unit tests
mvn package                  # compile + test + create JAR
mvn verify                   # compile + test + package + integration tests
mvn install                  # compile + test + package + verify + install to ~/.m2
mvn clean install            # clean + full build (the most common command)
mvn clean install -DskipTests  # build without running tests (fast, use sparingly)

# --- Run the Application ---
mvn spring-boot:run          # run Spring Boot app directly (dev mode)
mvn spring-boot:run -Dspring-boot.run.profiles=dev  # with specific profile

# --- Dependency Management ---
mvn dependency:tree          # show full dependency tree
mvn dependency:tree -Dincludes=com.fasterxml.jackson  # filter tree
mvn dependency:analyze       # find unused and undeclared dependencies
mvn dependency:resolve       # download all dependencies
mvn dependency:purge-local-repository  # re-download all dependencies

# --- Project Information ---
mvn help:effective-pom       # show the fully resolved POM (with inherited config)
mvn help:active-profiles     # show which profiles are active
mvn help:describe -Dplugin=spring-boot  # describe a plugin

# --- Testing ---
mvn test -Dtest=AccountServiceTest         # run a specific test class
mvn test -Dtest=AccountServiceTest#testWithdraw  # run a specific test method
mvn test -Dtest="*Transfer*"               # run tests matching a pattern
mvn failsafe:integration-test              # run only integration tests

# --- Release ---
mvn versions:set -DnewVersion=1.0.0        # update project version
mvn deploy                                 # deploy to remote repository

# --- Performance ---
mvn clean install -T 1C                    # parallel build (1 thread per CPU core)
mvn clean install -o                       # offline mode (no network, use cache)
mvn clean install -pl banking-api-core     # build specific module only
mvn clean install -pl banking-api-core -am # build module and its dependencies
```

### Maven — Multi-Module Projects

Large Java projects (especially microservices) are split into **modules** that share a parent POM:

```
banking-platform/                    ← parent project (packaging: pom)
├── pom.xml                          ← parent POM
├── banking-api-core/                ← shared domain models, DTOs, utils
│   ├── pom.xml
│   └── src/
├── banking-api-web/                 ← REST controllers, Spring Boot app
│   ├── pom.xml
│   └── src/
├── banking-api-persistence/         ← JPA entities, repositories
│   ├── pom.xml
│   └── src/
├── banking-api-security/            ← JWT, Spring Security config
│   ├── pom.xml
│   └── src/
└── banking-api-integration-tests/   ← integration test module
    ├── pom.xml
    └── src/
```

**Parent POM (`banking-platform/pom.xml`):**

```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>banking-platform</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging> <!-- THIS IS A PARENT, NOT A JAR -->

    <modules>
        <module>banking-api-core</module>
        <module>banking-api-persistence</module>
        <module>banking-api-security</module>
        <module>banking-api-web</module>
        <module>banking-api-integration-tests</module>
    </modules>

    <properties>
        <java.version>21</java.version>
        <spring-boot.version>3.2.3</spring-boot.version>
    </properties>

    <!-- Shared dependency versions for ALL modules -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- Internal module versions -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>banking-api-core</artifactId>
                <version>${project.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- Shared dependencies for ALL modules -->
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

**Child module POM (`banking-api-web/pom.xml`):**

```xml
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>banking-platform</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>banking-api-web</artifactId>
    <!-- version inherited from parent -->
    <!-- groupId inherited from parent -->

    <dependencies>
        <!-- Internal module dependency (version from parent's dependencyManagement) -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>banking-api-core</artifactId>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>banking-api-persistence</artifactId>
        </dependency>

        <!-- External dependencies (versions from Spring Boot BOM) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

### Maven — Dependency Conflict Resolution

```bash
# View the dependency tree to find conflicts
mvn dependency:tree

# Example output showing a conflict:
# [INFO] com.example:banking-api-web:jar:1.0.0-SNAPSHOT
# [INFO] +- org.springframework.boot:spring-boot-starter-web:jar:3.2.3
# [INFO] |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.15.4
# [INFO] +- com.example:banking-api-core:jar:1.0.0-SNAPSHOT
# [INFO] |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.14.2  ← CONFLICT

# Maven's resolution strategy: "nearest definition wins"
# The dependency closest to your project in the tree takes precedence.
# In this case, 2.15.4 wins because it's 2 levels deep vs 3 levels deep.

# Force a specific version by declaring it directly in your POM:
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.4</version> <!-- overrides all transitive versions -->
</dependency>

# Or exclude the conflicting transitive dependency:
<dependency>
    <groupId>com.example</groupId>
    <artifactId>banking-api-core</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### Maven — BOM (Bill of Materials)

A BOM is a special POM that declares **compatible versions** of a set of related libraries. You import it in `<dependencyManagement>` and then omit versions from your actual dependencies:

```xml
<dependencyManagement>
    <dependencies>
        <!-- Spring Boot BOM: defines compatible versions for 200+ libraries -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.2.3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- Spring Cloud BOM: compatible versions for Spring Cloud projects -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2023.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- No version needed — inherited from the Spring Boot BOM -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
    </dependency>
</dependencies>
```

### Maven Wrapper (mvnw)

The Maven Wrapper ensures everyone on the team uses the **exact same Maven version** without requiring a global Maven installation:

```bash
# Generate the wrapper (requires Maven installed once)
mvn wrapper:wrapper -Dmaven=3.9.6

# This creates:
# mvnw          → Unix shell script
# mvnw.cmd      → Windows batch script
# .mvn/wrapper/maven-wrapper.properties  → specifies Maven version

# Use the wrapper instead of mvn:
./mvnw clean install      # macOS/Linux
mvnw.cmd clean install    # Windows

# The wrapper automatically downloads the specified Maven version
# to ~/.m2/wrapper/ on first use.

# ALWAYS commit the wrapper files to Git:
git add mvnw mvnw.cmd .mvn/
git commit -m "build: add Maven wrapper"

# This is the Java equivalent of Python's pip/poetry lock files —
# it ensures reproducible builds across all environments.
```

---

### Gradle — Project Structure

```
banking-api/
├── build.gradle.kts                 ← Kotlin DSL build script (or build.gradle for Groovy)
├── settings.gradle.kts              ← project settings and module includes
├── gradle.properties                ← Gradle configuration properties
├── gradlew                          ← Gradle wrapper script (Unix)
├── gradlew.bat                      ← Gradle wrapper script (Windows)
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
└── build/                           ← build output (equivalent of Maven's target/)
    ├── classes/
    ├── libs/
    └── reports/
```

### Gradle — Kotlin DSL (`build.gradle.kts`)

Kotlin DSL is the modern, recommended Gradle syntax. It provides type safety, IDE autocompletion, and compile-time error checking:

```kotlin
// build.gradle.kts

plugins {
    java
    id("org.springframework.boot") version "3.2.3"
    id("io.spring.dependency-management") version "1.1.4"
    id("org.flywaydb.flyway") version "10.8.1"
    jacoco
}

group = "com.example"
version = "0.0.1-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

// Dependency versions as variables
val mapstructVersion = "1.5.5.Final"
val jjwtVersion = "0.12.5"
val testcontainersVersion = "1.19.7"

dependencies {
    // Spring Boot Starters
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("org.springframework.boot:spring-boot-starter-data-redis")

    // Database
    runtimeOnly("org.postgresql:postgresql")
    implementation("org.flywaydb:flyway-core")

    // Utilities
    implementation("org.mapstruct:mapstruct:$mapstructVersion")
    annotationProcessor("org.mapstruct:mapstruct-processor:$mapstructVersion")

    implementation("io.jsonwebtoken:jjwt-api:$jjwtVersion")
    runtimeOnly("io.jsonwebtoken:jjwt-impl:$jjwtVersion")
    runtimeOnly("io.jsonwebtoken:jjwt-jackson:$jjwtVersion")

    // API Documentation
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.3.0")

    // Development
    developmentOnly("org.springframework.boot:spring-boot-devtools")

    // Testing
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.springframework.security:spring-security-test")
    testImplementation("org.testcontainers:postgresql:$testcontainersVersion")
    testImplementation("org.testcontainers:junit-jupiter:$testcontainersVersion")
}

tasks.withType<Test> {
    useJUnitPlatform()
}

tasks.jacocoTestReport {
    reports {
        xml.required.set(true)
        html.required.set(true)
    }
}

tasks.jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                minimum = "0.80".toBigDecimal()
            }
        }
    }
}
```

### Gradle — Groovy DSL (`build.gradle`)

The older Groovy syntax. Still widely used, especially in legacy projects:

```groovy
// build.gradle

plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.3'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '21'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'org.postgresql:postgresql'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

### Gradle — Dependency Configurations

Gradle's dependency configurations are more granular than Maven's scopes:

```
implementation:
  → Available at compile time and runtime
  → NOT exposed to consumers of your library (unlike Maven's compile)
  → Use for internal dependencies
  → Equivalent to Maven compile (for applications)

api:
  → Available at compile time and runtime
  → IS exposed to consumers (transitive)
  → Use in library modules for dependencies that are part of your public API
  → Requires the java-library plugin

compileOnly:
  → Available at compile time only
  → NOT included in the runtime classpath or final package
  → Equivalent to Maven's provided
  → Example: Lombok, annotations that are processed at compile time

runtimeOnly:
  → NOT available at compile time
  → Included at runtime and in the final package
  → Equivalent to Maven's runtime
  → Example: JDBC drivers, logging implementations

testImplementation:
  → Available only during test compilation and execution
  → Equivalent to Maven's test
  → Example: JUnit, Mockito, Testcontainers

testCompileOnly:
  → Compile-time only for tests

testRuntimeOnly:
  → Runtime only for tests
  → Example: JUnit Platform launcher

annotationProcessor:
  → Dependencies used by the Java compiler's annotation processing
  → Example: MapStruct processor, Lombok, Hibernate metamodel generator
  → Not included in the final artifact

developmentOnly:
  → Available during development (spring-boot:run) but excluded from the JAR
  → Example: spring-boot-devtools
```

### Gradle — Settings File

```kotlin
// settings.gradle.kts

rootProject.name = "banking-platform"

// Multi-project setup
include("banking-api-core")
include("banking-api-persistence")
include("banking-api-security")
include("banking-api-web")
include("banking-api-integration-tests")

// Plugin management (centralize plugin versions)
pluginManagement {
    repositories {
        gradlePluginPortal()
        mavenCentral()
    }
}

// Dependency resolution management (centralize repositories)
dependencyResolutionManagement {
    repositories {
        mavenCentral()
    }
}
```

### Gradle — Common Commands

```bash
# --- Build ---
./gradlew clean                  # delete build/ directory
./gradlew build                  # compile + test + package (equivalent of mvn package)
./gradlew assemble               # compile + package (skip tests)
./gradlew check                  # run all verification tasks (tests, checkstyle)
./gradlew clean build            # clean + full build

# --- Run ---
./gradlew bootRun                # run Spring Boot application
./gradlew bootRun --args='--spring.profiles.active=dev'

# --- Testing ---
./gradlew test                   # run unit tests
./gradlew test --tests "com.example.AccountServiceTest"  # specific class
./gradlew test --tests "*Transfer*"                      # pattern match
./gradlew test --info            # verbose test output

# --- Dependencies ---
./gradlew dependencies           # show all dependency configurations
./gradlew dependencies --configuration runtimeClasspath  # specific configuration
./gradlew dependencyInsight --dependency jackson-databind  # why is this dep here?

# --- Project Info ---
./gradlew projects               # list all subprojects
./gradlew tasks                  # list all available tasks
./gradlew tasks --all            # list ALL tasks (including hidden)
./gradlew properties             # show all project properties

# --- Performance ---
./gradlew build --parallel       # parallel module builds
./gradlew build --build-cache    # use build cache (default in recent versions)
./gradlew build --dry-run        # show what would run without executing
./gradlew build --scan           # generate a build scan (performance analysis)
```

### Gradle — Tasks and Task Graph

```kotlin
// Gradle builds are composed of tasks. Each task has inputs, outputs, and dependencies.
// Gradle only runs tasks whose inputs have changed (incremental builds).

// View the task graph:
// ./gradlew build --dry-run
// :banking-api-core:compileJava SKIPPED
// :banking-api-core:processResources SKIPPED
// :banking-api-core:classes SKIPPED
// :banking-api-core:jar SKIPPED
// :banking-api-persistence:compileJava SKIPPED
// ...
// :banking-api-web:bootJar SKIPPED
// :banking-api-web:build SKIPPED

// Custom tasks:
tasks.register("printVersion") {
    doLast {
        println("Project version: ${project.version}")
    }
}

tasks.register("generateBuildInfo") {
    val outputFile = layout.buildDirectory.file("build-info.properties")
    outputs.file(outputFile)
    doLast {
        outputFile.get().asFile.writeText("""
            build.version=${project.version}
            build.timestamp=${System.currentTimeMillis()}
            build.java=${System.getProperty("java.version")}
        """.trimIndent())
    }
}

// Task dependencies:
tasks.named("build") {
    dependsOn("generateBuildInfo")
}
```

### Gradle — Multi-Project Builds

```kotlin
// Root build.gradle.kts
plugins {
    java
    id("org.springframework.boot") version "3.2.3" apply false
    id("io.spring.dependency-management") version "1.1.4" apply false
}

subprojects {
    apply(plugin = "java")

    group = "com.example"
    version = "1.0.0-SNAPSHOT"

    java {
        sourceCompatibility = JavaVersion.VERSION_21
    }

    repositories {
        mavenCentral()
    }

    dependencies {
        testImplementation("org.junit.jupiter:junit-jupiter:5.10.2")
    }

    tasks.withType<Test> {
        useJUnitPlatform()
    }
}

// banking-api-web/build.gradle.kts
plugins {
    id("org.springframework.boot")
    id("io.spring.dependency-management")
}

dependencies {
    implementation(project(":banking-api-core"))
    implementation(project(":banking-api-persistence"))
    implementation("org.springframework.boot:spring-boot-starter-web")
}
```

### Gradle — Build Cache

```kotlin
// gradle.properties
org.gradle.caching=true
org.gradle.parallel=true
org.gradle.jvmargs=-Xmx2g -XX:+HeapDumpOnOutOfMemoryError
org.gradle.configuration-cache=true  // Gradle 7+ (experimental → stable in 8.x)

// The build cache stores task outputs. If inputs haven't changed,
// Gradle reuses cached outputs instead of re-executing the task.
// This can reduce build times by 50-90% for large projects.

// Local cache: ~/.gradle/caches/build-cache-1/
// Remote cache: Gradle Enterprise or custom HTTP cache (for CI/CD sharing)
```

### Gradle Wrapper

```bash
# Generate the wrapper (requires Gradle installed once)
gradle wrapper --gradle-version 8.6

# This creates:
# gradlew, gradlew.bat, gradle/wrapper/*

# Use the wrapper:
./gradlew build

# Upgrade Gradle version:
./gradlew wrapper --gradle-version 8.7

# ALWAYS commit the wrapper to Git (same rationale as Maven wrapper)
```

---

## Important Notes

### Maven vs Gradle — Comparison

```
                    Maven                          Gradle
─────────────────────────────────────────────────────────────────────
Language            XML (pom.xml)                  Groovy or Kotlin DSL
Philosophy          Convention over configuration  Flexible, programmatic
Learning curve      Lower (rigid but predictable)  Higher (powerful but complex)
Build speed         Slower (rebuilds everything)   Faster (incremental, caching)
Dependency model    Scopes (compile, test, etc.)   Configurations (implementation, api, etc.)
Multi-module        <modules> in parent POM        include() in settings.gradle
Plugin system       XML configuration              DSL blocks with full code
Flexibility         Low (hard to customize)        High (can do anything)
Enterprise usage    Dominant in banking/fintech    Growing, dominant in Android
Spring Boot         Fully supported                Fully supported (default in Initializr)
IDE support         Excellent (IntelliJ, Eclipse)  Excellent (IntelliJ, VS Code)
Reproducibility     Good (pom.lock not standard)   Better (dependency locking built-in)
CI/CD               Ubiquitous                     Ubiquitous
Community           Massive, mature                Large, active
```

### When to Use Which

```
Choose Maven when:
  → You are working in enterprise/fintech (most existing projects use Maven)
  → Your team is large and values predictability over flexibility
  → You want the simplest possible build configuration
  → You are new to Java and want to focus on learning the language first
  → Your project follows standard conventions (Spring Boot, standard layout)
  → You need maximum compatibility with CI/CD tools and enterprise infrastructure

Choose Gradle when:
  → You are starting a new project and want faster builds
  → You have a complex, multi-module project with custom build logic
  → You need fine-grained control over the build process
  → You are building Android applications (Gradle is mandatory)
  → Your team is experienced with Java and wants build performance
  → You want Kotlin DSL for type-safe build scripts
  → You are using Spring Initializr's default (Gradle + Kotlin DSL)

In practice:
  → Learn both. You WILL encounter both in your career.
  → Start with Maven (this roadmap's recommendation).
  → Add Gradle when you reach Phase 05 (Spring Boot projects).
  → In fintech interviews, Maven knowledge is expected; Gradle is a bonus.
```

### Version Numbering Conventions

```
Semantic Versioning (SemVer): MAJOR.MINOR.PATCH
  → 1.0.0 → 1.0.1 (bug fix) → 1.1.0 (new feature) → 2.0.0 (breaking change)

Maven SNAPSHOT versions:
  → 1.0.0-SNAPSHOT = development version (not released)
  → Maven treats SNAPSHOTs specially: always checks for newer versions
  → Release versions (1.0.0) are immutable: once published, never changed
  → SNAPSHOT versions are mutable: can be overwritten

Spring Boot version alignment:
  → Spring Boot 3.2.x requires Java 17+
  → Spring Boot 3.2.x uses Spring Framework 6.1.x
  → Spring Boot 3.2.x uses Hibernate 6.4.x
  → The Spring Boot BOM ensures all these versions are compatible
  → NEVER mix Spring Boot versions with incompatible Spring Framework versions
```

### Common Build Problems and Solutions

```
1. "Could not resolve dependencies"
   → Check your internet connection
   → Verify the artifact exists on Maven Central: https://search.maven.org/
   → Check for typos in groupId, artifactId, or version
   → Clear local cache: rm -rf ~/.m2/repository/org/example/
   → Check if you need a private repository (company Artifactory)

2. "Compilation error: package does not exist"
   → Missing dependency in pom.xml / build.gradle
   → Wrong scope (e.g., test dependency used in main code)
   → Run mvn dependency:tree to verify the dependency is resolved

3. "Version conflict" / "NoSuchMethodError at runtime"
   → Two dependencies pull in different versions of the same library
   → Run mvn dependency:tree -Dverbose to find the conflict
   → Force the correct version or exclude the transitive dependency
   → In Gradle: ./gradlew dependencyInsight --dependency <name>

4. "Tests pass locally but fail in CI"
   → Different Java version (check CI JDK matches local)
   → Different OS (file paths, line endings)
   → Missing environment variables or configuration
   → Test ordering dependencies (tests that pass only in a specific order)
   → Timezone differences (use UTC in tests)

5. "Build is too slow"
   → Maven: use -T 1C for parallel builds
   → Maven: use -o for offline mode when dependencies haven't changed
   → Maven: skip tests during development: -DskipTests (re-enable before committing!)
   → Gradle: enable build cache and parallel builds in gradle.properties
   → Gradle: use --configuration-cache for faster configuration phase
   → Both: use the wrapper to avoid version resolution overhead

6. "Fat JAR is too large"
   → Check for unnecessary dependencies: mvn dependency:analyze
   → Exclude transitive dependencies you don't need
   → Use provided scope for libraries the runtime environment supplies
   → Spring Boot layered JARs for Docker optimization (Phase 07)
```

### Best Practices

```
1. ALWAYS use the Maven/Gradle wrapper (mvnw / gradlew)
   → Ensures reproducible builds across all machines and CI
   → Commit the wrapper files to version control

2. Centralize version numbers in properties/variables
   → Maven: <properties><jackson.version>2.15.4</jackson.version></properties>
   → Gradle: val jacksonVersion = "2.15.4"
   → Never hardcode versions in multiple places

3. Use BOMs for related dependency sets
   → Spring Boot BOM, Spring Cloud BOM, Testcontainers BOM
   → Reduces version conflicts and simplifies upgrades

4. Separate unit tests from integration tests
   → Maven: Surefire for unit (*Test.java), Failsafe for integration (*IntegrationTest.java)
   → Gradle: create separate test tasks or use tags
   → Unit tests run on every build; integration tests run on verify/CI

5. Never commit target/ or build/ directories
   → Add to .gitignore: target/, build/, .gradle/
   → These are generated artifacts, not source code

6. Pin plugin versions
   → Don't rely on "latest" — a plugin update can break your build
   → Maven: <version>3.12.1</version> on every plugin
   → Gradle: id("org.springframework.boot") version "3.2.3"

7. Use dependency locking (Gradle) or versions plugin (Maven)
   → Gradle: dependencyLocking { lockAllConfigurations() }
   → Maven: versions-maven-plugin to check for updates
   → Ensures reproducible builds even when transitive dependencies change

8. Run mvn dependency:analyze regularly
   → Finds declared-but-unused dependencies (remove them)
   → Finds used-but-undeclared dependencies (declare them explicitly)
   → Keeps your dependency tree clean and intentional
```

---

## Practice

```
1. Create a new Spring Boot project using Spring Initializr (start.spring.io)
   with Maven, Java 21, and starters: Web, JPA, PostgreSQL, Security, Validation
2. Examine the generated pom.xml and identify the parent, properties, dependencies,
   and build sections
3. Run mvn clean install and observe each lifecycle phase in the output
4. Run mvn dependency:tree and trace the transitive dependencies of spring-boot-starter-web
5. Add a dependency with a deliberate version conflict and resolve it using exclusions
6. Create a multi-module Maven project with a parent POM and two child modules
   (core and web), where web depends on core
7. Configure the Maven Surefire and Failsafe plugins to separate unit and integration tests
8. Generate and use the Maven wrapper (mvnw) for your project
9. Run mvn help:effective-pom and compare it to your raw pom.xml — identify what
   Spring Boot's parent POM adds automatically
10. Create the same Spring Boot project using Gradle with Kotlin DSL (build.gradle.kts)
11. Compare the Maven and Gradle build times for the same project (clean build + incremental)
12. Configure JaCoCo in both Maven and Gradle with an 80% coverage threshold
13. Add a custom Gradle task that prints the project version and build timestamp
14. Use ./gradlew dependencyInsight to find why a specific transitive dependency is included
15. Set up a Gradle multi-project build mirroring the Maven multi-module structure from #6
```

---

## References

- Maven Official Documentation: https://maven.apache.org/guides/
- Maven POM Reference: https://maven.apache.org/pom.html
- Maven Lifecycle Reference: https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
- Gradle User Manual: https://docs.gradle.org/current/userguide/userguide.html
- Gradle Kotlin DSL Primer: https://docs.gradle.org/current/userguide/kotlin_dsl.html
- Spring Boot Build Tool Plugins: https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins.html
- Maven Central Repository Search: https://search.maven.org/
- Gradle Build Scans: https://scans.gradle.com/
- "Maven: The Complete Reference" — Sonatype (free): https://books.sonatype.com/mvnref-book/reference/
