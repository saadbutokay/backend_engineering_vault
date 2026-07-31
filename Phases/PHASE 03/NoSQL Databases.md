```
You now know PostgreSQL (relational database).
Now learn when and why to use different types of databases.

The backend engineer's toolkit:
┌─────────────────────────────────────────────────────────┐
│  PostgreSQL  → Structured data, relationships, ACID     │
│  Redis       → Caching, sessions, real-time, queues     │
│  MongoDB     → Flexible documents, rapidly changing     │
│               schema, nested data                       │
└─────────────────────────────────────────────────────────┘

Real production systems use ALL of these together.

Example: Instagram's backend
  PostgreSQL → user profiles, followers, relationships
  Redis      → feed caching, session storage, rate limiting
  Cassandra  → activity timeline (similar to MongoDB use case)
```

---

## 1. 🧠 NoSQL Concepts

### Why NoSQL Exists

text

```
Relational databases have trade-offs:

STRENGTHS:
  ✅ ACID guarantees
  ✅ Complex joins and relationships
  ✅ Structured schema enforced
  ✅ Mature tooling and expertise

WEAKNESSES:
  ❌ Schema changes are painful (migrations required)
  ❌ Horizontal scaling is hard
  ❌ Not great for all data shapes
  ❌ Can be slow for simple key-value operations

NoSQL databases trade some of these guarantees
for specific advantages in specific use cases.
```

### Types of NoSQL Databases

text

```
┌───────────────────┬────────────────────┬──────────────────────┐
│ Type              │ Example            │ Best For             │
├───────────────────┼────────────────────┼──────────────────────┤
│ Key-Value         │ Redis, DynamoDB    │ Caching, sessions,   │
│                   │                    │ simple fast lookup   │
├───────────────────┼────────────────────┼──────────────────────┤
│ Document          │ MongoDB, CouchDB   │ Flexible schema,     │
│                   │                    │ nested data, content │
├───────────────────┼────────────────────┼──────────────────────┤
│ Column-Family     │ Cassandra, HBase   │ Time-series, large   │
│                   │                    │ scale writes         │
├───────────────────┼────────────────────┼──────────────────────┤
│ Graph             │ Neo4j, ArangoDB    │ Social networks,     │
│                   │                    │ recommendations      │
└───────────────────┴────────────────────┴──────────────────────┘

In this phase we cover:
  Redis   → key-value (most important for backend engineers)
  MongoDB → document (most common NoSQL in Python backends)
```

### The CAP Theorem

text

```
CAP Theorem: A distributed database can only guarantee
             TWO of these three properties:

C — Consistency
    Every read receives the most recent write.
    All nodes see the same data at the same time.

A — Availability
    Every request receives a response.
    System always responds (even if not up to date).

P — Partition Tolerance
    System continues working despite network failures.
    Required for any distributed system.

Since P is required (networks fail), you choose:

CP: Consistent + Partition tolerant
    May be unavailable during partition.
    Example: PostgreSQL, Redis (with cluster), MongoDB

AP: Available + Partition tolerant
    May return stale data during partition.
    Example: Cassandra, DynamoDB (eventually consistent)

PostgreSQL: CP (strong consistency, may be unavailable)
Redis:      CP for clusters, CA for single instance
MongoDB:    Configurable (default: CP)
Cassandra:  AP (available, eventually consistent)
```

---

## 2. 🔴 Redis — The Swiss Army Knife

### What is Redis?

text

```
Redis = Remote Dictionary Server

An in-memory data structure store.
Think: a dict that:
  ✅ Persists to disk (survives restarts if configured)
  ✅ Is accessible from multiple servers
  ✅ Has rich data types beyond simple strings
  ✅ Has built-in expiration (TTL)
  ✅ Operates at microsecond speed
  ✅ Handles millions of operations per second

Use cases in backend:
  1. Caching           → cache expensive DB queries
  2. Sessions          → store JWT/session data
  3. Rate limiting     → count requests per user
  4. Pub/Sub           → real-time messaging
  5. Job queues        → background task queuing
  6. Leaderboards      → sorted sets for rankings
  7. Distributed locks → prevent race conditions
  8. Feature flags     → toggle features without deploy
```

### Installing Redis

Bash

```
# Mac:
brew install redis
brew services start redis

# Linux (Ubuntu):
sudo apt update
sudo apt install redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server

# Verify:
redis-cli ping
# PONG

# Windows: Use WSL2 or Docker
docker run -d -p 6379:6379 redis:alpine

# Install Python client:
pip install redis python-dotenv
```

### Redis Data Types and Commands

Bash

```
# ─────────────────────────────────────────
# Connect to Redis CLI
# ─────────────────────────────────────────
redis-cli

# ─────────────────────────────────────────
# STRINGS — simplest type
# ─────────────────────────────────────────
SET name "Alice"           # set a key
GET name                   # get a key → "Alice"
DEL name                   # delete a key

SET counter 0
INCR counter               # atomic increment → 1
INCRBY counter 5           # increment by N → 6
DECR counter               # decrement → 5
DECRBY counter 2           # decrement by N → 3

# With expiration (TTL = Time To Live)
SET session_token "abc123" EX 3600    # expires in 3600 seconds (1 hour)
SET otp "123456" EX 300               # expires in 5 minutes
TTL session_token                     # check remaining time
PERSIST session_token                 # remove expiration

# ─────────────────────────────────────────
# HASHES — field:value pairs (like a dict)
# ─────────────────────────────────────────
HSET user:1 name "Alice" email "alice@test.com" age 25
HGET user:1 name               # → "Alice"
HMGET user:1 name email        # → "Alice", "alice@test.com"
HGETALL user:1                 # → all fields and values
HDEL user:1 age                # delete a field
HEXISTS user:1 email           # → 1 (exists)
HLEN user:1                    # → number of fields

# ─────────────────────────────────────────
# LISTS — ordered collection (linked list)
# ─────────────────────────────────────────
RPUSH tasks "task1" "task2"    # push to right (end)
LPUSH tasks "task0"            # push to left (front)
LRANGE tasks 0 -1              # get all: → task0, task1, task2
LLEN tasks                     # length → 3
LPOP tasks                     # pop from left → "task0"
RPOP tasks                     # pop from right → "task2"
LINDEX tasks 0                 # get by index

# Blocking pop (wait until item available)
BLPOP queue 30                 # wait up to 30 seconds

# ─────────────────────────────────────────
# SETS — unordered, unique values
# ─────────────────────────────────────────
SADD tags "python" "fastapi" "redis"
SADD tags "python"              # duplicate ignored
SMEMBERS tags                   # → {python, fastapi, redis}
SISMEMBER tags "python"         # → 1 (is member)
SCARD tags                      # → 3 (size)
SREM tags "redis"               # remove member

# Set operations
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"
SUNION set1 set2                # → {a, b, c, d}
SINTER set1 set2                # → {b, c}
SDIFF set1 set2                 # → {a} (in set1 but not set2)

# ─────────────────────────────────────────
# SORTED SETS — ordered by score
# ─────────────────────────────────────────
ZADD leaderboard 100 "alice"
ZADD leaderboard 85 "bob"
ZADD leaderboard 95 "charlie"
ZADD leaderboard 85 "dave"

ZRANGE leaderboard 0 -1 WITHSCORES    # all, ascending score
ZREVRANGE leaderboard 0 -1 WITHSCORES # all, descending score
ZRANK leaderboard "alice"              # rank (0-indexed, ascending)
ZREVRANK leaderboard "alice"           # rank from top → 0
ZSCORE leaderboard "alice"             # → 100
ZINCRBY leaderboard 5 "bob"            # increment bob's score
ZRANGEBYSCORE leaderboard 85 100       # by score range

# ─────────────────────────────────────────
# COMMON PATTERNS
# ─────────────────────────────────────────
# Check if key exists
EXISTS user:1                   # → 1 (yes) or 0 (no)

# Get all keys matching pattern (careful in production!)
KEYS user:*                     # → user:1, user:2, ...
SCAN 0 MATCH user:* COUNT 100  # safer: use SCAN instead

# Key expiration
EXPIRE key 3600                 # set expiry in seconds
EXPIREAT key 1705312800         # set expiry as Unix timestamp
TTL key                         # remaining time (-1=no expire, -2=not exist)

# Type of a key
TYPE user:1                     # → hash

# Rename a key
RENAME old_key new_key
```

