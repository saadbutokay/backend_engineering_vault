## Overview

Concurrency is the ability of a program to execute multiple tasks simultaneously. In a backend fintech system, concurrency is not a luxury — it is a necessity. Your application must handle thousands of simultaneous HTTP requests, process payment transactions in parallel, consume messages from Kafka while serving API responses, and run scheduled jobs without blocking user-facing operations. Java's concurrency model is one of its greatest strengths and one of the primary reasons it dominates enterprise and fintech backends. Unlike Python, which is constrained by the Global Interpreter Lock (GIL), Java provides true multithreading with full access to all CPU cores. This section covers the complete concurrency landscape: from the fundamentals of threads and synchronization to the modern `java.util.concurrent` framework and Java 21's virtual threads.

---

## Core Concepts

### Processes vs Threads

**Process:**
- An independent execution environment with its own memory space (heap, stack, code, data).
- The JVM itself is a single process. When you run `java -jar myapp.jar`, the operating system creates one process.
- Processes are isolated. One process cannot directly access another process's memory. Communication requires inter-process mechanisms (sockets, pipes, shared memory).
- Creating a process is expensive (memory allocation, OS scheduling overhead).

**Thread:**
- A unit of execution within a process. A single process can contain many threads.
- Threads within the same process share the heap and method area but have their own stack and PC register.
- Thread creation is cheaper than process creation but still involves OS-level resources (typically 1 MB of stack memory per thread).
- Because threads share memory, they can communicate directly by reading and writing shared variables. This is both a feature and a danger — shared mutable state is the root cause of most concurrency bugs.

```
JVM Process
├── Heap (shared by all threads)
│   ├── Objects
│   ├── Arrays
│   └── Class metadata
├── Method Area (shared)
│   └── Bytecode, static variables
├── Thread 1
│   ├── Stack (private)
│   │   ├── main() frame
│   │   └── processPayment() frame
│   └── PC Register (private)
├── Thread 2
│   ├── Stack (private)
│   │   └── handleRequest() frame
│   └── PC Register (private)
└── Thread 3
    ├── Stack (private)
    │   └── consumeMessage() frame
    └── PC Register (private)
```

### Creating Threads

**Method 1: Extending Thread**

```java
public class PaymentProcessor extends Thread {
    private final PaymentRequest request;

    public PaymentProcessor(PaymentRequest request) {
        this.request = request;
    }

    @Override
    public void run() {
        System.out.println("Processing payment on thread: " + Thread.currentThread().getName());
        processPayment(request);
    }
}

// Usage
PaymentProcessor processor = new PaymentProcessor(request);
processor.start();  // Creates a new OS thread and calls run()
// processor.run();  // WRONG — this executes on the current thread, no new thread
```

**Method 2: Implementing Runnable (preferred)**

```java
public class PaymentTask implements Runnable {
    private final PaymentRequest request;

    public PaymentTask(PaymentRequest request) {
        this.request = request;
    }

    @Override
    public void run() {
        processPayment(request);
    }
}

// Usage
Thread thread = new Thread(new PaymentTask(request));
thread.setName("payment-processor-1");
thread.start();

// Lambda (most concise)
Thread thread = new Thread(() -> processPayment(request));
thread.start();
```

**Why Runnable is preferred over extending Thread:**

- Java supports single inheritance. If your class extends `Thread`, it cannot extend any other class. Implementing `Runnable` leaves the inheritance slot free.
- `Runnable` separates the task (what to execute) from the thread (how to execute it). The same `Runnable` can be submitted to a thread pool, an executor, or a virtual thread.
- `Runnable` is a functional interface, so you can use lambdas.

**Method 3: Implementing Callable (when you need a return value)**

```java
public class BalanceCalculator implements Callable<BigDecimal> {
    private final String accountId;

    public BalanceCalculator(String accountId) {
        this.accountId = accountId;
    }

    @Override
    public BigDecimal call() throws Exception {
        // call() can return a value and throw checked exceptions
        return accountRepository.calculateBalance(accountId);
    }
}
```

`Callable` is used with `ExecutorService` (covered below). You cannot pass a `Callable` directly to the `Thread` constructor.

### Thread Lifecycle

A thread passes through six states defined in `Thread.State`:

```
NEW → RUNNABLE → (BLOCKED | WAITING | TIMED_WAITING) → RUNNABLE → TERMINATED
```

| State | Description | Example |
|-------|-------------|---------|
| `NEW` | Thread created but `start()` not yet called | `Thread t = new Thread(task);` |
| `RUNNABLE` | Executing or ready to execute (waiting for CPU time) | Thread is running your code |
| `BLOCKED` | Waiting to acquire a monitor lock | Thread is waiting to enter a `synchronized` block |
| `WAITING` | Waiting indefinitely for another thread's action | `Object.wait()`, `Thread.join()`, `LockSupport.park()` |
| `TIMED_WAITING` | Waiting for a specified time | `Thread.sleep(1000)`, `Object.wait(1000)`, `Thread.join(1000)` |
| `TERMINATED` | Execution completed (normally or via exception) | `run()` method returned or threw an uncaught exception |

```java
Thread thread = new Thread(() -> {
    try {
        Thread.sleep(1000);  // TIMED_WAITING
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

System.out.println(thread.getState());  // NEW
thread.start();
System.out.println(thread.getState());  // RUNNABLE (or TIMED_WAITING)
thread.join();
System.out.println(thread.getState());  // TERMINATED
```

### Thread Methods

