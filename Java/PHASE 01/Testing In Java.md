## Overview

Testing is the practice of writing code that verifies your code behaves correctly. In Java backend engineering, testing is not optional — it is a professional obligation. Every enterprise codebase you will ever work in has a test suite, and your pull requests will be rejected without adequate test coverage.

Java has the most mature testing ecosystem of any language. JUnit 5 is the standard test framework, Mockito is the standard mocking library, and AssertJ provides fluent, readable assertions. Together they form the testing trinity that every Java developer must master.

Testing in Java is also deeply integrated into the build lifecycle. Maven and Gradle run your tests automatically during `mvn test` or `gradle build`. CI pipelines gate merges on test results. This means tests are not something you "run sometimes" — they are a structural part of the software delivery pipeline.

This note covers the full testing stack: writing tests, mocking dependencies, asserting outcomes, measuring coverage, and understanding when and how to apply Test-Driven Development.

---

## Core Concepts

### The Testing Pyramid

```
         /\
        /  \        E2E Tests (few, slow, expensive)
       /----\
      /      \      Integration Tests (some, moderate)
     /--------\
    /          \    Unit Tests (many, fast, cheap)
   /____________\
```

**Unit tests** test a single class or method in isolation. Dependencies are mocked. They run in milliseconds. You should have thousands of these.

**Integration tests** test how components work together (e.g., your service + a real database). They are slower and require infrastructure.

**End-to-end tests** test the full system from HTTP request to database and back. They are the slowest and most brittle.

The pyramid tells you: write many unit tests, fewer integration tests, and very few E2E tests. Most of your testing time in this phase will be on unit tests.

### JUnit 5 Architecture

JUnit 5 is not a single library. It has three sub-projects:

```
JUnit 5
├── JUnit Platform    → the foundation for launching tests on the JVM
├── JUnit Jupiter     → the new programming model (@Test, assertions, extensions)
└── JUnit Vintage     → backward compatibility for JUnit 3/4 tests
```

You will use JUnit Jupiter exclusively. The Platform runs underneath automatically. Vintage is only relevant when migrating legacy codebases.

### Test Lifecycle

JUnit 5 controls when setup and teardown methods run relative to your tests:

```
@BeforeAll      → runs ONCE before all tests in the class (static method)
  @BeforeEach   → runs before EACH test method
    @Test       → the actual test
  @AfterEach    → runs after EACH test method
@AfterAll       → runs ONCE after all tests in the class (static method)
```

Execution order for a class with two tests:

```
@BeforeAll
  @BeforeEach → @Test method1 → @AfterEach
  @BeforeEach → @Test method2 → @AfterEach
@AfterAll
```

### Assertions vs Assumptions

**Assertions** (`assertEquals`, `assertTrue`, etc.) verify expected outcomes. A failed assertion fails the test.

**Assumptions** (`assumeTrue`, `assumeFalse`) check preconditions. A failed assumption aborts the test (marked as skipped, not failed). Use these when a test only makes sense under certain conditions (e.g., a specific OS or environment variable).

### Mocking

A mock is a fake object that simulates the behavior of a real dependency. You use mocks when:

- The real dependency is slow (database, HTTP call)
- The real dependency is unavailable in the test environment
- You want to test how your code interacts with the dependency (did it call the right method with the right arguments?)
- You want to simulate error conditions (what happens when the database throws?)

Mockito is the standard mocking library in Java. It creates mock objects at runtime using bytecode generation (via ByteBuddy). You tell the mock what to return when called, then verify it was called correctly.

### Spies vs Mocks

A **mock** is entirely fake. Every method returns a default value (null, 0, false, empty collection) unless you configure it.

A **spy** wraps a real object. Methods execute their real logic unless you explicitly stub them. Use spies when you want to verify interactions on a mostly-real object.

### Fluent Assertions (AssertJ)

JUnit's built-in assertions work but produce poor error messages and are hard to read when chained. AssertJ provides a fluent API that reads like English:

```java
// JUnit
assertEquals(3, list.size());
assertTrue(list.contains("apple"));

// AssertJ
assertThat(list).hasSize(3).contains("apple");
```

AssertJ also provides type-specific assertions (for strings, collections, dates, exceptions, optionals, etc.) with hundreds of built-in checks.

### Code Coverage