### Redis with Python

Python

```
# src/redis_client.py
import redis
import json
from typing import Any, Optional, List, Dict, Union
from datetime import timedelta
import os
from dotenv import load_dotenv
from contextlib import contextmanager
from functools import wraps
import time

load_dotenv()

REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379/0")

# ─────────────────────────────────────────
# CLIENT SETUP
# ─────────────────────────────────────────
# Create a Redis client (connection pool built-in)
redis_client = redis.from_url(
    REDIS_URL,
    decode_responses=True,   # return strings, not bytes
    socket_connect_timeout=5,
    socket_timeout=5,
    retry_on_timeout=True,
    health_check_interval=30,
)

# Test connection
redis_client.ping()
print("Redis connected successfully!")


# ─────────────────────────────────────────
# 1. CACHING PATTERN
# ─────────────────────────────────────────
class CacheService:
    """
    Generic caching service using Redis.
    Implements cache-aside pattern (most common).
    """

    def __init__(self, client: redis.Redis, prefix: str = "cache"):
        self.client = client
        self.prefix = prefix
        self.default_ttl = 3600  # 1 hour

    def _key(self, key: str) -> str:
        return f"{self.prefix}:{key}"

    def get(self, key: str) -> Optional[Any]:
        """Get value from cache. Returns None if not found."""
        data = self.client.get(self._key(key))
        if data is None:
            return None
        try:
            return json.loads(data)
        except json.JSONDecodeError:
            return data

    def set(
        self,
        key: str,
        value: Any,
        ttl: Optional[int] = None
    ) -> bool:
        """Set value in cache with optional TTL (seconds)."""
        serialized = json.dumps(value, default=str)
        expire = ttl or self.default_ttl
        return self.client.setex(
            self._key(key),
            expire,
            serialized
        )

    def delete(self, key: str) -> bool:
        """Remove key from cache."""
        return bool(self.client.delete(self._key(key)))

    def exists(self, key: str) -> bool:
        """Check if key exists in cache."""
        return bool(self.client.exists(self._key(key)))

    def invalidate_pattern(self, pattern: str) -> int:
        """
        Delete all keys matching pattern.
        Use carefully — SCAN is safe, KEYS is not for production.
        """
        count = 0
        cursor = 0
        match_pattern = self._key(pattern)
        while True:
            cursor, keys = self.client.scan(
                cursor,
                match=f"{match_pattern}*",
                count=100
            )
            if keys:
                self.client.delete(*keys)
                count += len(keys)
            if cursor == 0:
                break
        return count

    def get_or_set(
        self,
        key: str,
        factory,  # callable that returns the value
        ttl: Optional[int] = None
    ) -> Any:
        """
        Cache-aside pattern:
        1. Check cache
        2. If miss: call factory, store result, return it
        3. If hit: return cached value
        """
        cached = self.get(key)
        if cached is not None:
            return cached

        # Cache miss: compute value
        value = factory()
        self.set(key, value, ttl)
        return value


# Usage
cache = CacheService(redis_client, prefix="myapp")

# Simulate expensive database query
def get_user_from_db(user_id: int) -> dict:
    time.sleep(0.1)  # simulate DB query
    return {"id": user_id, "name": "Alice", "email": "alice@test.com"}

# With caching
def get_user(user_id: int) -> dict:
    cache_key = f"user:{user_id}"

    return cache.get_or_set(
        cache_key,
        factory=lambda: get_user_from_db(user_id),
        ttl=300  # 5 minutes
    )

# First call: cache miss (slow)
start = time.perf_counter()
user = get_user(1)
print(f"First call: {(time.perf_counter()-start)*1000:.0f}ms (DB query)")

# Second call: cache hit (fast)
start = time.perf_counter()
user = get_user(1)
print(f"Second call: {(time.perf_counter()-start)*1000:.0f}ms (cache)")


# ─────────────────────────────────────────
# DECORATOR: Cache results of a function
# ─────────────────────────────────────────
def cache_result(ttl: int = 300, key_prefix: str = ""):
    """Decorator that caches function results in Redis."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Build cache key from function name and arguments
            key_parts = [key_prefix or func.__name__]
            key_parts.extend(str(a) for a in args)
            key_parts.extend(f"{k}={v}" for k, v in sorted(kwargs.items()))
            cache_key = ":".join(key_parts)

            # Check cache
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)

            # Call function
            result = func(*args, **kwargs)

            # Store in cache
            redis_client.setex(
                cache_key,
                ttl,
                json.dumps(result, default=str)
            )
            return result

        # Add method to manually invalidate
        def invalidate(*args, **kwargs):
            key_parts = [key_prefix or func.__name__]
            key_parts.extend(str(a) for a in args)
            key_parts.extend(f"{k}={v}" for k, v in sorted(kwargs.items()))
            cache_key = ":".join(key_parts)
            redis_client.delete(cache_key)

        wrapper.invalidate = invalidate
        return wrapper
    return decorator


@cache_result(ttl=600, key_prefix="product")
def get_product(product_id: int) -> dict:
    """This expensive function is automatically cached."""
    time.sleep(0.05)  # simulate work
    return {"id": product_id, "name": "Laptop", "price": 999.99}

# Cached automatically
product = get_product(42)
product_again = get_product(42)  # from cache

# Invalidate when product changes
get_product.invalidate(42)
```

Python

