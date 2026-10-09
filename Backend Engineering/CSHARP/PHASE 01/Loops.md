## What Are Loops?
Loops repeat a block of code. Backend applications use them to process records, validate collections, build reports, handle messages, and work through data in batches.

C# has four main loop statements:

- **`for`** — useful when you need an index or explicit initialization, condition, and update steps.
- **`foreach`** — useful when you want to process each element without managing an index.
- **`while`** — checks its condition before each iteration, so its body can run zero or more times.
- **`do`–`while`** — checks its condition after the body, so its body runs at least once.

## The `for` Loop

A `for` loop is useful when you need an index or when the loop's initialization, continuation condition, and update are naturally expressed together.

### Syntax

```csharp
for (initialization; condition; iteration)
{
    // Loop body
}
```

- **Initialization** runs once, before the first condition check. It often declares and initializes a counter.
- **Condition** is checked before each iteration. If it is false, the loop ends.
- **Iteration** runs after the body completes normally or a `continue` transfers control to the next iteration. It commonly increments or decrements a counter.

### Basic Example

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Iteration {i}");
}
```

Output:

```text
Iteration 0
Iteration 1
Iteration 2
Iteration 3
Iteration 4
```

### Counting Backward

```csharp
for (int i = 10; i >= 0; i--)
{
    Console.Write($"{i} ");
}
Console.WriteLine("Liftoff!");
```

Output:

```text
10 9 8 7 6 5 4 3 2 1 0 Liftoff!
```

### Custom Step Size

```csharp
// Count by 5.
for (int i = 0; i <= 50; i += 5)
{
    Console.Write($"{i} ");
}
// Output: 0 5 10 15 20 25 30 35 40 45 50
```

### Iterating Over an Array by Index

Use `Length` for arrays. For a `List<T>`, use `Count`.

```csharp
string[] products = { "Laptop", "Mouse", "Monitor", "Keyboard" };

for (int i = 0; i < products.Length; i++)
{
    Console.WriteLine($"[{i}] {products[i]}");
}
```

Output:

```text
[0] Laptop
[1] Mouse
[2] Monitor
[3] Keyboard
```

### Multiple Variables in a `for` Loop

The initializer and iterator sections can contain multiple expressions separated by commas. Variables declared together in one declaration share a type.

```csharp
for (int left = 0, right = 9; left < right; left++, right--)
{
    Console.WriteLine($"Left: {left}, Right: {right}");
}
```

Output:

```text
Left: 0, Right: 9
Left: 1, Right: 8
Left: 2, Right: 7
Left: 3, Right: 6
Left: 4, Right: 5
```

This pattern can be useful when comparing elements from opposite ends of an array, for example while checking whether a sequence is a palindrome.

### Nested `for` Loops

```csharp
for (int row = 1; row <= 3; row++)
{
    for (int col = 1; col <= 3; col++)
    {
        Console.Write($"{row * col,4}");
    }

    Console.WriteLine();
}
```

Output:

```text
   1   2   3
   2   4   6
   3   6   9
```

If one loop runs `N` times and another runs `M` times for each outer iteration, the inner body runs `N × M` times. For example, 1,000 iterations in each loop produce 1,000,000 inner-body executions. Nested loops are not automatically a problem, but large cross-comparisons can be expensive; use an appropriate index or lookup structure when it fits the data and measure performance on important paths.

## The `foreach` Loop

`foreach` processes each element yielded by a collection or sequence without requiring you to manage an index. Arrays, lists, dictionaries, sets, and strings are common examples. A type can support `foreach` by implementing `IEnumerable` or `IEnumerable<T>`, or by providing the C# enumerator pattern (`GetEnumerator`, `Current`, and `MoveNext`). [3](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements)

### Syntax

```csharp
foreach (Type variable in collection)
{
    // Loop body
}
```

### Basic Example

```csharp
string[] cities = { "New York", "London", "Tokyo", "Paris" };

foreach (string city in cities)
{
    Console.WriteLine($"City: {city}");
}
```

Output:

```text
City: New York
City: London
City: Tokyo
City: Paris
```

### Type Inference with `var`

Use `var` when the element type is clear from the collection or when writing the type explicitly would add clutter.

```csharp
var numbers = new List<int> { 10, 20, 30, 40, 50 };

