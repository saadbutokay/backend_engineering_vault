Examples target **.NET 10** with nullable reference analysis enabled. With the SDK's normal defaults, .NET 10 uses C# 14 unless the project overrides `LangVersion`.

## 1. Conceptual Foundation

### 1.1 What Is an Abstract Class?

An **abstract class** is declared with the `abstract` modifier and can't be instantiated directly with `new`. It is a base type for a family of related classes. An abstract class can combine:

1. **Shared state and implemented behavior**, such as fields, constructors, and ordinary methods.
2. **Required extension points**, such as abstract methods or properties that concrete derived classes must override.

An abstract class doesn't have to declare any abstract members. It can be useful simply because the type itself shouldn't be instantiated. Conversely, a class with abstract members must itself be abstract.

```text
       +-------------------------------------------------------------+
       |                 abstract BaseHandler                       |
       |  - shared logger/state                                     |
       |  - Handle(): fixed workflow                                |
       |  - abstract ValidateRequest(): required hook               |
       |  - abstract ExecuteCore(): required hook                   |
       +-------------------------------------------------------------+
                                      ^
                                      | derives from
               +----------------------+----------------------+
               |                                             |
+------------------------------+             +------------------------------+
| CreateUserHandler            |             | UpdateInventoryHandler      |
| implements required hooks    |             | implements required hooks   |
+------------------------------+             +------------------------------+
```

Use an abstract base class when related types share state, constructors, or protected implementation—not just as a convenient place to put any reusable code. If unrelated types need the same capability, an interface is often a better contract.

## 2. Core Mechanics of `abstract`

### 2.1 Abstract Members

An explicitly `abstract` method, property, indexer, or event declares a contract without an implementation body. In a class hierarchy, an abstract member must be declared inside an abstract class; the declaration ends without a method body. A non-abstract derived class must override every inherited abstract member. An intermediate derived class can remain abstract and leave some members for a later class to implement.

An abstract class member can't be `private`, because a derived class would have no access with which to implement it. Choose an accessibility level that allows the intended derived classes to override it: for example, `protected` for a subclass-only hook, or `public` for part of the public contract. `internal`, `protected internal`, and `private protected` also have assembly-specific access rules. Interface members follow separate rules and are abstract by default unless an implementation is provided.

```csharp
#nullable enable

public abstract class ReportGenerator
{
    public abstract string OutputExtension { get; }
    protected abstract string FormatData(string rawData);
}
```

### 2.2 Constructors in Abstract Classes

An abstract class can declare ordinary constructors. Those constructors run as part of constructing a concrete derived object and can establish base state or validate constructor arguments. The constructor itself is **not abstract**: the derived constructor chooses which base constructor to call with `base(...)`.

A protected constructor is common when only derived types should initialize the base. Abstract classes can also contain implemented methods, properties, fields, and other constructors in addition to abstract members.

### 2.3 Abstract vs. Virtual

| Member kind | Base implementation | Must a concrete derived class override it? |
|---|---|---|
| `abstract` | None. It declares a required contract. | Yes, unless an intermediate class remains abstract and defers the implementation. |
| `virtual` | Present. It supplies default behavior. | No. A derived class may override it. |

An abstract member is inherently an override point; don't combine `abstract` and `virtual` on the same class member. A concrete base class can also contain virtual members, while an abstract class can contain both virtual and abstract members.

## 3. The Template Method Pattern

The **Template Method** pattern uses a base class to define the steps of an algorithm while letting derived classes customize selected steps:

1. A non-virtual base method lays out the shared workflow—for example, parsing, validation, mapping, and persistence.
2. The base method calls abstract or virtual hooks at specific points.
3. Concrete derived classes implement the variable steps without replacing the shared workflow.

A non-virtual template method helps keep common ordering and checks in one place, but it doesn't automatically make a pipeline production-ready. Cancellation, logging, partial failure, transaction boundaries, resource ownership, and retry behavior still need explicit contracts.

## 4. Basic Syntax Example: Report Generators

This complete source file demonstrates a constructor in an abstract class, implemented shared behavior, an abstract method, and an abstract property. `TimeProvider` makes the generated timestamp controllable in tests. The CSV escaping shown is deliberately small and doesn't replace a complete CSV library for complex report formats.

