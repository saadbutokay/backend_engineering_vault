## Overview

Generics allow you to write classes, interfaces, and methods that operate on types specified by the caller. Without generics, Java collections stored everything as `Object`, requiring manual casting and deferring type errors to runtime. With generics, the compiler enforces type safety at compile time, eliminating entire categories of `ClassCastException` bugs. Generics are used pervasively in the Java standard library (`List<T>`, `Map<K,V>`, `Optional<T>`, `CompletableFuture<T>`), in every Spring framework component, and in every enterprise codebase you will encounter. Understanding generics deeply — including their limitations due to type erasure — is essential for writing correct, reusable, and type-safe Java code.

---

## Core Concepts

### The Problem Generics Solve

Before generics (pre-Java 5), collections stored `Object` references. Every retrieval required an explicit cast, and type mismatches were not caught until runtime.

```java
// Pre-generics (Java 1.4 and earlier) — DO NOT WRITE CODE LIKE THIS
List accounts = new ArrayList();
accounts.add(new SavingsAccount("SAV-001"));
accounts.add(new CheckingAccount("CHK-001"));
accounts.add("This is not an account");  // Compiles! No type checking.

// Retrieval requires casting
SavingsAccount first = (SavingsAccount) accounts.get(0);  // Works
SavingsAccount third = (SavingsAccount) accounts.get(2);  // ClassCastException at runtime!
```

With generics, the compiler enforces the type constraint:

```java
// With generics (Java 5+)
List<Account> accounts = new ArrayList<>();
accounts.add(new SavingsAccount("SAV-001"));
accounts.add(new CheckingAccount("CHK-001"));
accounts.add("This is not an account");  // Compilation error! String is not an Account.

Account first = accounts.get(0);  // No cast needed. Compiler guarantees the type.
```

Generics move type checking from runtime to compile time. This is the single most important benefit. In a fintech system processing millions of transactions, a `ClassCastException` at 2 AM is not just an inconvenience — it is a production incident.

### Generic Classes

A generic class declares one or more type parameters in angle brackets after the class name.

```java
public class Repository<T> {
    private final List<T> entities = new ArrayList<>();

    public void save(T entity) {
        entities.add(entity);
    }

    public T findById(int index) {
        if (index < 0 || index >= entities.size()) {
            throw new IndexOutOfBoundsException("Invalid index: " + index);
        }
        return entities.get(index);
    }

    public List<T> findAll() {
        return Collections.unmodifiableList(entities);
    }

    public int count() {
        return entities.size();
    }
}
```

**Usage:**

```java
// Specify the type argument when creating an instance
Repository<Account> accountRepo = new Repository<>();
accountRepo.save(new SavingsAccount("SAV-001"));
accountRepo.save(new CheckingAccount("CHK-001"));

Account account = accountRepo.findById(0);  // No cast needed

Repository<Transaction> txRepo = new Repository<>();
txRepo.save(new Transaction("TX-001", new BigDecimal("100")));

Transaction tx = txRepo.findById(0);  // Correct type guaranteed
```

**Multiple type parameters:**

```java
public class Pair<K, V> {
    private final K key;
    private final V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }

    @Override
    public String toString() {
        return "(" + key + ", " + value + ")";
    }
}

// Usage
Pair<String, BigDecimal> exchangeRate = new Pair<>("EUR", new BigDecimal("0.92"));
Pair<Long, Account> indexedAccount = new Pair<>(1L, new SavingsAccount("SAV-001"));
```

**A more realistic generic class — a type-safe result wrapper:**

```java
public class Result<T> {
    private final T data;
    private final String error;
    private final boolean success;

    private Result(T data, String error, boolean success) {
        this.data = data;
        this.error = error;
        this.success = success;
    }

    public static <T> Result<T> success(T data) {
        return new Result<>(data, null, true);
    }

    public static <T> Result<T> failure(String error) {
        return new Result<>(null, error, false);
    }

    public boolean isSuccess() { return success; }
    public T getData() {
        if (!success) throw new IllegalStateException("Result is a failure: " + error);
        return data;
    }
    public String getError() {
        if (success) throw new IllegalStateException("Result is a success");
        return error;
    }
}

// Usage
Result<Account> result = accountService.findById("ACC-001");
if (result.isSuccess()) {
    Account account = result.getData();  // Type-safe
} else {
    log.error("Failed: {}", result.getError());
}
```