foreach (var number in numbers)
{
    Console.WriteLine(number * 2);
}
```

### Iterating Over a Dictionary

A dictionary's elements are `KeyValuePair<TKey, TValue>` values. Modern C# can deconstruct each pair into its key and value.

```csharp
using System;
using System.Collections.Generic;

var config = new Dictionary<string, string>
{
    ["Database"] = "Server=localhost;Database=App;",
    ["Cache"] = "localhost:6379",
    ["LogLevel"] = "Information"
};

foreach (KeyValuePair<string, string> entry in config)
{
    Console.WriteLine($"{entry.Key}: {entry.Value}");
}
```

One possible output is:

```text
Database: Server=localhost;Database=App;
Cache: localhost:6379
LogLevel: Information
```

`Dictionary<TKey, TValue>` does not guarantee an enumeration order. Use an ordered collection or sort the entries if output order matters. [2](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2?view=net-10.0)

You can also deconstruct each `KeyValuePair<TKey, TValue>` into its key and value (C# 7 tuple-deconstruction syntax, with a .NET collection that exposes `Deconstruct`):

```csharp
foreach (var (key, value) in config)
{
    Console.WriteLine($"{key}: {value}");
}
```

### Iterating Over a String

A string can be enumerated with `foreach`. Iterating as `char` visits UTF-16 code units; a supplementary Unicode character, such as many emoji, uses a surrogate pair and therefore occupies two `char` values. Use `Rune` when you need to process Unicode scalar values, and a text-element API when you need user-perceived grapheme clusters. [1](https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-encoding-introduction).

```csharp
string email = "alice@example.com";
int atCount = 0;

foreach (char c in email)
{
    if (c == '@')
    {
        atCount++;
    }
}

Console.WriteLine($"@ symbols found: {atCount}"); // Output: 1
```

### Read-Only Iteration Variable and Collection Changes

In an ordinary by-value `foreach`, you can't assign a new value to the iteration variable. If it refers to a mutable object, you can still change that object's properties. Also, whether a collection can change during enumeration depends on that collection and enumerator; many standard collections, including `List<T>`, invalidate an active enumerator when structurally modified.

```csharp
var names = new List<string> { "Alice", "Bob", "Charlie" };

foreach (string name in names)
{
    // name = "Dave"; // Compile error: the iteration variable can't be reassigned.
    Console.WriteLine(name);

    // Do not structurally modify this List<T> during its active enumeration:
    // names.Add("Dave");
}

names.Add("Dave"); // Safe after the foreach has completed.
```

If you need to remove matching values from a `List<T>`, `RemoveAll` is often the simplest option:

```csharp
var items = new List<int> { 1, 2, 3, 4, 5 };
items.RemoveAll(item => item % 2 == 0);

// items now contains 1, 3, 5.
```

You can also enumerate a snapshot and modify the original list, when that behavior is appropriate:

```csharp
var items = new List<int> { 1, 2, 3, 4, 5 };

foreach (int item in items.ToArray())
{
    if (item % 2 == 0)
    {
        items.Remove(item);
    }
}
```

Some enumerators support advanced `ref foreach` forms, but those are different from the ordinary read-only iteration variable shown above.

### `foreach` with an Index

`foreach` doesn't provide an index directly. Use a `for` loop when an index is central to the logic, or use LINQ's indexed `Select` overload and tuple deconstruction.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

var products = new List<string> { "Laptop", "Mouse", "Monitor" };

// Option 1: use a for loop.
for (int i = 0; i < products.Count; i++)
{
    Console.WriteLine($"[{i}] {products[i]}");
}

// Option 2: LINQ Select supplies both the element and its zero-based index.
foreach (var (product, index) in products.Select((product, index) => (product, index)))
{
    Console.WriteLine($"[{index}] {product}");
}
```

The second option uses LINQ's indexed overload; it isn't a special C# 8 loop feature. As with other LINQ queries, consider readability and allocations when using it in a hot path.

