Examples target .NET 10. Short code blocks are focused excerpts and may reuse type names; don't concatenate every excerpt into one source file. The **Basic Code Snippet** is a complete standalone console program, and the domain example is a separate, namespaced snippet.

## The Fundamental Difference

`struct` and `class` both define custom types, but they have different **value semantics** and **reference semantics**:

- A **struct is a value type**. A variable of that type holds the struct's value. Assignment or pass-by-value creates another value with the same field contents.
- A **class is a reference type**. A variable of that type holds a managed reference to an object. Assignment copies the reference, so both variables can refer to the same object.

This distinction is about what a variable means—not a promise that every struct lives on the stack or every class variable does. A struct is stored inline wherever its containing storage is: it might be a local, a field inside another object, or an element inside an array. Boxing a struct creates a heap object. A class instance is normally allocated on the managed heap, while a local variable holding its reference may be kept in a register or other storage chosen by the runtime.

A struct copy is also **not necessarily a deep clone**. If it contains reference-type fields, those references are copied too, so the copied structs can still refer to the same list, array, or other object.

## Syntax Comparison

### Class Declaration

```csharp
#nullable enable

class Customer
{
    public int Id { get; }
    public string Name { get; set; }
    public string Email { get; }

    public Customer(int id, string name, string email)
    {
        Id = id;
        Name = name;
        Email = email;
    }

    public string GetDisplayName() => $"{Name} ({Email})";
}
```

### Struct Declaration

```csharp
using System;

struct Point
{
    public double X { get; set; }
    public double Y { get; set; }

    public Point(double x, double y)
    {
        X = x;
        Y = y;
    }

    public double DistanceFromOrigin() => Math.Sqrt(X * X + Y * Y);
}
```

Both structs and classes can have fields, properties, constructors, methods, interfaces, nested types, operators, and indexers. The differences are in their value/reference semantics, inheritance rules, defaults, and runtime behavior.

## Behavioral Differences

### Assignment and Copying

```csharp
using System;

// Class: assignment copies the reference.
var customer1 = new Customer(1, "Alice", "alice@example.com");
var customer2 = customer1;

customer2.Name = "Bob";
Console.WriteLine(customer1.Name); // Bob: both references point to the same object.
Console.WriteLine(customer2.Name); // Bob

// Struct: assignment copies the value.
var point1 = new Point(10.0, 20.0);
var point2 = point1;

point2.X = 99.0;
Console.WriteLine(point1.X); // 10: point1 is unchanged.
Console.WriteLine(point2.X); // 99
```

The struct copy includes each field's value. If a struct has a `List<T>` field, the list reference is copied, not the list contents; the two struct values can still refer to the same list.

### Default Values and Constructors

A class variable's default value is `null`. A struct's default value is formed by setting its fields to their defaults (`0`, `false`, or `null` as appropriate). `default(T)` and array elements always use that zero-initialized default.

```csharp
#nullable enable
using System;

Point defaultPoint = default;
Point newPoint = new Point(); // No custom parameterless constructor: zero-initialized.
Console.WriteLine(defaultPoint.X); // 0
Console.WriteLine(defaultPoint.Y); // 0
Console.WriteLine(newPoint.X); // 0

Customer? noCustomer = null;
Console.WriteLine(noCustomer is null); // True

// A local must be assigned before it is read:
// Customer notAssigned;
// Console.WriteLine(notAssigned.Name); // Compile error: use of an unassigned local.

// C# 10+ permits an explicit parameterless struct constructor.
// new S() runs it, but default(S) and array elements still use zero initialization.
var fromNew = new CustomDefault();
var fromDefault = default(CustomDefault);
var fromArray = new CustomDefault[1];
Console.WriteLine(fromNew.Value);     // 42
Console.WriteLine(fromDefault.Value); // 0
Console.WriteLine(fromArray[0].Value); // 0

struct CustomDefault
{
    public int Value { get; set; }

    public CustomDefault()
    {
        Value = 42;
    }
}
```

Every struct has a default value. In C# 10 and later, a struct can also declare a public parameterless constructor. `new S()` invokes that constructor when one is declared; `default(S)` and newly created arrays of `S` still contain zero-initialized values. Don't design a struct whose zero state is unusable unless every path carefully validates it.

### Nullability

A class instance can be absent (`null`). A non-nullable reference annotation such as `Customer` is a compiler-analysis promise, not a runtime guarantee; use `Customer?` when null is an allowed value.

A non-nullable struct variable always has a value. `Point?` is shorthand for `Nullable<Point>` and can represent either no value or a `Point`.

```csharp
#nullable enable
using System;

Customer? maybeCustomer = null;
if (maybeCustomer is null)
{
    Console.WriteLine("No customer.");
}

Point? maybePoint = null;
Console.WriteLine(maybePoint.HasValue); // False

maybePoint = new Point(5, 10);
Console.WriteLine(maybePoint.Value.X); // 5

Point pointOrDefault = maybePoint ?? default;
```

