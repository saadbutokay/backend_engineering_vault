```
You've learned three frameworks.
Each can solve the same problem.

But HOW you organize your code determines:
  ❌ Spaghetti code vs ✅ Clean architecture
  ❌ Untestable mess vs ✅ 90% test coverage
  ❌ One person understands it vs ✅ Any engineer can navigate
  ❌ Adding a feature = 3 days vs ✅ Adding a feature = 3 hours

This phase covers:
  - Clean/Layered architecture
  - Repository pattern
  - Unit of Work pattern
  - Service layer pattern
  - Factory pattern
  - Dependency injection
  - Configuration management
  - Logging best practices
  - Error handling strategies

These patterns are framework-agnostic.
You'll use them in FastAPI, Django, Flask — all of them.
They separate junior engineers from mid/senior engineers.
```

---

## Setup

Bash

```
cd ~/projects
mkdir patterns_demo
cd patterns_demo
python3 -m venv venv
source venv/bin/activate
pip install fastapi uvicorn sqlalchemy pydantic \
    pydantic-settings structlog loguru python-dotenv
mkdir -p src/{core,models,schemas,repositories,services,api,infrastructure}
touch src/__init__.py
code .
```

---

## 1. 🏗️ Clean / Layered Architecture

### The Problem Without Architecture

Python

```
# BAD — everything in one place (seen in real codebases)
# This is what happens without structure

from fastapi import FastAPI
import psycopg2
import json
import hashlib

app = FastAPI()

@app.post("/register")
def register(data: dict):
    # Validation mixed with business logic mixed with DB access
    if not data.get("email"):
        return {"error": "email required"}

    conn = psycopg2.connect("postgresql://...")
    cur = conn.cursor()

    cur.execute("SELECT id FROM users WHERE email = %s", (data["email"],))
    if cur.fetchone():
        return {"error": "exists"}

    password_hash = hashlib.sha256(data["password"].encode()).hexdigest()
    cur.execute(
        "INSERT INTO users (email, password_hash) VALUES (%s, %s) RETURNING id",
        (data["email"], password_hash)
    )
    user_id = cur.fetchone()[0]
    conn.commit()
    cur.close()
    conn.close()

    # Business logic mixed into route
    if data.get("role") == "admin" and not data.get("admin_secret"):
        return {"error": "admin secret required"}

    return {"id": user_id, "email": data["email"]}

# Problems:
# - Can't test without a real database
# - Can't reuse the logic elsewhere
# - Change the database? Rewrite everything
# - One huge function doing 5 different things
# - No separation of concerns
```

### The Solution — Layered Architecture

text

```
┌─────────────────────────────────────────────────────────┐
│                    API LAYER                            │
│  FastAPI routers / Django views / Flask blueprints      │
│  Responsibility: HTTP in, HTTP out                      │
│  Knows about: HTTP, Pydantic schemas                    │
│  Does NOT know about: database, business rules          │
├─────────────────────────────────────────────────────────┤
│                  SERVICE LAYER                          │
│  Business logic — the rules of your application        │
│  Responsibility: orchestrate, enforce business rules    │
│  Knows about: domain objects, repositories              │
│  Does NOT know about: HTTP, databases                   │
├─────────────────────────────────────────────────────────┤
│               REPOSITORY LAYER                         │
│  Data access — all database operations                  │
│  Responsibility: CRUD for one entity type               │
│  Knows about: database, ORM models                      │
│  Does NOT know about: business rules, HTTP              │
├─────────────────────────────────────────────────────────┤
│                  DATA LAYER                             │
│  Database (PostgreSQL, Redis, MongoDB)                  │
└─────────────────────────────────────────────────────────┘

Rule: Each layer ONLY talks to the layer directly below it.
      API → Service → Repository → Database

Benefits:
  ✅ Test each layer independently
  ✅ Swap database without changing business logic
  ✅ Swap framework without changing business logic
  ✅ Clear responsibilities
  ✅ Easy to navigate for new team members
```

---

## 2. 📁 Project Structure

text

```
# The structure we'll build and explain

src/
├── core/                    ← cross-cutting concerns
│   ├── config.py            ← settings management
│   ├── database.py          ← database connection
│   ├── security.py          ← auth utilities
│   ├── exceptions.py        ← custom exceptions
│   ├── logging.py           ← logging setup
│   └── deps.py              ← dependency injection
│
├── models/                  ← SQLAlchemy ORM models
│   ├── base.py              ← base model with timestamps
│   ├── user.py
│   └── post.py
│
├── schemas/                 ← Pydantic schemas (input/output)
│   ├── common.py            ← shared schemas
│   ├── user.py
│   └── post.py
│
├── repositories/            ← database operations
│   ├── base.py              ← generic repository
│   ├── user_repo.py
│   └── post_repo.py
│
├── services/                ← business logic
│   ├── user_service.py
│   ├── post_service.py
│   └── auth_service.py
│
├── api/                     ← HTTP layer (FastAPI)
│   ├── v1/
│   │   ├── router.py        ← combine all routers
│   │   ├── auth.py
│   │   ├── users.py
│   │   └── posts.py
│   └── middleware.py
│
├── infrastructure/          ← external services
│   ├── email.py
│   ├── storage.py
│   └── cache.py
│
└── main.py                  ← app entry point
```

---

## 3. 🏛️ Repository Pattern

### What & Why

text

```
Repository = A class that encapsulates all database
             operations for ONE entity type.

Instead of writing SQL/ORM queries scattered everywhere,
you put ALL database operations for User in UserRepository.

Benefits:
  ✅ One place for all User database logic
  ✅ Easy to test (mock the repository)
  ✅ Easy to switch databases (change only repository)
  ✅ Consistent querying patterns
  ✅ Other layers don't touch the database directly
```

Python

```
# src/repositories/base.py
from typing import TypeVar, Generic, Type, Optional, List, Any
from sqlalchemy.orm import Session
from sqlalchemy import func

ModelType = TypeVar("ModelType")


class BaseRepository(Generic[ModelType]):
    """
    Generic base repository with common CRUD operations.
    All entity repositories inherit from this.
    """

    def __init__(self, model: Type[ModelType], db: Session):
        self.model = model
        self.db = db

    # ─────────────────────────────────────────
    # CREATE
    # ─────────────────────────────────────────
    def create(self, **kwargs) -> ModelType:
        """Create and flush (but not commit) a new record."""
        instance = self.model(**kwargs)
        self.db.add(instance)
        self.db.flush()     # assigns ID, doesn't commit
        return instance

    def bulk_create(self, records: List[dict]) -> List[ModelType]:
        """Create multiple records efficiently."""
        instances = [self.model(**data) for data in records]
        self.db.add_all(instances)
        self.db.flush()
        return instances

    # ─────────────────────────────────────────
    # READ
    # ─────────────────────────────────────────
    def get_by_id(self, id: int) -> Optional[ModelType]:
        """Get record by primary key."""
        return self.db.get(self.model, id)

    def get_all(
        self,
        skip: int = 0,
        limit: int = 100,
        **filters
    ) -> List[ModelType]:
        """Get multiple records with optional filters."""
        query = self.db.query(self.model)
        for key, value in filters.items():
            if value is not None:
                query = query.filter(
                    getattr(self.model, key) == value
                )
        return query.offset(skip).limit(limit).all()

    def count(self, **filters) -> int:
        """Count records matching filters."""
        query = self.db.query(func.count(self.model.id))
        for key, value in filters.items():
            if value is not None:
                query = query.filter(
                    getattr(self.model, key) == value
                )
        return query.scalar()

    def exists(self, **filters) -> bool:
        """Check if any record matches filters."""
        query = self.db.query(self.model.id)
        for key, value in filters.items():
            query = query.filter(getattr(self.model, key) == value)
        return query.first() is not None

    # ─────────────────────────────────────────
    # UPDATE
    # ─────────────────────────────────────────
    def update(self, instance: ModelType, **updates) -> ModelType:
        """Update fields on an existing instance."""
        for key, value in updates.items():
            if hasattr(instance, key) and value is not None:
                setattr(instance, key, value)
        self.db.flush()
        return instance

    def update_by_id(self, id: int, **updates) -> Optional[ModelType]:
        """Update a record by ID."""
        instance = self.get_by_id(id)
        if not instance:
            return None
        return self.update(instance, **updates)

    # ─────────────────────────────────────────
    # DELETE
    # ─────────────────────────────────────────
    def delete(self, instance: ModelType) -> None:
        """Hard delete a record."""
        self.db.delete(instance)
        self.db.flush()

    def delete_by_id(self, id: int) -> bool:
        """Hard delete by ID. Returns True if deleted."""
        instance = self.get_by_id(id)
        if not instance:
            return False
        self.delete(instance)
        return True
```

