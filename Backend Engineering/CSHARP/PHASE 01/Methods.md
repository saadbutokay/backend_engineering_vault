## What Is a Method?

A method is a named block of code that performs a task. A program executes its statements when the method is called with the required arguments. Methods help you:

- Organize code into logical, reusable units.
- Avoid duplicating logic.
- Hide implementation details behind a clear name.
- Test behavior in smaller pieces.

Backend work such as validating a request, calculating a price, querying a repository, or sending a notification is commonly organized into methods. The examples target .NET 10; version-specific C# features are noted where relevant.

**Parameters** are the input variables declared in a method signature; **arguments** are the values supplied at a call site.

## Method Declaration and Structure

Methods are declared in a class, struct, or interface. In a top-level `Program.cs`, you can also define local functions, but those don't have access modifiers such as `public`.

### Syntax

```csharp
accessModifier returnType MethodName(parameterType parameterName)
{
    // Method body.
    return value; // Required on normal completion when a value is returned.
}
```

A `void` method returns no value. An asynchronous method commonly returns `Task` or `Task<T>`; those return types are introduced later in this section.

### Components and Naming

- **Access modifier:** controls where the method can be called from, subject to the visibility of its containing type. Common modifiers include `public`, `private`, `protected`, and `internal`.
- **Return type:** the type of result, or `void` when there is no result.
- **Method name:** use PascalCase for methods, with a verb or verb phrase such as `CalculateTotal`, `GetUserById`, or `ValidateInput`.
- **Parameters:** zero or more typed inputs in parentheses.
- **Method body:** the statements that run when the method is called.

### Basic Example

```csharp
using System;

public sealed class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

Call the method through an instance of its containing type:

```csharp
using System;

var calculator = new Calculator();
int result = calculator.Add(5, 3);
Console.WriteLine(result); // 8
```

## Void Methods

A `void` method performs an action without returning a value. A `return;` statement can exit a `void` method early.

```csharp
using System;

public sealed class ConsoleLogger
{
    public void LogMessage(string message)
    {
        Console.WriteLine($"[{DateTimeOffset.UtcNow:HH:mm:ss}] {message}");
    }
}
```

```csharp
var logger = new ConsoleLogger();
logger.LogMessage("Application started.");
```

A guard clause can make the early-exit case clear:

```csharp
using System;

#nullable enable
public sealed record Order(int Id);

public static class OrderProcessor
{
    public static void ProcessOrder(Order? order)
    {
        if (order is null)
        {
            return; // Nothing to process.
        }

        Console.WriteLine($"Processing order #{order.Id}");
    }
}
```

## Parameters

C# provides several parameter forms that control how values are passed to and from methods.

### Value Parameters (Default)

By default, an argument is passed by value:

- For a value type such as `int`, the method receives a copy of the value.
- For a reference type such as `List<int>`, the method receives a copy of the reference. It can mutate the referenced object, but reassigning its local parameter doesn't reassign the caller's variable.

```csharp
using System;
using System.Collections.Generic;

public static class ParameterExamples
{
    public static void ModifyValue(int number)
    {
        number = 100; // Changes this local copy only.
    }

    public static void ModifyList(List<int> items)
    {
        items.Add(999);             // Mutates the same List<int> object.
        items = new List<int>();    // Reassigns only this local copy of the reference.
    }
}
```

```csharp
using System;
using System.Collections.Generic;

int x = 5;
ParameterExamples.ModifyValue(x);
Console.WriteLine(x); // 5

var list = new List<int> { 1, 2, 3 };
ParameterExamples.ModifyList(list);
Console.WriteLine(list.Count); // 4
```

### The `ref` Keyword

`ref` passes a variable by reference, so assigning to the parameter changes the caller's variable. The argument must be initialized before the call. Both the declaration and the call use `ref`.

```csharp
public static class RefExamples
{
    public static void DoubleValue(ref int number)
    {
        number *= 2;
    }
}
```

```csharp
using System;

int value = 10;
RefExamples.DoubleValue(ref value);
Console.WriteLine(value); // 20

// RefExamples.DoubleValue(value); // Compile error: the call must include ref.
```

### The `out` Keyword

`out` is useful when a method produces an additional result through a parameter. The caller doesn't need to initialize the variable, but the method must assign it on every path that returns normally.

```csharp
#nullable enable
public static class AgeParser
{
    public static bool TryParseAge(string? input, out int age)
    {
        if (int.TryParse(input, out int parsed) && parsed is >= 0 and <= 150)
        {
            age = parsed;
            return true;
        }

        age = 0; // Assigned even when parsing or validation fails.
        return false;
    }
}
```

```csharp
using System;

string userInput = "25";