```java
Thread current = Thread.currentThread();
current.getName();       // "main"
current.getId();         // Unique thread ID (long)
current.getPriority();   // 1-10, default 5 (NORM_PRIORITY)
current.isDaemon();      // false for user threads, true for daemon threads
current.isAlive();       // true if started and not yet terminated
current.isInterrupted(); // true if interrupt flag is set

// Static methods
Thread.sleep(1000);      // Pause current thread for 1000 ms
Thread.yield();          // Hint to scheduler to switch to another thread
Thread.onSpinWait();     // Hint for spin-wait loops (Java 9+)
```

### Daemon Threads

Daemon threads are background threads that do not prevent the JVM from exiting. When all user (non-daemon) threads terminate, the JVM shuts down, regardless of whether daemon threads are still running.

```java
Thread monitor = new Thread(() -> {
    while (true) {
        checkSystemHealth();
        try { Thread.sleep(5000); } catch (InterruptedException e) { break; }
    }
});
monitor.setDaemon(true);  // Must be set BEFORE start()
monitor.start();
```

Use daemon threads for background tasks like metrics collection, cache cleanup, and health monitoring. Do not use them for tasks that must complete (e.g., writing data to disk).

### Interrupts

An interrupt is a cooperative mechanism for signaling a thread to stop. It does not forcibly terminate the thread — it sets a flag that the thread should check.

```java
Thread worker = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        try {
            processNextTransaction();
            Thread.sleep(100);
        } catch (InterruptedException e) {
            // sleep() clears the interrupt flag when it throws
            // Re-set it so callers know the thread was interrupted
            Thread.currentThread().interrupt();
            break;
        }
    }
    cleanup();
});

worker.start();
// Later, when you want to stop the worker:
worker.interrupt();
```

**Key rules:**
- `Thread.interrupt()` sets the interrupt flag on the target thread.
- `Thread.sleep()`, `Object.wait()`, and `Thread.join()` check the flag and throw `InterruptedException` if it is set. They also clear the flag.
- When you catch `InterruptedException`, you should either re-set the flag (`Thread.currentThread().interrupt()`) or handle the interruption by exiting the thread. Swallowing the interrupt silently is a bug.
- `Thread.stop()` is deprecated and dangerous. Never use it. It can leave shared data in an inconsistent state.

---

### Synchronization

When multiple threads access shared mutable state, race conditions occur. A race condition is a situation where the outcome depends on the unpredictable timing of thread execution.

**The race condition problem:**

```java
public class BankAccount {
    private BigDecimal balance = BigDecimal.ZERO;

    public void deposit(BigDecimal amount) {
        // This is NOT atomic. It involves:
        // 1. Read balance
        // 2. Add amount
        // 3. Write new balance
        balance = balance.add(amount);
    }

    public BigDecimal getBalance() {
        return balance;
    }
}

// If two threads call deposit(100) simultaneously on the same account:
// Thread 1 reads balance = 0
// Thread 2 reads balance = 0
// Thread 1 writes balance = 100
// Thread 2 writes balance = 100  (should be 200!)
// Result: $100 is lost. In fintech, this is a catastrophic bug.
```

**The `synchronized` keyword:**

`synchronized` ensures that only one thread can execute a block of code at a time by acquiring a monitor lock (intrinsic lock) on an object.

```java
public class BankAccount {
    private BigDecimal balance = BigDecimal.ZERO;
    private final Object lock = new Object();

    // Synchronized method — locks on 'this'
    public synchronized void deposit(BigDecimal amount) {
        balance = balance.add(amount);
    }

    public synchronized void withdraw(BigDecimal amount) {
        if (amount.compareTo(balance) > 0) {
            throw new IllegalStateException("Insufficient funds");
        }
        balance = balance.subtract(amount);
    }

    public synchronized BigDecimal getBalance() {
        return balance;
    }

    // Synchronized block — locks on a specific object
    public void transfer(BankAccount target, BigDecimal amount) {
        synchronized (this.lock) {
            this.withdraw(amount);
            synchronized (target.lock) {
                target.deposit(amount);
            }
        }
    }
}
```

**Rules for `synchronized`:**

- Every object in Java has an intrinsic lock (monitor). `synchronized` acquires this lock.
- A `synchronized` instance method locks on `this`.
- A `synchronized` static method locks on the `Class` object (`MyClass.class`).
- A `synchronized (obj)` block locks on the specified object.
- Locks are reentrant. A thread that holds a lock can re-enter a `synchronized` block on the same object without deadlocking.
- Only one thread can hold a lock at a time. Other threads block until the lock is released.
- The lock is released when the thread exits the `synchronized` block or method, either normally or via an exception.

**The `volatile` keyword:**

`volatile` ensures that reads and writes to a variable are visible across all threads. Without `volatile`, the JVM may cache a variable's value in a CPU register or local cache, causing one thread to see a stale value.

```java
public class ShutdownFlag {
    private volatile boolean running = true;

    public void shutdown() {
        running = false;  // Immediately visible to all threads
    }

    public void workerLoop() {
        while (running) {  // Always reads the latest value
            processTask();
        }
    }
}
```

**`volatile` guarantees:**
- Visibility: writes to a `volatile` variable are immediately visible to all threads.
- Ordering: prevents instruction reordering around the volatile access.

**`volatile` does NOT guarantee:**
- Atomicity: `volatile int count; count++;` is NOT atomic. The increment involves a read, add, and write — three separate operations that can be interleaved.

**When to use `volatile`:**
- Flags and status variables that are written by one thread and read by many.
- Double-checked locking for lazy initialization.
- When you need visibility but not atomicity.

**When NOT to use `volatile`:**
- When you need atomic read-modify-write operations (use `synchronized` or `AtomicInteger`).
- When multiple variables must be updated together consistently (use `synchronized`).

