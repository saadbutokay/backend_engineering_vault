---
title: "Java - Date and Time API - LocalDate LocalDateTime"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - date-time
  - java8
  - temporal
  - jpa
status: "not-started"
---

# Java - Date and Time API - LocalDate LocalDateTime

> [!abstract] Overview
> The `java.time` package, introduced in Java 8 (JSR-310), is a modern, comprehensive, and thread-safe API for handling dates, times, durations, and time zones. It replaces the deeply flawed legacy classes `java.util.Date`, `java.util.Calendar`, and `java.text.SimpleDateFormat`, which were mutable, not thread-safe, poorly designed, and a source of countless bugs in production systems. In backend development, the Date and Time API is used for order timestamps, scheduled tasks, session expiration, report generation, SLA calculations, and database persistence. Every Spring Boot application interacts with dates and times, and using the correct types is critical for correctness across time zones and daylight saving transitions.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Classes and Objects]]
> - [[Java - Final Keyword]] (immutability concepts)
> - [[Java - Comparable and Comparator]]
> - [[Java - Java 8 Lambdas and Functional Interfaces]]

---

## Theory

### Why a New Date and Time API?

The legacy `java.util.Date` and `java.util.Calendar` classes, designed in the 1990s, have fundamental design flaws that caused decades of bugs:

| Problem | Legacy (`Date`/`Calendar`) | Modern (`java.time`) |
|---------|--------------------------|---------------------|
| **Mutability** | `Date` is mutable. Any code can call `date.setYear(99)` and corrupt shared state. | All `java.time` classes are **immutable** and thread-safe. |
| **Thread safety** | `SimpleDateFormat` is not thread-safe. Sharing it across threads causes corrupted dates and `NumberFormatException`. | All `java.time` formatters (`DateTimeFormatter`) are immutable and thread-safe. |
| **Month indexing** | `Calendar.JANUARY = 0`, `Date.getMonth()` returns 0-11. Off-by-one errors everywhere. | `Month.JANUARY = 1`. Months are 1-indexed and available as an enum. |
| **Year representation** | `Date.getYear()` returns years since 1900. `new Date(2025, 1, 1)` creates the year 3925. | `LocalDate.of(2025, 1, 1)` creates January 1, 2025. No surprises. |
| **Time zone handling** | `Date` stores UTC internally but `toString()` uses the local time zone. `Calendar` has confusing time zone behavior. | Clear separation: `LocalDateTime` has no time zone, `ZonedDateTime` has an explicit time zone, `Instant` is always UTC. |
| **API design** | `Date` mixes date and time. `Calendar` uses mutable fields and magic constants. | Clean separation of concerns: `LocalDate` (date only), `LocalTime` (time only), `LocalDateTime` (both), `ZonedDateTime` (with zone). |
| **Null safety** | No built-in support. | Works naturally with `Optional` and the Streams API. |

### The Core Classes

The `java.time` package provides a clear hierarchy of types, each representing a specific concept:

| Class | Represents | Example | Time Zone | Use Case |
|-------|-----------|---------|-----------|----------|
| `LocalDate` | Date without time | `2025-07-10` | None | Birthdays, holidays, invoice dates |
| `LocalTime` | Time without date | `14:30:45` | None | Store opening hours, daily schedules |
| `LocalDateTime` | Date and time | `2025-07-10T14:30:45` | None | Application-local timestamps, log entries |
| `ZonedDateTime` | Date, time, and time zone | `2025-07-10T14:30:45+06:00[Asia/Dhaka]` | Yes | Scheduling across time zones, international events |
| `OffsetDateTime` | Date, time, and UTC offset | `2025-07-10T14:30:45+06:00` | Offset only | Database storage, API responses |
| `Instant` | Point on the timeline (UTC) | `2025-07-10T08:30:45Z` | UTC | Timestamps, event ordering, auditing |
| `Duration` | Time-based amount | `PT2H30M` (2 hours 30 minutes) | N/A | Timeouts, SLA calculations, elapsed time |
| `Period` | Date-based amount | `P1Y2M3D` (1 year 2 months 3 days) | N/A | Age calculation, subscription periods |
| `Year`, `Month`, `YearMonth` | Partial dates | `2025`, `JULY`, `2025-07` | None | Monthly reports, fiscal years |

### `LocalDate`

`LocalDate` represents a date (year, month, day) without a time or time zone. It is the correct type for concepts like birthdays, holidays, and invoice due dates where the time of day is irrelevant.

```java
import java.time.*;

LocalDate today = LocalDate.now();                    // 2025-07-10
LocalDate independenceDay = LocalDate.of(1971, 3, 26); // Bangladesh Independence Day
LocalDate parsed = LocalDate.parse("2025-12-25");      // From ISO-8601 string

int year = today.getYear();           // 2025
Month month = today.getMonth();       // JULY (enum, not int!)
int dayOfMonth = today.getDayOfMonth(); // 10
DayOfWeek dayOfWeek = today.getDayOfWeek(); // THURSDAY (enum!)
int dayOfYear = today.getDayOfYear();   // 191
int lengthOfMonth = today.lengthOfMonth(); // 31
boolean isLeap = today.isLeapYear();    // false
```

