Examples target .NET 10. Code blocks are focused examples unless labeled as a complete program; don't concatenate every excerpt into one file.

## The “Billion-Dollar Mistake”

A null reference represents the absence of an object. Dereferencing one can throw `NullReferenceException`, a common runtime failure. Tony Hoare later described the null reference as his “billion-dollar mistake.” C# has two related but distinct nullable systems:

1. **Nullable value types**—`Nullable<T>`, written `T?`, wrap value types such as `int` or `DateTime` so they can represent a missing value.
2. **Nullable reference types (NRT)**—annotations such as `string?` that enable compiler warnings about possible null assignments and dereferences. These annotations don't add runtime null checks or create new CLR reference types.

Both are useful, but neither removes the need to validate data from databases, JSON, other services, or older code.

| Syntax | Meaning | Runtime representation |
|---|---|---|
| `int?` | Nullable value type | An actual `Nullable<int>` value that can be empty or contain an `int`. |
| `string?` | Nullable reference annotation | The same `System.String` reference type as `string`; the compiler tracks possible null. |

## Nullable Value Types

A non-nullable value type such as `int`, `bool`, `DateTime`, `decimal`, an enum, or a struct always has a value. Real applications often need to express “not provided,” “not known,” or a database `NULL`. `Nullable<T>` provides that state.

### `Nullable<T>` and `T?`

`Nullable<int>` and `int?` are equivalent syntax. The `?` form is the usual style.

```csharp
using System;

Nullable<int> nullableInt = null;
Nullable<DateTime> nullableDate = null;
Nullable<decimal> nullablePrice = null;

int? age = null;
DateTime? birthDate = null;
decimal? discount = null;
bool? isActive = null;
```

A nullable value type is a `System.Nullable<T>` value. Its default value represents no value (`HasValue == false`).

### `HasValue`, `Value`, and Safe Access

- `HasValue` is `true` when a value is present.
- `Value` returns the underlying value, but throws `InvalidOperationException` if the nullable has no value.
- `GetValueOrDefault()` returns the underlying value or the underlying type's default.

```csharp
using System;

int? age = 25;
Console.WriteLine(age.HasValue); // True
Console.WriteLine(age.Value);    // 25

int? missingAge = null;
Console.WriteLine(missingAge.HasValue); // False
// Console.WriteLine(missingAge.Value); // Throws InvalidOperationException.

if (missingAge is int actualAge)
{
    Console.WriteLine($"Age: {actualAge}");
}
else
{
    Console.WriteLine("Age not provided.");
}

int displayAge = missingAge ?? 0;
int defaultAge = missingAge.GetValueOrDefault();
int customDefault = missingAge.GetValueOrDefault(18);
Console.WriteLine($"{displayAge}, {defaultAge}, {customDefault}"); // 0, 0, 18
```

Use `.Value` only after a check that establishes `HasValue`; `??`, a pattern, or `GetValueOrDefault` is often clearer.

### Lifted Arithmetic and Comparisons

Most arithmetic and comparison operators are **lifted** to nullable value types. Arithmetic results are null if an operand is null. For relational operators (`<`, `>`, `<=`, `>=`), the result is `false` if either operand is null. Equality has separate rules: two null nullable values compare equal; one null and one non-null value compare unequal.

```csharp
using System;

int? a = 10;
int? b = null;
int? c = 5;

int? sumWithValue = a + c; // 15
int? sumWithNull = a + b;  // null
int? product = a * b;      // null

Console.WriteLine(sumWithValue); // 15
Console.WriteLine(sumWithNull is null); // True
Console.WriteLine(product is null);     // True

Console.WriteLine(a > 5);      // True
Console.WriteLine(b > 5);      // False
Console.WriteLine(b < 5);      // False
Console.WriteLine(b == null);  // True
Console.WriteLine(b != null);  // False
Console.WriteLine(a == b);     // False
Console.WriteLine(b >= null);  // False: relational comparison, not equality.
```

Don't infer that the opposite relational comparison must be true when one comparison is false because of null. For example, a null value is neither greater than nor less than a number. Nullable `bool?` also has special `&` and `|` truth tables.

### Nullable Value Types and Database Columns

A nullable database column commonly maps to a nullable value type. With EF Core, nullable value types are optional by convention and non-nullable value types are required by convention, subject to model configuration and the provider.

```csharp
using System;

public sealed class User
{
    public int Id { get; set; }                  // Required column by convention.
    public int? Age { get; set; }                // Optional column by convention.
    public DateTime? DeactivatedAt { get; set; } // Optional column by convention.
}
```

Keep the C# model, EF Core configuration, and actual database schema consistent. If an existing database contains `NULL` for a value the model treats as required, materialization can fail; don't rely on a particular exception without checking the provider and query path.

## Nullable Reference Types (NRT)

Nullable reference types were introduced in C# 8. New .NET project templates have commonly enabled the nullable context since .NET 6 (C# 10), but existing projects may leave it disabled. Check the project file rather than assuming that targeting .NET 10 turns it on automatically.

```xml
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

In a nullable-aware context, `string` expresses the intent that a value isn't null, while `string?` expresses that it may be null. Both are still the same runtime type, `System.String`. NRT is compile-time analysis: it produces warnings, but doesn't insert runtime checks or stop reflection, deserialization, `null!`, or oblivious code from providing null.

### Non-Nullable and Nullable References

```csharp
#nullable enable

