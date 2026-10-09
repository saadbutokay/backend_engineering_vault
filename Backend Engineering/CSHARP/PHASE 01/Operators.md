## What Are Operators?
Operators are symbols that perform operations on one or more operands (variables, values, or expressions). C# provides operators used throughout backend logic—from calculating order totals to evaluating authorization rules and filtering database queries.

Operators are categorized by the number of operands they work on:

- **Unary operators:** Operate on one operand. Examples: `-x`, `!isActive`, `count++`.
- **Binary operators:** Operate on two operands. Examples: `a + b`, `x > y`, `flag1 && flag2`.
- **Ternary operator:** The conditional operator `?:` takes three operands. Example: `condition ? valueIfTrue : valueIfFalse`.

## Arithmetic Operators

Arithmetic operators perform mathematical calculations. They work on numeric types such as `int`, `long`, `float`, `double`, and `decimal`.

| Operator | Name | Example | Result |
|---|---|---|---:|
| `+` | Addition | `5 + 3` | 8 |
| `-` | Subtraction | `10 - 4` | 6 |
| `*` | Multiplication | `6 * 7` | 42 |
| `/` | Division | `20 / 3` | 6 (integer division) |
| `%` | Modulus (remainder) | `20 % 3` | 2 |

### Integer Division vs. Floating-Point Division

When both operands are integers, division produces an integer result. The fractional part is truncated toward zero, not rounded.

```csharp
int a = 10;
int b = 3;
int result = a / b;
Console.WriteLine(result); // Output: 3 (not 3.333...)

// To get a floating-point result, at least one operand must be floating-point.
double preciseResult = (double)a / b;
Console.WriteLine(preciseResult); // Output: approximately 3.3333333333333335

decimal moneyResult = 10m / 3m;
Console.WriteLine(moneyResult); // Output: 3.3333333333333333333333333333
```

### Modulus Operator

The modulus operator returns the remainder of a division. It is useful in backend logic for tasks such as pagination, cycling through values, and determining whether a number is even or odd.

```csharp
int remainder = 17 % 5;
Console.WriteLine(remainder); // Output: 2

// Practical use: check whether a number is even.
int pageNumber = 4;
bool isEven = pageNumber % 2 == 0;
Console.WriteLine(isEven); // Output: True

// Practical use: wrap around an index (circular buffer).
int[] days = { 0, 1, 2, 3, 4, 5, 6 };
int currentIndex = 6;
int nextIndex = (currentIndex + 1) % days.Length;
Console.WriteLine(nextIndex); // Output: 0 (wraps around)
```

### Increment and Decrement Operators

These unary operators add or subtract 1 from a variable.

| Operator | Name | Example | Effect |
|---|---|---|---|
| `++` | Increment | `count++` or `++count` | Adds 1 |
| `--` | Decrement | `count--` or `--count` | Subtracts 1 |

The position of the operator matters when it is part of a larger expression:

- **Postfix (`count++`):** Produces the current value, then increments the variable.
- **Prefix (`++count`):** Increments the variable first, then produces the new value.

```csharp
int count = 5;

int postResult = count++;
Console.WriteLine(postResult); // Output: 5 (produced before increment)
Console.WriteLine(count);      // Output: 6

int preResult = ++count;
Console.WriteLine(preResult); // Output: 7 (incremented first)
Console.WriteLine(count);     // Output: 7
```

Industry convention: avoid using increment/decrement operators inside complex expressions. It can make code harder to read and introduce subtle bugs. Use them as standalone statements when possible.

```csharp
// Clear and readable
count++;

// Confusing — avoid this
// int result = array[i++] + array[++j];
```

## Comparison Operators

Comparison operators compare two values and return a `bool` (`true` or `false`). They are the foundation of conditional logic.

