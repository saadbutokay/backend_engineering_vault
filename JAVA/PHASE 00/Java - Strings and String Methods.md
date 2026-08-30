---
title: "Java - Strings and String Methods"
phase: "Phase 0 - Foundations"
language: "java"
tags:
  - backend
  - java
  - foundations
  - strings
  - string-methods
status: "not-started"
---

# Java - Strings and String Methods

> [!abstract] Overview
> A String in Java is an immutable sequence of characters stored as an object on the heap. Strings are the most frequently used reference type in backend development. Every HTTP request URL, every JSON payload, every SQL query, every log message, every email address, and every error message is a String. Understanding String behavior, immutability, the String pool, and the difference between String, StringBuilder, and StringBuffer is essential for writing correct and performant backend code.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Variables and Data Types]]
> - [[Java - Arrays - 1D and 2D]]
> - [[Java - Operators - Arithmetic Relational Logical]]

---

## Theory

### What is a String?

A String in Java is an object of the `java.lang.String` class that represents a sequence of characters. Unlike primitive types, Strings are reference types. They live on the heap, and the variable holds a reference to the String object.

```java
String name = "Saad";
// 'name' is a reference variable on the stack.
// The actual String object "Saad" lives on the heap.
```

Internally, a String stores its characters in an array. In Java 8 and earlier, this was a `char[]` array (2 bytes per character). Starting from Java 9, Strings use a `byte[]` array with a compact encoding called **Compact Strings**. If the String contains only Latin-1 characters (ASCII and Western European), each character uses 1 byte. If it contains characters outside Latin-1 (like Bangla, Chinese, or emoji), it uses 2 bytes per character (UTF-16). This optimization reduced memory usage by roughly 25-40% for typical backend applications that mostly process ASCII text.

### String Immutability

Strings in Java are **immutable**. Once a String object is created, its content cannot be changed. Any operation that appears to modify a String actually creates a new String object and returns it.

```java
String greeting = "Hello";
greeting = greeting + " World";
// The original "Hello" object is NOT modified.
// A new String object "Hello World" is created.
// The variable 'greeting' now points to the new object.
// The old "Hello" object still exists in memory until garbage collected.
```

**Why are Strings immutable?**

1. **Security**: Strings are used for database connection URLs, file paths, usernames, passwords, and API keys. If Strings were mutable, a malicious piece of code could change the content of a String after it passed a security check but before it was used. For example, a connection string could be changed from a safe database to a malicious one.

2. **Thread safety**: Backend servers handle thousands of concurrent requests using multiple threads. Immutable objects are inherently thread-safe because no thread can modify them. Multiple threads can read the same String simultaneously without synchronization.

3. **String pool optimization**: Java maintains a special area of the heap called the **String pool** (or intern pool). When you create a String literal like `"Hello"`, Java checks if an identical String already exists in the pool. If it does, the new variable points to the existing object instead of creating a duplicate. This saves enormous amounts of memory. Immutability makes the pool safe because no code can alter a pooled String and affect all other references to it.

4. **Hash code caching**: Strings are frequently used as keys in HashMaps (for example, HTTP header names, JSON field names, configuration keys). The hash code of a String is computed once and cached. If Strings were mutable, the hash code could change after insertion into a HashMap, making the entry unretrievable.

### The String Pool

```java
String a = "Hello";       // Created in the String pool
String b = "Hello";       // Reuses the same object from the pool
String c = new String("Hello");  // Creates a NEW object on the heap (outside the pool)

System.out.println(a == b);       // true (same reference in the pool)
System.out.println(a == c);       // false (different objects)
System.out.println(a.equals(c));  // true (same content)
```

**Memory layout:**

```
String Pool (special area in heap)     Regular Heap
+-------------------+                 +-------------------+
| "Hello"           |<--- a, b        | String@0x3A2F      |<--- c
+-------------------+                 | content: "Hello"   |
                                      +-------------------+
```

You can manually add a String to the pool using the `intern()` method, but this is rarely needed in application code. The JVM handles pooling automatically for String literals.

