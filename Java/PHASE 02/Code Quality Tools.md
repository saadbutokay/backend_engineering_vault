## Overview

Code quality tools are the **automated guardrails** that enforce consistency, catch bugs before they reach production, and maintain the long-term health of a codebase. In a professional Java backend environment—especially in fintech, where a single bug can move real money—code quality is not a luxury. It is a **regulatory and operational requirement**.

Java has the most mature code quality ecosystem of any language. The tools fall into four categories:

1. **Code Style Enforcement** — ensuring every file follows the same formatting and naming conventions (Checkstyle, google-java-format)
2. **Static Analysis** — finding bugs, code smells, and anti-patterns without running the code (SpotBugs, PMD, Error Prone, SonarQube)
3. **Auto-Formatting** — automatically rewriting code to match a style standard (Spotless, google-java-format)
4. **Dependency Vulnerability Scanning** — detecting known security vulnerabilities in your third-party libraries (OWASP Dependency-Check, Snyk)

These tools integrate into your **build pipeline** (Maven/Gradle), your **IDE** (IntelliJ), your **Git workflow** (pre-commit hooks), and your **CI/CD system** (GitHub Actions). The goal is to make it **impossible to merge bad code**—not through willpower, but through automation.

This note covers every tool you need, with real configuration examples for both Maven and Gradle, and shows how to wire them together into a unified quality gate.

---

## Core Concepts

### The Code Quality Pyramid

```
              ┌─────────────┐
              │  Security    │  OWASP Dependency-Check, Snyk
              │  Scanning    │  (known CVEs in dependencies)
              ├─────────────┤
              │   Static     │  SpotBugs, PMD, Error Prone, SonarQube
              │   Analysis   │  (bugs, code smells, anti-patterns)
              ├─────────────┤
              │    Code      │  Checkstyle, Spotless, google-java-format
              │    Style     │  (formatting, naming, conventions)
              ├─────────────┤
              │   Editor     │  .editorconfig, IDE settings
              │   Config     │  (tabs vs spaces, line endings, encoding)
              └─────────────┘
```

Each layer catches different problems. A project with only style checks but no static analysis will have perfectly formatted code that still contains null pointer bugs. A project with static analysis but no dependency scanning might ship code with a Log4j-level vulnerability. **You need all layers.**

### Code Style vs Static Analysis vs Formatting

```
Code Style (Checkstyle):
  → Enforces RULES about how code should look and be structured
  → "Methods must not exceed 80 lines"
  → "Class names must be PascalCase"
  → "No wildcard imports (import java.util.*)"
  → Reports violations; does NOT fix them automatically
  → Configurable via XML ruleset

Static Analysis (SpotBugs, PMD, Error Prone):
  → Finds actual BUGS and code smells by analyzing the code's logic
  → "This method may return null but the caller does not check"
  → "This resource is opened but never closed (potential leak)"
  → "This comparison uses == on Strings instead of .equals()"
  → Analyzes bytecode (SpotBugs) or source code (PMD, Error Prone)
  → Reports issues; does NOT fix them automatically

Auto-Formatting (Spotless, google-java-format):
  → REWRITES your code to match a formatting standard
  → Fixes indentation, spacing, brace placement, import ordering
  → Runs automatically on save or during the build
  → No configuration debates: the formatter decides
  → Idempotent: running it twice produces the same result
```

### Where Quality Tools Run

```
1. IDE (real-time, as you type):
   → SonarLint plugin highlights issues inline
   → Checkstyle plugin shows violations in the gutter
   → Spotless/format-on-save fixes formatting instantly
   → Fastest feedback loop — catch issues before you even save

2. Pre-commit hooks (before each git commit):
   → Run Checkstyle, Spotless, and compilation checks
   → Prevent bad code from entering the repository
   → Fast subset of checks (don't run full integration tests here)

3. Build pipeline (mvn verify / gradle check):
   → All quality tools run as part of the build
   → Build FAILS if quality gates are not met
   → This is the authoritative gate — if it passes here, the code is acceptable

4. CI/CD (GitHub Actions, Jenkins):
   → Same checks as the build pipeline, but in a clean environment
   → PR checks: the PR cannot be merged until CI is green
   → SonarQube analysis with full project history and trend tracking

5. Nightly/Weekly scans:
   → Dependency vulnerability scanning (OWASP, Snyk)
   → Full SonarQube analysis with coverage trends
   → License compliance checks
```

---

## Code Examples

### .editorconfig

`.editorconfig` is a **cross-editor configuration file** that ensures consistent formatting regardless of whether a developer uses IntelliJ, VS Code, Vim, or Emacs. Place it at the root of your repository:

```ini
# .editorconfig
# https://editorconfig.org

root = true

# All files
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 4

# Java files
[*.java]
indent_size = 4
max_line_length = 120

# XML files (Maven POMs, Spring configs)
[*.xml]
indent_size = 4

# YAML files (Spring Boot config, CI pipelines)
[*.{yml,yaml}]
indent_size = 2

# Properties files
[*.properties]
indent_size = 4

# Markdown
[*.md]
trim_trailing_whitespace = false
max_line_length = off

# Shell scripts
[*.sh]
end_of_line = lf
indent_size = 2

# Gradle Kotlin DSL
[*.gradle.kts]
indent_size = 4

# Makefiles (must use tabs)
[Makefile]
indent_style = tab
```

IntelliJ supports `.editorconfig` natively. VS Code requires the "EditorConfig for VS Code" extension.

### Checkstyle