For asynchronous sequences, C# also provides `await foreach` for `IAsyncEnumerable<T>`. It can asynchronously wait for each next element; it is useful for streams where fetching an item may involve I/O.

## The `while` Loop

A `while` loop checks its condition before each iteration. If the condition is false initially, the body doesn't run.

Use `while` when the number of iterations isn't known in advance and depends on a condition that can change during execution.

### Syntax

```csharp
while (condition)
{
    // Loop body
}
```

### Basic Example

```csharp
int count = 0;

while (count < 5)
{
    Console.WriteLine($"Count: {count}");
    count++;
}
```

Output:

```text
Count: 0
Count: 1
Count: 2
Count: 3
Count: 4
```

### Reading Until a Condition Is Met

This example also handles end-of-stream, when `Console.ReadLine()` returns `null`.

```csharp
using System;

int attempts = 0;
const int maxAttempts = 3;

Console.WriteLine("Enter 'quit' to exit.");

while (attempts < maxAttempts)
{
    Console.Write("> ");
    string? input = Console.ReadLine();

    if (input is null)
    {
        Console.WriteLine("Input ended.");
        break;
    }

    if (string.Equals(input, "quit", StringComparison.OrdinalIgnoreCase))
    {
        Console.WriteLine("Goodbye!");
        break;
    }

    attempts++;
    Console.WriteLine($"Attempt {attempts} of {maxAttempts}.");
}
```

### Processing a Queue

A `while` loop is useful for processing items until a queue or stack is empty.

```csharp
using System;
using System.Collections.Generic;

var taskQueue = new Queue<string>();
taskQueue.Enqueue("Send email to Alice");
taskQueue.Enqueue("Generate invoice #1042");
taskQueue.Enqueue("Update inventory");

int processedCount = 0;

while (taskQueue.Count > 0)
{
    string task = taskQueue.Dequeue();
    Console.WriteLine($"Processing: {task}");
    processedCount++;
}

Console.WriteLine($"Total tasks processed: {processedCount}");
```

Output:

```text
Processing: Send email to Alice
Processing: Generate invoice #1042
Processing: Update inventory
Total tasks processed: 3
```

### Long-Running Loop with an Exit Condition

Long-running workers and message consumers often repeat work until a queue is complete or shutdown is requested. Pass the cancellation token to asynchronous operations and check it between iterations.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

static async Task ConsumeMessagesAsync(CancellationToken stoppingToken)
{
    while (true)
    {
        stoppingToken.ThrowIfCancellationRequested();

        // Placeholder methods: the real implementations should observe the token.
        string? message = await ReceiveMessageAsync(stoppingToken);

        if (message is null)
        {
            Console.WriteLine("No more messages. Shutting down.");
            break;
        }

        await ProcessMessageAsync(message, stoppingToken);
    }
}
```

A long-running loop needs a real exit or cancellation path. A tight synchronous loop can consume CPU and a thread; an asynchronous operation that is awaiting I/O generally doesn't occupy a thread for the entire wait. In hosted services, propagate or handle cancellation consistently rather than relying on an unreachable `break`.

## The `do`–`while` Loop

A `do`–`while` loop evaluates its condition after executing the body, so the body runs at least once.

### Syntax

```csharp
do
{
    // Loop body
} while (condition);
```

The semicolon after the condition is required. A `do`–`while` loop is the loop form whose closing syntax ends with a semicolon after its condition.

### Basic Example: Read a Positive Number

This example prompts at least once and also stops cleanly if input reaches end-of-stream.

```csharp
using System;

int number = 0;
bool isValid = false;

do
{
    Console.Write("Enter a positive number: ");
    string? input = Console.ReadLine();

    if (input is null)
    {
        Console.WriteLine("No more input is available.");
        break;
    }

    isValid = int.TryParse(input, out number) && number > 0;

    if (!isValid)
    {
        Console.WriteLine("Please enter a valid positive number.");
    }
} while (!isValid);

