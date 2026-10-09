## Strings in `C#` - The Fundamentals
A C# `string` (`System.String`) is an immutable sequence of UTF-16 code units used to represent text. It is a reference type. A single user-perceived character can sometimes occupy more than one `char`, so `string.Length` counts UTF-16 code units rather than grapheme clusters.

### Immutability

Once a string is created, its contents cannot be changed. An operation that produces different text returns another string value; methods may return the original string when no change is needed.

```csharp
string original = "Hello";
string modified = original.ToUpperInvariant();

Console.WriteLine(original);  // Output: Hello (unchanged)
Console.WriteLine(modified);  // Output: HELLO
```

This immutability has important implications:

- **Concurrent reads:** The string object's contents cannot be mutated, so multiple threads can read the same string without locking its contents. Synchronization may still be needed if threads share and reassign the variable that refers to it.
- **Memory cost:** Operations that produce different text generally create a new string. Repeatedly growing a string in a loop can create many temporary objects and increase garbage-collection pressure.
- **String interning:** The runtime maintains an intern pool for string literals. Identical literals, including compile-time constant concatenations, can refer to the same interned object. Runtime-created strings are not automatically interned by default.

```csharp
string a = "Hello";
string b = "Hello";
Console.WriteLine(ReferenceEquals(a, b)); // Output: True (literals are interned)

string c = "Hel" + "lo"; // Compile-time constant concatenation
Console.WriteLine(ReferenceEquals(a, c)); // Output: True

string d = "Hel";
string e = d + "lo"; // Runtime concatenation; not automatically interned
Console.WriteLine(ReferenceEquals(a, e)); // Output: False
```

## String Concatenation

Concatenation joins two or more strings. The best approach depends on how many strings you are building and whether the operation repeats in a loop.

### The `+` Operator

The simplest and most readable approach for a small, fixed number of strings.

```csharp
string firstName = "Alice";
string lastName = "Smith";
string fullName = firstName + " " + lastName;

Console.WriteLine(fullName); // Output: Alice Smith
```

The compiler typically lowers string `+` operations to `string.Concat` or another optimized form. For a few strings, `+` is clear and perfectly appropriate.

### `string.Concat()`

`string.Concat` joins strings and has overloads that accept non-string values, converting them to text.

```csharp
string result = string.Concat("Order #", 12345, " is ", "Shipped");
Console.WriteLine(result); // Output: Order #12345 is Shipped
```

### Repeated Concatenation

Avoid repeatedly growing a string with `+` or `string.Concat` inside a loop. The work and temporary allocations can grow rapidly as the result gets longer. Use `StringBuilder` for repeated or large-scale construction.

```csharp
// Avoid: creates many intermediate strings as result grows.
string result = "";
for (int i = 0; i < 10000; i++)
{
    result += i.ToString();
}
```

## String Interpolation

String interpolation is a readable way to build formatted strings in modern C#. It uses the `$` prefix and lets you embed expressions inside braces.

### Basic Syntax

```csharp
string name = "Alice";
int age = 30;
decimal balance = 1250.75m;

string message = $"Customer {name}, age {age}, has a balance of {balance}.";
Console.WriteLine(message);
// Output: Customer Alice, age 30, has a balance of 1250.75.
```

Depending on the context and target framework, the compiler may use `string.Format`, `string.Concat`, or an optimized interpolated-string handler. Do not assume every interpolated string becomes a `string.Format` call.

### Format Specifiers

Format specifiers inside interpolation braces control how values are displayed. Numeric and date formatting usually uses the current culture unless you specify otherwise, so example output can vary by machine or application settings.

```csharp
decimal price = 49.99m;
double percentage = 0.856;
DateTime orderDate = new DateTime(2025, 3, 15, 14, 30, 0);
int orderId = 42;

// Currency format (example output assumes en-US culture)
Console.WriteLine($"Price: {price:C}");          // Example: Price: $49.99

// Percentage format
Console.WriteLine($"Discount: {percentage:P1}"); // Example: Discount: 85.6%

// Date formats
Console.WriteLine($"Date: {orderDate:yyyy-MM-dd}");      // 2025-03-15
Console.WriteLine($"Date: {orderDate:MMMM dd, yyyy}");   // March 15, 2025 (English culture)
Console.WriteLine($"Time: {orderDate:HH:mm:ss}");        // 14:30:00

// Padding and alignment (positive = right-align, negative = left-align)
Console.WriteLine($"Order: {orderId,5}");   // Right-aligned in 5 characters
Console.WriteLine($"Order: {orderId,-5}");  // Left-aligned in 5 characters

// Numeric formats (example output assumes en-US culture)
Console.WriteLine($"Hex: {255:X}");         // Hex: FF
Console.WriteLine($"Decimal: {1234567:N}"); // Decimal: 1,234,567.00
Console.WriteLine($"Zeros: {42:D5}");       // Zeros: 00042
```

