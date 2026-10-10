Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 What Does `static` Mean in C#?

A `static` member belongs to a **type**, rather than to one particular object. You normally refer to it through the type name, such as `PaymentGateway.DefaultTimeoutSeconds`.

- **Instance fields** are part of each object. Two objects can hold different values for the same instance field.
- **Static fields** are shared by code using the same loaded runtime type. For a generic type, each closed type such as `Cache<int>` and `Cache<string>` has its own static fields. Special cases such as thread-static fields have different sharing behavior.
- Static state is associated with a runtime type; its physical memory layout is an implementation detail. Avoid describing it as a field stored in a particular “high-frequency heap” or assuming one copy for every possible load context.

```text
       Instance fields                           Static field (conceptual)
+-----------------------------+            +-------------------------------+
| BankAccount object A        |            | BankAccount.GlobalFee         |
|   _balance = 100.00         |            |            = 1.50              |
+-----------------------------+            | shared by users of this       |
                                            | loaded BankAccount type        |
+-----------------------------+            +-------------------------------+
| BankAccount object B        |
|   _balance = 500.00         |
+-----------------------------+
```

This is a conceptual ownership diagram, not a map of CLR memory. Static state can be shared by many threads, so its mutability and synchronization deserve special care.

## 2. Core Rules and Components of `static`

### 2.1 Static Classes

A `static class` is useful for stateless operations and extension methods. It:

1. **Cannot be instantiated.** `new MyStaticClass()` is a compile-time error.
2. **Cannot be inherited from** and cannot derive from another class. It has the usual implicit `System.Object` base, but cannot specify another base type.
3. **Cannot implement interfaces.** C# interfaces can declare `static abstract` members for generic algorithms, but concrete implementing types—not static classes—provide those members.
4. **Cannot declare instance fields, properties, methods, events, or instance constructors.** Its class members must be static; nested type declarations are also allowed.
5. **Can have a static constructor** when one-time initialization logic is needed.

### 2.2 Static Members in Non-Static Classes

A regular class can mix static and instance fields, properties, methods, and events.

- Call a static member through its type: `PaymentGateway.DefaultTimeoutSeconds`.
- You can't access a static member through an instance, such as `gateway.DefaultTimeoutSeconds`.
- A static method doesn't have a `this` object and can't directly access instance members. Pass an object as a parameter when the operation needs instance data.
- Static members on ordinary classes aren't virtual instance extension points. Static abstract interface members are a separate compile-time polymorphism feature.

### 2.3 Static Constructors

A static constructor initializes static state that requires runtime work. Its key rules are:

- It has the form `static TypeName()`. It takes no parameters, has no access modifier, can't be called directly, and can't be overloaded or inherited.
- When type initialization is triggered, an explicit static constructor runs **at most once** for that runtime type. Each closed construction of a generic type has its own static state and initialization.
- The runtime serializes static-constructor execution for a type; you don't add a `lock` just to make the constructor run once. Avoid blocking, waiting on tasks, or taking locks inside it because initialization locks can contribute to deadlocks.
- If it throws, the constructor isn't retried. Access commonly fails with a `TypeInitializationException`, and that type remains uninitialized for the relevant runtime lifetime.
- An explicit static constructor limits the runtime's `beforefieldinit` optimization freedom. If ordinary static field initializers are enough, prefer them; add an explicit constructor when you need its initialization behavior.

Static field initializers run before that type's explicit static constructor. If there is no explicit static constructor, the runtime has more freedom about when to run field initialization; don't depend on an exact first-use moment for side effects.

## 3. Concurrency and Thread Safety

Static state is shared by code using the same loaded type, including concurrent ASP.NET Core requests. `static` does **not** make mutable state thread-safe.

- Prefer constants, immutable values, or immutable collections for shared lookup data.
- `static readonly` prevents reassignment of the field after initialization; it does **not** make the object referenced by that field immutable. A `static readonly List<T>` can still be modified.
- Compound operations on mutable data need synchronization or concurrency-aware types. For counters, `Interlocked` operations provide atomic updates; for a larger invariant, use an appropriate lock or concurrent design.
- A plain static cache can live far longer than an individual request and may grow without bound. Consider dependency-injected services, bounded caches, and explicit lifetime policies instead of process-wide mutable state.

## 4. Basic Syntax Example

This example combines a static utility class, a static constructor, a read-only static property, and an atomic count of objects created. The count is cumulative; it is **not** an active-instance or active-request count.

