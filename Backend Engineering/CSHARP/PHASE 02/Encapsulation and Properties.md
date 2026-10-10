Examples target **.NET 10** with nullable reference analysis enabled. The .NET 10 SDK normally defaults to C# 14 unless the project overrides `LangVersion`. `init` was introduced in C# 9; `required` was introduced in C# 11. Code blocks are self-contained unless labeled as a fragment.

## 1. Conceptual Foundation

### 1.1 What Is Encapsulation?

**Encapsulation** groups state with the operations that govern it and limits how callers can interact with that state. A type exposes a deliberate public contract—methods and properties—while keeping implementation details private or otherwise restricted.

In backend domain models, encapsulation helps enforce **invariants**: rules that should remain true whenever an object is in a valid state. For example, a wallet's balance shouldn't become negative, and an order shouldn't be marked shipped before it has been paid. A public setter that allows callers to bypass those rules weakens the model.

```text
+------------------------------------------------------+
|                    Domain object                    |
|                                                      |
|  Public contract:                                    |
|   Deposit(amount)     Submit()      TotalAmount       |
|          |                |              ^            |
|          v                v              |            |
|  Validation and business rules control state changes |
|          |                                            |
|          v                                            |
|  Private state: _balance, _status, _items             |
+------------------------------------------------------+
```

Encapsulation doesn't mean “make every property private.” It means callers use operations that preserve the type's contract. A simple data-transfer object (DTO) may appropriately have public `init` or `required` properties, while a domain entity with important business rules often needs narrower mutation paths.

### 1.2 Anemic and Rich Domain Models

- An **anemic domain model** is a domain model that mostly holds data while business rules and state transitions live in external services. In a domain-driven design (DDD) system with meaningful invariants, this can scatter rules and make valid state harder to protect.
- A **rich domain model** places behavior and invariant checks with the domain state they govern. Methods such as `Withdraw`, `Submit`, or `ChangeAddress` offer named transitions rather than unrestricted setters.

These are design choices, not rules for every class. DTOs, persistence projections, and simple CRUD records can be intentionally data-oriented. Put behavior where it best protects the application contract; don't add ceremony to types that have no meaningful rules.

## 2. Property Mechanisms

A C# property is an accessor-based member. It can store a value through an auto-property backing field, use an explicit private field, or compute a result when read. An accessor can also validate, normalize, or restrict changes.

### 2.1 The `init` Accessor

An `init` accessor allows assignment in the construction contexts permitted by C#, including object initializers, `with` expressions for types that support them, and constructor bodies of the declaring type or derived types. Ordinary assignment after initialization is rejected by the compiler. Object-initializer assignments run after the constructor, so an initializer can override a value the constructor assigned to an `init` property; use a getter-only property if callers must not do that. `init` is useful for values set at creation time, but it doesn't by itself validate a value or make an object graph deeply immutable.

This complete program shows an object initializer and a later assignment that would fail if uncommented:

```csharp
#nullable enable
using System;

var dto = new UserDto
{
    UserId = Guid.NewGuid(),
    Username = "jdoe"
};

Console.WriteLine($"{dto.Username} ({dto.UserId})");
// dto.Username = "newname"; // Compile-time error: init-only after construction.

public sealed class UserDto
{
    public Guid UserId { get; init; }
    public string Username { get; init; } = string.Empty;
}
```

`init` doesn't require the caller to assign a value: an omitted `UserId` still defaults to `Guid.Empty`, and `Username` uses its initializer. Use validation in a constructor or init accessor when the actual value must meet a business rule.

### 2.2 The `required` Modifier

`required` tells the compiler that code creating an instance must initialize a field or property. It was introduced in **C# 11**; .NET 7 normally defaults to C# 11, but this is a language/compiler feature rather than a special .NET 7 runtime validation feature.

```csharp
#nullable enable
using System;

var command = new CreateCustomerCommand
{
    Email = "jane@example.com",
    FullName = "Jane Doe"
};

Console.WriteLine($"Creating {command.FullName} ({command.Email})");

// Compile-time error if uncommented: Email is required.
// var incomplete = new CreateCustomerCommand { FullName = "Jane Doe" };

public sealed class CreateCustomerCommand
{
    public required string Email { get; init; }
    public required string FullName { get; init; }
}
```

`required` is not the same as non-nullable and isn't runtime validation. A caller can still assign `null!`, an empty string, or a semantically invalid value; deserialization and other runtime paths also need suitable validation. A constructor can be marked with `[SetsRequiredMembers]` to tell the compiler it initializes all required members, but that attribute is an assertion the compiler trusts.

### 2.3 Asymmetric Accessor Accessibility

A property can have an accessor that is more restricted than the property itself. At most one accessor can declare its own accessibility modifier, and that modifier must be more restrictive than the property's accessibility.

The following are **member fragments** to place inside a class:

```csharp
public decimal CurrentBalance { get; private set; } // Public read; class-only write.
public string InternalToken { get; internal init; } // Public read; same-assembly initialization.
```

The first form is common for state changed by domain methods. The second permits a public getter while restricting the `init` accessor to the declaring assembly. Choose accessor visibility to match the intended contract; a public `init` setter can still be used by callers in an object initializer.

## 3. Collection Encapsulation and Defensive Copies

A get-only property doesn't make a collection read-only. If the property returns a `List<T>`, callers can still call `Add`, `Remove`, or `Clear`:

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;

public sealed class LeakyBasket
{
    public List<string> Items { get; } = [];
}

public sealed class EncapsulatedBasket
{
    private readonly List<string> _items = [];
    private readonly ReadOnlyCollection<string> _itemsView;

    public EncapsulatedBasket()
    {
        _itemsView = _items.AsReadOnly();
    }

    // The wrapper blocks collection edits through the public view.
    public IReadOnlyList<string> Items => _itemsView;

    public void Add(string item)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(item);
        _items.Add(item.Trim());
    }

    // A shallow snapshot: callers can change this array without changing _items.
    public string[] GetSnapshot() => _items.ToArray();
}
```

`List<T>.AsReadOnly()` returns a **read-only wrapper**, not a copy. The wrapper reflects later changes made through `EncapsulatedBasket.Add`, but callers can't modify the underlying list through that wrapper. Caching the wrapper, as above, also avoids creating a new wrapper on every property access.

An `IReadOnlyList<T>` interface alone is only a compile-time view: if the property returns the original `List<T>` instance, a caller might cast it back to `List<T>` and mutate it. Expose a wrapper or an immutable collection when callers must not mutate the collection. A snapshot such as `ToArray()` separates the list structure, but it is **shallow**: if `T` is mutable, callers can still mutate objects referenced by the copied array. A read-only view or defensive copy doesn't automatically make the contained objects immutable.

## 4. Basic Syntax Example: Employee Salary Record

This class establishes a valid identity and base salary in its constructor. Its salary and bonus properties use private setters so callers change those values through named methods that enforce the sample's rules.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public sealed class EmployeeSalaryRecord
{
    private decimal _baseSalary;
    private decimal _bonus;

    public Guid EmployeeId { get; }

    public decimal BaseSalary
    {
        get => _baseSalary;
        private set
        {
            if (value < 30_000m)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(value), value, "Base salary is below this sample's configured minimum.");
            }

            _baseSalary = value;
        }
    }

    public decimal Bonus
    {
        get => _bonus;
        private set
        {
            if (value < 0m)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(value), value, "Bonus cannot be negative.");
            }

            _bonus = value;
        }
    }

    public decimal TotalCompensation => checked(BaseSalary + Bonus);

    public EmployeeSalaryRecord(Guid employeeId, decimal baseSalary)
    {
        if (employeeId == Guid.Empty)
        {
            throw new ArgumentException("Employee ID cannot be empty.", nameof(employeeId));
        }

        EmployeeId = employeeId;
        BaseSalary = baseSalary;
    }

    public void ChangeBaseSalary(decimal newBaseSalary)
    {
        BaseSalary = newBaseSalary;
    }

    public void AwardBonus(decimal percentage)
    {
        if (percentage is <= 0m or > 100m)
        {
            throw new ArgumentOutOfRangeException(
                nameof(percentage), percentage, "Percentage must be greater than zero and at most 100.");
        }

        // Illustrative two-decimal rounding; payroll policy must define its own rule.
        decimal award = decimal.Round(
            BaseSalary * percentage / 100m,
            2,
            MidpointRounding.ToEven);

        Bonus = checked(Bonus + award);
    }
}
```

The 30,000 minimum is an **illustrative domain rule**, not a statement about any jurisdiction's minimum wage. The example rounds bonus amounts to two decimals using `MidpointRounding.ToEven`; a production payroll system must use its documented currency and rounding policy.

## 5. Applied Backend Example: Encapsulated Digital Wallet

A wallet aggregate can keep its balance and ledger under one public contract. This example protects the list structure with a cached read-only wrapper, stores immutable ledger entries, validates each transaction before changing state, and injects `TimeProvider` so timestamps can be controlled in tests.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;

namespace MyBackendApp.Core.Domain.Wallets;

public enum TransactionType
{
    Deposit,
    Withdrawal
}

public sealed record LedgerEntry(
    Guid TransactionId,
    decimal Amount,
    TransactionType Type,
    DateTimeOffset TimestampUtc,
    string ReferenceCode);

public sealed class DigitalWallet
{
    private readonly List<LedgerEntry> _ledger = [];
    private readonly ReadOnlyCollection<LedgerEntry> _ledgerView;
    private readonly TimeProvider _timeProvider;
    private decimal _balance;

    public Guid WalletId { get; }
    public Guid OwnerId { get; }
    public string Currency { get; }

    public IReadOnlyList<LedgerEntry> Ledger => _ledgerView;
    public decimal Balance => _balance;
    public bool IsFrozen { get; private set; }

    public DigitalWallet(
        Guid walletId,
        Guid ownerId,
        string currency,
        TimeProvider? timeProvider = null)
    {
        if (walletId == Guid.Empty)
        {
            throw new ArgumentException("Wallet ID cannot be empty.", nameof(walletId));
        }

        if (ownerId == Guid.Empty)
        {
            throw new ArgumentException("Owner ID cannot be empty.", nameof(ownerId));
        }

        WalletId = walletId;
        OwnerId = ownerId;
        Currency = NormalizeCurrency(currency);
        _timeProvider = timeProvider ?? TimeProvider.System;
        _ledgerView = _ledger.AsReadOnly();
    }

    public LedgerEntry Deposit(decimal amount, string referenceCode)
    {
        EnsureWalletActive();
        ValidateAmount(amount);
        string normalizedReference = NormalizeReferenceCode(referenceCode);

        decimal updatedBalance = checked(_balance + amount);
        var entry = new LedgerEntry(
            TransactionId: Guid.NewGuid(),
            Amount: amount,
            Type: TransactionType.Deposit,
            TimestampUtc: _timeProvider.GetUtcNow(),
            ReferenceCode: normalizedReference);

        // Add the entry before committing the balance so a failed list growth doesn't change it.
        _ledger.Add(entry);
        _balance = updatedBalance;
        return entry;
    }

    public LedgerEntry Withdraw(decimal amount, string referenceCode)
    {
        EnsureWalletActive();
        ValidateAmount(amount);
        string normalizedReference = NormalizeReferenceCode(referenceCode);

        if (amount > _balance)
        {
            throw new InvalidOperationException(
                $"Insufficient funds. Balance: {_balance} {Currency}; requested: {amount} {Currency}.");
        }

        decimal updatedBalance = checked(_balance - amount);
        var entry = new LedgerEntry(
            TransactionId: Guid.NewGuid(),
            Amount: amount,
            Type: TransactionType.Withdrawal,
            TimestampUtc: _timeProvider.GetUtcNow(),
            ReferenceCode: normalizedReference);

        _ledger.Add(entry);
        _balance = updatedBalance;
        return entry;
    }

    public void Freeze()
    {
        if (IsFrozen)
        {
            throw new InvalidOperationException("Wallet is already frozen.");
        }

        IsFrozen = true;
    }

    public void Unfreeze()
    {
        if (!IsFrozen)
        {
            throw new InvalidOperationException("Wallet is not frozen.");
        }

        IsFrozen = false;
    }

    private void EnsureWalletActive()
    {
        if (IsFrozen)
        {
            throw new InvalidOperationException(
                "Wallet operations are suspended because this wallet is frozen.");
        }
    }

    private static void ValidateAmount(decimal amount)
    {
        if (amount <= 0m)
        {
            throw new ArgumentOutOfRangeException(
                nameof(amount), amount, "Transaction amount must be positive.");
        }
    }

    private static string NormalizeReferenceCode(string referenceCode)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(referenceCode);
        return referenceCode.Trim();
    }

    private static string NormalizeCurrency(string currency)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(currency);
        string normalized = currency.Trim().ToUpperInvariant();

        if (normalized.Length != 3)
        {
            throw new ArgumentException(
                "Currency must be a three-letter code.", nameof(currency));
        }

        foreach (char character in normalized)
        {
            if (!char.IsAsciiLetter(character))
            {
                throw new ArgumentException(
                    "Currency must contain only ASCII letters.", nameof(currency));
            }
        }

        return normalized;
    }
}
```

The wallet's `Ledger` property is a read-only **live view**; wallet methods can add entries, and callers see those entries without receiving a mutable list. `LedgerEntry` is a record with init-only positional properties, so callers can't normally change an entry after it's created. The currency helper checks only the shape of a three-letter code; a production system should validate against supported currency codes and define permitted minor units.

This aggregate protects one wallet's in-memory invariants. Encapsulation alone doesn't make an object thread-safe or provide database atomicity. A cross-wallet transfer or currency exchange must be coordinated by an application service and a database transaction (with concurrency and idempotency rules); calling two wallet methods separately doesn't make the overall operation atomic.

## 6. Common Mistakes and Design Guidance

- **Confusing a get-only property with an immutable collection:** the property may return a mutable `List<T>` reference.
- **Treating `IReadOnlyList<T>` as a runtime barrier:** the actual object may still be a mutable list that can be cast back. Return a read-only wrapper or an immutable collection.
- **Calling `AsReadOnly()` a defensive copy:** it wraps the original list. It is read-only to the caller but reflects owner changes; a snapshot is a separate copy.
- **Assuming a shallow copy makes elements immutable:** copied references still point to the same mutable objects.
- **Using `init` as validation:** it limits when assignment can occur; add validation where values enter the domain.
- **Treating `required` as non-null or runtime-safe:** it checks that construction code assigns a member, not that the value is meaningful or non-null at runtime.
- **Using public setters for important domain transitions:** prefer named operations when state changes have business rules.
- **Calling every data-only type an anti-pattern:** DTOs and simple persistence shapes may be intentionally anemic; richer behavior belongs where it clarifies and protects domain rules.
- **Assuming encapsulation provides thread safety or transactionality:** coordinate concurrent access and persistence with locks, optimistic concurrency, and database transactions as appropriate.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Encapsulation** | Grouping state with the operations that govern it and restricting direct access to implementation details. | Keeps callers on a contract that can preserve business rules. |
| **Invariant** | A condition that must hold for an object to be considered valid. | Prevents invalid states such as a negative wallet balance. |
| **Backing field** | A field that stores a property's value. | Supports validation, normalization, and controlled mutation. |
| **Auto-property** | A property whose backing field and accessors are generated by the compiler. | Reduces boilerplate for simple stored values. |
| **`init` accessor** | An accessor assignable during supported construction contexts but not by ordinary later assignment. | Allows object initialization while limiting subsequent writes. |
| **`required` member** | A field or property the compiler requires creation code to initialize. | Catches omitted assignments at compile time; it doesn't validate the assigned value at runtime. |
| **Asymmetric accessors** | A property whose getter and setter/init accessor have different accessibility. | Enables public reads with restricted writes. |
| **Read-only wrapper** | A wrapper that blocks collection edits through the exposed view while reflecting changes to the underlying list. | Prevents direct list mutation without copying the collection. |
| **Defensive copy / snapshot** | A separate copy returned so changes to its collection structure don't change the original. | Gives callers a snapshot; copying is shallow unless elements are copied too. |
| **Anemic domain model** | A domain data model with little behavior while rules are mostly outside it. | May scatter invariants in DDD; can still be appropriate for DTOs and simple data shapes. |
| **Rich domain model** | A model that places relevant domain behavior and invariant checks near the state they govern. | Provides explicit, rule-preserving state transitions. |

## 8. Official References

- [Properties — C# programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties) — accessors, backing fields, required properties, and accessor accessibility.
- [`init` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/init) — initialization-only property and indexer accessors.
- [`required` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/required) — required fields/properties and `SetsRequiredMembers`.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — default language versions, including .NET 10 / C# 14.
- [`List<T>.AsReadOnly` API](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.asreadonly?view=net-10.0) — read-only wrapper behavior.
- [`ReadOnlyCollection<T>` API](https://learn.microsoft.com/en-us/dotnet/api/system.collections.objectmodel.readonlycollection-1?view=net-10.0) — wrapper collection type.
- [`TimeProvider` API](https://learn.microsoft.com/en-us/dotnet/api/system.timeprovider?view=net-10.0) — injectable time abstraction for deterministic tests.
- [`ArgumentException.ThrowIfNullOrWhiteSpace` API](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception.throwifnullorwhitespace?view=net-10.0) — runtime validation helper available in .NET 8 and later.
- [Records — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) — record types and init-only properties.
