```
You know SQL and can write queries.
You know SQLAlchemy and can build models.
You know Redis and MongoDB.

But in professional work you also need:

┌─────────────────────────────────────────────────────────┐
│  pgAdmin / DBeaver    → GUI to explore your database    │
│  MongoDB Compass      → GUI to explore MongoDB          │
│  Redis CLI + Insight  → GUI to explore Redis            │
│  Migration strategies → Safely change schema in prod    │
└─────────────────────────────────────────────────────────┘

These tools:
  ✅ Speed up development (no typing SQL for everything)
  ✅ Make debugging visual and easy
  ✅ Required on every team you'll join
  ✅ Used daily by senior engineers
```

---

## 1. 🐘 pgAdmin — PostgreSQL GUI

### What is pgAdmin?

text

```
pgAdmin = the official PostgreSQL GUI tool.

Instead of typing SQL commands in a terminal,
you click through a visual interface to:
  - Browse tables, views, indexes
  - Run queries with syntax highlighting
  - See query execution plans visually
  - Monitor server performance
  - Manage users and permissions
  - Import/export data
```

### Installation

Bash

```
# Mac:
brew install --cask pgadmin4

# Linux (Ubuntu):
sudo apt install pgadmin4

# Windows:
# Download from pgadmin.org/download/pgadmin-4-windows/

# Or use the web version:
# pip install pgadmin4
```

### Connecting to PostgreSQL

text

```
1. Open pgAdmin
2. Click "Add New Server" (right-click on "Servers")
3. Fill in:
   Name:     My Local DB (any name)
   Host:     localhost
   Port:     5432
   Database: learning_db
   Username: myapp_user
   Password: myapp_password
4. Click "Save"
```

### Key Features to Know

text

```
LEFT PANEL (Browser):
  Servers
    └── My Local DB
          └── Databases
                └── learning_db
                      ├── Schemas
                      │     └── public
                      │           ├── Tables      ← your tables
                      │           ├── Views       ← your views
                      │           ├── Indexes     ← your indexes
                      │           └── Functions   ← stored procedures
                      └── Login/Group Roles       ← users

RIGHT PANEL (Main area):
  - Query Tool (Ctrl+Shift+Q) ← you'll use this most
  - Table data viewer
  - Properties panel
  - Statistics panel

QUERY TOOL:
  - Write and run SQL
  - See results in a table
  - Export results to CSV
  - View Explain plan graphically
  - Save queries as files

EXPLAIN VISUALIZER:
  - Run your query with Explain
  - Click "Explain" button (not just the text output)
  - See a VISUAL query plan with:
    - Node types (Seq Scan, Index Scan, Hash Join...)
    - Cost estimates
    - Row counts
    - Time percentages
  - Instantly spot the slow part of your query
```

### Using pgAdmin Query Tool

SQL

```
-- Open Query Tool: Tools → Query Tool (or Ctrl+Shift+Q)

-- Run queries and see results immediately
SELECT
    u.name,
    u.email,
    u.role,
    COUNT(p.id) AS post_count
FROM users u
LEFT JOIN posts p ON p.author_id = u.id
GROUP BY u.id, u.name, u.email, u.role
ORDER BY post_count DESC;

-- To see visual explain plan:
-- 1. Write your query
-- 2. Click the "Explain" button (NOT "Execute")
-- 3. Click "Explain Analyze" for actual timing
-- 4. Switch to "Graphical" tab
-- 5. Hover over nodes to see details

-- Export results:
-- After running a query, right-click results → Download as CSV
```

### pgAdmin Dashboard

text

```
Dashboard tab shows:
  - Server activity (connections, transactions)
  - Sessions graph (active connections over time)
  - Transactions per second
  - Tuples in/out per second (reads/writes)
  - Block I/O

Use this to:
  - Monitor query load
  - See if you have connection pool exhaustion
  - Identify slow periods
  - Watch for blocked queries
```

---

## 2. 🦫 DBeaver — Universal Database GUI

### Why DBeaver Over pgAdmin?

text

```
DBeaver works with EVERYTHING:
  PostgreSQL, MySQL, SQLite, MongoDB,
  Redis, Cassandra, Oracle, SQL Server...

If your team uses multiple databases,
DBeaver gives you ONE tool for all of them.

Free Community Edition is excellent.
```