Code coverage measures what percentage of your source code is executed by your tests. JaCoCo (Java Code Coverage) is the standard tool. It instruments your bytecode during test execution and produces HTML/XML reports.

**Critical caveat:** High coverage does not mean good tests. A test that executes a line but asserts nothing provides 100% coverage and 0% confidence. Coverage is a minimum threshold metric, not a quality metric. Aim for 80%+ line coverage on business logic, but focus on meaningful assertions.

### Test-Driven Development (TDD)

TDD is a development methodology where you write the test before the implementation:

```
1. RED       → Write a failing test for the desired behavior
2. GREEN     → Write the minimum code to make the test pass
3. REFACTOR  → Clean up the code while keeping tests green
```

TDD is a discipline, not a religion. It works exceptionally well for business logic, algorithms, and well-defined specifications. It works poorly for exploratory UI work, throwaway prototypes, and infrastructure configuration.

---

## Code Examples

### Maven Dependencies

```xml
<dependencies>
    <!-- JUnit 5 (Jupiter) -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>

    <!-- Mockito -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>

    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.25.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

For Maven, ensure the Surefire plugin is configured (it is by default in modern Maven):

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
```

### Basic JUnit 5 Test

The class under test:

```java
package com.example.finance;

public class Calculator {

    public double add(double a, double b) {
        return a + b;
    }

    public double divide(double a, double b) {
        if (b == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return a / b;
    }

    public boolean isEven(int n) {
        return n % 2 == 0;
    }
}
```

The test class:

```java
package com.example.finance;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    private final Calculator calculator = new Calculator();

    @Test
    void add_shouldReturnSumOfTwoNumbers() {
        double result = calculator.add(2.0, 3.0);
        assertEquals(5.0, result);
    }

    @Test
    void add_shouldHandleNegativeNumbers() {
        assertEquals(-1.0, calculator.add(2.0, -3.0));
    }

    @Test
    void divide_shouldReturnQuotient() {
        assertEquals(2.5, calculator.divide(5.0, 2.0));
    }

    @Test
    void divide_shouldThrowExceptionWhenDividingByZero() {
        ArithmeticException exception = assertThrows(
            ArithmeticException.class,
            () -> calculator.divide(10.0, 0.0)
        );
        assertEquals("Cannot divide by zero", exception.getMessage());
    }

    @Test
    void isEven_shouldReturnTrueForEvenNumbers() {
        assertTrue(calculator.isEven(4));
        assertTrue(calculator.isEven(0));
        assertTrue(calculator.isEven(-2));
    }

    @Test
    void isEven_shouldReturnFalseForOddNumbers() {
        assertFalse(calculator.isEven(3));
        assertFalse(calculator.isEven(-1));
    }
}
```

### JUnit 5 Assertions — Complete Reference

```java
import static org.junit.jupiter.api.Assertions.*;
import java.time.Duration;
import java.util.List;

class AssertionsShowcaseTest {

    @Test
    void equalityAssertions() {
        assertEquals(5, 5);
        assertEquals("hello", "hello");
        assertEquals(3.14, 3.14, 0.001);  // with delta for doubles
        assertNotEquals(5, 6);
    }

    @Test
    void booleanAssertions() {
        assertTrue(true);
        assertTrue(5 > 3, "5 should be greater than 3");  // custom message
        assertFalse(false);
    }

    @Test
    void nullAssertions() {
        String value = null;
        assertNull(value);

        String nonNull = "hello";
        assertNotNull(nonNull);
    }

    @Test
    void sameReferenceAssertion() {
        String a = "hello";
        String b = "hello";  // same string pool reference
        assertSame(a, b);

        String c = new String("hello");
        assertNotSame(a, c);  // different objects, same value
    }

    @Test
    void exceptionAssertions() {
        // Assert that an exception IS thrown
        Exception ex = assertThrows(IllegalArgumentException.class, () -> {
            throw new IllegalArgumentException("bad input");
        });
        assertEquals("bad input", ex.getMessage());

        // Assert that NO exception is thrown
        assertDoesNotThrow(() -> {
            int result = 2 + 2;
        });
    }

    @Test
    void groupedAssertions() {
        // assertAll groups multiple assertions; all run even if some fail
        String name = "Alice";
        assertAll("person properties",
            () -> assertEquals("Alice", name),
            () -> assertTrue(name.length() > 0),
            () -> assertTrue(name.startsWith("A"))
        );
    }

    @Test
    void timeoutAssertions() {
        // Fails if the operation takes longer than the specified duration
        assertTimeout(Duration.ofMillis(100), () -> {
            Thread.sleep(50);  // this is fine
        });

        // Preemptive timeout: interrupts the thread if it exceeds the limit
        assertTimeoutPreemptively(Duration.ofSeconds(1), () -> {
            // some operation that should complete quickly
        });
    }

    @Test
    void iterableAssertions() {
        List<String> fruits = List.of("apple", "banana", "cherry");
        assertIterableEquals(List.of("apple", "banana", "cherry"), fruits);
    }

    @Test
    void arrayAssertions() {
        int[] expected = {1, 2, 3};
        int[] actual = {1, 2, 3};
        assertArrayEquals(expected, actual);
    }
}
```