Python

```
# src/repositories/user_repo.py
from typing import Optional, List, Tuple
from sqlalchemy.orm import Session
from sqlalchemy import func, or_
from datetime import datetime, timezone

from .base import BaseRepository
from src.models.user import User


class UserRepository(BaseRepository[User]):
    """
    All database operations for User model.
    Extends BaseRepository with User-specific queries.
    """

    def __init__(self, db: Session):
        super().__init__(User, db)

    # ─────────────────────────────────────────
    # USER-SPECIFIC QUERIES
    # ─────────────────────────────────────────
    def get_active_by_id(self, user_id: int) -> Optional[User]:
        """Get non-deleted, active user by ID."""
        return self.db.query(User).filter(
            User.id == user_id,
            User.deleted_at.is_(None)
        ).first()

    def get_by_email(self, email: str) -> Optional[User]:
        """Find user by email (case-insensitive)."""
        return self.db.query(User).filter(
            func.lower(User.email) == email.lower().strip(),
            User.deleted_at.is_(None)
        ).first()

    def email_taken(self, email: str, exclude_id: Optional[int] = None) -> bool:
        """Check if email is already registered."""
        query = self.db.query(User.id).filter(
            func.lower(User.email) == email.lower()
        )
        if exclude_id:
            query = query.filter(User.id != exclude_id)
        return query.first() is not None

    def search(
        self,
        query: str,
        role: Optional[str] = None,
        is_active: Optional[bool] = None,
        skip: int = 0,
        limit: int = 20
    ) -> Tuple[List[User], int]:
        """
        Search users by name/email with filters.
        Returns (users, total_count).
        """
        db_query = self.db.query(User).filter(
            User.deleted_at.is_(None)
        )

        if query:
            term = f"%{query}%"
            db_query = db_query.filter(
                or_(
                    User.name.ilike(term),
                    User.email.ilike(term)
                )
            )
        if role:
            db_query = db_query.filter(User.role == role)
        if is_active is not None:
            db_query = db_query.filter(User.is_active == is_active)

        total = db_query.count()
        users = (
            db_query
            .order_by(User.created_at.desc())
            .offset(skip)
            .limit(limit)
            .all()
        )
        return users, total

    def soft_delete(self, user: User) -> User:
        """Mark user as deleted without removing from DB."""
        user.deleted_at = datetime.now(timezone.utc)
        user.is_active = False
        self.db.flush()
        return user

    def get_stats_by_role(self) -> List[dict]:
        """Get user counts grouped by role."""
        results = (
            self.db.query(
                User.role,
                func.count(User.id).label("total"),
                func.count(User.id).filter(
                    User.is_active == True
                ).label("active")
            )
            .filter(User.deleted_at.is_(None))
            .group_by(User.role)
            .all()
        )
        return [
            {"role": r.role, "total": r.total, "active": r.active}
            for r in results
        ]
```

---

## 4. 🔧 Unit of Work Pattern

### What & Why

text

```
Unit of Work = manages a group of operations
               as a single atomic transaction.

"All succeed together or all fail together."

Without UoW:
  You call UserRepository.create()
  Then PostRepository.create()
  If the second fails, the first already committed!
  Data is now inconsistent.

With UoW:
  You perform both operations
  Both are uncommitted (still in the transaction)
  Only at the end: UoW commits everything
  If anything fails: UoW rolls back everything
```

Python

```
# src/core/unit_of_work.py
from contextlib import contextmanager
from typing import Generator, Type
from sqlalchemy.orm import Session

from src.core.database import SessionLocal
from src.repositories.user_repo import UserRepository
from src.repositories.post_repo import PostRepository


class UnitOfWork:
    """
    Unit of Work pattern.

    Manages a database session and provides access
    to all repositories within the SAME session.

    All repositories share the same session, so
    changes from one are visible to others before commit.

    Usage:
        with UnitOfWork() as uow:
            user = uow.users.create(name="Alice", email="...")
            post = uow.posts.create(title="...", author_id=user.id)
            uow.commit()    # commit both atomically

        # Auto-rolled back if exception occurs
    """

    def __init__(self):
        self.session: Session = SessionLocal()

    def __enter__(self) -> "UnitOfWork":
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.rollback()
        self.close()

    # ─────────────────────────────────────────
    # REPOSITORIES (lazy-initialized)
    # ─────────────────────────────────────────
    @property
    def users(self) -> UserRepository:
        return UserRepository(self.session)

    @property
    def posts(self) -> PostRepository:
        return PostRepository(self.session)

    # ─────────────────────────────────────────
    # TRANSACTION MANAGEMENT
    # ─────────────────────────────────────────
    def commit(self) -> None:
        """Commit the current transaction."""
        self.session.commit()

    def rollback(self) -> None:
        """Roll back the current transaction."""
        self.session.rollback()

    def flush(self) -> None:
        """Flush changes without committing."""
        self.session.flush()

    def refresh(self, instance) -> None:
        """Refresh an instance from the database."""
        self.session.refresh(instance)

    def close(self) -> None:
        """Close the session."""
        self.session.close()


@contextmanager
def unit_of_work() -> Generator[UnitOfWork, None, None]:
    """
    Context manager for UnitOfWork.
    Commits on success, rolls back on failure.

    Usage:
        with unit_of_work() as uow:
            uow.users.create(...)
            # auto-commits at end
    """
    uow = UnitOfWork()
    try:
        yield uow
        uow.commit()
    except Exception:
        uow.rollback()
        raise
    finally:
        uow.close()


# Usage examples:
def example_usage():
    # Example 1: Basic usage
    with unit_of_work() as uow:
        user = uow.users.create(
            name="Alice",
            email="alice@test.com",
            password_hash="hashed"
        )
        post = uow.posts.create(
            title="My First Post",
            content="Content here",
            author_id=user.id     # user.id available because of flush in create
        )
        # Both committed together

    # Example 2: Handle failure
    try:
        with unit_of_work() as uow:
            user = uow.users.create(email="duplicate@test.com", ...)
            # If this fails (e.g., unique constraint), both roll back
            another = uow.users.create(email="duplicate@test.com", ...)
    except Exception as e:
        print(f"Both operations rolled back: {e}")

    # Example 3: Explicit control
    with UnitOfWork() as uow:
        try:
            user = uow.users.create(...)
            uow.commit()
        except Exception:
            uow.rollback()
            raise
```

---

## 5. 🎯 Service Layer Pattern

### What & Why

text

```
Service layer = where business logic lives.

Business logic examples:
  "User can't register with an email that's banned"
  "Posts can only be published by verified users"
  "When an order is placed, deduct from inventory"
  "New users get 30 days free trial"

Services:
  ✅ Don't know about HTTP (no request/response)
  ✅ Don't know about database details
  ✅ Only know about business rules
  ✅ Easily testable
  ✅ Reusable (called from API, background jobs, CLI)
```

Python