if (isValid)
{
    Console.WriteLine($"You entered: {number}");
}
```

### Retry Logic

A `do`–`while` loop can express retry logic when an operation must be attempted at least once. In asynchronous backend code, use `Task.Delay` with a cancellation token rather than blocking a thread with `Thread.Sleep`.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

static async Task<bool> ConnectWithRetryAsync(CancellationToken cancellationToken)
{
    const int maxAttempts = 3;
    int attempt = 0;
    bool success = false;

    do
    {
        cancellationToken.ThrowIfCancellationRequested();
        attempt++;
        Console.WriteLine($"Attempt {attempt} of {maxAttempts}...");

        // Placeholder: false represents a retryable connection failure.
        // Permanent failures should be reported or thrown rather than retried.
        success = await TryConnectToDatabaseAsync(cancellationToken);

        if (!success && attempt < maxAttempts)
        {
            // 1, 2, ... seconds: exponential backoff for this small example.
            TimeSpan delay = TimeSpan.FromSeconds(Math.Pow(2, attempt - 1));
            Console.WriteLine($"Waiting {delay.TotalSeconds:0} second(s) before retry...");
            await Task.Delay(delay, cancellationToken);
        }
    } while (!success && attempt < maxAttempts);

    return success;
}
```

Real retry policies should retry only transient failures, cap the delay, and usually add jitter to avoid synchronized retry spikes. Operations with side effects, such as charging a payment, also need idempotency protection.

## Loop Control Statements: `break` and `continue`

### `break`

In a loop, `break` exits the innermost enclosing loop and continues after it. A `break` inside a `switch` exits the `switch` instead.

```csharp
int[] numbers = { 1, 3, 5, 8, 11, 13, 16 };

foreach (int number in numbers)
{
    if (number % 2 == 0)
    {
        Console.WriteLine($"First even number found: {number}");
        break;
    }

    Console.WriteLine($"Checked: {number} (odd)");
}
```

Output:

```text
Checked: 1 (odd)
Checked: 3 (odd)
Checked: 5 (odd)
First even number found: 8
```

### `continue`

`continue` skips the rest of the current loop body and proceeds to the next iteration. In a `for` loop, the iteration expression runs next; in `while` and `do`–`while`, the condition is checked; in `foreach`, enumeration advances to the next element.

```csharp
for (int i = 1; i <= 10; i++)
{
    if (i % 3 == 0)
    {
        continue; // Skip multiples of 3.
    }

    Console.Write($"{i} ");
}
// Output: 1 2 4 5 7 8 10
```

### `break` and `continue` in Nested Loops

Within nested loops, `break` and `continue` affect the innermost enclosing loop. They don't directly exit or skip an iteration of an outer loop.

```csharp
for (int row = 1; row <= 3; row++)
{
    for (int col = 1; col <= 5; col++)
    {
        if (col == 3)
        {
            break; // Exits the inner loop only.
        }

        Console.Write($"({row},{col}) ");
    }

    Console.WriteLine();
}
```

Output:

```text
(1,1) (1,2)
(2,1) (2,2)
(3,1) (3,2)
```

## Performance Considerations for Backend Development

### Avoid Structural Collection Changes During Enumeration

Many standard mutable collections invalidate their enumerator when their structure changes during an active `foreach`. For example, modifying a `List<T>` while its enumerator is active usually causes an `InvalidOperationException` when the enumerator advances. The exact behavior depends on the collection; don't assume every collection behaves identically.

```csharp
using System.Collections.Generic;
using System.Linq;

var orders = new List<(int Id, bool IsExpired)>
{
    (1, false),
    (2, true),
    (3, false)
};

// Avoid structural changes to this List<T> inside the active foreach:
// foreach (var order in orders)
// {
//     if (order.IsExpired) orders.Remove(order);
// }

// Option 1: remove matching elements directly.
orders.RemoveAll(order => order.IsExpired);
```

Alternatively, keep the original list and materialize a separate filtered list:

```csharp
using System.Collections.Generic;
using System.Linq;

var orders = new List<(int Id, bool IsExpired)>
{
    (1, false),
    (2, true),
    (3, false)
};

var activeOrders = orders.Where(order => !order.IsExpired).ToList();
```

### Pre-allocate Collection Capacity When Useful

