```
Phase 3.1 → You know SQL and PostgreSQL
Phase 3.2 → You connect Python to PostgreSQL

The layers:

┌─────────────────────────────────────────────┐
│           YOUR PYTHON CODE                  │
│  user = User(name="Alice", email="a@b.com") │
├─────────────────────────────────────────────┤
│         SQLAlchemy ORM (Layer 3)            │
│  Translates Python objects ↔ SQL            │
├─────────────────────────────────────────────┤
│       SQLAlchemy Core (Layer 2)             │
│  SQL Expression Language                    │
├─────────────────────────────────────────────┤
│         psycopg2 (Layer 1)                  │
│  Raw PostgreSQL driver                      │
├─────────────────────────────────────────────┤
│           PostgreSQL                        │
│  The actual database                        │
└─────────────────────────────────────────────┘

You need to understand ALL layers:
  psycopg2  → foundation, sometimes used directly
  Core      → when you need SQL control with Python
  ORM       → 90% of daily backend work
  Alembic   → managing schema changes over time
```

---

## Setup

Bash

```
cd ~/projects/database_learning
source venv/bin/activate

# Install everything we need
pip install psycopg2-binary sqlalchemy alembic python-dotenv

# Create project structure
mkdir -p src/{models,repositories,migrations}
touch .env
touch src/__init__.py
touch src/models/__init__.py
touch src/repositories/__init__.py
```

Bash

```
# .env file
DATABASE_URL=postgresql://myapp_user:myapp_password@localhost:5432/learning_db
DATABASE_TEST_URL=postgresql://myapp_user:myapp_password@localhost:5432/learning_test_db
```

---

## 1. 🔌 psycopg2 — The Raw Driver

### What is psycopg2?

text

```
psycopg2 is Python's PostgreSQL adapter.
It's the actual bridge between Python and PostgreSQL.

Every higher-level tool (SQLAlchemy, Django ORM)
uses psycopg2 (or psycopg3) under the hood.

You won't always write raw psycopg2 in production,
but understanding it teaches you what's REALLY happening.
```

### Connecting and Basic Operations

Python

```
# src/raw_psycopg2.py
import psycopg2
from psycopg2 import sql
from psycopg2.extras import RealDictCursor, execute_values
import os
from dotenv import load_dotenv
from contextlib import contextmanager
from typing import Optional, List, Dict, Any

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")

# ─────────────────────────────────────────
# BASIC CONNECTION
# ─────────────────────────────────────────
def basic_connection_demo():
    # Create connection
    conn = psycopg2.connect(DATABASE_URL)

    # Create cursor (executes queries)
    cursor = conn.cursor()

    # Execute query
    cursor.execute("SELECT version();")

    # Fetch result
    result = cursor.fetchone()
    print(f"PostgreSQL version: {result[0]}")

    # Always close cursor and connection
    cursor.close()
    conn.close()

basic_connection_demo()


# ─────────────────────────────────────────
# CONNECTION WITH CONTEXT MANAGER (Better)
# ─────────────────────────────────────────
@contextmanager
def get_connection():
    """Context manager for database connections."""
    conn = psycopg2.connect(DATABASE_URL)
    try:
        yield conn
        conn.commit()       # commit if no exception
    except Exception:
        conn.rollback()     # rollback on exception
        raise
    finally:
        conn.close()        # always close


@contextmanager
def get_cursor(conn, cursor_factory=None):
    """Context manager for cursors."""
    kwargs = {}
    if cursor_factory:
        kwargs["cursor_factory"] = cursor_factory
    cursor = conn.cursor(**kwargs)
    try:
        yield cursor
    finally:
        cursor.close()


# ─────────────────────────────────────────
# CURSOR TYPES
# ─────────────────────────────────────────
def cursor_types_demo():
    with get_connection() as conn:
        # Default cursor: results as tuples
        with get_cursor(conn) as cur:
            cur.execute("SELECT id, name, email FROM users LIMIT 3")
            rows = cur.fetchall()
            print("Tuple cursor:")
            for row in rows:
                print(f"  {row}")
                print(f"  id={row[0]}, name={row[1]}")  # access by index

        # RealDictCursor: results as dicts (MUCH more useful)
        with get_cursor(conn, cursor_factory=RealDictCursor) as cur:
            cur.execute("SELECT id, name, email FROM users LIMIT 3")
            rows = cur.fetchall()
            print("\nDict cursor:")
            for row in rows:
                print(f"  {dict(row)}")
                print(f"  id={row['id']}, name={row['name']}")  # access by key

cursor_types_demo()
```

