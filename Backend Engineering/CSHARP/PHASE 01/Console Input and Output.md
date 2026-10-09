## Why Console I/O Matters for Backend Engineers

Console input and output may seem irrelevant to backend development because production APIs do not read from a keyboard. However, console I/O is useful to understand for several reasons:

- During development and debugging, `Console.WriteLine` is a quick way to inspect values.
- CLI tools, migration utilities, and some background processes use standard input and output.
- In containers such as Docker, text written to stdout and stderr is commonly captured by the container runtime and forwarded to logging infrastructure. These lines are plain text unless a logging system parses or enriches them.
- Integration tests and migration scripts often report progress through output streams.
- Streams, formatting, and text encoding also apply to reading HTTP request bodies and writing HTTP responses.

For long-running backend applications, prefer the logging abstractions (`ILogger`, Serilog, and similar) over ad hoc `Console.WriteLine` calls. Console logging providers can still send those logs to stdout or stderr.

## Console Output

### `Console.WriteLine`

`Console.WriteLine` writes text followed by a newline to the standard output stream. It is the most commonly used console output method.

```csharp
Console.WriteLine("Hello, World!");
Console.WriteLine(42);
Console.WriteLine(3.14);
Console.WriteLine(true);
Console.WriteLine(DateTime.UtcNow); // Changes each run; formatting depends on culture
```

`Console.WriteLine` formats values as text. For non-null objects, this typically calls `ToString()`; passing `null` writes an empty line.

```csharp
string? name = null;
Console.WriteLine(name); // Writes a blank line; does not throw
```

### `Console.Write`

`Console.Write` is like `Console.WriteLine`, except it does not append a newline. Subsequent output continues on the same line.

```csharp
Console.Write("Hello, ");
Console.Write("World!");
Console.WriteLine(); // Move to the next line manually

// Output: Hello, World!
```

### Formatted Output

Both `WriteLine` and `Write` support composite formatting with numbered placeholders, similar to `string.Format`.

```csharp
string name = "Alice";
int age = 30;
decimal balance = 1250.75m;

// Composite formatting
Console.WriteLine("Customer {0}, age {1}, balance {2:C}", name, age, balance);

// String interpolation
Console.WriteLine($"Customer {name}, age {age}, balance {balance:C}");
```

Both produce equivalent output. Currency and date formatting use the current culture by default, so the exact output can differ by machine. String interpolation is often clearer; composite formatting remains useful when the format string is stored separately, such as in a localization resource.

```csharp
string template = "Order #{0} placed on {1:yyyy-MM-dd} for {2:C}";
DateTime placedAt = new DateTime(2025, 6, 15);
Console.WriteLine(template, 1042, placedAt, 299.99m);
// Example output (en-US): Order #1042 placed on 2025-06-15 for $299.99
```

### Alignment and Padding in Output

Composite-format placeholders can align values within a fixed-width field. A positive width right-aligns; a negative width left-aligns.

```csharp
// Example output assumes en-US currency formatting.
Console.WriteLine("{0,10} | {1,-15} | {2,10:C}", "ID", "Name", "Amount");
Console.WriteLine("{0,10} | {1,-15} | {2,10:C}", 1, "Alice", 150.00m);
Console.WriteLine("{0,10} | {1,-15} | {2,10:C}", 2, "Bob", 2500.50m);
Console.WriteLine("{0,10} | {1,-15} | {2,10:C}", 103, "Charlie", 42.99m);
```

Example output:

```text
        ID | Name            |     Amount
         1 | Alice           |    $150.00
         2 | Bob             |  $2,500.50
       103 | Charlie         |     $42.99
```

This is useful for simple tabular reports in console tools and migration scripts.

## Console Input

### `Console.ReadLine`

`Console.ReadLine` reads a line from standard input. It blocks until a line is available or the input stream reaches its end. It returns the line as a string, or `null` at end-of-stream.

```csharp
Console.Write("Enter your name: ");
string? name = Console.ReadLine();

Console.Write("Enter your age: ");
string? ageInput = Console.ReadLine();

// ReadLine returns text; parse and validate it before using it as a number.
if (int.TryParse(ageInput, out int age))
{
    Console.WriteLine($"Hello, {name}. You are {age} years old.");
}
else
{
    Console.WriteLine("Invalid age entered.");
}
```

Key points about `Console.ReadLine`:

- Its return type is `string?`. It returns `null` when the input stream reaches end-of-file, such as when piped input is exhausted.
- It reads through the line terminator but does not include that terminator in the returned string.
- A successfully read line is text. Parse and validate it before treating it as a number, date, or other type.