```
# ─────────────────────────────────────────
# 2. SESSION STORAGE
# ─────────────────────────────────────────
import secrets
from datetime import datetime, timezone

class SessionStore:
    """
    Store user sessions in Redis.
    Better than database for sessions:
      - Much faster (memory vs disk)
      - Built-in TTL (no cleanup needed)
      - Shared across multiple servers
    """

    SESSION_TTL = 7 * 24 * 3600    # 7 days
    PREFIX = "session"

    def __init__(self, client: redis.Redis):
        self.client = client

    def create_session(
        self,
        user_id: int,
        metadata: Optional[Dict] = None
    ) -> str:
        """Create a new session and return the token."""
        token = secrets.token_urlsafe(32)
        session_data = {
            "user_id": user_id,
            "created_at": datetime.now(timezone.utc).isoformat(),
            "metadata": metadata or {}
        }
        self.client.setex(
            f"{self.PREFIX}:{token}",
            self.SESSION_TTL,
            json.dumps(session_data)
        )
        return token

    def get_session(self, token: str) -> Optional[Dict]:
        """Get session data by token."""
        data = self.client.get(f"{self.PREFIX}:{token}")
        if not data:
            return None
        return json.loads(data)

    def refresh_session(self, token: str) -> bool:
        """Extend session TTL."""
        return bool(
            self.client.expire(f"{self.PREFIX}:{token}", self.SESSION_TTL)
        )

    def delete_session(self, token: str) -> bool:
        """Logout: delete session."""
        return bool(self.client.delete(f"{self.PREFIX}:{token}"))

    def delete_all_user_sessions(self, user_id: int) -> int:
        """Logout from all devices."""
        count = 0
        cursor = 0
        while True:
            cursor, keys = self.client.scan(
                cursor,
                match=f"{self.PREFIX}:*",
                count=100
            )
            for key in keys:
                data = self.client.get(key)
                if data:
                    session = json.loads(data)
                    if session.get("user_id") == user_id:
                        self.client.delete(key)
                        count += 1
            if cursor == 0:
                break
        return count

    def get_ttl(self, token: str) -> int:
        """Get remaining TTL for session."""
        return self.client.ttl(f"{self.PREFIX}:{token}")


# Usage
store = SessionStore(redis_client)

token = store.create_session(
    user_id=42,
    metadata={"ip": "192.168.1.1", "device": "Chrome/Mac"}
)
print(f"Session token: {token[:20]}...")

session = store.get_session(token)
print(f"Session: user_id={session['user_id']}")

ttl = store.get_ttl(token)
print(f"TTL: {ttl} seconds ({ttl // 3600} hours)")

store.refresh_session(token)  # extend
store.delete_session(token)   # logout
```

Python

```
# ─────────────────────────────────────────
# 3. RATE LIMITING
# ─────────────────────────────────────────
class RateLimiter:
    """
    Redis-based rate limiter.
    Works across multiple servers (unlike in-memory).

    Algorithm: Sliding window with Redis.
    """

    def __init__(self, client: redis.Redis):
        self.client = client

    def is_allowed(
        self,
        identifier: str,      # e.g., "user:42" or "ip:192.168.1.1"
        limit: int,           # max requests
        window: int,          # time window in seconds
        action: str = "api"   # what action to rate limit
    ) -> Dict[str, Any]:
        """
        Check if request is allowed.
        Returns dict with:
          allowed: bool
          remaining: int (requests left)
          reset_at: int (Unix timestamp when limit resets)
        """
        key = f"ratelimit:{action}:{identifier}"
        now = int(time.time())
        window_start = now - window

        pipe = self.client.pipeline()

        # Remove expired entries
        pipe.zremrangebyscore(key, 0, window_start)

        # Count current window
        pipe.zcard(key)

        # Add current request
        pipe.zadd(key, {str(now): now})

        # Set key expiration
        pipe.expire(key, window)

        results = pipe.execute()
        current_count = results[1]

        allowed = current_count < limit
        remaining = max(0, limit - current_count - (1 if allowed else 0))

        return {
            "allowed": allowed,
            "remaining": remaining,
            "limit": limit,
            "window": window,
            "reset_at": now + window
        }

    def reset(self, identifier: str, action: str = "api") -> None:
        """Reset rate limit for an identifier."""
        self.client.delete(f"ratelimit:{action}:{identifier}")


# Usage
limiter = RateLimiter(redis_client)

# Simulate API endpoint
def api_endpoint(user_id: int) -> Dict:
    result = limiter.is_allowed(
        identifier=f"user:{user_id}",
        limit=5,         # 5 requests
        window=60,       # per minute
        action="api"
    )

    if not result["allowed"]:
        return {
            "error": "Rate limit exceeded",
            "retry_after": result["reset_at"],
            "limit": result["limit"]
        }

    return {
        "data": "your api response",
        "requests_remaining": result["remaining"]
    }

# Test rate limiting
for i in range(7):
    response = api_endpoint(user_id=42)
    if "error" in response:
        print(f"Request {i+1}: ❌ {response['error']}")
    else:
        print(f"Request {i+1}: ✅ remaining={response['requests_remaining']}")
```

Python

```
# ─────────────────────────────────────────
# 4. PUB/SUB — Real-time messaging
# ─────────────────────────────────────────
"""
Publisher sends messages to a channel.
Subscribers receive messages in real-time.

Use cases:
  - Real-time notifications
  - Chat messages
  - Live dashboards
  - Event broadcasting across servers
"""
import threading

def demo_pubsub():
    # PUBLISHER
    publisher = redis.from_url(REDIS_URL, decode_responses=True)

    # SUBSCRIBER
    subscriber = redis.from_url(REDIS_URL, decode_responses=True)
    pubsub = subscriber.pubsub()

    # Subscribe to channels
    messages_received = []

    def message_handler(message):
        if message["type"] == "message":
            data = json.loads(message["data"])
            messages_received.append(data)
            print(f"[SUBSCRIBER] Received: {data}")

    pubsub.subscribe(**{
        "notifications:user:42": message_handler,
        "notifications:broadcast": message_handler
    })

    # Start listening in background thread
    thread = pubsub.run_in_thread(sleep_time=0.1, daemon=True)
    time.sleep(0.1)  # Let subscriber start

    # PUBLISH messages
    def publish_notification(user_id: int, event: str, data: dict):
        channel = f"notifications:user:{user_id}"
        message = json.dumps({
            "event": event,
            "data": data,
            "timestamp": datetime.now(timezone.utc).isoformat()
        })
        publisher.publish(channel, message)
        print(f"[PUBLISHER] Published to {channel}: {event}")

    publish_notification(42, "new_message", {"from": "Alice", "text": "Hello!"})
    publish_notification(42, "order_shipped", {"order_id": 123})

    # Broadcast to all subscribers
    publisher.publish("notifications:broadcast", json.dumps({
        "event": "system_maintenance",
        "data": {"message": "Maintenance in 1 hour"}
    }))

    time.sleep(0.5)  # Wait for messages
    thread.stop()
    print(f"Messages received: {len(messages_received)}")


demo_pubsub()


# ─────────────────────────────────────────
# 5. LEADERBOARD with Sorted Sets
# ─────────────────────────────────────────
class Leaderboard:
    """
    Real-time leaderboard using Redis Sorted Sets.
    O(log n) for all operations.
    """

    def __init__(self, client: redis.Redis, name: str):
        self.client = client
        self.key = f"leaderboard:{name}"

    def add_score(self, user_id: int, score: float) -> None:
        """Add or update score. O(log n)."""
        self.client.zadd(self.key, {str(user_id): score})

    def increment_score(self, user_id: int, delta: float) -> float:
        """Increment score and return new score. O(log n)."""
        return self.client.zincrby(self.key, delta, str(user_id))

    def get_rank(self, user_id: int) -> Optional[int]:
        """Get user's rank (1-indexed). O(log n)."""
        rank = self.client.zrevrank(self.key, str(user_id))
        return rank + 1 if rank is not None else None

    def get_score(self, user_id: int) -> Optional[float]:
        """Get user's score. O(log n)."""
        score = self.client.zscore(self.key, str(user_id))
        return float(score) if score is not None else None

    def get_top(self, n: int = 10) -> List[Dict]:
        """Get top N users. O(log n + n)."""
        results = self.client.zrevrange(
            self.key, 0, n - 1, withscores=True
        )
        return [
            {"rank": i + 1, "user_id": int(user_id), "score": score}
            for i, (user_id, score) in enumerate(results)
        ]

    def get_around_user(
        self, user_id: int, context: int = 5
    ) -> List[Dict]:
        """Get users around a specific user."""
        rank = self.client.zrevrank(self.key, str(user_id))
        if rank is None:
            return []

        start = max(0, rank - context)
        end = rank + context

        results = self.client.zrevrange(
            self.key, start, end, withscores=True
        )
        return [
            {
                "rank": start + i + 1,
                "user_id": int(uid),
                "score": score,
                "is_current_user": int(uid) == user_id
            }
            for i, (uid, score) in enumerate(results)
        ]

    def total_players(self) -> int:
        return self.client.zcard(self.key)


# Usage
lb = Leaderboard(redis_client, "weekly_points")

# Add scores
scores = [(1, 1500), (2, 2300), (3, 1800), (4, 2300), (5, 950), (6, 1200)]
for user_id, score in scores:
    lb.add_score(user_id, score)

# Increment score (user 1 earned 50 more points)
lb.increment_score(1, 50)

print("\n=== Weekly Leaderboard ===")
for entry in lb.get_top(5):
    marker = " 👤" if entry["user_id"] == 1 else ""
    print(f"#{entry['rank']}: User {entry['user_id']} - {entry['score']:.0f} pts{marker}")

print(f"\nUser 1 rank: #{lb.get_rank(1)}")
print(f"User 1 score: {lb.get_score(1)}")
print(f"Total players: {lb.total_players()}")

# Show context around user 3
print("\nAround User 3:")
for entry in lb.get_around_user(3, context=2):
    marker = " ← YOU" if entry["is_current_user"] else ""
    print(f"  #{entry['rank']}: User {entry['user_id']} - {entry['score']:.0f}{marker}")
```

