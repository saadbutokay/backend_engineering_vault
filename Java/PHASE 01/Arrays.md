## Overview

Arrays are the most fundamental data structure in Java. They are fixed-size, contiguous blocks of memory that store elements of a single type. Every other collection in Java — `ArrayList`, `HashMap`, `HashSet` — is built on top of arrays internally. Understanding arrays is not optional. Even when you use higher-level collections in daily work, you need to understand the underlying array mechanics to reason about performance, memory layout, and the behavior of the collections framework. This section covers everything from basic declaration to the `Arrays` utility class and varargs.

---

## Core Concepts

### Array Declaration and Initialization

An array is a container object that holds a fixed number of values of a single type. The type can be a primitive or a reference type.

**Declaration syntax:**

```java
// Preferred style
int[] numbers;
String[] names;
Transaction[] transactions;

// C-style (legal but discouraged in Java)
int numbers[];
```

The preferred style places the brackets after the type (`int[]`) rather than after the variable name (`int numbers[]`). This makes the type clear: the variable is of type "array of int," not "int that happens to be an array."

**Initialization methods:**

```java
// 1. Declare and allocate (elements get default values)
int[] numbers = new int[5];
// numbers = [0, 0, 0, 0, 0]

String[] names = new String[3];
// names = [null, null, null]

boolean[] flags = new boolean[4];
// flags = [false, false, false, false]

// 2. Array literal (declare and initialize in one step)
int[] primes = {2, 3, 5, 7, 11};
String[] currencies = {"USD", "EUR", "GBP", "JPY"};

// 3. Explicit new with literal (required when assigning to an existing variable)
int[] scores;
scores = new int[]{85, 92, 78, 95};
// Note: you cannot use the shorthand {85, 92, 78, 95} here

// 4. Anonymous array (useful for passing to methods)
processTransactions(new Transaction[]{tx1, tx2, tx3});
```

**Default values by type:**

| Type | Default Value |
|------|--------------|
| `byte`, `short`, `int`, `long` | `0` |
| `float`, `double` | `0.0` |
| `char` | `'\u0000'` (null character) |
| `boolean` | `false` |
| All reference types | `null` |

### Array Access

Arrays are zero-indexed. The first element is at index 0, the last at index `length - 1`.

```java
int[] amounts = {100, 250, 500, 1000, 2500};

// Reading
int first = amounts[0];    // 100
int last = amounts[4];     // 2500
int middle = amounts[2];   // 500

// Writing
amounts[1] = 300;          // amounts is now {100, 300, 500, 1000, 2500}

// Out of bounds
int invalid = amounts[5];  // Throws ArrayIndexOutOfBoundsException
int negative = amounts[-1]; // Throws ArrayIndexOutOfBoundsException
```

**ArrayIndexOutOfBoundsException** is one of the most common runtime errors in Java. It occurs when you access an index less than 0 or greater than or equal to the array length. Always validate indices before access, especially when the index comes from user input or external data.

### Array Length

The `length` field is a public final property of every array. It is not a method — there are no parentheses.

```java
int[] data = new int[100];
System.out.println(data.length);  // 100

String[] names = {"Alice", "Bob", "Charlie"};
System.out.println(names.length);  // 3

// Common pattern: iterating with length
for (int i = 0; i < names.length; i++) {
    System.out.println(i + ": " + names[i]);
}
```

**Do not confuse:**

| Type | Length accessor | Example |
|------|----------------|---------|
| Array | `.length` (field) | `arr.length` |
| String | `.length()` (method) | `str.length()` |
| Collection | `.size()` (method) | `list.size()` |

This inconsistency is a historical artifact of Java's design. It is a common source of errors for beginners.

### Memory Layout

Arrays in Java are objects. They are allocated on the heap, even when they contain primitives. The array variable on the stack holds a reference (pointer) to the heap-allocated array object.

```
Stack                    Heap
─────                    ────
int[] arr ──────────→   [Array Object Header]
                         [length = 5]
                         [0] = 10
                         [1] = 20
                         [2] = 30
                         [3] = 40
                         [4] = 50
```

Key implications:

- **Arrays are reference types.** Assigning one array variable to another copies the reference, not the data. Both variables point to the same array.
- **Array elements are stored contiguously in memory.** This makes index-based access O(1) and is cache-friendly for sequential iteration.
- **The array object includes a header** with metadata (class pointer, length, GC flags). This adds a small overhead beyond the raw element storage.

