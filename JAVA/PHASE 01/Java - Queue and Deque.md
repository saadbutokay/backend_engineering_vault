---
title: "Java - Queue and Deque"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - collections
  - queue
  - deque
  - priorityqueue
  - arraydeque
status: "not-started"
---

# Java - Queue and Deque

> [!abstract] Overview
> The `Queue` interface represents a collection designed for holding elements prior to processing, typically in FIFO (First-In-First-Out) order. The `Deque` (double-ended queue) interface extends `Queue` and allows insertion and removal at both ends, enabling both FIFO and LIFO (Last-In-First-Out) behavior. In backend development, queues power task scheduling, event-driven architectures, message processing pipelines, breadth-first search algorithms, rate limiting windows, and asynchronous job execution. The primary implementations are `PriorityQueue` (ordered by priority), `ArrayDeque` (fastest general-purpose deque), and `LinkedList` (implements both `List` and `Deque`).

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Collections Framework Overview]]
> - [[Java - List - ArrayList LinkedList]]
> - [[Java - Abstraction - Abstract Classes and Interfaces]]
> - [[Java - Comparable and Comparator]] (basic understanding)

---

## Theory

### The `Queue` Interface

The `Queue` interface extends `Collection` and defines a collection that orders elements for processing. The standard ordering is FIFO: the first element added is the first element removed. However, priority queues order elements by priority instead of insertion order.

**The `Queue` interface provides two sets of methods for each operation:**

| Operation | Throws Exception | Returns Special Value |
|-----------|-----------------|----------------------|
| Insert | `add(e)` | `offer(e)` (returns false if full) |
| Remove | `remove()` | `poll()` (returns null if empty) |
| Examine | `element()` | `peek()` (returns null if empty) |

The "throws exception" methods (`add`, `remove`, `element`) throw exceptions when the operation cannot be performed (e.g., adding to a full queue or removing from an empty queue). The "returns special value" methods (`offer`, `poll`, `peek`) return `false` or `null` instead of throwing. In backend development, the `offer`/`poll`/`peek` variants are preferred because they allow graceful handling of edge cases without exception overhead.

**Key `Queue` methods:**

| Method | Description | Return |
|--------|-------------|--------|
| `offer(E e)` | Inserts the element if possible | `boolean` (false if full) |
| `poll()` | Retrieves and removes the head | `E` (null if empty) |
| `peek()` | Retrieves but does not remove the head | `E` (null if empty) |
| `add(E e)` | Inserts the element (throws if full) | `boolean` |
| `remove()` | Retrieves and removes the head (throws if empty) | `E` |
| `element()` | Retrieves the head without removing (throws if empty) | `E` |

### The `Deque` Interface

The `Deque` (pronounced "deck") interface extends `Queue` and adds methods for inserting and removing elements at **both ends** of the queue. This makes it usable as both a queue (FIFO) and a stack (LIFO).

**Deque methods for both ends:**

| Operation | First (Head) | Last (Tail) |
|-----------|-------------|-------------|
| Insert | `addFirst(e)` / `offerFirst(e)` | `addLast(e)` / `offerLast(e)` |
| Remove | `removeFirst()` / `pollFirst()` | `removeLast()` / `pollLast()` |
| Examine | `getFirst()` / `peekFirst()` | `getLast()` / `peekLast()` |

**Deque as a Queue (FIFO):**

| Queue Method | Equivalent Deque Method |
|-------------|------------------------|
| `add(e)` | `addLast(e)` |
| `offer(e)` | `offerLast(e)` |
| `remove()` | `removeFirst()` |
| `poll()` | `pollFirst()` |
| `element()` | `getFirst()` |
| `peek()` | `peekFirst()` |

**Deque as a Stack (LIFO):**

| Stack Method | Equivalent Deque Method |
|-------------|------------------------|
| `push(e)` | `addFirst(e)` |
| `pop()` | `removeFirst()` |
| `peek()` | `peekFirst()` |

> [!warning] Do not use the legacy `Stack` class
> Java's `java.util.Stack` class extends `Vector` and is synchronized, making it slower than necessary. The official Java documentation recommends using `Deque` instead: "A more complete and consistent set of LIFO stack operations is provided by the Deque interface and its implementations, which should be used in preference to this class." Use `ArrayDeque` as a stack.

### `PriorityQueue` Internals

`PriorityQueue` is backed by a **binary heap** (specifically, a min-heap by default). The element with the highest priority (lowest value in natural ordering) is always at the head of the queue.

**How a binary heap works:**

A binary heap is a complete binary tree stored in an array. The parent-child relationship is defined by index arithmetic:
- Parent of node at index `i`: `(i - 1) / 2`
- Left child of node at index `i`: `2 * i + 1`
- Right child of node at index `i`: `2 * i + 2`

In a min-heap, every parent node is less than or equal to its children. This guarantees that the minimum element is always at index 0 (the root).