string name = "Alice";
string? middleName = null;

// Warnings in a nullable-aware context:
// string warning1 = null; // Null assigned to a non-nullable reference.
// name = null;            // Possible null assignment.

middleName = null; // Allowed: the type is explicitly nullable.
```

`Customer` and `Customer?` are also the same CLR reference type. The annotation changes compiler analysis and is recorded as metadata; it doesn't make the value non-null at runtime.

### Compiler Null-State Analysis

The compiler tracks whether a reference is known to be non-null or might be null at a particular point. A null check can make later access safe in that branch.

```csharp
#nullable enable
using System;

string? nullableName = DateTime.Now.Hour < 12 ? "Good morning" : null;

if (nullableName is not null)
{
    int length = nullableName.Length; // Safe in this branch.
    Console.WriteLine(length);
}

int? safeLength = nullableName?.Length;
string displayName = nullableName ?? "Anonymous";
int displayLength = displayName.Length; // displayName is non-null here.
```

Static analysis is not omniscient. The compiler generally can't infer the null guarantees of arbitrary methods or mutable properties unless the API's annotations communicate them.

### The Null-Forgiving Operator (`!`)

The postfix `!` operator suppresses a nullable warning for that expression. It has **no runtime effect** and does not check or change the value.

```csharp
#nullable enable
using System;

string? nullableName = Environment.GetEnvironmentVariable("USER_NAME");
int length = nullableName!.Length; // Suppresses a warning; still throws if null.
```

Prefer a guard when null is invalid:

```csharp
#nullable enable
using System;

string? nullableName = Environment.GetEnvironmentVariable("USER_NAME");
if (nullableName is null)
{
    throw new InvalidOperationException("A user name is required.");
}

int safeLength = nullableName.Length; // No `!` is needed after this check.
```

Use `!` only when an invariant is real but the compiler can't see it, and document why. If a custom helper establishes the invariant, a nullable-analysis attribute may express that contract more clearly.

### Nullable Annotations on APIs

Annotations on method parameters and return values tell callers what to expect. A non-nullable return type should either return a non-null value or fail explicitly; a nullable return type requires callers to consider absence.

```csharp
#nullable enable
using System;

public sealed class UserService
{
    private readonly IUserRepository _repository;

    public UserService(IUserRepository repository)
    {
        ArgumentNullException.ThrowIfNull(repository);
        _repository = repository;
    }

    // A missing customer is represented as an exception in this API contract.
    public Customer GetCustomerById(int id)
    {
        Customer? customer = _repository.Find(id);
        return customer ?? throw new InvalidOperationException($"Customer {id} was not found.");
    }

    // A missing customer is an expected result in this API contract.
    public Customer? FindCustomerByEmail(string email) => _repository.FindByEmail(email);

    public void UpdateCustomerName(int id, string newName)
    {
        ArgumentNullException.ThrowIfNull(newName);
        _repository.UpdateName(id, newName);
    }

    public void UpdateCustomerNickname(int id, string? nickname)
    {
        if (nickname is not null)
        {
            _repository.UpdateNickname(id, nickname);
        }
    }
}

public sealed class Customer
{
    public int Id { get; init; }
    public string Name { get; init; } = string.Empty;
}

public interface IUserRepository
{
    Customer? Find(int id);
    Customer? FindByEmail(string email);
    void UpdateName(int id, string name);
    void UpdateNickname(int id, string nickname);
}
```

`ArgumentNullException.ThrowIfNull` is available since .NET 6, so it works on .NET 10; it isn't a .NET 10-only feature. NRT warnings don't remove the need to guard public boundaries that can be called by unannotated code or receive external input.

### Nullable Analysis Attributes

Attributes in `System.Diagnostics.CodeAnalysis` describe null contracts that are more specific than a plain `?`. They inform the compiler; the method still needs to implement the promised runtime behavior.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;

static bool IsNotNull([NotNullWhen(true)] object? value) => value is not null;

static void EnsureNotNull([NotNull] object? value, string paramName)
{
    if (value is null)
    {
        throw new ArgumentNullException(paramName);
    }
}

[return: MaybeNull]
static T FindFirst<T>(IEnumerable<T> items, Func<T, bool> predicate)
{
    ArgumentNullException.ThrowIfNull(items);
    ArgumentNullException.ThrowIfNull(predicate);

    foreach (T item in items)
    {
        if (predicate(item))
        {
            return item;
        }
    }

    return default!; // The [MaybeNull] contract permits no match to return null/default.
}

string? input = "hello";
if (IsNotNull(input))
{
    Console.WriteLine(input.Length); // The compiler knows input isn't null here.
}

EnsureNotNull(input, nameof(input));
Console.WriteLine(input.Length); // The helper's [NotNull] contract guarantees this on return.

string? first = FindFirst(new[] { "", "first", "second" }, value => value.Length > 0);
```

`[NotNullWhen(true)]` describes a nullable argument that is non-null when the method returns `true`; `[NotNull]` describes a value that is non-null when the method returns; `[MaybeNull]` allows a generic result to be the default value even when its declared type parameter isn't annotated nullable.

## Null-Coalescing Operator (`??`)

`left ?? right` evaluates to `left` when `left` is non-null; otherwise it evaluates to `right`. The right side is evaluated only when needed. It works with nullable value types and nullable references.