```java
int[] original = {1, 2, 3, 4, 5};
int[] reference = original;  // Both point to the same array

reference[0] = 99;
System.out.println(original[0]);  // 99 (original is modified!)

// To create an independent copy:
int[] copy = Arrays.copyOf(original, original.length);
copy[0] = 100;
System.out.println(original[0]);  // 99 (original is NOT modified)
```

### Multi-Dimensional Arrays

Java supports multi-dimensional arrays, which are actually arrays of arrays. This is an important distinction from languages like C where multi-dimensional arrays are contiguous blocks.

**Two-dimensional arrays:**

```java
// Declaration and allocation
int[][] matrix = new int[3][4];  // 3 rows, 4 columns
// All elements initialized to 0

// Literal initialization
int[][] grid = {
    {1, 2, 3, 4},
    {5, 6, 7, 8},
    {9, 10, 11, 12}
};

// Access
int value = grid[1][2];  // 7 (row 1, column 2)
grid[0][0] = 99;         // Modify top-left element

// Dimensions
System.out.println(grid.length);      // 3 (number of rows)
System.out.println(grid[0].length);   // 4 (number of columns in row 0)
```

**Jagged arrays (rows of different lengths):**

Because Java's 2D arrays are arrays of arrays, each row can have a different length.

```java
int[][] jagged = new int[3][];
jagged[0] = new int[2];   // Row 0 has 2 elements
jagged[1] = new int[5];   // Row 1 has 5 elements
jagged[2] = new int[1];   // Row 2 has 1 element

// Literal jagged array
int[][] triangle = {
    {1},
    {1, 1},
    {1, 2, 1},
    {1, 3, 3, 1},
    {1, 4, 6, 4, 1}
};
```

**Iterating over a 2D array:**

```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Traditional for loop
for (int row = 0; row < matrix.length; row++) {
    for (int col = 0; col < matrix[row].length; col++) {
        System.out.printf("%4d", matrix[row][col]);
    }
    System.out.println();
}

// Enhanced for loop
for (int[] row : matrix) {
    for (int value : row) {
        System.out.printf("%4d", value);
    }
    System.out.println();
}
```

**Three-dimensional arrays (rare but possible):**

```java
int[][][] cube = new int[2][3][4];  // 2 layers, 3 rows, 4 columns
cube[0][1][2] = 42;
```

In practice, arrays beyond two dimensions are rare. If you need complex multi-dimensional data, consider using a class to represent the structure explicitly.

### The Arrays Utility Class

`java.util.Arrays` provides static methods for common array operations. These methods are optimized and should always be preferred over manual implementations.

**Sorting:**

```java
int[] numbers = {5, 2, 8, 1, 9, 3};

// Sort entire array (ascending)
Arrays.sort(numbers);
// numbers = [1, 2, 3, 5, 8, 9]

// Sort a range [fromIndex, toIndex)
Arrays.sort(numbers, 1, 4);
// Sorts elements at indices 1, 2, 3

// Sort in descending order (requires Integer[], not int[])
Integer[] boxed = {5, 2, 8, 1, 9, 3};
Arrays.sort(boxed, Collections.reverseOrder());
// boxed = [9, 8, 5, 3, 2, 1]

// Sort objects with a custom comparator
Transaction[] txns = getTransactions();
Arrays.sort(txns, Comparator.comparing(Transaction::getAmount));
Arrays.sort(txns, Comparator.comparing(Transaction::getDate).reversed());
```

**Searching:**

```java
int[] sorted = {10, 20, 30, 40, 50, 60, 70};

// Binary search (array MUST be sorted)
int index = Arrays.binarySearch(sorted, 40);
// index = 3

int notFound = Arrays.binarySearch(sorted, 35);
// notFound = -4 (negative value = insertion point)
// Insertion point = -(notFound) - 1 = 3

// Binary search on a range
int rangeIndex = Arrays.binarySearch(sorted, 2, 5, 50);
// Searches only indices 2 through 4
```

**Filling:**

```java
int[] data = new int[10];

// Fill entire array
Arrays.fill(data, 42);
// data = [42, 42, 42, 42, 42, 42, 42, 42, 42, 42]

// Fill a range
Arrays.fill(data, 2, 5, 99);
// data = [42, 42, 99, 99, 99, 42, 42, 42, 42, 42]
```

