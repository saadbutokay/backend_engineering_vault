**Phase:** Level 0 - Environment & Mindset Setup
**Date Studied:** 28th July, 2026

---
## What Problem Does This Solve?
You are about to write thousands of lines of code over the next year.
Without Git, here is what happens:
```
project-final.java
project-final-v2.java
project-final-v2-FIXED.java
project-final-v2-FIXED-actually-final.java
project-backup-before-i-break-everything.java
```
You've seen this. Everyone has done this.
This is not how professionals work.

With Git, here is what happens:
```
project.java  ← one file, always the latest version

But Git secretly remembers:
  → Every version that ever existed
  → Exactly what changed between versions
  → Who changed it and when
  → Why it was changed (commit message)
  → Can restore ANY previous version instantly
  → Can work on new features without touching working code
  → Multiple people can work on the same file without conflict
```
Git is not optional. Git is not a nice-to-have. Git is as fundamental to software engineering as knowing how to read is to being a writer.

---
## The Full Mental Model - Understand This First
Before memorizing commands, understand HOW Git thinks.

### The Three Areas of Git
Everything in Git exists in one of three places:
```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR COMPUTER                            │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐  │
│  │  Working    │    │   Staging   │    │   Local         │  │
│  │  Directory  │    │    Area     │    │   Repository    │  │
│  │             │    │  (Index)    │    │   (.git folder) │  │
│  │             │    │             │    │                 │  │
│  │ Your actual │    │ Files you   │    │ All committed   │  │
│  │ files that  │    │ have marked │    │ history lives   │  │
│  │ you edit    │    │ "ready to   │    │ here            │  │
│  │ every day   │    │  commit"    │    │                 │  │
│  └──────┬──────┘    └──────┬──────┘    └────────┬────────┘  │
│         │                  │                    │           │
│         │   git add        │   git commit       │           │
│         │ ───────────────► │ ─────────────────► │           │
│         │                  │                    │           │
│         │ ◄─────────────── │ ◄───────────────── │           │
│         │   git restore    │   git reset        │           │
└─────────────────────────────────────────────────────────────┘
                                        │
                                        │  git push
                                        ▼
                              ┌─────────────────┐
                              │     GITHUB      │
                              │  (Remote Repo)  │
                              │                 │
                              │  Your code      │
                              │  stored online  │
                              │  Safe backup    │
                              │  Portfolio      │
                              └─────────────────┘
                                        │
                                        │  git pull
                                        ▼
                              (back to your computer)
```

### The Simple Analogy
```text
Think of it like packing a box to ship:

Working Directory = Your room (stuff is everywhere, changing)

Staging Area      = The box you're packing
                    You pick WHICH items to put in
                    You can add more or take things out
                    Nothing is shipped yet

Local Repository  = The sealed, labeled box
                    Once committed = permanent record
                    Has a tracking number (commit hash)
                    Can never be "unseen" (safely)

GitHub (Remote)   = The warehouse where boxes are stored
                    Safe, accessible from anywhere
                    Others can see it (if public)
                    Your backup if your laptop dies
```

### The Commit - Most Important Concept
```text
A COMMIT is a permanent snapshot of your code at a point in time.

Each commit has:
  ┌────────────────────────────────────────────────┐
  │  Commit: a3f8c91                               │
  │  Author: Your Name <email@gmail.com>           │
  │  Date:   Mon Jan 15 14:32:11 2025              │
  │  Message: feat: add user registration endpoint │
  │                                                │
  │  Changes:                                      │
  │    + UserController.java (added)               │
  │    + UserService.java (added)                  │
  │    ~ pom.xml (modified - added dependency)     │
  └────────────────────────────────────────────────┘

The hash (a3f8c91) is like a fingerprint.
Unique identifier for this exact snapshot.
You can always go back to this exact state.

Think of commits like save points in a game.
You can always load a previous save.
```

---
## Core Git Commands

