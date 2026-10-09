## What Is an Array?

An array is a fixed-length, ordered sequence whose elements share an element type. Arrays are useful when the number of elements is known or when an API works naturally with indexed data. `List<T>` is a common alternative when the number of elements needs to change.

Key characteristics of ordinary C# arrays:

- **Fixed length:** each array instance has a fixed length for its lifetime. `Array.Resize` creates a different array and updates a variable to refer to it; it does not change the original array object in place.
- **Zero-based indexing:** arrays created with normal C# array syntax use indices from `0` through `Length - 1`.
- **Same element type:** the array has an element type such as `int` or `Customer`. Reference-array covariance is an exception to simple compile-time intuition; an incompatible store through a covariant reference can throw at runtime.
- **Reference type:** even an `int[]` is an array reference type. Value-type elements such as `int` are stored as values in the array; reference-type elements store references to objects.
- **Indexed access:** reading or writing an element by index is an O(1) operation. Array elements are stored contiguously and in order, but actual performance depends on the operation and workload.

The key ideas are fixed length and indexed access. You can use arrays directly, or use higher-level collections such as `List<T>` when their operations better fit the problem.

## Single-Dimensional Arrays

A single-dimensional array is a linear sequence of elements. This is the most common array shape in backend development.

### Declaration and Initialization

```csharp
// 1. Create an array of a specified length; elements receive their default values.
int[] numbers = new int[5];
// numbers: { 0, 0, 0, 0, 0 }

// 2. Declare and initialize with specific values.
int[] scores = { 85, 92, 78, 95, 88 };

// 3. Explicit new expression with an initializer.
string[] names = new string[] { "Alice", "Bob", "Charlie" };

// 4. Specify both length and values; the length must match the initializer.
double[] prices = new double[3] { 19.99, 29.99, 39.99 };

// 5. Infer the variable's type from the array creation expression.
var flags = new bool[] { true, false, true };
```

With C# 12 and later, a collection expression can also initialize an array:

```csharp
int[] values = [10, 20, 30];
```

The resulting `int[]` still has a fixed length.

### Default Values

When an array is created without explicitly setting every element, each element begins with the default value for its element type. An enum's default is its underlying zero value, even when no named member has the value `0`.

| Element type | Default value |
|---|---|
| Integral numeric types | `0` |
| `float`, `double` | `0` |
| `decimal` | `0m` |
| `bool` | `false` |
| `char` | `\0` (the NUL character) |
| Reference types | `null` |
| Nullable value types such as `int?` | `null` |
| Other value types, including structs and enums | Their default value; fields are defaulted as well |

```csharp
#nullable enable

int[] integers = new int[3];          // { 0, 0, 0 }
bool[] booleans = new bool[3];        // { false, false, false }
string?[] strings = new string?[3];   // { null, null, null }
int?[] optionalNumbers = new int?[3]; // { null, null, null }
```

Nullable reference type annotations help the compiler identify possible null values; they don't change the runtime default. Even a `string[]` created with a length has `null` elements until they are assigned. Use `string?[]` when null elements are part of the intended data model.

### Accessing and Updating Elements

Use the index operator `[]` to read or assign an element.

```csharp
using System;

string[] fruits = { "Apple", "Banana", "Cherry", "Date", "Elderberry" };

// Read.
Console.WriteLine(fruits[0]); // Apple
Console.WriteLine(fruits[2]); // Cherry
Console.WriteLine(fruits[4]); // Elderberry

// Update.
fruits[1] = "Blueberry";
Console.WriteLine(fruits[1]); // Blueberry
```

### Index-from-End Operator (`^`)

The index-from-end operator counts backward from the end. `^1` means the last element, `^2` the second-to-last, and so on. This operator was introduced in C# 8.

```csharp
using System;

int[] data = { 10, 20, 30, 40, 50 };

Console.WriteLine(data[^1]); // 50, the last element
Console.WriteLine(data[^2]); // 40, the second-to-last element
Console.WriteLine(data[^5]); // 10, the first element
```

An index that resolves outside the array's bounds throws `IndexOutOfRangeException`.

### Range Operator (`..`)

A range has an inclusive start and an exclusive end. The range operator was introduced in C# 8. When a range is applied to an array, it creates a new array containing a shallow copy of the selected elements; it doesn't modify the source.

```csharp
int[] data = { 10, 20, 30, 40, 50 };

int[] firstThree = data[..3];            // { 10, 20, 30 }
int[] lastTwo = data[3..];               // { 40, 50 }
int[] middle = data[1..4];               // { 20, 30, 40 }
int[] all = data[..];                    // A copy: { 10, 20, 30, 40, 50 }
int[] suffixFromSecondLast = data[^2..]; // { 40, 50 }
```

