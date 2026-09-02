## Overview

Java 8 introduced functional programming constructs — lambda expressions, method references, and the Stream API — that fundamentally changed how Java code is written. Before Java 8, processing a list of transactions required explicit for-loops, mutable accumulator variables, and verbose anonymous inner classes. After Java 8, the same logic can be expressed as a declarative pipeline that reads like a specification of what to do rather than a recipe for how to do it. Functional programming in Java does not replace object-oriented programming. It complements it. You will use OOP to model your domain (Account, Transaction, Payment) and functional constructs to process collections of those objects (filter, transform, aggregate). Every Spring Boot application, every data pipeline, and every collection-processing operation you write will use these features. Mastery of lambdas and streams is not optional — it is the baseline for modern Java development.

---

## Core Concepts

### Functional Interfaces

A functional interface is an interface with exactly one abstract method. It may have any number of default or static methods, but only one abstract method. Functional interfaces are the target types for lambda expressions and method references.

**The `@FunctionalInterface` annotation:**

```java
@FunctionalInterface
public interface TransactionValidator {
    boolean validate(Transaction transaction);
    // Only one abstract method allowed

    // Default methods do NOT count toward the limit
    default void logValidation(Transaction tx, boolean result) {
        System.out.println("Validation of " + tx.getId() + ": " + result);
    }

    // Static methods do NOT count toward the limit
    static TransactionValidator alwaysValid() {
        return tx -> true;
    }
}

// Compilation error if you add a second abstract method:
// @FunctionalInterface
// public interface Broken {
//     void method1();
//     void method2();  // ERROR: not a functional interface
// }
```

**Predefined functional interfaces in `java.util.function`:**

Java provides 43 built-in functional interfaces. The four foundational ones are:

| Interface | Abstract Method | Purpose | Example |
|-----------|----------------|---------|---------|
| `Function<T, R>` | `R apply(T t)` | Transforms input to output | Extract a field, convert types |
| `Predicate<T>` | `boolean test(T t)` | Tests a condition | Filter, validate |
| `Consumer<T>` | `void accept(T t)` | Performs an action | Print, save, notify |
| `Supplier<T>` | `T get()` | Produces a value | Factory, lazy initialization |

**Function<T, R>:**

```java
// Transform a Transaction to its amount
Function<Transaction, BigDecimal> getAmount = tx -> tx.getAmount();

// Transform a String to its length
Function<String, Integer> stringLength = s -> s.length();

// Compose functions
Function<String, String> trim = String::trim;
Function<String, String> toUpper = String::toUpperCase;
Function<String, String> trimAndUpper = trim.andThen(toUpper);

String result = trimAndUpper.apply("  hello  ");  // "HELLO"

// Compose in reverse order
Function<String, String> upperAndTrim = trim.compose(toUpper);
```

**Predicate<T>:**

```java
// Test if a transaction amount exceeds a threshold
Predicate<Transaction> isLarge = tx ->
    tx.getAmount().compareTo(new BigDecimal("10000")) > 0;

// Test if a transaction is completed
Predicate<Transaction> isCompleted = tx ->
    tx.getStatus() == TransactionStatus.COMPLETED;

// Combine predicates
Predicate<Transaction> isLargeAndCompleted = isLarge.and(isCompleted);
Predicate<Transaction> isLargeOrCompleted = isLarge.or(isCompleted);
Predicate<Transaction> isNotLarge = isLarge.negate();

// Usage
boolean result = isLargeAndCompleted.test(someTransaction);
```

**Consumer<T>:**

```java
// Print a transaction
Consumer<Transaction> printTx = tx ->
    System.out.printf("TX %s: $%s%n", tx.getId(), tx.getAmount());

// Save a transaction
Consumer<Transaction> saveTx = tx -> repository.save(tx);

// Chain consumers
Consumer<Transaction> logAndSave = printTx.andThen(saveTx);

logAndSave.accept(transaction);  // Prints, then saves
```

**Supplier<T>:**

```java
// Generate a new UUID
Supplier<String> uuidSupplier = () -> UUID.randomUUID().toString();

// Lazy initialization
Supplier<BigDecimal> currentExchangeRate = () ->
    exchangeRateService.getRate("USD", "EUR");

// Factory
Supplier<Transaction> txFactory = () ->
    new Transaction(UUID.randomUUID().toString(), BigDecimal.ZERO, "USD");

String id = uuidSupplier.get();
BigDecimal rate = currentExchangeRate.get();  // Called only when needed
```

**Primitive specializations:**

The generic interfaces box primitive types, which adds overhead. Java provides primitive specializations for performance-critical code:

| Generic | Primitive Specializations |
|---------|--------------------------|
| `Function<T, R>` | `IntFunction<R>`, `LongFunction<R>`, `DoubleFunction<R>`, `ToIntFunction<T>`, `ToLongFunction<T>`, `ToDoubleFunction<T>`, `IntToLongFunction`, etc. |
| `Predicate<T>` | `IntPredicate`, `LongPredicate`, `DoublePredicate` |
| `Consumer<T>` | `IntConsumer`, `LongConsumer`, `DoubleConsumer` |
| `Supplier<T>` | `IntSupplier`, `LongSupplier`, `DoubleSupplier`, `BooleanSupplier` |

