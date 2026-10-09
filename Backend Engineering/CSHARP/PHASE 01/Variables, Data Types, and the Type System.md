## What Is a Variable?
A variable is a named storage location for a value. In C#, every variable has a type. The type determines which values are valid and which operations you can perform; it also influences how the value is represented in memory.

C# is a statically typed language. The type of a variable is determined at compile time and does not change during execution. Once you declare a variable as an `int`, it remains an `int`.

## Declaring and Initializing Variables

```csharp
// Declaration: specify the type and the name.
int age;

// Initialization: assign a value.
age = 25;

// Declaration and initialization in one line (most common).
string name = "Alice";

// Multiple variables of the same type on one line.
int x = 1, y = 2, z = 3;
```

## Naming Conventions

C# follows naming conventions recommended by the community and commonly enforced by IDE analyzers:

- **Local variables and parameters:** `camelCase`, such as `firstName`, `orderCount`, and `isActive`.
- **Constants:** Usually `PascalCase`; some codebases use `UPPER_SNAKE_CASE`. Examples: `MaxRetryCount` or `MAX_RETRY_COUNT`.
- **Private fields:** Often `_camelCase` with an underscore prefix, such as `_connectionString` and `_logger`.
- **Public fields, properties, methods, and classes:** `PascalCase`, such as `CustomerName` and `GetOrderById`.

Variable names must start with a letter or underscore. They cannot be C# reserved keywords (such as `int`, `class`, or `return`) unless prefixed with `@`, though that is discouraged.

```csharp
int validName = 10;
int _privateField = 20;
int @class = 30; // Valid, but avoid this.
```

## The C# Type System Overview

C# types are commonly grouped into three categories:

- **Value types:** Store their value directly. Assignment copies the value.
- **Reference types:** Variables hold references to objects. Assignment copies the reference; it does not automatically copy the object.
- **Pointer types:** Store unmanaged memory addresses. They can be used only in `unsafe` code and are uncommon in typical backend development.

For class types, `System.Object` is the base type. Value types derive from `System.ValueType` (or `System.Enum`), which ultimately derives from `System.Object`. Interfaces and pointer types are not classes in that inheritance hierarchy.

## Value Types

Value types hold their data directly. Assigning one value-type variable to another copies its fields, so changing a simple value in one variable does not change the other. If a struct contains reference-type fields, those references are copied too, so both struct values can still refer to the same objects.

A value type is **not always stored on the stack**. Its storage location depends on context: a value can be a local, a field inside another object, an array element, or a boxed value. The runtime may also optimize where values are stored.

### Built-in Value Types

The default-value column shows `default(T)` values. Local variables must still be definitely assigned before they are read; C# does not automatically initialize every local variable for you.

| C# Keyword | .NET Type | Size | Range | Default value |
|---|---|---:|---|---|
| `bool` | `System.Boolean` | 1 byte | `true` or `false` | `false` |
| `byte` | `System.Byte` | 1 byte | 0 to 255 | 0 |
| `sbyte` | `System.SByte` | 1 byte | -128 to 127 | 0 |
| `short` | `System.Int16` | 2 bytes | -32,768 to 32,767 | 0 |
| `ushort` | `System.UInt16` | 2 bytes | 0 to 65,535 | 0 |
| `int` | `System.Int32` | 4 bytes | -2,147,483,648 to 2,147,483,647 | 0 |
| `uint` | `System.UInt32` | 4 bytes | 0 to 4,294,967,295 | 0 |
| `long` | `System.Int64` | 8 bytes | approximately -9.22 × 10^18 to 9.22 × 10^18 | 0 |
| `ulong` | `System.UInt64` | 8 bytes | 0 to approximately 18.44 × 10^18 | 0 |
| `float` | `System.Single` | 4 bytes | approximately ±1.5 × 10^-45 to 3.4 × 10^38 | 0.0 |
| `double` | `System.Double` | 8 bytes | approximately ±5.0 × 10^-324 to 1.7 × 10^308 | 0.0 |
| `decimal` | `System.Decimal` | 16 bytes | approximately ±1.0 × 10^-28 to 7.9 × 10^28 | 0.0 |
| `char` | `System.Char` | 2 bytes | U+0000 to U+FFFF (a UTF-16 code unit) | `\0` |

### Integer Types