A shallow copy duplicates the element values or references, not objects referenced by reference-type elements. By contrast, a range over `Span<T>` or `Memory<T>` creates a view rather than a new array; see the overview later in this section.

### Length, Rank, and Bounds

```csharp
using System;

int[] numbers = { 1, 2, 3, 4, 5 };

Console.WriteLine(numbers.Length); // 5, the number of elements
Console.WriteLine(numbers.Rank);   // 1, the number of dimensions

// Valid indices are 0 through Length - 1.
// numbers[5]  throws IndexOutOfRangeException.
// numbers[-1] throws IndexOutOfRangeException.
```

`Length` counts all elements. For a multidimensional array, use `GetLength(dimension)` to get the size of a particular dimension.

### Iterating Over an Array

Use `foreach` when you don't need an index, and `for` when the index is part of the logic. LINQ can express transformations compactly; it returns a query or a new collection depending on the operation.

```csharp
using System;
using System.Linq;

string[] colors = { "Red", "Green", "Blue", "Yellow" };

// foreach: convenient when the index isn't needed.
foreach (string color in colors)
{
    Console.WriteLine(color);
}

// for: use when the index is needed.
for (int i = 0; i < colors.Length; i++)
{
    Console.WriteLine($"[{i}] {colors[i]}");
}

// Transform to a new array. ToUpperInvariant is culture-independent.
string[] upperColors = colors.Select(color => color.ToUpperInvariant()).ToArray();
```

## The `Array` Class

`System.Array` provides static methods for searching, sorting, copying, filling, and resizing arrays.

### Searching

```csharp
using System;

int[] numbers = { 5, 3, 8, 1, 9, 2, 7 };

// IndexOf returns the first matching index, or -1 if there is no match.
int index = Array.IndexOf(numbers, 8);
Console.WriteLine(index); // 2

int notFound = Array.IndexOf(numbers, 99);
Console.WriteLine(notFound); // -1

// LastIndexOf searches from the end.
int[] duplicates = { 1, 2, 3, 2, 1 };
int lastTwo = Array.LastIndexOf(duplicates, 2);
Console.WriteLine(lastTwo); // 3

// Find returns the first matching element, or default(T) if none matches.
int firstEven = Array.Find(numbers, number => number % 2 == 0);
Console.WriteLine(firstEven); // 8

// FindAll returns a new array containing matching elements.
int[] allEven = Array.FindAll(numbers, number => number % 2 == 0);
// allEven: { 8, 2 }

// FindIndex returns the index of the first matching element, or -1.
int indexOfFirstEven = Array.FindIndex(numbers, number => number % 2 == 0);
Console.WriteLine(indexOfFirstEven); // 2

// Exists checks whether at least one element matches.
bool hasNegative = Array.Exists(numbers, number => number < 0);
Console.WriteLine(hasNegative); // False

// TrueForAll checks whether every element matches.
bool allPositive = Array.TrueForAll(numbers, number => number > 0);
Console.WriteLine(allPositive); // True
```

For a value type, `Array.Find`'s `default(T)` result can be indistinguishable from a matching value. For example, `0` could mean either “found zero” or “no match.” Use `FindIndex` or another representation when that distinction matters.

### Sorting

`Array.Sort` sorts in place. For strings, the default comparison is culture-sensitive and case-sensitive; its ordering can depend on the current culture, so don't rely on a universal uppercase-before-lowercase result. Specify a comparer when you need consistent behavior.

```csharp
using System;

int[] numbers = { 5, 3, 8, 1, 9, 2, 7 };

// Ascending order, in place.
Array.Sort(numbers);
// numbers: { 1, 2, 3, 5, 7, 8, 9 }

// Descending order: sort, then reverse in place.
Array.Reverse(numbers);
// numbers: { 9, 8, 7, 5, 3, 2, 1 }

// Sort four elements starting at index 1 (indices 1 through 4).
int[] data = { 9, 5, 3, 8, 1, 7 };
Array.Sort(data, 1, 4);
// data: { 9, 1, 3, 5, 8, 7 }

string[] names = { "Charlie", "alice", "Bob" };

// Default string comparison can vary with current culture.
Array.Sort(names);

// Explicit ordinal ordering is deterministic; in this ASCII sample, capitals sort first.
string[] ordinalNames = { "Charlie", "alice", "Bob" };
Array.Sort(ordinalNames, StringComparer.Ordinal);
// ordinalNames: { "Bob", "Charlie", "alice" }

// Explicit ordinal, case-insensitive ordering is also culture-independent.
Array.Sort(names, StringComparer.OrdinalIgnoreCase);
// names: { "alice", "Bob", "Charlie" }
```

Use `StringComparer.Ordinal` or `StringComparer.OrdinalIgnoreCase` for culture-independent identifiers and protocol-like values. Use culture-aware comparison when sorting user-facing text according to language rules.