### Setup Commands (One Time)
```bash
# Set your identity (do this once globally)
git config --global user.name "Your Full Name"
git config --global user.email "your@email.com"

# Set default branch name to 'main'
git config --global init.defaultBranch main

# Set VS Code as default editor
git config --global core.editor "code --wait"

# Make Git output colorful (easier to read)
git config --global color.ui auto

# See all your config settings
git config --list

# See one specific setting
git config user.name
```

### Starting a Repository
```bash
# Method 1: Start fresh in an existing folder
cd ~/projects/my-project
git init

# What this does:
# Creates a hidden .git folder inside your project
# This folder IS the repository — all history lives here
# NEVER manually edit or delete .git

# Verify it worked:
ls -la
# You should see: .git folder listed

# Method 2: Clone an existing repository from GitHub
git clone git@github.com:username/repository-name.git

# This:
# Downloads the full repository to your computer
# Automatically sets up the connection to GitHub
# Creates a folder with the repo name

# Clone into a specific folder name:
git clone git@github.com:username/repo.git my-folder-name
```

### The Daily Workflow Commands
These are the commands you will use EVERY SINGLE DAY.

#### git status - Your Best Friend
```bash
git status

# This shows you:
# - Which files have been modified
# - Which files are staged (ready to commit)
# - Which files are untracked (new files Git doesn't know about)
# - Which branch you're on

# Example output:
# On branch main
# Changes to be committed:           ← STAGED (ready to commit)
#   (use "git restore --staged <file>..." to unstage)
#         new file:   UserService.java
#
# Changes not staged for commit:     ← MODIFIED but NOT staged
#   (use "git add <file>..." to update what will be committed)
#         modified:   Main.java
#
# Untracked files:                   ← NEW files Git doesn't track yet
#   (use "git add <file>..." to include in what will be committed)
#         notes.txt

# Run git status CONSTANTLY.
# Before every add. After every add. Before every commit.
# It tells you exactly what's happening.
```

#### git add - Staging Files
```bash
# Stage a specific file
git add UserService.java

# Stage multiple specific files
git add UserService.java UserController.java

# Stage ALL changes in current directory and subdirectories
git add .

# Stage all files with a specific extension
git add *.java

# Stage parts of a file (interactive — advanced)
git add -p filename.java
# Git will show each change and ask: stage this? (y/n)
# Useful when one file has multiple unrelated changes

# IMPORTANT RULE:
# Think before using git add .
# Always run git status after to verify what you staged
# Make sure you're not staging debug code, passwords,
# or generated files
```

#### git commit - Creating a Snapshot
```bash
# Commit with a message (always do this)
git commit -m "feat: add user registration endpoint"

# Commit and open editor for longer message
git commit
# Editor opens → write message → save → close editor

# Stage and commit all TRACKED files in one step
# (skips staging for modified files — not new files)
git commit -am "fix: correct null check in UserService"

# See your commit was created:
git log --oneline
```

#### Writing Good Commit Messages
```text
This is a professional skill. Learn it from day one.
Your commit messages are read by teammates, future you,
and potential employers looking at your GitHub.

FORMAT (Conventional Commits — industry standard):
  <type>: <short description>

  [optional longer body explaining WHY]

  [optional footer for issue references]

TYPES:
  feat     → new feature
  fix      → bug fix
  docs     → documentation only changes
  style    → formatting, no logic change (spaces, semicolons)
  refactor → code change that neither fixes bug nor adds feature
  test     → adding or fixing tests
  chore    → build process, dependency updates, tooling

RULES:
  ✅ Use present tense: "add feature" not "added feature"
  ✅ Keep first line under 72 characters
  ✅ Be specific: describe WHAT changed
  ✅ If needed, explain WHY in the body

EXAMPLES — Good vs Bad:

  ❌ Bad commit messages:
  "fix"
  "changes"
  "asdf"
  "WIP"
  "i have no idea what i'm doing"
  "fixed the thing"

  ✅ Good commit messages:
  "feat: add JWT authentication to user login endpoint"
  "fix: handle null user ID in order creation service"
  "docs: add API documentation to README"
  "refactor: extract payment logic into PaymentService class"
  "test: add unit tests for UserService registration method"
  "chore: update Spring Boot to 3.2.0"
  "fix: resolve N+1 query issue in product listing endpoint"

  ✅ Good commit with body:
  "fix: prevent duplicate order creation on retry

  When users clicked 'Order' multiple times quickly,
  duplicate orders were being created. Added idempotency
  key check before creating order record.

  Fixes #143"
```