### String Creation

There are two ways to create Strings, and they behave differently:

```java
// Method 1: String literal (uses the pool)
String s1 = "Java";

// Method 2: Using the new keyword (always creates a new object on the heap)
String s2 = new String("Java");

// Method 3: From a char array
char[] chars = {'J', 'a', 'v', 'a'};
String s3 = new String(chars);

// Method 4: From a byte array (common when reading files or network data)
byte[] bytes = {72, 101, 108, 108, 111};
String s4 = new String(bytes);  // "Hello"
```

In backend development, you almost always use String literals or receive Strings from frameworks (Spring deserializes JSON into Strings automatically). You rarely call `new String()` explicitly unless you are converting from byte arrays (for example, reading a file or decoding a JWT token).

### Essential String Methods

The String class has over 60 methods. Here are the ones you will use most frequently in backend development, grouped by category.

**Inspection methods:**

| Method | Returns | Description |
|--------|---------|-------------|
| `length()` | `int` | Number of characters |
| `charAt(int index)` | `char` | Character at the given index |
| `isEmpty()` | `boolean` | True if length is 0 |
| `isBlank()` | `boolean` | True if empty or only whitespace (Java 11+) |
| `contains(CharSequence)` | `boolean` | True if the String contains the given sequence |
| `startsWith(String)` | `boolean` | True if the String starts with the given prefix |
| `endsWith(String)` | `boolean` | True if the String ends with the given suffix |
| `indexOf(String)` | `int` | Index of the first occurrence, or -1 if not found |
| `lastIndexOf(String)` | `int` | Index of the last occurrence, or -1 if not found |

**Extraction methods:**

| Method | Returns | Description |
|--------|---------|-------------|
| `substring(int begin, int end)` | `String` | Extracts characters from begin (inclusive) to end (exclusive) |
| `trim()` | `String` | Removes leading and trailing whitespace |
| `strip()` | `String` | Like trim() but handles all Unicode whitespace (Java 11+) |
| `stripLeading()` | `String` | Removes leading whitespace only (Java 11+) |
| `stripTrailing()` | `String` | Removes trailing whitespace only (Java 11+) |

**Transformation methods:**

| Method | Returns | Description |
|--------|---------|-------------|
| `toUpperCase()` | `String` | Converts all characters to uppercase |
| `toLowerCase()` | `String` | Converts all characters to lowercase |
| `replace(char, char)` | `String` | Replaces all occurrences of a character |
| `replace(CharSequence, CharSequence)` | `String` | Replaces all occurrences of a substring |
| `replaceAll(String regex, String)` | `String` | Replaces all matches of a regex pattern |
| `split(String regex)` | `String[]` | Splits the String into an array by the regex delimiter |

**Comparison methods:**

| Method | Returns | Description |
|--------|---------|-------------|
| `equals(Object)` | `boolean` | True if content is identical (case-sensitive) |
| `equalsIgnoreCase(String)` | `boolean` | True if content is identical ignoring case |
| `compareTo(String)` | `int` | Negative if less, 0 if equal, positive if greater (lexicographic) |

**Construction methods:**

| Method | Returns | Description |
|--------|---------|-------------|
| `String.format(String, Object...)` | `String` | Creates a formatted String (like printf) |
| `String.join(CharSequence, CharSequence...)` | `String` | Joins multiple Strings with a delimiter (Java 8+) |
| `String.valueOf(Object)` | `String` | Converts any object to its String representation |

### StringBuilder and StringBuffer

Because Strings are immutable, concatenating Strings in a loop creates a new object on every iteration. This is extremely wasteful for large numbers of concatenations.

```java
// BAD: Creates 10,000 intermediate String objects
String result = "";
for (int i = 0; i < 10000; i++) {
    result += i;  // Each iteration creates a new String
}

// GOOD: Uses a single mutable buffer
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);  // Modifies the existing buffer, no new objects
}
String result = sb.toString();
```

