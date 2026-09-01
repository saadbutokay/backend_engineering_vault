## Overview

Your Java applications will run on Linux servers in production and on macOS during development. Every deployment, every debugging session, and every performance investigation requires you to navigate the file system, manage processes, configure environment variables, and understand permissions. This section covers the operating system fundamentals you need before touching any Java tooling.

---

## Core Concepts

### The File System Hierarchy

Both macOS and Linux follow the Unix file system hierarchy. Everything starts at the root directory `/`.

```
/                   Root directory. The top of the tree.
├── /bin            Essential user binaries (ls, cp, mv, cat)
├── /sbin           Essential system binaries (fdisk, iptables, reboot)
├── /etc            System-wide configuration files
│   ├── /etc/hosts          Hostname-to-IP mappings
│   ├── /etc/passwd         User account information
│   ├── /etc/group          Group definitions
│   ├── /etc/ssh/           SSH configuration
│   └── /etc/environment    System-wide environment variables (Linux)
├── /home           User home directories (Linux)
│   └── /home/username/     Each user gets a directory here
├── /Users          User home directories (macOS)
│   └── /Users/username/    macOS equivalent of /home
├── /root           Home directory for the root (superuser) account
├── /usr            User-installed programs and libraries
│   ├── /usr/bin            Non-essential user binaries
│   ├── /usr/local          Locally compiled/installed software
│   │   ├── /usr/local/bin      Binaries from Homebrew (macOS Intel)
│   │   └── /usr/local/lib      Libraries
│   └── /usr/lib            System libraries
├── /var            Variable data that changes during operation
│   ├── /var/log            System and application log files
│   ├── /var/tmp            Temporary files preserved across reboots
│   └── /var/lib            Application state data (databases, etc.)
├── /tmp            Temporary files (cleared on reboot)
├── /opt            Optional/third-party software packages
├── /dev            Device files (disks, terminals, random number generators)
├── /proc           Virtual filesystem exposing process and kernel info (Linux)
├── /sys            Virtual filesystem exposing hardware info (Linux)
├── /mnt            Temporary mount points for external filesystems
├── /media          Removable media mount points (USB, CD)
└── /lib            Essential shared libraries
```

**macOS differences:**

- User home directories are under `/Users/`, not `/home/`.
- Homebrew on Apple Silicon installs to `/opt/homebrew/`, not `/usr/local/`.
- macOS does not have `/proc` or `/sys`.
- System Integrity Protection (SIP) restricts modifications to `/usr`, `/bin`, `/sbin`, and `/System`.

**Why this matters for backend engineering:**

- Your application logs will go to `/var/log/` or a subdirectory.
- Configuration files live in `/etc/` or within your application directory.
- Temporary files for uploads or processing go to `/tmp/`.
- Your JDK will be installed under `/usr/lib/jvm/` (Linux) or `/Library/Java/JavaVirtualMachines/` (macOS).

### Users, Groups, and Permissions

Unix systems are multi-user. Every file and process belongs to a user and a group.

**Users:**

- **root** (UID 0) — The superuser. Can do anything. Never run your Java application as root in production.
- **Regular users** (UID 1000+) — Your daily account. Limited permissions.
- **Service accounts** — Dedicated users for running services (e.g., `postgres`, `redis`, `tomcat`). Your Java app should run under a dedicated service account.

**Groups:**

- A user can belong to multiple groups.
- Groups control shared access to files and resources.
- View your groups: `groups` or `id`.

**Permissions:**

Every file and directory has three sets of permissions: **owner**, **group**, and **others**.

Each set has three flags: **read (r)**, **write (w)**, **execute (x)**.

```
-rwxr-xr--  1 john  developers  4096 Jan 15 10:30 deploy.sh
│││││││││
││││││││└─ Others: read only
│││││││└── Others: no write
││││││└─── Others: no execute
│││││└──── Group: read
││││└───── Group: no write
│││└────── Group: execute
││└─────── Owner: read
│└──────── Owner: write
└───────── Owner: execute
```

The leading character indicates the file type: `-` for regular file, `d` for directory, `l` for symbolic link.

**Numeric (octal) notation:**

Each permission set is a 3-bit number:

```
r = 4
w = 2
x = 1
- = 0
```

| Octal | Binary | Permissions |
|-------|--------|-------------|
| 7     | 111    | rwx         |
| 6     | 110    | rw-         |
| 5     | 101    | r-x         |
| 4     | 100    | r--         |
| 0     | 000    | ---         |

Common permission combinations:

- `755` — Owner: rwx, Group: r-x, Others: r-x (directories, executables)
- `644` — Owner: rw-, Group: r--, Others: r-- (regular files)
- `600` — Owner: rw-, Group: ---, Others: --- (private files, SSH keys)
- `700` — Owner: rwx, Group: ---, Others: --- (private directories)

**Special note on directories:** Execute permission on a directory means you can enter it (`cd`) and access files within it. Without execute, you cannot traverse the directory even if you have read permission.

### Environment Variables

Environment variables are key-value pairs available to all processes in a session. They configure behavior without modifying code.

**Common environment variables:**

| Variable | Purpose | Example |
|----------|---------|---------|
| `HOME` | Current user's home directory | `/home/john` or `/Users/john` |
| `USER` | Current username | `john` |
| `SHELL` | Current shell | `/bin/zsh` |
| `PWD` | Current working directory | `/home/john/projects` |
| `LANG` | Locale and encoding | `en_US.UTF-8` |
| `PATH` | Directories to search for executables | `/usr/local/bin:/usr/bin:/bin` |
| `JAVA_HOME` | JDK installation directory | `/usr/lib/jvm/java-21` |
| `MAVEN_HOME` | Maven installation directory | `/opt/maven` |

**Setting environment variables:**

```bash
# Temporary (current session only)
export MY_VAR="hello"
echo $MY_VAR

# Permanent (add to shell config)
# macOS (Zsh):
echo 'export MY_VAR="hello"' >> ~/.zshrc
source ~/.zshrc

# Linux (Bash):
echo 'export MY_VAR="hello"' >> ~/.bashrc
source ~/.bashrc
```

**Viewing all environment variables:**

```bash
env
# or
printenv
```

**Why this matters for Java:**

Your Spring Boot applications read configuration from environment variables. For example, `SPRING_DATASOURCE_URL` overrides the database URL in `application.yml`. This is how you keep secrets out of source code and configure different behavior per environment (dev, staging, production).

### PATH and Command Resolution

When you type a command like `java` or `mvn`, the shell does not know where the executable lives. It searches each directory listed in the `PATH` variable, left to right, and runs the first match.

```bash
echo $PATH
# /opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

**Resolution process:**

1. You type `java` and press Enter.
2. The shell checks if `java` is a shell alias or function. If yes, runs it.
3. The shell checks if `java` is a shell builtin (like `cd` or `echo`). If yes, runs it.
4. The shell searches each directory in `PATH` in order:
    - Is there an executable file at `/opt/homebrew/bin/java`? No.
    - Is there one at `/usr/local/bin/java`? No.
    - Is there one at `/usr/bin/java`? Yes. Run it.
5. If no match is found, the shell prints `command not found`.

**Diagnosing which executable is being used:**

```bash
which java
# /usr/bin/java

# More detailed (shows all matches):
where java
# /usr/bin/java
# /Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home/bin/java

# Type information:
type java
# java is /usr/bin/java
```

**Modifying PATH:**

```bash
# Prepend a directory (searched first)
export PATH="/usr/lib/jvm/java-21/bin:$PATH"

# Append a directory (searched last)
export PATH="$PATH:/opt/myapp/bin"
```

Prepending is important when you have multiple versions of a tool installed. The first match wins.

### JAVA_HOME

`JAVA_HOME` is a convention, not a system requirement. Many Java tools (Maven, Gradle, Tomcat, IDEs) look for this variable to locate the JDK.

**Setting JAVA_HOME:**

```bash
# macOS (find the JDK path first)
/usr/libexec/java_home -V
# Matching Java Virtual Machines (2):
#     21.0.2 (arm64) "Eclipse Adoptium" - "OpenJDK 21.0.2" /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
#     17.0.9 (arm64) "Eclipse Adoptium" - "OpenJDK 17.0.9" /Library/Java/JavaVirtualMachines/temurin-17.jdk/Contents/Home

export JAVA_HOME=$(/usr/libexec/java_home -v 21)

# Linux
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
```

**Add to your shell config permanently:**

```bash
# ~/.zshrc (macOS) or ~/.bashrc (Linux)
export JAVA_HOME=$(/usr/libexec/java_home -v 21)  # macOS
# export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64  # Linux
export PATH="$JAVA_HOME/bin:$PATH"
```

**Verify:**

```bash
echo $JAVA_HOME
# /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home

java -version
# openjdk version "21.0.2" 2024-01-16 LTS
```

If `JAVA_HOME` is not set or points to the wrong version, Maven and Gradle will fail or use an unexpected JDK. This is one of the most common setup issues for new Java developers.

### Process Management

A **process** is a running instance of a program. Your JVM is a process. Your database is a process. Every `curl` command you run spawns a process.

**Viewing processes:**

```bash
# List all running processes
ps aux