```
# src/services/user_service.py
from typing import Optional, Dict, Any, List
from fastapi import HTTPException, status

from src.core.unit_of_work import UnitOfWork
from src.core.security import hash_password, verify_password
from src.core.exceptions import (
    NotFoundException,
    ConflictException,
    ForbiddenException,
    ValidationException
)
from src.models.user import User
from src.schemas.user import UserCreate, UserUpdate


class UserService:
    """
    Business logic for user operations.

    This class knows NOTHING about:
      - HTTP requests or responses
      - Database details (SQL, ORM syntax)

    This class DOES know about:
      - Business rules (who can do what)
      - What makes a valid user
      - What steps are needed for each operation
    """

    BANNED_EMAIL_DOMAINS = {"tempmail.com", "throwaway.com", "mailinator.com"}
    RESERVED_NAMES = {"admin", "root", "system", "support"}

    def __init__(self, uow: UnitOfWork):
        self.uow = uow

    # ─────────────────────────────────────────
    # REGISTRATION
    # ─────────────────────────────────────────
    def register_user(self, data: UserCreate) -> User:
        """
        Register a new user.

        Business rules:
          1. Email must not be from a banned domain
          2. Email must be unique
          3. Name must not be a reserved word
          4. Password strength validated by schema
        """
        email = data.email.lower().strip()
        domain = email.split("@")[-1]

        # Rule 1: Ban disposable email domains
        if domain in self.BANNED_EMAIL_DOMAINS:
            raise ValidationException(
                field="email",
                message=f"Email domain '{domain}' is not allowed"
            )

        # Rule 2: Email uniqueness
        if self.uow.users.email_taken(email):
            raise ConflictException(
                resource="User",
                field="email",
                value=email
            )

        # Rule 3: Reserved names
        if data.name.lower().strip() in self.RESERVED_NAMES:
            raise ValidationException(
                field="name",
                message=f"'{data.name}' is a reserved name"
            )

        # Create user
        user = self.uow.users.create(
            name=data.name.strip().title(),
            email=email,
            password_hash=hash_password(data.password),
            role="user"
        )

        return user

    # ─────────────────────────────────────────
    # RETRIEVAL
    # ─────────────────────────────────────────
    def get_user(self, user_id: int) -> User:
        """Get user or raise NotFoundException."""
        user = self.uow.users.get_active_by_id(user_id)
        if not user:
            raise NotFoundException(resource="User", id=user_id)
        return user

    def get_users(
        self,
        page: int = 1,
        page_size: int = 20,
        search: Optional[str] = None,
        role: Optional[str] = None,
        is_active: Optional[bool] = None
    ) -> Dict[str, Any]:
        """Get paginated list of users."""
        skip = (page - 1) * page_size
        users, total = self.uow.users.search(
            query=search or "",
            role=role,
            is_active=is_active,
            skip=skip,
            limit=page_size
        )
        return {
            "items": users,
            "total": total,
            "page": page,
            "page_size": page_size,
            "total_pages": (total + page_size - 1) // page_size,
            "has_next": page * page_size < total,
            "has_prev": page > 1,
        }

    # ─────────────────────────────────────────
    # UPDATES
    # ─────────────────────────────────────────
    def update_user(
        self,
        user_id: int,
        data: UserUpdate,
        requesting_user: User
    ) -> User:
        """
        Update a user profile.

        Business rules:
          1. Users can only update their own profile
          2. Admins can update any profile
          3. Email change requires uniqueness check
        """
        user = self.get_user(user_id)

        # Rule 1 & 2: Authorization
        if user.id != requesting_user.id and not requesting_user.is_admin:
            raise ForbiddenException(
                action="update another user's profile"
            )

        updates = data.model_dump(exclude_unset=True)

        # Rule 3: Email uniqueness check
        if "email" in updates:
            new_email = updates["email"].lower()
            if self.uow.users.email_taken(new_email, exclude_id=user_id):
                raise ConflictException("User", "email", new_email)
            updates["email"] = new_email

        return self.uow.users.update(user, **updates)

    def change_password(
        self,
        user_id: int,
        current_password: str,
        new_password: str,
        requesting_user: User
    ) -> None:
        """
        Change user password.

        Business rules:
          4. Must be own account (or admin)
          5. Must provide correct current password
          6. New password must differ from current
        """
        user = self.get_user(user_id)

        if user.id != requesting_user.id and not requesting_user.is_admin:
            raise ForbiddenException("change another user's password")

        if not verify_password(current_password, user.password_hash):
            raise ValidationException(
                field="current_password",
                message="Current password is incorrect"
            )

        if current_password == new_password:
            raise ValidationException(
                field="new_password",
                message="New password must differ from current"
            )

        self.uow.users.update(
            user,
            password_hash=hash_password(new_password)
        )

    def promote_to_admin(
        self,
        user_id: int,
        requesting_user: User
    ) -> User:
        """
        Promote a user to admin.

        Business rules:
          7. Only admins can promote
          8. Can't promote already-admin users
        """
        if not requesting_user.is_admin:
            raise ForbiddenException("promote users to admin")

        user = self.get_user(user_id)

        if user.is_admin:
            raise ValidationException(
                field="role",
                message="User is already an admin"
            )

        return self.uow.users.update(user, role="admin")

    # ─────────────────────────────────────────
    # DELETION
    # ─────────────────────────────────────────
    def delete_user(self, user_id: int, requesting_user: User) -> None:
        """
        Soft delete a user.

        Business rules:
          9. Users can delete their own account
          10. Admins can delete any account
          11. Cannot delete the last admin
        """
        user = self.get_user(user_id)

        if user.id != requesting_user.id and not requesting_user.is_admin:
            raise ForbiddenException("delete another user's account")

        # Protect last admin
        if user.is_admin:
            admin_count = self.uow.users.count(role="admin", is_active=True)
            if admin_count <= 1:
                raise ValidationException(
                    field="user_id",
                    message="Cannot delete the last admin account"
                )

        self.uow.users.soft_delete(user)
```

---

## 6. ⚠️ Custom Exceptions

Python

```
# src/core/exceptions.py
from typing import Optional, List, Any


class AppException(Exception):
    """Base exception for all application errors."""

    def __init__(
        self,
        message: str,
        status_code: int = 500,
        error_code: Optional[str] = None
    ):
        super().__init__(message)
        self.message = message
        self.status_code = status_code
        self.error_code = error_code or self.__class__.__name__.upper()

    def to_dict(self) -> dict:
        return {
            "success": False,
            "error_code": self.error_code,
            "message": self.message
        }


class NotFoundException(AppException):
    """Resource not found."""
    def __init__(self, resource: str, id: Any = None):
        msg = f"{resource} not found"
        if id is not None:
            msg = f"{resource} with id '{id}' not found"
        super().__init__(msg, status_code=404, error_code="NOT_FOUND")
        self.resource = resource
        self.resource_id = id


class ConflictException(AppException):
    """Resource already exists / duplicate."""
    def __init__(self, resource: str, field: str, value: str):
        super().__init__(
            f"{resource} with {field}='{value}' already exists",
            status_code=409,
            error_code="CONFLICT"
        )
        self.resource = resource
        self.field = field
        self.value = value


class ValidationException(AppException):
    """Business rule validation failed."""
    def __init__(self, message: str, field: Optional[str] = None):
        super().__init__(message, status_code=422, error_code="VALIDATION_ERROR")
        self.field = field

    def to_dict(self) -> dict:
        d = super().to_dict()
        if self.field:
            d["field"] = self.field
        return d


class ForbiddenException(AppException):
    """User lacks permission."""
    def __init__(self, action: str = "perform this action"):
        super().__init__(
            f"You are not allowed to {action}",
            status_code=403,
            error_code="FORBIDDEN"
        )


class AuthenticationException(AppException):
    """Authentication failed."""
    def __init__(self, message: str = "Authentication required"):
        super().__init__(
            message,
            status_code=401,
            error_code="AUTHENTICATION_REQUIRED"
        )


class RateLimitException(AppException):
    """Rate limit exceeded."""
    def __init__(self, retry_after: int = 60):
        super().__init__(
            f"Rate limit exceeded. Retry after {retry_after} seconds",
            status_code=429,
            error_code="RATE_LIMIT_EXCEEDED"
        )
        self.retry_after = retry_after


class ExternalServiceException(AppException):
    """External service (payment, email, etc.) failed."""
    def __init__(self, service: str, message: str = "Service unavailable"):
        super().__init__(
            f"{service}: {message}",
            status_code=503,
            error_code="EXTERNAL_SERVICE_ERROR"
        )
        self.service = service


# FastAPI exception handlers
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError


def register_exception_handlers(app: FastAPI) -> None:
    """Register all exception handlers with FastAPI."""

    @app.exception_handler(AppException)
    async def app_exception_handler(
        request: Request,
        exc: AppException
    ) -> JSONResponse:
        return JSONResponse(
            status_code=exc.status_code,
            content=exc.to_dict()
        )

    @app.exception_handler(RequestValidationError)
    async def validation_exception_handler(
        request: Request,
        exc: RequestValidationError
    ) -> JSONResponse:
        details = []
        for error in exc.errors():
            field_parts = [str(loc) for loc in error["loc"] if loc != "body"]
            details.append({
                "field": ".".join(field_parts) if field_parts else None,
                "message": error["msg"],
                "type": error["type"]
            })
        return JSONResponse(
            status_code=422,
            content={
                "success": False,
                "error_code": "VALIDATION_ERROR",
                "message": "Request validation failed",
                "details": details
            }
        )

    @app.exception_handler(Exception)
    async def generic_exception_handler(
        request: Request,
        exc: Exception
    ) -> JSONResponse:
        import logging
        logging.getLogger(__name__).exception("Unhandled exception")
        return JSONResponse(
            status_code=500,
            content={
                "success": False,
                "error_code": "INTERNAL_SERVER_ERROR",
                "message": "An unexpected error occurred"
            }
        )
```

