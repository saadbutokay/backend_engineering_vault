```
Everything you've learned so far:
  Python → the language you write in
  Data Structures → how you organize data in memory
  Databases → how you store data permanently

Now: How does your backend TALK to the outside world?

Answer: HTTP

Every time you:
  - Open Instagram → HTTP request to Instagram's backend
  - Send a message → HTTP request
  - Load a product page → HTTP request
  - Pay for something → HTTP request

HTTP is the protocol that EVERYTHING on the web uses.
Your FastAPI app (coming in 4.2) is an HTTP server.
Understanding HTTP deeply = understanding how backends work.
```

---
## Setup

```bash
cd ~/projects
mkdir http_fundamentals
cd http_fundamentals
python3 -m venv venv
source venv/bin/activate
pip install requests httpx rich
touch http_demo.py
code .
```

---

## 1. 🌐 HTTP Protocol Deep Dive

### What is HTTP?

text

```
HTTP = HyperText Transfer Protocol

A set of rules for how computers communicate over the internet.

The conversation model:
  CLIENT:  "Hey server, give me the list of users"
  SERVER:  "Sure, here they are: [Alice, Bob, Charlie]"

Every HTTP interaction has exactly two parts:
  1. Request  (client → server): "I want something"
  2. Response (server → client): "Here it is / Here's what happened"

HTTP is STATELESS:
  Each request is completely independent.
  Server doesn't remember previous requests.
  If you make 1000 requests, server treats each one fresh.
  (Sessions/cookies/tokens solve this — we'll cover that later)
```

### HTTP Request Anatomy

text

```
A complete HTTP request looks like this:

┌────────────────────────────────────────────────────────────┐
│                     REQUEST LINE                           │
│  POST /api/v1/users HTTP/1.1                               │
│  ↑     ↑             ↑                                     │
│  Method Path         Version                               │
├────────────────────────────────────────────────────────────┤
│                     HEADERS                                │
│  Host: api.example.com                                     │
│  Content-Type: application/json                            │
│  Authorization: Bearer eyJhbGc...                          │
│  Accept: application/json                                  │
│  User-Agent: MyApp/1.0                                     │
│  Content-Length: 52                                        │
├────────────────────────────────────────────────────────────┤
│                     BLANK LINE                             │
│  (separates headers from body)                             │
├────────────────────────────────────────────────────────────┤
│                     BODY (optional)                        │
│  {                                                         │
│    "name": "Alice Smith",                                  │
│    "email": "alice@example.com"                            │
│  }                                                         │
└────────────────────────────────────────────────────────────┘
```

### HTTP Response Anatomy

text

```
A complete HTTP response looks like this:

┌────────────────────────────────────────────────────────────┐
│                     STATUS LINE                            │
│  HTTP/1.1 201 Created                                      │
│  ↑         ↑   ↑                                          │
│  Version   Code Reason phrase                              │
├────────────────────────────────────────────────────────────┤
│                     HEADERS                                │
│  Content-Type: application/json                            │
│  Content-Length: 156                                       │
│  Location: /api/v1/users/42                                │
│  X-Request-ID: abc-123-def                                 │
│  Date: Mon, 15 Jan 2024 14:32:01 GMT                       │
├────────────────────────────────────────────────────────────┤
│                     BLANK LINE                             │
├────────────────────────────────────────────────────────────┤
│                     BODY                                   │
│  {                                                         │
│    "id": 42,                                               │
│    "name": "Alice Smith",                                  │
│    "email": "alice@example.com",                           │
│    "created_at": "2024-01-15T14:32:01Z"                    │
│  }                                                         │
└────────────────────────────────────────────────────────────┘
```

---

## 2. 🔧 HTTP Methods (Verbs)

Python

```
# http_demo.py
"""
HTTP Methods tell the server WHAT you want to do.
Think of them as verbs — actions.

REST convention: method + resource = action

GET    /users         → list all users
GET    /users/42      → get user 42
POST   /users         → create a new user
PUT    /users/42      → replace user 42 completely
PATCH  /users/42      → update user 42 partially
DELETE /users/42      → delete user 42
"""

# ─────────────────────────────────────────
# GET — Retrieve data, no body, safe & idempotent
# ─────────────────────────────────────────
"""
GET requests:
  ✅ Safe: doesn't modify anything
  ✅ Idempotent: same request = same result (call 100 times = same)
  ✅ Cacheable: can be cached by browsers and proxies
  ❌ No request body (data in URL only)
  ❌ Not for sensitive data (URL visible in logs)

Use for: fetching data, searching, filtering
"""

import httpx
import json

# GET a single resource
response = httpx.get("https://jsonplaceholder.typicode.com/users/1")
print(f"GET /users/1")
print(f"  Status: {response.status_code}")
print(f"  Body: {response.json()}")

# GET a collection
response = httpx.get(
    "https://jsonplaceholder.typicode.com/posts",
    params={
        "userId": 1,        # query parameters go in URL
        "_limit": 3,
        "_sort": "id"
    }
)
print(f"\nGET /posts?userId=1&_limit=3")
print(f"  Status: {response.status_code}")
print(f"  Count: {len(response.json())} posts")

# ─────────────────────────────────────────
# POST — Create a new resource
# ─────────────────────────────────────────
"""
POST requests:
  ❌ Not safe: creates something (has side effects)
  ❌ Not idempotent: call 3 times = creates 3 resources
  ✅ Has request body (the new resource data)

Use for: creating new resources, submitting forms,
         triggering actions (login, logout, send email)
"""

response = httpx.post(
    "https://jsonplaceholder.typicode.com/posts",
    json={                      # json= sets Content-Type: application/json
        "title": "My New Post",
        "body": "This is the content",
        "userId": 1
    }
)
print(f"\nPOST /posts")
print(f"  Status: {response.status_code}")   # 201 Created
print(f"  Created: {response.json()}")

# ─────────────────────────────────────────
# PUT — Replace a resource completely
# ─────────────────────────────────────────
"""
PUT requests:
  ❌ Not safe: modifies data
  ✅ Idempotent: calling 10 times = same result as calling once
  ✅ Has request body (the complete new version)

IMPORTANT: PUT replaces the ENTIRE resource.
  If you PUT without 'email', email gets deleted/null.
  
Use for: full updates where you send ALL fields
"""

response = httpx.put(
    "https://jsonplaceholder.typicode.com/posts/1",
    json={
        "id": 1,
        "title": "Updated Title",
        "body": "Updated body content",
        "userId": 1
        # ALL fields required — partial PUT is wrong
    }
)
print(f"\nPUT /posts/1")
print(f"  Status: {response.status_code}")   # 200 OK
print(f"  Updated: {response.json()}")

# ─────────────────────────────────────────
# PATCH — Partially update a resource
# ─────────────────────────────────────────
"""
PATCH requests:
  ❌ Not safe: modifies data
  ✅ Usually idempotent (depends on implementation)
  ✅ Has request body (only the fields you're changing)

IMPORTANT: PATCH updates ONLY the fields you send.
  Other fields stay unchanged.
  
Use for: partial updates (change just the name, not everything)
"""

response = httpx.patch(
    "https://jsonplaceholder.typicode.com/posts/1",
    json={
        "title": "Just the Title Changed"
        # Other fields (body, userId) stay the same!
    }
)
print(f"\nPATCH /posts/1")
print(f"  Status: {response.status_code}")
print(f"  Patched: {response.json()}")

# ─────────────────────────────────────────
# DELETE — Remove a resource
# ─────────────────────────────────────────
"""
DELETE requests:
  ❌ Not safe: removes data
  ✅ Idempotent: deleting already-deleted resource = same result
  Usually no request body
  
Use for: deleting resources
"""

response = httpx.delete(
    "https://jsonplaceholder.typicode.com/posts/1"
)
print(f"\nDELETE /posts/1")
print(f"  Status: {response.status_code}")   # 200 or 204 No Content

# ─────────────────────────────────────────
# HEAD — Same as GET but no body
# ─────────────────────────────────────────
"""
HEAD:
  Same as GET but server only returns headers, no body.
  
Use for:
  - Checking if resource exists (without downloading it)
  - Getting metadata (Content-Length, Last-Modified)
  - Checking if resource has changed (caching)
"""

response = httpx.head("https://jsonplaceholder.typicode.com/posts/1")
print(f"\nHEAD /posts/1")
print(f"  Status: {response.status_code}")
print(f"  Headers: {dict(response.headers)}")
print(f"  Body: '{response.text}'")  # empty!

# ─────────────────────────────────────────
# OPTIONS — What methods does this endpoint support?
# ─────────────────────────────────────────
"""
OPTIONS:
  Server returns what HTTP methods the endpoint allows.
  Used by browsers for CORS preflight requests.
  
Use for:
  - Checking available methods
  - CORS preflight (automatic, browser does this)
"""

response = httpx.options("https://jsonplaceholder.typicode.com/posts")
print(f"\nOPTIONS /posts")
print(f"  Status: {response.status_code}")
print(f"  Allow: {response.headers.get('allow', 'not specified')}")
```

