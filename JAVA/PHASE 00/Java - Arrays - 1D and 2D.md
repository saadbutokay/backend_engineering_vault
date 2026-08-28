---
title: "Java - Arrays - 1D and 2D"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - arrays
  - data-structures
status: "not-started"
---

# Java - Arrays - 1D and 2D

> [!abstract] Overview
> An array is a fixed-size, ordered collection of elements of the same data type stored in contiguous memory locations. Arrays are the most fundamental data structure in Java and the building block for more advanced structures like ArrayLists, HashMaps, and database result sets. In backend engineering, arrays appear in request payloads, batch processing, matrix computations, caching layers, and configuration management.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]
> - [[Java - Operators - Arithmetic Relational Logical]]
> - [[Java - Loops - For While Do-While]]

---

## Theory

### What is an Array?

An array is a container object that holds a fixed number of values of a single type. Once you create an array, its size cannot change. Each value in the array is called an **element**, and each element is accessed by its **index**, which is a zero-based integer.

```java
int[] scores = {85, 92, 78, 95, 88};
//              [0]  [1]  [2]  [3]  [4]
// 5 elements, indices 0 through 4
```

Key properties of Java arrays:

- **Fixed size**: The length is set at creation and cannot be changed. If you need a resizable collection, you use `ArrayList` (covered in Phase 1).
- **Homogeneous**: All elements must be the same type. You cannot mix `int` and `String` in the same array (unless the array type is `Object`, which is rarely a good idea).
- **Zero-indexed**: The first element is at index 0, the second at index 1, and the last at index `length - 1`.
- **Contiguous memory**: Elements are stored next to each other in memory, which makes access by index extremely fast (O(1) time complexity).
- **Reference type**: Even arrays of primitives (like `int[]`) are objects in Java. They live on the heap, and the variable holds a reference to the array object.

### Why Do Arrays Exist?

Before arrays, if you wanted to store 100 student scores, you would need 100 separate variables:

```java
int score1 = 85;
int score2 = 92;
int score3 = 78;
// ... 97 more variables
```

This is obviously impractical. Arrays solve this by letting you store all 100 values in a single variable and access them by index. More importantly, arrays enable **algorithmic processing**. You can write a single loop that calculates the average of any number of scores, regardless of whether there are 5 or 5 million.

In backend systems, arrays (and their dynamic cousins, ArrayLists) are the default way to handle collections of data. When a REST API returns a list of users, the JSON array `[{"name": "Alice"}, {"name": "Bob"}]` is deserialized into a Java array or List on the server side. When you query a database for all orders placed today, the result is loaded into an array-like structure in memory.

### How Does It Work Internally?

When you create an array, the JVM performs the following steps:

1. **Allocates contiguous memory on the heap**: If you create `int[5]`, the JVM finds a block of memory large enough to hold 5 integers (5 x 4 bytes = 20 bytes) plus a small header for the array object (which stores the length and type information).

2. **Initializes all elements to default values**: Unlike local variables (which must be explicitly initialized before use), array elements are automatically set to their type's default value. For `int`, the default is `0`. For `double`, it is `0.0`. For `boolean`, it is `false`. For reference types like `String`, it is `null`.

3. **Returns a reference**: The variable you assign the array to holds a reference (memory address) pointing to the array object on the heap.

```java
int[] numbers = new int[5];
// 'numbers' is on the stack, holding a reference.
// The actual array [0, 0, 0, 0, 0] is on the heap.
```

**Memory layout of a 1D array:**

```
Stack                    Heap
+-----------+           +------+------+------+------+------+------+
| numbers   |---------> | header |  0   |  0   |  0   |  0   |  0   |
| (ref)     |           | len=5  | [0]  | [1]  | [2]  | [3]  | [4]  |
+-----------+           +------+------+------+------+------+------+
```

**Memory layout of a 2D array:**

A 2D array in Java is actually an **array of arrays**. The outer array holds references to inner arrays. This means the inner arrays do not have to be the same length (jagged arrays).