### Test Lifecycle Annotations

```java
import org.junit.jupiter.api.*;

class LifecycleDemoTest {

    @BeforeAll
    static void setUpAll() {
        // Runs once before any test. Must be static.
        // Use for expensive setup: start embedded database, load config
        System.out.println("@BeforeAll — runs once");
    }

    @BeforeEach
    void setUp() {
        // Runs before each test method.
        // Use for per-test setup: create fresh objects, reset state
        System.out.println("@BeforeEach — runs before each test");
    }

    @Test
    void testOne() {
        System.out.println("testOne");
    }

    @Test
    void testTwo() {
        System.out.println("testTwo");
    }

    @AfterEach
    void tearDown() {
        // Runs after each test method.
        // Use for per-test cleanup: close connections, delete temp files
        System.out.println("@AfterEach — runs after each test");
    }

    @AfterAll
    static void tearDownAll() {
        // Runs once after all tests. Must be static.
        System.out.println("@AfterAll — runs once");
    }
}

// Output order:
// @BeforeAll — runs once
// @BeforeEach — runs before each test
// testOne
// @AfterEach — runs after each test
// @BeforeEach — runs before each test
// testTwo
// @AfterEach — runs after each test
// @AfterAll — runs once
```

### Display Names and Disabling Tests

```java
import org.junit.jupiter.api.*;

@DisplayName("Payment Service Tests")
class PaymentServiceTest {

    @Test
    @DisplayName("Should reject payment when balance is insufficient")
    void rejectInsufficientBalance() {
        // ...
    }

    @Test
    @DisplayName("Should apply 2% fee for international transfers")
    void applyInternationalFee() {
        // ...
    }

    @Test
    @Disabled("Disabled until the new tax calculation module is ready")
    void calculateTax() {
        // This test will be skipped in the test run
    }
}
```

### Nested Test Classes

Nested classes let you group related tests hierarchically, sharing setup logic:

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("Account")
class AccountTest {

    Account account;

    @BeforeEach
    void setUp() {
        account = new Account("ACC-001", 1000.00);
    }

    @Nested
    @DisplayName("when depositing")
    class Depositing {

        @Test
        @DisplayName("should increase balance")
        void increaseBalance() {
            account.deposit(500.00);
            assertEquals(1500.00, account.getBalance());
        }

        @Test
        @DisplayName("should reject negative amounts")
        void rejectNegative() {
            assertThrows(IllegalArgumentException.class,
                () -> account.deposit(-100.00));
        }
    }

    @Nested
    @DisplayName("when withdrawing")
    class Withdrawing {

        @Test
        @DisplayName("should decrease balance")
        void decreaseBalance() {
            account.withdraw(300.00);
            assertEquals(700.00, account.getBalance());
        }

        @Test
        @DisplayName("should reject overdraft")
        void rejectOverdraft() {
            assertThrows(InsufficientFundsException.class,
                () -> account.withdraw(2000.00));
        }
    }
}
```

Test output reads like a specification:

```
Account
  when depositing
    should increase balance ✓
    should reject negative amounts ✓
  when withdrawing
    should decrease balance ✓
    should reject overdraft ✓