### Generic Methods

A method can declare its own type parameters, independent of the class's type parameters. The type parameter is declared before the return type.

```java
public class CollectionUtils {

    // Generic method — T is determined by the argument at the call site
    public static <T> void printAll(List<T> items) {
        for (T item : items) {
            System.out.println(item);
        }
    }

    // Generic method with return type
    public static <T> T getFirst(List<T> items) {
        if (items.isEmpty()) {
            throw new NoSuchElementException("List is empty");
        }
        return items.get(0);
    }

    // Generic method with multiple type parameters
    public static <K, V> Map<K, List<V>> groupBy(List<V> items, Function<V, K> keyExtractor) {
        Map<K, List<V>> groups = new HashMap<>();
        for (V item : items) {
            K key = keyExtractor.apply(item);
            groups.computeIfAbsent(key, k -> new ArrayList<>()).add(item);
        }
        return groups;
    }
}

// Usage — type is inferred from arguments
List<String> names = List.of("Alice", "Bob", "Charlie");
CollectionUtils.printAll(names);  // T inferred as String

Account first = CollectionUtils.getFirst(accounts);  // T inferred as Account

// Explicit type argument (rarely needed)
Account explicit = CollectionUtils.<Account>getFirst(accounts);
```

**Generic methods in non-generic classes:**

```java
public class PaymentService {
    // This class is not generic, but this method is
    public <T extends Notification> void sendNotification(T notification) {
        notification.validate();
        notification.send();
        auditLog.record(notification);
    }
}
```

### Generic Interfaces

Interfaces can declare type parameters just like classes.

```java
public interface Repository<T, ID> {
    T findById(ID id);
    List<T> findAll();
    T save(T entity);
    void deleteById(ID id);
    boolean existsById(ID id);
    long count();
}

// Implementation specifies the type arguments
public class AccountRepository implements Repository<Account, String> {
    private final Map<String, Account> store = new HashMap<>();

    @Override
    public Account findById(String id) {
        return store.get(id);
    }

    @Override
    public List<Account> findAll() {
        return new ArrayList<>(store.values());
    }

    @Override
    public Account save(Account entity) {
        store.put(entity.getAccountId(), entity);
        return entity;
    }

    @Override
    public void deleteById(String id) {
        store.remove(id);
    }

    @Override
    public boolean existsById(String id) {
        return store.containsKey(id);
    }

    @Override
    public long count() {
        return store.size();
    }
}

// Another implementation with different type arguments
public class TransactionRepository implements Repository<Transaction, Long> {
    // ...
}
```

This is exactly the pattern used by Spring Data JPA. When you write `interface AccountRepository extends JpaRepository<Account, String>`, you are using generic interfaces.

### Bounded Type Parameters

Bounded type parameters restrict the types that can be used as type arguments. They ensure that the type argument has certain capabilities.

**Upper bound (`extends`):**

The type argument must be a subtype of the specified bound. `extends` in generics means "extends or implements."

```java
// T must be a Number or a subclass of Number (Integer, Double, BigDecimal, etc.)
public class StatisticsCalculator<T extends Number> {
    private final List<T> values = new ArrayList<>();

    public void add(T value) {
        values.add(value);
    }

    public double calculateAverage() {
        if (values.isEmpty()) return 0.0;
        double sum = 0;
        for (T value : values) {
            sum += value.doubleValue();  // Safe — Number has doubleValue()
        }
        return sum / values.size();
    }
}

// Valid
StatisticsCalculator<Integer> intCalc = new StatisticsCalculator<>();
StatisticsCalculator<Double> doubleCalc = new StatisticsCalculator<>();
StatisticsCalculator<BigDecimal> bigDecCalc = new StatisticsCalculator<>();

// Invalid — compilation error
StatisticsCalculator<String> stringCalc = new StatisticsCalculator<>();  // String is not a Number
```

**Upper bound with an interface:**

