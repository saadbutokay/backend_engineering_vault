```
Flask = micro-framework
  "Micro" means: gives you the minimum, you add what you need.

Why learn Flask in 2024?
┌─────────────────────────────────────────────────────────┐
│  Millions of existing Flask apps in production          │
│  Many companies still use it (you'll join their teams)  │
│  Simplest way to understand web frameworks              │
│  Great for: scripts, internal tools, simple APIs        │
│  Used in: ML model serving, data science backends       │
│  Foundation: FastAPI is built on Starlette, inspired    │
│              heavily by Flask's design                  │
└─────────────────────────────────────────────────────────┘

Flask vs others:
  Flask   → minimal, you build everything yourself
  Django  → maximum, everything included
  FastAPI → modern Flask with async + type hints

After FastAPI and Django, Flask will feel familiar.
You already know 80% — just different APIs.
```

---

## Setup

Bash

```
cd ~/projects
mkdir flask_project
cd flask_project
python3 -m venv venv
source venv/bin/activate

pip install flask flask-sqlalchemy flask-migrate \
    flask-jwt-extended flask-cors marshmallow \
    flask-restx psycopg2-binary python-dotenv \
    werkzeug

touch app.py
mkdir -p app/{models,resources,schemas,services}
touch app/__init__.py
touch .env
code .
```

Bash

```
# .env
DATABASE_URL=postgresql://myapp_user:myapp_password@localhost:5432/flask_blog
SECRET_KEY=your-secret-key-here
JWT_SECRET_KEY=your-jwt-secret-here
DEBUG=True
```

---

## 1. 🚀 Flask Fundamentals

### Your First Flask App

Python

```
# hello.py — absolute minimum Flask app
from flask import Flask

app = Flask(__name__)   # __name__ = module name

@app.route("/")         # URL decorator
def hello():
    return "Hello, Flask!"  # return string directly

@app.route("/json")
def hello_json():
    from flask import jsonify
    return jsonify({"message": "Hello!", "framework": "Flask"})

# Run: flask run
# Or:  python hello.py
if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

Bash

```
# Run Flask
flask --app hello run --debug
# Or set environment variable:
export FLASK_APP=hello.py
flask run --debug
```

### Routing Deep Dive

Python

```
# routing_demo.py
from flask import Flask, request, jsonify, abort, redirect, url_for

app = Flask(__name__)

# ─────────────────────────────────────────
# BASIC ROUTES
# ─────────────────────────────────────────

@app.route("/")
def index():
    return jsonify({"message": "Flask API"})

# Multiple methods on one route
@app.route("/users", methods=["GET", "POST"])
def users():
    if request.method == "GET":
        return jsonify({"action": "list users"})
    elif request.method == "POST":
        data = request.get_json()   # parse JSON body
        return jsonify({"action": "create user", "data": data}), 201

# ─────────────────────────────────────────
# PATH PARAMETERS (URL variables)
# ─────────────────────────────────────────

@app.route("/users/<int:user_id>")
def get_user(user_id: int):
    # <int:user_id> — type converter: int, float, string, path, uuid
    return jsonify({"user_id": user_id})

@app.route("/posts/<string:slug>")
def get_post_by_slug(slug: str):
    return jsonify({"slug": slug})

@app.route("/files/<path:filepath>")  # matches slashes too
def get_file(filepath: str):
    return jsonify({"path": filepath})

# ─────────────────────────────────────────
# QUERY PARAMETERS
# ─────────────────────────────────────────

@app.route("/search")
def search():
    # ?q=python&page=1&limit=20
    q = request.args.get("q", "")           # default ""
    page = request.args.get("page", 1, type=int)    # convert to int
    limit = request.args.get("limit", 20, type=int)

    return jsonify({
        "query": q,
        "page": page,
        "limit": limit,
        "results": []
    })

# ─────────────────────────────────────────
# REQUEST DATA
# ─────────────────────────────────────────

@app.route("/data", methods=["POST"])
def receive_data():
    # JSON body
    if request.is_json:
        data = request.get_json()
        return jsonify({"received_json": data})

    # Form data
    form_data = request.form.to_dict()
    # File upload
    file = request.files.get("file")

    # Headers
    content_type = request.headers.get("Content-Type")
    auth_header = request.headers.get("Authorization")

    # Cookies
    session_token = request.cookies.get("session")

    return jsonify({"form_data": form_data})

# ─────────────────────────────────────────
# RESPONSE TYPES
# ─────────────────────────────────────────

@app.route("/responses/<string:response_type>")
def response_demo(response_type: str):
    from flask import make_response, Response

    if response_type == "json":
        return jsonify({"key": "value"})    # 200 by default

    if response_type == "with_status":
        return jsonify({"error": "not found"}), 404

    if response_type == "with_headers":
        response = make_response(jsonify({"data": "here"}))
        response.headers["X-Custom-Header"] = "custom-value"
        response.headers["Cache-Control"] = "no-cache"
        return response

    if response_type == "redirect":
        return redirect(url_for("index"))   # redirect to index route

    if response_type == "stream":
        def generate():
            for i in range(10):
                yield f"data: {i}\n\n"   # Server-Sent Events format
        return Response(generate(), mimetype="text/event-stream")

    return jsonify({"type": response_type})

# ─────────────────────────────────────────
# ERROR HANDLING
# ─────────────────────────────────────────

@app.errorhandler(404)
def not_found(error):
    return jsonify({
        "success": False,
        "error": "Resource not found",
        "status_code": 404
    }), 404

@app.errorhandler(500)
def server_error(error):
    return jsonify({
        "success": False,
        "error": "Internal server error",
        "status_code": 500
    }), 500

@app.errorhandler(Exception)
def handle_exception(error):
    import traceback
    app.logger.error(traceback.format_exc())
    return jsonify({
        "success": False,
        "error": str(error)
    }), 500

if __name__ == "__main__":
    app.run(debug=True)
```

---

## 2. 🏗️ Application Factory Pattern

Python

```
# app/__init__.py
"""
Application Factory Pattern:
  Create Flask app inside a function, not at module level.

Why?
  - Multiple instances (testing, production)
  - Avoid circular imports
  - Clean initialization order
  - Environment-specific configuration
"""
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_jwt_extended import JWTManager
from flask_cors import CORS
import os
from dotenv import load_dotenv

load_dotenv()

# Extensions (created outside app to avoid circular imports)
db = SQLAlchemy()
migrate = Migrate()
jwt = JWTManager()
cors = CORS()