# Find a specific process
ps aux | grep java

# Output columns:
# USER  PID  %CPU  %MEM  VSZ  RSS  TTY  STAT  START  TIME  COMMAND
# john  4521  2.3   8.1   ...  ...  ...  S     10:00  1:23  java -jar app.jar
```

Key columns:
- **PID** — Process ID (unique identifier)
- **%CPU** — CPU usage percentage
- **%MEM** — Memory usage percentage
- **STAT** — Process state (S=sleeping, R=running, Z=zombie, T=stopped)

**Interactive process monitoring:**

```bash
# macOS and Linux
top

# Linux (more detailed, preferred)
htop

# macOS (if installed via Homebrew)
htop
```

**Killing processes:**

```bash
# Graceful termination (SIGTERM — process can clean up)
kill 4521

# Force kill (SIGKILL — immediate, no cleanup)
kill -9 4521

# Kill by name
pkill -f "java -jar app.jar"

# Kill all Java processes (use with caution)
killall java
```

**Signal types you should know:**

| Signal | Number | Effect |
|--------|--------|--------|
| SIGTERM | 15 | Graceful shutdown. The JVM runs shutdown hooks. |
| SIGKILL | 9 | Immediate termination. No cleanup. Data loss possible. |
| SIGINT | 2 | Interrupt (Ctrl+C). JVM runs shutdown hooks. |
| SIGHUP | 1 | Hangup. Often used to reload configuration. |

**Why this matters:** When you deploy a Spring Boot application, Kubernetes sends SIGTERM to stop it gracefully. Your application must handle this signal to finish in-flight requests and close database connections. Spring Boot handles this by default with `server.shutdown=graceful`.

### Package Managers

Package managers install, update, and remove software.

**macOS — Homebrew:**

```bash
# Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install a package
brew install git
brew install maven
brew install postgresql@16
brew install redis

# Search for a package
brew search java

# Update all packages
brew update && brew upgrade

# List installed packages
brew list

# Remove a package
brew uninstall maven

# View package info
brew info postgresql@16
```

**Linux (Debian/Ubuntu) — apt:**

```bash
# Update package index
sudo apt update

# Upgrade installed packages
sudo apt upgrade

# Install a package
sudo apt install git
sudo apt install maven
sudo apt install postgresql
sudo apt install redis-server

# Search for a package
apt search openjdk

# Remove a package
sudo apt remove maven

# Remove a package and its config files
sudo apt purge maven

# Clean up unused dependencies
sudo apt autoremove
```

**Linux (RHEL/CentOS/Fedora) — dnf/yum:**

```bash
sudo dnf install java-21-openjdk-devel
sudo dnf install maven
```

### SSH Basics

**SSH (Secure Shell)** is a protocol for securely connecting to remote machines. You will use SSH to access production servers, deploy applications, and clone Git repositories.

**Generating an SSH key pair:**

```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
# Creates:
#   ~/.ssh/id_ed25519      (private key — NEVER share this)
#   ~/.ssh/id_ed25519.pub  (public key — share this with servers/GitHub)
```

**Connecting to a remote server:**

```bash
ssh username@server-ip
ssh -p 2222 username@server-ip   # Custom port
ssh -i ~/.ssh/my_key username@server-ip  # Specific key
```

**Copying your public key to a server:**

```bash
ssh-copy-id username@server-ip
# Now you can log in without a password
```

**SSH config file (~/.ssh/config):**

```
Host production
    HostName 10.0.1.50
    User deploy
    Port 22
    IdentityFile ~/.ssh/production_key
    ForwardAgent yes

Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
```

With this config, you can simply type `ssh production` instead of the full command.

**SCP (secure file copy):**

```bash
# Copy local file to remote server
scp app.jar deploy@production:/opt/app/

# Copy remote file to local
scp deploy@production:/var/log/app.log ./logs/
```

---

## Code Examples

**Checking your system information:**

```bash
# OS version (macOS)
sw_vers
# ProductName:    macOS
# ProductVersion: 14.3
# BuildVersion:   23D56

# OS version (Linux)
cat /etc/os-release
# NAME="Ubuntu"
# VERSION="22.04.3 LTS (Jammy Jellyfish)"

# CPU info
sysctl -n machdep.cpu.brand_string   # macOS
lscpu                               # Linux