### Expressions Inside Interpolation

You can put any valid C# expression inside the braces, not just variable names.

```csharp
int quantity = 5;
decimal unitPrice = 19.99m;

Console.WriteLine($"Total: {quantity * unitPrice:C}");
// Example output (en-US): Total: $99.95

string status = "active";
Console.WriteLine($"Status: {status.ToUpperInvariant()}");
// Output: Status: ACTIVE

int[] scores = { 85, 92, 78 };
Console.WriteLine($"Average: {scores.Average():F1}");
// Output: Average: 85.0
```

### Escaping Curly Braces

To include literal curly braces in an interpolated string, double them.

```csharp
string name = "Alice";
int age = 30;
string json = $"{{ \"name\": \"{name}\", \"age\": {age} }}";
Console.WriteLine(json);
// Output: { "name": "Alice", "age": 30 }
```

## `string.Format()`

`string.Format` is another way to format strings. It uses numbered placeholders (`{0}`, `{1}`, and so on) instead of inline expressions, and you will encounter it in existing codebases and resource files.

```csharp
string name = "Alice";
int age = 30;
DateTime registeredOn = new DateTime(2025, 6, 15);

string message = string.Format(
    "Customer {0}, age {1}, registered on {2:yyyy-MM-dd}.",
    name, age, registeredOn);

Console.WriteLine(message);
// Output: Customer Alice, age 30, registered on 2025-06-15.
```

For new code, interpolation is usually easier to read. `string.Format` remains useful when a format string is stored separately, such as in localization resources. For structured logging, use the logging framework's message-template syntax (for example, `Log.Information("User {UserId}", userId)`) rather than interpolating values into the message.

## Verbatim Strings

A verbatim string is prefixed with `@` and treats backslashes as literal characters instead of escape sequences. This is useful for file paths and multi-line text.

```csharp
// Without verbatim syntax, backslashes must be escaped.
string path1 = "C:\\Users\\Alice\\Documents\\file.txt";

// With verbatim syntax, backslashes are literal.
string path2 = @"C:\Users\Alice\Documents\file.txt";

Console.WriteLine(path1 == path2); // Output: True

// Multi-line verbatim string
string sql = @"
    SELECT o.Id, o.Total, c.Name
    FROM Orders o
    INNER JOIN Customers c ON o.CustomerId = c.Id
    WHERE o.Status = 'Shipped'
    ORDER BY o.CreatedAt DESC";

Console.WriteLine(sql);
```

In a verbatim string, use two double quotes (`""`) to represent one quote character.

```csharp
string quote = @"She said, ""Hello, World!""";
Console.WriteLine(quote); // Output: She said, "Hello, World!"
```

## Interpolated Verbatim Strings

You can combine interpolation and verbatim behavior. Both `$@"..."` and `@$"..."` are valid; this example uses `$@`.

```csharp
string tableName = "Orders";
int limit = 100;

string query = $@"
    SELECT *
    FROM {tableName}
    WHERE Status = 'Active'
    ORDER BY CreatedAt DESC
    LIMIT {limit}";

Console.WriteLine(query);
```

Do not use interpolated values to build executable SQL from untrusted input. Use parameterized queries for values and allowlist any dynamic table or column identifiers.

## Raw String Literals (C# 11+, supported by .NET 10)

Raw string literals use three or more double-quote characters (`"""`). They are useful for multi-line content such as JSON, XML, and SQL because quotes and backslashes usually do not need escaping.

```csharp
string name = "Alice";
int age = 30;

// With two $ signs, interpolation uses double braces so JSON braces stay literal.
string json = $$"""
    {
        "name": "{{name}}",
        "age": {{age}},
        "isActive": true
    }
    """;

Console.WriteLine(json);
```

