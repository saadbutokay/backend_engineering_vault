**Phase:** Level 1 - Java Fundamentals
**Date Studied:**

---
## What Problem Does This Solve?
```text
You covered String basics in Level 1.8.
But Strings are so fundamental to backend engineering
that they deserve their own deep dive.

Every single day as a backend engineer you will:
  → Validate user input (email, phone, password)
  → Parse incoming JSON, CSV, XML, URLs
  → Format output (API responses, reports, logs)
  → Search and manipulate text in databases
  → Build SQL queries (carefully — injection risk)
  → Process file contents line by line
  → Format error messages for logs
  → Build email templates
  → Sanitize data before storing

If you don't understand Strings deeply:
  → You'll write slow code (concatenation in loops)
  → You'll have subtle bugs (== instead of .equals())
  → You'll miss NPEs (calling methods on null Strings)
  → You'll write verbose code (not knowing the right method)
  → You'll fail technical interviews (String questions are VERY common)

This note fills every gap.
After this, Strings will never surprise you again.
```

---
## 1. String Memory - The Complete Picture

```java
public class StringMemory {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // HOW STRINGS ARE STORED
        // ─────────────────────────────────────────
        
        // In Java 9+, String stores characters as byte[] internally
        // (previously char[] in Java 8)
        // This is called "Compact Strings" — saves 50% memory
        // for ASCII text (most common case)
        
        // You don't interact with this directly — just know it exists.
        
        // ─────────────────────────────────────────
        // THE STRING POOL — COMPLETE EXPLANATION
        // ─────────────────────────────────────────
        
        // Pool lives in HEAP memory (Java 7+ — was in PermGen before)
        
        String a = "hello";   // "hello" created in pool
        String b = "hello";   // "hello" already in pool — REUSED
        String c = "hel" + "lo"; // compile-time constant → resolved to "hello" → pool
        String d = new String("hello"); // FORCED new object on HEAP (not pool)
        
        System.out.println("a == b: " + (a == b)); // true  (same pool object)
        System.out.println("a == c: " + (a == c)); // true  (compile-time constant)
        System.out.println("a == d: " + (a == d)); // false (d is separate heap object)
        
        System.out.println("a.equals(d): " + a.equals(d)); // true (same content)
        
        // Runtime concatenation is NOT in pool:
        String prefix = "hel";
        String suffix = "lo";
        String e = prefix + suffix; // runtime operation → new heap object
        
        System.out.println("a == e: " + (a == e)); // false (runtime concat)
        System.out.println("a.equals(e): " + a.equals(e)); // true
        
        // ─────────────────────────────────────────
        // String.intern() — manually pool a string
        // ─────────────────────────────────────────
        
        String f = (prefix + suffix).intern(); // add to pool if not there, return pool reference
        System.out.println("a == f: " + (a == f)); // true (f now points to pool object)
        
        // When to use intern():
        // → When you have MANY identical strings (saves memory)
        // → When you need reference equality check for performance
        // → Rarely needed in modern applications — JVM is smart
        
        // ─────────────────────────────────────────
        // IMMUTABILITY — WHAT IT REALLY MEANS
        // ─────────────────────────────────────────
        
        String original = "Hello";
        
        // NONE of these change 'original':
        original.toUpperCase();      // creates "HELLO" — discarded
        original.replace('l', 'r');  // creates "Herro" — discarded
        original.concat(" World");   // creates "Hello World" — discarded
        original.trim();             // creates "Hello" — discarded
        
        System.out.println(original); // still "Hello" — UNCHANGED
        
        // To "change" a String you MUST reassign:
        original = original.toUpperCase(); // original now points to "HELLO"
        System.out.println(original); // "HELLO"
        
        // But "Hello" still exists in the pool!
        // It's just not referenced by 'original' anymore.
        // Eventually eligible for garbage collection.
        
        // WHY IMMUTABILITY PROTECTS YOU:
        
        String password = "secret123";
        storePassword(password); // this method cannot change your password variable
        System.out.println(password); // still "secret123" — safe!
    }
    
    static void storePassword(String pwd) {
        // Even if we do something to pwd, the caller's variable is safe
        pwd = "changed"; // only changes local reference — not caller's variable
        // (This is pass-by-value — the REFERENCE is copied, not the object)
    }
}
```

