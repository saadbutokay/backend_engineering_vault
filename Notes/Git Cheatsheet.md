## Setup & Config
```bash
# Identity (required before first commit)
git config --global user.name  "Your Name"
git config --global user.email "you@email.com"

# Default editor
git config --global core.editor "code --wait"    # VS Code
git config --global core.editor "vim"

# Default branch name
git config --global init.defaultBranch main

# Better diff output
git config --global core.pager "less -FX"

# View all config
git config --list
git config --global --list

# Where config is stored
# --global  →  ~/.gitconfig
# --local   →  .git/config (per project)
# --system  →  /etc/gitconfig
```

---
## Starting a Repo
```bash
git init                         # Initialize new repo in current folder
git init my_project              # Create new folder + initialize
git clone <url>                  # Clone remote repo
git clone <url> my_folder        # Clone into specific folder name
git clone --depth 1 <url>        # Shallow clone (latest snapshot only, faster)
git clone -b develop <url>       # Clone specific branch
```

---
## Snapshotting (The Core Loop)
```bash
# ─── CHECK STATUS ─────────────────────────────────────
git status                       # Full status
git status -s                    # Short/compact status

# ─── STAGING (git add) ────────────────────────────────
git add file.py                  # Stage one file
git add folder/                  # Stage entire folder
git add .                        # Stage ALL changes in current dir
git add -A                       # Stage ALL changes (entire repo)
git add -p                       # Interactively stage chunks (patch mode)
git add *.py                     # Stage all .py files

# ─── UNSTAGING ────────────────────────────────────────
git restore --staged file.py     # Unstage file (keep changes)
git reset HEAD file.py           # Older way to unstage

# ─── COMMITTING ───────────────────────────────────────
git commit -m "Your message"     # Commit with message
git commit                       # Opens editor for message
git commit -am "message"         # Stage tracked files + commit (skip add)
git commit --amend               # Edit last commit (message or content)
git commit --amend --no-edit     # Amend without changing message

# ─── DISCARDING CHANGES ───────────────────────────────
git restore file.py              # Discard working dir changes (irreversible!)
git restore .                    # Discard ALL working dir changes
git clean -fd                    # Remove untracked files + folders
git clean -n                     # Dry run (preview what would be removed)
```

---
## Status Symbols Explained
```
git status -s output:

?? file.py        → Untracked (git doesn't know about it)
A  file.py        → Staged (new file added)
M  file.py        → Modified + staged
 M file.py        → Modified but NOT staged
MM file.py        → Modified, partially staged
D  file.py        → Deleted + staged
 D file.py        → Deleted but NOT staged
R  old → new      → Renamed
```

---
## Branching
```bash
# ─── VIEWING BRANCHES ─────────────────────────────────
git branch                       # List local branches
git branch -r                    # List remote branches
git branch -a                    # List all (local + remote)
git branch -v                    # List with last commit info

# ─── CREATING BRANCHES ────────────────────────────────
git branch feature-login         # Create branch (don't switch)
git checkout -b feature-login    # Create + switch (classic)
git switch -c feature-login      # Create + switch (modern ✅)
git switch -c feat --track origin/feat  # Create tracking remote branch

# ─── SWITCHING BRANCHES ───────────────────────────────
git checkout main                # Switch (classic)
git switch main                  # Switch (modern ✅)
git switch -                     # Switch to previous branch

# ─── RENAMING / DELETING ──────────────────────────────
git branch -m old-name new-name  # Rename branch
git branch -d feature-login      # Delete (safe, merged only)
git branch -D feature-login      # Delete (force, even unmerged)
git push origin --delete feature # Delete remote branch
```

---
## Merging
```bash
git switch main
git merge feature-login          # Merge feature into main
git merge --no-ff feature-login  # Force merge commit (no fast-forward)
git merge --squash feature-login # Squash all commits into one
git merge --abort                # Abort a conflicted merge

# ─── AFTER A CONFLICT ─────────────────────────────────
# 1. Open conflicted files, look for:
#    <<<<<<< HEAD
#    your changes
#    =======
#    their changes
#    >>>>>>> feature-login
#
# 2. Edit file to desired state
# 3. git add file.py
# 4. git commit
```

---
## Rebasing
```bash
git rebase main                  # Rebase current branch onto main
git rebase -i HEAD~3             # Interactive rebase last 3 commits
git rebase --abort               # Abort rebase
git rebase --continue            # Continue after fixing conflict

# Interactive rebase commands (in editor):
# pick   → keep commit as-is
# reword → keep commit, edit message
# edit   → pause and amend commit
# squash → melt into previous commit (keep message)
# fixup  → melt into previous commit (discard message)
# drop   → delete commit entirely
```