Output:

```text
{
    "name": "Alice",
    "age": 30,
    "isActive": true
}
```

Key rules for raw string literals:

- The indentation of the closing quotes determines the indentation removed from each content line.
- The number of `$` signs determines the number of braces needed to begin an interpolation. With `$$`, write `{{expression}}`; single braces remain literal.
- Use `$` when single-brace interpolation is sufficient and the content does not need literal single braces.

```csharp
// Single $ — interpolation uses single braces.
string simple = $"""
    Hello, {name}!
    You are {age} years old.
    """;

// Double $$ — literal JSON braces remain single; interpolation uses double braces.
string withBraces = $$"""
    {
        "user": "{{name}}",
        "settings": {
            "theme": "dark"
        }
    }
    """;
```

## StringBuilder

`StringBuilder` (`System.Text`) is mutable: it builds text in an internal buffer and is useful when appending many pieces or building a string incrementally. It can avoid creating a new immutable string for every append, though the buffer itself may grow and allocate.

### Basic Usage

```csharp
using System.Text;

StringBuilder sb = new StringBuilder();

sb.Append("Hello");
sb.Append(" ");
sb.Append("World");
sb.AppendLine("!");  // Appends text plus a newline
sb.AppendLine("This is a new line.");

string result = sb.ToString();
Console.WriteLine(result);
```

Output:

```text
Hello World!
This is a new line.
```

### Chaining Methods

Most `StringBuilder` methods return the builder itself, which allows method chaining.

```csharp
string csv = new StringBuilder()
    .AppendLine("Id,Name,Email,Status")
    .AppendLine("1,Alice,alice@example.com,Active")
    .AppendLine("2,Bob,bob@example.com,Inactive")
    .AppendLine("3,Charlie,charlie@example.com,Active")
    .ToString();

Console.WriteLine(csv);
```

### Capacity and Performance

A default `StringBuilder` starts with a small capacity (commonly 16 characters). When more space is needed, it grows its internal buffer; growing may require allocating a larger buffer and copying existing content. If you can estimate the final size, an initial capacity can reduce resizing.

```csharp
// If you expect roughly 10,000 characters:
StringBuilder sb = new StringBuilder(10000);

for (int i = 0; i < 1000; i++)
{
    sb.Append($"Item {i}: {i * 10}\n");
}

Console.WriteLine($"Final length: {sb.Length}");
Console.WriteLine($"Capacity: {sb.Capacity}");
```

### When to Use StringBuilder vs. Interpolation

- **Interpolation (`$"..."`):** Use for a small, fixed number of values or concatenations. The compiler/runtime can optimize these cases.
- **`StringBuilder`:** Use when repeatedly growing a string in a loop, building large text incrementally, or when the number of appends is not known in advance.

For performance-critical code, measure with representative data rather than relying on a fixed concatenation-count rule.

```csharp
// Good: interpolation for a simple, fixed-size message.
string greeting = $"Hello, {name}. Your order #{orderId} is ready.";

// Good: StringBuilder for repeated output in a loop.
StringBuilder report = new StringBuilder();
foreach (var order in orders)
{
    report.AppendLine($"Order #{order.Id}: {order.Total:C}");
}
```

## Common String Methods

The `string` class provides methods for inspecting, searching, and transforming text.

### Checking for Null or Empty

```csharp
string? input1 = null;
string input2 = "";
string input3 = "   ";
string input4 = "Hello";

// IsNullOrEmpty checks for null or "".
Console.WriteLine(string.IsNullOrEmpty(input1)); // True
Console.WriteLine(string.IsNullOrEmpty(input2)); // True
Console.WriteLine(string.IsNullOrEmpty(input3)); // False (whitespace is not empty)
Console.WriteLine(string.IsNullOrEmpty(input4)); // False

// IsNullOrWhiteSpace checks for null, "", or whitespace only.
Console.WriteLine(string.IsNullOrWhiteSpace(input1)); // True
Console.WriteLine(string.IsNullOrWhiteSpace(input2)); // True
Console.WriteLine(string.IsNullOrWhiteSpace(input3)); // True
Console.WriteLine(string.IsNullOrWhiteSpace(input4)); // False
```