```csharp
byte smallNumber = 255;
short mediumNumber = 32000;
int standardNumber = 2_147_483_647; // Underscores for readability (C# 7+)
long bigNumber = 9_223_372_036_854_775_807L; // L suffix for long

// Unsigned types cannot represent negative values.
uint positiveOnly = 4_000_000_000U; // U suffix for uint
ulong veryLarge = 18_000_000_000_000_000_000UL; // UL suffix for ulong
```

### Floating-Point Types

```csharp
float singlePrecision = 3.14f;   // f suffix is required
double doublePrecision = 3.141592653589793; // double is the default for real-number literals
decimal money = 19.99m;          // m suffix is required
```

When to use which:

- **`float`:** Scientific calculations where lower precision and memory use are acceptable. Rarely used in backend APIs.
- **`double`:** General-purpose floating-point math and scientific computations.
- **`decimal`:** Often used for financial amounts when base-10 decimal precision is useful. It still has finite precision, so define appropriate rounding rules. Some systems instead store currency in integer minor units.

```csharp
// Demonstrating floating-point representation for money
double price1 = 0.1;
double price2 = 0.2;
double total = price1 + price2;
Console.WriteLine(total == 0.3); // False: binary floating-point rounding

decimal cost1 = 0.1m;
decimal cost2 = 0.2m;
decimal totalCost = cost1 + cost2;
Console.WriteLine(totalCost == 0.3m); // True
```

### Boolean and Char

```csharp
bool isActive = true;
bool isDeleted = false;

char grade = 'A';
char newline = '\n';
char unicodeChar = '\u0041'; // Same as 'A'
```

### Enums (Value Types)

Enums are value types that represent a set of named constants.

```csharp
OrderStatus currentStatus = OrderStatus.Processing;
Console.WriteLine(currentStatus); // Output: Processing
Console.WriteLine((int)currentStatus); // Output: 1 (zero-based by default)

enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}
```

### Structs (Value Types)

Structs are user-defined value types. They are covered in detail in topic 1.14; here is a brief introduction.

```csharp
Point p1 = new Point { X = 10.0, Y = 20.0 };
Point p2 = p1; // p2 is a copy of the struct's fields
p2.X = 99.0;
Console.WriteLine(p1.X); // Output: 10 (p1 is unchanged)

struct Point
{
    public double X;
    public double Y;
}
```

## Reference Types

A reference-type variable holds a reference to an object. Assigning the variable to another variable copies that reference, so both can refer to the same object. Objects are generally allocated on the managed heap, while the reference variable itself can be stored in different locations depending on context.

### Built-in Reference Types

| C# Keyword | .NET Type | Description |
|---|---|---|
| `string` | `System.String` | An immutable sequence of UTF-16 code units |
| `object` | `System.Object` | The base type for class types; value types can be boxed to it |
| `dynamic` | Runtime-bound | A keyword that defers member binding and many type checks until runtime; it is not a separate runtime type |

### String

Strings are reference types, but they can feel like value types because they are immutable. Once a string is created, it cannot be changed. An operation that appears to modify a string creates a new string value instead.

```csharp
string firstName = "Alice";
string lastName = "Smith";
string fullName = firstName + " " + lastName; // Creates a new string value

// String interpolation (preferred over concatenation)
string greeting = $"Hello, {fullName}. You are {30} years old.";

// Verbatim strings (ignore most escape sequences, useful for file paths)
string filePath = @"C:\Users\Alice\Documents\file.txt";

// Raw string literals (C# 11+, supported by .NET 10)
string json = """
    {
        "name": "Alice",
        "age": 30
    }
    """;

Console.WriteLine(greeting);
Console.WriteLine(filePath);
Console.WriteLine(json);
```

### Object

`object` (`System.Object`) is the base type of class types. Value types can be converted to `object` through boxing.

```csharp
object number = 42;       // Boxing: value type converted to object
object text = "Hello";    // Reference type assigned to object
object flag = true;       // Boxing again

// Cast back to the original type to use it as an int.
int unwrapped = (int)number;
```

### Dynamic

`dynamic` tells the compiler to defer certain type checks and member binding until runtime. It is rarely needed in backend code and is generally best avoided when static typing can be used.

```csharp
dynamic data = "Hello";
Console.WriteLine(data.Length); // Works: outputs 5

data = 42;
// Console.WriteLine(data.Length); // Runtime binder error: int has no Length property
```

### Arrays (Reference Types)

