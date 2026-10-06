```
FastAPI is the modern Python web framework.

Why FastAPI over Django/Flask?
┌─────────────────────────────────────────────────────────┐
│  Speed:      One of the fastest Python frameworks       │
│              (comparable to Node.js and Go)             │
│  Validation: Automatic with Pydantic                    │
│  Docs:       Auto-generated OpenAPI/Swagger             │
│  Type hints: First-class support                        │
│  Async:      Built for async from the ground up         │
│  Modern:     Python 3.8+ features throughout            │
└─────────────────────────────────────────────────────────┘

Used by: Netflix, Uber, Microsoft, Explosion AI (spaCy)

By end of this phase you will have built:
  ✅ A complete REST API with authentication
  ✅ Database integration with SQLAlchemy
  ✅ JWT authentication with access/refresh tokens
  ✅ Role-based access control
  ✅ File uploads
  ✅ Background tasks
  ✅ WebSocket endpoint
  ✅ Auto-generated API documentation
```

---

## Setup

Bash

```
cd ~/projects
mkdir fastapi_project
cd fastapi_project
python3 -m venv venv
source venv/bin/activate

pip install fastapi "uvicorn[standard]" sqlalchemy alembic \
    pydantic pydantic-settings python-dotenv \
    python-jose[cryptography] passlib[bcrypt] \
    python-multipart httpx pytest pytest-asyncio \
    psycopg2-binary

# Project structure
mkdir -p app/{routers,models,schemas,services,repositories,core,middleware}
touch app/__init__.py
touch app/{routers,models,schemas,services,repositories,core,middleware}/__init__.py
touch app/main.py
touch app/core/{config.py,database.py,security.py,deps.py}
touch app/models/{user.py,post.py}
touch app/schemas/{user.py,post.py,auth.py,common.py}
touch app/routers/{users.py,posts.py,auth.py}
touch app/services/{user_service.py,post_service.py,auth_service.py}
touch app/repositories/{user_repo.py,post_repo.py}
touch .env
touch .env.example

code .
```

Bash

```
# .env
DATABASE_URL=postgresql://myapp_user:myapp_password@localhost:5432/fastapi_db
SECRET_KEY=your-super-secret-key-change-in-production-at-least-32-chars
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7
ENVIRONMENT=development
DEBUG=true
```

---

## 1. 🚀 Your First FastAPI App

Python

```
# Start simple — understand the foundation first
# temp_main.py (we'll build the real one step by step)

from fastapi import FastAPI
from datetime import datetime

# Create the FastAPI application
app = FastAPI(
    title="My First FastAPI App",
    description="Learning FastAPI step by step",
    version="1.0.0"
)

# Your first endpoint
@app.get("/")
def root():
    return {"message": "Hello, FastAPI!"}

# Run with:
# uvicorn temp_main:app --reload
# --reload = restart server when code changes

# Visit:
# http://localhost:8000/           ← your endpoint
# http://localhost:8000/docs       ← Swagger UI (auto-generated!)
# http://localhost:8000/redoc      ← ReDoc UI
# http://localhost:8000/openapi.json ← raw OpenAPI spec
```

Bash

```
# Run it
uvicorn temp_main:app --reload --port 8000
```

---

## 2. ⚙️ Configuration & Settings

Python

```
# app/core/config.py
from pydantic_settings import BaseSettings
from pydantic import AnyHttpUrl, validator
from typing import List, Optional
from functools import lru_cache


class Settings(BaseSettings):
    """
    Application settings loaded from environment variables.
    Pydantic validates all values automatically.
    """

    # ─────────────────────────────────────────
    # Application
    # ─────────────────────────────────────────
    APP_NAME: str = "FastAPI Blog"
    APP_VERSION: str = "1.0.0"
    ENVIRONMENT: str = "development"
    DEBUG: bool = False
    API_PREFIX: str = "/api/v1"

    # ─────────────────────────────────────────
    # Security
    # ─────────────────────────────────────────
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    REFRESH_TOKEN_EXPIRE_DAYS: int = 7

    # ─────────────────────────────────────────
    # Database
    # ─────────────────────────────────────────
    DATABASE_URL: str

    # ─────────────────────────────────────────
    # CORS
    # ─────────────────────────────────────────
    ALLOWED_ORIGINS: List[str] = [
        "http://localhost:3000",
        "http://localhost:8080",
    ]

    # ─────────────────────────────────────────
    # Pagination
    # ─────────────────────────────────────────
    DEFAULT_PAGE_SIZE: int = 20
    MAX_PAGE_SIZE: int = 100

    # ─────────────────────────────────────────
    # File uploads
    # ─────────────────────────────────────────
    MAX_FILE_SIZE_MB: int = 10
    UPLOAD_DIR: str = "uploads"

    @property
    def is_production(self) -> bool:
        return self.ENVIRONMENT == "production"

    @property
    def is_development(self) -> bool:
        return self.ENVIRONMENT == "development"

    class Config:
        env_file = ".env"
        case_sensitive = True


@lru_cache()  # cache so it's only created once
def get_settings() -> Settings:
    return Settings()


settings = get_settings()
```

---

## 3. 🗄️ Database Setup

Python

```
# app/core/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session, DeclarativeBase
from sqlalchemy import Column, DateTime, Integer, func
from typing import Generator
from .config import settings


# Create engine
engine = create_engine(
    settings.DATABASE_URL,
    pool_size=5,
    max_overflow=10,
    pool_pre_ping=True,
    echo=settings.DEBUG,
)

# Session factory
SessionLocal = sessionmaker(
    bind=engine,
    autocommit=False,
    autoflush=False,
)


class Base(DeclarativeBase):
    """Base class for all SQLAlchemy models."""
    pass


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


def get_db() -> Generator[Session, None, None]:
    """
    FastAPI dependency that provides a database session.
    Used with: db: Session = Depends(get_db)
    """
    db = SessionLocal()
    try:
        yield db
        db.commit()
    except Exception:
        db.rollback()
        raise
    finally:
        db.close()
```

---

## 4. 🔐 Security Utilities

Python

```
# app/core/security.py
from datetime import datetime, timedelta, timezone
from typing import Optional, Dict, Any
from jose import JWTError, jwt
from passlib.context import CryptContext
from .config import settings

# Password hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


def hash_password(password: str) -> str:
    """Hash a password using bcrypt."""
    return pwd_context.hash(password)


def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Verify a password against its hash."""
    return pwd_context.verify(plain_password, hashed_password)


def create_access_token(
    data: Dict[str, Any],
    expires_delta: Optional[timedelta] = None
) -> str:
    """Create a JWT access token."""
    to_encode = data.copy()

    expire = datetime.now(timezone.utc) + (
        expires_delta or
        timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    to_encode.update({
        "exp": expire,
        "type": "access"
    })

    return jwt.encode(
        to_encode,
        settings.SECRET_KEY,
        algorithm=settings.ALGORITHM
    )


def create_refresh_token(data: Dict[str, Any]) -> str:
    """Create a JWT refresh token."""
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + timedelta(
        days=settings.REFRESH_TOKEN_EXPIRE_DAYS
    )
    to_encode.update({
        "exp": expire,
        "type": "refresh"
    })

    return jwt.encode(
        to_encode,
        settings.SECRET_KEY,
        algorithm=settings.ALGORITHM
    )


def decode_token(token: str) -> Dict[str, Any]:
    """Decode and verify a JWT token."""
    try:
        payload = jwt.decode(
            token,
            settings.SECRET_KEY,
            algorithms=[settings.ALGORITHM]
        )
        return payload
    except JWTError as e:
        raise ValueError(f"Invalid token: {e}")


def create_token_pair(user_id: int, email: str, role: str) -> Dict[str, str]:
    """Create both access and refresh tokens."""
    token_data = {
        "sub": str(user_id),
        "email": email,
        "role": role
    }
    return {
        "access_token": create_access_token(token_data),
        "refresh_token": create_refresh_token(token_data),
        "token_type": "bearer"
    }
```

---

## 5. 📋 Pydantic Schemas

Python