### Method Comparison Table

Python

```
"""
METHOD COMPARISON:

         Safe?  Idempotent?  Has Body?  Use Case
GET      ✅     ✅           ❌        Retrieve data
POST     ❌     ❌           ✅        Create resource
PUT      ❌     ✅           ✅        Replace resource
PATCH    ❌     Usually      ✅        Partial update
DELETE   ❌     ✅           Usually❌  Delete resource
HEAD     ✅     ✅           ❌        Check resource
OPTIONS  ✅     ✅           ❌        Check methods

Safe:        Doesn't change server state
Idempotent:  Multiple identical requests = same result as one

REAL WORLD MAPPING:
  Action                        Method    URL
  ─────────────────────────────────────────────
  List all products             GET       /products
  Get product #5                GET       /products/5
  Create new product            POST      /products
  Replace product #5 entirely   PUT       /products/5
  Update product #5 name only   PATCH     /products/5
  Delete product #5             DELETE    /products/5
  Check if product exists       HEAD      /products/5
  Check product endpoint caps   OPTIONS   /products
  Search products               GET       /products?q=laptop
  Register user                 POST      /auth/register
  Login                         POST      /auth/login
  Logout                        POST      /auth/logout
  Upload file                   POST      /files
  Download file                 GET       /files/123
"""
```

---

## 3. 📊 HTTP Status Codes — Complete Guide

Python

```
# Complete status codes reference
# Know these by heart — you'll use them daily

STATUS_CODES = {
    # ─────────────────────────────────────────
    # 1xx — INFORMATIONAL (rarely used directly)
    # ─────────────────────────────────────────
    100: "Continue — server received request headers, client should proceed",
    101: "Switching Protocols — upgrading to WebSocket",
    102: "Processing — server working, no response yet (WebDAV)",

    # ─────────────────────────────────────────
    # 2xx — SUCCESS
    # ─────────────────────────────────────────
    200: "OK — standard success, with body",
    201: "Created — resource created, include Location header",
    202: "Accepted — request accepted for async processing",
    204: "No Content — success but no body (DELETE, logout)",
    206: "Partial Content — range request (video streaming)",

    # ─────────────────────────────────────────
    # 3xx — REDIRECTS
    # ─────────────────────────────────────────
    301: "Moved Permanently — update your bookmarks forever",
    302: "Found — temporary redirect (keep using original URL)",
    304: "Not Modified — cached version is still valid",
    307: "Temporary Redirect — use same method (unlike 302)",
    308: "Permanent Redirect — use same method (unlike 301)",

    # ─────────────────────────────────────────
    # 4xx — CLIENT ERRORS (you did something wrong)
    # ─────────────────────────────────────────
    400: "Bad Request — malformed syntax, invalid data",
    401: "Unauthorized — not authenticated (login required)",
    403: "Forbidden — authenticated but not authorized",
    404: "Not Found — resource doesn't exist",
    405: "Method Not Allowed — wrong HTTP method for this endpoint",
    406: "Not Acceptable — server can't produce requested format",
    408: "Request Timeout — client too slow",
    409: "Conflict — state conflict (duplicate email, edit conflict)",
    410: "Gone — resource permanently deleted",
    413: "Payload Too Large — request body too big",
    414: "URI Too Long — URL too long",
    415: "Unsupported Media Type — wrong Content-Type",
    422: "Unprocessable Entity — valid syntax but semantic errors",
    429: "Too Many Requests — rate limited",

    # ─────────────────────────────────────────
    # 5xx — SERVER ERRORS (server did something wrong)
    # ─────────────────────────────────────────
    500: "Internal Server Error — generic server crash/bug",
    501: "Not Implemented — method not supported",
    502: "Bad Gateway — upstream server problem",
    503: "Service Unavailable — server down/overloaded",
    504: "Gateway Timeout — upstream server too slow",
    507: "Insufficient Storage — server out of disk space",
}

def explain_status(code: int) -> None:
    print(f"{code}: {STATUS_CODES.get(code, 'Unknown')}")

# The ones you MUST know:
print("=== Must-Know Status Codes ===")
for code in [200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500, 502, 503]:
    explain_status(code)
```

Python