Arrays are reference types, even when they contain value types.

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
string[] names = new string[3];
names[0] = "Alice";
names[1] = "Bob";
names[2] = "Charlie";
```

### Classes (Reference Types)

Classes are the most common reference type. They are covered in depth in Phase 2.

```csharp
Customer customer1 = new Customer { Name = "Alice", Email = "alice@example.com" };
Customer customer2 = customer1; // Both variables refer to the same object
customer2.Name = "Bob";
Console.WriteLine(customer1.Name); // Output: Bob (customer1 sees the object's change)

class Customer
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}
```

## Value Types vs Reference Types — The Critical Difference

Understanding assignment and storage helps prevent bugs in backend code.

### Memory Allocation

- **Stack:** A thread's stack stores call frames and may hold some local values. The runtime and JIT compiler decide where values are stored; a value type is not guaranteed to be on the stack.
- **Managed heap:** Objects are generally allocated on the managed heap and reclaimed by the garbage collector when they are no longer reachable. A reference variable is not itself necessarily stored on the heap.

### Assignment Behavior

```csharp
// VALUE TYPE: assignment copies the value
int a = 10;
int b = a;
b = 20;
Console.WriteLine(a); // Output: 10 (a is unchanged)

// REFERENCE TYPE: assignment copies the reference, not the object
var list1 = new List<int> { 1, 2, 3 };
var list2 = list1;
list2.Add(4);
Console.WriteLine(list1.Count); // Output: 4 (list1 refers to the same list)
```

### Method Parameter Behavior

C# passes arguments by value by default. For a value-type argument, the value is copied. For a reference-type argument, the reference is copied, so a method can mutate the referenced object but reassigning the parameter does not reassign the caller's variable.

```csharp
void ModifyValue(int number)
{
    number = 100; // Modifies the local copy only
}

void ModifyReference(List<int> items)
{
    items.Add(999); // Mutates the object referenced by the caller's variable
}

int myNumber = 5;
ModifyValue(myNumber);
Console.WriteLine(myNumber); // Output: 5 (unchanged)

var myList = new List<int> { 1, 2 };
ModifyReference(myList);
Console.WriteLine(myList.Count); // Output: 3 (modified)
```

### Summary Table

| Characteristic | Value Types | Reference Types |
|---|---|---|
| Storage | Depends on context; can be local, inline in an object or array, or boxed | Object is generally on the managed heap; the reference can be stored elsewhere |
| Assignment | Copies the value's fields | Copies the reference; both variables can refer to the same object |
| Default value | Zero-initialized for `default(T)`, fields, and array elements; locals must be assigned before use | `null` for `default(T)`; nullable-reference annotations affect compiler analysis |
| Examples | `int`, `double`, `bool`, `char`, `struct`, `enum` | `string`, `class`, array, delegate |
| Can be null? | No, unless wrapped in `Nullable<T>` or written as `T?` | Yes; nullable reference types provide compile-time analysis |
| Garbage collected? | Not individually; may be stored inside a garbage-collected object or boxed | Objects are managed by the garbage collector |
| Inheritance | Structs cannot inherit from another class or struct, but can implement interfaces | Classes support class inheritance; interfaces and other reference types have their own rules |

## Type Inference with `var`

The `var` keyword tells the compiler to infer the type from the right-hand side of an assignment. The variable is still statically typed; the compiler determines its type at compile time.

```csharp
var count = 10;           // Compiler infers int
var price = 19.99m;       // Compiler infers decimal
var name = "Alice";       // Compiler infers string
var isActive = true;      // Compiler infers bool
var customers = new List<Customer>(); // Compiler infers List<Customer>
```

Rules for `var`:

- You must initialize the variable on the same line. `var x;` is a compile error.
- The type is fixed at compile time. You cannot reassign a `var` variable to a different type.
- `var` is a convenience, not a dynamic type. The compiler knows the exact type.

Industry convention: use `var` when the type is obvious from the right-hand side. Use an explicit type when it improves readability.

```csharp
// Good: type is obvious
var customer = new Customer();
var orders = new List<Order>();
var count = 0;

// Debatable: type is not obvious from the method name
var result = repository.GetData(); // What type is result? Hard to tell.

// Better: explicit type when the return type is unclear
List<Order> result = repository.GetData();
```

## Constants and Readonly

### `const`

A `const` is a compile-time constant. Its value must be known at compile time and cannot change.

```csharp
const int MaxRetries = 3;
const string ApiVersion = "v1";
const double Pi = 3.14159265358979;