If you have a reasonable estimate of a result's size, providing a `List<T>` capacity can reduce internal array growth. Avoid reserving far more memory than needed.

```csharp
using System.Collections.Generic;

int expectedCount = 10_000;
var results = new List<string>(expectedCount);

for (int i = 0; i < expectedCount; i++)
{
    results.Add($"Item {i}");
}
```

### Choose LINQ or a Manual Loop for Clarity, Then Measure

LINQ can make straightforward filtering, mapping, and aggregation easier to read. It isn't automatically faster than a manual loop: some queries are deferred, materializing results allocates, and performance depends on the source and workload. Prefer clear code first, then benchmark hot paths.

```csharp
using System.Collections.Generic;
using System.Linq;

var orders = new List<(string Status, decimal Amount)>
{
    ("Completed", 125m),
    ("Pending", 50m),
    ("Completed", 75m)
};

// Manual one-pass aggregation.
decimal total = 0;
foreach (var order in orders)
{
    if (order.Status == "Completed")
    {
        total += order.Amount;
    }
}

// LINQ alternative; Sum enumerates the filtered sequence.
decimal totalLinq = orders
    .Where(order => order.Status == "Completed")
    .Sum(order => order.Amount);
```

### Avoid Unnecessary Cross-Comparisons on Large Data Sets

A nested loop comparing every customer with every order performs `N × M` comparisons. If the relationship can be indexed by key, a dictionary lookup can reduce expected work substantially.

```csharp
using System.Collections.Generic;
using System.Linq;

var customerIds = new List<int>(); // Assume 10,000 customer IDs are loaded here.
var orders = new List<(int CustomerId, decimal Amount)>(); // Assume 100,000 orders are loaded here.

// Naive comparison: O(N * M), or 1 billion comparisons at these sizes.
// foreach (int customerId in customerIds)
// {
//     foreach (var order in orders)
//     {
//         if (order.CustomerId == customerId)
//         {
//             // Process the match.
//         }
//     }
// }

// Build a lookup once, then retrieve each customer's matching orders.
var ordersByCustomer = orders
    .GroupBy(order => order.CustomerId)
    .ToDictionary(group => group.Key, group => group.ToList());

foreach (int customerId in customerIds)
{
    if (ordersByCustomer.TryGetValue(customerId, out var customerOrders))
    {
        // Process customerOrders.
    }
}
```

The grouping and dictionary construction require memory; dictionary lookup is expected O(1) on average, not a universal worst-case guarantee. Use this approach when the relationship and workload justify the additional index.

## Basic Code Snippet

This complete .NET 10 console example demonstrates the four loop forms, dictionary deconstruction, and loop control statements.

```csharp
using System;
using System.Collections.Generic;

// --- for loop ---
Console.WriteLine("=== for Loop ===");
for (int i = 1; i <= 5; i++)
{
    Console.WriteLine($"  {i} x {i} = {i * i}");
}

// --- foreach loop ---
Console.WriteLine("\n=== foreach Loop ===");
var fruits = new List<string> { "Apple", "Banana", "Cherry", "Date" };

foreach (string fruit in fruits)
{
    Console.WriteLine($"  Fruit: {fruit} ({fruit.Length} chars)");
}

// --- foreach with a dictionary ---
Console.WriteLine("\n=== foreach Dictionary ===");
var scores = new Dictionary<string, int>
{
    ["Alice"] = 92,
    ["Bob"] = 78,
    ["Charlie"] = 85
};

foreach (var (name, score) in scores)
{
    string grade = score >= 90 ? "A" : score >= 80 ? "B" : "C";
    Console.WriteLine($"  {name}: {score} ({grade})");
}

// --- while loop ---
Console.WriteLine("\n=== while Loop ===");
decimal balance = 1_000m;
decimal monthlyFee = 150.50m;
int month = 0;

while (balance >= monthlyFee)
{
    balance -= monthlyFee;
    month++;
}

Console.WriteLine($"  Cannot cover another fee after {month} months. Remaining: {balance}");

// --- do-while loop ---
Console.WriteLine("\n=== do-while Loop ===");
int factorial = 1;
int n = 5;
int counter = 1;

do
{
    factorial *= counter;
    counter++;
} while (counter <= n);

Console.WriteLine($"  {n}! = {factorial}");

// --- break and continue ---
Console.WriteLine("\n=== break and continue ===");
int[] data = { 3, 7, 1, 9, 4, 0, 8, 2 };

Console.Write("  Values (skip 0, stop at 9): ");
foreach (int value in data)
{
    if (value == 0)
    {
        continue;
    }

    if (value == 9)
    {
        Console.Write("[STOP] ");
        break;
    }

    Console.Write($"{value} ");
}
Console.WriteLine();
```