Python

```
# ─────────────────────────────────────────
# 6. DISTRIBUTED LOCK
# ─────────────────────────────────────────
"""
Problem: Multiple servers running same code.
         You want only ONE to execute a critical section.

Example:
  - Sending a daily digest email (don't send twice)
  - Processing a payment (don't charge twice)
  - Running a scheduled job (only one instance)
"""
import uuid

class RedisLock:
    """
    Distributed lock using Redis SET NX EX.
    Prevents multiple servers from running critical code simultaneously.
    """

    def __init__(self, client: redis.Redis, key: str, timeout: int = 30):
        self.client = client
        self.key = f"lock:{key}"
        self.timeout = timeout
        self.lock_id = str(uuid.uuid4())  # unique to this lock instance

    def acquire(self) -> bool:
        """
        Try to acquire the lock.
        NX = only set if not exists
        EX = with expiration (prevents deadlock)
        """
        return bool(
            self.client.set(
                self.key,
                self.lock_id,
                nx=True,        # only set if NOT exists
                ex=self.timeout # expire automatically
            )
        )

    def release(self) -> bool:
        """
        Release the lock.
        Only release if WE own it (compare lock_id).
        Uses Lua script for atomic check-and-delete.
        """
        lua_script = """
        if redis.call("GET", KEYS[1]) == ARGV[1] then
            return redis.call("DEL", KEYS[1])
        else
            return 0
        end
        """
        result = self.client.eval(lua_script, 1, self.key, self.lock_id)
        return bool(result)

    def extend(self, additional_seconds: int) -> bool:
        """Extend the lock TTL if we still own it."""
        lua_script = """
        if redis.call("GET", KEYS[1]) == ARGV[1] then
            return redis.call("EXPIRE", KEYS[1], ARGV[2])
        else
            return 0
        end
        """
        result = self.client.eval(
            lua_script, 1, self.key, self.lock_id, additional_seconds
        )
        return bool(result)

    def __enter__(self):
        acquired = self.acquire()
        if not acquired:
            raise Exception(f"Could not acquire lock: {self.key}")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.release()


# Usage
def send_daily_digest():
    """Should only run once even if called from multiple servers."""
    lock = RedisLock(redis_client, "daily_digest", timeout=300)

    try:
        if not lock.acquire():
            print("Another server is already processing daily digest")
            return

        # Critical section
        print("Processing daily digest...")
        time.sleep(0.1)  # simulate work
        print("Daily digest sent!")

    finally:
        lock.release()

# Context manager version
def process_payment(payment_id: int):
    try:
        with RedisLock(redis_client, f"payment:{payment_id}", timeout=60):
            print(f"Processing payment {payment_id}...")
            # Only one server processes this payment at a time
            time.sleep(0.05)
            print(f"Payment {payment_id} processed!")
    except Exception as e:
        print(f"Payment processing skipped: {e}")

send_daily_digest()
process_payment(12345)


# ─────────────────────────────────────────
# 7. CACHING PATTERNS
# ─────────────────────────────────────────
"""
Cache-aside (Lazy loading):
  Read: check cache → if miss, read DB, write cache
  Write: write DB, invalidate cache
  Best for: read-heavy, data can be stale

Write-through:
  Write: write DB AND cache simultaneously
  Read: always from cache (always fresh)
  Best for: write-heavy with always-fresh requirement

Write-behind (Write-back):
  Write: write to cache, batch-write to DB later
  Risk: data loss if cache fails before DB write
  Best for: very high write throughput

Read-through:
  Cache handles DB interaction transparently
  Best for: dedicated caching layer
"""

class WriteThroughCache:
    """Write-through caching pattern."""

    def __init__(self, cache: CacheService):
        self.cache = cache

    def get_user(self, user_id: int) -> Optional[Dict]:
        """Read from cache (always fresh due to write-through)."""
        return self.cache.get(f"user:{user_id}")

    def save_user(self, user: Dict) -> None:
        """Write to BOTH database and cache simultaneously."""
        user_id = user["id"]

        # 1. Write to database (actual DB call omitted)
        # db.save(user)

        # 2. Write to cache (always fresh)
        self.cache.set(f"user:{user_id}", user, ttl=3600)

    def delete_user(self, user_id: int) -> None:
        """Delete from both database and cache."""
        # db.delete(user_id)
        self.cache.delete(f"user:{user_id}")
```

---

## 3. 🍃 MongoDB — Document Database

### What is MongoDB?

text