### Binary Search

`Array.BinarySearch` performs an O(log N) search on an array that is already sorted according to the same ordering used by the search. Sorting first costs time too, so binary search is most useful when the sorted array will be searched repeatedly.

```csharp
using System;

int[] sorted = { 1, 3, 5, 7, 9, 11, 13, 15, 17, 19 };

int found = Array.BinarySearch(sorted, 11);
Console.WriteLine(found); // 5

int notFound = Array.BinarySearch(sorted, 6);
if (notFound >= 0)
{
    Console.WriteLine($"Found at index {notFound}");
}
else
{
    int insertionPoint = ~notFound; // Bitwise complement gives the insertion point.
    Console.WriteLine($"Not found; insertion point: {insertionPoint}");
}
```

The array must be sorted using the same comparer as the search. If it isn't, the result isn't reliable. When the value is absent, the negative return value encodes the insertion point as its bitwise complement.

### Copying and Cloning

Array copies are shallow: value-type elements are copied, while reference-type elements are copied as references. Referenced objects aren't cloned.

```csharp
using System;

int[] original = { 1, 2, 3, 4, 5 };

// Clone creates a shallow copy.
int[] cloned = (int[])original.Clone();
cloned[0] = 99;
Console.WriteLine(original[0]); // 1

// CopyTo copies into an existing array at the requested starting index.
int[] destination = new int[10];
original.CopyTo(destination, 2);
// destination: { 0, 0, 1, 2, 3, 4, 5, 0, 0, 0 }

// Array.Copy copies a range of elements.
int[] partial = new int[3];
Array.Copy(original, 1, partial, 0, 3);
// partial: { 2, 3, 4 }

// Copy an entire array into a newly created destination.
int[] fullCopy = new int[original.Length];
Array.Copy(original, fullCopy, original.Length);
```

### Filling and Clearing

`Array.Fill` assigns a value to all elements or to a selected range. `Array.Clear` resets a range to the element type's default value.

```csharp
using System;

int[] data = new int[5];

Array.Fill(data, 42);
// data: { 42, 42, 42, 42, 42 }

Array.Fill(data, 99, 1, 3); // Start at index 1; fill 3 elements.
// data: { 42, 99, 99, 99, 42 }

Array.Clear(data, 0, data.Length);
// data: { 0, 0, 0, 0, 0 }

Array.Fill(data, 5);
Array.Clear(data, 2, 2);
// data: { 5, 5, 0, 0, 5 }
```

`Array.Fill` is available in modern .NET versions, including .NET 10.

### Resizing

Arrays have a fixed length. `Array.Resize` allocates a new array when the requested length differs, copies the retained elements, and assigns the new array reference back through the `ref` argument. Other variables that still reference the old array continue to refer to it.

```csharp
using System;

int[] numbers = { 1, 2, 3 };
int[] oldReference = numbers;

Array.Resize(ref numbers, 5);
// numbers: { 1, 2, 3, 0, 0 }
Console.WriteLine(numbers.Length);      // 5
Console.WriteLine(oldReference.Length); // 3; it still refers to the original array

Array.Resize(ref numbers, 2);
// numbers: { 1, 2 }
```

Resizing repeatedly in a loop can allocate and copy many arrays. Use `List<T>` when the collection needs frequent growth or removal.

### Converting Between Element Types

`Array.ConvertAll` transforms every element into a new array of another type.

```csharp
using System;

int[] numbers = { 1, 2, 3, 4, 5 };

string[] strings = Array.ConvertAll(numbers, number => number.ToString());
// strings: { "1", "2", "3", "4", "5" }

double[] doubles = Array.ConvertAll(numbers, number => number * 1.5);
// doubles: { 1.5, 3.0, 4.5, 6.0, 7.5 }
```

## Multidimensional (Rectangular) Arrays

A multidimensional array has two or more dimensions. A rectangular array has a fixed length in each dimension, so each row has the same number of columns.

### Declaration and Initialization

```csharp
using System;

// 2D array: 3 rows and 4 columns.
int[,] grid =
{
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 }
};

Console.WriteLine(grid[0, 0]); // 1 (row 0, column 0)
Console.WriteLine(grid[1, 2]); // 7
Console.WriteLine(grid[2, 3]); // 12

grid[0, 0] = 99; // Update an element.
```

### Properties and Iteration