Accessing `.Value` when a nullable value type has no value throws `InvalidOperationException`; use a null check, pattern, or `??` when absence is possible.

### Method Parameter Behavior

C# passes ordinary parameters **by value** unless you use `ref`, `in`, or `out`. For a class parameter, the copied value is a reference to the same object. For a struct parameter, the copied value is the struct itself.

```csharp
static void Rename(Customer customer)
{
    customer.Name = "Modified"; // Changes the shared object.
    customer = new Customer(2, "Replacement", "new@example.com");
    // Reassigning this local reference does not replace the caller's variable.
}

static void ChangePoint(Point point)
{
    point.X = 999; // Changes only this parameter's copy.
}

static void ChangeCallerPoint(ref Point point)
{
    point.X = 999; // ref exposes the caller's variable.
}
```

Passing a large struct by `in` can avoid a by-value parameter copy when appropriate, but can introduce indirection and possible defensive copies depending on the type and members. Use it only when the API semantics and measurements justify it.

### Inheritance and Interfaces

Classes support single class inheritance and can implement multiple interfaces. Structs cannot inherit from another class or struct and cannot be a base class, but they can implement interfaces. Every struct implicitly derives from `System.ValueType`, which itself derives from `System.Object`.

```csharp
using System;

class Animal
{
    public string Name { get; set; } = string.Empty;
    public virtual void Speak() => Console.WriteLine("...");
}

class Dog : Animal
{
    public override void Speak() => Console.WriteLine("Woof!");
}

// A struct can implement an interface, but can't derive from Animal.
interface IHasLength
{
    double Length { get; }
}

struct Vector : IHasLength
{
    public double X { get; set; }
    public double Y { get; set; }
    public double Length => Math.Sqrt(X * X + Y * Y);
}
```

Converting a struct to an interface-typed variable usually boxes it. Generic code constrained to an interface can often avoid boxing; see the boxing section below.

## Complete Feature Comparison

| Feature | Struct | Class |
|---|---|---|
| Type category | Value type | Reference type |
| What a variable holds | The value itself | A managed reference to an object |
| Typical storage | Inline in its local, containing object, or array; may be boxed | Instance normally lives on the managed heap; local reference storage is separate |
| Default value | Zero-initialized value; all fields have their defaults | `null` |
| Assignment | Copies the value's fields (not a deep clone of referenced objects) | Copies the reference |
| Nullability | `T?` is `Nullable<T>` | `C?` is a nullable-reference annotation when nullable analysis is enabled |
| Inheritance | Cannot inherit from or be a base for another class/struct | Supports single class inheritance |
| Interfaces | Can implement interfaces | Can implement interfaces |
| Parameterless initialization | Always has a default value; an explicit parameterless constructor is allowed since C# 10 | Can declare constructors; a default one is synthesized if none are declared |
| `new S()` vs `default(S)` | `new S()` can run a custom parameterless constructor; `default(S)` always zero-initializes | `new C()` runs a constructor; `default(C)` is `null` |
| Finalizer | Not allowed | Allowed, but nondeterministic and uncommon |
| Abstract | Not allowed | Can be abstract |
| Sealed | Implicitly sealed | Can be sealed |
| Static type | Cannot be declared `static` | Can be a `static class` |
| Field initializers | Supported since C# 10; a struct with initializers must declare a constructor | Supported |
| Garbage collection | No separate object allocation for an unboxed value; containing objects/arrays are still GC-managed | Instances are normally GC-managed; a finalizer isn't deterministic cleanup |
| Copying cost | May copy the struct data; cost and optimization depend on size and use | Assignment copies a reference |

## Struct-Specific Features

### `readonly struct`

A `readonly struct` makes the struct's own instance fields and auto-properties read-only after construction. This is not deep immutability: a referenced object stored in a field could still be mutable. The compiler can avoid some defensive copies because members can't mutate the struct value, but `readonly` is not a blanket promise of a particular performance result.

```csharp
using System;

var start = new Vector2(3, 4);
var end = start.Add(new Vector2(1, 2));
Console.WriteLine(GetLength(start)); // 5
Console.WriteLine(end.Length);

static double GetLength(in Vector2 vector) => vector.Length;

readonly struct Vector2
{
    public double X { get; }
    public double Y { get; }

    public Vector2(double x, double y)
    {
        X = x;
        Y = y;
    }

    public double Length => Math.Sqrt(X * X + Y * Y);

    public Vector2 Add(Vector2 other) => new(X + other.X, Y + other.Y);
}
```

Use a readonly struct when the type has value semantics and its state should not change after creation. Coordinates, small measurements, and identifiers are common examples. Immutability does not make every type a good struct; size, defaults, boxing, and domain semantics still matter.

### `ref struct`