---
## 2. String Methods - Complete Reference
```java
public class StringMethodsComplete {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // INFORMATION / INSPECTION
        // ─────────────────────────────────────────
        
        String s = "Hello, World! Hello, Java!";
        
        System.out.println(s.length());           // 26
        System.out.println(s.isEmpty());          // false
        System.out.println("".isEmpty());         // true
        System.out.println("  ".isEmpty());       // false (has spaces)
        System.out.println(s.isBlank());          // false
        System.out.println("  ".isBlank());       // true (Java 11+)
        System.out.println("".isBlank());         // true
        
        // ─────────────────────────────────────────
        // CHARACTER ACCESS
        // ─────────────────────────────────────────
        
        System.out.println(s.charAt(0));          // 'H'
        System.out.println(s.charAt(s.length()-1)); // '!'
        
        // Convert to char array:
        char[] chars = s.toCharArray();
        System.out.println(chars.length);         // 26
        
        // ─────────────────────────────────────────
        // SEARCHING — indexOf family
        // ─────────────────────────────────────────
        
        // indexOf returns -1 if NOT FOUND
        System.out.println(s.indexOf('H'));       // 0
        System.out.println(s.indexOf('z'));       // -1 (not found)
        System.out.println(s.indexOf("Hello"));  // 0
        System.out.println(s.indexOf("Hello", 1)); // 14 (search from index 1)
        System.out.println(s.lastIndexOf('H'));  // 14 (last occurrence)
        System.out.println(s.lastIndexOf("Hello")); // 14
        
        System.out.println(s.contains("Java")); // true
        System.out.println(s.contains("Python")); // false
        System.out.println(s.startsWith("Hello")); // true
        System.out.println(s.endsWith("Java!")); // true
        System.out.println(s.startsWith("World", 7)); // true (at index 7)
        
        // ─────────────────────────────────────────
        // COMPARISON
        // ─────────────────────────────────────────
        
        String a = "apple";
        String b = "Apple";
        
        System.out.println(a.equals(b));              // false
        System.out.println(a.equalsIgnoreCase(b));    // true
        System.out.println(a.compareTo(b));           // positive (a > A in ASCII)
        System.out.println(a.compareToIgnoreCase(b)); // 0 (equal ignoring case)
        
        // Lexicographic order: "apple" < "banana"
        System.out.println("apple".compareTo("banana")); // negative
        System.out.println("banana".compareTo("apple")); // positive
        
        // ─────────────────────────────────────────
        // EXTRACTION
        // ─────────────────────────────────────────
        
        String text = "Hello, World!";
        
        System.out.println(text.substring(7));       // "World!"
        System.out.println(text.substring(7, 12));   // "World" (7 inclusive, 12 exclusive)
        System.out.println(text.substring(0, 5));    // "Hello"
        
        // ─────────────────────────────────────────
        // CASE CONVERSION
        // ─────────────────────────────────────────
        
        String mixed = "Hello World Java";
        System.out.println(mixed.toUpperCase());     // "HELLO WORLD JAVA"
        System.out.println(mixed.toLowerCase());     // "hello world java"
        
        // Locale-aware (important for international apps):
        // String turkish = "istanbul";
        // turkish.toUpperCase(new Locale("tr")); // "İSTANBUL" (dotted I)
        // Use Locale.ROOT for programming identifiers (locale-independent)
        System.out.println("hello".toUpperCase(java.util.Locale.ROOT)); // "HELLO"
        
        // ─────────────────────────────────────────
        // WHITESPACE HANDLING
        // ─────────────────────────────────────────
        
        String padded = "  \t Hello \n World \t  ";
        System.out.println(padded.trim());          // "Hello \n World" (outer only)
        System.out.println(padded.strip());         // "Hello \n World" (Unicode-aware, Java 11)
        System.out.println(padded.stripLeading());  // "Hello \n World \t  "
        System.out.println(padded.stripTrailing()); // "  \t Hello \n World"
        
        // trim() vs strip():
        // trim() removes chars ≤ '\u0020' (ASCII space)
        // strip() removes all Unicode whitespace (better for non-ASCII input)
        // Use strip() in new code (Java 11+)
        
        // ─────────────────────────────────────────
        // REPLACEMENT
        // ─────────────────────────────────────────
        
        String sentence = "The cat sat on the cat mat with the cat";
        
        System.out.println(sentence.replace("cat", "dog"));
        // "The dog sat on the dog mat with the dog" (ALL occurrences)
        
        System.out.println(sentence.replaceFirst("cat", "dog"));
        // "The dog sat on the cat mat with the cat" (FIRST only)
        
        System.out.println(sentence.replaceAll("\\bcat\\b", "dog"));
        // replaceAll uses REGEX — \b = word boundary
        
        // Replace with char (all occurrences of that char):
        System.out.println("banana".replace('a', 'o')); // "bonono"
        
        // ─────────────────────────────────────────
        // SPLITTING
        // ─────────────────────────────────────────
        
        String csv = "Java,Spring,PostgreSQL,Redis,Docker";
        String[] parts = csv.split(",");
        System.out.println(java.util.Arrays.toString(parts));
        // [Java, Spring, PostgreSQL, Redis, Docker]
        
        // Split limit — max pieces:
        String[] limited = csv.split(",", 3);
        System.out.println(java.util.Arrays.toString(limited));
        // [Java, Spring, PostgreSQL,Redis,Docker] (third element has rest)
        
        // Split by regex:
        String[] regexSplit = "one  two   three".split("\\s+"); // split on one or more whitespace
        
        // Trailing empty strings dropped by default:
        String[] tricky = "a,,b,,".split(",");
        System.out.println(tricky.length); // 3 (trailing empties removed)
        
        String[] withEmpties = "a,,b,,".split(",", -1); // -1 keeps all
        System.out.println(withEmpties.length); // 5 (a, "", b, "", "")
        
        // ─────────────────────────────────────────
        // JOINING AND CONCATENATION
        // ─────────────────────────────────────────
        
        // String.join() — static method:
        String joined = String.join(", ", "Java", "Spring", "Boot");
        System.out.println(joined); // "Java, Spring, Boot"
        
        // Join a collection:
        java.util.List<String> techs = java.util.List.of("Docker", "K8s", "AWS");
        System.out.println(String.join(" | ", techs));
        // "Docker | K8s | AWS"
        
        // concat() — like + but method:
        String s1 = "Hello".concat(", ").concat("World!");
        System.out.println(s1); // "Hello, World!"
        
        // ─────────────────────────────────────────
        // FORMATTING
        // ─────────────────────────────────────────
        
        String formatted = String.format("%-15s %5d %8.2f", "Product", 42, 19.99);
        System.out.println(formatted); // "Product          42    19.99"
        
        String modern = "Name: %s, Score: %d".formatted("Rahim", 95);
        System.out.println(modern); // "Name: Rahim, Score: 95"
        
        // ─────────────────────────────────────────
        // JAVA 11+ METHODS
        // ─────────────────────────────────────────
        
        // repeat():
        System.out.println("─".repeat(30));
        System.out.println("ab".repeat(5)); // "ababababab"
        
        // strip(), stripLeading(), stripTrailing() (already shown above)
        
        // isBlank() (already shown above)
        
        // lines() — split into stream of lines:
        String multiLine = "Line 1\nLine 2\nLine 3";
        multiLine.lines().forEach(System.out::println);
        
        // ─────────────────────────────────────────
        // JAVA 12+ METHODS
        // ─────────────────────────────────────────
        
        // indent() — adds indentation:
        String code = "int x = 5;\nint y = 10;";
        System.out.println(code.indent(4));
        //     int x = 5;
        //     int y = 10;
        
        // transform() — apply a function:
        String result = "  hello  "
            .transform(str -> str.strip())
            .transform(str -> str.toUpperCase());
        System.out.println(result); // "HELLO"
        
        // ─────────────────────────────────────────
        // JAVA 15+ METHODS (Text Blocks and more)
        // ─────────────────────────────────────────
        
        // formatted() (already shown)
        
        // stripIndent() — removes common leading whitespace:
        String block = """
                    Hello
                    World
                    """;
        System.out.println(block.stripIndent());
        
        // translateEscapes() — process escape sequences in string:
        String withEscape = "Hello\\nWorld"; // literal \n
        System.out.println(withEscape);                     // Hello\nWorld
        System.out.println(withEscape.translateEscapes());  // Hello (newline) World
    }
}
```

---
## 3. Text Blocks (Java 15+)
```java
public class TextBlocks {
    public static void main(String[] args) {
        
        // Text blocks are multiline String literals.
        // Open with triple quote + newline.
        // Close with triple quote.
        // Incidental whitespace is automatically stripped.
        
        // ─────────────────────────────────────────
        // OLD WAY — Verbose string escaping:
        // ─────────────────────────────────────────
        
        String oldJson = "{\n" +
                         "    \"name\": \"Rahim\",\n" +
                         "    \"age\": 21,\n" +
                         "    \"city\": \"Dhaka\"\n" +
                         "}";
        System.out.println(oldJson);
        
        // ─────────────────────────────────────────
        // NEW WAY — Text block:
        // ─────────────────────────────────────────
        
        String json = """
                {
                    "name": "Rahim",
                    "age": 21,
                    "city": "Dhaka"
                }
                """;
        System.out.println(json);
        // Same output, much more readable!
        
        // ─────────────────────────────────────────
        // INCIDENTAL WHITESPACE REMOVAL
        // ─────────────────────────────────────────
        
        // The position of the closing """ determines indentation
        String block1 = """
                Hello
                World
                """; // closing """ at column 16
        // Each line has 16 spaces of "incidental" whitespace removed
        System.out.println(block1);
        // Hello
        // World
        // (no leading spaces — they were incidental)
        
        // Move """ to change behavior:
        String block2 = """
                Hello
                World
            """; // closing """ at column 12
        // Only 12 spaces removed from each line (4 remain for "Hello"/"World")
        System.out.println(block2);
        //     Hello
        //     World
        
        // ─────────────────────────────────────────
        // TEXT BLOCKS IN BACKEND DEVELOPMENT
        // ─────────────────────────────────────────
        
        // SQL queries (most common use in Spring Boot):
        String userId = "123";
        String sql = """
                SELECT u.id, u.name, u.email, COUNT(o.id) as order_count
                FROM users u
                LEFT JOIN orders o ON u.id = o.user_id
                WHERE u.id = '%s'
                  AND u.active = true
                GROUP BY u.id, u.name, u.email
                ORDER BY u.name
                """.formatted(userId);
        System.out.println("SQL Query:\n" + sql);
        // NOTE: In real Spring Boot, use @Query or parameterized queries
        // NEVER string-interpolate user input into SQL — SQL injection risk!
        
        // HTML email template:
        String customerName = "Rahim Ahmed";
        String orderNumber = "ORD-2024-001";
        double amount = 1500.0;
        
        String emailHtml = """
                <!DOCTYPE html>
                <html>
                <body>
                    <h1>Order Confirmation</h1>
                    <p>Dear %s,</p>
                    <p>Your order <strong>%s</strong> has been confirmed.</p>
                    <p>Total Amount: <strong>৳%.2f</strong></p>
                    <p>Thank you for your purchase!</p>
                </body>
                </html>
                """.formatted(customerName, orderNumber, amount);
        
        System.out.println("Email Template:");
        System.out.println(emailHtml);
        
        // JSON request body for API tests:
        String requestBody = """
                {
                    "username": "rahim",
                    "password": "securepass",
                    "rememberMe": true
                }
                """;
        
        // Log messages:
        String logMessage = """
                [ERROR] Payment processing failed
                  Order ID  : ORD-001
                  User ID   : 12345
                  Amount    : ৳5000.00
                  Reason    : Insufficient funds
                  Timestamp : 2024-01-15T14:32:11
                """;
        System.out.println(logMessage);
        
        // ─────────────────────────────────────────
        // TEXT BLOCK ESCAPE SEQUENCES
        // ─────────────────────────────────────────
        
        // \s — trailing whitespace placeholder (prevents line trimming)
        String aligned = """
                Name:   Rahim  \s
                Email:  rahim@test.com  \s
                City:   Dhaka  \s
                """;
        // The \s prevents trailing spaces from being stripped
        
        // \ at end of line — line continuation (no newline added)
        String longLine = """
                This is a very long string that I want to write \
                across multiple lines in source code but have \
                it appear as one line at runtime.
                """;
        System.out.println(longLine);
        // "This is a very long string that I want to write across multiple lines..."
    }
}
```