| Operator | Name | Example | Result |
|---|---|---|---|
| `==` | Equal to | `5 == 5` | `true` |
| `!=` | Not equal to | `5 != 3` | `true` |
| `<` | Less than | `3 < 5` | `true` |
| `>` | Greater than | `5 > 3` | `true` |
| `<=` | Less than or equal to | `5 <= 5` | `true` |
| `>=` | Greater than or equal to | `5 >= 6` | `false` |

### Value Types vs. Reference Types with `==`

For built-in value types such as `int`, `==` compares values. User-defined structs do not automatically get a field-by-field `==` operator; a struct must define or overload `==` for that operator to be available.

```csharp
int x = 10;
int y = 10;
Console.WriteLine(x == y); // Output: True (same value)
```

For reference types, `==` compares references by default, unless the type overloads the operator. `string` overloads `==` to compare text content; some other types, such as records, also define value-based equality.

```csharp
string a = "hello";
string b = "hello";
Console.WriteLine(a == b); // Output: True (string compares text content)

var list1 = new List<int> { 1, 2, 3 };
var list2 = new List<int> { 1, 2, 3 };
Console.WriteLine(list1 == list2); // Output: False (List<T> uses reference equality)
```

### Comparing Floating-Point Numbers

Floating-point arithmetic can introduce small precision differences. Avoid direct equality checks when you need to compare approximate `float` or `double` results. Instead, choose an appropriate tolerance for the scale and requirements of the calculation.

```csharp
double result = 0.1 + 0.2;
Console.WriteLine(result == 0.3); // Output: False

// Example: compare with an absolute tolerance (epsilon).
double epsilon = 0.0000001;
bool areEqual = Math.Abs(result - 0.3) < epsilon;
Console.WriteLine(areEqual); // Output: True
```

For `decimal`, common base-10 fractions such as `0.1m` and `0.2m` can be represented exactly, so direct comparisons are often appropriate when the calculation and rounding rules are controlled. `decimal` still has finite precision; it is not a substitute for explicit rounding rules.

## Logical Operators

Logical operators combine or invert Boolean expressions. They are essential for conditions in authorization checks, validation rules, and business logic.

| Operator | Name | Example | Result |
|---|---|---|---|
| `&&` | Logical AND | `true && false` | `false` |
| `||` | Logical OR | `true || false` | `true` |
| `!` | Logical NOT | `!true` | `false` |

### Short-Circuit Evaluation

The `&&` and `||` operators use short-circuit evaluation: they stop evaluating as soon as the result is determined.

- **`&&`:** If the left side is `false`, the right side is not evaluated (the result must be `false`).
- **`||`:** If the left side is `true`, the right side is not evaluated (the result must be `true`).

```csharp
bool isAdult = true;
bool hasValidId = false;

// The second condition is evaluated only if the first is true.
if (isAdult && hasValidId)
{
    Console.WriteLine("Access granted");
}
else
{
    Console.WriteLine("Access denied"); // This runs
}

// Practical example: avoid dereferencing a null value.
string? username = null;

// If username is null, the second condition is not evaluated.
if (username != null && username.Length > 3)
{
    Console.WriteLine("Valid username");
}
```

### Logical AND vs. Bitwise AND

C# has both `&&` (conditional/logical AND) and `&` (bitwise AND). When used with Boolean operands, `&` evaluates both sides; it does not short-circuit. Use `&&` and `||` for ordinary Boolean logic unless you specifically need non-short-circuit evaluation.

```csharp
bool left = false;

// Short-circuit: the function is not called.
bool result1 = left && ExpensiveDatabaseCall();

// No short-circuit: the function is called even though left is false.
bool result2 = left & ExpensiveDatabaseCall();

static bool ExpensiveDatabaseCall()
{
    Console.WriteLine("The function was evaluated.");
    return true;
}
```

## Assignment Operators

Assignment operators assign a value to a variable. Compound assignment operators combine an operation with assignment.

