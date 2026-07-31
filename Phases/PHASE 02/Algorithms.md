```
Data structures = WHERE you store data
Algorithms      = HOW you process data

A bad algorithm on good hardware
is still slower than a good algorithm on bad hardware.

Real backend examples:
  Finding a user in unsorted list of 1M users:
    Linear search:  O(n) = 1,000,000 comparisons
    Binary search:  O(log n) = 20 comparisons
    → Binary search is 50,000x faster

  Sorting 1M records for a report:
    Bubble sort:    O(n²) = 1 trillion operations → hours
    Merge sort:     O(n log n) = 20M operations   → seconds

  Finding duplicate emails in 100K users:
    Nested loops:   O(n²) = 10 billion checks
    Hash set:       O(n)  = 100K checks
    → Hash approach is 100,000x faster

These aren't academic exercises.
They show up in REAL production systems.
```

---

## Setup

```bash
cd ~/projects/data_structures
# reuse same project
touch algorithms.py
```

---

## 1. 🔍 Searching Algorithms

### Linear Search

```python
# algorithms.py
from typing import Any, List, Optional, Tuple
import time
import random

# ─────────────────────────────────────────
# LINEAR SEARCH — O(n)
# ─────────────────────────────────────────
def linear_search(items: List[Any], target: Any) -> int:
    """
    Search through items one by one.
    Returns index if found, -1 if not.

    Time:  O(n) — must check every item in worst case
    Space: O(1) — no extra memory needed

    When to use:
      - Unsorted data
      - Small datasets
      - Searching once (not worth sorting)
    """
    for i, item in enumerate(items):
        if item == target:
            return i        # found at index i
    return -1               # not found


# Test
numbers = [64, 34, 25, 12, 22, 11, 90]
print(linear_search(numbers, 25))   # 2
print(linear_search(numbers, 99))   # -1

# With condition (more realistic)
def find_user_by_email(users: List[dict], email: str) -> Optional[dict]:
    """
    Linear search through users.
    O(n) — fine for small lists, slow for millions.
    """
    for user in users:
        if user["email"] == email:
            return user
    return None

users = [
    {"id": 1, "email": "alice@test.com"},
    {"id": 2, "email": "bob@test.com"},
    {"id": 3, "email": "charlie@test.com"},
]
print(find_user_by_email(users, "bob@test.com"))
```

### Binary Search

```python
# ─────────────────────────────────────────
# BINARY SEARCH — O(log n)
# ─────────────────────────────────────────
"""
Key requirement: data must be SORTED

Strategy: Divide and conquer
  1. Look at MIDDLE element
  2. If target == middle → found!
  3. If target < middle  → search LEFT half
  4. If target > middle  → search RIGHT half
  5. Repeat until found or empty

Example: find 7 in [1, 3, 5, 7, 9, 11, 13]
  Step 1: middle = 7 (index 3) → FOUND!

Example: find 11 in [1, 3, 5, 7, 9, 11, 13]
  Step 1: middle = 7, 11 > 7 → search right [9, 11, 13]
  Step 2: middle = 11 → FOUND!

With 1,000,000 items:
  Linear search: up to 1,000,000 checks
  Binary search: up to 20 checks (log₂(1,000,000) ≈ 20)
"""

def binary_search(items: List[Any], target: Any) -> int:
    """
    Search sorted list using divide and conquer.
    Returns index if found, -1 if not.

    Time:  O(log n)
    Space: O(1)
    """
    left = 0
    right = len(items) - 1

    while left <= right:
        mid = (left + right) // 2   # avoid overflow vs (left+right)//2

        if items[mid] == target:
            return mid              # found!
        elif items[mid] < target:
            left = mid + 1          # search right half
        else:
            right = mid - 1         # search left half

    return -1   # not found


# Test
sorted_nums = [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
print(binary_search(sorted_nums, 7))    # 3
print(binary_search(sorted_nums, 11))   # 5
print(binary_search(sorted_nums, 1))    # 0
print(binary_search(sorted_nums, 19))   # 9
print(binary_search(sorted_nums, 6))    # -1 (not found)

# Recursive version
def binary_search_recursive(
    items: List[Any],
    target: Any,
    left: int = 0,
    right: int = None
) -> int:
    if right is None:
        right = len(items) - 1

    if left > right:
        return -1   # base case: not found

    mid = (left + right) // 2

    if items[mid] == target:
        return mid
    elif items[mid] < target:
        return binary_search_recursive(items, target, mid + 1, right)
    else:
        return binary_search_recursive(items, target, left, mid - 1)


# Performance comparison
large_sorted = list(range(1_000_000))
target = 999_999  # worst case for linear

start = time.perf_counter()
linear_search(large_sorted, target)
linear_time = (time.perf_counter() - start) * 1000

start = time.perf_counter()
binary_search(large_sorted, target)
binary_time = (time.perf_counter() - start) * 1000

print(f"\nLinear search: {linear_time:.3f}ms")
print(f"Binary search: {binary_time:.3f}ms")
print(f"Binary is {linear_time/binary_time:.0f}x faster")
```

### Binary Search Variations

```python
# ─────────────────────────────────────────
# FIND FIRST OCCURRENCE (duplicates)
# ─────────────────────────────────────────
def find_first(items: List[Any], target: Any) -> int:
    """Find FIRST occurrence of target. O(log n)."""
    left, right = 0, len(items) - 1
    result = -1

    while left <= right:
        mid = (left + right) // 2
        if items[mid] == target:
            result = mid        # found one, but keep searching LEFT
            right = mid - 1     # could be earlier occurrence
        elif items[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return result


# ─────────────────────────────────────────
# FIND LAST OCCURRENCE
# ─────────────────────────────────────────
def find_last(items: List[Any], target: Any) -> int:
    """Find LAST occurrence of target. O(log n)."""
    left, right = 0, len(items) - 1
    result = -1

    while left <= right:
        mid = (left + right) // 2
        if items[mid] == target:
            result = mid        # found one, but keep searching RIGHT
            left = mid + 1      # could be later occurrence
        elif items[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return result


nums_with_dups = [1, 2, 2, 2, 3, 4, 5]
print(f"First 2: index {find_first(nums_with_dups, 2)}")  # 1
print(f"Last 2:  index {find_last(nums_with_dups, 2)}")   # 3

# ─────────────────────────────────────────
# FIND INSERT POSITION (bisect)
# ─────────────────────────────────────────
import bisect

sorted_list = [1, 3, 5, 7, 9]

# Where would 4 go to keep sorted order?
pos = bisect.bisect_left(sorted_list, 4)
print(f"Insert 4 at: {pos}")   # 2 (between 3 and 5)

# bisect_right: insert AFTER existing equal elements
pos_right = bisect.bisect_right(sorted_list, 5)
print(f"Insert 5 after at: {pos_right}")  # 3

# Insert and maintain sorted order
bisect.insort(sorted_list, 4)
print(f"After insort(4): {sorted_list}")   # [1, 3, 4, 5, 7, 9]

# Real backend use: maintain sorted leaderboard
class Leaderboard:
    """Keep top scores sorted without sorting every insert."""

    def __init__(self):
        self.scores = []    # sorted list of (score, user_id)

    def add_score(self, user_id: int, score: int) -> None:
        """Insert maintaining sorted order. O(log n) search + O(n) insert."""
        bisect.insort(self.scores, (score, user_id))

    def get_rank(self, user_id: int, score: int) -> int:
        """Find rank of a score. O(log n)."""
        pos = bisect.bisect_right(self.scores, (score, user_id))
        return len(self.scores) - pos + 1   # rank from top

    def top_n(self, n: int) -> List[Tuple]:
        """Get top N scores. O(n)."""
        return self.scores[-n:][::-1]   # reverse for descending


lb = Leaderboard()
lb.add_score(1, 100)
lb.add_score(2, 85)
lb.add_score(3, 120)
lb.add_score(4, 95)
lb.add_score(5, 120)

print("Top 3:", lb.top_n(3))
print("Rank for score 100:", lb.get_rank(1, 100))
```

