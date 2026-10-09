Examples target .NET 10. The short code blocks are focused examples and may refer to types introduced earlier; don't concatenate every block into one source file. The **Basic Code Snippet** is a complete standalone .NET 10 console program.

## What Is an Enum?

An **enum** (enumeration) is a value type whose named members are backed by an integral numeric type. It's useful for a finite set of related choices, such as an order status, a log level, or an action in an API.

Enums make code more readable than unexplained numbers and prevent accidental assignments from unrelated types. They don't, by themselves, guarantee that a runtime enum value matches a declared member: casts, deserialization, database reads, and the default value can produce unnamed values. Validate data at system boundaries when that matters.

Use an enum for a closed set of choices. If users or administrators can add new values without a code deployment, use a database lookup or another extensible representation instead.

## The Problem Enums Solve

Strings and integers can make code harder to read and easier to mistype.

```csharp
string status = "Shipped";

if (status == "shipped") // String comparisons are case-sensitive by default.
{
    // Process shipment.
}

int numericStatus = 2; // The meaning of 2 isn't visible here.
if (numericStatus == 2)
{
    // Process shipment.
}
```

An enum makes the value's purpose explicit:

```csharp
OrderStatus status = OrderStatus.Shipped;

if (status == OrderStatus.Shipped)
{
    ProcessShipment();
}

// status = "Delivered"; // Compile error: a string isn't an OrderStatus.
// status = 99;          // Compile error: a nonzero int isn't implicitly an OrderStatus.
```

C# does allow an implicit conversion from the constant value `0` to an enum, and an explicit cast can convert any underlying numeric value. Neither case proves that a named member exists; see validation below. For example, if no member has value zero, assigning `0` still compiles and produces an unnamed value.

## Basic Enum Declaration

### Simple Enum

```csharp
enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}
```

By default, the first member has value `0`, and following members increase by `1` in declaration order.

```csharp
using System;

Console.WriteLine((int)OrderStatus.Pending);    // 0
Console.WriteLine((int)OrderStatus.Processing); // 1
Console.WriteLine((int)OrderStatus.Shipped);    // 2
Console.WriteLine((int)OrderStatus.Delivered);  // 3
Console.WriteLine((int)OrderStatus.Cancelled);  // 4
```

An enum's default value is always `(E)0`, even if no member is explicitly assigned `0`. It's usually helpful to define a meaningful zero member, especially for flags enums.

### Explicit Values

You can assign explicit values when an enum maps to existing database values, a protocol, or another externally defined numeric code.

```csharp
enum ApiStatusCode
{
    Ok = 200,
    Created = 201,
    NoContent = 204,
    BadRequest = 400,
    Unauthorized = 401,
    Forbidden = 403,
    NotFound = 404,
    InternalServerError = 500
}
```

```csharp
using System;

Console.WriteLine((int)ApiStatusCode.Ok);                  // 200
Console.WriteLine((int)ApiStatusCode.NotFound);             // 404
Console.WriteLine((int)ApiStatusCode.InternalServerError);  // 500
```

### Mixed Explicit and Implicit Values

After an explicitly assigned value, an unassigned member is one greater than the preceding member.

```csharp
enum Priority
{
    None = 0,
    Low = 1,
    Medium = 5,
    High,          // 6
    Critical,      // 7
    Emergency = 10
}
```

```csharp
using System;

Console.WriteLine((int)Priority.Low);       // 1
Console.WriteLine((int)Priority.Medium);    // 5
Console.WriteLine((int)Priority.High);      // 6
Console.WriteLine((int)Priority.Critical);  // 7
Console.WriteLine((int)Priority.Emergency); // 10
```

### Persisted Numeric Values

If a database stores the numeric value, inserting a new member in the middle of an implicitly numbered enum changes the values assigned to later members. Existing rows can then be interpreted incorrectly. The before/after types below are separate illustrations; they wouldn't be declared together under the same name in one source file.

```csharp
enum OrderStatusV1
{
    Pending,     // 0
    Processing,  // 1
    Shipped,     // 2
    Delivered    // 3
}

enum OrderStatusV2
{
    Pending,         // 0
    AwaitingPayment, // 1: inserted member
    Processing,      // 2: was 1 in V1
    Shipped,         // 3: was 2 in V1
    Delivered        // 4: was 3 in V1
}
```

For a numeric database or protocol contract, assign stable values and never renumber or reuse values that have already been stored or released:

```csharp
enum OrderStatusCode
{
    Pending = 0,
    Processing = 10,
    Shipped = 20,
    Delivered = 30,
    AwaitingPayment = 40 // Added without changing existing codes.
}
```

Explicit values are important when the numbers are persisted or externally meaningful. They aren't necessary for every private enum. Gaps are optional; the important rule is to keep existing values stable.

## Underlying Types

The default underlying type is `int` (`System.Int32`). C# also permits `sbyte`, `byte`, `short`, `ushort`, `uint`, `long`, and `ulong`.

```csharp
using System;

// Default underlying type: int.
enum Color
{
    Red,
    Green,
    Blue
}

// A byte-backed enum has values from 0 through 255.
enum LogLevel : byte
{
    Trace = 0,
    Debug = 1,
    Information = 2,
    Warning = 3,
    Error = 4,
    Critical = 5
}

// A 64-bit flags enum can use higher bit positions.
[Flags]
enum FilePermissionMask : long
{
    None = 0,
    Read = 1L << 0,
    Write = 1L << 1,
    Archive = 1L << 40
}
```

Choose an underlying type to match an interop, storage, wire-format, or compact-array requirement. Don't switch from the default `int` solely because an enum has fewer than 256 members; actual memory savings depend on where and how values are stored, and the chosen type becomes part of the enum's contract.

## Casting Between an Enum and Its Underlying Type

An explicit cast converts between an enum and its underlying integral type. Casting does not validate that the value names a declared member. The constant-zero conversion is an exception to the usual requirement for explicit numeric casts:

```csharp
using System;

NonZeroStatus zero = 0; // Legal implicit conversion from constant zero; no member has this value.
Console.WriteLine(Enum.IsDefined(typeof(NonZeroStatus), zero)); // False

enum NonZeroStatus
{
    Pending = 1,
    Completed = 2
}
```

```csharp
using System;

OrderStatus status = OrderStatus.Shipped;

int numericValue = (int)status;
Console.WriteLine(numericValue); // 2

int databaseValue = 3;
OrderStatus fromDatabase = (OrderStatus)databaseValue;
Console.WriteLine(fromDatabase); // Delivered

int invalidValue = 99;
OrderStatus invalid = (OrderStatus)invalidValue; // Compiles; no such member is defined.
Console.WriteLine(invalid); // 99
Console.WriteLine(Enum.IsDefined(typeof(OrderStatus), invalid)); // False
```

For a non-flags enum, validate external numeric input before treating the cast result as a valid member:

```csharp
using System;

int databaseValue = 99;
if (Enum.IsDefined(typeof(OrderStatus), databaseValue))
{
    OrderStatus status = (OrderStatus)databaseValue;
    Console.WriteLine($"Valid status: {status}");
}
else
{
    Console.WriteLine("Unknown status code.");
}
```

For a flags enum, `Enum.IsDefined` only returns `true` for a value that is explicitly declared—including an explicitly named composite. It returns `false` for many valid combinations of atomic flags. Validate a flags value against the mask of known bits instead.

## The `Flags` Attribute

The `[Flags]` attribute signals that an enum is intended to represent a bit field. It affects formatting and communicates intent; it does **not** enforce powers of two or validate combinations.

Use `0` for “no flags” and a distinct power of two for each **atomic** flag. Composite members may combine atomic flags.

```csharp
using System;

[Flags]
enum Permission
{
    None = 0,
    Read = 1,          // 0000 0001
    Write = 2,         // 0000 0010
    Delete = 4,        // 0000 0100
    Execute = 8,       // 0000 1000
    Admin = 16,        // 0001 0000

    ReadWrite = Read | Write,
    All = Read | Write | Delete | Execute | Admin
}
```

### Combining and Checking Flags

Use bitwise OR (`|`) to combine flags. `HasFlag` and bitwise AND (`&`) can check whether all bits in a requested mask are present.

```csharp
using System;

Permission userPermissions = Permission.Read | Permission.Write;

Console.WriteLine(userPermissions);          // ReadWrite, because that composite is named
Console.WriteLine((int)userPermissions);     // 3
Console.WriteLine(userPermissions.HasFlag(Permission.Read)); // True
Console.WriteLine(userPermissions.HasFlag(Permission.Execute)); // False

bool canWrite = (userPermissions & Permission.Write) == Permission.Write;
Console.WriteLine($"Can write: {canWrite}"); // True
```

