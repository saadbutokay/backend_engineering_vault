Why do backend engineers need data structures?
```
Every day you will:
  - Store 1 million users in memory → which structure?
  - Find a user by ID in microseconds → which structure?
  - Maintain a queue of background jobs → which structure?
  - Find the shortest path in a graph → which structure?
  - Auto-complete search suggestions → which structure?

The WRONG data structure:
  → Your API takes 10 seconds instead of 10ms
  → Your server runs out of memory
  → Users complain, you get fired

The RIGHT data structure:
  → Instant responses
  → Minimal memory
  → Happy users, happy you
```

---
## Big O Notation
**Before we touch a single data structure, you need this.**
```
Big O = how does performance SCALE as data grows?

Not "how fast is it on 10 items?"
But "what happens when you have 10 MILLION items?"

O(1)      = constant    → always same speed, regardless of size
O(log n)  = logarithmic → slightly slower as n grows (very good)
O(n)      = linear      → proportionally slower as n grows (ok)
O(n log n)= linearithmic→ common in good sorting algorithms
O(n²)     = quadratic   → much slower as n grows (bad)
O(2ⁿ)     = exponential → unusable for large n (terrible)
```

**Visualizing growth:**

```
n = number of items

n          O(1)    O(log n)   O(n)    O(n²)      O(2ⁿ)
─────────────────────────────────────────────────────────
1          1       1          1       1          2
10         1       3          10      100        1,024
100        1       7          100     10,000     1.27×10³⁰
1,000      1       10         1,000   1,000,000  too large
1,000,000  1       20         1M      1 trillion  💀

If O(n²) takes 1 second with 1,000 items:
  With 10,000 items → 100 seconds
  With 100,000 items → 10,000 seconds (almost 3 hours!)

If O(log n) takes 1 second with 1,000 items:
  With 10,000 items → 1.3 seconds
  With 100,000 items → 1.7 seconds
  With 1,000,000 items → 2 seconds

THIS is why data structures matter.
```

---

## Setup

```bash
mkdir ~/projects/data_structures
cd ~/projects/data_structures
python3 -m venv venv
source venv/bin/activate
touch ds.py
code .
```

---

## 1. Arrays / Lists (Dynamic Arrays)

### What is an Array?

```
An array stores elements in CONTIGUOUS memory locations.
Each element sits right next to the previous one.

Memory layout:
Index:   0    1    2    3    4
Value: [10] [20] [30] [40] [50]
Addr:  100  104  108  112  116  ← each int = 4 bytes

Why contiguous?
  To access index 3:
  address = base_address + (index × element_size)
  address = 100 + (3 × 4) = 112
  → Direct calculation, no searching needed
  → O(1) access time
```

### Python Lists ARE Dynamic Arrays

```python
# ds.py

import sys
import time

# Python list = dynamic array under the hood
# "Dynamic" = automatically resizes when needed

# ─────────────────────────────────────────
# HOW DYNAMIC ARRAYS WORK INTERNALLY
# ─────────────────────────────────────────

# Python over-allocates to avoid resizing every append
my_list = []
prev_size = 0

print("Size growth as we append:")
print(f"{'Items':>6} {'Allocated Slots':>16} {'Memory (bytes)':>15}")
print("-" * 42)

for i in range(20):
    my_list.append(i)
    allocated = my_list.__sizeof__()    # internal size
    if allocated != prev_size:
        print(f"{len(my_list):>6} {allocated:>16} {sys.getsizeof(my_list):>15}")
        prev_size = allocated

# Output shows Python allocates extra slots
# When list is [1]:        allocates 4 slots
# When list is [1,2,3,4,5]: allocates 8 slots
# Growth factor ≈ 1.125x (amortized O(1) append)
```

### Time Complexity of List Operations

```python
# ─────────────────────────────────────────
# O(1) OPERATIONS — constant time
# ─────────────────────────────────────────

items = [10, 20, 30, 40, 50]

# Access by index → O(1)
# Direct memory calculation, no traversal
first = items[0]        # O(1)
last = items[-1]        # O(1)
middle = items[2]       # O(1)

# Append to end → O(1) amortized
# (occasionally O(n) when resizing, but rare)
items.append(60)        # O(1)

# Pop from end → O(1)
items.pop()             # O(1)

# Length → O(1)
# Python stores length separately
length = len(items)     # O(1)

# ─────────────────────────────────────────
# O(n) OPERATIONS — linear time
# ─────────────────────────────────────────

# Search by value → O(n)
# Must check each element until found
found = 30 in items     # O(n) — worst case: check all n items
index = items.index(30) # O(n)

# Insert at beginning → O(n)
# Must shift ALL existing elements right
items.insert(0, 0)      # O(n) — shifts every element

# Delete from beginning → O(n)
# Must shift ALL remaining elements left
items.pop(0)            # O(n) — shifts every element

# Delete by value → O(n)
items.remove(30)        # O(n) — search + shift

# Slicing → O(k) where k = slice size
sub = items[1:4]        # O(k)
```

### Demonstrating the Difference

```python
def measure_time(func, *args):
    start = time.perf_counter()
    result = func(*args)
    return (time.perf_counter() - start) * 1000

# Build a large list
large = list(range(1_000_000))

# O(1) vs O(n) comparison
t1 = measure_time(lambda: large[-1])        # O(1) — access last
t2 = measure_time(lambda: 999999 in large)  # O(n) — search

print(f"O(1) access last element: {t1:.4f}ms")
print(f"O(n) search for 999999:   {t2:.4f}ms")
# O(n) is significantly slower!
```

### Implementing a Dynamic Array from Scratch

```python
class DynamicArray:
    """
    Build a dynamic array from scratch.
    Shows exactly how Python's list works internally.
    """

    def __init__(self):
        self._capacity = 4          # initial allocated slots
        self._size = 0              # actual number of items
        self._data = [None] * self._capacity

    def __len__(self) -> int:
        return self._size

    def __getitem__(self, index: int):
        if not (-self._size <= index < self._size):
            raise IndexError(f"Index {index} out of range")
        if index < 0:
            index += self._size
        return self._data[index]

    def __setitem__(self, index: int, value):
        if not (0 <= index < self._size):
            raise IndexError(f"Index {index} out of range")
        self._data[index] = value

    def __repr__(self) -> str:
        items = [str(self._data[i]) for i in range(self._size)]
        return f"DynamicArray([{', '.join(items)}])"

    def append(self, value) -> None:
        """Add item to end. O(1) amortized."""
        if self._size == self._capacity:
            self._resize()  # double capacity when full
        self._data[self._size] = value
        self._size += 1

    def insert(self, index: int, value) -> None:
        """Insert at index. O(n)."""
        if not (0 <= index <= self._size):
            raise IndexError(f"Index {index} out of range")
        if self._size == self._capacity:
            self._resize()
        # Shift elements right
        for i in range(self._size, index, -1):
            self._data[i] = self._data[i - 1]
        self._data[index] = value
        self._size += 1

    def remove_at(self, index: int):
        """Remove item at index. O(n)."""
        if not (0 <= index < self._size):
            raise IndexError(f"Index {index} out of range")
        value = self._data[index]
        # Shift elements left
        for i in range(index, self._size - 1):
            self._data[i] = self._data[i + 1]
        self._data[self._size - 1] = None  # clear last slot
        self._size -= 1
        return value

    def _resize(self) -> None:
        """Double capacity. O(n)."""
        new_capacity = self._capacity * 2
        new_data = [None] * new_capacity
        for i in range(self._size):
            new_data[i] = self._data[i]
        self._data = new_data
        self._capacity = new_capacity
        print(f"  [Resized to capacity: {new_capacity}]")

    def search(self, value) -> int:
        """Find index of value. O(n)."""
        for i in range(self._size):
            if self._data[i] == value:
                return i
        return -1

    @property
    def capacity(self) -> int:
        return self._capacity


# Test it
arr = DynamicArray()
print(f"Empty array: {arr}, capacity: {arr.capacity}")

for i in [10, 20, 30, 40]:
    arr.append(i)

print(f"After 4 appends: {arr}, capacity: {arr.capacity}")

arr.append(50)  # triggers resize
print(f"After 5th append: {arr}, capacity: {arr.capacity}")

print(f"arr[0] = {arr[0]}")
print(f"arr[-1] = {arr[-1]}")

arr.insert(1, 15)
print(f"After insert(1, 15): {arr}")

arr.remove_at(0)
print(f"After remove_at(0): {arr}")

print(f"Search 30: index {arr.search(30)}")
print(f"Search 99: index {arr.search(99)}")
```

### Real Backend Use Cases

```python
from typing import List, Dict, Any

# ─────────────────────────────────────────
# PAGINATION — using list slicing
# ─────────────────────────────────────────
def paginate(
    items: List[Any],
    page: int = 1,
    page_size: int = 10
) -> Dict[str, Any]:
    """
    Paginate a list of items.
    O(1) for slice calculation, O(k) for actual slice.
    """
    total = len(items)                              # O(1)
    total_pages = (total + page_size - 1) // page_size
    start = (page - 1) * page_size                  # O(1)
    end = min(start + page_size, total)             # O(1)

    return {
        "items": items[start:end],                  # O(k)
        "total": total,
        "page": page,
        "page_size": page_size,
        "total_pages": total_pages,
        "has_next": page < total_pages,
        "has_prev": page > 1,
    }


# Test
all_users = [{"id": i, "name": f"User{i}"} for i in range(1, 101)]
page1 = paginate(all_users, page=1, page_size=10)
print(f"Page 1: {[u['id'] for u in page1['items']]}")
print(f"Total pages: {page1['total_pages']}")

# ─────────────────────────────────────────
# BATCH PROCESSING — chunking a list
# ─────────────────────────────────────────
def batch(items: List[Any], batch_size: int):
    """Split list into batches. Memory efficient generator."""
    for i in range(0, len(items), batch_size):
        yield items[i:i + batch_size]


# Process 10,000 users in batches of 100
all_records = list(range(10_000))
for chunk in batch(all_records, 100):
    # Send batch to database, API, etc.
    pass    # process chunk

# ─────────────────────────────────────────
# SLIDING WINDOW — monitoring last N events
# ─────────────────────────────────────────
class RollingWindow:
    """Keep last N items — like a sliding window."""

    def __init__(self, max_size: int):
        self.max_size = max_size
        self._data: List[Any] = []

    def add(self, item: Any) -> None:
        self._data.append(item)             # O(1)
        if len(self._data) > self.max_size:
            self._data.pop(0)               # O(n) — better to use deque!

    def get_all(self) -> List[Any]:
        return self._data.copy()

    def average(self) -> float:
        if not self._data:
            return 0.0
        return sum(self._data) / len(self._data)


# Track response times for last 100 requests
response_times = RollingWindow(100)
import random
for _ in range(150):
    response_times.add(random.uniform(10, 200))

print(f"Avg response time: {response_times.average():.1f}ms")
print(f"Window size: {len(response_times.get_all())}")  # 100
```

---

## 2. Linked Lists
Unlike arrays, linked lists do NOT store elements in contiguous memory.
Each element (node) stores:
1. The data value.
2. A pointer (reference) to the NEXT node.

```
Memory layout:
  [10 | •]──► [20 | •]──► [30 | •]──► [40 | None]
   node1        node2        node3        node4
```

```
Advantages:
  ✅ Insert/delete at front: O(1) (just update pointer)
  ✅ No need to shift elements
  ✅ Dynamic size (no pre-allocation)

Disadvantages:
  ❌ No random access (must traverse from head)
  ❌ More memory (storing pointers)
  ❌ Not cache-friendly (scattered in memory)
```

### Singly Linked List
```python
class Node:
    """A single node in a linked list."""

    def __init__(self, data):
        self.data = data
        self.next = None    # pointer to next node

    def __repr__(self):
        return f"Node({self.data})"

class SinglyLinkedList:
    """
    Linked list where each node points to next.
    Can only traverse FORWARD.
    """

    def __init__(self):
        self.head = None    # first node
        self._size = 0

    def __len__(self) -> int:
        return self._size

    def __repr__(self) -> str:
        nodes = []
        current = self.head
        while current:
            nodes.append(str(current.data))
            current = current.next
        return " → ".join(nodes) + " → None"

    def __iter__(self):
        current = self.head
        while current:
            yield current.data
            current = current.next

    # ─────────────────────────────────────────
    # INSERTION
    # ─────────────────────────────────────────
    def prepend(self, data) -> None:
        """Insert at beginning. O(1)."""
        new_node = Node(data)
        new_node.next = self.head   # new node points to old head
        self.head = new_node        # head moves to new node
        self._size += 1

    def append(self, data) -> None:
        """Insert at end. O(n)."""
        new_node = Node(data)
        if not self.head:
            self.head = new_node
            self._size += 1
            return
        # Traverse to last node
        current = self.head
        while current.next:
            current = current.next
        current.next = new_node     # last node points to new node
        self._size += 1

    def insert_at(self, index: int, data) -> None:
        """Insert at specific position. O(n)."""
        if index < 0 or index > self._size:
            raise IndexError(f"Index {index} out of range")
        if index == 0:
            self.prepend(data)
            return
        new_node = Node(data)
        current = self.head
        for _ in range(index - 1):  # get to node BEFORE index
            current = current.next
        new_node.next = current.next
        current.next = new_node
        self._size += 1

    # ─────────────────────────────────────────
    # DELETION
    # ─────────────────────────────────────────
    def delete_head(self):
        """Remove first node. O(1)."""
        if not self.head:
            raise IndexError("List is empty")
        data = self.head.data
        self.head = self.head.next
        self._size -= 1
        return data

    def delete_at(self, index: int):
        """Remove node at index. O(n)."""
        if index < 0 or index >= self._size:
            raise IndexError(f"Index {index} out of range")
        if index == 0:
            return self.delete_head()
        current = self.head
        for _ in range(index - 1):  # get to node BEFORE target
            current = current.next
        data = current.next.data
        current.next = current.next.next    # bypass deleted node
        self._size -= 1
        return data

    def delete_value(self, data) -> bool:
        """Remove first occurrence of value. O(n)."""
        if not self.head:
            return False
        if self.head.data == data:
            self.delete_head()
            return True
        current = self.head
        while current.next:
            if current.next.data == data:
                current.next = current.next.next
                self._size -= 1
                return True
            current = current.next
        return False

    # ─────────────────────────────────────────
    # SEARCH & ACCESS
    # ─────────────────────────────────────────
    def search(self, data) -> int:
        """Find index of value. O(n)."""
        current = self.head
        index = 0
        while current:
            if current.data == data:
                return index
            current = current.next
            index += 1
        return -1

    def get(self, index: int):
        """Get value at index. O(n)."""
        if index < 0 or index >= self._size:
            raise IndexError(f"Index {index} out of range")
        current = self.head
        for _ in range(index):
            current = current.next
        return current.data

    # ─────────────────────────────────────────
    # USEFUL OPERATIONS
    # ─────────────────────────────────────────
    def reverse(self) -> None:
        """Reverse the list in-place. O(n)."""
        prev = None
        current = self.head
        while current:
            next_node = current.next    # save next
            current.next = prev         # reverse pointer
            prev = current              # move prev forward
            current = next_node         # move current forward
        self.head = prev                # update head

    def to_list(self) -> list:
        return list(self)

    def contains(self, data) -> bool:
        return self.search(data) != -1


# Test it
ll = SinglyLinkedList()
print(f"Empty: {ll}")

ll.append(10)
ll.append(20)
ll.append(30)
ll.prepend(5)
print(f"After ops: {ll}")      # 5 → 10 → 20 → 30 → None

ll.insert_at(2, 15)
print(f"Insert 15 at 2: {ll}") # 5 → 10 → 15 → 20 → 30 → None

ll.delete_at(0)
print(f"Delete at 0: {ll}")    # 10 → 15 → 20 → 30 → None

ll.delete_value(20)
print(f"Delete 20: {ll}")      # 10 → 15 → 30 → None

print(f"Search 15: index {ll.search(15)}")  # 1
print(f"Get index 2: {ll.get(2)}")          # 30

ll.reverse()
print(f"Reversed: {ll}")       # 30 → 15 → 10 → None

print(f"As list: {ll.to_list()}")
print(f"Length: {len(ll)}")
```

### Doubly Linked List
```python
class DNode:
    """Node with pointers to both next AND previous."""

    def __init__(self, data):
        self.data = data
        self.next = None
        self.prev = None

    def __repr__(self):
        return f"DNode({self.data})"


class DoublyLinkedList:
    """
    Linked list where each node points both ways.
    Can traverse FORWARD and BACKWARD.
    O(1) insert/delete at both ends.
    """

    def __init__(self):
        self.head = None    # first node
        self.tail = None    # last node (for O(1) append)
        self._size = 0

    def __len__(self) -> int:
        return self._size

    def __repr__(self) -> str:
        nodes = []
        current = self.head
        while current:
            nodes.append(str(current.data))
            current = current.next
        return "None ⟺ " + " ⟺ ".join(nodes) + " ⟺ None"

    def prepend(self, data) -> None:
        """Insert at front. O(1)."""
        new_node = DNode(data)
        if not self.head:
            self.head = self.tail = new_node
        else:
            new_node.next = self.head
            self.head.prev = new_node
            self.head = new_node
        self._size += 1

    def append(self, data) -> None:
        """Insert at end. O(1) — we have tail pointer!"""
        new_node = DNode(data)
        if not self.tail:
            self.head = self.tail = new_node
        else:
            new_node.prev = self.tail
            self.tail.next = new_node
            self.tail = new_node
        self._size += 1

    def delete_head(self):
        """Remove from front. O(1)."""
        if not self.head:
            raise IndexError("List is empty")
        data = self.head.data
        if self.head == self.tail:  # only one node
            self.head = self.tail = None
        else:
            self.head = self.head.next
            self.head.prev = None
        self._size -= 1
        return data

    def delete_tail(self):
        """Remove from end. O(1) — we have tail pointer!"""
        if not self.tail:
            raise IndexError("List is empty")
        data = self.tail.data
        if self.head == self.tail:  # only one node
            self.head = self.tail = None
        else:
            self.tail = self.tail.prev
            self.tail.next = None
        self._size -= 1
        return data

    def delete_node(self, node: DNode):
        """Delete a specific node. O(1) — we have both pointers!"""
        if node.prev:
            node.prev.next = node.next
        else:
            self.head = node.next   # deleting head

        if node.next:
            node.next.prev = node.prev
        else:
            self.tail = node.prev   # deleting tail

        self._size -= 1


# Test
dll = DoublyLinkedList()
dll.append(10)
dll.append(20)
dll.append(30)
dll.prepend(5)
print(dll)          # None ⟺ 5 ⟺ 10 ⟺ 20 ⟺ 30 ⟺ None

dll.delete_head()
print(dll)          # None ⟺ 10 ⟺ 20 ⟺ 30 ⟺ None

dll.delete_tail()
print(dll)          # None ⟺ 10 ⟺ 20 ⟺ None

print(f"Length: {len(dll)}")
```

### When to Use Linked Lists
```
Use linked list when:
  ✅ Frequent insert/delete at beginning: O(1) vs O(n) for array
  ✅ Implementing queues and stacks
  ✅ You don't need random access
  ✅ Size changes frequently

Use array/list when:
  ✅ Need random access by index: O(1) vs O(n) for linked list
  ✅ Mostly reading, rarely inserting/deleting
  ✅ Cache performance matters (contiguous memory)
  ✅ Need sorting

In Python practice:
  collections.deque is a doubly linked list
  Use it when you need O(1) operations at both ends
```

---

## 3. Stacks & Queues

### Stack — LIFO (Last In, First Out)

```python
"""
Stack = like a pile of plates
  Add plate on top (push)
  Take plate from top (pop)
  Last added = first removed

LIFO: Last In, First Out

Real uses:
  - Function call stack (how recursion works)
  - Undo/redo functionality
  - Browser back button
  - Expression evaluation
  - Parsing (brackets matching)
"""

class Stack:
    """
    Stack implemented with a list.
    All operations are O(1).
    """

    def __init__(self):
        self._data = []

    def push(self, item) -> None:
        """Add item to top. O(1)."""
        self._data.append(item)

    def pop(self):
        """Remove and return top item. O(1)."""
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self._data.pop()

    def peek(self):
        """Look at top item without removing. O(1)."""
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self._data[-1]

    def is_empty(self) -> bool:
        return len(self._data) == 0

    def __len__(self) -> int:
        return len(self._data)

    def __repr__(self) -> str:
        if not self._data:
            return "Stack(empty)"
        return f"Stack(top → {self._data[::-1]})"


# Basic usage
stack = Stack()
stack.push(1)
stack.push(2)
stack.push(3)
print(stack)            # Stack(top → [3, 2, 1])
print(stack.peek())     # 3 (look without removing)
print(stack.pop())      # 3 (remove)
print(stack.pop())      # 2
print(len(stack))       # 1


# ─────────────────────────────────────────
# REAL USE 1: Validate bracket matching
# ─────────────────────────────────────────
def is_balanced(text: str) -> bool:
    """
    Check if brackets are properly matched.
    Used in: code editors, JSON parsers, expression evaluators.
    """
    stack = Stack()
    pairs = {")": "(", "]": "[", "}": "{"}
    opening = set("([{")

    for char in text:
        if char in opening:
            stack.push(char)
        elif char in pairs:
            if stack.is_empty():
                return False
            if stack.pop() != pairs[char]:
                return False

    return stack.is_empty()  # balanced if stack empty


print(is_balanced("({[]})"))    # True
print(is_balanced("([)]"))      # False
print(is_balanced("{[}]"))      # False
print(is_balanced("((()))"))    # True


# ─────────────────────────────────────────
# REAL USE 2: Undo/Redo system
# ─────────────────────────────────────────
class TextEditor:
    """Simple text editor with undo/redo."""

    def __init__(self):
        self.text = ""
        self.undo_stack = Stack()
        self.redo_stack = Stack()

    def type(self, text: str) -> None:
        self.undo_stack.push(self.text)     # save current state
        self.redo_stack = Stack()           # clear redo on new action
        self.text += text

    def delete(self, n: int) -> None:
        """Delete last n characters."""
        self.undo_stack.push(self.text)
        self.redo_stack = Stack()
        self.text = self.text[:-n]

    def undo(self) -> None:
        if self.undo_stack.is_empty():
            print("Nothing to undo")
            return
        self.redo_stack.push(self.text)
        self.text = self.undo_stack.pop()

    def redo(self) -> None:
        if self.redo_stack.is_empty():
            print("Nothing to redo")
            return
        self.undo_stack.push(self.text)
        self.text = self.redo_stack.pop()

    def __repr__(self):
        return f"TextEditor('{self.text}')"


editor = TextEditor()
editor.type("Hello")
print(editor)           # TextEditor('Hello')
editor.type(" World")
print(editor)           # TextEditor('Hello World')
editor.delete(6)
print(editor)           # TextEditor('Hello')
editor.undo()
print(editor)           # TextEditor('Hello World')
editor.undo()
print(editor)           # TextEditor('Hello')
editor.undo()
print(editor)           # TextEditor('')
editor.redo()
print(editor)           # TextEditor('Hello')
```

### Queue — FIFO (First In, First Out)

```python
"""
Queue = like a line at a coffee shop
  Join at back (enqueue)
  Served from front (dequeue)
  First arrived = first served

FIFO: First In, First Out

Real uses:
  - Background job processing (Celery)
  - HTTP request queuing
  - Message queues (RabbitMQ, Kafka)
  - Breadth-first search
  - Print queue
"""

from collections import deque

class Queue:
    """
    Queue implemented with collections.deque.
    deque gives O(1) for both ends (linked list internally).
    Using list would give O(n) for dequeue (shift all elements).
    """

    def __init__(self):
        self._data = deque()

    def enqueue(self, item) -> None:
        """Add to back. O(1)."""
        self._data.append(item)

    def dequeue(self):
        """Remove from front. O(1)."""
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self._data.popleft()     # O(1) with deque

    def peek(self):
        """Look at front item. O(1)."""
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self._data[0]

    def is_empty(self) -> bool:
        return len(self._data) == 0

    def __len__(self) -> int:
        return len(self._data)

    def __repr__(self) -> str:
        return f"Queue(front → {list(self._data)} ← back)"


# Basic usage
q = Queue()
q.enqueue("Task 1")
q.enqueue("Task 2")
q.enqueue("Task 3")
print(q)                      # Queue(front → ['Task 1', 'Task 2', 'Task 3'] ← back)
print(q.dequeue())            # Task 1 (first in, first out)
print(q.dequeue())            # Task 2
print(q.peek())               # Task 3 (look without removing)
print(len(q))                 # 1


# ─────────────────────────────────────────
# REAL USE: Job processing queue
# ─────────────────────────────────────────
import time
import random
from typing import Callable, Any
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class Job:
    id: int
    type: str
    payload: dict
    created_at: str = field(
        default_factory=lambda: datetime.now().isoformat()
    )
    status: str = "pending"


class JobQueue:
    """
    Simple background job queue.
    Real version: Celery + Redis/RabbitMQ
    """

    def __init__(self):
        self._queue = deque()
        self._job_id = 0
        self._processed = []

    def submit(self, job_type: str, payload: dict) -> Job:
        """Submit a job. O(1)."""
        self._job_id += 1
        job = Job(
            id=self._job_id,
            type=job_type,
            payload=payload
        )
        self._queue.append(job)
        print(f"[QUEUE] Job {job.id} ({job.type}) submitted")
        return job

    def process_next(self) -> bool:
        """Process the next job. O(1)."""
        if not self._queue:
            return False

        job = self._queue.popleft()
        job.status = "processing"
        print(f"[WORKER] Processing job {job.id}: {job.type}")

        # Simulate processing
        time.sleep(random.uniform(0.01, 0.05))

        job.status = "completed"
        self._processed.append(job)
        print(f"[WORKER] Job {job.id} completed")
        return True

    def process_all(self) -> None:
        """Process all pending jobs."""
        while self._queue:
            self.process_next()

    @property
    def pending_count(self) -> int:
        return len(self._queue)

    @property
    def processed_count(self) -> int:
        return len(self._processed)


# Simulate a backend job queue
job_queue = JobQueue()

# Submit various jobs
job_queue.submit("send_email", {"to": "alice@test.com", "subject": "Welcome!"})
job_queue.submit("resize_image", {"url": "image.jpg", "size": "800x600"})
job_queue.submit("generate_report", {"user_id": 42, "type": "monthly"})
job_queue.submit("send_sms", {"phone": "+1-555-0123", "message": "Your code: 1234"})

print(f"\nPending jobs: {job_queue.pending_count}")

# Process all
print("\nProcessing jobs...")
job_queue.process_all()

print(f"\nPending: {job_queue.pending_count}")
print(f"Processed: {job_queue.processed_count}")
```

### Priority Queue

```python
import heapq

class PriorityQueue:
    """
    Queue where items are dequeued by PRIORITY.
    Uses a heap internally.
    O(log n) for enqueue and dequeue.
    """

    def __init__(self):
        self._heap = []     # heap of (priority, counter, item)
        self._counter = 0   # ensures stable ordering for equal priorities

    def enqueue(self, item, priority: int = 0) -> None:
        """
        Add item with priority.
        Lower number = higher priority (like urgency level).
        O(log n).
        """
        heapq.heappush(self._heap, (priority, self._counter, item))
        self._counter += 1

    def dequeue(self):
        """Remove and return highest priority item. O(log n)."""
        if self.is_empty():
            raise IndexError("Priority queue is empty")
        priority, _, item = heapq.heappop(self._heap)
        return item, priority

    def peek(self):
        if self.is_empty():
            raise IndexError("Priority queue is empty")
        priority, _, item = self._heap[0]
        return item, priority

    def is_empty(self) -> bool:
        return len(self._heap) == 0

    def __len__(self) -> int:
        return len(self._heap)


# Test priority queue
pq = PriorityQueue()

# Lower priority number = processed first
pq.enqueue("Send newsletter", priority=3)    # low priority
pq.enqueue("Process payment", priority=1)   # high priority
pq.enqueue("Send welcome email", priority=2) # medium priority
pq.enqueue("Critical security patch", priority=0)  # critical!

print("Processing by priority:")
while not pq.is_empty():
    item, priority = pq.dequeue()
    print(f"  [{priority}] {item}")

# Output (by priority order):
# [0] Critical security patch
# [1] Process payment
# [2] Send welcome email
# [3] Send newsletter
```

---

## 4. `#Hash` Tables / Hash Maps

### What is a Hash Table?

```
A hash table stores key-value pairs.
Uses a HASH FUNCTION to convert a key to an index.

Key → hash function → index → value

Example:
  key = "alice"
  hash("alice") = 12345678
  index = 12345678 % 10 = 8  (10 slots in table)
  table[8] = {"email": "alice@test.com"}

Looking up "alice":
  1. Compute hash("alice") = 12345678  → O(1)
  2. index = 12345678 % 10 = 8        → O(1)
  3. Return table[8]                  → O(1)

No searching! Direct calculation.
This is why dict lookup is O(1).
```

### How Python Dicts Work Internally

```python
# Python's dict IS a hash table
# Let's understand what's happening under the hood

# ─────────────────────────────────────────
# HASH FUNCTION
# ─────────────────────────────────────────
print(hash("alice"))        # consistent integer (e.g., 4891729384)
print(hash("alice"))        # same every time (within one run)
print(hash("bob"))          # different value

print(hash(42))             # 42 (integers hash to themselves)
print(hash(3.14))           # some integer
print(hash((1, 2, 3)))      # tuples are hashable (immutable)

try:
    print(hash([1, 2, 3]))  # TypeError! lists not hashable (mutable)
except TypeError as e:
    print(e)

# ─────────────────────────────────────────
# COLLISION HANDLING
# ─────────────────────────────────────────
"""
What if two keys hash to same index? That's a COLLISION.

Example:
  hash("alice") % 10 = 3
  hash("dave")  % 10 = 3  ← collision!

Python uses OPEN ADDRESSING with PROBING:
  When collision at index 3 → try index 3+1=4, 3+4=7, etc.
  Find next empty slot.

Alternative: CHAINING (each slot is a list)
  index 3 → ["alice": ..., "dave": ...]
"""

# ─────────────────────────────────────────
# PERFORMANCE CHARACTERISTICS
# ─────────────────────────────────────────
import time

n = 1_000_000

# Build a list and dict with same data
data_list = list(range(n))
data_dict = {i: True for i in range(n)}

target = n - 1  # worst case for list (last element)

# List search: O(n)
start = time.perf_counter()
for _ in range(1000):
    _ = target in data_list
list_time = (time.perf_counter() - start) * 1000

# Dict lookup: O(1)
start = time.perf_counter()
for _ in range(1000):
    _ = target in data_dict
dict_time = (time.perf_counter() - start) * 1000

print(f"List search O(n): {list_time:.2f}ms")
print(f"Dict lookup O(1): {dict_time:.2f}ms")
print(f"Dict is {list_time/dict_time:.0f}x faster!")
# Dict can be 1000x+ faster for large datasets!
```

### Implementing a Hash Table from Scratch

```python
class HashTable:
    """
    Hash table using separate chaining for collision resolution.
    Shows exactly how Python's dict works internally.
    """

    def __init__(self, initial_capacity: int = 16):
        self.capacity = initial_capacity
        self._size = 0
        # Each bucket is a list of (key, value) pairs
        self.buckets = [[] for _ in range(self.capacity)]
        self.load_factor_threshold = 0.75  # resize when 75% full

    def _hash(self, key) -> int:
        """Convert key to bucket index."""
        return hash(key) % self.capacity

    def _load_factor(self) -> float:
        return self._size / self.capacity

    def _resize(self) -> None:
        """Double capacity and rehash all entries. O(n)."""
        old_buckets = self.buckets
        self.capacity *= 2
        self.buckets = [[] for _ in range(self.capacity)]
        self._size = 0

        for bucket in old_buckets:
            for key, value in bucket:
                self.set(key, value)    # rehash into new buckets

        print(f"  [HashTable resized to {self.capacity}]")

    def set(self, key, value) -> None:
        """Insert or update key-value pair. O(1) average."""
        if self._load_factor() >= self.load_factor_threshold:
            self._resize()

        index = self._hash(key)
        bucket = self.buckets[index]

        # Check if key exists in bucket (update)
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = (key, value)    # update
                return

        # Key not found — insert new
        bucket.append((key, value))
        self._size += 1

    def get(self, key, default=None):
        """Get value by key. O(1) average."""
        index = self._hash(key)
        bucket = self.buckets[index]
        for k, v in bucket:
            if k == key:
                return v
        return default

    def delete(self, key) -> bool:
        """Remove key-value pair. O(1) average."""
        index = self._hash(key)
        bucket = self.buckets[index]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket.pop(i)
                self._size -= 1
                return True
        return False

    def contains(self, key) -> bool:
        return self.get(key) is not None

    def keys(self) -> list:
        return [k for bucket in self.buckets for k, v in bucket]

    def values(self) -> list:
        return [v for bucket in self.buckets for k, v in bucket]

    def items(self) -> list:
        return [(k, v) for bucket in self.buckets for k, v in bucket]

    def __repr__(self) -> str:
        items = self.items()
        return f"HashTable({dict(items)})"


# Test
ht = HashTable(4)   # small capacity to see resizing

ht.set("alice", {"email": "alice@test.com", "role": "admin"})
ht.set("bob", {"email": "bob@test.com", "role": "user"})
ht.set("charlie", {"email": "charlie@test.com", "role": "user"})
ht.set("dave", {"email": "dave@test.com", "role": "moderator"})

print(ht)
print(ht.get("alice"))
print(ht.get("nobody", "not found"))

ht.set("alice", {"email": "alice@test.com", "role": "superadmin"})  # update
print(ht.get("alice"))

ht.delete("bob")
print(ht.contains("bob"))   # False
print(ht.keys())
```

### Real Backend Use Cases

```python
from collections import defaultdict
from typing import Optional, List
import time

# ─────────────────────────────────────────
# USE 1: O(1) User Lookup Cache
# ─────────────────────────────────────────
class UserCache:
    """
    In-memory cache for user objects.
    O(1) lookup instead of O(n) database query for cached users.
    """

    def __init__(self, max_size: int = 1000):
        self._cache: dict = {}
        self._access_times: dict = {}
        self.max_size = max_size
        self.hits = 0
        self.misses = 0

    def get(self, user_id: int) -> Optional[dict]:
        """O(1) lookup."""
        if user_id in self._cache:
            self.hits += 1
            self._access_times[user_id] = time.time()
            return self._cache[user_id]
        self.misses += 1
        return None

    def set(self, user_id: int, user: dict) -> None:
        """O(1) insert."""
        if len(self._cache) >= self.max_size:
            self._evict()
        self._cache[user_id] = user
        self._access_times[user_id] = time.time()

    def _evict(self) -> None:
        """Remove least recently accessed item."""
        oldest_id = min(
            self._access_times,
            key=self._access_times.get
        )
        del self._cache[oldest_id]
        del self._access_times[oldest_id]

    @property
    def hit_rate(self) -> float:
        total = self.hits + self.misses
        return self.hits / total if total > 0 else 0


cache = UserCache(max_size=3)
cache.set(1, {"id": 1, "name": "Alice"})
cache.set(2, {"id": 2, "name": "Bob"})

print(cache.get(1))     # {'id': 1, 'name': 'Alice'} — HIT
print(cache.get(99))    # None — MISS
print(f"Hit rate: {cache.hit_rate:.0%}")    # 50%

# ─────────────────────────────────────────
# USE 2: Frequency Counter
# ─────────────────────────────────────────
def analyze_requests(logs: List[dict]) -> dict:
    """
    Analyze API request logs.
    O(n) to build, O(1) to query.
    """
    endpoint_counts = defaultdict(int)
    status_counts = defaultdict(int)
    method_counts = defaultdict(int)
    user_counts = defaultdict(int)

    for log in logs:
        endpoint_counts[log["endpoint"]] += 1
        status_counts[log["status"]] += 1
        method_counts[log["method"]] += 1
        if "user_id" in log:
            user_counts[log["user_id"]] += 1

    return {
        "top_endpoints": sorted(
            endpoint_counts.items(),
            key=lambda x: x[1],
            reverse=True
        )[:5],
        "status_distribution": dict(status_counts),
        "methods": dict(method_counts),
        "most_active_users": sorted(
            user_counts.items(),
            key=lambda x: x[1],
            reverse=True
        )[:3],
    }


# Test
import random
endpoints = ["/users", "/posts", "/auth/login", "/products", "/orders"]
methods = ["GET", "POST", "PUT", "DELETE"]
statuses = [200, 200, 200, 201, 400, 404, 500]

logs = [
    {
        "endpoint": random.choice(endpoints),
        "method": random.choice(methods),
        "status": random.choice(statuses),
        "user_id": random.randint(1, 10)
    }
    for _ in range(1000)
]

analysis = analyze_requests(logs)
print("Top endpoints:", analysis["top_endpoints"][:3])
print("Status codes:", analysis["status_distribution"])
```

---

## 5. Trees

### What is a Tree?

```
A tree is a hierarchical data structure.

              [10]          ← root
             /    \
           [5]    [15]      ← internal nodes
          /  \   /   \
        [2] [7] [12] [20]  ← leaf nodes

Terms:
  Root:   top node (no parent)
  Parent: node above
  Child:  node below
  Leaf:   node with no children
  Height: longest path from root to leaf
  Depth:  distance from root to node

Why trees?
  Hierarchical data: file systems, org charts, categories
  Fast search: Binary Search Tree = O(log n)
  Foundation for: databases, file systems, DNS, HTML DOM
```

### Binary Tree

```python
class TreeNode:
    """A node in a binary tree."""

    def __init__(self, value):
        self.value = value
        self.left = None    # left child
        self.right = None   # right child

    def __repr__(self):
        return f"TreeNode({self.value})"


class BinaryTree:
    """Binary tree with traversal methods."""

    def __init__(self):
        self.root = None

    def insert_level_order(self, values: list) -> None:
        """Build tree from list (level by level)."""
        if not values:
            return
        self.root = TreeNode(values[0])
        queue = deque([self.root])
        i = 1
        while queue and i < len(values):
            node = queue.popleft()
            if i < len(values) and values[i] is not None:
                node.left = TreeNode(values[i])
                queue.append(node.left)
            i += 1
            if i < len(values) and values[i] is not None:
                node.right = TreeNode(values[i])
                queue.append(node.right)
            i += 1

    # ─────────────────────────────────────────
    # TRAVERSALS — three classic ways to visit all nodes
    # ─────────────────────────────────────────

    def inorder(self, node: TreeNode = None, result: list = None) -> list:
        """
        Left → Root → Right
        For BST: gives SORTED order!
        """
        if result is None:
            result = []
            node = self.root
        if node:
            self.inorder(node.left, result)     # go left
            result.append(node.value)           # visit node
            self.inorder(node.right, result)    # go right
        return result

    def preorder(self, node: TreeNode = None, result: list = None) -> list:
        """
        Root → Left → Right
        Used for: copying trees, expression trees
        """
        if result is None:
            result = []
            node = self.root
        if node:
            result.append(node.value)           # visit node first
            self.preorder(node.left, result)    # go left
            self.preorder(node.right, result)   # go right
        return result

    def postorder(self, node: TreeNode = None, result: list = None) -> list:
        """
        Left → Right → Root
        Used for: deleting trees, evaluating expressions
        """
        if result is None:
            result = []
            node = self.root
        if node:
            self.postorder(node.left, result)   # go left
            self.postorder(node.right, result)  # go right
            result.append(node.value)           # visit node last
        return result

    def level_order(self) -> list:
        """
        Level by level (BFS).
        Used for: finding shortest path, level-by-level processing.
        """
        if not self.root:
            return []
        result = []
        queue = deque([self.root])
        while queue:
            node = queue.popleft()
            result.append(node.value)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        return result

    def height(self, node: TreeNode = None) -> int:
        """Maximum depth of the tree. O(n)."""
        if node is None:
            node = self.root
        if node is None:
            return 0
        left_height = self.height(node.left)
        right_height = self.height(node.right)
        return 1 + max(left_height, right_height)

    def print_tree(self) -> None:
        """Visual representation of the tree."""
        def _print(node, level=0, prefix="Root: "):
            if node is not None:
                print(" " * (level * 4) + prefix + str(node.value))
                if node.left is not None or node.right is not None:
                    if node.left:
                        _print(node.left, level + 1, "L: ")
                    else:
                        print(" " * ((level + 1) * 4) + "L: None")
                    if node.right:
                        _print(node.right, level + 1, "R: ")
                    else:
                        print(" " * ((level + 1) * 4) + "R: None")

        _print(self.root)


# Build and traverse
tree = BinaryTree()
tree.insert_level_order([10, 5, 15, 2, 7, 12, 20])

tree.print_tree()
print(f"Inorder:    {tree.inorder()}")       # [2, 5, 7, 10, 12, 15, 20]
print(f"Preorder:   {tree.preorder()}")      # [10, 5, 2, 7, 15, 12, 20]
print(f"Postorder:  {tree.postorder()}")     # [2, 7, 5, 12, 20, 15, 10]
print(f"Level order:{tree.level_order()}")   # [10, 5, 15, 2, 7, 12, 20]
print(f"Height:     {tree.height()}")        # 3
```

### Binary Search Tree (BST)

```python
class BST:
    """
    Binary Search Tree.
    Property: left < node < right (always!)

    This property enables O(log n) search,
    insert, and delete in a balanced tree.
    """

    def __init__(self):
        self.root = None

    def insert(self, value) -> None:
        """Insert value maintaining BST property. O(log n) avg."""
        self.root = self._insert(self.root, value)

    def _insert(self, node: TreeNode, value) -> TreeNode:
        if node is None:
            return TreeNode(value)
        if value < node.value:
            node.left = self._insert(node.left, value)
        elif value > node.value:
            node.right = self._insert(node.right, value)
        # If equal: ignore (no duplicates)
        return node

    def search(self, value) -> bool:
        """Search for value. O(log n) avg."""
        return self._search(self.root, value)

    def _search(self, node: TreeNode, value) -> bool:
        if node is None:
            return False
        if value == node.value:
            return True
        if value < node.value:
            return self._search(node.left, value)   # go left
        return self._search(node.right, value)       # go right

    def delete(self, value) -> None:
        """Delete value. O(log n) avg."""
        self.root = self._delete(self.root, value)

    def _delete(self, node: TreeNode, value) -> TreeNode:
        if node is None:
            return None
        if value < node.value:
            node.left = self._delete(node.left, value)
        elif value > node.value:
            node.right = self._delete(node.right, value)
        else:
            # Found the node to delete!
            # Case 1: No children (leaf) → just remove
            if not node.left and not node.right:
                return None
            # Case 2: One child → replace with child
            if not node.left:
                return node.right
            if not node.right:
                return node.left
            # Case 3: Two children → replace with inorder successor
            # (smallest value in right subtree)
            successor = self._min_node(node.right)
            node.value = successor.value
            node.right = self._delete(node.right, successor.value)
        return node

    def _min_node(self, node: TreeNode) -> TreeNode:
        """Find minimum value node. O(h)."""
        while node.left:
            node = node.left
        return node

    def inorder(self) -> list:
        """Get sorted values. O(n)."""
        result = []
        self._inorder(self.root, result)
        return result

    def _inorder(self, node, result):
        if node:
            self._inorder(node.left, result)
            result.append(node.value)
            self._inorder(node.right, result)

    def min_value(self):
        if not self.root:
            return None
        return self._min_node(self.root).value

    def max_value(self):
        if not self.root:
            return None
        node = self.root
        while node.right:
            node = node.right
        return node.value


# Test BST
bst = BST()
for value in [10, 5, 15, 2, 7, 12, 20, 1, 6, 8]:
    bst.insert(value)

print(f"Inorder (sorted): {bst.inorder()}")
print(f"Search 7: {bst.search(7)}")     # True
print(f"Search 99: {bst.search(99)}")   # False
print(f"Min: {bst.min_value()}")        # 1
print(f"Max: {bst.max_value()}")        # 20

bst.delete(5)
print(f"After delete 5: {bst.inorder()}")
```

---

## 6. Heaps

### What is a Heap?

```
A heap is a COMPLETE binary tree with the heap property:

Min-Heap: parent is ALWAYS smaller than children
  Every parent ≤ its children
  Root = MINIMUM element
  Smallest element always at top → O(1) access

Max-Heap: parent is ALWAYS larger than children
  Root = MAXIMUM element

Heaps are stored as ARRAYS (not linked nodes):
  parent of index i = (i-1) // 2
  left child of i   = 2*i + 1
  right child of i  = 2*i + 2

Array: [1, 3, 2, 7, 5, 6, 4]
Tree:
        1
       / \
      3   2
     / \ / \
    7  5 6  4

Operations:
  peek min:    O(1)
  insert:      O(log n)
  extract min: O(log n)
```

```python
import heapq

# Python's heapq module implements a min-heap
# All operations work on a regular list

heap = []

# Insert (push)
heapq.heappush(heap, 5)
heapq.heappush(heap, 2)
heapq.heappush(heap, 8)
heapq.heappush(heap, 1)
heapq.heappush(heap, 9)

print(f"Heap: {heap}")      # [1, 2, 8, 5, 9] (heap order, not sorted)
print(f"Min: {heap[0]}")    # 1 — always at index 0, O(1)

# Extract minimum
print(heapq.heappop(heap))  # 1 (smallest)
print(heapq.heappop(heap))  # 2
print(heapq.heappop(heap))  # 5
# Items come out in sorted order!

# Build heap from existing list
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5]
heapq.heapify(numbers)      # O(n) — convert list to heap in-place
print(f"Heapified: {numbers}")

# Push and pop atomically (more efficient)
result = heapq.heappushpop(numbers, 0)  # push 0, pop minimum
print(f"Pushed 0, popped: {result}")

# n smallest/largest
nums = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
print(f"3 smallest: {heapq.nsmallest(3, nums)}")  # [1, 1, 2]
print(f"3 largest:  {heapq.nlargest(3, nums)}")   # [9, 6, 5]

# ─────────────────────────────────────────
# HEAP WITH OBJECTS
# ─────────────────────────────────────────
from dataclasses import dataclass, field

@dataclass(order=True)
class Task:
    priority: int
    name: str = field(compare=False)    # don't compare by name

tasks = []
heapq.heappush(tasks, Task(3, "Send newsletter"))
heapq.heappush(tasks, Task(1, "Process payment"))    # highest priority
heapq.heappush(tasks, Task(2, "Send welcome email"))
heapq.heappush(tasks, Task(0, "Critical: fix bug"))  # highest!

while tasks:
    task = heapq.heappop(tasks)
    print(f"[{task.priority}] {task.name}")
# [0] Critical: fix bug
# [1] Process payment
# [2] Send welcome email
# [3] Send newsletter
```

---

## 7. Graphs

### What is a Graph?

```
A graph consists of:
  VERTICES (nodes) — the things
  EDGES — connections between things

Examples:
  Social network: users = vertices, friendships = edges
  Road network:   cities = vertices, roads = edges
  Web pages:      pages = vertices, links = edges
  Microservices:  services = vertices, API calls = edges

Types:
  Directed:   edges have direction (A→B ≠ B→A)
              Like Twitter follows (you follow me ≠ I follow you)
  Undirected: edges are bidirectional (A—B = B—A)
              Like Facebook friends

  Weighted:   edges have values (road distances, costs)
  Unweighted: edges are just connections

Representations:
  Adjacency List:   {A: [B, C], B: [A], C: [A, D]}  ← common
  Adjacency Matrix: 2D grid, 1 if connected         ← dense graphs
```

```python
from collections import defaultdict, deque
from typing import Dict, List, Set, Optional

class Graph:
    """
    Undirected, unweighted graph.
    Using adjacency list representation.
    """

    def __init__(self):
        # {vertex: [list of neighbors]}
        self.adjacency: Dict[str, List[str]] = defaultdict(list)
        self.vertices: Set[str] = set()

    def add_vertex(self, vertex: str) -> None:
        """Add a vertex. O(1)."""
        self.vertices.add(vertex)
        if vertex not in self.adjacency:
            self.adjacency[vertex] = []

    def add_edge(self, u: str, v: str) -> None:
        """Add undirected edge. O(1)."""
        self.vertices.add(u)
        self.vertices.add(v)
        self.adjacency[u].append(v)
        self.adjacency[v].append(u)  # undirected: add both

    def remove_edge(self, u: str, v: str) -> None:
        """Remove edge. O(degree)."""
        self.adjacency[u] = [x for x in self.adjacency[u] if x != v]
        self.adjacency[v] = [x for x in self.adjacency[v] if x != u]

    def neighbors(self, vertex: str) -> List[str]:
        """Get all neighbors. O(1)."""
        return self.adjacency[vertex]

    def has_edge(self, u: str, v: str) -> bool:
        """Check if edge exists. O(degree)."""
        return v in self.adjacency[u]

    def __repr__(self) -> str:
        lines = []
        for vertex in sorted(self.vertices):
            neighbors = sorted(self.adjacency[vertex])
            lines.append(f"  {vertex}: {neighbors}")
        return "Graph:\n" + "\n".join(lines)

    # ─────────────────────────────────────────
    # BFS — Breadth-First Search
    # Explores level by level (closest nodes first)
    # Uses a QUEUE
    # ─────────────────────────────────────────
    def bfs(self, start: str) -> List[str]:
        """
        Visit all reachable vertices level by level.
        O(V + E) where V=vertices, E=edges.
        """
        if start not in self.vertices:
            return []

        visited = set()
        queue = deque([start])
        result = []
        visited.add(start)

        while queue:
            vertex = queue.popleft()    # take from front
            result.append(vertex)

            for neighbor in self.adjacency[vertex]:
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append(neighbor)  # add to back

        return result

    # ─────────────────────────────────────────
    # DFS — Depth-First Search
    # Explores as deep as possible before backtracking
    # Uses a STACK (or recursion)
    # ─────────────────────────────────────────
    def dfs(self, start: str) -> List[str]:
        """
        Visit all reachable vertices depth-first.
        O(V + E).
        """
        if start not in self.vertices:
            return []

        visited = set()
        result = []

        def _dfs(vertex: str) -> None:
            visited.add(vertex)
            result.append(vertex)
            for neighbor in self.adjacency[vertex]:
                if neighbor not in visited:
                    _dfs(neighbor)

        _dfs(start)
        return result

    def shortest_path(self, start: str, end: str) -> Optional[List[str]]:
        """
        Find shortest path using BFS.
        Works for UNWEIGHTED graphs.
        O(V + E).
        """
        if start not in self.vertices or end not in self.vertices:
            return None
        if start == end:
            return [start]

        # BFS tracking the path
        queue = deque([[start]])
        visited = {start}

        while queue:
            path = queue.popleft()
            vertex = path[-1]

            for neighbor in self.adjacency[vertex]:
                if neighbor not in visited:
                    new_path = path + [neighbor]
                    if neighbor == end:
                        return new_path     # found!
                    visited.add(neighbor)
                    queue.append(new_path)

        return None     # no path exists

    def is_connected(self) -> bool:
        """Check if all vertices are reachable from any start."""
        if not self.vertices:
            return True
        start = next(iter(self.vertices))
        return len(self.bfs(start)) == len(self.vertices)
```

### Test — Social Network Graph

```python
# ─────────────────────────────────────────
# TEST — Social network graph
# ─────────────────────────────────────────
graph = Graph()

# Build a social network
friendships = [
    ("Alice", "Bob"),
    ("Alice", "Charlie"),
    ("Bob", "Dave"),
    ("Charlie", "Dave"),
    ("Dave", "Eve"),
    ("Eve", "Frank"),
]

for u, v in friendships:
    graph.add_edge(u, v)

print(graph)

# Who are Alice's friends?
print(f"Alice's friends: {graph.neighbors('Alice')}")

# BFS from Alice — visit everyone level by level
print(f"BFS from Alice: {graph.bfs('Alice')}")

# DFS from Alice — go deep first
print(f"DFS from Alice: {graph.dfs('Alice')}")

# Shortest path
path = graph.shortest_path("Alice", "Frank")
print(f"Alice → Frank: {' → '.join(path)}")

print(f"Connected: {graph.is_connected()}")
```

### Weighted Graph with Dijkstra

```python
# ─────────────────────────────────────────
# WEIGHTED GRAPH — for distances, costs
# ─────────────────────────────────────────
class WeightedGraph:
    """Graph with edge weights (distances/costs)."""

    def __init__(self):
        # {vertex: [(neighbor, weight), ...]}
        self.adjacency: Dict[str, List[tuple]] = defaultdict(list)

    def add_edge(self, u: str, v: str, weight: float) -> None:
        self.adjacency[u].append((v, weight))
        self.adjacency[v].append((u, weight))    # undirected

    def dijkstra(self, start: str) -> Dict[str, float]:
        """
        Find shortest distances from start to all vertices.
        O((V + E) log V) with priority queue.
        """
        distances = defaultdict(lambda: float('inf'))
        distances[start] = 0
        pq = [(0, start)]   # (distance, vertex)
        visited = set()

        while pq:
            dist, vertex = heapq.heappop(pq)

            if vertex in visited:
                continue
            visited.add(vertex)

            for neighbor, weight in self.adjacency[vertex]:
                new_dist = dist + weight
                if new_dist < distances[neighbor]:
                    distances[neighbor] = new_dist
                    heapq.heappush(pq, (new_dist, neighbor))

        return dict(distances)


# Test Dijkstra
road_map = WeightedGraph()
road_map.add_edge("A", "B", 4)
road_map.add_edge("A", "C", 2)
road_map.add_edge("B", "D", 5)
road_map.add_edge("C", "B", 1)
road_map.add_edge("C", "D", 8)
road_map.add_edge("D", "E", 2)

distances = road_map.dijkstra("A")
print(f"Distances from A: {dict(distances)}")
# A=0, B=3(A→C→B), C=2, D=8(A→C→B→D), E=10
```

---

## 8. Tries

### What is a Trie?

```
A Trie (prefix tree) stores strings character by character.
Each path from root to a node represents a prefix.

Example with words: ["cat", "car", "card", "care", "bat"]:

        root
       /    \
      c      b
      |      |
      a      a
     / \     |
    t   r    t
        |\
        d  e

"cat" = root → c → a → t
"car" = root → c → a → r
"card" = root → c → a → r → d

Advantages:
  O(k) search where k = length of word
  O(k) insert
  Efficient prefix searching (autocomplete!)
  Common prefix stored once (memory efficient)
```

```python
class TrieNode:
    def __init__(self):
        self.children: Dict[str, "TrieNode"] = {}
        self.is_end_of_word: bool = False
        self.word: Optional[str] = None     # store complete word at end
        self.frequency: int = 0             # for ranking suggestions


class Trie:
    """
    Prefix tree for efficient string operations.
    Perfect for autocomplete, spell check, IP routing.
    """

    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str, frequency: int = 1) -> None:
        """Insert word. O(k) where k = word length."""
        node = self.root
        for char in word.lower():
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True
        node.word = word
        node.frequency += frequency

    def search(self, word: str) -> bool:
        """Check if exact word exists. O(k)."""
        node = self._find_node(word)
        return node is not None and node.is_end_of_word

    def starts_with(self, prefix: str) -> bool:
        """Check if any word starts with prefix. O(k)."""
        return self._find_node(prefix) is not None

    def _find_node(self, prefix: str) -> Optional[TrieNode]:
        """Find node at end of prefix. O(k)."""
        node = self.root
        for char in prefix.lower():
            if char not in node.children:
                return None
            node = node.children[char]
        return node

    def autocomplete(
        self,
        prefix: str,
        max_results: int = 10
    ) -> List[str]:
        """
        Get all words starting with prefix.
        O(k + n) where k=prefix length, n=matches found.
        """
        node = self._find_node(prefix)
        if not node:
            return []

        results = []
        self._collect_words(node, results)

        # Sort by frequency (most popular first)
        results.sort(key=lambda x: x[1], reverse=True)
        return [word for word, freq in results[:max_results]]

    def _collect_words(
        self,
        node: TrieNode,
        results: list
    ) -> None:
        """DFS to collect all words from this node."""
        if node.is_end_of_word:
            results.append((node.word, node.frequency))
        for child in node.children.values():
            self._collect_words(child, results)

    def delete(self, word: str) -> bool:
        """Delete a word. O(k)."""
        return self._delete(self.root, word, 0)

    def _delete(self, node: TrieNode, word: str, depth: int) -> bool:
        if depth == len(word):
            if not node.is_end_of_word:
                return False    # word not found
            node.is_end_of_word = False
            node.word = None
            return len(node.children) == 0  # True if node can be deleted

        char = word[depth].lower()
        if char not in node.children:
            return False

        should_delete_child = self._delete(
            node.children[char], word, depth + 1
        )
        if should_delete_child:
            del node.children[char]
            return len(node.children) == 0 and not node.is_end_of_word
        return False

    def count_words(self) -> int:
        """Count all words in trie."""
        count = [0]
        def _count(node):
            if node.is_end_of_word:
                count[0] += 1
            for child in node.children.values():
                _count(child)

        _count(self.root)
        return count[0]
```

### Test Autocomplete

```python
# Test autocomplete system
trie = Trie()

# Add words with frequencies (simulate search frequencies)
words_with_freq = [
    ("python", 1000),
    ("python tutorial", 800),
    ("python backend", 750),
    ("python fastapi", 600),
    ("python django", 550),
    ("python flask", 400),
    ("programming", 900),
    ("programmer", 700),
    ("program", 600),
    ("java", 500),
    ("javascript", 800),
    ("java tutorial", 400),
]

for word, freq in words_with_freq:
    trie.insert(word, freq)

# Autocomplete tests
print("=== Autocomplete Demo ===")
print(f"'py':   {trie.autocomplete('py', 5)}")
print(f"'pyt':  {trie.autocomplete('pyt', 5)}")
print(f"'pro':  {trie.autocomplete('pro', 5)}")
print(f"'java': {trie.autocomplete('java', 5)}")
print(f"'xyz':  {trie.autocomplete('xyz', 5)}")  # no results

print(f"\nSearch 'python': {trie.search('python')}")      # True
print(f"Search 'pyth':   {trie.search('pyth')}")         # False
print(f"Starts with 'py':{trie.starts_with('py')}")      # True
print(f"Word count:      {trie.count_words()}")

trie.delete("python")
print(f"After delete 'python': {trie.search('python')}")  # False
print(f"'python backend' still: {trie.search('python backend')}")  # True
```

---

## Visual Summary — Data Structures Comparison

```
┌────────────┬──────────┬──────────┬──────────┬───────────────┐
│Structure   │  Access  │  Search  │  Insert  │  Delete       │
├────────────┼──────────┼──────────┼──────────┼───────────────┤
│ Array/List │  O(1)    │  O(n)    │  O(n)*   │  O(n)         │
│            │          │          │  O(1)end │  O(1)end      │
├────────────┼──────────┼──────────┼──────────┼───────────────┤
│Linked List │  O(n)    │  O(n)    │  O(1)    │  O(1) with ref│
│ (singly)   │          │          │  front   │  O(n)         │   │            │          │          │          │ search+delete │
├────────────┼──────────┼──────────┼──────────┼───────────────┤
│ Stack      │  O(n)    │  O(n)    │  O(1)    │  O(1)         │
│            │  O(1)top │          │  top     │  top          │
├────────────┼──────────┼──────────┼──────────┼───────────────┤
│ Queue      │  O(n)    │  O(n)    │  O(1)    │  O(1)         │
│            │  O(1)frnt│          │  back    │  front        │
├────────────┼──────────┼──────────┼──────────┼───────────────┤
│ Hash Table │  O(1)*   │  O(1)*   │  O(1)*   │  O(1)*        │
│ (dict)     │  avg     │  avg     │  avg     │  avg          │
├────────────┼──────────┼──────────┼──────────┼───────────────┤
│ BST        │  O(log n)│  O(log n)│  O(log n)│  O(log n)     │
│ (balanced) │  avg     │  avg     │  avg     │  avg          │
├────────────┼──────────┼──────────┼──────────┼───────────────┤
│ Heap       │  O(1)    │  O(n)    │  O(log n)│  O(log n)     │
│            │  min/max │          │          │  min/max      │
├────────────┼──────────┼──────────┼──────────┼───────────────┤
│ Trie       │  O(k)    │  O(k)    │  O(k)    │  O(k)         │
│            │ k=length │ k=length │ k=length │  k=length     │
└────────────┴──────────┴──────────┴──────────┴───────────────┘
```

```
* = amortized or average case

WHEN TO USE WHICH:
  Array/List    → ordered data, random access, simple collections
  Linked List   → frequent insert/delete at front
                  (use deque in Python)
  Stack         → undo/redo, parsing, DFS
  Queue         → job processing, BFS, buffering
  Hash Table    → ANY key-value lookup (use dict!)
  BST           → sorted data + fast search
  Heap          → priority queues, top-K problems
  Trie          → string prefix search, autocomplete
```

---

## Knowledge Check

1. What is Big O notation and why does it matter?
2. Why is list access O(1) but search O(n)?
3. Why is insert at beginning O(n) for arrays?
4. What is the main advantage of linked lists over arrays?
5. What is the difference between a stack and a queue?
6. What does LIFO mean? FIFO?
7. How does a hash table achieve O(1) lookup?
8. What is a hash collision and how is it handled?
9. Why can't lists be used as dictionary keys?
10. What is the BST property?
11. What does inorder traversal of a BST give you?
12. What is a min-heap and what is its root?
13. Why is a Trie better than a hash set for autocomplete?
14. What is the difference between BFS and DFS?

---

## Practice Exercises

```python
# Exercise 1: Stack Application
# Implement an expression evaluator
# Evaluate: "3 + 4 * 2" = 11
# Handle: +, -, *, / with proper precedence
# Use two stacks: one for numbers, one for operators

# Exercise 2: Queue Application
# Build a rate limiter using a queue
# Allow max N requests per time window
# class RateLimiter:
#     def __init__(self, max_requests: int, window_seconds: int)
#     def allow_request(self, user_id: str) -> bool

# Exercise 3: Hash Table Application
# Given a list of integers and a target sum,
# find TWO numbers that add up to target.
# Must be O(n) time complexity (use a hash table)
# two_sum([2, 7, 11, 15], target=9) → (2, 7)

# Exercise 4: Binary Search Tree
# Implement these additional BST methods:
# - is_valid_bst() → check if tree follows BST property
# - kth_smallest(k) → find kth smallest element
# - range_query(low, high) → all values between low and high

# Exercise 5: Graph Application
# Build a simple dependency resolver
# Given packages and their dependencies:
# {"fastapi": ["starlette", "pydantic"],
#  "starlette": ["anyio"],
#  "pydantic": [],
#  "anyio": []}
# Return installation order (topological sort)
# So dependencies are installed before what needs them
```

---

## Phase 2.1 Complete!

**You now understand and can implement:**

```
✅ Big O notation and complexity analysis
✅ Dynamic Arrays — how Python lists work internally
✅ Singly and Doubly Linked Lists
✅ Stacks (LIFO) — with real use cases
✅ Queues (FIFO) — with priority queues
✅ Hash Tables — how Python dicts work internally
✅ Binary Trees and traversals
✅ Binary Search Trees
✅ Heaps (min/max) with heapq
✅ Graphs (directed/undirected, BFS, DFS, Dijkstra)
✅ Tries (prefix trees, autocomplete)
```

---

## What's Next?

```
Option 1: Continue  → Phase 2.2 (Algorithms)
                      Binary search, sorting, two pointers,
                      sliding window, dynamic programming...

Option 2: Practice  → Solve the exercises above first

Option 3: Questions → Anything unclear from 2.1?
```

**What do you want to do? 🚀**