Use `IsNullOrWhiteSpace` when a field must contain non-whitespace text. Choose validation based on the field's requirements; not every input should be treated the same way.

### Searching

```csharp
string text = "The quick brown fox jumps over the lazy dog";

// Contains: checks whether a substring exists.
Console.WriteLine(text.Contains("fox")); // True
Console.WriteLine(text.Contains("cat")); // False
Console.WriteLine(text.Contains("FOX")); // False (case-sensitive by default)
Console.WriteLine(text.Contains("FOX", StringComparison.OrdinalIgnoreCase)); // True

// Specify ordinal comparison for predictable identifier/text checks.
Console.WriteLine(text.StartsWith("The", StringComparison.Ordinal)); // True
Console.WriteLine(text.EndsWith("dog", StringComparison.Ordinal)); // True

// IndexOf: zero-based position of the first occurrence, or -1 if not found.
int position = text.IndexOf("fox", StringComparison.Ordinal);
Console.WriteLine(position); // 16

int notFound = text.IndexOf("cat", StringComparison.Ordinal);
Console.WriteLine(notFound); // -1

// LastIndexOf: searches from the end.
int lastO = text.LastIndexOf("o", StringComparison.Ordinal);
Console.WriteLine(lastO); // 41
```

For identifiers and other non-linguistic comparisons, pass a `StringComparison` value explicitly so the behavior does not depend on the current culture.

### Extracting Substrings

```csharp
string fullName = "Alice Smith";

// Substring(startIndex, length)
string firstName = fullName.Substring(0, 5);
Console.WriteLine(firstName); // Alice

// Substring(startIndex) — from index to end
string lastName = fullName.Substring(6);
Console.WriteLine(lastName); // Smith

// Index and Range operators (C# 8+)
string first = fullName[..5];    // Same as Substring(0, 5)
string last = fullName[6..];     // Same as Substring(6)
string middle = fullName[2..7];  // "ice S"
string fromEnd = fullName[^5..]; // Last 5 UTF-16 code units: "Smith"
```

### Replacing and Removing

```csharp
string original = "Hello World";

// Replace all occurrences of a substring.
string replaced = original.Replace("World", "C#");
Console.WriteLine(replaced); // Hello C#

// Replace is case-sensitive.
string noChange = original.Replace("world", "C#");
Console.WriteLine(noChange); // Hello World (no match)

// Remove characters starting at an index.
string removed = original.Remove(5);      // Removes from index 5 to end
Console.WriteLine(removed); // Hello

string removedMiddle = original.Remove(5, 6); // Removes 6 chars starting at index 5
Console.WriteLine(removedMiddle); // Hello
```

### Splitting and Joining

```csharp
// Split: breaks a string into an array of substrings.
string csvLine = "Alice,30,alice@example.com,Active";
string[] fields = csvLine.Split(',');

Console.WriteLine(fields[0]); // Alice
Console.WriteLine(fields[1]); // 30
Console.WriteLine(fields[2]); // alice@example.com
Console.WriteLine(fields[3]); // Active

// Split with multiple delimiters.
string messyInput = "one;two,three|four";
string[] parts = messyInput.Split(new char[] { ';', ',', '|' });
Console.WriteLine(parts.Length); // 4

// Remove empty entries when splitting a path.
string path = "/home//user///documents/";
string[] segments = path.Split('/', StringSplitOptions.RemoveEmptyEntries);
// Result: ["home", "user", "documents"]

// Join combines values into a single string with a separator.
string[] names = { "Alice", "Bob", "Charlie" };
string joined = string.Join(", ", names);
Console.WriteLine(joined); // Alice, Bob, Charlie

// Join with LINQ.
int[] ids = { 1, 2, 3, 4, 5 };
string idList = string.Join(", ", ids.Select(id => $"#{id}"));
Console.WriteLine(idList); // #1, #2, #3, #4, #5
```

### Trimming

```csharp
string padded = "   Hello World   ";

Console.WriteLine($"[{padded.Trim()}]");       // [Hello World]
Console.WriteLine($"[{padded.TrimStart()}]");  // [Hello World   ]
Console.WriteLine($"[{padded.TrimEnd()}]");    // [   Hello World]

// Trim specific characters.
string quoted = "\"Hello\"";
Console.WriteLine(quoted.Trim('"')); // Hello

string dashed = "---section---";
Console.WriteLine(dashed.Trim('-')); // section
```