```csharp
using System;

int? page = null;
int actualPage = page ?? 1;
Console.WriteLine(actualPage); // 1

page = 5;
actualPage = page ?? 1;
Console.WriteLine(actualPage); // 5

string? username = null;
string displayName = username ?? "Anonymous";
Console.WriteLine(displayName); // Anonymous
```

The operator associates from right to left, so `a ?? b ?? fallback` means `a ?? (b ?? fallback)`.

### Chaining and Throw Expressions

```csharp
using System;

string? primaryEmail = null;
string? secondaryEmail = null;
string? workEmail = "alice@company.com";

string contactEmail = primaryEmail ?? secondaryEmail ?? workEmail ?? "no-email@example.com";
Console.WriteLine(contactEmail); // alice@company.com
```

A `throw` expression can reject null at a method boundary:

```csharp
#nullable enable
using System;

static class OrderProcessor
{
    public static void ProcessOrder(Order? order)
    {
        Order validOrder = order ?? throw new ArgumentNullException(nameof(order));
        Console.WriteLine($"Processing order #{validOrder.Id}");
    }
}

sealed class Order
{
    public int Id { get; init; }
}
```

## Null-Coalescing Assignment (`??=`)

`left ??= right` assigns `right` to `left` only when `left` is null. Like `??`, it doesn't evaluate the right side if it isn't needed.

```csharp
#nullable enable
using System;
using System.Collections.Generic;

List<string>? tags = null;
tags ??= new List<string>();
tags.Add("backend");
tags.Add("csharp");

tags ??= new List<string>(); // No reset: tags is already non-null.
Console.WriteLine(tags.Count); // 2
```

### Common Uses

```csharp
#nullable enable
using System;
using System.Collections.Generic;

sealed class OrderHolder
{
    private List<int>? _orderIds;

    public List<int> OrderIds
    {
        get
        {
            _orderIds ??= new List<int>();
            return _orderIds;
        }
    }
}

sealed class AppSettings
{
    public string? ConnectionString { get; set; }
    public int? MaxRetries { get; set; }

    public void ApplyDefaults()
    {
        ConnectionString ??= "Server=localhost;Database=App;";
        MaxRetries ??= 3;
    }
}

sealed class SimpleCache
{
    private Dictionary<string, object>? _cache;

    public object GetOrCreate(string key, Func<object> factory)
    {
        ArgumentNullException.ThrowIfNull(key);
        ArgumentNullException.ThrowIfNull(factory);
        _cache ??= new Dictionary<string, object>();

        if (_cache.TryGetValue(key, out object? value))
        {
            return value ?? throw new InvalidOperationException("The cache contained a null value.");
        }

        value = factory() ?? throw new InvalidOperationException("The factory returned null.");
        _cache[key] = value;
        return value;
    }
}
```

`??=` is not a synchronization primitive. If multiple threads can access the same lazily initialized field, use appropriate synchronization or a thread-safe design.

## Null-Conditional Operators (`?.` and `?[]`)

The null-conditional operators skip member or indexer access when the receiver is null. The expression then evaluates to null (for a value-type result, a nullable value type). They protect against a **null receiver**, not other errors: for example, `array?[10]` still throws if the array is non-null but has no element at index 10.

### Member and Method Access

```csharp
using System;

string? name = null;
int? length = name?.Length;
Console.WriteLine(length.HasValue); // False

name = "Alice";
length = name?.Length;
Console.WriteLine(length); // 5

Action? callback = null;
callback?.Invoke(); // Does nothing when callback is null.

callback = () => Console.WriteLine("Callback executed!");
callback?.Invoke();
```

### Index Access and Chaining

```csharp
using System;

int[]? scores = null;
int? firstScore = scores?[0];
Console.WriteLine(firstScore.HasValue); // False

scores = new[] { 85, 92, 78 };
firstScore = scores?[0];
Console.WriteLine(firstScore); // 85
```

You can chain null-conditional access through nested objects. If a receiver in the chain is null, the rest of that conditional-access chain is skipped.

```csharp
#nullable enable
using System;

var order = new DemoOrder
{
    Customer = new DemoCustomer
    {
        Address = new DemoAddress { City = "Dhaka" }
    }
};

string? city = order?.Customer?.Address?.City;
Console.WriteLine(city); // Dhaka

order.Customer = null;
city = order?.Customer?.Address?.City;
Console.WriteLine(city ?? "Unknown"); // Unknown

sealed class DemoOrder { public DemoCustomer? Customer { get; set; } }
sealed class DemoCustomer { public int Id { get; set; } public DemoAddress? Address { get; set; } }
sealed class DemoAddress { public string? City { get; set; } }
```

Combine `?.` with `??` to provide a fallback:

```csharp
#nullable enable
DemoOrder? maybeOrder = null;
string cityName = maybeOrder?.Customer?.Address?.City ?? "Unknown";
int? customerId = maybeOrder?.Customer?.Id;
int displayId = customerId ?? -1;
```

## Null-Handling Patterns in Backend Code

### Guard Clauses at Method Entry

Validate required inputs before using them. `ArgumentNullException.ThrowIfNull` gives a concise guard and informs nullable analysis that execution continues only for a non-null argument.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

