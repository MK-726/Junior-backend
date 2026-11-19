\# Git Cheatsheet (20/80 Rule)

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

```svg
<svg width="720" height="220" xmlns="http://www.w3.org/2000/svg" font-family="system-ui, -apple-system, sans-serif">
  <!-- Working Directory -->
  <rect x="20" y="40" width="180" height="140" rx="12" fill="#f0f9ff" stroke="#0ea5e9" stroke-width="2.5"/>
  <text x="110" y="75" text-anchor="middle" font-size="16" font-weight="700" fill="#0369a1">Working Directory</text>
  <text x="110" y="100" text-anchor="middle" font-size="13" fill="#334155">Your actual files</text>
  <text x="110" y="120" text-anchor="middle" font-size="13" fill="#334155">on disk</text>
  <text x="110" y="150" text-anchor="middle" font-size="12" fill="#64748b">(edit here)</text>

  <!-- Arrow 1 -->
  <path d="M210 110 L250 110" stroke="#64748b" stroke-width="2.5" marker-end="url(#arrow)"/>
  <text x="230" y="100" text-anchor="middle" font-size="12" fill="#475569">git add</text>

  <!-- Staging Area -->
  <rect x="260" y="40" width="180" height="140" rx="12" fill="#f0fdf4" stroke="#22c55e" stroke-width="2.5"/>
  <text x="350" y="75" text-anchor="middle" font-size="16" font-weight="700" fill="#15803d">Staging Area</text>
  <text x="350" y="100" text-anchor="middle" font-size="13" fill="#334155">Files ready</text>
  <text x="350" y="120" text-anchor="middle" font-size="13" fill="#334155">to be committed</text>
  <text x="350" y="150" text-anchor="middle" font-size="12" fill="#64748b">(git add)</text>

  <!-- Arrow 2 -->
  <path d="M450 110 L490 110" stroke="#64748b" stroke-width="2.5" marker-end="url(#arrow)"/>
  <text x="470" y="100" text-anchor="middle" font-size="12" fill="#475569">git commit</text>

  <!-- Repository -->
  <rect x="500" y="40" width="180" height="140" rx="12" fill="#fefce8" stroke="#eab308" stroke-width="2.5"/>
  <text x="590" y="75" text-anchor="middle" font-size="16" font-weight="700" fill="#a16207">Repository</text>
  <text x="590" y="100" text-anchor="middle" font-size="13" fill="#334155">Saved snapshots</text>
  <text x="590" y="120" text-anchor="middle" font-size="13" fill="#334155">(commits)</text>
  <text x="590" y="150" text-anchor="middle" font-size="12" fill="#64748b">(permanent)</text>

  <defs>
    <marker id="arrow" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#64748b"/>
    </marker>
  </defs>
</svg>
```

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

```svg
<svg width="680" height="160" xmlns="http://www.w3.org/2000/svg" font-family="system-ui, -apple-system, sans-serif">
  <defs>
    <marker id="a" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto">
      <polygon points="0 0, 8 3, 0 6" fill="#475569"/>
    </marker>
  </defs>

  <!-- Boxes -->
  <rect x="20"  y="50" width="120" height="60" rx="10" fill="#e0f2fe" stroke="#0284c7" stroke-width="2"/>
  <text x="80"  y="85" text-anchor="middle" font-size="14" font-weight="600" fill="#0c4a6e">Edit files</text>

  <rect x="190" y="50" width="120" height="60" rx="10" fill="#dcfce7" stroke="#16a34a" stroke-width="2"/>
  <text x="250" y="85" text-anchor="middle" font-size="14" font-weight="600" fill="#14532d">git add</text>

  <rect x="360" y="50" width="120" height="60" rx="10" fill="#fef9c3" stroke="#ca8a04" stroke-width="2"/>
  <text x="420" y="85" text-anchor="middle" font-size="14" font-weight="600" fill="#713f12">git commit</text>

  <rect x="530" y="50" width="120" height="60" rx="10" fill="#f3e8ff" stroke="#9333ea" stroke-width="2"/>
  <text x="590" y="85" text-anchor="middle" font-size="14" font-weight="600" fill="#581c87">git push</text>

  <!-- Arrows -->
  <path d="M145 80 L185 80" stroke="#475569" stroke-width="2" marker-end="url(#a)"/>
  <path d="M315 80 L355 80" stroke="#475569" stroke-width="2" marker-end="url(#a)"/>
  <path d="M485 80 L525 80" stroke="#475569" stroke-width="2" marker-end="url(#a)"/>
</svg>
```

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

