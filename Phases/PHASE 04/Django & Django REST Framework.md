```
FastAPI = modern, fast, async-first
Django  = mature, feature-rich, "batteries included"

Why learn Django after FastAPI?

Market reality:
  - Django powers 40%+ of Python backend jobs
  - Huge existing codebases (10+ years of Django apps)
  - Many companies: Instagram, Pinterest, Disqus, Mozilla
  - Django REST Framework = most used Python REST library

When Django wins:
  ✅ Built-in admin panel (saves weeks of work)
  ✅ Built-in ORM (no SQLAlchemy needed)
  ✅ Built-in auth system
  ✅ Massive ecosystem of packages
  ✅ More structure enforced (easier for large teams)
  ✅ Mature, stable, well-documented

When FastAPI wins:
  ✅ Performance critical
  ✅ Pure async needed
  ✅ Microservices
  ✅ You control the structure

Reality: Know both. You WILL encounter both in your career.
```

---

## Setup

Bash

```
cd ~/projects
mkdir django_project
cd django_project
python3 -m venv venv
source venv/bin/activate

pip install django djangorestframework \
    djangorestframework-simplejwt django-filter \
    psycopg2-binary python-dotenv Pillow \
    django-cors-headers celery redis

# Create Django project
django-admin startproject config .

# Create apps
python manage.py startapp users
python manage.py startapp posts

# Project structure created:
# config/          ← project settings
#   settings.py
#   urls.py
#   wsgi.py
#   asgi.py
# users/           ← users app
#   models.py
#   views.py
#   serializers.py (we'll create this)
#   urls.py (we'll create this)
# posts/           ← posts app

code .
```

---

## 1. 🏗️ Django Architecture — Understanding the Basics

text

```
Django follows MVT pattern (Model-View-Template)
For APIs (DRF): MC pattern (Model-Serializer-ViewSet)

┌─────────────────────────────────────────────────────────┐
│  HTTP Request                                           │
│       ↓                                                 │
│  config/urls.py  → routes to app urls                  │
│       ↓                                                 │
│  app/urls.py     → routes to specific view             │
│       ↓                                                 │
│  View/ViewSet    → handles the request                  │
│       ↓                                                 │
│  Serializer      → validates input, formats output      │
│       ↓                                                 │
│  Model           → talks to database                   │
│       ↓                                                 │
│  Database (PostgreSQL)                                  │
└─────────────────────────────────────────────────────────┘

Django vs FastAPI architecture comparison:
  FastAPI Router  = Django URLconf
  FastAPI Schema  = DRF Serializer
  FastAPI Service = DRF View/ViewSet
  FastAPI Model   = Django Model
```

---

## 2. ⚙️ Configuration

Python

```
# config/settings.py
from pathlib import Path
from datetime import timedelta
import os
from dotenv import load_dotenv

load_dotenv()

BASE_DIR = Path(__file__).resolve().parent.parent
SECRET_KEY = os.getenv("SECRET_KEY", "django-insecure-change-in-production")
DEBUG = os.getenv("DEBUG", "True").lower() == "true"
ALLOWED_HOSTS = os.getenv("ALLOWED_HOSTS", "localhost,127.0.0.1").split(",")

# ─────────────────────────────────────────
# INSTALLED APPS
# ─────────────────────────────────────────
INSTALLED_APPS = [
    # Django built-ins
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",

    # Third-party
    "rest_framework",                  # Django REST Framework
    "rest_framework_simplejwt",        # JWT authentication
    "django_filters",                  # query filtering
    "corsheaders",                     # CORS support

    # Our apps
    "users",
    "posts",
]

# ─────────────────────────────────────────
# MIDDLEWARE
# ─────────────────────────────────────────
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "corsheaders.middleware.CorsMiddleware",      # CORS (must be high up)
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]

ROOT_URLCONF = "config.urls"

# ─────────────────────────────────────────
# DATABASE
# ─────────────────────────────────────────
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.getenv("DB_NAME", "django_blog"),
        "USER": os.getenv("DB_USER", "myapp_user"),
        "PASSWORD": os.getenv("DB_PASSWORD", "myapp_password"),
        "HOST": os.getenv("DB_HOST", "localhost"),
        "PORT": os.getenv("DB_PORT", "5432"),
        "OPTIONS": {
            "connect_timeout": 10,
        }
    }
}

# ─────────────────────────────────────────
# CUSTOM USER MODEL
# ─────────────────────────────────────────
AUTH_USER_MODEL = "users.User"   # use our custom User model

# ─────────────────────────────────────────
# DJANGO REST FRAMEWORK
# ─────────────────────────────────────────
REST_FRAMEWORK = {
    # Default authentication
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
        "rest_framework.authentication.SessionAuthentication",  # for admin
    ],

    # Default permission
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],

    # Pagination
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 20,

    # Filtering
    "DEFAULT_FILTER_BACKENDS": [
        "django_filters.rest_framework.DjangoFilterBackend",
        "rest_framework.filters.SearchFilter",
        "rest_framework.filters.OrderingFilter",
    ],

    # Throttling (rate limiting)
    "DEFAULT_THROTTLE_CLASSES": [
        "rest_framework.throttling.AnonRateThrottle",
        "rest_framework.throttling.UserRateThrottle",
    ],
    "DEFAULT_THROTTLE_RATES": {
        "anon": "100/hour",
        "user": "1000/hour",
    },

    # Exception handling
    "EXCEPTION_HANDLER": "config.exceptions.custom_exception_handler",

    # Response format
    "DEFAULT_RENDERER_CLASSES": [
        "rest_framework.renderers.JSONRenderer",
    ],
}

# ─────────────────────────────────────────
# JWT SETTINGS
# ─────────────────────────────────────────
SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=30),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=7),
    "ROTATE_REFRESH_TOKENS": True,         # rotate on use
    "BLACKLIST_AFTER_ROTATION": True,      # blacklist old refresh token
    "UPDATE_LAST_LOGIN": True,
    "ALGORITHM": "HS256",
    "SIGNING_KEY": SECRET_KEY,
    "AUTH_HEADER_TYPES": ("Bearer",),
    "AUTH_HEADER_NAME": "HTTP_AUTHORIZATION",
    "USER_ID_FIELD": "id",
    "USER_ID_CLAIM": "user_id",
}

# ─────────────────────────────────────────
# CORS
# ─────────────────────────────────────────
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://localhost:8080",
]
CORS_ALLOW_CREDENTIALS = True

# ─────────────────────────────────────────
# STATIC & MEDIA FILES
# ─────────────────────────────────────────
STATIC_URL = "/static/"
STATIC_ROOT = BASE_DIR / "staticfiles"
MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "media"

# ─────────────────────────────────────────
# CELERY (background tasks)
# ─────────────────────────────────────────
CELERY_BROKER_URL = os.getenv("REDIS_URL", "redis://localhost:6379/0")
CELERY_RESULT_BACKEND = os.getenv("REDIS_URL", "redis://localhost:6379/0")

# ─────────────────────────────────────────
# EMAIL
# ─────────────────────────────────────────
EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"
DEFAULT_FROM_EMAIL = "noreply@myapp.com"
```

---

## 3. 🗄️ Django Models

Python

