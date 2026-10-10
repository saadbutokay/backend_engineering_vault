Examples target **.NET 10** with nullable reference analysis enabled. .NET 10 normally defaults to C# 14 unless `LangVersion` is overridden. The `file` modifier was introduced in C# 11 and applies to top-level types; the other examples use long-established C# accessibility rules. Code blocks are self-contained unless explicitly labeled as a fragment or project-file excerpt.

## 1. Conceptual Foundation

**Access modifiers** control where source code can name and use a type or member. They help define an API boundary and support encapsulation: callers should depend on the operations a type intentionally exposes, not on every detail of its implementation.

Access modifiers are **not a security mechanism** for protecting secrets from reflection, privileged code, or other runtime inspection. Don't store credentials in source code or rely on `private` or `internal` as a security boundary.

### Assemblies, Projects, and Accessibility

`internal` means **the current assembly**, not strictly “the current project” or “the current folder.” A .NET assembly is typically the compiled `.dll` or `.exe` produced by a project. A project usually produces one assembly, but assembly names and project boundaries can differ; `InternalsVisibleTo` can also grant access to a friend assembly.

A declared accessibility is also constrained by the accessibility of its containing type. For example, a `public` method on an `internal` class is still usable only by code that can access that class. A member's parameter and return types generally must be at least as accessible as the member itself.

## 2. The Accessibility Levels

C# has six conventional accessibility levels for members and types: `public`, `private`, `protected`, `internal`, `protected internal`, and `private protected`. C# 11 also introduced `file` for **file-local top-level types**. `file` is not a general member modifier and can't be combined with another accessibility modifier.

### 2.1 `public`

- **Scope:** No restriction of its own; any code that can access the containing type can access the public member or type.
- **Typical use:** Supported library APIs, application contracts, and members intentionally exposed to callers.
- **Caution:** A public member on an internal type doesn't make that member public outside the containing type's accessibility domain.

### 2.2 `private`

- **Scope:** The declaring type and its nested types.
- **Typical use:** Backing fields, implementation helpers, and state that shouldn't be accessed directly by callers or derived types.
- A derived class doesn't gain access to a base class's private members merely by inheriting from it.

### 2.3 `protected`

- **Scope:** The declaring type and types derived from it, including derived types in other assemblies.
- **Typical use:** A deliberately extensible base-class API, such as a protected hook for derived implementations.
- **Receiver rule:** When a derived class accesses a protected instance member through an object, the object's compile-time type must be that derived class or a type derived from it—not an arbitrary instance typed as the base class. This rule applies to the protected part of `protected internal` as well.

### 2.4 `internal`

- **Scope:** Code in the current assembly.
- **Typical use:** Implementation shared across layers or namespaces in one assembly, but not intended as part of the public contract for other assemblies.
- `internal` can be accessed by non-derived code in the assembly. It is often a better fit than `protected` when inheritance isn't the reason to share a member.

### 2.5 `protected internal` — Union

- **Scope:** Code in the current assembly **or** a type derived from the declaring type, including a derived type in another assembly.
- **Typical use:** A library member that should be available to all code in its own assembly and also to external subclasses.
- In another assembly, access follows the `protected` receiver rule; the `internal` branch doesn't apply there.

### 2.6 `private protected` — Intersection

- **Scope:** The declaring type and derived types **within** the current assembly; for derived types, assembly and inheritance requirements both apply.
- **Typical use:** An inheritance hook intended only for a hierarchy maintained inside one assembly.
- A non-derived type in the same assembly can't access it; a derived type in another assembly can't access it either. A friend assembly granted by `InternalsVisibleTo` is treated as part of the accessibility domain for internal access, but the derivation requirement still applies.

### 2.7 `file` — File-Local Top-Level Type

- **Scope:** The source file where the top-level type is declared. Other files in the same namespace and assembly still can't name that type.
- **Typical use:** Source-generated helper types or a helper that is genuinely private to one source file.
- `file` applies to a top-level type declaration, not to a field, method, or property. It can't be combined with `public`, `internal`, or another accessibility modifier. A `public` member inside a file-local type is still limited by the containing type's file-local scope.
- A file-local type can't be used as the base type of a non-file-local type, or leak into a member signature or field of a non-file-local type. A more visible type can use a file-local helper internally, but shouldn't expose it as part of its API.