```
MongoDB stores data as DOCUMENTS (JSON-like BSON).
No fixed schema — documents in same collection
can have different fields.

Comparison:
┌─────────────────┬──────────────────────────┐
│ PostgreSQL      │ MongoDB                  │
├─────────────────┼──────────────────────────┤
│ Database        │ Database                 │
│ Table           │ Collection               │
│ Row             │ Document                 │
│ Column          │ Field                    │
│ JOIN            │ Embedded documents /     │
│                 │ $lookup aggregation      │
│ Fixed schema    │ Flexible schema          │
│ SQL             │ MQL (MongoDB Query Lang) │
└─────────────────┴──────────────────────────┘

When MongoDB wins:
  ✅ Schema changes frequently (startup, product iteration)
  ✅ Data is naturally document-shaped (nested objects)
  ✅ Horizontal scaling needed (sharding built-in)
  ✅ Different documents need different fields
  ✅ Storing semi-structured data (logs, events, user activity)

When PostgreSQL wins:
  ✅ Complex relationships and JOINs needed
  ✅ ACID transactions critical (financial data)
  ✅ Data is well-structured and stable
  ✅ Reporting and complex queries
```

### Installing MongoDB

Bash

```
# Mac:
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community

# Linux (Ubuntu):
sudo apt update
sudo apt install -y mongodb
sudo systemctl start mongodb
sudo systemctl enable mongodb

# Verify:
mongosh
# MongoDB shell version v7.x.x

# Docker (easiest):
docker run -d -p 27017:27017 --name mongodb \
    -e MONGO_INITDB_ROOT_USERNAME=admin \
    -e MONGO_INITDB_ROOT_PASSWORD=password \
    mongo:latest

# Install Python client:
pip install motor pymongo python-dotenv
# motor = async MongoDB driver
# pymongo = sync MongoDB driver
```

### MongoDB with Python (pymongo)

Python

```
# src/mongodb_client.py
from pymongo import MongoClient, ASCENDING, DESCENDING
from pymongo.errors import DuplicateKeyError, PyMongoError
from bson import ObjectId
from bson.json_util import dumps, loads
from datetime import datetime, timezone
from typing import Optional, List, Dict, Any
import os
from dotenv import load_dotenv
import json

load_dotenv()

MONGODB_URL = os.getenv("MONGODB_URL", "mongodb://localhost:27017")
MONGODB_DB = os.getenv("MONGODB_DB", "myapp")

# ─────────────────────────────────────────
# CONNECTION
# ─────────────────────────────────────────
client = MongoClient(
    MONGODB_URL,
    maxPoolSize=50,           # connection pool size
    serverSelectionTimeoutMS=5000,
    connectTimeoutMS=5000,
)

db = client[MONGODB_DB]

# Test connection
client.admin.command("ping")
print("MongoDB connected!")

# Collections (like tables in SQL)
users_collection = db["users"]
posts_collection = db["posts"]
events_collection = db["events"]
```

### CRUD with MongoDB

Python