// MaxRetries = 5; // Compile error: cannot assign to a constant
```

Restrictions:

- `const` can be used with built-in numeric types, `bool`, `char`, `string`, enum types, and certain null constants.
- The value must be a compile-time constant or expression.
- A `const` field is implicitly static; a local constant is scoped to its block.

### `readonly`

A `readonly` field can be assigned at its declaration or in a constructor. A `static readonly` field can also be assigned in a static constructor. After initialization, the field cannot be reassigned.

```csharp
var config = new DatabaseConfig("Server=localhost;Database=App;", 100);
// config.MaxPoolSize = 200; // Compile error: readonly field

class DatabaseConfig
{
    public readonly string ConnectionString;
    public readonly int MaxPoolSize;

    public DatabaseConfig(string connectionString, int maxPoolSize)
    {
        ConnectionString = connectionString;
        MaxPoolSize = maxPoolSize;
    }
}
```

Use `const` for values that are true compile-time constants. Use `readonly` for values determined at runtime but not meant to be reassigned after initialization, such as configuration values loaded from a file.

## Nullable Value Types

Value types cannot be `null` by default. To allow a value type to be `null`, use `Nullable<T>` or the `?` shorthand.

```csharp
int regularInt = 10;
// regularInt = null; // Compile error

int? nullableInt = 10;
nullableInt = null; // This is allowed

Console.WriteLine(nullableInt.HasValue); // Output: False
Console.WriteLine(nullableInt.GetValueOrDefault()); // Output: 0
```

This is useful when mapping database columns that allow `NULL`. A database column `Age INT NULL` typically maps to `int?` in C#, not `int`.

Reference types can also hold `null`. When nullable reference types are enabled, annotations such as `string` and `string?` help the compiler warn about possible null usage; they do not create different runtime types.

## Boxing and Unboxing

Boxing converts a value type to `object` (or an implemented interface). Unboxing converts a boxed value back to its value type.

```csharp
int number = 42;
object boxed = number;       // Boxing: int converted to object
int unboxed = (int)boxed;    // Unboxing: explicit cast required

// Unboxing to the wrong type throws an InvalidCastException.
// double wrong = (double)boxed; // Runtime error
```

Boxing can allocate an object and has a performance cost. In performance-critical backend code, avoid unnecessary boxing. Generics (covered in Phase 3) help preserve type information and avoid many boxing conversions.

## Basic Code Snippet

This demonstrates the structural concepts covered in this topic in a single program.

```csharp
// Program.cs — Variables and Data Types demo in .NET 10

// --- Value Types ---
int age = 28;
double temperature = 98.6;
decimal accountBalance = 1_250_000.75m;
bool isVerified = true;
char grade = 'A';
long population = 8_000_000_000L;

Console.WriteLine("=== Value Types ===");
Console.WriteLine($"Age: {age} (Type: {age.GetType().Name})");
Console.WriteLine($"Temperature: {temperature} (Type: {temperature.GetType().Name})");
Console.WriteLine($"Balance: {accountBalance} (Type: {accountBalance.GetType().Name})");
Console.WriteLine($"Verified: {isVerified} (Type: {isVerified.GetType().Name})");
Console.WriteLine($"Grade: {grade} (Type: {grade.GetType().Name})");
Console.WriteLine($"Population: {population} (Type: {population.GetType().Name})");

// --- Value Type Assignment (Copy) ---
int original = 100;
int copy = original;
copy = 999;
Console.WriteLine($"\nOriginal: {original}, Copy: {copy}");
// Output: Original: 100, Copy: 999

// --- Reference Types ---
string cityName = "New York";
int[] scores = { 85, 92, 78, 95 };

Console.WriteLine("\n=== Reference Types ===");
Console.WriteLine($"City: {cityName}");
Console.WriteLine($"Scores count: {scores.Length}");

// --- Reference Type Assignment (Same Object) ---
var list1 = new List<string> { "Apple", "Banana" };
var list2 = list1;
list2.Add("Cherry");
Console.WriteLine($"\nList1 count: {list1.Count}"); // Output: 3
Console.WriteLine($"List2 count: {list2.Count}"); // Output: 3

// --- Nullable Value Type ---
int? userId = null;
Console.WriteLine($"\nUserId has value: {userId.HasValue}");
userId = 42;
Console.WriteLine($"UserId has value: {userId.HasValue}, Value: {userId.Value}");

// --- Type Inference ---
var inferredInt = 10;
var inferredString = "Hello";
var inferredDecimal = 99.99m;
Console.WriteLine($"\nInferred types: {inferredInt.GetType().Name}, " +
                  $"{inferredString.GetType().Name}, {inferredDecimal.GetType().Name}");