**StringBuilder** is mutable and not thread-safe. It is the fastest option for single-threaded string building, which covers 95% of backend use cases.

**StringBuffer** is mutable and thread-safe (all methods are `synchronized`). It is slower than StringBuilder due to the synchronization overhead. Use it only when multiple threads need to modify the same string buffer concurrently, which is rare in modern backend code.

| Feature | String | StringBuilder | StringBuffer |
|---------|--------|---------------|--------------|
| Mutable | No | Yes | Yes |
| Thread-safe | Yes (immutable) | No | Yes |
| Performance | Slow for concatenation | Fast | Moderate |
| Use case | Fixed text, keys, constants | Building strings in loops | Multi-threaded string building |

> [!tip] Key Insight
> When you write `String s = "a" + "b" + "c"`, the Java compiler optimizes this at compile time into `String s = "abc"`. No StringBuilder is created. The performance problem only occurs when concatenation involves variables inside loops, because the compiler cannot predict the values at compile time. Modern compilers (Java 9+) also optimize simple variable concatenation by automatically generating `invokedynamic` calls to `StringConcatFactory`, which uses the most efficient strategy available. However, for loops with many iterations, explicit StringBuilder is still significantly faster.

---

## Syntax and Basic Examples

### Example 1: String creation and the String pool

```java
public class StringPoolDemo {
    public static void main(String[] args) {
        String a = "Backend";
        String b = "Backend";
        String c = new String("Backend");
        String d = c.intern();  // Returns the pooled reference

        System.out.println("a == b: " + (a == b));           // true (same pool object)
        System.out.println("a == c: " + (a == c));           // false (c is on the heap)
        System.out.println("a == d: " + (a == d));           // true (intern() returns pool ref)
        System.out.println("a.equals(c): " + a.equals(c));   // true (same content)
    }
}
```

**Output:**
```
a == b: true
a == c: false
a == d: true
a.equals(c): true
```

### Example 2: Common String methods in action

```java
public class StringMethodsDemo {
    public static void main(String[] args) {
        String email = "  saad@example.com  ";

        // Cleaning input
        String clean = email.strip();
        System.out.println("Stripped: '" + clean + "'");          // 'saad@example.com'

        // Inspection
        System.out.println("Length: " + clean.length());           // 16
        System.out.println("Contains @: " + clean.contains("@"));  // true
        System.out.println("Starts with saad: " + clean.startsWith("saad"));  // true
        System.out.println("Ends with .com: " + clean.endsWith(".com"));      // true
        System.out.println("Index of @: " + clean.indexOf("@"));   // 4
        System.out.println("Is blank: " + clean.isBlank());        // false
        System.out.println("Is empty: " + "".isEmpty());           // true

        // Extraction
        String username = clean.substring(0, clean.indexOf("@"));
        String domain = clean.substring(clean.indexOf("@") + 1);
        System.out.println("Username: " + username);  // saad
        System.out.println("Domain: " + domain);      // example.com

        // Transformation
        String upper = clean.toUpperCase();
        System.out.println("Upper: " + upper);  // SAAD@EXAMPLE.COM

        String replaced = clean.replace(".com", ".org");
        System.out.println("Replaced: " + replaced);  // saad@example.org

        // Splitting
        String csvLine = "Saad,22,CSE,3.72";
        String[] fields = csvLine.split(",");
        for (String field : fields) {
            System.out.println("Field: " + field);
        }

        // Joining
        String joined = String.join(" | ", fields);
        System.out.println("Joined: " + joined);  // Saad | 22 | CSE | 3.72

        // Formatting
        String formatted = String.format("Student %s (ID: %d) has CGPA %.2f", "Saad", 230145, 3.72);
        System.out.println(formatted);  // Student Saad (ID: 230145) has CGPA 3.72
    }
}
```