```java
// Generic — boxes int to Integer
Function<Integer, Integer> doubleIt = n -> n * 2;

// Primitive — no boxing
IntUnaryOperator doubleItFast = n -> n * 2;

// ToIntFunction avoids boxing the return value
ToIntFunction<Transaction> amountCents = tx ->
    tx.getAmount().multiply(new BigDecimal("100")).intValue();
```

**Bi-variants (two input parameters):**

| Interface | Method | Example |
|-----------|--------|---------|
| `BiFunction<T, U, R>` | `R apply(T t, U u)` | Merge two values |
| `BiPredicate<T, U>` | `boolean test(T t, U u)` | Compare two values |
| `BiConsumer<T, U>` | `void accept(T t, U u)` | Process a key-value pair |

```java
BiFunction<BigDecimal, BigDecimal, BigDecimal> add = BigDecimal::add;
BiPredicate<String, String> equalsIgnoreCase = String::equalsIgnoreCase;
BiConsumer<String, BigDecimal> printBalance = (name, balance) ->
    System.out.printf("%s: $%s%n", name, balance);
```

### Lambda Expressions

A lambda expression is an anonymous function — a block of code that can be passed as an argument, stored in a variable, or returned from a method.

**Syntax:**

```java
// Full syntax: (parameters) -> { body }
(Transaction tx) -> { return tx.getAmount().compareTo(BigDecimal.ZERO) > 0; }

// Single expression — return and braces are implicit
(Transaction tx) -> tx.getAmount().compareTo(BigDecimal.ZERO) > 0

// Single parameter — parentheses are optional
tx -> tx.getAmount().compareTo(BigDecimal.ZERO) > 0

// No parameters
() -> System.out.println("Processing...")
() -> UUID.randomUUID().toString()

// Multiple parameters
(a, b) -> a.compareTo(b)
(String name, BigDecimal amount) -> new Transaction(name, amount)

// Multiple statements — braces and explicit return required
tx -> {
    log.info("Processing transaction {}", tx.getId());
    validate(tx);
    return repository.save(tx);
}
```

**Type inference:**

The compiler infers the parameter types from the target functional interface. You rarely need to specify them explicitly.

```java
// The compiler knows that Predicate<Transaction>.test() takes a Transaction
Predicate<Transaction> isValid = tx -> tx.getAmount().compareTo(BigDecimal.ZERO) > 0;
// tx is inferred as Transaction

// Explicit types (rarely needed, but legal)
Predicate<Transaction> isValid = (Transaction tx) -> tx.getAmount().compareTo(BigDecimal.ZERO) > 0;
```

**Effectively final variables (closures):**

A lambda can access local variables from the enclosing scope, but those variables must be **effectively final** — assigned once and never reassigned.

```java
public List<Transaction> filterByCurrency(List<Transaction> transactions, String currency) {
    // currency is effectively final — never reassigned
    return transactions.stream()
        .filter(tx -> tx.getCurrency().equals(currency))  // Lambda captures currency
        .toList();
}

public void demonstrateEffectivelyFinal() {
    BigDecimal threshold = new BigDecimal("1000");

    // This works — threshold is effectively final
    Predicate<Transaction> isAboveThreshold = tx ->
        tx.getAmount().compareTo(threshold) > 0;

    // threshold = new BigDecimal("5000");  // If you uncomment this,
    // the lambda above will not compile because threshold is no longer effectively final
}
```

**Lambda vs anonymous inner class:**

```java
// Anonymous inner class (pre-Java 8)
Comparator<Transaction> byAmount = new Comparator<Transaction>() {
    @Override
    public int compare(Transaction t1, Transaction t2) {
        return t1.getAmount().compareTo(t2.getAmount());
    }
};

// Lambda (Java 8+)
Comparator<Transaction> byAmount = (t1, t2) ->
    t1.getAmount().compareTo(t2.getAmount());

// Method reference (even more concise)
Comparator<Transaction> byAmount = Comparator.comparing(Transaction::getAmount);
```

Key differences:
- Lambdas do not create a new class file. Anonymous inner classes do.
- In a lambda, `this` refers to the enclosing class. In an anonymous inner class, `this` refers to the anonymous class itself.
- Lambdas can only implement functional interfaces. Anonymous inner classes can implement any interface or extend any class.

### Method References

Method references are shorthand for lambdas that call a single existing method. They use the `::` operator.

**Four types of method references:**

**1. Static method reference — `ClassName::methodName`**

```java
// Lambda
Function<String, BigDecimal> parseAmount = s -> new BigDecimal(s);

// Method reference
Function<String, BigDecimal> parseAmount = BigDecimal::new;  // Constructor reference

// Static method
Function<String, Integer> parseInt = Integer::parseInt;
Predicate<String> isBlank = String::isBlank;
Consumer<Object> print = System.out::println;
```