### Installation

Bash

```
# Mac:
brew install --cask dbeaver-community

# Linux:
# Download from dbeaver.io/download/
# Or: snap install dbeaver-ce

# Windows:
# Download installer from dbeaver.io/download/
```

### Key DBeaver Features

text

```
1. DATABASE NAVIGATOR (left panel)
   - Browse all databases visually
   - See table structures, columns, types
   - View relationships (foreign keys)
   - Right-click → Edit Table, View Data

2. SQL EDITOR
   - Syntax highlighting for all databases
   - Auto-complete (table names, columns)
   - Multiple tabs for different queries
   - Save queries as files

3. ERD (Entity Relationship Diagram)
   - Right-click on schema → View Diagram
   - See ALL tables and their relationships visually
   - Export as image
   - This is how you understand existing database schemas!

4. DATA VIEWER
   - View table data in spreadsheet format
   - Edit data directly (for dev databases)
   - Filter rows visually
   - Sort by clicking column headers

5. EXPORT / IMPORT
   - Export: CSV, JSON, SQL, Excel
   - Import: CSV, JSON files
   - Database-to-database migration

6. ERD GENERATION
   - Auto-generates relationship diagrams
   - Critical for understanding legacy codebases
```

### Connecting Databases in DBeaver

text

```
PostgreSQL:
  1. Database → New Database Connection
  2. Select PostgreSQL
  3. Host: localhost, Port: 5432
  4. Database: learning_db
  5. Username/Password
  6. Test Connection → Finish

MongoDB:
  1. Database → New Database Connection
  2. Select MongoDB
  3. Host: localhost, Port: 27017
  4. Test Connection → Finish

Redis:
  1. Database → New Database Connection
  2. Select Redis
  3. Host: localhost, Port: 6379
  4. Test Connection → Finish
```

### DBeaver ERD — Understanding Existing Schemas

text

```
This is extremely valuable when joining a new project.

1. Right-click on schema "public"
2. Click "View Diagram"
3. DBeaver generates a visual ERD showing:
   - All tables as boxes
   - Columns listed inside boxes
   - Primary keys (PK) marked
   - Foreign key relationships as lines
   - Relationship types (1:N, M:N)

4. You can immediately understand the entire database structure
   without reading hundreds of lines of SQL.

Save it: Right-click → Save As Image
Share it: Put it in your project README.md
```

---

## 3. 🍃 MongoDB Compass

### What is MongoDB Compass?

text

```
MongoDB Compass = official GUI for MongoDB

Features:
  - Browse collections and documents visually
  - Build queries with a visual query builder
  - Build aggregation pipelines visually
  - Manage indexes
  - View query performance
  - Import/export data
  - Schema analysis (what fields exist and their types)
```

### Installation

Bash

```
# Mac:
brew install --cask mongodb-compass

# Linux/Windows:
# Download from mongodb.com/products/compass

# Or use mongosh (terminal):
brew install mongosh   # Mac
# Download from mongodb.com/try/download/shell for others
```

### Using MongoDB Compass

text

```
1. CONNECT
   Open Compass → New Connection
   URI: mongodb://localhost:27017
   Click Connect

2. BROWSE DATA
   Left panel: shows databases → collections
   Click a collection → see documents
   Documents shown as JSON (expandable)
   Filter documents:
     Click "Filter" field
     Enter: {"role": "admin"}
     Click Find

3. QUERY BUILDER
   Instead of typing JSON queries, use the visual builder:
   - Add filters with dropdowns
   - See generated query
   - Great for learning query syntax

4. AGGREGATION PIPELINE BUILDER
   Click "Aggregations" tab
   Add stages visually:
     Stage 1: $match → set filter
     Stage 2: $group → set group fields
     Stage 3: $sort → sort output
   See results after each stage
   Export pipeline as code

5. SCHEMA ANALYSIS
   Click "Schema" tab on a collection
   Shows:
     - All fields that exist
     - Type distribution (some docs have string, some int?)
     - Value distributions (most common values)
     - Missing field % (how many docs have this field?)
   Incredibly useful for understanding messy data

6. INDEX MANAGEMENT
   Click "Indexes" tab
   See all indexes on a collection
   Create new indexes
   Drop unused indexes
   See index usage statistics

7. EXPLAIN PLAN
   Run a query → click "Explain"
   See:
     - Whether index was used
     - Documents examined vs returned
     - Execution time
     - Full explain plan tree
```