A `ref struct` is **stack-confined**: it can't escape to the managed heap. This enables types such as `Span<T>` to refer safely to memory with restricted lifetimes. The compiler prevents a `ref struct` from being boxed, used as an array element, stored in an ordinary class field, or captured by a lambda or local function.

```csharp
using System;

int[] numbers = { 1, 2, 3, 4, 5 };
var wrapper = new SpanWrapper(numbers.AsSpan());
Console.WriteLine(wrapper.Sum()); // 15

// These are compile-time errors:
// object boxed = wrapper;             // ref structs can't be boxed.
// Action action = () => wrapper.Sum(); // Can't capture a ref struct.

ref struct SpanWrapper
{
    private Span<int> _data;

    public SpanWrapper(Span<int> data)
    {
        _data = data;
    }

    public int Sum()
    {
        int total = 0;
        foreach (int value in _data)
        {
            total += value;
        }
        return total;
    }
}
```

`Span<T>` and `ReadOnlySpan<T>` are common framework `ref struct` types. In C# 13 and later, a `ref struct` local can be used in an async method or iterator **only when it doesn't remain in use across an `await` or `yield return` boundary**. Earlier C# versions prohibit those locals in async methods and iterators. A `net10.0` project defaults to C# 14, but a project's `LangVersion` setting can change the available rules.

Since C# 13, a `ref struct` can implement an interface, but converting it to an interface value would require boxing and is still disallowed. The restriction is about safe lifetime and storage, not a claim that the type is physically allocated in one specific machine-memory location in every optimized build.

### `readonly ref struct`

You can combine both modifiers for a stack-confined type whose instance state is read-only.

```csharp
using System;

readonly ref struct ReadOnlyDataView
{
    private readonly ReadOnlySpan<byte> _data;

    public ReadOnlyDataView(ReadOnlySpan<byte> data)
    {
        _data = data;
    }

    public int Length => _data.Length;
    public byte this[int index] => _data[index];
}
```

## Class-Specific Features

### Inheritance and Polymorphism

Classes can use virtual dispatch and inheritance to represent substitutable reference types. Structs can implement interfaces, but they can't form a class inheritance hierarchy.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

Notification[] notifications =
{
    new EmailNotification { Recipient = "alice@example.com", Subject = "Order shipped" },
    new SmsNotification { Recipient = "+1-555-0100", Subject = "Delivery tomorrow" }
};

foreach (Notification notification in notifications)
{
    await notification.SendAsync(CancellationToken.None);
}

abstract class Notification
{
    public string Recipient { get; set; } = string.Empty;
    public string Subject { get; set; } = string.Empty;
    public abstract Task SendAsync(CancellationToken cancellationToken);
}

sealed class EmailNotification : Notification
{
    public string Body { get; set; } = string.Empty;

    public override async Task SendAsync(CancellationToken cancellationToken)
    {
        // Replace this delay with a real email provider call.
        await Task.Delay(100, cancellationToken);
        Console.WriteLine($"Email sent to {Recipient}: {Subject}");
    }
}

sealed class SmsNotification : Notification
{
    public override async Task SendAsync(CancellationToken cancellationToken)
    {
        // Replace this delay with a real SMS provider call.
        await Task.Delay(50, cancellationToken);
        Console.WriteLine($"SMS sent to {Recipient}: {Subject}");
    }
}
```

### Finalizers and Resource Cleanup

A class can declare a finalizer, but it may run much later than an object becomes unreachable, and it may not run before process shutdown. Structs can't declare finalizers. Finalizers are not a substitute for deterministic resource cleanup.

```csharp
class FinalizerExample
{
    ~FinalizerExample()
    {
        // A real finalizer should release only unmanaged resources.
        // Prefer SafeHandle for unmanaged handles.
    }
}
```

Use `IDisposable` and `using` for deterministic cleanup. For unmanaged handles, prefer a `SafeHandle` implementation rather than writing a finalizer directly in most application code.

## Performance Considerations

### Structs, Allocation, and Arrays

Structs can reduce **separate per-element object allocations** when used as small values or in arrays. That doesn't mean every struct is stack-allocated or that a struct array has no allocation: the array itself is a managed object and is garbage-collected.

```csharp
#nullable enable
using System;

var points = new CoordinateValue[1_000_000]; // One array allocation; values are inline.
for (int i = 0; i < points.Length; i++)
{
    points[i] = new CoordinateValue(40.7128, -74.0060);
}

var customers = new CustomerReference?[1_000_000]; // Array of references; elements start null.
for (int i = 0; i < customers.Length; i++)
{
    customers[i] = new CustomerReference(40.7128, -74.0060); // Separate class instance.
}

readonly record struct CoordinateValue(double Latitude, double Longitude);

sealed class CustomerReference
{
    public double Latitude { get; }
    public double Longitude { get; }

