```
Phase 2.1 → You know the data structures
Phase 2.2 → You know the algorithms
Phase 2.3 → You know how to write EFFICIENT Python specifically

This phase answers:
  "I know the right algorithm, but which Python tool do I use?"
  "Is list faster than tuple for this?"
  "When does dict beat set?"
  "How do I avoid the hidden O(n) operations?"
  "What do senior engineers know that juniors don't?"

These are the details that separate:
  Code that works → Code that works FAST
  Junior engineer → Mid/Senior engineer
```

---

## Setup

```bash
cd ~/projects/data_structures
touch efficiency.py
```

---

## 1. 📊 When to Use list vs tuple vs set vs dict

### The Decision Framework

```python
# efficiency.py

import sys
import time
from typing import Any

# ─────────────────────────────────────────
# THE CORE QUESTION TO ASK YOURSELF:
# ─────────────────────────────────────────
"""
Do I need to CHANGE the collection after creation?
  NO  → tuple (immutable, memory efficient)
  YES → continue...

Do I need to FIND elements by index/position?
  YES → list
  NO  → continue...

Do I need UNIQUE elements and/or FAST membership test?
  YES → set
  NO  → continue...

Do I need KEY → VALUE mapping?
  YES → dict
  NO  → list
"""
```

### Memory Comparison

```python
# ─────────────────────────────────────────
# MEMORY USAGE
# ─────────────────────────────────────────
data = range(1000)

list_data = list(data)
tuple_data = tuple(data)
set_data = set(data)
dict_data = {i: i for i in data}

print("Memory usage for 1000 integers:")
print(f"  list:  {sys.getsizeof(list_data):>8,} bytes")
print(f"  tuple: {sys.getsizeof(tuple_data):>8,} bytes")
print(f"  set:   {sys.getsizeof(set_data):>8,} bytes")
print(f"  dict:  {sys.getsizeof(dict_data):>8,} bytes")

# Typical output:
# list:     8,056 bytes
# tuple:    8,040 bytes  ← slightly smaller than list
# set:     32,984 bytes  ← much larger (hash table overhead)
# dict:    36,952 bytes  ← largest (key + value + hash table)

# KEY INSIGHT:
# list ≈ tuple for memory (tuple slightly smaller, no capacity buffer)
# set and dict use MUCH more memory (they're hash tables)
# You pay a memory cost for O(1) lookup
```

### Speed Comparison

```python
import timeit

n = 100_000
data_list = list(range(n))
data_tuple = tuple(range(n))
data_set = set(range(n))
data_dict = {i: True for i in range(n)}

target = n - 1  # worst case: last element

def measure(stmt: str, setup: str, number: int = 1000) -> float:
    return timeit.timeit(stmt, setup=setup, number=number) * 1000 / number

# ─────────────────────────────────────────
# MEMBERSHIP TEST: x in collection
# ─────────────────────────────────────────
print("=== Membership Test (x in collection) ===")

list_time = measure(
    f"{target} in data_list",
    "data_list = list(range(100_000))"
)
tuple_time = measure(
    f"{target} in data_tuple",
    "data_tuple = tuple(range(100_000))"
)
set_time = measure(
    f"{target} in data_set",
    "data_set = set(range(100_000))"
)
dict_time = measure(
    f"{target} in data_dict",
    "data_dict = {i: True for i in range(100_000)}"
)

print(f"  list:  {list_time:.4f}ms  O(n)")
print(f"  tuple: {tuple_time:.4f}ms  O(n)")
print(f"  set:   {set_time:.6f}ms  O(1) ← winner!")
print(f"  dict:  {dict_time:.6f}ms  O(1) ← winner!")

# Typical output:
# list:  0.8500ms  O(n)
# tuple: 0.8300ms  O(n)
# set:   0.0001ms  O(1) ← ~8500x faster!
# dict:  0.0001ms  O(1) ← ~8500x faster!

# ─────────────────────────────────────────
# ITERATION: for x in collection
# ─────────────────────────────────────────
print("\n=== Iteration (for x in collection) ===")

for name, col in [
    ("list ", data_list),
    ("tuple", data_tuple),
    ("set  ", data_set),
]:
    start = time.perf_counter()
    for _ in range(10):
        for item in col:
            pass
    elapsed = (time.perf_counter() - start) * 1000 / 10
    print(f"  {name}: {elapsed:.3f}ms")

# list and tuple: similar speed
# set: slightly slower (non-contiguous memory)

# ─────────────────────────────────────────
# INDEXED ACCESS: collection[index]
# ─────────────────────────────────────────
print("\n=== Indexed Access (collection[i]) ===")
# Only list and tuple support indexing
# set and dict do NOT support indexing by position

list_time = timeit.timeit(
    "data_list[50000]",
    setup="data_list = list(range(100_000))",
    number=1_000_000
) * 1000 / 1_000_000

tuple_time = timeit.timeit(
    "data_tuple[50000]",
    setup="data_tuple = tuple(range(100_000))",
    number=1_000_000
) * 1000 / 1_000_000

print(f"  list[i]:  {list_time:.6f}ms  O(1)")
print(f"  tuple[i]: {tuple_time:.6f}ms  O(1)")
# Nearly identical — both O(1) direct memory access

# ─────────────────────────────────────────
# APPEND / ADD
# ─────────────────────────────────────────
print("\n=== Adding Elements ===")

list_time = timeit.timeit(
    "data.append(1)",
    setup="data = list(range(100))",
    number=100_000
) * 1000 / 100_000

set_time = timeit.timeit(
    "data.add(1)",
    setup="data = set(range(100))",
    number=100_000
) * 1000 / 100_000

dict_time = timeit.timeit(
    "data[1] = True",
    setup="data = {i: True for i in range(100)}",
    number=100_000
) * 1000 / 100_000

print(f"  list.append(): {list_time:.6f}ms  O(1) amortized")
print(f"  set.add():     {set_time:.6f}ms   O(1) amortized")
print(f"  dict[k]=v:     {dict_time:.6f}ms  O(1) amortized")
```

