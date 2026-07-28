**Phase:** Level 0 - Environment & Mindset Setup  
**Date Studied:** 27th July, 2026

---
## What Problem Does This Solve?
Before you write a single line of production-quality code, you need a **professional engineering environment**.

Most beginners:
- Use random online compilers (bad habit)
- Don't use version control (catastrophic in real work)
- Don't know what half their tools actually do
- Install things randomly without understanding why

A professional engineer knows:
- **What every tool does**
- **Why that tool exists**
- **How tools connect to each other**
- **How to configure them properly**

Your tools are your workshop.  
A carpenter who doesn't know their tools builds bad furniture.  
An engineer who doesn't know their tools writes bad software.

---
## How All Tools Connect
Before installing anything, understand the big picture.
```
YOU (the engineer)
        │
        ▼
┌───────────────────┐
│   IntelliJ IDEA   │  ← Where you write Java code
│   (Your IDE)      │
└────────┬──────────┘
         │ uses
         ▼
┌───────────────────┐
│    JDK 21         │  ← What actually compiles and runs Java
│ (Java Dev Kit)    │
└────────┬──────────┘
         │ managed by
         ▼
┌───────────────────┐
│  Maven / Gradle   │  ← Builds your project, downloads libraries
│  (Build Tools)    │
└────────┬──────────┘
         │ code saved to
         ▼
┌───────────────────┐
│       Git         │  ← Tracks every change you make
│ (Version Control) │
└────────┬──────────┘
         │ pushed to
         ▼
┌───────────────────┐
│     GitHub        │  ← Stores code online, your portfolio
│  (Remote Repo)    │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐     ┌─────────────────┐
│     Docker        │────▶│   PostgreSQL    │  ← Your database
│   (Containers)    │     │   Redis         │  ← Your cache
└────────┬──────────┘     └─────────────────┘
         │ your app tested with
         ▼
┌───────────────────┐
│     Postman       │  ← You test your APIs here
│  (API Testing)    │
└───────────────────┘

Also used for database visualization:
┌───────────────────┐
│     DBeaver       │  ← See and query your database visually
└───────────────────┘
```

---
## TOOL 1 - JDK 21
Java Development Kit, -version 21.
### What Is It?
The JDK is the **foundation of everything Java**.  
Without it, you cannot write, compile, or run Java code.

```
JDK contains three things:

┌─────────────────────────────────────────────────────┐
│                      JDK 21                         │
│                                                     │
│  ┌─────────────┐  ┌──────────┐  ┌───────────────┐   │
│  │   javac     │  │   JRE    │  │  Development  │   │
│  │ (compiler)  │  │(runtime) │  │    Tools      │   │
│  │             │  │          │  │  (debugger,   │   │
│  │ turns your  │  │ contains │  │  profiler,    │   │
│  │ .java files │  │  JVM     │  │  javadoc...)  │   │
│  │ into .class │  │          │  │               │   │
│  │  bytecode   │  │          │  │               │   │
│  └─────────────┘  └──────────┘  └───────────────┘   │
└─────────────────────────────────────────────────────┘

JVM = Java Virtual Machine
  → Takes bytecode → runs it on YOUR specific computer
  → This is why Java is "write once, run anywhere"
  → Windows JVM, Linux JVM, Mac JVM all run the same bytecode
```

### JDK vs JRE vs JVM
```
JDK (Java Development Kit)
  → For DEVELOPERS
  → Contains: JRE + compiler + development tools
  → This is what YOU install

JRE (Java Runtime Environment)
  → For running Java programs (not developing)
  → Contains: JVM + standard libraries
  → End users install this to run Java apps

JVM (Java Virtual Machine)
  → The actual engine that RUNS the code
  → Converts bytecode to machine code at runtime
  → Handles memory, garbage collection
  → Just a specification — multiple implementations exist
     (OpenJDK, Oracle JDK, Amazon Corretto, Eclipse Temurin)

Simple rule for you:
  Install JDK → you get everything.
```

### Why JDK 21 Specifically?
```
Java releases a new version every 6 months.
BUT not every version is "LTS" (Long Term Support).

LTS versions receive updates and security fixes for years.
Non-LTS versions are abandoned after 6 months.

LTS versions:
  Java 8   (2014) → Still used in old codebases — know it exists
  Java 11  (2018) → Was the standard for years
  Java 17  (2021) → Very widely used RIGHT NOW
  Java 21  (2023) → Current LTS — use this ← YOUR CHOICE
  Java 25  (2025) → Next LTS (future)

Why Java 21:
  ✓ Latest LTS — most up to date stable version
  ✓ Spring Boot 3.x requires Java 17+ (21 works perfectly)
  ✓ New features: Records, Sealed Classes, Pattern Matching,
    Virtual Threads (Project Loom — revolutionary for backends)
  ✓ Companies are migrating to 21 right now
  ✓ Job postings increasingly say "Java 17+" or "Java 21"

Which JDK distribution to install?
  Eclipse Temurin (by Adoptium) ← INSTALL THIS
  → Free, open source, production-grade
  → Used by most companies that don't use Oracle JDK
  → Oracle JDK requires paid license for commercial use
  → Temurin is the safe, free choice
```