| Operator | Name | Example | Equivalent to |
|---|---|---|---|
| `=` | Simple assignment | `x = 5` | — |
| `+=` | Add and assign | `x += 3` | `x = x + 3` |
| `-=` | Subtract and assign | `x -= 3` | `x = x - 3` |
| `*=` | Multiply and assign | `x *= 3` | `x = x * 3` |
| `/=` | Divide and assign | `x /= 3` | `x = x / 3` |
| `%=` | Modulus and assign | `x %= 3` | `x = x % 3` |

```csharp
decimal total = 100m;

total += 50m;   // total is now 150
total -= 25m;   // total is now 125
total *= 2m;    // total is now 250
total /= 5m;    // total is now 50
total %= 30m;   // total is now 20

Console.WriteLine(total); // Output: 20
```

The `+=` operator also works with strings (concatenation) and delegates (event subscription). For repeated string construction in loops, `StringBuilder` is often more efficient.

```csharp
string message = "Hello";
message += " World";
Console.WriteLine(message); // Output: Hello World
```

## The Ternary Operator

The conditional operator `?:` is shorthand for a simple `if`/`else` that produces a value.

Syntax: `condition ? valueIfTrue : valueIfFalse`

```csharp
int age = 20;

// Using if-else
string status;
if (age >= 18)
{
    status = "Adult";
}
else
{
    status = "Minor";
}

// Using the conditional (ternary) operator
string statusTernary = age >= 18 ? "Adult" : "Minor";

Console.WriteLine(statusTernary); // Output: Adult
```

### When to Use the Ternary Operator

Use it for simple, single-condition expressions. Avoid deeply nested conditional operators; they are difficult to read.

```csharp
// Sample input values
decimal orderTotal = 1250m;
bool isActive = true;
int points = 7000;

// Good: simple and clear
decimal discount = orderTotal > 1000m ? 0.10m : 0.05m;
string label = isActive ? "Active" : "Inactive";

// Hard to read: nested conditional operators (shown as a comment).
// string nestedTier = points > 10000 ? "Platinum" : points > 5000 ? "Gold" : points > 1000 ? "Silver" : "Bronze";

// Better: use a switch expression for multiple conditions (covered in 1.9)
string tier = points switch
{
    > 10000 => "Platinum",
    > 5000 => "Gold",
    > 1000 => "Silver",
    _ => "Bronze"
};
```

## Null-Coalescing Operators

These operators handle null values and are used extensively in backend code.

### Null-Coalescing Operator (`??`)

Returns the left operand if it is not null; otherwise, returns the right operand.

```csharp
string? username = null;
string displayName = username ?? "Anonymous";
Console.WriteLine(displayName); // Output: Anonymous

username = "Alice";
displayName = username ?? "Anonymous";
Console.WriteLine(displayName); // Output: Alice
```

### Null-Coalescing Assignment Operator (`??=`)

Assigns the right operand to the left operand only if the left operand is null.

```csharp
List<string>? tags = null;

// Only initializes the list if tags is null.
tags ??= new List<string>();
tags.Add("backend");
tags.Add("csharp");

Console.WriteLine(tags.Count); // Output: 2

// This does nothing because tags is no longer null.
tags ??= new List<string>();
Console.WriteLine(tags.Count); // Output: 2 (not reset)
```

This is common in lazy-initialization patterns in backend services.

## Null-Conditional Operators

These operators safely access members of an object that might be null, without throwing a `NullReferenceException` for that access.

### Null-Conditional Member Access (`?.`)

```csharp
string? customerName = null;

// Without null-conditional access, this would throw if customerName were null.
// int length = customerName.Length;

// With null-conditional access, the result is null instead of an exception.
int? length = customerName?.Length;
Console.WriteLine(length.HasValue); // Output: False

customerName = "Alice";
length = customerName?.Length;
Console.WriteLine(length); // Output: 5
```

### Null-Conditional Index Access (`?[]`)

