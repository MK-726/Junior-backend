# Git Cheatsheet (20/80 Rule)

The 20% of Git commands that cover ~80% of everyday work.  
Simple language. Accurate diagrams. Useful flags included.

---

## 1. Configuring Git

Do this once on a new machine.

### Essential identity

```bash
git config --global user.name "Your Name"
git config --global user.email "you@email.com"
```

### Recommended settings

```bash
# Use main as the default branch name
git config --global init.defaultBranch main

# Always use a fast-forward only pull (safer, avoids unexpected merges)
git config --global pull.ff only

# Nice colored output
git config --global color.ui auto

# Default editor (examples)
git config --global core.editor "code --wait"   # VS Code
# git config --global core.editor "nvim"
# git config --global core.editor "nano"
```

### Useful aliases (optional but powerful)

```bash
git config --global alias.st "status -sb"
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.cm "commit -m"
git config --global alias.last "log -1 HEAD"
```

### Check your configuration

```bash
git config --list                 # all settings
git config --global --list        # only global
git config user.name              # check one value
```

> Settings levels:  
> `--local` (this repo only) → `--global` (your user) → `--system` (whole machine)

---

## 2. The three places where your code lives

Git always works with **three areas**:

<p align="center">
    <image alt="Git's three areas: working directory, staging area and repository" src="assets/three-areas.svg"/>
</p>

**Mental model**  

1. You edit files in the **Working Directory**  
2. You choose which changes to save with `git add` → they go to the **Staging Area**  
3. You take a snapshot with `git commit` → it goes into the **Repository**

---

## 3. Everyday workflow (the core loop)

```bash
# See what changed (short + branch info)
git status -sb

# Stage everything
git add .
# or stage interactively (pick hunks)
git add -p

# Save a snapshot
git commit -m "Short clear message"

# Better: open editor for a longer message
git commit

# Send to remote
git push

# Get latest from remote (safe version)
git pull --ff-only
```

### Visual of the daily loop

<p align="center">
    <image alt="Daily loop: edit, add, commit, pull, push" src="assets/daily-loop.svg" />
</p>

---

## 4. Starting a project

```bash
# New project from scratch
git init
git add .
git commit -m "Initial commit"

# Or start from an existing GitHub repo
git clone https://github.com/user/repo.git
cd repo
```

---

## 5. .gitignore – stop tracking junk

Create a file named `.gitignore` in the root of your project.

### Common patterns

```gitignore
# Dependencies
node_modules/
vendor/
.env
.env.local

# Build outputs
dist/
build/
*.o
*.exe

# IDE / Editor
.vscode/
.idea/
*.swp
.DS_Store

# Logs & temp
*.log
tmp/
temp/
.cache/

# OS files
Thumbs.db
```

### Useful commands

```bash
# Check what is ignored
git status --ignored

# Remove a file from tracking (but keep it on disk)
git rm --cached filename

# Remove a whole folder from tracking
git rm -r --cached foldername/
```

> Tip: Put `.gitignore` early. Once a file is committed, adding it to `.gitignore` does **not** untrack it. You need `git rm --cached`.

---

## 6. Branches (the most powerful feature)

A branch is just a movable label that points to a commit.

<p align="center">
    <image alt="A feature branch splits off main and merges back" src="assets/branches.svg" />
</p>

### Branch commands you actually need
```bash
# See all branches (* = current)
git branch

# Create + switch to a new branch
git switch -c feature/login

# Switch between branches
git switch main
git switch feature/login

# Merge a branch into the current one
git switch main
git merge feature/login

# Delete a branch after merging (safe)
git branch -d feature/login

# Force delete (even if not merged)
git branch -D feature/login
```

> Tip: Prefer `git switch` over the older `git checkout` for changing branches. It is clearer.

---

## 7. Looking at history & differences

```bash
# Compact history
git log --oneline

# Better history (graph + branches)
git log --oneline --graph --decorate --all

# See what changed (not yet staged)
git diff

# See what is staged
git diff --staged

# See changes in a specific commit
git show <commit-hash>

# Limit log to last 5 commits
git log -5 --oneline
```

---

## 8. Undoing things (safe first)

| Situation                        | Command                          | Risk     |
|----------------------------------|----------------------------------|----------|
| Unstage a file                   | `git restore --staged file`      | Safe     |
| Discard changes in working file  | `git restore file`               | Loses work |
| Undo last commit (keep changes)  | `git reset --soft HEAD~1`        | Medium   |
| Change last commit message       | `git commit --amend -m "new msg"`| Medium*  |
| Create a new commit that undoes  | `git revert <commit>`            | Safest for shared history |

\* Only amend if you have **not** pushed yet.

**Rule of thumb**  

- Already pushed? Prefer `git revert`  
- Only local? `git reset` or `--amend` is fine

---

## 9. Working with remote (GitHub / GitLab)

```bash
# See remotes
git remote -v

# First time pushing a new branch (sets upstream)
git push -u origin feature/login

# After that, just
git push

# Get latest changes without merging
git fetch

# Get + fast-forward only (recommended)
git pull --ff-only

# If you prefer rebase instead of merge when pulling
git pull --rebase
```

### Local vs Remote picture

<p align="center">
    <image alt="Local versus remote: push, fetch and pull" src="assets/local-vs-remote.svg" />
</p>

---

## 10. Quick reference card

| Goal                              | Command                                      |
|-----------------------------------|----------------------------------------------|
| New repo                          | `git init`                                   |
| Download existing repo            | `git clone <url>`                            |
| What's going on? (short)          | `git status -sb`                             |
| Stage all changes                 | `git add .`                                  |
| Stage interactively               | `git add -p`                                 |
| Commit                            | `git commit -m "message"`                    |
| Amend last commit                 | `git commit --amend`                         |
| Push to remote                    | `git push`                                   |
| Push new branch + set upstream    | `git push -u origin branch`                  |
| Pull (safe)                       | `git pull --ff-only`                         |
| Create + switch branch            | `git switch -c name`                         |
| Switch branch                     | `git switch name`                            |
| Merge branch                      | `git merge name`                             |
| Delete branch (safe)              | `git branch -d name`                         |
| See history                       | `git log --oneline --graph --decorate`       |
| See unstaged changes              | `git diff`                                   |
| See staged changes                | `git diff --staged`                          |
| Unstage a file                    | `git restore --staged file`                  |
| Discard local changes             | `git restore file`                           |
| Remove from tracking (keep file)  | `git rm --cached file`                       |

---

## 11. Good habits

1. **Commit often** with clear messages  
2. **Pull before you push** (`git pull --ff-only`)  
3. Work on **feature branches**, not directly on `main`  
4. Keep commits small and focused  
5. Never force-push to `main` / `master` unless you really know what you're doing  
6. Put a good `.gitignore` in place from day one  
7. Use `git status -sb` and `git log --oneline --graph` often — they give great overview

---

*Made for people who want the useful 20%, not the entire manual.*
