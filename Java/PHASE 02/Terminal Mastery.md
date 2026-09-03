## Overview

The terminal is the primary interface between you and your development environment. While IDEs like IntelliJ handle code editing and debugging, the terminal is where you build projects, run tests, manage processes, inspect logs, deploy services, administer databases, and automate repetitive tasks. A Java backend engineer who is slow in the terminal is slow everywhere—because every tool in the ecosystem (Maven, Gradle, Docker, Git, SSH, curl, kubectl) is a command-line tool first and a GUI second.

This note covers the Unix shell from the ground up: what the terminal actually is, how the shell interprets your commands, how to navigate and manipulate the filesystem, how to chain commands together with pipes and redirection, how to search and transform text, how to write shell scripts, and how to manage your terminal environment for long-term productivity.

The target shell is Zsh (the default on macOS since Catalina) with notes on Bash differences. Both are POSIX-compatible and share 95% of the syntax you will use daily. If you are on Linux, you likely default to Bash; the differences are minor and noted where relevant.

---

## Core Concepts

### Shell vs Terminal vs Console

These terms are often used interchangeably but mean different things:

```
Terminal (Terminal Emulator):
  → The graphical window that displays text and accepts keyboard input
  → Examples: iTerm2, macOS Terminal, GNOME Terminal, Windows Terminal, Alacritty, Kitty
  → It does NOT interpret commands — it just renders characters and sends keystrokes

Shell:
  → The program that runs INSIDE the terminal and interprets your commands
  → Examples: bash, zsh, fish, sh, dash
  → It reads your input, parses it, finds the executable, and runs it
  → It provides programming constructs: variables, loops, conditionals, functions

Console:
  → Historically, the physical hardware terminal connected to a mainframe
  → Today, used loosely to mean "the terminal" or "the shell"
  → "System console" = the primary text output of the OS (e.g., /dev/console)

Relationship:
  Terminal Emulator (iTerm2)
    └── Shell (zsh)
          ├── Command: mvn clean install
          ├── Command: docker ps
          └── Command: grep ERROR app.log
```

### Bash vs Zsh

```
Bash (Bourne Again Shell):
  → Default on most Linux distributions
  → Mature, ubiquitous, POSIX-compliant
  → Configuration: ~/.bashrc, ~/.bash_profile
  → Globbing: * matches files, but no ** for recursive by default
  → Array indexing: 0-based

Zsh (Z Shell):
  → Default on macOS since 10.15 Catalina
  → Backward-compatible with Bash (most Bash scripts work in Zsh)
  → Configuration: ~/.zshrc
  → Superior features:
      - Recursive globbing: **/*.java finds all Java files in all subdirectories
      - Better tab completion (completes flags, paths, git branches)
      - Spelling correction
      - Shared history across terminal sessions
      - Array indexing: 1-based (the one gotcha for Bash users)
  → Plugin ecosystem: Oh My Zsh, zinit, antigen

Key differences you will encounter:
  - Bash: echo ${array[0]}    → first element
  - Zsh:  echo ${array[1]}    → first element (1-indexed!)
  - Bash: shopt -s globstar   → enable ** globbing
  - Zsh:  ** works by default
  - Bash: [ ] for conditionals
  - Zsh:  [[ ]] preferred (also works in modern Bash)
```

### The Command Execution Model

When you type a command and press Enter, the shell does this:

```
1. Parse the input line (split into tokens by whitespace, handle quotes and escapes)
2. Expand variables ($HOME → /Users/alice)
3. Expand globs (*.java → Account.java Payment.java Transfer.java)
4. Expand command substitution ($(date) → Wed Jan 15 14:30:00 EST 2025)
5. Check if the command is a shell builtin (cd, echo, export, alias)
6. If not builtin, search $PATH for an executable file
7. Fork a child process and exec the command
8. Wait for the command to finish (unless backgrounded with &)
9. Capture the exit code ($?)
```

### Exit Codes

Every command returns an integer exit code when it finishes:

```
0       → success
1-255   → failure (specific meaning varies by command)
1       → general error
2       → misuse of shell builtin
126     → command found but not executable
127     → command not found
128+N   → process killed by signal N (e.g., 137 = killed by SIGKILL/9)
130     → process killed by Ctrl+C (SIGINT, signal 2 → 128+2)

Access the last exit code: echo $?
```

### Environment Variables and PATH