```
# app/schemas/common.py
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional, List, Any

T = TypeVar("T")


class PaginationMeta(BaseModel):
    """Pagination metadata."""
    total: int
    page: int
    page_size: int
    total_pages: int
    has_next: bool
    has_prev: bool


class PaginatedResponse(BaseModel, Generic[T]):
    """Generic paginated response."""
    success: bool = True
    data: List[T]
    pagination: PaginationMeta


class SuccessResponse(BaseModel, Generic[T]):
    """Generic success response."""
    success: bool = True
    message: str = "Success"
    data: Optional[T] = None


class ErrorDetail(BaseModel):
    """Error detail item."""
    field: Optional[str] = None
    message: str
    code: Optional[str] = None


class ErrorResponse(BaseModel):
    """Standard error response."""
    success: bool = False
    message: str
    error_code: Optional[str] = None
    details: Optional[List[ErrorDetail]] = None
```

Python

```
# app/schemas/user.py
from pydantic import BaseModel, EmailStr, Field, field_validator
from typing import Optional
from datetime import datetime
import re


class UserBase(BaseModel):
    """Fields shared across user schemas."""
    name: str = Field(..., min_length=2, max_length=100)
    email: EmailStr


class UserCreate(UserBase):
    """Schema for creating a user."""
    password: str = Field(..., min_length=8, max_length=100)
    role: str = Field(default="user")

    @field_validator("password")
    @classmethod
    def password_strength(cls, v: str) -> str:
        """Validate password strength."""
        if not any(c.isupper() for c in v):
            raise ValueError("Password must contain at least one uppercase letter")
        if not any(c.isdigit() for c in v):
            raise ValueError("Password must contain at least one number")
        return v

    @field_validator("role")
    @classmethod
    def validate_role(cls, v: str) -> str:
        allowed = {"user", "admin", "moderator"}
        if v not in allowed:
            raise ValueError(f"Role must be one of {allowed}")
        return v

    @field_validator("name")
    @classmethod
    def clean_name(cls, v: str) -> str:
        return v.strip().title()


class UserUpdate(BaseModel):
    """Schema for updating a user (all optional)."""
    name: Optional[str] = Field(None, min_length=2, max_length=100)
    bio: Optional[str] = Field(None, max_length=500)
    website: Optional[str] = Field(None, max_length=200)

    @field_validator("name")
    @classmethod
    def clean_name(cls, v: Optional[str]) -> Optional[str]:
        if v:
            return v.strip().title()
        return v


class UserResponse(UserBase):
    """Schema for returning user data (no password)."""
    id: int
    role: str
    is_active: bool
    is_verified: bool
    bio: Optional[str] = None
    created_at: datetime
    updated_at: datetime

    model_config = {"from_attributes": True}  # allow ORM objects


class UserListResponse(BaseModel):
    """User in a list (fewer fields)."""
    id: int
    name: str
    email: str
    role: str
    is_active: bool
    created_at: datetime

    model_config = {"from_attributes": True}
```

Python

```
# app/schemas/auth.py
from pydantic import BaseModel, EmailStr, Field
from typing import Optional
from .user import UserResponse


class LoginRequest(BaseModel):
    """Login credentials."""
    email: EmailStr
    password: str = Field(..., min_length=1)


class TokenResponse(BaseModel):
    """Token pair returned after login/register."""
    access_token: str
    refresh_token: str
    token_type: str = "bearer"
    user: UserResponse


class RefreshRequest(BaseModel):
    """Request to refresh access token."""
    refresh_token: str


class PasswordChangeRequest(BaseModel):
    """Change password."""
    current_password: str
    new_password: str = Field(..., min_length=8)


class PasswordResetRequest(BaseModel):
    """Request password reset (forgot password)."""
    email: EmailStr


class PasswordResetConfirm(BaseModel):
    """Complete password reset with token."""
    token: str
    new_password: str = Field(..., min_length=8)
```

Python

```
# app/schemas/post.py
from pydantic import BaseModel, Field, field_validator
from typing import Optional, List
from datetime import datetime
import re


class PostCreate(BaseModel):
    """Schema for creating a post."""
    title: str = Field(..., min_length=3, max_length=500)
    content: str = Field(..., min_length=10)
    tags: List[str] = Field(default_factory=list, max_length=10)
    status: str = Field(default="draft")

    @field_validator("title")
    @classmethod
    def clean_title(cls, v: str) -> str:
        return v.strip()

    @field_validator("tags")
    @classmethod
    def clean_tags(cls, v: List[str]) -> List[str]:
        return [tag.lower().strip() for tag in v if tag.strip()]

    @field_validator("status")
    @classmethod
    def validate_status(cls, v: str) -> str:
        allowed = {"draft", "published"}
        if v not in allowed:
            raise ValueError(f"Status must be one of {allowed}")
        return v

    @property
    def slug(self) -> str:
        """Generate URL slug from title."""
        slug = self.title.lower()
        slug = re.sub(r'[^a-z0-9\s-]', '', slug)
        slug = re.sub(r'[\s-]+', '-', slug)
        return slug.strip('-')


class PostUpdate(BaseModel):
    """Schema for updating a post."""
    title: Optional[str] = Field(None, min_length=3, max_length=500)
    content: Optional[str] = Field(None, min_length=10)
    tags: Optional[List[str]] = None
    status: Optional[str] = None


class PostResponse(BaseModel):
    """Post response schema."""
    id: int
    title: str
    content: str
    slug: str
    status: str
    tags: List[str]
    view_count: int
    like_count: int
    author_id: int
    author_name: Optional[str] = None
    published_at: Optional[datetime] = None
    created_at: datetime
    updated_at: datetime

    model_config = {"from_attributes": True}


class PostListResponse(BaseModel):
    """Post in a list (fewer fields, no full content)."""
    id: int
    title: str
    slug: str
    status: str
    tags: List[str]
    view_count: int
    like_count: int
    author_id: int
    author_name: Optional[str] = None
    published_at: Optional[datetime] = None
    created_at: datetime

    model_config = {"from_attributes": True}
```

---

## 6. 🏗️ SQLAlchemy Models

Python

```
# app/models/user.py
from sqlalchemy import (
    Column, Integer, String, Boolean,
    Text, DateTime, Numeric, func, CheckConstraint, Index
)
from sqlalchemy.orm import relationship, validates
from sqlalchemy.dialects.postgresql import JSONB
from app.core.database import Base, TimestampMixin
import re


class User(Base, TimestampMixin):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), nullable=False, unique=True, index=True)
    password_hash = Column(String(255), nullable=False)
    bio = Column(Text, nullable=True)
    website = Column(String(200), nullable=True)
    role = Column(String(20), nullable=False, server_default="user")
    is_active = Column(Boolean, nullable=False, server_default="true")
    is_verified = Column(Boolean, nullable=False, server_default="false")
    deleted_at = Column(DateTime(timezone=True), nullable=True)

    __table_args__ = (
        CheckConstraint(
            "role IN ('user', 'admin', 'moderator')",
            name="check_user_role"
        ),
        Index("idx_users_role_active", "role", "is_active"),
    )

    # Relationships
    posts = relationship(
        "Post",
        back_populates="author",
        cascade="all, delete-orphan"
    )

    @validates("email")
    def validate_email(self, key, email):
        return email.lower().strip()

    @property
    def is_admin(self) -> bool:
        return self.role == "admin"

    @property
    def is_deleted(self) -> bool:
        return self.deleted_at is not None

    def __repr__(self):
        return f"<User id={self.id} email={self.email!r}>"
```

Python

```
# app/models/post.py
from sqlalchemy import (
    Column, Integer, String, Text, Boolean,
    DateTime, ForeignKey, CheckConstraint, Index, func
)
from sqlalchemy.orm import relationship, validates
from sqlalchemy.dialects.postgresql import ARRAY
from sqlalchemy import String as SAString
from app.core.database import Base, TimestampMixin
import re


class Post(Base, TimestampMixin):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(500), nullable=False)
    content = Column(Text, nullable=False)
    slug = Column(String(500), unique=True, index=True)
    status = Column(String(20), nullable=False, server_default="draft")
    tags = Column(ARRAY(SAString), nullable=False, server_default="{}")
    view_count = Column(Integer, nullable=False, server_default="0")
    like_count = Column(Integer, nullable=False, server_default="0")
    published_at = Column(DateTime(timezone=True), nullable=True)
    deleted_at = Column(DateTime(timezone=True), nullable=True)

    author_id = Column(
        Integer,
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True
    )

    __table_args__ = (
        CheckConstraint(
            "status IN ('draft', 'published', 'archived')",
            name="check_post_status"
        ),
        Index("idx_posts_author_status", "author_id", "status"),
    )

    # Relationships
    author = relationship("User", back_populates="posts")

    @validates("slug")
    def validate_slug(self, key, slug):
        if slug:
            slug = slug.lower().strip()
            slug = re.sub(r'[^a-z0-9-]', '-', slug)
            slug = re.sub(r'-+', '-', slug)
            slug = slug.strip('-')
        return slug

    @property
    def is_published(self) -> bool:
        return self.status == "published"

    def __repr__(self):
        return f"<Post id={self.id} title={self.title!r}>"
```