```
MERGE vs REBASE:

MERGE:                              REBASE:
                                    
A---B---C  main                     A---B---C  main
     \                                           \
      D---E  feature                              D'--E'  feature
           \                                    
A---B---C---M  (M = merge commit)   A---B---C---D'---E'
                                    (linear, clean history)

✅ Use MERGE for: shared/public branches
✅ Use REBASE for: local cleanup, feature branches
⚠️  Never rebase shared/public branches
```

---
## Remote Operations
```bash
# ─── MANAGING REMOTES ─────────────────────────────────
git remote -v                         # List remotes
git remote add origin <url>           # Add remote
git remote rename origin upstream     # Rename remote
git remote remove origin              # Remove remote
git remote set-url origin <new-url>   # Change URL

# ─── FETCHING ─────────────────────────────────────────
git fetch                             # Fetch all remotes
git fetch origin                      # Fetch specific remote
git fetch origin main                 # Fetch specific branch
# Note: fetch downloads but does NOT merge

# ─── PULLING ──────────────────────────────────────────
git pull                              # fetch + merge
git pull origin main                  # Pull specific branch
git pull --rebase                     # fetch + rebase (cleaner)
git pull --rebase origin main

# ─── PUSHING ──────────────────────────────────────────
git push origin main                  # Push to remote
git push                              # Push (if tracking set)
git push -u origin feature-login      # Push + set tracking
git push --force-with-lease           # Safer force push ✅
git push --force                      # Force push (dangerous ⚠️)
git push origin --tags                # Push all tags
git push origin --delete feature      # Delete remote branch
```

---
## Viewing History & Diffs
```bash
# ─── LOG ──────────────────────────────────────────────
git log                          # Full log
git log --oneline                # Compact (one line per commit)
git log --oneline --graph        # Visual branch graph
git log --oneline --graph --all  # All branches graph
git log -n 5                     # Last 5 commits
git log --author="Alice"         # Filter by author
git log --since="2024-01-01"     # Filter by date
git log --grep="fix"             # Filter by message keyword
git log file.py                  # Commits that touched a file
git log -p file.py               # Commits + diffs for a file
git log --stat                   # Show file change stats

# ─── DIFF ─────────────────────────────────────────────
git diff                         # Unstaged changes
git diff --staged                # Staged changes (vs last commit)
git diff HEAD                    # All changes vs last commit
git diff main..feature           # Diff between branches
git diff abc123..def456          # Diff between commits
git diff HEAD~1 HEAD             # Last commit changes

# ─── INSPECTING ───────────────────────────────────────
git show abc1234                 # Show a commit's changes
git show HEAD                    # Show last commit
git blame file.py                # Who wrote each line
git blame -L 10,20 file.py       # Specific line range
```

---
## Undoing Things
```bash
# ─────────────────────────────────────────────────────────────────
#  SITUATION                HOW TO UNDO               SAFE?
# ─────────────────────────────────────────────────────────────────

# Unstaged change        → git restore file.py         ⚠️  irreversible
# Staged change          → git restore --staged file   ✅ safe
# Last commit (keep edits) → git reset --soft HEAD~1   ✅ safe
# Last commit (unstage)  → git reset --mixed HEAD~1    ✅ safe (default)
# Last commit (nuke it)  → git reset --hard HEAD~1     ⚠️  irreversible
# Old commit (public)    → git revert abc1234          ✅ safe (new commit)
# Whole file old version → git restore --source HEAD~2 file.py

# ─── RESET MODES EXPLAINED ────────────────────────────
git reset --soft HEAD~1   # Undo commit → changes go to STAGING
git reset --mixed HEAD~1  # Undo commit → changes go to WORKING DIR
git reset --hard HEAD~1   # Undo commit → changes are GONE 💀

# ─── REVERT (safe for shared branches) ────────────────
git revert abc1234         # Creates new commit that undoes abc1234
git revert HEAD            # Revert last commit
git revert HEAD~3..HEAD    # Revert last 3 commits
```

```
RESET vs REVERT:

BEFORE:   A ← B ← C ← D  (HEAD)

git reset --hard B:
AFTER:    A ← B  (HEAD)        ← C and D are GONE

git revert D:
AFTER:    A ← B ← C ← D ← D' ← (HEAD)
                              ↑ new commit that undoes D
```

