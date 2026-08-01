## Complete Project Tree
```
my_project/                      ← 🔵 Project Root Directory
│
├── 📄 pyproject.toml             ← ⭐ THE central config file (replaces setup.py, requirements.txt, etc.)
├── 📄 poetry.lock                ← 🔒 Locked dependency versions (auto-generated)
├── 📄 README.md                  ← 📖 Project documentation
├── 📄 .gitignore                 ← 🚫 Files Git should ignore
├── 📄 .env                       ← 🔑 Environment variables (secrets, config)
│
├── 📁 .venv/                    ←  Virtual Environment (if configured in-project)
│   ├── bin/ (or Scripts/ on Windows)
│   │   ├── python              ← The isolated Python interpreter
│   │   ├── pip
│   │   ├── poetry
│   │   └── activate            ← Script to activate the venv
│   ├── lib/
│   │   └── python3.11/
│   │       └── site-packages/  ← All installed packages live here
│   │           ├── requests/
│   │           ├── flask/
│   │           └── ...
│   └── pyvenv.cfg
│
├── 📁 my_project/         ←  SOURCE CODE (your actual package)
│   ├── 📄 __init__.py     ← Makes this directory a Python package
│   ├── 📄 main.py         ← Entry point / main application logic
│   ├── 📄 config.py       ← Configuration management
│   │
│   ├── 📁 models/         ← Data models / database schemas
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── product.py
│   │
│   ├── 📁 services/              ← Business logic layer
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   └── payment_service.py
│   │
│   ├── 📁 api/                   ← API routes / controllers
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   └── middleware.py
│   │
│   └── 📁 utils/                 ← Helper / utility functions
│       ├── __init__.py
│       ├── helpers.py
│       └── validators.py
│
├── 📁 tests/                     ← All tests
│   ├── __init__.py
│   ├── conftest.py               ← Pytest fixtures & config
│   ├── 📁 unit/
│   │   ├── __init__.py
│   │   ├── test_user.py
│   │   └── test_auth_service.py
│   └── 📁 integration/
│       ├── __init__.py
│       └── test_api.py
│
├── 📁 docs/               ← Documentation files
│   ├── architecture.md
│   └── api_reference.md
│
└── 📁 scripts/        ← Utility scripts (DB migrations, etc.)
    ├── seed_db.py
    └── deploy.sh
```

---
## The `pyproject.toml` - The Brain of the Project
This single file replaces `setup.py`, `setup.cfg`, `requirements.txt`, `MANIFEST.in`, `tox.ini`, etc.
```toml
# ============================================================
#  📄 pyproject.toml — COMPLETE EXAMPLE
# ============================================================

# ----- 🏷️ PROJECT METADATA -----
[tool.poetry]
name = "my-project"                     # Package name on PyPI
version = "0.1.0"                       # Semantic versioning
description = "A short description"
authors = ["Your Name <you@email.com>"]
license = "MIT"
readme = "README.md"
packages = [{include = "my_project"}]   # ← Points to your source package

# ----- 📦 PRODUCTION DEPENDENCIES -----
[tool.poetry.dependencies]
python = "^3.11"                        # Python version constraint
requests = "^2.31.0"                    # ^2.31.0 means >=2.31.0, <3.0.0
flask = "^3.0"
sqlalchemy = "^2.0"
pydantic = "^2.5"

# ----- 🧪 DEV-ONLY DEPENDENCIES -----
[tool.poetry.group.dev.dependencies]
pytest = "^7.4"
black = "^23.0"                         # Code formatter
ruff = "^0.1"                           # Linter
mypy = "^1.7"                           # Type checker

# ----- 🔧 SCRIPTS / ENTRY POINTS -----
[tool.poetry.scripts]
start = "my_project.main:app"           # `poetry run start` runs main.py's app()

# ----- ⚙️ BUILD SYSTEM -----
[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

# ----- 🛠️ TOOL CONFIGURATIONS -----
[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["."]

[tool.black]
line-length = 88

[tool.ruff]
line-length = 88
select = ["E", "F", "I"]

[tool.mypy]
strict = true
```