```

### Parameterized Tests

Parameterized tests run the same test logic with different inputs. This eliminates massive copy-paste:

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.api.Assertions.*;

class StringUtilsTest {

    @ParameterizedTest
    @ValueSource(strings = {"racecar", "madam", "level", "a"})
    void isPalindrome_shouldReturnTrueForPalindromes(String input) {
        assertTrue(StringUtils.isPalindrome(input));
    }

    @ParameterizedTest
    @ValueSource(ints = {2, 4, 6, 8, 100})
    void isEven_shouldReturnTrueForEvenNumbers(int number) {
        assertTrue(MathUtils.isEven(number));
    }

    @ParameterizedTest
    @CsvSource({
        "hello, 5",
        "world, 5",
        "'', 0",
        "java, 4"
    })
    void length_shouldReturnCorrectLength(String input, int expected) {
        assertEquals(expected, input.length());
    }

    @ParameterizedTest
    @CsvSource({
        "10.0, 20.0, 30.0",
        "-5.0, 5.0, 0.0",
        "0.0, 0.0, 0.0"
    })
    void add_shouldReturnSum(double a, double b, double expected) {
        assertEquals(expected, new Calculator().add(a, b), 0.001);
    }

    @ParameterizedTest
    @EnumSource(TransactionStatus.class)
    void process_shouldHandleAllStatuses(TransactionStatus status) {
        // Runs once for each enum constant
        assertNotNull(status.name());
    }

    @ParameterizedTest
    @MethodSource("provideTestAccounts")
    void validate_shouldAcceptValidAccounts(Account account) {
        assertTrue(account.isValid());
    }

    // Method source must be static and return a Stream
    static java.util.stream.Stream<Account> provideTestAccounts() {
        return java.util.stream.Stream.of(
            new Account("ACC-001", 1000.00),
            new Account("ACC-002", 0.00),
            new Account("ACC-003", 999999.99)
        );
    }
}
```

### Repeated Tests

```java
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.RepetitionInfo;

class RandomGeneratorTest {

    @RepeatedTest(value = 10, name = "Run {currentRepetition} of {totalRepetitions}")
    void generate_shouldProduceValueInRange(RepetitionInfo info) {
        int value = RandomGenerator.nextInt(1, 100);
        assertTrue(value >= 1 && value <= 100);
    }
}
```

### Assumptions

```java
import static org.junit.jupiter.api.Assumptions.*;

class EnvironmentSensitiveTest {

    @Test
    void testOnlyOnLinux() {
        assumeTrue(System.getProperty("os.name").contains("Linux"),
            "Test only runs on Linux");
        // This code only executes on Linux; skipped on macOS/Windows
    }

    @Test
    void testOnlyWhenDatabaseAvailable() {
        String dbUrl = System.getenv("DATABASE_URL");
        assumingThat(dbUrl != null, () -> {
            // This block only runs if DATABASE_URL is set
            assertNotNull(connectToDatabase(dbUrl));
        });
        // Code here always runs
    }
}
```

### Test Ordering

By default, JUnit 5 uses a deterministic but intentionally non-obvious ordering. You can control it:

```java
import org.junit.jupiter.api.*;

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class OrderedTest {

    @Test
    @Order(1)
    void first() { /* ... */ }

    @Test
    @Order(2)
    void second() { /* ... */ }

    @Test
    @Order(3)
    void third() { /* ... */ }
}

// Other ordering strategies:
// MethodOrderer.DisplayName  → alphabetical by @DisplayName
// MethodOrderer.MethodName   → alphabetical by method name
// MethodOrderer.Random       → random (useful for detecting order dependencies)
```

Best practice: Tests should be independent. If test B depends on test A running first, you have a design problem. Use ordering sparingly (e.g., integration tests that build on shared state).

### Test Instance Lifecycle

By default, JUnit 5 creates a new instance of the test class for each test method. This ensures isolation. You can change this:

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class SharedInstanceTest {
    // One instance for all tests. @BeforeAll/@AfterAll can be non-static.
    // Use with caution: shared mutable state between tests is dangerous.
    private int counter = 0;

    @BeforeAll
    void setUp() {
        counter = 0;  // non-static because PER_CLASS
    }
}
```

### Mockito — Core Usage

The class under test and its dependency:

```java
public class TransferService {

    private final AccountRepository accountRepository;
    private final NotificationService notificationService;

    public TransferService(AccountRepository accountRepository,
                           NotificationService notificationService) {
        this.accountRepository = accountRepository;
        this.notificationService = notificationService;
    }

    public void transfer(String fromId, String toId, BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }

        Account from = accountRepository.findById(fromId)
            .orElseThrow(() -> new AccountNotFoundException(fromId));
        Account to = accountRepository.findById(toId)
            .orElseThrow(() -> new AccountNotFoundException(toId));