```
# ─────────────────────────────────────────
# CREATE INDEXES (do this at startup!)
# ─────────────────────────────────────────
def setup_indexes():
    """Create indexes for performance."""

    # Users collection indexes
    users_collection.create_index("email", unique=True)
    users_collection.create_index("username", unique=True)
    users_collection.create_index([
        ("created_at", DESCENDING)
    ])
    # Compound index
    users_collection.create_index([
        ("role", ASCENDING),
        ("is_active", ASCENDING)
    ])

    # Posts collection indexes
    posts_collection.create_index("slug", unique=True)
    posts_collection.create_index([("author_id", ASCENDING)])
    posts_collection.create_index([
        ("status", ASCENDING),
        ("published_at", DESCENDING)
    ])
    # Text index for search
    posts_collection.create_index([
        ("title", "text"),
        ("content", "text")
    ])

    # TTL index (auto-delete documents after they expire)
    events_collection.create_index(
        "expires_at",
        expireAfterSeconds=0  # delete when expires_at is reached
    )

    print("Indexes created!")

setup_indexes()


# ─────────────────────────────────────────
# INSERT DOCUMENTS
# ─────────────────────────────────────────
def create_user(
    name: str,
    email: str,
    username: str,
    password_hash: str,
    role: str = "user"
) -> Dict:
    """Insert a new user document."""
    user = {
        "name": name,
        "email": email.lower(),
        "username": username.lower(),
        "password_hash": password_hash,
        "role": role,
        "is_active": True,
        "is_verified": False,
        "profile": {
            "bio": None,
            "avatar_url": None,
            "website": None,
            "location": None,
        },
        "stats": {
            "post_count": 0,
            "follower_count": 0,
            "following_count": 0,
        },
        "settings": {
            "email_notifications": True,
            "theme": "light",
        },
        "created_at": datetime.now(timezone.utc),
        "updated_at": datetime.now(timezone.utc),
    }

    try:
        result = users_collection.insert_one(user)
        user["_id"] = result.inserted_id
        return serialize_doc(user)
    except DuplicateKeyError as e:
        if "email" in str(e):
            raise ValueError(f"Email {email} already exists")
        if "username" in str(e):
            raise ValueError(f"Username {username} already taken")
        raise


def serialize_doc(doc: Dict) -> Dict:
    """Convert MongoDB document to JSON-serializable dict."""
    if doc is None:
        return None
    result = {}
    for key, value in doc.items():
        if key == "_id":
            result["id"] = str(value)  # ObjectId → string
        elif isinstance(value, datetime):
            result[key] = value.isoformat()
        elif isinstance(value, ObjectId):
            result[key] = str(value)
        elif isinstance(value, dict):
            result[key] = serialize_doc(value)
        elif isinstance(value, list):
            result[key] = [
                serialize_doc(item) if isinstance(item, dict) else item
                for item in value
            ]
        else:
            result[key] = value
    return result


# Insert multiple documents at once
def bulk_create_users(users_data: List[Dict]) -> int:
    """Insert multiple users at once."""
    now = datetime.now(timezone.utc)
    docs = [
        {**user, "created_at": now, "updated_at": now}
        for user in users_data
    ]
    result = users_collection.insert_many(docs, ordered=False)
    return len(result.inserted_ids)


# ─────────────────────────────────────────
# FIND DOCUMENTS
# ─────────────────────────────────────────
def get_user_by_id(user_id: str) -> Optional[Dict]:
    """Find user by MongoDB ObjectId."""
    try:
        doc = users_collection.find_one({"_id": ObjectId(user_id)})
        return serialize_doc(doc)
    except Exception:
        return None

def get_user_by_email(email: str) -> Optional[Dict]:
    """Find user by email."""
    doc = users_collection.find_one({"email": email.lower()})
    return serialize_doc(doc)

def get_users(
    page: int = 1,
    page_size: int = 10,
    role: Optional[str] = None,
    is_active: Optional[bool] = None,
    search: Optional[str] = None,
    sort_by: str = "created_at",
    sort_dir: int = DESCENDING
) -> Dict:
    """
    Get paginated users with filtering.
    MongoDB's flexible querying in action.
    """
    # Build filter query
    query = {}
    if role:
        query["role"] = role
    if is_active is not None:
        query["is_active"] = is_active
    if search:
        # Case-insensitive regex search
        query["$or"] = [
            {"name": {"$regex": search, "$options": "i"}},
            {"email": {"$regex": search, "$options": "i"}},
            {"username": {"$regex": search, "$options": "i"}},
        ]

    # Count total (before pagination)
    total = users_collection.count_documents(query)

    # Projection: which fields to return
    projection = {
        "password_hash": 0,    # exclude sensitive fields
        "settings": 0,
    }

    # Paginate and sort
    offset = (page - 1) * page_size
    cursor = users_collection.find(
        query,
        projection
    ).sort(
        sort_by, sort_dir
    ).skip(offset).limit(page_size)

    users = [serialize_doc(doc) for doc in cursor]

    return {
        "items": users,
        "total": total,
        "page": page,
        "page_size": page_size,
        "total_pages": (total + page_size - 1) // page_size
    }


# MongoDB Query Operators
def query_examples():
    """Common MongoDB query patterns."""

    # Comparison operators
    users_collection.find({"age": {"$gt": 18}})        # age > 18
    users_collection.find({"age": {"$gte": 18}})       # age >= 18
    users_collection.find({"age": {"$lt": 65}})        # age < 65
    users_collection.find({"age": {"$lte": 65}})       # age <= 65
    users_collection.find({"age": {"$ne": 0}})         # age != 0
    users_collection.find({"role": {"$in": ["admin", "mod"]}})  # IN
    users_collection.find({"role": {"$nin": ["admin"]}})        # NOT IN

    # Logical operators
    users_collection.find({
        "$and": [{"role": "admin"}, {"is_active": True}]
    })
    users_collection.find({
        "$or": [{"role": "admin"}, {"is_verified": True}]
    })
    users_collection.find({
        "role": {"$not": {"$eq": "admin"}}
    })

    # Element operators
    users_collection.find({"phone": {"$exists": True}})   # has phone field
    users_collection.find({"phone": {"$exists": False}})  # doesn't have phone

    # Array operators
    posts_collection.find({"tags": "python"})              # contains python tag
    posts_collection.find({"tags": {"$in": ["python", "fastapi"]}})
    posts_collection.find({"tags": {"$all": ["python", "fastapi"]}}) # has ALL
    posts_collection.find({"tags": {"$size": 3}})          # exactly 3 tags

    # Nested document queries (dot notation)
    users_collection.find({"profile.location": "New York"})
    users_collection.find({"settings.theme": "dark"})
    users_collection.find({"stats.post_count": {"$gt": 5}})

    # Regex
    users_collection.find({
        "email": {"$regex": "@gmail.com$", "$options": "i"}
    })

    # Text search (requires text index)
    posts_collection.find(
        {"$text": {"$search": "python tutorial"}},
        {"score": {"$meta": "textScore"}}  # include relevance score
    ).sort([("score", {"$meta": "textScore"})])


# ─────────────────────────────────────────
# UPDATE DOCUMENTS
# ─────────────────────────────────────────
def update_user(user_id: str, updates: Dict) -> Optional[Dict]:
    """Update user fields."""
    # Don't allow updating sensitive fields
    updates.pop("password_hash", None)
    updates.pop("email", None)  # email update needs special handling

    result = users_collection.find_one_and_update(
        {"_id": ObjectId(user_id)},
        {
            "$set": {**updates, "updated_at": datetime.now(timezone.utc)}
        },
        return_document=True  # return the UPDATED document
    )
    return serialize_doc(result)


def update_nested_field(user_id: str, field_path: str, value: Any) -> bool:
    """Update a nested field using dot notation."""
    result = users_collection.update_one(
        {"_id": ObjectId(user_id)},
        {"$set": {field_path: value, "updated_at": datetime.now(timezone.utc)}}
    )
    return result.modified_count > 0


def increment_stat(user_id: str, stat_name: str, delta: int = 1) -> bool:
    """Atomically increment a numeric field."""
    result = users_collection.update_one(
        {"_id": ObjectId(user_id)},
        {
            "$inc": {f"stats.{stat_name}": delta},
            "$set": {"updated_at": datetime.now(timezone.utc)}
        }
    )
    return result.modified_count > 0


def add_to_array(user_id: str, field: str, value: Any) -> bool:
    """Add item to array field."""
    result = users_collection.update_one(
        {"_id": ObjectId(user_id)},
        {
            "$addToSet": {field: value},  # add if not already present
            "$set": {"updated_at": datetime.now(timezone.utc)}
        }
    )
    return result.modified_count > 0


def remove_from_array(user_id: str, field: str, value: Any) -> bool:
    """Remove item from array field."""
    result = users_collection.update_one(
        {"_id": ObjectId(user_id)},
        {"$pull": {field: value}}
    )
    return result.modified_count > 0


# ─────────────────────────────────────────
# UPDATE OPERATORS REFERENCE
# ─────────────────────────────────────────
"""
$set      → set field value
$unset    → remove field
$inc      → increment numeric field
$mul      → multiply numeric field
$min      → update only if value is lower
$max      → update only if value is higher
$rename   → rename a field
$addToSet → add to array if not present
$push     → push to array (allows duplicates)
$pull     → remove from array by value
$pop      → remove first (-1) or last (1) from array
$pullAll  → remove multiple values from array
"""


# ─────────────────────────────────────────
# DELETE DOCUMENTS
# ─────────────────────────────────────────
def delete_user(user_id: str) -> bool:
    """Hard delete a user."""
    result = users_collection.delete_one({"_id": ObjectId(user_id)})
    return result.deleted_count > 0


def delete_many_users(filter_query: Dict) -> int:
    """Delete multiple users matching a query."""
    result = users_collection.delete_many(filter_query)
    return result.deleted_count


# ─────────────────────────────────────────
# AGGREGATION PIPELINE
# ─────────────────────────────────────────
def get_author_analytics() -> List[Dict]:
    """
    Complex analytics using MongoDB aggregation pipeline.
    Like SQL GROUP BY but much more powerful.

    Pipeline stages:
    $match    → filter (like WHERE)
    $group    → group and aggregate (like GROUP BY)
    $project  → reshape output (like SELECT)
    $sort     → sort results (like ORDER BY)
    $limit    → limit results (like LIMIT)
    $skip     → skip results (like OFFSET)
    $lookup   → join collections (like JOIN)
    $unwind   → flatten arrays
    $addFields → add computed fields
    """
    pipeline = [
        # Stage 1: Only published posts
        {"$match": {"status": "published"}},

        # Stage 2: Group by author
        {"$group": {
            "_id": "$author_id",
            "post_count": {"$sum": 1},
            "total_views": {"$sum": "$view_count"},
            "total_likes": {"$sum": "$like_count"},
            "avg_views": {"$avg": "$view_count"},
            "max_views": {"$max": "$view_count"},
            "tags_used": {"$addToSet": "$tags"},  # collect all tags
            "latest_post": {"$max": "$published_at"},
        }},

        # Stage 3: Filter authors with meaningful data
        {"$match": {"post_count": {"$gte": 1}}},

        # Stage 4: Join with users collection
        {"$lookup": {
            "from": "users",                   # collection to join
            "localField": "_id",               # field from posts group
            "foreignField": "_id",             # field from users
            "as": "author_info"                # output field name
        }},

        # Stage 5: Unwind (flatten) the author_info array
        {"$unwind": "$author_info"},

        # Stage 6: Reshape output
        {"$project": {
            "_id": 0,
            "author_id": "$_id",
            "author_name": "$author_info.name",
            "author_email": "$author_info.email",
            "post_count": 1,
            "total_views": 1,
            "total_likes": 1,
            "avg_views": {"$round": ["$avg_views", 0]},
            "max_views": 1,
            "latest_post": 1,
            "engagement_rate": {
                "$multiply": [
                    {"$divide": [
                        "$total_likes",
                        {"$cond": [
                            {"$eq": ["$total_views", 0]},
                            1,  # avoid division by zero
                            "$total_views"
                        ]}
                    ]},
                    100
                ]
            }
        }},

        # Stage 7: Sort by total views
        {"$sort": {"total_views": -1}},

        # Stage 8: Top 10 authors
        {"$limit": 10}
    ]

    results = list(posts_collection.aggregate(pipeline))
    return [serialize_doc(doc) for doc in results]


# Posts with nested comments example
def create_post_with_comments():
    """
    MongoDB shines with nested documents.
    Store related data TOGETHER in one document.
    No JOINs needed for simple cases.
    """
    post = {
        "title": "Python MongoDB Tutorial",
        "content": "Complete guide to MongoDB with Python...",
        "author_id": ObjectId("..."),  # reference to user
        "tags": ["python", "mongodb", "tutorial"],
        "status": "published",
        "view_count": 0,
        "like_count": 0,

        # Embedded comments (no separate collection needed for small sets)
        "comments": [
            {
                "comment_id": ObjectId(),
                "author_id": ObjectId("..."),
                "content": "Great tutorial!",
                "created_at": datetime.now(timezone.utc),
                "likes": 5,
            }
        ],

        # Embedded metadata
        "seo": {
            "title": "Python MongoDB Tutorial - Complete Guide",
            "description": "Learn MongoDB with Python...",
            "keywords": ["python", "mongodb"],
        },

        "published_at": datetime.now(timezone.utc),
        "created_at": datetime.now(timezone.utc),
        "updated_at": datetime.now(timezone.utc),
    }
    result = posts_collection.insert_one(post)
    return str(result.inserted_id)


# ─────────────────────────────────────────
# TRANSACTIONS (MongoDB 4.0+)
# ─────────────────────────────────────────
def transfer_data_with_transaction():
    """
    MongoDB supports multi-document transactions.
    Requires replica set or sharded cluster.
    """
    with client.start_session() as session:
        with session.start_transaction():
            try:
                # Multiple operations atomically
                users_collection.update_one(
                    {"_id": ObjectId("user1_id")},
                    {"$inc": {"stats.follower_count": -1}},
                    session=session
                )
                users_collection.update_one(
                    {"_id": ObjectId("user2_id")},
                    {"$inc": {"stats.following_count": -1}},
                    session=session
                )
                # If all succeed, transaction commits automatically
            except Exception:
                session.abort_transaction()
                raise
```