### Installation - JDK 21
```
METHOD 1: Direct Download (Simplest)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Go to: https://adoptium.net/
2. Click "Latest LTS Release"
3. Select: Temurin 21 (LTS)
4. Download for your OS (Windows x64 .msi installer)
5. Run the installer
6. ✅ CHECK "Add to PATH" during installation
   ✅ CHECK "Set JAVA_HOME variable"
   (These two checkboxes are critical)
7. Finish installation

METHOD 2: SDKMAN (Recommended for future flexibility)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SDKMAN lets you install multiple Java versions and switch between them.
Professional developers use this.

On Windows, use Git Bash or WSL for SDKMAN.
On Linux/Mac: 
  curl -s "https://get.sdkman.io" | bash
  source "$HOME/.sdkman/bin/sdkman-init.sh"
  sdk install java 21.0.3-tem
  sdk use java 21.0.3-tem

For now, Method 1 is fine.
Learn SDKMAN later when you need multiple Java versions.
```

### Verify Installation
```
Open Command Prompt (Windows) or Terminal:

  java -version

Expected output:
  openjdk version "21.0.x" 2024-xx-xx
  OpenJDK Runtime Environment Temurin-21...
  OpenJDK 64-Bit Server VM Temurin-21...

  javac -version

Expected output:
  javac 21.0.x

If you see errors:
  → PATH is not set correctly
  → JAVA_HOME is not set
  → Reinstall and make sure to check both boxes
```

### What Is JAVA_HOME and PATH?
```
PATH:
  A list of folders your computer searches when you
  type a command in the terminal.
  When you type "java", your computer looks through
  PATH folders to find the java.exe file.
  Without this, "java" command won't be found.

JAVA_HOME:
  An environment variable that points to your JDK folder.
  Example: C:\Program Files\Eclipse Adoptium\jdk-21...
  
  Many tools (Maven, Gradle, IntelliJ) look for JAVA_HOME
  to know where Java is installed.
  Without this, build tools may not work correctly.

How to verify JAVA_HOME on Windows:
  In Command Prompt:
  echo %JAVA_HOME%
  
  Expected:
  C:\Program Files\Eclipse Adoptium\jdk-21.0.x.x-hotspot

How to set manually if needed (Windows):
  1. Search "Environment Variables" in Windows search
  2. Click "Edit the system environment variables"
  3. Click "Environment Variables" button
  4. Under "System variables" → New
     Name:  JAVA_HOME
     Value: C:\Program Files\Eclipse Adoptium\jdk-21.0.x.x-hotspot
  5. Find "Path" in System variables → Edit → New
     Add: %JAVA_HOME%\bin
  6. OK → OK → OK
  7. Restart Command Prompt
```

---
## TOOL 2 - IntelliJ IDEA
Your IDE, You can choose VSCode if you want.
### What Is It?
**IDE = Integrated Development Environment**
An IDE is not just a text editor.  
It is a complete professional tool that helps you:
```
What IntelliJ does for you:
  ✓ Syntax highlighting (colors make code readable)
  ✓ Auto-completion (suggests code as you type)
  ✓ Error detection (shows errors BEFORE you run)
  ✓ Refactoring (rename variable everywhere safely)
  ✓ Debugger (step through code line by line)
  ✓ Integrated terminal
  ✓ Git integration (see changes, commit, push in UI)
  ✓ Database tools
  ✓ Maven/Gradle integration (build your project)
  ✓ Run configurations (run your Spring Boot app with one click)
  ✓ Code inspections (warns you about bad code patterns)
  ✓ Test runner (run all tests, see which fail)

Without an IDE you would:
  × Write code in Notepad
  × Switch to terminal to compile
  × Struggle to find which line has an error
  × Debug by adding print statements everywhere
  
Professional engineers use IntelliJ for Java.
It is the industry standard.
```