**Copying:**

```java
int[] original = {1, 2, 3, 4, 5};

// Copy entire array
int[] fullCopy = Arrays.copyOf(original, original.length);
// fullCopy = [1, 2, 3, 4, 5]

// Copy with new size (larger — padded with defaults)
int[] expanded = Arrays.copyOf(original, 8);
// expanded = [1, 2, 3, 4, 5, 0, 0, 0]

// Copy with new size (smaller — truncated)
int[] truncated = Arrays.copyOf(original, 3);
// truncated = [1, 2, 3]

// Copy a range
int[] range = Arrays.copyOfRange(original, 1, 4);
// range = [2, 3, 4] (indices 1, 2, 3)

// System.arraycopy (lower-level, faster for large arrays)
int[] source = {10, 20, 30, 40, 50};
int[] dest = new int[5];
System.arraycopy(source, 1, dest, 0, 3);
// dest = [20, 30, 40, 0, 0]
// Parameters: source, srcPos, dest, destPos, length
```

**Comparison:**

```java
int[] a = {1, 2, 3};
int[] b = {1, 2, 3};
int[] c = {1, 2, 4};

// Compare contents (not references)
Arrays.equals(a, b);  // true
Arrays.equals(a, c);  // false

// Compare ranges
Arrays.equals(a, 0, 2, b, 0, 2);  // true (compare first 2 elements)

// Compare multi-dimensional arrays
int[][] m1 = {{1, 2}, {3, 4}};
int[][] m2 = {{1, 2}, {3, 4}};
Arrays.equals(m1, m2);       // false (compares references of inner arrays)
Arrays.deepEquals(m1, m2);   // true (compares contents recursively)
```

**String representation:**

```java
int[] numbers = {1, 2, 3, 4, 5};

System.out.println(numbers);
// [I@1b6d3586 (useless default toString)

System.out.println(Arrays.toString(numbers));
// [1, 2, 3, 4, 5]

int[][] matrix = {{1, 2}, {3, 4}};
System.out.println(Arrays.toString(matrix));
// [[I@4554617c, [I@74a14482] (still useless for 2D)

System.out.println(Arrays.deepToString(matrix));
// [[1, 2], [3, 4]]
```

**Converting to List:**

```java
String[] names = {"Alice", "Bob", "Charlie"};

// Fixed-size list backed by the array
List<String> list = Arrays.asList(names);
list.set(0, "Alicia");
System.out.println(names[0]);  // "Alicia" (modifying the list modifies the array)

// WARNING: The returned list is fixed-size
list.add("Dave");  // Throws UnsupportedOperationException!

// To get a mutable list:
List<String> mutable = new ArrayList<>(Arrays.asList(names));
mutable.add("Dave");  // Works

// Modern alternative (Java 9+):
List<String> immutable = List.of(names);  // Truly immutable
```

**Parallel operations (Java 8+):**

```java
int[] largeArray = new int[10_000_000];
// ... populate array ...

// Parallel sort (uses Fork/Join pool for large arrays)
Arrays.parallelSort(largeArray);

// Parallel prefix (running computation)
int[] values = {1, 2, 3, 4, 5};
Arrays.parallelPrefix(values, Integer::sum);
// values = [1, 3, 6, 10, 15] (running sum)

// Parallel setAll
int[] squares = new int[100];
Arrays.parallelSetAll(squares, i -> i * i);
// squares = [0, 1, 4, 9, 16, ...]
```

### Array Limitations

Arrays have significant limitations that motivate the use of the Collections Framework (covered in 01.06).

**1. Fixed size:**

Once created, an array's length cannot change. To "resize" an array, you must create a new array and copy the elements.

```java
int[] original = {1, 2, 3};
// Cannot do: original.add(4);
// Must do:
int[] resized = Arrays.copyOf(original, original.length + 1);
resized[3] = 4;
```

This is exactly what `ArrayList` does internally — it maintains a backing array and resizes it (by creating a new, larger array and copying elements) when capacity is exceeded.

**2. No generics for primitives:**

You cannot create `List<int>`. Collections require wrapper types (`List<Integer>`). Arrays are the only way to store primitives without boxing overhead.