Checkstyle is the **industry standard** for enforcing Java coding conventions. It checks source code against a configurable set of rules covering naming, formatting, imports, Javadoc, class design, and more.

**Maven configuration (`pom.xml`):**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <version>3.3.1</version>
            <dependencies>
                <!-- Use the latest Checkstyle version -->
                <dependency>
                    <groupId>com.puppycrawl.tools</groupId>
                    <artifactId>checkstyle</artifactId>
                    <version>10.14.2</version>
                </dependency>
            </dependencies>
            <configuration>
                <configLocation>checkstyle.xml</configLocation>
                <consoleOutput>true</consoleOutput>
                <failsOnError>true</failsOnError>
                <failOnViolation>true</failOnViolation>
                <violationSeverity>warning</violationSeverity>
                <includeTestSourceDirectory>true</includeTestSourceDirectory>
                <linkXRef>false</linkXRef>
            </configuration>
            <executions>
                <execution>
                    <id>validate</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**Gradle configuration (`build.gradle.kts`):**

```kotlin
plugins {
    checkstyle
}

checkstyle {
    toolVersion = "10.14.2"
    configFile = file("${rootDir}/checkstyle.xml")
    isShowViolations = true
    maxWarnings = 0  // fail on any warning
}

tasks.checkstyleMain {
    source = fileTree("src/main/java")
}

tasks.checkstyleTest {
    source = fileTree("src/test/java")
}
```

**Checkstyle ruleset (`checkstyle.xml`):**

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">

<module name="Checker">
    <property name="charset" value="UTF-8"/>
    <property name="severity" value="warning"/>
    <property name="fileExtensions" value="java"/>

    <!-- File-level checks -->
    <module name="FileLength">
        <property name="max" value="500"/>
    </module>
    <module name="NewlineAtEndOfFile">
        <property name="lineSeparator" value="lf"/>
    </module>
    <module name="FileTabCharacter">
        <property name="eachLine" value="true"/>
    </module>

    <module name="TreeWalker">

        <!-- NAMING CONVENTIONS -->
        <module name="PackageName">
            <property name="format" value="^[a-z]+(\.[a-z][a-z0-9]*)*$"/>
        </module>
        <module name="TypeName">
            <property name="format" value="^[A-Z][a-zA-Z0-9]*$"/>
        </module>
        <module name="MethodName">
            <property name="format" value="^[a-z][a-zA-Z0-9_]*$"/>
        </module>
        <module name="MemberName">
            <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
        </module>
        <module name="LocalVariableName">
            <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
        </module>
        <module name="ConstantName">
            <property name="format" value="^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$"/>
        </module>
        <module name="ParameterName">
            <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
        </module>

        <!-- IMPORTS -->
        <module name="AvoidStarImport"/>
        <module name="IllegalImport"/>
        <module name="RedundantImport"/>
        <module name="UnusedImports"/>
        <module name="ImportOrder">
            <property name="groups" value="java,javax,org,com"/>
            <property name="ordered" value="true"/>
            <property name="separated" value="true"/>
            <property name="option" value="bottom"/>
            <property name="sortStaticImportsAlphabetically" value="true"/>
        </module>

        <!-- FORMATTING -->
        <module name="LineLength">
            <property name="max" value="120"/>
            <property name="ignorePattern" value="^package.*|^import.*|a]* href|href|http://|https://|ftp://"/>
        </module>
        <module name="MethodLength">
            <property name="max" value="50"/>
        </module>
        <module name="ParameterNumber">
            <property name="max" value="7"/>
        </module>
        <module name="Indentation">
            <property name="basicOffset" value="4"/>
            <property name="braceAdjustment" value="0"/>
            <property name="caseIndent" value="4"/>
            <property name="throwsIndent" value="8"/>
            <property name="lineWrappingIndentation" value="8"/>
        </module>
        <module name="LeftCurly"/>
        <module name="RightCurly"/>
        <module name="WhitespaceAround"/>
        <module name="WhitespaceAfter"/>
        <module name="NoWhitespaceBefore"/>
        <module name="GenericWhitespace"/>
        <module name="EmptyLineSeparator">
            <property name="allowNoEmptyLineBetweenFields" value="true"/>
        </module>

        <!-- CODING -->
        <module name="EqualsHashCode"/>
        <module name="MissingSwitchDefault"/>
        <module name="SimplifyBooleanExpression"/>
        <module name="SimplifyBooleanReturn"/>
        <module name="StringLiteralEquality"/>
        <module name="NestedIfDepth">
            <property name="max" value="3"/>
        </module>
        <module name="NestedTryDepth">
            <property name="max" value="2"/>
        </module>
        <module name="OneStatementPerLine"/>
        <module name="MultipleVariableDeclarations"/>
        <module name="FallThrough"/>
        <module name="UpperEll"/>
        <module name="IllegalThrows"/>
        <module name="IllegalCatch">
            <property name="illegalClassNames"
                      value="java.lang.Exception, java.lang.Throwable"/>
        </module>

        <!-- DESIGN -->
        <module name="FinalClass"/>
        <module name="HideUtilityClassConstructor"/>
        <module name="InterfaceIsType"/>
        <module name="OneTopLevelClass"/>
        <module name="MutableException"/>

        <!-- ANNOTATIONS -->
        <module name="MissingOverride"/>
        <module name="AnnotationLocation"/>

        <!-- JAVADOC (optional — can be noisy) -->
        <module name="JavadocMethod">
            <property name="accessModifiers" value="public"/>
            <property name="allowMissingParamTags" value="true"/>
            <property name="allowMissingReturnTag" value="true"/>
        </module>
        <module name="JavadocType">
            <property name="scope" value="public"/>
        </module>

    </module>