### Deadlock

A deadlock occurs when two or more threads are blocked forever, each waiting for a lock held by the other.

```java
public class DeadlockDemo {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    public void method1() {
        synchronized (lockA) {
            System.out.println("Thread 1: Holding lockA, waiting for lockB");
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            synchronized (lockB) {
                System.out.println("Thread 1: Holding both locks");
            }
        }
    }

    public void method2() {
        synchronized (lockB) {
            System.out.println("Thread 2: Holding lockB, waiting for lockA");
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            synchronized (lockA) {
                System.out.println("Thread 2: Holding both locks");
            }
        }
    }
}

// Thread 1 calls method1(): acquires lockA, waits for lockB
// Thread 2 calls method2(): acquires lockB, waits for lockA
// Neither can proceed. Deadlock.
```

**Preventing deadlock:**

1. **Lock ordering.** Always acquire locks in a consistent global order. If all threads acquire `lockA` before `lockB`, deadlock is impossible.
2. **Lock timeout.** Use `tryLock(timeout)` from `java.util.concurrent.locks.ReentrantLock` instead of `synchronized`. If the lock is not acquired within the timeout, back off and retry.
3. **Avoid nested locks.** Minimize the number of locks held simultaneously.
4. **Use higher-level concurrency utilities.** `ConcurrentHashMap`, `AtomicReference`, and other `java.util.concurrent` classes handle locking internally and are deadlock-free by design.

### Livelock and Starvation

**Livelock:** Threads are not blocked but are stuck in a loop of responding to each other's actions without making progress. Like two people in a hallway who keep stepping to the same side to let the other pass.

**Starvation:** A thread is perpetually denied access to a resource because higher-priority threads monopolize it. For example, a low-priority thread waiting for a lock that high-priority threads continuously acquire and release.

---

### java.util.concurrent — The Executor Framework

The Executor framework (Java 5+) is the modern way to manage threads. It separates task submission from thread management, providing thread pooling, lifecycle management, and result handling.

**Why not create threads manually:**

- Creating an OS thread is expensive (~1 MB stack, kernel-level context switch).
- Unbounded thread creation leads to `OutOfMemoryError` or OS thread limit exhaustion.
- Manual thread lifecycle management is error-prone.
- No built-in mechanism for task queuing, result retrieval, or graceful shutdown.

**Core interfaces:**

```
Executor
└── ExecutorService
    └── ScheduledExecutorService
```

**ExecutorService:**

```java
// Fixed thread pool — exactly N threads
ExecutorService fixedPool = Executors.newFixedThreadPool(10);

// Cached thread pool — creates threads as needed, reuses idle threads
// WARNING: unbounded — can create thousands of threads under load
ExecutorService cachedPool = Executors.newCachedThreadPool();

// Single thread — tasks execute sequentially
ExecutorService singleThread = Executors.newSingleThreadExecutor();

// Scheduled pool — for delayed and periodic tasks
ScheduledExecutorService scheduledPool = Executors.newScheduledThreadPool(4);

// Custom thread pool (recommended for production)
ExecutorService customPool = new ThreadPoolExecutor(
    4,                          // Core pool size (threads kept alive even when idle)
    16,                         // Maximum pool size
    60L, TimeUnit.SECONDS,      // Idle thread keep-alive time
    new LinkedBlockingQueue<>(1000),  // Task queue
    new ThreadFactory() {
        private final AtomicInteger counter = new AtomicInteger(1);
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, "payment-processor-" + counter.getAndIncrement());
            t.setDaemon(false);
            return t;
        }
    },
    new ThreadPoolExecutor.CallerRunsPolicy()  // Rejection policy
);
```

**Submitting tasks:**

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

// Submit a Runnable (no return value)
executor.submit(() -> processPayment(request));

// Submit a Callable (returns a Future)
Future<BigDecimal> future = executor.submit(() ->
    accountService.calculateBalance("ACC-001")
);

// Submit multiple tasks
List<Callable<PaymentResult>> tasks = requests.stream()
    .map(req -> (Callable<PaymentResult>) () -> processPayment(req))
    .toList();

List<Future<PaymentResult>> futures = executor.invokeAll(tasks);
// invokeAll blocks until ALL tasks complete

// Invoke any — returns the first completed result, cancels the rest
PaymentResult fastest = executor.invokeAny(tasks);
```

**Scheduled tasks:**

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

// Execute once after a delay
scheduler.schedule(() -> generateDailyReport(), 1, TimeUnit.HOURS);

// Execute periodically (fixed rate — regardless of task duration)
scheduler.scheduleAtFixedRate(
    () -> refreshExchangeRates(),
    0,          // Initial delay
    5,          // Period
    TimeUnit.MINUTES
);

// Execute periodically (fixed delay — wait between completions)
scheduler.scheduleWithFixedDelay(
    () -> reconcileTransactions(),
    0,
    30,
    TimeUnit.MINUTES
);
```

**Shutting down:**

```java
// Graceful shutdown — stops accepting new tasks, waits for running tasks to complete
executor.shutdown();
try {
    if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
        // Force shutdown if tasks did not complete in time
        executor.shutdownNow();
        if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
            log.error("Executor did not terminate");
        }
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}

// In Spring Boot, the application context handles executor shutdown automatically
// if the executor is a Spring bean.
```

**Thread pool sizing:**

| Task Type | Formula | Example |
|-----------|---------|---------|
| CPU-bound | Number of CPU cores + 1 | 8-core machine → pool size 9 |
| I/O-bound | CPU cores × (1 + wait time / compute time) | 8 cores, 90% I/O → pool size ~80 |
| Mixed | Benchmark and measure | Start with 2× cores, tune based on metrics |