### IntelliJ IDEA Community vs Ultimate
```
Community Edition (FREE):
  ✓ Java development — FULL support
  ✓ Maven and Gradle
  ✓ Git integration
  ✓ Debugger
  ✓ Testing tools
  ✓ Basic database tools
  × No Spring Boot specific features
  × No advanced web frameworks support
  × No profiler
  
  → INSTALL THIS. It is enough for learning everything.
  → You will not miss the Ultimate features until you're mid-level.

Ultimate Edition (PAID — ~$70/month):
  ✓ Everything in Community
  ✓ Spring Boot tooling (Spring-specific inspections, diagrams)
  ✓ Full database tools
  ✓ Profiler
  ✓ Docker integration
  ✓ HTTP client built-in
  
  → FREE for students! Apply with your university email.
  → Go to: https://www.jetbrains.com/community/education/
  → If you have university email, get Ultimate for free.
  → This is the professional version — get it if you can.
```

### Installation - IntelliJ IDEA
```
1. Go to: https://www.jetbrains.com/idea/download/
2. Download Community Edition (free) 
   OR apply for free Ultimate with student license first
3. Run installer
4. Installation options — CHECK THESE:
   ✅ Create Desktop Shortcut
   ✅ Update PATH variable (restart needed)
   ✅ Add "Open Folder as Project"
   ✅ .java (associate .java files with IntelliJ)
5. Install
6. Restart computer
```

### First-Time IntelliJ Setup
```
When IntelliJ opens for the first time:

STEP 1: Set JDK
  File → Project Structure → Project
  SDK → Add SDK → JDK
  Browse to your JDK installation folder
  (Usually: C:\Program Files\Eclipse Adoptium\jdk-21...)
  Apply → OK

STEP 2: Essential Plugins to Install
  File → Settings → Plugins → Marketplace
  Search and install:
  
  ✅ .ignore
     Creates .gitignore files easily
  
  ✅ Rainbow Brackets
     Colors matching brackets — makes code readable
  
  ✅ SonarLint
     Real-time code quality checks — catches bugs as you type
  
  ✅ Lombok (important for Spring Boot later)
     Reduces boilerplate code with annotations
  
  ✅ Docker (if Ultimate)
     Docker integration
  
  Later (when you start Spring Boot):
  ✅ Spring Boot Assistant
  ✅ Spring Initializr

STEP 3: Essential Settings
  File → Settings:
  
  Editor → General → Auto Import:
    ✅ Add unambiguous imports on the fly
    ✅ Optimize imports on the fly
  
  Editor → Code Style → Java:
    Tab size: 4
    Indent: 4
  
  Editor → General:
    ✅ Show line numbers
    ✅ Highlight current line
  
  Build, Execution, Deployment → Compiler:
    ✅ Build project automatically
```

### IntelliJ Keyboard Shortcuts You Must Know
```
These will save you hours every week.
Learn 2-3 per day until you know them all.

NAVIGATION:
  Ctrl + N           → Find any class by name
  Ctrl + Shift + N   → Find any file by name
  Ctrl + F           → Find text in current file
  Ctrl + Shift + F   → Find text in entire project
  Ctrl + Click       → Go to definition (where is this method defined?)
  Alt + F7           → Find all usages of this method/variable
  Ctrl + E           → Recent files
  Ctrl + B           → Go to declaration
  Ctrl + Alt + B     → Go to implementation
  Alt + Left/Right   → Navigate back/forward

CODE WRITING:
  Ctrl + Space       → Auto-complete
  Alt + Enter        → Quick fix (IntelliJ suggests fixes)
  Ctrl + D           → Duplicate line
  Ctrl + Y           → Delete line
  Ctrl + /           → Comment/uncomment line
  Ctrl + Shift + /   → Block comment
  Shift + Alt + Up/Down → Move line up/down
  Ctrl + Shift + Up/Down → Move method up/down
  Tab                → Accept auto-complete suggestion
  Ctrl + P           → Show method parameters

REFACTORING:
  Shift + F6         → Rename (renames everywhere safely)
  Ctrl + Alt + M     → Extract method
  Ctrl + Alt + V     → Extract variable
  Ctrl + Alt + F     → Extract field
  Ctrl + Alt + L     → Format code (DO THIS CONSTANTLY)

RUNNING & DEBUGGING:
  Shift + F10        → Run
  Shift + F9         → Debug
  F8                 → Step over (next line)
  F7                 → Step into (go inside method)
  Shift + F8        → Step out (exit current method)
  F9                 → Resume program
  Ctrl + F8          → Toggle breakpoint

GIT:
  Ctrl + K           → Commit
  Ctrl + Shift + K   → Push
  Ctrl + T           → Update (pull)
  Alt + 9            → Open Git panel

GENERAL:
  Ctrl + Shift + A   → Find any action (if you forget a shortcut)
  Double Shift       → Search everything
  Alt + 1            → Open/close Project panel
  Alt + 4            → Open/close Run panel
  Ctrl + F4          → Close current tab
```

