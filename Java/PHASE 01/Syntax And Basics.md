# ## Overview
Java is a statically typed, class-based language. Every piece of code lives inside a class. Every variable has a declared type that cannot change. The compiler checks your types before the program ever runs. This rigidity is a feature, not a limitation. It catches errors early, makes code self-documenting, and enables the tooling that makes large-scale fintech systems maintainable. This section covers the building blocks you will use in every Java program you write for the rest of your career.

---

## Core Concepts

### Java Syntax Structure

Every Java source file contains one or more class declarations. The file name must match the public class name exactly.

```java
public class MyClass {
    // Fields (state)
    // Methods (behavior)
    // Constructors (initialization)
}
```

Rules:
- One public class per file.
- The file name must be `MyClass.java` if the public class is `MyClass`.
- Class names use **PascalCase** (also called UpperCamelCase).
- Every statement ends with a semicolon `;`.
- Blocks of code are enclosed in curly braces `{}`.
- Java is case-sensitive. `myVariable` and `MyVariable` are different identifiers.

### Comments

```java
// Single-line comment

/*
 * Multi-line comment.
 * Use sparingly. Prefer self-documenting code.
 */

/**
 * Javadoc comment. Used for public API documentation.
 * IntelliJ renders this as formatted help text.
 *
 * @param name the user's name
 * @return a greeting string
 */
```

Javadoc comments are not just decoration. They are processed by the `javadoc` tool to generate HTML documentation. In professional Java codebases, every public class and method has Javadoc.

### Variables and Assignment

A variable is a named storage location with a declared type.

```java
int age = 25;
String name = "Alice";
double balance = 1500.75;
boolean isActive = true;
```

Declaration and assignment can be separate:

```java
int count;      // Declaration (no value yet)
count = 10;     // Assignment
```

Java variables must be initialized before use. The compiler will reject code that reads an uninitialized local variable.

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Class | PascalCase | `UserService`, `TransactionProcessor` |
| Interface | PascalCase | `Serializable`, `Comparable` |
| Method | camelCase | `calculateInterest()`, `findById()` |
| Variable | camelCase | `accountBalance`, `userName` |
| Constant | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT` |
| Package | lowercase, dot-separated | `com.example.fintech.service` |
| Enum | PascalCase | `TransactionStatus`, `Currency` |
| Type parameter | Single uppercase letter | `T`, `E`, `K`, `V` |

These are not suggestions. They are enforced by checkstyle in professional codebases. Deviating from them will cause your pull requests to be rejected.

### Primitive Data Types

Java has eight primitive types. They are not objects. They are stored directly on the stack (for local variables) and have fixed sizes.

| Type | Size | Range | Default Value | Use Case |
|------|------|-------|---------------|----------|
| `byte` | 8 bits | -128 to 127 | 0 | Small numbers, raw byte data |
| `short` | 16 bits | -32,768 to 32,767 | 0 | Rarely used |
| `int` | 32 bits | -2.1 billion to 2.1 billion | 0 | Default integer type |
| `long` | 64 bits | -9.2 quintillion to 9.2 quintillion | 0L | Large counts, timestamps |
| `float` | 32 bits | ~6-7 decimal digits | 0.0f | Rarely used in business logic |
| `double` | 64 bits | ~15-16 decimal digits | 0.0d | Scientific calculations |
| `char` | 16 bits | 0 to 65,535 (Unicode) | '\u0000' | Single characters |
| `boolean` | 1 bit (JVM-dependent) | true or false | false | Flags, conditions |

**Literal suffixes:**

```java
long population = 7_900_000_000L;   // L suffix required for long literals
float rate = 3.14f;                  // f suffix required for float literals
double pi = 3.14159265358979;        // d suffix optional (double is default)
```

**Underscores in numeric literals (Java 7+):**

```java
int million = 1_000_000;
long creditCard = 4111_1111_1111_1111L;
double amount = 1_500.75;
int binary = 0b1010_0101;
int hex = 0xFF_EC_DE_5E;
```

Underscores are ignored by the compiler. They exist solely for readability. Use them for large numbers.

**Critical warning about float and double for money:**

Never use `float` or `double` for financial calculations. Floating-point arithmetic produces rounding errors that are unacceptable in fintech.

```java
double a = 0.1;
double b = 0.2;
System.out.println(a + b);
// Output: 0.30000000000000004  (NOT 0.3)
```

Use `BigDecimal` for all monetary values. This is covered in Phase 01.17 and Phase 08.

### Wrapper Classes

Each primitive type has a corresponding object wrapper class in `java.lang`.

| Primitive | Wrapper | Example |
|-----------|---------|---------|
| `byte` | `Byte` | `Byte.valueOf((byte) 1)` |
| `short` | `Short` | `Short.valueOf((short) 1)` |
| `int` | `Integer` | `Integer.valueOf(42)` |
| `long` | `Long` | `Long.valueOf(100L)` |
| `float` | `Float` | `Float.valueOf(3.14f)` |
| `double` | `Double` | `Double.valueOf(3.14)` |
| `char` | `Character` | `Character.valueOf('A')` |
| `boolean` | `Boolean` | `Boolean.valueOf(true)` |

**Why wrappers exist:**

- Java collections (`List`, `Map`, `Set`) can only store objects, not primitives. You cannot have a `List<int>`. You must use `List<Integer>`.
- Wrappers allow `null` values. A primitive `int` cannot be null. An `Integer` can. This matters for database fields that are nullable.
- Wrappers provide utility methods: `Integer.parseInt("42")`, `Double.isNaN(value)`.

**Autoboxing and unboxing:**

Java automatically converts between primitives and wrappers.

```java
// Autoboxing: primitive → wrapper
Integer boxed = 42;  // Compiler inserts Integer.valueOf(42)