```csharp
#nullable enable
using System;

namespace MyBackendApp.Basics;

public abstract class BaseReportGenerator
{
    public string ReportTitle { get; }
    public DateTimeOffset GeneratedAtUtc { get; }

    protected BaseReportGenerator(string reportTitle, TimeProvider? timeProvider = null)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(reportTitle);

        ReportTitle = reportTitle.Trim();
        GeneratedAtUtc = (timeProvider ?? TimeProvider.System).GetUtcNow();
    }

    // Shared concrete behavior.
    public void PrintHeader()
    {
        Console.WriteLine($"=== Report: {ReportTitle} | Generated: {GeneratedAtUtc:O} ===");
    }

    // Required behavior for each concrete report generator.
    public abstract string FormatData(string rawData);
    public abstract string OutputExtension { get; }
}

public sealed class CsvReportGenerator : BaseReportGenerator
{
    public CsvReportGenerator(string title, TimeProvider? timeProvider = null)
        : base(title, timeProvider)
    {
    }

    public override string OutputExtension => ".csv";

    public override string FormatData(string rawData)
    {
        ArgumentNullException.ThrowIfNull(rawData);

        string header = EscapeCsvField("Header1") + "," + EscapeCsvField("Header2");
        string row = EscapeCsvField(rawData) + "," + EscapeCsvField("Processed");
        return header + Environment.NewLine + row;
    }

    private static string EscapeCsvField(string value) =>
        "\"" + value.Replace("\"", "\"\"") + "\"";
}

public static class ReportDemo
{
    public static void Main()
    {
        var generator = new CsvReportGenerator("Customer export");
        generator.PrintHeader();
        Console.WriteLine(generator.FormatData("Ava, example"));
        Console.WriteLine($"Output extension: {generator.OutputExtension}");
    }
}
```

`BaseReportGenerator` can't be instantiated directly. `CsvReportGenerator` calls the base constructor, inherits `PrintHeader`, and provides implementations for both abstract members. The base constructor establishes the title and timestamp before the derived constructor body runs.

## 5. Applied Backend Example: Data-Import Template Method

A batch importer can use a non-virtual template method to keep a common lifecycle while derived importers provide parsing, validation, mapping, and persistence. The sample injects `ILogger` and propagates failures after logging so a worker or host can apply its retry and failure policy. It doesn't turn every error into a success-shaped result.

> **Scope and package note:** This is an instructional pipeline skeleton, not a production-ready importer. It uses `Microsoft.Extensions.Logging.Abstractions` for `ILogger` (typically available through an ASP.NET Core or Generic Host project); logger registration and provider configuration aren't shown. Its CSV parser expects headerless rows with exactly two unquoted comma-separated fields, and the example holds a batch in memory. Use a standards-compliant CSV library, streaming/batching, a defined transaction and idempotency policy, and application-specific validation for production.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Net.Mail;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

namespace MyBackendApp.Infrastructure.DataProcessing;

public sealed record ImportResult(
    int RecordsRead,
    int RecordsImported,
    int RecordsRejected);

