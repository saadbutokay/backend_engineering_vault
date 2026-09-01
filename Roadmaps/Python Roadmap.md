# THE ROADMAP

---
## PHASE 00 / ENGINEERING FUNDAMENTALS
**Follow the structure down.**
*Duration: 2-3 weeks at 6 hours/day*

This phase exists because you said you have zero prior knowledge. You do not touch Python here. You build the mental model of what a computer is, what software engineering is, and how the internet works. Skipping this phase is why most self-taught developers have gaps that haunt them for years.

---

### 00.01 — What Is Computing

```
- What a computer actually does (input, processing, storage, output)
- Binary and data representation (bits, bytes, encoding, ASCII, UTF-8)
- CPU, RAM, storage — how they interact
- What an operating system does
- Processes vs threads (conceptual)
- What a program is (source code → compilation/interpretation → execution)
- Interpreted vs compiled languages
- What Python is and where it sits in the landscape
```

### 00.02 — How the Internet Works

```
- Client-server model
- IP addresses, DNS, ports
- TCP/IP stack (conceptual)
- HTTP/HTTPS — request/response cycle
- What a URL is (scheme, host, path, query, fragment)
- What an API is (conceptual)
- JSON and data interchange formats
- Latency, bandwidth, throughput (definitions)
```

### 00.03 — Operating System Basics (macOS and Linux)

```
- File system hierarchy (/, /home, /etc, /usr, /var, /tmp)
- Users, groups, permissions (rwx, chmod, chown)
- Environment variables
- PATH and how the shell resolves commands
- Process management (ps, top, kill)
- Package managers (brew for macOS, apt for Linux)
- SSH basics
```

### 00.04 — What Is Software Engineering

```
- Software engineering vs programming
- Software Development Life Cycle (SDLC)
- Agile, Scrum, Kanban (overview)
- What version control is and why it matters
- What testing is and why it matters
- What deployment means
- Roles: frontend, backend, fullstack, DevOps, SRE, data engineer
- What a backend engineer specifically does day-to-day
- Reading documentation as a skill
- How to ask technical questions (problem → context → what you tried)
```

### 00.05 — Engineering Culture and Mindset

```
- How to read error messages
- How to search for solutions effectively
- How to read source code you did not write
- Technical debt (concept)
- Code review culture
- Writing things down (why Obsidian matters)
- The compounding nature of fundamentals
```

### PHASE 00 PROJECT

```
Project: "My Machine" Documentation
- Document your machine's specs (CPU, RAM, storage, OS version)
- Map your file system hierarchy
- Set up Homebrew (macOS) or apt (Linux)
- Install and configure: terminal emulator, Obsidian
- Write your first 5 Obsidian notes using the templates above
- Create a personal glossary note with every new term from Phase 00
- Deliverable: A fully structured Obsidian vault ready for Phase 01
```

---

## PHASE 01 / PYTHON CORE
**Follow the structure down.**
*Duration: 4-5 weeks at 6 hours/day*

This is the longest foundational phase. You will not rush this. Every concept here is used daily by principal engineers. Mastery of fundamentals is what separates senior from mid-level.

---

### 01.01 — Environment Setup

```
- Installing Python (pyenv for version management)
- Python REPL
- Running .py files from terminal
- Virtual environments (venv, why they exist)
- pip and requirements.txt
- Project folder structure conventions
- PEP 8 style guide (overview)
```

### 01.02 — Syntax and Basics

```
- Variables and assignment
- Naming conventions (snake_case)
- Data types: int, float, str, bool, None
- Type checking (type(), isinstance())
- Type conversion (casting)
- Operators: arithmetic, comparison, logical, assignment, identity, membership
- String operations: concatenation, f-strings, slicing, methods
- Input/output (print, input)
- Comments and docstrings
- Indentation as syntax
```

### 01.03 — Control Flow

```
- if / elif / else
- Truthy and falsy values
- Ternary expressions
- match/case (Python 3.10+)
- while loops
- for loops
- range()
- break, continue, pass
- Nested loops
- Loop-else clause
- Comprehensions: list, dict, set, generator expressions
```

### 01.04 — Data Structures (Built-in)

```
- Lists: creation, indexing, slicing, methods, mutability
- Tuples: immutability, packing/unpacking, named tuples
- Dictionaries: creation, access, methods, iteration, nesting
- Sets: creation, operations (union, intersection, difference), frozensets
- Strings as sequences
- When to use which data structure
- Copying: shallow vs deep (copy module)
- Collections module: defaultdict, Counter, OrderedDict, deque, ChainMap
```

### 01.05 — Functions

```
- Defining functions (def)
- Parameters vs arguments
- Positional, keyword, default arguments
- *args and **kwargs
- Return values (single, multiple via tuple)
- Scope: local, enclosing, global, built-in (LEGB rule)
- global and nonlocal keywords
- First-class functions (functions as objects)
- Higher-order functions
- Lambda functions
- Closures
- Recursion (with base cases)
- Function annotations / type hints
- Docstrings (Google style, NumPy style)
```

### 01.06 — Object-Oriented Programming

```
- What OOP is and why it exists
- Classes and objects
- __init__ and self
- Instance attributes vs class attributes
- Instance methods, class methods (@classmethod), static methods (@staticmethod)
- Properties (@property, getter/setter)
- Encapsulation (name mangling, conventions)
- Inheritance (single, multiple, MRO — Method Resolution Order)
- Polymorphism
- Abstract base classes (abc module)
- Magic/dunder methods:
    __str__, __repr__, __len__, __getitem__, __setitem__,
    __eq__, __lt__, __hash__, __call__, __enter__, __exit__,
    __add__, __iter__, __next__, __contains__
- Composition vs inheritance (when to use each)
- Dataclasses (@dataclass)
- Slots (__slots__)
- Protocols (structural subtyping, Python 3.8+)
```

### 01.07 — Error Handling

```
- What exceptions are
- try / except / else / finally
- Built-in exception hierarchy
- Catching specific exceptions
- Raising exceptions (raise)
- Custom exception classes
- Exception chaining (from)
- Context managers (with statement)
- Writing custom context managers (__enter__, __exit__)
- contextlib module (contextmanager decorator)
- When to catch vs when to let exceptions propagate
- Logging errors vs printing errors
```

### 01.08 — File I/O and Serialization

```
- Reading/writing text files (open, read, write, modes)
- File paths (pathlib.Path)
- Working with CSV (csv module)
- Working with JSON (json module — loads, dumps, load, dump)
- Working with YAML (pyyaml)
- Working with environment files (.env, python-dotenv)
- Binary files (conceptual)
- Pickle (what it is, why to avoid it in production)
- StringIO and BytesIO
```

### 01.09 — Modules and Packages

```
- What a module is
- import, from...import, as
- __name__ == "__main__"
- What a package is (__init__.py)
- Relative vs absolute imports
- Creating your own packages
- The Python standard library (overview of key modules):
    os, sys, pathlib, datetime, math, random, re, itertools,
    functools, collections, typing, logging, argparse, unittest,
    json, csv, hashlib, secrets, decimal, uuid
- Third-party packages (PyPI)
- pip install, pip freeze
- pyproject.toml and modern packaging
```

