## The Simplest `C#` Program
Every C# application starts with a single entry point. In .NET 10, the default project template uses top-level statements, which means your entire program can be a single line of code.

Create a new project first:

```bash
dotnet new console -n HelloWorldApp --framework net10.0
cd HelloWorldApp
dotnet run
```

The generated `Program.cs` contains:

```csharp
Console.WriteLine("Hello, World!");
```

That is a complete, valid C# program. When you run it, the output is:

```text
Hello, World!
```

This looks deceptively simple. Under the hood, the compiler is doing a lot of work for you. Let us break down what is actually happening.

## Program Structure — What the Compiler Does for You

When you write top-level statements, the C# compiler automatically generates a class and a `Main` method behind the scenes. The single line above is transformed into something equivalent to this:

```csharp
using System;

// Conceptual equivalent. Top-level statements generate a Program type
// in the global namespace; the entry-point method's actual name is
// compiler-specific and isn't directly referenceable from source.
internal class Program
{
    private static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");
    }
}
```

Key points about this generated code:

- The compiler-generated class is named `Program` in the global namespace. It is internal by default, so it is not accessible from other assemblies unless you expose it (for example, by adding a public partial `Program` declaration).
- The generated entry-point method is private static. Its actual metadata name is an implementation detail and cannot be referenced directly from source; `Main` in the conceptual example is illustrative.
- The `args` parameter receives command-line arguments passed to the application.
- In a new .NET console project, implicit usings make `System` types available through SDK-generated global using directives. The compiler does not literally add `using System;` to your source file.

You do not see the synthesized wrapper in your source files, but understanding it helps explain how C# programs are structured. At runtime, the CLR starts from the entry point recorded in the assembly. Top-level statements are compiled into that entry-point method; they are syntactic sugar for the program entry point, not a special runtime feature.

## The Main Method — Traditional vs Modern

### Traditional Style (Explicit Main)

Before C# 9 (released with .NET 5), every C# program required an explicit `Main` method. C# 9 introduced top-level statements, and .NET 6+ console templates made them the default. The explicit style is still valid and common in legacy codebases.

```csharp
using System;

namespace HelloWorldApp
{
    public class Program
    {
        // The CLR calls this method to start the application.
        // It must be static. It can return void or int.
        // It can optionally accept a string array for command-line arguments.
        public static void Main(string[] args)
        {
            Console.WriteLine("Hello from the traditional Main method.");

            // Display command-line arguments if any were passed.
            if (args.Length > 0)
            {
                Console.WriteLine("Arguments received:");
                foreach (string arg in args)
                {
                    Console.WriteLine($"  - {arg}");
                }
            }
        }
    }
}
```

Valid `Main` method signatures:

```csharp
// Returns nothing, no arguments
static void Main()

// Returns nothing, accepts command-line arguments
static void Main(string[] args)

// Returns an exit code (0 = success, non-zero = error), no arguments
static int Main()

// Returns an exit code, accepts command-line arguments
static int Main(string[] args)

// Asynchronous versions (for async/await, covered in Phase 4)
static async Task Main(string[] args)
static async Task<int> Main(string[] args)
```

The exit code is useful in console applications and background services where the operating system or a CI/CD pipeline needs to know whether the program succeeded or failed.

### Modern Style (Top-Level Statements)

Top-level statements were introduced in C# 9 with .NET 5, and .NET 6+ console templates use them by default. In .NET 10, you can omit the class and `Main` method entirely.

```csharp
// Program.cs — Top-level statements in .NET 10

Console.WriteLine("Hello from top-level statements.");

// You can still access command-line arguments via the implicit 'args' variable.
if (args.Length > 0)
{
    Console.WriteLine($"First argument: {args[0]}");
}

// You can still return an exit code.
return 0;
```

Rules for top-level statements:

- Only one file in your entire project can contain top-level statements. By convention, this is `Program.cs`.
- The `args` variable is implicitly available. You do not need to declare it.
- You can use `return` with an integer to set the exit code.
- You can use `await` directly at the top level (the compiler generates an async `Main` for you).
- You can declare classes, records, structs, and other types below the top-level statements in the same file, but they must come after the top-level code.

## Namespaces

Namespaces are a way to organize your code into logical groups and prevent naming collisions. Without namespaces, two classes with the same name in different parts of your application would conflict.

### Traditional Block-Scoped Namespace

