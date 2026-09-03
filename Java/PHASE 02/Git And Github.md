## Overview

Git is a **distributed version control system** that tracks changes to files over time, enables collaboration across teams, and provides a complete history of every modification ever made to a codebase. GitHub is a **hosting platform** built on top of Git that adds pull requests, code review, CI/CD integration, issue tracking, and project management.

For a Java backend engineer, Git is not optional—it is the **single most important collaboration tool** you will use. Every line of code you write in a professional setting will pass through Git. Your commits will be reviewed by teammates. Your branching strategy will determine how smoothly your team ships features. Your commit messages will be read by future developers (including yourself) trying to understand why a change was made six months ago.

This note covers Git from the ground up: the internal data model that makes Git work, every command you need for daily development, branching strategies used in enterprise Java teams, conventional commit standards, pull request workflows, SSH authentication, and GitHub-specific features. The goal is not just to memorize commands but to **understand what Git is actually doing** so you can recover from mistakes, resolve complex merge conflicts, and use advanced features with confidence.

---

## Core Concepts

### Git Architecture — The Three Trees

Git manages three "trees" (states) of your project:

```
Working Directory          Staging Area (Index)         Repository (.git)
(the files you see         (the snapshot you are        (the permanent
 and edit)                  about to commit)             history)

  ┌──────────┐    git add    ┌──────────┐  git commit  ┌──────────┐
  │  src/    │ ──────────→  │  src/    │ ──────────→  │  commit  │
  │  pom.xml │              │  pom.xml │              │  object  │
  │  README  │              │          │              │  (SHA)   │
  └──────────┘              └──────────┘              └──────────┘
       ↑                                                  │
       │              git checkout / git restore           │
       └──────────────────────────────────────────────────┘
```

- **Working Directory:** The actual files on your filesystem. You edit these.
- **Staging Area (Index):** A snapshot of what will go into the next commit. You add files here with `git add`.
- **Repository:** The `.git` directory containing the complete history of all commits, branches, and tags.

### Git Object Model

Everything in Git is an **object** identified by a SHA-1 hash (a 40-character hexadecimal string):

```
Blob (Binary Large Object):
  → Stores file contents
  → "Hello World" always hashes to the same SHA, regardless of filename
  → git hash-object -t blob --stdin <<< "Hello World"

Tree:
  → Stores directory structure (filenames → blob/tree references)
  → Like a folder listing with pointers to blobs and sub-trees

Commit:
  → Points to a tree (the snapshot)
  → Points to parent commit(s) (one for normal, two for merge, zero for initial)
  → Contains metadata: author, committer, timestamp, message

Tag:
  → A named reference to a commit (usually for releases)
  → Lightweight tag: just a pointer
  → Annotated tag: a full object with message, author, date
```

The commit history is a **directed acyclic graph (DAG)** of commit objects, each pointing to its parent(s):

```
A ← B ← C ← D  (main)
         ↖
          E ← F  (feature-branch)
```

### HEAD, Branches, and Tags

```
HEAD:
  → A special pointer to the commit you are currently "on"
  → Usually points to a branch name, which in turn points to a commit
  → "Detached HEAD" = HEAD points directly to a commit (not a branch)
  → View: cat .git/HEAD → ref: refs/heads/main

Branch:
  → A lightweight, movable pointer to a commit
  → Creating a branch is instant (just creates a new pointer)
  → Moving a branch is instant (just updates the pointer)
  → This is why Git branching is so cheap compared to SVN/CVS

Tag:
  → A fixed pointer to a commit (does not move)
  → Used for releases: v1.0.0, v2.3.1
  → Annotated tags are preferred (they are full objects with metadata)
```

### Remote Repositories