### 01.10 — Functional Programming Concepts

```
- map(), filter(), reduce()
- Lambda functions (revisited in functional context)
- Iterators and iterables (__iter__, __next__)
- Generators (yield, generator expressions)
- Generator pipelines
- itertools: chain, islice, product, permutations, combinations, groupby, starmap
- functools: partial, lru_cache, reduce, wraps, total_ordering
- Decorators:
    - What they are
    - Writing decorators
    - Decorators with arguments
    - Stacking decorators
    - Class-based decorators
    - functools.wraps
- Immutability patterns in Python
```

### 01.11 — Type Hints and Static Analysis

```
- Why type hints matter in production code
- Basic type hints (int, str, float, bool, None)
- typing module: List, Dict, Tuple, Set, Optional, Union
- Python 3.10+ syntax (int | None, list[str])
- Type aliases
- TypeVar and Generic
- Callable
- Literal
- TypedDict
- Protocol
- mypy: installation, configuration, running checks
- When to use type hints (public APIs, always; scripts, optional)
```

### 01.12 — Testing Fundamentals

```
- Why testing matters
- Types of tests: unit, integration, end-to-end
- unittest module (basics)
- pytest:
    - Installation and configuration
    - Writing test functions
    - Assertions
    - Fixtures
    - Parametrize
    - Markers
    - Conftest
    - Coverage (pytest-cov)
- Test-Driven Development (TDD) — concept and workflow
- Mocking (unittest.mock — patch, MagicMock)
- What to test, what not to test
- Testing conventions and file structure
```

### 01.13 — Logging

```
- Why logging over print
- logging module
- Log levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
- Configuring loggers, handlers, formatters
- Logging to files
- Structured logging (conceptual)
- Logging best practices
```

### 01.14 — Regular Expressions

```
- What regex is
- re module: match, search, findall, finditer, sub, split
- Pattern syntax: literals, metacharacters, character classes,
  quantifiers, anchors, groups, alternation, lookahead, lookbehind
- Compiling patterns (re.compile)
- Named groups
- Common patterns (email, phone, URL, dates)
- When regex is the wrong tool
```

### PHASE 01 CHAPTER PROJECTS

```
Project 01A: "Python Fundamentals Drill Suite"
- 50 small exercises covering every syntax topic
- Organized by sub-topic in a single repository
- Each exercise has a test file using pytest
- Run all tests with a single command

Project 01B: "Personal Finance Calculator"
- CLI application
- Read transactions from CSV
- Categorize expenses (food, rent, transport, etc.)
- Calculate: total income, total expenses, savings rate, category breakdown
- Output results to JSON and formatted terminal output
- Use OOP (Transaction, Category, Report classes)
- Full pytest test suite
- Type hints throughout
- Logging for errors and info
- This is the seed for Portfolio Project 01
```

---

## PHASE 02 / DEVELOPER TOOLING
**Follow the structure down.**
*Duration: 2-3 weeks at 6 hours/day*

You cannot work on a team or contribute to open source without these. This phase runs partially in parallel with Phase 01 in practice, but is separated here for clarity.

---

### 02.01 — Terminal Mastery (Bash/Zsh)

```
- Shell vs terminal vs console
- Bash vs Zsh (macOS default is Zsh)
- Navigation: cd, ls, pwd, mkdir, rm, cp, mv, find, locate
- Viewing files: cat, less, head, tail, wc
- Redirection: >, >>, <, 2>, &>, |
- Piping and chaining commands
- grep, sed, awk (basics)
- xargs
- Aliases and shell configuration (.zshrc, .bashrc)
- Shell scripting basics:
    - Variables
    - Conditionals
    - Loops
    - Functions
    - Exit codes
    - Shebang line
- curl and wget
- chmod, chown
- cron jobs (crontab)
- tmux or screen (terminal multiplexing)
- Dotfiles management
```

### 02.02 — Git and GitHub

```
- What version control is
- Git architecture: working directory, staging area, repository
- git init, clone, status, add, commit, log, diff
- .gitignore (patterns, global gitignore)
- Branching: branch, checkout, switch, merge
- Merge conflicts: what they are, how to resolve them
- Rebasing: rebase, interactive rebase
- Cherry-pick
- Stashing: stash, stash pop, stash list
- Tags
- Remote repositories: remote, push, pull, fetch
- GitHub:
    - Repository creation
    - README.md (writing good READMEs)
    - Issues and issue templates
    - Pull requests and PR templates
    - Branch protection rules
    - GitHub Actions (introduction)
    - GitHub Pages (for portfolio)
    - SSH keys setup
- Git workflows:
    - Feature branch workflow
    - Gitflow
    - Trunk-based development
- Conventional commits
- Commit message conventions
- Git log visualization
- Reverting: revert, reset (soft, mixed, hard)
- Bisect
- Blame
```

### 02.03 — Virtual Environments and Package Management

```
- venv (revisited in depth)
- pyenv: installing multiple Python versions, setting local/global versions
- pyenv-virtualenv
- pip: install, uninstall, freeze, list
- requirements.txt vs requirements-dev.txt
- pip-tools (pip-compile, pip-sync)
- Poetry:
    - Installation
    - pyproject.toml
    - Dependency groups
    - Lock files
    - Publishing packages
- Comparison: pip vs poetry vs pipenv
- Dependency resolution
- Semantic versioning (semver)
- Security: pip audit, safety
```

### 02.04 — Code Quality Tools

```
- Linting: ruff, flake8, pylint
- Formatting: black, isort
- Type checking: mypy (revisited)
- Pre-commit hooks:
    - pre-commit framework
    - .pre-commit-config.yaml
    - Common hooks (black, ruff, mypy, trailing whitespace)
- Makefile for project commands
- EditorConfig
- Configuration: pyproject.toml as single config file
```

### 02.05 — IDE and Editor

```
- VS Code:
    - Essential extensions for Python
    - Settings.json configuration
    - Debugging (breakpoints, launch.json)
    - Integrated terminal
    - Workspace settings
- PyCharm (overview, when to use it)
- Vim basics (for server editing):
    - Modes (normal, insert, visual, command)
    - Basic navigation and editing
    - Saving and quitting
    - Search and replace
```

### PHASE 02 CHAPTER PROJECT

```
Project 02: "Dotfiles and Dev Environment"
- Create a dotfiles repository on GitHub
- Shell configuration files (.zshrc)
- Git configuration (.gitconfig)
- VS Code settings
- A setup.sh script that automates environment setup on a new machine
- Document everything in Obsidian
- Use conventional commits
- Set up pre-commit hooks
```

---

## PHASE 03 / DATA STRUCTURES AND ALGORITHMS
**Follow the structure down.**
*Duration: 4-5 weeks at 6 hours/day*