`HasFlag(mask)` returns `true` when **all** bits in `mask` are present. A zero mask is a special case: `value.HasFlag(Permission.None)` is always `true`. Don't use `None` as a required permission. Bitwise checks can be explicit in hot code, but don't assume a performance difference without measuring.

### Adding, Removing, and Toggling Flags

```csharp
using System;

Permission permissions = Permission.Read;

permissions |= Permission.Write; // Add.
Console.WriteLine(permissions);   // ReadWrite

permissions &= ~Permission.Read; // Remove.
Console.WriteLine(permissions);   // Write

permissions ^= Permission.Write; // Toggle.
Console.WriteLine(permissions);   // None
```

### Validating a Flags Value

A flags value can contain any combination of bits, including bits not assigned to a member. Check that no unknown bits are set:

```csharp
Permission permissions = Permission.Read | Permission.Write;
const Permission KnownPermissions =
    Permission.Read | Permission.Write | Permission.Delete | Permission.Execute | Permission.Admin;

bool containsOnlyKnownBits = (permissions & ~KnownPermissions) == Permission.None;
Console.WriteLine(containsOnlyKnownBits); // True
```

If you add an atomic flag, update the known-bit mask. Named composites such as `ReadWrite` and `All` are conveniences; they don't add new atomic bits.

## Enum Methods

The `System.Enum` APIs provide methods for converting, parsing, listing, and validating enum values.

### `ToString`

`ToString` returns a member name for a named value. For a flags enum, it can return comma-separated names for a combination of recognized flags.

```csharp
using System;

OrderStatus status = OrderStatus.Shipped;
Console.WriteLine(status.ToString()); // Shipped
Console.WriteLine($"{status}");       // Interpolation also formats the enum.

Permission permissions = Permission.Read | Permission.Write;
Console.WriteLine(permissions); // ReadWrite (the named composite)
```

### `Parse` and `TryParse`

`Enum.Parse` throws when parsing fails. `Enum.TryParse` returns a success value instead, but it accepts both names and numeric strings. A successful parse alone doesn't establish that the resulting value is a declared member.

```csharp
using System;

OrderStatus status = Enum.Parse<OrderStatus>("Shipped");
Console.WriteLine(status); // Shipped

OrderStatus caseInsensitive = Enum.Parse<OrderStatus>("shipped", ignoreCase: true);
Console.WriteLine(caseInsensitive); // Shipped

if (Enum.TryParse<OrderStatus>("Delivered", ignoreCase: true, out OrderStatus parsed)
    && Enum.IsDefined(typeof(OrderStatus), parsed))
{
    Console.WriteLine($"Parsed defined status: {parsed}");
}
else
{
    Console.WriteLine("Invalid status.");
}

// A numeric string can parse successfully but still be undefined.
if (Enum.TryParse<OrderStatus>("99", out OrderStatus numeric)
    && !Enum.IsDefined(typeof(OrderStatus), numeric))
{
    Console.WriteLine("Parsed a number, but it isn't a declared status.");
}
```

If an API contract accepts **names only**, also reject numeric strings explicitly. For a flags enum, validate the known-bit mask rather than relying on `Enum.IsDefined` for arbitrary combinations.

### `GetValues` and `GetNames`

`Enum.GetValues<TEnum>()` returns an array of the declared values; `Enum.GetNames<TEnum>()` returns their names. The results include composite members and duplicate values when aliases are declared. Values are sorted by their underlying unsigned numeric value, not necessarily by declaration order.

```csharp
using System;

OrderStatus[] allStatuses = Enum.GetValues<OrderStatus>();
foreach (OrderStatus status in allStatuses)
{
    Console.WriteLine($"{(int)status}: {status}");
}

string[] allNames = Enum.GetNames<OrderStatus>();
Console.WriteLine(string.Join(", ", allNames));
```

When enumerating a flags enum to display granted atomic permissions, use an explicit list of atomic flags; `Enum.GetValues` also includes named composites.

### `IsDefined` and `GetName`

`Enum.IsDefined` checks for an exact named constant. It is useful for validating a non-flags enum value, but it isn't a general validator for arbitrary flags combinations.

```csharp
#nullable enable
using System;

int databaseValue = 2;
bool isDefined = Enum.IsDefined(typeof(OrderStatus), databaseValue);
Console.WriteLine(isDefined); // True

int invalidValue = 99;
Console.WriteLine(Enum.IsDefined(typeof(OrderStatus), invalidValue)); // False

string? name = Enum.GetName(typeof(OrderStatus), 2);
Console.WriteLine(name); // Shipped

string? missing = Enum.GetName(typeof(OrderStatus), 99);
Console.WriteLine(missing is null); // True
```