if (AgeParser.TryParseAge(userInput, out int userAge))
{
    Console.WriteLine($"Valid age: {userAge}");
}
else
{
    Console.WriteLine("Invalid age.");
}
```

The `TryParse` pattern commonly returns a `bool` for success and writes the parsed value through an `out` parameter.

### `ref` vs. `out`

| Characteristic | `ref` | `out` |
|---|---|---|
| Must be initialized before the call | Yes | No |
| Must be assigned before normal return | No | Yes |
| Main intent | Read and/or update the caller's variable | Produce a value through a parameter |
| Common example | In-place update | `TryParse` pattern |

### The `in` Keyword

`in` declares an input parameter that the method can't assign. For a suitable variable, the argument is passed by read-only reference; the compiler may use a temporary for constants, properties, or conversions. It's mainly useful when passing a large value type by reference could avoid a copy. For a reference-type argument, `in` doesn't make the referenced object immutable.

```csharp
using System;

public readonly struct Point3D
{
    public double X { get; }
    public double Y { get; }
    public double Z { get; }

    public Point3D(double x, double y, double z)
    {
        X = x;
        Y = y;
        Z = z;
    }
}

public static class Geometry
{
    public static double CalculateDistance(in Point3D a, in Point3D b)
    {
        // a = default; // Compile error: an in parameter is read-only.
        double dx = a.X - b.X;
        double dy = a.Y - b.Y;
        double dz = a.Z - b.Z;
        return Math.Sqrt(dx * dx + dy * dy + dz * dz);
    }
}
```

```csharp
var a = new Point3D(0, 0, 0);
var b = new Point3D(3, 4, 0);

double distance = Geometry.CalculateDistance(in a, in b);
// The call-site in modifier is optional for an in parameter:
distance = Geometry.CalculateDistance(a, b);
```

`in` isn't a universal performance improvement, and mutable structs can involve defensive copies when members are called. Prefer immutable/`readonly struct` designs when appropriate, and benchmark before adding `in` to performance-sensitive APIs. There is no reliable fixed-size threshold such as “16 bytes or more” that guarantees a benefit.

### The `params` Keyword

An array-based `params` parameter lets the caller supply zero or more arguments of the element type, or pass an array. It must be the last parameter in the method declaration.

```csharp
public static class Statistics
{
    public static decimal CalculateAverage(params decimal[] values)
    {
        if (values.Length == 0)
        {
            return 0m; // This method chooses zero as its empty-input result.
        }

        decimal sum = 0m;
        foreach (decimal value in values)
        {
            sum += value;
        }

        return sum / values.Length;
    }
}
```

```csharp
decimal average1 = Statistics.CalculateAverage(10m, 20m, 30m); // 20

decimal average2 = Statistics.CalculateAverage(5m, 15m); // 10
decimal average3 = Statistics.CalculateAverage(100m); // 100
decimal average4 = Statistics.CalculateAverage(); // 0

decimal[] inputs = { 1m, 2m, 3m };
decimal average5 = Statistics.CalculateAverage(inputs); // 2
```

C# 13 and later also support `params` with other supported collection types. The `params T[]` form shown here remains valid and is supported by earlier C# versions.

### Named Arguments

Named arguments identify parameters by name. They're useful when a call has several parameters or boolean options.

```csharp
using System;

public static class EmailSender
{
    public static void SendEmail(
        string to,
        string subject,
        string body,
        bool isHtml = false,
        int priority = 0)
    {
        Console.WriteLine(
            $"To: {to}, Subject: {subject}, HTML: {isHtml}, Priority: {priority}");
    }
}
```

```csharp
// Positional call: the meaning of true and 1 isn't obvious at a glance.
EmailSender.SendEmail("alice@example.com", "Hello", "Hi Alice", true, 1);

// Named call.
EmailSender.SendEmail(
    to: "alice@example.com",
    subject: "Hello",
    body: "Hi Alice",
    isHtml: true,
    priority: 1);

// Named arguments can be written in any order when all arguments are named.
EmailSender.SendEmail(
    priority: 1,
    isHtml: true,
    body: "Hi Alice",
    subject: "Hello",
    to: "alice@example.com");

// Positional and named arguments can be mixed when the positional arguments
// remain in their corresponding parameter positions (C# 7.2 and later).
EmailSender.SendEmail("alice@example.com", "Hello", "Hi Alice", priority: 1);
EmailSender.SendEmail(to: "alice@example.com", "Hello", "Hi Alice");
```

### Optional Parameters

An optional parameter has a default value and can be omitted at the call site. Optional parameters must follow required parameters. Their defaults must be values permitted by the language, such as constants or `default` values.

```csharp
using System;
using System.Globalization;

public static class AmountFormatter
{
    // This is a simple invariant, symbol-prefixed format, not full localization.
    public static string FormatCurrency(
        decimal amount,
        string currencySymbol = "$",
        int decimalPlaces = 2)
    {
        if (decimalPlaces is < 0 or > 28)
        {
            throw new ArgumentOutOfRangeException(nameof(decimalPlaces));
        }

        string format = "F" + decimalPlaces.ToString(CultureInfo.InvariantCulture);
        string formattedAmount = amount.ToString(format, CultureInfo.InvariantCulture);
        return currencySymbol + formattedAmount;
    }
}
```

```csharp
using System;