---

## 7. 🏭 Factory Pattern

Python

```
# src/core/factory.py
"""
Factory pattern: centralize object creation.

When creating an object requires:
  - Complex setup
  - Configuration
  - Dependencies

A factory handles all of that, giving you
a clean object with everything ready.
"""
from typing import Optional
from sqlalchemy.orm import Session

from src.repositories.user_repo import UserRepository
from src.repositories.post_repo import PostRepository
from src.services.user_service import UserService
from src.services.post_service import PostService
from src.services.auth_service import AuthService
from src.core.unit_of_work import UnitOfWork
from src.infrastructure.email import EmailService
from src.infrastructure.cache import CacheService
from src.infrastructure.storage import StorageService


class ServiceFactory:
    """
    Factory for creating services with their dependencies.

    Benefits:
      ✅ One place to configure service dependencies
      ✅ Easy to swap implementations (e.g., mock for testing)
      ✅ Services don't manage their own dependencies
      ✅ Clean dependency injection
    """

    def __init__(
        self,
        uow: UnitOfWork,
        email_service: Optional[EmailService] = None,
        cache_service: Optional[CacheService] = None,
        storage_service: Optional[StorageService] = None,
    ):
        self.uow = uow
        self._email = email_service or EmailService()
        self._cache = cache_service or CacheService()
        self._storage = storage_service or StorageService()

    def user_service(self) -> UserService:
        return UserService(
            uow=self.uow,
            email_service=self._email
        )

    def post_service(self) -> PostService:
        return PostService(
            uow=self.uow,
            cache_service=self._cache,
            storage_service=self._storage
        )

    def auth_service(self) -> AuthService:
        return AuthService(
            uow=self.uow,
            cache_service=self._cache
        )


# ─────────────────────────────────────────
# Alternative: simpler function factories
# ─────────────────────────────────────────
def make_user_service(db: Session) -> UserService:
    """Create UserService with all its dependencies."""
    uow = UnitOfWork(db)
    return UserService(uow=uow)


def make_auth_service(db: Session) -> AuthService:
    """Create AuthService with all its dependencies."""
    uow = UnitOfWork(db)
    return AuthService(uow=uow)


# ─────────────────────────────────────────
# FastAPI Dependency Integration
# ─────────────────────────────────────────
from fastapi import Depends
from src.core.database import get_db


def get_user_service(db: Session = Depends(get_db)) -> UserService:
    """FastAPI dependency that creates UserService."""
    with UnitOfWork(db) as uow:
        yield UserService(uow)


def get_auth_service(db: Session = Depends(get_db)) -> AuthService:
    """FastAPI dependency that creates AuthService."""
    with UnitOfWork(db) as uow:
        yield AuthService(uow)


# Usage in router:
# @router.post("/users")
# def create_user(
#     data: UserCreate,
#     service: UserService = Depends(get_user_service)
# ):
#     return service.register_user(data)
```

---

## 8. 💉 Dependency Injection

Python

```
# src/core/deps.py
"""
Dependency Injection in FastAPI.

DI = provide dependencies to a function from the outside,
     rather than the function creating them itself.

Without DI:
  def create_user(data: UserCreate):
      db = get_database_connection()  # tightly coupled
      service = UserService(db)       # hard to test
      return service.create(data)

With DI:
  def create_user(
      data: UserCreate,
      service: UserService = Depends(get_user_service)  # injected!
  ):
      return service.create(data)

Benefits:
  ✅ Easy to test (inject mock service)
  ✅ Loose coupling
  ✅ Single responsibility
  ✅ Centralized dependency management
"""
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.orm import Session
from typing import Annotated, Optional

from src.core.database import get_db
from src.core.security import decode_token
from src.core.unit_of_work import UnitOfWork
from src.models.user import User
from src.repositories.user_repo import UserRepository

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")
oauth2_scheme_optional = OAuth2PasswordBearer(
    tokenUrl="/api/v1/auth/login",
    auto_error=False
)


# ─────────────────────────────────────────
# DATABASE SESSION
# ─────────────────────────────────────────
def get_uow(db: Session = Depends(get_db)) -> UnitOfWork:
    """Provide a Unit of Work."""
    return UnitOfWork(db)


# ─────────────────────────────────────────
# AUTHENTICATION
# ─────────────────────────────────────────
def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    """Get authenticated user from JWT token."""
    from src.core.exceptions import AuthenticationException

    try:
        payload = decode_token(token)
        user_id = payload.get("sub")
        token_type = payload.get("type", "access")

        if not user_id or token_type != "access":
            raise AuthenticationException("Invalid token")

    except Exception:
        raise AuthenticationException("Invalid or expired token")

    repo = UserRepository(db)
    user = repo.get_active_by_id(int(user_id))

    if not user:
        raise AuthenticationException("User not found")

    return user


def get_current_active_user(
    user: User = Depends(get_current_user)
) -> User:
    """Ensure user is active."""
    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Account is deactivated"
        )
    return user


def get_current_admin(
    user: User = Depends(get_current_active_user)
) -> User:
    """Ensure user is an admin."""
    if not user.is_admin:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Admin access required"
        )
    return user


def get_optional_user(
    token: Optional[str] = Depends(oauth2_scheme_optional),
    db: Session = Depends(get_db)
) -> Optional[User]:
    """Get user if authenticated, None otherwise."""
    if not token:
        return None
    try:
        payload = decode_token(token)
        user_id = payload.get("sub")
        if user_id:
            repo = UserRepository(db)
            return repo.get_active_by_id(int(user_id))
    except Exception:
        pass
    return None


def require_role(*allowed_roles: str):
    """
    Dependency factory: require specific role(s).

    Usage:
        @router.delete("/{id}", dependencies=[Depends(require_role("admin"))])
        def delete_something():
            ...
    """
    def role_checker(user: User = Depends(get_current_active_user)) -> User:
        if user.role not in allowed_roles:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Required role: {' or '.join(allowed_roles)}"
            )
        return user
    return role_checker


# ─────────────────────────────────────────
# TYPE ALIASES (cleaner route signatures)
# ─────────────────────────────────────────
CurrentUser = Annotated[User, Depends(get_current_active_user)]
AdminUser = Annotated[User, Depends(get_current_admin)]
OptionalUser = Annotated[Optional[User], Depends(get_optional_user)]
DBSession = Annotated[Session, Depends(get_db)]
UOW = Annotated[UnitOfWork, Depends(get_uow)]

# ─────────────────────────────────────────
# SERVICE DEPENDENCIES
# ─────────────────────────────────────────
from src.services.user_service import UserService
from src.services.post_service import PostService
from src.services.auth_service import AuthService


def get_user_service(uow: UnitOfWork = Depends(get_uow)) -> UserService:
    return UserService(uow)


def get_post_service(uow: UnitOfWork = Depends(get_uow)) -> PostService:
    return PostService(uow)


def get_auth_service(uow: UnitOfWork = Depends(get_uow)) -> AuthService:
    return AuthService(uow)


UserSvc = Annotated[UserService, Depends(get_user_service)]
PostSvc = Annotated[PostService, Depends(get_post_service)]
AuthSvc = Annotated[AuthService, Depends(get_auth_service)]
```

---

## 9. 📝 Logging

Python