---
## 4. Regular Expressions with Strings
```java
public class RegularExpressions {
    public static void main(String[] args) {
        
        // Regex = patterns for matching/searching text
        // Java String methods that use regex:
        // matches(), replaceAll(), replaceFirst(), split()
        
        // ─────────────────────────────────────────
        // BASIC REGEX PATTERNS
        // ─────────────────────────────────────────
        
        // .      → any single character (except newline)
        // *      → zero or more of previous
        // +      → one or more of previous
        // ?      → zero or one of previous
        // ^      → start of string
        // $      → end of string
        // [abc]  → any character from the set
        // [^abc] → any character NOT in the set
        // [a-z]  → any character in range
        // \d     → digit [0-9] (use \\d in Java strings)
        // \D     → non-digit
        // \w     → word char [a-zA-Z0-9_]
        // \W     → non-word char
        // \s     → whitespace [ \t\n\r\f]
        // \S     → non-whitespace
        // \b     → word boundary
        // {n}    → exactly n times
        // {n,}   → n or more times
        // {n,m}  → between n and m times
        // (abc)  → capturing group
        // |      → OR (this or that)
        
        // IMPORTANT: In Java strings, \ must be escaped as \\
        // \d in regex → "\\d" in Java string
        
        // ─────────────────────────────────────────
        // matches() — full string must match pattern
        // ─────────────────────────────────────────
        
        // Digit-only check:
        System.out.println("12345".matches("\\d+"));    // true
        System.out.println("123a5".matches("\\d+"));    // false
        System.out.println("".matches("\\d+"));         // false
        System.out.println("".matches("\\d*"));         // true (* = zero or more)
        
        // Phone number (Bangladesh: 01[3-9]XXXXXXXX):
        String bdPhone = "01712345678";
        System.out.println(bdPhone.matches("01[3-9]\\d{8}")); // true
        System.out.println("01012345678".matches("01[3-9]\\d{8}")); // false (0 not in [3-9])
        
        // Email (simplified):
        String emailPattern = "^[a-zA-Z0-9._%+\\-]+@[a-zA-Z0-9.\\-]+\\.[a-zA-Z]{2,}$";
        System.out.println("rahim@example.com".matches(emailPattern));    // true
        System.out.println("invalid-email".matches(emailPattern));        // false
        System.out.println("rahim@".matches(emailPattern));               // false
        System.out.println("rahim@.com".matches(emailPattern));           // false
        System.out.println("r@e.c".matches(emailPattern));               // false (TLD < 2)
        
        // Strong password:
        String pwdPattern = "^(?=.*[A-Z])(?=.*[a-z])(?=.*\\d)(?=.*[!@#$%^&*]).{8,}$";
        // (?=...) = lookahead (must contain without consuming)
        System.out.println("WeakPass".matches(pwdPattern));          // false (no digit/special)
        System.out.println("Strong@Pass1".matches(pwdPattern));      // true
        System.out.println("short@1A".matches(pwdPattern));          // false (< 8)
        
        // UUID format:
        String uuidPattern = "[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}";
        System.out.println("550e8400-e29b-41d4-a716-446655440000".matches(uuidPattern)); // true
        
        // ─────────────────────────────────────────
        // replaceAll() with regex
        // ─────────────────────────────────────────
        
        String messy = "Hello   World    Java    2024";
        
        // Replace multiple spaces with single space:
        System.out.println(messy.replaceAll("\\s+", " "));
        // "Hello World Java 2024"
        
        // Remove all non-alphanumeric:
        String dirty = "H3ll0! W0rld#2024@";
        System.out.println(dirty.replaceAll("[^a-zA-Z0-9]", ""));
        // "H3ll0W0rld2024"
        
        // Remove digits:
        System.out.println("abc123def456".replaceAll("\\d", ""));
        // "abcdef"
        
        // Mask email (keep first char and domain):
        String email = "rahim@example.com";
        int atIdx = email.indexOf('@');
        String masked = email.charAt(0)
                      + "*".repeat(atIdx - 1)
                      + email.substring(atIdx);
        System.out.println("Masked email: " + masked); // r****@example.com
        
        // Replace with captured groups:
        String date = "2024-01-15";
        String reformatted = date.replaceAll("(\\d{4})-(\\d{2})-(\\d{2})", "$3/$2/$1");
        System.out.println("Reformatted date: " + reformatted); // 15/01/2024
        // $1 = first group (year), $2 = month, $3 = day
        
        // ─────────────────────────────────────────
        // split() with regex
        // ─────────────────────────────────────────
        
        // Split on any whitespace:
        String text = "Java   Spring Boot   PostgreSQL";
        String[] words = text.split("\\s+");
        System.out.println(java.util.Arrays.toString(words));
        // [Java, Spring, Boot, PostgreSQL]
        
        // Split on comma or semicolon:
        String data = "one,two;three,four;five";
        String[] items = data.split("[,;]");
        System.out.println(java.util.Arrays.toString(items));
        // [one, two, three, four, five]
        
        // Split on dot (. is regex wildcard — must escape!):
        // WRONG: "a.b.c".split(".") → [] (. matches everything)
        String[] octets = "192.168.1.1".split("\\.");  // escape the dot!
        System.out.println(java.util.Arrays.toString(octets));
        // [192, 168, 1, 1]
        
        // ─────────────────────────────────────────
        // Pattern and Matcher (for complex regex)
        // ─────────────────────────────────────────
        
        // java.util.regex.Pattern — precompile for reuse (faster)
        java.util.regex.Pattern emailPat =
            java.util.regex.Pattern.compile(
                "^[a-zA-Z0-9._%+\\-]+@[a-zA-Z0-9.\\-]+\\.[a-zA-Z]{2,}$");
        
        String[] testEmails = {
            "rahim@example.com",
            "invalid",
            "test@test.co.uk",
            "@nodomain.com"
        };
        
        System.out.println("\nEmail validation with Pattern:");
        for (String e : testEmails) {
            java.util.regex.Matcher matcher = emailPat.matcher(e);
            System.out.printf("%-25s → %s%n",
                              e,
                              matcher.matches() ? "✅ Valid" : "❌ Invalid");
        }
        
        // Find all matches in a string:
        String html = "<a href='page1.html'>Link 1</a> text <a href='page2.html'>Link 2</a>";
        java.util.regex.Pattern hrefPat =
            java.util.regex.Pattern.compile("href='([^']+)'");
        java.util.regex.Matcher hrefMatcher = hrefPat.matcher(html);
        
        System.out.println("\nAll hrefs found:");
        while (hrefMatcher.find()) {
            System.out.println("  Found: " + hrefMatcher.group(1)); // group(1) = first capture group
        }
        // Found: page1.html
        // Found: page2.html
        
        // ─────────────────────────────────────────
        // COMMON REGEX PATTERNS FOR BACKEND
        // ─────────────────────────────────────────
        
        System.out.println("\nCommon patterns:");
        
        // Integer:
        System.out.println("-123".matches("-?\\d+")); // true (optional minus, one+ digits)
        
        // Decimal number:
        System.out.println("3.14".matches("-?\\d+\\.\\d+")); // true
        System.out.println("3".matches("-?\\d+\\.\\d+"));    // false (no decimal)
        
        // Alphanumeric username (3-20 chars):
        System.out.println("rahim123".matches("[a-zA-Z0-9_]{3,20}")); // true
        System.out.println("ra".matches("[a-zA-Z0-9_]{3,20}"));       // false (too short)
        
        // Date (yyyy-mm-dd):
        System.out.println("2024-01-15".matches("\\d{4}-\\d{2}-\\d{2}")); // true
        System.out.println("24-1-15".matches("\\d{4}-\\d{2}-\\d{2}"));   // false
        
        // IP address (simplified):
        System.out.println("192.168.1.1".matches("\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}")); // true
        
        // Credit card (16 digits, optional spaces):
        System.out.println("4532015112830366".matches("\\d{16}")); // true
        System.out.println("4532 0151 1283 0366".matches("\\d{4}( \\d{4}){3}")); // true
    }
}
```

