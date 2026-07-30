**Phase:** Level 1 - Java Fundamentals  
**Date Studied:**  

---
## What Problem Does This Solve?
Every program works with data.  
A banking app works with balances, account numbers, names.  
An e-commerce app works with prices, quantities, product names.  
A social app works with usernames, post content, timestamps.

```
The question is:
  How does your program STORE data while it's running?
  How does it KNOW what kind of data it's dealing with?
  How does it PROTECT data from being used the wrong way?

Without variables:
  You can't remember anything between lines of code.
  Every calculation starts from scratch.
  Impossible to write any real program.

Without types:
  Nothing stops you from dividing a name by a price.
  Nothing stops you from storing "hello" where a number should be.
  Bugs appear at the worst possible moment (in production).

Java's solution:
  VARIABLES → named storage boxes in memory
  TYPES → what kind of data each box can hold
  STRONG TYPING → compiler prevents the wrong type going in the wrong box
```
This is why Java catches more bugs at compile time than  
Python or JavaScript - because every variable has a declared type  
that is enforced strictly.

---
## What Is a Variable?
```
A variable is a NAMED LOCATION in memory that holds a value.

Think of it like a labeled box:
  ┌─────────────┐
  │    age      │  ← the LABEL (variable name)
  │    ─────    │
  │     21      │  ← the VALUE stored inside
  │             │
  │   (int)     │  ← the TYPE (what kind of value fits)
  └─────────────┘

Three things define a variable:
  1. TYPE  → what kind of data can be stored
  2. NAME  → how you refer to it in code
  3. VALUE → the actual data stored right now

In memory (simplified):
  Memory Address 0x1A3F → stores value 21
  You don't say "give me address 0x1A3F"
  You say "give me age"
  Java handles the memory address for you.
  The variable NAME is your human-friendly label.
```

### Variable Declaration vs Initialization
```
// DECLARATION: create the variable, tell Java the type and name
int age;
// Box exists in memory. Has a label "age". But has NO value yet.
// Using it here would cause compile error: "variable might not be initialized"

// INITIALIZATION: give it a value for the first time
age = 21;
// Now the box has value 21.

// DECLARATION + INITIALIZATION together (most common way)
int score = 100;
// Create box AND put 100 in it at the same time.

// ASSIGNMENT: change the value after initialization
age = 22; // put 22 in the box (21 is gone)
score = score + 10; // read score (100), add 10, store 110

// Multiple declarations of same type (comma-separated)
int x = 1, y = 2, z = 3;
// Creates THREE separate variables
// Generally avoid this — one variable per line is clearer

// FINAL: makes a variable a constant (value cannot change)
final int MAX_USERS = 1000;
MAX_USERS = 2000; // COMPILE ERROR: cannot assign to final variable
// Use UPPER_SNAKE_CASE for constants (Java convention)
// final = "this box is sealed — value locked forever"
```

---
## Java's Type System - The Full Picture
```
Java has TWO categories of types:

┌─────────────────────────────────────────────────────────┐
│                   JAVA TYPES                            │
│                                                         │
│  ┌───────────────────────┐  ┌───────────────────────┐   │
│  │   PRIMITIVE TYPES     │  │   REFERENCE TYPES     │   │
│  │                       │  │                       │   │
│  │  8 built-in types     │  │  Classes, Interfaces  │   │
│  │  Store value DIRECTLY │  │  Store a REFERENCE    │   │
│  │  (address in memory)  │  │  (pointer to object)  │   │
│  │                       │  │                       │   │
│  │  byte    short        │  │  String               │   │
│  │  int     long         │  │  Arrays               │   │
│  │  float   double       │  │  User                 │   │
│  │  boolean char         │  │  ArrayList            │   │
│  │                       │  │  Every class you write│   │
│  │  Live on STACK        │  │  Objects live on HEAP │   │
│  │  Fixed size in memory │  │  Variable size        │   │
│  │  Cannot be null       │  │  CAN be null          │   │
│  │  lowercase names      │  │  Capitalized names    │   │
│  └───────────────────────┘  └───────────────────────┘   │
└─────────────────────────────────────────────────────────┘

CRITICAL DIFFERENCE:
  int x = 5;
  → x IS the value 5. Stored directly. On the stack.
  
  String s = "Hello";
  → s is a REFERENCE (memory address).
  → The String object is on the HEAP.
  → s points to it.
  → s IS NOT "Hello" — it's an ADDRESS that leads to "Hello"

This difference explains why:
  int a = 5; int b = a; b = 10;
  → a is still 5 (copy of value was made)
  
  String s1 = new String("hi"); String s2 = s1; s2 = "bye";
  → s1 still points to "hi" (s2's reference changed, not the object)
  
  We'll explore this deeply in Level 1.9 (Strings) and Level 1.10 (OOP)
```

---
## The 8 Primitive Types - Complete Guide

