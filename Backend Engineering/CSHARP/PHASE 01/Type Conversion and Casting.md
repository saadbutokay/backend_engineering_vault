## Why Type Conversion Matters
In C#, every variable has a type determined at compile time. Real-world backend applications often need to move data between types: for example, parsing an integer from an HTTP query parameter, formatting a decimal for display, or passing a derived object to a method that expects a base type.

C# provides several conversion mechanisms. Choosing the wrong one can cause data loss, runtime exceptions, or subtle bugs in production.

## Implicit Conversions (Widening)

An implicit conversion is performed automatically by the compiler when the language defines a conversion between the source and target types. Many numeric implicit conversions preserve the full range of values, but not all preserve precision. For example, converting a sufficiently large integer to `float` or `double` can lose low-order digits.

### Numeric Implicit Conversions

Some common examples are:

```text
byte -> short -> int -> long
float -> double
```

There are additional implicit conversions between integral types and `float`, `double`, or `decimal`. The exact conversion rules depend on the source and target types. For example, `int` can be implicitly converted to `float`, but some large `int` values cannot be represented exactly as a `float`.

Conversions between `float`/`double` and `decimal` are not implicit in either direction; they require an explicit conversion because the types use different representations.

```csharp
// Implicit conversions between integral types.
byte smallNumber = 200;
int largerNumber = smallNumber;       // byte -> int
long evenLarger = largerNumber;       // int -> long

double floatingPoint = evenLarger;    // long -> double (may lose precision for large values)

Console.WriteLine(smallNumber);       // 200
Console.WriteLine(largerNumber);      // 200
Console.WriteLine(evenLarger);        // 200
Console.WriteLine(floatingPoint);     // 200

// int -> double is exact for this value.
int count = 42;
double precise = count;
Console.WriteLine(precise);           // 42

// float -> double is allowed, but it preserves the float's existing approximation.
float single = 3.14f;
double doubleValue = single;
Console.WriteLine(doubleValue);       // Approximately 3.140000104904175
```

### Reference Type Implicit Conversions

A derived class can be implicitly converted to its base class or to an interface it implements. This is the foundation of polymorphism.

```csharp
Dog myDog = new Dog();
Animal myAnimal = myDog; // Implicit: Dog is an Animal

// The object is still a Dog, but this variable exposes it as an Animal.

class Animal { }
class Dog : Animal { }
```

### Implicit Conversion from Constant Values

The compiler permits certain constant integer expressions to convert implicitly to smaller integral types when the value fits within the target type's range.

```csharp
int number = 100;
byte smallByte = 100;     // Constant value 100 fits in a byte (0–255)
// byte overflow = 300;   // Compile error: 300 does not fit in a byte
```

## Explicit Conversions (Narrowing / Casting)

An explicit conversion is required when the compiler cannot assume the conversion is suitable for every possible source value. You write a cast using `(TargetType)expression`.

### Numeric Explicit Conversions

```csharp
double preciseValue = 9.99;
int truncated = (int)preciseValue;
Console.WriteLine(truncated); // Output: 9 (fractional part is truncated, not rounded)

long bigNumber = 3_000_000_000L;
int wrapped = unchecked((int)bigNumber);
Console.WriteLine(wrapped); // Output: -1294967296 in an unchecked context

double negative = -5.7;
int negativeInt = (int)negative;
Console.WriteLine(negativeInt); // Output: -5 (truncates toward zero)
```

For an out-of-range integral-to-integral cast in an `unchecked` context, the result wraps to the target type's range. The behavior of out-of-range floating-point-to-integral conversions is different; do not rely on an unchecked cast to validate or clamp input. Check ranges or use a checked conversion when overflow must be detected.

### Checked and Unchecked Contexts

Use `checked` to request an `OverflowException` when a checked integral conversion or arithmetic operation overflows. Use `unchecked` when wraparound behavior is specifically intended.

