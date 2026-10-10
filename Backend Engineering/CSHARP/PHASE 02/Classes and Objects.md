Examples target **.NET 10** with nullable reference analysis enabled. A current .NET 10 SDK normally defaults to C# 14 unless the project overrides `LangVersion`; primary constructors for classes and structs were introduced in C# 12. Code blocks are self-contained snippets unless labeled as a fragment; fragments are meant to be placed inside a class or project file as indicated.

A minimal project setup can look like this:
```xml
<!-- PropertyGroup excerpt for a project file. -->
<PropertyGroup>
  <TargetFramework>net10.0</TargetFramework>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

## 1. Conceptual Foundation

### 1.1 Class, Object, and Reference

A **class** is a reference type declaration: it defines a type, its members, and the behavior of its instances. An **object** (or **instance**) is a particular runtime value of that class.

A variable of class type holds a **reference** to an object; it doesn't contain the object's instance fields. For example, `var customer = new Customer("Amina");` creates a `Customer` and stores a reference in `customer`. Assigning one class variable to another copies the reference, so both variables can refer to the same object:

```csharp
using System;

var first = new Counter();
var alias = first; // Copies the reference; it does not clone the object.

alias.Increment();
Console.WriteLine(first.Value); // 1: both variables refer to the same Counter.

public sealed class Counter
{
    public int Value { get; private set; }

    public void Increment() => Value++;
}
```

### 1.2 A Conceptual Memory View

A class instance is ordinarily allocated on the **managed heap** and managed by the garbage collector. A reference to it can be held in a local, a field of another object, a register, or another GC root. The diagram is conceptual: C# references aren't stable raw addresses, and the runtime can move objects while keeping tracked references valid.

```text
Reference-bearing location                    Managed heap
(location depends on the program/runtime)
+----------------------+                       +-------------------------+
| customer reference   | --------------------> | Customer object         |
+----------------------+                       | Name: "Amina"           |
                                                | ... instance state ...  |
                                                +-------------------------+
```

The CLR also maintains object and type information, but exact object-header layout, synchronization bookkeeping, and allocation details are runtime implementation details. Don't depend on a particular header size or on a fixed "method-table pointer plus sync-block index" layout.

At a high level, when `new` creates a class instance:

1. The runtime allocates storage for the object and initializes its fields to their default values (`0`, `false`, `null`, and so on).
2. Instance field and auto-property initializers run as part of construction.
3. Base and derived constructors execute in the order described in [Section 3.3](#33-instance-initialization-order).
4. If the creation expression has an object initializer, those assignments run after the constructor chain completes.

This sequence describes C# initialization behavior; the runtime and JIT remain free to optimize implementation details. A reference variable isn't guaranteed to live on the stack.

## 2. Structural Members of a Class

### 2.1 Fields

A **field** is a variable declared directly in a class or struct. An instance field belongs to one object; a `static` field belongs to the type and is shared by its instances. Fields are commonly used for internal state, while methods and properties provide controlled access to that state.

These are **field-declaration fragments** to place inside a class; the containing file needs `using System;` and `using System.Collections.Generic;`.

```csharp
private readonly Guid _id = Guid.NewGuid();
private decimal _balance;
private readonly List<string> _tags = [];
```

- Prefer `private` fields for implementation details. Public fields let callers change state without validation or a stable API boundary.
- Fields receive their type's default value before field initializers and constructors run. Use explicit initializers where a non-default value is part of the design.
- `readonly` restricts **reassignment of the field**: an instance `readonly` field can be assigned in its declaration or an instance constructor of its declaring type; a `static readonly` field can be assigned in its declaration or the declaring type's static constructor. This is a C# compile-time rule, not a security boundary.
- A `readonly` reference field can't be pointed at a different object after construction, but the referenced object can still be mutable. For example, `_tags.Add("priority")` is allowed even though `_tags` is `readonly`.

| Modifier | Main use | Important distinction |
|---|---|---|
| `const` | Compile-time constant | Must be initialized at its declaration and is substituted as a constant by the compiler. |
| `readonly` | Value fixed after the permitted initialization period | Can be initialized using runtime values and, for a reference type, doesn't make the referenced object immutable. |

### 2.2 Properties

A **property** is a member with accessors (`get`, `set`, or `init`) that lets callers read, write, or compute a value. Properties are implemented as accessor methods; a property isn't necessarily a field and doesn't always store a value.

| Property form | Typical use | Storage |
|---|---|---|
| Auto-implemented (`{ get; set; }`) | Simple state with no custom accessor logic | The compiler generates a private backing field. |
| Full property | Validation, normalization, notification, or other accessor logic | May use an explicit backing field, as needed. |
| Getter-only | State assigned by a constructor or initializer and then exposed read-only | Usually an auto-property backing field or a computed value. |
| `init` accessor | A value callers may set during object construction, but not through an ordinary later assignment | Can be auto-implemented or use a backing field. |
| Computed / expression-bodied | A value derived from other state | No separate property storage is required. |

This complete type shows several forms together:

```csharp
#nullable enable
using System;