---
## TOOL 3 - Git
Version Control.
### What Is It and Why Is It Non-Negotiable?
```
Imagine you're writing a 2000-line codebase.
You change something → now 500 lines break.
You don't remember what you changed.
Without Git: you cry and start over.
With Git: git revert → back to working state in 30 seconds.

Or:
You and a colleague work on the same file.
Without Git: you overwrite each other's work.
With Git: merge systems handle this automatically.

Or:
Something broke in production last Tuesday.
Without Git: good luck figuring out what changed.
With Git: git log → see exactly what changed, when, by whom.

Git tracks EVERY change to EVERY file in your project.
It's like save states in a video game — but for code.

Every professional software team in the world uses Git.
There is no alternative for professional work.
```

### Installation - Git
```
Windows:
  1. Go to: https://git-scm.com/download/win
  2. Download the installer
  3. Run installer with these important settings:
  
  "Adjusting your PATH environment":
    ✅ Select: "Git from the command line and also from
       3rd-party software"
       (This adds git to your PATH)
  
  "Choosing the default editor used by Git":
    Select: Use Visual Studio Code as Git's default editor
    (Or Nano if you want simple terminal editor)
  
  "Adjusting the name of the initial branch":
    ✅ Override the default branch name → type: main
    (Industry standard is now 'main' not 'master')
  
  "Configuring the line ending conversions":
    ✅ Select: "Checkout Windows-style, commit Unix-style"
    (Prevents line ending issues between Windows and Linux)
  
  Everything else: leave as default
  4. Finish installation

Linux (Ubuntu/Debian):
  sudo apt update
  sudo apt install git

Mac:
  brew install git
  (install Homebrew first: https://brew.sh)
```

### Verify Installation
```
Open Command Prompt or Git Bash:
  git --version

Expected output:
  git version 2.x.x.windows.x
```

### First-Time Git Configuration (CRITICAL)
```
Every commit you make is labeled with your name and email.
Set these ONCE globally — they apply to all your projects.

Open Git Bash or Terminal:

git config --global user.name "Your Full Name"
git config --global user.email "your.email@gmail.com"

Use the SAME email you will use for GitHub.

Set default branch name to 'main':
git config --global init.defaultBranch main

Set VS Code as default editor for Git messages:
git config --global core.editor "code --wait"

Verify your config:
git config --list

You should see:
  user.name=Your Full Name
  user.email=your.email@gmail.com
  init.defaultBranch=main
```

---
## TOOL 4 - GitHub
Your Online Portfolio.
### What Is It?
```
Git  = Local tool on your computer (tracks changes)
GitHub = Website that stores your Git repositories online

GitHub is:
  ✓ Cloud backup for your code
  ✓ Your professional portfolio
  ✓ Collaboration platform (teams work together here)
  ✓ Where open source lives
  ✓ What employers look at BEFORE your resume

Your GitHub profile is your engineering resume.
Active GitHub → shows you code regularly.
Good projects → shows you can build things.
Good README files → shows you can communicate.

Employers WILL check your GitHub.
Treat it professionally from day one.
```

### Setup GitHub
```
1. Go to: https://github.com
2. Sign Up
   Username: choose professionally
     ✅ Good: firstname-lastname, firstnamelastname, fnamelname
     ❌ Bad: xX_codemonkey_Xx, rahimcoder123
   Email: use the same as your git config
   Password: strong password

3. Verify email

4. Profile Setup (do this properly — employers see it):
   Click your avatar → Settings → Profile

   Name: Your Full Real Name
   Bio: "Computer Science student | Java Backend Engineer"
   Location: Dhaka, Bangladesh
   Website: (your portfolio when you have one)
   
   Profile README (optional but impressive):
   Create a repo named exactly as your username
   Add a README.md — it shows on your profile
```

### Connect Your Computer to GitHub with SSH
```
SSH keys let you push code without typing password every time.
This is the professional way to connect.

STEP 1: Generate SSH Key
  Open Git Bash (Windows) or Terminal:
  
  ssh-keygen -t ed25519 -C "your.email@gmail.com"
  
  When prompted:
  "Enter file in which to save the key":
    Press Enter (use default location)
  
  "Enter passphrase":
    Press Enter twice (no passphrase for now — simpler)
  
  This creates two files:
    ~/.ssh/id_ed25519       ← PRIVATE key (never share this!)
    ~/.ssh/id_ed25519.pub   ← PUBLIC key (you give this to GitHub)

STEP 2: Add SSH Key to SSH Agent
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519

STEP 3: Copy Your Public Key
  Windows Git Bash:
    cat ~/.ssh/id_ed25519.pub
    (copy the entire output)
  
  OR:
    clip < ~/.ssh/id_ed25519.pub
    (copies to clipboard automatically)

STEP 4: Add Key to GitHub
  1. GitHub → Click avatar → Settings
  2. Left sidebar: SSH and GPG keys
  3. New SSH key
     Title: "My Laptop" (or whatever your computer is)
     Key type: Authentication Key
     Key: Paste your public key
  4. Add SSH key

STEP 5: Test Connection
  ssh -T git@github.com
  
  Expected (first time):
    "The authenticity of host 'github.com' can't be established"
    Type: yes
  
  Expected after:
    "Hi username! You've successfully authenticated..."
  
  ✅ Now you can push code without password!
```