```csharp
#nullable enable
using System;
using System.Security.Cryptography;
using System.Text;
using System.Threading;

namespace MyBackendApp.Basics;

// Stateless helper methods belong to the type, not to utility instances.
public static class HashUtility
{
    public const string DefaultAlgorithm = "SHA-256";

    public static string ComputeSha256(string input)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(input);

        byte[] bytes = SHA256.HashData(Encoding.UTF8.GetBytes(input));
        return Convert.ToHexStringLower(bytes);
    }
}

public sealed class ApiClient
{
    private static int _createdInstanceCount;

    public static string GlobalEnvironment { get; }
    public static int CreatedInstanceCount => Volatile.Read(ref _createdInstanceCount);

    public Guid ClientSessionId { get; }

    // Runs once when this type is initialized, not once per ApiClient object.
    static ApiClient()
    {
        GlobalEnvironment =
            Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Development";
    }

    public ApiClient()
    {
        ClientSessionId = Guid.NewGuid();
        Interlocked.Increment(ref _createdInstanceCount);
    }
}
```

`GlobalEnvironment` is a snapshot read during type initialization. In an ASP.NET Core application, use the host environment and configuration through dependency injection when you need application configuration that is easier to test and manage. This sample's SHA-256 helper is for ordinary hashing only; don't use fast general-purpose hashes to store passwords.

## 5. Backend Example: Result Factories and an Immutable Lookup

Static factory methods can make object creation explicit without requiring callers to invoke a public constructor. A source-generated regular expression is reusable, and a `FrozenSet<T>` suits data built once and queried repeatedly. Neither choice makes the whole validation policy complete or universally optimal; profile real workloads and apply the appropriate domain rules.

```csharp
#nullable enable
using System;
using System.Collections.Frozen;
using System.Text.RegularExpressions;

namespace MyBackendApp.Core.Common;

public sealed class Result<T>
{
    public bool IsSuccess { get; }
    public bool IsFailure => !IsSuccess;
    public T? Value { get; }
    public string? Error { get; }

    private Result(bool isSuccess, T? value, string? error)
    {
        IsSuccess = isSuccess;
        Value = value;
        Error = error;
    }

    public static Result<T> Success(T value)
    {
        ArgumentNullException.ThrowIfNull(value);
        return new Result<T>(true, value, null);
    }

    public static Result<T> Failure(string errorMessage)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(errorMessage);
        return new Result<T>(false, default, errorMessage);
    }
}

public static partial class SecurityGuards
{
    // The source generator provides and caches the Regex implementation.
    [GeneratedRegex(
        @"^[A-Za-z0-9_.+-]+@[A-Za-z0-9-]+(?:\.[A-Za-z0-9-]+)+$",
        RegexOptions.IgnoreCase | RegexOptions.CultureInvariant)]
    private static partial Regex EmailRegex();

    private static readonly FrozenSet<string> DisallowedDomains;

    static SecurityGuards()
    {
        DisallowedDomains = new[]
        {
            "tempmail.com",
            "disposable.com",
            "trashmail.net"
        }.ToFrozenSet(StringComparer.OrdinalIgnoreCase);
    }

    public static Result<string> ValidateAndNormalizeEmail(string? rawEmail)
    {
        if (string.IsNullOrWhiteSpace(rawEmail))
        {
            return Result<string>.Failure("Email cannot be null or empty.");
        }

        string email = rawEmail.Trim();
        if (email.Length > 254)
        {
            return Result<string>.Failure("Email exceeds the sample's 254-character limit.");
        }

        if (!EmailRegex().IsMatch(email))
        {
            return Result<string>.Failure("Email syntax is invalid for this sample.");
        }

        // The pattern guarantees an @ separator. Normalize only the domain;
        // email local-part case handling is a separate application policy.
        int separator = email.LastIndexOf('@');
        string domain = email[(separator + 1)..];

        if (DisallowedDomains.Contains(domain))
        {
            return Result<string>.Failure($"Email domain '{domain}' is not permitted.");
        }

        string normalizedEmail = email[..(separator + 1)] + domain.ToLowerInvariant();
        return Result<string>.Success(normalizedEmail);
    }
}
```

The regular expression is only a lightweight syntax filter: it is not a complete implementation of every valid email-address form and doesn't prove that an address exists or can receive mail. The denylist checks exact domain names, not their subdomains. For registration or identity workflows, use a suitable product policy and verify control of the address where appropriate.