---

## 7. 🔒 Dependencies

Python

```
# app/core/deps.py
"""
FastAPI's dependency injection system.
Dependencies are functions that run before your route handler.
They can: validate auth, get DB session, check permissions, etc.
"""
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.orm import Session
from typing import Optional, Annotated
from jose import JWTError

from .database import get_db
from .security import decode_token
from .config import settings
from app.models.user import User

# OAuth2 scheme — tells FastAPI where to get the token
# tokenUrl = endpoint that issues tokens (for Swagger UI login)
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl=f"{settings.API_PREFIX}/auth/login"
)

# Optional OAuth2 — doesn't raise error if no token
oauth2_scheme_optional = OAuth2PasswordBearer(
    tokenUrl=f"{settings.API_PREFIX}/auth/login",
    auto_error=False
)


def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    """
    Dependency: Get the current authenticated user.
    Raises 401 if token is invalid or user not found.
    """
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    try:
        payload = decode_token(token)
        user_id: str = payload.get("sub")
        token_type: str = payload.get("type")

        if user_id is None or token_type != "access":
            raise credentials_exception

    except ValueError:
        raise credentials_exception

    # Get user from database
    user = db.query(User).filter(
        User.id == int(user_id),
        User.deleted_at.is_(None)
    ).first()

    if user is None:
        raise credentials_exception

    return user


def get_current_active_user(
    current_user: User = Depends(get_current_user)
) -> User:
    """Dependency: Ensure user is active."""
    if not current_user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Account is deactivated"
        )
    return current_user


def get_current_admin(
    current_user: User = Depends(get_current_active_user)
) -> User:
    """Dependency: Ensure user is an admin."""
    if current_user.role != "admin":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Admin access required"
        )
    return current_user


def get_optional_user(
    token: Optional[str] = Depends(oauth2_scheme_optional),
    db: Session = Depends(get_db)
) -> Optional[User]:
    """
    Dependency: Get user if authenticated, None otherwise.
    Use for endpoints that work differently for authenticated users.
    """
    if not token:
        return None
    try:
        payload = decode_token(token)
        user_id = payload.get("sub")
        if user_id:
            return db.query(User).filter(
                User.id == int(user_id),
                User.deleted_at.is_(None)
            ).first()
    except (ValueError, Exception):
        pass
    return None


def require_role(*roles: str):
    """
    Dependency factory: require user to have one of the given roles.
    
    Usage:
        @router.delete("/{id}")
        def delete(user = Depends(require_role("admin", "moderator"))):
    """
    def check_role(
        current_user: User = Depends(get_current_active_user)
    ) -> User:
        if current_user.role not in roles:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Required role: {' or '.join(roles)}"
            )
        return current_user
    return check_role


# Type aliases for cleaner code
CurrentUser = Annotated[User, Depends(get_current_active_user)]
AdminUser = Annotated[User, Depends(get_current_admin)]
OptionalUser = Annotated[Optional[User], Depends(get_optional_user)]
DBSession = Annotated[Session, Depends(get_db)]
```

---

## 8. 📁 Repositories

Python

```
# app/repositories/user_repo.py
from sqlalchemy.orm import Session
from sqlalchemy import func, or_
from typing import Optional, List, Dict, Any
from datetime import datetime, timezone
from app.models.user import User


class UserRepository:
    def __init__(self, db: Session):
        self.db = db

    def create(self, **kwargs) -> User:
        user = User(**kwargs)
        self.db.add(user)
        self.db.flush()
        return user

    def get_by_id(self, user_id: int) -> Optional[User]:
        return self.db.query(User).filter(
            User.id == user_id,
            User.deleted_at.is_(None)
        ).first()

    def get_by_email(self, email: str) -> Optional[User]:
        return self.db.query(User).filter(
            func.lower(User.email) == email.lower(),
            User.deleted_at.is_(None)
        ).first()

    def get_all(
        self,
        skip: int = 0,
        limit: int = 20,
        role: Optional[str] = None,
        is_active: Optional[bool] = None,
        search: Optional[str] = None
    ) -> tuple[List[User], int]:
        """Returns (users, total_count)."""
        query = self.db.query(User).filter(User.deleted_at.is_(None))

        if role:
            query = query.filter(User.role == role)
        if is_active is not None:
            query = query.filter(User.is_active == is_active)
        if search:
            term = f"%{search}%"
            query = query.filter(
                or_(User.name.ilike(term), User.email.ilike(term))
            )

        total = query.count()
        users = query.order_by(User.created_at.desc()).offset(skip).limit(limit).all()
        return users, total

    def update(self, user: User, **updates) -> User:
        for key, value in updates.items():
            if hasattr(user, key) and value is not None:
                setattr(user, key, value)
        self.db.flush()
        return user

    def soft_delete(self, user: User) -> User:
        user.deleted_at = datetime.now(timezone.utc)
        user.is_active = False
        self.db.flush()
        return user

    def email_exists(self, email: str) -> bool:
        return self.db.query(
            self.db.query(User).filter(
                func.lower(User.email) == email.lower()
            ).exists()
        ).scalar()
```

Python

```
# app/repositories/post_repo.py
from sqlalchemy.orm import Session, joinedload
from sqlalchemy import func, or_
from typing import Optional, List
from datetime import datetime, timezone
from app.models.post import Post
from app.models.user import User


class PostRepository:
    def __init__(self, db: Session):
        self.db = db

    def create(self, **kwargs) -> Post:
        post = Post(**kwargs)
        self.db.add(post)
        self.db.flush()
        return post

    def get_by_id(self, post_id: int) -> Optional[Post]:
        return self.db.query(Post).options(
            joinedload(Post.author)
        ).filter(
            Post.id == post_id,
            Post.deleted_at.is_(None)
        ).first()

    def get_by_slug(self, slug: str) -> Optional[Post]:
        return self.db.query(Post).options(
            joinedload(Post.author)
        ).filter(
            Post.slug == slug,
            Post.deleted_at.is_(None)
        ).first()

    def get_all(
        self,
        skip: int = 0,
        limit: int = 20,
        status: Optional[str] = None,
        author_id: Optional[int] = None,
        tag: Optional[str] = None,
        search: Optional[str] = None
    ) -> tuple[List[Post], int]:
        query = self.db.query(Post).options(
            joinedload(Post.author)
        ).filter(Post.deleted_at.is_(None))

        if status:
            query = query.filter(Post.status == status)
        if author_id:
            query = query.filter(Post.author_id == author_id)
        if tag:
            query = query.filter(Post.tags.contains([tag]))
        if search:
            term = f"%{search}%"
            query = query.filter(
                or_(Post.title.ilike(term), Post.content.ilike(term))
            )

        total = query.count()
        posts = query.order_by(Post.created_at.desc()).offset(skip).limit(limit).all()
        return posts, total

    def update(self, post: Post, **updates) -> Post:
        for key, value in updates.items():
            if hasattr(post, key) and value is not None:
                setattr(post, key, value)
        self.db.flush()
        return post

    def increment_views(self, post_id: int) -> None:
        self.db.query(Post).filter(Post.id == post_id).update(
            {"view_count": Post.view_count + 1}
        )

    def slug_exists(self, slug: str, exclude_id: Optional[int] = None) -> bool:
        query = self.db.query(Post).filter(Post.slug == slug)
        if exclude_id:
            query = query.filter(Post.id != exclude_id)
        return query.first() is not None

    def soft_delete(self, post: Post) -> Post:
        post.deleted_at = datetime.now(timezone.utc)
        self.db.flush()
        return post
```

---

## 9. 🔧 Services

Python