public abstract class BaseDataImporter<TRawRecord, TDomainEntity>
    where TRawRecord : notnull
    where TDomainEntity : notnull
{
    private readonly string _sourceSystem;
    private readonly ILogger _logger;

    public abstract string PipelineName { get; }

    protected BaseDataImporter(string sourceSystem, ILogger logger)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(sourceSystem);
        ArgumentNullException.ThrowIfNull(logger);

        _sourceSystem = sourceSystem.Trim();
        _logger = logger;
    }

    // Template method: the shared import order stays in the base class.
    public async Task<ImportResult> ExecuteImportPipelineAsync(
        Stream dataStream,
        CancellationToken cancellationToken = default)
    {
        ArgumentNullException.ThrowIfNull(dataStream);
        cancellationToken.ThrowIfCancellationRequested();

        _logger.LogInformation(
            "Starting {PipelineName} from {SourceSystem}.",
            PipelineName,
            _sourceSystem);

        try
        {
            IReadOnlyList<TRawRecord> rawRecords =
                await ParseStreamAsync(dataStream, cancellationToken);
            ArgumentNullException.ThrowIfNull(rawRecords);

            var validEntities = new List<TDomainEntity>(rawRecords.Count);
            int rejectedCount = 0;

            for (int index = 0; index < rawRecords.Count; index++)
            {
                cancellationToken.ThrowIfCancellationRequested();

                TRawRecord rawRecord = rawRecords[index];
                if (rawRecord is null)
                {
                    throw new InvalidOperationException("The parser returned a null record.");
                }

                if (!ValidateRawRecord(rawRecord, out string? validationCode))
                {
                    rejectedCount++;
                    _logger.LogWarning(
                        "Rejected record {RecordNumber} in {PipelineName}; validation code {ValidationCode}.",
                        index + 1,
                        PipelineName,
                        validationCode ?? "unspecified");
                    continue;
                }

                TDomainEntity entity = MapToDomain(rawRecord);
                if (entity is null)
                {
                    throw new InvalidOperationException("The mapper returned a null entity.");
                }

                validEntities.Add(entity);
            }

            if (validEntities.Count > 0)
            {
                await PersistBatchAsync(validEntities, cancellationToken);
            }

            var result = new ImportResult(
                RecordsRead: rawRecords.Count,
                RecordsImported: validEntities.Count,
                RecordsRejected: rejectedCount);

            _logger.LogInformation(
                "Completed {PipelineName}: {RecordsRead} read, {RecordsImported} imported, {RecordsRejected} rejected.",
                PipelineName,
                result.RecordsRead,
                result.RecordsImported,
                result.RecordsRejected);

            return result;
        }
        catch (OperationCanceledException)
        {
            // Preserve cancellation so the caller can apply its normal cancellation policy.
            throw;
        }
        catch (Exception exception)
        {
            _logger.LogError(
                exception,
                "{PipelineName} from {SourceSystem} failed.",
                PipelineName,
                _sourceSystem);
            throw;
        }
    }

    protected abstract Task<IReadOnlyList<TRawRecord>> ParseStreamAsync(
        Stream stream,
        CancellationToken cancellationToken);

    // Return a non-sensitive error code; don't log an entire raw record by default.
    protected abstract bool ValidateRawRecord(
        TRawRecord record,
        out string? validationCode);

    protected abstract TDomainEntity MapToDomain(TRawRecord record);

    protected abstract Task PersistBatchAsync(
        IReadOnlyList<TDomainEntity> entities,
        CancellationToken cancellationToken);
}

public sealed record CustomerRecord(Guid Id, string Email, decimal CreditLimit);

public sealed record CsvCustomerLine(string EmailText, string CreditLimitText);

