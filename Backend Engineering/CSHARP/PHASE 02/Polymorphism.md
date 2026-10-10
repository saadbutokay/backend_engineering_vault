Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 What Is Polymorphism?

**Polymorphism** means “many forms”: code can work through a shared base type or interface while the concrete implementation supplies type-specific behavior. A payment service, for example, can depend on `IPaymentGateway` rather than on a particular provider class.

```text
                    +--------------------------+
                    |      IPaymentGateway     |
                    | ProcessPaymentAsync(...) |
                    +--------------------------+
                                  ^
                   implements    |    implements
                    +-------------+-------------+
                    |                           |
       +-------------------------+  +-------------------------+
       | StripePaymentGateway    |  | PayPalPaymentGateway    |
       | provider-specific code  |  | provider-specific code  |
       +-------------------------+  +-------------------------+
```

The caller uses the common contract; it doesn't need a `switch` for every concrete implementation. This can support the **Open/Closed Principle (OCP)**—a component can be extended with another implementation without changing its stable caller. Polymorphism doesn't guarantee OCP by itself: registration, policy changes, or a poorly designed abstraction can still require modifying existing code.

### 1.2 Two Common Forms in C#

#### Static / Compile-Time Polymorphism

The compiler resolves which member to call during compilation. Common examples include:

- **Method overloading:** methods share a name but have distinct parameter signatures. Overloads can differ by parameter count, types, or order; they can't differ only by return type, parameter names, or default values. The compiler selects an applicable overload using the argument types and other compile-time information.
- **Operator overloading:** a user-defined type supplies a custom meaning for supported operators such as `+` or `==`. The compiler selects the operator based on the operand types.

These forms improve API expression, but they are not runtime selection among object implementations.

#### Dynamic / Runtime Polymorphism

For a **virtual call**, the runtime invokes the most-derived override for the receiver's actual type. For an **interface call**, the runtime invokes the implementation associated with the receiver's actual type. A caller can hold the object through a base-class or interface-typed variable without knowing its concrete type.

- **Inheritance-based dispatch:** a base member is `virtual` or `abstract`, and a derived class supplies an `override`.
- **Interface-based dispatch:** unrelated classes implement the same interface and can be used wherever that interface is expected. A shared base class isn't required.

The C# keyword `dynamic` is a separate runtime-binding feature; ordinary virtual and interface dispatch don't require a variable declared as `dynamic`.

## 2. A Safe Mental Model for Runtime Dispatch

At the language level, the important steps are:

1. The compiler determines that a call is virtual or made through an interface.
2. At runtime, the runtime type of the receiver determines the applicable implementation.
3. The selected implementation runs, while the call site continues to use the base or interface contract.

This behavior is sometimes taught using a **virtual method table** or method-table slots. The CLR uses runtime type metadata and dispatch structures to implement calls, but the exact object layout, headers, pointers, and lookup steps are runtime implementation details—not a C# language contract. Don't assume every object exposes a particular “type handle” pointer or depend on a specific vtable layout. The useful rule is that virtual and interface calls dispatch according to the receiver's runtime type; ordinary nonvirtual member selection is different.

For example, if `IPaymentGateway gateway` refers to a `StripePaymentGateway`, invoking `gateway.ProcessPaymentAsync(...)` uses Stripe's implementation. The caller still compiles against `IPaymentGateway`.

## 3. Basic Syntax Example

This source file demonstrates compile-time overload selection and runtime method/property overrides. The provider classes print illustrative messages; they don't call external services.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public sealed class DataExporter
{
    // The compiler selects this overload for a single string argument.
    public void Export(string rawData)
    {
        ArgumentNullException.ThrowIfNull(rawData);
        Console.WriteLine($"Exporting raw string payload: {rawData}");
    }

    // A different parameter list makes this a separate overload.
    public void Export(string rawData, string targetPath)
    {
        ArgumentNullException.ThrowIfNull(rawData);
        ArgumentException.ThrowIfNullOrWhiteSpace(targetPath);
        Console.WriteLine($"Exporting raw string to '{targetPath}': {rawData}");
    }

    public void Export(byte[] binaryData, string targetPath)
    {
        ArgumentNullException.ThrowIfNull(binaryData);
        ArgumentException.ThrowIfNullOrWhiteSpace(targetPath);
        Console.WriteLine($"Exporting {binaryData.Length} bytes to '{targetPath}'");
    }
}

public interface INotificationProvider
{
    string ProviderName { get; }
    void Send(string recipient, string message);
}

public abstract class BaseNotificationProvider : INotificationProvider
{
    public virtual string ProviderName => "Generic";

