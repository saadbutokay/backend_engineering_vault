Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 Two Strategies for Reuse

Object-oriented design commonly reuses behavior in two different ways:

1. **Inheritance (“is-a”)**: a derived class specializes a base class and can reuse or override its behavior. In C#, a class has at most one direct base class, though it can implement multiple interfaces.
2. **Composition (“has-a” or “uses-a”)**: an object holds references to collaborators and delegates some work to them. The collaborators are often expressed as interfaces, but composition can also use concrete types. Holding a reference doesn't necessarily mean the containing object owns the collaborator's lifetime.

```text
             INHERITANCE (“is-a”)                   COMPOSITION (“has-a” / “uses-a”)
+----------------------------------+          +--------------------------------------+
|          NotificationBase        |          |             OrderNotifier            |
|          Send(message)           |          |  - _channel: INotificationChannel   |
+----------------------------------+          |  + Notify(order) delegates to it     |
                 ^                            +--------------------------------------+
                 | derives from                                |
+----------------------------------+                           | holds a reference to
|        EmailNotification        |                           v
|        overrides Send           |          +--------------------------------------+
+----------------------------------+          |        INotificationChannel           |
                                              +--------------------------------------+
                                                          ^              ^
                                                +----------------+  +----------------+
                                                | EmailChannel   |  | SmsChannel     |
                                                +----------------+  +----------------+
```

### 1.2 “Favor Composition Over Inheritance” Is a Guideline

The Gang of Four design-patterns book helped popularize the guideline **“favor composition over inheritance.”** It isn't a rule against inheritance. Use inheritance when the derived type is a genuine, stable subtype that can honor the base type's contract. Use composition when independently changing behavior needs to be assembled, delegated, or selected.

A shared field or a convenient place to put code doesn't by itself establish an “is-a” relationship. If callers shouldn't be able to use a subtype wherever the base type is expected, inheritance may be the wrong reuse mechanism.

## 2. Why Inheritance Can Become Problematic

### 2.1 Fragile Base Classes

A base class is a contract for its subclasses. If derived classes rely on undocumented call order or internal behavior, a change to the base class—such as adding a virtual call or changing when a hook runs—can break subclasses that appeared unrelated to the change. Keep extension contracts small and documented, and avoid exposing virtual hooks unless inheritance is an intentional extension point.

### 2.2 Rigid Hierarchies and the Diamond Discussion

C# permits only one direct base class, so a type can't combine implementation from two separate class hierarchies. A model such as `FlyingBird` and `SwimmingBird` can become awkward when a penguin swims but doesn't fly, or a duck does both. Independent capabilities such as `IFlyBehavior` and `ISwimBehavior` can instead be composed where needed.

C# doesn't have the classic diamond of multiple class inheritance. A class can implement multiple interfaces; interface default implementations are a separate language feature, not multiple base-class inheritance.

### 2.3 Fixed Type Relationships vs. Selectable Collaborators

An object's class hierarchy is fixed by its type declaration and runtime type. Virtual dispatch can still choose an override at runtime, but the object's base-class relationship can't be replaced after construction.

Composition makes dependencies explicit and allows an implementation to be chosen when an object is constructed or when a dependency-injection (DI) scope is configured. Replacing a collaborator on a live object requires explicit support; a constructor-injected, read-only dependency isn't automatically hot-swappable. Per-tenant or per-request selection typically uses a policy selector or factory with deliberate service lifetimes.

## 3. When to Use Each Strategy