    public CustomerReference(double latitude, double longitude)
    {
        Latitude = latitude;
        Longitude = longitude;
    }
}
```

An array of structs stores the values inline in the array object. An array of classes stores references; each non-null referenced object is a separate allocation. Dense arrays of small structs can improve locality for sequential scans, but the actual result depends on the workload, layout, allocation patterns, and access behavior. Measure before choosing a type for performance.

### Large Structs and Copies

A large struct passed by value may involve copying more data than a reference assignment. The JIT can optimize some copies, so don't assume a particular number of bytes moves on every call. `in` passes a readonly reference and can be useful for large immutable structs, but it can also add indirection or defensive copies.

```csharp
readonly struct LargeData
{
    public readonly decimal A, B, C, D; // 64 bytes of decimal fields.
    public readonly long E, F, G, H;    // 32 bytes of long fields; raw payload totals 96 bytes.
}

static class LargeDataProcessor
{
    public static decimal ProcessByValue(LargeData data) => data.A + data.B;
    public static decimal ProcessByIn(in LargeData data) => data.A + data.B;
}
```

Use `in` when it suits the API and measurements; it isn't automatically faster for every struct size or call site.

### Boxing

Converting a struct value to `object` or to an interface it implements typically **boxes** it: a heap object is created and a copy of the struct is stored in that box. Unboxing copies the value back out. This can add allocations and runtime overhead. Generic code with appropriate constraints can often call value-type implementations without boxing.

```csharp
using System;

var counter = new Counter { Value = 42 };
object boxed = counter;       // Boxing: creates an object containing a copy.
Counter unboxed = (Counter)boxed; // Unboxing: copies the value out.

ICountable asInterface = new CountableStruct { Value = 10 }; // Boxes the struct.
Console.WriteLine(asInterface.Value);

interface ICountable
{
    int Value { get; }
}

struct Counter
{
    public int Value;
}

struct CountableStruct : ICountable
{
    public int Value { get; set; }
}
```

Avoid frequent boxing in hot paths, but don't choose a class solely to avoid a hypothetical boxing cost. Generic APIs, typed collections, and measured usage patterns matter.

### Memory Layout in Collections

- A value-type array contains its struct elements inline in one array object.
- A reference-type array contains references; its objects are separately allocated when created.
- Structs embedded in a class are part of that containing object's storage. If a struct contains references, the garbage collector still tracks those references.

This layout can help cache locality in data-heavy code, but structs aren't automatically faster or more memory-efficient. Large inline values can make arrays and containing objects large, and boxing can negate allocation savings.

## When to Use Structs

Consider a struct when most of the following fit:

1. The type represents a value (two instances with the same data should mean the same thing), not an entity with independent identity.
2. The value is relatively small and inexpensive to copy. Around 16 bytes or less is a commonly cited starting point, not a hard rule or a guarantee for .NET 10 workloads.
3. The value should be immutable or nearly so; `readonly struct` is often a good fit.
4. The zero/default state can be valid, or every boundary handles the default state explicitly.
5. The type won't be boxed frequently and doesn't need class inheritance.

Common examples include small coordinates, measurements, date/time values, and strongly typed IDs. Some larger value objects can still be structs when their semantics and measured use justify it; don't apply a byte threshold mechanically.

## When to Use Classes

A class is often the clearer choice when the type:

- Has identity and a lifecycle (for example, a customer or order).
- Is mutable and changes should be visible through shared references.
- Is large or expensive to copy.
- Needs class inheritance, virtual dispatch, or `null` as a natural state.
- Owns resources or coordinates a complex service. Use `IDisposable`/`SafeHandle` for resource cleanup where appropriate.

Common backend classes include domain entities, services, repositories, controllers, and many configuration or integration objects. Records can also be reference types when value equality is the better semantic choice.

## Records: Reference or Value Types

A `record` or `record class` is a reference type; a `record struct` is a value type. Records synthesize value-based equality and useful members such as `ToString`. For positional records, the compiler also creates properties and a matching constructor.

- Positional `record class` properties are `init`-only by default, but a record can still contain mutable properties or referenced mutable objects. This is shallow, not deep, immutability.
- Positional `record struct` properties are read/write by default.
- Positional `readonly record struct` properties are init-only and its state is readonly.
- A regular struct has value-type equality behavior through `ValueType.Equals`, but the compiler doesn't automatically create a `==` operator for it. Record types synthesize record equality operators.

```csharp
using System;

record CustomerSnapshot(string Name, string Email, DateTime CreatedAt);
record struct PointRecord(double X, double Y); // Mutable positional properties.
readonly record struct TemperatureRecord(double Celsius); // Readonly value record.
```

Records don't replace the semantic decision between identity and value. A `record class` remains a reference type; a `record struct` still has struct copying and default-value behavior. Records are a concise option, not a promise of automatic performance improvement.

## Common Pitfalls

### Pitfall 1: Mutable Structs

A mutable struct copied to another variable can be changed independently. A struct returned by a property or indexer is also a copy: direct assignment to one of its fields or properties commonly causes CS1612. A call to a mutating method on a returned copy may compile but update only that temporary value, with the change then discarded.

```csharp
using System;