```
# src/core/logging.py
"""
Structured logging — the production standard.

Regular logging:
  "User Alice logged in at 14:32:01"
  Hard to parse, hard to query, hard to aggregate.

Structured logging (JSON):
  {
    "timestamp": "2024-01-15T14:32:01Z",
    "level": "INFO",
    "event": "user_login",
    "user_id": 42,
    "email": "alice@test.com",
    "ip": "192.168.1.1",
    "duration_ms": 45
  }
  
  Now you can:
  - Search by user_id: all logs for user 42
  - Alert when error rate > 1%
  - Dashboard: logins per hour
  - Correlate with request_id across services
"""
import logging
import logging.config
import sys
from datetime import datetime, timezone
from typing import Any, Dict, Optional
import json
import traceback


class JSONFormatter(logging.Formatter):
    """Format log records as JSON for structured logging."""

    def format(self, record: logging.LogRecord) -> str:
        log_data = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
        }

        # Add extra fields (from extra= parameter)
        for key, value in record.__dict__.items():
            if key not in (
                "name", "msg", "args", "levelname", "levelno",
                "pathname", "filename", "module", "exc_info",
                "exc_text", "stack_info", "lineno", "funcName",
                "created", "msecs", "relativeCreated", "thread",
                "threadName", "processName", "process", "message",
                "taskName"
            ):
                log_data[key] = value

        # Add exception info
        if record.exc_info:
            log_data["exception"] = {
                "type": record.exc_info[0].__name__,
                "message": str(record.exc_info[1]),
                "traceback": traceback.format_exception(*record.exc_info)
            }

        return json.dumps(log_data, default=str)


def setup_logging(
    level: str = "INFO",
    json_format: bool = True
) -> None:
    """Configure application logging."""

    formatter = JSONFormatter() if json_format else logging.Formatter(
        "%(asctime)s [%(levelname)s] %(name)s: %(message)s"
    )

    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(formatter)

    # Configure root logger
    root_logger = logging.getLogger()
    root_logger.setLevel(getattr(logging, level.upper()))
    root_logger.handlers.clear()
    root_logger.addHandler(handler)

    # Silence noisy loggers
    logging.getLogger("sqlalchemy.engine").setLevel(logging.WARNING)
    logging.getLogger("uvicorn.access").setLevel(logging.WARNING)


# ─────────────────────────────────────────
# LOGGER CLASS WITH CONTEXT
# ─────────────────────────────────────────
class AppLogger:
    """
    Structured logger with automatic context fields.

    Usage:
        logger = AppLogger("user_service")

        logger.info("User registered",
            user_id=42,
            email="alice@test.com"
        )

        logger.error("Payment failed",
            user_id=42,
            amount=99.99,
            error="Card declined"
        )
    """

    def __init__(self, name: str):
        self._logger = logging.getLogger(name)
        self._context: Dict[str, Any] = {}

    def bind(self, **context) -> "AppLogger":
        """Add persistent context fields."""
        new = AppLogger(self._logger.name)
        new._context = {**self._context, **context}
        return new

    def _log(self, level: str, message: str, **extra) -> None:
        combined = {**self._context, **extra}
        log_fn = getattr(self._logger, level)
        log_fn(message, extra=combined)

    def debug(self, message: str, **extra) -> None:
        self._log("debug", message, **extra)

    def info(self, message: str, **extra) -> None:
        self._log("info", message, **extra)

    def warning(self, message: str, **extra) -> None:
        self._log("warning", message, **extra)

    def error(self, message: str, **extra) -> None:
        self._log("error", message, **extra)

    def exception(self, message: str, **extra) -> None:
        combined = {**self._context, **extra}
        self._logger.exception(message, extra=combined)


# ─────────────────────────────────────────
# USAGE IN SERVICES
# ─────────────────────────────────────────
logger = AppLogger("app")

class UserService:
    def __init__(self, uow):
        self.uow = uow
        self.logger = AppLogger("user_service")

    def register_user(self, data):
        log = self.logger.bind(email=data.email)
        log.info("Registration attempt started")

        try:
            user = self.uow.users.create(...)
            log.info(
                "User registered successfully",
                user_id=user.id,
                role=user.role
            )
            return user
        except Exception as e:
            log.exception(
                "Registration failed",
                error=str(e)
            )
            raise


# ─────────────────────────────────────────
# LOGURU (alternative — simpler, popular)
# ─────────────────────────────────────────
# pip install loguru

from loguru import logger as loguru_logger
import sys

def setup_loguru(level: str = "INFO", json: bool = True) -> None:
    """Configure loguru logger."""
    loguru_logger.remove()  # remove default handler

    if json:
        loguru_logger.add(
            sys.stdout,
            level=level,
            format="{message}",
            serialize=True  # output as JSON
        )
    else:
        loguru_logger.add(
            sys.stdout,
            level=level,
            format=(
                "<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
                "<level>{level: <8}</level> | "
                "<cyan>{name}</cyan>:<cyan>{line}</cyan> | "
                "<level>{message}</level>"
            ),
            colorize=True
        )

    # Also log to file with rotation
    loguru_logger.add(
        "logs/app.log",
        rotation="100 MB",
        retention="30 days",
        compression="gz",
        level=level,
        serialize=True
    )

# Usage with loguru:
# from loguru import logger
# logger.info("User created", user_id=42, email="alice@test.com")
# logger.error("Failed", error=str(exc))
```

---

## 10. ⚙️ Configuration Management

Python

