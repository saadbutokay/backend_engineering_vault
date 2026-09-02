## Overview

Regular expressions (regex) are a **pattern-matching language** for searching, extracting, and manipulating text. They allow you to describe complex string patterns concisely—validating email addresses, parsing log lines, extracting fields from unstructured data, and sanitizing inputs.

In Java backend engineering, regex appears constantly: **input validation** (is this a valid phone number?), **log parsing** (extract the timestamp and error code from this log line), **data extraction** (pull the account number from this free-text field), **string transformation** (mask credit card numbers), and **routing** (match URL patterns).

Java's regex engine lives in the `java.util.regex` package and is built around two core classes: **`Pattern`** (the compiled regex) and **`Matcher`** (the engine that applies the pattern to a specific input string). Java also exposes regex through convenience methods on the `String` class (`matches`, `replaceAll`, `replaceFirst`, `split`), which internally create `Pattern` and `Matcher` objects.

**The performance rule:** `String.matches()` recompiles the regex on every call. In hot code paths, always **precompile** your patterns as `static final` constants. This single practice can eliminate a surprising amount of CPU overhead in high-throughput backend services.

---

## Core Concepts

### The Two Core Classes

```
Pattern:
  → The compiled representation of a regular expression
  → Immutable and thread-safe (safe to share as a static constant)
  → Created via Pattern.compile(String regex)
  → Expensive to create — compile once, reuse many times

Matcher:
  → The engine that performs match operations on a character sequence
  → Created from a Pattern via pattern.matcher(input)
  → NOT thread-safe — each thread needs its own Matcher instance
  → Stateful: maintains position and match state during iteration
```

The workflow is always:

```
1. Compile the pattern:   Pattern p = Pattern.compile("regex");
2. Create a matcher:      Matcher m = p.matcher("input string");
3. Perform operations:    m.matches(), m.find(), m.group(), etc.
```

### Pattern Syntax — Character Classes

```
.       → any character (except line terminators, unless DOTALL flag is set)
\d      → digit [0-9]
\D      → non-digit [^0-9]
\w      → word character [a-zA-Z0-9_]
\W      → non-word character [^\w]
\s      → whitespace [ \t\n\x0B\f\r]
\S      → non-whitespace [^\s]
\p{L}   → any Unicode letter
\p{N}   → any Unicode number
\p{P}   → any Unicode punctuation
[abc]   → character class: a, b, or c
[^abc]  → negated class: anything except a, b, or c
[a-z]   → range: lowercase letters
[A-Z]   → range: uppercase letters
[a-zA-Z] → range: all letters
[a-zA-Z0-9] → alphanumeric
```

### Quantifiers

Quantifiers specify **how many times** the preceding element can occur:

```
Greedy (default — matches as much as possible):
  ?       → 0 or 1 times (optional)
  *       → 0 or more times
  +       → 1 or more times
  {n}     → exactly n times
  {n,}    → n or more times
  {n,m}   → between n and m times (inclusive)

Reluctant (lazy — matches as little as possible):
  ??      → 0 or 1, prefer 0
  *?      → 0 or more, prefer fewer
  +?      → 1 or more, prefer fewer
  {n,m}?  → between n and m, prefer fewer

Possessive (greedy and never backtracks):
  ?+      → 0 or 1, no backtrack
  *+      → 0 or more, no backtrack
  ++      → 1 or more, no backtrack
  {n,m}+  → between n and m, no backtrack
```

**Greedy vs Reluctant — the critical difference:**

```
Input:  "<b>bold</b> and <i>italic</i>"

Greedy:   <.*>    → matches "<b>bold</b> and <i>italic</i>" (entire string!)
Reluctant: <.*?>   → matches "<b>" then "</b>" then "<i>" then "</i>" (individual tags)
```

Greedy quantifiers consume as much as possible, then backtrack if the rest of the pattern fails. Reluctant quantifiers consume as little as possible, then expand if needed. Possessive quantifiers consume as much as possible and **never give back**—they are faster but can cause matches to fail that greedy would succeed on.

### Anchors and Boundaries

```
^       → start of line (or start of string in default mode)
$       → end of line (or end of string in default mode)
\b      → word boundary (between \w and \W)
\B      → non-word boundary
\A      → start of string (always, regardless of MULTILINE)
\Z      → end of string (before final terminator, if any)
\z      → absolute end of string
```