The `protected internal` and `private protected` combinations are not synonyms: `internal` joins `protected internal` with **OR**, while `private protected` requires **both** conditions.

The table assumes a different assembly isn't a friend assembly. `InternalsVisibleTo` broadens internal access as described in Section 4.

| Calling code is… | `public` | `private` | `protected` | `internal` | `protected internal` | `private protected` |
|---|---:|---:|---:|---:|---:|---:|
| In the declaring type (or a nested type) | Yes | Yes | Yes | Yes | Yes | Yes |
| Same assembly, unrelated type | Yes | No | No | Yes | Yes | No |
| Same assembly, derived type | Yes | No | Yes | Yes | Yes | Yes |
| Different assembly, unrelated type | Yes* | No | No | No | No | No |
| Different assembly, derived type | Yes* | No | Yes | No | Yes | No** |

\* Only if the containing type is itself accessible.  
\** A friend assembly changes internal access; a derived type in that friend assembly can satisfy `private protected`'s two conditions.

For a file-local type, the separate rule is simpler: code in the **same source file** can name it; code in another file cannot, regardless of assembly or inheritance. A top-level `file` type is either file-local or has an ordinary accessibility modifier—never both.

### Protected Receiver Example

This **fragment** shows the receiver restriction for `protected` access. `Derived` may use the member through itself or another `Derived`, but not through an arbitrary variable whose type is `Base`:

```csharp
public class Base
{
    protected int Value = 42;
}

public sealed class Derived : Base
{
    public int ReadValues(Derived other, Base baseReference)
    {
        int ownValue = Value;
        int otherValue = other.Value;
        // int invalid = baseReference.Value; // Compile-time error (CS1540).
        return ownValue + otherValue;
    }
}
```

## 3. Default Accessibility Rules

The default depends on the declaration's context. **Top-level** types are different from nested types, and interface members have different defaults from class members.

| Declaration context | Default | Notes on declared accessibility |
|---|---|---|
| Top-level type (class, struct, interface, record, enum, delegate) | `internal` | Usually `public` or `internal`; an eligible top-level type can instead use `file`. `file` can't be combined with another accessibility modifier. |
| Member of a class or record class | `private` | All six conventional levels are available where the member kind permits them. |
| Member of a struct or record struct | `private` | `public`, `private`, and `internal` are available; structs have no derived classes, so protected forms don't apply. |
| Member of an interface | `public` | Modern C# allows additional access modifiers for supported interface members. A `private` interface member must have an implementation; the exact choices depend on the member form. |
| Enum member | Implicitly public | Enum members have no declared accessibility modifier. Their accessibility follows the enum type. |
| Nested type in a class or struct | `private` | Its effective accessibility can't exceed that of its containing type. |
| Nested type in an interface | `public` | Its effective accessibility is still constrained by its containing type. |

For example, a top-level `class` with no modifier is `internal`, while a method declared inside a class with no modifier is `private`. An interface method without an access modifier is public by default:

```csharp
#nullable enable
using System;

public interface IAuditSink
{
    void Write(string message); // Public abstract member by default.

    void WriteNormalized(string message) // Public default implementation.
    {
        Write(Normalize(message));
    }

    private static string Normalize(string message)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(message);
        return message.Trim();
    }
}
```

## 4. Friend Assemblies and Unit Tests: `InternalsVisibleTo`

A separate test assembly normally can't access `internal` types or members in the assembly under test. Instead of making implementation details public solely for tests, the product assembly can grant a named **friend assembly** access with `InternalsVisibleTo`.

In an SDK-style .NET project, the MSBuild item is often the simplest option:

```xml
<!-- Add inside the product project's .csproj file. -->
<ItemGroup>
  <InternalsVisibleTo Include="MyBackendApp.Infrastructure.UnitTests" />
</ItemGroup>
```

The value must match the **assembly name** of the test output. If `AssemblyName` is set in the test project, use that value rather than assuming it matches the project or namespace name. The SDK emits the corresponding assembly attribute.