---

## 4. 🔴 Redis CLI Deep Dive

Bash

```
# ─────────────────────────────────────────
# CONNECTING
# ─────────────────────────────────────────
redis-cli                              # local default
redis-cli -h hostname -p 6379         # remote host
redis-cli -u redis://user:pass@host   # URL format
redis-cli -n 2                        # connect to database 2 (0-15)

# ─────────────────────────────────────────
# MONITORING AND DEBUGGING
# ─────────────────────────────────────────

# Server information
INFO                    # everything
INFO server             # server details
INFO clients            # connected clients
INFO memory             # memory usage
INFO stats              # request statistics
INFO keyspace           # key counts per database
INFO replication        # replication status

# Monitor all commands in real-time
MONITOR                 # WARNING: very verbose, use briefly

# Slow log (commands that took too long)
SLOWLOG GET 25          # last 25 slow commands
SLOWLOG LEN             # how many in slow log
SLOWLOG RESET           # clear slow log
CONFIG SET slowlog-log-slower-than 10000  # log if > 10ms

# ─────────────────────────────────────────
# KEY MANAGEMENT
# ─────────────────────────────────────────

# Count all keys
DBSIZE                  # total keys in current database

# Find keys by pattern (use SCAN in production!)
KEYS "user:*"           # all keys starting with user:
KEYS "*session*"        # all keys containing session
# ⚠️  KEYS is dangerous in production (blocks server)

# Safe key scanning
SCAN 0 MATCH "user:*" COUNT 100    # scan 100 keys at a time
# Returns: cursor, [keys]
# When cursor = 0, scan is complete

# Key information
TYPE key                # string, hash, list, set, zset
OBJECT ENCODING key     # internal encoding
OBJECT IDLETIME key     # seconds since last access
OBJECT FREQ key         # access frequency (LFU)
DEBUG OBJECT key        # detailed object info

# Memory usage
MEMORY USAGE key        # bytes used by this key

# ─────────────────────────────────────────
# BATCH OPERATIONS
# ─────────────────────────────────────────

# Pipeline: send multiple commands at once
# Much faster than sending one by one
redis-cli --pipe << 'EOF'
SET key1 value1
SET key2 value2
SET key3 value3
EOF

# ─────────────────────────────────────────
# DEBUGGING
# ─────────────────────────────────────────

# Test if key exists and its TTL
EXISTS mykey            # 1=exists, 0=not exists
TTL mykey              # -1=no expire, -2=not exist, else seconds
PTTL mykey             # milliseconds remaining

# Debug memory issues
MEMORY DOCTOR           # Redis's memory recommendations
MEMORY STATS            # detailed memory breakdown
MEMORY PURGE            # free memory (jemalloc)

# ─────────────────────────────────────────
# USEFUL REDIS CLI FLAGS
# ─────────────────────────────────────────

# Run a single command and exit
redis-cli GET mykey

# Count keys matching a pattern
redis-cli --scan --pattern "session:*" | wc -l

# Delete all keys matching a pattern
redis-cli --scan --pattern "cache:*" | xargs redis-cli DEL

# Latency measurement
redis-cli --latency                    # continuous latency
redis-cli --latency-history -i 1      # latency every 1 second
redis-cli --latency-dist              # latency distribution

# Measure throughput
redis-cli --stat                      # requests/sec and more

# Benchmark
redis-cli debug sleep 0               # test connectivity
redis-benchmark -n 100000 -c 50      # benchmark 100k ops, 50 clients
```

---

## 5. 💡 RedisInsight — Redis GUI

### Installation and Setup

Bash

```
# Download from redis.com/redis-enterprise/redis-insight/
# Or run with Docker:
docker run -d -p 8001:8001 redis/redisinsight:latest
# Access at: http://localhost:8001

# Mac:
brew install --cask redisinsight

# Add connection:
# 1. Open RedisInsight (http://localhost:8001)
# 2. Click "Add Redis Database"
# 3. Host: localhost, Port: 6379
# 4. Name: Local Dev
# 5. Click "Add Redis Database"
```

### RedisInsight Features

text

