```
Every backend application needs to store data permanently.

Without a database:
  User registers → data lives in memory
  Server restarts → ALL DATA GONE
  10,000 users register → RAM runs out

With a database:
  User registers → data written to disk
  Server restarts → data still there
  10,000 users → database handles it efficiently
  1,000,000 users → database still handles it

A database is a program that:
  ✅ Stores data permanently (survives restarts)
  ✅ Handles concurrent access (1000 users at once)
  ✅ Guarantees data integrity (no corruption)
  ✅ Provides fast querying (find anything quickly)
  ✅ Supports transactions (all-or-nothing operations)
```

---

## Why PostgreSQL?

```
Options available:
  MySQL       → widely used, good
  SQLite      → file-based, great for development/testing
  PostgreSQL  → most feature-rich, production favorite
  Oracle      → enterprise, expensive
  SQL Server  → Microsoft ecosystem

Why PostgreSQL wins for backend engineers:
  ✅ Open source and free
  ✅ ACID compliant (gold standard for data integrity)
  ✅ Advanced features (JSON, arrays, full-text search)
  ✅ Best performance for complex queries
  ✅ Used by: Instagram, Uber, Netflix, GitHub, Spotify
  ✅ AWS RDS, Google Cloud SQL, Azure all support it
  ✅ Industry standard for Python backends
```

---

## Setup

```bash
cd ~/projects
mkdir database_learning
cd database_learning
python3 -m venv venv
source venv/bin/activate
pip install psycopg2-binary
```

### Installing PostgreSQL

```bash
# Mac (Homebrew):
brew install postgresql@16
brew services start postgresql@16

# Linux (Ubuntu/Debian):
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Windows:
# Download installer from postgresql.org/download/windows
# Run the installer, remember the password you set

# Verify installation:
psql --version
# psql (PostgreSQL) 16.x
```

### Connecting for the First Time

```bash
# Mac/Linux — connect as postgres superuser
psql -U postgres

# If that fails on Mac:
psql -U $(whoami)

# Linux — switch to postgres user first
sudo -u postgres psql

# You'll see the PostgreSQL prompt:
# postgres=#

# Create a database for our learning
CREATE DATABASE learning_db;

# Connect to it
\c learning_db

# Exit
\q
```

---

## 1. 🧠 Database Concepts

### ACID Properties — The Foundation

```
ACID = what guarantees a DATABASE makes about your data.

Every transaction must be:

A — Atomicity
    All operations in a transaction succeed,
    OR none of them do. No partial results.

    Example: Transfer $100 from Alice to Bob
      1. Deduct $100 from Alice ← must both
      2. Add $100 to Bob        ← succeed or both fail
    If step 2 fails, step 1 is REVERSED.
    Alice never loses money without Bob receiving it.

C — Consistency
    Database always moves from one VALID state to another.
    All data integrity rules are maintained.
    Example: You can't have an order without a customer.

I — Isolation
    Concurrent transactions don't interfere with each other.
    1000 users updating at the same time → no corruption.
    Each transaction sees a consistent snapshot.

D — Durability
    Committed data survives crashes.
    Once database says "saved", it's REALLY saved.
    Even if server loses power right after, data is there.

Why ACID matters:
  Without Atomicity → partial data (corrupted state)
  Without Consistency → invalid data (orphaned records)
  Without Isolation → race conditions (wrong totals)
  Without Durability → data loss (imagine losing transactions)
```

### Normalization — Organizing Data Properly

```
Normalization = organizing data to reduce redundancy
                and improve data integrity.

Three main normal forms (1NF, 2NF, 3NF).
Understanding these prevents the most common design mistakes.
```

```sql
-- ─────────────────────────────────────────
-- BAD: Unnormalized (0NF)
-- ─────────────────────────────────────────
-- orders table with repeated data:
-- id | customer_name   | customer_email   | product_names      | prices
-- 1  | Alice Smith     | alice@test.com   | Laptop, Mouse      | 999, 29
-- 2  | Alice Smith     | alice@test.com   | Keyboard           | 79
-- 3  | Bob Jones       | bob@test.com     | Laptop             | 999

-- Problems:
-- Alice's email stored TWICE → update anomaly (if she changes email)
-- Multiple products in ONE cell → can't query by product
-- "999" appears twice → update anomaly (if laptop price changes)

-- ─────────────────────────────────────────
-- 1NF: First Normal Form
-- Each cell must have ONE value (atomic)
-- ─────────────────────────────────────────
-- customers table
-- id | name        | email
-- 1  | Alice Smith | alice@test.com
-- 2  | Bob Jones   | bob@test.com

-- orders table
-- id | customer_id | order_date
-- 1  | 1           | 2024-01-15
-- 2  | 1           | 2024-01-20
-- 3  | 2           | 2024-01-18

-- order_items table
-- id | order_id | product_id | quantity | unit_price
-- 1  | 1        | 101        | 1        | 999.00
-- 2  | 1        | 102        | 2        | 29.00
-- 3  | 2        | 103        | 1        | 79.00

-- ─────────────────────────────────────────
-- 2NF: No partial dependencies
-- Every non-key column depends on the WHOLE primary key
-- ─────────────────────────────────────────
-- products table (separate from order_items)
-- id  | name     | price
-- 101 | Laptop   | 999.00
-- 102 | Mouse    | 29.00
-- 103 | Keyboard | 79.00

-- ─────────────────────────────────────────
-- 3NF: No transitive dependencies
-- Non-key columns don't depend on other non-key columns
-- ─────────────────────────────────────────
-- Example: if we stored customer city AND zip code,
-- and city depends on zip code → violates 3NF
-- Solution: separate zip_codes table
--   zip_codes: zip_code | city | state
--   customers: id | name | email | zip_code (foreign key)
```

---

## 2. 🐘 PostgreSQL Basics & Setup

### Create Our Practice Database

```sql
-- Connect to PostgreSQL
-- psql -U postgres

-- Create a user for our app (best practice — don't use postgres superuser)
CREATE USER myapp_user WITH PASSWORD 'myapp_password';

-- Create database
CREATE DATABASE learning_db OWNER myapp_user;

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE learning_db TO myapp_user;

-- Connect to the database
\c learning_db

-- Verify connection
SELECT current_database(), current_user, version();
```

### psql Commands You Need

```bash
# Connection
psql -U myapp_user -d learning_db          # connect
psql -U postgres -h localhost -p 5432 -d db # full connection string

# Inside psql:
\l              # list all databases
\c dbname       # connect to database
\dt             # list tables
\d tablename    # describe table structure
\di             # list indexes
\du             # list users/roles
\x              # toggle expanded display (pretty output)
\timing         # show query execution time
\e              # open query in editor
\i file.sql     # execute SQL from file
\o file.txt     # save output to file
\q              # quit

# Help
\h              # SQL command help
\h CREATE TABLE # help for specific command
\?              # psql command help
```

---

## 3. 📋 SQL Fundamentals

### Creating Tables