    protected static void ValidateMessage(string recipient, string message)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(recipient);
        ArgumentNullException.ThrowIfNull(message);
    }

    public virtual void Send(string recipient, string message)
    {
        ValidateMessage(recipient, message);
        Console.WriteLine($"[Default Provider] Sending to {recipient}: {message}");
    }
}

public sealed class TwilioSmsNotificationProvider : BaseNotificationProvider
{
    public override string ProviderName => "Twilio-SMS";

    public override void Send(string recipient, string message)
    {
        ValidateMessage(recipient, message);
        Console.WriteLine($"[{ProviderName}] Dispatching SMS to {recipient}: {message}");
    }
}

public sealed class SendGridEmailNotificationProvider : BaseNotificationProvider
{
    public override string ProviderName => "SendGrid-Email";

    public override void Send(string recipient, string message)
    {
        ValidateMessage(recipient, message);
        Console.WriteLine($"[{ProviderName}] Dispatching email to {recipient}: {message}");
    }
}

public static class Demo
{
    public static void Main()
    {
        var exporter = new DataExporter();
        exporter.Export("payload");
        exporter.Export("payload", "out.txt");
        exporter.Export(new byte[] { 0x01, 0x02 }, "out.bin");

        INotificationProvider provider = new TwilioSmsNotificationProvider();
        provider.Send("+1-555-0100", "Your code is 123456");

        provider = new SendGridEmailNotificationProvider();
        provider.Send("customer@example.com", "Your order shipped");
    }
}
```

The `Export` overload is chosen from the argument list at compile time. By contrast, `provider.Send(...)` is called through `INotificationProvider`; the concrete object's implementation is selected at runtime. The derived overrides preserve the base method's input checks. The same overrides would be selected if the variable were declared as `BaseNotificationProvider`.

## 4. Applied Backend Example: Polymorphic Tax Strategies

A checkout pipeline can depend on an abstract tax-calculator contract and select a strategy by a configured jurisdiction key. Adding a new calculator can then leave the checkout algorithm unchanged, provided the new strategy is registered and conforms to the contract.

> **Tax-compliance note:** The rates, codes, categories, and rounding below are illustrative programming examples, not legal or accounting advice and not production-ready U.S. or EU tax rules. Real rules vary by jurisdiction, product, buyer, date, exemptions, and reporting requirements. In particular, a zero-rated sale and an exempt sale may both produce zero output tax in a simplified calculation while remaining different legal/reporting treatments.

```csharp
#nullable enable
using System;
using System.Collections.Generic;

namespace MyBackendApp.Core.Domain.Billing;

public enum TaxTreatment
{
    Standard,
    Reduced,
    ZeroRated,
    Exempt
}

public sealed record TaxableItem(string Sku, decimal Amount, TaxTreatment Treatment);

public sealed record TaxCalculationResult(
    string Jurisdiction,
    decimal Subtotal,
    decimal TaxAmount,
    decimal Total);

public abstract class TaxCalculator
{
    public abstract string JurisdictionCode { get; }

    // Shared validation and result construction stay in one place.
    public TaxCalculationResult Calculate(IReadOnlyList<TaxableItem> items)
    {
        ArgumentNullException.ThrowIfNull(items);

        int itemCount = items.Count;
        var validatedItems = new TaxableItem[itemCount];
        decimal subtotal = 0m;

        for (int index = 0; index < itemCount; index++)
        {
            TaxableItem item = items[index]
                ?? throw new ArgumentException("Items cannot contain null entries.", nameof(items));

            ArgumentException.ThrowIfNullOrWhiteSpace(item.Sku);

            if (item.Amount < 0m)
            {
                throw new ArgumentOutOfRangeException(
                    nameof(items), item.Amount, "Item amounts cannot be negative in this sample.");
            }

            if (!Enum.IsDefined(item.Treatment))
            {
                throw new ArgumentOutOfRangeException(
                    nameof(items), item.Treatment, "Unknown tax treatment.");
            }

            subtotal = checked(subtotal + item.Amount);
            validatedItems[index] = item;
        }

        decimal taxAmount = CalculateTaxCore(validatedItems);
        if (taxAmount < 0m)
        {
            throw new InvalidOperationException("A tax strategy cannot return negative tax in this sample.");
        }

        return new TaxCalculationResult(
            Jurisdiction: JurisdictionCode,
            Subtotal: subtotal,
            TaxAmount: taxAmount,
            Total: checked(subtotal + taxAmount));
    }

    protected abstract decimal CalculateTaxCore(IReadOnlyList<TaxableItem> items);