### Case Conversion

```csharp
string mixed = "Hello World";

Console.WriteLine(mixed.ToUpper());          // HELLO WORLD (current culture)
Console.WriteLine(mixed.ToLower());          // hello world (current culture)
Console.WriteLine(mixed.ToUpperInvariant()); // HELLO WORLD (culture-independent)
Console.WriteLine(mixed.ToLowerInvariant()); // hello world (culture-independent)
```

For protocol tokens and internal identifiers, use ordinal comparisons or invariant normalization as appropriate. Culture-sensitive casing can produce unexpected results in some locales (for example, Turkish casing of `i`). For end-user display, use culture-aware formatting when that is the intended behavior.

### Padding

```csharp
string number = "42";

Console.WriteLine(number.PadLeft(5));      // "   42" (padded with spaces)
Console.WriteLine(number.PadLeft(5, '0')); // "00042" (padded with zeros)
Console.WriteLine(number.PadRight(5, '-')); // "42---"
```

## String Comparison

Comparing strings correctly matters in backend code, especially for authentication, authorization, and data lookup.

### The `==` Operator and `Equals`

`string` overloads `==` to compare text content. The usual string equality methods perform a case-sensitive ordinal comparison unless you choose another comparison option.

```csharp
string a = "Hello";
string b = "hello";

Console.WriteLine(a == b);      // False
Console.WriteLine(a.Equals(b)); // False
```

### The `StringComparison` Enum

The `StringComparison` enum lets you specify how strings are compared.

| Value | Case-sensitive? | Culture-aware? | Typical use |
|---|---|---|---|
| `Ordinal` | Yes | No | Identifiers, protocol tokens, and exact backend comparisons |
| `OrdinalIgnoreCase` | No | No | Case-insensitive identifiers and protocol values; apply an explicit policy for email addresses |
| `CurrentCulture` | Yes | Yes | Linguistic comparison or sorting for the current user's locale |
| `CurrentCultureIgnoreCase` | No | Yes | Case-insensitive linguistic comparison for the current user's locale |
| `InvariantCulture` | Yes | Invariant linguistic rules | Culture-invariant linguistic comparison; rarely appropriate for identifiers |
| `InvariantCultureIgnoreCase` | No | Invariant linguistic rules | Case-insensitive invariant linguistic comparison; rarely appropriate for identifiers |

```csharp
string input = "ADMIN";
string expected = "admin";

// Ordinal (case-sensitive)
Console.WriteLine(string.Equals(input, expected, StringComparison.Ordinal));
// Output: False

// OrdinalIgnoreCase — common for case-insensitive backend tokens.
Console.WriteLine(string.Equals(input, expected, StringComparison.OrdinalIgnoreCase));
// Output: True

// Compare returns a negative value, zero, or a positive value.
int result = string.Compare(input, expected, StringComparison.OrdinalIgnoreCase);
Console.WriteLine(result); // Output: 0 (equal)
```

Rule of thumb: use `StringComparison.Ordinal` or `StringComparison.OrdinalIgnoreCase` for most backend identifiers and protocol values. Use culture-aware comparisons when linguistic rules are needed for user-facing text.

## Basic Code Snippet