```sql
-- Connect to learning_db first
-- \c learning_db

-- ─────────────────────────────────────────
-- CREATE TABLE
-- ─────────────────────────────────────────

-- Users table
CREATE TABLE users (
    -- Primary key: unique identifier for each row
    id          SERIAL PRIMARY KEY,    -- auto-increment integer
    -- SERIAL = sequence (1, 2, 3, ...)
    -- PRIMARY KEY = unique + not null + indexed

    -- Basic columns
    name        VARCHAR(100) NOT NULL,         -- string, max 100 chars
    email       VARCHAR(255) NOT NULL UNIQUE,  -- unique email
    password_hash VARCHAR(255) NOT NULL,
    age         INTEGER CHECK (age >= 0 AND age <= 150),
    bio         TEXT,                          -- unlimited length text
    website     VARCHAR(500),

    -- Enum-like constraint
    role        VARCHAR(20) NOT NULL DEFAULT 'user'
                CHECK (role IN ('user', 'admin', 'moderator')),

    -- Boolean
    is_active   BOOLEAN NOT NULL DEFAULT TRUE,
    is_verified BOOLEAN NOT NULL DEFAULT FALSE,

    -- Numeric with precision
    balance     DECIMAL(10, 2) DEFAULT 0.00,
    -- DECIMAL(precision, scale): 10 total digits, 2 after decimal
    -- Max value: 99,999,999.99

    -- JSON (PostgreSQL superpower!)
    metadata    JSONB DEFAULT '{}',

    -- Timestamps
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    deleted_at  TIMESTAMP WITH TIME ZONE NULL  -- NULL means not deleted
);

-- Posts table
CREATE TABLE posts (
    id          SERIAL PRIMARY KEY,
    title       VARCHAR(500) NOT NULL,
    content     TEXT NOT NULL,
    slug        VARCHAR(500) UNIQUE,  -- URL-friendly title
    status      VARCHAR(20) DEFAULT 'draft'
                CHECK (status IN ('draft', 'published', 'archived')),

    -- Foreign key: links to users table
    author_id   INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    -- ON DELETE CASCADE: if user deleted, their posts are too
    -- ON DELETE SET NULL: if user deleted, author_id becomes NULL
    -- ON DELETE RESTRICT: prevent deleting user if they have posts

    view_count  INTEGER DEFAULT 0,
    like_count  INTEGER DEFAULT 0,

    -- Array column (PostgreSQL specific)
    tags        TEXT[] DEFAULT '{}',

    published_at TIMESTAMP WITH TIME ZONE,
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Comments table (self-referencing for nested comments)
CREATE TABLE comments (
    id          SERIAL PRIMARY KEY,
    content     TEXT NOT NULL,
    author_id   INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    post_id     INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    -- Self-referencing: parent comment (for nested comments)
    parent_id   INTEGER REFERENCES comments(id) ON DELETE CASCADE,

    is_edited   BOOLEAN DEFAULT FALSE,
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Many-to-many: users can like many posts, posts can be liked by many users
CREATE TABLE post_likes (
    user_id     INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    post_id     INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- Composite primary key: each user can like each post once
    PRIMARY KEY (user_id, post_id)
);

-- Categories table
CREATE TABLE categories (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(100) NOT NULL UNIQUE,
    slug        VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    parent_id   INTEGER REFERENCES categories(id)  -- nested categories
);

-- Post-Category junction table (many-to-many)
CREATE TABLE post_categories (
    post_id     INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    category_id INTEGER NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, category_id)
);

-- Verify tables created
\dt
```

### PostgreSQL Data Types

```sql
-- ─────────────────────────────────────────
-- NUMERIC TYPES
-- ─────────────────────────────────────────
-- SMALLINT          -32,768 to 32,767           (2 bytes)
-- INTEGER / INT     -2,147,483,648 to 2,147...  (4 bytes)
-- BIGINT            very large numbers           (8 bytes)
-- SERIAL            auto-increment INTEGER
-- BIGSERIAL         auto-increment BIGINT
-- DECIMAL(p,s)      exact decimal (for money!)
-- NUMERIC(p,s)      same as DECIMAL
-- REAL              floating point               (4 bytes, inexact)
-- DOUBLE PRECISION  double float                 (8 bytes, inexact)

-- ─────────────────────────────────────────
-- TEXT TYPES
-- ─────────────────────────────────────────
-- CHAR(n)           fixed length, padded with spaces
-- VARCHAR(n)        variable length, max n characters
-- TEXT              unlimited length

-- ─────────────────────────────────────────
-- DATE/TIME TYPES
-- ─────────────────────────────────────────
-- DATE              date only (2024-01-15)
-- TIME              time only (14:30:00)
-- TIMESTAMP         date + time, no timezone
-- TIMESTAMPTZ       date + time WITH timezone (always use this!)
-- INTERVAL          time duration ('1 day', '2 hours')

-- ─────────────────────────────────────────
-- OTHER TYPES
-- ─────────────────────────────────────────
-- BOOLEAN           true/false
-- UUID              universally unique identifier
-- JSONB             binary JSON (indexed, fast!)
-- JSON              text JSON (slower, less features)
-- TEXT[]            array of text
-- INTEGER[]         array of integers
-- BYTEA             binary data

-- Using UUID as primary key (modern approach):
CREATE TABLE sessions (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     INTEGER NOT NULL REFERENCES users(id),
    token       VARCHAR(500) NOT NULL,
    expires_at  TIMESTAMP WITH TIME ZONE NOT NULL
);
```

### ALTER TABLE — Modifying Tables

```sql
-- Add a column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Add column with constraint
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- Drop a column
ALTER TABLE users DROP COLUMN phone;

-- Rename a column
ALTER TABLE users RENAME COLUMN bio TO biography;

-- Change column type
ALTER TABLE users ALTER COLUMN age TYPE SMALLINT;

-- Add NOT NULL constraint
ALTER TABLE users ALTER COLUMN avatar_url SET NOT NULL;

-- Remove NOT NULL constraint
ALTER TABLE users ALTER COLUMN avatar_url DROP NOT NULL;

-- Add default value
ALTER TABLE users ALTER COLUMN avatar_url SET DEFAULT 'default.jpg';

-- Add unique constraint
ALTER TABLE users ADD CONSTRAINT users_phone_unique UNIQUE (phone);

-- Drop constraint
ALTER TABLE users DROP CONSTRAINT users_phone_unique;

-- Add check constraint
ALTER TABLE users ADD CONSTRAINT age_check CHECK (age >= 0);

-- Rename table
ALTER TABLE posts RENAME TO articles;
ALTER TABLE articles RENAME TO posts;  -- rename back

-- Drop table
DROP TABLE IF EXISTS sessions;

-- Drop table and everything that depends on it
DROP TABLE IF EXISTS users CASCADE;
```

---

## 4. 📝 INSERT, SELECT, UPDATE, DELETE

### INSERT — Adding Data