```
# users/models.py
from django.db import models
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
from django.utils import timezone
from django.utils.translation import gettext_lazy as _


class UserManager(BaseUserManager):
    """Custom manager for User model with email as username."""

    def create_user(self, email: str, password: str = None, **extra_fields):
        if not email:
            raise ValueError("Email is required")
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)     # handles hashing!
        user.save(using=self._db)
        return user

    def create_superuser(self, email: str, password: str, **extra_fields):
        extra_fields.setdefault("is_staff", True)
        extra_fields.setdefault("is_superuser", True)
        extra_fields.setdefault("role", "admin")

        if not extra_fields.get("is_staff"):
            raise ValueError("Superuser must have is_staff=True")
        if not extra_fields.get("is_superuser"):
            raise ValueError("Superuser must have is_superuser=True")

        return self.create_user(email, password, **extra_fields)


class User(AbstractBaseUser, PermissionsMixin):
    """
    Custom User model.
    Uses email instead of username.

    Django's AbstractBaseUser gives us:
      - password (hashed)
      - last_login
      - is_active
      - password hashing methods

    PermissionsMixin gives us:
      - is_superuser
      - groups
      - user_permissions
    """

    ROLE_CHOICES = [
        ("user", "User"),
        ("moderator", "Moderator"),
        ("admin", "Admin"),
    ]

    # Core fields
    email = models.EmailField(
        _("email address"),
        unique=True,
        db_index=True,
    )
    name = models.CharField(_("full name"), max_length=100)
    bio = models.TextField(_("bio"), blank=True, null=True)
    website = models.URLField(_("website"), blank=True, null=True)
    avatar = models.ImageField(
        upload_to="avatars/",
        null=True,
        blank=True
    )

    # Status
    role = models.CharField(
        max_length=20,
        choices=ROLE_CHOICES,
        default="user",
        db_index=True
    )
    is_active = models.BooleanField(default=True)
    is_verified = models.BooleanField(default=False)

    # Required for admin
    is_staff = models.BooleanField(default=False)

    # Timestamps
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    deleted_at = models.DateTimeField(null=True, blank=True)

    objects = UserManager()

    USERNAME_FIELD = "email"         # login with email
    REQUIRED_FIELDS = ["name"]       # required for createsuperuser

    class Meta:
        verbose_name = _("user")
        verbose_name_plural = _("users")
        ordering = ["-created_at"]
        indexes = [
            models.Index(fields=["email"]),
            models.Index(fields=["role", "is_active"]),
        ]

    def __str__(self):
        return f"{self.name} <{self.email}>"

    @property
    def is_admin(self) -> bool:
        return self.role == "admin" or self.is_superuser

    @property
    def is_deleted(self) -> bool:
        return self.deleted_at is not None

    def soft_delete(self):
        self.deleted_at = timezone.now()
        self.is_active = False
        self.save(update_fields=["deleted_at", "is_active"])
```

Python

```
# posts/models.py
from django.db import models
from django.utils import timezone
from django.utils.text import slugify
from django.conf import settings
import re


class Tag(models.Model):
    """Tag model for categorizing posts."""
    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["name"]

    def __str__(self):
        return self.name

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)


class Post(models.Model):
    """Blog post model."""

    STATUS_DRAFT = "draft"
    STATUS_PUBLISHED = "published"
    STATUS_ARCHIVED = "archived"

    STATUS_CHOICES = [
        (STATUS_DRAFT, "Draft"),
        (STATUS_PUBLISHED, "Published"),
        (STATUS_ARCHIVED, "Archived"),
    ]

    # Core fields
    title = models.CharField(max_length=500)
    content = models.TextField()
    slug = models.SlugField(max_length=500, unique=True, blank=True)
    status = models.CharField(
        max_length=20,
        choices=STATUS_CHOICES,
        default=STATUS_DRAFT,
        db_index=True
    )

    # Author relationship
    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,         # delete posts if user deleted
        related_name="posts",             # user.posts.all()
        db_index=True
    )

    # Many-to-many with Tag
    tags = models.ManyToManyField(
        Tag,
        related_name="posts",
        blank=True
    )

    # Metadata
    view_count = models.PositiveIntegerField(default=0)
    like_count = models.PositiveIntegerField(default=0)
    featured_image = models.ImageField(
        upload_to="posts/images/",
        null=True,
        blank=True
    )

    # Timestamps
    published_at = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    deleted_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        ordering = ["-created_at"]
        indexes = [
            models.Index(fields=["author", "status"]),
            models.Index(fields=["status", "published_at"]),
            models.Index(fields=["slug"]),
        ]

    def __str__(self):
        return self.title

    def save(self, *args, **kwargs):
        # Auto-generate slug from title
        if not self.slug:
            self.slug = self._generate_unique_slug()

        # Set published_at when publishing
        if self.status == self.STATUS_PUBLISHED and not self.published_at:
            self.published_at = timezone.now()

        super().save(*args, **kwargs)

    def _generate_unique_slug(self) -> str:
        """Generate a unique slug from the title."""
        base_slug = slugify(self.title)
        slug = base_slug
        counter = 1
        while Post.objects.filter(slug=slug).exclude(pk=self.pk).exists():
            slug = f"{base_slug}-{counter}"
            counter += 1
        return slug

    def increment_views(self):
        """Atomically increment view count."""
        Post.objects.filter(pk=self.pk).update(
            view_count=models.F("view_count") + 1
        )

    @property
    def is_published(self) -> bool:
        return self.status == self.STATUS_PUBLISHED

    def soft_delete(self):
        self.deleted_at = timezone.now()
        self.save(update_fields=["deleted_at"])


class Comment(models.Model):
    """Comment on a post with nested replies support."""

    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name="comments"
    )
    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="comments"
    )
    parent = models.ForeignKey(
        "self",
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name="replies"
    )
    content = models.TextField()
    is_edited = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ["created_at"]

    def __str__(self):
        return f"Comment by {self.author} on {self.post}"


class Like(models.Model):
    """Track post likes — one per user per post."""

    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="likes"
    )
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name="likes"
    )
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ("user", "post")   # one like per user per post
        ordering = ["-created_at"]

    def __str__(self):
        return f"{self.user} likes {self.post}"
```

---

## 4. 📝 Django Serializers

Python