Console.WriteLine(AmountFormatter.FormatCurrency(49.99m));          // $49.99
Console.WriteLine(AmountFormatter.FormatCurrency(49.99m, "€"));     // €49.99
Console.WriteLine(AmountFormatter.FormatCurrency(49.99m, "¥", 0)); // ¥50
```

For localized currency display, use a `CultureInfo` and its currency formatting rather than simply prefixing a symbol. Optional argument values are supplied by the compiled caller, so changing a library's default doesn't change already-compiled call sites until they're recompiled.

## Return Types

### A Single Return Value

A method can return a value of any type, including a value type, a reference type, or a collection.

```csharp
public static class CustomerFormatting
{
    public static string GetFullName(string firstName, string lastName)
    {
        return $"{firstName} {lastName}";
    }

    public static bool IsAdult(int age)
    {
        return age >= 18;
    }

    public static decimal CalculateTax(decimal amount, decimal rate)
    {
        return amount * rate;
    }
}
```

### Returning Multiple Values with Tuples

A named value tuple is a concise way to return a small group of related values without defining a separate class or using `out` parameters.

```csharp
public static class OrderMath
{
    public static (decimal Subtotal, decimal Tax, decimal Total) CalculateOrderTotal(
        decimal itemPrice, int quantity, decimal taxRate)
    {
        decimal subtotal = itemPrice * quantity;
        decimal tax = subtotal * taxRate;
        decimal total = subtotal + tax;

        return (subtotal, tax, total);
    }
}
```

```csharp
using System;

var (subtotal, tax, total) = OrderMath.CalculateOrderTotal(29.99m, 3, 0.08m);
Console.WriteLine($"Subtotal: {subtotal}, Tax: {tax}, Total: {total}");

var result = OrderMath.CalculateOrderTotal(29.99m, 3, 0.08m);
Console.WriteLine($"Total: {result.Total}");
```

Tuples are useful for small results; use a named result type when the data has domain meaning, validation rules, or a larger/stable public shape. An `async` method can't declare `ref`, `out`, or `in` parameters, so asynchronous methods return additional results in `Task<T>`—often a tuple or a result type—instead.

### Returning Collections

The return type communicates how callers are expected to consume a sequence. Choose it based on the contract, materialization, and mutability—not a blanket rule that one interface is always best.

```csharp
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;

public sealed class Order
{
    public int CustomerId { get; init; }
    public string Status { get; init; } = string.Empty;
}

public sealed class OrderQueries
{
    private readonly List<Order> _orders;

    public OrderQueries(IEnumerable<Order> orders)
    {
        ArgumentNullException.ThrowIfNull(orders);
        _orders = orders.ToList();
    }

    private static readonly ReadOnlyCollection<string> AllowedRoles =
        Array.AsReadOnly(new[] { "Admin", "Manager", "Editor", "Viewer" });

    // Return an array when an independent, fixed-size result is useful.
    public string[] GetDaysOfWeek() =>
        new[] { "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday" };

    // A mutable snapshot: callers can change the returned list without changing _orders.
    public List<Order> GetActiveOrdersSnapshot() =>
        _orders.Where(order => order.Status == "Active").ToList();

    // Deferred query: enumeration happens later, against the source list.
    public IEnumerable<Order> GetOrdersByCustomer(int customerId) =>
        _orders.Where(order => order.CustomerId == customerId);

    // Read-only collection surface; the private backing data isn't exposed elsewhere.
    public IReadOnlyList<string> GetAllowedRoles() => AllowedRoles;
}
```

`IEnumerable<T>` can represent deferred execution; the work may happen when the caller enumerates, and repeated enumeration may repeat the work. `IReadOnlyList<T>` provides indexed read access and `Count`, but the interface alone doesn't guarantee that the underlying data or its elements are immutable. Use a defensive copy, a read-only wrapper, or an immutable collection when that distinction matters.

## Expression-Bodied Methods

An expression-bodied method uses `=>` for a single expression. It's a syntax alternative to a block body, not a performance feature.

```csharp
public static class Pricing
{
    // Traditional form.
    public static bool IsEven(int number)
    {
        return number % 2 == 0;
    }

    // Expression-bodied equivalent.
    public static bool IsEvenCompact(int number) => number % 2 == 0;

    public static decimal ApplyDiscount(decimal price, decimal discount) =>
        price * (1 - discount);

    public static string FormatName(string first, string last) =>
        $"{last}, {first}";

    // A quick shape check only; this isn't complete email-address validation.
    public static bool LooksLikeEmail(string? email) =>
        !string.IsNullOrWhiteSpace(email) && email.Contains('@');
}
```

Use an expression body when it makes a small method clearer. A block is easier to read when you need validation, multiple steps, or more involved control flow.

## Method Overloading

Overloading defines methods with the same name in a type but different parameter lists. The compiler chooses an applicable overload based on the arguments and conversion rules.

Important rules:

- Overloads must differ in their parameter lists, such as by parameter count, types, or order.
- The return type alone doesn't distinguish overloads; parameter names and optional default values alone don't either.
- A method can't be overloaded solely by changing a parameter from `ref` to `out`.

```csharp
using System;