</module>
```

**Running Checkstyle:**

```bash
# Maven
mvn checkstyle:check          # check and fail if violations exist
mvn checkstyle:checkstyle     # generate HTML report (target/site/checkstyle.html)

# Gradle
./gradlew checkstyleMain      # check main sources
./gradlew checkstyleTest      # check test sources
./gradlew checkstyleMain checkstyleTest  # check everything
```

### Google Java Style Guide

The **Google Java Style Guide** is the most widely adopted Java style standard in the industry. It is specific, unambiguous, and machine-enforceable. Rather than debating style in code reviews, you point to the guide and let the tools enforce it.

Key rules from the guide:

```
File structure:
  → One top-level class per file
  → Package statement, then imports, then class declaration
  → No wildcard imports (import java.util.*)
  → Imports ordered: static imports, then java.*, javax.*, org.*, com.*, others

Naming:
  → Classes: PascalCase (AccountService, TransferController)
  → Methods: camelCase (calculateBalance, findById)
  → Constants: UPPER_SNAKE_CASE (MAX_RETRY_COUNT, DEFAULT_TIMEOUT)
  → Packages: lowercase, dot-separated (com.example.banking.services)
  → Type parameters: single uppercase letter or short PascalCase (T, E, TKey)

Formatting:
  → 2-space indent (Google's choice — many teams use 4; pick one and enforce it)
  → 100-character line limit (Google) or 120 (common enterprise choice)
  → Braces: K&R style (opening brace on same line)
  → One statement per line
  → Column alignment: discouraged (fragile when names change)

Programming practices:
  → @Override on every overridden method
  → Catch specific exceptions, not Exception or Throwable
  → No finalizers
  → Use @Nullable and @NonNull annotations
```

**Using the Google Checkstyle configuration:**

```xml
<!-- In your maven-checkstyle-plugin configuration -->
<configuration>
    <!-- Use Google's official Checkstyle config directly -->
    <configLocation>google_checks.xml</configLocation>
    <!-- Or download and customize: -->
    <!-- <configLocation>https://raw.githubusercontent.com/checkstyle/checkstyle/master/src/main/resources/google_checks.xml</configLocation> -->
</configuration>
```

### SpotBugs

SpotBugs is the **successor to FindBugs** (which is no longer maintained). It analyzes Java **bytecode** (compiled `.class` files) to find bugs that the compiler misses: null pointer dereferences, resource leaks, infinite loops, incorrect synchronization, and more.

**Maven configuration:**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.github.spotbugs</groupId>
            <artifactId>spotbugs-maven-plugin</artifactId>
            <version>4.8.3.1</version>
            <configuration>
                <effort>Max</effort>          <!-- Min, Default, Max -->
                <threshold>Medium</threshold>  <!-- Low, Medium, High, Exp -->
                <failOnError>true</failOnError>
                <xmlOutput>true</xmlOutput>
                <excludeFilterFile>spotbugs-exclude.xml</excludeFilterFile>
                <plugins>
                    <!-- Find Security Bugs: security-specific patterns -->
                    <plugin>
                        <groupId>com.h3xstream.findsecbugs</groupId>
                        <artifactId>findsecbugs-plugin</artifactId>
                        <version>1.13.0</version>
                    </plugin>
                </plugins>
            </configuration>
            <executions>
                <execution>
                    <id>spotbugs-check</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**Gradle configuration:**

```kotlin
plugins {
    id("com.github.spotbugs") version "6.0.9"
}

spotbugs {
    toolVersion.set("4.8.3")
    effort.set(com.github.spotbugs.snom.Effort.MAX)
    reportLevel.set(com.github.spotbugs.snom.Confidence.MEDIUM)
    excludeFilter.set(file("spotbugs-exclude.xml"))
}

tasks.spotbugsMain {
    reports {
        create("html") {
            required.set(true)
            outputLocation.set(file("$buildDir/reports/spotbugs/main.html"))
        }
    }
}

dependencies {
    spotbugsPlugins("com.h3xstream.findsecbugs:findsecbugs-plugin:1.13.0")
}
```

**SpotBugs exclusion filter (`spotbugs-exclude.xml`):**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<FindBugsFilter>
    <!-- Exclude generated code (MapStruct, JPA metamodel) -->
    <Match>
        <Source name="~.*Impl\.java"/>
        <Bug pattern="RV_RETURN_VALUE_IGNORED_NO_SIDE_EFFECT"/>
    </Match>

    <!-- Exclude test code from certain checks -->
    <Match>
        <Class name="~.*Test"/>
        <Bug pattern="URF_UNREAD_FIELD"/>
    </Match>

    <!-- Exclude specific false positives -->
    <Match>
        <Class name="com.example.banking.config.SecurityConfig"/>
        <Bug pattern="EI_EXPOSE_REP"/>
    </Match>
</FindBugsFilter>
```

**Common SpotBugs findings in Java backend code:**

```
NP_NULL_ON_SOME_PATH:
  → A variable may be null on some code path but is dereferenced without a check
  → Fix: add null check, use Optional, or annotate with @NonNull

RCN_REDUNDANT_NULLCHECK_OF_NONNULL_VALUE:
  → Checking a value for null that cannot be null
  → Fix: remove the redundant check

DM_DEFAULT_ENCODING:
  → Using new String(bytes) without specifying a charset
  → Fix: new String(bytes, StandardCharsets.UTF_8)

SQL_INJECTION_JDBC:
  → Building SQL queries with string concatenation (from Find Security Bugs)
  → Fix: use PreparedStatement with parameterized queries

EI_EXPOSE_REP / EI_EXPOSE_REP2:
  → Exposing internal mutable state (returning a direct reference to an array/list)
  → Fix: return a defensive copy or unmodifiable view

IS2_INCONSISTENT_SYNC:
  → A field is sometimes accessed with synchronization and sometimes without
  → Fix: consistently synchronize or use concurrent data structures

SE_BAD_FIELD:
  → A Serializable class contains a non-serializable field
  → Fix: mark the field transient or make it serializable
```

### PMD

PMD analyzes Java **source code** (not bytecode) to find code smells, suboptimal patterns, and potential bugs. It complements SpotBugs by catching different categories of issues.

**Maven configuration:**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-pmd-plugin</artifactId>
            <version>3.21.2</version>
            <configuration>
                <rulesets>
                    <ruleset>/rulesets/java/quickstart.xml</ruleset>
                    <ruleset>pmd-rules.xml</ruleset>  <!-- custom rules -->
                </rulesets>
                <failOnViolation>true</failOnViolation>
                <printFailingErrors>true</printFailingErrors>
                <targetJdk>21</targetJdk>
                <includeTests>true</includeTests>
            </configuration>
            <executions>
                <execution>
                    <id>pmd-check</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>check</goal>
                        <goal>cpd-check</goal>  <!-- Copy-Paste Detector -->
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**Gradle configuration:**

```kotlin
plugins {
    pmd
}

pmd {
    toolVersion = "7.0.0"
    isConsoleOutput = true
    ruleSets = emptyList()  // use ruleSetFiles instead
    ruleSetFiles = files("pmd-rules.xml")
    isIgnoreFailures = false
}

tasks.pmdMain {
    reports {
        html.required.set(true)
        xml.required.set(true)
    }
}
```

**Custom PMD ruleset (`pmd-rules.xml`):**

```xml
<?xml version="1.0"?>
<ruleset name="Banking API Rules"
         xmlns="http://pmd.sourceforge.net/ruleset/2.0.0">

    <description>PMD rules for the banking API project</description>

    <!-- Best Practices -->
    <rule ref="category/java/bestpractices.xml">
        <exclude name="JUnitTestsShouldIncludeAssert"/>
        <exclude name="JUnitTestContainsTooManyAsserts"/>
    </rule>

    <!-- Code Style -->
    <rule ref="category/java/codestyle.xml">
        <exclude name="AtLeastOneConstructor"/>
        <exclude name="OnlyOneReturn"/>
        <exclude name="LongVariable"/>
    </rule>
    <rule ref="category/java/codestyle.xml/ClassNamingConventions">
        <properties>
            <property name="utilityClassPattern" value="[A-Z][a-zA-Z0-9]+(Utils?|Helper|Constants)"/>
        </properties>
    </rule>

    <!-- Design -->
    <rule ref="category/java/design.xml">
        <exclude name="LawOfDemeter"/>
        <exclude name="LoosePackageCoupling"/>
    </rule>
    <rule ref="category/java/design.xml/CyclomaticComplexity">
        <properties>
            <property name="methodReportLevel" value="15"/>
            <property name="classReportLevel" value="80"/>
        </properties>
    </rule>
    <rule ref="category/java/design.xml/ExcessiveMethodLength">
        <properties>
            <property name="minimum" value="50"/>
        </properties>
    </rule>

    <!-- Error Prone -->
    <rule ref="category/java/errorprone.xml">
        <exclude name="BeanMembersShouldSerialize"/>
        <exclude name="DataflowAnomalyAnalysis"/>
    </rule>

    <!-- Performance -->
    <rule ref="category/java/performance.xml"/>

    <!-- Security -->
    <rule ref="category/java/security.xml"/>

    <!-- Multithreading -->
    <rule ref="category/java/multithreading.xml"/>

</ruleset>
```

**PMD's Copy-Paste Detector (CPD):**

```bash
# CPD finds duplicated code blocks across your codebase
# Maven: runs automatically with pmd:cpd-check
# Gradle:
./gradlew cpdCheck

# Configuration in build.gradle.kts:
tasks.register<net.sourceforge.pmd.cpd.Cpd>("cpdCheck") {
    language = net.sourceforge.pmd.cpd.CppLanguage()  // or JavaLanguage()
    minimumTokenCount = 100  // ~10-15 lines of Java
    source = fileTree("src/main/java")
    reports {
        text.required.set(true)
    }
}
```

### Error Prone

Error Prone is **Google's compile-time static analysis tool** for Java. It runs as a **compiler plugin** during `javac` and catches bugs at compile time that would otherwise only appear at runtime. It is faster than SpotBugs or PMD because it integrates directly into the compilation step.

**Maven configuration:**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>21</source>
                <target>21</target>
                <compilerArgs>
                    <arg>-XDcompilePolicy=simple</arg>
                    <arg>-Xplugin:ErrorProne</arg>
                </compilerArgs>
                <annotationProcessorPaths>
                    <path>
                        <groupId>com.google.errorprone</groupId>
                        <artifactId>error_prone_core</artifactId>
                        <version>2.26.1</version>
                    </path>
                    <!-- Add other annotation processors here (MapStruct, etc.) -->
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Gradle configuration:**

```kotlin
plugins {
    id("net.ltgt.errorprone") version "3.1.0"
}

dependencies {
    errorprone("com.google.errorprone:error_prone_core:2.26.1")
}

tasks.withType<JavaCompile>().configureEach {
    options.errorprone {
        disableWarningsInGeneratedCode.set(true)
        error("UnusedVariable", "UnusedMethod", "MissingOverride")
        warn("StringSplitter", "ImmutableEnumChecker")
        // Disable noisy checks
        disable("MissingSummary")
    }
}
```

**Common Error Prone checks:**

```
UnusedVariable:
  → A local variable is declared but never read
  → Fix: remove the variable or use it

MissingOverride:
  → A method overrides a superclass method but lacks @Override
  → Fix: add @Override annotation

StringSplitter:
  → String.split() has surprising behavior with regex special characters
  → Fix: use Splitter from Guava or Pattern.split()

ImmutableEnumChecker:
  → Enum fields should be final (enums are supposed to be immutable)
  → Fix: make all enum fields final

ReturnValueIgnored:
  → A method returns a value that is silently discardedarded
  → Fix: capture the return value or annotate the method with @CanIgnoreReturnValue

EqualsIncompatibleType:
  → Calling .equals() with an argument of an incompatible type
  → Fix: fix the type or use instanceof before comparing

CollectionIncompatibleType:
  → Looking up a key of the wrong type in a Map or Set
  → Fix: use the correct key type

NullOptional:
  → Passing null to Optional.of() (use Optional.ofNullable() instead)
  → Fix: use Optional.ofNullable() or ensure the value is non-null
```

### SonarLint and SonarQube

**SonarLint** is a free IDE plugin that provides real-time feedback as you code. **SonarQube** is a self-hosted server that tracks code quality trends across your entire project over time.

**SonarLint (IDE plugin):**

```
Installation:
  → IntelliJ: Settings → Plugins → search "SonarLint" → Install
  → VS Code: Extensions → search "SonarLint" → Install

Configuration:
  → Works out of the box with sensible defaults
  → Can connect to a SonarQube server to sync rules and quality profiles
  → Highlights issues inline with severity (blocker, critical, major, minor, info)
  → Provides fix suggestions and explanations for each issue

Categories of issues SonarLint catches:
  → Bugs: null pointer dereferences, resource leaks, infinite recursion
  → Vulnerabilities: SQL injection, XSS, hardcoded credentials, weak crypto
  → Code smells: long methods, complex conditionals, duplicated blocks
  → Security hotspots: code that requires manual security review
```

**SonarQube (server — overview):**

```
SonarQube is a self-hosted (or cloud: SonarCloud) platform that:
  → Analyzes your entire codebase on every CI run
  → Tracks quality trends over time (is the codebase getting better or worse?)
  → Enforces Quality Gates (e.g., "no new bugs, 80% coverage on new code")
  → Provides a dashboard for engineering managers and tech leads
  → Integrates with GitHub/GitLab to block PRs that fail quality gates

Setup (Docker for local development):
  docker run -d --name sonarqube \
    -p 9000:9000 \
    -e SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar \
    sonarqube:lts

Maven analysis:
  mvn sonar:sonar \
    -Dsonar.projectKey=banking-api \
    -Dsonar.host.url=http://localhost:9000 \
    -Dsonar.token=sqp_xxx

Gradle analysis:
  plugins { id("org.sonarqube") version "4.4.1.3373" }
  ./gradlew sonar \
    -Dsonar.projectKey=banking-api \
    -Dsonar.host.url=http://localhost:9000 \
    -Dsonar.token=sqp_xxx

Quality Gate example:
  → Coverage on new code ≥ 80%
  → No new blocker or critical bugs
  → No new vulnerabilities
  → Duplicated lines on new code ≤ 3%
  → Technical debt ratio on new code ≤ 5%
```

### Spotless — Auto-Formatting

Spotless is a **multi-language auto-formatter** that integrates with Maven and Gradle. It can enforce formatting for Java, Kotlin, SQL, Markdown, YAML, and more. The key advantage: **it fixes violations automatically** instead of just reporting them.

**Maven configuration:**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.diffplug.spotless</groupId>
            <artifactId>spotless-maven-plugin</artifactId>
            <version>2.43.0</version>
            <configuration>
                <java>
                    <!-- Use Google Java Format -->
                    <googleJavaFormat>
                        <version>1.19.2</version>
                        <style>GOOGLE</style> <!-- or AOSP (4-space indent) -->
                    </googleJavaFormat>

                    <!-- Import ordering -->
                    <importOrder>
                        <order>java,javax,org,com,</order>
                    </importOrder>

                    <!-- Remove unused imports -->
                    <removeUnusedImports/>

                    <!-- Add license header -->
                    <licenseHeader>
                        <content>/* (C) $YEAR Example Corp */</content>
                    </licenseHeader>

                    <!-- Trim trailing whitespace -->
                    <trimTrailingWhitespace/>

                    <!-- Ensure newline at end of file -->
                    <endWithNewline/>
                </java>

                <!-- Format SQL files (Flyway migrations) -->
                <sql>
                    <includes>
                        <include>src/main/resources/db/migration/*.sql</include>
                    </includes>
                    <dbeaver/>
                </sql>

                <!-- Format YAML files -->
                <yaml>
                    <includes>
                        <include>src/main/resources/**/*.yml</include>
                    </includes>
                    <jackson/>
                </yaml>

                <!-- Format Markdown -->
                <markdown>
                    <includes>
                        <include>**/*.md</include>
                    </includes>
                    <flexmark/>
                </markdown>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>check</goal>  <!-- fail build if not formatted -->
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**Gradle configuration:**

```kotlin
plugins {
    id("com.diffplug.spotless") version "6.25.0"
}

spotless {
    java {
        target("src/**/*.java")
        googleJavaFormat("1.19.2").aosp()  // AOSP = 4-space indent
        importOrder("java", "javax", "org", "com", "")
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
        licenseHeader("/* (C) \$YEAR Example Corp */")
    }

    kotlinGradle {
        target("*.gradle.kts")
        ktlint()
    }

    sql {
        target("src/main/resources/db/migration/*.sql")
        dbeaver()
    }

    yaml {
        target("src/main/resources/**/*.yml")
        jackson()
    }

    format("misc") {
        target("**/*.md", "**/.gitignore")
        trimTrailingWhitespace()
        endWithNewline()
    }
}
```

**Running Spotless:**

```bash
# Check formatting (fails if any file is not formatted correctly)
mvn spotless:check          # Maven
./gradlew spotlessCheck     # Gradle