### Groups and Capturing

```
(abc)       → capturing group: matches "abc" and captures it for later reference
(?:abc)     → non-capturing group: matches "abc" but does not capture
(?<name>abc) → named capturing group: captures as "name"
\1, \2      → backreference to group 1, group 2 (in the pattern)
$1, $2      → backreference to group 1, group 2 (in replacement strings)
```

Groups are numbered **left to right by opening parenthesis**:

```
Pattern: ((a)(b(c)))d
Groups:  12 3 4    5
         │  │ │
         │  │ └─ group 4: "c"
         │  └─── group 3: "bc"
         └────── group 2: "a"
Group 1: "abc"
Group 0: "abcd" (the entire match, always)
```

### Lookahead and Lookbehind (Zero-Width Assertions)

These assert that a pattern exists (or does not exist) at the current position **without consuming characters**:

```
(?=abc)   → positive lookahead:  "abc" must follow (not consumed)
(?!abc)   → negative lookahead:  "abc" must NOT follow
(?<=abc)  → positive lookbehind: "abc" must precede (not consumed)
(?<!abc)  → negative lookbehind: "abc" must NOT precede
```

### Pattern Flags (Compilation Options)

```
Pattern.CASE_INSENSITIVE  (?i)  → case-insensitive matching
Pattern.MULTILINE         (?m)  → ^ and $ match line boundaries, not just string boundaries
Pattern.DOTALL            (?s)  → . matches line terminators too
Pattern.UNICODE_CASE      (?u)  → Unicode-aware case folding
Pattern.COMMENTS          (?x)  → ignore whitespace and allow # comments in pattern
Pattern.LITERAL                 → treat the entire pattern as a literal string
Pattern.UNICODE_CHARACTER_CLASS (?U) → Unicode-aware character classes
```

Flags can be embedded in the pattern itself: `(?i)hello` matches "Hello", "HELLO", "hello".

### Java-Specific Escaping

Java strings use `\` as an escape character. Regex also uses `\` as an escape character. This means **every regex backslash must be doubled** in a Java string literal:

```
Regex:    \d+          (one or more digits)
Java:     "\\d+"       (the string literal that produces \d+)

Regex:    \w+\.\w+     (word.word)
Java:     "\\w+\\.\\w+"

Regex:    \\           (literal backslash)
Java:     "\\\\"       (four backslashes!)
```

This is the single most common source of confusion for Java regex beginners. **Text blocks** (Java 13+) do not help here—they still process `\` escapes.

---

## Code Examples

### Basic Pattern and Matcher Usage

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class RegexBasics {

    public static void main(String[] args) {

        // 1. Compile the pattern (do this ONCE)
        Pattern pattern = Pattern.compile("\\d{3}-\\d{3}-\\d{4}");

        // 2. Create a matcher for a specific input
        Matcher matcher = pattern.matcher("Call me at 555-123-4567 or 555-987-6543");

        // 3. matches() — does the ENTIRE string match the pattern?
        System.out.println(matcher.matches());  // false (string has extra text)

        // 4. find() — does the pattern appear ANYWHERE in the string?
        //    Call find() repeatedly to iterate through all matches
        while (matcher.find()) {
            System.out.println("Found: " + matcher.group());
            // Found: 555-123-4567
            // Found: 555-987-6543
        }
    }
}
```

### matches() vs find() vs lookingAt()

```java
Pattern pattern = Pattern.compile("\\d+");

// matches(): entire input must match
Matcher m1 = pattern.matcher("12345");
m1.matches();   // true

Matcher m2 = pattern.matcher("abc12345def");
m2.matches();   // false (entire string is not digits)

// find(): pattern appears anywhere in the input
Matcher m3 = pattern.matcher("abc12345def");
m3.find();      // true (finds "12345" inside the string)

// lookingAt(): pattern matches at the BEGINNING of the input
Matcher m4 = pattern.matcher("12345abc");
m4.lookingAt(); // true (starts with digits)

Matcher m5 = pattern.matcher("abc12345");
m5.lookingAt(); // false (does not start with digits)
```

### Capturing Groups and Extraction