**Output:**
```
Stripped: 'saad@example.com'
Length: 16
Contains @: true
Starts with saad: true
Ends with .com: true
Index of @: 4
Is blank: false
Is empty: true
Username: saad
Domain: example.com
Upper: SAAD@EXAMPLE.COM
Replaced: saad@example.org
Field: Saad
Field: 22
Field: CSE
Field: 3.72
Joined: Saad | 22 | CSE | 3.72
Student Saad (ID: 230145) has CGPA 3.72
```

### Example 3: StringBuilder for efficient concatenation

```java
public class StringBuilderDemo {
    public static void main(String[] args) {
        // Building a SQL query dynamically (common in backend code)
        StringBuilder query = new StringBuilder();
        query.append("SELECT o.id, o.total, u.name ");
        query.append("FROM orders o ");
        query.append("JOIN users u ON o.user_id = u.id ");
        query.append("WHERE o.status = 'PAID' ");
        query.append("AND o.created_at >= '2025-01-01' ");
        query.append("ORDER BY o.created_at DESC ");
        query.append("LIMIT 50");

        System.out.println(query.toString());

        // Building a JSON-like response manually (for demonstration)
        String[] names = {"Alice", "Bob", "Charlie"};
        StringBuilder json = new StringBuilder();
        json.append("[");
        for (int i = 0; i < names.length; i++) {
            json.append("{\"name\":\"").append(names[i]).append("\"}");
            if (i < names.length - 1) {
                json.append(",");
            }
        }
        json.append("]");

        System.out.println(json.toString());
        // [{"name":"Alice"},{"name":"Bob"},{"name":"Charlie"}]

        // StringBuilder methods
        StringBuilder sb = new StringBuilder("Hello World");
        System.out.println("Original: " + sb);
        sb.reverse();
        System.out.println("Reversed: " + sb);  // dlroW olleH
        sb.delete(0, 6);
        System.out.println("After delete: " + sb);  // olleH
        sb.insert(0, "H");
        System.out.println("After insert: " + sb);  // HolleH
        sb.replace(1, 4, "ell");
        System.out.println("After replace: " + sb);  // HelloH
    }
}
```

### Example 4: String comparison methods

```java
public class StringComparison {
    public static void main(String[] args) {
        String a = "apple";
        String b = "banana";
        String c = "Apple";

        // equals vs equalsIgnoreCase
        System.out.println("a.equals(c): " + a.equals(c));               // false
        System.out.println("a.equalsIgnoreCase(c): " + a.equalsIgnoreCase(c)); // true

        // compareTo (lexicographic order based on Unicode values)
        System.out.println("a.compareTo(b): " + a.compareTo(b));  // negative (a < b)
        System.out.println("b.compareTo(a): " + b.compareTo(a));  // positive (b > a)
        System.out.println("a.compareTo(a): " + a.compareTo(a));  // 0 (equal)

        // Practical use: sorting
        String[] fruits = {"Mango", "apple", "Banana", "cherry"};
        java.util.Arrays.sort(fruits);
        System.out.println("Sorted: " + java.util.Arrays.toString(fruits));
        // [Banana, Mango, apple, cherry] (uppercase comes before lowercase in Unicode!)

        // Case-insensitive sorting
        java.util.Arrays.sort(fruits, String.CASE_INSENSITIVE_ORDER);
        System.out.println("Case-insensitive: " + java.util.Arrays.toString(fruits));
        // [apple, Banana, cherry, Mango]
    }
}
```

**Output:**
```
a.equals(c): false
a.equalsIgnoreCase(c): true
a.compareTo(b): -1
b.compareTo(a): 1
a.compareTo(a): 0
Sorted: [Banana, Mango, apple, cherry]
Case-insensitive: [apple, Banana, cherry, Mango]
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> String manipulation is at the core of every backend system. Here are three realistic scenarios.

### Scenario 1: Input validation and sanitization in a service layer

Every backend API must validate and clean user input before processing it. String methods are the primary tools for this.

```java
package com.company.orderservice.service;

public class UserInputValidator {