**Operations:**

1. **`offer(element)` (insertion)**: The new element is placed at the end of the array (the bottom of the tree) and then "bubbled up" (swapped with its parent) until the heap property is restored. O(log n).

2. **`poll()` (removal)**: The root element (minimum) is removed. The last element in the array is moved to the root and then "bubbled down" (swapped with the smaller child) until the heap property is restored. O(log n).

3. **`peek()` (examine)**: Returns the element at index 0. O(1).

**Memory layout:**

```text
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(30);
pq.offer(10);
pq.offer(20);
pq.offer(5);

Internal array (min-heap):
  Index:  0   1   2   3
  Value: [5, 10, 20, 30]

Tree representation:
        5
       / \
     10   20
     /
    30

The smallest element (5) is always at the root (index 0).
```

**Important characteristics:**

- `PriorityQueue` does NOT maintain insertion order. Iteration order is undefined and does not reflect priority order. Only `poll()` guarantees retrieval in priority order.
- Elements must be mutually comparable (implement `Comparable`) or a `Comparator` must be provided.
- `PriorityQueue` does NOT allow null elements.
- `PriorityQueue` is NOT thread-safe. Use `PriorityBlockingQueue` for concurrent access.

**Performance:**

| Operation | Time |
|-----------|------|
| `offer()` | O(log n) |
| `poll()` | O(log n) |
| `peek()` | O(1) |
| `remove(Object)` | O(n) (must search the entire heap) |
| `contains(Object)` | O(n) |

### `ArrayDeque` Internals

`ArrayDeque` is backed by a **circular resizable array**. It is the fastest general-purpose `Deque` implementation and should be preferred over `LinkedList` for both queue and stack operations.

**How a circular array works:**

The internal array has two pointers: `head` (index of the first element) and `tail` (index of the next available slot). When elements are added to the tail or removed from the head, the pointers advance circularly (wrapping around to index 0 when they reach the end of the array).

```text
ArrayDeque<String> deque = new ArrayDeque<>(4);
deque.addLast("A");
deque.addLast("B");
deque.addLast("C");

Internal array (capacity 4):
  Index:  0   1   2   3
  Value: [A,  B,  C,  null]
  head = 0, tail = 3

After deque.removeFirst():  // Removes "A"
  Index:  0   1   2   3
  Value: [null, B, C, null]
  head = 1, tail = 3

After deque.addLast("D"):
  Index:  0   1   2   3
  Value: [null, B, C, D]
  head = 1, tail = 0  (tail wrapped around!)

After deque.addLast("E"):
  Index:  0   1   2   3
  Value: [E, B, C, D]
  head = 1, tail = 1
  (Elements in order: B, C, D, E)
```

**Why `ArrayDeque` is faster than `LinkedList`:**

1. **Cache locality**: Elements are stored in a contiguous array, which fits into CPU cache lines. `LinkedList` nodes are scattered across the heap, causing cache misses.
2. **No per-element allocation**: `ArrayDeque` allocates one large array. `LinkedList` allocates a new `Node` object for every element, creating garbage collection pressure.
3. **Lower memory overhead**: `ArrayDeque` uses one reference per element (the array slot). `LinkedList` uses three references per element (the element, `prev`, and `next` pointers) plus object header overhead.

**Performance:**

| Operation | Time |
|-----------|------|
| `addFirst()` / `addLast()` | O(1) amortized |
| `removeFirst()` / `removeLast()` | O(1) |
| `peekFirst()` / `peekLast()` | O(1) |
| `size()` | O(1) |
| Random access by index | Not supported |

### `LinkedList` as a Queue and Deque

`LinkedList` implements both `List` and `Deque`. It is backed by a doubly-linked list and supports all queue and deque operations. However, it is slower than `ArrayDeque` for queue/stack operations due to poor cache locality and per-node allocation overhead. Use `LinkedList` only when you need both `List` and `Deque` functionality in the same data structure.

### Comparison of All Implementations

| Feature | PriorityQueue | ArrayDeque | LinkedList |
|---------|--------------|------------|------------|
| Backing structure | Binary heap | Circular array | Doubly-linked list |
| Ordering | Priority (min-heap) | Insertion order | Insertion order |
| `offer()` | O(log n) | O(1) amortized | O(1) |
| `poll()` | O(log n) | O(1) | O(1) |
| `peek()` | O(1) | O(1) | O(1) |
| Null elements | Not allowed | Not allowed | Allowed |
| Thread-safe | No | No | No |
| Use as stack | No | Yes (push/pop) | Yes (push/pop) |
| Use as queue | Yes (priority) | Yes (FIFO) | Yes (FIFO) |
| Random access | No | No | Yes (O(n), via List) |
| Best for | Priority-based processing | General queue/stack | Mixed List + Deque |

**Decision guide:**