```java
// Extract name and domain from email addresses
Pattern emailPattern = Pattern.compile("([a-zA-Z0-9._%+-]+)@([a-zA-Z0-9.-]+\\.[a-zA-Z]{2,})");
Matcher matcher = emailPattern.matcher("Contact: alice@example.com or bob.smith@company.org");

while (matcher.find()) {
    String fullMatch = matcher.group(0);   // "alice@example.com"
    String username = matcher.group(1);    // "alice"
    String domain = matcher.group(2);      // "example.com"

    System.out.println("Full: " + fullMatch);
    System.out.println("User: " + username);
    System.out.println("Domain: " + domain);
    System.out.println("Start: " + matcher.start() + " End: " + matcher.end());
    System.out.println("---");
}
// Full: alice@example.com
// User: alice
// Domain: example.com
// Start: 9 End: 26
// ---
// Full: bob.smith@company.org
// User: bob.smith
// Domain: company.org
// Start: 30 End: 49
```

### Named Groups

Named groups make extraction self-documenting and immune to renumbering when the pattern changes:

```java
// Parse a log line: "2025-01-15 14:30:45 ERROR PaymentService - Transaction failed"
Pattern logPattern = Pattern.compile(
    "(?<date>\\d{4}-\\d{2}-\\d{2})\\s+" +
    "(?<time>\\d{2}:\\d{2}:\\d{2})\\s+" +
    "(?<level>\\w+)\\s+" +
    "(?<logger>\\S+)\\s+-\\s+" +
    "(?<message>.+)"
);

String logLine = "2025-01-15 14:30:45 ERROR PaymentService - Transaction failed for user 42";
Matcher matcher = logPattern.matcher(logLine);

if (matcher.matches()) {
    System.out.println("Date:    " + matcher.group("date"));    // 2025-01-15
    System.out.println("Time:    " + matcher.group("time"));    // 14:30:45
    System.out.println("Level:   " + matcher.group("level"));   // ERROR
    System.out.println("Logger:  " + matcher.group("logger"));  // PaymentService
    System.out.println("Message: " + matcher.group("message")); // Transaction failed for user 42
}
```

### Non-Capturing Groups

Use `(?:...)` when you need grouping for quantifiers or alternation but don't need to capture the result:

```java
// Match "http://" or "https://" but don't capture the protocol
Pattern urlPattern = Pattern.compile("(?:https?://)([\\w.-]+)(/.*)?");
Matcher matcher = urlPattern.matcher("https://api.example.com/v1/users");

if (matcher.matches()) {
    // group(1) is the host, not the protocol
    System.out.println("Host: " + matcher.group(1));  // api.example.com
    System.out.println("Path: " + matcher.group(2));  // /v1/users
}
```

### Lookahead and Lookbehind

```java
// Positive lookahead: find "Java" only when followed by "Script"
// (This is a classic trick question — it will NOT match because
//  "JavaScript" contains "Java" followed by "Script")
Pattern p1 = Pattern.compile("Java(?=Script)");
Matcher m1 = p1.matcher("Java and JavaScript and JavaFX");
while (m1.find()) {
    System.out.println("Found at: " + m1.start());  // Found at: 9 (the "Java" in "JavaScript")
}

// Negative lookahead: find "Java" only when NOT followed by "Script"
Pattern p2 = Pattern.compile("Java(?!Script)");
Matcher m2 = p2.matcher("Java and JavaScript and JavaFX");
while (m2.find()) {
    System.out.println("Found at: " + m2.start());  // 0 ("Java "), 26 ("JavaFX")
}

// Positive lookbehind: find amounts preceded by "$"
Pattern p3 = Pattern.compile("(?<=\\$)\\d+\\.\\d{2}");
Matcher m3 = p3.matcher("Price: $19.99, Tax: $1.50, Total: $21.49");
while (m3.find()) {
    System.out.println("Amount: " + m3.group());  // 19.99, 1.50, 21.49
}

// Negative lookbehind: find numbers NOT preceded by "$"
Pattern p4 = Pattern.compile("(?<!\\$)\\b\\d+\\b");
Matcher m4 = p4.matcher("Order 42: $100 and 3 items");
while (m4.find()) {
    System.out.println("Non-dollar number: " + m4.group());  // 42, 3
}
```