```java
// T must implement Comparable
public class SortedList<T extends Comparable<T>> {
    private final List<T> items = new ArrayList<>();

    public void add(T item) {
        items.add(item);
        Collections.sort(items);  // Safe — T is Comparable
    }

    public T getMin() {
        return items.isEmpty() ? null : items.get(0);
    }

    public T getMax() {
        return items.isEmpty() ? null : items.get(items.size() - 1);
    }
}

// Valid — String implements Comparable<String>
SortedList<String> names = new SortedList<>();

// Valid — LocalDate implements Comparable<LocalDate>
SortedList<LocalDate> dates = new SortedList<>();
```

**Multiple bounds:**

A type parameter can have multiple bounds. If one bound is a class, it must come first.

```java
// T must extend Account AND implement Auditable AND implement Serializable
public class SecureAccountService<T extends Account & Auditable & Serializable> {
    public void processAndAudit(T account) {
        account.deposit(new BigDecimal("100"));  // From Account
        account.recordAudit("DEPOSIT", "system");  // From Auditable
        serialize(account);  // From Serializable
    }

    private void serialize(Serializable obj) {
        // ...
    }
}
```

### Wildcards

Wildcards (`?`) represent unknown types. They are used in method parameters and variable declarations when you want to accept a range of types without knowing the exact type.

**Unbounded wildcard (`?`):**

Accepts any type. Equivalent to `? extends Object`.

```java
public void printSize(List<?> list) {
    System.out.println("Size: " + list.size());
    // You can read elements as Object
    for (Object item : list) {
        System.out.println(item);
    }
    // You CANNOT add elements (except null)
    // list.add("hello");  // Compilation error!
}

printSize(List.of("a", "b", "c"));     // List<String>
printSize(List.of(1, 2, 3));           // List<Integer>
printSize(List.of(new Account()));     // List<Account>
```

**Upper bounded wildcard (`? extends T`):**

Accepts T or any subtype of T. You can read elements as T, but you cannot add elements (because the actual type is unknown).

```java
// Accepts List<Number>, List<Integer>, List<Double>, List<BigDecimal>, etc.
public BigDecimal sumAll(List<? extends Number> numbers) {
    BigDecimal sum = BigDecimal.ZERO;
    for (Number number : numbers) {  // Safe to read as Number
        sum = sum.add(new BigDecimal(number.toString()));
    }
    return sum;
}

List<Integer> ints = List.of(1, 2, 3);
List<Double> doubles = List.of(1.5, 2.5, 3.5);
List<BigDecimal> bigDecimals = List.of(new BigDecimal("100"));

sumAll(ints);         // Valid
sumAll(doubles);      // Valid
sumAll(bigDecimals);  // Valid
```

**Why you cannot add to `? extends T`:**

```java
List<? extends Number> numbers = new ArrayList<Integer>();

// numbers.add(3.14);   // Error — what if the list is actually List<Integer>?
// numbers.add(42);     // Error — what if the list is actually List<Double>?
// numbers.add(null);   // This is the only thing you can add (null is every type)

// The compiler does not know the actual type, so it prevents all additions
// to guarantee type safety.
```

**Lower bounded wildcard (`? super T`):**

Accepts T or any supertype of T. You can add elements of type T, but you can only read elements as `Object`.

```java
// Accepts List<Number>, List<Object>, List<Serializable>, etc.
public void addIntegers(List<? super Integer> list) {
    list.add(1);      // Safe — Integer is a subtype of whatever the list holds
    list.add(2);
    list.add(3);

    // Reading returns Object, not Integer
    Object first = list.get(0);  // Must read as Object
    // Integer first = list.get(0);  // Compilation error!
}

List<Integer> intList = new ArrayList<>();
List<Number> numList = new ArrayList<>();
List<Object> objList = new ArrayList<>();

addIntegers(intList);  // Valid
addIntegers(numList);  // Valid
addIntegers(objList);  // Valid
```

**Why you can only read as `Object` from `? super T`:**

```java
List<? super Integer> list = new ArrayList<Number>();
list.add(42);  // Fine — Integer IS-A Number

Object item = list.get(0);  // Must be Object
// The actual list could be List<Number>, List<Object>, List<Integer>, etc.
// The only type guaranteed to be a supertype of all possibilities is Object.
```

### The PECS Principle

**PECS** stands for **Producer Extends, Consumer Super**. This is the most important rule for using wildcards correctly.