- Need elements processed by priority? Use `PriorityQueue`.
- Need a fast FIFO queue or LIFO stack? Use `ArrayDeque`.
- Need both `List` and `Deque` operations? Use `LinkedList`.
- Need a thread-safe queue? Use `ConcurrentLinkedQueue` or `LinkedBlockingQueue`.
- Need a bounded blocking queue for producer-consumer? Use `ArrayBlockingQueue` or `LinkedBlockingQueue`.

### Blocking Queues (Brief Overview)

In multi-threaded backend applications, standard queues are not sufficient because they are not thread-safe. Java provides **blocking queues** in the `java.util.concurrent` package that support thread-safe insertion and retrieval with optional blocking (waiting) when the queue is full or empty.

| Implementation | Bounded | Backing Structure | Use Case |
|---------------|---------|-------------------|----------|
| `LinkedBlockingQueue` | Optional | Linked nodes | General-purpose concurrent queue |
| `ArrayBlockingQueue` | Yes | Circular array | Fixed-capacity concurrent queue |
| `PriorityBlockingQueue` | No | Binary heap | Priority-based concurrent queue |
| `SynchronousQueue` | Zero capacity | Direct handoff | Thread-to-thread transfer |
| `DelayQueue` | No | Binary heap | Delayed task execution |

Blocking queues are the foundation of producer-consumer patterns in backend systems. You will study them in detail in Phase 2 when you learn multithreading.

> [!tip] Key Insight
> The most important distinction between `Queue` and `Deque` is that `Queue` only allows insertion at the tail and removal from the head (FIFO), while `Deque` allows insertion and removal at both ends. This makes `Deque` a superset of `Queue` functionality. In practice, `ArrayDeque` (which implements `Deque`) is the best choice for almost all queue and stack use cases because it is faster than both `LinkedList` and the legacy `Stack` class. The only time you need a different implementation is when you need priority ordering (`PriorityQueue`) or thread safety (blocking queues).

---

## Syntax and Basic Examples

### Example 1: PriorityQueue basics

```java
import java.util.*;

public class PriorityQueueDemo {
    public static void main(String[] args) {
        // Min-heap (default): smallest element has highest priority
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        minHeap.offer(30);
        minHeap.offer(10);
        minHeap.offer(50);
        minHeap.offer(20);
        minHeap.offer(40);

        System.out.println("Peek (min): " + minHeap.peek());  // 10

        // poll() removes elements in priority order (smallest first)
        System.out.print("Poll order: ");
        while (!minHeap.isEmpty()) {
            System.out.print(minHeap.poll() + " ");
        }
        System.out.println();  // 10 20 30 40 50

        // Max-heap: largest element has highest priority
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
        maxHeap.offer(30);
        maxHeap.offer(10);
        maxHeap.offer(50);
        maxHeap.offer(20);

        System.out.print("Max-heap poll order: ");
        while (!maxHeap.isEmpty()) {
            System.out.print(maxHeap.poll() + " ");
        }
        System.out.println();  // 50 30 20 10

        // WARNING: iteration order is NOT priority order!
        PriorityQueue<Integer> pq = new PriorityQueue<>(List.of(30, 10, 50, 20));
        System.out.println("Iteration: " + pq);  // [10, 20, 50, 30] (heap order, not sorted!)
        // Only poll() guarantees priority order.
    }
}
```

### Example 2: PriorityQueue with custom objects

```java
import java.util.*;

public class PriorityTaskDemo {

    record Task(String name, int priority, String description) {}

    public static void main(String[] args) {
        // Lower priority number = higher urgency (processed first)
        PriorityQueue<Task> taskQueue = new PriorityQueue<>(
            Comparator.comparingInt(Task::priority)
        );

        taskQueue.offer(new Task("Fix login bug", 1, "Users cannot log in"));
        taskQueue.offer(new Task("Update README", 5, "Documentation update"));
        taskQueue.offer(new Task("Deploy hotfix", 2, "Critical security patch"));
        taskQueue.offer(new Task("Refactor utils", 4, "Code cleanup"));
        taskQueue.offer(new Task("Database migration", 3, "Schema update"));

        System.out.println("Processing tasks by priority:");
        while (!taskQueue.isEmpty()) {
            Task task = taskQueue.poll();
            System.out.printf("  [P%d] %s - %s%n",
                task.priority(), task.name(), task.description());
        }
        // Output:
        //   [P1] Fix login bug - Users cannot log in
        //   [P2] Deploy hotfix - Critical security patch
        //   [P3] Database migration - Schema update
        //   [P4] Refactor utils - Code cleanup
        //   [P5] Update README - Documentation update
    }
}
```

### Example 3: ArrayDeque as a queue (FIFO)