```csharp
long bigNumber = 3_000_000_000L;

// Unchecked: integral narrowing wraps around.
int wrapped = unchecked((int)bigNumber);
Console.WriteLine(wrapped); // -1294967296

// Checked: throws OverflowException.
try
{
    int safe = checked((int)bigNumber);
}
catch (OverflowException ex)
{
    Console.WriteLine($"Overflow detected: {ex.Message}");
}
```

You can enable checked arithmetic for a project in its `.csproj` file:

```xml
<PropertyGroup>
    <CheckForOverflowUnderflow>true</CheckForOverflowUnderflow>
</PropertyGroup>
```

The default arithmetic context is unchecked unless the project or code specifies otherwise. Choose a project-wide policy deliberately, and use `checked` at boundaries where overflow would make a result invalid.

## Reference Type Explicit Conversions (Downcasting)

Casting from a base class to a derived class requires an explicit cast because the compiler cannot guarantee that the object has the derived type at runtime.

```csharp
Animal myAnimal = new Dog();

// Explicit cast: this succeeds because the object is actually a Dog.
Dog myDog = (Dog)myAnimal;

// This compiles, but throws InvalidCastException at runtime.
// Cat myCat = (Cat)myAnimal;

class Animal { }
class Dog : Animal { }
class Cat : Animal { }
```

## The `is` and `as` Operators

These operators provide safer alternatives to an unconditional cast for reference types.

### The `is` Operator

The `is` operator checks whether an object is compatible with a type. It returns a Boolean and does not throw when the type test fails.

```csharp
Animal myAnimal = new Dog();

bool isDog = myAnimal is Dog;
bool isCat = myAnimal is Cat;

Console.WriteLine(isDog); // True
Console.WriteLine(isCat); // False

// Pattern matching with is (C# 7+)
if (myAnimal is Dog dog)
{
    // 'dog' is a variable of type Dog, already checked and converted.
    Console.WriteLine("It is a dog!");
}

class Animal { }
class Dog : Animal { }
class Cat : Animal { }
```

### The `as` Operator

The `as` operator attempts a conversion to a reference type or nullable value type. If the conversion is incompatible, it returns `null` instead of throwing `InvalidCastException`.

```csharp
Animal myAnimal = new Dog();

Dog? myDog = myAnimal as Dog;
Cat? myCat = myAnimal as Cat;

Console.WriteLine(myDog != null); // True
Console.WriteLine(myCat != null); // False (myCat is null)

if (myCat != null)
{
    Console.WriteLine("It is a cat!");
}
else
{
    Console.WriteLine("Not a cat.");
}

class Animal { }
class Dog : Animal { }
class Cat : Animal { }
```

### `is` vs. `as` vs. Direct Cast

| Approach | What happens if the object has the wrong type? | Throws on wrong type? | Result on wrong type |
|---|---|---|---|
| `(Dog)animal` | Conversion fails | Yes (`InvalidCastException`) | No value is produced |
| `animal as Dog` | Conversion fails | No | `null` |
| `animal is Dog` | Type test fails | No | `false` |

Prefer `is` with pattern matching for type checks. Use `as` when `null` is an acceptable result. Use a direct cast when a failed conversion indicates a programming error and should be reported.

## The `Convert` Class

`System.Convert` provides methods for converting between many built-in types. The exact behavior depends on the overload: for example, `Convert.ToInt32(double)` rounds to the nearest integer using midpoint-to-even rounding, while an explicit cast from `double` to `int` truncates toward zero.

### `Convert` vs. Cast — Key Differences

| Behavior | Explicit cast, such as `(int)value` | `Convert.ToInt32(value)` |
|---|---|---|
| `double` to `int` | Truncates toward zero | Rounds to the nearest integer; midpoint values round to even |
| Null input | Depends on source and target; a cast does not generally turn `null` into zero | `Convert.ToInt32` returns 0 for a null string or object reference |
| String input | A cast does not parse a string to a number | Parses supported strings using culture-sensitive rules; can throw on invalid input |
| Overflow | Unchecked integral narrowing can wrap; floating-point edge cases should not be relied on for validation | Numeric conversions outside the supported range throw `OverflowException` |