### Integer Types (Whole Numbers)
```java
public class IntegerTypes {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // BYTE
        // Size: 8 bits (1 byte) in memory
        // Range: -128 to 127
        // Use: raw binary data, file I/O, saving memory in arrays
        // ─────────────────────────────────────────
        byte smallNumber = 100;
        byte maxByte = 127;
        byte minByte = -128;
        // byte overflow = 128; // COMPILE ERROR: out of range
        
        // Real use: reading bytes from files/network
        // byte[] fileData = new byte[1024]; // 1KB buffer
        
        System.out.println("byte max: " + maxByte);   // 127
        System.out.println("byte min: " + minByte);   // -128
        
        // ─────────────────────────────────────────
        // SHORT
        // Size: 16 bits (2 bytes) in memory
        // Range: -32,768 to 32,767
        // Use: rarely used in modern Java
        //      occasionally in audio processing, embedded systems
        // ─────────────────────────────────────────
        short smallerInt = 30000;
        short maxShort = 32767;
        
        System.out.println("short max: " + maxShort); // 32767
        
        // ─────────────────────────────────────────
        // INT ← THE DEFAULT INTEGER TYPE
        // Size: 32 bits (4 bytes) in memory
        // Range: -2,147,483,648 to 2,147,483,647 (~2.1 billion)
        // Use: MOST COMMON. Counts, IDs, quantities, loops
        //      Default integer literal type in Java
        // ─────────────────────────────────────────
        int age = 21;
        int quantity = 500;
        int score = -100;       // can be negative
        int maxInt = 2147483647;
        int minInt = -2147483648;
        
        // Integer overflow (no error — wraps around! DANGEROUS)
        int overflow = maxInt + 1;
        System.out.println(overflow); // -2147483648 (wrapped to min!)
        // This is a classic bug — use long if values might exceed int range
        
        // Underscores for readability (Java 7+)
        int million = 1_000_000;       // same as 1000000
        int creditCard = 1234_5678_9012_3456; // NOT valid for int (too big)
        
        System.out.println("million: " + million); // 1000000
        System.out.println("int max: " + Integer.MAX_VALUE); // 2147483647
        System.out.println("int min: " + Integer.MIN_VALUE); // -2147483648
        
        // ─────────────────────────────────────────
        // LONG ← FOR BIG NUMBERS
        // Size: 64 bits (8 bytes) in memory
        // Range: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807
        //        (~9.2 quintillion — essentially unlimited for most apps)
        // Use: Database IDs, timestamps, file sizes, money in cents
        //      Population counts, distances in space
        // MUST add L suffix to long literals
        // ─────────────────────────────────────────
        long bigNumber = 9_999_999_999L;      // L suffix required!
        long timestamp = 1705312800000L;       // milliseconds since 1970
        long fileSize = 10_737_418_240L;       // 10 GB in bytes
        
        // WITHOUT L suffix: error or wrong behavior
        // long bad = 9999999999; // COMPILE ERROR: int literal too large
        // long tricky = 2147483647 + 1; // WRONG: overflow happens in int!
        // long correct = 2147483647L + 1L; // correct: 2147483648
        
        System.out.println("Timestamp: " + timestamp);
        System.out.println("long max: " + Long.MAX_VALUE);
        
        // WHICH INTEGER TYPE TO USE?
        // int  → default choice for whole numbers (99% of cases)
        // long → when values exceed ~2 billion:
        //        database IDs (auto-increment can exceed int)
        //        timestamps (milliseconds since epoch)
        //        financial calculations (large amounts)
        //        user IDs in large systems (Twitter uses long IDs)
        // byte → binary data, raw file I/O, memory-critical large arrays
        // short → almost never in modern Java
    }
}
```

### Floating Point Types (Decimal Numbers)
```java
public class FloatingPointTypes {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // FLOAT
        // Size: 32 bits (4 bytes)
        // Precision: ~7 significant decimal digits
        // Range: ±3.4 × 10^38
        // Use: RARELY in modern Java
        //      Graphics/game engines (memory efficiency)
        //      Machine learning (where precision < speed)
        // MUST add F or f suffix to float literals
        // ─────────────────────────────────────────
        float temperature = 36.5f;    // f suffix required
        float price = 9.99f;
        float pi = 3.14159f;
        
        System.out.println("float pi: " + pi); // 3.14159 (approximately)
        
        // ─────────────────────────────────────────
        // DOUBLE ← THE DEFAULT DECIMAL TYPE
        // Size: 64 bits (8 bytes)
        // Precision: ~15-16 significant decimal digits
        // Range: ±1.7 × 10^308
        // Use: MOST COMMON decimal type.
        //      Scientific calculations, coordinates, measurements
        //      Default floating point literal type in Java
        // ─────────────────────────────────────────
        double salary = 75000.50;
        double gpa = 3.1415926535;
        double earthRadius = 6_371_000.0; // meters
        double avogadro = 6.022e23;       // scientific notation
        double tiny = 1.5e-10;            // very small number
        
        System.out.println("salary: " + salary);
        System.out.println("GPA: " + gpa);
        System.out.println("Avogadro: " + avogadro); // 6.022E23
        
        // FLOATING POINT PRECISION PROBLEM ← CRITICAL TO KNOW
        double a = 0.1;
        double b = 0.2;
        double sum = a + b;
        System.out.println(sum); // 0.30000000000000004 ← NOT 0.3!
        
        // WHY? Floating point numbers are stored in binary.
        // 0.1 cannot be represented exactly in binary.
        // Just like 1/3 cannot be written exactly in decimal.
        // This is a fundamental limitation of IEEE 754 floating point.
        
        // CONSEQUENCES:
        // NEVER use double for money/financial calculations!
        // 0.1 + 0.2 ≠ 0.3 in floating point!
        
        // ❌ WRONG way to handle money:
        double price1 = 9.99;
        double price2 = 0.01;
        double total = price1 + price2;
        System.out.println(total); // 9.999999999999998 ← WRONG!
        
        // ✅ RIGHT ways to handle money:
        // Option 1: Store as LONG (cents, not dollars)
        long priceInCents1 = 999;   // $9.99
        long priceInCents2 = 1;     // $0.01
        long totalCents = priceInCents1 + priceInCents2; // 1000 = $10.00
        System.out.println("Total: $" + totalCents / 100.0); // $10.0 ✓
        
        // Option 2: Use BigDecimal (Level 2 topic — exact decimal math)
        // BigDecimal bd1 = new BigDecimal("9.99");
        // BigDecimal bd2 = new BigDecimal("0.01");
        // BigDecimal bdTotal = bd1.add(bd2); // exactly 10.00
        
        // Comparing doubles (don't use ==)
        double x = 0.1 + 0.2;
        // BAD: x == 0.3 → false (due to precision error)
        // GOOD: check if difference is tiny enough
        double epsilon = 1e-10; // very small tolerance
        boolean isEqual = Math.abs(x - 0.3) < epsilon;
        System.out.println("Is equal: " + isEqual); // true
        
        // Special double values
        double posInfinity = Double.POSITIVE_INFINITY; // 1.0/0.0
        double negInfinity = Double.NEGATIVE_INFINITY; // -1.0/0.0
        double notANumber  = Double.NaN;               // 0.0/0.0
        
        System.out.println(1.0 / 0.0);   // Infinity (no exception!)
        System.out.println(-1.0 / 0.0);  // -Infinity
        System.out.println(0.0 / 0.0);   // NaN
        System.out.println(Double.isNaN(notANumber));     // true
        System.out.println(Double.isInfinite(posInfinity)); // true
        
        // Integer division vs floating point division ← TRAP!
        int p = 7;
        int q = 2;
        System.out.println(p / q);       // 3 (integer division, truncates!)
        System.out.println(p / 2.0);     // 3.5 (double division)
        System.out.println((double)p / q); // 3.5 (cast to double first)
        // RULE: if BOTH operands are int → result is int (truncated)
        // If EITHER operand is double → result is double
    }
}
```