```csharp
using System;

int[,] matrix =
{
    { 1, 2, 3 },
    { 4, 5, 6 }
};

Console.WriteLine(matrix.Length);       // 6, total number of elements
Console.WriteLine(matrix.Rank);         // 2, number of dimensions
Console.WriteLine(matrix.GetLength(0)); // 2, number of rows
Console.WriteLine(matrix.GetLength(1)); // 3, number of columns

// foreach visits the elements in row-major order.
foreach (int value in matrix)
{
    Console.Write($"{value} ");
}
// Output: 1 2 3 4 5 6

// Use nested loops when both row and column indices are needed.
for (int row = 0; row < matrix.GetLength(0); row++)
{
    for (int column = 0; column < matrix.GetLength(1); column++)
    {
        Console.Write($"{matrix[row, column],4}");
    }

    Console.WriteLine();
}
```

Output of the nested loops:

```text
   1   2   3
   4   5   6
```

### Three-Dimensional Arrays

```csharp
using System;

// 3D array: 2 layers, 3 rows per layer, 4 columns per row.
int[,,] cube = new int[2, 3, 4];

cube[0, 0, 0] = 1;
cube[1, 2, 3] = 99;

Console.WriteLine(cube.Rank);         // 3
Console.WriteLine(cube.GetLength(0)); // 2
Console.WriteLine(cube.GetLength(1)); // 3
Console.WriteLine(cube.GetLength(2)); // 4
```

Higher-dimensional arrays are less common in general backend code. If the dimensions represent domain concepts, a named type may communicate the model more clearly.

## Jagged Arrays (Arrays of Arrays)

A jagged array is an array whose elements are themselves arrays. Each inner array can have a different length, and each inner array is a separate array object.

### Declaration and Initialization

```csharp
// Create an outer array for three inner arrays.
int[][] jagged = new int[3][];

// Initialize each inner array.
jagged[0] = new int[] { 1, 2, 3 };
jagged[1] = new int[] { 4, 5 };
jagged[2] = new int[] { 6, 7, 8, 9, 10 };

// Shorthand initialization when all rows are known.
int[][] shorthand =
{
    new int[] { 1, 2, 3 },
    new int[] { 4, 5 },
    new int[] { 6, 7, 8, 9, 10 }
};
```

The outer array's elements default to `null` until each inner array is assigned. Each inner array also has a fixed length; to change a row's length, create and assign a replacement array.

### Accessing and Iterating

```csharp
using System;

int[][] jagged =
{
    new int[] { 1, 2, 3 },
    new int[] { 4, 5 },
    new int[] { 6, 7, 8, 9, 10 }
};

Console.WriteLine(jagged[0][2]); // 3: first inner array, third element
Console.WriteLine(jagged[1][1]); // 5
Console.WriteLine(jagged[2][4]); // 10

for (int row = 0; row < jagged.Length; row++)
{
    Console.Write($"Row {row} ({jagged[row].Length} elements): ");

    foreach (int value in jagged[row])
    {
        Console.Write($"{value} ");
    }

    Console.WriteLine();
}
```

Output:

```text
Row 0 (3 elements): 1 2 3
Row 1 (2 elements): 4 5
Row 2 (5 elements): 6 7 8 9 10
```

The notation `[][]` means an array of arrays. A rectangular array uses comma-separated dimensions such as `[,]`.

### Jagged vs. Rectangular Arrays

| Characteristic | Rectangular array (`[,]`) | Jagged array (`[][]`) |
|---|---|---|
| Shape | Rectangular; each dimension has a fixed length | Inner arrays can have different lengths |
| Storage | One multidimensional array object with elements laid out in order | An outer array plus separately allocated inner arrays |
| Access syntax | `grid[row, column]` | `jagged[row][column]` |
| Flexibility | Dimensions are fixed for that array instance | Each row can be a different length and can be replaced independently |
| Performance | Depends on access pattern and runtime | Depends on access pattern and runtime; benchmark if it matters |
| Common use | Matrices, grids, fixed-size tables | Variable-length rows or grouped sequences |

Neither representation is universally faster; choose based on the shape and operations your data needs.

## Arrays of Reference Types

When an array's element type is a class, its elements hold references to objects. Creating the array initializes its elements to `null`; it doesn't create each `Customer` object.

```csharp
#nullable enable
using System;

Customer?[] customers = new Customer?[3];

Console.WriteLine(customers[0] is null); // True

customers[0] = new Customer { Name = "Alice", Email = "alice@example.com" };
customers[1] = new Customer { Name = "Bob", Email = "bob@example.com" };
customers[2] = new Customer { Name = "Charlie", Email = "charlie@example.com" };

Customer first = customers[0]!; // This element was assigned above.
first.Name = "Alicia";
Console.WriteLine(customers[0]!.Name); // Alicia; both references point to the same object

// Type declarations follow the top-level statements in Program.cs.
class Customer
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}
```

`Customer?[]` makes the possibility of uninitialized `null` elements explicit. For an array of value types such as `int[]`, the values themselves are stored in the array elements.

Reference arrays are covariant: a `string[]` can be assigned to an `object[]` variable. The runtime still enforces the array's actual element type when you write through the `object[]` reference:

```csharp
string[] words = { "hello" };
object[] objects = words; // Array covariance.

// objects[0] = 42; // Throws ArrayTypeMismatchException at runtime.
```

## Arrays vs. `List<T>`

| Characteristic | Array (`T[]`) | `List<T>` |
|---|---|---|
| Size | Fixed for each array instance | Resizable; internally manages capacity |
| Add/remove elements | Not supported as in-place length changes | `Add`, `Remove`, `Insert`, and related methods |
| Indexed access | O(1) | O(1) |
| Storage overhead | Array object and its elements | List object plus backing array and possible unused capacity |
| LINQ | Supported | Supported |
| API boundaries | Required or convenient for some APIs | Common when callers need list operations |
| Interop | Often used by APIs such as `string.Split` and `File.ReadAllLines` | Can be converted with `ToArray()` |

Both arrays and `List<T>` provide O(1) indexed access. An array can have less wrapper overhead, but don't assume it will be measurably faster for every workload.

### When to Use Arrays

- The length is fixed or already known.
- The API you're calling or implementing requires an array.
- You need an array-specific operation or interop representation.
- You need to expose a contiguous memory region through `Span<T>` or `Memory<T>`.

### When to Use `List<T>`

- The number of elements changes over time.
- You need convenient insertion, removal, or growth.
- The collection is general-purpose and list operations make the code clearer.

### Converting Between Arrays and Lists

These conversions create a new collection and copy the elements; they don't make a resizable view over the original.

```csharp
using System.Collections.Generic;
using System.Linq;

int[] array = { 1, 2, 3, 4, 5 };
List<int> list = array.ToList();

List<string> stringList = new List<string> { "A", "B", "C" };
string[] stringArray = stringList.ToArray();
```

## `Span<T>` and `Memory<T>` (Overview)

`Span<T>` and `Memory<T>` represent views over contiguous memory. Slicing either type creates a view instead of copying the array elements.

```csharp
using System;

int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

Span<int> span = numbers.AsSpan();
Span<int> slice = span[2..5]; // View over elements 3, 4, 5; no array allocation.

slice[0] = 99; // Changes the original array.
Console.WriteLine(numbers[2]); // 99

Memory<int> memory = numbers.AsMemory();
Memory<int> memorySlice = memory[2..5]; // Also a view; Memory<T> can cross await boundaries.
Console.WriteLine(memorySlice.Span[0]); // 99
```

`Span<T>` is a `ref struct`: it has lifetime restrictions and can't be stored in ordinary class fields or used across an `await`/`yield` suspension. C# 13 and later allow some `Span<T>` locals inside async methods as long as they aren't used across a suspension point. `Memory<T>` can be stored in fields and passed across asynchronous boundaries, but it doesn't own the underlying memory by itself.

Use these types when a non-copying view is useful and the lifetime rules are clear. They are valuable tools for high-performance code, but don't add them solely for an assumed speed improvement; measure the real workload.

## Basic Code Snippet

This complete .NET 10 console example demonstrates one-dimensional, rectangular, and jagged arrays, along with common `Array` methods.

```csharp
using System;

// --- Single-dimensional array ---
Console.WriteLine("=== Single-Dimensional ===");
string[] languages = { "C#", "Python", "Java", "Go", "Rust" };

Console.WriteLine($"Length: {languages.Length}");
Console.WriteLine($"First: {languages[0]}");
Console.WriteLine($"Last: {languages[^1]}");

string[] topThree = languages[..3];
Console.WriteLine($"Top 3: {string.Join(", ", topThree)}");

// --- Array methods ---
Console.WriteLine("\n=== Array Methods ===");
int[] scores = { 72, 95, 88, 61, 83, 97, 55 };

Array.Sort(scores);
Console.WriteLine($"Sorted: {string.Join(", ", scores)}");

int highScore = Array.Find(scores, score => score >= 95);
Console.WriteLine($"First score >= 95: {highScore}");

int[] passing = Array.FindAll(scores, score => score >= 70);
Console.WriteLine($"Passing scores: {string.Join(", ", passing)}");

bool allPassing = Array.TrueForAll(scores, score => score >= 50);
Console.WriteLine($"All >= 50: {allPassing}");

// --- Rectangular array ---
Console.WriteLine("\n=== Rectangular (2D) ===");
int[,] seating =
{
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 }
};

Console.WriteLine($"Rows: {seating.GetLength(0)}, Columns: {seating.GetLength(1)}");

for (int row = 0; row < seating.GetLength(0); row++)
{
    for (int column = 0; column < seating.GetLength(1); column++)
    {
        Console.Write($"{seating[row, column],4}");
    }

    Console.WriteLine();
}

// --- Jagged array ---
Console.WriteLine("\n=== Jagged Array ===");
int[][] weeklyHours =
{
    new int[] { 8, 8, 8, 8, 8 },    // Week 1: 5 days
    new int[] { 8, 8, 6 },           // Week 2: 3 days
    new int[] { 8, 8, 8, 8, 8, 4 }   // Week 3: 6 days
};

for (int week = 0; week < weeklyHours.Length; week++)
{
    int total = 0;
    foreach (int hours in weeklyHours[week])
    {
        total += hours;
    }

    Console.WriteLine(
        $"Week {week + 1}: {weeklyHours[week].Length} days, {total} total hours");
}

// --- Fill and clear ---
Console.WriteLine("\n=== Fill and Clear ===");
int[] buffer = new int[6];
Array.Fill(buffer, 7);
Console.WriteLine($"Filled: {string.Join(", ", buffer)}");

Array.Clear(buffer, 2, 2);
Console.WriteLine($"Cleared middle: {string.Join(", ", buffer)}");
```