```
1. BROWSER
   - Visual key browser
   - Filter by pattern, type, TTL
   - View/edit values for all data types
   - See memory usage per key
   - Delete keys visually

2. WORKBENCH (Query Editor)
   - Write Redis commands with syntax highlighting
   - Auto-complete
   - Command history
   - Run commands and see formatted results
   - Save command snippets

3. PROFILER
   - Real-time command log (like MONITOR but better)
   - Filter by command type
   - See which commands your app runs
   - Identify unexpected operations

4. ANALYSIS TOOLS
   - Memory Analysis: which keys use most memory?
   - SlowLog: show slow commands
   - Database Analysis: key type distribution, TTL distribution
   - Recommendations: Redis tells you how to optimize

5. CLUSTER MANAGER
   - View cluster topology
   - Monitor all nodes
   - See key distribution across shards

KEY WORKFLOW:
  1. Profiler to see what your app is doing
  2. Browser to inspect specific keys
  3. Analysis to find memory hogs
  4. Workbench to debug issues
```

---

## 6. 🔄 Database Migration Strategies

### Why Migration Strategy Matters

text

```
Database migrations in production are RISKY.

Wrong approach:
  1. Deploy new code that expects new schema
  2. Run migrations on live database
  3. 30 seconds of downtime → migration
  4. Users see errors during migration
  5. If migration fails → data corruption

Right approach depends on:
  - How large is the table?
  - Can the app handle both old and new schema?
  - How long does migration take?
  - What happens if it fails?

The key principle:
  Make schema changes BACKWARD COMPATIBLE
  Old code + new schema = still works
  New code + old schema = still works (during deployment)
```

### Migration Patterns

Python

```
# migration_strategies.py

"""
PATTERN 1: EXPAND → MIGRATE → CONTRACT
The safest strategy for zero-downtime deployments.

Example: Rename column 'username' to 'display_name'

Step 1: EXPAND (add new column)
  ALTER TABLE users ADD COLUMN display_name VARCHAR(100);
  Old code: uses username ← still works
  New code: uses display_name ← works

Step 2: MIGRATE (copy data)
  UPDATE users SET display_name = username;
  Backfill trigger for new rows.

Step 3: CONTRACT (remove old column)
  ALTER TABLE users DROP COLUMN username;
  After ALL instances of old code are gone.

Timeline:
  Deploy 1: adds display_name, copies data
  Deploy 2: uses display_name everywhere
  Deploy 3: removes username column
"""
```

SQL

```
-- ─────────────────────────────────────────
-- PATTERN 1: Adding a Column Safely
-- ─────────────────────────────────────────

-- SAFE: Adding a nullable column (no downtime)
ALTER TABLE users ADD COLUMN last_login TIMESTAMP WITH TIME ZONE;
-- Table briefly locked but operation is instant
-- Old code: ignores new column (fine)
-- New code: uses new column (fine)

-- SAFE: Adding column with default (instant in PostgreSQL 11+)
ALTER TABLE users ADD COLUMN login_count INTEGER DEFAULT 0;
-- PostgreSQL 11+: metadata-only change, instant!
-- PostgreSQL <11: rewrites entire table (downtime for large tables)

-- UNSAFE: Adding NOT NULL without default on large table
-- ALTER TABLE users ADD COLUMN required_field VARCHAR(100) NOT NULL;
-- This requires rewriting the entire table if it has data!

-- SAFE alternative for NOT NULL:
-- Step 1: Add as nullable
ALTER TABLE users ADD COLUMN required_field VARCHAR(100);
-- Step 2: Backfill existing rows
UPDATE users SET required_field = 'default_value' WHERE required_field IS NULL;
-- Step 3: Add NOT NULL constraint (validates existing data)
ALTER TABLE users ALTER COLUMN required_field SET NOT NULL;

-- ─────────────────────────────────────────
-- PATTERN 2: Renaming a Column Safely
-- ─────────────────────────────────────────

-- WRONG: Just rename (breaks old code immediately)
-- ALTER TABLE users RENAME COLUMN username TO display_name;

-- RIGHT: Expand → Migrate → Contract

-- Deploy 1: Add new column
ALTER TABLE users ADD COLUMN display_name VARCHAR(100);

-- Copy existing data
UPDATE users SET display_name = username;

-- Add trigger to keep both in sync during migration period
CREATE OR REPLACE FUNCTION sync_username_display_name()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' OR TG_OP = 'UPDATE' THEN
        IF NEW.username IS DISTINCT FROM OLD.username THEN
            NEW.display_name = NEW.username;
        END IF;
        IF NEW.display_name IS DISTINCT FROM OLD.display_name THEN
            NEW.username = NEW.display_name;
        END IF;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER sync_columns
BEFORE INSERT OR UPDATE ON users
FOR EACH ROW EXECUTE FUNCTION sync_username_display_name();

-- Deploy 2: Update all application code to use display_name
-- (all servers now use display_name)

-- Deploy 3: Remove old column and trigger
DROP TRIGGER sync_columns ON users;
DROP FUNCTION sync_username_display_name();
ALTER TABLE users DROP COLUMN username;

-- ─────────────────────────────────────────
-- PATTERN 3: Large Table Migrations
-- ─────────────────────────────────────────

-- Problem: UPDATE 10 million rows at once = hours of table lock

-- WRONG: Updating all rows at once
-- UPDATE posts SET view_count = 0 WHERE view_count IS NULL;
-- This locks the entire table!

-- RIGHT: Update in small batches
```