```
Remote:
  → A named reference to a repository on another server (GitHub, GitLab, Bitbucket)
  → "origin" is the default name for the remote you cloned from
  → View: git remote -v
  → Remote-tracking branches: origin/main, origin/feature-x
    (read-only mirrors of the remote's branches, updated by git fetch)

Fetch vs Pull:
  git fetch  → downloads new data from remote but does NOT modify your working files
  git pull   → git fetch + git merge (downloads AND integrates changes)
  → Best practice: fetch first, review, then merge or rebase

Push:
  git push   → uploads your local commits to the remote
  → You can only push if your local branch is ahead of or up-to-date with the remote
  → If the remote has new commits, you must pull first (or force push, which is dangerous)
```

### Merge vs Rebase

Both integrate changes from one branch into another, but they produce different histories:

```
MERGE (git merge feature):
  → Creates a new "merge commit" with two parents
  → Preserves the exact history of when branches diverged and converged
  → Non-destructive: existing commits are not altered
  → History: A—B—C—M  (M is the merge commit)
                  ↗
             D—E

REBASE (git rebase main):
  → Replays your commits on top of the target branch
  → Creates new commit objects (different SHAs) with the same changes
  → Produces a linear, clean history
  → History: A—B—C—D'—E'  (D' and E' are new commits)
  → ⚠️  NEVER rebase commits that have been pushed to a shared branch
     (it rewrites history and breaks everyone else's local copies)

When to use which:
  → Merge: integrating feature branches into main (preserves context)
  → Rebase: updating your local feature branch with latest main (clean history)
  → Interactive rebase: cleaning up your own commits before pushing
```

---

## Code Examples

### Initial Setup

```bash
# One-time global configuration (do this first on any new machine)
git config --global user.name "Alice Johnson"
git config --global user.email "alice@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"   # VS Code as commit editor
git config --global core.autocrlf input          # macOS/Linux: LF line endings
git config --global pull.rebase true             # default to rebase on pull
git config --global fetch.prune true             # auto-delete stale remote branches
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --decorate --all"

# View configuration
git config --list
git config --global user.name
```

### Creating and Cloning Repositories

```bash
# Initialize a new local repository
mkdir banking-api && cd banking-api
git init
# Creates .git/ directory — the entire repository lives here

# Clone an existing repository
git clone https://github.com/alice/banking-api.git
git clone git@github.com:alice/banking-api.git    # SSH (preferred)
git clone --depth 1 https://github.com/alice/banking-api.git  # shallow clone (faster)
git clone -b develop https://github.com/alice/banking-api.git # clone specific branch
```

### The Daily Workflow

```bash
# 1. Check status (do this constantly)
git status
git status -s                    # short format

# 2. Stage changes
git add src/main/java/Account.java       # stage specific file
git add src/                             # stage entire directory
git add .                                # stage all changes in current directory
git add -p                               # interactive: stage hunks selectively
git add -u                               # stage only tracked files (no new files)

# 3. Commit
git commit -m "Add account balance validation"
git commit -m "fix: prevent negative balance on withdrawal"   # conventional commit
git commit -am "update: adjust transfer fees"  # stage tracked + commit (skip git add)
git commit --amend                           # modify the last commit (message or files)
git commit --amend --no-edit                 # amend without changing the message

# 4. View history
git log                              # full log (q to quit)
git log --oneline                    # one line per commit
git log --oneline -10                # last 10 commits
git log --oneline --graph --all      # visual branch graph
git log --author="Alice"             # filter by author
git log --since="2025-01-01"        # filter by date
git log -- src/main/java/Account.java  # history of a specific file
git log -p -- src/main/java/Account.java  # history with diffs
git log --stat                       # show files changed per commit

# 5. View differences
git diff                             # unstaged changes (working dir vs index)
git diff --staged                    # staged changes (index vs last commit)
git diff HEAD                        # all changes (working dir vs last commit)
git diff main..feature-x             # differences between branches
git diff HEAD~3..HEAD                # changes in the last 3 commits
git diff --stat                      # summary of changes (files and line counts)
```

### Branching