---
## TOOL 5 - Docker Desktop

### What Is It?
```
The Problem Docker Solves:
  "It works on my computer but not on the server"
  
  Your laptop runs Windows.
  Production server runs Linux.
  Your teammate has a Mac.
  
  Without Docker:
    Different OS → different behavior → bugs appear
    Installing PostgreSQL on everyone's laptop = nightmare
    "Did you use the same PostgreSQL version as production?"
  
  With Docker:
    Package your app + all dependencies into a "container"
    Container runs identically everywhere
    PostgreSQL in Docker = same version, same config everywhere
    One command → entire dev environment running

For you right now, Docker = easy way to run:
  • PostgreSQL (database) without installing it locally
  • Redis (cache) without installing it locally
  • Your Spring Boot app + database together
  • Later: your entire microservices system

Think of Docker containers like shipping containers:
  Standard size, standard shape
  Load them on any ship (any computer/server)
  Contents don't change based on the ship
```

### Installation - Docker Desktop
```
Windows Requirements:
  Windows 10/11 Home or Pro (64-bit)
  WSL 2 enabled (Windows Subsystem for Linux)
  Hardware virtualization enabled in BIOS

STEP 1: Enable WSL 2
  Open PowerShell as Administrator:
  wsl --install
  Restart your computer

STEP 2: Download Docker Desktop
  Go to: https://www.docker.com/products/docker-desktop/
  Download Docker Desktop for Windows

STEP 3: Install
  Run the installer
  ✅ Use WSL 2 instead of Hyper-V (recommended)
  ✅ Add shortcut to desktop
  Restart when prompted

STEP 4: First Run
  Docker Desktop opens
  Complete tutorial (optional but good to do once)
  Sign up for Docker Hub account at hub.docker.com
  (Free account — you'll push images here later)
```

### Verify Docker
```
Open Command Prompt or PowerShell:

docker --version
Expected: Docker version 24.x.x, build ...

docker compose version  
Expected: Docker Compose version v2.x.x

Test Docker works:
docker run hello-world

Expected output:
  "Hello from Docker!
   This message shows that your installation appears to be working correctly."

If you see this → Docker is working perfectly.
```

### Quick Docker Test - Run PostgreSQL
```
Let's immediately use Docker for something real.
Run a PostgreSQL database with one command:

docker run --name my-postgres \
  -e POSTGRES_PASSWORD=password123 \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -d postgres:16

What this does:
  --name my-postgres    → give container a name
  -e POSTGRES_PASSWORD  → set database password
  -e POSTGRES_USER      → set database user
  -e POSTGRES_DB        → create a database named mydb
  -p 5432:5432          → expose port 5432 to your laptop
  -d                    → run in background (detached)
  postgres:16           → use PostgreSQL version 16 image

Check it's running:
docker ps

Stop it:
docker stop my-postgres

Start it again:
docker start my-postgres

Delete it:
docker rm my-postgres

You just ran a full PostgreSQL database
without installing PostgreSQL!
This is the power of Docker.
```

---
## TOOL 6 - Postman
API Testing.
### What Is It?
```
When you build a backend API, you need to TEST it.
The frontend isn't built yet.
You can't click buttons that don't exist.

Postman lets you:
  • Send HTTP requests to your API (GET, POST, PUT, DELETE)
  • See the response (data, status code, headers)
  • Save requests in collections (organized test suites)
  • Test different scenarios (valid input, invalid input, auth)
  • Share API requests with teammates

Every backend engineer uses Postman (or similar tools).
You'll use it every single day.

Alternative: Bruno (free, open source, files stored locally)
Alternative: Insomnia (similar to Postman)
Alternative: curl (terminal — know this too)
```

### Installation - Postman
```
1. Go to: https://www.postman.com/downloads/
2. Download for your OS
3. Install
4. Create free account (sign up)
   → Lets you sync your saved requests across computers

No complex setup needed.
You'll learn to use it when you build your first API.
```

### Quick Postman Test
```
Let's test a real public API right now:

1. Open Postman
2. Click "+" to create new request
3. Method: GET
4. URL: https://jsonplaceholder.typicode.com/users/1
5. Click Send

You should see a JSON response with user data.
This is exactly what your Spring Boot API will return later.

Try:
  GET https://jsonplaceholder.typicode.com/posts
  → Returns list of posts

  GET https://jsonplaceholder.typicode.com/posts/1
  → Returns one specific post

This public API is great for practice.
```