### Viewing History
```bash
# See full commit history
git log

# See compact one-line history (use this most)
git log --oneline

# Example output:
# a3f8c91 (HEAD -> main) feat: add order creation endpoint
# b2e7d44 fix: resolve null pointer in UserService
# c1a5f33 feat: add user registration
# d0b4e22 chore: initial project setup

# See history with graph (shows branches visually)
git log --oneline --graph --all

# See what changed in a specific commit
git show a3f8c91

# See what changed in the last commit
git show HEAD

# See history of a specific file
git log --oneline -- UserService.java

# See who changed what line in a file (blame = find who wrote this)
git blame UserService.java

# See changes between two commits
git diff a3f8c91 b2e7d44
```

### Undoing Things (CRITICAL)
```text
This is where Git saves you from disasters.
Different situations need different solutions.

SITUATION 1: I changed a file, haven't staged it.
I want to DISCARD my changes and go back.

  git restore filename.java
  # OR (older syntax):
  git checkout -- filename.java

  ⚠️  WARNING: This PERMANENTLY discards your changes.
  Cannot be undone. The file goes back to last commit.

SITUATION 2: I staged a file but haven't committed.
I want to UNSTAGE it (keep changes but remove from staging).

  git restore --staged filename.java
  # File is now modified but NOT staged
  # Your changes are still there in the working directory

SITUATION 3: I committed something wrong.
I want to UNDO the last commit but KEEP my changes.

  git reset --soft HEAD~1
  # Moves HEAD back one commit
  # Your changes are back in staging area
  # Commit is gone, but code is still there
  # You can fix and recommit

SITUATION 4: I committed something wrong.
I want to UNDO the last commit AND unstage changes.

  git reset HEAD~1
  # OR:
  git reset --mixed HEAD~1
  # Moves HEAD back one commit
  # Your changes are back in working directory (not staged)
  # You need to git add again before committing

SITUATION 5: I committed something wrong.
I want to completely THROW AWAY last commit and changes.

  git reset --hard HEAD~1
  ⚠️  DANGER: This PERMANENTLY deletes the commit AND changes.
  Use only when you're absolutely sure.

SITUATION 6: I already PUSHED to GitHub.
I want to undo it WITHOUT rewriting history.

  git revert HEAD
  # Creates a NEW commit that undoes the last commit
  # The old commit still exists in history
  # Safe for pushed code — doesn't rewrite history
  # This is the CORRECT way to undo pushed commits

SITUATION 7: My working directory is a mess.
I want to start over completely from last commit.

  git reset --hard HEAD
  ⚠️  DANGER: Permanently discards ALL uncommitted changes.
  Use when you've broken everything and want a clean slate.

HEAD explained:
  HEAD = pointer to the current commit you're on
  HEAD~1 = one commit before HEAD
  HEAD~2 = two commits before HEAD
  HEAD~n = n commits before HEAD
```

### Branching - Working on Features Safely
```text
THE MOST IMPORTANT GIT CONCEPT FOR PROFESSIONAL WORK.

Why branches?
  main branch = production code. Always working. Always stable.

  Without branches:
    You're working on a new feature.
    Feature takes 3 days.
    While working: production bug found.
    You can't fix it cleanly — your broken feature is mixed in.
    Chaos.

  With branches:
    Create feature branch → work there safely.
    main stays clean and stable.
    Bug found? Switch to main, fix, deploy.
    Come back to feature branch → continue.
    When feature is done → merge into main.

MENTAL MODEL:
  main branch:
  A → B → C → D (all working, all tested)
                │
                │  (branch off here)
                ▼
  feature/user-auth:
                D → E → F → G (your feature work)
                              │
                              │  (merge back when done)
                              ▼
  main branch:
  A → B → C → D ──────────── H (merged!)
```