public sealed class Calculator
{
    public int Add(int a, int b) => a + b;

    public double Add(double a, double b) => a + b;

    public decimal Add(decimal a, decimal b) => a + b;

    public int Add(int a, int b, int c) => a + b + c;

    public double Add(params double[] values)
    {
        double sum = 0;
        foreach (double value in values)
        {
            sum += value;
        }

        return sum;
    }
}
```

```csharp
using System;

var calculator = new Calculator();
Console.WriteLine(calculator.Add(2, 3));                 // 5: int overload
Console.WriteLine(calculator.Add(2.5, 3.5));             // 6: double overload
Console.WriteLine(calculator.Add(1.99m, 2.99m));         // 4.98: decimal overload
Console.WriteLine(calculator.Add(1, 2, 3));              // 6: three-int overload
Console.WriteLine(calculator.Add(1.0, 2.0, 3.0, 4.0));   // 10: params overload
```

### Overloading with Optional Parameters

Combining overloads and optional parameters can make calls surprising, but not every such combination is ambiguous. If an exact overload needs no omitted optional arguments, it is generally preferred over an otherwise equal candidate that needs a default substituted.

For example, with `Log(string message)` and `Log(string message, string level = "INFO")`, `Log("Hello")` selects the one-parameter overload; it isn't ambiguous.

A genuinely ambiguous call can occur when two candidates both need an optional value and neither is a better match:

```csharp
using System;

public static class Logger
{
    public static void Log(string message, string level = "INFO")
    {
        Console.WriteLine($"[{level}] {message}");
    }

    public static void Log(string message, bool includeTimestamp = false)
    {
        string timestamp = includeTimestamp ? $"{DateTimeOffset.UtcNow:O} " : string.Empty;
        Console.WriteLine($"{timestamp}{message}");
    }
}
```

```csharp
// Logger.Log("Hello"); // Compile-time ambiguity: both optional arguments would be omitted.
Logger.Log("Hello", level: "WARN"); // Unambiguous.
Logger.Log("Hello", includeTimestamp: true); // Unambiguous.
```

Prefer overload sets whose intent is easy to determine, and avoid optional-parameter combinations that make common calls unclear.

## Local Functions

A local function is declared inside another member and is callable only within that member. It can keep a small helper close to the code that uses it, and a non-static local function can capture variables from the enclosing scope.

```csharp
public static class ShippingCalculator
{
    public static decimal CalculateShipping(decimal weight, string region)
    {
        static decimal GetBaseRate(string area) => area switch
        {
            "Domestic" => 5.00m,
            "Continental" => 15.00m,
            "International" => 25.00m,
            _ => 10.00m
        };

        decimal ApplySurcharge(decimal baseRate)
        {
            decimal surcharge = weight > 50m ? baseRate * 0.20m : 0m;
            return baseRate + surcharge;
        }

        decimal baseRate = GetBaseRate(region);
        return ApplySurcharge(baseRate);
    }
}
```

A `static` local function can't capture local variables or instance state. Use `static` when capture isn't needed; that restriction improves clarity and can help avoid closure state. Don't assume every non-static local function creates a heap allocation—allocation behavior depends on how it captures variables and is used.

```csharp
using System;

public static class MathHelpers
{
    public static int Factorial(int n)
    {
        if (n < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(n));
        }

        return Compute(n);

        static int Compute(int value)
        {
            if (value <= 1)
            {
                return 1;
            }

            return checked(value * Compute(value - 1));
        }
    }
}
```

This `int` example is suitable only for small values; checked multiplication throws if the result overflows.

## Extension Methods

A traditional extension method is a `static` method in a top-level, non-nested `static` class. Its first parameter has the `this` modifier, which lets callers use instance-method syntax without changing the extended type.

```csharp
#nullable enable
using System;

public static class StringExtensions
{
    // A heuristic only, not full email validation.
    public static bool LooksLikeEmail(this string? email)
    {
        return !string.IsNullOrWhiteSpace(email)
            && email.Contains('@')
            && email.Contains('.');
    }

    public static string Truncate(this string text, int maxLength)
    {
        ArgumentNullException.ThrowIfNull(text);
        ArgumentOutOfRangeException.ThrowIfNegative(maxLength);

        if (text.Length <= maxLength)
        {
            return text;
        }

        return text[..maxLength] + "...";
    }
}
```

```csharp
using System;

string email = "alice@example.com";
Console.WriteLine(email.LooksLikeEmail()); // True (only a simple heuristic)