```
# src/core/config.py
"""
Configuration management.

Best practices:
  1. All config from environment variables
  2. Validate at startup (fail fast)
  3. Type-safe (Pydantic converts types)
  4. Organized by concern
  5. Different settings per environment
"""
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field, validator, AnyHttpUrl
from typing import List, Optional
from functools import lru_cache
import secrets


class DatabaseSettings(BaseSettings):
    """Database configuration."""
    URL: str = Field(..., description="PostgreSQL connection URL")
    POOL_SIZE: int = Field(5, ge=1, le=50)
    MAX_OVERFLOW: int = Field(10, ge=0, le=100)
    POOL_TIMEOUT: int = Field(30, ge=5)
    POOL_RECYCLE: int = Field(3600)
    ECHO: bool = Field(False, description="Log all SQL statements")

    model_config = SettingsConfigDict(env_prefix="DB_")


class JWTSettings(BaseSettings):
    """JWT configuration."""
    SECRET_KEY: str = Field(..., min_length=32)
    ALGORITHM: str = Field("HS256")
    ACCESS_TOKEN_EXPIRE_MINUTES: int = Field(30, ge=1)
    REFRESH_TOKEN_EXPIRE_DAYS: int = Field(7, ge=1)

    model_config = SettingsConfigDict(env_prefix="JWT_")


class CORSSettings(BaseSettings):
    """CORS configuration."""
    ALLOWED_ORIGINS: List[str] = Field(
        default=["http://localhost:3000"],
        description="Allowed origins (comma-separated in env)"
    )
    ALLOW_CREDENTIALS: bool = True
    ALLOW_METHODS: List[str] = ["GET", "POST", "PUT", "PATCH", "DELETE"]
    ALLOW_HEADERS: List[str] = ["Authorization", "Content-Type"]

    model_config = SettingsConfigDict(env_prefix="CORS_")


class RedisSettings(BaseSettings):
    """Redis configuration."""
    URL: str = Field("redis://localhost:6379/0")
    MAX_CONNECTIONS: int = Field(10)

    model_config = SettingsConfigDict(env_prefix="REDIS_")


class Settings(BaseSettings):
    """Main application settings."""

    # ─────────────────────────────────────────
    # Application
    # ─────────────────────────────────────────
    APP_NAME: str = "MyBackendApp"
    APP_VERSION: str = "1.0.0"
    ENVIRONMENT: str = Field("development")
    DEBUG: bool = Field(False)
    API_PREFIX: str = "/api/v1"
    SECRET_KEY: str = Field(default_factory=lambda: secrets.token_hex(32))

    # ─────────────────────────────────────────
    # Database
    # ─────────────────────────────────────────
    DATABASE_URL: str = Field(
        "postgresql://myapp_user:password@localhost:5432/myapp"
    )
    DATABASE_POOL_SIZE: int = Field(5, ge=1)
    DATABASE_MAX_OVERFLOW: int = Field(10, ge=0)
    DATABASE_ECHO: bool = Field(False)

    # ─────────────────────────────────────────
    # JWT
    # ─────────────────────────────────────────
    JWT_SECRET_KEY: str = Field(
        default_factory=lambda: secrets.token_hex(32)
    )
    JWT_ALGORITHM: str = "HS256"
    JWT_ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    JWT_REFRESH_TOKEN_EXPIRE_DAYS: int = 7

    # ─────────────────────────────────────────
    # Redis
    # ─────────────────────────────────────────
    REDIS_URL: str = "redis://localhost:6379/0"

    # ─────────────────────────────────────────
    # CORS
    # ─────────────────────────────────────────
    ALLOWED_ORIGINS: List[str] = ["http://localhost:3000"]

    # ─────────────────────────────────────────
    # Files
    # ─────────────────────────────────────────
    UPLOAD_DIR: str = "uploads"
    MAX_UPLOAD_SIZE_MB: int = Field(10, ge=1, le=100)

    # ─────────────────────────────────────────
    # Email
    # ─────────────────────────────────────────
    EMAIL_FROM: str = "noreply@myapp.com"
    SMTP_HOST: Optional[str] = None
    SMTP_PORT: int = 587
    SMTP_USER: Optional[str] = None
    SMTP_PASSWORD: Optional[str] = None

    # ─────────────────────────────────────────
    # Rate Limiting
    # ─────────────────────────────────────────
    RATE_LIMIT_PER_MINUTE: int = 60
    RATE_LIMIT_BURST: int = 100

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=True,
        extra="ignore"
    )

    # ─────────────────────────────────────────
    # COMPUTED PROPERTIES
    # ─────────────────────────────────────────
    @property
    def is_production(self) -> bool:
        return self.ENVIRONMENT == "production"

    @property
    def is_development(self) -> bool:
        return self.ENVIRONMENT == "development"

    @property
    def is_testing(self) -> bool:
        return self.ENVIRONMENT == "testing"

    @property
    def max_upload_bytes(self) -> int:
        return self.MAX_UPLOAD_SIZE_MB * 1024 * 1024

    # ─────────────────────────────────────────
    # VALIDATION
    # ─────────────────────────────────────────
    def validate_for_production(self) -> None:
        """
        Extra validation for production.
        Called at startup to catch misconfigurations early.
        """
        if not self.is_production:
            return

        errors = []

        if len(self.JWT_SECRET_KEY) < 32:
            errors.append("JWT_SECRET_KEY must be at least 32 characters")

        if len(self.SECRET_KEY) < 32:
            errors.append("SECRET_KEY must be at least 32 characters")

        if self.DEBUG:
            errors.append("DEBUG must be False in production")

        if "localhost" in self.DATABASE_URL:
            errors.append("DATABASE_URL should not point to localhost in production")

        if errors:
            raise ValueError(
                f"Production configuration errors:\n" +
                "\n".join(f"  - {e}" for e in errors)
            )


@lru_cache()
def get_settings() -> Settings:
    """
    Get cached settings instance.
    @lru_cache ensures settings are only loaded once.
    """
    settings = Settings()
    settings.validate_for_production()
    return settings


settings = get_settings()


# ─────────────────────────────────────────
# ENVIRONMENT-SPECIFIC CONFIG
# ─────────────────────────────────────────
class DevelopmentSettings(Settings):
    """Development overrides."""
    ENVIRONMENT: str = "development"
    DEBUG: bool = True
    DATABASE_ECHO: bool = True  # show SQL in dev


class TestingSettings(Settings):
    """Testing overrides."""
    ENVIRONMENT: str = "testing"
    DEBUG: bool = True
    DATABASE_URL: str = "sqlite:///./test.db"


class ProductionSettings(Settings):
    """Production — strictest."""
    ENVIRONMENT: str = "production"
    DEBUG: bool = False
    DATABASE_ECHO: bool = False


def get_settings_for_env(env: str) -> Settings:
    """Get settings for a specific environment."""
    settings_map = {
        "development": DevelopmentSettings,
        "testing": TestingSettings,
        "production": ProductionSettings,
    }
    cls = settings_map.get(env, Settings)
    return cls()
```

---

## 11. 🎨 Design Patterns in Practice

### Strategy Pattern

Python

```
# src/infrastructure/notifications.py
"""
Strategy Pattern: define a family of algorithms,
encapsulate each, make them interchangeable.

Example: notification strategies
  EmailNotification, SMSNotification, PushNotification
  All implement the same interface.
  The service doesn't care which one is used.
"""
from abc import ABC, abstractmethod
from typing import Dict, Any
from dataclasses import dataclass


@dataclass
class NotificationPayload:
    recipient: str
    subject: str
    message: str
    metadata: Dict[str, Any] = None


class NotificationStrategy(ABC):
    """Interface that all notification strategies must implement."""

    @abstractmethod
    def send(self, payload: NotificationPayload) -> bool:
        """Send notification. Returns True if successful."""
        pass

    @abstractmethod
    def is_available(self) -> bool:
        """Check if this strategy is available/configured."""
        pass


class EmailNotificationStrategy(NotificationStrategy):
    """Send notifications via email."""

    def __init__(self, smtp_host: str, smtp_port: int, user: str, password: str):
        self.smtp_host = smtp_host
        self.smtp_port = smtp_port
        self.user = user
        self.password = password

    def send(self, payload: NotificationPayload) -> bool:
        print(f"[EMAIL] To: {payload.recipient}")
        print(f"[EMAIL] Subject: {payload.subject}")
        print(f"[EMAIL] Body: {payload.message}")
        # Real: use smtplib or SendGrid
        return True

    def is_available(self) -> bool:
        return bool(self.smtp_host and self.user)


class SMSNotificationStrategy(NotificationStrategy):
    """Send notifications via SMS."""

    def __init__(self, api_key: str, from_number: str):
        self.api_key = api_key
        self.from_number = from_number

    def send(self, payload: NotificationPayload) -> bool:
        print(f"[SMS] To: {payload.recipient}")
        print(f"[SMS] Message: {payload.message[:160]}")  # SMS limit
        # Real: use Twilio
        return True

    def is_available(self) -> bool:
        return bool(self.api_key and self.from_number)


class WebhookNotificationStrategy(NotificationStrategy):
    """Send notifications via webhook."""

    def __init__(self, webhook_url: str):
        self.webhook_url = webhook_url

    def send(self, payload: NotificationPayload) -> bool:
        import json
        print(f"[WEBHOOK] POST to: {self.webhook_url}")
        print(f"[WEBHOOK] Payload: {json.dumps({'message': payload.message})}")
        # Real: use httpx
        return True

    def is_available(self) -> bool:
        return bool(self.webhook_url)


class LogNotificationStrategy(NotificationStrategy):
    """Just log — useful for testing."""

    def send(self, payload: NotificationPayload) -> bool:
        print(f"[LOG NOTIFICATION] recipient={payload.recipient} message={payload.message}")
        return True

    def is_available(self) -> bool:
        return True


class NotificationService:
    """
    Uses Strategy pattern to send notifications.
    The caller doesn't care which strategy is used.
    """

    def __init__(self, *strategies: NotificationStrategy):
        self.strategies = [s for s in strategies if s.is_available()]
        if not self.strategies:
            # Always fall back to logging
            self.strategies = [LogNotificationStrategy()]

    def notify(self, payload: NotificationPayload) -> Dict[str, bool]:
        """Send via all available strategies."""
        results = {}
        for strategy in self.strategies:
            try:
                results[strategy.__class__.__name__] = strategy.send(payload)
            except Exception as e:
                print(f"Notification failed via {strategy.__class__.__name__}: {e}")
                results[strategy.__class__.__name__] = False
        return results

    def notify_single(
        self,
        payload: NotificationPayload,
        strategy_type: type
    ) -> bool:
        """Send via a specific strategy."""
        for strategy in self.strategies:
            if isinstance(strategy, strategy_type):
                return strategy.send(payload)
        return False


# Usage
notification_service = NotificationService(
    EmailNotificationStrategy("smtp.gmail.com", 587, "user@gmail.com", "pass"),
    SMSNotificationStrategy("twilio_key", "+1-555-0123"),
)

notification_service.notify(NotificationPayload(
    recipient="alice@test.com",
    subject="Welcome!",
    message="Welcome to our platform!"
))
```