```
# users/serializers.py
from rest_framework import serializers
from django.contrib.auth import get_user_model
from django.contrib.auth.password_validation import validate_password

User = get_user_model()


class UserRegistrationSerializer(serializers.ModelSerializer):
    """Serializer for user registration."""
    password = serializers.CharField(
        write_only=True,
        min_length=8,
        style={"input_type": "password"}
    )
    password_confirm = serializers.CharField(
        write_only=True,
        style={"input_type": "password"}
    )

    class Meta:
        model = User
        fields = ["id", "name", "email", "password", "password_confirm"]
        read_only_fields = ["id"]

    def validate_email(self, value: str) -> str:
        """Validate email is unique."""
        email = value.lower().strip()
        if User.objects.filter(email=email).exists():
            raise serializers.ValidationError(
                "A user with this email already exists."
            )
        return email

    def validate_name(self, value: str) -> str:
        value = value.strip()
        if len(value) < 2:
            raise serializers.ValidationError(
                "Name must be at least 2 characters."
            )
        return value.title()

    def validate(self, attrs: dict) -> dict:
        """Cross-field validation."""
        if attrs["password"] != attrs["password_confirm"]:
            raise serializers.ValidationError({
                "password_confirm": "Passwords do not match."
            })
        # Use Django's password validators
        try:
            validate_password(attrs["password"])
        except Exception as e:
            raise serializers.ValidationError({"password": list(e.messages)})

        return attrs

    def create(self, validated_data: dict) -> User:
        """Create user with hashed password."""
        validated_data.pop("password_confirm")
        password = validated_data.pop("password")
        user = User(**validated_data)
        user.set_password(password)   # hashes password
        user.save()
        return user


class UserPublicSerializer(serializers.ModelSerializer):
    """Public user data — safe to expose."""
    post_count = serializers.SerializerMethodField()

    class Meta:
        model = User
        fields = [
            "id", "name", "bio", "website",
            "avatar", "role", "post_count", "created_at"
        ]

    def get_post_count(self, obj) -> int:
        return obj.posts.filter(
            status="published",
            deleted_at__isnull=True
        ).count()


class UserDetailSerializer(serializers.ModelSerializer):
    """Full user data — for authenticated user's own profile."""
    post_count = serializers.SerializerMethodField()

    class Meta:
        model = User
        fields = [
            "id", "name", "email", "bio", "website",
            "avatar", "role", "is_active", "is_verified",
            "post_count", "created_at", "updated_at"
        ]
        read_only_fields = [
            "id", "email", "role", "is_active",
            "is_verified", "created_at"
        ]

    def get_post_count(self, obj) -> int:
        return obj.posts.filter(status="published").count()


class UserUpdateSerializer(serializers.ModelSerializer):
    """Update user profile."""
    class Meta:
        model = User
        fields = ["name", "bio", "website", "avatar"]

    def validate_name(self, value: str) -> str:
        if len(value.strip()) < 2:
            raise serializers.ValidationError("Name too short")
        return value.strip().title()


class ChangePasswordSerializer(serializers.Serializer):
    """Change user password."""
    current_password = serializers.CharField(required=True)
    new_password = serializers.CharField(required=True, min_length=8)
    confirm_password = serializers.CharField(required=True)

    def validate(self, attrs: dict) -> dict:
        if attrs["new_password"] != attrs["confirm_password"]:
            raise serializers.ValidationError({
                "confirm_password": "Passwords do not match."
            })
        try:
            validate_password(attrs["new_password"])
        except Exception as e:
            raise serializers.ValidationError({"new_password": list(e.messages)})
        return attrs
```

Python

```
# posts/serializers.py
from rest_framework import serializers
from django.db import transaction
from .models import Post, Comment, Like, Tag
from users.serializers import UserPublicSerializer


class TagSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tag
        fields = ["id", "name", "slug"]
        read_only_fields = ["id", "slug"]


class CommentSerializer(serializers.ModelSerializer):
    """Serializer for comments."""
    author = UserPublicSerializer(read_only=True)
    replies = serializers.SerializerMethodField()
    reply_count = serializers.SerializerMethodField()

    class Meta:
        model = Comment
        fields = [
            "id", "content", "author", "parent",
            "replies", "reply_count", "is_edited",
            "created_at", "updated_at"
        ]
        read_only_fields = ["id", "author", "is_edited", "created_at"]

    def get_replies(self, obj) -> list:
        """Get nested replies (one level deep)."""
        if obj.parent is None:  # only top-level comments get replies
            replies = obj.replies.select_related("author").all()
            return CommentSerializer(replies, many=True, context=self.context).data
        return []

    def get_reply_count(self, obj) -> int:
        return obj.replies.count()


class PostListSerializer(serializers.ModelSerializer):
    """Lightweight serializer for listing posts."""
    author = UserPublicSerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    is_liked = serializers.SerializerMethodField()

    class Meta:
        model = Post
        fields = [
            "id", "title", "slug", "status",
            "author", "tags", "view_count", "like_count",
            "is_liked", "published_at", "created_at"
        ]

    def get_is_liked(self, obj) -> bool:
        """Check if current user has liked this post."""
        request = self.context.get("request")
        if not request or not request.user.is_authenticated:
            return False
        return obj.likes.filter(user=request.user).exists()


class PostDetailSerializer(serializers.ModelSerializer):
    """Full post serializer with all fields."""
    author = UserPublicSerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    tag_names = serializers.ListField(
        child=serializers.CharField(max_length=50),
        write_only=True,
        required=False,
        default=list
    )
    comments = serializers.SerializerMethodField()
    comment_count = serializers.SerializerMethodField()
    is_liked = serializers.SerializerMethodField()

    class Meta:
        model = Post
        fields = [
            "id", "title", "content", "slug", "status",
            "author", "tags", "tag_names",
            "view_count", "like_count", "is_liked",
            "featured_image", "comment_count", "comments",
            "published_at", "created_at", "updated_at"
        ]
        read_only_fields = [
            "id", "slug", "author", "tags",
            "view_count", "like_count",
            "published_at", "created_at", "updated_at"
        ]

    def get_comments(self, obj) -> list:
        """Get top-level comments only."""
        comments = obj.comments.filter(
            parent__isnull=True
        ).select_related("author").prefetch_related("replies__author")
        return CommentSerializer(
            comments, many=True, context=self.context
        ).data

    def get_comment_count(self, obj) -> int:
        return obj.comments.count()

    def get_is_liked(self, obj) -> bool:
        request = self.context.get("request")
        if not request or not request.user.is_authenticated:
            return False
        return obj.likes.filter(user=request.user).exists()

    def validate_title(self, value: str) -> str:
        return value.strip()

    def validate_tag_names(self, value: list) -> list:
        """Validate and normalize tag names."""
        return [tag.lower().strip() for tag in value if tag.strip()][:10]

    @transaction.atomic
    def create(self, validated_data: dict) -> Post:
        tag_names = validated_data.pop("tag_names", [])
        post = Post.objects.create(**validated_data)
        self._set_tags(post, tag_names)
        return post

    @transaction.atomic
    def update(self, instance: Post, validated_data: dict) -> Post:
        tag_names = validated_data.pop("tag_names", None)
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        instance.save()
        if tag_names is not None:
            self._set_tags(instance, tag_names)
        return instance

    def _set_tags(self, post: Post, tag_names: list) -> None:
        """Create tags if needed and associate with post."""
        tags = []
        for name in tag_names:
            tag, _ = Tag.objects.get_or_create(
                name=name,
                defaults={"slug": name.replace(" ", "-")}
            )
            tags.append(tag)
        post.tags.set(tags)


class PostCreateSerializer(serializers.ModelSerializer):
    """Simplified serializer for creating posts."""
    tag_names = serializers.ListField(
        child=serializers.CharField(max_length=50),
        required=False,
        default=list
    )

    class Meta:
        model = Post
        fields = ["title", "content", "status", "tag_names"]

    def validate_status(self, value: str) -> str:
        allowed = {"draft", "published"}
        if value not in allowed:
            raise serializers.ValidationError(
                f"Status must be one of: {allowed}"
            )
        return value

    @transaction.atomic
    def create(self, validated_data: dict) -> Post:
        tag_names = validated_data.pop("tag_names", [])
        # Author set from request.user in view
        post = Post.objects.create(**validated_data)
        for name in tag_names:
            tag, _ = Tag.objects.get_or_create(name=name.lower().strip())
            post.tags.add(tag)
        return post
```

---

## 5. 🔐 Custom Permissions

Python