static async Task<Order> CreateOrderAsync(
    CreateOrderRequest? request,
    CancellationToken cancellationToken)
{
    ArgumentNullException.ThrowIfNull(request);

    string customerEmail = request.CustomerEmail
        ?? throw new ArgumentNullException(nameof(request.CustomerEmail));
    List<string> items = request.Items
        ?? throw new ArgumentNullException(nameof(request.Items));

    if (items.Count == 0)
    {
        throw new ArgumentException("Order must contain at least one item.", nameof(request));
    }

    await Task.CompletedTask; // Placeholder for actual asynchronous work.
    return new Order { CustomerEmail = customerEmail, ItemCount = items.Count };
}

sealed class CreateOrderRequest
{
    public string? CustomerEmail { get; init; }
    public List<string>? Items { get; init; }
}

sealed class Order
{
    public string CustomerEmail { get; init; } = string.Empty;
    public int ItemCount { get; init; }
}
```

A non-nullable annotation is not a substitute for validation of JSON, database, or other external data. Also validate business rules such as empty collections, formats, and numeric ranges.

### Null Object Pattern

When “no policy” has a meaningful neutral behavior, a null object can remove repetitive null checks. Don't use it to disguise a missing record or another error that callers should distinguish.

```csharp
#nullable enable
using System;

sealed class DiscountPolicy
{
    public static DiscountPolicy NoDiscount { get; } = new(0m, "No discount");

    public decimal Percentage { get; }
    public string Description { get; }

    public DiscountPolicy(decimal percentage, string description)
    {
        if (percentage < 0m || percentage > 100m)
        {
            throw new ArgumentOutOfRangeException(nameof(percentage));
        }

        ArgumentNullException.ThrowIfNull(description);
        Percentage = percentage;
        Description = description;
    }

    public decimal Apply(decimal amount) => amount * (1m - Percentage / 100m);
}

static class DiscountPolicySelector
{
    public static DiscountPolicy ResolvePolicy(DiscountPolicy? policy) => policy ?? DiscountPolicy.NoDiscount;
}
```

### Optional and Result Types

A nullable return is often right when “not found” is a normal outcome. If callers must distinguish “found,” “not found,” and “failed,” use an explicit result/option type instead of overloading null with several meanings. C# doesn't include a single built-in `Optional<T>` type; libraries and applications define types suited to their contracts.

```csharp
abstract record LookupResult<T>;
sealed record Found<T>(T Value) : LookupResult<T>;
sealed record NotFound<T>() : LookupResult<T>;
sealed record Failed<T>(string Reason) : LookupResult<T>;

static class LookupResultFormatter
{
    public static string Describe(LookupResult<string> result) => result switch
    {
        Found<string> found => $"Found: {found.Value}",
        NotFound<string> => "No matching item.",
        Failed<string> failed => $"Lookup failed: {failed.Reason}",
        _ => "Unknown result."
    };
}
```

The final discard arm provides a fallback if an unexpected derived result is introduced; omitting it allows the compiler to warn about missing cases when the known variants change.

## Common Null-Related Exceptions

| Exception | Typical cause | Prevention |
|---|---|---|
| `NullReferenceException` | Dereferencing a null reference or using a null value unexpectedly | Nullable annotations, checks, `?.`, and input validation |
| `InvalidOperationException` | Reading `.Value` from a nullable value type with no value | Check `HasValue`, use a pattern, `??`, or `GetValueOrDefault` |
| `ArgumentNullException` | A method receives null where its contract requires a value | Nullable API annotations and `ArgumentNullException.ThrowIfNull` |
| `NullReferenceException` in a LINQ query | A query lambda dereferences a null element | Model nullable elements (`T?`) and check or filter according to the query's semantics |

LINQ itself doesn't throw merely because a sequence contains a null element; the exception occurs when code dereferences that element or otherwise assumes it is non-null.

## Basic Code Snippet

This complete .NET 10 console program demonstrates nullable value types, nullable references, `??`, `??=`, `?.`, and a throw expression.

```csharp
#nullable enable
using System;
using System.Collections.Generic;

Console.WriteLine("=== Nullable value types ===");
int? age = null;
DateTime? birthday = null;
decimal? discount = 15.5m;

string birthdayText = birthday?.ToString("yyyy-MM-dd") ?? "Not set";
string discountText = discount?.ToString("F1") ?? "None";
Console.WriteLine($"Age has value: {age.HasValue}");
Console.WriteLine($"Birthday: {birthdayText}");
Console.WriteLine($"Discount: {discountText}%");

age = 25;
Console.WriteLine($"Age: {age.GetValueOrDefault()}");

Console.WriteLine("\n=== Lifted arithmetic ===");
int? x = 10;
int? y = null;
int? sum = x + y;
int? product = x * 5;
Console.WriteLine($"10 + null is null: {sum is null}"); // True
Console.WriteLine($"10 * 5 = {product}"); // 50
Console.WriteLine($"null > 5: {y > 5}"); // False
Console.WriteLine($"null == null: {y == null}"); // True

Console.WriteLine("\n=== Nullable references ===");
string nonNullable = "Hello";
string? nullable = null;
Console.WriteLine($"Non-null string length: {nonNullable.Length}");
Console.WriteLine($"Nullable length: {nullable?.Length?.ToString() ?? "null"}");

string display = nullable ?? "Default value";
Console.WriteLine($"Display: {display}");