## Backend Batch Processing Example (Illustrative)

This example combines a `while` loop for batched retrieval, `foreach` for each order, and `do`–`while` for bounded retries. It is a teaching example, not a drop-in production batch processor. It uses keyset pagination, propagates cancellation, and shows a simple failure threshold; real payment, persistence, notification, and retry behavior require application-specific safeguards.

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace MyBackendApp.Core.Services;

public sealed class BatchOrderProcessor
{
    private const int BatchSize = 500;
    private const int MaxRetryAttempts = 3;
    private const int BaseRetryDelayMilliseconds = 1_000;
    private const int MaxRetryDelayMilliseconds = 8_000;
    private const int MaxConsecutiveFailures = 10;

    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentGateway _paymentGateway;
    private readonly INotificationService _notificationService;
    private readonly IBatchLogger _logger;

    public BatchOrderProcessor(
        IOrderRepository orderRepository,
        IPaymentGateway paymentGateway,
        INotificationService notificationService,
        IBatchLogger logger)
    {
        _orderRepository = orderRepository;
        _paymentGateway = paymentGateway;
        _notificationService = notificationService;
        _logger = logger;
    }

    public async Task<BatchProcessingResult> ProcessPendingOrdersAsync(
        CancellationToken cancellationToken)
    {
        var result = new BatchProcessingResult();
        int? afterOrderId = null;
        int consecutiveFailures = 0;

        _logger.LogInformation("Starting batch order processing.");

        while (true)
        {
            cancellationToken.ThrowIfCancellationRequested();

            // The repository must return rows ordered by increasing Id and use
            // Id > afterOrderId as its cursor condition.
            List<Order> batch = await _orderRepository.GetPendingOrdersAfterAsync(
                afterOrderId, BatchSize, cancellationToken);

            if (batch.Count == 0)
            {
                _logger.LogInformation("No more pending orders. Batch processing complete.");
                break;
            }

            _logger.LogInformation(
                "Fetched {Count} orders after cursor {AfterOrderId}.",
                batch.Count, afterOrderId);

            foreach (Order order in batch)
            {
                cancellationToken.ThrowIfCancellationRequested();

                bool succeeded = await ProcessOrderWithRetryAsync(order, cancellationToken);
                afterOrderId = order.Id;
                result.TotalProcessed++;

                if (succeeded)
                {
                    result.SuccessCount++;
                    consecutiveFailures = 0;
                }
                else
                {
                    result.FailureCount++;
                    result.FailedOrderIds.Add(order.Id);
                    consecutiveFailures++;
                }

                // This is a simple stop threshold, not a complete circuit-breaker
                // implementation with open, half-open, and reset states.
                if (consecutiveFailures >= MaxConsecutiveFailures)
                {
                    result.StoppedAfterFailureThreshold = true;
                    _logger.LogError(
                        "Stopping after {Count} consecutive failed orders.",
                        consecutiveFailures);
                    return result;
                }
            }

            // For a stable ascending keyset query, a short batch means no more
            // rows were available at the time of the query.
            if (batch.Count < BatchSize)
            {
                break;
            }
        }

        _logger.LogInformation(
            "Batch complete. Success: {Success}, Failed: {Failed}, Total: {Total}.",
            result.SuccessCount, result.FailureCount, result.TotalProcessed);

        return result;
    }

