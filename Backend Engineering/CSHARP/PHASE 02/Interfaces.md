Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 What Is an Interface?

An **interface** is a reference type that defines a contract of members such as methods, properties, events, and indexers. A class or struct can implement that contract, and callers can use the implementation through the interface type.

An interface doesn't hold per-instance fields or have instance constructors. Modern C# interfaces can also provide default member implementations and declare certain static members, so “interface = signatures only” is no longer a complete description.

```text
       +------------------------------------+
       |       <<interface>>                |
       |       INotificationService         |
       |  SendAsync(message, recipient)      |
       +------------------------------------+
                         ^
            implements   |   implements
        +----------------+----------------+
        |                                 |
+----------------------+        +----------------------+
| TwilioSmsService     |        | SmtpEmailService     |
| provider-specific    |        | provider-specific    |
+----------------------+        +----------------------+
```

Interfaces are useful in backend systems because they can:

- **Support the Dependency Inversion Principle (DIP):** high-level policy and low-level implementation details depend on abstractions, rather than business logic being tied to a specific database or provider. The abstraction should belong to the policy it serves.
- **Reduce coupling and aid testing:** a caller can receive a production adapter, a lightweight fake, or another implementation through the same contract. An interface isn't required just to use a mocking library; introduce one when it represents a useful boundary.
- **Allow multiple capabilities:** a class can derive from one class and implement multiple interfaces; an interface can also inherit from multiple interfaces. Structs can implement interfaces too.

**Dependency Injection (DI)** is a way to supply dependencies—often through constructor parameters and a service container. DI is a composition technique that can help apply DIP; it isn't the same principle as DIP.

## 2. Interface Features in C#

### 2.1 Default Interface Implementations

Since C# 8, an interface can provide a body for a method or other supported member. An implementing type can use that default or provide its own implementation. This can help library authors add a member without requiring every existing implementer to write it, when the target runtime supports default interface members.

```csharp
#nullable enable
using System;

public interface IAuditableLogger
{
    void Log(string message);

    // Default implementation supplied by the interface.
    public void LogWarning(string message)
    {
        Log($"[WARNING] {message}");
    }
}

public sealed class ConsoleLogger : IAuditableLogger
{
    public void Log(string message)
    {
        ArgumentNullException.ThrowIfNull(message);
        Console.WriteLine(message);
    }
}

public static class DefaultInterfaceDemo
{
    public static void Main()
    {
        IAuditableLogger logger = new ConsoleLogger();
        logger.LogWarning("Retrying the request.");
    }
}
```

The default `LogWarning` implementation is available through an `IAuditableLogger` reference. It doesn't automatically become a normal public member of `ConsoleLogger`; `ConsoleLogger` would need to declare its own `LogWarning` if callers should use that member through the concrete type. Default interface members help evolve contracts, but they still require compatible compiler/runtime support and careful versioning.

### 2.2 Explicit Interface Implementation

A class can implement an interface member **explicitly** by qualifying the member name with the interface name. An explicit implementation has no access modifier and is callable through that interface—not as a normal member on the class. It's useful when two interfaces have the same member signature or when an implementation shouldn't be part of the class's general public API.

```csharp
#nullable enable
using System;

public interface IOrderProcessor
{
    void Process();
}

public interface IPaymentProcessor
{
    void Process();
}

public sealed class TransactionManager : IOrderProcessor, IPaymentProcessor
{
    void IOrderProcessor.Process() => Console.WriteLine("Processing order...");
    void IPaymentProcessor.Process() => Console.WriteLine("Processing payment...");
}

public static class ExplicitImplementationDemo
{
    public static void Main()
    {
        var manager = new TransactionManager();

        // manager.Process(); // Compile-time error: no public Process member on the class.
        ((IOrderProcessor)manager).Process();
        ((IPaymentProcessor)manager).Process();
    }
}
```

The two interface calls select different implementations. Explicit implementations can still be invoked from code that holds the object through the matching interface type.

### 2.3 Static Abstract Members in Interfaces

C# 11 introduced `static abstract` interface members. They let a generic algorithm require static operations—such as a factory method or arithmetic operator—from a type argument. Unlike instance virtual dispatch, these calls are resolved by the compiler using the generic constraint and type argument. .NET 7 introduced runtime support; .NET 10 supports the feature.

```csharp
#nullable enable

public interface IEntityFactory<TSelf>
    where TSelf : IEntityFactory<TSelf>
{
    static abstract TSelf CreateDefault();
}

public sealed record Customer(string Name) : IEntityFactory<Customer>
{
    public static Customer CreateDefault() => new("New customer");
}

public static class EntityFactory
{
    public static TSelf CreateDefault<TSelf>()
        where TSelf : IEntityFactory<TSelf>
        => TSelf.CreateDefault();
}

public static class StaticAbstractDemo
{
    public static void Main()
    {
        Customer customer = EntityFactory.CreateDefault<Customer>();
        System.Console.WriteLine(customer.Name);
    }
}
```

The static contract is called through the constrained type parameter `TSelf`, not through an interface instance.

## 3. Interface vs. Abstract Class

| Dimension | Interface | Abstract class |
|---|---|---|
| **Inheritance model** | A class or struct can implement multiple interfaces. | A class can derive from only one base class and can also implement interfaces. |
| **Instance state** | Can't declare instance fields; can include default implementations and certain static members. | Can hold instance fields and shared state. |
| **Constructors** | Can't declare instance constructors. | Can declare constructors that initialize base state. |
| **Implementation** | Defines a capability or contract; default implementations are available for supported members. | Can combine implemented members with required abstract members. |
| **Typical use** | Cross-cutting capability or port, such as a payment gateway contract. | Shared state or protected workflow for a cohesive family of related classes. |

“Can-do” for an interface and “is-a” for an abstract class are useful design heuristics, not rigid rules. Use the smallest abstraction that expresses what the caller needs; don't choose an interface or a base class merely because it is a familiar pattern.

## 4. Basic Syntax Example: Multiple and Explicit Implementations

This complete source file shows a class implementing two interfaces. `Document.Print()` is the implicit `IPrintable` implementation; `IExportable.Print()` is explicit, so callers choose it through an `IExportable` reference.

```csharp
#nullable enable
using System;
using System.Text;

namespace MyBackendApp.Basics;

public interface IPrintable
{
    void Print();
}

public interface IExportable
{
    void Print();
    byte[] ExportBytes();
}

public sealed class Document : IPrintable, IExportable
{
    public string Content { get; init; } = string.Empty;

    // Implicit implementation of IPrintable.Print.
    public void Print()
    {
        Console.WriteLine($"[Document Content]: {Content}");
    }

    // Explicit implementation of IExportable.Print.
    void IExportable.Print()
    {
        Console.WriteLine($"[Export Queue]: {Content}");
    }

    public byte[] ExportBytes() => Encoding.UTF8.GetBytes(Content);
}

public static class DocumentDemo
{
    public static void Main()
    {
        var document = new Document { Content = "Quarterly report" };

        document.Print();
        ((IExportable)document).Print();
        byte[] bytes = ((IExportable)document).ExportBytes();
        Console.WriteLine($"Exported {bytes.Length} bytes.");
    }
}
```

A public `Print()` with the same signature would normally implement both interfaces. The explicit `IExportable.Print()` provides a separate implementation for that interface, while `IPrintable.Print()` remains directly available on `Document`.

## 5. Applied Backend Example: Payment and Infrastructure Ports

In a Ports and Adapters (Hexagonal) design, application code can depend on interfaces for outbound systems—such as payment gateways, caches, or an outbox dispatcher—while infrastructure projects provide the concrete adapters. Constructor injection makes the dependency visible and allows tests to supply a fake.

The example uses a cache only as a fast path for previously completed successful results. It does **not** treat cache `Get` followed by `Set` as an atomic payment lock.

```csharp
#nullable enable
using System;
using System.Threading;
using System.Threading.Tasks;

namespace MyBackendApp.Core.Interfaces
{
    public sealed record PaymentRequest(
        Guid TransactionId,
        string IdempotencyKey,
        decimal Amount,
        string Currency,
        string CustomerReference);

    public sealed record PaymentResult(
        bool IsSuccess,
        string? GatewayTransactionId,
        string? ErrorCode = null);

    // Port implemented by Stripe, PayPal, or another payment adapter.
    public interface IPaymentGatewayService
    {
        string ProviderCode { get; }

        // The adapter should pass request.IdempotencyKey to the provider when supported.
        Task<PaymentResult> ProcessChargeAsync(
            PaymentRequest request,
            CancellationToken cancellationToken = default);

        Task<PaymentResult> RefundChargeAsync(
            string gatewayTransactionId,
            decimal amount,
            CancellationToken cancellationToken = default);
    }

    public interface ICacheService
    {
        // This class-only cache contract uses null to signal a cache miss.
        Task<T?> GetAsync<T>(string cacheKey, CancellationToken cancellationToken = default)
            where T : class;

        Task SetAsync<T>(
            string cacheKey,
            T value,
            TimeSpan expiration,
            CancellationToken cancellationToken = default)
            where T : class;

        Task RemoveAsync(string cacheKey, CancellationToken cancellationToken = default);
    }

    // A separate port normally consumed by a background worker.
    public interface IOutboxMessageDispatcher
    {
        Task DispatchPendingAsync(CancellationToken cancellationToken = default);
    }
}

namespace MyBackendApp.Core.Services
{
    using MyBackendApp.Core.Interfaces;

    public sealed class BillingOrchestrator
    {
        private readonly IPaymentGatewayService _paymentGateway;
        private readonly ICacheService _cacheService;

        public BillingOrchestrator(
            IPaymentGatewayService paymentGateway,
            ICacheService cacheService)
        {
            ArgumentNullException.ThrowIfNull(paymentGateway);
            ArgumentNullException.ThrowIfNull(cacheService);

            _paymentGateway = paymentGateway;
            _cacheService = cacheService;
        }

        public async Task<PaymentResult> ExecuteBillingAsync(
            PaymentRequest request,
            CancellationToken cancellationToken = default)
        {
            ArgumentNullException.ThrowIfNull(request);
            ArgumentException.ThrowIfNullOrWhiteSpace(request.IdempotencyKey);
            ArgumentException.ThrowIfNullOrWhiteSpace(request.Currency);
            ArgumentException.ThrowIfNullOrWhiteSpace(request.CustomerReference);

            if (request.TransactionId == Guid.Empty)
            {
                throw new ArgumentException("Transaction ID cannot be empty.", nameof(request));
            }

            if (request.Amount <= 0m)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(request), request.Amount, "Payment amount must be positive.");
            }

            string cacheKey = $"payment-result:{request.IdempotencyKey}";
            PaymentResult? cachedResult =
                await _cacheService.GetAsync<PaymentResult>(cacheKey, cancellationToken);

            if (cachedResult is not null)
            {
                return cachedResult;
            }

            // The adapter should pass this stable key to a provider that supports idempotency.
            PaymentResult result = await _paymentGateway.ProcessChargeAsync(request, cancellationToken);

            if (result.IsSuccess)
            {
                await _cacheService.SetAsync(
                    cacheKey,
                    result,
                    TimeSpan.FromHours(24),
                    cancellationToken);
            }

            return result;
        }
    }
}
```

At the composition root, register a concrete adapter for `IPaymentGatewayService`; tests can provide a fake implementing the same interface. `IOutboxMessageDispatcher` would normally be injected into a separate background worker. Its implementation must define how messages are claimed, retried, and marked as dispatched.

**Idempotency caveat:** two concurrent requests can both miss the cache before either stores a result. Use one stable idempotency key per logical payment request, and don't reuse it with a different payload. Pass it to a provider that supports idempotent requests; durable systems may also need an atomic database record or uniqueness constraint. Cache success results as an optimization—not as the correctness mechanism that prevents double charges. Production payment code also needs provider-specific response handling, transaction reconciliation, secure handling of customer data, and explicit currency and amount policies.

## 6. Design Guidance and Common Mistakes

- **Treating every interface as a mock hook:** define an interface when it represents a useful contract or boundary; small deterministic classes can be tested without a separate interface.
- **Confusing DI with DIP:** DI supplies dependencies; DIP is about the direction of source-code dependencies toward abstractions.
- **Assuming interfaces contain only signatures:** default implementations and static members exist, but interfaces still can't carry per-instance fields or instance constructors.
- **Calling a default interface method on the concrete class:** a default member is available through the interface contract unless the class declares its own member.
- **Using explicit implementation unnecessarily:** it hides the member from the class's public surface; use it for collisions or intentional API boundaries.
- **Calling static abstract members through an interface instance:** use a constrained generic type parameter instead.
- **Treating cache read-then-write as an atomic lock:** concurrent calls can race. Use an actual atomic/durable idempotency mechanism where correctness depends on it.
- **Putting infrastructure implementations in the core contract project:** keep the interface at the policy boundary and put provider-specific adapters in the appropriate infrastructure/composition project.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Interface** | A reference-type contract implemented by classes or structs. | Lets callers depend on required behavior rather than a concrete implementation. |
| **Dependency Inversion Principle (DIP)** | High- and low-level modules depend on abstractions owned by policy, not directly on implementation details. | Keeps domain/application decisions independent of infrastructure choices. |
| **Dependency Injection (DI)** | A technique for supplying an object's dependencies, often through constructor parameters and a service container. | Makes implementation selection explicit and supports composition and testing. |
| **Default interface implementation** | A supported interface member with a provided body. | Can add behavior without requiring every existing implementer to provide it. |
| **Explicit interface implementation** | An implementation callable through a particular interface reference. | Resolves member collisions or keeps a member off the concrete type's public API. |
| **Static abstract interface member** | A static member contract that a generic type argument must implement. | Enables generic factories and mathematical algorithms without instance dispatch. |
| **Port and adapter** | A core-owned interface (port) and an external implementation (adapter). | Separates application policy from databases, payment providers, caches, and brokers. |
| **Idempotency key** | A stable request identifier used to recognize retries of the same operation. | Helps prevent duplicate side effects when a provider and the system honor it correctly. |

## 8. Official References

- [Interfaces — C# fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces) — interface contracts, implementations, and comparison with abstract classes.
- [`interface` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface) — interface members and default implementations.
- [Static abstract and virtual interface members — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface#static-abstract-and-virtual-members) — compile-time static contracts for generic algorithms.
- [Safely update interfaces with default interface methods](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interface-implementation/default-interface-methods-versions) — default implementation and interface versioning.
- [Explicit interface implementation — C# programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/explicit-interface-implementation) — syntax and interface-only member access.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — default language versions, including .NET 10 / C# 14.
- [.NET dependency injection](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/overview) — constructor injection, registration, and service-container behavior.
- [Dependency Inversion Principle — .NET architecture](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#dependency-inversion) — the dependency direction underlying DIP.
- [`ArgumentException.ThrowIfNullOrWhiteSpace` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception.throwifnullorwhitespace?view=net-10.0) — input guard used in the examples.