Python

```
# Large table migration in batches
def migrate_in_batches():
    """
    Update large table in small batches.
    This approach:
    - Keeps individual transactions small
    - Allows other queries to run between batches
    - Can be paused and resumed
    - Won't lock the table for extended periods
    """
    import psycopg2
    import time

    conn = psycopg2.connect(os.getenv("DATABASE_URL"))
    conn.autocommit = True
    cursor = conn.cursor()

    batch_size = 1000
    total_updated = 0
    last_id = 0

    print("Starting large table migration...")

    while True:
        # Update a batch of rows
        cursor.execute("""
            UPDATE posts
            SET view_count = 0
            WHERE id > %s
              AND id <= %s + %s
              AND view_count IS NULL
            RETURNING id
        """, (last_id, last_id, batch_size))

        updated_ids = cursor.fetchall()
        count = len(updated_ids)

        if count == 0:
            break  # No more rows to update

        total_updated += count
        last_id += batch_size

        print(f"Updated {total_updated} rows so far...")

        # Small pause between batches to reduce DB load
        time.sleep(0.01)  # 10ms

    cursor.close()
    conn.close()
    print(f"Migration complete. Total updated: {total_updated}")
```

SQL

```
-- ─────────────────────────────────────────
-- PATTERN 4: Creating Index Without Locking
-- ─────────────────────────────────────────

-- WRONG: Locks table during index creation (blocks all writes!)
-- CREATE INDEX idx_posts_created_at ON posts(created_at);

-- RIGHT: Create index concurrently
-- Takes longer but doesn't lock the table
CREATE INDEX CONCURRENTLY idx_posts_created_at
ON posts(created_at);

-- Note: CONCURRENTLY can't be run inside a transaction
-- Check progress:
SELECT
    phase,
    blocks_done,
    blocks_total,
    tuples_done,
    tuples_total
FROM pg_stat_progress_create_index
WHERE relid = 'posts'::regclass;

-- Drop index concurrently (also non-locking)
DROP INDEX CONCURRENTLY idx_posts_created_at;

-- ─────────────────────────────────────────
-- PATTERN 5: Table Partitioning
-- (for very large tables)
-- ─────────────────────────────────────────

-- When your table has billions of rows,
-- partition it by time or another key.

-- Create partitioned table
CREATE TABLE events (
    id          BIGSERIAL,
    user_id     INTEGER NOT NULL,
    event_type  VARCHAR(50) NOT NULL,
    payload     JSONB,
    created_at  TIMESTAMP WITH TIME ZONE NOT NULL
) PARTITION BY RANGE (created_at);

-- Create partitions (one per month)
CREATE TABLE events_2024_01
    PARTITION OF events
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE events_2024_02
    PARTITION OF events
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- PostgreSQL automatically routes queries to right partition
-- Drop old partitions to free space (instant!)
DROP TABLE events_2024_01;
-- Much faster than deleting millions of rows!

-- ─────────────────────────────────────────
-- PATTERN 6: Zero-Downtime Table Rename
-- ─────────────────────────────────────────

-- Rename with view for backward compatibility

-- 1. Create new table with correct name
CREATE TABLE user_profiles (LIKE users INCLUDING ALL);

-- 2. Copy data
INSERT INTO user_profiles SELECT * FROM users;

-- 3. Create view with old name
CREATE VIEW users_old AS SELECT * FROM user_profiles;

-- 4. Update application to use new name
-- 5. Drop old view
DROP VIEW users_old;
DROP TABLE users;

-- ─────────────────────────────────────────
-- CHECKING MIGRATION PROGRESS
-- ─────────────────────────────────────────

-- Check long-running queries
SELECT
    pid,
    now() - query_start AS duration,
    state,
    query
FROM pg_stat_activity
WHERE state != 'idle'
  AND query_start IS NOT NULL
ORDER BY duration DESC;

-- Check locks
SELECT
    pid,
    relation::regclass AS table_name,
    mode,
    granted
FROM pg_locks
WHERE relation IS NOT NULL;

-- Check table sizes
SELECT
    relname AS table_name,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size,
    pg_size_pretty(pg_relation_size(relid)) AS data_size,
    pg_size_pretty(
        pg_total_relation_size(relid) - pg_relation_size(relid)
    ) AS index_size
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Check index bloat (after many updates/deletes)
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Production Migration Checklist

Python

```
# migration_checklist.py
"""
Before running any migration in production:

1. BACKUP FIRST
   pg_dump -U postgres -d mydb -Fc -f backup_$(date +%Y%m%d_%H%M%S).dump
   Test the backup: pg_restore --list backup.dump

2. TEST IN STAGING
   Run migration on staging database first.
   Verify application works with new schema.

3. ESTIMATE DURATION
   For large table operations, test on a copy:
   CREATE TABLE users_copy AS SELECT * FROM users;
   -- Time the operation on the copy
   SELECT count(*) FROM users_copy;  -- know the size

4. CHECK FOR LOCKS
   Will this migration lock tables?
   For how long?
   During what traffic period?

5. ROLLBACK PLAN
   What's your downgrade migration?
   Can it be done quickly?
   Have you tested the downgrade?

6. MONITOR DURING MIGRATION
   Watch pg_stat_activity for blocked queries
   Watch application error rates
   Watch database CPU and I/O

7. DEPLOYMENT ORDER
   Deploy in the right order:
     a. Run schema migration (additive changes)
     b. Deploy new application code
     c. Run data migration (if needed)
     d. Remove old columns (after all code deployed)

8. FEATURE FLAGS
   Use feature flags to gradually roll out
   code that uses new schema.
   Rollback by toggling the flag.
"""