This phase is for interviews and for understanding why certain backend decisions are made (why use a hash map here, why a B-tree index there). You will implement everything in Python.

---

### 03.01 — Complexity Analysis

```
- What Big-O notation is
- Time complexity: O(1), O(log n), O(n), O(n log n), O(n²), O(2ⁿ), O(n!)
- Space complexity
- Best case, worst case, average case
- Amortized analysis (conceptual)
- How to analyze loops, nested loops, recursive calls
- Common complexity of Python operations (list, dict, set)
```

### 03.02 — Arrays and Strings

```
- Arrays vs Python lists (dynamic arrays)
- Two-pointer technique
- Sliding window technique
- Prefix sums
- String manipulation algorithms
- Anagrams, palindromes, substrings
- Practice problems (10 minimum)
```

### 03.03 — Linked Lists

```
- Singly linked list (implementation from scratch)
- Doubly linked list (implementation from scratch)
- Operations: insert, delete, search, reverse
- Cycle detection (Floyd's algorithm)
- Merge two sorted lists
- LRU Cache concept (linked list + hash map)
- Practice problems (8 minimum)
```

### 03.04 — Stacks and Queues

```
- Stack (implementation with list and linked list)
- Queue (implementation with deque)
- Applications: balanced parentheses, expression evaluation
- Monotonic stack
- Priority queue / heap (heapq module)
- Min-heap, max-heap
- Practice problems (8 minimum)
```

### 03.05 — Hash Tables

```
- How hashing works
- Collision resolution: chaining, open addressing
- Implementing a hash table from scratch
- Python dict internals (CPython implementation overview)
- Hash sets
- Frequency counting patterns
- Two-sum and its variants
- Practice problems (8 minimum)
```

### 03.06 — Trees

```
- Tree terminology (root, node, leaf, height, depth, level)
- Binary trees (implementation)
- Binary search trees (BST):
    - Insert, search, delete
    - In-order, pre-order, post-order traversal
    - Level-order traversal (BFS)
- Balanced BSTs (concept: AVL, Red-Black)
- Tries (prefix trees):
    - Implementation
    - Autocomplete use case
- Heaps (revisited as trees)
- Practice problems (10 minimum)
```

### 03.07 — Graphs

```
- Graph terminology (vertex, edge, directed, undirected, weighted, cycle)
- Representations: adjacency matrix, adjacency list
- BFS (breadth-first search)
- DFS (depth-first search)
- Topological sort
- Dijkstra's shortest path
- Cycle detection
- Connected components
- Union-Find (disjoint set)
- Practice problems (10 minimum)
```

### 03.08 — Sorting and Searching

```
- Bubble sort, selection sort, insertion sort (understand, not production use)
- Merge sort (implementation, analysis)
- Quick sort (implementation, analysis, pivot selection)
- Counting sort, radix sort (when to use)
- Python's Timsort (what it is)
- Binary search (iterative and recursive)
- Binary search variations (lower bound, upper bound, rotated array)
- Practice problems (8 minimum)
```

### 03.09 — Dynamic Programming

```
- What dynamic programming is
- Overlapping subproblems
- Optimal substructure
- Memoization (top-down)
- Tabulation (bottom-up)
- Classic problems:
    - Fibonacci
    - Climbing stairs
    - Coin change
    - Longest common subsequence
    - Knapsack (0/1)
    - Edit distance
    - Maximum subarray (Kadane's algorithm)
- How to identify DP problems
- Practice problems (10 minimum)
```

### 03.10 — Greedy Algorithms and Backtracking

```
- Greedy strategy and when it works
- Activity selection
- Interval scheduling
- Huffman coding (conceptual)
- Backtracking:
    - N-Queens
    - Sudoku solver
    - Permutations and combinations
- Practice problems (6 minimum)
```

### PHASE 03 CHAPTER PROJECT

```
Project 03: "Algorithm Visualizer CLI"
- CLI tool that visualizes sorting algorithms step-by-step in terminal
- Supports: bubble, selection, insertion, merge, quick sort
- Shows comparisons and swaps count
- Benchmarks each algorithm against random, sorted, reverse-sorted arrays
- Includes a BST mode: visualize insertions, traversals
- Full pytest suite
- Published to GitHub with clear README
```

---

## PHASE 04 / DATABASES
**Follow the structure down.**
*Duration: 4-5 weeks at 6 hours/day*

Backend engineering is 50% data management. This phase is heavy and critical. Fintech is entirely data-dependent.

---

### 04.01 — Database Fundamentals

```
- What a database is
- DBMS vs database
- Relational vs non-relational (conceptual comparison)
- ACID properties (Atomicity, Consistency, Isolation, Durability)
- CAP theorem (conceptual)
- OLTP vs OLAP
- Data modeling concepts:
    - Entities, attributes, relationships
    - ER diagrams
    - Cardinality (one-to-one, one-to-many, many-to-many)
- Normalization (1NF, 2NF, 3NF, BCNF)
- Denormalization (when and why)
- Schemas and schema design
```

### 04.02 — SQL Fundamentals

```
- What SQL is
- DDL: CREATE, ALTER, DROP, TRUNCATE
- Data types (integer, varchar, text, boolean, date, timestamp, decimal, numeric, UUID, JSONB)
- Constraints: PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL, CHECK, DEFAULT
- DML: INSERT, UPDATE, DELETE, UPSERT (ON CONFLICT)
- DQL: SELECT
    - WHERE, AND, OR, NOT, IN, BETWEEN, LIKE, ILIKE
    - ORDER BY, LIMIT, OFFSET
    - DISTINCT
    - Aliases
- Aggregate functions: COUNT, SUM, AVG, MIN, MAX
- GROUP BY, HAVING
- JOINs:
    - INNER JOIN
    - LEFT JOIN
    - RIGHT JOIN
    - FULL OUTER JOIN
    - CROSS JOIN
    - Self join
- Subqueries (scalar, column, table, correlated)
- Common Table Expressions (CTEs — WITH clause)
- Window functions:
    - ROW_NUMBER, RANK, DENSE_RANK
    - LAG, LEAD
    - SUM/AVG/COUNT OVER (PARTITION BY ... ORDER BY ...)
    - NTILE
    - Running totals, moving averages
- UNION, INTERSECT, EXCEPT
- CASE expressions
- COALESCE, NULLIF
- EXISTS vs IN
- Views and materialized views
- Transactions: BEGIN, COMMIT, ROLLBACK, SAVEPOINT
- Isolation levels (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE)
```

### 04.03 — PostgreSQL Deep Dive