---
## 5. String Conversion - Every Direction
```java
public class StringConversion {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // PRIMITIVE → STRING
        // ─────────────────────────────────────────
        
        int intVal      = 42;
        double dblVal   = 3.14;
        boolean boolVal = true;
        char charVal    = 'A';
        long longVal    = 9999999999L;
        
        // Best way: String.valueOf() — works for all types, null-safe
        String s1 = String.valueOf(intVal);    // "42"
        String s2 = String.valueOf(dblVal);    // "3.14"
        String s3 = String.valueOf(boolVal);   // "true"
        String s4 = String.valueOf(charVal);   // "A"
        String s5 = String.valueOf(longVal);   // "9999999999"
        
        // Wrapper class toString():
        String s6 = Integer.toString(intVal);  // "42"
        String s7 = Double.toString(dblVal);   // "3.14"
        String s8 = Boolean.toString(boolVal); // "true"
        
        // Concatenation (works but can be confusing):
        String s9 = "" + intVal; // "42" — works but avoid
        
        // Integer in different bases:
        System.out.println(Integer.toBinaryString(255)); // "11111111"
        System.out.println(Integer.toHexString(255));    // "ff"
        System.out.println(Integer.toOctalString(255));  // "377"
        System.out.println(Integer.toString(255, 16));   // "ff" (any base)
        System.out.println(Integer.toString(255, 2));    // "11111111"
        
        // ─────────────────────────────────────────
        // STRING → PRIMITIVE
        // ─────────────────────────────────────────
        
        // String → int:
        int parsed1 = Integer.parseInt("42");       // 42
        int parsed2 = Integer.parseInt("-17");      // -17
        int parsed3 = Integer.parseInt("FF", 16);   // 255 (hex string)
        int parsed4 = Integer.parseInt("11111111", 2); // 255 (binary string)
        
        // String → long:
        long parsed5 = Long.parseLong("9999999999"); // 9999999999L
        
        // String → double:
        double parsed6 = Double.parseDouble("3.14"); // 3.14
        double parsed7 = Double.parseDouble("1e5");  // 100000.0
        
        // String → boolean:
        boolean parsed8 = Boolean.parseBoolean("true");  // true
        boolean parsed9 = Boolean.parseBoolean("True");  // true (case-insensitive)
        boolean parsed10 = Boolean.parseBoolean("yes");  // false! not "yes"
        boolean parsed11 = Boolean.parseBoolean("1");    // false! not "1"
        // Only "true" (case-insensitive) gives true. Everything else is false.
        
        // ─────────────────────────────────────────
        // HANDLING PARSE ERRORS
        // ─────────────────────────────────────────
        
        // parseInt on invalid string throws NumberFormatException:
        try {
            int bad = Integer.parseInt("hello");
        } catch (NumberFormatException e) {
            System.out.println("Cannot parse 'hello' as int: " + e.getMessage());
        }
        
        try {
            int bad2 = Integer.parseInt("99999999999"); // exceeds int range
        } catch (NumberFormatException e) {
            System.out.println("Out of int range: " + e.getMessage());
        }
        
        // SAFE parsing helper:
        System.out.println(safeParseInt("42", 0));    // 42
        System.out.println(safeParseInt("bad", 0));   // 0 (default)
        System.out.println(safeParseInt(null, -1));   // -1 (null default)
        
        // ─────────────────────────────────────────
        // STRING ↔ CHAR ARRAY
        // ─────────────────────────────────────────
        
        String str = "Hello";
        
        // String to char[]:
        char[] charArr = str.toCharArray();
        System.out.println(charArr[0]); // 'H'
        charArr[0] = 'J';               // modify the array (not the original String!)
        System.out.println(str);        // "Hello" — unchanged (immutable!)
        
        // char[] to String:
        String fromArr = new String(charArr);
        System.out.println(fromArr); // "Jello"
        
        // Useful: reverse a string
        char[] toReverse = str.toCharArray();
        int left = 0, right = toReverse.length - 1;
        while (left < right) {
            char temp = toReverse[left];
            toReverse[left++] = toReverse[right];
            toReverse[right--] = temp;
        }
        System.out.println(new String(toReverse)); // "olleH"
        
        // ─────────────────────────────────────────
        // STRING ↔ BYTE ARRAY
        // ─────────────────────────────────────────
        
        // Default encoding (UTF-8 recommended):
        byte[] bytes = "Hello".getBytes(java.nio.charset.StandardCharsets.UTF_8);
        System.out.println(bytes.length); // 5
        
        // Byte array back to String:
        String fromBytes = new String(bytes, java.nio.charset.StandardCharsets.UTF_8);
        System.out.println(fromBytes); // "Hello"
        
        // Non-ASCII characters:
        String bengali = "বাংলা";
        byte[] bengaliBytes = bengali.getBytes(java.nio.charset.StandardCharsets.UTF_8);
        System.out.println("Bengali bytes: " + bengaliBytes.length); // more than 5!
        // Each Bengali character is 3 bytes in UTF-8
        
        // ─────────────────────────────────────────
        // STRING ↔ StringBuilder/StringBuffer
        // ─────────────────────────────────────────
        
        // String to StringBuilder:
        StringBuilder sb = new StringBuilder("Hello");
        sb.append(" World");
        
        // StringBuilder to String:
        String backToString = sb.toString();
        System.out.println(backToString); // "Hello World"
        
        System.out.println("Parsed values:");
        System.out.println(parsed1 + parsed3); // 42 + 255 = 297
        System.out.println(parsed6 + parsed7); // 3.14 + 100000.0
        System.out.println(parsed8 && !parsed9); // false
    }
    
    static int safeParseInt(String str, int defaultValue) {
        if (str == null) return defaultValue;
        try {
            return Integer.parseInt(str.trim());
        } catch (NumberFormatException e) {
            return defaultValue;
        }
    }
}
```