## Array Processing Example (Illustrative)

This example demonstrates array preallocation, parsing, rectangular-array aggregation, batching, and sorting. It is **illustrative, not a production CSV or payment/reporting pipeline**. The `Split(',')` call doesn't handle quoted CSV fields or embedded commas; use a CSV parser for real CSV input. The example also specifies invariant parsing rules so dates and decimal values don't depend on the machine's current culture.

```csharp
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Threading;
using System.Threading.Tasks;

namespace MyBackendApp.Core.Services;

public sealed class ReportDataProcessor
{
    private const int MaxBatchSize = 1_000;
    private const int CsvExpectedColumnCount = 6;

    private readonly IReportLogger _logger;

    public ReportDataProcessor(IReportLogger logger)
    {
        ArgumentNullException.ThrowIfNull(logger);
        _logger = logger;
    }

    // Simplified comma-delimited format: this is not a general CSV parser.
    // Expected date format is yyyy-MM-dd; decimal fields use invariant notation.
    public ParsedCsvResult ParseCsvData(string[] csvLines)
    {
        ArgumentNullException.ThrowIfNull(csvLines);

        if (csvLines.Length == 0)
        {
            return new ParsedCsvResult
            {
                Errors = new[] { "CSV input is empty." }
            };
        }

        string[] headers = csvLines[0].Split(',', StringSplitOptions.TrimEntries);
        if (headers.Length != CsvExpectedColumnCount)
        {
            return new ParsedCsvResult
            {
                Errors = new[]
                {
                    $"Expected {CsvExpectedColumnCount} header columns, found {headers.Length}."
                }
            };
        }

        // Upper bound: every remaining line could represent one valid record.
        int dataRowCount = csvLines.Length - 1;
        var records = new SalesRecord[dataRowCount];
        var errors = new List<string>();
        int validCount = 0;

        for (int i = 1; i < csvLines.Length; i++)
        {
            string line = csvLines[i];
            if (string.IsNullOrWhiteSpace(line))
            {
                continue;
            }

            string[] fields = line.Split(',', StringSplitOptions.TrimEntries);
            if (fields.Length != CsvExpectedColumnCount)
            {
                errors.Add(
                    $"Row {i}: expected {CsvExpectedColumnCount} fields, found {fields.Length}.");
                continue;
            }

            if (!int.TryParse(
                    fields[0], NumberStyles.Integer, CultureInfo.InvariantCulture, out int orderId))
            {
                errors.Add($"Row {i}: invalid OrderId '{fields[0]}'.");
                continue;
            }

            if (!DateTime.TryParseExact(
                    fields[1], "yyyy-MM-dd", CultureInfo.InvariantCulture,
                    DateTimeStyles.None, out DateTime orderDate))
            {
                errors.Add($"Row {i}: invalid OrderDate '{fields[1]}'.");
                continue;
            }

            if (!decimal.TryParse(
                    fields[3], NumberStyles.Number, CultureInfo.InvariantCulture, out decimal amount))
            {
                errors.Add($"Row {i}: invalid Amount '{fields[3]}'.");
                continue;
            }

            if (!int.TryParse(
                    fields[4], NumberStyles.Integer, CultureInfo.InvariantCulture, out int quantity))
            {
                errors.Add($"Row {i}: invalid Quantity '{fields[4]}'.");
                continue;
            }

            records[validCount] = new SalesRecord
            {
                OrderId = orderId,
                OrderDate = orderDate,
                CustomerName = fields[2],
                Amount = amount,
                Quantity = quantity,
                Region = fields[5]
            };

            validCount++;
        }

        // This single resize trims unused slots left by invalid or blank rows.
        if (validCount < records.Length)
        {
            Array.Resize(ref records, validCount);
        }

        _logger.LogInformation(
            "Parsed {Valid} valid records from {Total} CSV lines. Errors: {Errors}.",
            validCount, csvLines.Length, errors.Count);

        return new ParsedCsvResult
        {
            Records = records,
            Headers = headers,
            Errors = errors.ToArray()
        };
    }

    // Rows represent regions; columns represent January through December.
    public MonthlyRevenueReport GenerateMonthlyReport(SalesRecord[] records)
    {
        ArgumentNullException.ThrowIfNull(records);

        string[] regions = { "North", "South", "East", "West" };
        int regionCount = regions.Length;
        const int monthCount = 12;

        decimal[,] revenue = new decimal[regionCount, monthCount];
        int[,] orderCounts = new int[regionCount, monthCount];
        bool hasKnownRegionRecords = false;

        foreach (SalesRecord record in records)
        {
            // This small fixed list uses exact, case-sensitive string matching.
            int regionIndex = Array.IndexOf(regions, record.Region);
            if (regionIndex < 0)
            {
                _logger.LogWarning(
                    "Unknown region '{Region}' in order {OrderId}.",
                    record.Region, record.OrderId);
                continue;
            }

            hasKnownRegionRecords = true;
            int monthIndex = record.OrderDate.Month - 1;
            revenue[regionIndex, monthIndex] += record.Amount;
            orderCounts[regionIndex, monthIndex]++;
        }

        decimal[] monthlyTotals = new decimal[monthCount];
        decimal[] regionTotals = new decimal[regionCount];
        decimal grandTotal = 0m;

        for (int region = 0; region < regionCount; region++)
        {
            for (int month = 0; month < monthCount; month++)
            {
                decimal amount = revenue[region, month];
                monthlyTotals[month] += amount;
                regionTotals[region] += amount;
                grandTotal += amount;
            }
        }

        int peakMonth = 0; // 0 means no records had a recognized region.
        if (hasKnownRegionRecords)
        {
            int peakMonthIndex = 0;
            for (int month = 1; month < monthCount; month++)
            {
                if (monthlyTotals[month] > monthlyTotals[peakMonthIndex])
                {
                    peakMonthIndex = month;
                }
            }

            peakMonth = peakMonthIndex + 1; // Return a 1-based month number.
        }

        return new MonthlyRevenueReport
        {
            Regions = regions,
            RevenueMatrix = revenue,
            OrderCountMatrix = orderCounts,
            MonthlyTotals = monthlyTotals,
            RegionTotals = regionTotals,
            GrandTotal = grandTotal,
            PeakMonth = peakMonth
        };
    }

    // The callback receives a read-only view rather than a copied array.
    public async Task<int> ProcessInBatchesAsync(
        SalesRecord[] records,
        Func<ReadOnlyMemory<SalesRecord>, CancellationToken, Task<int>> batchProcessor,
        CancellationToken cancellationToken)
    {
        ArgumentNullException.ThrowIfNull(records);
        ArgumentNullException.ThrowIfNull(batchProcessor);

        int totalProcessed = 0;
        int totalBatches = records.Length / MaxBatchSize;
        if (records.Length % MaxBatchSize != 0)
        {
            totalBatches++;
        }

        _logger.LogInformation(
            "Processing {Count} records in {Batches} batches of up to {Size}.",
            records.Length, totalBatches, MaxBatchSize);

        for (int offset = 0; offset < records.Length;)
        {
            cancellationToken.ThrowIfCancellationRequested();

            int batchSize = Math.Min(MaxBatchSize, records.Length - offset);
            ReadOnlyMemory<SalesRecord> batch = records.AsMemory(offset, batchSize);

            int batchResult = await batchProcessor(batch, cancellationToken);
            totalProcessed += batchResult;

            int batchNumber = offset / MaxBatchSize + 1;
            _logger.LogInformation(
                "Batch {Batch}/{Total}: processed {Count} records.",
                batchNumber, totalBatches, batchResult);

            offset += batchSize;
        }

        return totalProcessed;
    }

    public SalesRecord[] GetTopPerformingOrders(SalesRecord[] records, int topN)
    {
        ArgumentNullException.ThrowIfNull(records);
        ArgumentOutOfRangeException.ThrowIfNegative(topN);

        if (records.Length == 0 || topN == 0)
        {
            return Array.Empty<SalesRecord>();
        }

        // Copy first so sorting doesn't change the caller's array.
        SalesRecord[] sorted = (SalesRecord[])records.Clone();
        Array.Sort(sorted, (left, right) => right.Amount.CompareTo(left.Amount));

        int count = Math.Min(topN, sorted.Length);
        SalesRecord[] topOrders = new SalesRecord[count];
        Array.Copy(sorted, topOrders, count);
        return topOrders;
    }
}

public sealed class SalesRecord
{
    public int OrderId { get; set; }
    public DateTime OrderDate { get; set; }
    public string CustomerName { get; set; } = string.Empty;
    public decimal Amount { get; set; }
    public int Quantity { get; set; }
    public string Region { get; set; } = string.Empty;
}

public sealed class ParsedCsvResult
{
    public SalesRecord[] Records { get; set; } = Array.Empty<SalesRecord>();
    public string[] Headers { get; set; } = Array.Empty<string>();
    public string[] Errors { get; set; } = Array.Empty<string>();
    public bool IsValid => Errors.Length == 0;
}

public sealed class MonthlyRevenueReport
{
    public string[] Regions { get; set; } = Array.Empty<string>();
    public decimal[,] RevenueMatrix { get; set; } = new decimal[0, 0];
    public int[,] OrderCountMatrix { get; set; } = new int[0, 0];
    public decimal[] MonthlyTotals { get; set; } = Array.Empty<decimal>();
    public decimal[] RegionTotals { get; set; } = Array.Empty<decimal>();
    public decimal GrandTotal { get; set; }
    public int PeakMonth { get; set; } // 1-12, or 0 when no record had a recognized region.
}

public interface IReportLogger
{
    void LogInformation(string message, params object[] args);
    void LogWarning(string message, params object[] args);
}
```

