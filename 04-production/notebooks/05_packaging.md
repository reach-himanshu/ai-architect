# Production 05: Packaging & Distribution

Learn to package your code as a reusable library.

## 1. Project Structure

```
my_package/
├── pyproject.toml
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── core.py
├── tests/
│   └── test_core.py
└── README.md
```

## 2. pyproject.toml (Modern Standard)

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
version = "0.1.0"
authors = [{ name = "Your Name" }]
description = "A short description"
dependencies = ["requests>=2.28"]

[project.scripts]
my-cli = "my_package.cli:main"
```

## 3. Building & Installing

```bash
# Build
pip install build
python -m build

# Install locally (editable mode)
pip install -e .
```

## 4. Publishing to PyPI

```bash
pip install twine
twine upload dist/*
```