```java
int[] primitiveArray = new int[1_000_000];  // 4 MB of memory
Integer[] boxedArray = new Integer[1_000_000];  // ~16 MB+ (object overhead per element)
```

For performance-critical numerical computation, primitive arrays are significantly more memory-efficient and faster than collections of wrapper types.

**3. No built-in methods:**

Arrays do not have methods like `add()`, `remove()`, `contains()`, or `indexOf()`. You must use the `Arrays` utility class or write your own logic.

**4. Covariance (a design flaw):**

Arrays are covariant in Java, which means `String[]` is considered a subtype of `Object[]`. This allows type-unsafe operations that compile but fail at runtime.

```java
Object[] objects = new String[3];  // Legal due to covariance
objects[0] = "Hello";              // Fine
objects[1] = 42;                   // Compiles! But throws ArrayStoreException at runtime
```

Collections do not have this problem because generics are invariant. `List<String>` is NOT a subtype of `List<Object>`. This is one reason collections are preferred over arrays in modern Java.

### Varargs (Variable-Length Arguments)

Varargs allow a method to accept zero or more arguments of a specified type. Internally, the compiler converts varargs to an array.

```java
// Declaration: Type... parameterName (must be the last parameter)
public BigDecimal sum(BigDecimal... amounts) {
    BigDecimal total = BigDecimal.ZERO;
    for (BigDecimal amount : amounts) {
        total = total.add(amount);
    }
    return total;
}

// Usage:
sum();                                          // 0 arguments
sum(new BigDecimal("100"));                     // 1 argument
sum(new BigDecimal("100"), new BigDecimal("200")); // 2 arguments
sum(new BigDecimal("100"), new BigDecimal("200"), new BigDecimal("300")); // 3 arguments
```

**Rules:**

- A method can have only one varargs parameter.
- The varargs parameter must be the last parameter in the method signature.
- Inside the method, the varargs parameter is a regular array. You can use `.length`, iterate with for-each, and access by index.
- You can pass an explicit array instead of individual arguments.

```java
public void log(String level, String... messages) {
    System.out.println("[" + level + "]");
    for (String msg : messages) {
        System.out.println("  " + msg);
    }
}

log("INFO", "Server started", "Listening on port 8080");

// Equivalent to passing an array:
log("ERROR", new String[]{"Connection failed", "Retrying in 5s"});
```

**Varargs pitfalls:**

```java
// Pitfall 1: Null varargs
public void printAll(String... items) {
    for (String item : items) {  // NullPointerException if items is null!
        System.out.println(item);
    }
}
printAll(null);  // Throws NPE — items is null, not an empty array

// Fix:
public void printAll(String... items) {
    if (items == null) return;
    for (String item : items) {
        System.out.println(item);
    }
}

// Pitfall 2: Overloading ambiguity
public void process(int... values) { }
public void process(long... values) { }
// process(1, 2, 3) — ambiguous! Both match.
```

### Command-Line Arguments

The `main` method receives command-line arguments as a `String[]`.

```java
public class App {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println("Usage: java App <name> <amount>");
            return;
        }

        String name = args[0];
        BigDecimal amount = new BigDecimal(args[1]);
        System.out.println("Processing $" + amount + " for " + name);
    }
}
```

```bash
java App Alice 1500.75
# Output: Processing $1500.75 for Alice
```

All command-line arguments are strings. You must parse them to the appropriate type. In production applications, you will use a library like `picocli` or Spring Boot's `@Value` and `@ConfigurationProperties` instead of parsing `args` manually.

---

## Code Examples

**A complete example demonstrating array operations in a financial context:**