**2. Instance method reference on a specific object — `object::methodName`**

```java
Account account = new Account("ACC-001", "Alice", new BigDecimal("5000"));

// Lambda
Supplier<BigDecimal> getBalance = () -> account.getBalance();

// Method reference
Supplier<BigDecimal> getBalance = account::getBalance;

// Another example
String prefix = "TX-";
Predicate<String> hasPrefix = prefix::startsWith;  // "TX-" is the specific object
```

**3. Instance method reference on an arbitrary object — `ClassName::methodName`**

```java
// Lambda
Function<String, String> toUpper = s -> s.toUpperCase();

// Method reference — the parameter becomes the object on which the method is called
Function<String, String> toUpper = String::toUpperCase;

// More examples
Function<String, Integer> length = String::length;
Predicate<String> isEmpty = String::isEmpty;
Comparator<String> compareTo = String::compareTo;

// With two parameters — the first parameter is the object, the second is the argument
BiPredicate<String, String> equalsIgnoreCase = String::equalsIgnoreCase;
// Equivalent to: (s1, s2) -> s1.equalsIgnoreCase(s2)
```

**4. Constructor reference — `ClassName::new`**

```java
// Lambda
Function<String, StringBuilder> createBuilder = s -> new StringBuilder(s);

// Constructor reference
Function<String, StringBuilder> createBuilder = StringBuilder::new;

// No-arg constructor
Supplier<ArrayList<Transaction>> listFactory = ArrayList::new;

// Multi-arg constructor
BiFunction<String, BigDecimal, Transaction> txFactory = Transaction::new;
// Equivalent to: (id, amount) -> new Transaction(id, amount)

// Array constructor
Function<Integer, String[]> stringArrayFactory = String[]::new;
String[] array = stringArrayFactory.apply(10);  // new String[10]
```

### The Stream API

A stream is a sequence of elements that supports sequential and parallel aggregate operations. Streams are not data structures — they do not store elements. They are pipelines that carry elements from a source through a series of operations to a terminal result.

**Key characteristics:**

- **Lazy.** Intermediate operations are not executed until a terminal operation is invoked. This allows the stream to optimize the entire pipeline.
- **Single-use.** A stream can be consumed only once. Attempting to reuse a stream throws `IllegalStateException`.
- **Non-mutating.** Stream operations do not modify the source collection. They produce new results.
- **Functional.** Stream operations take functions (lambdas, method references) as parameters and do not have side effects (ideally).

**Stream pipeline structure:**

```
Source → Intermediate Operation(s) → Terminal Operation
         (lazy, zero or more)        (eager, exactly one)
```

#### Creating Streams

```java
// From a Collection
List<Transaction> transactions = getTransactions();
Stream<Transaction> stream = transactions.stream();

// From an array
String[] currencies = {"USD", "EUR", "GBP"};
Stream<String> currencyStream = Arrays.stream(currencies);
Stream<String> partial = Arrays.stream(currencies, 1, 3);  // "EUR", "GBP"

// From individual values
Stream<String> single = Stream.of("USD");
Stream<String> multiple = Stream.of("USD", "EUR", "GBP", "JPY");

// From a file (lazy, line by line)
try (Stream<String> lines = Files.lines(Path.of("data/transactions.csv"))) {
    lines.filter(line -> line.startsWith("TX-"))
         .forEach(System.out::println);
}

// Infinite streams
Stream<Integer> naturals = Stream.iterate(1, n -> n + 1);
// 1, 2, 3, 4, 5, ... (infinite — must be limited)

Stream<BigDecimal> randomAmounts = Stream.generate(() ->
    new BigDecimal(ThreadLocalRandom.current().nextDouble(1, 10000))
        .setScale(2, RoundingMode.HALF_UP)
);

// Java 9+ iterate with condition
Stream<Integer> bounded = Stream.iterate(1, n -> n <= 100, n -> n + 1);
// 1, 2, 3, ..., 100

// Java 9+ takeWhile and dropWhile
Stream<Integer> whilePositive = Stream.of(1, 2, 3, -1, 4, 5)
    .takeWhile(n -> n > 0);  // 1, 2, 3

// Primitive streams (avoid boxing overhead)
IntStream intStream = IntStream.range(1, 100);       // 1 to 99
IntStream intStreamClosed = IntStream.rangeClosed(1, 100);  // 1 to 100
LongStream longStream = LongStream.of(1L, 2L, 3L);
DoubleStream doubleStream = DoubleStream.of(1.0, 2.0, 3.0);

// From a pattern (regex)
Stream<String> words = Pattern.compile("\\s+")
    .splitAsStream("Hello World from Java");
// "Hello", "World", "from", "Java"
```

#### Intermediate Operations (Lazy)

Intermediate operations transform a stream into another stream. They are not executed until a terminal operation triggers the pipeline.

**filter — keep elements matching a predicate:**