---
## 6. String Performance - What Every Backend Engineer Must Know
```java
public class StringPerformance {
    public static void main(String[] args) {
        
        final int ITERATIONS = 100_000;
        
        // ─────────────────────────────────────────
        // BENCHMARK 1: String += vs StringBuilder
        // ─────────────────────────────────────────
        
        long start = System.currentTimeMillis();
        String concat = "";
        for (int i = 0; i < ITERATIONS; i++) {
            concat += i; // NEW String object EVERY iteration!
        }
        long concatTime = System.currentTimeMillis() - start;
        
        start = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < ITERATIONS; i++) {
            sb.append(i); // SAME object modified
        }
        String sbResult = sb.toString();
        long sbTime = System.currentTimeMillis() - start;
        
        System.out.printf("String +=     : %,d ms%n", concatTime);
        System.out.printf("StringBuilder : %,d ms%n", sbTime);
        System.out.printf("Speed ratio   : %.1fx faster%n", (double) concatTime / sbTime);
        System.out.println("Same result   : " + concat.equals(sbResult));
        
        // WHY is StringBuilder so much faster?
        // String += creates new String each time: O(n) copy each iteration = O(n²) total
        // StringBuilder has a resizable char buffer: O(1) amortized per append = O(n) total
        
        // ─────────────────────────────────────────
        // WHEN COMPILER OPTIMIZES FOR YOU
        // ─────────────────────────────────────────
        
        // These are EQUIVALENT — compiler converts to StringBuilder:
        String s1 = "Hello" + ", " + "World" + "!"; // compile-time constant folding
        // Compiler actually does: String s1 = "Hello, World!";
        
        // One-liner concatenation — compiler uses StringBuilder:
        String name = "Rahim";
        int age = 21;
        String bio = "Name: " + name + ", Age: " + age; // OK — compiler handles this
        // Equivalent to: new StringBuilder("Name: ").append(name).append(", Age: ").append(age).toString()
        
        // The PROBLEM is in LOOPS:
        String bad = "";
        for (int i = 0; i < 100; i++) {
            bad += i; // compiler does NOT optimize this to StringBuilder!
            // Each iteration: new StringBuilder(bad).append(i).toString()
            // The intermediate String "bad" grows each time!
        }
        
        // ─────────────────────────────────────────
        // BENCHMARK 2: String.format vs +
        // ─────────────────────────────────────────
        
        start = System.currentTimeMillis();
        for (int i = 0; i < ITERATIONS; i++) {
            String s = String.format("Name: %s, Age: %d", "Rahim", 21);
        }
        long formatTime = System.currentTimeMillis() - start;
        
        start = System.currentTimeMillis();
        for (int i = 0; i < ITERATIONS; i++) {
            String s = "Name: " + "Rahim" + ", Age: " + 21;
        }
        long concatTime2 = System.currentTimeMillis() - start;
        
        System.out.printf("%nString.format : %,d ms%n", formatTime);
        System.out.printf("Concatenation : %,d ms%n", concatTime2);
        // String.format is SLOWER than + for simple cases
        // Use + for simple, format() for complex/aligned output
        
        // ─────────────────────────────────────────
        // WHEN TO USE WHAT
        // ─────────────────────────────────────────
        
        System.out.println("""
                
                PERFORMANCE GUIDE:
                ─────────────────────────────────────
                Simple concat (few ops)  → String +
                In loops (many ops)      → StringBuilder
                Formatted output         → String.format() or printf
                Thread-safe building     → StringBuffer
                Multiline text           → Text blocks
                """);
        
        // ─────────────────────────────────────────
        // MEMORY CONSIDERATIONS
        // ─────────────────────────────────────────
        
        // String interning saves memory for many duplicate strings:
        // e.g., processing CSV with 10,000 rows with "Dhaka" as city
        // Without intern: 10,000 separate "Dhaka" objects
        // With intern: all 10,000 point to the SAME pooled "Dhaka"
        
        // But: intern() itself has a cost — use only when you have
        // MANY duplicates AND memory is a concern
        
        // Modern practice:
        // → Use String naturally for most code
        // → Use StringBuilder in loops
        // → Trust the JVM and JIT compiler to optimize
        // → Profile BEFORE optimizing (premature optimization is evil)
    }
}
```

---
## 7. String in Spring Boot - What You'll Actually Use
```java
public class StringInSpringBoot {
    
    // These are the patterns you'll write EVERY DAY
    // in real Spring Boot applications.
    // You don't need Spring Boot for these examples —
    // just understand the patterns.
    
    // ─────────────────────────────────────────
    // PATTERN 1: Input normalization in service layer
    // ─────────────────────────────────────────
    
    public static String normalizeEmail(String email) {
        if (email == null || email.isBlank()) {
            throw new IllegalArgumentException("Email cannot be blank");
        }
        return email.trim().toLowerCase();
    }
    
    public static String normalizeName(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        // Trim, normalize whitespace, title case
        String trimmed = name.trim().replaceAll("\\s+", " ");
        String[] words = trimmed.split(" ");
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < words.length; i++) {
            if (words[i].isEmpty()) continue;
            result.append(Character.toUpperCase(words[i].charAt(0)));
            result.append(words[i].substring(1).toLowerCase());
            if (i < words.length - 1) result.append(" ");
        }
        return result.toString();
    }
    
    // ─────────────────────────────────────────
    // PATTERN 2: Validation methods
    // ─────────────────────────────────────────
    
    public static boolean isValidEmail(String email) {
        return email != null
            && email.matches("^[a-zA-Z0-9._%+\\-]+@[a-zA-Z0-9.\\-]+\\.[a-zA-Z]{2,}$");
    }
    
    public static boolean isValidBdPhone(String phone) {
        if (phone == null) return false;
        String cleaned = phone.replaceAll("[\\s\\-()]", ""); // remove spaces, dashes, parens
        return cleaned.matches("(\\+88)?01[3-9]\\d{8}");
    }
    
    public static boolean isStrongPassword(String password) {
        if (password == null || password.length() < 8) return false;
        return password.matches("^(?=.*[A-Z])(?=.*[a-z])(?=.*\\d)(?=.*[!@#$%^&*]).+$");
    }
    
    // ─────────────────────────────────────────
    // PATTERN 3: Generating tokens and IDs
    // ─────────────────────────────────────────
    
    public static String generateOrderId() {
        // Format: ORD-YYYYMMDD-XXXX (where XXXX is random)
        java.time.LocalDate today = java.time.LocalDate.now();
        int random = (int)(Math.random() * 9000) + 1000;
        return String.format("ORD-%d%02d%02d-%04d",
                             today.getYear(),
                             today.getMonthValue(),
                             today.getDayOfMonth(),
                             random);
    }
    
    public static String maskSensitiveData(String data, int visibleChars) {
        if (data == null || data.length() <= visibleChars) return data;
        return "*".repeat(data.length() - visibleChars)
               + data.substring(data.length() - visibleChars);
    }
    
    // ─────────────────────────────────────────
    // PATTERN 4: Building dynamic messages
    // ─────────────────────────────────────────
    
    public static String buildErrorMessage(String field,
                                            String reason,
                                            Object value) {
        return "Validation failed for field '%s': %s (received: '%s')"
               .formatted(field, reason, value);
    }
    
    public static String buildWelcomeMessage(String name,
                                              String planType,
                                              int trialDays) {
        return """
               Welcome to the platform, %s!
               
               Your %s plan is now active.
               You have %d days of free trial.
               
               Get started at: https://app.example.com
               """.formatted(name, planType, trialDays);
    }
    
    // ─────────────────────────────────────────
    // PATTERN 5: Parsing structured data
    // ─────────────────────────────────────────
    
    public static java.util.Map<String, String> parseQueryString(String query) {
        java.util.Map<String, String> params = new java.util.LinkedHashMap<>();
        if (query == null || query.isBlank()) return params;
        
        for (String pair : query.split("&")) {
            String[] kv = pair.split("=", 2); // limit 2 — value might contain =
            if (kv.length == 2) {
                params.put(kv[0].trim(), kv[1].trim());
            }
        }
        return params;
    }
    
    public static String buildQueryString(java.util.Map<String, String> params) {
        StringBuilder sb = new StringBuilder();
        boolean first = true;
        for (java.util.Map.Entry<String, String> entry : params.entrySet()) {
            if (!first) sb.append("&");
            sb.append(entry.getKey()).append("=").append(entry.getValue());
            first = false;
        }
        return sb.toString();
    }
    
    // ─────────────────────────────────────────
    // PATTERN 6: Log formatting
    // ─────────────────────────────────────────
    
    public static String formatLogEntry(String level,
                                         String requestId,
                                         String message,
                                         long durationMs) {
        return "[%s] [%s] %s (took %dms)".formatted(
               level, requestId, message, durationMs);
    }
    
    public static void main(String[] args) {
        
        System.out.println("=== Normalization ===");
        System.out.println(normalizeEmail("  RAHIM@EXAMPLE.COM  "));
        System.out.println(normalizeName("  md rahim ahmed  "));
        
        System.out.println("\n=== Validation ===");
        String[] emails = {"rahim@test.com", "invalid", "a@b.co"};
        String[] phones = {"01712345678", "+8801812345678", "017-1234-5678", "123"};
        
        for (String e : emails) {
            System.out.printf("%-25s → %s%n", e, isValidEmail(e) ? "✅" : "❌");
        }
        for (String p : phones) {
            System.out.printf("%-20s → %s%n", p, isValidBdPhone(p) ? "✅" : "❌");
        }
        
        System.out.println("\n=== ID Generation ===");
        for (int i = 0; i < 3; i++) {
            System.out.println(generateOrderId());
        }
        
        System.out.println("\n=== Masking ===");
        System.out.println(maskSensitiveData("4532015112830366", 4)); // card
        System.out.println(maskSensitiveData("secretpassword", 0));   // password
        System.out.println(maskSensitiveData("rahim@example.com", 4)); // partial email
        
        System.out.println("\n=== Messages ===");
        System.out.println(buildErrorMessage("email", "must be valid format", "notanemail"));
        System.out.println(buildWelcomeMessage("Rahim", "Professional", 14));
        
        System.out.println("\n=== Query String ===");
        String qs = "page=2&size=10&sort=name&order=asc";
        java.util.Map<String, String> params = parseQueryString(qs);
        params.forEach((k, v) -> System.out.println("  " + k + " = " + v));
        System.out.println("Rebuilt: " + buildQueryString(params));
        
        System.out.println("\n=== Logging ===");
        System.out.println(formatLogEntry("INFO", "req-abc123", "User login successful", 45));
        System.out.println(formatLogEntry("ERROR", "req-xyz789", "Payment processing failed", 1203));
    }
}
```