### Key Observations

- Arrays are useful at API boundaries; for example, `File.ReadAllLines` returns a `string[]`. It reads the full file into memory, so use a streaming API for very large input.
- The parser preallocates an upper-bound array and calls `Array.Resize` once to remove unused slots. `Array.Resize` allocates and copies when the length changes; it doesn't resize the original object in place.
- `Array.Empty<T>()` provides a reusable empty array and is convenient for empty-array defaults. It avoids repeatedly creating an empty array when that value is reused; `new T[0]` is still valid, just usually unnecessary for this case.
- The revenue matrix is naturally rectangular: every region has 12 month columns. A jagged array would be useful only if row lengths needed to differ.
- `Array.IndexOf` is linear but appropriate for a small fixed list of four regions. For larger or repeated lookups, use a dictionary or another index.
- The top-orders method sorts a copy so it doesn't modify the caller's array. Elements with equal amounts aren't guaranteed to preserve their original relative order.
- `ReadOnlyMemory<SalesRecord>` gives each asynchronous batch callback a view without copying the batch into a new array. The view is read-only through that parameter; other references can still mutate the underlying array.
- The batch loop handles a final batch smaller than `MaxBatchSize` and passes the cancellation token to the callback.

## Key Terms Summary

| Term | Definition |
|---|---|
| Array | A fixed-length, ordered sequence of elements with one element type. |
| Single-dimensional array | A linear array accessed with one index, such as `int[]`. |
| Multidimensional array | A rectangular array accessed with multiple indices, such as `int[,]`. |
| Jagged array | An array whose elements are arrays, such as `int[][]`; inner arrays can have different lengths. |
| Index operator | `[]`, used to read or assign an array element. |
| Index-from-end | `^`; `^1` refers to the last element. |
| Range | `..`; for arrays, returns a new shallow copy of the selected range. |
| `Array.Resize` | Allocates a differently sized one-dimensional array and updates a reference to it. |
| `Array.Empty<T>()` | Returns an empty array of `T`, useful as a reusable empty value. |
| Shallow copy | Copies the array structure and element values/references, not referenced objects. |
| Binary search | O(log N) search on an array sorted using the same comparer. |
| `Span<T>` | A ref-like view over contiguous memory with restricted lifetime. |
| `Memory<T>` | A memory view that can be stored and used across asynchronous boundaries. |

## Further Reading

- [Arrays — C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/arrays) — array types, initialization, lengths, and indexing.
- [Indices and ranges — C# tutorial](https://learn.microsoft.com/en-us/dotnet/csharp/tutorials/ranges-indexes) — inclusive starts, exclusive ends, array-copy behavior, and non-copying `Span<T>`/`Memory<T>` ranges.
- [What's new in C# 13](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-13) — ref-struct locals in async methods and their suspension-point restrictions.
- [`Array.Sort` API](https://learn.microsoft.com/en-us/dotnet/api/system.array.sort?view=net-10.0) — sorting arrays and using comparers.
- [`Array.BinarySearch` API](https://learn.microsoft.com/en-us/dotnet/api/system.array.binarysearch?view=net-10.0) — search results and sorted-array requirements.
- [`Array.Resize` API](https://learn.microsoft.com/en-us/dotnet/api/system.array.resize?view=net-10.0) — resizing behavior and element copying.
- [`Array.Empty<T>` API](https://learn.microsoft.com/en-us/dotnet/api/system.array.empty?view=net-10.0) — returning an empty array.
- [Memory and spans](https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/) — `Span<T>` and `Memory<T>` views over contiguous memory.