### Boolean Type
```java
public class BooleanType {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // BOOLEAN
        // Size: JVM-dependent (usually 1 bit conceptually, 1 byte actual)
        // Values: ONLY true or false (nothing else)
        // Use: conditions, flags, state tracking
        //      Most common use: if statements, while loops
        // ─────────────────────────────────────────
        boolean isLoggedIn = true;
        boolean hasPermission = false;
        boolean isAdult = true;
        
        // Boolean from expressions (very common)
        int age = 20;
        boolean canVote = age >= 18;       // true
        boolean isChild = age < 12;        // false
        boolean isTeen = age >= 13 && age <= 19; // false (20 is not teen)
        
        System.out.println("Can vote: " + canVote);       // true
        System.out.println("Is child: " + isChild);       // false
        System.out.println("Is teen: " + isTeen);         // false
        
        // NOT operator (!) flips the boolean
        System.out.println(!isLoggedIn);   // false
        System.out.println(!hasPermission); // true
        
        // Common naming convention for booleans: is, has, can, should
        // isActive, hasRole, canDelete, shouldRetry
        
        // Boolean in conditions
        if (isLoggedIn && hasPermission) {
            System.out.println("Access granted");
        } else if (isLoggedIn && !hasPermission) {
            System.out.println("Logged in but no permission");
        } else {
            System.out.println("Please log in");
        }
        
        // WARNING: boolean ≠ Boolean (capital B)
        // boolean → primitive (cannot be null)
        // Boolean → wrapper class (CAN be null — reference type)
        boolean primitive = true;
        Boolean wrapper = true;   // auto-boxed from primitive
        Boolean nullable = null;  // valid for wrapper type!
        
        // Null Boolean can cause NullPointerException:
        // boolean result = nullable; // EXCEPTION at runtime!
        
        // In Spring Boot / JPA entities:
        // Use Boolean (wrapper) for database columns that can be NULL
        // Use boolean (primitive) when value must always be present
    }
}
```

### Character Type
```java
public class CharType {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // CHAR
        // Size: 16 bits (2 bytes)
        // Represents: A SINGLE Unicode character
        // Range: '\u0000' to '\uFFFF' (0 to 65,535)
        // Use: individual characters in text processing
        //      rare in modern Java (String is used instead)
        // NOTE: uses SINGLE quotes ' ' (not double quotes " ")
        // ─────────────────────────────────────────
        char letter = 'A';
        char digit = '7';
        char space = ' ';
        char newline = '\n';  // escape sequence: newline character
        char tab = '\t';      // escape sequence: tab character
        char singleQuote = '\''; // escape sequence: single quote itself
        char backslash = '\\';   // escape sequence: backslash itself
        
        System.out.println(letter);    // A
        System.out.println(digit);     // 7
        
        // char is ACTUALLY a number internally (Unicode code point)
        char a = 'A';
        int numericValue = a;           // char auto-converts to int
        System.out.println(numericValue); // 65 ('A' = 65 in Unicode/ASCII)
        
        // Arithmetic on chars (works because char is numeric)
        char next = (char)(a + 1);     // cast back to char
        System.out.println(next);       // B ('A' + 1 = 'B')
        
        char nextNext = (char)('A' + 2);
        System.out.println(nextNext);   // C
        
        // Unicode characters
        char copyright = '\u00A9';  // ©
        char rupee = '\u20B9';      // ₹
        char omega = '\u03A9';      // Ω
        System.out.println(copyright); // ©
        System.out.println(rupee);     // ₹
        
        // Escape sequences you must know:
        // \n  → newline (next line)
        // \t  → tab (horizontal spacing)
        // \r  → carriage return (Windows line ending uses \r\n)
        // \\  → literal backslash
        // \"  → literal double quote (inside a String)
        // \'  → literal single quote (inside a char)
        // \0  → null character
        // \uXXXX → Unicode character by hex code
        
        System.out.println("Line1\nLine2");  // two lines
        System.out.println("Col1\tCol2");    // tab separated
        System.out.println("He said \"Hello\""); // He said "Hello"
        
        // Character utility methods (Character class)
        System.out.println(Character.isLetter('A'));    // true
        System.out.println(Character.isDigit('5'));     // true
        System.out.println(Character.isWhitespace(' ')); // true
        System.out.println(Character.isUpperCase('A')); // true
        System.out.println(Character.isLowerCase('a')); // true
        System.out.println(Character.toUpperCase('a')); // A
        System.out.println(Character.toLowerCase('A')); // a
    }
}
```

