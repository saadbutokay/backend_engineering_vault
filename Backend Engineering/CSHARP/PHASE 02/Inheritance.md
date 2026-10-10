Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 What Is Inheritance?

**Inheritance** lets a class derive from another class and specialize its contract. The class being extended is the **base class** (or superclass); the new class is the **derived class** (or subclass). A derived class can add members and override inherited virtual behavior.

Inheritance is intended to express an **is-a** relationship: an `EmailNotification` is a `Notification`. That relationship should be meaningful to callers: code that accepts a `Notification` should still work correctly when given an `EmailNotification`.

```text
       +-------------------------------------+
       | Notification (abstract base class)  |
       | Id, CreatedAtUtc, Dispatch(...)      |
       +-------------------------------------+
                         ^
                         | derives from
       +-------------------------------------+
       | EmailNotification                  |
       | SmtpServer, specialized Dispatch    |
       +-------------------------------------+
```

Use inheritance when the derived type truly is a specialized form of the base type and can honor its contract. If two types only share implementation, or one merely *has* another capability, composition or an interface is often a better fit.

### 1.2 Core Rules of Class Inheritance in C#

1. **One direct base class:** A class can derive from only one class. It can implement multiple interfaces. Structs don't derive from classes, but they can implement interfaces.
2. **Implicit root:** A class with no explicit class base ultimately derives from `System.Object` (also written as the C# alias `object`). The instance methods `ToString()`, `Equals(object?)`, and `GetHashCode()` are virtual; `GetType()` is inherited but isn't virtual.
3. **Transitive inheritance:** If `C` derives from `B`, and `B` derives from `A`, instances of `C` also have the accessible members declared by `A` and `B`.
4. **Constructors and finalizers aren't inherited:** Each derived class declares its own constructors, even though construction always initializes the base part of the object.
5. **Accessibility still applies:** Inheritance doesn't make every base member accessible to derived code.

| Base member modifier | Access in a derived class |
|---|---|
| `public` | Accessible wherever the public member is accessible. |
| `protected` | Accessible in the declaring class and derived classes, subject to C#'s protected-access rules. |
| `internal` | Accessible from code in the same assembly, whether or not that code derives from the type. It is not a namespace-level permission. |
| `private` | Not directly accessible by name from a derived class. The base class retains and manages its own private state. |

A derived object includes the state of its base-class portion, but a derived class can't directly read or write a private base field. Avoid treating a particular object memory layout as a C# language guarantee. C# also provides `protected internal` (accessible from the same assembly or a derived class) and `private protected` (accessible from the declaring class or a derived class in the same assembly) for combined scopes.

## 2. Base-Class Initialization and the `base` Keyword

Every derived-class constructor must initialize its base-class portion before its own body runs. A constructor can call another constructor in the same class with `this(...)`, or choose a base constructor with `base(...)`—not both in the same initializer. If neither is written, the compiler supplies an implicit `base()` call. That works only if the base class has an accessible parameterless constructor; otherwise, specify an accessible base constructor explicitly.

The `base` keyword refers to the **immediate** base class. In a constructor initializer it selects a base constructor; in an instance method or property accessor it can call the base implementation, including the base implementation of an overridden member.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public abstract class Entity
{
    public Guid Id { get; }

    protected Entity(Guid id)
    {
        if (id == Guid.Empty)
        {
            throw new ArgumentException("Entity ID cannot be empty.", nameof(id));
        }

        Id = id;
    }
}

public sealed class Product : Entity
{
    public string Name { get; }

    // Entity has no parameterless constructor, so Product selects base(Guid).
    public Product(Guid id, string name) : base(id)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(name);
        Name = name.Trim();
    }
}
```

Here, `Product` must supply `base(id)`: without it, the compiler would try to call `Entity()` and report an error because no accessible parameterless constructor exists.

## 3. Overriding, Polymorphism, and Member Hiding

### 3.1 Overriding with `virtual`, `abstract`, and `override`

A base class marks an instance method, property, indexer, or event as `virtual` to allow a derived implementation. An `abstract` member has no base implementation and must be implemented by a concrete derived class. A derived member uses `override` to extend or replace an inherited `virtual`, `abstract`, or `override` member; it can't override a nonvirtual member.

For a virtual call, C# dispatches to the most-derived override available on the **runtime object**, even when the variable is declared as the base type. This is the basis of runtime polymorphism. A `sealed override` can stop further derived classes from overriding that member; a `sealed` class can't be derived from at all.

### 3.2 Hiding a Member with `new`

A derived declaration with the same name can **hide** a base member rather than override it. Use the `new` modifier to make intentional hiding explicit; otherwise, the compiler normally warns that the inherited member is hidden. Hiding creates a distinct member; it doesn't override or extend the base member's virtual slot. For a nonvirtual call, member lookup uses the expression's compile-time type.

```csharp
#nullable enable
using System;