#### Branch Commands
```bash
# See all branches (* = current branch)
git branch

# See all branches including remote
git branch -a

# Create a new branch
git branch feature/user-login

# Switch to a branch
git checkout feature/user-login
# OR (modern syntax - use this):
git switch feature/user-login

# Create AND switch in one command (use this most)
git checkout -b feature/user-login
# OR (modern):
git switch -c feature/user-login

# Rename current branch
git branch -m new-name

# Delete a branch (after merging)
git branch -d feature/user-login

# Force delete (even if not merged — careful!)
git branch -D feature/user-login

# See which branch you're on (also shown in git status)
git branch --show-current
```

#### Merging Branches
```bash
# STEP 1: Switch to the branch you want to merge INTO
git switch main

# STEP 2: Merge the feature branch INTO main
git merge feature/user-login

# If no conflicts → Fast-forward or merge commit created
# If conflicts → Git pauses and asks you to resolve

# After successful merge, delete the feature branch
git branch -d feature/user-login
```

#### Understanding Merge Conflicts
```text
A merge conflict happens when:
  Two branches changed the SAME LINE of the SAME FILE.
  Git doesn't know which version to keep.
  Git asks YOU to decide.

Example conflict in a file:

<<<<<<< HEAD
    String greeting = "Hello, World!";
=======
    String greeting = "Hi there, World!";
>>>>>>> feature/greeting-change

What this means:
  <<<<<<< HEAD      → What YOUR current branch has
  =======           → Separator
  >>>>>>> feature/  → What the OTHER branch has

HOW TO RESOLVE:
  1. Open the file in IntelliJ
  2. IntelliJ shows a visual merge tool (much better than raw text)
     Click: "Resolve conflicts" button that appears
  3. Choose: Accept Yours / Accept Theirs / Merge manually
  4. Delete ALL the conflict markers (<<<<, ====, >>>>)
  5. Make the file exactly how you WANT it to be
  6. Save the file
  7. git add the resolved file
  8. git commit (Git pre-fills a merge commit message)

IN INTELLIJ:
  When a conflict happens after merge:
  IntelliJ shows: "Conflicts detected. Resolve conflicts?"
  Click "Resolve"
  Three-panel view:
    Left: Your version
    Center: Result (what you edit)
    Right: Their version
  Use arrow buttons to accept changes from either side
  Click Apply

Prevention:
  → Merge main into your feature branch BEFORE merging feature into main
  → Small, frequent commits reduce conflict size
  → Communicate with teammates about who touches what
```

### Working with GitHub (Remote)
```bash
# See remote connections
git remote -v

# Add a remote (usually done once when connecting to GitHub)
git remote add origin git@github.com:username/repo.git

# 'origin' is just a conventional name for your main remote
# You can name it anything, but 'origin' is the standard

# Remove a remote
git remote remove origin

# Change remote URL
git remote set-url origin git@github.com:username/new-repo.git

# Push to GitHub (first time - sets up tracking)
git push -u origin main
# -u = --set-upstream (remember where to push to)
# After this, just:
git push

# Push a new branch to GitHub
git push -u origin feature/user-login

# Pull latest changes from GitHub
git pull
# This = git fetch + git merge combined

# Fetch changes WITHOUT merging (just download, don't apply)
git fetch origin

# See what's on GitHub that you don't have yet
git fetch origin
git log HEAD..origin/main --oneline

# Delete a branch on GitHub
git push origin --delete feature/user-login
```