For a typical Spring Boot web application handling HTTP requests (I/O-bound), a pool size of 50-200 is common. Tomcat's default is 200 threads.

### Future and CompletableFuture

**Future<V>:**

A `Future` represents the result of an asynchronous computation. It provides methods to check completion, wait for the result, and cancel the task.

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

Future<BigDecimal> future = executor.submit(() -> {
    Thread.sleep(2000);  // Simulate slow computation
    return new BigDecimal("1500.75");
});

// Check status
future.isDone();      // false (initially)
future.isCancelled(); // false

// Get the result (BLOCKS until complete)
try {
    BigDecimal result = future.get();           // Blocks indefinitely
    BigDecimal result2 = future.get(5, TimeUnit.SECONDS);  // Blocks with timeout
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
} catch (ExecutionException e) {
    log.error("Task failed", e.getCause());  // The actual exception thrown by the task
} catch (TimeoutException e) {
    log.warn("Task timed out");
    future.cancel(true);  // Attempt to interrupt the running task
}
```

**Problems with Future:**
- `get()` blocks the calling thread. In a web server, this ties up a request thread.
- No way to chain futures (compose asynchronous operations).
- No way to combine multiple futures without blocking.
- No built-in error handling beyond `ExecutionException`.

**CompletableFuture<V> (Java 8+):**

`CompletableFuture` solves all the problems of `Future`. It supports non-blocking chaining, combining, and error handling.

```java
// Creating CompletableFutures
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return fetchUserProfile(userId);  // Runs on ForkJoinPool.commonPool()
});

CompletableFuture<String> customPool = CompletableFuture.supplyAsync(
    () -> fetchUserProfile(userId),
    executorService  // Runs on a custom executor
);

CompletableFuture<Void> fireAndForget = CompletableFuture.runAsync(() -> {
    sendNotification(userId, "Welcome!");
});

// Chaining — transform the result
CompletableFuture<BigDecimal> balanceFuture = CompletableFuture
    .supplyAsync(() -> fetchAccount(accountId))
    .thenApply(account -> account.getBalance())
    .thenApply(balance -> balance.multiply(new BigDecimal("1.05")));  // Add 5% interest

// Chaining — async transformation (when the next step is also async)
CompletableFuture<PaymentResult> paymentFuture = CompletableFuture
    .supplyAsync(() -> validateRequest(request))
    .thenCompose(validRequest -> processPaymentAsync(validRequest))  // Returns CompletableFuture
    .thenCompose(result -> persistTransactionAsync(result));

// Combining — wait for two independent futures
CompletableFuture<BigDecimal> usdRate = fetchExchangeRateAsync("USD");
CompletableFuture<BigDecimal> eurRate = fetchExchangeRateAsync("EUR");

CompletableFuture<BigDecimal> combined = usdRate.thenCombine(eurRate, (usd, eur) -> {
    return usd.add(eur);  // Runs when BOTH futures complete
});

// Combining — wait for all futures
CompletableFuture<Void> allDone = CompletableFuture.allOf(
    processPaymentAsync(req1),
    processPaymentAsync(req2),
    processPaymentAsync(req3)
);

// Combining — first to complete
CompletableFuture<BigDecimal> fastest = CompletableFuture.anyOf(
    fetchPriceFromExchange1(symbol),
    fetchPriceFromExchange2(symbol),
    fetchPriceFromExchange3(symbol)
).thenApply(result -> (BigDecimal) result);

// Error handling
CompletableFuture<PaymentResult> resilient = CompletableFuture
    .supplyAsync(() -> processPayment(request))
    .exceptionally(ex -> {
        log.error("Payment failed", ex);
        return PaymentResult.failure(ex.getMessage());
    });

// Handle both success and failure
CompletableFuture<PaymentResult> handled = CompletableFuture
    .supplyAsync(() -> processPayment(request))
    .handle((result, ex) -> {
        if (ex != null) {
            log.error("Payment failed", ex);
            return PaymentResult.failure(ex.getMessage());
        }
        log.info("Payment succeeded: {}", result.getTransactionId());
        return result;
    });

// Getting the result (blocking — use sparingly)
PaymentResult result = paymentFuture.join();  // Unchecked exception on failure
PaymentResult result2 = paymentFuture.get();  // Checked exception

// Getting the result (non-blocking — preferred in reactive code)
paymentFuture.thenAccept(result -> {
    sendResponse(result);
});
```

**CompletableFuture in a real-world service:**

```java
@Service
public class PaymentOrchestrationService {

    private final ExecutorService executor = Executors.newFixedThreadPool(10);

    public CompletableFuture<PaymentResponse> processPaymentAsync(PaymentRequest request) {
        return CompletableFuture
            .supplyAsync(() -> validateRequest(request), executor)
            .thenCompose(validated -> CompletableFuture.allOf(
                checkFraudAsync(validated).thenAccept(fraudResult -> {
                    if (fraudResult.isSuspicious()) {
                        throw new FraudDetectedException(validated.getId());
                    }
                }),
                checkBalanceAsync(validated).thenAccept(balance -> {
                    if (balance.compareTo(validated.getAmount()) < 0) {
                        throw new InsufficientFundsException(validated.getAmount(), balance);
                    }
                })
            ).thenApply(v -> validated))
            .thenCompose(validated -> chargeGatewayAsync(validated))
            .thenCompose(gatewayResult -> persistTransactionAsync(gatewayResult))
            .thenApply(tx -> new PaymentResponse(tx.getId(), "SUCCESS"))
            .exceptionally(ex -> {
                log.error("Payment failed for request {}", request.getId(), ex);
                return new PaymentResponse(request.getId(), "FAILED: " + ex.getMessage());
            });
    }
}
```

### Concurrent Collections

The `java.util.concurrent` package provides thread-safe collection implementations that are far more efficient than wrapping a regular collection with `Collections.synchronizedMap()`.

**ConcurrentHashMap:**

```java
// Thread-safe map with fine-grained locking (lock per bucket, not per map)
ConcurrentHashMap<String, BigDecimal> balances = new ConcurrentHashMap<>();