| Scenario | Often a good fit | Reasoning and cautions |
|---|---|---|
| A stable subtype relationship with substitutable behavior | **Inheritance** | Use a base class when derived instances can honor its contract and benefit from shared implementation. |
| Shared identity or audit fields on domain entities | **Depends on the model** | A small stable entity base can be useful. Audit metadata or capabilities don't automatically make every entity a subtype; composition, interfaces, or persistence configuration may fit better. |
| Swappable business rules such as discounts, payment gateways, or notification channels | **Composition** | Inject a strategy or collaborator so the consumer doesn't inherit unrelated behavior. |
| Reusing one algorithm across otherwise unrelated types | **Composition or a focused helper** | Avoid forcing unrelated types into the same class hierarchy just to share an implementation. |
| An extension point explicitly designed around a base class | **Inheritance** | For example, .NET hosted workers can derive from `BackgroundService`. Follow the framework's documented extension model. |
| ASP.NET Core middleware | **Usually composition/pipeline delegation** | Conventional middleware uses `RequestDelegate` and `Invoke`/`InvokeAsync`; it doesn't require inheriting from a middleware base class. |
| Service construction with .NET DI | **Composition** | DI builds object graphs. It can inject interfaces or concrete types; interfaces are common when implementations need to be replaceable, not mandatory. |

## 4. Basic Syntax Example

The inheritance design is concise for a small, stable family of report formats. If independent features such as encryption and compression are represented as subclasses in every combination, the number of classes can grow quickly. Composition keeps formatting and post-processing as separate choices.

```csharp
#nullable enable
using System;
using System.IO;
using System.IO.Compression;
using System.Text;

namespace MyBackendApp.Basics;

// --- INHERITANCE: a small report-format family ---
public abstract class ReportBase
{
    public abstract string Format(string content);
}

public sealed class PdfReport : ReportBase
{
    public override string Format(string content) => $"[PDF]: {content}";
}

public sealed class CsvReport : ReportBase
{
    public override string Format(string content) => $"[CSV]: {content}";
}

// These small formatters illustrate the design seam; they are not full
// PDF or CSV document serializers.
public interface IReportFormatter
{
    string Format(string content);
}

public interface ICompressor
{
    string Compress(string content);
}

public sealed class PdfLabelFormatter : IReportFormatter
{
    public string Format(string content) => $"[PDF]: {content}";
}

public sealed class CsvLabelFormatter : IReportFormatter
{
    public string Format(string content) => $"[CSV]: {content}";
}

// Compresses UTF-8 text with GZip, then encodes the compressed bytes as Base64.
public sealed class GzipBase64Compressor : ICompressor
{
    public string Compress(string content)
    {
        ArgumentNullException.ThrowIfNull(content);

        byte[] input = Encoding.UTF8.GetBytes(content);
        using var output = new MemoryStream();
        using (var gzip = new GZipStream(output, CompressionLevel.Fastest, leaveOpen: true))
        {
            gzip.Write(input);
        }

        return Convert.ToBase64String(output.ToArray());
    }
}

// --- COMPOSITION: the generator delegates to independently selected parts ---
public sealed class ReportGenerator
{
    private readonly IReportFormatter _formatter;
    private readonly ICompressor? _compressor;

    public ReportGenerator(IReportFormatter formatter, ICompressor? compressor = null)
    {
        ArgumentNullException.ThrowIfNull(formatter);
        _formatter = formatter;
        _compressor = compressor;
    }

    public string Generate(string content)
    {
        ArgumentNullException.ThrowIfNull(content);

        string formatted = _formatter.Format(content);
        return _compressor is null ? formatted : _compressor.Compress(formatted);
    }
}

// For example, select a formatter and optional compressor when composing:
// var generator = new ReportGenerator(new PdfLabelFormatter(), new GzipBase64Compressor());
```

The composition-based generator keeps its dependencies for its lifetime; to change them, create another generator or add an explicit selector/factory. The formatters are deliberately simplified labels, while `GzipBase64Compressor` performs actual compression and Base64 encoding.

## 5. Backend Example: Composable Order Processing

An order-processing service can delegate pricing and notification work to separate collaborators. This version chooses dependencies when the service is constructed. Selecting a different policy per tenant or request would require a policy resolver/factory or an appropriately scoped service; swapping constructor dependencies on an existing object isn't automatic.