- If a parameter **produces** values (you read from it), use `? extends T`.
- If a parameter **consumes** values (you write to it), use `? super T`.
- If a parameter both produces and consumes, use the exact type `T` (no wildcard).

```java
public class CollectionUtils {

    // "source" is a producer — we read from it → extends
    // "destination" is a consumer — we write to it → super
    public static <T> void copy(List<? extends T> source, List<? super T> destination) {
        for (T item : source) {        // Read from source (producer)
            destination.add(item);      // Write to destination (consumer)
        }
    }
}

List<Integer> source = List.of(1, 2, 3);
List<Number> destination = new ArrayList<>();
CollectionUtils.copy(source, destination);
// source produces Integers (extends Number)
// destination consumes Numbers (super Integer)
```

**Real-world example — processing transactions:**

```java
public class TransactionProcessor {

    // Producer: we read transactions from the source
    public BigDecimal calculateTotal(List<? extends Transaction> transactions) {
        BigDecimal total = BigDecimal.ZERO;
        for (Transaction tx : transactions) {
            total = total.add(tx.getAmount());
        }
        return total;
    }

    // Consumer: we write validated transactions to the destination
    public void validateAndStore(
            List<Transaction> pending,
            List<? super ValidatedTransaction> validated) {
        for (Transaction tx : pending) {
            if (tx.getAmount().compareTo(BigDecimal.ZERO) > 0) {
                validated.add(new ValidatedTransaction(tx));
            }
        }
    }

    // Both producer and consumer: exact type, no wildcard
    public void sortTransactions(List<Transaction> transactions) {
        transactions.sort(Comparator.comparing(Transaction::getTimestamp));
        // We read AND write to the same list
    }
}
```

### Type Erasure

Type erasure is the mechanism by which the Java compiler removes all generic type information at compile time. The JVM never sees generic types. This is a fundamental design decision made for backward compatibility with pre-Java 5 code.

**What erasure does:**

```java
// What you write:
List<String> names = new ArrayList<>();
names.add("Alice");
String first = names.get(0);

// What the compiler generates (approximately):
List names = new ArrayList();
names.add("Alice");
String first = (String) names.get(0);  // Compiler inserts the cast
```

**Erasure rules:**

1. Unbounded type parameters (`<T>`) are replaced with `Object`.
2. Bounded type parameters (`<T extends Number>`) are replaced with the first bound (`Number`).
3. The compiler inserts casts where necessary to preserve type safety.
4. Bridge methods are generated to preserve polymorphism.

**Consequences of type erasure:**

```java
// 1. You cannot instantiate a generic type
public <T> void create() {
    T obj = new T();          // Compilation error!
    T[] arr = new T[10];      // Compilation error!
}

// 2. You cannot use instanceof with generic types
if (obj instanceof List<String>) {  // Compilation error!
    // ...
}
if (obj instanceof List<?>) {       // This works — unbounded wildcard
    // ...
}

// 3. You cannot use primitive types as type arguments
List<int> numbers = new ArrayList<>();  // Compilation error!
List<Integer> numbers = new ArrayList<>();  // Correct

// 4. Generic type information is lost at runtime
List<String> strings = new ArrayList<>();
List<Integer> integers = new ArrayList<>();
System.out.println(strings.getClass() == integers.getClass());  // true!
// Both are ArrayList.class at runtime. The <String> and <Integer> are erased.

// 5. You cannot create arrays of generic types
List<String>[] arrayOfLists = new List<String>[10];  // Compilation error!

// 6. Static fields cannot use the class type parameter
public class Repository<T> {
    private static T defaultEntity;  // Compilation error!
    // Static members belong to the class, not to a specific type instantiation.
    // Repository<Account> and Repository<Transaction> share the same static field.
}
```

**Bridge methods:**

The compiler generates bridge methods to preserve polymorphism after erasure. You rarely interact with these directly, but they explain certain reflection behaviors.

```java
public interface Comparable<T> {
    int compareTo(T o);
}

public class Money implements Comparable<Money> {
    @Override
    public int compareTo(Money other) {
        return this.amount.compareTo(other.amount);
    }
}

// After erasure, the interface method becomes compareTo(Object).
// The compiler generates a bridge method:
// public int compareTo(Object o) {
//     return compareTo((Money) o);  // Delegates to the typed method
// }
```

### Generic Methods with Varargs