# Auto-fix formatting (rewrites files in place)
mvn spotless:apply          # Maven
./gradlew spotlessApply     # Gradle

# Workflow:
# 1. Write code (don't worry about formatting)
# 2. Run spotless:apply / spotlessApply
# 3. Review the changes (git diff)
# 4. Commit
```

### google-java-format

`google-java-format` is the standalone formatter that Spotless uses under the hood. You can also use it directly:

```bash
# Install (via Homebrew)
brew install google-java-format

# Format a single file
google-java-format -i src/main/java/com/example/Account.java

# Format all Java files in a directory
google-java-format -i -r src/

# Check without modifying (for CI)
google-java-format --dry-run --set-exit-if-changed -r src/

# IntelliJ plugin:
# Settings → Plugins → search "google-java-format" → Install
# Settings → Other Settings → google-java-format → Enable
# Now Ctrl+Alt+L (Reformat Code) uses Google style
```

### Pre-Commit Hooks

Pre-commit hooks run **before each `git commit`** and can reject the commit if checks fail. This catches issues at the earliest possible moment.

**Using the `pre-commit` framework:**

```bash
# Install
pip install pre-commit   # or: brew install pre-commit

# Install hooks into the Git repository
pre-commit install
pre-commit install --hook-type commit-msg  # for commit message validation
```

**`.pre-commit-config.yaml`:**

```yaml
repos:
  # General file checks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-xml
      - id: check-json
      - id: check-merge-conflict
      - id: detect-private-key
      - id: no-commit-to-branch
        args: ['--branch', 'main']

  # Java formatting (Spotless)
  - repo: local
    hooks:
      - id: spotless
        name: Spotless Format Check
        entry: ./mvnw spotless:check
        language: system
        pass_filenames: false
        files: '\.java$'
        stages: [commit]

  # Checkstyle
  - repo: local
    hooks:
      - id: checkstyle
        name: Checkstyle
        entry: ./mvnw checkstyle:check
        language: system
        pass_filenames: false
        files: '\.java$'
        stages: [commit]

  # Compilation check (catches syntax errors early)
  - repo: local
    hooks:
      - id: compile
        name: Java Compilation
        entry: ./mvnw compile -q
        language: system
        pass_filenames: false
        files: '\.java$'
        stages: [commit]

  # Conventional commit message validation
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: v3.1.0
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
        args: [feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert]