### String Convenience Methods

Java's `String` class provides regex methods that internally create `Pattern` and `Matcher` objects:

```java
String input = "The quick brown fox jumps over the lazy dog";

// String.matches() — entire string must match (implicitly anchored with ^ and $)
"12345".matches("\\d+");          // true
"abc123".matches("\\d+");         // false (entire string is not digits)
"abc123".matches(".*\\d+.*");     // true (equivalent to find with \\d+)

// String.replaceAll() — replace all matches
String cleaned = "Call 555-123-4567 or 555-987-6543"
    .replaceAll("\\d{3}-\\d{3}-\\d{4}", "***-***-****");
// "Call ***-***-**** or ***-***-****"

// String.replaceFirst() — replace only the first match
String first = "Error: 404, Error: 500, Error: 503"
    .replaceFirst("Error: \\d+", "Error: REDACTED");
// "Error: REDACTED, Error: 500, Error: 503"

// String.split() — split by regex delimiter
String csv = "apple,  banana , cherry,   date";
String[] fruits = csv.split("\\s*,\\s*");
// ["apple", "banana", "cherry", "date"]

// Split with limit
String path = "/home/user/documents/file.txt";
String[] parts = path.split("/", 3);
// ["", "home", "user/documents/file.txt"]

// String.replaceAll with group references ($1, $2)
String date = "2025-01-15";
String usDate = date.replaceAll("(\\d{4})-(\\d{2})-(\\d{2})", "$2/$3/$1");
// "01/15/2025"
```

**Critical performance warning:** Every call to `String.matches()`, `String.replaceAll()`, `String.replaceFirst()`, and `String.split()` **recompiles the regex internally**. In a loop or hot path, this is wasteful. Precompile instead.

### Precompiled Patterns for Performance

```java
public class TransactionParser {

    // PRECOMPILE: static final, compiled once at class load time
    private static final Pattern AMOUNT_PATTERN =
        Pattern.compile("\\$?(\\d{1,3}(?:,\\d{3})*(?:\\.\\d{2})?)");

    private static final Pattern ACCOUNT_PATTERN =
        Pattern.compile("ACC-(\\d{6,10})");

    private static final Pattern DATE_PATTERN =
        Pattern.compile("(\\d{4})-(\\d{2})-(\\d{2})");

    // This method may be called millions of times — no recompilation overhead
    public BigDecimal extractAmount(String transactionText) {
        Matcher matcher = AMOUNT_PATTERN.matcher(transactionText);
        if (matcher.find()) {
            String raw = matcher.group(1).replace(",", "");
            return new BigDecimal(raw);
        }
        throw new IllegalArgumentException("No amount found in: " + transactionText);
    }

    public String extractAccountId(String transactionText) {
        Matcher matcher = ACCOUNT_PATTERN.matcher(transactionText);
        if (matcher.find()) {
            return "ACC-" + matcher.group(1);
        }
        throw new IllegalArgumentException("No account ID found");
    }
}
```

### Pattern Flags in Practice

```java
// Case-insensitive matching
Pattern ci = Pattern.compile("error", Pattern.CASE_INSENSITIVE);
ci.matcher("ERROR").find();   // true
ci.matcher("Error").find();   // true
ci.matcher("eRrOr").find();   // true

// Inline flag
Pattern ci2 = Pattern.compile("(?i)error");  // same effect

// Multiline: ^ and $ match at line boundaries
String multiline = "line1\nline2\nline3";
Pattern singleLine = Pattern.compile("^line\\d$");
singleLine.matcher(multiline).find();  // false (^ matches only start of string)

Pattern multiLine = Pattern.compile("^line\\d$", Pattern.MULTILINE);
Matcher ml = multiLine.matcher(multiline);
int count = 0;
while (ml.find()) count++;
System.out.println(count);  // 3 (matches "line1", "line2", "line3")

// Dotall: . matches newlines
String withNewlines = "start\nmiddle\nend";
Pattern normal = Pattern.compile("start.*end");
normal.matcher(withNewlines).matches();  // false (. doesn't match \n)

Pattern dotall = Pattern.compile("start.*end", Pattern.DOTALL);
dotall.matcher(withNewlines).matches();  // true (. matches everything)

// Comments mode: ignore whitespace and # comments for complex patterns
Pattern readable = Pattern.compile(
    "\\(" +           // opening paren
    "\\s*" +          // optional whitespace
    "(\\d{3})" +      // area code (group 1)
    "\\s*" +          // optional whitespace
    "\\)" +           // closing paren
    "\\s*" +          // optional whitespace
    "(\\d{3})" +      // exchange (group 2)
    "-" +             // hyphen
    "(\\d{4})",       // subscriber (group 3)
    Pattern.COMMENTS  // allows whitespace and # comments (not useful in this inline form,
                      // but powerful with text blocks)
);
```