### `LocalTime`

`LocalTime` represents a time of day (hour, minute, second, nanosecond) without a date or time zone.

```java
import java.time.*;

LocalTime now = LocalTime.now();                // 14:30:45.123456789
LocalTime opening = LocalTime.of(9, 0);         // 09:00
LocalTime parsed = LocalTime.parse("14:30:45");  // From ISO-8601 string

int hour = now.getHour();       // 14
int minute = now.getMinute();   // 30
int second = now.getSecond();   // 45
```

### `LocalDateTime`

`LocalDateTime` combines a `LocalDate` and a `LocalTime`. It represents a date and time without a time zone. This is the most commonly used date-time type in backend applications that operate in a single time zone.

```java
import java.time.*;

LocalDateTime now = LocalDateTime.now();
LocalDateTime specific = LocalDateTime.of(2025, 7, 10, 14, 30, 0);
LocalDateTime fromParts = LocalDateTime.of(LocalDate.now(), LocalTime.NOON);
LocalDateTime parsed = LocalDateTime.parse("2025-07-10T14:30:00");
```

### `Instant`

`Instant` represents a point on the UTC timeline, measured in seconds and nanoseconds from the Unix epoch (1970-01-01T00:00:00Z). It is the correct type for machine-readable timestamps, event ordering, and auditing.

```java
import java.time.*;

Instant now = Instant.now();                          // 2025-07-10T08:30:45.123Z
Instant epoch = Instant.EPOCH;                         // 1970-01-01T00:00:00Z
Instant fromMillis = Instant.ofEpochMilli(1720620000000L);
long millis = now.toEpochMilli();                      // Convert back to milliseconds
```

**`Instant` vs `LocalDateTime`:**

- `Instant` is an absolute point in time. `2025-07-10T08:30:45Z` is the same moment regardless of where you are in the world.
- `LocalDateTime` is a human-readable date and time without a time zone. `2025-07-10T14:30:45` could mean different moments depending on the time zone.

**Rule of thumb**: Use `Instant` for storing and comparing timestamps. Use `LocalDateTime` for displaying dates and times to users in their local context.

### Date Arithmetic: `Period` and `Duration`

**`Period`** represents a date-based amount of time (years, months, days). Use it for human-scale calculations like age, subscription duration, and due dates.

```java
import java.time.*;

Period oneYear = Period.ofYears(1);
Period twoMonths = Period.ofMonths(2);
Period complex = Period.of(1, 2, 15);  // 1 year, 2 months, 15 days

LocalDate birthday = LocalDate.of(2003, 5, 15);
LocalDate today = LocalDate.of(2025, 7, 10);
Period age = Period.between(birthday, today);
System.out.println("Age: " + age.getYears() + " years, "
    + age.getMonths() + " months, " + age.getDays() + " days");
// Age: 22 years, 1 months, 25 days
```

**`Duration`** represents a time-based amount of time (seconds, nanoseconds). Use it for machine-scale calculations like timeouts, response times, and scheduling intervals.

```java
import java.time.*;

Duration twoHours = Duration.ofHours(2);
Duration thirtyMinutes = Duration.ofMinutes(30);
Duration complex = Duration.ofHours(2).plusMinutes(30);  // PT2H30M

Instant start = Instant.now();
// ... some operation ...
Instant end = Instant.now();
Duration elapsed = Duration.between(start, end);
System.out.println("Elapsed: " + elapsed.toMillis() + "ms");
```

### Date Manipulation

All `java.time` classes are immutable. Methods like `plusDays()`, `minusMonths()`, and `withYear()` return **new** instances rather than modifying the original.

```java
import java.time.*;
import java.time.temporal.TemporalAdjusters;

LocalDate today = LocalDate.of(2025, 7, 10);

LocalDate tomorrow = today.plusDays(1);         // 2025-07-11
LocalDate nextMonth = today.plusMonths(1);       // 2025-08-10
LocalDate lastYear = today.minusYears(1);        // 2024-07-10
LocalDate endOfMonth = today.withDayOfMonth(today.lengthOfMonth()); // 2025-07-31
LocalDate startOfYear = today.withDayOfYear(1);  // 2025-01-01

// TemporalAdjusters: predefined date adjustments
LocalDate nextMonday = today.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
LocalDate firstDayOfMonth = today.with(TemporalAdjusters.firstDayOfMonth());
LocalDate lastDayOfYear = today.with(TemporalAdjusters.lastDayOfYear());
LocalDate nextFriday = today.with(TemporalAdjusters.nextOrSame(DayOfWeek.FRIDAY));
```

### Formatting and Parsing

`DateTimeFormatter` is the modern replacement for `SimpleDateFormat`. It is **immutable and thread-safe**, unlike `SimpleDateFormat`.

```java
import java.time.*;
import java.time.format.DateTimeFormatter;

LocalDateTime now = LocalDateTime.of(2025, 7, 10, 14, 30, 45);

// Predefined formatters
String iso = now.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);
// "2025-07-10T14:30:45"

// Custom formatters
DateTimeFormatter banglaFormat = DateTimeFormatter.ofPattern("dd/MM/yyyy hh:mm a");
String formatted = now.format(banglaFormat);
// "10/07/2025 02:30 PM"

DateTimeFormatter shortDate = DateTimeFormatter.ofPattern("dd MMM yyyy");
String shortFormatted = now.format(shortDate);
// "10 Jul 2025"

// Parsing
LocalDate parsed = LocalDate.parse("10/07/2025",
    DateTimeFormatter.ofPattern("dd/MM/yyyy"));
```