### Stashing - Save Work Temporarily
```bash
# SCENARIO:
# You're in the middle of coding feature A.
# Urgent bug found — need to switch to main to fix it.
# But your feature A code isn't ready to commit.
# git stash saves it temporarily.

# Stash your current changes (saves them, cleans working dir)
git stash

# Give it a descriptive name
git stash push -m "WIP: user authentication form validation"

# See all stashes
git stash list
# Output:
# stash@{0}: WIP: user authentication form validation
# stash@{1}: On main: experimenting with new parser

# Apply most recent stash (keeps stash in list)
git stash apply

# Apply most recent stash AND remove from stash list
git stash pop

# Apply a specific stash
git stash apply stash@{1}

# Delete a stash
git stash drop stash@{0}

# Delete all stashes
git stash clear

# TYPICAL WORKFLOW:
git stash push -m "WIP: feature A"  # save current work
git switch main                     # switch to main
# fix the bug
git commit -m "fix: urgent production bug"
git push
git switch feature/a                # go back to feature
git stash pop                       # restore your work
# continue where you left off
```

### Other Useful Commands
```bash
# See difference between working directory and last commit
git diff

# See difference between staged and last commit
git diff --staged

# See difference between two branches
git diff main feature/user-login

# See difference in a specific file
git diff UserService.java

# Cherry-pick: apply ONE specific commit from another branch
git cherry-pick a3f8c91
# Useful: "I want just this one commit from that branch"

# See a specific version of a file from history
git show HEAD~3:src/main/java/UserService.java

# Restore a file to its state from 3 commits ago
git restore --source HEAD~3 UserService.java

# Find which commit introduced a bug (bisect)
git bisect start
git bisect bad HEAD          # current version is bad
git bisect good a3f8c91      # this version was good
# Git checks out commits between them
# You test → git bisect good OR git bisect bad
# Git finds the exact commit that broke things
git bisect reset             # when done
```

---
## Professional Git Workflow - How Teams Work

### The Feature Branch Workflow (Standard at Most Companies)
```text
This is what you'll do at every job.
Learn this workflow, not just the commands.

DAILY WORKFLOW:

Step 1: Start your day — get latest code
  git switch main
  git pull
  # Your main branch is now up to date

Step 2: Create a branch for your task
  # Branch naming convention:
  # feature/  → new feature
  # fix/      → bug fix
  # docs/     → documentation
  # refactor/ → code cleanup
  # test/     → tests

  git switch -c feature/user-password-reset
  # Now you're on your own branch. Safe to work.

Step 3: Work → commit frequently
  # Work for 30-60 minutes on a small piece
  git add .
  git status    # verify what you're committing
  git commit -m "feat: add password reset email sending"

  # Work more
  git add .
  git commit -m "feat: add password reset token validation"

  # Work more
  git add .
  git commit -m "test: add unit tests for password reset flow"

  # Small commits = easier to review, easier to revert if needed
  # Commit at least once before leaving for the day

Step 4: Push to GitHub regularly
  git push -u origin feature/user-password-reset
  # After first push, just: git push
  # Why? Backup. If your laptop dies, code is on GitHub.

Step 5: Keep your branch updated (do this daily)
  git switch main
  git pull
  git switch feature/user-password-reset
  git merge main
  # Brings latest main into your feature branch
  # Resolve any conflicts now (easier when small)
  # Don't wait until you're done to merge main!

Step 6: Create a Pull Request when feature is done
  GitHub → Your repository
  "Compare & pull request" button appears
  Fill in:
    Title: "feat: add user password reset flow"
    Description:
      ## What does this PR do?
      Adds the ability for users to reset their password via email.

      ## Changes
      - Added /auth/forgot-password endpoint
      - Added /auth/reset-password endpoint
      - Added password reset email template
      - Added PasswordResetToken entity

      ## How to test
      1. Call POST /auth/forgot-password with email
      2. Check email (or logs in dev) for reset link
      3. Call POST /auth/reset-password with token and new password
      4. Verify login works with new password

      ## Related issue
      Closes #47

  Click: "Create Pull Request"

Step 7: Code review
  A senior engineer reviews your code
  They leave comments on specific lines
  You respond to comments and make fixes
  Push new commits to same branch → PR updates automatically
  When approved → merge

Step 8: Merge and cleanup
  GitHub: Click "Merge Pull Request"
  GitHub: Click "Delete Branch" (keeps things clean)

  On your computer:
  git switch main
  git pull        # get the merged code
  git branch -d feature/user-password-reset  # delete local branch
```

---
## `.gitignore` - What NOT to Track