### `Console.Read`

`Console.Read` reads one character from the input stream and returns its integer value. It returns `-1` at end-of-stream. For text outside the Basic Multilingual Plane, a single Unicode character can consist of more than one UTF-16 code unit.

```csharp
Console.Write("Press any character: ");
int charCode = Console.Read();

if (charCode != -1)
{
    Console.WriteLine($"You entered: {(char)charCode} (UTF-16 code unit: {charCode})");
}
```

This method is rarely used in backend development; `Console.ReadLine` is usually more practical.

### `Console.ReadKey`

`Console.ReadKey` reads a key press without waiting for Enter. It returns a `ConsoleKeyInfo` struct containing the key, character, and modifier keys (`Shift`, `Ctrl`, and `Alt`).

```csharp
Console.Write("Press any key to continue...");
ConsoleKeyInfo keyInfo = Console.ReadKey();
Console.WriteLine(); // Move to the next line
Console.WriteLine($"You pressed: {keyInfo.Key}");
Console.WriteLine($"Character: {keyInfo.KeyChar}");
Console.WriteLine($"Modifiers: {keyInfo.Modifiers}");
```

You can suppress the key from appearing on screen by passing `true`:

```csharp
// The pressed key will not be displayed (useful for simple password prompts).
ConsoleKeyInfo hiddenKey = Console.ReadKey(intercept: true);
```

`Console.ReadKey` requires an interactive console. It is not supported when input is redirected and can throw `InvalidOperationException`.

```csharp
if (Console.IsInputRedirected)
{
    Console.WriteLine("Interactive input is not available.");
}
else
{
    ConsoleKeyInfo key = Console.ReadKey();
}
```

## Console Properties and Configuration

### Colors

Foreground and background colors can highlight errors, warnings, and success messages in interactive command-line tools. Terminal support varies, so reset colors and avoid assuming a real terminal is attached.

```csharp
Console.ForegroundColor = ConsoleColor.Green;
Console.WriteLine("SUCCESS: Operation completed.");

Console.ForegroundColor = ConsoleColor.Yellow;
Console.WriteLine("WARNING: Disk space is low.");

Console.ForegroundColor = ConsoleColor.Red;
Console.WriteLine("ERROR: Database connection failed.");

Console.ResetColor(); // Restore default colors
Console.WriteLine("INFO: Back to normal.");
```

### Window and Buffer Size

Window and buffer properties are not supported consistently by all terminals and hosting environments. Use them only for interactive tools, and be prepared for platform-specific exceptions.

```csharp
if (!Console.IsOutputRedirected)
{
    Console.WriteLine($"Window width: {Console.WindowWidth}");
    Console.WriteLine($"Window height: {Console.WindowHeight}");
    Console.WriteLine($"Buffer width: {Console.BufferWidth}");
    Console.WriteLine($"Buffer height: {Console.BufferHeight}");
}
```

### Cursor Position

Cursor positioning is intended for interactive terminals and may not work when output is redirected.

```csharp
Console.SetCursorPosition(0, 5); // Move cursor to column 0, row 5
Console.Write("This appears at row 5.");
```

### Detecting Redirection

A program can check whether each standard stream is redirected.

```csharp
Console.WriteLine($"Input redirected: {Console.IsInputRedirected}");
Console.WriteLine($"Output redirected: {Console.IsOutputRedirected}");
Console.WriteLine($"Error redirected: {Console.IsErrorRedirected}");
```

## Standard Output vs. Standard Error

A console application has three standard streams:

- **Standard input (`stdin`):** Where the program reads input. `Console.ReadLine` reads from here.
- **Standard output (`stdout`):** For normal output. `Console.WriteLine` writes here.
- **Standard error (`stderr`):** For diagnostics and error messages. `Console.Error.WriteLine` writes here.

Separating normal output from diagnostic output lets a calling process route the streams differently. Writing to `stderr` does not, by itself, mean the process failed; the process exit code is the usual success/failure signal.

```csharp
// Normal output goes to stdout.
Console.WriteLine("Processing order #1042...");
Console.WriteLine("Order processed successfully.");

// Diagnostics go to stderr.
Console.Error.WriteLine("ERROR: Failed to send confirmation email.");
Console.Error.WriteLine("WARNING: Retry limit approaching.");
```

When running from a terminal, both streams often appear on screen. They can be redirected separately:

```bash
# Redirect stdout to a file; stderr remains on screen.
dotnet run > output.log

# Redirect stdout and stderr to separate files.
dotnet run > output.log 2> errors.log

# Redirect both streams to the same file.
dotnet run > all.log 2>&1
```