**Common pattern symbols:**

| Symbol | Meaning | Example |
|--------|---------|---------|
| `yyyy` | Year (4 digits) | 2025 |
| `MM` | Month (2 digits) | 07 |
| `MMM` | Month (abbreviated) | Jul |
| `MMMM` | Month (full name) | July |
| `dd` | Day of month (2 digits) | 10 |
| `HH` | Hour (24-hour, 2 digits) | 14 |
| `hh` | Hour (12-hour, 2 digits) | 02 |
| `mm` | Minute (2 digits) | 30 |
| `ss` | Second (2 digits) | 45 |
| `a` | AM/PM | PM |
| `Z` | Time zone offset | +0600 |
| `VV` | Time zone ID | Asia/Dhaka |

### Time Zones

Bangladesh Standard Time (BST) is UTC+6. The IANA time zone ID is `Asia/Dhaka`.

```java
import java.time.*;

ZoneId dhakaZone = ZoneId.of("Asia/Dhaka");
ZoneId utcZone = ZoneId.of("UTC");

// Current time in Dhaka
ZonedDateTime dhakaNow = ZonedDateTime.now(dhakaZone);
System.out.println("Dhaka: " + dhakaNow);
// 2025-07-10T14:30:45+06:00[Asia/Dhaka]

// Convert between time zones
ZonedDateTime utcNow = dhakaNow.withZoneSameInstant(utcZone);
System.out.println("UTC: " + utcNow);
// 2025-07-10T08:30:45Z[UTC]

// LocalDateTime to ZonedDateTime
LocalDateTime local = LocalDateTime.of(2025, 7, 10, 14, 30);
ZonedDateTime zoned = local.atZone(dhakaZone);
Instant instant = zoned.toInstant();
```

> [!tip] Key Insight
> The most important decision in backend date-time handling is choosing the right type for the right concept. Use `LocalDate` for dates without time (birthdays, due dates). Use `LocalDateTime` for local timestamps (log entries, UI display). Use `Instant` for absolute timestamps (auditing, event ordering, API contracts). Use `ZonedDateTime` only when you need to perform time zone conversions. Storing `LocalDateTime` in a database when you mean `Instant` is the most common date-time bug in backend systems, because `LocalDateTime` loses the time zone context and becomes ambiguous during daylight saving transitions.

---

## Syntax and Basic Examples

### Example 1: Creating and querying dates

```java
import java.time.*;
import java.time.temporal.ChronoUnit;

public class DateCreationDemo {
    public static void main(String[] args) {
        // Current date and time
        LocalDate today = LocalDate.now();
        LocalTime now = LocalTime.now();
        LocalDateTime dateTime = LocalDateTime.now();
        Instant instant = Instant.now();

        System.out.println("Today: " + today);          // 2025-07-10
        System.out.println("Now: " + now);              // 14:30:45.123
        System.out.println("DateTime: " + dateTime);    // 2025-07-10T14:30:45.123
        System.out.println("Instant: " + instant);      // 2025-07-10T08:30:45.123Z

        // Specific dates
        LocalDate independence = LocalDate.of(1971, Month.MARCH, 26);
        LocalDate victory = LocalDate.of(1971, 12, 16);

        // Querying
        System.out.println("\nIndependence Day: " + independence);
        System.out.println("Day of week: " + independence.getDayOfWeek());  // FRIDAY
        System.out.println("Day of year: " + independence.getDayOfYear());  // 85
        System.out.println("Is leap year: " + independence.isLeapYear());   // false
        System.out.println("Month length: " + independence.lengthOfMonth()); // 31

        // Comparing
        System.out.println("\nIndependence before Victory: " + independence.isBefore(victory)); // true
        System.out.println("Days between: " + ChronoUnit.DAYS.between(independence, victory));  // 265

        // Age calculation
        LocalDate birthday = LocalDate.of(2003, 5, 15);
        Period age = Period.between(birthday, today);
        System.out.printf("Age: %d years, %d months, %d days%n",
            age.getYears(), age.getMonths(), age.getDays());
    }
}
```

### Example 2: Date arithmetic and TemporalAdjusters

