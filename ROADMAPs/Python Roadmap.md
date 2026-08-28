# Python Roadmap - Secondary Language

> [!info] Secondary Language
> Python is your second language. You will spend roughly 30% of your study time here.
> Start Python after completing Java Phase 2 (around Week 14).
> Python will be especially useful for your data engineering interest.

---
## Phase 0: Python Basics (Weeks 1-3, start after Java Phase 2)

> [!tip] Goal
> Learn Python syntax. Notice how it differs from Java. Python will feel much shorter and simpler.

### Topics
1. [[Python - Installation and Setup - venv pip]]
2. [[Python - Variables and Data Types - Dynamic Typing]]
3. [[Python - Operators]]
4. [[Python - Control Flow - if elif else]]
5. [[Python - Loops - for while - range enumerate]]
6. [[Python - Lists and List Comprehensions]]
7. [[Python - Tuples]]
8. [[Python - Dictionaries and Dict Comprehensions]]
9. [[Python - Sets]]
10. [[Python - Strings and f-strings]]
11. [[Python - Functions - args kwargs default params]]
12. [[Python - Lambda Functions]]
13. [[Python - File Handling - read write with statement]]
14. [[Python - Error Handling - try except finally raise]]
15. [[Python - Modules and Imports]]
16. [[Python - Virtual Environments and pip]]

### Practice Projects
- **File Organizer Script**: Scans a downloads folder, moves files into subfolders by extension (Images, Documents, Videos).
- **CSV Data Summarizer**: Reads a CSV file, calculates statistics, writes a summary report.

### Java vs Python Note
> [!note] Key Differences from Java
> No semicolons. No curly braces (uses indentation). No explicit type declarations (dynamic typing). No `public static void main`. Lists instead of ArrayLists. Dictionaries instead of HashMaps. Everything is an object but you do not need to create classes for simple scripts.

---
## Phase 1: Python OOP and Intermediate (Weeks 4-7)

> [!tip] Goal
> Understand Python's OOP model. Learn Pythonic patterns that have no direct Java equivalent.

### Topics
1. [[Python - Classes and Objects - __init__ self]]
2. [[Python - Inheritance and super()]]
3. [[Python - Multiple Inheritance and MRO]]
4. [[Python - Encapsulation - Name Mangling]]
5. [[Python - Property Decorator - @property]]
6. [[Python - Magic Dunder Methods - __str__ __repr__ __eq__ __len__]]
7. [[Python - Class Methods and Static Methods - @classmethod @staticmethod]]
8. [[Python - Abstract Base Classes - abc module]]
9. [[Python - Decorators - Function and Class Decorators]]
10. [[Python - Generators and yield]]
11. [[Python - Context Managers - __enter__ __exit__ contextlib]]
12. [[Python - Type Hints - typing module]]
13. [[Python - Dataclasses - @dataclass]]
14. [[Python - Iterators and Iterables]]
15. [[Python - Testing with pytest]]

### Practice Projects
- **Web Scraper**: Use `requests` and `BeautifulSoup` to scrape a news website. Store articles in a dataclass. Save to JSON. Write pytest tests.
- **Expense Tracker CLI**: OOP-based expense tracker. Categories, budgets, reports. Use decorators for logging. Use type hints throughout.

---
## Phase 2: Python Backend with FastAPI (Weeks 8-13)

> [!tip] Goal
> Build REST APIs with Python. FastAPI is the modern choice and closest to Spring Boot in philosophy.

### Topics
1. [[Python - FastAPI Introduction and Setup]]
2. [[Python - FastAPI Path and Query Parameters]]
3. [[Python - FastAPI Request Body and Pydantic Models]]
4. [[Python - Pydantic Validation In Depth]]
5. [[Python - FastAPI Response Models and Status Codes]]
6. [[Python - FastAPI Dependency Injection]]
7. [[Python - SQLAlchemy ORM Setup with FastAPI]]
8. [[Python - SQLAlchemy Models and Relationships]]
9. [[Python - Database Migrations with Alembic]]
10. [[Python - FastAPI Authentication - JWT]]
11. [[Python - FastAPI Middleware and CORS]]
12. [[Python - FastAPI Background Tasks]]
13. [[Python - FastAPI Testing with TestClient]]
14. [[Python - FastAPI Auto Documentation - Swagger]]
15. [[Python - Project Structure for FastAPI]]