```csharp
int[]? scores = null;

// Returns null instead of throwing when scores is null.
int? firstScore = scores?[0];
Console.WriteLine(firstScore.HasValue); // Output: False

scores = new int[] { 85, 92, 78 };
firstScore = scores?[0];
Console.WriteLine(firstScore); // Output: 85
```

### Chaining Null-Conditional Operators

You can chain `?.` to safely navigate object graphs. If any part of the chain is null, the expression short-circuits and evaluates to null.

```csharp
// Without null-conditional access, this can throw if Customer or Address is null.
// string city = order.Customer.Address.City;

// With null-conditional access.
Order? order = null;
string? city = order?.Customer?.Address?.City;

record Order(Customer? Customer);
record Customer(Address? Address);
record Address(string City);
```

## Bitwise Operators

Bitwise operators manipulate individual bits of integer values. They are less common in typical backend CRUD applications but appear in permission systems, feature flags, and low-level optimizations.

| Operator | Name | Example | Description |
|---|---|---|---|
| `&` | Bitwise AND | `5 & 3` | A result bit is 1 only if both corresponding bits are 1 |
| `|` | Bitwise OR | `5 \| 3` | A result bit is 1 if either corresponding bit is 1 |
| `^` | Bitwise XOR | `5 ^ 3` | A result bit is 1 if the corresponding bits differ |
| `~` | Bitwise complement | `~5` | Flips all bits |
| `<<` | Left shift | `5 << 1` | Shifts bits left |
| `>>` | Right shift | `5 >> 1` | Shifts bits right |

```csharp
// Practical example: permission flags using bitwise OR.
Permission userPermissions = Permission.Read | Permission.Write; // 0011

// Check whether the user has Read permission using bitwise AND.
bool canRead = (userPermissions & Permission.Read) == Permission.Read;
Console.WriteLine(canRead); // Output: True

bool canDelete = (userPermissions & Permission.Delete) == Permission.Delete;
Console.WriteLine(canDelete); // Output: False

[Flags]
enum Permission
{
    None = 0,       // 0000
    Read = 1,       // 0001
    Write = 2,      // 0010
    Delete = 4,     // 0100
    Admin = 8       // 1000
}
```

## Operator Precedence

When multiple operators appear in an expression, C# evaluates them according to a defined precedence order. Operators with higher precedence are evaluated first.

From highest to lowest precedence (simplified for the operators covered here):

| Precedence | Operators | Category |
|---:|---|---|
| 1 (highest) | `?.`, `?[]`, `++`, `--`, `+x`, `-x`, `!`, `~` | Member access and unary |
| 2 | `*`, `/`, `%` | Multiplicative |
| 3 | `+`, `-` | Additive |
| 4 | `<<`, `>>` | Shift |
| 5 | `<`, `>`, `<=`, `>=` | Relational |
| 6 | `==`, `!=` | Equality |
| 7 | `&` | Bitwise AND |
| 8 | `^` | Bitwise XOR |
| 9 | `\|` | Bitwise OR |
| 10 | `&&` | Logical AND |
| 11 | `\|\|` | Logical OR |
| 12 | `??` | Null-coalescing |
| 13 | `?:` | Conditional (ternary) |
| 14 (lowest) | `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `??=` | Assignment |

Best practice: when in doubt, use parentheses. They make your intent explicit and prevent precedence bugs.

```csharp
// Unclear: relies on precedence rules.
bool result = a + b > c * d && e == f;

// Clear: parentheses make the order obvious.
bool result = ((a + b) > (c * d)) && (e == f);
```

## Basic Code Snippet

```csharp
// Program.cs — Operators demo in .NET 10

// --- Arithmetic ---
int quantity = 5;
decimal unitPrice = 29.99m;
decimal subtotal = quantity * unitPrice;
decimal tax = subtotal * 0.08m;
decimal total = subtotal + tax;