```
# Status code decision guide for API design
def choose_status_code(scenario: str) -> int:
    """Choose the right status code for each scenario."""

    scenarios = {
        # Success cases
        "GET request found data":               200,
        "POST created a new resource":          201,
        "PATCH updated successfully":           200,
        "DELETE succeeded":                     204,  # or 200 with body
        "Long task accepted for processing":    202,

        # Client error cases
        "Missing required field":               400,
        "Invalid data type (string for int)":  400,
        "No auth token provided":               401,
        "Auth token expired":                   401,
        "Valid token but wrong role":           403,
        "User exists but no permission":        403,
        "Resource ID doesn't exist":            404,
        "Wrong HTTP method used":               405,
        "Duplicate email on registration":      409,
        "Business logic validation failed":     422,
        "Request body validation failed":       422,
        "Rate limit exceeded":                  429,

        # Server error cases
        "Unhandled exception in code":          500,
        "Database connection failed":           500,
        "External API is down":                 502,
        "Planned maintenance":                  503,
        "Dependency timeout":                   504,
    }

    code = scenarios.get(scenario, 400)
    print(f"'{scenario}' → {code}")
    return code

# Common confusions
print("\n=== Common 401 vs 403 Confusion ===")
print("401 Unauthorized: 'WHO are you? Show me your ID'")
print("                  Token missing or invalid")
print("403 Forbidden:    'I know WHO you are, but NO'")
print("                  Token valid, but insufficient permissions")

print("\n=== Common 400 vs 422 Confusion ===")
print("400 Bad Request:          'Your request is malformed'")
print("                          Invalid JSON, missing headers")
print("422 Unprocessable Entity: 'I understood it but it's wrong'")
print("                          Valid JSON but email format invalid")
print("                          FastAPI uses 422 for validation errors")

print("\n=== Common 404 vs 410 ===")
print("404 Not Found: Resource might exist, I just don't have it")
print("410 Gone:      Resource USED to exist but was permanently deleted")
```

---

## 4. 📬 HTTP Headers

Python

```
# Headers are metadata about the request or response
# Key: Value format
# Case-insensitive (Content-Type = content-type)

# ─────────────────────────────────────────
# REQUEST HEADERS — sent by client
# ─────────────────────────────────────────

request_headers = {
    # CONTENT
    "Content-Type": "application/json",
    # What format is the body? REQUIRED for POST/PUT/PATCH with body
    # application/json        — JSON data (most common in APIs)
    # multipart/form-data     — file uploads
    # application/x-www-form-urlencoded — HTML form data
    # text/plain              — plain text
    # application/octet-stream — raw binary

    "Content-Length": "256",
    # How many bytes is the body? (often set automatically)

    "Accept": "application/json",
    # What format does client want the RESPONSE in?
    # application/json        — JSON response
    # text/html               — HTML response
    # */*                     — anything (default)

    "Accept-Language": "en-US,en;q=0.9",
    # Preferred response language

    "Accept-Encoding": "gzip, deflate, br",
    # Compression formats the client supports

    # AUTHENTICATION
    "Authorization": "Bearer eyJhbGciOiJIUzI1NiJ9...",
    # Most common format for APIs
    # Bearer <jwt_token>         — JWT authentication (most common)
    # Basic <base64(user:pass)>  — Basic authentication
    # ApiKey <your_api_key>      — API key

    # CACHING
    "Cache-Control": "no-cache",
    # max-age=3600 — cached for 1 hour
    # no-cache     — validate with server before using cache
    # no-store     — don't cache at all

    "If-None-Match": '"abc123"',
    # Send request only if resource has changed
    # (ETag from previous response)

    "If-Modified-Since": "Mon, 15 Jan 2024 00:00:00 GMT",
    # Send only if modified since this date

    # CLIENT INFO
    "User-Agent": "Mozilla/5.0... Chrome/120",
    # Who is making the request? Browser, app, etc.

    "Referer": "https://example.com/page",
    # Which page sent this request? (note: intentional typo in spec)

    "Origin": "https://myapp.com",
    # Origin for CORS requests (browser-set automatically)

    # SERVER ROUTING
    "Host": "api.example.com",
    # Which host to send to (REQUIRED in HTTP/1.1)

    # CUSTOM HEADERS (x- prefix for custom, though deprecated)
    "X-Request-ID": "abc-123-def-456",
    # Unique ID for this request (for tracing/debugging)

    "X-API-Version": "2024-01-15",
    # API version (alternative to URL versioning)

    "X-Forwarded-For": "203.0.113.195",
    # Real client IP when behind proxy/load balancer
}

# ─────────────────────────────────────────
# RESPONSE HEADERS — sent by server
# ─────────────────────────────────────────

response_headers = {
    # CONTENT
    "Content-Type": "application/json; charset=utf-8",
    # What format is the response body?

    "Content-Length": "1024",
    # How many bytes in the body?

    "Content-Encoding": "gzip",
    # Is the body compressed? (gzip, deflate, br)

    # CACHING
    "Cache-Control": "public, max-age=3600",
    # How should response be cached?
    # public        — can be cached anywhere (CDN, browser)
    # private       — only browser cache (user-specific data)
    # no-store      — don't cache (sensitive data)
    # max-age=3600  — cache for 1 hour

    "ETag": '"abc123"',
    # Unique identifier for this version of the resource
    # Client sends back in If-None-Match to check if changed

    "Last-Modified": "Mon, 15 Jan 2024 14:32:01 GMT",
    # When was this resource last changed?

    "Expires": "Tue, 16 Jan 2024 14:32:01 GMT",
    # Exact expiration date (older, prefer Cache-Control)

    # LOCATION
    "Location": "/api/v1/users/42",
    # Where to find the created resource (with 201 Created)
    # Or where to redirect (with 3xx)

    # SECURITY
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
    # HSTS: always use HTTPS for this domain

    "X-Content-Type-Options": "nosniff",
    # Don't guess content type

    "X-Frame-Options": "DENY",
    # Don't embed in iframes (clickjacking protection)

    "Content-Security-Policy": "default-src 'self'",
    # What resources can be loaded

    # CORS (Cross-Origin Resource Sharing)
    "Access-Control-Allow-Origin": "*",
    # Which origins can access this API?
    # * = anyone, or specific: https://myapp.com

    "Access-Control-Allow-Methods": "GET, POST, PUT, PATCH, DELETE",
    "Access-Control-Allow-Headers": "Authorization, Content-Type",
    "Access-Control-Max-Age": "3600",
    # Cache preflight for 1 hour

    # CUSTOM
    "X-Request-ID": "abc-123-def-456",
    # Echo back request ID for correlation

    "X-RateLimit-Limit": "100",
    "X-RateLimit-Remaining": "95",
    "X-RateLimit-Reset": "1705312800",
    # Rate limit information

    "X-Response-Time": "45ms",
    # How long did the server take?

    # DEPRECATION
    "Deprecation": "Sun, 11 Nov 2025 23:59:59 GMT",
    "Sunset": "Sun, 11 Nov 2026 23:59:59 GMT",
    # This API is being deprecated
}

# Demonstration: reading response headers
response = httpx.get("https://httpbin.org/get")
print("=== Response Headers ===")
for key, value in response.headers.items():
    print(f"  {key}: {value}")
```

---

## 5. 🎯 REST API Design Principles

### What is REST?

text

```
REST = Representational State Transfer

Not a protocol (like HTTP), but an ARCHITECTURAL STYLE.
A set of design principles for building web APIs.

An API that follows REST principles is called "RESTful".

The 6 constraints of REST:
  1. Client-Server separation
  2. Stateless
  3. Cacheable
  4. Uniform Interface
  5. Layered System
  6. Code on Demand (optional)

In practice, REST means:
  - Use HTTP methods correctly
  - Use meaningful URLs (resources, not actions)
  - Use appropriate status codes
  - Return proper content types
  - Be stateless (no server-side sessions)
```

### Resource Naming Conventions

Python