```

**Git hook without the framework (`.git/hooks/pre-commit`):**

```bash
#!/bin/sh
set -e

echo "Running pre-commit checks..."

# Check formatting
echo ">> Spotless check"
./mvnw spotless:check -q

# Check style
echo ">> Checkstyle"
./mvnw checkstyle:check -q

# Compile
echo ">> Compilation"
./mvnw compile -q

echo "✓ All pre-commit checks passed"
```

### OWASP Dependency-Check

OWASP Dependency-Check scans your project's dependencies against the **National Vulnerability Database (NVD)** and other sources to find known CVEs (Common Vulnerabilities and Exposures).

**Maven configuration:**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.owasp</groupId>
            <artifactId>dependency-check-maven</artifactId>
            <version>9.0.10</version>
            <configuration>
                <failBuildOnCVSS>7</failBuildOnCVSS>  <!-- fail on HIGH or CRITICAL -->
                <suppressionFiles>
                    <suppressionFile>dependency-check-suppressions.xml</suppressionFile>
                </suppressionFiles>
                <formats>
                    <format>HTML</format>
                    <format>JSON</format>
                </formats>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**Gradle configuration:**

```kotlin
plugins {
    id("org.owasp.dependencycheck") version "9.0.10"
}

dependencyCheck {
    failBuildOnCVSS = 7.0f  // fail on HIGH or CRITICAL
    suppressionFile = "dependency-check-suppressions.xml"
    formats = listOf("HTML", "JSON")
}
```

**Running the scan:**

```bash
# Maven
mvn dependency-check:check
# Report: target/dependency-check-report.html