// Unboxing: wrapper → primitive
int unboxed = boxed;  // Compiler inserts boxed.intValue()

// This works in collections
List<Integer> numbers = new ArrayList<>();
numbers.add(1);       // Autoboxing: int 1 → Integer.valueOf(1)
int first = numbers.get(0);  // Unboxing: Integer → int
```

**Autoboxing pitfalls:**

```java
// Pitfall 1: NullPointerException on unboxing
Integer nullable = null;
int value = nullable;  // Throws NullPointerException at runtime!

// Pitfall 2: Integer caching
Integer a = 127;
Integer b = 127;
System.out.println(a == b);  // true (cached)

Integer c = 128;
Integer d = 128;
System.out.println(c == d);  // false (not cached, different objects)

// ALWAYS use .equals() for wrapper comparison
System.out.println(c.equals(d));  // true
```

The JVM caches `Integer` values from -128 to 127. Outside that range, each autoboxing creates a new object. This is a common source of subtle bugs. Always use `.equals()` to compare wrapper objects, never `==`.

### Type Casting

**Widening (implicit, safe):**

Converting a smaller type to a larger type. No data loss. The compiler does this automatically.

```java
int intVal = 42;
long longVal = intVal;      // int → long (automatic)
double doubleVal = longVal;  // long → double (automatic)

// Widening order:
// byte → short → int → long → float → double
```

**Narrowing (explicit, potentially lossy):**

Converting a larger type to a smaller type. Requires an explicit cast. Data may be truncated.

```java
double pi = 3.14159;
int truncated = (int) pi;    // Result: 3 (decimal part lost)

long bigNumber = 3_000_000_000L;
int overflow = (int) bigNumber;  // Result: -1294967296 (overflow!)

double precise = 1_000_000_000.123;
float imprecise = (float) precise;  // Precision lost
```

Always be deliberate about narrowing casts. In fintech, an accidental cast from `long` to `int` on a transaction amount could silently corrupt data.

### Operators

**Arithmetic:**

```java
int a = 10, b = 3;
int sum = a + b;       // 13
int diff = a - b;      // 7
int product = a * b;   // 30
int quotient = a / b;  // 3 (integer division truncates)
int remainder = a % b; // 1 (modulo)

// Integer division warning:
double result = 7 / 2;     // 3.0 (integer division happens first!)
double correct = 7.0 / 2;  // 3.5 (at least one operand must be double)
```

**Increment and decrement:**

```java
int count = 5;
count++;   // count is now 6 (postfix)
count--;   // count is now 5
++count;   // count is now 6 (prefix)

// Difference matters in expressions:
int x = 5;
int y = x++;  // y = 5, x = 6 (postfix: use then increment)
int z = ++x;  // z = 7, x = 7 (prefix: increment then use)
```

**Comparison:**

```java
int a = 10, b = 20;
boolean eq = (a == b);   // false (equal to)
boolean ne = (a != b);   // true (not equal to)
boolean gt = (a > b);    // false (greater than)
boolean lt = (a < b);    // true (less than)
boolean ge = (a >= b);   // false (greater than or equal)
boolean le = (a <= b);   // true (less than or equal)
```

**Logical:**

```java
boolean x = true, y = false;
boolean and = x && y;   // false (short-circuit AND)
boolean or = x || y;    // true (short-circuit OR)
boolean not = !x;       // false (NOT)