```sql
-- ─────────────────────────────────────────
-- BASIC INSERT
-- ─────────────────────────────────────────

-- Insert one row
INSERT INTO users (name, email, password_hash, role)
VALUES ('Alice Smith', 'alice@test.com', 'hashed_password_1', 'admin');

-- Insert with RETURNING (get back the created row)
INSERT INTO users (name, email, password_hash)
VALUES ('Bob Jones', 'bob@test.com', 'hashed_password_2')
RETURNING id, name, email, created_at;
-- This returns the new row immediately — very useful!

-- Insert multiple rows at once
INSERT INTO users (name, email, password_hash, role) VALUES
    ('Charlie Brown', 'charlie@test.com', 'hash3', 'user'),
    ('Dave Wilson', 'dave@test.com', 'hash4', 'user'),
    ('Eve Martinez', 'eve@test.com', 'hash5', 'moderator');

-- Insert with subquery
INSERT INTO posts (title, content, slug, author_id, status)
VALUES (
    'My First Post',
    'Hello World! This is my first post.',
    'my-first-post',
    (SELECT id FROM users WHERE email = 'alice@test.com'),
    'published'
)
RETURNING id, title, author_id;

-- ─────────────────────────────────────────
-- INSERT OR UPDATE (UPSERT)
-- ─────────────────────────────────────────

-- Insert if not exists, update if exists
INSERT INTO users (name, email, password_hash)
VALUES ('Alice Smith', 'alice@test.com', 'new_hash')
ON CONFLICT (email) DO UPDATE
    SET password_hash = EXCLUDED.password_hash,
        updated_at = NOW();
-- EXCLUDED = the row that WOULD have been inserted

-- Insert if not exists, do nothing if exists
INSERT INTO post_likes (user_id, post_id)
VALUES (1, 1)
ON CONFLICT (user_id, post_id) DO NOTHING;

-- Add more sample data
INSERT INTO posts (title, content, slug, author_id, status, tags) VALUES
    ('Python Tips', 'Great Python tips...', 'python-tips', 1, 'published',
     ARRAY['python', 'programming']),
    ('FastAPI Guide', 'Complete FastAPI guide...', 'fastapi-guide', 1, 'published',
     ARRAY['python', 'fastapi', 'api']),
    ('Database Design', 'How to design databases...', 'db-design', 2, 'draft',
     ARRAY['database', 'postgresql']),
    ('Draft Post', 'Work in progress...', 'draft-post', 2, 'draft',
     ARRAY['misc']);

INSERT INTO categories (name, slug) VALUES
    ('Technology', 'technology'),
    ('Programming', 'programming'),
    ('Database', 'database');

INSERT INTO post_categories (post_id, category_id) VALUES
    (1, 1), (1, 2),
    (2, 1), (2, 2),
    (3, 1), (3, 3);
```

### SELECT — Reading Data

```sql
-- ─────────────────────────────────────────
-- BASIC SELECT
-- ─────────────────────────────────────────

-- Select all columns
SELECT * FROM users;

-- Select specific columns (ALWAYS prefer this over * in production)
SELECT id, name, email, role FROM users;

-- Column aliases
SELECT
    id          AS user_id,
    name        AS full_name,
    email       AS email_address,
    created_at  AS joined_at
FROM users;

-- ─────────────────────────────────────────
-- WHERE — filtering rows
-- ─────────────────────────────────────────

-- Equality
SELECT * FROM users WHERE role = 'admin';

-- Multiple conditions
SELECT * FROM users WHERE role = 'admin' AND is_active = TRUE;

-- OR condition
SELECT * FROM users WHERE role = 'admin' OR role = 'moderator';

-- NOT
SELECT * FROM users WHERE NOT role = 'admin';

-- IN (multiple values)
SELECT * FROM users WHERE role IN ('admin', 'moderator');

-- NOT IN
SELECT * FROM users WHERE role NOT IN ('admin');

-- BETWEEN (inclusive)
SELECT * FROM users WHERE age BETWEEN 18 AND 30;
SELECT * FROM posts WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';

-- NULL checks
SELECT * FROM users WHERE deleted_at IS NULL;      -- not deleted
SELECT * FROM users WHERE deleted_at IS NOT NULL;  -- deleted

-- LIKE (pattern matching — case sensitive)
SELECT * FROM users WHERE email LIKE '%@gmail.com';
-- % = any sequence of characters
-- _ = exactly one character
SELECT * FROM users WHERE name LIKE 'A%';  -- starts with A
SELECT * FROM users WHERE name LIKE '%son'; -- ends with son
SELECT * FROM users WHERE name LIKE '_ob%'; -- second char is 'o', third is 'b'

-- ILIKE (case insensitive — PostgreSQL specific)
SELECT * FROM users WHERE email ILIKE '%GMAIL%';

-- ─────────────────────────────────────────
-- ORDER BY — sorting results
-- ─────────────────────────────────────────

-- Ascending (default)
SELECT id, name, created_at FROM users ORDER BY name;
SELECT id, name, created_at FROM users ORDER BY name ASC;

-- Descending
SELECT id, name, created_at FROM users ORDER BY created_at DESC;

-- Multiple columns
SELECT id, name, role, created_at
FROM users
ORDER BY role ASC, created_at DESC;
-- First by role (A-Z), then by date (newest first) within each role

-- Order by expression
SELECT id, name, LENGTH(name) as name_length
FROM users
ORDER BY LENGTH(name) DESC;

-- ─────────────────────────────────────────
-- LIMIT & OFFSET — pagination
-- ─────────────────────────────────────────

-- Get first 10 rows
SELECT * FROM users LIMIT 10;

-- Get rows 11-20 (page 2 with page_size=10)
SELECT * FROM users
ORDER BY id          -- always order when paginating!
LIMIT 10 OFFSET 10;

-- Page 3
SELECT * FROM users
ORDER BY id
LIMIT 10 OFFSET 20;

-- General formula: OFFSET = (page - 1) * page_size

-- ─────────────────────────────────────────
-- AGGREGATE FUNCTIONS
-- ─────────────────────────────────────────

SELECT COUNT(*) FROM users;                    -- total rows
SELECT COUNT(*) FROM users WHERE is_active;    -- active users
SELECT COUNT(DISTINCT role) FROM users;        -- unique roles

SELECT AVG(age) FROM users;                    -- average age
SELECT SUM(balance) FROM users;                -- total balance
SELECT MIN(created_at) FROM users;             -- earliest signup
SELECT MAX(created_at) FROM users;             -- latest signup

-- Multiple aggregates at once
SELECT
    COUNT(*) AS total_users,
    COUNT(*) FILTER (WHERE is_active) AS active_users,
    COUNT(*) FILTER (WHERE role = 'admin') AS admin_count,
    AVG(age) AS avg_age,
    MIN(created_at) AS first_signup,
    MAX(created_at) AS latest_signup
FROM users;

-- ─────────────────────────────────────────
-- GROUP BY — aggregate per group
-- ─────────────────────────────────────────

-- Count users per role
SELECT role, COUNT(*) AS count
FROM users
GROUP BY role
ORDER BY count DESC;

-- Average age per role
SELECT
    role,
    COUNT(*) AS user_count,
    ROUND(AVG(age), 1) AS avg_age,
    MIN(age) AS youngest,
    MAX(age) AS oldest
FROM users
GROUP BY role;

-- ─────────────────────────────────────────
-- HAVING — filter AFTER grouping
-- ─────────────────────────────────────────
-- WHERE filters individual rows (before grouping)
-- HAVING filters grouped results (after grouping)

-- Roles with more than 1 user
SELECT role, COUNT(*) AS count
FROM users
GROUP BY role
HAVING COUNT(*) > 1;

-- Categories with more than 2 posts
SELECT
    c.name AS category,
    COUNT(pc.post_id) AS post_count
FROM categories c
JOIN post_categories pc ON c.id = pc.category_id
GROUP BY c.id, c.name
HAVING COUNT(pc.post_id) > 2
ORDER BY post_count DESC;
```