```
Environment variables:
  → Key-value pairs inherited by child processes
  → Set with: export KEY=value
  → Access with: $KEY or ${KEY}
  → List all: env or printenv

PATH:
  → A colon-separated list of directories the shell searches for executables
  → When you type "java", the shell searches each directory in $PATH
    in order until it finds an executable named "java"
  → View: echo $PATH
  → Modify: export PATH="/new/dir:$PATH" (prepend) or export PATH="$PATH:/new/dir" (append)
  → JAVA_HOME: the root directory of your JDK installation
    export JAVA_HOME="$HOME/.sdkman/candidates/java/current"
    export PATH="$JAVA_HOME/bin:$PATH"

Other critical variables for Java developers:
  JAVA_HOME    → JDK installation root
  MAVEN_HOME   → Maven installation root (if not using wrapper)
  GRADLE_HOME  → Gradle installation root (if not using wrapper)
  CLASSPATH    → where the JVM looks for .class files (rarely set manually now)
  SDKMAN_DIR   → SDKMAN installation directory
```

### File System Hierarchy (Unix)

```
/                  → root directory (the top of everything)
├── /bin           → essential user binaries (ls, cp, mv, cat)
├── /sbin          → system administration binaries (fdisk, iptables)
├── /usr           → user programs and data
│   ├── /usr/bin   → non-essential user binaries
│   ├── /usr/local → locally installed software (brew installs here on macOS)
│   └── /usr/lib   → shared libraries
├── /etc           → system configuration files
├── /var           → variable data (logs, caches, spools)
│   └── /var/log   → system log files
├── /tmp           → temporary files (cleared on reboot)
├── /home          → user home directories (Linux: /home/alice)
├── /Users         → user home directories (macOS: /Users/alice)
├── /opt           → optional/third-party software
├── /dev           → device files (disks, terminals, /dev/null)
├── /proc          → process information (Linux: /proc/1234/status)
└── /Library       → macOS system-wide resources
```

### File Permissions

```
Every file and directory has three permission sets: owner, group, others

  rwx r-x r--
  │││ │││ │││
  │││ │││ └└└─ others: read only
  │││ └└└───── group: read + execute
  └└└───────── owner: read + write + execute

  r = read    (4)  → view file contents / list directory
  w = write   (2)  → modify file / create/delete files in directory
  x = execute (1)  → run file as program / enter directory

Numeric notation:
  755 = rwxr-xr-x (owner: full, group: read+exec, others: read+exec)
  644 = rw-r--r-- (owner: read+write, group: read, others: read)
  700 = rwx------ (owner: full, everyone else: nothing)
  600 = rw------- (owner: read+write, everyone else: nothing — for private keys)

Commands:
  chmod 755 script.sh     → set permissions
  chmod +x script.sh      → add execute permission
  chmod -w file.txt       → remove write permission
  chmod -R 644 src/       → recursive
  chown alice:dev file    → change owner and group
  ls -la                  → view permissions
```

---

## Code Examples

### Navigation

```bash
# Print current directory
pwd
# /Users/alice/projects/banking-api

# List directory contents
ls                    # basic listing
ls -la                # long format, all files (including hidden .dotfiles)
ls -lh                # human-readable sizes (KB, MB, GB)
ls -lt                # sort by modification time (newest first)
ls -lS                # sort by size (largest first)
ls -lR                # recursive listing
ls src/main/java/com/example/**/*.java  # Zsh: recursive glob for all Java files

# Change directory
cd /Users/alice/projects    # absolute path
cd projects/banking-api     # relative path
cd ..                       # parent directory
cd ../..                    # two levels up
cd ~                        # home directory
cd -                        # previous directory (toggle)
cd                          # same as cd ~

# Directory stack (pushd/popd)
pushd /var/log              # push current dir onto stack, cd to /var/log
pushd /etc                  # push again
dirs                        # show stack: /etc /var/log ~/projects
popd                        # return to /var/log
popd                        # return to ~/projects
```

### File Operations