public sealed class PersonProfile
{
    private string _email = string.Empty;

    public Guid Id { get; } = Guid.NewGuid(); // Auto-property; getter-only after construction.

    public string Email
    {
        get => _email;
        set
        {
            ArgumentException.ThrowIfNullOrWhiteSpace(value);
            _email = value.Trim();
        }
    }

    public string FirstName { get; init; } = string.Empty; // Auto-property with init.
    public string LastName { get; init; } = string.Empty;

    public string FullName => $"{FirstName} {LastName}".Trim(); // Computed property.

    public PersonProfile(string email)
    {
        Email = email;
    }
}
```

The `Email` setter rejects null, empty, or whitespace-only input and trims surrounding whitespace; it does **not** perform full email-address validation. `init` limits when a property can be assigned, but doesn't make an entire object deeply immutable. For example, an `init` property can hold a reference to a mutable list. Also, `init` doesn't require a caller to provide a value; the separate C# `required` modifier can express a compile-time initialization requirement, but runtime validation may still be needed.

Property accessors can execute arbitrary code, so keep getters inexpensive and unsurprising. A read-only property may still calculate its result on every access.

## 3. Constructors and Object Lifecycle

A **constructor** is a special member called to initialize a new class instance. It has the same name as its class and no return type. Constructors are a natural place to establish the object's initial invariants: the rules that should remain true for every valid instance.

### 3.1 Constructor Forms

- **Implicit parameterless constructor:** If a class declares no instance constructors, the compiler supplies a parameterless constructor with the class's accessibility. If you declare an instance constructor—including a primary constructor—the compiler doesn't also supply that implicit constructor. Declare one explicitly if callers need it.
- **Parameterized constructor:** Accepts the values needed to initialize the object. Validate required inputs before the object is exposed to callers.
- **Constructor chaining with `this(...)`:** Delegates to another constructor in the same class. The delegated constructor runs first, so shared validation and initialization can live in one place.
- **Base-constructor call with `base(...)`:** Selects a constructor on the base class. If the base class has no accessible parameterless constructor, a derived constructor must call an appropriate base constructor explicitly.
- **Primary constructor:** C# 12 introduced primary constructors for classes and structs. Parameters appear in the type declaration and are in scope throughout the type body. For an ordinary class, those parameters are **not automatically properties**; the compiler creates storage only when the parameters are used in a way that requires it. Use explicit properties or fields when the type should expose or retain values.
- **Static constructor:** Initializes static state for a type. It has no parameters and isn't an instance constructor.

A primary constructor can be concise, but it doesn't have the same explicit constructor body as a traditional constructor. If initialization requires a longer validation sequence, a normal constructor or a factory method may be clearer.

### 3.2 Constructor Chaining

In this **constructor fragment**, the two-argument constructor delegates to the three-argument constructor, so the validation and initialization logic is not duplicated:

```csharp
public BankAccount(string accountNumber, string accountHolder)
    : this(accountNumber, accountHolder, 0m)
{
}
```

A constructor initializer (`: this(...)` or `: base(...)`) is executed before that constructor's body. A constructor can choose one of these paths, not both; a `this(...)` chain eventually reaches a constructor that calls a base constructor.

### 3.3 Instance Initialization Order

For an instance created with `new`, C# initialization proceeds in this order:

1. Instance fields start with their default values (`0`, `false`, `null`, and so on).
2. Field and auto-property initializers run for the most-derived class, in textual order.
3. Field and auto-property initializers run for each base class, from the direct base toward `System.Object`, in textual order within each class.
4. Base-constructor bodies run from `System.Object` toward the direct base class.
5. The most-derived class's constructor body runs.
6. Any object-initializer assignments run after the constructor chain, in the order written in the initializer.

So, for a derived object, **derived field initializers run before base field initializers**, while **base constructor bodies run before the derived constructor body**. This is a common source of surprises. Avoid calling overridable instance methods from constructors: the call can dispatch to a derived override before that derived constructor body has finished.

This small complete program prints the order of the field initializers and constructor bodies:

```csharp
using System;