### UPDATE — Modifying Data

```sql
-- ─────────────────────────────────────────
-- BASIC UPDATE
-- ─────────────────────────────────────────

-- Update specific row
UPDATE users
SET name = 'Alice Johnson', updated_at = NOW()
WHERE id = 1;

-- Update multiple columns
UPDATE users
SET
    is_verified = TRUE,
    role = 'admin',
    updated_at = NOW()
WHERE email = 'alice@test.com';

-- Update with calculation
UPDATE posts
SET view_count = view_count + 1
WHERE id = 1;

-- Update multiple rows
UPDATE users
SET is_active = FALSE
WHERE last_login < NOW() - INTERVAL '90 days';

-- Update with RETURNING
UPDATE users
SET role = 'admin', updated_at = NOW()
WHERE id = 1
RETURNING id, name, role, updated_at;

-- Update with subquery
UPDATE posts
SET status = 'published', published_at = NOW()
WHERE author_id = (
    SELECT id FROM users WHERE email = 'alice@test.com'
)
AND status = 'draft';

-- ─────────────────────────────────────────
-- SAFE UPDATE: always use WHERE!
-- ─────────────────────────────────────────
-- BAD: this updates ALL rows!
-- UPDATE users SET is_active = FALSE;

-- GOOD: always specify which rows
UPDATE users SET is_active = FALSE WHERE id = 42;
```

### DELETE — Removing Data

```sql
-- ─────────────────────────────────────────
-- BASIC DELETE
-- ─────────────────────────────────────────

-- Delete specific row
DELETE FROM users WHERE id = 5;

-- Delete with condition
DELETE FROM posts WHERE status = 'draft' AND author_id = 2;

-- Delete with RETURNING
DELETE FROM sessions WHERE expires_at < NOW()
RETURNING id, user_id;

-- Delete all rows (dangerous!)
-- DELETE FROM sessions;

-- ─────────────────────────────────────────
-- SOFT DELETE (preferred in production)
-- ─────────────────────────────────────────
-- Instead of actually deleting, mark as deleted.
-- Data preserved for:
--   - Audit logs
--   - Recovery
--   - Analytics

UPDATE users
SET deleted_at = NOW(), is_active = FALSE
WHERE id = 5;

-- Query only non-deleted users
SELECT * FROM users WHERE deleted_at IS NULL;

-- ─────────────────────────────────────────
-- TRUNCATE — delete all rows fast
-- ─────────────────────────────────────────
-- Faster than DELETE for clearing entire table
-- Cannot be filtered (all rows removed)
-- Resets sequences

TRUNCATE TABLE sessions;           -- fast, all rows
TRUNCATE TABLE sessions RESTART IDENTITY;  -- reset auto-increment too
```

---

## 5. 🔗 JOINs — Combining Tables

```sql
-- ─────────────────────────────────────────
-- INNER JOIN — only matching rows from BOTH tables
-- ─────────────────────────────────────────

-- Get posts with their author names
SELECT
    p.id,
    p.title,
    p.status,
    u.name AS author_name,
    u.email AS author_email
FROM posts p
INNER JOIN users u ON p.author_id = u.id;
-- Only returns posts that HAVE a matching user
-- (orphaned posts with no author are excluded)

-- ─────────────────────────────────────────
-- LEFT JOIN — all from left, matching from right
-- ─────────────────────────────────────────

-- ALL users and their posts (even users with no posts)
SELECT
    u.id,
    u.name,
    u.email,
    COUNT(p.id) AS post_count
FROM users u
LEFT JOIN posts p ON p.author_id = u.id
GROUP BY u.id, u.name, u.email
ORDER BY post_count DESC;
-- Users with no posts: post_count = 0

-- ─────────────────────────────────────────
-- RIGHT JOIN — all from right, matching from left
-- ─────────────────────────────────────────
-- Less common, LEFT JOIN is preferred
-- A RIGHT JOIN B = B LEFT JOIN A

-- ─────────────────────────────────────────
-- FULL OUTER JOIN — all from BOTH tables
-- ─────────────────────────────────────────
-- Returns all rows from both tables
-- NULLs where there's no match on either side
SELECT
    u.name AS user_name,
    p.title AS post_title
FROM users u
FULL OUTER JOIN posts p ON p.author_id = u.id;

-- ─────────────────────────────────────────
-- CROSS JOIN — every combination (cartesian product)
-- ─────────────────────────────────────────
-- Rarely used intentionally
-- 5 users × 3 categories = 15 rows
SELECT u.name, c.name
FROM users u
CROSS JOIN categories c;

-- ─────────────────────────────────────────
-- SELF JOIN — table joined to itself
-- ─────────────────────────────────────────
-- Get comments with their parent comments
SELECT
    c.id,
    c.content AS comment,
    p.content AS parent_comment
FROM comments c
LEFT JOIN comments p ON c.parent_id = p.id;

-- ─────────────────────────────────────────
-- MULTIPLE JOINS
-- ─────────────────────────────────────────

-- Get posts with author, categories, and comment count
SELECT
    p.id,
    p.title,
    p.status,
    p.created_at,
    u.name AS author,
    STRING_AGG(c.name, ', ') AS categories,
    COUNT(DISTINCT cm.id) AS comment_count,
    p.view_count,
    p.like_count
FROM posts p
JOIN users u ON p.author_id = u.id
LEFT JOIN post_categories pc ON p.id = pc.post_id
LEFT JOIN categories c ON pc.category_id = c.id
LEFT JOIN comments cm ON p.id = cm.post_id
GROUP BY p.id, u.name
ORDER BY p.created_at DESC;

-- ─────────────────────────────────────────
-- JOIN VISUAL GUIDE
-- ─────────────────────────────────────────
/*
INNER JOIN:
  users  | posts
  ───────┼──────
    A    │   A  ← matched
    B    │   B  ← matched
    C    │       ← C has no posts (excluded)
         │   D  ← D has no user (excluded)

LEFT JOIN:
  users  | posts
  ───────┼──────
    A    │   A  ← matched
    B    │   B  ← matched
    C    │ NULL ← C included with NULLs for post columns

RIGHT JOIN:
  users  | posts
  ───────┼──────
  NULL   │   D  ← D included with NULLs for user columns
    A    │   A  ← matched
    B    │   B  ← matched

FULL OUTER JOIN: includes ALL rows from both
*/
```

---