```bash
# Create
touch file.txt              # create empty file (or update timestamp)
mkdir newdir                # create directory
mkdir -p src/main/java      # create nested directories (parents as needed)

# Copy
cp file.txt backup.txt      # copy file
cp -r src/ backup-src/      # copy directory recursively
cp -i file.txt dest/        # interactive: prompt before overwriting

# Move / Rename
mv old.txt new.txt          # rename
mv file.txt /tmp/           # move to different directory
mv -i file.txt dest/        # interactive

# Delete
rm file.txt                 # delete file (PERMANENT — no trash)
rm -r directory/            # delete directory recursively
rm -rf directory/           # force delete (no prompts, no errors for missing files)
# ⚠️  rm -rf / is the most dangerous command in Unix. Never run it.
# ⚠️  Always double-check before pressing Enter on rm -rf

# Links
ln -s /path/to/target linkname   # symbolic link (shortcut)
ln target linkname               # hard link (same inode)
ls -la linkname                  # shows -> /path/to/target

# File information
file document.pdf           # identify file type
stat file.txt               # detailed metadata (size, permissions, timestamps)
du -sh *                    # disk usage of each item in current directory
du -sh src/                 # total size of src directory
df -h                       # disk free space on all mounted filesystems

# Find files
find . -name "*.java"                    # all Java files in current tree
find . -name "*.java" -newer pom.xml     # Java files modified after pom.xml
find . -type f -size +10M                # files larger than 10MB
find . -type d -name "target"            # all directories named "target"
find . -name "*.class" -delete           # delete all .class files
find . -name "*.log" -mtime +7 -delete   # delete log files older than 7 days
```

### Reading and Viewing Files

```bash
# Display entire file
cat file.txt                # concatenate and print (best for short files)
cat -n file.txt             # with line numbers

# View file with paging (for long files)
less file.txt               # scroll with arrows, search with /, quit with q
more file.txt               # simpler pager (space to advance, q to quit)

# View beginning / end
head -20 file.txt           # first 20 lines (default: 10)
tail -20 file.txt           # last 20 lines
tail -f application.log     # follow: print new lines as they are appended
                              # CRITICAL for watching live logs
tail -f app.log | grep ERROR  # follow and filter for errors

# Word / line / character count
wc -l file.txt              # line count
wc -w file.txt              # word count
wc -c file.txt              # byte count
wc -l src/**/*.java         # Zsh: count lines in all Java files

# Compare files
diff file1.txt file2.txt    # show differences
diff -u file1.txt file2.txt # unified diff format (used in patches)
diff -r dir1/ dir2/         # recursive directory comparison
colordiff file1 file2       # colored diff (if installed)
```

### Pipes and Redirection

Pipes and redirection are the superpower of the Unix shell. They let you chain small, single-purpose tools into powerful data processing pipelines.

```bash
# PIPE (|): send stdout of one command as stdin to the next
cat access.log | grep "ERROR" | wc -l
# "Count the number of ERROR lines in the access log"
# (cat is unnecessary here — grep "ERROR" access.log | wc -l is better)

# Java-specific pipeline examples:
mvn test 2>&1 | grep "Tests run:"
# Run tests, redirect stderr to stdout, filter for test summary lines

ps aux | grep java | grep -v grep
# List all running Java processes (grep -v grep excludes the grep itself)

find . -name "*.java" | xargs wc -l | tail -1
# Count total lines of Java code in the project

docker ps | grep banking-api | awk '{print $1}'
# Get the container ID of the banking-api container

# REDIRECTION:
# >   redirect stdout to file (OVERWRITE)
# >>  redirect stdout to file (APPEND)
# 2>  redirect stderr to file
# 2>&1 redirect stderr to stdout
# &>  redirect both stdout and stderr to file (Bash 4+ / Zsh)
# <   redirect file to stdin

mvn clean install > build.log 2>&1
# Run build, send ALL output (stdout + stderr) to build.log

echo "JAVA_HOME=$JAVA_HOME" >> ~/.zshrc
# Append a line to your shell config

sort < unsorted.txt > sorted.txt
# Read from file, sort, write to file

# /dev/null: the black hole (discard output)
mvn test > /dev/null 2>&1
# Run tests silently (discard all output — useful in scripts)

# /dev/zero and /dev/random: infinite data sources
head -c 100 /dev/random | base64
# Generate 100 random bytes, base64 encode them
```

### grep — Text Search

`grep` is the most-used text search tool. You will use it dozens of times per day.

```bash
# Basic search
grep "ERROR" application.log          # find lines containing "ERROR"
grep -i "error" application.log       # case-insensitive
grep -r "NullPointerException" src/   # recursive search in directory
grep -n "TODO" src/**/*.java          # with line numbers (Zsh glob)
grep -c "WARN" application.log        # count matching lines
grep -v "DEBUG" application.log       # invert: show lines NOT matching
grep -l "import java.sql" src/**/*.java  # list filenames only (which files match)
grep -L "import java.sql" src/**/*.java  # list files that do NOT match

# Context lines
grep -A 5 "Exception" app.log         # 5 lines After each match
grep -B 3 "Exception" app.log         # 3 lines Before each match
grep -C 3 "Exception" app.log         # 3 lines of Context (before + after)

# Extended regex (grep -E or egrep)
grep -E "(ERROR|FATAL)" app.log       # match ERROR or FATAL
grep -E "^\d{4}-\d{2}-\d{2}" app.log # lines starting with a date

# Fixed string (no regex interpretation — faster for literal searches)
grep -F "java.lang.NullPointerException" app.log

# Search in compressed files
zgrep "ERROR" application.log.gz

# Java developer grep recipes:
grep -rn "TODO\|FIXME\|HACK" src/     # find all code debt markers
grep -rn "System.out.println" src/     # find debug prints that should be logging
grep -rn "catch.*Exception.*{" src/ | grep -v "log\."  # swallowed exceptions
grep -rn "@Deprecated" src/            # find deprecated code
```