### Real-World Decision Examples

```python
# ─────────────────────────────────────────
# EXAMPLE 1: Email validation
# ─────────────────────────────────────────

# BAD: checking against a list
INVALID_DOMAINS_LIST = [
    "tempmail.com", "throwaway.com", "fakeemail.com",
    # ... imagine 10,000 blocked domains
]

def is_valid_email_slow(email: str) -> bool:
    domain = email.split("@")[-1]
    return domain not in INVALID_DOMAINS_LIST  # O(n) every time!

# GOOD: checking against a set
INVALID_DOMAINS_SET = {
    "tempmail.com", "throwaway.com", "fakeemail.com",
    # ... same 10,000 domains
}

def is_valid_email_fast(email: str) -> bool:
    domain = email.split("@")[-1]
    return domain not in INVALID_DOMAINS_SET  # O(1) every time!

# ─────────────────────────────────────────
# EXAMPLE 2: User lookup
# ─────────────────────────────────────────

# BAD: list of user dicts — O(n) search
users_list = [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"},
    # ... 1 million users
]

def get_user_slow(user_id: int) -> dict:
    for user in users_list:     # O(n) — check every user!
        if user["id"] == user_id:
            return user
    return None

# GOOD: dict keyed by id — O(1) lookup
users_dict = {
    1: {"id": 1, "name": "Alice"},
    2: {"id": 2, "name": "Bob"},
    # ... same 1 million users
}

def get_user_fast(user_id: int) -> dict:
    return users_dict.get(user_id)  # O(1) — direct hash lookup!

# ─────────────────────────────────────────
# EXAMPLE 3: Configuration values (constants)
# ─────────────────────────────────────────

# BAD: list — allows duplicates, no semantic meaning
CONFIG_VALUES_BAD = ["host", "port", "debug", "secret_key"]

# GOOD: tuple — immutable constants, slightly less memory
CONFIG_KEYS = ("host", "port", "debug", "secret_key")

# BETTER for membership: frozenset (immutable set)
REQUIRED_FIELDS = frozenset({"host", "port", "secret_key"})

def validate_config(config: dict) -> bool:
    return REQUIRED_FIELDS.issubset(config.keys())  # O(|REQUIRED_FIELDS|)

# ─────────────────────────────────────────
# EXAMPLE 4: Unique tags
# ─────────────────────────────────────────

# BAD: list — allows duplicates, O(n) dedup
def add_tag_slow(tags: list, new_tag: str) -> list:
    if new_tag not in tags:     # O(n) membership check
        tags.append(new_tag)
    return tags

# GOOD: set — automatic dedup, O(1) operations
def add_tag_fast(tags: set, new_tag: str) -> set:
    tags.add(new_tag)           # O(1) — set handles dedup automatically
    return tags
```

---

## 2. ⏱️ Time Complexity of Python Built-ins

