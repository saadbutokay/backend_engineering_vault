## What Are Conditional Statements?

Conditional statements let a program choose which code to execute based on a Boolean condition. Backend code uses them for decisions such as validating input, applying business rules, and determining whether a request may proceed.

C# provides several conditional constructs, including `if`, the `switch` statement, and the value-producing `switch` expression. Pattern matching extends these constructs so they can test types, ranges, properties, and combinations of values.

The examples target the C# compiler used with .NET 10. C# language features are determined by the selected language version, not by the runtime alone.

## The `if` Statement

An `if` statement executes its body when its condition evaluates to `true`. Its condition must be a Boolean expression (or implicitly convertible to `bool`); values such as `1` or a non-empty string are not implicitly treated as true.

### Basic `if`

```csharp
int temperature = 35;

if (temperature > 30)
{
    Console.WriteLine("It is hot outside.");
}
```

Output:

```text
It is hot outside.
```

### `if`–`else`

The `else` block runs when the `if` condition is false.

```csharp
int age = 16;

if (age >= 18)
{
    Console.WriteLine("You are an adult.");
}
else
{
    Console.WriteLine("You are a minor.");
}
```

Output:

```text
You are a minor.
```

### `if`–`else if`–`else` Chain

Use an `else if` chain when you need to check several alternatives. Conditions are evaluated in order. The first true condition runs its block, and later conditions are skipped.

```csharp
int score = 78;

if (score >= 90)
{
    Console.WriteLine("Grade: A");
}
else if (score >= 80)
{
    Console.WriteLine("Grade: B");
}
else if (score >= 70)
{
    Console.WriteLine("Grade: C");
}
else if (score >= 60)
{
    Console.WriteLine("Grade: D");
}
else
{
    Console.WriteLine("Grade: F");
}
```

Output:

```text
Grade: C
```

### Use Braces for Clarity

C# permits omitting braces when a control statement has a single embedded statement. That can make later edits misleading:

```csharp
bool isActive = false;

// Legal, but easier to misread or accidentally change later.
if (isActive)
    Console.WriteLine("Process order.");

// Only the first statement belongs to the if. This message always prints.
if (isActive)
    Console.WriteLine("Process order.");
    Console.WriteLine("Send notification.");
```

Use braces even for one-line bodies:

```csharp
if (isActive)
{
    Console.WriteLine("Process order.");
    Console.WriteLine("Send notification.");
}
```

Braces are a strong readability convention, not a guarantee that every .NET project enforces automatically. Code-style analyzers and editor settings can be configured to require them.

## Nested Conditions and Guard Clauses

An `if` statement can contain another `if`, but deep nesting can make code harder to follow. There is no universal maximum nesting depth; simplify the structure when doing so improves clarity.

```csharp
bool isAuthenticated = true;
bool isAdmin = false;
string resource = "user-settings";

if (isAuthenticated)
{
    if (isAdmin)
    {
        Console.WriteLine("Full access granted.");
    }
    else
    {
        if (resource == "user-settings")
        {
            Console.WriteLine("Limited access granted.");
        }
        else
        {
            Console.WriteLine("Access denied.");
        }
    }
}
else
{
    Console.WriteLine("Please log in.");
}
```

A **guard clause** returns early when a precondition or exceptional case applies. It is a common way to reduce nesting and keep the main logic easy to scan; it is a technique, not a rule that fits every method.

```csharp
public sealed class NestedAccessPolicy
{
    public string GetAccessLevel(bool isAuthenticated, bool isAdmin, string resource)
    {
        if (isAuthenticated)
        {
            if (isAdmin)
            {
                return "Full";
            }

            if (resource == "user-settings")
            {
                return "Limited";
            }

            return "Denied";
        }

        return "Unauthenticated";
    }
}

public sealed class GuardClauseAccessPolicy
{
    public string GetAccessLevel(bool isAuthenticated, bool isAdmin, string resource)
    {
        if (!isAuthenticated)
        {
            return "Unauthenticated";
        }

        if (isAdmin)
        {
            return "Full";
        }

        if (resource == "user-settings")
        {
            return "Limited";
        }

        return "Denied";
    }
}
```

The guard-clause version handles early exits first, leaving the remaining decisions at the method's main level.

## The `switch` Statement

A `switch` statement selects a statement section based on the value or pattern matched by its input. It is often clearer than a long `if`–`else if` chain when several alternatives concern the same value.

```csharp
string dayOfWeek = "Monday";

switch (dayOfWeek)
{
    case "Monday":
    case "Tuesday":
    case "Wednesday":
    case "Thursday":
    case "Friday":
        Console.WriteLine("Weekday");
        break;

    case "Saturday":
    case "Sunday":
        Console.WriteLine("Weekend");
        break;

    default:
        Console.WriteLine("Invalid day");
        break;
}
```

Output:

```text
Weekday
```

### Key Rules for a `switch` Statement

- In the classic constant-label form, a `case` value must be a compile-time constant, such as a literal or an enum member. Modern C# also allows patterns—including type and relational patterns—and `when` guards in a `switch` statement.
- A non-empty switch section cannot complete normally and fall through into the next section. Common ways to leave it are `break`, `return`, or `throw`; `goto case` and `goto default` are also available for explicit jumps. A `continue` can transfer control to an enclosing loop.
- Multiple labels can share one section, as the weekday cases do above. This is not implicit fall-through from a section containing statements.
- `default` is optional. If no section matches and there is no `default`, the switch statement completes without executing a section.

### `switch` with an Enum

Enums are often used for a fixed set of named states. The enum declaration appears after the top-level statements in this example, as required for a top-level `Program.cs` file.

```csharp
using System;

OrderStatus status = OrderStatus.Shipped;

switch (status)
{
    case OrderStatus.Pending:
        Console.WriteLine("Order is awaiting payment.");
        break;
    case OrderStatus.Processing:
        Console.WriteLine("Order is being prepared.");
        break;
    case OrderStatus.Shipped:
        Console.WriteLine("Order is in transit.");
        break;
    case OrderStatus.Delivered:
        Console.WriteLine("Order has been delivered.");
        break;
    case OrderStatus.Cancelled:
        Console.WriteLine("Order was cancelled.");
        break;
    default:
        Console.WriteLine("Unknown order status.");
        break;
}

enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}
```

## The `switch` Expression (C# 8 and Later)

A `switch` expression matches an input against a series of patterns and **produces a value**. It is useful in assignments, return expressions, and other places where a value is needed.

```csharp
string dayOfWeek = "Monday";

string dayType = dayOfWeek switch
{
    "Monday" or "Tuesday" or "Wednesday" or "Thursday" or "Friday" => "Weekday",
    "Saturday" or "Sunday" => "Weekend",
    _ => "Invalid day"
};

Console.WriteLine(dayType); // Weekday
```

The `or` pattern is available in C# 9 and later. Key differences from a `switch` statement:

- The input expression comes before the `switch` keyword.
- Each arm uses `=>` and yields an expression value; there are no `break` statements.
- The discard pattern `_` can act as a catch-all arm, similar in purpose to `default`.
- Arms are considered in order, so a more specific arm must not be hidden by an earlier broader arm.

### `switch` Expression with an Enum

```csharp
using System;

OrderStatus status = OrderStatus.Shipped;

string message = status switch
{
    OrderStatus.Pending => "Order is awaiting payment.",
    OrderStatus.Processing => "Order is being prepared.",
    OrderStatus.Shipped => "Order is in transit.",
    OrderStatus.Delivered => "Order has been delivered.",
    OrderStatus.Cancelled => "Order was cancelled.",
    _ => "Unknown order status."
};

Console.WriteLine(message);

enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}
```

### `switch` Expression in a Return Value

```csharp
public sealed class PricingService
{
    public decimal GetDiscountRate(string customerTier)
    {
        return customerTier switch
        {
            "Platinum" => 0.20m,
            "Gold" => 0.15m,
            "Silver" => 0.10m,
            "Bronze" => 0.05m,
            _ => 0.00m
        };
    }
}
```

### Exhaustiveness and Enum Values

A `switch` expression must produce a value for every input that can reach it. The compiler can warn when an expression does not handle a declared enum member. Enum values can also contain unnamed underlying numeric values, so a switch that lists every named member but has no catch-all may still produce an exhaustiveness warning for unnamed values.

This intentionally incomplete example demonstrates the missing-member warning: `Refunded` is not handled, and there is no catch-all arm.

```csharp
using System;

OrderStatus status = OrderStatus.Refunded;

string label = status switch
{
    OrderStatus.Pending => "Pending",
    OrderStatus.Processing => "Processing",
    OrderStatus.Shipped => "Shipped",
    OrderStatus.Delivered => "Delivered",
    OrderStatus.Cancelled => "Cancelled"
    // Intentionally missing OrderStatus.Refunded and a catch-all arm.
};

Console.WriteLine(label);

enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled,
    Refunded
}
```

Add a catch-all arm when unknown values should be handled at runtime:

```csharp
_ => "Unknown status"
```

A catch-all prevents an unhandled value from reaching the switch expression, but it also means adding a new enum member will not be reported as a missing arm. Choose deliberately between compiler help for newly declared members and an explicit fallback for unexpected runtime values.

## Pattern Matching Basics

Pattern matching checks more than equality. It can test a value's runtime type, compare it with a range, inspect properties, or match a tuple or sequence. Patterns can be used in `is` expressions, `switch` statements, and `switch` expressions.

### Type Patterns

A declaration pattern checks a value's runtime type and, if the match succeeds, assigns the value to a variable of that type.

```csharp
using System;

object data = "Hello, World!";

// Traditional two-step approach.
if (data is string)
{
    string text = (string)data;
    Console.WriteLine($"String length: {text.Length}");
}

// Declaration pattern: test the type and capture the value in one step.
if (data is string textValue)
{
    Console.WriteLine($"String length: {textValue.Length}");
}

string description = data switch
{
    string s => $"String with {s.Length} characters",
    int i => $"Integer: {i}",
    double d => $"Double: {d}",
    null => "Null value",
    _ => $"Unknown type: {data.GetType().Name}"
};

Console.WriteLine(description); // String with 13 characters
```

The `null` arm comes before the catch-all expression that calls `GetType`, so a null input does not cause a `NullReferenceException` there.

### Relational Patterns

Relational patterns use `<`, `>`, `<=`, and `>=` to test a value against a constant. Arms are considered from top to bottom; in this ordered example, each later arm handles values not matched earlier.

```csharp
int temperature = 35;

string weather = temperature switch
{
    < 0 => "Freezing",
    < 15 => "Cold",
    < 25 => "Mild",
    < 35 => "Warm",
    _ => "Hot"
};

Console.WriteLine(weather); // Hot
```

### Logical Patterns: `and`, `or`, and `not`

Logical patterns combine simpler patterns. They were introduced in C# 9.

```csharp
int age = 25;
bool hasLicense = true;

string drivingStatus = (age, hasLicense) switch
{
    (< 16, _) => "Too young to drive",
    (>= 16, false) => "Eligible but no license",
    (>= 16 and < 80, true) => "Licensed driver",
    (>= 80, true) => "Licensed senior driver",
    _ => "Unknown status"
};

Console.WriteLine(drivingStatus); // Licensed driver
```

The `_` in a tuple pattern matches any value in that position. In a production domain rule, validate inputs such as negative ages separately if they should be rejected rather than classified as “too young.”

### Property Patterns

Property patterns inspect one or more properties of an object. They are useful when a decision depends on an object's state.

```csharp
using System;

var order = new Order
{
    Total = 250m,
    Status = "Processing",
    IsPriority = true
};

string action = order switch
{
    { Status: "Cancelled" } => "Do nothing",
    { Status: "Delivered", Total: > 1000m } => "Send premium thank-you email",
    { Status: "Delivered" } => "Send standard thank-you email",
    { Status: "Processing", IsPriority: true } => "Expedite shipping",
    { Status: "Processing" } => "Process normally",
    { Total: > 500m } => "Flag for manual review",
    _ => "Standard handling"
};

Console.WriteLine(action); // Expedite shipping

// Type declarations follow the top-level statements in this file.
class Order
{
    public decimal Total { get; set; }
    public string Status { get; set; } = string.Empty;
    public bool IsPriority { get; set; }
}
```

### Positional Patterns with Tuples

A positional pattern matches the elements of a tuple (or another type that supports deconstruction).

```csharp
(int x, int y) point = (3, 7);

string quadrant = point switch
{
    (0, 0) => "Origin",
    (> 0, > 0) => "Quadrant I",
    (< 0, > 0) => "Quadrant II",
    (< 0, < 0) => "Quadrant III",
    (> 0, < 0) => "Quadrant IV",
    _ => "On an axis"
};

Console.WriteLine(quadrant); // Quadrant I
```

The final arm handles points on an axis that did not match one of the quadrant patterns.

### List Patterns (C# 11 and Later)

List patterns test the length and elements of arrays and other supported sequence-like types. The C# compiler included with .NET 10 supports them.

```csharp
int[] numbers = { 1, 2, 3 };

string description = numbers switch
{
    [] => "Empty array",
    [var single] => $"Single element: {single}",
    [1, 2, 3] => "Sequential 1-2-3",
    [1, .., 5] => "Starts with 1, ends with 5",
    [_, _, _] => "Three elements",
    _ => "Other"
};

Console.WriteLine(description); // Sequential 1-2-3
```

The more specific `[1, .., 5]` arm comes before the general three-element arm so that a three-element sequence beginning with `1` and ending with `5` gets the more specific result.

## When to Use Each Construct

| Scenario | Useful construct |
|---|---|
| One Boolean condition | `if` |
| Two mutually exclusive outcomes | `if`–`else` or conditional operator `?:` |
| Conditions based on different variables or steps | `if`–`else if` chain |
| One value matched against several alternatives | `switch` statement or `switch` expression |
| Runtime type check | `is` with a declaration/type pattern, or a switch with type patterns |
| Decision based on object properties | `switch` with property patterns |
| Range checks on one value | Relational patterns in a switch |
| Combination of values and conditions | Tuple patterns and `when` guards |
| Simple value selection | Conditional operator `?:` or `switch` expression |

## Basic Code Snippet

This is a complete .NET 10 console-program example. It uses top-level statements; the enum type declaration is placed after those statements.

```csharp
using System;

// --- if-else chain ---
int score = 82;

string grade;
if (score >= 90)
{
    grade = "A";
}
else if (score >= 80)
{
    grade = "B";
}
else if (score >= 70)
{
    grade = "C";
}
else if (score >= 60)
{
    grade = "D";
}
else
{
    grade = "F";
}

Console.WriteLine($"Score: {score}, Grade: {grade}");

// --- switch expression with an enum ---
UserRole role = UserRole.Moderator;

string permissions = role switch
{
    UserRole.Guest => "Read only",
    UserRole.Member => "Read and write",
    UserRole.Moderator => "Read, write, and moderate",
    UserRole.Admin => "Full access",
    _ => "No access"
};

Console.WriteLine($"Role: {role}, Permissions: {permissions}");

// --- relational and logical patterns ---
int age = 22;
bool hasId = true;

string entryStatus = (age, hasId) switch
{
    (< 18, _) => "Denied: underage",
    (>= 18, false) => "Denied: no ID",
    (>= 18 and <= 21, true) => "Admitted with wristband",
    (> 21, true) => "Full admission",
    _ => "Unknown"
};

Console.WriteLine($"Entry: {entryStatus}");

// --- property pattern ---
var order = new { Total = 150m, Status = "Shipped", IsExpress = false };

string notification = order switch
{
    { Status: "Cancelled" } => "Refund initiated",
    { Status: "Shipped", IsExpress: true } => "Express delivery tomorrow",
    { Status: "Shipped" } => "Standard delivery in 3-5 days",
    { Total: > 500m } => "High-value order alert",
    _ => "Order received"
};

Console.WriteLine($"Notification: {notification}");

// --- guard-clause example as a local function ---
static string ProcessOrder(decimal amount, bool isPaid, string region)
{
    if (amount <= 0)
    {
        return "Invalid amount";
    }

    if (!isPaid)
    {
        return "Payment required";
    }

    if (region == "Restricted")
    {
        return "Region not supported";
    }

    return "Order processed successfully";
}

Console.WriteLine(ProcessOrder(100m, true, "US"));
Console.WriteLine(ProcessOrder(0m, true, "US"));
Console.WriteLine(ProcessOrder(100m, false, "US"));

enum UserRole
{
    Guest,
    Member,
    Moderator,
    Admin
}
```

## Backend Authorization Example (Illustrative)

This example demonstrates guard clauses, tuple patterns, and switch expressions in a backend-style service. It is **illustrative decision logic, not production-ready authorization**. Real ASP.NET Core applications should use the authentication and authorization systems, policies/handlers, endpoint metadata, and resource checks appropriate to the application. Do not treat raw path-string checks alone as an authorization boundary.

```csharp
using System;
using System.Collections.Generic;

namespace MyBackendApp.Core.Services;

public sealed class AuthorizationService
{
    private readonly HashSet<string> _restrictedEndpoints = new(StringComparer.OrdinalIgnoreCase)
    {
        "/admin/users",
        "/admin/settings",
        "/admin/audit-log",
        "/finance/reports",
        "/finance/refunds"
    };

    public AuthorizationResult EvaluateAccess(AuthorizationContext context)
    {
        ArgumentNullException.ThrowIfNull(context);

        if (string.IsNullOrWhiteSpace(context.HttpMethod) ||
            string.IsNullOrWhiteSpace(context.EndpointPath))
        {
            return AuthorizationResult.Deny(
                reason: "Request method or endpoint path is missing.",
                statusCode: 400);
        }

        // Guard clause: reject unauthenticated requests immediately.
        if (!context.IsAuthenticated)
        {
            return AuthorizationResult.Deny(
                reason: "Authentication required.",
                statusCode: 401);
        }

        // Guard clause: reject suspended or deactivated accounts.
        if (context.UserStatus is UserStatus.Suspended or UserStatus.Deactivated)
        {
            return AuthorizationResult.Deny(
                reason: "Account is suspended or deactivated.",
                statusCode: 403);
        }

        // Fail closed for pending or otherwise unexpected account states.
        if (context.UserStatus is not UserStatus.Active)
        {
            return AuthorizationResult.Deny(
                reason: "Account status does not permit access.",
                statusCode: 403);
        }

        // Assumes EndpointPath is a normalized, query-free path from the routing layer.
        string method = context.HttpMethod.ToUpperInvariant();
        string endpoint = context.EndpointPath;
        bool isWriteOperation = method is "POST" or "PUT" or "PATCH" or "DELETE";
        bool isRestricted = IsRestrictedEndpoint(endpoint);

        AccessLevel accessLevel = (context.Role, isWriteOperation, isRestricted) switch
        {
            // Admins have full access in this simplified example.
            (UserRole.Admin, _, _) => AccessLevel.Full,

            // Managers have read/write access on non-restricted paths,
            // and read-only access on the restricted paths listed above.
            (UserRole.Manager, _, false) => AccessLevel.ReadWrite,
            (UserRole.Manager, _, true) => AccessLevel.ReadOnly,

            // Editors can write to content paths; they can read other non-restricted paths.
            (UserRole.Editor, true, false) when IsPathOrChild(endpoint, "/content")
                => AccessLevel.ReadWrite,
            (UserRole.Editor, false, false) => AccessLevel.ReadOnly,
            (UserRole.Editor, true, false) => AccessLevel.ReadOnly,

            // Viewers are read-only.
            (UserRole.Viewer, _, _) => AccessLevel.ReadOnly,

            // Guests can read public paths only.
            (UserRole.Guest, false, false) when IsPathOrChild(endpoint, "/public")
                => AccessLevel.ReadOnly,
            (UserRole.Guest, _, _) => AccessLevel.None,

            _ => AccessLevel.None
        };

        return accessLevel switch
        {
            AccessLevel.Full => AuthorizationResult.Allow(),
            AccessLevel.ReadWrite => AuthorizationResult.Allow(),

            AccessLevel.ReadOnly when !isWriteOperation => AuthorizationResult.Allow(),
            AccessLevel.ReadOnly => AuthorizationResult.Deny(
                reason: "Read-only access. Write operations are not permitted.",
                statusCode: 403),

            AccessLevel.None => AuthorizationResult.Deny(
                reason: $"Role '{context.Role}' does not have access to {method} {endpoint}.",
                statusCode: 403),

            _ => AuthorizationResult.Deny(
                reason: "Unexpected authorization state.",
                statusCode: 500)
        };
    }

    public string GetRateLimitTier(AuthorizationContext context)
    {
        ArgumentNullException.ThrowIfNull(context);

        return context switch
        {
            { Role: UserRole.Admin } => "Unlimited",
            { Role: UserRole.Manager, SubscriptionTier: "Enterprise" } => "10000/hour",
            { Role: UserRole.Manager } => "5000/hour",
            { SubscriptionTier: "Enterprise" } => "5000/hour",
            { SubscriptionTier: "Professional" } => "2000/hour",
            { SubscriptionTier: "Starter" } => "500/hour",
            { IsAuthenticated: true } => "100/hour",
            _ => "20/hour"
        };
    }

    public string GetCachePolicy(string httpMethod, string endpoint, int? maxAgeOverride)
    {
        // Use a private cache policy by default for overrides. A public policy
        // must be selected only when the resource is known to be safe for shared caching.
        if (string.Equals(httpMethod, "GET", StringComparison.OrdinalIgnoreCase) &&
            maxAgeOverride.HasValue && maxAgeOverride.Value >= 0)
        {
            return $"private, max-age={maxAgeOverride.Value}";
        }

        return (httpMethod.ToUpperInvariant(), endpoint) switch
        {
            ("GET", var path) when PathEquals(path, "/public/products")
                => "public, max-age=3600",
            ("GET", var path) when PathEquals(path, "/public/categories")
                => "public, max-age=86400",
            ("GET", var path) when IsPathOrChild(path, "/api/users")
                => "private, max-age=60",
            ("GET", _) => "private, max-age=300",
            ("POST" or "PUT" or "PATCH" or "DELETE", _) => "no-cache, no-store",
            _ => "no-cache"
        };
    }

    private bool IsRestrictedEndpoint(string path)
    {
        foreach (string restrictedPath in _restrictedEndpoints)
        {
            if (IsPathOrChild(path, restrictedPath))
            {
                return true;
            }
        }

        return false;
    }

    private static bool PathEquals(string path, string expected) =>
        string.Equals(path, expected, StringComparison.OrdinalIgnoreCase);

    private static bool IsPathOrChild(string path, string prefix) =>
        string.Equals(path, prefix, StringComparison.OrdinalIgnoreCase) ||
        path.StartsWith(prefix + "/", StringComparison.OrdinalIgnoreCase);
}

public sealed class AuthorizationContext
{
    public bool IsAuthenticated { get; set; }
    public UserRole Role { get; set; }
    public UserStatus UserStatus { get; set; }
    public string HttpMethod { get; set; } = string.Empty;
    public string EndpointPath { get; set; } = string.Empty;
    public string? SubscriptionTier { get; set; }
}

public enum UserRole
{
    Guest,
    Viewer,
    Editor,
    Manager,
    Admin
}

public enum UserStatus
{
    Active,
    Suspended,
    Deactivated,
    PendingVerification
}

public enum AccessLevel
{
    None,
    ReadOnly,
    ReadWrite,
    Full
}

public sealed class AuthorizationResult
{
    public bool IsAllowed { get; private set; }
    public string Reason { get; private set; } = string.Empty;
    public int StatusCode { get; private set; }

    public static AuthorizationResult Allow() => new()
    {
        IsAllowed = true,
        Reason = "Access granted.",
        StatusCode = 200
    };

    public static AuthorizationResult Deny(string reason, int statusCode) => new()
    {
        IsAllowed = false,
        Reason = reason,
        StatusCode = statusCode
    };
}
```

### Key Observations

- Guard clauses reject common invalid states before the main access decision, reducing nesting.
- `is UserStatus.Suspended or UserStatus.Deactivated` uses an `or` pattern to check multiple enum values.
- The tuple switch matches the role, write/read status, and restricted-path status together.
- A `when` guard adds a Boolean condition to a switch arm.
- `IsPathOrChild` uses explicit ordinal, case-insensitive comparisons and a path-segment boundary. It is still only a demonstration: production authorization should use the application's routing and policy model, not rely on this helper as its security boundary.
- The cache-policy example uses a private policy for overrides; public caching should be limited to resources confirmed to be safe for shared caches.
- This code does not define complete business, identity, routing, or rate-limiting policies. Those must be designed and tested for the application.

## Key Terms Summary

| Term | Definition |
|---|---|
| `if` statement | Executes a statement or block when a Boolean condition is true. |
| `else` clause | Provides the alternative block when the preceding `if` condition is false. |
| `else if` clause | Adds another condition, checked only when earlier conditions were false. |
| Guard clause | An early return that handles an edge case or failed precondition. |
| `switch` statement | Selects a statement section by matching an expression against case patterns. |
| `switch` expression | Matches an expression against arms and produces a value. |
| Type pattern | Tests a value's runtime type; a declaration pattern can also capture the value. |
| Relational pattern | Compares a value with a constant using `<`, `>`, `<=`, or `>=`. |
| Logical pattern | Combines patterns with `and`, `or`, or `not`. |
| Property pattern | Matches one or more properties or fields of an object. |
| Positional pattern | Matches values produced by deconstruction, often tuple elements. |
| List pattern | Matches the length and elements of an array or another supported sequence-like value. |
| Discard pattern | `_`; matches a value without capturing it, often as a catch-all arm. |
| Case guard | A `when` condition that must also be true for a switch case or arm to match. |
| Exhaustiveness | Whether a switch expression handles every value that can reach it. |

## Further Reading

- [4](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements) — C# `if` and `switch` statements.
- [3](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression) — C# `switch` expressions and arms.
- [2](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns) — C# pattern forms, including relational, tuple, property, logical, and list patterns.
- [1](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/pattern-matching-warnings) — Compiler warnings and errors for non-exhaustive or unreachable patterns.