## 6. 🏗️ Subqueries & CTEs

### Subqueries

```sql
-- ─────────────────────────────────────────
-- SUBQUERY IN WHERE
-- ─────────────────────────────────────────

-- Find all posts by admin users
SELECT id, title FROM posts
WHERE author_id IN (
    SELECT id FROM users WHERE role = 'admin'
);

-- Find users who have never posted
SELECT id, name, email FROM users
WHERE id NOT IN (
    SELECT DISTINCT author_id FROM posts
);

-- ─────────────────────────────────────────
-- CORRELATED SUBQUERY
-- Runs once per row in outer query
-- ─────────────────────────────────────────

-- Users with their post count (correlated)
SELECT
    u.id,
    u.name,
    (SELECT COUNT(*) FROM posts WHERE author_id = u.id) AS post_count
FROM users u;

-- ─────────────────────────────────────────
-- SUBQUERY IN FROM (derived table)
-- ─────────────────────────────────────────

-- Average posts per user
SELECT AVG(post_count) AS avg_posts_per_user
FROM (
    SELECT author_id, COUNT(*) AS post_count
    FROM posts
    GROUP BY author_id
) AS post_counts;

-- ─────────────────────────────────────────
-- EXISTS / NOT EXISTS
-- ─────────────────────────────────────────

-- Users who have at least one published post
SELECT u.id, u.name
FROM users u
WHERE EXISTS (
    SELECT 1 FROM posts
    WHERE author_id = u.id AND status = 'published'
);

-- Users with no posts
SELECT u.id, u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM posts WHERE author_id = u.id
);
```

### CTEs — Common Table Expressions

```sql
-- ─────────────────────────────────────────
-- CTE: WITH clause
-- Named temporary result set
-- Improves readability over nested subqueries
-- ─────────────────────────────────────────

-- Basic CTE
WITH admin_users AS (
    SELECT id, name, email
    FROM users
    WHERE role = 'admin'
)
SELECT * FROM admin_users;

-- CTE with multiple steps
WITH
-- Step 1: Get active published posts
published_posts AS (
    SELECT id, title, author_id, view_count, like_count
    FROM posts
    WHERE status = 'published'
),

-- Step 2: Get post statistics per author
author_stats AS (
    SELECT
        author_id,
        COUNT(*) AS post_count,
        SUM(view_count) AS total_views,
        SUM(like_count) AS total_likes
    FROM published_posts
    GROUP BY author_id
),

-- Step 3: Join with user info
final_result AS (
    SELECT
        u.name AS author_name,
        u.email,
        s.post_count,
        s.total_views,
        s.total_likes,
        ROUND(s.total_likes::NUMERIC / NULLIF(s.total_views, 0) * 100, 2)
            AS engagement_rate
    FROM users u
    JOIN author_stats s ON u.id = s.author_id
)

SELECT *
FROM final_result
ORDER BY total_views DESC;

-- ─────────────────────────────────────────
-- RECURSIVE CTE — for hierarchical data
-- ─────────────────────────────────────────

-- Category hierarchy (parent → children)
WITH RECURSIVE category_tree AS (
    -- Base case: root categories (no parent)
    SELECT id, name, parent_id, 0 AS depth, name AS path
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive case: children
    SELECT
        c.id,
        c.name,
        c.parent_id,
        ct.depth + 1,
        ct.path || ' > ' || c.name
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT id, name, depth, path
FROM category_tree
ORDER BY path;

-- ─────────────────────────────────────────
-- REAL BACKEND USE: Pagination with count
-- ─────────────────────────────────────────

-- Efficient: get page of results AND total count in one query
WITH post_data AS (
    SELECT
        p.id, p.title, p.status, p.created_at,
        u.name AS author
    FROM posts p
    JOIN users u ON p.author_id = u.id
    WHERE p.status = 'published'
),
total AS (
    SELECT COUNT(*) AS total_count FROM post_data
)
SELECT
    pd.*,
    t.total_count,
    CEIL(t.total_count::NUMERIC / 10) AS total_pages
FROM post_data pd, total t
ORDER BY pd.created_at DESC
LIMIT 10 OFFSET 0;
```

---

## 7. 📈 Indexes & Query Optimization

### What is an Index?

```
Without index: finding email 'alice@test.com' in 1M rows
  → scan ALL 1M rows: O(n) = 1,000,000 comparisons

With index: B-tree structure
  → jump directly to the right area: O(log n) = ~20 comparisons

Index = separate data structure that
        makes certain lookups much faster.
        Trade-off: faster reads, slower writes, more storage.
```

### Creating Indexes

```sql
-- ─────────────────────────────────────────
-- BASIC INDEX
-- ─────────────────────────────────────────

-- B-tree index (default, most common)
-- Use for: equality (=), range (<, >, BETWEEN), ORDER BY
CREATE INDEX idx_posts_author_id ON posts(author_id);
-- Now: WHERE author_id = 5 is fast

CREATE INDEX idx_posts_status ON posts(status);
-- Now: WHERE status = 'published' is fast

CREATE INDEX idx_posts_created_at ON posts(created_at);
-- Now: ORDER BY created_at is fast

-- ─────────────────────────────────────────
-- UNIQUE INDEX (implicit with UNIQUE constraint)
-- ─────────────────────────────────────────
-- Already created by: email VARCHAR(255) UNIQUE
-- But you can create explicitly:
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- ─────────────────────────────────────────
-- COMPOSITE INDEX — multiple columns
-- ─────────────────────────────────────────
-- Most useful when you query by MULTIPLE columns together
CREATE INDEX idx_posts_author_status ON posts(author_id, status);
-- Fast for: WHERE author_id = 1 AND status = 'published'
-- Also fast for: WHERE author_id = 1 (uses leftmost column)
-- NOT used for: WHERE status = 'published' alone (not leftmost)

-- ─────────────────────────────────────────
-- PARTIAL INDEX — index on subset of rows
-- ─────────────────────────────────────────
-- Smaller, faster index for specific conditions
CREATE INDEX idx_posts_published ON posts(created_at)
WHERE status = 'published';
-- Only indexes published posts — smaller and faster!

CREATE INDEX idx_users_active ON users(email)
WHERE is_active = TRUE AND deleted_at IS NULL;
-- Only active, non-deleted users

-- ─────────────────────────────────────────
-- GIN INDEX — for arrays, JSONB, full-text search
-- ─────────────────────────────────────────
-- B-tree doesn't work for: arrays, JSON, text search

-- Index for JSONB queries
CREATE INDEX idx_users_metadata ON users USING gin(metadata);
-- Fast for: metadata @> '{"plan": "premium"}'

-- Index for array containment
CREATE INDEX idx_posts_tags ON posts USING gin(tags);
-- Fast for: tags @> ARRAY['python'] (contains python tag)

-- ─────────────────────────────────────────
-- MANAGING INDEXES
-- ─────────────────────────────────────────

-- List all indexes
SELECT tablename, indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY tablename;

-- Drop index
DROP INDEX idx_posts_status;

-- Rebuild index (fix bloat after many updates/deletes)
REINDEX INDEX idx_posts_author_id;

-- Index usage statistics
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan AS times_used,
    idx_tup_read AS rows_read,
    idx_tup_fetch AS rows_fetched
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

### EXPLAIN & EXPLAIN ANALYZE

```sql
-- ─────────────────────────────────────────
-- EXPLAIN — show query execution plan
-- ─────────────────────────────────────────