### Observer Pattern

Python

```
# src/core/events.py
"""
Observer Pattern: when something happens, notify interested parties.
Also called: event-driven, pub/sub, signals.

When a user registers:
  → Send welcome email
  → Create their default settings
  → Log the event
  → Update analytics

All these "observers" react to the same "event"
without the UserService knowing about them.
"""
from typing import Callable, Dict, List, Any
from dataclasses import dataclass, field
from datetime import datetime, timezone
import asyncio
import logging

logger = logging.getLogger(__name__)


@dataclass
class Event:
    """Base event class."""
    name: str
    data: Dict[str, Any] = field(default_factory=dict)
    occurred_at: datetime = field(
        default_factory=lambda: datetime.now(timezone.utc)
    )


# Domain events
@dataclass
class UserRegisteredEvent(Event):
    name: str = "user.registered"

@dataclass
class UserDeletedEvent(Event):
    name: str = "user.deleted"

@dataclass
class PostPublishedEvent(Event):
    name: str = "post.published"

@dataclass
class PaymentCompletedEvent(Event):
    name: str = "payment.completed"


EventHandler = Callable[[Event], None]


class EventBus:
    """
    Simple synchronous event bus.
    Allows decoupled communication between components.
    """

    def __init__(self):
        self._handlers: Dict[str, List[EventHandler]] = {}

    def subscribe(self, event_name: str, handler: EventHandler) -> None:
        """Register a handler for an event."""
        if event_name not in self._handlers:
            self._handlers[event_name] = []
        self._handlers[event_name].append(handler)
        logger.debug(f"Subscribed {handler.__name__} to {event_name}")

    def unsubscribe(self, event_name: str, handler: EventHandler) -> None:
        """Remove a handler."""
        if event_name in self._handlers:
            self._handlers[event_name].remove(handler)

    def publish(self, event: Event) -> None:
        """Notify all handlers of an event."""
        handlers = self._handlers.get(event.name, [])
        logger.info(
            f"Publishing event: {event.name}",
            extra={"event": event.name, "data": event.data}
        )
        for handler in handlers:
            try:
                handler(event)
            except Exception as e:
                logger.exception(
                    f"Handler {handler.__name__} failed for {event.name}: {e}"
                )


# Global event bus
event_bus = EventBus()


# ─────────────────────────────────────────
# EVENT HANDLERS (observers)
# ─────────────────────────────────────────
def send_welcome_email_handler(event: Event) -> None:
    """Send welcome email when user registers."""
    email = event.data.get("email")
    name = event.data.get("name")
    print(f"[EMAIL] Sending welcome email to {name} at {email}")


def create_user_settings_handler(event: Event) -> None:
    """Create default settings for new user."""
    user_id = event.data.get("user_id")
    print(f"[SETTINGS] Creating default settings for user {user_id}")


def log_registration_handler(event: Event) -> None:
    """Log user registration for analytics."""
    logger.info("User registered", extra=event.data)


def invalidate_user_cache_handler(event: Event) -> None:
    """Invalidate user cache when user changes."""
    user_id = event.data.get("user_id")
    print(f"[CACHE] Invalidating cache for user {user_id}")


# Register handlers
event_bus.subscribe("user.registered", send_welcome_email_handler)
event_bus.subscribe("user.registered", create_user_settings_handler)
event_bus.subscribe("user.registered", log_registration_handler)
event_bus.subscribe("user.deleted", invalidate_user_cache_handler)


# ─────────────────────────────────────────
# USAGE IN SERVICE
# ─────────────────────────────────────────
class UserService:
    def __init__(self, uow, event_bus: EventBus):
        self.uow = uow
        self.event_bus = event_bus

    def register_user(self, data) -> "User":
        user = self.uow.users.create(...)

        # Publish event — don't know/care who handles it
        self.event_bus.publish(Event(
            name="user.registered",
            data={
                "user_id": user.id,
                "email": user.email,
                "name": user.name
            }
        ))

        return user
```

### Middleware Pattern

Python

```
# src/api/middleware.py
"""
Middleware = code that runs for EVERY request.
Layer around your application that adds behavior.
"""
from fastapi import FastAPI, Request, Response
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.responses import JSONResponse
import time
import uuid
import logging
from typing import Callable

logger = logging.getLogger(__name__)


class RequestContextMiddleware(BaseHTTPMiddleware):
    """
    Add context to every request:
    - Unique request ID
    - Timing information
    - Basic logging
    """

    async def dispatch(self, request: Request, call_next: Callable) -> Response:
        # Generate request ID
        request_id = str(uuid.uuid4())[:8]
        request.state.request_id = request_id
        request.state.start_time = time.perf_counter()

        logger.info(
            "Request started",
            extra={
                "request_id": request_id,
                "method": request.method,
                "path": request.url.path,
                "client_ip": self._get_client_ip(request)
            }
        )

        # Process request
        response = await call_next(request)

        # Calculate duration
        duration_ms = (time.perf_counter() - request.state.start_time) * 1000

        # Add headers
        response.headers["X-Request-ID"] = request_id
        response.headers["X-Response-Time"] = f"{duration_ms:.2f}ms"

        logger.info(
            "Request completed",
            extra={
                "request_id": request_id,
                "status_code": response.status_code,
                "duration_ms": round(duration_ms, 2)
            }
        )

        return response

    def _get_client_ip(self, request: Request) -> str:
        forwarded = request.headers.get("X-Forwarded-For")
        if forwarded:
            return forwarded.split(",")[0].strip()
        return request.client.host if request.client else "unknown"


class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    """Add security headers to every response."""

    async def dispatch(self, request: Request, call_next: Callable) -> Response:
        response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        if request.url.scheme == "https":
            response.headers["Strict-Transport-Security"] = (
                "max-age=31536000; includeSubDomains"
            )
        return response


class MaintenanceModeMiddleware(BaseHTTPMiddleware):
    """
    Return 503 when maintenance mode is enabled.
    Toggle via environment variable or feature flag.
    """

    def __init__(self, app, maintenance_mode: bool = False):
        super().__init__(app)
        self.maintenance_mode = maintenance_mode

    async def dispatch(self, request: Request, call_next: Callable) -> Response:
        # Allow health checks through
        if request.url.path == "/health":
            return await call_next(request)

        if self.maintenance_mode:
            return JSONResponse(
                status_code=503,
                content={
                    "success": False,
                    "error": "Service temporarily unavailable for maintenance",
                    "retry_after": 300
                },
                headers={"Retry-After": "300"}
            )

        return await call_next(request)


def register_middleware(app: FastAPI, settings) -> None:
    """Register all middleware in correct order."""

    # Order matters: first added = outermost (runs first)
    app.add_middleware(
        MaintenanceModeMiddleware,
        maintenance_mode=False
    )
    app.add_middleware(SecurityHeadersMiddleware)
    app.add_middleware(RequestContextMiddleware)

    # CORS (from fastapi)
    from fastapi.middleware.cors import CORSMiddleware
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.ALLOWED_ORIGINS,
        allow_credentials=True,
        allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"],
        allow_headers=["Authorization", "Content-Type", "X-Request-ID"],
        expose_headers=["X-Request-ID", "X-Response-Time"],
    )
```

---

## 12. 🎯 Complete Application Entry Point

Python