### CRUD with psycopg2
```
# ─────────────────────────────────────────
# CREATE — INSERT
# ─────────────────────────────────────────
def create_user(
    name: str,
    email: str,
    password_hash: str,
    role: str = "user"
) -> Dict[str, Any]:
    """Insert a new user and return the created record."""
    with get_connection() as conn:
        with get_cursor(conn, cursor_factory=RealDictCursor) as cur:
            # ⚠️  ALWAYS use parameterized queries
            # NEVER use string formatting for user input
            # This prevents SQL injection attacks!

            cur.execute(
                """
                INSERT INTO users (name, email, password_hash, role)
                VALUES (%s, %s, %s, %s)
                RETURNING id, name, email, role, created_at
                """,
                (name, email, password_hash, role)
                # ^ parameters are safely escaped by psycopg2
            )

            return dict(cur.fetchone())


# SQL INJECTION EXAMPLE — WHAT NOT TO DO:
def get_user_UNSAFE(email: str):  # NEVER DO THIS
    with get_connection() as conn:
        with get_cursor(conn) as cur:
            # ❌ DANGEROUS! SQL injection vulnerability
            query = f"SELECT * FROM users WHERE email = '{email}'"
            # If email = "' OR '1'='1" → returns ALL users!
            # If email = "'; DROP TABLE users; --" → deletes table!
            cur.execute(query)

# ✅ SAFE: parameterized query
def get_user_safe(email: str) -> Optional[Dict]:
    with get_connection() as conn:
        with get_cursor(conn, cursor_factory=RealDictCursor) as cur:
            cur.execute(
                "SELECT * FROM users WHERE email = %s",
                (email,)  # note: tuple, even for single parameter
            )
            row = cur.fetchone()
            return dict(row) if row else None


# ─────────────────────────────────────────
# READ — SELECT
# ─────────────────────────────────────────
def get_users_paginated(
    page: int = 1,
    page_size: int = 10,
    role: Optional[str] = None
) -> Dict[str, Any]:
    """Get paginated users with optional role filter."""
    with get_connection() as conn:
        with get_cursor(conn, cursor_factory=RealDictCursor) as cur:
            offset = (page - 1) * page_size

            # Build dynamic query safely
            if role:
                cur.execute(
                    """
                    SELECT id, name, email, role, is_active, created_at
                    FROM users
                    WHERE role = %s AND deleted_at IS NULL
                    ORDER BY created_at DESC
                    LIMIT %s OFFSET %s
                    """,
                    (role, page_size, offset)
                )
            else:
                cur.execute(
                    """
                    SELECT id, name, email, role, is_active, created_at
                    FROM users
                    WHERE deleted_at IS NULL
                    ORDER BY created_at DESC
                    LIMIT %s OFFSET %s
                    """,
                    (page_size, offset)
                )
            users = [dict(row) for row in cur.fetchall()]

            # Get total count
            if role:
                cur.execute(
                    "SELECT COUNT(*) FROM users WHERE role = %s AND deleted_at IS NULL",
                    (role,)
                )
            else:
                cur.execute(
                    "SELECT COUNT(*) FROM users WHERE deleted_at IS NULL"
                )
            total = cur.fetchone()[0]

            return {
                "users": users,
                "total": total,
                "page": page,
                "page_size": page_size,
                "total_pages": (total + page_size - 1) // page_size
            }


# ─────────────────────────────────────────
# UPDATE
# ─────────────────────────────────────────
def update_user(
    user_id: int,
    updates: Dict[str, Any]
) -> Optional[Dict[str, Any]]:
    """Update specific fields of a user."""
    if not updates:
        return None

    with get_connection() as conn:
        with get_cursor(conn, cursor_factory=RealDictCursor) as cur:
            # Build SET clause dynamically but SAFELY
            # sql.Identifier safely quotes column names
            set_clause = sql.SQL(", ").join(
                sql.SQL("{} = {}").format(
                    sql.Identifier(key),
                    sql.Placeholder()
                )
                for key in updates.keys()
            )

            query = sql.SQL(
                "UPDATE users SET {}, updated_at = NOW() "
                "WHERE id = {} AND deleted_at IS NULL "
                "RETURNING id, name, email, role, is_active, updated_at"
            ).format(
                set_clause,
                sql.Placeholder()
            )

            values = list(updates.values()) + [user_id]
            cur.execute(query, values)

            row = cur.fetchone()
            return dict(row) if row else None


# ─────────────────────────────────────────
# DELETE (soft delete)
# ─────────────────────────────────────────
def delete_user(user_id: int) -> bool:
    """Soft delete a user."""
    with get_connection() as conn:
        with get_cursor(conn) as cur:
            cur.execute(
                """
                UPDATE users
                SET deleted_at = NOW(), is_active = FALSE
                WHERE id = %s AND deleted_at IS NULL
                """,
                (user_id,)
            )
            return cur.rowcount > 0  # True if row was updated


# ─────────────────────────────────────────
# BULK INSERT — execute_values (fast!)
# ─────────────────────────────────────────
def bulk_create_users(users_data: List[Dict]) -> int:
    """
    Insert multiple users at once.
    execute_values is much faster than individual inserts.
    """
    with get_connection() as conn:
        with get_cursor(conn) as cur:
            # Prepare data as list of tuples
            values = [
                (user["name"], user["email"], user["password_hash"])
                for user in users_data
            ]

            execute_values(
                cur,
                """
                INSERT INTO users (name, email, password_hash)
                VALUES %s
                ON CONFLICT (email) DO NOTHING
                """,
                values,
                page_size=1000  # insert 1000 at a time
            )

            return cur.rowcount  # number inserted


# ─────────────────────────────────────────
# TRANSACTIONS
# ─────────────────────────────────────────
def transfer_funds(
    from_user_id: int,
    to_user_id: int,
    amount: float
) -> Dict[str, Any]:
    """
    Transfer funds between users atomically.
    Both operations succeed or both fail.
    """
    if amount <= 0:
        raise ValueError("Transfer amount must be positive")

    with get_connection() as conn:
        # The context manager handles commit/rollback
        with get_cursor(conn, cursor_factory=RealDictCursor) as cur:
            # Check sender has sufficient funds
            cur.execute(
                "SELECT id, name, balance FROM users WHERE id = %s FOR UPDATE",
                (from_user_id,)
                # FOR UPDATE: locks this row until transaction ends
                # Prevents race conditions with concurrent transfers
            )
            sender = cur.fetchone()
            if not sender:
                raise ValueError(f"Sender {from_user_id} not found")
            if sender["balance"] < amount:
                raise ValueError(
                    f"Insufficient funds: {sender['balance']} < {amount}"
                )

            # Check receiver exists
            cur.execute(
                "SELECT id, name FROM users WHERE id = %s FOR UPDATE",
                (to_user_id,)
            )
            receiver = cur.fetchone()
            if not receiver:
                raise ValueError(f"Receiver {to_user_id} not found")

            # Perform transfer
            cur.execute(
                "UPDATE users SET balance = balance - %s WHERE id = %s",
                (amount, from_user_id)
            )
            cur.execute(
                "UPDATE users SET balance = balance + %s WHERE id = %s",
                (amount, to_user_id)
            )

            return {
                "success": True,
                "from": dict(sender),
                "to": dict(receiver),
                "amount": amount,
                "message": f"Transferred ${amount} from {sender['name']} to {receiver['name']}"
            }
            # Context manager commits here if no exception


# Test everything
print("=== Testing psycopg2 ===")

# Create users
user1 = create_user("Test Alice", "test_alice@test.com", "hash1", "admin")
print(f"Created: {user1}")

user2 = create_user("Test Bob", "test_bob@test.com", "hash2")
print(f"Created: {user2}")

# Get user
found = get_user_safe("test_alice@test.com")
print(f"Found: {found['name']}")

# Paginate
result = get_users_paginated(page=1, page_size=5)
print(f"Total users: {result['total']}")

# Update
updated = update_user(user1["id"], {"name": "Alice Updated", "role": "admin"})
print(f"Updated: {updated}")

# Bulk insert
bulk_data = [
    {"name": f"Bulk User {i}", "email": f"bulk{i}@test.com", "password_hash": f"hash{i}"}
    for i in range(10)
]
count = bulk_create_users(bulk_data)
print(f"Bulk inserted: {count} users")
```

### Connection Pooling

Python

```
# ─────────────────────────────────────────
# CONNECTION POOLING
# ─────────────────────────────────────────
"""
Problem without pooling:
  Every request creates a new database connection.
  Creating a connection takes ~50-100ms.
  With 1000 requests/second → 50,000-100,000ms overhead!
  Plus: PostgreSQL has limited max connections (~100 default)

Solution: Connection Pool
  Pre-create N connections at startup.
  Borrow a connection for each request.
  Return it to pool when done.
  Dramatically reduces latency and resource usage.
"""
from psycopg2 import pool

class DatabasePool:
    """Manages a pool of database connections."""

    _instance = None
    _pool = None

    @classmethod
    def initialize(
        cls,
        dsn: str,
        min_connections: int = 2,
        max_connections: int = 10
    ):
        """Create the connection pool (call once at startup)."""
        if cls._pool is None:
            cls._pool = pool.ThreadedConnectionPool(
                minconn=min_connections,
                maxconn=max_connections,
                dsn=dsn
            )
            print(f"Pool initialized: {min_connections}-{max_connections} connections")

    @classmethod
    @contextmanager
    def get_connection(cls):
        """Get a connection from the pool."""
        if cls._pool is None:
            raise RuntimeError("Pool not initialized. Call DatabasePool.initialize() first.")

        conn = cls._pool.getconn()
        try:
            yield conn
            conn.commit()
        except Exception:
            conn.rollback()
            raise
        finally:
            cls._pool.putconn(conn)  # return to pool

    @classmethod
    def close_all(cls):
        """Close all connections (call at shutdown)."""
        if cls._pool:
            cls._pool.closeall()


# Initialize at application startup
DatabasePool.initialize(DATABASE_URL, min_connections=2, max_connections=10)

# Use pooled connections
def get_user_pooled(user_id: int) -> Optional[Dict]:
    with DatabasePool.get_connection() as conn:
        with get_cursor(conn, cursor_factory=RealDictCursor) as cur:
            cur.execute(
                "SELECT id, name, email FROM users WHERE id = %s",
                (user_id,)
            )
            row = cur.fetchone()
            return dict(row) if row else None

result = get_user_pooled(1)
print(f"Pooled query result: {result}")

# Close pool at application shutdown
DatabasePool.close_all()
```