// Basic operations (thread-safe)
balances.put("ACC-001", new BigDecimal("5000"));
BigDecimal balance = balances.get("ACC-001");
balances.remove("ACC-001");

// Atomic operations (critical for correctness)
// compute — atomically read, modify, and write
balances.compute("ACC-001", (key, currentBalance) -> {
    BigDecimal balance = currentBalance != null ? currentBalance : BigDecimal.ZERO;
    return balance.add(new BigDecimal("500"));
});

// computeIfAbsent — atomically check and insert
balances.computeIfAbsent("ACC-002", key -> BigDecimal.ZERO);

// computeIfPresent — atomically check and modify
balances.computeIfPresent("ACC-001", (key, balance) -> balance.subtract(new BigDecimal("100")));

// merge — atomically merge a value
balances.merge("ACC-001", new BigDecimal("200"), BigDecimal::add);

// putIfAbsent — atomically insert only if absent
balances.putIfAbsent("ACC-003", new BigDecimal("1000"));

// replace — atomically replace only if current value matches
balances.replace("ACC-001", new BigDecimal("5500"), new BigDecimal("5000"));
// CAS (Compare-And-Swap): replaces only if current value is 5500
```

**Why ConcurrentHashMap over synchronized HashMap:**

- `Collections.synchronizedMap(new HashMap<>())` acquires a single lock for every operation. Under high concurrency, all threads serialize on this lock.
- `ConcurrentHashMap` uses fine-grained locking (CAS operations and synchronized blocks on individual nodes). Multiple threads can read and write different keys simultaneously without contention.
- `ConcurrentHashMap` never locks the entire map for reads. Reads are lock-free.
- `ConcurrentHashMap` does not allow null keys or null values (unlike `HashMap`). This is intentional — null is ambiguous in a concurrent context (does `get(key)` returning null mean the key is absent or the value is null?).

**CopyOnWriteArrayList:**

```java
// Thread-safe list optimized for read-heavy workloads
// Every write creates a new copy of the underlying array
CopyOnWriteArrayList<String> listeners = new CopyOnWriteArrayList<>();

listeners.add("listener1");
listeners.add("listener2");

// Iteration is safe even while other threads modify the list
// The iterator sees a snapshot of the array at the time of creation
for (String listener : listeners) {
    notifyListener(listener);  // No ConcurrentModificationException
}
```

**When to use CopyOnWriteArrayList:**
- Read operations vastly outnumber writes (e.g., event listener lists, configuration caches).
- The list is small (copying a large array on every write is expensive).

**When NOT to use CopyOnWriteArrayList:**
- Write-heavy workloads. Every `add()`, `remove()`, or `set()` copies the entire array.
- Large lists. A list with 100,000 elements copies 100,000 references on every write.

**BlockingQueue:**

A thread-safe queue that blocks on `put()` when full and `take()` when empty. Essential for producer-consumer patterns.

```java
// Bounded blocking queue
BlockingQueue<PaymentRequest> queue = new ArrayBlockingQueue<>(1000);

// Producer
executor.submit(() -> {
    while (running) {
        PaymentRequest request = receiveFromKafka();
        queue.put(request);  // Blocks if queue is full
    }
});

// Consumer
executor.submit(() -> {
    while (running) {
        PaymentRequest request = queue.take();  // Blocks if queue is empty
        processPayment(request);
    }
});

// Other implementations
BlockingQueue<Task> linkedQueue = new LinkedBlockingQueue<>(10000);  // Linked nodes
BlockingQueue<Task> priorityQueue = new PriorityBlockingQueue<>();   // Priority ordering
BlockingQueue<Task> delayQueue = new DelayQueue<>();                 // Elements available after delay
```

### Atomic Classes

Atomic classes provide lock-free, thread-safe operations on single variables using hardware-level Compare-And-Swap (CAS) instructions.

```java
// Atomic integer
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();   // Atomically increment and return new value
counter.decrementAndGet();   // Atomically decrement
counter.addAndGet(5);        // Atomically add 5
counter.compareAndSet(5, 10); // Atomically set to 10 only if current value is 5
int current = counter.get();
counter.getAndIncrement();   // Return old value, then increment

// Atomic long (for high-volume counters)
AtomicLong transactionCount = new AtomicLong(0);
transactionCount.incrementAndGet();

// Atomic boolean
AtomicBoolean isProcessing = new AtomicBoolean(false);
if (isProcessing.compareAndSet(false, true)) {
    try {
        processBatch();
    } finally {
        isProcessing.set(false);
    }
}

// Atomic reference (for thread-safe object references)
AtomicReference<BigDecimal> exchangeRate = new AtomicReference<>(new BigDecimal("0.92"));
exchangeRate.updateAndGet(current -> current.multiply(new BigDecimal("1.01")));
exchangeRate.compareAndSet(
    new BigDecimal("0.92"),  // Expected current value
    new BigDecimal("0.93")   // New value
);

