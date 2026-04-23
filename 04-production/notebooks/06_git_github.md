# Production 06: Git & GitHub Essentials

A comprehensive reference for version control with Git and GitHub.

---

## 1. Initial Setup

### Check if Git is Installed
```powershell
git --version
```

### Configure User Identity
```powershell
# Global (applies to all repos)
git config --global user.name "Your Name"
git config --global user.email "your-github-email@example.com"

# Local (this repo only - overrides global)
git config user.name "Work Name"
git config user.email "work@company.com"
```

### Check Current Config
```powershell
git config --global user.name
git config --global user.email
```

---

## 2. Creating a Repository

### Initialize Locally
```powershell
cd your-project-folder
git init
```

### Create Repo on GitHub via CLI
```powershell
# Install GitHub CLI
winget install GitHub.cli

# Login (one-time)
gh auth login

# Create and push in one command
gh repo create repo-name --public --source=. --remote=origin --push
```

---

## 3. The `.gitignore` File

### Why It Matters
- Prevents unnecessary files from being tracked
- Must be created **BEFORE** first `git add`

### Generate Automatically
```powershell
# Download Python-specific gitignore
iwr "https://www.toptal.com/developers/gitignore/api/python,windows,visualstudiocode,jupyternotebooks" -o .gitignore
```

### Fix: Already Tracked Files
```powershell
# Remove from Git but keep locally
git rm -r --cached .venv
git commit -m "chore: remove .venv from tracking"
git push
```

---

## 4. Basic Workflow

### Stage, Commit, Push
```powershell
git add .                           # Stage all changes
git commit -m "feat: add feature"   # Commit with message
git push                            # Push to remote
```

### Check Status
```powershell
git status                  # See uncommitted changes
git diff                    # See line-by-line changes
git diff --staged           # See staged changes
git log --oneline -5        # Last 5 commits
git show --stat             # Files in last commit
```

---

## 5. Commit Message Convention

### Format (Conventional Commits)
```
<type>(<scope>): <description>

[optional body]
```

### Types
| Type | Use Case |
|:---|:---|
| `feat:` | New feature |
| `fix:` | Bug fix |
| `docs:` | Documentation |
| `style:` | Formatting (no logic change) |
| `refactor:` | Code restructuring |
| `test:` | Adding tests |
| `chore:` | Maintenance tasks |

### Multi-Line Messages
```powershell
# Method 1: Multiple -m flags
git commit -m "feat: add auth" -m "- Added login" -m "- Added logout"

# Method 2: Opens editor
git commit
```

---

## 6. Branching

### Branch Naming Standards
| Pattern | Example |
|:---|:---|
| `main` | Production-ready (default) |
| `feature/<name>` | `feature/user-auth` |
| `fix/<name>` | `fix/login-bug` |
| `hotfix/<name>` | `hotfix/security-patch` |

### Commands
```powershell
git branch                     # List branches
git branch -M main             # Rename to main
git checkout -b feature/xyz    # Create and switch
git push -u origin main        # Push and set upstream
```

---

## 7. Remote Operations

### Add Remote
```powershell
git remote add origin https://github.com/user/repo.git
```

### Push to Remote
```powershell
git push -u origin main        # First push (sets upstream)
git push                       # Subsequent pushes
```

### URL Note
Both work identically:
- `https://github.com/user/repo.git`
- `https://github.com/user/repo`

---

## 8. Line Endings (Windows)

### The Warning
```
LF will be replaced by CRLF
```

### What It Means
- **LF** (`\n`): Unix/macOS
- **CRLF** (`\r\n`): Windows
- Git converts automatically—this is harmless

### Configure
```powershell
git config --global core.autocrlf true   # Windows default
git config --global core.autocrlf input  # Keep LF everywhere
```

---

## 9. Useful Commands Reference

| Command | Description |
|:---|:---|
| `git status` | Show working tree status |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Commit with message |
| `git push` | Push to remote |
| `git pull` | Fetch and merge from remote |
| `git log --oneline` | Compact commit history |
| `git diff` | Show unstaged changes |
| `git show --stat` | Show last commit details |
| `git rm --cached <file>` | Untrack file (keep locally) |

---

## 10. Quick Start Template

```powershell
# New project setup
cd my-project
git init
iwr "https://www.toptal.com/developers/gitignore/api/python,windows,visualstudiocode" -o .gitignore
git add .
git commit -m "feat: initial commit"
gh repo create my-project --public --source=. --remote=origin --push
```