Console.WriteLine("=== Arithmetic ===");
Console.WriteLine($"Subtotal: {subtotal}");
Console.WriteLine($"Tax: {tax}");
Console.WriteLine($"Total: {total}");
Console.WriteLine($"Remainder of 17 / 5: {17 % 5}");

// --- Comparison ---
int minAge = 18;
int userAge = 21;
bool isEligible = userAge >= minAge;

Console.WriteLine("\n=== Comparison ===");
Console.WriteLine($"User age: {userAge}, Min age: {minAge}");
Console.WriteLine($"Is eligible: {isEligible}");
Console.WriteLine($"Ages equal: {userAge == minAge}");

// --- Logical ---
bool hasAccount = true;
bool isVerified = false;
bool canPost = hasAccount && isVerified;
bool canBrowse = hasAccount || isVerified;

Console.WriteLine("\n=== Logical ===");
Console.WriteLine($"Can post (AND): {canPost}");
Console.WriteLine($"Can browse (OR): {canBrowse}");
Console.WriteLine($"Not verified: {!isVerified}");

// --- Ternary ---
decimal orderAmount = 150m;
decimal shippingCost = orderAmount > 100m ? 0m : 9.99m;

Console.WriteLine("\n=== Ternary ===");
Console.WriteLine($"Order: {orderAmount}, Shipping: {shippingCost}");

// --- Null-Coalescing ---
string? configValue = null;
string connectionString = configValue ?? "Server=localhost;Database=Default;";

Console.WriteLine("\n=== Null-Coalescing ===");
Console.WriteLine($"Connection: {connectionString}");

// --- Null-Conditional ---
string? userName = null;
int? nameLength = userName?.Length;

Console.WriteLine("\n=== Null-Conditional ===");
Console.WriteLine($"Name length: {nameLength?.ToString() ?? "null"}");

userName = "Alice";
nameLength = userName?.Length;
Console.WriteLine($"Name length: {nameLength}");
```

## Industry-Level Code Snippet

This is a realistic pricing service from an e-commerce backend. It demonstrates operators in business logic: calculating discounts, validating rules, handling nulls, and combining conditions.

```csharp
// PricingService.cs — Pricing logic in an e-commerce API
using System;
using System.Collections.Generic;

namespace MyBackendApp.Core.Services;

public class PricingService
{
    private const decimal StandardTaxRate = 0.08m;
    private const decimal ReducedTaxRate = 0.04m;
    private const decimal FreeShippingThreshold = 75.00m;
    private const decimal StandardShippingCost = 12.99m;
    private const int MaxDiscountPercentage = 50;

    public OrderPricingResult CalculatePricing(OrderPricingRequest request)
    {
        // Null-coalescing: provide defaults for optional inputs.
        decimal couponDiscount = request.CouponDiscountPercentage ?? 0m;
        string? promoCode = request.PromoCode?.Trim().ToUpperInvariant();
        bool isTaxExempt = request.IsTaxExempt ?? false;

        // Validate discount range using comparison and logical operators.
        if (couponDiscount < 0m || couponDiscount > MaxDiscountPercentage)
        {
            throw new ArgumentOutOfRangeException(
                nameof(couponDiscount),
                $"Discount must be between 0 and {MaxDiscountPercentage}%.");
        }

        // Calculate subtotal using arithmetic operators.
        decimal subtotal = 0m;
        foreach (var item in request.Items)
        {
            subtotal += item.Quantity * item.UnitPrice;
        }

        // Apply discount using compound assignment.
        decimal discountAmount = subtotal * (couponDiscount / 100m);
        decimal afterDiscount = subtotal - discountAmount;

        // Determine tax rate using the conditional (ternary) operator.
        decimal taxRate = isTaxExempt
            ? 0m
            : (afterDiscount < 50m ? ReducedTaxRate : StandardTaxRate);

        decimal taxAmount = afterDiscount * taxRate;

        // Determine shipping cost using comparison and conditional operators.
        decimal shippingCost = afterDiscount >= FreeShippingThreshold
            ? 0m
            : StandardShippingCost;

        // Calculate final total.
        decimal totalAmount = afterDiscount + taxAmount + shippingCost;

        // Determine order tier using null-conditional and null-coalescing operators.
        string customerTier = request.Customer?.LoyaltyTier ?? "Standard";
        bool qualifiesForPriority = customerTier == "Platinum" || customerTier == "Gold";

        // Build result using null-coalescing assignment for optional notes.
        string? notes = null;
        if (promoCode != null && promoCode.Length > 0)
        {
            notes ??= $"Promo code applied: {promoCode}";
        }

        if (shippingCost == 0m)
        {
            string freeShippingNote = "Free shipping applied.";
            notes = notes != null ? $"{notes} | {freeShippingNote}" : freeShippingNote;
        }

        return new OrderPricingResult
        {
            Subtotal = subtotal,
            DiscountAmount = discountAmount,
            TaxAmount = taxAmount,
            ShippingCost = shippingCost,
            TotalAmount = totalAmount,
            TaxRate = taxRate,
            CustomerTier = customerTier,
            IsPriorityOrder = qualifiesForPriority,
            Notes = notes
        };
    }
}