```java
import java.util.*;

public class ArrayDequeQueueDemo {
    public static void main(String[] args) {
        // ArrayDeque as a FIFO queue
        Deque<String> orderQueue = new ArrayDeque<>();

        // Enqueue: add orders to the tail
        orderQueue.offerLast("ORD-001");
        orderQueue.offerLast("ORD-002");
        orderQueue.offerLast("ORD-003");
        orderQueue.offerLast("ORD-004");

        System.out.println("Queue: " + orderQueue);  // [ORD-001, ORD-002, ORD-003, ORD-004]
        System.out.println("Front: " + orderQueue.peekFirst());  // ORD-001

        // Dequeue: process orders from the head (FIFO)
        while (!orderQueue.isEmpty()) {
            String order = orderQueue.pollFirst();
            System.out.println("Processing: " + order);
        }
        // Processing: ORD-001
        // Processing: ORD-002
        // Processing: ORD-003
        // Processing: ORD-004

        System.out.println("Queue empty: " + orderQueue.isEmpty());  // true
        System.out.println("Poll empty: " + orderQueue.pollFirst());  // null (no exception)
    }
}
```

### Example 4: ArrayDeque as a stack (LIFO)

```java
import java.util.*;

public class ArrayDequeStackDemo {
    public static void main(String[] args) {
        // ArrayDeque as a LIFO stack (replaces the legacy Stack class)
        Deque<String> callStack = new ArrayDeque<>();

        // Push: add frames to the top (head)
        callStack.push("main()");
        callStack.push("createOrder()");
        callStack.push("validatePayment()");
        callStack.push("chargeCard()");

        System.out.println("Call stack: " + callStack);
        // [chargeCard(), validatePayment(), createOrder(), main()]
        System.out.println("Top: " + callStack.peek());  // chargeCard()

        // Pop: remove frames from the top (LIFO)
        while (!callStack.isEmpty()) {
            System.out.println("Returning from: " + callStack.pop());
        }
        // Returning from: chargeCard()
        // Returning from: validatePayment()
        // Returning from: createOrder()
        // Returning from: main()
    }
}
```

### Example 5: Deque operations at both ends

```java
import java.util.*;

public class DequeBothEndsDemo {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();

        // Add at both ends
        deque.addFirst("B");
        deque.addFirst("A");       // Head: A, B
        deque.addLast("C");        // Head: A, B, C
        deque.addLast("D");        // Head: A, B, C, D

        System.out.println("Deque: " + deque);  // [A, B, C, D]
        System.out.println("First: " + deque.peekFirst());  // A
        System.out.println("Last: " + deque.peekLast());    // D

        // Remove from both ends
        System.out.println("Remove first: " + deque.pollFirst());  // A
        System.out.println("Remove last: " + deque.pollLast());    // D
        System.out.println("Remaining: " + deque);  // [B, C]

        // Practical use: sliding window
        // Maintain a window of the last 3 events
        Deque<String> window = new ArrayDeque<>();
        String[] events = {"click", "scroll", "hover", "click", "submit"};

        for (String event : events) {
            window.addLast(event);
            if (window.size() > 3) {
                window.removeFirst();  // Remove the oldest event
            }
            System.out.println("Window: " + window);
        }
        // Window: [click]
        // Window: [click, scroll]
        // Window: [click, scroll, hover]
        // Window: [scroll, hover, click]
        // Window: [hover, click, submit]
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Queues and deques are fundamental to backend architectures. Here are three realistic scenarios.

### Scenario 1: Task scheduling with PriorityQueue

Backend systems often need to process tasks based on priority rather than arrival order. A `PriorityQueue` ensures that the most urgent tasks are processed first.

```java
package com.company.orderservice.scheduler;

import org.springframework.stereotype.Service;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.PriorityQueue;

@Service
public class TaskScheduler {

    // Thread-safe priority queue for concurrent task scheduling
    private final PriorityQueue<ScheduledTask> taskQueue =
        new PriorityQueue<>(Comparator.comparing(ScheduledTask::executeAt));

    public synchronized void scheduleTask(String taskName, Runnable action,
                                           LocalDateTime executeAt, int priority) {
        taskQueue.offer(new ScheduledTask(taskName, action, executeAt, priority));
    }

    public synchronized List<ScheduledTask> getDueTasks() {
        LocalDateTime now = LocalDateTime.now();
        List<ScheduledTask> dueTasks = new ArrayList<>();

        // Peek at the head: if it is not due yet, no tasks are due
        // (because the queue is sorted by execution time)
        while (!taskQueue.isEmpty() && !taskQueue.peek().executeAt().isAfter(now)) {
            dueTasks.add(taskQueue.poll());
        }

        return dueTasks;
    }

    public int getPendingTaskCount() {
        return taskQueue.size();
    }
}

record ScheduledTask(
    String name,
    Runnable action,
    LocalDateTime executeAt,
    int priority
) {}
```

```java
// Usage in a scheduled job:
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.util.List;

@Component
public class TaskProcessor {