def create_app(config_name: str = "development") -> Flask:
    """Application factory function."""
    app = Flask(__name__)

    # ─────────────────────────────────────────
    # CONFIGURATION
    # ─────────────────────────────────────────
    app.config["SECRET_KEY"] = os.getenv("SECRET_KEY", "dev-secret")
    app.config["SQLALCHEMY_DATABASE_URI"] = os.getenv(
        "DATABASE_URL",
        "postgresql://myapp_user:myapp_password@localhost:5432/flask_blog"
    )
    app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
    app.config["SQLALCHEMY_ENGINE_OPTIONS"] = {
        "pool_size": 5,
        "max_overflow": 10,
        "pool_pre_ping": True,
    }
    app.config["JWT_SECRET_KEY"] = os.getenv("JWT_SECRET_KEY", "jwt-secret")
    app.config["JWT_ACCESS_TOKEN_EXPIRES"] = 1800    # 30 minutes
    app.config["JWT_REFRESH_TOKEN_EXPIRES"] = 604800  # 7 days
    app.config["MAX_CONTENT_LENGTH"] = 16 * 1024 * 1024  # 16MB upload limit

    if config_name == "testing":
        app.config["TESTING"] = True
        app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///:memory:"
        app.config["JWT_ACCESS_TOKEN_EXPIRES"] = False  # no expiry in tests

    # ─────────────────────────────────────────
    # INITIALIZE EXTENSIONS
    # ─────────────────────────────────────────
    db.init_app(app)
    migrate.init_app(app, db)
    jwt.init_app(app)
    cors.init_app(
        app,
        resources={
            r"/api/*": {
                "origins": ["http://localhost:3000"],
                "methods": ["GET", "POST", "PUT", "PATCH", "DELETE"],
                "allow_headers": ["Authorization", "Content-Type"]
            }
        }
    )

    # ─────────────────────────────────────────
    # JWT CALLBACKS
    # ─────────────────────────────────────────
    @jwt.expired_token_loader
    def expired_token_callback(jwt_header, jwt_payload):
        return {
            "success": False,
            "error": "Token has expired",
            "error_code": "TOKEN_EXPIRED"
        }, 401

    @jwt.invalid_token_loader
    def invalid_token_callback(error):
        return {
            "success": False,
            "error": "Invalid token",
            "error_code": "INVALID_TOKEN"
        }, 401

    @jwt.unauthorized_loader
    def missing_token_callback(error):
        return {
            "success": False,
            "error": "Authorization required",
            "error_code": "MISSING_TOKEN"
        }, 401

    # ─────────────────────────────────────────
    # REGISTER BLUEPRINTS
    # ─────────────────────────────────────────
    from app.resources.auth import auth_bp
    from app.resources.users import users_bp
    from app.resources.posts import posts_bp

    app.register_blueprint(auth_bp, url_prefix="/api/v1/auth")
    app.register_blueprint(users_bp, url_prefix="/api/v1/users")
    app.register_blueprint(posts_bp, url_prefix="/api/v1/posts")

    # ─────────────────────────────────────────
    # ERROR HANDLERS
    # ─────────────────────────────────────────
    register_error_handlers(app)

    # ─────────────────────────────────────────
    # HEALTH CHECK
    # ─────────────────────────────────────────
    @app.route("/health")
    def health():
        return {
            "status": "healthy",
            "environment": config_name
        }

    return app


def register_error_handlers(app: Flask) -> None:
    """Register global error handlers."""
    from flask import jsonify
    from sqlalchemy.exc import SQLAlchemyError
    from marshmallow import ValidationError

    @app.errorhandler(400)
    def bad_request(error):
        return jsonify({
            "success": False,
            "error": "Bad request",
            "message": str(error.description)
        }), 400

    @app.errorhandler(401)
    def unauthorized(error):
        return jsonify({
            "success": False,
            "error": "Unauthorized",
            "message": "Authentication required"
        }), 401

    @app.errorhandler(403)
    def forbidden(error):
        return jsonify({
            "success": False,
            "error": "Forbidden",
            "message": "You do not have permission"
        }), 403

    @app.errorhandler(404)
    def not_found(error):
        return jsonify({
            "success": False,
            "error": "Not found",
            "message": str(error.description)
        }), 404

    @app.errorhandler(422)
    def unprocessable(error):
        return jsonify({
            "success": False,
            "error": "Unprocessable entity",
            "message": str(error.description)
        }), 422

    @app.errorhandler(429)
    def too_many_requests(error):
        return jsonify({
            "success": False,
            "error": "Too many requests",
            "message": "Rate limit exceeded"
        }), 429

    @app.errorhandler(ValidationError)
    def marshmallow_validation_error(error):
        return jsonify({
            "success": False,
            "error": "Validation failed",
            "details": error.messages
        }), 422

    @app.errorhandler(SQLAlchemyError)
    def database_error(error):
        app.logger.error(f"Database error: {error}")
        db.session.rollback()
        return jsonify({
            "success": False,
            "error": "Database error"
        }), 500

    @app.errorhandler(Exception)
    def generic_error(error):
        app.logger.exception("Unexpected error")
        return jsonify({
            "success": False,
            "error": "Internal server error"
        }), 500
```

---

## 3. 🗄️ Flask-SQLAlchemy Models

Python

```
# app/models/user.py
from app import db
from werkzeug.security import generate_password_hash, check_password_hash
from datetime import datetime, timezone
from sqlalchemy import event


class User(db.Model):
    """User model."""
    __tablename__ = "users"

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(255), nullable=False, unique=True, index=True)
    password_hash = db.Column(db.String(255), nullable=False)
    bio = db.Column(db.Text, nullable=True)
    role = db.Column(
        db.String(20),
        nullable=False,
        default="user",
        index=True
    )
    is_active = db.Column(db.Boolean, default=True, nullable=False)
    is_verified = db.Column(db.Boolean, default=False, nullable=False)
    created_at = db.Column(
        db.DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc),
        nullable=False
    )
    updated_at = db.Column(
        db.DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc),
        onupdate=lambda: datetime.now(timezone.utc),
        nullable=False
    )
    deleted_at = db.Column(db.DateTime(timezone=True), nullable=True)

    # Relationship
    posts = db.relationship(
        "Post",
        back_populates="author",
        lazy="dynamic",         # returns query object, not list
        cascade="all, delete-orphan"
    )

    # ─────────────────────────────────────────
    # PASSWORD HANDLING
    # ─────────────────────────────────────────
    def set_password(self, password: str) -> None:
        self.password_hash = generate_password_hash(password)

    def check_password(self, password: str) -> bool:
        return check_password_hash(self.password_hash, password)

    # ─────────────────────────────────────────
    # PROPERTIES
    # ─────────────────────────────────────────
    @property
    def is_admin(self) -> bool:
        return self.role == "admin"

    @property
    def is_deleted(self) -> bool:
        return self.deleted_at is not None

    # ─────────────────────────────────────────
    # CLASS METHODS (alternative constructors)
    # ─────────────────────────────────────────
    @classmethod
    def find_by_email(cls, email: str) -> "User | None":
        return cls.query.filter(
            cls.email == email.lower(),
            cls.deleted_at.is_(None)
        ).first()

    @classmethod
    def find_by_id(cls, user_id: int) -> "User | None":
        return cls.query.filter(
            cls.id == user_id,
            cls.deleted_at.is_(None)
        ).first()

    def soft_delete(self) -> None:
        self.deleted_at = datetime.now(timezone.utc)
        self.is_active = False

    def to_dict(self) -> dict:
        """Convert to dictionary."""
        return {
            "id": self.id,
            "name": self.name,
            "email": self.email,
            "role": self.role,
            "bio": self.bio,
            "is_active": self.is_active,
            "is_verified": self.is_verified,
            "created_at": self.created_at.isoformat() if self.created_at else None,
        }

    def __repr__(self):
        return f"<User {self.email}>"