Console.WriteLine("\n=== Null-coalescing assignment ===");
List<string>? items = null;
items ??= new List<string>();
items.Add("First");
items.Add("Second");
Console.WriteLine($"Items: {items.Count}"); // 2

Console.WriteLine("\n=== Chained null-conditional access ===");
var order = new DemoOrder
{
    Customer = new DemoCustomer
    {
        Id = 7,
        Address = new DemoAddress { City = "Dhaka" }
    }
};

string? city = order?.Customer?.Address?.City;
int? customerId = order?.Customer?.Id;
Console.WriteLine($"City: {city ?? "Unknown"}");
Console.WriteLine($"Customer id: {customerId?.ToString() ?? "Unknown"}");

order.Customer = null;
city = order?.Customer?.Address?.City;
Console.WriteLine($"City after removing customer: {city ?? "Unknown"}");

Console.WriteLine("\n=== Throw expression ===");
try
{
    ProcessInput(null);
}
catch (ArgumentNullException ex)
{
    Console.WriteLine($"Caught: {ex.ParamName} is null");
}

string appName = Environment.GetEnvironmentVariable("APP_NAME") ?? "MyApp";
Console.WriteLine($"App: {appName}");

static void ProcessInput(string? input)
{
    string valid = input ?? throw new ArgumentNullException(nameof(input));
    Console.WriteLine($"Processing: {valid}");
}

sealed class DemoOrder
{
    public DemoCustomer? Customer { get; set; }
}

sealed class DemoCustomer
{
    public int Id { get; set; }
    public DemoAddress? Address { get; set; }
}

sealed class DemoAddress
{
    public string? City { get; set; }
}
```

## Applied Customer Lookup Example (Illustrative)

This example shows nullable returns, optional filters, null-safe DTO construction, partial-update semantics, and a compiler-annotated helper. It is illustrative rather than a complete production service: database filtering should generally happen in the repository/query provider, pagination must be bounded, authorization and validation are application-specific, and all order amounts below are assumed to be normalized to one reporting currency before aggregation.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;
using System.Globalization;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace MyBackendApp.Core.Services;

public sealed class CustomerLookupService
{
    private readonly ICustomerRepository _customerRepository;
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger _logger;

    public CustomerLookupService(
        ICustomerRepository customerRepository,
        IOrderRepository orderRepository,
        ILogger logger)
    {
        ArgumentNullException.ThrowIfNull(customerRepository);
        ArgumentNullException.ThrowIfNull(orderRepository);
        ArgumentNullException.ThrowIfNull(logger);

        _customerRepository = customerRepository;
        _orderRepository = orderRepository;
        _logger = logger;
    }

    // A missing customer is an expected outcome: the caller can map null to HTTP 404.
    public async Task<CustomerProfileDto?> GetCustomerProfileAsync(
        int customerId,
        CancellationToken cancellationToken = default)
    {
        Customer? customer = await _customerRepository.GetByIdAsync(customerId, cancellationToken);
        if (customer is null)
        {
            _logger.LogWarning("Customer {CustomerId} not found.", customerId);
            return null;
        }

        IReadOnlyList<Order> orders = await _orderRepository
            .GetByCustomerIdAsync(customerId, cancellationToken);
        List<Order> activeOrders = orders
            .Where(order => order.Status != OrderStatus.Cancelled)
            .ToList();
        Order? latestOrder = activeOrders.MaxBy(order => order.CreatedAt);
        List<string> tags = customer.Tags is { } existingTags
            ? new List<string>(existingTags)
            : new List<string>();

        return new CustomerProfileDto
        {
            CustomerId = customer.Id,
            FullName = customer.FullName,
            Email = customer.Email,
            Phone = customer.PhoneNumber ?? "Not provided",
            DateOfBirth = customer.DateOfBirth?.ToString("yyyy-MM-dd", CultureInfo.InvariantCulture),
            MembershipTier = customer.Membership?.TierName ?? "Standard",
            MembershipExpiry = customer.Membership?.ExpiresAt?.ToString(
                "yyyy-MM-dd", CultureInfo.InvariantCulture),
            PreferredStore = customer.Preferences?.PreferredStore?.Name ?? "Online",
            TotalOrders = activeOrders.Count,
            // Assumes all values are in the same reporting currency.
            TotalSpentInReportingCurrency = activeOrders.Sum(order => order.TotalAmountInReportingCurrency),
            LastOrderDate = latestOrder?.CreatedAt.ToString("yyyy-MM-dd", CultureInfo.InvariantCulture),
            LoyaltyPoints = customer.LoyaltyAccount?.Points ?? 0,
            HasActiveSubscription = customer.Subscription?.IsActive ?? false,
            Tags = tags
        };
    }

    public async Task<SearchResultDto<CustomerSummaryDto>> SearchCustomersAsync(
        string? nameFilter = null,
        string? emailDomain = null,
        int? minimumOrderCount = null,
        decimal? minimumSpendInReportingCurrency = null,
        DateTimeOffset? registeredAfter = null,
        int page = 1,
        int pageSize = 20,
        CancellationToken cancellationToken = default)
    {
        page = Math.Max(page, 1);
        pageSize = pageSize is < 1 or > 100 ? 20 : pageSize;

        // Illustrative only: a production repository should apply these filters in the database.
        IReadOnlyList<Customer> allCustomers = await _customerRepository.GetAllAsync(cancellationToken);
        IEnumerable<Customer> filtered = allCustomers;

        if (!string.IsNullOrWhiteSpace(nameFilter))
        {
            string searchName = nameFilter.Trim();
            filtered = filtered.Where(customer =>
                customer.FullName.Contains(searchName, StringComparison.OrdinalIgnoreCase));
        }

        if (!string.IsNullOrWhiteSpace(emailDomain))
        {
            string normalizedDomain = emailDomain.Trim();
            filtered = filtered.Where(customer =>
                customer.Email.EndsWith($"@{normalizedDomain}", StringComparison.OrdinalIgnoreCase));
        }

        if (registeredAfter.HasValue)
        {
            DateTimeOffset cutoff = registeredAfter.Value;
            filtered = filtered.Where(customer => customer.CreatedAt >= cutoff);
        }

        if (minimumOrderCount.HasValue || minimumSpendInReportingCurrency.HasValue)
        {
            var customersPassingOrderFilters = new List<Customer>();
            foreach (Customer customer in filtered.ToList())
            {
                IReadOnlyList<Order> orders = await _orderRepository
                    .GetByCustomerIdAsync(customer.Id, cancellationToken);
                List<Order> activeOrders = orders
                    .Where(order => order.Status != OrderStatus.Cancelled)
                    .ToList();

                int orderCount = activeOrders.Count;
                decimal totalSpend = activeOrders.Sum(order => order.TotalAmountInReportingCurrency);
                bool passesOrderCount = !minimumOrderCount.HasValue
                    || orderCount >= minimumOrderCount.Value;
                bool passesSpend = !minimumSpendInReportingCurrency.HasValue
                    || totalSpend >= minimumSpendInReportingCurrency.Value;

                if (passesOrderCount && passesSpend)
                {
                    customersPassingOrderFilters.Add(customer);
                }
            }

            filtered = customersPassingOrderFilters;
        }

        int totalCount = filtered.Count();
        int skip = (int)Math.Min((long)(page - 1) * pageSize, int.MaxValue);
        List<Customer> paged = filtered.Skip(skip).Take(pageSize).ToList();

        return new SearchResultDto<CustomerSummaryDto>
        {
            Items = paged.Select(customer => new CustomerSummaryDto
            {
                CustomerId = customer.Id,
                FullName = customer.FullName,
                Email = customer.Email,
                MembershipTier = customer.Membership?.TierName ?? "Standard",
                Status = customer.Status.ToString()
            }).ToList(),
            TotalCount = totalCount,
            Page = page,
            PageSize = pageSize,
            TotalPages = (int)Math.Ceiling((double)totalCount / pageSize)
        };
    }

    public async Task<UpdateResult> UpdateCustomerProfileAsync(
        int customerId,
        UpdateProfileRequest request,
        CancellationToken cancellationToken = default)
    {
        ArgumentNullException.ThrowIfNull(request);

        Customer? customer = await _customerRepository.GetByIdAsync(customerId, cancellationToken);
        if (customer is null)
        {
            return UpdateResult.NotFound($"Customer {customerId} not found.");
        }

        // Validate and stage every change first. Don't partially mutate a tracked entity
        // before discovering that a later request field is invalid.
        int changesApplied = 0;
        string? newFullName = null;
        string? requestedName = request.FullName;
        if (requestedName is not null)
        {
            if (string.IsNullOrWhiteSpace(requestedName))
            {
                return UpdateResult.ValidationFailed("Full name cannot be empty.");
            }

            newFullName = requestedName.Trim();
            changesApplied++;
        }

        string? newEmail = null;
        string? requestedEmail = request.Email;
        if (requestedEmail is not null)
        {
            newEmail = NormalizeEmail(requestedEmail);
            if (newEmail.Length == 0)
            {
                return UpdateResult.ValidationFailed("A valid email is required.");
            }

            Customer? existing = await _customerRepository
                .GetByEmailAsync(newEmail, cancellationToken);
            if (existing is not null && existing.Id != customerId)
            {
                return UpdateResult.ValidationFailed("Email is already in use.");
            }

            changesApplied++;
        }

        string? requestedPhoneNumber = request.PhoneNumber;
        bool updatePhoneNumber = request.ClearPhoneNumber;
        string? newPhoneNumber = null;
        if (request.ClearPhoneNumber)
        {
            changesApplied++;
        }
        else if (requestedPhoneNumber is not null)
        {
            if (string.IsNullOrWhiteSpace(requestedPhoneNumber))
            {
                return UpdateResult.ValidationFailed(
                    "Use ClearPhoneNumber to remove the phone number.");
            }

            newPhoneNumber = requestedPhoneNumber.Trim();
            updatePhoneNumber = true;
            changesApplied++;
        }

        bool updateDateOfBirth = request.ClearDateOfBirth;
        DateOnly? newDateOfBirth = null;
        if (request.ClearDateOfBirth)
        {
            changesApplied++;
        }
        else if (request.DateOfBirth is DateOnly dateOfBirth)
        {
            DateOnly latestAllowedBirthDate = DateOnly.FromDateTime(DateTime.UtcNow).AddYears(-13);
            if (dateOfBirth > latestAllowedBirthDate)
            {
                return UpdateResult.ValidationFailed("Customer must be at least 13 years old.");
            }

            newDateOfBirth = dateOfBirth;
            updateDateOfBirth = true;
            changesApplied++;
        }

        if (changesApplied == 0)
        {
            return UpdateResult.NoChanges();
        }

        if (newFullName is not null)
        {
            customer.FullName = newFullName;
        }

        if (newEmail is not null)
        {
            customer.Email = newEmail;
        }

        if (updatePhoneNumber)
        {
            customer.PhoneNumber = newPhoneNumber;
        }

        if (updateDateOfBirth)
        {
            customer.DateOfBirth = newDateOfBirth;
        }

        customer.UpdatedAt = DateTimeOffset.UtcNow;
        await _customerRepository.UpdateAsync(customer, cancellationToken);
        _logger.LogInformation(
            "Customer {CustomerId} updated. {Changes} field(s) changed.",
            customerId, changesApplied);

        return UpdateResult.Success(changesApplied);
    }

    private static string NormalizeEmail(string email)
    {
        string normalized = email.Trim().ToLowerInvariant();
        return normalized.Contains('@') ? normalized : string.Empty;
    }

    private static bool TryGetActiveMembership(
        Customer customer,
        [NotNullWhen(true)] out Membership? membership)
    {
        ArgumentNullException.ThrowIfNull(customer);
        membership = customer.Membership;

        if (membership is null)
        {
            return false;
        }

        if (membership.ExpiresAt is DateTimeOffset expiry && expiry < DateTimeOffset.UtcNow)
        {
            membership = null;
            return false;
        }

        return true;
    }

    public string GetMembershipSummary(Customer customer)
    {
        ArgumentNullException.ThrowIfNull(customer);

        if (TryGetActiveMembership(customer, out Membership? activeMembership))
        {
            return activeMembership.ExpiresAt is DateTimeOffset expiry
                ? $"{activeMembership.TierName} (expires {expiry.ToString("yyyy-MM-dd", CultureInfo.InvariantCulture)})"
                : $"{activeMembership.TierName} (no expiry)";
        }

        return "No active membership.";
    }
}

public sealed class CustomerProfileDto
{
    public int CustomerId { get; init; }
    public string FullName { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;
    public string Phone { get; init; } = string.Empty;
    public string? DateOfBirth { get; init; }
    public string MembershipTier { get; init; } = string.Empty;
    public string? MembershipExpiry { get; init; }
    public string PreferredStore { get; init; } = string.Empty;
    public int TotalOrders { get; init; }
    public decimal TotalSpentInReportingCurrency { get; init; }
    public string? LastOrderDate { get; init; }
    public int LoyaltyPoints { get; init; }
    public bool HasActiveSubscription { get; init; }
    public List<string> Tags { get; init; } = new();
}

public sealed class CustomerSummaryDto
{
    public int CustomerId { get; init; }
    public string FullName { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;
    public string MembershipTier { get; init; } = string.Empty;
    public string Status { get; init; } = string.Empty;
}

public sealed class SearchResultDto<T>
{
    public List<T> Items { get; init; } = new();
    public int TotalCount { get; init; }
    public int Page { get; init; }
    public int PageSize { get; init; }
    public int TotalPages { get; init; }
}

public sealed class UpdateProfileRequest
{
    // In this contract, null means "leave unchanged"; explicit clear flags distinguish clearing from omission.
    public string? FullName { get; init; }
    public string? Email { get; init; }
    public string? PhoneNumber { get; init; }
    public bool ClearPhoneNumber { get; init; }
    public DateOnly? DateOfBirth { get; init; }
    public bool ClearDateOfBirth { get; init; }
}

public sealed class UpdateResult
{
    public bool IsSuccess { get; private init; }
    public string Message { get; private init; } = string.Empty;
    public int ChangesApplied { get; private init; }

    public static UpdateResult Success(int changes) => new()
    {
        IsSuccess = true,
        Message = $"{changes} field(s) updated.",
        ChangesApplied = changes
    };

    public static UpdateResult NotFound(string message) => new()
    {
        Message = message
    };

    public static UpdateResult ValidationFailed(string message) => new()
    {
        Message = message
    };

    public static UpdateResult NoChanges() => new()
    {
        IsSuccess = true,
        Message = "No changes requested."
    };
}

public sealed class Customer
{
    public int Id { get; set; }
    public string FullName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string? PhoneNumber { get; set; }
    public DateOnly? DateOfBirth { get; set; }
    public CustomerStatus Status { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
    public DateTimeOffset UpdatedAt { get; set; }
    public Membership? Membership { get; set; }
    public CustomerPreferences? Preferences { get; set; }
    public LoyaltyAccount? LoyaltyAccount { get; set; }
    public Subscription? Subscription { get; set; }
    public List<string>? Tags { get; set; }
}

public sealed class Membership
{
    public string TierName { get; set; } = string.Empty;
    public DateTimeOffset? ExpiresAt { get; set; }
}

public sealed class CustomerPreferences
{
    public Store? PreferredStore { get; set; }
    public bool EmailNotifications { get; set; }
}

public sealed class Store
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
}

public sealed class LoyaltyAccount
{
    public int Points { get; set; }
    public DateTimeOffset MemberSince { get; set; }
}

public sealed class Subscription
{
    public bool IsActive { get; set; }
    public string Plan { get; set; } = string.Empty;
    public DateTimeOffset? NextBillingDate { get; set; }
}

public sealed class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public OrderStatus Status { get; set; }
    public decimal TotalAmountInReportingCurrency { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
}

public enum CustomerStatus
{
    Unknown = 0,
    Active = 10,
    Suspended = 20,
    Deactivated = 30
}

public enum OrderStatus
{
    Pending = 0,
    Confirmed = 1,
    Shipped = 2,
    Delivered = 3,
    Cancelled = 4
}

public interface ICustomerRepository
{
    Task<Customer?> GetByIdAsync(int id, CancellationToken cancellationToken);
    Task<Customer?> GetByEmailAsync(string email, CancellationToken cancellationToken);
    Task<IReadOnlyList<Customer>> GetAllAsync(CancellationToken cancellationToken);
    Task UpdateAsync(Customer customer, CancellationToken cancellationToken);
}

public interface IOrderRepository
{
    Task<IReadOnlyList<Order>> GetByCustomerIdAsync(int customerId, CancellationToken cancellationToken);
}

public interface ILogger
{
    void LogInformation(string message, params object?[] args);
    void LogWarning(string message, params object?[] args);
}
```

### Key Observations

- `GetCustomerProfileAsync` returns `CustomerProfileDto?` because “not found” is an expected outcome; an API controller can map it to HTTP 404. Whether absence should be `null`, an exception, or a result type is an API design choice.
- `?.` and `??` make optional nested fields explicit. `Membership?.ExpiresAt?.ToString(...)` handles both an absent membership and an absent expiry date. `MaxBy` returns null for an empty sequence; `CreatedAt` itself is non-nullable, so only the `MaxBy` result needs null-conditional access.
- Optional search parameters use `string?`, `int?`, `decimal?`, and `DateTimeOffset?`. Null means “don't apply this filter.” In a real backend, filtering and pagination should generally be translated into the database query instead of loading all rows.
- Order totals are explicitly named as reporting-currency amounts. Adding amounts in different currencies without conversion or grouping would be incorrect; conversion and rounding rules belong to the business contract.
- Nullable PATCH fields alone can't distinguish “field omitted” from “set this field to null.” The sample uses explicit clear flags. Other APIs can use a presence-aware wrapper, JSON Patch, or inspect field presence during deserialization.
- `ArgumentNullException.ThrowIfNull` guards the request object, while business validations check non-null values for empty names, email shape, item counts, and age. NRT annotations don't replace runtime validation.
- `[NotNullWhen(true)]` tells compiler analysis that the `out` membership is non-null when `TryGetActiveMembership` returns `true`; the implementation enforces that promise.
- A nullable entity `Tags` collection is projected to a non-nullable DTO list. Returning an empty list rather than null is an API convention, not a universal rule; select the shape clients expect.
- `UpdateResult` factory methods always return a non-null result. The `IsSuccess` flag and message represent outcomes without making the result reference nullable.
- NRT annotations affect EF Core conventions when enabled: `string` is generally required by convention and `string?` optional. Review migrations when enabling NRT in an existing model because column nullability may change.

## Key Terms Summary

| Term | Definition |
|---|---|
| Nullable value type | A `Nullable<T>` wrapper, written `T?`, that can represent either a value or null. |
| Nullable reference type | A compile-time nullability annotation such as `string?`; it doesn't create a different runtime type. |
| `HasValue` | `Nullable<T>` property indicating whether an underlying value is present. |
| `Value` | The underlying value of `Nullable<T>`; throws if `HasValue` is false. |
| Lifted operator | An operator applied to a nullable value type; arithmetic results can propagate null. |
| Null-coalescing (`??`) | Returns the left value when non-null; otherwise evaluates and returns the right value. |
| Null-coalescing assignment (`??=`) | Assigns the right value to the left only when the left is null. |
| Null-conditional (`?.`, `?[]`) | Skips member or index access when the receiver is null. |
| Null-forgiving (`!`) | Suppresses a compiler null warning for an expression; has no runtime effect. |
| Null-state analysis | Compiler analysis tracking whether an expression is known null or non-null at a point in code. |
| `[NotNullWhen]` | Attribute describing a conditional non-null postcondition for an argument. |
| `ArgumentNullException.ThrowIfNull` | Runtime guard method, available since .NET 6, that throws when its argument is null. |
| Partial update | An API contract where request fields indicate which values change; missing and explicit null may need separate representation. |

## Further Reading

- [Nullable value types — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types) — `Nullable<T>`, lifted operators, and comparison rules.
- [Nullable reference types — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references) — annotations, compiler analysis, and migration guidance.
- [Nullable reference types — language reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-reference-types) — null-forgiving operator and null-state rules.
- [Nullable analysis attributes — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/attributes/nullable-analysis) — `NotNullWhen`, `MaybeNull`, and related attributes.
- [`??` and `??=` operators — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator) — null-coalescing and null-coalescing assignment.
- [Member access operators — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators) — null-conditional member and index access.
- [`ArgumentNullException.ThrowIfNull` API](https://learn.microsoft.com/en-us/dotnet/api/system.argumentnullexception.throwifnull?view=net-10.0) — runtime null guard available on .NET 10.
- [Entity properties — EF Core](https://learn.microsoft.com/en-us/ef/core/modeling/entity-properties) — required and optional properties and database column nullability.