## Enums in Switch Expressions

Switch expressions work well with enums. If you list every declared member and omit a discard (`_`) arm, the compiler can warn when a later edit adds a member that isn't handled. A catch-all arm handles unnamed or future values but also prevents that missing-member warning.

```csharp
public static class OrderStatusFormatter
{
    public static string GetStatusMessage(OrderStatus status) => status switch
    {
        OrderStatus.Pending => "Your order is awaiting confirmation.",
        OrderStatus.Processing => "Your order is being prepared.",
        OrderStatus.Shipped => "Your order is on its way.",
        OrderStatus.Delivered => "Your order has been delivered.",
        OrderStatus.Cancelled => "Your order was cancelled."
    };
}
```

A switch expression without a matching arm can throw at runtime if it receives an unnamed numeric value. Validate untrusted values before calling it, or add a catch-all arm when runtime fallback behavior is more important than compiler exhaustiveness warnings. For example, the `_` arm below handles every other value; because it covers any future member too, the compiler no longer warns that a newly added member is missing:

```csharp
public static string GetShortLabel(OrderStatus status) => status switch
{
    OrderStatus.Pending => "pending",
    _ => "other or unknown"
};
```

## Enums with Databases and JSON

### Database Mapping

Entity Framework Core stores an enum as its underlying numeric value by default. You can configure a string conversion with `.HasConversion<string>()`:

```csharp
// Default numeric mapping for a property:
public class Order
{
    public OrderStatus Status { get; set; }
}

// In OnModelCreating:
// modelBuilder.Entity<Order>()
//     .Property(order => order.Status)
//     .HasConversion<string>();
```

Numeric storage is compact and can use explicit stable values. String storage is easier to inspect, but the stored strings are based on enum names unless you configure another mapping; renaming a member can require a data migration. Choose based on schema, compatibility, and operational needs rather than a universal performance rule.

### JSON Serialization

`System.Text.Json` serializes enum values as numbers by default. `JsonStringEnumConverter` writes names as strings. Property naming and enum-value naming are separate settings.

```csharp
using System;
using System.Text.Json;
using System.Text.Json.Serialization;

var order = new { Id = 1, Status = OrderStatus.Shipped };

string jsonNumber = JsonSerializer.Serialize(order);
Console.WriteLine(jsonNumber); // {"Id":1,"Status":2}

var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    Converters =
    {
        new JsonStringEnumConverter(
            JsonNamingPolicy.CamelCase,
            allowIntegerValues: false)
    }
};

string jsonString = JsonSerializer.Serialize(order, options);
Console.WriteLine(jsonString); // {"id":1,"status":"shipped"}
```

String values are often easier for API consumers to read, but changing a member name can change the wire value. Treat enum strings as versioned API contract values. Setting `allowIntegerValues: false` prevents the converter from accepting or writing numeric enum values when the contract is string-only.

## Enum Best Practices

### Do

- Use enums for a closed set of mutually exclusive choices; use `[Flags]` only when values are intentionally combinable.
- Use explicit values when the underlying numbers are persisted or form a numeric protocol contract. Keep released values stable.
- For flags, use `None = 0` and powers of two for atomic members; define named composites only for convenience.
- Validate data from external sources. Use `Enum.IsDefined` for ordinary enums, and a known-bit mask for flags combinations.
- Use `Enum.TryParse` when parsing untrusted text, then validate the result against the contract.
- Use an exhaustive switch expression without `_` when you want compiler warnings after enum members are added; use a fallback when handling unnamed values is more important.

### Do Not

- Don't use an enum for an open-ended set that users can extend at runtime.
- Don't renumber or reuse numeric values already stored in a database or published in a protocol.
- Don't assume a cast or `Enum.TryParse` guarantees the result is a declared member.
- Don't use `Enum.IsDefined` as the sole check for a valid flags combination.
- Don't use `[Flags]` for mutually exclusive alternatives, or use `None = 0` as a required permission.

## Basic Code Snippet

This complete .NET 10 console example demonstrates ordinary enums, explicit values, parsing, validation, flags, and an exhaustive switch expression.