```bash
# Create and switch to a new branch
git branch feature/transfer-api       # create branch (stay on current)
git checkout feature/transfer-api     # switch to it
git checkout -b feature/transfer-api  # create AND switch (shorthand)
git switch -c feature/transfer-api    # modern equivalent (Git 2.23+)

# List branches
git branch                  # local branches (* = current)
git branch -a               # all branches (local + remote)
git branch -r               # remote branches only
git branch -v               # with last commit info
git branch --merged         # branches already merged into current
git branch --no-merged      # branches NOT yet merged

# Delete branches
git branch -d feature/transfer-api   # delete (safe: only if merged)
git branch -D feature/transfer-api   # force delete (even if unmerged)
git push origin --delete feature/transfer-api  # delete remote branch

# Rename branch
git branch -m old-name new-name

# Switch branches
git checkout main
git switch main              # modern equivalent
git checkout -               # switch to previous branch
git switch -                 # modern equivalent

# Stash changes (when you need to switch branches but have uncommitted work)
git stash                    # save working changes and revert to clean state
git stash save "WIP: transfer validation"  # with a message
git stash list               # list all stashes
git stash pop                # apply most recent stash and remove it
git stash apply              # apply but keep in stash list
git stash drop stash@{0}     # delete a specific stash
git stash clear              # delete all stashes
```

### Merging

```bash
# Merge a branch into the current branch
git checkout main
git merge feature/transfer-api

# Merge strategies
git merge --no-ff feature/transfer-api   # force merge commit (even if fast-forward possible)
git merge --squash feature/transfer-api  # squash all commits into one staged change
git merge --abort                        # cancel an in-progress merge

# Fast-forward merge (when main has no new commits since the branch point):
# main:    A—B—C
# feature:      C—D—E
# After merge: A—B—C—D—E  (main pointer moves to E, no merge commit)

# Non-fast-forward (when main has new commits):
# main:    A—B—C—F
# feature:      C—D—E
# After merge: A—B—C—F—M  (M is merge commit with parents F and E)
#                     ↗
#                D—E
```

### Rebasing

```bash
# Rebase current branch onto main
git checkout feature/transfer-api
git rebase main
# Replays your feature commits on top of the latest main

# Interactive rebase (clean up your commits before merging)
git rebase -i HEAD~5
# Opens an editor with the last 5 commits:
# pick abc1234 Add transfer endpoint
# pick def5678 Fix validation bug
# pick ghi9012 Add tests
# pick jkl3456 Fix typo in error message
# pick mno7890 Update documentation
#
# Commands:
# pick   → use commit as-is
# reword → use commit but edit message
# edit   → pause for amending
# squash → combine with previous commit (keep messages)
# fixup  → combine with previous commit (discard message)
# drop   → remove commit entirely

# Abort a rebase in progress
git rebase --abort

# Continue after resolving conflicts during rebase
git rebase --continue
```

### Resolving Merge Conflicts

```bash
# When a conflict occurs, Git marks the conflicting sections in the file:
# <<<<<<< HEAD
# BigDecimal fee = new BigDecimal("0.02");
# =======
# BigDecimal fee = new BigDecimal("0.015");
# >>>>>>> feature/transfer-api

# Steps to resolve:
# 1. Open the conflicted file(s)
git status   # shows "Unmerged paths"

# 2. Edit the file to resolve the conflict (choose one side, combine, or write new code)
# Remove the <<<<<<<, =======, and >>>>>>> markers

# 3. Stage the resolved file
git add src/main/java/TransferService.java

# 4. Complete the merge or rebase
git commit              # for merge (Git auto-generates the merge message)
git rebase --continue   # for rebase

# Useful tools for conflict resolution:
git mergetool           # open a visual merge tool (configure with git config)
git diff --name-only --diff-filter=U  # list only conflicted files

# Abort if the conflict is too complex
git merge --abort
git rebase --abort
```

### Remote Operations