---
## Type Conversion - Widening and Narrowing
```java
public class TypeConversion {
    public static void main(String[] args) {
        
        // ══════════════════════════════════════════
        // WIDENING CONVERSION (automatic, implicit)
        // Going from SMALLER type to LARGER type
        // Safe — no data loss possible
        // Java does this automatically
        // ══════════════════════════════════════════
        
        // Widening chain: byte → short → int → long → float → double
        //                                      char ─┘
        
        byte b = 100;
        short s = b;      // byte → short (automatic) ✓
        int i = s;        // short → int (automatic) ✓
        long l = i;       // int → long (automatic) ✓
        float f = l;      // long → float (automatic) ✓
        double d = f;     // float → double (automatic) ✓
        
        System.out.println("byte: " + b);    // 100
        System.out.println("short: " + s);   // 100
        System.out.println("int: " + i);     // 100
        System.out.println("long: " + l);    // 100
        System.out.println("float: " + f);   // 100.0
        System.out.println("double: " + d);  // 100.0
        
        // int to double is automatic in expressions
        int count = 7;
        double average = count / 2.0; // count widens to double automatically
        System.out.println(average);  // 3.5
        
        // ══════════════════════════════════════════
        // NARROWING CONVERSION (manual, explicit cast)
        // Going from LARGER type to SMALLER type
        // UNSAFE — data loss POSSIBLE
        // Java REQUIRES explicit cast: (targetType)value
        // ══════════════════════════════════════════
        
        double price = 9.99;
        int truncated = (int) price;  // MUST cast explicitly
        System.out.println(truncated); // 9 (decimal part is LOST, not rounded!)
        
        long bigLong = 9_999_999_999L;
        int narrowed = (int) bigLong;  // value exceeds int range!
        System.out.println(narrowed);  // garbage value (data corrupted)
        // This is a DANGEROUS bug if you don't check range first
        
        double temp = 98.6;
        int tempInt = (int) temp;  // truncation, NOT rounding
        System.out.println(tempInt);  // 98 (not 99!)
        
        // To ROUND instead of truncate:
        int rounded = (int) Math.round(temp);
        System.out.println(rounded);  // 99 ✓
        
        // char and int conversions
        char ch = 'A';
        int charNum = ch;          // widening: char → int (automatic)
        System.out.println(charNum); // 65
        
        int num = 66;
        char backToChar = (char) num; // narrowing: int → char (explicit cast)
        System.out.println(backToChar); // B
        
        // Numeric string to primitive
        String numStr = "42";
        int parsed = Integer.parseInt(numStr);    // String → int
        double parsedDouble = Double.parseDouble("3.14"); // String → double
        long parsedLong = Long.parseLong("999999999999");  // String → long
        
        System.out.println(parsed + 1);       // 43 (now it's a number)
        System.out.println(parsedDouble + 1); // 4.140000000000001
        
        // String to number FAILURE
        try {
            int bad = Integer.parseInt("hello"); // throws NumberFormatException
        } catch (NumberFormatException e) {
            System.out.println("Cannot parse 'hello' as integer!");
        }
        
        // Primitive to String
        int number = 42;
        String str1 = String.valueOf(number);   // "42" ← preferred
        String str2 = Integer.toString(number); // "42" ← also works
        String str3 = "" + number;             // "42" ← works but avoid
        
        System.out.println(str1 + " is a String now"); // 42 is a String now
    }
}
```