_ = new Derived();

public class Base
{
    public string BaseValue { get; } = Log("Base field initializer");

    public Base() => Console.WriteLine("Base constructor body");

    protected static string Log(string message)
    {
        Console.WriteLine(message);
        return message;
    }
}

public sealed class Derived : Base
{
    public string DerivedValue { get; } = Log("Derived field initializer");

    public Derived() => Console.WriteLine("Derived constructor body");
}
```

Expected output:

```text
Derived field initializer
Base field initializer
Base constructor body
Derived constructor body
```

## 4. Basic Syntax Example: `BankAccount`

This complete type demonstrates private fields, a `readonly` field, auto and full properties, an `init` accessor, a computed property, constructor validation, and constructor chaining. It is a compile-ready class file; add it to a .NET 10 project.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public sealed class BankAccount
{
    private readonly string _accountNumber;
    private string _accountHolder = string.Empty;
    private decimal _balance;

    public Guid Id { get; } = Guid.NewGuid(); // Auto-property with a getter only.

    public string AccountNumber => _accountNumber; // Read-only public view of the field.

    public string AccountHolder
    {
        get => _accountHolder;
        init
        {
            ArgumentException.ThrowIfNullOrWhiteSpace(value);
            _accountHolder = value.Trim();
        }
    }

    public decimal Balance
    {
        get => _balance;
        private set
        {
            if (value < 0)
            {
                throw new ArgumentOutOfRangeException(nameof(value), "Balance cannot be negative.");
            }

            _balance = value;
        }
    }

    public bool HasPositiveBalance => Balance > 0;

    public BankAccount(string accountNumber, string accountHolder, decimal initialDeposit)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(accountNumber);

        if (initialDeposit < 0)
        {
            throw new ArgumentOutOfRangeException(
                nameof(initialDeposit), initialDeposit, "Initial deposit must be non-negative.");
        }

        _accountNumber = accountNumber.Trim();
        AccountHolder = accountHolder;
        _balance = initialDeposit;
    }

    public BankAccount(string accountNumber, string accountHolder)
        : this(accountNumber, accountHolder, 0m)
    {
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
        {
            throw new ArgumentOutOfRangeException(
                nameof(amount), amount, "Deposit amount must be greater than zero.");
        }

        Balance = checked(Balance + amount);
    }
}
```

`AccountHolder` uses an `init` accessor and validates both assignments made by the constructor and values supplied by an object initializer. A valid initializer can override the constructor-supplied value during the creation expression:

A call site can use it like this (usage fragment):

```csharp
var account = new BankAccount("AC-1042", "Amina", 100m)
{
    AccountHolder = "Amina Hassan"
};
```

Use a getter-only property instead if that override shouldn't be allowed. `readonly` on `_accountNumber` prevents replacing the stored string reference after construction; `string` itself is immutable.

## 5. Applied Backend Example: An Order Aggregate

In domain-oriented backend design, an **entity** has identity and behavior, while an **aggregate root** controls changes to a group of related objects. The example below keeps order state private, validates line items, permits mutations only while the order is a draft, and exposes a read-only collection view. It also demonstrates a C# 12 primary constructor on a line-item type whose identity, SKU, and unit price are fixed after creation, while quantity changes are mediated by the order aggregate, plus a named factory method for creating a draft order.

The example assumes every price in one order uses the same currency. Production code should make its currency and rounding rules explicit. The factory uses `DateTimeOffset.UtcNow` for brevity; a tested production design may supply a clock such as `TimeProvider` from the application layer.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;

namespace MyBackendApp.Core.Domain.Entities;

public enum OrderStatus
{
    Draft = 0,
    Submitted = 1,
    Paid = 2,
    Cancelled = 3
}

// C# 12 primary constructor. These parameters are used to initialize explicit properties;
// they don't become public properties automatically.
public sealed class OrderItem(Guid productId, string sku, decimal unitPrice, int quantity)
{
    public Guid Id { get; } = Guid.NewGuid();
    public Guid ProductId { get; } = ValidateProductId(productId);
    public string Sku { get; } = NormalizeSku(sku);
    public decimal UnitPrice { get; } = ValidateUnitPrice(unitPrice);
    public int Quantity { get; private set; } = ValidateQuantity(quantity);