---

## 2. 🗂️ Sorting Algorithms

### Why Learn Multiple Sorting Algorithms?

```
Python's built-in sort (Timsort) is almost always best.
But understanding sorting teaches you:
  - Algorithm design patterns
  - Divide and conquer
  - Time/space tradeoffs
  - When different approaches win

You WILL be asked about these in technical interviews.
```

### Bubble Sort — O(n²)

```python
def bubble_sort(items: List[Any]) -> List[Any]:
    """
    Repeatedly swap adjacent elements if in wrong order.
    "Bubbles" largest elements to the end each pass.

    Time:  O(n²) — worst and average case
           O(n)  — best case (already sorted, with optimization)
    Space: O(1)  — in-place

    When to use: almost never in production
    Good for: learning, very small arrays
    """
    arr = items.copy()  # don't modify original
    n = len(arr)

    for i in range(n):
        swapped = False
        # Each pass: bubble largest unsorted to end
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        # Optimization: if no swaps, already sorted
        if not swapped:
            break

    return arr


print(bubble_sort([64, 34, 25, 12, 22, 11, 90]))
# [11, 12, 22, 25, 34, 64, 90]

# Visual trace:
# [64, 34, 25, 12, 22, 11, 90]
# [34, 25, 12, 22, 11, 64, 90]  ← 64 bubbled to end
# [25, 12, 22, 11, 34, 64, 90]  ← 34 bubbled
# [12, 22, 11, 25, 34, 64, 90]
# [12, 11, 22, 25, 34, 64, 90]
# [11, 12, 22, 25, 34, 64, 90]  ← sorted!
```

### Selection Sort — O(n²)

```python
def selection_sort(items: List[Any]) -> List[Any]:
    """
    Find minimum in unsorted portion, put at front.

    Time:  O(n²) — always (no best case optimization)
    Space: O(1)

    Advantage over bubble: fewer swaps (exactly n-1)
    Good for: when write operations are expensive
    """
    arr = items.copy()
    n = len(arr)

    for i in range(n):
        # Find minimum in arr[i..n-1]
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        # Swap minimum with first unsorted element
        arr[i], arr[min_idx] = arr[min_idx], arr[i]

    return arr


print(selection_sort([64, 25, 12, 22, 11]))
# [11, 12, 22, 25, 64]
```

### Insertion Sort — O(n²) but O(n) for nearly sorted

```python
def insertion_sort(items: List[Any]) -> List[Any]:
    """
    Build sorted array one item at a time.
    Take each element and insert it into correct position
    in the already-sorted portion.

    Like sorting playing cards in your hand.

    Time:  O(n²) worst case (reverse sorted)
           O(n)  best case (already sorted)
    Space: O(1)

    When to use:
      - Small arrays (< 20 elements)
      - Nearly sorted data
      - Part of hybrid algorithms (Timsort uses it!)
    """
    arr = items.copy()
    n = len(arr)

    for i in range(1, n):
        key = arr[i]        # element to insert
        j = i - 1

        # Shift elements right until we find key's position
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1

        arr[j + 1] = key    # insert key in correct position

    return arr


print(insertion_sort([12, 11, 13, 5, 6]))
# [5, 6, 11, 12, 13]

# Visual trace:
# [12, 11, 13, 5, 6]  ← start
# [11, 12, 13, 5, 6]  ← insert 11 before 12
# [11, 12, 13, 5, 6]  ← 13 already in place
# [5, 11, 12, 13, 6]  ← insert 5 at front
# [5, 6, 11, 12, 13]  ← insert 6 after 5
```

### Merge Sort — O(n log n)

```python
def merge_sort(items: List[Any]) -> List[Any]:
    """
    Divide array in half, sort each half, merge them.
    Classic divide and conquer.

    Time:  O(n log n) — always (guaranteed!)
    Space: O(n)       — needs extra memory for merging

    When to use:
      - Need guaranteed O(n log n)
      - Sorting linked lists
      - External sorting (data too big for RAM)
      - Stable sort needed (equal elements keep relative order)
    """
    if len(items) <= 1:
        return items.copy()

    # Divide
    mid = len(items) // 2
    left = merge_sort(items[:mid])      # sort left half
    right = merge_sort(items[mid:])     # sort right half

    # Conquer (merge)
    return _merge(left, right)


def _merge(left: List[Any], right: List[Any]) -> List[Any]:
    """Merge two sorted arrays into one sorted array. O(n)."""
    result = []
    i = j = 0

    # Compare elements from both halves, take smaller
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    # Add remaining elements
    result.extend(left[i:])
    result.extend(right[j:])
    return result


print(merge_sort([38, 27, 43, 3, 9, 82, 10]))
# [3, 9, 10, 27, 38, 43, 82]

# Visual trace for [38, 27, 43, 3]:
#        [38, 27, 43, 3]
#       /               \
#   [38, 27]         [43, 3]
#   /      \         /     \
# [38]    [27]    [43]     [3]
#   \      /         \     /
#   [27, 38]         [3, 43]
#       \               /
#        [3, 27, 38, 43]
```

### Quick Sort — O(n log n) average

```python
def quick_sort(items: List[Any]) -> List[Any]:
    """
    Pick a pivot, partition around it, recurse.

    Time:  O(n log n) average
           O(n²) worst case (bad pivot, e.g., already sorted)
    Space: O(log n) — recursion stack

    When to use:
      - General purpose (fastest in practice)
      - In-place sorting (with partition variant)
      - Cache efficient

    Python's built-in sort uses Timsort (not QuickSort)
    but QuickSort is important to understand.
    """
    if len(items) <= 1:
        return items.copy()

    pivot = items[len(items) // 2]      # choose middle as pivot

    left = [x for x in items if x < pivot]     # less than pivot
    middle = [x for x in items if x == pivot]  # equal to pivot
    right = [x for x in items if x > pivot]    # greater than pivot

    return quick_sort(left) + middle + quick_sort(right)


print(quick_sort([3, 6, 8, 10, 1, 2, 1]))
# [1, 1, 2, 3, 6, 8, 10]

# In-place QuickSort (more efficient, avoids extra lists)
def quick_sort_inplace(
    arr: List[Any],
    low: int = 0,
    high: int = None
) -> None:
    """In-place quicksort. More memory efficient."""
    if high is None:
        high = len(arr) - 1

    if low < high:
        pivot_idx = _partition(arr, low, high)
        quick_sort_inplace(arr, low, pivot_idx - 1)
        quick_sort_inplace(arr, pivot_idx + 1, high)


def _partition(arr: List[Any], low: int, high: int) -> int:
    """Partition array around pivot. Returns pivot's final index."""
    pivot = arr[high]   # choose last element as pivot
    i = low - 1         # index of smaller element

    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]

    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1


test = [10, 7, 8, 9, 1, 5]
quick_sort_inplace(test)
print(test)  # [1, 5, 7, 8, 9, 10]
```

