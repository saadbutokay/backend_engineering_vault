## Overview

Reflection is Java's mechanism for inspecting and manipulating classes, methods, fields, and constructors at runtime. It allows a program to examine its own structure and modify its behavior dynamically, without knowing the types at compile time. Reflection is the engine that powers every major Java framework. When Spring scans your classes for `@RestController`, when Hibernate reads your `@Entity` annotations to generate SQL, when Jackson maps JSON fields to private Java fields, and when JUnit discovers `@Test` methods — they are all using reflection. As a backend engineer, you will rarely write reflection code directly in application logic. However, understanding how reflection works is essential for debugging framework behavior, writing custom annotations, building libraries, and diagnosing the performance and security implications of reflective access.

---

## Core Concepts

### What Reflection Is

Reflection is the ability of a running program to:

1. **Discover** classes, interfaces, fields, methods, constructors, and annotations at runtime.
2. **Instantiate** objects of classes that were not known at compile time.
3. **Invoke** methods dynamically by name.
4. **Access and modify** fields, including private ones.
5. **Read** annotations attached to any program element.

Without reflection, a Java program is statically bound. The compiler determines which methods are called and which fields are accessed. With reflection, these decisions can be deferred to runtime, enabling the dynamic behavior that frameworks require.

**The tradeoff:**

| Aspect | Without Reflection | With Reflection |
|--------|-------------------|-----------------|
| Type safety | Compile-time checked | Runtime only — `ClassCastException` possible |
| Performance | Fast (direct invocation) | Slow (10-100x slower than direct calls) |
| Readability | Clear and explicit | Verbose and opaque |
| Refactoring | IDE can rename safely | String-based names break silently on rename |
| Security | Respects access modifiers | Can bypass `private` with `setAccessible(true)` |
| Flexibility | Fixed at compile time | Dynamic at runtime |

### The `Class` Object

Every type in Java — classes, interfaces, enums, records, arrays, primitives — has a corresponding `Class<?>` object that represents it at runtime. The `Class` object is the entry point for all reflection operations.

**Obtaining a `Class` object:**

```java
// Method 1: .class literal (compile-time, preferred when the type is known)
Class<String> stringClass = String.class;
Class<int[]> intArrayClass = int[].class;
Class<Void> voidClass = void.class;

// Method 2: .getClass() on an instance (runtime, when you have an object)
String name = "Alice";
Class<? extends String> runtimeClass = name.getClass();

Object obj = new SavingsAccount("SAV-001");
Class<?> objClass = obj.getClass();  // Returns SavingsAccount.class, not Account.class

// Method 3: Class.forName() (runtime, from a string — used by frameworks)
Class<?> clazz = Class.forName("com.example.fintech.model.Account");
// Throws ClassNotFoundException if the class does not exist on the classpath

// Method 4: ClassLoader (advanced, used by frameworks and plugin systems)
ClassLoader loader = Thread.currentThread().getContextClassLoader();
Class<?> loaded = loader.loadClass("com.example.fintech.model.Account");
```

**Key `Class` methods for inspection:**

```java
Class<?> clazz = Account.class;

// Identity
clazz.getName();          // "com.example.fintech.model.Account"
clazz.getSimpleName();    // "Account"
clazz.getPackageName();   // "com.example.fintech.model"
clazz.getCanonicalName(); // "com.example.fintech.model.Account"

// Type checks
clazz.isInterface();      // false
clazz.isEnum();           // false
clazz.isRecord();         // false
clazz.isArray();          // false
clazz.isPrimitive();      // false
clazz.isAnnotation();     // false
clazz.isSealed();         // false (Java 17+)
clazz.isMemberClass();    // false (true for inner classes)
clazz.isLocalClass();     // false
clazz.isAnonymousClass(); // false

// Hierarchy
clazz.getSuperclass();    // Account.class.getSuperclass() → Object.class
clazz.getInterfaces();    // [Auditable.class, Serializable.class]

// Modifiers
import java.lang.reflect.Modifier;
Modifier.isPublic(clazz.getModifiers());    // true
Modifier.isAbstract(clazz.getModifiers());  // false
Modifier.isFinal(clazz.getModifiers());     // false
```

### Inspecting Constructors