```
"""
REST is about RESOURCES, not actions.
A resource is a noun: user, post, comment, product, order.

URL = identifier of a resource
Method = what to do with it

GOOD URL DESIGN:
  /users                    → collection of all users
  /users/42                 → specific user
  /users/42/posts           → all posts by user 42
  /users/42/posts/7         → specific post by user 42

NAMING RULES:

1. Use NOUNS, not verbs
   ❌ /getUser              ✅ /users (GET method does the getting)
   ❌ /createPost           ✅ /posts (POST method does the creating)
   ❌ /deleteUser/42        ✅ /users/42 (DELETE method does the deleting)
   ❌ /updateUserProfile    ✅ /users/42/profile (PATCH)

2. Use PLURAL nouns for collections
   ❌ /user                 ✅ /users
   ❌ /post/5               ✅ /posts/5
   ❌ /product              ✅ /products

3. Use LOWERCASE and HYPHENS (not underscores, not camelCase)
   ❌ /UserProfiles         ✅ /user-profiles
   ❌ /user_profiles        ✅ /user-profiles
   ❌ /userProfiles         ✅ /user-profiles

4. Use HIERARCHY for related resources
   /users/42/posts          → posts by user 42
   /users/42/followers      → followers of user 42
   /posts/7/comments        → comments on post 7
   /posts/7/comments/15     → specific comment on post 7

5. Keep URLs SHORT and INTUITIVE
   ❌ /api/v1/system/users/profile/settings/notifications
   ✅ /users/42/notification-settings

6. Resource IDs in the path
   ✅ /users/42             → user with ID 42
   ✅ /posts/my-first-post  → post with slug "my-first-post"

WHAT ABOUT ACTIONS?
   Some things are actions, not resources.
   
   For these, use a "noun-ish" sub-resource:
   POST /users/42/activate   → activate user (action)
   POST /auth/login          → login (action)
   POST /auth/logout         → logout (action)
   POST /orders/42/cancel    → cancel an order (action)
   POST /emails/send         → send email (action)
   POST /posts/7/publish     → publish a post (action)
"""

# Complete API URL examples:

api_design_examples = {
    # Users
    "GET    /users":                    "List all users (paginated)",
    "GET    /users/42":                 "Get user with ID 42",
    "POST   /users":                    "Create a new user",
    "PUT    /users/42":                 "Replace user 42 entirely",
    "PATCH  /users/42":                 "Update user 42 partially",
    "DELETE /users/42":                 "Delete user 42",
    "GET    /users/42/posts":           "All posts by user 42",
    "POST   /users/42/avatar":          "Upload avatar for user 42",

    # Posts
    "GET    /posts":                    "List all published posts",
    "GET    /posts?tag=python":         "Posts tagged with 'python'",
    "GET    /posts/my-first-post":      "Get post by slug",
    "POST   /posts":                    "Create new post",
    "PATCH  /posts/7":                  "Update post 7",
    "DELETE /posts/7":                  "Delete post 7",
    "POST   /posts/7/publish":          "Publish post 7",
    "GET    /posts/7/comments":         "Comments on post 7",
    "POST   /posts/7/comments":         "Add comment to post 7",
    "POST   /posts/7/likes":            "Like post 7",
    "DELETE /posts/7/likes":            "Unlike post 7",

    # Auth
    "POST   /auth/register":            "Register new user",
    "POST   /auth/login":               "Login, get token",
    "POST   /auth/logout":              "Logout, invalidate token",
    "POST   /auth/refresh":             "Refresh access token",
    "POST   /auth/forgot-password":     "Request password reset",
    "POST   /auth/reset-password":      "Complete password reset",
    "POST   /auth/verify-email":        "Verify email address",

    # Orders (e-commerce)
    "GET    /orders":                   "List my orders",
    "POST   /orders":                   "Place a new order",
    "GET    /orders/123":               "Get order details",
    "POST   /orders/123/cancel":        "Cancel order",
    "GET    /orders/123/items":         "Items in order",

    # Search
    "GET    /search?q=python":          "Search everything",
    "GET    /users/search?q=alice":     "Search users",
}

for endpoint, description in api_design_examples.items():
    print(f"  {endpoint:<40} {description}")
```

### Versioning Strategies

Python

```
"""
APIs change over time. You must version them so
old clients still work when you make breaking changes.

STRATEGY 1: URL Versioning (most common)
  /api/v1/users
  /api/v2/users
  
  Pros: Simple, visible, easy to test in browser
  Cons: URL changes, violates REST (URL = resource, not version)
  Used by: Twitter, Facebook, Stripe

STRATEGY 2: Header Versioning
  GET /users
  Accept-Version: 2.0
  or
  API-Version: 2024-01-15
  
  Pros: URL stays clean
  Cons: Less visible, harder to test in browser
  Used by: GitHub, Azure

STRATEGY 3: Query Parameter Versioning
  GET /users?version=2
  or
  GET /users?api_version=2024-01-15
  
  Pros: Simple, visible
  Cons: Pollutes URL, can break caching
  Used by: Stripe (for date-based versioning)

STRATEGY 4: Content Type Versioning
  Accept: application/vnd.myapi.v2+json
  
  Pros: True REST, clean URLs
  Cons: Complex, hard to use
  Used by: GitHub (older API)

RECOMMENDATION: Use URL versioning for its simplicity
"""

# URL versioning example:
endpoints_v1 = {
    "base": "/api/v1",
    "users": "/api/v1/users",
    "posts": "/api/v1/posts",
}

endpoints_v2 = {
    "base": "/api/v2",
    "users": "/api/v2/users",      # same resource, new version
    "posts": "/api/v2/posts",
    "articles": "/api/v2/articles", # new resource in v2
}

# Version by date (Stripe's approach — very good)
endpoints_date_versioned = {
    "header": "API-Version: 2024-01-15",
    "users": "/api/users",  # same URL, version in header
}
```

### Pagination Strategies

Python