`FrozenSet<T>` is immutable after construction and optimized for repeated lookups, at the cost of building the set. For tiny or rarely used data, a simpler representation may be preferable. `Result<T>.Success` and `Result<T>.Failure` are ordinary factory methods; they still create result objects and are not “zero-overhead.”

## 6. Design Guidance and Common Mistakes

- **Treating static state as per-request state:** all callers of the same loaded type share it. Keep request-specific data on request-scoped objects.
- **Claiming there is exactly one static copy everywhere:** static state is per loaded runtime type, and each closed generic type has its own copy. Runtime/load-context details matter; avoid hard-coding an AppDomain or heap-layout claim.
- **Assuming `static readonly` means immutable:** it freezes the field reference, not the referenced object's contents. Use an immutable collection or keep a mutable collection private and never mutate it after publication.
- **Assuming a static constructor runs exactly once even if unused:** it runs at most once when initialization is triggered. An explicit constructor also restricts `beforefieldinit` optimizations.
- **Doing blocking work in a static constructor:** avoid waits, task blocking, and complicated cross-type initialization that can deadlock or turn a transient startup problem into a permanent type-initialization failure.
- **Using a static class as an implementation of a generic-math interface:** static classes can't implement interfaces. A non-static class or struct can implement an interface that declares static abstract members.
- **Calling static methods through an object:** access them through the type name. Static methods don't receive an implicit instance.
- **Assuming static means faster:** static members avoid per-instance state, but performance depends on the operation and runtime. A static cache may add memory retention, contention, or complexity.
- **Using `const` for a public value that must change independently of consumers:** constants are substituted into consuming assemblies at compile time. Use `static readonly` or configuration when the value must be read at runtime.
- **Using a plain static dictionary as an unbounded cache:** select a concurrency strategy, size limit, expiration policy, and ownership/lifetime deliberately.
- **Treating a regex as complete email validation:** regex can provide a limited syntax check, not proof of deliverability or full standards compliance.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Static member** | A member belonging to a type rather than to one object. | Provides type-level behavior or data shared by users of the same loaded type. |
| **Static class** | A non-instantiable, non-inheritable class whose class members are static. | Organizes stateless helpers and extension methods. |
| **Static constructor** | A parameterless type initializer invoked automatically when type initialization is triggered. | Initializes state that requires one-time runtime work. |
| **Static factory method** | A static method that creates and returns an instance, often with a descriptive name. | Makes construction intent explicit, as in `Result<T>.Success(value)`. |
| **`static readonly`** | A field assignable at declaration or in the type's static constructor. | Prevents reassigning the field after initialization; it doesn't freeze a referenced object. |
| **`Interlocked`** | APIs for atomic operations on shared variables. | Supports simple concurrent updates such as an instance-created counter. |
| **`FrozenSet<T>`** | An immutable set optimized for lookup after it is built. | Fits trusted lookup data initialized infrequently and queried often. |
| **`beforefieldinit`** | A CLR type attribute that permits more flexibility in the timing of static field initialization. | Explains why an explicit static constructor can constrain runtime optimization. |

## 8. Official References

- [`static` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static) — static members, classes, and type-based access.
- [Static classes and static class members — C# guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members) — class restrictions and static-member behavior, including generic types.
- [Static constructors — C# guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors) — execution, thread safety, exceptions, and `beforefieldinit`.
- [`interface` — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface#static-abstract-and-virtual-members) — static abstract and virtual interface members.
- [.NET regular expression source generators](https://learn.microsoft.com/en-us/dotnet/standard/base-types/regular-expression-source-generators) — generated regex usage and why `RegexOptions.Compiled` isn't needed with source generation.
- [`FrozenSet<T>` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.collections.frozen.frozenset-1?view=net-10.0) — immutable set optimized for repeated lookups.
- [`Interlocked.Increment` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.threading.interlocked.increment?view=net-10.0) — atomic counter increments.
- [`Volatile.Read` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.threading.volatile.read?view=net-10.0) — synchronized reads for the counter example.
- [`Convert.ToHexStringLower` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.convert.tohexstringlower?view=net-10.0) — lowercase SHA-256 hexadecimal output.
- [`SHA256.HashData` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.sha256.hashdata?view=net-10.0) — one-shot hash API used in the syntax example.
- [`ArgumentException.ThrowIfNullOrWhiteSpace` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception.throwifnullorwhitespace?view=net-10.0) — input guard used by the samples.