```bash
# View remotes
git remote -v

# Add a remote
git remote add upstream https://github.com/original-org/banking-api.git

# Fetch (download without merging)
git fetch origin              # fetch from origin
git fetch --all               # fetch from all remotes
git fetch origin main         # fetch specific branch

# Pull (fetch + merge/rebase)
git pull                      # pull current branch from its tracking remote
git pull origin main          # pull specific branch
git pull --rebase             # pull with rebase instead of merge

# Push
git push origin main                      # push main to origin
git push origin feature/transfer-api      # push feature branch
git push -u origin feature/transfer-api   # push and set upstream tracking
git push --force-with-lease               # force push (safer than --force)
# ⚠️  NEVER use git push --force on shared branches (main, develop)
# ⚠️  --force-with-lease is safer: it fails if the remote has new commits you haven't seen

# Tracking branches
git branch --set-upstream-to=origin/feature/transfer-api feature/transfer-api
git branch -vv   # show tracking info for all branches
```

### Undoing Mistakes

```bash
# Undo changes in working directory (discard edits to a file)
git checkout -- src/main/java/Account.java   # old way
git restore src/main/java/Account.java       # modern way (Git 2.23+)

# Unstage a file (remove from index, keep working changes)
git reset HEAD src/main/java/Account.java    # old way
git restore --staged src/main/java/Account.java  # modern way

# Undo the last commit (keep changes staged)
git reset --soft HEAD~1

# Undo the last commit (keep changes in working directory, unstaged)
git reset --mixed HEAD~1   # default behavior of git reset HEAD~1

# Undo the last commit AND discard all changes (DESTRUCTIVE)
git reset --hard HEAD~1
# ⚠️  This permanently deletes the commit and all changes. Use with extreme caution.

# Recover a "lost" commit using reflog
git reflog                    # shows all HEAD movements, even after reset
git reflog show HEAD@{5}      # show a specific reflog entry
git reset --hard HEAD@{5}     # recover to that state
# Reflog entries expire after 90 days by default

# Revert a commit (create a new commit that undoes the changes — safe for shared branches)
git revert abc1234            # revert a specific commit
git revert HEAD               # revert the last commit
git revert --no-commit HEAD~3..HEAD  # revert multiple commits into one staged change
```

### Tags

```bash
# Create tags
git tag v1.0.0                          # lightweight tag
git tag -a v1.0.0 -m "Release 1.0.0"   # annotated tag (preferred)
git tag -a v1.0.1 abc1234 -m "Hotfix"  # tag a specific commit

# List tags
git tag                     # all tags
git tag -l "v1.*"          # pattern match
git tag -n                  # with annotation messages

# Push tags
git push origin v1.0.0      # push specific tag
git push origin --tags      # push all tags

# Delete tags
git tag -d v1.0.0           # delete local tag
git push origin --delete v1.0.0  # delete remote tag

# Checkout a tag (detached HEAD)
git checkout v1.0.0
git switch --detach v1.0.0
```

### Inspecting History

```bash
# Show a specific commit
git show abc1234
git show HEAD              # last commit
git show HEAD~3            # 3 commits ago
git show v1.0.0            # tagged commit

# Blame: who last modified each line of a file
git blame src/main/java/Account.java
git blame -L 10,20 src/main/java/Account.java  # specific line range
git blame -w src/main/java/Account.java         # ignore whitespace changes

# Search commit messages
git log --grep="fix"                # commits with "fix" in the message
git log --grep="BUG-1234"           # commits referencing a ticket

# Search code changes (which commit introduced a specific line?)
git log -S "BigDecimal.ZERO" -- src/main/java/  # commits that added/removed this string
git log -G "balance.*compareTo" -- src/          # commits where this regex matches the diff

# Find when a bug was introduced (binary search through history)
git bisect start
git bisect bad HEAD          # current version is broken
git bisect good v1.0.0       # this version was working
# Git checks out the middle commit. Test it, then:
git bisect good              # or: git bisect bad
# Repeat until Git identifies the exact commit that introduced the bug
git bisect reset             # return to original branch
```

### Java-Specific .gitignore