```
- Why PostgreSQL (industry standard, fintech standard)
- Installation (brew install postgresql)
- psql CLI
- pgAdmin (GUI overview)
- PostgreSQL-specific features:
    - JSONB (storing, querying, indexing JSON)
    - Arrays
    - ENUM types
    - UUID generation (gen_random_uuid())
    - Full-text search (tsvector, tsquery)
    - LISTEN/NOTIFY
    - Table partitioning (range, list, hash)
    - Schemas (PostgreSQL schemas, not just public)
- Indexing:
    - What an index is (B-tree default)
    - Types: B-tree, Hash, GIN, GiST, BRIN
    - Composite indexes
    - Partial indexes
    - Expression indexes
    - When to index, when not to
    - EXPLAIN and EXPLAIN ANALYZE
    - Reading query plans
- Performance:
    - VACUUM, ANALYZE, AUTOVACUUM
    - Connection pooling (PgBouncer concept)
    - pg_stat_statements
    - Slow query identification
- Backup and restore: pg_dump, pg_restore
- Roles and permissions (GRANT, REVOKE)
```

### 04.04 — Python and PostgreSQL

```
- psycopg2 / psycopg3:
    - Connection, cursor
    - Parameterized queries (preventing SQL injection)
    - Transactions
    - Connection pooling
    - Async support (psycopg3)
- Database connection patterns in applications
```

### 04.05 — SQLAlchemy (ORM)

```
- What an ORM is
- SQLAlchemy Core vs ORM
- SQLAlchemy 2.0 style (current):
    - Engine and connection
    - Declarative mapping
    - Mapped classes
    - Column types
    - Relationships (one-to-many, many-to-many, one-to-one)
    - Backref and back_populates
    - Session and unit of work pattern
    - Querying (select, where, join, order_by, group_by)
    - Eager loading vs lazy loading (selectinload, joinedload)
    - Bulk operations
    - Events and hooks
- When to use raw SQL vs ORM
```

### 04.06 — Database Migrations

```
- What migrations are and why they matter
- Alembic:
    - Installation and configuration
    - Generating migrations (autogenerate)
    - Writing manual migrations
    - Upgrading and downgrading
    - Migration history
    - Handling data migrations
    - Migrations in team workflows
```

### 04.07 — Redis

```
- What Redis is (in-memory data structure store)
- Installation and redis-cli
- Data types: strings, lists, sets, sorted sets, hashes, streams
- Key expiration (TTL)
- Common use cases:
    - Caching
    - Session storage
    - Rate limiting
    - Leaderboards
    - Pub/Sub
    - Job queues
- Python + Redis (redis-py):
    - Connection
    - Basic operations
    - Pipelining
    - Transactions
- Redis persistence: RDB, AOF
- Redis eviction policies
```

### 04.08 — MongoDB (Overview)

```
- What a document database is
- When to use MongoDB vs PostgreSQL
- Documents, collections, databases
- BSON
- CRUD operations
- Indexing in MongoDB
- Aggregation pipeline
- Python + MongoDB (pymongo):
    - Connection
    - CRUD
    - Queries
- When MongoDB is the wrong choice (most fintech — know why)
```

### PHASE 04 CHAPTER PROJECT

```
Project 04: "Financial Data Store"
- Design a normalized schema for:
    - Users, accounts, transactions, categories, budgets
- Implement in PostgreSQL
- Write raw SQL for all CRUD operations
- Write complex queries:
    - Monthly spending by category
    - Running account balance over time
    - Top merchants by transaction volume
    - Year-over-year comparison
- Implement the same with SQLAlchemy ORM
- Add Alembic migrations
- Add Redis caching for frequently accessed queries
- Benchmark: raw SQL vs ORM query performance
- Full test suite
```

---

## PHASE 05 / BACKEND FRAMEWORKS
**Follow the structure down.**
*Duration: 5-6 weeks at 6 hours/day*

This is where you become a backend developer. The primary framework is FastAPI (industry momentum, async-first, fintech-friendly). Django and Flask are covered because you will encounter them.

---

### 05.01 — HTTP Deep Dive

```
- HTTP/1.1, HTTP/2, HTTP/3 (overview)
- Request structure: method, URL, headers, body
- Response structure: status code, headers, body
- Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD
- Status codes:
    - 1xx Informational
    - 2xx Success (200, 201, 204)
    - 3xx Redirection (301, 302, 304)
    - 4xx Client Error (400, 401, 403, 404, 409, 422, 429)
    - 5xx Server Error (500, 502, 503, 504)
- Headers: Content-Type, Authorization, Accept, Cache-Control, CORS headers
- Content types: application/json, multipart/form-data, application/x-www-form-urlencoded
- Cookies and sessions
- HTTPS and TLS (conceptual)
- REST constraints:
    - Client-server
    - Stateless
    - Cacheable
    - Uniform interface
    - Layered system
- REST API design:
    - Resource naming conventions
    - URL structure
    - HTTP methods mapping to CRUD
    - Pagination (offset, cursor)
    - Filtering and sorting
    - Versioning (URL vs header)
    - HATEOAS (concept)
- Request/response lifecycle in a web framework
```

### 05.02 — FastAPI Fundamentals

```
- What FastAPI is and why it dominates modern Python backend
- Installation and project structure
- ASGI vs WSGI
- Uvicorn
- Path operations (routes):
    - Path parameters
    - Query parameters
    - Request body (Pydantic models)
- Pydantic v2:
    - BaseModel
    - Field validation
    - Custom validators
    - Nested models
    - Config (model_config)
    - Serialization (model_dump, model_dump_json)
    - Computed fields
- Response models
- Status codes
- Automatic documentation (Swagger UI, ReDoc)
- Type hints driving the API (how FastAPI uses them)
```

### 05.03 — FastAPI Intermediate

```
- Dependency injection system
- Path operation dependencies
- Class-based dependencies
- Sub-dependencies
- Database session dependencies
- Request and Response objects
- Form data and file uploads
- Header and cookie parameters
- Error handling:
    - HTTPException
    - Custom exception handlers
    - Validation error handling
- Middleware:
    - What middleware is
    - CORS middleware
    - Custom middleware
    - Timing middleware
- Background tasks
- APIRouter (organizing large applications)
- Application events (lifespan)
```

### 05.04 — Authentication and Authorization

```
- Authentication vs authorization
- Password hashing (bcrypt, passlib)
- JWT (JSON Web Tokens):
    - Structure (header, payload, signature)
    - Access tokens and refresh tokens
    - Token expiration
    - python-jose or PyJWT
- OAuth2 flow in FastAPI:
    - OAuth2PasswordBearer
    - OAuth2PasswordRequestForm
    - Token endpoint
    - Protected routes
- Role-Based Access Control (RBAC)
- API keys
- Session-based auth vs token-based auth (comparison)
- Security headers
- CORS in depth
- Rate limiting (slowapi)
- Input validation as security (Pydantic)
- SQL injection prevention (parameterized queries)
- OWASP Top 10 (overview)
```

### 05.05 — FastAPI Advanced

```
- Async/await in Python:
    - What async programming is
    - Event loop
    - Coroutines
    - asyncio module
    - async def vs def in FastAPI
    - When to use async (I/O bound tasks)
    - asyncio.gather, asyncio.create_task
    - Async context managers
    - Async generators
- Async database access (asyncpg, async SQLAlchemy)
- Async Redis (aioredis / redis.asyncio)
- Async HTTP client (httpx)
- WebSockets:
    - What WebSockets are
    - WebSocket endpoint in FastAPI
    - Connection management
    - Broadcasting
    - Use cases: real-time data feeds, notifications
- Server-Sent Events (SSE)
- Streaming responses
- Testing FastAPI:
    - TestClient (httpx)
    - Async test client
    - Overriding dependencies for testing
    - Test database setup
    - Factory patterns for test data
    - pytest fixtures for FastAPI
```