```java
package com.example.arrays;

import java.math.BigDecimal;
import java.util.Arrays;
import java.util.Comparator;

public class ArrayDemo {

    public static void main(String[] args) {
        // 1. Basic array of transaction amounts
        double[] amounts = {150.00, 42.50, 1200.00, 89.99, 3500.75, 15.00};

        System.out.println("Original: " + Arrays.toString(amounts));
        System.out.println("Count: " + amounts.length);

        // 2. Sort and search
        Arrays.sort(amounts);
        System.out.println("Sorted: " + Arrays.toString(amounts));

        int index = Arrays.binarySearch(amounts, 1200.00);
        System.out.println("1200.00 found at index: " + index);

        // 3. Statistics
        double sum = 0;
        double min = Double.MAX_VALUE;
        double max = Double.MIN_VALUE;

        for (double amount : amounts) {
            sum += amount;
            if (amount < min) min = amount;
            if (amount > max) max = amount;
        }

        System.out.printf("Sum: $%.2f%n", sum);
        System.out.printf("Min: $%.2f%n", min);
        System.out.printf("Max: $%.2f%n", max);
        System.out.printf("Avg: $%.2f%n", sum / amounts.length);

        // 4. Copy and resize
        double[] expanded = Arrays.copyOf(amounts, amounts.length + 2);
        expanded[amounts.length] = 5000.00;
        expanded[amounts.length + 1] = 75.25;
        System.out.println("Expanded: " + Arrays.toString(expanded));

        // 5. 2D array: monthly expenses by category
        // Rows: Jan, Feb, Mar | Columns: Rent, Food, Transport, Utilities
        double[][] monthlyExpenses = {
            {1200, 450, 120, 85},
            {1200, 520, 95,  92},
            {1200, 380, 140, 78}
        };

        System.out.println("\nMonthly Expenses:");
        String[] categories = {"Rent", "Food", "Transport", "Utilities"};
        String[] months = {"Jan", "Feb", "Mar"};

        for (int m = 0; m < monthlyExpenses.length; m++) {
            double monthTotal = 0;
            System.out.printf("%s: ", months[m]);
            for (int c = 0; c < monthlyExpenses[m].length; c++) {
                monthTotal += monthlyExpenses[m][c];
            }
            System.out.printf("Total = $%.2f%n", monthTotal);
        }

        // 6. Array of objects
        String[] currencies = {"JPY", "USD", "EUR", "GBP", "CHF"};
        Arrays.sort(currencies);
        System.out.println("\nCurrencies sorted: " + Arrays.toString(currencies));

        // 7. Varargs usage
        BigDecimal total = calculateTotal(
            new BigDecimal("100.00"),
            new BigDecimal("250.50"),
            new BigDecimal("75.25")
        );
        System.out.println("Total: $" + total);
    }

    public static BigDecimal calculateTotal(BigDecimal... amounts) {
        BigDecimal total = BigDecimal.ZERO;
        for (BigDecimal amount : amounts) {
            total = total.add(amount);
        }
        return total;
    }
}
```

**Array-based implementation of a simple ledger:**

```java
public class SimpleLedger {
    private String[] descriptions;
    private BigDecimal[] amounts;
    private int count;

    public SimpleLedger(int capacity) {
        descriptions = new String[capacity];
        amounts = new BigDecimal[capacity];
        count = 0;
    }

    public void addEntry(String description, BigDecimal amount) {
        if (count == descriptions.length) {
            // Resize: double the capacity
            descriptions = Arrays.copyOf(descriptions, count * 2);
            amounts = Arrays.copyOf(amounts, count * 2);
        }
        descriptions[count] = description;
        amounts[count] = amount;
        count++;
    }

    public BigDecimal getBalance() {
        BigDecimal balance = BigDecimal.ZERO;
        for (int i = 0; i < count; i++) {
            balance = balance.add(amounts[i]);
        }
        return balance;
    }

    public void printEntries() {
        for (int i = 0; i < count; i++) {
            System.out.printf("%-30s $%10.2f%n", descriptions[i], amounts[i]);
        }
        System.out.println("─".repeat(43));
        System.out.printf("%-30s $%10.2f%n", "BALANCE", getBalance());
    }

    public static void main(String[] args) {
        SimpleLedger ledger = new SimpleLedger(4);
        ledger.addEntry("Salary deposit", new BigDecimal("5000.00"));
        ledger.addEntry("Rent payment", new BigDecimal("-1500.00"));
        ledger.addEntry("Grocery store", new BigDecimal("-127.43"));
        ledger.addEntry("Freelance income", new BigDecimal("800.00"));
        ledger.addEntry("Electric bill", new BigDecimal("-85.50"));  // Triggers resize

        ledger.printEntries();
    }
}
```

This is essentially what `ArrayList` does internally. Understanding this resize-and-copy pattern is critical for reasoning about collection performance.

---

## Important Notes