        from.withdraw(amount);
        to.deposit(amount);

        accountRepository.save(from);
        accountRepository.save(to);

        notificationService.sendTransferConfirmation(fromId, toId, amount);
    }
}
```

The test with Mockito:

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.math.BigDecimal;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class TransferServiceTest {

    @Mock
    private AccountRepository accountRepository;

    @Mock
    private NotificationService notificationService;

    @InjectMocks
    private TransferService transferService;

    @Test
    void transfer_shouldMoveMoneyBetweenAccounts() {
        // Given (Arrange)
        Account from = new Account("ACC-001", new BigDecimal("1000.00"));
        Account to = new Account("ACC-002", new BigDecimal("500.00"));
        BigDecimal amount = new BigDecimal("200.00");

        when(accountRepository.findById("ACC-001")).thenReturn(Optional.of(from));
        when(accountRepository.findById("ACC-002")).thenReturn(Optional.of(to));

        // When (Act)
        transferService.transfer("ACC-001", "ACC-002", amount);

        // Then (Assert)
        assertEquals(new BigDecimal("800.00"), from.getBalance());
        assertEquals(new BigDecimal("700.00"), to.getBalance());

        verify(accountRepository).save(from);
        verify(accountRepository).save(to);
        verify(notificationService).sendTransferConfirmation("ACC-001", "ACC-002", amount);
    }

    @Test
    void transfer_shouldThrowWhenSourceAccountNotFound() {
        when(accountRepository.findById("ACC-999")).thenReturn(Optional.empty());

        assertThrows(AccountNotFoundException.class,
            () -> transferService.transfer("ACC-999", "ACC-002", BigDecimal.TEN));

        verify(accountRepository, never()).save(any());
        verify(notificationService, never()).sendTransferConfirmation(any(), any(), any());
    }

    @Test
    void transfer_shouldRejectNonPositiveAmount() {
        assertThrows(IllegalArgumentException.class,
            () -> transferService.transfer("ACC-001", "ACC-002", BigDecimal.ZERO));
    }
}
```

### Mockito — Stubbing Patterns

```java
// Return a value
when(repository.findById("id")).thenReturn(Optional.of(account));

// Return different values on successive calls
when(repository.nextId()).thenReturn(1).thenReturn(2).thenReturn(3);

// Throw an exception
when(repository.save(any())).thenThrow(new DatabaseException("connection lost"));

// Answer dynamically based on arguments
when(repository.findById(anyString())).thenAnswer(invocation -> {
    String id = invocation.getArgument(0);
    return id.startsWith("ACC") ? Optional.of(new Account(id)) : Optional.empty();
});

// For void methods — use doThrow/doNothing/doAnswer
doThrow(new RuntimeException("fail")).when(notificationService).send(any());
doNothing().when(notificationService).send(any());
doAnswer(invocation -> {
    String msg = invocation.getArgument(0);
    System.out.println("Captured: " + msg);
    return null;
}).when(notificationService).send(any());
```

### Mockito — Verification Patterns

```java
// Verify a method was called exactly once (default)
verify(repository).save(account);

// Verify specific call counts
verify(repository, times(2)).save(any());
verify(repository, times(1)).findById("ACC-001");
verify(notificationService, never()).send(any());
verify(repository, atLeast(1)).save(any());
verify(repository, atMost(3)).findById(anyString());

// Verify no interactions at all
verifyNoInteractions(notificationService);

// Verify no more interactions beyond what was already verified
verifyNoMoreInteractions(repository);

// Verify call order across mocks
InOrder inOrder = inOrder(repository, notificationService);
inOrder.verify(repository).save(from);
inOrder.verify(repository).save(to);
inOrder.verify(notificationService).sendTransferConfirmation(any(), any(), any());
```

### Mockito — ArgumentCaptor

Use ArgumentCaptor when you need to inspect the actual arguments passed to a mock:

```java
@Test
void transfer_shouldSaveAccountsWithUpdatedBalances() {
    // ... setup ...

    transferService.transfer("ACC-001", "ACC-002", new BigDecimal("200.00"));

    ArgumentCaptor<Account> captor = ArgumentCaptor.forClass(Account.class);
    verify(accountRepository, times(2)).save(captor.capture());

    List<Account> savedAccounts = captor.getAllValues();
    assertEquals(2, savedAccounts.size());

    Account savedFrom = savedAccounts.stream()
        .filter(a -> a.getId().equals("ACC-001")).findFirst().orElseThrow();
    Account savedTo = savedAccounts.stream()
        .filter(a -> a.getId().equals("ACC-002")).findFirst().orElseThrow();

    assertEquals(new BigDecimal("800.00"), savedFrom.getBalance());
    assertEquals(new BigDecimal("700.00"), savedTo.getBalance());
}
```