### 05.06 — FastAPI Production Patterns

```
- Project structure for large applications:
    /app
        /api
            /v1
                /endpoints
                /dependencies
        /core (config, security)
        /db (session, base)
        /models (SQLAlchemy)
        /schemas (Pydantic)
        /services (business logic)
        /repositories (data access)
        /utils
        /tests
        main.py
- Configuration management (pydantic-settings, .env)
- Repository pattern
- Service layer pattern
- Separation of concerns
- Pagination utilities
- Health check endpoint
- Structured logging (structlog)
- Request ID tracing
- Error response standardization
- OpenAPI customization
```

### 05.07 — Django (Comprehensive Overview)

```
- What Django is and when to choose it over FastAPI
- Django philosophy (batteries-included, ORM, admin)
- Installation and project structure
- Django apps
- Models and migrations (Django ORM)
- Admin panel
- URL routing
- Views (function-based and class-based)
- Templates (overview only — backend focus)
- Django REST Framework (DRF):
    - Serializers
    - ViewSets
    - Routers
    - Authentication (token, JWT via djangorestframework-simplejwt)
    - Permissions
    - Pagination
    - Filtering (django-filter)
    - Throttling
- Django ORM queries (comparison with SQLAlchemy)
- Signals
- Management commands
- Celery integration (introduction)
- When Django wins: admin-heavy apps, rapid prototyping, monoliths
```

### 05.08 — Flask (Focused Overview)

```
- What Flask is (micro-framework)
- When Flask is used (small services, microservices, ML model serving)
- Installation and minimal app
- Routing and views
- Request/response objects
- Blueprints
- Flask-SQLAlchemy
- Flask-Migrate
- Flask-RESTful or Flask-RESTX
- Testing Flask apps
- Why Flask is losing ground to FastAPI
```

### 05.09 — GraphQL (Introduction)

```
- What GraphQL is
- REST vs GraphQL (tradeoffs)
- Queries, mutations, subscriptions
- Schema definition
- Strawberry (Python GraphQL library for FastAPI)
- When to use GraphQL (client-driven APIs, complex data requirements)
- When NOT to use GraphQL
```

### PHASE 05 CHAPTER PROJECT

```
Project 05: "Banking REST API"
- Full REST API using FastAPI
- Features:
    - User registration and login (JWT)
    - Account management (create, view, close)
    - Transaction processing (deposit, withdraw, transfer)
    - Transaction history with filtering, sorting, pagination
    - Budget creation and tracking
    - Spending analytics endpoints
    - Real-time balance updates via WebSocket
- PostgreSQL with SQLAlchemy (async)
- Alembic migrations
- Redis caching for balance lookups
- Rate limiting on sensitive endpoints
- RBAC (admin vs user)
- Comprehensive input validation
- Structured logging
- Full test suite (unit + integration)
- API documentation via Swagger
- This is Portfolio Project 02
```

---

## PHASE 06 / SYSTEM DESIGN
**Follow the structure down.**
*Duration: 4-5 weeks at 6 hours/day*

This separates junior from senior. You learn to think about systems, not just code.

---

### 06.01 — System Design Fundamentals

```
- What system design is
- Functional vs non-functional requirements
- Back-of-the-envelope estimation:
    - Daily active users
    - Requests per second
    - Storage calculations
    - Bandwidth calculations
- Latency numbers every engineer should know
- Horizontal vs vertical scaling
- Stateless vs stateful services
- Single points of failure
- Redundancy and replication
- Consistency vs availability (revisiting CAP)
- SLAs, SLOs, SLIs
```

### 06.02 — Caching Strategies

```
- Why caching matters
- Cache-aside (lazy loading)
- Write-through
- Write-behind (write-back)
- Read-through
- Cache invalidation strategies (TTL, event-based, version-based)
- Cache stampede and thundering herd
- Caching layers:
    - Application-level (in-memory, functools.lru_cache)
    - Distributed cache (Redis)
    - CDN caching
    - Database query cache
- Cache eviction policies (LRU, LFU, FIFO)
```

### 06.03 — Message Queues and Async Processing

```
- What a message queue is
- Producer-consumer pattern
- Point-to-point vs pub/sub
- RabbitMQ:
    - Concepts: exchange, queue, binding, routing key
    - Python client (pika)
    - Direct, topic, fanout exchanges
    - Acknowledgments and durability
- Celery:
    - Task queues
    - Workers
    - Brokers (Redis, RabbitMQ)
    - Results backend
    - Periodic tasks (Celery Beat)
    - Error handling and retries
    - Task chaining and grouping
- Apache Kafka (introduction):
    - Topics, partitions, consumer groups
    - When Kafka vs RabbitMQ
    - Confluent Kafka Python client
    - Event streaming vs message queuing
- Dead letter queues
- Idempotency in message processing
```

### 06.04 — Load Balancing and Reverse Proxies

```
- What a load balancer does
- Layer 4 vs Layer 7 load balancing
- Algorithms: round-robin, least connections, IP hash, weighted
- Health checks
- NGINX:
    - Installation and configuration
    - Reverse proxy setup
    - Load balancing configuration
    - SSL termination
    - Static file serving
    - Rate limiting
- HAProxy (conceptual comparison)
- Sticky sessions and their problems
```

### 06.05 — API Gateway Pattern

```
- What an API gateway is
- Authentication at the gateway
- Rate limiting at the gateway
- Request routing
- Request/response transformation
- Kong, AWS API Gateway (conceptual)
- Building a simple gateway with NGINX
```

### 06.06 — Microservices Architecture

```
- Monolith vs microservices
- When to use microservices (and when not to)
- Service boundaries (domain-driven design basics)
- Bounded contexts
- Inter-service communication:
    - Synchronous (REST, gRPC)
    - Asynchronous (message queues, events)
- gRPC:
    - What it is (protobuf, HTTP/2)
    - Python gRPC (grpcio)
    - Defining services (.proto files)
    - When to use gRPC vs REST
- Service discovery
- Circuit breaker pattern
- Saga pattern (managing distributed transactions)
- Strangler fig pattern (migrating monolith to microservices)
- API composition
- Shared database vs database-per-service
- Data consistency across services
- Distributed tracing (OpenTelemetry, Jaeger)
```

### 06.07 — Event-Driven Architecture

```
- What event-driven architecture is
- Events vs commands vs queries
- Event sourcing:
    - Event store
    - Replaying events
    - Snapshots
    - Benefits and drawbacks
- CQRS (Command Query Responsibility Segregation):
    - Read models vs write models
    - Why CQRS with event sourcing
- Domain events
- Integration events
- Event schemas and versioning
- Eventual consistency
```