```java
List<Transaction> largeTransactions = transactions.stream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("10000")) > 0)
    .filter(tx -> tx.getStatus() == TransactionStatus.COMPLETED)
    .toList();
```

**map — transform each element:**

```java
// Extract amounts
List<BigDecimal> amounts = transactions.stream()
    .map(Transaction::getAmount)
    .toList();

// Transform to DTOs
List<TransactionDto> dtos = transactions.stream()
    .map(tx -> new TransactionDto(tx.getId(), tx.getAmount(), tx.getCurrency()))
    .toList();

// Convert types
List<String> ids = transactions.stream()
    .map(Transaction::getId)
    .toList();
```

**flatMap — transform each element into a stream, then flatten:**

```java
// Each account has a list of transactions. Flatten all transactions into one stream.
List<Account> accounts = getAccounts();

List<Transaction> allTransactions = accounts.stream()
    .flatMap(account -> account.getTransactions().stream())
    .toList();

// Split strings into words
List<String> sentences = List.of("Hello World", "Java Streams", "Functional Programming");
List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .toList();
// ["Hello", "World", "Java", "Streams", "Functional", "Programming"]
```

**distinct — remove duplicates (uses equals/hashCode):**

```java
List<String> uniqueCurrencies = transactions.stream()
    .map(Transaction::getCurrency)
    .distinct()
    .toList();
```

**sorted — sort elements:**

```java
// Natural order
List<String> sortedNames = names.stream()
    .sorted()
    .toList();

// Custom comparator
List<Transaction> byAmountDesc = transactions.stream()
    .sorted(Comparator.comparing(Transaction::getAmount).reversed())
    .toList();

// Multi-field sort
List<Transaction> byDateThenAmount = transactions.stream()
    .sorted(Comparator.comparing(Transaction::getDate)
        .thenComparing(Transaction::getAmount, Comparator.reverseOrder()))
    .toList();
```

**peek — inspect elements without modifying (primarily for debugging):**

```java
List<Transaction> processed = transactions.stream()
    .filter(tx -> tx.getAmount().compareTo(BigDecimal.ZERO) > 0)
    .peek(tx -> log.debug("Processing: {}", tx.getId()))  // Debug logging
    .map(tx -> enrichWithExchangeRate(tx))
    .peek(tx -> log.debug("Enriched: {}", tx))
    .toList();
```

**limit and skip — truncate the stream:**

```java
// First 10 transactions
List<Transaction> first10 = transactions.stream()
    .limit(10)
    .toList();

// Skip first 100, take next 50 (pagination)
List<Transaction> page3 = transactions.stream()
    .skip(100)
    .limit(50)
    .toList();
```

**mapToInt, mapToLong, mapToDouble — convert to primitive streams:**

```java
// Avoids boxing overhead for numerical operations
int totalCents = transactions.stream()
    .mapToInt(tx -> tx.getAmount().multiply(new BigDecimal("100")).intValue())
    .sum();

double averageAmount = transactions.stream()
    .mapToDouble(tx -> tx.getAmount().doubleValue())
    .average()
    .orElse(0.0);
```

#### Terminal Operations (Eager)

Terminal operations trigger the execution of the entire pipeline and produce a result or side effect.

**forEach — perform an action on each element:**

```java
transactions.stream()
    .filter(tx -> tx.getStatus() == TransactionStatus.FAILED)
    .forEach(tx -> notificationService.alert(tx));

// forEachOrdered — guarantees encounter order (important for parallel streams)
transactions.stream()
    .parallel()
    .sorted(Comparator.comparing(Transaction::getDate))
    .forEachOrdered(tx -> System.out.println(tx.getId()));
```

**collect — accumulate elements into a collection or other result:**

```java
// To List
List<Transaction> list = transactions.stream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0)
    .collect(Collectors.toList());

// To unmodifiable List (Java 16+ — preferred)
List<Transaction> immutable = transactions.stream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0)
    .toList();

// To Set
Set<String> currencies = transactions.stream()
    .map(Transaction::getCurrency)
    .collect(Collectors.toSet());

// To Map
Map<String, Transaction> byId = transactions.stream()
    .collect(Collectors.toMap(Transaction::getId, Function.identity()));

// To Map with merge function (handle duplicate keys)
Map<String, BigDecimal> totalByCurrency = transactions.stream()
    .collect(Collectors.toMap(
        Transaction::getCurrency,
        Transaction::getAmount,
        BigDecimal::add  // Merge function: add amounts for same currency
    ));

// To Map with specific Map implementation
Map<String, List<Transaction>> byCurrencyOrdered = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getCurrency,
        LinkedHashMap::new,  // Preserve insertion order
        Collectors.toList()
    ));
```

**reduce — combine all elements into a single value:**

```java
// Sum of all amounts
BigDecimal totalAmount = transactions.stream()
    .map(Transaction::getAmount)
    .reduce(BigDecimal.ZERO, BigDecimal::add);

// Maximum amount
Optional<BigDecimal> maxAmount = transactions.stream()
    .map(Transaction::getAmount)
    .reduce(BigDecimal::max);

// Count characters in all transaction descriptions
int totalChars = transactions.stream()
    .map(Transaction::getDescription)
    .reduce(0, (sum, desc) -> sum + desc.length(), Integer::sum);
    // Three-arg reduce: identity, accumulator, combiner (for parallel streams)
```