```csharp
using System;

Console.WriteLine("=== Basic Enum ===");
OrderStatus status = OrderStatus.Shipped;
Console.WriteLine($"Status: {status}");
Console.WriteLine($"Numeric value: {(int)status}");

Console.WriteLine("\n=== Explicit Values ===");
ApiVerb verb = ApiVerb.Post;
Console.WriteLine($"{verb} = {(int)verb}");

Console.WriteLine("\n=== Enum Methods ===");
string[] names = Enum.GetNames<OrderStatus>();
Console.WriteLine($"All statuses: {string.Join(", ", names)}");

foreach (OrderStatus value in Enum.GetValues<OrderStatus>())
{
    Console.WriteLine($"  {(int)value}: {value}");
}

Console.WriteLine("\n=== Parsing and validation ===");
if (Enum.TryParse<OrderStatus>("Delivered", ignoreCase: true, out OrderStatus parsed)
    && Enum.IsDefined(typeof(OrderStatus), parsed))
{
    Console.WriteLine($"Parsed: {parsed}");
}

Console.WriteLine($"Is 2 defined? {Enum.IsDefined(typeof(OrderStatus), 2)}");
Console.WriteLine($"Is 99 defined? {Enum.IsDefined(typeof(OrderStatus), 99)}");

Console.WriteLine("\n=== Flags ===");
Permission permissions = Permission.Read | Permission.Write | Permission.Delete;
Console.WriteLine($"Permissions: {permissions}");
Console.WriteLine($"Numeric value: {(int)permissions}");
Console.WriteLine($"Has Read: {permissions.HasFlag(Permission.Read)}");
Console.WriteLine($"Has Execute: {permissions.HasFlag(Permission.Execute)}");
Console.WriteLine($"Has None: {permissions.HasFlag(Permission.None)}"); // True: zero-mask rule

permissions |= Permission.Execute;
Console.WriteLine($"After adding Execute: {permissions}");
permissions &= ~Permission.Write;
Console.WriteLine($"After removing Write: {permissions}");

Console.WriteLine("\n=== Switch expression ===");
OrderStatus current = OrderStatus.Processing;
string message = OrderStatusFormatter.GetStatusMessage(current);
Console.WriteLine($"Status message: {message}");

enum OrderStatus
{
    Pending = 0,
    Processing = 1,
    Shipped = 2,
    Delivered = 3,
    Cancelled = 4
}

enum ApiVerb
{
    Get = 1,
    Post = 2,
    Put = 3,
    Patch = 4,
    Delete = 5
}

[Flags]
enum Permission
{
    None = 0,
    Read = 1,
    Write = 2,
    Delete = 4,
    Execute = 8,
    Admin = 16,
    ReadWrite = Read | Write,
    All = Read | Write | Delete | Execute | Admin
}

static class OrderStatusFormatter
{
    public static string GetStatusMessage(OrderStatus status) => status switch
    {
        OrderStatus.Pending => "Awaiting confirmation",
        OrderStatus.Processing => "Being prepared",
        OrderStatus.Shipped => "In transit",
        OrderStatus.Delivered => "Delivered",
        OrderStatus.Cancelled => "Cancelled"
    };
}
```

## Applied Access-Control Example (Illustrative)

This example illustrates enums in a simplified authorization service. It's **not a drop-in production security policy**: real authorization must be reviewed, scoped to the application, and tested. For brevity, `UserAccount.RolePermissions` is assumed to be an effective permission mask already resolved from the user's assigned role(s); the example doesn't implement that role-to-permission data source. The code deliberately denies resource/action pairs that don't have an explicit permission mapping. It also validates the permission bitmask rather than treating an unmapped permission (`None`) as a successful check.