```java
import java.time.*;
import java.time.temporal.TemporalAdjusters;

public class DateArithmeticDemo {
    public static void main(String[] args) {
        LocalDate date = LocalDate.of(2025, 7, 10);

        // Basic arithmetic
        System.out.println("Original: " + date);
        System.out.println("+1 day: " + date.plusDays(1));
        System.out.println("+1 month: " + date.plusMonths(1));
        System.out.println("+1 year: " + date.plusYears(1));
        System.out.println("-2 weeks: " + date.minusWeeks(2));

        // with() methods
        System.out.println("\nFirst day of month: " + date.withDayOfMonth(1));
        System.out.println("Last day of month: " + date.withDayOfMonth(date.lengthOfMonth()));
        System.out.println("Day 100 of year: " + date.withDayOfYear(100));

        // TemporalAdjusters
        System.out.println("\nNext Monday: " + date.with(TemporalAdjusters.next(DayOfWeek.MONDAY)));
        System.out.println("Previous Friday: " + date.with(TemporalAdjusters.previous(DayOfWeek.FRIDAY)));
        System.out.println("First day of year: " + date.with(TemporalAdjusters.firstDayOfYear()));
        System.out.println("Last day of year: " + date.with(TemporalAdjusters.lastDayOfYear()));
        System.out.println("First day of next month: " + date.with(TemporalAdjusters.firstDayOfNextMonth()));
        System.out.println("Next or same Monday: " + date.with(TemporalAdjusters.nextOrSame(DayOfWeek.MONDAY)));

        // Duration arithmetic
        Instant start = Instant.parse("2025-07-10T08:00:00Z");
        Instant end = Instant.parse("2025-07-10T10:30:00Z");
        Duration elapsed = Duration.between(start, end);
        System.out.println("\nElapsed: " + elapsed.toHours() + "h " +
            elapsed.toMinutesPart() + "m");  // 2h 30m
    }
}
```

### Example 3: Formatting and parsing

```java
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.format.FormatStyle;
import java.util.Locale;

public class FormattingDemo {
    public static void main(String[] args) {
        LocalDateTime dateTime = LocalDateTime.of(2025, 7, 10, 14, 30, 45);
        LocalDate date = dateTime.toLocalDate();

        // ISO-8601 (default, used for APIs and databases)
        System.out.println("ISO date: " + date.format(DateTimeFormatter.ISO_LOCAL_DATE));
        // 2025-07-10
        System.out.println("ISO datetime: " + dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
        // 2025-07-10T14:30:45

        // Custom patterns
        DateTimeFormatter banglaDate = DateTimeFormatter.ofPattern("dd/MM/yyyy");
        System.out.println("BD format: " + date.format(banglaDate));  // 10/07/2025

        DateTimeFormatter readable = DateTimeFormatter.ofPattern("EEEE, dd MMMM yyyy");
        System.out.println("Readable: " + date.format(readable));
        // Thursday, 10 July 2025

        DateTimeFormatter withTime = DateTimeFormatter.ofPattern("dd MMM yyyy, hh:mm a");
        System.out.println("With time: " + dateTime.format(withTime));
        // 10 Jul 2025, 02:30 PM

        // Localized formatting
        DateTimeFormatter localized = DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG)
            .withLocale(Locale.US);
        System.out.println("US long: " + date.format(localized));  // July 10, 2025

        // Parsing
        LocalDate parsed1 = LocalDate.parse("2025-07-10");  // ISO-8601 (no formatter needed)
        LocalDate parsed2 = LocalDate.parse("10/07/2025", banglaDate);
        LocalDateTime parsed3 = LocalDateTime.parse("10 Jul 2025, 02:30 PM", withTime);

        System.out.println("\nParsed 1: " + parsed1);
        System.out.println("Parsed 2: " + parsed2);
        System.out.println("Parsed 3: " + parsed3);
    }
}
```

### Example 4: Time zones and conversions

```java
import java.time.*;

public class TimeZoneDemo {
    public static void main(String[] args) {
        ZoneId dhaka = ZoneId.of("Asia/Dhaka");
        ZoneId london = ZoneId.of("Europe/London");
        ZoneId newYork = ZoneId.of("America/New_York");

        // Current time in different zones
        ZonedDateTime dhakaTime = ZonedDateTime.now(dhaka);
        ZonedDateTime londonTime = dhakaTime.withZoneSameInstant(london);
        ZonedDateTime nyTime = dhakaTime.withZoneSameInstant(newYork);

        System.out.println("Dhaka:   " + dhakaTime);
        System.out.println("London:  " + londonTime);
        System.out.println("New York:" + nyTime);

        // Converting LocalDateTime to Instant (requires a time zone)
        LocalDateTime local = LocalDateTime.of(2025, 7, 10, 14, 30);
        Instant instant = local.atZone(dhaka).toInstant();
        System.out.println("\nLocal (Dhaka): " + local);
        System.out.println("Instant (UTC): " + instant);

        // Converting Instant to LocalDateTime in a specific zone
        LocalDateTime inLondon = instant.atZone(london).toLocalDateTime();
        System.out.println("In London: " + inLondon);

        // OffsetDateTime (useful for database storage)
        OffsetDateTime offset = OffsetDateTime.now(ZoneOffset.ofHours(6));
        System.out.println("\nOffset (+06:00): " + offset);
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Date and time handling is critical in every backend system. Here are three realistic scenarios.

### Scenario 1: JPA entity with date-time fields

The choice of date-time type in JPA entities affects how data is stored in the database and how it behaves across time zones.

```java
package com.company.orderservice.model;

import jakarta.persistence.*;
import java.time.*;