### Replacing with Callbacks (Matcher.replaceAll with Function — Java 9+)

```java
import java.util.function.MatchResult;
import java.util.function.Function;

// Replace each match using a function (Java 9+)
Pattern pattern = Pattern.compile("\\b\\w+\\b");
String input = "hello world java regex";

String result = pattern.matcher(input)
    .replaceAll(match -> match.group().toUpperCase());
// "HELLO WORLD JAVA REGEX"

// Fintech example: mask credit card numbers, keeping last 4 digits
Pattern cardPattern = Pattern.compile("\\b(\\d{4})[- ]?(\\d{4})[- ]?(\\d{4})[- ]?(\\d{4})\\b");
String transaction = "Payment with card 4532-1234-5678-9012 for $150.00";

String masked = cardPattern.matcher(transaction)
    .replaceAll(match -> "****-****-****-" + match.group(4));
// "Payment with card ****-****-****-9012 for $150.00"

// Java 8 equivalent (before replaceAll(Function)):
StringBuffer sb = new StringBuffer();
Matcher m = cardPattern.matcher(transaction);
while (m.find()) {
    m.appendReplacement(sb, "****-****-****-" + m.group(4));
}
m.appendTail(sb);
String masked8 = sb.toString();
```

### Splitting with Regex

```java
// Split on multiple delimiters
String data = "apple;banana|cherry, date\tgrape";
String[] items = data.split("[;|,\\s]+");
// ["apple", "banana", "cherry", "date", "grape"]

// Split camelCase into words
String camelCase = "getAccountBalanceById";
String[] words = camelCase.split("(?=[A-Z])");
// ["get", "Account", "Balance", "By", "Id"]

// Split but keep the delimiter (using lookahead)
String log = "INFO:startup WARN:memory ERROR:disk";
String[] entries = log.split("(?=\\b(?:INFO|WARN|ERROR):)");
// ["INFO:startup ", "WARN:memory ", "ERROR:disk"]

// Split with limit (keep the rest as one string)
String config = "key=value=with=equals";
String[] kv = config.split("=", 2);
// ["key", "value=with=equals"]
```

### Matcher Reset and Region

```java
Pattern pattern = Pattern.compile("\\d+");
Matcher matcher = pattern.matcher("abc 123 def 456 ghi 789");

// find() advances through the string
matcher.find();  // "123" at index 4
matcher.find();  // "456" at index 12
matcher.find();  // "789" at index 20
matcher.find();  // false (no more matches)

// reset() starts over from the beginning
matcher.reset();
matcher.find();  // "123" at index 4 again

// reset with new input
matcher.reset("new 999 input");
matcher.find();  // "999"

// region() restricts matching to a substring
matcher.reset("abc 123 def 456 ghi 789");
matcher.region(8, 19);  // only search "def 456 ghi"
while (matcher.find()) {
    System.out.println(matcher.group() + " at " + matcher.start());
    // "456" at 12
    // "789" is outside the region, not found
}
```

### Quoting Literal Strings

When you need to match a literal string that might contain regex metacharacters:

```java
// Pattern.quote() escapes all regex metacharacters
String userInput = "price is $100.00 (USD)";
String search = "$100.00";

// WRONG: $ and . are regex metacharacters
Pattern wrong = Pattern.compile(search);
wrong.matcher(userInput).find();  // might match unexpected things

// RIGHT: quote the literal string
Pattern right = Pattern.compile(Pattern.quote(search));
right.matcher(userInput).find();  // true, matches exactly "$100.00"

// Equivalent to: Pattern.compile("\\Q$100.00\\E")
```

---

## Common Patterns

### Validation Patterns