### 06.08 — System Design Case Studies

```
- Design a URL shortener
- Design a rate limiter
- Design a notification system
- Design a payment processing system
- Design a real-time stock price feed
- Design a transaction ledger system
- Each case study:
    - Requirements gathering
    - High-level architecture
    - Component deep dive
    - Data model
    - API design
    - Scaling considerations
    - Tradeoffs
```

### PHASE 06 CHAPTER PROJECT

```
Project 06: "Real-Time Data Pipeline"
- Ingest financial market data (mock or free API)
- Kafka producer: publishes price updates
- Kafka consumer: processes and stores in PostgreSQL
- Redis: caches latest prices
- FastAPI: serves REST endpoints for historical data
- FastAPI WebSocket: streams real-time prices to clients
- Celery: periodic aggregation jobs (1-min, 5-min, 1-hour candles)
- Basic alerting: price threshold notifications
- Docker Compose for all services
- Full documentation
- This is Portfolio Project 03
```

---

## PHASE 07 / DEVOPS AND CLOUD
**Follow the structure down.**
*Duration: 5-6 weeks at 6 hours/day*

You cannot call yourself a backend engineer if you cannot deploy and operate your services.

---

### 07.01 — Linux Administration

```
- Linux distributions (Ubuntu, Debian, Alpine — when to use each)
- systemd: services, units, journalctl
- User management in depth
- File permissions in depth (sticky bit, setuid, setgid)
- Disk management: df, du, mount
- Networking tools: ip, ss, netstat, nslookup, dig, traceroute, curl
- Firewall: ufw, iptables (basics)
- Log files (/var/log, journalctl)
- Shell scripting for automation (revisited for ops)
- Monitoring: htop, iostat, vmstat
```

### 07.02 — Docker

```
- What containers are (vs VMs)
- Docker architecture: daemon, client, registry
- Images and containers
- Dockerfile:
    - FROM, RUN, COPY, ADD, WORKDIR, EXPOSE, CMD, ENTRYPOINT
    - Multi-stage builds
    - .dockerignore
    - Layer caching optimization
    - Security best practices (non-root user, minimal base images)
- Docker CLI:
    - build, run, exec, logs, ps, stop, rm, images, rmi
    - Volume mounts (-v)
    - Port mapping (-p)
    - Environment variables (-e, --env-file)
    - Networks
- Docker Compose:
    - docker-compose.yml structure
    - Services, networks, volumes
    - Environment files
    - Health checks
    - Depends_on
    - Profiles
    - Compose for development vs production
- Docker Hub (pushing images)
- Image tagging strategy
- Container logging
- Debugging containers
```

### 07.03 — CI/CD

```
- What CI/CD is
- Continuous Integration:
    - Automated testing on push
    - Linting and formatting checks
    - Type checking
    - Security scanning
- Continuous Delivery vs Continuous Deployment
- GitHub Actions:
    - Workflow files (.github/workflows/)
    - Triggers (push, pull_request, schedule, manual)
    - Jobs and steps
    - Actions marketplace
    - Secrets management
    - Matrix builds (multiple Python versions)
    - Caching dependencies
    - Building and pushing Docker images
    - Deploy workflows
- Pipeline stages: lint → test → build → deploy
- Branch-based deployments
- Rollback strategies
```

### 07.04 — AWS (Core Services)

```
- AWS account setup and IAM:
    - Root account vs IAM users
    - Policies and roles
    - MFA
    - Principle of least privilege
- Compute:
    - EC2: instances, AMIs, security groups, key pairs, elastic IPs
    - ECS: task definitions, services, Fargate vs EC2 launch type
    - Lambda: serverless functions, triggers, cold starts, Python runtime
- Networking:
    - VPC: subnets (public, private), route tables, internet gateway, NAT gateway
    - Security groups vs NACLs
    - Elastic Load Balancer (ALB, NLB)
- Storage:
    - S3: buckets, objects, versioning, lifecycle policies, presigned URLs
    - EBS: volume types
- Database:
    - RDS: PostgreSQL on RDS, parameter groups, backups, read replicas
    - ElastiCache: Redis on AWS
    - DynamoDB (overview for NoSQL use cases)
- Messaging:
    - SQS: standard vs FIFO queues
    - SNS: topics and subscriptions
    - EventBridge (overview)
- Monitoring:
    - CloudWatch: metrics, logs, alarms, dashboards
    - X-Ray (distributed tracing)
- Other:
    - Route 53 (DNS)
    - ACM (SSL certificates)
    - Secrets Manager / Parameter Store
    - ECR (container registry)
- AWS CLI and boto3 (Python SDK)
- Cost management and billing alerts
```

### 07.05 — Infrastructure as Code

```
- What IaC is and why it matters
- Terraform:
    - HCL syntax
    - Providers (AWS)
    - Resources
    - Variables and outputs
    - State (local, remote with S3 + DynamoDB locking)
    - Modules
    - Workspaces
    - Plan, apply, destroy
    - Terraform for:
        - VPC and networking
        - EC2 / ECS
        - RDS
        - S3
        - IAM
        - Load balancers
- CloudFormation (overview, comparison with Terraform)
- Ansible (overview for configuration management)
```

### 07.06 — Monitoring, Logging, Observability

```
- Three pillars: metrics, logs, traces
- Structured logging:
    - structlog in Python
    - JSON log format
    - Log levels in production
    - Correlation IDs
- Centralized logging:
    - ELK stack (Elasticsearch, Logstash, Kibana) — conceptual
    - CloudWatch Logs
    - Loki + Grafana (alternative)
- Metrics:
    - Prometheus:
        - Metric types: counter, gauge, histogram, summary
        - Python client (prometheus_client)
        - Instrumenting FastAPI
    - Grafana dashboards
    - CloudWatch metrics
- Distributed tracing:
    - OpenTelemetry
    - Jaeger
    - Trace context propagation
- Alerting:
    - Alert design (severity, actionability)
    - PagerDuty / Opsgenie (conceptual)
    - CloudWatch alarms
- Health checks and readiness probes
- SLI/SLO dashboards
```

### 07.07 — Kubernetes (Introduction to Intermediate)

```
- What Kubernetes is and why it exists
- Architecture: control plane, nodes, pods
- Core objects:
    - Pod
    - Deployment
    - Service (ClusterIP, NodePort, LoadBalancer)
    - ConfigMap
    - Secret
    - Ingress
    - PersistentVolumeClaim
    - Namespace
- kubectl CLI
- YAML manifests
- Scaling: HPA (Horizontal Pod Autoscaler)
- Rolling updates and rollbacks
- Liveness and readiness probes
- Resource limits and requests
- Helm (package manager):
    - Charts
    - Values
    - Installing community charts
- EKS (AWS managed Kubernetes) — overview
- When Kubernetes is overkill
```

### PHASE 07 CHAPTER PROJECT