string longText = "This is a very long description that needs truncation.";
Console.WriteLine(longText.Truncate(19)); // This is a very long...
```

The `Truncate` example counts UTF-16 code units and appends three more characters, so its result can be `maxLength + 3` code units long. For user-facing text, consider Unicode text-element boundaries; a simple range can split a surrogate pair or grapheme cluster. Extension methods are resolved at compile time, don't add members to the original type, and don't replace matching instance members. LINQ methods such as `Where`, `Select`, and `ToList` are familiar examples of extension methods.

The traditional `this` syntax remains supported in .NET 10. C# 14 also adds extension blocks, which are outside this introductory example.

## Async Methods (Overview)

An asynchronous method commonly returns `Task` or `Task<T>` and uses `async` and `await`. The caller can await the returned task to observe completion, results, and exceptions. Use `async void` only for event handlers; callers can't await it or reliably observe its exceptions.

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

public sealed class UserService
{
    public async Task<string> FetchUserNameAsync(
        int userId,
        CancellationToken cancellationToken)
    {
        // Demonstration placeholder; a real service would await an I/O API.
        await Task.Delay(100, cancellationToken);
        return $"User {userId}";
    }

    public async Task<int> CalculateTotalAsync(
        IEnumerable<int> orderIds,
        Func<int, CancellationToken, Task<int>> getOrderAmountAsync,
        CancellationToken cancellationToken)
    {
        ArgumentNullException.ThrowIfNull(orderIds);
        ArgumentNullException.ThrowIfNull(getOrderAmountAsync);

        int total = 0;
        foreach (int orderId in orderIds)
        {
            // These requests are intentionally awaited one at a time.
            int amount = await getOrderAmountAsync(orderId, cancellationToken);
            total = checked(total + amount);
        }

        return total;
    }
}
```

Async methods can't declare `ref`, `out`, or `in` parameters. If an asynchronous operation needs to return multiple values, include them in its `Task<T>` result, for example as a named tuple or a result type. Pass a `CancellationToken` to operations that support cancellation. A loop that awaits each item sequentially is appropriate when order or dependencies require it; for independent I/O, use an intentional, bounded concurrency strategy rather than automatically starting unbounded work.

## Basic Code Snippet

This complete .NET 10 console example demonstrates methods, `ref`, `out`, `params`, tuples, named/optional arguments, overloading, expression-bodied methods, and local functions.

```csharp
using System;
using System.Globalization;

Console.WriteLine("=== Basic Methods ===");
int sum = Add(10, 20);
Console.WriteLine($"10 + 20 = {sum}");
Console.WriteLine(BuildGreeting("Alice", "Good morning"));

Console.WriteLine("\n=== ref and out ===");
int number = 5;
DoubleIt(ref number);
Console.WriteLine($"After DoubleIt: {number}");

string input = "42";
if (TryParsePositive(input, out int parsed))
{
    Console.WriteLine($"Parsed positive: {parsed}");
}

Console.WriteLine("\n=== params ===");
decimal average = Average(10m, 20m, 30m, 40m);
Console.WriteLine($"Average: {average}");

Console.WriteLine("\n=== Tuple return ===");
var (min, max, total) = GetStats(new[] { 5, 12, 3, 8, 21, 7 });
Console.WriteLine($"Min: {min}, Max: {max}, Total: {total}");

Console.WriteLine("\n=== Named and optional arguments ===");
SendNotification(to: "alice@example.com", message: "Your order shipped!");
SendNotification(to: "bob@example.com", message: "Password reset", channel: "SMS");

Console.WriteLine("\n=== Method overloading ===");
Console.WriteLine(PriceFormatter.FormatPrice(49.99m));
Console.WriteLine(PriceFormatter.FormatPrice(49.99m, "EUR"));

Console.WriteLine("\n=== Expression-bodied method ===");
Console.WriteLine($"Is 7 even? {IsEven(7)}");
Console.WriteLine($"Is 8 even? {IsEven(8)}");

Console.WriteLine("\n=== Local function ===");
Console.WriteLine($"6! = {Factorial(6)}");

static int Add(int a, int b) => a + b;

static string BuildGreeting(string name, string timeOfDay = "Hello") =>
    $"{timeOfDay}, {name}!";

static void DoubleIt(ref int value) => value *= 2;

static bool TryParsePositive(string input, out int result)
{
    if (int.TryParse(input, out int parsed) && parsed > 0)
    {
        result = parsed;
        return true;
    }

    result = 0;
    return false;
}

static decimal Average(params decimal[] values)
{
    if (values.Length == 0)
    {
        return 0m;
    }

    decimal sum = 0m;
    foreach (decimal value in values)
    {
        sum += value;
    }

    return sum / values.Length;
}

static (int Min, int Max, int Total) GetStats(int[] numbers)
{
    ArgumentNullException.ThrowIfNull(numbers);
    if (numbers.Length == 0)
    {
        throw new ArgumentException("At least one value is required.", nameof(numbers));
    }

    int min = numbers[0];
    int max = numbers[0];
    int total = 0;

    foreach (int number in numbers)
    {
        if (number < min) min = number;
        if (number > max) max = number;
        total = checked(total + number);
    }

    return (min, max, total);
}

static void SendNotification(string to, string message, string channel = "Email")
{
    Console.WriteLine($"[{channel}] To: {to} — {message}");
}

static bool IsEven(int number) => number % 2 == 0;

static int Factorial(int n)
{
    if (n < 0)
    {
        throw new ArgumentOutOfRangeException(nameof(n));
    }

    return Compute(n);

    static int Compute(int value) =>
        value <= 1 ? 1 : checked(value * Compute(value - 1));
}

// Type members can be overloaded; top-level local functions cannot.
public static class PriceFormatter
{
    public static string FormatPrice(decimal amount) =>
        "$" + amount.ToString("F2", CultureInfo.InvariantCulture);

    public static string FormatPrice(decimal amount, string currency) =>
        currency + " " + amount.ToString("F2", CultureInfo.InvariantCulture);
}
```