    public decimal Subtotal => UnitPrice * Quantity;

    // Internal so callers outside the domain assembly must ask the aggregate root to change it.
    internal void ChangeQuantity(int quantity)
    {
        Quantity = ValidateQuantity(quantity);
    }

    private static Guid ValidateProductId(Guid productId)
    {
        if (productId == Guid.Empty)
        {
            throw new ArgumentException("Product ID cannot be empty.", nameof(productId));
        }

        return productId;
    }

    private static string NormalizeSku(string sku)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(sku);
        return sku.Trim();
    }

    private static decimal ValidateUnitPrice(decimal unitPrice)
    {
        if (unitPrice <= 0)
        {
            throw new ArgumentOutOfRangeException(
                nameof(unitPrice), unitPrice, "Unit price must be greater than zero.");
        }

        return unitPrice;
    }

    private static int ValidateQuantity(int quantity)
    {
        if (quantity <= 0)
        {
            throw new ArgumentOutOfRangeException(
                nameof(quantity), quantity, "Quantity must be at least one.");
        }

        return quantity;
    }
}

public sealed class Order
{
    private readonly List<OrderItem> _items = [];
    private readonly ReadOnlyCollection<OrderItem> _itemsView;

    private Order(Guid customerId)
    {
        if (customerId == Guid.Empty)
        {
            throw new ArgumentException("Customer ID cannot be empty.", nameof(customerId));
        }

        Id = Guid.NewGuid();
        CustomerId = customerId;
        CreatedAtUtc = DateTimeOffset.UtcNow;
        Status = OrderStatus.Draft;
        _itemsView = _items.AsReadOnly();
    }

    public Guid Id { get; }
    public Guid CustomerId { get; }
    public DateTimeOffset CreatedAtUtc { get; }
    public OrderStatus Status { get; private set; }

    // A read-only live view: callers can't add or remove entries through this property.
    public IReadOnlyCollection<OrderItem> Items => _itemsView;

    public decimal TotalAmount => _items.Sum(item => item.Subtotal);

    public static Order CreateDraft(Guid customerId) => new(customerId);

    public void AddItem(Guid productId, string sku, decimal unitPrice, int quantity)
    {
        EnsureDraft();
        _items.Add(new OrderItem(productId, sku, unitPrice, quantity));
    }

    public void ChangeItemQuantity(Guid itemId, int newQuantity)
    {
        EnsureDraft();
        FindItem(itemId).ChangeQuantity(newQuantity);
    }

    public void RemoveItem(Guid itemId)
    {
        EnsureDraft();
        _items.Remove(FindItem(itemId));
    }

    public void Submit()
    {
        EnsureDraft();

        if (_items.Count == 0)
        {
            throw new InvalidOperationException("Cannot submit an empty order.");
        }

        Status = OrderStatus.Submitted;
    }

    public void MarkAsPaid()
    {
        if (Status != OrderStatus.Submitted)
        {
            throw new InvalidOperationException("Only a submitted order can be marked as paid.");
        }

        Status = OrderStatus.Paid;
    }

    public void Cancel()
    {
        if (Status is OrderStatus.Paid or OrderStatus.Cancelled)
        {
            throw new InvalidOperationException($"An order in '{Status}' status cannot be cancelled.");
        }

        Status = OrderStatus.Cancelled;
    }

    private void EnsureDraft()
    {
        if (Status != OrderStatus.Draft)
        {
            throw new InvalidOperationException($"This operation requires a draft order; current status is '{Status}'.");
        }
    }