```python
# ─────────────────────────────────────────
# LIST OPERATIONS
# ─────────────────────────────────────────
"""
Operation                   Complexity    Notes
──────────────────────────────────────────────────
lst[i]                      O(1)         index access
lst[i] = v                  O(1)         index assignment
len(lst)                    O(1)         stored separately
lst.append(v)               O(1)*        amortized
lst.pop()                   O(1)         remove last
lst.pop(0)                  O(n)         ← SLOW! shifts all elements
lst.insert(0, v)            O(n)         ← SLOW! shifts all elements
lst.insert(i, v)            O(n)         shifts n-i elements
v in lst                    O(n)         ← SLOW for large lists!
lst.remove(v)               O(n)         search + shift
lst.index(v)                O(n)
lst.count(v)                O(n)
lst.reverse()               O(n)
lst.sort()                  O(n log n)
lst + lst2                  O(n+m)       creates new list
lst * n                     O(n*k)
lst[i:j]                    O(k)         k = slice size
min(lst) / max(lst)         O(n)
sum(lst)                    O(n)
sorted(lst)                 O(n log n)   creates new list
list(iterable)              O(n)
"""

# ─────────────────────────────────────────
# COMMON MISTAKE: list.pop(0) in a loop
# ─────────────────────────────────────────

# BAD: O(n²) — pop(0) is O(n), done n times
def process_queue_bad(items: list) -> None:
    while items:
        item = items.pop(0)     # O(n) each time!
        # process item

# GOOD: use deque — O(n) total
from collections import deque

def process_queue_good(items: list) -> None:
    queue = deque(items)
    while queue:
        item = queue.popleft()  # O(1) each time!
        # process item

# Demonstration
n = 100_000

start = time.perf_counter()
items = list(range(n))
while items:
    items.pop(0)
slow_time = (time.perf_counter() - start) * 1000

start = time.perf_counter()
items = deque(range(n))
while items:
    items.popleft()
fast_time = (time.perf_counter() - start) * 1000

print(f"list.pop(0) loop: {slow_time:.1f}ms  O(n²)")
print(f"deque.popleft():  {fast_time:.1f}ms  O(n)")
# list.pop(0) loop: ~2500ms  (for n=100k)
# deque.popleft():     ~8ms  (for n=100k)

# ─────────────────────────────────────────
# DICT OPERATIONS
# ─────────────────────────────────────────
"""
Operation                   Complexity    Notes
──────────────────────────────────────────────────
d[key]                      O(1)*        average
d[key] = v                  O(1)*        average
del d[key]                  O(1)*        average
key in d                    O(1)*        average
d.get(key)                  O(1)*        average
d.keys()                    O(1)         returns view
d.values()                  O(1)         returns view
d.items()                   O(1)         returns view
len(d)                      O(1)
d.update(other)             O(len(other))
d.copy()                    O(n)
dict(d)                     O(n)
{**d1, **d2}                O(n+m)

* = O(n) worst case with hash collisions (rare)
"""

# ─────────────────────────────────────────
# SET OPERATIONS
# ─────────────────────────────────────────
"""
Operation                   Complexity    Notes
──────────────────────────────────────────────────
v in s                      O(1)*        average
s.add(v)                    O(1)*        average
s.remove(v)                 O(1)*        raises KeyError if missing
s.discard(v)                O(1)*        no error if missing
s.pop()                     O(1)         removes arbitrary element
len(s)                      O(1)
s | t  (union)              O(len(s)+len(t))
s & t  (intersection)       O(min(len(s),len(t)))
s - t  (difference)         O(len(s))
s ^ t  (symmetric diff)     O(len(s)+len(t))
s.issubset(t)               O(len(s))
s.issuperset(t)             O(len(t))
s == t                      O(min(len(s),len(t)))
"""

# ─────────────────────────────────────────
# STRING OPERATIONS
# ─────────────────────────────────────────
"""
Operation                   Complexity    Notes
──────────────────────────────────────────────────
s[i]                        O(1)
len(s)                      O(1)
s + t                       O(len(s)+len(t))  creates NEW string
s * n                       O(n*len(s))
v in s                      O(n)         substring search
s.split()                   O(n)
s.join(iterable)            O(n)         where n = total chars
s.replace(old, new)         O(n)
s.find(sub)                 O(n)
s.startswith(prefix)        O(len(prefix))
s.strip()                   O(n)
s.upper() / s.lower()       O(n)
"""

# ─────────────────────────────────────────
# STRING CONCATENATION TRAP
# ─────────────────────────────────────────
# BAD: O(n²) — creates new string each iteration
def build_string_slow(words: list) -> str:
    result = ""
    for word in words:
        result += word + ", "   # creates new string each time!
    return result

# GOOD: O(n) — join all at once
def build_string_fast(words: list) -> str:
    return ", ".join(words)

# GOOD: collect then join
def build_string_fast2(words: list) -> str:
    parts = []
    for word in words:
        parts.append(word)
    return ", ".join(parts)

n = 10_000
words = ["word"] * n

start = time.perf_counter()
build_string_slow(words)
slow = (time.perf_counter() - start) * 1000

start = time.perf_counter()
build_string_fast(words)
fast = (time.perf_counter() - start) * 1000

print(f"\nString building ({n} words):")
print(f"  += concatenation: {slow:.2f}ms")
print(f"  join():           {fast:.2f}ms")
print(f"  join is {slow/fast:.0f}x faster")
```

---

## 3. 📚 Collections Module for Performance