## Applied Backend Example (Illustrative)

The following service sketch demonstrates method organization, overloads, parameter validation, an `out` parameter, local functions, expression-bodied calculations, asynchronous calls, and cancellation. It is **illustrative rather than production-ready payment code**. A real payment workflow needs provider idempotency, a reliable persistence strategy, secure secret handling, and reconciliation for failures between an external charge and a database write.

The example avoids unobserved fire-and-forget work. A confirmation is queued through a repository operation intended to persist an outbox message atomically with the order update; a background worker can send the email later.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace MyBackendApp.Core.Services;

public sealed class OrderProcessingService
{
    private readonly IOrderRepository _repository;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IOrderLogger _logger;

    public OrderProcessingService(
        IOrderRepository repository,
        IPaymentGateway paymentGateway,
        IOrderLogger logger)
    {
        ArgumentNullException.ThrowIfNull(repository);
        ArgumentNullException.ThrowIfNull(paymentGateway);
        ArgumentNullException.ThrowIfNull(logger);

        _repository = repository;
        _paymentGateway = paymentGateway;
        _logger = logger;
    }

    // Create and process a new order.
    public async Task<OrderProcessingResult> ProcessOrderAsync(
        CreateOrderRequest request,
        CancellationToken cancellationToken = default)
    {
        ArgumentNullException.ThrowIfNull(request);

        ValidationResult validation = ValidateRequest(request);
        if (!validation.IsValid)
        {
            return OrderProcessingResult.Failure(validation.Errors);
        }

        Order order = BuildOrder(request);
        order = await _repository.SavePendingAsync(order, cancellationToken);
        return await ProcessPendingOrderAsync(order, cancellationToken);
    }

    // Overload: resume a previously saved pending order by ID.
    public async Task<OrderProcessingResult> ProcessOrderAsync(
        int orderId,
        CancellationToken cancellationToken = default)
    {
        if (orderId <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(orderId));
        }

        Order? order = await _repository.GetByIdAsync(orderId, cancellationToken);
        if (order is null)
        {
            return OrderProcessingResult.Failure(new[] { $"Order {orderId} was not found." });
        }

        if (order.Status != OrderStatus.Pending)
        {
            return OrderProcessingResult.Failure(
                new[] { $"Order is already {order.Status}." });
        }