@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String orderNumber;

    // LocalDateTime: stores date and time WITHOUT time zone.
    // The database stores exactly what you give it.
    // Use this when the time zone is implied by the application context
    // (e.g., a single-country application operating in Asia/Dhaka).
    @Column(nullable = false)
    private LocalDateTime createdAt;

    @Column
    private LocalDateTime updatedAt;

    // LocalDate: stores date only, no time component.
    // Use this for business dates where the time of day is irrelevant.
    @Column(nullable = false)
    private LocalDate deliveryDate;

    // Instant: stores an absolute point in time (UTC).
    // The database converts to/from the server's time zone automatically.
    // Use this for auditing and event ordering across time zones.
    @Column
    private Instant paidAt;

    @Column
    private Instant cancelledAt;

    protected Order() {}  // For JPA

    public Order(String orderNumber, LocalDate deliveryDate) {
        this.orderNumber = orderNumber;
        this.createdAt = LocalDateTime.now();  // Application-local time
        this.updatedAt = LocalDateTime.now();
        this.deliveryDate = deliveryDate;
    }

    public void markAsPaid() {
        this.paidAt = Instant.now();  // Absolute UTC timestamp
        this.updatedAt = LocalDateTime.now();
    }

    public void cancel() {
        this.cancelledAt = Instant.now();
        this.updatedAt = LocalDateTime.now();
    }

    // Check if the order is overdue
    public boolean isOverdue() {
        return this.deliveryDate.isBefore(LocalDate.now())
            && this.paidAt == null;
    }

    // Calculate days until delivery
    public long getDaysUntilDelivery() {
        return java.time.temporal.ChronoUnit.DAYS.between(LocalDate.now(), this.deliveryDate);
    }

    // Getters...
    public Long getId() { return id; }
    public String getOrderNumber() { return orderNumber; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDate getDeliveryDate() { return deliveryDate; }
    public Instant getPaidAt() { return paidAt; }
}
```

**What to notice:**

- `LocalDateTime` is used for `createdAt` and `updatedAt`. This is appropriate for a single-country application where all servers and users are in the same time zone. The database stores the exact date and time without time zone conversion.
- `LocalDate` is used for `deliveryDate`. Delivery dates are calendar dates, not specific moments in time. A delivery on July 15 is July 15 regardless of the time zone.
- `Instant` is used for `paidAt` and `cancelledAt`. These are absolute events that need to be ordered precisely across time zones. If a payment is processed at 14:30 in Dhaka and 08:30 in London, both timestamps represent the same moment and should be stored as the same `Instant`.
- The `isOverdue()` method compares `LocalDate` values using `isBefore()`. This is clean and readable compared to the legacy approach of comparing `Date.getTime()` millisecond values.

### Scenario 2: Date-based query methods in Spring Data

```java
package com.company.orderservice.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.time.*;
import java.util.List;

public interface OrderRepository extends JpaRepository<Order, Long> {

    // Spring Data derives the query from the method name
    List<Order> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    List<Order> findByDeliveryDate(LocalDate date);

    List<Order> findByDeliveryDateBefore(LocalDate date);

    List<Order> findByPaidAtIsNullAndDeliveryDateBefore(LocalDate date);

    // Custom JPQL query with date parameters
    @Query("SELECT o FROM Order o WHERE o.createdAt >= :since AND o.status = 'DELIVERED'")
    List<Order> findDeliveredOrdersSince(LocalDateTime since);

    // Count orders per day for a date range
    @Query("SELECT o.deliveryDate, COUNT(o) FROM Order o " +
           "WHERE o.deliveryDate BETWEEN :start AND :end " +
           "GROUP BY o.deliveryDate ORDER BY o.deliveryDate")
    List<Object[]> countOrdersPerDay(LocalDate start, LocalDate end);
}
```

```java
// Usage in a service:
import org.springframework.stereotype.Service;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.util.List;

@Service
public class ReportService {

    public List<Order> getTodayOrders() {
        LocalDateTime startOfDay = LocalDate.now().atStartOfDay();
        LocalDateTime endOfDay = LocalDate.now().atTime(LocalTime.MAX);
        return orderRepository.findByCreatedAtBetween(startOfDay, endOfDay);
    }

    public List<Order> getOverdueOrders() {
        return orderRepository.findByPaidAtIsNullAndDeliveryDateBefore(LocalDate.now());
    }

    public List<Order> getThisMonthOrders() {
        LocalDate firstDay = LocalDate.now().withDayOfMonth(1);
        LocalDate lastDay = LocalDate.now().withDayOfMonth(LocalDate.now().lengthOfMonth());
        return orderRepository.findByCreatedAtBetween(
            firstDay.atStartOfDay(),
            lastDay.atTime(LocalTime.MAX)
        );
    }