The source-level alternative is an assembly attribute in the product assembly:

```csharp
using System.Runtime.CompilerServices;

[assembly: InternalsVisibleTo("MyBackendApp.Infrastructure.UnitTests")]
```

Both assemblies must be unsigned, or both must be strong-named. When both are unsigned, the friend assembly's simple name is sufficient. When both are strong-named, the friend declaration must include the friend's **full public key** (not its public-key token). In MSBuild, the `InternalsVisibleTo` item supports a `Key` value for this case.

A friend declaration grants access to the assembly's internal surface broadly; it doesn't select one test class or one member. Keep tests focused on public behavior where practical, and use friend access when testing internal behavior is worth the coupling.

## 5. Basic Syntax Example

This class-file example demonstrates the access levels in one assembly. It compiles as part of a class library or alongside an existing application entry point. `DetailedAudit` is derived from `SecurityAuditBase`; `AuditTextFormatter` is a file-local helper. The `RegionalScope` property also shows that an unrelated type in the same assembly can use the `internal` branch of `protected internal`.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public class SecurityAuditBase
{
    private readonly string _privateMarker = "configured";

    public string AuditName { get; } = "Global Audit";
    protected DateTime CreatedAtUtc { get; } = DateTime.UtcNow;
    internal string AssemblyClusterId { get; } = "cluster-east-01";
    protected internal string RegionalScope { get; } = "us-east-1";
    private protected string InternalSubsystemCode { get; } = "SEC-409";

    public bool HasPrivateMarker => _privateMarker.Length > 0;
}

public sealed class DetailedAudit : SecurityAuditBase
{
    public string DescribeAccessibleMembers()
    {
        // All of these are accessible from a derived type in the same assembly.
        string summary = $"{AuditName}; {CreatedAtUtc:O}; {AssemblyClusterId}; " +
            $"{RegionalScope}; {InternalSubsystemCode}";

        // _privateMarker is intentionally inaccessible here: it is private to the base type.
        return summary;
    }
}

file static class AuditTextFormatter
{
    public static string Format(SecurityAuditBase audit)
    {
        ArgumentNullException.ThrowIfNull(audit);
        return $"{audit.AuditName} — {audit.RegionalScope}";
    }
}
```

In this file, `DetailedAudit` can access `protected`, `internal`, `protected internal`, and `private protected` members. It can't access the base class's private field. `AuditTextFormatter` is not derived, but it can access `RegionalScope` because it is in the same assembly. In another assembly, an unrelated formatter could access only the public API; a derived class could access `protected` and the protected branch of `protected internal`, subject to the receiver rule.

## 6. Applied Backend Example: Payment Infrastructure

This example demonstrates an `internal` payment client, a base transaction with a private-protected validation contract and a protected event, and file-local payload helpers. The test assembly is a friend so it can inspect the infrastructure assembly's internals without making the client public.

The code is one .NET 10 source file. Configure the `HttpClient` base address in dependency injection; the sample posts to a relative `payments` endpoint. In a real payment system, use the provider's tokenization and idempotency mechanisms, persist transaction state reliably, and publish domain events at an appropriate transaction boundary.

```csharp
#nullable enable
using System;
using System.Linq;
using System.Net.Http;
using System.Net.Http.Json;
using System.Runtime.CompilerServices;
using System.Threading;
using System.Threading.Tasks;

[assembly: InternalsVisibleTo("MyBackendApp.Infrastructure.UnitTests")]

namespace MyBackendApp.Infrastructure.Payments;

public abstract class PaymentTransaction
{
    private DateTimeOffset? _processedAtUtc;

    public Guid Id { get; } = Guid.NewGuid();

    // Visible to this assembly (and friend assemblies); the backing field remains private.
    internal DateTimeOffset? ProcessedAtUtc => _processedAtUtc;

    // Permitted derived types can subscribe; only this declaring type can raise the event.
    protected event EventHandler? ProcessingRecorded;

    protected void RecordProcessingTime()
    {
        if (_processedAtUtc is not null)
        {
            throw new InvalidOperationException("This transaction has already been processed.");
        }

        _processedAtUtc = DateTimeOffset.UtcNow;
        ProcessingRecorded?.Invoke(this, EventArgs.Empty);
    }