### Async MongoDB with Motor

Python

```
# src/async_mongodb.py
"""
Motor = async MongoDB driver.
Use with FastAPI, asyncio-based applications.
"""
import motor.motor_asyncio
import asyncio
from typing import Optional, List, Dict, Any
from bson import ObjectId
from datetime import datetime, timezone
import os
from dotenv import load_dotenv

load_dotenv()

MONGODB_URL = os.getenv("MONGODB_URL", "mongodb://localhost:27017")
MONGODB_DB = os.getenv("MONGODB_DB", "myapp")

# Async client
async_client = motor.motor_asyncio.AsyncIOMotorClient(MONGODB_URL)
async_db = async_client[MONGODB_DB]


class AsyncUserRepository:
    """Async repository for user operations."""

    def __init__(self):
        self.collection = async_db["users"]

    async def create(self, user_data: Dict) -> Dict:
        """Create a new user asynchronously."""
        user_data["created_at"] = datetime.now(timezone.utc)
        user_data["updated_at"] = datetime.now(timezone.utc)
        result = await self.collection.insert_one(user_data)
        created = await self.collection.find_one({"_id": result.inserted_id})
        return self._serialize(created)

    async def get_by_id(self, user_id: str) -> Optional[Dict]:
        """Get user by ID asynchronously."""
        try:
            doc = await self.collection.find_one({"_id": ObjectId(user_id)})
            return self._serialize(doc)
        except Exception:
            return None

    async def get_many(
        self,
        query: Dict = None,
        skip: int = 0,
        limit: int = 10,
        sort_field: str = "created_at",
        sort_dir: int = -1
    ) -> List[Dict]:
        """Get multiple users asynchronously."""
        query = query or {}
        cursor = self.collection.find(query).sort(
            sort_field, sort_dir
        ).skip(skip).limit(limit)

        docs = await cursor.to_list(length=limit)
        return [self._serialize(doc) for doc in docs]

    async def update(self, user_id: str, updates: Dict) -> Optional[Dict]:
        """Update user asynchronously."""
        updates["updated_at"] = datetime.now(timezone.utc)
        result = await self.collection.find_one_and_update(
            {"_id": ObjectId(user_id)},
            {"$set": updates},
            return_document=True
        )
        return self._serialize(result)

    async def delete(self, user_id: str) -> bool:
        """Delete user asynchronously."""
        result = await self.collection.delete_one({"_id": ObjectId(user_id)})
        return result.deleted_count > 0

    async def aggregate(self, pipeline: List[Dict]) -> List[Dict]:
        """Run aggregation pipeline asynchronously."""
        cursor = self.collection.aggregate(pipeline)
        results = await cursor.to_list(length=None)
        return [self._serialize(doc) for doc in results]

    async def count(self, query: Dict = None) -> int:
        """Count documents asynchronously."""
        return await self.collection.count_documents(query or {})

    def _serialize(self, doc: Optional[Dict]) -> Optional[Dict]:
        """Convert MongoDB document to JSON-serializable dict."""
        if doc is None:
            return None
        result = {}
        for key, value in doc.items():
            if key == "_id":
                result["id"] = str(value)
            elif isinstance(value, datetime):
                result[key] = value.isoformat()
            elif isinstance(value, ObjectId):
                result[key] = str(value)
            else:
                result[key] = value
        return result


async def demo_async_mongodb():
    """Demonstrate async MongoDB operations."""
    repo = AsyncUserRepository()

    # Create user
    user = await repo.create({
        "name": "Alice",
        "email": "alice@test.com",
        "role": "admin"
    })
    print(f"Created: {user['name']} (id: {user['id']})")

    # Find user
    found = await repo.get_by_id(user["id"])
    print(f"Found: {found['name']}")

    # Update
    updated = await repo.update(user["id"], {"name": "Alice Updated"})
    print(f"Updated: {updated['name']}")

    # Count
    count = await repo.count()
    print(f"Total users: {count}")

    # Delete
    deleted = await repo.delete(user["id"])
    print(f"Deleted: {deleted}")


# Run async demo
asyncio.run(demo_async_mongodb())
```

---

## 4. 🗄️ Database Tools

### MongoDB Compass (GUI)

Bash

```
# Download from mongodb.com/products/compass
# Connect to: mongodb://localhost:27017
# Features:
#   - Visual document browser
#   - Query builder
#   - Index management
#   - Aggregation pipeline builder
#   - Performance monitoring
```

### Redis CLI & RedisInsight

Bash