### sed — Stream Editor

`sed` performs find-and-replace and text transformation on streams:

```bash
# Basic substitution: s/pattern/replacement/flags
sed 's/ERROR/WARN/' app.log           # replace first "ERROR" per line with "WARN"
sed 's/ERROR/WARN/g' app.log          # replace ALL "ERROR" per line (g = global)
sed 's/foo/bar/gi' file.txt           # global + case-insensitive

# In-place editing (modify the file directly)
sed -i '' 's/old/new/g' file.txt      # macOS (requires '' after -i)
sed -i 's/old/new/g' file.txt         # Linux (no '' needed)
# ⚠️  Always test without -i first, or backup: sed -i '.bak' 's/old/new/g' file

# Delete lines
sed '/DEBUG/d' app.log                # delete all lines containing "DEBUG"
sed '5d' file.txt                     # delete line 5
sed '1,10d' file.txt                  # delete lines 1-10
sed '/^$/d' file.txt                  # delete blank lines

# Print specific lines
sed -n '5,10p' file.txt               # print lines 5-10 only
sed -n '/ERROR/p' app.log             # print only ERROR lines (like grep)

# Insert and append
sed '3i\NEW LINE HERE' file.txt       # insert before line 3
sed '3a\NEW LINE HERE' file.txt       # append after line 3

# Java developer sed recipes:
# Replace all occurrences of a class name across all Java files
find src/ -name "*.java" -exec sed -i '' 's/OldService/NewService/g' {} +

# Remove all trailing whitespace
sed -i '' 's/[[:space:]]*$//' file.java

# Comment out a line in a config file
sed -i '' 's/^server.port=8080/#server.port=8080/' application.properties

# Extract version from pom.xml
sed -n 's/.*<version>\(.*\)<\/version>.*/\1/p' pom.xml | head -1
```

### awk — Pattern Scanning and Processing

`awk` is a mini programming language for processing structured text (columns, fields):

```bash
# Default: split each line by whitespace into $1, $2, $3, ...
# $0 = entire line, NF = number of fields, NR = line number

# Print specific columns
ps aux | awk '{print $1, $2, $11}'
# USER PID COMMAND

# Filter and format
ps aux | awk '$3 > 50.0 {print $2, $11, $3"%"}'
# Print PID, command, and CPU% for processes using >50% CPU

# Custom field separator
awk -F',' '{print $1, $3}' data.csv
# Print columns 1 and 3 from a CSV file

awk -F':' '{print $1}' /etc/passwd
# Print all usernames from the passwd file

# Sum a column
awk -F',' '{sum += $3} END {print "Total:", sum}' transactions.csv
# Sum the 3rd column (e.g., transaction amounts)

# Count occurrences
awk '{count[$1]++} END {for (k in count) print k, count[k]}' access.log
# Count requests per IP address

# Java developer awk recipes:
# Parse Maven test output for failure counts
mvn test 2>&1 | awk '/Tests run:/ {print $0}'

# Extract process IDs of all Java processes
ps aux | awk '/[j]ava/ {print $2}'

# Calculate average response time from a log
awk '/response_time/ {sum += $NF; count++} END {print "Avg:", sum/count, "ms"}' app.log

# Format a list of JAR files with sizes
ls -lh target/*.jar | awk '{printf "%-40s %s\n", $NF, $5}'
```

### Shell Scripting Basics

Shell scripts automate sequences of commands. Every script is a file with executable permissions and a shebang line:

```bash
#!/bin/zsh
# ^^^ shebang: tells the OS which interpreter to use

# --- Variables ---
NAME="Alice"
GREETING="Hello, $NAME"
echo $GREETING              # Hello, Alice
echo "Project: ${PROJECT_NAME:-default}"  # default value if unset

# Command substitution
CURRENT_DATE=$(date +%Y-%m-%d)
JAVA_VERSION=$(java -version 2>&1 | head -1)
echo "Date: $CURRENT_DATE, Java: $JAVA_VERSION"

# --- Conditionals ---
if [ -f "pom.xml" ]; then
    echo "Maven project detected"
elif [ -f "build.gradle" ]; then
    echo "Gradle project detected"
else
    echo "Unknown project type"
fi

# File test operators:
# -f file    → file exists and is a regular file
# -d dir     → directory exists
# -e path    → path exists (any type)
# -r file    → file is readable
# -w file    → file is writable
# -x file    → file is executable
# -s file    → file is non-empty
# -z string  → string is empty
# -n string  → string is non-empty

# String comparison
if [ "$ENV" = "production" ]; then
    echo "Running in production mode"
fi

# Numeric comparison
if [ "$EXIT_CODE" -ne 0 ]; then
    echo "Build failed with exit code $EXIT_CODE"
fi
# -eq (equal), -ne (not equal), -lt (less than), -gt (greater than)
# -le (less or equal), -ge (greater or equal)

# --- Loops ---
# For loop
for file in src/main/java/com/example/*.java; do
    echo "Checking $file"
    grep -l "TODO" "$file"
done

# While loop (read lines from a file)
while IFS= read -r line; do
    echo "Processing: $line"
done < dependencies.txt

# C-style for loop (Bash/Zsh)
for ((i=1; i<=10; i++)); do
    echo "Iteration $i"
done

# --- Functions ---
build_project() {
    local project_dir=$1
    local profile=${2:-dev}
    echo "Building $project_dir with profile $profile"
    cd "$project_dir" && mvn clean install -P"$profile"
}

build_project "/path/to/api" "staging"

# --- Error handling ---
set -e    # exit immediately if any command fails
set -u    # treat unset variables as errors
set -o pipefail  # pipeline fails if any command in the pipe fails
# Use all three at the top of production scripts:
set -euo pipefail

# Trap: run cleanup on exit
cleanup() {
    echo "Cleaning up temp files..."
    rm -rf "$TEMP_DIR"
}
trap cleanup EXIT

# --- Exit codes ---
if ! mvn test; then
    echo "Tests failed!" >&2   # write to stderr
    exit 1
fi
echo "All tests passed"
exit 0
```

### Practical Shell Script: Build and Deploy

```bash
#!/bin/zsh
set -euo pipefail

# --- Configuration ---
PROJECT_NAME="banking-api"
DOCKER_REGISTRY="123456789.dkr.ecr.us-east-1.amazonaws.com"
IMAGE_TAG=$(git rev-parse --short HEAD)
ENVIRONMENT=${1:-dev}

echo "=== Building $PROJECT_NAME ==="
echo "Environment: $ENVIRONMENT"
echo "Image tag: $IMAGE_TAG"

# --- Build ---
echo ">> Running tests..."
./mvnw clean verify -P"$ENVIRONMENT"

# --- Docker ---
echo ">> Building Docker image..."
docker build -t "$DOCKER_REGISTRY/$PROJECT_NAME:$IMAGE_TAG" .
docker tag "$DOCKER_REGISTRY/$PROJECT_NAME:$IMAGE_TAG" \
           "$DOCKER_REGISTRY/$PROJECT_NAME:latest"

# --- Deploy (only if not dev) ---
if [ "$ENVIRONMENT" != "dev" ]; then
    echo ">> Pushing to registry..."
    docker push "$DOCKER_REGISTRY/$PROJECT_NAME:$IMAGE_TAG"
    docker push "$DOCKER_REGISTRY/$PROJECT_NAME:latest"

    echo ">> Deploying to $ENVIRONMENT..."
    kubectl set image deployment/$PROJECT_NAME \
        app="$DOCKER_REGISTRY/$PROJECT_NAME:$IMAGE_TAG" \
        --namespace="$ENVIRONMENT"
    kubectl rollout status deployment/$PROJECT_NAME --namespace="$ENVIRONMENT"
fi

echo "=== Done ==="
```

### curl and wget