var container = new Container { Counter = new MutableCounter { Count = 0 } };

// container.Counter.Count = 1; // CS1612: property result is a copy.
container.Counter.Increment(); // May mutate a temporary copy; don't rely on it persisting.

// Copy, change, and assign back:
var temporary = container.Counter;
temporary.Increment();
container.Counter = temporary;
Console.WriteLine(container.Counter.Count); // 1

struct MutableCounter
{
    public int Count;
    public void Increment() => Count++;
}

class Container
{
    public MutableCounter Counter { get; set; }
}
```

Prefer immutable structs (`readonly struct` or `readonly record struct`) for value-like data. If the type needs shared mutable state, a class is usually easier to reason about.

### Pitfall 2: Assuming Structs Are Always Faster

A struct can avoid a separate object allocation in some contexts, but copying, larger inline storage, boxing, and cache behavior all affect performance. Benchmark representative workloads before changing a type for speed.

### Pitfall 3: Misunderstanding Structs in Async Code

A normal struct local that must survive an `await` is stored as part of the async state machine; it isn't automatically copied on every `await`. Large structs can increase state-machine size or copying costs, but the effect depends on compiler and runtime optimizations. Choose based on value semantics first, then measure.

`ref struct` locals have stricter lifetime rules. In C# 13 and later, an async method can use a `ref struct` local only when it doesn't need to remain live across an `await` boundary. Earlier language versions prohibit such locals in async methods.

## Basic Code Snippet

This complete .NET 10 console example demonstrates class reference sharing, struct value copying, a readonly record struct, default values, equality, and struct arrays.

```csharp
#nullable enable
using System;

Console.WriteLine("=== Class (reference type) ===");
var customer1 = new DemoCustomer(1, "Alice", "alice@example.com");
var customer2 = customer1;
customer2.Name = "Bob";
Console.WriteLine($"customer1.Name: {customer1.Name}"); // Bob
Console.WriteLine($"Same object: {ReferenceEquals(customer1, customer2)}"); // True

Console.WriteLine("\n=== Struct (value type) ===");
var point1 = new DemoPoint(10, 20);
var point2 = point1;
point2.X = 99;
Console.WriteLine($"point1: ({point1.X}, {point1.Y})"); // (10, 20)
Console.WriteLine($"point2: ({point2.X}, {point2.Y})"); // (99, 20)

Console.WriteLine("\n=== Readonly record struct ===");
var price = new DemoMoney(49.99m, DemoCurrency.USD);
var tax = new DemoMoney(4.00m, DemoCurrency.USD);
Console.WriteLine($"Total: {price.Add(tax)}");

Console.WriteLine("\n=== Record value equality ===");
var coord1 = new DemoCoordinate(40.7128, -74.0060);
var coord2 = new DemoCoordinate(40.7128, -74.0060);
var coord3 = new DemoCoordinate(34.0522, -118.2437);
Console.WriteLine($"coord1 == coord2: {coord1 == coord2}"); // True
Console.WriteLine($"coord1 == coord3: {coord1 == coord3}"); // False

Console.WriteLine("\n=== Defaults and arrays ===");
DemoPoint defaultPoint = default;
DemoCustomer? noCustomer = null;
Console.WriteLine($"Default point: ({defaultPoint.X}, {defaultPoint.Y})"); // (0, 0)
Console.WriteLine($"Customer is null: {noCustomer is null}"); // True

DemoPoint[] points = { new(1, 2), new(3, 4), new(5, 6) };
foreach (DemoPoint point in points)
{
    Console.WriteLine($"({point.X}, {point.Y}) — distance {point.DistanceFromOrigin():F2}");
}

sealed class DemoCustomer
{
    public int Id { get; }
    public string Name { get; set; }
    public string Email { get; }

    public DemoCustomer(int id, string name, string email)
    {
        Id = id;
        Name = name;
        Email = email;
    }
}

struct DemoPoint
{
    public double X { get; set; }
    public double Y { get; set; }

    public DemoPoint(double x, double y)
    {
        X = x;
        Y = y;
    }

    public double DistanceFromOrigin() => Math.Sqrt(X * X + Y * Y);
}

enum DemoCurrency
{
    USD = 0,
    EUR = 1
}

readonly record struct DemoMoney(decimal Amount, DemoCurrency Currency)
{
    public DemoMoney Add(DemoMoney other)
    {
        if (Currency != other.Currency)
        {
            throw new InvalidOperationException("Cannot add different currencies.");
        }

        return new DemoMoney(Amount + other.Amount, Currency);
    }

    public override string ToString() => $"{Amount:F2} {Currency}";
}