        return await ProcessPendingOrderAsync(order, cancellationToken);
    }

    private async Task<OrderProcessingResult> ProcessPendingOrderAsync(
        Order order,
        CancellationToken cancellationToken)
    {
        // The gateway must honor this idempotency key on retries.
        PaymentResult payment = await _paymentGateway.ChargeAsync(
            order.TotalAmount,
            order.PaymentToken,
            order.IdempotencyKey,
            cancellationToken);

        if (!payment.IsSuccess || string.IsNullOrWhiteSpace(payment.TransactionId))
        {
            string reason = payment.ErrorMessage ?? "Payment was declined or returned no reference.";
            order.Status = OrderStatus.PaymentFailed;
            await _repository.MarkPaymentFailedAsync(order.Id, reason, cancellationToken);
            _logger.LogWarning("Payment failed for order {OrderNumber}: {Reason}", order.OrderNumber, reason);
            return OrderProcessingResult.Failure(new[] { $"Payment failed: {reason}" });
        }

        order.PaymentReference = payment.TransactionId;
        order.Status = OrderStatus.Confirmed;
        order.ConfirmedAt = DateTimeOffset.UtcNow;

        // This operation should save the order and outbox message in one transaction.
        await _repository.ConfirmAndQueueNotificationAsync(order, cancellationToken);

        _logger.LogInformation(
            "Order {OrderNumber} confirmed. Total: {Total}",
            order.OrderNumber, order.TotalAmount);

        return OrderProcessingResult.Success(order);
    }

    private ValidationResult ValidateRequest(CreateOrderRequest request)
    {
        var errors = new List<string>();

        // Local function keeps repeated required-field checks close to this method.
        void RequireNonEmpty(string? value, string fieldName)
        {
            if (string.IsNullOrWhiteSpace(value))
            {
                errors.Add($"{fieldName} is required.");
            }
        }

        RequireNonEmpty(request.IdempotencyKey, "Idempotency key");
        RequireNonEmpty(request.CustomerEmail, "Customer email");
        RequireNonEmpty(request.ShippingAddress, "Shipping address");
        RequireNonEmpty(request.PaymentToken, "Payment token");
        RequireNonEmpty(request.Region, "Region");

        if (request.Items is null || request.Items.Count == 0)
        {
            errors.Add("Order must contain at least one item.");
        }
        else
        {
            foreach (OrderItemRequest item in request.Items)
            {
                if (item.Quantity <= 0)
                {
                    errors.Add($"Item '{item.ProductName}' must have a positive quantity.");
                }

                if (item.UnitPrice < 0m)
                {
                    errors.Add($"Item '{item.ProductName}' can't have a negative price.");
                }
            }
        }

        return new ValidationResult(errors.Count == 0, errors.ToArray());
    }

    private Order BuildOrder(CreateOrderRequest request)
    {
        List<OrderItemRequest> items = request.Items;
        decimal subtotal = CalculateSubtotal(items);
        decimal tax = CalculateTax(subtotal, request.Region, out decimal taxRate);
        decimal shipping = CalculateShipping(items, request.ShippingMethod);
        decimal discount = CalculateDiscount(subtotal, request.PromoCode);

        return new Order
        {
            OrderNumber = GenerateOrderNumber(),
            IdempotencyKey = request.IdempotencyKey,
            CustomerEmail = request.CustomerEmail,
            ShippingAddress = request.ShippingAddress,
            PaymentToken = request.PaymentToken,
            ShippingMethod = request.ShippingMethod,
            Subtotal = subtotal,
            TaxAmount = tax,
            TaxRate = taxRate,
            ShippingCost = shipping,
            DiscountAmount = discount,
            TotalAmount = subtotal + tax + shipping - discount,
            Status = OrderStatus.Pending,
            CreatedAt = DateTimeOffset.UtcNow
        };
    }

    private static decimal CalculateSubtotal(List<OrderItemRequest> items) =>
        items.Sum(item => item.Quantity * item.UnitPrice);

    // Demonstrates out; these rates are placeholders, not a tax rules engine.
    private static decimal CalculateTax(decimal subtotal, string region, out decimal taxRate)
    {
        taxRate = region.ToUpperInvariant() switch
        {
            "US-CA" => 0.0725m,
            "US-NY" => 0.08m,
            "US-TX" => 0.0625m,
            "EU" => 0.20m,
            "UK" => 0.20m,
            _ => 0m
        };

        return subtotal * taxRate;
    }

    private decimal CalculateShipping(
        List<OrderItemRequest> items,
        string method = "Standard")
    {
        int totalItems = items.Sum(item => item.Quantity);
        decimal weight = totalItems * 0.5m; // Simplified demonstration estimate.

        decimal baseRate = method switch
        {
            "Express" => 24.99m,
            "Overnight" => 49.99m,
            _ => 9.99m
        };

        decimal subtotal = CalculateSubtotal(items);
        if (subtotal >= 75m && method == "Standard")
        {
            return 0m;
        }

        return baseRate + (weight > 10m ? (weight - 10m) * 1.50m : 0m);
    }

    private static decimal CalculateDiscount(decimal subtotal, string? promoCode)
    {
        if (string.IsNullOrWhiteSpace(promoCode))
        {
            return 0m;
        }

        decimal percentage = promoCode.ToUpperInvariant() switch
        {
            "SAVE10" => 0.10m,
            "SAVE20" => 0.20m,
            "WELCOME" => 0.15m,
            _ => 0m
        };

        return subtotal * percentage;
    }

    private static string GenerateOrderNumber() => $"ORD-{Guid.NewGuid():N}";
}

public sealed class CreateOrderRequest
{
    public string IdempotencyKey { get; set; } = string.Empty;
    public string CustomerEmail { get; set; } = string.Empty;
    public string ShippingAddress { get; set; } = string.Empty;
    public string PaymentToken { get; set; } = string.Empty;
    public string Region { get; set; } = "US";
    public string ShippingMethod { get; set; } = "Standard";
    public string? PromoCode { get; set; }
    public List<OrderItemRequest> Items { get; set; } = new();
}

public sealed class OrderItemRequest
{
    public string ProductName { get; set; } = string.Empty;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public sealed class Order
{
    public int Id { get; set; }
    public string OrderNumber { get; set; } = string.Empty;
    public string IdempotencyKey { get; set; } = string.Empty;
    public string CustomerEmail { get; set; } = string.Empty;
    public string ShippingAddress { get; set; } = string.Empty;
    public string PaymentToken { get; set; } = string.Empty;
    public string? PaymentReference { get; set; }
    public string ShippingMethod { get; set; } = "Standard";
    public decimal Subtotal { get; set; }
    public decimal TaxAmount { get; set; }
    public decimal TaxRate { get; set; }
    public decimal ShippingCost { get; set; }
    public decimal DiscountAmount { get; set; }
    public decimal TotalAmount { get; set; }
    public OrderStatus Status { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
    public DateTimeOffset? ConfirmedAt { get; set; }
}

public enum OrderStatus
{
    Pending,
    Confirmed,
    Processing,
    Shipped,
    Delivered,
    Cancelled,
    PaymentFailed
}

public sealed record PaymentResult(bool IsSuccess, string? TransactionId, string? ErrorMessage);
public sealed record ValidationResult(bool IsValid, string[] Errors);

public sealed record OrderProcessingResult(bool IsSuccess, Order? Order, string[] Errors)
{
    public static OrderProcessingResult Success(Order order) =>
        new(true, order, Array.Empty<string>());

    public static OrderProcessingResult Failure(string[] errors) =>
        new(false, null, errors);
}

public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id, CancellationToken cancellationToken);
    Task<Order> SavePendingAsync(Order order, CancellationToken cancellationToken);
    Task MarkPaymentFailedAsync(int orderId, string reason, CancellationToken cancellationToken);