```
Stack                    Heap
+-----------+           +------+
| matrix    |---------> | ref0 |-----> [1, 2, 3]
| (ref)     |           | ref1 |-----> [4, 5, 6]
+-----------+           | ref2 |-----> [7, 8, 9]
                        +------+
```

Because array access is a simple arithmetic calculation (`base_address + index * element_size`), reading or writing an element by index takes O(1) time regardless of the array's size. This makes arrays the fastest data structure for random access.

> [!tip] Key Insight
> Arrays are objects in Java, even when they hold primitives. This means they have a `.length` property (not a method, no parentheses), they can be assigned to `Object` variables, and they inherit methods from `Object` like `toString()` and `equals()`. However, the default `equals()` checks reference equality, not content equality. To compare array contents, use `Arrays.equals()`.

---

## Syntax and Basic Examples

### Example 1: Declaring and initializing 1D arrays

```java
import java.util.Arrays;

public class ArrayBasics {
    public static void main(String[] args) {
        // Method 1: Declare and initialize in one line (literal syntax)
        int[] scores = {85, 92, 78, 95, 88};

        // Method 2: Declare with a size, elements get default values
        int[] zeros = new int[5];  // [0, 0, 0, 0, 0]

        // Method 3: Declare first, initialize later
        String[] cities;
        cities = new String[]{"Dhaka", "Chittagong", "Sylhet"};

        // Accessing elements by index
        System.out.println("First score: " + scores[0]);    // 85
        System.out.println("Last score: " + scores[4]);     // 88
        System.out.println("Array length: " + scores.length); // 5 (property, not method!)

        // Modifying elements
        scores[2] = 100;
        System.out.println("Updated scores: " + Arrays.toString(scores));
        // [85, 92, 100, 95, 88]

        // Default values for different types
        int[] intArray = new int[3];        // [0, 0, 0]
        double[] doubleArray = new double[3]; // [0.0, 0.0, 0.0]
        boolean[] boolArray = new boolean[3]; // [false, false, false]
        String[] stringArray = new String[3]; // [null, null, null]

        System.out.println("Int defaults: " + Arrays.toString(intArray));
        System.out.println("String defaults: " + Arrays.toString(stringArray));
    }
}
```

**Output:**
```
First score: 85
Last score: 88
Array length: 5
Updated scores: [85, 92, 100, 95, 88]
Int defaults: [0, 0, 0]
String defaults: [null, null, null]
```

### Example 2: Iterating over arrays

```java
public class ArrayIteration {
    public static void main(String[] args) {
        String[] frameworks = {"Spring Boot", "Hibernate", "Quarkus", "Micronaut"};

        // Method 1: Traditional for loop (when you need the index)
        System.out.println("--- Indexed for loop ---");
        for (int i = 0; i < frameworks.length; i++) {
            System.out.println("Index " + i + ": " + frameworks[i]);
        }

        // Method 2: Enhanced for-each loop (when you only need the value)
        System.out.println("\n--- For-each loop ---");
        for (String framework : frameworks) {
            System.out.println(framework);
        }

        // Method 3: Reverse iteration (only possible with indexed for loop)
        System.out.println("\n--- Reverse ---");
        for (int i = frameworks.length - 1; i >= 0; i--) {
            System.out.println(frameworks[i]);
        }
    }
}
```

**Output:**
```
--- Indexed for loop ---
Index 0: Spring Boot
Index 1: Hibernate
Index 2: Quarkus
Index 3: Micronaut

--- For-each loop ---
Spring Boot
Hibernate
Quarkus
Micronaut

--- Reverse ---
Micronaut
Quarkus
Hibernate
Spring Boot
```

### Example 3: 2D arrays