# Memory
sysctl hw.memsize                    # macOS (bytes)
free -h                              # Linux
```

**Working with file permissions:**

```bash
# View permissions
ls -la deploy.sh
# -rwxr-xr--  1 john  developers  2048 Jan 15 10:30 deploy.sh

# Change permissions (make executable for owner only)
chmod 700 deploy.sh

# Change permissions (read/write for owner, read for group and others)
chmod 644 config.yml

# Change permissions recursively (directory and all contents)
chmod -R 755 /opt/myapp/

# Change owner
sudo chown deploy:deploy /opt/myapp/

# Change owner recursively
sudo chown -R deploy:deploy /opt/myapp/
```

**Managing environment variables in a Java context:**

```bash
# Set multiple variables for a Spring Boot application
export SPRING_PROFILES_ACTIVE=production
export SPRING_DATASOURCE_URL=jdbc:postgresql://db.example.com:5432/mydb
export SPRING_DATASOURCE_USERNAME=app_user
export SPRING_DATASOURCE_PASSWORD=secret123
export JAVA_OPTS="-Xms512m -Xmx2g -XX:+UseG1GC"

# Run the application with these variables
java $JAVA_OPTS -jar myapp.jar

# Or pass variables inline (without exporting)
SPRING_PROFILES_ACTIVE=staging java -jar myapp.jar
```

**Process management for a Java application:**

```bash
# Start a Spring Boot app in the background
nohup java -jar myapp.jar > /var/log/myapp.log 2>&1 &

# Find the process
ps aux | grep myapp
# john  12345  3.2  12.5  ...  java -jar myapp.jar

# Check what ports it is using
lsof -i :8080
# COMMAND   PID  USER  FD  TYPE  DEVICE  SIZE/OFF  NODE  NAME
# java    12345  john  12u  IPv6  0x...   0t0       TCP   *:8080 (LISTEN)

# Graceful shutdown
kill 12345

# Verify it stopped
ps aux | grep myapp
```

---

## Important Notes

- Never run your Java application as root. Create a dedicated service account with minimal permissions. If the application is compromised, the attacker gains only the permissions of that account.
- The `/tmp/` directory is world-writable. Any user can read files placed there. Never store sensitive data in `/tmp/`. Use it only for short-lived, non-sensitive temporary files.
- On macOS, Homebrew on Apple Silicon (M1/M2/M3) installs to `/opt/homebrew/`. On Intel Macs, it installs to `/usr/local/`. This difference causes frequent PATH issues. Always verify with `brew --prefix`.
- The `JAVA_HOME` variable must point to the JDK root directory, not the `bin` subdirectory. The `bin` directory is `$JAVA_HOME/bin`. Setting `JAVA_HOME` to the `bin` directory is a common mistake that breaks Maven and Gradle.
- When you kill a Java process with `kill -9`, the JVM cannot run shutdown hooks. This means database connections may not be closed cleanly, in-memory caches are lost, and temporary files may not be cleaned up. Always try `kill` (SIGTERM) first and wait a few seconds before resorting to `kill -9`.
- SSH keys are more secure than passwords. Disable password authentication on production servers and use key-based authentication exclusively. This is a standard security practice in fintech environments.
- File permissions are the first line of defense for sensitive configuration files. Your `.env` files, database credentials, and private keys should always be `600` (owner read/write only).

---

## Practice

1. Open your terminal. Run `ls -la /` and identify every top-level directory. Write down what each one contains in your Obsidian vault.
2. Run `echo $PATH` and split the output by `:`. For each directory, run `ls` to see what executables live there. Note which directories contain Java-related binaries.
3. Run `echo $JAVA_HOME`. If it is empty, set it correctly following the instructions above. Add it to your shell config file and verify it persists after opening a new terminal.
4. Create a file called `test.sh` with the content `#!/bin/bash\necho "Hello"`. Try to run it with `./test.sh`. It will fail. Fix the permissions with `chmod` and run it again.
5. Run `ps aux | grep java`. If no Java processes are running, start one (even the HelloWorld from 00.01) in the background and find it with `ps`. Kill it gracefully with `kill`.
6. Generate an SSH key pair with `ssh-keygen -t ed25519`. View the public key with `cat ~/.ssh/id_ed25519.pub`. You will use this key when setting up GitHub in Phase 02.

---

## References

- Linux File System Hierarchy: https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html
- macOS File System: https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html
- SSH Manual: `man ssh` and `man ssh-keygen`
- Homebrew Documentation: https://docs.brew.sh/
- Ubuntu Package Management: https://ubuntu.com/server/docs/package-management