    private final TaskScheduler scheduler;

    @Scheduled(fixedRate = 5000)  // Run every 5 seconds
    public void processDueTasks() {
        List<ScheduledTask> dueTasks = scheduler.getDueTasks();

        for (ScheduledTask task : dueTasks) {
            try {
                task.action().run();
                // logger.info("Executed task: {}", task.name());
            } catch (Exception e) {
                // logger.error("Task failed: {}", task.name(), e);
                // Reschedule with lower priority
                scheduler.scheduleTask(
                    task.name() + " (retry)",
                    task.action(),
                    LocalDateTime.now().plusMinutes(5),
                    task.priority() + 1
                );
            }
        }
    }
}
```

**What to notice:**

- The `PriorityQueue` is sorted by `executeAt` time. The `peek()` method returns the task with the earliest execution time. This allows the scheduler to efficiently check if any tasks are due without scanning the entire queue.
- The `while` loop in `getDueTasks()` polls tasks from the head as long as they are due. Because the queue is sorted, once a task is found that is not yet due, all remaining tasks are also not due, so the loop can stop.
- In production, you would use a thread-safe queue like `PriorityBlockingQueue` or a distributed task scheduler like Quartz or Spring's `@Scheduled` with a database-backed job store.

### Scenario 2: Event processing pipeline with ArrayDeque

Event-driven backends use queues to buffer events between producers and consumers. An `ArrayDeque` provides fast FIFO processing for in-memory event buffers.

```java
package com.company.orderservice.events;

import org.springframework.stereotype.Service;
import java.util.ArrayDeque;
import java.util.Deque;

@Service
public class EventProcessor {

    private final Deque<DomainEvent> eventBuffer = new ArrayDeque<>();
    private static final int MAX_BUFFER_SIZE = 10000;

    // Producer: add events to the tail of the queue
    public synchronized boolean enqueue(DomainEvent event) {
        if (eventBuffer.size() >= MAX_BUFFER_SIZE) {
            // logger.warn("Event buffer full. Dropping event: {}", event.getEventId());
            return false;  // Buffer is full, reject the event
        }
        eventBuffer.offerLast(event);
        return true;
    }

    // Consumer: process events from the head of the queue (FIFO)
    public synchronized int processBatch(int batchSize) {
        int processed = 0;

        while (processed < batchSize && !eventBuffer.isEmpty()) {
            DomainEvent event = eventBuffer.pollFirst();
            try {
                dispatchEvent(event);
                processed++;
            } catch (Exception e) {
                // logger.error("Failed to process event: {}", event.getEventId(), e);
                // Optionally re-enqueue at the tail for retry
                eventBuffer.offerLast(event);
                break;  // Stop processing to avoid infinite retry loop
            }
        }

        return processed;
    }

    public synchronized int getBufferSize() {
        return eventBuffer.size();
    }

    private void dispatchEvent(DomainEvent event) {
        // Route the event to the appropriate handler based on its type
        // This uses polymorphism: each event type has its own handler
        // logger.info("Dispatching event: {} ({})", event.getEventId(), event.getClass().getSimpleName());
    }
}
```

**What to notice:**

- `offerLast()` adds events to the tail (enqueue). `pollFirst()` removes events from the head (dequeue). This is the standard FIFO pattern.
- The buffer has a maximum size to prevent memory exhaustion. When the buffer is full, new events are rejected. In production, you would use a bounded blocking queue (`ArrayBlockingQueue`) or a message broker (RabbitMQ, Kafka) instead of an in-memory deque.
- The `synchronized` keyword ensures thread safety. In a real backend, you would use a concurrent queue from `java.util.concurrent` instead of manual synchronization.

### Scenario 3: Sliding window rate limiter with Deque

A sliding window rate limiter uses a deque to track request timestamps and enforce rate limits per client.

```java
package com.company.orderservice.security;

import org.springframework.stereotype.Service;
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Service
public class SlidingWindowRateLimiter {

    // Map of client ID to their request timestamp deque
    private final Map<String, Deque<Long>> clientWindows = new ConcurrentHashMap<>();
    private final int maxRequests;
    private final long windowSizeMs;

    public SlidingWindowRateLimiter(int maxRequests, long windowSizeMs) {
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
    }

    public boolean isAllowed(String clientId) {
        long now = System.currentTimeMillis();

        Deque<Long> timestamps = clientWindows.computeIfAbsent(
            clientId, k -> new ArrayDeque<>()
        );

        synchronized (timestamps) {
            // Remove timestamps that are outside the sliding window
            // pollFirst() removes from the head (oldest timestamps)
            while (!timestamps.isEmpty() && now - timestamps.peekFirst() > windowSizeMs) {
                timestamps.pollFirst();
            }

            // Check if the client has exceeded the rate limit
            if (timestamps.size() >= maxRequests) {
                return false;  // Rate limit exceeded
            }

            // Record this request
            timestamps.offerLast(now);
            return true;
        }
    }