    public List<Order> getLast7DaysOrders() {
        LocalDateTime sevenDaysAgo = LocalDateTime.now().minusDays(7);
        return orderRepository.findDeliveredOrdersSince(sevenDaysAgo);
    }
}
```

**What to notice:**

- `LocalDate.now().atStartOfDay()` converts a `LocalDate` to a `LocalDateTime` at midnight (00:00:00). This is the standard pattern for "start of day" queries.
- `LocalDate.now().atTime(LocalTime.MAX)` creates a `LocalDateTime` at the last nanosecond of the day (23:59:59.999999999). This ensures that the query includes all orders created on that day.
- Spring Data JPA handles the conversion between Java `java.time` types and SQL `DATE`, `TIMESTAMP`, and `TIMESTAMP WITH TIME ZONE` columns automatically. No manual conversion is needed.

### Scenario 3: SLA calculation and scheduling

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.time.Duration;
import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.util.List;
import java.util.Map;
import java.util.TreeMap;

@Service
public class SlaService {

    private static final Duration STANDARD_SLA = Duration.ofHours(48);
    private static final Duration EXPRESS_SLA = Duration.ofHours(24);
    private static final Duration SAME_DAY_SLA = Duration.ofHours(6);

    public SlaStatus checkSla(Order order) {
        if (order.getPaidAt() == null) {
            return new SlaStatus("PENDING_PAYMENT", Duration.ZERO, false);
        }

        Duration sla = switch (order.getShippingMethod()) {
            case EXPRESS -> EXPRESS_SLA;
            case SAME_DAY -> SAME_DAY_SLA;
            default -> STANDARD_SLA;
        };

        Instant deadline = order.getPaidAt().plus(sla);
        Instant now = Instant.now();
        Duration remaining = Duration.between(now, deadline);

        boolean isBreached = remaining.isNegative();
        String status = isBreached ? "BREACHED" : "ON_TRACK";

        return new SlaStatus(status, remaining.abs(), isBreached);
    }

    public Duration getAverageFulfillmentTime(List<Order> orders) {
        long totalSeconds = orders.stream()
            .filter(o -> o.getPaidAt() != null && o.getDeliveredAt() != null)
            .mapToLong(o -> Duration.between(o.getPaidAt(), o.getDeliveredAt()).getSeconds())
            .sum();

        long count = orders.stream()
            .filter(o -> o.getPaidAt() != null && o.getDeliveredAt() != null)
            .count();

        return count > 0 ? Duration.ofSeconds(totalSeconds / count) : Duration.ZERO;
    }

    public Map<LocalDate, Long> getDailyOrderCounts(int days) {
        LocalDate today = LocalDate.now();
        Map<LocalDate, Long> counts = new TreeMap<>();

        for (int i = 0; i < days; i++) {
            LocalDate date = today.minusDays(i);
            LocalDateTime start = date.atStartOfDay();
            LocalDateTime end = date.atTime(LocalTime.MAX);
            long count = orderRepository.findByCreatedAtBetween(start, end).size();
            counts.put(date, count);
        }

        return counts;  // TreeMap ensures chronological order
    }
}

record SlaStatus(String status, Duration remaining, boolean isBreached) {
    public String getRemainingFormatted() {
        long hours = remaining.toHours();
        long minutes = remaining.toMinutesPart();
        return String.format("%dh %dm", hours, minutes);
    }
}
```

**What to notice:**

- `Duration` is used for SLA calculations because SLAs are time-based (hours, minutes), not date-based (years, months). `Duration.between()` calculates the exact elapsed time between two `Instant` values.
- `Instant.plus(Duration)` calculates the deadline by adding the SLA duration to the payment timestamp. This is an absolute calculation that works correctly across time zones and daylight saving transitions.
- The `getDailyOrderCounts()` method uses `LocalDate` for the map keys and `LocalDateTime` for the query range. The `TreeMap` ensures the results are sorted chronologically.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Using the legacy `java.util.Date` in new code

**Wrong:**

```java
import java.util.Date;
import java.text.SimpleDateFormat;

public class Order {
    private Date createdAt;  // Mutable, not thread-safe, confusing API

    public String formatDate() {
        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
        return sdf.format(createdAt);  // NOT thread-safe! Sharing this across
        // threads causes corrupted dates and exceptions.
    }
}
```

**Right:**

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Order {
    private LocalDateTime createdAt;  // Immutable, thread-safe, clear API

    private static final DateTimeFormatter FORMATTER =
        DateTimeFormatter.ofPattern("dd/MM/yyyy");
        // DateTimeFormatter IS thread-safe. Safe to share as a static constant.

    public String formatDate() {
        return createdAt.format(FORMATTER);
    }
}
```

**Why it is wrong:** `java.util.Date` is mutable (any code can call `setTime()`), uses confusing indexing (months are 0-based, years are since 1900), and its companion `SimpleDateFormat` is not thread-safe. Using these classes in a multi-threaded backend server causes data corruption and `NumberFormatException`. The `java.time` API was designed specifically to fix all of these problems. Never use `Date`, `Calendar`, or `SimpleDateFormat` in new code.

### Mistake 2: Using `LocalDateTime` when `Instant` is needed

**Wrong:**

```java
// Storing an event timestamp as LocalDateTime
private LocalDateTime eventOccurredAt;

// Problem: LocalDateTime has no time zone. If the server is in Dhaka (UTC+6)
// and the client is in London (UTC+0), the same event will have different
// LocalDateTime values depending on which server processes it.
// Sorting events by LocalDateTime across servers gives wrong results.
```

**Right:**

```java
// Storing an event timestamp as Instant (absolute UTC)
private Instant eventOccurredAt;