---

## 2. 🏗️ SQLAlchemy Core

### What is SQLAlchemy Core?

text

```
SQLAlchemy has TWO layers:

Core:   SQL Expression Language
        Write SQL using Python constructs
        More control than ORM
        Less verbose than raw strings

ORM:    Object Relational Mapper
        Map Python classes to tables
        Work with objects, not SQL
        Most common in production

Core sits between raw SQL and ORM.
Use it when you need SQL control but want Python syntax.
```

Python

```
# src/sqlalchemy_core.py
from sqlalchemy import (
    create_engine, text, MetaData, Table, Column,
    Integer, String, Boolean, DateTime, Numeric,
    ForeignKey, Index, func, select, insert,
    update, delete, and_, or_, not_
)
from sqlalchemy.pool import QueuePool
from datetime import datetime, timezone
import os
from dotenv import load_dotenv

load_dotenv()
DATABASE_URL = os.getenv("DATABASE_URL")

# ─────────────────────────────────────────
# CREATE ENGINE — the connection factory
# ─────────────────────────────────────────
engine = create_engine(
    DATABASE_URL,

    # Connection pool configuration
    pool_size=5,            # permanent connections
    max_overflow=10,        # extra connections allowed
    pool_pre_ping=True,     # test connections before use
    pool_recycle=3600,      # recycle after 1 hour

    # Logging (shows generated SQL — great for debugging)
    echo=False,             # set True to see all SQL
)

print("Engine created successfully")

# ─────────────────────────────────────────
# EXECUTING RAW SQL with text()
# ─────────────────────────────────────────
def raw_sql_with_sqlalchemy():
    """Execute raw SQL safely through SQLAlchemy."""
    with engine.connect() as conn:
        # text() marks string as SQL — still parameterized!
        result = conn.execute(
            text("SELECT id, name, email FROM users WHERE role = :role LIMIT :limit"),
            {"role": "admin", "limit": 5}
        )
        # result is a CursorResult
        rows = result.fetchall()
        for row in rows:
            print(f"  {row.id}: {row.name} ({row.email})")

        # Or access as dict
        result2 = conn.execute(text("SELECT * FROM users LIMIT 1"))
        row = result2.mappings().fetchone()  # returns dict-like
        if row:
            print(f"  Dict access: {dict(row)}")

raw_sql_with_sqlalchemy()
```

---

## 3. 🗺️ SQLAlchemy ORM — The Main Event

### Defining Models

Python

```
# src/models/base.py
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker
from sqlalchemy import Column, DateTime, func
from datetime import datetime, timezone
import os
from dotenv import load_dotenv

load_dotenv()
DATABASE_URL = os.getenv("DATABASE_URL")

# Create engine
engine = create_engine(
    DATABASE_URL,
    pool_size=5,
    max_overflow=10,
    pool_pre_ping=True,
    echo=False,  # Set True to debug SQL
)

# Session factory — creates database sessions
SessionLocal = sessionmaker(
    bind=engine,
    autocommit=False,   # we handle commits manually
    autoflush=False,    # we control when to flush
)

# Base class for all models
class Base(DeclarativeBase):
    """Base class that all models inherit from."""
    pass


# ─────────────────────────────────────────
# MIXIN: Reusable columns
# ─────────────────────────────────────────
class TimestampMixin:
    """Add created_at and updated_at to any model."""
    created_at = Column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False
    )
    updated_at = Column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False
    )


class SoftDeleteMixin:
    """Add soft delete to any model."""
    deleted_at = Column(DateTime(timezone=True), nullable=True)

    @property
    def is_deleted(self) -> bool:
        return self.deleted_at is not None
```

Python

```
# src/models/user.py
from sqlalchemy import (
    Column, Integer, String, Boolean,
    DateTime, Numeric, Text, CheckConstraint,
    Index, func
)
from sqlalchemy.orm import relationship, validates
from sqlalchemy.dialects.postgresql import JSONB
from .base import Base, TimestampMixin, SoftDeleteMixin
import re


class User(Base, TimestampMixin, SoftDeleteMixin):
    """User model — maps to 'users' table."""

    __tablename__ = "users"

    # ─────────────────────────────────────────
    # COLUMNS
    # ─────────────────────────────────────────
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), nullable=False, unique=True, index=True)
    password_hash = Column(String(255), nullable=False)
    age = Column(Integer, nullable=True)
    bio = Column(Text, nullable=True)

    role = Column(
        String(20),
        nullable=False,
        default="user",
        server_default="user"
    )

    is_active = Column(Boolean, nullable=False, default=True, server_default="true")
    is_verified = Column(Boolean, nullable=False, default=False, server_default="false")

    balance = Column(Numeric(10, 2), nullable=False, default=0)

    # PostgreSQL-specific: JSONB for flexible data
    metadata_ = Column("metadata", JSONB, nullable=False, default={})

    # ─────────────────────────────────────────
    # TABLE-LEVEL CONSTRAINTS
    # ─────────────────────────────────────────
    __table_args__ = (
        CheckConstraint(
            "role IN ('user', 'admin', 'moderator')",
            name="check_user_role"
        ),
        CheckConstraint(
            "age >= 0 AND age <= 150",
            name="check_user_age"
        ),
        # Composite index — fast for filtering active users by role
        Index("idx_users_role_active", "role", "is_active"),
        # Partial index — only index non-deleted users
        Index(
            "idx_users_email_active",
            "email",
            postgresql_where="deleted_at IS NULL"
        ),
    )

    # ─────────────────────────────────────────
    # RELATIONSHIPS
    # ─────────────────────────────────────────
    # One user has many posts
    posts = relationship(
        "Post",
        back_populates="author",
        lazy="select",          # load posts when accessed
        cascade="all, delete-orphan"  # delete posts if user deleted
    )

    # One user has many comments
    comments = relationship(
        "Comment",
        back_populates="author",
        lazy="select"
    )

    # ─────────────────────────────────────────
    # VALIDATORS
    # ─────────────────────────────────────────
    @validates("email")
    def validate_email(self, key, email):
        """Validate email format before saving."""
        if not email:
            raise ValueError("Email is required")
        email = email.lower().strip()
        pattern = r'^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$'
        if not re.match(pattern, email):
            raise ValueError(f"Invalid email format: {email}")
        return email

    @validates("name")
    def validate_name(self, key, name):
        if not name or len(name.strip()) < 2:
            raise ValueError("Name must be at least 2 characters")
        return name.strip().title()

    @validates("role")
    def validate_role(self, key, role):
        valid_roles = {"user", "admin", "moderator"}
        if role not in valid_roles:
            raise ValueError(f"Role must be one of {valid_roles}")
        return role

    # ─────────────────────────────────────────
    # PROPERTIES
    # ─────────────────────────────────────────
    @property
    def is_admin(self) -> bool:
        return self.role == "admin"

    @property
    def public_data(self) -> dict:
        """Safe data to return in API responses."""
        return {
            "id": self.id,
            "name": self.name,
            "email": self.email,
            "role": self.role,
            "is_active": self.is_active,
            "is_verified": self.is_verified,
            "created_at": self.created_at.isoformat() if self.created_at else None,
        }

    # ─────────────────────────────────────────
    # DUNDER METHODS
    # ─────────────────────────────────────────
    def __repr__(self) -> str:
        return f"<User id={self.id} name={self.name!r} email={self.email!r}>"

    def __str__(self) -> str:
        return f"{self.name} <{self.email}>"
```