```csharp
#nullable enable
using System;
using System.Collections.Generic;
using System.Text.Json;
using System.Text.Json.Serialization;

namespace MyBackendApp.Core.Services;

public sealed class AccessControlService
{
    private static readonly RolePermission[] AtomicPermissions =
    {
        RolePermission.ReadUsers,
        RolePermission.CreateUsers,
        RolePermission.UpdateUsers,
        RolePermission.DeleteUsers,
        RolePermission.ReadOrders,
        RolePermission.CreateOrders,
        RolePermission.UpdateOrders,
        RolePermission.DeleteOrders,
        RolePermission.ReadFinancials,
        RolePermission.ManageFinancials,
        RolePermission.ExportFinancials,
        RolePermission.ManageSystem
    };

    private readonly IUserRepository _userRepository;
    private readonly IAccessLogger _logger;

    public AccessControlService(IUserRepository userRepository, IAccessLogger logger)
    {
        ArgumentNullException.ThrowIfNull(userRepository);
        ArgumentNullException.ThrowIfNull(logger);
        _userRepository = userRepository;
        _logger = logger;
    }

    public AccessCheckResult CheckAccess(int userId, Resource resource, ResourceAction action)
    {
        UserAccount? user = _userRepository.GetById(userId);
        if (user is null)
        {
            return AccessCheckResult.Deny("User not found.");
        }

        if (user.Status != UserStatus.Active)
        {
            return AccessCheckResult.Deny($"User account is {user.Status}. Access denied.");
        }

        if (!HasOnlyKnownPermissionBits(user.RolePermissions))
        {
            _logger.LogWarning(
                "User {UserId} has unknown permission bits: {Permissions}.",
                userId, user.RolePermissions);
            return AccessCheckResult.Deny("Invalid permission data.");
        }

        RolePermission? requiredPermission = GetRequiredPermission(resource, action);
        if (requiredPermission is null)
        {
            _logger.LogWarning(
                "No permission mapping for resource {Resource} and action {Action}.",
                resource, action);
            return AccessCheckResult.Deny("Unsupported resource/action combination.");
        }

        RolePermission required = requiredPermission.Value;
        if ((user.RolePermissions & required) != required)
        {
            _logger.LogWarning(
                "Access denied for user {UserId}. Required: {Required}. Has: {Has}.",
                userId, required, user.RolePermissions);
            return AccessCheckResult.Deny($"Insufficient permissions. Required: {required}.");
        }

        if (resource == Resource.FinancialReport
            && user.Department != Department.Finance
            && user.Department != Department.Executive)
        {
            return AccessCheckResult.Deny(
                "Financial reports are restricted to Finance and Executive departments.");
        }

        return AccessCheckResult.Allow();
    }

    private static RolePermission? GetRequiredPermission(Resource resource, ResourceAction action) =>
        (resource, action) switch
        {
            (Resource.UserProfile, ResourceAction.Read) => RolePermission.ReadUsers,
            (Resource.UserProfile, ResourceAction.Create) => RolePermission.CreateUsers,
            (Resource.UserProfile, ResourceAction.Update) => RolePermission.UpdateUsers,
            (Resource.UserProfile, ResourceAction.Delete) => RolePermission.DeleteUsers,

            (Resource.Order, ResourceAction.Read) => RolePermission.ReadOrders,
            (Resource.Order, ResourceAction.Create) => RolePermission.CreateOrders,
            (Resource.Order, ResourceAction.Update) => RolePermission.UpdateOrders,
            (Resource.Order, ResourceAction.Delete) => RolePermission.DeleteOrders,

            (Resource.FinancialReport, ResourceAction.Read) => RolePermission.ReadFinancials,
            (Resource.FinancialReport, ResourceAction.Export) => RolePermission.ExportFinancials,
            (Resource.FinancialReport, ResourceAction.Create) => RolePermission.ManageFinancials,
            (Resource.FinancialReport, ResourceAction.Update) => RolePermission.ManageFinancials,
            (Resource.SystemSettings, ResourceAction.Read or ResourceAction.Update) => RolePermission.ManageSystem,

            _ => null
        };

    private static bool HasOnlyKnownPermissionBits(RolePermission permissions) =>
        (permissions & ~RolePermission.FullAccess) == RolePermission.None;

    // Accept defined names (case-insensitively), but reject numeric strings such as "2".
    public ResourceAction? ParseAction(string? actionString)
    {
        if (string.IsNullOrWhiteSpace(actionString))
        {
            return null;
        }

        string candidate = actionString.Trim();
        if (Enum.TryParse<ResourceAction>(candidate, ignoreCase: true, out ResourceAction action)
            && Enum.IsDefined(typeof(ResourceAction), action)
            && string.Equals(
                Enum.GetName(typeof(ResourceAction), action),
                candidate,
                StringComparison.OrdinalIgnoreCase))
        {
            return action;
        }

        _logger.LogWarning("Invalid resource action requested: '{Action}'.", actionString);
        return null;
    }

    public PermissionsSummaryDto GetPermissionsSummary(int userId)
    {
        UserAccount? user = _userRepository.GetById(userId);
        if (user is null)
        {
            return new PermissionsSummaryDto
            {
                UserId = userId,
                IsActive = false,
                GrantedPermissions = Array.Empty<string>()
            };
        }

        if (!HasOnlyKnownPermissionBits(user.RolePermissions))
        {
            _logger.LogWarning(
                "Cannot summarize unknown permission bits for user {UserId}.", userId);
            return new PermissionsSummaryDto
            {
                UserId = userId,
                IsActive = user.Status == UserStatus.Active,
                Role = user.Role.ToString(),
                Department = user.Department.ToString(),
                GrantedPermissions = Array.Empty<string>()
            };
        }

        var grantedPermissions = new List<string>();
        foreach (RolePermission permission in AtomicPermissions)
        {
            if ((user.RolePermissions & permission) == permission)
            {
                grantedPermissions.Add(permission.ToString());
            }
        }

        return new PermissionsSummaryDto
        {
            UserId = userId,
            IsActive = user.Status == UserStatus.Active,
            Role = user.Role.ToString(),
            Department = user.Department.ToString(),
            GrantedPermissions = grantedPermissions.ToArray()
        };
    }

    public string SerializeUserForApi(UserAccount user)
    {
        ArgumentNullException.ThrowIfNull(user);

        // Project to an allowlisted DTO rather than serializing the persistence entity directly.
        var response = new UserApiDto(user.Id, user.Status, user.Department);
        var options = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            Converters =
            {
                new JsonStringEnumConverter(
                    JsonNamingPolicy.CamelCase,
                    allowIntegerValues: false)
            },
            WriteIndented = false
        };

        return JsonSerializer.Serialize(response, options);
    }
}

// Explicit numeric values are used for statuses/roles/actions if these values are stored numerically.
public enum UserStatus
{
    PendingVerification = 0,
    Active = 10,
    Suspended = 20,
    Deactivated = 30,
    Locked = 40
}

// Numeric values are explicit for numeric persistence; string storage still depends on member names.
public enum Department
{
    Engineering = 0,
    Sales = 10,
    Marketing = 20,
    Finance = 30,
    HumanResources = 40,
    Operations = 50,
    Executive = 60,
    Support = 70
}

public enum UserRole
{
    Guest = 0,
    Viewer = 1,
    Editor = 2,
    Manager = 3,
    Director = 4,
    Administrator = 5,
    SuperAdmin = 6
}

public enum Resource
{
    UserProfile,
    Order,
    Product,
    FinancialReport,
    AuditLog,
    SystemSettings
}

public enum ResourceAction
{
    Read = 1,
    Create = 2,
    Update = 3,
    Delete = 4,
    Export = 5,
    Approve = 6
}

[Flags]
public enum RolePermission
{
    None = 0,

    // User permissions (bits 0-3).
    ReadUsers = 1,
    CreateUsers = 2,
    UpdateUsers = 4,
    DeleteUsers = 8,

    // Order permissions (bits 4-7).
    ReadOrders = 16,
    CreateOrders = 32,
    UpdateOrders = 64,
    DeleteOrders = 128,

    // Financial permissions (bits 8-10).
    ReadFinancials = 256,
    ManageFinancials = 512,
    ExportFinancials = 1024,

    // System permission (bit 11).
    ManageSystem = 2048,

    // Named composites don't represent additional atomic bits.
    UserManagement = ReadUsers | CreateUsers | UpdateUsers | DeleteUsers,
    OrderManagement = ReadOrders | CreateOrders | UpdateOrders | DeleteOrders,
    FullAccess = UserManagement | OrderManagement | ReadFinancials |
                 ManageFinancials | ExportFinancials | ManageSystem
}

public sealed class UserAccount
{
    public int Id { get; set; }
    public string Email { get; set; } = string.Empty;
    public string FullName { get; set; } = string.Empty;
    public UserRole Role { get; set; }
    public UserStatus Status { get; set; }
    public Department Department { get; set; }
    public RolePermission RolePermissions { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
    public DateTimeOffset? LastLoginAt { get; set; }
}

public sealed record AccessCheckResult(bool IsAllowed, string Reason)
{
    public static AccessCheckResult Allow() => new(true, "Access granted.");
    public static AccessCheckResult Deny(string reason) => new(false, reason);
}

public sealed class PermissionsSummaryDto
{
    public int UserId { get; set; }
    public bool IsActive { get; set; }
    public string? Role { get; set; }
    public string? Department { get; set; }
    public string[] GrantedPermissions { get; set; } = Array.Empty<string>();
}

// Deliberately omits private fields and the internal permission bitmask.
public sealed record UserApiDto(int Id, UserStatus Status, Department Department);

public interface IUserRepository
{
    UserAccount? GetById(int id);
}

public interface IAccessLogger
{
    void LogWarning(string message, params object?[] args);
}
```