### Common `Convert` Methods

```csharp
using System.Globalization;

// --- From double to int ---
double price = 9.7;
int castResult = (int)price;                 // 9 (truncated)
int convertResult = Convert.ToInt32(price);  // 10 (rounded)

Console.WriteLine($"Cast: {castResult}, Convert: {convertResult}");

// --- From string to numeric ---
string numberString = "42";
int fromString = Convert.ToInt32(numberString);
double fromStringDouble = Convert.ToDouble(numberString, CultureInfo.InvariantCulture);
decimal fromStringDecimal = Convert.ToDecimal(numberString, CultureInfo.InvariantCulture);

Console.WriteLine(fromString);        // 42
Console.WriteLine(fromStringDouble);  // 42
Console.WriteLine(fromStringDecimal); // 42

// --- Null handling for Convert.ToInt32 ---
string? nullString = null;
int fromNull = Convert.ToInt32(nullString);
Console.WriteLine(fromNull); // 0

// --- Boolean conversions ---
int zero = 0;
int one = 1;
Console.WriteLine(Convert.ToBoolean(zero)); // False
Console.WriteLine(Convert.ToBoolean(one));  // True

string trueString = "True";
Console.WriteLine(Convert.ToBoolean(trueString)); // True

// --- Base64 encoding (common in backend APIs) ---
byte[] data = System.Text.Encoding.UTF8.GetBytes("Hello, World!");
string base64 = Convert.ToBase64String(data);
Console.WriteLine(base64); // SGVsbG8sIFdvcmxkIQ==

byte[] decoded = Convert.FromBase64String(base64);
string original = System.Text.Encoding.UTF8.GetString(decoded);
Console.WriteLine(original); // Hello, World!
```

### `Convert` Rounding Behavior (Banker's Rounding)