- Arrays are objects in Java. They live on the heap. The variable on the stack holds a reference. Assigning one array variable to another does not copy the data — it copies the reference. Both variables point to the same array. Use `Arrays.copyOf()` to create an independent copy.
- The `length` field is a property, not a method. Do not write `arr.length()`. This is a common error when switching between arrays (`.length`), strings (`.length()`), and collections (`.size()`).
- Array indices start at 0. The last valid index is `length - 1`. Accessing `arr[length]` throws `ArrayIndexOutOfBoundsException`. This off-by-one error is one of the most frequent bugs in all of programming.
- Arrays are fixed-size. You cannot add or remove elements. If you need a resizable sequence, use `ArrayList`. If you need a fixed-size sequence with maximum performance and minimal memory overhead, use an array.
- Primitive arrays (`int[]`, `double[]`, `long[]`) are significantly more memory-efficient than their boxed equivalents (`Integer[]`, `Double[]`, `Long[]`). Each boxed element carries object header overhead (12-16 bytes per object on a 64-bit JVM). For a million-element array, this difference is roughly 4 MB vs 20 MB. In fintech systems processing large numerical datasets, this matters.
- `Arrays.asList()` returns a fixed-size list backed by the original array. You cannot add or remove elements from it. To get a mutable list, wrap it in `new ArrayList<>(Arrays.asList(...))`. To get a truly immutable list, use `List.of()` (Java 9+).
- `Arrays.sort()` uses dual-pivot quicksort for primitive arrays and TimSort for object arrays. Both are O(n log n) average case. The sort is not stable for primitives (equal elements may be reordered) but is stable for objects (equal elements retain their relative order).
- `Arrays.binarySearch()` requires the array to be sorted. Searching an unsorted array produces undefined results (not an exception — just wrong answers). Always sort before searching.
- Varargs are syntactic sugar for arrays. The compiler converts `method(String... args)` to `method(String[] args)` at the call site. Inside the method, the parameter is a regular array. Be aware of the null pitfall: passing `null` explicitly to a varargs parameter results in a null array, not an array containing null.
- Array covariance (`String[]` is a subtype of `Object[]`) is a design flaw in Java that predates generics. It allows type-unsafe assignments that compile but throw `ArrayStoreException` at runtime. Collections with generics do not have this problem. Prefer collections over arrays in public APIs.
- Multi-dimensional arrays in Java are arrays of arrays, not contiguous blocks. This means rows can have different lengths (jagged arrays) and each row is a separate heap allocation. For performance-critical matrix operations, a flat 1D array with manual index calculation (`row * columns + col`) is faster due to better cache locality.
- The `System.arraycopy()` method is the fastest way to copy array elements. It is a native method that uses optimized memory copy instructions. `Arrays.copyOf()` and `Arrays.copyOfRange()` use `System.arraycopy()` internally.

---

## Practice

1. Create an array of 10 `double` values representing daily stock prices. Write code to find the minimum price, maximum price, and average price. Then sort the array and use binary search to find a specific price.

2. Write a method `int[] removeElement(int[] arr, int index)` that returns a new array with the element at the given index removed. The original array must not be modified. Handle edge cases: empty array, index out of bounds.

3. Create a 2D array representing a 12-month budget with 5 expense categories. Write code to calculate: total spending per month, total spending per category, and the month with the highest spending.

4. Write a varargs method `BigDecimal average(BigDecimal... values)` that calculates the average of any number of `BigDecimal` values. Handle the edge case of zero arguments. Handle null values gracefully.

5. Demonstrate the array covariance pitfall. Create an `Object[]` reference pointing to a `String[]`. Attempt to store an `Integer` in it. Catch the `ArrayStoreException` and explain in a comment why this happens.

6. Implement a simple dynamic array class (a primitive `ArrayList` for `int`). It should support `add(int value)`, `get(int index)`, `size()`, and automatic resizing when capacity is exceeded. Track the number of resize operations and print them.

7. In your Obsidian vault, create a comparison table: Array vs ArrayList. Compare them on: size flexibility, primitive support, performance, memory overhead, available methods, and type safety.

---

## References

- Java Language Specification — Chapter 10 (Arrays): https://docs.oracle.com/javase/specs/jls/se21/html/jls-10.html
- Arrays API Documentation: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html
- Oracle Tutorial — Arrays: https://docs.oracle.com/javase/tutorial/java/nutsandbolts/arrays.html
- "Effective Java" by Joshua Bloch — Item 28 (Prefer lists to arrays)