Combining generics and varargs produces a compiler warning about heap pollution. The `@SafeVarargs` annotation suppresses this warning when the method is safe.

```java
// Compiler warning: "Possible heap pollution from parameterized vararg type"
public static <T> List<T> listOf(T... elements) {
    return new ArrayList<>(Arrays.asList(elements));
}

// Suppress the warning when you guarantee safety
@SafeVarargs
public static <T> List<T> safeList(T... elements) {
    List<T> list = new ArrayList<>();
    for (T element : elements) {
        list.add(element);
    }
    return list;
}

// @SafeVarargs can only be applied to:
// - static methods
// - final instance methods
// - private instance methods (Java 9+)
// - constructors (Java 9+)
```

**Heap pollution** occurs when a variable of a parameterized type refers to an object that is not of that type. With varargs, the compiler creates an array of the generic type, which is unsafe due to type erasure.

### Recursive Type Bounds

A type parameter can be bounded by a type that itself uses the type parameter. This pattern is common in the `Comparable` interface and in builder patterns.

```java
// T must implement Comparable<T> — the type is comparable to itself
public static <T extends Comparable<T>> T findMax(List<T> items) {
    if (items.isEmpty()) throw new NoSuchElementException();
    T max = items.get(0);
    for (T item : items) {
        if (item.compareTo(max) > 0) {
            max = item;
        }
    }
    return max;
}

// Usage
List<LocalDate> dates = List.of(
    LocalDate.of(2024, 1, 1),
    LocalDate.of(2024, 6, 15),
    LocalDate.of(2024, 3, 10)
);
LocalDate latest = findMax(dates);  // 2024-06-15
```

**Recursive bounds in the builder pattern:**

```java
public abstract class AccountBuilder<T extends AccountBuilder<T>> {
    protected String owner;
    protected BigDecimal balance;

    @SuppressWarnings("unchecked")
    public T withOwner(String owner) {
        this.owner = owner;
        return (T) this;  // Returns the concrete subclass type
    }

    @SuppressWarnings("unchecked")
    public T withBalance(BigDecimal balance) {
        this.balance = balance;
        return (T) this;
    }

    public abstract Account build();
}

public class SavingsAccountBuilder extends AccountBuilder<SavingsAccountBuilder> {
    private BigDecimal interestRate;

    public SavingsAccountBuilder withInterestRate(BigDecimal rate) {
        this.interestRate = rate;
        return this;
    }

    @Override
    public Account build() {
        return new SavingsAccount(owner, balance, interestRate);
    }
}

// Usage — fluent API with correct return types
Account account = new SavingsAccountBuilder()
    .withOwner("Alice")          // Returns SavingsAccountBuilder, not AccountBuilder
    .withBalance(new BigDecimal("5000"))
    .withInterestRate(new BigDecimal("0.04"))  // This works because of recursive bound
    .build();
```

### The Diamond Operator

The diamond operator (`<>`) allows the compiler to infer type arguments from the context. Introduced in Java 7 and enhanced in Java 9.

```java
// Before Java 7 — verbose
Map<String, List<Transaction>> transactions = new HashMap<String, List<Transaction>>();

// Java 7+ — diamond operator
Map<String, List<Transaction>> transactions = new HashMap<>();

// Java 9+ — diamond with anonymous classes
Comparator<Transaction> byAmount = new Comparator<>() {
    @Override
    public int compare(Transaction t1, Transaction t2) {
        return t1.getAmount().compareTo(t2.getAmount());
    }
};
```

The diamond operator does not change the type. It simply tells the compiler to infer the type arguments from the left-hand side of the assignment. Always use it. Writing out the full type on the right-hand side is unnecessary noise.

---

## Code Examples

**A complete generic repository pattern (the foundation of Spring Data):**