readonly record struct DemoCoordinate(double Latitude, double Longitude);
```

## Applied Domain Example (Illustrative)

This separate .NET 10 snippet uses structs for value-like data and classes for entities with identity and lifecycle. It's illustrative, not a drop-in production financial model: real systems need application-specific persistence mappings, currency and tax rules, authorization, clock abstractions, and validation.

```csharp
#nullable enable
using System;
using System.Collections.Generic;

namespace MyBackendApp.Core.Domain;

public readonly record struct CustomerId(int Value)
{
    public bool IsEmpty => Value <= 0;
    public override string ToString() => $"CUST-{Value:D6}";
}

public readonly record struct OrderId(int Value)
{
    public bool IsEmpty => Value <= 0;
    public override string ToString() => $"ORD-{Value:D8}";
}

// Deliberately closed example set; production currency codes may need an extensible validated type.
public enum CurrencyCode
{
    USD = 0, // Makes default(Money) a valid zero-USD value.
    EUR = 1
}

public readonly record struct Money(decimal Amount, CurrencyCode Currency)
{
    public bool IsValid => Enum.IsDefined(Currency);

    public static Money Zero(CurrencyCode currency)
    {
        if (!Enum.IsDefined(currency))
        {
            throw new ArgumentOutOfRangeException(nameof(currency));
        }

        return new Money(0m, currency);
    }

    public static Money Usd(decimal amount) => new(amount, CurrencyCode.USD);
    public static Money Eur(decimal amount) => new(amount, CurrencyCode.EUR);

    public Money Add(Money other)
    {
        EnsureSameCurrency(other);
        return new Money(Amount + other.Amount, Currency);
    }

    public Money Subtract(Money other)
    {
        EnsureSameCurrency(other);
        return new Money(Amount - other.Amount, Currency);
    }

    public Money Multiply(decimal factor)
    {
        EnsureValid();
        return new Money(Amount * factor, Currency);
    }

    private void EnsureSameCurrency(Money other)
    {
        EnsureValid();
        other.EnsureValid();
        if (Currency != other.Currency)
        {
            throw new InvalidOperationException(
                $"Cannot operate on different currencies: {Currency} and {other.Currency}.");
        }
    }

    private void EnsureValid()
    {
        if (!IsValid)
        {
            throw new InvalidOperationException("Unknown currency code.");
        }
    }

    public override string ToString() => $"{Amount:F2} {Currency}";
}

public readonly record struct DateRange
{
    public DateOnly Start { get; }
    public DateOnly End { get; }

    public DateRange(DateOnly start, DateOnly end)
    {
        if (end < start)
        {
            throw new ArgumentException("End date must be on or after start date.", nameof(end));
        }

        Start = start;
        End = end;
    }

    public int DayCountInclusive => End.DayNumber - Start.DayNumber + 1;
    public bool Contains(DateOnly date) => date >= Start && date <= End;
    public bool Overlaps(DateRange other) => Start <= other.End && other.Start <= End;

    // Pass the date in so callers can use a consistent clock and test boundary cases.
    public static DateRange Last30Days(DateOnly today) => new(today.AddDays(-29), today);
}

public readonly record struct Percentage
{
    public decimal Value { get; }

    public Percentage(decimal value)
    {
        if (value < 0m || value > 100m)
        {
            throw new ArgumentOutOfRangeException(
                nameof(value), "Percentage must be between 0 and 100.");
        }

        Value = value;
    }

    public decimal AsFraction => Value / 100m;
    public Money ApplyTo(Money amount) => amount.Multiply(AsFraction);
    public override string ToString() => $"{Value:F2}%";
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
    Confirmed = 10,
    Processing = 20,
    Shipped = 30,
    Delivered = 40,
    Cancelled = 50
}

public readonly record struct OrderLineItem
{
    public string? ProductName { get; }
    public int Quantity { get; }
    public Money UnitPrice { get; }

    public OrderLineItem(string productName, int quantity, Money unitPrice)
    {
        if (string.IsNullOrWhiteSpace(productName))
        {
            throw new ArgumentException("Product name is required.", nameof(productName));
        }

        if (quantity <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(quantity), "Quantity must be positive.");
        }

        if (!unitPrice.IsValid || unitPrice.Amount < 0m)
        {
            throw new ArgumentException("Unit price must be valid and non-negative.", nameof(unitPrice));
        }

        ProductName = productName.Trim();
        Quantity = quantity;
        UnitPrice = unitPrice;
    }

    // default(OrderLineItem) is possible even though its values don't satisfy this type's invariants.
    public bool IsValid => !string.IsNullOrWhiteSpace(ProductName)
        && Quantity > 0
        && UnitPrice.IsValid
        && UnitPrice.Amount >= 0m;

    public Money LineTotal
    {
        get
        {
            if (!IsValid)
            {
                throw new InvalidOperationException("Cannot total an invalid line item.");
            }

            return UnitPrice.Multiply(Quantity);
        }
    }
}