    public ValidatedUser validateAndSanitize(String name, String email, String phone) {
        // 1. Null and blank checks
        if (name == null || name.isBlank()) {
            throw new ValidationException("Name cannot be empty");
        }
        if (email == null || email.isBlank()) {
            throw new ValidationException("Email cannot be empty");
        }

        // 2. Trim and normalize
        String cleanName = name.strip().replaceAll("\\s+", " ");
        // strip() removes leading/trailing whitespace
        // replaceAll("\\s+", " ") collapses multiple spaces into one
        // "  Abdullah   Al   Sayb  " becomes "Abdullah Al Sayb"

        String cleanEmail = email.strip().toLowerCase();
        // Emails are case-insensitive, so normalize to lowercase
        // "Saad@Example.COM" becomes "saad@example.com"

        // 3. Format validation
        if (!cleanEmail.contains("@") || !cleanEmail.contains(".")) {
            throw new ValidationException("Invalid email format");
        }

        String emailDomain = cleanEmail.substring(cleanEmail.indexOf("@") + 1);
        if (!emailDomain.contains(".")) {
            throw new ValidationException("Email domain is invalid");
        }

        // 4. Length validation
        if (cleanName.length() < 2 || cleanName.length() > 100) {
            throw new ValidationException("Name must be between 2 and 100 characters");
        }
        if (cleanEmail.length() > 255) {
            throw new ValidationException("Email must not exceed 255 characters");
        }

        // 5. Phone number normalization
        String cleanPhone = "";
        if (phone != null && !phone.isBlank()) {
            cleanPhone = phone.replaceAll("[^0-9+]", "");
            // Remove all characters except digits and '+'
            // "(+880) 1712-345678" becomes "+8801712345678"

            if (!cleanPhone.startsWith("+880") && !cleanPhone.startsWith("01")) {
                throw new ValidationException("Phone number must be a valid Bangladeshi number");
            }
        }

        return new ValidatedUser(cleanName, cleanEmail, cleanPhone);
    }
}
```

**What to notice:**

- `isBlank()` (Java 11+) is preferred over `isEmpty()` for input validation because it also catches strings that contain only spaces or tabs. A user submitting `"   "` as their name should be rejected.
- `strip()` is preferred over `trim()` because it handles all Unicode whitespace characters, not just ASCII spaces. This matters for internationalized applications where users might input non-breaking spaces or other Unicode whitespace.
- `toLowerCase()` normalizes email addresses. Email systems treat `Saad@Example.com` and `saad@example.com` as the same address. Normalizing to lowercase prevents duplicate accounts.
- `replaceAll("[^0-9+]", "")` uses a regular expression to strip all non-numeric characters from phone numbers. This is a common pattern for normalizing user input that may contain formatting characters like parentheses, dashes, and spaces.

### Scenario 2: Building dynamic SQL queries safely

```java
package com.company.orderservice.repository;

import java.util.List;
import java.util.ArrayList;

public class OrderQueryBuilder {

    // This demonstrates how String building works in query construction.
    // In real Spring Boot apps, you would use Spring Data JPA or QueryDSL
    // instead of building SQL strings manually. But understanding the
    // underlying String mechanics is still important.