---
## Tags
```bash
git tag                          # List all tags
git tag v1.0.0                   # Lightweight tag
git tag -a v1.0.0 -m "Release"  # Annotated tag (recommended)
git tag -a v1.0.0 abc1234        # Tag a specific commit
git push origin v1.0.0           # Push one tag
git push origin --tags           # Push all tags
git tag -d v1.0.0                # Delete local tag
git push origin --delete v1.0.0  # Delete remote tag
git checkout v1.0.0              # Checkout a tag (detached HEAD)
```

---

## Stashing
```bash
git stash                        # Stash all uncommitted changes
git stash push -m "wip: login"  # Stash with a name
git stash -u                     # Include untracked files
git stash list                   # Show all stashes
git stash pop                    # Apply latest + remove from stash
git stash apply stash@{2}        # Apply specific stash (keep in list)
git stash drop stash@{0}         # Delete specific stash
git stash clear                  # Delete ALL stashes
git stash show -p                # Show stash content
git stash branch feature stash@{0} # Create branch from stash
```

---
## Cherry Pick
```bash
git cherry-pick abc1234          # Copy commit onto current branch
git cherry-pick abc1234 def5678  # Multiple commits
git cherry-pick abc1234..def5678 # Range of commits
git cherry-pick --no-commit abc  # Apply changes without committing
git cherry-pick --abort          # Abort cherry-pick
```

---
## Searching
```bash
git grep "TODO"                  # Search in tracked files
git grep "TODO" -- "*.py"        # Search only .py files
git log -S "function_name"       # Find commits that added/removed text
git log -G "regex_pattern"       # Find commits matching regex
git bisect start                 # Start binary search for a bug
git bisect bad                   # Mark current commit as bad
git bisect good v1.0.0           # Mark known good commit
git bisect reset                 # End bisect session
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

### Common `.gitignore` for Python:

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
## Useful Aliases
```bash
# Add to ~/.gitconfig under [alias]
git config --global alias.st    "status -s"
git config --global alias.co    "checkout"
git config --global alias.sw    "switch"
git config --global alias.br    "branch"
git config --global alias.lg    "log --oneline --graph --all"
git config --global alias.last  "log -1 HEAD"
git config --global alias.undo  "reset --soft HEAD~1"
git config --global alias.unstage "restore --staged"

# Use them:
git st                           # git status -s
git lg                           # pretty graph log
git undo                         # undo last commit safely
```

---
## Typical Daily Workflow
```bash
# ─── MORNING: START WORK ──────────────────────────────
git switch main
git pull                         # Get latest changes

git switch -c feature/user-auth  # Create feature branch

# ─── DURING: WORKING ──────────────────────────────────
# ... edit files ...
git status                       # Check what changed
git diff                         # Review changes
git add -p                       # Stage selectively
git commit -m "feat: add login form"

# ─── SYNC WITH MAIN ───────────────────────────────────
git fetch origin
git rebase origin/main           # Keep branch up to date

# ─── DONE: OPEN PULL REQUEST ──────────────────────────
git push -u origin feature/user-auth
# → Go to GitHub and open PR

# ─── AFTER PR MERGED ──────────────────────────────────
git switch main
git pull
git branch -d feature/user-auth  # Cleanup local branch
```

---
## Emergency Commands
```bash
# "I committed to wrong branch!"
git reset --soft HEAD~1          # Undo commit, keep changes staged
git switch correct-branch
git commit -m "same message"

# "I need to save my work RIGHT NOW"
git stash -u                     # Stash everything including untracked

# "I broke everything, revert to last commit"
git restore .                    # Discard all unstaged changes
git clean -fd                    # Remove untracked files

# "I want to go back to how the repo looked 2 days ago"
git log --before="2 days ago" --oneline    # Find the commit hash
git checkout abc1234             # Detached HEAD to explore
git switch -c recovery-branch    # Save it as a branch

# "I accidentally deleted a branch"
git reflog                       # Find the commit hash
git switch -c recovered-branch abc1234

# "What did I just do?!"
git reflog                       # Full history of HEAD movements
```

---
## Command Danger Levels
```bash
# ✅SAFE    (can always undo)
├── git add, git status, git log, git diff
├── git commit, git stash
├── git fetch, git switch, git branch
└── git revert, git reset --soft, git reset --mixed

# ⚠️CAREFUL  (harder to undo)
├── git reset --hard
├── git restore  # (working dir)
├── git clean -fd
└── git merge, git rebase

# 💀DANGEROUS  (may lose work permanently)
├── git push --force       # use --force-with-lease instead
├── git reset --hard       # on shared branches
├── git clean -fdx         # also removes .gitignore'd files
└── git rebase             # on shared/public branches
```