```gitignore
# --- Maven ---
target/
!.mvn/wrapper/maven-wrapper.jar
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties

# --- Gradle ---
.gradle/
build/
!gradle/wrapper/gradle-wrapper.jar
!**/src/main/**/build/
!**/src/test/**/build/

# --- IDE: IntelliJ IDEA ---
.idea/
*.iml
*.iws
*.ipr
out/
!**/src/main/**/out/
!**/src/test/**/out/

# --- IDE: Eclipse ---
.classpath
.project
.settings/
bin/

# --- IDE: VS Code ---
.vscode/
*.code-workspace

# --- OS ---
.DS_Store
Thumbs.db
*.swp
*.swo
*~

# --- Environment ---
.env
.env.local
.env.*.local
*.log

# --- Compiled ---
*.class
*.jar
*.war
*.ear
*.nar

# --- Testing ---
hs_err_pid*
replay_pid*

# --- Secrets (CRITICAL) ---
*.pem
*.key
*.p12
*.jks
keystore
credentials.json
```

---

## Important Notes

### Conventional Commits

Conventional Commits is a **standardized commit message format** that makes history readable, enables automated changelogs, and supports semantic versioning:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Types:**

```
feat:     → new feature (triggers minor version bump)
fix:      → bug fix (triggers patch version bump)
docs:     → documentation only
style:    → formatting, semicolons, whitespace (no code change)
refactor: → code change that neither fixes a bug nor adds a feature
perf:     → performance improvement
test:     → adding or correcting tests
build:    → build system or dependency changes (Maven, Gradle)
ci:       → CI/CD configuration changes (GitHub Actions, Jenkins)
chore:    → maintenance tasks, no production code change
revert:   → reverts a previous commit
```

**Examples for a Java banking API:**

```
feat(transfer): add international wire transfer endpoint
fix(account): prevent overdraft when concurrent withdrawals occur
docs(api): update OpenAPI spec for transaction history endpoint
refactor(payment): extract fee calculation into separate service
perf(query): add database index for transaction date range queries
test(transfer): add integration tests for cross-currency transfers
build(deps): upgrade Spring Boot to 3.2.3
ci(pipeline): add Testcontainers step to GitHub Actions
chore(config): update logback configuration for production
revert: revert "feat(transfer): add international wire transfer endpoint"
```

**Breaking changes** are indicated by `!` or a `BREAKING CHANGE:` footer:

```
feat(api)!: change transaction response format to RFC 7807 ProblemDetail

BREAKING CHANGE: TransactionResponse no longer includes the raw error field.
Use the new 'detail' and 'instance' fields instead.
```

**Rules:**
```
- Use imperative mood: "add feature" not "added feature" or "adds feature"
- Do not capitalize the first letter of the description
- No period at the end of the description
- Keep the first line under 72 characters
- Use the body for "why", not "what" (the diff shows "what")
- Reference tickets: "fix(account): prevent overdraft (#1234)"
```

### Branching Strategies

**Git Flow** (traditional, good for release cycles):

```
main ─────────●─────────────────●──── (production releases)
             ↗                 ↗
release/1.0 ───●───●───●───●───
              ↗           ↗
develop ──●──●──●──●──●──●──●── (integration branch)
         ↗      ↗        ↗
feature/ ──●──●  feature/ ──●──
```

```
Branches:
  main        → production-ready code, tagged with releases
  develop     → integration branch for features
  feature/*   → individual features (branched from develop)
  release/*   → release preparation (branched from develop, merged to main + develop)
  hotfix/*    → emergency fixes (branched from main, merged to main + develop)
```

**GitHub Flow** (simpler, good for continuous deployment):

```
main ──●──●──●──●──●──●──●── (always deployable)
      ↗      ↗        ↗
feature/ ──●  feature/ ──●──
```

```
Branches:
  main        → always deployable, protected
  feature/*   → short-lived feature branches (branched from main, merged via PR)
```

**Trunk-Based Development** (advanced, used by high-velocity teams):