```java
public class Validators {

    // Email (simplified — full RFC 5322 regex is 6000+ characters)
    private static final Pattern EMAIL = Pattern.compile(
        "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    );

    // Phone number (US format: 555-123-4567, (555) 123-4567, 5551234567)
    private static final Pattern PHONE_US = Pattern.compile(
        "^(?:\\(\\d{3}\\)|\\d{3})[-.\\s]?\\d{3}[-.\\s]?\\d{4}$"
    );

    // IPv4 address
    private static final Pattern IPV4 = Pattern.compile(
        "^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)$"
    );

    // URL (http/https)
    private static final Pattern URL = Pattern.compile(
        "^https?://[\\w.-]+(?:/[\\w./?%&=-]*)?$"
    );

    // Date (YYYY-MM-DD, basic validation)
    private static final Pattern ISO_DATE = Pattern.compile(
        "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])$"
    );

    // Credit card (Luhn check should be done separately)
    private static final Pattern CREDIT_CARD = Pattern.compile(
        "^\\d{4}[- ]?\\d{4}[- ]?\\d{4}[- ]?\\d{4}$"
    );

    // Strong password (8+ chars, upper, lower, digit, special)
    private static final Pattern STRONG_PASSWORD = Pattern.compile(
        "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$"
    );

    public static boolean isValidEmail(String email) {
        return EMAIL.matcher(email).matches();
    }

    public static boolean isValidPhone(String phone) {
        return PHONE_US.matcher(phone).matches();
    }

    public static boolean isValidIpv4(String ip) {
        return IPV4.matcher(ip).matches();
    }

    public static boolean isValidUrl(String url) {
        return URL.matcher(url).matches();
    }

    public static boolean isValidDate(String date) {
        return ISO_DATE.matcher(date).matches();
    }

    public static boolean isValidCreditCard(String card) {
        return CREDIT_CARD.matcher(card).matches();
    }

    public static boolean isStrongPassword(String password) {
        return STRONG_PASSWORD.matcher(password).matches();
    }
}
```

### Extraction Patterns

```java
public class Extractors {

    // Extract all URLs from text
    private static final Pattern URL_PATTERN = Pattern.compile(
        "https?://[\\w.-]+(?:/[\\w./?%&=#-]*)?"
    );

    public static List<String> extractUrls(String text) {
        List<String> urls = new ArrayList<>();
        Matcher matcher = URL_PATTERN.matcher(text);
        while (matcher.find()) {
            urls.add(matcher.group());
        }
        return urls;
    }

    // Extract key-value pairs from "key=value" format
    private static final Pattern KV_PATTERN = Pattern.compile(
        "(\\w+)\\s*=\\s*\"?([^\"\\n]+)\"?"
    );

    public static Map<String, String> extractKeyValuePairs(String config) {
        Map<String, String> map = new LinkedHashMap<>();
        Matcher matcher = KV_PATTERN.matcher(config);
        while (matcher.find()) {
            map.put(matcher.group(1), matcher.group(2).trim());
        }
        return map;
    }

    // Extract all monetary amounts from text
    private static final Pattern MONEY_PATTERN = Pattern.compile(
        "\\$\\d{1,3}(?:,\\d{3})*(?:\\.\\d{2})?"
    );

    public static List<String> extractAmounts(String text) {
        List<String> amounts = new ArrayList<>();
        Matcher matcher = MONEY_PATTERN.matcher(text);
        while (matcher.find()) {
            amounts.add(matcher.group());
        }
        return amounts;
    }
}
```

### Fintech-Specific Patterns