# Gradle
./gradlew dependencyCheckAnalyze
# Report: build/reports/dependency-check-report.html
```

**Suppression file (`dependency-check-suppressions.xml`):**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
    <!-- Suppress a known false positive -->
    <suppress>
        <notes>False positive: this CVE affects a different library with the same name</notes>
        <gav regex="true">com\.example:some-lib:.*</gav>
        <cve>CVE-2024-12345</cve>
    </suppress>

    <!-- Suppress until a fix is available (with expiration) -->
    <suppress until="2025-06-01Z">
        <notes>Awaiting upstream fix in Spring Framework 6.2</notes>
        <gav regex="true">org\.springframework:spring-.*:6\.1\..*</gav>
        <cve>CVE-2025-99999</cve>
    </suppress>
</suppressions>
```

### Snyk (Overview)

Snyk is a **commercial vulnerability scanner** (with a free tier) that goes beyond OWASP Dependency-Check:

```
Features:
  → Scans dependencies for known vulnerabilities (like OWASP)
  → Scans Docker images for OS-level vulnerabilities
  → Scans Infrastructure as Code (Terraform, Kubernetes manifests)
  → Provides fix recommendations and automated PRs
  → Integrates with GitHub, GitLab, Bitbucket
  → Monitors for new vulnerabilities after deployment

Integration:
  → CLI: snyk test (run locally)
  → GitHub App: automatic PR checks on every pull request
  → CI/CD: snyk test in your pipeline
  → IDE plugin: real-time vulnerability highlighting

Maven/Gradle integration:
  → Maven: mvn snyk:test (requires snyk-maven-plugin)
  → Gradle: ./gradlew snyk-test (requires snyk-gradle-plugin)
  → Or simply: snyk test --file=pom.xml

When to use Snyk vs OWASP:
  → OWASP Dependency-Check: free, offline, good for CI pipelines
  → Snyk: more comprehensive, better UI, fix automation, Docker scanning
  → Many teams use both: OWASP in CI, Snyk for continuous monitoring
```