    // Illustrative policy: rates are decimal fractions between 0 and 1 (for example, 0.0825 = 8.25%).
    protected static decimal ValidateRate(decimal rate, string parameterName)
    {
        if (rate is < 0m or > 1m)
        {
            throw new ArgumentOutOfRangeException(
                parameterName, rate, "This sample expects a decimal rate from 0 through 1.");
        }

        return rate;
    }

    protected static decimal RoundTax(decimal amount) =>
        decimal.Round(amount, 2, MidpointRounding.AwayFromZero);
}

// Simplified flat-rate example; not a complete U.S. sales-tax implementation.
public sealed class UsStyleFlatSalesTaxCalculator : TaxCalculator
{
    private readonly decimal _rate;

    public override string JurisdictionCode => "US-SAMPLE";

    public UsStyleFlatSalesTaxCalculator(decimal rate)
    {
        _rate = ValidateRate(rate, nameof(rate));
    }

    protected override decimal CalculateTaxCore(IReadOnlyList<TaxableItem> items)
    {
        decimal taxableAmount = 0m;

        foreach (TaxableItem item in items)
        {
            switch (item.Treatment)
            {
                case TaxTreatment.Standard:
                    taxableAmount = checked(taxableAmount + item.Amount);
                    break;

                case TaxTreatment.ZeroRated:
                case TaxTreatment.Exempt:
                    break;

                case TaxTreatment.Reduced:
                    throw new NotSupportedException(
                        "This flat-rate example doesn't model reduced rates.");

                default:
                    throw new ArgumentOutOfRangeException(nameof(items), "Unknown tax treatment.");
            }
        }

        return RoundTax(checked(taxableAmount * _rate));
    }
}

// Simplified standard/reduced-rate VAT strategy; not a jurisdiction's complete VAT rules.
public sealed class EuStyleVatCalculator : TaxCalculator
{
    private readonly decimal _standardRate;
    private readonly decimal _reducedRate;

    public override string JurisdictionCode => "EU-SAMPLE";

    public EuStyleVatCalculator(decimal standardRate, decimal reducedRate)
    {
        _standardRate = ValidateRate(standardRate, nameof(standardRate));
        _reducedRate = ValidateRate(reducedRate, nameof(reducedRate));
    }

    protected override decimal CalculateTaxCore(IReadOnlyList<TaxableItem> items)
    {
        decimal unroundedTax = 0m;

        foreach (TaxableItem item in items)
        {
            decimal rate = item.Treatment switch
            {
                TaxTreatment.Standard => _standardRate,
                TaxTreatment.Reduced => _reducedRate,
                TaxTreatment.ZeroRated => 0m,
                TaxTreatment.Exempt => 0m,
                _ => throw new ArgumentOutOfRangeException(nameof(items), "Unknown tax treatment.")
            };

            unroundedTax = checked(unroundedTax + checked(item.Amount * rate));
        }

        return RoundTax(unroundedTax);
    }
}

public sealed class CheckoutService
{
    private readonly Dictionary<string, TaxCalculator> _taxCalculators =
        new(StringComparer.OrdinalIgnoreCase);

    public CheckoutService(IEnumerable<TaxCalculator> calculators)
    {
        ArgumentNullException.ThrowIfNull(calculators);

        foreach (TaxCalculator calculator in calculators)
        {
            ArgumentNullException.ThrowIfNull(calculator);
            ArgumentException.ThrowIfNullOrWhiteSpace(calculator.JurisdictionCode);

            string code = calculator.JurisdictionCode.Trim();
            if (!_taxCalculators.TryAdd(code, calculator))
            {
                throw new ArgumentException(
                    $"More than one tax calculator is registered for '{code}'.",
                    nameof(calculators));
            }
        }
    }

    public TaxCalculationResult ProcessOrderTaxes(
        string regionCode,
        IReadOnlyList<TaxableItem> items)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(regionCode);

        string normalizedRegionCode = regionCode.Trim();
        if (!_taxCalculators.TryGetValue(normalizedRegionCode, out TaxCalculator calculator))
        {
            throw new NotSupportedException(
                $"No tax calculator is registered for region '{normalizedRegionCode}'.");
        }

        return calculator.Calculate(items);
    }
}

public static class TaxDemo
{
    public static void Main()
    {
        TaxCalculator[] calculators =
        {
            new UsStyleFlatSalesTaxCalculator(0.0825m),
            new EuStyleVatCalculator(0.20m, 0.10m)
        };

        var checkout = new CheckoutService(calculators);

        TaxableItem[] usItems =
        {
            new("book", 100m, TaxTreatment.Standard),
            new("medicine", 25m, TaxTreatment.Exempt)
        };

        TaxableItem[] euItems =
        {
            new("standard-item", 100m, TaxTreatment.Standard),
            new("reduced-item", 50m, TaxTreatment.Reduced),
            new("zero-rated-item", 25m, TaxTreatment.ZeroRated),
            new("exempt-item", 10m, TaxTreatment.Exempt)
        };

        TaxCalculationResult us = checkout.ProcessOrderTaxes("US-SAMPLE", usItems);
        TaxCalculationResult eu = checkout.ProcessOrderTaxes("EU-SAMPLE", euItems);

        Console.WriteLine($"{us.Jurisdiction}: tax {us.TaxAmount}; total {us.Total}");
        Console.WriteLine($"{eu.Jurisdiction}: tax {eu.TaxAmount}; total {eu.Total}");
    }
}
```

`CheckoutService` stores strategies behind the `TaxCalculator` base type. `Calculate` runs the shared validation and result logic, then the virtual dispatch through the abstract `CalculateTaxCore` selects the jurisdiction-specific algorithm. The dictionary avoids a growing jurisdiction `switch`, and duplicate registrations fail during construction.

In a production tax engine, don't infer legal treatment from a single `IsExempt` Boolean or a broad region code. Model applicable tax categories and effective-dated rates explicitly, and define jurisdiction-specific eligibility, exemptions, discounts, returns, currency precision, line-versus-invoice rounding, and audit/reporting behavior. The sample's rate bounds and two-decimal `AwayFromZero` rounding are demonstration policies only.

## 5. Common Mistakes and Design Guidance

- **Treating overloads as runtime polymorphism:** overload resolution is based on the compile-time call and argument types; virtual/interface dispatch selects an implementation at runtime.
- **Assuming vtable details are language guarantees:** method tables and object headers are runtime implementation details. Program against C# contracts, not memory-layout assumptions.
- **Using a base class when an interface is enough:** unrelated providers can implement one interface without sharing a class hierarchy.
- **Assuming polymorphism automatically satisfies OCP:** stable abstractions help, but registration, configuration, and changing policies still need design.
- **Putting all provider selection in a `switch`:** select an implementation at a composition/registration boundary, then call the common contract.
- **Breaking the base contract in an override:** keep the same promised preconditions and postconditions. The sample overrides repeat the base input checks.
- **Calling demonstration code production-ready:** the tax and notification examples omit external API integration, security, idempotency, jurisdiction rules, retries, and operational concerns.

## 6. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Polymorphism** | Working through a shared contract while concrete types provide specialized behavior. | Lets callers use providers or strategies uniformly. |
| **Method overloading** | Same method name with distinct parameter signatures, resolved at compile time. | Offers one API name for related operations or input types. |
| **Operator overloading** | A user-defined implementation of a supported operator for a custom type. | Makes domain-value operations readable when the meaning is natural and unsurprising. |
| **Runtime polymorphism** | Virtual or interface dispatch based on the receiver's runtime type. | Enables pluggable implementations behind base classes or interfaces. |
| **Dynamic dispatch** | Runtime selection of an implementation for a virtual or interface call. | Calls the appropriate implementation through a shared contract. |
| **Virtual method table (vtable)** | A common teaching model for runtime dispatch; exact structures are runtime implementation details. | Helps explain virtual slots, but shouldn't be treated as a public object-layout guarantee. |
| **Open/Closed Principle (OCP)** | A design principle: open for extension, closed for modification. | Polymorphic contracts can help add implementations without changing stable callers. |
| **Strategy pattern** | A design that encapsulates interchangeable algorithms behind a shared contract. | Supports selectable tax, pricing, routing, or validation policies. |

## 7. Official References

- [Polymorphism — C# fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/polymorphism) — base-class polymorphism and virtual dispatch.
- [Interfaces — C# fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces) — interface contracts and interface-based polymorphism.
- [Methods in C#](https://learn.microsoft.com/en-us/dotnet/csharp/methods) — method signatures and overloading.
- [Operator overloading — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading) — user-defined operator declarations.
- [`virtual` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual) — virtual members and runtime dispatch.
- [`override` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override) — override requirements and behavior.
- [`abstract` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract) — abstract classes and members.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — default language versions, including .NET 10 / C# 14.
- [`TimeProvider` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.timeprovider?view=net-10.0) — time abstraction used by the notification sample.
- [`ArgumentException.ThrowIfNullOrWhiteSpace` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception.throwifnullorwhitespace?view=net-10.0) — input guard used by the examples.
- [`Enum.IsDefined<TEnum>` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.enum.isdefined?view=net-10.0) — validates that a tax-treatment value is a defined enum member.