```
# config/permissions.py
from rest_framework import permissions
from posts.models import Post


class IsOwnerOrAdmin(permissions.BasePermission):
    """
    Object-level permission.
    Allow access to object's owner or admin users.
    """
    message = "You do not have permission to perform this action."

    def has_object_permission(self, request, view, obj) -> bool:
        # Read permissions for safe methods
        if request.method in permissions.SAFE_METHODS:
            return True

        # Admin can do anything
        if request.user.is_admin:
            return True

        # Check object ownership
        if hasattr(obj, "author"):
            return obj.author == request.user
        if hasattr(obj, "user"):
            return obj.user == request.user
        # User model — check if it's the same user
        return obj == request.user


class IsAdminUser(permissions.BasePermission):
    """Allow access only to admin users."""

    def has_permission(self, request, view) -> bool:
        return (
            request.user and
            request.user.is_authenticated and
            request.user.is_admin
        )


class IsAuthorOrReadOnly(permissions.BasePermission):
    """Allow read to everyone, write only to author."""

    def has_permission(self, request, view) -> bool:
        if request.method in permissions.SAFE_METHODS:
            return True
        return request.user and request.user.is_authenticated

    def has_object_permission(self, request, view, obj) -> bool:
        if request.method in permissions.SAFE_METHODS:
            return True
        if request.user.is_admin:
            return True
        return obj.author == request.user


class IsVerifiedUser(permissions.BasePermission):
    """Only verified users can perform this action."""
    message = "Email verification required."

    def has_permission(self, request, view) -> bool:
        return (
            request.user and
            request.user.is_authenticated and
            request.user.is_verified
        )
```

---

## 6. 🎭 Views & ViewSets

Python

```
# users/views.py
from rest_framework import generics, status, permissions
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.viewsets import ModelViewSet
from rest_framework.views import APIView
from rest_framework_simplejwt.tokens import RefreshToken
from django.contrib.auth import get_user_model
from django.utils import timezone

from .serializers import (
    UserRegistrationSerializer,
    UserDetailSerializer,
    UserPublicSerializer,
    UserUpdateSerializer,
    ChangePasswordSerializer,
)
from config.permissions import IsOwnerOrAdmin, IsAdminUser

User = get_user_model()


class RegisterView(generics.CreateAPIView):
    """
    Register a new user.
    Returns JWT tokens on success.

    POST /api/auth/register/
    """
    serializer_class = UserRegistrationSerializer
    permission_classes = [permissions.AllowAny]   # no auth needed

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()

        # Generate JWT tokens
        refresh = RefreshToken.for_user(user)
        return Response({
            "user": UserDetailSerializer(user, context={"request": request}).data,
            "access_token": str(refresh.access_token),
            "refresh_token": str(refresh),
        }, status=status.HTTP_201_CREATED)


class LoginView(APIView):
    """
    Login with email and password.
    Returns JWT tokens.

    POST /api/auth/login/
    """
    permission_classes = [permissions.AllowAny]

    def post(self, request):
        from django.contrib.auth import authenticate

        email = request.data.get("email", "").lower().strip()
        password = request.data.get("password", "")

        if not email or not password:
            return Response(
                {"detail": "Email and password are required."},
                status=status.HTTP_400_BAD_REQUEST
            )

        user = authenticate(request, username=email, password=password)

        if not user:
            return Response(
                {"detail": "Invalid email or password."},
                status=status.HTTP_401_UNAUTHORIZED
            )

        if not user.is_active:
            return Response(
                {"detail": "Account is deactivated."},
                status=status.HTTP_403_FORBIDDEN
            )

        refresh = RefreshToken.for_user(user)
        return Response({
            "user": UserDetailSerializer(user, context={"request": request}).data,
            "access_token": str(refresh.access_token),
            "refresh_token": str(refresh),
        })


class LogoutView(APIView):
    """
    Logout — blacklist the refresh token.

    POST /api/auth/logout/
    """
    def post(self, request):
        try:
            refresh_token = request.data.get("refresh_token")
            if refresh_token:
                token = RefreshToken(refresh_token)
                token.blacklist()   # invalidate this token
            return Response(status=status.HTTP_204_NO_CONTENT)
        except Exception:
            return Response(status=status.HTTP_204_NO_CONTENT)


class MeView(generics.RetrieveUpdateAPIView):
    """
    Get or update the current user's profile.

    GET  /api/auth/me/
    PATCH /api/auth/me/
    """
    def get_serializer_class(self):
        if self.request.method in ["PUT", "PATCH"]:
            return UserUpdateSerializer
        return UserDetailSerializer

    def get_object(self):
        return self.request.user


class ChangePasswordView(APIView):
    """
    Change the current user's password.

    POST /api/auth/change-password/
    """
    def post(self, request):
        serializer = ChangePasswordSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)

        user = request.user
        if not user.check_password(serializer.validated_data["current_password"]):
            return Response(
                {"current_password": ["Current password is incorrect."]},
                status=status.HTTP_400_BAD_REQUEST
            )

        user.set_password(serializer.validated_data["new_password"])
        user.save(update_fields=["password"])
        return Response(
            {"detail": "Password changed successfully."},
            status=status.HTTP_200_OK
        )


class UserViewSet(ModelViewSet):
    """
    ViewSet for user management.

    GET    /api/users/        → list (admin only)
    GET    /api/users/{id}/   → detail
    PATCH  /api/users/{id}/   → update
    DELETE /api/users/{id}/   → delete
    POST   /api/users/{id}/activate/   → activate user
    POST   /api/users/{id}/deactivate/ → deactivate user
    """
    queryset = User.objects.filter(deleted_at__isnull=True)
    permission_classes = [permissions.IsAuthenticated]

    def get_serializer_class(self):
        if self.request.user.is_admin:
            return UserDetailSerializer
        return UserPublicSerializer

    def get_permissions(self):
        """Different permissions for different actions."""
        if self.action == "list":
            return [IsAdminUser()]
        if self.action in ["update", "partial_update", "destroy"]:
            return [permissions.IsAuthenticated(), IsOwnerOrAdmin()]
        return [permissions.IsAuthenticated()]

    def get_queryset(self):
        queryset = super().get_queryset()
        # Non-admins can only see active users
        if not self.request.user.is_admin:
            queryset = queryset.filter(is_active=True)
        return queryset

    def destroy(self, request, *args, **kwargs):
        """Soft delete instead of hard delete."""
        user = self.get_object()
        user.soft_delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

    @action(detail=True, methods=["post"], permission_classes=[IsAdminUser])
    def activate(self, request, pk=None):
        """Activate a user account."""
        user = self.get_object()
        user.is_active = True
        user.deleted_at = None
        user.save(update_fields=["is_active", "deleted_at"])
        return Response({"detail": "User activated."})

    @action(detail=True, methods=["post"], permission_classes=[IsAdminUser])
    def deactivate(self, request, pk=None):
        """Deactivate a user account."""
        user = self.get_object()
        user.is_active = False
        user.save(update_fields=["is_active"])
        return Response({"detail": "User deactivated."})
```

Python