public class BaseService
{
    public virtual void Process() => Console.WriteLine("Base Process");
    public void Notify() => Console.WriteLine("Base Notify");
}

public sealed class ExtendedService : BaseService
{
    public override void Process() => Console.WriteLine("Derived Process");
    public new void Notify() => Console.WriteLine("Derived Notify");
}

public static class Demo
{
    public static void Main()
    {
        var derived = new ExtendedService();
        BaseService baseReference = derived;

        baseReference.Process(); // Derived Process: virtual dispatch uses runtime type.
        baseReference.Notify();  // Base Notify: this member isn't virtual.
        derived.Notify();        // Derived Notify: lookup uses ExtendedService.
    }
}
```

Prefer `override` when a derived type is implementing the base contract polymorphically. Use `new` only when a distinct hidden member is intentional; it can surprise callers that hold the object through a base-type reference. The runtime's implementation mechanism (often described using virtual method tables) isn't the important contract—the distinction between overriding and hiding is.

## 4. Basic Syntax Example: Notifications

This source file demonstrates a protected base constructor, inherited state, a `protected` member, constructor forwarding, and an override that calls `base.Dispatch`. `TimeProvider` makes the creation timestamp controllable in tests. The `Console.WriteLine` calls illustrate dispatch; they aren't an SMTP implementation.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public abstract class Notification
{
    public Guid Id { get; }
    public DateTimeOffset CreatedAtUtc { get; }

    protected string ChannelType { get; }

    protected Notification(string channelType, TimeProvider? timeProvider = null)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(channelType);

        Id = Guid.NewGuid();
        ChannelType = channelType.Trim();
        CreatedAtUtc = (timeProvider ?? TimeProvider.System).GetUtcNow();
    }

    public virtual void Dispatch(string recipient, string message)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(recipient);
        ArgumentNullException.ThrowIfNull(message);

        Console.WriteLine($"[{ChannelType}] Sending payload to {recipient}: {message}");
    }
}

public sealed class EmailNotification : Notification
{
    public string SmtpServer { get; }

    public EmailNotification(string smtpServer, TimeProvider? timeProvider = null)
        : base("SMTP_EMAIL", timeProvider)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(smtpServer);
        SmtpServer = smtpServer.Trim();
    }

    public override void Dispatch(string recipient, string message)
    {
        base.Dispatch(recipient, message);
        Console.WriteLine($"Routed via SMTP server: {SmtpServer}");
    }
}
```

`EmailNotification` inherits `Id` and `CreatedAtUtc`; it can't access private state in `Notification`, but it can read the protected `ChannelType`. Its `Dispatch` override can call the base implementation with `base.Dispatch(...)` before adding its specialized behavior.

## 5. Applied Backend Example: Entity and Audit Base Types

A backend may use a small **layer supertype** for genuinely shared entity behavior, such as identity or domain-event handling, and a more specific `AuditableEntity` for entities that all need audit metadata. This is an optional design pattern—not a requirement of Clean Architecture or Entity Framework Core. Keep base classes cohesive; audit stamping may instead belong in persistence infrastructure, and a capability that doesn't define an “is-a” relationship may fit composition better.

The following domain-model sketch shows a base entity, an auditable intermediate class, and a concrete account. It uses private setters for persistence-friendly mapping and a private parameterless `UserAccount` constructor as one possible EF Core materialization path. The snippet is **not** a complete EF Core model configuration.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;

namespace MyBackendApp.Core.Domain.Common;

public interface IDomainEvent
{
}

public abstract class Entity
{
    private readonly List<IDomainEvent> _domainEvents = [];
    private readonly ReadOnlyCollection<IDomainEvent> _domainEventsView;

    public Guid Id { get; private set; }
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEventsView;

    protected Entity() : this(Guid.NewGuid())
    {
    }

    protected Entity(Guid id)
    {
        if (id == Guid.Empty)
        {
            throw new ArgumentException("Entity ID cannot be empty.", nameof(id));
        }

        Id = id;
        _domainEventsView = _domainEvents.AsReadOnly();
    }

    protected void AddDomainEvent(IDomainEvent domainEvent)
    {
        ArgumentNullException.ThrowIfNull(domainEvent);
        _domainEvents.Add(domainEvent);
    }

    public void ClearDomainEvents() => _domainEvents.Clear();
}

public abstract class AuditableEntity : Entity
{
    public DateTimeOffset CreatedAtUtc { get; private set; }
    public string CreatedBy { get; private set; } = string.Empty;
    public DateTimeOffset? LastModifiedAtUtc { get; private set; }
    public string? LastModifiedBy { get; private set; }

    // A parameterless path can be used when an ORM materializes a stored entity.
    protected AuditableEntity() : base()
    {
        CreatedAtUtc = TimeProvider.System.GetUtcNow();
    }

    protected AuditableEntity(string createdBy, TimeProvider? timeProvider = null) : base()
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(createdBy);

        CreatedBy = createdBy.Trim();
        CreatedAtUtc = (timeProvider ?? TimeProvider.System).GetUtcNow();
    }

    protected void RecordModification(string modifiedBy, TimeProvider? timeProvider = null)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(modifiedBy);

        string normalizedModifiedBy = modifiedBy.Trim();
        DateTimeOffset modifiedAtUtc = (timeProvider ?? TimeProvider.System).GetUtcNow();

        LastModifiedBy = normalizedModifiedBy;
        LastModifiedAtUtc = modifiedAtUtc;
    }
}

public sealed record UserTierUpgradedEvent(Guid UserId, string NewTier) : IDomainEvent;

public sealed class UserAccount : AuditableEntity
{
    public string Username { get; private set; } = string.Empty;
    public string MembershipTier { get; private set; } = string.Empty;

    // EF Core can use this constructor for materialization; callers use the validated constructor.
    private UserAccount() : base()
    {
    }

    public UserAccount(
        string username,
        string initialTier,
        string createdBy,
        TimeProvider? timeProvider = null) : base(createdBy, timeProvider)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(username);
        ArgumentException.ThrowIfNullOrWhiteSpace(initialTier);

        Username = username.Trim();
        MembershipTier = initialTier.Trim();
    }

    public void UpgradeTier(
        string newTier,
        string updatedBy,
        TimeProvider? timeProvider = null)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(newTier);
        ArgumentException.ThrowIfNullOrWhiteSpace(updatedBy);

        string normalizedTier = newTier.Trim();
        if (MembershipTier.Equals(normalizedTier, StringComparison.OrdinalIgnoreCase))
        {
            return;
        }

        var domainEvent = new UserTierUpgradedEvent(Id, normalizedTier);
        RecordModification(updatedBy, timeProvider);
        MembershipTier = normalizedTier;
        AddDomainEvent(domainEvent);
    }
}
```

`DomainEvents` is a cached read-only **live view** of the private list: the entity can add and clear events, while callers can't edit the list through the exposed collection. Event records are used here to keep the sample event data simple; choose an event abstraction that fits the application.

For EF Core, map the hierarchy and persistence properties deliberately. EF Core uses **table-per-hierarchy (TPH)** by default; **table-per-type (TPT)** and **table-per-concrete-type (TPC)** are alternatives. Include the intended derived types in the model and explicitly ignore `DomainEvents` (or persist events separately), because that in-memory collection is domain infrastructure rather than an entity relationship. EF Core supports private setters and nonpublic constructors, but constructor binding follows its own conventions; this example includes a private parameterless constructor instead of relying on EF Core to bind the domain-creation constructor.

## 6. Design Guidance and Common Mistakes

- **Inheriting only to reuse code:** shared implementation alone doesn't make one type a subtype of another. Consider a helper, a composed service, or an interface.
- **Breaking the base contract:** a derived type should remain usable anywhere the base type is expected. Don't weaken guarantees callers rely on.
- **Confusing `new` with `override`:** `new` hides a member; `override` participates in virtual dispatch.
- **Assuming constructors are inherited:** a derived type declares its own constructors and explicitly selects a base constructor when a parameterless one isn't available.
- **Treating `protected` as private:** protected members are available to derived types and therefore become part of the inheritance contract. Keep that surface intentional.
- **Making one oversized base class:** combining persistence, web/API, audit, identity, and domain-event concerns couples otherwise unrelated types. Prefer small cohesive base types or composition.
- **Assuming a base class is required for EF Core:** EF Core can map inheritance, but the CLR hierarchy and database mapping are separate design decisions. TPH, TPT, and TPC have different schema and query trade-offs.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Base class** | The class whose members form the inherited part of a derived object. | Defines common state or behavior for a genuine type family. |
| **Derived class** | A class that extends one direct base class. | Specializes the base contract and can add or override behavior. |
| **`base` keyword** | A reference to the immediate base class in a derived constructor or instance member. | Selects a base constructor or calls a base implementation. |
| **`virtual` member** | An overridable instance member declared by a base class. | Defines an extension point for polymorphic behavior. |
| **`abstract` member/class** | A member without an implementation, or a class that can't be instantiated directly. | States behavior a concrete subtype must provide. |
| **`override`** | A derived implementation of an inherited virtual or abstract member. | Provides polymorphism while keeping the base contract. |
| **Member hiding (`new`)** | A distinct derived member that hides a base member with the same name. | Makes deliberate hiding explicit; it doesn't override the base member. |
| **Single inheritance** | A C# class can derive from one class, while implementing multiple interfaces. | Encourages focused class hierarchies and separate capability contracts. |
| **Layer supertype** | A base class that centralizes behavior shared by a cohesive set of types in one application layer. | Can reduce duplication, but should remain small and optional. |
| **TPH / TPT / TPC** | EF Core relational strategies for mapping inheritance to tables. | Connects a CLR hierarchy to a database schema; each strategy has trade-offs. |

## 8. Official References

- [Inheritance — C# fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance) — base/derived types, abstract classes, and inheritance rules.
- [`base` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base) — base constructor calls and base-member access.
- [`virtual` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual) — runtime dispatch for virtual members.
- [`override` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override) — overriding virtual and abstract members.
- [`new` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/new-modifier) — explicit member hiding.
- [Access modifiers — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/access-modifiers) — `public`, `protected`, `internal`, and `private` accessibility.
- [`System.Object` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.object?view=net-10.0) — root class and its members.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — default language versions, including .NET 10 / C# 14.
- [EF Core 10 inheritance mapping](https://learn.microsoft.com/en-us/ef/core/modeling/inheritance?view=efcore-10.0) — TPH, TPT, and TPC mapping strategies.
- [EF Core entity constructors](https://learn.microsoft.com/en-us/ef/core/modeling/constructors?view=efcore-10.0) — constructor binding and read-only properties.
- [`TimeProvider` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.timeprovider?view=net-10.0) — controllable time abstraction used by the examples.
- [`ArgumentException.ThrowIfNullOrWhiteSpace` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception.throwifnullorwhitespace?view=net-10.0) — input guard used by the examples.