```
# app/services/auth_service.py
from sqlalchemy.orm import Session
from fastapi import HTTPException, status
from typing import Dict, Any
from app.repositories.user_repo import UserRepository
from app.core.security import (
    hash_password, verify_password, create_token_pair, decode_token
)
from app.schemas.user import UserCreate
from app.schemas.auth import LoginRequest
from app.models.user import User


class AuthService:
    def __init__(self, db: Session):
        self.db = db
        self.user_repo = UserRepository(db)

    def register(self, data: UserCreate) -> Dict[str, Any]:
        """Register a new user."""
        # Check email uniqueness
        if self.user_repo.email_exists(data.email):
            raise HTTPException(
                status_code=status.HTTP_409_CONFLICT,
                detail="Email already registered"
            )

        # Create user
        user = self.user_repo.create(
            name=data.name,
            email=data.email.lower(),
            password_hash=hash_password(data.password),
            role=data.role
        )

        # Generate tokens
        tokens = create_token_pair(user.id, user.email, user.role)
        return {"user": user, **tokens}

    def login(self, data: LoginRequest) -> Dict[str, Any]:
        """Login and return tokens."""
        user = self.user_repo.get_by_email(data.email)

        if not user or not verify_password(data.password, user.password_hash):
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Incorrect email or password"
            )

        if not user.is_active:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Account is deactivated"
            )

        tokens = create_token_pair(user.id, user.email, user.role)
        return {"user": user, **tokens}

    def refresh_tokens(self, refresh_token: str) -> Dict[str, str]:
        """Refresh the access token using a refresh token."""
        try:
            payload = decode_token(refresh_token)
            if payload.get("type") != "refresh":
                raise ValueError("Not a refresh token")

            user_id = int(payload.get("sub"))
        except (ValueError, Exception):
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Invalid refresh token"
            )

        user = self.user_repo.get_by_id(user_id)
        if not user or not user.is_active:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="User not found or inactive"
            )

        return create_token_pair(user.id, user.email, user.role)
```

Python

```
# app/services/user_service.py
from sqlalchemy.orm import Session
from fastapi import HTTPException, status
from typing import Dict, Any, Optional
from app.repositories.user_repo import UserRepository
from app.core.security import hash_password, verify_password
from app.schemas.user import UserCreate, UserUpdate
from app.models.user import User


class UserService:
    def __init__(self, db: Session):
        self.db = db
        self.repo = UserRepository(db)

    def get_user_or_404(self, user_id: int) -> User:
        user = self.repo.get_by_id(user_id)
        if not user:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"User {user_id} not found"
            )
        return user

    def get_users(
        self,
        page: int = 1,
        page_size: int = 20,
        **filters
    ) -> Dict[str, Any]:
        skip = (page - 1) * page_size
        users, total = self.repo.get_all(skip=skip, limit=page_size, **filters)
        return {
            "users": users,
            "total": total,
            "page": page,
            "page_size": page_size,
            "total_pages": (total + page_size - 1) // page_size,
        }

    def get_user(self, user_id: int) -> User:
        return self.get_user_or_404(user_id)

    def update_user(
        self,
        user_id: int,
        data: UserUpdate,
        current_user: User
    ) -> User:
        """Update user — only self or admin."""
        user = self.get_user_or_404(user_id)

        # Authorization check
        if user.id != current_user.id and not current_user.is_admin:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Cannot update another user's profile"
            )

        updates = data.model_dump(exclude_unset=True)
        return self.repo.update(user, **updates)

    def change_password(
        self,
        user_id: int,
        current_password: str,
        new_password: str,
        current_user: User
    ) -> None:
        user = self.get_user_or_404(user_id)

        if user.id != current_user.id and not current_user.is_admin:
            raise HTTPException(status_code=403, detail="Forbidden")

        if not verify_password(current_password, user.password_hash):
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Current password is incorrect"
            )

        self.repo.update(user, password_hash=hash_password(new_password))

    def delete_user(self, user_id: int, current_user: User) -> None:
        user = self.get_user_or_404(user_id)

        if user.id != current_user.id and not current_user.is_admin:
            raise HTTPException(status_code=403, detail="Forbidden")

        self.repo.soft_delete(user)
```

Python

```
# app/services/post_service.py
from sqlalchemy.orm import Session
from fastapi import HTTPException, status
from typing import Dict, Any, Optional
from datetime import datetime, timezone
from app.repositories.post_repo import PostRepository
from app.schemas.post import PostCreate, PostUpdate
from app.models.post import Post
from app.models.user import User
import re


class PostService:
    def __init__(self, db: Session):
        self.db = db
        self.repo = PostRepository(db)

    def _generate_unique_slug(self, title: str, exclude_id: Optional[int] = None) -> str:
        """Generate unique slug from title."""
        base_slug = title.lower().strip()
        base_slug = re.sub(r'[^a-z0-9\s-]', '', base_slug)
        base_slug = re.sub(r'[\s-]+', '-', base_slug).strip('-')

        slug = base_slug
        counter = 1
        while self.repo.slug_exists(slug, exclude_id=exclude_id):
            slug = f"{base_slug}-{counter}"
            counter += 1
        return slug

    def get_post_or_404(self, post_id: int) -> Post:
        post = self.repo.get_by_id(post_id)
        if not post:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"Post {post_id} not found"
            )
        return post

    def get_posts(
        self,
        page: int = 1,
        page_size: int = 20,
        **filters
    ) -> Dict[str, Any]:
        skip = (page - 1) * page_size
        posts, total = self.repo.get_all(skip=skip, limit=page_size, **filters)
        return {
            "posts": posts,
            "total": total,
            "page": page,
            "page_size": page_size,
            "total_pages": (total + page_size - 1) // page_size,
        }

    def get_post(self, post_id: int) -> Post:
        post = self.get_post_or_404(post_id)
        self.repo.increment_views(post_id)
        return post

    def create_post(self, data: PostCreate, author: User) -> Post:
        slug = self._generate_unique_slug(data.title)
        published_at = (
            datetime.now(timezone.utc)
            if data.status == "published"
            else None
        )

        return self.repo.create(
            title=data.title,
            content=data.content,
            slug=slug,
            status=data.status,
            tags=data.tags,
            author_id=author.id,
            published_at=published_at
        )

    def update_post(
        self,
        post_id: int,
        data: PostUpdate,
        current_user: User
    ) -> Post:
        post = self.get_post_or_404(post_id)

        # Authorization
        if post.author_id != current_user.id and not current_user.is_admin:
            raise HTTPException(status_code=403, detail="Forbidden")

        updates = data.model_dump(exclude_unset=True)

        if "title" in updates:
            updates["slug"] = self._generate_unique_slug(
                updates["title"], exclude_id=post_id
            )

        if updates.get("status") == "published" and post.status != "published":
            updates["published_at"] = datetime.now(timezone.utc)

        return self.repo.update(post, **updates)

    def delete_post(self, post_id: int, current_user: User) -> None:
        post = self.get_post_or_404(post_id)

        if post.author_id != current_user.id and not current_user.is_admin:
            raise HTTPException(status_code=403, detail="Forbidden")

        self.repo.soft_delete(post)
```

---

## 10. 🛣️ Routers

Python

```
# app/routers/auth.py
from fastapi import APIRouter, Depends, HTTPException, status, BackgroundTasks
from sqlalchemy.orm import Session
from app.core.deps import DBSession, CurrentUser
from app.services.auth_service import AuthService
from app.schemas.auth import LoginRequest, TokenResponse, RefreshRequest
from app.schemas.user import UserCreate, UserResponse
from app.schemas.common import SuccessResponse

router = APIRouter(prefix="/auth", tags=["Authentication"])


@router.post(
    "/register",
    response_model=TokenResponse,
    status_code=status.HTTP_201_CREATED,
    summary="Register a new user"
)
def register(
    data: UserCreate,
    db: DBSession,
    background_tasks: BackgroundTasks
) -> TokenResponse:
    """
    Register a new user account.
    Returns access and refresh tokens upon successful registration.
    """
    service = AuthService(db)
    result = service.register(data)

    # Background task: send welcome email (doesn't block response)
    background_tasks.add_task(
        send_welcome_email,
        result["user"].email,
        result["user"].name
    )

    return TokenResponse(
        access_token=result["access_token"],
        refresh_token=result["refresh_token"],
        user=result["user"]
    )


@router.post(
    "/login",
    response_model=TokenResponse,
    summary="Login and get tokens"
)
def login(data: LoginRequest, db: DBSession) -> TokenResponse:
    """
    Login with email and password.
    Returns access token (short-lived) and refresh token (long-lived).
    """
    service = AuthService(db)
    result = service.login(data)

    return TokenResponse(
        access_token=result["access_token"],
        refresh_token=result["refresh_token"],
        user=result["user"]
    )


@router.post(
    "/refresh",
    response_model=TokenResponse,
    summary="Refresh access token"
)
def refresh_token(
    data: RefreshRequest,
    db: DBSession
) -> TokenResponse:
    """
    Get a new access token using your refresh token.
    Use when access token has expired.
    """
    service = AuthService(db)
    result = service.refresh_tokens(data.refresh_token)
    return result


@router.get(
    "/me",
    response_model=UserResponse,
    summary="Get current user"
)
def get_me(current_user: CurrentUser) -> UserResponse:
    """Get the currently authenticated user's profile."""
    return current_user


@router.post(
    "/logout",
    status_code=status.HTTP_204_NO_CONTENT,
    summary="Logout"
)
def logout(current_user: CurrentUser):
    """
    Logout the current user.
    In a real app you'd add the token to a blocklist.
    """
    # In production: add token to Redis blocklist
    # redis_client.setex(f"blocklist:{token}", ttl, "1")
    return None


def send_welcome_email(email: str, name: str) -> None:
    """Background task: send welcome email."""
    # In production: use SendGrid, SES, etc.
    print(f"[EMAIL] Sending welcome email to {name} at {email}")
```