```

Python

```
# app/models/post.py
from app import db
from datetime import datetime, timezone
from sqlalchemy import event
import re

# Many-to-many association table
post_tags = db.Table(
    "post_tags",
    db.Column("post_id", db.Integer, db.ForeignKey("posts.id"), primary_key=True),
    db.Column("tag_id", db.Integer, db.ForeignKey("tags.id"), primary_key=True)
)


class Tag(db.Model):
    __tablename__ = "tags"

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), unique=True, nullable=False)
    slug = db.Column(db.String(50), unique=True, nullable=False)
    created_at = db.Column(
        db.DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc)
    )

    @classmethod
    def get_or_create(cls, name: str) -> "Tag":
        slug = re.sub(r"[^a-z0-9-]", "-", name.lower().strip())
        tag = cls.query.filter_by(name=name).first()
        if not tag:
            tag = cls(name=name, slug=slug)
            db.session.add(tag)
        return tag

    def to_dict(self) -> dict:
        return {"id": self.id, "name": self.name, "slug": self.slug}

    def __repr__(self):
        return f"<Tag {self.name}>"


class Post(db.Model):
    __tablename__ = "posts"

    STATUS_DRAFT = "draft"
    STATUS_PUBLISHED = "published"
    STATUS_ARCHIVED = "archived"

    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(500), nullable=False)
    content = db.Column(db.Text, nullable=False)
    slug = db.Column(db.String(500), unique=True, index=True)
    status = db.Column(
        db.String(20),
        nullable=False,
        default=STATUS_DRAFT,
        index=True
    )

    author_id = db.Column(
        db.Integer,
        db.ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True
    )

    view_count = db.Column(db.Integer, default=0, nullable=False)
    like_count = db.Column(db.Integer, default=0, nullable=False)

    published_at = db.Column(db.DateTime(timezone=True), nullable=True)
    created_at = db.Column(
        db.DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc),
        nullable=False
    )
    updated_at = db.Column(
        db.DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc),
        onupdate=lambda: datetime.now(timezone.utc),
        nullable=False
    )
    deleted_at = db.Column(db.DateTime(timezone=True), nullable=True)

    # Relationships
    author = db.relationship("User", back_populates="posts")
    tags = db.relationship(
        "Tag",
        secondary=post_tags,
        backref=db.backref("posts", lazy="dynamic"),
        lazy="subquery"     # load tags with post in same query
    )

    def generate_slug(self) -> str:
        """Generate unique slug from title."""
        base = re.sub(r"[^a-z0-9\s-]", "", self.title.lower())
        base = re.sub(r"[\s-]+", "-", base).strip("-")
        slug = base
        counter = 1
        while Post.query.filter(
            Post.slug == slug,
            Post.id != self.id
        ).first():
            slug = f"{base}-{counter}"
            counter += 1
        return slug

    def publish(self) -> None:
        self.status = self.STATUS_PUBLISHED
        if not self.published_at:
            self.published_at = datetime.now(timezone.utc)

    def increment_views(self) -> None:
        """Atomic view count increment."""
        db.session.execute(
            db.text("UPDATE posts SET view_count = view_count + 1 WHERE id = :id"),
            {"id": self.id}
        )

    def to_dict(self, include_content: bool = True) -> dict:
        data = {
            "id": self.id,
            "title": self.title,
            "slug": self.slug,
            "status": self.status,
            "author": self.author.to_dict() if self.author else None,
            "tags": [tag.to_dict() for tag in self.tags],
            "view_count": self.view_count,
            "like_count": self.like_count,
            "published_at": self.published_at.isoformat() if self.published_at else None,
            "created_at": self.created_at.isoformat() if self.created_at else None,
            "updated_at": self.updated_at.isoformat() if self.updated_at else None,
        }
        if include_content:
            data["content"] = self.content
        return data

    def __repr__(self):
        return f"<Post {self.title!r}>"


# Auto-generate slug before insert
@event.listens_for(Post, "before_insert")
def generate_slug_on_insert(mapper, connection, target):
    if not target.slug and target.title:
        target.slug = target.generate_slug()
```

---

## 4. 🏷️ Marshmallow Schemas

Python

```
# app/schemas/user.py
from marshmallow import Schema, fields, validate, validates, ValidationError, post_load
from app.models.user import User
import re


class UserRegistrationSchema(Schema):
    """Validate user registration data."""
    name = fields.Str(
        required=True,
        validate=validate.Length(min=2, max=100)
    )
    email = fields.Email(required=True)
    password = fields.Str(
        required=True,
        validate=validate.Length(min=8),
        load_only=True   # never include in output
    )

    @validates("password")
    def validate_password_strength(self, value: str) -> str:
        if not any(c.isupper() for c in value):
            raise ValidationError("Password needs at least one uppercase letter")
        if not any(c.isdigit() for c in value):
            raise ValidationError("Password needs at least one number")
        return value

    @validates("email")
    def validate_email_unique(self, value: str) -> str:
        if User.find_by_email(value):
            raise ValidationError("Email already registered")
        return value.lower()

    @post_load
    def clean_data(self, data: dict, **kwargs) -> dict:
        data["name"] = data["name"].strip().title()
        return data


class UserResponseSchema(Schema):
    """Serialize user for API response."""
    id = fields.Int(dump_only=True)
    name = fields.Str()
    email = fields.Email()
    role = fields.Str()
    bio = fields.Str(allow_none=True)
    is_active = fields.Bool()
    is_verified = fields.Bool()
    created_at = fields.DateTime(dump_only=True)


class UserUpdateSchema(Schema):
    """Validate user update data (all optional)."""
    name = fields.Str(validate=validate.Length(min=2, max=100))
    bio = fields.Str(validate=validate.Length(max=500), allow_none=True)
    website = fields.Url(allow_none=True)


class LoginSchema(Schema):
    """Validate login credentials."""
    email = fields.Email(required=True)
    password = fields.Str(required=True, load_only=True)
```

Python

```
# app/schemas/post.py
from marshmallow import Schema, fields, validate, validates, ValidationError, post_load
from marshmallow import EXCLUDE