    public int getRemainingRequests(String clientId) {
        Deque<Long> timestamps = clientWindows.get(clientId);
        if (timestamps == null) return maxRequests;

        long now = System.currentTimeMillis();
        synchronized (timestamps) {
            while (!timestamps.isEmpty() && now - timestamps.peekFirst() > windowSizeMs) {
                timestamps.pollFirst();
            }
            return Math.max(0, maxRequests - timestamps.size());
        }
    }
}
```

```java
// Usage in a filter or interceptor:
import org.springframework.stereotype.Component;

@Component
public class RateLimitFilter {

    private final SlidingWindowRateLimiter rateLimiter =
        new SlidingWindowRateLimiter(60, 60_000);  // 60 requests per minute

    public void checkRateLimit(String clientId) {
        if (!rateLimiter.isAllowed(clientId)) {
            throw new RateLimitExceededException(
                "Rate limit exceeded. Try again in 60 seconds.",
                rateLimiter.getRemainingRequests(clientId)
            );
        }
    }
}
```

**What to notice:**

- The `Deque<Long>` stores request timestamps in chronological order. The oldest timestamp is at the head, the newest at the tail.
- `peekFirst()` checks the oldest timestamp without removing it. `pollFirst()` removes timestamps that have fallen outside the sliding window. This is the key advantage of a deque: efficient removal from the head.
- `offerLast(now)` adds the current timestamp to the tail. The deque grows and shrinks dynamically as requests arrive and old timestamps expire.
- This sliding window approach is more accurate than a fixed window counter because it tracks the exact timestamps of individual requests rather than aggregating them into time buckets.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using `LinkedList` instead of `ArrayDeque` for queue/stack operations

**Wrong:**

```java
// Using LinkedList as a queue (slower due to cache misses and node allocation)
Queue<String> queue = new LinkedList<>();
queue.offer("task1");
queue.offer("task2");
String task = queue.poll();

// Using the legacy Stack class (synchronized, extends Vector, slow)
Stack<String> stack = new Stack<>();
stack.push("frame1");
stack.push("frame2");
String frame = stack.pop();
```

**Right:**

```java
// ArrayDeque is faster for both queue and stack operations
Queue<String> queue = new ArrayDeque<>();
queue.offer("task1");
queue.offer("task2");
String task = queue.poll();

Deque<String> stack = new ArrayDeque<>();
stack.push("frame1");
stack.push("frame2");
String frame = stack.pop();
```

**Why it is wrong:** `LinkedList` allocates a new `Node` object for every element, causing garbage collection pressure and cache misses. `ArrayDeque` stores elements in a contiguous array, which is cache-friendly and allocation-efficient. Benchmarks consistently show `ArrayDeque` outperforming `LinkedList` by 2-5x for queue and stack operations. The legacy `Stack` class is even worse because it extends `Vector`, which synchronizes every method call.

### Mistake 2: Expecting PriorityQueue iteration to return elements in sorted order

**Wrong:**

```java
PriorityQueue<Integer> pq = new PriorityQueue<>(List.of(50, 30, 10, 40, 20));

// Expecting sorted output from iteration
for (int n : pq) {
    System.out.print(n + " ");
}
// Actual output: 10 20 30 40 50 (might look sorted by coincidence)
// But with different data: 10 30 50 40 20 (NOT sorted!)
// Iteration order is the internal heap order, which is NOT sorted.
```

**Right:**

```java
PriorityQueue<Integer> pq = new PriorityQueue<>(List.of(50, 30, 10, 40, 20));

// Only poll() guarantees priority order
while (!pq.isEmpty()) {
    System.out.print(pq.poll() + " ");
}
// Output: 10 20 30 40 50 (guaranteed sorted)

// If you need a sorted view without destroying the queue:
Object[] sorted = pq.stream().sorted().toArray();
```

**Why it is wrong:** A `PriorityQueue` is a binary heap, not a sorted list. The heap property only guarantees that the minimum element is at the root (index 0). The rest of the elements are in a partial order that satisfies the heap invariant but is not fully sorted. Iteration traverses the internal array in index order, which does not correspond to priority order. Only `poll()` extracts elements in priority order by repeatedly removing the root and rebalancing the heap.

### Mistake 3: Using `add()`/`remove()` instead of `offer()`/`poll()`

**Wrong:**

```java
Queue<String> queue = new ArrayDeque<>();
String item = queue.remove();  // NoSuchElementException! Queue is empty.
```

**Right:**

```java
Queue<String> queue = new ArrayDeque<>();
String item = queue.poll();  // Returns null. No exception.
if (item != null) {
    process(item);
}
```

**Why it is wrong:** The `add()`, `remove()`, and `element()` methods throw exceptions when the operation cannot be performed (queue full or empty). The `offer()`, `poll()`, and `peek()` methods return `false` or `null` instead. In backend code, you almost always want the non-throwing variants because empty queues are a normal condition, not an exceptional one. Throwing and catching exceptions for expected conditions is expensive and clutters the code.

### Mistake 4: Not considering thread safety for shared queues

**Wrong:**

```java
@Service
public class EventService {
    // Shared across all request threads! Not thread-safe!
    private final Queue<DomainEvent> eventQueue = new ArrayDeque<>();