```
"""
Never return ALL records at once.
A table with 10 million rows would crash your server.
Always paginate.

TWO main strategies:

1. OFFSET-BASED PAGINATION (simple)
   GET /posts?page=2&page_size=10
   GET /posts?offset=20&limit=10

   SQL: SELECT * FROM posts LIMIT 10 OFFSET 20

   Pros:
     ✅ Simple to implement
     ✅ Can jump to any page
     ✅ Easy for users to understand
   
   Cons:
     ❌ Slow for large offsets (DB scans all skipped rows)
     ❌ Inconsistent if data changes between requests
        (new item inserted → page 2 shows item from page 1 again)
   
   Use for: small datasets, when page jumping needed

2. CURSOR-BASED PAGINATION (better for large data)
   GET /posts?cursor=abc123&limit=10
   
   Response includes:
   {
     "items": [...],
     "next_cursor": "def456",
     "has_more": true
   }
   
   SQL: SELECT * FROM posts WHERE id > 20 LIMIT 10

   Pros:
     ✅ Consistent (no items missed when data changes)
     ✅ Fast for any offset (uses index)
     ✅ Works well for real-time feeds
   
   Cons:
     ❌ Can't jump to arbitrary page
     ❌ More complex to implement
   
   Use for: large datasets, real-time feeds, social media

STANDARD RESPONSE FORMAT:
"""

# Offset-based pagination response
offset_pagination_response = {
    "data": [
        {"id": 11, "title": "Post 11"},
        {"id": 12, "title": "Post 12"},
        # ... 10 items
    ],
    "pagination": {
        "total": 1000,          # total records
        "page": 2,              # current page
        "page_size": 10,        # items per page
        "total_pages": 100,     # total pages
        "has_next": True,       # is there a next page?
        "has_prev": True,       # is there a previous page?
    }
}

# Cursor-based pagination response
cursor_pagination_response = {
    "data": [
        {"id": 50, "title": "Post 50"},
        # ... items
    ],
    "pagination": {
        "cursor": "eyJpZCI6IDUwfQ==",  # base64 encoded cursor
        "next_cursor": "eyJpZCI6IDQwfQ==",
        "has_more": True,
        "limit": 10,
    }
}

# Implementation example
def offset_paginate(page: int, page_size: int = 20):
    """Standard offset pagination calculation."""
    if page < 1:
        page = 1
    if page_size < 1:
        page_size = 20
    if page_size > 100:
        page_size = 100  # cap max page size

    offset = (page - 1) * page_size
    return offset, page_size

def cursor_paginate(cursor: str = None, limit: int = 20):
    """Decode cursor to get the last ID."""
    import base64
    import json

    if cursor:
        try:
            decoded = base64.b64decode(cursor).decode()
            data = json.loads(decoded)
            return data.get("id"), limit
        except Exception:
            return None, limit
    return None, limit

def encode_cursor(item_id: int) -> str:
    """Encode item ID as a cursor."""
    import base64
    import json
    data = json.dumps({"id": item_id})
    return base64.b64encode(data.encode()).decode()
```

### Filtering, Sorting, Searching

Python

```
"""
Query parameters for data manipulation.

FILTERING:
  GET /products?category=electronics
  GET /products?min_price=100&max_price=500
  GET /products?is_active=true
  GET /products?status=published&author_id=42

SORTING:
  GET /products?sort=price              (ascending)
  GET /products?sort=-price             (descending, - prefix)
  GET /products?sort=price,-created_at  (multi-sort)
  GET /products?order_by=price&order=asc

SEARCHING:
  GET /products?q=laptop
  GET /products?search=python tutorial
  GET /users?email=alice@test.com

FIELD SELECTION (sparse fieldsets):
  GET /users?fields=id,name,email     (only return these fields)
  GET /posts?exclude=content          (return all except content)

COMBINING:
  GET /posts?
      status=published
      &tag=python
      &author_id=42
      &sort=-published_at
      &page=2
      &page_size=10
      &fields=id,title,author,published_at
"""

# Building filter parameters
from urllib.parse import urlencode

def build_query_params(**kwargs) -> str:
    """Build query string from parameters, removing None values."""
    clean = {k: v for k, v in kwargs.items() if v is not None}
    return urlencode(clean)

params = build_query_params(
    status="published",
    tag="python",
    sort="-created_at",
    page=1,
    page_size=20
)
print(f"Query string: ?{params}")
# ?status=published&tag=python&sort=-created_at&page=1&page_size=20

# Standardized filter implementation
class QueryFilter:
    """Parse and validate query parameters for filtering."""

    def __init__(self, params: dict):
        self.params = params

    def get_page(self, default: int = 1) -> int:
        try:
            return max(1, int(self.params.get("page", default)))
        except (ValueError, TypeError):
            return default

    def get_page_size(
        self,
        default: int = 20,
        max_size: int = 100
    ) -> int:
        try:
            size = int(self.params.get("page_size", default))
            return min(max_size, max(1, size))
        except (ValueError, TypeError):
            return default

    def get_sort(
        self,
        allowed: list,
        default: str = "created_at"
    ) -> tuple:
        """Return (field, direction) tuple."""
        sort = self.params.get("sort", default)
        if sort.startswith("-"):
            field = sort[1:]
            direction = "desc"
        else:
            field = sort
            direction = "asc"

        if field not in allowed:
            field = default
            direction = "desc"

        return field, direction

    def get_search(self) -> str | None:
        q = self.params.get("q") or self.params.get("search")
        if q:
            return q.strip()[:200]  # limit search term length
        return None
```

### Idempotency

Python

```
"""
IDEMPOTENCY: calling an operation multiple times
             has the same effect as calling it once.

Why it matters:
  Network failures happen.
  Client sends request → network fails → client retries.
  Was the first request processed?
  If not idempotent → you process it TWICE.

Examples:
  POST /orders              ← NOT idempotent by default
    - Retry → two orders!

  PUT /orders/42            ← Idempotent
    - Retry → same order, same state
    
  DELETE /orders/42         ← Idempotent  
    - Retry → 404 is fine (or 204 if already deleted)

SOLUTION for POST: Idempotency Keys

Client generates a unique key for the request.
If server has seen this key → return same result.
If not seen → process and store result.

Pattern used by: Stripe, PayPal, AWS

Request:
  POST /payments
  Idempotency-Key: a8098c1a-f86e-11da-bd1a-00112444be1e
  
  {"amount": 100, "card": "..."}

Server:
  1. Check if Idempotency-Key seen before
  2. Yes: return stored response (don't process again!)
  3. No: process, store with key, return response
"""

import redis
import json
import hashlib
from datetime import datetime, timezone

redis_client = redis.from_url("redis://localhost:6379", decode_responses=True)

def idempotent_handler(
    idempotency_key: str,
    handler_func,
    ttl: int = 86400  # store results for 24 hours
):
    """
    Wrap any handler to make it idempotent.
    First call: execute and store result.
    Subsequent calls with same key: return stored result.
    """
    cache_key = f"idempotency:{idempotency_key}"

    # Check if we've seen this key
    cached = redis_client.get(cache_key)
    if cached:
        print(f"Idempotency hit! Returning cached result.")
        return json.loads(cached)

    # First time: execute the handler
    result = handler_func()

    # Store result for future duplicate requests
    redis_client.setex(cache_key, ttl, json.dumps(result, default=str))

    return result


# Usage example
def create_payment(amount: float, user_id: int) -> dict:
    """The actual payment processing logic."""
    print(f"Processing payment of ${amount} for user {user_id}")
    # In real code: charge credit card, update database, etc.
    return {
        "payment_id": "pay_abc123",
        "amount": amount,
        "status": "completed",
        "created_at": datetime.now(timezone.utc).isoformat()
    }

idempotency_key = "client-generated-unique-key-12345"

# First request
print("=== First Request ===")
result = idempotent_handler(
    idempotency_key,
    lambda: create_payment(99.99, 42)
)
print(f"Result: {result}\n")

# Duplicate request (retry)
print("=== Retry Request (should not charge again) ===")
result = idempotent_handler(
    idempotency_key,
    lambda: create_payment(99.99, 42)
)
print(f"Result: {result}")
```

---

## 6. 📖 OpenAPI / Swagger Documentation

Python