// Atomic reference with stamped lock (for optimistic reads)
AtomicStampedReference<Config> config = new AtomicStampedReference<>(initialConfig, 1);
int[] stampHolder = new int[1];
Config current = config.get(stampHolder);
int stamp = stampHolder[0];
config.compareAndSet(current, newConfig, stamp, stamp + 1);
```

**When to use atomics vs locks:**
- Atomics are faster for simple operations on single variables (counters, flags, references).
- Locks are necessary when multiple variables must be updated together atomically (e.g., debiting one account and crediting another).
- Atomics use CAS, which can suffer from the ABA problem (a value changes from A to B and back to A, making a CAS succeed when it should not). `AtomicStampedReference` solves this.

### Synchronization Utilities

**CountDownLatch:**

A one-shot barrier that allows one or more threads to wait until a set of operations completes.

```java
// Wait for 3 services to initialize before accepting requests
CountDownLatch latch = new CountDownLatch(3);

executor.submit(() -> {
    initializeDatabase();
    latch.countDown();  // Decrement count
});

executor.submit(() -> {
    initializeCache();
    latch.countDown();
});

executor.submit(() -> {
    initializeMessageQueue();
    latch.countDown();
});

latch.await();  // Blocks until count reaches 0
log.info("All services initialized. Ready to accept requests.");
```

**CyclicBarrier:**

A reusable barrier that allows a set of threads to wait for each other at a common point.

```java
// 4 threads must all reach the barrier before any can proceed
CyclicBarrier barrier = new CyclicBarrier(4, () -> {
    log.info("All threads reached the barrier. Proceeding to next phase.");
});

for (int i = 0; i < 4; i++) {
    executor.submit(() -> {
        processPartition(partitionId);
        barrier.await();  // Wait for all 4 threads
        aggregateResults();
    });
}
```

**Semaphore:**

A counting semaphore that controls access to a limited number of resources.

```java
// Limit concurrent database connections to 10
Semaphore dbSemaphore = new Semaphore(10);

public void queryDatabase(String sql) throws InterruptedException {
    dbSemaphore.acquire();  // Blocks if 10 threads already hold permits
    try {
        executeQuery(sql);
    } finally {
        dbSemaphore.release();  // Return the permit
    }
}

// Try to acquire with timeout
if (dbSemaphore.tryAcquire(5, TimeUnit.SECONDS)) {
    try {
        executeQuery(sql);
    } finally {
        dbSemaphore.release();
    }
} else {
    throw new TimeoutException("Could not acquire database connection");
}
```

---

### Virtual Threads (Java 21)

Virtual threads (Project Loom) are the most significant change to Java concurrency in two decades. They are lightweight threads managed by the JVM, not the operating system. A single JVM can run millions of virtual threads simultaneously.

**The problem virtual threads solve:**

Traditional platform threads (OS threads) are expensive. Each thread consumes ~1 MB of stack memory and requires a kernel-level context switch. A server with 200 platform threads can handle ~200 concurrent requests. If each request takes 100 ms (mostly waiting for database or network I/O), the CPU is idle 95% of the time while threads block on I/O.

**Creating virtual threads:**

```java
// Method 1: Direct creation
Thread.startVirtualThread(() -> {
    processPayment(request);
});

// Method 2: Builder
Thread vt = Thread.ofVirtual()
    .name("payment-vt-", 0)  // Auto-numbered: payment-vt-0, payment-vt-1, ...
    .start(() -> processPayment(request));

// Method 3: Executor (recommended for production)
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    // Each submitted task gets its own virtual thread
    Future<PaymentResult> f1 = executor.submit(() -> processPayment(req1));
    Future<PaymentResult> f2 = executor.submit(() -> processPayment(req2));
    Future<PaymentResult> f3 = executor.submit(() -> processPayment(req3));

    PaymentResult r1 = f1.get();
    PaymentResult r2 = f2.get();
    PaymentResult r3 = f3.get();
}
```

**Virtual threads vs platform threads:**

| Feature | Platform Thread | Virtual Thread |
|---------|----------------|---------------|
| Managed by | Operating system | JVM |
| Memory per thread | ~1 MB stack | ~few KB (grows on demand) |
| Max threads | ~1,000-10,000 | Millions |
| Creation cost | Expensive (OS syscall) | Cheap (JVM object) |
| Context switch | Kernel-level (slow) | JVM-level (fast) |
| Thread pool | Required | Not needed (one per task) |
| Blocking I/O | Blocks the OS thread | Unmounts from OS thread, frees it |
| `synchronized` | Works | Works but may pin to carrier thread |
| `ThreadLocal` | Works | Works but use sparingly (millions of threads) |
| Best for | CPU-bound work | I/O-bound work |

**Impact on backend applications:**

```java
// Before virtual threads: 200 platform threads, each blocking on DB I/O
// Throughput: ~200 concurrent requests
ExecutorService executor = Executors.newFixedThreadPool(200);

// After virtual threads: millions of virtual threads, each blocking on DB I/O
// Throughput: ~10,000+ concurrent requests (limited by DB, not threads)
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
```

**Spring Boot 3.2+ integration:**

```yaml
# application.yml — enable virtual threads with one line
spring:
  threads:
    virtual:
      enabled: true
```

This single configuration enables virtual threads for all HTTP request handling, `@Async` methods, and scheduled tasks. No code changes required.

**Caveats:**

- Virtual threads are beneficial for I/O-bound workloads. For CPU-bound workloads (number crunching, encryption), platform threads with a bounded pool are still appropriate.
- `synchronized` blocks can cause "pinning," where a virtual thread holds onto its carrier (OS) thread during the synchronized block. Use `ReentrantLock` instead of `synchronized` to avoid pinning.
- `ThreadLocal` variables are allocated per virtual thread. With millions of virtual threads, excessive `ThreadLocal` usage can consume significant memory. Use `ScopedValue` (Java 21 preview) instead.
- Virtual threads should not be pooled. The executor `Executors.newVirtualThreadPerTaskExecutor()` creates a new virtual thread for each task and destroys it when the task completes. This is the intended usage pattern.

---

## Code Examples

**A complete concurrent payment processing system:**

```java
package com.example.fintech.concurrent;