For numeric overloads such as `Convert.ToInt32(double)`, midpoint values use **round to nearest, ties to even** (also called banker's rounding). For other values, the nearest integer is returned. This rounding rule reduces statistical bias in some large data sets.

```csharp
Console.WriteLine(Convert.ToInt32(2.5)); // 2 (nearest even)
Console.WriteLine(Convert.ToInt32(3.5)); // 4 (nearest even)
Console.WriteLine(Convert.ToInt32(4.5)); // 4 (nearest even)
Console.WriteLine(Convert.ToInt32(5.5)); // 6 (nearest even)
```

## Parsing Strings to Other Types

Parsing converts a string representation into a typed value. This is common in backend development because HTTP query parameters, configuration values, and other textual inputs may need to be converted before use.

### `Parse` Methods

Numeric types and several other types provide static `Parse` methods. Parsing behavior for numbers and dates can depend on culture, so specify the expected culture for machine-readable formats.

```csharp
using System.Globalization;

string intString = "42";
string doubleString = "3.14";
string decimalString = "19.99";
string boolString = "true";
string dateString = "2025-06-15";

int number = int.Parse(intString, CultureInfo.InvariantCulture);
double pi = double.Parse(doubleString, CultureInfo.InvariantCulture);
decimal price = decimal.Parse(decimalString, CultureInfo.InvariantCulture);
bool flag = bool.Parse(boolString);
DateTime date = DateTime.Parse(dateString, CultureInfo.InvariantCulture);

Console.WriteLine(number); // 42
Console.WriteLine(pi);     // 3.14
Console.WriteLine(price);  // 19.99
Console.WriteLine(flag);   // True
Console.WriteLine(date.ToString("yyyy-MM-dd", CultureInfo.InvariantCulture)); // 2025-06-15
```

If the string is not valid, `Parse` throws an exception: commonly `FormatException` for malformed input, `OverflowException` when a numeric value is outside the target type's range, or `ArgumentNullException` for a null string.

```csharp
// These throw exceptions if executed:
// int.Parse("abc");        // FormatException
// int.Parse("9999999999"); // OverflowException
// int.Parse(null);         // ArgumentNullException
// int.Parse("");           // FormatException
```

### `TryParse` Methods (Preferred for Untrusted Input)

`TryParse` returns a Boolean indicating success or failure and provides the parsed value through an `out` parameter. It returns `false` for ordinary invalid or out-of-range input instead of throwing a parsing exception.

```csharp
string userInput = "42";

if (int.TryParse(userInput, out int parsedNumber))
{
    Console.WriteLine($"Parsed successfully: {parsedNumber}");
}
else
{
    Console.WriteLine("Invalid input. Please enter a valid integer.");
}

string badInput = "abc";
if (int.TryParse(badInput, out int failedNumber))
{
    Console.WriteLine($"Parsed: {failedNumber}");
}
else
{
    Console.WriteLine("Failed to parse."); // This runs
    Console.WriteLine($"Default value: {failedNumber}"); // 0
}
```

### Parsing with Culture and Format

Numbers and dates are formatted differently across cultures. For machine-readable input, follow the format defined by the API or file contract and parse with that culture. For localized user input, use the user's intended culture.

```csharp
using System.Globalization;

string usPrice = "1,234.56";
string euPrice = "1.234,56";

// Parse with US culture.
decimal usDecimal = decimal.Parse(usPrice, CultureInfo.GetCultureInfo("en-US"));
Console.WriteLine(usDecimal); // 1234.56

// Parse with German culture.
decimal euDecimal = decimal.Parse(euPrice, CultureInfo.GetCultureInfo("de-DE"));
Console.WriteLine(euDecimal); // 1234.56

// Parse an invariant, machine-readable number.
decimal invariant = decimal.Parse("1234.56", CultureInfo.InvariantCulture);
Console.WriteLine(invariant); // 1234.56

// Parse a date with an explicit format.
string dateString = "15/06/2025";
DateTime parsed = DateTime.ParseExact(
    dateString,
    "dd/MM/yyyy",
    CultureInfo.InvariantCulture);
Console.WriteLine(parsed.ToString("yyyy-MM-dd", CultureInfo.InvariantCulture)); // 2025-06-15
```

For API payloads and configuration files, use the culture and format specified by the contract; invariant culture is common for machine-readable numeric formats. For localized UI input, parse with the user's culture rather than assuming invariant formatting.

### Parsing Enums

```csharp
string statusString = "Shipped";

// Parse throws if the text cannot be parsed.
OrderStatus status = Enum.Parse<OrderStatus>(statusString);
Console.WriteLine(status); // Shipped

// TryParse is safer for untrusted input.
if (Enum.TryParse<OrderStatus>("Delivered", out OrderStatus parsedStatus))
{
    Console.WriteLine(parsedStatus); // Delivered
}

// Case-insensitive parsing.
if (Enum.TryParse<OrderStatus>("shipped", ignoreCase: true, out OrderStatus ciStatus))
{
    Console.WriteLine(ciStatus); // Shipped
}

// Enum.TryParse can also accept numeric strings that are not named members.
// Use Enum.IsDefined when only declared values are allowed for this enum.
if (Enum.IsDefined(typeof(OrderStatus), ciStatus))
{
    Console.WriteLine("Status is a declared enum value.");
}

enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered
}
```

## `ToString()` — Converting a Value to Text

All .NET types provide a `ToString()` method. The implementation inherited from `System.Object` returns the type name, while many built-in types override it to return a useful representation. Numeric and date output may be culture-sensitive.

```csharp
using System.Globalization;

int number = 42;
double pi = 3.14159;
decimal price = 19.99m;
bool flag = true;
DateTime now = new DateTime(2025, 6, 15, 14, 30, 0, DateTimeKind.Utc);

Console.WriteLine(number.ToString()); // 42
Console.WriteLine(pi.ToString(CultureInfo.InvariantCulture)); // 3.14159
Console.WriteLine(price.ToString(CultureInfo.InvariantCulture)); // 19.99
Console.WriteLine(flag.ToString()); // True
Console.WriteLine(now.ToString("yyyy-MM-dd HH:mm:ss", CultureInfo.InvariantCulture)); // 2025-06-15 14:30:00

// With format specifiers
Console.WriteLine(price.ToString("C", CultureInfo.GetCultureInfo("en-US"))); // $19.99
Console.WriteLine(pi.ToString("F2", CultureInfo.InvariantCulture));            // 3.14
Console.WriteLine(number.ToString("D5", CultureInfo.InvariantCulture));        // 00042
```

## User-Defined Conversions

You can define custom implicit and explicit conversions for your own types with the `implicit` and `explicit` operator keywords. This is an advanced feature and should be used sparingly.

```csharp
// Usage: the implicit conversion below assigns a default USD currency.
// That default is only illustrative; real money types should require a currency.
Money price = 49.99m;
Console.WriteLine(price.Amount);   // 49.99
Console.WriteLine(price.Currency); // USD

decimal rawAmount = (decimal)price; // Explicit conversion discards currency information
Console.WriteLine(rawAmount);       // 49.99

public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public static implicit operator Money(decimal amount)
    {
        return new Money(amount, "USD");
    }

    public static explicit operator decimal(Money money)
    {
        return money.Amount;
    }
}
```

The implicit conversion from `decimal` to `Money` above makes an assumption about currency; in real domain models, prefer a constructor or factory that requires the currency. Explicit conversions are usually clearer when information could be lost.

## Conversion Summary Table

| From | To | Method | Risk or note |
|---|---|---|---|
| Integral numeric type | A compatible wider numeric type | Implicit conversion | Usually preserves range; conversions to `float` or `double` can lose precision |
| Larger numeric type | Smaller numeric type | Explicit cast, such as `(int)x` | May truncate, lose precision, or overflow |
| `double`/`float` | `int` | Cast or `Convert.ToInt32` | Cast truncates; `Convert` rounds midpoint values to even |
| `string` | Numeric type | `Parse` / `TryParse` | `Parse` throws for invalid input; `TryParse` returns `false` |
| Numeric type | `string` | `ToString()` or interpolation | Specify culture for stable machine-readable output |
| `object` | Derived type | Cast, `as`, or `is` | Direct cast can throw `InvalidCastException` |
| Derived type | Base type/interface | Implicit conversion | Object remains the derived runtime type |
| `string` | Enum | `Enum.Parse` / `Enum.TryParse` | Validate allowed enum members when needed |
| Null string/object | `int` via `Convert.ToInt32` | `Convert.ToInt32(null)` | Returns 0; behavior is overload-specific |
| Any value | `string` | `ToString()` or interpolation | Output may be culture-sensitive |

## Basic Code Snippet

```csharp
// Program.cs — Type Conversion and Casting demo in .NET 10

using System.Globalization;

// --- Implicit conversions ---
int count = 100;
long bigCount = count;        // int -> long
double preciseCount = count;  // int -> double

Console.WriteLine("=== Implicit ===");
Console.WriteLine($"int: {count}, long: {bigCount}, double: {preciseCount}");

// --- Explicit conversions ---
double temperature = 98.6;
int wholeTemp = (int)temperature;                 // Truncates to 98
int roundedTemp = Convert.ToInt32(temperature);  // Rounds to 99

Console.WriteLine("\n=== Explicit ===");
Console.WriteLine($"Original: {temperature}");
Console.WriteLine($"Cast (truncate): {wholeTemp}");
Console.WriteLine($"Convert (round): {roundedTemp}");

// --- TryParse (safe parsing) ---
Console.WriteLine("\n=== TryParse ===");
string?[] inputs = { "42", "3.14", "abc", "", null };

foreach (string? input in inputs)
{
    if (int.TryParse(input, out int result))
    {
        Console.WriteLine($"'{input}' -> {result}");
    }
    else
    {
        Console.WriteLine($"'{input}' -> Failed to parse");
    }
}

// --- Convert class ---
Console.WriteLine("\n=== Convert ===");
Console.WriteLine($"Convert null to int: {Convert.ToInt32((string?)null)}");
Console.WriteLine($"Convert 'true' to bool: {Convert.ToBoolean("true")}");
Console.WriteLine($"Convert 2.5 to int: {Convert.ToInt32(2.5)}");
Console.WriteLine($"Convert 3.5 to int: {Convert.ToInt32(3.5)}");

// --- Culture-aware parsing ---
Console.WriteLine("\n=== Culture ===");
string usNumber = "1,234.56";
decimal parsed = decimal.Parse(usNumber, CultureInfo.GetCultureInfo("en-US"));
Console.WriteLine($"Parsed '{usNumber}': {parsed.ToString(CultureInfo.InvariantCulture)}");

// --- is and as operators ---
Console.WriteLine("\n=== is and as ===");
object data = "Hello, World!";

if (data is string text)
{
    Console.WriteLine($"Data is a string: {text.ToUpperInvariant()}");
}

string? asString = data as string;
int? asInt = data as int?;
Console.WriteLine($"as string: {asString ?? "null"}");
Console.WriteLine($"as int: {asInt?.ToString() ?? "null"}");
```

## Industry-Level Code Snippet

This example is a focused query-parameter parser that demonstrates safe conversions. In a typical ASP.NET Core API, prefer typed model binding and validation when they fit the request; a custom parser is useful when an application needs explicit parsing policies.

```csharp
// QueryParameterParser.cs — Query string parsing utility
using System;
using System.Collections.Generic;
using System.Globalization;
using Microsoft.AspNetCore.Http;

namespace MyBackendApp.Api.Infrastructure;

public class QueryParameterParser
{
    private readonly Dictionary<string, string> _parameters;

    public QueryParameterParser(IQueryCollection query)
    {
        _parameters = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);

        foreach (var kvp in query)
        {
            // Normalize missing or empty query values to an empty string.
            _parameters[kvp.Key] = kvp.Value.ToString().Trim();
        }
    }

    public int GetInt(string key, int defaultValue = 0)
    {
        if (!_parameters.TryGetValue(key, out string? value) ||
            string.IsNullOrWhiteSpace(value))
        {
            return defaultValue;
        }

        // Use TryParse because query-string input is untrusted.
        if (int.TryParse(value, NumberStyles.Integer, CultureInfo.InvariantCulture, out int result))
        {
            return result;
        }

        return defaultValue;
    }

    public int GetPositiveInt(string key, int defaultValue = 1)
    {
        int value = GetInt(key, defaultValue);
        return value > 0 ? value : defaultValue;
    }

    public decimal GetDecimal(string key, decimal defaultValue = 0m)
    {
        if (!_parameters.TryGetValue(key, out string? value) ||
            string.IsNullOrWhiteSpace(value))
        {
            return defaultValue;
        }

        // Permit these forms only if they are part of the API's input contract.
        NumberStyles style = NumberStyles.AllowDecimalPoint |
                             NumberStyles.AllowThousands |
                             NumberStyles.AllowCurrencySymbol |
                             NumberStyles.AllowLeadingSign;

        if (decimal.TryParse(value, style, CultureInfo.InvariantCulture, out decimal result))
        {
            return result;
        }

        return defaultValue;
    }

    public bool GetBool(string key, bool defaultValue = false)
    {
        if (!_parameters.TryGetValue(key, out string? value) ||
            string.IsNullOrWhiteSpace(value))
        {
            return defaultValue;
        }

        string normalized = value.ToLowerInvariant();
        return normalized switch
        {
            "true" or "1" or "yes" or "on" => true,
            "false" or "0" or "no" or "off" => false,
            _ => defaultValue
        };
    }

    public DateTime? GetDate(string key)
    {
        if (!_parameters.TryGetValue(key, out string? value) ||
            string.IsNullOrWhiteSpace(value))
        {
            return null;
        }

        // This API contract accepts these UTC formats.
        string[] formats =
        {
            "yyyy-MM-dd",
            "yyyy-MM-dd'T'HH:mm:ss'Z'",
            "yyyy-MM-dd'T'HH:mm:ss.fff'Z'"
        };

        DateTimeStyles styles = DateTimeStyles.AdjustToUniversal | DateTimeStyles.AssumeUniversal;
        if (DateTime.TryParseExact(value, formats, CultureInfo.InvariantCulture, styles, out DateTime result))
        {
            return result;
        }

        return null;
    }

    public TEnum? GetEnum<TEnum>(string key) where TEnum : struct, Enum
    {
        if (!_parameters.TryGetValue(key, out string? value) ||
            string.IsNullOrWhiteSpace(value))
        {
            return null;
        }

        // TryParse can accept numeric values not declared by the enum.
        if (Enum.TryParse<TEnum>(value, ignoreCase: true, out TEnum result) &&
            Enum.IsDefined(typeof(TEnum), result))
        {
            return result;
        }

        return null;
    }

    public List<int> GetIntList(string key)
    {
        if (!_parameters.TryGetValue(key, out string? value) ||
            string.IsNullOrWhiteSpace(value))
        {
            return new List<int>();
        }

        var results = new List<int>();

        // Accept comma-separated or pipe-separated values: "1,2,3" or "1|2|3".
        string[] segments = value.Split(
            new[] { ',', '|' },
            StringSplitOptions.RemoveEmptyEntries | StringSplitOptions.TrimEntries
        );

        foreach (string segment in segments)
        {
            if (int.TryParse(segment, NumberStyles.Integer,
                CultureInfo.InvariantCulture, out int parsed))
            {
                results.Add(parsed);
            }
            // This example skips invalid entries; an API may instead return a validation error.
        }

        return results;
    }
}
```

Key observations from this example:

- `TryParse` handles ordinary invalid input without throwing parsing exceptions. A production API should decide whether invalid parameters should produce a validation response or a default.
- Parsing uses `CultureInfo.InvariantCulture` because this sample's API contract defines invariant machine-readable formats.
- `NumberStyles` controls which characters are accepted. Allow currency symbols and thousands separators only when the API contract permits them.
- `DateTime.TryParseExact` enforces the listed date formats. The sample treats these inputs as UTC; APIs that accept offsets or local times should use an explicit `DateTimeOffset` policy.
- `Enum.TryParse` can accept numeric strings that are not named members, so this example also checks `Enum.IsDefined`. For `[Flags]` enums, validate combinations according to the application's rules.
- `GetIntList` skips invalid entries here as a deliberate policy; another API may reject the entire request instead.
- Defaults make this parser forgiving, but returning defaults can hide invalid input. Choose the behavior that fits the API contract.

## Key Terms Summary

| Term | Definition |
|---|---|
| Implicit conversion | A conversion the compiler can perform without an explicit cast; some numeric conversions can still lose precision. |
| Explicit conversion | A conversion written with a cast, such as `(int)value`, when the conversion is not implicit. |
| Widening conversion | An implicit conversion to a type that can represent a broader range; it does not always preserve precision. |
| Narrowing conversion | A conversion to a type with a smaller range or precision; it may lose data or overflow. |
| Truncation | Removing the fractional part without rounding; a floating-point-to-integer cast truncates toward zero. |
| Banker's rounding | Rounding to the nearest integer, with exact midpoint values rounded to the nearest even integer. |
| `Parse` | Converts a string to a type and throws when parsing fails. |
| `TryParse` | Attempts a string conversion and returns a Boolean success result instead of throwing for ordinary invalid input. |
| Boxing | Converting a value type to `object` or an interface type; may allocate an object. |
| Unboxing | Converting a boxed value back to its value type; requires an explicit cast. |
| `CultureInfo` | Provides culture-specific or invariant formatting and parsing rules. |
| Invariant culture | Culture-independent formatting/parsing rules often used for machine-readable data when required by the contract. |