class MigrationRunner:
    """
    Safe migration runner with built-in safeguards.
    """

    def __init__(self, db_url: str, dry_run: bool = True):
        self.db_url = db_url
        self.dry_run = dry_run  # default to dry run for safety

    def run_migration(
        self,
        name: str,
        sql: str,
        batch_size: int = None
    ) -> dict:
        """
        Run a migration with safety checks.

        Returns:
            dict with status, duration, rows_affected
        """
        import time
        import psycopg2

        print(f"\n{'='*50}")
        print(f"Migration: {name}")
        print(f"Mode: {'DRY RUN' if self.dry_run else 'LIVE'}")
        print(f"{'='*50}")
        print(f"SQL:\n{sql}\n")

        if self.dry_run:
            print("DRY RUN: No changes made. Pass dry_run=False to execute.")
            return {"status": "dry_run", "sql": sql}

        # Confirm before running live
        confirm = input("Type 'yes' to proceed: ")
        if confirm.lower() != "yes":
            return {"status": "cancelled"}

        conn = psycopg2.connect(self.db_url)
        start = time.perf_counter()

        try:
            with conn:
                with conn.cursor() as cur:
                    cur.execute(sql)
                    rows = cur.rowcount

            duration = time.perf_counter() - start
            print(f"✅ Success: {rows} rows affected in {duration:.2f}s")

            return {
                "status": "success",
                "rows_affected": rows,
                "duration_seconds": duration
            }

        except Exception as e:
            duration = time.perf_counter() - start
            print(f"❌ Failed after {duration:.2f}s: {e}")
            return {
                "status": "failed",
                "error": str(e),
                "duration_seconds": duration
            }
        finally:
            conn.close()

    def check_table_size(self, table: str) -> dict:
        """Check table size before running migrations on it."""
        import psycopg2

        conn = psycopg2.connect(self.db_url)
        with conn.cursor() as cur:
            cur.execute("""
                SELECT
                    pg_size_pretty(pg_total_relation_size(%s)) AS total,
                    pg_size_pretty(pg_relation_size(%s)) AS data,
                    (SELECT COUNT(*) FROM """ + table + """) AS row_count
            """, (table, table))
            row = cur.fetchone()
        conn.close()

        return {
            "table": table,
            "total_size": row[0],
            "data_size": row[1],
            "row_count": row[2]
        }

    def check_active_connections(self) -> list:
        """Check for active connections that might be affected."""
        import psycopg2

        conn = psycopg2.connect(self.db_url)
        with conn.cursor() as cur:
            cur.execute("""
                SELECT pid, usename, application_name, state, query
                FROM pg_stat_activity
                WHERE state != 'idle'
                AND pid != pg_backend_pid()
                ORDER BY query_start
            """)
            connections = cur.fetchall()
        conn.close()

        return [
            {
                "pid": row[0],
                "user": row[1],
                "app": row[2],
                "state": row[3],
                "query": row[4][:100] if row[4] else None
            }
            for row in connections
        ]