```java
Class<?> clazz = Account.class;

// Get all public constructors
Constructor<?>[] publicConstructors = clazz.getConstructors();

// Get ALL constructors (including private)
Constructor<?>[] allConstructors = clazz.getDeclaredConstructors();

// Get a specific constructor by parameter types
Constructor<Account> constructor = clazz.getConstructor(String.class, BigDecimal.class);

// Inspect constructor details
for (Constructor<?> c : allConstructors) {
    System.out.println("Constructor: " + c.getName());
    System.out.println("  Parameters: " + Arrays.toString(c.getParameterTypes()));
    System.out.println("  Modifiers: " + Modifier.toString(c.getModifiers()));
    System.out.println("  Exceptions: " + Arrays.toString(c.getExceptionTypes()));
}

// Create an instance via reflection
Constructor<Account> ctor = clazz.getConstructor(String.class, BigDecimal.class);
Account account = ctor.newInstance("ACC-001", new BigDecimal("5000"));
// Equivalent to: new Account("ACC-001", new BigDecimal("5000"))
```

### Inspecting Fields

```java
Class<?> clazz = Account.class;

// Get all public fields (including inherited)
Field[] publicFields = clazz.getFields();

// Get ALL declared fields (including private, NOT inherited)
Field[] allFields = clazz.getDeclaredFields();

// Get a specific field by name
Field balanceField = clazz.getDeclaredField("balance");

// Inspect field details
for (Field field : allFields) {
    System.out.println("Field: " + field.getName());
    System.out.println("  Type: " + field.getType().getSimpleName());
    System.out.println("  Generic type: " + field.getGenericType());
    System.out.println("  Modifiers: " + Modifier.toString(field.getModifiers()));
    System.out.println("  Is final: " + Modifier.isFinal(field.getModifiers()));
    System.out.println("  Is static: " + Modifier.isStatic(field.getModifiers()));
}

// Read a field value from an instance
Account account = new Account("ACC-001", "Alice", new BigDecimal("5000"));
Field balanceField = clazz.getDeclaredField("balance");
balanceField.setAccessible(true);  // Bypass private access
BigDecimal balance = (BigDecimal) balanceField.get(account);
System.out.println("Balance: " + balance);  // 5000

// Write a field value
balanceField.set(account, new BigDecimal("6000"));
System.out.println("New balance: " + account.getBalance());  // 6000

// Read a static field
Field countField = Account.class.getDeclaredField("accountCount");
countField.setAccessible(true);
int count = (int) countField.get(null);  // null for static fields
```

### Inspecting Methods

```java
Class<?> clazz = Account.class;

// Get all public methods (including inherited from Object)
Method[] publicMethods = clazz.getMethods();

// Get ALL declared methods (including private, NOT inherited)
Method[] allMethods = clazz.getDeclaredMethods();

// Get a specific method by name and parameter types
Method depositMethod = clazz.getMethod("deposit", BigDecimal.class);
Method getBalanceMethod = clazz.getMethod("getBalance");

// Inspect method details
for (Method method : allMethods) {
    System.out.println("Method: " + method.getName());
    System.out.println("  Return type: " + method.getReturnType().getSimpleName());
    System.out.println("  Parameters: " + Arrays.toString(method.getParameterTypes()));
    System.out.println("  Exceptions: " + Arrays.toString(method.getExceptionTypes()));
    System.out.println("  Modifiers: " + Modifier.toString(method.getModifiers()));
    System.out.println("  Is default: " + method.isDefault());
    System.out.println("  Is bridge: " + method.isBridge());
    System.out.println("  Is synthetic: " + method.isSynthetic());
}

// Invoke a method on an instance
Account account = new Account("ACC-001", "Alice", new BigDecimal("5000"));

// Invoke a method with no parameters
Method getBalance = clazz.getMethod("getBalance");
BigDecimal balance = (BigDecimal) getBalance.invoke(account);
// Equivalent to: account.getBalance()

// Invoke a method with parameters
Method deposit = clazz.getMethod("deposit", BigDecimal.class);
deposit.invoke(account, new BigDecimal("1000"));
// Equivalent to: account.deposit(new BigDecimal("1000"))

// Invoke a private method
Method validate = clazz.getDeclaredMethod("validateAmount", BigDecimal.class);
validate.setAccessible(true);  // Bypass private access
validate.invoke(account, new BigDecimal("500"));

// Invoke a static method
Method createDefault = clazz.getMethod("createDefault");
Account defaultAccount = (Account) createDefault.invoke(null);
// null for static methods (no instance needed)
```

### Inspecting Annotations

This connects directly to the previous section (1.13 Annotations). Reflection is the mechanism by which annotations are read at runtime.