---

## Important Notes

### The Unified Quality Pipeline

Here is how all these tools fit together in a production-grade Java project:

```
Developer writes code
    │
    ▼
IDE (real-time):
  → SonarLint highlights issues inline
  → Spotless auto-formats on save
  → Checkstyle plugin shows violations
    │
    ▼
git commit (pre-commit hooks):
  → Spotless check (formatting)
  → Checkstyle (style rules)
  → Compilation check
  → Conventional commit message validation
    │
    ▼
git push → CI pipeline (GitHub Actions):
  → mvn clean verify
      ├── compile (Error Prone runs here)
      ├── test (JUnit 5 + JaCoCo coverage)
      ├── checkstyle:check
      ├── spotbugs:check
      ├── pmd:check + cpd-check
      ├── spotless:check
      └── dependency-check:check
  → SonarQube analysis (quality gate)
  → Snyk vulnerability scan
    │
    ▼
PR Review:
  → All CI checks must be green
  → SonarQube quality gate must pass
  → No new vulnerabilities above threshold
  → Coverage on new code ≥ 80%
    │
    ▼
Merge to main → Deploy
```

### Maven Configuration — All Tools Together

```xml
<build>
    <plugins>
        <!-- 1. Compiler with Error Prone -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>21</source>
                <target>21</target>
                <compilerArgs>
                    <arg>-XDcompilePolicy=simple</arg>
                    <arg>-Xplugin:ErrorProne</arg>
                </compilerArgs>
                <annotationProcessorPaths>
                    <path>
                        <groupId>com.google.errorprone</groupId>
                        <artifactId>error_prone_core</artifactId>
                        <version>2.26.1</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>

        <!-- 2. Checkstyle (runs in validate phase) -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <version>3.3.1</version>
            <configuration>
                <configLocation>checkstyle.xml</configLocation>
                <failsOnError>true</failsOnError>
            </configuration>
            <executions>
                <execution>
                    <phase>validate</phase>
                    <goals><goal>check</goal></goals>
                </execution>
            </executions>
        </plugin>

        <!-- 3. Spotless (runs in verify phase) -->
        <plugin>
            <groupId>com.diffplug.spotless</groupId>
            <artifactId>spotless-maven-plugin</artifactId>
            <version>2.43.0</version>
            <configuration>
                <java>
                    <googleJavaFormat><style>AOSP</style></googleJavaFormat>
                    <removeUnusedImports/>
                    <importOrder><order>java,javax,org,com,</order></importOrder>
                </java>
            </configuration>
            <executions>
                <execution>
                    <phase>verify</phase>
                    <goals><goal>check</goal></goals>
                </execution>
            </executions>
        </plugin>

        <!-- 4. SpotBugs (runs in verify phase) -->
        <plugin>
            <groupId>com.github.spotbugs</groupId>
            <artifactId>spotbugs-maven-plugin</artifactId>
            <version>4.8.3.1</version>
            <configuration>
                <effort>Max</effort>
                <threshold>Medium</threshold>
                <failOnError>true</failOnError>
            </configuration>
            <executions>
                <execution>
                    <phase>verify</phase>
                    <goals><goal>check</goal></goals>
                </execution>
            </executions>
        </plugin>

        <!-- 5. PMD (runs in verify phase) -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-pmd-plugin</artifactId>
            <version>3.21.2</version>
            <configuration>
                <failOnViolation>true</failOnViolation>
                <targetJdk>21</targetJdk>
            </configuration>
            <executions>
                <execution>
                    <phase>verify</phase>
                    <goals><goal>check</goal></goals>
                </execution>
            </executions>
        </plugin>

        <!-- 6. JaCoCo (runs in test phase) -->
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

        <!-- 7. OWASP Dependency-Check (runs in verify phase) -->
        <plugin>
            <groupId>org.owasp</groupId>
            <artifactId>dependency-check-maven</artifactId>
            <version>9.0.10</version>
            <configuration>
                <failBuildOnCVSS>7</failBuildOnCVSS>
            </configuration>
            <executions>
                <execution>
                    <phase>verify</phase>
                    <goals><goal>check</goal></goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### Handling False Positives

Every static analysis tool produces false positives. The key is to **suppress them explicitly and document why**, not to disable the entire rule:

```
Checkstyle:
  → // CHECKSTYLE:OFF: MethodLength
    ... long method with justified complexity ...
    // CHECKSTYLE:ON: MethodLength
  → Or use @SuppressWarnings("checkstyle:MethodLength")