```java
import java.util.Arrays;

public class TwoDArrayDemo {
    public static void main(String[] args) {
        // Method 1: Literal initialization
        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        // Accessing elements: matrix[row][column]
        System.out.println("Element at [1][2]: " + matrix[1][2]);  // 6
        System.out.println("Number of rows: " + matrix.length);     // 3
        System.out.println("Columns in row 0: " + matrix[0].length); // 3

        // Printing a 2D array
        System.out.println("\n--- Matrix ---");
        for (int row = 0; row < matrix.length; row++) {
            for (int col = 0; col < matrix[row].length; col++) {
                System.out.printf("%4d", matrix[row][col]);
            }
            System.out.println();
        }

        // Method 2: Create with dimensions (all zeros)
        int[][] grid = new int[4][4];
        grid[0][0] = 1;
        grid[3][3] = 16;

        // Method 3: Jagged array (rows of different lengths)
        int[][] jagged = new int[3][];
        jagged[0] = new int[]{1, 2};
        jagged[1] = new int[]{3, 4, 5, 6};
        jagged[2] = new int[]{7};

        System.out.println("\n--- Jagged Array ---");
        for (int[] row : jagged) {
            System.out.println(Arrays.toString(row));
        }

        // Deep printing for 2D arrays
        System.out.println("\n--- Deep toString ---");
        System.out.println(Arrays.deepToString(matrix));
        // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    }
}
```

**Output:**
```
Element at [1][2]: 6
Number of rows: 3
Columns in row 0: 3

--- Matrix ---
   1   2   3
   4   5   6
   7   8   9

--- Jagged Array ---
[1, 2]
[3, 4, 5, 6]
[7]

--- Deep toString ---
[[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

### Example 4: Common array operations using `java.util.Arrays`

```java
import java.util.Arrays;

public class ArrayOperations {
    public static void main(String[] args) {
        int[] numbers = {45, 12, 78, 3, 56, 91, 23};

        // Sorting
        Arrays.sort(numbers);
        System.out.println("Sorted: " + Arrays.toString(numbers));
        // [3, 12, 23, 45, 56, 78, 91]

        // Binary search (array MUST be sorted first)
        int index = Arrays.binarySearch(numbers, 56);
        System.out.println("56 found at index: " + index);  // 4

        int notFound = Arrays.binarySearch(numbers, 50);
        System.out.println("50 search result: " + notFound);
        // Negative number means not found. The value is -(insertion point) - 1.

        // Filling
        int[] filled = new int[5];
        Arrays.fill(filled, 7);
        System.out.println("Filled: " + Arrays.toString(filled));
        // [7, 7, 7, 7, 7]

        // Copying
        int[] original = {1, 2, 3, 4, 5};
        int[] copy = Arrays.copyOf(original, original.length);
        int[] partial = Arrays.copyOfRange(original, 1, 4);  // indices 1, 2, 3
        System.out.println("Full copy: " + Arrays.toString(copy));       // [1, 2, 3, 4, 5]
        System.out.println("Partial copy: " + Arrays.toString(partial)); // [2, 3, 4]

        // Comparing
        int[] a = {1, 2, 3};
        int[] b = {1, 2, 3};
        int[] c = {1, 2, 4};
        System.out.println("a == b: " + (a == b));               // false (different objects)
        System.out.println("Arrays.equals(a, b): " + Arrays.equals(a, b)); // true (same content)
        System.out.println("Arrays.equals(a, c): " + Arrays.equals(a, c)); // false
    }
}
```

**Output:**
```
Sorted: [3, 12, 23, 45, 56, 78, 91]
56 found at index: 4
50 search result: -5
Filled: [7, 7, 7, 7, 7]
Full copy: [1, 2, 3, 4, 5]
Partial copy: [2, 3, 4]
a == b: false
Arrays.equals(a, b): true
Arrays.equals(a, c): false
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Arrays are everywhere in backend systems, often behind the scenes. Here are three realistic scenarios.

### Scenario 1: Processing a JSON array from a REST API request

When a client sends a JSON array in a POST request body, Spring Boot deserializes it into a Java array or List. Here is how a controller handles a batch order creation request.