```text
Some files should NEVER go into Git.
.gitignore tells Git to completely ignore them.

CREATE .gitignore IN YOUR PROJECT ROOT.
```

Standard Java/Spring Boot `.gitignore`:
```gitignore
# Compiled class files
*.class

# Log files
*.log
logs/

# BlueJ files
*.ctxt

# Mobile Tools for Java
.mtj.tmp/

# Package Files
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

# Gradle (if using Gradle instead)
.gradle/
build/
!gradle/wrapper/gradle-wrapper.jar

# IntelliJ IDEA
.idea/
*.iws
*.iml
*.ipr
out/

# VS Code
.vscode/
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json

# Eclipse (other devs might use it)
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

# Operating System files
.DS_Store
Thumbs.db
desktop.ini

# Environment variables / Secrets
.env
.env.local
.env.production
application-prod.yml
secrets.properties

# Docker
docker-compose.override.yml
```

```text
GOLDEN RULE:
  NEVER commit:
  × Passwords
  × API keys
  × Database credentials
  × Secret tokens
  × .env files

  These should NEVER appear in Git history.
  If you accidentally commit a secret:
  1. Immediately rotate/change the secret (assume compromised)
  2. Use git filter-branch or BFG Repo Cleaner to remove from history
  3. Force push (with team's knowledge)

EASY WAY: Generate .gitignore automatically
  Website: https://www.toptal.com/developers/gitignore
  Type: java, intellij, macos, windows, gradle, maven
  Copy the generated content → paste into your .gitignore
```

---
## GitHub - Professional Profile Setup

### Making Your GitHub Look Professional
```text
Your GitHub IS your portfolio.
Employers spend 2-3 minutes on your GitHub.
Make those minutes count.

PROFILE PAGE (github.com/yourusername):
  Photo:    Professional or neutral. Not memes.
  Name:     Real full name.
  Bio:      "Java Backend Engineer | CSE Student | Building
             production-grade systems"
  Location: Dhaka, Bangladesh
  Website:  Link to portfolio (when you have one)

PINNED REPOSITORIES (choose 6 best projects):
  Pin your best 3-4 projects.
  These are the first thing employers see.
  Pin only projects with:
    ✓ Good README
    ✓ Real functionality
    ✓ Clean code
    ✓ Consistent commits

CONTRIBUTION GRAPH (the green squares):
  The calendar of your commits.
  Consistent green = consistent work.
  Employers look at this.
  Goal: green squares EVERY day or nearly every day.
  Even small commits count.
  Don't make fake commits to look active — build real things.

README FOR EVERY PROJECT:
  Every project MUST have a README.md.
  This is what people read first.
```

Structure:
````markdown
# Project Name

Short description of what this project does.

## Tech Stack
- Java 21
- Spring Boot 3.2
- PostgreSQL
- Redis
- Docker

## Features
- User registration and JWT authentication
- Product catalog with search and filtering
- Order management
- ...

## Architecture
[diagram or description]

## Getting Started
### Prerequisites
- Java 21
- Docker Desktop

### Running Locally

```bash
git clone git@github.com:username/project.git
cd project
docker compose up -d
mvn spring-boot:run
```

## API Documentation

Swagger UI available at: **http://localhost:8080/swagger-ui.html**

## Screenshots

[add screenshots of Swagger UI, Postman tests]
````

---
## Complete Practice Exercise
*Do this right now. Step by step.*
```text
EXERCISE: Practice the complete Git workflow
```

### SETUP:
```bash
mkdir ~/projects/git-practice
cd ~/projects/git-practice
git init
```

### ROUND 1 - Basic commits:
```bash
# Create a file
echo "# My Java Learning Journal" > README.md

# Check status
git status
# You should see: Untracked files: README.md

# Stage it
git add README.md
git status
# You should see: Changes to be committed: new file: README.md

# Commit it
git commit -m "docs: add README"

# See the commit
git log --oneline
```