Python

```
# app/routers/users.py
from fastapi import APIRouter, Depends, Query, status
from typing import Optional
from app.core.deps import DBSession, CurrentUser, AdminUser
from app.services.user_service import UserService
from app.schemas.user import UserResponse, UserUpdate, UserListResponse
from app.schemas.auth import PasswordChangeRequest
from app.schemas.common import PaginatedResponse, PaginationMeta, SuccessResponse

router = APIRouter(prefix="/users", tags=["Users"])


@router.get(
    "",
    summary="List all users",
    response_model=PaginatedResponse[UserListResponse]
)
def list_users(
    db: DBSession,
    _: AdminUser,             # only admins can list all users
    page: int = Query(1, ge=1, description="Page number"),
    page_size: int = Query(20, ge=1, le=100, description="Items per page"),
    role: Optional[str] = Query(None, description="Filter by role"),
    is_active: Optional[bool] = Query(None, description="Filter by active status"),
    search: Optional[str] = Query(None, max_length=100, description="Search by name or email"),
):
    """
    List all users with pagination and filtering.
    **Admin only.**
    """
    service = UserService(db)
    result = service.get_users(
        page=page,
        page_size=page_size,
        role=role,
        is_active=is_active,
        search=search
    )

    return PaginatedResponse(
        data=result["users"],
        pagination=PaginationMeta(
            total=result["total"],
            page=result["page"],
            page_size=result["page_size"],
            total_pages=result["total_pages"],
            has_next=result["page"] * result["page_size"] < result["total"],
            has_prev=result["page"] > 1
        )
    )


@router.get(
    "/{user_id}",
    response_model=UserResponse,
    summary="Get a user by ID"
)
def get_user(
    user_id: int,
    db: DBSession,
    current_user: CurrentUser
):
    """Get a user's profile. Users can see their own profile, admins can see all."""
    service = UserService(db)
    user = service.get_user(user_id)

    # Non-admins can only see their own profile
    if user.id != current_user.id and not current_user.is_admin:
        from fastapi import HTTPException
        raise HTTPException(status_code=403, detail="Forbidden")

    return user


@router.patch(
    "/{user_id}",
    response_model=UserResponse,
    summary="Update a user"
)
def update_user(
    user_id: int,
    data: UserUpdate,
    db: DBSession,
    current_user: CurrentUser
):
    """Update a user's profile. Users can only update their own."""
    service = UserService(db)
    return service.update_user(user_id, data, current_user)


@router.post(
    "/{user_id}/change-password",
    status_code=status.HTTP_204_NO_CONTENT,
    summary="Change password"
)
def change_password(
    user_id: int,
    data: PasswordChangeRequest,
    db: DBSession,
    current_user: CurrentUser
):
    """Change user's password."""
    service = UserService(db)
    service.change_password(
        user_id,
        data.current_password,
        data.new_password,
        current_user
    )


@router.delete(
    "/{user_id}",
    status_code=status.HTTP_204_NO_CONTENT,
    summary="Delete a user"
)
def delete_user(
    user_id: int,
    db: DBSession,
    current_user: CurrentUser
):
    """Delete a user account. Users can delete their own, admins can delete any."""
    service = UserService(db)
    service.delete_user(user_id, current_user)
```

Python

```
# app/routers/posts.py
from fastapi import (
    APIRouter, Depends, Query, status,
    UploadFile, File, BackgroundTasks
)
from typing import Optional, List
from app.core.deps import DBSession, CurrentUser, OptionalUser
from app.services.post_service import PostService
from app.schemas.post import PostCreate, PostUpdate, PostResponse, PostListResponse
from app.schemas.common import PaginatedResponse, PaginationMeta

router = APIRouter(prefix="/posts", tags=["Posts"])


@router.get(
    "",
    response_model=PaginatedResponse[PostListResponse],
    summary="List posts"
)
def list_posts(
    db: DBSession,
    current_user: OptionalUser = None,  # optional auth
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    status: Optional[str] = Query(None),
    author_id: Optional[int] = Query(None),
    tag: Optional[str] = Query(None),
    search: Optional[str] = Query(None, max_length=200),
):
    """
    List posts with filtering.
    Public users see only published posts.
    Authenticated users can also see their own drafts.
    """
    service = PostService(db)

    # Enforce published-only for non-authenticated users
    if not current_user:
        status = "published"

    result = service.get_posts(
        page=page,
        page_size=page_size,
        status=status,
        author_id=author_id,
        tag=tag,
        search=search
    )

    return PaginatedResponse(
        data=result["posts"],
        pagination=PaginationMeta(
            total=result["total"],
            page=result["page"],
            page_size=result["page_size"],
            total_pages=result["total_pages"],
            has_next=result["page"] * result["page_size"] < result["total"],
            has_prev=result["page"] > 1
        )
    )


@router.post(
    "",
    response_model=PostResponse,
    status_code=status.HTTP_201_CREATED,
    summary="Create a post"
)
def create_post(
    data: PostCreate,
    db: DBSession,
    current_user: CurrentUser
):
    """Create a new post. Authentication required."""
    service = PostService(db)
    return service.create_post(data, current_user)


@router.get(
    "/{post_id}",
    response_model=PostResponse,
    summary="Get a post"
)
def get_post(post_id: int, db: DBSession):
    """Get a post by ID. Increments view count."""
    service = PostService(db)
    return service.get_post(post_id)


@router.patch(
    "/{post_id}",
    response_model=PostResponse,
    summary="Update a post"
)
def update_post(
    post_id: int,
    data: PostUpdate,
    db: DBSession,
    current_user: CurrentUser
):
    """Update a post. Only author or admin can update."""
    service = PostService(db)
    return service.update_post(post_id, data, current_user)


@router.delete(
    "/{post_id}",
    status_code=status.HTTP_204_NO_CONTENT,
    summary="Delete a post"
)
def delete_post(
    post_id: int,
    db: DBSession,
    current_user: CurrentUser
):
    """Delete a post. Only author or admin can delete."""
    service = PostService(db)
    service.delete_post(post_id, current_user)


@router.post(
    "/{post_id}/publish",
    response_model=PostResponse,
    summary="Publish a post"
)
def publish_post(
    post_id: int,
    db: DBSession,
    current_user: CurrentUser
):
    """Publish a draft post."""
    service = PostService(db)
    return service.update_post(
        post_id,
        PostUpdate(status="published"),
        current_user
    )
```

---

## 11. 📤 File Uploads

Python