```bash
# curl: transfer data from/to a URL (more versatile, default on macOS)

# Basic GET request
curl https://api.example.com/health
curl -s https://api.example.com/health     # silent (no progress bar)
curl -v https://api.example.com/health     # verbose (show headers)

# Save response to file
curl -o response.json https://api.example.com/users
curl -O https://example.com/file.tar.gz    # save with original filename

# HTTP methods
curl -X POST https://api.example.com/users \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $TOKEN" \
     -d '{"name": "Alice", "email": "alice@example.com"}'

curl -X PUT https://api.example.com/users/1 \
     -H "Content-Type: application/json" \
     -d '{"name": "Alice Updated"}'

curl -X DELETE https://api.example.com/users/1

# Read request body from file
curl -X POST https://api.example.com/transactions \
     -H "Content-Type: application/json" \
     -d @request-body.json

# Follow redirects
curl -L https://example.com/redirect

# Show only response headers
curl -I https://api.example.com/health

# Show timing information
curl -o /dev/null -s -w "DNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTotal: %{time_total}s\nHTTP Code: %{http_code}\n" \
     https://api.example.com/health

# Send form data
curl -X POST https://api.example.com/login \
     -d "username=alice&password=secret"

# Upload a file
curl -X POST https://api.example.com/upload \
     -F "file=@document.pdf"

# Java developer curl recipes:
# Test a Spring Boot actuator endpoint
curl -s http://localhost:8080/actuator/health | jq .

# Test a JWT-protected endpoint
TOKEN=$(curl -s -X POST http://localhost:8080/api/auth/login \
    -H "Content-Type: application/json" \
    -d '{"username":"admin","password":"secret"}' | jq -r '.token')
curl -s http://localhost:8080/api/accounts \
    -H "Authorization: Bearer $TOKEN" | jq .

# wget: simpler file downloader (default on most Linux)
wget https://example.com/file.tar.gz
wget -q https://example.com/file.tar.gz    # quiet
wget -r -np https://example.com/docs/      # recursive, no parent
wget -O output.html https://example.com    # save as specific filename
```

### tmux — Terminal Multiplexer

`tmux` lets you run multiple terminal sessions inside a single window, detach from them (they keep running), and reattach later. Essential for SSH sessions to remote servers.

```bash
# Installation
brew install tmux          # macOS
sudo apt install tmux      # Ubuntu/Debian

# Session management
tmux                       # start a new session
tmux new -s banking-api    # start named session
tmux ls                    # list all sessions
tmux attach -t banking-api # reattach to a session
tmux kill-session -t banking-api  # kill a session

# Inside tmux (prefix key is Ctrl+b by default, then release and press the command):
# Ctrl+b c     → create new window
# Ctrl+b n     → next window
# Ctrl+b p     → previous window
# Ctrl+b 0-9   → switch to window 0-9
# Ctrl+b %     → split pane vertically
# Ctrl+b "     → split pane horizontally
# Ctrl+b o     → switch between panes
# Ctrl+b d     → detach (session keeps running in background)
# Ctrl+b [     → enter scroll mode (q to exit)
# Ctrl+b ?     → list all key bindings

# Practical workflow for Java development:
# Window 0: IntelliJ / code editor
# Window 1: mvn spring-boot:run (application server)
# Window 2: tail -f logs/application.log (live logs)
# Window 3: docker ps / kubectl (infrastructure)
# Window 4: git operations
```

### Dotfiles Management

Dotfiles are configuration files that start with a `.` (hidden files). They define your shell environment, editor settings, Git configuration, and tool preferences. Managing them in a Git repository lets you reproduce your development environment on any machine in minutes.

```bash
# Common dotfiles for a Java developer:
~/.zshrc              # Zsh configuration (aliases, PATH, plugins)
~/.bashrc             # Bash configuration (if using Bash)
~/.gitconfig          # Git global configuration
~/.gitignore_global   # Global gitignore patterns
~/.ssh/config         # SSH host configurations
~/.sdkman/etc/config  # SDKMAN configuration
~/.maven/settings.xml # Maven settings (mirrors, profiles)
~/.gradle/gradle.properties  # Gradle properties
~/.editorconfig       # Editor-agnostic formatting rules
~/.tmux.conf          # tmux configuration

# Example ~/.zshrc for a Java developer:
cat << 'EOF' > ~/.zshrc
# --- SDKMAN ---
export SDKMAN_DIR="$HOME/.sdkman"
[[ -s "$HOME/.sdkman/bin/sdkman-init.sh" ]] && source "$HOME/.sdkman/bin/sdkman-init.sh"

# --- Java ---
export JAVA_HOME="$HOME/.sdkman/candidates/java/current"
export PATH="$JAVA_HOME/bin:$PATH"

# --- Aliases ---
alias ll='ls -lah'
alias la='ls -A'
alias ..='cd ..'
alias ...='cd ../..'
alias gs='git status'
alias gl='git log --oneline -20'
alias gd='git diff'
alias gp='git pull'
alias gc='git commit'
alias mvn='mvn -T 1C'              # Maven with parallel builds
alias mvnc='mvn clean'
alias mvnt='mvn test'
alias mvni='mvn clean install -DskipTests'
alias mvnfull='mvn clean verify'
alias dkc='docker compose'
alias dkps='docker ps'
alias dklog='docker logs -f'
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'

# --- History ---
HISTSIZE=10000
SAVEHIST=10000
setopt SHARE_HISTORY
setopt HIST_IGNORE_DUPS
setopt HIST_IGNORE_SPACE

# --- Completion ---
autoload -Uz compinit && compinit
zstyle ':completion:*' menu select

# --- Prompt (or use Starship/Powerlevel10k) ---
PROMPT='%F{blue}%~%f %F{green}❯%f '
EOF

# Dotfiles repository setup:
mkdir ~/dotfiles && cd ~/dotfiles
git init
mv ~/.zshrc ~/dotfiles/
mv ~/.gitconfig ~/dotfiles/
ln -s ~/dotfiles/.zshrc ~/.zshrc        # symlink back
ln -s ~/dotfiles/.gitconfig ~/.gitconfig
git add . && git commit -m "Initial dotfiles"
# Push to GitHub for portability

# On a new machine:
git clone https://github.com/alice/dotfiles.git ~/dotfiles
cd ~/dotfiles && ./install.sh            # script that creates all symlinks
```