---
## The `var` Keyword - Type Inference (Java 10+)
```java
public class VarKeyword {
    public static void main(String[] args) {
        
        // var = local variable type inference
        // Java INFERS the type from the right side
        // The type IS still set at compile time — Java just figures it out
        // This is NOT like JavaScript's var (which is dynamic)
        
        // Instead of:
        int count = 10;
        String name = "Rahim";
        double price = 9.99;
        
        // You can write:
        var count2 = 10;      // Java infers: int
        var name2 = "Rahim";  // Java infers: String
        var price2 = 9.99;    // Java infers: double
        
        // SAME BEHAVIOR — type is fixed at compile time
        count2 = 20;          // fine — still int
        // count2 = "hello";  // COMPILE ERROR — still int, cannot store String
        
        // var shines with complex types (more useful in Level 2+):
        // Without var:
        // ArrayList<HashMap<String, List<Integer>>> data = new ArrayList<>();
        // With var:
        // var data = new ArrayList<HashMap<String, List<Integer>>>();
        // Much cleaner! Same type, less typing.
        
        // RULES for var:
        // ✅ Local variables only (inside methods)
        // ✅ Must be initialized immediately (type must be inferrable)
        // ❌ Cannot be used for method parameters
        // ❌ Cannot be used for method return types
        // ❌ Cannot be used for class fields
        // ❌ Cannot be: var x; (no initialization — type unknown)
        // ❌ Cannot be: var x = null; (null has no type)
        
        // WHEN TO USE var:
        // ✅ When the type is OBVIOUS from the right side
        var list = new java.util.ArrayList<String>(); // obviously ArrayList
        var map = new java.util.HashMap<String, Integer>(); // obviously HashMap
        
        // ❌ When the type is NOT obvious:
        var result = someMethod(); // what type is this? unclear without looking
        
        // Rule of thumb:
        // Use var when it makes code MORE readable
        // Don't use var when it makes code LESS clear
        // When in doubt, write the full type
    }
    
    static String someMethod() { return "result"; }
}
```

---
## Variable Naming - Professional Standards
```java
public class NamingConventions {
    
    // ─────────────────────────────────────────────
    // CLASS FIELDS (instance and static)
    // ─────────────────────────────────────────────
    
    // camelCase for regular fields
    private String firstName;
    private int orderCount;
    private boolean isActive;
    
    // UPPER_SNAKE_CASE for constants (static final)
    private static final int MAX_RETRY_COUNT = 3;
    private static final String DEFAULT_CURRENCY = "BDT";
    private static final double TAX_RATE = 0.15;
    
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────────
        // LOCAL VARIABLES — camelCase
        // ─────────────────────────────────────────────
        
        // ✅ GOOD names (descriptive, camelCase, meaningful)
        int userAge = 21;
        String productName = "Laptop";
        double totalOrderAmount = 1500.00;
        boolean isEmailVerified = true;
        int maxAttempts = 3;
        
        // ❌ BAD names (avoid these)
        int a = 21;           // meaningless single letter
        int x1 = 100;         // numbered variables (smell!)
        String str = "Laptop"; // generic type name as variable
        double d = 1500.00;   // single letter
        boolean b = true;     // meaningless
        int MAX = 3;          // wrong case (MAX is for constants)
        int user_age = 21;    // snake_case (Python style, not Java)
        int UserAge = 21;     // PascalCase (for classes, not variables)
        
        // ─────────────────────────────────────────────
        // EXCEPTIONS (acceptable single letters)
        // ─────────────────────────────────────────────
        // These are WIDELY accepted conventions:
        int i = 0;  // loop counter (i, j, k for nested loops)
        int n = 10; // array size/limit
        // x, y, z for coordinates
        // e for exception in catch blocks
        // t for generic type parameters
        // But AVOID in non-loop contexts
        
        // ─────────────────────────────────────────────
        // BOOLEAN NAMING CONVENTION
        // ─────────────────────────────────────────────
        // Always name booleans as questions or states:
        boolean isAdmin = false;           // ✅ is + adjective
        boolean hasSubscription = true;    // ✅ has + noun
        boolean canUploadFiles = false;    // ✅ can + verb
        boolean shouldSendEmail = true;    // ✅ should + verb
        boolean wasProcessed = false;      // ✅ was + past participle
        
        // ❌ avoid:
        boolean admin = false;    // not clear it's boolean
        boolean active = true;    // ambiguous
        boolean flag = false;     // meaningless
        
        // ─────────────────────────────────────────────
        // REAL-WORLD EXAMPLES from backend code
        // ─────────────────────────────────────────────
        
        // User-related
        long userId = 1001L;
        String userEmail = "rahim@example.com";
        String hashedPassword = "$2a$10$...";
        boolean isEmailVerified2 = false;
        
        // Order-related
        long orderId = 5001L;
        double subtotalAmount = 1000.00;
        double discountAmount = 100.00;
        double taxAmount = 135.00;
        double totalAmount = subtotalAmount - discountAmount + taxAmount;
        
        // Pagination
        int pageNumber = 0;   // 0-indexed
        int pageSize = 20;
        int totalRecords = 1547;
        int totalPages = (int) Math.ceil((double) totalRecords / pageSize);
        
        System.out.println("Total pages: " + totalPages); // 78
        
        // Status flags
        boolean isPaymentSuccessful = true;
        boolean hasInventory = true;
        boolean shouldApplyDiscount = totalAmount > 500;
        
        System.out.println("Apply discount: " + shouldApplyDiscount); // true
    }
}
```

---