// Short-circuit evaluation:
// && does not evaluate the right side if the left is false
// || does not evaluate the right side if the left is true
String name = null;
boolean safe = (name != null && name.length() > 0);  // No NPE
```

**Assignment:**

```java
int x = 10;
x += 5;   // x = x + 5  → 15
x -= 3;   // x = x - 3  → 12
x *= 2;   // x = x * 2  → 24
x /= 4;   // x = x / 4  → 6
x %= 4;   // x = x % 4  → 2
```

**Bitwise (used less frequently but important for low-level operations):**

```java
int a = 0b1010;  // 10
int b = 0b1100;  // 12
int and = a & b;   // 0b1000 = 8
int or = a | b;    // 0b1110 = 14
int xor = a ^ b;   // 0b0110 = 6
int not = ~a;      // 0b...0101 = -11 (two's complement)
int left = a << 1; // 0b10100 = 20 (multiply by 2)
int right = a >> 1;// 0b0101 = 5 (divide by 2)
```

**Ternary:**

```java
int age = 20;
String status = (age >= 18) ? "adult" : "minor";
// Equivalent to:
// if (age >= 18) status = "adult"; else status = "minor";
```

### Strings

Strings are the most commonly used reference type in Java. They are **immutable** — once created, a String object cannot be changed. Every "modification" creates a new String object.

**Creation:**

```java
String literal = "Hello";           // String pool
String object = new String("Hello"); // Heap (avoid this)
String empty = "";
String blank = "   ";
```

**String pool:**

String literals are stored in a special memory area called the **string pool**. Identical literals share the same object to save memory.

```java
String a = "Hello";
String b = "Hello";
System.out.println(a == b);  // true (same object in pool)

String c = new String("Hello");
System.out.println(a == c);  // false (different object on heap)
System.out.println(a.equals(c));  // true (same content)
```

Always use `.equals()` to compare string content. Using `==` compares object references, not content.

**Essential String methods:**

```java
String s = "  Hello, World!  ";

// Length and access
s.length();            // 17
s.charAt(0);           // ' ' (first character)
s.isEmpty();           // false
s.isBlank();           // false (true if only whitespace)

// Searching
s.contains("World");   // true
s.startsWith("  He");  // true
s.endsWith("!  ");     // true
s.indexOf("World");    // 9
s.lastIndexOf("o");    // 12

// Extraction
s.substring(9, 14);    // "World"
s.strip();             // "Hello, World!" (removes leading/trailing whitespace)
s.trim();              // "Hello, World!" (older method, strip() is preferred)
s.stripLeading();      // "Hello, World!  "
s.stripTrailing();     // "  Hello, World!"

// Transformation
s.toUpperCase();       // "  HELLO, WORLD!  "
s.toLowerCase();       // "  hello, world!  "
s.replace("World", "Java");  // "  Hello, Java!  "
s.replace('l', 'L');   // "  HeLLo, WorLd!  "

// Splitting and joining
"one,two,three".split(",");       // ["one", "two", "three"]
String.join("-", "a", "b", "c");  // "a-b-c"

// Formatting
String.format("Balance: $%.2f", 1500.75);  // "Balance: $1500.75"
"Balance: $%.2f".formatted(1500.75);       // "Balance: $1500.75" (Java 15+)
```

**String concatenation:**

```java
String first = "Hello";
String second = "World";

// + operator (fine for simple cases)
String result = first + ", " + second + "!";

// StringBuilder (use in loops or complex concatenation)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append("item").append(i).append(",");
}
String csv = sb.toString();

// String.join (for joining collections)
List<String> names = List.of("Alice", "Bob", "Charlie");
String joined = String.join(", ", names);  // "Alice, Bob, Charlie"
```

**Text blocks (Java 13+):**

```java
String json = """
        {
            "name": "Alice",
            "balance": 1500.75,
            "active": true
        }
        """;
// Produces a properly indented multi-line string.
// Essential for SQL queries, JSON templates, and HTML in tests.

String sql = """
        SELECT u.name, a.balance
        FROM users u
        JOIN accounts a ON u.id = a.user_id
        WHERE a.balance > ?
        ORDER BY a.balance DESC
        """;
```

### Input and Output

**Console output:**

```java
System.out.println("Hello");          // Prints with newline
System.out.print("Hello");            // Prints without newline
System.out.printf("Name: %s, Age: %d%n", "Alice", 25);  // Formatted

// Format specifiers:
// %s = string
// %d = integer
// %f = floating-point
// %.2f = floating-point with 2 decimal places
// %n = platform-independent newline
// %b = boolean
// %x = hexadecimal
```

**Console input:**

```java
import java.util.Scanner;

Scanner scanner = new Scanner(System.in);