```svg
<svg width="700" height="260" xmlns="http://www.w3.org/2000/svg" font-family="system-ui, -apple-system, sans-serif">
  <!-- Main line -->
  <line x1="40" y1="80" x2="660" y2="80" stroke="#94a3b8" stroke-width="3"/>
  
  <!-- Commits on main -->
  <circle cx="80"  cy="80" r="12" fill="#3b82f6"/>
  <circle cx="180" cy="80" r="12" fill="#3b82f6"/>
  <circle cx="280" cy="80" r="12" fill="#3b82f6"/>
  <circle cx="480" cy="80" r="12" fill="#3b82f6"/>
  <circle cx="600" cy="80" r="12" fill="#3b82f6"/>

  <!-- Branch line -->
  <path d="M280 80 Q 320 80 340 140" stroke="#f59e0b" stroke-width="3" fill="none"/>
  <line x1="340" y1="140" x2="520" y2="140" stroke="#f59e0b" stroke-width="3"/>
  
  <!-- Commits on feature branch -->
  <circle cx="380" cy="140" r="12" fill="#f59e0b"/>
  <circle cx="480" cy="140" r="12" fill="#f59e0b"/>

  <!-- Labels -->
  <text x="80"  y="50" text-anchor="middle" font-size="13" fill="#1e40af" font-weight="600">main</text>
  <text x="180" y="50" text-anchor="middle" font-size="12" fill="#64748b">C1</text>
  <text x="280" y="50" text-anchor="middle" font-size="12" fill="#64748b">C2</text>
  <text x="480" y="50" text-anchor="middle" font-size="12" fill="#64748b">C4</text>
  <text x="600" y="50" text-anchor="middle" font-size="12" fill="#64748b">C5</text>

  <text x="430" y="175" text-anchor="middle" font-size="13" fill="#b45309" font-weight="600">feature branch</text>
  <text x="380" y="175" text-anchor="middle" font-size="12" fill="#64748b">C3a</text>
  <text x="480" y="175" text-anchor="middle" font-size="12" fill="#64748b">C3b</text>

  <!-- Explanation -->
  <text x="350" y="230" text-anchor="middle" font-size="14" fill="#334155">
    You create a branch → work safely → merge back into main when ready
  </text>
</svg>
```

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

```svg
<svg width="640" height="200" xmlns="http://www.w3.org/2000/svg" font-family="system-ui, -apple-system, sans-serif">
  <!-- Local box -->
  <rect x="30" y="30" width="240" height="140" rx="12" fill="#f0f9ff" stroke="#0ea5e9" stroke-width="2.5"/>
  <text x="150" y="60" text-anchor="middle" font-size="16" font-weight="700" fill="#0369a1">Your Computer</text>
  <text x="150" y="90" text-anchor="middle" font-size="13" fill="#334155">Local repository</text>
  <text x="150" y="115" text-anchor="middle" font-size="13" fill="#334155">+ working files</text>
  <text x="150" y="145" text-anchor="middle" font-size="12" fill="#64748b">git commit happens here</text>

  <!-- Arrows -->
  <path d="M280 80 L360 80" stroke="#64748b" stroke-width="2.5" marker-end="url(#arr)"/>
  <text x="320" y="70" text-anchor="middle" font-size="12" fill="#475569">git push</text>
  
  <path d="M360 120 L280 120" stroke="#64748b" stroke-width="2.5" marker-end="url(#arr)"/>
  <text x="320" y="145" text-anchor="middle" font-size="12" fill="#475569">git pull / fetch</text>

  <!-- Remote box -->
  <rect x="370" y="30" width="240" height="140" rx="12" fill="#faf5ff" stroke="#a855f7" stroke-width="2.5"/>
  <text x="490" y="60" text-anchor="middle" font-size="16" font-weight="700" fill="#7e22ce">GitHub / Remote</text>
  <text x="490" y="90" text-anchor="middle" font-size="13" fill="#334155">Shared repository</text>
  <text x="490" y="115" text-anchor="middle" font-size="13" fill="#334155">for the team</text>
  <text x="490" y="145" text-anchor="middle" font-size="12" fill="#64748b">source of truth</text>

  <defs>
    <marker id="arr" markerWidth="9" markerHeight="6" refX="8" refY="3" orient="auto">
      <polygon points="0 0, 9 3, 0 6" fill="#64748b"/>
    </marker>
  </defs>
</svg>
```

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
```