**count, min, max, sum, average:**

```java
long count = transactions.stream().count();

Optional<Transaction> smallest = transactions.stream()
    .min(Comparator.comparing(Transaction::getAmount));

Optional<Transaction> largest = transactions.stream()
    .max(Comparator.comparing(Transaction::getAmount));

int sum = IntStream.rangeClosed(1, 100).sum();  // 5050

OptionalDouble avg = transactions.stream()
    .mapToDouble(tx -> tx.getAmount().doubleValue())
    .average();
```

**findFirst and findAny:**

```java
// First matching element (deterministic)
Optional<Transaction> firstLarge = transactions.stream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("10000")) > 0)
    .findFirst();

// Any matching element (faster in parallel streams, non-deterministic)
Optional<Transaction> anyFailed = transactions.stream()
    .filter(tx -> tx.getStatus() == TransactionStatus.FAILED)
    .findAny();
```

**anyMatch, allMatch, noneMatch:**

```java
boolean hasFailed = transactions.stream()
    .anyMatch(tx -> tx.getStatus() == TransactionStatus.FAILED);

boolean allPositive = transactions.stream()
    .allMatch(tx -> tx.getAmount().compareTo(BigDecimal.ZERO) > 0);

boolean noDuplicates = transactions.stream()
    .map(Transaction::getId)
    .noneMatch(id -> seenIds.contains(id));
```

**toArray:**

```java
Transaction[] array = transactions.stream()
    .filter(tx -> tx.getStatus() == TransactionStatus.COMPLETED)
    .toArray(Transaction[]::new);
```

#### Collectors

`Collectors` is a utility class with factory methods for common reduction operations. It is the most powerful part of the Stream API.

**Grouping:**

```java
// Group by currency
Map<String, List<Transaction>> byCurrency = transactions.stream()
    .collect(Collectors.groupingBy(Transaction::getCurrency));

// Group by status, count each
Map<TransactionStatus, Long> countByStatus = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getStatus,
        Collectors.counting()
    ));

// Group by currency, sum amounts
Map<String, BigDecimal> totalByCurrency = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getCurrency,
        Collectors.reducing(BigDecimal.ZERO, Transaction::getAmount, BigDecimal::add)
    ));

// Multi-level grouping
Map<String, Map<TransactionStatus, List<Transaction>>> byCurrencyAndStatus =
    transactions.stream()
        .collect(Collectors.groupingBy(
            Transaction::getCurrency,
            Collectors.groupingBy(Transaction::getStatus)
        ));

// Partition (special case of grouping by a boolean predicate)
Map<Boolean, List<Transaction>> partitioned = transactions.stream()
    .collect(Collectors.partitioningBy(
        tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0
    ));
// partitioned.get(true)  — large transactions
// partitioned.get(false) — small transactions
```

**Joining:**

```java
String csv = transactions.stream()
    .map(Transaction::getId)
    .collect(Collectors.joining(","));
// "TX-001,TX-002,TX-003"

String formatted = transactions.stream()
    .map(tx -> "%s: $%s".formatted(tx.getId(), tx.getAmount()))
    .collect(Collectors.joining("\n", "=== Transactions ===\n", "\n=== End ==="));
```

**Summarizing:**

```java
DoubleSummaryStatistics stats = transactions.stream()
    .mapToDouble(tx -> tx.getAmount().doubleValue())
    .summaryStatistics();

System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Avg: " + stats.getAverage());
```

**Mapping within collectors:**

```java
// Collect distinct currencies per account
Map<String, Set<String>> currenciesPerAccount = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getAccountId,
        Collectors.mapping(Transaction::getCurrency, Collectors.toSet())
    ));

// Flat mapping within collectors (Java 9+)
Map<String, List<String>> tagsPerAccount = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getAccountId,
        Collectors.flatMapping(
            tx -> tx.getTags().stream(),
            Collectors.toList()
        )
    ));
```

**collectingAndThen:**

```java
// Collect to unmodifiable list
List<Transaction> unmodifiable = transactions.stream()
    .filter(tx -> tx.getStatus() == TransactionStatus.COMPLETED)
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));
```

### Parallel Streams

Parallel streams distribute the workload across multiple threads using the Fork/Join framework. They can improve performance for CPU-bound operations on large datasets.

```java
// Sequential (default)
long count = transactions.stream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0)
    .count();

// Parallel
long countParallel = transactions.parallelStream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0)
    .count();

// Or convert an existing stream
long countParallel2 = transactions.stream()
    .parallel()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0)
    .count();
```

**When to use parallel streams:**

- The dataset is large (typically 10,000+ elements).
- The operation is CPU-bound (computation, not I/O).
- The operation is stateless and side-effect-free.
- The stream source is efficiently splittable (`ArrayList`, arrays).