    // Only derived transaction types in this assembly (or a friend assembly) can implement this contract.
    private protected abstract void ValidateGatewayPayload();

    internal void EnsureCanBeProcessed()
    {
        if (ProcessedAtUtc is not null)
        {
            throw new InvalidOperationException("This transaction has already been processed.");
        }

        ValidateGatewayPayload();
    }

    internal void MarkProcessed() => RecordProcessingTime();
}

public sealed class CreditCardPayment : PaymentTransaction
{
    private readonly decimal _amount;
    private readonly string _currency;

    public decimal Amount => _amount;
    public string Currency => _currency;
    public string MaskedCardNumber { get; }

    public CreditCardPayment(decimal amount, string currency, string cardLastFour)
    {
        if (amount <= 0)
        {
            throw new ArgumentOutOfRangeException(
                nameof(amount), amount, "Payment amount must be positive.");
        }

        _amount = amount;
        _currency = PaymentFormattingUtils.NormalizeCurrency(currency);
        MaskedCardNumber = PaymentFormattingUtils.MaskLastFour(cardLastFour);
    }

    private protected override void ValidateGatewayPayload()
    {
        if (_amount <= 0 ||
            MaskedCardNumber.Length != 19 ||
            !MaskedCardNumber.StartsWith("****-****-****-", StringComparison.Ordinal))
        {
            throw new InvalidOperationException("Invalid payment payload for the gateway.");
        }
    }
}

// Internal to the infrastructure assembly; the friend test assembly can access it as well.
internal sealed class PaymentGatewayClient
{
    private readonly HttpClient _httpClient;

    public PaymentGatewayClient(HttpClient httpClient)
    {
        ArgumentNullException.ThrowIfNull(httpClient);
        _httpClient = httpClient;
    }

    public async Task<bool> ExecutePaymentAsync(
        CreditCardPayment payment,
        CancellationToken cancellationToken)
    {
        ArgumentNullException.ThrowIfNull(payment);
        payment.EnsureCanBeProcessed();

        GatewayPaymentPayload payload = PayloadBuilder.Build(payment);
        using HttpResponseMessage response = await _httpClient.PostAsJsonAsync(
            "payments", payload, cancellationToken);

        if (!response.IsSuccessStatusCode)
        {
            return false;
        }

        payment.MarkProcessed();
        return true;
    }
}

file static class PayloadBuilder
{
    public static GatewayPaymentPayload Build(CreditCardPayment payment) => new(
        payment.Id,
        payment.Amount,
        payment.Currency,
        payment.MaskedCardNumber);
}

file record GatewayPaymentPayload(
    Guid TransactionId,
    decimal Total,
    string Currency,
    string MaskedCardNumber);

file static class PaymentFormattingUtils
{
    public static string NormalizeCurrency(string currency)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(currency);
        string normalized = currency.Trim().ToUpperInvariant();

        if (normalized.Length != 3 || !normalized.All(char.IsAsciiLetter))
        {
            throw new ArgumentException("Currency must be a three-letter code.", nameof(currency));
        }

        return normalized;
    }

    public static string MaskLastFour(string cardLastFour)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(cardLastFour);
        string digits = cardLastFour.Trim();

        if (digits.Length != 4 || !digits.All(char.IsAsciiDigit))
        {
            throw new ArgumentException("Supply exactly four ASCII digits.", nameof(cardLastFour));
        }

        return $"****-****-****-{digits}";
    }
}
```

Access-control observations:

- Code in the infrastructure assembly can construct `PaymentGatewayClient`; unrelated assemblies can't. The named test assembly can access it because of `InternalsVisibleTo`.
- `PaymentTransaction.ProcessedAtUtc` has an internal getter over a private backing field. Code in the assembly can read it, but only `PaymentTransaction` can change the field.
- `RecordProcessingTime` and `ProcessingRecorded` are `protected`: derived transaction types in the permitted hierarchy can use or subscribe to them. The base class raises its event. The event illustrates accessibility; production domain events are often recorded and dispatched after the database transaction commits rather than raised synchronously from an entity.
- `ValidateGatewayPayload` is `private protected abstract`: a derived type in another assembly can't implement this contract unless that assembly is a friend. `CreditCardPayment` can implement it because it is in the same assembly.
- The formatting helpers and payload record are `file`-local. They can support types in this source file but aren't available to another file, even one in the same assembly.

## 7. Common Mistakes and Design Guidance

- **Treating `internal` as synonymous with “same project”:** accessibility follows the compiled assembly boundary. Use the assembly name when configuring friend access.
- **Reading `protected internal` as “protected and internal”:** it is a union. `private protected` is the intersection.
- **Forgetting the protected receiver rule:** an external derived class can't use a protected base member through an arbitrary base-typed object.
- **Assuming `public` always means globally reachable:** containing types and signature types also have accessibility constraints.
- **Using `public` fields for mutable state:** properties and methods offer a controlled API and allow invariants to be preserved.
- **Using `protected` by default for reuse:** protected members become part of the inheritance contract. Prefer private/internal helpers unless derived classes genuinely need an extension point.
- **Treating `file` as an access modifier for members:** it applies to top-level types only and can't be combined with another accessibility modifier.
- **Treating access modifiers as a security boundary:** they're language/API controls, not a substitute for secret management or authorization.
- **Making types public just for tests:** consider `InternalsVisibleTo`, but remember it exposes the assembly's internal surface to that entire friend assembly.
- **Exposing less-accessible types in public signatures:** a public member can't use a less-accessible type in its public signature; the compiler reports an inconsistent-accessibility error.

## 8. Key Terms Summary

| Term | Definition | Typical backend use |
|---|---|---|
| **Accessibility** | The parts of a program where a type or member can be referenced. | Shapes APIs and assembly boundaries. |
| **`public`** | No additional access restriction beyond the containing type. | Contracts intended for callers. |
| **`private`** | Accessible within the declaring type and its nested types. | Implementation details and owned state. |
| **`protected`** | Accessible within the declaring type and from derived types. | Intentional inheritance hooks. |
| **`internal`** | Accessible within the current assembly. | Collaboration among components in one assembly. |
| **`protected internal`** | Accessible in the current assembly **or** from derived types. | Same-assembly access plus external extensibility. |
| **`private protected`** | Accessible from derived types in the current assembly. | Restricted inheritance within one assembly. |
| **`file`-local type** | A top-level type nameable only in its declaring source file. | Source-generator and file-specific helpers. |
| **Assembly** | A compiled .NET unit, commonly a `.dll` or `.exe`. | The boundary used by `internal`. |
| **Friend assembly** | An assembly granted access to another assembly's internal members. | Unit tests or tightly coupled companion assemblies. |
| **`InternalsVisibleTo`** | Assembly attribute/MSBuild item that names friend assemblies. | Grants internal access without making members public. |
| **Effective accessibility** | Actual reachability after the declared modifier and containing types are both considered. | Explains why a public member on an internal type isn't externally public. |

## 9. Official References

- [Access modifiers — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/access-modifiers) — overview of access modifiers and the `file` modifier.
- [Accessibility levels — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/accessibility-levels) — permitted modifiers and default accessibility by context.
- [Accessibility domain — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/accessibility-domain) — how the containing type constrains a member's access domain.
- [`protected` — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/protected) — derived-type access and receiver restrictions.
- [`protected internal` — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/protected-internal) — union semantics and cross-assembly behavior.
- [`private protected` — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/private-protected) — intersection semantics and friend-assembly notes.
- [`file` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/file) — file-local type scope and restrictions.
- [Restrictions on using accessibility levels — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/restrictions-on-using-accessibility-levels) — accessibility consistency in signatures and inheritance.
- [`InternalsVisibleToAttribute` API](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.internalsvisibletoattribute?view=net-10.0) — friend assemblies and strong-name requirements.
- [Common MSBuild project items](https://learn.microsoft.com/en-us/visualstudio/msbuild/common-msbuild-project-items?view=visualstudio#internalsvisibleto) — SDK-style `InternalsVisibleTo` item and optional `Key` metadata.