```
Project 07: "Deploy the Banking API"
- Dockerize the Phase 05 Banking API
- Docker Compose for local development (app, PostgreSQL, Redis)
- Write GitHub Actions CI/CD pipeline:
    - Lint, test, type-check on PR
    - Build and push Docker image on merge to main
    - Deploy to AWS ECS Fargate
- Terraform for AWS infrastructure:
    - VPC, subnets, security groups
    - ECS cluster and service
    - RDS PostgreSQL
    - ElastiCache Redis
    - ALB
    - ECR
- Monitoring:
    - Prometheus metrics endpoint
    - CloudWatch logging
    - Health check endpoint
    - Basic Grafana dashboard (local)
- Full documentation: architecture diagram, deployment guide
- This completes Portfolio Project 02 (now production-grade)
```

---

## PHASE 08 / FINTECH, DATA SYSTEMS, AND SPECIALIZATION
**Follow the structure down.**
*Duration: 5-6 weeks at 6 hours/day*

This is where you specialize. Generic backend engineers are common. Backend engineers who understand financial systems and data-heavy architectures are not.

---

### 08.01 — Financial Data Concepts

```
- Financial instruments: equities, bonds, derivatives, forex (overview)
- Market data: order books, ticks, OHLCV candles
- Time series data
- Currency handling:
    - Why floats are dangerous for money
    - Decimal module
    - Storing money in cents/smallest unit
    - Currency codes (ISO 4217)
- Double-entry bookkeeping:
    - Debits and credits
    - Ledger design
    - Journal entries
    - Trial balance
- Payment processing:
    - Payment lifecycle (authorization, capture, settlement, refund)
    - Payment gateways (Stripe API as example)
    - Idempotency in payments
    - Reconciliation
- Interest calculations (simple, compound)
- Regulatory concepts:
    - KYC (Know Your Customer)
    - AML (Anti-Money Laundering)
    - PCI DSS (overview)
    - SOC 2 (overview)
    - Audit trails
```

### 08.02 — Data Pipeline Architecture

```
- What a data pipeline is
- ETL vs ELT
- Batch processing vs stream processing
- Apache Kafka deep dive:
    - Producer API (Python confluent-kafka)
    - Consumer API
    - Consumer groups and rebalancing
    - Exactly-once semantics
    - Schema Registry (Avro)
    - Kafka Connect (overview)
    - Kafka Streams vs external processing
- Apache Airflow:
    - What it is (workflow orchestration)
    - DAGs
    - Operators (PythonOperator, BashOperator, PostgresOperator)
    - Scheduling
    - Dependencies and task ordering
    - XCom (cross-task communication)
    - Sensors
    - Connections and hooks
    - Local development setup
- Data validation (Great Expectations or Pandera — overview)
- Idempotent pipelines
- Backfilling
```

### 08.03 — Streaming and Real-Time Systems

```
- Real-time vs near-real-time
- WebSocket protocols (revisited for data)
- Server-Sent Events (revisited)
- Long polling
- Redis Streams (for lightweight streaming)
- Kafka Streams patterns
- Windowed aggregations
- Watermarks and late data
- Back-pressure handling
- Time semantics: event time vs processing time
```

### 08.04 — High-Performance Python

```
- Python performance characteristics (GIL, interpreted)
- Profiling:
    - cProfile
    - line_profiler
    - memory_profiler
    - py-spy
- Concurrency:
    - Threading (threading module)
    - Thread pool (concurrent.futures.ThreadPoolExecutor)
    - Multiprocessing (multiprocessing module)
    - Process pool (concurrent.futures.ProcessPoolExecutor)
    - asyncio (revisited for performance)
    - When to use each (CPU-bound vs I/O-bound)
- Performance optimization:
    - Data structure choice
    - Generator pipelines for memory efficiency
    - __slots__
    - Caching (functools.lru_cache, @cache)
    - Lazy evaluation
    - Avoiding unnecessary copies
    - NumPy for numerical computation (overview)
- C extensions and alternatives:
    - Cython (overview)
    - ctypes (overview)
    - Rust extensions with PyO3 (awareness)
- Connection pooling in depth
- Batch processing patterns
- Memory management (reference counting, garbage collection)
```

### 08.05 — Security for Fintech

```
- OWASP Top 10 in depth:
    - Injection
    - Broken authentication
    - Sensitive data exposure
    - XML external entities
    - Broken access control
    - Security misconfiguration
    - Cross-site scripting (XSS)
    - Insecure deserialization
    - Using components with known vulnerabilities
    - Insufficient logging and monitoring
- Encryption:
    - Symmetric vs asymmetric
    - AES, RSA (concepts)
    - Hashing: SHA-256, bcrypt, argon2
    - Encryption at rest
    - Encryption in transit (TLS)
    - Python: cryptography library, hashlib, secrets
- Secrets management:
    - Environment variables
    - AWS Secrets Manager
    - HashiCorp Vault (concept)
    - Never commit secrets
- Audit logging:
    - What to log
    - Immutable audit trails
    - Compliance requirements
- API security:
    - Input validation (always)
    - Output encoding
    - Rate limiting
    - IP whitelisting
    - mTLS (mutual TLS)
    - API key rotation
- Dependency security:
    - pip audit
    - Dependabot
    - Snyk (overview)
- Security headers (HSTS, CSP, X-Frame-Options)
- Incident response basics
```

### PHASE 08 CHAPTER PROJECTS

```
Project 08A: "Transaction Ledger System"
- Double-entry bookkeeping engine
- PostgreSQL with proper decimal handling
- Immutable transaction log (append-only)
- Balance calculation from journal entries
- Reconciliation endpoint
- Audit trail with timestamps and actor
- Idempotent transaction processing
- Full test suite with edge cases (overdrafts, currency conversion)

Project 08B: "Market Data Pipeline"
- Ingest mock stock market data (or free API: Alpha Vantage, Yahoo Finance)
- Kafka producer publishing tick data
- Kafka consumer computing OHLCV candles (1-min, 5-min, 1-hour)
- Store in PostgreSQL (time-partitioned tables)
- Redis for latest price cache
- FastAPI REST endpoints for historical candle data
- FastAPI WebSocket for real-time price streaming
- Airflow DAG for daily aggregation and cleanup jobs
- Prometheus metrics for pipeline health
- Docker Compose for entire stack
- This is Portfolio Project 03 (evolved from Phase 06)
```

---

## PHASE 09 / SENIOR TO PRINCIPAL TRAJECTORY
**Follow the structure down.**
*Duration: Ongoing — this is the rest of your career*

This phase is not time-boxed. It represents the skill domains you develop over years from senior through staff through principal. I will detail what to learn at each level.

---

### 09.01 — Architecture Patterns (Senior Level)