```java
Class<?> clazz = Account.class;

// Class-level annotations
if (clazz.isAnnotationPresent(Entity.class)) {
    Entity entity = clazz.getAnnotation(Entity.class);
    System.out.println("Table: " + entity.name());
}

// All annotations on the class
Annotation[] annotations = clazz.getAnnotations();

// Method-level annotations
for (Method method : clazz.getDeclaredMethods()) {
    if (method.isAnnotationPresent(Audited.class)) {
        Audited audited = method.getAnnotation(Audited.class);
        System.out.println(method.getName() + " is audited: " + audited.action());
    }
}

// Field-level annotations
for (Field field : clazz.getDeclaredFields()) {
    if (field.isAnnotationPresent(Column.class)) {
        Column column = field.getAnnotation(Column.class);
        System.out.println(field.getName() + " → column: " + column.name());
    }
}

// Check for inherited annotations
Annotation[] inherited = clazz.getAnnotationsByType(Transactional.class);
```

### Access Control and `setAccessible(true)`

By default, reflection respects Java's access modifiers. Attempting to read a `private` field or invoke a `private` method throws `IllegalAccessException`. The `setAccessible(true)` method overrides this restriction.

```java
Field privateField = Account.class.getDeclaredField("balance");

// Without setAccessible — throws IllegalAccessException
// BigDecimal balance = (BigDecimal) privateField.get(account);

// With setAccessible — succeeds
privateField.setAccessible(true);
BigDecimal balance = (BigDecimal) privateField.get(account);
```

**Security implications:**

- `setAccessible(true)` bypasses the encapsulation that the class author intended. It can read and modify private state, break invariants, and access internal implementation details.
- In modular applications (JPMS), `setAccessible(true)` throws `InaccessibleObjectException` unless the target module explicitly `opens` the package to the calling module. This is why you see `--add-opens java.base/java.lang=ALL-UNNAMED` in JVM arguments for frameworks like Hibernate and Mockito.
- Frameworks use `setAccessible(true)` legitimately to inject dependencies into private fields (`@Autowired`), map JSON to private fields (`@JsonProperty`), and manage entity state (`@Id`). Application code should almost never use it.

### Performance Cost of Reflection

Reflection is significantly slower than direct method calls and field access. The JVM cannot optimize reflective calls the way it optimizes direct calls (no inlining, no escape analysis, no speculative optimization).

**Approximate performance comparison:**

| Operation | Direct Call | Reflection | Ratio |
|-----------|-------------|------------|-------|
| Method invocation | ~1 ns | ~10-50 ns | 10-50x slower |
| Field access | ~1 ns | ~5-20 ns | 5-20x slower |
| Constructor invocation | ~5 ns | ~50-200 ns | 10-40x slower |

**Why this matters:**

- A single reflective call is negligible. The cost becomes significant when reflection is used in tight loops or hot paths (e.g., processing millions of transactions).
- Frameworks mitigate this by caching reflective lookups. Spring resolves `@Autowired` fields once at startup and stores the `Field` objects. Jackson caches serializer/deserializer instances. Hibernate caches entity metadata. The expensive reflection happens once; subsequent operations use the cached results.
- If you write custom reflection code, cache the `Method`, `Field`, and `Constructor` objects. Do not call `getDeclaredMethod()` or `getDeclaredField()` inside a loop.

```java
// BAD — reflection lookup inside a loop
for (Transaction tx : transactions) {
    Method method = tx.getClass().getMethod("getAmount");  // Expensive lookup every iteration
    BigDecimal amount = (BigDecimal) method.invoke(tx);
}

// GOOD — cache the Method object
Method getAmount = Transaction.class.getMethod("getAmount");  // Lookup once
for (Transaction tx : transactions) {
    BigDecimal amount = (BigDecimal) getAmount.invoke(tx);  // Reuse cached Method
}

// BEST — avoid reflection entirely
for (Transaction tx : transactions) {
    BigDecimal amount = tx.getAmount();  // Direct call, JIT-optimized
}
```

### Method Handles (Modern Alternative)

Java 7 introduced `java.lang.invoke.MethodHandle` as a more performant and type-safe alternative to the `Method` class. Method handles are closer to JVM-level invocation and can be optimized by the JIT compiler.