```
# posts/views.py
from rest_framework import generics, status, permissions, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.viewsets import ModelViewSet
from rest_framework.views import APIView
from django_filters.rest_framework import DjangoFilterBackend
from django.db.models import F
import django_filters

from .models import Post, Comment, Like, Tag
from .serializers import (
    PostListSerializer,
    PostDetailSerializer,
    PostCreateSerializer,
    CommentSerializer,
    TagSerializer,
)
from config.permissions import IsAuthorOrReadOnly, IsAdminUser


# ─────────────────────────────────────────
# FILTERS
# ─────────────────────────────────────────
class PostFilter(django_filters.FilterSet):
    """Custom filter for posts."""
    status = django_filters.CharFilter(field_name="status")
    author_id = django_filters.NumberFilter(field_name="author__id")
    tag = django_filters.CharFilter(field_name="tags__slug")
    published_after = django_filters.DateTimeFilter(
        field_name="published_at",
        lookup_expr="gte"
    )
    published_before = django_filters.DateTimeFilter(
        field_name="published_at",
        lookup_expr="lte"
    )

    class Meta:
        model = Post
        fields = ["status", "author_id", "tag"]


# ─────────────────────────────────────────
# POSTS
# ─────────────────────────────────────────
class PostViewSet(ModelViewSet):
    """
    ViewSet for blog posts.

    GET    /api/posts/          → list posts
    POST   /api/posts/          → create post
    GET    /api/posts/{id}/     → get post
    PATCH  /api/posts/{id}/     → update post
    DELETE /api/posts/{id}/     → delete post
    POST   /api/posts/{id}/publish/    → publish
    POST   /api/posts/{id}/like/       → like/unlike
    GET    /api/posts/{id}/comments/   → list comments
    POST   /api/posts/{id}/comments/   → add comment
    """
    permission_classes = [IsAuthorOrReadOnly]
    filter_backends = [
        DjangoFilterBackend,
        filters.SearchFilter,
        filters.OrderingFilter,
    ]
    filterset_class = PostFilter
    search_fields = ["title", "content", "author__name"]
    ordering_fields = ["created_at", "published_at", "view_count", "like_count"]
    ordering = ["-created_at"]

    def get_queryset(self):
        """
        Public: published posts only.
        Authenticated: own posts (any status) + published posts.
        Admin: all posts.
        """
        queryset = Post.objects.filter(
            deleted_at__isnull=True
        ).select_related("author").prefetch_related("tags")

        user = self.request.user

        if not user.is_authenticated:
            return queryset.filter(status=Post.STATUS_PUBLISHED)

        if user.is_admin:
            return queryset

        # Own posts + published posts from others
        from django.db.models import Q
        return queryset.filter(
            Q(author=user) | Q(status=Post.STATUS_PUBLISHED)
        )

    def get_serializer_class(self):
        if self.action == "list":
            return PostListSerializer
        if self.action == "create":
            return PostCreateSerializer
        return PostDetailSerializer

    def perform_create(self, serializer):
        """Set author to current user on creation."""
        serializer.save(author=self.request.user)

    def retrieve(self, request, *args, **kwargs):
        """Get post and increment view count."""
        instance = self.get_object()
        instance.increment_views()
        serializer = self.get_serializer(instance)
        return Response(serializer.data)

    def destroy(self, request, *args, **kwargs):
        """Soft delete."""
        post = self.get_object()
        post.soft_delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

    @action(detail=True, methods=["post"])
    def publish(self, request, pk=None):
        """Publish a draft post."""
        post = self.get_object()

        if post.author != request.user and not request.user.is_admin:
            return Response(
                {"detail": "Permission denied."},
                status=status.HTTP_403_FORBIDDEN
            )

        if post.status == Post.STATUS_PUBLISHED:
            return Response(
                {"detail": "Post is already published."},
                status=status.HTTP_400_BAD_REQUEST
            )

        post.status = Post.STATUS_PUBLISHED
        post.save()

        return Response(PostDetailSerializer(
            post, context={"request": request}
        ).data)

    @action(detail=True, methods=["post", "delete"])
    def like(self, request, pk=None):
        """Like or unlike a post."""
        post = self.get_object()

        if request.method == "POST":
            like, created = Like.objects.get_or_create(
                user=request.user,
                post=post
            )
            if created:
                # Atomic increment
                Post.objects.filter(pk=post.pk).update(
                    like_count=F("like_count") + 1
                )
                return Response(
                    {"liked": True, "like_count": post.like_count + 1},
                    status=status.HTTP_201_CREATED
                )
            return Response(
                {"liked": True, "detail": "Already liked."},
                status=status.HTTP_200_OK
            )

        else:  # DELETE = unlike
            deleted, _ = Like.objects.filter(
                user=request.user, post=post
            ).delete()

            if deleted:
                Post.objects.filter(pk=post.pk).update(
                    like_count=F("like_count") - 1
                )
                return Response(status=status.HTTP_204_NO_CONTENT)

            return Response(
                {"detail": "Not liked."},
                status=status.HTTP_400_BAD_REQUEST
            )

    @action(detail=True, methods=["get", "post"])
    def comments(self, request, pk=None):
        """List or create comments for a post."""
        post = self.get_object()

        if request.method == "GET":
            comments = post.comments.filter(
                parent__isnull=True
            ).select_related("author").prefetch_related(
                "replies__author"
            )
            serializer = CommentSerializer(
                comments, many=True, context={"request": request}
            )
            return Response(serializer.data)

        # POST: create comment
        if not request.user.is_authenticated:
            return Response(
                {"detail": "Authentication required."},
                status=status.HTTP_401_UNAUTHORIZED
            )

        serializer = CommentSerializer(
            data=request.data,
            context={"request": request}
        )
        serializer.is_valid(raise_exception=True)
        serializer.save(post=post, author=request.user)
        return Response(serializer.data, status=status.HTTP_201_CREATED)


# ─────────────────────────────────────────
# COMMENTS
# ─────────────────────────────────────────
class CommentViewSet(ModelViewSet):
    """CRUD for comments."""
    serializer_class = CommentSerializer
    permission_classes = [IsAuthorOrReadOnly]

    def get_queryset(self):
        return Comment.objects.select_related(
            "author", "post"
        ).prefetch_related("replies__author")

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

    def perform_update(self, serializer):
        serializer.save(is_edited=True)


# ─────────────────────────────────────────
# TAGS
# ─────────────────────────────────────────
class TagListView(generics.ListAPIView):
    """List all tags."""
    serializer_class = TagSerializer
    permission_classes = [permissions.AllowAny]
    queryset = Tag.objects.annotate(
        post_count=Count("posts")
    ).order_by("-post_count")
    filter_backends = [filters.SearchFilter]
    search_fields = ["name"]
```

---

## 7. 🗺️ URL Configuration

Python

```
# users/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register("", views.UserViewSet, basename="users")

urlpatterns = [
    path("", include(router.urls)),
]
```

Python

```
# posts/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register("posts", views.PostViewSet, basename="posts")
router.register("comments", views.CommentViewSet, basename="comments")

urlpatterns = [
    path("", include(router.urls)),
    path("tags/", views.TagListView.as_view(), name="tag-list"),
]
```

Python

```
# config/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView,
)
from users import views as user_views

# API URL prefix
API_PREFIX = "api/v1/"

urlpatterns = [
    # Django Admin (HUGE advantage of Django!)
    path("admin/", admin.site.urls),

    # Auth endpoints
    path(f"{API_PREFIX}auth/register/",
         user_views.RegisterView.as_view(), name="register"),
    path(f"{API_PREFIX}auth/login/",
         user_views.LoginView.as_view(), name="login"),
    path(f"{API_PREFIX}auth/logout/",
         user_views.LogoutView.as_view(), name="logout"),
    path(f"{API_PREFIX}auth/me/",
         user_views.MeView.as_view(), name="me"),
    path(f"{API_PREFIX}auth/change-password/",
         user_views.ChangePasswordView.as_view(), name="change-password"),

    # JWT token endpoints (built-in)
    path(f"{API_PREFIX}auth/token/",
         TokenObtainPairView.as_view(), name="token-obtain"),
    path(f"{API_PREFIX}auth/token/refresh/",
         TokenRefreshView.as_view(), name="token-refresh"),
    path(f"{API_PREFIX}auth/token/verify/",
         TokenVerifyView.as_view(), name="token-verify"),

    # Users
    path(f"{API_PREFIX}users/", include("users.urls")),

    # Posts, Comments, Tags
    path(f"{API_PREFIX}", include("posts.urls")),

    # DRF browsable API login (for development)
    path("api-auth/", include("rest_framework.urls")),
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(
        settings.MEDIA_URL,
        document_root=settings.MEDIA_ROOT
    )
```

