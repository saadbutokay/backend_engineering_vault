## Prerequisites
Before writing any C# code, you need three things installed on your machine:

1. The .NET SDK (Software Development Kit)
2. A code editor or IDE
3. Familiarity with the dotnet CLI (Command Line Interface)

---
## Step 1: Install the .NET SDK

The .NET SDK contains the compiler (Roslyn), the runtime (CLR), the standard libraries (BCL), and the dotnet CLI tool.

### How to install
1. Go to https://dotnet.microsoft.com/download
2. Download the latest LTS (Long-Term Support) version. As of October 8, 2026, that is .NET 10. Check Microsoft's [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) for current status and end-of-support dates.
3. Run the installer and follow the prompts.
4. After installation, open a terminal (Command Prompt, PowerShell, or Terminal on macOS/Linux) and verify:

```bash
dotnet --version
```

Expected output:

```
10.0.xxx
```

The exact patch number depends on the SDK version you installed; use the latest available .NET 10 SDK patch.

Also check what SDKs and runtimes are installed:

```bash
dotnet --list-sdks
dotnet --list-runtimes
```

### SDK vs Runtime

- The SDK is what you need as a developer. It includes the compiler, the CLI tools, and the runtime.
- The Runtime is what end users need to run your application. It does not include the compiler.
- When you deploy to production, the server only needs the runtime (or you can publish a self-contained application that bundles the runtime inside).

---
## Step 2: Choose Your IDE or Editor

### Option A: Visual Studio (Windows)