---
## TOOL 7 - DBeaver (Database GUI)
### What Is It?
```
DBeaver lets you:
  • Connect to any database (PostgreSQL, MySQL, MongoDB...)
  • See all tables visually
  • Write and run SQL queries
  • See data in table format (like Excel for your database)
  • Export data to CSV
  • View table relationships (ER diagrams)

You'll use this constantly to:
  • Check if your code actually saved data correctly
  • Run SQL queries to investigate problems
  • Design and modify database schemas
  • Explore production databases (read-only) when debugging
```

### Installation - DBeaver
```
1. Go to: https://dbeaver.io/download/
2. Download "Community Edition" (free)
3. Install with defaults
4. Open DBeaver

Connect to your Docker PostgreSQL:
  1. Database → New Database Connection
  2. Select: PostgreSQL
  3. Fill in:
     Host:     localhost
     Port:     5432
     Database: mydb
     Username: myuser
     Password: password123
  4. Test Connection → if green → Finish

Now you can see your database visually!
```

---

## TOOL 8 - Visual Studio Code (Secondary Editor)

### What Is It?
```
VS Code is a lightweight, powerful code editor.
It's NOT your main Java IDE (IntelliJ is for that).
You'll use VS Code for:
  • Writing Python code (Level 12)
  • Editing YAML files (Docker Compose, Kubernetes configs)
  • Editing Markdown (your Obsidian notes, GitHub README)
  • Quick file edits when IntelliJ is overkill
  • Shell scripts
  • Everything non-Java
```

### Installation - VS Code
```
1. Go to: https://code.visualstudio.com/
2. Download for your OS
3. Install
4. Important: ✅ "Add to PATH" during installation

Useful Extensions to install later:
  • Python (when you get to Level 12)
  • GitLens (better Git integration)
  • Docker (manage containers in VS Code)
  • YAML (YAML syntax support)
  • Markdown All in One (for README and notes)
  • Thunder Client (lightweight Postman alternative inside VS Code)
```

---
## TOOL 9 - Account Setups

### Accounts You Need Right Now
```
1. GITHUB (already done above)
   https://github.com
   Your code portfolio.

2. LEETCODE
   https://leetcode.com
   Daily coding practice.
   Sign up → Set username professionally.
   Start with Easy problems in Arrays section.

3. LINKEDIN
   https://linkedin.com
   Your professional network.
   
   Profile Setup (do this properly):
   • Professional photo (clean background, smile)
   • Headline: "Computer Science Student | Aspiring Java Backend Engineer"
   • About: Short paragraph about your goal
   • Education: Add your university, degree, expected graduation
   • Skills: Java, SQL, Git, Spring Boot (add as you learn)
   • Connect with: classmates, professors, Bangladeshi engineers
   
   Later: Add projects, certifications here.
   Companies WILL search your LinkedIn before interviews.

4. DOCKER HUB
   https://hub.docker.com
   Store your Docker images.
   Sign up with same email.

5. HACKERRANK (optional but useful)
   https://hackerrank.com
   Some Bangladesh companies test here.
   Complete Java Basic and Intermediate certificates.
   Free certificates look good on LinkedIn.

6. SOLOLEARN (optional — for quick reference)
   https://sololearn.com
   Good supplementary resource.
```

---
## TOOL 10 - Terminal / Command Line

### Why Engineers Must Know the Terminal
```
The terminal is the most powerful tool an engineer has.
Servers don't have graphical interfaces.
When you SSH into a production server to debug an issue
at 2 AM, there is no clicking. Only commands.

On Windows you have options:
  Command Prompt (cmd) — basic, avoid for engineering work
  PowerShell          — better, built into Windows
  Git Bash            — comes with Git, Unix-like commands ← use this
  WSL 2 (Ubuntu)      — full Linux on Windows ← best option
```

### Setup WSL 2 (Windows Subsystem for Linux)
```
If you installed Docker, WSL 2 is already installed.

Open Ubuntu in WSL:
  Search "Ubuntu" in Windows Start menu
  OR: In PowerShell: wsl

First time Ubuntu opens:
  Set username (lowercase, no spaces)
  Set password (you won't see it typing — that's normal)

Now you have a full Linux terminal on Windows.
Everything we do in the terminal, do it here.

Why WSL:
  ✓ Real Linux environment
  ✓ Same as production servers (Linux)
  ✓ Unix commands work exactly as documented everywhere
  ✓ Better than Git Bash for complex work
  ✓ Docker Desktop integrates with WSL
```