---

## 8. 🎛️ Django Admin

Python

```
# users/admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.utils.translation import gettext_lazy as _
from .models import User


@admin.register(User)
class UserAdmin(BaseUserAdmin):
    """
    Custom User admin.
    Django Admin is one of Django's BIGGEST advantages.
    You get a full management interface for FREE.
    """

    # What to show in list view
    list_display = [
        "email", "name", "role", "is_active",
        "is_verified", "created_at"
    ]
    list_filter = ["role", "is_active", "is_verified", "is_staff"]
    search_fields = ["email", "name"]
    ordering = ["-created_at"]

    # Clickable link in list
    list_display_links = ["email", "name"]

    # Actions (bulk operations)
    actions = ["activate_users", "deactivate_users", "verify_users"]

    # Detail view fieldsets
    fieldsets = (
        (None, {"fields": ("email", "password")}),
        (_("Personal info"), {"fields": ("name", "bio", "website", "avatar")}),
        (_("Permissions"), {
            "fields": (
                "role", "is_active", "is_verified",
                "is_staff", "is_superuser",
                "groups", "user_permissions"
            )
        }),
        (_("Important dates"), {"fields": ("last_login", "created_at", "updated_at")}),
    )

    readonly_fields = ["created_at", "updated_at", "last_login"]

    # Create user fieldsets
    add_fieldsets = (
        (None, {
            "classes": ("wide",),
            "fields": ("email", "name", "password1", "password2", "role"),
        }),
    )

    # Custom admin actions
    @admin.action(description="Activate selected users")
    def activate_users(self, request, queryset):
        count = queryset.update(is_active=True)
        self.message_user(request, f"{count} users activated.")

    @admin.action(description="Deactivate selected users")
    def deactivate_users(self, request, queryset):
        count = queryset.update(is_active=False)
        self.message_user(request, f"{count} users deactivated.")

    @admin.action(description="Mark selected users as verified")
    def verify_users(self, request, queryset):
        count = queryset.update(is_verified=True)
        self.message_user(request, f"{count} users verified.")
```

Python

```
# posts/admin.py
from django.contrib import admin
from django.utils.html import format_html
from .models import Post, Comment, Like, Tag


@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    list_display = ["name", "slug", "post_count"]
    search_fields = ["name"]
    prepopulated_fields = {"slug": ("name",)}

    def post_count(self, obj):
        return obj.posts.count()
    post_count.short_description = "Posts"


class CommentInline(admin.TabularInline):
    """Show comments inline in post admin."""
    model = Comment
    extra = 0
    readonly_fields = ["author", "content", "created_at"]
    can_delete = True


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = [
        "title", "author", "status", "view_count",
        "like_count", "published_at", "created_at"
    ]
    list_filter = ["status", "tags", "created_at"]
    search_fields = ["title", "content", "author__name", "author__email"]
    ordering = ["-created_at"]
    readonly_fields = [
        "slug", "view_count", "like_count",
        "created_at", "updated_at"
    ]
    filter_horizontal = ["tags"]   # better UI for M2M
    inlines = [CommentInline]

    # Custom column with formatting
    def status_badge(self, obj):
        colors = {
            "published": "green",
            "draft": "orange",
            "archived": "gray"
        }
        color = colors.get(obj.status, "black")
        return format_html(
            '<span style="color: {}; font-weight: bold;">{}</span>',
            color,
            obj.get_status_display()
        )
    status_badge.short_description = "Status"

    actions = ["publish_posts", "archive_posts"]

    @admin.action(description="Publish selected posts")
    def publish_posts(self, request, queryset):
        from django.utils import timezone
        count = queryset.filter(status="draft").update(
            status="published",
            published_at=timezone.now()
        )
        self.message_user(request, f"{count} posts published.")

    @admin.action(description="Archive selected posts")
    def archive_posts(self, request, queryset):
        count = queryset.update(status="archived")
        self.message_user(request, f"{count} posts archived.")


@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ["author", "post", "content_preview", "created_at"]
    list_filter = ["created_at"]
    search_fields = ["content", "author__name", "post__title"]
    readonly_fields = ["created_at", "updated_at"]

    def content_preview(self, obj):
        return obj.content[:50] + "..." if len(obj.content) > 50 else obj.content
    content_preview.short_description = "Content"
```

---

## 9. 🔄 Django Signals

Python

```
# posts/signals.py
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from .models import Post, Like, Comment


@receiver(post_save, sender=Post)
def post_created_or_updated(sender, instance, created, **kwargs):
    """Run after a post is saved."""
    if created:
        print(f"[SIGNAL] New post created: {instance.title}")
        # In production: send notifications, update search index, etc.
    else:
        if instance.status == "published":
            print(f"[SIGNAL] Post published: {instance.title}")
            # Notify followers, update sitemap, etc.


@receiver(post_save, sender=Comment)
def comment_added(sender, instance, created, **kwargs):
    """Notify post author when someone comments."""
    if created and instance.author != instance.post.author:
        print(
            f"[SIGNAL] New comment on '{instance.post.title}' "
            f"by {instance.author.name}"
        )
        # In production: send email notification
        # send_notification_email.delay(
        #     to=instance.post.author.email,
        #     subject=f"New comment on {instance.post.title}",
        #     message=f"{instance.author.name} commented: {instance.content}"
        # )


# Register signals in app config
```

Python

```
# posts/apps.py
from django.apps import AppConfig


class PostsConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "posts"

    def ready(self):
        """Import signals when app is ready."""
        import posts.signals  # noqa
```

---

## 10. ⚙️ Custom Middleware

Python

```
# config/middleware.py
import time
import logging
import uuid

logger = logging.getLogger(__name__)


class RequestLoggingMiddleware:
    """Log all requests with timing."""

    def __init__(self, get_response):
        self.get_response = get_response  # next middleware or view

    def __call__(self, request):
        # Generate request ID
        request_id = str(uuid.uuid4())[:8]
        request.request_id = request_id

        start = time.perf_counter()

        logger.info(
            f"[{request_id}] → {request.method} {request.path}"
            f" ({request.META.get('REMOTE_ADDR', 'unknown')})"
        )

        # Call next middleware / view
        response = self.get_response(request)

        duration = (time.perf_counter() - start) * 1000

        response["X-Request-ID"] = request_id
        response["X-Response-Time"] = f"{duration:.2f}ms"

        logger.info(
            f"[{request_id}] ← {response.status_code} {duration:.2f}ms"
        )

        return response
```

---

## 11. 🌿 Celery Integration

Python