### Mockito — Mocking Static Methods (Mockito 3.4+)

```java
import org.mockito.MockedStatic;

@Test
void shouldMockStaticMethod() {
    try (MockedStatic<LocalDate> mockedDate = mockStatic(LocalDate.class)) {
        mockedDate.when(LocalDate::now).thenReturn(LocalDate.of(2025, 1, 15));

        assertEquals(LocalDate.of(2025, 1, 15), LocalDate.now());

        // Test code that depends on LocalDate.now()
        assertTrue(service.isWithinCurrentMonth(someDate));
    }
    // Static mock is automatically released after the try-with-resources block
}
```

### Mockito — BDD Style (Given/When/Then)

Mockito provides a BDD (Behavior-Driven Development) API that reads more naturally:

```java
import static org.mockito.BDDMockito.*;

@Test
void transfer_bddStyle() {
    // Given
    Account from = new Account("ACC-001", new BigDecimal("1000.00"));
    given(accountRepository.findById("ACC-001")).willReturn(Optional.of(from));
    given(accountRepository.findById("ACC-002")).willReturn(Optional.of(to));

    // When
    transferService.transfer("ACC-001", "ACC-002", new BigDecimal("200.00"));

    // Then
    then(accountRepository).should(times(2)).save(any());
    then(notificationService).should().sendTransferConfirmation(any(), any(), any());
}
```

### Mockito — Spies

```java
@Test
void spyExample() {
    List<String> realList = new ArrayList<>();
    List<String> spyList = spy(realList);

    spyList.add("one");
    spyList.add("two");

    // Real methods are called
    assertEquals(2, spyList.size());

    // But you can stub specific methods
    when(spyList.size()).thenReturn(100);
    assertEquals(100, spyList.size());

    verify(spyList).add("one");
    verify(spyList).add("two");
}
```

**Warning with spies:** Use `doReturn().when(spy)` instead of `when(spy.method()).thenReturn()` to avoid calling the real method during stubbing.

### AssertJ — Fluent Assertions

```java
import static org.assertj.core.api.Assertions.*;

class AssertJShowcaseTest {

    @Test
    void stringAssertions() {
        String name = "Alice Johnson";

        assertThat(name)
            .isNotNull()
            .isNotEmpty()
            .startsWith("Alice")
            .endsWith("Johnson")
            .contains("ce Jo")
            .hasSize(13)
            .isEqualToIgnoringCase("alice johnson");
    }

    @Test
    void collectionAssertions() {
        List<String> fruits = List.of("apple", "banana", "cherry", "date");

        assertThat(fruits)
            .hasSize(4)
            .contains("banana", "date")
            .doesNotContain("grape")
            .containsExactly("apple", "banana", "cherry", "date")
            .allMatch(f -> f.length() > 0)
            .anyMatch(f -> f.startsWith("c"))
            .noneMatch(f -> f.isEmpty());
    }

    @Test
    void mapAssertions() {
        Map<String, Integer> ages = Map.of("Alice", 30, "Bob", 25);

        assertThat(ages)
            .hasSize(2)
            .containsEntry("Alice", 30)
            .containsKey("Bob")
            .doesNotContainKey("Charlie")
            .containsValues(25, 30);
    }

    @Test
    void exceptionAssertions() {
        assertThatThrownBy(() -> {
            throw new IllegalArgumentException("bad input");
        })
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("bad input")
            .hasMessageContaining("bad");

        // Alternative syntax
        assertThatExceptionOfType(IllegalArgumentException.class)
            .isThrownBy(() -> { throw new IllegalArgumentException("bad"); })
            .withMessage("bad");
    }

    @Test
    void optionalAssertions() {
        Optional<String> present = Optional.of("hello");
        Optional<String> empty = Optional.empty();

        assertThat(present).isPresent().hasValue("hello");
        assertThat(empty).isEmpty();
    }

    @Test
    void bigDecimalAssertions() {
        BigDecimal price = new BigDecimal("19.99");

        assertThat(price)
            .isEqualByComparingTo(new BigDecimal("19.99"))
            .isGreaterThan(BigDecimal.ZERO)
            .isLessThan(new BigDecimal("20.00"));
    }

    @Test
    void softAssertions() {
        // Soft assertions collect ALL failures instead of stopping at the first
        SoftAssertions softly = new SoftAssertions();

        softly.assertThat("hello").hasSize(5);
        softly.assertThat(42).isGreaterThan(50);  // this fails
        softly.assertThat(true).isTrue();

        softly.assertAll();  // reports all failures at once
    }
}
```