// Instant is always UTC. The same event has the same Instant value
// regardless of which server or time zone processes it.
// Sorting by Instant always gives the correct chronological order.
```

**Why it is wrong:** `LocalDateTime` represents a date and time without a time zone. It is ambiguous: "2025-07-10T14:30" could mean 14:30 in Dhaka, 14:30 in London, or 14:30 in New York -- three different moments in time. For timestamps that need to be compared, sorted, or shared across time zones, use `Instant`. Use `LocalDateTime` only for display purposes or for applications that operate entirely within a single time zone.

### Mistake 3: Mutating date-time objects (they are immutable!)

**Wrong:**

```java
LocalDate date = LocalDate.now();
date.plusDays(7);  // Return value is DISCARDED!
System.out.println(date);  // Still today's date, not 7 days from now!
```

**Right:**

```java
LocalDate date = LocalDate.now();
date = date.plusDays(7);  // Assign the return value!
System.out.println(date);  // 7 days from now
```

**Why it is wrong:** All `java.time` classes are immutable. Methods like `plusDays()`, `minusMonths()`, and `withYear()` return a **new** object. They do not modify the original. If you discard the return value, the original object is unchanged. This is the same pattern as `String` methods: `str.toUpperCase()` does not modify `str`; it returns a new string.

### Mistake 4: Using `==` to compare date-time objects

**Wrong:**

```java
LocalDate date1 = LocalDate.of(2025, 7, 10);
LocalDate date2 = LocalDate.of(2025, 7, 10);
System.out.println(date1 == date2);  // false! Different objects in memory.
```

**Right:**

```java
LocalDate date1 = LocalDate.of(2025, 7, 10);
LocalDate date2 = LocalDate.of(2025, 7, 10);
System.out.println(date1.equals(date2));  // true (same value)
System.out.println(date1.isEqual(date2));  // true (same value, date-specific method)
System.out.println(date1.isBefore(date2));  // false
System.out.println(date1.isAfter(date2));   // false
```

**Why it is wrong:** `==` compares object references (memory addresses), not values. Two `LocalDate` objects with the same date are different objects in memory. Use `equals()` for value comparison or the date-specific methods `isEqual()`, `isBefore()`, and `isAfter()` for readability.

---

## Key Takeaways

> [!tip] Remember these points
> 1. The `java.time` API (Java 8+) replaces the broken legacy `Date`, `Calendar`, and `SimpleDateFormat` classes. All `java.time` classes are **immutable and thread-safe**. Never use the legacy classes in new code.
> 2. Choose the right type for the right concept: `LocalDate` for dates without time, `LocalTime` for times without dates, `LocalDateTime` for local timestamps, `Instant` for absolute UTC timestamps, `ZonedDateTime` for time zone-aware calculations, `Duration` for time-based amounts, and `Period` for date-based amounts.
> 3. All date-time objects are **immutable**. Methods like `plusDays()` and `withMonth()` return new objects. Always assign the return value: `date = date.plusDays(7)`.
> 4. Use `DateTimeFormatter` for formatting and parsing. It is immutable and thread-safe, unlike `SimpleDateFormat`. Use ISO-8601 format (`yyyy-MM-dd`, `yyyy-MM-ddTHH:mm:ss`) for APIs and databases. Use custom patterns only for user-facing display.
> 5. In backend systems, use `Instant` for auditing and event ordering, `LocalDate` for business dates, and `LocalDateTime` for application-local timestamps. Store timestamps in the database as `TIMESTAMP WITH TIME ZONE` (mapped to `Instant` or `OffsetDateTime`) to preserve time zone information across distributed systems.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Date Basics (Easy)

Write a program that:

1. Prints today's date, the current time, and the current date-time.
2. Creates a `LocalDate` for your birthday and calculates your exact age (years, months, days) using `Period.between()`.
3. Calculates the number of days between Bangladesh's Independence Day (March 26, 1971) and Victory Day (December 16, 1971).
4. Determines the day of the week for January 1, 2030.
5. Checks if the current year is a leap year.

> **Hint:** Use `LocalDate.of()` to create specific dates, `Period.between()` for age calculation, and `ChronoUnit.DAYS.between()` for day counts.

### Exercise 2: Date Formatting and Parsing (Medium)

Write a program that:

1. Formats the current date-time in at least 4 different patterns (ISO, dd/MM/yyyy, "EEEE, dd MMMM yyyy", "hh:mm a").
2. Parses date strings in different formats back into `LocalDate` objects.
3. Handles a parsing error gracefully using try-catch (e.g., parsing "31/02/2025" which is an invalid date).
4. Converts a `LocalDateTime` to a formatted string and back, verifying that the round-trip preserves the value.

> **Hint:** Use `DateTimeFormatter.ofPattern()` for custom formats. Catch `DateTimeParseException` for invalid date strings.

### Exercise 3: Order SLA Calculator (Medium)

Create an `Order` record with fields `orderNumber`, `createdAt` (Instant), `shippingMethod` (String: "STANDARD", "EXPRESS", "SAME_DAY"), and `deliveredAt` (Instant, nullable). Write a service that:

1. Calculates the SLA deadline for each order (48h for STANDARD, 24h for EXPRESS, 6h for SAME_DAY).
2. Determines if the order was delivered on time.
3. Calculates the actual fulfillment duration.
4. Formats the duration as "Xh Ym" for display.

Test with 5 orders: some delivered on time, some late, some not yet delivered.

> **Hint:** Use `Instant.plus(Duration)` to calculate deadlines. Use `Duration.between()` to calculate elapsed time. Use `Duration.isNegative()` to check if a deadline has passed.

### Exercise 4: Monthly Report Generator (Hard, Optional)

Build a report generator that produces a daily summary for a given month:

1. Generate 100 random orders with `createdAt` timestamps spread across the specified month.
2. Group orders by date using `Collectors.groupingBy()` with `order.getCreatedAt().toLocalDate()` as the classifier.
3. For each day, calculate: order count, total revenue, and average order value.
4. Find the busiest day and the quietest day.
5. Calculate the day-over-day growth rate.
6. Format the report as a table with aligned columns.

> **Hint:** Use `LocalDate.datesUntil()` (Java 9+) or a loop with `plusDays(1)` to iterate over all days in the month. Use `TreeMap<LocalDate, List<Order>>` to ensure chronological ordering.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.time.*;
import java.time.temporal.ChronoUnit;

public class DateBasics {
    public static void main(String[] args) {
        System.out.println("Today: " + LocalDate.now());
        System.out.println("Time: " + LocalTime.now());
        System.out.println("DateTime: " + LocalDateTime.now());

        LocalDate birthday = LocalDate.of(2003, 5, 15);
        Period age = Period.between(birthday, LocalDate.now());
        System.out.printf("Age: %d years, %d months, %d days%n",
            age.getYears(), age.getMonths(), age.getDays());

        LocalDate independence = LocalDate.of(1971, 3, 26);
        LocalDate victory = LocalDate.of(1971, 12, 16);
        System.out.println("Days between: " + ChronoUnit.DAYS.between(independence, victory));

        LocalDate jan2030 = LocalDate.of(2030, 1, 1);
        System.out.println("Jan 1 2030: " + jan2030.getDayOfWeek());

        System.out.println("Leap year: " + LocalDate.now().isLeapYear());
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
import java.time.*;
import java.util.*;

record Order(String orderNumber, Instant createdAt, String shippingMethod, Instant deliveredAt) {}

public class SlaCalculator {
    static Duration getSla(String method) {
        return switch (method) {
            case "SAME_DAY" -> Duration.ofHours(6);
            case "EXPRESS" -> Duration.ofHours(24);
            default -> Duration.ofHours(48);
        };
    }

    static String formatDuration(Duration d) {
        return String.format("%dh %dm", d.toHours(), d.toMinutesPart());
    }

    public static void main(String[] args) {
        Instant now = Instant.now();
        List<Order> orders = List.of(
            new Order("ORD-1", now.minusHours(50), "STANDARD", now.minusHours(2)),
            new Order("ORD-2", now.minusHours(30), "EXPRESS", now.minusHours(2)),
            new Order("ORD-3", now.minusHours(10), "SAME_DAY", now.minusHours(2)),
            new Order("ORD-4", now.minusHours(5), "EXPRESS", null),
            new Order("ORD-5", now.minusHours(60), "STANDARD", null)
        );

        for (Order o : orders) {
            Duration sla = getSla(o.shippingMethod());
            Instant deadline = o.createdAt().plus(sla);
            boolean onTime = o.deliveredAt() != null && !o.deliveredAt().isAfter(deadline);
            String fulfillment = o.deliveredAt() != null
                ? formatDuration(Duration.between(o.createdAt(), o.deliveredAt()))
                : "Not delivered";

            System.out.printf("%s [%s] SLA: %s | Fulfillment: %s | %s%n",
                o.orderNumber(), o.shippingMethod(),
                formatDuration(sla), fulfillment,
                o.deliveredAt() == null ? "PENDING" : (onTime ? "ON TIME" : "LATE"));
        }
    }
}
```