### ROUND 2 - Modify and commit:
```bash
# Edit README (add a line)
echo "Learning Java backend engineering" >> README.md

git status
# Modified: README.md

git diff
# See exactly what changed (+ = added, - = removed)

git add README.md
git commit -m "docs: add learning goal to README"

git log --oneline
# Now see 2 commits
```

### ROUND 3 - Branching:
```bash
# Create and switch to a new branch
git switch -c feature/java-notes

# Create a new file on this branch
echo "Java is strongly typed" > java-notes.md
git add .
git commit -m "docs: add initial Java notes"

echo "OOP: Classes, Objects, Inheritance" >> java-notes.md
git add .
git commit -m "docs: add OOP notes"

# See that main doesn't have these files
git switch main
ls  # java-notes.md is NOT here

git log --oneline
# Only the 2 README commits
```

### ROUND 4 - Merge:
```bash
# Merge feature branch into main
git merge feature/java-notes

ls  # NOW java-notes.md is here
git log --oneline  # See all 4 commits
```

### ROUND 5 - Undo something:
```bash
# Make a "mistake"
echo "THIS IS A MISTAKE" >> java-notes.md
git add .
git commit -m "oops: accidentally added wrong content"

git log --oneline  # See the bad commit

# Undo it (keep changes in working directory)
git reset HEAD~1

git log --oneline  # Bad commit is gone
git status         # Changes are back, unstaged

# Discard the changes completely
git restore java-notes.md
git status  # Clean!
```

### ROUND 6 - Stashing:
```bash
# Make some changes (simulate half-done work)
echo "half done work here" >> README.md
git status  # Modified but not committed

# Need to quickly switch task — stash it
git stash push -m "WIP: updating README"
git status  # Clean working directory

# Do some other work
git switch -c hotfix/typo
echo "fixed" > fix.md
git add .
git commit -m "fix: typo in documentation"
git switch main
git merge hotfix/typo
git branch -d hotfix/typo

# Come back to your stashed work
git switch main  # already here
git stash pop
git status  # Your half-done README change is back

# Clean up (don't commit the test content)
git restore README.md
```

### ROUND 7 - Push to GitHub:
```bash
# Create repo on GitHub first:
# GitHub → New Repository
# Name: git-practice
# Don't initialize (you have local code)

git remote add origin git@github.com:YOURUSERNAME/git-practice.git
git push -u origin main

# Visit GitHub — your repo is live!
```
**DONE!** You practiced every core Git concept.

---
## Quick Reference Card
*Print this or keep it open while coding.*
```text
DAILY COMMANDS:
  git status                    → What's happening?
  git add .                     → Stage everything
  git add filename              → Stage specific file
  git commit -m "type: msg"     → Create snapshot
  git push                      → Send to GitHub
  git pull                      → Get from GitHub
  git log --oneline             → See history

BRANCHING:
  git switch -c feature/name    → Create + switch
  git switch main               → Go to main
  git branch                    → List branches
  git merge feature/name        → Merge into current
  git branch -d feature/name    → Delete branch

UNDOING:
  git restore file              → Discard changes (working dir)
  git restore --staged file     → Unstage file
  git reset HEAD~1              → Undo last commit (keep code)
  git reset --hard HEAD~1       → Undo last commit (delete code)
  git revert HEAD               → Undo last commit (safe, pushed)

STASHING:
  git stash                     → Save work temporarily
  git stash pop                 → Restore saved work
  git stash list                → See all stashes

VIEWING:
  git diff                      → See unstaged changes
  git diff --staged             → See staged changes
  git log --oneline --graph     → Visual history
  git show HEAD                 → See last commit details
  git blame filename            → Who wrote each line

REMOTE:
  git remote -v                 → See remote connections
  git push -u origin branch     → Push new branch
  git fetch origin              → Download without merging
  git clone URL                 → Download full repository
```