class PostCreateSchema(Schema):
    """Validate post creation data."""
    title = fields.Str(
        required=True,
        validate=validate.Length(min=3, max=500)
    )
    content = fields.Str(
        required=True,
        validate=validate.Length(min=10)
    )
    status = fields.Str(
        validate=validate.OneOf(["draft", "published"]),
        load_default="draft"
    )
    tags = fields.List(
        fields.Str(validate=validate.Length(max=50)),
        load_default=[],
        validate=validate.Length(max=10)
    )

    @validates("title")
    def clean_title(self, value: str) -> str:
        return value.strip()

    @post_load
    def normalize_tags(self, data: dict, **kwargs) -> dict:
        data["tags"] = [t.lower().strip() for t in data.get("tags", []) if t.strip()]
        return data


class PostUpdateSchema(Schema):
    """Validate post update (all optional)."""
    title = fields.Str(validate=validate.Length(min=3, max=500))
    content = fields.Str(validate=validate.Length(min=10))
    status = fields.Str(validate=validate.OneOf(["draft", "published", "archived"]))
    tags = fields.List(fields.Str(), load_default=None)

    class Meta:
        unknown = EXCLUDE   # ignore unknown fields


class PostResponseSchema(Schema):
    """Serialize post for API response."""
    id = fields.Int(dump_only=True)
    title = fields.Str()
    content = fields.Str()
    slug = fields.Str()
    status = fields.Str()
    author = fields.Method("get_author")
    tags = fields.Method("get_tags")
    view_count = fields.Int()
    like_count = fields.Int()
    published_at = fields.DateTime(allow_none=True)
    created_at = fields.DateTime()
    updated_at = fields.DateTime()

    def get_author(self, obj) -> dict:
        if obj.author:
            return {"id": obj.author.id, "name": obj.author.name}
        return None

    def get_tags(self, obj) -> list:
        return [{"id": t.id, "name": t.name} for t in obj.tags]


class PostListResponseSchema(Schema):
    """Lightweight schema for list views (no full content)."""
    id = fields.Int()
    title = fields.Str()
    slug = fields.Str()
    status = fields.Str()
    author = fields.Method("get_author")
    tags = fields.Method("get_tags")
    view_count = fields.Int()
    like_count = fields.Int()
    published_at = fields.DateTime(allow_none=True)
    created_at = fields.DateTime()

    def get_author(self, obj) -> dict:
        if obj.author:
            return {"id": obj.author.id, "name": obj.author.name}
        return None

    def get_tags(self, obj) -> list:
        return [t.name for t in obj.tags]
```

---

## 5. 📁 Blueprints — Modular Routes

Python

```
# app/resources/auth.py
from flask import Blueprint, request, jsonify
from flask_jwt_extended import (
    create_access_token, create_refresh_token,
    jwt_required, get_jwt_identity, get_jwt
)
from marshmallow import ValidationError
from app import db
from app.models.user import User
from app.schemas.user import UserRegistrationSchema, LoginSchema, UserResponseSchema

auth_bp = Blueprint("auth", __name__)

# Schema instances (reuse)
register_schema = UserRegistrationSchema()
login_schema = LoginSchema()
user_response_schema = UserResponseSchema()


@auth_bp.route("/register", methods=["POST"])
def register():
    """Register a new user."""
    # 1. Validate and parse input
    try:
        data = register_schema.load(request.get_json() or {})
    except ValidationError as err:
        return jsonify({
            "success": False,
            "error": "Validation failed",
            "details": err.messages
        }), 422

    # 2. Create user
    user = User(
        name=data["name"],
        email=data["email"]
    )
    user.set_password(data["password"])
    db.session.add(user)
    db.session.commit()

    # 3. Generate tokens
    access_token = create_access_token(identity=user.id)
    refresh_token = create_refresh_token(identity=user.id)

    return jsonify({
        "success": True,
        "message": "Registration successful",
        "data": {
            "user": user_response_schema.dump(user),
            "access_token": access_token,
            "refresh_token": refresh_token,
            "token_type": "bearer"
        }
    }), 201


@auth_bp.route("/login", methods=["POST"])
def login():
    """Login and get tokens."""
    try:
        data = login_schema.load(request.get_json() or {})
    except ValidationError as err:
        return jsonify({
            "success": False,
            "details": err.messages
        }), 422

    user = User.find_by_email(data["email"])

    if not user or not user.check_password(data["password"]):
        return jsonify({
            "success": False,
            "error": "Invalid email or password"
        }), 401

    if not user.is_active:
        return jsonify({
            "success": False,
            "error": "Account is deactivated"
        }), 403

    access_token = create_access_token(identity=user.id)
    refresh_token = create_refresh_token(identity=user.id)

    return jsonify({
        "success": True,
        "data": {
            "user": user_response_schema.dump(user),
            "access_token": access_token,
            "refresh_token": refresh_token,
            "token_type": "bearer"
        }
    })


@auth_bp.route("/refresh", methods=["POST"])
@jwt_required(refresh=True)     # requires refresh token
def refresh():
    """Get new access token using refresh token."""
    user_id = get_jwt_identity()
    user = User.find_by_id(user_id)

    if not user or not user.is_active:
        return jsonify({
            "success": False,
            "error": "User not found"
        }), 401

    access_token = create_access_token(identity=user.id)
    return jsonify({
        "success": True,
        "data": {
            "access_token": access_token,
            "token_type": "bearer"
        }
    })


@auth_bp.route("/me", methods=["GET"])
@jwt_required()         # requires access token
def me():
    """Get current user profile."""
    user_id = get_jwt_identity()
    user = User.find_by_id(user_id)

    if not user:
        return jsonify({"success": False, "error": "User not found"}), 404

    return jsonify({
        "success": True,
        "data": user_response_schema.dump(user)
    })


@auth_bp.route("/logout", methods=["POST"])
@jwt_required()
def logout():
    """
    Logout. In production, add token to blocklist.
    Use Redis: redis.setex(f"blocklist:{jti}", ttl, "true")
    """
    # jti = JWT ID (unique identifier for this token)
    # jti = get_jwt()["jti"]
    # redis_client.setex(f"blocklist:{jti}", 1800, "true")
    return jsonify({"success": True, "message": "Logged out"}), 200
```

Python

```
# app/resources/posts.py
from flask import Blueprint, request, jsonify, g
from flask_jwt_extended import jwt_required, get_jwt_identity, verify_jwt_in_request
from marshmallow import ValidationError
from functools import wraps
from app import db
from app.models.post import Post, Tag
from app.models.user import User
from app.schemas.post import (
    PostCreateSchema, PostUpdateSchema,
    PostResponseSchema, PostListResponseSchema
)

posts_bp = Blueprint("posts", __name__)

# Schema instances
create_schema = PostCreateSchema()
update_schema = PostUpdateSchema()
post_schema = PostResponseSchema()
posts_schema = PostListResponseSchema(many=True)


# ─────────────────────────────────────────
# HELPER: Get current user (optional auth)
# ─────────────────────────────────────────
def get_current_user() -> User | None:
    """Get current user if authenticated."""
    try:
        verify_jwt_in_request(optional=True)
        user_id = get_jwt_identity()
        if user_id:
            return User.find_by_id(user_id)
    except Exception:
        pass
    return None