---
## Build This - String Processing Engine
```java
// File: StringEngine.java
// A comprehensive string processing utility
// demonstrating all String concepts in a real context

import java.util.*;
import java.util.regex.*;

public class StringEngine {
    
    // ─────────────────────────────────────────
    // SECTION 1: USER PROFILE PROCESSOR
    // ─────────────────────────────────────────
    
    static Map<String, String> processUserProfile(String rawInput) {
        Map<String, String> profile = new LinkedHashMap<>();
        
        // rawInput format: "name|email|phone|city|password"
        String[] parts = rawInput.split("\\|");
        if (parts.length < 5) {
            profile.put("error", "Incomplete data: expected 5 fields, got " + parts.length);
            return profile;
        }
        
        String name     = parts[0].trim();
        String email    = parts[1].trim().toLowerCase();
        String phone    = parts[2].trim().replaceAll("[\\s\\-()]", "");
        String city     = parts[3].trim();
        String password = parts[4].trim();
        
        // Validate and process
        List<String> errors = new ArrayList<>();
        
        if (name.isBlank() || name.length() < 2) {
            errors.add("name: must be at least 2 characters");
        }
        if (!email.matches("^[a-zA-Z0-9._%+\\-]+@[a-zA-Z0-9.\\-]+\\.[a-zA-Z]{2,}$")) {
            errors.add("email: invalid format");
        }
        if (!phone.matches("(\\+88)?01[3-9]\\d{8}")) {
            errors.add("phone: invalid Bangladesh phone number");
        }
        if (password.length() < 8 ||
            !password.matches(".*[A-Z].*") ||
            !password.matches(".*\\d.*")) {
            errors.add("password: must be 8+ chars with uppercase and digit");
        }
        
        if (!errors.isEmpty()) {
            profile.put("status", "INVALID");
            profile.put("errors", String.join("; ", errors));
            return profile;
        }
        
        // Build clean profile
        profile.put("status", "VALID");
        profile.put("name", toTitleCase(name));
        profile.put("email", email);
        profile.put("phone", formatPhone(phone));
        profile.put("city", toTitleCase(city));
        profile.put("passwordStrength", assessPasswordStrength(password));
        profile.put("maskedPassword", "*".repeat(password.length()));
        profile.put("username", generateUsername(name, email));
        
        return profile;
    }
    
    // ─────────────────────────────────────────
    // SECTION 2: TEXT ANALYSIS
    // ─────────────────────────────────────────
    
    static Map<String, Object> analyzeText(String text) {
        Map<String, Object> analysis = new LinkedHashMap<>();
        
        if (text == null || text.isBlank()) {
            analysis.put("error", "Empty text");
            return analysis;
        }
        
        // Word analysis
        String[] words = text.trim().split("\\s+");
        int wordCount = words.length;
        int charCount = text.length();
        int charNoSpaces = text.replace(" ", "").length();
        int sentenceCount = text.split("[.!?]+").length;
        
        // Word frequency
        Map<String, Integer> frequency = new LinkedHashMap<>();
        for (String word : words) {
            String clean = word.replaceAll("[^a-zA-Z]", "").toLowerCase();
            if (!clean.isEmpty()) {
                frequency.merge(clean, 1, Integer::sum);
            }
        }
        
        // Most frequent word
        String mostFrequent = frequency.entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElse("none");
        
        // Average word length
        double avgWordLen = 0;
        for (String w : words) {
            avgWordLen += w.replaceAll("[^a-zA-Z]", "").length();
        }
        avgWordLen /= wordCount;
        
        // Longest word
        String longest = "";
        for (String w : words) {
            String clean = w.replaceAll("[^a-zA-Z]", "");
            if (clean.length() > longest.length()) longest = clean;
        }
        
        // Vowel/consonant count
        int vowels = 0, consonants = 0;
        String vowelStr = "aeiouAEIOU";
        for (char c : text.toCharArray()) {
            if (Character.isLetter(c)) {
                if (vowelStr.indexOf(c) >= 0) vowels++;
                else consonants++;
            }
        }
        
        // Is palindrome (ignore spaces, case)
        String cleaned = text.replaceAll("[^a-zA-Z]", "").toLowerCase();
        String reversed = new StringBuilder(cleaned).reverse().toString();
        boolean isPalindrome = cleaned.equals(reversed);
        
        // Populate results
        analysis.put("wordCount", wordCount);
        analysis.put("charCount", charCount);
        analysis.put("charNoSpaces", charNoSpaces);
        analysis.put("sentenceCount", sentenceCount);
        analysis.put("uniqueWords", frequency.size());
        analysis.put("mostFrequent", mostFrequent + " (" + frequency.get(mostFrequent) + "x)");
        analysis.put("avgWordLength", String.format("%.1f", avgWordLen));
        analysis.put("longestWord", longest);
        analysis.put("vowels", vowels);
        analysis.put("consonants", consonants);
        analysis.put("isPalindrome", isPalindrome);
        
        return analysis;
    }
    
    // ─────────────────────────────────────────
    // SECTION 3: LOG PARSER
    // ─────────────────────────────────────────
    
    static void parseLogs(String[] logLines) {
        Pattern logPattern = Pattern.compile(
            "\\[(\\d{4}-\\d{2}-\\d{2})\\] \\[(\\w+)\\] \\[([^\\]]+)\\] (.+)"
        );
        
        int info = 0, warn = 0, error = 0, debug = 0;
        List<String> errors = new ArrayList<>();
        
        System.out.println("╔═══════════════════════════════════════════════╗");
        System.out.println("║              LOG ANALYSIS                     ║");
        System.out.println("╠═══════════════════════════════════════════════╣");
        
        for (String line : logLines) {
            Matcher m = logPattern.matcher(line);
            if (m.matches()) {
                String date    = m.group(1);
                String level   = m.group(2);
                String reqId   = m.group(3);
                String message = m.group(4);
                
                switch (level) {
                    case "INFO"  -> info++;
                    case "WARN"  -> warn++;
                    case "ERROR" -> { error++; errors.add(reqId + ": " + message); }
                    case "DEBUG" -> debug++;
                }
                
                String icon = switch (level) {
                    case "INFO"  -> "ℹ️ ";
                    case "WARN"  -> "⚠️ ";
                    case "ERROR" -> "❌";
                    case "DEBUG" -> "🔍";
                    default      -> "❓";
                };
                
                System.out.printf("║ %s [%s] %-10s %s%n",
                                  icon, date, "[" + level + "]",
                                  message.length() > 30
                                      ? message.substring(0, 27) + "..."
                                      : message);
            } else {
                System.out.println("║ ⚠️  UNPARSEABLE: " + line);
            }
        }
        
        System.out.println("╠═══════════════════════════════════════════════╣");
        System.out.printf( "║  INFO: %-3d  WARN: %-3d  ERROR: %-3d  DEBUG: %-3d ║%n",
                           info, warn, error, debug);
        
        if (!errors.isEmpty()) {
            System.out.println("╠═══════════════════════════════════════════════╣");
            System.out.println("║ ERRORS REQUIRING ATTENTION:                   ║");
            for (String err : errors) {
                System.out.printf("║  • %-43s║%n",
                                  err.length() > 43 ? err.substring(0, 40) + "..." : err);
            }
        }
        System.out.println("╚═══════════════════════════════════════════════╝");
    }
    
    // ─────────────────────────────────────────
    // HELPER METHODS
    // ─────────────────────────────────────────
    
    static String toTitleCase(String s) {
        if (s == null || s.isBlank()) return "";
        String[] words = s.trim().toLowerCase().split("\\s+");
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < words.length; i++) {
            if (words[i].isEmpty()) continue;
            sb.append(Character.toUpperCase(words[i].charAt(0)));
            sb.append(words[i].substring(1));
            if (i < words.length - 1) sb.append(" ");
        }
        return sb.toString();
    }
    
    static String formatPhone(String phone) {
        // Normalize to +880XXXXXXXXXX format
        String digits = phone.replaceAll("[^\\d]", "");
        if (digits.startsWith("88")) digits = digits.substring(2);
        if (digits.startsWith("0")) digits = digits.substring(1);
        return "+880" + digits;
    }
    
    static String generateUsername(String name, String email) {
        String namePart = name.toLowerCase().replaceAll("[^a-z]", "").substring(0, Math.min(6, name.length()));
        int emailHash = Math.abs(email.hashCode() % 1000);
        return namePart + emailHash;
    }
    
    static String assessPasswordStrength(String password) {
        int score = 0;
        if (password.length() >= 8)  score++;
        if (password.length() >= 12) score++;
        if (password.matches(".*[A-Z].*")) score++;
        if (password.matches(".*[a-z].*")) score++;
        if (password.matches(".*\\d.*")) score++;
        if (password.matches(".*[!@#$%^&*].*")) score++;
        
        return switch (score) {
            case 0, 1, 2 -> "WEAK";
            case 3, 4    -> "MODERATE";
            case 5       -> "STRONG";
            case 6       -> "VERY STRONG";
            default      -> "UNKNOWN";
        };
    }
    
    // ─────────────────────────────────────────
    // MAIN
    // ─────────────────────────────────────────
    
    public static void main(String[] args) {
        
        // ── USER PROFILES ──
        System.out.println("╔══════════════════════════════════════════════╗");
        System.out.println("║          USER PROFILE PROCESSING             ║");
        System.out.println("╚══════════════════════════════════════════════╝");
        
        String[] profiles = {
            "rahim ahmed|rahim@example.com|01712345678|dhaka|SecurePass1",
            "  KARIM  |karim@test.org|+8801812345678|chittagong|Strong@Pass2",
            "ab|invalid-email|12345|sylhet|weak", // multiple validation errors
            "Hasan Ali||01612345678|khulna|GoodPass1",  // missing email
        };
        
        for (String raw : profiles) {
            System.out.println("\nInput: " + raw);
            Map<String, String> result = processUserProfile(raw);
            result.forEach((k, v) ->
                System.out.printf("  %-20s: %s%n", k, v));
        }
        
        // ── TEXT ANALYSIS ──
        System.out.println("\n╔══════════════════════════════════════════════╗");
        System.out.println("║              TEXT ANALYSIS                   ║");
        System.out.println("╚══════════════════════════════════════════════╝");
        
        String text = "Java is a powerful language. Java enables backend development. "
                    + "Backend engineers use Java with Spring Boot. "
                    + "Spring Boot makes Java development easier.";
        
        System.out.println("Text: \"" + text.substring(0, 50) + "...\"");
        analyzeText(text).forEach((k, v) ->
            System.out.printf("  %-18s: %s%n", k, v));
        
        // ── LOG PARSING ──
        System.out.println();
        String[] logs = {
            "[2024-01-15] [INFO] [req-001] User login successful",
            "[2024-01-15] [ERROR] [req-002] Payment processing failed: timeout",
            "[2024-01-15] [WARN] [req-003] Rate limit approaching: 90% used",
            "[2024-01-15] [DEBUG] [req-004] Cache miss for key: user:123",
            "[2024-01-15] [INFO] [req-005] Order created: ORD-2024-001",
            "[2024-01-15] [ERROR] [req-006] Database connection failed",
            "MALFORMED LOG LINE WITHOUT PROPER FORMAT",
            "[2024-01-15] [INFO] [req-008] Email sent successfully",
        };
        
        parseLogs(logs);
    }
}
```