```python
from collections import (
    Counter, defaultdict, OrderedDict,
    namedtuple, deque, ChainMap
)

# ─────────────────────────────────────────
# COUNTER — counting made fast
# ─────────────────────────────────────────

# BAD: manual counting
def count_words_slow(text: str) -> dict:
    counts = {}
    for word in text.split():
        if word in counts:
            counts[word] += 1
        else:
            counts[word] = 0
    return counts

# GOOD: Counter
def count_words_fast(text: str) -> Counter:
    return Counter(text.split())    # one line, same performance

text = "the quick brown fox jumps over the lazy dog the fox"
print(Counter(text.split()).most_common(3))
# [('the', 3), ('fox', 2), ('quick', 1)]

# Counter arithmetic — very useful
week1_errors = Counter({"404": 45, "500": 12, "403": 8})
week2_errors = Counter({"404": 30, "500": 18, "400": 5})

combined = week1_errors + week2_errors
print("Combined:", dict(combined))
# Combined: {'404': 75, '500': 30, '403': 8, '400': 5}

increase = week2_errors - week1_errors
print("Increase:", dict(increase))
# Only shows POSITIVE differences
# {'500': 6, '400': 5}

# ─────────────────────────────────────────
# defaultdict — avoid key existence checks
# ─────────────────────────────────────────

# BAD: manual check
def group_by_role_slow(users: list) -> dict:
    result = {}
    for user in users:
        role = user["role"]
        if role not in result:  # O(1) but verbose
            result[role] = []
        result[role].append(user["name"])
    return result

# GOOD: defaultdict
def group_by_role_fast(users: list) -> dict:
    result = defaultdict(list)
    for user in users:
        result[user["role"]].append(user["name"])
    return dict(result)

users = [
    {"name": "Alice", "role": "admin"},
    {"name": "Bob", "role": "user"},
    {"name": "Charlie", "role": "admin"},
    {"name": "Dave", "role": "user"},
]
print(group_by_role_fast(users))
# {'admin': ['Alice', 'Charlie'], 'user': ['Bob', 'Dave']}

# defaultdict with complex default factories
def make_user_stats():
    return {"total": 0, "errors": 0, "latency_sum": 0}

user_stats = defaultdict(make_user_stats)
user_stats["alice"]["total"] += 1
user_stats["alice"]["latency_sum"] += 45
user_stats["bob"]["total"] += 1    # bob auto-initialized!
print(dict(user_stats))

# ─────────────────────────────────────────
# deque — O(1) at both ends
# ─────────────────────────────────────────

# Key operations and when to use deque vs list:
"""
list.append()   → O(1)  ← same as deque
list.pop()      → O(1)  ← same as deque
list.insert(0)  → O(n)  ← deque.appendleft() is O(1)!
list.pop(0)     → O(n)  ← deque.popleft() is O(1)!

Use deque when:
  - You need to add/remove from BOTH ends
  - Implementing queues, BFS
  - Sliding window with removal from left
  - LRU cache implementation

Use list when:
  - You need index access: list[5] (deque is O(n)!)
  - Mostly appending to end and reading
"""

# deque maxlen — automatic size limiting
from collections import deque

# Last N events (sliding log window)
recent_errors = deque(maxlen=5)
for error in ["E1", "E2", "E3", "E4", "E5", "E6", "E7"]:
    recent_errors.append(error)
    print(f"  Added {error}: {list(recent_errors)}")
# Automatically removes oldest when maxlen exceeded

# ─────────────────────────────────────────
# namedtuple — readable, memory-efficient
# ─────────────────────────────────────────

# Instead of:
user_as_dict = {
    "id": 1,
    "name": "Alice",
    "email": "alice@test.com",
    "role": "admin"
}
# Memory: ~232 bytes for dict

# Use namedtuple for read-only data
User = namedtuple("User", ["id", "name", "email", "role"])
user_as_tuple = User(1, "Alice", "alice@test.com", "admin")
# Memory: ~72 bytes — 3x more efficient!

# Access by name (readable)
print(user_as_tuple.name)   # Alice
print(user_as_tuple.role)   # admin

# Access by index (fast)
print(user_as_tuple[0])     # 1

# Works as dict key (hashable!)
user_access_log = {user_as_tuple: ["login", "view_profile"]}

# Convert to dict when needed
print(user_as_tuple._asdict())
# OrderedDict([('id', 1), ('name', 'Alice'), ...])

# Memory comparison
import sys
print(f"\nDict user:     {sys.getsizeof(user_as_dict)} bytes")
print(f"Namedtuple:    {sys.getsizeof(user_as_tuple)} bytes")

# ─────────────────────────────────────────
# ChainMap — multiple dicts as one view
# ─────────────────────────────────────────

# Default config
defaults = {"debug": False, "host": "localhost", "port": 8000}

# Environment-specific overrides
production = {"debug": False, "port": 80}

# User overrides
user_config = {"debug": True}

# ChainMap: searches dicts in order
# First dict takes priority
config = ChainMap(user_config, production, defaults)

print(config["debug"])  # True   (from user_config)
print(config["port"])   # 80     (from production)
print(config["host"])   # localhost (from defaults)

# Great for layered config systems
# Changes to ChainMap go to FIRST dict
config["timeout"] = 30
print(user_config)  # {"debug": True, "timeout": 30}
```

---

## 4. 🔢 Bisect Module

```python
import bisect

"""
bisect module: binary search on sorted lists.
All operations: O(log n) for search, O(n) for insertion.

Use when:
  - Maintaining a sorted list with insertions
  - Finding insertion point for a value
  - Range queries on sorted data
  - Simpler than implementing BST for many use cases
"""

# ─────────────────────────────────────────
# BASIC OPERATIONS
# ─────────────────────────────────────────
scores = [10, 20, 30, 40, 50, 60, 70, 80, 90]

# bisect_left: insertion point before existing equal elements
print(bisect.bisect_left(scores, 50))   # 4  (index where 50 is)
print(bisect.bisect_left(scores, 45))   # 4  (where 45 would go)
print(bisect.bisect_left(scores, 5))    # 0  (before everything)
print(bisect.bisect_left(scores, 100))  # 9  (after everything)

# bisect_right: insertion point AFTER existing equal elements
print(bisect.bisect_right(scores, 50))  # 5  (after the 50)
print(bisect.bisect_right(scores, 45))  # 4  (same as bisect_left for missing)

# insort: insert maintaining sort order — O(n)
bisect.insort(scores, 45)
print(scores)   # [10, 20, 30, 40, 45, 50, 60, 70, 80, 90]

bisect.insort_left(scores, 45)  # insert BEFORE existing 45
print(scores)   # [10, 20, 30, 40, 45, 45, 50, 60, 70, 80, 90]

# ─────────────────────────────────────────
# REAL USE 1: Grade calculator
# ─────────────────────────────────────────
def get_grade(score: int) -> str:
    """
    Convert score to letter grade using bisect.
    O(log n) — faster than if/elif chain for many breakpoints.
    """
    breakpoints = [60, 70, 80, 90, 100]
    grades = ["F", "D", "C", "B", "A"]
    position = bisect.bisect_left(breakpoints, score)
    return grades[position]

for score in [45, 65, 75, 85, 95, 100]:
    print(f"  Score {score}: {get_grade(score)}")

# ─────────────────────────────────────────
# REAL USE 2: Rate tier pricing
# ─────────────────────────────────────────
def get_api_price_tier(requests_per_month: int) -> dict:
    """
    Get pricing tier based on usage.
    O(log n) — perfect for tiered pricing lookup.
    """
    tiers = [
        (1_000, {"name": "Free", "price": 0}),
        (10_000, {"name": "Starter", "price": 29}),
        (100_000, {"name": "Growth", "price": 99}),
        (1_000_000, {"name": "Business", "price": 299}),
        (float('inf'), {"name": "Enterprise", "price": 999}),
    ]

    limits = [t[0] for t in tiers]
    position = bisect.bisect_left(limits, requests_per_month)

    if position >= len(tiers):
        return tiers[-1][1]
    return tiers[position][1]

for usage in [500, 5_000, 50_000, 500_000, 5_000_000]:
    tier = get_api_price_tier(usage)
    print(f"  {usage:>10,} requests → {tier['name']} (${tier['price']}/mo)")

# ─────────────────────────────────────────
# REAL USE 3: Event log range query
# ─────────────────────────────────────────
from datetime import datetime

def range_query(
    sorted_events: list,
    start_time: int,
    end_time: int
) -> list:
    """
    Find all events in a time range.
    O(log n) to find bounds, O(k) to extract.
    """
    # Find insertion point for start_time
    left = bisect.bisect_left(sorted_events, start_time)
    # Find insertion point for end_time
    right = bisect.bisect_right(sorted_events, end_time)

    return sorted_events[left:right]

timestamps = [1000, 1100, 1200, 1300, 1400, 1500, 1600, 1700, 1800]
result = range_query(timestamps, 1200, 1600)
print(f"Events 1200-1600: {result}")  # [1200, 1300, 1400, 1500, 1600]
```

---

## 5. ⛰️ heapq Module

```python
import heapq

"""
heapq: min-heap implementation in Python.
The heap invariant: heap[k] <= heap[2*k+1] and heap[k] <= heap[2*k+2]

Key operations:
  heappush(heap, item):   O(log n)
  heappop(heap):          O(log n)
  heap[0]:                O(1)  — peek at minimum
  heapify(list):          O(n)  — convert list to heap
  nlargest(n, iter):      O(n + k log n)
  nsmallest(n, iter):     O(n + k log n)

Use when:
  - Need to repeatedly get minimum/maximum
  - Priority queues
  - Top-K problems
  - Merge K sorted lists
  - Dijkstra's algorithm
"""

# ─────────────────────────────────────────
# CORE OPERATIONS
# ─────────────────────────────────────────
heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 2)
heapq.heappush(heap, 8)
heapq.heappush(heap, 1)
heapq.heappush(heap, 9)
heapq.heappush(heap, 3)

print(f"Heap:   {heap}")        # [1, 2, 3, 5, 9, 8] (heap order)
print(f"Min:    {heap[0]}")     # 1 — always O(1)

print(heapq.heappop(heap))      # 1 — removes and returns minimum
print(heapq.heappop(heap))      # 2
print(f"After pops: {heap}")

# heapify: faster than pushing one by one
numbers = [7, 1, 8, 2, 6, 3, 9, 4, 5]
heapq.heapify(numbers)          # O(n) in-place
print(f"Heapified: {numbers}")  # valid heap structure

# pushpop: push then pop (more efficient than two operations)
result = heapq.heappushpop(numbers, 0)  # push 0, pop min
print(f"Pushed 0, popped: {result}")    # 0 (just pushed, then popped)

# ─────────────────────────────────────────
# TOP-K PROBLEMS
# ─────────────────────────────────────────
data = [random.randint(1, 1000) for _ in range(100_000)]

# Find top 10 largest
top_10 = heapq.nlargest(10, data)
print(f"Top 10: {top_10}")

# Find top 10 smallest
bottom_10 = heapq.nsmallest(10, data)
print(f"Bottom 10: {bottom_10}")

# Find top K by key (common backend use)
users_with_scores = [
    {"name": "Alice", "score": 92, "purchases": 15},
    {"name": "Bob", "score": 85, "purchases": 8},
    {"name": "Charlie", "score": 97, "purchases": 22},
    {"name": "Dave", "score": 78, "purchases": 5},
    {"name": "Eve", "score": 92, "purchases": 19},
]

top_3_by_score = heapq.nlargest(3, users_with_scores, key=lambda u: u["score"])
print("Top 3 by score:")
for u in top_3_by_score:
    print(f"  {u['name']}: {u['score']}")

# ─────────────────────────────────────────
# REAL USE: Merge K sorted lists
# ─────────────────────────────────────────
def merge_k_sorted(lists: list) -> list:
    """
    Merge K sorted lists into one sorted list.
    O(n log k) where n = total elements, k = number of lists.
    Much better than sorting everything: O(n log n).
    """
    result = []
    # Heap contains: (value, list_index, element_index)
    heap = []

    # Initialize heap with first element from each list
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))

    while heap:
        val, list_idx, elem_idx = heapq.heappop(heap)
        result.append(val)

        # Add next element from same list if available
        if elem_idx + 1 < len(lists[list_idx]):
            next_val = lists[list_idx][elem_idx + 1]
            heapq.heappush(heap, (next_val, list_idx, elem_idx + 1))

    return result


k_lists = [
    [1, 4, 7, 10],
    [2, 5, 8, 11],
    [3, 6, 9, 12],
]
print(f"Merged: {merge_k_sorted(k_lists)}")
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]

# ─────────────────────────────────────────
# REAL USE: Running median (hard problem!)
# ─────────────────────────────────────────
class RunningMedian:
    """
    Maintain running median as numbers arrive.
    O(log n) per insertion, O(1) to get median.

    Strategy: two heaps
      max_heap: lower half (as max-heap using negation)
      min_heap: upper half (as min-heap)
    """

    def __init__(self):
        self.max_heap = []  # lower half, max at top
        self.min_heap = []  # upper half, min at top

    def add(self, num: float) -> None:
        # Add to max heap (lower half)
        heapq.heappush(self.max_heap, -num)

        # Balance: max of lower half ≤ min of upper half
        if self.max_heap and self.min_heap:
            if -self.max_heap[0] > self.min_heap[0]:
                val = -heapq.heappop(self.max_heap)
                heapq.heappush(self.min_heap, val)

        # Balance sizes: max_heap can have at most 1 more element
        if len(self.max_heap) > len(self.min_heap) + 1:
            val = -heapq.heappop(self.max_heap)
            heapq.heappush(self.min_heap, val)
        elif len(self.min_heap) > len(self.max_heap):
            val = heapq.heappop(self.min_heap)
            heapq.heappush(self.max_heap, -val)

    def get_median(self) -> float:
        if len(self.max_heap) > len(self.min_heap):
            return -self.max_heap[0]    # odd count: top of max_heap
        return (-self.max_heap[0] + self.min_heap[0]) / 2  # even: average

    def __repr__(self):
        return f"RunningMedian(median={self.get_median()})"


rm = RunningMedian()
for num in [5, 3, 8, 1, 9, 2, 7]:
    rm.add(num)
    print(f"Added {num}, median = {rm.get_median()}")
```

---

## 6. ⚡ Generator Efficiency

```python
"""
Generators are the most important performance tool
for processing large datasets in backend systems.

Memory: generators use O(1) vs O(n) for lists
Speed:  generators can be faster due to lazy evaluation
"""

import sys

# ─────────────────────────────────────────
# MEMORY COMPARISON
# ─────────────────────────────────────────
# List comprehension — stores ALL values in memory
list_comp = [x ** 2 for x in range(1_000_000)]
print(f"List: {sys.getsizeof(list_comp):,} bytes")  # ~8,000,056 bytes

# Generator expression — stores NO values
gen_exp = (x ** 2 for x in range(1_000_000))
print(f"Generator: {sys.getsizeof(gen_exp)} bytes")  # 104 bytes!

# ─────────────────────────────────────────
# GENERATORS FOR DATABASE PROCESSING
# ─────────────────────────────────────────

def simulate_db_query(query: str, batch_size: int = 1000):
    """Simulate database that returns results in batches."""
    # Real: would use SQLAlchemy with yield_per()
    total = 100_000
    for offset in range(0, total, batch_size):
        # Simulate one batch
        batch = [
            {"id": i, "email": f"user{i}@test.com", "active": i % 3 != 0}
            for i in range(offset, min(offset + batch_size, total))
        ]
        yield from batch    # yield each record individually

def process_users_efficiently():
    """
    Process 100k users without loading all into memory.
    Memory stays constant regardless of dataset size.
    """
    count = 0
    active_count = 0

    for user in simulate_db_query("SELECT * FROM users"):
        count += 1
        if user["active"]:
            active_count += 1
        # Memory: only holds ONE user at a time!

    return count, active_count

total, active = process_users_efficiently()
print(f"Total: {total:,}, Active: {active:,}")

# ─────────────────────────────────────────
# GENERATOR PIPELINE — composable processing
# ─────────────────────────────────────────
def read_csv_rows(filepath: str):
    """Read CSV one row at a time."""
    import csv
    try:
        with open(filepath, "r") as f:
            reader = csv.DictReader(f)
            for row in reader:
                yield row
    except FileNotFoundError:
        # For demo purposes
        fake_data = [
            {"email": f"user{i}@test.com", "age": str(20 + i % 50), "active": "true"}
            for i in range(10000)
        ]
        yield from fake_data

def filter_active(records):
    """Keep only active records."""
    for record in records:
        if record.get("active") == "true":
            yield record

def parse_types(records):
    """Convert string values to proper types."""
    for record in records:
        record["age"] = int(record.get("age", 0))
        yield record

def filter_adults(records, min_age: int = 18):
    """Keep only adults."""
    for record in records:
        if record["age"] >= min_age:
            yield record

def transform_for_api(records):
    """Transform to API response format."""
    for record in records:
        yield {
            "email": record["email"].lower(),
            "age": record["age"],
        }

# BUILD THE PIPELINE — nothing runs yet!
pipeline = read_csv_rows("users.csv")
pipeline = filter_active(pipeline)
pipeline = parse_types(pipeline)
pipeline = filter_adults(pipeline, min_age=25)
pipeline = transform_for_api(pipeline)

# Only NOW does data flow through
count = 0
for user in pipeline:
    count += 1
    # process each user

print(f"Processed {count} adult active users")
# Memory used: O(1) — one record at a time through all stages!

# ─────────────────────────────────────────
# ITERTOOLS FOR EFFICIENT COMBINATIONS
# ─────────────────────────────────────────
import itertools

# chain — combine iterables without creating new list
list1 = [1, 2, 3]
list2 = [4, 5, 6]
list3 = [7, 8, 9]

# BAD: creates new list in memory
combined_bad = list1 + list2 + list3  # O(n) memory

# GOOD: lazy chain
combined_good = itertools.chain(list1, list2, list3)  # O(1) memory
for item in combined_good:
    pass  # processes one at a time

# islice — take N items from potentially infinite iterator
def infinite_counter():
    n = 0
    while True:
        yield n
        n += 1

first_10 = list(itertools.islice(infinite_counter(), 10))
print(f"First 10: {first_10}")

# groupby — group consecutive items efficiently
from itertools import groupby

events = [
    {"type": "login", "user": "alice"},
    {"type": "login", "user": "bob"},
    {"type": "purchase", "user": "alice"},
    {"type": "purchase", "user": "charlie"},
    {"type": "logout", "user": "alice"},
]

# Must sort first!
sorted_events = sorted(events, key=lambda e: e["type"])

for event_type, group in groupby(sorted_events, key=lambda e: e["type"]):
    users = [e["user"] for e in group]
    print(f"{event_type}: {users}")
# login: ['alice', 'bob']
# logout: ['alice']
# purchase: ['alice', 'charlie']
```

---

## 7. 🎯 Choosing the Right Tool — Complete Decision Guide