Python

```
# src/models/post.py
from sqlalchemy import (
    Column, Integer, String, Text, Boolean,
    DateTime, ForeignKey, Index,
    CheckConstraint, func
)
from sqlalchemy.orm import relationship, validates
from sqlalchemy.dialects.postgresql import ARRAY
from sqlalchemy import String as SAString
from .base import Base, TimestampMixin, SoftDeleteMixin


class Post(Base, TimestampMixin, SoftDeleteMixin):
    """Post model."""

    __tablename__ = "posts"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(500), nullable=False)
    content = Column(Text, nullable=False)
    slug = Column(String(500), unique=True, index=True)
    status = Column(String(20), nullable=False, default="draft")

    # Foreign key to users table
    author_id = Column(
        Integer,
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True
    )

    view_count = Column(Integer, nullable=False, default=0)
    like_count = Column(Integer, nullable=False, default=0)

    # PostgreSQL array
    tags = Column(ARRAY(SAString), nullable=False, default=[])

    published_at = Column(DateTime(timezone=True), nullable=True)

    __table_args__ = (
        CheckConstraint(
            "status IN ('draft', 'published', 'archived')",
            name="check_post_status"
        ),
        Index("idx_posts_author_status", "author_id", "status"),
        Index(
            "idx_posts_published",
            "published_at",
            postgresql_where="status = 'published'"
        ),
    )

    # Relationships
    author = relationship("User", back_populates="posts")
    comments = relationship(
        "Comment",
        back_populates="post",
        cascade="all, delete-orphan"
    )

    @validates("status")
    def validate_status(self, key, status):
        valid = {"draft", "published", "archived"}
        if status not in valid:
            raise ValueError(f"Status must be one of {valid}")
        return status

    @validates("slug")
    def validate_slug(self, key, slug):
        if slug:
            import re
            slug = slug.lower().strip()
            slug = re.sub(r'[^a-z0-9-]', '-', slug)
            slug = re.sub(r'-+', '-', slug)
            slug = slug.strip('-')
        return slug

    @property
    def is_published(self) -> bool:
        return self.status == "published"

    def __repr__(self) -> str:
        return f"<Post id={self.id} title={self.title!r} status={self.status!r}>"


class Comment(Base, TimestampMixin):
    """Comment model with nested comment support."""

    __tablename__ = "comments"

    id = Column(Integer, primary_key=True, index=True)
    content = Column(Text, nullable=False)
    is_edited = Column(Boolean, nullable=False, default=False)

    author_id = Column(
        Integer,
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True
    )
    post_id = Column(
        Integer,
        ForeignKey("posts.id", ondelete="CASCADE"),
        nullable=False,
        index=True
    )
    # Self-referencing for nested comments
    parent_id = Column(
        Integer,
        ForeignKey("comments.id", ondelete="CASCADE"),
        nullable=True,
        index=True
    )

    # Relationships
    author = relationship("User", back_populates="comments")
    post = relationship("Post", back_populates="comments")
    replies = relationship(
        "Comment",
        backref=__import__("sqlalchemy.orm", fromlist=["backref"]).backref(
            "parent", remote_side=[id]
        ),
        lazy="select"
    )

    def __repr__(self) -> str:
        return f"<Comment id={self.id} post_id={self.post_id}>"
```

### Session Management

Python

```
# src/database.py
from contextlib import contextmanager
from sqlalchemy.orm import Session
from .models.base import SessionLocal

@contextmanager
def get_db():
    """
    Context manager for database sessions.
    Ensures session is always closed.
    Automatically commits on success, rolls back on failure.
    """
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()


# Usage:
# with get_db() as db:
#     user = db.query(User).filter(User.id == 1).first()

# ─────────────────────────────────────────
# For FastAPI: dependency injection style
# ─────────────────────────────────────────
def get_db_session():
    """
    FastAPI dependency for database sessions.
    Used as: db: Session = Depends(get_db_session)
    """
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()
```

### ORM CRUD Operations

Python