    public String buildSearchQuery(String status, String customerName,
                                    String dateFrom, String dateTo,
                                    int page, int size) {
        StringBuilder query = new StringBuilder();
        List<String> conditions = new ArrayList<>();

        query.append("SELECT * FROM orders WHERE 1=1");
        // "WHERE 1=1" is a common trick that lets you append
        // all conditions with "AND" without checking if it's the first one.

        if (status != null && !status.isBlank()) {
            conditions.add("status = '" + status.toUpperCase() + "'");
        }

        if (customerName != null && !customerName.isBlank()) {
            // WARNING: This is vulnerable to SQL injection!
            // In real code, use PreparedStatement with ? placeholders.
            // This example shows String manipulation, not secure coding.
            conditions.add("customer_name ILIKE '%" + customerName.strip() + "%'");
        }

        if (dateFrom != null && !dateFrom.isBlank()) {
            conditions.add("created_at >= '" + dateFrom + "'");
        }

        if (dateTo != null && !dateTo.isBlank()) {
            conditions.add("created_at <= '" + dateTo + "'");
        }

        // Append all conditions
        for (String condition : conditions) {
            query.append(" AND ").append(condition);
        }

        // Append pagination
        int offset = page * size;
        query.append(" ORDER BY created_at DESC");
        query.append(" LIMIT ").append(size);
        query.append(" OFFSET ").append(offset);

        return query.toString();
    }
}
```

**What to notice:**

- `StringBuilder.append()` is used throughout instead of `+` concatenation. This method returns the StringBuilder itself, enabling method chaining: `sb.append("a").append("b").append("c")`.
- The `WHERE 1=1` trick simplifies the logic. Without it, you would need to check whether to append `WHERE` or `AND` before each condition, which makes the code messier.
- The comment about SQL injection is critical. String concatenation in SQL queries is the number one cause of SQL injection vulnerabilities. In real backend code, you use parameterized queries (`PreparedStatement` in JDBC, `@Query` with `?1` in Spring Data JPA). This example shows the String mechanics, not the security best practice.

### Scenario 3: Parsing and constructing URLs and paths

```java
package com.company.orderservice.util;

public class UrlUtils {

    public static String buildApiUrl(String baseUrl, String version,
                                      String resource, String id) {
        // Ensure baseUrl does not end with a slash
        if (baseUrl.endsWith("/")) {
            baseUrl = baseUrl.substring(0, baseUrl.length() - 1);
        }

        // Build the path
        StringBuilder url = new StringBuilder(baseUrl);
        url.append("/api/").append(version);
        url.append("/").append(resource);

        if (id != null && !id.isBlank()) {
            url.append("/").append(id);
        }

        return url.toString();
    }

    public static String extractTokenFromHeader(String authHeader) {
        // HTTP Authorization header format: "Bearer eyJhbGciOiJIUzI1NiJ9..."
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            throw new SecurityException("Missing or invalid Authorization header");
        }

        // Extract the token part after "Bearer "
        return authHeader.substring(7).strip();
        // "Bearer " is 7 characters long
    }

    public static String maskSensitiveData(String data, int visibleChars) {
        if (data == null || data.length() <= visibleChars) {
            return data;
        }

        String visible = data.substring(0, visibleChars);
        String masked = "*".repeat(data.length() - visibleChars);
        return visible + masked;
    }
}
```

**What to notice:**

- `endsWith("/")` and `substring()` are used to normalize the base URL. URL construction is a common source of bugs in backend systems because double slashes (`//api/v1`) or missing slashes (`/apiv1`) cause 404 errors.
- `startsWith("Bearer ")` validates the Authorization header format before extracting the token. The `substring(7)` call extracts everything after the 7-character prefix "Bearer ".
- `"*".repeat(n)` (Java 11+) creates a String of `n` asterisks. This is used to mask sensitive data like credit card numbers, phone numbers, or API keys in log output. You never want to log full credit card numbers in a production backend.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using `==` to compare Strings

**Wrong:**
```java
String input = getUserInput();  // Returns "admin" from user input
if (input == "admin") {
    grantAccess();  // This will NEVER execute for user input
}
```

**Right:**
```java
String input = getUserInput();
if ("admin".equals(input)) {
    grantAccess();  // This works correctly
}
```

**Why it is wrong:** This was covered in the operators note, but it bears repeating because it is the single most common String bug in Java. `==` compares references (memory addresses), not content. String literals from the same source file may share the same reference due to the String pool, making `==` appear to work during testing. But Strings from user input, database queries, API responses, and file reads are always new objects with different references. Always use `.equals()` for content comparison.

### Mistake 2: String concatenation in loops

**Wrong:**
```java
String csv = "";
for (int i = 0; i < 100000; i++) {
    csv += i + ",";  // Creates 100,000 intermediate String objects
}
// This takes several seconds and generates massive garbage collection pressure.
```