```
# config/celery.py
import os
from celery import Celery

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")

app = Celery("config")
app.config_from_object("django.conf:settings", namespace="CELERY")
app.autodiscover_tasks()


# posts/tasks.py
from celery import shared_task
from django.core.mail import send_mail
from django.conf import settings


@shared_task(bind=True, max_retries=3)
def send_comment_notification(self, post_id: int, commenter_name: str):
    """Send email notification when someone comments on a post."""
    from posts.models import Post

    try:
        post = Post.objects.select_related("author").get(pk=post_id)
        send_mail(
            subject=f"New comment on '{post.title}'",
            message=f"{commenter_name} commented on your post.",
            from_email=settings.DEFAULT_FROM_EMAIL,
            recipient_list=[post.author.email],
        )
    except Post.DoesNotExist:
        pass
    except Exception as exc:
        # Retry with exponential backoff
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)


@shared_task
def update_post_stats():
    """Periodic task to update post statistics."""
    from posts.models import Post
    # Example: calculate engagement scores, update rankings, etc.
    print("Updating post stats...")
```

---

## 12. 🔄 Migrations

Bash

```
# ─────────────────────────────────────────
# DJANGO MIGRATIONS (different from Alembic!)
# ─────────────────────────────────────────

# Create the database
createdb -U postgres django_blog

# Create migrations from your models
python manage.py makemigrations

# Review what migrations will do
python manage.py sqlmigrate users 0001
python manage.py sqlmigrate posts 0001

# Apply migrations
python manage.py migrate

# Create superuser for admin access
python manage.py createsuperuser
# Enter email, name, password

# Run the development server
python manage.py runserver 8000

# Visit admin at: http://localhost:8000/admin/
```

---

## 13. 🔌 Django Channels (WebSockets)

Python

```
# Install: pip install channels channels-redis

# config/asgi.py
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack
from channels.security.websocket import AllowedHostsOriginValidator

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")

from posts.routing import websocket_urlpatterns

application = ProtocolTypeRouter({
    "http": get_asgi_application(),
    "websocket": AllowedHostsOriginValidator(
        AuthMiddlewareStack(
            URLRouter(websocket_urlpatterns)
        )
    ),
})


# posts/consumers.py
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from django.contrib.auth import get_user_model

User = get_user_model()


class ChatConsumer(AsyncWebsocketConsumer):
    """WebSocket consumer for chat rooms."""

    async def connect(self):
        self.room_name = self.scope["url_route"]["kwargs"]["room_name"]
        self.room_group_name = f"chat_{self.room_name}"

        # Join room group
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )
        await self.accept()

        await self.send(text_data=json.dumps({
            "type": "connected",
            "room": self.room_name,
            "message": f"Welcome to room {self.room_name}!"
        }))

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

    async def receive(self, text_data):
        data = json.loads(text_data)
        message_type = data.get("type", "message")

        if message_type == "message":
            await self.channel_layer.group_send(
                self.room_group_name,
                {
                    "type": "chat_message",
                    "message": data.get("content", ""),
                    "username": data.get("username", "Anonymous"),
                }
            )

    async def chat_message(self, event):
        """Receive message from room group and send to WebSocket."""
        await self.send(text_data=json.dumps({
            "type": "message",
            "message": event["message"],
            "username": event["username"],
        }))


# posts/routing.py
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r"ws/chat/(?P<room_name>\w+)/$", consumers.ChatConsumer.as_asgi()),
]
```

---

## 14. 🧪 Testing DRF

Python

```
# tests/test_posts.py
from django.test import TestCase
from django.contrib.auth import get_user_model
from rest_framework.test import APIClient
from rest_framework import status
from posts.models import Post, Tag

User = get_user_model()


class PostAPITest(TestCase):
    """Test suite for Post API endpoints."""

    def setUp(self):
        """Set up test data."""
        self.client = APIClient()

        # Create test users
        self.user1 = User.objects.create_user(
            email="user1@test.com",
            password="SecurePass1!",
            name="User One"
        )
        self.user2 = User.objects.create_user(
            email="user2@test.com",
            password="SecurePass1!",
            name="User Two"
        )
        self.admin = User.objects.create_superuser(
            email="admin@test.com",
            password="AdminPass1!",
            name="Admin User"
        )

        # Create test post
        self.post = Post.objects.create(
            title="Test Post",
            content="Test content for testing purposes.",
            author=self.user1,
            status="published"
        )

    def _auth(self, user) -> APIClient:
        """Get authenticated client for a user."""
        client = APIClient()
        client.force_authenticate(user=user)
        return client

    # ─────────────────────────────────────────
    # LIST POSTS
    # ─────────────────────────────────────────
    def test_list_posts_public(self):
        """Public users see only published posts."""
        response = self.client.get("/api/v1/posts/")
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data["results"]), 1)

    def test_list_posts_filtering(self):
        """Test filtering by status."""
        Draft = Post.objects.create(
            title="Draft Post",
            content="Draft content here.",
            author=self.user1,
            status="draft"
        )
        client = self._auth(self.user1)
        response = client.get("/api/v1/posts/?status=draft")
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        # User sees their own draft
        results = [p["title"] for p in response.data["results"]]
        self.assertIn("Draft Post", results)

    # ─────────────────────────────────────────
    # CREATE POST
    # ─────────────────────────────────────────
    def test_create_post_authenticated(self):
        """Authenticated users can create posts."""
        client = self._auth(self.user1)
        response = client.post("/api/v1/posts/", {
            "title": "My New Post",
            "content": "Content here for my new post.",
            "status": "published",
            "tag_names": ["python", "django"]
        }, format="json")

        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data["title"], "My New Post")
        self.assertEqual(response.data["author"]["name"], "User One")

    def test_create_post_unauthenticated(self):
        """Unauthenticated users cannot create posts."""
        response = self.client.post("/api/v1/posts/", {
            "title": "Unauthorized Post",
            "content": "This should fail.",
        })
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)

    # ─────────────────────────────────────────
    # UPDATE POST
    # ─────────────────────────────────────────
    def test_update_own_post(self):
        """Authors can update their own posts."""
        client = self._auth(self.user1)
        response = client.patch(
            f"/api/v1/posts/{self.post.pk}/",
            {"title": "Updated Title"},
            format="json"
        )
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data["title"], "Updated Title")

    def test_update_other_users_post(self):
        """Users cannot update other users' posts."""
        client = self._auth(self.user2)
        response = client.patch(
            f"/api/v1/posts/{self.post.pk}/",
            {"title": "Stealing this post"},
            format="json"
        )
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)

    def test_admin_can_update_any_post(self):
        """Admins can update any post."""
        client = self._auth(self.admin)
        response = client.patch(
            f"/api/v1/posts/{self.post.pk}/",
            {"title": "Admin Updated"},
            format="json"
        )
        self.assertEqual(response.status_code, status.HTTP_200_OK)

    # ─────────────────────────────────────────
    # LIKE SYSTEM
    # ─────────────────────────────────────────
    def test_like_post(self):
        """Users can like posts."""
        client = self._auth(self.user2)
        response = client.post(f"/api/v1/posts/{self.post.pk}/like/")
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertTrue(response.data["liked"])

    def test_cannot_like_twice(self):
        """Users cannot like the same post twice."""
        client = self._auth(self.user2)
        client.post(f"/api/v1/posts/{self.post.pk}/like/")
        response = client.post(f"/api/v1/posts/{self.post.pk}/like/")
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        # Second like returns 200 "Already liked" not 201

    def test_publish_post(self):
        """Authors can publish their draft posts."""
        draft = Post.objects.create(
            title="My Draft",
            content="Draft content here.",
            author=self.user1,
            status="draft"
        )
        client = self._auth(self.user1)
        response = client.post(f"/api/v1/posts/{draft.pk}/publish/")
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data["status"], "published")


# Run with:
# python manage.py test
# python manage.py test tests.test_posts -v 2
```