## The Console Class and Text Encoding

Console encoding is affected by the runtime, operating system, locale, and terminal. UTF-8 is common, but terminal font and encoding support can still affect how international characters appear. Set the encoding explicitly when your tool requires UTF-8.

```csharp
using System.Text;

Console.OutputEncoding = Encoding.UTF8;
Console.InputEncoding = Encoding.UTF8;

Console.WriteLine("Japanese: こんにちは");
Console.WriteLine("Arabic: مرحبا");
Console.WriteLine("Emoji: 🚀"); // Display depends on terminal support
```

## Console I/O in Backend Applications

Production APIs receive input through HTTP, not from a keyboard. Console streams are still relevant, but prefer a logging framework for application diagnostics so messages can include levels, scopes, and structured fields.

### Scenario 1: Startup Diagnostics

Console output can be useful for quick local diagnostics. In production, use the application's logging framework (including a bootstrap logger if needed) rather than relying on `Console.WriteLine` as the only logging option.

```csharp
Console.WriteLine($"[{DateTime.UtcNow:yyyy-MM-dd HH:mm:ss}] Application starting...");
Console.WriteLine($"Environment: {Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT")}");
```

### Scenario 2: Database Migration Tools

Migration runners and data-seeding utilities often use console output to report progress. In production tools, consider using `ILogger` so the messages have consistent levels and can be routed centrally.

```csharp
Console.WriteLine("Applying migration 20250615_AddOrdersTable...");
// ... apply migration ...
Console.ForegroundColor = ConsoleColor.Green;
Console.WriteLine("Migration applied successfully.");
Console.ResetColor();
```

### Scenario 3: Docker Container Output

Docker commonly captures stdout and stderr from a container. `Console.WriteLine` produces a text log line, not automatically a structured log entry. A logging provider such as the .NET console logger can emit structured fields in a format understood by downstream tools.

```csharp
// A plain-text line written from inside a container.
Console.WriteLine($"[INFO] Request processed in {elapsedMs}ms");
Console.Error.WriteLine($"[ERROR] Unhandled exception: {ex.Message}");
```

### Scenario 4: Background Worker Services

A .NET Worker Service can write to console through its logging provider. For recurring worker diagnostics, use `ILogger` rather than scattering direct `Console.WriteLine` calls.

```csharp
// Inside a BackgroundService; prefer injected ILogger for real application logs.
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    Console.WriteLine("Background worker started.");

    while (!stoppingToken.IsCancellationRequested)
    {
        Console.WriteLine($"[{DateTime.UtcNow:HH:mm:ss}] Processing batch...");
        await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
    }

    Console.WriteLine("Background worker stopped.");
}
```

## Basic Code Snippet

```csharp
// Program.cs — Console Input and Output demo in .NET 10

using System.Text;

// Set UTF-8 encoding for international character support.
Console.OutputEncoding = Encoding.UTF8;

// --- Basic output ---
Console.WriteLine("=== Console Output Demo ===");
Console.WriteLine($"Current time: {DateTime.UtcNow:yyyy-MM-dd HH:mm:ss} UTC");
Console.WriteLine($"Machine: {Environment.MachineName}");
Console.WriteLine($".NET Version: {Environment.Version}");

// --- Formatted output ---
Console.WriteLine("\n=== Formatted Table ===");
Console.WriteLine("{0,-5} | {1,-20} | {2,10} | {3,8}", "ID", "Product", "Price", "Stock");
Console.WriteLine(new string('-', 50));
Console.WriteLine("{0,-5} | {1,-20} | {2,10:C} | {3,8}", 1, "Laptop", 999.99m, 42);
Console.WriteLine("{0,-5} | {1,-20} | {2,10:C} | {3,8}", 2, "Mouse", 29.99m, 150);
Console.WriteLine("{0,-5} | {1,-20} | {2,10:C} | {3,8}", 3, "Monitor", 349.50m, 18);

// --- Colored output ---
Console.WriteLine("\n=== Colored Output ===");
Console.ForegroundColor = ConsoleColor.Green;
Console.WriteLine("  [PASS] All tests passed.");
Console.ForegroundColor = ConsoleColor.Yellow;
Console.WriteLine("  [WARN] Deprecated API endpoint used.");
Console.ForegroundColor = ConsoleColor.Red;
Console.WriteLine("  [FAIL] Database connection timeout.");
Console.ResetColor();

// --- Standard error ---
Console.WriteLine("\n=== Standard Error ===");
Console.WriteLine("This goes to stdout (normal output).");
Console.Error.WriteLine("This goes to stderr (diagnostic output).");

// --- Input (only if interactive) ---
Console.WriteLine("\n=== Input Demo ===");
if (Console.IsInputRedirected)
{
    Console.WriteLine("Input is redirected. Skipping interactive input.");
}
else
{
    Console.Write("Enter your name: ");
    string? name = Console.ReadLine();

    Console.Write("Enter your age: ");
    string? ageInput = Console.ReadLine();

    if (!string.IsNullOrWhiteSpace(name) && int.TryParse(ageInput, out int age))
    {
        Console.WriteLine($"\nWelcome, {name}! You are {age} years old.");
    }
    else
    {
        Console.WriteLine("\nInvalid input provided.");
    }
}

// --- Redirection detection ---
Console.WriteLine("\n=== Stream Status ===");
Console.WriteLine($"stdin redirected: {Console.IsInputRedirected}");
Console.WriteLine($"stdout redirected: {Console.IsOutputRedirected}");
Console.WriteLine($"stderr redirected: {Console.IsErrorRedirected}");
```