```
# Redis CLI (comes with Redis installation)
redis-cli                      # connect
redis-cli -h host -p port -n db  # connect to specific DB

# Useful commands in CLI:
INFO                           # server statistics
INFO keyspace                  # database key counts
CONFIG GET maxmemory           # check memory limit
MONITOR                        # real-time command log (careful in prod!)
SLOWLOG GET 10                 # last 10 slow operations
DEBUG SLEEP 0                  # test connection
FLUSHDB                        # clear current database (dangerous!)
FLUSHALL                       # clear ALL databases (very dangerous!)

# RedisInsight — GUI for Redis
# Download from redis.com/redis-enterprise/redis-insight/
# Connect to: redis://localhost:6379
```

---

## 5. ⚖️ When to Use SQL vs NoSQL

text

```
The honest answer: use them together.

RULE 1: Start with PostgreSQL
  - You know your data needs
  - Need ACID guarantees
  - Relationships matter
  - Strong querying needed

RULE 2: Add Redis for:
  - Caching hot data
  - Session storage
  - Rate limiting
  - Real-time features
  - Simple fast lookups

RULE 3: Add MongoDB when:
  - Schema is genuinely flexible
  - Data is hierarchical/nested
  - You need horizontal scaling
  - Different documents, different fields

RULE 4: Never use NoSQL just because it's "modern"
  - "NoSQL" doesn't mean better
  - Most problems need relational data
  - MongoDB without JOINs = lots of code

REAL PATTERNS:

E-commerce:
  PostgreSQL → products, orders, users, inventory
  Redis      → cart, sessions, product cache, rate limit
  MongoDB    → product reviews (flexible schema), user activity log

Social Media:
  PostgreSQL → user profiles, follows, core data
  Redis      → feed cache, notifications, rate limiting, online status
  MongoDB    → posts (varying fields), comments
  Elasticsearch → search

Real-time Chat:
  PostgreSQL → user accounts, chat rooms
  Redis      → pub/sub for message delivery, online status
  MongoDB    → message history (large volume, flexible)
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                    NOSQL OVERVIEW                              │
│                                                                │
│  REDIS                                                         │
│  ──────                                                        │
│  Type:      Key-Value (in-memory)                             │
│  Speed:     Microseconds                                       │
│  Data:      Strings, Hashes, Lists, Sets, Sorted Sets         │
│  Use:       Cache, Sessions, Rate Limit, Pub/Sub,            │
│             Leaderboard, Distributed Lock, Queues             │
│  TTL:       Built-in key expiration                           │
│  Python:    redis-py (sync), aioredis (async)                │
│                                                                │
│  MONGODB                                                       │
│  ───────                                                       │
│  Type:      Document (BSON/JSON)                              │
│  Speed:     Milliseconds                                       │
│  Schema:    Flexible (no migrations needed)                    │
│  Use:       Content, User Activity, Product Catalog,         │
│             Logs, Events, IoT Data                            │
│  Query:     Rich query language, aggregation pipeline         │
│  Python:    pymongo (sync), motor (async)                    │
│                                                                │
│  DECISION GUIDE:                                               │
│  Need fast lookup + temporary data?    → Redis                │
│  Need flexible schema + nesting?       → MongoDB              │
│  Need ACID + complex queries?          → PostgreSQL           │
│  Best production setup:               → ALL THREE             │
│                                                                │
│  CACHING PATTERNS:                                             │
│  Cache-aside:    Check cache → miss → DB → store             │
│  Write-through:  Write to DB AND cache simultaneously         │
│  Write-behind:   Write to cache → batch write to DB          │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What are the four types of NoSQL databases?
    Give a use case for each.
2.  What does CAP theorem say? Which two does PostgreSQL choose?
3.  What are Redis sorted sets? Give a real use case.
4.  Why is Redis faster than PostgreSQL for session storage?
5.  What is the cache-aside pattern?
6.  How does Redis rate limiting work?
7.  What is a Redis distributed lock and when do you need one?
8.  What is the difference between MongoDB documents and
    PostgreSQL rows?
9.  What is the MongoDB aggregation pipeline?
10. When would you embed documents vs reference them in MongoDB?
11. What is the N+1 problem in MongoDB?
    How do you solve it with $lookup?
12. What is Motor and why use it instead of pymongo?
13. When would you choose MongoDB over PostgreSQL?
14. Name 5 real use cases for Redis in a backend system.
```

---

## 🛠️ Practice Exercises

Python

```
# Exercise 1: Redis Caching Layer
# Build a caching layer for a blog API:
# - Cache individual posts by ID (TTL: 1 hour)
# - Cache list of published posts (TTL: 5 minutes)
# - When a post is updated, invalidate its cache
# - When a new post is published, invalidate list cache
# - Track cache hit/miss ratio
# - Add a method to warm the cache (pre-load top 10 posts)

# Exercise 2: Redis-based Job Queue
# Build a simple job queue using Redis Lists:
# class JobQueue:
#     def enqueue(self, job_type: str, payload: dict) -> str
#     def dequeue(self, timeout: int = 30) -> Optional[dict]
#     def get_queue_length(self) -> int
#     def get_failed_jobs(self) -> List[dict]
#     def retry_failed(self, job_id: str) -> bool
# Support: priority queues (high/normal/low)

# Exercise 3: MongoDB Event Logging
# Build an event logging system:
# class EventLogger:
#     def log(self, user_id, event_type, metadata={}) -> str
#     def get_user_events(user_id, event_type=None, limit=50) -> List
#     def get_events_in_range(start, end, event_type=None) -> List
#     def get_event_stats(start, end) -> dict
#         (count per event_type, unique users, etc.)
# Use TTL index to auto-delete events older than 90 days

# Exercise 4: Combined Redis + MongoDB
# Build a real-time analytics system:
# - Events come in as user actions (MongoDB stores all events)
# - Redis keeps real-time counters (page views, active users)
# - Redis sorted set for trending content (updated in real-time)
# - Every 5 minutes, flush Redis counters to MongoDB
# - API endpoints:
#   GET /analytics/realtime  → from Redis (fast)
#   GET /analytics/history   → from MongoDB (detailed)
#   GET /analytics/trending  → from Redis sorted set

# Exercise 5: Cache Invalidation Strategy
# You have a blog platform with:
#   - Users (profile, settings)
#   - Posts (content, tags)
#   - Comments (nested)
# Design and implement a cache invalidation strategy:
# - What gets cached?
# - What TTL for each?
# - When is each cache invalidated?
# - How do you handle cache stampede
#   (many requests hitting DB when cache expires)?
# Implement with Redis and document your decisions.
```

---

## 🎉 Phase 3 Complete!

text

```
✅ 3.1 — PostgreSQL & SQL
         ACID, normalization, all SQL operations,
         JOINs, CTEs, window functions, indexes,
         transactions, views, full-text search

✅ 3.2 — Python + Database
         psycopg2, SQLAlchemy ORM, models,
         relationships, repository pattern,
         Alembic migrations, loading strategies

✅ 3.3 — NoSQL Databases
         Redis (caching, sessions, rate limiting,
         pub/sub, leaderboards, distributed locks)
         MongoDB (documents, aggregation, Motor async)
         When to use each database type
```

**You now know the complete data layer of backend systems.**v