```csharp
// Program.cs — String Handling demo in .NET 10

using System.Text;

// --- Interpolation ---
string productName = "Wireless Keyboard";
decimal price = 79.99m;
int stock = 142;

string display = $"Product: {productName} | Price: {price:C} | Stock: {stock}";
Console.WriteLine(display);

// --- Verbatim and interpolated strings ---
string sqlQuery = $@"
    SELECT Id, Name, Email
    FROM Customers
    WHERE Status = 'Active'
    AND CreatedAt >= '{DateTime.UtcNow.AddDays(-30):yyyy-MM-dd}'";

Console.WriteLine("SQL Query:");
Console.WriteLine(sqlQuery);

// --- StringBuilder in a loop ---
var items = new List<string> { "Laptop", "Mouse", "Monitor", "Keyboard" };
var sb = new StringBuilder();

sb.AppendLine("=== Order Summary ===");
for (int i = 0; i < items.Count; i++)
{
    sb.AppendLine($"  {i + 1}. {items[i]}");
}
sb.AppendLine($"Total items: {items.Count}");

Console.WriteLine(sb.ToString());

// --- Common methods ---
string email = "  Alice.Smith@Example.COM  ";

string trimmed = email.Trim();
string lower = trimmed.ToLowerInvariant();
bool isValidDomain = lower.EndsWith("@example.com", StringComparison.Ordinal);
string username = lower.Split('@')[0];

Console.WriteLine($"Trimmed: [{trimmed}]");
Console.WriteLine($"Lowercase: {lower}");
Console.WriteLine($"Valid domain: {isValidDomain}");
Console.WriteLine($"Username: {username}");

// --- Null checks ---
string? userInput = "   ";
bool isEmpty = string.IsNullOrWhiteSpace(userInput);
Console.WriteLine($"Input is empty/whitespace: {isEmpty}"); // True

// --- String comparison ---
string role1 = "Admin";
string role2 = "admin";
bool match = string.Equals(role1, role2, StringComparison.OrdinalIgnoreCase);
Console.WriteLine($"Roles match (ignore case): {match}"); // True
```

Do not use interpolated strings to construct SQL commands from untrusted input. Use parameterized database APIs; this sample's SQL text is only for demonstrating string formatting.

## Industry-Level Code Snippet

This example demonstrates string methods used in request processing and normalization. It is illustrative rather than a complete email, phone-number, or security validator. Production systems should use suitable validation libraries and context-appropriate output encoding; sanitization alone is not a security boundary.

```csharp
// InputSanitizer.cs — Illustrative input validation and normalization service
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace MyBackendApp.Core.Services;

public class InputSanitizer
{
    private static readonly HashSet<string> BlockedDomains = new(StringComparer.OrdinalIgnoreCase)
    {
        "tempmail.com",
        "throwaway.email",
        "guerrillamail.com",
        "mailinator.com"
    };

    private static readonly char[] CsvDelimiters = { ',', ';', '\t' };

    public SanitizationResult SanitizeUserRegistration(RegistrationRequest request)
    {
        var errors = new List<string>();

        // --- Email normalization ---
        if (string.IsNullOrWhiteSpace(request.Email))
        {
            errors.Add("Email is required.");
        }
        else
        {
            // This simple shape check is not a complete email validator.
            string email = request.Email.Trim().ToLowerInvariant();

            if (!email.Contains('@') || !email.Contains('.'))
            {
                errors.Add("Email format is invalid.");
            }
            else
            {
                string domain = email.Split('@')[1];

                if (BlockedDomains.Contains(domain))
                {
                    errors.Add($"Email domain '{domain}' is not allowed.");
                }
            }

            request.Email = email;
        }

        // --- Username normalization ---
        if (string.IsNullOrWhiteSpace(request.Username))
        {
            errors.Add("Username is required.");
        }
        else
        {
            string username = request.Username.Trim();

            // Keep letters, digits, underscores, and hyphens.
            var cleanChars = new StringBuilder(username.Length);
            foreach (char c in username)
            {
                if (char.IsLetterOrDigit(c) || c == '_' || c == '-')
                {
                    cleanChars.Append(c);
                }
            }

            string sanitized = cleanChars.ToString();

            if (sanitized.Length < 3)
            {
                errors.Add("Username must be at least 3 characters after sanitization.");
            }
            else if (sanitized.Length > 50)
            {
                errors.Add("Username must not exceed 50 characters.");
            }

            request.Username = sanitized.ToLowerInvariant();
        }

        // --- Name normalization ---
        if (!string.IsNullOrWhiteSpace(request.FirstName))
        {
            request.FirstName = request.FirstName.Trim();

            // Simple capitalization example; real name rules vary by language.
            if (request.FirstName.Length > 0)
            {
                request.FirstName = string.Concat(
                    request.FirstName[..1].ToUpperInvariant(),
                    request.FirstName[1..].ToLowerInvariant()
                );
            }
        }

        // --- Phone number normalization ---
        if (!string.IsNullOrWhiteSpace(request.PhoneNumber))
        {
            // Keep ASCII digits for this simplified example.
            string digits = new string(
                request.PhoneNumber.Where(char.IsAsciiDigit).ToArray()
            );

            if (digits.Length < 10 || digits.Length > 15)
            {
                errors.Add("Phone number must contain between 10 and 15 digits.");
            }
            else
            {
                // Simplified US-centric formatting; this is not a general E.164 parser.
                request.PhoneNumber = digits.StartsWith('1') && digits.Length == 11
                    ? $"+{digits}"
                    : $"+1{digits}";
            }
        }

        // --- Address normalization ---
        if (!string.IsNullOrWhiteSpace(request.Address))
        {
            // Normalize repeated ASCII spaces.
            string[] words = request.Address.Split(
                ' ',
                StringSplitOptions.RemoveEmptyEntries | StringSplitOptions.TrimEntries
            );
            request.Address = string.Join(" ", words);
        }

        return new SanitizationResult
        {
            IsValid = errors.Count == 0,
            Errors = errors,
            SanitizedRequest = request
        };
    }

    public List<string> ParseCsvLine(string csvLine)
    {
        if (string.IsNullOrWhiteSpace(csvLine))
        {
            return new List<string>();
        }

        return csvLine
            .Split(CsvDelimiters, StringSplitOptions.TrimEntries)
            .Where(field => !string.IsNullOrEmpty(field))
            .Select(field => field.Trim('"'))
            .ToList();
    }

    public string GenerateSlug(string title)
    {
        if (string.IsNullOrWhiteSpace(title))
        {
            return string.Empty;
        }

        string lower = title.ToLowerInvariant().Trim();

        var slug = new StringBuilder(lower.Length);
        bool lastWasDash = false;

        foreach (char c in lower)
        {
            if (char.IsLetterOrDigit(c))
            {
                slug.Append(c);
                lastWasDash = false;
            }
            else if (!lastWasDash && slug.Length > 0)
            {
                slug.Append('-');
                lastWasDash = true;
            }
        }

        return slug.ToString().TrimEnd('-');
    }
}

public class RegistrationRequest
{
    public string Email { get; set; } = string.Empty;
    public string Username { get; set; } = string.Empty;
    public string? FirstName { get; set; }
    public string? LastName { get; set; }
    public string? PhoneNumber { get; set; }
    public string? Address { get; set; }
}

public class SanitizationResult
{
    public bool IsValid { get; set; }
    public List<string> Errors { get; set; } = new();
    public RegistrationRequest SanitizedRequest { get; set; } = new();
}
```