# ─────────────────────────────────────────
# DECORATOR: Require authentication
# ─────────────────────────────────────────
def auth_required(f):
    """Decorator to require authentication."""
    @wraps(f)
    @jwt_required()
    def decorated(*args, **kwargs):
        user_id = get_jwt_identity()
        user = User.find_by_id(user_id)
        if not user:
            return jsonify({"error": "User not found"}), 401
        if not user.is_active:
            return jsonify({"error": "Account deactivated"}), 403
        g.current_user = user    # store in request context
        return f(*args, **kwargs)
    return decorated


# ─────────────────────────────────────────
# LIST POSTS
# ─────────────────────────────────────────
@posts_bp.route("", methods=["GET"])
def list_posts():
    """
    GET /api/v1/posts
    Query params: page, per_page, status, author_id, tag, q
    """
    current_user = get_current_user()

    # Build query
    query = Post.query.filter(Post.deleted_at.is_(None))

    # Non-authenticated users see only published
    if not current_user:
        query = query.filter(Post.status == "published")

    # Filters
    status = request.args.get("status")
    if status:
        query = query.filter(Post.status == status)

    author_id = request.args.get("author_id", type=int)
    if author_id:
        query = query.filter(Post.author_id == author_id)

    tag = request.args.get("tag")
    if tag:
        query = query.filter(Post.tags.any(Tag.slug == tag))

    search = request.args.get("q", "").strip()
    if search:
        query = query.filter(
            Post.title.ilike(f"%{search}%") |
            Post.content.ilike(f"%{search}%")
        )

    # Sorting
    sort = request.args.get("sort", "-created_at")
    if sort.startswith("-"):
        order_col = getattr(Post, sort[1:], Post.created_at)
        query = query.order_by(order_col.desc())
    else:
        order_col = getattr(Post, sort, Post.created_at)
        query = query.order_by(order_col.asc())

    # Pagination
    page = request.args.get("page", 1, type=int)
    per_page = min(request.args.get("per_page", 20, type=int), 100)

    paginated = query.paginate(
        page=page,
        per_page=per_page,
        error_out=False    # return empty list instead of 404
    )

    return jsonify({
        "success": True,
        "data": posts_schema.dump(paginated.items),
        "pagination": {
            "total": paginated.total,
            "page": paginated.page,
            "per_page": paginated.per_page,
            "total_pages": paginated.pages,
            "has_next": paginated.has_next,
            "has_prev": paginated.has_prev
        }
    })


# ─────────────────────────────────────────
# CREATE POST
# ─────────────────────────────────────────
@posts_bp.route("", methods=["POST"])
@auth_required
def create_post():
    """POST /api/v1/posts"""
    try:
        data = create_schema.load(request.get_json() or {})
    except ValidationError as err:
        return jsonify({
            "success": False,
            "error": "Validation failed",
            "details": err.messages
        }), 422

    # Create post
    post = Post(
        title=data["title"],
        content=data["content"],
        status=data.get("status", "draft"),
        author_id=g.current_user.id
    )

    if post.status == "published":
        post.publish()

    # Handle tags
    for tag_name in data.get("tags", []):
        tag = Tag.get_or_create(tag_name)
        post.tags.append(tag)

    db.session.add(post)
    db.session.commit()

    return jsonify({
        "success": True,
        "message": "Post created",
        "data": post_schema.dump(post)
    }), 201


# ─────────────────────────────────────────
# GET POST
# ─────────────────────────────────────────
@posts_bp.route("/<int:post_id>", methods=["GET"])
def get_post(post_id: int):
    """GET /api/v1/posts/<post_id>"""
    post = Post.query.filter(
        Post.id == post_id,
        Post.deleted_at.is_(None)
    ).first_or_404(description=f"Post {post_id} not found")

    # Check access
    current_user = get_current_user()
    if post.status != "published":
        if not current_user:
            return jsonify({"error": "Not found"}), 404
        if post.author_id != current_user.id and not current_user.is_admin:
            return jsonify({"error": "Not found"}), 404

    post.increment_views()
    db.session.commit()

    return jsonify({
        "success": True,
        "data": post_schema.dump(post)
    })


# ─────────────────────────────────────────
# UPDATE POST
# ─────────────────────────────────────────
@posts_bp.route("/<int:post_id>", methods=["PATCH"])
@auth_required
def update_post(post_id: int):
    """PATCH /api/v1/posts/<post_id>"""
    post = Post.query.filter(
        Post.id == post_id,
        Post.deleted_at.is_(None)
    ).first_or_404()

    # Authorization
    if post.author_id != g.current_user.id and not g.current_user.is_admin:
        return jsonify({"error": "Forbidden"}), 403

    try:
        data = update_schema.load(request.get_json() or {})
    except ValidationError as err:
        return jsonify({"error": "Validation failed", "details": err.messages}), 422

    # Apply updates
    if "title" in data:
        post.title = data["title"].strip()
        post.slug = post.generate_slug()

    if "content" in data:
        post.content = data["content"]

    if "status" in data:
        if data["status"] == "published" and post.status != "published":
            post.publish()
        else:
            post.status = data["status"]

    if "tags" in data and data["tags"] is not None:
        post.tags.clear()
        for tag_name in data["tags"]:
            tag = Tag.get_or_create(tag_name.lower().strip())
            post.tags.append(tag)

    db.session.commit()

    return jsonify({
        "success": True,
        "data": post_schema.dump(post)
    })


# ─────────────────────────────────────────
# DELETE POST
# ─────────────────────────────────────────
@posts_bp.route("/<int:post_id>", methods=["DELETE"])
@auth_required
def delete_post(post_id: int):
    """DELETE /api/v1/posts/<post_id>"""
    post = Post.query.filter(
        Post.id == post_id,
        Post.deleted_at.is_(None)
    ).first_or_404()

    if post.author_id != g.current_user.id and not g.current_user.is_admin:
        return jsonify({"error": "Forbidden"}), 403

    from datetime import datetime, timezone
    post.deleted_at = datetime.now(timezone.utc)
    db.session.commit()

    return "", 204   # 204 No Content


# ─────────────────────────────────────────
# PUBLISH POST
# ─────────────────────────────────────────
@posts_bp.route("/<int:post_id>/publish", methods=["POST"])
@auth_required
def publish_post(post_id: int):
    """POST /api/v1/posts/<post_id>/publish"""
    post = Post.query.filter(
        Post.id == post_id,
        Post.deleted_at.is_(None)
    ).first_or_404()

    if post.author_id != g.current_user.id and not g.current_user.is_admin:
        return jsonify({"error": "Forbidden"}), 403

    post.publish()
    db.session.commit()

    return jsonify({
        "success": True,
        "message": "Post published",
        "data": post_schema.dump(post)
    })