EXPLAIN SELECT * FROM posts WHERE status = 'published';
-- Shows:
-- Seq Scan on posts (cost=0.00..1.05 rows=1 width=520)
--   Filter: ((status)::text = 'published'::text)
-- "Seq Scan" = sequential scan (reading all rows)
-- "cost=0.00..1.05" = startup cost..total cost (arbitrary units)

-- Create index and re-explain
CREATE INDEX idx_posts_status2 ON posts(status);
EXPLAIN SELECT * FROM posts WHERE status = 'published';
-- Index Scan using idx_posts_status2 on posts
-- "Index Scan" = using the index (much faster!)

-- ─────────────────────────────────────────
-- EXPLAIN ANALYZE — actually RUN and show timing
-- ─────────────────────────────────────────

EXPLAIN ANALYZE SELECT * FROM posts WHERE status = 'published';
-- Shows ACTUAL timing:
-- Seq Scan on posts (cost=0.00..1.05 rows=3 width=520)
--                   (actual time=0.012..0.018 rows=3 loops=1)
-- Planning Time: 0.05 ms
-- Execution Time: 0.05 ms

-- Key terms to understand:
-- Seq Scan:        Reading ALL rows (slow for large tables)
-- Index Scan:      Using index, then fetching rows
-- Index Only Scan: Using index only (no row fetch, fastest!)
-- Bitmap Scan:     Multiple index scans combined
-- Nested Loop:     Join by iterating
-- Hash Join:       Join using hash table (good for large sets)
-- Merge Join:      Join on sorted data (good for large sorted sets)
-- cost=X..Y:       Estimated cost (X=startup, Y=total)
-- rows=N:          Estimated rows
-- actual time=X..Y: Real timing in milliseconds
-- loops=N:         How many times this node ran

-- ─────────────────────────────────────────
-- EXPLAIN ANALYZE with BUFFERS
-- ─────────────────────────────────────────
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT p.*, u.name
FROM posts p
JOIN users u ON p.author_id = u.id
WHERE p.status = 'published'
ORDER BY p.created_at DESC
LIMIT 10;
```

---

## 8. 🔍 Window Functions

```sql
-- ─────────────────────────────────────────
-- WINDOW FUNCTIONS — calculations across rows
-- WITHOUT collapsing them (unlike GROUP BY)
-- ─────────────────────────────────────────

-- INSERT more sample data for better examples
INSERT INTO users (name, email, password_hash, role, age) VALUES
    ('Frank Lee', 'frank@test.com', 'hash6', 'user', 28),
    ('Grace Kim', 'grace@test.com', 'hash7', 'user', 32),
    ('Henry Ford', 'henry@test.com', 'hash8', 'moderator', 45);

-- ─────────────────────────────────────────
-- ROW_NUMBER — unique sequential number
-- ─────────────────────────────────────────

-- Rank users by age within each role
SELECT
    name,
    email,
    role,
    age,
    ROW_NUMBER() OVER (
        PARTITION BY role           -- restart numbering per role
        ORDER BY age DESC           -- sort within each partition
    ) AS rank_within_role
FROM users;

-- ─────────────────────────────────────────
-- RANK vs DENSE_RANK
-- ─────────────────────────────────────────

-- Add same-age data
INSERT INTO users (name, email, password_hash, age) VALUES
    ('Igor Smith', 'igor@test.com', 'hash9', 28);

SELECT
    name,
    age,
    RANK() OVER (ORDER BY age DESC) AS rank,
    -- 1, 2, 2, 4 (skips 3 after tie)

    DENSE_RANK() OVER (ORDER BY age DESC) AS dense_rank,
    -- 1, 2, 2, 3 (no skip after tie)

    ROW_NUMBER() OVER (ORDER BY age DESC) AS row_num
    -- 1, 2, 3, 4 (always unique)
FROM users;

-- ─────────────────────────────────────────
-- LAG / LEAD — access previous/next row
-- ─────────────────────────────────────────

-- Compare each post's views with previous post's views
SELECT
    title,
    created_at::DATE AS date,
    view_count,
    LAG(view_count, 1, 0) OVER (
        ORDER BY created_at
    ) AS prev_views,
    view_count - LAG(view_count, 1, 0) OVER (
        ORDER BY created_at
    ) AS views_change
FROM posts
ORDER BY created_at;

-- ─────────────────────────────────────────
-- SUM/AVG — running totals
-- ─────────────────────────────────────────

-- Running total of views per author
SELECT
    u.name AS author,
    p.title,
    p.view_count,
    SUM(p.view_count) OVER (
        PARTITION BY p.author_id    -- per author
        ORDER BY p.created_at       -- in chronological order
    ) AS running_total_views,
    AVG(p.view_count) OVER (
        PARTITION BY p.author_id
    ) AS author_avg_views
FROM posts p
JOIN users u ON p.author_id = u.id
ORDER BY u.name, p.created_at;

-- ─────────────────────────────────────────
-- NTILE — divide into N buckets
-- ─────────────────────────────────────────

-- Assign users to quartiles by age
SELECT
    name,
    age,
    NTILE(4) OVER (ORDER BY age) AS age_quartile
FROM users
WHERE age IS NOT NULL;

-- ─────────────────────────────────────────
-- REAL BACKEND USE: Top post per author
-- ─────────────────────────────────────────

-- Get each author's most viewed post
WITH ranked_posts AS (
    SELECT
        p.*,
        u.name AS author_name,
        ROW_NUMBER() OVER (
            PARTITION BY p.author_id
            ORDER BY p.view_count DESC
        ) AS view_rank
    FROM posts p
    JOIN users u ON p.author_id = u.id
)
SELECT author_name, title, view_count
FROM ranked_posts
WHERE view_rank = 1;  -- only the top post per author
```

---

## 9. 🔒 Transactions & Isolation Levels

```sql
-- ─────────────────────────────────────────
-- TRANSACTIONS
-- ─────────────────────────────────────────

-- START TRANSACTION (or BEGIN)
BEGIN;

    -- All operations inside are atomic
    UPDATE users SET balance = balance - 100 WHERE id = 1;
    UPDATE users SET balance = balance + 100 WHERE id = 2;

    -- If everything looks good:
COMMIT;
-- OR if something went wrong:
-- ROLLBACK;

-- ─────────────────────────────────────────
-- SAVEPOINTS — partial rollback
-- ─────────────────────────────────────────

BEGIN;
    INSERT INTO posts (title, content, author_id)
    VALUES ('Post 1', 'Content 1', 1);

    SAVEPOINT my_savepoint;

    INSERT INTO posts (title, content, author_id)
    VALUES ('Post 2', 'Content 2', 999);  -- fails: user 999 doesn't exist

    -- Instead of rolling back EVERYTHING:
    ROLLBACK TO SAVEPOINT my_savepoint;

    -- First insert still valid
    INSERT INTO posts (title, content, author_id)
    VALUES ('Post 2 fixed', 'Content 2', 1);