### Essential Terminal Commands to Know Now
```
These are UNIVERSAL — work on Linux, Mac, WSL, any server.

NAVIGATION:
  pwd               → Print Working Directory (where am I?)
  ls                → List files in current directory
  ls -la            → List ALL files with details (hidden too)
  cd foldername     → Change directory (go into folder)
  cd ..             → Go up one level
  cd ~              → Go to home directory
  cd /              → Go to root directory

FILE OPERATIONS:
  mkdir foldername  → Create new directory
  touch file.txt    → Create empty file
  cat file.txt      → Print file contents to terminal
  cp source dest    → Copy file
  mv source dest    → Move file (also used to rename)
  rm file.txt       → Delete file (CAREFUL — no recycle bin!)
  rm -rf folder     → Delete folder and everything inside (VERY CAREFUL)

VIEWING FILES:
  less file.txt     → View file page by page (q to quit)
  head -20 file.txt → Show first 20 lines
  tail -20 file.txt → Show last 20 lines
  tail -f file.log  → Follow file as it grows (watch live logs!)

SEARCHING:
  grep "word" file  → Find "word" in file
  grep -r "word" .  → Find "word" in all files here
  find . -name "*.java" → Find all Java files

SYSTEM:
  clear             → Clear terminal screen (Ctrl+L also works)
  history           → Show command history
  Ctrl+C            → Stop current process
  Ctrl+D            → Exit terminal/logout

NETWORK:
  curl URL          → Make HTTP request (like Postman in terminal)
  ping google.com   → Check internet connection
  
JAVA SPECIFIC:
  java -version     → Check Java version
  javac FileName.java → Compile Java file
  java ClassName    → Run compiled Java class
  mvn --version     → Check Maven version
  mvn clean install → Build Maven project

EXAMPLES:
  # Go to your projects folder and create a new project
  cd ~/projects
  mkdir my-first-java-app
  cd my-first-java-app
  
  # See what's here
  ls -la
  
  # Make an HTTP request to test an API
  curl https://jsonplaceholder.typicode.com/users/1
```

---
## TOOL 11 - Maven (Build Tool - Quick Introduction)

### What Is It?
```
When you write Java code, you need external libraries.
(Spring Boot, Jackson, JUnit, Hibernate — all external)

Without Maven:
  → Download each library manually as .jar files
  → Put them in the right folder
  → Hope versions are compatible
  → Do this for EVERY developer on the team
  → Update versions manually → nightmare

With Maven:
  → List what you need in pom.xml
  → Maven downloads everything automatically
  → Right versions, right dependencies
  → Same for every developer
  → This is how every Java project in the world works

Maven also:
  → Compiles your code
  → Runs your tests
  → Packages your app into a .jar file
  → Deploys to servers

You will use Maven from day one of Spring Boot.
```

### Installation - Maven
```
Option 1: IntelliJ handles it automatically
  When you create a Maven project in IntelliJ,
  it uses the bundled Maven. This is fine to start.

Option 2: Install manually (recommended)
  1. Go to: https://maven.apache.org/download.cgi
  2. Download: apache-maven-3.9.x-bin.zip
  3. Extract to: C:\Program Files\Apache\maven
  4. Add to PATH:
     MAVEN_HOME = C:\Program Files\Apache\maven
     Add to Path: %MAVEN_HOME%\bin
  5. Restart terminal

Verify:
  mvn --version
  Expected: Apache Maven 3.9.x...
```

---
## Complete Setup Verification Checklist
Run through every item. Don't proceed until all pass.
```
JDK 21:
  □ java -version → shows 21.x.x
  □ javac -version → shows 21.x.x
  □ JAVA_HOME set correctly
  □ PATH includes java

IntelliJ IDEA:
  □ Opens successfully
  □ JDK 21 configured in Project Structure
  □ Can create and run a Hello World program
  □ Plugins installed (Rainbow Brackets, SonarLint)

Git:
  □ git --version → shows version
  □ git config user.name shows your name
  □ git config user.email shows your email
  □ init.defaultBranch = main

GitHub:
  □ Account created with professional username
  □ Email verified
  □ Profile name and bio set
  □ SSH key generated and added to GitHub
  □ ssh -T git@github.com → "successfully authenticated"

Docker:
  □ docker --version → shows version
  □ docker run hello-world → success message
  □ Docker Desktop opens and shows running

Postman:
  □ Installed and opens
  □ Account created
  □ GET request to test API works

DBeaver:
  □ Installed and opens
  □ Can connect to Docker PostgreSQL

VS Code:
  □ Installed and opens
  □ Added to PATH (code . works in terminal)

Accounts:
  □ GitHub (professional username)
  □ LinkedIn (professional profile started)
  □ LeetCode (account created)
  □ Docker Hub (account created)
  □ HackerRank (optional)
```