**Right:**
```java
StringBuilder csv = new StringBuilder();
for (int i = 0; i < 100000; i++) {
    csv.append(i).append(",");  // Modifies the same buffer
}
String result = csv.toString();
// This takes milliseconds.
```

**Why it is wrong:** Each `+=` operation creates a new String object, copies the old content, appends the new content, and discards the old String. For 100,000 iterations, this creates 100,000 temporary objects that the garbage collector must clean up. In a backend server handling many concurrent requests, this can cause GC pauses that degrade response times.

### Mistake 3: Not handling null before calling String methods

**Wrong:**
```java
String name = getUserNameFromDatabase();  // Might return null
if (name.toLowerCase().equals("admin")) {  // NullPointerException if name is null!
    grantAccess();
}
```

**Right:**
```java
String name = getUserNameFromDatabase();

// Option 1: Null check first
if (name != null && name.equalsIgnoreCase("admin")) {
    grantAccess();
}

// Option 2: Put the known non-null String first (Yoda condition)
if ("admin".equalsIgnoreCase(name)) {
    grantAccess();  // Safe even if name is null. "admin".equalsIgnoreCase(null) returns false.
}
```

**Why it is wrong:** Calling any method on a null reference throws a `NullPointerException`. In backend systems, null Strings come from database columns that allow NULL, missing JSON fields, optional request parameters, and failed API responses. Always check for null before calling String methods, or use the Yoda condition pattern where the known non-null literal calls `.equals()` on the potentially null variable.

### Mistake 4: Forgetting that String methods return new Strings

**Wrong:**
```java
String email = "  SAAD@EXAMPLE.COM  ";
email.trim();
email.toLowerCase();
System.out.println(email);  // "  SAAD@EXAMPLE.COM  " (unchanged!)
// The methods returned new Strings, but the return values were discarded.
```

**Right:**
```java
String email = "  SAAD@EXAMPLE.COM  ";
email = email.trim().toLowerCase();
System.out.println(email);  // "saad@example.com"
```

**Why it is wrong:** Strings are immutable. Methods like `trim()`, `toLowerCase()`, `replace()`, and `substring()` do not modify the original String. They return a new String with the result. If you do not assign the return value to a variable, the result is lost. This is a very common beginner mistake that leads to subtle bugs where input validation appears to pass but the data remains uncleaned.

---

## Key Takeaways

> [!tip] Remember these points
> 1. Strings are **immutable** reference types. Every modification creates a new String object. This ensures thread safety, security, and enables the String pool optimization.
> 2. Use `.equals()` for content comparison, never `==`. Use `"literal".equals(variable)` (Yoda condition) to avoid NullPointerException when the variable might be null.
> 3. Use `StringBuilder` for string concatenation inside loops. For simple concatenation of a few variables outside loops, the `+` operator is fine because the compiler optimizes it.
> 4. The most important String methods for backend development are: `strip()`, `isBlank()`, `contains()`, `startsWith()`, `endsWith()`, `substring()`, `split()`, `replace()`, `toLowerCase()`, `toUpperCase()`, `indexOf()`, and `String.format()`.
> 5. Strings created with literals (`"text"`) go into the String pool and share references. Strings created with `new String("text")` always create new heap objects. In practice, you almost always use literals or receive Strings from frameworks.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Palindrome Checker (Easy)
Write a program that takes a String as input and checks whether it is a palindrome (reads the same forwards and backwards). The check should be case-insensitive and should ignore spaces and punctuation.

Examples:
- "racecar" -> true
- "A man a plan a canal Panama" -> true
- "hello" -> false

**Hint:** First clean the string by removing non-alphanumeric characters and converting to lowercase. Then compare characters from the beginning and end moving toward the center.

### Exercise 2: CSV Parser (Medium)
Write a program that parses a CSV string into a 2D array of Strings. The CSV may contain quoted fields that include commas.

Input: `"Saad,22,\"Computer Science, CSE\",3.72"`
Expected output: `["Saad", "22", "Computer Science, CSE", "3.72"]`

For simplicity, you can start with a version that does not handle quoted fields (just use `split(",")`), then extend it to handle quotes.