---
## Common Mistakes to Avoid
```text
MISTAKE 1: Committing to main directly
  ❌ Working on main branch for new features
  ✅ Always create a feature branch
  Why: main should always be stable and deployable

MISTAKE 2: Giant commits
  ❌ Working for 8 hours then making one commit: "did stuff"
  ✅ Small, logical commits every 30-60 minutes
  Why: easier to review, easier to revert, clearer history

MISTAKE 3: Committing secrets
  ❌ Committing .env files, passwords, API keys
  ✅ Add to .gitignore BEFORE first commit
  Why: once in Git history, it's there forever (even if deleted)
  Rotate any secret that ever touched a commit

MISTAKE 4: Using git add . blindly
  ❌ git add . without checking what you're staging
  ✅ git status before and after git add
  Why: might accidentally stage debug code, test data, binaries

MISTAKE 5: Force pushing to shared branches
  ❌ git push --force on main or shared feature branches
  ✅ git push --force-with-lease (safer) — only on YOUR branches
  Why: rewrites history, destroys other people's work

MISTAKE 6: Ignoring merge conflicts
  ❌ Accepting all "yours" or all "theirs" without reading
  ✅ Actually read both versions and make the RIGHT choice
  Why: merge conflicts mean two people changed the same thing
  The right answer might be a combination of both

MISTAKE 7: Not pulling before starting work
  ❌ Starting new work on stale code
  ✅ git pull at the start of every work session
  Why: avoid conflicts, avoid duplicating work already done

MISTAKE 8: Vague commit messages
  ❌ "fix", "update", "changes", "wip"
  ✅ "fix: resolve NullPointerException when user has no orders"
  Why: future you and teammates need to understand history
```

---
## Interview Questions

**Q: What is the difference between git merge and git rebase?**
A: Both integrate changes from one branch into another.
Merge creates a new "merge commit" that joins two branch histories — preserves full history of both branches.
Rebase rewrites your branch's commits on top of another branch — creates a linear, cleaner history but rewrites commit hashes.
Rule: Never rebase shared/public branches.
Use merge for feature → main. Use rebase for keeping feature branch updated with main privately.

**Q: What is the difference between git fetch and git pull?**
A: `git fetch` downloads changes from remote but does NOT apply them. Your working directory is unchanged. Safe to run any time.
`git pull` = `git fetch` + `git merge`. Downloads AND applies changes.
Use fetch when you want to see what changed before merging. Use pull when you're ready to integrate remote changes.

**Q: What is git stash and when would you use it?**
A: `git stash` temporarily saves uncommitted changes to a stack, cleaning your working directory without committing.
Use it when you need to switch branches urgently but have half-done work you're not ready to commit.
`git stash pop` restores the saved changes.

**Q: How do you resolve a merge conflict?**
A: Open the conflicted file — Git marks conflicts with `<<<<<<<`, `=======`, `>>>>>>>` markers.
Decide which version is correct (or combine both). Remove all conflict markers. `git add` the resolved file. `git commit` to complete the merge.
In IntelliJ, the visual merge tool makes this much easier.

**Q: What is the difference between git reset and git revert?**
A: `git reset` moves the branch pointer backward — rewrites history. Dangerous for pushed commits because it changes shared history.
`git revert` creates a new commit that undoes a previous commit — history is preserved, nothing is rewritten.
Always use `git revert` for commits already pushed to shared branches.

---
## Key Takeaways
1. **Git has THREE areas:**
   Working Directory → Staging Area → Repository
   `add` moves to staging. `commit` moves to repository.
2. **Commits are permanent snapshots.**
   Write meaningful commit messages — they are documentation.
   Use Conventional Commits format: `"type: description"`
3. **Branch for EVERYTHING.**
   `main` = stable, always deployable.
   Every feature, fix, and experiment gets its own branch.
4. **Push daily.**
   Your GitHub is your backup AND your portfolio.
   If your laptop dies, your code lives on GitHub.
5. **Learn the undo commands.**
   `restore` → undo working directory changes
   `reset` → undo commits (local only)
   `revert` → undo commits (safe for pushed code)
6. **.gitignore from day one.**
   Never commit secrets, credentials, or generated files.
7. **Git is a skill that improves with use.**
   Don't just memorize commands.
   Use them every day. Make mistakes. Learn from them.
   In 3 months, this will feel completely natural.

---