```
# src/repositories/user_repository.py
from sqlalchemy.orm import Session
from sqlalchemy import func, and_, or_
from typing import Optional, List, Dict, Any
from ..models.user import User
from ..models.post import Post


class UserRepository:
    """
    Repository pattern: encapsulate all database operations for User.

    Benefits:
      - Single place for all user database logic
      - Easy to test (mock the repository)
      - Can swap database implementation
      - Clean separation of concerns
    """

    def __init__(self, session: Session):
        self.session = session

    # ─────────────────────────────────────────
    # CREATE
    # ─────────────────────────────────────────
    def create(
        self,
        name: str,
        email: str,
        password_hash: str,
        role: str = "user",
        **kwargs
    ) -> User:
        """Create a new user."""
        user = User(
            name=name,
            email=email,
            password_hash=password_hash,
            role=role,
            **kwargs
        )
        self.session.add(user)
        self.session.flush()  # get the ID without committing
        return user

    def bulk_create(self, users_data: List[Dict]) -> List[User]:
        """Create multiple users at once."""
        users = [User(**data) for data in users_data]
        self.session.add_all(users)
        self.session.flush()
        return users

    # ─────────────────────────────────────────
    # READ
    # ─────────────────────────────────────────
    def get_by_id(self, user_id: int) -> Optional[User]:
        """Get user by ID."""
        return self.session.query(User).filter(
            User.id == user_id,
            User.deleted_at.is_(None)
        ).first()

    def get_by_email(self, email: str) -> Optional[User]:
        """Get user by email (case insensitive)."""
        return self.session.query(User).filter(
            func.lower(User.email) == email.lower(),
            User.deleted_at.is_(None)
        ).first()

    def get_all(
        self,
        page: int = 1,
        page_size: int = 10,
        role: Optional[str] = None,
        is_active: Optional[bool] = None,
        search: Optional[str] = None,
        order_by: str = "created_at",
        order_dir: str = "desc"
    ) -> Dict[str, Any]:
        """
        Get paginated users with filtering and search.
        """
        query = self.session.query(User).filter(
            User.deleted_at.is_(None)
        )

        # Apply filters
        if role:
            query = query.filter(User.role == role)
        if is_active is not None:
            query = query.filter(User.is_active == is_active)
        if search:
            search_term = f"%{search}%"
            query = query.filter(
                or_(
                    User.name.ilike(search_term),
                    User.email.ilike(search_term)
                )
            )

        # Get total count BEFORE pagination
        total = query.count()

        # Apply ordering
        order_column = getattr(User, order_by, User.created_at)
        if order_dir == "desc":
            query = query.order_by(order_column.desc())
        else:
            query = query.order_by(order_column.asc())

        # Apply pagination
        offset = (page - 1) * page_size
        users = query.offset(offset).limit(page_size).all()

        return {
            "items": users,
            "total": total,
            "page": page,
            "page_size": page_size,
            "total_pages": (total + page_size - 1) // page_size,
            "has_next": page * page_size < total,
            "has_prev": page > 1
        }

    def get_with_posts(self, user_id: int) -> Optional[User]:
        """Get user with their posts loaded (eager loading)."""
        from sqlalchemy.orm import joinedload
        return self.session.query(User).options(
            joinedload(User.posts)  # load posts in same query
        ).filter(
            User.id == user_id,
            User.deleted_at.is_(None)
        ).first()

    def exists_by_email(self, email: str) -> bool:
        """Check if email already exists."""
        return self.session.query(
            self.session.query(User).filter(
                func.lower(User.email) == email.lower()
            ).exists()
        ).scalar()

    def count(self, role: Optional[str] = None) -> int:
        """Count users."""
        query = self.session.query(func.count(User.id)).filter(
            User.deleted_at.is_(None)
        )
        if role:
            query = query.filter(User.role == role)
        return query.scalar()

    # ─────────────────────────────────────────
    # UPDATE
    # ─────────────────────────────────────────
    def update(self, user_id: int, **updates) -> Optional[User]:
        """Update user fields."""
        user = self.get_by_id(user_id)
        if not user:
            return None

        for key, value in updates.items():
            if hasattr(user, key):
                setattr(user, key, value)

        self.session.flush()
        return user

    def activate(self, user_id: int) -> Optional[User]:
        return self.update(user_id, is_active=True)

    def deactivate(self, user_id: int) -> Optional[User]:
        return self.update(user_id, is_active=False)

    def verify(self, user_id: int) -> Optional[User]:
        return self.update(user_id, is_verified=True)

    def promote_to_admin(self, user_id: int) -> Optional[User]:
        return self.update(user_id, role="admin")

    # ─────────────────────────────────────────
    # DELETE
    # ─────────────────────────────────────────
    def soft_delete(self, user_id: int) -> bool:
        """Soft delete a user."""
        from datetime import datetime, timezone
        user = self.get_by_id(user_id)
        if not user:
            return False
        user.deleted_at = datetime.now(timezone.utc)
        user.is_active = False
        self.session.flush()
        return True

    def hard_delete(self, user_id: int) -> bool:
        """Permanently delete a user."""
        user = self.session.query(User).filter(User.id == user_id).first()
        if not user:
            return False
        self.session.delete(user)
        self.session.flush()
        return True

    # ─────────────────────────────────────────
    # COMPLEX QUERIES
    # ─────────────────────────────────────────
    def get_user_stats(self) -> List[Dict]:
        """
        Get statistics per role.
        Demonstrates aggregation with ORM.
        """
        results = self.session.query(
            User.role,
            func.count(User.id).label("total"),
            func.count(User.id).filter(User.is_active).label("active"),
            func.avg(User.age).label("avg_age")
        ).filter(
            User.deleted_at.is_(None)
        ).group_by(
            User.role
        ).all()

        return [
            {
                "role": row.role,
                "total": row.total,
                "active": row.active,
                "avg_age": round(float(row.avg_age or 0), 1)
            }
            for row in results
        ]

    def get_users_with_post_count(
        self,
        min_posts: int = 0
    ) -> List[Dict]:
        """Get users with their post count using JOIN."""
        from sqlalchemy import outerjoin
        results = self.session.query(
            User.id,
            User.name,
            User.email,
            User.role,
            func.count(Post.id).label("post_count")
        ).outerjoin(
            Post,
            and_(Post.author_id == User.id, Post.deleted_at.is_(None))
        ).filter(
            User.deleted_at.is_(None)
        ).group_by(
            User.id, User.name, User.email, User.role
        ).having(
            func.count(Post.id) >= min_posts
        ).order_by(
            func.count(Post.id).desc()
        ).all()

        return [
            {
                "id": row.id,
                "name": row.name,
                "email": row.email,
                "role": row.role,
                "post_count": row.post_count
            }
            for row in results
        ]


# ─────────────────────────────────────────
# USAGE DEMONSTRATION
# ─────────────────────────────────────────
from .database import get_db
from .models.base import Base, engine

# Create tables
Base.metadata.create_all(bind=engine)

def demo_orm_operations():
    """Demonstrate ORM CRUD operations."""

    print("=== ORM CRUD Demo ===\n")

    with get_db() as session:
        repo = UserRepository(session)

        # CREATE
        print("1. Creating users...")
        user1 = repo.create(
            name="alice smith",  # validator will title-case this
            email="ALICE@TEST.COM",  # validator will lowercase
            password_hash="hashed_password",
            role="admin"
        )
        print(f"   Created: {user1}")

        user2 = repo.create(
            name="bob jones",
            email="bob@test.com",
            password_hash="hashed_password2"
        )
        print(f"   Created: {user2}")

        # READ
        print("\n2. Reading users...")
        found = repo.get_by_email("alice@test.com")
        print(f"   Found by email: {found}")

        paginated = repo.get_all(page=1, page_size=5)
        print(f"   Total users: {paginated['total']}")
        print(f"   Users on page 1: {len(paginated['items'])}")

        # Search
        searched = repo.get_all(search="alice")
        print(f"   Search 'alice': {len(searched['items'])} results")

        # UPDATE
        print("\n3. Updating user...")
        updated = repo.update(user1.id, bio="Backend developer")
        print(f"   Updated: bio={updated.bio}")

        repo.promote_to_admin(user2.id)
        print(f"   Promoted bob to: {user2.role}")

        # STATS
        print("\n4. User statistics...")
        stats = repo.get_user_stats()
        for stat in stats:
            print(f"   {stat['role']}: {stat['total']} total, "
                  f"{stat['active']} active")

        # DELETE (soft)
        print("\n5. Soft deleting user...")
        deleted = repo.soft_delete(user2.id)
        print(f"   Deleted: {deleted}")

        still_exists = repo.get_by_id(user2.id)
        print(f"   Still in DB: {still_exists is None}")  # True (filtered out)

demo_orm_operations()
```

---

## 4. ⚡ Loading Strategies

Python

```
# src/loading_strategies.py
"""
The N+1 Problem:

Get 10 users, then for each user get their posts:
  Query 1: SELECT * FROM users LIMIT 10          ← 1 query
  Query 2: SELECT * FROM posts WHERE author_id=1  ← N queries
  Query 3: SELECT * FROM posts WHERE author_id=2
  ...
  Query 11: SELECT * FROM posts WHERE author_id=10

Total: N+1 = 11 queries instead of 1!

Solution: eager loading (joinedload or selectinload)
"""
from sqlalchemy.orm import Session, joinedload, selectinload, lazyload
from .models.user import User
from .models.post import Post
from .database import get_db

def demonstrate_loading():
    with get_db() as session:
        print("=== Loading Strategies ===\n")

        # ─────────────────────────────────────────
        # LAZY LOADING (default — N+1 problem!)
        # ─────────────────────────────────────────
        print("1. Lazy Loading (N+1 problem):")
        users = session.query(User).limit(3).all()
        for user in users:
            # Each access triggers a NEW query!
            post_count = len(user.posts)  # ← SQL query here
            print(f"   {user.name}: {post_count} posts")
        # Total queries: 1 (users) + 3 (posts for each user) = 4

        # ─────────────────────────────────────────
        # JOINEDLOAD — single JOIN query
        # ─────────────────────────────────────────
        print("\n2. Joined Load (single query with JOIN):")
        users_with_posts = session.query(User).options(
            joinedload(User.posts)  # loads posts in SAME query via JOIN
        ).limit(3).all()

        for user in users_with_posts:
            # No extra query! posts already loaded
            post_count = len(user.posts)
            print(f"   {user.name}: {post_count} posts")
        # Total queries: 1 (users JOIN posts)
        # Best for: loading one related object (many-to-one)
        # WARNING: can cause row multiplication with collections

        # ─────────────────────────────────────────
        # SELECTINLOAD — separate IN query
        # ─────────────────────────────────────────
        print("\n3. Select In Load (separate IN query):")
        users_select = session.query(User).options(
            selectinload(User.posts)  # loads posts with WHERE IN query
        ).limit(3).all()

        for user in users_select:
            # No extra query! posts already loaded
            post_count = len(user.posts)
            print(f"   {user.name}: {post_count} posts")
        # Total queries: 2
        #   1: SELECT * FROM users LIMIT 3
        #   2: SELECT * FROM posts WHERE author_id IN (1, 2, 3)
        # Best for: loading collections (one-to-many)

        # ─────────────────────────────────────────
        # NESTED LOADING
        # ─────────────────────────────────────────
        print("\n4. Nested Loading (users → posts → comments):")
        users_deep = session.query(User).options(
            selectinload(User.posts).selectinload(Post.comments)
        ).limit(2).all()

        for user in users_deep:
            for post in user.posts:
                comment_count = len(post.comments)
                print(f"   {user.name} / {post.title}: {comment_count} comments")
        # Total queries: 3
        #   1: SELECT users
        #   2: SELECT posts WHERE author_id IN (...)
        #   3: SELECT comments WHERE post_id IN (...)

        # ─────────────────────────────────────────
        # WHEN TO USE WHICH
        # ─────────────────────────────────────────
        """
        lazy loading:    default, only when you KNOW you won't need related data
        joinedload:      many-to-one (post.author), single related object
        selectinload:    one-to-many (user.posts), collections
        contains_eager:  when you already wrote the JOIN in your query
        """


demonstrate_loading()
```