System.out.print("Enter your name: ");
String name = scanner.nextLine();

System.out.print("Enter your age: ");
int age = scanner.nextInt();

System.out.print("Enter your balance: ");
double balance = scanner.nextDouble();

scanner.close();  // Always close when done
```

`Scanner` is fine for learning and simple CLI tools. In production backend applications, input comes from HTTP requests, message queues, and files — not the console.

### The `var` Keyword (Java 10+)

Local variable type inference allows the compiler to deduce the type from the initializer.

```java
var name = "Alice";          // Compiler infers String
var age = 25;                // Compiler infers int
var balance = 1500.75;       // Compiler infers double
var users = new ArrayList<String>();  // Compiler infers ArrayList<String>

// Rules:
// - Only for local variables (not fields, parameters, or return types)
// - Must have an initializer
// - The type is fixed at compile time (not dynamic like Python)
// - Cannot be null without a cast: var x = (String) null;

// Good use: reduces noise for obvious types
var transactionMap = new HashMap<String, List<Transaction>>();

// Bad use: obscures the type
var result = service.process(data);  // What type is result?
```

Use `var` when the type is obvious from the right-hand side. Avoid it when the type is not immediately clear to the reader.

### Constants

Constants are declared with the `final` keyword. By convention, they use UPPER_SNAKE_CASE.

```java
public class PaymentConstants {
    public static final int MAX_RETRY_COUNT = 3;
    public static final BigDecimal MINIMUM_TRANSFER = new BigDecimal("0.01");
    public static final String DEFAULT_CURRENCY = "USD";
    public static final Duration TIMEOUT = Duration.ofSeconds(30);
}

// Usage:
if (retries >= PaymentConstants.MAX_RETRY_COUNT) {
    throw new PaymentFailedException("Max retries exceeded");
}
```

`final` on a variable means the reference cannot be reassigned. For objects, the internal state can still change.

```java
final List<String> names = new ArrayList<>();
names.add("Alice");   // Allowed (modifying the object)
names = new ArrayList<>();  // Compilation error (reassigning the reference)
```

---

## Code Examples

**A complete program demonstrating all syntax basics:**

```java
package com.example.basics;

public class SyntaxDemo {

    // Constants
    private static final double TAX_RATE = 0.08;
    private static final String CURRENCY = "USD";

    public static void main(String[] args) {
        // Primitives
        int quantity = 5;
        double unitPrice = 29.99;
        boolean isTaxable = true;
        char grade = 'A';

        // Arithmetic
        double subtotal = quantity * unitPrice;
        double tax = isTaxable ? subtotal * TAX_RATE : 0.0;
        double total = subtotal + tax;

        // String formatting
        String receipt = """
                === RECEIPT ===
                Item:     Widget
                Qty:      %d
                Price:    $%.2f
                Subtotal: $%.2f
                Tax:      $%.2f
                Total:    $%.2f
                Currency: %s
                ===============
                """.formatted(quantity, unitPrice, subtotal, tax, total, CURRENCY);

        System.out.println(receipt);

        // Type casting
        int roundedTotal = (int) Math.round(total);
        System.out.println("Rounded: $" + roundedTotal);

        // String operations
        String customerName = "  alice smith  ";
        String normalized = customerName.strip().toLowerCase();
        String displayName = normalized.substring(0, 1).toUpperCase()
                + normalized.substring(1);
        System.out.println("Customer: " + displayName);

        // Wrapper classes
        Integer nullableAge = null;
        if (nullableAge != null) {
            int age = nullableAge;  // Safe unboxing
            System.out.println("Age: " + age);
        } else {
            System.out.println("Age not provided");
        }

        // var keyword
        var items = new java.util.ArrayList<String>();
        items.add("Widget");
        items.add("Gadget");
        System.out.println("Items: " + String.join(", ", items));
    }
}
```

**Output:**

```
=== RECEIPT ===
Item:     Widget
Qty:      5
Price:    $29.99
Subtotal: $149.95
Tax:      $12.00
Total:    $161.95
Currency: USD
===============