```

---

## 6. 🔌 Flask-RESTX (OpenAPI Docs)

Python

```
# app_restx.py — Flask-RESTX for auto-generated Swagger docs
"""
Flask-RESTX gives Flask:
  - Auto-generated Swagger/OpenAPI docs
  - Request/response marshaling
  - Namespace-based routing (like blueprints)
  - Input validation with expect() decorators
"""
from flask import Flask
from flask_restx import Api, Resource, fields, Namespace
from flask_jwt_extended import JWTManager, jwt_required, create_access_token, get_jwt_identity

app = Flask(__name__)
app.config["JWT_SECRET_KEY"] = "your-secret-key"
app.config["SECRET_KEY"] = "secret"

jwt = JWTManager(app)

# Create API with Swagger UI
api = Api(
    app,
    version="1.0",
    title="Flask Blog API",
    description="A blog API with auto-generated Swagger docs",
    doc="/docs",           # Swagger UI at /docs
    authorizations={
        "Bearer": {
            "type": "apiKey",
            "in": "header",
            "name": "Authorization",
            "description": "Enter: Bearer <your_token>"
        }
    },
    security="Bearer"
)

# Create namespaces (like blueprints)
auth_ns = Namespace("auth", description="Authentication operations")
posts_ns = Namespace("posts", description="Post operations")

api.add_namespace(auth_ns, path="/api/v1/auth")
api.add_namespace(posts_ns, path="/api/v1/posts")

# ─────────────────────────────────────────
# MODELS (for Swagger documentation)
# ─────────────────────────────────────────
login_model = auth_ns.model("LoginRequest", {
    "email": fields.String(required=True, example="alice@test.com"),
    "password": fields.String(required=True, example="SecurePass1")
})

user_model = api.model("User", {
    "id": fields.Integer(readonly=True),
    "name": fields.String(required=True, example="Alice Smith"),
    "email": fields.String(required=True, example="alice@test.com"),
    "role": fields.String(example="user"),
})

token_model = api.model("TokenResponse", {
    "access_token": fields.String(),
    "refresh_token": fields.String(),
    "token_type": fields.String(default="bearer"),
    "user": fields.Nested(user_model)
})

post_create_model = posts_ns.model("PostCreate", {
    "title": fields.String(required=True, min_length=3, example="My Post Title"),
    "content": fields.String(required=True, min_length=10),
    "status": fields.String(enum=["draft", "published"], default="draft"),
    "tags": fields.List(fields.String, example=["python", "flask"])
})

post_model = posts_ns.model("Post", {
    "id": fields.Integer(readonly=True),
    "title": fields.String(),
    "slug": fields.String(readonly=True),
    "content": fields.String(),
    "status": fields.String(),
    "view_count": fields.Integer(),
    "like_count": fields.Integer(),
    "created_at": fields.DateTime(),
})


# ─────────────────────────────────────────
# RESOURCES (class-based views)
# ─────────────────────────────────────────
@auth_ns.route("/login")
class LoginResource(Resource):
    @auth_ns.expect(login_model, validate=True)
    @auth_ns.response(200, "Success", token_model)
    @auth_ns.response(401, "Invalid credentials")
    def post(self):
        """Login with email and password."""
        data = auth_ns.payload
        email = data.get("email")
        password = data.get("password")

        # Simplified auth (use real DB lookup in production)
        if email == "admin@test.com" and password == "admin123":
            access_token = create_access_token(identity=1)
            return {
                "access_token": access_token,
                "token_type": "bearer"
            }, 200

        return {"error": "Invalid credentials"}, 401


@posts_ns.route("")
class PostList(Resource):
    @posts_ns.doc("list_posts")
    @posts_ns.param("page", "Page number", type=int, default=1)
    @posts_ns.param("per_page", "Items per page", type=int, default=20)
    @posts_ns.response(200, "Success")
    def get(self):
        """List all published posts."""
        page = int(posts_ns.payload.get("page", 1)) if posts_ns.payload else 1
        return {
            "data": [],
            "pagination": {"total": 0, "page": page, "per_page": 20}
        }

    @posts_ns.doc("create_post", security="Bearer")
    @posts_ns.expect(post_create_model, validate=True)
    @posts_ns.response(201, "Created", post_model)
    @posts_ns.response(401, "Unauthorized")
    @jwt_required()
    def post(self):
        """Create a new post. Requires authentication."""
        data = posts_ns.payload
        user_id = get_jwt_identity()
        return {
            "id": 1,
            "title": data["title"],
            "status": data.get("status", "draft"),
            "message": "Post created"
        }, 201


@posts_ns.route("/<int:post_id>")
@posts_ns.param("post_id", "The post ID")
class PostDetail(Resource):
    @posts_ns.doc("get_post")
    @posts_ns.response(200, "Success", post_model)
    @posts_ns.response(404, "Post not found")
    def get(self, post_id: int):
        """Get a specific post by ID."""
        return {"id": post_id, "title": f"Post {post_id}"}

    @posts_ns.doc(security="Bearer")
    @posts_ns.expect(post_create_model)
    @jwt_required()
    def patch(self, post_id: int):
        """Update a post. Requires authentication."""
        return {"id": post_id, "updated": True}

    @posts_ns.doc(security="Bearer")
    @posts_ns.response(204, "Deleted")
    @jwt_required()
    def delete(self, post_id: int):
        """Delete a post."""
        return "", 204


if __name__ == "__main__":
    print("Swagger UI: http://localhost:5000/docs")
    app.run(debug=True)
```

---

## 7. 🔧 Flask Middleware

Python

```
# app/middleware.py
from flask import Flask, request, g
import time
import uuid
import logging

logger = logging.getLogger(__name__)


def register_middleware(app: Flask) -> None:
    """Register all middleware as before/after request hooks."""

    @app.before_request
    def start_timer():
        """Record start time for each request."""
        g.start_time = time.perf_counter()
        g.request_id = str(uuid.uuid4())[:8]

    @app.before_request
    def log_request():
        """Log incoming requests."""
        logger.info(
            f"[{g.request_id}] → {request.method} {request.path}"
            f" from {request.remote_addr}"
        )

    @app.after_request
    def add_headers(response):
        """Add standard headers to all responses."""
        if hasattr(g, "start_time"):
            duration = (time.perf_counter() - g.start_time) * 1000
            response.headers["X-Response-Time"] = f"{duration:.2f}ms"

        if hasattr(g, "request_id"):
            response.headers["X-Request-ID"] = g.request_id

        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"

        if hasattr(g, "start_time"):
            logger.info(
                f"[{g.request_id}] ← {response.status_code}"
                f" {(time.perf_counter()-g.start_time)*1000:.2f}ms"
            )

        return response

    @app.teardown_request
    def cleanup(error=None):
        """Cleanup after request (even if error)."""
        if error:
            logger.error(f"Request teardown error: {error}")
```

---

## 8. 🔌 Flask Context Variables

Python

```
# Flask has special context variables
# These are "global" but actually per-request