```
main ──●──●──●──●──●──●──●── (everyone commits here frequently)
      ↗   ↗   ↗
  short-lived branches (< 1 day)
```

```
Rules:
  → Everyone commits to main (or very short-lived branches) multiple times per day
  → Feature flags control visibility of incomplete features
  → Requires strong CI/CD and test coverage
  → Common in fintech for teams practicing continuous delivery
```

**Recommendation for this roadmap:** Start with **GitHub Flow**. It is simple, works well with Spring Boot projects, and maps directly to the PR workflow you will use in professional settings.

### Pull Request Workflow

```
1. Create a feature branch from main:
   git checkout main && git pull
   git checkout -b feature/transfer-api

2. Make changes, commit frequently with conventional commits:
   git add . && git commit -m "feat(transfer): add TransferController"

3. Push the branch to GitHub:
   git push -u origin feature/transfer-api

4. Open a Pull Request on GitHub:
   → Title: feat(transfer): add international transfer API
   → Description: what, why, how to test, screenshots if applicable
   → Assign reviewers
   → Link related issues

5. Address review feedback:
   → Make changes locally
   → git add . && git commit -m "fix(transfer): address review comments"
   → git push (PR updates automatically)

6. After approval, merge:
   → Squash merge (preferred for clean history):
     All feature commits → single commit on main
   → Merge commit (preserves branch history)
   → Rebase merge (linear history, rebases feature onto main)

7. Clean up:
   → Delete the feature branch (GitHub can do this automatically)
   → git checkout main && git pull
   → git branch -d feature/transfer-api
```

### SSH Setup for GitHub

```bash
# 1. Generate an SSH key pair
ssh-keygen -t ed25519 -C "alice@example.com"
# Accept default location (~/.ssh/id_ed25519)
# Set a strong passphrase

# 2. Start the SSH agent and add your key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. Copy the public key to clipboard
cat ~/.ssh/id_ed25519.pub | pbcopy    # macOS
cat ~/.ssh/id_ed25519.pub | xclip     # Linux (xclip must be installed)

# 4. Add to GitHub:
#    Settings → SSH and GPG keys → New SSH key
#    Paste the public key

# 5. Test the connection
ssh -T git@github.com
# Expected: "Hi alice! You've successfully authenticated..."

# 6. Switch existing repos from HTTPS to SSH
git remote set-url origin git@github.com:alice/banking-api.git

# 7. Configure SSH for multiple GitHub accounts (if needed)
# ~/.ssh/config:
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work

# Clone with specific account:
git clone git@github-work:company/banking-api.git
```

### Git Hooks

Git hooks are scripts that run automatically at specific points in the Git workflow:

```bash
# Hooks live in .git/hooks/ (not version-controlled by default)
# Use a tool like Husky (for Node) or pre-commit (Python) to manage them
# For Java, the pre-commit framework works well:

# .pre-commit-config.yaml:
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-merge-conflict
  - repo: local
    hooks:
      - id: checkstyle
        name: Checkstyle
        entry: ./mvnw checkstyle:check
        language: system
        pass_filenames: false
        stages: [commit]
      - id: compile
        name: Compile
        entry: ./mvnw compile -q
        language: system
        pass_filenames: false
        stages: [commit]

# Common hooks for Java projects:
# pre-commit:  run checkstyle, compile, unit tests
# commit-msg:  validate conventional commit format
# pre-push:    run full test suite
# post-merge:  run mvn clean install after pulling changes
```

### GitHub-Specific Features