```java
package com.company.orderservice.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    // Client sends: [{"productId": 1, "qty": 2}, {"productId": 5, "qty": 1}]
    // Spring deserializes the JSON array into an OrderItem array.
    @PostMapping("/batch")
    public ResponseEntity<BatchOrderResponse> createBatchOrder(
            @RequestBody OrderItem[] items) {

        // Validate: batch must not be empty
        if (items == null || items.length == 0) {
            return ResponseEntity.badRequest()
                .body(new BatchOrderResponse("Order items array cannot be empty"));
        }

        // Validate: batch size limit to prevent abuse
        if (items.length > 100) {
            return ResponseEntity.badRequest()
                .body(new BatchOrderResponse("Maximum 100 items per batch order"));
        }

        int successCount = 0;
        int failureCount = 0;
        String[] errorMessages = new String[items.length];

        // Process each item in the array
        for (int i = 0; i < items.length; i++) {
            OrderItem item = items[i];

            if (item.getProductId() <= 0 || item.getQuantity() <= 0) {
                errorMessages[i] = "Item at index " + i + " has invalid data";
                failureCount++;
                continue;
            }

            try {
                orderService.createOrder(item);
                successCount++;
            } catch (Exception e) {
                errorMessages[i] = "Failed to process item " + i + ": " + e.getMessage();
                failureCount++;
            }
        }

        return ResponseEntity.ok(
            new BatchOrderResponse(successCount, failureCount, errorMessages)
        );
    }
}
```

**What to notice:**

- `@RequestBody OrderItem[] items` tells Spring to deserialize the incoming JSON array into a Java array. The framework handles the parsing automatically.
- The null check `items == null` is necessary because a malformed request body could result in a null array.
- The batch size limit (`items.length > 100`) is a security measure. Without it, a malicious client could send an array with millions of items and crash your server.
- The `errorMessages` array is pre-allocated to the same size as the input array. This is a common pattern when you need to track per-element results in a batch operation.

### Scenario 2: Configuration arrays loaded from application properties

Backend applications often load configuration values as arrays from property files.

```java
package com.company.orderservice.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class AppConfiguration {

    // In application.yml:
    // app:
    //   allowed-origins: "https://example.com,https://admin.example.com"
    //   blocked-ips: "192.168.1.100,10.0.0.50"
    //   max-retry-attempts: 3

    @Value("${app.allowed-origins}")
    private String[] allowedOrigins;

    @Value("${app.blocked-ips}")
    private String[] blockedIps;

    public boolean isOriginAllowed(String origin) {
        for (String allowed : allowedOrigins) {
            if (allowed.equals(origin)) {
                return true;
            }
        }
        return false;
    }

    public boolean isIpBlocked(String ipAddress) {
        for (String blockedIp : blockedIps) {
            if (blockedIp.equals(ipAddress)) {
                return true;
            }
        }
        return false;
    }
}
```

**What to notice:**

- Spring automatically splits comma-separated property values into String arrays when you inject them with `@Value`.
- The linear search through the array (`for` loop with `equals`) is fine for small configuration arrays. If the list grows to thousands of entries, you would switch to a `HashSet` for O(1) lookups. But for 5-10 allowed origins, an array is perfectly adequate and simpler.

### Scenario 3: Rate limiting with a sliding window using a 2D array concept

```java
package com.company.orderservice.security;

import java.time.Instant;

public class SimpleRateLimiter {

    // Track request timestamps per user.
    // In production, you would use Redis, not in-memory arrays.
    // This example shows the array logic behind rate limiting.

    private static final int MAX_REQUESTS = 60;
    private static final long WINDOW_SECONDS = 60;

    // Simplified: store the last 60 request timestamps for a single user
    private long[] requestTimestamps = new long[MAX_REQUESTS];
    private int currentIndex = 0;
    private int totalRequests = 0;

    public synchronized boolean isAllowed() {
        long now = Instant.now().getEpochSecond();

        // Count how many requests fall within the current window
        int requestsInWindow = 0;
        for (int i = 0; i < Math.min(totalRequests, MAX_REQUESTS); i++) {
            if (now - requestTimestamps[i] <= WINDOW_SECONDS) {
                requestsInWindow++;
            }
        }

        if (requestsInWindow >= MAX_REQUESTS) {
            return false;  // Rate limit exceeded
        }

        // Record this request using circular array indexing
        requestTimestamps[currentIndex] = now;
        currentIndex = (currentIndex + 1) % MAX_REQUESTS;
        totalRequests++;

        return true;
    }
}
```