```java
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;

// Obtain a MethodHandle
MethodHandles.Lookup lookup = MethodHandles.lookup();

// For a virtual method
MethodHandle getBalance = lookup.findVirtual(
    Account.class,
    "getBalance",
    MethodType.methodType(BigDecimal.class)
);

// Invoke
Account account = new Account("ACC-001", "Alice", new BigDecimal("5000"));
BigDecimal balance = (BigDecimal) getBalance.invoke(account);

// For a constructor
MethodHandle accountCtor = lookup.findConstructor(
    Account.class,
    MethodType.methodType(void.class, String.class, BigDecimal.class)
);
Account newAccount = (Account) accountCtor.invoke("ACC-002", new BigDecimal("1000"));

// For a private field (requires privateLookupIn, Java 9+)
MethodHandles.Lookup privateLookup = MethodHandles.privateLookupIn(
    Account.class, MethodHandles.lookup()
);
MethodHandle balanceGetter = privateLookup.findGetter(
    Account.class, "balance", BigDecimal.class
);
BigDecimal privateBalance = (BigDecimal) balanceGetter.invoke(account);
```

Method handles are faster than `Method.invoke()` for repeated invocations and are the preferred mechanism in performance-sensitive reflective code. However, they are more verbose and complex. Frameworks like Spring and Hibernate increasingly use method handles internally.

### When Reflection Is Used (and When It Is Not)

**Legitimate uses of reflection:**

- **Framework internals.** Spring's dependency injection, JPA's entity mapping, Jackson's serialization, JUnit's test discovery. You benefit from reflection without writing it.
- **Annotation processing at runtime.** Reading custom annotations to configure behavior (as demonstrated in the 1.13 Annotations section).
- **Plugin systems.** Loading and instantiating classes whose names are specified in configuration files.
- **Generic type resolution.** Reading generic type parameters at runtime (e.g., determining that a `Repository<Account>` manages `Account` entities).
- **Testing utilities.** Mockito creates mock objects via reflection. Testcontainers inspects test classes for `@Container` annotations.

**When to avoid reflection in application code:**

- **Business logic.** Never use reflection to call a service method or access a field in your domain logic. Use direct method calls.
- **Hot paths.** Never use reflection in code that executes millions of times per second (e.g., transaction processing loops).
- **When a type-safe alternative exists.** If you know the type at compile time, use it directly. Reflection is for situations where the type is genuinely unknown until runtime.
- **When performance matters more than flexibility.** Reflection bypasses JIT optimizations. In latency-sensitive fintech systems, every nanosecond counts.

---

## Code Examples

**A reflection-based object inspector (useful for debugging):**

```java
package com.example.fintech.util;

import java.lang.reflect.*;
import java.util.Arrays;

public class ObjectInspector {

    public static String inspect(Object obj) {
        if (obj == null) return "null";

        Class<?> clazz = obj.getClass();
        StringBuilder sb = new StringBuilder();
        sb.append(clazz.getSimpleName()).append(" {\n");

        // Fields
        for (Field field : clazz.getDeclaredFields()) {
            if (Modifier.isStatic(field.getModifiers())) continue;
            field.setAccessible(true);
            try {
                Object value = field.get(obj);
                String valueStr = formatValue(value);
                sb.append(String.format("  %s %s = %s%n",
                    field.getType().getSimpleName(),
                    field.getName(),
                    valueStr
                ));
            } catch (IllegalAccessException e) {
                sb.append(String.format("  %s %s = [inaccessible]%n",
                    field.getType().getSimpleName(),
                    field.getName()
                ));
            }
        }

        // Methods (public, non-inherited, no-arg only for brevity)
        sb.append("\n  Methods:\n");
        for (Method method : clazz.getDeclaredMethods()) {
            if (Modifier.isPublic(method.getModifiers())
                && method.getParameterCount() == 0
                && !method.getName().startsWith("get")
                && !method.getName().equals("toString")
                && !method.getName().equals("hashCode")) {
                sb.append(String.format("    %s %s()%n",
                    method.getReturnType().getSimpleName(),
                    method.getName()
                ));
            }
        }

        sb.append("}");
        return sb.toString();
    }

    private static String formatValue(Object value) {
        if (value == null) return "null";
        if (value instanceof String s) return "\"" + s + "\"";
        if (value.getClass().isArray()) {
            if (value instanceof Object[] arr) return Arrays.deepToString(arr);
            if (value instanceof int[] arr) return Arrays.toString(arr);
            if (value instanceof long[] arr) return Arrays.toString(arr);
            if (value instanceof double[] arr) return Arrays.toString(arr);
            return "[array]";
        }
        return value.toString();
    }
}

// Usage
Account account = new Account("ACC-001", "Alice", new BigDecimal("5000.00"));
System.out.println(ObjectInspector.inspect(account));
// Output:
// Account {
//   String accountId = "ACC-001"
//   String ownerName = "Alice"
//   BigDecimal balance = 5000.00
//   List transactions = []
//
//   Methods:
//     void deposit()
//     void withdraw()
// }
```