```

## Industry-Level Code Snippet

This is a realistic domain model from a production e-commerce backend. It demonstrates how value types, reference types, nullable types, constants, and enums can be used together in an entity design.

```csharp
// Order.cs — Domain entity in a production e-commerce API
namespace MyBackendApp.Core.Entities;

public class Order
{
    // Constants for business rules
    public const int MaxItemsPerOrder = 500;
    public const decimal MinimumOrderTotal = 0.01m;
    public const decimal TaxRate = 0.08m;

    // Value types
    public int Id { get; set; }
    public decimal Subtotal { get; set; }
    public decimal TaxAmount { get; set; }
    public decimal TotalAmount { get; set; }
    public bool IsPaid { get; set; }
    public OrderStatus Status { get; set; }

    // Nullable value types — these database columns allow NULL
    public int? DiscountPercentage { get; set; }
    public DateTime? ShippedAt { get; set; }
    public DateTime? DeliveredAt { get; set; }

    // Non-nullable value types with defaults
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime UpdatedAt { get; set; } = DateTime.UtcNow;

    // Reference types
    public string OrderNumber { get; set; } = string.Empty;
    public string CustomerEmail { get; set; } = string.Empty;
    public string? ShippingNotes { get; set; } // Nullable reference type

    // Reference type: collection navigation property
    public List<OrderItem> Items { get; set; } = new();

    // Calculated property — computed, not stored
    public decimal CalculatedTotal =>
        Subtotal + TaxAmount - (DiscountPercentage.HasValue
            ? Subtotal * (DiscountPercentage.Value / 100m)
            : 0m);

    public void MarkAsShipped()
    {
        if (Status != OrderStatus.Processing)
        {
            throw new InvalidOperationException(
                $"Cannot ship an order with status '{Status}'. " +
                $"Expected '{OrderStatus.Processing}'.");
        }

        Status = OrderStatus.Shipped;
        ShippedAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
    }

    public void ApplyDiscount(int percentage)
    {
        if (percentage < 0 || percentage > 100)
        {
            throw new ArgumentOutOfRangeException(
                nameof(percentage),
                "Discount must be between 0 and 100.");
        }

        DiscountPercentage = percentage;
        TaxAmount = Subtotal * TaxRate;
        TotalAmount = CalculatedTotal;
        UpdatedAt = DateTime.UtcNow;
    }
}

public enum OrderStatus
{
    Pending = 0,
    Processing = 1,
    Shipped = 2,
    Delivered = 3,
    Cancelled = 4,
    Refunded = 5
}

public class OrderItem
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public string ProductName { get; set; } = string.Empty;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }

    // Value-type calculation
    public decimal LineTotal => Quantity * UnitPrice;
}
```

Key observations from this industry code:

- `decimal` is used for monetary values, a common choice when decimal arithmetic is needed. Define explicit rounding and currency-precision rules for the application.
- `int?` and `DateTime?` are used for optional database columns. They can map to nullable SQL columns.
- `string?` (nullable reference type) is used for optional text fields like `ShippingNotes`. This is enabled by `<Nullable>enable</Nullable>` in the `.csproj` file.
- `string.Empty` is used as the default for required string properties instead of `null`. This avoids an initially null value, though domain validation is still important.
- `const` is used for compile-time business-rule values.
- The enum provides named, type-safe status values instead of magic strings or integers.
- `List<OrderItem>` is a reference type initialized with `new()` (a target-typed `new` expression, C# 9+).
- Calculated properties like `CalculatedTotal` and `LineTotal` use expression-bodied members and value-type arithmetic.

## Key Terms Summary

| Term | Definition |
|---|---|
| Variable | A named storage location for a value of a specific type. |
| Value type | A type whose values are copied on assignment; storage location depends on context. |
| Reference type | A type whose variables hold references to objects; assignment copies the reference. |
| Stack | A per-thread call stack used for method frames; some local values may be stored there. |
| Managed heap | The area where objects are generally allocated and managed by the garbage collector. |
| Boxing | Converting a value type to `object` or an interface type; may allocate an object. |
| Unboxing | Converting a boxed value back to its value type; requires an explicit cast. |
| Nullable value type | A wrapper that allows a value type to hold `null`, written as `T?` or `Nullable<T>`. |
| Type inference | The compiler determines a local variable's type from its initialization, using `var`. |
| `const` | A compile-time constant whose value must be known at compile time. |
| `readonly` | A field that can be assigned during initialization but not reassigned afterward. |
| Immutable | An object whose state cannot be changed after creation. Strings are immutable. |