**What to notice:**

- The array is used as a **circular buffer**. The `currentIndex` wraps around using the modulus operator `(currentIndex + 1) % MAX_REQUESTS`. This is a classic array technique that avoids shifting elements or allocating new memory.
- The `synchronized` keyword ensures thread safety. In a real backend, multiple threads (handling concurrent HTTP requests) could call this method simultaneously.
- The comment about Redis is important. In-memory arrays work for single-server deployments, but production backends run on multiple servers. A real rate limiter stores timestamps in a shared data store like Redis so all servers see the same request counts.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: ArrayIndexOutOfBoundsException

**Wrong:**
```java
int[] scores = {85, 92, 78};  // Valid indices: 0, 1, 2

System.out.println(scores[3]);  // CRASH: ArrayIndexOutOfBoundsException
System.out.println(scores[-1]); // CRASH: ArrayIndexOutOfBoundsException
```

**Right:**
```java
int[] scores = {85, 92, 78};

// Always check bounds before accessing
int index = 3;
if (index >= 0 && index < scores.length) {
    System.out.println(scores[index]);
} else {
    System.out.println("Index " + index + " is out of bounds.");
}

// Or use a loop that respects the length
for (int i = 0; i < scores.length; i++) {  // < not <=
    System.out.println(scores[i]);
}
```

**Why it is wrong:** Java arrays have fixed bounds. Accessing an index outside `0` to `length - 1` throws an `ArrayIndexOutOfBoundsException` at runtime. This is one of the most common exceptions in Java and frequently appears in backend logs when API clients send unexpected data.

### Mistake 2: Confusing `.length` (property) with `.length()` (method)

**Wrong:**
```java
int[] numbers = {1, 2, 3};
System.out.println(numbers.length());  // COMPILATION ERROR

String text = "Hello";
System.out.println(text.length);  // COMPILATION ERROR
```

**Right:**
```java
int[] numbers = {1, 2, 3};
System.out.println(numbers.length);   // 3 (property, no parentheses)

String text = "Hello";
System.out.println(text.length());    // 5 (method, with parentheses)
```

**Why it is wrong:** Arrays use `.length` as a public final field (property). Strings use `.length()` as a method. This inconsistency in the Java API is a historical design decision that confuses every beginner. Memorize the distinction: arrays have `length`, Strings have `length()`, and Collections (ArrayList, etc.) have `size()`.

### Mistake 3: Trying to resize an array

**Wrong:**
```java
int[] scores = {85, 92, 78};
scores.add(95);  // COMPILATION ERROR. Arrays do not have an add() method.
scores[3] = 95;  // RUNTIME ERROR. Index 3 does not exist in a length-3 array.
```

**Right:**
```java
// Option 1: Create a new, larger array and copy elements
int[] scores = {85, 92, 78};
int[] newScores = Arrays.copyOf(scores, scores.length + 1);
newScores[3] = 95;

// Option 2: Use ArrayList instead (recommended for dynamic collections)
import java.util.ArrayList;
ArrayList<Integer> scoreList = new ArrayList<>();
scoreList.add(85);
scoreList.add(92);
scoreList.add(78);
scoreList.add(95);  // Works! ArrayList resizes automatically.
```

**Why it is wrong:** Arrays are fixed-size by design. Once created, you cannot add or remove elements. If you need a resizable collection, use `ArrayList`. Arrays are best when you know the exact size in advance or when performance is critical (arrays have less memory overhead than ArrayLists).