### JaCoCo — Code Coverage

Maven configuration:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.12</version>
            <executions>
                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
                <execution>
                    <id>check</id>
                    <goals>
                        <goal>check</goal>
                    </goals>
                    <configuration>
                        <rules>
                            <rule>
                                <element>BUNDLE</element>
                                <limits>
                                    <limit>
                                        <counter>LINE</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.80</minimum>
                                    </limit>
                                </limits>
                            </rule>
                        </rules>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

Running coverage:

```bash
mvn clean test
# Report generated at: target/site/jacoco/index.html
# Build fails if coverage < 80%
```

Coverage metrics JaCoCo tracks:

```
Instructions  → bytecode instructions (most granular)
Branches      → if/else, switch, ternary branches
Lines         → source code lines
Methods       → method-level coverage
Classes       → class-level coverage
Complexity    → cyclomatic complexity coverage
```

### TDD Example — Red-Green-Refactor

**Step 1: RED — Write a failing test**

```java
@Test
void fizzBuzz_shouldReturnFizzForMultiplesOfThree() {
    FizzBuzz fizzBuzz = new FizzBuzz();
    assertEquals("Fizz", fizzBuzz.convert(3));  // FAILS: class doesn't exist yet
}
```

**Step 2: GREEN — Write minimum code to pass**

```java
public class FizzBuzz {
    public String convert(int n) {
        if (n % 3 == 0) return "Fizz";
        return String.valueOf(n);
    }
}
```

**Step 3: Add more tests (RED again)**

```java
@Test
void fizzBuzz_shouldReturnBuzzForMultiplesOfFive() {
    assertEquals("Buzz", new FizzBuzz().convert(5));  // FAILS
}

@Test
void fizzBuzz_shouldReturnFizzBuzzForMultiplesOfFifteen() {
    assertEquals("FizzBuzz", new FizzBuzz().convert(15));  // FAILS
}
```

**Step 4: GREEN — Make all tests pass**

```java
public class FizzBuzz {
    public String convert(int n) {
        if (n % 15 == 0) return "FizzBuzz";
        if (n % 3 == 0) return "Fizz";
        if (n % 5 == 0) return "Buzz";
        return String.valueOf(n);
    }
}
```

**Step 5: REFACTOR — Clean up while tests stay green**

```java
public class FizzBuzz {
    public String convert(int n) {
        if (n % 15 == 0) return "FizzBuzz";
        if (n % 3 == 0) return "Fizz";
        if (n % 5 == 0) return "Buzz";
        return Integer.toString(n);
    }
}
// (In this simple case, not much to refactor. In real code, this is where
// you extract methods, rename variables, and improve structure.)
```

---

## Important Notes

### Test Naming Conventions

Good test names are specifications. They should describe the behavior, not the implementation:

```
// BAD
void test1() { }
void testAdd() { }
void testTransferMethod() { }

// GOOD (method_shouldDoSomething_whenCondition)
void add_shouldReturnSumOfTwoPositiveNumbers() { }
void transfer_shouldThrowException_whenBalanceIsInsufficient() { }
void withdraw_shouldRejectNegativeAmount() { }

// GOOD (BDD style)
void givenSufficientBalance_whenWithdraw_thenBalanceDecreases() { }
```

### Given-When-Then / Arrange-Act-Assert

Every test should have three clear sections:

```java
@Test
void transfer_shouldMoveMoneyBetweenAccounts() {
    // GIVEN (Arrange): set up preconditions and inputs
    Account from = new Account("ACC-001", new BigDecimal("1000.00"));
    Account to = new Account("ACC-002", new BigDecimal("500.00"));
    when(repository.findById("ACC-001")).thenReturn(Optional.of(from));
    when(repository.findById("ACC-002")).thenReturn(Optional.of(to));

    // WHEN (Act): execute the behavior under test
    service.transfer("ACC-001", "ACC-002", new BigDecimal("200.00"));

    // THEN (Assert): verify the expected outcomes
    assertEquals(new BigDecimal("800.00"), from.getBalance());
    verify(repository, times(2)).save(any());
}
```