```
"""
OpenAPI (formerly Swagger) = standard for documenting APIs.

Your API documentation in machine-readable JSON/YAML format.

Benefits:
  ✅ Auto-generates interactive documentation
  ✅ Clients can auto-generate SDK code from it
  ✅ Teams agree on API contract before coding
  ✅ Testing tools can use it
  ✅ FastAPI generates this automatically!

Structure:
  openapi: "3.0.0"
  info:
    title: My API
    version: 1.0.0
  paths:
    /users:
      get:
        summary: List users
        parameters: [...]
        responses:
          200:
            description: Success
            content:
              application/json:
                schema:
                  type: array
                  items:
                    $ref: '#/components/schemas/User'
  components:
    schemas:
      User:
        type: object
        properties:
          id: {type: integer}
          name: {type: string}
          email: {type: string, format: email}
"""

# FastAPI auto-generates OpenAPI!
# When we build FastAPI in 4.2, visit:
#   http://localhost:8000/docs    ← Swagger UI
#   http://localhost:8000/redoc  ← ReDoc UI
#   http://localhost:8000/openapi.json ← raw JSON

# For now, here's what a manually written OpenAPI spec looks like:
openapi_spec = {
    "openapi": "3.0.0",
    "info": {
        "title": "Blog API",
        "description": "A simple blog API",
        "version": "1.0.0",
        "contact": {
            "name": "API Support",
            "email": "api@example.com"
        }
    },
    "servers": [
        {"url": "https://api.example.com/v1", "description": "Production"},
        {"url": "http://localhost:8000/api/v1", "description": "Development"}
    ],
    "paths": {
        "/users": {
            "get": {
                "summary": "List all users",
                "description": "Returns paginated list of users",
                "tags": ["Users"],
                "parameters": [
                    {
                        "name": "page",
                        "in": "query",
                        "required": False,
                        "schema": {"type": "integer", "default": 1}
                    },
                    {
                        "name": "page_size",
                        "in": "query",
                        "required": False,
                        "schema": {"type": "integer", "default": 20, "maximum": 100}
                    }
                ],
                "responses": {
                    "200": {
                        "description": "Success",
                        "content": {
                            "application/json": {
                                "schema": {
                                    "$ref": "#/components/schemas/UserList"
                                }
                            }
                        }
                    }
                },
                "security": [{"bearerAuth": []}]
            },
            "post": {
                "summary": "Create a new user",
                "tags": ["Users"],
                "requestBody": {
                    "required": True,
                    "content": {
                        "application/json": {
                            "schema": {"$ref": "#/components/schemas/CreateUser"}
                        }
                    }
                },
                "responses": {
                    "201": {"description": "User created"},
                    "422": {"description": "Validation error"},
                    "409": {"description": "Email already exists"}
                }
            }
        },
        "/users/{user_id}": {
            "get": {
                "summary": "Get a specific user",
                "tags": ["Users"],
                "parameters": [
                    {
                        "name": "user_id",
                        "in": "path",
                        "required": True,
                        "schema": {"type": "integer"}
                    }
                ],
                "responses": {
                    "200": {"description": "Success"},
                    "404": {"description": "User not found"}
                }
            }
        }
    },
    "components": {
        "schemas": {
            "User": {
                "type": "object",
                "properties": {
                    "id": {"type": "integer"},
                    "name": {"type": "string", "example": "Alice Smith"},
                    "email": {"type": "string", "format": "email"},
                    "role": {
                        "type": "string",
                        "enum": ["user", "admin", "moderator"]
                    },
                    "is_active": {"type": "boolean"},
                    "created_at": {"type": "string", "format": "date-time"}
                }
            },
            "CreateUser": {
                "type": "object",
                "required": ["name", "email", "password"],
                "properties": {
                    "name": {"type": "string", "minLength": 2, "maxLength": 100},
                    "email": {"type": "string", "format": "email"},
                    "password": {"type": "string", "minLength": 8}
                }
            },
            "UserList": {
                "type": "object",
                "properties": {
                    "data": {
                        "type": "array",
                        "items": {"$ref": "#/components/schemas/User"}
                    },
                    "pagination": {
                        "type": "object",
                        "properties": {
                            "total": {"type": "integer"},
                            "page": {"type": "integer"},
                            "page_size": {"type": "integer"},
                            "total_pages": {"type": "integer"}
                        }
                    }
                }
            }
        },
        "securitySchemes": {
            "bearerAuth": {
                "type": "http",
                "scheme": "bearer",
                "bearerFormat": "JWT"
            }
        }
    }
}

import json
print("=== OpenAPI Spec Preview ===")
print(json.dumps(openapi_spec, indent=2)[:500] + "...")
```

---

## 7. 🌐 GraphQL Basics

Python

```
"""
GraphQL = Alternative to REST for API design.
Created by Facebook, now widely used.

KEY DIFFERENCE from REST:

REST:
  Multiple endpoints, each returns fixed data:
    GET /users/42          → ALL user data
    GET /users/42/posts    → ALL post data
    GET /users/42/followers→ ALL follower data
  
  Problem: Over-fetching (get more than needed)
           Under-fetching (need multiple requests)

GraphQL:
  ONE endpoint: POST /graphql
  Client SPECIFIES what data it wants.

  Query:
    query {
      user(id: 42) {
        name
        email
        posts(limit: 5) {
          title
          publishedAt
        }
      }
    }

  Response:
    {
      "data": {
        "user": {
          "name": "Alice",
          "email": "alice@test.com",
          "posts": [
            {"title": "Post 1", "publishedAt": "2024-01-15"}
          ]
        }
      }
    }

  Benefits:
    ✅ Exact data fetching (no over/under-fetching)
    ✅ Single request for complex data
    ✅ Strongly typed schema
    ✅ Self-documenting
    ✅ Great for mobile (limited bandwidth)

  Drawbacks:
    ❌ More complex to implement
    ❌ Harder to cache (all POST requests)
    ❌ N+1 problem needs explicit handling
    ❌ Learning curve

WHEN TO USE:
  GraphQL:
    - Mobile apps (need exact data)
    - Complex, nested data
    - Many different clients (web, mobile, desktop)
    - Public API where clients have varying needs
  
  REST:
    - Simple CRUD operations
    - File uploads
    - When caching is critical
    - Smaller teams, simpler APIs
    - Most backend projects (REST is still dominant)

We cover GraphQL with Strawberry in Phase 9.4.
For now, understand the concept.
"""

# Simple GraphQL query example (conceptual)
graphql_examples = {
    "query": """
        # Get user with specific fields + related data
        query GetUser {
          user(id: 42) {
            id
            name
            email
            posts(
              filter: {status: PUBLISHED}
              limit: 5
              orderBy: {field: CREATED_AT, direction: DESC}
            ) {
              id
              title
              publishedAt
              commentCount
            }
            followers {
              count
            }
          }
        }
    """,

    "mutation": """
        # Create/update/delete operations
        mutation CreatePost {
          createPost(input: {
            title: "My New Post"
            content: "Post content here"
            tags: ["python", "fastapi"]
          }) {
            id
            title
            slug
            createdAt
          }
        }
    """,

    "subscription": """
        # Real-time updates
        subscription OnNewMessage {
          newMessage(chatRoomId: "room-123") {
            id
            content
            sender {
              name
              avatar
            }
            createdAt
          }
        }
    """
}

for operation_type, query in graphql_examples.items():
    print(f"\n=== GraphQL {operation_type.title()} ===")
    print(query)
```