---

## 5. 🔄 Alembic — Database Migrations

### What is Alembic?

text

```
Alembic manages database schema changes over time.

Problem without migrations:
  - Developer A adds a column
  - Developer B doesn't have it
  - Production doesn't have it
  - Server crashes after deploy because column missing

Problem solved with Alembic:
  - Every schema change is a "migration" file
  - Files committed to Git
  - Run "alembic upgrade head" on any machine
  - Everyone gets identical schema

Think of it like Git for your database schema.
```

### Setting Up Alembic

Bash

```
# Install (already done)
# pip install alembic

# Initialize Alembic in your project
cd ~/projects/database_learning
alembic init migrations

# This creates:
# migrations/
#   versions/         ← migration files go here
#   env.py            ← configuration
#   script.py.mako    ← template for new migrations
# alembic.ini         ← main config file
```

Python

```
# Edit migrations/env.py to use your models and URL

# migrations/env.py
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
import os
import sys
from dotenv import load_dotenv

# Add project to path
sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))

load_dotenv()

# Import your Base and all models
from src.models.base import Base
from src.models.user import User      # must import so tables are registered
from src.models.post import Post      # must import so tables are registered
from src.models.post import Comment   # must import so tables are registered

config = context.config

# Override sqlalchemy.url from environment variable
config.set_main_option(
    "sqlalchemy.url",
    os.getenv("DATABASE_URL", "")
)

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# Tell Alembic about your models
target_metadata = Base.metadata  # ← THIS IS KEY


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode (generate SQL without connecting)."""
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    """Run migrations in 'online' mode (connect to database)."""
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,       # detect column type changes
            compare_server_default=True,  # detect default changes
        )
        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

ini

```
# alembic.ini — update this section
[alembic]
# sqlalchemy.url is set in env.py from environment variable
sqlalchemy.url = driver://user:pass@localhost/dbname

script_location = migrations
file_template = %%(year)d%%(month).2d%%(day).2d_%%(hour).2d%%(minute).2d_%%(rev)s_%%(slug)s
# This gives files like: 20240115_1432_a1b2c3d4_add_users_table.py
```

### Creating and Running Migrations

Bash

```
# ─────────────────────────────────────────
# CREATE YOUR FIRST MIGRATION
# ─────────────────────────────────────────

# Auto-detect changes from your models vs current database
alembic revision --autogenerate -m "create users posts comments tables"

# This creates: migrations/versions/20240115_1432_abc123_create_users_posts_comments_tables.py

# Review the generated migration BEFORE applying!
# Alembic isn't perfect — always review the generated SQL
```

Python

```
# Example generated migration file:
# migrations/versions/20240115_1432_abc123_create_initial_tables.py

"""create initial tables

Revision ID: abc123
Revises:
Create Date: 2024-01-15 14:32:00.000000
"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql

revision = 'abc123'
down_revision = None  # first migration has no parent
branch_labels = None
depends_on = None


def upgrade() -> None:
    """Apply the migration (forward)."""

    # Create users table
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('name', sa.String(100), nullable=False),
        sa.Column('email', sa.String(255), nullable=False),
        sa.Column('password_hash', sa.String(255), nullable=False),
        sa.Column('age', sa.Integer(), nullable=True),
        sa.Column('bio', sa.Text(), nullable=True),
        sa.Column('role', sa.String(20), nullable=False, server_default='user'),
        sa.Column('is_active', sa.Boolean(), nullable=False, server_default='true'),
        sa.Column('is_verified', sa.Boolean(), nullable=False, server_default='false'),
        sa.Column('balance', sa.Numeric(10, 2), nullable=False, server_default='0'),
        sa.Column('metadata', postgresql.JSONB(), nullable=False, server_default='{}'),
        sa.Column('created_at', sa.DateTime(timezone=True), server_default=sa.func.now(), nullable=False),
        sa.Column('updated_at', sa.DateTime(timezone=True), server_default=sa.func.now(), nullable=False),
        sa.Column('deleted_at', sa.DateTime(timezone=True), nullable=True),
        sa.CheckConstraint("role IN ('user', 'admin', 'moderator')", name='check_user_role'),
        sa.CheckConstraint('age >= 0 AND age <= 150', name='check_user_age'),
        sa.PrimaryKeyConstraint('id')
    )
    op.create_index('ix_users_id', 'users', ['id'], unique=False)
    op.create_index('ix_users_email', 'users', ['email'], unique=True)
    op.create_index('idx_users_role_active', 'users', ['role', 'is_active'])

    # Create posts table
    op.create_table(
        'posts',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('title', sa.String(500), nullable=False),
        sa.Column('content', sa.Text(), nullable=False),
        sa.Column('slug', sa.String(500), nullable=True),
        sa.Column('status', sa.String(20), nullable=False, server_default='draft'),
        sa.Column('author_id', sa.Integer(), nullable=False),
        sa.Column('view_count', sa.Integer(), nullable=False, server_default='0'),
        sa.Column('like_count', sa.Integer(), nullable=False, server_default='0'),
        sa.Column('tags', postgresql.ARRAY(sa.String()), nullable=False, server_default='{}'),
        sa.Column('published_at', sa.DateTime(timezone=True), nullable=True),
        sa.Column('created_at', sa.DateTime(timezone=True), server_default=sa.func.now(), nullable=False),
        sa.Column('updated_at', sa.DateTime(timezone=True), server_default=sa.func.now(), nullable=False),
        sa.Column('deleted_at', sa.DateTime(timezone=True), nullable=True),
        sa.CheckConstraint("status IN ('draft', 'published', 'archived')", name='check_post_status'),
        sa.ForeignKeyConstraint(['author_id'], ['users.id'], ondelete='CASCADE'),
        sa.PrimaryKeyConstraint('id')
    )
    op.create_index('ix_posts_slug', 'posts', ['slug'], unique=True)
    op.create_index('idx_posts_author_status', 'posts', ['author_id', 'status'])


def downgrade() -> None:
    """Reverse the migration (backward)."""
    op.drop_table('posts')
    op.drop_table('users')
```

### Writing Manual Migrations

Python

```
# Sometimes you need to write migrations manually
# For data migrations, complex schema changes, etc.

# migrations/versions/20240120_1000_def456_add_phone_to_users.py

"""add phone column to users