**A reflection-based generic type resolver (used by Spring Data and similar frameworks):**

```java
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

// Given: public class AccountRepository extends BaseRepository<Account, String>
// How does the framework know that this repository manages Account entities?

public abstract class BaseRepository<T, ID> {

    private final Class<T> entityType;
    private final Class<ID> idType;

    @SuppressWarnings("unchecked")
    protected BaseRepository() {
        // Resolve generic type parameters at runtime via reflection
        Type superClass = getClass().getGenericSuperclass();
        if (superClass instanceof ParameterizedType pt) {
            Type[] typeArgs = pt.getActualTypeArguments();
            this.entityType = (Class<T>) typeArgs[0];  // Account.class
            this.idType = (Class<ID>) typeArgs[1];      // String.class
        } else {
            throw new IllegalStateException("Cannot resolve generic types");
        }
    }

    public Class<T> getEntityType() { return entityType; }
    public Class<ID> getIdType() { return idType; }
}

public class AccountRepository extends BaseRepository<Account, String> {
    // No need to specify the types — they are resolved from the superclass
}

// Usage
AccountRepository repo = new AccountRepository();
System.out.println(repo.getEntityType());  // class com.example.fintech.model.Account
System.out.println(repo.getIdType());      // class java.lang.String
```

This is the exact mechanism that Spring Data JPA uses to determine the entity type of a repository interface. When you write `interface UserRepository extends JpaRepository<User, Long>`, Spring reads the generic type parameters via reflection and knows that the repository manages `User` entities with `Long` IDs.