```python
# ─────────────────────────────────────────
# THE COMPLETE CHEAT SHEET
# ─────────────────────────────────────────

def choose_data_structure(scenario: str) -> str:
    """Decision guide for choosing Python data structures."""
    guide = {
        "unique_items_fast_lookup": "set",
        "key_value_mapping": "dict",
        "ordered_sequence_mutable": "list",
        "ordered_sequence_immutable": "tuple",
        "counting_occurrences": "Counter",
        "grouping_items": "defaultdict",
        "dedup_with_order_preserved": "dict.fromkeys()",
        "sorted_insertions": "bisect + list",
        "priority_queue": "heapq",
        "both_ends_queue": "deque",
        "bounded_sliding_window": "deque(maxlen=n)",
        "large_dataset_processing": "generator",
        "layered_config": "ChainMap",
        "readonly_named_data": "namedtuple or dataclass(frozen=True)",
    }
    return guide.get(scenario, "list (default)")


# ─────────────────────────────────────────
# QUICK REFERENCE TABLE
# ─────────────────────────────────────────
print("""
╔══════════════════╦═══════╦═══════╦═══════╦═════════╗
║ Operation        ║ list  ║ tuple ║  set  ║  dict   ║
╠══════════════════╬═══════╬═══════╬═══════╬═════════╣
║ index access [i] ║  O(1) ║  O(1) ║  ❌   ║   ❌    ║
║ search (in)      ║  O(n) ║  O(n) ║  O(1) ║  O(1)   ║
║ insert/add       ║  O(1) ║  ❌   ║  O(1) ║  O(1)   ║
║ delete           ║  O(n) ║  ❌   ║  O(1) ║  O(1)   ║
║ ordered          ║  ✅   ║  ✅   ║  ❌   ║  ✅*    ║
║ duplicates       ║  ✅   ║  ✅   ║  ❌   ║  keys❌ ║
║ memory           ║  low  ║  low  ║  high ║  high   ║
║ hashable         ║  ❌   ║  ✅   ║  ❌   ║  ❌     ║
╚══════════════════╩═══════╩═══════╩═══════╩═════════╝
* dict preserves insertion order since Python 3.7
""")

# ─────────────────────────────────────────
# ANTI-PATTERNS TO AVOID
# ─────────────────────────────────────────

# ❌ Anti-pattern 1: Linear search in hot path
# BAD
VALID_ROLES = ["admin", "user", "moderator", "guest"]
def check_role_slow(role: str) -> bool:
    return role in VALID_ROLES  # O(n) every call

# GOOD
VALID_ROLES_SET = {"admin", "user", "moderator", "guest"}
def check_role_fast(role: str) -> bool:
    return role in VALID_ROLES_SET  # O(1) every call

# ❌ Anti-pattern 2: list.pop(0) in a loop
# BAD — O(n²)
def process_bad(items: list) -> list:
    result = []
    while items:
        item = items.pop(0)     # O(n)!
        result.append(item * 2)
    return result

# GOOD — O(n)
def process_good(items: list) -> list:
    q = deque(items)
    result = []
    while q:
        item = q.popleft()      # O(1)
        result.append(item * 2)
    return result

# ❌ Anti-pattern 3: string concatenation in loop
# BAD — O(n²)
def build_csv_bad(rows: list) -> str:
    result = ""
    for row in rows:
        result += ",".join(str(v) for v in row) + "\n"
    return result

# GOOD — O(n)
def build_csv_good(rows: list) -> str:
    lines = [",".join(str(v) for v in row) for row in rows]
    return "\n".join(lines)

# ❌ Anti-pattern 4: creating list when generator suffices
# BAD — loads all into memory
def sum_squares_bad(n: int) -> int:
    return sum([x**2 for x in range(n)])  # creates full list

# GOOD — no intermediate list
def sum_squares_good(n: int) -> int:
    return sum(x**2 for x in range(n))    # generator expression

# ❌ Anti-pattern 5: using sort when you need top-k
# BAD — O(n log n) to get k elements
def top_k_bad(nums: list, k: int) -> list:
    return sorted(nums, reverse=True)[:k]  # sorts everything

# GOOD — O(n log k)
def top_k_good(nums: list, k: int) -> list:
    return heapq.nlargest(k, nums)         # only tracks top k
```

---

## 8. 🔬 Practical Benchmarking

```python
import timeit
import functools

def benchmark(func, *args, number: int = 1000, **kwargs) -> float:
    """Benchmark a function call, return ms per call."""
    timer = timeit.Timer(
        functools.partial(func, *args, **kwargs)
    )
    total = timer.timeit(number=number)
    return (total / number) * 1000  # ms per call

# ─────────────────────────────────────────
# BENCHMARK: Set vs list for membership
# ─────────────────────────────────────────
sizes = [100, 1_000, 10_000, 100_000]

print("Membership test: x in collection")
print(f"{'Size':>10} {'list (ms)':>12} {'set (ms)':>12} {'speedup':>10}")
print("-" * 46)

for n in sizes:
    data_list = list(range(n))
    data_set = set(range(n))
    target = n - 1  # worst case

    list_time = benchmark(lambda: target in data_list)
    set_time = benchmark(lambda: target in data_set)
    speedup = list_time / set_time if set_time > 0 else float('inf')

    print(f"{n:>10,} {list_time:>12.4f} {set_time:>12,.6f} {speedup:>9.0f}x")

# ─────────────────────────────────────────
# BENCHMARK: deque vs list for queue
# ─────────────────────────────────────────
print("\nQueue operations: pop from left")
print(f"{'Size':>10} {'list (ms)':>12} {'deque (ms)':>12}")
print("-" * 36)

for n in [1_000, 10_000]:

    def pop_all_deque():
        d = deque(range(n))
        while d:
            d.popleft()

    def pop_all_list():
        lst = list(range(n))
        while lst:
            lst.pop(0)

    deque_time = benchmark(pop_all_deque, number=100)
    list_time = benchmark(pop_all_list, number=100)

    print(f"{n:>10,} {list_time:>12.2f} {deque_time:>12.2f}")
```