```
# src/main.py
"""
Application entry point.
Wires everything together.
"""
import logging
from contextlib import asynccontextmanager

from fastapi import FastAPI

from src.core.config import settings
from src.core.database import engine, Base
from src.core.logging import setup_logging
from src.core.exceptions import register_exception_handlers
from src.api.middleware import register_middleware
from src.api.v1.router import api_router

# Setup logging first
setup_logging(
    level="DEBUG" if settings.DEBUG else "INFO",
    json_format=settings.is_production
)
logger = logging.getLogger(__name__)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application startup and shutdown."""
    # ─────────────────────────────────────────
    # STARTUP
    # ─────────────────────────────────────────
    logger.info(
        "Application starting",
        extra={
            "app_name": settings.APP_NAME,
            "version": settings.APP_VERSION,
            "environment": settings.ENVIRONMENT,
        }
    )

    # Create database tables
    Base.metadata.create_all(bind=engine)
    logger.info("Database tables ready")

    yield  # App runs here

    # ─────────────────────────────────────────
    # SHUTDOWN
    # ─────────────────────────────────────────
    engine.dispose()
    logger.info("Application shutdown complete")


def create_application() -> FastAPI:
    """Create and configure the FastAPI application."""
    app = FastAPI(
        title=settings.APP_NAME,
        version=settings.APP_VERSION,
        description="Production-ready backend API",
        docs_url="/docs" if not settings.is_production else None,
        redoc_url="/redoc" if not settings.is_production else None,
        openapi_url="/openapi.json" if not settings.is_production else None,
        lifespan=lifespan,
    )

    # Register components in order
    register_middleware(app, settings)
    register_exception_handlers(app)
    app.include_router(api_router, prefix=settings.API_PREFIX)

    @app.get("/health", tags=["Health"])
    def health_check():
        return {
            "status": "healthy",
            "version": settings.APP_VERSION,
            "environment": settings.ENVIRONMENT
        }

    return app


app = create_application()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "src.main:app",
        host="0.0.0.0",
        port=8000,
        reload=settings.DEBUG,
        log_level="debug" if settings.DEBUG else "info"
    )
```

---

## 13. 📊 Pattern Summary & Decision Guide

Python

```
"""
WHEN TO USE EACH PATTERN:

REPOSITORY PATTERN:
  ✅ Use when: database logic gets complex
  ✅ Use when: you want to test business logic without DB
  ✅ Use when: team is large, need clear boundaries
  ❌ Skip when: tiny app, just CRUD, no complex queries

UNIT OF WORK:
  ✅ Use when: multiple repositories in one transaction
  ✅ Use when: you need atomicity across multiple entities
  ❌ Skip when: simple app, single entity operations

SERVICE LAYER:
  ✅ Use when: business logic is complex
  ✅ Use when: same logic called from multiple places
  ✅ Use when: you want to test business rules
  ❌ Skip when: pure CRUD, no business logic

FACTORY PATTERN:
  ✅ Use when: complex object creation
  ✅ Use when: multiple implementations
  ✅ Use when: you want to inject mocks for testing
  ❌ Skip when: simple, direct creation is fine

STRATEGY PATTERN:
  ✅ Use when: multiple ways to do something
  ✅ Use when: behavior changes at runtime
  Example: payment methods, notification channels

OBSERVER/EVENTS:
  ✅ Use when: one action triggers many reactions
  ✅ Use when: decoupling is needed
  ✅ Use when: reactions might change
  Example: user registration → email, settings, analytics

MIDDLEWARE PATTERN:
  ✅ Use when: behavior needed for ALL requests
  Example: logging, auth check, rate limiting, headers

GENERAL RULE:
  Start simple.
  Add patterns when you feel the pain they solve.
  Don't over-engineer from day one.
"""
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│              PROJECT STRUCTURE & PATTERNS                      │
│                                                                │
│  ARCHITECTURE LAYERS:                                          │
│  API Layer      → HTTP in/out (no business logic)            │
│  Service Layer  → business rules (no HTTP, no DB)            │
│  Repository     → data access (no business rules)            │
│  Database       → actual data storage                         │
│                                                                │
│  KEY PATTERNS:                                                 │
│  Repository   → centralize data access per entity            │
│  Unit of Work → atomic multi-repository transactions         │
│  Service      → business logic, reusable, testable           │
│  Factory      → centralize complex object creation           │
│  Strategy     → interchangeable algorithms                    │
│  Observer     → decouple event producers from consumers      │
│  Middleware   → cross-cutting concerns for all requests      │
│                                                                │
│  CONFIGURATION:                                                │
│  Environment variables → never hardcode                       │
│  Pydantic Settings   → type-safe, validated                  │
│  @lru_cache         → load once, reuse                       │
│  Environment classes → dev/test/prod overrides               │
│                                                                │
│  LOGGING:                                                      │
│  Structured JSON   → machine-readable                        │
│  Context fields    → user_id, request_id, duration           │
│  Log levels        → DEBUG/INFO/WARNING/ERROR                 │
│  Never log secrets → passwords, tokens, PII                  │
│                                                                │
│  EXCEPTIONS:                                                   │
│  Custom hierarchy  → NotFoundException, ConflictException     │
│  Raise early       → fail fast with clear messages           │
│  Global handlers   → consistent response format              │
│  Never swallow     → always log unexpected exceptions        │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What are the four layers in clean architecture?
    What does each layer know/not know about?
2.  What is the Repository pattern?
    What problem does it solve?
3.  What is the Unit of Work pattern?
    Give an example where it's necessary.
4.  What belongs in the Service layer?
    What does NOT belong there?
5.  What is the Factory pattern?
    When would you use it for services?
6.  What is the Strategy pattern?
    Give a real backend example.
7.  What is the Observer pattern?
    How does it decouple code?
8.  What is the difference between flush() and commit()?
9.  Why should configuration come from environment variables?
10. What is structured logging and why is it better?
11. What should custom exceptions include?
12. Why are type aliases useful in FastAPI dependencies?
13. What is the Middleware pattern used for?
14. What does "fail fast" mean in configuration?
```

---

## 🛠️ Practice Exercises

Python

```
# Exercise 1: Complete Repository
# Implement a PostRepository extending BaseRepository with:
# - get_published(skip, limit) → only published, not deleted
# - get_by_author(author_id, status=None) → author's posts
# - get_trending(limit=10) → most viewed in last 7 days
# - search(query, tag=None, skip=0, limit=20) → search title/content
# - increment_views(post_id) → atomic increment
# - slug_exists(slug, exclude_id=None) → check uniqueness

# Exercise 2: PostService with Business Rules
# Implement PostService with these rules:
# - Only verified users can publish posts (not just save drafts)
# - Posts can't be published if title < 5 words
# - Authors can't delete published posts with > 100 views
#   (must archive first, then admin can delete)
# - Draft → Published → Archived is the only valid flow
# - Archived posts can only be restored by admins
# Publish event: PostPublishedEvent when post goes live

# Exercise 3: Event System
# Implement a complete event system:
# Events: UserRegistered, UserDeleted, PostPublished, PostDeleted
# Handlers:
#   UserRegistered → create default settings, send welcome email (log)
#   UserDeleted → soft-delete all their posts, send farewell email (log)
#   PostPublished → notify followers (log), update search index (log)
#   PostDeleted → invalidate cache (log)
# Test: register a user and verify all handlers ran

# Exercise 4: Strategy Pattern
# Build a StorageStrategy for file uploads:
# Strategies: LocalStorage, S3Storage, MinIOStorage
# All implement: save(file, filename) → url
#                delete(filename) → bool
#                exists(filename) → bool
#                get_url(filename) → str
# Build StorageService that uses the strategy
# Config determines which strategy is used

# Exercise 5: Refactor
# Take this spaghetti code:
#
# @app.post("/transfer")
# def transfer_funds(data: dict, db = Depends(get_db)):
#     from_user = db.execute(
#         "SELECT * FROM users WHERE id = ?", data["from_id"]
#     ).fetchone()
#     if not from_user:
#         return {"error": "not found"}, 404
#     to_user = db.execute(
#         "SELECT * FROM users WHERE id = ?", data["to_id"]
#     ).fetchone()
#     if not to_user:
#         return {"error": "not found"}, 404
#     if from_user["balance"] < data["amount"]:
#         return {"error": "insufficient"}, 400
#     db.execute(
#         "UPDATE users SET balance = balance - ? WHERE id = ?",
#         (data["amount"], data["from_id"])
#     )
#     db.execute(
#         "UPDATE users SET balance = balance + ? WHERE id = ?",
#         (data["amount"], data["to_id"])
#     )
#     db.commit()
#     return {"success": True}
#
# Refactor into proper layers with:
#   - Pydantic schema for validation
#   - UserRepository for DB operations
#   - TransferService for business logic
#   - Proper exceptions
#   - Logging
#   - Unit of Work for atomicity
```

---

## 🎉 Phase 4 Complete!

text

```
✅ 4.1 — HTTP & API Fundamentals
✅ 4.2 — FastAPI (with full project)
✅ 4.3 — Django & DRF
✅ 4.4 — Flask
✅ 4.5 — Project Structure & Patterns
```

**You can now build production-grade APIs in any Python framework.**