### Practice Projects (Portfolio Worthy)
- **URL Shortener API**: Create short URLs, redirect, track click counts. JWT auth for admin dashboard. SQLAlchemy with PostgreSQL. Deployed on Render.
- **Blog API with FastAPI**: Similar to your Java blog API but in Python. Compare the two implementations in your notes. This comparison will impress interviewers.

### Real Backend Connection
> [!note] Why FastAPI
> FastAPI is the fastest growing Python web framework. It uses type hints for automatic validation (like Spring validation). It generates Swagger docs automatically (like Springdoc). Many companies use Python for microservices alongside a Java main system. Knowing both makes you very versatile.

---
## Phase 3: Python for Data Engineering (Weeks 14-19)

> [!tip] Goal
> Use Python for data processing. This complements your Java data engineering track.

### Topics
1. [[Python - NumPy Basics - Arrays Operations]]
2. [[Python - Pandas Basics - DataFrame Series]]
3. [[Python - Pandas Data Cleaning - Missing Values Duplicates]]
4. [[Python - Pandas GroupBy and Aggregation]]
5. [[Python - Pandas Merging and Joining]]
6. [[Python - Working with CSV JSON Parquet Excel]]
7. [[Python - Data Visualization Basics - Matplotlib Seaborn]]
8. [[Python - Web Scraping for Data Collection]]
9. [[Python - API Data Extraction - requests library]]
10. [[Python - Scheduling Scripts - cron and schedule library]]
11. [[Python - PySpark Basics - Python Spark Interface]]
12. [[Python - Airflow DAGs - Python Operators]]

### Practice Projects
- **Data Cleaning Pipeline**: Download a messy real-world dataset (Kaggle). Clean it with Pandas. Handle missing values, normalize formats, remove duplicates. Output clean Parquet files.
- **ETL Pipeline with Airflow**: Extract data from a public API (weather, cryptocurrency). Transform with Pandas. Load into PostgreSQL. Schedule with Airflow to run daily.

### Real Backend Connection
> [!note] Why this matters
> Python dominates data engineering. Most data pipelines at companies are written in Python even when the main backend is Java. Your Java Spark knowledge plus Python Pandas/Airflow knowledge makes you a strong data engineering candidate.

---
## Phase 4: Python Advanced Backend (Weeks 20-24)

> [!tip] Goal
> Master advanced Python backend patterns for high-performance applications.

### Topics
1. [[Python - Async Programming - asyncio await]]
2. [[Python - Async FastAPI Endpoints]]
3. [[Python - Celery for Background Tasks]]
4. [[Python - Redis with Python - redis-py]]
5. [[Python - WebSockets with FastAPI]]
6. [[Python - gRPC with Python]]
7. [[Python - Docker for Python Applications]]
8. [[Python - CI CD for Python - GitHub Actions]]
9. [[Python - Performance Profiling - cProfile]]
10. [[Python - Design Patterns in Python]]

### Practice Projects
- **Async Data Processing Service**: FastAPI endpoint accepts large file uploads. Celery worker processes the file asynchronously. Results stored in Redis. Client polls for status.

---
## Python Job Readiness Checklist

> [!warning] For roles that require Python backend or data engineering:

- [ ] Completed all Python phases
- [ ] At least 1 FastAPI project deployed
- [ ] At least 1 data pipeline project
- [ ] Comfortable reading and writing Python and Java in the same week
- [ ] Understanding of when to use Python vs Java for a task

---
## When to Use Which Language

| Scenario | Use |
|----------|-----|
| Main backend API for enterprise | Java + Spring Boot |
| Quick microservice or prototype | Python + FastAPI |
| Data processing pipeline | Python + Pandas/Spark |
| Real-time streaming at scale | Java + Kafka/Flink |
| Machine learning integration | Python |
| High-performance concurrent system | Java |
| Scripting and automation | Python |
| Large team enterprise project | Java |

---