    private async Task<bool> ProcessOrderWithRetryAsync(
        Order order,
        CancellationToken cancellationToken)
    {
        int attempt = 0;

        do
        {
            cancellationToken.ThrowIfCancellationRequested();
            attempt++;

            try
            {
                await ProcessSingleOrderAsync(order, cancellationToken);
                return true;
            }
            catch (OperationCanceledException)
            {
                // Do not convert cancellation into an ordinary order failure.
                throw;
            }
            catch (PaymentGatewayException ex) when (ex.IsTransient)
            {
                _logger.LogWarning(
                    "Transient error on order {OrderId}, attempt {Attempt}: {Message}",
                    order.Id, attempt, ex.Message);

                if (attempt < MaxRetryAttempts)
                {
                    int multiplier = 1 << (attempt - 1);
                    int delayMilliseconds = Math.Min(
                        MaxRetryDelayMilliseconds,
                        BaseRetryDelayMilliseconds * multiplier);

                    await Task.Delay(delayMilliseconds, cancellationToken);
                }
            }
            catch (Exception ex)
            {
                _logger.LogError(
                    ex, "Non-retryable error processing order {OrderId}.", order.Id);
                return false;
            }
        } while (attempt < MaxRetryAttempts);

        _logger.LogError(
            "Order {OrderId} failed after {Attempts} attempts.",
            order.Id, MaxRetryAttempts);
        return false;
    }

    private async Task ProcessSingleOrderAsync(
        Order order,
        CancellationToken cancellationToken)
    {
        // The gateway should honor this stable idempotency key so a retry after
        // a timeout cannot accidentally charge the same order twice.
        string idempotencyKey = $"order-{order.Id}";
        PaymentResult payment = await _paymentGateway.ChargeAsync(
            order.PaymentDetails, idempotencyKey, cancellationToken);

        if (!payment.IsSuccess)
        {
            throw new PaymentGatewayException(
                $"Payment failed: {payment.ErrorMessage}",
                isTransient: payment.IsRetryable);
        }

        order.Status = OrderStatus.Processing;
        order.PaymentReference = payment.TransactionId;
        order.UpdatedAt = DateTime.UtcNow;

        await _orderRepository.UpdateAsync(order, cancellationToken);

        // Production notification delivery is often made reliable with an outbox.
        await _notificationService.SendOrderConfirmationAsync(
            order.CustomerEmail, order.OrderNumber, cancellationToken);
    }

    public Dictionary<string, int> GenerateStatusSummary(List<Order> orders)
    {
        var summary = new Dictionary<string, int>();

        foreach (Order order in orders)
        {
            string statusKey = order.Status.ToString();

            if (summary.TryGetValue(statusKey, out int currentCount))
            {
                summary[statusKey] = currentCount + 1;
            }
            else
            {
                summary[statusKey] = 1;
            }
        }

        return summary;
    }

    public List<Order> FilterOrdersByDateRange(
        List<Order> orders,
        DateTime startDate,
        DateTime endDate)
    {
        var filtered = new List<Order>(orders.Count);

        // Use for when list indexing or the index itself is useful.
        for (int i = 0; i < orders.Count; i++)
        {
            Order order = orders[i];

            if (order.CreatedAt >= startDate && order.CreatedAt <= endDate)
            {
                filtered.Add(order);
            }
        }

        _logger.LogInformation(
            "Filtered {Filtered} of {Total} orders for date range {Start} to {End}.",
            filtered.Count, orders.Count, startDate, endDate);

        return filtered;
    }
}

public sealed class BatchProcessingResult
{
    // TotalProcessed counts orders for which success/failure handling completed.
    public int TotalProcessed { get; set; }
    public int SuccessCount { get; set; }
    public int FailureCount { get; set; }
    public List<int> FailedOrderIds { get; } = new();
    public bool StoppedAfterFailureThreshold { get; set; }
}

public sealed class Order
{
    public int Id { get; set; }
    public string OrderNumber { get; set; } = string.Empty;
    public string CustomerEmail { get; set; } = string.Empty;
    public OrderStatus Status { get; set; }
    public decimal TotalAmount { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    public string? PaymentReference { get; set; }
    public PaymentDetails PaymentDetails { get; set; } = new();
}

public sealed class PaymentDetails
{
    public string Method { get; set; } = string.Empty;
    public string Token { get; set; } = string.Empty;
}

public sealed class PaymentResult
{
    public bool IsSuccess { get; set; }
    public string? TransactionId { get; set; }
    public string? ErrorMessage { get; set; }
    public bool IsRetryable { get; set; }
}

public sealed class PaymentGatewayException : Exception
{
    public bool IsTransient { get; }