```java
package com.example.generics;

import java.util.*;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.stream.Collectors;

// Generic interface
public interface CrudRepository<T, ID> {
    T save(T entity);
    Optional<T> findById(ID id);
    List<T> findAll();
    void deleteById(ID id);
    long count();
    boolean existsById(ID id);
}

// Generic abstract implementation
public abstract class InMemoryRepository<T, ID> implements CrudRepository<T, ID> {
    private final Map<ID, T> store = new LinkedHashMap<>();

    protected abstract ID getId(T entity);

    @Override
    public T save(T entity) {
        store.put(getId(entity), entity);
        return entity;
    }

    @Override
    public Optional<T> findById(ID id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public List<T> findAll() {
        return new ArrayList<>(store.values());
    }

    @Override
    public void deleteById(ID id) {
        store.remove(id);
    }

    @Override
    public long count() {
        return store.size();
    }

    @Override
    public boolean existsById(ID id) {
        return store.containsKey(id);
    }

    // Generic query method using bounded wildcard
    public List<T> findBy(Predicate<T> condition) {
        return store.values().stream()
            .filter(condition)
            .collect(Collectors.toList());
    }

    // Generic projection method
    public <R> List<R> project(Function<T, R> projection) {
        return store.values().stream()
            .map(projection)
            .collect(Collectors.toList());
    }
}

// Concrete implementation
public class AccountRepository extends InMemoryRepository<Account, String> {
    @Override
    protected String getId(Account entity) {
        return entity.getAccountId();
    }
}

// Demonstration
public class GenericDemo {
    public static void main(String[] args) {
        AccountRepository repo = new AccountRepository();
        repo.save(new Account("ACC-001", "Alice", new BigDecimal("5000")));
        repo.save(new Account("ACC-002", "Bob", new BigDecimal("1200")));
        repo.save(new Account("ACC-003", "Charlie", new BigDecimal("50000")));

        // Type-safe retrieval
        repo.findById("ACC-001").ifPresent(acc ->
            System.out.println("Found: " + acc.getOwnerName())
        );

        // Generic query with Predicate
        List<Account> wealthy = repo.findBy(
            acc -> acc.getBalance().compareTo(new BigDecimal("10000")) > 0
        );
        System.out.println("Wealthy accounts: " + wealthy.size());

        // Generic projection with Function
        List<String> names = repo.project(Account::getOwnerName);
        System.out.println("All names: " + names);

        // Wildcard usage
        List<? extends Account> readOnly = repo.findAll();
        printBalances(readOnly);  // Producer — use extends
    }

    // PECS: source is a producer
    public static void printBalances(List<? extends Account> accounts) {
        for (Account acc : accounts) {
            System.out.printf("%s: $%s%n", acc.getOwnerName(), acc.getBalance());
        }
    }
}
```

**A generic event bus demonstrating bounded wildcards:**

```java
public class EventBus {
    private final Map<Class<?>, List<Consumer<?>>> listeners = new HashMap<>();

    // Consumer: we write events to the listener — use super
    public <T> void subscribe(Class<T> eventType, Consumer<? super T> listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(listener);
    }

    // Producer: we read events from the bus — use extends
    @SuppressWarnings("unchecked")
    public <T> void publish(T event) {
        List<Consumer<?>> eventListeners = listeners.get(event.getClass());
        if (eventListeners != null) {
            for (Consumer<?> listener : eventListeners) {
                ((Consumer<T>) listener).accept(event);
            }
        }
    }
}

// Usage
EventBus bus = new EventBus();

// Subscribe with a consumer that accepts PaymentEvent or any supertype
bus.subscribe(PaymentCompleted.class, (Consumer<DomainEvent>) event ->
    System.out.println("Audit: " + event)
);

bus.publish(new PaymentCompleted("TX-001", new BigDecimal("500")));
```

---

## Important Notes