Rounded: $162
Customer: Alice smith
Age not provided
Items: Widget, Gadget
```

**Demonstrating common pitfalls:**

```java
public class PitfallDemo {
    public static void main(String[] args) {
        // Pitfall 1: Integer division
        double wrong = 7 / 2;
        double right = 7.0 / 2;
        System.out.println("Wrong: " + wrong);   // 3.0
        System.out.println("Right: " + right);   // 3.5

        // Pitfall 2: Floating-point comparison
        double a = 0.1 + 0.2;
        System.out.println(a == 0.3);  // false!
        System.out.println(Math.abs(a - 0.3) < 1e-9);  // true (correct approach)

        // Pitfall 3: String comparison with ==
        String s1 = new String("hello");
        String s2 = new String("hello");
        System.out.println(s1 == s2);       // false (different objects)
        System.out.println(s1.equals(s2));  // true (same content)

        // Pitfall 4: Integer cache boundary
        Integer x = 127;
        Integer y = 127;
        Integer m = 128;
        Integer n = 128;
        System.out.println(x == y);  // true (cached)
        System.out.println(m == n);  // false (not cached)

        // Pitfall 5: Narrowing overflow
        long bigAmount = 3_000_000_000L;
        int truncated = (int) bigAmount;
        System.out.println("Original: " + bigAmount);     // 3000000000
        System.out.println("Truncated: " + truncated);    // -1294967296

        // Pitfall 6: Null unboxing
        Integer nullable = null;
        try {
            int value = nullable;  // NullPointerException!
        } catch (NullPointerException e) {
            System.out.println("Caught NPE from null unboxing");
        }
    }
}
```

---

## Important Notes

- Java's static type system is your safety net. The compiler catches type mismatches before your code ever runs. This is why Java is preferred for fintech: a type error in a payment calculation is caught at compile time, not at 2 AM in production.
- Never use `float` or `double` for monetary values. Floating-point representation cannot exactly represent most decimal fractions. Use `BigDecimal` for all financial calculations. This is non-negotiable in fintech.
- Always use `.equals()` to compare objects (Strings, Integers, etc.). The `==` operator compares memory addresses for objects, not content. This is the single most common beginner bug in Java.
- The string pool is a memory optimization. String literals with identical content share the same object. Strings created with `new String()` are always separate objects on the heap. In production code, always use string literals, never `new String()`.
- StringBuilder is essential when concatenating strings in loops. The `+` operator creates a new String object on every iteration, which generates garbage and slows down the application. For a loop of 10,000 iterations, StringBuilder can be 100x faster.
- The `var` keyword does not make Java dynamically typed. The type is still determined at compile time and cannot change. `var x = 10;` makes `x` an `int` forever. You cannot later assign a String to it.
- Integer caching (-128 to 127) is a JVM optimization specified by the Java Language Specification. It exists to reduce memory usage for commonly used small integers. Do not rely on it for correctness. Always use `.equals()`.
- The `final` keyword on a variable prevents reassignment, not mutation. A `final List` can still have items added or removed. If you need true immutability, use `List.of()` or `Collections.unmodifiableList()`.
- Text blocks (triple-quoted strings) are one of the most useful modern Java features. Use them for SQL queries, JSON payloads, HTML templates, and multi-line log messages. They eliminate the need for string concatenation with `+` and `\n` escape sequences.
- Underscores in numeric literals are purely cosmetic. The compiler strips them. Use them to make large numbers readable: `1_000_000` is clearer than `1000000`.

---

## Practice

1. Write a program that declares variables of all eight primitive types, assigns values to each, and prints them. Include a `long` timestamp using `System.currentTimeMillis()`.
2. Write a program that demonstrates autoboxing and unboxing. Create an `Integer` from an `int`, add it to a `List<Integer>`, retrieve it, and unbox it back to `int`. Then demonstrate the NullPointerException that occurs when unboxing a null `Integer`.
3. Write a program that calculates the total cost of a purchase including tax. Use `double` for the calculation. Then rewrite it using `BigDecimal`. Compare the results for a purchase of 3 items at $19.99 each with 8.25% tax. Note the difference.
4. Write a program that takes a full name as a single string (e.g., `"  john   doe  "`), normalizes it (strips whitespace, proper capitalization), and prints the result. Use only String methods — no external libraries.
5. Write a program that demonstrates all six pitfalls from the `PitfallDemo` example. Add comments explaining why each pitfall occurs and how to avoid it.
6. Create a program using text blocks to generate a formatted invoice. Include item name, quantity, unit price, tax, and total. Use `String.formatted()` to insert the values.
7. In your Obsidian vault, create a reference table of all primitive types with their sizes, ranges, and default values. You will refer to this frequently.

---

## References

- Java Language Specification — Chapter 4 (Types, Values, and Variables): https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html
- Java Language Specification — Chapter 15 (Expressions): https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html
- Oracle Tutorial — Primitive Data Types: https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html
- JEP 286 (Local-Variable Type Inference): https://openjdk.org/jeps/286
- JEP 378 (Text Blocks): https://openjdk.org/jeps/378
- "Effective Java" by Joshua Bloch — Items 6 (Avoid creating unnecessary objects) and 61 (Use primitive types in preference to boxed primitives)
