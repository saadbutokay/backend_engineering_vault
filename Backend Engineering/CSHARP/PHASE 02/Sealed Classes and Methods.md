Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 What Does `sealed` Do?

The `sealed` modifier closes an inheritance extension point:

- On a **class**, it prevents another class from deriving from that class.
- On an **overridden method or property**, `sealed override` prevents a more-derived class from overriding that member again. The containing class can still be derived from, and other virtual members can still be overridden.

```text
       +---------------------------------------------+
       |                 BaseProcessor               |
       |  virtual Validate()     virtual Execute()   |
       +---------------------------------------------+
                              ^
                              | derives from
       +---------------------------------------------+
       |             SecureOrderProcessor            |
       |  sealed override Validate()                 |
       |  override Execute()                         |
       +---------------------------------------------+
                              ^
                              | may still derive
       +---------------------------------------------+
       |             CustomOrderProcessor            |
       |  cannot override Validate()                 |
       |  can override Execute()                     |
       +---------------------------------------------+

       +---------------------------------------------+
       |       sealed class FinalPipeline            |
       |       no class can derive from it           |
       +---------------------------------------------+
```

A method or property can be marked `sealed` only when it is an `override`; a new member is made non-overridable by not declaring it `virtual`. Structs are already sealed by the language and can't be used as base classes.

## 2. Why Seal Classes or Members?

### 2.1 Intent and Invariants

Sealing communicates that inheritance isn't a supported extension point. It can help keep a value object's equality rules or a carefully designed base-class workflow from being changed by a subclass. However, `sealed` doesn't make an object immutable, validate its inputs, or form a complete security boundary. Security still depends on correct validation, access control, trusted dependencies, and application configuration. A sealed service that implements an interface can still be replaced by registering a different implementation.

### 2.2 JIT Optimization: Possible, Not Guaranteed

A sealed class or a `sealed override` can give the just-in-time (JIT) compiler more information about possible runtime types. In some call sites, that information can enable **devirtualization**—resolving a virtual call to a known implementation—and may make inlining or further optimization possible.

Don't assume that every virtual call incurs a fixed vtable lookup, that sealing automatically removes dispatch, or that a particular method will be inlined. The JIT can also devirtualize calls when a type is known from other context, and inlining depends on size, call-site information, runtime version, and other heuristics. Profile and benchmark real workloads before sealing types for performance.

## 3. Sealing an Overridden Member

A member must first be declared `virtual` or `abstract` in a base class. A derived class can override it and mark that override `sealed` so that further derived classes inherit that implementation but can't replace it.

```csharp
#nullable enable
using System;

public class BaseService
{
    public virtual void Run() { }
}

public class IntermediateService : BaseService
{
    // Further subclasses inherit this implementation but cannot override it.
    public sealed override void Run()
    {
        Console.WriteLine("Locked implementation.");
    }
}

public class FinalService : IntermediateService
{
    // Compile-time error if uncommented: the inherited override is sealed.
    // public override void Run() { }
}
```

`FinalService` can still inherit other base behavior and override other virtual members that weren't sealed. The `sealed` modifier closes only the indicated override point.

## 4. Basic Syntax Example: Sealed Class and Sealed Override

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public class DocumentRenderer
{
    public virtual void RenderHeader() => Console.WriteLine("Standard header");
    public virtual void RenderBody() => Console.WriteLine("Standard body");
}

public class StrictPdfRenderer : DocumentRenderer
{
    public sealed override void RenderHeader() =>
        Console.WriteLine("Strict PDF compliance header");

    public override void RenderBody() => Console.WriteLine("Strict PDF body");
}

public sealed class HighSecurityDocumentExporter : StrictPdfRenderer
{
    // Allowed: RenderBody remains overridable.
    public override void RenderBody() =>
        Console.WriteLine("Encrypted high-security PDF body");

    // Compile-time error if uncommented: StrictPdfRenderer sealed this override.
    // public override void RenderHeader() { }
}

// Compile-time error if uncommented: HighSecurityDocumentExporter is sealed.
// public class CustomExporter : HighSecurityDocumentExporter { }
```

`HighSecurityDocumentExporter` can't be subclassed, while the `RenderHeader` override was already sealed one level earlier. Sealing a class and sealing one override are separate decisions.

## 5. Applied Backend Example: Value Object and Token-Validation Adapter

### 5.1 A Sealed `Money` Value Object

A value object can be sealed when its equality and arithmetic rules aren't intended to support derived variants. This example uses a deliberately simple nonnegative amount policy and checks that the currency has a three-letter ASCII shape; it doesn't verify that the code is supported or define currency-specific precision.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Core.Domain.ValueObjects
{
    public sealed class Money : IEquatable<Money>
    {
        public decimal Amount { get; }
        public string Currency { get; }

        public Money(decimal amount, string currency)
        {
            if (amount < 0m)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(amount), amount, "This sample doesn't allow negative amounts.");
            }

            ArgumentException.ThrowIfNullOrWhiteSpace(currency);
            string normalizedCurrency = currency.Trim().ToUpperInvariant();

            if (normalizedCurrency.Length != 3)
            {
                throw new ArgumentException(
                    "Currency must have three letters in this sample.", nameof(currency));
            }

            foreach (char character in normalizedCurrency)
            {
                if (!char.IsAsciiLetter(character))
                {
                    throw new ArgumentException(
                        "Currency must contain only ASCII letters.", nameof(currency));
                }
            }

            Amount = amount;
            Currency = normalizedCurrency;
        }

        public Money Add(Money other)
        {
            ArgumentNullException.ThrowIfNull(other);

            if (!Currency.Equals(other.Currency, StringComparison.Ordinal))
            {
                throw CreateCurrencyMismatchException(Currency, other.Currency);
            }

            return new Money(checked(Amount + other.Amount), Currency);
        }

        private static InvalidOperationException CreateCurrencyMismatchException(
            string leftCurrency,
            string rightCurrency) =>
            new($"Cannot add mismatched currencies: '{leftCurrency}' and '{rightCurrency}'.");

        public bool Equals(Money? other)
        {
            if (other is null)
            {
                return false;
            }

            if (ReferenceEquals(this, other))
            {
                return true;
            }

            return Amount == other.Amount && Currency == other.Currency;
        }

        public override bool Equals(object? obj) => obj is Money other && Equals(other);

        public override int GetHashCode() => HashCode.Combine(Amount, Currency);
    }
}
```

The class is sealed, and its public state is immutable after construction. Sealing alone wouldn't make a mutable object immutable; the read-only properties and absence of mutating methods are also important. A production money type should use an explicit supported-currency policy, scale/rounding rules, and a documented policy for credits, refunds, and negative balances.

### 5.2 A Sealed Adapter Around JWT Validation

Sealing a token-validation adapter can communicate that inheritance isn't its extension mechanism. It does **not** make token validation secure by itself. The following sketch delegates to a vetted verifier; it intentionally does not implement JWT parsing or cryptography.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Core.Interfaces
{
    public interface ITokenValidationPipeline
    {
        bool ValidateToken(string rawToken, out string? subjectId);
    }
}

namespace MyBackendApp.Infrastructure.Security
{
    using MyBackendApp.Core.Interfaces;

    // Implement this port with an approved JWT library or framework component.
    public interface IJwtTokenVerifier
    {
        bool TryValidate(string rawToken, out string? subjectId);
    }

    public sealed class JwtTokenValidationAdapter : ITokenValidationPipeline
    {
        private readonly IJwtTokenVerifier _verifier;

        public JwtTokenValidationAdapter(IJwtTokenVerifier verifier)
        {
            ArgumentNullException.ThrowIfNull(verifier);
            _verifier = verifier;
        }

        public bool ValidateToken(string rawToken, out string? subjectId)
        {
            ArgumentException.ThrowIfNullOrWhiteSpace(rawToken);
            return _verifier.TryValidate(rawToken, out subjectId);
        }
    }
}
```

The adapter can't be subclassed, but it can still be replaced at the composition root or by a test double through `ITokenValidationPipeline`. For an ASP.NET Core API, prefer the framework's JWT bearer authentication and a vetted token-validation library over hand-written parsing. A token having three dot-separated segments is not proof that its signature, issuer, audience, lifetime, or claims are valid.

## 6. Design Guidance and Common Mistakes

- **Sealing everything for speed:** `sealed` can help the JIT in some contexts, but isn't a performance guarantee. Measure before making performance-driven API decisions.
- **Treating `sealed` as a security boundary:** it blocks subclassing, not dependency replacement, incorrect validation, reflection, or other application-level risks.
- **Assuming sealed means immutable:** control state through private/read-only members and validate values independently of inheritance restrictions.
- **Sealing ORM entities without checking proxy requirements:** EF Core lazy-loading proxies require entity classes that can be inherited from and virtual navigation properties. Sealed entities won't work with that proxy strategy; other EF Core configurations may still use sealed types.
- **Sealing a method before it is an override:** use `sealed override`; a new method is simply non-virtual unless declared otherwise.
- **Sealing a whole class when only one hook is fixed:** seal the specific override if the rest of the type is intentionally extensible.
- **Calling inlining guaranteed:** inlining is a JIT decision based on the call site and runtime heuristics, not a promise made by `sealed`.
- **Treating “default to sealed” as a law:** sealing is a good default for types not designed for inheritance, but public extension points and framework requirements may need an open class.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **`sealed` class** | A class that can't be used as a base class. | States that inheritance isn't a supported extension point. |
| **`sealed override`** | An override that further-derived classes can't override. | Closes one virtual extension point while leaving other members open. |
| **Devirtualization** | An optimization that resolves a virtual call to a known target when the runtime can prove it is safe. | Can reduce dispatch overhead and enable other optimizations in suitable call sites. |
| **Inlining** | A JIT optimization that substitutes a method body at a call site when profitable. | May reduce call overhead and enable further optimizations; not guaranteed by sealing. |
| **Value object** | A domain object whose identity comes from its values rather than a persistent identifier. | Often sealed when derived variants aren't part of the value-equality design. |
| **Extension point** | A method or type intentionally designed for consumers or subclasses to customize. | Helps decide which classes or overrides should remain unsealed. |

## 8. Official References

- [`sealed` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed) — sealed classes and sealed overrides.
- [`override` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override) — rules for overriding inherited members.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — default language versions, including .NET 10 / C# 14.
- [CA1859: Use concrete types when possible](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1859) — virtual/interface dispatch and generated-code performance considerations.
- [What's new in the .NET 10 runtime](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/runtime) — JIT devirtualization and inlining improvements in .NET 10.
- [EF Core lazy loading](https://learn.microsoft.com/en-us/ef/core/querying/related-data/lazy) — proxy requirements for inheritable entities and virtual navigation properties.
- [Configure JWT bearer authentication in ASP.NET Core 10](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication?view=aspnetcore-10.0) — framework-based bearer-token validation.
- [`ArgumentException.ThrowIfNullOrWhiteSpace` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception.throwifnullorwhitespace?view=net-10.0) — input guard used by the examples.
- [`Char.IsAsciiLetter` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.char.isasciiletter?view=net-10.0) — currency-code shape check used by the value-object sample.
