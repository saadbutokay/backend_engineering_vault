---
title: "Java - Setting Up Environment - IntelliJ and JDK"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - setup
  - ide
status: "not-started"
---

# Java - Setting Up Environment - IntelliJ and JDK

> [!abstract] Overview
> A professional Java backend developer needs three things installed correctly: a JDK, an IDE, and a build tool. This note walks you through setting up JDK 21 via SDKMAN, configuring IntelliJ IDEA on Mac and Linux, creating your first project, and understanding the tooling choices that real backend teams make.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - How Java Works - JVM JRE JDK]]
> - Basic terminal usage (cd, ls, mkdir)

---

## Theory

### What is a Development Environment?
A development environment is the collection of tools you use to write, compile, run, debug, and test your code. In Java backend development, your environment consists of several layers:

1. **The JDK**: The compiler and runtime. Without it, nothing works.
2. **The IDE (Integrated Development Environment)**: A code editor with deep Java intelligence. It understands your code structure, catches errors before compilation, provides autocomplete, and integrates with debuggers and build tools.
3. **The Build Tool**: A program that manages your project's dependencies (external libraries), compiles your code, runs your tests, and packages your application. In Java, the two main build tools are Maven and Gradle. You will learn these in Phase 2, but your IDE will set them up for you from day one.
4. **Version Control (Git)**: Tracks changes to your code over time. You will learn Git properly in Phase 6, but you should install it now.

### Why IntelliJ IDEA?

There are several Java IDEs available: Eclipse, NetBeans, VS Code with Java extensions, and IntelliJ IDEA. In the professional Java backend world, IntelliJ IDEA is the overwhelming industry standard. Surveys consistently show that 70-80% of professional Java developers use IntelliJ.

The reasons are practical:

- **Code intelligence**: IntelliJ understands Java deeply. It can predict what you want to type, detect bugs before you compile, and suggest refactoring improvements.
- **Framework support**: IntelliJ has first-class support for Spring Boot, JPA, Maven, Gradle, Docker, and databases. When you reach Phase 4, IntelliJ will auto-detect your Spring configuration and provide navigation between controllers, services, and repositories.
- **Debugger**: IntelliJ's visual debugger lets you pause execution, inspect variables, and step through code line by line. This is essential for finding bugs in backend services.
- **Database tools**: IntelliJ Ultimate includes a full database client (DataGrip) built into the IDE. You can query your PostgreSQL database without leaving the editor.

IntelliJ comes in two editions:

- **Community Edition**: Free and open source. Supports core Java, Maven, Gradle, and Git. This is sufficient for Phases 0 through 3.
- **Ultimate Edition**: Paid, but free for students with a university email. Supports Spring Boot, Jakarta EE, database tools, Docker, and web frameworks. You should get this as a student because it will be essential from Phase 4 onward.

> [!tip] Key Insight
> As a CSE student, you can get IntelliJ IDEA Ultimate for free through the JetBrains Student License program. Use your university email to apply at https://www.jetbrains.com/student/. Do this now before you need it. The approval usually takes a few days.

### Why SDKMAN for JDK Management?
You could download a JDK installer from Oracle's website and click through the setup wizard. This works, but it creates problems later:

- Professional projects often require different Java versions. One project might use Java 17, another Java 21, and a legacy system might need Java 11.
- Switching between manually installed JDKs on Mac and Linux involves editing environment variables, which is error-prone.
- Oracle's JDK has licensing restrictions for commercial use. OpenJDK distributions like Temurin (from Adoptium) or Corretto (from Amazon) are free for all uses.

**SDKMAN** solves all of this. It is a command-line tool for Mac and Linux that lets you install, switch between, and manage multiple JDK versions with single commands. It also manages other JVM tools like Maven, Gradle, and Spring Boot CLI.

---

## Syntax and Basic Examples

### Step 1: Install SDKMAN (Mac and Linux)

Open your terminal and run:
```bash
# Download and install SDKMAN
curl -s "https://get.sdkman.io" | bash

# Close and reopen your terminal, or run this to activate SDKMAN in the current session:
source "$HOME/.sdkman/bin/sdkman-init.sh"

# Verify the installation
sdk version
```