---

## 📊 Visual Summary

```
┌────────────────────────────────────────────────────────────────┐
│              PYTHON EFFICIENCY CHEAT SHEET                     │
│                                                                │
│  CHOOSE THE RIGHT STRUCTURE:                                   │
│  ─────────────────────────────                                 │
│  Need fast lookup?        → set (O(1)) or dict (O(1))         │
│  Need ordered sequence?   → list or tuple                      │
│  Need immutable data?     → tuple or frozenset                 │
│  Need to count things?    → Counter                            │
│  Need to group things?    → defaultdict(list)                  │
│  Need both-end queue?     → deque                              │
│  Need minimum/maximum?    → heapq                              │
│  Need sorted insertions?  → bisect + list                      │
│  Need large data?         → generator                          │
│                                                                │
│  AVOID THESE MISTAKES:                                         │
│  ──────────────────────                                        │
│  ❌ x in my_list (large) → use set                            │
│  ❌ list.pop(0)          → use deque.popleft()                 │
│  ❌ result += string     → use list + join()                   │
│  ❌ sorted(data)[:k]     → use heapq.nlargest(k, data)        │
│  ❌ [x for x in data]    → use (x for x in data) if consuming │
│  ❌ list + list + list   → use itertools.chain()               │
│                                                                │
│  COMPLEXITY REFERENCE:                                         │
│  ───────────────────────                                       │
│  list[i]:         O(1)   list.pop(0):     O(n)               │
│  list.append():   O(1)   list.insert(0):  O(n)               │
│  list search:     O(n)   sorted():        O(n log n)          │
│  dict/set lookup: O(1)   dict/set add:    O(1)               │
│  deque both ends: O(1)   heappush/pop:    O(log n)           │
│  bisect search:   O(logn) bisect insert:  O(n)               │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

1. When would you use a set instead of a list? What is the memory trade-off?
2. Why is `list.pop(0)` slow? What should you use instead?
3. Why is string concatenation (`+=`) in a loop bad? What is the correct approach?
4. When would you use `Counter` vs `defaultdict`?
5. What is `deque(maxlen=n)` good for?
6. When would you use `bisect` instead of a regular dict?
7. Why is `heapq.nlargest` better than `sorted()[:k]`?
8. Why are generator expressions more memory efficient?
9. What does `ChainMap` do?
10. When would you use `namedtuple` instead of a dict?
11. What is the time complexity of:
    a) `x in set`   b) `x in list`   c) `dict[key]`
    d) `heappush`   e) `bisect_left`   f) `deque.popleft()`
12. How would you maintain a sorted list with frequent insertions?

---

## 🛠️ Practice Exercises

```python
# Exercise 1: Collection Choice
# You're building a spam filter.
# You have 500,000 known spam email addresses.
# Your API receives 10,000 requests/second.
# Each request: check if sender is in spam list.
#
# Question: Which data structure? Why?
# Write the implementation.

# Exercise 2: Sliding Window + deque
# Implement a function that finds the maximum value
# in every window of size k in an array.
# Must be O(n) total, not O(nk).
# Use a deque to track indices.
# sliding_window_max([1,3,-1,-3,5,3,6,7], k=3)
# → [3, 3, 5, 5, 6, 7]

# Exercise 3: Generator Pipeline
# Build a data processing pipeline using generators:
# 1. read_logs() → generates log lines (simulate 1M lines)
# 2. parse_log(lines) → parses each line to dict
# 3. filter_errors(records) → keeps only error logs
# 4. extract_ip(records) → extracts IP addresses
# 5. count_by_ip(ips) → counts occurrences per IP
# All steps except the last should be generators.
# Count top 5 IPs with most errors.

# Exercise 4: heapq — K closest points
# Given a list of (x, y) coordinates and origin (0,0),
# find the K closest points to origin.
# Use a max-heap of size K for O(n log k) solution.
# k_closest([(1,1),(3,3),(0,2),(2,0)], k=2)
# → [(1,1), (0,2)]  (distance sqrt(2) and 2)

# Exercise 5: bisect — Inventory management
# Build an inventory system that:
# - Maintains a sorted list of item prices
# - Supports: add_item(price), remove_item(price)
# - count_in_range(low, high) → items in price range
# - median_price() → median price
# All operations should be as efficient as possible.
# Use bisect for O(log n) operations.
```

---

## 🎉 Phase 2 Complete!

```
✅ 2.1 — Data Structures
         Arrays, Linked Lists, Stacks, Queues,
         Hash Tables, Trees, Heaps, Graphs, Tries

✅ 2.2 — Algorithms
         Searching, Sorting, Two Pointers,
         Sliding Window, Recursion, Backtracking,
         BFS, DFS, Dynamic Programming, Greedy, Hashing

✅ 2.3 — Python-Specific Efficiency
         list vs tuple vs set vs dict decisions,
         Built-in time complexities,
         Collections module, bisect, heapq,
         Generator efficiency, anti-patterns
```

**You now think algorithmically AND write efficient Python.**

---

## What's Next?

```
Option 1: Continue  → Phase 3: Databases
                      PostgreSQL, SQL, SQLAlchemy,
                      Redis, MongoDB — the real backend work begins!

Option 2: Projects  → Build Phase 2 projects:
                      Project 4: LeetCode Tracker
                      Project 5: AutoComplete System

Option 3: Questions → Anything unclear?
```

**What do you want to do? 🚀**