## Industry-Level Code Snippet

This illustrative migration-runner pattern shows how a console tool can keep normal output separate from errors and return exit codes for automation. Database methods are placeholders; a production migration tool should also use the application's database, logging, and cancellation policies.

```csharp
// MigrationRunner.cs — Migration runner pattern
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Threading;
using System.Threading.Tasks;

namespace MyBackendApp.Tools.MigrationRunner;

public class MigrationRunner
{
    private readonly string _connectionString;
    private readonly TextWriter _output;
    private readonly TextWriter _error;

    public MigrationRunner(
        string connectionString,
        TextWriter? output = null,
        TextWriter? error = null)
    {
        _connectionString = connectionString;
        _output = output ?? Console.Out;
        _error = error ?? Console.Error;
    }

    public async Task<int> RunAsync(CancellationToken cancellationToken = default)
    {
        var stopwatch = Stopwatch.StartNew();
        WriteHeader();

        try
        {
            WriteStep(1, "Validating database connection...");
            bool isConnected = await ValidateConnectionAsync(cancellationToken);

            if (!isConnected)
            {
                WriteError("Cannot connect to the database. Check your connection string.");
                return 1; // Non-zero exit code signals failure to automation.
            }

            WriteSuccess("Connection established.");

            WriteStep(2, "Checking for pending migrations...");
            List<string> pendingMigrations = await GetPendingMigrationsAsync(cancellationToken);

            if (pendingMigrations.Count == 0)
            {
                WriteInfo("Database is up to date. No migrations to apply.");
                stopwatch.Stop();
                WriteSummary(0, stopwatch.Elapsed);
                return 0;
            }

            WriteInfo($"Found {pendingMigrations.Count} pending migration(s).");
            WriteStep(3, "Applying migrations...");
            int appliedCount = 0;

            foreach (string migration in pendingMigrations)
            {
                cancellationToken.ThrowIfCancellationRequested();
                WriteInfo($"  Applying: {migration}");

                try
                {
                    await ApplyMigrationAsync(migration, cancellationToken);
                    appliedCount++;
                    WriteSuccess($"  Applied: {migration}");
                }
                catch (OperationCanceledException) when (cancellationToken.IsCancellationRequested)
                {
                    WriteWarning("Migration cancelled.");
                    return 2;
                }
                catch (Exception ex)
                {
                    WriteError($"  Failed: {migration}");
                    WriteError($"  Reason: {ex.Message}");
                    return 1;
                }
            }

            WriteStep(4, "Verifying database state...");
            bool isHealthy = await VerifyDatabaseAsync(cancellationToken);

            if (!isHealthy)
            {
                WriteError("Post-migration verification failed.");
                return 1;
            }

            WriteSuccess("Database verification passed.");
            stopwatch.Stop();
            WriteSummary(appliedCount, stopwatch.Elapsed);
            return 0;
        }
        catch (OperationCanceledException) when (cancellationToken.IsCancellationRequested)
        {
            WriteWarning("Migration cancelled.");
            return 2;
        }
        catch (Exception ex)
        {
            WriteError($"Unexpected error: {ex.Message}");
            return 1;
        }
    }

    private void WriteHeader()
    {
        _output.WriteLine("========================================");
        _output.WriteLine("  MyBackendApp Database Migration Runner");
        _output.WriteLine($"  Timestamp: {DateTime.UtcNow:yyyy-MM-dd HH:mm:ss} UTC");
        _output.WriteLine($"  Machine: {Environment.MachineName}");
        _output.WriteLine($"  Runtime: .NET {Environment.Version}");
        _output.WriteLine("========================================");
        _output.WriteLine();
    }

    private void WriteStep(int number, string description)
    {
        _output.WriteLine();
        _output.WriteLine($"--- Step {number}: {description} ---");
    }

    private void WriteInfo(string message)
    {
        _output.WriteLine($"[INFO]  {message}");
    }

    private void WriteSuccess(string message)
    {
        bool useColor = ReferenceEquals(_output, Console.Out) && !Console.IsOutputRedirected;
        if (useColor) Console.ForegroundColor = ConsoleColor.Green;
        _output.WriteLine($"[OK]    {message}");
        if (useColor) Console.ResetColor();
    }

    private void WriteWarning(string message)
    {
        bool useColor = ReferenceEquals(_output, Console.Out) && !Console.IsOutputRedirected;
        if (useColor) Console.ForegroundColor = ConsoleColor.Yellow;
        _output.WriteLine($"[WARN]  {message}");
        if (useColor) Console.ResetColor();
    }

    private void WriteError(string message)
    {
        bool useColor = ReferenceEquals(_error, Console.Error) && !Console.IsErrorRedirected;
        if (useColor) Console.ForegroundColor = ConsoleColor.Red;
        _error.WriteLine($"[ERROR] {message}");
        if (useColor) Console.ResetColor();
    }

    private void WriteSummary(int appliedCount, TimeSpan elapsed)
    {
        _output.WriteLine();
        _output.WriteLine("========================================");
        _output.WriteLine($"  Migrations applied: {appliedCount}");
        _output.WriteLine($"  Total duration: {elapsed.TotalSeconds:F2}s");
        _output.WriteLine("  Status: SUCCESS");
        _output.WriteLine("========================================");
    }

    // Placeholder methods: replace with real database logic.
    private Task<bool> ValidateConnectionAsync(CancellationToken ct)
    {
        return Task.FromResult(true);
    }

    private Task<List<string>> GetPendingMigrationsAsync(CancellationToken ct)
    {
        return Task.FromResult(new List<string>
        {
            "20250615_InitialCreate",
            "20250616_AddOrdersTable",
            "20250617_AddIndexOnCustomerEmail"
        });
    }

    private Task ApplyMigrationAsync(string migration, CancellationToken ct)
    {
        return Task.Delay(100, ct);
    }

    private Task<bool> VerifyDatabaseAsync(CancellationToken ct)
    {
        return Task.FromResult(true);
    }
}

// Program.cs — example call site
// string connectionString = "...";
// var runner = new MigrationRunner(connectionString);
// int exitCode = await runner.RunAsync();
// return exitCode;
```