---
## Hello World the RIGHT Way
_Not just running Hello World — doing it like a professional._
```
STEP 1: Create a folder for ALL your projects
  In terminal (WSL or Git Bash):
  mkdir ~/projects
  cd ~/projects

STEP 2: Create your first project in IntelliJ
  File → New → Project
  Select: Java
  Name: hello-world
  Location: ~/projects/hello-world
  Build system: Maven (important!)
  JDK: 21
  Create

STEP 3: Write Hello World
  src/main/java → right-click → New → Java Class
  Name: Main
  
  public class Main {
      public static void main(String[] args) {
          System.out.println("Hello, World!");
          System.out.println("My name is [Your Name]");
          System.out.println("I am becoming a backend engineer.");
      }
  }

STEP 4: Run it
  Click green play button next to main method
  OR: Shift + F10
  See output in Run panel at bottom

STEP 5: Initialize Git (THE PROFESSIONAL PART)
  Open terminal in IntelliJ (Alt + F12)
  OR use WSL terminal:
  
  cd ~/projects/hello-world
  git init
  
STEP 6: Create .gitignore
  In IntelliJ: right-click project root → New → File
  Name: .gitignore
  
  Add this content:
  # Maven
  target/
  
  # IntelliJ
  .idea/
  *.iml
  *.iws
  *.ipr
  
  # Java
  *.class
  *.jar
  
  # OS
  .DS_Store
  Thumbs.db

STEP 7: First Commit
  git add .
  git status
  (see all files staged — should NOT include .idea or target)
  
  git commit -m "feat: initial hello world project"

STEP 8: Push to GitHub
  1. Go to GitHub.com
  2. Click "+" → New repository
  3. Name: hello-world
  4. Description: "My first Java project — learning backend engineering"
  5. Public (so employers can see it)
  6. Do NOT initialize with README (you already have local code)
  7. Create repository
  
  GitHub shows you commands. Use the SSH version:
  git remote add origin git@github.com:yourusername/hello-world.git
  git branch -M main
  git push -u origin main

STEP 9: Verify
  Refresh GitHub page
  You should see your code online
  
  Congratulations.
  You just did what professional engineers do every day.
```

---
## Common Problems & Solutions
```
PROBLEM: "java is not recognized as a command"
SOLUTION: JAVA_HOME not set or PATH missing %JAVA_HOME%\bin
  → Reinstall JDK, check both checkboxes
  → Or manually set environment variables
  → Restart terminal AFTER setting variables

PROBLEM: "git is not recognized as a command"
SOLUTION: Git not added to PATH
  → Reinstall Git, select "Git from command line and 3rd party"
  → Restart terminal

PROBLEM: "Permission denied (publickey)" when pushing to GitHub
SOLUTION: SSH key not set up correctly
  → Run ssh -T git@github.com to test
  → Re-add SSH key to GitHub
  → Make sure you're using SSH URL (git@github.com:...)
  → Not HTTPS URL (https://github.com/...)

PROBLEM: Docker Desktop won't start
SOLUTION: Virtualization not enabled
  → Restart computer → Enter BIOS (usually F2 or Delete key)
  → Find: Virtualization Technology → Enable
  → Save and exit
  → Or: WSL 2 not properly installed
  → Run: wsl --update in PowerShell as Administrator

PROBLEM: IntelliJ can't find JDK
SOLUTION: 
  → File → Project Structure → Project → SDK → Add SDK
  → Point to your JDK installation folder
  → Usually: C:\Program Files\Eclipse Adoptium\jdk-21...

PROBLEM: Maven downloads are slow
SOLUTION: Normal — first time downloads many dependencies
  → Let it finish
  → Future builds use cached dependencies (fast)
  → Don't interrupt the first download
```

---
## Key Takeaways
```
1. Your tools are your workstation.
   Know what each tool does and WHY it exists.
   Don't just install blindly.

2. JDK 21 = the engine that runs Java.
   IntelliJ = where you write it.
   Maven = what builds it.
   Git = what tracks it.
   GitHub = where you store and show it.
   Docker = how you run dependencies easily.

3. SSH key for GitHub = one-time setup that saves
   thousands of password prompts over your career.

4. .gitignore is not optional.
   Always set it up before first commit.
   Never commit .idea/, target/, or .class files.

5. Professional commit messages matter.
   "feat: add user login endpoint" is professional.
   "fixed stuff" is not.

6. Your GitHub profile = your real resume in this field.
   Start treating it like one from today.

7. The terminal is not scary.
   It is the most powerful tool you have.
   Learn 5 commands today. Use them. Learn 5 more next week.
```

---