**When to avoid parallel streams:**

- The dataset is small (overhead of thread management exceeds the benefit).
- The operation involves I/O (database calls, network requests, file reads). Parallel I/O can overwhelm the target system and exhaust connection pools.
- The operation has side effects (modifying shared state). Parallel streams with side effects produce race conditions.
- The stream source is a `LinkedList` (poor split characteristics).
- Order matters and you are using `limit()`, `skip()`, or `findFirst()` (ordering overhead in parallel).

**Thread safety with parallel streams:**

```java
// WRONG — race condition on shared mutable state
List<Transaction> results = new ArrayList<>();  // NOT thread-safe
transactions.parallelStream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0)
    .forEach(results::add);  // ConcurrentModificationException or data loss!

// CORRECT — use collect (thread-safe accumulation)
List<Transaction> results = transactions.parallelStream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0)
    .toList();

// CORRECT — use a concurrent collection if you must use forEach
List<Transaction> results = new CopyOnWriteArrayList<>();
transactions.parallelStream()
    .filter(tx -> tx.getAmount().compareTo(new BigDecimal("1000")) > 0)
    .forEach(results::add);
```

---

## Code Examples

**A complete financial data processing pipeline:**

```java
package com.example.fintech.streams;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.time.LocalDate;
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class StreamDemo {

    public static void main(String[] args) {
        List<Transaction> transactions = generateSampleData();

        // 1. Filter and transform
        List<String> largeUsdTransactionIds = transactions.stream()
            .filter(tx -> "USD".equals(tx.currency()))
            .filter(tx -> tx.amount().compareTo(new BigDecimal("1000")) > 0)
            .sorted(Comparator.comparing(Transaction::amount).reversed())
            .map(Transaction::id)
            .toList();

        System.out.println("Large USD transactions: " + largeUsdTransactionIds);

        // 2. Aggregate
        BigDecimal totalAmount = transactions.stream()
            .filter(tx -> tx.status() == TransactionStatus.COMPLETED)
            .map(Transaction::amount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);

        System.out.println("Total completed amount: $" + totalAmount);

        // 3. Group and summarize
        Map<String, DoubleSummaryStatistics> statsByCurrency = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::currency,
                Collectors.summarizingDouble(tx -> tx.amount().doubleValue())
            ));

        statsByCurrency.forEach((currency, stats) ->
            System.out.printf("%s: count=%d, avg=$%.2f, max=$%.2f%n",
                currency, stats.getCount(), stats.getAverage(), stats.getMax())
        );

        // 4. Partition
        Map<Boolean, List<Transaction>> partitioned = transactions.stream()
            .collect(Collectors.partitioningBy(
                tx -> tx.amount().compareTo(new BigDecimal("500")) >= 0
            ));

        System.out.println("Large: " + partitioned.get(true).size());
        System.out.println("Small: " + partitioned.get(false).size());

        // 5. Complex grouping — monthly spending by category
        Map<String, Map<String, BigDecimal>> monthlySpending = transactions.stream()
            .filter(tx -> tx.status() == TransactionStatus.COMPLETED)
            .collect(Collectors.groupingBy(
                tx -> tx.date().getYear() + "-" +
                      String.format("%02d", tx.date().getMonthValue()),
                Collectors.groupingBy(
                    Transaction::category,
                    Collectors.reducing(
                        BigDecimal.ZERO,
                        Transaction::amount,
                        BigDecimal::add
                    )
                )
            ));

        monthlySpending.forEach((month, categories) -> {
            System.out.println(month + ":");
            categories.forEach((cat, total) ->
                System.out.printf("  %-12s $%10.2f%n", cat, total)
            );
        });

        // 6. FlatMap — flatten nested collections
        List<Account> accounts = generateAccounts();
        List<Transaction> allTransactions = accounts.stream()
            .flatMap(account -> account.transactions().stream())
            .sorted(Comparator.comparing(Transaction::date))
            .toList();

        System.out.println("Total transactions across all accounts: " + allTransactions.size());

        // 7. Custom collector — find the top N transactions
        List<Transaction> top5 = transactions.stream()
            .collect(Collectors.collectingAndThen(
                Collectors.toCollection(() ->
                    new TreeSet<>(Comparator.comparing(Transaction::amount).reversed())
                ),
                sorted -> sorted.stream().limit(5).toList()
            ));

        System.out.println("Top 5 transactions:");
        top5.forEach(tx ->
            System.out.printf("  %s: $%s%n", tx.id(), tx.amount())
        );

        // 8. Functional composition
        Predicate<Transaction> isCompleted = tx -> tx.status() == TransactionStatus.COMPLETED;
        Predicate<Transaction> isUsd = tx -> "USD".equals(tx.currency());
        Predicate<Transaction> isLarge = tx -> tx.amount().compareTo(new BigDecimal("1000")) > 0;

        Predicate<Transaction> targetFilter = isCompleted.and(isUsd).and(isLarge);

        long count = transactions.stream()
            .filter(targetFilter)
            .count();

        System.out.println("Completed USD transactions > $1000: " + count);
    }

    // Records for immutable data
    public record Transaction(
        String id,
        BigDecimal amount,
        String currency,
        LocalDate date,
        TransactionStatus status,
        String category,
        List<String> tags
    ) {}

    public record Account(
        String id,
        String owner,
        List<Transaction> transactions
    ) {}

    public enum TransactionStatus {
        PENDING, COMPLETED, FAILED, CANCELLED
    }

    private static List<Transaction> generateSampleData() {
        return List.of(
            new Transaction("TX-001", new BigDecimal("1500.00"), "USD",
                LocalDate.of(2024, 1, 15), TransactionStatus.COMPLETED, "SALARY", List.of("income")),
            new Transaction("TX-002", new BigDecimal("1200.00"), "USD",
                LocalDate.of(2024, 1, 1), TransactionStatus.COMPLETED, "RENT", List.of("housing", "recurring")),
            new Transaction("TX-003", new BigDecimal("45.50"), "USD",
                LocalDate.of(2024, 1, 20), TransactionStatus.COMPLETED, "FOOD", List.of("groceries")),
            new Transaction("TX-004", new BigDecimal("89.99"), "EUR",
                LocalDate.of(2024, 2, 5), TransactionStatus.COMPLETED, "UTILITIES", List.of("electric")),
            new Transaction("TX-005", new BigDecimal("3500.00"), "GBP",
                LocalDate.of(2024, 2, 1), TransactionStatus.COMPLETED, "SALARY", List.of("income")),
            new Transaction("TX-006", new BigDecimal("250.00"), "USD",
                LocalDate.of(2024, 2, 14), TransactionStatus.FAILED, "TRANSFER", List.of("outgoing")),
            new Transaction("TX-007", new BigDecimal("75.00"), "EUR",
                LocalDate.of(2024, 3, 10), TransactionStatus.PENDING, "FOOD", List.of("restaurant")),
            new Transaction("TX-008", new BigDecimal("5000.00"), "USD",
                LocalDate.of(2024, 3, 1), TransactionStatus.COMPLETED, "SALARY", List.of("income")),
            new Transaction("TX-009", new BigDecimal("320.00"), "GBP",
                LocalDate.of(2024, 3, 15), TransactionStatus.COMPLETED, "SHOPPING", List.of("electronics")),
            new Transaction("TX-010", new BigDecimal("15.00"), "USD",
                LocalDate.of(2024, 3, 20), TransactionStatus.CANCELLED, "SUBSCRIPTION", List.of("streaming"))
        );
    }

    private static List<Account> generateAccounts() {
        return List.of(
            new Account("ACC-001", "Alice", generateSampleData().subList(0, 4)),
            new Account("ACC-002", "Bob", generateSampleData().subList(4, 8))
        );
    }
}
```