```
# app/routers/files.py
from fastapi import APIRouter, UploadFile, File, HTTPException, Depends
from fastapi.responses import FileResponse
from pathlib import Path
import shutil
import uuid
from app.core.deps import CurrentUser
from app.core.config import settings

router = APIRouter(prefix="/files", tags=["Files"])

UPLOAD_DIR = Path(settings.UPLOAD_DIR)
UPLOAD_DIR.mkdir(exist_ok=True)

ALLOWED_TYPES = {
    "image/jpeg": ".jpg",
    "image/png": ".png",
    "image/gif": ".gif",
    "image/webp": ".webp",
    "application/pdf": ".pdf",
}


@router.post("/upload", summary="Upload a file")
async def upload_file(
    file: UploadFile = File(...),
    current_user: CurrentUser = Depends()
) -> dict:
    """
    Upload a file.
    Supported: JPEG, PNG, GIF, WebP, PDF
    Max size: 10MB
    """
    # Validate file type
    if file.content_type not in ALLOWED_TYPES:
        raise HTTPException(
            status_code=400,
            detail=f"File type {file.content_type} not allowed. "
                   f"Allowed: {list(ALLOWED_TYPES.keys())}"
        )

    # Check file size
    max_size = settings.MAX_FILE_SIZE_MB * 1024 * 1024
    content = await file.read()
    if len(content) > max_size:
        raise HTTPException(
            status_code=413,
            detail=f"File too large. Max size: {settings.MAX_FILE_SIZE_MB}MB"
        )

    # Generate unique filename
    extension = ALLOWED_TYPES[file.content_type]
    filename = f"{uuid.uuid4()}{extension}"
    file_path = UPLOAD_DIR / filename

    # Save file
    file_path.write_bytes(content)

    return {
        "filename": filename,
        "original_name": file.filename,
        "size_bytes": len(content),
        "content_type": file.content_type,
        "url": f"/api/v1/files/{filename}"
    }


@router.post("/upload-multiple", summary="Upload multiple files")
async def upload_multiple(
    files: list[UploadFile] = File(...),
    current_user: CurrentUser = Depends()
) -> dict:
    """Upload up to 10 files at once."""
    if len(files) > 10:
        raise HTTPException(status_code=400, detail="Max 10 files at once")

    results = []
    for file in files:
        content = await file.read()
        extension = ALLOWED_TYPES.get(file.content_type, ".bin")
        filename = f"{uuid.uuid4()}{extension}"
        (UPLOAD_DIR / filename).write_bytes(content)
        results.append({
            "filename": filename,
            "original_name": file.filename,
            "size_bytes": len(content)
        })

    return {"uploaded": results, "count": len(results)}


@router.get("/{filename}", summary="Download a file")
def download_file(filename: str) -> FileResponse:
    """Download/view an uploaded file."""
    file_path = UPLOAD_DIR / filename

    # Security: prevent path traversal
    if not file_path.is_file() or not file_path.is_relative_to(UPLOAD_DIR):
        raise HTTPException(status_code=404, detail="File not found")

    return FileResponse(file_path)
```

---

## 12. 🔄 Middleware

Python

```
# app/middleware/logging.py
from fastapi import Request, Response
from starlette.middleware.base import BaseHTTPMiddleware
import time
import uuid
import logging

logger = logging.getLogger(__name__)


class RequestLoggingMiddleware(BaseHTTPMiddleware):
    """Log all incoming requests and responses."""

    async def dispatch(self, request: Request, call_next) -> Response:
        # Generate unique request ID
        request_id = str(uuid.uuid4())[:8]
        request.state.request_id = request_id

        start_time = time.perf_counter()

        # Log request
        logger.info(
            f"[{request_id}] → {request.method} {request.url.path}"
            f" from {request.client.host if request.client else 'unknown'}"
        )

        # Process request
        response = await call_next(request)

        # Calculate duration
        duration_ms = (time.perf_counter() - start_time) * 1000

        # Add headers to response
        response.headers["X-Request-ID"] = request_id
        response.headers["X-Response-Time"] = f"{duration_ms:.2f}ms"

        # Log response
        logger.info(
            f"[{request_id}] ← {response.status_code}"
            f" {duration_ms:.2f}ms"
        )

        return response
```

Python

```
# app/middleware/rate_limit.py
from fastapi import Request, Response, HTTPException
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.responses import JSONResponse
import time
from collections import defaultdict
from threading import Lock


class RateLimitMiddleware(BaseHTTPMiddleware):
    """
    Simple in-memory rate limiter.
    In production: use Redis-based rate limiting.
    """

    def __init__(self, app, max_requests: int = 100, window: int = 60):
        super().__init__(app)
        self.max_requests = max_requests
        self.window = window
        self.requests = defaultdict(list)
        self.lock = Lock()

    def _get_client_id(self, request: Request) -> str:
        # Use X-Forwarded-For if behind a proxy
        forwarded = request.headers.get("X-Forwarded-For")
        if forwarded:
            return forwarded.split(",")[0].strip()
        return request.client.host if request.client else "unknown"

    async def dispatch(self, request: Request, call_next) -> Response:
        client_id = self._get_client_id(request)
        now = time.time()

        with self.lock:
            # Clean old requests
            self.requests[client_id] = [
                req_time for req_time in self.requests[client_id]
                if now - req_time < self.window
            ]

            request_count = len(self.requests[client_id])

            if request_count >= self.max_requests:
                return JSONResponse(
                    status_code=429,
                    content={
                        "success": False,
                        "message": "Rate limit exceeded",
                        "detail": f"Max {self.max_requests} requests per {self.window}s"
                    },
                    headers={
                        "X-RateLimit-Limit": str(self.max_requests),
                        "X-RateLimit-Remaining": "0",
                        "Retry-After": str(self.window)
                    }
                )

            self.requests[client_id].append(now)
            remaining = self.max_requests - request_count - 1

        response = await call_next(request)
        response.headers["X-RateLimit-Limit"] = str(self.max_requests)
        response.headers["X-RateLimit-Remaining"] = str(remaining)
        return response
```

---

## 13. 🔌 WebSockets

Python

```
# app/routers/websocket.py
from fastapi import APIRouter, WebSocket, WebSocketDisconnect, Query
from typing import Dict, List
import json
from datetime import datetime, timezone

router = APIRouter(prefix="/ws", tags=["WebSocket"])


class ConnectionManager:
    """Manage WebSocket connections."""

    def __init__(self):
        # room_id → list of connections
        self.rooms: Dict[str, List[WebSocket]] = {}
        # websocket → user info
        self.user_info: Dict[WebSocket, dict] = {}

    async def connect(
        self,
        websocket: WebSocket,
        room_id: str,
        user_id: int,
        username: str
    ):
        await websocket.accept()
        if room_id not in self.rooms:
            self.rooms[room_id] = []
        self.rooms[room_id].append(websocket)
        self.user_info[websocket] = {
            "user_id": user_id,
            "username": username,
            "room_id": room_id
        }
        # Notify room that user joined
        await self.broadcast_to_room(room_id, {
            "type": "user_joined",
            "user_id": user_id,
            "username": username,
            "timestamp": datetime.now(timezone.utc).isoformat()
        }, exclude=websocket)

    def disconnect(self, websocket: WebSocket):
        user = self.user_info.pop(websocket, {})
        room_id = user.get("room_id")
        if room_id and room_id in self.rooms:
            self.rooms[room_id].remove(websocket)
            if not self.rooms[room_id]:
                del self.rooms[room_id]
        return user

    async def send_personal(self, websocket: WebSocket, data: dict):
        """Send message to one connection."""
        await websocket.send_text(json.dumps(data))

    async def broadcast_to_room(
        self,
        room_id: str,
        data: dict,
        exclude: WebSocket = None
    ):
        """Broadcast message to everyone in a room."""
        if room_id not in self.rooms:
            return
        message = json.dumps(data)
        dead = []
        for connection in self.rooms[room_id]:
            if connection == exclude:
                continue
            try:
                await connection.send_text(message)
            except Exception:
                dead.append(connection)
        for conn in dead:
            self.rooms[room_id].remove(conn)


manager = ConnectionManager()


@router.websocket("/chat/{room_id}")
async def websocket_chat(
    websocket: WebSocket,
    room_id: str,
    user_id: int = Query(...),
    username: str = Query(...)
):
    """
    WebSocket endpoint for real-time chat.
    
    Connect: ws://localhost:8000/api/v1/ws/chat/general?user_id=1&username=Alice
    
    Send: {"type": "message", "content": "Hello!"}
    Receive: {"type": "message", "from": "Alice", "content": "Hello!", "timestamp": "..."}
    """
    await manager.connect(websocket, room_id, user_id, username)

    # Send current room info to new connection
    await manager.send_personal(websocket, {
        "type": "connected",
        "room_id": room_id,
        "users_online": len(manager.rooms.get(room_id, [])),
        "message": f"Welcome to room '{room_id}', {username}!"
    })

    try:
        while True:
            # Receive message from this client
            raw = await websocket.receive_text()

            try:
                data = json.loads(raw)
            except json.JSONDecodeError:
                await manager.send_personal(websocket, {
                    "type": "error",
                    "message": "Invalid JSON"
                })
                continue

            msg_type = data.get("type", "message")

            if msg_type == "message":
                # Broadcast to everyone in room
                await manager.broadcast_to_room(room_id, {
                    "type": "message",
                    "user_id": user_id,
                    "username": username,
                    "content": data.get("content", ""),
                    "timestamp": datetime.now(timezone.utc).isoformat()
                })

            elif msg_type == "typing":
                # Broadcast typing indicator
                await manager.broadcast_to_room(room_id, {
                    "type": "typing",
                    "user_id": user_id,
                    "username": username,
                    "is_typing": data.get("is_typing", True)
                }, exclude=websocket)

            elif msg_type == "ping":
                # Keep-alive
                await manager.send_personal(websocket, {"type": "pong"})

    except WebSocketDisconnect:
        user = manager.disconnect(websocket)
        await manager.broadcast_to_room(room_id, {
            "type": "user_left",
            "user_id": user.get("user_id"),
            "username": user.get("username"),
            "timestamp": datetime.now(timezone.utc).isoformat()
        })
```