Revision ID: def456
Revises: abc123
Create Date: 2024-01-20 10:00:00
"""
from alembic import op
import sqlalchemy as sa

revision = 'def456'
down_revision = 'abc123'  # parent migration
branch_labels = None
depends_on = None


def upgrade() -> None:
    # Add column
    op.add_column(
        'users',
        sa.Column('phone', sa.String(20), nullable=True)
    )

    # Add index on new column
    op.create_index('idx_users_phone', 'users', ['phone'])

    # Add constraint
    op.create_check_constraint(
        'check_phone_format',
        'users',
        "phone ~ '^\\+?[0-9]{10,15}$' OR phone IS NULL"
    )


def downgrade() -> None:
    op.drop_constraint('check_phone_format', 'users')
    op.drop_index('idx_users_phone', table_name='users')
    op.drop_column('users', 'phone')


# ─────────────────────────────────────────
# DATA MIGRATION — changing existing data
# ─────────────────────────────────────────

# migrations/versions/20240125_0900_ghi789_normalize_user_emails.py

"""normalize existing user emails to lowercase

Revision ID: ghi789
Revises: def456
Create Date: 2024-01-25 09:00:00
"""
from alembic import op
import sqlalchemy as sa

revision = 'ghi789'
down_revision = 'def456'


def upgrade() -> None:
    # Use op.execute for data manipulation
    op.execute(
        "UPDATE users SET email = LOWER(email) WHERE email != LOWER(email)"
    )

    # For large datasets, do it in batches
    connection = op.get_bind()
    connection.execute(sa.text("""
        UPDATE users
        SET name = INITCAP(LOWER(TRIM(name)))
        WHERE name != INITCAP(LOWER(TRIM(name)))
        AND deleted_at IS NULL
    """))


def downgrade() -> None:
    # Can't undo normalization (data was already normalized or lowercase)
    # This migration is not reversible
    pass
```

### Alembic Commands

Bash

```
# ─────────────────────────────────────────
# ESSENTIAL ALEMBIC COMMANDS
# ─────────────────────────────────────────

# Check current migration status
alembic current

# Show migration history
alembic history
alembic history --verbose

# Apply all pending migrations
alembic upgrade head

# Apply next migration only
alembic upgrade +1

# Apply to specific revision
alembic upgrade abc123

# Rollback last migration
alembic downgrade -1

# Rollback all migrations
alembic downgrade base

# Rollback to specific revision
alembic downgrade abc123

# Generate new migration (auto-detect from models)
alembic revision --autogenerate -m "description of change"

# Generate empty migration (write manually)
alembic revision -m "manual migration description"

# Show SQL without applying (dry run)
alembic upgrade head --sql

# Mark current state without running migrations
# (useful when you already have tables from CREATE TABLE)
alembic stamp head

# ─────────────────────────────────────────
# TYPICAL WORKFLOW
# ─────────────────────────────────────────

# 1. Make changes to your SQLAlchemy models
# 2. Generate migration
alembic revision --autogenerate -m "add user phone number"
# 3. Review the generated file!!! Don't trust it blindly
# 4. Apply migration
alembic upgrade head
# 5. Commit models AND migration to Git

# ─────────────────────────────────────────
# IN PRODUCTION
# ─────────────────────────────────────────
# Run migrations as part of your deployment:
# 1. Deploy new code
# 2. Run: alembic upgrade head
# 3. Restart application servers
```

---

## 6. 🏭 Raw SQL vs ORM Trade-offs

Python

```
# src/comparison.py
"""
When to use raw SQL vs ORM — practical guide.
"""
from sqlalchemy import text
from .database import get_db
from .models.user import User
from .models.post import Post

def when_to_use_each():
    """
    Raw psycopg2 / text():
      ✅ Extremely complex queries ORM can't express
      ✅ Performance-critical operations
      ✅ Bulk operations with specific optimization
      ✅ Database-specific features
      ✅ Reporting queries with complex aggregations

    SQLAlchemy ORM:
      ✅ Standard CRUD operations (90% of backend work)
      ✅ When model validation/relationships matter
      ✅ When you want Python objects, not raw data
      ✅ Portability (switching databases)
      ✅ Automatic change tracking

    SQLAlchemy Core (text()):
      ✅ Complex queries with some structure
      ✅ Bulk operations
      ✅ When you need SQL control but want parameterization
    """
    pass


def complex_query_example():
    """
    Example: complex reporting query that's better as raw SQL.
    """
    with get_db() as session:
        # This complex analytics query is clearer in SQL
        result = session.execute(text("""
            WITH monthly_stats AS (
                SELECT
                    DATE_TRUNC('month', created_at) AS month,
                    COUNT(*) AS new_users,
                    COUNT(*) FILTER (WHERE role = 'admin') AS new_admins
                FROM users
                WHERE created_at >= NOW() - INTERVAL '6 months'
                AND deleted_at IS NULL
                GROUP BY DATE_TRUNC('month', created_at)
            ),
            post_stats AS (
                SELECT
                    DATE_TRUNC('month', created_at) AS month,
                    COUNT(*) AS new_posts,
                    SUM(view_count) AS total_views
                FROM posts
                WHERE created_at >= NOW() - INTERVAL '6 months'
                AND deleted_at IS NULL
                GROUP BY DATE_TRUNC('month', created_at)
            )
            SELECT
                ms.month,
                ms.new_users,
                ms.new_admins,
                COALESCE(ps.new_posts, 0) AS new_posts,
                COALESCE(ps.total_views, 0) AS total_views
            FROM monthly_stats ms
            LEFT JOIN post_stats ps ON ms.month = ps.month
            ORDER BY ms.month DESC
        """))

        rows = result.mappings().all()
        return [dict(row) for row in rows]


def orm_vs_raw_performance():
    """
    Demonstrate when raw SQL is faster for bulk operations.
    """
    # ORM bulk insert (slower — creates Python objects)
    with get_db() as session:
        users = [
            User(
                name=f"User {i}",
                email=f"user{i}@test.com",
                password_hash=f"hash{i}"
            )
            for i in range(1000)
        ]
        session.add_all(users)
        # All 1000 Python objects created in memory!

    # Raw SQL bulk insert (faster — no Python objects)
    with get_db() as session:
        session.execute(
            text("""
                INSERT INTO users (name, email, password_hash)
                SELECT
                    'Bulk User ' || i,
                    'bulk' || i || '@test.com',
                    'bulkhash' || i
                FROM generate_series(1, 1000) AS i
                ON CONFLICT (email) DO NOTHING
            """)
        )
        # One SQL statement, no Python objects!
```

---

## 7. 🔧 Complete Working Example

Python

```
# src/service.py — putting it all together
from typing import Optional, List, Dict, Any
from sqlalchemy.orm import Session
from .repositories.user_repository import UserRepository
from .models.user import User
from .database import get_db


class UserService:
    """
    Service layer: business logic on top of repository.

    Architecture:
      Router/API endpoint
          ↓
      Service (business logic)
          ↓
      Repository (database operations)
          ↓
      Database
    """

    def __init__(self, session: Session):
        self.session = session
        self.repo = UserRepository(session)

    def register(
        self,
        name: str,
        email: str,
        password: str,
        role: str = "user"
    ) -> Dict[str, Any]:
        """
        Register a new user.
        Business logic: validate, hash password, create user.
        """
        # Check if email already taken
        if self.repo.exists_by_email(email):
            raise ValueError(f"Email {email} is already registered")

        # Hash password (simplified — use bcrypt in real code)
        import hashlib
        password_hash = hashlib.sha256(password.encode()).hexdigest()

        # Create user
        user = self.repo.create(
            name=name,
            email=email,
            password_hash=password_hash,
            role=role
        )

        return user.public_data

    def login(self, email: str, password: str) -> Dict[str, Any]:
        """Authenticate a user."""
        user = self.repo.get_by_email(email)

        if not user:
            raise ValueError("Invalid email or password")

        if not user.is_active:
            raise ValueError("Account is deactivated")

        if user.is_deleted:
            raise ValueError("Account not found")

        # Verify password (simplified)
        import hashlib
        password_hash = hashlib.sha256(password.encode()).hexdigest()
        if user.password_hash != password_hash:
            raise ValueError("Invalid email or password")

        return {
            "user": user.public_data,
            "message": "Login successful"
        }

    def get_profile(self, user_id: int) -> Dict[str, Any]:
        """Get user profile."""
        user = self.repo.get_by_id(user_id)
        if not user:
            raise ValueError(f"User {user_id} not found")
        return user.public_data

    def update_profile(
        self,
        user_id: int,
        requesting_user_id: int,
        **updates
    ) -> Dict[str, Any]:
        """Update user profile (user can only update their own)."""
        # Authorization check
        if user_id != requesting_user_id:
            requesting_user = self.repo.get_by_id(requesting_user_id)
            if not requesting_user or not requesting_user.is_admin:
                raise PermissionError("Cannot update another user's profile")

        # Don't allow role changes through profile update
        updates.pop("role", None)
        updates.pop("password_hash", None)

        user = self.repo.update(user_id, **updates)
        if not user:
            raise ValueError(f"User {user_id} not found")

        return user.public_data

    def delete_account(
        self,
        user_id: int,
        requesting_user_id: int
    ) -> bool:
        """Delete user account."""
        if user_id != requesting_user_id:
            requesting_user = self.repo.get_by_id(requesting_user_id)
            if not requesting_user or not requesting_user.is_admin:
                raise PermissionError("Cannot delete another user's account")

        return self.repo.soft_delete(user_id)


# ─────────────────────────────────────────
# USAGE
# ─────────────────────────────────────────
def main():
    print("=== Service Layer Demo ===\n")

    with get_db() as session:
        service = UserService(session)

        # Register
        try:
            user = service.register(
                name="test user",
                email="testuser@example.com",
                password="SecurePassword123"
            )
            print(f"Registered: {user['name']} ({user['email']})")
        except ValueError as e:
            print(f"Registration failed: {e}")

        # Login
        try:
            result = service.login(
                "testuser@example.com",
                "SecurePassword123"
            )
            print(f"Logged in: {result['user']['name']}")
        except ValueError as e:
            print(f"Login failed: {e}")


if __name__ == "__main__":
    main()
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│              PYTHON + DATABASE ARCHITECTURE                    │
│                                                                │
│  API LAYER                                                     │
│    FastAPI routes, request/response handling                   │
│           ↓                                                    │
│  SERVICE LAYER                                                 │
│    Business logic, validation, authorization                   │
│           ↓                                                    │
│  REPOSITORY LAYER                                              │
│    Database operations, queries, transactions                  │
│           ↓                                                    │
│  ORM (SQLAlchemy)                                              │
│    Python ↔ SQL translation, session management               │
│           ↓                                                    │
│  DRIVER (psycopg2)                                             │
│    Raw database protocol                                       │
│           ↓                                                    │
│  PostgreSQL                                                    │
│                                                                │
│  KEY TOOLS:                                                    │
│  psycopg2     → raw driver, use for bulk ops                  │
│  SQLAlchemy   → ORM for daily CRUD, relationships             │
│  Alembic      → migration management                          │
│  Connection pool → efficiency under load                       │
│                                                                │
│  LOADING STRATEGIES:                                           │
│  lazy load    → default, careful of N+1                       │
│  joinedload   → many-to-one relationships                     │
│  selectinload → one-to-many collections                       │
│                                                                │
│  ALEMBIC WORKFLOW:                                             │
│  Edit models → autogenerate → review → upgrade head          │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is the difference between psycopg2, SQLAlchemy Core,
    and SQLAlchemy ORM?
2.  Why should you NEVER use string formatting for SQL queries?
3.  What is connection pooling and why does it matter?
4.  What is the difference between session.add() and session.flush()?
5.  What is the N+1 problem? How do you fix it?
6.  When would you use joinedload vs selectinload?
7.  What is Alembic and why do you need it?
8.  What does autogenerate do in Alembic?
9.  What is the Repository pattern? Why use it?
10. What is the difference between commit and flush?
11. When would you use raw SQL over the ORM?
12. What does FOR UPDATE do in a transaction?
```

---

## 🛠️ Practice Exercises

Python

```
# Exercise 1: Complete Repository
# Build a PostRepository class with:
#   create(title, content, author_id, tags=[]) -> Post
#   get_by_id(post_id) -> Optional[Post]
#   get_by_slug(slug) -> Optional[Post]
#   get_published(page, page_size, tag=None) -> Dict
#   publish(post_id) -> Optional[Post]
#   increment_views(post_id) -> None
#   get_trending(limit=10) -> List[Post]
#     (most viewed in last 7 days)
#   search(query, page, page_size) -> Dict

# Exercise 2: Alembic Migration
# Your users table needs:
#   - Add: last_login TIMESTAMP WITH TIME ZONE
#   - Add: login_count INTEGER DEFAULT 0
#   - Add: profile_picture_url TEXT
#   - Add index on last_login
# Write the Alembic migration file (upgrade + downgrade).

# Exercise 3: Avoid N+1
# This code has the N+1 problem. Fix it:
# def get_posts_with_author_names(session, limit=10):
#     posts = session.query(Post).limit(limit).all()
#     return [
#         {"title": p.title, "author": p.author.name}
#         for p in posts
#     ]

# Exercise 4: Transaction
# Write a service method: transfer_post_ownership(
#     post_id, from_user_id, to_user_id
# )
# Requirements:
#   - Verify post belongs to from_user_id
#   - Verify to_user_id exists and is active
#   - Transfer the post
#   - Log the transfer (insert into an audit_log table)
#   - All in one transaction

# Exercise 5: Complex Query
# Using SQLAlchemy ORM, write a query that returns:
# Top 5 authors by total engagement
# (engagement = views + likes * 2 + comments * 3)
# Include: author name, post count, total engagement
# Only published posts in the last 30 days
```

---

## ✅ Phase 3.2 Complete!

**You now know:**

text

```
✅ psycopg2 — raw PostgreSQL driver
✅ Parameterized queries (SQL injection prevention)
✅ Connection pooling
✅ Bulk operations with execute_values
✅ Transactions with psycopg2
✅ SQLAlchemy engine and sessions
✅ Defining ORM models with all column types
✅ Relationships (one-to-many, many-to-many, self-referencing)
✅ Validators on ORM models
✅ Repository pattern for database operations
✅ Full CRUD with SQLAlchemy ORM
✅ Lazy vs eager loading (joinedload, selectinload)
✅ The N+1 problem and how to fix it
✅ Alembic migrations (setup, autogenerate, manual)
✅ Migration commands (upgrade, downgrade, history)
✅ Service layer pattern
✅ Raw SQL vs ORM trade-offs
```