---

## Important Notes

### Keyboard Shortcuts (Readline / ZLE)

These shortcuts work in both Bash and Zsh and will dramatically speed up your terminal usage:

```
Navigation:
  Ctrl+A          → move cursor to beginning of line
  Ctrl+E          → move cursor to end of line
  Alt+F / Esc+F   → move forward one word
  Alt+B / Esc+B   → move backward one word
  Ctrl+XX         → toggle between beginning of line and current position

Editing:
  Ctrl+U          → delete from cursor to beginning of line
  Ctrl+K          → delete from cursor to end of line
  Ctrl+W          → delete word backward
  Alt+D           → delete word forward
  Ctrl+Y          → paste (yank) last deleted text
  Ctrl+_          → undo

History:
  Ctrl+R          → reverse search through command history (type to search)
  Ctrl+S          → forward search (may need stty -ixon first)
  Ctrl+P / Up     → previous command in history
  Ctrl+N / Down   → next command in history
  !!              → repeat last command (e.g., sudo !!)
  !$              → last argument of previous command
  !*              → all arguments of previous command

Process control:
  Ctrl+C          → send SIGINT (interrupt/kill current process)
  Ctrl+Z          → suspend current process (send to background)
  Ctrl+D          → send EOF (exit shell if empty, or end input)
  Ctrl+L          → clear screen (same as 'clear' command)
  bg              → resume suspended process in background
  fg              → bring background process to foreground
  jobs            → list background jobs
```

### Quoting Rules

```
Double quotes ("..."):
  → Variables ARE expanded: echo "$HOME" → /Users/alice
  → Command substitution works: echo "$(date)"
  → Escapes work: echo "line1\nline2"
  → Use when you need variable expansion

Single quotes ('...'):
  → Variables are NOT expanded: echo '$HOME' → $HOME (literal)
  → Everything is literal
  → Use when you want exact strings (regex patterns, sed commands)

Backticks (`...`):
  → Legacy command substitution: echo `date`
  → Use $(...) instead: echo $(date)
  → Backticks are harder to read and nest poorly

No quotes:
  → Word splitting occurs on spaces
  → Glob expansion occurs: echo *.java → Account.java Payment.java
  → Use only for simple, known-safe values
```

### Process Management

```bash
# View running processes
ps aux                     # all processes, detailed
ps aux | grep java         # find Java processes
ps -ef                     # all processes, full format
top                        # interactive process monitor (q to quit)
htop                       # better top (install: brew install htop)
pgrep -f "banking-api"     # find PIDs by name pattern
jps                        # Java-specific: list JVM processes
jstack <pid>               # Java-specific: thread dump
jmap -heap <pid>           # Java-specific: heap summary

# Kill processes
kill <pid>                 # send SIGTERM (graceful shutdown)
kill -9 <pid>              # send SIGKILL (force kill, no cleanup)
killall java               # kill all processes named "java"
pkill -f "banking-api"     # kill by pattern

# Background and foreground
./mvnw spring-boot:run &   # run in background (& at end)
nohup ./script.sh &        # run in background, immune to terminal close
jobs                       # list background jobs
fg %1                      # bring job 1 to foreground
bg %1                      # resume job 1 in background

# Screen/tmux for persistent sessions (preferred over nohup)
tmux new -s server
./mvnw spring-boot:run     # inside tmux
# Ctrl+b d to detach — server keeps running
# tmux attach -t server to reconnect
```