    private OrderItem FindItem(Guid itemId)
    {
        if (itemId == Guid.Empty)
        {
            throw new ArgumentException("Order-item ID cannot be empty.", nameof(itemId));
        }

        return _items.Find(item => item.Id == itemId)
            ?? throw new KeyNotFoundException("Item not found in this order.");
    }
}
```

Key design points:

- `Order.CreateDraft` is a named factory method. Its private constructor prevents callers from bypassing the creation path; the factory can later grow to use an injected clock or additional creation policy.
- The primary constructor parameters on `OrderItem` initialize explicit properties. In an ordinary class, primary-constructor parameters don't automatically become properties.
- A failed `OrderItem` validation occurs before the item is added, so a rejected `AddItem` call doesn't partially mutate the order.
- The aggregate root checks order status before adding, changing, or removing line items. The line item's quantity mutation is internal to the domain assembly rather than exposed as a public operation.
- `Items` is a read-only live view over the list. The view blocks collection edits through this property; the line item also limits quantity changes, so callers can't bypass the aggregate's status checks through a public item method.
- `TotalAmount` is computed from the line-item snapshots. `decimal` is appropriate for many financial calculations, but real systems must still define currency, precision, rounding, and overflow policy.

## 6. Common Mistakes and Design Guidance

- **Treating a class variable as the object itself:** assignment copies a reference. Mutating through one alias can be observed through another.
- **Assuming a reference always lives on the stack:** storage and JIT optimizations vary; reason about managed references and object lifetime instead.
- **Depending on object-header layout:** exact layout is not a portable C# contract.
- **Exposing fields publicly:** callers can bypass validation and make the type harder to change safely.
- **Assuming `readonly` means deeply immutable:** a readonly reference to a mutable collection still permits changes to that collection.
- **Assuming every property has a backing field:** auto-properties do; computed properties need not. A full property may use a field or compute its value.
- **Assuming `init` guarantees a valid or deeply immutable object:** `init` restricts assignment timing. Validation and object-graph immutability are separate concerns.
- **Doing all initialization in property setters or object initializers:** prefer constructors or factories when several values must be valid together or callers must not create invalid states.
- **Forgetting constructor chaining rules:** `this(...)` delegates to another constructor; it is not a call that can be combined with `base(...)` in the same initializer.
- **Calling virtual methods from a constructor:** a derived override may run before the derived constructor body has completed.
- **Overusing primary constructors:** they are concise when parameters initialize straightforward state. A traditional constructor or factory is often clearer for complex validation, side effects, or multiple creation paths.

## 7. Key Terms Summary

| Term | Definition | Backend use |
|---|---|---|
| **Class** | A reference type declaration that defines members and behavior for its instances. | Models entities, services, repositories, and other reference-oriented types. |
| **Object / instance** | A runtime value created from a class. | Holds state and participates in application behavior. |
| **Reference** | A managed value that identifies an object; copying it can create another alias to the same object. | Explains shared identity and mutation across layers. |
| **Field** | A variable declared directly in a class or struct. | Stores implementation state, commonly with private access. |
| **`readonly` field** | A field that can be assigned only during its permitted initialization contexts. | Keeps an identity/reference from being replaced after construction. |
| **Property** | A member whose accessors read, write, or compute a value. | Exposes a stable API with controlled access and validation. |
| **Backing field** | A field that stores the value exposed by a property. | Supports encapsulation and accessor logic. |
| **Auto-property** | A property for which the compiler generates accessors and a backing field. | Reduces boilerplate for simple stored values. |
| **`init` accessor** | A setter restricted to object-initialization contexts. | Allows initialization while preventing ordinary later assignments. |
| **Computed property** | A property whose value is derived when read. | Exposes totals, display values, and other derived state without duplicated storage. |
| **Constructor** | A member invoked to initialize a new instance. | Establishes object invariants and required state. |
| **Constructor chaining** | Delegation to another constructor with `this(...)` or a base constructor with `base(...)`. | Avoids duplicated initialization and supports inheritance. |
| **Primary constructor** | C# 12 syntax that declares constructor parameters on a class or struct declaration. | Concise initialization; class parameters aren't automatically properties. |
| **Factory method** | A method that creates and returns an instance. | Names creation intent and centralizes validation or creation policy. |
| **Aggregate root** | The entity through which an aggregate's state changes are controlled. | Helps protect business invariants across related domain objects. |

## 8. Official References

- [C# classes](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes) — class declarations, object references, and construction.
- [Fields — C# programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/fields) — instance and static fields, access, and initialization.
- [Properties — C# programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties) — auto-properties, accessors, computed properties, and access control.
- [`init` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/init) — init-only property semantics.
- [`readonly` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly) — field assignment rules and reference-type caveats.
- [Constructors — C# programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors) — constructor forms and instance initialization order.
- [Primary constructors — C# tutorial](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/primary-constructors) — C# 12 primary-constructor parameters and storage behavior.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — default language versions, including .NET 10 / C# 14.
- [Fundamentals of garbage collection — .NET](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals) — managed-heap allocation and object lifetime.
- [`ArgumentException.ThrowIfNullOrWhiteSpace` API](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception.throwifnullorwhitespace?view=net-10.0) — string argument guard available in .NET 8 and later.