---

## 8. ⚡ gRPC Basics

Python

```
"""
gRPC = Google Remote Procedure Call

Another alternative to REST, uses Protocol Buffers.

WHEN gRPC beats REST:
  - Internal service-to-service communication
  - High performance (10x faster than REST typically)
  - Strongly typed contracts (.proto files)
  - Streaming support built-in
  - Multiple languages (Python, Go, Java, etc.)

HOW IT WORKS:
  1. Define service in .proto file (contract)
  2. Generate code for client and server
  3. Call remote functions like local functions

Example .proto file:
  syntax = "proto3";
  
  service UserService {
    rpc GetUser (GetUserRequest) returns (UserResponse);
    rpc ListUsers (ListUsersRequest) returns (stream UserResponse);
    rpc CreateUser (CreateUserRequest) returns (UserResponse);
  }
  
  message GetUserRequest {
    int32 user_id = 1;
  }
  
  message UserResponse {
    int32 id = 1;
    string name = 2;
    string email = 3;
  }

COMPARISON:
  Protocol:   REST        gRPC
  Format:     JSON        Protocol Buffers (binary)
  Transport:  HTTP/1.1    HTTP/2
  Speed:      Baseline    ~10x faster
  Streaming:  Limited     Full (4 types)
  Browser:    Native      Needs proxy
  Debugging:  Easy        Harder (binary)
  Use case:   Public API  Internal services

We cover gRPC with Python in Phase 9.5.
"""

# gRPC service types (conceptual):
grpc_service_types = {
    "Unary RPC": "One request → One response (like REST)",
    "Server streaming": "One request → Stream of responses",
    "Client streaming": "Stream of requests → One response",
    "Bidirectional": "Stream of requests ↔ Stream of responses",
}

for service_type, description in grpc_service_types.items():
    print(f"  {service_type}: {description}")
```

---

## 9. 🔌 WebSockets Basics

Python

```
"""
HTTP is request-response.
Client asks → Server answers → Connection done.

Problem: What if SERVER needs to PUSH data to client?
  - Live chat messages
  - Real-time notifications
  - Live stock prices
  - Collaborative editing
  - Game state updates

HTTP solution (bad):
  Client polls every second: "Any new messages?"
  Server: "No... no... no... yes! Here they are."
  Waste of resources, high latency.

WebSocket solution:
  Client: "Let's upgrade to WebSocket connection"
  Server: "Agreed"
  [Connection stays open permanently]
  Server can now PUSH data anytime → instant delivery
  Client can also send data anytime

WebSocket connection:
  1. HTTP handshake (upgrade request)
  2. Both agree to upgrade
  3. TCP connection stays open
  4. Both sides can send messages freely
  5. Either side closes when done

Use WebSockets for:
  ✅ Chat applications
  ✅ Live notifications
  ✅ Real-time dashboards
  ✅ Collaborative tools
  ✅ Gaming
  ✅ Live data feeds

Use HTTP for everything else (most things).

We implement WebSockets in FastAPI in Phase 4.2.
"""

# WebSocket message types (conceptual)
websocket_examples = {
    "connect": {
        "type": "connect",
        "user_id": 42,
        "room_id": "general"
    },
    "message": {
        "type": "message",
        "content": "Hello everyone!",
        "timestamp": "2024-01-15T14:32:01Z"
    },
    "notification": {
        "type": "notification",
        "event": "new_follower",
        "data": {"follower_id": 7, "follower_name": "Bob"}
    },
    "typing": {
        "type": "typing",
        "user_id": 42,
        "is_typing": True
    },
    "disconnect": {
        "type": "disconnect",
        "reason": "user_closed"
    }
}

for message_type, payload in websocket_examples.items():
    print(f"\nWebSocket '{message_type}' message:")
    print(f"  {json.dumps(payload, indent=4)}")
```

---

## 10. 🔗 CORS — Cross-Origin Resource Sharing

Python

```
"""
CORS = Browser security feature.

Browser security rule:
  JavaScript on website-A cannot make requests to website-B.
  Called the "Same-Origin Policy".

This is GOOD for security (prevents malicious sites from
using your credentials on other sites).

But it's also a PROBLEM for legitimate APIs:
  Your frontend: https://myapp.com
  Your API:      https://api.myapp.com
  
  Browser: "These are different origins! BLOCKED!"

SOLUTION: CORS headers
  API server tells browser: "It's OK, allow requests from myapp.com"

HOW CORS WORKS:

1. Simple Requests (GET, POST with basic headers):
   Browser sends request with Origin header.
   Server responds with Access-Control-Allow-Origin header.
   If browser is satisfied → request allowed.

2. Preflight Requests (PUT, DELETE, custom headers, etc.):
   Browser first sends OPTIONS request:
     OPTIONS /api/users
     Origin: https://myapp.com
     Access-Control-Request-Method: PUT
     Access-Control-Request-Headers: Authorization
   
   Server responds:
     Access-Control-Allow-Origin: https://myapp.com
     Access-Control-Allow-Methods: GET, POST, PUT, DELETE
     Access-Control-Allow-Headers: Authorization, Content-Type
     Access-Control-Max-Age: 3600
   
   If OK → browser sends actual PUT request.
"""

# CORS configuration examples
cors_configs = {
    "development": {
        "allow_origins": ["*"],  # allow all (only for dev!)
        "allow_credentials": False,
        "allow_methods": ["*"],
        "allow_headers": ["*"],
    },

    "production_specific": {
        "allow_origins": [
            "https://myapp.com",
            "https://admin.myapp.com",
        ],
        "allow_credentials": True,      # allow cookies/auth headers
        "allow_methods": ["GET", "POST", "PUT", "PATCH", "DELETE"],
        "allow_headers": [
            "Authorization",
            "Content-Type",
            "X-Request-ID",
        ],
        "max_age": 3600,  # cache preflight for 1 hour
    },

    "public_api": {
        "allow_origins": ["*"],  # public API, any origin OK
        "allow_credentials": False,  # no cookies with *
        "allow_methods": ["GET", "POST"],
        "allow_headers": ["Authorization", "Content-Type"],
    }
}

# In FastAPI (preview — we'll implement this in 4.2):
fastapi_cors_code = """
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://myapp.com",
        "http://localhost:3000",  # local frontend dev
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
"""
print("FastAPI CORS setup (preview):")
print(fastapi_cors_code)
```

---

## 11. 🔄 Request & Response Patterns

Python