public sealed class CustomerCsvImporter
    : BaseDataImporter<CsvCustomerLine, CustomerRecord>
{
    public override string PipelineName => "Customer-CSV-Ingestion";

    public CustomerCsvImporter(ILogger<CustomerCsvImporter> logger)
        : base(sourceSystem: "LegacyBillingSftp", logger: logger)
    {
    }

    protected override async Task<IReadOnlyList<CsvCustomerLine>> ParseStreamAsync(
        Stream stream,
        CancellationToken cancellationToken)
    {
        // Leave the caller-owned stream open after disposing the reader.
        using var reader = new StreamReader(
            stream,
            Encoding.UTF8,
            detectEncodingFromByteOrderMarks: true,
            bufferSize: 1024,
            leaveOpen: true);

        var records = new List<CsvCustomerLine>();
        int lineNumber = 0;

        while (await reader.ReadLineAsync(cancellationToken) is { } line)
        {
            lineNumber++;
            if (string.IsNullOrWhiteSpace(line))
            {
                continue;
            }

            // Demonstration parser only: expects headerless rows; quoted commas and escaped quotes aren't supported.
            string[] columns = line.Split(',');
            if (columns.Length != 2)
            {
                throw new FormatException(
                    $"Malformed row at line {lineNumber}; expected two unquoted columns.");
            }

            records.Add(new CsvCustomerLine(columns[0].Trim(), columns[1].Trim()));
        }

        return records;
    }

    protected override bool ValidateRawRecord(
        CsvCustomerLine record,
        out string? validationCode)
    {
        if (!MailAddress.TryCreate(record.EmailText, out _))
        {
            validationCode = "invalid_email_syntax";
            return false;
        }

        if (!decimal.TryParse(
                record.CreditLimitText,
                NumberStyles.Number,
                CultureInfo.InvariantCulture,
                out decimal creditLimit) || creditLimit < 0m)
        {
            validationCode = "invalid_credit_limit";
            return false;
        }

        validationCode = null;
        return true;
    }

    protected override CustomerRecord MapToDomain(CsvCustomerLine record)
    {
        decimal creditLimit = decimal.Parse(
            record.CreditLimitText,
            NumberStyles.Number,
            CultureInfo.InvariantCulture);

        // Preserve the supplied address; normalization should follow an explicit domain policy.
        return new CustomerRecord(
            Id: Guid.NewGuid(),
            Email: record.EmailText.Trim(),
            CreditLimit: creditLimit);
    }

    protected override async Task PersistBatchAsync(
        IReadOnlyList<CustomerRecord> entities,
        CancellationToken cancellationToken)
    {
        // Replace this delay with a database batch operation and defined transaction semantics.
        await Task.Delay(TimeSpan.FromMilliseconds(50), cancellationToken);
    }
}
```

`ExecuteImportPipelineAsync` is the template method: it parses, validates, maps, and persists in a fixed order. Concrete importers can change those hooks but don't override the public workflow. Rejected rows are counted separately, and validation logs use error codes rather than raw customer data. Cancellation is rethrown; other exceptions are logged and rethrown so the caller retains control of retries and failure handling. Production logging should also redact sensitive fields that might appear in exception details and avoid duplicate logging at multiple boundaries.

`MailAddress.TryCreate` checks whether an address can be parsed; it doesn't prove that the mailbox exists or that the application's account policy accepts it. Likewise, the sample parser's `Split(',')` isn't a general CSV parser, and the importer buffers the full parsed batch. A production implementation must also decide whether persistence is atomic, how partial commits are retried, and how to prevent duplicate imports.

## 6. Design Guidance and Common Mistakes

- **Calling a constructor abstract:** C# doesn't support abstract constructors. An ordinary constructor in an abstract class initializes shared state as derived objects are created.
- **Assuming every abstract class needs abstract members:** an abstract class can provide only shared concrete behavior and still prevent direct instantiation.
- **Making abstract members private:** derived classes need sufficient access to implement the contract.
- **Making every hook abstract:** use a virtual member with a sensible default when a derived class shouldn't be forced to replace the behavior.
- **Exposing template hooks too broadly:** `protected` hooks are part of the subclass contract. Keep them narrow and make the high-level workflow non-virtual when callers must not reorder it.
- **Swallowing cancellation or failures:** preserve cancellation and make failure/retry semantics explicit. Don't convert a partial persistence failure into a misleading successful result.
- **Calling a demonstration importer production-grade:** real pipelines need structured logging, safe handling of sensitive data, robust parsing, bounded memory, persistence transactions, idempotency, and operational monitoring.

## 7. Key Terms Summary

| Term | Definition | Purpose in backend development |
|---|---|---|
| **Abstract class** | A class that can't be instantiated directly and can combine shared implementation with required extension points. | Provides a reusable foundation for a cohesive family of derived types. |
| **Abstract member** | A method, property, indexer, or event declaration without a class implementation. | Requires a concrete derived class to provide behavior. |
| **Abstract-class constructor** | An ordinary constructor declared in an abstract class and invoked during derived-object construction. | Initializes shared state and validates base invariants. |
| **Virtual member** | A member with a base implementation that derived classes may override. | Supplies default behavior while allowing specialization. |
| **Override** | A derived implementation of an inherited abstract or virtual member. | Fulfills an abstract contract or customizes default behavior. |
| **Template Method pattern** | A base-class method defines workflow order and delegates selected steps to overridable hooks. | Keeps shared processing steps consistent while permitting specialized operations. |
| **`ILogger`** | A .NET logging abstraction used to emit structured application logs. | Separates pipeline logging calls from the logging provider and destination. |

## 8. Official References

- [`abstract` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract) — abstract classes, constructors, and members.
- [Inheritance — C# fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance) — base and derived types, abstract members, and overriding.
- [`virtual` keyword — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual) — optional overriding of implemented members.
- [`override` modifier — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override) — implementing abstract and virtual members.
- [Interfaces — C# fundamentals](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces) — when to use an interface instead of an abstract class.
- [C# language versioning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-versioning) — default language versions, including .NET 10 / C# 14.
- [`TimeProvider` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.timeprovider?view=net-10.0) — injectable time abstraction used by the report example.
- [Logging in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging/overview) — structured logging and `ILogger` usage.
- [`StreamReader.ReadLineAsync(CancellationToken)` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.io.streamreader.readlineasync?view=net-10.0) — asynchronous, cancellable line reading.
- [`MailAddress.TryCreate` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.net.mail.mailaddress.trycreate?view=net-10.0) — parse-checking an address without throwing.
- [`ArgumentException.ThrowIfNullOrWhiteSpace` API for .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception.throwifnullorwhitespace?view=net-10.0) — input guard used by the examples.