Key observations from this example:

- The class writes to `TextWriter` fields rather than directly calling `Console.WriteLine`. The constructor accepts writers, so tests can inject `StringWriter` instances and capture output.
- Color is used only when the writer is the actual console stream and that stream is not redirected. Redirection checks are useful, but they do not guarantee every terminal supports color.
- Normal messages go to stdout and errors go to stderr. These streams can be routed separately; the process exit code remains the main success/failure signal.
- The runner returns `0` for success, `1` for failure, and `2` for cancellation. CI/CD tools use exit codes to decide whether a step succeeded.
- A `CancellationToken` is passed through async operations, and cancellation is handled separately from other failures.
- `Stopwatch` measures elapsed time for the summary.
- Prefixes such as `[INFO]` and `[ERROR]` make plain-text lines easier to scan. A logging framework is preferable when structured, queryable fields are needed.

## Key Terms Summary

| Term | Definition |
|---|---|
| `Console.WriteLine` | Writes text followed by a newline to stdout. |
| `Console.Write` | Writes text to stdout without appending a newline. |
| `Console.ReadLine` | Reads a line from stdin; returns `string?`, with `null` at end-of-stream. |
| `Console.ReadKey` | Reads a key press from an interactive console and returns `ConsoleKeyInfo`. |
| `stdout` | Standard output stream, normally used for regular program output. |
| `stderr` | Standard error stream, normally used for diagnostics and errors. |
| `stdin` | Standard input stream, used for keyboard or piped input. |
| Composite formatting | The `{0}`, `{1}` placeholder syntax used by `WriteLine` and `string.Format`. |
| Redirection | Sending console streams to files or other processes instead of directly to a terminal. |
| Encoding | The character encoding used to convert text to and from bytes; UTF-8 is commonly used. |
| Exit code | An integer returned to the operating system; conventionally `0` means success and non-zero means failure. |