```
"""
STANDARD API RESPONSE FORMAT

No official standard, but these patterns are widely adopted.

Pattern 1: Data envelope
{
    "data": {...},         ← actual payload
    "meta": {...},         ← metadata (pagination, etc.)
    "errors": null         ← errors if any
}

Pattern 2: Success/error split
Success:
{
    "success": true,
    "data": {...}
}

Error:
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Email is invalid",
        "details": [...]
    }
}

Pattern 3: JSON:API spec (formal standard)
{
    "data": {
        "type": "users",
        "id": "42",
        "attributes": {...},
        "relationships": {...}
    }
}

RECOMMENDATION: Use Pattern 2 (simple, clear).
FastAPI can enforce this with response models.
"""

# Standard response builders
from datetime import datetime, timezone
from typing import Any, Optional

def success_response(
    data: Any,
    message: str = "Success",
    status_code: int = 200,
    meta: Optional[dict] = None
) -> dict:
    """Build a standardized success response."""
    response = {
        "success": True,
        "data": data,
        "message": message,
    }
    if meta:
        response["meta"] = meta
    return response


def error_response(
    message: str,
    code: str = "ERROR",
    status_code: int = 400,
    details: Optional[list] = None,
    field: Optional[str] = None
) -> dict:
    """Build a standardized error response."""
    error = {
        "code": code,
        "message": message,
    }
    if details:
        error["details"] = details
    if field:
        error["field"] = field

    return {
        "success": False,
        "error": error
    }


def paginated_response(
    items: list,
    total: int,
    page: int,
    page_size: int
) -> dict:
    """Build a standardized paginated response."""
    return {
        "success": True,
        "data": items,
        "pagination": {
            "total": total,
            "page": page,
            "page_size": page_size,
            "total_pages": (total + page_size - 1) // page_size,
            "has_next": page * page_size < total,
            "has_prev": page > 1
        }
    }


# Examples
user = {"id": 42, "name": "Alice", "email": "alice@test.com"}
print(json.dumps(success_response(user, "User retrieved"), indent=2))

print(json.dumps(error_response(
    "Email is already registered",
    code="DUPLICATE_EMAIL",
    status_code=409,
    field="email"
), indent=2))

users = [{"id": i, "name": f"User {i}"} for i in range(1, 21)]
print(json.dumps(paginated_response(
    items=users[:10],
    total=100,
    page=1,
    page_size=10
), indent=2))
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                HTTP & API FUNDAMENTALS                         │
│                                                                │
│  HTTP REQUEST:                                                 │
│  METHOD  /path?query  HTTP/1.1                                 │
│  Headers: key: value                                           │
│  [blank line]                                                  │
│  Body (optional)                                               │
│                                                                │
│  METHODS:                                                      │
│  GET    → read (safe, idempotent, no body)                    │
│  POST   → create (not safe, not idempotent, has body)         │
│  PUT    → replace all (idempotent, has body)                  │
│  PATCH  → update part (usually idempotent, has body)          │
│  DELETE → remove (idempotent, usually no body)                │
│                                                                │
│  STATUS CODES:                                                 │
│  2xx → success    3xx → redirect                              │
│  4xx → your fault 5xx → server fault                          │
│  200 OK  201 Created  204 No Content                          │
│  400 Bad Request  401 Unauthorized  403 Forbidden             │
│  404 Not Found  409 Conflict  422 Validation  429 Rate Limit  │
│  500 Server Error  502 Bad Gateway  503 Unavailable           │
│                                                                │
│  REST DESIGN:                                                  │
│  Nouns not verbs  Plural  Lowercase-hyphens                   │
│  /users /posts /orders /auth/login                            │
│  Hierarchical: /users/42/posts                                │
│                                                                │
│  ALTERNATIVES:                                                 │
│  GraphQL  → flexible queries, one endpoint                    │
│  gRPC     → fast, binary, streaming, internal services        │
│  WebSocket→ real-time bidirectional communication             │
│                                                                │
│  PAGINATION:                                                   │
│  Offset: ?page=2&page_size=10 (simple, inconsistent)         │
│  Cursor: ?cursor=abc&limit=10 (complex, consistent)           │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What are the two parts of every HTTP interaction?
2.  What makes HTTP stateless? Is this good or bad?
3.  What is the difference between PUT and PATCH?
4.  What does "idempotent" mean? Which methods are idempotent?
5.  What is the difference between 401 and 403?
6.  What is the difference between 400 and 422?
7.  When would you use 204 vs 200?
8.  What does the Content-Type header do?
9.  What is the Authorization header used for?
10. What does REST stand for and what are its key principles?
11. Why use plural nouns in REST URLs?
12. What is the difference between offset and cursor pagination?
13. What is idempotency key and when do you need it?
14. What is CORS and why does it exist?
15. What is OpenAPI/Swagger?
16. When would you choose GraphQL over REST?
17. When would you choose gRPC over REST?
18. What are WebSockets used for?
```

---

## 🛠️ Practice Exercises

Python

```
# Exercise 1: HTTP Methods
# For a "Library Management System", design REST endpoints for:
# - Books (list, get, create, update, delete)
# - Authors (list, get)
# - Borrowing (checkout a book, return a book, history)
# - Members (register, profile, search)
# Write out: METHOD  /path  → description
# for at least 15 endpoints

# Exercise 2: Status Codes
# For each scenario, choose the right status code and explain:
# a) User tries to register with an email that exists
# b) Client sends request without Authorization header
# c) Admin successfully deletes a user account
# d) Client sends {"age": "twenty"} (wrong type)
# e) User tries to delete another user's post
# f) Database connection fails during request
# g) Product successfully added to shopping cart
# h) Search returns zero results
# i) File upload exceeds 10MB limit
# j) Same POST request sent twice (idempotent key matches)

# Exercise 3: API Design Review
# Review this API design and list everything wrong with it:
# POST   /getUser?id=42
# GET    /CreateNewBlogPost
# DELETE /user_delete/42
# PUT    /api/v1/users/42/updateEmail
# GET    /api/get_all_users_list
# POST   /api/v1/users/42/profile/settings/notifications/update

# Exercise 4: Request/Response Headers
# For a POST /users request to create a user:
# a) List all headers the CLIENT should send
# b) List all headers the SERVER should respond with
# c) What status code and body for successful creation?
# d) What status code and body if email already exists?
# e) What status code and body if JWT token is expired?

# Exercise 5: Pagination
# Implement both offset and cursor pagination:
# def offset_paginate(data: list, page: int, size: int) -> dict
# def cursor_paginate(data: list, cursor: str, limit: int) -> dict
# Each should return proper pagination metadata
# Test with a list of 100 fake users
```

---

## ✅ Phase 4.1 Complete!

**You now know:**

text

```
✅ HTTP protocol — request/response structure
✅ All HTTP methods (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS)
✅ HTTP status codes — every important one
✅ HTTP headers — request and response
✅ REST design principles
✅ Resource naming conventions
✅ API versioning strategies
✅ Pagination (offset and cursor)
✅ Filtering, sorting, searching patterns
✅ Idempotency and idempotency keys
✅ OpenAPI/Swagger documentation
✅ GraphQL concepts (when to use)
✅ gRPC concepts (when to use)
✅ WebSocket concepts (when to use)
✅ CORS — what it is and how to configure
✅ Standard API response formats
```