```

---

## 7. 🎯 Putting It All Together — Complete Workflow

Python

```
# workflow_example.py
"""
Real production workflow:

1. Feature development:
   - Use DBeaver to explore schema
   - pgAdmin query tool for complex queries
   - RedisInsight to check cache state

2. Schema changes:
   - Write Alembic migration
   - Test in local development
   - Review generated SQL
   - Test in staging
   - Run in production with checklist

3. Performance issues:
   - pgAdmin dashboard to spot load spikes
   - EXPLAIN ANALYZE in pgAdmin query tool
   - Create index CONCURRENTLY
   - Use Redis for hot data
   - MongoDB Compass to check slow operations

4. Debugging:
   - DBeaver to browse data visually
   - Redis CLI to check cache values
   - MongoDB Compass to inspect documents
"""

import os
from dotenv import load_dotenv

load_dotenv()

# Example: Complete database initialization for a project
def initialize_database():
    """
    Typical database initialization sequence
    for a new backend project.
    """

    print("1. Running PostgreSQL migrations...")
    # alembic upgrade head

    print("2. Setting up Redis indexes...")
    import redis
    r = redis.from_url(os.getenv("REDIS_URL", "redis://localhost:6379"))

    # Verify Redis connection
    r.ping()
    print("   Redis: connected")

    # Set application metadata
    r.hset("app:info", mapping={
        "version": "1.0.0",
        "started_at": "2024-01-15",
        "environment": os.getenv("ENVIRONMENT", "development")
    })

    print("3. Setting up MongoDB indexes...")
    from pymongo import MongoClient, ASCENDING, DESCENDING
    mongo = MongoClient(os.getenv("MONGODB_URL", "mongodb://localhost:27017"))
    db = mongo[os.getenv("MONGODB_DB", "myapp")]

    # Users collection
    db.users.create_index("email", unique=True)
    db.users.create_index([("created_at", DESCENDING)])

    # Events collection with TTL
    db.events.create_index(
        "expires_at",
        expireAfterSeconds=0
    )
    db.events.create_index([("user_id", ASCENDING), ("created_at", DESCENDING)])

    print("   MongoDB: indexes created")
    mongo.close()

    print("\n✅ All databases initialized successfully!")


initialize_database()
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                    DATABASE TOOLS                              │
│                                                                │
│  POSTGRESQL TOOLS                                              │
│  ─────────────────                                             │
│  pgAdmin     → Official GUI, visual explain plans,           │
│                performance dashboard, query tool              │
│  DBeaver     → Universal GUI, ERD diagrams, all databases,   │
│                data export/import                             │
│  psql        → Terminal CLI, fastest for quick operations    │
│                                                                │
│  MONGODB TOOLS                                                 │
│  ──────────────                                                │
│  Compass     → Official GUI, visual query builder,           │
│                aggregation builder, schema analysis           │
│  mongosh     → Terminal shell, scripting                     │
│                                                                │
│  REDIS TOOLS                                                   │
│  ─────────────                                                 │
│  redis-cli   → Terminal CLI, monitoring, benchmark           │
│  RedisInsight → GUI, profiler, memory analysis,              │
│                 key browser                                    │
│                                                                │
│  MIGRATION STRATEGIES                                          │
│  ─────────────────────                                         │
│  Expand → Migrate → Contract  (zero-downtime column rename)  │
│  Batch updates               (large table updates)           │
│  CREATE INDEX CONCURRENTLY   (non-locking index creation)    │
│  Table partitioning          (very large tables)             │
│                                                                │
│  PRODUCTION CHECKLIST                                          │
│  ──────────────────────                                        │
│  1. Backup first                                              │
│  2. Test in staging                                           │
│  3. Estimate duration                                         │
│  4. Check for locks                                           │
│  5. Have rollback plan                                        │
│  6. Monitor during migration                                  │
│  7. Correct deployment order                                  │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is the main difference between pgAdmin and DBeaver?
    When would you choose one over the other?