```java
public class FintechPatterns {

    // SWIFT/BIC code (8 or 11 characters)
    private static final Pattern SWIFT = Pattern.compile(
        "^[A-Z]{4}[A-Z]{2}[A-Z0-9]{2}([A-Z0-9]{3})?$"
    );

    // IBAN (International Bank Account Number, simplified)
    private static final Pattern IBAN = Pattern.compile(
        "^[A-Z]{2}\\d{2}[A-Z0-9]{4,30}$"
    );

    // ISO 4217 currency code (3 uppercase letters)
    private static final Pattern CURRENCY_CODE = Pattern.compile(
        "^[A-Z]{3}$"
    );

    // Transaction reference (e.g., TXN-20250115-ABC123)
    private static final Pattern TXN_REF = Pattern.compile(
        "^TXN-\\d{8}-[A-Z0-9]{6,12}$"
    );

    // Mask a credit card number: show only last 4 digits
    private static final Pattern CARD_MASK = Pattern.compile(
        "(\\d{4})[- ]?(\\d{4})[- ]?(\\d{4})[- ]?(\\d{4})"
    );

    public static String maskCardNumber(String cardNumber) {
        return CARD_MASK.matcher(cardNumber)
            .replaceAll("****-****-****-$4");
    }

    // Parse a monetary amount string into BigDecimal
    private static final Pattern AMOUNT = Pattern.compile(
        "-?\\$?([\\d,]+(?:\\.\\d{1,2})?)"
    );

    public static BigDecimal parseAmount(String text) {
        Matcher matcher = AMOUNT.matcher(text);
        if (matcher.find()) {
            String cleaned = matcher.group(1).replace(",", "");
            return new BigDecimal(cleaned);
        }
        throw new IllegalArgumentException("No valid amount in: " + text);
    }
}
```

---

## Important Notes

### Performance Best Practices

```
1. ALWAYS precompile patterns used more than once:
   private static final Pattern P = Pattern.compile("...");
   → Pattern compilation involves parsing, optimization, and bytecode generation.
   → String.matches() recompiles on every call. In a loop processing 1M records,
     this can add seconds of overhead.

2. Use possessive quantifiers when backtracking is unnecessary:
   "\\d++" instead of "\\d+" when you know digits won't need to backtrack.
   → Prevents catastrophic backtracking on pathological inputs.

3. Avoid .* at the beginning of patterns:
   ".*error" forces the engine to scan the entire string before backtracking.
   Use find() without the leading .* instead.

4. Use non-capturing groups (?:...) when you don't need the captured text:
   → Slightly faster, uses less memory.

5. Use Matcher.reset(input) to reuse a Matcher with new input:
   → Avoids creating a new Matcher object per input string.

6. For simple literal searches, use String.contains() or String.indexOf():
   → Much faster than regex for exact substring matching.
   → "hello".contains("ell") is faster than Pattern.compile("ell").matcher("hello").find()
```

### Catastrophic Backtracking (ReDoS)

Certain regex patterns can cause **exponential time complexity** on specific inputs. This is a real security vulnerability called **Regular Expression Denial of Service (ReDoS)**:

```java
// VULNERABLE pattern: nested quantifiers with overlapping alternatives
Pattern evil = Pattern.compile("^(a+)+$");

// This input causes exponential backtracking:
String input = "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaab";
evil.matcher(input).matches();
// This will hang for a very long time (or forever)

// SAFE alternative: use possessive quantifiers or atomic groups
Pattern safe = Pattern.compile("^(a++)+$");
// Or restructure to avoid nested quantifiers
Pattern safe2 = Pattern.compile("^a+$");
```

**Rules to avoid ReDoS:**
```
- Avoid nested quantifiers: (a+)+, (a*)*, (a+)*
- Avoid overlapping alternatives: (a|a)+, (\\d|\\w)+
- Use possessive quantifiers (++, *+) or atomic groups (?>...) when possible
- Test patterns with long, non-matching inputs
- Use tools like https://regex101.com to check for backtracking issues
- Set timeouts on regex matching if processing untrusted input (not built into
  java.util.regex — requires running in a separate thread with a timeout)
```

### Common Pitfalls

```
1. Forgetting to double backslashes:
   WRONG: Pattern.compile("\d+")    → Java interprets \d as an invalid escape
   RIGHT: Pattern.compile("\\d+")   → regex engine sees \d+

2. Using matches() when you mean find():
   "abc123def".matches("\\d+")  → false (entire string must be digits)
   "abc123def" contains digits, but matches() requires full-string match

3. Forgetting that . does not match newlines by default:
   "line1\nline2".matches(".*")  → false
   Use Pattern.DOTALL or (?s) flag

4. Greedy quantifiers consuming too much:
   "<b>bold</b>".replaceAll("<.*>", "")  → "" (greedy eats everything)
   "<b>bold</b>".replaceAll("<.*?>", "") → "bold" (reluctant stops at first >)

5. Not anchoring validation patterns:
   Pattern.compile("\\d{3}") matches "abc123def" with find()
   Pattern.compile("^\\d{3}$") only matches exactly "123" with matches()
   Always use ^ and $ (or matches()) for validation

6. Assuming regex validates semantics:
   Regex can validate format (e.g., "99/99/9999" matches \\d{2}/\\d{2}/\\d{4})
   but cannot validate meaning (month 99 is invalid). Use java.time for date validation.

7. Using regex to parse HTML or XML:
   Don't. Use a proper parser (JSoup, Jackson XML, DOM/SAX).
   Regex cannot handle nested structures reliably.

8. Mutable Matcher shared across threads:
   Pattern is thread-safe. Matcher is NOT.
   Each thread must create its own Matcher from the shared Pattern.
```