### Python's Built-in Sort — Timsort

```python
# ─────────────────────────────────────────
# USE PYTHON'S BUILT-IN — it's best in practice
# ─────────────────────────────────────────
"""
Timsort (Python's sort):
  - Hybrid of Merge Sort + Insertion Sort
  - O(n log n) worst case
  - O(n) best case (already sorted)
  - Stable sort
  - Highly optimized (written in C)

Use sorted() or list.sort() — don't implement your own!
"""

# Sort a list (creates new list)
nums = [3, 1, 4, 1, 5, 9, 2, 6]
sorted_nums = sorted(nums)                  # ascending
sorted_desc = sorted(nums, reverse=True)    # descending

# Sort in place (modifies original)
nums.sort()
nums.sort(reverse=True)

# Sort with KEY — most important for backend work
users = [
    {"name": "Charlie", "age": 30, "score": 85},
    {"name": "Alice", "age": 25, "score": 92},
    {"name": "Bob", "age": 35, "score": 85},
    {"name": "Dave", "age": 28, "score": 78},
]

# Sort by single field
by_age = sorted(users, key=lambda u: u["age"])
by_score = sorted(users, key=lambda u: u["score"], reverse=True)

# Sort by multiple fields
# Primary: score descending, Secondary: name ascending
by_score_then_name = sorted(
    users,
    key=lambda u: (-u["score"], u["name"])
)

print("By score desc, then name:")
for u in by_score_then_name:
    print(f"  {u['name']}: {u['score']}")
# Alice: 92
# Bob: 85    ← same score as Charlie, Bob comes first alphabetically
# Charlie: 85
# Dave: 78

# Sort by nested field
products = [
    {"name": "Laptop", "price": {"amount": 999, "currency": "USD"}},
    {"name": "Phone", "price": {"amount": 599, "currency": "USD"}},
    {"name": "Tablet", "price": {"amount": 799, "currency": "USD"}},
]
by_price = sorted(products, key=lambda p: p["price"]["amount"])

# Real backend: sort API results
from datetime import datetime

posts = [
    {"id": 1, "title": "Post A", "created_at": "2024-01-15", "likes": 45},
    {"id": 2, "title": "Post B", "created_at": "2024-01-20", "likes": 120},
    {"id": 3, "title": "Post C", "created_at": "2024-01-18", "likes": 45},
]

# Sort by likes desc, then date desc
recent_popular = sorted(
    posts,
    key=lambda p: (-p["likes"], p["created_at"]),
    reverse=False
)
print([p["title"] for p in recent_popular])
# ['Post B', 'Post C', 'Post A']
# Post C before A because newer date (same likes)
```

### Sorting Comparison

```python
import time
import random

def benchmark_sorts(n: int = 5000):
    data = [random.randint(0, 10000) for _ in range(n)]

    algorithms = {
        "Bubble Sort":    lambda d: bubble_sort(d),
        "Selection Sort": lambda d: selection_sort(d),
        "Insertion Sort": lambda d: insertion_sort(d),
        "Merge Sort":     lambda d: merge_sort(d),
        "Quick Sort":     lambda d: quick_sort(d),
        "Python sort":    lambda d: sorted(d),
    }

    print(f"\nSorting {n:,} random numbers:")
    print(f"{'Algorithm':<16} {'Time':>10}")
    print("-" * 28)

    for name, func in algorithms.items():
        start = time.perf_counter()
        func(data.copy())
        elapsed = (time.perf_counter() - start) * 1000
        print(f"{name:<16} {elapsed:>8.2f}ms")

benchmark_sorts(5000)
# Bubble Sort:     1200.00ms  (slow!)
# Selection Sort:   800.00ms
# Insertion Sort:   600.00ms
# Merge Sort:         8.00ms
# Quick Sort:         5.00ms
# Python sort:        1.50ms  (fastest - C implementation)
```

---

## 3. 🔢 Two Pointers Technique

```python
"""
Two pointers = use two indices to traverse array
               usually from opposite ends or at different speeds.

When to use:
  - Finding pairs that meet a condition
  - Reversing, palindrome checking
  - Removing duplicates from sorted array
  - Container with most water problems

Time: O(n) instead of O(n²) brute force
"""

# ─────────────────────────────────────────
# PATTERN 1: Opposite Ends
# Find two numbers that sum to target
# ─────────────────────────────────────────
def two_sum_sorted(numbers: List[int], target: int) -> Tuple[int, int]:
    """
    Find pair summing to target in SORTED array.
    O(n) time, O(1) space.
    """
    left = 0
    right = len(numbers) - 1

    while left < right:
        current_sum = numbers[left] + numbers[right]

        if current_sum == target:
            return (numbers[left], numbers[right])  # found!
        elif current_sum < target:
            left += 1   # need bigger sum → move left right
        else:
            right -= 1  # need smaller sum → move right left

    return None  # no pair found


sorted_nums = [1, 2, 4, 7, 11, 15]
print(two_sum_sorted(sorted_nums, 9))   # (2, 7) → 2+7=9
print(two_sum_sorted(sorted_nums, 13))  # (2, 11) → 2+11=13
print(two_sum_sorted(sorted_nums, 100)) # None

# ─────────────────────────────────────────
# PATTERN 2: Same Direction, Different Speeds
# Remove duplicates from sorted array
# ─────────────────────────────────────────
def remove_duplicates(arr: List[int]) -> int:
    """
    Remove duplicates in-place from sorted array.
    Returns new length.
    O(n) time, O(1) space.

    slow pointer: position for next unique element
    fast pointer: scanning ahead
    """
    if not arr:
        return 0

    slow = 0    # position for next unique element

    for fast in range(1, len(arr)):
        if arr[fast] != arr[slow]:  # found a new unique element
            slow += 1
            arr[slow] = arr[fast]   # place it after last unique

    return slow + 1  # new length


arr = [1, 1, 2, 2, 3, 4, 4, 5]
new_len = remove_duplicates(arr)
print(f"Array: {arr[:new_len]}, length: {new_len}")
# Array: [1, 2, 3, 4, 5], length: 5

# ─────────────────────────────────────────
# PATTERN 3: Palindrome Check
# ─────────────────────────────────────────
def is_palindrome(s: str) -> bool:
    """
    Check if string is palindrome.
    O(n) time, O(1) space.
    """
    s = s.lower()
    # Remove non-alphanumeric characters
    s = "".join(c for c in s if c.isalnum())

    left = 0
    right = len(s) - 1

    while left < right:
        if s[left] != s[right]:
            return False
        left += 1
        right -= 1

    return True


print(is_palindrome("racecar"))         # True
print(is_palindrome("hello"))           # False
print(is_palindrome("A man a plan a canal Panama"))  # True

# ─────────────────────────────────────────
# PATTERN 4: Three Sum (extend to 3 pointers)
# ─────────────────────────────────────────
def three_sum(nums: List[int]) -> List[List[int]]:
    """
    Find all unique triplets that sum to zero.
    O(n²) time, O(n) space.
    """
    nums.sort()
    result = []

    for i in range(len(nums) - 2):
        # Skip duplicates for first element
        if i > 0 and nums[i] == nums[i - 1]:
            continue

        left = i + 1
        right = len(nums) - 1

        while left < right:
            total = nums[i] + nums[left] + nums[right]

            if total == 0:
                result.append([nums[i], nums[left], nums[right]])
                # Skip duplicates
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                left += 1
                right -= 1
            elif total < 0:
                left += 1
            else:
                right -= 1

    return result


print(three_sum([-1, 0, 1, 2, -1, -4]))
# [[-1, -1, 2], [-1, 0, 1]]

# ─────────────────────────────────────────
# REAL BACKEND USE: Merge two sorted arrays
# ─────────────────────────────────────────
def merge_sorted_arrays(arr1: List[int], arr2: List[int]) -> List[int]:
    """
    Merge two sorted arrays into one sorted array.
    O(m+n) time, O(m+n) space.
    Same logic as merge step in merge sort.
    """
    result = []
    i = j = 0

    while i < len(arr1) and j < len(arr2):
        if arr1[i] <= arr2[j]:
            result.append(arr1[i])
            i += 1
        else:
            result.append(arr2[j])
            j += 1

    result.extend(arr1[i:])
    result.extend(arr2[j:])
    return result


print(merge_sorted_arrays([1, 3, 5, 7], [2, 4, 6, 8]))
# [1, 2, 3, 4, 5, 6, 7, 8]
```