---

## 14. 🚀 Main Application

Python

```
# app/main.py
from fastapi import FastAPI, Request, status
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from sqlalchemy.exc import SQLAlchemyError
from contextlib import asynccontextmanager
import logging

from app.core.config import settings
from app.core.database import engine, Base
from app.routers import auth, users, posts, websocket
from app.middleware.logging import RequestLoggingMiddleware
from app.middleware.rate_limit import RateLimitMiddleware

# Configure logging
logging.basicConfig(
    level=logging.DEBUG if settings.DEBUG else logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)
logger = logging.getLogger(__name__)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application startup and shutdown events."""
    # STARTUP
    logger.info("Starting up...")
    # Create database tables if they don't exist
    Base.metadata.create_all(bind=engine)
    logger.info(f"Database tables ready")
    logger.info(f"Environment: {settings.ENVIRONMENT}")
    logger.info(f"Docs: http://localhost:8000/docs")

    yield  # App runs here

    # SHUTDOWN
    logger.info("Shutting down...")
    engine.dispose()
    logger.info("Database connections closed")


# Create FastAPI application
app = FastAPI(
    title=settings.APP_NAME,
    description="""
## FastAPI Blog API

A production-ready blog API built with FastAPI.

### Features
- 🔐 JWT Authentication (access + refresh tokens)
- 👥 User management with roles
- 📝 Blog post CRUD with drafts
- 📁 File uploads
- 🔌 WebSocket real-time chat
- 📖 Auto-generated OpenAPI docs

### Authentication
Use the `/auth/login` endpoint to get a token.
Click **Authorize** and enter: `Bearer your_token_here`
    """,
    version=settings.APP_VERSION,
    docs_url="/docs",
    redoc_url="/redoc",
    openapi_url="/openapi.json",
    lifespan=lifespan,
)


# ─────────────────────────────────────────
# MIDDLEWARE (order matters — first added = outermost)
# ─────────────────────────────────────────

# CORS — must be first
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"],
    allow_headers=["Authorization", "Content-Type", "X-Request-ID"],
    expose_headers=["X-Request-ID", "X-Response-Time", "X-RateLimit-Remaining"],
)

# Rate limiting
app.add_middleware(
    RateLimitMiddleware,
    max_requests=100,
    window=60
)

# Request logging
app.add_middleware(RequestLoggingMiddleware)


# ─────────────────────────────────────────
# EXCEPTION HANDLERS
# ─────────────────────────────────────────
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(
    request: Request,
    exc: RequestValidationError
) -> JSONResponse:
    """Handle Pydantic validation errors (422)."""
    errors = []
    for error in exc.errors():
        field = " → ".join(str(loc) for loc in error["loc"] if loc != "body")
        errors.append({
            "field": field or None,
            "message": error["msg"],
            "code": error["type"]
        })

    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "success": False,
            "message": "Validation failed",
            "details": errors
        }
    )


@app.exception_handler(SQLAlchemyError)
async def database_exception_handler(
    request: Request,
    exc: SQLAlchemyError
) -> JSONResponse:
    """Handle database errors."""
    logger.error(f"Database error: {exc}", exc_info=True)
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "success": False,
            "message": "Database error occurred"
        }
    )


@app.exception_handler(Exception)
async def general_exception_handler(
    request: Request,
    exc: Exception
) -> JSONResponse:
    """Catch-all exception handler."""
    logger.error(f"Unexpected error: {exc}", exc_info=True)
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "success": False,
            "message": "An unexpected error occurred"
        }
    )


# ─────────────────────────────────────────
# ROUTES
# ─────────────────────────────────────────
app.include_router(auth.router, prefix=settings.API_PREFIX)
app.include_router(users.router, prefix=settings.API_PREFIX)
app.include_router(posts.router, prefix=settings.API_PREFIX)
app.include_router(websocket.router, prefix=settings.API_PREFIX)


# ─────────────────────────────────────────
# UTILITY ENDPOINTS
# ─────────────────────────────────────────
@app.get("/health", tags=["Health"])
def health_check() -> dict:
    """Health check endpoint. Returns server status."""
    return {
        "status": "healthy",
        "app": settings.APP_NAME,
        "version": settings.APP_VERSION,
        "environment": settings.ENVIRONMENT
    }


@app.get("/", tags=["Root"])
def root() -> dict:
    """API root."""
    return {
        "message": f"Welcome to {settings.APP_NAME}",
        "version": settings.APP_VERSION,
        "docs": "/docs"
    }
```

---

## 15. ▶️ Running & Testing

Bash

```
# ─────────────────────────────────────────
# CREATE DATABASE
# ─────────────────────────────────────────
createdb -U postgres fastapi_db

# ─────────────────────────────────────────
# RUN THE APPLICATION
# ─────────────────────────────────────────
uvicorn app.main:app --reload --port 8000

# Production (multiple workers):
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4

# Or with gunicorn:
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker

# ─────────────────────────────────────────
# VISIT DOCS
# ─────────────────────────────────────────
# http://localhost:8000/docs       ← Swagger UI
# http://localhost:8000/redoc      ← ReDoc
# http://localhost:8000/health     ← Health check
```

Python

```
# tests/test_api.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.core.database import Base, get_db

# Use SQLite for testing (no PostgreSQL needed)
TEST_DATABASE_URL = "sqlite:///./test.db"
test_engine = create_engine(
    TEST_DATABASE_URL,
    connect_args={"check_same_thread": False}
)
TestSessionLocal = sessionmaker(bind=test_engine, autocommit=False, autoflush=False)


def override_get_db():
    """Override database for testing."""
    db = TestSessionLocal()
    try:
        yield db
        db.commit()
    except Exception:
        db.rollback()
        raise
    finally:
        db.close()


# Override the dependency
app.dependency_overrides[get_db] = override_get_db

# Create tables
Base.metadata.create_all(bind=test_engine)

client = TestClient(app)


# ─────────────────────────────────────────
# FIXTURES
# ─────────────────────────────────────────
@pytest.fixture
def registered_user():
    """Create and return a registered user with tokens."""
    response = client.post("/api/v1/auth/register", json={
        "name": "Test User",
        "email": "test@test.com",
        "password": "SecurePass1"
    })
    assert response.status_code == 201
    return response.json()


@pytest.fixture
def auth_headers(registered_user):
    """Return authorization headers."""
    token = registered_user["access_token"]
    return {"Authorization": f"Bearer {token}"}


# ─────────────────────────────────────────
# TESTS
# ─────────────────────────────────────────
class TestHealth:
    def test_health_check(self):
        response = client.get("/health")
        assert response.status_code == 200
        assert response.json()["status"] == "healthy"


class TestAuth:
    def test_register_success(self):
        response = client.post("/api/v1/auth/register", json={
            "name": "Alice Smith",
            "email": "alice@example.com",
            "password": "SecurePass1"
        })
        assert response.status_code == 201
        data = response.json()
        assert "access_token" in data
        assert "refresh_token" in data
        assert data["user"]["email"] == "alice@example.com"

    def test_register_duplicate_email(self, registered_user):
        response = client.post("/api/v1/auth/register", json={
            "name": "Another User",
            "email": "test@test.com",  # already registered
            "password": "SecurePass1"
        })
        assert response.status_code == 409

    def test_register_weak_password(self):
        response = client.post("/api/v1/auth/register", json={
            "name": "Bob",
            "email": "bob@test.com",
            "password": "weak"  # too short, no uppercase, no digit
        })
        assert response.status_code == 422

    def test_login_success(self, registered_user):
        response = client.post("/api/v1/auth/login", json={
            "email": "test@test.com",
            "password": "SecurePass1"
        })
        assert response.status_code == 200
        assert "access_token" in response.json()

    def test_login_wrong_password(self, registered_user):
        response = client.post("/api/v1/auth/login", json={
            "email": "test@test.com",
            "password": "WrongPassword1"
        })
        assert response.status_code == 401

    def test_get_me(self, auth_headers):
        response = client.get("/api/v1/auth/me", headers=auth_headers)
        assert response.status_code == 200
        assert response.json()["email"] == "test@test.com"

    def test_get_me_no_auth(self):
        response = client.get("/api/v1/auth/me")
        assert response.status_code == 401


class TestPosts:
    def test_create_post(self, auth_headers):
        response = client.post("/api/v1/posts", json={
            "title": "My First Post",
            "content": "This is the content of my first post.",
            "tags": ["python", "fastapi"],
            "status": "published"
        }, headers=auth_headers)
        assert response.status_code == 201
        data = response.json()
        assert data["title"] == "My First Post"
        assert data["slug"] == "my-first-post"
        assert data["status"] == "published"

    def test_create_post_no_auth(self):
        response = client.post("/api/v1/posts", json={
            "title": "Post without auth",
            "content": "Content here"
        })
        assert response.status_code == 401

    def test_list_posts(self, auth_headers):
        # Create a post first
        client.post("/api/v1/posts", json={
            "title": "Test Post",
            "content": "Test content here",
            "status": "published"
        }, headers=auth_headers)

        response = client.get("/api/v1/posts")
        assert response.status_code == 200
        data = response.json()
        assert "data" in data
        assert "pagination" in data

    def test_get_post_increments_views(self, auth_headers):
        # Create post
        create = client.post("/api/v1/posts", json={
            "title": "View Count Test",
            "content": "Content for view count test."
        }, headers=auth_headers)
        post_id = create.json()["id"]

        # Get it twice
        client.get(f"/api/v1/posts/{post_id}")
        client.get(f"/api/v1/posts/{post_id}")

        response = client.get(f"/api/v1/posts/{post_id}")
        # view count should be 3 (or more if other tests ran)
        assert response.json()["view_count"] >= 2

    def test_delete_post_not_owner(self, registered_user, auth_headers):
        # Create post as registered user
        create = client.post("/api/v1/posts", json={
            "title": "Someone's Post",
            "content": "Some content here."
        }, headers=auth_headers)
        post_id = create.json()["id"]

        # Create another user and try to delete
        other_user = client.post("/api/v1/auth/register", json={
            "name": "Other User",
            "email": "other@test.com",
            "password": "SecurePass1"
        })
        other_token = other_user.json()["access_token"]
        other_headers = {"Authorization": f"Bearer {other_token}"}

        response = client.delete(
            f"/api/v1/posts/{post_id}",
            headers=other_headers
        )
        assert response.status_code == 403


# Run with: pytest tests/ -v
```

