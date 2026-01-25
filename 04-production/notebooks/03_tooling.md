# Production 03: Python Tooling

Professional Python development requires a consistent tooling setup.

## 1. Virtual Environments
Isolate project dependencies.

```bash
# Create
python -m venv venv

# Activate (Windows)
venv\Scripts\activate

# Activate (macOS/Linux)
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

## 2. Dependency Management

### requirements.txt
```
requests==2.28.0
flask>=2.0,<3.0
```

### pyproject.toml (Modern)
```toml
[project]
dependencies = [
    "requests>=2.28",
    "flask>=2.0",
]
```

## 3. Linting & Formatting
| Tool | Purpose |
|:---|:---|
| `ruff` | Fast linter (replaces flake8, isort) |
| `black` | Code formatter |
| `mypy` | Static type checker |

## 4. Pre-Commit Hooks
Automate checks before every commit.

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    hooks:
      - id: ruff
      - id: ruff-format
```