## Wrapper Classes - The Bridge Between Primitives and Objects
```java
public class WrapperClasses {
    public static void main(String[] args) {
        
        // Every primitive has a corresponding Wrapper Class:
        // byte    → Byte
        // short   → Short
        // int     → Integer   ← most important
        // long    → Long      ← very important
        // float   → Float
        // double  → Double    ← very important
        // boolean → Boolean
        // char    → Character
        
        // WHY DO WRAPPERS EXIST?
        // 1. Collections can only hold OBJECTS, not primitives:
        //    List<int> list = new ArrayList<>();    // COMPILE ERROR
        //    List<Integer> list = new ArrayList<>(); // ✓ wrappers work
        //
        // 2. Methods that need Objects (not primitives)
        //    void process(Object obj) → needs object, not primitive
        //
        // 3. Nullable values (primitive cannot be null)
        //    Integer age = null; // valid (database might not have age)
        //    int age = null;     // COMPILE ERROR (primitive can't be null)
        //
        // 4. Utility methods on the wrapper class
        
        // AUTOBOXING: Java automatically converts primitive → wrapper
        int primitive = 42;
        Integer wrapper = primitive;    // Java auto-boxes (wraps it)
        System.out.println(wrapper);    // 42
        
        // UNBOXING: Java automatically converts wrapper → primitive
        Integer wrappedNumber = 100;
        int unwrapped = wrappedNumber;  // Java auto-unboxes
        System.out.println(unwrapped);  // 100
        
        // This happens automatically in many places:
        java.util.List<Integer> numbers = new java.util.ArrayList<>();
        numbers.add(1);  // autoboxing: int 1 → Integer 1
        numbers.add(2);
        numbers.add(3);
        int first = numbers.get(0); // unboxing: Integer → int
        
        // ─────────────────────────────────────────────
        // CRITICAL: Null unboxing → NullPointerException
        // ─────────────────────────────────────────────
        Integer maybeNull = null;
        // int result = maybeNull; // RUNTIME EXCEPTION: NullPointerException!
        // Java tries to unbox null → crash
        
        // Always null-check wrappers before unboxing:
        if (maybeNull != null) {
            int result = maybeNull;
            System.out.println(result);
        } else {
            System.out.println("Value was null");
        }
        
        // ─────────────────────────────────────────────
        // USEFUL WRAPPER UTILITY METHODS
        // ─────────────────────────────────────────────
        
        // Integer utility methods
        System.out.println(Integer.MAX_VALUE);       // 2147483647
        System.out.println(Integer.MIN_VALUE);       // -2147483648
        System.out.println(Integer.parseInt("42"));  // String → int: 42
        System.out.println(Integer.toBinaryString(10)); // "1010"
        System.out.println(Integer.toHexString(255));   // "ff"
        System.out.println(Integer.toOctalString(8));   // "10"
        System.out.println(Integer.compare(5, 10));     // negative (5 < 10)
        System.out.println(Integer.max(5, 10));         // 10
        System.out.println(Integer.min(5, 10));         // 5
        System.out.println(Integer.sum(5, 10));         // 15
        
        // Double utility methods
        System.out.println(Double.parseDouble("3.14")); // 3.14
        System.out.println(Double.isNaN(0.0 / 0.0));   // true
        System.out.println(Double.isInfinite(1.0/0.0)); // true
        System.out.println(Double.MAX_VALUE);           // 1.7976931348623157E308
        
        // Boolean utility methods
        System.out.println(Boolean.parseBoolean("true"));  // true
        System.out.println(Boolean.parseBoolean("True"));  // true
        System.out.println(Boolean.parseBoolean("yes"));   // false! not "yes"
        System.out.println(Boolean.toString(true));         // "true"
        
        // Character utility methods (already seen in char section)
        System.out.println(Character.isLetter('A'));   // true
        System.out.println(Character.toUpperCase('z')); // Z
        
        // ─────────────────────────────────────────────
        // Integer CACHING (tricky interview topic!)
        // ─────────────────────────────────────────────
        // Java caches Integer objects for -128 to 127
        // This means the SAME object is reused in this range
        
        Integer a = 100;
        Integer b = 100;
        System.out.println(a == b);      // true! (same cached object)
        System.out.println(a.equals(b)); // true
        
        Integer c = 200;
        Integer d = 200;
        System.out.println(c == d);      // false! (200 > 127, new objects)
        System.out.println(c.equals(d)); // true
        
        // LESSON: NEVER use == to compare Integer objects
        // ALWAYS use .equals() for wrapper type comparison
    }
}
```