---
## Exercises
```text
EXERCISE 1: String Pool Detective
  Create StringPoolDetective.java
  Demonstrate and explain:
  a) When == returns true for Strings
  b) When == returns false for same-content Strings
  c) Effect of .intern() on == comparison
  d) Compile-time constant folding (+)
  e) Runtime concatenation (variable +)
  For each case: print the result AND a comment explaining WHY.
  This builds deep understanding before interviews.

EXERCISE 2: Password Strength Analyzer
  Create PasswordAnalyzer.java
  Given a list of 10 passwords (mix of weak and strong):
  For each password, determine:
  - Length check (8+)
  - Has uppercase
  - Has lowercase
  - Has digit
  - Has special character (!@#$%^&*)
  - No common patterns (123, abc, password, qwerty)
  - Score out of 6
  - Strength: VERY WEAK / WEAK / FAIR / STRONG / VERY STRONG
  - Specific feedback on what's missing
  Print a detailed report for each.

EXERCISE 3: Template Engine
  Create TemplateEngine.java
  Build a simple template engine that:
  - Takes a template String with {placeholders}
  - Takes a Map<String, String> of values
  - Replaces all {key} with map.get(key)
  - Leaves {unknown} if key not in map
  - Handles null values gracefully
  - Counts replacements made
  Test with:
  - Email confirmation template
  - SMS template
  - Log message template

EXERCISE 4: CSV Parser
  Create CSVParser.java
  Build a robust CSV parser that handles:
  - Basic comma-separated values
  - Quoted fields ("field, with comma")
  - Quoted fields with escaped quotes ("field ""with"" quotes")
  - Leading/trailing whitespace in fields
  - Empty fields
  - Missing fields
  Input: 10 lines of CSV data (hardcoded)
  Output: parsed as String[][] and pretty-printed as table

EXERCISE 5: Log Analyzer
  Create LogAnalyzer.java
  Given an array of 20 log lines in this format:
  "2024-01-15 14:32:11.432 INFO  [http-nio-8080-exec-1] UserService - User 123 logged in"
  "2024-01-15 14:32:15.891 ERROR [http-nio-8080-exec-2] PaymentService - Payment failed"

  Extract: date, time, level, thread, class, message
  Then produce:
  - Count by log level
  - Count by class name
  - All ERROR messages with their timestamps
  - Timeline (first and last log time)
  - Average messages per minute
  Push to GitHub: "feat: string processing engine and log analyzer"
```