### SSH Basics

```bash
# Connect to a remote server
ssh user@hostname
ssh alice@192.168.1.100
ssh -p 2222 alice@server   # custom port

# SSH config (~/.ssh/config)
Host banking-prod
    HostName 10.0.1.50
    User deploy
    Port 22
    IdentityFile ~/.ssh/banking-prod-key
    ForwardAgent yes

# Now just: ssh banking-prod

# Copy files over SSH
scp file.txt alice@server:/home/alice/
scp -r src/ alice@server:/home/alice/project/
scp alice@server:/var/log/app.log ./local-copy.log

# Rsync (faster for large/synced transfers)
rsync -avz --progress src/ alice@server:/home/alice/project/src/
rsync -avz --delete src/ alice@server:/project/  # mirror (delete extra files)

# SSH tunneling (port forwarding)
ssh -L 5432:localhost:5432 alice@db-server
# Now connect to localhost:5432 and it tunnels to the remote PostgreSQL

# Generate SSH key pair
ssh-keygen -t ed25519 -C "alice@example.com"
# Public key: ~/.ssh/id_ed25519.pub → add to GitHub / server's authorized_keys
# Private key: ~/.ssh/id_ed25519 → NEVER share this
```

### Package Managers

```bash
# macOS (Homebrew)
brew install java           # install a package
brew install maven gradle   # install multiple
brew update                 # update package list
brew upgrade                # upgrade all installed packages
brew list                   # list installed packages
brew search docker          # search for a package
brew info docker            # package details
brew services list          # list running services
brew services start postgresql  # start a service
brew cask install intellij-idea-ce  # install GUI apps

# Ubuntu/Debian (apt)
sudo apt update             # update package list
sudo apt upgrade            # upgrade installed packages
sudo apt install openjdk-21-jdk maven docker.io
sudo apt remove package
sudo apt search keyword
apt list --installed

# SDKMAN (Java-specific)
sdk list java               # list available JDK versions
sdk install java 21.0.2-tem # install specific JDK
sdk use java 21.0.2-tem     # switch JDK for current shell
sdk default java 21.0.2-tem # set default JDK
sdk current java            # show current JDK
sdk install maven           # install Maven
sdk install gradle          # install Gradle
```

---

## Practice

1. Open your terminal and navigate to your home directory using only relative paths
2. Create a nested directory structure: `projects/banking-api/src/main/java/com/example` using a single `mkdir` command
3. Find all `.java` files in your project that contain the word "TODO" and display the filename, line number, and matching line
4. Write a pipeline that counts the total number of lines of Java code in your project (excluding test files and the `target/` directory)
5. Use `sed` to replace all occurrences of "System.out.println" with "log.info" in a Java file (test without `-i` first!)
6. Use `awk` to parse the output of `ps aux | grep java` and display only the PID, CPU%, memory%, and command columns
7. Write a shell script that:
   - Checks if `JAVA_HOME` is set (error if not)
   - Runs `mvn clean test`
   - Reports success or failure with the exit code
   - Logs the output to a timestamped file
8. Use `curl` to hit a local Spring Boot actuator health endpoint and format the JSON response with `jq`
9. Set up a `tmux` session with 3 panes: one running your app, one tailing logs, one for Git commands
10. Create a dotfiles repository on GitHub with your `.zshrc`, `.gitconfig`, and a `setup.sh` script that creates symlinks
11. Use `grep` with context (`-C`) to find all exception stack traces in a log file and display 5 lines of context around each
12. Write a one-liner that finds all Java files modified in the last 24 hours and counts their total lines
13. Use `curl` to test a REST API endpoint with a JSON body and Authorization header
14. Create an SSH config entry for a remote server and test the connection
15. Write a shell script that uses a `for` loop to build all Maven modules in a multi-module project and reports which ones failed

---

## References

- GNU Bash Manual: https://www.gnu.org/software/bash/manual/
- Zsh Documentation: https://zsh.sourceforge.io/Doc/
- Oh My Zsh: https://ohmyz.sh/
- tmux Manual: https://man.openbsd.org/tmux
- "The Linux Command Line" — William Shotts (free PDF): https://linuxcommand.org/tlcl.php
- "Unix Power Tools" — Jerry Peek, Tim O'Reilly, Mike Loukides
- Explain Shell (paste any command for breakdown): https://explainshell.com/
- ShellCheck (lint shell scripts): https://www.shellcheck.net/