public class OrderPricingRequest
{
    public List<OrderItemDto> Items { get; set; } = new();
    public int? CouponDiscountPercentage { get; set; }
    public string? PromoCode { get; set; }
    public bool? IsTaxExempt { get; set; }
    public CustomerInfo? Customer { get; set; }
}

public class OrderItemDto
{
    public string ProductName { get; set; } = string.Empty;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public class CustomerInfo
{
    public int Id { get; set; }
    public string? LoyaltyTier { get; set; }
}

public class OrderPricingResult
{
    public decimal Subtotal { get; set; }
    public decimal DiscountAmount { get; set; }
    public decimal TaxAmount { get; set; }
    public decimal ShippingCost { get; set; }
    public decimal TotalAmount { get; set; }
    public decimal TaxRate { get; set; }
    public string CustomerTier { get; set; } = string.Empty;
    public bool IsPriorityOrder { get; set; }
    public string? Notes { get; set; }
}
```

Key observations from this industry code:

- `??` provides defaults for nullable inputs from the API request.
- `?.` safely accesses `request.PromoCode` and `request.Customer?.LoyaltyTier`; if an object in the chain is null, the expression evaluates to null.
- `??=` initializes the notes string only when it is null.
- `||` and `&&` combine conditions into readable business rules.
- The conditional operator is used for simple two-way decisions, such as selecting a tax rate or shipping cost.
- Comparison operators enforce business constraints, such as the discount range and free-shipping threshold.
- `decimal` is used for financial values. Apply explicit rounding and currency-precision rules; `decimal` does not eliminate every possible rounding issue.
- Compound assignment (`+=`) accumulates the subtotal in the loop.

## Key Terms Summary

| Term | Definition |
|---|---|
| Unary operator | An operator that takes one operand. Examples: `!`, `++`, `--`, `-`. |
| Binary operator | An operator that takes two operands. Examples: `+`, `-`, `==`, `&&`. |
| Ternary operator | The `?:` conditional operator. It takes a condition, a true value, and a false value. |
| Short-circuit evaluation | `&&` and `||` may skip evaluating the right operand when the result is already known. |
| Integer division | Division between integer operands that truncates the fractional part toward zero. |
| Modulus | The `%` operator, which returns the remainder of a division. |
| Null-coalescing | The `??` operator, which returns the left value if it is not null; otherwise it returns the right value. |
| Null-conditional | `?.` and `?[]`, which safely access members or elements when the receiver may be null. |
| Operator precedence | The order in which operators are evaluated in an expression. |
| Bitwise operator | An operator that manipulates individual bits. Examples: `&`, `|`, `^`, `~`. |