public sealed class Customer
{
    private readonly List<Order> _orders = new();

    public CustomerId Id { get; private set; }
    public string FullName { get; private set; } = string.Empty;
    public string Email { get; private set; } = string.Empty;
    public CustomerStatus Status { get; private set; }
    public DateTimeOffset CreatedAt { get; private set; }
    public DateTimeOffset? LastModifiedAt { get; private set; }
    public IReadOnlyList<Order> Orders => _orders;

    private Customer() { }

    public static Customer Create(CustomerId id, string fullName, string email)
    {
        if (id.IsEmpty)
        {
            throw new ArgumentException("Customer ID must be positive.", nameof(id));
        }

        if (string.IsNullOrWhiteSpace(fullName))
        {
            throw new ArgumentException("Full name is required.", nameof(fullName));
        }

        string normalizedEmail = NormalizeEmail(email);
        return new Customer
        {
            Id = id,
            FullName = fullName.Trim(),
            Email = normalizedEmail,
            Status = CustomerStatus.Active,
            CreatedAt = DateTimeOffset.UtcNow
        };
    }

    public void UpdateEmail(string email)
    {
        Email = NormalizeEmail(email);
        LastModifiedAt = DateTimeOffset.UtcNow;
    }

    public void Deactivate()
    {
        Status = CustomerStatus.Deactivated;
        LastModifiedAt = DateTimeOffset.UtcNow;
    }

    public void AddOrder(Order order)
    {
        ArgumentNullException.ThrowIfNull(order);
        if (order.CustomerId != Id)
        {
            throw new ArgumentException("Order belongs to a different customer.", nameof(order));
        }

        _orders.Add(order);
    }

    public Money GetTotalSpent(CurrencyCode currency)
    {
        Money total = Money.Zero(currency);
        foreach (Order order in _orders)
        {
            if (order.Status != OrderStatus.Cancelled)
            {
                // Money.Add rejects a different currency; conversion must be explicit.
                total = total.Add(order.TotalAmount);
            }
        }

        return total;
    }

    public int GetOrderCount(DateRange? range = null)
    {
        if (!range.HasValue)
        {
            return _orders.Count;
        }

        DateRange selectedRange = range.Value;
        int count = 0;
        foreach (Order order in _orders)
        {
            DateOnly orderDate = DateOnly.FromDateTime(order.CreatedAt.UtcDateTime);
            if (selectedRange.Contains(orderDate))
            {
                count++;
            }
        }

        return count;
    }

    private static string NormalizeEmail(string email)
    {
        // This is only a minimal example; use the application's real email validation policy.
        if (string.IsNullOrWhiteSpace(email) || !email.Contains('@'))
        {
            throw new ArgumentException("A valid email address is required.", nameof(email));
        }

        return email.Trim().ToLowerInvariant();
    }
}

public sealed class Order
{
    private readonly List<OrderLineItem> _lineItems = new();

    public OrderId Id { get; private set; }
    public CustomerId CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    public Money Subtotal { get; private set; }
    public Money DiscountAmount { get; private set; }
    public Money TaxAmount { get; private set; }
    public Money TotalAmount { get; private set; }
    public Percentage? DiscountRate { get; private set; }
    public DateTimeOffset CreatedAt { get; private set; }
    public DateTimeOffset? ShippedAt { get; private set; }
    public IReadOnlyList<OrderLineItem> LineItems => _lineItems;

    private Order() { }

    public static Order Create(
        OrderId id,
        CustomerId customerId,
        IReadOnlyList<OrderLineItem> lineItems,
        Percentage? discountRate = null)
    {
        ArgumentNullException.ThrowIfNull(lineItems);
        if (id.IsEmpty)
        {
            throw new ArgumentException("Order ID must be positive.", nameof(id));
        }

        if (customerId.IsEmpty)
        {
            throw new ArgumentException("Customer ID must be positive.", nameof(customerId));
        }

        if (lineItems.Count == 0)
        {
            throw new ArgumentException("Order must have at least one line item.", nameof(lineItems));
        }

        if (!lineItems[0].IsValid)
        {
            throw new ArgumentException("Line items must be valid.", nameof(lineItems));
        }

        CurrencyCode currency = lineItems[0].UnitPrice.Currency;
        Money subtotal = Money.Zero(currency);
        var copiedItems = new List<OrderLineItem>(lineItems.Count);

        foreach (OrderLineItem item in lineItems)
        {
            if (!item.IsValid)
            {
                throw new ArgumentException("Line items must be valid.", nameof(lineItems));
            }

            if (item.UnitPrice.Currency != currency)
            {
                throw new InvalidOperationException("All order lines must use the same currency.");
            }

            subtotal = subtotal.Add(item.LineTotal);
            copiedItems.Add(item);
        }

        Money discount = discountRate.HasValue
            ? discountRate.Value.ApplyTo(subtotal)
            : Money.Zero(currency);
        Money afterDiscount = subtotal.Subtract(discount);
        Money tax = new Percentage(8m).ApplyTo(afterDiscount); // Simplified example rate.
        Money total = afterDiscount.Add(tax);

        var order = new Order
        {
            Id = id,
            CustomerId = customerId,
            Status = OrderStatus.Pending,
            Subtotal = subtotal,
            DiscountAmount = discount,
            TaxAmount = tax,
            TotalAmount = total,
            DiscountRate = discountRate,
            CreatedAt = DateTimeOffset.UtcNow
        };
        order._lineItems.AddRange(copiedItems);
        return order;
    }