import java.math.BigDecimal;
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicLong;

public class ConcurrentPaymentProcessor {

    private final ExecutorService executor;
    private final ConcurrentHashMap<String, BigDecimal> accountBalances;
    private final AtomicLong processedCount;
    private final BlockingQueue<PaymentRequest> requestQueue;

    public ConcurrentPaymentProcessor(int threadPoolSize) {
        this.executor = new ThreadPoolExecutor(
            threadPoolSize,
            threadPoolSize * 2,
            60L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(10_000),
            new ThreadFactory() {
                private final AtomicLong counter = new AtomicLong(1);
                @Override
                public Thread newThread(Runnable r) {
                    Thread t = new Thread(r, "payment-worker-" + counter.getAndIncrement());
                    t.setDaemon(false);
                    return t;
                }
            },
            new ThreadPoolExecutor.CallerRunsPolicy()
        );
        this.accountBalances = new ConcurrentHashMap<>();
        this.processedCount = new AtomicLong(0);
        this.requestQueue = new LinkedBlockingQueue<>(5_000);
    }

    // Thread-safe balance update using ConcurrentHashMap.compute
    public void creditAccount(String accountId, BigDecimal amount) {
        accountBalances.compute(accountId, (key, currentBalance) -> {
            BigDecimal balance = currentBalance != null ? currentBalance : BigDecimal.ZERO;
            return balance.add(amount);
        });
    }

    // Thread-safe balance check and debit using synchronized
    public boolean debitAccount(String accountId, BigDecimal amount) {
        // Use compute for atomicity — check and debit in one operation
        boolean[] success = {false};
        accountBalances.compute(accountId, (key, currentBalance) -> {
            BigDecimal balance = currentBalance != null ? currentBalance : BigDecimal.ZERO;
            if (balance.compareTo(amount) >= 0) {
                success[0] = true;
                return balance.subtract(amount);
            }
            return balance;  // Insufficient funds — do not modify
        });
        return success[0];
    }

    // Process a batch of payments concurrently
    public List<PaymentResult> processBatch(List<PaymentRequest> requests)
            throws InterruptedException {
        List<Callable<PaymentResult>> tasks = requests.stream()
            .map(request -> (Callable<PaymentResult>) () -> processSinglePayment(request))
            .toList();

        List<Future<PaymentResult>> futures = executor.invokeAll(tasks);

        List<PaymentResult> results = new ArrayList<>();
        for (Future<PaymentResult> future : futures) {
            try {
                results.add(future.get(30, TimeUnit.SECONDS));
            } catch (ExecutionException e) {
                results.add(PaymentResult.failure("Processing error: " + e.getCause().getMessage()));
            } catch (TimeoutException e) {
                results.add(PaymentResult.failure("Processing timeout"));
            }
        }
        return results;
    }

    // Process payments asynchronously with CompletableFuture
    public CompletableFuture<List<PaymentResult>> processBatchAsync(List<PaymentRequest> requests) {
        List<CompletableFuture<PaymentResult>> futures = requests.stream()
            .map(request -> CompletableFuture.supplyAsync(
                () -> processSinglePayment(request), executor
            ))
            .toList();

        return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                .map(CompletableFuture::join)
                .toList()
            );
    }

    private PaymentResult processSinglePayment(PaymentRequest request) {
        try {
            if (!debitAccount(request.sourceAccountId(), request.amount())) {
                return PaymentResult.failure("Insufficient funds");
            }
            creditAccount(request.targetAccountId(), request.amount());
            processedCount.incrementAndGet();
            return PaymentResult.success(request.id());
        } catch (Exception e) {
            return PaymentResult.failure(e.getMessage());
        }
    }

    public long getProcessedCount() {
        return processedCount.get();
    }

    public void shutdown() {
        executor.shutdown();
        try {
            if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }

    public record PaymentRequest(
        String id, String sourceAccountId, String targetAccountId, BigDecimal amount
    ) {}

    public record PaymentResult(String transactionId, boolean success, String message) {
        public static PaymentResult success(String txId) {
            return new PaymentResult(txId, true, "OK");
        }
        public static PaymentResult failure(String reason) {
            return new PaymentResult(null, false, reason);
        }
    }
}
```

**Virtual threads example (Java 21):**

```java
public class VirtualThreadDemo {