    public PaymentGatewayException(string message, bool isTransient)
        : base(message)
    {
        IsTransient = isTransient;
    }
}

public enum OrderStatus
{
    Pending,
    Processing,
    Completed,
    Cancelled
}

// Interfaces are simplified stubs for this teaching example.
public interface IOrderRepository
{
    // Returns pending orders in ascending Id order, after the supplied cursor.
    Task<List<Order>> GetPendingOrdersAfterAsync(
        int? afterOrderId,
        int pageSize,
        CancellationToken cancellationToken);

    Task UpdateAsync(Order order, CancellationToken cancellationToken);
}

public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(
        PaymentDetails details,
        string idempotencyKey,
        CancellationToken cancellationToken);
}

public interface INotificationService
{
    Task SendOrderConfirmationAsync(
        string email,
        string orderNumber,
        CancellationToken cancellationToken);
}

public interface IBatchLogger
{
    void LogInformation(string message, params object[] args);
    void LogWarning(string message, params object[] args);
    void LogError(string message, params object[] args);
    void LogError(Exception exception, string message, params object[] args);
}
```

### Key Observations

- The outer `while` loop fetches fixed-size batches. This example uses a keyset cursor (`Id > afterOrderId`) instead of assuming offset pages remain stable while processed records change state. A real repository must define ordering and concurrency/claiming behavior.
- `foreach` processes each order without managing an index; `for` is used in the date-range method because the example indexes a list.
- The `do`–`while` makes at least one attempt and retries only exceptions classified as transient. The delay doubles by attempt and is capped; production policies commonly add jitter and should be designed for the service's timeouts and retry budget.
- The cancellation token is checked between iterations and passed through asynchronous operations. `OperationCanceledException` is propagated rather than recorded as an ordinary failure; the caller or host can then handle shutdown.
- The failure counter increments once for each order that ultimately fails and resets after a success. The threshold is a simplified stop rule, not a complete circuit breaker.
- Payment retries require idempotency, and reliable notification delivery often needs an outbox or another delivery design. A retry around side effects must not duplicate charges or notifications.
- `GenerateStatusSummary` is a single-pass dictionary aggregation. It avoids building LINQ grouping objects, but which approach is faster depends on the data and should be measured.
- `FilterOrdersByDateRange` pre-allocates a result list using the input count as an upper bound. This can reduce resizing, but may reserve more capacity than the filtered result needs.

## Key Terms Summary

| Term | Definition |
|---|---|
| `for` loop | A loop with initialization, a condition, and an iteration section. |
| `foreach` loop | Enumerates values from a collection or a type that supports the C# enumerator pattern. |
| `while` loop | Checks a condition before each iteration; its body may run zero times. |
| `do`–`while` loop | Checks a condition after the body; its body runs at least once. |
| `break` | Exits the nearest enclosing loop or `switch` statement. |
| `continue` | Skips the rest of the current loop iteration and proceeds to the next. |
| Nested loop | A loop inside another loop; total work can be the product of their iteration counts. |
| Exponential backoff | A retry delay that grows exponentially, often with a cap and jitter. |
| Pagination | Retrieving data in batches instead of loading an entire data set at once. |
| Keyset pagination | Paging from a stable ordered key/cursor rather than an offset. |
| `CancellationToken` | A cooperative signal used to request cancellation of operations and loops. |
| Enumerator | An object or pattern that supplies the next value in a sequence. |

## Further Reading

- [3](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements) — C# iteration statements and the `foreach` enumerator pattern.
- [2](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2?view=net-10.0) — `Dictionary<TKey, TValue>` enumeration and ordering remarks.
- [1](https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-encoding-introduction) — UTF-16 code units, `Rune`, and text elements in .NET.