---
## Build This - Complete Variable Practice Program
```java
// File: VariablePractice.java
// Build this entirely yourself after reading the note
// This simulates a simple bank account using variables

public class VariablePractice {
    
    public static void main(String[] args) {
        
        // ═══════════════════════════════════
        // ACCOUNT SETUP
        // ═══════════════════════════════════
        
        // Account information
        long accountNumber = 1234_5678_9012L;  // long for large numbers
        String accountHolder = "Rahim Ahmed";
        String accountType = "Savings";
        boolean isActive = true;
        boolean isVerified = true;
        
        // Balance (using long cents to avoid floating point errors)
        long balanceInPoisha = 150_000_00L;  // 150,000 BDT in poisha
        // (1 BDT = 100 poisha, like dollars and cents)
        
        // Interest rate
        double annualInterestRate = 0.045;  // 4.5%
        
        // Transaction limits
        int maxDailyTransactions = 10;
        long maxTransactionAmount = 50_000_00L; // 50,000 BDT limit
        
        // ═══════════════════════════════════
        // DISPLAY ACCOUNT INFO
        // ═══════════════════════════════════
        
        System.out.println("═══════════════════════════════════════");
        System.out.println("         BANK ACCOUNT DETAILS          ");
        System.out.println("═══════════════════════════════════════");
        System.out.printf("Account Number : %d%n", accountNumber);
        System.out.printf("Account Holder : %s%n", accountHolder);
        System.out.printf("Account Type   : %s%n", accountType);
        System.out.printf("Status         : %s%n", isActive ? "Active" : "Inactive");
        System.out.printf("Verified       : %s%n", isVerified ? "Yes" : "No");
        System.out.printf("Balance        : %.2f BDT%n", balanceInPoisha / 100.0);
        System.out.printf("Interest Rate  : %.1f%%%n", annualInterestRate * 100);
        System.out.println("═══════════════════════════════════════");
        
        // ═══════════════════════════════════
        // PERFORM A DEPOSIT
        // ═══════════════════════════════════
        
        long depositAmountPoisha = 25_000_00L; // 25,000 BDT
        balanceInPoisha = balanceInPoisha + depositAmountPoisha;
        int transactionCount = 1;
        
        System.out.println("\n[DEPOSIT]");
        System.out.printf("Deposited: %.2f BDT%n", depositAmountPoisha / 100.0);
        System.out.printf("New Balance: %.2f BDT%n", balanceInPoisha / 100.0);
        
        // ═══════════════════════════════════
        // PERFORM A WITHDRAWAL
        // ═══════════════════════════════════
        
        long withdrawAmountPoisha = 10_000_00L; // 10,000 BDT
        boolean hasEnoughBalance = balanceInPoisha >= withdrawAmountPoisha;
        boolean withinDailyLimit = transactionCount < maxDailyTransactions;
        boolean withinAmountLimit = withdrawAmountPoisha <= maxTransactionAmount;
        boolean canWithdraw = hasEnoughBalance && withinDailyLimit && withinAmountLimit;
        
        System.out.println("\n[WITHDRAWAL ATTEMPT]");
        System.out.printf("Requested: %.2f BDT%n", withdrawAmountPoisha / 100.0);
        System.out.println("Sufficient balance: " + hasEnoughBalance);
        System.out.println("Within daily limit: " + withinDailyLimit);
        System.out.println("Within amount limit: " + withinAmountLimit);
        
        if (canWithdraw) {
            balanceInPoisha = balanceInPoisha - withdrawAmountPoisha;
            transactionCount++;
            System.out.println("Withdrawal successful!");
            System.out.printf("New Balance: %.2f BDT%n", balanceInPoisha / 100.0);
        } else {
            System.out.println("Withdrawal denied.");
        }
        
        // ═══════════════════════════════════
        // CALCULATE MONTHLY INTEREST
        // ═══════════════════════════════════
        
        double monthlyRate = annualInterestRate / 12;
        double interestPoisha = balanceInPoisha * monthlyRate;
        long interestRounded = (long) interestPoisha; // truncate (bank's favor)
        long balanceWithInterest = balanceInPoisha + interestRounded;
        
        System.out.println("\n[MONTHLY INTEREST]");
        System.out.printf("Monthly Rate   : %.4f%%%n", monthlyRate * 100);
        System.out.printf("Interest Earned: %.2f BDT%n", interestRounded / 100.0);
        System.out.printf("Balance After  : %.2f BDT%n", balanceWithInterest / 100.0);
        
        // ═══════════════════════════════════
        // SUMMARY
        // ═══════════════════════════════════
        
        System.out.println("\n═══════════════════════════════════════");
        System.out.println("              SUMMARY                  ");
        System.out.println("═══════════════════════════════════════");
        System.out.printf("Transactions Today: %d/%d%n", 
                          transactionCount, maxDailyTransactions);
        System.out.printf("Final Balance: %.2f BDT%n", 
                          balanceWithInterest / 100.0);
        System.out.println("═══════════════════════════════════════");
    }
}
```

---
## Exercises
```
EXERCISE 1: Type Explorer
  Create TypeExplorer.java
  For each of the 8 primitive types:
  - Declare a variable with a meaningful name
  - Print its value, max value, min value (where applicable)
  - Show what happens at the boundary (overflow)
  
EXERCISE 2: Conversion Calculator  
  Create ConversionCalc.java
  Build a unit converter:
  - km to miles (1 km = 0.621371 miles)
  - celsius to fahrenheit (F = C * 9/5 + 32)
  - BDT to USD (use a fake rate of 110.5)
  - bytes to KB, MB, GB
  Print all with proper formatting (printf)

EXERCISE 3: Floating Point Investigation
  Create FloatInvestigation.java
  Demonstrate the floating point problem:
  - 0.1 + 0.2 ≠ 0.3
  - Show the actual value
  - Show the epsilon comparison fix
  - Show why money should use long (poisha/cents)
  Write comments explaining WHAT and WHY

EXERCISE 4: The Naming Game
  Create NamingGame.java
  Write a program about a student:
  - Their info (name, age, cgpa, university, year)
  - Their goal (target company, target salary in BDT)
  - Their progress (topics learned as int count, 
    days studied, problems solved)
  - Boolean states (isEmployed, hasInternship, 
    hasGithubProfile, isLeetcodeActive)
  Print a formatted report
  Push to GitHub

EXERCISE 5: var vs explicit type
  Rewrite Exercise 4 using var for ALL variables
  where it makes sense (type is obvious from right side)
  Which ones should you NOT use var for? Comment why.
```