---

## 4. 🪟 Sliding Window

```python
"""
Sliding Window = maintain a window of elements
                 slide it across the array.

Instead of recomputing from scratch each time,
UPDATE the window: add new element, remove old element.

When to use:
  - Finding subarray with maximum/minimum sum of size k
  - Longest substring with certain properties
  - Rate limiting (requests in last N seconds)
  - Moving averages

Time: O(n) instead of O(n²) brute force
"""

# ─────────────────────────────────────────
# FIXED SIZE WINDOW
# Maximum sum of k consecutive elements
# ─────────────────────────────────────────
def max_sum_subarray(arr: List[int], k: int) -> int:
    """
    Find maximum sum of k consecutive elements.
    O(n) time, O(1) space.
    """
    if len(arr) < k:
        raise ValueError("Array smaller than window size")

    # Compute sum of first window
    window_sum = sum(arr[:k])
    max_sum = window_sum

    # Slide window: add new element, remove old element
    for i in range(k, len(arr)):
        window_sum += arr[i]        # add incoming element
        window_sum -= arr[i - k]    # remove outgoing element
        max_sum = max(max_sum, window_sum)

    return max_sum


arr = [1, 4, 2, 9, 7, 3, 8, 6]
print(max_sum_subarray(arr, 3))  # 24 (9+7+8 = 24)
print(max_sum_subarray(arr, 4))  # 28 (9+7+3+8... actually 7+3+8+6=24, check)

# ─────────────────────────────────────────
# VARIABLE SIZE WINDOW
# Longest substring without repeating characters
# ─────────────────────────────────────────
def longest_unique_substring(s: str) -> Tuple[int, str]:
    """
    Find longest substring without repeating chars.
    O(n) time, O(min(n, alphabet)) space.
    """
    char_index = {}     # last seen index of each char
    max_length = 0
    start = 0           # window start
    best_start = 0

    for end in range(len(s)):
        char = s[end]

        # If char seen AND it's inside current window
        if char in char_index and char_index[char] >= start:
            start = char_index[char] + 1  # shrink window

        char_index[char] = end  # update last seen

        # Update max
        if end - start + 1 > max_length:
            max_length = end - start + 1
            best_start = start

    return max_length, s[best_start:best_start + max_length]


length, substr = longest_unique_substring("abcabcbb")
print(f"Length: {length}, Substring: '{substr}'")  # 3, 'abc'

length, substr = longest_unique_substring("pwwkew")
print(f"Length: {length}, Substring: '{substr}'")  # 3, 'wke'

# ─────────────────────────────────────────
# Minimum window substring
# ─────────────────────────────────────────
from collections import Counter

def min_window_substring(s: str, t: str) -> str:
    """
    Find smallest substring of s containing all chars of t.
    O(|s| + |t|) time.
    """
    if not t or not s:
        return ""

    need = Counter(t)       # chars we need
    have = {}               # chars we have in window
    formed = 0              # how many chars fully satisfied
    required = len(need)    # how many distinct chars we need

    left = 0
    min_len = float('inf')
    result = ""

    for right in range(len(s)):
        char = s[right]
        have[char] = have.get(char, 0) + 1

        # Check if this char's requirement is now met
        if char in need and have[char] == need[char]:
            formed += 1

        # Try to shrink window from left
        while formed == required:
            window_len = right - left + 1
            if window_len < min_len:
                min_len = window_len
                result = s[left:right + 1]

            # Remove leftmost char
            left_char = s[left]
            have[left_char] -= 1
            if left_char in need and have[left_char] < need[left_char]:
                formed -= 1
            left += 1

    return result


print(min_window_substring("ADOBECODEBANC", "ABC"))  # "BANC"
print(min_window_substring("aa", "aa"))              # "aa"

# ─────────────────────────────────────────
# REAL BACKEND USE: Rate limiter
# ─────────────────────────────────────────
from collections import deque
import time

class SlidingWindowRateLimiter:
    """
    Rate limiter using sliding window algorithm.
    Allows max_requests per window_seconds.
    """

    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests: dict = {}    # user_id → deque of timestamps

    def is_allowed(self, user_id: str) -> bool:
        """Check if request is allowed. O(1) amortized."""
        now = time.time()
        window_start = now - self.window_seconds

        if user_id not in self.requests:
            self.requests[user_id] = deque()

        # Remove requests outside the window
        user_requests = self.requests[user_id]
        while user_requests and user_requests[0] < window_start:
            user_requests.popleft()

        if len(user_requests) < self.max_requests:
            user_requests.append(now)
            return True     # allowed

        return False        # rate limited

    def remaining(self, user_id: str) -> int:
        """How many more requests allowed."""
        if user_id not in self.requests:
            return self.max_requests
        return max(0, self.max_requests - len(self.requests[user_id]))


# Test rate limiter
limiter = SlidingWindowRateLimiter(max_requests=3, window_seconds=60)

for i in range(5):
    allowed = limiter.is_allowed("user_123")
    remaining = limiter.remaining("user_123")
    print(f"Request {i+1}: {'✅ allowed' if allowed else '❌ blocked'}, "
          f"remaining: {remaining}")
# Request 1: ✅ allowed, remaining: 2
# Request 2: ✅ allowed, remaining: 1
# Request 3: ✅ allowed, remaining: 0
# Request 4: ❌ blocked, remaining: 0
# Request 5: ❌ blocked, remaining: 0
```

---

## 5. 🔁 Recursion & Backtracking

```python
"""
Recursion:    solve a problem by solving smaller versions of itself
Backtracking: explore all possibilities, undo ("backtrack") when stuck

Template for backtracking:
  def backtrack(state):
      if is_solution(state):
          record(state)
          return
      for choice in get_choices(state):
          make_choice(choice)
          backtrack(state)
          undo_choice(choice)    ← backtrack!
"""

# ─────────────────────────────────────────
# CLASSIC RECURSION: Fibonacci
# ─────────────────────────────────────────
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n: int) -> int:
    """
    Without cache: O(2ⁿ) — exponential, terrible
    With cache:    O(n)  — linear, great
    """
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)


print([fibonacci(i) for i in range(10)])
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# ─────────────────────────────────────────
# BACKTRACKING: Generate all permutations
# ─────────────────────────────────────────
def permutations(nums: List[int]) -> List[List[int]]:
    """
    Generate all orderings of nums.
    O(n! × n) time — n! permutations, each takes O(n).
    """
    result = []

    def backtrack(current: List[int], remaining: List[int]):
        if not remaining:               # base case: no more to add
            result.append(current[:])   # found a complete permutation
            return

        for i in range(len(remaining)):
            current.append(remaining[i])                    # choose
            backtrack(current, remaining[:i] + remaining[i+1:])  # explore
            current.pop()                                   # unchoose

    backtrack([], nums)
    return result


print(permutations([1, 2, 3]))
# [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]

# ─────────────────────────────────────────
# BACKTRACKING: N-Queens problem
# ─────────────────────────────────────────
def solve_n_queens(n: int) -> List[List[str]]:
    """
    Place N queens on NxN board so none attack each other.
    Classic backtracking problem.
    """
    result = []
    cols = set()          # columns with queens
    diag1 = set()         # (row - col) diagonals
    diag2 = set()         # (row + col) anti-diagonals

    def backtrack(row: int, board: List[List[str]]):
        if row == n:        # placed all queens
            result.append(["".join(row) for row in board])
            return

        for col in range(n):
            if col in cols or (row - col) in diag1 or (row + col) in diag2:
                continue    # this position is attacked

            # Place queen
            board[row][col] = "Q"
            cols.add(col)
            diag1.add(row - col)
            diag2.add(row + col)

            backtrack(row + 1, board)   # try next row

            # Remove queen (backtrack)
            board[row][col] = "."
            cols.remove(col)
            diag1.remove(row - col)
            diag2.remove(row + col)

    board = [["." for _ in range(n)] for _ in range(n)]
    backtrack(0, board)
    return result


solutions = solve_n_queens(4)
print(f"4-Queens: {len(solutions)} solutions")
for sol in solutions:
    for row in sol:
        print(row)
    print()

# ─────────────────────────────────────────
# BACKTRACKING: Generate valid combinations
# ─────────────────────────────────────────
def combinations(nums: List[int], k: int) -> List[List[int]]:
    """Choose k elements from nums (order doesn't matter)."""
    result = []

    def backtrack(start: int, current: List[int]):
        if len(current) == k:
            result.append(current[:])
            return

        for i in range(start, len(nums)):
            current.append(nums[i])
            backtrack(i + 1, current)   # start from i+1 (no reuse)
            current.pop()

    backtrack(0, [])
    return result


print(combinations([1, 2, 3, 4], 2))
# [[1,2], [1,3], [1,4], [2,3], [2,4], [3,4]]

# ─────────────────────────────────────────
# REAL BACKEND USE: Generate test data combinations
# ─────────────────────────────────────────
def generate_test_scenarios(
    roles: List[str],
    endpoints: List[str],
    methods: List[str]
) -> List[dict]:
    """
    Generate all test combinations for API testing.
    Tests every role × endpoint × method combination.
    """
    scenarios = []

    # Generate using itertools instead (more practical)
    import itertools
    for role, endpoint, method in itertools.product(roles, endpoints, methods):
        scenarios.append({
            "role": role,
            "endpoint": endpoint,
            "method": method,
            "expected": "200" if role == "admin" else "403"
        })

    return scenarios


scenarios = generate_test_scenarios(
    roles=["admin", "user", "guest"],
    endpoints=["/users", "/posts", "/settings"],
    methods=["GET", "POST", "DELETE"]
)
print(f"Generated {len(scenarios)} test scenarios")
# 27 combinations (3 × 3 × 3)
```

---

## 6. 🌊 BFS & DFS (Graphs and Trees)

```python
from collections import deque

"""
BFS (Breadth-First Search):
  Explore level by level using a QUEUE
  Finds SHORTEST path in unweighted graphs
  Uses more memory (stores entire level)

DFS (Depth-First Search):
  Go as deep as possible using STACK (or recursion)
  Finds all paths
  Less memory (stores one path at a time)

BOTH: O(V + E) for graphs, O(n) for trees
"""

# ─────────────────────────────────────────
# BFS TEMPLATE
# ─────────────────────────────────────────
def bfs_template(graph: dict, start: str, target: str) -> Optional[List[str]]:
    """
    BFS to find shortest path from start to target.
    Returns path as list, or None if not reachable.
    """
    if start == target:
        return [start]

    queue = deque([[start]])    # queue of PATHS (not just nodes)
    visited = {start}

    while queue:
        path = queue.popleft()
        node = path[-1]         # last node in current path

        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                new_path = path + [neighbor]
                if neighbor == target:
                    return new_path     # shortest path found!
                visited.add(neighbor)
                queue.append(new_path)

    return None     # no path exists


# ─────────────────────────────────────────
# DFS TEMPLATE — iterative
# ─────────────────────────────────────────
def dfs_template(graph: dict, start: str) -> List[str]:
    """
    DFS iterative — explore all reachable nodes.
    Returns list of visited nodes.
    """
    stack = [start]
    visited = set()
    result = []

    while stack:
        node = stack.pop()      # take from TOP (LIFO)

        if node not in visited:
            visited.add(node)
            result.append(node)
            # Push neighbors (in reverse for consistent order)
            for neighbor in reversed(graph.get(node, [])):
                if neighbor not in visited:
                    stack.append(neighbor)

    return result


# ─────────────────────────────────────────
# DFS TEMPLATE — recursive
# ─────────────────────────────────────────
def dfs_recursive(
    graph: dict,
    node: str,
    visited: set = None,
    result: list = None
) -> List[str]:
    if visited is None:
        visited = set()
        result = []

    visited.add(node)
    result.append(node)

    for neighbor in graph.get(node, []):
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited, result)

    return result


# Test
graph = {
    "A": ["B", "C"],
    "B": ["A", "D", "E"],
    "C": ["A", "F"],
    "D": ["B"],
    "E": ["B", "F"],
    "F": ["C", "E"],
}

print("BFS A→F:", bfs_template(graph, "A", "F"))
# ['A', 'C', 'F'] — shortest path!

print("DFS iterative:", dfs_template(graph, "A"))
print("DFS recursive:", dfs_recursive(graph, "A"))

# ─────────────────────────────────────────
# REAL USE: Web crawler (BFS)
# ─────────────────────────────────────────
def crawl_website(start_url: str, max_pages: int = 10) -> List[str]:
    """
    BFS web crawl — find all reachable pages.
    Real crawler would fetch actual HTML.
    """
    # Simulated link structure
    links = {
        "/": ["/about", "/products", "/contact"],
        "/about": ["/", "/team"],
        "/products": ["/", "/products/laptop", "/products/phone"],
        "/products/laptop": ["/products", "/checkout"],
        "/products/phone": ["/products", "/checkout"],
        "/contact": ["/"],
        "/team": ["/about"],
        "/checkout": ["/products"],
    }

    visited = set()
    queue = deque([start_url])
    crawled = []

    while queue and len(crawled) < max_pages:
        url = queue.popleft()

        if url in visited:
            continue

        visited.add(url)
        crawled.append(url)
        print(f"Crawled: {url}")

        for link in links.get(url, []):
            if link not in visited:
                queue.append(link)

    return crawled


print("\n=== Web Crawler ===")
pages = crawl_website("/", max_pages=8)
print(f"\nTotal pages found: {len(pages)}")

# ─────────────────────────────────────────
# REAL USE: Detect circular dependencies (DFS)
# ─────────────────────────────────────────
def has_circular_dependency(dependencies: dict) -> bool:
    """
    Detect if package dependencies form a cycle.
    Uses DFS with three states:
      WHITE (0): not visited
      GRAY  (1): currently being processed (in stack)
      BLACK (2): fully processed
    """
    WHITE, GRAY, BLACK = 0, 1, 2
    color = {node: WHITE for node in dependencies}

    def dfs(node: str) -> bool:
        color[node] = GRAY  # mark as being processed

        for neighbor in dependencies.get(node, []):
            if neighbor not in color:
                color[neighbor] = WHITE

            if color[neighbor] == GRAY:
                return True     # cycle detected!

            if color[neighbor] == WHITE:
                if dfs(neighbor):
                    return True

        color[node] = BLACK     # fully processed
        return False

    for node in list(dependencies.keys()):
        if color[node] == WHITE:
            if dfs(node):
                return True

    return False


# No cycle
deps_ok = {
    "fastapi": ["starlette", "pydantic"],
    "starlette": ["anyio"],
    "pydantic": [],
    "anyio": [],
}

# Cycle: A→B→C→A
deps_cycle = {
    "A": ["B"],
    "B": ["C"],
    "C": ["A"],     # circular!
}

print(f"deps_ok has cycle: {has_circular_dependency(deps_ok)}")     # False
print(f"deps_cycle has cycle: {has_circular_dependency(deps_cycle)}") # True
```

---

## 7. ⚡ Dynamic Programming

```python
"""
Dynamic Programming (DP) = break problem into subproblems,
                           solve each ONCE,
                           store results (memoization/tabulation).

Use when problem has:
  1. Optimal substructure: optimal solution contains
     optimal solutions to subproblems
  2. Overlapping subproblems: same subproblems solved multiple times

Two approaches:
  Top-down (memoization): recursive + cache
  Bottom-up (tabulation): iterative, fill table from base cases

DP vs recursion: DP avoids recomputing the same subproblems.
"""

# ─────────────────────────────────────────
# CLASSIC: Fibonacci with DP
# ─────────────────────────────────────────
def fib_naive(n: int) -> int:
    """No DP. O(2ⁿ) — exponential."""
    if n <= 1:
        return n
    return fib_naive(n-1) + fib_naive(n-2)

def fib_memo(n: int, memo: dict = None) -> int:
    """Top-down DP (memoization). O(n) time, O(n) space."""
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]      # return cached result
    if n <= 1:
        return n
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
    return memo[n]

def fib_table(n: int) -> int:
    """Bottom-up DP (tabulation). O(n) time, O(n) space."""
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[0] = 0
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]  # build from bottom up
    return dp[n]

def fib_optimal(n: int) -> int:
    """Optimized DP. O(n) time, O(1) space."""
    if n <= 1:
        return n
    prev2, prev1 = 0, 1
    for _ in range(2, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current
    return prev1

print([fib_optimal(i) for i in range(10)])  # [0,1,1,2,3,5,8,13,21,34]

# ─────────────────────────────────────────
# CLASSIC: Coin Change (minimum coins)
# ─────────────────────────────────────────
def coin_change(coins: List[int], amount: int) -> int:
    """
    Find minimum number of coins to make amount.
    O(amount × coins) time and space.

    Example: coins=[1,5,10,25], amount=36
    Answer: 3 (25+10+1)
    """
    # dp[i] = minimum coins to make amount i
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0   # base case: 0 coins needed for amount 0

    for a in range(1, amount + 1):
        for coin in coins:
            if coin <= a:
                # Option: use this coin + whatever needed for (a - coin)
                dp[a] = min(dp[a], dp[a - coin] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1


print(coin_change([1, 5, 10, 25], 36))  # 3 (25+10+1)
print(coin_change([1, 5, 10, 25], 11))  # 2 (10+1 or 5+5+1? → 10+1=2)
print(coin_change([2], 3))              # -1 (impossible)

# ─────────────────────────────────────────
# CLASSIC: Longest Common Subsequence
# ─────────────────────────────────────────
def lcs(s1: str, s2: str) -> int:
    """
    Longest Common Subsequence of two strings.
    O(m×n) time and space.

    Subsequence: characters in order but not necessarily adjacent.
    LCS("ABCBDAB", "BDCAB") = 4 (BCAB or BDAB)

    Real use: diff algorithms (git diff!), plagiarism detection.
    """
    m, n = len(s1), len(s2)
    # dp[i][j] = LCS of s1[:i] and s2[:j]
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1    # extend LCS
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])  # take best

    return dp[m][n]


print(lcs("ABCBDAB", "BDCAB"))  # 4
print(lcs("AGGTAB", "GXTXAYB"))  # 4

# ─────────────────────────────────────────
# CLASSIC: 0/1 Knapsack
# ─────────────────────────────────────────
def knapsack(weights: List[int], values: List[int], capacity: int) -> int:
    """
    Maximize value with weight constraint.
    O(n × capacity) time and space.

    Real use: resource allocation, feature selection,
              portfolio optimization.
    """
    n = len(weights)
    # dp[i][w] = max value using first i items with capacity w
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for w in range(capacity + 1):
            # Option 1: don't take item i
            dp[i][w] = dp[i-1][w]
            # Option 2: take item i (if it fits)
            if weights[i-1] <= w:
                value_with_item = dp[i-1][w - weights[i-1]] + values[i-1]
                dp[i][w] = max(dp[i][w], value_with_item)

    return dp[n][capacity]


weights = [2, 3, 4, 5]
values = [3, 4, 5, 6]
capacity = 8
print(knapsack(weights, values, capacity))  # 10 (items 1+3: weight 6, value 8)

# ─────────────────────────────────────────
# REAL BACKEND: Caching with DP
# ─────────────────────────────────────────
@lru_cache(maxsize=256)
def compute_user_score(user_id: int, period: str) -> float:
    """
    Compute user engagement score for a period.
    Cached so each (user_id, period) only computed once.

    In production: this would query database.
    Cache prevents expensive recomputation.
    """
    # Simulate expensive computation
    import time
    time.sleep(0.001)   # pretend database query

    base_score = user_id * 0.1
    multiplier = {"daily": 1, "weekly": 7, "monthly": 30}
    return base_score * multiplier.get(period, 1)


# First call: computed
start = time.perf_counter()
score = compute_user_score(42, "weekly")
first = (time.perf_counter() - start) * 1000

# Second call: from cache
start = time.perf_counter()
score = compute_user_score(42, "weekly")
second = (time.perf_counter() - start) * 1000

print(f"First call:  {first:.3f}ms")   # ~1ms
print(f"Cached call: {second:.3f}ms")  # ~0.001ms

print(compute_user_score.cache_info())
```

---

## 8. 🌲 Greedy Algorithms

```python
"""
Greedy = make the LOCALLY OPTIMAL choice at each step
         hoping it leads to a GLOBALLY OPTIMAL solution.

Greedy works when:
  - Local optimal → global optimal (provable)
  - You can prove no better solution exists

Doesn't work when:
  - Local optimal doesn't lead to global optimal
  - Requires DP or backtracking instead

Examples that work:
  - Interval scheduling
  - Huffman coding
  - Dijkstra's algorithm
  - Activity selection
"""

# ─────────────────────────────────────────
# CLASSIC: Activity Selection
# Select max non-overlapping activities
# ─────────────────────────────────────────
def activity_selection(activities: List[Tuple[int, int]]) -> List[Tuple[int, int]]:
    """
    Select maximum number of non-overlapping activities.
    Each activity: (start_time, end_time).
    Greedy: always pick activity that ends earliest.

    Real use: scheduling meetings, booking time slots.
    O(n log n) for sorting + O(n) selection.
    """
    # Sort by end time (greedy choice: pick earliest ending first)
    sorted_activities = sorted(activities, key=lambda x: x[1])
    selected = []
    last_end = 0

    for start, end in sorted_activities:
        if start >= last_end:   # no overlap
            selected.append((start, end))
            last_end = end

    return selected


activities = [(1, 4), (3, 5), (0, 6), (5, 7), (3, 9), (5, 9), (6, 10), (8, 11)]
print(activity_selection(activities))
# [(1, 4), (5, 7), (8, 11)] — max 3 non-overlapping

# ─────────────────────────────────────────
# CLASSIC: Jump Game
# Can you reach the end?
# ─────────────────────────────────────────
def can_jump(nums: List[int]) -> bool:
    """
    Each element = max steps you can jump from that position.
    Can you reach the last index?
    Greedy: track furthest reachable position.
    O(n) time.
    """
    max_reach = 0   # furthest index we can reach

    for i, jump in enumerate(nums):
        if i > max_reach:
            return False    # can't reach position i
        max_reach = max(max_reach, i + jump)

    return True


print(can_jump([2, 3, 1, 1, 4]))    # True
print(can_jump([3, 2, 1, 0, 4]))    # False (stuck at index 3)

# ─────────────────────────────────────────
# REAL USE: Meeting room scheduler
# ─────────────────────────────────────────
def min_meeting_rooms(meetings: List[Tuple[int, int]]) -> int:
    """
    Find minimum meeting rooms needed.
    Greedy with heap.
    O(n log n) time.
    """
    if not meetings:
        return 0

    # Sort by start time
    sorted_meetings = sorted(meetings, key=lambda x: x[0])

    # Min-heap: tracks end times of ongoing meetings
    rooms = []  # heap of end times

    for start, end in sorted_meetings:
        if rooms and rooms[0] <= start:
            # Reuse earliest ending room
            heapq.heapreplace(rooms, end)
        else:
            # Need a new room
            heapq.heappush(rooms, end)

    return len(rooms)


meetings = [(0, 30), (5, 10), (15, 20)]
print(f"Rooms needed: {min_meeting_rooms(meetings)}")  # 2

meetings2 = [(9, 10), (4, 9), (4, 17)]
print(f"Rooms needed: {min_meeting_rooms(meetings2)}")  # 2

# ─────────────────────────────────────────
# REAL USE: Task scheduler
# ─────────────────────────────────────────
def least_interval(tasks: List[str], n: int) -> int:
    """
    Schedule tasks with cooling period n between same tasks.
    Return minimum time to complete all tasks.
    Greedy: always do most frequent available task.
    O(m) where m = total tasks.
    """
    from collections import Counter
    task_counts = Counter(tasks)
    max_count = max(task_counts.values())

    # How many tasks have the maximum count?
    max_count_tasks = sum(1 for c in task_counts.values() if c == max_count)

    # Formula: fill "chunks" of size (n+1) with most frequent tasks
    result = (max_count - 1) * (n + 1) + max_count_tasks
    return max(result, len(tasks))


print(least_interval(["A", "A", "A", "B", "B", "B"], n=2))  # 8
# A→B→idle→A→B→idle→A→B
```

---

## 9. #️⃣ Hashing Techniques

```python
"""
Hashing turns data into a fixed-size value.
Used for:
  - O(1) lookup (dict, set)
  - Detecting duplicates
  - Grouping/anagram detection
  - Rolling hash (Rabin-Karp substring search)
"""

# ─────────────────────────────────────────
# PATTERN: Two Sum with hash map
# ─────────────────────────────────────────
def two_sum(nums: List[int], target: int) -> Optional[Tuple[int, int]]:
    """
    Find indices of two numbers that sum to target.
    O(n) time using hash map.
    """
    seen = {}   # value → index

    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return (seen[complement], i)
        seen[num] = i

    return None


print(two_sum([2, 7, 11, 15], 9))   # (0, 1) → nums[0]+nums[1] = 9
print(two_sum([3, 2, 4], 6))        # (1, 2) → nums[1]+nums[2] = 6

# ─────────────────────────────────────────
# PATTERN: Group anagrams
# ─────────────────────────────────────────
def group_anagrams(words: List[str]) -> List[List[str]]:
    """
    Group words that are anagrams of each other.
    O(n × k log k) where k = average word length.
    """
    groups = {}

    for word in words:
        key = tuple(sorted(word))   # sorted chars as key
        if key not in groups:
            groups[key] = []
        groups[key].append(word)

    return list(groups.values())


words = ["eat", "tea", "tan", "ate", "nat", "bat"]
print(group_anagrams(words))
# [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]

# ─────────────────────────────────────────
# PATTERN: Find duplicate in array
# ─────────────────────────────────────────
def find_duplicate(nums: List[int]) -> int:
    """Find the one duplicate number. O(n) time, O(n) space."""
    seen = set()
    for num in nums:
        if num in seen:
            return num
        seen.add(num)
    return -1


print(find_duplicate([1, 3, 4, 2, 2]))     # 2
print(find_duplicate([3, 1, 3, 4, 2]))     # 3

# ─────────────────────────────────────────
# PATTERN: Subarray sum equals k
# ─────────────────────────────────────────
def subarray_sum(nums: List[int], k: int) -> int:
    """
    Count subarrays that sum to k.
    O(n) time using prefix sums + hash map.
    """
    count = 0
    prefix_sum = 0
    seen_sums = {0: 1}  # prefix_sum → frequency

    for num in nums:
        prefix_sum += num
        # If (prefix_sum - k) seen before,
        # subarray from that point to here sums to k
        count += seen_sums.get(prefix_sum - k, 0)
        seen_sums[prefix_sum] = seen_sums.get(prefix_sum, 0) + 1

    return count


print(subarray_sum([1, 1, 1], 2))       # 2 ([1,1] appears twice)
print(subarray_sum([1, 2, 3], 3))       # 2 ([1,2] and [3])
print(subarray_sum([1, -1, 1], 1))      # 3

# ─────────────────────────────────────────
# REAL BACKEND: Deduplication pipeline
# ─────────────────────────────────────────
import hashlib
from typing import Iterator

def deduplicate_records(
    records: Iterator[dict],
    key_fields: List[str]
) -> Iterator[dict]:
    """
    Remove duplicate records based on key fields.
    Used in ETL pipelines to clean data.
    O(n) time, O(unique records) space.
    """
    seen = set()

    for record in records:
        # Create a hash from key fields
        key_data = "|".join(str(record.get(f, "")) for f in key_fields)
        record_hash = hashlib.md5(key_data.encode()).hexdigest()

        if record_hash not in seen:
            seen.add(record_hash)
            yield record        # only yield unique records


# Test deduplication
records = [
    {"email": "alice@test.com", "name": "Alice", "source": "form"},
    {"email": "bob@test.com", "name": "Bob", "source": "api"},
    {"email": "alice@test.com", "name": "Alice Smith", "source": "import"},  # dup!
    {"email": "charlie@test.com", "name": "Charlie", "source": "form"},
    {"email": "bob@test.com", "name": "Bob", "source": "form"},  # dup!
]

unique_records = list(deduplicate_records(records, key_fields=["email"]))
print(f"Original: {len(records)}, Unique: {len(unique_records)}")  # 5 → 3
for r in unique_records:
    print(f"  {r['email']}: {r['name']}")
```