COMMIT;

-- ─────────────────────────────────────────
-- ISOLATION LEVELS
-- ─────────────────────────────────────────
/*
Four isolation levels (weakest to strongest):

1. READ UNCOMMITTED
   Can read uncommitted changes from other transactions.
   "Dirty reads" possible.
   PostgreSQL doesn't implement this — uses READ COMMITTED.

2. READ COMMITTED (PostgreSQL default)
   Only sees committed data.
   Each query sees the latest committed data.
   "Non-repeatable reads" possible (same query = different results).

3. REPEATABLE READ
   All queries in transaction see the SAME snapshot.
   No "non-repeatable reads".
   "Phantom reads" still possible in some databases.

4. SERIALIZABLE
   Strongest. Transactions appear to run one at a time.
   Most consistent but slowest.
   Use for: financial transactions, critical operations.
*/

-- Set isolation level for a transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    -- your operations here
COMMIT;

-- Default (read committed) is usually fine for most operations
BEGIN;
    -- reads and writes
COMMIT;

-- ─────────────────────────────────────────
-- LOCKING
-- ─────────────────────────────────────────

-- Explicit row locking (prevent others from modifying)
BEGIN;
    -- Lock specific rows for update
    SELECT * FROM users WHERE id = 1 FOR UPDATE;
    -- Other transactions trying to update user 1 will WAIT

    UPDATE users SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Lock without waiting (fail immediately if locked)
SELECT * FROM users WHERE id = 1 FOR UPDATE NOWAIT;

-- Advisory locks (application-level)
-- Lock by a custom integer key
SELECT pg_advisory_lock(12345);      -- acquire lock
-- ... critical section ...
SELECT pg_advisory_unlock(12345);    -- release lock
```

---

## 10. 📊 Views & Materialized Views

```sql
-- ─────────────────────────────────────────
-- VIEWS — saved queries
-- ─────────────────────────────────────────

-- Create a view for commonly accessed data
CREATE VIEW published_posts_with_author AS
SELECT
    p.id,
    p.title,
    p.slug,
    p.content,
    p.view_count,
    p.like_count,
    p.published_at,
    u.id AS author_id,
    u.name AS author_name,
    u.email AS author_email
FROM posts p
JOIN users u ON p.author_id = u.id
WHERE p.status = 'published'
AND p.deleted_at IS NULL;

-- Use view like a table
SELECT * FROM published_posts_with_author LIMIT 5;
SELECT title, author_name FROM published_posts_with_author
WHERE author_name = 'Alice Smith';

-- Views are always up-to-date (run query fresh each time)
-- Good for: hiding complexity, reusable queries

-- Drop view
DROP VIEW IF EXISTS published_posts_with_author;

-- ─────────────────────────────────────────
-- MATERIALIZED VIEWS — stored results
-- ─────────────────────────────────────────

-- Like a view but STORES the results (cached)
-- Must be manually refreshed
-- Use for: expensive queries, reporting, dashboards

CREATE MATERIALIZED VIEW author_statistics AS
SELECT
    u.id AS author_id,
    u.name AS author_name,
    u.email,
    COUNT(p.id) AS total_posts,
    COUNT(p.id) FILTER (WHERE p.status = 'published') AS published_posts,
    SUM(p.view_count) AS total_views,
    SUM(p.like_count) AS total_likes,
    MAX(p.created_at) AS last_post_date,
    ROUND(AVG(p.view_count), 2) AS avg_views_per_post
FROM users u
LEFT JOIN posts p ON u.id = p.author_id
GROUP BY u.id, u.name, u.email;

-- Query (reads from stored results — very fast!)
SELECT * FROM author_statistics ORDER BY total_views DESC;

-- Refresh when underlying data changes
REFRESH MATERIALIZED VIEW author_statistics;

-- Refresh without locking (allows reads during refresh)
REFRESH MATERIALIZED VIEW CONCURRENTLY author_statistics;
-- Requires unique index on the view:
CREATE UNIQUE INDEX ON author_statistics(author_id);
```

---

## 11. 🔍 Full-Text Search

```sql
-- ─────────────────────────────────────────
-- POSTGRESQL FULL TEXT SEARCH
-- ─────────────────────────────────────────

-- Basic full text search
SELECT title, content
FROM posts
WHERE to_tsvector('english', title || ' ' || content)
      @@ to_tsquery('english', 'python & tutorial');
-- to_tsvector: convert text to searchable format
-- to_tsquery: convert search term to query
-- @@: match operator

-- More user-friendly search (handles user input)
SELECT title
FROM posts
WHERE to_tsvector('english', title || ' ' || content)
      @@ plainto_tsquery('english', 'python tutorial');
-- plainto_tsquery: doesn't require & syntax

-- Add full-text search index for performance
ALTER TABLE posts ADD COLUMN search_vector TSVECTOR;

-- Update the column
UPDATE posts SET search_vector =
    to_tsvector('english', title || ' ' || COALESCE(content, ''));

-- Create GIN index on it
CREATE INDEX idx_posts_search ON posts USING gin(search_vector);

-- Query using the index
SELECT title FROM posts
WHERE search_vector @@ plainto_tsquery('python');

-- Highlight matches (for display)
SELECT
    title,
    ts_headline(
        'english',
        content,
        plainto_tsquery('python'),
        'MaxWords=50, MinWords=20, ShortWord=3'
    ) AS excerpt
FROM posts
WHERE search_vector @@ plainto_tsquery('python');

-- Rank results by relevance
SELECT
    title,
    ts_rank(search_vector, plainto_tsquery('python')) AS rank
FROM posts
WHERE search_vector @@ plainto_tsquery('python')
ORDER BY rank DESC;
```

---

## 12. 🛠️ Stored Procedures & Triggers

```sql
-- ─────────────────────────────────────────
-- STORED PROCEDURES — reusable SQL logic
-- ─────────────────────────────────────────

-- Function to update updated_at automatically
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- ─────────────────────────────────────────
-- TRIGGERS — run function on events
-- ─────────────────────────────────────────

-- Automatically update updated_at on any update
CREATE TRIGGER trigger_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER trigger_posts_updated_at
    BEFORE UPDATE ON posts
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();

-- Test it
UPDATE users SET name = 'Alice Updated' WHERE id = 1;
SELECT name, updated_at FROM users WHERE id = 1;
-- updated_at is now automatically current time!

-- Function to update post like count
CREATE OR REPLACE FUNCTION update_post_like_count()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE posts SET like_count = like_count + 1
        WHERE id = NEW.post_id;
    ELSIF TG_OP = 'DELETE' THEN
        UPDATE posts SET like_count = like_count - 1
        WHERE id = OLD.post_id;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_post_likes
    AFTER INSERT OR DELETE ON post_likes
    FOR EACH ROW
    EXECUTE FUNCTION update_post_like_count();