### When NOT to Use Regex

```
- Simple string operations: use String.contains(), startsWith(), endsWith(), indexOf()
- Parsing structured formats: use JSON (Jackson), XML (JAXB), CSV (OpenCSV) parsers
- Date/time validation: use java.time's DateTimeFormatter.parse()
- Numeric parsing: use Integer.parseInt(), BigDecimal constructor
- HTML/XML extraction: use JSoup or a DOM parser
- Complex grammar parsing: use a parser generator (ANTLR)

Regex is the right tool for:
- Pattern-based validation (emails, phone numbers, IDs)
- Extracting fields from semi-structured text (log lines, free-form input)
- Simple text transformations (masking, reformatting)
- Tokenizing simple formats (splitting on delimiters)
```

### Regex and Unicode

```java
// \w only matches [a-zA-Z0-9_] by default — not accented characters
"café".matches("\\w+");  // false (é is not in \w)

// Use UNICODE_CHARACTER_CLASS flag for Unicode-aware \w, \d, \s
Pattern unicode = Pattern.compile("\\w+", Pattern.UNICODE_CHARACTER_CLASS);
unicode.matcher("café").matches();  // true

// Or use Unicode property escapes
Pattern letters = Pattern.compile("\\p{L}+");  // any Unicode letter
letters.matcher("café").matches();  // true
letters.matcher("日本語").matches(); // true
letters.matcher("العربية").matches(); // true

// Specific Unicode blocks
Pattern cyrillic = Pattern.compile("\\p{IsCyrillic}+");
Pattern greek = Pattern.compile("\\p{IsGreek}+");
```

---

## Practice

```
1. Write a regex to validate US ZIP codes (5 digits, optional +4: "12345" or "12345-6789")
2. Extract all email addresses from a block of text using Pattern and Matcher
3. Write a regex to validate ISO 8601 dates (YYYY-MM-DD) with basic month/day range checking
4. Use String.replaceAll() to convert "camelCaseString" to "snake_case_string"
5. Parse a log line "2025-01-15 14:30:45 [ERROR] PaymentService: Transaction 42 failed"
   into its components using named groups
6. Write a method that masks all credit card numbers in a string, showing only last 4 digits
7. Use a lookahead to find all words in a string that are followed by a comma
8. Write a regex to validate IPv4 addresses (each octet 0-255)
9. Use Pattern.split() to tokenize a mathematical expression "3+5*2-8/4" into numbers and operators
10. Demonstrate the difference between greedy, reluctant, and possessive quantifiers
    on the input "aaaaab" with the pattern "a+b"
11. Write a regex to extract all hashtags (#word) from a social media post
12. Use Matcher.replaceAll(Function) (Java 9+) to title-case every word in a sentence
13. Write a validator for SWIFT/BIC codes and test it with valid and invalid inputs
14. Create a precompiled Pattern for parsing CSV lines (handling quoted fields with commas)
15. Benchmark String.matches() vs precompiled Pattern.matcher().matches() in a loop
    of 1,000,000 iterations and compare execution times
```

---

## References

- `java.util.regex` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/regex/package-summary.html
- `Pattern` Javadoc (full syntax reference): https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/regex/Pattern.html
- Oracle Regex Tutorial: https://docs.oracle.com/javase/tutorial/essential/regex/
- Regular-Expressions.info (comprehensive reference): https://www.regular-expressions.info/
- Regex101 (interactive tester with Java flavor): https://regex101.com/
- OWASP ReDoS: https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS
