Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 What Is an Object Initializer?

An **object initializer** assigns accessible fields, properties, or indexers as part of an object-creation expression. The selected constructor still runs first; the listed assignments then initialize the new object in order. The initializer doesn't replace the constructor or make its validation unnecessary.

```csharp
#nullable enable

public sealed class UserAccount
{
    public string Username { get; init; } = string.Empty;
    public bool IsActive { get; init; }
}

public static class ObjectInitializerExample
{
    public static UserAccount Create() => new UserAccount
    {
        Username = "jdoe",
        IsActive = true
    };
}
```

Use an initializer when it makes setup more readable. If an invariant must be enforced atomically by construction, prefer a constructor or factory that receives and validates the required values.

### 1.2 What Is an Anonymous Type?

An **anonymous type** is a compiler-generated, unnamed reference type created with `new { ... }`. Its properties are public and read-only, and the generated type supplies value-based `Equals`, `GetHashCode`, and `ToString` implementations.

```csharp
var employeeSnapshot = new { Name = "Alice", Department = "Engineering" };
```

The compiler creates an `internal sealed` class, but its generated name can't be written in source code. Use `var` when you want to retain the inferred type and access its properties. You can't declare that anonymous type by name in a normal method, property, field, or parameter signature. It can flow through generic type inference or be converted to `object` or `dynamic`, but those aren't substitutes for a stable named public contract.

Anonymous types with the same property names, types, and order in the same assembly use the same generated type. Their `Equals` method compares property values using each property's own equality behavior. Anonymous types don't provide a generated value-based `==` operator, so `==` still tests reference identity; use `.Equals()` for their value comparison.

## 2. Object Initializers — Deeper Mechanics

### 2.1 Writable Members and Constructors

Object initializers can set accessible writable fields and properties with accessible `set` or `init` accessors. They can be combined with a parameterless or parameterized constructor. C# also supports initializing indexers. A `required` field or property must be assigned by the caller in the initializer (or by a constructor marked with the required-members contract).

```csharp
#nullable enable

public sealed class ShippingAddress
{
    public string Street { get; set; } = string.Empty;
    public string City { get; set; } = string.Empty;
    public string PostalCode { get; set; } = string.Empty;
}

public static class ShippingAddressExample
{
    public static ShippingAddress Create() => new ShippingAddress
    {
        Street = "123 Main St",
        City = "Springfield",
        PostalCode = "90210"
    };
}
```

When post-construction assignments are part of an invariant, don't assume the constructor validated them: the constructor runs before the initializer assignments. Use a validating constructor/factory, or validate at an explicit boundary.

### 2.2 Nested Object Initializers

You can build an object graph by using an initializer inside another initializer:

```csharp
#nullable enable

public sealed class Address
{
    public string Street { get; set; } = string.Empty;
    public string City { get; set; } = string.Empty;
}

public sealed class Customer
{
    public string Name { get; set; } = string.Empty;
    public Address Address { get; set; } = new();
}

public static class NestedInitializerExample
{
    public static Customer Create() => new Customer
    {
        Name = "Jane Smith",
        Address = new Address
        {
            Street = "456 Oak Ave",
            City = "Metropolis"
        }
    };
}
```

If `Address` already refers to an object, `Address = { City = "Metropolis" }` can initialize that existing nested object without replacing it. That form requires the nested object to have been created already.

### 2.3 Collection Initializers and Collection Expressions

A **collection initializer** invokes the collection's `Add` method for each listed element. A **collection expression** (`[...]`) is a separate, target-typed C# 12 feature that can create many supported collection types; it isn't valid for every arbitrary collection-like type.

```csharp
using System.Collections.Generic;

public static class CollectionSyntaxExample
{
    public static void Run()
    {
        var skills = new List<string> { "C#", "SQL", "Docker" }; // Collection initializer
        List<string> modernSkills = ["C#", "SQL", "Docker"];     // Collection expression (C# 12+)
    }
}
```

Use collection expressions when the target type is known and supports the conversion. They create a collection containing the listed values; they aren't a lazy query syntax.

## 3. Anonymous Types — Deeper Mechanics

### 3.1 Characteristics and Property Inference

1. **Read-only properties:** generated properties can't be reassigned after creation. This is shallow immutability: a property can still refer to a mutable object.
2. **Inferred names:** `new { name, age }` produces properties named `name` and `age`. A member access such as `new { employee.FullName }` infers `FullName`.
3. **Shape matters:** property names, types, and order determine the generated type within an assembly.
4. **Value equality uses member equality:** two instances with the same shape and equal property values compare equal through `.Equals()`. Nested arrays or collections may still use reference equality unless their own equality implementation compares elements.
5. **Local-friendly, not a public API shape:** the type name is inaccessible in source. Use a named class or record when the shape must appear in an API or reusable method signature.

```csharp
string name = "Carlos";
int age = 34;

var inferred = new { name, age };
// Same shape as: new { name = name, age = age }
```

### 3.2 Common Uses

- **LINQ projections:** shape only the fields needed for a local query result.
- **Temporary groupings:** carry a few related values through nearby code.
- **Structured logging:** create a temporary payload when the logging provider supports it. Destructuring behavior is provider-specific; avoid logging sensitive fields accidentally.

## 4. Basic Syntax Example

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public sealed class Product
{
    public string Sku { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
}

public static class InitializerDemo
{
    public static void Run()
    {
        var product = new Product
        {
            Sku = "ELEC-001",
            Name = "USB-C Charger",
            Price = 19.99m,
            StockQuantity = 150
        };

        // Temporary, inferred projection; no named ProductSummary type is needed here.
        var productSummary = new
        {
            product.Sku,
            product.Name,
            IsLowStock = product.StockQuantity < 10
        };

        Console.WriteLine(productSummary);
        Console.WriteLine(productSummary.Equals(new
        {
            product.Sku,
            product.Name,
            IsLowStock = product.StockQuantity < 10
        })); // True: same generated shape and member values
    }
}
```

The anonymous type's generated `ToString()` is useful for display, not a stable serialization format. Its `Equals()` example compares member values; `productSummary == anotherSummary` would still use reference identity.

## 5. Backend Example: Test Data and EF Core Projection

Object initializers are handy for constructing test data and mutable EF Core entities. In a provider-backed LINQ query, `Select` can project the columns the query needs into an anonymous type. EF Core translates supported projections to SQL; the projection—not the fact that the type is anonymous—is what limits the selected columns. A named DTO can also be projected directly.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace MyBackendApp.Infrastructure.Reporting;

public class EmployeeRecord
{
    public Guid Id { get; set; }
    public string FullName { get; set; } = string.Empty;
    public string Department { get; set; } = string.Empty;
    public decimal AnnualSalary { get; set; }
    public DateTime HireDateUtc { get; set; }
}

public sealed class ReportingDbContext : DbContext
{
    public ReportingDbContext(DbContextOptions<ReportingDbContext> options)
        : base(options)
    {
    }

    public DbSet<EmployeeRecord> Employees => Set<EmployeeRecord>();
}

// Named type for a service/API boundary; the anonymous type stays local to the query.
public sealed record EmployeeSummaryDto(Guid Id, string FullName, string Department);

public sealed class EmployeeReportingService
{
    private readonly ReportingDbContext _db;

    public EmployeeReportingService(ReportingDbContext db)
    {
        ArgumentNullException.ThrowIfNull(db);
        _db = db;
    }

    // Stable identifiers make this sample data repeatable in tests or seed setup.
    public static List<EmployeeRecord> CreateSampleEmployees() => new()
    {
        new EmployeeRecord
        {
            Id = new Guid("11111111-1111-1111-1111-111111111111"),
            FullName = "Maria Chen",
            Department = "Engineering",
            AnnualSalary = 112000m,
            HireDateUtc = new DateTime(2021, 3, 15, 0, 0, 0, DateTimeKind.Utc)
        },
        new EmployeeRecord
        {
            Id = new Guid("22222222-2222-2222-2222-222222222222"),
            FullName = "David Okafor",
            Department = "Finance",
            AnnualSalary = 98000m,
            HireDateUtc = new DateTime(2019, 7, 1, 0, 0, 0, DateTimeKind.Utc)
        }
    };

    public async Task<IReadOnlyList<EmployeeSummaryDto>> GetDepartmentEmployeesAsync(
        string department,
        CancellationToken cancellationToken = default)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(department);

        var projectedRows = await _db.Employees
            .AsNoTracking()
            .Where(employee => employee.Department == department)
            .OrderBy(employee => employee.FullName)
            .Select(employee => new
            {
                employee.Id,
                employee.FullName,
                employee.Department
            })
            .ToListAsync(cancellationToken);

        return projectedRows
            .Select(row => new EmployeeSummaryDto(row.Id, row.FullName, row.Department))
            .ToList();
    }
}
```

The LINQ query runs against the EF Core provider until `ToListAsync`; it selects `Id`, `FullName`, and `Department`, not the full employee entity with salary and hire-date columns. After materialization, the service maps the anonymous rows to a named DTO for its return contract. Check provider translation for more complex query shapes, and avoid assuming an in-memory LINQ query reduces database I/O.

For structured logs, prefer explicit message-template fields or a named logging DTO. Some providers support destructuring anonymous objects, but the syntax and behavior vary; don't log salary or other sensitive data without an approved reason.

## 6. Design Guidance and Common Mistakes

- **Thinking an object initializer replaces a constructor:** the constructor still runs first; the initializer then assigns members.
- **Relying on a constructor to validate initializer-only values:** those values aren't available to the constructor. Use constructor parameters, a factory, required members, or an explicit validation step for important invariants.
- **Confusing nested replacement with nested mutation:** `Address = new Address { ... }` replaces the reference; `Address = { ... }` modifies an already initialized nested object.
- **Treating collection expressions as collection initializers:** `[...]` is target-typed collection-expression syntax; `{ ... }` after `new List<T>()` uses collection-initializer/Add semantics.
- **Assuming every collection accepts `[...]`:** collection expressions require a supported target conversion or collection builder.
- **Treating anonymous types as named API DTOs:** their generated type can't be declared in a public signature. Use a named class or record for stable contracts.
- **Comparing anonymous types with `==`:** generated value equality is available through `.Equals()`, but `==` remains reference equality.
- **Assuming equality recursively compares nested values:** each property uses its own equality implementation; a list property may compare by reference rather than element-by-element.
- **Claiming anonymous-type LINQ projections always reduce SQL:** only a translatable provider-backed query can reduce database columns, and the selected projection determines what is fetched.
- **Returning `IEnumerable<object>` just to hide an anonymous type:** callers lose compile-time access to the shape. Materialize and map to a named DTO when data must cross a method or API boundary.
- **Logging anonymous objects without checking the provider:** destructuring and formatting differ between logging libraries, and generated `ToString()` can expose values.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Object initializer** | Syntax that assigns accessible members after an object is constructed. | Keeps setup readable without adding a constructor overload for every optional combination. |
| **Nested object initializer** | An initializer that constructs or populates a member object inside another initializer. | Builds object graphs in a declarative form. |
| **Collection initializer** | Syntax that adds elements to a collection through its `Add` method. | Makes small, known test or seed collections concise. |
| **Collection expression (`[...]`)** | Target-typed C# syntax for creating many supported collection types. | Provides concise collection creation where the target type supports it. |
| **Anonymous type** | A compiler-generated internal class with read-only properties and no source-accessible type name. | Shapes temporary values, especially local LINQ projections. |
| **Property inference** | Inferring an anonymous property name from a variable or member-access expression. | Reduces repetitive naming in projections. |
| **Anonymous-type value equality** | `.Equals()` compares values for matching generated shapes using each property's equality behavior. | Helps compare temporary projections; `==` still checks reference identity. |
| **Projection** | Selecting and shaping only the result members needed from a sequence or query. | Can reduce transferred columns in a translated EF Core query. |

## 8. Official References

- [Object and collection initializers — C# guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers) — object, nested, and collection initializers.
- [Anonymous types — C# guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types) — generated type characteristics, inferred properties, and equality.
- [Collection expressions — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/collection-expressions) — target-typed `[...]` syntax and supported collection targets.
- [Efficient querying — EF Core](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying#project-only-properties-you-need) — projecting only required columns.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — .NET 10 defaults to C# 14.
- [C# version history](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history) — collection expressions were introduced in C# 12.