from flask import (
    g,              # request-scoped storage (like request state in FastAPI)
    request,        # current request object
    current_app,    # current Flask application
    session,        # server-side session (cookie-based)
    url_for,        # generate URLs from endpoint names
    redirect,       # redirect response
    abort,          # raise HTTP errors
    flash,          # flash messages (for templates)
    render_template # render Jinja2 templates (not for APIs)
)


# g — request-scoped storage
@app.before_request
def load_user():
    """Load user once and store in g for this request."""
    from flask_jwt_extended import verify_jwt_in_request, get_jwt_identity
    try:
        verify_jwt_in_request(optional=True)
        user_id = get_jwt_identity()
        if user_id:
            g.current_user = User.find_by_id(user_id)
        else:
            g.current_user = None
    except Exception:
        g.current_user = None


# In any route/function during this request:
def some_function():
    user = g.current_user   # set in before_request
    app = current_app       # the Flask app instance
    req = request           # the current request


# url_for — generate URLs
def example():
    # Generate URL for a route
    login_url = url_for("auth.login")              # blueprint.endpoint
    post_url = url_for("posts.get_post", post_id=42)  # with params
    external = url_for("auth.login", _external=True)  # full URL

    return redirect(url_for("auth.me"))  # redirect to /api/v1/auth/me


# abort — raise HTTP errors
def check_permission(user):
    if not user:
        abort(401)  # raises 401 error, handled by error handler
    if not user.is_admin:
        abort(403, description="Admin access required")
    if not user.is_verified:
        abort(403, description="Email verification required")
```

---

## 9. 🧪 Testing Flask

Python

```
# tests/test_app.py
import pytest
from app import create_app, db as _db
from app.models.user import User


@pytest.fixture(scope="session")
def app():
    """Create test application."""
    app = create_app("testing")
    return app


@pytest.fixture(scope="session")
def _database(app):
    """Create test database."""
    with app.app_context():
        _db.create_all()
        yield _db
        _db.drop_all()


@pytest.fixture(autouse=True)
def db(_database, app):
    """Rollback after each test."""
    with app.app_context():
        yield _database
        _database.session.rollback()


@pytest.fixture
def client(app):
    """Test client."""
    return app.test_client()


@pytest.fixture
def auth_client(client, app):
    """Authenticated test client."""
    with app.app_context():
        # Create user
        user = User(name="Test User", email="test@test.com")
        user.set_password("SecurePass1!")
        _db.session.add(user)
        _db.session.commit()

    # Login
    response = client.post("/api/v1/auth/login", json={
        "email": "test@test.com",
        "password": "SecurePass1!"
    })
    token = response.json["data"]["access_token"]

    # Set auth header
    client.environ_base["HTTP_AUTHORIZATION"] = f"Bearer {token}"
    return client, user


class TestAuth:
    def test_register_success(self, client):
        response = client.post("/api/v1/auth/register", json={
            "name": "Alice Smith",
            "email": "alice@test.com",
            "password": "SecurePass1!"
        })
        assert response.status_code == 201
        data = response.json
        assert data["success"] is True
        assert "access_token" in data["data"]

    def test_register_duplicate_email(self, client, app):
        with app.app_context():
            user = User(name="Existing", email="existing@test.com")
            user.set_password("SecurePass1!")
            _db.session.add(user)
            _db.session.commit()

        response = client.post("/api/v1/auth/register", json={
            "name": "New User",
            "email": "existing@test.com",
            "password": "SecurePass1!"
        })
        assert response.status_code == 422

    def test_login_wrong_password(self, client, auth_client):
        response = client.post("/api/v1/auth/login", json={
            "email": "test@test.com",
            "password": "WrongPassword1!"
        })
        assert response.status_code == 401

    def test_me_authenticated(self, auth_client):
        client, user = auth_client
        response = client.get("/api/v1/auth/me")
        assert response.status_code == 200
        assert response.json["data"]["email"] == user.email

    def test_me_unauthenticated(self, client):
        response = client.get("/api/v1/auth/me")
        assert response.status_code == 401


class TestPosts:
    def test_list_posts_public(self, client):
        response = client.get("/api/v1/posts")
        assert response.status_code == 200
        assert "data" in response.json
        assert "pagination" in response.json

    def test_create_post_auth_required(self, client):
        response = client.post("/api/v1/posts", json={
            "title": "Test Post",
            "content": "Test content here"
        })
        assert response.status_code == 401

    def test_create_post_success(self, auth_client):
        client, user = auth_client
        response = client.post("/api/v1/posts", json={
            "title": "My Flask Post",
            "content": "Content of my flask post here.",
            "tags": ["flask", "python"]
        })
        assert response.status_code == 201
        assert response.json["data"]["title"] == "My Flask Post"

    def test_delete_post_not_owner(self, client, auth_client, app):
        auth_c, user = auth_client

        # Create post as user1
        post_response = auth_c.post("/api/v1/posts", json={
            "title": "User1 Post",
            "content": "Some content here."
        })
        post_id = post_response.json["data"]["id"]

        # Create user2 and try to delete
        with app.app_context():
            user2 = User(name="User Two", email="user2@test.com")
            user2.set_password("SecurePass1!")
            _db.session.add(user2)
            _db.session.commit()

        login = client.post("/api/v1/auth/login", json={
            "email": "user2@test.com",
            "password": "SecurePass1!"
        })
        token2 = login.json["data"]["access_token"]

        response = client.delete(
            f"/api/v1/posts/{post_id}",
            headers={"Authorization": f"Bearer {token2}"}
        )
        assert response.status_code == 403


# Run: pytest tests/ -v
```

---

## 10. 🔧 Why Companies Still Use Flask

Python

```
"""
Flask use cases you'll encounter in real jobs:

1. MICROSERVICES
   Flask's minimal size makes it perfect for
   small, focused services:
   
   payment-service/    ← Flask app, handles payments only
   notification-service/ ← Flask app, sends emails/SMS
   image-processing/   ← Flask app, resizes images
   
   Each service is small, fast, independently deployable.

2. ML MODEL SERVING
   Data scientists love Flask for exposing ML models:
   
   from flask import Flask, request, jsonify
   import pickle
   
   app = Flask(__name__)
   model = pickle.load(open("model.pkl", "rb"))
   
   @app.route("/predict", methods=["POST"])
   def predict():
       features = request.json["features"]
       prediction = model.predict([features])
       return jsonify({"prediction": prediction[0]})
   
   Many ML deployments at companies use Flask for this.

3. INTERNAL TOOLS & SCRIPTS
   Quick admin tools, data pipelines, internal dashboards:
   
   from flask import Flask
   app = Flask(__name__)
   
   @app.route("/run-report")
   def run_report():
       result = generate_monthly_report()
       return jsonify(result)
   
   Much simpler than setting up Django for a small tool.

4. LEGACY CODEBASES
   Companies started 5-10 years ago often have Flask.
   You'll be maintaining and adding to existing Flask apps.

5. PROTOTYPING
   Get an API running in 10 minutes.
   Test an idea before building the full product.

BOTTOM LINE:
  Flask is NOT dying. It's finding its niche:
  microservices, ML serving, internal tools, legacy apps.
  FastAPI is taking new greenfield API projects.
  But Flask knowledge is still highly valuable.
"""
```

---

## 11. 🎯 Flask vs FastAPI vs Django Summary

Python

```
# Complete comparison for making real decisions