Visual Studio is Microsoft's full-featured IDE for Windows and a common choice in enterprise .NET shops. Visual Studio for Mac was retired on August 31, 2024. On macOS, use Visual Studio Code with C# Dev Kit or JetBrains Rider instead. See Microsoft's [Visual Studio for Mac retirement guidance](https://learn.microsoft.com/visualstudio/mac/what-happened-to-vs-for-mac).

- Download from https://visualstudio.microsoft.com/
- The Community edition is free for individuals and small teams.
- During installation, select the "ASP.NET and web development" workload. This installs everything you need for backend development.
- Visual Studio provides: debugging, IntelliSense (code completion), integrated terminal, NuGet package manager UI, database tools, Git integration, and test runners.

### Option B: Visual Studio Code (Windows, macOS, Linux)

VS Code is a lightweight, cross-platform code editor for Windows, macOS, and Linux. It is a good choice if you work across multiple languages, develop on macOS or Linux, or prefer a lightweight editor.

- Download from https://code.visualstudio.com/
- After installation, install these extensions from the Extensions marketplace:
  - C# Dev Kit (by Microsoft) — this is the main extension. It includes the C# language server, solution explorer, and test explorer.
  - C# (by Microsoft) — the base language support extension. It is installed automatically with C# Dev Kit.
- VS Code provides: debugging, IntelliSense, integrated terminal, Git integration. It does not have a visual NuGet manager or database tools built in, but you can use the CLI for those.

### Option C: JetBrains Rider (Windows, macOS, Linux)

Rider is a cross-platform IDE built on the IntelliJ platform. JetBrains offers a free non-commercial license for learning and self-education, hobby development, and other eligible non-commercial use; commercial development requires a paid subscription. A 30-day commercial trial is also available. See [Rider pricing and licensing](https://www.jetbrains.com/rider/buy/). Many .NET developers choose Rider for its speed and refactoring tools.

### Recommendation for this course

On Windows, Visual Studio Community is a free, capable option for eligible use; VS Code with C# Dev Kit is another free cross-platform option. On macOS or Linux, use VS Code with C# Dev Kit or Rider. The dotnet CLI works identically regardless of which editor you choose.

---

## Step 3: The `.net` CLI

The dotnet CLI is the command-line tool that comes with the .NET SDK. It is how you create, build, run, test, and publish .NET projects. Even if you use Visual Studio, understanding the CLI is essential because CI/CD pipelines and Docker containers use it.

### Essential commands

**Create a new project:**

```bash
dotnet new console -n MyFirstApp
```

This creates a new console application in a folder called MyFirstApp.

**Navigate into the project:**

```bash
cd MyFirstApp
```

**Run the project:**

```bash
dotnet run
```

**Build the project (compile without running):**

```bash
dotnet build
```

**Restore NuGet packages:**

```bash
dotnet restore
```

This downloads all external libraries your project depends on. It runs automatically before `dotnet build` and `dotnet run`, but it is useful to run explicitly when you add a new package.

**Add a NuGet package:**

```bash
dotnet add package Newtonsoft.Json
```

**Remove a NuGet package:**

```bash
dotnet remove package Newtonsoft.Json
```

**Run tests:**

```bash
dotnet test
```

**Publish for deployment:**

```bash
dotnet publish -c Release -o ./publish
```

This compiles your application in Release mode and outputs the deployable files to the `./publish` folder.

**Create a solution file:**

```bash
dotnet new sln -n MySolution
```

With the .NET 10 SDK, this creates `MySolution.slnx` by default. To create the older `.sln` format instead, run `dotnet new sln -n MySolution --format sln`.

**Add a project to a solution:**

```bash
dotnet sln add MyFirstApp/MyFirstApp.csproj
```

**List available project templates:**

```bash
dotnet new list
```

### Common project templates

| Template | Command | Use Case |
|----------|---------|----------|
| Console App | `dotnet new console` | Learning, scripts, background workers |
| Web API | `dotnet new webapi` | REST APIs (minimal API by default; add `--use-controllers` for controller-based APIs) |
| Class Library | `dotnet new classlib` | Shared logic, domain models |
| xUnit Tests | `dotnet new xunit` | Unit test projects |
| MVC Web App | `dotnet new mvc` | Full-stack web apps with views |
| gRPC Service | `dotnet new grpc` | High-performance RPC services |
| Worker Service | `dotnet new worker` | Background services and daemons |

---

## Step 4: Understanding Project Structure

When you run `dotnet new console -n MyFirstApp`, the following files are created:

```
MyFirstApp/
    MyFirstApp.csproj
    Program.cs
```

### The .csproj file

This is the project file. It is an XML file that tells the .NET SDK how to build your project. It specifies the target framework, output type, and package dependencies.

A basic .csproj file looks like this:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <RootNamespace>MyFirstApp</RootNamespace>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

Explanation of each element:

- `Sdk="Microsoft.NET.Sdk"`: tells the build system which SDK to use.
- `OutputType`: `Exe` means this project produces an executable. `Library` would produce a .dll.
- `TargetFramework`: specifies which version of .NET to target. `net10.0` means .NET 10.
- `RootNamespace`: the default namespace for code files in this project.
- `ImplicitUsings`: when enabled, common namespaces like `System`, `System.Collections.Generic`, `System.Linq`, and `System.Threading.Tasks` are automatically imported. You do not need to write `using System;` at the top of every file.
- `Nullable`: when enabled, the compiler warns you about potential null reference issues. This is a best practice in modern C#.

### The Program.cs file

This is the entry point of your application. In modern .NET (6+), it uses top-level statements, which means you do not need to explicitly define a class and Main method.

```csharp
// Program.cs — Modern .NET top-level statements

Console.WriteLine("Hello, World!");
```

Under the hood, the compiler wraps this in a class and Main method for you. The older, explicit version looks like this:

```csharp
// Program.cs — Traditional style (pre-.NET 6)

using System;

namespace MyFirstApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

Both are valid. The top-level statement style is the modern default and is what you will see in most new projects.

---

## Step 5: Solution Files

A solution file groups multiple related projects together. With the .NET 10 SDK, `dotnet new sln` creates an `.slnx` file by default; the older `.sln` format is also supported. In a real backend application, you will often have separate projects for your API, business logic, data access layer, and tests.

### Creating a multi-project solution

```bash
# Create the solution file
dotnet new sln -n MyBackendApp

# Create the projects
dotnet new webapi -n MyBackendApp.Api --use-controllers
dotnet new classlib -n MyBackendApp.Core
dotnet new classlib -n MyBackendApp.Infrastructure
dotnet new xunit -n MyBackendApp.Tests

# Add all projects to the solution
dotnet sln add MyBackendApp.Api/MyBackendApp.Api.csproj
dotnet sln add MyBackendApp.Core/MyBackendApp.Core.csproj
dotnet sln add MyBackendApp.Infrastructure/MyBackendApp.Infrastructure.csproj
dotnet sln add MyBackendApp.Tests/MyBackendApp.Tests.csproj

# Add project references (Api depends on Core and Infrastructure)
dotnet add MyBackendApp.Api/MyBackendApp.Api.csproj reference MyBackendApp.Core/MyBackendApp.Core.csproj
dotnet add MyBackendApp.Api/MyBackendApp.Api.csproj reference MyBackendApp.Infrastructure/MyBackendApp.Infrastructure.csproj

# Tests depend on Core
dotnet add MyBackendApp.Tests/MyBackendApp.Tests.csproj reference MyBackendApp.Core/MyBackendApp.Core.csproj
```

The resulting folder structure:

```
MyBackendApp/
    MyBackendApp.slnx
    MyBackendApp.Api/
        MyBackendApp.Api.csproj
        Program.cs
        Controllers/
    MyBackendApp.Core/
        MyBackendApp.Core.csproj
        Class1.cs
    MyBackendApp.Infrastructure/
        MyBackendApp.Infrastructure.csproj
        Class1.cs
    MyBackendApp.Tests/
        MyBackendApp.Tests.csproj
        UnitTest1.cs
```

This is the standard Clean Architecture project structure that you will use throughout this course. We will explore it in depth in Phase 10.

---

## Basic Code Snippet

This demonstrates creating and running a simple console application using the CLI workflow.

```csharp
// Program.cs — A simple console app created with "dotnet new console"
// Run it with: dotnet run

// ImplicitUsings are enabled, so System is already imported.

string developerName = "Backend Student";
int currentPhase = 1;
string currentTopic = "Environment Setup";

Console.WriteLine($"Welcome, {developerName}!");
Console.WriteLine($"You are on Phase {currentPhase}: {currentTopic}");
Console.WriteLine($"Current .NET version: {Environment.Version}");
Console.WriteLine($"Operating System: {Environment.OSVersion}");
Console.WriteLine($"Machine Name: {Environment.MachineName}");
Console.WriteLine($"Number of processors: {Environment.ProcessorCount}");
```

Output when you run `dotnet run`:

```
Welcome, Backend Student!
You are on Phase 1: Environment Setup
Current .NET version: 10.0.x
Operating System: Microsoft Windows 10.0.xxxxx
Machine Name: YOUR-MACHINE
Number of processors: 8
```

---

## Industry-Level Code Snippet

This is a realistic Program.cs from a production ASP.NET Core Web API project. It demonstrates how a real backend application is bootstrapped using the dotnet CLI-generated template with additional production-ready configuration.

```csharp
// Program.cs — Production ASP.NET Core Web API entry point
// Created with: dotnet new webapi -n MyBackendApp.Api --use-controllers
// Add Swagger support: dotnet add package Swashbuckle.AspNetCore
// Run with: dotnet run --urls "https://localhost:5001"

using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;

WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

// --- Service Registration ---

// Add controllers with JSON serialization options
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy =
            System.Text.Json.JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.WriteIndented = false;
    });