```
- Layered architecture
- Hexagonal architecture (ports and adapters)
- Clean architecture
- Domain-Driven Design (DDD):
    - Ubiquitous language
    - Entities, value objects, aggregates
    - Repositories
    - Domain services
    - Application services
    - Bounded contexts (deep dive)
    - Context mapping
    - Anti-corruption layer
- SOLID principles applied to Python:
    - Single responsibility
    - Open/closed
    - Liskov substitution
    - Interface segregation
    - Dependency inversion
- Design patterns in Python:
    - Creational: factory, abstract factory, builder, singleton, prototype
    - Structural: adapter, decorator, facade, proxy, composite
    - Behavioral: observer, strategy, command, chain of responsibility, state, template method
    - Repository pattern (revisited)
    - Unit of work pattern
    - Specification pattern
- Twelve-Factor App methodology
```

### 09.02 — Distributed Systems (Staff Level)

```
- Fallacies of distributed computing
- Consensus algorithms:
    - Paxos (conceptual)
    - Raft (conceptual + implementation understanding)
- Distributed data:
    - Replication (leader-follower, multi-leader, leaderless)
    - Partitioning (sharding — range, hash, consistent hashing)
    - Replication lag and read-your-writes consistency
- Distributed transactions:
    - Two-phase commit (2PC)
    - Saga pattern (orchestration vs choreography)
    - Compensating transactions
- Distributed locking:
    - Redis distributed locks (Redlock)
    - ZooKeeper (conceptual)
- Clock synchronization:
    - Wall clock vs monotonic clock
    - Logical clocks (Lamport timestamps, vector clocks)
- Idempotency at scale
- Eventually consistent systems design
- CRDTs (Conflict-Free Replicated Data Types — awareness)
- Chaos engineering (concept)
- Books to read:
    - "Designing Data-Intensive Applications" (Martin Kleppmann) — essential
    - "System Design Interview" (Alex Xu) — both volumes
```

### 09.03 — Technical Leadership (Staff Level)

```
- Writing RFCs (Request for Comments):
    - Problem statement
    - Proposed solution
    - Alternatives considered
    - Migration plan
    - Risks and mitigations
    - Open questions
- Writing design documents:
    - Context and scope
    - Goals and non-goals
    - System architecture
    - Data model
    - API design
    - Security considerations
    - Observability
    - Testing strategy
    - Rollout plan
- Code review:
    - What to look for
    - How to give constructive feedback
    - How to receive feedback
- Architecture Decision Records (ADRs)
- Technical debt management:
    - Identifying tech debt
    - Quantifying impact
    - Prioritizing and scheduling
    - Communicating to non-technical stakeholders
- On-call and incident management:
    - Runbooks
    - Incident response process
    - Post-mortems (blameless)
    - SLO-based alerting
```

### 09.04 — Organizational Impact (Principal Level)

```
- Defining technical vision and strategy
- Technology radar (evaluating and adopting technologies)
- Cross-team architecture alignment
- Build vs buy decisions
- Vendor evaluation
- Platform engineering concepts
- Developer experience (DX) as a discipline
- Mentoring senior engineers
- Growing other staff engineers
- Influencing without authority
- Communication:
    - Writing for executives
    - Writing for engineers
    - Presenting technical strategy
- Books to read:
    - "Staff Engineer" (Will Larson)
    - "An Elegant Puzzle" (Will Larson)
    - "The Staff Engineer's Path" (Tanya Reilly)
    - "Accelerate" (Forsgren, Humble, Kim)
    - "Team Topologies" (Skelton, Pais)
```

### 09.05 — Continuous Learning Map

```
- Databases to learn when needed:
    - TimescaleDB (time-series on PostgreSQL)
    - ClickHouse (analytics)
    - Cassandra (wide-column)
    - Neo4j (graph)
- Languages to learn alongside Python:
    - Go (systems, high-performance services)
    - Rust (performance-critical components)
    - SQL (you should already be strong by now)
    - TypeScript (if you ever touch frontend or Node.js services)
- Cloud certifications path:
    - AWS Solutions Architect Associate
    - AWS Developer Associate
    - AWS Solutions Architect Professional
- Tools to pick up as needed:
    - Pulumi (IaC in Python)
    - ArgoCD (GitOps)
    - Istio (service mesh)
    - Vault (secrets)
    - Consul (service discovery)
- Open source contribution
- Speaking at meetups and conferences
- Writing technical blog posts
- Building in public
```

---

## PORTFOLIO PROJECTS — SUMMARY

These five projects tell a story from CLI tool to distributed system.

---

### Portfolio Project 01: CLI Finance Tracker
```
Phase built in: 01
Stack: Python, CSV, JSON, pytest
Demonstrates: Python fundamentals, OOP, file I/O, testing, CLI design
GitHub: clean README, CI with GitHub Actions, proper project structure
```

### Portfolio Project 02: Banking REST API (Production-Grade)
```
Phases built in: 05 + 07
Stack: FastAPI, PostgreSQL, SQLAlchemy, Redis, JWT, Docker, AWS ECS, Terraform, GitHub Actions
Demonstrates: API design, authentication, database design, caching, containerization, CI/CD, cloud deployment, monitoring
GitHub: architecture diagram, API docs, deployment guide, test coverage report
```

### Portfolio Project 03: Real-Time Market Data Pipeline
```
Phases built in: 06 + 08
Stack: Kafka, FastAPI, PostgreSQL, Redis, Airflow, WebSocket, Docker, Prometheus
Demonstrates: event-driven architecture, data pipelines, streaming, real-time systems, time-series data
GitHub: architecture diagram, data flow documentation, performance benchmarks
```

### Portfolio Project 04: Microservices Payment Platform
```
Phase built in: 08 + 09
Stack: FastAPI (multiple services), gRPC, Kafka, PostgreSQL, Redis, Docker, Kubernetes, Terraform
Services:
    - User service
    - Account service
    - Payment service (double-entry ledger)
    - Notification service
    - API Gateway
Demonstrates: microservices, distributed transactions (saga), inter-service communication, service discovery, container orchestration
GitHub: mono-repo or multi-repo with documentation, ADRs, deployment manifests
```

### Portfolio Project 05: Distributed Fintech System
```
Phase built in: 09
Stack: Everything from previous projects + CQRS, event sourcing, distributed tracing
This is the evolution of Project 04 with:
    - Event sourcing for the ledger
    - CQRS for read/write separation
    - OpenTelemetry distributed tracing
    - Chaos testing
    - Comprehensive observability dashboard
    - Technical design document
    - Architecture Decision Records
Demonstrates: principal-level system design, distributed systems, technical documentation
```

---

## TIMELINE ESTIMATE (at 6 hours/day)

```
Phase 00: 2-3 weeks
Phase 01: 4-5 weeks
Phase 02: 2-3 weeks
Phase 03: 4-5 weeks
Phase 04: 4-5 weeks
Phase 05: 5-6 weeks
Phase 06: 4-5 weeks
Phase 07: 5-6 weeks
Phase 08: 5-6 weeks
---
Total to job-ready (junior-mid): ~36-44 weeks (8-10 months)

Phase 09: Ongoing (years of practice, reading, building, leading)
- Senior: 2-4 years of professional experience
- Staff: 5-8 years of professional experience
- Principal: 10+ years of professional experience
```

---