    public void Confirm()
    {
        if (Status != OrderStatus.Pending)
        {
            throw new InvalidOperationException($"Cannot confirm an order in '{Status}' status.");
        }

        Status = OrderStatus.Confirmed;
    }

    public void BeginProcessing()
    {
        if (Status != OrderStatus.Confirmed)
        {
            throw new InvalidOperationException($"Cannot process an order in '{Status}' status.");
        }

        Status = OrderStatus.Processing;
    }

    public void MarkAsShipped()
    {
        if (Status != OrderStatus.Confirmed && Status != OrderStatus.Processing)
        {
            throw new InvalidOperationException($"Cannot ship an order in '{Status}' status.");
        }

        Status = OrderStatus.Shipped;
        ShippedAt = DateTimeOffset.UtcNow;
    }
}
```

### Key Observations

- `CustomerId` and `OrderId` are value-like wrappers with value equality, so the compiler won't let a method expecting one ID type accept the other by mistake. Their payload is an `int`; don't promise zero runtime overhead without checking the actual JIT, serializer, database mapping, and call sites.
- The code deliberately makes `CurrencyCode.USD` zero so `default(Money)` is a meaningful zero-USD value. A production currency model may need a validated string or another extensible representation rather than a closed enum.
- `DateRange` uses inclusive endpoints and reports an inclusive day count. Its zero-initialized default has the same start and end date, so it represents one day. `Percentage` uses a valid zero as its default and validates values created through its constructor.
- `default(OrderLineItem)` is still possible and is not a valid order line. The example exposes `IsValid` and checks items at `Order.Create`; a real model should make default values safe or keep validation at every boundary where a default can enter.
- `Customer` and `Order` are classes because they have identity, lifecycle, and mutable state. `OrderLineItem` is an immutable value with no independent identity.
- The sample rejects cross-currency totals instead of silently adding unlike amounts. Production code must define explicit conversion/rate and rounding rules; a decimal amount by itself isn't enough to represent financial correctness.
- A struct isn't necessarily stack-allocated, and an array of structs is still a managed heap allocation. The advantage shown is inline storage of array elements, not “no GC.”
- The `Percentage` example uses a regular explicit constructor. C# doesn't support a Kotlin-style compact constructor body such as `public Percentage { ... }` for a record struct.

## Key Terms Summary

| Term | Definition |
|---|---|
| Struct | A value type whose variable holds the value; assignment copies its fields. |
| Class | A reference type whose variable holds a reference to an object. |
| Value semantics | Copies represent separate values; changes to one copy don't change another. |
| Reference semantics | Multiple references can identify the same mutable object. |
| `readonly struct` | A struct whose instance state can't be changed after construction. |
| `ref struct` | A stack-confined type that can't escape to the managed heap. |
| `record struct` | A struct record with compiler-generated value equality and other data-oriented members. |
| Boxing | Wrapping a value-type copy in a heap object to use it as a reference type or interface value. |
| Defensive copy | A compiler-generated copy used in some readonly contexts to prevent mutation through a readonly reference. |
| `in` parameter | A by-reference, readonly input parameter; use when its semantics and measured performance fit. |
| `Nullable<T>` | A value-type wrapper that can represent either no value or a `T`; written `T?`. |
| Entity | A domain object defined by identity and lifecycle, commonly modeled as a class. |
| Value object | A domain concept defined by its content rather than identity, commonly modeled as an immutable struct or record. |

## Further Reading

- [Structure types — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct) — value semantics, defaults, constructors, and struct limitations.
- [C# structs — fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/structs) — constructors, readonly structs, and struct usage.
- [`ref struct` types — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct) — lifetime restrictions and C# 13 updates.
- [What's new in C# 13](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-13) — `ref struct` locals in async methods and iterators.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-versioning) — .NET 10 defaults to C# 14; a project can override the language version.
- [Choosing between class and struct — Framework Design Guidelines](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct) — design guidance on size, mutability, boxing, and value semantics.
- [Avoid memory allocations and data copies — C#](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/) — performance considerations and passing values by reference.
- [Records — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) — record classes, record structs, and generated equality.
- [Compiler error CS1612](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/cs1612) — modifying struct values returned from properties or indexers.
- [Use constructors — C# programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/using-constructors) — constructor behavior for classes and structs.