```
Branch Protection Rules (Settings → Branches):
  → Require pull request reviews before merging
  → Require status checks to pass (CI must be green)
  → Require branches to be up to date before merging
  → Require signed commits
  → Restrict who can push to main
  → CRITICAL for any team project — protect main from the start

GitHub Actions (CI/CD):
  → .github/workflows/ci.yml
  → Triggers on push, pull_request, schedule
  → Runs tests, builds, deploys
  → Covered in detail in [[07.03 — CI/CD]]

GitHub Issues and Projects:
  → Track bugs, features, tasks
  → Link to PRs with keywords: "Fixes #123", "Closes #456"
  → Project boards for Kanban-style tracking

GitHub Releases:
  → Create from tags
  → Auto-generate release notes from PRs
  → Attach build artifacts (JAR files, Docker images)

CODEOWNERS file (.github/CODEOWNERS):
  → Define who must review changes to specific paths
  → /src/main/java/com/example/payment/ @payment-team
  → /pom.xml @tech-leads

Dependabot:
  → Automated dependency update PRs
  → Critical for Java security (Log4j, Spring vulnerabilities)
  → Configure in .github/dependabot.yml
```

### Common Anti-Patterns

```
1. Committing generated files:
   → NEVER commit target/, build/, .class, .jar files
   → Use .gitignore (see the Java-specific template above)
   → If accidentally committed: git rm -r --cached target/

2. Large binary files in Git:
   → Git is terrible at tracking large binaries (JARs, images, datasets)
   → Use Git LFS (Large File Storage) if absolutely necessary
   → Better: store binaries in S3/Artifactory, reference by URL

3. Committing secrets:
   → NEVER commit API keys, passwords, private keys, .env files
   → Use .gitignore for .env, *.pem, *.key, *.jks
   → If accidentally committed: rotate the secret IMMEDIATELY
     (removing from Git history does not help — it is in the reflog and forks)
   → Use git-filter-repo or BFG Repo Cleaner to purge from history

4. Giant commits:
   → A commit should be one logical change
   → "Add transfer API, fix login bug, update README, refactor database"
     should be 4 separate commits
   → Small commits are easier to review, revert, and bisect

5. Vague commit messages:
   → "fix stuff", "update code", "WIP", "changes" are useless
   → Future you will curse past you when git-blaming at 2 AM

6. Force pushing to shared branches:
   → git push --force on main will destroy your teammates' work
   → Use --force-with-lease on your own feature branches only

7. Ignoring merge conflicts:
   → Never resolve conflicts by blindly accepting "ours" or "theirs"
   → Read the conflict markers, understand both sides, write the correct resolution
   → Test after resolving conflicts before committing

8. Not pulling before pushing:
   → If the remote has new commits, your push will be rejected
   → git pull --rebase before pushing to integrate remote changes cleanly
```

---

## Practice

1. Initialize a new Git repository for a Maven project and create a proper .gitignore
2. Make 5 commits with conventional commit messages for different types of changes
3. Create a feature branch, make changes, and merge it back to main with --no-ff
4. Create a feature branch, make changes, and rebase it onto main before merging
5. Intentionally create a merge conflict between two branches and resolve it manually
6. Use git stash to save work, switch branches, and restore the stashed changes
7. Use git log -S to find which commit introduced a specific method in your code
8. Use git bisect to find a bug-introducing commit in a 20-commit history
9. Use interactive rebase (git rebase -i) to squash 5 small commits into 1 clean commit
10. Use git reflog to recover a commit you accidentally deleted with git reset --hard
11. Set up SSH authentication with GitHub and clone a repository using SSH
12. Create a GitHub repository with branch protection rules requiring PR reviews
13. Open a pull request from a feature branch, request a review, and squash-merge it
14. Write a commit-msg hook that validates conventional commit format
15. Use git blame to trace the history of a specific line in a Java file and understand why a particular design decision was made

---

## References

- Pro Git (free online book): https://git-scm.com/book/en/v2
- Git Reference Documentation: https://git-scm.com/docs
- Conventional Commits Specification: https://www.conventionalcommits.org/
- GitHub Docs: https://docs.github.com/
- GitHub Flow Guide: https://docs.github.com/en/get-started/using-github/github-flow
- Oh My Zsh Git Plugin (aliases): https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git
- "Git from the Bottom Up" — John Wiegley: https://jwiegley.github.io/git-from-the-bottom-up/
- Visualizing Git (interactive): https://git-school.github.io/visualizing-git/