### Mistake 4: Shallow copy vs deep copy with 2D arrays

**Wrong:**
```java
int[][] original = {{1, 2}, {3, 4}};
int[][] copy = original.clone();  // Shallow copy!

copy[0][0] = 99;
System.out.println(original[0][0]);  // 99! The original is modified!
// clone() copies the outer array but the inner arrays are still shared references.
```

**Right:**
```java
int[][] original = {{1, 2}, {3, 4}};

// Deep copy: clone each inner array individually
int[][] deepCopy = new int[original.length][];
for (int i = 0; i < original.length; i++) {
    deepCopy[i] = original[i].clone();
}

deepCopy[0][0] = 99;
System.out.println(original[0][0]);  // 1 (original is unchanged)
```

**Why it is wrong:** `clone()` on a 2D array creates a new outer array but copies the references to the inner arrays. Both the original and the copy point to the same inner arrays in memory. Modifying an element through one reference affects the other. This is a subtle bug that can corrupt data in backend systems that process matrix-like structures.

---

## Key Takeaways

> [!tip] Remember these points
> 1. Arrays are fixed-size, zero-indexed, homogeneous collections stored in contiguous memory. Access by index is O(1), making arrays the fastest structure for random access.
> 2. Arrays are objects in Java, even `int[]`. They live on the heap and have a `.length` property (not a method). Use `Arrays.toString()` to print them and `Arrays.equals()` to compare them.
> 3. 2D arrays are arrays of arrays. They can be rectangular (all rows same length) or jagged (rows of different lengths). Use `Arrays.deepToString()` to print 2D arrays.
> 4. The `java.util.Arrays` utility class provides essential operations: `sort()`, `binarySearch()`, `fill()`, `copyOf()`, `copyOfRange()`, and `equals()`. Use these instead of writing your own implementations.
> 5. Arrays are fixed-size. If you need to add or remove elements dynamically, use `ArrayList` (covered in Phase 1). Use arrays when the size is known in advance or when raw performance matters.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Array Statistics (Easy)
Write a program that creates an array of 10 `double` values representing daily temperatures in Dhaka for a week (you can make up the values). Calculate and print:
- The average temperature
- The highest temperature and its day (index)
- The lowest temperature and its day (index)
- The number of days the temperature was above 35 degrees

Use a single `for` loop to compute all four statistics in one pass through the array.

**Hint:** Initialize `max` to `Double.MIN_VALUE` and `min` to `Double.MAX_VALUE` before the loop. Update them inside the loop as you find higher or lower values.

### Exercise 2: Array Reversal (Medium)
Write a program that reverses an array of integers **in place** (without creating a second array). Use two pointers: one starting at the beginning and one at the end. Swap the elements at both pointers and move them toward the center until they meet.

Then write a second version that creates a new reversed array instead of modifying the original. Compare the two approaches.

**Hint:** Use a `while` loop with `left < right` as the condition. Swap using a temporary variable: `int temp = arr[left]; arr[left] = arr[right]; arr[right] = temp;`

### Exercise 3: 2D Array - Matrix Multiplication (Hard)
Write a program that multiplies two 2x3 and 3x2 matrices and stores the result in a 2x2 matrix. Use three nested loops: the outer two iterate over the result matrix's rows and columns, and the innermost loop computes the dot product.

Matrix A (2x3):
```
1  2  3
4  5  6
```

Matrix B (3x2):
```
7   8
9  10
11 12
```

Expected result (2x2):
```
 58   64
139  154
```

**Hint:** The element at `result[i][j]` is the sum of `A[i][k] * B[k][j]` for all valid values of `k`.

### Exercise 4: Remove Duplicates (Hard, Optional)
Write a program that takes a sorted array of integers and removes all duplicate values **in place**, returning the new length of the array. You cannot use any additional data structures (no HashSet, no ArrayList). Use two pointers: one for reading and one for writing.