---
## Common Mistakes
```text
MISTAKE 1: Using == to compare Strings
  "hello" == "hello" → sometimes true (pool), sometimes false
  ALWAYS use .equals() for content comparison.
  null-safe: "expected".equals(variable)

MISTAKE 2: Ignoring return value of String methods
  String s = "hello";
  s.toUpperCase(); // result discarded! s is unchanged!
  Fix: s = s.toUpperCase();

MISTAKE 3: NullPointerException from null String
  String s = null;
  s.length()     // NPE!
  s.toUpperCase() // NPE!
  Fix: null check, or "literal".equals(s) pattern

MISTAKE 4: String += in a loop
  String result = "";
  for (...) result += something; // O(n²) → SLOW
  Fix: StringBuilder sb = new StringBuilder();
       for (...) sb.append(something);
       String result = sb.toString();

MISTAKE 5: Substring bounds confusion
  "Hello".substring(1, 3) → "el" (not "ell")
  End index is EXCLUSIVE. Remember: [start, end)

MISTAKE 6: Split with regex special characters unescaped
  "192.168.1.1".split(".")  → [] (empty! . matches everything)
  Fix: "192.168.1.1".split("\\.") → [192, 168, 1, 1]
  
  Other special chars needing escape in regex:
  . * + ? ^ $ { } [ ] | ( ) \
  Escape as: \\. \\* etc.

MISTAKE 7: Boolean.parseBoolean() surprise
  Boolean.parseBoolean("yes") → false
  Boolean.parseBoolean("1")   → false
  Boolean.parseBoolean("TRUE") → true
  Only "true" (any case) → true. Everything else → false.

MISTAKE 8: String.format() vs printf confusion
  System.out.printf("format", args) → prints to stdout
  String.format("format", args)     → returns String
  "format".formatted(args)          → returns String (Java 15+)

MISTAKE 9: Mutating String in method doesn't affect caller
  static void method(String s) {
      s = "changed"; // local reference only — caller unaffected
  }
  String s = "original";
  method(s);
  System.out.println(s); // still "original"

MISTAKE 10: Text block indentation confusion
  String s = """
      Hello
      World
      """;
  // "Hello\nWorld\n" — not "    Hello\n    World\n"
  // Closing """ determines indent removal level
  // Place closing """ to control indentation
```

---
## Interview Questions

**Q: Explain String immutability. Why does it matter?**
A: Once a String object is created, its character sequence cannot be changed. Any operation that seems to modify it (toUpperCase, replace, concat) actually creates and returns a NEW String object. The original is unchanged. This matters for: thread safety (multiple threads can safely read the same String without synchronization), security (passwords and paths can't be altered after validation), the String Pool optimization (immutable objects can be safely shared — if one changed, all references would be affected), and hashCode caching (HashMap keys work reliably because the hash never changes).

**Q: What is the String Pool? When does == work for Strings?**
A: The String Pool is a special area in the heap where Java stores unique String literals. When you write String s = "hello", Java checks if "hello" is in the pool — if yes, returns the existing reference. If no, creates it and adds it. == works for Strings when: both variables reference the same pooled object (both created with literals of the same value, or compile-time constant expressions). == fails when either String was created with new String() or resulted from runtime operations (variable concatenation). Rule: ALWAYS use .equals() for String content comparison — == is unreliable for Strings.

**Q: What is the difference between String, StringBuilder, and StringBuffer?**
A: String is immutable — operations create new objects. Suitable for fixed text, short concatenations, and situations where immutability provides safety benefits. StringBuilder is mutable — appends modify the same underlying buffer. Not thread-safe but fastest. Use for building Strings in loops or with many operations in a single thread. StringBuffer is mutable AND thread-safe (synchronized methods). Slower than StringBuilder due to synchronization overhead. Use ONLY when multiple threads need to share and modify the same builder — which is rare. In practice: String for simple use, StringBuilder for building, StringBuffer almost never.

**Q: How does String concatenation with + work internally?**
A: For simple one-liner concatenation, the Java compiler converts + to StringBuilder operations: "a" + b + "c" becomes new StringBuilder("a").append(b).append("c").toString(). This optimization ONLY applies to one-liner expressions. In a loop, the compiler does NOT optimize — each += creates a new StringBuilder, appends the old String, appends the new value, and calls toString(), creating many intermediate objects. This is why String += in a loop is O(n²) and you must use StringBuilder explicitly in loops.

**Q: What does String.intern() do?**
A: intern() adds the String to the String Pool if not already present, then returns the pool reference. After interning, you can use == for comparison. Use cases: when you have thousands of duplicate Strings (e.g., city names in a huge dataset) and want to save memory by sharing references. However, intern() has overhead (pool lookup/insertion) and the pool has limited size in older JVMs. In modern applications, it's rarely needed — the JVM and JIT handle most optimizations. Use it only after profiling shows String duplication is consuming significant memory.

**Q: How would you validate and sanitize user input Strings in a backend API?**
A: 1) Null check first — throw IllegalArgumentException or return 400 if required field is null. 2) Blank check — trim() then isBlank() for text fields. 3) Normalize — trim(), toLowerCase() for emails, normalize whitespace with replaceAll("\\s+", " "). 4) Validate format — use matches() with regex for emails, phones, passwords, UUIDs. 5) Length bounds — check min/max length. 6) Reject dangerous patterns — check for SQL special chars if building dynamic queries (but better: use PreparedStatement). In Spring Boot, use Bean Validation annotations (@NotBlank, @Email, @Size, @Pattern) on DTO fields — Spring calls the validator automatically before your service code runs.

---

## Key Takeaways
```text
1. String is IMMUTABLE. Always capture return values.
   s.toUpperCase() does nothing. s = s.toUpperCase() works.

2. String POOL: literals are reused. new String() bypasses pool.
   ALWAYS use .equals() for content comparison. NEVER ==.
   Safe null pattern: "literal".equals(variable)

3. String is thread-safe (immutable).
   StringBuilder is NOT thread-safe (mutable, single thread).
   StringBuffer IS thread-safe (mutable, multi-thread, slow).

4. String += in loops = O(n²) = SLOW.
   StringBuilder in loops = O(n) = FAST.
   Compiler optimizes simple one-liner + but NOT loops.

5. Essential String methods (know cold):
   length(), charAt(), substring(start, end)
   indexOf(), contains(), startsWith(), endsWith()
   equals(), equalsIgnoreCase(), compareTo()
   toUpperCase(), toLowerCase()
   trim()/strip(), isEmpty(), isBlank()
   replace(), replaceAll(), split()
   String.join(), String.valueOf(), String.format()

6. replaceAll() and split() use REGEX.
   Escape special regex chars: \\. \\* \\+ etc.
   matches() must match the ENTIRE string.

7. Text blocks (Java 15+) for multiline strings.
   SQL queries, HTML, JSON templates — use text blocks.
   Closing """ position controls indentation removal.

8. String → int: Integer.parseInt() → throws NumberFormatException
   int → String: String.valueOf() or Integer.toString()
   Always handle NumberFormatException with try-catch.

9. toCharArray() and new String(chars[]) for char-level work.
   getBytes() and new String(bytes, charset) for binary/encoding.
   Always specify charset explicitly (StandardCharsets.UTF_8).

10. In real backend engineering:
    normalize (trim + lowercase) before validating
    validate with regex or dedicated validator
    never concatenate user input into SQL (injection!)
    use StringBuilder for building dynamic queries/responses
```

---