---

## 15. 🔧 Management Commands

Python

```
# posts/management/commands/seed_data.py
from django.core.management.base import BaseCommand
from django.contrib.auth import get_user_model
from posts.models import Post, Tag
import random

User = get_user_model()


class Command(BaseCommand):
    """
    Custom management command to seed test data.
    Run: python manage.py seed_data
    """
    help = "Seed the database with sample data"

    def add_arguments(self, parser):
        parser.add_argument(
            "--users", type=int, default=5,
            help="Number of users to create"
        )
        parser.add_argument(
            "--posts", type=int, default=20,
            help="Number of posts to create"
        )
        parser.add_argument(
            "--clear", action="store_true",
            help="Clear existing data first"
        )

    def handle(self, *args, **options):
        if options["clear"]:
            self.stdout.write("Clearing existing data...")
            Post.objects.all().delete()
            User.objects.filter(is_superuser=False).delete()

        self.stdout.write("Creating tags...")
        tags = []
        tag_names = ["python", "django", "fastapi", "api", "backend",
                     "database", "postgresql", "redis", "tutorial"]
        for name in tag_names:
            tag, _ = Tag.objects.get_or_create(name=name)
            tags.append(tag)

        self.stdout.write(f"Creating {options['users']} users...")
        users = []
        for i in range(options["users"]):
            user, created = User.objects.get_or_create(
                email=f"user{i}@example.com",
                defaults={
                    "name": f"User {i}",
                    "is_active": True,
                    "is_verified": True,
                }
            )
            if created:
                user.set_password("SecurePass1!")
                user.save()
            users.append(user)

        self.stdout.write(f"Creating {options['posts']} posts...")
        for i in range(options["posts"]):
            post = Post.objects.create(
                title=f"Sample Post {i+1}: Python and Django Tips",
                content=f"This is sample content for post {i+1}. " * 5,
                author=random.choice(users),
                status=random.choice(["draft", "published", "published"]),
                view_count=random.randint(0, 1000),
                like_count=random.randint(0, 100),
            )
            post.tags.set(random.sample(tags, random.randint(1, 4)))

        self.stdout.write(
            self.style.SUCCESS(
                f"Successfully seeded {options['users']} users "
                f"and {options['posts']} posts!"
            )
        )
```

---

## 📊 FastAPI vs Django Comparison

text

```
┌────────────────────────┬────────────────────────┬────────────────────────┐
│ Feature                │ FastAPI                 │ Django + DRF           │
├────────────────────────┼────────────────────────┼────────────────────────┤
│ Performance            │ ⚡ Excellent             │ 🟡 Good                │
│ Admin Panel            │ ❌ None (build yourself) │ ✅ Built-in, powerful  │
│ ORM                    │ SQLAlchemy (external)   │ ✅ Built-in Django ORM │
│ Auth System            │ Build yourself          │ ✅ Built-in            │
│ Migrations             │ Alembic                 │ ✅ Built-in            │
│ Async Support          │ ✅ Native               │ 🟡 Partial (Channels) │
│ Type Safety            │ ✅ Excellent (Pydantic) │ 🟡 Manual              │
│ Auto API Docs          │ ✅ Auto OpenAPI         │ 🟡 DRF browsable API  │
│ Learning Curve         │ 🟡 Medium              │ 🔴 Steeper             │
│ Structure              │ Flexible               │ ✅ Opinionated          │
│ Job Market             │ Growing fast           │ ✅ Established, large  │
│ Testing                │ pytest + TestClient    │ Django TestCase        │
│ Background Tasks       │ BackgroundTasks        │ ✅ Celery integration  │
│ WebSockets             │ ✅ Built-in            │ Channels (external)    │
│ File Uploads           │ UploadFile             │ ✅ Built-in handling   │
│ Email                  │ External library       │ ✅ Built-in email      │
│ Validation             │ Pydantic (excellent)   │ Serializers (good)     │
│ Cache                  │ External               │ ✅ Built-in cache API  │
└────────────────────────┴────────────────────────┴────────────────────────┘

Bottom line:
  FastAPI → new projects, microservices, high performance, modern
  Django  → full-featured apps, admin needed, large teams, enterprise
```

---

## ✅ Knowledge Check

text

```
1.  What is the MVT pattern in Django?
2.  What is a Django app vs a Django project?
3.  What is AUTH_USER_MODEL and why do you set it?
4.  What is the difference between AbstractBaseUser
    and AbstractUser?
5.  What is a Django migration and how do you create one?
6.  What does DRF Serializer do? Compare to FastAPI's Pydantic.
7.  What is a ModelViewSet and what endpoints does it provide?
8.  What is the difference between has_permission
    and has_object_permission?
9.  What are Django signals and give two use cases?
10. What does @action do on a ViewSet?
11. What is Django admin and why is it valuable?
12. What is Celery used for in Django?
13. How does DRF authentication differ from FastAPI's?
14. What is django-filter used for?
15. When would you choose Django over FastAPI?
```

---

## 🛠️ Practice Exercises

Python

```
# Exercise 1: Custom Pagination
# Create a custom pagination class that returns:
# {
#     "count": 100,
#     "next": "http://api/posts/?page=3",
#     "previous": "http://api/posts/?page=1",
#     "total_pages": 10,
#     "current_page": 2,
#     "results": [...]
# }
# Apply it globally in settings

# Exercise 2: Nested Router
# Install drf-nested-routers
# Create nested routes:
# /api/v1/posts/{post_id}/comments/
# /api/v1/posts/{post_id}/comments/{comment_id}/
# /api/v1/users/{user_id}/posts/
# Ensure parent-child relationship is enforced

# Exercise 3: Custom Exception Handler
# Create a custom exception handler that formats ALL errors as:
# {
#     "success": false,
#     "error_code": "VALIDATION_ERROR",
#     "message": "Validation failed",
#     "details": [
#         {"field": "email", "message": "Invalid email format"}
#     ]
# }
# Handle: ValidationError, PermissionDenied,
#         NotAuthenticated, NotFound, Http404

# Exercise 4: Admin Customization
# Enhance the PostAdmin:
# - Add a "Content Preview" column showing first 100 chars
# - Add a "Publish All" bulk action
# - Add a "Total Views" summary at the top of the list
# - Make clicking the title open the post's detail view
# - Add filters by date range
# - Add export to CSV action

# Exercise 5: Celery Task
# Create a Celery task: generate_weekly_report()
# That runs every Monday at 9am (use Celery Beat)
# Generates a report with:
# - New users this week
# - New posts this week
# - Most viewed posts
# - Most active users
# Sends the report via email to admin users
```

---

## ✅ Phase 4.3 Complete!

**You now know Django + DRF:**

text

```
✅ Django project & app structure
✅ Custom User model with AbstractBaseUser
✅ Django ORM models (ForeignKey, M2M, signals)
✅ DRF Serializers (validation, nested, custom methods)
✅ Generic views and ViewSets
✅ Custom permissions (object-level)
✅ URL configuration with routers
✅ JWT authentication with SimpleJWT
✅ Django Admin customization
✅ Django signals
✅ Custom middleware
✅ Celery integration
✅ Django Channels (WebSockets)
✅ Django migrations
✅ Testing with DRF TestCase
✅ Custom management commands
```