comparison = {
    "Flask": {
        "best_for": [
            "Microservices (tiny, focused)",
            "ML model serving",
            "Internal tools",
            "Prototyping",
            "Legacy maintenance",
            "Learning web frameworks",
        ],
        "strengths": [
            "Extremely simple and minimal",
            "Maximum flexibility",
            "Large ecosystem",
            "Easy to learn",
            "Great for small apps",
        ],
        "weaknesses": [
            "No built-in validation",
            "No auto docs (need Flask-RESTX)",
            "Synchronous by default",
            "Everything manual",
            "Easy to make messy apps",
        ],
        "example_companies": ["Pinterest (old), Netflix (some services)"],
        "job_market": "Large (existing codebases) + ML companies",
    },

    "FastAPI": {
        "best_for": [
            "New REST APIs",
            "High performance needed",
            "Async operations",
            "Microservices (modern)",
            "When type safety matters",
        ],
        "strengths": [
            "Automatic validation (Pydantic)",
            "Auto OpenAPI docs",
            "Async-first",
            "Type hints throughout",
            "Fast (near Node.js speed)",
        ],
        "weaknesses": [
            "No built-in admin panel",
            "Younger ecosystem",
            "No built-in ORM",
            "Manual session management",
        ],
        "example_companies": ["Netflix, Uber, Microsoft"],
        "job_market": "Fastest growing, premium salaries",
    },

    "Django + DRF": {
        "best_for": [
            "Full-featured web applications",
            "When admin panel needed",
            "Large teams",
            "Enterprise applications",
            "Complex business logic",
        ],
        "strengths": [
            "Batteries included",
            "Built-in admin panel",
            "Built-in ORM, auth, migrations",
            "Massive ecosystem",
            "Very structured",
        ],
        "weaknesses": [
            "Slow startup, heavier",
            "Less flexible",
            "Steeper learning curve",
            "Not async-first",
        ],
        "example_companies": ["Instagram, Pinterest, Mozilla, Disqus"],
        "job_market": "Largest (40%+ of Python backend jobs)",
    }
}

for framework, info in comparison.items():
    print(f"\n{'='*50}")
    print(f"  {framework}")
    print(f"  Best for: {', '.join(info['best_for'][:3])}")
    print(f"  Job market: {info['job_market']}")
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                        FLASK ESSENTIALS                        │
│                                                                │
│  CORE CONCEPTS:                                                │
│  Flask()               → create app                           │
│  @app.route()          → define URL endpoint                  │
│  request               → access request data                  │
│  jsonify()             → create JSON response                 │
│  Blueprint             → modular route organization            │
│  g                     → request-scoped storage               │
│  before_request        → middleware equivalent                │
│  after_request         → response middleware                  │
│                                                                │
│  EXTENSIONS USED:                                              │
│  Flask-SQLAlchemy      → ORM (same as SQLAlchemy)            │
│  Flask-Migrate         → Alembic migrations                   │
│  Flask-JWT-Extended    → JWT authentication                   │
│  Flask-CORS            → CORS headers                         │
│  Flask-RESTX           → Swagger/OpenAPI docs                 │
│  Marshmallow           → validation and serialization         │
│                                                                │
│  APP FACTORY PATTERN:                                          │
│  def create_app():                                             │
│      app = Flask(__name__)                                     │
│      db.init_app(app)                                          │
│      jwt.init_app(app)                                         │
│      app.register_blueprint(auth_bp)                          │
│      return app                                                │
│                                                                │
│  REQUEST LIFECYCLE:                                            │
│  before_request → route handler → after_request               │
│  teardown_request (always runs, even on error)                │
│                                                                │
│  WHEN TO USE FLASK:                                            │
│  Microservices • ML serving • Legacy apps                     │
│  Internal tools • Prototyping • Simple APIs                   │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is the "application factory" pattern and why use it?
2.  What is a Blueprint? How is it similar to FastAPI's APIRouter?
3.  What is Flask's g object used for?
4.  What are before_request and after_request decorators?
5.  How does Flask handle request data (JSON, form, files)?
6.  What is Marshmallow and how does it compare to Pydantic?
7.  How does Flask-JWT-Extended work?
8.  What is Flask-RESTX and why use it?
9.  What is first_or_404() in Flask-SQLAlchemy?
10. How do you test Flask applications?
11. When would you choose Flask over FastAPI?
12. What does abort() do in Flask?
```

---

## 🛠️ Practice Exercises

Python

```
# Exercise 1: Flask Middleware
# Create a middleware that:
# - Logs all requests (method, path, IP, duration)
# - Adds X-Request-ID to every response
# - Adds X-Response-Time to every response
# - Rate limits to 100 requests/minute per IP
#   using an in-memory dict (or Redis)

# Exercise 2: File Upload Endpoint
# Build a file upload endpoint:
# POST /api/v1/upload
# - Accepts: jpg, png, gif, pdf
# - Max size: 10MB
# - Saves to uploads/ directory with UUID filename
# - Returns file URL
# - Requires authentication
# - GET /api/v1/files/<filename> to download

# Exercise 3: Pagination Helper
# Create a reusable pagination function:
# def paginate_query(query, page, per_page=20, max_per_page=100):
#     Returns dict with: items, total, page, per_page,
#                        total_pages, has_next, has_prev,
#                        next_url, prev_url
# Use request.url to build next/prev URLs
# Apply it to the posts list endpoint

# Exercise 4: Flask CLI Commands
# Add these management commands:
# flask create-admin <email> <password> → create admin user
# flask seed-data --users 10 --posts 50 → seed test data
# flask clear-data → delete all data (with confirmation)
# flask show-routes → display all registered routes

# Exercise 5: Convert to Flask-RESTX
# Take your posts blueprint and convert it to Flask-RESTX:
# - Create namespaces for auth and posts
# - Add Swagger models for all request/response bodies
# - Add @expect decorators for validation
# - Add @response decorators for documentation
# - Test the auto-generated Swagger UI
```

---

## ✅ Phase 4.4 Complete!

**You now know Flask:**

text

```
✅ Flask routing and HTTP methods
✅ Request/response handling
✅ Application factory pattern
✅ Blueprints for modular code
✅ Flask-SQLAlchemy models
✅ Flask-Migrate (Alembic)
✅ Flask-JWT-Extended (auth)
✅ Marshmallow (validation)
✅ Flask-RESTX (Swagger docs)
✅ Before/after request middleware
✅ Error handlers
✅ Flask context variables (g, request, current_app)
✅ Testing Flask apps
✅ When to use Flask vs FastAPI vs Django
```