    public static void main(String[] args) throws Exception {
        // Process 10,000 concurrent I/O-bound tasks
        int taskCount = 10_000;
        long start = System.currentTimeMillis();

        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            List<Future<String>> futures = new ArrayList<>();

            for (int i = 0; i < taskCount; i++) {
                final int taskId = i;
                futures.add(executor.submit(() -> {
                    // Simulate I/O-bound work (database query, API call)
                    Thread.sleep(100);
                    return "Task " + taskId + " completed on " + Thread.currentThread();
                }));
            }

            int completed = 0;
            for (Future<String> future : futures) {
                future.get();
                completed++;
            }

            long elapsed = System.currentTimeMillis() - start;
            System.out.printf("Completed %d tasks in %d ms%n", completed, elapsed);
            // With virtual threads: ~200-500 ms
            // With 200 platform threads: ~5000 ms
        }
    }
}
```

---

## Important Notes

- Never share mutable state between threads without synchronization. This is the single most important rule of concurrent programming. If two threads can read and write the same variable, you must use `synchronized`, `volatile`, atomic classes, or concurrent collections to ensure correctness.
- `synchronized` is the simplest synchronization mechanism but also the coarsest. It locks an entire method or block, preventing all other threads from entering. For high-contention scenarios, prefer `ReentrantLock`, `ConcurrentHashMap`, or atomic classes, which offer finer-grained locking.
- The `volatile` keyword guarantees visibility but not atomicity. `volatile int count; count++;` is a race condition because the increment involves three operations (read, add, write) that can be interleaved. Use `AtomicInteger` for atomic increments.
- `ConcurrentHashMap` is the correct choice for thread-safe maps in production. Do not use `Collections.synchronizedMap(new HashMap<>())` — it serializes all operations on a single lock and is significantly slower under contention.
- `CompletableFuture` is the modern replacement for raw `Future`. It supports non-blocking chaining, combining, and error handling. Use it for orchestrating multiple asynchronous operations (e.g., calling three external APIs in parallel and combining the results).
- Virtual threads (Java 21) are a paradigm shift for I/O-bound applications. They allow you to write blocking, synchronous code that scales to millions of concurrent operations. Enable them in Spring Boot 3.2+ with a single configuration property. Do not pool virtual threads — create one per task.
- Thread pools must be sized appropriately for the workload. CPU-bound tasks should use a pool size close to the number of CPU cores. I/O-bound tasks can use much larger pools (50-200+). An incorrectly sized pool either wastes resources (too large) or creates bottlenecks (too small).
- Always shut down executors gracefully. Call `shutdown()` first, then `awaitTermination()`, then `shutdownNow()` if necessary. Failing to shut down executors prevents the JVM from exiting and leaks threads.
- `ThreadLocal` provides per-thread storage. It is commonly used for request-scoped data (user context, transaction context, correlation IDs). In Spring, `RequestContextHolder` uses `ThreadLocal` internally. With virtual threads, `ThreadLocal` usage should be minimized because millions of virtual threads can consume significant memory.
- Deadlocks are caused by inconsistent lock ordering. The most reliable prevention strategy is to always acquire locks in a globally consistent order. If thread A acquires lock X then lock Y, thread B must also acquire lock X before lock Y.
- The `java.util.concurrent` package is one of the most well-designed APIs in the Java standard library. Its classes (`ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue`, `CountDownLatch`, `Semaphore`) have been battle-tested for two decades. Prefer these over manual synchronization whenever possible.
- In a Spring Boot application, most concurrency is managed by the framework. Tomcat handles HTTP request threading, `@Async` handles background tasks, and `@Scheduled` handles periodic jobs. You rarely need to create threads manually. Focus on writing thread-safe service methods and configuring thread pools correctly.
- The Fork/Join framework (`ForkJoinPool`) is the underlying engine for parallel streams and `CompletableFuture.supplyAsync()`. It uses work-stealing to balance load across threads. You rarely interact with it directly, but understanding that it exists helps when tuning parallel stream performance.

---

## Practice

1. Create a `BankAccount` class with a `deposit()` and `withdraw()` method. Spawn 100 threads, each depositing $1.00 into the same account. Without synchronization, the final balance will be less than $100.00. Fix the race condition using `synchronized`, then using `ReentrantLock`, then using `AtomicReference<BigDecimal>`. Benchmark all three approaches.

2. Implement a producer-consumer system using `BlockingQueue`. The producer generates `PaymentRequest` objects and puts them on the queue. Three consumer threads take requests from the queue and process them. Add a poison pill to signal consumers to stop. Verify that all requests are processed exactly once.

3. Write a method that fetches data from three external APIs in parallel using `CompletableFuture`. Combine the results into a single response object. Add error handling so that if one API fails, the others still succeed and the failed result is replaced with a default value.

4. Implement a rate limiter using `Semaphore`. The rate limiter should allow at most 10 concurrent requests. Requests that cannot acquire a permit within 5 seconds should be rejected with a `RateLimitExceededException`.

5. Create a `CountDownLatch` example that simulates a microservice startup sequence. The API gateway must wait for the user service, payment service, and notification service to initialize before accepting traffic. Each service initializes in a separate thread with a random delay.

6. Demonstrate the difference between `synchronized` and `ReentrantLock`. Implement a shared counter with both approaches. Show that `ReentrantLock` supports `tryLock(timeout)`, which `synchronized` does not. Demonstrate how `tryLock` prevents deadlock.

7. Write a program using virtual threads (Java 21) that simulates 10,000 concurrent database queries (use `Thread.sleep()` to simulate I/O). Compare the execution time with a fixed thread pool of 200 platform threads. Document the results.

8. In your Obsidian vault, create a decision table: "Which concurrency mechanism should I use?" with rows for scenarios (shared counter, shared map, producer-consumer, parallel API calls, scheduled tasks, rate limiting, startup coordination) and columns recommending the specific class or pattern.

---

## References

- Java Concurrency Tutorial: https://docs.oracle.com/javase/tutorial/essential/concurrency/
- java.util.concurrent Package: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/package-summary.html
- CompletableFuture API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html
- JEP 444 (Virtual Threads, Java 21): https://openjdk.org/jeps/444
- JEP 425 (Virtual Threads, Preview, Java 19): https://openjdk.org/jeps/425
- Spring Boot Virtual Threads: https://docs.spring.io/spring-boot/reference/features/spring-application.html#features.spring-application.virtual-threads
- "Java Concurrency in Practice" by Brian Goetz, Tim Peierls, Joshua Bloch, Joseph Bowbeer, David Holmes, and Doug Lea
- "Effective Java" by Joshua Bloch — Items 78-84 (Concurrency)