// Add API documentation
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new Microsoft.OpenApi.OpenApiInfo
    {
        Title = "MyBackendApp API",
        Version = "v1",
        Description = "Production API for MyBackendApp"
    });
});

// Add CORS policy for frontend communication
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("https://myfrontend.com")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

// Add health checks for load balancer and container orchestration
builder.Services.AddHealthChecks();

// Configure logging
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();

// --- Middleware Pipeline ---

WebApplication app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseCors("AllowFrontend");
app.UseAuthorization();
app.MapControllers();
app.MapHealthChecks("/health");

app.Run();
```

Key observations about this industry-level setup:

- The project uses `dotnet new webapi -n MyBackendApp.Api --use-controllers`, not `dotnet new console`. In .NET 10, the Web API template defaults to minimal APIs unless `--use-controllers` is specified.
- JSON serialization is explicitly configured for consistent API responses.
- This example uses Swashbuckle for the interactive Swagger UI. Add it with `dotnet add package Swashbuckle.AspNetCore`; the .NET 9+ Web API template includes built-in OpenAPI support by default but does not add Swashbuckle automatically.
- CORS is configured to allow specific frontend origins. In production, you never allow all origins.
- Health checks are mapped to `/health`. Kubernetes, Docker, and load balancers use this endpoint to determine if your application is alive.
- Error handling differs between Development and Production. In production, you never expose stack traces to clients.
- HSTS (HTTP Strict Transport Security) is enabled in production to enforce HTTPS.

---

## Quick Reference: dotnet CLI Cheat Sheet

| Task | Command |
|------|---------|
| Check SDK version | `dotnet --version` |
| Create console app | `dotnet new console -n AppName` |
| Create web API | `dotnet new webapi -n ApiName` |
| Create class library | `dotnet new classlib -n LibName` |
| Create test project | `dotnet new xunit -n TestName` |
| Create solution (`.slnx` by default with .NET 10) | `dotnet new sln -n SolutionName` |
| Create legacy `.sln` solution | `dotnet new sln -n SolutionName --format sln` |
| Add project to solution | `dotnet sln add ProjectPath/Project.csproj` |
| Add project reference | `dotnet add ProjectA.csproj reference ProjectB.csproj` |
| Add NuGet package | `dotnet add package PackageName` |
| Restore packages | `dotnet restore` |
| Build project | `dotnet build` |
| Run project | `dotnet run` |
| Run with specific URL | `dotnet run --urls "https://localhost:5001"` |
| Run tests | `dotnet test` |
| Publish for production | `dotnet publish -c Release -o ./publish` |
| Clean build output | `dotnet clean` |
| Watch for changes and rebuild | `dotnet watch run` |

---
