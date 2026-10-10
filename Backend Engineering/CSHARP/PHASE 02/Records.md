Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 What Is a Record?

A **record** is a C# class or struct form with compiler-generated support for value equality, readable `ToString()` output, and nondestructive copying. Record classes arrived in C# 9; record structs arrived in C# 10.

Records are often useful for data-centric types, but **a record isn't automatically immutable**. A positional `record class` has `init`-only properties by default, while a positional `record struct` has read/write properties. Records can also declare mutable properties or fields. Even `init`-only properties provide only *shallow* immutability when they refer to mutable objects.

For a plain class that hasn't customized equality, `==` and `Equals` use object identity. Record equality instead compares the record's data members using each member's own equality rules. This is value equality, not necessarily a recursive comparison of every nested object or collection.

```text
Plain class with no equality override       Record class
+-----------------------------------+       +-----------------------------------+
| p1 = new PointClass(1, 2)         |       | p1 = new PointRecord(1, 2)        |
| p2 = new PointClass(1, 2)         |       | p2 = new PointRecord(1, 2)        |
| p1 == p2  -> false                |       | p1 == p2  -> true                 |
| different object identities       |       | same record type and member values|
+-----------------------------------+       +-----------------------------------+
```

This comparison assumes the class hasn't overloaded equality and the record hasn't customized its generated equality. Avoid describing identity as a stable “heap address”: object locations can change, while object identity is the relevant concept.

### 1.2 Why Records Can Help in Backend Development

Records are often a good fit for:

- **Data transfer objects (DTOs)** with snapshot-like data.
- **CQRS commands and queries** that carry request values.
- **Domain events** representing facts that have occurred.
- **Value objects** when member-based equality matches the domain rules.

Choose a regular class when identity, mutable lifecycle state, or reference equality is central. In particular, EF Core tracks entity instances by reference identity; record value equality and shallow immutability can be a poor fit for EF Core entity types. Records are a useful option, not a universal backend standard.

## 2. Record Syntax Variants

### 2.1 Positional Records

```csharp
public record Coordinates(double Latitude, double Longitude);
```

For a positional record class, the compiler supplies a primary constructor and `init`-only properties for the positional parameters. It also synthesizes a `Deconstruct` method, value-equality members, and display formatting such as `Coordinates { Latitude = ..., Longitude = ... }`. The compiler-generated implementation includes details beyond a short hand-written expansion, so treat the source declaration—not a pseudo-expansion—as the contract.

Equality is based on the record's declared data members and each member's equality implementation. For example, a `string` compares by value, but a collection may compare by its own implementation rather than by walking every element. If sequence equality is required, supply an appropriate comparer or define equality explicitly.

### 2.2 Record Classes and Record Structs

- `record` and `record class` declare a **reference type**. In a positional record class, generated properties are `init`-only by default.
- `record struct` declares a **value type**. Its positional properties are mutable (`get`/`set`) by default.
- `readonly record struct` makes the value type readonly; its positional properties are init-only.

```csharp
public record class Employee(string Name, decimal Salary);
public record struct Point(int X, int Y);                       // Mutable value type
public readonly record struct ImmutablePoint(int X, int Y);     // Readonly value type
```

Regular structs also have value equality through `ValueType.Equals`, though that implementation relies on reflection. Record structs synthesize strongly typed equality members and operators using their declared data members. A record struct is **not** automatically stack-allocated: value types can be stored inline in other objects or arrays, in registers or stack locations, and can be boxed.

### 2.3 The `with` Expression

A `with` expression creates a copy with selected members changed:

```csharp
var original = new Coordinates(40.7128, -74.0060);
var updated = original with { Latitude = 41.0000 };

// original.Latitude is still 40.7128; updated.Latitude is 41.0000.
```

For a record class, the copy is a new object but is **shallow**: any referenced mutable object is shared unless custom copy behavior is supplied. A record struct is copied by value. A `with` expression isn't a deep-cloning or validation mechanism.

### 2.4 Record Inheritance

Record classes can inherit from other record classes; a record class can't inherit from a non-record class, and an ordinary class can't inherit from a record. Record structs don't support class inheritance.

```csharp
public record Shape(string Color);
public record Circle(string Color, double Radius) : Shape(Color);
```

For record-class equality, the runtime types must match, and synthesized equality accounts for data members in the base and derived records. Synthesized `ToString()` formatting includes inherited and derived members. A derived positional record repeats the base record's primary-constructor parameters, so its generated `Deconstruct` includes those positional values plus its own. It doesn't automatically include arbitrary non-positional properties, and a base-typed variable uses the base record's deconstruction signature.