Example: `[1, 1, 2, 2, 3, 4, 4, 4, 5]` should become `[1, 2, 3, 4, 5, ...]` with a returned length of 5.

**Hint:** Since the array is sorted, duplicates are always adjacent. The write pointer only advances when you find a value different from the previous one.

### Solution
For Exercise 1:
```java
public class TemperatureStats {
    public static void main(String[] args) {
        double[] temps = {33.5, 35.2, 36.8, 34.1, 37.5, 36.0, 35.8, 38.2, 34.9, 36.3};

        double sum = 0;
        double max = Double.MIN_VALUE;
        double min = Double.MAX_VALUE;
        int maxDay = 0;
        int minDay = 0;
        int hotDays = 0;

        for (int i = 0; i < temps.length; i++) {
            sum += temps[i];

            if (temps[i] > max) {
                max = temps[i];
                maxDay = i;
            }
            if (temps[i] < min) {
                min = temps[i];
                minDay = i;
            }
            if (temps[i] > 35.0) {
                hotDays++;
            }
        }

        System.out.println("Average: " + (sum / temps.length));
        System.out.println("Highest: " + max + " on day " + (maxDay + 1));
        System.out.println("Lowest: " + min + " on day " + (minDay + 1));
        System.out.println("Days above 35: " + hotDays);
    }
}
```

For Exercise 2: (in-place reversal)
```java
import java.util.Arrays;

public class ArrayReversal {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5, 6, 7};
        System.out.println("Original: " + Arrays.toString(arr));

        int left = 0;
        int right = arr.length - 1;

        while (left < right) {
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
            left++;
            right--;
        }

        System.out.println("Reversed: " + Arrays.toString(arr));
    }
}
```

For Exercise 3:
```java
import java.util.Arrays;

public class MatrixMultiplication {
    public static void main(String[] args) {
        int[][] A = {
            {1, 2, 3},
            {4, 5, 6}
        };

        int[][] B = {
            {7, 8},
            {9, 10},
            {11, 12}
        };

        int rowsA = A.length;
        int colsA = A[0].length;
        int colsB = B[0].length;

        int[][] result = new int[rowsA][colsB];

        for (int i = 0; i < rowsA; i++) {
            for (int j = 0; j < colsB; j++) {
                for (int k = 0; k < colsA; k++) {
                    result[i][j] += A[i][k] * B[k][j];
                }
            }
        }

        System.out.println("Result:");
        for (int[] row : result) {
            System.out.println(Arrays.toString(row));
        }
    }
}
```

For Exercise 4:

```java
import java.util.Arrays;

public class RemoveDuplicates {
    public static void main(String[] args) {
        int[] arr = {1, 1, 2, 2, 3, 4, 4, 4, 5};
        int newLength = removeDuplicates(arr);

        System.out.println("New length: " + newLength);
        System.out.print("Array: ");
        for (int i = 0; i < newLength; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }

    static int removeDuplicates(int[] arr) {
        if (arr.length == 0) return 0;

        int writeIndex = 1;

        for (int readIndex = 1; readIndex < arr.length; readIndex++) {
            if (arr[readIndex] != arr[readIndex - 1]) {
                arr[writeIndex] = arr[readIndex];
                writeIndex++;
            }
        }

        return writeIndex;
    }
}
```

---
## Related Notes

- [[Java - Loops - For While Do-While]]
- [[Java - Strings and String Methods]]
- [[Java - List - ArrayList LinkedList]] (Phase 1)

---
## Resources

- [Oracle Java Tutorials: Arrays](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/arrays.html) - Official documentation covering array creation, access, and copying.
- [Oracle Java Documentation: java.util.Arrays](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html) - Complete API reference for the Arrays utility class.
- [Baeldung: Java Arrays Guide](https://www.baeldung.com/java-arrays-guide) - Comprehensive guide covering all array operations with examples.
- [Baeldung: Java 2D Arrays](https://www.baeldung.com/java-multi-dimensional-arrays) - Detailed explanation of rectangular and jagged 2D arrays.

---