**A real-world service method using streams:**

```java
@Service
public class ReportingService {

    private final TransactionRepository transactionRepository;

    public ReportingService(TransactionRepository transactionRepository) {
        this.transactionRepository = transactionRepository;
    }

    public MonthlyReport generateMonthlyReport(String accountId, int year, int month) {
        List<Transaction> transactions = transactionRepository
            .findByAccountIdAndDateBetween(
                accountId,
                LocalDate.of(year, month, 1),
                LocalDate.of(year, month, 1).plusMonths(1).minusDays(1)
            );

        BigDecimal totalIncome = transactions.stream()
            .filter(tx -> tx.getAmount().compareTo(BigDecimal.ZERO) > 0)
            .map(Transaction::getAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);

        BigDecimal totalExpenses = transactions.stream()
            .filter(tx -> tx.getAmount().compareTo(BigDecimal.ZERO) < 0)
            .map(Transaction::getAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add)
            .abs();

        Map<String, BigDecimal> expensesByCategory = transactions.stream()
            .filter(tx -> tx.getAmount().compareTo(BigDecimal.ZERO) < 0)
            .collect(Collectors.groupingBy(
                Transaction::getCategory,
                Collectors.reducing(
                    BigDecimal.ZERO,
                    Transaction::getAmount,
                    BigDecimal::add
                )
            ));

        Optional<Transaction> largestExpense = transactions.stream()
            .filter(tx -> tx.getAmount().compareTo(BigDecimal.ZERO) < 0)
            .min(Comparator.comparing(Transaction::getAmount));

        return new MonthlyReport(
            accountId, year, month,
            totalIncome, totalExpenses,
            totalIncome.subtract(totalExpenses),
            expensesByCategory,
            largestExpense.orElse(null)
        );
    }
}
```

---

## Important Notes