## 3. Records vs. Classes: Decision Matrix

| Dimension | Plain `class` (default behavior) | `record` |
|---|---|---|
| **Equality** | Reference identity unless equality is customized. | Compiler-generated value equality over declared data members, using each member's equality rules. |
| **Mutability** | Can be mutable or made immutable by design. | Can be mutable; positional record-class properties are init-only by default, but this is shallow immutability. |
| **`ToString()`** | Default output is the type name unless overridden. | Compiler-generated display of public data members; avoid logging sensitive records blindly. |
| **Copying** | No built-in record-style `with` copy. | Supports `with` for a shallow copy with selected changes. |
| **Typical fit** | Identity-bearing entities and mutable lifecycle state. | Snapshot-like DTOs, commands, queries, events, and value objects when value equality is appropriate. |

## 4. Basic Syntax Example

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public sealed record ProductDto(string Sku, string Name, decimal Price);

public static class RecordDemo
{
    public static void Run()
    {
        var product1 = new ProductDto("SKU-001", "Wireless Mouse", 24.99m);
        var product2 = new ProductDto("SKU-001", "Wireless Mouse", 24.99m);

        Console.WriteLine(product1 == product2); // True: same record type and values
        Console.WriteLine(product1);              // Generated, readable record format

        // Nondestructive update: creates a copy with a changed Price.
        var discounted = product1 with { Price = 19.99m };
        Console.WriteLine(discounted);
        Console.WriteLine(product1.Price);        // Still 24.99

        // The positional parameters generate a Deconstruct method.
        var (sku, name, price) = product1;
        Console.WriteLine($"{sku} | {name} | {price}");
    }
}
```

The record's formatted numeric output follows the current culture. Don't treat generated `ToString()` as a stable serialization format or log sensitive values without considering data exposure.

## 5. Backend Example: Commands, Queries, Events, and DTOs

These types use `ImmutableArray<T>` for the command's item collection. This avoids exposing a caller-owned mutable `List<T>` through an `IReadOnlyList<T>` property. It still doesn't make record equality recursively compare collection elements: equality follows `ImmutableArray<T>`'s own equality behavior. If command deduplication requires sequence-by-sequence equality, define that explicitly.

```csharp
#nullable enable
using System;
using System.Collections.Immutable;

namespace MyBackendApp.Core.Application.Orders;

public sealed record CreateOrderCommand(
    Guid CustomerId,
    ImmutableArray<OrderLineItemDto> Items,
    string ShippingRegion);

public sealed record OrderLineItemDto(
    Guid ProductId,
    string Sku,
    int Quantity,
    decimal UnitPrice);

public sealed record GetOrderByIdQuery(Guid OrderId);

public sealed record OrderCreatedEvent(
    Guid OrderId,
    Guid CustomerId,
    decimal TotalAmount,
    DateTimeOffset OccurredAtUtc);

public sealed record OrderSummaryResponse(
    Guid OrderId,
    string Status,
    decimal TotalAmount,
    DateTimeOffset CreatedAtUtc)
{
    // Pass the reference time in so this calculation is deterministic for a caller.
    public bool IsRecentAt(DateTimeOffset now) =>
        now >= CreatedAtUtc && now - CreatedAtUtc < TimeSpan.FromHours(24);
}

public sealed record CreateOrderResult(
    OrderSummaryResponse Response,
    OrderCreatedEvent CreatedEvent);

public sealed class OrderApplicationService
{
    public CreateOrderResult Handle(CreateOrderCommand command)
    {
        ArgumentNullException.ThrowIfNull(command);
        ArgumentException.ThrowIfNullOrWhiteSpace(command.ShippingRegion);

        if (command.CustomerId == Guid.Empty)
        {
            throw new ArgumentException("Customer ID is required.", nameof(command));
        }

        if (command.Items.IsDefaultOrEmpty)
        {
            throw new InvalidOperationException("An order must contain at least one line item.");
        }

        decimal total = 0m;
        foreach (OrderLineItemDto item in command.Items)
        {
            ArgumentNullException.ThrowIfNull(item);

            if (item.Quantity <= 0)
            {
                throw new ArgumentOutOfRangeException(nameof(command), "Quantity must be positive.");
            }

            if (item.UnitPrice < 0m)
            {
                throw new ArgumentOutOfRangeException(nameof(command), "Unit price can't be negative.");
            }

            total = checked(total + checked(item.UnitPrice * item.Quantity));
        }

        Guid orderId = Guid.NewGuid();
        DateTimeOffset occurredAtUtc = DateTimeOffset.UtcNow;

        var createdEvent = new OrderCreatedEvent(
            OrderId: orderId,
            CustomerId: command.CustomerId,
            TotalAmount: total,
            OccurredAtUtc: occurredAtUtc);

        var response = new OrderSummaryResponse(
            OrderId: orderId,
            Status: "Created",
            TotalAmount: total,
            CreatedAtUtc: occurredAtUtc);

        return new CreateOrderResult(response, createdEvent);
    }

    public CreateOrderCommand ReapplyToNewRegion(
        CreateOrderCommand original,
        string newRegion)
    {
        ArgumentNullException.ThrowIfNull(original);
        ArgumentException.ThrowIfNullOrWhiteSpace(newRegion);

        // The original command remains unchanged; its immutable items are shared.
        return original with { ShippingRegion = newRegion };
    }
}

// Example command construction:
// var command = new CreateOrderCommand(
//     Guid.NewGuid(),
//     ImmutableArray.Create(new OrderLineItemDto(Guid.NewGuid(), "SKU-001", 2, 24.99m)),
//     "DOMESTIC");
```

This sample constructs an event snapshot but doesn't persist the order or publish the event. A production handler would use appropriate persistence and event-publishing boundaries, often with an outbox when transactional consistency matters. The example also omits currency, tax, and rounding policies.

## 6. Design Guidance and Common Mistakes

- **Calling records immutable by default:** record types can have `set` properties and mutable fields. Positional record classes are init-only by default, but referenced objects can still change.
- **Treating `with` as a deep clone:** references inside a copied record are shared unless custom copy behavior is implemented.
- **Assuming record equality recursively compares nested collections:** equality uses each data member's equality implementation. Arrays and many list-like types don't compare their contents by default.
- **Using records for identity-based entities automatically:** value equality can conflict with identity maps and change tracking. Prefer a regular class for EF Core entity types unless the model and tracking behavior have been deliberately designed otherwise.
- **Assuming record structs are stack allocated:** storage depends on context; boxing is also possible. Choose a struct based on value-type semantics and measured needs, not a stack-allocation promise.
- **Assuming deconstruction includes every record member:** a derived positional record repeats base primary-constructor parameters, but generated `Deconstruct` methods don't include arbitrary non-positional members; the compile-time type determines which signature is used.
- **Treating generated `ToString()` as serialization:** its output is for display and can expose public data. Use a serializer with an explicit contract for wire formats and logs.
- **Assuming `IReadOnlyList<T>` means immutable:** the original mutable collection may still be changed by its owner. Use an immutable collection or take a defensive copy when that matters.
- **Using record equality as a cache or deduplication key without checking members:** mutable members can change hash behavior, and collection equality may not match the domain's intended comparison.
- **Using records for every DTO or domain type:** choose based on identity, equality, mutability, serialization needs, and lifecycle—not just concise syntax.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Record** | A class or struct form with compiler-generated value-equality and display/copy support. | Concisely models data-centric types when member-based equality is appropriate. |
| **Positional record** | A record whose primary-constructor parameters generate properties and a `Deconstruct` method. | Reduces boilerplate for simple data carriers. |
| **Record class** | A reference-type record with value equality and record-specific behavior. | Useful for snapshot-like DTOs, commands, queries, and events. |
| **Record struct** | A value-type record with compiler-generated equality members. | Useful when value-type semantics are appropriate; not a stack-allocation guarantee. |
| **Value equality** | Equality determined by a type's data members and each member's equality rules. | Helps compare value-like records, but doesn't promise recursive collection equality. |
| **Shallow immutability** | Properties/references can't be reassigned after initialization, but referenced objects can still mutate. | Important when passing records across asynchronous or concurrent boundaries. |
| **`with` expression** | Creates a copy with selected fields or properties changed. | Supports nondestructive updates; class-record copies are shallow by default. |
| **Deconstruction** | A generated method that assigns positional members to `out` variables. | Provides tuple-like extraction from positional records. |

## 8. Official References

- [Records — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) — positional syntax, equality, mutability, inheritance, `ToString()`, and EF Core entity guidance.
- [`with` expression — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression) — nondestructive copies and shallow-copy behavior.
- [Choosing between class and struct — .NET design guidelines](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct) — reference-type and value-type design tradeoffs.
- [`ImmutableArray<T>` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.collections.immutable.immutablearray-1?view=net-10.0) — immutable collection used for command line items.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — .NET 10 defaults to C# 14.
- [C# version history](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history) — language feature and release history, including record classes and record structs.