**A dynamic method dispatcher (simplified version of Spring's request mapping):**

```java
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;

public class CommandDispatcher {
    private final Map<String, Method> commandHandlers = new HashMap<>();
    private final Object handler;

    public CommandDispatcher(Object handler) {
        this.handler = handler;
        // Scan for methods annotated with @Command
        for (Method method : handler.getClass().getDeclaredMethods()) {
            Command cmd = method.getAnnotation(Command.class);
            if (cmd != null) {
                commandHandlers.put(cmd.value(), method);
            }
        }
    }

    public Object dispatch(String commandName, Object... args) throws Exception {
        Method method = commandHandlers.get(commandName);
        if (method == null) {
            throw new IllegalArgumentException("Unknown command: " + commandName);
        }
        method.setAccessible(true);
        return method.invoke(handler, args);
    }
}

// Annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Command {
    String value();
}

// Handler
public class PaymentCommands {
    @Command("process")
    public String processPayment(String txId, BigDecimal amount) {
        return "Processed " + txId + " for $" + amount;
    }

    @Command("refund")
    public String refundPayment(String txId) {
        return "Refunded " + txId;
    }
}

// Usage
CommandDispatcher dispatcher = new CommandDispatcher(new PaymentCommands());
String result = (String) dispatcher.dispatch("process", "TX-001", new BigDecimal("500"));
// "Processed TX-001 for $500"
```

---

## Important Notes

- Reflection is the foundation of every Java framework you will use. Spring, Hibernate, Jackson, JUnit, Mockito, and dozens of other libraries rely on reflection to discover annotations, inject dependencies, map data, and generate proxies. Understanding reflection helps you understand why these frameworks behave the way they do and how to debug them when they do not.
- You should rarely write reflection code in application-level business logic. Reflection is a framework-level tool. If you find yourself using reflection in a service method or controller, there is almost certainly a type-safe alternative (interfaces, generics, strategy pattern, or a framework feature you are not aware of).
- The `Class` object is a singleton per class per classloader. `Account.class == account.getClass()` is always true (assuming the same classloader). This means `Class` objects can be used as keys in maps and sets.
- `getMethods()` returns all public methods including those inherited from superclasses and interfaces. `getDeclaredMethods()` returns all methods declared in the specific class, including private ones, but NOT inherited methods. This distinction is a common source of confusion. Use `getDeclaredMethods()` when you need to inspect a specific class's implementation. Use `getMethods()` when you need the full public API.
- `setAccessible(true)` is a security-sensitive operation. In modular applications (JPMS), it may throw `InaccessibleObjectException` if the target module does not open the package. This is the root cause of the `--add-opens` JVM flags you will encounter when running older libraries on Java 17+. The long-term fix is for libraries to stop using deep reflection, but this migration is ongoing.
- Reflection bypasses compile-time type checking. A reflective method invocation can throw `NoSuchMethodException`, `IllegalAccessException`, `InvocationTargetException`, and `ClassCastException` at runtime. Every reflective call should be wrapped in appropriate error handling.
- `InvocationTargetException` wraps the actual exception thrown by the invoked method. When a reflectively invoked method throws an `IllegalArgumentException`, you will catch an `InvocationTargetException` whose `getCause()` is the `IllegalArgumentException`. Always unwrap `InvocationTargetException` to find the real error.
- The performance cost of reflection is real but often overstated for typical application code. A single reflective call takes ~10-50 nanoseconds. In a web request that takes 50 milliseconds, this is negligible. The cost becomes significant only in tight loops processing millions of items. Frameworks cache reflective lookups to amortize the cost.
- Method handles (`java.lang.invoke.MethodHandle`) are the modern, higher-performance alternative to `Method.invoke()`. They are JIT-optimizable and can approach the speed of direct calls when used correctly. However, they are more complex to set up and are primarily used by framework authors, not application developers.
- Reflection cannot access local variables, read method bodies, or determine the implementation logic of a method. It can inspect signatures, annotations, and modifiers, but the actual bytecode of a method is not accessible through the standard reflection API. For bytecode inspection, you need libraries like ASM or ByteBuddy.
- Synthetic methods and bridge methods are generated by the compiler and visible through reflection. `method.isSynthetic()` and `method.isBridge()` allow you to filter them out. Bridge methods are generated to preserve polymorphism after type erasure (covered in the Generics section). Synthetic methods are generated for inner class access and other compiler-internal purposes.
- In the context of Spring Boot, the most common reflection-related error you will encounter is `BeanCreationException` caused by a failed `@Autowired` injection. This happens when Spring's reflection-based component scanner cannot find a matching bean for a dependency. The fix is usually a missing `@Component`/`@Service` annotation or a missing `@Bean` method, not a reflection problem per se.

---

## Practice

1. Write a method `void printClassStructure(Class<?> clazz)` that uses reflection to print the complete structure of a class: its superclass, interfaces, all fields (with types and modifiers), all constructors (with parameter types), and all methods (with return types, parameter types, and modifiers). Test it with `Account.class`, `String.class`, and `ArrayList.class`.

2. Write a method that takes any object and creates a deep string representation of all its fields (including private fields) using reflection. Handle nested objects recursively up to a depth of 3 to avoid infinite loops. Compare the output with the object's `toString()` method.

3. Create a simple factory that instantiates objects from their fully qualified class name. The factory should accept a class name as a string (e.g., `"com.example.fintech.model.Account"`), load the class via `Class.forName()`, find a constructor that matches the provided arguments, and return the new instance. Handle all reflection exceptions gracefully.

4. Demonstrate the performance difference between direct method calls and reflective method calls. Create a method that adds two `BigDecimal` values. Call it 10,000,000 times directly and 10,000,000 times via `Method.invoke()`. Measure and compare the execution times. Then repeat the reflective test with a cached `Method` object and with a `MethodHandle`.

5. Write a reflection-based utility that reads all fields annotated with a custom `@Encrypted` annotation from an object and "encrypts" their string values (e.g., by Base64-encoding them). Then write the corresponding "decryption" utility. This simulates what frameworks like Hibernate Envers or Jasypt do internally.

6. Inspect the generic type parameters of a class that extends a generic superclass (e.g., `class AccountRepository extends BaseRepository<Account, String>`). Use `getGenericSuperclass()` and `ParameterizedType.getActualTypeArguments()` to extract the type arguments at runtime. Document why this works despite type erasure.

7. In your Obsidian vault, create a diagram showing how Spring Boot uses reflection at startup: component scanning (reading `@Component` annotations), dependency injection (setting `@Autowired` fields), request mapping (reading `@GetMapping` methods), and entity mapping (reading `@Entity` and `@Column` annotations). This will help you debug Spring configuration issues in Phase 05.

---

## References

- java.lang.reflect Package: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/package-summary.html
- java.lang.invoke Package (Method Handles): https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/invoke/package-summary.html
- Class API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html
- Java Tutorial — Reflection: https://docs.oracle.com/javase/tutorial/reflect/
- JEP 416 (Reimplement Core Reflection with Method Handles, Java 18): https://openjdk.org/jeps/416
- "Effective Java" by Joshua Bloch — Item 65 (Prefer interfaces to reflection)
- "Java Reflection in Action" by Ira R. Forman and Nate Forman