```csharp
#nullable enable
using System;
using System.Threading;
using System.Threading.Tasks;

namespace MyBackendApp.Core.Domain.Orders;

public sealed record ShoppingCart(Guid CustomerId, decimal Subtotal, string Region);

public interface IDiscountStrategy
{
    decimal ApplyDiscount(decimal subtotal);
}

public interface IShippingCalculator
{
    decimal CalculateShippingCost(decimal subtotal, string region);
}

public interface IOrderNotifier
{
    Task NotifyAsync(Guid customerId, decimal finalTotal, CancellationToken cancellationToken);
}

public sealed class SeasonalDiscountStrategy : IDiscountStrategy
{
    private readonly decimal _percentOff;

    public SeasonalDiscountStrategy(decimal percentOff)
    {
        if (percentOff is < 0m or > 100m)
        {
            throw new ArgumentOutOfRangeException(nameof(percentOff));
        }

        _percentOff = percentOff;
    }

    public decimal ApplyDiscount(decimal subtotal)
    {
        if (subtotal < 0m)
        {
            throw new ArgumentOutOfRangeException(nameof(subtotal));
        }

        return subtotal - (subtotal * (_percentOff / 100m));
    }
}

public sealed class NoDiscountStrategy : IDiscountStrategy
{
    public decimal ApplyDiscount(decimal subtotal)
    {
        if (subtotal < 0m)
        {
            throw new ArgumentOutOfRangeException(nameof(subtotal));
        }

        return subtotal;
    }
}

public sealed class FlatRateShippingCalculator : IShippingCalculator
{
    public decimal CalculateShippingCost(decimal subtotal, string region)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(region);
        return region.Equals("DOMESTIC", StringComparison.OrdinalIgnoreCase) ? 5.00m : 25.00m;
    }
}

// Demo notifier: writes to the console rather than sending an actual email.
public sealed class ConsoleOrderNotifier : IOrderNotifier
{
    public Task NotifyAsync(Guid customerId, decimal finalTotal, CancellationToken cancellationToken)
    {
        cancellationToken.ThrowIfCancellationRequested();
        Console.WriteLine($"Order notification for customer {customerId}: total {finalTotal:F2}");
        return Task.CompletedTask;
    }
}

public sealed class OrderProcessingService
{
    private readonly IDiscountStrategy _discountStrategy;
    private readonly IShippingCalculator _shippingCalculator;
    private readonly IOrderNotifier _notifier;

    public OrderProcessingService(
        IDiscountStrategy discountStrategy,
        IShippingCalculator shippingCalculator,
        IOrderNotifier notifier)
    {
        ArgumentNullException.ThrowIfNull(discountStrategy);
        ArgumentNullException.ThrowIfNull(shippingCalculator);
        ArgumentNullException.ThrowIfNull(notifier);

        _discountStrategy = discountStrategy;
        _shippingCalculator = shippingCalculator;
        _notifier = notifier;
    }

    public async Task<decimal> ProcessCartAsync(
        ShoppingCart cart,
        CancellationToken cancellationToken = default)
    {
        ArgumentNullException.ThrowIfNull(cart);

        if (cart.Subtotal < 0m)
        {
            throw new ArgumentOutOfRangeException(nameof(cart), "Subtotal can't be negative.");
        }

        ArgumentException.ThrowIfNullOrWhiteSpace(cart.Region);

        decimal discountedSubtotal = _discountStrategy.ApplyDiscount(cart.Subtotal);
        if (discountedSubtotal < 0m || discountedSubtotal > cart.Subtotal)
        {
            throw new InvalidOperationException("Discount strategy returned an invalid subtotal.");
        }

        decimal shippingCost = _shippingCalculator.CalculateShippingCost(
            discountedSubtotal, cart.Region);
        if (shippingCost < 0m)
        {
            throw new InvalidOperationException("Shipping calculator returned a negative cost.");
        }

        decimal finalTotal = checked(discountedSubtotal + shippingCost);
        await _notifier.NotifyAsync(cart.CustomerId, finalTotal, cancellationToken);
        return finalTotal;
    }
}

// Two compositions of the same orchestrator, selected at construction time:
// var holiday = new OrderProcessingService(
//     new SeasonalDiscountStrategy(20m), new FlatRateShippingCalculator(), new ConsoleOrderNotifier());
// var standard = new OrderProcessingService(
//     new NoDiscountStrategy(), new FlatRateShippingCalculator(), new ConsoleOrderNotifier());
```

The interfaces make each collaborator replaceable in tests and at the composition root. The example uses simplified monetary rules: it omits currency, rounding, tax, persistence, and reliable notification delivery. A production order workflow should define those policies and handle order/notification consistency deliberately.

## 6. Design Guidance and Common Mistakes

- **Using inheritance just to reuse a few lines:** inheritance exposes a subtype relationship and couples the derived class to the base contract. Prefer a helper or collaborator when the types aren't genuine substitutes.
- **Treating composition as automatic live reconfiguration:** dependencies can be selected at construction or resolved through an explicit policy selector; a read-only dependency won't change by itself.
- **Assuming all composition must use interfaces:** interfaces are useful seams, but a small stable concrete dependency can also be composed.
- **Assuming DI requires interfaces:** the built-in .NET container can construct concrete types too. Choose abstractions when they clarify a boundary or allow alternative implementations.
- **Putting cross-cutting capabilities in a base class by default:** audit fields, logging, caching, and similar concerns don't always define an “is-a” relationship. Consider decorators, services, interfaces, or persistence configuration.
- **Extending a framework class without checking its contract:** inherit when the framework documents that base class as an extension point, such as `BackgroundService`; don't invent framework base classes for convenience.
- **Assuming ASP.NET Core middleware uses a base class:** conventional middleware normally delegates through `RequestDelegate` and `Invoke`/`InvokeAsync`.
- **Claiming composition is always better:** composition adds collaborators and indirection. A small, stable, substitutable hierarchy can be clearer.
- **Ignoring substitutability:** before inheriting, ask whether every instance of the derived type can safely be used wherever the base type is expected.
- **Calling construction-time selection “hot swapping”:** examples that pass different strategies to two newly created services demonstrate alternative composition, not mutation of a running service.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Inheritance (“is-a”)** | A class derives from one base class and reuses or specializes its behavior. | Models stable subtypes whose instances honor the base type's contract. |
| **Composition (“has-a” / “uses-a”)** | An object collaborates with other objects and delegates work to them. | Assembles independent behaviors and makes dependencies replaceable at composition boundaries. |
| **Fragile base class problem** | Base-class changes can break subclasses that depend on its internal behavior or call order. | Encourages narrow, documented inheritance contracts. |
| **Favor composition over inheritance** | A design guideline to prefer delegation for flexible behavior reuse, without banning valid subtype hierarchies. | Helps avoid tightly coupled or combinatorial subclass trees. |
| **Strategy pattern** | A composition-based pattern that delegates an operation to one of several interchangeable algorithms. | Supports swappable discount, shipping, or notification policies. |
| **Delegation** | Forwarding work from one object to a collaborator. | The mechanism through which composition reuses behavior. |
| **Liskov Substitution Principle (LSP)** | A subtype should be usable wherever its base type is expected without breaking the base contract. | A key test for whether inheritance represents a sound subtype relationship. |
| **Dependency injection (DI)** | Supplying an object's dependencies from outside instead of constructing them inside that object. | Separates object-graph construction from business behavior and simplifies substitutions. |

## 8. Official References

- [Inheritance — C# fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance) — base and derived types, single class inheritance, and specialization.
- [Interfaces — C# fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces) — interface contracts and multiple interface implementation.
- [.NET dependency injection](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/overview) — dependency graphs, registrations, and constructor injection.
- [Write custom ASP.NET Core middleware for .NET 10](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write?view=aspnetcore-10.0) — middleware conventions, `RequestDelegate`, and `InvokeAsync`.
- [Worker services in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/workers) — `BackgroundService` as a documented base-class extension point.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — language-version defaults, including .NET 10 / C# 14.