**Hint:** For the simple version, `String.split(",")` is sufficient. For the quoted version, iterate through the string character by character, tracking whether you are inside quotes.

### Exercise 3: Log Line Parser (Medium)
Write a program that parses log lines in the following format and extracts the timestamp, level, and message:

```
[2025-07-10 14:30:45] ERROR - Database connection failed: timeout after 30s
[2025-07-10 14:31:02] INFO  - Order #12345 created successfully
[2025-07-10 14:31:15] WARN  - Rate limit approaching for user saad@example.com
```

Extract and print:
- Timestamp: `2025-07-10 14:30:45`
- Level: `ERROR`
- Message: `Database connection failed: timeout after 30s`

Use `substring()`, `indexOf()`, and `strip()` to parse each line.

**Hint:** The timestamp is between `[` and `]`. The level is between `] ` and ` - `. The message is everything after ` - `.

### Exercise 4: StringBuilder Performance Test (Hard, Optional)
Write a program that compares the performance of String concatenation vs StringBuilder for building a large string. Concatenate numbers 1 through 100,000 using both approaches. Measure the time taken by each using `System.nanoTime()`. Print the results and calculate how many times faster StringBuilder is.

**Hint:**
```java
long start = System.nanoTime();
// ... do the work ...
long end = System.nanoTime();
double durationMs = (end - start) / 1_000_000.0;
```

### Solution
For Exercise 1:
```java
import java.util.Scanner;

public class PalindromeChecker {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter a string: ");
        String input = scanner.nextLine();

        // Clean: remove non-alphanumeric, convert to lowercase
        String cleaned = input.replaceAll("[^a-zA-Z0-9]", "").toLowerCase();

        // Check palindrome using two pointers
        int left = 0;
        int right = cleaned.length() - 1;
        boolean isPalindrome = true;

        while (left < right) {
            if (cleaned.charAt(left) != cleaned.charAt(right)) {
                isPalindrome = false;
                break;
            }
            left++;
            right--;
        }

        System.out.println("\"" + input + "\" is "
            + (isPalindrome ? "" : "not ") + "a palindrome.");
        scanner.close();
    }
}
```


For Exercise 3:
```java
public class LogParser {
    public static void main(String[] args) {
        String[] logs = {
            "[2025-07-10 14:30:45] ERROR - Database connection failed: timeout after 30s",
            "[2025-07-10 14:31:02] INFO  - Order #12345 created successfully",
            "[2025-07-10 14:31:15] WARN  - Rate limit approaching for user saad@example.com"
        };

        for (String log : logs) {
            int closeBracket = log.indexOf("]");
            String timestamp = log.substring(1, closeBracket);

            int dashIndex = log.indexOf(" - ");
            String level = log.substring(closeBracket + 2, dashIndex).strip();

            String message = log.substring(dashIndex + 3).strip();

            System.out.println("Timestamp: " + timestamp);
            System.out.println("Level:     " + level);
            System.out.println("Message:   " + message);
            System.out.println("---");
        }
    }
}
```

---
## Related Notes

- [[Java - Arrays - 1D and 2D]]
- [[Java - Methods - Parameters Return Types Overloading]]
- [[Java - File I-O - FileReader FileWriter BufferedReader]]

---
## Resources

- [Oracle Java Tutorials: Strings](https://docs.oracle.com/javase/tutorial/java/data/strings.html) - Official documentation covering String creation, comparison, and manipulation.
- [Oracle Java Documentation: java.lang.String](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html) - Complete API reference for all 60+ String methods.
- [Baeldung: Java String Pool](https://www.baeldung.com/java-string-pool) - Detailed explanation of the String pool with memory diagrams.
- [Baeldung: StringBuilder vs StringBuffer](https://www.baeldung.com/java-string-builder-string-buffer) - Performance comparison and use case guide.
- [Baeldung: Java String Methods Cheat Sheet](https://www.baeldung.com/java-string-operations) - Quick reference for the most commonly used String methods.