```csharp
namespace HelloWorldApp.Services
{
    public class EmailService
    {
        public void SendEmail(string to, string subject, string body)
        {
            Console.WriteLine($"Sending email to {to}: {subject}");
        }
    }
}

namespace HelloWorldApp.Services
{
    public class SmsService
    {
        public void SendSms(string phoneNumber, string message)
        {
            Console.WriteLine($"Sending SMS to {phoneNumber}: {message}");
        }
    }
}
```

The curly braces define the scope of the namespace. Everything inside the braces belongs to that namespace.

### File-Scoped Namespace (C# 10+)

Introduced in C# 10 and the standard in .NET 10 projects, file-scoped namespaces remove one level of indentation by applying the namespace to the entire file.

```csharp
// EmailService.cs
namespace HelloWorldApp.Services;

public class EmailService
{
    public void SendEmail(string to, string subject, string body)
    {
        Console.WriteLine($"Sending email to {to}: {subject}");
    }
}
```

The semicolon after the namespace declaration tells the compiler that everything in this file belongs to the `HelloWorldApp.Services` namespace. This is the preferred style in modern C# because it reduces nesting and improves readability.

Rules for file-scoped namespaces:

- Only one namespace declaration is allowed per file when using file-scoped syntax.
- The namespace declaration must be the first declaration in the file (after using directives).
- You cannot mix file-scoped and block-scoped namespaces in the same file.

### Namespace Hierarchy

Namespaces can be nested to create a hierarchy. The convention is to match your namespace structure to your project and folder structure.

```text
Project:   MyBackendApp.Core
Folder:    Services/Email/
File:      SmtpEmailSender.cs

Namespace: MyBackendApp.Core.Services.Email
```

```csharp
// SmtpEmailSender.cs
namespace MyBackendApp.Core.Services.Email;

public class SmtpEmailSender
{
    public void Send(string to, string subject, string body)
    {
        Console.WriteLine($"SMTP: Sending to {to}");
    }
}
```

## Using Directives

A using directive imports a namespace so you can use its types without writing the fully qualified name every time.

### Without a using directive

```csharp
System.Console.WriteLine("Hello");
System.Collections.Generic.List<string> names = new System.Collections.Generic.List<string>();
System.Text.StringBuilder sb = new System.Text.StringBuilder();
```

### With using directives

```csharp
using System;
using System.Collections.Generic;
using System.Text;

Console.WriteLine("Hello");
List<string> names = new List<string>();
StringBuilder sb = new StringBuilder();
```

### Using aliases

You can create an alias for a namespace or type to avoid ambiguity.

```csharp
using ProjectConfig = MyBackendApp.Core.Configuration.AppSettings;

// Now you can use the alias instead of the full name.
ProjectConfig config = new ProjectConfig();
```

### Static using

You can import static members of a class so you do not need to prefix them with the class name.

```csharp
using static System.Console;
using static System.Math;

// Now you can call these directly without the class prefix.
WriteLine("Hello");
double result = Sqrt(144);
WriteLine($"Square root of 144 is {result}");
```

### Global using

Introduced in C# 10, global usings apply to every file in the project. You only need to declare them once.

```csharp
// GlobalUsings.cs (or any file, typically placed in a dedicated file)
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
```

## Implicit Usings in .NET 10

When `ImplicitUsings` is set to `enable` in your `.csproj` file (which is the default for new projects), the .NET SDK automatically adds global using directives for common namespaces based on your project type.

For a console application, these are implicitly included:

```text
System
System.Collections.Generic
System.IO
System.Linq
System.Net.Http
System.Threading
System.Threading.Tasks
```

For an ASP.NET Core Web API project, additional namespaces are implicitly included:

```text
System.Net.Http.Json
Microsoft.AspNetCore.Builder
Microsoft.AspNetCore.Hosting
Microsoft.AspNetCore.Http
Microsoft.AspNetCore.Routing
Microsoft.Extensions.Configuration
Microsoft.Extensions.DependencyInjection
Microsoft.Extensions.Hosting
Microsoft.Extensions.Logging
```

This means you do not need to write `using System;` or `using System.Linq;` at the top of your files. They are already available.

You can disable implicit usings in the `.csproj` if you prefer explicit control:

```xml
<ImplicitUsings>disable</ImplicitUsings>
```

## The `.csproj` File in .NET 10

A .NET 10 console application project file looks like this:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <RootNamespace>HelloWorldApp</RootNamespace>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

Note the `TargetFramework` is `net10.0`. This tells the compiler and runtime to use .NET 10 features and APIs.

## Basic Code Snippet

This demonstrates all the structural concepts covered in this topic in a single, easy-to-follow program.