- Streams are lazy. Intermediate operations like `filter`, `map`, and `sorted` do not process any elements until a terminal operation like `collect`, `forEach`, or `count` is invoked. This allows the stream to fuse operations and short-circuit when possible (e.g., `findFirst` stops after finding the first match).
- Streams are single-use. Once a terminal operation completes, the stream is consumed and cannot be reused. Attempting to call a second terminal operation throws `IllegalStateException`. If you need to process the same data multiple times, create a new stream from the source.
- Stream operations should be side-effect-free. The functions passed to `filter`, `map`, `sorted`, and other intermediate operations should not modify external state. Side effects in parallel streams cause race conditions and non-deterministic behavior. The only acceptable side effect is in `forEach`, and even there, prefer `collect` for accumulation.
- `collect` is more powerful and safer than `forEach` for building result collections. `forEach` with a mutable accumulator (e.g., `list::add`) is not thread-safe in parallel streams. `collect` handles thread-safe accumulation internally and works correctly in both sequential and parallel modes.
- `Collectors.toMap()` throws `IllegalStateException` on duplicate keys. If your data may contain duplicates, provide a merge function as the third argument: `Collectors.toMap(keyMapper, valueMapper, (v1, v2) -> v1)`.
- `Collectors.groupingBy()` returns a `HashMap` by default, which has no guaranteed iteration order. If you need insertion order, specify `LinkedHashMap::new` as the map factory. If you need sorted order, specify `TreeMap::new`.
- `Optional` is designed to work seamlessly with streams. `Stream.flatMap()` can unwrap `Optional` values: `stream.map(this::findAccount).flatMap(Optional::stream)` (Java 9+). In Java 8, use `.filter(Optional::isPresent).map(Optional::get)`.
- Parallel streams use the common Fork/Join pool by default. If your parallel stream performs blocking I/O, it can starve other parallel streams and the entire application. For I/O-bound parallelism, use `CompletableFuture` with a custom executor instead of parallel streams.
- The `peek()` operation is intended for debugging, not for production logic. It is not guaranteed to be called for every element in all stream implementations (e.g., when the stream can determine the result without processing all elements). Do not use `peek()` for side effects that your application depends on.
- Primitive streams (`IntStream`, `LongStream`, `DoubleStream`) avoid the boxing overhead of `Stream<Integer>`, `Stream<Long>`, `Stream<Double>`. For numerical aggregation (sum, average, min, max), always use primitive streams. The difference is significant for large datasets.
- The `reduce` operation is the most general terminal operation. `sum`, `count`, `min`, `max`, and `collect` are all special cases of `reduce`. Use `reduce` when no specialized collector exists for your use case. The three-argument form (`reduce(identity, accumulator, combiner)`) is required for parallel streams to work correctly.
- Method references are preferred over lambdas when the lambda body is a single method call. `String::toUpperCase` is clearer than `s -> s.toUpperCase()`. However, if the method reference obscures the type or the logic is non-trivial, a lambda is more readable.
- The Stream API is not a replacement for all loops. Simple loops with early exit, complex control flow, or multiple exit conditions are often clearer as traditional for-loops. Use streams for declarative data processing pipelines. Use loops for imperative control flow.

---

## Practice

1. Given a `List<Transaction>`, write a stream pipeline that filters for completed transactions, groups them by currency, and calculates the total amount per currency. Return a `Map<String, BigDecimal>`.

2. Write a method that takes a `List<Account>` and returns a `Map<String, List<Transaction>>` mapping each account owner's name to their sorted list of transactions (sorted by date, then by amount descending). Use `flatMap` to flatten the nested transaction lists.

3. Write a stream pipeline that reads a CSV file of transactions using `Files.lines()`, parses each line into a `Transaction` object, filters out invalid lines, and collects the results into a `List<Transaction>`. Handle malformed lines gracefully using `flatMap` with `Optional`.

4. Implement a method `List<Transaction> getTopNByAmount(List<Transaction> transactions, int n)` using streams and collectors. Do not sort the entire list — use a `TreeSet` or `PriorityQueue` within a collector to maintain only the top N elements.

5. Write a stream pipeline that demonstrates the difference between `map` and `flatMap`. Create a `List<Order>` where each `Order` contains a `List<LineItem>`. Use `flatMap` to extract all line items into a single stream, then calculate the total revenue.

6. Benchmark a sequential stream vs a parallel stream for filtering and summing a list of 10 million `BigDecimal` values. Run each approach 10 times and compare the average execution time. Document when the parallel stream is faster and when it is slower.

7. In your Obsidian vault, create a reference card for the Stream API. Include: all intermediate operations with their signatures, all terminal operations with their return types, the most common `Collectors` methods, and a decision tree for choosing between `map`, `flatMap`, `filter`, and `reduce`.

---

## References

- java.util.function Package: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html
- java.util.stream Package: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html
- Stream API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html
- Collectors API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html
- JEP 126 (Lambda Expressions and Virtual Extension Methods): https://openjdk.org/jeps/126
- Oracle Tutorial — Lambda Expressions: https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html
- Oracle Tutorial — Aggregate Operations: https://docs.oracle.com/javase/tutorial/collections/streams/
- "Modern Java in Action" by Raoul-Gabriel Urma, Mario Fusco, and Alan Mycroft
- "Effective Java" by Joshua Bloch — Items 42-46 (Lambdas and Streams)