### Key Observations

- `UserStatus`, `Department`, `UserRole`, and `ResourceAction` have explicit numeric values so persisted or externally meaningful values don't shift when members are inserted. `UserStatus` leaves gaps; gaps are optional, but existing numeric values must remain stable. Use numeric values only where they are part of the storage or protocol contract.
- A flags enum stores a bitmask. It's compact for a small, bounded set of capabilities, but a relational permission table may be more appropriate for dynamic permissions, querying, or auditing.
- `RolePermission` uses powers of two for atomic permissions and composite values for convenience. Update `FullAccess` and the known-bit mask whenever a new atomic permission is introduced.
- `HasFlag(None)` is always true—including `RolePermission.None`—so `None` must not be used as a required permission. The access-control code rejects unmapped resource/action pairs rather than mapping them to `None`.
- `Enum.TryParse` accepts numeric strings as well as names. `Enum.IsDefined` checks a named value for ordinary enums, but doesn't validate arbitrary flags combinations. For string-only external contracts, reject numeric strings; for flags, check the allowed-bit mask.
- `Enum.GetValues<RolePermission>()` would include composites such as `UserManagement` and `FullAccess`. The summary instead iterates only the atomic permission list.
- `JsonStringEnumConverter` can make API values self-describing. Treat string names as stable wire values, and configure `allowIntegerValues: false` if the API is string-only. The sample serializes an allowlisted DTO rather than exposing the persistence entity and internal permission mask.
- EF Core's string conversion stores enum member names. Renames require data migrations or a stable custom conversion; numeric storage requires stable explicit codes.
- The switch expression without `_` lets the compiler warn when a new `OrderStatus` member is unhandled. A catch-all arm suppresses that warning, so validate untrusted numeric values separately.
- The authorization example intentionally has no `None` fallback. A missing mapping denies access instead of passing a zero-valued permission check.