```csharp
// Program.cs — Demonstrating program structure in .NET 10

// Implicit usings are enabled, so System, System.Collections.Generic,
// System.Linq, and others are already available.

// Top-level statements begin here.

Console.WriteLine("=== Program Structure Demo ===");
Console.WriteLine($"Runtime: {Environment.Version}");
Console.WriteLine($"Arguments count: {args.Length}");

// You can call methods defined below the top-level statements.
string greeting = BuildGreeting("Backend Student");
Console.WriteLine(greeting);

// You can use types defined below.
var calculator = new Calculator();
int sum = calculator.Add(10, 20);
Console.WriteLine($"10 + 20 = {sum}");

// Return an exit code (0 means success).
return 0;

// --- Methods and types defined below top-level statements ---

static string BuildGreeting(string name)
{
    return $"Hello, {name}! Welcome to .NET 10.";
}

class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }

    public int Subtract(int a, int b)
    {
        return a - b;
    }
}
```

## Industry-Level Code Snippet

This is a realistic `Program.cs` from a .NET 10 ASP.NET Core Web API. It demonstrates how namespaces, using directives, and program structure come together in a production backend application. The example assumes a controller-based project (`dotnet new webapi --use-controllers`), references to the Core and Infrastructure projects, and the required Serilog and Swashbuckle packages. The .NET 9+ Web API template uses built-in OpenAPI support by default and does not add Swashbuckle automatically.

```csharp
// Program.cs — .NET 10 ASP.NET Core Web API
// Most using directives are implicit. Only non-standard ones are explicit.

using MyBackendApp.Api.Middleware;
using MyBackendApp.Core.DependencyInjection;
using MyBackendApp.Infrastructure.DependencyInjection;
using Serilog;

// --- Application Bootstrap ---

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .CreateLogger();

try
{
    Log.Information("Starting application on .NET {Version}", Environment.Version);

    WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

    // Replace default logging with Serilog
    builder.Host.UseSerilog();

    // Register application services using extension methods from other projects.
    // These extension methods live in the Core and Infrastructure namespaces.
    builder.Services.AddCoreServices(builder.Configuration);
    builder.Services.AddInfrastructureServices(builder.Configuration);

    builder.Services.AddControllers();
    builder.Services.AddEndpointsApiExplorer();
    builder.Services.AddSwaggerGen();
    builder.Services.AddHealthChecks();

    WebApplication app = builder.Build();

    // --- Middleware Pipeline ---

    if (app.Environment.IsDevelopment())
    {
        app.UseSwagger();
        app.UseSwaggerUI();
    }

    // Custom global exception handling middleware from the Api project namespace.
    app.UseMiddleware<GlobalExceptionHandlingMiddleware>();

    app.UseHttpsRedirection();
    app.UseAuthorization();
    app.MapControllers();
    app.MapHealthChecks("/health");

    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
    return 1;
}
finally
{
    Log.CloseAndFlush();
}

return 0;
```

### Explanation of the structure

- The explicit using directives at the top import namespaces from the other projects in the solution (Core, Infrastructure, Api). The standard .NET namespaces are implicit.
- The `try`/`catch`/`finally` block wraps the entire application startup. This is a production best practice to ensure that fatal startup errors are logged before the process exits.
- `return 1;` in the `catch` block tells the operating system that the application failed. Container orchestrators like Kubernetes use this exit code to decide whether to restart the container.
- `return 0;` at the end signals a clean shutdown.
- The using aliases and static usings are not used here, but they are common in utility classes and test files.
- The namespace references (`MyBackendApp.Api.Middleware`, `MyBackendApp.Core.DependencyInjection`) follow the convention of matching the folder and project structure.

## Key Terms Summary

| Term | Definition |
|---|---|
| Top-level statements | A modern C# feature that lets you write code without an explicit class and `Main` method. |
| Main method | The entry point of a C# application. The CLR calls this method first. |
| Namespace | A logical container that organizes types and prevents naming collisions. |
| File-scoped namespace | A namespace declaration that applies to the entire file, using a semicolon instead of curly braces. |
| Using directive | An instruction that imports a namespace so its types can be used without full qualification. |
| Global using | A using directive that applies to all files in the project. |
| Implicit usings | A .NET SDK feature that automatically adds common global using directives based on the project type. |
| Exit code | An integer returned by `Main` to indicate success (`0`) or failure (non-zero) to the operating system. |
| `.csproj` | The project file that defines build settings, target framework, and dependencies. |