-- Test
INSERT INTO post_likes (user_id, post_id) VALUES (2, 1);
SELECT like_count FROM posts WHERE id = 1;  -- should be 1

DELETE FROM post_likes WHERE user_id = 2 AND post_id = 1;
SELECT like_count FROM posts WHERE id = 1;  -- should be 0
```

---

## 13. 🗄️ Backup & Restore

```bash
# ─────────────────────────────────────────
# BACKUP
# ─────────────────────────────────────────

# Backup entire database (SQL format)
pg_dump -U postgres -d learning_db > backup.sql

# Backup in compressed format (recommended)
pg_dump -U postgres -d learning_db -Fc -f backup.dump

# Backup only schema (no data)
pg_dump -U postgres -d learning_db --schema-only > schema.sql

# Backup only data (no schema)
pg_dump -U postgres -d learning_db --data-only > data.sql

# Backup specific table
pg_dump -U postgres -d learning_db -t users > users_backup.sql

# ─────────────────────────────────────────
# RESTORE
# ─────────────────────────────────────────

# Restore from SQL file
psql -U postgres -d learning_db < backup.sql

# Restore from compressed dump
pg_restore -U postgres -d learning_db backup.dump

# Restore to new database
createdb -U postgres new_db
pg_restore -U postgres -d new_db backup.dump
```

---

## 📊 Visual Summary

```
┌────────────────────────────────────────────────────────────────┐
│                 POSTGRESQL ESSENTIALS                          │
│                                                                │
│  DATA TYPES:                                                   │
│  SERIAL/BIGSERIAL  auto-increment  TEXT      unlimited str     │
│  INTEGER/BIGINT    numbers          VARCHAR(n) limited str     │
│  DECIMAL(p,s)      exact decimal    BOOLEAN   true/false       │
│  TIMESTAMPTZ       date+time+tz     JSONB     JSON data        │
│  UUID              unique id        TEXT[]    text array       │
│                                                                │
│  SQL OPERATIONS:                                               │
│  INSERT INTO table VALUES (...)  → create                      │
│  SELECT cols FROM table WHERE    → read                        │
│  UPDATE table SET col=val WHERE  → update                      │
│  DELETE FROM table WHERE         → delete                      │
│                                                                │
│  JOINS:                                                        │
│  INNER JOIN  → matching rows only                              │
│  LEFT JOIN   → all from left + matching right                  │
│  RIGHT JOIN  → all from right + matching left                  │
│  FULL OUTER  → all from both                                   │
│                                                                │
│  PERFORMANCE:                                                  │
│  CREATE INDEX       → B-tree for equality/range               │
│  CREATE INDEX USING gin → JSON, arrays, full-text            │
│  EXPLAIN ANALYZE    → see query execution plan                │
│  PARTIAL INDEX      → index subset of rows                    │
│                                                                │
│  ADVANCED:                                                     │
│  CTE (WITH)         → named temp results                      │
│  Window Functions   → calculations across rows                 │
│  Transactions       → atomic operations                       │
│  Views              → saved queries                            │
│  Materialized Views → cached query results                    │
│  Full-Text Search   → tsvector + tsquery                      │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

```
1.  What does ACID stand for? Why does each property matter?
2.  What is the difference between 1NF, 2NF, and 3NF?
3.  When would you use INNER JOIN vs LEFT JOIN?
4.  What is the difference between WHERE and HAVING?
5.  What is a CTE and when is it better than a subquery?
6.  What is an index and what are the trade-offs?
7.  When would you use a GIN index vs B-tree?
8.  What does EXPLAIN ANALYZE tell you?
9.  What is the difference between a view and a materialized view?
10. What is a window function? Give an example.
11. What is REPEATABLE READ vs READ COMMITTED?
12. What is soft delete and why is it preferred in production?
13. What is the difference between TRUNCATE and DELETE?
14. What does ON DELETE CASCADE do?
```

---

## 🛠️ Practice Exercises

```sql
-- Exercise 1: Schema Design
-- Design a complete schema for a simple e-commerce system:
-- Tables: customers, products, orders, order_items, reviews
-- Requirements:
--   - Customers have addresses (multiple per customer)
--   - Products have categories (many-to-many)
--   - Orders track status history
--   - Reviews are per product per customer (one each)
-- Write the CREATE TABLE statements with all constraints.

-- Exercise 2: Complex Query
-- Using the blog schema we created, write a query that returns:
-- For each user: their name, email, number of posts,
-- total views across all posts, their most popular post title,
-- and their "engagement score" = total_likes / total_views * 100
-- Only include users with at least one published post.
-- Order by engagement score descending.

-- Exercise 3: Window Function
-- Write a query that shows, for each post:
-- - The post title
-- - Its view count
-- - The average view count for all posts by the same author
-- - How far above/below the author's average this post is
-- - Its rank among all the author's posts by views

-- Exercise 4: CTE + Transaction
-- Write a transaction that:
-- 1. Creates a new user
-- 2. Creates their first post
-- 3. Adds it to two categories
-- Use CTEs within the transaction.
-- If any step fails, the whole thing rolls back.

-- Exercise 5: Performance
-- Given this slow query:
-- SELECT * FROM posts
-- WHERE author_id IN (SELECT id FROM users WHERE role = 'admin')
-- AND created_at > NOW() - INTERVAL '30 days'
-- AND status = 'published'
-- ORDER BY view_count DESC
-- LIMIT 20;
--
-- a) Identify what indexes would help
-- b) Create those indexes
-- c) Use EXPLAIN ANALYZE to verify improvement
-- d) Rewrite using JOIN instead of subquery
```

---

## ✅ Phase 3.1 Complete!

**You now know:**

```
✅ ACID properties and why they matter
✅ Database normalization (1NF, 2NF, 3NF)
✅ PostgreSQL installation and setup
✅ All SQL data types including PostgreSQL-specific ones
✅ CREATE TABLE with constraints, foreign keys, defaults
✅ ALTER TABLE for schema changes
✅ INSERT (including UPSERT, RETURNING)
✅ SELECT with WHERE, ORDER BY, LIMIT, OFFSET
✅ All aggregate functions (COUNT, SUM, AVG, MIN, MAX)
✅ GROUP BY and HAVING
✅ All JOIN types (INNER, LEFT, RIGHT, FULL, SELF, CROSS)
✅ Subqueries and CTEs (including recursive)
✅ Indexes (B-tree, GIN, partial, composite)
✅ EXPLAIN and EXPLAIN ANALYZE
✅ Window functions (ROW_NUMBER, RANK, LAG, LEAD, SUM OVER)
✅ Transactions and isolation levels
✅ Views and Materialized Views
✅ Full-text search
✅ Stored procedures and triggers
✅ Backup and restore
```

---

## What's Next?

```
Option 1: Continue  → Phase 3.2 (Python + Database)
                      psycopg2, SQLAlchemy Core, ORM,
                      Alembic migrations — connecting
                      your Python code to PostgreSQL

Option 2: Practice  → Work through the exercises above

Option 3: Questions → Anything unclear?
```

**What do you want to do? 🚀**