SpotBugs:
  → @SuppressFBWarnings(value = "NP_NULL_ON_SOME_PATH",
      justification = "Null check performed in calling method")
  → Or use spotbugs-exclude.xml for file-level suppressions

PMD:
  → @SuppressWarnings("PMD.ExcessiveMethodLength")
  → // NOPMD comment at end of line

Error Prone:
  → @SuppressWarnings("UnusedVariable")
  → Or configure in the compiler plugin:
    <arg>-Xep:UnusedVariable:OFF</arg>

General rule:
  → Suppress at the NARROWEST scope possible (method, not class)
  → Always include a justification
  → Review suppressions periodically (they may become stale)
  → If a rule produces too many false positives, reconsider the rule, not the code
```

### Performance Considerations

```
Quality tools add build time. Here is how to keep it manageable:

1. Run fast checks early, slow checks late:
   → validate phase: Checkstyle, Spotless (seconds)
   → compile phase: Error Prone (adds ~10-20% to compilation)
   → test phase: JaCoCo (adds ~5-10% to test execution)
   → verify phase: SpotBugs, PMD, OWASP (can take minutes)

2. Skip slow checks during local development:
   → mvn clean install -Dcheckstyle.skip -Dspotbugs.skip -Dpmd.skip
   → Use a Maven profile for "fast" local builds
   → Let CI run the full suite

3. OWASP Dependency-Check is the slowest:
   → First run downloads the NVD database (~1-2 GB, takes 10-20 minutes)
   → Subsequent runs are faster (incremental updates)
   → Run in CI nightly, not on every PR
   → Use -DnvdApiKey=xxx for faster NVD downloads (free API key from NIST)

4. SpotBugs and PMD on large codebases:
   → Configure to analyze only changed files in CI (incremental analysis)
   → SonarQube does this automatically with its "new code" focus

5. Spotless is fast:
   → Formatting checks take seconds even on large projects
   → Safe to run on every commit and every CI build
```

### Common Anti-Patterns

```
1. "We'll add quality tools later"
   → Adding Checkstyle to a 500K-line codebase produces 50,000 violations
   → Start from day one. It is much harder to retrofit.

2. Disabling rules instead of fixing code
   → Turning off SpotBugs because "it's too noisy" means you miss real bugs
   → Tune the rules, don't disable them

3. Running quality tools only in CI, not locally
   → Developers waste time pushing code that fails CI checks
   → Pre-commit hooks and IDE plugins catch issues in seconds, not minutes

4. Treating all violations equally
   → A missing Javadoc comment is not the same as a SQL injection vulnerability
   → Configure severity levels: ERROR for bugs/security, WARNING for style

5. No suppression review process
   → Suppressions accumulate over time and hide real issues
   → Review suppressions quarterly; remove stale ones

6. Ignoring dependency vulnerabilities
   → "It's a transitive dependency, we can't control it"
   → You CAN: exclude it, force a newer version, or find an alternative
   → Log4Shell (CVE-2021-44228) affected thousands of teams who ignored this
```

---

## Practice

```
1. Add .editorconfig to your project and verify that IntelliJ respects the settings
2. Configure Checkstyle with the Google Java Style ruleset and run it against your
   existing code — fix the top 10 most common violations
3. Create a custom checkstyle.xml that enforces your team's naming conventions
   (PascalCase classes, camelCase methods, UPPER_SNAKE_CASE constants)
4. Add SpotBugs with Find Security Bugs plugin to your Maven/Gradle build and
   analyze the findings
5. Configure PMD with the quickstart ruleset and run the Copy-Paste Detector (CPD)
   to find duplicated code in your project
6. Add Error Prone to your compiler configuration and fix the warnings it produces
7. Install SonarLint in IntelliJ and observe the inline feedback as you write code
8. Configure Spotless with google-java-format and run spotless:apply to auto-format
   your entire codebase
9. Set up pre-commit hooks that run Spotless, Checkstyle, and compilation checks
   before every commit
10. Add OWASP Dependency-Check to your build and review the vulnerability report
11. Create a Maven profile called "fast" that skips all quality checks for local
    development, and a "full" profile that runs everything for CI
12. Configure a SpotBugs suppression for a known false positive with a documented
    justification
13. Set up a SonarQube instance using Docker and run a full analysis of your project
14. Write a CI pipeline step (GitHub Actions) that runs all quality tools and
    fails the build if any check fails
15. Benchmark your build time with and without quality tools — identify which tool
    adds the most overhead and optimize its configuration
```

---

## References

- Checkstyle Documentation: https://checkstyle.org/
- Google Java Style Guide: https://google.github.io/styleguide/javaguide.html
- SpotBugs Documentation: https://spotbugs.readthedocs.io/
- Find Security Bugs: https://find-sec-bugs.github.io/
- PMD Documentation: https://pmd.github.io/
- Error Prone: https://errorprone.info/
- Spotless: https://github.com/diffplug/spotless
- google-java-format: https://github.com/google/google-java-format
- SonarLint: https://www.sonarsource.com/products/sonarlint/
- SonarQube: https://www.sonarsource.com/products/sonarqube/
- OWASP Dependency-Check: https://jeremylong.github.io/DependencyCheck/
- Snyk: https://snyk.io/
- pre-commit Framework: https://pre-commit.com/
- EditorConfig: https://editorconfig.org/