---
## How Everything Connects - Visual Flow
```
┌─────────────────────────────────────────────────────────┐
│                    YOU (Developer)                      │
└─────────────┬───────────────────────────┬───────────────┘
              │                           │
              ▼                           ▼
┌──────────────────────┐    ┌──────────────────────────┐
│   Terminal Commands  │    │      IDE (VS Code)       │
│                      │    │                          │
│ poetry install       │    │  Selects .venv/bin/python│
│ poetry add requests  │    │  as interpreter          │
│ poetry run python    │    │                          │
│ poetry run pytest    │    └──────────┬───────────────┘
└──────────┬───────────┘               │
           │                           │
           ▼                           ▼
┌─────────────────────────────────────────────────────────┐
│                                                         │
│                 pyproject.toml                          │
│              (reads dependencies & config)              │
│                                                         │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│                                                         │
│                 poetry.lock                             │
│     (exact resolved versions of ALL dependencies        │
│      including sub-dependencies)                        │
│                                                         │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│                                                         │
│                 .venv/  (Virtual Environment)           │
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │  site-packages/                                    │ │
│  │  ├── requests/     ← installed from poetry.lock    │ │
│  │  ├── flask/        ← installed from poetry.lock    │ │
│  │  ├── sqlalchemy/   ← installed from poetry.lock    │ │
│  │  └── pytest/       ← dev dependency                │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
┌──────────────────┐   ┌──────────────────┐
│    my_project/   │   │    tests/        │
│                  │   │                  │
│ Your source code │   │ Your test code   │
│ IMPORTS packages │   │ IMPORTS from     │
│ from .venv       │   │ my_project/ &    │
│                  │   │ .venv            │
└──────────────────┘   └──────────────────┘
```

---
## Creating This From Scratch

### Step 1: Create the project
```bash
# Poetry creates the entire scaffold for you
poetry new my_project

# OR if you already have a folder:
cd existing_folder
poetry init          # Interactive setup
```

### Step 2: Configure `venv` to live IN the project
```bash
# This makes .venv appear inside your project folder (recommended)
poetry config virtualenvs.in-project true
```

### Step 3: Install dependencies
```bash
# Add production dependencies
poetry add requests flask sqlalchemy pydantic

# Add dev-only dependencies
poetry add --group dev pytest black ruff mypy

# Install everything from existing pyproject.toml
poetry install
```

### Step 4: Run things
```bash
# Run any command inside the virtual environment
poetry run python my_project/main.py
poetry run pytest
poetry run black .

# OR activate the shell
poetry shell
python my_project/main.py     # Now you're "inside" the venv
exit                           # Leave the venv shell
```

---
## Key Files Explained

### `__init__.py` - Package Marker

```python
# my_project/__init__.py

# Can be empty! Its mere existence makes the folder a "package"
# Or you can expose key items:

from my_project.main import app
from my_project.config import settings

__version__ = "0.1.0"
```

### `main.py` - Entry Point

```python
# my_project/main.py

from my_project.services.auth_service import authenticate
from my_project.models.user import User
from my_project.config import settings

def app():
    print(f"Starting {settings.APP_NAME}...")
    user = User(name="Alice")
    authenticate(user)

if __name__ == "__main__":
    app()
```

### `conftest.py` - Shared Test Fixtures

```python
# tests/conftest.py

import pytest
from my_project.models.user import User

@pytest.fixture
def sample_user():
    return User(name="TestUser", email="test@test.com")
```

---
## Where Does the `.venv` Actually Live?

```
OPTION A: Inside project (recommended)          OPTION B: Centralized (default)
poetry config virtualenvs.in-project true       poetry config virtualenvs.in-project false

my_project/                                     my_project/
├── .venv/  ✅ RIGHT HERE                       ├── (no .venv here)
├── pyproject.toml                              ├── pyproject.toml
└── ...                                         └── ...

                                                 ~/.cache/pypoetry/virtualenvs/
                                                 └── my-project-Ab3x7K-py3.11/  ← Hidden away
```

Check where yours is:
```bash
poetry env info --path
```

---
## Quick Reference: Import Relationships
```
my_project/
├── models/user.py         →  from my_project.models.user import User
├── services/auth.py       →  from my_project.services.auth import login
├── utils/helpers.py       →  from my_project.utils.helpers import format_date
└── config.py              →  from my_project.config import settings

tests/
└── test_user.py           →  from my_project.models.user import User  (same!)
```

---
## Common Poetry Commands Cheatsheet
```bash
poetry new project_name          # 🆕 Create new project
poetry init                      # 🆕 Initialize in existing folder
poetry install                   # 📥 Install all dependencies
poetry add package_name          # ➕ Add a dependency
poetry add --group dev pytest    # ➕ Add dev dependency
poetry remove package_name       # ➖ Remove a dependency
poetry update                    # 🔄 Update all packages
poetry show                      # 📋 List installed packages
poetry show --tree               # 🌳 Show dependency tree
poetry run python script.py      # ▶️  Run inside venv
poetry shell                     # 🐚 Activate venv shell
poetry build                     # 📦 Build distributable package
poetry publish                   # 🚀 Publish to PyPI
poetry env info                  # ℹ️  Show venv info
poetry lock                      # 🔒 Regenerate lock file
```
This structure scales from small scripts to large production applications. The key insight is that **`pyproject.toml` is the single source of truth**, and the **`.venv` isolates your project's dependencies** from every other Python project on your machine.

---