2.  How do you view a visual query execution plan in pgAdmin?
    What are you looking for?
3.  What is the ERD feature in DBeaver useful for?
4.  What does the Schema Analysis feature in MongoDB Compass show?
5.  Why is KEYS dangerous in production Redis?
    What should you use instead?
6.  What is the Expand → Migrate → Contract pattern?
    Give a step-by-step example.
7.  Why is adding a NOT NULL column to a large table risky?
    How do you do it safely?
8.  What does CREATE INDEX CONCURRENTLY do differently?
    What is the trade-off?
9.  Why should you update large tables in batches?
10. What should you always do before running a migration in prod?
11. What does pg_stat_activity show you?
12. What is table partitioning and when would you use it?
```

---

## 🛠️ Practice Exercises

text

```
Exercise 1: pgAdmin / DBeaver Setup
  1. Install either pgAdmin or DBeaver
  2. Connect to your learning_db PostgreSQL database
  3. Browse the users and posts tables
  4. Use the query tool to run:
     - A JOIN query between users and posts
     - An aggregate query (posts per user)
  5. Click "Explain" on one query
  6. Identify: is it doing a Seq Scan or Index Scan?
  7. If Seq Scan on a large-ish result, create an index
  8. Re-run Explain and compare

Exercise 2: DBeaver ERD
  1. Install DBeaver
  2. Connect to learning_db
  3. Right-click "public" schema → View Diagram
  4. Export as image
  5. Share it or save to your project README

Exercise 3: MongoDB Compass
  1. Install MongoDB Compass
  2. Connect to localhost:27017
  3. Browse your collections
  4. Use the Aggregation builder to create:
     - Group by user role, count documents
     - Sort by count descending
  5. Export the aggregation pipeline code

Exercise 4: Redis CLI Monitoring
  1. Open redis-cli
  2. Run: INFO memory
     What is used_memory_human?
  3. Run: SLOWLOG GET 5
     Are there any slow commands?
  4. Run: redis-cli --stat
     Watch requests/second while your app runs
  5. Find all your session keys:
     SCAN 0 MATCH "session:*" COUNT 100

Exercise 5: Safe Migration
  Write a migration that does ALL of the following SAFELY:
  (No downtime, no full table locks on large tables)
  
  a. Add a 'last_login' TIMESTAMP column to users table
  b. Add a 'login_count' INTEGER column (default 0)
  c. Add an index on 'last_login'
  d. Backfill 'login_count' based on a hypothetical
     login_events table (do it in batches of 500)
  e. Make 'login_count' NOT NULL after backfill
  
  Write both the SQL and the Alembic migration file.
  Include the rollback (downgrade) steps.
```

---

## ✅ Phase 3.4 Complete!

**You now know:**

text

```
✅ pgAdmin — connecting, query tool, visual explain, dashboard
✅ DBeaver — universal GUI, ERD generation, data browsing
✅ MongoDB Compass — document browser, aggregation builder,
                     schema analysis, explain plans
✅ Redis CLI — monitoring commands, SCAN vs KEYS,
               slow log, memory analysis
✅ RedisInsight — profiler, key browser, memory analysis
✅ Expand → Migrate → Contract pattern
✅ Safe column additions (nullable first, then NOT NULL)
✅ Batch updates for large tables
✅ CREATE INDEX CONCURRENTLY (non-blocking)
✅ Table partitioning concepts
✅ Production migration checklist
✅ Monitoring active connections and locks
```