</details>

---

## Related Notes

- [[Java - Optional Class]]
- [[Java - Java 8 Streams API]]
- [[Java - Enum and Enum Methods]] (next note)
- [[Java - File I-O - FileReader FileWriter BufferedReader]]

---

## Resources

- [Oracle Java Tutorials: Date Time](https://docs.oracle.com/javase/tutorial/datetime/) - Official documentation covering the entire java.time package.
- [Oracle Java Documentation: java.time](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/package-summary.html) - Complete API reference for all date-time classes.
- [Baeldung: Java 8 Date and Time API](https://www.baeldung.com/java-8-date-time-intro) - Comprehensive guide with examples for all core classes.
- [Baeldung: Java DateTimeFormatter](https://www.baeldung.com/java-datetimeformatter) - Detailed guide to formatting and parsing patterns.
- [Baeldung: JPA Java 8 Date and Time](https://www.baeldung.com/jpa-java-time) - Guide to mapping java.time types to database columns with JPA/Hibernate. Essential for backend development.
- [Effective Java by Joshua Bloch - Item 58: Prefer for-each Loops to Traditional for Loops](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Related to iterating over date ranges.
- [JSR-310: Date and Time API](https://jcp.org/en/jsr/detail?id=310) - The original specification for the java.time API. Useful for understanding the design decisions.