---
## Common Mistakes
```
MISTAKE 1: Forgetting L suffix on long literals
  long id = 9876543210;   // COMPILE ERROR: int literal too large
  long id = 9876543210L;  // ✓ correct

MISTAKE 2: Forgetting f suffix on float literals
  float temp = 36.6;      // COMPILE ERROR: double literal, can't assign to float
  float temp = 36.6f;     // ✓ correct

MISTAKE 3: Integer division when you want decimal
  int a = 7, b = 2;
  double result = a / b;      // 3.0 ← WRONG (integer division first!)
  double result2 = (double)a / b; // 3.5 ← correct

MISTAKE 4: Floating point for money
  double price = 9.99;
  double tax = price * 0.1;
  System.out.println(tax);    // 0.9990000000000001 ← WRONG
  // Use long (cents/poisha) or BigDecimal instead

MISTAKE 5: Using == for wrapper comparison
  Integer a = 200;
  Integer b = 200;
  a == b   // false! (different objects)
  a.equals(b) // true ✓

MISTAKE 6: Unboxing null wrapper
  Integer value = null;
  int x = value;  // NullPointerException at runtime!
  // Always null-check wrappers before unboxing

MISTAKE 7: Overflow without noticing
  int max = Integer.MAX_VALUE;
  int overflow = max + 1;  // no error, silently wraps to MIN_VALUE!
  // Use Math.addExact(max, 1) → throws ArithmeticException instead

MISTAKE 8: Using variables before initialization
  int count;
  System.out.println(count); // COMPILE ERROR: variable might not be initialized
  // Always initialize local variables before use

MISTAKE 9: Wrong naming convention
  int UserAge = 21;    // PascalCase → looks like a class name
  int user_age = 21;   // snake_case → Python style, not Java
  int USERAGE = 21;    // ALL CAPS → reserved for constants
  int userAge = 21;    // ✓ camelCase (correct!)
```

---

## Interview Questions
```
Q: What is the difference between int and Integer in Java?
A: int is a primitive type that stores the value directly in
   memory (on the stack for local variables). Integer is a
   wrapper class — it's an object that wraps an int value on
   the heap. Key differences: Integer can be null (useful for
   optional database fields), int cannot. Collections like
   ArrayList require objects, so you use List<Integer> not
   List<int>. Java autoboxes between them automatically, but
   unboxing a null Integer causes NullPointerException.
   Use int for local computations, Integer when you need
   an object or nullable value.

Q: Why should you not use double for monetary calculations?
A: Floating point numbers (float and double) use binary
   representation internally. Numbers like 0.1 cannot be
   represented exactly in binary — just like 1/3 cannot
   be written exactly in decimal. This causes precision
   errors: 0.1 + 0.2 = 0.30000000000000004 in Java.
   For money, use long (store amounts in the smallest unit
   like cents or poisha: ৳10 = 1000 poisha), or use
   BigDecimal which supports exact decimal arithmetic.

Q: What is the difference between widening and narrowing 
   type conversion?
A: Widening conversion goes from a smaller type to a larger
   type — like int to long or float to double. No data loss
   is possible, so Java does it automatically (implicit cast).
   Narrowing conversion goes from a larger type to a smaller
   type — like double to int. Data loss IS possible (the
   decimal part is truncated), so Java requires an explicit
   cast: int x = (int) 9.99; gives 9, not 10. You must
   explicitly cast to acknowledge you accept the potential loss.

Q: What happens when an int overflows in Java?
A: Java integer arithmetic wraps around silently — no exception.
   Integer.MAX_VALUE + 1 = Integer.MIN_VALUE (-2147483648).
   This is a dangerous silent bug. To detect overflow, use
   Math.addExact(), Math.multiplyExact() etc. which throw
   ArithmeticException on overflow. Or use long if values
   might exceed int range.

Q: What is autoboxing and unboxing?
A: Autoboxing is Java's automatic conversion of primitives to
   their wrapper objects: int → Integer, double → Double etc.
   Unboxing is the reverse: Integer → int. Java does this
   automatically when needed — for example when you add an
   int to a List<Integer>. The danger is unboxing a null
   wrapper: Integer x = null; int y = x; throws
   NullPointerException at runtime.

Q: When would you use byte or short instead of int?
A: In practice, almost never for regular code. byte is used
   for raw binary data: reading files (byte[] buffer),
   network I/O, binary protocols. short is rarely used in
   modern Java. The JVM internally promotes byte and short
   to int for arithmetic anyway, so they save memory only
   in large arrays. Use int as your default integer type
   and only reach for byte/short for specific binary data needs.
```

---

## Key Takeaways
```
1. Java has 8 primitive types and reference types.
   Primitives store VALUE directly (on stack).
   References store an ADDRESS to an object (on heap).

2. DEFAULT integer type: int (32-bit, ±2.1 billion)
   Use long for: database IDs, timestamps, big numbers.
   Always add L suffix: 9999999999L

3. DEFAULT decimal type: double (64-bit, ~15 digit precision)
   NEVER use for money — floating point precision errors.
   Store money as long (cents/poisha) or use BigDecimal.

4. boolean has ONLY two values: true or false.
   Name booleans as questions: isActive, hasRole, canEdit.

5. char is a 16-bit Unicode character.
   Uses single quotes: 'A'  (not "A")
   Strings use double quotes: "Hello"

6. Widening (small→large): automatic, safe, no cast needed.
   Narrowing (large→small): manual cast required, data may be lost.
   (int) 9.99 → 9 (truncated, not rounded!)

7. var infers type from right side. Type is still fixed.
   Only for local variables. Only when type is obvious.

8. Every primitive has a Wrapper class (int→Integer).
   Wrappers can be null. Primitives cannot.
   NEVER use == to compare wrapper objects. Use .equals().
   Unboxing null wrapper → NullPointerException at runtime.

9. Integer.MAX_VALUE + 1 wraps silently (no exception).
   Use Math.addExact() to detect overflow.

10. Naming conventions are mandatory in professional code:
    variables/methods → camelCase
    classes           → PascalCase
    constants         → UPPER_SNAKE_CASE
```

---