    // Persist the confirmed order and notification outbox item atomically.
    Task ConfirmAndQueueNotificationAsync(Order order, CancellationToken cancellationToken);
}

public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(
        decimal amount,
        string paymentToken,
        string idempotencyKey,
        CancellationToken cancellationToken);
}

public interface IOrderLogger
{
    void LogInformation(string message, params object?[] args);
    void LogWarning(string message, params object?[] args);
}
```

### Key Observations

- Methods are members of types, except local functions and top-level local functions. Use access modifiers on type members, not top-level local functions. Local functions can't be overloaded, so overloaded examples belong to a type.
- A reference-type parameter is still passed by value by default: the copied reference can point to the same mutable object, while reassignment affects only the local copy.
- Use `ref` when the caller's variable itself must be changed. Use `out` when the method produces a value through a parameter; a tuple or result object can be clearer for multiple related results.
- `in` expresses a read-only reference, not a guaranteed speedup. Compiler temporaries, defensive copies, struct size, and call patterns all matter.
- `params T[]` is the familiar variable-argument form. In C# 13 and later, `params` can also target supported collection types.
- Named arguments can appear in any order when all are named. Positional and named arguments can be mixed when their positions remain valid; the old rule “positional arguments must always come first” is too strict for modern C#.
- Optional parameters improve call-site convenience, but their default values are part of the caller's compiled code. Avoid changing public defaults without considering compatibility.
- A tuple is convenient for a small set of related return values. A dedicated result type is usually clearer when the result is a domain concept or grows in complexity.
- `IEnumerable<T>` may defer work until enumeration. `IReadOnlyList<T>` provides a read-only API surface but doesn't by itself guarantee deep immutability.
- A static local function cannot capture outer variables. Use it to enforce no capture; treat any performance benefit as context-dependent.
- Don't discard a task for non-critical notifications with `_ = SomeAsyncMethod()` unless the task is safely observed and managed. A durable queue/outbox is a common backend pattern.
- Let cancellation and unexpected failures propagate to an appropriate error-handling boundary; avoid converting an `OperationCanceledException` into a generic failure result.
- The payment example is intentionally a sketch: real systems must handle idempotency, transaction boundaries, retries, and reconciliation. The `PaymentToken` is assumed to be an opaque provider reference, never raw card data. Placeholder tax and shipping rules aren't production policy.

## Key Terms Summary

| Term | Definition |
|---|---|
| Method | A named block of code invoked to perform an operation. |
| Parameter | A named input variable in a method declaration. |
| Argument | A value supplied for a parameter at a method call. |
| Return type | The type of result a method produces; `void` means no direct value. |
| `ref` | Passes a caller variable by reference so the method can read or update it. |
| `out` | Passes a caller variable for the method to assign before normal return. |
| `in` | Declares a read-only by-reference parameter; it doesn't guarantee a performance win. |
| `params` | Allows a variable number of arguments for a supported collection parameter. |
| Overloading | Defining methods with the same name and different parameter lists. |
| Optional parameter | A parameter with a default value that may be omitted at the call site. |
| Named argument | An argument associated with a parameter by name. |
| Expression-bodied method | A concise method written with `=>` and a single expression. |
| Local function | A function declared inside another member and scoped to that member. |
| Extension method | A static method callable with instance syntax through a `this` parameter. |
| Tuple return | Returning several related values together as a tuple. |
| `CancellationToken` | A value passed to support cooperative cancellation of an operation. |

## Further Reading

- [Methods — C# programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods) — method declarations, signatures, and return values.
- [Method parameters and modifiers](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters) — `ref`, `out`, `in`, and `params`, including C# 13 collection support.
- [Named and optional arguments](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments) — positional/named mixing, default values, and overload resolution.
- [Async methods](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async) — async return types and parameter restrictions.
- [Task-based Asynchronous Programming](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap) — task naming, cancellation, and async API patterns.
- [Local functions](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions) — capture rules and static local functions.
- [Extension methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) — traditional extension methods and C# 14 extension blocks.
- [Expression-bodied members](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members) — concise member syntax.
- [.NET coding conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions) — naming and formatting guidance.
