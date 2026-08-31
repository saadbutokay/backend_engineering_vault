## `git` commands

```bash
# Setup and Configuration
git config --global user.name "Name"      # Set the name attached to your commits
git config --global user.email "Email"    # Set the email attached to your commits
git config --global core.editor "vim"     # Set your default text editor for commit messages
git config --list                         # List all current Git configuration settings
git init                                  # Create a new, empty Git repository in the current directory
git clone <url>                           # Download an existing repository from a remote server to your machine
```

```bash
# Basic Snapshotting
git status                                # Show modified files in working directory, staged for your next commit
git add <file>                            # Add a specific file's changes to the staging area
git add .                                 # Add all current directory changes to the staging area
git add -p                                # Interactively review and stage patches/chunks of code
git commit -m "Message"                   # Commit your staged changes with a descriptive message
git commit --amend                        # Add new staged changes to the previous commit, or edit its message
git rm <file>                             # Delete a file from your working directory and stage the deletion
git rm --cached <file>                    # Stop tracking a file but keep it in your local working directory
git mv <old_path> <new_path>              # Rename a file or move it, and stage the change
```

```bash
# undoing things
git restore <file>                        # Discard changes in the working directory (revert to last commit)
git restore --staged <file>               # Remove a file from the staging area, but keep the working directory changes
git revert <commit_hash>                  # Create a new commit that completely undoes a previous commit
git reset <commit_hash>                   # Move the current branch tip backward to a specific commit, leaving working directory intact
git reset --hard <commit_hash>            # Move the branch tip backward AND wipe out all uncommitted working directory changes
git clean -fd                             # Forcefully remove all untracked files and directories from your working tree
```

```bash
# branching & merging
git branch                                # List all local branches in the repository
git branch -a                             # List all local and remote-tracking branches
git branch <branch_name>                  # Create a new branch at the current commit
git branch -d <branch_name>               # Delete a local branch (fails if it contains unmerged changes)
git branch -D <branch_name>               # Forcefully delete a local branch regardless of its merge status
git switch <branch_name>                  # Switch to an existing branch
git switch -c <branch_name>               # Create a new branch and immediately switch to it
git checkout <branch_name>                # Older command to switch branches (often replaced by 'switch')
git checkout -b <branch_name>             # Older command to create and switch to a new branch
git merge <branch_name>                   # Combine the specified branch's history into your current branch
git rebase <base_branch>                  # Reapply your current branch's commits on top of another base branch
```

```bash
# sharing & updating
git remote -v                             # List all connected remote repositories and their URLs
git remote add <name> <url>               # Connect your local repository to a new remote server
git remote remove <name>                  # Disconnect a remote server from your local repository
git fetch <remote_name>                   # Download all new history from the remote, but do not merge it
git pull                                  # Download history from the remote and immediately merge it into your branch
git pull --rebase                         # Download history and reapply your local commits on top of the remote changes
git push <remote> <branch>                # Upload your local branch commits to the remote repository
git push -u <remote> <branch>             # Push and set the remote as the default upstream for future pulls/pushes
git push --force                          # Overwrite the remote history with your local history (use with caution)
```

```bash
# Inspection and History
git log                                   # Show the chronological commit history for the current branch
git log --oneline                         # Show the commit history compressed into a single line per commit
git log --graph                           # Draw a text-based graphical representation of the commit history and branches
git diff                                  # Show exact line-by-line changes between working directory and staging area
git diff --staged                         # Show exact changes between the staging area and the last commit
git diff <branch_1> <branch_2>            # Show the differences between two branches
git show <commit_hash>                    # Display the metadata and code changes introduced by a specific commit
git blame <file>                          # Show exactly who last modified each line of a file and in which commit
```

```bash
# temp storage (stashing)
git stash                                 # Temporarily save modified, tracked files without committing them
git stash -u                              # Temporarily save both tracked and untracked files
git stash list                            # Show all currently stashed changes
git stash pop                             # Apply the most recently stashed changes and remove them from the stash list
git stash apply                           # Apply the most recently stashed changes but keep them in the stash list
git stash drop                            # Delete the most recently stashed changes from the stash list
```

```bash
# Advance tools & debugging
git cherry-pick <commit_hash>             # Take the exact changes from a specific commit and apply them to your current branch
git tag <tag_name>                        # Create a lightweight tag (like a bookmark) at the current commit
git tag -a <tag_name> -m "Message"        # Create an annotated tag with a message (used for release versions)
git push --tags                           # Push all local tags to the remote repository
git reflog                                # Show a log of every movement of your branch tips (critical for recovering lost commits)
git bisect start                          # Start a binary search to find exactly which commit introduced a bug
git bisect good <commit_hash>             # Mark a past commit as "good" (bug-free) during a binary search
git bisect bad                            # Mark the current commit as "bad" (buggy) during a binary search
git archive --format=zip HEAD > out.zip   # Export the current state of the repository into a zip file without the .git folder
```

---
## `.gitignore`

```bash
# ─── PATTERNS ─────────────────────────────────────────
*.log                  # All .log files
*.py[cod]              # .pyc, .pyo, .pyd
build/                 # Entire folder
/config.json           # Only root-level config.json
!important.log         # Exception (do NOT ignore this)
**/temp                # temp folder anywhere in tree
doc/**/*.txt           # .txt inside doc/ (any depth)

# ─── COMMANDS ─────────────────────────────────────────
git check-ignore -v file.py      # Why is this file ignored?
git ls-files --ignored           # List all ignored files
git rm --cached file.py          # Stop tracking (keep on disk)
git rm --cached -r folder/       # Stop tracking folder
```

---
### Common `.gitignore` for Python

```bash
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.eggs/

# Virtual Environments
.venv/
venv/
env/

# Testing
.pytest_cache/
.coverage
htmlcov/

# Tools
.mypy_cache/
.ruff_cache/

# IDE
.vscode/
.idea/

# Environment
.env
.env.local

# OS
.DS_Store
Thumbs.db
```

---
### Global `.gitignore`

```bash
# System Files (OS specific)
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Common IDEs and Editors
# Visual Studio Code
.vscode/
!.vscode/extensions.json
*.code-workspace

# IntelliJ / JetBrains
.idea/
*.iml
*.iws

# Eclipse
.metadata/
.classpath
.project
.settings/

# Sublime Text / Vim
*.sublime-project
*.sublime-workspace
.*.swp
*~

# Logs and Databases
*.log
logs/
*.sqlite

# Local Environment / Secrets (Safety net)
.env
.env.local
.env.*.local
```

---