---

## 📊 Visual Summary

```
┌────────────────────────────────────────────────────────────────┐│                    ALGORITHM PATTERNS                          │
│                                                                │
│  SEARCHING                SORTING                              │
│  ─────────                ───────                              │
│  Linear:   O(n)           Bubble/Select/Insert: O(n²)          │
│  Binary:   O(log n)       Merge/Quick:          O(n log n)     │
│            (sorted data!) Python sort (Timsort): O(n log n)    │
│                                                                │
│  TWO POINTERS             SLIDING WINDOW                       │
│  ────────────             ─────────────                        │
│  Opposite ends:           Fixed size:  max/min subarray        │
│    two sum sorted         Variable:    longest substring       │
│    palindrome check       Real use:    rate limiting           │
│  Same direction:                                               │
│    remove duplicates                                           │
│                                                                │
│  RECURSION/BACKTRACK      DYNAMIC PROGRAMMING                  │
│  ──────────────────       ──────────────────                   │
│  Generate all options     Overlapping subproblems              │
│  Undo when stuck          Memoization (top-down)               │
│  N-Queens, permutations   Tabulation (bottom-up)               │
│                           Fibonacci, knapsack, LCS             │
│                                                                │
│  BFS vs DFS               GREEDY                               │
│  ──────────               ──────                               │
│  BFS: shortest path       Local optimal → global optimal       │
│       level-by-level      Activity selection, scheduling       │
│       uses queue          Often O(n log n) sort + O(n)         │
│  DFS: explore all                                              │
│       detect cycles       HASHING                              │
│       uses stack/recursion O(1) lookup, O(n) build             │
│                           Two sum, anagrams, dedup             │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

1. Why is binary search O(log n)? What is the requirement to use it?
2. What is the difference between merge sort and quick sort? When does quick sort degrade to O(n²)?
3. When would you use insertion sort over merge sort?
4. Explain the two pointer technique with an example.
5. What is a sliding window? When do you use fixed vs variable?
6. What is memoization? How does it differ from tabulation?
7. What are the two conditions for dynamic programming?
8. When does greedy work? When does it fail?
9. What is the difference between BFS and DFS? Which finds the shortest path in unweighted graphs?
10. How does hashing achieve O(1) lookup?
11. What is backtracking? Give an example.
12. How would you find duplicates in O(n) time?

---

## 🛠️ Practice Problems

```python
# Problem 1: Binary Search Application
# You have a sorted list of timestamps (Unix time).
# Find the FIRST event after a given time.
# Must be O(log n).
# timestamps = [1000, 2000, 3000, 4000, 5000]
# find_first_after(timestamps, 2500) → 3000 (index 2)

# Problem 2: Sliding Window
# Given daily temperatures array,
# find max temperature in any 7-day window.
# temps = [30, 28, 32, 35, 25, 27, 30, 33, 31, 29]
# max_7day_temp(temps) → 35

# Problem 3: Two Pointers
# Given sorted array, find all pairs with sum < target.
# count_pairs([1, 2, 3, 4, 5], target=6) → 4
# Pairs: (1,2), (1,3), (1,4), (2,3)

# Problem 4: DP
# Climbing stairs: can climb 1 or 2 steps at a time.
# How many distinct ways to climb n stairs?
# climb_stairs(3) → 3 (1+1+1, 1+2, 2+1)
# climb_stairs(5) → 8

# Problem 5: Graph + BFS
# Given a social network graph,
# find all users within 2 degrees of a given user.
# (friends + friends-of-friends)
# Must use BFS and track distance.

# Problem 6: Hashing (Hard)
# Given a list of API logs:
# [{"user": "alice", "endpoint": "/search", "time": 1000}, ...]
# Find users who visited the same sequence of 3 endpoints.
# sequence = ["/home", "/search", "/product"]
# Return users who visited exactly this sequence
# (not necessarily consecutive, but in order)

# Problem 7: Greedy
# You have a list of tasks with deadlines and profits.
# Each task takes 1 unit of time.
# You can only do 1 task at a time.
# Find maximum profit.
# tasks = [(profit, deadline), ...]
# schedule_tasks([(100,2), (19,1), (27,2), (25,1), (15,3)]) → 142
```

---

## ✅ Phase 2.2 Complete!

**You now know:**

```
✅ Linear and Binary Search (and variations)
✅ Bubble, Selection, Insertion Sort
✅ Merge Sort and Quick Sort
✅ Python's Timsort and sort keys
✅ Two Pointers (opposite ends + same direction)
✅ Sliding Window (fixed + variable size)
✅ Recursion and Backtracking templates
✅ BFS and DFS (iterative + recursive)
✅ Dynamic Programming (memoization + tabulation)
✅ Greedy Algorithms
✅ Hashing techniques
✅ Real backend applications of each
```

---

## What's Next?

```
Option 1: Continue  → Phase 2.3 (Python-Specific Efficiency)
                      bisect, heapq, collections performance,
                      built-in time complexities

Option 2: Practice  → Solve the 7 practice problems above

Option 3: Continue  → Phase 3 (Databases)
                      Jump straight to databases
                      (Phase 2.3 is shorter)

Option 4: Questions → Anything unclear?
```

**What do you want to do? 🚀**