## Key Terms Summary

| Term | Definition |
|---|---|
| Enum | A value type with named constants backed by an integral type. |
| Enum member | A named constant in an enum declaration. |
| Underlying type | The integral type used to store enum values; `int` by default. |
| Explicit value | A chosen underlying numeric value for an enum member. |
| `[Flags]` | An attribute that marks an enum as intended for bitwise combinations. |
| Atomic flag | A single power-of-two flag representing one independent bit. |
| Composite flag | A named combination of atomic flags. |
| `HasFlag` | Checks whether all bits in the supplied mask are present; a zero mask always matches. |
| `Enum.Parse` | Converts a name or numeric string to an enum value and throws on parse failure. |
| `Enum.TryParse` | Attempts to parse a name or numeric string and reports parse success. |
| `Enum.IsDefined` | Checks whether an exact declared enum value/name exists; not a general flags-combination validator. |
| `Enum.GetValues` | Returns an array containing the declared enum values, including composites and aliases. |
| `JsonStringEnumConverter` | A `System.Text.Json` converter that serializes enum values as strings. |

## Further Reading

- [Enumeration types — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum) — declaration, underlying types, conversions, and zero values.
- [`FlagsAttribute` API](https://learn.microsoft.com/en-us/dotnet/api/system.flagsattribute?view=net-10.0) — intended use and formatting for flags enums.
- [`Enum.IsDefined` API](https://learn.microsoft.com/en-us/dotnet/api/system.enum.isdefined?view=net-10.0) — exact-member validation and flags caveats.
- [`Enum.TryParse` API](https://learn.microsoft.com/en-us/dotnet/api/system.enum.tryparse?view=net-10.0) — parsing names and numeric strings.
- [`Enum.GetValues` API](https://learn.microsoft.com/en-us/dotnet/api/system.enum.getvalues?view=net-10.0) — returned values, aliases, and ordering.
- [`Enum.HasFlag` API](https://learn.microsoft.com/en-us/dotnet/api/system.enum.hasflag?view=net-10.0) — bit-mask behavior, including the zero flag.
- [Resolve pattern-matching errors and warnings](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/pattern-matching-warnings) — switch-expression completeness warnings.
- [Customize properties and values with `System.Text.Json`](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/customize-properties) — string enum converters and naming policies.
- [Value conversions — EF Core](https://learn.microsoft.com/en-us/ef/core/modeling/value-conversions) — storing enum values as numbers or strings.