---

## 16. 📊 OpenAPI Customization

Python

```
# Enhanced main.py additions for better API docs

from fastapi.openapi.utils import get_openapi


def custom_openapi():
    """Customize the OpenAPI schema."""
    if app.openapi_schema:
        return app.openapi_schema

    openapi_schema = get_openapi(
        title=settings.APP_NAME,
        version=settings.APP_VERSION,
        description="Production-ready FastAPI blog API",
        routes=app.routes,
    )

    # Add security scheme
    openapi_schema["components"]["securitySchemes"] = {
        "bearerAuth": {
            "type": "http",
            "scheme": "bearer",
            "bearerFormat": "JWT",
            "description": "Enter your JWT token from /auth/login"
        }
    }

    # Add global security (all endpoints require auth by default)
    # Individual endpoints can override this
    openapi_schema["security"] = [{"bearerAuth": []}]

    # Add server information
    openapi_schema["servers"] = [
        {"url": "http://localhost:8000", "description": "Development"},
        {"url": "https://api.yourapp.com", "description": "Production"},
    ]

    app.openapi_schema = openapi_schema
    return app.openapi_schema


app.openapi = custom_openapi
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                     FASTAPI ARCHITECTURE                       │
│                                                                │
│  REQUEST                                                       │
│     ↓                                                          │
│  Middleware (CORS, Rate Limit, Logging)                        │
│     ↓                                                          │
│  Router (path + method matching)                               │
│     ↓                                                          │
│  Dependencies (auth, DB session, permissions)                  │
│     ↓                                                          │
│  Route Handler (controller)                                    │
│     ↓                                                          │
│  Service (business logic)                                      │
│     ↓                                                          │
│  Repository (database queries)                                 │
│     ↓                                                          │
│  Database (PostgreSQL)                                         │
│     ↑                                                          │
│  Pydantic validates at every boundary                          │
│                                                                │
│  KEY COMPONENTS:                                               │
│  FastAPI app     → main entry point, middleware, routes       │
│  Pydantic schemas → request/response validation               │
│  SQLAlchemy ORM  → database models                            │
│  Depends()       → dependency injection                        │
│  APIRouter       → route grouping                             │
│  BackgroundTasks → async work after response                  │
│  WebSocket       → real-time bidirectional                    │
│                                                                │
│  ENDPOINTS BUILT:                                              │
│  POST /auth/register     POST /auth/login                     │
│  POST /auth/refresh      GET  /auth/me                        │
│  GET  /users             GET  /users/{id}                     │
│  PATCH /users/{id}       DELETE /users/{id}                   │
│  GET  /posts             POST /posts                          │
│  GET  /posts/{id}        PATCH /posts/{id}                    │
│  DELETE /posts/{id}      POST /posts/{id}/publish             │
│  POST /files/upload      WS   /ws/chat/{room}                 │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is uvicorn and why do you need it to run FastAPI?
2.  What is the difference between @app.get and @router.get?
3.  What does Depends() do? Give a real example.
4.  What is the difference between path params, query params,
    and request body?
5.  How does FastAPI validate request data automatically?
6.  What is the purpose of response_model?
7.  What is the lifespan context manager for?
8.  How do you make an endpoint require authentication?
9.  What does BackgroundTasks do?
10. What is the Repository pattern and why use it?
11. How do you handle validation errors globally?
12. What is the difference between 401 and 403 in FastAPI context?
13. How does OAuth2PasswordBearer work?
14. What does model_config = {"from_attributes": True} do?
15. How do you test FastAPI endpoints?
```

---

## 🛠️ Practice Exercises

Python

```
# Exercise 1: Add Comments to Posts
# Extend the API with a comments system:
# POST   /posts/{post_id}/comments   → create comment
# GET    /posts/{post_id}/comments   → list comments
# PATCH  /comments/{id}              → update comment (author only)
# DELETE /comments/{id}              → delete (author or admin)
# POST   /comments/{id}/replies      → reply to comment
# Include: pagination, author info in response

# Exercise 2: Like System
# Add likes to posts:
# POST   /posts/{post_id}/like       → like a post (authenticated)
# DELETE /posts/{post_id}/like       → unlike a post
# GET    /posts/{post_id}/likes      → get like count and status
# Rules: can only like once, track who liked

# Exercise 3: User Following
# Build a follow system:
# POST   /users/{user_id}/follow     → follow a user
# DELETE /users/{user_id}/follow     → unfollow
# GET    /users/{user_id}/followers  → list followers
# GET    /users/{user_id}/following  → list following
# GET    /feed                       → posts by followed users

# Exercise 4: Search Endpoint
# Build a unified search endpoint:
# GET /search?q=python&type=posts,users
# Returns: {
#     "posts": [...],
#     "users": [...],
#     "total_results": 25
# }
# Should search post title/content and user name/email
# Paginate results
# Only show published posts for non-auth users

# Exercise 5: API Key Authentication
# Add API key authentication as an alternative to JWT:
# POST /api-keys          → create API key (authenticated)
# GET  /api-keys          → list my API keys
# DELETE /api-keys/{id}  → delete API key
# Usage: X-API-Key: my-api-key header
# Create a dependency that accepts both JWT and API key
```

---

## 🎉 Phase 4.2 Complete!

**You have built a production-ready FastAPI application with:**

text

```
✅ Application structure (routers, services, repos, schemas)
✅ Pydantic schemas with validation
✅ SQLAlchemy ORM models and repositories
✅ JWT authentication (access + refresh tokens)
✅ Role-based access control
✅ Dependency injection system
✅ CORS middleware
✅ Rate limiting middleware
✅ Request logging middleware
✅ Global exception handling
✅ File upload/download
✅ WebSocket real-time chat
✅ Background tasks
✅ Auto-generated API documentation
✅ Complete test suite
✅ Health check endpoint
✅ Lifespan events (startup/shutdown)
```