**Expected output:**
```
SDKMAN!
script: 5.18.2
native: 0.4.6
```

The exact version numbers will differ. The important thing is that you see "SDKMAN!" in the output.

### Step 2: Install JDK 21 (LTS)

LTS stands for Long-Term Support. LTS versions receive security patches and bug fixes for many years. Java 21 is the current LTS release and the version most new backend projects target.

```bash
# List all available Java versions
sdk list java

# Install Temurin JDK 21 (Temurin is the recommended OpenJDK distribution)
sdk install java 21.0.2-tem

# When prompted "Do you want java 21.0.2-tem to be set as default?", type Y
```

**Verify the installation:**

```bash
java --version
javac --version
```

**Expected output:**
```
openjdk 21.0.2 2024-01-16
OpenJDK Runtime Environment Temurin-21.0.2+13 (build 21.0.2+13)
OpenJDK 64-Bit Server VM Temurin-21.0.2+13 (build 21.0.2+13, mixed mode)
```

### Step 3: Useful SDKMAN Commands

```bash
# Install a different JDK version (for example, Java 17 for a legacy project)
sdk install java 17.0.10-tem

# Switch to a different version for the current terminal session only
sdk use java 17.0.10-tem

# Set a different version as the global default
sdk default java 21.0.2-tem

# See which versions you have installed
sdk list java | grep installed

# Uninstall a version you no longer need
sdk uninstall java 17.0.10-tem
```

### Step 4: Install IntelliJ IDEA

**On Mac:**

```bash
# Option A: Using Homebrew (recommended for Mac)
brew install --cask intellij-idea-ce    # Community Edition
# OR
brew install --cask intellij-idea       # Ultimate Edition (requires license)

# Option B: Download from https://www.jetbrains.com/idea/download/
# Choose the Apple Silicon (ARM) version for your M4 chip.
```

**On Linux:**
```bash
# Option A: Using SDKMAN (simplest)
# SDKMAN does not manage IntelliJ, so use snap or download directly.

# Using snap (Ubuntu/Debian-based):
sudo snap install intellij-idea-community --classic
# OR
sudo snap install intellij-idea-ultimate --classic

# Option B: Download the tar.gz from JetBrains website,
# extract it, and run the bin/idea.sh script.
```

### Step 5: First-Time IntelliJ Setup
When you open IntelliJ for the first time, it will walk you through a setup wizard. Here are the recommended choices:

1. **UI Theme**: Choose Darcula (dark) or Light based on your preference. Dark themes are easier on the eyes during long coding sessions.
2. **Plugins**: Accept the defaults for now. You will add more plugins later.
3. **JDK Detection**: IntelliJ should automatically detect the JDK you installed via SDKMAN. If it does not, you can add it manually:
   - Go to File > Project Structure > SDKs
   - Click the "+" button
   - Select "Add JDK"
   - Navigate to `~/.sdkman/candidates/java/21.0.2-tem`
   - Click OK

### Step 6: Create Your First Project
1. Open IntelliJ and click **New Project**.
2. Set the following:
   - **Name**: `hello-backend`
   - **Location**: Choose a folder where you keep your projects (for example, `~/Projects/`)
   - **Language**: Java
   - **Build system**: IntelliJ (for now; switch to Maven in Phase 2)
   - **JDK**: 21 (the one you installed)
   - **Add sample code**: Check this box
3. Click **Create**.

IntelliJ will generate a project with a `Main.java` file that looks like this:
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
```

### Step 7: Run the Program

- Click the green play button (triangle) next to the `main` method in the gutter (left margin of the editor).
- Or press `Ctrl + Shift + F10` (Linux) or `Control + Shift + R` (Mac).
- The output will appear in the **Run** panel at the bottom of the IDE.

**Output:**
```
Hello world!
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> In a real backend team, you never set up projects by clicking through an IDE wizard. Projects are created from templates, cloned from Git repositories, and configured through build files. Here is what a real project setup looks like.

### Scenario: Cloning and running an existing Spring Boot backend service

When you join a company as a junior backend engineer, your first task will look something like this:

```bash
# Step 1: Clone the project repository
git clone git@github.com:company/order-service.git
cd order-service

# Step 2: Check which Java version the project requires
# This is usually documented in the README or in a .sdkmanrc file
cat .sdkmanrc
```

**Example `.sdkmanrc` content:**
```
java=21.0.2-tem
maven=3.9.6
```

```bash
# Step 3: Enable auto-switching in SDKMAN so it reads .sdkmanrc automatically
sdk env

# Step 4: Open the project in IntelliJ
# On Mac:
idea .
# On Linux (if installed via snap):
intellij-idea-community .

# Step 5: IntelliJ will detect the pom.xml (Maven) or build.gradle (Gradle)
# file and automatically download all dependencies. This may take a few minutes
# the first time.

# Step 6: Run the application from the terminal
./mvnw spring-boot:run
```

**What to notice:**

- The `.sdkmanrc` file ensures every developer on the team uses the exact same JDK and Maven versions. This eliminates the "it works on my machine" problem.
- The `./mvnw` command is the Maven Wrapper. It is a script included in the project that downloads the correct Maven version automatically, so developers do not even need Maven installed globally.
- IntelliJ recognizes `pom.xml` and `build.gradle` files and imports the project structure automatically. You do not manually configure source folders or classpaths.
- The `idea .` command opens the current directory in IntelliJ. You need to enable this once in IntelliJ: Tools > Create Command-line Launcher.

### Scenario: Project structure of a real backend service
When you open a Spring Boot project in IntelliJ, the folder structure looks like this:

```
order-service/
├── .sdkmanrc                  # SDK versions for the team
├── pom.xml                    # Maven build configuration and dependencies
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/orderservice/
│   │   │       ├── OrderServiceApplication.java   # Entry point (main method)
│   │   │       ├── controller/                    # HTTP endpoints
│   │   │       ├── service/                       # Business logic
│   │   │       ├── repository/                    # Database access
│   │   │       ├── model/                         # Data classes
│   │   │       └── dto/                           # Data transfer objects
│   │   └── resources/
│   │       ├── application.yml                    # Configuration
│   │       └── db/migration/                      # Database migrations
│   └── test/
│       └── java/
│           └── com/company/orderservice/          # Unit and integration tests
├── Dockerfile                 # Container configuration
├── docker-compose.yml         # Local development environment
└── README.md                  # Setup instructions
```

You will understand every folder in this structure by the end of Phase 5. For now, just recognize that real backend projects have a very specific layout, and IntelliJ understands this layout natively.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Installing JDK from Oracle's website without understanding licensing

**Wrong:** Downloading "Oracle JDK" from oracle.com and using it for a commercial project or a company product.

**Right:** Installing an OpenJDK distribution like Temurin (Adoptium), Corretto (Amazon), or Zulu (Azul). These are free for all uses, including commercial.

**Why it is wrong:** Oracle JDK has a commercial license that requires payment for production use in many scenarios. OpenJDK distributions are fully open source and functionally identical. In professional settings, almost all companies use OpenJDK. SDKMAN makes this easy because you specify the distribution in the install command (for example, `21.0.2-tem` for Temurin).

### Mistake 2: Using VS Code as your primary Java IDE

**Wrong:** Trying to do all Java backend development in VS Code with the Java Extension Pack.

**Right:** Using IntelliJ IDEA for Java development. You can keep VS Code for Python, JavaScript, or markdown files.

**Why it is wrong:** VS Code is an excellent editor, but it is not a full IDE. For Java backend development with Spring Boot, IntelliJ provides significantly better support for dependency injection navigation, Spring configuration, JPA entity mapping, database tools, and debugging. The time you save with IntelliJ's code intelligence will be substantial, especially when your projects grow beyond a few files. Use the right tool for the job.

### Mistake 3: Not setting JAVA_HOME correctly

**Wrong:** Installing the JDK but getting errors like `javac: command not found` or build tools failing to find Java.

**Right:** SDKMAN handles `JAVA_HOME` automatically. If you are not using SDKMAN, you must set it manually:

```bash
# Add to your ~/.zshrc (Mac) or ~/.bashrc (Linux)
export JAVA_HOME="$HOME/.sdkman/candidates/java/current"
export PATH="$JAVA_HOME/bin:$PATH"
```

**Why it is wrong:** Many Java tools (Maven, Gradle, Tomcat, Spring Boot) look for the `JAVA_HOME` environment variable to locate the JDK. If it is not set, they fail silently or with confusing error messages. SDKMAN manages this for you, which is another reason to use it.

### Mistake 4: Creating projects in random locations

**Wrong:** Saving Java projects on the Desktop, in Downloads, or scattered across different folders.

**Right:** Create a dedicated workspace directory:

```bash
mkdir -p ~/Projects/java
mkdir -p ~/Projects/python
cd ~/Projects/java
# Create all Java projects here
```

**Why it is wrong:** As you build more projects, you will need to find them quickly, reference them in your portfolio, and push them to GitHub. A consistent directory structure saves time and prevents confusion. Professional developers always have an organized workspace.

---

## Key Takeaways

> [!tip] Remember these points
> 1. Install **SDKMAN** first, then use it to install **JDK 21 Temurin**. SDKMAN makes version management trivial.
> 2. Install **IntelliJ IDEA**. Use the Community Edition to start, but apply for the **free student license** for Ultimate Edition immediately.
> 3. Set your JDK path in IntelliJ under File > Project Structure > SDKs if auto-detection fails.
> 4. Create a `~/Projects/` directory and keep all your code organized there from day one.
> 5. In professional teams, projects define their JDK version in a `.sdkmanrc` file and use Maven Wrapper (`mvnw`) so every developer has an identical environment.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Full Setup Verification (Easy)
Complete the entire setup from this note. Then open a new terminal and run these commands. Write the output of each command in this note:

```bash
sdk version
java --version
javac --version
echo $JAVA_HOME
```

All four commands should produce valid output with no errors.

**Hint:** If `echo $JAVA_HOME` is empty, restart your terminal or run `source "$HOME/.sdkman/bin/sdkman-init.sh"`.

### Exercise 2: IntelliJ Project (Easy)
Create a new IntelliJ project called `my-first-backend`. Write a program that prints the following three lines:
- Your name
- The Java version you are using (hardcode the string, do not use System.getProperty yet)
- The IDE you are using

Run the program from IntelliJ using the green play button and also from the terminal using `javac` and `java`.

**Hint:** The source file will be in `src/Main.java` inside your project folder. To compile from the terminal, navigate to the `src` directory first.

### Exercise 3: Multiple JDK Versions (Medium)
Install JDK 17 alongside JDK 21 using SDKMAN. Switch between them and verify the version changes. Then switch back to 21 as the default.

```bash
sdk install java 17.0.10-tem
sdk use java 17.0.10-tem
java --version    # Should show 17
sdk use java 21.0.2-tem
java --version    # Should show 21
```

**Hint:** The `sdk use` command only changes the version for the current terminal session. Open a new terminal and check which version is active to understand the difference between `use` and `default`.

### Exercise 4: Apply for Student License (Easy, Important)
Go to https://www.jetbrains.com/student/ and apply for the free student license using your university email. Bookmark the page and note the expected approval time.

**Hint:** You will need this license when you reach Phase 4 and start working with Spring Boot. Getting it now means you will not be blocked later.

---

## Related Notes

- [[Java - How Java Works - JVM JRE JDK]]
- [[Java - Variables and Data Types]]

---

## Resources

- [SDKMAN Official Site](https://sdkman.io/) - Installation guide and full command reference.
- [JetBrains Student License](https://www.jetbrains.com/student/) - Apply for free IntelliJ Ultimate.
- [IntelliJ IDEA Documentation](https://www.jetbrains.com/help/idea/) - Official guide covering every feature.
- [Adoptium Temurin](https://adoptium.net/) - The OpenJDK distribution recommended for production use.
- [IntelliJ Keyboard Shortcuts](https://resources.jetbrains.com/storage/products/intellij-idea/docs/IntelliJIDEA_ReferenceCard.pdf) - Printable shortcut reference. Learn these early to code faster.