- Generics are a compile-time feature. The JVM has no knowledge of generic types at runtime. All generic type information is erased during compilation. This is the single most important fact about Java generics and the source of every limitation you will encounter.
- Because of type erasure, you cannot instantiate generic types (`new T()`), create generic arrays (`new T[10]`), use `instanceof` with parameterized types (`obj instanceof List<String>`), or use primitives as type arguments (`List<int>`). These are not oversights — they are direct consequences of erasure.
- The PECS principle (Producer Extends, Consumer Super) is the definitive guide for wildcard usage. When in doubt, ask: am I reading from this parameter (producer → `? extends T`) or writing to it (consumer → `? super T`)? If both, use the exact type with no wildcard.
- `List<String>` is NOT a subtype of `List<Object>`. This is called **invariance**. If `List<String>` were a subtype of `List<Object>`, you could add an `Integer` to a `List<String>` through the `List<Object>` reference, breaking type safety. Wildcards (`? extends`, `? super`) provide controlled flexibility while maintaining type safety.
- `extends` in generic bounds means "extends or implements." `<T extends Comparable<T>>` includes classes that implement the `Comparable` interface, not just classes that extend a superclass. This is a common source of confusion because `extends` has a different meaning in class declarations.
- Always use the diamond operator (`<>`) when the type can be inferred from context. Writing `new ArrayList<String>()` when `List<String> list = new ArrayList<>()` suffices is unnecessary verbosity.
- `@SafeVarargs` should be used only when you have verified that the method does not store the varargs array in a way that could cause heap pollution. If the method only reads from the varargs array and does not assign it to a variable of a less specific type, it is safe.
- Recursive type bounds (`<T extends Comparable<T>>`) are a powerful but advanced pattern. They ensure that a type is comparable to itself. You will encounter them in sorting utilities, builder patterns, and the `Enum` class definition (`public abstract class Enum<E extends Enum<E>>`).
- Generic type parameters cannot be used in static contexts. A static field or static method of a generic class cannot reference the class's type parameter because static members are shared across all type instantiations. The static method must declare its own type parameter if it needs one.
- When designing APIs, prefer interfaces with generics (`List<T>`, `Map<K,V>`) over concrete implementations (`ArrayList<T>`, `HashMap<K,V>`). This allows callers to choose their implementation and makes your API more flexible.
- Spring Data JPA's `JpaRepository<T, ID>` is the most prominent example of generic interfaces in enterprise Java. When you write `interface UserRepository extends JpaRepository<User, Long>`, Spring generates a complete CRUD implementation at runtime using the type arguments you specified. Understanding generics is essential for understanding how Spring Data works.
- Type erasure means that `List<String>` and `List<Integer>` are the same class at runtime (`List.class`). This is why you cannot overload methods that differ only in generic type: `void process(List<String> items)` and `void process(List<Integer> items)` have the same erasure and cannot coexist in the same class.

---

## Practice

1. Create a generic `Stack<T>` class backed by an `ArrayList<T>`. Implement `push(T)`, `pop()`, `peek()`, `isEmpty()`, and `size()`. Ensure that `pop()` and `peek()` throw `NoSuchElementException` when the stack is empty. Write tests with `Stack<String>`, `Stack<Integer>`, and `Stack<Transaction>`.

2. Create a generic `Pair<K, V>` class with `getKey()`, `getValue()`, and a static factory method `of(K, V)`. Override `equals()`, `hashCode()`, and `toString()`. Then rewrite it as a record and compare the two implementations.

3. Write a generic method `<T extends Comparable<T>> T findSecondLargest(List<T> items)` that returns the second-largest element in a list. Handle edge cases: empty list, single-element list, all elements equal. Test with `List<Integer>`, `List<String>`, and `List<LocalDate>`.

4. Write a method that demonstrates PECS. The method should take a source list (`? extends Number`) and a destination list (`? super Number`) and copy all elements from source to destination. Test with `List<Integer>` as source and `List<Object>` as destination.

5. Demonstrate type erasure by creating a `List<String>` and a `List<Integer>`, then using reflection to show that their runtime classes are identical. Attempt to add an `Integer` to the `List<String>` via reflection and observe that it succeeds (bypassing compile-time checks).

6. Create a generic `Result<T, E>` class that represents either a success value of type `T` or an error value of type `E`. Implement `isSuccess()`, `isError()`, `getSuccess()`, `getError()`, `map()`, and `flatMap()` methods. This is a simplified version of the `Either` type from functional programming.

7. In your Obsidian vault, create a reference card for generics. Include: syntax for generic classes, methods, and interfaces; the three wildcard types with examples; the PECS principle; and a list of things you cannot do due to type erasure.

---

## References

- Java Language Specification — Chapter 4.4 (Type Variables): https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html#jls-4.4
- Java Language Specification — Chapter 8.1.2 (Generic Classes): https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.1.2
- Java Tutorial — Generics: https://docs.oracle.com/javase/tutorial/java/generics/
- "Effective Java" by Joshua Bloch — Items 26-33 (Generics)
- "Java Generics and Collections" by Maurice Naftalin and Philip Wadler
- Angelika Langer's Java Generics FAQ: https://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html