    public void publish(DomainEvent event) {
        eventQueue.offer(event);  // Race condition with concurrent consumers
    }

    public DomainEvent consume() {
        return eventQueue.poll();  // Race condition with concurrent producers
    }
}
```

**Right:**

```java
@Service
public class EventService {
    // Thread-safe concurrent queue
    private final Queue<DomainEvent> eventQueue = new ConcurrentLinkedQueue<>();

    public void publish(DomainEvent event) {
        eventQueue.offer(event);  // Thread-safe
    }

    public DomainEvent consume() {
        return eventQueue.poll();  // Thread-safe
    }
}
```

**Why it is wrong:** `ArrayDeque`, `PriorityQueue`, and `LinkedList` are not thread-safe. When multiple threads access them concurrently, the internal data structures can become corrupted, leading to lost events, infinite loops, or `NullPointerException` inside the queue's internal code. Use concurrent queues from `java.util.concurrent` for shared queues in multi-threaded backends.

---

## Key Takeaways

> [!tip] Remember these points
> 1. **`Queue`** is a FIFO collection for holding elements prior to processing. Use `offer()`/`poll()`/`peek()` (non-throwing) instead of `add()`/`remove()`/`element()` (throwing) for graceful handling of empty/full conditions.
> 2. **`Deque`** extends `Queue` with operations at both ends. It can function as both a FIFO queue (`offerLast`/`pollFirst`) and a LIFO stack (`push`/`pop`). `ArrayDeque` is the fastest general-purpose implementation for both use cases.
> 3. **`PriorityQueue`** processes elements by priority (min-heap by default). `offer()` and `poll()` are O(log n). `peek()` is O(1). Iteration does NOT return elements in priority order; only `poll()` guarantees priority ordering. Use a custom `Comparator` for max-heap or domain-specific priority.
> 4. **`ArrayDeque`** is backed by a circular resizable array. It is faster than `LinkedList` for all queue and stack operations due to cache locality and lower memory overhead. It does not allow null elements. Use it as the default choice for FIFO queues and LIFO stacks.
> 5. Standard queues and deques are **not thread-safe**. In multi-threaded backend applications, use concurrent implementations: `ConcurrentLinkedQueue` for unbounded non-blocking queues, `ArrayBlockingQueue` or `LinkedBlockingQueue` for bounded blocking queues, and `PriorityBlockingQueue` for concurrent priority queues.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: PriorityQueue Task Scheduler (Easy)

Create a `Task` record with fields `name` (String), `priority` (int, 1 = highest), and `estimatedMinutes` (int). Create a `PriorityQueue<Task>` that processes tasks by priority (lowest number first). For tasks with the same priority, process the one with the shorter estimated time first. Add 6-8 tasks with varying priorities and durations. Poll all tasks and print them in processing order.

> **Hint:** Use `Comparator.comparingInt(Task::priority).thenComparingInt(Task::estimatedMinutes)` for the comparator.

### Exercise 2: Sliding Window Average with Deque (Medium)

Implement a `SlidingWindowAverage` class that maintains a sliding window of the last `n` numbers and calculates the running average:

1. Constructor takes the window size `n`.
2. `addNumber(double number)`: adds a number to the window. If the window is full, removes the oldest number.
3. `getAverage()`: returns the average of the numbers currently in the window.
4. `getMin()` and `getMax()`: return the minimum and maximum values in the current window.

Use an `ArrayDeque<Double>` internally. Test with a stream of 20 numbers and a window size of 5, printing the average after each addition.

> **Hint:** Maintain a running sum variable that you update on each `addNumber()` call (add the new number, subtract the removed number if the window was full). This avoids recalculating the sum from scratch on every call.

### Exercise 3: BFS Graph Traversal with Queue (Medium)

Implement a breadth-first search (BFS) algorithm to find the shortest path between two nodes in a graph. Represent the graph as a `Map<String, List<String>>` (adjacency list). Use an `ArrayDeque<String>` as the BFS queue.

Graph example:

```text
Dhaka -> [Chittagong, Sylhet, Rajshahi]
Chittagong -> [Dhaka, Cox's Bazar]
Sylhet -> [Dhaka, Rangpur]
Rajshahi -> [Dhaka, Rangpur]
Rangpur -> [Sylhet, Rajshahi]
Cox's Bazar -> [Chittagong]
```

Find the shortest path from "Dhaka" to "Rangpur". Print the path and the number of hops.

> **Hint:** Use a `Queue<String>` for BFS and a `Map<String, String>` to track the parent of each visited node (for path reconstruction). Start by enqueuing the source node, then dequeue and enqueue all unvisited neighbors until you reach the target.

### Exercise 4: Multi-Priority Event Dispatcher (Hard, Optional)

Build an event dispatcher that processes events from multiple priority levels:

1. Create three `PriorityQueue<Event>` instances: CRITICAL (priority 1), HIGH (priority 2), NORMAL (priority 3).
2. Create an `Event` record with fields `id`, `type`, `payload`, `priority`, `timestamp`.
3. Implement a `dispatch()` method that always processes the highest-priority queue first. Only process lower-priority events when all higher-priority queues are empty.
4. Implement a `processBatch(int maxEvents)` method that processes up to `maxEvents` events across all priority levels.
5. Add 15-20 events with mixed priorities and timestamps. Process them in batches of 5 and print the processing order.

> **Hint:** The `dispatch()` method checks queues in priority order: if the CRITICAL queue is not empty, poll from it; otherwise check HIGH; otherwise check NORMAL. This ensures critical events are always processed before lower-priority ones, regardless of arrival time.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.*;

record Task(String name, int priority, int estimatedMinutes) {}

public class Main {
    public static void main(String[] args) {
        PriorityQueue<Task> queue = new PriorityQueue<>(
            Comparator.comparingInt(Task::priority)
                .thenComparingInt(Task::estimatedMinutes)
        );

        queue.offer(new Task("Deploy hotfix", 1, 15));
        queue.offer(new Task("Fix login bug", 1, 30));
        queue.offer(new Task("Database backup", 2, 60));
        queue.offer(new Task("Code review", 2, 45));
        queue.offer(new Task("Update docs", 3, 20));
        queue.offer(new Task("Refactor utils", 3, 90));
        queue.offer(new Task("Team meeting", 2, 30));

        System.out.println("Processing order:");
        while (!queue.isEmpty()) {
            Task t = queue.poll();
            System.out.printf("  [P%d, %dmin] %s%n", t.priority(), t.estimatedMinutes(), t.name());
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.util.*;

public class SlidingWindowAverage {
    private final Deque<Double> window;
    private final int maxSize;
    private double sum;

    public SlidingWindowAverage(int maxSize) {
        this.window = new ArrayDeque<>(maxSize);
        this.maxSize = maxSize;
        this.sum = 0;
    }

    public void addNumber(double number) {
        if (window.size() >= maxSize) {
            sum -= window.pollFirst();
        }
        window.offerLast(number);
        sum += number;
    }

    public double getAverage() {
        return window.isEmpty() ? 0 : sum / window.size();
    }

    public double getMin() {
        return window.stream().mapToDouble(Double::doubleValue).min().orElse(0);
    }

    public double getMax() {
        return window.stream().mapToDouble(Double::doubleValue).max().orElse(0);
    }

    public static void main(String[] args) {
        SlidingWindowAverage swa = new SlidingWindowAverage(5);
        double[] data = {10, 20, 30, 40, 50, 60, 70, 80, 90, 100};

        for (double n : data) {
            swa.addNumber(n);
            System.out.printf("Added %5.0f | Avg: %5.1f | Min: %5.0f | Max: %5.0f | Window: %s%n",
                n, swa.getAverage(), swa.getMin(), swa.getMax(), swa.window);
        }
    }
}
```

</details>

---

## Related Notes

- [[Java - Map - HashMap TreeMap LinkedHashMap]]
- [[Java - Generics - Classes Methods Wildcards]] (next note)
- [[Java - Comparable and Comparator]]
- [[Java - Multithreading Basics - Thread Runnable]] (Phase 2)

---

## Resources

- [Oracle Java Tutorials: The Queue Interface](https://docs.oracle.com/javase/tutorial/collections/interfaces/queue.html) - Official documentation covering Queue and Deque contracts.
- [Oracle Java Documentation: PriorityQueue](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/PriorityQueue.html) - Complete API reference with implementation details.
- [Oracle Java Documentation: ArrayDeque](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html) - Complete API reference.
- [Baeldung: Java Queue Guide](https://www.baeldung.com/java-queue) - Comprehensive guide covering all Queue implementations.
- [Baeldung: Java Deque Guide](https://www.baeldung.com/java-deque) - Detailed guide on Deque operations and use cases.
- [Baeldung: Java PriorityQueue](https://www.baeldung.com/java-priority-queue) - In-depth guide with custom comparator examples.
- [Java Concurrency in Practice by Brian Goetz - Chapter 5: Building Blocks](https://www.oreilly.com/library/view/java-concurrency-in/0321349601/) - Essential reading on blocking queues and producer-consumer patterns for backend systems.