### Unit Test vs Integration Test Separation

**Unit Tests:**
- No Spring context (no `@SpringBootTest`)
- Dependencies are mocked with Mockito
- Run in milliseconds
- Test one class in isolation
- Live in `src/test/java` alongside the class under test
- Example: `TransferServiceTest` with mocked `AccountRepository`

**Integration Tests:**
- Real dependencies (database, Redis, message queue)
- Use `@SpringBootTest` or test slices (`@DataJpaTest`, `@WebMvcTest`)
- Use Testcontainers for infrastructure
- Run in seconds
- Test component interactions
- Example: `TransferServiceIntegrationTest` with real PostgreSQL

### What Coverage Numbers Mean (and Don't Mean)

**Coverage tells you:**
- Which lines of code are executed by tests
- Which branches are taken
- Where you have zero test coverage (danger zones)

**Coverage does NOT tell you:**
- Whether your assertions are meaningful
- Whether your tests verify correct behavior
- Whether edge cases are handled
- Whether your tests are maintainable

A test with 100% coverage and no assertions is worthless. A test with 60% coverage and thorough assertions is valuable.

**Target:** 80%+ line coverage on business logic. Do not obsess over 100%. The last 10-20% is often getters, configuration, and framework glue that provides diminishing returns.

### Common Testing Anti-Patterns

- Testing implementation details instead of behavior (e.g., asserting that a specific private method was called)
- Tests that depend on execution order
- Tests that depend on external state (network, filesystem, time) without controlling it
- Tests with no assertions (they pass but prove nothing)
- Catching exceptions in tests and swallowing them
- Using `Thread.sleep()` to wait for async operations (use Awaitility or `assertTimeout` instead)
- Giant test methods that test 10 things at once (one assertion concept per test)
- Copy-pasting setup code instead of using `@BeforeEach` or helper methods
- Mocking the class under test (you should mock dependencies, not the SUT)

### When TDD Helps vs Hinders

**TDD helps when:**
- Requirements are clear and well-defined
- You are writing business logic, algorithms, or data transformations
- You want to ensure comprehensive edge case coverage
- You are working in a team and tests serve as documentation
- The code will be long-lived and frequently modified

**TDD hinders when:**
- You are prototyping or exploring a solution
- Requirements are vague and changing rapidly
- You are writing infrastructure glue code (configuration, wiring)
- The framework dictates the structure (e.g., Spring Boot auto-config)
- You are learning a new technology and don't yet know what to test

---

## Practice

1. Set up a Maven project with JUnit 5, Mockito, AssertJ, and JaCoCo
2. Write 10 unit tests for a Calculator class covering all assertion types
3. Write parameterized tests for a StringUtils class (palindrome, reverse, capitalize)
4. Use `@Nested` to organize tests for a BankAccount class (deposit, withdraw, transfer)
5. Mock a UserRepository and test a UserService that depends on it
6. Use ArgumentCaptor to verify the exact object passed to a mock's `save()` method
7. Write a test that verifies an exception is thrown with the correct message
8. Use AssertJ soft assertions to verify multiple properties of a returned object
9. Mock a static method (e.g., `LocalDateTime.now()`) in a test
10. Write a spy test for a class where you want to verify one method but use real logic for others
11. Configure JaCoCo with an 80% minimum coverage threshold and intentionally break it
12. Implement FizzBuzz using strict TDD (red-green-refactor for every case)
13. Write a test using `@ParameterizedTest` with `@CsvSource` for a currency converter
14. Refactor a test class that uses JUnit `assertEquals` to use AssertJ `assertThat`
15. Write a test with `@RepeatedTest` to verify a random number generator stays in bounds

---

## References

- JUnit 5 User Guide: https://junit.org/junit5/docs/current/user-guide/
- Mockito Documentation: https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html
- AssertJ Documentation: https://assertj.github.io/doc/
- JaCoCo Documentation: https://www.jacoco.org/jacoco/trunk/doc/
- "Effective Unit Testing" — Lasse Koskela
- "Test-Driven Development: By Example" — Kent Beck