Key observations from this example:

- `string.IsNullOrWhiteSpace()` checks for null, empty, or whitespace-only input. Use validation that matches each field's requirements.
- `ToLowerInvariant()` applies culture-independent casing, but email normalization should follow the application's explicit identity policy.
- `StringComparer.OrdinalIgnoreCase` makes the blocked-domain lookup case-insensitive.
- `StringBuilder` is useful for incrementally constructing usernames and slugs.
- `StringSplitOptions.TrimEntries` (available since .NET 5) trims entries during splitting; it is available in .NET 10.
- Range operators (`[..1]` and `[1..]`) are used to split the first character from the rest of the name.
- The phone-number example is US-centric and simplified. Use a region-aware phone-number library for international validation and E.164 formatting.
- `string.Concat` joins the two name segments here; choose it or `+` for clarity rather than assuming a performance difference.
- Use parameterized database access and encode text for its output context; string sanitization alone does not prevent injection or cross-site scripting.

## Key Terms Summary

| Term | Definition |
|---|---|
| Immutable | A string's contents cannot be changed after creation; operations that produce different text return another string. |
| Interpolation | The `$"..."` syntax that embeds expressions inside a string literal. |
| Verbatim string | A string prefixed with `@` that treats backslashes as literal characters. |
| Raw string literal | A string delimited by three or more quotes that reduces escaping for multi-line or quote-heavy text. |
| `StringBuilder` | A mutable text builder in `System.Text`, useful for repeated or incremental appends. |
| String interning | The runtime's sharing of identical interned string literals. |
| Ordinal comparison | A culture-independent comparison of UTF-16 code units. |
| Format specifier | A code in interpolation or formatting placeholders that controls display, such as `C` for currency, `P1` for percentage, or `yyyy-MM-dd` for a date. |
| `TrimEntries` | A `StringSplitOptions` flag that trims whitespace from each split result. |
